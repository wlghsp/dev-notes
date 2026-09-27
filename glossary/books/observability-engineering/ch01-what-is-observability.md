# Chapter 1 — What Is Observability?

이 챕터에서 생성된 키워드 파일: cardinality-dimensionality.md, known-unknown-vs-unknown-unknown.md, three-pillars.md

(observability.md, signal.md는 이 챕터 내용을 기반으로 스터디 예습 시작 시점에 먼저 만들어둔 파일이라, 이 문서와 내용이 상당 부분 겹친다.)

---

## Observability라는 단어의 원래 뜻

Observability는 1960년 Rudolf E. Kálmán이 제어 이론에서 정의한 용어다. 어떤 시스템의 외부 출력만 관찰해서 내부 상태를 얼마나 잘 추론할 수 있는가를 뜻했다. 소프트웨어 쪽에서는 이 정의를 그대로 쓰기보다, "새로운 코드를 배포하지 않고도 시스템이 어떤 상태에 놓여 있는지 이해하고 설명할 수 있는 정도"로 재해석해서 쓴다.

## 소프트웨어 시스템이 Observable하려면

책은 이를 판단하는 구체적인 기준을 리스트로 제시한다. 예측하지 못한 이상 상황도 막다른 골목 없이 계속 파고들 수 있는가, 특정 사용자 한 명이 겪는 경험을 짚어낼 수 있는가, 집계된 값부터 개별 요청까지 자유롭게 오갈 수 있는가, 문제를 미리 예측해서 모니터를 만들어두지 않고도 답을 찾을 수 있는가. 이 기준들을 종합하면 결국 두 가지 속성으로 수렴한다: cardinality-dimensionality.md에서 다루는 cardinality(데이터 값의 고유함)와 dimensionality(데이터가 가진 필드 수)가 모두 높아야 한다는 것.

## Monitoring이 못하는 것

Monitoring은 사전에 정한 조건(스레시홀드)을 감시하는 방식이다. known-unknown-vs-unknown-unknown.md에서 다루듯, 이 방식은 "무엇을 모르는지는 알고 있는" known-unknown에는 잘 작동하지만, 한 번도 본 적 없는 실패 형태인 unknown-unknown에는 무력하다.

과거의 모놀리식 시스템은 실패 방식이 비교적 예측 가능해서 known-unknown 위주였다. 분산 시스템, 컨테이너, 폴리글랏 퍼시스턴스가 일반화된 지금은 조합 가능한 실패의 경우의 수가 사실상 무한에 가까워졌고, unknown-unknown의 비중이 압도적으로 커졌다. Monitoring 기반 도구, 특히 metric은 구조적으로 cardinality가 높은 데이터를 감당하지 못하기 때문에 이 변화를 따라가지 못한다.

## "Three Pillars"는 왜 틀린 프레임인가

Trace/Metric/Log 세 가지를 갖추면 observability가 달성된다는 설명은 저자들이 명시적으로 비판하는 지점이다. 세 데이터 타입을 그냥 나열하는 것 자체가 목적이 되어버리면, 정작 중요한 조건들을 놓치게 된다. 자세한 내용은 three-pillars.md 참고.

## 이 챕터 다음에 오는 것

이 챕터는 "Observability란 무엇인가, 왜 필요한가"를 정의했을 뿐, 그걸 실제로 어떤 데이터 형태로 구현하는지는 아직 다루지 않았다. 그 답은 Part II(5장)의 structured-event.md에서 이어진다.
