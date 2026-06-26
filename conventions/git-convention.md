# Git Convention

> **역할:** 브랜치명·커밋 메시지·Pull Request의 **형식**을 정하는 팀 규칙.
> 이 형식이 Jira에서 일으키는 **자동 상태 전환·자동화 규칙의 동작**은 `conventions/jira-workflow.md`에서 확인한다. (이 문서는 형식만 다룬다)

## 1. 기본 원칙

- 브랜치명, 커밋 메시지, Pull Request 제목/본문 중 **최소 한 곳**에는 Jira 이슈 키(`YMC-XXX`)를 포함한다.
- 권장: 브랜치명 + 커밋 메시지 둘 다 키를 넣는다. (이력 추적이 가장 안정적)

## 2. 브랜치명 규칙

형식:

```
YMC-XXX-<short-description>
```

- 맨 앞에 Jira 이슈 키(`YMC-XXX`)를 둔다. **키는 필수.** (브랜치 생성 시 일어나는 자동 전환 → `jira-workflow.md`)

예시:

```
YMC-12-login
YMC-13-map-error
YMC-14-update-config
YMC-162-features-초안-작성
```

> **타입 prefix(`feat/`, …)를 붙이지 않는 이유:** `YMC-XXX` 키 자체가 이미 해당 이슈의 맥락을 담고 있어, 브랜치명에 type을 중복으로 넣지 않는다.

## 3. 커밋 타입(type) 정의

커밋 메시지의 `type`에 쓰는 어휘.

| type | 의미 |
|---|---|
| `feat` | 새 기능 |
| `fix` | 버그 수정 |
| `docs` | 문서 추가·수정 |
| `refactor` | 동작 변화 없는 구조 개선 |
| `chore` | 설정·빌드·잡일 (코드 동작 무관) |
| `test` | 테스트 추가·수정 |
| `style` | 포맷·세미콜론 등 비기능 변경 |

## 4. 커밋 메시지 규칙

형식:

```
[YMC-XXX] type(scope): subject
```

- `[YMC-XXX]`: 맨 앞에 대괄호로 Jira 이슈 키. (커밋 시 일어나는 자동 전환 → `jira-workflow.md`)
- `type`: 위 §3 타입.
- `(scope)`: 변경 범위(모듈·도메인). 예: `auth`, `features`, `map`. 범위가 모호하면 생략 가능하나, **기본은 명시**한다.
- `subject`: 무엇을 했는지 한 줄.

예시:

```
[YMC-12] feat(auth): 로그인 화면 구현
[YMC-13] fix(map): 지도 마커 좌표 오류 수정
[YMC-14] chore(config): 환경 변수 설정 정리
[YMC-162] docs(features): features 초안 작성
```

> 브랜치명에 이미 키가 있어도 커밋에 키를 함께 남긴다. <br> 브랜치는 머지 후 사라지지만 커밋 이력은 영구히 남아, 키가 커밋에 박혀 있어야 나중에도 이슈↔변경을 역추적할 수 있다.

## 5. Pull Request 규칙

- PR 제목 **또는** 본문에 Jira 이슈 키를 포함한다.
- ⚠️ `dev -> main` PR은 브랜치명에 키가 없으니 **제목/본문에 키를 꼭** 넣는다. (안 넣으면 자동 완료 전환이 조용히 누락됨)
- 머지 시 일어나는 자동 상태 전환(`Dev Done` / `Completely Done`)의 동작은 `jira-workflow.md` 참고.

PR 제목 예시:

```
[YMC-12] 로그인 기능 구현
[YMC-13] 지도 오류 수정
```

> PR 본문 양식은 `.github/pull_request_template.md`에 두면 PR 생성 시 자동으로 채워진다.
> ⚠️ 현재 템플릿은 **제안 초안**이며 팀 합의 전이다. 합의 후 이 문구를 제거한다.

## 6. 작업 흐름 예시

```bash
# 1. Jira에서 이슈 키 확인 → YMC-12

# 2. 브랜치 생성 (→ YMC-12 자동으로 '진행 중')
git checkout -b YMC-12-login

# 3. 커밋 (커밋 이력이 YMC-12에 연결)
git commit -m "[YMC-12] feat(auth): 로그인 화면 구현"

# 4. dev로 PR 머지 (→ YMC-12 'Dev Done')
#    YMC-12-login -> dev

# 5. dev로 main PR 머지 (→ YMC-12 'Completely Done')
#    dev -> main   (PR 제목/본문에 YMC-12 명시 필수)
```

> 위 주석의 상태 전환이 어떤 자동화 규칙으로 일어나는지(Task→Story 롤업 포함)는 `conventions/jira-workflow.md` 참고.
