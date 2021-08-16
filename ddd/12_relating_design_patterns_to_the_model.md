# 12. 모델과 디자인 패턴의 연결

디자인 패턴이란 무엇인가?
- 특정한 상황에서 일반적인 설계 문제를 해결하고자 상호 교류하는 수정 가능한 객체와 클래스에 관해 설명한 것
- 다양한 환경에서 공통적으로 발생하는 문제를 해결했던 설계 요소 목록
- 순수하게 기술적인 용어를 사용하여 표현

도메인 패턴으로 사용할 수 있는 것 - 디자인 패턴의 일부. 강조하는 사항을 도메인 패턴에 맞게 적절하게 수정해야 함

## STRATEGY
여러 종류의 프로세스 중 하나를 선택해야 할 경우

프로세스의 중심 개념과 변경되는 부분을 분리
- 프로세스에서 변화하는 부분을 별도의 strategy 객체로 분리하여 모델에 표현
- 프로세스의 규칙, 프로세스를 제어하는 행위를 분리
- 다양한 방식으로 변형된 strategy 객체는 프로세스의 서로 다른 처리 방식 표현

#### 예제
항로 결정 시 계산 (Leg Magnitude Policy, 구간 등급 정책) -> 가장 빠른 항로, 가장 저렴한 항로

#### 기타
애플리케이션의 객체 수가 늘어날 수 있다  
=> 컨텍스트(클래스)가 공유할 수 있는, 상태 없는(stateless) 객체로 strategies 를 구현 (번역이 되게 애매하게 되어 있는 듯)  
=> 하나의 클래스 안에 static 으로 만들어라 이런건가  
=> the overhead can be reduced by implementing STRATEGIES as stateless objects that contexts can share  
=> 책에서 번역: 컨텍스트를 공유하는 상태없는 객체로 strategy 를 구현해서 부담을 줄일 수 있다

![image](https://upload.wikimedia.org/wikipedia/commons/3/39/Strategy_Pattern_in_UML.png)

## COMPOSITE
적용 기준
- 도메인 개념 간의 부분 / 전체 계층구조가 존재할 때
- 하위에 공통적인 유형의 개념으로 구성되는 추상화를 발견할 때

#### 간단 용어 설명
- Component: 집합 관계에 정의될 모든 객체에 대한 인터페이스
  - `Route`
  - Graphic
- Leaf: 가장 마지막 객체, 자식
  - `Door Leg`, `Leg`
  - Rectangle, Line, Text
- Composite
  - `Composite Route`
  - Picture
  - 자식이 있는 구성요소에 대한 행동 정의
  - 자신이 복합하는 요소들을 저장
  - Component 인터페이스에 정의된 자식 관련 연산 구현
