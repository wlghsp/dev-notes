# SSL Passthrough (SSL 통과)

HAProxy가 TLS를 복호화하지 않고 암호화된 패킷을 그대로 백엔드 서버로 전달하는 방식.

## 구조

```
클라이언트 ---HTTPS---> HAProxy ---HTTPS---> 백엔드 서버
```

HAProxy는 내용을 열어보지 않는다. TCP 모드에서만 동작한다.

## 설정 예시

```
frontend https_front
    bind *:443
    mode tcp
    default_backend api_back

backend api_back
    mode tcp
    server api1 192.168.1.10:443 check
    server api2 192.168.1.11:443 check
```

## SSL Termination과의 차이

SSL Passthrough는 HAProxy가 내용을 볼 수 없어서 ACL로 URL이나 헤더를 판단하는 L7 기능을 쓸 수 없다. TCP 모드라 IP와 포트만으로 라우팅해야 한다.

반면 end-to-end 암호화가 유지된다. 백엔드 서버까지 평문이 노출되지 않아야 하는 보안 요구사항이 있을 때 선택한다.

## SNI 기반 라우팅

TCP 모드지만 TLS의 SNI(Server Name Indication) 필드는 암호화되지 않는다. HAProxy는 SNI를 읽어 도메인 기준으로 라우팅할 수 있다.

```
frontend https_front
    bind *:443
    mode tcp
    tcp-request inspect-delay 5s
    tcp-request content accept if { req_ssl_hello_type 1 }
    use_backend api_back if { req_ssl_sni -i api.example.com }
    use_backend web_back if { req_ssl_sni -i web.example.com }
```

참고: ssl-termination.md, sni.md
