# Language AI (언어 AI)

텍스트를 이해하고 생성하는 기술 전반을 가리키는 말이다.
NLP(Natural Language Processing)와 거의 같은 의미로 쓰이며,
머신러닝 기반 언어 처리 방법이 주류가 된 이후로 두 용어는 사실상 혼용된다.

Language AI가 다루는 태스크는 크게 세 가지다.

1. **텍스트 출력** — 문장을 생성하거나 번역, 요약하는 작업 (generative modeling)
2. **임베딩 출력** — 텍스트를 숫자 벡터로 변환해 의미를 수치화하는 작업
3. **분류 출력** — 텍스트에 카테고리 레이블을 붙이는 작업 (감정 분석, 스팸 필터 등)

## 역사 흐름

- **2000년대 이전**: Bag-of-Words. 단어 빈도수를 세는 방식으로 텍스트를 숫자로 표현.
- **2013**: word2vec. 단어의 의미를 고차원 벡터로 표현하는 임베딩 등장.
- **2017**: Attention 메커니즘, Transformer 아키텍처 등장 ("Attention is All You Need").
- **2018~2019**: BERT, GPT, GPT-2. 사전학습 + 파인튜닝 패러다임 정착.
- **2022~2023**: ChatGPT. LLM이 대중화되면서 Language AI가 산업 전반으로 확산.

참고: llm.md, embedding.md
