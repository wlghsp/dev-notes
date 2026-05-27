# MySQL 접속 방식: TCP vs Unix Socket

MySQL 클라이언트가 서버에 접속하는 방법은 두 가지다.

## TCP 접속

```
mysql -u root -p -h 127.0.0.1
```

`-h`에 IP 주소를 명시하면 네트워크 TCP 연결로 접속한다. 포트 3306을 사용한다.
원격 서버 접속이나 같은 서버라도 TCP를 강제하고 싶을 때 쓴다.

## Unix 도메인 소켓 접속

```
mysql -u root -p --socket=/var/lib/mysql/mysql.sock
```

`-h localhost` 또는 호스트를 생략하면 MySQL은 TCP 대신 Unix 도메인 소켓으로 접속을 시도한다.
소켓은 파일 시스템에 존재하는 특수 파일(`.sock`)을 통해 같은 서버 내에서 프로세스끼리 통신하는 방식이다.
네트워크 스택을 거치지 않아서 TCP보다 빠르다.

소켓 파일의 기본 경로는 `/tmp/mysql.sock` 또는 `/var/lib/mysql/mysql.sock`인데,
`my.cnf`에서 `socket=` 설정이 다른 경로로 바뀌어 있으면 클라이언트가 못 찾는다.
그때 `--socket`으로 직접 경로를 명시해야 한다.

## --socket을 써야 하는 상황

소켓 파일 경로가 비표준일 때가 가장 흔한 경우다.

같은 서버에 MySQL 인스턴스를 여러 개 띄운 경우에도 `--socket`이 필요하다.
인스턴스마다 소켓 파일이 달라서 어느 인스턴스에 붙을지 명시해야 한다.

## 에러로 판단하는 법

```
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock'
```

이 에러는 소켓 파일 경로가 달라서 생긴다. MySQL이 실제로 쓰는 소켓 경로는 아래로 확인한다.

```sql
-- TCP로 붙어서 확인
mysql -u root -p -h 127.0.0.1 -e "SHOW VARIABLES LIKE 'socket';"
```

## 핵심

`-h localhost` → 소켓 접속 시도  
`-h 127.0.0.1` → TCP 접속  

localhost와 127.0.0.1은 같은 의미처럼 보이지만, MySQL에서는 접속 방식 자체가 달라진다.
