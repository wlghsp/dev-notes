# control-flow (제어 흐름)

기계어 수준에서 if/while/for/switch 같은 조건부 실행이 어떻게 구현되는지. 조건 코드를 검사하고 jump 명령어로 실행 흐름을 바꾼다.

## jump 명령어

조건 없이 또는 조건에 따라 실행 위치를 바꾼다.

```
jmp  label    # 무조건 점프
je   label    # ZF=1이면 점프 (equal)
jne  label    # ZF=0이면 점프 (not equal)
jl   label    # SF≠OF이면 점프 (less than, signed)
jg   label    # ZF=0 && SF=OF이면 점프 (greater than, signed)
jb   label    # CF=1이면 점프 (below, unsigned)
```

## if-else의 어셈블리 패턴

```c
if (x > 0) { y = 1; } else { y = -1; }
```

컴파일되면 대략:
```asm
    testq  %rdi, %rdi     # x와 0 비교
    jle    .else           # x <= 0이면 else로 점프
    movq   $1, %rax        # y = 1
    jmp    .done
.else:
    movq   $-1, %rax       # y = -1
.done:
    ret
```

## while 루프의 어셈블리 패턴

GCC는 while을 do-while처럼 변환한다. 먼저 조건을 체크하는 점프를 루프 끝으로 보내고, 본문을 실행하고, 다시 조건 체크로 돌아온다.

```c
while (i < n) { sum += a[i]; i++; }
```

```asm
    jmp  .test             # 먼저 조건 확인으로 점프
.loop:
    # 루프 본문
.test:
    cmpq %rsi, %rdi        # i < n 비교
    jl   .loop             # 참이면 루프로
```

## switch의 구현 — jump table

case 수가 많으면 컴파일러가 jump table을 만든다. 값을 인덱스로 사용해서 테이블에서 주소를 읽고 한 번에 점프한다. O(1) 시간.

```asm
jmp  *.L7(,%rdi,8)    # .L7[rdi * 8] 주소로 점프
```

case 수가 적거나 값이 듬성듬성하면 if-else 체인으로 변환한다.

## 관련 개념

- condition-codes.md 참고 — 분기의 기반이 되는 플래그
- cmov-instruction.md 참고 — 분기 없이 조건부 실행
