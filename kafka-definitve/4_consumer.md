# 4. Consumer
- `KafkaConsumer` 를 사용하여 kafka 토픽 읽기 + 메시지 받기

## Consumer, Consumer Group
Producer 들이 토픽에 새로운 메시지를 쓰는 속도가, 애플리케이션에서 처리하는 속도보다 빠르면?  
-> 하나의 Consumer 만으로 처리한다면 처리 속도가 처짐  
-> 다수의 Consumer 로 구성하자

- **Consumer 들은 Conusmer group 에 속한다**
- 다수의 Consumer 가 같은 토픽을 소비하면서 같은 Consumer group 에 속할 때는
- 각 Consumer 가 해당 토픽의 서로 다른 Partition 을 분담해서 메시지를 읽을 수 있다

#### 구성
![image](https://user-images.githubusercontent.com/10507662/117542009-aad72c80-b051-11eb-960e-28c7f51e065a.png)

|구성|매핑|비고|
|-|-|-|
|partition 4개 - Consumer 4개|1:1||
|partition 2개 - Consumer 4개|1:1|Consumer 2개는 놀게 됨|
|partition 4개 - Consumer 2개|2:1||

- 위 이미지처럼 Consumer group 은 각각 토픽을 읽음 (다른 Consumer group 과의 관계는 없음)

#### topic message 처리
- 읽으려면? -> Consumer group 생성
- 확장하려면? -> 기존 group 에 새 consumer 추가

### Consumer group 과 Rebalancing
- Consumer group 의 Consumer 들은 **topic partition 의 소유권 (ownership) 공유**
  - 새 Conusmer 를 추가하면 이전 다른 Consumer 가 읽던 partition 의 메시지들을 읽음
  - 특정 Consumer 가 중단될 때도 동일
- **Rebalancing**: Consumer 간 partition 소유권을 이전하는 것
  - Rebalancing 과정에선 Consumer 들이 메시지를 읽을 수 없음
  - partition 이 옮겨질 때 이전 partition 에 대한 현재 상태 정보가 지워짐

#### Group coordinator, Heartbeat (소유권 관련)
- group coordinator 로 지정된 kafka broker 에 Consumer 가 heartbeat 를 전송하면
  - 자신이 속한 Consumer group 의 멤버십과 partition 소유권을 유지할 수 있음
  - `GroupCoordinator` 로 백그라운드에서 떠있으며, Consumer group 마다 broker 가 다를 수 있음
- heartbeat
  - Consumer 의 상태를 알리기 위해 전송되는 신호
  - polling or 메시지를 commit 할 때 자동 전송
  - 0.10.1 부터 별도의 heartbeat thread 가 추가됨
  - Consumer 가 session timeout 까지 heartbeat 를 보내지 않으면, `GroupCoordinator` 는 리밸런싱 시작

#### partition assignment 과정
Consumer 가 Group 에 합류하고 싶을 때? -> `GroupCoordinator` 에게 `JoinGroup` 요청 전송
- Group 에 첫 번째로 합류하는 Consumer 는 **leader**
- leader 가 consumer 들에 partition 할당하는 책임, `PartitionAssigner`

## Kafka Consumer 생성하기
Producer 와 비슷
- `bootstrap.servers`, `key.serializer`, `value.serializer`
- `group.id`: Consumer Group

```java
Properties properties = new Properties();
properties.put("bootstrap.servers", "broker1:9092, broker2:9092");
properties.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
properties.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

KafkaConsumer<String, String> producer = new KafkaConsumer<>(properties);
```

## Subscribe topic
```java
consumer.subscribe(Collections.singletonList("customerCountries"));
```
- topic List 를 받으며, regex 도 가능

## Polling loop
- 서버로 부터 많은 데이터를 계속 읽기 위해 polling loop
- thread safe 하지 않으므로, 하나의 thread 에서 여러 consumer 를 다루지 않아야 함
```java
try {
  while (true) {
    ConsumerRecords<String, String> records = consumer.poll(100); // 100ms 동안 broker 에서 데이터 가져오기 대기
    for (ConsumerRecord<String, String> record : records) {
      log.debug("topic = %s, partition = %s, offset = %d, customer = %s, country = %s\n", 
        record.topic(), record.partition(), record.offset(), record.key(), record.value());
    
      int updatedCount = 1;
      if (custCountryMap.countainsValue(record.value())) {
        updatedCount = custCountryMap.get(record.value()) + 1;
      }
    
      custCountryMap.put(record.value(), updatedCount);
      JSONObject json = new JSONObject(custCountryMap);
      log.debug(json.toString(4))
    }
  }
} finally {
  consumer.close(); // 종료 시 닫아줌
}
```

## Consumer 구성하기
#### `fetch.min.bytes`
- broker 에서 가져오는 데이터 최소 크기
- broker 에 모인게 설정 값보다 작으면, 채워질 때까지 기다렸다가 Consumer 로 보냄

#### `fetch.max.wait.ms`
- 위에 설정한 값만큼 모이지 않아도, 이 설정 시간에 되면 보냄. 디폴트 500ms

#### `max.partition.fetch.bytes`
- partition 당 반환하는 최대 bytes 제어. 디폴트 1MB

#### `session.timeout.ms`
- Consumer - broker 가 연결이 끊기는 시간. 디폴트 10s
  - Consumer 가 heatbeat 를 전송하지 않고 살아있을 수 있는 시간
- `heartbeat.interval.ms`: heartbeat 를 전송하는 시간 간격 제어. 보통 session 의 1/3 로 설정

#### `auto.offset.reset`
- 커밋된 offset 이 없는 partition / 커밋된 offset 이 있지만 유효하지 않을 때 **Consumer 가 어떤 레코드를 읽을지 제어**
- `latest`: Consumer 가 실행한 이후 가장 최근 레코드부터 읽음. 디폴트
- `earliest`: partition 의 맨 앞부터 읽음. 중복해서 읽을 가능성이 있지만, 유실 최소화

#### `enable.auto.commit`
- Consumer 의 offset commit 을 자동으로 할지. 디폴트 true
- `auto.commit.interval`: 자동으로 offset 을 commit 하는 시간 간격

#### `partition.assignment.strategy`
- topic partition 을 어떻게 할당할지 지정하는 클래스 명. 디폴트 Range

##### Range
- `org.apache.kafka.clients.RangeAssignor`
- 연속으로 할당
- partiton 3개를 갖는 T1, T2 가 있고 C1, C2 가 있으면
  - 1) C1 - T1, T2 의 0, 1 받음
  - 2) C2 - T1, T2 의 2 받음

##### Round Robin
- `org.apache.kafka.clients.RoundRobinAssignor`
- 번갈아 할당
- partiton 3개를 갖는 T1, T2 가 있고 C1, C2 가 있으면
  - 1) C1 - T1 0, 2 + T1 1 받음
  - 2) C2 - T1 1 + T2 0, 2 받음

#### client.id

#### max.poll.records
- 1회 `poll()` 시 반환되는 레코드 최대 개수 제어
- 11번가 발표에서 배포 속도와 관계

#### `receive.buffer.bytes`, `send.buffer.bytes`
- TCP buffer 제어

## Commit, Offset
- `Offset`: Consumer 가 partition 별로 자신이 읽는 레코드의 현재 위치
- `Commit`: partition 내부의 현재 위치를 변경하는 것
- `__consumer_offsets` topic 으로 내부적으로 관리

### Automatic commit
- `KafkaConsumer` 가 자동으로 offset commit

### 현재의 offset commit
- 생략

## Rebalancing Listener
- Consumer 는 종료되기 전이나 partition rebalacing 이 시작되기 전에 cleanup 처리해야 함
  - 마지막 offset commit
- `subscribe()` 에 `ConsumerRebalanceListener` interface 구현체를 전달

#### public void onPartitionsRevoked(Collection<TopicPartition> partitions)
- rebalancing 시작 전 + Consumer 가 consume 중단 후 호출
- offset commit 실행 시점

#### public void onPartitionsAssgined(Collection<TopicPartition> partitions)
- partition 이 재할당된 후, Consumer 가 partition 을 새로 받아 consume 시작할 때

## 특정 offset 을 사용하여 레코드 consume
- 특정 offset 을 찾을 수 있음
- 이전 메시지들로 돌아가거나, 메시지를 건너뛰는 경우
