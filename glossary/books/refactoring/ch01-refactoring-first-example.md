# Chapter 1 종합 정리

이 챕터에서 생성된 키워드 파일: refactoring-first-example.md

## 흐름 요약

`statement` 함수(청구서 출력) 하나를 예제로, HTML 출력 추가 + 장르 확장이라는 두 요구사항을 처리하기 쉬운 구조로 바꿔가는 과정. 세부 before/after 코드는 refactoring-first-example.md.

1. **분해** — switch문을 Extract Function으로 뽑고(`amountFor`), 지역 변수를 Replace Temp with Query로 제거해(`playFor`, `usd`) 추출을 쉽게 만든다.
2. **누적 변수 제거** — 반복문 안에서 누적되는 `volumeCredits`는 Split Loop → Slide Statements → Extract Function → Inline Variable, 4개의 작은 스텝으로 나눠 없앤다.
3. **Split Phase** — 계산과 출력을 분리해 중간 데이터 구조(`statementData`)로 연결. 이 덕분에 HTML 버전이 계산 로직 중복 없이 추가된다.
4. **Replace Conditional with Polymorphism** — 장르별 switch문을 `PerformanceCalculator` 클래스 계층으로 대체. 새 장르 추가 시 조건문을 안 건드리고 서브클래스만 추가하면 된다.

## 핵심 규율

매 스텝마다 컴파일-테스트-커밋. 걸음이 작을수록 실수했을 때 원인이 바로 보이고, 테스트가 실패하면 최근 커밋으로 되돌려 더 작은 스텝으로 재시도한다. 리팩토링의 첫 단계는 항상 자가 검증 테스트 준비(4장, building-tests.md).

## 한 줄 결론

좋은 코드의 시험은 "얼마나 쉽게 바꿀 수 있는가"다. 그 바꾸기 쉬움을 지키는 방법이 이 챕터가 보여준 리듬(작은 스텝 + 매 스텝 검증)이다.
