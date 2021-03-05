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
- HTTP/1.1 에는 TCP 커넥션을 유지, 다음 HTTP 요청에 재사용 가능
- 처리가 완료된 후에도 **계속 연결된 상태**로 있는 TCP 커넥션
- 커넥션을 맺기 위한 준비작업 시간 절약 + TCP slow start 로 인한 지연 피할 수 있음

### Persistent vs Parallel
- parallel 의 단점
  - 각 트랜잭션마다 새로운 커넥션을 맺고 끊음 -> 시간과 대역폭 소요
  - TCP slow start 로 인한 저성능
  - 실제 연결할 수 있는 parallel 수는 제한이 있음
- persistent 의 장점
  - 커넥션을 맺기위한 작업과 지연을 줄여줌
  - 튜닝된 커넥션 유지
  - 커넥션 수 줄여줌
- persistent 를 잘못 관리할 경우 계속 연결된 상태로 있는 수많은 커넥션이 쌓을 수 있음
- persistent + parallel 가 가장 효과적
- HTTP/1.0+ - keep-alive, HTTP/1.1 Persistent

### HTTP/1.0+ Keep-Alive connection
- interoperability 에 관련된 설계에 문제가 있음, HTTP/1.1 에서 수정됨

![image](https://user-images.githubusercontent.com/10507662/110123784-b36a7700-7e04-11eb-8d04-d5eb684bba92.png)

### Keep-Alive 동작
- `Connection:Keep-Alive` 헤더를 포함
- 서버가 이 요청을 받은 경우, 응답에 같은 헤더 포함

### Keep-Alive Option
- keep-alive 요청을 받았다고 해서 무조건 유지할 필요는 없으며
- 언제든지 끊거나, 트랜잭션 수 제한 가능

#### 파라미터
- timeout: 커넥션이 얼마간 유지될 것인지
- max: 커넥션이 몇 개의 HTTP 트랜잭션을 처리할 때까지 유지할 것인지
- 진단, 디버깅을 위한 임의 속성도 지원. `name[=value]` 형식

```
Connection: Keep-Alive
Keep-Alive: max=5, timeout=120
```

### Keep-Alive connection restrictions and rules
- HTTP/1.0 에서 디폴트는 아님, 헤더 명시
- 유지하려면 모든 메시지에 Connection: Keep-Alive 헤더 포함
- 커넥션이 끊어지기 전에 entity body 의 길이를 알아야 커넥션 유지 가능
- Proxy, Gateway 는 Connection 헤더 규칙을 지켜야함
  - 메시지를 전달하거나 캐시에 넣기 전에 Connection 헤더에 명시된 모든 헤더 필드와 Connection 필드 제거해야 함
- keep-alive 커넥션은 Connection 헤더를 인식하지 못하는 Proxy 서버와는 맺어지면 안됨
  - dumb proxy 로 인해 발생할 문제 예방
- HTTP/1.0 을 따르는 Connection 헤더 필드는 무시해야 함
  - 오래된 Proxy 서버로부터 실수로 전달될 수 있음
- 클라이언트는 응답 전체를 모두 받기 전에 커넥션이 끊어진 경우, 요청을 다시 보낼 수 있게 준비해야 함

### Keep-Alive and dumb proxy
- Connection 헤더는 hop-by-hop 헤더임을 기억

![image](https://user-images.githubusercontent.com/10507662/110125018-304a2080-7e06-11eb-9f40-ac0319a1174c.png)
- 클라이언트 - proxy, proxy - 서버에 각각 keep-alive 커넥션이 유지됨
  - 클라이언트 - 서버는 Connection 헤더가 잘 넘어왔기에 서로 연결된 것으로 생각
- proxy 는 Connection 헤더 자체, 즉 keep-alive 가 어떤 의미인지 모름
- 클라이언트 - proxy
  - 클라이언트가 유지된 커넥션으로 다음 요청을 보내도
  - proxy 는 같은 커넥션에서 다른 요청이 오는 경우를 예상하지 못함 -> **아무것도 하지 않음**
- proxy - 서버
  - 서버는 커넥션을 계속 유지하고 있으므로, proxy 쪽과 연결된 리소스가 무한정 대기

### Proxy-Connection
- 바로 위 문제를 우회하기 위해, 넷스케이프에서 도입한 헤더
- dumb proxy 는 무조건 헤더를 보냄
  - Proxy-Connection 은 keep-alive 스펙이 아니므로, 위와 같은 문제가 생기지 않음
- 이 헤더를 인식하는 smart proxy 는 이 값을 변경하여 커넥션을 유지

![image](https://user-images.githubusercontent.com/10507662/110126180-84093980-7e07-11eb-8a92-9027653312ef.png)

- 물론 아래와 같이 dumb proxy 가 어느 사이에 끼어있다면 문제가 발생할 수 있음

![image](https://user-images.githubusercontent.com/10507662/110126282-9daa8100-7e07-11eb-8245-f9be760f3d44.png)

### HTTP/1.1 의 Persistent connection
- 별도 설정이 없는 경우, 디폴트로 활성화
- 끊으려면 `Connection: close` 헤더를 명시
  - 이 헤더를 보내지 않아도, 커넥션을 영원히 유지하겠다는 뜻은 아님

### Persistent connection restrictions and rules
- 클라이언트가 요청에 `Connection: close` 를 보냈으면, 같은 커넥션으로 추가 요청 불가능
- 클라이언트가 해당 커넥션으로 추가 요청을 보내지 않는다면 `Connection: close` 헤더 보내야 함
- 커넥션에 있는 모든 메시지가 자신의 길이 정보를 정확히 가지고 있을 때에만 커넥션 지속 가능
  - entity body 는 정확한 Content-Length 를 가지거나, chunked transfer encoding 으로 인코딩 되어 있어야 함
- HTTP/1.1 proxy 는 클라이언트/서버 각각에 대해 별도 Persistent 커넥션을 맺고 관리해야 함
- HTTP/1.1 proxy 는 클라이언트가 커넥션 관련 기능에 대한 클라이언트의 지원 범위를 알지 못하면, Persistent 커넥션을 맺으면 암됨
  - 위에서 본 오래된 proxy 가 Connection 헤더를 전달하는 문제가 발생할 수 있기 때문
- 서버는 메시지를 전송하는 중간에 커넥션을 끊지 않을 것이고, 커넥션을 끊기 전에 적어도 한 개의 요청에 대해 응답을 할 것이긴 하지만, HTTP/1.1 기기는 Connection 헤더의 값과는 상관없이 언제든지 커넥션을 끊을 수 있다
- HTTP/1.1 애플리케이션은 중간에 끊어지는 커넥션을 복구할 수 있어야 한다
  - 클라이언트는 다시 보내도 문제가 없는 요청이라면 가능한 다시 보내야 한다
- 클라이언트는 전체 응답을 받기 전에 커넥션이 끊어지면, 요청을 반복해서 보내도 문제가 없는 경우에는 요청을 다시 보낼 준비가 되어 있어야 한다.
- 하나의 사용자 클라이언트는 서버의 과부하를 방지하기 위해서, 넉넉잡아 2개의 지속 커넥션만을 유지해야 한다.
  - N 명의 사용자가 서버로 접근하려 한다면, proxy 는 서버나 상위 proxy 에 넉넉잡아 약 2N 개의 커넥션을 유지해야 한다

## Pipeline connection
- Persistent 커넥션을 통한 요청 pipelining

![image](https://user-images.githubusercontent.com/10507662/110127299-d434cb80-7e08-11eb-93b7-9ee567f1af8e.png)

- Persistent connection 인지 확인 전에는 파이프라인을 이으면 안됨
- 응답은 요청 순서와 같게 와야 함
- 커넥션이 끊어지더라도, 완료되지 않은 요청이 파이프라인에 있으면 언제든 다시 요청을 보낼 준비가 되어 있어야 함
- 클라이언트는 POST 요청같이 반복해서 보낼 경우 문제가 생기는 요청 (nonidempotent) 은 파이프라인을 통해 보내면 안됨

## Connection Close 에 대한 미스터리
- 명확한 기준이 없음, 관련 기술 문서도 별로 없음

### 마음대로 끊기
- HTTP 클라이언트, 서버, proxy 모두 언제든지 TCP 전송 커넥션을 끊을 수 있음
- 보통은 메시지를 다 보내고 끊지만, 에러가 있는 상황에선 엉뚱한 곳에서 끊길 수 있음
- Persistent 커넥션이 일정 시간 동안 요청을 전송하지 않고 유휴 상태에 있으면, 서버는 그 커넥션을 끊을 수 있음

### Content-Length 와 Truncation
- 각 HTTP 응답은 본문의 정확한 크기 값을 가지는 Content-Length 헤더를 가지고 있어야 함
- 클라이언트나 proxy 가 커넥션이 끊어졌다는 HTTP 응답을 받은 후
  - 실제 전달된 entity 의 길이와 Content-Length 값이 일치하지 않거나
  - Content-Length 자체가 존재하지 않으면
  - receiver 는 데이터의 정확한 길이를 서버에게 물어봐야 함
- receiver 가 cache proxy 일 경우 응답을 캐시하면 안됨
- proxy 는 Content-Length 를 수정하려고 하지 말고 메시지를 받은 그대로 전달해야 함

### Connection Close Tolerance, Retries, Idempotency
- HTTP 애플리케이션은 예상치 못하게 커넥션이 끊어졌을 때 적절히 대응할 수 있는 준비가 되어 있어야 함
- GET, HEAD, PUT, DELETE, TRACE, OPTIONS 는 idempotent 로 보고
- POST 와 같은 요청은 파이프라인을 통해 요청하면 안됨
- nonidempotent method 나 sequence 에 대해 자동으로 재시도하면 안됨

### Graceful Connection Close
![image](https://user-images.githubusercontent.com/10507662/110128999-c08a6480-7e0a-11eb-8017-bfc3d5824a77.png)

#### Full and Half close
![image](https://user-images.githubusercontent.com/10507662/110129424-4a3a3200-7e0b-11eb-9ba3-a3f99465f695.png)

- TCP in, out 채널을 하나만 끊거나, 둘 다 끊을 수 있음
- close()/shutdown()

#### TCP close and reset errors
- simple HTTP 애플리케이션은 Full close 만 사용할 수 있음
- 각기 다른 HTTP 클라이언트, 서버, proxy 와 통신할 때 + pipelined persistent connection 을 사용할 때
  - 기기들에 예상치 못한 쓰기 에러를 발생하는 것을 예방하기 위해 shutdown 을 사용해야 함
- 커넥션의 out channel 을 끊는 것이 안전
- 커넥션의 반대편에 있는 기기는 모든 데이터를 버퍼로부터 읽고 나서, 데이터 전송이 끊남과 동시에 커넥션을 끊었다는 것을 알게 됨
- 클라이언트에서 더는 데이터를 보내지 않을 것임을 확신할 수 없다면, 커넥션의 in channel을 끊는 것은 위험
- 클라이언트에서 이미 끊긴 in channel 에 데이터를 전송하면 OS 단에서 **`connection reset by peer`** 출력
  - OS 단에서 심각한 에러로 취급하여 버퍼에 저장된 데이터 제거
  - ex) pipelined persistent connection 으로 10개의 request 를 보냈고, 응답이 OS 버퍼에는 있으나 아직 애플리케이션까진 안옴
  - 11번째 request 를 보낼 때 server in channel 이 끊긴 경우, connection reset by peer 가 넘어오고
  - 해당 응답이 오면 OS 는 버퍼를 날림 -> 애플리케이션에서 데이터를 읽을 때 이전 데이터는 없고 reset 메시지만 받음

#### Graceful close
- HTTP 명세에서는 클라이언트나 서버가 예기치 않게 커넥션을 끊어야 한다면, "issue a graceful close on the transport connection," 해야 한다고 하나, 방법에 대한 설명은 없음
- 애플리케이션이 자신의 out channel 을 끊고 -> 다른 쪽에 있는 기기의 out channel 이 끊기는 것을 기다리면 가능
- 상대방이 half close 를 구현했다는 보장도 없고, half close 를 했는지 검사해준다는 보장도 없음
- graceful 하려면 out channel half close 를 한 후에도 데이터나 스트림의 끝을 식별하기 위해 in channel 에 대해 상태 검사를 주기적으로 해야 함
- in channel 이 특정 타임아웃 내에 끊어지지 않으면, 애플리케이션이 리소스 보호를 위해 커넥션을 강제로 끊을 수도 있음