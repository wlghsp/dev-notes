# protobuf-thrift-avro (Protocol Buffers, Thrift, Avro)

참고: encoding.md, schema-evolution.md, backward-forward-compatibility.md

---

셋 다 스키마 기반 바이너리 인코딩 라이브러리다. Protocol Buffers(Google)와 Thrift(Facebook)는 같은 원리를 공유하고, Avro는 이들과 스키마 진화 방식이 다르다.

## Thrift와 Protocol Buffers — 필드 태그 방식

둘 다 인코딩 전에 스키마를 정의해야 한다. 스키마에서 각 필드에 태그 번호(1, 2, 3...)를 붙이고, 인코딩된 데이터에는 필드 이름 대신 이 태그 번호만 들어간다. 필드 이름을 매번 문자열로 반복해서 넣는 것보다 훨씬 컴팩트하다.

예를 들어 아래와 같은 Protocol Buffers 스키마가 있다면:

```
message Person {
    required string user_name      = 1;
    optional int64 favorite_number = 2;
    repeated string interests      = 3;
}
```

인코딩된 바이트에는 `user_name`이라는 문자열 대신 태그 `1`만 들어간다. 각 필드는 타입 애너테이션(문자열/정수/리스트 등)과 필요시 길이 정보를 갖는다.

> 📷 Figure 4-2 (책 p.118) — Thrift BinaryProtocol로 인코딩한 예시 레코드
> 📷 Figure 4-3 (책 p.119) — Thrift CompactProtocol로 인코딩한 예시 레코드 (가변 길이 정수 사용)
> 📷 Figure 4-4 (책 p.119) — Protocol Buffers로 인코딩한 예시 레코드

### 필드 태그와 스키마 진화

- 필드 이름은 바꿔도 된다. 인코딩된 데이터가 이름을 참조하지 않기 때문이다.
- 필드 태그는 절대 바꾸면 안 된다. 이미 인코딩된 기존 데이터가 무효화된다.
- 새 필드를 추가할 수 있다. 단, 새 태그 번호를 부여해야 하고, backward compatibility를 유지하려면 optional이거나 기본값이 있어야 한다. required로 추가하면 구버전 코드가 쓴 데이터를 신버전이 읽을 때 그 필드가 없어서 체크가 실패한다.
- 옛날 코드가 모르는 태그 번호를 만나면 타입 애너테이션을 보고 그만큼 바이트를 건너뛰면 된다. 이게 forward compatibility의 핵심이다.
- 필드를 제거할 때는 반대로, optional 필드만 제거 가능하고 그 태그 번호는 다시 쓰면 안 된다.

### 데이터 타입 변경

타입을 바꾸는 것도 가능하지만 정밀도 손실 위험이 있다. 32비트 정수를 64비트로 바꾸면 신버전이 구버전 데이터를 읽을 때는 문제없지만(빈 비트를 0으로 채움), 구버전이 신버전 데이터를 읽으면 64비트 값이 32비트에 잘려 들어갈 수 있다.

Protocol Buffers는 list 타입이 따로 없고 `repeated` 마커로 표현한다. 그래서 단일 값 필드를 리스트 필드로 바꾸는 게 자유롭다(같은 태그가 여러 번 나타나는 것뿐이므로). Thrift는 전용 list 타입이 있어서 이런 진화는 안 되지만, 중첩 리스트를 지원하는 대신 얻는다.

## Avro — 태그 번호가 없는 방식

Avro는 스키마에 태그 번호가 없다. 인코딩된 데이터는 필드 타입/이름 정보 없이 값들만 순서대로 이어 붙인 것이다. 그래서 Protocol Buffers/Thrift보다도 더 컴팩트하다.

> 📷 Figure 4-5 (책 p.122) — Avro로 인코딩한 예시 레코드, 필드/타입 마커 없이 값만 나열됨

디코딩하려면 스키마에 정의된 필드 순서를 그대로 따라가면서 각 필드의 타입을 스키마에서 읽어야 한다. 그래서 "쓸 때 쓴 스키마"와 "읽을 때 쓰는 스키마"가 정확히 일치해야만 올바르게 디코딩된다.

### writer's schema와 reader's schema

Avro의 핵심 아이디어는 이 둘이 완전히 같을 필요 없이 호환되기만 하면 된다는 것이다. 디코딩 시점에 Avro 라이브러리가 두 스키마를 나란히 놓고 차이를 해석한다.

- 필드 순서가 달라도 이름으로 매칭하기 때문에 문제없다.
- writer 스키마에는 있지만 reader 스키마에는 없는 필드는 무시한다.
- reader 스키마가 기대하는 필드가 writer 스키마에 없으면 reader 스키마에 정의된 기본값으로 채운다.

> 📷 Figure 4-6 (책 p.124) — Avro reader가 writer 스키마와 reader 스키마의 차이를 해석하는 과정

### 스키마 진화 규칙

기본값이 있는 필드만 추가/제거할 수 있다. 기본값 없는 필드를 추가하면 신버전 writer가 쓴 데이터를 구버전 reader가 못 읽어서 backward compatibility가 깨진다. 기본값 없는 필드를 제거하면 반대로 forward compatibility가 깨진다.

Avro에는 Protocol Buffers/Thrift 같은 optional/required 마커가 없다. 대신 union 타입과 기본값으로 처리한다. null을 허용하려면 `union { null, long }`처럼 명시적으로 선언해야 한다.

### writer's schema는 어디서 아는가

인코딩된 데이터마다 전체 스키마를 포함시키면 스키마가 데이터보다 커질 수 있어서 비효율적이다. 대신 상황별로 다르게 해결한다.

- 레코드가 많은 대용량 파일 하나: 파일 맨 앞에 writer 스키마를 한 번만 포함 (Avro object container file)
- 레코드별로 다른 시점에 쓰이는 DB: 레코드마다 스키마 버전 번호만 넣고, 버전별 스키마는 별도 DB에 보관
- 양방향 네트워크 연결: 연결 설정 시점에 스키마 버전을 협상하고 그 연결 동안 유지

### 동적으로 생성되는 스키마에 유리한 이유

Avro 스키마에는 태그 번호가 없기 때문에, 관계형 DB 스키마에서 Avro 스키마를 자동 생성하기 쉽다. 컬럼이 추가/삭제돼도 필드 이름으로 매칭하면 되기 때문에 스키마 변환기가 신경 쓸 게 없다. Protocol Buffers/Thrift였다면 컬럼명-태그 매핑을 사람이 관리해야 했을 것이다.

## 코드 생성

Thrift/Protocol Buffers는 스키마로부터 코드를 생성해서 정적 타입 언어(Java, C++ 등)에서 컴파일 타임 타입 체크와 자동완성을 지원한다. Avro도 코드 생성을 지원하지만 필수는 아니다. writer 스키마가 내장된 object container file은 코드 생성 없이도 JSON 파일처럼 바로 열어볼 수 있다. 이 점은 동적 타입 언어나 Apache Pig 같은 동적 데이터 처리 도구에서 특히 유용하다.

## 스키마 방식의 장점

XML Schema나 JSON Schema보다 훨씬 단순하면서도,

1. 필드 이름을 생략할 수 있어 "바이너리 JSON" 계열보다 컴팩트하다.
2. 스키마 자체가 최신 상태로 유지되는 문서 역할을 한다(디코딩에 필수이기 때문에 실제 코드와 어긋날 수 없음).
3. 스키마 데이터베이스를 유지하면 배포 전에 forward/backward compatibility를 미리 검증할 수 있다.
4. 정적 타입 언어에서 코드 생성으로 컴파일 타임 체크가 가능하다.
