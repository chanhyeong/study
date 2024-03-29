# 13. 더 심층적인 통찰력을 향한 리팩토링

1. Live in the domain. (활동의 근거지를 도메인으로 삼는다)
2. Keep looking at things a different way. (현상과 사물을 다른 방식으로 바라보도록 노력한다)
3. Maintain an unbroken dialog with domain experts. (도메인 전문가와 지속적으로 대화한다)

고전적인 리팩토링 - 1 ~ 2명의 개발자가 개선의 여지가 있는 코드를 발견하고 즉석에서 변경

## 시작
문제 발견
- 복잡성이나 부자연스러움, 도메인 모델 개념 누락, 관계가 이상, ...

## Exploration Team (조사팀)
- 모델을 명확하고 자연스럽게 의사소통할 수 있게 만들어줄 개선안 조사
- 코드 변경에 참여하는 대상 4~5명 선정
  - 개발자 2명 선정
    1. 해당 유형의 문제에 대한 사고 능력이 좋음
    2. 해당 도메인 영역을 잘 알고 있음
    3. 모델링 기술이 뛰어난
  - 난해하다면 도메인 전문가도 함께 참여
- 30분 ~ 1시간 30분 동안 브레인스토밍
  - UML or 객체 시나리오 재현
  - 발견한 내용을 코드로 옮김
- 생산성 있게 진행하기
  - 자기 결정: 설계 문제를 조사하기 위해 며칠 동안 작업하고 해산하는 규모가 작은 팀 구성
  - 범위와 휴식: 며칠, 짧은 회의 후 설계안 도출. 막히면 범위가 너무 큰지 확인
  - UBIQUITOUS LANGUAGE 사용

## 선행 기술
서적, 도메인 자체 지식을 정리한 자료, 분석 패턴, ...

## 개발자를 위한 설계
작성한 코드를 시스템의 다른 부분과 통합하는..

반복적으로 코드 리팩토링 -> 유연한 설계 = 설계 의도 전달 + 코드 영향을 쉽게 예측 -> 리팩토링이 쉬워짐

## 타이밍
**지속적인 리팩토링**
- 이걸 망설이게 하는건..
  - 코드 변경에 따르는 위험과 변경에 소요되는 개발자 시간 비용을 인식
  - 부자연스러운 설계와 그러한 설계를 사용해서 작업하는 데 따르는 비용은 알지 못함
- 리팩토링을 원하는 개발자: 당위성을 설명해야 함
  - 당위성이 받아들여져도 이미 어려웠던 작업이 불가능할 정도로 어려워짐. 리팩토링을 할 수 없게 됨
  - (공식적인 승인 없이 암암리에..)

#### 도메인을 지속적으로 조사하고, 개발자를 교육하며, 개발자와 도메인 전문가가 의견의 일치를 이루는 과정의 일부가 되어야 함
- 현재 팀에서 도메인을 이해하고 있는 바가 설계에 없는 경우
- 중요한 개념이 설계 상에 암시적인 경우
- 설계 상의 중요한 부분을 유연하게 만들 수 있는 경우

#### 안되는 것
- 출시 전날에 리팩토링
- 기술적임을 뽐내기 위한 유연한 설계
- 도메인 전문가가 납득하지 못하는 deeper model

## 위기를 기회로
점진적인 변화를 거쳐 짧은 기간의 폭발적이면서 신속한 변화. 전에 계속 하던 얘기

모델을 일정 기간 꾸준히 정제 -> 통찰력

기회가 아닌 위기로 보임 -> 모델 내에 결점 발견 = 팀이 새로운 이해 수준에 도달