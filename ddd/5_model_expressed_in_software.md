# 5. 소프트웨어에서 표현되는 모델
연속성 (continuity), 식별성 (identity), 상태를 기술하는 속성 -> ENTITY, VALUE OBJECT 를 구분하는 가장 기본적인 방법  
action, operation -> SERVICE  
high cohesion, low coupling -> MODULE

## Association
연관관계를 더 쉽게 다루기 (many to many 를 줄이자)

1. 탐색 방향을 부여
    - 국가 -> 대통령
2. qualifier 를 추가 -> multiplicity 를 줄임
    - 기간 조건을 추가
3. 중요하지 않은 연관관계 제거

## ENTITY
- 식별성 - 객체와 해당 객체의 저장 형태, 구현 일치
- ENTITY 의 근본적인 개념: 객체의 lifecyle 간 이어지는 추상적인 연속성, 여러 형태를 거쳐 전달

#### 정의
**모델링과 설계들이 담긴, 식별성이 정의된 객체**
- entity bean (단순 db 매핑) 이 아님
- 생명주기 동안 내용이 변할 수 있으나 연속성은 유지해야 함
- 클래스 정의와 책임, 속성, 연관관계는 ENTITY 의 정체성에 초점 (특정 속성에 맞추지 않고)

클래스의 정의를 단순하게 + lifecycle 의 연속성과 식별성에 집중  
- identity 는 필요에 의해 보충된 의미

### ENTITY MODELING
가장 기본적인 책임: 객체의 행위가 명확하고, 예측 가능해질 수 있게 연속성을 확립  
개념에 필수적인 행위만 추가, 행위에 필요한 속성만 추가

그 외는 ENTITY 와 연관관계가 있는 다른 객체로 옮김 -> 일부는 또 다른 ENTITY, 어떤 것은 VALUE OBJECT

#### identity operation 설계

## VALUE OBJECT
식별성을 갖지 않으면서 도메인의 서술적 측면을 나타내는 객체  
설계 요소가 어느 것인지가 (who or which) 아닌 **무엇인지에 관심** (what)

모델에 포함된 속성에만 관심 -> VALUE OBJECT  
immutable, no identity

#### VALUE OBJECT 설계
- copy or share
  - webapp 에서 이 개념이 필요한가
  - share 가 가능하긴 한가
- immutable

#### VALUE OBJECT, denormalization
value object 를 따로 테이블 분리하지 않고 entity 테이블에 같이 포함 (접근 시간 우선, 동일 데이터가 많은 것은 어쩔 수 없다)

## SERVICE
(spring 의 `@Service` 도 여기서 가져온 것)
도메인 개념 중 객체로는 모델에 어울리지 않는 것
- 모델에서 독립적인 인터페이스로 제공되는 연산
- 상태를 캡슐화하지 않음
- 다른 객체와의 관계를 강조
- 활동으로 이름을 지음, 연산의 명칭은 UBIQUITOUS LANGUAGE 에서 도입

1. ENTITY, VALUE OBJECT 의 일부를 구성하는 것이 아닌, 도메인과 관련
2. 인터페이스가 도메인 모델의 외적 요소로 정의
3. 연산이 상태를 갖지 않음

### layer
- SERVICE 는 도메인, 응용, 인프라 layer 계층에 각각 속할 수 있다

#### application SERVICE
- 입력 내용 암호화
- 도메인 서비스로의 메시지 전송
- infra SERVICE 를 이용한 이후 작업
- controller 는 여기가 아닌가? 왜 infra 에 들어가있지

#### domain SERVICE
- 객체 간 상호작용
- 상호작용 결과 제공
- commands, queries 는 여기가 아닌가? 왜 application 에 들어가있지
- repository 도 interface 라지만 결국은 infra 아닌가?
- ddd start 에서는 직접 보지 말라는게 있었는데 여기는 아직 내용이 안나온건지 그 내용은 없음

#### infra SERVICE
- 정해진 작업 (이메일 발송 등)

### Granularity (초월 번역 -> 구성 단위)
입상: 낟알이나 알갱이 모양

- medium-grained stateless SERVICE - 재사용이 쉬움
- fine-grained Objects - 분산 시스템에서 비효율적인 메시지 전송 초래할 수 있음
- fine-grained 도메인 객체는 도메인에서 지식이 새어나오게 하여 -> 도메인 객체의 행위를 조정하는 application layer 로 흘러가게 할 수 있음
  - 상호작용의 복잡성이 application layer 에서 처리되고
  - 도메인 계층에서 사라진 도메인 지식이 application or user interface layer 코드로 스며든다

## MODULE (a.k.a Package)
- MODULE 간에는 loose coupling, high cohesion
- 리팩토링이 어렵긴 함 (전체적인 구조 변경 때문에)
  - 도메인을 이해하는 바가 바뀔 때마다 모듈에 반영하면, 모듈 내의 객체도 더 발전
- 클래스들을 한 모듈 안에 둔다면 -> 그 **클래스들을 하나로 묶어서 생각하자** 라는 의미와 같음
- **하나의 개념적 객체를 구현하는 코드는 모두 같은 MODULE 에 두어야 한다**