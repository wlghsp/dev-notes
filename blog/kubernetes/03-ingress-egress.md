# Kubernetes Ingress와 Egress: 들어옴과 나감

Ingress와 Egress는 Kubernetes에서 트래픽의 방향을 가리키는 한 쌍의 단어다. 어원이 방향 그 자체라서, 뜻만 알면 사용처가 금방 정리된다.

## 어원: 걸어 들어옴과 걸어 나감

두 단어 모두 라틴어 gradi(걷다, 걸음을 옮기다)에서 파생됐다.

- **Ingress**: 라틴어 ingressus, "안으로 들어감". in(안으로) + gradi(걷다)
- **Egress**: 라틴어 egressus, "밖으로 나감". e(밖으로) + gradi(걷다)

건물의 입구와 출구를 영어로 ingress, egress라고 쓰는 것과 같다. 비상구 표지판의 EGRESS가 이 단어다.

## Kubernetes에서의 의미

기준은 항상 **클러스터(또는 Pod)를 기준으로 한 방향**이다.

```mermaid
flowchart LR
    C[외부 클라이언트] -- ingress --> K[클러스터 / Pod]
    K -- egress --> E[외부 DB, API]
```

- Ingress: 밖에서 안으로 들어오는 트래픽
- Egress: 안에서 밖으로 나가는 트래픽

이 단어는 두 곳에서 쓰이는데 의미가 조금씩 다르다.

**1. NetworkPolicy의 ingress / egress 규칙**
Pod에 들어오는 트래픽과 나가는 트래픽을 따로 허용하거나 막는다. 여기서는 순수하게 방향을 가리키는 말이다.

**2. Ingress 리소스**
외부 HTTP 요청을 클러스터 안의 Service로 연결하는 오브젝트다. 이름이 Ingress인 이유는 "클러스터 안으로 들어오는 입구"를 정의하기 때문이다. Egress라는 이름의 대응 리소스는 없다.

## Ingress에서 Gateway API로

Ingress 리소스는 기능이 동결(feature-frozen)된 상태이고, 후속 규격으로 Gateway API가 자리 잡고 있다. Gateway와 HTTPRoute 같은 리소스를 쓴다. 이름도 "입구(Ingress)"에서 "관문(Gateway)"으로 바뀌었다. Ingress가 들어오는 방향이라는 하나의 의미만 담았다면, Gateway는 트래픽이 지나는 문이라는 더 넓은 의미를 쓴다.

Kind 클러스터 위에 Traefik을 Gateway로 두고 HTTPRoute로 Service에 연결하는 구성이 바로 이 방식이다.

## Recap

Ingress는 "안으로 걸어 들어옴", Egress는 "밖으로 걸어 나감"이라는 뜻 그대로 트래픽 방향을 가리킨다. 기준은 항상 클러스터(Pod)다. NetworkPolicy에서는 두 방향을 모두 다루고, Ingress 리소스는 들어오는 쪽만 정의한다. Ingress의 후속 규격인 Gateway API에서는 입구라는 이름이 관문(Gateway)으로 바뀌었다.
