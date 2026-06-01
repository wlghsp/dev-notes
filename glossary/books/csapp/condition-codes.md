# condition-codes (조건 코드)

산술/논리 연산 후 CPU가 자동으로 설정하는 1비트 플래그들. 조건부 분기와 조건부 이동의 기반이다.

## 4가지 주요 플래그

CF (Carry Flag) — 부호 없는 오버플로우 발생 시 1. unsigned 연산의 오버플로우 감지.

ZF (Zero Flag) — 연산 결과가 0이면 1. `a == b`는 `a - b`를 계산하고 ZF를 확인한다.

SF (Sign Flag) — 결과의 최상위 비트(부호 비트)가 1이면 1. 결과가 음수인지 나타낸다.

OF (Overflow Flag) — 부호 있는 오버플로우 발생 시 1. signed 연산의 오버플로우 감지.

## 플래그를 설정하는 명령어들

대부분의 산술/논리 명령어가 연산 후 플래그를 자동으로 설정한다.
- ADD, SUB, AND, OR, XOR, INC, DEC, NEG, NOT 등

`leaq`는 주소 계산 명령어라 플래그를 설정하지 않는다.

## CMP와 TEST — 플래그만 설정

결과를 저장하지 않고 플래그만 설정하는 전용 명령어.

```
cmpq S1, S2    # S2 - S1 계산 (결과 버림), 플래그만 설정
testq S1, S2   # S1 & S2 계산 (결과 버림), 플래그만 설정
```

`cmpq %rsi, %rdi`는 `%rdi - %rsi`를 계산하고 그 결과에 따라 플래그를 설정한다. 두 값이 같으면 결과가 0이므로 ZF=1.

`testq %rax, %rax`는 자기 자신과 AND. 결과가 0이면 %rax가 0이라는 뜻이므로 ZF=1.

## SET 명령어

플래그 조합을 읽어서 0 또는 1을 레지스터에 저장한다.

```
sete  %al    # ZF=1이면 %al=1 (equal)
setne %al    # ZF=0이면 %al=1 (not equal)
setl  %al    # SF≠OF이면 %al=1 (less than, signed)
setb  %al    # CF=1이면 %al=1 (below, unsigned)
```

## 관련 개념

- control-flow.md 참고 — 플래그를 이용한 분기
- cmov-instruction.md 참고 — 플래그 기반 조건부 이동
