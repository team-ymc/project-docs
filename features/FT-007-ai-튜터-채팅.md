# Feature: FT-007 AI 튜터 채팅

## 1. Overview

- Summary: 학습 페이지의 AI 채팅에서 사용자가 논문에 대해 질문하면 스트리밍으로 답하고, 능동형 티칭 모드 흐름까지 포함한다.
- Status: Draft
- Tracking: -
- Depends on: FT-004 논문 학습 뷰어

## 2. Feature Boundary

### TASK In Scope

- AI 질문 입력 / 전송 (WF-012)
- 응답 대기 / 스트리밍 생성 / 완료 (WF-013~015)
- 후속 질문 (대화 이어가기)
- 능동형 티칭 모드 플로우 (시스템 주도 학습 진행) — 통합
- 논문 컨텍스트 기반 응답 (현재 섹션 / 위키 연계)

#### DRAFT
  - AI 패널 입력창에 질문 작성(WF-012) → 전송 → 대화 영역 표시 + 대기(WF-013)
  - 응답 생성 중 일부 표시(WF-014) → 완료 말풍선(WF-015)
  - 같은 패널에서 후속 질문 이어가기
  - 티칭 모드: 시스템이 학습 흐름을 주도(질문 유도/단계 진행) — 상세 미정, 커지면 별도 feature 분리
  - 응답 컨텍스트로 현재 섹션/위키(FT-008)/구조 맵(FT-009) 활용 가능
  - 대화의 영구 저장 경계는 FT-010과 분담
#### FE

#### BE

#### AI

### TASK Out of Scope

- 응답 생성 중단 / 취소 — UF-003 Separate Flow Candidate
- 응답 실패 / 재시도
- 인라인에서의 AI 질문 진입 트리거 — FT-006 (질문 처리는 본 feature)
- 대화 히스토리 영구 저장·복원 — FT-010 학습 기록

#### DRAFT
  - 스트리밍 중단/취소 인터랙션
  - 실패 상태 표시·재시도
  - 장기 대화 히스토리 저장/이어보기 → FT-010
#### FE

#### BE

#### AI

## 3. Userflow References

| Userflow | Relation |
|---|---|
| UF-003 AI 채팅 창에서 논문 질문하기 | 이 Feature가 직접 구현하는 flow |
| TBD (티칭 모드) | UF 미작성 — feature 선행, 추후 userflow 등록 시 연결 |

## 4. Stories

### Story 1. 논문에 대해 질문하기 `MVP`

- Story: 로그인 사용자로서, 논문을 읽다가 궁금한 점을 AI 채팅에 질문하고 싶다.
- type: USER
- Source Userflows: UF-003 Step 1-3
- Acceptance Criteria:
  - AI 패널 입력창에 질문을 작성(WF-012)하고 전송하면 질문이 대화 영역에 표시되고 대기 상태(WF-013)가 된다.
- Depends on: FT-004 Story 1

### Story 2. BE → AI 서버: 튜터 응답 스트리밍 요청 `MVP`

- Story: 시스템으로서, 사용자 질문을 논문 컨텍스트와 함께 AI 서버에 보내 답변을 스트리밍으로 받는다.
- type: SYSTEM
- Source Userflows: UF-003 Step 4-5
- Acceptance Criteria:
  - BE가 질문 + 논문 컨텍스트를 AI 서버에 요청하고 응답을 스트리밍으로 수신한다. (BE → AI 서버)
  - 대기 상태에서 생성이 시작되면 생성 중 일부(WF-014)가 표시되고, 완료 시 답변 말풍선(WF-015)이 표시된다.
  - 응답은 해당 논문 컨텍스트를 반영한다.
- Depends on: FT-004 Story 1

### Story 3. 후속 질문 이어가기 `MVP`

- Story: 로그인 사용자로서, 완성된 답변에 이어 같은 대화에서 추가 질문을 하고 싶다.
- type: USER
- Source Userflows: UF-003 Step 5
- Acceptance Criteria:
  - 완료된 답변 이후 같은 패널에서 후속 질문을 보내면 이전 맥락을 이은 응답이 생성된다.
- Depends on: Story 2

> **Deferred (UF 미작성)** — 능동형 티칭 모드: 시스템이 학습 흐름을 주도(질문 유도/단계 진행). AI 튜터에 통합 예정이며, UF 작성 후 Story로 추가. 규모 커지면 별도 feature 분리.

## 5. Done Criteria

- 질문 입력 → 전송 → 스트리밍 응답 → 완료 → 후속 질문까지 동작한다.
- 응답이 해당 논문 컨텍스트를 반영한다.

## 6. Open Questions

- 응답 컨텍스트 주입 범위: 논문 전체 vs 현재 섹션 vs 위키(FT-008)/구조 맵(FT-009)
- 티칭 모드의 구체 플로우(시나리오·진행 단위) 미정 — 규모 커지면 별도 feature로 분리
- 대화 저장 소유 경계: 세션 내 임시(FT-007) vs 영구 기록(FT-010)
