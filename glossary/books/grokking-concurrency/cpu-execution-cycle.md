# CPU Execution Cycle

CPU가 명령어 하나를 처리하는 반복 과정. Fetch-Decode-Execute-Store의 4단계로 구성된다. CPU 사이클(CPU cycle)이라고도 한다.

## 4단계

1. **Fetch** — CU가 메모리(또는 캐시)에서 다음 명령어를 가져온다. 어떤 명령어를 가져올지는 내부 카운터가 추적한다.
2. **Decode** — 가져온 명령어를 해석한다. 명령어 종류에 따라 어떤 처리 장치로 보낼지 결정한다.
3. **Execute** — ALU로 넘겨 실제 연산을 수행한다.
4. **Store** — 연산 결과를 RAM에 기록한다. 이후 다시 1단계로 돌아간다.

> 📷 Figure 3-3 (책 p.48) — CPU 실행 사이클 다이어그램 (Fetch → Decode → Execute → Store, RAM/Cache/CU/ALU 연결)

## 핵심

프로세서는 명령어가 없어질 때까지 이 사이클을 끝없이 반복한다. 프로그램 실행이란 결국 이 사이클의 연속이다.

캐시가 없다면 Fetch 단계마다 RAM에서 명령어를 가져와야 하므로 CPU가 매번 기다려야 한다. 캐시가 이 병목을 완화한다.

참고: processor.md, cache.md
