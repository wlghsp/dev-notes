# WireMock

HTTP 기반 API를 흉내 내는 mock 서버 도구다. 실제 외부 시스템을 띄우지 않고도, 정해진 요청에 정해진 응답을 돌려주는 가짜 서버를 만들어준다.

## 왜 필요한가

외부 API에 의존하는 코드를 테스트하거나 실습하려면 그 외부 API가 항상 응답 가능한 상태여야 한다. 하지만 실제 외부 시스템은 로컬 환경에 없거나, 테스트마다 특정 응답(느린 응답, 에러, 특정 데이터)을 재현하기 어렵다. WireMock은 이 외부 시스템 자리에 대신 세워두는 서버로, 요청 경로와 조건에 따라 원하는 응답을 그대로 지정할 수 있다.

## 매핑(mapping)

WireMock이 어떤 요청에 어떤 응답을 줄지 정의한 규칙을 매핑이라 부른다. JSON 파일로 작성하며, 요청 조건(경로, 메서드, 파라미터)과 응답 내용(상태 코드, 바디, 지연 시간)을 지정한다.

```json
{
  "request": {
    "method": "GET",
    "urlPathPattern": "/some/path"
  },
  "response": {
    "status": 200,
    "fixedDelayMilliseconds": 8000,
    "jsonBody": { "key": "value" }
  }
}
```

`fixedDelayMilliseconds`처럼 응답을 일부러 지연시키는 설정을 활용하면, 실제 방화벽 차단이나 서버 무응답 같은 장애 상황을 재현성 있게 만들 수 있다.

## 장애 재현 실습에서의 쓰임

정상 응답을 주던 매핑에 지연이나 에러를 추가하면, 타임아웃·서킷브레이커 같은 장애 대응 코드가 실제로 어떻게 동작하는지 로컬에서 반복 재현하며 관찰할 수 있다. 다만 매핑 파일을 고친 뒤 바로 반영되지 않는 함정이 있다 — 자세한 내용은 wiremock-mapping-reload.md 참고.

참고: wiremock-mapping-reload.md
참고: connect-timeout-vs-read-timeout.md
