# Kafka: The Definitive Guide (2nd Edition) Glossary 진도표

참고: Kafka: The Definitive Guide, 2nd Edition — Gwen Shapira, Todd Palino, Rajini Sivaram, Krit Petty (O'Reilly, 2021)
원본 파일: assets/kafka/Kafka The Definitive Guide Real-Time Data and Stream Processing at Scale, Second Edition by Gwen Shapira, Todd Palino, Rajini Sivaram, Krit Petty (z-lib.org).pdf

이 로드맵은 책을 챕터 순서대로 읽어나가며 새로 생기는 키워드를 챕터 단위로 관리하기 위한 것이다.

목차는 PDF 북마크(아웃라인) 메타데이터에서 추출했다.

---

## Chapter 1 — Meet Kafka

Pub/Sub 메시징의 기원, Kafka 핵심 개념(메시지/배치, 스키마, 토픽/파티션, 프로듀서/컨슈머, 브로커/클러스터), Kafka가 필요했던 이유, LinkedIn에서의 탄생 배경.

- [x] pub-sub-messaging.md — 직접 연결의 문제와 브로커를 둔 pub/sub 패턴, Kafka와의 관계
- [x] kafka-message-and-batch.md — 메시지/key/batch, 처리량-지연시간 트레이드오프
- [x] kafka-schema.md — 스키마가 프로듀서-컨슈머 결합을 없애는 방식, Avro
- [x] kafka-topic-and-partition.md — 토픽/파티션 구조, 파티션이 확장성과 복제를 가능하게 하는 이유, stream 용어
- [x] kafka-producer-and-consumer.md — 프로듀서/컨슈머 기본 개념, offset
- [x] kafka-broker-and-cluster.md — 브로커/클러스터/컨트롤러, 파티션 leader/follower
- [x] kafka-retention.md — 보존 기간(시간/용량 기준)과 로그 컴팩션
- [x] kafka-multi-cluster-mirroring.md — 멀티 클러스터가 필요한 이유, MirrorMaker

## Chapter 2 — Installing Kafka

환경 설정(OS, Java, ZooKeeper), 브로커 설치/설정, 하드웨어 선택(디스크/메모리/네트워크/CPU), 클라우드 배포, 클러스터 구성, 운영 시 고려사항(GC, 데이터센터 레이아웃).

(아직 진행 전)

## Chapter 3 — Kafka Producers: Writing Messages to Kafka

프로듀서 구조, 동기/비동기 전송, 주요 설정(acks, linger.ms, batch.size, enable.idempotence 등), 직렬화(커스텀, Avro), 파티셔닝, 헤더, 인터셉터, 쿼터.

(아직 진행 전)

## Chapter 4 — Kafka Consumers: Reading Data from Kafka

컨슈머 그룹과 파티션 재분배, 폴 루프, 주요 설정, 커밋과 오프셋(자동/동기/비동기 커밋), 리밸런스 리스너, 역직렬화, 그룹 없는 단독 컨슈머.

(아직 진행 전)

## Chapter 5 — Managing Apache Kafka Programmatically

AdminClient 개요와 생명주기, 토픽/설정/컨슈머 그룹 관리, 클러스터 메타데이터, 파티션 추가/레코드 삭제/리더 선출/리플리카 재할당 같은 고급 관리 작업.

(아직 진행 전)

## Chapter 6 — Kafka Internals

클러스터 멤버십, 컨트롤러(KRaft 포함), 복제, 요청 처리(Produce/Fetch), 물리적 스토리지(파티션 할당, 파일 포맷, 인덱스, 컴팩션).

(아직 진행 전)

## Chapter 7 — Reliable Data Delivery

신뢰성 보장의 정의, 복제, 브로커 설정(복제 팩터, Unclean Leader Election, min.insync.replicas), 신뢰성 있는 프로듀서/컨슈머 사용법, 시스템 신뢰성 검증.

(아직 진행 전)

## Chapter 8 — Exactly-Once Semantics

멱등 프로듀서의 동작 원리와 한계, 트랜잭션(사용 사례, 동작 원리, 트랜잭셔널 ID와 펜싱), 트랜잭션 성능.

(아직 진행 전)

## Chapter 9 — Building Data Pipelines

데이터 파이프라인 설계 시 고려사항(적시성, 신뢰성, 처리량, 데이터 포맷 등), Kafka Connect 개요와 실습(File/MySQL-Elasticsearch 커넥터), Single Message Transformation, Connect 대안들.

(아직 진행 전)

## Chapter 10 — Cross-Cluster Data Mirroring

크로스 클러스터 미러링 사용 사례, 멀티클러스터 아키텍처(Hub-and-Spoke, Active-Active, Active-Standby, Stretch Cluster), MirrorMaker 설정/배포/튜닝, 기타 미러링 솔루션.

(아직 진행 전)

## Chapter 11 — Securing Kafka

보안 프로토콜, 인증(SSL, SASL), 암호화(End-to-End), 인가(AclAuthorizer), 감사, ZooKeeper 보안, 플랫폼 보안.

(아직 진행 전)

## Chapter 12 — Administering Kafka

토픽/컨슈머 그룹/설정 관리를 위한 커맨드라인 도구, 프로듀싱/컨슈밍 콘솔 도구, 파티션 관리, 위험한(unsafe) 운영 작업.

(아직 진행 전)

## Chapter 13 — Monitoring Kafka

메트릭 기본기, SLO/SLI, 브로커 메트릭(Under-Replicated Partitions 진단 포함), 클라이언트 메트릭, 컨슈머 랙 모니터링, End-to-End 모니터링.

(아직 진행 전)

## Chapter 14 — Stream Processing

스트림 처리 정의와 핵심 개념(토폴로지, 시간, 상태, 스트림-테이블 이중성, 윈도우, 처리 보장), 스트림 처리 설계 패턴, Kafka Streams 실습과 아키텍처, 스트림 처리 프레임워크 선택 기준.

(아직 진행 전)

---

## Appendix A — Installing Kafka on Other Operating Systems

Windows(WSL/Native Java), macOS(Homebrew/수동 설치) 환경에서의 설치.

## Appendix B — Additional Kafka Tools

종합 플랫폼, 클러스터 배포/관리, 모니터링/데이터 탐색, 클라이언트 라이브러리, 스트림 처리 관련 생태계 도구 목록.
