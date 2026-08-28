# Runtime Timeout SSOT

요청 경로별 timeout/deadline 값과 대소 관계의 SSOT. 값을 바꾸면 이 문서를 먼저 고치고 코드·인프라가 따라간다.

- Tracking: YMC-347
- 조사 기준: 2026-08-27, 3개 저장소(app·ai·infra) 전수 확인. 값은 dev 실배포 기준
- Related: ADR-001 §5, ADR-002, ADR-004, ADR-006

## 1. 요청 경로

| 경로 | 구간 |
|---|---|
| P1 | FE → CloudFront → ALB → BE (HTTP/SSE) |
| P2 | BE → AI (Service Connect HTTP/SSE) |
| P3 | BE ↔ AI (SQS `parse-requests` / `parse-results`) |
| P4 | AI 워커 → PaddleOCR |
| P5 | FE → S3 (presigned PUT) |

## 2. 채팅 SSE (P1 + P2)

| 구간 | 설정 | 역할 | dev | 초과 시 |
|---|---|---|---|---|
| FE→CF | fetch timeout 없음 | — | 브라우저 기본 | — |
| CF→ALB | `origin_read_timeout` | 응답 데이터 사이 대기 상한 | 60s | 504 |
| CF→ALB | `origin_keepalive_timeout` | 응답 종료 후 커넥션 재사용 유지 | 기본값 5s | 커넥션 재수립 |
| CF↔ALB↔BE | `idle_timeout` (ALB) | 커넥션 데이터 없음 상한 | 120s | 절단 |
| BE→FE | `chat.stream.heartbeat-interval` | 침묵 시 연결 유지 이벤트 — 위 두 절단 방지 | 15s | — |
| AI→BE 수신 | `chat.stream.idle-timeout` | AI 이벤트 사이 침묵 감지 | 60s | `FAILED`(`AI_TIMEOUT`), 예약 해제 |
| run 전체 | `chat.stream.deadline` | 답변 생성 전체 시한 안전망 | 10m | `FAILED`(`AI_TIMEOUT`), 예약 해제 |
| BE 내부 | `chat.stream.emitter-timeout` | MVC async 요청 수명 — deadline의 뒷단 안전망 | 11m | 요청 종료 |
| BE→AI | CONNECT_TIMEOUT_MILLIS (하드코딩) | TCP 연결 수립(핸드셰이크) 상한 | 5s | 접속 실패 |
| BE 내부 | reactor-netty pendingAcquireTimeout | WebClient 커넥션 풀 대여 대기 상한 — 스트림이 커넥션을 점유하므로 동시 채팅 수가 풀 크기를 넘으면 발동 | 기본값 45s (풀 500) | 요청 실패 |
| BE↔AI 프록시 | Service Connect per-request / idle (하드코딩) | Envoy 요청 총 시간 / 데이터 없음 상한 | 무한 / 300s | P2 절단 |
| AI→모델 API | 에이전트별 `timeout` / `max_retries` | 모델 호출 1회 상한과 재시도 | 30~60s / 6회 | SDK 재시도 후 실패 |
| 모델 답변 | `max_tokens` | 답변 길이 상한 | 상한 없음 | BE `max-content-length` 65,536자가 유일한 상한 |

대소 관계:

- `idle(60s) < deadline(10m) < emitter(11m)` — BE 기동 시 검증
- `heartbeat(15s) < origin_read(60s) < ALB idle(120s)` — FE 방향 유지 조건. 검증 없음
- `BE idle(60s) ≤ Service Connect idle(300s)` — BE가 먼저 끊어야 FAILED 판정 주체가 BE다. 검증 없음
- LLM timeout(60s) = BE idle(60s) — 동값. LLM 재시도 중 침묵이 60s를 넘으면 BE가 먼저 끊는다 (§6-1)

## 3. 문서 파싱 (P3 + P4)

| 홉 | 설정 | 역할 | dev | prod(example) | local | 초과 시 |
|---|---|---|---|---|---|---|
| requests visibility | `job_visibility_timeout_seconds` | 워커 처리 중 재전달 방지 시간 | 180s | 900s | 900s | 재전달 |
| requests maxReceiveCount | `requests_max_receive_count` | DLQ 이동 전 수신 횟수 상한 | 1 | 5 | 3 | DLQ → dlq-handler가 `PARSE_RETRIES_EXHAUSTED` 발행 |
| 워커 heartbeat | `parser_heartbeat_interval_seconds` | 처리 중 visibility 연장 주기 | 60s | 300s | 앱 기본 300s | 연장 실패는 로깅만 (§6-2) |
| 워커 잡 deadline | `DOCUMENT_PARSER_DEADLINE_SECONDS` | 파싱 잡 전체 시한 안전망 | 3600s | 〃 | 〃 | `DeadlineExceeded`, 삭제 없이 재전달/DLQ |
| PaddleOCR 요청 | `paddleocr_request_timeout_seconds` | 외부 API 요청 1건 상한 | 600s | 120s | 120s | 재시도 없이 잡 실패 |
| PaddleOCR 폴링 간격 | `..._POLL_INTERVAL_SECONDS` | 결과 확인·진행 이벤트 주기 | 3s | 3s | 3s | — |
| results visibility | `result_visibility_timeout_seconds` | BE 처리 중 재전달 방지 시간 | 60s | 120s | 기본값 30s | 재전달 |
| results maxReceiveCount | `results_max_receive_count` | DLQ 이동 전 수신 횟수 상한 | 3 | 5 | 5 | DLQ. 결과 유실, 레코드는 `PROCESSING` 정체 |
| BE 리스너 visibility | 명시 없음 | 〃 (큐 속성에 위임) | 큐 값 | 〃 | 〃 | 〃 |
| DLQ visibility | lambda 60×6+5 | 람다 재처리 간격 | 365s | 365s | 기본값 30s | 람다 재시도 간격 |
| retention | 본 큐 / DLQ (하드코딩) | 미소비 메시지 보관 기간 | 4일 / 14일 | 〃 | 〃 | 메시지 소멸 |

- dev requests maxReceiveCount=1은 실패 원인 분석을 위한 의도된 값이다. 재시도 정책은 prod 기준(5)으로 본다.
- PaddleOCR 600s는 평균 처리 ~3분의 여유분.
- 워커 deadline 3600s와 wait time은 infra가 env로 주입하지 않아 앱 기본값으로 동작한다 (§6-3).
- 개별 HTTP timeout은 `min(설정값, 잔여 deadline)`으로 줄어든다.

대소 관계:

- `heartbeat × 3 ≤ requests visibility` — infra 검증. dev는 60×3=180으로 경계값
- `1 ≤ heartbeat < visibility` — 앱 검증

## 4. 업로드·조회·기타 (P5, P1 일반)

| 항목 | 설정 | 값 |
|---|---|---|
| presigned 만료 (PUT·GET 공용) | `aws.s3.presign-expiry` | 10m |
| FE 업로드 XHR timeout | 없음 | 브라우저 기본 |
| FE 상태 폴링 | `refetchInterval` | 2s, non-terminal 존재 시만 |
| JWT access / refresh | `auth.access-ttl` / `refresh-ttl` | 30m / 14d |
| Tomcat / Hikari / 트랜잭션 / RDS statement_timeout | 명시 없음 | 프레임워크 기본 (§6-8) |
| ECS stopTimeout / TG deregistration | | 120s / 120s |

- presigned 만료는 요청 시작 시점에만 검증된다. 진행 중인 업로드는 끊지 않으며, 실패 후 같은 URL로 재시도할 수 있는 창의 상한이다.
- 10m 유지 결정(2026-08-28). PUT·GET 분리와 단축은 필요해지면 다시 본다. GET 단축은 본문 이미지 lazy 로딩의 만료 복구 빈도와 엮인다.

## 5. 사용량 예약과의 연결 (ADR-006)

예약 해제는 terminal 판정에 걸려 있고, terminal 판정은 이 문서의 deadline들이 만든다.

| 실행 | 서버가 살아서 실패를 관측 | 서버가 죽어 정체 — 정체 레코드 정리 |
|---|---|---|
| AI 질의 | idle 60s·deadline 10m → `FAILED`, 해제 | deadline 지난 `GENERATING` → `FAILED`, 해제 |
| 문서 등록 | 파싱 `FAILED` 수신, 해제 | `UPLOAD_PENDING → EXPIRED`(S3 객체 유무 무관), `UPLOADED`·`PROCESSING → FAILED`, 해제 |

- 정리의 스캔 deadline은 채팅 `GENERATING` 30m, `UPLOAD_PENDING` 1h, `UPLOADED`·`PROCESSING` 3h. 실행 주기는 10m (2026-08-28, YMC-351). 채팅은 deadline(10m)+emitter(11m)보다, 문서는 presigned 만료(10m)와 워커 deadline(3600s)보다 커야 정상 실행을 오판하지 않는다 — 스캔이 짧으면 진행 중인 정상 실행을 종결시키고, 길면 죽은 예약의 반환만 늦어진다. 3h는 dev(requests maxReceiveCount 1) 기준이다 — prod 전환 시 SQS 재시도 총 수명(maxReceiveCount × 워커 deadline) 기준으로 재계산한다.
- 정리는 종결·해제만 한다. 복구는 post-MVP (ADR-001 §5).
- requests DLQ 경로는 dlq-handler가 `PARSE_RETRIES_EXHAUSTED`를 발행해 `FAILED` 전이로 해제된다. 실발동 검증은 YMC-305. results DLQ 유실은 정리가 회수한다.

## 6. 미해결

조사에서 발견됐지만 아직 처리가 결정되지 않은 항목. 수정은 별도 티켓으로 하고, 착수하면 항목에 티켓 키를 단다. 처리되면 본문 값을 갱신한 뒤 항목을 지우고, 의도로 확정되거나 유지로 결정되면 해당 절의 근거로 옮긴다.

1. LLM timeout(60s) = BE idle(60s) 동값. 재시도 6회가 무의미해질 수 있다
2. 워커 heartbeat 연장 실패 시 로깅만 하고 계속 진행 — 재전달 후 이중 처리 창
3. 워커 deadline·wait time infra 미주입
4. AI 채팅 SSE 3종에 heartbeat와 `X-Accel-Buffering: no` 없음. 파싱 스트림에만 있다
5. 동기 파서 API는 deadline 없이 무한 폴링. 워커 경로만 `ParseDeadline` 보호
6. FE에 fetch·XHR timeout 없음. 만료 URL 재시도의 403을 만료로 구분하지 못한다
7. BE SQS 리스너 visibility 명시 없음 — 자동설정이 물러나 큐 기본값 의존
8. Hikari·트랜잭션·RDS statement_timeout 전부 기본값 — 무제한 쿼리 가능
9. local·dev·prod 큐 설정 불일치. 로컬은 인프라 절단(60s·120s·300s) 재현 불가
10. dev tfvars에 `alb_idle_timeout_seconds` 누락 — default 120과 우연히 일치
11. 채팅 스트림 총 시간을 막는 것은 BE deadline(10m)뿐 — `max_tokens` 무제한, per-request 무한
12. 실행 상한(채팅 deadline 10m, 워커 deadline 3600s)이 실측 없이 잡은 값 — 1차 배포 전 평균 실행 시간을 실측해 상한을 줄이고, §5의 스캔 deadline도 따라 줄인다
