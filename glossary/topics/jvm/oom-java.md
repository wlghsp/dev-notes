# OOM in Java (Java OutOfMemoryError 발생 케이스)

JVM이 메모리를 더 이상 할당할 수 없을 때 `java.lang.OutOfMemoryError`를 던진다. 발생 위치에 따라 에러 메시지가 다르다.

## Heap 고갈

가장 흔한 케이스. 객체가 계속 생성되는데 GC가 회수하지 못할 때 발생한다. 메모리 누수가 있거나 대용량 데이터를 한꺼번에 메모리에 올릴 때 일어난다.

에러 메시지: `Java heap space`

## Metaspace 고갈

클래스 메타데이터가 저장되는 영역이 꽉 찰 때 발생한다. 동적으로 클래스를 생성하거나(CGLib, 리플렉션 남용), 클래스로더가 반복적으로 새 클래스를 로드하고 언로드하지 않을 때 일어난다.

에러 메시지: `Metaspace`

## GC Overhead Limit Exceeded

GC가 너무 자주 실행되지만 회수하는 메모리가 거의 없을 때 JVM이 직접 던진다. 전체 시간의 98% 이상을 GC에 쓰면서 2% 미만만 회수되는 상황이 기준이다. Heap이 완전히 고갈되기 전에 먼저 터진다.

에러 메시지: `GC overhead limit exceeded`

## Direct Memory 고갈

NIO의 `ByteBuffer.allocateDirect()`나 Netty 같은 라이브러리가 JVM Heap 바깥의 메모리를 사용할 때 발생한다. `-XX:MaxDirectMemorySize`로 제한되며, 이 한도를 넘으면 터진다.

에러 메시지: `Direct buffer memory`

## 단일 대용량 배열 할당 실패

`new byte[Integer.MAX_VALUE]`처럼 한 번에 너무 큰 배열을 할당하려 할 때 발생한다. Heap에 여유가 있어도 연속된 공간이 없으면 실패한다.

에러 메시지: `Requested array size exceeds VM limit`

## StackOverflowError와의 차이

재귀 호출이 너무 깊어지면 `StackOverflowError`가 발생한다. OOM과 달리 Heap이 아닌 스레드 스택이 한계를 넘는 경우라 엄밀히는 다른 에러지만, 메모리 관련 문제로 함께 다뤄진다.
