# Lazy Deletion (지연 삭제)

삭제 요청이 들어왔을 때 즉시 제거하지 않고, **삭제됐다는 표시(tombstone)만 남기고 나중에 처리**하는 전략.

힙에서는 특히 "특정 원소를 중간에 삭제해야 할 때" 문제가 생긴다.
힙은 최상위 원소만 빠르게 꺼낼 수 있고, 내부 임의 위치 삭제는 O(n) 탐색이 필요하다.
이 비용을 피하기 위해 lazy deletion을 쓴다.

## 동작 방식

1. 삭제 요청이 들어오면 실제로 제거하지 않는다.
2. 해당 원소를 "무효(invalidated)" 상태로 표시해둔다.
3. 나중에 힙에서 pop할 때, 꺼낸 원소가 무효라면 버리고 다음 원소를 꺼낸다.
4. 유효한 원소가 나올 때까지 반복한다.

결과적으로 "진짜 최솟값"은 항상 유효한 원소 중 가장 앞에 있는 것이 된다.

## 코드 예시 (Python)

```python
import heapq

class LazyHeap:
    def __init__(self):
        self._heap = []
        self._invalid = set()  # 삭제된 원소를 추적

    def push(self, val):
        heapq.heappush(self._heap, val)

    def delete(self, val):
        # 실제로 힙에서 꺼내지 않고 invalid로 표시만 한다
        self._invalid.add(val)

    def pop(self):
        # 유효한 원소가 나올 때까지 건너뛴다
        while self._heap and self._heap[0] in self._invalid:
            removed = heapq.heappop(self._heap)
            self._invalid.discard(removed)
        if not self._heap:
            raise IndexError("pop from empty heap")
        return heapq.heappop(self._heap)

    def peek(self):
        while self._heap and self._heap[0] in self._invalid:
            removed = heapq.heappop(self._heap)
            self._invalid.discard(removed)
        return self._heap[0] if self._heap else None
```

사용 예시:

```python
h = LazyHeap()
h.push(3)
h.push(1)
h.push(5)
h.push(2)

h.delete(1)  # 1을 삭제 표시

print(h.pop())  # 2 (1은 건너뜀)
print(h.pop())  # 3
```

## 주의점

- `_invalid` 셋이 계속 쌓이면 메모리가 낭비된다. pop할 때 정리하거나, 별도 gc 로직이 필요하다.
- 같은 값이 힙에 여러 개 있으면 오동작할 수 있다. 값 대신 (val, id) 쌍으로 추적하는 게 더 안전하다.
- 실제로는 값보다 **작업 ID나 태스크 객체**를 기준으로 invalid를 판단하는 경우가 많다.

## 어디서 쓰이나

다익스트라 알고리즘에서 방문한 노드를 힙에서 즉시 제거하는 대신,
꺼낼 때 이미 방문된 노드이면 건너뛰는 방식이 대표적인 lazy deletion이다.

우선순위 큐 기반 스케줄러, 타이머 구현에서도 동일한 패턴이 자주 나온다.
