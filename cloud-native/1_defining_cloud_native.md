# 1. '클라우드 네이티브' 로 정의한 단어 계속 사용하기

## 1.1 최근 애플리케이션 요구 사항
- 무중단 (Zero downtime)
- 짧아진 피드백 주기 (Shorten feedback cycles)
- 모바일과 다중 장치 지원 (Mobile and multidevice support)
- Connected devices—also known as the Internet of Things
- Data-driven

## 1.2 클라우드 네이티브 소프트웨어 소개
### Cloud-native 정의
- **인프라 장애 및 변경에 항상 대처할 수 있어야 함**
- **smaller, loosely coupled, dependently deployable and releasable component**
- **scales dynamically, continues to function**

극한의 분산 + 끊임없는 변화 (extreme distribution and constant change)

![image](https://user-images.githubusercontent.com/10507662/144228266-12b3bb8e-474e-439d-a598-60fd458b74c1.png)

## A mental model for Cloud-native Software
#### Cloud-native App
비즈니스 로직인 코드가 작성되는 곳  
확장, 업그레이드 같은 Cloud-native 운영이 적용될 수 있는 방식으로 구축

#### Cloud-native Data
Cloud-native Software 의 데이터가 있는 곳  
코드를 여러 작은 module (app) 으로 나누며, DB 도 비슷하개 분해되고 분산

#### Cloud-native Interactions
App 과 Data 의 상호작용하는 방식 - 기능과 품질을 결정

![image](https://user-images.githubusercontent.com/10507662/144229812-c8e11762-811a-4eee-a866-b7d821ba50c2.png)

### Cloud-native App
- instance 를 추가/제거하여 가용성 조절 - scale out/in
    - 수를 조정하여 안정적이지 않은 환경에서도 resilience 를 유지할 수 있음
- app configuration 은 instance 가 구축되고 지속적으로 변경될 때 문제
    - 새 env 를 알려진 파일 시스템에 놓고 재시작할 일은 없을 것 - 분산된 토폴로지 전체에서 옮겨질 수도 있다
- application lifecycle 을 관리하는 방식이 변경되어야 함
    - 새로운 context 에서 app 의 start, configure, reconfigure, shut down 방법 다시 검토

### Cloud-native Data
- data monolith 분리해야 한다 -> distributed database fabric
    - independent, fit-for-purpose database (polyglot database)
    - 일부 DB 의 경우 VIEW 로만 구성
    - caching 은 Cloud-native software 의 핵심 패턴이자 기술
- 여러 DB 에 존재해야 하는 엔티티의 경우, 여러 instance 에 걸쳐 있는 공통된 정보를 동기화 상태로 유지하는 방법을 설명해야 함
- 상태를 이벤트의 결과로 다루는 것이 distributed database fabric 의 핵심
    - event sourcing pattern 은 state-change event 캡처
    - unified log 는 state-change event 를 수집해 분산 멤버들이 사용할 수 있도록 함

### Cloud-native interactions
- routing system 필요 - `synchronous request/response`, `asynchronous event-driven patterns`
- 접근 시도가 실패하는 것을 고려해야 함. automatic retry 는 필수
    - circuit breaker 는 재시도가 필요할 때 필수
- Cloud-native software 는 복합 소프트웨어이므로 다양한 서비스 호출
    - 좋은 사용자 경험을 위해서는 서비스-서비스 간 상호작용를 적절히 관리
    - application metric, logging 은 여기에 특수화되어야 함
- module system 일부를 독립적으로, 쉽게 개발
    - protocol 은 Cloud-native context 에 적합해야 함

## Cloud-native and world peace
### Cloud and Cloud-native
**Cloud 는 where, Cloud-native 는 how**

on-premise 환경에서도 Cloud-native solution 을 구현하는 것은 가능하다

### Cloud-native 가 아닌 것은?
Cloud-native 를 사용하지 않는 일반적인 3가지 이유

1. 요구하지 않는 경우
    - 배포되고 거의 변경되지 않는 경우 - 안정성 수준 확신
    - 세탁기, 밥솥, ...
2. Cloud-native software 의 특성이 적합하지 않는 경우
    - eventual consistency 를 제공
        - 새로운 값이 없으면 최종적으로 업데이트된 값을 반환하는 것을 보장하는 고가용성을 달성하기 위해 분산 컴퓨팅 환경에서 쓰이는 일관성 모델
        - optimistic replication
        - 언젠가는 모두 일치됨
3. 다시 개발해야 하는 가치가 없는 경우

### Cloud-native plays nice
- all or nothing 이 아님
- 이미 실행되고 있는 소프트웨어의 상당 부분이 완전히 Cloud-native 가 되지는 않을 것  
- 하지만 일부분이라도 적용은 가능
- 리팩토링하고 싶은 legacy 가 있을 때, 단번에 할 필요는 없음
    - netflix 는 7년 동안 일부분을 리팩토링하여 이점을 얻음