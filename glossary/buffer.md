# Buffer (버퍼)

**한 줄 정의**: 속도가 다른 두 주체 사이에서 데이터를 임시로 들고 있는 공간.

---

## 왜 필요한가?

빠른 쪽이 느린 쪽을 매번 직접 기다리면 낭비다.

```
CPU 속도:  ~1 ns
Disk 속도: ~10 ms
→ CPU가 Disk를 기다리면 99.9999% 시간을 놀게 됨
```

버퍼를 두면 느린 쪽 접근 횟수를 줄이고, 빠른 쪽은 버퍼에서 빠르게 가져간다.

---

## 같은 개념, 다른 이름들

| 이름 | 어디서 | 뭘 버퍼링 |
|---|---|---|
| **Buffer Pool** | MySQL InnoDB | Disk Page → 메모리 |
| **Producer Buffer** | Kafka | 메시지 → 브로커 전송 전 모아두기 |
| **OS Page Cache** | Linux 커널 | Disk → 메모리 |
| **TCP Send Buffer** | 네트워크 | 앱 데이터 → 네트워크 카드 |
| **JVM TLAB** | JVM G1GC | 객체 할당용 스레드 전용 버퍼 |

이름만 다를 뿐, 전부 같은 원리: **느린 쪽 접근 횟수를 줄이기 위해 빠른 쪽에 임시 보관.**

---

## 관련 개념
- [DB Buffer Pool](../db-internals/db-storage-basics.md#4-buffer-pool--디스크를-덜-읽기-위한-핵심-장치)
