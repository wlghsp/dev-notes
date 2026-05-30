# Write 전략

버퍼가 있을 때 데이터를 언제, 어떻게 디스크에 쓸 것인가에 대한 전략.

---

## Write-through

버퍼에 쓰는 동시에 디스크에도 바로 쓴다.

```
App → Buffer에 쓰기
           ↓ 즉시
         Disk에 쓰기
```

버퍼와 디스크가 항상 동기 상태를 유지한다. 크래시가 나도 데이터 유실이 없다. 대신 모든 쓰기가 디스크 속도에 묶이므로 느리다.

---

## Write-behind (Write-back)

버퍼에만 써두고 디스크 쓰기는 나중으로 미룬다.

```
App → Buffer에 쓰기 (Dirty 상태)
           ↓ 나중에 Flush
         Disk에 쓰기
```

디스크 접근을 모아서 한 번에 처리하므로 빠르다. 대신 Flush 전에 크래시가 나면 버퍼에 있던 데이터는 유실된다. MySQL InnoDB Buffer Pool, OS Page Cache가 이 방식이다.

참고: buffer.md — Dirty, Flush, 크래시 시 유실 위험

---

## Write-around

버퍼를 건너뛰고 디스크에 직접 쓴다.

```
App → Disk에 직접 쓰기 (버퍼 우회)
```

한 번 쓰고 다시 읽을 일이 없는 데이터에 쓴다. 버퍼를 오염시키지 않아서 자주 읽는 데이터가 버퍼에서 밀려나는 문제를 막는다. 단점은 첫 읽기 때 반드시 디스크를 거쳐야 한다는 점이다.

---

## 선택 기준

안전이 우선이면 Write-through, 성능이 우선이면 Write-behind, 일회성 대용량 쓰기면 Write-around.

실제로는 Write-behind를 기본으로 쓰되, 유실 위험을 WAL(Write-Ahead Log)이나 Replication으로 보완하는 조합이 가장 흔하다.
