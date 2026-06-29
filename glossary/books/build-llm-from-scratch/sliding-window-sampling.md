# Sliding Window Sampling (슬라이딩 윈도우 샘플링)

LLM 사전학습을 위한 입력-타깃 쌍을 만드는 방법이다. 텍스트 위를 일정 크기의 윈도우가 슬라이드하며 입력 시퀀스와 그에 대응하는 타깃 시퀀스를 생성한다.

## 입력-타깃 쌍의 구조

LLM은 다음 단어 예측 태스크로 학습된다. 입력 시퀀스가 있고, 타깃은 입력을 한 칸 오른쪽으로 밀어낸 시퀀스다.

```
입력: [290, 4920, 2241, 287]
타깃: [4920, 2241, 287, 257]
```

타깃의 각 위치는 입력의 해당 위치 다음에 오는 토큰이다. 즉 모델은 매 위치에서 다음 토큰을 예측하도록 학습된다.

> 📷 Figure 2.12 (책 p.35) — "LLMs learn to predict one word at a time" 텍스트에서 윈도우가 한 칸씩 이동하며 입력-타깃 쌍이 만들어지는 다이어그램

## stride와 max_length

`max_length`는 윈도우 크기, 즉 한 번에 모델이 받는 토큰 수다. `stride`는 윈도우가 다음 배치로 이동할 때 몇 칸 이동하는지를 결정한다.

- stride=1: 배치마다 윈도우가 1칸씩 이동 → 배치 간 겹침이 많다
- stride=max_length: 배치마다 윈도우가 윈도우 크기만큼 이동 → 겹침 없음, 데이터 100% 활용

> 📷 Figure 2.14 (책 p.40) — stride=1일 때와 stride=4일 때 배치 1과 배치 2의 입력이 어떻게 달라지는지 비교

## PyTorch DataLoader 구현

```python
class GPTDatasetV1(Dataset):
    def __init__(self, txt, tokenizer, max_length, stride):
        token_ids = tokenizer.encode(txt)
        for i in range(0, len(token_ids) - max_length, stride):
            input_chunk = token_ids[i:i + max_length]
            target_chunk = token_ids[i + 1: i + max_length + 1]
            self.input_ids.append(torch.tensor(input_chunk))
            self.target_ids.append(torch.tensor(target_chunk))
```

> 📷 Figure 2.13 (책 p.37) — 배치를 텐서로 표현할 때 x(입력)와 y(타깃)가 각각 행렬로 구성되는 다이어그램

참고: token-id.md, bpe.md, token-embedding.md
