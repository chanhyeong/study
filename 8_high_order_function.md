# 8. 고차 함수
- 간략하게 넘김

## high order function
- lambda or method reference 를 인자로 넘길 수 있거나, 반환하는 함수

### function type
#### lambda 를 로컬 변수로 선언하는 경우
```kotlin
val sum = {x : Int, y: Int -> x + y} // (Int, Int) -> Int = { x, y -> x + y }
val action = { println(42) } // () -> Unit = { println(42) }
```

### parameter 로 함수 받기
```kotlin
fun test(operation: (Int, Int) -> Int) {
  val result = operation(2, 3) // 일반적인 함수 호출 방법과 같음
  println(result)
}
```

### JAVA 에서 kotlin function type 사용
- function type -> (컴파일) -> 일반 인터페이스 FunctionN
  - N 은 인자의 개수
  - 각 인터페이스에는 `invoke` 메소드 정의

```kotlin
// 코틀린에서 람다를 인자로 받음
fun method(f: (Int) -> Int) {
  println(f(42))
}
```
```java
// JAVA 8+ 에서 사용
method(num -> num + 1) // 43

// JAVA 7 이하에서 사용
method(
  new Function1<Integer, Integer>() {
    @Override
    public Integer invoke(Integer num) {
      return number + 2;
    }
  }
) // 44

// JAVA 에서 kotlin lambda 로 넘기기
CollectionsKt.forEach(Collections.singletonList("42"), s -> {
  log.debug(s)
  return Unit.INSTANCE; // void 를 넘길 수 없고, kotlin Unit 을 명시해야 함
})
```

### function type parameter 에 대한 default value 와 nullable 
- 이전에 본 다른 방식과 같음
  - `=` 명시
  - `?` 명시

### function 을 반환하는 function
- 일반적인 선언처럼 리턴 타입으로 lambda 를 명시

### lambda 를 활용한 중복 제거
```kotlin
data class SiteVisit( // 방문 데이터
  val path: String,
  val duration: Double,
  val os: OS
)

enum class OS { WINDOWS, LINUX, MAC, IOS, ANDROID }

val log = listOf( ... )

// predicate 에 해당하는 체류 시간 평균 구하기
fun List<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean) =
  filter(predicate).map(SiteVisit::duration).average()

log.averageDurationFor { it.os == OS.IOS && it.path == "/signup" }
```

## inline function
- lambada 를 활요한 코드의 성능은?
  - inner class 로 컴파일하지만, 사용할 때마다 새로운 클래스가 생성되지는 않는다
  - 변수를 capture 하면, 생성되는 시점마다 새로운 inner class object 가 생성된다 (부가 비용 발생)