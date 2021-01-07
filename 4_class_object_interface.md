# 4. 클래스, 객체, 인터페이스
- `static` 개념이 없음
- 인터페이스에 프로퍼티 선언 가능
- 기본으로 `final`, `public`
- `data class` 는 `toString, equals, hashCode` 를 자동으로 생성

## 클래스 정의
### 인터페이스
- JAVA 와 거의 동일
  - 기본 구현 추가 가능 (JAVA default method)
  - 인터페이스는 다수 상속 가능, 클래스는 1개만
- **`override` 는 무조건 명시해야 함**

#### 상속한 서로 다른 인터페이스의 메소드 명이 겹치는 경우
- `super<클래스명>.메소드()` 로 명시적으로 선택해야 함
  - 미구현 시 컴파일 에러
```kotlin
interface Clickable() {
  fun showOff() = println("clickable");
}

interface Focusable() {
  fun showOff() = println("forcusable");
}

class Button : Clickable, Focusable {
  override fun showOfF() {
    super<Clickable>.showOff() // super<클래스명>.메소드()
    super<Focusable>.showOff()
  }
}
```
### open, final, abstract
- 클래스와 메소드는 기본적으로 `final`
- 상속을 허용하려면 `open` 을 명시
- 의도
  - Effective Java - `상속을 위한 설계와 문서를 갖추거나, 그럴 수 없다면 상속을 금지하라`
```kotlin
open class RichButton : Clickable {
  fun disable() {} // final
  open fun animate() {} // 상속 가능
  override fun showOfF() { } // 상속 가능 (override 메소드는 기본적으로 open 상태)
}

open class RichButton : Clickable {
  final override fun showOfF() { } // 상속 불가능 (override 메소드 상속 제한)
}
```

- `abstract` 로 선언한 클래스
  - 인스턴스로 만들지 못함
  - 기본적으로 `open` 상태, 명시할 필요 없음
```kotlin
abstract class Animated {
  abstract fun animate() // 상속 시 반드시 override

  open fun stopAnimating() { } // 구현 override 가능

  fun animateTwice() { } // override 불가능
}
```

### 가시성
- JAVA 의 `package-private` 없음, 디폴트는 `public`

|변경자|클래스 멤버 가시성|최상위 클래스 선언 시 가시성|
| ----------- | ----------- | ----------- |
|public|모든 곳|모든 곳|
|internal|같은 모듈 내|같은 모듈 내|
|protected|하위 클래스에서만|(선언 불가능)|
|private|같은 클래스에서만|같은 파일에서만|

#### 기타 (package)
- Kotlin: 네임스페이스를 관리하기 위한 용도로만 사용, 가시성 제어에는 미사용
- Java: 가시성 제어에 사용

### Inner class, Nested class
- 외부 참조: 대상을 선언한 클래스의 한 단계 껍데기
- Inner class: 외부 참조를 갖는 클래스
- Nested class: 외부 참조가 없는 클래스
- 개념 이해하기: https://siyoon210.tistory.com/141

|구분|JAVA|Kotlin|
| ----------- | ----------- | ----------- |
|Nested|`static class A`|`class A`|
|Inner|`class A`|`inner class A`|

- Inner class 의 외부 참조를 가져올 경우 `this@Outer` 로 가져와야 함
```kotlin
class Outer {
  inner clas Inner {
    fun getOuterReference(): Outer = this@Outer
  }
}
```

### sealed
- sealed class 를 상속하려면, 반드시 Nested class 로 만들어야 함 (Kotlin 1.0 기준)
- sealed class 는 디폴트로 `private` 생성자를 가짐
- Kotlin 1.1 부터 같은 파일에서의 상속은 가능하도록 변경
```kotlin
sealed class Expr {
  class Num(val: value) : Expr()
  class Sum(val left: Expr, val right: Expr) : Expr()
}

// 불가능
class Multiply : Expr() { }
```

## 클래스 선언
- primary constructor, secondary constructor
- initializer block (초기화 블록)
- `new` 키워드 없이 constructor 호출
- named parameter, default parameter 

### primary constructor, initializer block
```kotlin
class User(val nickname: String) // primary constructor
class User constructor(_nickname: String) { // consturctor 생략 가능
  val nickname: String // 초기화 블록 없이 val nickname = _nickname 로 쓸 수 있음

  init { // 초기화 블록
    nickname = _nickname
  }
}

// 상속 시 예제, 상위 클래스의 생성자 호출
class TwitterUser(nickname: String): User(nickname) { }
```

### secondary constructor
- 사용할 일은 거의 없음, JAVA 와의 상호운용성을 위해 주로 사용
- 예시) 프레임워크의 특정 코드를 extend 하는 경우

```kotlin
class MyButton: View {
  constructor(ctx: Context): this(ctx, MY_STYLE) { ... }

  constructor(ctx: Context, attr: AttributeSet: super(ctx, attr) { ... }
}
```

### Interface 에 property 구현
```kotlin
interface User {
  val nickname: String
}

// 직접 선언
class PrivateUser(override val nickname: String): User

// 커스텀 getter
class SubscribingUser(val email: String): User {
  override val nickname: String
    get() = email.substringBefore("@")
}

// 프로퍼티 초기화
class FacebookUser(val accountId: Int): User {
  override val nickname = getFacebookName(accountId)
}
```

### getter, setter 에서 backing field (뒷받침 필드) 접근
- 값을 변경, 읽을 때마다 수행되어야하는 로직이 있는 경우

```kotlin
class User(val name: String) {
  var address: String = "unspecified"
    set(value: String) {
      println("""
        Address was changed for $name:
        "$field" -> "$value".""".trimIndent()) // field 값 읽기
      field = value // field 값 변경
    }
  
  var visitCount: Int = 0
    private set // 이 클래스 밖에서 값 변경 불가능
  
  fun visit() {
    visitCount++
  }
}
```

## data class, delegation
### 클래스 주요 메소드
- JAVA 와 마찬가지로 기본적인 메소드들이 있고, 코틀린도 마찬가지
- JAVA 에서는 이런 귀찮은 것들을 `projectlombok` 을 이용해서 해결하긴 하지만
  - 라이브러리 의존성 추가해야 함
  - 코드 1줄 씩 들어가야 함 (`@ToString, @EqualsAndHashcode`)
- 아래는 일반적인 JAVA 와 동일한 구현 관련 내용

```kotlin
class Client(val name: String, val postalCode: Int)
```

#### toString()
```kotlin
override fun toString = "Client(name=$name, postalCode=$postalCode)"
```

#### equals()
> kotlin 에서는 객체 비교 시 `==` 도 사용 가능  
> (사용 시 equals 를 호출해주며, 구현되어 있어야 정확한 비교)
```kotlin
val client = Client("name", 4122)
val client2 = Client("name", 4122)

print(client == client2) // false
```
```kotlin
override fun equals(other: Any?): Boolean {
  if (other !is Client)
    return false
  
  return name == other.name && postalCode == other.postalCode
}
```

#### hashCode()
- `equals()` 를 구현하더라도 일치하지 않는 경우가 있음
- JVM 제약
  - equals() 가 true 를 반환하는 두 객체는 반드시 같은 hashCode() 를 반환해야 한다
```kotlin
val clientSet = hashsetOf(Client("name", 4122))
print(clientSet.contains(Client("name", 4122))) // false
```
```kotlin
override fun hashCode(): Int = name.hashCode() * 31 + postalCode
```

### data class
- `toString()`, `equals()`, `hashCode()` 를 자동으로 생성
```kotlin
data class Client(val name: String, val postalCode: Int)
```

#### copy()
- 객체를 그대로 복사하면서, 일부 프로퍼티를 변경하기
- `override` 가 붙지 않는 것을 봐선 정식 기능으로 뭔가 있는건 아닌 듯
```kotlin
data class Client(val name: String, val postalCode: Int) {
  fun copy(name: String = this.name, postalCode = this.postalCode) = Client(name, postalCode)
}
```

### delegation (위임)
- 상속이 불가능한 클래스에 새로운 동작을 추가해야 할 때 -> **`by`** 키워드
- **Decorator 패턴**
  - 상속 불가능한 클래스 대신 새로운 클래스 생성
  - 기존 클래스와 같은 인터페이스를 Decorator 가 제공
  - 기존 클래스를 Decorator 내부에 필드로 유지
  - 기존 기능이 필요한 부분은 Decorator 의 메소드가 기존 클래스에 forwarding
- 기존 서비스 구현 예시 (`kin`): `WebClientFactory`

```kotlin
class DelegatingCollection<T>(
  innerList: Collection<T> = ArrayList<T>()
): Collection<T> by innerList { } // { } 블록 안에 있는 메소드 추가 or 재정의

class CountingSet<T>(
  val innerSet: MutableCollection<T> = HashSet<T>()
): MutableCollection<T> by innerSet {
  var objectsAdded = 0

  override fun add(element: T): Boolean {
    objectsAdded++
    return innerSet.add(element)
  }

  override fun add(c: Collection<T>): Boolean {
    objectsAdded += c.size
    return innerSet.addAll(c)
  }
}
```

## 클래스 선언과 인스턴스 생성
- object 선언 - singleton
- companion object - 연관 메소드, 팩토리 메소드
- 객체 식

### object 선언
- 프로퍼티, 메소드, 초기화 블록이 들어갈 수 있음
  - primary, secondary constructor 는 불가능
- 클래스, 인터페이스 상속 가능

```kotlin
object CaseInsensitiveFileComparator: Compartor<File> {
  override fun compare(file1: File, file2: File): Int {
    return file1.path.compareTo(file2.path, ignoreCase = true)
  }
}

val files = listOf(File("/Z"), File("/a"))
print(files.sortedWith(CaseInsensitiveFileComparator)) // [/a, /Z]
```

#### kotlin object 를 JAVA 에서 사용
```java
CaseInsensitiveFileCompartor.INSTANCE.compre(file1, file2)
```

### companion object
- JAVA 의 static 메소드, static field 에 대응
```kotlin
class User private constructor(val nickname: String) {
  companion object {
    fun newSubscribingUser(email: String) = User(eamil.substringBefore("@"))
    fun newFacebookUser(accountId: Int) = User(getFacebookName(accountId))
  }
}

User.newSubscribingUser("foo@bar.com)
User.newFacebookUser(4)
```

- 클래스 안에 정의된 일반 객체
  - 이름을 붙일 수 있음, 붙이지 않은 경우 `Companion`
  - 인터페이스 상속 가능
  - 확장 함수와 프로퍼티 정의 가능

#### 인터페이스 확장
```kotlin
interface JSONFactory<T> {
  fun fromJSON(jsonText: String): T
}

class Person(val name: String) {
  companion object: JSONFactory<Person> {
    override fun fromJSON(jsonText: String: Person = ...
  }
}
```

#### 동반 객체 확장
- 위 코드를 분리
  - Person 은 도메인이므로 fromJSON 은 통신 모듈 쪽에 정의

```kotlin
// Person
class Person(val name: String) {
  companion object {
    
  }
}

// 통신 모듈
fun Person.Companion.fromJSON(json: String): Person {
  ...
}

val p = Person.fromJSON(json)
```

### 객체 식 (object expression)
- anonymous 객체를 정의할 때도 `object` 키워드 사용
- singleton 은 아님
- 객체 식 내에 포함된 경우 식이 포함된 함수의 변수에 접근할 수 있음
  - \+ final 이 아닌 변수도 사용 가능 (JAVA 는 불가능)

```java
window.addMouseListener(
  new MouseAdapter() {
    @Override
    public mouseClicked(MouseEvent e) { }
    
    @Override
    public mouseEntered(MouseEvent e) { }
  }
)
```
```kotlin
window.addMouseListener(object: MouseAdapter( {
  override fun mouseClicked(e: MouseEvent) { }
  override fun mouseEntered(e: MouseEvent) { }
}))

// 변수 접근 예
fun countClicks(window: Window) {
  var clickCount = 0

  window.addMouseListener(object: MouseAdapter( {
    override fun mouseClicked(e: MouseEvent) {
      clickCount++;
    }
}))
}
```