# BPE (Byte Pair Encoding)

BPE는 서브워드 단위로 어휘를 구성하는 토크나이제이션 알고리즘이다. GPT-2, GPT-3, ChatGPT 원본 모델이 모두 BPE 토크나이저를 사용한다. Python 라이브러리 `tiktoken`이 Rust 기반으로 구현해 제공한다.

## 핵심 아이디어

단어를 통째로 토큰으로 쓰는 대신, 자주 등장하는 문자 조합을 반복적으로 합쳐서 서브워드 단위의 어휘를 만든다.

1. 처음에는 개별 문자 하나하나가 어휘가 된다 ("a", "b", "c", ...)
2. 가장 자주 함께 등장하는 쌍을 하나로 합친다 ("d" + "e" → "de")
3. 이 과정을 정해진 횟수만큼 반복한다

결과적으로 흔한 단어는 통째로 하나의 토큰이 되고, 드문 단어는 여러 서브워드 토큰으로 쪼개진다.

## 미등록 단어 처리

BPE의 핵심 장점이다. 어휘사전에 없는 완전히 낯선 단어도 개별 문자로 분해할 수 있기 때문에 `<|unk|>` 토큰이 필요 없다.

> 📷 Figure 2.11 (책 p.34) — "Akwirw ier" 같은 미등록 단어가 서브워드와 개별 문자로 분해되는 다이어그램

예: "Akwirw ier" → ["Ak", "w", "ir", "w", " ", "ier"] → [33901, 86, 343, 86, 220, 959]

## GPT-2 어휘사전 크기

GPT-2 BPE 토크나이저의 전체 어휘사전 크기는 50,257개다. `<|endoftext|>`가 가장 큰 ID인 50,256을 갖는다.

```python
import tiktoken
tokenizer = tiktoken.get_encoding("gpt2")
integers = tokenizer.encode(text, allowed_special={"<|endoftext|>"})
strings = tokenizer.decode(integers)
```

참고: tokenization-impl.md, special-token.md, token-id.md
