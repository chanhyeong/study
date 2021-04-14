# 실행 계획
### 쿼리 실행 절차
1. SQL 파싱 -> MySQL 서버가 이해할 수 있는 수준으로 분리
2. SQL 파싱 정보를 확인하면서 어떤 테이블부터 읽고 어떤 인덱스를 이용해 읽을지 선택
3. 2에서 결정된 테이블 read 순서나 인덱스를 이용하여 storage engine 으로부터 데이터 가져옴


#### 1 - SQL Parsing
- MySQL server `SQL Parser` 모듈로 처리
- Syntax check 수행
- SQL Parse tree 가 생성됨 (쿼리 실행의 기준)

#### 2 - (Parse tree 이용) 최적화 및 실행 계획 수립
- MySQL server `Optimizer` 에서 처리
- 불필요 condition 제거 및 복잡한 연산 단순화
- join 이 있는 경우 어떤 순서로 읽을지 결정
- 각 테이블에 사용된 contion 과 index 통계 정보를 이용해 사용할 인덱스 결정
- 가져온 record 들을 임시 테이블에 넣고 다시 한 번 가공해야 하는지 결정

#### 3 - 가져오기
- MySQL server + Storage engine
- server: 넘어온 레코드를 조인하거나 정렬하는 작업

### Cost-based optimizer (CBO)
- 현재 대부분의 DBMS 가 선택하고 있는 방법
- 쿼리를 처리하기 위한 여러 방법을 만들고, 부하와 테이블 통계 정보를 이용하여 비용 산출 -> 최소 값으로 실행

### 통계 정보
- 대략적인 레코드 건수 + 인덱스의 유니크한 값 개수
- 순간 + 자동적으로 변경, 동적임

## 실행 계획 분석
`EXPLAIN` 으로 조회, `EXPLAIN` + SELECT QUERY

- 라인수: 쿼리문에서 사용한 테이블 개수
- 실행 순서: 위 -> 아래 순서로 표시
- 위쪽에 출력된 결과일 수록 쿼리의 Outer or 먼저 접근한 테이블
- 필요에 따라 쿼리의 일부분을 직접 실행할 때도 있음

|id|select_type|table|type|Key|key_len|ref|rows|Extra|
| - | - | - | - | - | - | - | - | - |

### id
- **단위 SELECT 쿼리별로 부여되는 식별자 값**

#### SELECT JOIN
- 아이디가 증가하지 않고 같은 id 로 부여
```sql
EXPLAIN
SELECT e.emp_no, e.first_name, s.from_date, s.salary
FROM employeees e, salaries s
WHERE e.emp_no=s.emp_no
LIMIT 10
```
|id|select_type|table|type|Key|key_len|ref|rows|Extra|
| - | - | - | - | - | - | - | - | - |
|1|SIMPLE|e|index|ix_firstname|44| |300584|Using index|
|1|SIMPLE|s|ref|PRIMARY|4| |employees.<br/>e.emp_no| |

#### SUBQUERY
```sql
EXPLAIN
SELECT 
( (SELECT COUNT(*) FROM employees) + (SELECT COUNT(*) FROM departments) ) AS total_count
```
|id|select_type|table|type|Key|key_len|ref|rows|Extra|
| - | - | - | - | - | - | - | - | - |
|1|PRIMARY| | | | | | |No tables used|
|3|SUBQUERY|departments|index|ux_deptname|123| |9|Using index|
|2|SUBQUERY|employees|index|ix_hiredate|3| |300584|Using index|

### select_type
#### SIMPLE
- UNION, SUBQUERY 를 사용하지 않는 단순 SELECT 쿼리
- 실행 계획에서 SIMPLE 인 쿼리는 1개만 존재, 일반적으로 제일 바깥 SELECT 쿼리

#### PRIMARY
- UNION, SUBQUERY 가 포함된 SELECT 쿼리에서 제일 바깥쪽에 있는 단위 쿼리

#### UNION
- UNION 으로 결합하는 단위 SELECT 쿼리 중 첫 번쨰를 제외한 두 번째 이후 단위 SELECT 쿼리

```sql
EXPLAIN
SELECT * FROM (
  (SELECT emp_no FROM employees e1 LIMIT 10)
  UNION ALL
  (SELECT emp_no FROM employees e2 LIMIT 10)
  UNION ALL
  (SELECT emp_no FROM employees e1 LIMIT 10)
) tb;
```
|id|select_type|table|type|Key|key_len|ref|rows|Extra|
| - | - | - | - | - | - | - | - | - |
|1|PRIMARY|<derived2>|ALL| | | |30| |
|2|DERIVED|e1|index|ix_hiredate|3| |300584|Using index|
|3|DERIVED|e2|index|ix_hiredate|3| |300584|Using index|
|4|DERIVED|e3|index|ix_hiredate|3| |300584|Using index|
| |UNION RESULT|<union2,3,4>|ALL| | | | |

#### DEPENDENT UNION (외부 영향을 받음), UNION RESULT (UNION 임시 테이블)
#### SUBQUERY, DEPENDENT SUBQUERY
- FROM 절 이외에서 사용되는 서브 쿼리만을 의미
#### DERIVED
- FROM 절에 서브 쿼리가 사용된 경우
- 메모리나 디스크에 임시 테이블 생성
  - 인덱스가 없으므로 다른 테이블과 조인할 때 성능 불리

**비고 (서브 쿼리)**
```
Nested Query
-> SELECT column 에 사용된 서브쿼리

Sub Query
-> WHERE 절에 사용된 경우

Derived
-> FROM
```

#### UNCACHEABLE SUBQUERY, UNCACHEABLE UNION
- 서브 쿼리의 실행 결과를 캐시에 담아두는데, 서브 쿼리에 포함된 요소에 의해 캐시 자체가 불가능한 경우
  - user variable 이 사용된 경우
  - NOT-DETERMINISTIC 속성의 stored routine 이 사용된 경우
  - UUID(), RAND() 등의 함수가 포함된 경우

### table
- 테이블 명이 없는 경우 `(NULL)` 로 표시됨
- `<>` 로 명시된 케이스 - 임시 테이블
