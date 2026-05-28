# Mapping (매핑)

인덱스 안에 저장되는 Document의 필드 구조와 타입을 정의하는 스키마. 관계형 DB의 테이블 스키마에 해당한다.

## 역할

Elasticsearch는 JSON 문서를 받아 역색인을 만들 때, 각 필드를 어떻게 처리할지 Mapping을 보고 결정한다.

- `text` 필드 → Analyzer를 거쳐 토큰으로 분리 후 역색인에 저장
- `keyword` 필드 → 분석 없이 원문 그대로 저장
- `date` 필드 → 날짜 포맷 파싱 후 숫자로 변환해 저장

## 동적 매핑 vs 명시적 매핑

동적 매핑(Dynamic Mapping): 문서를 색인할 때 필드가 없으면 Elasticsearch가 타입을 추론해 자동 추가한다. 빠르게 시작할 수 있지만 의도치 않은 타입으로 매핑될 수 있다.

명시적 매핑(Explicit Mapping): 인덱스 생성 시 필드 타입을 직접 지정한다. 프로덕션에서는 명시적 매핑을 권장한다.

```json
PUT /products
{
  "mappings": {
    "properties": {
      "name": { "type": "text" },
      "category": { "type": "keyword" },
      "price": { "type": "integer" },
      "created_at": { "type": "date" }
    }
  }
}
```

## 주의점

Mapping은 한번 정해진 필드 타입을 변경할 수 없다. 타입을 바꾸려면 새 인덱스를 만들고 데이터를 재색인(reindex)해야 한다.

참고: field.md, document.md, index.md, analyzer.md
