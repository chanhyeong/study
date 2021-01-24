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
