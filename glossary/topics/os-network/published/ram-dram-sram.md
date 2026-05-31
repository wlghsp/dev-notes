# ram-dram-sram

RAM(Random Access Memory)은 임의 위치에 접근 가능한 메모리의 총칭이다. 구현 방식에 따라 DRAM과 SRAM으로 나뉜다.

## DRAM (Dynamic RAM)

커패시터에 전하를 충전해서 비트를 저장한다. 시간이 지나면 전하가 방전되기 때문에 주기적으로 재충전(refresh)해야 한다. 그래서 "Dynamic"이다.

셀 구조가 단순해서 집적도가 높고 가격이 싸다. 우리가 "서버 메모리 32GB", "RAM 8GB"라고 할 때 그 메모리가 DRAM이다. 메인 메모리로 쓰인다.

## SRAM (Static RAM)

플립플롭 회로로 비트를 저장한다. refresh가 필요 없고 속도가 DRAM보다 훨씬 빠르다. 대신 셀 구조가 복잡해서 집적도가 낮고 비싸다. CPU 캐시(L1/L2/L3)가 SRAM이다.

## 왜 구분이 필요한가

"물리 메모리가 부족하다"고 할 때 그 메모리는 DRAM이다. CPU 캐시(SRAM)는 용량이 수 MB 수준이라 부족하다고 말하는 대상이 아니다. OS가 관리하고, swap과 OOM Killer가 개입하는 대상은 DRAM이다.

문서에서 DRAM이라고 명시하는 이유가 이것이다. 그냥 RAM이라고 쓰면 SRAM과 혼동될 수 있다.

## 관련 개념

- 메모리 계층 전체 구조는 storage-hierarchy.md 참고
- DRAM 부족 시 OS 동작은 oom-killer.md 참고
- DRAM 부족을 디스크로 완충하는 방법은 swap.md 참고
