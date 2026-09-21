# ADR-010: Parser Worker & SQS Scale in/out 전략
## 1. Overview

- Date: 2026-09-20
- Status: Proposed
- Deciders: 근흐흐
- Tracking: [YMC-398](https://geunhh.atlassian.net/browse/YMC-398)
- Related: [FT-003 논문 등록·분석](../features/FT-003-논문-등록-분석.md), [ADR-002 BE ↔ AI 비동기 배선](ADR-002-be-ai-messaging-sqs.md), [AWS Infrastructure](../architecture/aws-infrastructure.md), [문서 파서 용량 테스트](../testing/capacity/document-parser/plan.md)

## 2. Context

Parser Worker는 SQS `parse-requests`를 long polling하고 문서를 비동기로 파싱한다. CPU/Memory 사용률은
외부 PaddleOCR 응답 대기와 로컬 처리 시간을 구분하지 못하므로 대기 중인 작업량을 직접 나타내는
큐 지표가 확장 기준에 더 적합하다.

초기 사용자 성능 목표는 정상적으로 성공한 신규 문서의 95%가 Backend의 업로드 완료 요청 수락부터
`Paper.COMPLETED` commit까지 10분 이내에 끝나는 것이다. 현재 측정된 단일 문서 파싱 실행 시간
p95는 약 4분이다. 표본 수와 입력 분포는 추가 확인이 필요하며, PaddleOCR 서버 지연으로 P99는 약 30분이며, 응답 거절도 관측됐다.

Parser Worker 한 Task는 문서를 최대 4개 병렬 처리한다. 10분 end-to-end 목표에서 실행 시간
p95 4분을 제외하면 큐 대기에 사용할 수 있는 시간 예산은 6분이다.

```text
허용 큐 대기 시간 = 10분 - 4분 = 6분

Task당 처리율 = concurrency 4개 / 문서당 4분
                = 분당 약 1개

target backlog per task = 허용 큐 대기 6분 × 분당 1개
                         = 6개
```

이 계산은 p95 실행 시간을 일정한 처리율로 환산한 근사다. CloudWatch 수집과 평가, Fargate Task
기동, PaddleOCR tail latency가 추가되므로 계산 자체가 10분 목표를 보장하지 않는다. DEV 용량
테스트에서 end-to-end 지연과 성공률을 함께 검증해야 한다.

Scale-out은 backlog가 쌓일 때 빠르게 처리 용량을 확보해야 하지만, scale-in은 처리 중인 Task를
종료하지 않도록 보수적이어야 한다. 현재 AI 코드에 ECS task scale-in protection을 추가하면
애플리케이션과 인프라 양쪽의 변경이 필요하므로 이번 범위에서는 인프라 설정만으로 동작하는 전략을
선택한다.
부하 많을 것이라고 생각하지 않고, 중복된 논문의 경우 1회만 파싱 큐에 올라가기에 결정한다.

## 3. Decision

### 사용자 성능 목표

- 성공한 신규 문서 파싱의 end-to-end duration p95 10분을 초기 엔지니어링 목표로 둔다.
- 시작은 `POST /api/papers/{paperId}/complete`가 성공하고 신규 파싱 요청이 수락된 시점이다.
- 종료는 파싱 결과를 반영해 `Paper`가 `COMPLETED`로 commit된 시점이다.
- 지연 목표와 별도로 전체 성공률과 실패 원인을 측정한다. 성공률 목표는 측정 후 확정한다.
- DEV 검증 후 제품 요구사항으로 확정할 때 FT-003 Acceptance Criteria에 반영한다.

### Scale-out

Application Auto Scaling의 Target Tracking과 CloudWatch metric math를 사용한다.

```text
backlog per task =
  AWS/SQS ApproximateNumberOfMessagesVisible (Sum)
  / ECS/ContainerInsights RunningTaskCount (Average)
```

- target backlog per task: 6
- 최소 Task: 1
- 최대 Task: 4
- scale-out cooldown: 60초
- Target Tracking의 scale-in: 비활성화
- `RunningTaskCount` 제공을 위해 ECS Container Insights를 활성화

Target Tracking은 현재 Task 수에 고정 증분을 더하지 않고, metric을 목표값에 가깝게 만들 수 있도록
필요 capacity를 비례 계산한다. 단순화하면 원하는 Task 수는 `ceil(Visible / 6)`에 가깝고 최대 4로
제한된다.

```text
Visible 7개  → desired 약 2개
Visible 18개 → desired 약 3개
Visible 21개 이상 → desired 약 4개
```

따라서 backlog가 크게 증가하면 `1 → 3`, `1 → 4`처럼 여러 Task를 한 번에 추가할 수 있다.
Scale-out cooldown 중에도 이전 확장보다 큰 capacity가 필요하다고 계산되면 이미 추가한 capacity를
포함해 차이만 즉시 추가할 수 있다.

### Scale-in

Target Tracking의 자동 scale-in은 사용하지 않고 CloudWatch alarm과 Step Scaling을 별도로 둔다.

다음 세 지표의 합이 0인 1분 datapoint가 N회 연속 관측될 때 queue idle로 판단한다.

- `ApproximateNumberOfMessagesVisible`
- `ApproximateNumberOfMessagesNotVisible`
- `ApproximateNumberOfMessagesDelayed`

queue idle alarm이 유지되는 동안 한 번에 Task 하나만 줄이고 최소 1개를 유지한다.

| 환경 | Queue idle N | Scale-in adjustment | Scale-in cooldown |
|---|---:|---:|---:|
| DEV | 3분 | -1 Task | 5분 |
| PROD | 10분 | -1 Task | 5분 |

DEV에서는 대기 중·처리 중·지연 메시지가 모두 없는 상태가 3분 연속 확인되면 Task를 하나 줄인다. CloudWatch 수집과 평가에 시간이 걸리므로 실제 축소는 3분보다 조금 늦게 발생할 수 있다. 이후 5분 cooldown은 다음 축소까지의 최소 간격이며, 실제 간격은 경보 재평가와 ECS 반영 시간만큼 더 길 수 있다.

### 처리 중 Task 보호

이번 결정에서는 AI 애플리케이션의 ECS task scale-in protection을 추가하지 않는다.

- Visible만 0인 상태는 처리 중 작업이 남아 있을 수 있으므로 scale-in 조건으로 쓰지 않는다.
- NotVisible과 Delayed까지 모두 0인 상태가 지속될 때만 scale-in한다.
- CloudWatch 지표의 근사성과 수집 지연 때문에 새 메시지 유입과 scale-in이 경합할 가능성은 남는다.
- 예외적으로 처리 중 Task가 종료되면 SQS visibility timeout과 redrive가 재전달을 담당한다.
- 중복 전달이 발생해도 최종 상태와 산출물이 일관되도록 기존 멱등 처리를 전제로 한다.

## 4. Options Considered

### Target Tracking scale-out + queue-idle Step Scaling scale-in — 채택

- backlog 크기에 비례해 여러 Task를 빠르게 추가할 수 있다.
- scale-out과 scale-in을 비대칭으로 운영해 응답 지연과 종료 안전성을 각각 다룬다.
- Lambda나 custom metric publisher 없이 AWS 기본 metric math로 구성할 수 있다.
- AI 코드 변경 없이 Terraform 범위에서 시작할 수 있다.

### 애플리케이션 task scale-in protection - 후속

- 각 Task가 자신에게 진행 중 작업이 있는지 가장 정확하게 판단할 수 있다.
- AI 코드, ECS 권한, protection TTL과 실패 복구를 함께 구현해야 해 이번 인프라 작업 범위를 넘는다.
- 인프라 지표만으로 안전성이 부족하다고 확인되면 후속으로 도입한다.


## 5. Consequences

- backlog가 클수록 scale-out은 `+1`보다 크게 발생할 수 있고, scale-in은 항상 `-1`씩 진행된다.
- 최대 동시 파싱 수는 `4 Task × concurrency 4 = 16개`다. PaddleOCR도 최대 16개 동시 요청의 영향을
  받을 수 있으므로 거절률과 timeout을 함께 관찰한다.
- Container Insights 비용이 DEV와 PROD에 추가된다.
- SQS와 ECS CloudWatch metric은 근사값이며 1분 단위 수집·평가 지연이 있다.
- queue가 비기 전에 계속 낮은 빈도로 메시지가 들어오면 추가 Task가 오래 유지될 수 있다.
- Task protection이 없으므로 지표 경합에 의한 드문 재전달 가능성을 수용한다.
- CloudWatch dashboard에서 queue depth·oldest age·backlog per task·Desired/Running Task·DLQ를 함께
  관측한다. 애플리케이션의 parser duration과 end-to-end duration은 Grafana에서 측정한다.

## 6. Validation

- DEV에서 서로 다른 PDF 30건을 짧은 시간에 등록해 Target Tracking과 최대 4 Task 확장을 확인한다.
- 필요하면 30건 burst와 반복 실행으로 end-to-end p50·p95·p99와 성공률을 측정한다.
- Visible, NotVisible, Delayed가 모두 0이 된 뒤 DEV에서 3분 후 첫 scale-in이 발생하는지 확인한다.
- 이후 5분 cooldown마다 한 개씩 감소하고 최소 1개에서 멈추는지 확인한다.
- PaddleOCR 거절·timeout, 요청/결과 DLQ와 중복 결과를 기록한다.
- 상세 절차와 실행 결과 형식은 [문서 파서 용량 테스트](../testing/capacity/document-parser/plan.md)를 따른다.

## 7. Revisit Triggers

- 성공한 신규 파싱의 end-to-end p95가 10분을 지속적으로 초과할 때
- PaddleOCR 거절·timeout이 Task 수 또는 worker concurrency 증가와 함께 커질 때
- queue oldest age가 증가하는 동안 Parser Worker가 최대 4개에 고정될 때
- 짧은 queue idle로 scale-out과 scale-in이 반복되어 비용이나 기동 지연이 커질 때
- scale-in 중 처리 작업 종료나 중복 처리 사례가 관측될 때
- AWS 외부 Parser 환경으로 이전해 `RunningTaskCount`를 사용할 수 없게 될 때

## 8. References

- [Scale Amazon ECS services based on an Amazon SQS queue](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-autoscaling-queue.html)
- [Create a target tracking scaling policy using metric math](https://docs.aws.amazon.com/autoscaling/application/userguide/application-auto-scaling-target-tracking-metric-math.html)
- [How target tracking scaling works](https://docs.aws.amazon.com/autoscaling/application/userguide/target-tracking-scaling-policy-overview.html)
- [How step scaling works](https://docs.aws.amazon.com/autoscaling/application/userguide/step-scaling-policy-overview.html)
