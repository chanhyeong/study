# 5. Configuration Properties 사용하기

## auto configuration 세부 조정하기
- bean wiring: bean 으로 생성되는 애플리케이션 컴포넌트 및 상호 간에 주입되는 방법을 선언
- property injection: bean 의 속성 값을 설정

### Spring environment abstraction
configurable 한 모든 property 를 한 곳에서 관리하는 개념

properties 를 하나로 모아 각 property 가 주입되는 spring bean 을 사용할 수 있게 해줌줌

173 https keytool

## Custom ConfigurationProperties 생성
```yml
test:
  pageSize: 10
```

방법 3가지

#### 사용하는 component 에 직접 주입
```kotlin
@Controller
@ConfigurationProperties(prefix = "test")
class OrderController(
  val pageSize: Int = 20
)
```

#### holder 생성
```kotlin
@Component
@ConfigurationProperties(prefix = "test")
data class OrderProps(
  val pageSize: Int = 20
)
```

#### EnableConfigurationProperties
```kt
@Configuration
@EnableConfigurationProperties(OrderProps::class)
class OrderConfiguration(
  val orderProps: OrderProps
)

@ConfigurationProperties(prefix = "test")
data class OrderProps(
  val pageSize: Int = 20
)
```

### configuration property metadata 설정하기
IDE 에 힌트 제공

`spring-boot-configuration-processor` 추가

`src/main/resources/META-INF` > `additional-spring-configuration-metadata.json`
```json
{
  "properties": [
    {
      "name": "test.page-size",
      "type": "int",
      "description": "페이지 당 최대 수"
    }
  ]
}
```

## Profile 설정
https://1minute-before6pm.tistory.com/12

spring boot 2.4 부터 기존 설정이 deprecate 되었고
groups 가 추가됨