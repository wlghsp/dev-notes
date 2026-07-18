# Iceberg 학습 로드맵

마이다스 백엔드(CDC 기술자) 채용공고 준비용. "S3 기반 데이터 레이크, Iceberg 테이블 기반 데이터 관리 구조 설계"가 주요 업무로 명시되어 있는데 현재 지식이 전무해서 시작.

CDC(Debezium/Kafka)는 이미 어느 정도 경험이 있어 보류. Iceberg부터 진행.

## 왜 필요한가 (배경)

기존 Hive 테이블 방식은 파티션을 디렉토리 구조로 관리해서 스키마/파티션을 바꾸기 어렵고, 어떤 파일이 테이블에 속하는지 파일시스템 리스팅으로 알아내야 했다. Iceberg는 테이블 상태를 메타데이터 레이어로 분리해서 이 문제를 해결하는 테이블 포맷이다. CDC로 실시간 수집한 데이터를 S3에 쌓고 Athena/DBT로 분석하는 구조에서, 그 데이터를 안전하게 관리하는 레이어가 Iceberg.

## 개념 목록 (예상 — 학습하며 실제 생성 목록으로 교체)

- [x] iceberg-table-format — 테이블 포맷이란 무엇인가, Hive 테이블과 무엇이 다른가
- [x] iceberg-snapshot — 스냅샷 기반 버저닝, time travel
- [x] iceberg-manifest — manifest file/manifest list가 메타데이터를 관리하는 구조
- [x] iceberg-schema-evolution — 스키마 변경을 안전하게 처리하는 방식
- [x] iceberg-partition-evolution — 파티션 전략을 나중에 바꿀 수 있는 이유
- [x] iceberg-compaction — 작은 파일 문제와 compaction

## 진행 상태

완료. 6개 파일로 Iceberg 핵심 개념 전체를 커버.
흐름: 테이블은 메타데이터가 파일 목록을 관리(table-format) → 그 상태의 버전이 스냅샷(snapshot) → 스냅샷이 가리키는 파일 목록의 실체(manifest) → 컬럼 ID 기반이라 안전한 스키마 변경(schema-evolution) → 파티션 스펙도 메타데이터라 안전한 파티션 변경(partition-evolution) → CDC로 쌓인 작은 파일을 합치는 운영 작업(compaction).

다음 주제: Athena/Presto/DBT — Iceberg 테이블을 실제로 쿼리·분석하는 도구 계층.

## 완료 후 참고 (실제 생성된 파일만 기록 — 예상 아님)

- iceberg-table-format.md
- iceberg-snapshot.md
- iceberg-manifest.md
- iceberg-schema-evolution.md
- iceberg-partition-evolution.md
- iceberg-compaction.md
