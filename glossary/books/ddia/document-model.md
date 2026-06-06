# document-model
참고: relational-model.md, schema-on-write-vs-read.md, impedance-mismatch.md

---

데이터를 JSON이나 XML 같은 자기완결적(self-contained) 문서 단위로 저장하는 모델이다. MongoDB, RethinkDB, CouchDB 등이 이 모델을 따른다.

## 장점: locality와 schema 유연성

연관된 데이터가 하나의 문서 안에 모여 있기 때문에, 전체 문서를 한 번에 읽어야 하는 경우 관계형 모델보다 조회가 빠르다. 관계형 모델에서는 여러 테이블을 조인해야 할 데이터가, 문서 모델에서는 쿼리 한 번으로 끝난다. 이것이 storage locality 이점이다.

스키마를 강제하지 않아서(schema-on-read) 데이터 구조가 다양하거나 외부 시스템에서 들어오는 경우 유연하게 대응할 수 있다.

## 한계: join과 many-to-many 관계

one-to-many 트리 구조에는 잘 맞지만, many-to-many 관계가 생기는 순간 문서 모델은 어색해진다. 문서 DB는 조인 지원이 약하거나 없기 때문에, 조인이 필요하면 애플리케이션 코드에서 여러 번 쿼리를 날려서 직접 합쳐야 한다. 이 복잡성이 DB에서 애플리케이션으로 이동하는 것뿐이다.

또한 문서 내 중첩 항목을 직접 참조하기 어렵다. "user 251의 positions 목록의 두 번째 항목"처럼 접근해야 하는데, 이는 계층형 모델의 access path와 비슷한 문제다.

## 관계형 모델과의 수렴

최근에는 두 모델이 서로 가까워지고 있다. PostgreSQL, MySQL 등 관계형 DB들이 JSON 컬럼을 지원하고, 일부 문서 DB들은 조인 기능을 추가했다. 데이터 특성에 따라 두 모델을 혼합하는 방식이 현실적인 선택이다.
