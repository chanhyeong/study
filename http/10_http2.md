# 10. HTTP/2.0
- 원서에는 없는 내용

## 등장 배경
- HTTP/1.1 은 단순성, 접근성에 맞춰 최적화
- 2009년 구글이 발표한 SPDY protocol 기반
  - header 압축 - 대역폭 절약
  - Multiple requests per 1 TCP connection - reduce latency
  - resource push
- 2012년 10월부터 SPDY 기반으로 HTTP/2.0 protocol 설계

## 개요
- HTTP/2.0 request, response 는 fixed length (16,383 byte) 1개 이상의 frame 에 담김
  - header 는 압축
- frame 에 담긴 request, response 는 stream 을 통해 보내짐
- 하나의 connection 위에 여러 개의 stream 이 동시에 만들어질 수 있음

## HTTP/1.1 과의 차이점
### Frame
- [RFC 7540](https://tools.ietf.org/html/rfc7540#section-4) 에서 발췌
- 10가지 frame 이 있으며 자세한건 명세 참조
```
+-----------------------------------------------+
|                 Length (24)                   |
+---------------+---------------+---------------+
|   Type (8)    |   Flags (8)   |
+-+-------------+---------------+-------------------------------+
|R|                 Stream Identifier (31)                      |
+=+=============================================================+
|                   Frame Payload (0...)                      ...
+---------------------------------------------------------------+
```
#### Length
- The length of the frame payload expressed as an unsigned 24-bit integer.
- Values greater than 2^14 (16,384) MUST NOT be sent unless the receiver has set a larger value for SETTINGS_MAX_FRAME_SIZE.
- The 9 octets of the frame header are not included in this value.

#### Type
- The 8-bit type of the frame.

#### Flags
- An 8-bit field reserved for boolean flags specific to the frame type.
- Flags are assigned semantics specific to the indicated frame type.
  - frame type 에 따라 flag 의미가 결정된다
- Flags that have no defined semantics for a particular frame type MUST be ignored and MUST be left unset (0x0) when sending.

### R
- A reserved 1-bit field.
- The semantics of this bit are undefined, and the bit MUST remain unset (0x0) when sending and MUST be ignored when receiving.

### Stream Identifier
- A stream identifier expressed as an unsigned 31-bit integer.
- The value 0x0 is reserved for frames that are associated with the connection as a whole as opposed to an individual stream.

## stream, multiplexing
- **stream**
  - HTTP/2.0 connection 을 통해 client - server 간 교환되는 frame 들의 독립된 양방향 시퀀스
- HTTP/1.1 에는 1 tcp connection 을 통해 request ~ response 가 완료되어야 같은 tcp connection 으로 보낼 수 있음
- HTTP/2.0 에서는 1 tcp connection 에 multiple stream 이 동시에 열릴 수 있음

### header compression
- HPACK 명세에 정의된 방법으로 압축 -> header block 으로 쪼개져서 전송
- 초기에는 deflate 알고리즘을 썼으나, 보안 문제로 변경

## Server push
- 생략