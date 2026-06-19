# tablespace

InnoDB가 데이터를 디스크에 저장할 때 사용하는 논리적 저장 단위다. 실제로는 하나 이상의 물리적 파일(.ibd, ibdata)로 구성된다.

## 왜 필요한가

InnoDB는 데이터를 그냥 파일에 저장하지 않는다. Page 단위(기본 16KB)로 나눠서 관리하고, 그 Page들을 담는 컨테이너가 tablespace다. tablespace가 있어야 InnoDB가 어느 파일의 어느 오프셋에 어떤 Page가 있는지 추적할 수 있다.

## 종류

InnoDB에는 세 가지 주요 tablespace가 있다.

**시스템 테이블스페이스 (ibdata1)**

InnoDB 전체가 공유하는 공간이다. 초기 MySQL에서는 모든 테이블 데이터와 인덱스, Undo Log, 이중 쓰기 버퍼(doublewrite buffer)가 여기에 들어갔다. 한 파일에 모든 것이 몰리다 보니 파일이 커지면 줄이기가 어렵다는 문제가 있었다.

**File-Per-Table 테이블스페이스 (.ibd)**

`innodb_file_per_table=ON` 설정 시(MySQL 5.6부터 기본값) 테이블마다 별도 `.ibd` 파일이 생긴다. 테이블을 DROP하면 해당 파일이 삭제되어 공간이 바로 회수된다. 시스템 테이블스페이스의 비대화 문제를 해결한다.

**Undo 테이블스페이스**

Undo Log를 시스템 테이블스페이스에서 분리한 것이다. MySQL 8.0부터 기본적으로 별도 파일(undo_001, undo_002)로 분리된다. Undo Log가 많이 쌓이는 환경에서 시스템 테이블스페이스가 비대해지는 것을 막는다.

## Page와의 관계

tablespace 안은 Page로 채워진다. InnoDB는 데이터를 읽고 쓸 때 항상 Page 단위로 처리한다. Row 하나를 읽어도 그 Row가 속한 Page 전체를 Buffer Pool로 올린다.

```
tablespace (.ibd 파일)
  └── Segment (테이블, 인덱스 단위)
        └── Extent (1MB, 64개 Page 묶음)
              └── Page (16KB, 실제 데이터 단위)
                    └── Row
```

참고: wal.md, buffer-pool.md
