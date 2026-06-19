# NET (Network Connection Descriptor)

NET는 네트워크 커넥션 디스크립터다. MySQL이 클라이언트/서버 통신에 자체 프로토콜을 OS 프로토콜 위에 얹는데, NET가 그 프로토콜 구현의 핵심이다.

정의는 `include/mysql_com.h`. 클라이언트 라이브러리도 같은 파일을 쓴다. C로 작성되어 메서드가 없고, `NET*`를 인자로 받는 함수들이 별도로 존재한다. 이 함수들은 챕터 4(Client/Server Communication)에서 다룬다.

THD 안에 `NET net` 멤버로 포함된다. 모든 네트워크 통신 함수는 NET를 어떤 형태로든 사용한다.

## 구조의 핵심 멤버

`Vio* vio`가 가장 중요한 멤버다. VIO는 Low-Level Network I/O 추상화로, TCP/IP, Unix 소켓, SSL, Windows named pipe를 동일한 인터페이스로 감싼다. SSL 지원을 위해 원래 만들어졌지만 지금은 크로스플랫폼 이식을 위해서도 쓰인다.

버퍼 관련으로는 `buff`(버퍼 시작), `buff_end`(버퍼 끝), `write_pos`(다음 쓰기 위치), `read_pos`(다음 읽기 위치)가 있다. 패킷 크기 제어는 `max_packet`(현재 패킷 버퍼 크기, `net-buffer-length`에서 시작해 `max-allowed-packet`까지 늘어날 수 있음)과 `max_packet_size`로 한다.

`pkt_nr`은 현재 패킷 시퀀스 번호다. MySQL 비압축 프로토콜에서 패킷 순서 검증에 쓰인다. 순서가 어긋나면 버그 또는 하드웨어 문제를 의미한다.

에러 상태는 `last_errno`(MySQL 에러 코드), `last_error`(에러 메시지 텍스트), `sqlstate`(SQLSTATE 코드, 4.1부터 서버가 직접 채움)로 관리된다.

## 압축 프로토콜

`compress` 플래그가 1이면 데이터 압축을 사용하고, `compress_pkt_nr`로 압축 프로토콜의 별도 시퀀스 번호를 관리한다. 압축 모드에서는 읽기 시 다음 패킷의 일부를 미리 읽을 수 있어 `remain_in_buf`로 초과 읽은 바이트 수를 추적한다.

참고: thd.md
