# cmov-instruction (조건부 이동 명령어)

조건이 참일 때만 값을 이동하는 명령어. jump를 쓰지 않아서 branch prediction miss를 피할 수 있다.

## 형태

```
cmove  S, D    # ZF=1이면 D = S (equal)
cmovne S, D    # ZF=0이면 D = S
cmovl  S, D    # SF≠OF이면 D = S (less than)
cmovg  S, D    # greater than
```

## 왜 유리한가

현대 CPU는 분기를 미리 예측해서 파이프라인에 명령어를 채워 넣는다(branch prediction). 예측이 틀리면 파이프라인을 비우고 다시 시작해야 해서 수십 클럭의 패널티가 발생한다.

`cmov`는 항상 양쪽 값을 계산해두고 조건에 따라 선택만 한다. 분기 자체가 없으므로 예측 실패 패널티가 없다.

## 컴파일러가 cmov를 선택하는 경우

```c
int max(int x, int y) {
    return x > y ? x : y;
}
```

이것이 cmov로 컴파일될 수 있다:
```asm
    movq  %rdi, %rax       # result = x
    cmpq  %rsi, %rdi       # x vs y 비교
    cmovle %rsi, %rax      # x <= y이면 result = y
    ret
```

분기가 전혀 없다.

## cmov가 부적합한 경우

양쪽 계산 비용이 크거나, 사이드 이펙트가 있거나(포인터 역참조, 함수 호출), 조건 예측률이 높은 경우엔 일반 분기가 더 빠르다. 컴파일러가 자동으로 판단한다.

## 관련 개념

- condition-codes.md 참고
- control-flow.md 참고
