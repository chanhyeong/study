# 9. 스프링 통합하기
Enterprise Integration Patterns (2003) 에서 보여진 통합 패턴을 사용할 수 있게 구현한 것

정의법
1. XML (책에 예시가 하나 있지만, 비추천하여 정리하지 않음)
2. Java
3. Java DSL
4. (책에는 없음) [Kotlin DSL](https://docs.spring.io/spring-integration/reference/html/kotlin-dsl.html#kotlin-dsl)

## Component 구성
| 종류 | 설명 |
| - | - |
| Channel | a -> b 로 메시지 전달 |
| Filter | 조건에 맞는 메시지가 flow 를 통과하게 해줌 |
| Transformer | 메시지 값을 변경하거나 payload 타입을 다른 타입으로 변환 |
| Router | 여러 Channel 중 하나로 메시지 전달 (대부분 헤더 기반) |
| Splitter | 메시지를 두 개 이상으로 분할하여 각 다른 채널로 전송 |
| Aggregator | 별개의 채널로부터 전달되는 메시지를 하나로 결합 |
| Service activator | 메시지를 처리하도록 java 메소드에 넘겨주고, 반환 값을 outbound channel 로 전송 |
| Channel adpater | 외부 시스템에 채널을 연결, 외부로 부터 read/write 가능 |
| Gateway | 인터페이스를 통해 통합 플로우로 데이터 전달 |

### Channel
Integration pipeline 을 통해 메시지가 이동하는 수단 (= spring integration 의 다른 부분을 연결하는 통로)

- `PublishSubscribeChannel`: 하나 이상의 consumer 로 모두 전달
- `QueueChannel`: FIFO 방식으로 consumer 가 가져갈 때까지 큐에 저장. consumer 가 여러 개인 경우 하나만 수신
- `PriorityChannel`: Queue 와 유사하지만 priority header 를 기반으로 consumer 가 가져감
- `RendezvousChannel`: Queue 와 유사하지만, consumer 가 메시지를 수신할 때까지 publisher 가 channel block
- `DirectChannel`: PubSub 과 유사하지만, publisher 와 동일한 thread 로 실행되는 consumer 를 호출하여 단일 consumer 에게 메시지 전송. 트랜잭션 지원
- `ExecutorChannel`: Direct 와 유사하지만 TaskExecutor 를 통해 메시지가 전송, transaction 미지원
- `FluxMessageChannel`: Project Reactor 의 Flux 를 기반으로 하는 Reactive Streams Publisher channel

```kotlin
// 메소드 정의
@Bean
fun orderChannel(): MessageChannel = PublishSubscribeChannel()

// subscribe
@ServiceActivator(inputChannel = "orderChannel")

// =====================

// Java DSL
@Bean
fun orderFlow(): IntegrationFlow = IntegrationFlows
  ...
  .channel("orderChannel")
  ...
  .get()

// =====================

// queue channel 정의
@Bean
fun orderChannel(): MessageChannel = QueueChannel()

// subscribe 시 주기 설정
@ServiceActivator(inputChannel = "orderChannel", poller = @Poller(fixedRate = "1000"))
```

### Filter
numberFilter -> 짝수일 때만 evenNumberChannel 로 전송 예시

```kotlin
// 정의 (@Bean 은 책에서 누락된 것 같은 느낌낌)
@Filter(inputChannel = "numberChannel", outputChannel = "evenNumberChannel")
fun evenNumberFilter(number: Int): Boolean = number % 2 == 0

// =====================

// Java DSL
@Bean
fun evenNumberFlow(): IntegrationFlow = IntegrationFlows
  ...
  .<Int>filter { p -> p % 2 == 0 } // GenericSelector
  ...
  .get()
```

### Transformer
value or type 변경

```kotlin
// 정의
@Bean
@Transformer(inputChannel = "numberChannel", outputChannel = "romanNumberChannel")
fun romanNumTransformer(): GenericTransformer<Integer, String> = RomanNumbers::toRoman

// =====================

// Java DSL
@Bean
fun tranformerFlow(): IntegrationFlow = IntegrationFlows
  ...
  .transform(RomanNumbers::toRoman)
  ...
  .get()
```

### Router
numberChannel 의 값 중 짝수, 홀수 채널 구분

```kotlin
// 정의
@Bean
@Router(inputChannel = "numberChannel")
fun evenOddRouter(): AbstractMessageRouter = object : AbstractMessageRouter() {
    override fun determineTargetChannels(Message<*> message) : Collection<MessageChannel> {
      val number = message.payload as Int

      return if (number % 2 == 0) {
        setOf(evenChannel())
      } else {
        setOf(oddChannel())
      }
    }
  }

@Bean
fun evenChannel() = DirectChannel()

@Bean
fun oddChannel() = DirectChannel()

// =====================

// Java DSL
@Bean
fun numberRoutingFlow(): IntegrationFlow = IntegrationFlows
  ...
  .<Integer, String>route({n -> if (n % 2 == 0) "EVEN" else "ODD"}) { mapping ->
    mapping.subFlowMapping("EVEN") {
      it.<Integer, Integer>transform(n -> n * 10).handle { i, h -> ... }
    }
    .subFlowMapping("ODD") {
      it.transform(RomanNumbers::toRoman)
    }
  }
  ...
  .get()
```

### Splitter
1. message payload 가 같은 타입의 컬렉션 항목을 포함하며, 각 message payload 별로 처리하고 할 때
    - 여러 종류의 제품이 있고, 제품 리스트를 전달하는 메시지는 각각 한 종류 제품의 payload 를 갖는 다수의 메시지로 분할
2. 연관된 정보를 함께 전달하는 하나의 message payload -> 2개 이상의 서로 다른 타입 메시지로 분할

```kotlin
class OrderSplitter() {
  fun splitOrderIntoParts(po : PurchaseOrder) : Collection<Any> {
    val parts = listOf<Any>()
    parts.add(po.billingInfo)
    parts.add(po.lineItems)
    return parts
  }
}

// 정의
@Bean
@Splitter(inputChannel = "poChannel", outputChannel = "splitOrderChannel")
fun orderSplitter(): OrderSplitter = OrderSplitter()

// router 에서 받아서 적합한 하위 플로우로 전달 (상위 설명의 2번)
@Bean
@Router(inputChannel = "splitOrderChannel")
fun splitOrderRouter(): MessageRouter = MessageRouter {
  val router = PayloadTypeRouter()
  router.setChannelMapping(BillingInfo.javaClass.name, "billingInfoChannel")
  router.setChannelMapping(List.javaClass.name, "lineItemsChannel")
  return router
}

// List<LineItem> 을 개별 LineItem 을 처리하는 채널로 분할 (상위 설명의 1번)
@Bean
@Splitter(inputChannel = "lineItemsChannel", outputChannel = "lineItemChannel")
fun lineItemSplitter(lineItems: List<LineItems>): List<LineItem> = lineItems

// =====================

// Java DSL
IntegrationFlows
  ...
  .split(orderSplitter())
  .<Object, String>route({p -> if (p::class.java.isAssignableFrom(BillingInfo::class.java)) "BILLING_INFO" else "LINE_ITEMS"}) { mapping ->
    mapping.subFlowMapping("BILLING_INFO") {
      it.<BillingInfo>handle { billingInfo, h -> ... }
    .subFlowMapping("LINE_ITEMS") {
      it.split()
        .<LineItem>handle { lineItem, h -> ... }
    }
  ...
  .get()
```

### Service Activator
input channel 의 메시지를 수신하여 `MessageHandler` 인터페이스를 구현한 bean 에 전달

```kotlin
// input 을 받아서 처리
@Bean
@ServiceActivator(inputChannel = "someChannel")
fun sysoutHandler(): MessageHandler = MessageHandler {
  println("Message payload: ${it.payload}")
}

// =====================

// Java DSL
@Bean
fun someFlow(): IntegrationFlow = IntegrationFlows
  ...
  .handle {
    println("Message payload: ${it.payload}")
  }
  ...
  .get()

// =====================

// input 을 받아서 output 으로 전달
@Bean
@ServiceActivator(inputChannel = "orderChannel", outputChannel = "completeChannel")
fun orderHandler(): GenericHandler<Order> = GenericHandler { payload, headers ->
  orderRepo.save(payload)
}

// =====================

// Java DSL
@Bean
fun orderFlow(orderRepo: OrderRepository): IntegrationFlow = IntegrationFlows
  ...
  .<Order>handle { payload, headers
    orderRepo.save(payload)
  }
  ...
  .get()
```

### Gateway
application 이 Integration Flow 로 데이터를 submit + optional 하게 처리 결과를 받는 수단

**인터페이스로 정의하며, spring integration 이 runtime 에 자동으로 구현체를 제공**

```kotlin
// 정의
@Bean
@Gateway(defaultRequestChannel = "inChannel", defaulReplyChannel = "outChannel")
interface UpperCaseGateway {
  fun uppercase(in: String): String
}

// =====================

// Java DSL
@Bean
fun uppercaseFlow(): IntegrationFlow = IntegrationFlows
  .from("inChannel")
  .<String, String>transfrom { it.uppercase() }
  .channel("outChannel")
  .get()
```

### Channel adpater
IntegrationFlow 의 입/출구 (inbound/outbound)

```kotlin
// 정의
@Bean
@InboundChannelAdapter(poller = @Poller(fixedRate = "1000"), channel = "numberChannel")
fun numberSource(source: AtomicInteger): MessageSource<Int> = MessageSource {
  GenericMessage(source.getAndIncrement())
}

// =====================

// Java DSL
@Bean
fun someFlow(integerSource: AtomicInteger): IntegrationFlow = IntegrationFlows
  .from(integerSource, "getAndIncrement") {
    it.poller(Pollers.fixedRate(1000))
  }
  ...
  .get()
```

### Endpoint module
https://docs.spring.io/spring-integration/reference/html/endpoint-summary.html#spring-integration-endpoints