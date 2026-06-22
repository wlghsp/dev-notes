# serial vs sequential vs parallel

세 개념 모두 "순서"나 "동시성"과 관련이 있어서 혼용되기 쉽지만, 가리키는 대상이 다르다.

## serial (직렬)

실행 방식을 말한다. 하나의 작업이 완전히 끝난 후 다음 작업이 시작된다. 작업들이 서로 독립적이더라도 그냥 하나씩 순서대로 실행하는 것이다.

작업들이 독립적이라면 serial을 parallel로 바꿀 수 있다.

## sequential (순차)

연산 간의 의존 관계를 말한다. 앞 연산의 결과가 뒤 연산의 입력이 되는 구조다. 순서를 바꿀 수 없다.

sequential computation은 병렬로 바꿀 수 없다. 의존성이 순서를 강제하기 때문이다.

## parallel (병렬)

실행 방식을 말한다. 여러 작업이 물리적으로 동시에 실행된다. 멀티코어가 있어야 가능하고, 작업들이 독립적이어야 한다.

## 핵심 구분

serial과 sequential은 둘 다 "하나씩"처럼 보이지만 성격이 다르다.

- serial은 실행 방식의 선택이다. 독립적인 작업을 그냥 하나씩 돌리는 것
- sequential은 의존성에서 오는 제약이다. 순서를 바꾸고 싶어도 못 바꾸는 것

serial 실행이더라도 작업들이 독립적이면 parallel로 전환할 수 있다. sequential computation이면 아무리 코어가 많아도 parallel로 전환할 수 없다.

parallel은 serial의 반대지만, sequential의 반대는 아니다.

## 한 줄 정리

- serial — 하나씩 실행 (방식의 문제, 바꿀 수 있음)
- sequential — 의존성 때문에 순서가 고정 (구조의 문제, 바꿀 수 없음)
- parallel — 동시에 실행 (멀티코어 필요, 독립적인 작업에만 가능)
