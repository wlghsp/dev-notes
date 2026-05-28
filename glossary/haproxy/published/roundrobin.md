# Round Robin

HAProxy의 기본 로드밸런싱 알고리즘. 서버 목록을 순서대로 돌아가며 요청을 분배한다.

## 동작

서버가 A, B, C 세 개라면 요청을 A → B → C → A → B → C 순서로 보낸다. 서버마다 가중치(weight)를 다르게 설정하면 비율을 조정할 수 있다.

```
backend api_back
    balance roundrobin
    server api1 192.168.1.10:8080 check weight 2
    server api2 192.168.1.11:8080 check weight 1
```

위 설정이면 api1이 api2보다 2배 많은 요청을 받는다.

## 언제 쓰나

요청마다 처리 시간이 비슷하고 서버 스펙이 동일할 때 적합하다. 요청 하나가 끝났는지와 상관없이 순서대로 보내기 때문에, 처리 시간이 긴 요청이 섞이면 특정 서버에 부하가 몰릴 수 있다.

참고: leastconn.md, source.md, uri.md
