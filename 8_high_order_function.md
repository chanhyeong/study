# 8. 고차 함수

## high order function
- 간략하게 넘김
- lambda or method reference 를 인자로 넘길 수 있거나, 반환하는 함수

### function type
#### lambda 를 로컬 변수로 선언하는 경우
```kotlin
val sum = {x : Int, y: Int -> x + y} // (Int, Int) -> Int = { x, y -> x + y }
val action = { println(42) } // () -> Unit = { println(42) }
```

### parameter 로 함수 받기
```kotlin
fun test(operation: (Int, Int) -> Int) {
  val result = operation(2, 3) // 일반적인 함수 호출 방법과 같음
  println(result)
}
```

### JAVA 에서 kotlin function type 사용
- function type -> (컴파일) -> 일반 인터페이스 FunctionN
  - N 은 인자의 개수
  - 각 인터페이스에는 `invoke` 메소드 정의

```kotlin
// 코틀린에서 람다를 인자로 받음
fun method(f: (Int) -> Int) {
  println(f(42))
}
```
```java
// JAVA 8+ 에서 사용
method(num -> num + 1) // 43

// JAVA 7 이하에서 사용
method(
  new Function1<Integer, Integer>() {
    @Override
    public Integer invoke(Integer num) {
      return number + 2;
    }
  }
) // 44

// JAVA 에서 kotlin lambda 로 넘기기
CollectionsKt.forEach(Collections.singletonList("42"), s -> {
  log.debug(s)
  return Unit.INSTANCE; // void 를 넘길 수 없고, kotlin Unit 을 명시해야 함
})
```

### function type parameter 에 대한 default value 와 nullable 
- 이전에 본 다른 방식과 같음
  - `=` 명시
  - `?` 명시

### function 을 반환하는 function
- 일반적인 선언처럼 리턴 타입으로 lambda 를 명시

### lambda 를 활용한 중복 제거
```kotlin
data class SiteVisit( // 방문 데이터
  val path: String,
  val duration: Double,
  val os: OS
)

enum class OS { WINDOWS, LINUX, MAC, IOS, ANDROID }

val log = listOf( ... )

// predicate 에 해당하는 체류 시간 평균 구하기
fun List<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean) =
  filter(predicate).map(SiteVisit::duration).average()

log.averageDurationFor { it.os == OS.IOS && it.path == "/signup" }
```

## inline function
- lambada 를 활요한 코드의 성능은?
  - inner class 로 컴파일하지만, 사용할 때마다 새로운 클래스가 생성되지는 않는다
  - 변수를 capture 하면, 생성되는 시점마다 새로운 inner class object 가 생성된다 (부가 비용 발생)
  - (*) lambda 를 사용한 구현은 똑같은 작업을 수행하는 일반 함수를 사용한 구현보다 덜 효율적
- `inline` 을 붙이면 컴파일러가 bytecode 로 바꿔치기 해줌

### 동작 방식
- 함수를 호출하는 byte 대신에 함수 자체를 bytecode 로 컴파일
```kotlin
// inline fun 정의
inline fun <T> synchronized(lock: Lock, action: () -> T): T {
  lock.lock()
  try {
    return action()
  }
  finally {
    lock.unlock()
  }
}

// kotlin fun 으로 작성
fun foo(l: Lock) {
  println("before")
  synchronized(1) { println("Action") } // lambda 를 넘김
  println("After")
}

// 생성된 bytecode
fun __foo__(l: Lock) {
  println("Before")
  l.lock()
  try {
    println("Action") // synchronized 가 펼쳐지면서 lambda 가 같이 들어감
  } finally {
    l.unlock()
  }
  println("After")
}

// lambda 가 아닌 다른 곳에서 정의된 function type 을 넘기는 경우
class LockOwner(val lock: lock) {
  fun runUnderLock(body: () -> Unit) {
    synchronized(lock, body) // function type 을 넘김
  }
}

// 생성된 bytecode 2
class LockOwner(val lock: lock) {
  fun __runUnderLock__(body: () -> Unit) {
    try {
      body() // 알수 없는 lambda 이므로 inlining 되지 않음
    } finally {
      l.unlock()
    }
  }
}
```

### 한계
- (번역이 조금 이해하기 어렵게 되어 있음, 다섯 번은 읽은 듯)
  - 내가 문제인가?
- 임의로 이해한 내용
#### inline function 을 lambda expression 의 형태로 파라미터로 넘길 때
- lambda expression 을 바로 호출하거나 lambda expression 을 인자로 받아 바로 호출하는 경우
  - 해당 lambda 를 inlining 할 수 있음
- lambda expression 을 변수에 저장하여 나중에 그 변수를 사용하면
  - Illegal usage of inline-parameter 발생 (compile level)
  - lambda 를 표현하는 객체가 어딘가는 존재해야하기 때문에

```kotlin
// inline 을 사용할 수 없는 예
// transform 을 받아서 새로운 Sequence 를 만드는데, 이 과정에서 TransformingSequence 의 property 로 저장됨
fun <T, R> Sequence<T>.map(transform: (T) -> R): Sequence<R> {
  return TransformingSequence(this, transform)
}
```

- 일부 lambda 만 inline 처리 가능: `noinline`
  - 특정 lambda 에 너무 많은 코드가 들어가거나
  - inlining 을 하면 안되는 코드가 들어갈 가능성이 있는 경우
```kotlin
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) { ... }
```

### collection inlining
- collection standard library 에서 제공하는 함수
  - inline function
  - 추가 객체나 클래스 생성이 없음 = 성능 이슈 없음
- `asSequence`
  - lambda 를 객체로 저장, inline 하지 않음
    - lazy operation 으로 성능 향상을 위해 모든 collection 에 asSequence 에 붙이는 것은 좋지 않음
  - collection 크기가 작은 경우 standard library 가 성능이 더 나을 수 있음

### inline 으로 선언해야 하는 경우
#### 일반적인 함수
- 이미 JVM 은 강력한 inlining 을 지원함
  - byte code -> machine code (JIT) 과정에서 처리
  - 함수 구현이 1개가 있으면, 해당하는 함수를 호출하는 부분에서 함수 코드를 중복할 필요가 없음
  - stacktrace 가 깔끔함
- kotlin inline function
  - 함수 호출 지점을 bytecode 에서 대체하기 때문에 코드 중복이 생김

#### lambda
- inline 으로 선언할 경우 함수 호출 비용을 줄일 수 있음
  - (궁금한 점) 로그는 어떻게 하지?
- lambda 를 표현하는 클래스와 객체를 만들 필요가 없음
  - 이 부분은 JAVA 쪽이 더 나은 듯함
  > 여기까지 본다면 Java의 람다는 앞에서 본 다른 JVM 언어가 그랬듯 익명 클래스 선언 문법을 단순히 대체한 것처럼 보인다. 그러나 람다는 다른 JVM 언어처럼 컴파일 시점에 익명 클래스를 생성하지 않는다. 컴파일된 소스 폴더나 역컴파일을 해도 익명 클래스의 흔적은 없다. 람다 표현식은 익명 클래스 문법과는 다른 바이트코드를 생성한다.  
  > 결과적으로 Java 8에서 람다 표현식으로 객체를 생성하는 코드는 invokedynamic이라는 바이트코드로 변환된다. Java의 역어셈블러(disassembler)인 javap로 이를 확인했다. 예제 30은 단순하게 람다 표현식으로 Runnable 인터페이스의 인스턴스를 생성했다.  
  > ([D2 람다가 이끌어 갈 모던 Java](https://d2.naver.com/helloworld/4911107))
  - `invokedynamic`
    - 컴파일 시점에 타입이 확정되지 않은 메서드를 런타임에 호출
      - 람다 표현식의 해석을 컴파일 시점이 아닌 실행 시점으로 미룸
    - Bootstrap 메서드, 정적 파라미터 목록, 동적 파라미터 목록 등 세 가지 정보를 필요
      - Bootstrap: 호출 대상을 찾아서 연결하고 invokedynamic을 쓰는 메서드가 처음 호출될 때만 실행
      - 정적 파라미터: 상수풀(constant pool)에 저장된 정보
      - 동적 파라미터: 메서드의 런타임에서 참조할 수 있는 변수
  - kotlin 은 JAVA 6 과 호환을 지원 하는데, `invokedynamic` 기능은 JAVA 7 에 해당하는 JVM 에서 등장함
- 코드 크기에 주의해야 함
  - 크기 만큼의 bytecode 를 전부 넣으므로, 전체적인 bytecode 의 크기가 아주 커질 수 있음

### inlined lambda for resource management
- file, lock, database transaction 등
- 일반적인 방법
  - `try/finally` 를 사용하고
  - try 시작 전에 resource 획득 -> finally 블록해서 해제

#### withLock
```kotlin
val l: Lock = ...
l.withLock { // resource lock 을 얻고 작업 수행
  // resource 사용
}
```

#### use
- JAVA `try-with-resource` 개념
- extension function of `Closable`
- inline function 임
```kotlin
fun readFirstLineFromFile(path: String): String {
  BufferedREader(FileReader(path)).use { br ->
    return br.readLine()
  }
}
```

## 흐름 제어
### lambda 를 둘러싼 함수로부터 반환 (번역이..)
- inline function 의 경우
  - lambda 내에서 return 을 사용하면, lambda 가 아닌 바깥 쪽의 함수가 반환됨
  - `non-local`

```kotlin
fun abcd(people: List<Person>) {
  people.forEach { // forEach 로 넘긴 lambda 자체가 inline 처리됨
    if (it.name == "Chan") {
      println("Found!")
      return // forEach 가 아니라 abcd 함수 자체가 끝남
    }
  }
}
```

### return from lambda
- lambda 자체에 대한 local return
- `label` 을 사용해야 함

```kotlin
people.forEach label@{ // 명칭은 꼭 label 이 아니어도 됨, @ 이 본체
  if (it.name == "Alice") return@label
}

people.forEach { 
  if (it.name == "Alice") return@forEach // 함수명으로도 사용 가능
}

// 동시 사용은 안됨
people.forEach label@{ 
  if (it.name == "Alice") return@forEach
}
```

### anonymous function
- 위 2개의 방법은 장황하므로 다른 방법
- 거의 javascript 처럼 보임
```kotlin
people.filter(fun (person): Boolean { // lambda 대신 anonymous function
  return person.age < 30 // return 은 가장 가까운 함수에 대한 반환, 여기서는 anonymous function
})

people.filter(fun (person) = person.age < 30) // expression 이므로 타입 명시하지 않아도 됨
```