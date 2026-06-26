# Chapter 1 — Understanding Large Language Models

키워드 파일: llm-what-is.md, llm-build-stages.md, transformer-architecture.md, gpt-architecture.md, pretraining-data.md

---

## 이 챕터가 답하는 질문

LLM이 무엇인지, 어떤 아키텍처로 만들어졌는지, 이 책에서 어떤 순서로 구현할 것인지를 개괄한다.

---

## LLM = Next-Word Prediction × Scale

LLM이 하는 일의 본질은 단순하다. 이전 토큰들을 보고 다음 토큰을 예측한다. 이 단순한 self-supervised 학습 태스크가 수천억 개 토큰 규모로 반복되면, 번역·요약·코드 생성 같은 능력이 명시적으로 학습하지 않아도 창발적으로 나타난다.

이전 NLP 모델들은 특정 태스크 하나를 잘 하도록 설계됐다. LLM은 하나의 모델이 다양한 태스크를 수행한다는 점에서 패러다임이 다르다.

---

## Transformer: LLM의 근간

현대 LLM은 2017년 "Attention Is All You Need" 논문에서 나온 Transformer 아키텍처를 기반으로 한다. 원본 Transformer는 Encoder + Decoder 구조였지만, LLM 계열에서는 둘로 분화됐다.

**BERT 계열 (Encoder-only)**
- 입력 전체를 양방향으로 참조
- 학습: 마스킹된 단어 맞추기
- 강점: 분류, 감성분석 등 이해 태스크

**GPT 계열 (Decoder-only)**
- 왼쪽에서 오른쪽으로 단방향 처리
- 학습: 다음 단어 예측
- 강점: 텍스트 생성, 번역, 요약 등 생성 태스크

이 책은 GPT 아키텍처를 처음부터 구현한다.

---

## GPT의 Autoregressive 생성

GPT는 한 번에 토큰 하나를 생성한다. 이전 출력이 다음 입력으로 들어간다. "This" → "This is" → "This is an" → "This is an example" 식으로 이전 생성 결과를 누적하며 진행된다.

---

## 사전학습 → 파인튜닝 2단계

LLM 개발은 두 단계로 이루어진다.

1. **Pretraining**: 수천억 토큰의 비레이블 텍스트로 다음 단어 예측 학습 → Foundation Model
2. **Fine-tuning**: 레이블 데이터로 특정 태스크에 맞게 추가 학습 → 분류기 또는 어시스턴트

Foundation Model은 그 자체로 텍스트 완성·few-shot 능력을 갖추지만, 특정 태스크에 특화된 성능이 필요할 때 파인튜닝한다.

> 📷 Figure 1.3 (책 p.6) — Pretraining → Foundation Model → Fine-tuning 흐름도

---

## 이 책의 구현 계획

Stage 1 (Ch 1~4): 데이터 처리 + Attention + GPT 모델 구현

Stage 2 (Ch 5): 비레이블 데이터로 사전학습, 평가, GPT-2 가중치 로드

Stage 3 (Ch 6~7): 분류 파인튜닝 + 지시문 파인튜닝

> 📷 Figure 1.9 (책 p.14) — Stage 1/2/3 전체 9단계 흐름 다이어그램
