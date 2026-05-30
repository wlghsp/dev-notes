# uri (URI 해싱)

요청 URI를 해싱해 같은 URI는 항상 같은 서버로 보내는 로드밸런싱 알고리즘.

## 동작

`/api/users/1`과 `/api/users/2`는 다른 서버로 갈 수 있지만, 동일한 URI는 항상 동일한 서버로 라우팅된다.

```
backend cache_back
    balance uri
    server cache1 192.168.1.10:8080 check
    server cache2 192.168.1.11:8080 check
```

URI 전체를 해싱하는 것이 기본이고, `uri whole`(기본), `uri path-only`(쿼리스트링 제외) 옵션으로 조정할 수 있다.

## 언제 쓰나

캐시 서버 앞단에 HAProxy를 두는 경우에 유용하다. 같은 리소스 요청이 항상 같은 캐시 서버로 가면 캐시 히트율이 높아진다. 여러 서버에 분산되면 각 서버가 같은 캐시를 중복으로 갖게 된다.

## 한계

서버 수가 바뀌면 해시 결과가 달라져 기존 캐시가 무효화된다. 캐시 서버를 추가/제거할 때 히트율이 일시적으로 떨어질 수 있다.

참고: roundrobin.md, leastconn.md
