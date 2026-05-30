# SSL Termination (SSL 종료)

클라이언트와 HAProxy 사이에서 TLS를 처리하고, 백엔드 서버로는 평문 HTTP로 전달하는 방식.

## 구조

```
클라이언트 ---HTTPS---> HAProxy ---HTTP---> 백엔드 서버
```

클라이언트는 HAProxy와 TLS 핸드셰이크를 맺는다. HAProxy가 TLS를 복호화한 뒤 내부 서버로 평문으로 전달한다. 백엔드 서버는 TLS 처리를 하지 않아도 된다.

## 장점

백엔드 서버들이 TLS 처리 부담을 지지 않아도 된다. 인증서 관리를 HAProxy 한 곳에서만 하면 된다. 서버를 내부망에서 평문으로 운영할 수 있어 구성이 단순하다.

ACL, 헤더 조작 등 L7 기능을 TLS 복호화 후에 적용할 수 있다.

## 설정 예시

```
frontend https_front
    bind *:443 ssl crt /etc/haproxy/certs/example.com.pem
    default_backend api_back

backend api_back
    server api1 192.168.1.10:8080 check
    server api2 192.168.1.11:8080 check
```

`.pem` 파일은 인증서 + 개인키를 합친 파일이다.

## SSL Passthrough와의 차이

SSL Termination은 HAProxy가 TLS를 복호화한다. 백엔드 서버는 평문을 받는다.

SSL Passthrough는 HAProxy가 TLS를 건드리지 않고 그대로 서버로 넘긴다. TCP 모드에서 사용하고, HAProxy는 내용을 볼 수 없다. 서버가 직접 TLS를 처리한다.

참고: ssl-passthrough.md
