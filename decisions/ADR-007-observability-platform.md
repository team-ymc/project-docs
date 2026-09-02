# ADR-007: 관측 플랫폼은 Grafana Cloud로 시작하고 공개 표준으로 이전 가능성을 유지한다

## 1. Overview

- Date: 2026-08-31
- Status: Accepted
- Deciders: 근흐흐
- Tracking: [YMC-357](https://geunhh.atlassian.net/browse/YMC-357)
- Related: [`architecture/observability/`](../architecture/observability/README.md), [AWS Infrastructure](../architecture/aws-infrastructure.md)

## 2. Context

Paper Teacher는 AWS ECS on Fargate, ALB, RDS, SQS, Lambda를 사용하며 현재 AWS 리소스 metric과 애플리케이션 로그를 CloudWatch에 보낸다. `infra`에는 DLQ, ALB 5xx, unhealthy target, RDS CPU·storage, Lambda error, ECS running task를 감시하는 CloudWatch Alarm과 환경별 SNS topic이 이미 있다. 다만 팀이 일상적으로 사용할 통합 대시보드, 애플리케이션 metric 수집 경로, 로그 조회 화면, Slack 전달 경로는 완성되지 않았다.

Parser와 AI 서버는 GPU 자원과 비용 요구에 따라 향후 Runpod이나 온프레미스 GPU 서버 등 AWS 외부 환경에서 운영할 수 있다. 따라서 애플리케이션 metric의 저장·조회 경로가 CloudWatch에 종속되면 실행 환경에 따라 관측 체계를 별도로 운영해야 한다. 현재 팀 규모에서는 관측 플랫폼을 직접 운영하는 부담도 피해야 한다.

## 3. Decision

대시보드는 Grafana Cloud, 애플리케이션 metric 저장소는 Grafana Cloud Mimir로 시작한다. 현재 예상 사용량을 Free tier로 수용할 수 있고, AWS 외부 workload도 같은 저장·조회 경로를 사용할 수 있다는 점을 핵심 근거로 삼는다.

### 저장소 분리 기준

AWS 관리형 리소스 metric과 ECS 로그는 AWS가 CloudWatch에 기본 게시하므로 CloudWatch에 유지한다. 이를 Mimir나 Loki로 다시 전송하지 않아 중복 수집·저장 비용과 운영 복잡도를 피한다.

애플리케이션 metric은 Spring Backend뿐 아니라 향후 AWS 외부에서 실행될 Parser와 AI 서버도 같은 저장·조회 경로를 사용할 수 있도록 Grafana Cloud Mimir에 저장한다. 실행 환경에 따라 Collector 배치는 달라질 수 있지만 수집 규격, 저장소와 조회 방식은 동일하게 유지한다. 저장소는 신호의 생성 위치와 용도에 따라 분리하되 조회 화면은 Grafana로 통합한다.

### AWS 인프라 metric과 로그

- ECS, ALB, RDS, SQS, Lambda 등 AWS 관리형 리소스 metric은 CloudWatch Metrics에 유지한다.
- ECS 애플리케이션 로그는 JSON 구조로 stdout/stderr에 기록하고 ECS `awslogs` log driver를 통해 CloudWatch Logs에 유지한다.
- Grafana Cloud는 CloudWatch의 기본 metric과 로그를 원격 조회한다.
- AWS 인프라의 즉시 알림은 기존 `CloudWatch Alarm → SNS` 경로를 유지하고 Slack을 최종 전달 채널로 사용한다.

### 애플리케이션 metric

- 애플리케이션 metric의 기본 저장소는 Grafana Cloud Mimir로 하고 Grafana에서 PromQL로 조회한다.
- 수집과 전송에는 Prometheus exposition, OpenTelemetry Collector와 OTLP 같은 공개 표준을 사용한다.
- AWS Application Auto Scaling처럼 AWS가 직접 읽어야 하는 집계 metric만 목적과 비용을 명시한 뒤 CloudWatch에 선택적으로 중복 게시한다.

구체적인 수집·전송 경로는 [Telemetry Pipelines](../architecture/observability/telemetry-pipelines.md)에서 관리한다.

### Collector 배치

Spring Backend의 초기 수집기는 Fargate Task마다 sidecar로 배치한다. 애플리케이션 API는 8080, 내부 management endpoint는 8081을 사용하며 Collector는 같은 Task의 loopback endpoint를 scrape한다. 따라서 별도의 ECS task discovery 없이 각 replica를 수집할 수 있지만 애플리케이션과 Task 자원을 공유하고 Task 수만큼 Collector가 늘어난다.

FastAPI와 Parser도 ECS에서 운영되는 동안 같은 Task sidecar 배치 방식을 사용한다. 각 서비스의 metric 계측과 노출 방식은 Backend pipeline 검증 후 별도로 결정한다.

Collector 자원 비용이 의미 있게 커지거나 중앙 처리 기능이 필요해지면 독립 Collector ECS service를 다시 비교한다.

### 환경 경계

dev와 prod는 하나의 Grafana Cloud stack을 사용한다. Mimir 전송 token, CloudWatch data source와 AWS read role은 환경별로 분리하며, AWS read role은 공통 Terraform module로 정의해 각 환경에서 생성한다.

## 4. Options Considered

### Grafana Cloud + Mimir — 채택

- 결정 시점의 Free tier는 최대 3 active users, 10,000 active series와 14일 보관을 제공해 현재 예상 사용량을 수용한다.
- AWS 밖의 실행 환경이 추가되거나 AWS를 떠나더라도 같은 애플리케이션 metric pipeline을 유지할 수 있다.
- Free tier 한도나 보관 기간이 부족해지면 유료 플랜과 다른 저장소를 다시 비교한다.

### Grafana Cloud + CloudWatch OTel Metrics

- OTLP 수집, PromQL 조회와 15개월 보관을 제공하고 AWS 밖에서도 전송할 수 있다.
- AWS를 떠난 뒤에도 metric 저장·인증·조회가 CloudWatch에 의존하므로 채택하지 않는다.

용어:

| 용어 | 의미 |
|---|---|
| OpenTelemetry (`OTel`) | metric·로그·trace를 같은 모델로 계측하고 처리하기 위한 공개 표준과 도구 모음 |
| OpenTelemetry Protocol (`OTLP`) | OTel 데이터를 Collector에서 저장소로 전송할 때 사용하는 통신 규격 |
| PromQL | Prometheus 호환 metric 저장소의 데이터를 조회하고 계산하는 언어 |
| series | 하나의 metric 이름과 전체 label 값 조합으로 식별되는 시계열 |

### Amazon Managed Grafana + AWS 저장소

- 최대 5명의 90일 체험 이후 active user 비용이 발생하고 metric 저장소 비용도 별도다.
- AWS 권한 연동은 편하지만 AWS 중심 구조가 되므로 채택하지 않는다.

### Grafana stack 직접 호스팅

- 데이터와 보관 정책을 직접 통제할 수 있지만 현재 팀에는 운영 부담이 크다.
- Grafana Cloud 비용이나 보안 요구가 직접 운영 비용보다 커질 때 재검토한다.

## 5. Consequences

- 팀은 CloudWatch console 대신 Grafana를 기본 조회 화면으로 사용한다.
- AWS 인프라 metric·로그는 CloudWatch에, 애플리케이션 metric은 Mimir에 저장하므로 두 서비스의 연동과 권한을 함께 관리한다.
- Grafana Cloud Free tier 한도와 CloudWatch 조회 비용을 관찰하고, 현재 선택의 비용 이점이 사라지면 저장소를 재검토한다.

## 6. Follow-up Decisions

- FastAPI·Parser의 metric 계측과 노출 방식은 Backend pipeline 검증 후 담당자가 결정한다.
- Loki, error tracking, trace, SLO와 세부 alert 정책은 기본 pipeline 검증 후 별도로 결정한다.

## 7. References

- [Grafana Cloud documentation](https://grafana.com/docs/grafana-cloud/)
- [Grafana Cloud pricing](https://grafana.com/pricing/)
- [Grafana Cloud metrics cost calculation](https://grafana.com/docs/learning-paths/billing-usage/learn-cost-calculations/)
- [Grafana CloudWatch data source](https://grafana.com/docs/grafana/latest/datasources/aws-cloudwatch/)
- [OpenTelemetry Collector](https://opentelemetry.io/docs/collector/)
- [OpenTelemetry Prometheus/OpenMetrics compatibility](https://opentelemetry.io/docs/specs/otel/compatibility/prometheus_and_openmetrics/)
- [OpenTelemetry deployment attributes](https://opentelemetry.io/docs/specs/semconv/registry/attributes/deployment/)
- [OpenTelemetry ECS resource detector](https://github.com/open-telemetry/opentelemetry-collector-contrib/blob/main/processor/resourcedetectionprocessor/internal/aws/ecs/documentation.md)
- [Spring Boot Actuator Prometheus endpoint](https://docs.spring.io/spring-boot/reference/actuator/endpoints.html)
- [Amazon ECS application metrics with ADOT sidecar](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/metrics-data.html)
- [Amazon CloudWatch OTel Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/metrics-otel-overview.html)
- [Amazon CloudWatch PromQL limits](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-PromQL.html)
- [Amazon CloudWatch pricing](https://aws.amazon.com/cloudwatch/pricing/)
- [Amazon Managed Grafana pricing](https://aws.amazon.com/grafana/pricing/)
- [Amazon ECS container restart policies](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/container-restart-policy.html)
- [Grafana Mimir OpenTelemetry metric mapping](https://grafana.com/docs/mimir/latest/configure/configure-otel-collector/)
- [Amazon Q Developer in chat applications](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/q-in-chat-applications.html)
