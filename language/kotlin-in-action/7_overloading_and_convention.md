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
- `val (a, b) = p` -> `val a = p.component1()`, `val b = p.component2()`
  - `componentN` 함수 호출
- js 의 json 내 key 를 바로 꺼내서 쓸 수 있는 기능과 비슷하면서 다름

```kotlin
// 커스텀 선언 예
class Point(val x:Int, val y:Int) {
  operator fun component1() = x
  operator fun component2() = y
}

// loop 예제
fun printEntries(map: Map<String, String>) {
  // 1. in (iteration) convention
  // 2. Map.Entry 의 destructing declaration
  for ((key, value) in map) {
    println("$key -> $value")
  }
}
```
```js
// 비슷한 js 예
const json = {
  name: "123",
  age: 23,
  gender: "Male"
}

const { name, age } = json
```

## property accessor 로직 재활용
- backing field 에 단순히 저장하는 것보다 더 복잡한 방식으로 동작하는 프로퍼티를 구현 가능
- 객체가 직접 작업을 수행하지 않고 다른 helper 객체가 작업을 처리하게 하는 디자인 패턴

### delegated property 소개
- Delegate 는 `getValue`, `setValue` 메소드를 제공해야 함
- `by` 는 property 와 delegate 를 연결
```kotlin
// 일반적인 문법
class Foo {
  var p: Type by Delegate()
}

// 컴파일러가 생성하는 형태
class Foo {
  private val delegate = Delegate() // 컴파일러가 private property 로 생성
  var p: Type
    set(value: Type) = delegate.setValue(..., value)
    get() = delegate.getValue(...)
}

val foo = Foo()
val oldValue = foo.p // delegate.getValue(...)
foo.p = newValue // delegate.setValue(..., value)
```

### delegated property 사용 - lazy initalization
```kotlin
// email 조회가 오래 걸려서 한 번만 lazy 하게 조회하는 경우 예시

// 조회 함수
fun loadEmails(person: Person): List<Email> { ... }

class Person(val name: String) {
  val emails by lazy { loadEmails(this) } // by lazy
}
```

### delegated property 구현
- operator `getValue`, `setValue` 구현 + by 로 연결
```kotlin
// JAVA UI 예제, 다수 생략됨
open class PropertyChangeAware() {
  protected val changeSupport = PropertyChangeSupport(this)
}

// getValue, setValue 구현
class ObservableProperty() {
  operator fun getValue()
  operator fun setValue()
}

// by 로 연결
class Person():PropertyChangeAware() {
  val age: Int by ObserverableProperty(age, changeSupport)
  val salary: Int by ObserverableProperty(salary, changeSupport)
}
```

### delegated property 컴파일 룰
- 인스턴스를 `<delegate>` 라는 이름으로 저장
- 인스턴스의 프로퍼티를 `KProperty` 타입의 객체로 표현하며, `<property>` 라고 명칭
```kotlin
class C {
  var prop: Type by MyDelegate()
}

// 컴파일
class C {
  private val <delegate> = MyDelegate()
  var prop: Type
    get() = <delegate>.getValue(this, <property>)
    set() = <delegate>.setValue(this, <property>, value)
}

val c = C()
val x = c.prop // val x = <delegate>.getValue(c, <property>)
c.prop = x // <delegate>.setValue(c, <property>, x)
```

### property value 를 map 에 저장
- **expando object** (확장 가능한 객체)
  - 자신의 property 를 동적으로 정의할 수 있는 객체
  - ex) 연략처 별 임의의 정보를 저장할 수 있게 허용하는 경우
    - 정보를 map 에 저장하되 map 을 처리하는 property 를 통해 정보 제공
```kotlin
class Person {
  private val _attributes = hashMapOf<String, String>() // Map, MutableMap 에서 getValue, setValue 가 제공됨

  fun setAttribute(attrName: String, value: String) {
    _attribute[attrName] = value
  }

  val name: String by _attrbutes // delegated property
}

val p = Person(0
val data = mapOf("name" to "Chanhyeong", "company" to "Test Cooperation")
for ((attrName, value) in data) {
  p.setAttribute(attrName, value)
}

p.name // _attribute.getValue(p, prop) -> _attribute[prop.name]
```

### 프레임워크에서의 활용
- 생략