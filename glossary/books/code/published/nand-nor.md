# nand-nor (NAND와 NOR)

NAND는 AND의 반전, NOR는 OR의 반전이다. 이 두 게이트는 각각 단독으로 AND/OR/NOT을 포함한 모든 논리 게이트를 만들 수 있는 범용(universal) 게이트다.

## NAND 진리표

AND의 출력을 뒤집은 것이다. 둘 다 1일 때만 0, 나머지는 1.

```
A  B  출력
0  0   1
0  1   1
1  0   1
1  1   0
```

## NOR 진리표

OR의 출력을 뒤집은 것이다. 둘 다 0일 때만 1, 나머지는 0.

```
A  B  출력
0  0   1
0  1   0
1  0   0
1  1   0
```

## 왜 범용 게이트인가

NAND 하나로 NOT, AND, OR를 전부 만들 수 있다.

NOT — NAND의 두 입력에 같은 신호를 연결하면 NOT이 된다.
```
A NAND A = NOT A
```

AND — NAND 뒤에 NOT(= NAND로 만든 NOT)을 붙이면 AND.

OR — 드모르간 법칙으로 OR를 NAND 조합으로 표현할 수 있다.

즉 NAND 게이트만 있으면 세상의 모든 논리 회로를 만들 수 있다. 실제로 현대 반도체 공정에서 NAND 게이트를 기본 단위로 설계하는 경우가 많다. 하나의 게이트 타입만 최적화하면 되기 때문이다.

## 관련 개념

- logic-gate.md 참고
- boolean-logic.md 참고
