# 11. Client Identification and Cookies
- 사용자 식별하는 유형들
## HTTP Headers
|name|type|description|
| ----------- | ----------- | ----------- |
|From|request|email address|
|User-Agent|request|browser|
|Referer|request|Page user came from by following link|
|Authorization|request|username, password|
|Client-ip|extension (request)||
|X-Forwarded-For|extension (request)||
|Cookie|extension (request)|Server-generated ID label|

## Client IP Address
초창기에는 CLient IP 로 사용자를 식별하려고 했으나

1. 사용자가 아닌 컴퓨터를 가리킴
2. ISP 에서 동적으로 IP 할당
3. NAT (Network Adress Translation) 장비를 통하면 해당 장비의 IP 만 나타남
4. HTTP proxy, gateway 는 original server 에 새로운 TCP 연결을 맺음 -> server 가 client 가 아닌 proxy, gateway server 의 IP 를 봄

## User Login
- username, password 로 인증을 요구하여 명시적으로 누군지 알 수 있게 요청할 수 있음

## Fat URLs
- 사용자의 상태 정보를 URL 에 포함

### problems
#### ugly URLs
사용자 혼란 야기

#### URL 공유 불가능
특정 사용자와 세션에 대한 상태 정보 포함 -> 개인정보를 공유하는 꼴

#### cache 사용 불가능
#### server 부하 가중
#### 이탈
- session 정보가 추가된 링크만을 사용해야 잘 동작하는데
- 다른 URL 로 이탈하기 쉽고, 그러면 정보 연속성이 없음

#### Not persistent across sessions
- URL 을 저장하고 있지 않는 이상, 로그아웃하면 모든 정보가 날아감

## Cookie
- 사용자를 식별하고 session 을 유지하는 방식 중에서 현재까지 가장 널리 사용하는 방식

### Type of cookies
- session cookie: 관련 설정과 perference 들을 저장하는 임시, 브라우저 종료 시 제거
- persistent cookie: disk 에 저장, 삭제되지 않음 (주기적으로 방문하는 사이트에 대한 설정 정보, login name)
- `Expires` 나 `Max-Age` parameter 가 없는 경우 session cookie

### cookie 의 동작
- `name=value` 형태의 리스트를 가지고, `Set-Cookie` / `Set-Cookie2` 같은 response header 에 기술되어 전달됨

### Cookie Jar: Client-side state
- browser 가 server 관련 정보를 저장하고, client 가 server 에 접근할 때마다 그 정보를 함께 전송하는 것이 기본 개념
- browser 는 cookie 정보를 저장할 책임이 있음 - **client-side state**
  - browser 마다 구현 방식이 다름
  - chrome 은 SQLite file, ...

#### Different Cookies for Different Sites
- cookie 를 생성한 server 에게만 cookie 에 담긴 정보 전달
  - 가진 cookie 를 모두 전달 시 성능 저하, 대부분의 사이트에는 의미없는 값, 잠재적인 개인정보 문제 등

##### Cookie Domain attribute
```
Set-cookie: user="mary17"; domain="naver.com"
```

##### Cookie Path attribute
```
Set-cookie: pref=compact; domain="naver.com"; path=/my/
```

### Cookie Ingredients
#### Version 0
- netscape cookie 라고 불림

#### Version 1
- Version 0 의 확장이나 잘 안 쓰임

### Version 0 (netscape) cookie
```
Set-cookie: name=value; [; expires=date] [; path=path] [; domain=domain] [; secure]

Cookie: name1=value1 [; name2=value2] ...
```

|Set-Cookie attribute|description|
| ----------- | ----------- |
|name=value|" 로 감싸지 않은 세미콜론, 쉼포, 등호, 공백을 포함하지 않는 문자열|
|expires|lifecycle 을 가리키는 문자열<br>`expires= Wednesday, 09-Nov-99 23:12:40 GMT`|
|domain|cookie 전송 범위 제한, 2 ~ 3개 영역의 도메인을 가져야 함|
|path|cookie 전송 범위 제한|
|secure|HTTP 가 SSL 연결을 사용할 때만 쿠키를 전송|

### Version 1
생략

### Cookie 는 session tracking 에 사용할 수 있다

### Cookie and caching
- 이전 사용자의 쿠키가 다른 사용자에 할당되는 등으로 개인 정보가 노출될 수 있음
- 아래는 기본 원칙

#### 캐시되지 말아야 할 문서가 있다면 표시
- document 가 `Set-Cookie` 를 제외하고 캐시를 해도 된다면 명시적으로 표시
```
Cache-Control: no-cache="Set-Cookie"
```

#### Set-Cookie header 를 cache 하는 것에 유의
- ?
- response 에 Set-Cookie header 가 있다면, document 는 cache 할 수 있지만
  - Set-Cookie header 를 cache 하는 것에는 주의해야함
  - 같은 Set-Cookie header 를 여러 사용자에게 보내면, 사용자 추적에 실패
- cache 가 모든 요청마다 original server 와 revaldate 하도록 할 수 있음
```
Cache-Control: must-revalidate, max-age=0
```

#### Cookie header 를 가진 request 를 주의
- 개인정보를 담고 있을 수 있음

### Cookie, Security and Privacy
- 보안상으로 엄청 위험한건 아님
- 개인정보 정책에만 유의한다면, cookie 에 관련한 위험성보다 session 조작이나 transaction 상의 편리함이 큼
- 1998년에 cookie 에 대한 위험성이 과대평가됐다는 보고서도 있음

# 12. Basic Auth
## Authentication
### HTTP's Challenge/Response Authentication Framework
![image](https://user-images.githubusercontent.com/10507662/112994171-f1a94b00-91a4-11eb-8a4b-84a34f994155.png)

### Authentication Protocols and Headers
|phase|headers|description|method/state|
| ----------- | ----------- | ----------- | ----------- |
|request||-|GET|
|challenge|WWW-Authenticate|username 과 password 요구|401 Unauthorized|
|Authorization|Authorization|username, password 기술|GET|
|Success|Authentication-Info|성공, 간혹 header 에 session 에 대한 추가 정보가 있는 경우도 있음|200 OK|

### Security Realms (영역)
- 보호된 문서를 security realm 그룹으로 나눔
- 다른 사용자 권한 요구

```
HTTP/1.0 401 Unauthorized
WWW-Authenticate: Basic realm="Corporate Financials"
```

## Basic Auth
### 과정
![image](https://user-images.githubusercontent.com/10507662/112995457-3681b180-91a6-11eb-87ea-bf57c88ca1ee.png)

### Security Flaws (보안 결함)
1. base-64 encoding 방식은 decoding 이 쉬워 정보가 그대로 노출
2. 예기치 않은 부분을 수정하여 transaction 의 본래 의도를 변경하는 proxies, intermediaries 가 개입하는 경우 정상 동작을 보장하지 않음
3. counterfeit (fake) server 에 취약

13장은 digest authentication 에 대한 이야기가 있으나, 현재에도 사용하지 않는 방식이며
OAuth 를 주로 사용하므로 생략