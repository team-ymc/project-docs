# 체험 페이지

광고로 유입된 방문자가 가입 없이 Paper Teacher 뷰어를 바로 써볼 수 있도록, 주제별로 미리 준비한 논문 아홉 편 가운데 한 편을 고르는 데스크톱 전용 페이지 시안이다.

## 화면

- [Paper Trial Page - Marketing](Paper%20Trial%20Page%20-%20Marketing.html)
- 한 화면(1440×900) 안에 상단 바, 헤드라인과 `내 논문 업로드하기` 버튼, 논문 영역이 모두 들어가며 스크롤이 없다.
- 논문 영역은 왼쪽 `오늘의 논문` 한 편과 오른쪽 나머지 여덟 편 목록으로 나뉜다.
  - `오늘의 논문`은 첫 페이지 이미지, 주제, 영문 제목, 한국어 한 줄 소개, 연도·출처, `읽어보기` 버튼을 바로 보여 준다. 마우스를 올리면 첫 페이지가 바로 선다.
  - 목록의 각 줄은 작은 첫 페이지 이미지, 주제, 영문 제목, 연도로 구성한다. 마우스를 올리면 줄이 밝아지며 화살표가 나타난다. 줄을 누르면 체험용 뷰어가 열린다.
- `내 논문 업로드하기`, 상단 바의 `로그인`, 체험 뷰어의 AI 질문 입력은 모두 같은 모달(`가입하고 무료로 이용해보세요.`)을 띄운다. 정적 시안의 Google 버튼은 동작하지 않는다.
- 모바일 유입은 이 페이지로 보내지 않고 기존 마케팅 랜딩으로 보낸다. 반응형은 두지 않고 최소 폭 1024px만 잡았다.

## 체험 논문

| 분야 | 논문 | 출처 |
|---|---|---|
| 게임 | Playing Atari with Deep Reinforcement Learning (Mnih 외, 2013) | arXiv 1312.5602 |
| 수면 | Why Dream? A Conjecture on Dreaming (Stoneham, 2009) | arXiv |
| 돈 | Bitcoin: A Peer-to-Peer Electronic Cash System (Nakamoto, 2008) | bitcoin.org |
| 미술 | Instance Normalization: The Missing Ingredient for Fast Stylization (Ulyanov 외, 2016) | arXiv 1607.08022 |
| 사랑 | Do LLMs Give Good Romantic Relationship Advice? A Study on User Satisfaction and Attitude Change (Manchanda 외, 2025) | TBD. PDF에 게재처 표기가 없다 |
| 음악 | Simple and Controllable Music Generation (Copet 외, 2023) | arXiv 2306.05284 |
| 언어 | Efficient Estimation of Word Representations in Vector Space (Mikolov 외, 2013) | arXiv 1301.3781 |
| 우주 | Dissolving the Fermi Paradox (Sandberg 외, 2018) | arXiv 1806.02404 |
| 행복 | Happiness Is Assortative in Online Social Networks (Bollen 외, 2011) | arXiv 1103.0784 |

`trial-assets/`의 이미지는 위 PDF의 첫 페이지를 가로 900px JPEG로 만든 것이다. 서비스 구현에서는 파서가 만든 첫 페이지 이미지로 대체한다.

## 체험 뷰어

- 별도 아트보드를 두지 않는다. 기존 학습 화면([Design v2 Study Page](../v2/Paper%20Study%20Page.dc.html), 선행지식은 [Design v3](../v3/README.md))을 그대로 쓰고, AI 질의만 막는다.
- 질문을 보내는 순간(Enter 또는 보내기) 체험 페이지와 같은 가입 모달(`가입하고 무료로 이용해보세요.` / `지금 보던 논문은 서재에서 이어서 읽을 수 있어요.`)이 뜬다. 입력 자체는 막지 않고, 모달을 닫으면 입력한 질문은 그대로 남는다.
- 본문·번역·선행지식·지식 그래프는 로그인 상태와 같다.

## 참고

- `오늘의 논문`은 아홉 편을 날짜별로 돌려 가며 세운다. 시안은 게임 논문을 세워 두었다.
- 체험 뷰어 상단 바의 플랜 배지·프로필 자리를 비로그인 상태에서 어떻게 둘지(비움 / 로그인 버튼)는 TBD.
- 체험 논문 데이터는 `contracts/frontend-backend/openapi.yaml` 0.14.0의 `/api/trial/papers/{paperId}/*` 세 경로(본문·지식 그래프·선행지식 설명)를 따른다. 목록 API는 없고 주제·소개 문구와 paperId는 FE가 호스트별 정적 설정(`fe/src/trial/trialPapers.ts`)으로 갖는다.
