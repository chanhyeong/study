# 7. 언어의 사용 (확장 예제)
화물 해운 시스템
1. 화물 처리상황 추적
2. 화물 사전 예약
3. 일정 처리 지점 도달 시 자동으로 송장 발송

## 도메인 격리: Application
1. Tracking Query - 특정 화물의 처리 상태
2. Booking Application - Cargo 등록 및 처리 준비
3. Incident Logging Application - Cargo 처리 내력 기록

coordinator: 질문에 답하면 안됨, 답하는건 도메인 layer 에서 할 역할

## Entity, Value Object 구분
### Customer
별도 ID 를 갖는 entity

### Cargo (화물)
컨테이너를 구별할 수 있어야 하므로 entity

### Handling Event, Carrier Movement
현실세계의 사건, entity

### Location
value object???

### Delivery History
value object

Delivery History 는 서로 대체할 수 없으므로 entity 이지만  
Cargo 와 one-to-one 이므로 자체적인 식별성은 없음

Delivery History 의 식별성은 Cargo 에서 가져온 것

### Delivery Specification
value object

Delivery History 의 가상적인 상태를 표현함?  
=> Delivery History 가 Delivery Specification 을 충족하길 바람

## Aggregate 의 경계
Cargo Aggregate 는 **Cargo 가 없다면 존재하지 않는 것들을 포함**

Handling Event 는 특정 Carrier Movenet 를 적재 및 준비하기 위한 활동을 찾는데 사용  
=> Cargo 처리 (Handling Event)가 Cargo 자체와는 별개로 여길 수 있음  
=> 자체적인 Aggregate root 로 선정

## Repository 선정
### Booking Application
- Customer Repository
- Location Repository

### Activity Logging Application
- Carrier Movement Repository
- Cargo Repository

## 시나리오 연습
### 화물의 목적지 변경
- Value Object 인 기존 Delivery Specification 을 버리고 새로 대체

### 반복 업무
- 반복적으로 예약
- Prototype 패턴으로 설계 (clone))
  - Delivery History 는 새로 만듦
  - Customer Role 은 동일한 역할 복사
  - Tracking ID 는 새 것으로 제공

## 객체 생성
### Cargo 에 대한 Factory, constructor
- default constructor
- prototype factory method

```java
// 다 동일한 값 반환
public Cargo copyPrototype(String newTrackingID) // FACTORY method
public Cargo newCargo(Cargo prototype, String newTrackingID) // standalone FACTORY
public Cargo newCargo(Cargo prototype) // 아이디 생성 캡슐화

public Cargo (String id) {
  trackingID = id;
  deliveryHistory = new DeliveryHistory(this);
  customerRoles = new HashMap();
}
```

### Handling Event 추가
Incident Logging Application

```java
// default constructor
public HandlingEvent (Cargo c, String eventType, Instant timestamp) {
  handled = c;
  type = eventType;
  completionTime = timestamp;
}

// factory
public static HandlingEvent newLoading (
  Cargo c,
  CarrierMovement loadedOnto,
  Date timestamp
) {
  HandlingEvent result = new HandlingEvent(c, LOADING_EVENT, timestamp);
  result.setCarrierMovement(loadedOnto);
  return result;
}
```

## Refactoring: An alternative design of Carge Aggregate
현재: `Handling Event` 를 추가할 때 `Delivery History` 를 갱신해야 함 -> Cargo 가 트랜잭션에 참여

`Handling Event` 에 대한 Repository 추가 -> Cargo 와 관계된 Event 에 대한 질의 지원

## MODULE
![image](https://user-images.githubusercontent.com/10507662/127512361-7eebad6e-e771-4355-a326-a696557f0a18.png)

cohesive concept 을 찾고 프로젝트에 참여 중인 다른 사람들과 의사소통 하는데 집중

팀 언어에 기여하는 형태

## New Feature: Allocation Checking
Booking Application 에 들어가자

### 두 시스템의 연계
Sale Management System (영업관리 시스템) - Booking Application

Booking 에서 직접 상호 작용하지 말고 **모델과 Sales Management System 에서 쓰는 언어를 서로 번역하는 역할을 지닌 클래스를 만들자**
- 애플리케이션에서 필요한 기능만 노출
- 도메인 관점에서 관련 기능을 재추상화
- Anticorruption layer (14장)
- 여기서의 명칭: Allocation Checker
  - (이건 도메인 계층인가?)
  - Customer repository 대신 Sales Management 의 것을 사용한다면, 그와 같은 책임을 갖는 SERVICE 로 translator 가 생길 수 있음

### Enhancing the Model: Segmenting the Business (모델 강화: 업무 분야 나누기)
#### 제공하고자하는 인터페이스: 특정 유형의 Cargo 를 얼마나 예약할 수 있는가 에 대한 질문을 답변
- 근데 특정 유형이라는건 Cargo 에 지금 없음
  - 분석 패턴(Analysis Pattern) 에서 해결법을 찾을 수 있음 (아래 예시)
- **Enterprise Segment**: 업무를 분할하는 방법을 정의한 dimension 의 집합
  - month, date, ...
  - value object
- Cargo Repository 가 Enterprise Segment 를 기반으로 query 제공

##### 문제
1. Booking Application 에 규칙을 적용하는 책임 부여 -> 업무 규칙은 도메인의 책임, 응용에서 수행하면 안됨
2. Booking Application 에서 어떻게 Enterprise Segment 를 도출하는지 분명하지 않음

이런 책임은 모두 Allocation Checker 에 속하게

![image](https://user-images.githubusercontent.com/10507662/127515166-ba80f9af-2078-4e77-a06c-1a8690256730.png)
