# Hands-On LLM Glossary 진도표
참고: Hands-On Large Language Models — Jay Alammar & Maarten Grootendorst (O'Reilly, 2024)

---

## Part I — Understanding Language Models

### Chapter 1 — An Introduction to Large Language Models

- [x] llm.md — Large Language Model, 두 타입(representation / generative) 구분
- [x] language-ai.md — Language AI / NLP, 역사 흐름 (Bag-of-Words → word2vec → Transformer → LLM)
- [x] embedding.md — 텍스트를 의미를 담은 숫자 벡터로 변환한 표현
- [x] representation-model-vs-generative-model.md — Encoder-only vs Decoder-only, 용도 차이

### Chapter 2 — Tokens and Embeddings

- [ ] token.md — 모델이 처리하는 최소 단위, 단어/서브워드/문자/바이트 토큰 비교
- [ ] tokenization.md — 텍스트를 토큰으로 분리하는 과정, BPE 등 알고리즘
- [ ] vocabulary.md — 모델이 알고 있는 토큰 전체 집합
- [ ] context-window.md — 모델이 한 번에 처리할 수 있는 최대 토큰 수
- [ ] word2vec.md — 단어 임베딩 학습 알고리즘, 주변 단어 예측으로 의미 관계 포착
- [ ] contextualized-embedding.md — 문맥에 따라 달라지는 임베딩, BERT 방식

### Chapter 3 — Looking Inside Large Language Models

- [ ] transformer.md — LLM의 핵심 아키텍처, Encoder/Decoder 블록 구조
- [ ] attention.md — 토큰 간 관계를 가중치로 표현하는 메커니즘 ("Attention is All You Need")
- [ ] self-attention.md — 입력 시퀀스 내 토큰들이 서로를 참조하는 어텐션
- [ ] kv-cache.md — 추론 속도 향상을 위해 Key/Value를 캐싱하는 기법
- [ ] sampling-decoding.md — 다음 토큰을 선택하는 방식 (greedy, temperature, top-p 등)

---

## Part II — Using Pretrained Language Models

### Chapter 4 — Text Classification

- [ ] text-classification.md — 텍스트에 카테고리 레이블을 붙이는 태스크
- [ ] fine-tuning.md — 사전학습 모델을 특정 태스크에 맞게 추가 학습하는 것
- [ ] zero-shot-few-shot.md — 예시 없이 또는 소수 예시만으로 태스크 수행하는 방식

### Chapter 5 — Text Clustering and Topic Modeling

- [ ] text-clustering.md — 유사한 문서를 그룹으로 묶는 비지도 학습 방식
- [ ] dimensionality-reduction.md — 고차원 임베딩을 저차원으로 압축하는 기법 (UMAP 등)
- [ ] topic-modeling.md — 문서 집합에서 주제를 자동 추출하는 방법 (BERTopic 등)

### Chapter 6 — Prompt Engineering

- [ ] prompt.md — 모델에게 전달하는 입력 텍스트, system prompt / user prompt 구조
- [ ] prompt-engineering.md — 원하는 출력을 얻기 위해 프롬프트를 설계하는 방법
- [ ] chain-of-thought.md — 단계적 추론을 유도하는 프롬프팅 기법
- [ ] in-context-learning.md — 파라미터 업데이트 없이 예시만으로 태스크를 수행하게 하는 것

### Chapter 7 — Advanced Text Generation Techniques and Tools

- [ ] langchain.md — LLM 애플리케이션 개발 프레임워크, 체인/에이전트/메모리 조합
- [ ] chain.md — LLM 호출을 순서대로 연결하는 패턴
- [ ] agent.md — LLM이 툴을 사용해 스스로 행동을 결정하고 실행하는 패턴
- [ ] tool-use.md — LLM이 외부 함수/API를 호출하는 기능 (function calling)
- [ ] react.md — Reasoning + Acting, LLM 에이전트가 추론-행동-관찰을 반복하는 패턴

### Chapter 8 — Semantic Search and Retrieval-Augmented Generation

- [ ] semantic-search.md — 키워드 매칭이 아닌 의미 유사도 기반 검색
- [ ] dense-retrieval.md — 쿼리와 문서를 임베딩 벡터로 변환해 유사도로 검색하는 방식
- [ ] vector-store.md — 임베딩 벡터를 저장하고 유사도 검색을 지원하는 데이터베이스
- [ ] reranking.md — 검색 결과를 다시 정렬해 정확도를 높이는 단계
- [ ] rag.md — Retrieval-Augmented Generation, 외부 문서를 검색해 LLM 생성에 활용하는 패턴

### Chapter 9 — Multimodal Large Language Models

- [ ] multimodal.md — 텍스트 외 이미지 등 다른 모달리티를 함께 처리하는 모델
- [ ] clip.md — 텍스트-이미지 쌍을 함께 학습해 멀티모달 임베딩을 만드는 모델

---

## Part III — Training and Fine-Tuning Language Models

### Chapter 10 — Creating Text Embedding Models

- [ ] contrastive-learning.md — 유사한 쌍은 가깝게, 다른 쌍은 멀게 학습하는 방식
- [ ] sbert.md — Sentence-BERT, 문장 임베딩을 위해 BERT를 튜닝한 모델

### Chapter 11 — Fine-Tuning Representation Models for Classification

- [ ] bert.md — Bidirectional Encoder Representations from Transformers, encoder-only 대표 모델
- [ ] few-shot-classification.md — 적은 레이블 데이터로 분류 모델을 학습하는 방법

### Chapter 12 — Fine-Tuning Generation Models

- [ ] pretraining-sft-rlhf.md — LLM 학습 3단계: 사전학습 → 지도 파인튜닝 → 선호도 정렬
- [ ] lora.md — Low-Rank Adaptation, 파라미터 효율적 파인튜닝 기법
- [ ] rlhf.md — Reinforcement Learning from Human Feedback, 인간 선호도로 모델을 정렬하는 방법
- [ ] dpo.md — Direct Preference Optimization, 리워드 모델 없이 선호도 학습하는 방법
- [ ] quantization.md — 모델 파라미터 정밀도를 낮춰 메모리와 속도를 최적화하는 기법

---

진행 방식: 챕터 단위로 완료 후 다음 챕터 이동. 완료된 항목은 [x]로 표시.
