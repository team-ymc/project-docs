# Contracts

FE·BE·AI 사이 계약의 SSOT 디렉토리.

```
contracts/
├── README.md
├── frontend-backend/
│   └── openapi.yaml                      FE ↔ BE HTTP (REST + SSE, OpenAPI 3.2)
└── backend-ai/
    ├── openapi.yml                       BE ↔ AI HTTP API와 request/response schema
    ├── sqs/
    │   ├── README.md                     BE·AI SQS 애플리케이션 처리 가이드 (소비·ACK·DLQ)
    │   └── messaging.yml                 BE ↔ AI SQS 메시지 payload schema
    ├── sse-contract.yml                  BE ↔ AI SSE 공통 규칙
    └── sse/
        └── simple-agent-run-stream.yml   BE ↔ AI SSE endpoint
```

계약이 코드보다 앞선다 — 스키마를 바꿔야 하면 여기부터 PR로 고치고 코드가 따라간다.
참조할 계약이 없으면 코드에서 임의로 정의하지 말고 여기에 먼저 만든다.
