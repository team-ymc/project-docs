# Observability

Paper Teacher의 metric, 로그, trace와 alert가 생성되어 수집·저장·조회·전달되는 구조를 관리한다.

- Decision: [ADR-007](../../decisions/ADR-007-observability-platform.md) (`Accepted`)

## Documents

| 문서 | 역할 |
|---|---|
| [Telemetry Pipelines](telemetry-pipelines.md) | metric과 로그의 생성·수집·전송·저장·조회 책임 |

내용이 실제로 커질 때만 conventions, alerting, SLO와 dashboard 문서를 분리한다. 이 README에 상세 규칙이나 상태를 복제하지 않는다.

## Principles

- 관측 장애가 사용자 요청의 성공 여부를 결정하지 않는다.
- dev와 prod는 같은 pipeline과 규칙 구조를 사용하고 환경별 threshold와 notification 강도만 조정한다.
- OpenTelemetry, OTLP, Prometheus exposition처럼 backend와 독립적인 규격을 우선한다.
- metric label의 cardinality를 제한하고 요청 식별자는 로그와 trace에 둔다.
- 민감정보, 인증정보, PDF 본문, 사용자 질문과 AI 답변 본문을 기본 telemetry에 넣지 않는다.
- dashboard, alert rule과 collector 설정은 가능한 한 코드로 관리한다.
- custom metric과 SLO를 만들기 전에 기본 pipeline을 end-to-end로 검증한다.

## Scope

이 설계의 범위는 AWS의 dev/prod 환경과 Spring Backend다. Runpod 같은 외부 AI 실행 환경, Loki 전송, error tracking 제품, FastAPI·Parser metric 구현, trace sampling과 구체적인 SLO는 별도 결정으로 다룬다.

구현별 실행 절차와 장애 복구 runbook은 코드와 가까운 `infra/docs/runbooks/`에 두고 여기에서는 시스템 책임과 계약만 관리한다.
