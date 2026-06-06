# declarative-vs-imperative-query
참고: relational-model.md

---

쿼리를 어떻게 표현하느냐의 차이다.

**명령형(imperative)** — 원하는 결과를 얻기 위해 어떻게 할지를 단계별로 지시한다. 루프를 돌고, 조건을 확인하고, 결과를 직접 조립한다. CODASYL이나 IMS의 쿼리 방식이 명령형이었다.

**선언형(declarative)** — 어떤 결과를 원하는지만 선언한다. 어떻게 가져올지는 DB가 결정한다. SQL이 대표적이다. `SELECT * FROM animals WHERE family = 'Sharks'`는 "Sharks 패밀리인 것들"이라는 조건만 선언하고, 어떤 인덱스를 쓸지, 어떤 순서로 스캔할지는 query optimizer가 결정한다.

## 선언형의 이점

query optimizer가 최적의 실행 계획을 자동으로 선택하기 때문에 개발자가 신경 쓸 필요가 없다. 새 인덱스를 추가해도 쿼리를 바꾸지 않아도 된다. DB 내부적으로 데이터를 재배치하거나 최적화해도 쿼리가 깨지지 않는다.

병렬 실행에도 유리하다. 선언형 언어는 결과의 패턴만 지정하고 알고리즘은 지정하지 않기 때문에, DB가 멀티코어나 여러 머신에 걸쳐 병렬로 실행할 여지가 생긴다. 명령형 코드는 순서가 고정되어 있어 병렬화가 어렵다.

## CSS와의 유사성

선언형 쿼리의 장점은 DB에만 국한되지 않는다. CSS는 선언형이고 JavaScript DOM 조작은 명령형이다. CSS로 `li.selected > p { background-color: blue }` 한 줄로 표현할 수 있는 것을, JavaScript로 명령형으로 구현하면 훨씬 길고 상태 관리도 직접 해야 한다.
