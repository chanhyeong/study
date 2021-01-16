# 6. 타입 시스템

## Nullability
- 컴파일 시점에서 null 확인

### nullable type
- kotlin 의 디폴트는 전부 not null
- `?` 기호를 명시 -> nullable
  - `Type?` = `Type` or `null`
```kotlin
// null 을 받을 수 없음
fun strLen(s: String) = s.length
strLen(null) // 컴파일 에러

// null 을 받을 수 있지만, 메소드 직접 호출 불가능
fun strLenSafe(s: String?) = s.length // 컴파일 에러

// nullable 타입을 not null 타입에 넣을 수 없음
val x: String? = null
var y: String = x

// nullable 타입을 not null 타입을 받는 인자로 넣을 수 없음
strlen(x)
```

### 타입의 의미
- classification (분류), 값들과 수행할 수 있는 연산을 정의
- JAVA 에서의 타입 예시 -> `String` 타입
  - String, null 두 가지 값이 들어갈 수 있음
  - 하지만 null 의 경우
    - `instanceof` 수행 시 `String` 은 아님
    - `String` 값이 있는 경우 연관 연산을 모두 쓸 수 있으나, null 은 불가능한게 많음
    - -> JAVA 타입 시스템이 null 을 제대로 다루지 못한다
- JAVA 에서의 NPE 다루기
  - `@Nullable`, `@NotNull` 이 있긴 하지만 컴파일 절차의 일부가 아니기 때문에 정확히 보장되진 않음
  - `Optional`
    - 코드가 지저분해지고 wrapper 사용에 따른 성능 저하 우려
- kotlin 에서는 ?
  - 런타임에서는 nullable type, not null type 의 객체는 모두 같음 (wrapper 가 아님)
  - 모든 처리는 컴파일 시점에 진행되므로, 런타임에서 비용 없음

### `?.`, `?:`, `as?`, `!!`
- kotlin 에서 null 을 체크하는 주요 연산자
  - `?.`: null 이면 호출 무시, 아니면 결과값
  - `?:`: null 대신 사용할 디폴트 값 지정
    - elvis operator 라고도 함
  - `as?`: 타입 캐스팅, 불가능하면 null
  - `!!`: not-null assertion, null 인 경우 NPE 발생
    - 컴파일러에게 null 이 아님을 알고 있고, 잘못 넘겨서 예외가 발생해도 감수하겠다는 표시
    - 못생긴 기호인데, 의도한 것 -> 이거 사용하기 보단 다른 방법을 사용해라

```kotlin
// ?., ?: 예제
class Address(val streetAddress: String, val zipCode: Int, val city: String, val country: String)
class Company(val name: String, val address: Address?) // 주소가 없을 수 있음
class Person(val name: String, val company: Company?) // 회사가 없을 수 있음

fun Person.countryName(): =
  this.company?.address?.country ?: "Unknown" // 회사의 주소의 국가가 있으면 해당 값 반환, 아니면 Unknown

// as? 예제
override fun equals(other: Any?): Boolean {
  /*
  이 코드를 한 줄로 변경할 수 있음
  if (other !is Client)
    return false
  */
  val otherPerson = other as? Person ?: return false
  
  return name == otherPerson.name && postalCode == otherPerson.postalCode
}

// !! 예제
fun ignoreNulls(s: String?) {
  val sNotNull: String = s!!
  println(sNoutNull.length)
}

ignoreNulls(null)

// 이런식으로는 !! 를 쓰지 말기, 로그 라인 추적 불가능
person.company!!.address!!.country
```

### let
- 자신의 수신 객체를 인자로 전달받은 람다에 넘김
- 넘어온 값이 널이 아닌 경우에만 수행해야하는 식이 있을 때 사용
- `let` 이 중첩되는 경우 코드가 복잡해지니 기존처럼 `if` 로 null 체크하는게 더 나음
```kotlin
fun sendEmailTo(email: String) { ... }

val email: String? = ...
sendEmailTo(email) // 불가능

email?.let { sendEmailTo(it) }

// person 변수의 추가 선언 없이 바로 하나의 코드 라인으로 null 체크 및 expression 수행 가능
fun getPerson(): Person? = null
getPerson()?.let { sendEmailTo(it.email) }
```

### lateinit
- 프로퍼티를 나중에 초기화(lazy initailize) 할 수 있음
- JUnit 과 같은 프레임워크 사용 예

```kotlin
class MyService {
  fun performAction(): String = "foo"
}

class MyTest {
  private lateinit var myService: MyService // var 로 선언해야 함

  @Before fun setup() {
    myService = MyService();
  }

  @Test fun testAction() {
    assertEquals("foo", myService.performAction()) // null check 없이 프로퍼티 사용
  }
}
```

### nullable type extension
- nullable 타입에 대한 확장 함수를 정의하면, null 을 더 효율적으로 다룰 수 잇음
```kotlin
// 기본 제공 확장 함수
fun String?.isNullOrBlank(): Boolean = this == null || this.isBlank()

fun verify(input: String?) {
  if (input.isNullOrBlank()) { // null check 없이 String extension function 호출
    println("error")
  }
}
```

### nullablility of type parameter
- 모든 함수나 클래스의 모든 타입 파라미터는 **nullable**
```kotlin
fun <T> printHashCode(t: T) { // ? 를 명시하지 않았지만 nullable 함
  println(t?.hashCode());
}

printHashCode(null) // T = Any? 로 추론됨
```
- not null 처리를 하려면 **upper bound** 지정
  - 상세한 내용은 9장 generics 에서 예정
```kotlin
fun <T: Any> printHashCode(t: T) { // not null
  println(t.hashCode());
}

printHashCode(null) // 컴파일 에러
```

### JAVA 상호 운용
- 어노테이션이 있는 경우 kotlin 에서도 그 정보를 활용
  - JSR-305, 안드로이드 (`android.support.annotation`), jetbrains (`org.jetbrains.annotiation`)
> @Nullable String -> String?  
> @NotNull String -> String

#### (*) platform type
- kotlin 이 null 관련 정보를 알 수 없는 타입
  - kotlin class 로는 선언 불가능
- null 일 수도 있고, null 이 아닐 수도 있음
  - 컴파일러는 모든 연산을 허용
  - 책임은 개발자에게
  - 틀렸다면 NPE
  - `Type` (JAVA) -> `Type?` or `Type` (kotlin)
- 왜 이런 방식으로 도입했나? 모든 JAVA 타입을 nullable 로 보면 되지 않나?
  - 불필요한 null check 가 들어가므로, 개발자 재량으로 둠
```java
// JAVA
public class Person {
  private final String name;
  { ... }
}
```
```kotlin
// kotlin
fun yellAt(person: Person) {
  println(person.name.toUpperCase() + "!") // JAVA Person 의 name 이 null 인지 아닌지 알 수 없지만, 예외 처리 없이 그냥 호출
}

yellAt(Person(null)) // NPE

// safety call
fun yellAtSafe(person: Person) {
  println((person.name ?: "Anyone").toUpperCase() + "!")
}

yellAtSafe(Person(null)) // ANYONE!
```

## Primitive Type

### Int, Boolean, ...
#### JAVA 의 경우
- `primitive type`: 값이 들어감
  - int, boolean, ...
- `reference type`: 변수의 메모리 상 주소 값
  - Integer, Boolean, ...

#### kotlin 에서는
- JAVA 와 같은 구별이 없음
- primitive type 에 대한 메소드 호출 가능
- 런타임에서 효율적인 방식으로 표현
- 대부분의 경우 kotlin Int -> JAVA int 와 같은 타입으로 컴파일
- collection 등의 generic class 를 사용할 때는 JAVA reference type

> 정수: Byte, Short, Int, Long  
> 소수점: Float, Double  
> 문자: Char  
> Boolean

### Int?, Boolean?
- nullable 하기 때문에 JAVA 의 reference type 으로 변환됨

#### 번외 - generic class 에서 reference type (wrapper) 를 사용해야 하는 이유?
- JVM 에서의 generic 구현 방식 때문
- JVM 은 타입 인자로 primitive type 을 허용하지 않음
- JAVA, kotlin 모두 generic 은 boxed type 을 사용해야 함

### 숫자 변환
- 숫자의 자동 형변환 불가능
- `toByte()`, `toShort()`, `toChar()` 등을 이용
  - boxed type 을 비교하는 경우의 문제 때문에 분리
  - JAVA `new Integer(42).equals(new Long(42))` = false

```kotlin
val one = 1
val longOne = Long = i // 컴파일 에러
val longOne = one.toLong()

val x = 1
val list = listOf(1L, 2L, 3L)
x in list // false
x.toLong() in list // true

// 기타 (String -> 숫자로 변환, 실패 시 NumberFormatException)
"42".toInt() // toLong(), toBoolean()
```

### Any, Any?
- JAVA 의 `Object`: 모든 클래스의 최상위 타입
- `Any`: kotlin 의 **not null type** 에 대한 최상위 타입
  - Int 등의 primitive type 을 포함 (JAVA 에서는 Object 가 int 를 포함하지 않음)
  - JAVA 의 `Object` 에 대응
```kotlin
val answer: Any = 42 // boxed int
```

### Unit
- JAVA `void` 와 같은 기능
- 다른점?
  - 타입 인자로 사용 가능
```kotlin
interface Processor<T> {
  fun process(): T
}

class NoResultProcessor: Processor<Unit> {
  override fun process() { ... } // Unit 이므로 return 을 명시할 필요 없음
}

// JAVA 도 Void 라는 개념이 있지만, return null 로 리턴해야 함
```

### Nothing
- 아무 값도 반환하지 않음 + **정상 종료되지 않음을 명시**

```kotlin
fun fail(message: String?): Nothing {
  throw IllegalStateException(message)
}

val address = company.address ?: fail("No")
println(address.city) // 예외 발생
```

## Collection, Array

### nullable 과 collection
```kotlin
List<Int?> // list 내의 값이 nullable
List<Int>? // list 자체가 nullable

List.filterNotNull() // list 내 null 은 제거한 새로운 list 구하기
```

### read-only 와 mutable collection
- `Collection`: 추가, 제거 불가능
- `MutableCollection`: 추가, 제거 가능, `Collection` 상속

#### 주의
- `MutableCollection` 을 받는 파라미터에 `Collection` 은 넘길 수 없으나
- `Collection` 을 받는 파라미터에는 `MutableCollection` 을 넘길 수 있음
  - 동일 collection 을 어느 메소드에서는 `Collection`, 다른 메소드에서는 `MutableCollection` 으로 처리하여
  - `ConcurrentModificationException` 이 발생할 수 있음
  - multithread 인 경우, read-only 로 받더라도 thread-safe 하지 않을 수 있음을 알아야 함

### JAVA collection 과의 운용
|type|read-only type|mutable type|
| ----------- | ----------- | ----------- |
|List|listOf|mutableListOf, arrayListOf|
|Set|setOf|mutableSetOf, hashSetOf, linkedSetOf, sortedSetOf|
|Map|mapOf|mutableMapOf, hashMapOf, linkedMapOf, sortedMapOf|

#### JAVA 에서 kotlin collection 사용 시 주의
- kotlin 에서 read-only 로 선언했더라도, JAVA 에서는 개념이 없기 때문에 알 수 없음
  - JAVA 로 넘기는 경우 값이 변경될 수 있음
- kotlin 에서 값을 not null 로 처리했더라도, JAVA 에서 값을 null 로 넣어버릴 수도 있음
- JAVA 쪽으로 넘길 때에는 JAVA 에서 코드 처리를 더 잘 해야함

### platform type 으로 collection 처리
- JAVA 쪽에서 선언한 collection type -> platform type
  - kotlin 에서 read-only 로 받든, mutable 로 받든 상관 없음

### Object Array, Primitive type Array
- kotlin 에서 배열 생성하기
  - `arrayOf`, `arrayOfNulls`
  - `Array` consturctor -> 크기와 람다를 인자로 받음
- `List.toTypedArray` 로 리스트 -> 배열 변환

```kotlin
Array<String>(26) { i -> ('a' + i).toString() } // abcdefghijklmnopqrstuvwxyz

println("%s/%s/%s".format(*listOf("a", "b", "c").toTypedArray()))

Array<Int> // == JAVA new Integer[]
IntArray // == JAVA new int[]
```

- collection 과 같이 `map`, `filter` 등의 메소드 지원
  - 반환되는 값은 배열이 아닌 list
