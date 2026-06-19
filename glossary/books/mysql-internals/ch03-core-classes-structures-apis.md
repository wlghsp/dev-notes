# Chapter 3 — Core Classes, Structures, Variables, and APIs

키워드 파일: thd.md, net.md, mysql-table-structure.md, field-class.md

---

## 이 챕터의 목적

MySQL 소스는 수십만 줄이다. 하지만 핵심 클래스 몇 개의 역할을 알면 대부분의 코드가 읽히기 시작한다. THD, NET, TABLE, Field — 이 네 개가 MySQL 내부 코드의 공용어다.

## THD: 모든 문맥의 출발점

THD는 스레드 디스크립터다. 클라이언트 커넥션 하나를 처리하는 스레드의 상태 전체를 담는다. 현재 쿼리 텍스트, parse tree, 메모리 풀, 네트워크 커넥션, 트랜잭션 상태, 권한 정보, 경고 목록까지 전부 여기 있다.

거의 모든 함수가 `THD*`를 첫 인자로 받는 이유가 이것이다. "지금 이 코드가 어떤 문맥에서 실행되는가"를 알려면 THD를 봐야 한다. 클라이언트 커넥션뿐 아니라 복제 슬레이브 스레드, delayed insert 스레드에도 THD가 생성된다.

4.1에서 THD 일부 멤버들이 `Statement` 클래스로 이동했고, THD가 Statement의 서브클래스가 됐다.

## NET: 네트워크 프로토콜의 핵심

NET는 네트워크 커넥션 디스크립터다. THD 안에 `net` 멤버로 들어가 있고, 모든 네트워크 통신 함수가 NET를 사용한다.

가장 중요한 멤버는 `Vio* vio`다. VIO가 TCP/IP, Unix 소켓, SSL, Windows named pipe를 동일한 인터페이스로 추상화하기 때문에, NET 위의 코드는 실제 전송 방식을 몰라도 된다.

NET는 `include/mysql_com.h`에 정의되어 있고 클라이언트 라이브러리도 같은 파일을 쓴다. 서버와 클라이언트가 같은 구조체를 공유하는 것이다.

## TABLE: 열린 테이블의 모든 것

TABLE은 테이블 디스크립터다. 테이블이 쿼리에 사용될 때 생성되고 테이블 캐시에 보관된다.

안에는 `handler *file`(이 테이블의 스토리지 엔진 객체), `Field **field`(컬럼 디스크립터 배열), `byte *record[2]`(레코드 버퍼 쌍), 인덱스 맵들이 들어 있다.

`record[0]`이 현재 레코드, `record[1]`이 UPDATE 시 이전 레코드다. Field의 `ptr` 멤버는 이 `record[0]` 안의 특정 위치를 직접 가리킨다.

버전 5.1에서 TABLE이 리팩토링되면서 공유 가능한 메타데이터가 `TABLE_SHARE`로 분리됐다. 여러 스레드가 같은 테이블을 열어도 정적 메타데이터는 하나만 존재한다.

## Field: 컬럼 접근의 통일된 인터페이스

Field는 컬럼 하나의 디스크립터다. 추상 기반 클래스이고 타입마다 서브클래스(`Field_long`, `Field_varchar` 등)가 있다.

`ptr`이 `record[0]` 안에서 이 컬럼 값의 위치를 가리키고, `null_ptr`과 `null_bit`이 NULL 여부를 관리한다. `store()` 메서드로 값을 쓰고, `val_real()` / `val_str()`로 읽는다.

옵티마이저는 Field 타입을 몰라도 된다. 인터페이스만 호출하면 각 서브클래스가 알아서 처리한다. `result_type()`이 range optimizer가 타입별 비교 의미론을 판단하는 데 쓰인다.

## 이 챕터의 핵심

THD → NET → TABLE → Field 계층은 "커넥션이 네트워크를 통해 테이블을 열고 컬럼 값을 읽는다"는 동작을 그대로 구조체로 표현한 것이다. 이 네 개를 알면 MySQL 소스 대부분이 낯설지 않아진다.
