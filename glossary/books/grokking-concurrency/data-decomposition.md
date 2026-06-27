# data-decomposition

동일한 연산을 서로 다른 데이터 조각에 독립적으로 적용함으로써 병렬 처리를 실현하는 분해 방식.

"데이터를 서로 독립적으로 처리 가능한 청크로 어떻게 나눌 것인가"라는 질문에 답한다. 작업의 종류가 아니라 데이터가 분해의 기준이다.

## task decomposition과의 차이

task decomposition은 서로 다른 기능을 병렬로 실행한다(세탁 vs 소금 뿌리기). data decomposition은 같은 기능을 서로 다른 데이터에 병렬로 실행한다(삽 두 개로 각자 다른 구역의 눈을 치우기).

## loop-level parallelism

data decomposition의 가장 흔한 형태. 루프의 각 반복이 서로 독립적일 때, 반복을 여러 스레드에 분산해서 동시에 실행한다.

일반 for 루프에서 각 반복이 이전 반복의 결과에 의존하지 않는다면 data decomposition 적용 후보다.

## 구현 패턴들

map pattern — 컬렉션의 모든 원소에 같은 함수를 독립적으로 적용한다. 각 작업이 프로그램 상태를 변경하지 않고 입력을 출력으로 변환하는 순수 함수여야 한다. side effect가 없어야 한다.

fork/join pattern — 데이터를 여러 청크로 나눠서(fork) 병렬로 처리한 후, 결과를 하나로 합친다(join). join 단계는 순차적이며 모든 fork 작업이 끝날 때까지 기다리는 동기화 지점이다.

map/reduce pattern — map 단계에서 각 데이터 청크에 같은 연산을 병렬 적용하고, reduce 단계에서 결과를 집계한다. fork/join과 구조적으로 유사하지만, 단일 머신을 넘어 여러 컴퓨터로 수평 확장이 가능하다. Google MapReduce, Apache Hadoop, Apache Spark가 이 패턴 기반이다.

## 적합한 아키텍처

동일한 연산을 여러 데이터에 동시 적용하므로 SIMD 아키텍처(Single Instruction Multiple Data)에 가장 잘 맞는다. 분산 시스템에서는 MIMD 방식으로도 구현된다.

data decomposition은 실제 병렬 하드웨어가 있을 때 의미가 있다. 하드웨어 없이는 효과가 거의 없다.

> 📷 Figure 7 (책 p.122) — data decomposition의 chunk → task → result 구조 다이어그램
> 📷 Figure 7 (책 p.126) — map pattern 다이어그램
> 📷 Figure 7 (책 p.130) — fork/join pattern 다이어그램
> 📷 Figure 7 (책 p.131) — map/reduce pattern 다이어그램

참고: task-decomposition.md, pipeline-pattern.md, granularity.md, flynns-taxonomy.md
