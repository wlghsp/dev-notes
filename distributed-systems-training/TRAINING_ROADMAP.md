# Distributed Systems 트레이닝 로드맵
## "실제 시스템이 어떻게 만들어졌는지 사례로 설명할 수 있는 개발자 되기"

> 가상의 서비스를 처음부터 설계하는 연습이 아니다 (그건 glossary/books/system-design-interview/ 트랙의 목적).
> **구글시트, Kafka, etcd 같은 실존하는 시스템이 "실제로 이렇게 만들어졌다"는 사례를 파고드는 게 목적**이다.
> 접근 방식: 익숙한 SaaS/서비스에서 질문을 시작해서, 그 아래 깔린 분산 시스템 개념과 실제 구현 선택으로 내려간다.

---

## 진행 원칙

```
개념 하나로 끝내지 않는다 — 여러 개념을 엮어서 "실제 시스템"으로 설명할 수 있어야 완료다
가상 설계가 아니라 실존 시스템 사례 중심이다 — "이런 것도 가능하다"가 아니라 "이 시스템은 실제로 이렇게 만든다"
같은 문제를 다르게 푼 시스템끼리 비교한다 — 비교해야 "왜 이렇게 만들었는가"가 드러난다
"이런 게 있다더라"가 아니라 "왜 그렇게 설계했는가"를 설명할 수 있어야 Phase 완료
```

---

## Phase 구성 전략

Phase 순서는 고정된 커리큘럼이 아니다. 궁금한 실제 시스템이 생기면 그때 새 Phase로 추가한다 (책 트랙처럼 챕터 순서에 매이지 않음). 각 Phase는 같은 문제를 서로 다르게 푼 **여러 실제 시스템을 교차 비교**하는 방식으로 진행한다. "A 시스템은 이렇게 하고, B 시스템은 저렇게 한다 — 왜 다른가"를 설명할 수 있어야 Phase 완료다.

지금까지 진행/예정된 Phase (아래 목록은 고정이 아니라 계속 추가될 수 있음):

```
Phase 1 : 실시간 협업 편집 — 구글시트/Figma/Notion은 동시 편집을 어떻게 병합하는가
Phase 2 : 합의(Consensus) — etcd/Kafka KRaft/ZooKeeper는 하나의 값에 어떻게 동의하는가
Phase 3 : 메시징/이벤트 스트리밍 — Kafka/RabbitMQ/SQS는 순서와 중복을 어떻게 다루는가
Phase 4 : 데이터 분산/파티셔닝 — DynamoDB/Cassandra/CDN은 데이터를 어떻게 쪼개고 어디 두는가
```

---

## Phase 현황

- Phase 1 : 실시간 협업 편집 (OT/CRDT) — ✅ 문서 작성 완료
- Phase 2 : 합의(Consensus) — ✅ 문서 작성 완료
- Phase 3 : 메시징/이벤트 스트리밍 — ✅ 문서 작성 완료
- Phase 4 : 데이터 분산/파티셔닝 — ⬜

새 Phase가 필요해지면 "각 Phase 상세" 아래에 이어서 추가한다.

---

## 각 Phase 상세

### Phase 1: 실시간 협업 편집 — 구글시트는 동시 편집을 어떻게 병합하는가
**파일**: operational-transformation.md, crdt.md, realtime-collaboration-architecture.md
**완료 기준**: "WebSocket이 왜 충돌 해결과 무관한지, OT와 CRDT가 각각 어떤 상황에 맞는지 동료에게 설명 가능"
**핵심 질문**:
- WebSocket만으로 왜 동시 편집 문제가 해결되지 않는가?
- OT와 CRDT는 각각 "누가 순서를 중재하는가"에 대해 다른 답을 낸다 — 그 답은 무엇인가?
- 오프라인 편집 후 재접속 시나리오에서 왜 CRDT가 유리한가?
- 구글시트와 Figma가 서로 다른 방식을 택했다면, 그 이유는 각자의 어떤 제약 때문일까?

---

### Phase 2: 합의(Consensus) — 여러 노드가 하나의 값에 어떻게 동의하는가
**파일**: consensus.md, quorum.md (본 트레이닝 폴더) / raft.md (glossary/topics/distributed-systems/, 기존 작성분)
**완료 기준**: "왜 노드 수를 홀수로 구성하는지, 리더가 죽었을 때 무슨 일이 일어나는지 설명 가능"
**핵심 질문**:
- Consensus가 왜 필요한가? 그냥 하나의 서버만 두면 안 되는 이유는?
- Quorum(과반수) 개념이 왜 등장하는가?
- 리더 선출 도중에 쓰기 요청이 오면 어떻게 되는가?
- etcd, Kafka KRaft가 Raft를 쓰는 이유는 각각 무엇을 보장받고 싶어서인가?

---

### Phase 3: 메시징/이벤트 스트리밍 — Kafka/RabbitMQ/SQS는 순서와 중복을 어떻게 다루는가
**파일**: delivery-guarantee.md, partitioning-in-messaging.md (본 트레이닝 폴더)
**완료 기준**: "Kafka, RabbitMQ, SQS가 각각 순서·중복을 다르게 보장하는 이유를, 트레이드오프 관점에서 설명 가능"
**핵심 질문**:
- At-most-once, at-least-once, exactly-once는 각각 무엇을 포기하고 무엇을 얻는가?
- 완벽한 exactly-once가 전달 계층만으로는 왜 불가능한가?
- Kafka 파티션과 SQS FIFO 메시지 그룹은 같은 문제를 어떻게 다르게 풀었는가?
- 파티션 키를 잘못 고르면 왜 hot partition이 생기는가?

---

### Phase 4: 데이터 분산/파티셔닝 — DynamoDB/Cassandra는 데이터를 어떻게 쪼개고 어디 두는가
**glossary 파일**: (미작성)
**완료 기준**: "샤딩 키를 잘못 고르면 왜 문제가 생기는지, Consistent Hashing이 무엇을 해결하는지 실제 시스템 예시로 설명 가능"
**핵심 질문**:
- 샤딩 키를 잘못 고르면 어떤 문제가 생기는가?
- Consistent Hashing은 일반 해싱과 비교해 왜 필요한가?
- 노드가 추가/제거될 때 DynamoDB, Cassandra는 각각 데이터를 어떻게 재분배하는가?

---

## 참고
- 각 Phase는 "실제 시스템이 왜 이렇게 만들어졌는가"라는 질문에서 출발하고, 가상 설계 연습이 아니라 실존 사례 중심이다
- Phase 순서는 고정 커리큘럼이 아니다 — 궁금한 시스템이 생기면 그때 새 Phase로 추가한다
- 산출물은 distributed-systems-training/ 아래 개념별 단독 파일로 쌓이고, 필요하면 여러 개념을 엮은 종합 아키텍처 문서를 별도로 만든다
- 문서 완성 후 곧바로 이해도 테스트를 던지지 않는다 (CLAUDE.md 트레이닝 규칙과 동일: 문서 완성 → 블로그 발행 → 테스트)
