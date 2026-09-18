> Spring Boot로 Todo API의 201/400/404 계약을 구현하면서, `@RequestBody`와 `@ResponseBody`가 당연하게 되는 일 뒤에 항상 같은 녀석이 있다는 걸 알게 됐다.
>
> ObjectMapper. Controller 코드에는 한 줄도 안 나오는데, 요청이 들어올 때도 응답이 나갈 때도 매번 이 녀석이 일하고 있었다.

---

## 계기 — "이 변환은 대체 누가 하는 거지?"

Todo API를 만들면서 이런 코드를 짰다.

```java
@PostMapping("/todos")
public ResponseEntity<TodoResponse> create(@Valid @RequestBody TodoCreateRequest request) {
    Todo todo = todoService.create(request.title(), request.completed());
    return ResponseEntity.status(HttpStatus.CREATED).body(TodoResponse.from(todo));
}
```

JSON을 보내면 `TodoCreateRequest` 객체가 파라미터에 채워져 있고, `TodoResponse` 객체를 리턴하면 JSON이 나간다. 코드만 보면 마법 같다. 근데 이 마법을 뜯어보니 입구와 출구에 매번 같은 컴포넌트가 서 있었다.

## 요청이 들어올 때 — JSON → 객체

`@RequestBody`가 붙은 파라미터를 스프링이 채워주는 과정을 따라가 보면:

1. `DispatcherServlet`이 요청을 받고, `HandlerAdapter`가 컨트롤러 메서드를 실행하기 직전
2. 파라미터에 `@RequestBody`가 있으면 `RequestResponseBodyMethodProcessor`(`HandlerMethodArgumentResolver` 구현체)가 개입
3. 이 프로세서가 내부적으로 `HttpMessageConverter`를 호출
4. JSON이면 `MappingJackson2HttpMessageConverter`가 선택되고, 그 안의 **Jackson `ObjectMapper`**가 JSON 문자열을 자바 객체로 역직렬화

여기서 변환이 실패하면(생성자가 없거나, 타입이 안 맞거나, 모르는 필드가 오거나) `HttpMessageNotReadableException`이 던져진다. 컨트롤러 메서드는 아예 실행되지도 않는다. 400을 만드는 흐름에 내 코드는 없었다 — ObjectMapper가 못 읽었다고 거부한 것뿐이다.

## 응답이 나갈 때 — 객체 → JSON

반대 방향도 똑같은 구조다.

1. 컨트롤러가 객체를 반환하면 `ReturnValueHandler`가 개입
2. `@ResponseBody`(또는 `@RestController`)가 있으면 뷰로 안 넘기고 본문에 직접 쓰기로 결정
3. `Accept` 헤더와 반환 타입을 보고 `HttpMessageConverter`를 다시 선택
4. JSON이면 또 `MappingJackson2HttpMessageConverter` → 그 안의 **같은 ObjectMapper**가 이번엔 직렬화

## 깨달은 것 — 한 컴포넌트가 양방향을 다 맡고 있다

요청 읽기(`read`)와 응답 쓰기(`write`)가 서로 다른 컴포넌트일 거라고 막연히 생각했는데, 사실은 `MappingJackson2HttpMessageConverter` 하나가 방향만 바꿔서 양쪽 다 처리하고 있었다. 그리고 그 안에서 실제 변환을 담당하는 게 Jackson의 `ObjectMapper`다.

```
요청 JSON  --[ObjectMapper.readValue]-->  자바 객체  --(컨트롤러 실행)-->
자바 객체  --[ObjectMapper.writeValue]--> 응답 JSON
```

컨트롤러 코드에는 `ObjectMapper`가 단 한 줄도 등장하지 않는다. `@RequestBody`, `@ResponseBody` 뒤에 숨어서 요청도, 응답도, 심지어 400 에러도 다 이 녀석의 손을 거친다.

## 그래서 뭐가 달라지나

- 역직렬화 실패로 400이 나는 원인(`MismatchedInputException`, `UnrecognizedPropertyException`, `JsonParseException` 등)을 볼 때 "내 검증 로직 어디서 터졌지"가 아니라 "ObjectMapper가 왜 못 읽었지"로 접근하게 됐다
- DTO에 기본 생성자가 왜 필요한지(또는 record가 왜 되는지), 필드명이 JSON 키와 왜 정확히 맞아야 하는지가 "관례"가 아니라 ObjectMapper의 동작 방식으로 이해됐다
- 커스텀 직렬화/역직렬화가 필요해지면 컨트롤러가 아니라 이 지점(ObjectMapper 설정, `@JsonProperty` 등)을 건드려야 한다는 게 명확해졌다

보이지 않는 곳에서 요청과 응답 양쪽을 다 쥐고 있는 녀석이라는 게, 이번에 코드를 직접 짜면서 가장 크게 남은 감각이다.
