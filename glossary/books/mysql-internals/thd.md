# THD (Thread Descriptor)

THD는 스레드 디스크립터다. 클라이언트 커넥션을 처리하는 스레드 하나의 상태 전체를 담는다. MySQL 소스에서 가장 자주 등장하는 타입이다.

정의는 `sql/sql_class.h`, 구현은 `sql/sql_class.cc`. 코드에서는 거의 항상 `THD*` 포인터 변수명 `thd`로 참조된다. 특정 멤버를 코드에서 찾고 싶으면 `grep "thd->var_name" sql/*.cc`로 바로 찾을 수 있다.

## 클라이언트 커넥션만을 위한 게 아니다

THD는 클라이언트 커넥션 스레드 외에도 여러 곳에서 생성된다. 복제 슬레이브 스레드, delayed insert 스레드, bootstrap 모드(설치 시 시스템 테이블 생성)에서도 THD 객체가 만들어진다. 커넥션이 없는데도 THD가 있을 수 있다는 뜻이다.

MySQL 개발자들은 thread와 connection을 거의 동의어로 쓴다. THD 관점에서는 맞는 말이다.

## 왜 거의 모든 함수가 THD*를 첫 인자로 받는가

THD에는 현재 실행 중인 쿼리 텍스트(`query`), parse tree 디스크립터(`lex`), 메모리 풀(`mem_root`), 네트워크 커넥션 디스크립터(`net`), 트랜잭션 상태(`transaction`), 경고/에러 목록(`warn_list`), 현재 선택된 DB(`db`), 사용자 변수(`user_vars`), 권한 정보(`master_access`, `db_access`) 등이 담겨 있다.

쿼리 하나를 처리하는 데 필요한 모든 문맥이 THD 안에 있다. 그래서 어떤 함수든 "지금 이 쿼리가 어떤 상태에서 실행되고 있는가"를 알려면 THD를 봐야 한다.

## version 4.1에서 생긴 변화

4.1에서 THD의 일부 멤버들이 새로 만들어진 `Statement` 클래스로 이동했다. `lex`, `query`, `query_length`, `free_list`, `mem_root` 등이 Statement 소속이 됐고, THD가 Statement의 서브클래스가 됐다. 코드에서 이 멤버들이 `thd->` 로 접근되는 건 THD가 Statement를 상속했기 때문이다.

## killed 플래그

`bool volatile killed`는 스레드에게 종료를 요청할 때 1로 설정된다. 시간이 오래 걸리는 작업(정렬, 인덱스 스캔 등)을 수행하는 코드는 주기적으로 이 플래그를 확인하고, 1이면 즉시 정리하고 빠져나와야 한다. KILL 명령이 동작하는 원리다.

참고: mysql-table-structure.md, net.md
