# Token Embedding (토큰 임베딩)

토큰 ID를 연속 벡터로 변환하는 임베딩 레이어다. 정수 인덱스를 받아서 해당 행을 가중치 행렬에서 조회(lookup)하는 방식으로 동작한다.

## 동작 원리

임베딩 레이어는 `vocab_size × embedding_dim` 크기의 가중치 행렬을 가진다. 토큰 ID가 들어오면 그 ID에 해당하는 행을 반환한다. 이 가중치는 학습 중에 역전파로 최적화된다.

```python
embedding_layer = torch.nn.Embedding(vocab_size, output_dim)
# 토큰 ID 3의 임베딩 벡터 조회
embedding_layer(torch.tensor([3]))
```

> 📷 Figure 2.16 (책 p.44) — 가중치 행렬에서 토큰 ID에 해당하는 행을 조회하는 lookup 연산 다이어그램

## 위치 독립성의 한계

토큰 임베딩은 같은 토큰 ID라면 항상 같은 벡터를 반환한다. 시퀀스에서 어느 위치에 있든 상관없다. "fox"가 첫 번째에 오든 네 번째에 오든 동일한 벡터다.

> 📷 Figure 2.17 (책 p.44) — 같은 토큰 ID 5가 1번 위치에 있든 4번 위치에 있든 동일한 임베딩 벡터가 나오는 다이어그램

이 때문에 토큰 임베딩만으로는 순서 정보가 없다. Positional Embedding을 더해서 위치 정보를 주입한다.

## GPT-2 기준 스케일

GPT-2 BPE 어휘사전 크기는 50,257이고, 임베딩 차원은 768(small 기준)이다. 임베딩 행렬 크기는 50,257 × 768이 된다.

참고: token-id.md, positional-encoding.md, word-embedding.md
