# ADR-003: 파일 콘텐츠 기반 Document 중복 제거와 기본 파싱 결과 공유

## 1. Overview

- Date: 2026-07-21
- Status: Proposed
- Deciders: geunhh
- Tracking: FT-003 논문 등록 · 분석 Story 7
- Related: ADR-001, ADR-002, `contracts/frontend-backend/openapi.yaml`,
  `contracts/backend-ai/sqs/messaging.yml`

## 2. Context

`Paper`는 사용자가 자신의 서재에 등록한 항목이다. 동일한 PDF가 다른 파일명이나 사용자로부터
반복 등록될 수 있지만, 파일명만으로는 동일 콘텐츠를 판별할 수 없고 등록마다 파싱하면 비용이
중복된다.

FE는 presigned URL로 S3에 직접 업로드하므로 BE는 업로드 전에 파일 내용을 검증할 수 없다.
또한 현재 원본, 파싱 요청·결과와 DB 산출물이 모두 `paperId`에 귀속되어 있어 여러 `Paper`가
같은 파일의 파싱 결과를 공유할 수 없다.

## 3. Decision

### Paper와 Document를 분리한다

- `Paper`는 사용자별 등록·소유 단위다.
- `Document`는 검증된 파일 바이트, 대표 원본과 기본 파싱 결과의 공유 단위다.
- 여러 `Paper`가 하나의 `Document`를 참조한다.
- `paperId`와 내부 `documentId`는 각각 UUID로 만들며 콘텐츠 해시에서 파생하지 않는다.
- 외부 API와 인가는 계속 `paperId`를 기준으로 한다.

```text
User 1 ── N Paper N ── 1 Document
                              ├─ 대표 원본
                              ├─ 파싱 상태와 오류
                              └─ manifest / content / blocks / assets / ...
```

업로드 검증 전에는 `Paper.documentId`가 없다. 연결 전 업로드 상태는 `Paper`가, 연결 후 파싱
상태와 오류는 `Document`가 관리하며 Paper 상태 API는 이를 기존 `PaperStatus`로 보여준다.

### S3가 검증한 SHA-256으로 동일 파일을 판정한다

전체 파일 바이트의 SHA-256이 같은 경우에만 동일한 `Document`로 판정한다. DOI·arXiv ID·제목·
저자 등을 이용한 의미적 동일 논문 판정은 별도 후속 범위다.

FE는 업로드 전에 SHA-256을 계산하고 presigned PUT의 서명 헤더에 checksum을 포함한다. S3는
업로드된 실제 바이트와 checksum의 일치를 검증하며, BE는 S3가 검증한 값만 사용한다.

클라이언트가 제출한 checksum만으로 업로드를 생략하거나 기존 `Document`에 연결하지 않는다.
그렇게 하면 실제 파일을 제출하지 않은 사용자가 이미 알고 있는 checksum만으로 기존 산출물에
접근하거나, 서비스가 특정 파일을 보유하는지 확인할 수 있기 때문이다.

checksum에는 DB 중복 방지 제약을 두어 동시 요청에서도 하나의 `Document`만 생성되게 한다.

### complete에서 Document를 결정한다

모든 등록은 실제 파일을 S3에 먼저 업로드한다. complete는 S3가 검증한 checksum으로 신규 또는
기존 `Document`를 결정한다.

```text
S3 HEAD로 실제 객체와 checksum 검증
→ checksum으로 Document 조회 또는 생성

신규 checksum
  → 현재 업로드 객체를 대표 원본으로 사용하는 Document 생성
  → Paper.documentId 연결
  → 현재 paperId로 기본 파싱 요청 발행

기존 checksum
  → Paper.documentId를 기존 Document에 연결
  → 파싱 요청을 발행하지 않음
  → 방금 업로드한 중복 객체 삭제
```

신규 파일은 최초 업로드 객체 자체를 대표 원본으로 사용하며 다른 S3 key로 복제하지 않는다.
중복 파일은 Paper 연결이 DB에 반영된 뒤 방금 업로드한 객체를 삭제한다. S3 삭제 실패는 complete를
실패로 되돌리지 않고, 남은 객체는 후속 정리 작업에서 다시 삭제한다.

### 파싱 결과는 Document가 공유한다

원본, 파싱 상태, 오류, manifest와 DB의 content·block·asset은 `documentId`를 기준으로 저장한다.
동시 요청에서도 하나의 기본 파싱 작업만 시작되도록 조건부 갱신으로 시작 권한을 선점한다.

BE 내부 공유 단위는 `Document`로 바꾸지만 초기 BE↔AI 메시지는 기존 `paper_id`와 `file_key`를
유지한다. 신규 `Document`를 만든 최초 `paperId`를 AI의 작업 식별자로 사용하고, BE는 이
`paperId`와 `Document`의 연결을 보존하여 AI 결과를 공유 `Document`에 반영한다. 이 대표
`paperId`는 내부 작업 참조일 뿐 `Document`의 소유자를 뜻하지 않는다.

### 초기에는 파싱 버전을 분리하지 않는다

초기 구현은 `Document`당 하나의 파싱 결과만 관리하며 `pipelineVersion`이나 `DocumentParse`
테이블을 두지 않는다. 두 번째 파싱 버전을 운영하기 전에 기존 결과 이관과 버전별 산출물 저장
정책을 결정한다. 기존 `manifest_version`과 `schema_version`은 산출물 형식 버전이므로 이 결정과
별개다.

### 실패 유형을 구분한다

- 손상·암호화 PDF 같은 영구 실패는 `Document`에 기록하고 모든 연결 `Paper`에 동일하게 보여준다.
- 네트워크·S3·외부 API timeout 같은 일시적 실패는 재시도하고, 재시도 소진이나 DLQ 이동 시
  `Document`의 최종 실패로 처리한다.

### Paper 삭제는 Document를 삭제하지 않는다

Paper 삭제는 서재 항목과 `Paper → Document` 관계만 제거한다. 대표 `Paper`가 삭제돼도
`Document`, 대표 원본, 파싱 산출물과 AI 결과 연결 정보는 유지한다.

초기 구현에는 사용자가 `Document`를 물리적으로 삭제하는 기능을 두지 않는다. 참조하는 `Paper`가
없는 `Document`의 보존 기간과 S3 객체 삭제는 후속 정책으로 결정한다.

사용자는 `documentId`, checksum이나 공유 S3 key로 직접 접근하지 않는다. 원본과 산출물 조회는
항상 사용자 소유 `Paper`를 확인한 뒤 `Document`를 따라간다.

## 4. Options Considered

### Option A. Paper와 Document를 분리한다 — 채택

사용자 소유권과 파일 동일성을 분리하면서 외부 `paperId`와 초기 AI 메시지 계약을 유지할 수 있다.
대신 두 단위의 연결과 상태 조회를 관리해야 한다.

### Option B. 콘텐츠 해시를 paperId로 사용한다

동일 파일이 같은 ID를 갖지만 업로드 전에는 ID를 결정할 수 없고, 사용자별 소유 단위를 위해 결국
별도 엔티티가 필요하다. 의미적 동일 논문 판정 기준이 추가될 때 외부 ID도 불안정해지므로
채택하지 않는다.

### Option C. 파일명으로 중복을 판정한다

파일명이 다른 동일 파일을 찾지 못하고 같은 이름의 다른 파일을 잘못 판정할 수 있다. 사용자별
동일 파일명을 `409 DUPLICATE_FILENAME`으로 거절하는 UX 규칙에만 사용한다.

## 5. Consequences

### Positive

- 동일 PDF의 원본, 파싱 비용과 산출물 중복을 줄인다.
- 사용자 소유권과 공유 콘텐츠를 분리한다.
- 외부 Paper API와 초기 BE↔AI 메시지 계약을 유지할 수 있다.

### Trade-offs

- `Paper`와 `Document` 연결, 상태 조회와 동시 요청 충돌 처리가 추가된다.
- checksum을 포함하도록 업로드 계약을 변경해야 한다.
- AI 메시지가 중복 전달돼도 결과가 중복 저장되지 않도록 해야 한다.
- 삭제에 실패한 중복 업로드 객체와 장기간 참조가 없는 `Document`의 정리 정책이 필요하다.
- 두 번째 파싱 버전을 도입할 때 저장 모델과 산출물 경로를 이관해야 한다.

### Follow-ups

- checksum을 포함한 업로드 계약과 Paper 상태 조회 규칙 정의
- `Paper` / `Document` 저장 모델, 기존 데이터와 DB 파싱 산출물 이관 설계
- 동시 complete, 파싱 시작 권한과 S3 중복 객체 삭제의 처리 순서 설계
- 대표 `paperId`를 이용한 AI 결과 연결, 중복 메시지와 DLQ 처리 설계
- 삭제에 실패한 업로드 객체와 참조 없는 `Document`의 정리 정책 결정
- 두 번째 파싱 버전 배포 전 `DocumentParse`와 기존 결과 이관 정책 도입
- 의미적 동일 논문 판정 기준 결정

## 6. Updates

- **2026-07-21** — 사용자별 `Paper`와 파일 내용 단위를 분리하고, 검증된 SHA-256이 같은 파일은
  기본 파싱 결과를 공유하기로 제안.
- **2026-08-11** — 공유 단위를 `Document`로 정리. complete의 신규·중복 처리, 대표 원본과
  파싱 결과의 공유, 기존 AI `paper_id` 유지, Paper 삭제와 파싱 버전 범위를 구체화.
