# 4. Connection Management
## TCP connection
- 커넥션이 맺어지면 메시지의 손실, 손상되지 않음 + 순서가 바뀌지 않고 안전하게 전달

### 신뢰할 수 있는 데이터 전송 통로 (Reliable Data Pipes)
- HTTP 커넥션은 몇 규칙을 제외하고는 거의 TCP 커넥션임

### TCP Stream 은 segment 로 나뉘어 IP packet 을 통해 전송
- IP packet (or IP datagram) 의 작은 단위로 전송  
![image](https://user-images.githubusercontent.com/10507662/109794524-48307180-7c59-11eb-89cf-512da0b52be3.png)
- HTTPS 는 암호화 계층 추가
- TCP 는 segment 단위로 나누고
- segment 를 IP packet 이라는 봉투에 담아서 전달
  - IP packet header (보통 20 byte)
    - IP address, 크기, 기타 flags
  - TCP segment header (보통 20 byte)
    - TCP port number, TCP control flags, 데이터의 순서와 무결성을 검사하기 위한 숫자 값
  - A chunk of TCP data (0 혹은 그 이상 byte)  

![image](https://user-images.githubusercontent.com/10507662/109795148-f1776780-7c59-11eb-8478-e87558c87e7b.png)

### TCP 커넥션 유지
- 여러 개의 TCP 커넥션을 가지며, port number 를 통해 유지
- `<source-IP-address, source-port, destination-IP-address, destination-port>`
  - 이 4개로 커넥션 식별

## TCP 성능 고려
- HTTP 는 TCP 성능에 영향을 받음

### HTTP Transaction delay
- 딜레이의 원인이 될 수 있는 요소

1. IP address + port number 찾기, 기존 데이터가 없으면 DNS resolution infrastructure 로 변환
2. Client 가 TCP connection request 를 보내고 Server 의 응답 대기
3. connection 이 맺어지면 TCP pipe 를 통해 데이터 전송 및 서버에서 처리
4. 웹 서버가 HTTP 응답을 보내는 것 자체

### 성능 관련 중요 요소
#### 1) TCP connection handshake delay
![image](https://user-images.githubusercontent.com/10507662/109797026-3e5c3d80-7c5c-11eb-99da-445dee598c78.png)
- c의 ACK 을 보내면서 데이터 같이 보낼 수 있음
- 지연 요소
  - HTTP Transaction 이 아주 큰 데이터를 주고 받지 않는, 평범한 경우에는 **SYN/SYN+ACK handshake** 과정에 큰 지연 발생
- 크기가 작은 HTTP Transaction 은 50% 이상의 시간을 TCP 를 구성하는데 사용

#### 2) Delayed Acknowledgments (확인응답 지연)
- 인터넷 자체가 패킷 전송을 완벽히 보장하지 않기 때문에 (라우터가 과부하 걸렸을 때 패킷을 마음대고 파기할 수 있음)
- TCP 는 자체적인 확인 체계를 가짐
  - sequence number, data-integrity checksum
  - `receiver` 가 segment 를 온전히 받으면 작은 **acknowledgement** packet 을 `sender` 에게 보냄
  - `sender` 가 특정 시간 안에 **acknowledgment** 를 받지 못하면 다시 전송 (파기되었거나, 오류로 인지)
  - **acknowledgement** 는 크기가 작기 때문에, 같은 방향으로 나가는 데이터 패킷에 acknowledgement 를 piggyback(편승, 업기) 시킴, 네트워크를 효율적으로 사용
- acknowledgement 가 piggyback 되는 경우를 늘리기 위해 **delayed acknowledgment** 를 구현
- **delayed acknowledgment** 는 acknowledgment 를 0.1~0.2 초 간 버퍼에 두고 데이터 패킷을 찾음
  - 못 찾은 경우 acknowledgment 를 별도 패킷에 만들어 전송
- request/response 만 존재하는 HTTP 의 경우 piggyback 할 기회가 적음
  - **delayed acknowledgment** 알고리즘에 의한 지연이 자주 발생

#### 3) TCP slow start
- TCP 의 데이터 전송 속도는 TCP connection 이 만들어진지 **얼마나 지났는지**에 따라 다를 수 있음
- TCP connection 은 시간이 지나면서 자체적으로 튜닝
  - 처음에는 최대 속도 제한 -> 데이터가 성공적으로 전송됨에 따라 속도 제한을 늘림
  - 이 조율 과정 = TCP slow start
  - 인터넷의 급작스러운 부하와 혼잡을 방지하는데 쓰임
- 한 번에 전송할 수 있는 패킷의 수를 제한
  - 패킷이 성공적으로 전달되는 각 시점에 sender 가 추가로 2개의 패킷을 더 전송할 수 있는 권한을 얻음
  - acknowledgment 를 받으면 2개의 패킷을 보낼 수 있고, 그거에 대한 acknowledgment 를 받으면 총 4개를 보낼 수 있음
  - 이 과정을 `opening the congestion window` 라고 함

#### 4) Nagle 알고리즘과 TCP_NODELAY
- (네트워크의 효율을 위하여) 패킷을 전송하기 전에 많은 양의 TCP 데이터를 한 개의 덩어리로 합침
- segment 가 최대 크기가 되지 않으면 전송하지 않음
  - LAN: 1500 byte, Internet: thousands of bytes
  - 모든 패킷이 acknowledgment 를 받은 경우에는 최대 크기보다 작은 패킷의 전송을 허락
  - 다른 패킷들이 아직 전송 중이면, 데이터는 버퍼에 저장됨
  - 전송 후 acknowledgment 를 받았거나 전송하기 충분한 패킷이 쌓였을 때 버퍼에 있던 데이터 전송
- 문제
  - 크기가 작은 HTTP 메시지는 패킷을 채우지 못해, 추가적인 데이터를 기다리며 지연될 수 있음
  - 바로 위의 delayed acknowledgment 알고리즘과 함께 쓰일 경우 형편없이 동작
    - 이거는 acknowledgment 가 올 때까지 전송을 멈추고, delayed acknowledgment 알고리즘은 acknowledgment 를 0.1~0.2초 지연
- HTTP 애플리케이션은 성능 향상을 위해 HTTP 스택에 `TCP_NODELAY` 파라미터를 설정하여 Nagle 을 비활성화하기도 함
  - nginx 의 경우 `tcp_nodelay = on;`

#### 5) TIME_WAIT 의 누적과 port exhaustion
- TCP endpoint 에서 TCP connection 을 끊으면, endpoint 에서는 connection 의 IP address + port number 를 메모리의 `control block` 에 기록
  - 같은 address 와 port number 를 사용하는 새로운 TCP connection 이 일정 시간 동안에는 생성되지 않게 하기 위함
  - 보통 segment 최대 lifetime 의 2배 (2MSL, 보통은 2분 - 최근에는 더 짧아졌다는 주석 있음) 동안 유지
  - 이전 connection 과 관련된 패킷이 같은 정보를 가진 새로운 connection 에 삽입되는 문제를 방지
- 요즘에는 빠른 라우터들 덕분에 connection close 후 중복되는 패킷이 생기는 경우가 거의 없긴 하지만
  - **이전 패킷이 같은 정보를 가진 새로운 connection 에 들어온 경우, 패킷은 중복되고 TCP 데이터 충돌할 수 있음**
- 보통은 문제가 안되지만, 성능 테스트 시 문제될 수 있음
- connection 을 너무 많이 맺거나, 대기 상태로 있는 control block 이 너무 많아지는 상황도 주의

## HTTP connection handling
### 잘못 이해하는 Connection 헤더
- HTTP 는 Client - Server 사이에 Proxy, Cache server 등의 중개 서버를 허용
- HTTP 메시지는 하나하나를 거치며 전달됨
- HTTP Connection 헤더는 connection token 을 `,` 로 구분하여 가짐, <ins>다른 커넥션에 전달되지 않음</ins>
  - HTTP 헤더 필드명 - 이 connection 에만 해당되는 헤더를 나열
  - Arbitrary token 값 - connection 에 대한 비표준 옵션
  - close 값 - connection 이 작업이 완료되면 종료되어야 함을 의미
- Connection 헤더에 있는 모든 헤더 필드는 메시지를 다른 곳으로 전달하는 시점에 삭제되어야 함
- **protecting the header**: Connection 헤더는 hop-by-hop (server-to-server) 헤더 명을 기술

1. HTTP 애플리케이션이 Connection 헤더와 메시지를 전달 받음
2. receiver 는 sender 에서 온 모든 옵션 적용
3. 다음 hop 에 메시지를 전달하기 전에 Connection 헤더 제거

### Serial Transaction Delay (순차 트랜잭션 처리에 의한 지연)
![image](https://user-images.githubusercontent.com/10507662/109806136-ad8b5f00-7c67-11eb-927a-d9443fdc4c52.png)
- 다음 섹션부터 이 문제를 해결할 여러 방법들이 나옴

## Parallel connection
- HTTP Client 가 커넥션을 여러 개 맺어서 병렬로 내려받기

### Parallel connection 은 페이지를 더 빠르게 내려받음
![image](https://user-images.githubusercontent.com/10507662/109806580-32767880-7c68-11eb-8b43-27717af4dbca.png)

### Parallel connection 이 항상 더 빠르지는 않음
- 네트워크 자체의 성능이 좋지 않을 때
- 오히려 병렬로 해당 네트워크롤 사용하므로 Single 인 경우보다 느릴 수 있음
- 최근 브라우저들은 6~8 개로 parallel 수 제한이 있음

### Parallel connection 은 더 빠르게 느껴질 수 있음
- 페이지를 항상 더 빠르게 로드하는 건 아니지만
- 여러 개가 동시에 나타나는걸 사용자가 볼 수 있음 -> 더 빠르게 느낌

## Persistent connection
