# Wireframe Spec

## 1. Product Brief

### Product Name

Paper Teacher

### Product Definition

논문을 혼자 읽기 어렵게 만드는 부분을 AI가 옆에서 같이 풀어주는 논문 학습 플랫폼.

## 2. Wireframe Brief

### Platform

Responsive web

### Wireframe Goal

개발자가 Paper Teacher의 주요 화면 구조, 논문 읽기와 AI 설명이 함께 배치되는 방식, 사용자 학습 흐름, 그리고 화면 간 전환 관계를 구현 전에 명확히 이해할 수 있도록 한다.

### Fidelity

High fidelity

### Drawing Tool

Figma

## 3. Frame Registry

| Frame ID | Frame Name | Frame Category | Base Frame | Frame Status | Frame Spec File |
|---|---|---|---|---|---|
| WF-001 | 랜딩 페이지 - 비로그인 | Base | none | Finished | wireframes/frames/WF-001-랜딩-페이지-비로그인.md |
| WF-002 | 랜딩 페이지 - 로그인 | State | WF-001 | Finished | wireframes/frames/WF-002-랜딩-페이지-로그인.md |
| WF-003 | 회원가입 / 로그인 모달 | Overlay | WF-001 | Finished | wireframes/frames/WF-003-회원가입-로그인-모달.md |
| WF-004 | 서재 페이지 - 빈 상태 | Base | none | Finished | wireframes/frames/WF-004-서재-페이지-빈-상태.md |
| WF-005 | 논문 등록 - 모달 | Overlay | WF-004 | In Progress | wireframes/frames/WF-005-논문-등록-모달.md |
| WF-006 | 서재 페이지 - 논문 업로드 진행 상태 | Base | none | In Progress | wireframes/frames/WF-006-서재-페이지-논문-업로드-진행-상태.md |
| WF-007 | 서재 페이지 - 논문 업로드 완료 상태 | State | WF-006 | In Progress | wireframes/frames/WF-007-서재-페이지-논문-업로드-완료-상태.md |
| WF-008 | 논문 학습 페이지 - 초기 상태 | Base | none | In Progress | wireframes/frames/WF-008-논문-학습-페이지-초기-상태.md |
| WF-009 | 논문 학습 페이지 - 한글 번역 상태 | State | WF-008 | In Progress | wireframes/frames/WF-009-논문-학습-페이지-한글-번역-상태.md |
| WF-010 | 논문 학습 페이지 - 텍스트 선택 액션 상태 | State | WF-008 | In Progress | wireframes/frames/WF-010-논문-학습-페이지-텍스트-선택-액션-상태.md |
| WF-011 | 논문 학습 페이지 - 인라인 번역 결과 상태 | State | WF-010 | In Progress | wireframes/frames/WF-011-논문-학습-페이지-인라인-번역-결과-상태.md |
| WF-012 | 논문 학습 페이지 - AI 질문 입력 상태 | State | WF-008 | In Progress | wireframes/frames/WF-012-논문-학습-페이지-ai-질문-입력-상태.md |
| WF-013 | 논문 학습 페이지 - AI 응답 대기 상태 | State | WF-012 | In Progress | wireframes/frames/WF-013-논문-학습-페이지-ai-응답-대기-상태.md |
| WF-014 | 논문 학습 페이지 - AI 응답 생성 중 상태 | State | WF-013 | In Progress | wireframes/frames/WF-014-논문-학습-페이지-ai-응답-생성-중-상태.md |
| WF-015 | 논문 학습 페이지 - AI 응답 완료 상태 | State | WF-014 | In Progress | wireframes/frames/WF-015-논문-학습-페이지-ai-응답-완료-상태.md |
| WF-016 | 서재 페이지 - 논문 목록 기본 상태 | Base | none | In Progress | wireframes/frames/WF-016-서재-페이지-논문-목록-기본-상태.md |

## 4. Frame Relation Map

| From Frame | Trigger / Event | To Frame | What Changes |
|---|---|---|---|
| WF-001 | 비로그인 사용자가 로그인 / 회원가입 버튼을 선택 | WF-003 | 비로그인 랜딩 페이지 위에 회원가입 / 로그인 모달이 열린다. |
| WF-003 | 비로그인 사용자가 닫기 버튼을 선택 | WF-001 | 회원가입 / 로그인 모달이 닫히고 비로그인 랜딩 페이지가 다시 보인다. |
| WF-003 | 비로그인 사용자가 Google, Kakao, 또는 Naver 인증을 완료 | WF-004 | 인증 모달이 사라지고 서재 페이지의 빈 상태로 이동한다. |
| WF-002 | 서재에 등록된 논문이 없는 로그인 사용자가 내 서재 가기 버튼을 선택 | WF-004 | 로그인 랜딩 페이지에서 서재 페이지의 빈 상태로 이동한다. |
| WF-004 | 사용자가 논문 등록 버튼을 선택 | WF-005 | 서재 빈 상태 위에 논문 등록 모달이 열린다. |
| WF-005 | 사용자가 닫기 버튼을 선택 | WF-004 | 논문 등록 모달이 닫히고 서재 빈 상태가 다시 보인다. |
| WF-005 | 사용자가 PDF를 선택하고 업로드 버튼을 선택 | WF-006 | 논문 등록 모달이 사라지고 서재에 업로드 진행 중인 논문 행이 표시된다. |
| WF-006 | 논문 업로드와 분석이 진행 중 | WF-007 | 진행률 표시가 사라지고 논문 행이 완료 상태로 전환된다. |
| WF-016 | 사용자가 논문 목록에서 논문 행을 선택 | WF-008 | 서재 목록에서 선택한 논문의 학습 페이지 초기 상태로 이동한다. |
| WF-008 | 사용자가 번역 버튼을 선택 | WF-009 | 논문 원문 영역이 한글 번역 본문 상태로 전환된다. |
| WF-009 | 사용자가 번역 버튼을 다시 선택 | WF-008 | 한글 번역 본문이 원문 학습 페이지 초기 상태로 돌아간다. |
| WF-008 | 사용자가 논문 본문 텍스트를 드래그해 선택 | WF-010 | 선택된 본문 옆에 텍스트 선택 액션 창이 나타난다. |
| WF-010 | 사용자가 선택 액션 창에서 번역을 선택 | WF-011 | 선택 액션 창이 인라인 번역 결과 창으로 바뀐다. |
| WF-008 | 사용자가 AI 튜터 입력창에 질문을 작성 | WF-012 | AI 튜터 입력창에 작성 중인 질문 텍스트가 표시된다. |
| WF-012 | 사용자가 전송 버튼을 선택 | WF-013 | 작성한 질문이 대화 영역에 올라가고 AI 응답 대기 상태가 표시된다. |
| WF-013 | AI 튜터가 응답 생성을 시작 | WF-014 | 로딩 표시 아래에 생성 중인 AI 답변 일부가 나타난다. |
| WF-014 | AI 튜터 응답 생성이 완료 | WF-015 | 생성 중 표시가 사라지고 완성된 AI 답변 말풍선이 표시된다. |
