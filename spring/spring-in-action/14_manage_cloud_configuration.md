# 14. 클라우드 구성 관리

## Spring cloud config server
```kt
implementation("org.springframework.cloud:spring-cloud-config-server:${version}")

@EnableConfigServer
```

```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/....
          # mark sub directory
          search-paths: config, database
          # mark label / branch
          default-label: sidework
          # authentication
          username: username
          password: password
```

- `http://localhost:8888/application/default/master`
- `http://{configuration server host:port}/{application name}/{activated spring profile}/{(optional) git label or branch}`

# Spring cloud config client

```kt
implementation("org.springframework.cloud:spring-cloud-config:${version}")
```

```yml
spring:
  cloud:
    config:
      uri: https://config.customcloud.com:8888
```

## Application or Profile 별 Configuration
### Application
- Configuration file 명칭에 application name 을 기입
- ex) A.yml, B.yml

### Profile
- Configuration file 뒤로 profile 명시
- ex) A-real.yml, B-real.yml, application-real.yml

## Encryption
필요할 때 찾아서 쓰자

## Refresh configuration properties in realtime
### Manual
- `spring actuator` 의 `/actuator/refresh` 엔드포인트 수동 호출

### Automatic
- `spring cloud bus` 를 활용하여 publish-subscribe 형태로 자동 갱신
- 이 때 중간 브로커로 RabbitMQ, Kafka 등이 필요
- Flow
  - git push -> git webhook -> config server publish -> Kafka or RabbitMQ -> config client poll