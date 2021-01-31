# 10. annotation & reflection

## annotation
```kotlin
// Deprecated 어노테이션에 ReplacedWith 어노테이션을 추가로 받아서 대체 명시
@Deprecated("Use removeAt(index) instead", ReplacedWith("removeAt(index)"))
```
- 다른 어노테이션을 parameter 로 넘길 때는 `@` 을 넣지 말아야함 (위의 `ReplacedWith`)
- 배열을 받는 예시 `@RequestMapping(path = arrayOf("/foo", "/bar"))`
- parameter 는 컴파일 시점에 알 수 있는 타입이어야 함
  - 뒤에 이유가 나옴
  - `Only const val can be used in constant expressions`

### target
- 기본적으로 kotlin property 는 java field, getter 에 대응하는데
- use-site target 선언으로 어노테이션을 추가할 곳을 명시할 수 있음 (getter, field, setter 등)

```kotlin
// @use-site target:annotation
@get:JsonExclude
```

|use-site target|description|
| ----------- | -----------
|**property**|property 전체, JAVA 에서 선언된 어노테이션에선 이 값 사용 불가능|
|**field**|property 에 의해 생성되는 필드|
|**get**|getter|
|**set**|setter|
|**reciever**|extension function or receiver object parameter|
|**param**|consturctor parameter|
|**setparam**|setter parameter|
|**delegate**|delegating instance field of delgating property|
|**file**|파일 내 최상위 함수와 프로퍼티를 담아두는 클래스|

#### JAVA API - Kotlin annotation
> `@JvmName` - kotlin declaration -> java 변환 시 field or method name 변경  
> `@JvmStatic` - method, object declaration, companion object -> java static method  
> `@JvmOverloads` - default parameter  
> `@JvmField` - kotlin property -> public java field

### JSON serialization
- JAVA - `Jackson` or `GSON`
- kotlin native - `Jkid`
  - 여기부터 Jkid 를 예시로 설명이 이어질 예정

```kotlin
data class Person(
  @JsonName("alias") val firstName: String, // Jackson @JsonProperty
  @JsonExclude val age: Int? = null // Jackson @JsonIgnore
)
```

### declare annotation
```kotlin
annotation class JsonName(val name: String)
```
```java
public @interface JsonName {
  String value();
}
```

### meta annotation
- annotation 에 붙는 annotation
- `@Target` - annotation 을 적용할 수 있는 유형
  - class, method, property, ... 등등
  - 명시를 안한 annoation 의 경우 전체 유형에 적용 가능

```kotlin
@Target(AnnoationTarget.ANNOTATION_CLASS)
annotation class BindingAnnotation

@BindingAnnotation
annoation class MyBinding
```

#### `@Retention` 어노테이션
- annotation 을
  - source 수준에서만 유지
  - .class 로 저장
  - runtime 에 reflection 을 사용하여 접근할 수 있을지
- JAVA compiler 에선 기본적으로 .class 에는 저장하지만, runtime 에서는 사용할 수 없음
- kotlin 은 기본적으로 RUNTIME 으로 지정되어 있음

### annoation parameter 로 class, generic class 받기
```kotlin
annotation class DeserializeInterface(val targetClass: KClass<out Any>)

data class Person(
  val name: String,
  @DeserializeInterface(CompanyImpl::class) val company: Company
)

annotation class CustomSerializer(val serializerClass: KClass<out ValueSerializer<*>>)
```

## reflection
- runtime 에 객체의 property, method 에 접근할 수 있게 해주는 방법
- 타입과 관계없이 객체를 다루거나, 객체가 제공하는 method, property 명을 runtime 에서만 알 수 있는 경우 - JSON
- 방법
  - 1 - java.lang.reflect
  - 2 - kotlin.reflect
    - JAVA 를 완전히 대체하진 못함

### kotlin reflection API
- `KClass`: kotlin 에서의 클래스 표현
```kotlin
interface KClass<T: Any> {
  val simpleName: String?
  val qualifiedName: String?
  val members: Collection<KCallable<*>>
  val constructors: Collection<KFunction<*>>
  val nestedClasses: Collection<KClass<*>>
  val memberProperties ...
}
```
- `KCallable`: function, property 를 아우르는 공통 상위 인터페이스
  - call 메소드: function or property getter 호출
```kotlin
interface KCallable<out R> {
  fun call(varagrs args: Any?): R
}

fun foo(x: Int) = println(x)
val kFunction = ::foo
kFunction.call(42)
```
- `KFunctionN`
  - 컴파일 시점에 생성
  - N 개의 parameter 를 받는 function
  - invoke 메소드
```kotlin
import kotlin.reflection.KFunction2

fun sum(x: Int, y: Int) = x + y

val kFunction: KFunction2<Int, Int, Int> = ::sum
kFunction.invoke(1, 2) + kFunction (3, 4)
```
- `KProperty` : get 메소드 제공
- kotlin refection api hierarchy  
![image](https://user-images.githubusercontent.com/10507662/106360307-188d0180-635b-11eb-99f2-bce41f4f2be8.png)

### implementing object serialization using reflection
```kotlin
fun serialize(obj: Any): String

private fun StringBuilder.serializeObject(obj: Any) {
    val kClass = obj.javaClass.kotlin // KClass 
    val properties = kClass.memberProperties // all properties of the class

    properties.joinToStringBuilder(this, prefix = "{", postfix = "}") { prop -> // KProperty<Any, *>
        serializeString(prop.name) // property name
        append(": ")
        serializePropertyValue(prop.get(obj)) // property value
    }
}
```

### customizing serialization with annotations
- `JsonExclude`, `JsonName`, `CustomSerializer` 의 구현
```kotlin
// KAnnotatedElement 의 annotations property - `findAnnotation`
inline fun <reified T> KAnnotatedElement.findAnnotation(): T?
  = annotations.filterIsInstance<T>().firstOrNull()

// findAnnotation 을 이용한 필터링 구현
// @JsonExclude
val properties = kClass.memberProperties.filter { it.findAnnotation<JsonExclude>() == null }

// @JsonName
val jsonNameAnnotation = prop.findAnnotation<JsonName>()
val propName = jsonNameAnnotation?.name ?: prop.name

// @CustomSerializer
annotation class CustomSerializer(
  val serializerClass: KClass<out ValueSerializer<*>>
)

fun KProperty<*>.getSerializer(): ValueSerializer<Any?>? {
  val customSerializerAnnotation = findAnnotation<CustomSerializer>() ?: return null
  val serializerClass = customSerializerAnnotation.serializerClass
  val valueSerializer = serializerClass.objectInstance ?: serializerClass.createInstance()
  @Suppress("UNCHECKED_CAST")
  return valueSerializer as ValueSerializer<Any?>
}
```
- 위 구현들을 포함한 최종 serializer
```kotlin
private fun StringBuilder.serializeProperty(
  prop: KProperty1<Any, *>, object: Any
) {
  val name = prop.findAnnotation<JsonName>()?.name ?: prop.name
  serializeString(name)
  append(": ")

  val value = prop.get(obj)
  val jsonValue = prop.getSerializer?.toJsonValue(value) ?: value
  serializePropertyValue(prop.get(obj))
}

private fun StringBuilder.serializeObject(obj: Any) {
  obj.javaClass.kotlin.memberProperties
    .filter { it.findAnnotation<JsonExclude>() == null }
    .joinToStringBuilder(this, prefix = "{", postfix = "}") {
        serializeProperty(it, obj)
    }
}
```

### JSON parsing & object deserialization
```kotlin
inline fun <reified T: Any> deserialize(json: String): T
```
- 단계별 구분
  - 1 - lexer: lexical analyzer
    - character token, value token 구분
  - 2 - parser: syntax analyzer
    - 1 의 tokin list 를 structured representation 으로 변환

![image](https://user-images.githubusercontent.com/10507662/106375388-e4532880-63ce-11eb-8000-e12a6451ea25.png)

- 여기부터 아래는 너무 구현적이어서 넘어가도 됨
- 객체의 하위 요소를 저장 - builder pattern
  - Jkid 에서는 Seed 라는 명칭을 사용
  - `ObjectSeed`, `ObjectListSeed`, `ValueListSeed`

```kotlin
interface Seed: JsonObject {
  fun spawn(): Any?
  fun createCompositeProperty(propertyName: String, isList: Boolean): JsonObject

  ...
}

fun <T: Any> deserialize(json: Reader, targetClass: KClass<T>): T {
  val seed = ObjectSeed(targetClass, ClassInfoCache(0))
  Parser(json, seed).parse()
  return seed.spawn()
}
```

### 최종 deserialization: callBy(), create object using reflection
- `call`: default parameter 를 지원하지 않음
- `callBy`: default parameter 지원
```kotlin
interface KCallable<out R> {
  fun callBy(args: Map<KParameter, Any?>): R
  ...
}
```

- 이하 생략