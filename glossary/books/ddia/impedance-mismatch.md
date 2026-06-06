# impedance-mismatch
참고: relational-model.md, document-model.md

---

객체지향 언어의 데이터 구조와 관계형 DB의 테이블 구조 사이의 불일치다. 전자공학에서 빌려온 용어로, 두 회로의 임피던스가 맞지 않으면 신호 손실이 생기는 것에 비유한다.

## 왜 생기는가

애플리케이션 코드는 객체와 중첩 구조로 데이터를 다룬다. 예를 들어 LinkedIn 프로필 하나는 여러 직장 경력, 학력, 연락처를 가진 단일 객체다. 관계형 DB는 이것을 users, positions, education, contact_info 같은 여러 테이블로 쪼갠다(shredding). 코드에서 객체를 저장하고 불러올 때마다 이 두 표현 사이를 변환해야 한다.

ActiveRecord나 Hibernate 같은 ORM 프레임워크가 이 변환의 boilerplate를 줄여주지만, 두 모델의 차이를 완전히 없애지는 못한다.

## 문서 모델이 해결하는 부분

JSON 표현은 프로필 전체를 하나의 구조로 담을 수 있어 임피던스 불일치가 줄어든다. 중첩된 리스트가 자연스럽게 표현되고, 한 번의 쿼리로 전체 문서를 가져올 수 있다. 다만 many-to-many 관계가 생기면 문서 모델도 이 이점을 잃는다.
