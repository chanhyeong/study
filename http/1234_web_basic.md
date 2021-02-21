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
## Resource 탐색하기
- **`scheme://host/path`**
  - how, where, what
  - mailto:president@bluehouse.gov
  - ftp://ftp.test.com/excel.xls

## URL Syntax
**`<scheme>://<username>:<password>:<host>:<port>/<path>;<params>?<query>#<fragment>`**

|Component|Description|Default value|
| ----------- | ----------- | ----------- |
|scheme|어떤 프로토콜을 사용하여 서버에 접근해야 하는지|-|
|username|특정 scheme 은 username 요구|anonymous|
|password|-|email address|
|host|서버의 hostname or IP address|-|
|port|port number|scheme 마다 다름|
|path|서버 내 리소스가 어디에 있는지|-|
|parameter|특정 scheme 에서 파라미터를 기술하는 요도로 사용, name/value 값을 가지고 `;` 로 구분|-|
|query|application 에 파라미터를 전달하는데 사용, `?`|-|
|fragment|리소스의 조각이나 일부분을 가리키는 이름, `#`|-|

### Scheme: 사용할 프로토콜
- 리소스에 어떻게 접근하는지. 대소문자 구분 없음

### Host, Port
- 리소스를 호스팅하는 장비 및 장비 내에서 접근할 수 있는 서버가 어디인지

### Username, Password
### Path
- 리소스가 서버의 어디에 있는지

### Parameter
- 서버에 정확한 요청을 하기 위해 필요한 입력 파라미터를 받는데 사용
```
ftp://prep.ai.mit.edu/pub/gnu;type=d
```
- `name`: type, `value`: d

### Query String
- 많이 사용하니 그림으로 대체
![image](https://user-images.githubusercontent.com/10507662/108624311-8a062e80-7487-11eb-8aa9-4b77cec40b6b.png)

### Fragment
- 서버에 fragment 는 보내지 않음
- 리소스를 받은 후 fragment 를 통해 해당하는 일부를 보여줌

## 단축 URL (URL Shorcuts)
### Relative URL
- URL 을 짧게 표시하는 방식
  - scheme, host 및 다른 component 를 입력하지 않아도 됨
  - resource 의 base URL 에서 알아낼 수 있음 (변환)
- `fragment` 이거나 `URL 의 일부`

![image](https://user-images.githubusercontent.com/10507662/108624441-5677d400-7488-11eb-8bda-82a0befd0f0f.png)

#### base URL
변환의 첫 단계는 base URL 찾기
- 리소스에서 명시적으로 제공 - <BASE>태그로 제공
- 리소스를 포함하고 있는 base URL - 위 그림과 같은 예
- base URL 이 없는 경우 - absolute or imcomplete or broken

#### Relative reference 해석
- Relative -> Absolute 변환
- 앞에서 본 component 단위로 분리
- RFC 1808 에 최초 기술, RFC 2396 에 포함
![image](https://user-images.githubusercontent.com/10507662/108624632-6d6af600-7489-11eb-9be7-24d4beec766d.png)

### URL 확장 (Expandomatic URL)
- 브라우저의 URL 을 입력한 다음 or 입력하는 동안에 자동으로 URL 확장

#### hostname expansion
- heuristic 만을 사용하여 일부 -> 전체로 확장
- ex) `yahoo` 입력 시 -> www. .com 을 붙여서 만듦

#### history expansion
- 과거에 사용자가 방문한 URL 을 저장

## Shady Characters (안전하지 않은 문자)
- 안전한 전송?
 - 정보가 유실될 위험 없이 URL 을 전송할 수 있다
 - 반대 케이스) SMTP 는 특정 문자를 제거할 수 있는 전송 방식을 사용 (7bit encoding)

### URL Character Set
- 기존 Character Set 은 `US-ASCII`, `7bit` 를 사용
  - 유럽 언어 or 비 라틴계 언어들을 지원하지 않음
- 다른 언어 + 특정 이진 데이터 등을 지원하기 위해 escape 문자열 설계
  - US-ASCII 에서는 사용이 금지된 문자들

### Encoding Mechanism
- `%` + ASCII 코드

|Character|ASCII Code|Example|
| ----------- | ----------- | ----------- |
|-|126 (0x7E)|http://www.naver.com?%7Ejoe|
|SPACE|32 (0x20)|http://www.naver.com?naver%20pay|

### Character Restriction 문자 제한
- 예약어, 반드시 encoding 해야하는 문자들

![image](https://user-images.githubusercontent.com/10507662/108625051-0438b200-748c-11eb-8773-6198d473ba97.png)

## Schemes
- http, https, mailto, ftp
- rtsp, rtspu (real time streaming protocol)
- file, news, telnet

## 미래
### URN
- 객체가 이동되더라도 항상 객체를 가리킬 수 있는 이름 제공
- PURL: Persistent Uniform Resource Locations

### 언제?
- URL -> URN 로 체계를 바꾸기에는 매우 큰 작업
- 언젠가 바뀌지 않을까 책에서는 이야기하지만 먼 듯 (2002년에 초판이 나왔으니..)