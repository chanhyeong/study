# 13. 서비스 탐구하기

#### 단일 애플리케이션의 문제
- 전체를 파악하기 어려움
- 테스트가 더 어려움
- 라이브러리 간의 충돌이 생기기 쉬움
- 확장 시 비효율적
- 적용할 기술을 결정할 때도 애플리케이션 전체를 고려해야 함
- 프로덕션으로 내기에 많은 노력이 필요

#### 마이크로서비스 아키텍쳐의 특성
- 쉽게 이해할 수 있음
- 테스트가 쉬움
- 라이브러리 비호환성 문제가 없음
- 독자적으로 규모 조정 가능
- 각 MSA 적용할 기술을 다르게 선택 가능
- 언제든 프로덕션으로 낼 수 있음

## Service Registry 설정하기
- Euraka: MSA 에 있는 모든 서비스의 중앙 집중 레지스트리로 작동
- Ribbon: A -> B 의 서비스를 호출 시, A 가 B 의 어떤 인스턴스를 호출할지 결정할 때 사용되는 client load balancer
  - B 의 이름을 갖는 여러 인스턴스 중 선택

#### `eureka.server.enableSelfPreservation`
eurake server 는 service instance (client) 가 등록하고, 등록 갱신하는 것을 30초마다 전송할 것으로 예상  
3번의 갱신 (or 90초) 을 초과하면 instance 등록을 취소 -> **등록된 나머지 서비스 데이터를 보존하기 위해  self-preservation mode** -> 추가적인 service instance 의 등록 취소 방지

---
- (spring config 에 `other.euraka.host` 로 고정하는게 있는데) eureka 는 kubernetes 에서 구동이 불가능한가? -> 찾아보니 가능하긴 한 듯
- `spring.application.name` 을 설정하고 eureka server 에 등록하면, 다른 client 에서 해당 이름으로 호출 가능
  - ex. `some-service` 를 등록하면 다른 client 에서는 `http://some-service/..` 로 호출
  - 근데 이건 kubernetes 에서 Ingress 같은 거로 지원하는거라 이제는 의미가..
  - kubernetes 를 안쓰고 + 초기 사내 서버 툴이 없는 경우에는 그냥 쓸만할 듯