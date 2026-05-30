# Document (문서)

Elasticsearch에서 데이터를 저장하는 기본 단위. JSON 형식으로 표현된다.

관계형 DB로 비유하면 테이블의 row 한 줄에 해당한다.

## 구조

```json
{
  "_index": "products",
  "_id": "1",
  "_source": {
    "name": "노트북",
    "price": 1500000,
    "category": "전자기기"
  }
}
```

- `_index`: 이 문서가 속한 인덱스
- `_id`: 문서 고유 식별자. 지정하지 않으면 Elasticsearch가 자동 생성
- `_source`: 실제 저장된 원본 JSON 데이터

## 특징

문서는 스키마가 유연하다. 같은 인덱스 안에 서로 다른 필드를 가진 문서가 공존할 수 있다. 다만 같은 필드명이면 타입이 일관돼야 한다 — Mapping이 이를 강제한다.

문서는 한번 색인되면 수정이 불가능하다. "수정"은 내부적으로 기존 문서를 삭제 표시하고 새 문서를 색인하는 방식으로 처리된다.

참고: field.md, mapping.md, index.md
