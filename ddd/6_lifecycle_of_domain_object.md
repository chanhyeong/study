# 6. 도메인 객체의 생명주기
![image](https://user-images.githubusercontent.com/10507662/127336163-d18b28f1-c429-41e0-b58c-2ccf8d681540.png)

1. 생명주기 동안의 무결성 유지
2. 생명주기 관리의 복잡성으로 모델이 난해해지는 것 방비

## AGGREGATE
데이터의 변경 단위로 다루는 연관 객체의 묶음, **root + boundary**

- boundary: 무엇이 포함되고 포함되지 않는지를 정의
  - boundary 내부 객체는 서로 참조할 수 있음
  - local identity 를 지님. aggregate 내부에서만 구분되면 됨
    - 외부에서는 여기를 볼 수 없음
  - boundary 외부 객체는 aggregate root 만 참조할 수 있음
- root: 하나만 존재, aggregate 에 포함된 특정 Entity

#### 예시
![image](https://user-images.githubusercontent.com/10507662/127338199-f4467650-a877-43a0-afcb-b1a0b47e2533.png)
- Customer (Aggregate 외부 객체) 는 Car, Engine 을 를 참조하거나 DB 에서 조회할 수 있음
  - Car 내의 Tire 에 대한 참조는 가질 수 없음

![image](https://user-images.githubusercontent.com/10507662/127338582-14bc85c8-ca68-4604-9700-465891d9e2fc.png)
- Car, Wheel, Tire 는 각각 Entity

### Rule
1. root entity - Global Identity + responsible for checking invariants (불변함을 검사할 책임)
2. entities in boundary - Local Identity. Aggregate 내에서만 unique
3. Aggregate 외부에서는 root entity 를 제외한 내부 구성요소를 참조할 수 없다
    - 내부 entity 참조 전달은 가능하지만 일시적으로만 사용해야 한다
    - value object 사본 전달은 상관 없음
4. db query 를 이용하면 aggregate root 만 직접적으로 획득 -> 다른 객체는 모두 aggregate 를 탐색
5. aggregate 내 객체는 다른 aggregate 의 root 만 참조할 수 있음
6. 삭제 연산은 aggregate 에 속한 모든 요소를 한 번에 제거해야 함

> Entity + Value Object 를 Aggregate 로 모으고 boundary 정의  
> Root 를 정하고 boundary 내의 객체는 Root 를 통해서만 접근  
> Root 를 통하지 않고는 Aggregate 내부를 변경할 수 없음

## FACTORY
aggregate 생성이 복잡해지거나 내부 구조를 너무 많이 드러내는 경우 -> 캡슐화

다른 객체를 생성하는 책임을 갖는 program element

**복잡한 객체와 aggregate 를 생성하는 책임을 별도의 객체로 두기**  
복잡한 객체 조립 과정을 캡슐화 + 클라이언트가 인스턴스화되는 객체의 구현(concrete) 클래스를 참조할 필요가 없는 인터페이스 제공

### 방법
factory method, abstract factory, builder (Gamma et al, et al 은 그 외라는 뜻)

잘 설계하기 위한 기본 요건

1. 생성은 atomic, 생성된 객체나 aggregate 의 invariants 를 모두 지켜야 함
2. FACTORY 는 생성된 클래스보다는 생성하고자 하는 타입으로 추상화되어야 함

factory 는 해당 factory 에서 만들어내는 객체외 매우 강하게 결합  
-> 자신의 생성물과 가장 밀접한 관계에 있는 객체

#### constructor 만으로 충분한 경우?
effective java 에서도 그렇고 constructor 대신 factory method 를 쓰라고 하지만..

- polymorphic 하지 않은 경우
- 클라이언트가 strategy 를 선택하는 경우
- ...

JAVA List 구현의 예시

### Invariant logic 의 위치
Factory 는 이미 내부 구조를 알고 있고, 구현과 밀접하게 관련이 있으므로  
Invariant logic 를 factory 에 두어서 생성물의 복잡한 요소를 줄이는 것도 이점

불변식? - entity 내의 identity 를 보장하는 식?