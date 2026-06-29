# message-queue (메시지 큐)

메모리에 저장되는 내구성 있는 컴포넌트로, 비동기 통신을 지원한다. 버퍼 역할을 하며 비동기 요청을 분산 처리한다.

## 기본 구조

Producer(생산자/발행자)가 메시지를 생성해서 큐에 발행한다. Consumer(소비자/구독자)가 큐에 연결해서 메시지를 꺼내 처리한다.

> 📷 Figure 1-17 (책 p.24) — Producer → Message Queue → Consumer 기본 구조

## Decoupling의 이점

Producer는 Consumer가 처리 가능한지 상관없이 메시지를 큐에 넣을 수 있다. Consumer가 오프라인 상태여도 메시지는 큐에 남아 있다. Consumer가 준비되면 꺼내서 처리한다. Producer와 Consumer가 서로 독립적으로 확장 가능하다.

## 사용 예시

사진 처리 앱에서 웹 서버(Producer)가 사진 크롭·샤프닝 작업을 큐에 발행한다. 사진 처리 워커(Consumer)가 큐에서 작업을 꺼내 비동기로 처리한다. 큐가 크면 워커를 늘리고 큐가 비어 있으면 워커를 줄일 수 있다.

> 📷 Figure 1-18 (책 p.25) — 사진 처리 시스템에서의 메시지 큐 활용

## Logging, Metrics, Automation과 함께

대규모 시스템에서는 메시지 큐와 함께 다음 도구들도 필수가 된다.

- Logging — 서버별 에러 로그를 중앙 집중식 서비스로 모아 검색
- Metrics — CPU, 메모리, 캐시 성능, 일일 활성 사용자 같은 지표 수집
- Automation — CI/CD로 코드 체크인마다 자동 검증, 빌드·테스트·배포 자동화

> 📷 Figure 1-19 (책 p.26) — 메시지 큐 + Logging·Metrics·Automation 추가된 아키텍처

참고: stateless-architecture.md, data-center.md
