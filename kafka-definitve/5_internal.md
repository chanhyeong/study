# 5. Kafka Internals
- replication 이 동작하는 방법
- Kafka producer 와 Consumer request 를 처리하는 방법
- Storage 를 처리하는 방법

## Cluster Membership
### Zookeeper
- Zookeeper 는 보통 파일 시스템처럼 hierarchical tree 구조
  - znode: 저장하는 node, 이름 앞에는 `/`.
    - 디렉토리처럼 path 사용하여 node 위치 식별
- 각 node 에는 상태, 구성 정보, 위치 정보 등의 작은 데이터, 메모리에 저장 (1KB 미만)
- ephemeral node: client 가 연결되어 있을 때만 존재, 끊어지면 제거
- persistent node: client 가 삭제하지 않는 한 계속 보존
- `Watch`: node 의 상태 모니터링
  - client 가 특정 node watch 설정 시, 변경이 일어난 경우 callback 으로 client 에게 알려줌

### kafka 와 zookeeper
- zookeeper 를 이용하여 broker 들의 metadata 유지 관리
- 예시
  - /kafka-main
    - /kafka-main/controller
    - /kafka-main/brokers
      - /kafka-main/brokers/ids
        - broker 가 뜰 때 자동 생성하여 ephemeral 저장하며, 같은 아이디로 새로 추가하려고 하면 에러
      - /kafka-main/brokers/topics
    - /kafka-main/config

#### broker 중단 시
1. 해당 broker 의 zookeeper node 는 삭제됨
    - 해당 broker id 는 다른 데이터들에 존재 (topic replica 내역 등)
2. 완전히 새로운 broker id 를 갖는 broker 시작
3. 누락된 broker 에 할당되었던 partition 과 topic 을 갖고 클러스터에 합류

## Controller
- **partition leader 를 선출하는 책임을 갖는 broker**
- cluster 에서 시작하는 첫 번째 broker
  - 여기서 `/controller` 를 생성
  - 다른 broker 에서도 시도하나, 이미 존재하므로 실패. 다른 broker 들은 controller 를 Watch 하는 상태임

### controller 가 내려가면?
#### 선정
- cluster 내에서 `/controller` 를 선점하는 broker 가 새로 차지
- 새로 선정 시 새로운 **controller epoch** 를 받음 (번역이 세대 번호로 되어 있음..)
- 나머지 broker 들도 현재의 controller epoch 를 알게 되며, 이전 세대의 controller epoch 를 받으면 무시

#### 재할당
- 기존 controller 가 leader 로 갖고 있던 partition 들에 새로운 leader 가 필요함
- 새로운 controller 가 해당 partition 들을 점검하고, 새로 리더가 될 broker 결정
- 새로운 leader/follower 정보를 모든 broker 들에 전송
  - leader partition 들은 producer/consumer 요청 처리를 시작
  - follower 는 새로운 leader 의 메시지 복제

### 새 broker 가 추가되면?
- broker id 를 사용하여 그 broker 의 replica 로 사용할 broker 가 있는지 확인
- 있으면 새 broker 와 기존 broker 모두에게 변경 사항을 알리고, 새 broker 의 replica 들은 기존 leader 의 메시지 복제 시작

## Replication
- topic 이 있고 이게 여러 partition 에 저장될 수 있음
- 각 partition 은 다수의 replica 를 가질 수 있음
