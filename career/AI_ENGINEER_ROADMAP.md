# AI 엔지니어 성장 로드맵 (백엔드 엔지니어 베이스)
작성: 2026-06-28
전제: 현업 백엔드 유지하면서 AI 활용 역량 추가. 수학보다 코드와 직관 중심.

---

## 목표 포지션

"AI를 활용해서 제품/시스템을 만드는 백엔드 엔지니어"
모델을 직접 연구하는 게 아니라, 모델을 도구로 써서 실제 문제를 푸는 사람.

---

## Phase 1 — 내부 구조 이해 (현재 진행 중)

목표: LLM이 내부에서 어떻게 동작하는지 직관을 갖는다.
완료 기준: GPT 모델을 처음부터 짜서 텍스트를 생성해본다.

참고: Build a Large Language Model From Scratch (Sebastian Raschka)

진행 순서:

1. Chapter 2 — Working with Text Data
   토큰화, BPE, 슬라이딩 윈도우, 임베딩 레이어 직접 구현

2. Chapter 3 — Coding Attention Mechanisms
   Self-Attention, Causal Attention, Multi-Head Attention 코드로 구현

3. Chapter 4 — Implementing a GPT Model from Scratch
   전체 GPT 아키텍처 조립, 텍스트 생성까지

4. Chapter 5 — Pretraining on Unlabeled Data
   학습 루프, temperature/top-k sampling, GPT-2 가중치 로드

완료 기준: GPT-2 가중치 불러와서 직접 텍스트 생성 동작 확인.

---

## Phase 2 — 활용 패턴 습득

목표: LLM을 애플리케이션에 붙이는 주요 패턴을 익힌다.
완료 기준: RAG 파이프라인 하나를 직접 만들어본다.

참고: Hands-On Large Language Models (Jay Alammar & Maarten Grootendorst)

진행 순서:

1. Chapter 4-6 — Text Classification, Clustering, Prompt Engineering
   파인튜닝 개념, zero-shot/few-shot, 프롬프트 설계

2. Chapter 7 — Advanced Text Generation Techniques and Tools
   LangChain, 에이전트 패턴, tool use, ReAct

3. Chapter 8 — Semantic Search and RAG
   dense retrieval, vector store, reranking, RAG 파이프라인 전체 흐름

완료 기준: 로컬 문서(개인 노트나 기술 문서)에 대해 질문-답변 되는 RAG 앱 하나 만들기.

---

## Phase 3 — 파인튜닝과 배포

목표: 모델을 특정 태스크에 맞게 튜닝하고 서빙하는 경험을 쌓는다.
완료 기준: LoRA로 파인튜닝한 모델을 API 형태로 올려본다.

진행 순서:

1. Build LLM Chapter 6-7 — Fine-Tuning (분류 + 지시 따르기)
   분류 헤드, instruction fine-tuning, 평가 방법

2. Hands-On LLM Chapter 10-12 — Embedding Models, Fine-Tuning
   contrastive learning, SBERT, LoRA

3. 실전: LoRA로 작은 모델 파인튜닝 후 FastAPI로 서빙

완료 기준: 파인튜닝된 모델에 POST 요청 날려서 응답 받기.

---

## Phase 4 — 프로덕션 경험 (백엔드 역량 연결)

목표: AI 기능을 실제 백엔드 시스템에 붙이는 경험. 여기서 백엔드 경력이 차별화 포인트가 된다.

집중할 주제:

1. LLM API 비용 최적화
   캐싱, 배치 처리, 프롬프트 압축, 모델 선택 기준

2. 스트리밍 응답 처리
   SSE, 토큰 단위 스트리밍을 백엔드에서 처리하는 방법

3. 관찰 가능성 (Observability)
   LLM 호출 로깅, 레이턴시 추적, 응답 품질 모니터링

4. 에이전트 시스템 설계
   멀티 에이전트 워크플로우, 툴 체이닝, 에러 핸들링

완료 기준: 사이드 프로젝트 하나에 AI 기능 붙여서 실제로 쓸 수 있는 상태로 만들기.

---

## 수학에 대해서

수학 깊이 팔 필요 없다. 대신 아래 두 가지는 감으로 알아두면 충분하다.

- 벡터 내적이 "유사도"를 의미한다는 직관
- softmax가 확률 분포를 만든다는 것

이 이상은 Phase 1-2 하면서 코드 보다 보면 자연스럽게 생긴다. 수학 책 따로 파지 않는다.

---

## 타임라인 (현업 병행 기준)

Phase 1 — 3개월 (Build LLM 책 완독 + 코드 실행)
Phase 2 — 2개월 (Hands-On LLM 주요 챕터)
Phase 3 — 2개월 (파인튜닝 실전)
Phase 4 — 진행 중인 프로젝트에 붙이면서 병행

총 7개월 정도. 현업 유지하면서 주말 + 평일 저녁 조금씩 하면 연말 안에 Phase 3까지 가능하다.

---

## 현재 위치

- Phase 1 진입 — Build LLM Chapter 2부터 시작
- Hands-On LLM Chapter 1-2는 완료 (Phase 2 앞부분 선행됨)
