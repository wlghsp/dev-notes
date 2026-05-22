# Socket

네트워크 통신의 양 끝단을 나타내는 추상화. 프로세스가 네트워크를 통해 데이터를 주고받기 위한 출입구다.

소켓은 개념이고, TCP/UDP는 그 소켓이 어떤 방식으로 통신할지 결정하는 프로토콜이다.

## 소켓의 정체

소켓은 파일이다. 유닉스 계열 OS에서는 "모든 것이 파일"이라는 철학에 따라 네트워크 연결도 파일 디스크립터로 다룬다.

```
int fd = socket(AF_INET, SOCK_STREAM, 0);  // 소켓 생성 = 파일 디스크립터 반환
```

프로세스는 이 파일 디스크립터에 read/write 하는 것으로 네트워크 데이터를 주고받는다.
OS 커널이 실제 네트워크 전송을 처리하고, 프로세스는 파일 다루듯이 쓰면 된다.

## TCP 소켓 연결 흐름

서버와 클라이언트가 소켓 수준에서 하는 일이 다르다.

서버 쪽:
1. `socket()` — 소켓 생성
2. `bind()` — 포트에 붙이기
3. `listen()` — 연결 대기
4. `accept()` — 클라이언트 연결 수락, 새 소켓 반환
5. `read()/write()` — 데이터 송수신

클라이언트 쪽:
1. `socket()` — 소켓 생성
2. `connect()` — 서버에 연결 (3-way handshake 발생)
3. `read()/write()` — 데이터 송수신

`accept()`가 새 소켓을 반환한다는 점이 핵심이다. 서버는 연결마다 별도의 소켓을 가지고, 최초 `listen()`하던 소켓은 계속 새 연결을 받는다.

## Network Socket vs Unix Domain Socket

소켓에는 두 종류가 있다.

**Network Socket (TCP/IP Socket)**
- IP + 포트 번호로 식별
- 네트워크 스택 전체를 거친다
- 다른 서버와 통신 가능

**Unix Domain Socket**
- 파일 시스템 경로로 식별 (예: `/var/lib/mysql/mysql.sock`)
- 같은 서버 내 프로세스 간 통신에만 사용
- 네트워크 스택을 거치지 않아서 더 빠르다
- MySQL, PostgreSQL, Redis 등이 로컬 접속에 활용

MySQL에서 `-h localhost`로 접속하면 Unix Domain Socket을 쓰고, `-h 127.0.0.1`이면 Network Socket(TCP)을 쓰는 이유가 여기 있다. 참고: mysql-socket.md

## 포트와 소켓의 관계

포트만으로 연결을 식별하지 않는다. 소켓은 4개의 값으로 식별된다.

```
(클라이언트 IP, 클라이언트 포트, 서버 IP, 서버 포트)
```

서버 포트가 3306으로 같아도, 클라이언트 IP와 포트가 다르면 다른 연결이다.
그래서 MySQL 서버 포트 3306 하나로 수백 개의 클라이언트 연결을 동시에 처리할 수 있다.

참고: tcp.md, mysql-socket.md
