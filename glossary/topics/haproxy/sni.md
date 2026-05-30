# SNI (Server Name Indication)

TLS 핸드셰이크 초기에 클라이언트가 접속하려는 도메인 이름을 서버에 알려주는 TLS 확장(extension).

## 왜 필요한가

하나의 IP에서 여러 도메인을 호스팅할 때 문제가 생긴다. HTTPS 연결은 TLS 핸드셰이크가 먼저 일어나고 그 안에서 HTTP 요청이 오가는데, 핸드셰이크 시점에 서버는 어떤 도메인으로 접속한 건지 알 수 없다. 어떤 인증서를 내려줘야 할지 모른다.

SNI는 핸드셰이크 초기(ClientHello)에 클라이언트가 도메인명을 포함시켜 이 문제를 해결한다.

## HAProxy에서의 활용

SSL Termination 모드에서는 HAProxy가 TLS를 처리하므로 SNI를 자연스럽게 읽을 수 있다. 요청한 도메인에 따라 다른 인증서를 내려주거나, 다른 backend로 라우팅할 수 있다.

SSL Passthrough(TCP 모드)에서도 SNI 필드는 암호화되지 않아 읽을 수 있다. 내용은 볼 수 없어도 도메인 기준 라우팅은 가능하다.

```
# SSL Termination에서 SNI로 인증서 자동 선택
frontend https_front
    bind *:443 ssl crt /etc/haproxy/certs/    # 디렉토리 지정 시 SNI로 자동 선택
    default_backend api_back
```

여러 인증서 파일을 디렉토리에 넣으면 HAProxy가 SNI를 보고 맞는 인증서를 자동으로 선택한다.

## ESNI / ECH

SNI는 평문으로 전송되기 때문에 중간에서 어떤 도메인에 접속하는지 볼 수 있다. 이를 암호화한 것이 ESNI, 그리고 더 발전된 ECH(Encrypted Client Hello)다. 현재 일부 브라우저에서 지원한다.

참고: ssl-termination.md, ssl-passthrough.md
