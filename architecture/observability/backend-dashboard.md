# Backend Dashboard

Grafana Backend dashboard가 무엇을 왜 보는지의 SSOT다. 패널 쿼리의 원본은 infra `deploy/grafana/dashboards/backend.json.tftpl`이다.

- Decision: [ADR-007](../../decisions/ADR-007-observability-platform.md) (`Accepted`)
- Pipeline: [Telemetry Pipelines](telemetry-pipelines.md)

## 1. 목적

dashboard는 분석의 도구다. 패널은 두 가지 이유로만 추가한다.

- 장애 때 첫 5분 안에 "어느 흐름이 왜 깨졌나"를 답하는 데 필요하다.
- 계획된 실험(용량 테스트, 확장 검증)의 근거 화면이 된다.

둘 다 아니면 만들지 않는다. 실험용 패널은 실험을 실행할 때 그 분석과 함께 추가한다.

## 2. 지금 구성

| 행 | 패널 | 소스 | 쿼리 요지 | 목적 |
|---|---|---|---|---|
| 0 상태 | 인스턴스 수 | Mimir | `count(up == 1)` | 0이면 다른 패널을 볼 이유가 없음 |
| 0 상태 | 5xx / min | Mimir | 알림과 동일 식 | 알림 착지 값 |
| 0 상태 | 채팅 실패율 5m | Mimir | `outcome!="success"` 종료 수 / 전체 종료 수 | HTTP 200이어도 SSE는 실패함 |
| 0 상태 | 진행 중 채팅 | Mimir | `sum(chat_active_runs)` | 실시간 동시성 |
| 0 상태 | 파싱 DLQ 합 | CloudWatch | DLQ 4개 `ApproximateNumberOfMessagesVisible` 합 | 0이 아니면 논문 영구 실패 |
| 1 채팅 | 결과별 종료율 | Mimir | `sum by (outcome) rate(chat_runs_total)` | error는 AI 오류, timeout은 침묵 60초 |
| 1 채팅 | 진행 중 실행 | Mimir | `chat_active_runs` 인스턴스별 | 한 Task 쏠림 감지 |
| 1 채팅 | TTFT p50·p95 | Mimir | `histogram_quantile` over `chat_ttft_seconds_bucket` | 답이 나오기 시작하는 시간 |
| 1 채팅 | 성공 처리시간 p95 | Mimir | `chat_duration_seconds_bucket{outcome="success"}` | timeout을 빼야 p95가 60초에 고정되지 않음 |
| 2 HTTP | 요청률·응답시간 | Mimir | 기존 패널, SSE 경로 제외 | 스트림 지속시간이 섞이면 값이 무의미 |
| 3 자원 | JVM 힙 % | Mimir | 인스턴스별 used/max, `area="heap"` | 기존 sum/sum 식 교체 |
| 3 자원 | Hikari 풀 | Mimir | `active/max`, `pending` | pending > 0이면 DB 커넥션 포화 |
| 4 로그 | Backend ERROR 로그 | CloudWatch Logs | 기존 패널 유지 | 시간 범위·log group 제한 |

DLQ 4개는 `parse-requests`, `parse-results`, `knowledge-compile-requests`, `knowledge-compile-results`의 DLQ다. SSE 경로는 채팅 메시지와 인라인 번역이다.

## 3. 실험과 함께 추가할 패널

| 실험 | 읽을 패널 | 추가 시점 |
|---|---|---|
| 채팅 단일 Task 용량: 동시 실행 증가 시 TTFT가 꺾이는 지점과 먼저 포화되는 자원 | 위 구성 + 프로세스 CPU, GC 정지 비율, JVM 스레드 상태, SSE 연결 수(계측 전에는 ALB `ActiveConnectionCount`로 대체) | 채팅 SSE 용량 테스트(재설계 예정) 실행 시 |
| 확장 동작: scale-out이 트래픽을 받기까지의 시간, scale-in의 SSE 절단 여부 | 인스턴스 수, 인스턴스별 진행 중 실행, outcome error | prod 오토스케일링 검증 시 |
| 파싱 backlog: 워커가 큐를 따라가는지, backlog 오토스케일링 발동 시점 | SQS 최고령 메시지 나이, 대기·처리 중 수, 발행·처리 처리량, `RunningTaskCount` | [document-parser 용량 테스트](../../testing/capacity/document-parser/plan.md) 실행 시 |

조회 API 성능 분석은 계획된 실험이 없어 패널을 만들지 않는다.

## 4. 구성 원칙

- 환경별 한 장. dev·prod가 같은 JSON template을 쓰고 환경 값만 다르다.
- 위에서 아래로 증상 → 원인. 행 0은 stat 타일만, 추세는 행 1 이하.
- 알림이 판정하는 값은 알림과 같은 쿼리를 쓴다. 화면 값과 Slack 값이 일치해야 한다.
- 카운터·비율은 `sum`, 히스토그램은 `sum by (le)` 뒤 `histogram_quantile`, 사용률 게이지는 인스턴스별 선. 게이지를 합산하면 가장 나쁜 인스턴스가 가려진다.
- 변수는 `instance` 하나. `up` 시계열에서 뽑고 기본값 All, Prometheus 패널에만 적용한다.
- rate 창은 `$__rate_interval`. scrape 주기 60초에서 고정 `[1m]`은 표본이 1개라 값이 나오지 않는다. 높은 해상도가 필요하면 Collector의 `scrape_interval`을 낮춘다.
- CloudWatch 패널은 period 60초, dashboard refresh 1분. AWS 자원 상세는 AWS dashboard에 두고 링크한다.

## 5. Threshold

- 5xx 타일은 알림과 같은 값(1분 3건)을 쓴다.
- 나머지는 배포 후 1주 baseline을 보고 환경별로 정한다. 그 전까지는 색 없이 값만 표시한다.
- 실패율 타일의 0/0은 "데이터 없음"으로 둔다.

## 6. 후속 계측

기존 메트릭으로 볼 수 없어 app 저장소 계측이 필요한 항목이다. 별도 PR로 다룬다.

| 항목 | 필요한 변경 |
|---|---|
| HTTP 요청 p95 | `management.metrics.distribution` histogram 설정 |
| Tomcat 스레드 풀 | `server.tomcat.mbeanregistry.enabled` |
| BE→AI 호출 outcome | WebClient를 Spring 빌더 주입으로 전환 |
| 파싱 lead time | 요청 발행 시각을 메시지에 실어 Timer 기록 |
| 동시 실행 상한 거절 | `chat_runs`에 rejected outcome |
| SSE 연결 수 | 채팅·인라인 번역 공통 Gauge. 실행 수와 달리 FE가 떠나면 즉시 감소 |

prod dashboard는 prod의 Grafana CloudWatch data source와 read role이 준비된 뒤 Grafana Terraform `environments`에 prod를 추가해 같은 template으로 생성한다.
