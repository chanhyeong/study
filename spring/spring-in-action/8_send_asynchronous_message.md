# 8. 비동기 메시지 전송하기

## RabbitMQ 와 AMQP 사용하기
JMS 는 java 에서만 사용 가능

AMQP message 는 exchange 와 routing key 를 주소로 사용

message -> broker -> exchange -> queue

exchange type, exchange - queue binding, message routing key 를 기반으로 처리

#### exchange 종류
- Default: broker 가 자동 생성. routing key 와 이름이 같은 queue 로 메시지 전달. 모든 queue 는 자동으로 여기와 연결됨
- Direct: binding key 가 해당 메시지의 routing key 와 같은 queue 에 메시지 전달
- Topic: binding key (wildcard 포함) 가 해당 메시지의 routing key 와 일치하는 하나 이상의 queue 에 바인딩
- Fanout: binding key 나 routing key 에 상관 없이 모든 queue 에 메시지 전달
- Header: Topic 과 유사, routing key 대신 header 값을 기반
- Dead letter: 어떤 exchange - queue binding 과도 일치하지 않는 모든 메시지를 보관

### RabbitTemplate 을 이용한 메시지 전송
JmsTemplate 과 유사

- `send()`, `convertAndSend()` - exchange 와 routing key 의 형태로 메시지 전송
- `MessagePostProcessor` - 전송되기 전에 Message 객체를 조작하는데 사용

#### MessageConverter 구성
- MessageConverter bean 을 등록하면 자동으로 RabbitTemplate 에 주입
- Jackson2JsonMessageConverter: 객체를 JSON
- MarshallingMessageConverter: spring Marsaller, Unmarshaller 이용
- SerializerMessageConverter: spring Serializer, Deserializer
- SimpleMessageConverter: String, byte array, Serializable 을 변환
- ContentTypeDelegatingMessageConverter: contentType 헤더를 기반으로 다른 converter 에 위임

### RabbitMQ 에서 메시지 수신
#### RabbitTemplate 으로 가져오기
`receive()`, `receiveAndConvert()`

타임아웃 동안 수신할 수 있는 메시지가 없으면 null 이 반환됨

#### listener 를 사용하여 처리
`@RabbitListener(queues= "queue-name")`

## Kafka 사용하기
topic 사용, topic 은 cluster 의 모든 broker 에 복제됨

### KafkaTemplate 을 사용하여 메시지 전송
#### `send()`
- topic
- partition, record key, timestamp (optional)
- payload

### KafkaListener 작성
KafkaTemplate 으로는 수신할 수 없음

```kotlin
@KafkaListener(topics="topic-name")
fun handle(Order order) { ... }

@KafkaListener(topics="topic-name")
fun handle(Order order, ConsumerRecord<Order> record) { ... }

@KafkaListener(topics="topic-name")
fun handle(Order order, Message<Order> record) { ... }
```
