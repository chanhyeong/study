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