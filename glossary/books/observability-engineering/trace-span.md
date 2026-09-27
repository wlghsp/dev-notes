# Trace와 Span

Trace는 요청 하나가 여러 서비스를 거치며 처리되는 전체 과정을 나타내는, 서로 연결된 이벤트들의 집합이다. Span은 그 과정을 이루는 개별 조각으로, 하나의 서비스 또는 하나의 작업 단위가 처리한 부분을 나타낸다.

요청이 서비스 A → B → C 순서로 흘러가면, A/B/C 각각의 작업이 하나씩의 span이 되고 그 span들을 모으면 하나의 trace가 된다.

## Span 간의 관계: parent-child

Span은 root span(그 trace에서 최상위, 가장 먼저 시작된 span)과 그 아래 중첩된 span들로 구성된다. A가 B를 호출하고 B가 C를 호출하면, A의 span이 B의 span의 parent이고, B의 span이 C의 span의 parent다. 하나의 서비스가 같은 trace 안에서 여러 번 등장할 수도 있다(재귀 호출이나 병렬 처리 등).

## Span을 구성하는 필수 필드

- Trace ID: 이 trace 전체를 식별하는 고유 값. root span에서 생성되어 이후 모든 하위 span에 전파된다.
- Span ID: 개별 span을 식별하는 고유 값.
- Parent ID: 이 span의 부모 span을 가리키는 값. root span은 parent ID가 없다.
- Timestamp: span이 시작된 시각.
- Duration: span이 처리되는 데 걸린 시간.

이 다섯 가지가 있어야 waterfall 형태의 trace 시각화를 재구성할 수 있다. 이 외에 service name, span name, 그리고 자유롭게 추가하는 custom field(태그)들이 span을 더 유용하게 만든다.

## 서비스 경계를 넘어 전파되는 방법

같은 프로세스 안에서는 span 정보가 메모리에서 관리되지만, 서비스 경계를 넘어갈 때는 이 정보를 어떻게든 실어 보내야 한다. 가장 흔한 방법은 HTTP 헤더에 trace id와 parent span id를 실어 보내는 것이다. W3C Trace Context나 B3 같은 표준이 이 헤더 형식을 정의한다. 수신 측 서비스는 헤더에서 이 값을 꺼내 자신의 span을 생성하고, 다시 trace id를 물려받아 다음 서비스로 전파한다. 이 흐름 전체를 context propagation이라 부른다.

## Trace가 service-to-service 호출에만 국한되지 않는 이유

Trace/span의 개념은 원격 호출 사이의 관계를 표현하는 데서 시작했지만, 굳이 분산 호출이 아니어도 쓸 수 있다. 하나의 서비스 안에서 유난히 느린 구간(JSON 파싱처럼 CPU를 많이 쓰는 부분)을 별도 span으로 감싸서 어디서 시간이 소모되는지 더 세밀하게 볼 수도 있다. 즉 trace를 구성하는 이벤트들이 서로 연결되어 있기만 하면, 그 구조를 그대로 활용할 수 있다.

참고: structured-event.md, w3c-trace-context.md

## Recap

Trace는 요청 하나가 여러 서비스를 거치는 전체 과정이고, Span은 그 과정의 개별 조각이다. Span들은 trace id / span id / parent id / timestamp / duration 다섯 필드로 부모-자식 관계를 맺으며, 서비스 경계를 넘을 때는 이 정보를 HTTP 헤더 등으로 실어 보내는 context propagation이 필요하다. Trace는 분산 호출뿐 아니라 하나의 서비스 내부에서 느린 구간을 쪼개 보는 데도 쓸 수 있다.
