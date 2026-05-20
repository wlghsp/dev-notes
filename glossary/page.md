# page

**"데이터베이스가 데이터를 읽고 쓰는 최소 단위"**

InnoDB는 데이터를 바이트 단위로 다루지 않는다. 16KB 덩어리(페이지) 단위로 읽고 쓴다. 레코드 하나를 조회해도 그 레코드가 속한 페이지 전체를 디스크에서 읽어온다.

---

## 같은 페이지, 다른 이름

혼란스러운 이유는 동일한 16KB 덩어리를 어디에 있느냐에 따라 다르게 부르기 때문이다.

```
디스크의 데이터 파일(.ibd)
  → disk page (또는 그냥 page)

Buffer Pool(메모리)에 올라온 상태
  → memory page (또는 buffer page)

Buffer Pool에서 수정된 상태 (디스크에 아직 안 쓴)
  → dirty page

디스크와 동일한 상태
  → clean page
```

물리적으로 같은 데이터다. 위치와 상태에 따라 이름이 달라진다.

---

## 왜 페이지 단위인가

디스크 I/O는 비싸다. 바이트 하나 읽으려고 디스크를 건드리면 너무 느리다. 어차피 인접한 데이터를 같이 쓸 가능성이 높으니 16KB씩 묶어서 한 번에 읽고 쓰는 게 효율적이다. 이걸 Buffer Pool에 캐싱해두면 다음 접근 시 디스크를 안 건드려도 된다.

---

## OS page와의 차이

OS도 메모리를 page 단위로 관리한다 (보통 4KB). DB page와는 별개다.

- OS page: 가상 메모리 관리 단위 (4KB)
- DB page: InnoDB 데이터 관리 단위 (16KB)

InnoDB의 16KB page는 OS page 4개에 걸쳐 있는 셈이다. 이름이 같지만 다른 개념이다.

---

## 한 줄 요약

> page = InnoDB가 다루는 16KB 덩어리. 디스크/메모리/수정 여부에 따라 disk page, buffer page, dirty page로 불린다.

참고: dirty-page.md, redo-log.md
