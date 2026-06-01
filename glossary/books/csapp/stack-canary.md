# stack-canary (스택 카나리)

버퍼 오버플로우를 탐지하기 위해 스택 프레임에 삽입하는 무작위 값. 함수 반환 직전에 값이 변조됐는지 확인한다.

## 원리

함수 진입 시, 지역 배열과 반환 주소 사이에 무작위 값(canary)을 삽입한다. 함수가 반환하기 직전에 그 값이 바뀌었는지 확인한다. 바뀌었으면 오버플로우가 발생했다는 의미이므로 즉시 프로그램을 종료한다.

```
[ 이전 프레임      ]
[ 반환 주소        ]  ← 공격 목표
[ canary 값        ]  ← 여기가 바뀌면 오버플로우 감지
[ 지역 배열 buf    ]  ← 오버플로우 발생 위치
[ %rsp             ]
```

## 어셈블리 패턴

GCC가 자동으로 삽입한다.

```asm
# 함수 진입 시
movq %fs:40, %rax      # 무작위 canary 값을 fs 세그먼트에서 읽음
movq %rax, -8(%rbp)    # 스택에 저장

# 함수 반환 직전
movq -8(%rbp), %rax    # 저장된 canary 읽기
xorq %fs:40, %rax      # 원래 값과 XOR — 같으면 0
je   .ok               # 0이면 정상
call __stack_chk_fail  # 다르면 프로그램 종료
.ok:
    ret
```

## canary 값의 특성

- 프로그램 실행마다 무작위로 생성된다
- 보통 최하위 바이트가 0x00이다 — C 문자열 함수는 null byte에서 멈추기 때문에 gets/strcpy로는 canary를 넘기기 어렵다
- `%fs:40`(TLS, Thread Local Storage)에 저장되어 스택에서 직접 읽기 어렵다

## GCC에서의 동작

char 배열이 있는 함수에 자동으로 삽입된다. 명시적으로 제어하려면:
- `-fstack-protector`: 취약해 보이는 함수에만 적용 (기본값)
- `-fstack-protector-all`: 모든 함수에 적용
- `-fno-stack-protector`: 비활성화

## 다른 방어 기법과의 조합

canary 단독으로는 공격자가 canary 값을 먼저 읽어서 우회하는 방법이 있다. ASLR, NX bit와 함께 사용하면 방어가 훨씬 강해진다.

## 관련 개념

- buffer-overflow.md 참고 — 공격 원리와 전체 방어 전략
- stack-frame.md 참고 — canary가 프레임 어디에 위치하는지
