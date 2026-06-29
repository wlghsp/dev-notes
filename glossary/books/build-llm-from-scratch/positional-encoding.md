# Positional Encoding (위치 인코딩)

토큰 임베딩은 같은 토큰 ID라면 시퀀스 어느 위치에 있든 동일한 벡터를 반환한다. Self-Attention도 본질적으로 위치를 고려하지 않는다. 이 때문에 위치 정보를 별도로 주입해야 한다. 토큰 임베딩 벡터에 위치 임베딩 벡터를 더하는 방식으로 한다.

```
입력 임베딩 = 토큰 임베딩 + 위치 임베딩
```

> 📷 Figure 2.18 (책 p.45) — 토큰 임베딩 [1,1,1]에 위치 임베딩이 더해져 입력 임베딩이 만들어지는 다이어그램
> 📷 Figure 2.19 (책 p.48) — 텍스트 → 토크나이즈 → Token ID → 토큰 임베딩 + 위치 임베딩 → 입력 임베딩 → GPT 모델 전체 파이프라인

## 두 가지 방식

절대 위치 임베딩(Absolute Positional Embedding) — 시퀀스의 각 위치에 고정된 고유 벡터를 부여한다. 1번 위치는 항상 같은 벡터, 2번 위치도 항상 같은 벡터다.

상대 위치 임베딩(Relative Positional Embedding) — 위치 자체가 아니라 토큰 간 거리에 집중한다. "몇 번째"가 아니라 "얼마나 떨어졌는가"를 인코딩한다. 학습 때 보지 못한 길이의 시퀀스에도 더 잘 일반화된다.

## GPT의 선택

GPT는 절대 위치 임베딩을 사용한다. 원본 Transformer의 고정된 sinusoidal 인코딩과 달리, GPT는 위치 임베딩도 학습 가능한 파라미터로 두고 학습 중에 최적화한다.

```python
context_length = max_length        # 지원하는 최대 입력 길이
pos_embedding_layer = torch.nn.Embedding(context_length, output_dim)
pos_embeddings = pos_embedding_layer(torch.arange(context_length))
input_embeddings = token_embeddings + pos_embeddings
```

`torch.arange(context_length)`는 [0, 1, 2, ..., context_length-1] 위치 인덱스를 만든다. 각 위치 인덱스가 임베딩 레이어를 통해 벡터로 변환된다.

참고: token-embedding.md, word-embedding.md
