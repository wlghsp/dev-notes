# backward-forward-compatibility (하위/상위 호환성)

참고: schema-evolution.md, encoding.md, dataflow.md

---

신버전과 구버전 코드, 신버전과 구버전 데이터가 시스템 안에 동시에 존재할 수 있는 상황에서 시스템이 계속 정상 동작하려면 두 방향의 호환성이 모두 필요하다.

## Backward compatibility (하위 호환성)

신버전 코드가 구버전 코드가 쓴 데이터를 읽을 수 있는 것. 신버전 코드를 작성하는 입장에서는 구버전 데이터 형식을 이미 알고 있으므로 명시적으로 처리하면 되기 때문에 상대적으로 달성하기 쉽다.

## Forward compatibility (상위 호환성)

구버전 코드가 신버전 코드가 쓴 데이터를 읽을 수 있는 것. 구버전 코드는 신버전에서 무엇이 추가됐는지 알 수 없으므로, 모르는 필드를 만나면 무시하고 넘어가는 능력이 필요하다. 이게 backward compatibility보다 더 까다로운 이유다.

## 어디서 필요한가

이 두 방향은 데이터가 흐르는 경로(dataflow.md)마다 요구되는 정도가 다르다.

- DB를 통한 흐름: 여러 프로세스가 같은 DB를 동시에 읽고 쓰기 때문에, 그리고 신버전이 쓴 값을 구버전이 나중에 읽고 다시 쓰는 상황이 생길 수 있기 때문에 backward/forward 모두 필요하다. 이때 구버전 코드가 모르는 필드를 읽어서 그대로 다시 썼을 때, 그 필드를 보존하지 못하고 유실시키는 경우가 있어 애플리케이션 레벨에서 주의가 필요하다.
- 서비스 호출(REST/RPC): 서버가 항상 먼저 배포되고 클라이언트가 나중에 배포된다고 가정할 수 있으므로, 요청에는 backward compatibility만, 응답에는 forward compatibility만 있으면 충분하다.
- 메시지 큐를 통한 흐름: publisher와 consumer를 독립적으로 배포하고 순서에 상관없이 동작하게 하려면 인코딩이 양방향 모두 호환돼야 한다.

## 호환성의 실제 구현

각 인코딩 포맷이 이를 어떻게 보장하는지는 protobuf-thrift-avro.md에 정리했다. 공통적으로, 필드를 새로 추가할 때 기본값을 주거나 optional로 만드는 것이 backward compatibility의 핵심이고, 옛날 코드가 모르는 필드를 무시하도록 만드는 것이 forward compatibility의 핵심이다.
