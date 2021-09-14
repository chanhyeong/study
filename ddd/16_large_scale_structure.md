# 16. 대규모 구조
책임을 자세히 알지 못해도 전체적인 관점에서 위치를 어느 정도 이해하는 데 도움을 주는 규칙, 규칙과 관계의 패턴 고안

## EVOLVING ORDER (발전하는 질서)
**개념적인 대규모 구조**를 애플리케이션 발전 과정에서 다른 형식의 구조로도 변화할 수 있게 만들어라  
세부 설계 및 모델을 과도하게 제약해서는 안된다 (개별 부분에 전역적인 규칙을 적절하게 타협하여 적용)

#### 생각
구조 및 규칙들을 코드 저장소에 문서화 (CONVENTION.md)

## SYSTEM METHPHOR (시스템 은유)
추상적이고 파악하기 힘든 소프트웨어를 이해하고 시스템을 전체적으로 바라보는 시각을 공유할 수단

시스템의 구체적인 비유가 나타나면 그것을 대규모 구조로 채택, 중심으로 설계를 구성, UL 로 흡수

## RESPONSIBILITY LAYER (책임 계층)
RELAXED LAYERED SYSTEM: 한 계층의 구성요소가 바로 아래 + 모든 하위 계층에 접근 허용

개념적 의존성과 도메인의 변화율, 변화의 근본을 검토  
도메인에서 자연적인 층을 추상적인 책임으로 간주  
AGGREGATE, MODULE 등의 도메인 객체의 책임이 한 계층의 책임 안에 있도록

#### 생각
예제를 보니 도메인에 대한 계층 그림이 잘 관리가 되어야 기능 추가할 때도 생각하기가 편할 것 같다

### 계층 관련 내용이 변경될 때마다
1. 스토리텔링: 도메인의 상황과 우선순위를 전해주어야 함 (기술적 결정이 아닌 업무 관련 모델링 결정임)
2. 개념적 의존성: 상위는 하위를 배경으로 하는 의미를 지녀야 하고, 하위 개념은 독자적인 의미를 지녀야 함
3. CONCEPTUAL CONTOUR (개념적 윤곽, 10장)

## KNOWLEDGE LEVEL (지식 수준)
![image](https://user-images.githubusercontent.com/10507662/133217051-3ba5d8a8-7d77-4867-815a-cf3085902d3e.png)

다른 객체 집단이 어떻게 행동해야 하는지를 기술하는 객체 그룹

REFLECTION 패턴의 도메인 계층을 응용한 것 - 소프트웨어를 self-aware 하게 만듦  
base level (운영적 책임), meta level (구조와 행위에 대한 지식)

언어의 reflection 은 도메인을 위한 것이 아님 - 해당 언어 구성물 자체의 구조와 행위  
KNOWLEDGE LEVEL 은 반드시 일반 객체로 만들어야 함

애플리케이션의 도메인에 초점 + 완전한 보편성을 달성하진 않음

**모델의 구조와 행위를 서술하고 제약하는 데 쓸 수 있는 별도의 객체 집합을 만드는데**  
**하나는 매우 구체적으로, 하나는 사용자/관리자의 맞춤화가 가능한 규칙과 지식을 반영하게**

#### 생각
그냥 도메인하고 무슨 차이지????

RL 과 차이? - KL 에서는 의존성이 양방향으로 작용

## PLUGGABLE COMPONENT FRAMEWORK (착탈식 컴포넌트 프레임워크)
interface 와 interaction 에 대한 abstract core 를 만들어 구현이 자유롭게 대체될 수 있는 프레임워크를 만들어라

## 구조는 얼마나 제약성을 가져아 하는가
구조적 규칙은 개발을 용이하게 해야 한다

## 알맞는 구조를 향한 리팩토링
### Minimalism
초반엔 SYSTEM METHPHOR, RESPONSIBILITY LAYER 정도

### 의사소통과 자기 훈련
팀 전체가 구조를 이해하고 따라야 함. 용어와 관계는 반드시 UL 에

대화를 UL 에 통합하여 끊임없이 연습하게 만들어야 함

### Restructuring 이 유연한 설계를 도출
구조가 바뀔 때마다 전체 시스템을 새로운 질서에 맞게 바꾸기

### Distillation 은 부하를 줄인다
어렵다..