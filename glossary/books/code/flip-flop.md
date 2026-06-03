# flip-flop (플립플롭)

1비트 정보를 저장하는 회로. 출력이 두 가지 안정 상태(0 또는 1) 중 하나를 유지하며, 특정 입력 신호가 들어올 때만 상태가 바뀐다.

## R-S 플립플롭

가장 기본적인 플립플롭. NOR 게이트 두 개를 교차 피드백으로 연결해 만든다.

Set 입력이 1이 되면 Q 출력이 1로 고정된다. Reset 입력이 1이 되면 Q 출력이 0으로 고정된다. 두 입력이 모두 0이면 마지막 상태를 유지한다. Set과 Reset이 동시에 1이 되는 건 허용되지 않는다.

## D형 플립플롭 (level-triggered)

R-S 플립플롭의 "두 입력 동시 1" 문제를 해결한 버전. 입력을 Data 하나로 줄이고 Hold That Bit(= Clock)이 1일 때만 Data 값이 저장된다.

Clock이 1인 동안에는 Data 변화가 출력에 반영된다. Clock이 0이 되면 마지막 Data 값이 고정된다.

## D형 플립플롭 (edge-triggered)

level-triggered의 문제는 Clock이 1인 동안 Data가 바뀌면 출력도 바뀐다는 것이다. edge-triggered는 Clock이 0→1로 전환되는 순간에만 Data를 캡처한다. 그 이후 Data가 바뀌어도 출력은 유지된다.

R-S 플립플롭 두 단계를 직렬로 연결하되, 첫 번째 단계의 Clock을 반전시켜 구현한다. 첫 번째 단계는 Clock=0일 때 Data를 받아들이고, Clock이 1로 바뀌는 순간 두 번째 단계로 전달된다.

## 주파수 분주기

edge-triggered D형 플립플롭의 Q-bar 출력을 Data 입력으로 되먹이면 Clock이 0→1로 바뀔 때마다 Q 출력이 토글된다. Clock 주파수의 절반 속도로 출력이 바뀐다. 이것을 주파수 분주기(frequency divider)라 한다. 이 플립플롭 여러 개를 연결하면 이진 카운터가 된다.

## 관련 개념

- feedback.md 참고
- latch.md 참고
- clock-signal.md 참고
- counter.md 참고
