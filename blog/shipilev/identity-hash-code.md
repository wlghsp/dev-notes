# hashCode()는 실제로 어디서 오는가

원문: "JVM Anatomy Quark #26: Identity Hash Code" — Aleksey Shipilëv (shipilev.net)

## Why

`hashCode()`를 오버라이드하지 않은 객체도 항상 정수 하나를 반환한다. 이 값은 어디서 오고, 언제 계산되고, 호출할 때마다 같은 값을 어떻게 보장하는지 다룬다.

## 저장 위치

`System.identityHashCode()`가 반환하는 identity hash code는 필요할 때마다 새로 계산하는 게 아니라, 객체 헤더의 Mark Word 안에 저장된다(참고: java-objects-inside-out.md). `identityHashCode()`를 처음 호출하기 전에는 Mark Word가 빈 상태지만, 호출 후에는 계산된 해시 값이 Mark Word 안에 박혀서, 이후에는 그 값을 그대로 재사용한다.

hashCode의 계약(contract)은 같은 객체에 대해 여러 번 호출해도 항상 같은 정수를 반환해야 한다는 것인데, 헤더에 값을 저장해두는 방식이 바로 이 계약을 지키는 방법이다.

## 값을 생성하는 여섯 가지 방식

HotSpot은 `-XX:hashCode` 옵션으로 선택 가능한 여섯 가지 해시 코드 생성 방식을 제공한다.

0. `os::random()` 기반 PRNG
1. 객체 주소 기반, stop-the-world 필요
2. 항상 고정값(1)
3. 전역 카운터
4. 객체의 실제 메모리 주소를 그대로 사용
5. 스레드별 PRNG (현재 기본값)

## 왜 객체 주소를 그대로 쓰지 않는가

객체 주소를 해시로 쓰면 직관적일 것 같지만 두 가지 문제가 있다. 첫째, bump-pointer 방식의 순차 할당 때문에 주소값 자체의 엔트로피가 매우 낮다. 둘째, GC가 객체를 이동시키면 주소가 바뀌는데, hashCode는 절대 바뀌면 안 된다는 계약을 위반하게 된다. 그래서 주소 기반 생성은 값이 한 번 정해지면 더 이상 움직이지 않도록 별도로 관리해야 한다.

## 성능 비교

이미 계산되어 캐시된 값을 읽는 경우(warm)는 어떤 생성 방식을 쓰든 약 5나노초로 동일하다. 처음 계산하는 경우(cold)는 방식별로 차이가 크다.

- `os::random()` PRNG: 약 400ns (원자적 연산 경합 때문에 느림)
- 주소 기반: 약 86ns
- 전역 카운터: 약 124ns
- 스레드별 PRNG(기본값): 약 90ns

## 참고

과거 Javadoc에는 "내부 주소를 변환하여" 해시를 만든다는 문구가 있었지만, 실제로는 identity hash code 계산에 주소 연산이 전혀 관여하지 않기 때문에 이 문구는 삭제되었다.

---

참고: java-objects-inside-out.md
