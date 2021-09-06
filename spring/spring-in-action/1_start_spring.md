# 1. 스프링 시작하기
### Spring application context 
- container - application components (bean) 를 생성 및 관리
- bean 은 context 내부에서 서로 연결

### Dependency Injection
- bean 의 상호 연결
- bean 에서 의존하는 다른 bean 의 생성과 관리를 container 가 해주고
- container 는 모든 components 를 생성, 관리 + 필요로 하는 bean 에 injection

### `@Configuration`
- 각 bean 을 context 에 제공하는 클래스라고 spring 에 알려줌
- 내부 메소드는 `@Bean` 지정 - context 의 bean 으로 추가되어야 한다
  - bean id 는 메소드명과 동일

### Auto configuration
- autowiring + component scanning
- component scanning: 자동으로 classpath 에 지정된 컴포넌트를 찾아 context 의 bean 으로 생성
- autowiring: 의존 관계가 있는 컴포넌트를 자동으로 다른 bean 에 주입

### Application Bootstrap
executable JAR 에서 애플리케이션 실행 - 제일 먼저 시작되는 부트스트랩 클래스 `Application.java`

#### `@SpringBootApplication` 은 3개가 결합된 annotation
- `@SpringBootConfiguration`: 해당 클래스를 Configuration class 로 지정
- `@EnableAutoConfiguration`: auto configuration 활성화
- `@ComponentScan`: component scan 활성화.
  - `@Component, @Controller, @Service` 등의 annotation 과 함께 클래스를 선언할 수 있게 해주고
  - 그런 클래스를 찾아 context 에 bean 으로 등록

#### `@SpringBootTest`
- Spring boot 기능으로 테스트 시작