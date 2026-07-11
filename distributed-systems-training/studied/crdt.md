# CRDT (Conflict-free Replicated Data Type)

여러 복제본(replica)에 동시에 변경이 가해져도, 어떤 순서로 병합하든 수학적으로 항상 같은 최종 상태로 수렴하도록 설계된 데이터 구조.

Figma, Notion 등 최근 실시간 협업 툴들이 선호하는 방식이다.

## 핵심 문제

OT(operational-transformation.md)는 중앙 서버가 연산 순서를 중재해야 충돌 없이 동작한다. 하지만 서버 없이(P2P) 동기화하거나, 오프라인 상태에서 각자 편집한 뒤 나중에 합치는 경우 "누가 순서를 정할지"가 애매해진다.

CRDT는 순서를 중재하는 주체 없이도, 각 클라이언트가 받은 변경을 아무 순서로 적용해도 결과가 항상 같아지도록 데이터 구조 자체를 설계해서 이 문제를 없앤다.

## 수렴이 가능한 이유

CRDT의 병합 연산은 수학적으로 다음 성질을 만족하도록 설계된다.

- 교환법칙(commutative): merge(A, B) = merge(B, A)
- 결합법칙(associative): merge(merge(A, B), C) = merge(A, merge(B, C))
- 멱등성(idempotent.md): merge(A, A) = A

이 세 성질을 만족하면 변경이 어떤 순서로, 몇 번 도착하든 최종 상태가 하나로 수렴한다. 이걸 Strong Eventual Consistency라고 부른다 (eventual-consistency.md 참고).

## 대표적인 예시

**G-Counter (증가만 하는 카운터)**
각 노드가 자신의 증가분만 따로 기록하고, 전체 값은 모든 노드의 증가분을 합해서 계산한다. 노드마다 자기 것만 건드리므로 충돌이 발생하지 않는다.

**LWW-Register (Last-Write-Wins)**
각 값에 타임스탬프를 붙여서, 병합 시 타임스탬프가 더 큰 값이 이긴다. 구현이 단순하지만 동시 쓰기 중 하나가 무조건 유실된다는 단점이 있다.

**텍스트 편집용 CRDT (RGA, Peritext 등)**
텍스트의 각 글자에 고유하고 순서를 비교할 수 있는 ID를 부여한다. 삽입/삭제를 이 ID 기반으로 표현하면, 동시 편집이 도착해도 ID 순서로 정렬해서 항상 같은 결과를 만들 수 있다.

## 특징

- 중앙 서버 없이 병합 가능 → 오프라인 편집, P2P 동기화에 강하다
- 데이터 구조 자체에 병합 로직이 내장되어 있어 서버 로직이 단순해진다
- 대신 각 원소마다 삭제해도 완전히 지우지 못하고 "삭제됨" 표시(tombstone)로 남기는 등 메타데이터 오버헤드가 크다
- 자료구조 설계가 까다롭고, 자료구조마다 새로 증명해야 한다

참고: operational-transformation.md, idempotency.md, eventual-consistency.md, realtime-collaboration-architecture.md
