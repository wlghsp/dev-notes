# Flush (플러시)

Segment를 디스크에 영구 저장(fsync)하고, 그 시점까지의 Translog를 잘라내는 작업.

Refresh가 "검색 가능하게"라면, Flush는 "디스크에 안전하게" 저장하는 단계다.

## 흐름

1. Refresh로 생성된 Segment들이 OS 파일 캐시에 있다.
2. Flush 실행 → 파일 캐시의 Segment를 디스크에 fsync.
3. Flush 완료 시점을 Checkpoint로 기록.
4. Checkpoint 이전의 Translog는 더 이상 필요 없으므로 삭제.

## Translog와의 관계

Flush 전까지는 Translog가 안전망 역할을 한다. 장애 시 Checkpoint 이후의 Translog를 재실행해 복구한다. Flush가 완료되면 해당 구간의 Translog는 필요 없어진다.

## 언제 실행되나

자동으로 실행되는 조건이 두 가지다.

- Translog 크기가 임계값(기본 512MB)을 초과할 때
- 마지막 Flush 이후 일정 시간(기본 30분)이 지났을 때

## Refresh vs Flush 정리

Refresh: buffer → OS 캐시의 Segment. 검색 가능해짐. 빠름.
Flush: OS 캐시의 Segment → 디스크. 영구 저장. 비교적 무거운 작업.

참고: segment.md, refresh.md, translog.md
