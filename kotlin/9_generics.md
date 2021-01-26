# 9. Generics

## Type parameter
- kotlin 에서는 generic type parameter 를 명시하거나 컴파일러가 추론할 수 있어야 함
  - JAVA 는 1.5부터 이 개념이 들어가서 이런 제약이 없음

### generic function, property, class
```kotlin
// function
// <T>: type parameter
fun <T> List<T>.slice(indices: IntRange): List<T>

// property
val <T> List<T>.penultimate: T
  get() = this[size - 2]

// class
interface List<T> {
  operator fun get(index: Int): T
}
```

### type parameter constraint
- type argument 를 제한하는 기능
- upper bound 를 지정

```kotlin
fun <T : Number> List<T>.sum: T
```
```java
<T extends Number> T sum(List<T> list)
```

- 2개 이상의 constraint 설정
  - (JAVA 에 이런 개념이 있던가?)
```kotlin
fun <T> ensure(seq: T)
  where T: CharSequence, T: Appendable // 2개 설정
  ...
```

### type parameter as nonnull type
- 아무 것도 명시하지 않은 경우 디폴트는 `Any?` 가 upper bound 로 지정, nullable
```kotlin
// default: nullable
class Processor<T> {
  fun process(value: T) {
    value?.hashCode() // nullable
  }
}

// nonnull
class Processor<T: Any> {
  fun process(value: T) {
    value.hashCode()
  }
}
```

## erased type parameter & reified type parameter
- JVM Generics: **type erasure** 를 사용하여 구현됨
  - 런타임에 인스턴스에 대한 type 정보가 들어있지 않음
  - 저장해야하는 정보 크기가 줄어서 메모리 사용량이 줄어듦

### Generics on runtime
- type argument 정보는 runtime 에 지워짐
- 인스턴스를 생성할 때 사용된 type argument 에 대한 정보를 유지하지 않음
  - `List<String>` 객체를 만들더라도, 런타임에서는 `List` 로만 볼 수 있음

#### 한계
```kotlin
if (value is List<String>) // Cannot check for instance of erased type
```

- `as`, `as?` 에서도 generic type 사용 가능
```kotlin
fun sum(c: Collection<*>) {
  val intList = c as? List<Int> ?: throws IllegalArgumentException("List is expected") // 컴파일 시점 Unchecked cast 경고
  println(intList.sum())
}

sum(listOf(1, 2, 3)) // 정상
sum(setOf(1, 2, 3)) // IllegalArgumentException
sum(listOf("1", "2", "3")) // ClassCastException (String -> Int)

// 컴파일 시점에 타입 정보가 주어진 경우, is check 는 가능
fun sum(c: Collection<Int>) {
  if (c is List<Int>) {
    println(c.sum())
  }
}
```

### declare function using reified type parameter
```kotlin
fun <T> isA(value: Any) = value is T // Cannot check for instance of erased type: T

inline fun <reified T> isA(value: Any) = value is T
isA<String>("abc") // true

inline fun <reified T> Iterable<*>.filterIsInstance(): List<T> {
  val result = mutableListOf<T>()
  for (element in this) {
    if (element is T) {
      result.add(element)
    }
  }

  return result
}

val items = listOf("one", 2, "three")
items.fliterIsInstance<String>()
```
#### inline function 에서만 사용 가능
이유?
- 컴파일러가 bytecode 로 변환 시, `reified` 가 명시된 부분에 대한 구체적인 클래스를 확인하는 코드를 생성
```kotlin
// filterIsInstance<String>() 의 경우
for (element in this) {
  if (element is String) {
    result.add(element)
  }
}
```

- 9.2.3, 9.2.4 생략

## variance

### class, type, subtype
- type != class

#### class
- generic 이 아닌 경우: 이름을 바로 type 으로 사용 가능
- generic 인 경우: 올바른 type 을 얻으려면 구체적인 type argument 를 제공해야 함 (`List<Int>`, `List<String?>`, ...)

#### subtype <-> supertype
- Int 는 Number 의 subtype
- String 은 String? 의 subtype
- **subclass** 는 간단한 경우 subtype 과 동일

#### invariant, covariant
- generic type 을 인스턴스화 할 때 서로 다른 type argument 가 들어간 경우
  - 인스턴스 간 subtype 관계가 성립하지 않으면 -> **invariant**
  - JAVA 에서는 모든 클래스가 invariant
- A 가 B 의 subtype 이면 List\<A> 는 List\<B> 의 subtype -> **covariant**

### covariance
- (공변성)
- subtype 관계를 유지
- out position 에서만 사용 가능
- `public, protected, internal` 에만 해당, `private` 은 해당 없음
- `out` 키워드 명시 (generic 클래스가 type 에 대한 covariant 함을 표현)
  - 없는 경우 invariant
- 기능
  - 위험할 여지가 있는 메소드를 호출할 수 없게 만듦
  - generic instance 를 잘못 사용하는 일이 없게 방지
```kotlin
interface Producer<out T> {
  fun produce(): T // T 는 out position 에서만 사용 가능

  // in position 에서는 불가능, 전혀 다른 subtype 으로 변경할 수 있으므로
  // Type parameter T is declared as 'out' but occurs in 'in' position
  fun transform(t: T): T
}
```

### contravariance
- (반공변성)
- in position 에서만 사용 가능
- A 가 B 의 subtype 인데 Consumer\<B> 가 Consumer\<A> 의 subtype 인 경우
- `in` 키워드 명시
```kotlin
interface Comparator<in T> {
  fun compare(e1: T, e2: T): Int {...} // in position 에서만 사용 가능
}
```

|covariance|contravariance|invariance|
| ----------- | ----------- | ----------- |
|out|int|-|
|type parameter 의 subtype 이<br /> generic 에서도 이어짐|type parameter 의 subtype 이<br /> generic 에서 뒤집힘|subtype 성립 없음|
|T - out position 에서만 가능|T - in position 에서만 가능|아무데서나 가능|

- convariance + contravariance 인 예시
```kotlin
interface Function1<in P, out R> {
  operator fun invoke(p: P): R
}
```

### use-site variance
- wildcard type 을 사용하는 JAVA 에서의 방식
  - `? extends`, `? super`