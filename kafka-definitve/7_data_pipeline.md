# 7. Data pipeline 구축하기
- data pipeline: 시스템 간의 데이터 이동/흐름
- pipeline 을 효율적으로 구축하면 시스템 간의 데이터 전달과 통합을 효율적으로 할 수 있다
- pipeline 에서 데이터를 생성하는 producer, 소비하는 consumer 를 분리 -> kafka 는 reliable buffer 역할

## Data pipeline 구축 시 고려사항
### Timeliness (적시성)
- real-time 처리와, 시간 단위의 batch pipeline 둘 다 지원

### Reliability
- kafka 는 `at-least-once` 를 제공
  - 외부 데이터스토어를 사용하면 `exactly-once` 도 가능

### High and varying (조절 가능한) throughput
- 데이터 파이프라인은 매우 높은 throughput 을 갖도록 확장할 수 있어야 함, 처리량이 증거하더라도 조정할 수 있어야 함
- producer/consumer 가 독ㄹ핍적이므로 어느 쪽이든 동적으로 확장하기 쉬움

### 데이터 형식
- 카프카는 데이터 형식에 구애받지 않음
- source, sink 는 데이터 구조를 나타내는 schema 를 가짐
  - source - 데이터 제공
  - sink - 데이터 받음
- 카프카로부터 외부 시스템에 데이터를 쓸 때는 sink connector 가 데이터 형식에 맞게 처리하는 책임

### 변환
#### ETL (Extract-Transform-Load)
- 순서
  - 서로 다른 source 시스템에서 데이터를 가져와서
  - 적절한 형식이나 구조로 변환
  - 다른 시스템으로 저장
- 데이터 파이프라인이 변환의 책임
  - 보존할 필요가 없어 시간과 스토리지를 절약할 수 있음
- 파이프라인에서 변환되므로, 사용자나 애플리케이션의 유연성이 떨어짐

#### ELT (Extract-Load-Transform)
- 가능한 source 데이터와 유사하게 받기 위해 파이프라인에서는 최소한의 변환만 수행
- 대상 시스템에서 raw 데이터를 변환
- 대상 시스템의 cpu 와 스토리지가 부담될 수 잇음

### 보안
- 암호화된 데이터의 네트워크 전송 허용
  - SASL 인증 (simple authentication and security layer)
- 시스템 접근을 추적하기 위한 audit log 제공

### 장애 처리
- 모든 데이터를 긴 시간 동안 저장하므로, 필요하다면 해당 시점에 맞게 이전으로 돌아가서 에러 복구

### 결합과 민첩성
- 생략

## Kafka Connect VS Producer/Consumer
- Kafka Client: 외부 애플리케이션 코드를 변경할 수 있을 때, 카프카에 데이터 read/write 를 원할 때
- Kafka Connect: 코드를 작성하지 않았고 변경도 할 수 없는 모든 외부 시스템에 카프카 연결
  - db, amazon s3, hadoop hdfs, es, ...
  - 외부 시스템의 데이터를 읽어서 카프카로 쓰거나, 카프카에서 데이터를 읽어서 외부 시스템에 쓰는데 사용되는 컴포넌트 클래스

## Kafka Connect
추후 추가