# Chapter 1 — MySQL History and Architecture

키워드 파일: mysql-architecture.md, mysql-source-modules.md

---

## MySQL이 이렇게 만들어진 이유

1979년 Monty Widenius가 TcX라는 작은 회사에서 BASIC으로 만든 리포팅 툴이 시작이다. 4MHz CPU, 16KB RAM짜리 컴퓨터에서 돌아가는 물건이었다. 제약이 심한 환경이 Monty를 효율적인 코드 작성에 단련시켰고, 그게 MySQL의 성능 DNA가 됐다.

1990년대 TcX 고객들이 SQL 인터페이스를 원하기 시작했다. 상용 DB는 너무 느렸고, mSQL 코드를 통합하는 시도도 실패했다. 그래서 Monty가 직접 SQL 레이어를 만들었다. 1996년 5월 version 1.0이 나오고, 같은 해 10월 version 3.11.1이 공개됐다.

## 스토리지 엔진 플러그인 구조가 생긴 경위

처음에 MySQL은 트랜잭션이 없었다. 1999~2000년에 Berkeley DB를 통합해서 트랜잭션을 추가하려 했는데, Berkeley DB 인터페이스의 quirk들을 끝내 해결하지 못해 실패했다. 그런데 그 시도 덕분에 어떤 스토리지 엔진이든 플러그인처럼 붙일 수 있는 hooks가 코드에 남았다.

그 직후 Heikki Tuuri가 자신의 InnoDB를 MySQL에 통합하겠다고 제안했다. 이미 닦아놓은 handler 인터페이스 덕분에 InnoDB는 훨씬 매끄럽게 통합됐다. MySQL 4.0에서 안정화되고 2003년 3월 production-stable 선언됐다.

즉, 플러그인 구조는 처음부터 설계한 게 아니라 실패한 Berkeley DB 통합의 부산물이다.

## 모듈 구조와 쿼리 처리 흐름

책은 MySQL을 약 20개 모듈로 분류한다. 실제 MySQL 개발자들은 모듈 단위로 생각하지 않고 파일/함수/클래스 단위로 생각하지만, 외부에서 전체 구조를 파악할 때 유용한 분류다.

클라이언트 요청이 들어올 때 흐름:

1. Connection Manager가 접속을 수락 (`handle_connections_sockets()`)
2. Thread Manager가 스레드 할당 (신규 생성 또는 캐시에서 꺼냄)
3. Connection Thread가 요청 처리 루프 시작 (`handle_one_connection()`)
4. User Authentication Module이 인증 처리
5. Command Dispatcher가 요청 종류에 따라 라우팅 (`dispatch_command()`)
6. Query Cache Module 확인 → 캐시 hit이면 즉시 반환
7. Parser가 SQL 파싱 → parse tree 생성
8. Access Control Module이 권한 확인
9. Table Manager가 테이블 열고 락 획득
10. SELECT면 Optimizer로, DML이면 각 Table Modification Module로
11. Storage Engine Interface(handler)를 통해 실제 엔진 호출
12. 결과를 Client/Server Protocol API로 클라이언트에 전송

> 📷 Figure 1-1 (책 p.8) — MySQL 코어 모듈 전체 흐름도. Connection Manager부터 스토리지 엔진까지 내려가는 구조 다이어그램

## query와 command의 구분

MySQL 내부 용어로 query는 파서를 거쳐야 하는 모든 SQL문이다. SELECT뿐 아니라 DELETE, INSERT도 query다. command는 파서 없이 Command Dispatcher가 직접 처리하는 요청으로, 데이터베이스 변경, 커넥션 종료, 복제 스트리밍 등이 해당한다.

## Core API의 역할

`mysys/`와 `strings/` 디렉토리에 있는 Core API는 MySQL의 OS 추상화 레이어다. 파일 I/O, 메모리, 문자열, 자료구조를 모두 여기서 제공한다. MySQL 개발자들은 `libc`를 직접 쓰지 말고 Core API를 쓰도록 권장된다. 덕분에 Linux/Windows/macOS 이식이 가능하다.

## 이 챕터의 핵심

MySQL 아키텍처는 처음부터 설계된 게 아니다. 실용적인 문제들을 해결하면서 만들어진 코드가 쌓였고, 그 코드들이 품질이 좋아서 데이터베이스 서버로 조립됐다. 스토리지 엔진 플러그인 구조도 마찬가지다. 설계 의도가 아니라 실패의 부산물이 훗날 MySQL 최대의 강점이 됐다.
