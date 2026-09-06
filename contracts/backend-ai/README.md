# Backend ↔ AI 계약

이 디렉토리는 Backend 서버와 AI 서버 사이의 wire contract를 전송 방식별로 정의한다.
구현 기준은 `team-ymc/ai` 저장소의 현재 FastAPI router, Pydantic schema, SSE event model,
document parser worker와 관련 테스트다.

- 마지막 동기화: `team-ymc/ai@153462a` (2026-09-06)

## 계약 선택

| 통신 방식 | 진입점 | 용도 |
|---|---|---|
| HTTP request/response | [`openapi.yml`](openapi.yml) | agent 실행과 legacy HTTP document parser |
| HTTP + SSE | [`sse-contract.yml`](sse-contract.yml), [`sse/`](sse/) | 장기 실행 응답의 event 이름, 순서와 payload |
| SQS | [`sqs/README.md`](sqs/README.md), [`sqs/messaging.yml`](sqs/messaging.yml) | 비동기 document parser 요청과 최종 결과 |

OpenAPI의 `/stream` path는 HTTP request body와 최초 응답만 표현한다. stream 내부의
event 계약은 반드시 endpoint별 SSE 파일을 함께 읽는다. SQS worker는 별도 프로세스이며
OpenAPI endpoint가 아니다.

## 현재 기능

| 기능 | 일반 HTTP | SSE | SQS |
|---|---:|---:|---:|
| Simple agent | O | O | - |
| Base PDF agent | O | O | - |
| Inline PDF agent | O | O | - |
| Universal PDF agent | O | O | - |
| Inline translate agent | O | O | - |
| Document parser (legacy) | O | O | - |
| Document parser worker | - | - | O |

## 공통 식별자와 선택 영역

- `thread_id`는 client가 정하는 대화 또는 요청 상관관계 식별자다.
- `paper_id`는 paper package 식별자다. AI는 빈 값, 절대 경로, `/`, `\\`, `.`, `..`처럼
  안전한 단일 path segment가 아닌 값을 거부한다.
- inline PDF, universal PDF, inline translate의 text offset은 JavaScript 문자열과 같은
  UTF-16 code unit 기준이다. `start.offset`은 inclusive, `end.offset`은 exclusive다.
- image, table, formula block은 atomic이다. 이 block의 anchor에는 `offset`을 보내지 않는다.
- 선택 영역은 `structure/document.json` schema version 2의 document-wide `blocks` 순서를
  기준으로 해석한다.

## 오류 처리

- 일반 HTTP endpoint의 request validation 실패는 HTTP `422`다.
- 일반 HTTP 실행 중 알려진 오류는 `detail: {code, message}` 형태의 HTTP 오류로 반환된다.
- 유효한 `/stream` request가 HTTP `200`으로 시작된 뒤 발생한 실행 오류는 HTTP status를
  바꿀 수 없으므로 terminal SSE event로 전달된다.
- SQS는 at-least-once 전송이다. 결과 발행과 request ACK 순서는
  [`sqs/README.md`](sqs/README.md)를 따른다.
- client는 사람이 읽는 `message` 문자열이 아니라 안정적인 `code`, `status`, event 이름으로
  분기한다.

## 호환성과 갱신

- 송신자는 문서에 정의되지 않은 필드를 보내지 않는다.
- 수신자는 특별히 금지된 경우가 아니면 알 수 없는 필드를 무시해 forward compatibility를
  확보할 수 있다.
- model provider와 server-owned variant는 wire field가 아니다. variant 변경만으로 client
  payload를 바꾸지 않는다.
- AI router/schema/event/message model을 변경할 때 이 디렉토리와 contract validation을 같은
  변경에서 갱신한다.
