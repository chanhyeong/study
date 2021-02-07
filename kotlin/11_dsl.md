# 11. DSL 만들기
- Domain-Specific Language
- lambda with receiver, invoke convention

## from API to DSL
- API 가 깔끔하다는 것이란?
  - 코드를 읽는 대상이 이 코드가 어떤 일을 하는지 명확하게 이해할 수 있어야 함
  - 간결해야함

#### 여태까지 본 kotlin 에서 제공하는 간결한 구문
|일반|간결|특성|
| ----------- | ----------- | ----------- |
|StringUtil.capitalize(s)|s.capitalize()|extension function|
|1.to("on")|1 to one|infix call|
|set.add(2)|set += 2|expression overloading|
|map.get("key")|map["key"]|get method convention|
|file.use({ f -> f.read() })|file.use { it.read() }|lambda outside of parentheses|
|sb.append("yes")|with(sb) { append("yes") }|lambda with receiver|

### DSL 의 개념
- general-purpose programming language
  - 컴퓨터로 풀 수 있는 모든 문제를 충분히 풀 수 있는 기능을 제공하는 범용 프로그래밍 언어
- domain-specifc language
  - 특정 과업 또는 영역에 초점을 맞추고, 그 영역에 필요하지 않은 기능을 없앤 언어
  - SQL, Regular expression 등
- DSL 은 `declarative` (선언적), general language 는 `imperative` (명령적)
  - 결과를 기술하기만 하고 세부 실행은 언어를 해석하는 엔진에 맡김
  - 엔진이 결과를 얻는 과정을 최적화하기 때문
- 단점으로는 DSL 과 general language 는 **조합이 어렵다**
  - 호스트 프로그램과 DSL 의 상호작용을 컴파일 시점에 검증이 어려움
  - 두 언어 모두 배워야 함, 코드를 읽기 어려울 수도
  - 예시) SQL 문과 그걸 실행하는 서버 언어
- 이런 문제 해결 + DSL 의 다른 이점을 살리는 방법 -> **Internal DSL**

### Internal DSL
- general language 로 작성된 프로그램의 일부, 동일한 문법을 사용
```sql
SELECT Country.name, COUNT(Customer.id)
FROM Country
JOIN Customer ON Country.id = Customer.country_id
GROUP BY Country.name
ORDER BY COUNT(Customer.id) DESC
LIMIT 1
```
```kotlin
// Kotlin Exposed Framework
(Country join Customer)
  .slice(Country.name, Count(Customer.id))
  .selectAll()
  .groupBy(Country.name)
  .orderBy(Count(Customer.id), isAsc = false)
  .limit(1)
```

### DSL 의 구조
- Typical library: multiple methods
  - 사용자는 한 번에 하나씩만 호출, 서로 다른 호출 간에는 아무런 context 가 없음
  - command-query API
- DSL: DSL grammer 에 의함
  - nested lambda or method chaining 으로 구조를 만듦
  - 읽기 쉬움
- 장점: 같은 context 를 재사용할 수 있음
```groovy
// nested lambda
dependencies {
  compile("junit:junit:4.11")
  compile("come.google.inject:guice:4.1.0")
}

// commend-query
project.dependencies.add("complile", "junit:junit:4.11")
project.dependencies.add("complile", "come.google.inject:guice:4.1.0")
```

## Structured API 구축
### lambda with receiver, extension function type
```kotlin
// 1. buildString function 선언
fun buildString(
  builderAction: StringBuilder.() -> Unit // extension function type parameter with receiver, 여기서 parameter 를 extension function 으로 만들어서
) : String {
  val sb = StringBuilder()
  sb.builderAction() // StringBuilder instance as lamdba receiver, 여기서 받아서 수행
  return sb.toString()
}

// 2. 사용 -> Hello, World!
val s = buildString(
  append("Hello, ")
  append("World!")
)

// Receiver Type . Paremeter Type -> Return Type
String.(Int, Int) -> Unit

// 3. buildString 을 더 간결하게
fun buildString(builderAction: StringBuilder.() -> Unit): String =
  StringBuilder().apply(builderAction).toString()

// 4. apply 와 with 복기
inline fun <T> T.apply(block: T.() -> Unit): T {
  block() // block 을 호출하고
  return this // 그대로 반환
}

inline fun <T, R> with(receiver: T, block: T.() -> R): R =
  receiver.block() // receiver 와 block 같이 받고 block 을 호출하여 반환
```

- DSL 로 HTML 을 만드는 간단한 예시 (상세 내용은 너무 길어서 생략)
  - 운영툴 만들기 좋을 듯
  - 뭔가 jade 랑 비슷한.. 거 같은건 잘못 생각한건가
```kotlin
// <table><tr><td></td></tr></table>
fun createSimpleTable() = createHTML().
  table {
    tr {
      td {
      }
    }
  }
```

## invoke convention - flexible nested block
- 객체를 함수처럼 호출

### invoke convention
```kotlin
data class Issue(
  val id: String, val project: String, val type: String,
  val priority: String, val description: String
)

class ImportantIssuesPredicate(val project: String): (Issue) -> Boolean { // function type as parent class
  override fun invoke(issue: Issue): Boolean { // implement "invoke"
    return issue.project == project && issue.isImportant()
  }
  private fun Issue.isImportant(): Boolean {
    return type == "Bug" && (priority == "Major" || priority == "Critical")
  }
}

val i1 = Issue("IDEA-154446", "IDEA", "Bug", "Major", "Save settings failed")
val i2 = Issue("KT-12183", "Kotlin", "Feature", "Normal", "Intention: convert several calls on the same receiver to with/apply")
val predicate = ImportantIssuesPredicate("IDEA") // (Issue) -> Boolean 타입의 객체 생성
for (issue in listOf(i1, i2).filter(predicate)) { // filter 로 넘김
  println(issue.id)
}
IDEA-154446
```

- gradle 에서의 예시
```kotlin
class DependencyHandler {
  fun compile(coordinate: String) {             
    println("Added dependency on $coordinate")
  }
  operator fun invoke(body: DependencyHandler.() -> Unit) {  
    body()                                  
  }
}

val dependencies = DependencyHandler()
dependencies.compile("org.jetbrains.kotlin:kotlin-stdlib:1.0.0")
Added dependency on org.jetbrains.kotlin:kotlin-stdlib:1.0.0

// 이거는
dependencies {
  compile("org.jetbrains.kotlin:kotlin-reflect:1.0.0")
}
Added dependency on org.jetbrains.kotlin:kotlin-reflect:1.0.0

// 이렇게 컴파일됨
dependencies.invoke({
    this.compile("org.jetbrains.kotlin:kotlin-reflect:1.0.0")
})
```

## Kotlin DSL in practice
### infix call chaining
- `kotlintest` 를 예시로 듦

```kotlin
interface Matcher<T> {
  fun test(value: T)
}

infix fun <T> T.should(matcher: Matcher<T>) = matcher.test(this)

class startWith(val prefix: String) : Matcher<String> {
  override fun test(value: String) {
    if(!value.startsWith(prefix)) throw AssertionError("String $value does not start with $prefix")
  }
}

// 위에 선언한 기반으로 테스트 수행
s should startWith("kot")
```
```kotlin
// method chaining
object start

infix fun String.should(x: start) : StartWrapper = StartWrapper(this)

class StartWrapper(val value: String) {
  infix fun with(prefix: String) = if (!value.startsWith(prefix)) throw AssertionError("String $value does not start with $prefix") else Unit
}

"kotlin" should start with "kot"
"kotlin".should(start).with("kot")
```
- `infix call` + `object` (singleton object instance) 를 조합하면 DSL 에 복잡한 문법을 도입할 수 있음

### extension function about primitive type
```kotlin
val yesterday = 1.days.ago
val tomorrow = 1.days.fromNow

val Int.days: Period // primitive Int 에 대한 extension
  get() = Period.ofDays(this)

val Period.ago: LocalDate
  get() = LocalDate.now() - this
val Period.fromNow: LocalDate
  get() = LocalDate.now() + this
```

### member extension function
- 클래스 안에서 extension function, extension property 를 선언하는 것
```kotlin
class Table {
  fun integer(name: String): Column<Int>
  fun varchar(name: String, length: Int): Column<String>
  ...
  // extension 의 범위를 클래스 내로만 지정하기 위해 아래와 같이 extend
  fun <T> Column<T>.primaryKey(): Column<T>
  fun Column<Int>.autoIncrement(): Column<Int>
}
```