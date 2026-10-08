# 체험 페이지

광고로 유입된 방문자가 가입 없이 Paper Teacher 뷰어를 바로 써볼 수 있도록, 주제별로 미리 준비한 논문 한 편씩을 고르는 데스크톱 전용 페이지 시안이다.

## 화면

- [Paper Trial Page - Marketing](Paper%20Trial%20Page%20-%20Marketing.html)
- 한 화면(1440×900) 안에 상단 바, 헤드라인과 `내 논문 업로드하기` 버튼, 주제별 논문 패널 다섯 장이 모두 들어가며 스크롤이 없다.
- 패널은 어두운 분야 명판(사랑 · AI · 반도체 · 우주 · 수면)과 그 아래 논문 첫 페이지, 영문 제목, 연도·학술지로 구성한다. 마우스를 올리면 그 패널이 넓어지며 한국어 한 줄 소개와 `읽어보기`가 나타난다. 패널을 누르면 체험용 뷰어가 열린다.
- `내 논문 업로드하기`, 상단 바의 `로그인`, 체험 뷰어의 AI 질문 입력은 모두 같은 모달(`가입하고 무료로 이용해보세요.`)을 띄운다. 정적 시안의 Google 버튼은 동작하지 않는다.
- 모바일 유입은 이 페이지로 보내지 않고 기존 마케팅 랜딩으로 보낸다. 반응형은 두지 않고 최소 폭 1024px만 잡았다.

## 체험 논문

| 분야 | 논문 | 출처 |
|---|---|---|
| 사랑 | Love-related Changes in the Brain: A Resting-state fMRI Study (Song 외, 2015) | Frontiers in Human Neuroscience, CC BY |
| AI | Attention Is All You Need (Vaswani 외, 2017) | arXiv 1706.03762 |
| 반도체 | Cramming More Components onto Integrated Circuits (Moore, 1965) | Electronics 재인쇄본. 공개 라이선스가 아니라 서비스 게재 전 확인 필요 |
| 우주 | Observation of Gravitational Waves from a Binary Black Hole Merger (LIGO·Virgo, 2016) | arXiv 1602.03837 |
| 수면 | Sleep Loss Causes Social Withdrawal and Loneliness (Ben Simon·Walker, 2018) | Nature Communications, CC BY |

`trial-assets/`의 이미지는 위 PDF의 첫 페이지를 가로 900px JPEG로 만든 것이다. 서비스 구현에서는 파서가 만든 첫 페이지 이미지로 대체한다.

## 체험 뷰어

- 별도 아트보드를 두지 않는다. 기존 학습 화면([Design v2 Study Page](../v2/Paper%20Study%20Page.dc.html), 선행지식은 [Design v3](../v3/README.md))을 그대로 쓰고, AI 질의만 막는다.
- 질문을 보내는 순간(Enter 또는 보내기) 체험 페이지와 같은 가입 모달(`가입하고 무료로 이용해보세요.` / `지금 보던 논문은 서재에서 이어서 읽을 수 있어요.`)이 뜬다. 입력 자체는 막지 않고, 모달을 닫으면 입력한 질문은 그대로 남는다.
- 본문·번역·선행지식·지식 그래프는 로그인 상태와 같다.

## 참고

- 체험 뷰어 상단 바의 플랜 배지·프로필 자리를 비로그인 상태에서 어떻게 둘지(비움 / 로그인 버튼)는 TBD.
- 체험 논문 데이터는 `contracts/frontend-backend/openapi.yaml` 0.14.0의 `/api/trial/papers/{paperId}/*` 세 경로(본문·지식 그래프·선행지식 설명)를 따른다. 목록 API는 없고 주제·소개 문구는 FE가 정적으로 갖는다.
