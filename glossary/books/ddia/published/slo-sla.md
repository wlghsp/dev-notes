# SLO / SLA / SLI
참고: DDIA Ch.1 — Reliable, Scalable, and Maintainable Applications

---

세 개념은 계층 구조로 연결된다.

## SLI (Service Level Indicator)

측정 가능한 실제 지표. "지금 시스템이 어떤 상태인가"를 숫자로 나타낸 것이다.

예시:
- 최근 5분간 p99 응답 시간: 230ms
- 지난 30일간 요청 성공률: 99.97%
- 에러율: 0.03%

SLI는 퍼센타일(p95, p99 등) 기반 응답 시간이 가장 많이 쓰인다. 평균은 outlier에 묻히기 때문에 잘 쓰지 않는다.
참고: latency-vs-response-time.md

## SLO (Service Level Objective)

SLI에 대해 팀 내부에서 정한 목표값. "이 수치를 지키겠다"는 내부 기준선이다.

예시:
- p99 응답 시간 < 300ms (30일 롤링 윈도우 기준)
- 가용성 > 99.9%
- 에러율 < 0.1%

SLO는 알림(alert) 기준이 된다. p99가 SLO를 초과하면 on-call 엔지니어에게 알림을 보내는 식이다.

SLO를 정할 때 흔한 실수는 "99.999% (five nines)"처럼 너무 높게 잡는 것이다. 목표가 높을수록 달성 비용이 기하급수적으로 올라간다. 99.9% → 99.99%로 올리면 허용 다운타임이 1/10로 줄어든다.

## SLA (Service Level Agreement)

SLO를 외부 고객 또는 계약 상대방과 맺는 약속. SLO의 외부 버전이다. 위반 시 환불, 크레딧 지급 등 계약상 결과가 따른다.

예시:
- "월간 가용성 99.9% 미달 시 서비스 크레딧 10% 지급"
- AWS, GCP 같은 클라우드 벤더가 고객과 맺는 계약이 대표적

SLA는 보통 SLO보다 느슨하게 잡는다. SLO를 내부 목표로 더 엄격하게 유지하고, SLA는 그 밑에 안전망으로 둔다. SLO가 깨져도 SLA는 유지되는 구조.

---

## 세 개념의 관계 요약

SLI는 측정, SLO는 내부 목표, SLA는 외부 계약이다.

실제 흐름:
1. SLI를 수집한다 (예: p99 = 230ms)
2. SLO와 비교한다 (예: p99 < 300ms → 현재 통과)
3. SLO 위반이 누적되면 SLA 위반 위험 신호

---

## error budget

SLO와 함께 자주 쓰이는 개념이다.

"99.9% 가용성"을 SLO로 잡으면 한 달에 허용되는 다운타임은 약 43분이다. 이 43분이 error budget이다.

error budget이 남아있으면 배포와 실험을 더 적극적으로 할 수 있다. 다 소진되면 안정화에 집중해야 한다. SRE(Site Reliability Engineering) 팀이 개발팀과 배포 속도를 협상할 때 error budget을 기준으로 쓴다.
