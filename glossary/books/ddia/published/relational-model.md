# relational-model
참고: document-model.md, declarative-vs-imperative-query.md

---

데이터를 테이블(relation)과 행(tuple)으로 표현하는 모델이다. Edgar Codd가 1970년에 제안했고, 1980년대 중반 SQL과 함께 데이터 저장의 표준이 됐다.

## 핵심 아이디어

구현 세부사항을 숨기고 깔끔한 인터페이스만 노출한다는 것이다. 이전의 계층형 모델(hierarchical model)이나 네트워크 모델(CODASYL)은 데이터에 접근하려면 access path를 직접 따라가야 했다. 어떤 경로로 데이터에 접근할지 애플리케이션 코드가 알아야 했고, 새로운 쿼리 방식이 생기면 코드를 전부 다시 짜야 했다.

관계형 모델은 이 문제를 query optimizer에게 넘겼다. 개발자는 "무엇을 원하는가"만 선언하고(SQL), 어떤 인덱스를 쓸지, 어떤 순서로 조인할지는 DB가 알아서 결정한다. 새 인덱스를 추가해도 기존 쿼리를 바꿀 필요가 없다.

## 적합한 경우

many-to-one, many-to-many 관계가 많은 데이터에 강하다. 정규화(normalization)를 통해 중복을 제거하고 ID로 참조하는 방식이 자연스럽기 때문이다. 조인이 쉽고, 참조 무결성을 DB 수준에서 보장할 수 있다.
