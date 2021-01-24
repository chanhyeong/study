# 5. 람다
- 기존 JAVA 에서는 anonymous inner class 로 수행, JAVA 8 부터 도입

## 기본
### 문법
```kotlin
// { 파라미터 -> 본문 }
{ x: Int, y: Int -> x + y }

// 예시1) 변수에 저장
val sum = { x: Int, y: Int -> x + y }
sum(1, 2)

// 예시2) 직접 호출
{ println(42) } ()
run { println(42) } // 본문 직접 실행, run { }
```
- collection 에 속하는 최대값 찾기
```kotlin
val people = listOf(Person("Alice", 29), Person("Bob", 31))

// 보통의 사용 예
people.maxBy { it.age }

// 위를 풀어쓴 예 (순서대로)
people.maxBy({ p: Person -> p.age }) // 기본 문법
people.maxBy() { p: Person -> p.age } // 함수 호출 시 맨 뒤에 있는 인자가 람다 식이면, 람다를 괄호 밖으로 빼낼 수 있음
people.maxBy { p: Person -> p.age } // 소괄호 생략 가능
people.maxBy { p -> p.age } // 파라미터 타입 컴파일러 추론

// member reference (JAVA 의 method reference)
people.maxBy(Person::age)
```

#### 유의
- 람다가 중첩되는 경우 `it` 로 사용하기보단, 파라미터를 명시하는 편이 좋음
- 변수에 저장 시에는 타입 명시

### 변수 접근 가능
- JAVA 는 final 만 접근 가능, 값 변경 불가능
- kotlin 은 제약 없음, 값 변경 가능

#### variables captured in the closure (람다가 포획한 변수)
- 일반적으로
  - 로컬 변수의 생명주기는 함수가 반환되는 끝남
- kotlin 에서
  - 어떤 함수가 자신의 로컬 변수를 capture 한 람다를 반환하거나 다른 변수에 저장한다면?
  - 로컬 변수 - 함수의 생명주기가 다를 수 있음
  - 1\) final 변수를 capture
    - 람다 코드를 변수 값과 함께 저장
  - 2\) final 이 아닌 변수를 capture
    - 변수를 특별한 래퍼로 감싸서 나중에 READ/UPDATE 할 수 있게 하고
    - 래퍼에 대한 참조를 람도 코드와 함께 저장

### member reference
- `::`
```kotlin
// 클래스::멤버
Person::age

// JAVA 의 this::study 의 경우
fun study() = println("Study!")
run(::study)
```
- constructor reference
```kotlin
// JAVA 의 Person::new 와 동일
val createPerson = ::Person
val p = createPerson("Alice", 29)
```
#### bound member reference
- 1.0 에서는 클래스의 메소드, 프로퍼티에 대한 참조를 얻고, 인스턴스 객체 제공
- 1.1 부터 인스턴스의 멤버 호출 가능

```kotlin
// member reference
val p = Person("Test", 29)
val personAgeFunction = Person::age
personAgeFunction(p) // 29

// bound member reference
val personAgeFunction = p::age
personAgeFunction(p) // 29
```

## Collection Functional API
### filter, map
- 기본적으로 연산이 완료된 **새로운 컬렉션을 반환**
- JAVA 처럼 stream 으로 변환할 필요 없음

### all, any, count, find
- JAVA 의 `allMatch, anyMatch, count, findAny`
- 모두 람다를 받음
- `find`
  - 만족하는 값이 없는 경우 null
  - `firstOrNull` 과 같음
```
people.count { it.age <= 25 }
```

### groupBy
- JAVA 의 `.collect(toMap())`
- 훨씬 간결
```kotlin
val people = listOf(Person("A", 31), Person("B", 32))
people.groupBy { it.age }
```

### flatMap
- map -> flatten
  - 람다 수행 -> 하나의 컬렉션으로 모음
```kotlin
val strings = listOf("abc", "def")
println(strings.flatMap { it.toList() })
```

## lazy operation
- `Collection Functional API` 에서의 내용은 전부 즉시 생성 (eagerly)
  - 매 연산마다 중간 결과를 새로운 컬렉션으로 생성
- sequence 를 사용 -> 컬렉션 생성 없이 연산을 연속적으로 수행
  - JAVA stream 와 비슷함
  - JAVA stream 이 있는데 왜 따로 만들었는가?
    - kotlin 의 지원 범위는 JAVA 1.6 부터이므로
```kotlin
people.asSequence() // sequence 로 변환
  .map(Person::name)
  .filter { it.startsWith("A") }
  .toList() // 다시 리스트로 변환
```

### intermediate, terminal operation
- 중간 연산: 다른 sequence 를 반환 (`map`, `filter`)
- 최종 연산: 결과 반환
```kotlin
listOf(1, 2 , 3 ,4).asSequence()
  .map { print("map($it "); it * it }
  .filter { print("filter($it "); it % 2 == 0 }
  .toList() // 최종 연산, 이게 없으면 연산이 이루어지지 않음

map(1) filter(1) map(2) filter(4) map(3) filter(9) map(4) filter(16)

[4, 16]

// 리스트에서 2 까지만 연산
listOf(1, 2, 3, 4).asSequence()
  .map { it * it }.find { it > 3 }
```

### sequence 생성
- `asSequence()`
- `generateSequence()`
```kotlin
val nums = generateSequence(0) { it + 1 }
val numsTo100 = nums.takeWhile { it <= 100 }
numsTo100.sum() // 5050
```

## JAVA Functional Interface 와 연동
- `functional interface` or `SAM interface (single abstract method)`
  - 추상 메소드가 단 하나만 있는 인터페이스
- kotlin 의 함수 타입은 8장에서 나올 예정

```java
public class Button {
  public void setOnClickListener(OnclickListener l) { ... }
}
```
```kotlin
button.setOnClickListener { view -> ... }
```

### JAVA 메소드에 람다로 전달
```java
void example(int delay, Runnable computation)
```
```kotlin
// anonymous inner class (무명 클래스) 로 넘기기
// !! 메소드를 호출할 때마다 새로운 객체가 생성됨
example(1000, object: Runnable {
  override fun run() {
    println(42)
  }
})

// lambda 로 넘기기
// !! 변수가 없다면, 람다에 대응하는 무명 객체를 메소드가 호출할 때마다 반복 사용
// !! 변수를 capture 한다면 같은 인스턴스 사용 불가
example(1000) { println(42) }
```

### SAM 생성자
- 람다를 함수형 인터페이스로 명시적으로 변경
- 컴파일러가 자동으로 함수형 인터페이스 클래스로 변환하지 못하는 경우, SAM 생성자 사용
- 그냥 함수를 변수에 넣는거 아닌가..?

```kotlin
fun createHelloRunnable: Runnable {
  return Runnable { println("Hello") }
}

createHelloRunnable().run()

val listener = OnClickListener { view ->
  val text = when (view.id) {
    id.button1 -> "1"
    id.button2 -> "2"
    else -> "unknown"
  }

  toast(text)
}

button1.setOnClickListener(listener)
button2.setOnClickListener(listener)
```

## with, apply (수신 객체 지정 람다)
- `lambda with receiver`
  - 수신 객체를 명시하지 않고, 람다의 본문 안에서 다른 객체의 메소드를 호출
  - JAVA 에는 없는 기능

### with
- 객체의 이름을 반복하지 않고도, 객체에 대한 다양한 연산을 수행
```kotlin
fun alphabet() = with(StringBuilder()) {
  for (letter in 'A'...'Z') {
    this.append(letter) // this 로 명시
  }
  append("\nOK") // this 생략
  toString()
}
```
- 첫 번째 파라미터를 두 번째 람다의 수신 객체로 사용
- 람다에서 `this` 로 접근 가능
- 반환 값 = 람다 코드를 실행한 결과

#### 메소드 이름이 충돌하는 경우?
- ex. `toString()` 이 클래스의 메소드로 이미 존재하고, 사용하고 싶은 경우
```kotlin
this@OuterClass.toString()
```

### apply
- 람다의 결과 대신 수신 객체가 필요한 
  - 객체의 인스턴스를 만들면서 즉시 프로퍼티 중 일부를 초기화해야 하는 경우
```kotlin
fun alphabet() = StringBuilder().apply {
  for (letter in 'A'...'Z') {
    append(letter)
  }
  append("\nOK")
}.toString()
```
- 11장 DSL 에서 좋은 도구로써 사용 예정