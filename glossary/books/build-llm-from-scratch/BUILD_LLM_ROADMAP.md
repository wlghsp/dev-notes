# Build a Large Language Model (From Scratch) 진도표
참고: Build a Large Language Model (From Scratch) — Sebastian Raschka (Manning, 2025)

실습 환경: 저자 GitHub(rasbt/LLMs-from-scratch)의 챕터별 Jupyter Notebook 기준
실습 체크포인트는 개념 glossary 완료 후 진행. 코드를 직접 돌리고 출력을 눈으로 확인해야 완료.

---

## Stage 1 — Building an LLM

### Chapter 1 — Understanding Large Language Models

- [x] llm-what-is.md — LLM이란 무엇인가, GPT 계열의 특징과 위치
- [x] llm-build-stages.md — LLM 구축 3단계: 아키텍처 구현 → 사전학습 → 파인튜닝
- [x] transformer-architecture.md — Transformer 아키텍처 개요, Encoder/Decoder 역할 분리
- [x] gpt-architecture.md — GPT 아키텍처 상세, decoder-only 구조와 자기회귀 생성 방식
- [x] pretraining-data.md — 사전학습에 사용되는 대규모 비레이블 데이터셋의 역할
- [x] ch01-understanding-llms.md — Chapter 1 종합 복습 문서

> Chapter 1은 이론 챕터. 별도 코딩 실습 없음.

### Chapter 2 — Working with Text Data

- [ ] word-embedding.md — 단어를 연속 벡터 공간에 표현하는 방법, 의미 유사도 포착
- [ ] tokenization-impl.md — 텍스트를 토큰으로 분리하는 구체적 구현 방법
- [ ] token-id.md — 토큰을 정수 ID로 변환하는 과정, 어휘 사전과의 매핑
- [ ] special-token.md — [BOS], [EOS], [PAD], [UNK] 등 특수 컨텍스트 토큰의 역할
- [ ] bpe.md — Byte Pair Encoding, 서브워드 단위로 어휘를 구성하는 알고리즘
- [ ] sliding-window-sampling.md — 슬라이딩 윈도우로 학습 데이터 샘플링하는 방식
- [ ] token-embedding.md — 토큰 ID를 임베딩 벡터로 변환하는 임베딩 레이어
- [ ] positional-encoding.md — 토큰 순서 정보를 임베딩에 추가하는 위치 인코딩

실습 체크포인트 (ch02 notebook):
- [ ] 실습 2-1: 텍스트를 직접 토크나이저로 분리하고 token ID 목록 출력 확인 (§2.2~2.3)
- [ ] 실습 2-2: tiktoken으로 BPE 적용 — "Ak" 같은 미등록 단어가 서브워드로 쪼개지는 것 확인 (§2.5)
- [ ] 실습 2-3: 슬라이딩 윈도우 DataLoader 만들어서 input/target 쌍 배치 출력 (§2.6)
- [ ] 실습 2-4: 임베딩 레이어 + 위치 인코딩 합산해서 최종 입력 텐서 shape 확인 (§2.7~2.8)

### Chapter 3 — Coding Attention Mechanisms

- [ ] attention-mechanism.md — 시퀀스 내 토큰 간 의존성을 가중치로 표현하는 메커니즘
- [ ] self-attention-impl.md — Self-Attention 수식 전개 및 Python 코드 구현
- [ ] attention-weight.md — 어텐션 가중치 계산: Query·Key 내적 후 Softmax 적용
- [ ] causal-attention.md — 미래 토큰을 마스킹해 자기회귀 생성을 가능하게 하는 어텐션
- [ ] attention-dropout.md — 어텐션 가중치에 드롭아웃을 적용해 과적합을 방지하는 기법
- [ ] multi-head-attention.md — 여러 어텐션 헤드를 병렬로 실행해 다양한 의존성을 포착하는 방식

실습 체크포인트 (ch03 notebook):
- [ ] 실습 3-1: 가중치 없는 단순 Self-Attention 직접 계산 — 어텐션 행렬 값 눈으로 확인 (§3.3)
- [ ] 실습 3-2: Q·K·V 행렬 도입한 Self-Attention 클래스 구현 및 출력 shape 검증 (§3.4)
- [ ] 실습 3-3: Causal mask 적용해서 상삼각 부분이 -inf로 마스킹되는 것 확인 (§3.5)
- [ ] 실습 3-4: MultiHeadAttention 클래스 구현 후 입력 대비 출력 shape 확인 (§3.6)

### Chapter 4 — Implementing a GPT Model from Scratch to Generate Text

- [ ] gpt-model-impl.md — GPT 전체 모델 코드 구현: 임베딩 → Transformer 블록 → 출력 레이어
- [ ] layer-normalization.md — 레이어 정규화, 학습 안정성을 위해 활성화 값을 정규화하는 기법
- [ ] feed-forward-network.md — Transformer 내 FFN 구조, GELU 활성화 함수 사용 이유
- [ ] gelu.md — GELU 활성화 함수, ReLU 대비 부드러운 비선형성 제공
- [ ] shortcut-connection.md — 잔차 연결(Residual Connection), 깊은 네트워크의 기울기 소실 방지
- [ ] transformer-block.md — Attention + FFN + LayerNorm + Residual을 하나로 묶은 블록 구조
- [ ] text-generation.md — 학습된 GPT로 텍스트 생성하는 과정, 토큰 예측의 반복

실습 체크포인트 (ch04 notebook):
- [ ] 실습 4-1: GPT 설정 dict 작성 후 더미 데이터로 전체 모델 forward pass 실행 — 출력 shape 확인 (§4.1~4.6)
- [ ] 실습 4-2: 파라미터 수 직접 계산해서 GPT-2 small(124M)과 맞는지 검증 (§4.6)
- [ ] 실습 4-3: generate_text_simple 함수로 텍스트 생성 — 의미없어도 토큰이 나오는 것 확인 (§4.7)

---

## Stage 2 — Pretraining

### Chapter 5 — Pretraining on Unlabeled Data

- [ ] text-generation-loss.md — 텍스트 생성 손실 계산, cross-entropy loss 적용 방식
- [ ] llm-training-loop.md — LLM 학습 루프: 배치 샘플링 → forward → loss → backward → optimizer
- [ ] decoding-strategy.md — 텍스트 생성 시 무작위성 제어 전략: temperature, top-k sampling
- [ ] temperature-scaling.md — temperature 값으로 토큰 분포의 날카로움/부드러움을 조절하는 방법
- [ ] top-k-sampling.md — 상위 k개 토큰만 후보로 유지해 생성 품질을 높이는 방식
- [ ] model-weight-saving.md — PyTorch에서 모델 가중치를 저장하고 불러오는 방법
- [ ] pretrained-weight-loading.md — OpenAI GPT-2 사전학습 가중치를 직접 로드하는 방법

실습 체크포인트 (ch05 notebook):
- [ ] 실습 5-1: train/validation loss 계산 함수 구현 후 학습 전 초기 loss 값 확인 (§5.1)
- [ ] 실습 5-2: 학습 루프 실행 — epoch마다 loss가 줄어드는 것 직접 눈으로 확인 (§5.2)
- [ ] 실습 5-3: temperature와 top-k 값 바꿔가며 생성 텍스트가 달라지는 것 비교 (§5.3)
- [ ] 실습 5-4: GPT-2 사전학습 가중치 로드 후 텍스트 생성 — 학습 전과 품질 차이 체감 (§5.5)

---

## Stage 3 — Fine-Tuning

### Chapter 6 — Fine-Tuning for Classification

- [ ] fine-tuning-types.md — 파인튜닝의 종류: feature extraction vs full fine-tuning vs LoRA
- [ ] classification-head.md — 사전학습 LLM 위에 분류 헤드를 붙여 분류 모델로 전환하는 방법
- [ ] classification-loss.md — 분류 태스크의 손실 계산, cross-entropy와 정확도 측정
- [ ] spam-classifier.md — GPT를 스팸 분류기로 파인튜닝하는 실전 예시

실습 체크포인트 (ch06 notebook):
- [ ] 실습 6-1: 스팸 데이터셋 로드 후 DataLoader 구성 및 배치 shape 확인 (§6.2~6.3)
- [ ] 실습 6-2: GPT-2 가중치 로드 후 분류 헤드 붙이고 파인튜닝 전 정확도 측정 (§6.4~6.5)
- [ ] 실습 6-3: 파인튜닝 실행 후 정확도 변화 확인 — 스팸 문장 직접 입력해서 예측 결과 확인 (§6.7~6.8)

### Chapter 7 — Fine-Tuning to Follow Instructions

- [ ] instruction-fine-tuning.md — 지시문 따르기 학습, supervised instruction fine-tuning 개요
- [ ] instruction-dataset.md — 지시문-응답 쌍으로 구성된 데이터셋 준비 방법
- [ ] training-batch-collation.md — 가변 길이 시퀀스를 배치로 묶기 위한 패딩 및 마스킹 처리
- [ ] llm-evaluation.md — 파인튜닝된 LLM 응답 품질을 평가하는 방법

실습 체크포인트 (ch07 notebook):
- [ ] 실습 7-1: Alpaca 형식 instruction 데이터셋 로드 후 포맷 변환 확인 (§7.2)
- [ ] 실습 7-2: 가변 길이 배치 collate 함수 구현 — 패딩 마스킹이 loss 계산에서 제외되는 것 확인 (§7.3)
- [ ] 실습 7-3: instruction fine-tuning 실행 후 직접 지시문 입력해서 응답 품질 확인 (§7.6~7.7)
- [ ] 실습 7-4: 파인튜닝 전/후 모델 응답 비교 — 같은 질문에 대한 차이 체감 (§7.8)

---

## Appendix

### Appendix E — Parameter-Efficient Fine-Tuning with LoRA

- [ ] lora-impl.md — LoRA 수식 및 Python 구현, 적은 파라미터로 효율적 파인튜닝하는 방법

실습 체크포인트 (appendix-E notebook):
- [ ] 실습 E-1: LoRA 레이어 직접 구현 후 기존 Linear 레이어와 파라미터 수 비교
- [ ] 실습 E-2: LoRA 적용 모델로 파인튜닝 — 전체 파인튜닝 대비 학습 파라미터 수 차이 확인

---

진행 방식: 챕터 단위로 완료 후 다음 챕터 이동. 완료된 항목은 [x]로 표시.
glossary 완료 → 실습 체크포인트 완료 → 다음 챕터 이동.
