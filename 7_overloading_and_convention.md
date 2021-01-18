# 7. operator overloading, convention
- JAVA: 언어 기능이 특정 **타입** 에 연결되어 있음
- kotlin: 언어 기능이 특정 **함수명** 과 연관됨
  - `convention`: 언어 기능 - 미리 정해진 이름의 함수를 연결해주는 기법
  - `...`, `in`, ... 등

## arithmetic operator

### binary
|expression|method name|
| ----------- | ----------- |
|a * b|times|
|a / b|div|
|a % b|rem (1.0 은 mod)|
|a * b|times|
|a + b|plus|
|a - b|minus|

- 예제
```kotlin
data class Point(val x:Int, val y: Int) {
  operator fun plus(other: Point): Point = Point(x + other.x, y + other.y)
}

val p1 = Point(1, 2)
val p2 = Point(3, 4)
p1 + p2 // Point(4, 6)

// extension 으로도 정의할 수 있고, 피 연산자가 다른 타입일 수도 있음
// Double 예제
operator fun times(scale: Double): Point = Point((x * scale).toInt(), (y * scale).toInt())
Point(10, 20) * 1.5 // Point(15, 30)

// 교환 법칙은 지원하지 않음
// 아래 예를 지원하려면 Double 쪽에 extension 정의 필요
1.5 * Point(10, 20)
```

#### bit
- infix call 로 호출되며 목록은 아래와 같음

|kotlin|JAVA|
| ----------- | ----------- |
|shl|<<|
|shr|>>|
|ushr|>>>|
|and|&|
|or||\||
|xor|^|
|inv|~|

```kotlin
0x0F and 0xF0
```

### compound assignment
- +=, -=
- return type `Unit` 함수를 정의하면 됨
  - `plusAssign`, `minusAssign`, `timesAssign`
- mutable collection 에 대해서는 기본 제공중
- 아래와 같이 사용하도록 권장, 하나의 메소드만 정의


|사용 케이스|메소드|
| ----------- | ----------- |
|mutable|plusAssign|
|immutable|plus|

### unary
|expression|method name|
| ----------- | ----------- |
|+a|unaryPlus|
|-a|unaryMinus|
|!a|not|
|++a, a++|inc|
|--a, a--|dec|

## comparison operator

### equals
- `a == b` -> `a?.equals(b) ?: (b == null)`
- `a != b` 도 `equals` 가 호출됨

### compareTo
- `a >= b` -> `a.compareTo(b) >= 0`
```kotlin
class Person(val firstName: String, val lastName: String): Comparable<Person> {
  override fun compareTo(other: Person): Int = 
    compareValuesBy(this, other, Person::lastName, Person::firstName)
}
```

## collection, range convention
### 인덱스로 접근
- `get`, `set`
```kotlin
// array 처럼 사용 가능
mutableMap[key] = newValue

// 클래스에 get, set 을 구현하여 위와 같이 사용하는 예제
operator fun Point.get(index: Int): Int { // Int 가 아닌 파라미터도 정의 가능
  return when(index) {
    0 -> x
    1 -> y
    else -> throw IndexOutOfBoundsException("")
  }
}

val p = Point(10, 20)
p[1] // 20

operator fun MutablePoint.get(index: Int, value: Int): Int {
  return when(index) {
    0 -> x = value
    1 -> y = value
    else -> throw IndexOutOfBoundsException("")
  }
}

val p = MutablePoint(10, 20)
p[1] = 42
p // 10, 42
```

### in
- `contains`
```kotlin
// 커스텀 in (contains) 구현
data class Rectangle(val upperLeft: Point, val lowerRight: Point)

operator fun Rectangle.contains(p: Point): Boolean =
  p.x in upperLeft.x until lowerRigth.x
    && p.y in upperLeft.y until lowerRight.y

val rect = Rectangle(Point(10, 20), Point(50, 50))
Point(20, 30) in rect // true
```

### ..
- `rangeTo`
- `Comparable` 을 구현하고 있는 클래스라면, 별도 정의할 필요 없음
  - kotlin 에서 기본적으로 `Comparable` 에 대한 extension 으로 제공
```kotlin
operator fun <T: Comparable<T>>.T.rangeTo(that: T): ClosedRange<T>

// operator 우선순위가 낮으므로 사용 시 주의
0..(n + 1)
(0..n).forEach { ... }
```

### for in
- `iterator`
```kotlin
operator fun ClosedRange<LocalDate>.iterator(): Iterator<LocalDate> =
  object: Interator<LocalDate> {
    var current = start

    override fun hasNext() = current <= endInclusive
    override fun next() = current.apply { current = plusDays(1) }
  }

val year = LocalDate.now()
val range = year.minusDays(1)..year

for (day in range) { ... }
```

## destructing declaration
- 구조 분해 선언
- js 의 json 내 key 를 바로 꺼내서 쓸 수 있는 기능과 비슷하면서 다름