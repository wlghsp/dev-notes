# LSTM (Long Short-Term Memory)

LSTM은 RNN(Recurrent Neural Network)의 장기 의존성 문제를 해결하기 위해 1997년 제안된 모델이다.
트랜스포머가 등장하기 전까지 NLP의 주력 아키텍처였다.

## RNN의 한계

RNN은 텍스트를 왼쪽에서 오른쪽으로 순서대로 처리한다.
각 단계에서 이전 정보를 hidden state라는 벡터에 압축해 다음 단계로 넘기는데,
문장이 길어질수록 앞부분 정보가 계속 덮어씌워져 희석된다.
이를 장기 의존성 문제(long-term dependency problem)라고 한다.

## LSTM의 해결 방식

LSTM은 hidden state 외에 cell state라는 별도 통로를 추가했다.
중요한 정보는 오래 유지하고, 불필요한 정보는 버리는 세 가지 게이트로 흐름을 제어한다.

1. **forget gate** — 이전 cell state에서 버릴 정보를 결정한다
2. **input gate** — 새로운 정보 중 저장할 것을 결정한다
3. **output gate** — cell state에서 출력할 정보를 결정한다

이 구조 덕분에 RNN보다 훨씬 긴 문장을 처리할 수 있었다.

## 한계

LSTM도 결국 순서대로 처리해야 한다는 구조적 한계를 벗어나지 못했다.
병렬화가 불가능해 학습이 느리고, 매우 긴 시퀀스에서는 여전히 정보 손실이 발생한다.

트랜스포머는 이 순차 처리 구조 자체를 버리고,
Self-Attention으로 모든 토큰을 동시에 참조하는 방식으로 이 한계를 근본적으로 해결했다.

참고: transformer.md, llm.md
