# Backend ↔ AI SSE endpoint 계약

이 디렉토리의 파일 하나는 SSE endpoint 하나의 request, event 순서, payload와 Backend 처리
규칙을 정의한다. 공통 framing과 종료 규칙은 [`../sse-contract.yml`](../sse-contract.yml),
HTTP request schema는 [`../openapi.yml`](../openapi.yml)을 따른다.

| Endpoint ID | Path | 성공 terminal | 실패 terminal |
|---|---|---|---|
| [`simple-agent-run-stream`](simple-agent-run-stream.yml) | `/api/v1/agents/simple-agent/runs/stream` | `run.completed` | `run.failed` |
| [`base-pdf-agent-run-stream`](base-pdf-agent-run-stream.yml) | `/api/v1/agents/base-pdf-agent/runs/stream` | `run.completed` | `run.failed` |
| [`inline-pdf-agent-run-stream`](inline-pdf-agent-run-stream.yml) | `/api/v1/agents/inline-pdf-agent/runs/stream` | `run.completed` | `run.failed` |
| [`universal-pdf-agent-run-stream`](universal-pdf-agent-run-stream.yml) | `/api/v1/agents/universal-pdf-agent/runs/stream` | `run.completed` | `run.failed` |
| [`inline-translate-agent-run-stream`](inline-translate-agent-run-stream.yml) | `/api/v1/agents/inline-translate-agent/runs/stream` | `run.completed` | `run.failed` |
| [`document-parser-parse-stream`](document-parser-parse-stream.yml) | `/api/v1/document-parser/parse/stream` | `parse.completed` | `parse.failed` |

Backend는 terminal event를 받기 전 연결이 끊기면 성공으로 확정하지 않는다. `message.delta`가
있어도 최종 값은 `message.completed.message`를 우선하고, parser 진행률은
`parse.completed`가 오기 전까지 완료로 보지 않는다.
