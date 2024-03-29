# 5. 앱 다중화: scale out & statelessness (상태 비저장)

## 많은 인스턴스를 배포하는 클라우드 네이티브 앱
고가용성, 안정성, 운영 효율성

## HTTP 세션과 sticky session
#### sticky session
사용자의 첫 번째 요청에 대한 응답에 session id 를 포함하는 구현 패턴, 사용자의 고유한 정보 -> 쿠키를 통해 모든 후속 요청에 포함  
로드밸런서가 어떤 인스턴스에 제일 처음 접속했는지 기억하여 계속 라우팅 (하도록 최선을 다함)

#### 여기서 문제
LB 는 적절한 인스턴스에 전송하지 못할 수 있음  
인스턴스가 사라지거나, 네트워크 이상

## 상태 저장 서비스와, 상태 비저장 앱

**특별히 설계된 상태 저장 서비스에 상태 정보를 두고, 앱에서는 제거한다**

상태 비저장 앱의 장점?
- 분산된 데이터를 복원하는 복잡성을 낮춤 (재배포가 쉬움)
- 모든 앱의 여러 버전을 합리적으로 관리

개발자가 할 일:  
보존해야 할 상태와 그렇지 않은 것을 명시해 앱을 설계 + 데이터를 상태 저장 서비스에 위치

### 앱을 상태 비저장으로 만들기
데이터를 유지하기 위한 key/value store -> 앱으르 해당 서비스에 바인딩 -> 모든 앱 인스턴스는 유효한 데이터에 접근
