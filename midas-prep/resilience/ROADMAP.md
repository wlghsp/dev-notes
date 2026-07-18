# Resilience4j / Circuit Breaker 학습 로드맵

마이다스 백엔드(CDC 기술자) 채용공고 준비용. "장애 상황에서도 서비스 지속성을 보장하는 고가용성 아키텍처 설계", "Retry, Circuit Breaker, 비동기 처리 등을 활용한 안정성 및 확장성 개선"이 주요 업무로 명시됨. 인재상에도 "장애를 사후 대응이 아닌 구조 설계로 예방"이 명시되어 있어 비중이 낮지 않은 주제.

## 배경

CDC 파이프라인과 downstream 시스템(Kafka consumer, 외부 API 등)이 얽힌 구조에서는 한 시스템의 장애가 전체로 전파되기 쉽다. Resilience4j는 Spring 생태계에서 이런 장애 전파를 막는 패턴들을 라이브러리로 제공한다.

## 개념 목록 (예상 — 학습하며 실제 생성 목록으로 교체)

- [ ] circuit-breaker — 실패가 반복되는 대상 호출을 일시적으로 차단해서 장애를 전파시키지 않는 패턴 (Closed/Open/Half-Open 상태)
- [ ] retry-backoff — 재시도 시 간격을 늘려가며 재시도하는 이유 (thundering herd 방지)
- [ ] bulkhead — 자원 풀을 격리해서 한 기능의 장애가 다른 기능까지 잠식하지 않게 하는 패턴
- [ ] rate-limiter — 호출 빈도를 제한해서 downstream 과부하를 막는 패턴
- [ ] resilience4j-vs-hystrix — Hystrix가 아니라 Resilience4j를 쓰는 이유 (경량, 함수형, 유지보수 상태)

## 진행 상태

로드맵만 작성. 아직 개념 파일 생성 전.

## 완료 후 참고 (실제 생성된 파일만 기록 — 예상 아님)

(아직 없음)
