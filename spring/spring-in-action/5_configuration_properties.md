# 5. Configuration Properties 사용하기

## auto configuration 세부 조정하기
- bean wiring: bean 으로 생성되는 애플리케이션 컴포넌트 및 상호 간에 주입되는 방법을 선언
- property injection: bean 의 속성 값을 설정

### Spring environment abstraction
configurable 한 모든 property 를 한 곳에서 관리하는 개념

properties 를 하나로 모아 각 property 가 주입되는 spring bean 을 사용할 수 있게 해줌줌

173 https keytool

## Custom ConfigurationProperties 생성