# 7. Cache
- 성능 개선, 비용 절감
- 캐시 lifecycle, interoperability 에 대해 이야기할 예정
- 다음은 느려지는 주요 원인 4개를 쭉 나열

## 불필요한 데이터 전송
- 같은 문서를 여러 클라이언트에 각각 한 번씩 전송

## 대역폭 (bandwidth) 병목
- 원격 서버 < 로컬 네트워크 클라이언트 (더 넓은 bandwidth 제공)

## 요청량 폭증 (Flash Crowds)

## 거리로 인한 지연

## hit and miss
### Revalidation
- contents 가 변경될 수 있기 때문에 up-to-date 여부를 체크
- copy (사본) 의 revalidation 이 필요할 때, original server 에 작은 요청을 보내고
  - 변경되지 않았다면 `304 Not Modified`
  - revalidate or slow hit
- `If-Modified-Since`

#### Revalidate hit
`304 Not Modified`

#### Revalidation miss
contents 전체와 HTTP 200 OK

#### Object deleted
`404 Not Found`

### Hit rate
- web transaction 을 외부로 내보내지 않았는지에 대한 비율

### Byte hit rate
- document 마다 크기가 다르므로 Byte 단위의 hit rate 를 측정하려는 사람들도 있음
  - 요금을 매기려는 사업자들
- 캐시를 통해 제공된 모든 byte 의 rate 를 표현
- Byte hit rate 100% - 모든 byte 가 cache 에서 왔으며, 어떤 트래픽도 인터넷으로 나가지 않았음

### hit 와 miss 의 구별
- Client 는 cache 인지 original 인지 구별하지 못함
- commercial proxy cache 중 `Via` 를 붙이는게 있고
- `Date` 헤더를 보고 판단할 수 있음

## Cache topology
### Private cache
- 브라우저 단 캐시 (disk, memory)

### Public proxy cache
![image](https://user-images.githubusercontent.com/10507662/111899514-5d3a3c80-8a70-11eb-9b4e-4cb088312c7d.png)
- public 상태의 공용 캐시에서 자주 찾는 객체를 1번만 가져와서 사본으로 저장하여 제공
- X 는 css, js 등을 이렇게 제공

### Proxy cache hierarchy
- small cache 에서 miss 가 발생했을 때 parent cache 가 트래픽을 처리

### Cach meshes, content routing, peering
- 몇 네트워크 아키텍쳐는 단순한 hierarchy 대신 복잡한 cache mesh 를 만듦
- parent cache 와 소통할 것인지 or original server 로 바로 가도록 할 것인지 동적으로 선택

#### 할 수 있는 일
- URL 기반으로 parent cache or original server 를 동적으로 선택
- URL 기반으로 특정 parent cache 를 동적으로 선택
- parent cache 에 가기 전에, cached copy 을 로컬에서 찾아봄
- 다른 cache 들이 cached content 를 접근할 수 있도록 허용하되, cache 를 통한 Internet transit 은 허용하지 않음
  - Internet transit: 트래픽이 다른 네트워크로 건너가는 것

## Cache processing steps
### 1) Receiving
들어오는 데이터를 읽음

### 2) Parsing
request message 를 파싱하여 header 를 다루기 쉬운 자료 구조에 담음

### 3) Lookup
- cache URL 을 알아내고 local copy 가 있는지 검사 (memory or disk or another nearby computer)
- 없다면 original server or parent cache 에서 가져옴, or failure
- cached object 는 다음을 가지고 있음
  - server response body
  - original server response header
  - 얼마나 오랫동안 cache 에 머무르고 있었는지를 알려주는 기록, 자주 사용되었는지 등에 대한 메타데이터 등을 포함

### 4) Freshness check
- 검사 규칙은 뒤에서 설명

### 5) Response creation
- cache 를 original server 에서 온 것처럼 보이게 cached server response header 를 기반으로 생성
- cachec freshness information (Cache-Control, Age, Expire header) 를 포함해야 함
- Date 를 조정해서는 안됨

### 6) Sending
### 7) Logging
- log file, cache usage 에 대한 통계를 유지
- cache trasaction 이 끝난 후 cache hit/miss 횟수에 대한 통계 갱신, request type, what happened 를 저장
- `Netscape extended common log format` - 가장 많이 쓰이는 포맷

### Cache processing flow chart
![image](https://user-images.githubusercontent.com/10507662/111899886-ab503f80-8a72-11eb-94fa-50f2f7659cf0.png)

## Keeping Copies Fresh
### Document expiration
`Cache-Control`, `Expires` 로 original server 가 유효기간을 붙일 수 있음  
![image](https://user-images.githubusercontent.com/10507662/111899921-f10d0800-8a72-11eb-91bc-212b6149d6ba.png)

### Expiration Dates and Age
#### Cache-Control:max-age (HTTP/1.1)
- maximum age of the document 
- 처음 생성된 이후부터 경과한 시간의 최댓값 (seconds)
- `Cache-Control: max-age=123450`

#### Expires (HTTP/1.0+)
- absolute expiration date
- `Expires: Fri, 05 Jul 2002, 05:00:00 GMT`

### Server Revalidation
- cache 가 original server 에게 문서가 변경되었는지 확인하는 것
- contents 가 변경되었다면 새로운 copy 를 가져와 저장, Client 에게 보냄
- 변경되지 않았다면 새로운 expiration date 를 포함한 새 헤더들만 가져와서 갱신

#### HTTP protocol cache requirements
- A cached copy is fresh enough
- A cached copy that has been revalidated with the server to ensure it's still fresh
- An error message, if the origin server to revalidate with is down
  - original server 에 접근할 수 없다면 에러 반환
- A cached copy, with an attached warning that it might be incorrect 

### Revalidation with Conditional Methods
#### If-Modified-Since (IMS)
- Last-Modified response header 와 같이 사용됨
- 특정 날짜 이후로 수정되었다면 request method 처리
- 변경되지 않았다면 `304 Not Modified` 리턴, body 는 보내지 않음
- `If-Modified-Since: <date>`

#### If-None-Match
![image](https://user-images.githubusercontent.com/10507662/111901031-038a4000-8a79-11eb-9e7b-a65d7c9d05cd.png)

### Weak and Strong Validators
#### Weak validator
- cached copy 를 invalid 처리하지 않고 살짝 고치는 것을 허용하고 싶을 때
- contents 가 조금 변경되었더라도 같은 것이라고 서버가 주장할 수 있도록 해주는 validator

#### Strong validator
- 무조건 변경

#### 구분?
`W/` prefix 로 구분
```
ETag: W/"v.2.6"
If-None-Match: W/"v2.6"
```

### 언제 Entity Tag 를 사용하고 언제 Last-Modified Date 를 사용?
- HTTP/1.1 Client 는 server 가 entity tag 를 반환했다면 Entity Tag Validator 를 사용해야 함
- Last-Modified 값만 반환했다면 If-Modified-Since validation 을 사용할 수 있음

## Controlling Cachability
### no-cache, no-store
```
Cache-Control: no-store
Cache-Control: no-cache
Pragma: no-cache
```
- freshness 를 관리하기 위해 HTTP/1.1 에서 cache 를 제한하거나 cached object 를 제공하는 여러 방법 제공
- cache 가 검증되지 않은 cached object 로 응답하는 것을 막음

#### no-store
cache 가 response copy 를 만드는 것을 금지

#### no-cache
- local cache storage 에 저장될 수 있으나, server revalidation 을 거치치 않고는 cache 에서 client 로 제공될 수 없음
- Do-Not-Serve-From-Cache-Without-Revalidation
- `Pragma: no-cache` 는 HTTP/1.0+ 하위호환성을 위함

### Max-Age
- 위에서 설명
- s-maxage: 동일하지만 shared(public) cache 에만 적용

### Expires

### Must-Revalidate
- cache 는 성능을 개선하기 위해 fresh 하지 않은 객체를 제공하도록 설정할 수 있음
```
Cache-Control: must-revalidate
```
- revalidation 없이는 cache 를 제공해서는 안된다는 의미
- original server 를 접근할 수 없다면 `504 Gateway Timeout error` 를 반환해야 함

### Heuristic Expiration
- max-age 나 Expires 가 제공되지 않은 경우
- heuristic (경험적인 방법으로) max age 계산
- 계산 결과가 24시간보다 크면 Heuristic Expiration Warning (Warning 13) 헤더가 response header 에 추가되어야 함
  - 브라우저에서는 볼 수 없음
- 계산으로는 **`LM-Factor algorithm`** 이 유명함
  - cached document 가 매우 이전에 변경됐다면, 안정적인 문서이고 바뀔 가능성이 크지 않음 -> 더 보관해도 안전함
  - cached document 가 최근에 변경되었다면, 자주 변경될 것 -> server 와 revalidation 하기 전까지 짧은 기간 동안만 cache

### Client Freshness Constraints
- 브라우저 단 강제 갱신

### Cautions
- document expiration 은 완벽한 시스템이 아님
- 많은 publisher 가 이 기간을 길게 잡지 않음

캐시 계산 알고리즘 등은 생략