# write-amplification
참고: lsm-tree.md, b-tree.md, compaction.md

---

하나의 논리적 쓰기(애플리케이션이 요청한 쓰기 1회)가 실제 물리적 디스크 쓰기를 여러 번 유발하는 현상이다. 쓰기 증폭(write amplification) 비율은 실제 디스크 쓰기 바이트 / 논리 쓰기 바이트로 표현한다.

## 발생 원인

LSM-tree에서는 compaction 때문에 발생한다. 데이터를 한 번 쓰면 이후 여러 레벨의 compaction을 거치며 반복해서 읽히고 다시 쓰인다. 레벨이 깊어질수록 같은 데이터가 더 많이 재작성된다.

B-tree에서는 페이지 분할과 WAL 때문에 발생한다. 하나의 행을 삽입해도 WAL에 먼저 쓰고, 리프 페이지도 덮어쓰고, 분할이 일어나면 부모 페이지까지 수정해야 한다.

## 왜 문제인가

SSD는 쓰기 횟수에 한계가 있다(P/E cycle). Write amplification이 높으면 SSD 수명이 단축된다. 또한 디스크 쓰기 대역폭을 소모하므로, write amplification이 높은 시스템은 동일한 하드웨어에서 처리할 수 있는 쓰기 throughput이 낮아진다.

## 트레이드오프

write amplification을 낮추면 compaction 빈도가 줄어들지만, 읽기 성능이 저하될 수 있다(더 많은 SSTable 파일을 확인해야 함). compaction을 더 공격적으로 하면 읽기는 빨라지지만 write amplification이 커진다. 어느 수준의 write amplification을 감수할지는 워크로드(읽기 heavy vs 쓰기 heavy)에 따라 다르다.
