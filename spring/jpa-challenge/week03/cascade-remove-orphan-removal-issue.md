# cascade REMOVE + orphanRemoval 조합에서 실제로 겪은 문제

B-4 실험(`Board`/`Comment`) 중 재현된 현상과 원인. 실제로 여러 버전(Hibernate 6.2.13 / 6.4.10 / 6.5.3)에서 직접 재현하고 확인함.

## 증상

```java
@OneToMany(mappedBy = "board", cascade = CascadeType.REMOVE, orphanRemoval = true)
private List<Comment> comments = new ArrayList<>();
```

이 매핑에서 `board.getComments().remove(comment)`로 컬렉션에서만 자식을 제거하면, orphanRemoval이 트리거되지 않았다. `em.flush()`를 해도 DELETE는커녕 UPDATE도 나가지 않았다.

- `remove()`, `clear()`, index 기반 제거, Iterator.remove() — 방식을 바꿔도 동일하게 실패
- 완전히 새로운 EntityManager/트랜잭션에서 시도해도 동일
- `@DataJpaTest`의 트랜잭션 래핑과는 무관 (독립 트랜잭션에서도 재현)
- `List` → `Set`으로 컬렉션 타입을 바꿔도 동일
- `equals()`/`hashCode()` 유무와도 무관

반면 `em.remove(board)`로 **부모 자체를 삭제**하는 cascade REMOVE는 이 매핑 그대로 정상 동작했다(자식도 함께 DELETE됨).

## 원인 확인 과정

1. Hibernate 버전 문제로 의심 → `pom.xml`에서 6.5.3.Final → 6.4.10.Final → 6.2.13.Final로 낮춰서 테스트 → **세 버전 모두 동일하게 실패**. 버전 문제 아님.
2. `Board`/`Comment`와 무관한 완전히 새로운 엔티티 쌍(`cascade = CascadeType.ALL, orphanRemoval = true`)으로 같은 시나리오를 실행 → **정상 동작**(자식이 실제로 삭제됨).
3. 두 케이스의 유일한 차이는 `cascade` 값(`REMOVE` 단독 vs `ALL`). `Board.comments`의 cascade를 `REMOVE`에서 `ALL`로 바꾸자 → **정상 동작**으로 확인.

## 결론

`orphanRemoval = true`를 쓸 때 `cascade`를 `REMOVE` 단독으로 좁혀서 쓰면, 컬렉션에서 자식을 제거하는 방식의 orphan 삭제가 트리거되지 않는 현상을 실제로 재현했다. `cascade = CascadeType.ALL`(또는 최소한 PERSIST를 포함하는 조합)로 바꾸면 정상 동작한다.

이 프로젝트에서는 `Board.comments`의 cascade를 `ALL`로 변경해서 해결함. 그에 따라 "cascade PERSIST가 없으면 저장 안 된다"는 전제로 짰던 테스트(`cascade_persist_saves_children_added_to_collection`)도 "PERSIST가 있으면(ALL 포함) 함께 저장된다"로 검증 내용을 뒤집었다.

## 주의

이 현상의 정확한 내부 메커니즘(왜 REMOVE 단독일 때 orphan 판정이 스킵되는지)은 Hibernate 소스 레벨까지 확인하지 못했다. 재현 여부와 해결책(cascade ALL로 변경)만 확인된 사실이고, "왜 그런가"는 미확인 상태로 남겨둔다.
