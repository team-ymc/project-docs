# Feature: FT-XXX <Feature Name>

> **역할:** 하나의 feature = 독립적으로 개발·검증할 수 있는 기능 덩어리.
> ID는 `FT-XXX`, 이름은 역량을 나타내는 명사구(예: 소셜 인증, 논문 등록·분석).
> feature는 "어떻게 만드는가(how)"가 아니라 "무엇을 할 수 있는가(what)"의 계약 문서다.

## 1. Overview

> **역할:** feature를 한눈에 식별하는 메타 정보.

- Summary: <One-sentence capability statement>
  > 이 feature가 제공하는 역량을 한 문장으로. "무엇을 할 수 있는가".
- Status: Draft
  > 진행 상태. `Not Started`(틀만) / `In Progress`(작성 중) / `Finished`(계약 확정) / `Deprecated`. feature-spec.md 레지스트리와 동기화한다.
- Tracking: -
  > 관련 이슈·티켓·기술설계(ADR) 문서 링크. 없으면 `-`.
- Depends on: <FT-XXX Feature Name or ->
  > 이 feature가 동작하려면 먼저 있어야 하는 다른 feature. 없으면 `-`.

## 2. Feature Boundary

> **역할:** 이 feature가 무엇을 포함/제외하는지 경계를 못 박는다. 스코프 분쟁을 막는 장치.

### TASK In Scope

> **역할:** 이 feature가 책임지는 역량 목록. 상단 `-` 불릿은 "무엇을"의 요약, 아래 트랙은 "어떻게"의 작업.

- <Capability included in this feature>
#### DRAFT 
  > **역할:** 생각의 흐름을 막 적는 스크래치(구현 절차·아이디어 포함). 스프린트 시작 시 확정된 것만 FE/BE/AI로 distill한다.
#### FE
#### BE
#### AI

### TASK Out of Scope

> **역할:** 의도적으로 제외한 것. "안 한다"를 명시해 오해를 막는다. 다른 feature가 맡는 건 `→ FT-XXX`로 표기.

- <Capability excluded from this feature>
#### DRAFT 
  > **역할:** 제외 사유·이연 항목 스크래치.
#### FE

#### BE

#### AI


## 3. Userflow References

> **역할:** 이 feature가 직접 구현하는 Userflow를 추적한다. UF가 아직 없으면 행을 `TBD`로 둔다(feature 선행).

| Userflow | Relation |
|---|---|
| UF-XXX <Flow Name> | 이 Feature가 직접 구현하는 flow |

## 4. Stories

> **역할:** feature를 검증 가능한 단위로 쪼갠 것. 각 Story는 하나의 사용자/시스템 목표를 다룬다.
> `USER` = 사용자 행동(클릭·입력)이 트리거.<br> `SYSTEM` = 사용자 행동 없이 자동/비동기로 도는 로직.

### Story 1. <Story name> `MVP`

> 제목 옆 `MVP` 태그 = MVP 범위에 포함되는 Story.

- Story: <User goal>
  > 누가·무엇을·왜(목표). 보통 "~로서, ~하고 싶다" / SYSTEM은 "시스템으로서, ~한다".
- type: USER/SYSTEM
  > 트리거가 사용자면 USER, 시스템 자동/비동기면 SYSTEM.
- Source Userflows: <UF-XXX Step N, UF-YYY Step N-M>
  > 이 Story의 근거가 되는 UF Step. UF 없으면 `TBD`.
- Acceptance Criteria:
  - <Observable behavior or state>
  > 이 Story가 "완료됐다"고 인정할 **관찰 가능한 조건**. how가 아니라 what. 그대로 테스트/QA 케이스가 된다.
  > 스코프 결정(예: "MVP 중복확인은 파일명 1:1")도 여기에 박는다.
- Depends on: -
  > 선행 Story 또는 feature. 없으면 `-`.
 

## 5. Done Criteria

> **역할:** feature **전체**가 끝났다고 볼 조건. <br>보통 핵심 Story들의 Acceptance가 모두 충족되면 만족된다. <br>(Acceptance는 Story당, Done Criteria는 feature당 1개)

- <Feature-level completion condition>

## 6. Open Questions

> **역할:** 아직 결정하지 못한 기술/제품 질문을 격리하는 곳. <br>구현 선택(DB·전송방식·큐 등)이나 미정 스코프를 여기 모아 본문 계약(Story/Acceptance)을 깨끗하게 유지한다.
> 결정되면 `> **Resolved** — ...`로 내려 결론을 남기고, 스코프 결정은 Acceptance로 승격한다.

- <결정이 필요한 열린 질문>
