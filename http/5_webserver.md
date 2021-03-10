# 5. 웹 서버
## 다양한 웹서버
- HTTP request 를 처리하고 response 제공

### 구현
- HTTP protocol 구현
- 웹 리소스 관리
- 웹 서버 관리 (설정, 통제, 확장하기 위한 기능)
- TCP 커넥션 관리에 대한 책임을 운영체제외 나눠 가짐
- OS: 하드웨어 관리, TCP/IP 네트워크 지원, 리소스 유지를 위한 filesystem, process management 제공
- 형태
  - 설치형
  - 임베디드 기기 내장 (공유기 자체의 관리 기능 등)

## 간단한 Perl 웹 서버
```perl
use Socket;
use Carp;
use FileHandle;

# (1) 8080 이 디폴트, 명령어로 변경 가능
line 
$port = (@ARGV ? $ARGV[0] : 8080);

# (2) 로컬 TCP 소켓 생성, listen 설정
connections
$proto = getprotobyname('tcp');
socket(S, PF_INET, SOCK_STREAM, $proto) || die;
setsockopt(S, SOL_SOCKET, SO_REUSEADDR, pack("l", 1)) || die;
bind(S, sockaddr_in($port, INADDR_ANY)) || die;
listen(S, SOMAXCONN) || die;

# (3) 시작 메시지 출력
printf(" <<<Type-O-Serve Accepting on Port%d>>>\n\n",$port);

while (1)
{
 # (4) 커넥션 C 를 기다림
 $cport_caddr = accept(C, S);
 ($cport,$caddr) = sockaddr_in($cport_caddr);
 C->autoflush(1);

 # (5) 커넥션 host 출력
 $cname = gethostbyaddr($caddr,AF_INET);
 printf(" <<<Request From '%s'>>>\n",$cname);

 # (6) 빈 라인이 나올 때까지 읽어서 출력
 while ($line = <C>)
 {
 print $line;
 if ($line =~ /^\r/) { last; }
 }

 # (7) response message 를 위한 prompt 를 만들고 response 를 입력 받음
 # . 하나만 있는 라인이 들어올 때까지 반복
 printf(" <<<Type Response Followed by '.'>>>\n");

 while ($line = <STDIN>)
 {
 $line =~ s/\r//;
 $line =~ s/\n//;
 if ($line =~ /^\./) { last; }
 print C $line . "\r\n";
 }
 close(C);
}
```

## 웹 서버가 하는 일
![image](https://user-images.githubusercontent.com/10507662/110622189-7c1e1080-81de-11eb-9ced-5a5a46599f72.png)

1. connection 맺음 - 클라이언트 접속을 받거나 닫음
2. request 받음 - HTTP request message 를 네트워크에서 읽어들임
3. request 처리 - request message 를 해석하고 행동
4. resource 접근 - message 에서 지정한 리소스 접근
5. response 생성 - 올바른 헤더를 포함한 HTTP response message 생성
6. response 전송 - response 를 클라이언트에 돌려줌
7. transaction 을 로그로 남김 - 트랜잭션 완료에 대한 기록을 남김

## Step 1) 클라이언트 커넥션 수락
- 이미 Persistent connection 을 갖고 있다면 사용
- 아니라면

### Handling New Connections
- 순서
  - 웹 서버가 클라이언트의 TCP 커넥션 요청을 받아 맺음
  - TCP 커넥션에서 IP 주소를 추출하여 어떤 클라이언트가 있는지 확인
  - 서버는 새 커넥션을 커넥션 목록에 추가하고, 커넥션에서 오가는 데이터를 지켜보기 위한 준비
- 웹 서버는 어떤 커넥션이든 마음대로 거절하거나 즉시 닫을 수 있음

### Client Hostname Identification
- 대부분의 웹서버는 reverse DNS 를 통해 Client IP 주소를 hostname 으로 변환하도록 설정되어 있음
- access control, logging 을 위해 사용될 수 있으나
- hostname lookup 은 시간을 잡아먹어 트랜잭션을 느리게 할 수 있음
- 대부분 hostname resolution 을 꺼두거나 특정 컨텐츠에만 켜놓음

### ident 를 통해 클라이언트 사용자 알아내기
- IETF ident protocol
  - 어떤 username 이 HTTP connection 을 initialize 했는지 찾아낼 수 있게 함
  - `identd`
- 거의 사용하지 않음, 포트는 113
  - daemon 켜두지 않음, 방화벽이 막음, 지연, 조작이 쉬움, virtual IP 미지원, privacy 침해 등

## Step 2) Receiving Request Messages
- request line 을 파싱하여 method, URI, version number 를 찾음. 끝 라인은 CRLF 로 종료
- message header 들을 읽음, 각 헤더는 CRLF
- header 의 끝을 의미하는 CRLF 만 있는 라인 찾음 (if exists)
- body 가 있다면 읽어 들임 (길이는 Content-Length 헤더로 정의)

### Internal Representations of Messages
- request message 를 쉽게 다룰 수 있도록 내부 자료 구조로 저장
  - ex) header 를 속도가 빠른 lookup table 에 저장, 빠르게 접근

### Connection Input/Output Processing Architectures
![image](https://user-images.githubusercontent.com/10507662/110627972-e8504280-81e5-11eb-999d-85f830d025cd.png)

#### single threaded
- 1 request at a time

### multiprocess and multithreaded
- multiple request
- thread/process 관리

### Multiplexed I/O
- 모든 커넥션은 동시에 그 활동을 감시당한다
  - all the connections are simultaneously watched for activity
- 커넥션의 상태가 바뀌면 처리가 수행 -> 완료되면 커넥션 목록으로 돌아감
- thread/process 가 유휴 상태의 connection 을 기다리느라 리소스를 낭비하지 않음

### Multiplexed multithreaded
- 다수의 CPU 를 활용하기 위해 multithreading + multiplexing 을 결합
- 여러 thread 가 각각 열려있는 커넥션을 감시하고, 각 커넥션에 대한 작업 수행

## Step 3) Processing Requests
- request 에서 method, resource, header, body 를 얻어서 처리
- 나중에 이 처리 과정이 전부 기술될 예정

## Step 4) Mapping and Accessing Resources
- 웹 서버 = 리소스 서버
- 미리 만들어진 컨텐츠 + 애플리케이션을 통해 생성된 동적 컨텐츠 제공

### Docroot
- 웹 서버의 file system 에 있는 특별한 폴더를 예약
- Apache httpd 의 경우: `DocumentRoot`
- nginx 의 경우: `root`

#### Virtually hosted docroot
- IP address 나 hostname 으로 부터 document root 식별
- Apache httpd 의 경우: `VirtualHost`
- nginx 의 경우: `server`

#### User home directory docroots
![image](https://user-images.githubusercontent.com/10507662/110628968-010d2800-81e7-11eb-8567-0fd25c8543c0.png)

- 사용자들이 한 대의 웹 서버에서 각자의 개인 웹사이트를 만들 수 있도록 해주는 것
- `/` 과 `~` 다음에 사용자 이름이 오는 것으로 시작하는 URI
  - 개인 문서 루트를 가리킴
- (개인 의견) 사실 이렇게 주는 경우는 거의 없을 듯

### Directory Listings
- 파일이 아닌 디렉토리를 가리키는 디렉토리 URL 에 대한 요청
- Apache httpd 의 경우: `DirectoryIndex`
- (개인 경험) 느리기도 하고, 서버의 디렉토리 구조를 전부 보여주는 방식이어서 사용하지 않음

### Dynamic Content Resource Mapping
- request 에 맞게 컨텐츠를 생성하는 프로그램에 URI 매핑
- CGI 는 웹 초창기에 많이 쓰임
  - 애플리케이션을 실행하기 위한 간단한 인터페이스
- 요즘에는 (책이 오래되서 이거도 아니지만)
  - Microsoft Active Server Pages
  - Java servlets

### Server-Side Includes (SSI)
- 어떤 리소스가 SSI 를 포함하고 있는 것으로 설정되어 있다면
  - 리소스의 컨텐츠를 클라이언트에게 보내기 전에 처리
- 컨텐츠에 변수명이나 내장된 스크립트가 될 수 있는 특별한 패턴이 있는지 검사 받음
  - 특별한 패턴은 executable 스크립트 출력 값으로 치환
  - dynamic contents 를 만드는 쉬운 방법

### Access Controls
- 각 리소스에 access control 을 할당할 수 있음
- IP 에 대해 접근을 제한하거나 password 요구 등, 12장에서 예정

## Step 5) Building Responses
### Response Entities
- `Content-Type`: response body 의 MIME type 을 서술
- `Content-Length`: 길이를 서술
- 실제 response body

### MIME 타입 결정
#### mime.types
- MIME type 을 나타내기 위해 파일명의 extension 사용할 수 있음

#### Magic typing
- Apache httpd 에서 MIME type 을 알아내기 위해 파일의 내용 검사 -> 알려진 패턴에 대한 테이블에 있는지 확인

#### Explicit typing
- 특정 값을 갖도록 지정

#### Type negotiation
- 1 resource - multiple document formats

### Redirection
#### Permanently moved
- 새 URL 이 부여되어 새로운 위치로 옮겨졌거나, 이름 변경됨
- `301 Moved Permanently`

#### Temporarily moved
- 임시로 옮겨지거나 이름이 변경됨
- 나중에는 원래 URL 로 찾아오고, 북마크도 갱신하지 않기를 원함
- `303 See Other`, `307 Temporary Redirect`

#### URL augmentation (증강)
- 문맥 정보를 포함시키기 위해 다시 작성된 URL 로 redirect
- 서버가 상태 정보를 포함한 새 URL 을 생성 -> 사용자를 이 URL 로 redirect
- 클라이언트가 redirect 를 따라가서, 상태정보가 추가된 완전한 URL을 포함한 요청을 다시 보냄
- **Transaction 간 상태를 유지하는 유용한 방법**
- `303 See Other`, `307 Temporary Redirect`

### Load balancing
- 부하가 덜 있는 서버로 redirect
- `303 See Other`, `307 Temporary Redirect`

#### Server affinity (친밀한 다른 서버가 있을 때)
- 클라이언트에 대한 정보를 갖고 있는 다른 서버로 redirect
- `303 See Other`, `307 Temporary Redirect`

#### Canonicalizing directory names (디렉토리명 정규화)
- directory name 에 대한 URI 요청에서 `/` 를 빼먹은 경우 -> `/` 를 포함한 URI 로 redirect

## Step 6) Sending Responses
- nonpersistent - 메시지 전송 후 커넥션 닫음
- persistent - `Content-Length` 헤더를 명확히 계산하기 위한 주의를 필요로 하는 경우 or 클라이언트가 응답이 언제 끝나는지 알 수 없는 경우 -> 커넥션 유지

## Step 7) Logging
- transaction 이 완료되었을 때 로그 파일에 기록