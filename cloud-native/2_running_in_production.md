# 2. 프로덕션 환경에서 클라우드 네이티브 애플리케이션 실행

대부분 프로덕션 환경 배포 프로세스는 어려움  
위험을 줄이고 효율성을 높이기 위해 느리고 번거로움

배포 후 유지도 어려움

**소프트웨어 배포 + 실행 및 유지** 가 어렵다

## 장애물들
### Snowflakes (= 눈송이)
"제 컴에서는 되는데요"
- Sofeware Development Life Cycle (SDLC) 전반에 걸친 다양성
    - 배포 후 안정성과 최초 배포 시 장애 둘 다 영향
    - 배포되는 artifact 와 배포되는 환경의 (OS 등) 불일치
- 환경별로 `application.properties` 내부 값이 다른 경우도 불일치로 봄

### 위험한 배포
배포 과정에서 downtime 이 필요하거나 예기치 않게 발생 -> 매출 손실 가능성

릴리즈가 길면 batch 크기도 더 커짐 - 다른 부분과 연관되어 예기치 않은 결과를 초래할 가능성

### 변화는 예외다 (Change is the exception)
(무슨 소리를 하는지 모르겠다)

초기 배포에 개발자를 참여시켜야 함
- 불확실성이 존재
- 구현을 깊이 이해하는 팀을 참여시키는 것이 필수
- 프로덕션 -> 운영으로 넘겨지면 동작 상태를 유지하는 방법은 runbook 으로 제공
    - 발생 가능한 장애와 해결 방안에 대한 기술
    - = 장애 시나리오가 알려져있다는 가정

### 프로덕션 설치성 (Production instability)
안정적인 프로덕션 환경 -> 새로운 배포의 전제 조건

## 조력자 (Enablers)
![image](https://drek4537l1klr.cloudfront.net/cdavis/Figures/02fig04_alt.jpg)

- Repeatability: 빠름, 안정성
- Safe deployments: 민첩성, 안정성
- 변경없는 환경에 의존하는 방식과 소프트웨어 설계를 지속적인 변경이 예상되는 환경으로 대체하면, 장애 해결에 소요되는 시간을 획기적으로 단축한다.

### Continous Delivery (CD)
amazon 은 평균 1초마다 amazon.com 프로덕션으로 코드 릴리즈

**CD 는 Continous 하지 않다**  
= 모든 코드 변경이 프로덕션에 배포되는 것은 아니다
= 새로은 소프트웨어 버전을 언제든지 배포할 수 있다

- CD 모델 (enabler)
![image](https://drek4537l1klr.cloudfront.net/cdavis/Figures/02fig05_alt.jpg)
- 이전 모델 (blocker)
![image](https://drek4537l1klr.cloudfront.net/cdavis/Figures/02fig06_alt.jpg)

선 출시는 제품의 후속 버전을 위한 사용자 피드백을 수집할 수 있음

소프트웨어를 만드는데 걸리는 시간 예측은 자주 실패함
- 구현 문제 - 네트워크 지연시간, 복잡한 비동기 통신 프로토콜 등

짧은 이터레이션으로 빈번한 릴리즈, 소프트웨어 안정성을 가지자

(이게 fe 에서 가능한건가)  
(근데 결국은 ready to ship 에서 배포가 아니라 릴리즈 단계가 별도로 있는데 의미가 있나)  
(기능이 추가되면서 테스트 길이가 길어질 수 있고, 이전 기능에 영향이 갈 수 있는데 그거에 대한건?)  
(기획에서 한 번에 빵 오픈하는걸 원할텐데 이게 어떻게 되는건지)

### Repeatability
