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
**`<scheme>://<username>:<password>@<host>:<port>/<path>;<params>?<query>#<fragment>`**

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

# 3. HTTP message
- inbound message, outbound message
  - original server 쪽으로 향하는건 inbound, 사용자에게 돌아오는 것은 outbound
- (request, response 의 관계없이) 모든 메시지는 downstream 으로 흐른다.
![image](https://user-images.githubusercontent.com/10507662/109492019-ac262f00-7acd-11eb-846f-654799be2632.png)

## The Parts of a Message
- **start line**, **headers**, **body**
### Syntax
```
// request message
<method> <request-URL> <version>
<headers>

<entity-body>

// response message
<version> <status code> <reason-phrase>
<headers>

<entity-body>
```

![image](https://user-images.githubusercontent.com/10507662/109492552-82b9d300-7ace-11eb-8dbb-bfff2187067c.png)

### Start line
- request 에서는 무엇을 해야 하는지
- response 에서는 무슨 일이 일어났는지

#### Request line
- `<method> <request-URL> <version>`
- 어떤 동작 + 대상 + HTTP 버전

#### Response line
- `<version> <status code> <reason-phrase>`
- HTTP 버전 + 상태 + 상태에 대한 텍스트 사유

#### method
- 요청 시 서버에게 무엇을 해야 하는지에 대한 내용

|method|description|body|
| ----------- | ----------- | ----------- |
|GET|서버에서 문서를 가져옴|X|
|HEAD|서버에서 문서의 헤더만 가져옴|X|
|POST|서버가 처리해야 할 데이터를 보냄|O|
|PUT|서버에 요청 메시지의 본문을 저장|O|
|TRACE|메시지가 proxy 를 거쳐 서버까지 도달하는 과정 추적|X|
|OPTIONS|서버가 어떤 메소드를 수행할 수 있는지 확인|X|
|DELETE|서버에서 문서 제거|X|

#### status code
#### reason phrase
- status code 에 대한 글로 된 설명

#### version number
- **`HTTP/x.y`** 형식
- HTTP 통신을 하는 애플리케이션 간 capabilities 와 메시지 형식에 대한 단서 제공
- 애플리케이션이 지원하는 가장 높은 HTTP version 을 의미함
  - HTTP/1.0 의 애플리케이션 A 가 다른 애플리케이션 B 로 부터 응답으로 HTTP/1.1 로 받은 경우
  - A 는 해당 메시지를 HTTP/1.1 메시지로 다룰 수 없음
  - 여기서는 보낸 주체인 B 가 HTTP/1.1 까지 지원한다는 의미

### Headers
- General, Request, Response, Entity, Extension header

### Entity body
- optional
- 이미지, 비디오, HTML, ...

## Method
- 서버가 모든 메소드를 구현하지는 않음
- Safe method: GET, HEAD
  - HTTP 요청의 결과로 서버에 어떤 변화도 없음

### GET
- 서버에 리소스 요청

### HEAD
- GET 처럼 행동하나 서버는 헤더만 돌려줌
  - 리소스를 가져오지 않고 타입 등의 정보를 알 수 있음
  - status code 를 통해 개체 존재 확인 가능
  - 헤더를 확인하여 리소스 변경 검사
- 서버 개발자는 GET 으로 받는 헤더가 정확히 일치함을 보장해야 함

### PUT
- 서버가 request body 를 가지고 URL 대로 새 문서를 만들거나
- 이미 존재한다면 body 로 교체

### POST
- 서버에 입력 데이터를 전송하기 위해 설계됨

### TRACE
- 자신의 요청이 firewall, proxy, gateway 등을 통과하여 서버에 어떻게 도착했는지 보여줌
- 서버에서 loopback 진단을 수행 -> 받은 요청 메시지를 body 에 넣어 응답
- 주로 진단을 위해 사용
  - request 가 의도한 request/response chain 을 이루는지
  - proxy, 다른 애플리케이션이 request 에 어떤 영향을 미치는지

### OPTIONS
### DELETE
- request URL 로 지정한 리소스 삭제 요청
- 삭제 수행을 보장하지 못함
  - HTTP 명세에서 서버가 클라이언트에게 알리지 않고 요청을 무시하는 것을 허용함

### Extension method
- HTTP 는 필요에 따라 확장해도 문제가 없도록 설계되어 있음
- 새로 기능을 추가해도 과거에 구현된 소프트웨어들의 오동작을 유발하지 않음
- 알려지지 않은 메소드가 담긴 메시지를 전달하고자 하면, `501 Not Implemented` 로 응답할 수 있음

## Status code
- 필요 시 찾아보는 정도로 하고 많이 생략

### 100 - 199: Informational
- HTTP/1.1 에서 도입
- 복잡한데 이 복잡성을 감수할 만한 가치가 있는지 논란이 되고 있음
  - 안 쓰이는거 보면 복잡하고 의미없긴 한 듯
- 간단하게 표로만 추가

|status code|reason phrase|meaning|
| ----------- | ----------- | ----------- |
|100|Continue|요청의 시작 부분 일부가 받아들여졌으며, 클라이언트는 나머지를<br/>계속이어서 보내야함, 서버는 반드시 요청을 받아 응답해야 함|
|101|Switching Protocols|클라이언트가 Upgrade 헤더에 나열한 것 중 하나로<br/> 서버가 프로토콜을 바꾸었음|

### 200 - 299: Success
### 300 - 399: Redirection
- 리소스에 대해 다른 위치를 사용하라고 말해주거나, 다른 대안 응답을 제공
- 특정 몇 가지는 애플리케이션의 로컬 복사본이 원래 서버와 비교했을 때 유효한지 확인하기 위해 사용
![image](https://user-images.githubusercontent.com/10507662/109499343-1d6adf80-7ad8-11eb-9733-3c90afad9635.png)

### 400 - 499: Client Error
### 500 - 599: Server Error

## Header
### Genenal header
- 기본적인 정보 제공
- 클라이언트, 서버 둘 다 사용
- Connection, Date, MIME-Version, Trailer chunked transfer, Transfer-Encoding, Upgrade, Via

#### general cache header
- 매번 원 서버로부터 객체를 가져오는 대신, 로컬 복사본으로 캐시하는 헤더 (HTTP/1.0 에서 도입)
- `Cache-Control`
- `Pragma` 도 있는데 Deprecated

### Request header
- request message 에서만 의미를 갖는 헤더
- Client-IP, From, Host, Referer, User-Agent 등
- (별개로) User-Agent 는 Deprecated 처리 예정이며, Client Hints 로 대체될 예정

#### Accept 관련 헤더
- 클라이언트가 서버에게 자신의 preference 와 capability 를 알려줌
- Accept, Accept-Charset, Accept-Encoding, Accept-Language, TE

#### Conditional request header
- Expect, Range 외

|header|description|
| ----------- | ----------- |
|If-Match|entity tag 가 일치하는 경우에만 가져옴|
|If-Modified-Since|주어진 날짜 이후에 리소스가 변경되지 않았따면 요청 제한|
|If-None-Match||
|If-Range||
|If-Unmodified-Since||

#### Request security header
|header|description|
| ----------- | ----------- |
|Authorization|클라이언트가 서버에게 제공하는 인증 정보|
|Cookie|클라이언트가 서버에게 서버에게 토큰 전달, 보안 헤더는 아니지만 영향은 있음|
|Cookie2|쿠키의 버전을 알려줌|

#### Proxy request header
|header|description|
| ----------- | ----------- |
|Max-Forwards|요청이 전달될 수 있는 최대 횟수, TRACE 와 사용|
|Proxy-Authorization|proxy 에서 인증|
|Proxy-Connection||

### Response header
- Age, Public, Retry-After, Server, Title, Warning

#### Negotiation header
- Accept-Ranges: 서버가 리소스에 대해 받아들일 수 있는 범위의 형태
- Vary: 서버가 확인해야하며, 응답에 영향을 줄 수 있는 헤더 목록

#### Response security header
- Proxy-Authenticate, Set-Cookie, Set-Cookie2, WWW-Authenticate

### Entity header
- Allow, Location

#### Content header
- Content-
  - Base, Encoding, Language, Length, Location, MD5, Range, Type

#### Entity caching header
- ETag, Expires, Last-Modified