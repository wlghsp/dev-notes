# Known-unknown과 Unknown-unknown

Monitoring과 Observability가 각각 어떤 종류의 문제에 대응하는지를 구분하는 틀.

Known-unknown은 "무엇을 모르는지는 알고 있는 상태"다. 예를 들어 "에러율이 오를 수 있다"는 건 알고 있고, 지금 그 값이 정확히 얼마인지만 모르는 경우다. 이런 문제는 미리 그 값을 관찰할 대시보드나 알람을 만들어두면 대응할 수 있다. Monitoring이 다루는 영역이 여기다.

Unknown-unknown은 "무엇을 모르는지조차 모르는 상태"다. 한 번도 겪어본 적 없는 새로운 형태의 장애, 예측 범위 밖에 있던 조합의 실패다. 이런 문제는 애초에 어떤 대시보드를 미리 만들어둬야 할지조차 알 수 없기 때문에, 사전에 준비된 monitoring으로는 대응할 수 없다.

## 왜 이 구분이 Observability의 존재 이유가 되는가

과거의 소프트웨어(모놀리식, 정적인 인프라)는 실패 방식이 비교적 예측 가능했고, 엔지니어가 경험으로 대부분의 known-unknown을 커버할 수 있었다. 분산 시스템, 마이크로서비스, 컨테이너 오케스트레이션 환경에서는 조합 가능한 실패의 경우의 수가 사실상 무한에 가까워진다. Unknown-unknown의 비중이 압도적으로 커지고, 이런 환경에서는 monitoring만으로 시스템을 이해하는 데 한계가 뚜렷해진다.

Observability는 사전에 무엇을 볼지 정의해두지 않고도, 문제가 생긴 후에 데이터를 자유롭게 조합해서 원인을 찾아나갈 수 있어야 한다. 이게 가능하려면 데이터가 충분히 높은 cardinality와 dimensionality를 갖추고 있어야 하고, 그 데이터를 예측 없이 반복적으로 탐색할 수 있는 explorability가 있어야 한다.

참고: observability.md, cardinality-dimensionality.md

## Recap

Monitoring은 known-unknown(무엇을 모르는지는 아는 문제)에, Observability는 unknown-unknown(무엇을 모르는지조차 모르는 문제)에 대응한다. 분산 시스템일수록 unknown-unknown의 비중이 커지기 때문에, Monitoring만으로는 한계가 뚜렷해지고 Observability가 필요해진다.
