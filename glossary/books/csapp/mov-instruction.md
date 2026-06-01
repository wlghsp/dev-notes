# mov-instruction (데이터 이동 명령어)

데이터를 한 위치에서 다른 위치로 복사하는 명령어. x86-64에서 가장 많이 쓰이는 명령어 중 하나다.

## 기본 MOV 계열

크기에 따라 네 가지가 있다.
```
movb  — 1바이트
movw  — 2바이트 (word)
movl  — 4바이트 (double word)
movq  — 8바이트 (quad word)
```

소스(S)에서 목적지(D)로 복사한다: `D ← S`

소스는 immediate/register/memory 모두 가능. 목적지는 register/memory만 가능. 메모리→메모리는 불가.

## 64비트 즉값 이동

`movq`의 소스 immediate는 32비트 범위로 제한된다. 64비트 상수를 직접 넣으려면 `movabsq`를 써야 한다.
```
movabsq $0x0011223344556677, %rax
```

## Zero-extension (MOVZ 계열)

작은 소스를 큰 목적지로 옮길 때 나머지를 0으로 채운다.
```
movzbq  — byte → quad word (zero-extend)
movzwq  — word → quad word (zero-extend)
movzbl  — byte → double word
```

## Sign-extension (MOVS 계열)

작은 소스를 큰 목적지로 옮길 때 나머지를 부호 비트로 채운다.
```
movsbq  — byte → quad word (sign-extend)
movswq  — word → quad word
movslq  — double word → quad word
cltq    — %eax → %rax (sign-extend), 오퍼랜드 없음
```

## movl의 암묵적 zero-extension

`movl`로 레지스터에 쓰면 상위 4바이트가 자동으로 0이 된다. x86-64 설계 규칙이다.

```
movabsq $0x0011223344556677, %rax  # rax = 0011223344556677
movl    $-1, %eax                   # rax = 00000000FFFFFFFF (상위 4바이트 0)
movq    $-1, %rax                   # rax = FFFFFFFFFFFFFFFF
```

## 관련 개념

- operand-types.md 참고
- x86-64-registers.md 참고
