# task-decomposition (작업 분해)

프로그램을 기능적으로 독립된 작업들로 쪼개어 동시에 실행할 수 있게 만드는 분해 방식.

"문제를 동시에 실행 가능한 독립적인 기능 단위로 어떻게 쪼갤 것인가"라는 질문에 답한다.

task parallelism이라고도 불린다.

## 핵심 아이디어

문제가 본래 서로 다른 종류의 작업들로 구성될 때 적용한다. 각 작업이 서로 다른 기능을 수행하며, 데이터를 독립적으로 다룬다.

예를 들어 이메일 앱은 UI 렌더링, 이메일 수신, 이메일 전송이라는 기능들이 서로 독립적이다. 같은 데이터를 다루더라도 서로의 실행에 의존하지 않으므로 각각 별도 스레드(또는 프로세스)로 분리할 수 있다.

중요한 점: 혼자서 삽질하는 것보다 친구를 불러서 나는 눈을 치우고 친구는 소금을 뿌리는 식으로 역할을 나누는 게 효율적이다. 같은 삽 하나를 번갈아 쓰는 것은 context switch 비용만 늘릴 뿐이다.

## 적용 가능한 아키텍처

task decomposition은 서로 다른 명령어를 다른 데이터에 실행하므로 MIMD(Multiple Instruction Multiple Data)와 MISD(Multiple Instruction Single Data) 시스템에서 쓸 수 있다.

## 의존성 분석

분해 전 먼저 작업 의존성 그래프(task dependency graph)를 그린다. 그래프에서 직접 연결된 엣지가 없는 작업들이 독립적으로 실행 가능한 후보다.

의존성이 있는 작업들은 순서를 지켜야 하므로 동시 실행 불가다. 독립적인 작업만 병렬화할 수 있다.

## 한계

task decomposition은 직관적이지 않고 주관적이다. 같은 문제를 보더라도 어떻게 쪼개느냐에 따라 결과가 다르다. 또한 작업의 기능이 다양하면 자동화하기 어렵다.

참고: data-decomposition.md, pipeline-pattern.md, granularity.md, dependency-graph.md
