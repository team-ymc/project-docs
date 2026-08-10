# ADR-005: 문서 업로드 크기 제한은 Presigned PUT의 서명된 Content-Length로 강제한다

## 1. Overview

- Date: 2026-08-10 (Accepted: 2026-08-10)
- Status: Accepted
- Deciders: 근흐흐
- Tracking: YMC-315
- Supersedes / Superseded by: -

## 2. Context

ADR-001에 따라 PDF 원본은 BE를 경유하지 않고 FE에서 S3로 직접 업로드한다. 현재 BE는 논문 등록 시 S3 bucket과 전체 `fileKey`를 결정하고, 해당 key에 대한 Presigned PUT URL을 FE에 발급한다.

파일 업로드는 parsing 으로 이어지고, 현재 API 기반으로 동작하는 문서 파싱은 페이지 단위로 요금이 부과된다. 이는 파서 로컬 서빙에서도 서버 자원을 사용하는 건 마찬가지다. 그러나, 기존 흐름에는 단일 파일 크기나 페이지 상한이 없다. FE에서 1차적으로 크기를 검사하더라도 클라이언트 검증은 우회할 수 있으므로, 제한을 초과한 파일이 S3에 저장되기 전에 S3가 요청을 거절하도록 강제해야 한다.

이번 결정의 요구사항은 다음과 같다.

- 단일 PDF 원본의 최대 크기는 50 MiB(`52,428,800` bytes)다.
- BE가 bucket과 전체 `fileKey`를 결정하여 presigned URL을 발급한다.
- 등록 시점에 브라우저의 `File.size`로 업로드할 정확한 바이트 수를 알 수 있다.
- FE는 PDF를 압축ㆍ암호화ㆍ변환하지 않고 원본 바이트를 그대로 업로드한다.
- 동일한 key에 대한 업로드 재시도가 예측 가능한 최종 상태를 만들어야 한다.
- 파일 바이트는 BE를 경유하지 않는다.

## 3. Decision Drivers

[AWS의 PUT/POST 선택 가이드](https://aws.amazon.com/ko/blogs/networking-and-content-delivery/implementing-secure-file-uploads-to-amazon-s3-at-the-edge-choosing-the-right-pattern/)가 제시하는 판단 기준을 현재 요구사항과 대조했다.

| AWS의 판단 기준 | 현재 요구사항 | 판단 |
|---|---|---|
| bucket과 전체 object key를 알고 있고 업로드를 확정적으로 제어한다 | BE가 전체 `fileKey`를 생성한다 | PUT에 부합한다 |
| 동일 요청의 재시도가 같은 최종 상태를 만든다 | 네트워크 실패 시 같은 key로 재시도해야 한다 | PUT의 멱등적 의미가 적합하다 |
| 파일 이름, 크기와 content type이 애플리케이션에서 미리 결정된다 | 등록 시 정확한 크기와 `application/pdf`를 알 수 있다 | PUT에 부합한다 |
| HTML form을 통한 브라우저 업로드다 | React 애플리케이션이 JavaScript로 직접 업로드한다 | 브라우저라는 이유만으로 POST가 필수는 아니다 |
| 크기 범위ㆍ타입 등 여러 정책 조건을 S3가 검증해야 한다 | 현재 필요한 조건은 상한과 등록한 정확한 크기의 일치다.     | POST Policy의 범위 조건 없이 PUT 서명으로 충족 가능한지 검증한다 |
| key prefix를 사용해 업로드 key를 구성한다 | key 생성과 소유권은 BE가 담당한다 | POST의 key 구성 유연성이 필요하지 않다 |

해당 문서에서는 몇몇의 이유로 HTML 폼 기반 요청과 웹 애플리케이션에는 POST policy를 권장한다. 그러나, PUT과 POST의 메커니즘과 현 요구사항을 비교하며 결정한다.

S3 POST가 브라우저 업로드용으로 설계된 배경에는, 당시 JavaScript에서 임의의 요청 헤더를 다루기 어려워 인증값과 정책을 HTML form field로 전달해야 했던 제약이 있다. 현대 브라우저는 Fetch API 등으로 Presigned PUT URL에 `File` 원본을 직접 전송할 수 있으므로 **클라이언트가 브라우저라는 사실만으로 POST를 선택하지 않는다.**

다만 POST의 장점이 과거의 호환성에만 있는 것은 아니다. POST Policy의 `content-length-range`는 현재도 S3가 업로드 크기 범위를 저장 전에 검증할 수 있는 명시적인 수단이다. 이 장점과 현재 구조의 적합성ㆍ변경 비용을 함께 비교한다.

## 4. Decision

Option A인 **Presigned PUT과 서명된 정확한 `Content-Length`**를 채택한다.

BE는 등록 요청으로 전달된 파일 크기가 1 byte 이상 50 MiB 이하인 경우에만 Presigned PUT URL을 발급한다. Presigned PUT 요청에는 BE가 생성한 전체 `fileKey`, `Content-Type: application/pdf`와 등록 시 전달된 정확한 `Content-Length`를 포함한다.

FE는 압축이나 변환 없이 PDF 원본을 PUT body로 전송한다. 실제 업로드 크기가 서명된 `Content-Length`와 다르면 S3가 요청을 거절해야 한다.

이 결정은 허용 범위 안의 임의 크기를 받는 것이 아니라 다음 두 조건을 순서대로 강제한다.

1. BE: `K <= 50 MiB`
2. S3: `actual Content-Length == K`

등록, 업로드 완료 통보, S3 HEAD 확인과 파싱 요청으로 이어지는 전체 처리 흐름은 ADR-001을 따른다.

`Content-Length`는 파일 크기의 일치만 보장한다. 같은 크기의 다른 파일까지 구분해야 한다면 checksum 도입을 별도로 결정한다.

## 5. Options Considered

### Option A. Presigned PUT + 서명된 정확한 `Content-Length` — 채택

- BE가 전체 key와 예상 크기를 결정하는 현재 모델에 부합한다.
- 등록한 크기와 실제 업로드 크기를 정확히 결합할 수 있다.
- 기존 raw PUT, API 응답과 S3 CORS 구조를 유지한다.
- POST Policy와 별도의 서명 로직을 도입하지 않는다.
- 정확한 크기를 사전에 알아야 한다.
- 브라우저가 생성하는 `Content-Length`가 실제 AWS S3의 SigV4 검증에 포함되는지 실환경 검증이 필요하다.

### Option B. Presigned POST + `content-length-range`

- POST Policy에 `content-length-range`, 정확한 key, content type과 만료 조건을 명시할 수 있다.
- S3가 정책 조건에 맞지 않는 업로드를 object 저장 전에 거절한다.
- FE 전송을 raw PUT에서 `multipart/form-data`로 변경해야 한다.
- BE가 URL과 함께 policy field들을 반환하도록 API 계약이 바뀐다.
- S3 CORS에 POST를 허용해야 하며 배포 중 PUT/POST 호환 순서를 관리해야 한다.
- 현재 사용하는 AWS SDK v2의 S3 presigner에는 Presigned POST를 만드는 고수준 API가 없어 POST Policy와 SigV4 서명 로직을 별도로 소유해야 한다.
- POST도 정확한 key를 고정할 수 있으므로, AWS 글의 서로 다른 key로 인한 중복 object 사례를 이 시스템의 직접적인 탈락 사유로 삼지는 않는다.

### Option C. BE 프록시 업로드

- BE가 업로드 stream을 직접 보면서 크기와 파일 내용을 검사할 수 있다.
- 대용량 파일 트래픽과 동시 업로드 부하가 일반 API를 처리하는 BE에 결합된다.
- ADR-001에서 선택한 FE -> S3 직접 업로드 구조와 어긋난다.
- 크기 제한만을 위해 파일 byte를 BE가 중계할 필요가 없으므로 선택하지 않는다.

## 6. Consequences

### Positive

- 기존 Presigned PUT 흐름과 BE 소유의 key 생성 규칙을 유지한다.
- 상한을 넘는 신고값에는 업로드 권한을 발급하지 않는다.
- 등록한 크기와 정확히 일치하는 body만 저장된다. 범위를 허용하는 POST Policy와 달리 신고값과 다른 업로드는 상한 이내라도 거절되므로, BE가 아는 크기와 실제 object가 항상 같다.

### Trade-offs & 한계

- FE가 보내는 `size` 자체는 신뢰값이 아니다. BE가 허용 범위를 검사하고 그 값을 서명 조건으로 고정함으로써 이후 실제 요청과의 일치만 강제한다.
- 정확한 크기를 사전에 알아야 하므로 업로드 중 압축ㆍ변환이 불가능하다. 현재는 원본을 그대로 올려 드러나지 않는다.
- 정확한 크기만으로는 같은 크기의 다른 파일을 구분하지 못한다. 파일 동일성이 필요하면 checksum이 추가로 필요하다.
- Presigned URL은 만료 전까지 재사용할 수 있다. 크기 상한은 요청 한 건의 상한이지 횟수나 누적 용량의 상한이 아니다.
- 파일 개수ㆍ누적 용량ㆍ요청 빈도는 제어하지 않는다. 이는 BE quota와 rate limit의 책임이다.
- Complete API의 HEAD 검증은 방어 계층이지만, S3 업로드 요청 자체가 소비한 request 비용까지 제거하지는 않는다.

## 7. Revisit Triggers

다음 조건이 생기면 PUT과 POST 선택을 다시 검토한다.

- 업로드 전에 정확한 파일 크기를 알 수 없는 stream을 지원한다.
- FE에서 압축ㆍ암호화ㆍ파일 변환을 수행한다.
- 정확한 크기가 아니라 다양한 크기 범위와 복수의 policy condition을 S3에서 관리해야 한다.
- 대용량 파일의 중단 후 재개가 필요해 S3 Multipart Upload를 도입한다.
- checksum을 포함한 요청 헤더를 대상 브라우저에서 일관되게 전송할 수 없다.

## 8. References

- [AWS — S3에서 안전한 파일 업로드 구현: 적합한 패턴 선택](https://aws.amazon.com/ko/blogs/networking-and-content-delivery/implementing-secure-file-uploads-to-amazon-s3-at-the-edge-choosing-the-right-pattern/)
- [AWS — Download and upload objects with presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html)
- [AWS — PutObject API](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutObject.html)
- [AWS — Checking object integrity for data uploads](https://docs.aws.amazon.com/AmazonS3/latest/userguide/checking-object-integrity-upload.html)

## 9. Updates

- **2026-08-10** — 초안 작성. Presigned PUT의 정확한 `Content-Length` 서명 동작을 실제 Chrome과 AWS 개발 bucket에서 검증한 뒤 Accepted 여부를 결정한다.
- **2026-08-10** — **Accepted로 전환.** dev 환경과 실제 Dev 서버에서 확인했다(검증 절차는 team-ymc/app#39). 발급된 URL의 `X-Amz-SignedHeaders`에 `content-length`가 포함되고, 등록한 크기 `K`의 PUT은 성공하는 반면 `K+1`은 `403`으로 거절된다. 거절된 PUT은 객체를 남기지 않아 이어진 `complete`가 `409 UPLOAD_NOT_FOUND`를 반환했고, 같은 URL에 `K`로 다시 올리면 성공해 재시도가 complete 흐름과 충돌하지 않는다.
- **2026-08-10** — §6이 별도 결정으로 남긴 "versioning과 URL 재사용 정책"은 원본 버킷 versioning을 `Suspended`로 내리는 것으로 결정했다.
