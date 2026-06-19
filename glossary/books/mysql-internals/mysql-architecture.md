# MySQL Architecture

MySQL 아키텍처는 공식 스펙이 있어서 설계된 게 아니다. Pachev이 책에서 직접 밝히듯, 처음에는 그냥 특정 문제를 잘 해결하는 코드 조각들이었다. 그게 쌓이다 보니 데이터베이스 서버를 조립할 수 있는 품질 좋은 부품들이 됐다.

## 커넥션에서 실행까지

클라이언트가 접속하면 Connection Manager가 받아서 Thread Manager에 넘긴다. Thread Manager는 스레드를 새로 만들거나 캐시에서 꺼내 커넥션에 붙인다. 이 스레드가 Connection Thread가 되어 이후 모든 요청을 처리한다.

첫 요청에서 User Authentication이 인증을 마치면, 이후 요청들은 Command Dispatcher로 들어온다. 여기서 두 갈래로 나뉜다. 파서를 거쳐야 하는 것은 query, 파서 없이 바로 처리되는 것은 command다. SELECT는 물론이고 INSERT, DELETE도 query다. 데이터베이스를 바꾸거나 커넥션을 끊는 건 command다.

query는 먼저 Query Cache를 거친다. 캐시 hit이면 파서까지 가지 않고 바로 반환된다. miss면 Parser가 SQL을 parse tree로 변환하고, Access Control이 권한을 확인한다. 그 다음 Table Manager가 테이블을 열고 락을 잡는다. SELECT는 Optimizer로, DML은 각 Table Modification Module로 간다. 최종적으로 Storage Engine Interface(handler)를 통해 실제 엔진을 호출한다.

> 📷 Figure 1-1 (책 p.8) — MySQL 코어 모듈 전체 흐름도. Connection Manager부터 스토리지 엔진까지 내려가는 구조 다이어그램

## 두 개의 분리된 계층

쿼리 처리 흐름에서 핵심은 handler 추상 클래스다. Optimizer와 Table Modification Module은 어떤 스토리지 엔진이 데이터를 저장하는지 모른다. handler 인터페이스만 호출할 뿐이다. 인터페이스 아래에서 InnoDB가 응답하든 MyISAM이 응답하든 상관없다.

이 분리 덕분에 SQL 처리 레이어와 스토리지 레이어가 독립적으로 발전할 수 있다. 모듈별 소스 위치는 mysql-source-modules.md를 참고한다.

## query vs command

책이 명확히 구분하는 개념이다. query는 파서를 거쳐야 하는 모든 SQL문이다. SELECT뿐 아니라 DELETE, INSERT도 query다. command는 파서 없이 Command Dispatcher가 직접 처리하는 요청이다. 예컨대 active database를 바꾸거나, 커넥션을 끊거나, 복제 업데이트를 스트리밍하는 것 등이 command다.

## 스토리지 엔진 인터페이스가 만들어진 이유

Berkeley DB를 통합하려는 시도(version 3.23)가 실패로 끝났지만, 그 과정에서 어떤 스토리지 엔진도 플러그인처럼 붙일 수 있는 hooks가 MySQL 소스에 남았다. 그 유산이 handler 추상 클래스다. InnoDB가 이 인터페이스를 통해 훨씬 매끄럽게 통합됐고, MySQL 4.0에서 안정화됐다.

참고: mysql-source-modules.md
