# 7. 애플리케이션 생명 주기: 지속적인 변경에 대한 설명

- SDLC (소프트웨어 개발 생명 주기): 소프트웨어 설계, 개발, 단위 테스트 통과, 통합 테스트 -> 프로덕션 딜리버리
- 애플리케이션 생명 주기: 애플리케이션이 프로덕션 배포를 위해 준비된 후 진행되는 모든 단계
    - 애플리케이션 자체의 상태에 관심
        - 배포? 실행? 정지? 장애?
    - environment provisioned - start - shutdown - environment removed
    - (환경 프로비저닝됨 - 시작 - 종료 - 환경 삭제)

## 운영에 대한 공감대 형성

- 관리 용이성: 관리 기능 자동화, 업무 효율 및 안정
- 복원력: 운영자를 대신해 앱이 계속 동작하도록 하는 플랫폼
- 반응성: 사용자가 적절한 결과를 받을 수 있도록 동작
- 원가 관리: 비용 효율성

## 단일 애플리케이션, 다중 인스턴스 생명 주기

애플리케이션 설정을 변경해야하는 경우?

3가지 옵션
1. ~앱이 실행되는 동안 설정 변경~: 거의 불가능
2. blue/green
3. 롤링 업그레이드

## 서로 다른 앱 생명 주기 전반에서 조율

서로 다른 애플리케이션의 공통 설정을 동시에 변경해야하는 경우

클라이언트 - 서버의 secret key 를 변경해주어야 하는 경우

1. 서버에 일시적으로 2개의 secret key 를 모두 처리할 수 있도록 배포
2. 클라이언트에 변경된 secret key 를 배포
3. 서버에 변경된 secret key 만 처리하도록 배포

kubernetes 에서 롤링 업데이트 하기

```sh
# 기존 deployment.yaml 의 값을 변경해줌
# 이 책에선 VERSION_TRIGGER
kubectl apply -f deployment.yaml
```

## 생명 주기 짧은 런타임 환경 처리

런타임 컨텍스트의 짧은 수명이 앱의 관리성에 미치는 영향

1. 문제 해결
    - 로깅과 메트릭
2. 반복성
    - 예시: 플랫폼은 애플리케이션 배포가 100% 재현 가능해야 한다 (배포되는 특정 조각, 실행되는 방법, 컨텍스트)
        - 장점: ssh 로 런타임 환경을 허용하지 않는 것 - 복제 불가능한 앱의 실행 인스턴스를 만들 수 없게 함
        - ssh 로 조작하게 되면 다음에 배포되는 인스턴스에서는 안됨

## 앱 생명 주기 상태 가시성

한 애플리케이션이 다른 앱 생명 주기 이벤트를 알아야 할 필요성이 있을 때

## 12 factor app
https://12factor.net/