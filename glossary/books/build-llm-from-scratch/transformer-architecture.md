# Transformer 아키텍처

2017년 논문 "Attention Is All You Need"에서 소개된 딥 뉴럴 네트워크 아키텍처. 원래 기계번역을 위해 설계됐고, 현대 LLM의 근간이 된다.

## 원래 구조: Encoder + Decoder

원본 Transformer는 두 서브모듈로 구성된다.

**Encoder**
- 입력 텍스트 전체를 읽어 문맥 정보를 담은 임베딩 벡터로 변환
- 입력 시퀀스의 양방향(앞뒤 모두)을 참조할 수 있다

**Decoder**
- Encoder가 만든 벡터를 받아 출력 텍스트를 한 단어씩 생성
- 이미 생성한 단어들만 참조하며 다음 단어를 예측

번역 예시: "This is an example" → Encoder가 벡터로 인코딩 → Decoder가 "Das ist ein Beispiel"을 한 단어씩 생성.

> 📷 Figure 1.4 (책 p.8) — Encoder/Decoder 구조 다이어그램, 번역 과정 8단계

## 핵심 메커니즘: Self-Attention

Transformer의 핵심은 Self-Attention 메커니즘이다. 시퀀스 내 각 토큰이 다른 모든 토큰과의 관계 가중치를 계산해, 어떤 토큰에 집중할지를 동적으로 결정한다. 이를 통해 long-range dependency(문장 앞부분과 뒷부분의 관계)를 포착할 수 있다.

## BERT vs GPT: 아키텍처 분화

원본 Transformer에서 두 계열이 파생됐다.

**BERT** (Encoder-only)
- Encoder 부분만 사용
- 학습 방식: 문장 내 일부 단어를 마스킹하고 맞추는 Masked Word Prediction
- 강점: 텍스트 분류, 감성 분석처럼 전체 문맥 이해가 필요한 태스크

**GPT** (Decoder-only)
- Decoder 부분만 사용
- 학습 방식: 이전 토큰들로 다음 토큰을 예측하는 Next-Word Prediction
- 강점: 텍스트 생성, 번역, 요약처럼 텍스트를 만들어내는 태스크

> 📷 Figure 1.5 (책 p.9) — BERT(Encoder)와 GPT(Decoder) 비교 다이어그램

## Transformers vs LLMs

모든 LLM이 Transformer는 아니고, 모든 Transformer가 LLM도 아니다. Transformer는 컴퓨터 비전에도 쓰인다. 일부 LLM은 RNN이나 CNN 기반 아키텍처를 사용하기도 한다. 다만 현재 주류 LLM은 Transformer 기반이기 때문에 두 용어가 혼용되는 경향이 있다.

참고: gpt-architecture.md, attention-mechanism.md, self-attention-impl.md
