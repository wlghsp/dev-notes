# LLVM 생태계

## 한 줄 요약

**컴파일러를 만드는 컴파일러 인프라.** 언어(C, C++, Rust, Swift 등)와 실행 환경(x86, ARM, WebAssembly 등) 사이의 중간 다리 역할을 하는 모듈형 컴파일러 툴체인.

---

## 왜 알아야 하는가 (Java/Spring 맥락에서)

GraalVM Native Image가 LLVM 생태계의 개념을 차용해 만들어졌다. JVM의 클라우드 환경 제약(느린 기동, 큰 메모리)을 해결하는 방향이 "AOT 컴파일 → 네이티브 바이너리"인데, 이 구조가 LLVM이 오랫동안 해온 것과 같다.

참고: jvm_cloud_constraints.md

---

## 기본 구조: 3단계 컴파일 파이프라인

LLVM 이전 컴파일러들은 언어마다, 플랫폼마다 처음부터 다시 만들어야 했다. LLVM은 이 문제를 **중간 표현(IR, Intermediate Representation)** 으로 해결했다.

```
[Frontend]          [Middle-end]          [Backend]
소스코드     →      LLVM IR       →      기계어
(C, Rust, Swift)   (공통 형태)          (x86, ARM, WASM)
```

**Frontend**: 각 언어의 문법을 파싱해서 LLVM IR로 변환
**Middle-end**: IR을 최적화 (언어도, 플랫폼도 모름. 순수 최적화만)
**Backend**: IR을 각 CPU 아키텍처 기계어로 변환

새 언어를 만들고 싶으면 Frontend만 구현하면 된다. LLVM의 최적화와 모든 플랫폼 지원이 자동으로 따라온다. 새 CPU를 지원하고 싶으면 Backend만 구현하면 된다.

---

## LLVM IR: 공통 중간 언어

LLVM IR은 어떤 언어로 짰는지, 어떤 CPU에서 돌릴지 모르는 상태의 코드다. 플랫폼 독립적이고, SSA(Static Single Assignment) 형태를 따른다.

```llvm
; C 코드: int add(int a, int b) { return a + b; }
; LLVM IR로 변환된 형태
define i32 @add(i32 %a, i32 %b) {
  %result = add i32 %a, %b
  ret i32 %result
}
```

사람이 직접 쓰지 않고 컴파일러가 생성한다. 중요한 건 **이 형태를 거치면 어떤 언어든, 어떤 플랫폼이든 같은 최적화 파이프라인을 공유한다**는 것이다.

---

## 주요 구성 요소

**Clang**
C / C++ / Objective-C 언어의 LLVM Frontend. GCC를 대체하는 컴파일러. Apple의 macOS, iOS 개발 도구체인이 Clang 기반이다.

**lld**
LLVM 링커. 컴파일된 오브젝트 파일들을 하나의 실행 파일로 묶는 역할.

**LLDB**
LLVM 디버거. Clang으로 컴파일한 코드의 디버깅 도구.

**Sulong**
LLVM IR을 실행하는 인터프리터. GraalVM이 C, Rust로 작성된 코드를 JVM 위에서 실행할 수 있는 게 이 컴포넌트 덕분이다.

---

## Java 생태계와의 연결: GraalVM

GraalVM은 LLVM의 아이디어를 JVM 세계에 가져온 프로젝트다.

JVM도 LLVM처럼 IR 개념이 있다. Java 바이트코드(.class 파일)가 JVM의 IR이다.

```
[Java/Kotlin 소스]  →  [바이트코드 (.class)]  →  [JIT 기계어]
    (javac)              (JVM의 IR)              (실행 시 변환)
```

GraalVM Native Image는 이 흐름을 바꾼다.

```
[Java 소스]  →  [바이트코드]  →  [AOT 컴파일]  →  [네이티브 바이너리]
                                 (빌드 타임)         (JVM 없이 실행)
```

AOT(Ahead-Of-Time) 컴파일은 LLVM이 C/Rust를 처리하는 방식과 같다. 빌드 시점에 모든 최적화를 끝내고, 실행 시점에는 바로 기계어를 돌린다.

결과:
- JVM 없이 단독 실행 가능한 바이너리
- 기동 시간 수십 ms (JVM 대비 수십 배 빠름)
- 메모리 사용량 대폭 감소
- JIT 워밍업 없음 — 처음부터 최적 성능

---

## AOT vs JIT 비교

**JIT (Just-In-Time)** — 전통 JVM 방식
- 실행하면서 Hot Path를 감지하고 그때그때 기계어로 컴파일
- 런타임 정보를 반영한 최적화 가능 (실제 입력 패턴 기반)
- 기동 시간 느림, 워밍업 필요

**AOT (Ahead-Of-Time)** — LLVM / GraalVM Native 방식
- 빌드 타임에 미리 모든 코드를 기계어로 컴파일
- 런타임 정보 없이 정적 분석만 가능 → 일부 최적화 한계
- 기동 시간 빠름, 워밍업 없음

장기 실행 서버에서는 JIT가 더 높은 피크 성능을 낼 수 있다. 짧게 뜨고 죽는 컨테이너 환경에서는 AOT가 유리하다.

---

## LLVM이 영향을 준 언어들

LLVM Frontend를 구현한 언어들이 LLVM의 최적화와 멀티플랫폼 지원을 공짜로 얻는다.

- **Rust**: LLVM을 컴파일러 백엔드로 사용. Rust의 성능이 C에 근접하는 이유 중 하나
- **Swift**: Apple이 LLVM 기반으로 만든 언어
- **Kotlin Native**: Kotlin을 iOS, 임베디드 등에서 실행하기 위한 LLVM 기반 컴파일
- **WebAssembly**: LLVM의 Backend 중 하나. C/Rust 코드를 WASM으로 컴파일 가능

---

## 요약

- LLVM은 컴파일러 인프라 — 언어와 플랫폼 사이의 공통 IR을 제공
- Frontend(언어 파싱) → IR(최적화) → Backend(기계어) 3단계 구조
- 새 언어는 Frontend만, 새 플랫폼은 Backend만 구현하면 LLVM 생태계에 편입
- GraalVM Native Image가 이 AOT 컴파일 방식을 JVM에 적용
- JVM의 클라우드 환경 제약(느린 기동, 큰 메모리)을 AOT로 해결하는 방향이 LLVM과 연결됨
- Rust, Swift, Kotlin Native 등이 LLVM을 백엔드로 사용
