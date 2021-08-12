# 10. 유연한 설계

심층 모델링을 보완

overengineering 이 유연성이라는 명목으로 정당화되었지만  
너무 과도한 추상 계층과 간접 계층이 존재하면 유연성에 방해됨

![image](https://user-images.githubusercontent.com/10507662/129029574-9780d214-387c-4ed7-989c-8ae8ca52bfef.png)

## INTENTION-REVEALING INTERFACE
(의도를 드러내는 인터페이스)

**결과와 목적만을 표현**하도록 클래스와 연산 명명 (수행 방법은 언급하지 않음)  
의미를 쉽게 추측할 수 있게 UBIQUITOUS LANGUAGE 에 포함된 용어  
클래스와 연산을 추가하기 전에 행위에 대한 테스트 먼저 작성

관계와 규칙 자체, 이벤트와 액션 자체만 기술 -> 문제를 내고, 푸는 방법을 표현하면 안됨

#### 예시
paint -> mixIn

## SIDE-EFFECT-FREE FUNCTION
command (modifier), query

command 는 side effect (시스템의 상태에 대한 영향력, 시스템 상태의 변경)

- function
  - side effect 를 일으키지 않으면서 결과를 반환하는 연산
  - 여러번 호출해도 동일한 값

command 사용에 따른 문제를 완화하는 법
1. command, query 를 분리된 연산으로 유지
    - 변경을 발생시키는 메소드는 도메인 데이터를 반환하지 않음, 단순하게 유지
    - query 와 계산은 side effect 을 발생시키지 않는 메소드에서 수행
2. 기존 객체 변경 없이, command, query 분리 없이 연산의 결과를 표현하는 새로운 Value Object 생성
    - 상태 변경하는 로직과 계산이 혼합된 연산은 2개의 연산으로 분리해야 함
    - 

### 핵심
대부분의 프로그램 로직을 side effect 이 없는 함수 안에 작성  
command 를 도메인 정보를 반환하지 않는 단순한 연산으로 분리  
책임에 적합한 개념이 나타나면 로직을 Value Object 로 옮겨 side effect 를 통제 (뭘 어떻게?????)

#### 예시
Pigment Color 라는 Value Object 생성

## ASSERTION

command 에서 다른 command 호출  
상위 command 를 사용하는 개발자는 각 command 의 결과를 알아야 함 (이건 어쩔 수 없지 않나)

### 핵심
연산의 post condition 과 클래스 & aggregate 의 불변식을 명시  
명시할 수 없다면 자동화된 단위 테스트를 작성하여 assertion 표현  
문서나 다이어그램으로 assertion 명시

개발자들이 의도된 assertion 을 추측할 수 있게 인도, 쉽게 배울 수 있고 모순된 코드를 작성하는 위험을 줄이는 응집도 높은 개념이 포함된 모델을 만들어라

#### 예시
도대체 뭔소린지 갑자기 왜 분할

Paint 를 인터페이스로 변경하고

MixedPaint 와 Stock Paint 로 분할

## CONCEPTUAL CONTOUR (개념적 윤곽)
도메인의 잠재적인 일관성에 대해, 일부 영역에서 적절한 모델을 발견하면  
이 모델이 나중에 발견되는 다른 영역과도 일관성을 유지할 가능성이 높다

새로 발견된 영역이 기존 모델에 조화가 어려울 때도 있지만  
이 경우에는 더 심층적인 모델을 향해 리팩토링할 수 있다 -> 다음 발견에서는 이걸 사용하여 더 조화로울 수 있음

새로 알게 된 개념이나 요구사항을 코드에 적용하면 CONCEPTUAL CONTOUR 가 나타난다
- 이 개념이 현재 모델과 코드에 포함된 관계를 기준으로 했을 때 적절한가?
- 현재 기반을 이루는 도메인과 유사한 윤곽을 나타내는가?

반복적인 리팩토링으로 유연한 설계

### 핵심
- 설계 요소 (operation, interface, class, aggregate) 를 응집력 있는 단위로 분해
- 반복된 리팩토링에서 변경되는 부분과 변경되지 않는 부분을 나누는 것을 식별
  - 변경을 분리하기 위한 패턴을 명확하게 표현하는 CONCEPTUAL CONTOUR 를 찾을 것
- 확실한 지식을 구성하는 도메인의 일관성 있는 측면과 모델을 조화

#### 예제
대출 관리 시스템을 회계 개념으로 리팩토링

여기서의 CONCEPTUAL CONTOUR 는 무엇? - Payment 인가?  
(조기 상환 및 연체 상환 처리 규칙)

---

중간 정리

> INTENTION-REVEALING INTERFACE: 클라이언트에 의미 단위로 객체를 제공하게 해줌  
> SIDE-EFFECT-FREE FUNCTION, ASSERTION: 위를 사용한 복잡한 조합을 만드는 일을 안전하게  
> CONCEPTUAL CONTOUR: 모델의 각 부분이 안정화되고, 단위를 직관적으로 사용하고 조합할 수 있게 됨

---

## STANDALONE CLASS (독립형 클래스)
상호 의존성은 모델과 설계를 이해하기 어렵게 함

응집도가 매우 높은 하위 도메인을 MODULE 로 만들 경우, 객체를 분리하기 때문에 외부 시스템과 연관된 개념의 수를 제한할 수 있음

MODULE 내에서 의존성이 증가하면 설계 파악이 어려움.   
개발자가 다룰 수 있는 설계의 복잡도 제한.   
암시적인 개념이 너무 많아짐

### 핵심
- 결합도를 낮추어라 = 의존성을 최대한 줄여라
- 무관한 모든 개념을 제거 -> self-contained
- 같은 모듈에 속하는 클래스 간 의존성은 외부보다는 문제가 적음
- 복잡한 계산을 STANDALONE CLASS 로 도출
  - VALUE OBJECT 로 모델링하고, 밀접한 클래스에서 VALUE OBJECT 를 참조하도록 함

## CLOSURE OF FUNCTION (연산의 닫힘)
return type 과 parameter type 이 동일한 연산 정의  
주로 VALUE OBJECT 의 연산

## Declarative design
executable specification 로써, 프로그램 전체 혹은 프로그램의 일부를 작성  
property 를 정확하게 기술

### DSL
특정 도메인을 위해 특정 모델에 맞게 조정된 프로그래밍 언어

프로그램의 표현력을 향상

## A Declarative Style of Design (선언적인 형식의 설계)
`isSatisfiedBy()`, `and(), or(), not()`

### Subsumption (포섭 관계)
포함

## Angles of Attack
(받음각, 커지면 상승, 작아지면 하강 - 임계 각을 넘어가면 난류)

### Carve Off Subdomains. 하위 도메인으로 분할
조금씩 뜯어내기

모델의 일부 영역이 특화된 수학 영역으로 보이면 분리

애플리케이션에서 상태 변경에 제약을 가하는 복잡한 규칙을 적용하고 있다면 별도의 모델이나 규칙을 선언적으로 표현해주는 프레임워크 내부로 옮김

15장에서 하위 도메인을 선택하고 관리하는 방법을 알아볼 예정

### Draw on Established Formalisms, When You Can. 가능하다면 정립된 정형화를 활용
현재 혹은 다른 도메인 영역에서 정립되어 온 개념적인 체계 이용

수학 - 도메인 어딘가의 수학적인 개념을 찾아 분할