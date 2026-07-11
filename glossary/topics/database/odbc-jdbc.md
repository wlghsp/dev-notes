# ODBC / JDBC

애플리케이션이 어떤 DB 벤더를 쓰든 동일한 API로 접근할 수 있게 해주는 DB 연결 표준. 애플리케이션 코드가 DB마다 다른 통신 프로토콜을 직접 알 필요가 없도록 만드는 추상화 계층이다.

## 핵심 문제

DB마다 통신 프로토콜이 다르다. MySQL, PostgreSQL, Oracle, SQL Server는 각각 고유한 와이어 프로토콜을 쓴다. 이걸 애플리케이션이 직접 다루면 DB를 바꿀 때마다 코드를 다시 짜야 하고, 벤더에 종속된다.

ODBC/JDBC는 애플리케이션과 DB 사이에 표준 인터페이스를 두고, 실제 벤더별 프로토콜 변환은 드라이버가 담당하게 해서 이 문제를 없앤다.

## ODBC (Open Database Connectivity)

Microsoft가 만든 언어 중립적 DB 접근 표준. C 기반 API이며, OS 레벨의 드라이버(Windows는 .dll, Linux/macOS는 .so)를 통해 실제 DB와 통신한다.

- 어떤 프로그래밍 언어에서든(C, Python, R 등) OS에 등록된 ODBC 드라이버를 통해 DB에 접근 가능
- Excel, Access처럼 DB에 붙는 데스크톱 도구들이 흔히 사용
- OS별로 드라이버를 따로 설치/설정해야 한다 (Windows의 ODBC Data Source Administrator 등)

## JDBC (Java Database Connectivity)

Java용 DB 접근 API. ODBC의 개념을 Java 생태계에 맞게 만든 것으로, Sun(현 Oracle)이 정의했다.

- `Connection`, `Statement`, `ResultSet` 같은 표준 인터페이스로 DB에 접근
- 드라이버 종류에 따라 OS 네이티브 라이브러리 없이도 동작 가능 (Type 4 드라이버는 DB 와이어 프로토콜을 순수 Java로 직접 구현해서 JVM만 있으면 어디서든 동작)
- JVM 위에서 동작하므로 플랫폼 독립적

## 차이

| | ODBC | JDBC |
|---|---|---|
| 언어 | 언어 중립(C API) | Java 전용 |
| 동작 위치 | OS 레벨 드라이버 | JVM 레벨(플랫폼 독립적) |
| 만든 곳 | Microsoft | Sun(현 Oracle) |

## 정리

둘 다 "DB 벤더별 프로토콜을 애플리케이션이 몰라도 되게 하는 표준 인터페이스"라는 같은 역할을 하며, ODBC는 OS/C 생태계, JDBC는 Java 생태계에서 그 역할을 담당한다.
