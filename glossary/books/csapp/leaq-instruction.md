# leaq-instruction (load effective address)

`leaq`는 메모리 주소를 계산해서 레지스터에 저장하는 명령어다. 이름은 "메모리 읽기"처럼 생겼지만 실제로 메모리를 읽지 않는다.

## 핵심 차이

```
movq  (%rax), %rdx    # rax가 가리키는 메모리 값을 rdx에 저장
leaq  (%rax), %rdx    # rax 자체의 값(주소)을 rdx에 저장
```

`leaq`는 주소 계산 결과를 레지스터에 넣을 뿐, 메모리 접근을 하지 않는다.

## 포인터 생성 용도

C의 `&` 연산자와 같은 역할을 한다.
```c
long *p = &x;  // x의 주소를 p에 저장
```
이것이 `leaq`로 컴파일된다.

## 컴파일러가 산술에 활용하는 방법

`leaq`는 복잡한 주소 계산 형식(`Imm(rb, ri, s)`)을 그대로 산술 연산으로 활용할 수 있다. 컴파일러가 곱셈과 덧셈을 leaq로 표현하는 이유다.

```c
long scale(long x, long y, long z) {
    return x + 4*y + 12*z;
}
```

```asm
scale:
    leaq  (%rdi,%rsi,4), %rax    # x + 4*y
    leaq  (%rdx,%rdx,2), %rdx    # z + 2*z = 3*z
    leaq  (%rax,%rdx,4), %rax    # (x+4*y) + 4*(3*z) = x + 4*y + 12*z
    ret
```

leaq 세 번으로 곱셈 없이 `x + 4y + 12z`를 계산했다. 스케일 팩터(1/2/4/8)를 이용해서 ×2, ×4, ×8을 표현한다.

## 관련 개념

- operand-types.md 참고 — 주소 계산 공식
- x86-64-registers.md 참고
