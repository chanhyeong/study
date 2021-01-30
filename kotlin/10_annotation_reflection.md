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
