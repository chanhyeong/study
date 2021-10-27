# 10. Reactor

## Reactive programming
functional, declarative.  
데이터가 흘러가는 pipeline, stream 을 포함

### Reactive stream 정의
차단되지 않는 backpressure 를 갖는 asyncronous stream processing 표준을 제공하는 것이 목적

backpressure: 데이터를 소비하는 컨슈머가 처리할 수 있는 만큼으로 전달 데이터를 제한 -> 데이터 전달 폭주를 피할 수 있음

####  Publisher, Subscriber, Scription, Processor.
- Publisher 는 하나의 Subcription 당 하나의 Subscriber 에 발행하는 데이터를 생성
  - Subscriber 가 구독 신청할 수 있는 `subscribe()` 한 개를 선언
- Subscriber
  - 수신할 첫 번째 이벤트는 `onSubscribe()` 의 호출
  - Publisher 가 `onSubcribe()` 를 호출할 때, argument 로 Subscription 객체를 넘김
  - Subscriber 는 Subscription 으로 구독 관리
    - `request()` 로 요청, `cancel()` 로 구독 취소
  - `onNext()`  메소드 호출되어 Publisher -> Subscriber 로 데이터 전달
  - `onError()` - 오류 발생 시
  - `onComplete()` - 작업 완료

## Reactor

### Marble Diagram: Diagram of Reactive flow
1. 제일 위에는 Flux or Mono 를 통해 전달되는 데이터의 타임라인
2. 중앙에는 operation
3. 제일 밑에는 결과로 생성되는 Flux or Mono 의 타임라인

![image](https://projectreactor.io/docs/core/release/reference/images/legend-operator-method.svg)

## Reactive Operation 적용

500개 이상의 operation
- creation, combination, transformation, logic

### creation
```kt
// 객체에서 생성
val fruitFlux = Flux.just("Apple", "Banana", "Kiwi")

// 컬렉션에서 생성
val fruits = arrayOf("Apple", "Banana", "Kiwi")

val fruitFlux = Flux.fromArray(fruits)

val fruitList = listOf("Apple", "Banana", "Kiwi")

val fruitFlux = Flux.fromIterable(fruits)

val fruitStream = Stream.of("Apple", "Banana", "Kiwi")

val fruitFlux = Flux.fromIterable(fruits)

// Flux 데이터 생성
val intervalFlux = Flux.range(1, 5)
val intervalFlux2 = Flux.interval(Duration.ofSeconds(1))
```

### combination
- [`mergeWith()`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#mergeWith-org.reactivestreams.Publisher-): 단순히 합침
- [`zip()`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#zip-org.reactivestreams.Publisher-org.reactivestreams.Publisher-): 순서대로 합침
  - 기본적으로 Tuple2 로 묶이지만
  - [2개를 조합하여 새로운 결과를 낼 수 있음](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#zip-org.reactivestreams.Publisher-org.reactivestreams.Publisher-java.util.function.BiFunction-)
- [`firstWithSignal()`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#firstWithSignal-org.reactivestreams.Publisher...-): 2개의 Flux 중 먼저 발행되는 것만 전부 취함

### transformation and filtering
- `skip(n)`: 처음 n 개를 버리고 그 이후부터 취하여 새로운 flux 로 생성
  - 개수, Duration 등 설정 가능
- `take(n)`: 처음 n 개만 취해서 새로운 flux 를 만들고 나머지는 버림
  - 개수, Duration 등 설정 가능
- `filter()`: 로직에 해당되는 것만 취해서 새로운 flux 로 생성

### mapping
- `map()`: operation 의 수행 결과로 변환된 값을 새로운 flux 로 생성
  - **synchronous**
- `flatMap()`: operation 이 새로운 Mono or Flux 가 되고, 결과가 하나의 새로운 Flux 가 됨
  - 어떻게 정의하느냐에 따라서 병렬 매핑을 수행할 수 있음
  ```kt
  val playerFlux = Flux.just("M J", "S P", "S K")
    .flatMap { name -> Mono.just(name)
      .map {
        val split = it.split(" ")
        Player(split[0], split[1])
      }
    }.subscribe(Schedulers.parallel()) // 각 구독이 병렬로 수행되어야 함

  val playerList = listOf(
    Player("M", "J"), Player("S", "P"), Player("S", "K")
  )

  StepVerifier.create(playerFlux)
    .expectNextMatches { it in playerList }
    .expectNextMatches { it in playerList }
    .expectNextMatches { it in playerList }
    .verifyComplete()
  ```
- Schedulers
  - `immediate()`: 현재 thread 에서 구독
  - `single()`: 재사용 가능한 단일 thread. 모든 호출자에 대해 동일한 thread 재사용
  - `newSingle()`: 매 호출마다 전용 thread 에서 구독 실행
  - `elastic()`: infinite pool 에서 가져온 thread 에서 구독 수행, 필요 시 새로운 thread 수행 및 유휴 thread 제거
  - `parallel()`: fixed pool 에서 가져온 thread 에서 구독 수행. CPU 코어의 개수가 크기

### buffering
```kt
val fruitFlux = Flux.just("Apple", "Banana", "Kiwi", "Tomato") // Flux<String>

val bufferedFlux = fruitFlux.buffer(3) // Flux<List<String>>

StepVerifier.create(bufferedFlux)
  .expectNext(listOf("Apple", "Banana", "Kiwi"))
  .expectNext(listOf("Tomato"))
  .verifyComplete()

// ================
val fruitFlux = Flux.just("Apple", "Banana", "Kiwi", "Tomato") // Flux<String>

val bufferedFlux = fruitFlux.buffer() // Flux<List<String>>
val bufferedFlux2 = fruitFlux.collectList() // Mono<List<String>>

// ================
val fruitFlux = Flux.just("Apple", "Banana", "Kiwi", "Tomato")

val numFruitMapMono = fruitFlux.collectMap { it.charAt(0) } // Mono<Map<Char, String>>
```

### logic
`all()`, `any()`