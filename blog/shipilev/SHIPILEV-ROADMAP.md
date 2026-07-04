# Aleksey Shipilëv (shipilev.net) 정리 로드맵

Aleksey Shipilëv 블로그에서 JVM 내부 동작(객체 메모리 레이아웃, GC, JIT 등)을 다루는 글을 가져와 정리하는 트랙.
CLAUDE.md 블로그 규칙에 맞춰 정리한다 (표 금지, 개념 중심, 파일명 텍스트 참조).

## 정리 완료

- java-objects-inside-out.md — Java Objects Inside Out — 객체 헤더, 필드 패킹, 압축 참조
- identity-hash-code.md — JVM Anatomy Quark #26: Identity Hash Code — hashCode()의 실제 위치와 계산 방식
- object-initialization-costs.md — JVM Anatomy Quark #7: Initialization Costs — 객체 생성 비용의 실체

## 다음에 정리할 후보

JVM Anatomy Quarks 시리즈(shipilev.net/jvm/anatomy-quarks/)에서 후보를 고른다.

- Arrays of Wisdom of the Ancients (2016) — 배열 관련 저수준 최적화
- JMM(Java Memory Model)과 pragmatics 관련 발표/글
- 그 외 Anatomy Quarks 시리즈 중 GC, JIT, 동시성 관련 주제

다음에 "shipilev 로드맵에서 다음거 정리해줘" 라고 요청하면, 위 후보 또는 Anatomy Quarks 목록 페이지에서 새 주제를 찾아 진행한다.
