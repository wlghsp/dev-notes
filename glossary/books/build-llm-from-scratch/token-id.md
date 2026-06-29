# Token ID (토큰 ID)

토큰 ID는 토큰(문자열)을 정수로 변환한 값이다. 신경망은 숫자를 처리하므로, 텍스트 토큰을 정수 ID로 바꾸는 중간 단계가 필요하다. 이 ID는 이후 임베딩 벡터로 변환된다.

> 📷 Figure 2.6 (책 p.25) — 훈련 텍스트 → 토크나이즈 → 알파벳 정렬 → 고유 토큰마다 정수 ID 부여하는 어휘사전 구축 다이어그램

## 어휘사전(Vocabulary) 구축

전체 훈련 텍스트를 토크나이즈하고, 중복을 제거한 뒤 알파벳순으로 정렬해서 만든다.

```python
all_words = sorted(set(preprocessed))
vocab = {token: integer for integer, token in enumerate(all_words)}
```

"The Verdict" 단편소설 기준으로 어휘사전 크기는 1,130개다.

## 토큰 ID → 텍스트 역변환

LLM이 숫자를 출력하면 다시 텍스트로 변환해야 한다. 어휘사전의 역방향 매핑(`int_to_str`)으로 처리한다.

> 📷 Figure 2.7 (책 p.26) — 새 텍스트 샘플을 토크나이즈하고 기존 어휘사전으로 Token ID 시퀀스로 변환하는 다이어그램

## 어휘사전의 범위 제한

어휘사전은 훈련 데이터에서 만들어지기 때문에, 훈련에 없던 단어는 처리할 수 없다. 이 문제를 해결하는 방법이 `<|unk|>` 특수 토큰과 BPE다.

참고: tokenization-impl.md, special-token.md, token-embedding.md
