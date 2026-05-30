# Inverted Index (역색인)

검색 엔진의 핵심 자료구조. 일반 인덱스가 "문서 → 단어" 방향이라면, 역색인은 **"단어 → 문서 목록"** 방향으로 구성된다.

## 왜 필요한가

"elasticsearch란 무엇인가"라는 문서가 100만 개 있을 때, "elasticsearch"라는 단어가 포함된 문서를 찾으려면 전체를 순회해야 한다. 역색인은 이 탐색을 O(1)에 가깝게 만든다.

## 구조

단어(term)를 키로, 그 단어가 등장하는 문서 ID 목록(Posting List)을 값으로 저장한다.

```
"elasticsearch" → [doc1, doc3, doc7]
"검색"          → [doc1, doc2, doc5]
"엔진"          → [doc2, doc3]
```

Posting List에는 단순 문서 ID뿐 아니라 단어 등장 빈도(TF), 위치 정보(position)도 함께 저장된다. 이 정보가 나중에 relevance score 계산에 쓰인다.

## 색인(Indexing) 과정

문서가 들어오면 Analyzer가 텍스트를 토큰으로 쪼개고, 각 토큰을 역색인에 추가한다.

1. 원문: "Elasticsearch는 강력한 검색 엔진이다"
2. 토큰화: ["elasticsearch", "강력한", "검색", "엔진"]
3. 역색인에 각 토큰 → 현재 문서 ID 등록

## Elasticsearch에서

Elasticsearch는 Lucene 위에 구축되어 있고, 역색인은 Lucene의 Segment 단위로 저장된다. Segment가 생성될 때 역색인이 함께 만들어지고, 한번 작성된 Segment는 불변(immutable)이다.

참고: segment.md, analyzer.md, relevance-score.md
