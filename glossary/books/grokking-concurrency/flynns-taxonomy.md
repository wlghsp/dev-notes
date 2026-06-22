# Flynn's Taxonomy

컴퓨터 아키텍처를 명령어 흐름(instruction)과 데이터 흐름(data)의 수로 분류하는 체계. 병렬 하드웨어를 고를 때 판단 기준이 된다.

## 4가지 분류

**SISD** (Single Instruction, Single Data)
전통적인 단일 코어 CPU. 명령어 하나가 데이터 하나를 처리한다. 병렬성 없음.

**MISD** (Multiple Instruction, Single Data)
여러 명령어가 같은 데이터를 처리한다. 실용적인 사례가 거의 없다. 참조용으로만 언급됨.

**SIMD** (Single Instruction, Multiple Data)
하나의 명령어가 여러 데이터에 동시 적용된다. 하나의 제어 유닛이 여러 코어를 지휘하는 구조다. GPU가 대표적이다. 같은 연산을 대량의 데이터에 반복 적용하는 작업(행렬 연산, 이미지 처리, 머신러닝)에 특화돼 있다.

**MIMD** (Multiple Instruction, Multiple Data)
각 처리 자원이 독립된 명령어를 독립된 데이터에 실행한다. 멀티코어 CPU, 컴퓨터 클러스터가 여기에 해당한다. 가장 범용적인 아키텍처로, 가장 널리 사용된다.

> 📷 Figure 3-9 (책 p.54) — SISD / MISD / SIMD / MIMD 4가지 아키텍처 비교 다이어그램

## 동시성과의 관계

SIMD와 MIMD는 병렬 실행을 지원하는 두 가지 방향이다. SIMD는 단순하고 반복적인 대규모 병렬 작업에, MIMD는 복잡하고 다양한 작업의 동시 처리에 적합하다.

이 분류를 알면 어떤 하드웨어가 자신의 문제에 맞는지 판단할 수 있다. GPU(SIMD)와 CPU(MIMD) 중 어느 쪽을 써야 하는지 판단하는 근거가 된다.

참고: multiprocessor.md, cpu-vs-gpu.md
