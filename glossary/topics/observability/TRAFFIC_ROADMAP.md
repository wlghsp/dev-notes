# 트래픽·성능 지표 학습 로드맵

> 목표: p95/p99/TPS부터 용량 산정, 장애 방어, 알람 설계까지 팀 대화에서 근거를 들어 말할 수 있는 수준.
> 개인 학습용 로드맵이라 이해도 테스트나 Phase 완료 기준은 두지 않는다. 각 문서 끝의 "스스로 점검"으로 확인한다.
> 다음 단계 문서는 지호님이 요청할 때 만든다. Claude가 먼저 만들지 않는다.
> 로드맵 파일에는 실제로 만든 문서만 파일명으로 적는다. 예상 문서는 후보로만 표시한다.

자료 위치: dev-notes/assets/observability/ (Release It!, Google SRE 북, System Performance, Observability Engineering)

---

## 진행 현황

- 1단계 지표 읽기: ✅ percentile-and-tps.md
- 2단계 용량과 판단 틀: ✅ traffic-basics.md
- 3단계 과부하와 장애 방어: ✅ overload-and-resilience-patterns.md
- 4단계 스레드 풀·커넥션 풀 사이징: ⬜
- 5단계 병목 찾기 (USE 심화): ⬜
- 6단계 SLO와 알람 설계: ⬜
- 7단계 부하 테스트 실전: ⬜
- 8단계 분산 트레이싱 (선택): ⬜

---

## 단계별 내용

### 1단계 — 지표 읽기
평균의 함정, 백분위수(p50/p95/p99), TPS와 rate, 히스토그램 bucket, histogram_quantile, 평균내면 안 되는 이유, Little's Law, 꼬리 지연 증폭.
**작성된 문서**: percentile-and-tps.md

### 2단계 — 용량과 판단 틀
RED/USE/Golden Signals, 동시 사용자와 TPS의 차이, 피크 기준 용량 산정, 에러 구분, 포화 방어 개요, 캐시 히트율.
**작성된 문서**: traffic-basics.md

### 3단계 — 과부하와 장애 방어
연쇄 장애, 타임아웃, 서킷 브레이커, 벌크헤드, 부하 차단, 백프레셔, 큐 길이, 재시도 폭주와 재시도 예산.
**작성된 문서**: overload-and-resilience-patterns.md
**자료**: Release It! 4~5장, SRE 북 21~22장

### 4단계 — 스레드 풀·커넥션 풀 사이징
Little's Law를 실제 풀 크기에 적용. Tomcat의 maxThreads/acceptCount/maxConnections, HikariCP 풀 크기 근거, 풀 대기 시간 지표.
**자료 후보**: HikariCP 위키(About Pool Sizing), Tomcat 공식 문서(Connector). 웹 공개 자료라 요청 시 직접 가져올 수 있다.
**작성된 문서**: (없음)

### 5단계 — 병목 찾기 (USE 심화)
RED에서 본 증상을 USE로 내려가 자원별 병목을 확인하는 방법. CPU, 메모리, 디스크, 네트워크의 사용률·포화·에러 읽기.
**자료**: System Performance (방법론 장과 자원별 장)
**작성된 문서**: (없음)

### 6단계 — SLO와 알람 설계
SLI/SLO/SLA, 에러 버짓, burn rate 기반 알람, 알람을 걸 것과 걸지 않을 것.
**자료**: SRE 북 모니터링·SLO 관련 장, Observability Engineering의 SLO 관련 장
**작성된 문서**: (없음)

### 7단계 — 부하 테스트 실전
시나리오 설계, think time 반영, 한계 지점 찾기, 결과를 앞 단계 지표로 해석하기, coordinated omission 회피. 1~6단계를 우리 환경에서 검증하는 종착점.
**자료**: 웹 자료와 실제 환경 (sns-app 등)
**작성된 문서**: (없음)

### 8단계 — 분산 트레이싱 (선택)
느린 요청이 어느 호출 구간에서 느린지 찾기. 이 환경에서 필요해질 때 진행한다.
**자료**: Observability Engineering
**작성된 문서**: (없음)

---

## 순서 메모

- 4단계는 3단계에서 "풀"이 반복해서 나왔기 때문에 바로 이어서 가는 것이 자연스럽다.
- 5단계와 6단계는 서로 의존하지 않아 순서를 바꿔도 된다.
- 7단계는 최소 4단계까지 끝낸 뒤에 하는 편이 낫다. 풀 크기를 모르면 테스트 결과를 해석하기 어렵다.
