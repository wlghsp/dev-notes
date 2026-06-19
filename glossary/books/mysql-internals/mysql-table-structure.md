# TABLE (Table Descriptor)

TABLE은 데이터베이스 테이블 디스크립터다. 테이블이 서버에서 사용되려면 반드시 열린 상태여야 하고, 그때 TABLE 객체가 생성된다. 생성된 TABLE은 테이블 캐시에 놓여 같은 테이블을 참조하는 다른 요청이 재사용할 수 있다.

정의는 `sql/table.h`에 `struct st_table`로 되어 있고, `sql/handler.h`에서 `typedef`로 TABLE로 불린다. 파서, 옵티마이저, 접근 제어, 쿼리 캐시 코드에서 빈번하게 참조된다.

## TABLE과 TABLE_SHARE의 분리

버전 5.1에서 TABLE이 리팩토링됐다. 같은 물리적 테이블의 여러 인스턴스가 공유할 수 있는 부분이 `TABLE_SHARE`로 분리됐다. TABLE_SHARE는 테이블 정의 캐시(table definition cache)에 별도로 캐시된다.

이 분리의 의미는 여러 스레드가 같은 테이블을 동시에 열더라도, 컬럼 정의나 인덱스 정보 같은 정적인 메타데이터는 TABLE_SHARE 하나를 공유하고, 각 스레드의 런타임 상태(레코드 버퍼, 락 상태 등)는 별도 TABLE 객체에 유지된다는 것이다.

## 핵심 멤버들

`handler *file`은 이 테이블을 담당하는 스토리지 엔진 객체를 가리킨다. 모든 저수준 데이터 읽기/쓰기는 이 포인터를 통해 이루어진다.

`Field **field`는 이 테이블의 모든 컬럼 디스크립터 배열이다. 배열 길이는 `fields`에 저장된다.

`byte *record[2]`는 레코드 버퍼 쌍이다. `record[0]`은 현재 읽힌 레코드, `record[1]`은 UPDATE 처리 시 이전 레코드를 저장한다. 옵티마이저가 레코드 조작에 쓰는 임시 버퍼다.

`reclength`는 옵티마이저 레벨에서 메모리에 올라온 레코드의 바이트 길이다. 스토리지 엔진이 디스크에 저장할 때의 길이와 다를 수 있다.

## 인덱스 관련 멤버들

`keys_in_use`는 현재 사용 가능한 인덱스 맵이다. `ALTER TABLE ... DISABLE KEYS`로 비활성화된 인덱스는 여기서 빠진다. `quick_keys`는 범위 최적화에 쓸 수 있는 인덱스 맵이다. `used_keys`는 `keys_in_use`에서 `FORCE KEY` / `IGNORE KEY` 지시를 반영한 최종 인덱스 맵이다.

`table_cache_key`는 테이블 캐시에서 이 TABLE을 찾는 해시 키다. `database_name\0table_name\0` 형태로 구성된다.

`version`은 캐시된 TABLE이 최신인지 확인하는 데 쓴다. 다른 스레드가 `FLUSH TABLES`를 수행하면 전역 `refresh_version`이 올라가고, 이 값과 비교해 TABLE을 무효화한다.

참고: field-class.md, thd.md
