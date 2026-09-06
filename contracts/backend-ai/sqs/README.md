# SQS 처리 가이드

이 문서는 SQS를 사용하는 **BE·AI 애플리케이션**과 큐를 구성하는 **인프라**의 책임을
구분한다. 아래 환경변수 이름은 구현을 위한 예시이며 계약으로 고정하지 않는다.

메시지 payload 스키마는 [`messaging.yml`](messaging.yml)을 따른다.
AI 구현 기준은 `team-ymc/ai`의 `app/modules/document_parser_worker/`이며 worker는
`python -m app.modules.document_parser_worker`로 FastAPI와 별도 실행된다.

## 애플리케이션

| 큐 | 흐름 | 용도 |
|---|---|---|
| `parse-requests` | BE → AI | PDF 파싱 요청 |
| `parse-results` | 파싱 결과 producer → BE | 파싱 완료·실패 결과 |

### 클라이언트와 QueueUrl

region, endpoint override, credentials와 queue name은 인프라가 환경별로 제공한다.

- LocalStack 설정값과 실행 방법은 `{infra-repo}/local/localstack/README.md`를 따른다.

앱은 시작 시점 또는 큐를 처음 사용할 때 SDK를 통해 SQS `GetQueueUrl` API를 호출하고,
반환된 QueueUrl을 프로세스에 캐시한다. 이후 SQS 메시지 API에는 캐시한 QueueUrl을 사용한다.

- SDK 메서드명과 호출 형식은 언어별 AWS SDK를 따른다.

### 애플리케이션 설정

| 범위 | 설정 | 환경변수 예시 | 설명 |
|---|---|---|---|
| AI | 요청 queue name | `DOCUMENT_PARSER_SQS_REQUEST_QUEUE_NAME` | 기본값 `parse-requests` |
| AI | 결과 queue name | `DOCUMENT_PARSER_SQS_RESULT_QUEUE_NAME` | 기본값 `parse-results` |
| 공통 | AWS region | `AWS_DEFAULT_REGION` | 기본값 `ap-northeast-2` |
| 공통 | endpoint override | `AWS_ENDPOINT_URL` | LocalStack에서만 주입하고 실제 AWS에서는 미설정 |
| 공통 | LocalStack credential | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` | 로컬에서만 test 값 사용 |
| AI | visibility 연장 시간 | `DOCUMENT_PARSER_SQS_VISIBILITY_TIMEOUT_SECONDS` | heartbeat 호출 시점부터 다시 확보할 시간 |
| AI | heartbeat 간격 | `DOCUMENT_PARSER_SQS_HEARTBEAT_INTERVAL_SECONDS` | visibility 연장 시간보다 짧게 설정 |
| AI | 전체 실행 deadline | `DOCUMENT_PARSER_DEADLINE_SECONDS` | heartbeat와 별도로 제한할 파싱 시간 |

LocalStack에서만 endpoint와 test credential을 주입한다. 실제 AWS에서는
endpoint를 지정하지 않고 ECS task role이나 EKS workload role을 사용한다.

### 메시지 처리

두 큐는 at-least-once로 전달된다. 애플리케이션은 중복 전달을 전제로 상태 전이와 부수 효과를 멱등하게 처리해야 한다.

AI worker는 한 번의 `ReceiveMessage`에서 최대 한 요청을 받고, request의 `file_key`를
해석하거나 `paper_id`로 재조립하지 않고 정확한 S3 object key로 사용한다. 생성 artifact는
paper package prefix 아래에 저장하고 `manifest.json`을 package completion marker로 마지막에
올린 뒤 completed 결과를 발행한다.

| 항목 | 설명 |
|---|---|
| `GetQueueUrl` | queue name으로 QueueUrl을 조회하는 SQS API다. 최초 한 번 조회한 뒤 캐시한다. |
| `QueueUrl` | `SendMessage`, `ReceiveMessage`, `DeleteMessage`, `ChangeMessageVisibility`의 대상 큐를 지정한다. |
| `SendMessage` | 계약 형식의 요청 또는 결과를 대상 큐에 발행한다. |
| `ReceiveMessage` | 메시지를 수신한다. 수신만으로 ACK되거나 삭제되지는 않는다. |
| `DeleteMessage` | 처리가 끝난 메시지를 ACK한다. AI는 결과 발행 후, BE는 결과 반영 후 호출한다. |
| `ChangeMessageVisibility` | 장기 작업의 visibility를 연장하는 heartbeat로 사용한다. |
| `ReceiptHandle` | `DeleteMessage`와 `ChangeMessageVisibility`에 가장 최근 수신 값을 사용한다. |
| `WaitTimeSeconds` | long polling 대기 시간이다. HTTP read timeout은 이 값보다 길게 둔다. |
| `MaxNumberOfMessages` | 한 번에 받을 메시지 수다. 처리 가능한 동시성보다 크게 잡지 않는다. |
| 전체 실행 deadline | heartbeat와 별도로 둔다. 초과하면 메시지를 삭제하지 않아 재전달되게 한다. |

### AI worker 처리 순서

1. request JSON을 decode하고 상관관계에 사용할 수 있는 `paper_id`와 `file_key`를 검증한다.
2. visibility heartbeat를 시작한다.
3. 정확한 `file_key`의 원본을 내려받고 OCR, source language 감지, 필요한 영어→한국어 block
   번역과 document package 생성을 수행한다.
4. generated package upload와 completion manifest 저장을 완료한다.
5. `parse-results`에 completed 또는 확정 failed 결과를 발행한다.
6. 결과 발행이 성공한 뒤에만 `parse-requests`의 원본 메시지를 삭제한다.

중간 progress 메시지는 SQS에 발행하지 않는다. `source_language`, `text_kor`와 asset metadata는
결과 메시지가 아니라 생성된 package에 들어간다.

### 실패·재시도 처리 원칙

1. 일시 실패에서는 결과를 발행하거나 원본 요청을 삭제하지 않는다.
2. 확정 실패는 `failed` 결과 발행에 성공한 뒤 원본 요청을 삭제한다. Consumer는 결과 발행 주체와 관계없이
   `completed`와 `failed`를 동일한 consumer에서 처리한다.
3. 앱은 재시도 횟수나 마지막 시도를 판단하지 않는다. `ApproximateReceiveCount`, DLQ
URL·ARN과 `maxReceiveCount`도 애플리케이션 설정으로 받지 않는다.
    - 즉 DLQ의 존재를 애플리케이션은 모른다.

| 상황 | 결과 발행 | request 삭제 | 다음 처리 |
|---|---:|---:|---|
| 정상 완료 | `completed` | O | BE가 결과 반영 |
| usable `paper_id` + 비어 있거나 잘못된 `file_key` | `failed / FILE_KEY_INVALID` | O | BE가 확정 실패 처리 |
| parser가 거부한 `paper_id` | `failed / PAPER_ID_INVALID` | O | BE가 확정 실패 처리 |
| source object 없음 | `failed / DOCUMENT_SOURCE_NOT_FOUND` | O | BE가 확정 실패 처리 |
| 깨진 JSON 또는 상관관계 불가능한 `paper_id` | 없음 | X | SQS 재전달 후 request DLQ |
| PaddleOCR/translation credential 설정 오류 | 없음 | X | 운영 설정 복구 후 재전달 |
| attempt deadline 초과 | 없음 | X | SQS 재전달 |
| provider, upload, 결과 발행 등 예상하지 못한 오류 | 없음 | X | SQS 재전달 |

heartbeat 연장 실패는 기록하지만 진행 중인 parse를 즉시 중단하지 않는다. 이때 visibility가
만료되면 동일 요청이 병렬 또는 후속 전달될 수 있으므로 AI의 package overwrite와 BE의 결과
반영 모두 중복을 허용해야 한다.

## 인프라

인프라는 다음 항목을 구성한다.

- 요청·결과 큐와 각 DLQ
- `VisibilityTimeout`, `maxReceiveCount`, retention, redrive policy
- request DLQ에 연결되는 Lambda
- 큐별 IAM 최소 권한과 모니터링

SQS가 재시도 소진 요청을 `parse-requests-dlq`로 이동시키면 Lambda가 기존 계약의
`failed / PARSE_RETRIES_EXHAUSTED` 결과를 `parse-results`에 발행한다.
이 결과는 AI worker가 아니라 인프라 Lambda가 만드는 동일 `ParseFailedResult` wire shape다.

주요 권한은 다음과 같다.

| 주체 | 권한 |
|---|---|
| BE | `GetQueueUrl`, 요청 `SendMessage`, 결과 `ReceiveMessage`·`DeleteMessage` |
| AI | `GetQueueUrl`, 요청 `ReceiveMessage`·`DeleteMessage`·`ChangeMessageVisibility`, 결과 `SendMessage` |
| Lambda | request DLQ 소비, 결과 `SendMessage` |
