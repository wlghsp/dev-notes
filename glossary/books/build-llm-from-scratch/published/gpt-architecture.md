# GPT 아키텍처

GPT(Generative Pre-trained Transformer)는 OpenAI가 2018년 "Improving Language Understanding by Generative Pre-Training" 논문에서 소개한 모델이다. 원본 Transformer의 Decoder 부분만 사용하는 decoder-only 아키텍처다.

## 핵심 특징: Decoder-Only

원본 Transformer는 Encoder + Decoder 구조였지만, GPT는 Decoder만 사용한다. Encoder 없이 작동하므로 구조가 단순해진다.

Decoder-only가 가능한 이유는 학습 태스크 덕분이다. 입력 텍스트의 다음 단어를 예측하는 것만으로도 언어의 구조, 문맥, 관계를 학습하기에 충분하다.

## 자기회귀(Autoregressive) 생성

GPT는 텍스트를 한 번에 하나씩 생성한다. 이전에 생성한 출력이 다음 입력의 일부가 된다.

```
입력: "This"
→ 출력: "is"

입력: "This is"
→ 출력: "an"

입력: "This is an"
→ 출력: "example"
```

이처럼 자신의 이전 출력을 다음 입력으로 쓰는 방식을 autoregressive라고 한다.

> 📷 Figure 1.8 (책 p.13) — GPT autoregressive 생성 3단계 반복 다이어그램

## Self-Supervised Learning

GPT는 레이블 없는 텍스트로 학습된다. 다음 단어 예측 태스크는 레이블을 따로 만들 필요가 없다. 텍스트 자체에서 레이블이 자동 생성된다 — 현재 단어 다음에 오는 단어가 곧 레이블이다. 이를 self-supervised learning 또는 self-labeling이라고 한다.

## 스케일: GPT-3

GPT-3는 GPT의 대규모 버전으로 96개의 Transformer 레이어와 1,750억 개의 파라미터를 가진다. ChatGPT는 GPT-3를 대규모 지시문 데이터셋으로 파인튜닝(InstructGPT 방법)한 모델이다.

## 창발적 능력(Emergent Behavior)

GPT는 번역을 목적으로 훈련된 게 아닌데도 번역을 할 수 있다. 다음 단어 예측이라는 단순한 태스크로만 학습됐지만, 수조 개 토큰 규모의 다국어 데이터에 노출된 결과로 번역 패턴이 자연스럽게 학습됐다. 이처럼 명시적으로 훈련받지 않은 능력이 나타나는 현상을 창발적 행동이라 한다.

## Zero-Shot, Few-Shot

GPT 계열 모델은 파인튜닝이나 아키텍처 변경 없이도 태스크를 수행할 수 있다.

- Zero-Shot: 예시 없이 지시문만으로 태스크 수행
- Few-Shot: 입력에 소수의 예시를 포함해 태스크 수행

> 📷 Figure 1.6 (책 p.10) — Text Completion / Zero-Shot / Few-Shot 비교 다이어그램

참고: transformer-architecture.md, llm-build-stages.md, pretraining-data.md
