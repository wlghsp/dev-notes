# Julia Evans (jvns.ca) 정리 로드맵

Julia Evans 블로그에서 네트워크/OS 내부 동작 원리를 다루는 글을 가져와 정리하는 트랙.
CLAUDE.md 블로그 규칙에 맞춰 정리한다 (표 금지, 개념 중심, 파일명 텍스트 참조).

## 정리 완료

- ways-dns-can-break.md — Some ways DNS can break (2022)
- what-makes-udp-interesting.md — What's interesting about UDP? (2016)
- osi-model-vs-tcp-ip.md — The OSI model doesn't map well to TCP/IP (2021)

## 다음에 정리할 후보

- How do you tell if a problem is caused by DNS? (2021) — DNS 문제인지 판별하는 법
- How to use dig (2021) — dig 명령어 실전 사용법
- DNS "propagation" is actually caches expiring (2021) — DNS propagation 개념 정정
- New tool: Mess with DNS! (2021) — DNS 실습 도구 소개
- Implementing a toy version of TLS 1.3 (2022) — TLS 1.3 직접 구현하며 이해하기
- Introducing "Implement DNS in a Weekend" (2023) — DNS 리졸버 직접 구현

### 리눅스/OS — strace 시리즈

- A zine about strace (2015) — strace가 시스템 콜을 추적하는 원리
- What problems do people solve with strace? (2021) — strace의 실전 사용 사례
- Why strace doesn't work in Docker (2020) — 컨테이너 환경에서 strace가 실패하는 이유(권한/네임스페이스)

다음에 "julia evans 로드맵에서 다음거 정리해줘" 라고 요청하면 이 목록에서 가져와서 진행한다.
