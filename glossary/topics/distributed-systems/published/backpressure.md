# Backpressure (백프레셔)

**한 줄 정의**: 데이터를 받는 쪽(downstream)이 처리 속도를 감당 못할 때, 보내는 쪽(upstream)에게 속도를 늦추라고 신호를 보내는 메커니즘.

---

## Upstream / Downstream 이란

데이터 흐름에는 방향이 있다. 그 방향을 기준으로 두 가지 위치를 구분한다.

**Upstream**: 데이터를 만들어서 보내는 쪽. 생산자(Producer)에 가깝다.  
**Downstream**: 데이터를 받아서 처리하는 쪽. 소비자(Consumer)에 가깝다.

```
Upstream → → → Downstream
[Producer]       [Consumer]
```

이 용어는 시스템 어느 지점에서든 상대적으로 쓰인다. A → B → C 파이프라인에서 B 입장에서 A는 upstream, C는 downstream이다. 절대적인 위치가 아니라 데이터 흐름 기준의 상대 위치다.

### 어원

강(river) 비유에서 온 말이다.

**stream**은 원래 "흐르는 물", 시내, 강줄기. 물이 끊임없이 흐르듯 데이터가 연속으로 흘러간다는 뜻에서 그대로 가져왔다.

**upstream**은 강에서 "물이 흘러오는 방향", 즉 상류(上流). 근원지 쪽이다.  
**downstream**은 반대로 "물이 흘러가는 방향", 하류(下流). 물이 도달하는 쪽이다.

이 비유는 소프트웨어 여러 곳에서 같은 방식으로 쓰인다.

- 오픈소스: 원본 프로젝트가 upstream, fork한 프로젝트가 downstream. Debian은 Linux kernel의 downstream이다.
- 공급망: 원자재 공급사가 upstream, 완제품 판매사가 downstream.
- git: `git push origin`에서 origin이 upstream, 로컬이 downstream.

전부 "근원 → 파생" 방향이 공통이다. 무언가가 흘러가는 방향의 앞이냐 뒤냐.

---

## 왜 백프레셔가 필요한가

생산자가 소비자보다 빠를 때 문제가 생긴다.

```
Producer (초당 10,000건)
        ↓
  [메모리 버퍼]   ← 점점 차오름
        ↓
Consumer (초당 1,000건)
```

버퍼가 꽉 차면 선택지는 두 가지다.

1. 새 데이터를 버린다 (loss)
2. 메모리가 터진다 (OOM)

어느 쪽이든 나쁘다. 백프레셔는 이 상황을 "버리거나 터지기 전에 미리 upstream에게 알리는" 방식으로 해결한다.

---

## 백프레셔의 동작 방식

핵심은 downstream이 처리 가능한 양을 upstream에게 명시적으로 알리는 것이다.

```
Consumer → "나 지금 100개밖에 못 받아" → Producer
Producer → (100개만 전송) →
Consumer → "이제 500개 받을 수 있어" →
Producer → (500개 전송) →
```

이 신호가 역방향으로 흐르기 때문에 "pressure(압력)가 upstream으로 거슬러 올라간다(back)"는 표현을 쓴다.

---

## 실제 시스템에서 어떻게 구현되는가

**TCP**: 수신 윈도우 크기(receive window)로 구현한다. 수신 버퍼가 가득 차면 윈도우 크기를 0으로 알려 송신을 멈추게 한다. OS 레벨에서 자동으로 처리된다.

**Kafka**: Consumer가 직접 `poll()`을 호출해 가져가는 구조(pull 방식)라, 소비자가 처리 가능할 때만 데이터를 가져온다. 이 자체가 백프레셔의 형태다. Consumer lag(얼마나 밀려 있는가)를 모니터링해 upstream 생산 속도를 조절한다.

**Reactive Streams (Project Reactor, RxJava)**: `request(n)` 메서드로 소비자가 몇 개를 처리할 수 있는지 명시적으로 요청한다. 생산자는 그 수만큼만 방출한다. Spring WebFlux가 이 모델 위에 있다.

**gRPC / HTTP2**: 플로우 컨트롤 메커니즘이 내장되어 있어 수신측이 처리 가능한 바이트 수를 송신측에 알린다.

---

## 백프레셔가 없을 때 흔히 일어나는 증상

- Consumer 서비스의 메모리 사용량이 지속적으로 증가한다
- 특정 시간대에 OOM이 발생하고 Pod가 재시작된다
- 큐나 버퍼가 가득 차서 데이터가 유실된다
- Producer는 정상인데 Consumer 쪽 지연(latency)이 폭발적으로 늘어난다

---

## 백프레셔 vs. 버퍼링 vs. 쓰로틀링

세 가지는 모두 속도 차이를 다루지만 접근이 다르다.

**버퍼링(Buffering)**: 속도 차이를 임시로 흡수한다. 근본적인 해결이 아니라 시간을 버는 것이다. 버퍼가 차면 결국 같은 문제가 생긴다.

**쓰로틀링(Throttling)**: Upstream이 일방적으로 속도를 제한한다. Downstream의 현재 상태를 반영하지 않는다.

**백프레셔**: Downstream이 자신의 처리 가능 용량을 upstream에게 알려서, 양쪽이 협력해 속도를 맞춘다. 동적이고 반응적이다.

---

## 관련 개념
- 참고: stream-processing.md
- 참고: source-transform-sink.md
