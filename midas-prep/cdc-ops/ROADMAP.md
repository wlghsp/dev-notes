# CDC 운영 심화 학습 로드맵

마이다스 백엔드(CDC 기술자) 채용공고 준비용. CDC 개념 자체(blog/cdc/, glossary/topics/distributed-systems/published/cdc.md 등)는 이미 학습했음. 이 로드맵은 "설계·운영"까지 요구하는 공고 수준에 맞춰 실전 운영 관점만 보강하는 용도.

보류 중 — 다른 주제(Iceberg, Athena/Presto/DBT) 먼저 진행하고 이어서 착수.

## 배경

기존 학습은 CDC가 무엇이고 Debezium/binlog가 어떻게 동작하는지에 집중되어 있다. 공고는 "CDC 기반 데이터 파이프라인과 고가용성 백엔드 시스템을 설계"를 요구하므로, 장애 상황에서 파이프라인이 어떻게 무너지고 어떻게 복구되는지가 갭이다.

## 개념 목록 (예상 — 학습하며 실제 생성 목록으로 교체)

- [ ] debezium-offset-recovery — Debezium/Kafka Connect가 재시작 시 어디부터 다시 읽는지, offset 유실 시나리오
- [ ] kafka-connect-rebalancing — Connect 클러스터에서 커넥터/태스크가 재분배되는 조건과 그 순간의 이벤트 처리 공백
- [ ] replication-lag-impact — MySQL 복제 지연이 CDC 파이프라인의 지연으로 어떻게 이어지는지
- [ ] cdc-idempotent-consumer — 중복 이벤트(at-least-once)를 downstream에서 멱등하게 처리하는 설계 패턴

## 진행 상태

보류. Resilience4j 로드맵과 함께 대기 중.

## 완료 후 참고 (실제 생성된 파일만 기록 — 예상 아님)

(아직 없음)
