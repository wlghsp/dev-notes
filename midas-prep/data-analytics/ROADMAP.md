# Athena / Presto / DBT 학습 로드맵

마이다스 백엔드(CDC 기술자) 채용공고 준비용. "Athena, DBT를 활용한 데이터 분석 환경 연계 개발"이 주요 업무, "Athena, Presto, DBT 등 데이터 분석 및 모델링 경험"이 우대조건.

glossary/topics/iceberg/ 학습을 마치고 이어서 진행. Iceberg가 "데이터를 어떻게 테이블로 관리하는가"라면, 이쪽은 "그 테이블을 어떻게 쿼리하고 변환 파이프라인으로 조립하는가".

## 배경 — 이 셋이 어떻게 이어지는가

CDC로 수집한 데이터가 S3에 Iceberg 테이블로 쌓인다. Presto/Athena는 그 테이블에 SQL로 쿼리를 날리는 엔진이고, DBT는 그 SQL 쿼리들을 "원본 테이블 → 정제된 테이블 → 분석용 테이블"처럼 여러 단계로 이어 붙여 관리하는 도구다.

## 개념 목록 (예상 — 학습하며 실제 생성 목록으로 교체)

- [x] presto-architecture — Coordinator/Worker 구조, 쿼리가 분산 실행되는 방식
- [x] presto-iceberg-connector — Presto가 Iceberg 메타데이터를 읽어 쿼리하는 방식 (pruning 연결)
- [x] dbt-model — DBT의 model이 SQL SELECT를 어떻게 테이블/뷰로 관리하는가
- [ ] athena-vs-presto — Athena가 Presto(Trino) 기반의 서버리스 쿼리 서비스인 이유
- [ ] dbt-dag — 모델 간 의존성이 DAG로 관리되는 방식 (ref() 함수)
- [ ] dbt-materialization — table/view/incremental 등 materialization 전략의 차이

## 진행 상태

1차 3개(presto-architecture → presto-iceberg-connector → dbt-model) 완료.
흐름: Presto는 Coordinator/Worker로 쿼리를 분산 실행하는 엔진(presto-architecture) → 그 Worker가 Iceberg 메타데이터를 먼저 읽어 스캔 범위를 좁힌다(presto-iceberg-connector) → 그 위에서 DBT가 SELECT 문들을 ref()로 엮어 원본→가공→분석 테이블 체인을 관리한다(dbt-model).

dbt-model에서 dbt-dag를 언급했으니 다음은 그쪽으로 이어가면 자연스러움. athena-vs-presto는 이미 presto-architecture를 이해했으면 상대적으로 가벼운 주제.

## 완료 후 참고 (실제 생성된 파일만 기록 — 예상 아님)

- presto-architecture.md
- presto-iceberg-connector.md
- dbt-model.md
