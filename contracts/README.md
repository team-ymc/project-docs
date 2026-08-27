# Contracts

FE·BE·AI 사이 계약의 SSOT 디렉토리.

## Contract Map

| 통신 경계 | Source of Truth | 용도 |
|---|---|---|
| FE ↔ BE HTTP | [OpenAPI](frontend-backend/openapi.yaml) | REST와 SSE path, request와 response schema |
| BE ↔ AI | [Backend ↔ AI Contracts](backend-ai/README.md) | HTTP, SQS와 SSE 계약 진입점 |
| BE ↔ AI HTTP | [OpenAPI](backend-ai/openapi.yml) | 일반 HTTP와 SSE path/request schema |
| BE ↔ AI SQS | [SQS 처리 가이드](backend-ai/sqs/README.md), [Message Schema](backend-ai/sqs/messaging.yml) | 비동기 요청·결과 payload와 처리 책임 |
| BE ↔ AI SSE | [SSE 공통 계약](backend-ai/sse-contract.yml), [Endpoint Contracts](backend-ai/sse/README.md) | 공통 SSE 규칙과 endpoint별 event 계약 |

## Usage

- FE ↔ BE는 `frontend-backend/openapi.yaml`을 읽는다.
- BE ↔ AI는 `backend-ai/README.md`에서 전송 방식을 선택한다.
- SSE는 OpenAPI와 endpoint별 SSE 계약을 함께 읽는다.
- SQS는 Message Schema와 처리 가이드를 함께 읽는다.

계약이 코드보다 앞선다. 참조할 계약이 없으면 코드에서 정의하지 않고 여기에 먼저 만든다.

## Related Documentation

- 기능 요구사항: [Features](../features/README.md)
- 시스템 구성과 통신 경로: [Architecture](../architecture/README.md)
- 설계 결정과 근거: [Decision Records](../decisions/README.md)
