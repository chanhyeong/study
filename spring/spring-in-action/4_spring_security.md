# 4. Spring Security
`spring-boot-starter-security`, `spring-security-test`

아무 설정 없이 추가했을 때 기본 제공
- 모든 HTTP request 는 인증되어야 한다
- 특정 역할이나 권한이 없다
- 로그인 페이지가 따로 없다
- http basic auth 를 사용하여 인증된다
- 사용자는 하나만 있고 이름은 user, 비밀번호는 암호화해준다

## 구성하기
spring security 5 부터는 PasswordEncoder 를 사용하여 비밀번호 암호화가 의무

#### user store
1. inmemory
2. JDBC
    - spring security 내부에는 기본적으로 사용하는 table, row 명이 있음
    ```java
    jdbcAuthentication()
      .dataSoruce(dataSource)
      // table, row 명이 다를 때, row data 최종 명칭은 동일해야함
      .usersByUsernameQuery("SELECT username, password, enabled FROM users WHERE username = ?")
      .authoritiesByUsernameQuery("...")
      // 비밀번호 암호화
      .passwordEncoder(new BCryptPasswordEncoder())
    ```
    - 암호화
      - BCrypt, NoOp, Pbkdf2, Scrypt, Standard, or PasswordEncoder 구현
3. LDAP
4. customizing
    - user domain object 와 persistence 정의 (`UserDetails` 인터페이스 구현)
    - `UserDetailsService` 구현

## request 보안 처리하기
`configure(HttpSecurity http)`

### 보안 처리되는 방법을 정의하는 메소드
https://docs.spring.io/spring-security/site/docs/current/reference/html5/#el-common-built-in

`"hasRole('ROLE_USER')"`

### 로그아웃
HttpSecurity 객체의 logout 호출

### CSRF 공격 방어
Cross-Site Request Forgery (크로스 사이트 요청 위조)

악의적인 코드 삽입된 페이지 오픈

`.and().csrf()` 로 방어

## 사용자 인지하기
- `Principal` 을 컨트롤러 메소드에 주입
- `Authentication` 을 컨트롤러 메소드에 주입
- `SecurityContextHolder` 를 사용, 보안 컨텍스트를 얻음
- `@AuthenticationPrincipal` 어노테이션을 메소드에 지정
