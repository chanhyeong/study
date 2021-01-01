# 1. 소개
## 대상 플랫폼
- 안드로이드, 서버
- 자바스크립트로 컴파일 가능 (브라우저 단 가능)
- 기타
  - 코틀린 네이티브
    - JVM 위에서가 아닌 OS 별로 직접 컴파일하여 실행

## 철학
#### 1) 실용성
- 다른 언어의 장점이나 최신 언어 설계 반영
#### 2) 간결성
- 코드의 의미없는 라인을 줄임 (getter, setter 등)
```java
// JAVA 기준
public class Person {
  private String name;

  ...

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }
}
```
```kotlin
class Person(val name: String)
```
#### 3) 안전성
- 컴파일 단계에서 npe 체크, 타입 스마트 캐스트 등
#### 4) 상호운용성
- JAVA 와 섞어서 사용 가능

#  2. 기초
## 예시
```kotlin
fun max(a: Int, b: Int): Int {
  return if (a > b) a else b
}

// 위와 동일
fun max(a: Int, b: Int): Int = if (a > b) a else b
// 반환 값 생략 가능 (컴파일러가 타입 추론)
fun max(a: Int, b: Int) = if (a > b) a else b
```
1\) `fun 함수명(파라미터): 반환 타입` 의 형태  
2\) 삼항 연산자 (ternary operator) 없음
- `(a > b) ? a : b` => `if (a > b) a else b`
- `return` 생략 가능
- if 가 문(statement)이 아니라 **식(expression)** 이다 🤔
  - 식: 값을 만들어 내고 다른 식의 하위 요소로 계산에 참여
  - 문: 가장 안쪽 블록의 최상위 요소로 존재, 아무런 값을 만들어내지 않음

## 변수
### val
- immutable (value)
- 자바의 final 과 비슷하지만, 빈 값으로 선언 후 정의 가능
```java
// JAVA, 불가능
final int answer;
answer = 42;
```
```kotlin
val answer: Int
answer = 42
```

### var
- mutable (variable)

### 문자열 템플릿
- 웬만한 언어에는 다 있는데, JAVA 에는 없음
- 중괄호는 생략 가능하나 영어가 아닌 언어를 붙이는 경우 오류 발생 가능, 생략하지 않는 것이 좋음
```kotlin
fun main() {
  val test = "World!"
  println("Hello, ${test}")
}
```
```javascript
function main() {
  const test = "World!"
  console.log(`Hello, ${test}`)
}
```

## 클래스와 프로퍼티
코틀린에서 내부 필드는 기본적으로 `public`
### 프로퍼티
#### JAVA
- field + accessor method (접근자) (getter, setter) 를 묶은 개념
- 각각 생성해야 함
#### Kotlin
- 클래스 선언 시 언어 단에서 자동으로 생성, 제공
- 코틀린에서 선언한 클래스를 자바에서 사용 시, 자바에서 기존에 사용하던 방식과 동일하게 accessor method 를 생성해 줌

### 커스텀 접근자
```kotlin
class Rectangle(val height: Int, val width, Int) {
  val isSquare: Boolean
    // property getter
    get() = height == width
}
```

## enum, when, smart cast
### enum
```kotlin
enum class Color (
  val num: Int, val korean: String
) {
  RED(1, "빨강"), ORANGE(2, "주황"), GREEN(3, "초록");

  fun combination() = "${num} ${korean}"
}
```

### when
값, 식 모두 사용 가능
```kotlin
fun getMnemonic(color: Color) =
  when (color) {
    RED -> "빨"
    ORANGE -> "주"
    GREEN -> "초"
    else -> "X"
  }

fun mix(c1: Color, c2: Color) =
  when {
    (c1 == RED && c2 == YELLOW) || (c1 == YELLOW && c1 == RED) -> ORANGE
    else -> throw Exception("Wrong")
  }
```

### smart cast
JAVA 와 Kotlin 비교
```java
// JAVA 에서는 instanceof 로 체크를 한 후에도 명시적으로 형 변환이 필요
if (object instanceof RuntimeException) {
  return ((RuntimeException)object).getMessage();
} else {
  throw new IllegalArgumentException();
}
```
```kotlin
// Kotlin 에서는 is 로 체크하면 자동으로 변환해줌
when (object)
  is RuntimeException -> object.getMessage();
  else -> throw IllegalArgumentException();
```

## iteration
- while, do-while
  - 다른 언어와 다르지 않음

### 범위로 조회
```kotlin
// .. 연산자로 범위 생성 가능, 시작 - 끝 모두 inclusive
val oneToTwenty = 0..20
// 문자도 가능
val aToF = 'A' ..'F'

// 369 게임
fun threeSixNine(i: Int) = 
  when {
    i % 3 == 0 -> "clap"
    else -> "$i "
  }

// for 루프에서 범위로 탐색 가능
for (i in 1..100) {
  print(threeSixNine(i))
}

// 역순으로 2 씩 차감하면서 탐색
for (i in 100 downTo 0 step 2) {
  print(threeSixNine(i))
}

// 마지막 수는 포함하지 않고 탐색
for (i in 0 until 100) {
  print(threeSixNine(i))
}
```

### map, collection 탐색
- map 은 java array 처럼 [] 로 변경, 조회 가능
  - `map[key] = value`
- (개인적으로) JAVA 보다 매우 간결함
```kotlin
// map
val map = TreeMap<Char, String>
for ((letter, binary) in map) {
  println("$letter = $binary")
}

// collection
val list = arrayListOf("10", "11", "111");
for ((index, element) in list.withIndex()) {
  println("$index: $element")
}
```
```java
// map (코드는 정확하지 않음)
Map<Char, String> map = new TreeMap<Char, String>
for (Map.Entry<Char, String> entry in map.entrySet()) {
  System.out.println(String.format("%s %s", entry.getKey(), entry.getValue());
}

// collection
List<String> list = List.of("10", "11", "111");
for (int i = 0; i < list.size(); i++) {
  System.out.println(String.format("%d %s", i, list.get(i));
}
```

#### in 으로 탐색 추가 기능
```kotlin
// 범위에 속하는지 검사
fun test(c: Char) = when (c) {
  in '0'..'9' -> "숫자"
  in 'a'..'z' -> "문자"
  else -> "알 수 없음"
}

// collection 에 속하는지 검사
println("Kotlin" in setOf("Java", "Scala"))
```

## 예외 처리
- `new` 가 필요 없음
- `try, catch, finally`
- 문이 아닌 식
- 함수 끝에 `throws` 명시 필요 없음
  - checked, unchecked exception 을 구분하지 않음
- `try-with-resource` 는 지원하지 않음

# 3. 함수
- 코틀린은 자체 collection 을 제공하지 않음, 자바 기반
  - 자바 코드와 상호 작용하기 쉽게 하기 위함
  - 자바 collection 보다 확장된 기능은 제공
    - `last(), max()` 등

## 함수 기본
### named parameter, default parameter
- 다른 언어에는 있지만 JAVA 에는 없는 기능
```kotlin
fun <T> joinToString(
  collection: Collection<T>,
  saperator: String = ", ",
  prefix: String = "",
  postfix: String = ""
)

joinToString(list)
joinToString(list, separator = "|")
joinToString(list, "|", "pre = ")
```

### static utility class 없애기 => 최상위 함수, 프로퍼티
- 예시 1 (함수)
- `package strings` 만 import 시키면 사용 가능
```kotlin
package strings

fun joinToString(...) : String {...}
```
```java
// 보통 JAVA 의 경우 위와 동일한 경우에는 아래와 같은 클래스를 추가해야 함
package strings;

public StringUtils {
  public static joinToString(...) {...}
}
```
- 예시 2 (프로퍼티)
```kotlin
var opCount = 0

// JAVA 의 public static final
// primitive, String 만 선언 가능
const val CURRENT_YEAR = 2021
```
### 기타 (Kotlin 에서 선언한 코드를 JAVA 에서 사용하기)
#### `@JvmOverloads`
- JAVA 에서 지원하지 않는 default parameter 를 이용 중인 메소드를 변환 시
  - 파라미터 하나 씩 제거된 메소드를 자동으로 생성해줌
#### `@JvmName`
- 디폴트: 파일 명이 `join.kt` 인 경우 `JoinKt` 라는 JAVA class 로 변환
- 사용 시: `@file:JvmName("StringFunctions")` => `StringFunctions` 를 이름으로 갖는 class 로 변환

## 확장 (extension) 함수와 프로퍼티
- 기존 JVM 기반 언어들의 코드를 Kotlin 으로 확장

### function
```kotlin
/**
 * JAVA String 에 함수 추가
 */
package strings

fun String.lastChar() : Char = get(length - 1)

println("Kotlin".lastChar())
```
```kotlin
// 명칭을 바꿔서도 사용 가능, js import 와 비슷한 듯?
package strings.lastChar as last

val c = "Kotlin".last()
```
```java
// JAVA 에서 호출하기
char c = StringUtilKt.lastChar("잡아");
```
```kotlin
/**
 * Collection 메소드 확장
 */
fun <T> Collection<T>.joinToString(
  separator: String = ", "
) : String {
  return this.stream().collect(Collectors.joining(separator))
}

println(listOf(1, 2, 3).joinToString())

/**
 * Collection<String> 의 경우에만 메소드 확장
 */
fun Collection<String>.join(
  separator: String = ", "
) = joinToString(separator)
```
```kotlin
// 확장 함수는 override 할 수 없음
fun Animal.showOff() = println("Animal")
fun Tiger.showOff() = println("Tiger")

val animal: Animal = Tiger()
animal.showOff() // Animal
```

### property
- function 과 비슷
- 최소 getter 는 구현해야 함

```Kotlin
var StringBuilder.lastChar: Char
  get() = get(length - 1) // getter
  set(value: Char) {
    this.setCharAt(length - 1, value) // setter
  }
```

## 컬렉션
### `varargs`
- JAVA 에서는 `...`
- 다른점: 배열을 varargs 로 넘길 경우 `spread 연산자 (*)` 로 풀어서 넘겨야함
```Kotlin
fun man(args: Array<String>) {
  println(listOf("args: ", *args))
}
```

### `infix call` (중위 함수 호출 구문)
- 아래 두 구문은 동일 (일반적인 호출, 중위 호출)
> 1.to("one")  
> 1 to "one"
- `to` 는 코틀린 키워드가 아니라 확장 함수
- 메소드를 infix call 이 가능하게 하려면 `infix` 를 선언 시 제일 앞에 추가해야 함
```kotlin
// 간단한 to 함수 정의, 실제는 더 복잡함
infix fun Any.to(other: Any) = Pair(this. other)

// 이 선언 방식을 destructuring declaration 이라고 함 (구조 분해 선언)
val (number, name) = 1 to "one"
```
- `destructuring declaration` (구조 분해 선언): 복합적인 값 분해 -> 여러 변수로 나누기

## 문자열과 정규식
- JAVA 와 대부분 동일
### split 예시
- `"123.456".split(".")` 을 실행하는 경우
  - JAVA 에서는 인자를 **regex 로 인식** 하기때문에 "." 자체의 split 불가능, 헷갈릴 수 있는 여지
  - Kotlin 은 `Regex` 타입을 받는 확장 제공
```kotlin
"123.456-.9".split(".") // 123 456- 9
"123.456-.9".split("\\.".toRegex()) // 123 456- 9
"123.456-.9".split(".", "-") // 123 456 9
```

### 편의를 위한 extension
- `substringBeforeLast`, `substringAfterLast`

### 3중 따옴표
- escape 처리 필요 없음
```kotlin
val regex = """(.+)/(.+)\.(.+)""".toRegex()
val matchResult = regex.matchEntire(path)
if (matchResult != null) {
  val (directory, filename, extension) = matchResult.destructured
  println()
}
```
- 들여쓰기, newline 등 전부 포함
- 제거하고 싶은 경우 구분자 지정 후 `trimMargin` 처리
```kotlin
val kotlinLogo = """| //
                    .| //
                    .| / \"""
println(kotlinLogo.trimMargin("."))
```

## 로컬 함수와 확장
- 함수에서 추출한 함수를 원 함수 내부에 중첩 가능
- 중첩이 많아지면 가독성이 떨어지므로 한 단계만 권장
```kotlin
class User(...)

fun saveUser(user: User) {
  fun validate(value: String, fieldName: String) {
    if (value.isEmtpy()) {
      throw IllegalArgumentException("${user.id}")
    }
  }

  validate(user.name, "Name")
  validate(user.address, "Address")
}
```
```java
// JAVA 단에서는 BiConsumer 으로 사용할 수 있을 듯
class User() {}

public void saveUser(User user) {
  BiConsumer<String, String> validator = (value, fieldName) -> 
    if (value.isEmpty()) throw new IllegalArgumentException(user.id);
  
  validator.consume(user.name, "Name")
  validator.consume(user.address, "Address")
}
```
```javascript
// 참고용 js 예제
function saveUser(user) {
  const validator = (value, fieldName) =>
    if (value === null || value === "") throw `${user.id}`
  
  validator(user.name, "Name")
  validator(user.address, "Address")
}
```