# SQS 처리 가이드

이 문서는 SQS를 사용하는 **BE·AI 애플리케이션**과 큐를 구성하는 **인프라**의 책임을
구분한다. 아래 환경변수 이름은 구현을 위한 예시이며 계약으로 고정하지 않는다.

메시지 payload 스키마는 [`messaging.yml`](messaging.yml)을 따른다.

## 애플리케이션

| 큐 | 흐름               | 용도 |
|---|------------------|---|
| `parse-requests` | BE → AI          | PDF 파싱 요청 |
| `parse-results` | 파싱 결과 producer → BE | 파싱 완료·실패 결과 |
| `knowledge-compile-requests` | BE → AI | 완료된 파싱 패키지의 지식 번들 생성 요청 |
| `knowledge-compile-results` | 지식 번들 결과 producer → BE | 지식 번들 생성 완료·실패 결과 |

## 단계 연결

1. BE가 `parse-requests`를 발행한다.
2. 파싱 worker는 기존 파싱 패키지와 `manifest.json`을 완성하고
   `parse-results`를 발행한 뒤 요청을 ACK한다.
3. BE는 파싱 완료 결과를 반영해 기존 문서 기능을 사용 가능하게 한 뒤,
   같은 `paper_id`와 받은 `manifest_key`로 `knowledge-compile-requests`를 발행한다.
4. knowledge compile worker는 지식 번들 산출물을 모두 저장한 뒤, 요청으로 받은
   중앙 `manifest_key`를 포함해 `knowledge-compile-results`를 발행하고 요청을 ACK한다.
5. BE는 knowledge compile 완료 결과를 반영한 뒤에만 전체 번역과 지식 번들 기능을
   활성화한다. 전체 번역 사이드카는 파싱이 아니라 이 단계의 산출물이다.

knowledge compile 성공 결과의 `manifest_key`는 최초 파싱 성공 결과와 같은 중앙
`manifest.json`을 가리킨다. 별도의 지식 번들 manifest는 생성하지 않는다. 모든
compile 산출물을 저장한 뒤 중앙 manifest를 마지막으로 다시 발행하며, `artifacts`에
`structure_translation_ko`, `frontend_translation_ko`,
`structure_prerequisite_highlights`, `frontend_prerequisite_highlights`,
`knowledge_bundle_viz`를 기록한다.

지식 번들 생성은 기존 파싱의 후속 작업이지만 별도의 요청·결과·재시도
주기를 갖는다. 지식 번들 생성이 진행 중이거나 실패해도 이미 완료된
파싱 패키지의 사용 가능 상태를 되돌리지 않는다. 파싱 패키지를 다시 생성하면
사이드카와 지식 번들이 함께 제거되므로, BE는 재파싱 완료 뒤
`knowledge-compile-requests`를 다시 발행한다.

BE는 컴파일 `failed` 결과를 받으면 컴파일 상태와 실패 코드를 기록하고 전체 번역과
지식 번들을 비활성으로 둔다. 문서의 파싱 상태는 바꾸지 않는다. 같은 `manifest_key`로
자동 재요청하지 않으며, 다시 시도하려면 재파싱 경로를 탄다.

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
| AI | knowledge compile 요청 queue name | `KNOWLEDGE_COMPILE_SQS_REQUEST_QUEUE_NAME` | 기본값 `knowledge-compile-requests` |
| AI | knowledge compile 결과 queue name | `KNOWLEDGE_COMPILE_SQS_RESULT_QUEUE_NAME` | 기본값 `knowledge-compile-results` |
| 공통 | AWS region | `AWS_DEFAULT_REGION` | 기본값 `ap-northeast-2` |
| 공통 | endpoint override | `AWS_ENDPOINT_URL` | LocalStack에서만 주입하고 실제 AWS에서는 미설정 |
| 공통 | LocalStack credential | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` | 로컬에서만 test 값 사용 |
| AI | visibility 연장 시간 | `DOCUMENT_PARSER_SQS_VISIBILITY_TIMEOUT_SECONDS` | heartbeat 호출 시점부터 다시 확보할 시간 |
| AI | heartbeat 간격 | `DOCUMENT_PARSER_SQS_HEARTBEAT_INTERVAL_SECONDS` | visibility 연장 시간보다 짧게 설정 |
| AI | 전체 실행 deadline | `DOCUMENT_PARSER_DEADLINE_SECONDS` | heartbeat와 별도로 제한할 파싱 시간 |
| AI | knowledge compile visibility 연장 시간 | `KNOWLEDGE_COMPILE_SQS_VISIBILITY_TIMEOUT_SECONDS` | heartbeat 호출 시점부터 다시 확보할 시간 |
| AI | knowledge compile heartbeat 간격 | `KNOWLEDGE_COMPILE_SQS_HEARTBEAT_INTERVAL_SECONDS` | visibility 연장 시간보다 짧게 설정 |
| AI | knowledge compile 전체 실행 deadline | `KNOWLEDGE_COMPILE_DEADLINE_SECONDS` | heartbeat와 별도로 제한할 지식 번들 생성 시간 |

LocalStack에서만 endpoint와 test credential을 주입한다. 실제 AWS에서는
endpoint를 지정하지 않고 ECS task role이나 EKS workload role을 사용한다.

### 메시지 처리

모든 큐는 at-least-once로 전달된다. 애플리케이션은 중복 전달을 전제로 상태 전이와 부수 효과를 멱등하게 처리해야 한다.

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

### 실패·재시도 처리 원칙

1. 일시 실패에서는 결과를 발행하거나 원본 요청을 삭제하지 않는다.
2. 확정 실패는 해당 단계의 result 큐에 `failed` 결과 발행을 성공한 뒤 원본
   요청을 삭제한다. Consumer는 결과 발행 주체와 관계없이 `completed`와 `failed`를
   해당 result consumer에서 처리한다.
3. 앱은 재시도 횟수나 마지막 시도를 판단하지 않는다. `ApproximateReceiveCount`, DLQ
URL·ARN과 `maxReceiveCount`도 애플리케이션 설정으로 받지 않는다.
    - 즉 DLQ의 존재를 애플리케이션은 모른다.

## 인프라

인프라는 다음 항목을 구성한다.

- 요청·결과 큐와 각 DLQ
- `VisibilityTimeout`, `maxReceiveCount`, retention, redrive policy
- request DLQ에 연결되는 Lambda
- 큐별 IAM 최소 권한과 모니터링

SQS가 재시도 소진 요청을 request DLQ로 이동시키면 Lambda가 해당 result 큐에
최종 실패를 발행한다.

- `parse-requests-dlq` → `parse-results`: `PARSE_RETRIES_EXHAUSTED`
- `knowledge-compile-requests-dlq` → `knowledge-compile-results`: `KNOWLEDGE_COMPILE_RETRIES_EXHAUSTED`

주요 권한은 다음과 같다.

| 주체 | 권한 |
|---|---|
| BE | `GetQueueUrl`, 두 요청 큐 `SendMessage`, 두 결과 큐 `ReceiveMessage`·`DeleteMessage` |
| 파서 워커 | `GetQueueUrl`, `parse-requests` `ReceiveMessage`·`DeleteMessage`·`ChangeMessageVisibility`, `parse-results` `SendMessage` |
| 컴파일 워커 | `GetQueueUrl`, `knowledge-compile-requests` `ReceiveMessage`·`DeleteMessage`·`ChangeMessageVisibility`, `knowledge-compile-results` `SendMessage` |
| Lambda | 두 request DLQ 소비, 해당 결과 큐 `SendMessage` |
