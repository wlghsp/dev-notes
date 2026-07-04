# memory_order_consume은 왜 존재하는가

원문: "The Purpose of memory_order_consume in C++11" — Preshing on Programming (2014)

## 핵심

`memory_order_consume`은 acquire-and-release-semantics.md의 acquire보다 더 제한적이지만 더 효율적인 동기화 방법이다. consume은 항상 acquire로 안전하게 대체할 수 있지만 반대는 불가능하다. 대신 PowerPC나 ARM 같은 약한 메모리 모델의 프로세서에서, acquire가 필요로 하는 메모리 배리어 명령어를 생략할 수 있다는 이점이 있다.

이 이점은 소스 코드에 명시적인 **의존성 체인(dependency chain)**이 있을 때만 성립한다. 예를 들어 `g = Guard.load(memory_order_consume); p = *g;`처럼 읽어온 값을 포인터 역참조로 바로 사용하면, 프로세서가 배리어 없이도 순서를 지켜준다.

다만 실제로는 GCC/Clang 모두 consume을 그냥 acquire로 취급해버려서 이 최적화를 살리지 못하고, x86-64에서는 애초에 이점이 없기 때문에 실무에서 거의 쓰이지 않는다.

---

참고: acquire-and-release-semantics.md
