# dirty-page

**"디스크에 아직 반영되지 않은, 메모리에서 수정된 페이지"**

InnoDB는 데이터를 페이지(16KB) 단위로 관리한다. 쿼리가 실행되면 디스크에서 페이지를 Buffer Pool(메모리)로 올린 뒤 수정한다. 이 수정된 페이지가 아직 디스크에 쓰이지 않은 상태일 때 dirty page라고 한다.

---

## 생성 시점

COMMIT 때 생기는 게 아니다. 쿼리 실행 시점에 이미 만들어진다.

UPDATE뿐 아니라 페이지 내용이 바뀌는 모든 경우에 발생한다. INSERT는 새 행이 페이지에 쓰이고, DELETE는 행 삭제 마킹이 페이지에 기록된다. 공통점은 "메모리 ↔ 디스크 불일치" 상태가 된다는 것이다.

```
INSERT / UPDATE / DELETE 실행
  → Buffer Pool에 해당 페이지 로드 (없으면 디스크에서 읽어옴)
  → 메모리에서 수정 → dirty page 생성
  → Redo Log buffer에 변경 내용 기록

COMMIT
  → Redo Log fsync (여기서 완료 응답)
  → dirty page는 Buffer Pool에 그대로 존재

이후 background flush
  → dirty page → 데이터 파일(.ibd) 반영 → clean page
```

---

## flush 시점

dirty page가 데이터 파일에 기록되는 건 background에서 일어난다. 강제로 발생하는 경우도 있다.

- Buffer Pool이 꽉 차서 새 페이지를 올려야 할 때
- checkpoint 주기가 됐을 때
- 서버 종료 시

---

## 서버가 죽으면?

dirty page는 메모리에 있으므로 서버가 죽으면 사라진다. 하지만 Redo Log가 디스크에 남아있기 때문에 재시작 시 Redo Log를 재생해서 복구한다. dirty page가 유실돼도 안전한 이유가 이것이다.

---

## 한 줄 요약

> dirty page = 메모리에서 바뀐 페이지. 디스크 반영은 나중에. Redo Log 덕분에 안전.

참고: redo-log.md
