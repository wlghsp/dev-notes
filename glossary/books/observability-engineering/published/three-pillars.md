# Three Pillars

Trace, Metric, Log 세 가지 데이터 타입을 갖추면 observability가 달성된다고 설명하는 프레임. 일부 벤더가 자사 도구를 "이 세 가지를 다 지원한다"고 마케팅하면서 널리 퍼진 표현이다.

## 왜 틀린 프레임인가

이 프레임의 문제는, 세 데이터 타입을 나열하는 것 자체를 목적으로 만들어버린다는 점이다. Trace 저장소 하나, Metric 저장소 하나, Log 저장소 하나를 각각 갖추고 있으면 observability를 갖췄다고 착각하게 만든다.

하지만 세 가지 데이터를 따로따로 모아두는 도구가 있다고 해서 자동으로 observability가 생기지 않는다. 정작 중요한 건 데이터 타입의 개수가 아니라, 그 데이터가 실제로 얼마나 세밀한 질의를 감당할 수 있는가다.

## 대신 봐야 하는 것

세 가지를 그냥 나열하는 대신, 데이터가 다음 조건을 갖췄는지를 봐야 한다.

- Cardinality와 Dimensionality가 충분히 높은가 (cardinality-dimensionality.md)
- Explorability가 있는가 — 문제가 무엇인지 미리 예측하지 않고도, "여기 이상해 보이네 → 이 필드로 걸러보자 → 아직 이상하네 → 다른 필드로 다시 걸러보자" 하는 식으로 한 걸음씩 질문을 이어가며 답에 다가갈 수 있는 능력

Explorability가 가능하려면 cardinality와 dimensionality가 뒷받침돼야 한다. 데이터 값이 뭉개져 있거나(cardinality 낮음) 필드가 몇 개 안 되면(dimensionality 낮음), 아무리 반복해서 질문을 던져도 막다른 골목에 부딪힌다.

## Monitoring 벤더가 이 프레임을 선호하는 이유

책은 이 지점을 특히 비판적으로 짚는다. 이미 Metric/Log/Trace를 각각 따로 수집·저장하는 도구를 파는 벤더 입장에서는, "이 셋만 갖추면 observability"라는 정의가 자신들의 기존 상품군을 그대로 재포장해서 팔 수 있게 해준다. 즉 이 정의는 중립적인 기술적 정의라기보다, 기존 도구를 계속 팔고 싶은 쪽의 이해관계가 반영된 정의에 가깝다는 것이다.

참고: cardinality-dimensionality.md, known-unknown-vs-unknown-unknown.md

## Recap

Three Pillars는 Trace/Metric/Log 세 가지를 갖추면 observability가 된다는 프레임인데, 데이터 타입을 나열하는 것 자체가 목적이 되어버려 정작 중요한 조건(cardinality, dimensionality, explorability)을 가린다. 세 저장소를 각각 갖췄다고 자동으로 observability가 생기지 않는다.
