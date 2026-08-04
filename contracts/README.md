# Contracts

FE·BE·AI 사이 계약의 SSOT 디렉토리

구조는 아래와 같다.

```
contracts/
├── README.md
├── frontend-backend/
│   └── openapi.yaml                      FE ↔ BE HTTP (REST + SSE, OpenAPI 3.2)
└── backend-ai/
    ├── openapi.yml                       BE ↔ AI HTTP API와 request/response schema
    ├── sqs/
    │   ├── README.md                     BE·AI SQS 애플리케이션 처리 가이드
    │   └── messaging.yml                 BE ↔ AI SQS 메시지 payload schema
    ├── sse-contract.yml                  BE ↔ AI SSE 공통 규칙
    └── sse/
        └── simple-agent-run-stream.yml   BE ↔ AI SSE endpoint
```

기존 `asyncapi.yaml`과 `schema/parse-*.schema.json`은 초기 비동기 계약을 위한 임시 파일이었으며
현재는 사용하지 않는다. BE↔AI HTTP API와 그 request/response schema는 `backend-ai/openapi.yml`이 소유한다.
SQS 채널 토폴로지와 전달 의미론(at-least-once, visibility timeout, DLQ)은 ADR-002가,
SQS 메시지 payload schema는 `backend-ai/sqs/messaging.yml`이 소유한다.
앱 개발자가 지켜야 할 소비·ACK·heartbeat·DLQ 처리 규칙은
`backend-ai/sqs/README.md`에서 두 SSOT를 구현 관점으로 연결한다.

## 규칙

- 계약이 코드보다 앞선다. 스키마를 바꿔야 하면 여기부터 PR로 고치고 코드가 따라간다.
- 참조할 계약이 없으면 코드에서 임의로 정의하지 말고 여기에 먼저 만든다.
- 한 사실은 한 곳에만 쓴다. 다른 문서는 링크만 건다.
- FE↔BE의 REST와 SSE는 `frontend-backend/openapi.yaml` 한 파일이 SSOT다.
- BE↔AI의 HTTP endpoint와 request/response payload는 `backend-ai/openapi.yml`이 SSOT다.
- BE↔AI의 SQS 메시지 payload는 `backend-ai/sqs/messaging.yml`이 SSOT다. wire field는 camelCase이며,
  송신 측·계약 검증 테스트는 strict(additionalProperties: false), 수신 측은 미지 필드를 관대하게 무시할 수 있다.
- SSE event와 JSON data payload는 OpenAPI 3.2의 `text/event-stream.itemSchema`로 정의한다.
- OpenAPI가 표현하지 못하는 event 순서·terminal 조건·upstream 매핑은 같은 operation의
  `x-sse-lifecycle`, `x-upstream-event-mapping` 등 specification extension에 둔다.
- OpenAPI 3.2 문서를 시각화할 때는 3.2를 지원하는 Swagger UI 5.32.0 이상을 사용한다.

## 버전

`0.x` 동안은 breaking change를 minor로 올린다 (0.1.0 → 0.2.0). 호환되는 필드 추가·설명 보완은 patch (0.1.1).
첫 안정 계약이 나오면 1.0.0으로 올린다.

## 소유권

- `PaperStatus`의 단일 writer는 BE다. 파싱 서버는 결과만 발행하고 상태를 직접 쓰지 않는다. (ADR-001)
- BE↔AI HTTP API의 request/response schema는 `backend-ai/openapi.yml`에서 관리한다.
- BE↔AI SQS 메시지 payload schema는 `backend-ai/sqs/messaging.yml`에서 관리한다.
  채널 토폴로지·전달 의미론은 ADR-002가 소유한다.
- 메시지의 `fileKey`는 불투명한 S3 object key다. 키 형식은 BE 저장소 내부 구현이며 계약이 아니다.
  AI는 전달받은 값을 그대로 사용한다.

## 미확정

- **파싱 산출물 조회 계약** — `DocumentParseResponse`는 아직 Reader가 사용할 Markdown·미디어 구조를
  정의하지 않는다. FT-004 Reader 착수 전에 `backend-ai/openapi.yml`에서 확정한다.
- **`parse-results`의 `result` 본문** — FT-004 Reader 착수 전까지 미확정이며 BE는 해석하지 않는다.
  `backend-ai/sqs/messaging.yml`에서 확정한다.
