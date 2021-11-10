# 5. Kafka Internals
- replication 이 동작하는 방법
- Kafka producer 와 Consumer request 를 처리하는 방법
- Storage 를 처리하는 방법

## Cluster Membership
### Zookeeper
- Zookeeper 는 보통 파일 시스템처럼 hierarchical tree 구조
  - znode: 저장하는 node, 이름 앞에는 `/`.
    - 디렉토리처럼 path 사용하여 node 위치 식별
- 각 node 에는 상태, 구성 정보, 위치 정보 등의 작은 데이터, 메모리에 저장 (1KB 미만)
- ephemeral node: client 가 연결되어 있을 때만 존재, 끊어지면 제거
- persistent node: client 가 삭제하지 않는 한 계속 보존
- `Watch`: node 의 상태 모니터링
  - client 가 특정 node watch 설정 시, 변경이 일어난 경우 callback 으로 client 에게 알려줌

### kafka 와 zookeeper
- zookeeper 를 이용하여 broker 들의 metadata 유지 관리
- 예시
  - /kafka-main
    - /kafka-main/controller
    - /kafka-main/brokers
      - /kafka-main/brokers/ids
        - broker 가 뜰 때 자동 생성하여 ephemeral 저장하며, 같은 아이디로 새로 추가하려고 하면 에러
      - /kafka-main/brokers/topics
    - /kafka-main/config

#### broker 중단 시
1. 해당 broker 의 zookeeper node 는 삭제됨
    - 해당 broker id 는 다른 데이터들에 존재 (topic replica 내역 등)
2. 완전히 새로운 broker id 를 갖는 broker 시작
3. 누락된 broker 에 할당되었던 partition 과 topic 을 갖고 클러스터에 합류

## Controller
- **partition leader 를 선출하는 책임을 갖는 broker**
- cluster 에서 시작하는 첫 번째 broker
  - 여기서 `/controller` 를 생성
  - 다른 broker 에서도 시도하나, 이미 존재하므로 실패. 다른 broker 들은 controller 를 Watch 하는 상태임

### controller 가 내려가면?
#### 선정
- cluster 내에서 `/controller` 를 선점하는 broker 가 새로 차지
- 새로 선정 시 새로운 **controller epoch** 를 받음 (번역이 세대 번호로 되어 있음..)
- 나머지 broker 들도 현재의 controller epoch 를 알게 되며, 이전 세대의 controller epoch 를 받으면 무시

#### 재할당
- 기존 controller 가 leader 로 갖고 있던 partition 들에 새로운 leader 가 필요함
- 새로운 controller 가 해당 partition 들을 점검하고, 새로 리더가 될 broker 결정
- 새로운 leader/follower 정보를 모든 broker 들에 전송
  - leader partition 들은 producer/consumer 요청 처리를 시작
  - follower 는 새로운 leader 의 메시지 복제

### 새 broker 가 추가되면?
- broker id 를 사용하여 그 broker 의 replica 로 사용할 broker 가 있는지 확인
- 있으면 새 broker 와 기존 broker 모두에게 변경 사항을 알리고, 새 broker 의 replica 들은 기존 leader 의 메시지 복제 시작

## Replication
- topic 이 있고 이게 여러 partition 에 저장될 수 있음
- 각 partition 은 다수의 replica 를 가질 수 있음

#### Leader replica
- 일관성을 보장하기 위해 모든 producer/consumer client request 를 리더를 통해 처리

#### Follower replica
- leader 를 제외한 나머지, client request 를 처리하지 않음
- leader 의 메시지 복제
- leader 가 중단되는 경우 나머지 중 하나 leader 로 선정

### 상세
- replica 들은 leader 에게 `Fetch` request (받고자하는 offset)

#### 수신된 순서대로 처리
- follower -> leader 1, 2, 3 을 요청
  - 4 를 요청할 때는 follower 가 3을 받았음을 leader 가 알 수 있음
  - 복제 지연이 어느 정도인지 알 수 있음
- follower 가 10초 이상 요청이 없거나, 10초 간에도 가장 최근의 메시지를 복제하지 못하면 -> **out-sync**
  - leader 장애 시 해당 follwer 가 leader 로 선출될 수 없음
- **in-sync replica (ISR)**: 최신 메시지를 계속 요청하는 follower replica
- 시간은 `replica.lag.time.max.ms` 로 제어

#### preferred leader
- topic 이 생성될 때 각 파티션의 leader 였던 replica
- 파티션을 처음 생성할 때는 여러 브로커가 고르게 할당받으므로, 여기서 설정된 leader 가 preferred 임
- `auto.leader.rebalance.enable=true`
  - preferred leader replica 가 현제 leader 가 아닐 경우, in-sync 인지 확인 -> leader 선출 시 이걸 leader 로 선정

> preferred leader 는 `kafka-topics.sh` 에서 처음 나오는 replica

## Request processing
- client 와 partition replica, controller 로부터 partition leader 에게 전송되는 요청 처리
- TCP, binary protocol
  - 언어별로 제공되는 client API 로 요청
- client -> broker 요청은 수신된 순서대로 처리, 순서 보장

### Standard header ([link](https://kafka.apache.org/protocol.html))
- Request type (also called API key), 16 bit
- Request version (so the brokers can handle clients of different versions and respond accordingly), 16 bit
  - 브로커가 kafka client 버전을 보고 그에 맞게 응답 내려줌
- Correlation ID (cID), 32 bit
  - 사용자가 지정한 값, trouble shooting 시 유용. 에러 발생 시 같이 로깅됨
- Client ID
  - client application 식별

### broker 단 과정
![image](https://user-images.githubusercontent.com/10507662/118360059-f1340a80-b5c0-11eb-9fe3-35ddff01dc8b.png)

- `acceptor` thread: listen. 연결 생성 및 processor 로 넘김
- `processor` (network) thread: request queue 에 넣기 + response queue 에서 가져와서 응답 전송. 개수 조정 가능
- `IO` thread: request queue 에서 가져와서 처리
- 가장 많은 타입
  - Produce request: producer 가 broker 에게 쓰기
  - Fetch request: consumer/follower replica 가 메시지 읽을 때
- 모든 요청은 partition leader replica 가 받아야 함

#### client 가 parititon leader 가 누군지 어떻게 알 수 있는지?
- kafka client 의 `metadata request`: 응답으로 topic 의 partitions, partition replica, leader 정보 반환
- client 단 cache timeout: `metadata.max.age.ms`
- broker 추가로 <ins>Not a Leader for Partition</ins> 를 받은 경우, 새로 metadata request 를 보냄

### Produce request
#### acks
- 0: broker 응답 기다리지 않음
- 1: leader 가 메시지를 받았는지 확인
- all: 모든 in-sync replica 가 받았는지

#### request 처리 과정
1. 확인
    - topic write 권한을 가졌는지
    - `acks` 값이 유효한지
    - `acks=all` 이면 충분한 in-sync replica 가 있는지
2. local disk 에 write
    - linux 의 경우 file system cache 에 쓰고, 언제 disk 에 쓰는지는 모름
    - disk write 를 기다리지 않고 replication 에 의존함
3. leader 에 쓴 후 acks 확인
    - 0, 1 이면 바로 응답
    - all 이면 follower 까지 완료됐는지 leader 가 확인할 때까지 **purgatory** (catholic 관런 단어) 라는 buffer 에 요청 저장

### Fetch request
- partition 에서 제거된 메시지나, 없는 offset 을 요청하면 에러 응답
- `zero-copy`: file 을 중간 buffer 에 쓰지 않고 바로 네트워크 채널로 전송
- data 크기 관련
  - broker 는 upper boundary 를 지정할 수 있음
  - client 는 lower boundary 를 지정할 수 있음
    - 데이터가 적은 topic 을 계속 호출하면 CPU/네트워크 비용 소모
    - 지정한 크기 만큼이 채워졌을 때 broker 가 반환하도록 설정
    - timeout 설정하여 무한정 대기하지 않을 수 있음

#### partition leader 의 모든 데이터를 읽을 수는 없음
- in-sync replica 에 쓴 메시지들만 읽을 수 있음
- 완료되지 않은 경우 empty resposne
- 왜?
  - replica 에 복제되지 않은건 unsafe 로 봄
  - leader 가 중단되어 다른 replica 가 leader 가 되면, 그 메시지는 더 이상 존재하지 않는 것이 됨 -> **일관성이 없어짐**

### Other requests
- `LeaderAndIsr`: 새로운 leader 가 선출된 것을 controller 가 알릴 때
- `CreateTopic`, `ApiVersion` 등

## Physical storage
- `logs.dirs`: 파티션이 저장될 디렉토리

### Partition allocation
- 기본적으로 round-robin 이나, rack-aware 인 경우 rack 정보를 포함하여 분산
- rack: 여러 대의 서버를 같이 모아둔 것

#### 예시
broker 6개, 10개 partition, replication factor 3
- broker 당 5개 replica (partition replica 를 고르게 분산)
- leader 와 follower partition replica 분산
- rack-aware 하다면, 서로 다른 rack 으로 분산

![image](https://user-images.githubusercontent.com/10507662/118361762-c39e8f80-b5c7-11eb-8b08-6e30de47ff3e.png)

### File management
`retention` (보존)

- 각 partition 을 **segment** 로 나눔
  - 최대 1GB or 1주일 동안 데이터 보존
- active segment: 메시지를 쓰기 위하 사용 중인 segment

### File format
![image](https://user-images.githubusercontent.com/10507662/118361885-3b6cba00-b5c8-11eb-9d76-c307395d565d.png)
- 일반 메시지와 (압축된) wrapper 메시지

```
bin/kafka-run-class.sh kafka-.tools.DumpLogSegments
```
- file system 의 segment 와 내용을 자세히 볼 수 있음

### Index
- 특정 offset 에서 메시지를 읽을 수 있게 함
- 이것도 segment 로 분할됨

### Compaction
- 같은 key 를 갖는 메시지들이 topic 에 여러 번 저장됐을 때 처리하는 방법
  - compression 과는 다른 개념
- `delete`: 보존 기간 이전 메시지 삭제
- `compact`: 각 key 의 가장 최근 값만 topic 에 저장

### How compaction works
- clean: 압축된 데이터
- dirty: 압축 이후에 새로 들어온 데이터
- 키워드만 정리
  - 1 compack manager thread, multiple compack thread

### Deleted message
- 개인정보 등으로 kafka 에서 아예 날려야 할때
- key 를 설정하여 value = null