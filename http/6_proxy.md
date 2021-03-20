# 6. Proxy
## Web Intermediaries (중개자)
- Proxy Server: Web Server 이면서 동시에 Web Client

### Public proxy
- 대부분의 proxy
- cache proxy server 등

### Private proxy
- 브라우저 기능 확장, 성능 개선, 무료 ISP 서비스를 위한 광고 운영 등
- 사용자의 컴퓨터에서 직접 실행

### Proxy vs Gateway
- Proxy: **같은 프로토콜을 사용하는** 둘 이상의 애플리케이션 연결
- Gateway: **서로 다른 프로토콜을 사용하는** 둘 이상을 연결
- commercial proxy server 는 SSL security protocol, SOCKS firewall, FTP access 등의 게이트웨이 기능을 구현하기도 함
  - gateway 는 8장에서

## 왜 proxy 를 사용?
- 보안 개선, 성능 향상, 비용 절약
- HTTP 트래픽을 보고, 수정할 수 있음 -> 유용한 기능을 구현하기 위해 감시 + 수정

### 예시
- Child filter: 특정 사이트는 열고 나머지는 접근 제한
- Document access controller: 많은 웹 서버들과 웹 리소스에 대한 단일 접근 제어 전략 구현
  - 특정 문서에 password lock 을 건다던지
- Security firewall: in/outbound 네트워크를 한 지점에서 통제. 트래픽을 자세하게 볼 수 잇는 hook 제공
- Web cache
  - popular documents 에 대한 사본을 저장하여 해당 요청인 경우 빠르게 제공
  - network communication 비용을 줄임
- Surrogate (reverse proxy)  
![image](https://user-images.githubusercontent.com/10507662/111861756-03644480-8994-11eb-9c62-cfd6a138937c.png)
  - real web server 처럼 요청을 받지만 실제 컨텐츠를 가져오기 위해 다른 서버와 커뮤니케이션 시작
  - 느린 웹 서버의 성능을 개선되기 위해 사용될 수 있음 (server accelerators)
  - contents routing 기능과 결합하여 on-demand replication contents 에 대한 distributed network 구성 가능
- Content router
  - 트래픽 condition 과 종류에 따라 특정 웹 서버로 유도
  - 사용자의 결제 금액에 따라 가까운 replication cache 로 전달 등
- Transcoder: 본문 포맷을 수정 (GIF -> JPG 로 변환 등)  
![image](https://user-images.githubusercontent.com/10507662/111861823-8daca880-8994-11eb-8ec3-2a02ffe42a8f.png)
- Anonymizer: 사용자를 식별할 수 있는 IP address, From, Referer, Cookie 등을 제거하여 개인정보보호, 익명성 보장

## Proxy 는 어디에?
### Proxy server deployment (배치)
![image](https://user-images.githubusercontent.com/10507662/111861875-f3993000-8994-11eb-881d-c8804b7d876d.png)
#### Egress proxy (출구)
- 방화벽 기능, 요금 절약, 성능 개선 등

#### Access proxy (입구)
- ISP 가 사용자들의 다운로드 속도 개선, 대역폭 비용을 줄이기 위해 cache proxy 사용

#### Surrogate
- 웹 서버로 향하는 모든 요청 처리, 필요할 때만 웹 서버에 자원 요청, 보안 기능 추가 등

#### Network exchange proxy
- 캐시를 이용해 혼잡 완화, 트래픽 흐름 감시

### Proxy hierarchies
![image](https://user-images.githubusercontent.com/10507662/111861982-889c2900-8995-11eb-9dea-c8c70162b116.png)

#### Proxy hierarchy content routing
- 그림은 static
- **dynamic parent selection** 예
  - Load balancing
  - Geographic proximity routing
  - Protocol/type routing
  - Subscription-based routing

### proxy 가 트래픽 처리를 어떻게 하는가
![image](https://user-images.githubusercontent.com/10507662/111862300-cac66a00-8997-11eb-8f33-80854b9d19c5.png)

- Client 가 proxy 로 가도록 만드는 방법

#### Client 수정
- 브라우저 단 수동/자동 proxy 설정

#### Network 수정
- network infra 를 가로채서 처리. interceptor proxy 라고 부름
- switching, routing device 가 필요

#### DNS namespace 수정
#### Web server 수정
- HTTP redirection status code 를 넘겨서 proxy 로 보낼 수 있음

## Client proxy 설정
- 수동 설정
- 브라우저 기본 설정

### Proxy auto-configuration (PAC)
- javascript PAC file 에 대한 URI 제공
- 수동 설정보다 유연하게 제공 - 상황에 맞게 계산해주는 작은 javascript 프로그램
- `.pac` 확장자, MIME `application/x-ns-proxy-autoconfig`
- `FindProxyForUrl(url, host)` 함수를 정의해야 함

```js
function FindProxyForURL(url, host) {
 if (url.substring(0,5) == "http:") {
 return "PROXY http-proxy.mydomain.com:8080";
 } else if (url.substring(0,4) =="ftp:") {
 return "PROXY ftp-proxy.mydomain.com:8080";
 } else {
 return "DIRECT";
 }
}
```

#### 반환값
|value|description|
| ----------- | ----------- |
|DIRECT|proxy 없이 연결이 직접|
|PROXY host:port|지정한 proxy 사용|
|SOCKS host:port|지정한 SOCKS 서버 사용|

### WPAD proxy discovery
- Web Proxy Autodiscovery Protocol
- 여러 discovery mechanism 에 대한 escalating strategy 를 이용하여 **알맞은 PAC 파일을 자동으로 찾아주는 알고리즘**
- 하는 일
  - PAC URI 를 찾기 위해 WPAD 사용
  - 주어진 URI 에서 PC 파일 가져옴
  - proxy server 를 알아내기 위해 PAC 파일 실행
  - 알아낸 proxy server 를 이용하여 요청 처리

## Proxy request 에 대한 tricky things
### Proxy URI 는 Server URI 와 다르다
- proxy 로 요청을 보내는 경우 (scheme, host, port 등이 없는) partial URI 가 아닌 전체 정보를 담은 URI 로 보내야 함

### Virtual hosting 에서 발생하는 동일한 문제
- scheme/host/port 누락은 여기서도 문제
- 해결된 방법
  - 명시적인 proxy 는 client 요청이 full URI 를 갖도록 요구
  - Virtual hosting 은 (host, port 정보가 담긴) Host 헤더 요구

### Intercepting proxy 는 partial URI 를 받음
- client 가 항상 proxy 와 communication 하고 있는지 알지는 못함
- surrogate, intercepting proxy 등은 그대로 받음

### Proxy 는 Proxy request 와 Server request 모두 다를 수 있음
- full URI 가 주어진 경우 그대로 사용
- partial URI 가 주어지고 Host 헤더가 있다면 Host 헤더를 이용해 원 서버의 name, port 를 알아내야 함
- partial URI 가 주어졌는데 Host 헤더가 없다면
  - Surrogate 라면 proxy 에 실제 서버의 name, port 가 설정되어 있을 수 있음
  - 이전 Intercepting proxy 가 가로챘던 트래픽을 받았고, 그 proxy 가 ip address 와 port 를 사용할 수 있게 했다면 그대로 사용
  - 모두 실패했다면 Client 에 에러 메시지 반환

### In-Flight URI Modification
- interoperability 문제를 일으킬 수 있으므로 거의 변경하면 안됨

### URI Client Auto-Expansion and Hostname Resolution
### URI Resolution Without a Proxy
### URI Resolution with an Explicit Proxy
### URI Resolution with an Intercepting Proxy

## Message 추적
### Via header
- message 가 지나는 각 중간 노드의 정보를 나열
- 중간 노드는 Via 목록의 끝에 반드시 추가되어야 함
- message 전달 추적, 루프 진단, 중간의 모든 sender 들에 대한 프로토콜 처리를 보기 위함
- routing loop 를 탐지하기 위해 proxy 는 요청을 보내기 전에 unique 문자열을 Via 헤더에 삽입해야 함
  - 네트워크에 routing loop 가 있는지 탐지하기 위해 이 문자열에 들어온 요청에 있는지 검사

#### syntax
`Via: 1.1 proxy.net, 1.0 ers.naver.com`

#### Via request and response paths
![image](https://user-images.githubusercontent.com/10507662/111863490-40820400-899f-11eb-9f9c-cd4529a133cf.png)
- request 와 reponse header 는 반대

#### Via and gateway
- protocol 변환을 기록, FTP/1.0 와 같은 식

#### Server header and Via header
- Server response header 는 원 서버에 의해 사용되는 소프트웨어를 알려줌
- proxy 는 Server header 를 수정하면 안됨

#### Via 가 개인정보보호와 보안에 미치는 영향
- 방화벽 뒤의 네트워크 아키텍처에 대한 정보가 이용될 수 있음
  - proxy 가 hostname 을 가명으로 교체해야 함
  - 합치기
  - `Via: 1.0 foo, 1.1 bar.company.com, 1.1 logger.company.com` -> `Via: 1.0 foo, 1.1 concealed-stuff`

### TRACE method
- proxy 를 거칠 때마다 message content 가 어떻게 바뀌는지 관찰
- destination server 에 도착했을 때, 전체 request message 를 response body 에 포함시켜 보냄
- Content-Type: message/http

#### Max-Forwards
- 디폴트는 무한
- hop 개수를 제한하기 위해 사용할 수 있음

## Proxy authentication
- `407 Proxy Authentication Required` + `Proxy-Authenticate` 헤더로 자격 제출에 대한 설명 반환
- Client 는 `Proxy-Authenticate` 헤더에 정보를 담아서 다시 요청
- proxy 가 유효 체크하여 보내든, 407 로 다시 보내든 정함

## Proxy Interoperation
### 지원하지 않는 header, method 다루기
- 이해할 수 없는 header/method 는 반드시 그대로 전달, 순서도 그대로 전달
- method 를 통과시킬 수 없는 proxy 는 대부분의 네트워크에서 살아남지 못함 (없어졌다는 뜻?)

### OPTIONS: 어떤 기능을 지원하는지 알아보기
![image](https://user-images.githubusercontent.com/10507662/111863885-65777680-89a1-11eb-9682-093a633a4768.png)

### Allow header
- 지원하는 method 나열
- 새 resource 가 지원했으면 하는 method 를 추천하기 위해 request header 로 사용 가능