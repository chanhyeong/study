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

### type
- 테이블의 레코드를 어떤 방식으로 읽었는지를 의미
- system, const, eq_ref, ref, fulltext, ref_or_null, unique_subquery, index_subquery, range, index_merge, index, ALL
  - 빠른 순서대로 나열
  - index_merge 외의 타입은 하나의 인덱스만 표시됨
  - ALL 을 제외한 나머지는 모두 인덱스를 사용

#### system
- 레코드가 1건 이하로 존재하는 테이블을 참조할 때
- InnoDB 에서는 없고 MyISAM 이나 MEMORY

#### const
- 쿼리가 Primary key 나 Unique key 를 이용하는 WHERE 조건절을 가지고 + 1건을 반환하는 처리 방식
- UNIQUE INDEX SCAN
- pk 의 일부만 조건으로 사용할 때는 ref 로 표시됨
  - ex) composite primary key 에서 1개만 WHERE 에 입력

#### eq_ref
- 여러 테이블이 조인되는 쿼리에서 표시
- 조인에서 처음 읽은 테이블의 column 을 다음 테이블의 pk or unique key 검색 조건에 사용할 때

```sql
EXPLAIN
SELECT * FROM dept_emp de, employees e
WHERE e.emp_no = de.emp_no AND de.dept_no = 'd005';
```
|id|select_type|table|type|Key|key_len|ref|rows|Extra|
| - | - | - | - | - | - | - | - | - |
|1|SIMPLE|de|ref|PRIMARY|12|const|3333|Using where|
|1|SIMPLE|e|eq_ref|PRIMARY|4|emplyees.de.emp_no|1||

#### ref
- 조인의 순서와 관계없이 사용, pk, unique key 등의 조건도 없음
- 인덱스의 종류와 관계없이 Equal 조건으로 검색할 때는 ref 접근 방법
- 1건만 반환된다는 보장이 없으므로 const 나 eq_ref 보다는 빠르지 않음
  - Equal 이므로 매우 빠른 방법 중 하나

```sql
EXPLAIN
SELECT * FROM dept_emp
WHERE dept_no = 'd005';
```
|id|select_type|table|type|Key|key_len|ref|rows|Extra|
| - | - | - | - | - | - | - | - | - |
|1|SIMPLE|dept_emp|ref|PRIMARY|12|cpnst|3333|Using where|

#### fulltext
- fulltext index 를 사용하여 레코드를 읽은 접근 방법
- 쓸 일이 거의 없으므로 생략

#### ref_or_null
- ref 와 같은데 NULL 비교가 추가된 형태, 거의 안 씀
```sql
EXPLAIN
SELECT * FROM titles WHERE to_date = '1993-12-08' OR to_date IS NULL
```

#### unique_subquery
- WHERE 에 `IN` 형태 쿼리를 위한 접근 방식
- sub query 결과가 중복이 없을 때

#### index_subquery
- WHERE 에 `IN` 형태 쿼리를 위한 접근 방식
- sub query 결과가 중복이 있을 수 있을 때, 인덱스를 이용해 중복된 값을 제거

#### range
- index range scan (<, >, IS NULL, BETWEEN, IN, LIKE)
- MySQL 에서 우선순위는 낮으나, 상당히 빠름. 이것만 사용해도 어느 정도읫 ㅓㅇ능은 보장

#### index_merge
- 2개 이상의 인덱스를 이용해 결과를 만들어 내고, 결과를 병합
- 특징
  - 여러 인덱스를 읽어야 하므로 range 보다 효울성이 떨어짐
  - AND, OR 연산이 복잡하게 연결된 쿼리에서는 최적화되지 못할 때 있음
  - fulltext 에서는 적용 안됨
  - 2개 이상의 집합을 위한 추가 작업이 필요
- 실제로는 ref_or_null 다음으로 우선순위를 가지나, 책의 저자의 경험 상 여기로 위치

#### index
- 인덱스를 처음부터 끝까지 읽는 index full scan 을 의미
- 효율적으로 인덱스를 사용한다는 의미가 아님
- 사용되는 경우
  - range/const/ref 와 같은 접근 방식으로 인덱스를 사용하지 못하는 경우 +
    - 인덱스에 포함된 column 만으로 처리할 수 있는 쿼리인 경우 (추가 데이터 파일 안읽어도 됨)
    - 인덱스를 이용해 정렬이나 그룹핑 작업이 가능한 경우 (별도 정렬 작업을 피할 수 있음)

#### ALL
- FULL TABLE SCAN

### possible_kyes
- MySQL Optimizer 가 실행 계획에 사용할 후보 인덱스 목록
- 모든 인덱스가 표시되는 경우가 많기에 무시하는게 좋음

### key
- 최종 선택된 인덱스

### key_len
- 쿼리를 처리하기 위해 다중 column 으로 구성된 인덱스에서 몇 개의 column 까지 사용했는데
  - 몇 바이트까지 사용했는지
- 아래는 dept_no + emp_no 2개의 column 으로 만들어진 pk 중에서 dept_no 만 사용하는 예시
  - key_len = 12 -> 앞 쪽 12바이트만 유효하게 사용
  - dept_no CHAR(4), utf8 charset 은 1 ~ 3 바이트 가변적 = 4 * 3 = 12
```sql
EXPLAIN
SELECT * FROM dept_emp WHERE dept_no = 'd005';
```
|id|select_type|table|type|Key|key_len|ref|rows|Extra|
| - | - | - | - | - | - | - | - | - |
|1|SIMPLE|de|ref|PRIMARY|12|const|3333|Using where|

- 전부 쓰인 경우 16
```sql
EXPLAIN
SELECT * FROM dept_emp WHERE dept_no = 'd005' AND emp_no = 1001;
```
|id|select_type|table|type|Key|key_len|ref|rows|Extra|
| - | - | - | - | - | - | - | - | - |
|1|SIMPLE|de|const|PRIMARY|16|const,const|1|Using where|

### ref
- type 이 ref 면 어떤 값이 제공됐는지 보여줌
- 상수면 const, 다른 테이블의 column 이면 테이블 명과 column 명
- 조건에 식이 들어간 경우 `func` 로 표시되는데, 되도록 이런 경우는 피하는게 좋음

### rows
- 실행 계획의 효율성 판단을 위해 예측했던 레코드 건수
- 통계 정보를 참조하여 MySQL Optimizer 가 산출한 것이어서 정확하진 않음
- 레코드 예측치가 아니라 쿼리를 처리하기 위해 **얼마나 많은 레코드를 디스크로부터 읽고 체크해야하는지**

### Extra
- 성능에 관련된 중요한 내용이 표시됨
- 책에 매우 많은 종류가 있으나 대부분 거의 볼 일이 없고, 자주 보는 것만 추가함

#### Using filesort
- ORDER BY 를 처리하기 위해 적절한 인덱스를 사용하지 못할 때 -> MySQL Server 가 정렬
- **메모리 버퍼에 복사에 quicksort 알고리즘 수행**
- 쿼리를 튜닝하거나 인덱스를 생성하는 것이 좋음

#### Using index (covering index)
- 데이터 파일을 전혀 읽지 않고 인덱스만 읽어서 쿼리를 모두 처리할 수 있을 때
  - covering index
- 인덱스에 없는 column 값은 추가적인 disk read 가 발생
- InnoDB 의 경우 Clustering index 로 구성
  - 레코드의 pk 를 가지고 있음

#### Using temporary
- 쿼리를 처리하는 동안 중간 결과를 담아두기 위해 임시 테이블 사용 (memory or disk)

#### Using where
- MySQL engine 레이어에서 별도의 가공으로 필터링 작업을 처리한 경우
- storage engine 에서 넘어온 결과를 MySQL engine 추가적으로 필터링
- Filtered column 으로 성능 이슈를 확인할 수 있음 (아래에서 확인)

### EXPLAIN EXTENDED (Filtered)
- MySQL engine 에 의해서 얼마나 필터링되었는지
- 최종적으로 레코드가 얼마나 남았는지에 대한 비율 (%), rows 와 동일하게 추정치임 (실제와 다를 수 있음)
- 아래 예는 100 건 읽은 후 20% 만 남음
```sql
EXPLAIN
SELECT * FROM employees
WHERE AND emp_no BETWEEN 11 AND 110 AND gender = 'M';
```
|id|select_type|table|type|Key|key_len|ref|rows|filtered|Extra|
| - | - | - | - | - | - | - | - | - | - |
|1|SIMPLE|employees|range|PRIMARY|4|NULL|100|20|Using where|

### EXPLAIN EXTEDNED (추가 Optimizer 정보)
- 분석된 Parse tree 를 재조합하여 쿼리 문장과 비슷한 순서대로 나열해서 보여주는 기능
- Optimizer 가 쿼리를 어떻게 해석했고, 어떻게 쿼리를 변환했으며 어떤 처리가 수행했는지 판단 가능

### EXPLAIN PARTITIONS
- 파티션 테이블의 실행 계획 정보 확인
- 파티션 테이블에 실행되는 쿼리가 얼마나 파티션 기능을 잘 활용하고 있는지 확인