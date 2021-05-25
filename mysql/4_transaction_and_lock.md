# 4. Transaction and Lock
**Transaction**: 데이터의 정합성, 작업의 완전성을 보장
-> 완벽하게 처리 or 못하는 경우에는 원 상태로 복구 - partial update 방지

**Lock**: concurrency 를 제어하기 위한 기능

Isolation Level: 하나 혹은 그 이상의 transaction 간의 작업 내용을 어떻게 공유하고 차단할 것인지

## Transaction
### MySQL 에서의 transaction
```sql
# MyISAM (최종 결과: 3), partial update 발생
INSERT INTO tb_myisam (id) VALUES (3)
INSERT INTO tb_myisam (id) VALUES (1), (2), (3)

# InnoDB (최종 결과: 1, 2, 3)
INSERT INTO tb_innodb (id) VALUES (3)
INSERT INTO tb_myisam (id) VALUES (1), (2), (3)
```

### 주의사항
- transaction 과정 (begin - commit) 을 짧게 가져가는 것이 중요
- DB 테이블을 다룰 때에만 transaction 을 잡고 다른 외부 호출 요소들은 그 외에서 처리하는 것이 좋음
  - transaction 과정에서는 connection pool 을 잡고 있게 됨
  - 외부 포인트 장애 시 connection 을 반환 시간 지연

## MySQL Engine Lock
Storage engine level, MySQL engine level

### Global Lock
`FLUSH TABLES WITH READ LOCK` 으로 획득
DDL, DML 을 실행하는 경우, **MySQL server 전체에 영향**

### Table Lock
table 단위
- 명시적으로는 `LOCK TABLES table_name [ READ | WRITE ]`, `UNLOCK TABLES`
- 묵시적으로는 테이블 데이터 변경 쿼리 등 (InnoDB 는 해당 없음)

### User Lock
```sql
SELECT GET_LOCK('test', 2) # 2초 간

SELECT IS_FREE_LOCK('test')

SELECT RELEASE_LOCK('test')
```

### Name Lock
table, view 등의 데이터베이스 객체의 이름을 변경할 때 자동으로 획득

## InnoDB 에서의 Lock
MySQL 에서 제공하는 것과 별개로 record (row) 기반 Lock 탑재

### Lock 방식
Pessimistic locking (비관적 잠금)
- Pessimistic locking
  - 현재 transaction 에서 **변경하고자 하는 레코드에 대한 lock 을 획득하고 변경** 작업 처리
  - 변경하고자하는 레코드를 다른 transaction 에서도 변경할 수 있다는 비관적인 가정
- Optimistic locking (낙관적 잠금)
  - **우선 변경을 진행**하고 마지막에 lock 충돌이 있었는지 확인 -> 있다면 ROLLBACK
  - 각 transaction 이 같은 레코드를 변경할 가능성이 희박할 것이라고 생각

### Lock 종류
#### Record Lock (Record only lock)
- record 자체만을 locking
  - record 자체가 아닌, index 의 record locking
  - index 가 하나도 없는 테이블이어도, 내부적으로 자동 생성된 클러스터의 인덱스를 이용해 lock
- InnoDB 에서 대부분의 인덱스를 이요한 변경 작업은 Next key lock or Gap lock 을 사용하지만 PK, Unique index 는 record lock

#### Gap lock
- record 자체가 아닌 record 와 바로 인접한 record 사이의 간격만 locking
- record - record 사이에 새로운 record 가 INSERT 되는 것을 제어
- 거의 사용이 없고 Next key lock 의 일부로 사용

#### Next key lock
- record lock + gap lock 을 합쳐놓은 형태
- `STATEMENT` format 의 binlog 를 사용 + REPETABLE READ level
  - 이 때 next key lock 방식으로 lock 이 걸림
  - 쿼리가 slave 에서 실행될 때 master 에서 만들어 낸 결과와 **동일한 결과를 만들어내도록 보장하는 것이 주 목적**
  - next key lock 으로 인해 deadlock 발생할 수 있음
- `ROW` format (5.7 부터 디폴트) 으로 바꾸어 lock 을 줄일 수 있음
- 어제 메신저에서도 나왔는데.. 처음 봄

#### Auto increment lock
`AUTO_INCREMENT`
- table 단위, 매우 짧음, 자동, `INSERT, REPLACE`
- 하나가 수행되면 lock 이 걸리고 동시에 다른 쿼리 수행 시 대기해야 함
- 방식 조정 `innodb_autoinc_lock_mode`
  - 0: 위와 동일
  - 1: `INSERT` 에서 개수를 예측할 수 있을 때에는 가벼운 latch (mutex) 를 이용하여 처리
    - `INSERT ... SELECT` 처럼 대량 INSERT 가 수행될 때는
      - 1회에 여러 개의 increment 를 할당받아두고 레코드에 사용
  - 2: lock 을 걸지 않고 항상 latch (mutex) 사용
    - 하나의 `INSERT` 에서 연속적인 increment 값을 보장하지 않음

### Index 와 Lock
MySQL 은 record lock 에서 실제 record locking 이 아닌 index locking

```sql
KEY idx_firstname (first_name)

# first_name 이 동일한 사람이 253 건이고, 아래 last_name 까지 동일한건 1건이라고 해도
# 254 건의 index 에 대해 전부 lock 이 걸림
UPDATE employees SET hired_at = NOW()
WHERE first_name = 'Chanhyeong' AND last_name = 'Cho'
```
- 위 문제로 인해 index 설계를 잘 해야 함
- next key lock 으로 인해 발생하는 문제

## MySQL 의 Isolation Level
Isolation level: 동시에 여러 transaction 이 처리될 때, 특정 transaction 이 다른 transaction 에서 변경하거나 조회하는 데이터를 볼 수 있도록 허용할지 말지를 결정

||DIRTY READ|NON-REPEATABLE READ|PHANTOM READ|
|-|-|-|-|
|READ UNCOMMITTED|O|O|O|
|READ COMMITTED|X|O|O|
|REPETABLE READ|X|X|O (InnoDB 제외)|
|SERIALIZABLE|X|X|X|

### READ UNCOMMITTED
보통의 Database 는 이보다 높은 isolation level 로 제공
#### Dirty read
- 변경이 COMMIT, ROLLBACK 에 관계 없이 다른 transaction 에서 보여짐  

![image](https://vladmihalcea.com/wp-content/uploads/2018/05/DirtyRead-788x431.png)

### READ COMMITTED
- UNDO 영역이라는 것을 통해, transaction 에서 commit 완료된 데이터만 조회
- 이미지: https://nesoy.github.io/articles/2019-05/Database-Transaction-isolation#read-committed

#### Non-repeatable read
- transaction commit 이 되기 전 조회한 데이터와, commit 이후에 조회한 데이터가 다를 수 있음

### REPETABLE READ
- UNDO 영역 + MVCC (Multi Version Concurrency Control)
  - InnoDB 의 transaction 은 고유한 transaction no 를 가짐
  - Undo 영역에 들어간 레코드는 변경을 발생시킨 transaction no 가 포함되어 있음

#### Phantom read
- 다른 transaction 에서 수행한 변경 작업에 의해 레코드가 보였다가 안 보였다가 하는 현상

### SERIALIZABLE
