# 3. Kafka Producer
![image](https://user-images.githubusercontent.com/10507662/116772307-171dc300-aa89-11eb-84a9-53dbc934f62c.png)
- `ProducerRecord`
  - 카프카에 쓰려는 메시지
  - 토픽 + 값
  - (optional) key, partition
- `Serializer`: byte array 로 serialization
- `Partitioner`
  - partition 지정 시 처리 안함
  - 지정하지 않은 경우 key 를 기준으로 파티션 선택
- partition 지정까지 완료된 경우 **Producer 는 메시지가 저장될 토픽과 파티션을 알게 됨**
  - 같은 토픽 + 파티션으로 전송될 레코드를 모은 record batch 에 추가, 별도의 thread 가 broker 에게 전송
- broker 는 수신된 레코드의 메시지를 처리한 후 응답 전송
  - success? -> `RecordMetadata` 반환: topic, partition, partition 내부의 message offset
  - fail? -> error 반환: producer 는 포기하거나 retry 를 할 수 있음

## kafka producer 구성
### required
#### `bootstrap.servers`
- kafka cluster 에 최초로 연결하기 위해 producer 가 사용하는 broker 들의 host:port 목록
- 최소 2개를 설정하는 것이 좋음 - 1개의 broker 가 중단되어도 다른 곳으로 시도
#### `key.serializer`
- ProducerRecord 의 message key 를 serialization 하기 위해 사용되는 클래스명
- `org.apache.kafka.common.serializer.Serializer` interface 구현하는 클래스
- 기본 제공: ByteArraySerializer, StringSerializer, IntergerSerializer

#### `value.serializer`
- message value, 위 동일

```java
private Properties properties = new Properties();
properties.put("bootstrap.servers", "broker1:9092, broker2:9092");
properties.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
properties.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

Producer<String, String> producer = new KafkaProducer<>(properties);
```

### 전송 방법
#### Fire-and-forget
- 가장 간단한 방법, `send()` 로 전송만 하고 success/fail 결과 후속 작업은 안함
- 일부 메시지 유실 가능

#### Synchronous send
- `send()` 전송 시 `Future` 객체가 반환, `get()` 호출하면 결과가 반환되므로 성공적으로 수행했는지 확인 가능

#### Asynchronous send
- `send()` 호출 시 callback method 를 구현한 `Callable` object 를 전달


## Sending a message to kafka
```java
// topic, key, value
ProducerRecord<String, String> record = new ProducerRecord<>("CustomerCountry", "Precision Products", "France");

// Fire-and-forget
try {
  producer.send(record);
} catch (Exception e) {
  e.printStackTrace();
}

// Synchronous
try {
  producer.send(record).get();
} catch (Exception e) {
  e.printStackTrace();
}


private class ProducerCallback implements Callback {
  @Override
  public void onCompletion(RecordMetadata data, Exception e) {
    if (e != null) {
      e.printStackTrace();
    }
  }
}

// Asynchronous
producer.send(record, new ProduceCallback());
```

- retriable 에러와 불가능한 에러가 있음
  - ex1) no leader: leader 가 재선출되면 해결됨 -> 재시도
  - ex2) message size 가 너무 클 때: retry 안 함
- Roundtrip time (RTT) 가 10ms 라면, blocking 시 100개의 message 전송 시 1초
  - Async 하면 기다리지 않아도 되어 시간 소모 없음

## producer 구성
- producer 의 **message usage, performance, reliability** 에 영향을 주는 parameters

#### `acks`
|acks value|description|
|-|-|
|0|producer 가 broker response 를 기다리지 않음<br>broker 가 메시지를 받지 못해도 producer 는 몰라서 메시지 유실될 수 있음<br>그만큼 빠르게 전송 가능|
|1|leader replica 가 메시지를 받는 순간 producer 는 broker 로부터 성공 수신 응답을 받음<br>leader 에서 메시지를 쓸 수 없다면 broker 는 에러 응답을 받음, 재전송 가능<br><br>leader 가 중단되고 새로운 leader 가 선출된 경우 메시지 유실될 수 있는데<br>이 때 Synchronous 인 경우 대기 시간 때문에 처리량이 떨어질 수 있음|
|all|모든 replica 가 메시지를 받으면 broker 성공 응답<br>acks=1 보다 대기 시간은 더 길어짐|

#### `buffer.memory`
- broker 전송 전 message buffer 로 사용할 메모리 양 설정

#### `compression.type`
- default 는 압축하지 않음
- `snappy, gzip, lz4` 중 하나를 입력하면 해당 알고리즘으로 전송 전 압축
  - snappy: CPU 부담이 적고 성능 좋고 양호한 압축
  - gzip: CPU 와 압축 시간을 더 많이 사용, 압축률이 더 좋음 (네트워크 처리량이 제한적일 때)
- 네트워크와 스토리지 사용을 줄일 수 있음

#### `retries`
- 에러 발생 시 메시지 전송을 포기하고 에러로 처리하기 전 producer 가 재전송하는 횟수
- retry 간 대기 시간은 `retry.backoff.ms` 로 설정할 수 있으며 default 100ms
- 설정 값 코멘트
  - 하나의 broker 가 중단될 때 처리가 복구되는 시간을 테스트한 후 설정

#### `batch.size`
- 같은 partition 에 쓰는 다수의 ProducerRecord 를 batch 로 모으는데
- 각 batch 에 사용될 메모리 크기 (byte) 제어
- 가득 차면 전송하지만, 전발이나 하나의 메시지도 전송하긴 함

#### `linger.ms`
- batch 를 전송하기 전까지 기다리는 시간
- 설정한 제한 시간이 되면 전송

#### `client.id`
- client 식별

#### `max.in.flight.requests.per.connection`
- 서버의 응답을 받지 않고 producer 가 전송하는 메시지의 개수 제어
- 1로 설정 시 메시지의 전송 순서대로 broker 가 씀
  - 한 메시지 batch 의 쓰기가 ;다시 시도되는 동안에는, 다른 메시지들이 추가로 전송되지 않음
- 너무 큰 경우 message batch 처리가 비효율적, 처리량 감소
- 메시지 순서 보장과 관계 있음

#### `timeout.ms, request.timeout.ms, metadata.fetch.timeout.ms`
- producer 가 서버의 응답을 기다리는 제한 시간
  - `request.timeout.ms`: 데이터를 전송할 때
  - `metadata.fetch.timeout.ms`: metadata (메시지를 쓰는 partition 의 현재 leader) 요청
- `timeout.ms`: 동기화된 replica 들이 메시지를 인지하는 동안, broker 가 대기하는 시간

#### `max.block.ms`
- `send()` 를 호출할 때 buffer 가 가득 차거나, `partitionsFor()` 로 metadata 를 요청했지만 사용할 수 없을 때
- producer 가 이 시간 동안 일시 중단 (그 사이에 에러가 해결될 수 있도록)
- 시간이 되면 timeout exception 발생

#### `max.request.size`
- producer 가 전송하는 쓰기 요청 (produce request) 크기 제어
- default 1MB
  - 1KB size 의 메시지를 최대 1024개까지 갖는 batch 로 모아서 하나의 request 로 전송 가능
- broker 의 `message.max.bytes` 와 값을 일치시키는 것이 좋음

#### `receive.buffer.bytes`, `send.buffer.bytes`
- data send/recevie 시 socket 이 사용하는 TCP sending/reading buffer size
- -1 인 경우 OS default

## Serializer
### Custom serializer
- `org.apache.kafka.common.serializer.Serializer` interface 구현
- 생략

### Apache Avro
- language-neutral data serialization format (언어 중립적인)
- 데이터 구조를 표현 시 JSON 형식으로 기술
- serialization 도 JSON 을 지원하지만 주로 binary
- Avro 가 file read/write 시 schema 가 있다고 간주함
- 장점
  - 메시지를 쓰는 애플리케이션이 새로운 schema 로 전환하더라도 해당 메시지를 읽은 애플리케이션은 변경없이 계속 처리 가능
  - JSON 으로 해도 똑같지 않나?
```json
{"namespace": "customerManagement.avro",
 "type": "record",
 "name": "Customer",
 "fields": [
 {"name": "id", "type": "int"},
 {"name": "name", "type": "string"},
 {"name": "faxNumber", "type": ["null", "string"], "default": "null"}
 ]
}

// faxNumber -> email 로 변경
{"namespace": "customerManagement.avro",
 "type": "record",
 "name": "Customer",
 "fields": [
 {"name": "id", "type": "int"},
 {"name": "name", "type": "string"},
 {"name": "email", "type": ["null", "string"], "default": "null"}
 ]
}

```
- 주의점
  - 데이터를 쓰는데 사용되는 schema 와 애플리케이션에서 기대하는 schema 가 호환될 수 있어야 함 (Avro 호환성 규칙 참고)
  - Deserializer 는 데이터를 쓸 때 사용되었던 schema 를 사용해야 함
    - Avro 파일은 header + data block 으로 되어 있으며, header 에 schema 저장

#### Kafka 에서 Avro record 사용하기