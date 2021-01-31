# 1. HTTP 개관
- 신뢰성 있는 데이터 전송 프로토콜 - 전송 중 손상되거나 꼬이지 않음을 보장

## Web Client, Server
![image](https://user-images.githubusercontent.com/10507662/106375931-e53a8900-63d3-11eb-9d1a-5083e85275d3.png)

## Resource
- static resource, dynamic resource

### Media type
- MIME: Multipurpose Internet Mail Extensions
  - 서로 다른 이메일 시스템에서 메시지가 오갈 때 겪는 문제점을 해결하기 위해 설계됨
  - HTTP 에서도 멀티미디어 컨텐츠를 기술하고, 라벨을 붙이기 위해 채택
- 모든 객체 데이터에 MIME 타입을 붙임
  - 이 정보를 통해 다룰 수 있는 객체인지 판별
- `primary object type/specific subtype`
  - `text/html`, `text/plain`, `video/quicktime`, ...

### URI
- Uniform Resource Identifier
  - 리소스를 고유하게 식별하고 위치를 지정
- URL, URN 두 가지가 있음

![image](https://user-images.githubusercontent.com/10507662/106376031-a953f380-63d4-11eb-94c6-3fccbdff1c30.png)

### URL
- Uniform Resource Locator

#### 구성
- first part (`scheme`): 사용되는 프르토콜 서술 (ex. `http://`)
- second part: 서버의 인터넷 주소 (`kin.naver.com`)
- rest: 웹 서버의 리소스 (`/index.nhn`)

- 대부분의 URI = URL

### URN
- Uniform Resource Name
- 한 리소스에 대해, 리소스의 위치에 영향을 받지 않은 유일무이한 이름
- location-independent: 리소스를 옮기더라도 문제없이 동작
- experimental, 아직 채택되지 않음

## Transaction
- HTTP Transaction
  - request command (client -> server) + response result (server -> client)

### Method
- 서버에게 어떤 동작을 해야하는 지 알려줌
- 3장에서 다룰 예정

### Status Code
- 클라이언트에 요청에 대한 정보 반환
- 3장에서 다룰 예정

## Message
![image](https://user-images.githubusercontent.com/10507662/106376169-f2f10e00-63d5-11eb-878a-4f204e3de049.png)
- `start line`
  - request: 무엇을 해야하는 지
  - response: 무슨 일이 일어났는지
- `header`: name:value 의 형태, 헤더는 blank line 으로 끝남
- `body`: 아무 데이터든 들어갈 수 있는 메시지 본문 (binary, text, ...)

## TCP (Transmission Control Protocol) Connection
### TCP/IP
- HTTP: Application layer, 네트워크 통신의 핵심은 신경쓰지 않음
- TCP 의 제공 기능
  - 오류 없는 데이터 전송
  - 순서에 맞는 전달
  - 조각나지 않는 데이터 스트림
- 네트워크와 하드웨어의 특성을 숨기고, 신뢰성 있는 의사소통을 하게 해줌
- HTTP <- TCP <- IP

### Connection, IP address, Port number
- ip address, port number 를 사용해 client-server connection 을 맺어야 함
- 순서
  - URL 에서 hostname 추출 (browser)
  - hostname 을 IP 로 변환 (browser)
  - URL 에서 port number 추출 (browser)
  - server 와 TCP connection 맺음 (browser)
  - server 에 HTTP request (browser)
  - browser 에 HTTP response (server)
  - connection 이 닫히면 문서 노출 (browser)

### 예제
- telnet, nc (netcat)

## Protocol Version
- HTTP/0.9
  - 결함이 많음, GET 만 지원, ...
- HTTP/1.0
  - 버전 번호, HTTP 헤더, 추가적인 메소드, 멀티미디어 객체 처리
- HTTP/1.0+
  - keep-alive, virtual hosting, proxy connection
- HTTP/1.1
  - HTTP 설계의 구조적 결함 교정, 성능 최적화, 잘못된 기능 제거
- HTTP/2
  - google SPDY 기반

## Components
### Proxy
- client ~ server 사이에 위치
- client 의 요청을 받아 server 로 전달
- 보안 (요청/응답 필터링), 성능 최적화 등
- 6장에서

### Cache
- 자주 조회되는 문서에 대한 사본을 저장해두는 서버
- 7장에서

### Gateway
- HTTP protocol 을 다른 protocol 로 변환하기 위해 사용
- 8장에서

### Tunnel
- 두 connection 사이에서 raw data 를 보지 않고 그대로 전달해주는 HTTP applcation
- ex) 암호화된 SSL 트래픽을 HTTP connection 으로 전송, 웹 트래픽만 사내 방화벽을 통과

### Agent
- HTTP request 를 만들어주는 client program
- 9장에서

# 2. URL 과 Resource
