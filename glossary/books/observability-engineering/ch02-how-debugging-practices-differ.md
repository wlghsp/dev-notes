# Chapter 2 — How Debugging Practices Differ Between Observability and Monitoring

이 챕터에서 생성된 키워드 파일: institutional-knowledge-debugging.md, confirmation-bias-debugging.md, monitoring-debugging-failure-modes.md

---

## 이 챕터가 답하는 질문

1장은 Observability를 정의하고, Monitoring이 known-unknown에만 대응한다는 한계를 짚었다. 이 챕터는 그 한계가 실제 디버깅 현장에서 구체적으로 어떤 모습으로 나타나는지를 보여준다.

## 아침에 대시보드를 보는 엔지니어의 모습

책은 익숙한 장면으로 시작한다. 엔지니어가 출근해서 대시보드 2~30개짜리 그래프를 훑어본다. 각 그래프가 정확히 뭘 측정하는지는 모르지만, 오랫동안 봐와서 "이 그래프가 이렇게 움직이면 캐싱 서버 문제"라는 패턴을 몸으로 알고 있다. 이게 institutional-knowledge-debugging.md에서 다루는 방식이다.

문제는 이 직관이 그 시스템에만 묶여 있다는 점이다. 완전히 다른 아키텍처 앞에 같은 사람을 데려다놔도 같은 직관이 통하지 않는다.

## 직관 기반 트러블슈팅이 무너지는 네 가지 패턴

책은 네 가지 구체적 실패 패턴으로 이 한계를 보여준다. Insufficient correlation(질문과 데이터의 간극), Not drilling down(더 세밀하게 못 파고듦), Tool-hopping(도구를 옮겨다님), Context switching(맥락을 사람이 직접 이어붙임). 각각의 자세한 내용은 monitoring-debugging-failure-modes.md 참고.

인덱스를 추가한 효과를 확인하려 해도 host 단위 대시보드로는 user/query 단위로 못 쪼개보고(insufficient correlation), 한 샤드의 이상 징후가 실제로 전체 문제인지 국소적 문제인지 더 파고들 수 없고(not drilling down), 에러 스파이크 하나를 쫓다가 대시보드→로그→trace를 오가며 request id를 손으로 복사-붙여넣기 하게 된다(tool-hopping, context switching).

## 이 모든 게 결국 확증 편향으로 이어진다

세 사례의 공통점은, 엔지니어가 먼저 직감으로 가설을 세우고("MySQL 문제인 것 같다") 그 가설을 확인하러 간다는 점이다. confirmation-bias-debugging.md에서 다루듯, 이 과정에는 그 가설이 틀렸을 가능성을 검토하는 단계가 없다. 확인된 것처럼 보이는 지점에서 바로 멈추기 때문에, 실제로는 증상일 뿐인 걸 원인으로 착각하기 쉽다.

## Institutional Knowledge에서 공유된 데이터로

이 챕터의 결론은, Observability가 이 문제를 "더 좋은 직관"으로 푸는 게 아니라 애초에 직관에 의존할 필요를 없애는 방식으로 푼다는 것이다. Monitoring 기반 팀에서는 제일 오래 있던 사람이 제일 뛰어난 디버거지만, Observability 기반 팀에서는 제일 호기심 많은 사람이 뛰어난 디버거가 될 수 있다. 데이터가 cardinality와 dimensionality를 충분히 갖추고 한 곳에 모여 있으면, 도구를 옮겨다니며 맥락을 사람이 직접 이어 붙이지 않아도 된다. 암묵적 경험이었던 것이 누구나 확인 가능한 명시적 데이터로 바뀐다.
