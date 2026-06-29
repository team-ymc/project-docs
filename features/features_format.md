# Feature: FT-XXX <Feature Name>

> **역할:** 하나의 feature = 독립적으로 개발·검증할 수 있는 역량 덩어리.
> ID는 `FT-XXX`, 이름은 역량을 나타내는 명사구(예: 소셜 인증, 논문 등록·분석).
> feature는 "무엇을 할 수 있는가(what)"의 **SSOT 계약 문서**다.

## 1. Overview

> **역할:** feature를 한눈에 식별하는 메타 정보.

- Summary: <한 문장 역량 단언 — "무엇을 할 수 있는가">
- Status: Draft
  > `Not Started`(틀만) / `In Progress`(작성 중) / `Finished`(계약 확정, 스프린트 진입 가능) / `Deprecated`. feature-spec.md 레지스트리와 동기화.
- Tracking: -
  > "어떻게"를 정의한 문서 링크. 관련 이슈·티켓·design doc(DD-XXX)·ADR·contracts. 없으면 `-`.
- Depends on: <FT-XXX Feature Name 또는 ->
  > 이 feature가 동작하려면 먼저 있어야 하는 다른 feature.

## 2. Feature Boundary

> **역할:** 이 feature가 무엇을 포함/제외하는지 경계를 못 박는다. 스코프 분쟁을 막는 장치.

### In Scope

> 이 feature가 책임지는 **역량**(무엇을) 목록. 구현 task는 여기 적지 않는다 — 티켓 몫.

- <포함하는 역량>

### Out of Scope

> 의도적으로 제외한 것. "안 한다"를 명시해 오해를 막는다.
> - 다른 feature가 맡으면 `→ FT-XXX`
> - 지금은 안 하고 나중에 할 거면 `→ 후속(post-MVP)`

- <제외 역량> → FT-XXX / 후속(post-MVP)

## 3. Userflow References

> **역할:** 이 feature가 직접 구현하는 Userflow를 추적한다. UF가 아직 없으면 행을 `TBD`로 둔다.

| Userflow | Relation |
|---|---|
| UF-XXX <Flow Name> | 이 Feature가 직접 구현하는 flow |

## 4. Stories

> **역할:** feature를 검증 가능한 단위로 쪼갠 것. 각 Story는 하나의 사용자/시스템 목표를 다룬다.
> - `USER` = 사용자 행동(클릭·입력)이 트리거 → **"사용자는 ~할 수 있다"**
> - `SYSTEM` = 사용자 행동 없이 자동/비동기로 도는 로직 → **"시스템은 ~한다"**

### Story 1. <사용자는 ~할 수 있다 / 시스템은 ~한다> `MVP`

> 제목 옆 `MVP` 태그 = MVP 범위에 포함되는 Story.

- type: USER / SYSTEM
- Source Userflows: <UF-XXX Step N, UF-YYY Step N-M>
  > 이 Story의 근거가 되는 UF Step. UF 없으면 `TBD`.
- Acceptance Criteria:
  - <관찰 가능한 동작·상태·룰>
  > 이 Story가 "완료됐다"고 인정할 **관찰 가능한 조건**으로 테스트/QA 케이스가 될 수 있다.
  > 스코프 결정(예: "MVP는 Google만", "중복확인은 파일명 1:1")도 여기에 박는다.
  > **seam 규칙:** 내부 seam(client → 우리 BE)은 criteria에 둔다. 외부 seam(우리 BE → 제3자 IdP/API)은 빼고 contracts로 보낸다. 단 외부 연동이 강제하는 *우리 도메인 룰*(계정 dedup 등)은 criteria에 남긴다.
- Depends on: -
  > 선행 Story 또는 feature. 없으면 `-`.

## 5. Done Criteria

> **역할:** feature **전체**가 끝났다고 볼 조건. <br>보통 핵심 Story들의 Acceptance가 모두 충족되면 만족된다. <br>(Acceptance는 Story당, Done Criteria는 feature당 1개)

- <feature 레벨 완료 조건>

## 6. Open Questions

> **역할:** 아직 결정하지 못한 기술/제품 질문을 격리하는 곳. 구현 선택(DB·전송방식·큐 등)이나 미정 스코프를 여기 모아 본문 계약(Story/Acceptance)을 깨끗하게 유지한다.
> 결정되면 `> **Resolved** — ...`로 내려 결론을 남기고, 스코프 결정은 Acceptance로 승격한다.
> **Open Question 밀도가 높으면 = 이 feature 뒷단이 복잡하다는 신호.** design doc(DD-XXX)을 떠서 거기서 풀고, 결론을 ADR/contracts로 내린 뒤 Tracking에 링크한다.

- <결정이 필요한 열린 질문>
