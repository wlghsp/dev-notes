# Analyzer (분석기)

텍스트를 역색인에 저장 가능한 토큰으로 변환하는 파이프라인. 색인 시와 검색 시 모두 동작한다.

## 구성 요소

Analyzer는 3단계로 구성된다.

1. Character Filter: 원문 텍스트를 토크나이저에 넘기기 전에 전처리한다. HTML 태그 제거, 특수문자 치환 등.

2. Tokenizer: 텍스트를 토큰으로 쪼갠다. 공백 기준, 형태소 기준 등 다양한 방식이 있다. Analyzer 하나에 Tokenizer는 반드시 하나.

3. Token Filter: 토큰을 후처리한다. 소문자 변환, 불용어(stopword) 제거, 동의어 처리, 어근 추출(stemming) 등.

## 동작 예시

입력: "Elasticsearch는 강력한 검색 엔진이다"

- Tokenizer(공백 기준): ["Elasticsearch는", "강력한", "검색", "엔진이다"]
- Token Filter(소문자): ["elasticsearch는", "강력한", "검색", "엔진이다"]
- Token Filter(어미 제거, 한국어 형태소): ["elasticsearch", "강력하다", "검색", "엔진"]

최종 역색인에 저장되는 토큰: ["elasticsearch", "강력하다", "검색", "엔진"]

## 색인 시 vs 검색 시

색인할 때와 검색 쿼리를 분석할 때 같은 Analyzer를 써야 한다. 색인 시 "running"을 "run"으로 저장했는데 검색 시 "running" 그대로 찾으면 매칭이 안 된다.

`match` 쿼리는 기본적으로 해당 필드의 Analyzer를 검색 쿼리에도 적용한다.

## 내장 Analyzer

- `standard`: 공백/구두점 기준 분리, 소문자 변환. 영어에 기본.
- `simple`: 소문자 알파벳만 남기고 나머지 제거.
- `whitespace`: 공백만 기준으로 분리, 그 외 변환 없음.
- `korean` (nori): 한국어 형태소 분석기. 별도 플러그인.

참고: inverted-index.md, field.md, mapping.md, full-text-search.md
