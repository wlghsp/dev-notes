# SOLID 원칙

깔끔한 객체지향 설계를 위해 적용할 수 있는 다섯 가지 소프트웨어 설계 원칙이다.
함수와 데이터 구조를 클래스로 어떻게 배치할지, 그리고 그 클래스들을 서로 어떻게 결합할지에 대한 지침을 담고 있다.
목적은 모듈과 컴포넌트 내부 구조를 이해하기 쉽게 만들고, 변경에 유연하게 대응하도록 만드는 것이다.

- Single responsibility principle (단일 책임 원칙): 하나의 클래스는 하나의 책임만 가져야 한다.
- Open/closed principle (개방 폐쇄 원칙): 소프트웨어 개체는 확장에는 열려 있어야 하고, 변경에는 닫혀 있어야 한다.
- Liskov substitution principle (리스코프 치환 원칙): 자식 타입은 언제나 부모 타입으로 치환할 수 있어야 한다.
- Interface segregation principle (인터페이스 분리 원칙): 클라이언트는 자신이 사용하지 않는 인터페이스에 의존하면 안 된다.
- Dependency inversion principle (의존성 역전 원칙): 상위 정책은 하위 정책에 의존하면 안 된다. 하위 정책이 상위 정책에 정의된 추상 타입에 의존해야 한다.

## 개방 폐쇄 원칙과 의존성 역전 원칙의 관계

이 다섯 가지 중 스프링의 제어의 역전(IoC)과 가장 밀접하게 맞닿아 있는 것이 개방 폐쇄 원칙과 의존성 역전 원칙이다.

MovieFinder(상위 정책)는 MovieReader 인터페이스에 의존하고, CsvMovieReader나 XmlMovieReader(하위 정책)가 그 인터페이스를 실체화한다.
새로운 형식의 리더가 필요해지면 MovieReader를 구현하는 클래스를 하나 더 추가하면 된다. MovieFinder의 코드는 손댈 필요가 없다.
이것이 확장에는 열려 있고 변경에는 닫혀 있다는 개방 폐쇄 원칙이다.

동시에 상위 정책인 MovieFinder가 하위 정책(구체 클래스)에 직접 의존하지 않고, 하위 정책이 상위 정책에서 정의한 추상 타입(MovieReader 인터페이스)에 의존한다.
이것이 의존성 역전 원칙이다.

## 관련 개념

- 참고: ioc.md
- 참고: di.md
- 참고: cohesion.md
- 참고: loose-coupling.md
