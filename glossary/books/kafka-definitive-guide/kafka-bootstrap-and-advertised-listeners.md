# Bootstrap과 Advertised Listeners

클라이언트(producer, consumer)가 Kafka 클러스터에 처음 연결할 때부터 실제로 원하는 파티션의 브로커에 도달하기까지의 흐름.

## listeners: 브로커가 여는 주소

브로커는 `server.properties`의 `listeners` 설정(예: `PLAINTEXT://broker1:9092`)에 정의된 IP:port로 TCP 소켓을 열고 리스닝한다. 이 주소가 바로 kafka-wire-protocol.md에서 다루는 바이너리 프로토콜이 얹히는 연결 지점이다.

## bootstrap.servers: 클라이언트의 진입점

클라이언트는 클러스터의 브로커 전체 주소를 처음부터 알 필요가 없다. 설정값 `bootstrap.servers`에 브로커 몇 대의 주소만 적어두면, 클라이언트는 그중 하나에 TCP 연결을 맺고 MetadataRequest를 보낸다.

브로커는 이 요청에 응답해서 클러스터 전체의 메타데이터, 즉 어떤 토픽의 어떤 파티션이 어떤 브로커에 있는지(그리고 어느 브로커가 leader인지, 참고: kafka-broker-and-cluster.md)를 클라이언트에 돌려준다.

클라이언트는 이 메타데이터를 받은 뒤, 자신이 실제로 쓰거나 읽어야 할 파티션을 담당하는 브로커에 직접 새 연결을 맺는다. 그래서 `bootstrap.servers`에 클러스터의 브로커를 전부 나열할 필요는 없다 — 일부만 살아있어도 나머지 메타데이터를 그 브로커로부터 얻어올 수 있다.

## advertised.listeners: 브로커가 스스로를 광고하는 주소

`listeners`가 브로커 프로세스가 실제로 바인딩하는 주소라면, `advertised.listeners`는 브로커가 MetadataRequest 응답에 담아 클라이언트에게 "나에게 연결하려면 이 주소로 오라"고 알려주는 주소다.

이 둘이 갈라지는 대표적인 상황이 컨테이너/클라우드 환경이다. 브로커 프로세스는 컨테이너 내부 주소(예: `0.0.0.0:9092`)에 바인딩하지만, 클러스터 외부의 클라이언트는 그 내부 주소로 접근할 수 없다. 이럴 때 `advertised.listeners`에 외부에서 접근 가능한 주소(예: `broker1.example.com:9092`)를 따로 지정해서, 클라이언트가 메타데이터 응답을 받고도 연결에 실패하는 문제를 막는다.

`advertised.listeners`가 설정되지 않으면 `listeners` 값이 그대로 광고된다.

참고: kafka-wire-protocol.md, kafka-broker-and-cluster.md
