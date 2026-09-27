# Cardinality와 Dimensionality

Observability 데이터를 얼마나 세밀하게 다룰 수 있는지를 결정하는 두 축.

## Cardinality

어떤 필드가 가진 값이 얼마나 고유한가를 나타낸다. 값의 종류가 몇 개 안 되면 low cardinality다. 예를 들어 성별 필드는 값의 가짓수가 적다. 반대로 거의 모든 값이 서로 다르면 high cardinality다. user id나 UUID처럼 유일성을 보장하는 필드가 가장 높은 cardinality를 가진다.

디버깅 관점에서는 high-cardinality 필드가 훨씬 유용하다. "특정 user id를 가진 요청만" 같은 조건으로 정확히 하나를 짚어낼 수 있기 때문이다. 문제는 전통적인 metric 기반 시스템(TSDB)이 구조상 high-cardinality 값을 태그로 다루기 어렵다는 점이다. 태그 값의 조합마다 별도의 시계열을 저장해야 하는데, 태그 값의 종류가 많아지면 저장해야 할 시계열 개수가 기하급수적으로 늘어난다(cardinality explosion).

## Dimensionality

하나의 데이터가 가진 필드(key)의 개수를 나타낸다. Observability에서 다루는 이벤트는 수십에서 수백 개의 key-value 쌍을 가진 "넓은(wide)" 구조로 기록된다. 필드가 많을수록, 나중에 "이 요청들의 공통점이 뭐지"를 찾을 때 조합해볼 수 있는 경우의 수가 늘어난다.

## 왜 둘 다 필요한가

Cardinality가 높아도 dimensionality가 낮으면(필드가 몇 개 안 되면) 정밀하게 걸러낼 수 있는 조건 자체가 몇 가지뿐이다. 반대로 dimensionality가 높아도 각 필드의 cardinality가 낮으면(예: status가 success/fail 두 가지뿐이면) 필드를 아무리 조합해도 개별 요청 수준까지 좁혀지지 않는다.

두 축이 동시에 높아야 "이 요청들의 공통점"을 정확히 찾아낼 수 있다. 예를 들어 "캐나다에 있고, iOS 11.0.4를 쓰고, 프랑스어 언어팩을 쓰고, 지난주 화요일에 앱을 설치하고, shard3에 사진을 저장하는 us-west-1 리전의 사용자"라는 조건은 여러 high-cardinality 필드를 동시에 조합한 결과다. 이런 조합형 질의를 감당하려면 두 축이 모두 필요하다.

참고: observability.md

## Recap

Cardinality는 값이 얼마나 고유한가, Dimensionality는 필드가 얼마나 많은가를 뜻한다. 둘 다 높아야 "이 요청들의 공통점이 뭐지" 같은 세밀한 조합형 질의가 가능해진다. 전통적인 metric 기반 시스템은 high-cardinality 데이터를 구조적으로 감당하지 못한다.
