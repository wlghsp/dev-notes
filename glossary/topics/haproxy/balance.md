# balance

HAProxy backend 블록에서 로드밸런싱 알고리즘을 지정하는 지시어.

```
backend api_back
    balance roundrobin
```

`balance` 뒤에 알고리즘 이름을 붙여 "어떤 기준으로 서버를 선택할 것인가"를 선언한다.

## 알고리즘 종류

- `roundrobin` — 순서대로 분배. 기본값. 참고: roundrobin.md
- `leastconn` — 현재 연결 수가 가장 적은 서버. 참고: leastconn.md
- `source` — 클라이언트 IP 해싱으로 서버 고정. 참고: source.md
- `uri` — 요청 URI 해싱으로 서버 고정. 참고: uri.md
- `random` — 무작위 선택. 대규모 클러스터에서 roundrobin보다 분산이 고를 수 있다.
