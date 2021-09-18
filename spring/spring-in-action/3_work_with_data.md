# 3. 데이터로 작업하기

## JDBC 를 사용한 read/write

spring-jdbc `JdbcTemplate`

insert 후 생성된 id 값을 알기 위한 방법 2가지
1. `PreparedStatementCreator` + `KeyHolder` (`jdbc.upate(psc, keyHolder)`)
2. `SimpleJdbcInsert` (JdbcTemplate 으로 생성)

## JPA 를 사용한 read/write

애플리케이션이 시작될 때 spring-data-jpa 가 각 인터페이스 구현체를 자동으로 생성