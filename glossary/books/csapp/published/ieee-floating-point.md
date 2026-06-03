# ieee-floating-point (IEEE 754 부동소수점)

실수를 이진수로 표현하는 국제 표준. 거의 모든 프로세서와 언어가 이 표준을 따른다.

## 구조

부동소수점 수는 세 필드로 구성된다.

부호 비트(s) — 1비트. 0이면 양수, 1이면 음수.
지수부(exp) — 지수 값을 저장. bias를 더해서 저장한다.
가수부(frac) — 소수점 이하 비트. 정규화된 수에서 정수 부분 1은 생략된다.

single precision (float, 32비트):
- 부호 1비트 + 지수 8비트 + 가수 23비트
- bias = 127

double precision (double, 64비트):
- 부호 1비트 + 지수 11비트 + 가수 52비트
- bias = 1023

## 값 계산 (정규화된 수)

V = (−1)^s × M × 2^E

- E = exp 필드 값 − bias
- M = 1 + frac (숨겨진 1비트, implied leading 1)

예: 12345를 single precision으로 변환하면 1.1000000111001₂ × 2¹³

## 특수 값

exp가 전부 0 — denormalized 수. 0에 가까운 매우 작은 수. M = 0 + frac.
exp가 전부 1, frac = 0 — ±∞
exp가 전부 1, frac ≠ 0 — NaN (Not a Number). 0/0, ∞−∞ 등의 결과.

## 정밀도 한계

0.1을 이진수로 정확하게 표현할 수 없다. 10진수 0.1은 이진수로 0.000110011001100...으로 무한히 반복된다. 이 때문에 부동소수점 연산은 반올림 오차가 발생한다.

```
0.1 + 0.2 == 0.3  // false (부동소수점 비교의 함정)
```

## 관련 개념

- floating-point-rounding.md 참고 — 반올림 모드
- twos-complement.md 참고 — 정수 표현과 대비
