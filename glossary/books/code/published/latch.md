# latch (래치)

Clock 신호의 레벨(level)에 따라 데이터를 저장하거나 통과시키는 회로. flip-flop.md의 level-triggered D형 플립플롭이 바로 래치다.

## 동작

Clock이 1인 동안은 Data 입력이 출력에 그대로 반영된다(transparent). Clock이 0이 되면 마지막 Data 값이 고정된다(opaque). 이 "고정"된 상태가 래치(latch)다. 자물쇠처럼 값을 잠근다.

## 8비트 래치

1비트 래치 8개를 하나의 박스로 묶으면 8비트 래치가 된다. Clock 입력은 공유하고, D0~D7의 8개 입력이 들어와서 Q0~Q7의 8개 출력으로 나간다.

Clock이 1이 되면 8비트 값이 저장된다. Clock이 다시 0이 되면 그 값이 유지된다. 8비트 덧셈기의 중간 결과를 저장해두고 다음 계산에 활용하는 용도로 쓸 수 있다.

## 래치 vs 플립플롭

래치는 Clock 레벨에 민감하다. Clock이 1인 동안 Data가 바뀌면 출력도 바뀐다. 플립플롭(edge-triggered)은 Clock의 전환 순간에만 반응한다. 더 예측 가능한 동작이 필요한 곳에서는 edge-triggered 플립플롭을 쓴다.

## 관련 개념

- flip-flop.md 참고
- feedback.md 참고
