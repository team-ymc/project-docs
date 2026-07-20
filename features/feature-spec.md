# Feature Spec

## 1. Product Brief

### Product Name

Paper Teacher

### Product Definition

논문을 혼자 읽기 어렵게 만드는 부분을 AI가 옆에서 같이 풀어주는 논문 학습 플랫폼. 최종적으로는 학습형 E-Book 엔진을 지향하며, MVP에서는 동일한 학습 문제를 가진 더 작고 정형화된 콘텐츠인 **논문**을 첫 검증 대상으로 한다.

## 2. Feature Brief

### Feature Goal

Paper Teacher의 제품 역량을 독립적으로 개발·검증 가능한 feature 단위로 분해한다. 각 feature는 관찰 가능한 동작(Story/Acceptance)을 계약으로 삼고, 구현 절차는 트랙(FE/BE/AI)과 Open Questions로 분리한다.

### Source Of Truth

- Userflow: `userflows/userflow-spec.md`
- Wireframe: `wireframes/wireframe-spec.md`
- Feature 양식: `features/features_format.md`

### Authoring Rule

- UF가 있는 feature는 해당 UF Step을 `Source Userflows`로 추적해 작성한다.
- UF가 아직 없는 feature는 `Not Started`로 틀만 두고 `Source Userflows: TBD`로 둔다. (feature 선행, UF는 추후 보강)

## 3. Feature Registry

| Feature ID | Feature Name | Depends on | Source Userflows | Feature Status | Feature Spec File |
|---|---|---|---|---|---|
| FT-001 | 소셜 인증 | - | UF-001 | In Progress | features/FT-001-소셜-인증.md |
| FT-002 | 서재 | FT-001 | UF-002 | In Progress | features/FT-002-서재.md |
| FT-003 | 논문 등록·분석 | FT-002 | UF-002 | In Progress | features/FT-003-논문-등록-분석.md |
| FT-004 | 논문 학습 뷰어 | FT-002, FT-003 | UF-003, UF-004, UF-005 | In Progress | features/FT-004-논문-학습-뷰어.md |
| FT-005 | 전체 번역 | FT-004 | UF-004 | In Progress | features/FT-005-전체-번역.md |
| FT-006 | 인라인 액션 | FT-004 | UF-005 | In Progress | features/FT-006-인라인-액션.md |
| FT-007 | AI 튜터 채팅 | FT-004 | UF-003 | In Progress | features/FT-007-ai-튜터-채팅.md |
| FT-008 | 위키 생성 | FT-003 | TBD | Not Started | features/FT-008-위키-생성.md |
| FT-009 | 구조 맵 | FT-003 | TBD | Not Started | features/FT-009-구조-맵.md |
| FT-010 | 학습 기록 저장 | FT-001, FT-004 | TBD | Not Started | features/FT-010-학습-기록.md |

## 4. Feature Dependency Map

```text
FT-001 소셜 인증
  └→ FT-002 서재
        └→ FT-003 논문 등록·분석 (구조 추출)
              ├→ FT-004 논문 학습 뷰어 (공통 캔버스)
              │     ├→ FT-005 전체 번역
              │     ├→ FT-006 인라인 액션
              │     └→ FT-007 AI 튜터 채팅
              ├→ FT-008 위키 생성        → (FT-004 / FT-007 참조)
              └→ FT-009 구조 맵          → (FT-004 / FT-007 참조)

FT-001 + FT-004 → FT-010 학습 기록 저장
```

## 5. Status Legend

| Status | 의미 |
|---|---|
| Not Started | 틀만 등록(레지스트리 행 + skeleton). 내용 TBD |
| In Progress | UF 기반으로 Story/Acceptance 작성 중 |
| Finished | feature 계약 확정, 스프린트 진입 가능 |
| Deprecated | 더 이상 진행하지 않음 |
