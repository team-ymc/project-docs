# ADR-012: prod 배포는 infra 저장소의 수동 워크플로로 dev 산출물을 SHA 지정 승격

## 1. Overview

- Date: 2026-09-25
- Status: Accepted
- Deciders: 근흐흐
- Tracking: YMC-418, YMC-399
- Related: [CI/CD](../architecture/ci-cd.md), [ADR-009](ADR-009-backend-database-schema-migrations.md)

## 2. Context

dev는 `app`·`ai` 저장소 `main` push가 곧 배포다. prod는 dev에서 검증한 AI·Backend·FE 조합을 사람이 골라 올려야 하고, 세 저장소의 산출물을 정해진 순서로 배포해야 한다. 팀은 3명이고 GitHub 저장소는 private이다.

## 3. Decision

- prod 배포는 `infra` 저장소의 `workflow_dispatch` 워크플로 하나로 한다. 입력은 AI·Backend·FE commit SHA 세 개이며 모두 선택이다. 채운 대상만 AI → Backend → FE 순서로 직렬 배포한다.
- prod에서 다시 빌드하지 않는다. AI·Backend는 dev ECR 이미지를 prod ECR로 복사하고, FE는 dev 배포 때 보관한 `dist` tar를 prod S3에 푼다.
- 승인 단계를 두지 않는다. 워크플로를 실행하는 사람이 승인자다.
- AI는 API와 두 Worker가 같은 이미지를 쓰므로 SHA 하나로 함께 배포한다.
- 롤백 전용 워크플로는 만들지 않고 같은 워크플로에 이전 SHA를 넣는다.

## 4. Options Considered

### infra 저장소 수동 dispatch, 승인 없음 — 채택

조합과 순서를 한 곳에서 다루고 private 저장소·현재 플랜에서 추가 설정 없이 동작한다. 기록은 Actions 실행 이력과 summary에 남는다.

### 저장소별 prod 워크플로

`app`·`ai`에 각각 두면 순서와 조합을 사람이 세 곳에서 맞춰야 한다.

### 버전 파일 커밋 기반 GitOps

`versions.yml`을 PR로 바꾸면 배포되는 방식. 기록과 리뷰는 좋지만 배포마다 PR 한 단계가 늘어난다. 필요해지면 같은 워크플로에 push 트리거를 얹어 전환할 수 있다.

### GitHub Environment required reviewers

private 저장소에서는 Enterprise 플랜이 필요하다. 저장소를 public으로 바꾸면 인프라 코드가 노출된다.

## 5. Consequences

- prod에 무엇이 올라가 있는지는 Actions summary와 ECS task definition 이미지 태그로 확인한다.
- Backend 롤백은 Flyway 마이그레이션이 expand/contract를 지켰다는 전제가 필요하다.
- FE 산출물 버킷은 dev가 쓰고 prod가 읽는 계정 공용 자원이라 Terraform `bootstrap`이 관리한다.
