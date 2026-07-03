# data-warehouse
참고: oltp-vs-olap.md, column-oriented-storage.md

---

OLAP 워크로드 전용으로 분리된 별도의 데이터베이스다. 운영 시스템(OLTP DB)에서 데이터를 주기적으로 추출해서 분석에 최적화된 형태로 적재한다.

## 왜 분리하는가

분석 쿼리는 수백만 행을 스캔하고 집계한다. 이 쿼리를 운영 DB에서 직접 돌리면 인덱스를 안 타고 전체 테이블을 읽느라 운영 트래픽까지 느려진다. 데이터 웨어하우스는 분석 전용이므로 OLAP에 맞는 방식(컬럼 지향 저장, 스타 스키마 등)으로 최적화할 수 있다.

## ETL

Extract-Transform-Load. 운영 DB에서 데이터를 추출(Extract)하고, 분석에 맞는 형태로 변환(Transform)한 뒤, 웨어하우스에 적재(Load)하는 과정이다. 변환 단계에서 여러 운영 시스템의 데이터를 합치거나, 컬럼명을 통일하거나, 형식을 정규화한다. 보통 배치로 주기적으로 실행되거나, CDC(change-data-capture.md)로 실시간에 가깝게 처리된다.

## 스타 스키마

데이터 웨어하우스에서 자주 쓰는 데이터 모델이다. 중심에 fact 테이블이 있고, 그 주변에 dimension 테이블들이 붙는 구조다. fact 테이블은 "이벤트(판매, 클릭, 세션)"를 행으로 가지며 외래 키로 dimension 테이블을 참조한다. dimension 테이블은 "누가, 언제, 어디서, 무엇을"에 해당하는 속성을 담는다. 분석 쿼리는 대개 fact 테이블에서 시작해 dimension으로 join한다.

분석 DB 제품으로는 Redshift, BigQuery, Snowflake, ClickHouse 등이 있으며, 이들은 모두 column-oriented-storage.md 방식을 쓴다.
