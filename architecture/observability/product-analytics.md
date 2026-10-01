# Product Analytics

사용자 행동을 무엇으로 측정하고 어떤 이벤트로 수집하는지의 SSOT다. 수집 도구는 Mixpanel이고 FE가 브라우저에서 직접 전송한다.

## 1. 목적

마케팅을 시작하기 전에 아래 세 질문에 답할 수 있어야 한다.

- 유입된 사용자 중 얼마가 가입하는가.
- 가입한 사용자가 어디까지 써보는가.
- 채널별로 가입 한 건에 광고비가 얼마 드는가.

이 질문에 쓰이지 않는 이벤트는 추가하지 않는다.

## 2. 지표

| 지표 | 정의 |
|---|---|
| DAU | 하루 동안 `viewer_opened`를 한 번 이상 보낸 사용자 수 |
| MAU | 최근 30일 동안 `viewer_opened`를 한 번 이상 보낸 사용자 수 |
| 가입 전환율 | 랜딩 페이지를 조회한 방문자 중 `signup_completed`에 도달한 비율 |
| 가입당 비용 | 채널에 쓴 광고비 ÷ 그 채널의 `utm_source`로 들어와 `signup_completed`에 도달한 사용자 수 |

- 활성의 기준은 접속이 아니라 뷰어 진입이다.
- 광고비는 Mixpanel에 없다. 광고 플랫폼의 지출액으로 직접 계산한다.
- 가입자 수의 기준 값은 DB다. Mixpanel 수치는 광고 차단으로 누락될 수 있어 추세와 비율을 보는 데 쓴다.

## 3. 이벤트

| 이벤트 | 발생 시점 | 속성 |
|---|---|---|
| `signup_completed` | 로그인 완료 신호가 신규 가입을 알렸을 때 | 없음 |
| `paper_uploaded` | 논문 업로드 완료 요청이 성공했을 때 | `paper_id` |
| `viewer_opened` | 학습 화면에 논문 본문이 표시됐을 때. 진입마다 한 번 | `paper_id` |
| `prerequisite_toggled` | 사용자가 선행지식 표시 토글을 눌렀을 때 | `paper_id`, `enabled` |
| `prerequisite_clicked` | 사용자가 선행지식 단어를 눌러 설명을 열었을 때 | `paper_id` |
| `knowledge_graph_opened` | 지식 그래프 화면에 진입했을 때 | `paper_id` |
| `translation_requested` | 전체 번역 표시를 꺼진 상태에서 켰을 때, 또는 인라인 번역을 요청했을 때 | `paper_id`, `type` |
| `chat_question_sent` | AI 튜터에게 질문을 보냈을 때 | `paper_id` |

- `enabled`는 토글 후 상태다. 저장된 설정이 복원되어 켜진 경우에는 보내지 않는다.
- `type`은 `full` 또는 `inline`이다. 전체 번역의 표시 위치만 바꾼 경우에는 보내지 않는다.
- 페이지 조회는 화면 이동마다 SDK가 보낸다. 랜딩 조회는 경로가 `/`인 페이지 조회로 본다.
- 자동 수집과 세션 녹화는 사용하지 않는다.

## 4. 사용자 식별

- 로그인 전에는 SDK가 브라우저에 저장한 익명 ID로 수집한다.
- 로그인과 세션 복원 시 [FE↔BE 계약](../../contracts/frontend-backend/openapi.yaml)의 `AuthUser.userId`로 식별한다. 로그인 전 기록이 같은 사용자로 이어진다.
- 로그아웃과 세션 만료 시 식별을 초기화한다.
- 신규 가입 여부는 로그인 콜백이 FE로 복귀할 때 전달한다. 형식은 같은 계약의 FT-001 주석에 있다.

## 5. 보내지 않는 것

- 이메일, 표시 이름
- 논문 제목, 파일 이름
- 질문과 답변 본문, 선택한 문장, 번역 결과
- 선행지식 단어와 설명

사용자 프로필 속성은 저장하지 않는다.

## 6. 환경

| 접속 주소 | 전송 대상 |
|---|---|
| prod | 운영 프로젝트 |
| dev | 테스트 프로젝트 |
| 그 외 | 전송하지 않음 |

prod는 dev에서 빌드한 FE 산출물을 그대로 쓰므로 빌드 시점 값으로 환경을 나눌 수 없다. 실행 시 접속 주소로 고른다.

분석 전송이 실패하거나 차단되어도 화면 동작에 영향을 주지 않는다.

## 7. UTM 규칙

마케팅 링크에는 UTM을 붙인다. SDK가 주소에서 읽어 이후 이벤트에 붙이므로 코드 작업은 없다.

- 값은 모두 소문자로 쓴다. 대소문자가 다르면 다른 채널로 집계된다.
- 공백 대신 `_`를 쓴다.

| 파라미터 | 의미 | 값 예시 |
|---|---|---|
| `utm_source` | 유입 채널 | `meta`, `google`, 커뮤니티 이름 |
| `utm_medium` | 유입 방식 | `paid_social`, `cpc`, `community` |
| `utm_campaign` | 캠페인 | `launch_2026_10` |
| `utm_content` | 광고 소재 구분 | `video_a`, `image_b` |

```text
https://papertutor.co.kr/?utm_source=meta&utm_medium=paid_social&utm_campaign=launch_2026_10&utm_content=video_a
```

UTM은 Urchin Tracking Module의 약자다. 방문자가 어느 채널에서 왔는지를 링크 주소 뒤에 적어 두는 표준 표기이고, 구글 애널리틱스의 전신인 Urchin에서 유래했다. 주소에 붙어도 사용자가 보는 화면은 달라지지 않는다.

## 8. 리포트

| 리포트 | 구성 |
|---|---|
| 가입 퍼널 | 랜딩 조회 → `signup_completed`. `utm_source`로 나눠 본다 |
| 활성화 퍼널 | `signup_completed` → `paper_uploaded` → `viewer_opened` |
| 기능별 사용률 | `viewer_opened` 사용자 중 각 기능 이벤트를 보낸 비율 |

뷰어 진입 이후의 기능은 순서가 없으므로 한 퍼널로 잇지 않는다.

## Scope

광고 플랫폼 전환 추적은 범위 밖이다. 메타와 구글에 가입을 알리는 추적 코드는 광고비를 본격적으로 쓰기로 할 때 별도로 결정한다.
