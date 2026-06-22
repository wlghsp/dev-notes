# OOM in Spring (Spring 환경에서 OOM 발생 케이스)

Spring/Spring Boot 애플리케이션에서 OOM이 발생하는 케이스는 대부분 JVM 일반 케이스와 원인은 같지만, Spring의 특성(스레드 풀, JPA, 캐시, 요청 처리 구조)이 트리거가 된다.

## JPA N+1 + 대용량 데이터 로드

연관 엔티티를 Lazy Loading으로 가져오다가 루프 안에서 N번 추가 쿼리가 발생하고, 결과 전체를 List로 메모리에 올릴 때 Heap이 고갈된다. `findAll()`로 수백만 건을 한 번에 조회하는 것도 같은 맥락이다.

Pageable 없이 전체 조회하거나, `@EntityGraph`나 fetch join 없이 컬렉션을 순회하는 코드가 주범이다.

## HttpSession 무한 증가

세션에 데이터를 계속 쌓고 만료 설정을 하지 않으면 사용자가 늘수록 메모리가 선형으로 증가한다. 특히 세션에 대용량 객체(파일, 리스트 등)를 직접 저장할 때 빠르게 고갈된다.

## 로컬 캐시 무한 증가

`@Cacheable`을 사용하면서 캐시 만료(TTL)나 최대 크기(max-size)를 설정하지 않으면 캐시가 계속 쌓인다. 기본 `ConcurrentMapCacheManager`는 만료 정책이 없어서 특히 위험하다.

## ThreadLocal 누수

스레드 풀 환경에서 `ThreadLocal`에 값을 넣고 요청 처리 후 `remove()`를 하지 않으면 스레드가 재사용될 때 이전 값이 그대로 남는다. 값이 클수록, 요청이 많을수록 누수가 쌓인다.

Spring Security의 `SecurityContextHolder`나 MDC 같은 것들이 내부적으로 ThreadLocal을 쓰는데, 커스텀 필터에서 직접 ThreadLocal을 다룰 때 실수하기 쉽다.

## Multipart / InputStream 미닫기

파일 업로드 처리 시 `MultipartFile`이나 `InputStream`을 명시적으로 `close()`하지 않으면 Direct Memory 또는 Heap이 누수된다. try-with-resources 없이 처리하다가 예외가 발생하면 close가 건너뛰어진다.

## 응답 스트리밍 없이 대용량 응답 생성

REST API에서 대용량 데이터를 `List<T>`로 전부 담아 응답하면, 직렬화 전까지 전체 데이터가 Heap에 올라간다. `StreamingResponseBody`나 페이지네이션 없이 전체를 한 번에 내보낼 때 발생한다.
