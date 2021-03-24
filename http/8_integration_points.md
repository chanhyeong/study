# 8. Integration Points: Gateway, Tunnel, Relay
## Gateway
- interpreter 와 같은 resource 를 받기 위한 경로를 안내하는 역할
- HTTP 를 다른 protocol 로 변환하여 서버에 접속할 수 있게

## Protocol Gateway
### HTTP/*
- HTTP -> FTP 등

### HTTP/HTTPS: Server-side security gateway
![image](https://user-images.githubusercontent.com/10507662/112316659-a7c8ec80-8cee-11eb-82c3-f3193ea1ad12.png)

### HTTPS/HTTP: Client-sdide security accelerator gateway
![image](https://user-images.githubusercontent.com/10507662/112316821-cfb85000-8cee-11eb-9279-9608f2567530.png)

## Resource Gateway
![image](https://user-images.githubusercontent.com/10507662/112317248-32115080-8cef-11eb-8cf0-cc031cb05cbf.png)  
![image](https://user-images.githubusercontent.com/10507662/112317397-5a994a80-8cef-11eb-986f-ab5cfa997246.png)
- 요청 -> 애플리케이션 생성, 실행 -> 응답 반환

### Common Gateway Interface (CGI)
- 최초의 server extension
  - dynamic HTML, credit card processing, database query, etc..
- Perl, Tcl, C, various shell languages
- 매 요청마다 **새로운 프로세스를 만듦**
  - FastCGI -> daemon 으로 동작

### Server extension APIs
- 서버 자체의 동작을 바꾸거나, 서버의 performance 를 최대로 사용하고 싶을 때
- module 을 HTTP 직접 연결할 수 있는 powerful interface
- ex) FrontPage Server Extension (FPSE)
  - RPC 명령 인식
  - http 에 piggybacked 되어 넘어옴 (19장)

## Application Interface & Web Service
- 각 Web application 이 서로 통신하는데 사용할 표준과 protocal set 을 개발
- Web Service 는 SOAP (Simple Object Access Protocol) 을 통해 XML 을 사용하여 정보 교환

## Tunnel
- HTTP protocol 을 지원하지 않는 애플리케이션에 HTTP 애플리케이션을 사용해 접근하는 방법 제공
- HTTP connection 을 통해 HTTP 가 아닌 트래픽 전송

### CONNECT 로 HTTP tunnel connection 맺기
- http method `CONNECT`
  - 모든 서버나 protocol 에 TCP connection 을 맺는데 사용할 수 있음

![image](https://user-images.githubusercontent.com/10507662/112318935-e8c20080-8cf0-11eb-8d89-86fda483ceb9.png)

#### CONNECT request
```
CONNECT home.netscape.com:443 HTTP1.0
User-agent: Mozilla/4.0
```

#### CONNECT response
```
HTTP 200 Connection Established
Proxy-agent: Netscape-Proxy/1.1
```

### Data tunneling, Timing, Connection Management
- tunnel 을 통해 전달되는 데이터는 gateway 에서 볼 수 없음
  - packet 의 순서나 흐름에 대한 어떤 가정도 할 수 없음
  - 일단 연결되면 어디로든 흘러갈 수 있음
- client 는 성능으로 높이기 위해 CONNECT request 를 날리고 response 를 받기 전에 tunnel data 를 전송 가능
- gateway 는 network I/O request 가 헤더 데이터만을 반환해줄거라고 가정할 수 없어, 읽어들인 모든 데이터를 서버에 전송해야 함
- authentication challenge 나 200 외의 response 가 왔을 때 response 를 다시 보낼 준비가 되어 있어야 함
- connection 이 끊어지면, 끊어진 곳으로부터 온 데이터는 반대편으로 전달 -> 버려짐

### SSL Tunneling
![image](https://user-images.githubusercontent.com/10507662/112320068-05126d00-8cf2-11eb-90e0-1751024ee679.png)

- web tunnel 은 원래 방화벽을 통해 암호화된 SSL traffic 을 전달하려고 개발됨
- SSL traffic 을 HTTP connection 으로 전송하여 80 port 의 HTTP만을 허용하는 방화벽을 통과시킬 수 있음

![image](https://user-images.githubusercontent.com/10507662/112320188-207d7800-8cf2-11eb-94f0-18ca77f92f7e.png)

### SSL tunneling vs HTTP/HTTPS gateway
- 생략. 뭔 얘긴지 이해가 잘..

### Tunnel authentication
![image](https://user-images.githubusercontent.com/10507662/112324182-d8605480-8cf5-11eb-9c3d-f8b0b9bceed0.png)

#### security consideration
- tunnel gateway 는 통신하고 있응 protocol 이 tunnel 을 올바른 용도로 사용하고 있는지 검증할 방법이 없음
- tunnel 의 오용을 최소화하기 위해서, gateway 는 HTTPS 443 같이 잘 알려진 특정 port 만을 tunneling 할 수 있게 허용해야 함

## Relay
- HTTP 명세를 완전시 준수하지는 않는 간단한 HTTP proxy
- connection 을 맺기 위한 HTTP 통신을 한 다음, byte 만 보냄
  - 모든 header, method 로직을 수행하지 않음
- keep-alive hang 에 걸릴 위험이 있음, 생략