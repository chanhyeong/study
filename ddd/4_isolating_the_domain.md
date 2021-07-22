# 4. 도메인의 격리
navigation map  
![image](https://user-images.githubusercontent.com/10507662/126060186-65da084d-6b55-43b6-90e9-7e6f3c838447.png)

layered architecture  
![image](https://user-images.githubusercontent.com/10507662/126060217-1b818bf1-3bdd-432b-a6e1-eb2cb29856c3.png)

#### 도메인에 관련된 코드가 도메인과 관련이 없는 다른 코드들에 퍼져있는 경우  
- 도메인에 관련된 코드를 확인하고 추론하기가 힘들어진다  
- UI 를 변경하는 것이 업무 로직을 변경하는 것으로 이어질 수 있다

#### LAYERING 의 가치
- 각 계층에서 프로그램의 특정 측면만을 다룬다
- 토대로 응집력 있는 설계 -> 설계를 쉽게 이해

| name | description |
| - | - |
| User Interface | 사용자에 view + 사용자의 command<br/>외부 컴퓨터가 될 수도 있음 |
| Application | 소프트웨어가 수행할 작업 정의 + 도메인 객체가 문제를 해결<br/> 다른 시스템의 application layer 와 상호작용<br> 얇게 유지 (업뮤 규칙 및 지식이 없음, 작업을 조정하고 도메인 객체의 협력자에 작업 위임)<br/> 업무 상황의 상태는 없지만 프로그램 작업의 진행 상태는 가질 수 있음 |
| Domain | 업무 개념 + 상황 + 규칙을 표현<br/> 업무 상태 제어 및 사용, 상태 저장과 기술적인 세부 사항은 infra 에 위임<br/> 업무용 소프트웨어의 핵심 |
| Infrastructure | 기술적인 기능<br/> 메시지 전송, persistence, ui rendering |

#### 여러 layer 로 나누자
- 응집력 있고, 오직 하위에 layer 에만 의존하는 각 layer 에서 설계를 발전
- 상위 layer 와는 loose coupling
- 도메인 모델과 관련된 코드는 한 layer 에 모아라

### layer 간 관계 설정
- 상위 layer 는 하위 layer 의 public interface 를 호출 및 구성요소를 직접 사용하거나 조작 가능
- 하위 객체가 상위 객체와 소통해야 할 경우?
  - callback 이나 Observer 패턴 활용
- application / domain 에서는 infra 가 제공하는 SERVICE 를 요청
  - 일부 요소는 다른 layer 의 기본 기능을 직접 지원하도록 만들어짐 (모든 도메인 객체에 대한 abstract class 등)

### Architecture Framework
- infra 가 SERVICE 형태로만 제공된다면 loose coupling + 직관적
- 일반적으론 침습적인 (intrusive) 형태가 필요
  - framework class 의 하위 클래스가 되어야 한다
  - 일정한 method signature 를 가져야 한다 등
- framework 를 적용할 때 팀은 framework 의 목적에 집중해야 함
  - 도메인 모델을 표현하고 구현하는 것
- framework 에서 제공하는 모든 기능을 사용하는 것은 아님
- 하나의 framework 로 전부 해결하는 게 아니라, 여러 개를 선택적으로 적용
  - 구현 - framework 간의 결합이 줄어들어 유연하게

## Domain layer 는 Model 이 사는 곳
#### Domain model 은 개념을 모아놓은 것
- domain layer 는 업무 로직에 대한 설계와 구현으로 구성
- DDD 의 전제 조건은 도메인 구현을 격리
