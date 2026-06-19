# Field (Field Descriptor)

Field는 테이블의 컬럼 하나를 표현하는 디스크립터다. 파서와 옵티마이저에서 핵심적인 역할을 한다. 쿼리를 처리하는 거의 모든 작업이 테이블 컬럼을 다루기 때문이다.

정의는 `sql/field.h`, 부분 구현은 `sql/field.cc`. 추상 기반 클래스(abstract base class)라 직접 인스턴스화되지 않는다. 모든 서브클래스 이름이 `Field_`로 시작한다.

## 추상 클래스인 이유

Field는 메서드가 많고 데이터 멤버는 적다. 타입마다 값을 저장하고 읽는 방식이 다르기 때문에, 공통 인터페이스를 Field가 정의하고 실제 구현은 서브클래스에 맡긴다. 옵티마이저는 컬럼이 INT인지 VARCHAR인지 몰라도 `field->val_real()`이나 `field->store()`만 호출하면 된다.

## 핵심 멤버들

`char *ptr`은 인메모리 레코드 복사본(즉 TABLE의 `record[0]`)에서 이 필드의 데이터가 있는 위치를 가리킨다. 스토리지 엔진이 레코드를 읽어 `record[0]`에 채우면, 각 Field는 자기 `ptr`로 자신의 값이 어디 있는지 바로 안다.

`uchar *null_ptr`은 레코드 prefix에 있는 NULL 플래그 바이트를 가리킨다. `null_bit`은 그 바이트에서 이 필드의 NULL 여부를 나타내는 비트 위치다. NULL 허용 컬럼만 이 두 멤버가 의미 있다.

`uint32 field_length`는 이 필드에 저장 가능한 최대 바이트 수다. `uint16 flags`는 `NOT NULL`, `AUTO_INCREMENT`, `ZEROFILL` 같은 속성을 비트마스크로 저장한다.

## 중요한 메서드들

`store()` 계열 메서드는 인메모리 레코드에 값을 쓴다. 문자열, double, longlong, TIME 타입별 오버로드가 있다.

`val_real()`, `val_str()` 계열은 인메모리 레코드에서 값을 읽어 반환한다. `result_type()`은 range optimizer가 범위 최적화 적용 가능 여부를 판단하는 데 쓴다. 예컨대 문자열 컬럼에 숫자 범위 조건을 걸면 사전순 정렬이 적용되어 결과가 달라질 수 있는데, 이를 감지하기 위해 필요하다.

`is_null()`, `set_null()`, `maybe_null()`은 NULL 상태를 다룬다.

`cmp()`는 필드 값과 문자열을 비교한다. `key_cmp()`는 인덱스 키 컨텍스트에서의 비교다.

참고: mysql-table-structure.md
