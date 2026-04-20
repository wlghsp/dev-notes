# JVM 트레이닝 로드맵
## "블로그 + 발표로 실력 있는 개발자 되기"

> 문서를 만드는 게 목표가 아니다. **설명할 수 있는 수준**이 목표다.
> 각 Phase는 읽기 → 실습 → 피드백 → 블로그 발행 → 발표 순서를 반드시 지킨다.

---

## 진행 원칙

```
실습 없이 다음 Phase로 넘어가지 않는다
블로그를 발행하지 않으면 완료가 아니다
"대충 알 것 같다"는 완료가 아니다 — 설명할 수 있어야 완료다
```

---

## Phase 현황

| Phase | 주제 | 실습 | 블로그 발행 | 발표 |
|---|---|---|---|---|
| 0 | 컴퓨터 메모리 구조 (선행지식) | ⬜ | ⬜ | ⬜ |
| 1 | JVM이란 무엇인가 | ✅ javap bytecode 확인 | ⬜ | ⬜ |
| 2 | 메모리 영역 해부 | ✅ StackOverflow + OOM 직접 발생 | ⬜ | ⬜ |
| 3 | Garbage Collection | ⬜ | ⬜ | ⬜ |
| 4 | Class Loading | ⬜ | ⬜ | ⬜ |
| 5 | JIT 컴파일러 | ⬜ | ⬜ | ⬜ |
| 6 | 실전 트러블슈팅 | ⬜ | ⬜ | ⬜ |

---

## 각 Phase 상세

### Phase 1: JVM이란 무엇인가
**파일**: `jvm-what-and-why.md`
**완료 기준**: "JVM이 뭔지, bytecode가 뭔지 동료에게 5분 설명 가능"
**실습**: `javap -c HelloJVM.class` → bytecode 직접 읽기 ✅
**블로그**: ⬜ 발행 전 지호님이 읽고 피드백 필요

### Phase 2: Runtime Data Areas — 메모리 영역 해부
**파일**: `jvm-memory-areas.md`
**완료 기준**: "Heap/Stack/Method Area 차이를 코드와 함께 설명 가능"
**실습**: StackOverflowError + OutOfMemoryError 직접 발생 ✅
**블로그**: ⬜ 발행 전 지호님이 읽고 피드백 필요

### Phase 3: Garbage Collection
**파일**: `jvm-garbage-collection.md` (미작성)
**완료 기준**: "GC 로그를 보고 Minor/Major GC를 구분하고 이상 징후 감지 가능"
**실습**: GC 로그 찍기 + G1GC/ZGC 알고리즘 직접 바꿔보기

### Phase 4: Class Loading
**파일**: `jvm-class-loading.md` (미작성)
**완료 기준**: "ClassCastException이 왜 ClassLoader 문제인지 설명 가능"
**실습**: 커스텀 ClassLoader 직접 작성

### Phase 5: JIT 컴파일러
**파일**: `jvm-jit-compiler.md` (미작성)
**완료 기준**: "JIT warm-up이 왜 필요한지 수치로 보여줄 수 있음"
**실습**: `-XX:+PrintCompilation` 로그 분석, `-Xint` 성능 비교

### Phase 6: 실전 트러블슈팅
**파일**: `jvm-troubleshooting.md` (미작성)
**완료 기준**: "Heap dump를 받아서 메모리 누수 위치를 찾을 수 있음"
**실습**: jmap + jstack + VisualVM 실전 사용

---

## 지금 당장 할 것

**Phase 1 문서 읽기** → `jvm-what-and-why.md`
- 이해 안 되는 부분 표시
- 추가하고 싶은 내 경험 메모
- Claude에게 피드백 → 문서 다듬기 → 블로그 발행
