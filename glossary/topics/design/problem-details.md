# Problem Details (RFC 9457)

HTTP API가 에러를 응답할 때 쓰는 JSON 형식의 표준. 정식 이름은 "Problem Details for HTTP APIs"이고, 이전 표준인 RFC 7807을 개정해서 대체했다.

---

## 왜 표준이 필요한가

표준이 없으면 서비스마다 에러 응답의 모양이 다르다. `{"error": "..."}`, `{"message": "...", "errCode": 1}`, `{"result": "FAIL"}`처럼 제각각이라 클라이언트가 서비스마다 파싱 코드를 따로 짜야 한다. 성공 응답뿐 아니라 실패 응답의 모양도 API 계약으로 정해 두자는 것이 Problem Details의 취지다.

## 응답 모양

```
HTTP/1.1 409 Conflict
Content-Type: application/problem+json

{
  "type": "about:blank",
  "title": "Sold Out",
  "status": 409,
  "detail": "쿠폰이 매진되었습니다",
  "instance": "/api/coupons/1/issue",
  "code": "SOLD_OUT"
}
```

`Content-Type: application/problem+json`은 이 응답이 Problem Details 형식이라는 표시다.

## 필드

- `type`: 문제 종류를 식별하는 URI. 보통 그 에러를 설명하는 문서 주소를 쓴다. 따로 정의할 게 없으면 `about:blank`로 두며, 이는 HTTP 상태 코드 자체가 의미의 전부라는 뜻이다
- `title`: 문제 유형의 짧은 요약. 같은 `type`이면 항상 같은 값이다
- `status`: HTTP 상태 코드. 헤더와 같은 값을 본문에도 넣는다
- `detail`: 이번 발생 건에 대한 사람이 읽는 설명
- `instance`: 이번 발생 건을 특정하는 URI. 보통 문제가 난 요청 경로를 쓴다

위 5개 외에는 자유롭게 필드를 추가할 수 있다. 예시의 `code`가 확장 필드다.

## 클라이언트는 무엇으로 분기하는가

`detail`은 사람이 읽는 문구라 바뀔 수 있으므로 분기 기준으로 쓰면 안 된다. 같은 409라도 "매진", "이미 발급받음", "중복 요청"처럼 원인이 여럿일 수 있는데, 이를 기계가 구분하는 값이 `type`이나 확장 필드 `code`다. `type`을 `about:blank`로 두었다면 `code`가 그 역할을 맡는다.

## 상태 코드 선택

예시가 409 Conflict인 이유는 요청 형식은 올바르지만 현재 리소스 상태(재고 0)와 충돌했기 때문이다. 요청 자체가 잘못됐으면 400을 쓴다.

## Spring에서

Spring 6 / Boot 3부터 `ProblemDetail` 클래스를 기본 지원한다.

```java
@ExceptionHandler(SoldOutException.class)
ProblemDetail handle(SoldOutException e) {
    ProblemDetail pd = ProblemDetail.forStatusAndDetail(HttpStatus.CONFLICT, "쿠폰이 매진되었습니다");
    pd.setTitle("Sold Out");
    pd.setProperty("code", "SOLD_OUT");
    return pd;
}
```

`spring.mvc.problemdetails.enabled=true`를 켜면 Spring MVC가 던지는 기본 예외(400, 405 등)도 이 형식으로 응답한다.

## Recap

Problem Details(RFC 9457)는 HTTP API의 에러 응답을 `application/problem+json`이라는 공통 JSON 형식으로 통일하는 표준이다. `type`, `title`, `status`, `detail`, `instance`가 기본 필드이고, 서비스가 필요한 필드를 `code`처럼 덧붙일 수 있다. 사람이 읽는 `detail`은 바뀔 수 있으므로 클라이언트는 `type`이나 `code`로 분기해야 한다. Spring 6 이상에서는 `ProblemDetail`로 바로 만들 수 있어서, 실패의 모양까지 API 계약으로 관리할 수 있다.
