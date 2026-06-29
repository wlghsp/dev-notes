# Tokenization (토크나이제이션)

토크나이제이션은 입력 텍스트를 개별 토큰으로 분리하는 전처리 단계다. LLM은 원본 텍스트를 직접 처리할 수 없기 때문에, 토큰으로 쪼갠 뒤 정수 ID로 변환해서 임베딩 벡터로 만든다.

> 📷 Figure 2.4 (책 p.21) — 입력 텍스트 → 토크나이즈 → Token ID → 임베딩 → GPT 모델로 이어지는 전체 파이프라인

## 기본 토크나이저 구현

Python의 `re` 라이브러리로 단순 토크나이저를 만들 수 있다.

공백으로만 분리하면 구두점이 단어에 붙어 있는 문제가 생긴다. 공백, 쉼표, 마침표, 물음표, 느낌표, 따옴표, 대시 등을 기준으로 분리하면 구두점도 별도 토큰이 된다.

```python
result = re.split(r'([,.:;?_!"()\']|--|\s)', text)
result = [item.strip() for item in result if item.strip()]
```

> 📷 Figure 2.5 (책 p.24) — "Hello, world. Is this-- a test?"가 10개 토큰으로 분리되는 예시

## 대소문자 유지

소문자로 통일하지 않는다. 고유명사와 일반명사를 구별하고, 문장 구조를 이해하고, 올바른 대소문자로 텍스트를 생성하는 데 도움이 되기 때문이다.

## SimpleTokenizerV1 구조

```python
class SimpleTokenizerV1:
    def __init__(self, vocab):
        self.str_to_int = vocab
        self.int_to_str = {i:s for s,i in vocab.items()}

    def encode(self, text):  # 텍스트 → token ID 리스트
        ...

    def decode(self, ids):   # token ID 리스트 → 텍스트
        ...
```

encode는 텍스트를 토큰으로 분리하고 어휘사전으로 정수 ID를 조회한다. decode는 역방향 매핑으로 ID를 다시 단어로 변환한다.

> 📷 Figure 2.8 (책 p.28) — encode/decode 방향 다이어그램

## 미등록 단어 문제

훈련 데이터에 없던 단어는 어휘사전에 없기 때문에 KeyError가 발생한다. 이를 해결하기 위해 `<|unk|>` 특수 토큰을 도입하거나, BPE처럼 서브워드 단위로 쪼개는 방식을 쓴다.

참고: token-id.md, special-token.md, bpe.md
