# Refactoring: A First Example (Chapter 1 예제)

참고: Refactoring 2nd Edition, Chapter 1

## 시작 코드

```javascript
function statement (invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `Statement for ${invoice.customer}\n`;
  const format = new Intl.NumberFormat("en-US",
                        { style: "currency", currency: "USD",
                          minimumFractionDigits: 2 }).format;
  for (let perf of invoice.performances) {
    const play = plays[perf.playID];
    let thisAmount = 0;

    switch (play.type) {
    case "tragedy":
      thisAmount = 40000;
      if (perf.audience > 30) {
        thisAmount += 1000 * (perf.audience - 30);
      }
      break;
    case "comedy":
      thisAmount = 30000;
      if (perf.audience > 20) {
        thisAmount += 10000 + 500 * (perf.audience - 20);
      }
      thisAmount += 300 * perf.audience;
      break;
    default:
        throw new Error(`unknown type: ${play.type}`);
    }

    volumeCredits += Math.max(perf.audience - 30, 0);
    if ("comedy" === play.type) volumeCredits += Math.floor(perf.audience / 5);

    result += `  ${play.name}: ${format(thisAmount/100)} (${perf.audience} seats)\n`;
    totalAmount += thisAmount;
  }
  result += `Amount owed is ${format(totalAmount/100)}\n`;
  result += `You earned ${volumeCredits} credits\n`;
  return result;
}
```

요구사항: HTML 출력 추가, 연극 장르 확장. 지금 구조로는 둘 다 이 함수 안 조건문을 계속 복사/수정해야 한다.

---

## Extract Function — switch문을 함수로

```javascript
function amountFor(aPerformance, play) {
  let result = 0;
  switch (play.type) {
  case "tragedy":
    result = 40000;
    if (aPerformance.audience > 30) {
      result += 1000 * (aPerformance.audience - 30);
    }
    break;
  case "comedy":
    result = 30000;
    if (aPerformance.audience > 20) {
      result += 10000 + 500 * (aPerformance.audience - 20);
    }
    result += 300 * aPerformance.audience;
    break;
  default:
      throw new Error(`unknown type: ${play.type}`);
  }
  return result;
}
```

## Replace Temp with Query — play 변수 제거

`play`는 `perf`에서 다시 계산 가능하니 매개변수로 넘기지 않고 함수로 대체한다.

```javascript
function playFor(aPerformance) {
  return plays[aPerformance.playID];
}
// amountFor(aPerformance) { ... play.type 대신 playFor(aPerformance).type ... }
```

같은 방식으로 `format` → `usd(aNumber)` 함수로, `totalAmount` → `totalAmount()` 함수로 지역 변수를 제거한다. 지역 변수가 줄어들수록 다음 추출이 쉬워진다.

## Split Loop + Slide Statements + Extract Function + Inline Variable — volumeCredits 제거

반복문 안에서 누적되는 변수라 한 번에 못 없앤다. 4개의 작은 스텝으로 쪼갠다.

```javascript
// 1. Split Loop: 적립 포인트 누적을 별도 반복문으로 분리
// 2. Slide Statements: let volumeCredits = 0; 를 그 반복문 옆으로 이동
// 3. Extract Function
function totalVolumeCredits() {
  let result = 0;
  for (let perf of invoice.performances) {
    result += volumeCreditsFor(perf);
  }
  return result;
}
// 4. Inline Variable: volumeCredits 변수를 없애고 totalVolumeCredits() 호출로 대체
```

매 스텝 뒤에 컴파일-테스트-커밋. 걸음이 작을수록 실수의 원인을 바로 찾을 수 있다.

---

## Split Phase — 계산과 출력을 분리

`statement`가 "계산"과 "텍스트로 조립" 두 가지 일을 함께 하고 있다. HTML 버전을 추가하려면 계산 로직을 재사용해야 하므로 중간 데이터 구조로 분리한다.

```javascript
function statement (invoice, plays) {
  return renderPlainText(createStatementData(invoice, plays));
}

function createStatementData(invoice, plays) {
  const result = {};
  result.customer = invoice.customer;
  result.performances = invoice.performances.map(enrichPerformance);
  result.totalAmount = totalAmount(result);
  result.totalVolumeCredits = totalVolumeCredits(result);
  return result;

  function enrichPerformance(aPerformance) {
    const result = Object.assign({}, aPerformance);
    result.play = playFor(result);
    result.amount = amountFor(result);
    result.volumeCredits = volumeCreditsFor(result);
    return result;
  }
}
```

`renderPlainText(data)`는 이제 순수 데이터만 보고 텍스트를 조립한다. 같은 `createStatementData`를 재사용해서 `renderHtml(data)`만 새로 짜면 HTML 버전이 계산 로직 중복 없이 나온다.

---

## Replace Conditional with Polymorphism — 장르 확장 대비

`amountFor`/`volumeCreditsFor` 안의 switch문을 다형성으로 바꾼다. 준비 단계가 필요하다.

**1. Move Function으로 계산 로직을 클래스에 담기, Replace Type Code with Subclasses + Replace Constructor with Factory Function으로 서브클래스 구조 준비:**

```javascript
function createPerformanceCalculator(aPerformance, aPlay) {
  switch (aPlay.type) {
    case "tragedy": return new TragedyCalculator(aPerformance, aPlay);
    case "comedy" : return new ComedyCalculator(aPerformance, aPlay);
    default:
      throw new Error(`unknown type: ${aPlay.type}`);
  }
}

class PerformanceCalculator {
  constructor(aPerformance, aPlay) {
    this.performance = aPerformance;
    this.play = aPlay;
  }
  get amount() {
    throw new Error('subclass responsibility'); // 결코 호출되면 안 되는 표지석
  }
  get volumeCredits() {
    return Math.max(this.performance.audience - 30, 0);
  }
}
```

**2. Replace Conditional with Polymorphism — 각 case를 서브클래스 오버라이드로:**

```javascript
class TragedyCalculator extends PerformanceCalculator {
  get amount() {
    let result = 40000;
    if (this.performance.audience > 30) {
      result += 1000 * (this.performance.audience - 30);
    }
    return result;
  }
}

class ComedyCalculator extends PerformanceCalculator {
  get amount() {
    let result = 30000;
    if (this.performance.audience > 20) {
      result += 10000 + 500 * (this.performance.audience - 20);
    }
    result += 300 * this.performance.audience;
    return result;
  }
  get volumeCredits() {
    return super.volumeCredits + Math.floor(this.performance.audience / 5);
  }
}
```

새 장르가 추가되면 조건문을 안 건드리고 서브클래스 하나만 추가하면 된다.

---

## 이 챕터의 결론

작은 스텝 + 매 스텝 컴파일-테스트-커밋이 기법 목록보다 중요한 교훈. 좋은 코드의 시험은 "바꾸기 얼마나 쉬운가"다.

## 관련 기법

extract-function.md, replace-temp-with-query.md, split-loop.md, slide-statements.md, split-phase.md, move-function.md, replace-type-code-with-subclasses.md, replace-constructor-with-factory-function.md, replace-conditional-with-polymorphism.md
refactoring-and-performance.md — 반복문을 두 번 도는 것에 대한 성능 우려 관련.
