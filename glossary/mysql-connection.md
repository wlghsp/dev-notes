# MySQL 접속 흐름

클라이언트가 `mysql -u root -p`를 치는 순간부터 쿼리를 날릴 수 있는 상태가 되기까지의 흐름.

## 1. 네트워크 연결 수립

TCP면 3-way handshake로 연결을 맺는다. Socket이면 소켓 파일을 통해 연결한다.
연결이 맺어지면 MySQL 서버는 클라이언트 전용 **스레드**를 하나 생성한다.
이 스레드가 이후 모든 통신을 담당한다.

## 2. 서버 → 클라이언트: Initial Handshake Packet

연결이 수립되면 서버가 먼저 말을 건다. 클라이언트가 아니다.

서버가 보내는 내용은 다음과 같다.

- 서버 버전 (예: 8.0.32)
- connection_id — 이 연결의 고유 ID (`SHOW PROCESSLIST`에서 보이는 그것)
- 인증 방식 (auth plugin 이름, 예: `caching_sha2_password`)
- 랜덤 challenge 값 (salt) — 비밀번호 해싱에 쓰인다

## 3. 클라이언트 → 서버: Login Request Packet

클라이언트는 서버가 보낸 salt를 이용해 비밀번호를 해싱해서 보낸다.
평문 비밀번호는 네트워크로 전송되지 않는다.

패킷에 담기는 내용은 다음과 같다.

- 사용자명
- 해싱된 비밀번호
- 접속할 데이터베이스명 (옵션)
- 클라이언트 capability flags — 클라이언트가 지원하는 기능 목록

## 4. 서버 인증 처리

서버는 받은 해싱값을 `mysql.user` 테이블의 정보와 비교한다.

인증 방식은 MySQL 버전마다 다르다.

- MySQL 5.7 이하: `mysql_native_password` — SHA1 기반
- MySQL 8.0+: `caching_sha2_password` — SHA256 기반, 첫 인증 후 캐싱

인증 외에 접속 허용 여부도 여기서 결정된다.
`user@host` 조합으로 접근 제어를 한다. `root@localhost`와 `root@%`는 다른 계정이다.

인증 실패 시 `ERROR 1045 (28000): Access denied` 를 반환하고 연결을 끊는다.

## 5. 세션 수립

인증이 통과되면 서버는 OK 패킷을 보내고 세션이 시작된다.

세션에는 다음 상태가 붙는다.

- 현재 선택된 데이터베이스
- 트랜잭션 상태 (autocommit 여부 등)
- 세션 변수 (time_zone, character_set 등)
- connection_id

이 상태는 연결이 끊어질 때까지 유지된다.

## 6. 쿼리 실행 준비 완료

이후 클라이언트가 쿼리를 보내면 서버는 파싱 → 최적화 → 실행 순서로 처리한다.
이 흐름은 별도로 다룬다.

## 운영에서 유용한 지점

접속 흐름을 알면 장애 상황에서 어디가 문제인지 좁힐 수 있다.

네트워크 연결 자체가 안 되는 경우는 방화벽, bind-address, 포트 문제다.

```
ERROR 2003 (HY000): Can't connect to MySQL server on '...' (111)
```

소켓 파일을 못 찾는 경우는 경로 문제다. 참고: mysql-socket.md

```
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '...'
```

인증은 됐지만 접근이 거부되는 경우는 user@host 매핑 문제다.

```
ERROR 1045 (28000): Access denied for user 'root'@'localhost'
```

현재 연결 상태는 아래로 확인한다.

```sql
SHOW PROCESSLIST;         -- 전체 연결 목록
SELECT CONNECTION_ID();   -- 현재 connection_id
SHOW STATUS LIKE 'Threads_connected';  -- 현재 연결 수
```
