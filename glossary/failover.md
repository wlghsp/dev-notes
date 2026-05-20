# Failover

운영 중인 서버(Primary)가 장애로 응답 불가 상태가 됐을 때, 대기 중인 서버(Secondary)가 그 역할을 대신 맡아 서비스를 이어가는 것.

핵심은 "자동으로" 또는 "빠르게" 전환된다는 점이다. 사람이 수동으로 서버를 바꾸는 건 failover가 아니라 switchover라고 구분해서 부른다.

---

## DB 환경에서의 Failover

Master-Slave 구조에서 Master가 죽으면 Slave 중 하나가 새 Master가 되어야 한다. 이 전환 과정이 DB failover다.

전환이 완료되기 전까지 쓰기 요청은 처리되지 않는다. 얼마나 빨리 전환하느냐가 서비스 다운타임을 결정한다.

---

## Failover 과정에서 발생하는 문제들

**데이터 유실 가능성**  
비동기 복제 환경에서 Master가 죽으면, 마지막으로 쓴 데이터가 Slave에 아직 전달되지 않았을 수 있다. 새 Master로 승격된 Slave에는 그 데이터가 없다. 참고: binlog.md

**Split-brain**  
기존 Master가 실제로 죽은 게 아니라 네트워크 단절로 응답이 없는 것일 수 있다. 이 상태에서 Slave를 Master로 승격하면 Master가 두 개가 된다. 양쪽에 쓰기가 발생하면 데이터가 분기된다.

**복제 재연결 비용**  
새 Master가 정해지면 나머지 Slave들이 새 Master를 바라보도록 `CHANGE MASTER TO`를 실행해야 한다. 이 과정에서 GTID 또는 binlog position이 맞지 않으면 복제가 다시 깨진다.

---

## MaxScale에서의 자동 Failover

MaxScale의 `mariadbmon` 모니터가 자동 failover를 담당한다. Master 무응답을 감지하면 아래 순서로 전환한다.

1. Master를 `Down`으로 표시하고 쓰기 트래픽 차단
2. 복제 지연이 가장 적은 Slave를 새 Master 후보로 선택
3. 후보 서버에서 `STOP SLAVE` 실행 후 Master 역할로 전환
4. 나머지 Slave들이 새 Master를 바라보도록 자동으로 `CHANGE MASTER TO` 실행
5. 라우터가 새 Master로 쓰기 트래픽을 보내기 시작

`mariadb-replication-recovery.md`에서 이 이후 복구 절차를 다룬다.

---

## Failover vs Switchover

failover는 장애 상황에서 강제로 전환하는 것이고, switchover는 계획된 유지보수를 위해 의도적으로 역할을 바꾸는 것이다.

switchover는 기존 Master가 살아있는 상태에서 진행하기 때문에 데이터 유실 없이 깔끔하게 전환할 수 있다. MaxScale에서는 `maxctrl call command mariadbmon switchover <모니터명>`으로 실행한다.

---

## RTO와 RPO

failover의 품질을 측정하는 두 가지 지표다.

RTO(Recovery Time Objective) — 장애 발생 후 서비스가 복구되기까지 허용 가능한 최대 시간. failover가 빠를수록 RTO가 낮다.

RPO(Recovery Point Objective) — 장애 발생 시점으로부터 얼마나 이전 데이터까지 복구할 수 있는가. 비동기 복제 환경에서는 복제 lag만큼 데이터가 유실될 수 있으므로 RPO가 0이 아니다.
