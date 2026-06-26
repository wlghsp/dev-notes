# LLM 구축 3단계

이 책에서 LLM을 만드는 과정은 세 단계로 나뉜다.

**Stage 1 — Building an LLM**

데이터 전처리, Attention 메커니즘, LLM 아키텍처를 구현하는 단계. 모델이 어떻게 작동하는지 기계 수준에서 이해하는 것이 목적이다.

- 데이터 준비 및 샘플링
- Attention 메커니즘 구현
- LLM 아키텍처 구현

**Stage 2 — Pretraining (Foundation Model)**

구현한 LLM을 비레이블 텍스트로 사전학습하는 단계. 다음 단어 예측 태스크로 학습해 Foundation Model을 만든다. 실제 대규모 학습은 수백만 달러가 들기 때문에, 이 책에서는 소규모 교육용 데이터로 구현하고 OpenAI의 GPT-2 사전학습 가중치를 로드하는 방법도 함께 다룬다.

- 학습 루프 구현
- 모델 평가
- 사전학습 가중치 로드

**Stage 3 — Fine-Tuning**

Foundation Model을 레이블 데이터로 특정 태스크에 맞게 파인튜닝하는 단계. 두 방향으로 나뉜다.

- 분류 파인튜닝: 스팸 분류기처럼 텍스트에 레이블을 붙이는 모델
- 지시문 파인튜닝: 사용자 지시를 따르는 개인 어시스턴트 모델

> 📷 Figure 1.9 (책 p.14) — Stage 1/2/3 전체 흐름 다이어그램 (책 앞부분 삽화와 동일)

## 왜 직접 구축하는가

커스텀 LLM은 일반 목적 LLM(ChatGPT)보다 특정 도메인에서 더 나은 성능을 낼 수 있다. BloombergGPT(금융), 의료 QA 모델이 그 예다. 또한 민감한 데이터를 외부 LLM 제공자에게 보내지 않아도 된다는 프라이버시 장점도 있다.

참고: gpt-architecture.md, pretraining-data.md, fine-tuning-types.md
