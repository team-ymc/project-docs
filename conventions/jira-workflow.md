# Jira Workflow

> **역할:** Git 작업과 Jira 이슈를 자동으로 연결하는 규칙과 그 동작 원리.
> 브랜치·커밋·PR의 **형식**은 `conventions/git-convention.md`에서 정의한다. 이 문서는 그 형식이 Jira에서 **어떻게 동작하는지**를 설명한다.

## 목적

브랜치 생성·커밋·Pull Request 병합 시 Jira 이슈 상태가 자동으로 바뀌도록 한다.
이 규칙을 지키면 작업 상태를 Jira에서 수동으로 바꿀 필요가 없고, Git 이력과 Jira 이슈를 함께 추적할 수 있다.

- Jira 프로젝트: `여행에서만남을추구하면안되는걸까`
- 프로젝트 키: `YMC`

## Jira 이슈 키를 넣는 이유

Jira Automation은 Git 작업에 포함된 **Jira 이슈 키를 기준으로 어떤 이슈를 변경할지 판단**한다.
`YMC-12` 같은 키가 없으면 Jira는 그 브랜치·커밋·PR이 어떤 이슈와 관련된 작업인지 알 수 없다.

| 위치 | 넣는 이유 | 주 목적 |
|---|---|---|
| 브랜치명 | 작업을 시작했음을 Jira가 감지해 이슈를 진행 중으로 이동 | **작업 시작** |
| 커밋 메시지 | 커밋 이력과 이슈를 연결, 커밋 기준 자동화 동작 | 이력 연결 |
| PR 제목/본문 | 병합 시 이슈를 `Dev Done` / `Completely Done`으로 이동 | **작업 완료** |

### 시작 신호 — 브랜치명

```
YMC-12-login
```

위 브랜치를 만들면 Jira는 `YMC-12` 작업이 시작됐다고 판단해 이슈를 진행 중으로 옮긴다.

### 완료 신호 — PR 제목/본문

```
[YMC-12] 로그인 기능 구현
```

이 PR이 `dev`로 병합되면 `YMC-12`를 `Dev Done`으로, `main`으로 병합되면 `Completely Done`으로 옮긴다.

> `dev -> main` PR은 브랜치명이 `dev`/`main`만 남아 키가 없다. 이 경우 PR 제목/본문에 관련 이슈 키를 **반드시** 적는다.

## 브랜치별 자동 상태 전환

### dev 브랜치로 병합

```
YMC-12-login -> dev
# 결과: YMC-12 -> Dev Done
```

### main 브랜치로 병합

```
dev -> main
# 결과: YMC-12 -> Completely Done   (PR 제목/본문에 YMC-12 포함 시)
```

## 현재 Jira Automation 규칙

> 자동화는 **2층**으로 동작한다.
> **(1) Git 이벤트가 Task를 움직이고 → (2) 자식 Task 상태가 모이면 부모 Story가 따라 움직인다.**
> 그래서 작업은 **Task 단위로만**(브랜치 키 = Task) 하면 되고, Story 완료는 알아서 굴러간다.

### Task 레벨 — Git 이벤트 기반

| 자동화 규칙 | 트리거 | 동작 |
|---|---|---|
| 진행 중 전환 → 시작일 기록 | 이슈가 진행 중으로 전환됨 | 시작 날짜 필드를 현재 날짜로 설정 |
| 커밋 → 진행 중 | 커밋 메시지에 이슈 키 포함 | 이슈를 진행 중으로 이동 |
| 브랜치 생성 → 진행 중 | 브랜치명에 이슈 키 포함 | 이슈를 진행 중으로 이동 |
| PR → dev 병합 | 대상 브랜치가 `dev` | 이슈를 `Dev Done`으로 이동 |
| PR → main 병합 | 대상 브랜치가 `main` | 이슈를 `Completely Done`으로 이동 |

### Story 레벨 — Task 롤업

| 자동화 규칙 | 트리거 | 동작 |
|---|---|---|
| Task 전부 Dev Done → Story Dev Done | 연결된 모든 Task가 `Dev Done` 이상 | 부모 Story를 `Dev Done`으로 이동 |
| Task 전부 Completely Done → Story Completely Done | 연결된 모든 Task가 `Completely Done` 이상 | 부모 Story를 `Completely Done`으로 이동 |

> **핵심:** Story는 자식 Task가 **전부** 목표 상태에 도달하면 자동 완료.

> 위 표는 현재 설정의 스냅샷이다. Jira에서 자동화 규칙을 바꾸면 이 표도 같은 PR에서 갱신한다.