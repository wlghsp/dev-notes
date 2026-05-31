# segfault

Segmentation Fault. 프로세스가 접근 권한이 없는 메모리 주소에 접근했을 때 OS가 그 프로세스를 강제 종료하는 것.

## 발생 흐름

프로세스가 가상 주소로 메모리에 접근하면 CPU 안의 MMU가 가상→물리 변환을 시도한다. 이때 MMU가 "이 주소는 이 프로세스에게 허가된 영역이 아님"을 감지하면 CPU가 인터럽트를 발생시킨다. OS는 해당 프로세스에 SIGSEGV 시그널을 보내고 프로세스가 종료된다.

## 자주 발생하는 케이스

- NULL 포인터 역참조 — `ptr`이 NULL인데 `*ptr`로 읽으려 할 때
- use-after-free — `free()` 한 메모리를 다시 접근할 때
- 스택 오버플로우 — 재귀가 너무 깊어져 허가된 스택 영역을 넘어설 때
- 커널 공간 접근 — 유저 모드에서 커널 주소를 직접 읽으려 할 때
- 읽기 전용 영역에 쓰기 — 텍스트 세그먼트(실행 코드)에 덮어쓰려 할 때

## Java에서 segfault가 거의 안 나는 이유

JVM이 배열 인덱스와 null 참조를 미리 검사해서 `ArrayIndexOutOfBoundsException`, `NullPointerException`을 먼저 던지기 때문이다. OS까지 내려가기 전에 JVM이 가로챈다. C/C++는 이런 검사가 없어서 그대로 segfault가 난다.

## 관련 개념

- 가상→물리 변환 메커니즘은 virtual-memory.md 참고
- 커널 공간과 유저 공간 구분은 process.md 참고
