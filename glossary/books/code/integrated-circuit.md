# 집적회로 (Integrated Circuit, IC)

트랜지스터, 저항, 배선을 하나의 실리콘 조각 위에 통째로 만든 회로. 칩(chip)이라고도 부른다. 1958~1959년에 Jack Kilby(Texas Instruments)와 Robert Noyce(Fairchild Semiconductor)가 독립적으로 발명했다.

## 왜 등장했는가

트랜지스터는 릴레이보다 훨씬 작고 빠르지만, 수백만 개를 손으로 배선하는 것은 여전히 비용이 크고 오류가 많았다. 회로를 구성하는 특정 조합(게이트 → 플립플롭 → 가산기 → 셀렉터)이 반복적으로 쓰인다는 점에 착안해, 이 조합을 미리 실리콘 위에 새겨놓는 아이디어가 나왔다.

## 제조 방식

실리콘 웨이퍼 위에 얇은 층을 쌓고 정밀하게 도핑하고 식각하는 과정을 반복해서 만든다. 초기 개발 비용은 크지만, 대량 생산할수록 단가가 낮아진다.

## 집적도의 발전

- SSI (Small Scale Integration): 게이트 10개 미만
- MSI (Medium Scale Integration): 10~100개
- LSI (Large Scale Integration): 100~5000개
- VLSI (Very Large Scale Integration): 5000~50,000개
- SLSI, ULSI: 그 이상

1965년 Gordon Moore는 칩 하나에 올라가는 트랜지스터 수가 매년 두 배씩 늘어났다고 관찰했다. 이것이 무어의 법칙(Moore's Law)이다. 나중에 18개월마다 두 배로 수정됐다.

## TTL과 CMOS

1970년대에 가장 많이 쓰인 두 가지 IC 계열이다.

TTL(Transistor-Transistor Logic)은 빠르지만 5볼트 전원이 필요하고 전력 소모가 크다. 7400 시리즈가 대표적이다.

CMOS(Complementary Metal-Oxide Semiconductor)는 TTL보다 느리지만 전력 소모가 훨씬 적고 동작 전압 범위가 넓다(3~18V). 배터리 구동 기기에 유리하다. 4000 시리즈가 대표적이다.

## 관련 개념

- 참고: transistor.md
- 참고: vacuum-tube.md
- 참고: logic-gate.md
