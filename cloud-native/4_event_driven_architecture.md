# 4. 이벤트 기반 마이크로서비스: 단순히 request/response 만을 의미하지 않는다

각각의 컴포넌트를 너무 단단히 or 너무 일찍 결합해서 monolith 를 다시 만들지 않도록 주의해야 한다  
여기부터 다루는 패턴은 민첩성과 탄력성 (agility and resilience) 을 극대화하는 방식을 동시에 제공하는 독립된 컴포넌트들의 집합, 견고하고 날렵한 소프트웨어를 만들기 위해 설계

request/response 와 event-driven 사고는 근복적으로 다르다

## (일반적으로는) 명령형 프로그래밍 (imperative programming) 을 배운다
- 프로그램이 어디로 흘러가는지 확실히 볼 수 있다 -> 자연스러운 request/response 방식의 생각
- 명령에 대한 특정한 응답을 기대 + 단일 프로세스 실행 환경에서는 정상적으로 동작

하지만 소프트웨어는 더 이상 단일 프로세스가 아님 = 분산, 끊임없는 변화  
클라이언트는 React.js 등의 프레임워크가 있지만, 서버 단은 아직도 뭐가 없음

한 사용자가 넷플릭스에 접속할 때만 하더라도, (다수의) following MSA 에 대한 중요한 fan-out request 가 발생함

## event-driven computing 재도입
event-driven 에서는 코드 실행을 일으키는 엔티티가 어떤 형태의 response 도 기다리지 않음 (발행 후 망각)  
-> 실행은 어떠한 결과를 가짐

#### 145p 의 다이어그램
1. request/reponse: 2개의 참여자가 긴밀하게 의지
2. event-driven: 1개의 참여자가 결과를 내지만, **누가 발생시켰는지는 중요하지 않음 = 의존성이 낮음**
    - 하나의 이벤트와 그 결과는 완전히 단절됨

## CQRS (Command Query Responsibility Segregation). 명령 쿼리 책임 분리
이 책에선 컨트롤러 단 까지 read, write 를 구분

read, write controller 에서 사용되는 모델은 각각 다르다  
둘 다 있는 필드도 있지만, 어떤건 한 쪽에서만 의미있음
- 이해하기 쉽고, 유지보수가 편리한 코드 작성
- 버그 발생 가능성도 줄일 수 있음

#### 읽기 로직 (Query) 으로부터 쓰기 로직 (Command) 를 분리하자
- 책의 예시에서 Connections'Post 서비스의 경우
    - request/response 에서는 command 쿼리가 없다
    - stateless 서비스이므로 다른 두 서비스의 데이터를 집계해서 결과를 내기 위해 command 가 필요하지 않음
- command 처리에서 query 를 분리하면 -> 양쪽에서 서로 다른 프로토콜을 사용할 수 있음
- 비동기 or FaaS (Function) 의 구현

## 다른 스타일, 유사한 도전 과제
cloud 의 특징이고, 전부 만족한다면 request/response 와 event-driven 중 하나를 선택할 수 있다
1. 어떤 네트워크 파티션도 서로 다른 MS 를 차단하지 않는다
2. 내가 팔로우하는 개인 목록을 생성하는 데 예상치 못한 지연 시간은 없다
3. 서비스를 실행하는 모든 컨테이너는 안정된 IP 주소를 유지한다
4. credential 이나 인증서를 교체할 필요가 없다

request/response 의 경우 특정 서비스에 접속할 수 없으면 재시도를 해볼 수 있음

event-driven 에서는 비동기 방식이므로, 네트워크 파티션 간에 처리하는 이벤트를 보관하기 위해 RabbitMQ 나 Apache Kafka 같은 메시징 시스템을 구현