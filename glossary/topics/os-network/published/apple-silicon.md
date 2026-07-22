# Apple Silicon

ARM 아키텍처 기반으로 Apple이 직접 설계한 칩 시리즈(M1/M2/M3). 조용하고 빠른 이유는 칩 설계 철학 자체가 다르기 때문이다.

## Intel/AMD와 뭐가 다른가

Intel/AMD는 x86 아키텍처 기반으로, 수십 년간 하위 호환성을 유지하면서 점점 복잡해진 설계다. 성능을 올리려면 클럭을 높이고 코어를 더 넣는 방식이었는데, 클럭이 올라가면 발열이 생기고 팬이 돌아야 한다.

Apple Silicon은 iPhone에서 쌓은 저전력 설계 노하우를 그대로 가져왔다.

## 조용하고 빠른 이유

**전력 효율**

ARM 명령어 셋 자체가 단순하다(RISC). 같은 연산을 더 적은 전력으로 처리할 수 있다. 전력을 덜 쓰니 발열이 적고, 발열이 적으니 팬이 필요 없다.

**Unified Memory Architecture**

CPU, GPU, Neural Engine이 메모리를 공유한다. Intel Mac은 CPU 메모리 따로, GPU VRAM 따로 있어서 데이터를 복사해서 주고받아야 한다. Apple Silicon은 복사 없이 바로 접근하니 빠르고 전력도 아낀다.

**SoC (System on a Chip)**

일반 PC는 CPU, GPU, 메모리 컨트롤러가 각각 별도 칩이라 데이터가 버스를 타고 오간다. M 시리즈는 이걸 하나의 칩에 통합해서 이동 거리 자체가 짧다.

## OS 관점에서 다른 점

기본 페이지 크기가 16KB다. x86은 4KB가 기본인데, 페이지가 크면 TLB 엔트리 하나로 더 넓은 메모리를 커버할 수 있어서 TLB 미스가 줄어든다. 참고: tlb.md

참고: tlb.md, hypervisor.md
