# 문서 파서 용량·오토스케일링 테스트

## 문서 상태

- 상태: Draft
- 대상 환경: AWS DEV
- 관련 작업: YMC-398
- 관련 문서:
  - [논문 등록·분석](../../../features/FT-003-논문-등록-분석.md)
  - [AWS Infrastructure](../../../architecture/aws-infrastructure.md)
  - [BE ↔ AI 비동기 배선 결정](../../../decisions/ADR-002-be-ai-messaging-sqs.md)
  - [Parser Worker 오토스케일링 결정](../../../decisions/ADR-010-parser-worker-sqs-backlog-autoscaling.md)

이 문서는 Parser Worker의 사용자 응답 목표와 초기 오토스케일링 값의 계산 근거를 기록하고,
AWS DEV에서 이를 검증하는 방법을 정의한다. 실제 실행 결과는
`runs/<실행일>-<순번>.md`에 별도로 남긴다. 이 문서의 측정값과 계산은 오토스케일링 결정의
검증 자료다. 전략과 선택 근거의 SSOT는 ADR-010이며, Terraform 변수의 실제 값은 `infra` 저장소가
관리한다.

## 성능 목표

### 제안 목표

정상적으로 파싱에 성공한 신규 문서의 95%는 Backend가 업로드 완료 요청을 수락한 시점부터
`Paper`가 `COMPLETED`로 commit되는 시점까지 10분 이내에 완료되어야 한다.

이 목표는 아직 DEV 부하 테스트로 검증되지 않은 **제안값**이다. 테스트를 통과하고 팀이 제품
요구사항으로 확정하면 FT-003의 Acceptance Criteria에 반영한다.

### 측정 경계

| 항목 | 정의 |
|---|---|
| 시작 | `POST /api/papers/{paperId}/complete`가 성공하고 신규 파싱 요청이 수락된 시점 |
| 종료 | 파싱 성공 결과를 반영해 `Paper`가 `COMPLETED`로 commit된 시점 |
| 대상 | 기존 결과 재사용이 아닌 신규 기본 파싱 |
| 지연 지표 | 성공한 신규 파싱의 end-to-end duration p50·p95·p99 |
| 신뢰성 지표 | 전체 요청 중 `COMPLETED`, `FAILED`, deadline 초과 비율과 실패 원인 |

성공한 요청만으로 지연을 계산하면 빠른 성공만 남는 survivorship bias가 생길 수 있으므로,
성공률과 지연을 반드시 함께 기록한다. 목표 성공률은 실제 오류 분포와 제품 정책을 확인한 뒤
확정하며 현재는 `TBD`다.

## 기준 측정과 가정

현재 계산에 사용한 값은 다음과 같다.

| 항목 | 값 | 근거 상태 |
|---|---:|---|
| 사용자 end-to-end 목표 | 10분 | 제안값, 검증 전 |
| 단일 문서 파싱 실행 시간 p95 | 약 4분 | 기존 측정값, 표본·입력 분포·측정 구간 보완 필요 |
| 관측된 최장 실행 시간 | 약 22분 | PaddleOCR 지연 등 tail 사례, 원인별 재분류 필요 |
| Task당 worker concurrency | 4 | 현재 배포 설정 |
| Parser Worker 최소/최대 Task | 1 / 4 | 초기 제안값 |

4분 p95의 원본 표본 수, 측정 기간, PDF 페이지 수·용량 분포, 성공/실패 포함 여부와 시작·종료
경계는 첫 실행 결과를 작성하기 전에 보완한다. 22분 tail과 PaddleOCR 응답 거절은 Task 부족으로
단정하지 않고 외부 provider 지연·제한, 네트워크, 재시도와 내부 처리 시간을 분리해 기록한다.

## Target backlog 계산

사용자 목표 10분에서 파싱 실행 시간 p95 4분을 빼면 큐에서 기다릴 수 있는 시간 예산은 6분이다.

```text
허용 큐 대기 시간 = 10분 - 4분 = 6분

Task당 처리율 = concurrency 4개 / 문서당 4분
                = 분당 약 1개

target backlog per task = 허용 큐 대기 6분 × 분당 1개
                         = 6개
```

따라서 초기 `target_backlog_per_task`는 6으로 둔다. 오토스케일링 지표는 다음과 같다.

```text
ApproximateNumberOfMessagesVisible / RunningTaskCount
```

이 계산은 p95 처리 시간을 일정한 처리율로 환산한 보수적 근사다. CloudWatch 지표 수집, Target
Tracking 평가, Fargate Task 기동과 PaddleOCR tail latency가 추가되므로 수학적 SLO 보장이 아니다.
실제 end-to-end 측정으로 10분 목표를 검증하고 target을 조정한다.

최대 4개 Task가 모두 실행되면 이론상 최대 동시 파싱은 16개이고 근사 처리율은 분당 4개다.

```text
최대 동시 파싱 = 4 Task × concurrency 4 = 16개
근사 처리율    = 4 Task × 분당 1개 = 분당 4개
```

30건 burst에서 4개 Task가 즉시 준비되고 각 문서가 4분 걸린다고 단순화하면 첫 16건은 약 4분,
나머지 14건은 약 8분에 완료된다. 실제 환경에서는 scale-out 준비 시간이 더해지므로 이 burst가
10분 목표를 만족하는지는 테스트 결과로 판단한다.

## 초기 오토스케일링 전략

### Scale-out

- `ApproximateNumberOfMessagesVisible / RunningTaskCount`를 Target Tracking 지표로 사용한다.
- 목표 backlog per task는 6이다.
- 최소 Task는 1, 최대 Task는 4다.
- scale-out cooldown 초기값은 60초다.
- Target Tracking 자체의 scale-in은 비활성화한다.
- `RunningTaskCount`를 위해 ECS Container Insights를 활성화한다.

### Scale-in

- 다음 세 SQS 지표의 합이 0일 때만 queue idle로 본다.
  - `ApproximateNumberOfMessagesVisible`
  - `ApproximateNumberOfMessagesNotVisible`
  - `ApproximateNumberOfMessagesDelayed`
- queue idle 상태가 N분 연속 유지되면 Task를 한 개 줄인다.
- 한 번에 하나씩 줄이고 최소 1개를 유지한다.
- scale-in cooldown 초기값은 5분이다.
- DEV의 N은 테스트와 관측 시간을 줄이기 위해 3분으로 둔다.
- PROD의 N은 10분을 초기 후보로 두며 DEV 결과와 운영 비용을 확인한 뒤 확정한다.

이번 범위에서는 AI 애플리케이션의 ECS task scale-in protection을 추가하지 않는다. 세 큐 상태를
모두 확인해 처리 중 작업이 있는 동안 scale-in을 피하고, 예외적인 종료에서는 SQS visibility
timeout과 redrive가 재처리를 담당한다. 중복 처리 가능성과 멱등성은 테스트 결과에서 함께 확인한다.

## 테스트 범위

### 확인할 것

- backlog per task 증가에 따른 Parser Worker scale-out
- 최대 4개 Task 상한
- 큐 대기, 파싱 실행과 end-to-end 완료 시간
- queue idle 3분 이후의 단계적 scale-in
- PaddleOCR 지연·거절이 처리량과 실패율에 미치는 영향
- scale-in 또는 재전달 상황의 중복 결과와 최종 상태
- 요청·결과 DLQ 증가 여부

### 확인하지 않는 것

- PROD의 장기 비용 최적화
- 사용자별 동시 파싱 제한 정책
- PaddleOCR 자체의 공식 용량 한도
- 25~30건 단일 burst만으로 통계적으로 유효한 장기 SLO 확정

## 입력 데이터와 사전 조건

- 테스트 계정은 유효한 Pro 플랜과 최소 30회의 남은 문서 등록 사용량을 가진다.
- 완전히 동일한 파일은 checksum 기반으로 기존 파싱을 재사용할 수 있으므로 서로 다른 PDF를 쓴다.
- PDF별 페이지 수, 파일 크기, 언어, 이미지·표 포함 여부를 fixture 목록에 기록한다.
- 동일한 입력 세트를 재실행할 때는 기존 결과 재사용을 피할 수 있는 격리 방법을 먼저 정한다.
- 성공한 25~30건은 Pro 월간 문서 사용량에 확정되므로 테스트 전후 사용량을 기록한다.
- DEV 스케줄러의 종료 시간과 테스트가 겹치지 않게 한다.
- 실행 전에 Parser Worker가 desired/running 1/1이고 요청 큐와 DLQ가 비어 있는지 확인한다.

PaddleOCR에는 scale-out 후 최대 16개의 요청이 동시에 전달될 수 있다. 외부 API 비용·quota와
테스트 중단 기준을 실행 전에 확인한다. token과 사용자 식별 정보는 결과 문서에 기록하지 않는다.

## 실행 시나리오

### 1. 사전 정상 경로

1. 대표 PDF 한 건을 등록한다.
2. SQS 요청 발행, Worker 수신, 결과 발행과 `COMPLETED` 전이를 확인한다.
3. 측정 시작·종료 timestamp와 대시보드 지표가 같은 요청을 반영하는지 확인한다.

### 2. 25건 burst

1. 서로 다른 PDF 25건을 가능한 한 1분 안에 등록한다.
2. 큐 깊이, backlog per task와 desired/running Task 변화를 1분 단위로 기록한다.
3. 각 문서의 queue wait, 실행 시간과 end-to-end 시간을 수집한다.
4. 모든 요청의 terminal 상태와 실패 원인을 확인한다.

25건은 오토스케일링 배선과 최대 Task 접근 여부를 확인하는 첫 smoke test다. 표본이 작으므로 이
한 번의 p95를 제품 SLO 충족 증거로 사용하지 않는다.

### 3. 30건 burst 및 반복

1. 첫 실행의 오류와 외부 quota를 확인한 뒤 서로 다른 PDF 30건으로 반복한다.
2. 같은 조건의 burst를 최소 3회 반복할지 비용과 quota를 보고 결정한다.
3. 구성 변경 없이 반복한 결과를 합쳐 p50·p95·p99와 성공률을 계산한다.

### 4. Scale-in

1. Visible, NotVisible, Delayed가 모두 0이 된 시점을 기록한다.
2. 3개의 연속 1분 datapoint 후 첫 Task 감소가 시작되는지 확인한다.
3. cooldown마다 한 개씩 줄고 최소 1개에서 멈추는지 확인한다.
4. scale-in 중 새 요청을 한 건 추가해 정상적으로 처리되는지 확인한다.

## 관측 항목

### AWS 인프라 지표

| 지표 | 확인 목적 |
|---|---|
| SQS `ApproximateNumberOfMessagesVisible` | 대기 backlog |
| SQS `ApproximateNumberOfMessagesNotVisible` | Worker가 처리 중인 작업 |
| SQS `ApproximateNumberOfMessagesDelayed` | 지연 전달 중인 작업 |
| SQS `ApproximateAgeOfOldestMessage` | 가장 오래 기다린 메시지의 queue age |
| SQS `NumberOfMessagesSent/Received/Deleted` | 유입·수신·완료 흐름 비교 |
| ECS `DesiredTaskCount/RunningTaskCount/PendingTaskCount` | scale-out·기동·scale-in 상태 |
| metric math `Visible / RunningTaskCount` | Target Tracking 입력값 |
| 요청·결과 DLQ visible/oldest age | 최종 실패와 적체 |
| Task CPU·메모리 | 자원 포화가 처리 지연의 원인인지 확인 |

`ApproximateAgeOfOldestMessage`는 queue wait만 나타내며 메시지가 NotVisible로 바뀐 뒤의 파싱
실행 시간은 포함하지 않는다. 이 지표만으로 end-to-end 10분 목표를 판정하지 않는다.

### 애플리케이션 지표

| 지표 | 측정 경계 |
|---|---|
| queue wait | SQS 발행부터 Worker 처리 시작까지 |
| parser duration | Worker 처리 시작부터 성공·실패 결과 발행까지 |
| end-to-end duration | complete 수락부터 `Paper` terminal 상태 commit까지 |
| 결과 수 | 성공·실패·deadline·PaddleOCR 거절/timeout·재시도 소진 구분 |
| 활성 파싱 수 | Worker가 실제 처리 중인 문서 수 |

기존 로그·DB timestamp와 metric만으로 각 경계를 정확히 측정할 수 있는지는 실행 전에 확인한다.
부족한 지표는 이름, histogram bucket과 label cardinality를 별도 구현 작업에서 정한다. paper ID,
파일명과 사용자 ID는 metric label에 넣지 않는다.

## 초기 판정 기준

| 항목 | 기준 |
|---|---|
| Scale-out 동작 | backlog per task가 목표를 넘을 때 desired/running Task가 1보다 증가 |
| 최대 상한 | desired/running Task가 4를 초과하지 않음 |
| Scale-in 안전성 | 세 큐 상태 중 하나라도 0보다 크면 idle scale-in이 시작되지 않음 |
| Scale-in 동작 | queue idle 3분 뒤 한 개씩 감소하고 최소 1에서 정지 |
| End-to-end 지연 | 성공한 신규 파싱 p95 10분 이하 — 반복 표본 확보 후 판정 |
| 성공률 | `TBD` — 실패 원인 분포를 수집한 뒤 확정 |
| DLQ | 테스트가 의도하지 않은 메시지가 DLQ에 남지 않음 |
| 정합성 | 요청별 terminal 상태가 하나이며 중복 완료·중복 사용량 확정이 없음 |

25건 단일 실행은 scale 동작의 합격 여부만 판정한다. end-to-end p95와 성공률은 반복 실행으로
표본을 확보한 뒤 판정한다.

## 대시보드와 알람

첫 단계는 CloudWatch dashboard 하나에 다음 패널을 구성한다.

1. Visible, NotVisible, Delayed와 oldest message age
2. backlog per task와 목표값 6
3. Desired, Running, Pending Task count
4. 요청·결과 메시지 유입/삭제량
5. 요청·결과 DLQ visible/oldest age
6. Task CPU·메모리

애플리케이션 histogram 수집이 준비되면 Grafana dashboard에 parser duration과 end-to-end duration의
p50·p95·p99, 결과별 비율과 활성 파싱 수를 추가한다. CloudWatch dashboard는 AWS 리소스 상태,
Grafana는 애플리케이션 SLO와 원인 분석을 담당한다.

알람 threshold와 Slack 전달 정책은 이번 smoke test 결과를 본 뒤 별도로 확정한다. 최소한 DLQ
메시지 발생과 oldest message age의 지속적인 증가를 알람 후보로 둔다.

## 실행 결과 기록

실제 실행마다 `runs/<실행일>-<순번>.md`에 다음을 남긴다.

- 실행 ID·시간·환경과 적용한 plan·infra commit SHA
- BE·AI image URI/digest, ECS Task Definition revision
- min/max Task, concurrency, target backlog, cooldown과 scale-in N
- Container Insights 상태와 CloudWatch metric 주기
- PDF fixture 분포와 건수, 등록에 걸린 시간, Pro 사용량 전후
- Task 증가·감소 시각과 Application Auto Scaling activity
- queue wait, parser duration, end-to-end p50·p95·p99
- 성공률과 오류·timeout·PaddleOCR 거절·DLQ 건수
- CPU·메모리, queue depth와 oldest age
- 합격 여부, 계산과 다른 동작, 병목과 다음 조정값

결과에는 dashboard snapshot 또는 조회 링크를 연결하되 credential, token, 사용자 식별 정보와 PDF
본문은 포함하지 않는다.

## 결과에 따른 조정 원칙

- queue wait이 6분을 넘고 Task가 최대 4에 오래 머무르면 max Task 또는 외부 provider 용량을 검토한다.
- Task가 늦게 증가하면 CloudWatch 수집·평가 지연과 Fargate 기동 시간을 먼저 확인한 뒤 target 또는
  scale-out cooldown을 조정한다.
- PaddleOCR 거절률이 Task 수와 함께 증가하면 max Task나 worker concurrency를 낮추고 provider별
  동시성 제한을 별도로 둔다.
- queue는 비었지만 parser duration이 길면 오토스케일링보다 파서·외부 provider를 개선한다.
- 잦은 scale-out/in이 발생하면 PROD scale-in N과 cooldown을 늘린다.
- 충분한 반복에서 10분 목표를 만족하면 FT-003에 확정 요구사항으로 반영하고 오토스케일링 ADR에서
  이 결과를 근거로 링크한다.
