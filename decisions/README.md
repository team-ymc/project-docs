# Decision Records (ADR)

설계 결정을 기록하는 ADR 로그. 새 결정은 `ADR-XXX-*.md`로 추가하고 아래 표에 한 줄 등록한다.

| ADR | 제목 | Status | Date |
|---|---|---|---|
| [ADR-001](ADR-001-pdf-upload-presigned-url.md) | PDF 업로드는 presigned URL 기반 FE → S3 직접 업로드 | Accepted | 2026-07-03 |
| [ADR-002](ADR-002-be-ai-messaging-sqs.md) | BE ↔ AI 비동기 배선은 SQS로 한다 | Accepted | 2026-07-15 |
| [ADR-003](ADR-003-paper-id-and-duplicate-check.md) | 파일 콘텐츠 기반 중복 제거와 기본 파싱 결과 공유 | Proposed | 2026-07-21 |
| [ADR-004](ADR-004-chat-streaming-be-relay.md) | 채팅 스트리밍은 BE 경유 SSE로 하고, 대화 원본은 BE에서 관리 | Proposed | 2026-07-22 |
| [ADR-005](ADR-005-paper-upload-presigned-put-vs-post.md) | 문서 업로드 크기 제한은 Presigned PUT의 서명된 Content-Length로 강제 | Accepted | 2026-08-10 |
| [ADR-006](ADR-006-usage-reservation.md) | 유한 사용량은 외부 실행 전에 예약하고 성공 시 확정 | Accepted | 2026-08-26 |
| [ADR-007](ADR-007-observability-platform.md) | 관측 플랫폼은 Grafana Cloud로 시작하고 공개 표준으로 이전 가능성을 유지 | Accepted | 2026-08-31 |

Status: `Proposed`(합의 전) · `Accepted`(확정) · `Superseded`(다른 ADR로 대체) · `Deprecated`(폐기)
