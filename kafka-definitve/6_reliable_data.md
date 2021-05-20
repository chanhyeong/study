# 6. 신뢰성있는 데이터 전달
## Reliability Guarantees
- 가장 많이 알려진 것은 ACID (atomicity, consistency, isolation, durability)
- kafka 에서 보장하는 것
1. 메시지 순서 보장
    - A, B 순서대로 쓰여지면, 컨슈머도 A, B 순서로 읽음
2. in-sync replica (ISA) 에 썼다면, commit 된 것으로 간주함
3. 최소한 하나의 replica 가 살아있다면, commit 된 메시지는 유실되지 않음
4. 컨슈머는 commit 된 메시지만 읽을 수 있음

아래는 신뢰성을 보장하기 위해 수반되는 trade-off 에 대한 이야기가 나옴

## Replication
replica 가 leader 이거나, 아래와 같은 follower 일 때 in-sync 상태로 간주
- zookeeper session 이 연결되어 있음 (최근 6초)
- 최근 10초 내 leader 로부터 메시지를 읽었음
- 최근 10초 내 leader 로부터 가장 최근 메시지를 읽었음

## Broker Configuration
reliable 한 구성을 위한 3가지 설정변수

### Replication factor
- topic level: `replication.factor`
- broker level: `default.replication.factor` (3)
- N 인 경우 최소 N 개의 broker 구성
- 구성 예
  - 1: broker 하나가 죽으면 아예 사용 불가능
  - 2: broker 하나가 죽어도 동작하지만, 구버전에서는 클러스터가 unstable state 로 되어 다른 broker 를 재시작해야하는 경우
  - 3: 편안

### Unclean leader selection
- broker level: `unclean.leader.selection.enable` (true)
- partition leader 를 사용할 수 없다면, in-sync replica 에서 새로운 리더 선출
  - 유실되지 않음이 보장되므로 **clean leader selection**
- in-sync replica 가 없다면?
  - out-sync replica 를 leader 로 선출하지 않을 경우, leader 가 online 이 될 때까지 offline 상태 (빠른 복구를 보장할 수 없음, ex. 하드웨어 교체)
  - out-sync replica 를 leader 로 선출할 경우
    - 이전에 leader 에 썼던 메시지가 유실되고, consumer 간 일관성도 깨질 수 있음

### Minimum in-sync replicas
- topic, broker level: `min.insync.replicas`
- commit 된 데이터를 하나 이상의 replica 에 확실하게 쓰고자 한다면, 최소 개수를 더 큰 값으로 설정
- 토픽이 3개의 replica 를 가질 때, 이 값을 2로 설정하면
  - 3개의 replica 중에 최소 2개가 in-sync 될 때 토픽의 파티션에 쓸 수 있다
  - 3개 중 2개를 사용할 수 없게 되면, broker 가 write request 를 더 이상 받지 않음
  - consumer 는 읽을 수 있음, 1개의 in-sync replica 는 read-only 상태가 됨

## Reliable system 에서 producer 사용하기
broker 를 잘 구성하더라도 producer 도 잘 구성해야 함
#### 예시1
- 3개의 replica 를 갖는 broker, unclean leader selection false, producer 의 `acks=1`
- leader 가 받고, in-sync 에 쓰지 않은 상태에서 leader down
- 다른 replica 중에 하나가 leader 가 됨 (이 상태에서는 in-sync 상태가 계속 유지됨)
- leader 에 쓰여진 하나의 메시지는 유실됨
#### 에시2
- 3개의 replica 를 갖는 broker, unclean leader selection false
- partition leader 가 방금 down 되어 새로운 leader 가 선출되는 중 -> 유실될 수 있음

### Send acknowledgements
- `acks`
  - 1: leader 선출 중인 경우 `LeaderNotAvailableException`, leader 중단 시점에 follower 에 복제되지 않았다면 유실 가능
  - all: broker 의 `min.insync.replicas` 만큼 전체 수신 확인

### Retry 구성
Producer 가 자동으로 처리하는 에러 or 개발자가 Library 로 직접 처리해야하는 에러
- Retriable error: `LEADER_NOT_AVAILABLE` 등
  - 보통 시간이 지나면 해결되는 것
- non retriable error: `INVALID_CONFIG` 등
- 중복될 수 있음에 유의

#### 추가적인 에러
개발자가 처리해야하는
- Message size, authentication error 등
- broker 전송되기 전 에러. ex) serialization
- 재시도 메시지들을 저장하는 것 때문에 producer 가 사용할 수 있는 메모리가 가득 찼을 때

## Reliable system 에서 consumer 사용하기
### Consumer configuration
1. `group.id`
2. `auto.offset.reset` (earliest, latest)
3. `enable.auto.commit`
4. `auto.commit.interval.ms`

### Consumer 에서 offset commit
#### offset commit 은 항상 메시지가 처리된 후에
#### offset commit 빈도는 성능 / 중복 메시지 개수 간의 trade-off
#### 어떤 offset 을 commit 하는지 정확히 알자
#### rebalacing
rebalancing 이 시작되기 전에 마지막 메시지의 offset commit 하는 등의 clean up 필요
#### consumer 는 상태를 유지해야 함
#### 긴 처리 시간 대응
heartbeat thread
#### Exactly-once delivery
외부 DB, ES 등

## 시스템 신뢰성 체크
### Configuration 체크
application 로직에서 broker, client 구성을 따로 테스트하기 쉬움
- requirements 를 충족할 수 있는지 테스트하는데 도움
- 시스템에 기대하는 동작을 판단할 수 있음

`org.apache.kafka.tools`: `VerifiableProducer`, `VerifiableConsumer`

- VerfiableProducer
  - 1부터 설정한 값까지의 숫자를 포함하는 메시지 write
- VerifiableConsumer
  - 메시지를 일고 순서대로 출력, commit, rebalancing 에 대한 내용도 출력
- 테스트 시나리오
  - leader 를 중단시키면?
  - controller 를 재시작 후 시스템 재개 시간 체크
  - 메시지 유실 없이 broker rolling restart 가능?
  - unclean leader election 테스트

### Application 체크
어떤걸 테스트하면 좋을지 예시
- client - server connection lose
- leader election
- broker rolling restart
- consumer rolling restart
- producer rolling restart

## Production 에서 Reliability 모니터링
- JMX metric
- lag (처리 지연)
  - broker 의 partiton 에 마지막으로 commit 된 메시지로 부터 얼마나 뒤처져 있는지
    - LinkedIn Burrow 를 사용하면 쉽게 확인 가능