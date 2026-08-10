# ADR-006: 로그는 구조화 JSON으로 남기고, 메트릭은 Prometheus 포맷으로 노출한다

## 1. Overview

- Date: 2026-07-25
- Status: Proposed
- Deciders: 근흐흐
- Tracking: YMC-252
- Implements: 로그 포맷·공통 필드·correlation ID 전파·메트릭 노출 방식을 정한다. 상세 로깅 규칙은 `conventions/logging.md`가 소유한다.
- Related: ADR-002(SQS), ADR-004(SSE), ADR-005(관측성 파이프라인)
- Supersedes / Superseded by: -

## 2. Context

BE(Spring Boot)와 AI(FastAPI)는 서로 다른 스택을 사용한다. 두 서비스의 로그를 함께 조회하고 SQS·HTTP·SSE를 넘나드는 흐름을 찾으려면 공통 포맷과 식별자가 필요하다.

앱에 들어가는 계측 규칙은 저장소보다 바꾸기 어렵다. 따라서 이 ADR은 앱이 로그와 메트릭을 만드는 방식만 정하고, 수집·저장 방식은 ADR-005에 맡긴다.

## 3. Decision

### 3.1 구조화 로그

- 로그는 한 이벤트당 JSON 한 줄로 표준 출력에 남긴다.
- BE는 Spring Boot 내장 ECS 포맷(`logging.structured.format.console=ecs`)을 사용한다.
- AI는 Python logging에 JSON formatter를 사용한다.
- 두 서비스는 아래 조회 필드의 이름과 타입을 맞춘다.

실제 로그는 한 줄로 출력한다. 아래 예시는 읽기 쉽도록 줄을 나눴으며, 포매터가 자동으로 추가하는 부가 필드는 생략했다.

```json
{
  "@timestamp": "2026-07-25T09:12:33.123Z",
  "log": {
    "level": "ERROR"
  },
  "service": {
    "name": "be"
  },
  "message": "파싱 응답 대기 시간 초과",
  "event_action": "paper.parse.failed",
  "correlation_id": "d3f1a0...",
  "paper_id": "0198c9a2-7e31-7c8b-b1a4-...",
  "duration_ms": 30000
}
```

| 필드 | 설명 |
|---|---|
| `@timestamp` | UTC 시각, 포매터 자동 생성 |
| `log.level` | 로그 레벨 |
| `service.name` | `be` 또는 `ai` |
| `message` | 사람이 읽는 설명 |
| `event_action` | 집계할 이벤트 이름 |
| `correlation_id` | 서비스 간 요청 흐름 식별자 |
| `paper_id` 등 도메인 ID | 해당 이벤트에 필요할 때만 추가 |
| `duration_ms` | 소요시간(ms), 해당 이벤트에 필요할 때만 추가 |

Spring Boot ECS의 기본 필드는 중첩 JSON 객체로 출력된다. 표의 `log.level`, `service.name`은 JSON 키가 아니라 조회 경로를 뜻한다. 커스텀 필드는 소문자 `snake_case`를 사용한다.

첫 구현 태스크에서 BE와 AI의 실제 출력 한 줄을 비교하는 테스트를 추가해 필드명과 타입을 확정한다.

### 3.2 correlation ID 전파

- BE는 외부 요청의 `X-Correlation-Id`가 없거나 유효하지 않으면 새 값을 만들고 응답 헤더에 돌려준다.
- BE↔AI HTTP/SSE 요청은 `X-Correlation-Id` 헤더로 전달한다.
- `parse-requests`와 `parse-results` SQS 메시지는 같은 값을 `correlationId` 필드로 전달한다. 필드 정의는 메시지 계약이 소유한다.
- BE는 MDC, AI는 contextvars에 값을 넣어 해당 흐름의 모든 앱 로그에 `correlation_id`로 기록한다.
- 비동기 실행과 스레드 전환에서도 값이 유지되는지 통합 테스트한다.

### 3.3 메트릭 노출

- BE는 Actuator + Micrometer, AI는 prometheus-client로 `/metrics`를 노출한다.
- 어떤 앱 메트릭을 수집할지와 라벨 규칙은 이 ADR에서 정하지 않는다.

### 3.4 LLM 실행 추적

LLM 호출과 LangGraph 실행 단계는 LangSmith로 추적한다.

### 3.5 민감정보

CloudWatch 앱 로그에는 인증정보, 논문 본문, 개인정보 원문, 사용자 질문과 LLM 응답 본문을 남기지 않는다. 필요한 경우 ID·길이·토큰 수처럼 본문을 복원할 수 없는 값만 기록한다.

## 4. Options Considered

### Option A. 구조화 JSON 로그 — 채택

- 서비스가 달라도 같은 필드로 로그를 조회할 수 있다.
- 저장소를 바꿔도 앱 계측을 재사용할 수 있다.

### Option B. 평문 로그

- 바로 읽기는 쉽지만 조회할 때마다 문자열 파싱 규칙이 필요하고 포맷 변경에 취약하다.

## 5. Consequences

### Positive

- correlation ID로 BE와 AI의 관련 로그를 함께 찾을 수 있다.
- 로그 필드와 메트릭 형식이 서비스마다 달라지는 것을 막는다.

### Trade-offs

- JSON 원문은 평문 로그보다 사람이 바로 읽기 어렵다.
- MDC와 contextvars가 비동기 경계에서 유지되도록 구현과 테스트가 필요하다.
- 필드와 라벨을 추가할 때 조회 목적과 카디널리티를 검토해야 한다.

### Follow-ups

- SQS 메시지 계약에 `correlationId`를 추가한다.
- BE·AI 구조화 로그 출력 비교 테스트를 추가한다.
- BE MDC와 AI contextvars 전파를 통합 테스트한다.
- 앱 메트릭 수집이 필요해지면 수집 대상과 라벨 허용·금지 규칙을 정한다.

## 6. Updates

- (없음)
