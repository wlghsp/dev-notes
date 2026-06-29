# LLM이란 무엇인가

LLM(Large Language Model)은 방대한 텍스트 데이터로 훈련된 딥 뉴럴 네트워크로, 인간 언어를 이해하고 생성하도록 설계된 모델이다.

"Large"는 두 가지를 동시에 의미한다. 수십억 개에 달하는 파라미터 수, 그리고 학습에 사용된 방대한 데이터셋의 규모. GPT-3는 1,750억 개의 파라미터를 가진다.

LLM이 하는 핵심 작업은 단순하다. **다음 단어 예측(next-word prediction)**. 입력된 텍스트 시퀀스를 보고 그 다음에 올 단어를 예측하는 것을 반복하면서 텍스트를 생성한다. 이 단순한 작업이 수조 개의 토큰 규모로 학습되면, 번역·요약·질의응답·코드 생성 같은 복잡한 능력이 자연스럽게 나타난다. 이를 **창발적 행동(emergent behavior)**이라고 부른다.

LLM은 AI 계층 구조에서 이렇게 위치한다.

```
Artificial Intelligence
  └── Machine Learning
        └── Deep Learning
              └── LLM (+ GenAI)
```

이전 NLP 모델들은 이메일 스팸 분류, 패턴 인식처럼 좁은 단일 태스크에 특화됐다. LLM은 하나의 모델로 다양한 NLP 태스크를 수행할 수 있다는 점에서 근본적으로 다르다.

LLM이 언어를 "이해"한다고 할 때, 이는 의식이나 진짜 이해를 의미하지 않는다. 문맥상 일관성 있는 텍스트를 생성하는 능력을 의미한다.

> 📷 Figure 1.1 (책 p.3) — AI > Machine Learning > Deep Learning > LLM/GenAI 계층 다이어그램

참고: gpt-architecture.md, transformer-architecture.md, llm-build-stages.md
