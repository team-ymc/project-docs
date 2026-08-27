# ADR-006: 유한 사용량은 외부 실행 전에 예약하고 성공 시 확정한다

## 1. Overview

- Date: 2026-08-26
- Status: Accepted
- Deciders: 근흐흐
- Tracking: YMC-344 / `features/FT-011-플랜-사용량-제한.md` (Story 2·3)
- Implements: `contracts/frontend-backend/openapi.yaml` (`/api/me/plan`, 429 `*_USAGE_LIMIT_EXCEEDED`)
- Related: ADR-001 §5(정체 레코드 정리), ADR-004

## 2. Context

Free와 Pro의 유한한 사용량 정책은 AI 질의와 문서 등록의 성공 횟수를 제한한다. 실패한 실행은 사용 횟수로 확정하지 않되, 동시 요청으로 한도를 초과하지 않도록 외부 실행 전에 사용 가능 횟수를 확보해야 한다.

## 3. Decision

유한한 사용량 정책에서는 한도 판정과 1회 예약을 원자적으로 처리한다. 예약은 성공하면 확정하고, 시스템 실패 또는 정해진 실행 deadline 초과로 실패하면 해제한다. 같은 실행은 한 번만 예약ㆍ확정ㆍ해제한다.

| 사용량 | 예약 | 확정 | 해제 |
|---|---|---|---|
| AI 질의 | 동기 검증 후 외부 AI 호출 전 | 최종 답변 저장과 `ASSISTANT: GENERATING → COMPLETED` 전이가 commit됨 | AI 실행이 `FAILED`로 확정됨 |
| 문서 등록ㆍ파싱 | `POST /api/papers`의 동기 검증 후, `Paper(UPLOAD_PENDING)` 생성과 presigned URL 발급 전 | `Paper → COMPLETED` 전이가 commit됨 | 문서 처리가 `FAILED` 또는 `EXPIRED`로 확정됨 |

문서 등록은 예약과 `Paper(UPLOAD_PENDING)` 생성을 같은 트랜잭션으로 처리한다. 한도 초과 시 Paper와 presigned URL을 생성하지 않는다.

사용량은 기간별 `usage_bucket`과 실행별 `usage_record`로 저장하며 버킷에는 집계 카운터를 두지 않는다. 유한한 정책의 예약 트랜잭션은 버킷 행을 잠근 뒤 같은 버킷의 `RESERVED + CONFIRMED` 원장을 집계하고, 한도 미만일 때 `(usageType, sourceId)`가 유일한 `RESERVED` 원장을 추가한다. AI 질의의 `sourceId`는 `clientMessageId`, 문서 등록의 `sourceId`는 `paperId`를 사용한다.

실행 주체가 종료되어 terminal 판정이 남지 않은 경우는 주기적으로 도는 정체 레코드 정리가 회수한다. deadline이 지난 채팅 `GENERATING`은 `FAILED`로, 문서의 비종결 상태는 `EXPIRED` 또는 `FAILED`로 내리고 예약을 해제한다(ADR-001 §5).

구체적인 runtime timeout과 deadline은 YMC-347에서 정리하는 SSOT를 따른다. 사용자 취소에 따른 예약 해제는 MVP에서 다루지 않는다.

## 4. Options Considered

### 문서 등록 시 예약 — 채택

`POST /api/papers`에서 예약해 한도 초과 사용자가 파일을 업로드하기 전에 요청을 거절한다.

- 한도 판정과 예약, Paper row 생성을 한 트랜잭션으로 처리해 동시 등록이 한도를 넘지 않게 한다.
- 한도 초과 시 Paper와 S3 객체가 생기지 않아 별도의 사용자 복구 상태가 필요 없다.
- 업로드와 파싱이 끝날 때까지 예약이 유지되므로 terminal 상태가 남지 않은 실행을 정리해야 한다.

### 문서 업로드 완료 시 예약

`POST /api/papers/{paperId}/complete`에서 예약하면 업로드 후 한도 초과를 안내하고 별도의 대기ㆍ복구 상태를 제공해야 하므로 채택하지 않는다.

- 실제 파싱 직전에 예약하므로 예약 유지 시간은 짧다.
- 한도 초과 시점에는 Paper와 S3 객체가 이미 생성되어 있어, FE가 업로드 후 한도 초과로 실패했음을 별도로 안내해야 한다.
- 이 상태를 기존 파싱 `FAILED`와 동일하게 다룰지 별도 상태로 구분할지 결정하고, 재시도 흐름도 제공해야 한다.
- 파싱하지 않을 파일의 S3 업로드ㆍ임시 저장과 객체 보관ㆍ삭제에 대한 비용 및 정리 책임이 생긴다.

### 사용량 집계 — 원장 직접 집계 채택

MVP에서는 `usage_record`를 직접 집계한다. 버킷 카운터를 함께 유지하는 방식은 조회 비용을 줄이지만 원장과 카운터의 정합성을 추가로 관리해야 하므로 채택하지 않는다.

## 5. Consequences

- AI 호출과 문서 업로드ㆍ파싱은 예약 성공 후에만 시작한다.
- 동시 요청이 유한한 한도를 초과하지 않는다.
- 현재 사용량은 같은 버킷의 `RESERVED + CONFIRMED` 원장 수로 계산한다.
- 문서 업로드 또는 처리가 terminal 상태에 도달하지 못하면 예약을 회수하기 위한 deadline 기반 정리가 필요하다.

## 6. Updates

- **2026-08-27** — deadline 초과 정체를 종결 상태로 내리고 예약을 해제하는 **정체 레코드 정리를 MVP로 확정**했다
- **2026-08-28** — 정리 대상에 채팅 `GENERATING` 정체를 명시했다. BE가 실행 중 종료되면 실행을 감시하던 타이머도 함께 사라지므로, terminal 판정과 예약 해제를 정체 레코드 정리가 대신한다.
