# 2. 카프카 설치와 구성하기

## broker configuration
### 핵심
#### `broker.id`
- host1.example.com, host2.example.com 등

#### `port`
- default 9092

#### `zookeeper.connect`
- broker metadata 를 저장하기 위한 zookeeper host

#### `log.dirs`
- 모든 메시지에 대한 log segment 파일 위치

#### `num.recovery.threads.per.data.dir`
- log segment 처리 thread pool
- 설정값 * `log.dirs` 에 기입한 디렉토리 수만큼 생성됨

#### `auto.create.topics.enable`
- 토픽이 없을 때 자동으로 생성
  - producer 가 쓸 때, consumer 가 읽을 때, client 에서 topic metadata 를 요청할 때

### topic 관련
#### `num.partitions`
- 새로운 topic 이 몇 개의 파티션으로 생성되는지. default 1
- partition 수는 증가만 가능하고 감소될 수 없음

##### 파티션 개수의 산정 방법
- 단위 시간 당 토픽의 throughtput? (ex. 100KB/s 1GB/s)
- 한 partition 의 데이터를 읽을 때 목표로 하는 최대 throughput?
  - consumer 가 초당 50MB 밖에 처리하지 못한다면, 최대 throughput 은 50MB 로 제한
- 하나의 partition 에 데이터를 생성하는 throughput per producer 도 비슷한 방법으로 산정
- key 를 사용해서 partition 에 쓰는 경우, partition 추가 시 개수 산정이 어려울 수 있음
- broker 마다 num of partitions 및 network processing speed 를 고려
- partition 개수를 너무 많이 ㄴㄴ
  - 각 partition 은 broker memory 와 그 외 자원을 사용하므로, leader election 에 많은 시간이 소요 됨
- ex) 초당 1GB 로 topic 을 read/write 하려고 하는데
  - consumer 가 50MB 만 처리할 수 있다면, 최소한 20개의 (1000/50) 파티션이 필요
  - 20개의 consumer 가 topic 을 읽음

#### `log.retention.ms`
- 얼마 동안 메시지를 보존할지
- hours, minutes 도 있음

#### `log.retention.bytes`
- byte 크기 기준

#### `log.segment.bytes`
- segment 기준 크기

#### `log.segment.ms`
- segment 파일 닫히는 주기

#### `message.max.bytes`
- broker 로 쓰기 요청이 들어오는 메시지의 최대 크기 제한. default 1MB