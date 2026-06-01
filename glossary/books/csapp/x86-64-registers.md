# x86-64-registers (x86-64 레지스터)

x86-64 CPU에는 64비트 범용 레지스터 16개가 있다. 각 레지스터는 역할이 정해져 있고, 하위 32/16/8비트에도 별도 이름으로 접근할 수 있다.

## 16개 레지스터와 역할

```
%rax — 반환값 (return value)
%rbx — callee-saved
%rcx — 4번째 인자
%rdx — 3번째 인자
%rsi — 2번째 인자
%rdi — 1번째 인자
%rbp — callee-saved (프레임 포인터로도 사용)
%rsp — 스택 포인터 (stack pointer)
%r8  — 5번째 인자
%r9  — 6번째 인자
%r10 — caller-saved
%r11 — caller-saved
%r12 — callee-saved
%r13 — callee-saved
%r14 — callee-saved
%r15 — callee-saved
```

## 하위 비트 접근

하나의 레지스터를 크기별로 다른 이름으로 접근한다.

```
%rax  — 64비트 전체
%eax  — 하위 32비트
%ax   — 하위 16비트
%al   — 하위 8비트
```

`%r8`~`%r15`는 `%r8d`(32비트), `%r8w`(16비트), `%r8b`(8비트)로 접근한다.

## 32비트 연산의 특별한 동작

32비트 연산(`movl`, `addl` 등)은 상위 32비트를 자동으로 0으로 초기화한다. 다른 크기의 연산은 나머지 바이트를 건드리지 않는다. 이는 IA32 → x86-64 확장 시 도입된 규칙이다.

## %rsp는 특별하다

`%rsp`는 항상 스택의 최상단(가장 낮은 주소)을 가리킨다. push/pop, call/ret 명령어가 자동으로 이 레지스터를 조작한다. 임의로 변경하면 스택이 깨진다.

## 관련 개념

- calling-convention.md 참고 — callee/caller-saved 구분
- stack-frame.md 참고 — %rsp와 %rbp의 역할
- operand-types.md 참고
