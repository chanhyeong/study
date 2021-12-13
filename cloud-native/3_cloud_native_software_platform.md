# 3. 클라우드 네이티브 소프트웨어 플랫폼
cloud-native platform 이 어떤 기능을 가지는지

## 클라우드에서 시작
2006년 AWS - EC2 (Elastic Computing Cloud), S3 (Simple Storage Service), message service  
- 개발자, 운영자가 자체 하드웨어를 가지고 관리 필요 없음
- 인프라가 불안정하더라도 소프트웨어가 유지될 수 있도록 availability zones (AZs) 를 제공

구글 - 웹 애플리케이션 실행을 위한 GAE (Google App Engine)
- AWS 는 컴퓨팅, 스토리지, 네트워크 기본 요소 노출
- 구글은 인프라 자산을 직접 노출하지 않음 

MS - medium trust code 기능 제공
- 컴퓨팅, 스토리지, 네트워크에 직접 접근할 수 없고, 사용자의 코드가 실행되는 인프라를 만듦 (코드가 인프라 내에서 할 수 있는 것을 제한)

#### Cloud-native platform 의 형성으로 이어짐
높은 수준의 추상화

## Cloud-native dial tone
dial tone - 해당 영역임을 알아차리게 하는 독특한 의미 (정도로 해석)

#### Cloud-native platform 이 운영자가 떠안았던 짐들을 담당
- 애플리케이션 topology 를 이해
- topology 를 활용하여 모든 인스턴스의 로그 집계
- 운영자에게 관심있는 엔티티에 필요한 데이터 제공
    - 로그를 보관하는 디렉토리는 관심사가 아니고 (인프라 중심)
    - 애플리케이션의 로그가 필요 (애플리케이션 중심)

아래와 같은 레이어로 구분됨

Software  
\------- (Application dial tone)  
Cloud-native platform  
\------- (Infrastructure dial tone)  
IaaS

#### Application dial tone
- 앱의 설정, 모니터링, 보안, 운영
- 데이터의 구조, 접근, 캐시 (Not DB)
- 앱과 데이터의 연결 및 조정
- ex) Google App Engine, AWS Elastic Beanstalk, Azure App Service

#### Infrastructure dial tone
- VM 과 Storage device 의 프로비저닝 및 설정
- 장애 도메인 (server rack) 에 매핑하는 가용 영역 정의
- 시스템 경계에서의 방화벽 규칙 설정
- 시스템 로그 검사

## Cloud-native platform 의 핵심 원리
철학적 토대와 기본 원리

### Container
- 호스트의 기능을 사용하는 computing context
- 여러 컨테이너는 서로 격리
- VM 보다는 가벼움 - 더 짧은 시간 내 생성 및 더 적은 리소스 소비

1. 인프라에 여러 호스트 (PM or VM) 이 있고
2. 호스트에 여러 컨테이너가 실행되고 있으며
3. 애플리케이션은 기능을 위해 설치된 OS 와 runtime 을 사용한다

Container 를 사용하는 Cloud-native platform 은 다음과 같은 기능 제공
- 애플리케이션 인스턴스 생성
- 애플리케이션 상태 모니터링
- 인프라 전체에 적절한 애플리케이션 인스턴스 배포
- 컨테이너에 IP 주소 할당
- 앱 인스턴스로의 동적 라우팅
- 환경 설정 주입
- etc.

### Constantly changing 을 지원
실행 중인 플랫폼에 문제가 발생해도 애플리케이션이 어떻게 안정적으로 유지될 수 있는지

개발자가 복원력을 달성하는데 중요한 역할을 하지만, 모든 안정성 기능을 직접 구현할 필요는 없다

#### AWS 예시
- availability zone 은 EC2 서비스 사용자에게 노출하는 추상화지만  
Cloud-native platform 사용자에게 노출될 필요는 없다
- 대신 해당 availability zone 에서의 인스턴스 orchestration 은 플랫폼에 의해 처리된다
- 더 자세한 개인적인 예시
    - 춘천, 세종에 배포하는걸 굳이 배포하는 사용자에게 알려줄 필요가 없다
    - 춘천이 죽어도 같은 수의 인스턴스가 세종에 배포되면 정상 동작하는 것

#### 끊임없이 변화하는 상황에서의 Eventual consistency
지속적으로 모니터링해 희망 상태와 비교, 필요한 경우 조정  
구현이 어려우니 Cloud-native platform 을 통해 기능을 실현하는 것이 필수

Kubenetes, Cloud Foundry 등의 여러 플랫폼은 이런 패턴을 구현 중

(그림 3.8) Platform API, State Storage, Comparator, Scheduler - 주요 actor 와 그 사이의 흐름

- 플랫폼에 구현된 알고리즘은 여러 방해 요소로 인한 상태 변화를 고려해야 함
- State Storage 와 같은 컴포넌트는 입력값이 충돌할 때 상태를 유지할 수 있는 방법이 있어야 한다
    - Paxos, Raft protocol 등 (분산 환경에서 상태를 공유하기 위한 합의 프로토콜)
    - (개인 추가) kafka 도 KRaft 로 갈아타는 움직임
- 플랫폼은 복잡한 분산 시스템이며, 복원력이 있어야 한다

### Highly distributed 를 지원
singleton or internal process 로 구성했던 것을 분산 컴포넌트로 구성 -> 복잡성 발생

다양한 케이스
- 다른 컴포넌트와 통신해야하는 경우, 다른 컴포넌트는 어디서 찾아야 하는지?
- 앱이 scale out 되면 재부팅 없이 모든 인스턴스의 환경 설정을 변경할 수 있는 방법
- 사용자 요청이 12개 MSA 를 통과할 때 tracing
- DDoS 발생 시 retry 유지

#### Service discovery
방법
1. www 의 패턴인 DNS, routing 을 통하여 처리
2. 하나의 서비스가 다른 서비스의 IP address 들을 알고 load balancing

#### Service environment configuration
1. 사용자가 설정 변경
2. 인스턴스는 시작 시 config 서비스에 접근, 값 변경이나 이벤트 발생 시에도 접근
3. 설정이 변경되면 인스턴스를 자체적으로 새로 고치게 함
    - 플랫폼이 모든 인스턴스를 알고 있음
4. 플랫폼은 인스턴스에 새로운 값을 사용할 수 있음을 알림
    - 인스턴스는 새로운 값을 가져옴

위와 같은 프로토콜을 구현할 책임이 없고, 클라우드 네이티브 플랫폼에 배포된 앱에 자동으로 제공  
(근데 결론은 설치랑 연동은 다 해야하고 어떻게 동작하는지 파악은 해야하긴 하는데)  
(심지어 spring cloud config 는 메시지 브로커 설치도 해야 함)

### 기타
- tracer - 분산 추적 메커니즘
- retry 폭증으로 이한 공격 방지 - Circuit breaker

## 누가 무엇을 하는가
#### Cloud-native platform 과 software 사이의 경계선
다음과 같은 계약
- software 와 software topology 를 지정할 수 있는 API
- 애플리케이션을 모니터링하고 관리할 수 있는 API
- 소프트웨어 복원력과 같은 영향을 미치는 Service Level Agreement
- 플랫폼의 설정에 따라 제공되는 SLA 가 결정됨
    - 2 개의 인프라 AZ 에 배포된 경우에만 99.999% 의 가동 시간 보장
- 플랫폼의 설정에 따라 배포된 소프트웨어에 대해 자동으로 달성되는 보안 규정 준수 수준이 결정

위와 같은걸 설정하면 팀을 나눌 수 있음
1. 조직에서 요구하는 서비스 수준을 제공하는 방식으로 Cloud-native platform 설정
    - 구성원은 특정 기술 프로필이 있음
    - 인프라 리소스를 사용하는 방법을 알고 있음
    - CLP 의 내부 작업과 플랫폼의 동작을 미세 조정할 수 있는 기본 요소를 이해
2. 소비자를 위한 소프트웨어를 만들고 운영하는 애플리케이션 팀
    - 플랫폼 API 를 사용하여 애플리케이션을 배포하고 관리
    - Cloud-native software architecture 를 이해 + 최적의 성능을 위해 모니터링하고 설정

자율적으로 운영되는 팀의 구성 및 담당하는 전체 스택 (의 일부)
1. 애플리케이션 팀
    - 배포 가능한 artifact 생성
    - 프로덕션 환경 설정, 프로덕선에 애플리케이션 배포
    - 애플리케이션 모니터링, 확장
    - 무중단으로 새로운 앱 버전 배포
2. 플랫폼 팀
    - 플랫폼 배포, 모니터링, 확장
    - 표준 런타임과 서비스 제공
    - 무중단으로 플랫폼 업그레이드