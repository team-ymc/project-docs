# ADR-003: 파일 콘텐츠 기반 중복 제거와 기본 파싱 결과 공유

## 1. Overview

- Date: 2026-07-21
- Status: Proposed
- Deciders: geunhh
- Tracking: FT-003 논문 등록 · 분석 Story 7
- Related: ADR-001, ADR-002, `contracts/frontend-backend/openapi.yaml`

## 2. Context

`Paper`는 사용자가 자신의 서재에 등록한 항목이다.

동일한 PDF 파일이 다른 파일명 또는 다른 사용자에 의해 반복 등록될 수 있지만, 파일명만으로는 동일 콘텐츠를 판별할 수 없다. 등록 건마다 기본 파싱을 실행하면 동일한 비용이 반복된다.

현재는 presigned URL을 이용해 FE가 S3로 직접 업로드한다. 따라서 BE는 업로드 전에 실제 파일 내용을 검증할 수 없으며, `paperId`는 콘텐츠를 알기 전에 생성되어 API, S3 fileKey, 비동기 메시지에서 사용된다.

## 3. Decision

### Paper와 DocumentContent 테이블을 분리한다

- `Paper`는 사용자별 등록·소유 단위다.
- `DocumentContent`는 업로드된 파일의 내용 단위다.
- 여러 `Paper`가 하나의 `DocumentContent`를 참조할 수 있다.
- `paperId`는 업로드 단위 UUID로 유지하며 콘텐츠 해시에서 파생하지 않는다.

```text
User
└─ Paper
   └─ DocumentContent
```

### 동일 콘텐츠는 검증된 SHA-256으로 판정한다

전체 파일 바이트의 SHA-256이 같은 경우에만 동일한 `DocumentContent`로 판정한다.

이는 완전히 동일한 파일을 판정하는 기준이다. DOI·arXiv ID·제목·저자 등을 이용한 의미적 동일 논문 판정과는 구분하며, 의미적 판정은 후속 단계에서 별도 기준으로 추가한다.

FE는 업로드 전에 SHA-256을 계산하고 presigned PUT에 checksum을 포함한다. S3는 업로드된 실제 바이트와 checksum의 일치를 검증하며, BE는 S3가 검증한 checksum만 중복 판정에 사용한다.

중복 여부는 업로드가 완료되기 전에 공개하지 않는다.

동일한 SHA-256은 하나의 `DocumentContent`로 수렴해야 한다.

### 동일 콘텐츠의 기본 파싱 결과를 공유한다

동일한 `DocumentContent`를 참조하는 `Paper`들은 PDF를 Markdown 본문과 미디어로 변환한 기본 파싱 결과를 공유한다.

동시 요청에서도 하나의 논리적인 기본 파싱 작업만 생성되어야 한다.

초기 구현은 단일 파싱 파이프라인 버전을 전제로 하며 별도의 `pipelineVersion`은 두지 않는다. 파싱 API·엔진 또는 산출물 계약의 변경으로 결과 호환성이 깨지는 시점에는 변경 배포 전에 기존 결과의 이관·무효화 정책과 함께 `pipelineVersion`을 도입한다.

### 실패 유형을 구분한다

- 손상 PDF, 암호화 PDF 등 콘텐츠 자체의 영구 실패는 동일 `DocumentContent`를 기다리는 모든 `Paper`에 반영한다.
- 네트워크, S3, 외부 API timeout 등 일시적 실패는 즉시 최종 실패로 반영하지 않고 설정된 최대 횟수까지 재시도한다.
- 최대 재시도 횟수를 초과하거나 DLQ로 이동한 작업은 최종 실패로 처리한다.

## 4. Options Considered

### Option A. Paper와 DocumentContent를 분리한다 — 채택

- 사용자 소유권과 콘텐츠 동일성을 분리할 수 있다.
- 여러 `Paper`가 동일 콘텐츠의 기본 파싱 결과를 공유할 수 있다.
- 중복 판정 기준이 바뀌어도 외부에 노출된 `paperId`에는 영향을 주지 않는다.
- 단점: 별도 내용 단위와 연결 관계를 관리해야 한다.

### Option B. 콘텐츠 해시를 paperId로 사용한다

- 장점: 동일한 파일이 자동으로 같은 ID를 갖는다.
- 장점: 사용자 소유권을 별도 연결로 관리하면 동일 콘텐츠를 여러 사용자가 공유할 수 있다.
- 단점: 업로드 전에 필요한 `paperId`를 파일 업로드 후에야 결정할 수 있다.
- 단점: 사용자별 등록·소유를 분리하려면 콘텐츠 해시와 별도의 사용자별 식별자 및 연결 엔티티가 필요하며, 이는 Option A와 같은 구조로 수렴한다.
- 단점: 파일 바이트 동일성과 의미적인 논문 동일성은 다르다.
- 탈락 사유: DOI·arXiv ID·제목·저자 등 새로운 중복 판정 기준이 추가되면 기존 ID를 병합하거나 alias로 관리해야 한다. 중복 판정 기준이 변해도 외부 식별자는 안정적으로 유지되어야 하므로 ID와 content hash를 분리한다.

### Option C. 파일명만으로 콘텐츠 중복을 판정한다

- 장점: 업로드 전에 간단하게 판정할 수 있다.
- 단점: 파일명이 다른 동일 파일을 찾지 못한다.
- 단점: 파일명이 같은 서로 다른 파일을 잘못 중복으로 판단할 수 있다.
- 탈락 사유: 파일명은 콘텐츠 동일성을 보장하지 않는다.
- 사용자별 동일 파일명 등록을 `409 DUPLICATE_FILENAME`으로 거절하는 정책은 서재의 별도 UX 규칙으로 유지하되, 콘텐츠 동일성 및 기본 파싱 결과 공유 여부를 판단하는 데는 사용하지 않는다.

## 5. Consequences

### Positive

- 동일 PDF의 기본 파싱 비용을 줄인다.
- 사용자 소유권과 공유 콘텐츠를 분리한다.
- 중복 판정 기준이 `paperId`와 결합되지 않는다.

### Trade-offs

- `Paper`와 `DocumentContent` 사이의 조회와 상태 처리가 추가된다.
- 업로드 완료 전에는 `Paper`의 `DocumentContent`가 결정되지 않는다.
- 동일 콘텐츠와 논리적인 기본 파싱 작업의 단일성을 보장하기 위해 충돌 처리와 원자적인 작업 선점이 추가되며, 구현·테스트 복잡도가 증가한다.
- checksum을 포함하도록 업로드 계약을 변경해야 한다.
- 비동기 처리 과정에서 메시지 재전달이나 consumer 재시작으로 물리적인 실행이 중복될 수 있으므로 AI worker와 산출물 저장은 멱등해야 한다. 현재 메시징 전달 방식은 ADR-002를 따른다.
- 재시도 소진과 DLQ 이동을 최종 실패로 반영하는 장치가 필요하다.

### Follow-ups

- OpenAPI checksum 필드와 presigned upload 계약 정의
- `Paper` / `DocumentContent` 저장 모델 설계
- 동일 content hash의 중복 생성 방지와 충돌 처리 방식 설계 (예: DB UNIQUE 제약과 upsert)
- 기본 파싱 시작 권한을 원자적으로 선점하는 방식과 트랜잭션 경계 설계 (예: 조건부 UPDATE/CAS)
- AI worker와 산출물 저장의 멱등 처리 설계
- 기본 파싱 결과와 오류 계약 정의
- 파싱 결과 호환성이 깨지는 최초 변경 전에 `pipelineVersion`과 기존 결과 이관·무효화 정책 도입
- DLQ의 최종 실패 반영 방식 결정
- 의미적 동일 논문 판정 기준과 `DocumentContent` 연결 방식 결정
- 공유 산출물의 저장 경로와 삭제·garbage collection 정책 결정

## 6. Updates

- **2026-07-21** — 초안 작성. 사용자별 `Paper`와 파일 내용 단위 `DocumentContent`를 분리하고, 검증된 SHA-256이 동일한 콘텐츠는 기본 파싱 결과를 공유하기로 제안.
