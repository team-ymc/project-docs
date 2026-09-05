# Design v2

Paper Teacher v2의 화면 디자인과 디자인 시스템 산출물 인덱스다. 폰트·아이콘·수식 렌더러를 `vendor/`에 로컬로 두어 네트워크 없이 열린다.

## Screens

| 화면 | 파일 |
|---|---|
| Landing Page | [Paper Landing Page](Paper%20Landing%20Page.dc.html) |
| Bookshelf Page | [Paper Bookshelf Page](Paper%20Bookshelf%20Page.dc.html) |
| Study Page | [Paper Study Page](Paper%20Study%20Page.dc.html) |

## Screen States — 플랜·사용량 (FT-011 / YMC-348)

기존 화면 위에 플랜·사용량 표시와 한도 소진 상태를 얹은 아트보드다. 기존 목업과 같은 방식(`x-import` 디자인 시스템 컴포넌트 + Phosphor `ph ph-*` 아이콘)으로 작성해 FE가 컴포넌트·아이콘 이름을 그대로 찾을 수 있다. 트윅 `plan`(FREE/PRO), `aiUsed`, `paperUsed`는 `data-props` 기본값으로 들어 있다.

| 상태 | 파일 | 요약 |
|---|---|---|
| 서재 · 플랜 배지와 사용량 메뉴 | [Paper Bookshelf Page - Plan Usage](Paper%20Bookshelf%20Page%20-%20Plan%20Usage.dc.html) | 글로벌 상단 바의 프로필 버튼 왼쪽에 `FREE`/`PRO` 배지를 상시 표시한다. 프로필 메뉴(프로필 / 사용량 / 설정 / 로그아웃, 우측 정렬)에서 `< 사용량`을 누르면 왼쪽으로 패널이 펼쳐지며 AI 질의·문서 등록 미터와 초기화 시각을 보여준다. Pro도 Free와 동일하게 `사용 / 상한` 횟수 미터로 표시한다(베타 정책: 질의 1,000·문서 100 — FT-011). 미터의 `무제한`(사선 무늬 바) 표시는 계약의 `UNLIMITED` 모드 전용으로, 현재 정책값에서는 나타나지 않는다. 아트보드의 PRO 트윅은 무제한 표시 시연용으로 남겨둔 것이다. |
| 학습 · AI 질의 한도 소진 | [Paper Study Page - Chat Limit](Paper%20Study%20Page%20-%20Chat%20Limit.dc.html) | 입력창 모양은 유지한 채 placeholder 자리에 흐린 글씨로 `금월 사용량 소진 · 9월 1일 초기화`를 적고, 보내기 버튼을 비활성화한다. 빠른 질문 칩은 숨긴다. |
| 서재 · 문서 등록 한도 소진 | [Paper Bookshelf Page - Upload Limit](Paper%20Bookshelf%20Page%20-%20Upload%20Limit.dc.html) | 업로드 버튼 왼쪽에 `Free 플랜 월 3회를 모두 소진했습니다 · 9월 1일 초기화` 안내 박스를 두고 업로드 버튼을 비활성화한다. |
| 서재 · 행 메뉴(다운로드 / 이름 변경 / 삭제) | [Paper Bookshelf Page - Row Actions](Paper%20Bookshelf%20Page%20-%20Row%20Actions.dc.html) | 목록 행 오른쪽 끝에 `⋯` 버튼을 상시 두고, 누르면 원본 PDF 다운로드 / 이름 변경 / 삭제 드롭다운이 열린다(분석 중·실패 행은 다운로드 비활성). 이름 변경은 제목을 그 자리에서 입력창(확장자 `.pdf` 고정)으로 바꾸고 Enter 저장·Esc 취소. 삭제는 확인 다이얼로그를 거친다. 격자 카드는 같은 메뉴를 썸네일 우상단에 둔다(YMC-369). |

데이터는 `GET /api/me/plan`(`contracts/frontend-backend/openapi.yaml`)을 따른다. `plan` → 배지, `planExpiresAt` → Pro 만료일, `usage.aiQuery`·`usage.paperRegistration` → 각 미터(`MONTHLY`는 `used / limit`와 `resetAt`, `UNLIMITED`는 `무제한`). 베타 정책은 Free·Pro 모두 `MONTHLY`다. 미터는 우선 횟수로 제공하고 % 표시 전환은 추후 검토한다. `used`는 확정과 진행 중 예약의 합이다. 한도 소진 상태는 `remaining === 0`일 때 진입하며, 진입 후 BE가 429(`CHAT_USAGE_LIMIT_EXCEEDED` / `PAPER_USAGE_LIMIT_EXCEEDED`)를 반환한 경우에도 같은 상태로 전환한다.

### 이 상태를 위해 디자인 시스템에 추가한 것

| 추가 | 위치 | 내용 |
|---|---|---|
| `UsageMeter` | `components/feedback/UsageMeter.jsx` | props `label`, `used`, `limit`, `unlimited`, `resetLabel`, `compact`. MONTHLY는 `used / limit`와 바, UNLIMITED는 `무제한`과 사선 무늬 바(횟수 미노출). 남은 횟수 0이면 바가 `--color-danger`. |
| `Badge` 톤 | `components/feedback/Badge.jsx` | `pro`(Navy Ink), `neutralOnDark`·`proOnDark`(월넛 상단 바용). `_adherence.oxlintrc.json`의 tone enum도 갱신. |
| 토큰 | `tokens/colors.css` | `--brass-on-dark: #E8C98F`, `--color-accent-brass-on-dark` — 다크 바 위 Pro 배지 색. |
| 아이콘 | Phosphor | 새로 쓰는 이름: `user`, `seal-check`, `warning-circle`, `books`, `file-text`, `magnifying-glass`, `squares-four`, `arrow-left`. FE `icons.ts`에 등록 필요. |

## Design System

- [Paper Teacher Design System](_ds/paper-teacher-design-system-1a53a7a7-d059-48b5-b12a-0094ed1cc07b/readme.md)
- `support.js` — 공통 support 파일
- `vendor/` — 로컬 폰트(Noto Serif KR, Pretendard), Phosphor 아이콘, KaTeX, React, Cytoscape

## Related Documentation

- 디자인 버전 인덱스: [Design](../README.md)
- 화면 구조와 상태: [Wireframes](../../wireframes/README.md)
- 사용자 행동 흐름: [Userflows](../../userflows/README.md)
- 플랜·사용량 제한 기능: [FT-011](../../features/FT-011-플랜-사용량-제한.md)
