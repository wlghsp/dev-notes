# Structured Event

요청 하나가 서비스와 상호작용하는 동안 일어난 모든 일을 기록한, key-value 쌍으로 이루어진 레코드.

요청이 서비스에 들어오는 순간 빈 map을 하나 만들고, 요청이 처리되는 동안 알게 되는 값(파라미터, 실행 시간, 호출한 원격 서비스, 에러 여부 등)을 계속 그 map에 채워 넣는다. 요청이 끝나거나 실패해서 나갈 때, 그 map 전체를 하나의 레코드로 기록한다. 이게 structured event다.

## 왜 "arbitrarily wide"인가

Structured event는 필드 개수에 제한을 두지 않는다. 성숙하게 계측된 시스템에서는 이벤트 하나가 300~400개의 필드를 가지기도 한다. 필드 수를 미리 제한하면, 나중에 어떤 조합으로 문제를 찾아야 할지 예측할 수 없는 상황에서 필요한 정보가 애초에 기록되지 않은 상태가 된다. 그래서 observability 시스템은 이벤트를 넓게(wide) 기록하는 것을 기본 전제로 삼는다.

## 왜 Metric으로는 안 되는가

Metric은 미리 정한 기간 동안 값을 집계한 숫자 하나다. 예를 들어 `page_load_time`이 5초 동안의 평균이라면, 그 5초 사이에 있었던 개별 요청 각각의 사정은 이미 사라진 뒤다. 특정 요청이 왜 느렸는지, 어떤 필드가 그 요청과 공통점을 가지는지는 집계된 숫자만 보고는 알 수 없다. 반면 structured event는 요청 하나하나를 개별 레코드로 남기기 때문에, 나중에 어떤 축으로든 다시 잘라서 볼 수 있다.

## 왜 기존 Log로는 부족한가

기존 로그(unstructured log)는 사람이 읽기 위해 설계된 텍스트 블록이다. 하나의 요청을 처리하는 동안 여러 줄의 로그가 흩어져서 남고, 그 여러 줄을 다시 하나의 요청으로 묶어주는 장치가 없는 경우가 많다. Structured log(JSON 형식 등으로 기계가 파싱하기 쉽게 정리한 로그)로 바꾸는 것만으로는 부족하고, 한 unit of work에 해당하는 모든 로그 줄을 하나의 이벤트로 합쳐야 structured event가 된다.

## Cardinality, Dimensionality와의 관계

Structured event가 유용하려면 담긴 필드들이 cardinality와 dimensionality를 모두 감당할 수 있어야 한다. observability.md에서 다룬 개념과 이어지는 지점으로, event 하나하나가 넓고(dimensionality) 그 안의 필드 값이 세밀할수록(cardinality) 나중에 정확한 원인을 좁혀나갈 수 있다.

참고: observability.md, signal.md

## Recap

Structured event는 요청 하나가 처리되는 동안 알게 된 모든 정보를 key-value로 채워, 요청이 끝날 때 하나의 레코드로 남기는 것이다. Metric은 미리 집계해서 개별 사정을 잃고, 기존 Log는 여러 줄로 흩어져서 한 요청 단위로 묶기 어렵다는 두 한계를 동시에 해결한다. 필드 수를 미리 제한하지 않는(arbitrarily wide) 것이 핵심 전제다.
