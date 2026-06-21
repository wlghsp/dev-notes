# MySQL 소스 모듈별 역할

MySQL 소스를 처음 열면 디렉토리가 많아서 어디서 봐야 할지 막막하다. 각 디렉토리가 왜 존재하는지 알면 탐색이 빠르다.

## 디렉토리별 존재 이유

**sql/**
MySQL의 두뇌 전체. 커넥션 처리, 인증, 파싱, 최적화, 실행, 복제, 프로토콜까지 SQL 레이어의 거의 모든 것이 여기 있다. 소스를 읽을 때 대부분의 시간을 여기서 보내게 된다.

**storage/** (또는 버전에 따라 sql/ 안에 위치)
스토리지 엔진 구현체들. `handler` 추상 클래스를 상속한 구체 엔진들이 여기 있다. InnoDB, MyISAM, MEMORY 등 엔진마다 서브디렉토리가 있다. SQL 레이어와 완전히 분리돼 있어서, 엔진 코드는 sql/을 거의 몰라도 된다.

**mysys/**
Core API. OS 추상화 레이어다. 파일 I/O, 메모리 할당, 뮤텍스, 스레드, 자료구조를 여기서 제공한다. MySQL 개발자들은 libc를 직접 쓰지 않고 이 API를 쓰도록 권장된다. 덕분에 Linux/Windows/macOS 이식이 가능하다. 함수명이 `my_`로 시작한다.

**strings/**
문자열 처리와 charset 관련 코드. mysys/와 함께 Core API를 구성한다.

**vio/**
Low-Level Network I/O 추상화. TCP/IP, Unix 소켓, SSL을 동일한 인터페이스로 감싼다. 네트워크 코드를 플랫폼마다 따로 짜지 않아도 되는 이유다. 함수명이 `vio_`로 시작한다.

## 처음 읽을 때 시작점

쿼리가 처리되는 전체 흐름을 따라가고 싶으면 `sql/mysqld.cc`의 `main()`에서 시작한다. `handle_connections_sockets()` → `handle_one_connection()` → `dispatch_command()` 순서로 내려가면 커넥션 수립부터 쿼리 실행까지 한 줄기로 읽힌다.

특정 모듈의 세부 파일 위치는 책 Chapter 1 p.9~18을 참고한다.

참고: mysql-architecture.md
