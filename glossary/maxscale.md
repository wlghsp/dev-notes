# MaxScale

MariaDB Corporation이 만든 데이터베이스 프록시. 클라이언트와 MariaDB(또는 MySQL) 사이에 위치해서 쿼리를 라우팅하고, 서버 상태를 모니터링하며, 장애 시 자동으로 역할을 재조정한다.

DB 자체를 건드리지 않는다. 복제나 동기화는 DB가 알아서 하고, MaxScale은 그 앞에서 트래픽을 제어하는 레이어다.

---

## 핵심 구성 요소

MaxScale은 크게 세 가지로 구성된다.

**Monitor**  
주기적으로 DB 서버들의 상태를 체크한다. 어느 서버가 Master인지, Slave인지, 살아있는지를 판단하는 역할이다. Master-Slave 환경에서는 `mariadbmon`을 쓰고, Galera 환경에서는 `galeramon`을 쓴다.

**Router**  
클라이언트의 쿼리를 어느 서버로 보낼지 결정한다. `readwritesplit`이 가장 많이 쓰이는 라우터로, SELECT는 Slave로, INSERT/UPDATE/DELETE는 Master로 자동으로 분리해준다. 트랜잭션 안의 SELECT는 Master로 보내는 등 세밀한 규칙도 설정 가능하다.

**Listener**  
클라이언트가 접속하는 포트를 열어두는 역할이다. 클라이언트는 MaxScale의 포트로 접속하고, MaxScale이 내부적으로 실제 DB 서버로 연결을 중계한다.

---

## 왜 쓰는가

Master 하나로 모든 트래픽을 받으면 읽기 부하가 쏠린다. Slave를 여러 개 두더라도 애플리케이션이 직접 읽기/쓰기 서버를 구분해서 접속해야 하면 코드가 복잡해진다.

MaxScale을 두면 애플리케이션은 MaxScale 하나에만 접속하고, 나머지는 MaxScale이 처리한다. Master가 죽으면 MaxScale이 Slave 중 하나를 Master로 승격시키고 클라이언트 연결을 재조정하는 자동 페일오버도 지원한다.

---

## 자동 페일오버 동작 방식

`mariadbmon`이 Master 서버에 응답이 없음을 감지하면 아래 순서로 동작한다.

1. Master 서버를 `Down` 상태로 표시
2. Slave 중 복제 지연이 가장 적은 서버를 새 Master 후보로 선택
3. 후보 서버에서 `STOP SLAVE`를 실행하고 Master 역할로 전환
4. 나머지 Slave들이 새 Master를 바라보도록 `CHANGE MASTER TO` 자동 실행
5. MaxScale 라우터가 새 Master로 쓰기 트래픽을 보내기 시작

이 과정이 수십 초 내로 자동으로 이루어진다. 단, 기존 Master가 살아서 돌아오면 자동으로 Slave로 편입(`replication_manager` 설정에 따라)되거나 수동 조치가 필요한 상황이 생길 수 있다.

---

## maxctrl — 운영 명령어

MaxScale은 `maxctrl` CLI로 런타임에 상태 확인과 제어를 한다.

```bash
# 서버 목록과 상태 확인
maxctrl list servers

# 서비스(라우터) 목록 확인
maxctrl list services

# 모니터 목록 확인
maxctrl list monitors

# 특정 서버 상세 정보
maxctrl show server <서버명>

# 서버를 수동으로 maintenance 모드로 전환 (트래픽 차단)
maxctrl set server <서버명> maintenance

# maintenance 해제
maxctrl clear server <서버명> maintenance
```

Slave 복구 후 MaxScale이 해당 서버를 다시 인식하게 하려면 `list servers`로 상태를 확인하고, `Running`과 `Slave` 역할이 표시되는지 체크한다.

---

## Master-Slave 환경에서의 한계

MaxScale은 복제 상태를 모니터링할 뿐, 복제 자체를 관리하지 않는다. Slave가 복제 에러로 멈추면 MaxScale은 해당 Slave를 라우팅 대상에서 제외하지만, 복제를 고쳐주지는 않는다.

복제 에러 원인 분석과 복구는 MariaDB 레벨에서 직접 해야 한다.  
참고: mariadb-replication-recovery.md

---

## Galera와의 차이

MaxScale은 프록시(라우터)고, Galera는 동기 복제 엔진이다.

Master-Slave 복제는 비동기라서 Slave가 Master를 따라가는 데 지연(lag)이 생기고, 에러가 나면 멈춘다. Galera는 모든 노드가 동시에 같은 데이터를 가지는 동기 복제라서 lag 개념이 없다.

MaxScale은 두 환경 모두 앞단에서 쓸 수 있다. Galera 클러스터 앞에 MaxScale을 두는 구성도 흔하며, 이 경우 `galeramon` 모니터를 사용한다.
