# 알고리즘 챌린지 1~2주차 풀이 정리

프로그래머스 문제 3개 푼 거 정리해봅니다. 순서대로 로또 순위, H-Index, 점 찍기.

## 1. 로또의 최고 순위와 최저 순위

지워진 번호(0)가 있는 로또 용지로 최고/최저 순위를 구하는 문제. 핵심은 0을 "당첨될 수도, 안 될 수도 있는 수"로 보는 것.

- 확정으로 맞은 개수: 0이 아니면서 당첨번호에 있는 수
- 미확정 개수: 0의 개수 (최선의 경우 전부 당첨번호가 됨)

```python
def solution(lottos, win_nums):
    ranks = [6, 6, 5, 4, 3, 2, 1]
    win_set = set(win_nums)
    zero_cnt = lottos.count(0)
    common_cnt = sum(1 for n in lottos if n != 0 and n in win_set)
    return [ranks[common_cnt + zero_cnt], ranks[common_cnt]]
```

최고 순위는 확정 + 미확정을 다 맞은 걸로, 최저 순위는 확정만 맞은 걸로 계산하면 끝. `ranks` 배열에 인덱스로 순위를 매핑해두면 0개 일치와 1개 일치가 둘 다 6등인 경계 처리도 자연스럽게 됩니다.

당첨번호를 매번 배열에서 찾으면 O(N²)인데, set으로 바꾸면 O(N)으로 줄어듭니다.

## 2. H-Index

인용 횟수 배열이 주어질 때 h-index 구하기. 완전탐색으로 h를 하나씩 다 검사하는 것보단, 정렬하고 한 번만 순회하는 게 낫습니다.

```python
def solution(citations):
    n = len(citations)
    i = 0
    citations.sort()
    while i < n:
        if citations[i] >= n - i:
            return n - i
        i += 1
    return 0
```

오름차순 정렬하면 인덱스 i부터 끝까지는 "인용 횟수가 citations[i] 이상인 논문이 n - i편 있다"는 게 보장됩니다. i를 늘려가면서 처음으로 `citations[i] >= n - i`를 만족하는 지점을 찾으면 그게 최대 h입니다. i가 커질수록 n - i는 계속 줄어드니까, 맨 처음 만족하는 지점이 곧 최댓값이 되는 거죠.

h가 꼭 배열에 있는 값일 필요는 없다는 것도 포인트 — `[100, 100, 100]`이면 h는 3이지 100이 아닙니다.

## 3. 점 찍기

원점에서 반지름 d인 원 안쪽에, k 간격의 격자점이 몇 개 있는지 세는 문제. x를 하나 고정하면 조건을 만족하는 y의 범위는

```
(a*k)² + (b*k)² <= d²
```

를 만족하는 b의 최댓값까지인데, 이게 a가 커질수록 단조 감소하니까 이진 탐색으로 찾을 수 있습니다.

```python
def condition(a, b, k, d):
    return (a * k) * (a * k) + (b * k) * (b * k) <= d * d

def binary_search(a, k, d):
    lo, hi = 0, d // k
    result = 0
    while lo <= hi:
        mid = (lo + hi) // 2
        if condition(a, mid, k, d):
            result = mid
            lo = mid + 1
        else:
            hi = mid - 1
    return result

def solution(k, d):
    cnt = 0
    for a in range(d // k + 1):
        cnt += binary_search(a, k, d) + 1
    return cnt
```

처음엔 이중 반복문으로 풀었는데 d가 커지면 O((d/k)²)이라 시간초과가 났습니다. 안쪽 반복을 이진 탐색으로 바꾸니 O((d/k) log(d/k))로 줄어서 통과. sqrt로 직접 y 최댓값을 계산하는 방법도 있는데, 부동소수점 오차가 걱정돼서 정수 비교(제곱 비교)로 이진 탐색하는 쪽을 택했습니다.

---

세 문제 다 "완전탐색을 어떻게 줄일까"가 공통 주제였네요. 로또는 배열 탐색을 집합으로, H-Index는 전체 h 검사를 정렬+순회로, 점 찍기는 이중 반복을 이진 탐색으로 바꾸는 식.
