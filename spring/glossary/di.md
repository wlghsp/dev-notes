# 의존관계 주입 (Dependency Injection, DI)

의존관계에 있는 객체들을 런타임 시점에 외부에서 연결해주는 작업이다.
제어의 역전(IoC)이 지향하는 목표를 실제로 구현하는 대표적인 방법이라, IoC와 DI는 거의 같은 맥락에서 함께 등장한다.

먼저 의존관계란 무엇인지부터 짚어야 한다.
어떤 클래스나 모듈이 다른 클래스나 모듈을 사용할 때 그 둘 사이에 의존관계가 형성된다.
예를 들어 MovieFinder가 MovieReader 인터페이스를 사용하면, MovieFinder는 MovieReader에 의존한다.
DI는 이 의존관계에 있는 객체(MovieReader의 실제 구현체)를 MovieFinder가 스스로 찾지 않고, 외부에서 넣어주는 것을 말한다.

## 컴파일타임 의존과 런타임 의존은 다르다

코드만 보면 MovieFinder는 MovieReader라는 인터페이스에 의존하는 것처럼 보인다. 이것이 컴파일타임(코드 시점) 의존관계다.
하지만 실제로 프로그램이 동작할 때는 CsvMovieReader나 XmlMovieReader 같은 구체 클래스의 인스턴스가 필요하다. 이것이 런타임(실행 시점) 의존관계다.
DI는 이 런타임 의존관계를 컨테이너가 대신 연결해주는 작업이다.

## 세 가지 주입 방법

- 생성자 주입(constructor injection): 객체를 생성하는 시점에 생성자를 통해 의존관계를 주입한다.
- 설정자 주입(setter injection): 객체를 생성한 후 설정자(setter) 메소드를 통해 의존관계를 주입한다.
- 메소드 주입(method injection): 메소드를 실행할 때 인자로 의존관계를 주입한다.

스프링에서는 생성자 주입에 `@Autowired`를 선언하는 방식이 가장 권장된다. 스프링 4.3 이상에서는 생성자가 하나뿐이면 `@Autowired`도 생략할 수 있다.

## 왜 필요한가

MovieFinder가 CsvMovieReader를 직접 `new`로 생성하면, MovieFinder는 CsvMovieReader라는 구체 클래스에 묶인다.
DI를 적용하면 MovieFinder는 MovieReader 인터페이스만 알면 되고, 어떤 구현체를 쓸지는 컨테이너의 설정에 달려 있다.
그 결과 테스트할 때는 가짜(mock) 구현체를 주입하고, 운영 환경에서는 실제 구현체를 주입하는 식으로 유연하게 바꿔 쓸 수 있다.

## 관련 개념

- 참고: ioc.md
- 참고: solid.md
