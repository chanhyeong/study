# 15. 디스틸레이션
Distillation: 증류

혼합된 요소를 분리해서 본질을 유용한 형태로 뽑아내기

## CORE DOMAIN
애플리케이션의 목적에 중심적인 모델

CORE DOMAIN 을 찾아서 support 하는 모델과 코드로부터 분리  
CORE 는 작게  
CORE DOMAIN 에는 가장 재능있는 인력 할당

### 누가 하는가
기술적인 팀원이 도메인 지식을 갖춘 경우는 거의 없다 - 도메인 지식을 보조 컴포넌트로 배정하는 경향

**오랜 기간 팀에 참여하고 도메인 지식에 관심있는 능력있는 개발자 + 업무를 깊이 알고 있는 도메인 전문가**

## GENERIC SUBDOMAIN
전문 지식이 아닌 복잡성을 더하는 모델, 부수적인 요소

**진행 중인 프로젝트를 위한 것이 아닌 응집력 있는 하위 도메인**  
-> 추출하여 별도 모듈에 배치, 해당 모듈에는 전문성이 없음

하위 도메인이 분리되고 나면 이후 개발은 CORE DOMAIN 보다 낮은 우선순위 부여  
핵심 개발자를 배치하지 않음 (개발자가 도메인 지식을 얻지 못함)

기성 솔루션이나 공표된 모델을 고려

1. 기성 솔루션: 제품 구입 or 오픈소스
2. 공표된 설계나 모델: 이미 정형화되어 있고 엄밀한 모델이 있을 때 (회계, 무리 등)
3. 외주제작
4. 사내 구현

#### 예제
핵심이 아닌 시간대와 같은 것에 중요한 사람을 배치해선 안된다

? 443p 해운에서 불리한 점에 들어간 내용은 뭐지. 계약직이 했는데 ?

#### 보조적인 성격의 문서를 활용하기
- DOMAIN VISION STATEMENT
- HIGHLIGHTED CORE

## DOMAIN VISION STATEMENT
CORE DOMAIN 을 짧게 기술하고 가치에 해당하는 "가치 제안"을 작성
- 도메인 모델이 어떻게 다양한 관심사를 충족하고 균형을 이루는지
- 한정된 범위에서 내용 유지
- 작성 후 새로운 통찰력을 얻을 때마다 개정

모델과 코드 자체의 디스틸레이션 과정에서 팀 내의 방향성

447p 예제

## HIGHLIGHTED CORE
CORE DOMAIN 을 잘 보이게끔

모든 사람이 CORE DOMAIN 을 쉽게 알 수 있게 하는 것

### 디스틸레이션 문서
CORE DOMAIN 과 CORE 의 구성요소 사이에서 일어나는 상호작용을 매우 간결한 문서로 작성  
비기술 팀원도 이해할 수 있는 문서

관리되지 않을 수도, 아무도 읽지 않을 수도, 문서가 복잡해질 수도 있음 (그래서 최소, 간결을 지향)

-> 예전부터 아이디어이긴 한데, 문서에 커밋 해시를 비교하게 해서 변경이 일어나면 문서도 같이 수정해야하는지 체크하는 기능이 있으면 좋을 것 같은데. 여기서 중요한 내용은 아니니 스킵

#### 프로세스 도구로써 활용
모델 변경의 중요성을 나타내는 실질적인 지표

### 표시된 (Flagged) CORE
모델 저장소에 있는 CORE DOMAIN 의 구성요소에 설명 말고 표시  
개발자가 힘들이지 않고도 CORE 의 안/밖을 알 수 있게

---

모델과 설계 자체를 구조적으로 변경하는 방법...

## COHESIVE MECHANISM
how 에 대한 것을 lightweight framework 로 분리  
INTENTION-REVEALING INTERFACE 로 프레임워크의 기능 노출

(생각) INTENTION-REVEALING INTERFACE 로 노출하는건 약간 [spring-data-commons](https://github.com/spring-projects/spring-data-commons) 와 실제 구현이 분리되어 있는 느낌

### GENERIC SUBDOMAIN 과의 차이?
둘 다 CORE DOMAIN 의 부담을 덜어내지만

GS 는 도메인의 관점, CM 은 도메인이 아닌 계산 등

### Declarative Style
ASSERTION + SIDE-EFFECT-FREE FUNCTION + INTENTION-REVEALING INTERFACE 일 때 가장 유용

## SEGREGATED CORE
GENERIC SUBDOMAIN 을 추출하는 방식으로 도메인에서 일부 CORE 를 불분명하게 만드는 세부사항을 제거해 CORE 를 눈에 띄게

**보조적인 역할에서 CORE 개념을 분리 -> CORE 와 다른 코드와의 결합을 줄임 -> CORE 의 응집력 강화**

1. CORE subdomain 식별
2. 새로운 모듈로 옮김
3. 리팩토링
4. 새로운 SEGREGATED CORE MODULE 의 관계와 상호작용을 단순하고 전달력있게, 다른 모듈과의 관계과 최소화되고 분명해지게
5. SEGREGATED CORE 가 완전해질 때까지 또 다른 CORE subdomain 을 대상으로 반복

#### 개인적인 정리 - GENERIC SUBDOMAIN 과 SEGREGATED CORE 차이
GS 는 CORE 에서 표현하는 것과 거리가 먼 개념 (배송 알림 등)  
SC 는 CORE 가 너무 클 때 뜯어내는 개념

## ABSTRACT CORE
수직적 (MODULE) 보단 수평적 (ABSTRACT) 으로 자르기

**모델의 가장 근본적인 개념을 식별 -> 별도의 클래스 or abstract or interface 로 추출**

디스틸레이션 문서와 비슷해보일 것

#### 개인 의견
abstract class 에 들어가는건 별로인 것 같다
- 실제 어떤 값이 있는지가 구현한 클래스에서 바로 보이지 않음. ArticleVO 같은..