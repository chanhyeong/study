## MySQL 의 주요 처리 방식
- 다음 방식 중 full table scan 외 나머지는 MySQL 엔진에서 처리
  - 쿼리 성능을 저하 시켜 성능에 미치는 영향이 큼

### Full table scan
- 인덱스를 사용하지 않고 테이블을 처음부터 끝까지 읽어서 처리

#### 케이스
- 레코드 수가 너무 작아서 인덱스보다 처리가 빠른 경우
- WHERE, ON 에 인덱스를 이용할 적절한 조건이 없는 경우
- range 스캔이 가능해도 Optimizer 가 판단한 레코드 수가 너무 많은 경우
  - max_seeks_for_key 을 작게 설정할 수록 MySQL 서버가 인덱스를 사용하도록 유도할 수 있음

#### InnoDB 에서의 처리 방식
- 특정 테이블의 연속된 데이터 페이지가 읽히면, 백그라운드 thread 에 의해 Read ahead 작업이 자동으로 시작
- **Read ahead**: 데이터 수요를 예측하여 요청이 오기 전에 미리 디스크에서 읽어 InnoDB buffer pool 에 두는 것
- Full table scan 실행 시 처음은 Foreground thread 가 읽어오지만 특정 시점 이후 부턴 Background thread 로 넘김

### ORDER BY 처리 (Using filesort)
||장점|단점|
| - | - | - |
|index|INSERT, UPDATE, DELETE 실행 시 <br>이미 정렬되어 있어 빠름|INSERT, UPDATE, DELETE 작업 시<br>index 추가/삭제 작업도 필요, 느림<br>index 때문에 디스크/캐시 메모리가 더 필요|
|filesort|index 단점이 없음<br>레코드가 적으면 충분히 빠름|대상이 많아질 수록 쿼리 응답 속도가 느림|

#### Sort buffer
- 정렬을 수행하기 위한 별도 메모리 공간, 실행 후 시스템으로 반납, `sort_buffer_size`
- sort buffer 보다 레코드가 많다면 디스크로 임시 저장
- 크기를 늘려도 속도에는 별 효과가 없음
  - 저자의 경험상 56KB ~ 1MB 가 적절

#### Sort algorithm
- Single pass
  - sort buffer 에 정렬 기준 컬럼 + select 컬럼을 모두 담아서 정렬
- Two pass
  - 정렬 대상과 pk 값만을 sort buffer 에 담아서 정렬 수행
  - 정랠된 pk 순서대로 SELECT 할 컬럼을 가져옴
  - 예전 방식

#### 정렬 처리 방식
|처리 방법|Extra comment|
| - | - |
|index 사용|-|
|driving table 만 정렬<br>(join 이 없는 경우 포함)|Using filesort|
|join 결과를 임시 테이블에 저장 후<br>임시 테이블에서 정렬|Using temporary; Using filesort|

- 1차로 index 사용 검토
- index 사용이 불가능한 경우
  - driving table 만 정렬한 다음 join
  - join 이 끝나고 일치하는 레코드를 가져온 후 정렬

##### index 이용 정렬
- `ORDER BY` 에 명시된 컬럼이 driving table (제일 먼저 읽는 테이블) 에 속하고, ORDER BY 의 순서대로 생성된 index 가 있어야 함
- WHERE 절에 첫 번째 테이블에 대한 조건이 있다면, ORDER BY 는 같은 인덱스를 사용할 수 있어야 함
- B-Tree 가 아닌 hash, fulltext index 에서는 불가능
- 여러 테이블이 조인되는 경우에는 Nested-loop 에서만 사용 가능

```sql
SELECT * FROM employees e, salaries s
WHERE s.emp_no = e.emp_no
AND e.emp_no BETWEEN 102 AND 1020
ORDER BY e.emp_no
```

##### driving table 만 정렬
- join 시 레코드 수가 늘어나므로, join 실행 전 첫 번째 테이블의 레코드를 먼저 정렬하면 더 나음
- driving table 의 컬럼만으로 `ORDER BY` 가 작성되야 함

```sql
SELECT * FROM employees e, salaries s
WHERE s.emp_no = e.emp_no
AND e.emp_no BETWEEN 102 AND 1020
ORDER BY e.last_name
```
1. emp_no range 로 수를 줄이고
2. 정렬
3. table join


##### temporary table 을 이용한 정렬
- 2개 이상을 join 해서 정렬할 때 `ORDER BY` 의 기준이 driven 테이블인 경우

```sql
SELECT * FROM employees e, salaries s
WHERE s.emp_no = e.emp_no
AND e.emp_no BETWEEN 102 AND 1020
ORDER BY s.salary
```

1. join 하여 임시 테이블 생성
2. 임시 테이블 정렬

#### 정렬 방식의 성능 비교
- 정렬에서 index 사용 시에는 바로 Streaming, 아닌 경우는 Buffering

##### Streaming
- 일치하는 레코드가 검색될 때마다 바로바로 클라이언트로 전송해주는 방식
- LIMIT 을 걸면 쿼리의 전체 실행 시간을 상당히 줄일 수 있음

> JDBC 의 경우 자체적인 버퍼에 담아두고 모두 받으면 애플리케이션에 넘기기 때문에, MySQL 에서는 Streaming 으로 보내지만 그렇지 않은 것 처럼 보임  
> 버퍼링하는 이유? -> MySQL 은 무조건 보내고, JDBC 는 전송되는 데이터를 받아서 저장만 하므로 불필요한 네트워크 요청이 최소화되어 전체 처리량이 뛰어남

##### Buffering
- ORDER BY, GROUP BY 와 같은 쿼리
- 클라이언트가 아무것도 하지 않고 기다림

### GROUP BY 처리
- `HAVING`: GROUP BY 결과 필터링
  - 이걸 처리하려고 인덱스를 추가할 필요는 없음

#### index scan 이용 (tight index scan)
- join 의 driving table 에 속한 컬럼만 이용할 때 index 가 있으면 index 를 차례대로 읽으면서 수행하고 join
- Extra 에 아무것도 안나옴

#### loose index scan 이용
- loose index scan: index 의 레코드를 건너뛰면서 필요한 부분만 가져오는 것
- 넘어감

#### temporary table 을 이용
- index 를 전혀 사용하지 못할 때

```sql
SELECT e.last_name, AVG(s.salary)
FROM employees e, salaries s
WHERE s.emp_no = e.emp_no
GROUP BY e.last_name
```
|id|select_type|table|type|key|key_len|ref|rows|Extra|
|-|-|-|-|-|-|-|-|-|
|1|SIMPLE|e|ALL||||300|Using temporary; Using filesort|
|1|SIMPLE|s|ref|PRIMARY|4|e.emp_no|4||

1. employees full table scan
2. emp_no 로 salaries table 검색
3. 2 로 얻은 join 레코드를 temp table 에 저장. GROUP BY 절에 사용된 컬럼으로 UNIQUE KEY 를 생성
4. 1 ~ 3 을 join 이 완료될 때까지 반복

### DISTINCT 처리
- 특정 컬럼의 유니크 값 조회

#### SELECT DISTINCT ...
- GROUP BY 와 거의 같은 방식이나 정렬이 보장되지 않음
- 아래 쿼리 모두 같은 작업을 수행하고 인덱스를 이용하여 부가적인 정렬 작업이 필요하지 않음
```sql
SELECT DISTINCT emp_no FROM salaries;
SELECT emp_no FROM salaries GROUP BY emp_no;
```

- DISTINCT 는 SELECT tuple (row) 를 유니크하게 하지 컬럼을 유니크하게 하는 것이 아님
- 아래는 (first_name + last_name) 을 유니크하게 가져옴
```sql
SELECT DISTINCT first_name, last_name FROM employees;
SELECT DISTINCT(first_name), last_name FROM employees; # 이렇게 하더라도 위와 결과는 같음
```

#### 집합 함수와 함께 사용된 DISTINCT
- `COUNT(), MIN(), MAX()` 등
- 함수의 인자로 전달된 컬럼이 유니크한 것들만 가져옴

```sql
SELECT COUNT(DISTINCT s.salary), COUNT(DISTINCT e.last_name)
FROM employees e, salaries s
WHERE e.emp_no = s.emp_no
```

```sql
SELECT COUNT(DISTINCT s.salary), COUNT(DISTINCT e.last_name)
FROM employees e, salaries s
WHERE e.emp_no = s.emp_no
```

### 임시 테이블 (Using temporary)
- storage engine 에서 가져온 레코드를 정렬하거나 그룹핑할 때
- 메모리를 주로, 크면 디스크
- 메모리인 경우 MEMORY, 디스크인 경우 MyISAM
- 필요한 쿼리는 생략

#### 디스크에 생성되는 경우
- BLOB 이나 TEXT 와 같은 대용량 컬럼이 있는 경우
- 컬럼 중 길이가 512바이트 이상이 있는 경우
- 설정 값 보다 큰 경우 (tmp_table_size, max_heap_table_size)

#### 주의 사항
- 메모리에 생성되는 경우에는 거의 문제 없고, 디스크가 문제
- 메모리에 생성되는 경우, 모든 컬럼은 고정 컬럼임
  - VARCHAR(512) 면 512*3 을 차지함 (디스크에서는 실제 크기만큼만 먹지만)

### Table join
- MySQL 에서는 Nested-loop 방식만 지원
  - 두 개의 FOR or WHILE 과 같은 반복 루프 문장을 실행하는 형태로 join
- 바깥 쪽을 OUTER, 안 쪽을 INNER

#### INNER JOIN
- 선택되는 레코드가 inner table 에 의해 결정되는 경우
- outer 컬럼과 inner 컬럼이 일치되는 경우 join
- 없으면 포함 안함

#### OUTER JOIN
- inner 반대
- outer 컬럼과 inner 컬럼이 일치되는 경우 join
- inner 에 해당하는 값이 없으면 null
  - left, right 가 있으며 outer 테이블 기준

#### CARTESIAN JOIN
- FULL JOIN / CROSS JOIN
- 2개 테이블의 모든 레코드 조합을 결과로 가져오는 JOIN
- 성능에 무리가 갈 수 있으므로 조심해야 함 (쓰는 경우 못 봄)

```sql
SELECT * FROM departments WHERE dept_no = 'd001';
SELECT * FROM employees WHERE emp_no = 1001;

SELECT d.*, e.*
FROM departments d, employees e
WHERE dept_no = 'd001' AND emp_no = 1001
```

#### NATURAL JOIN
- 같은 컬럼 명으로 join 을 명시하는 inner join
```sql
SELECT *
FROM employees e INNER JOIN salaries s USING (emp_no)
```
- 책에서는 테이블 구조 변경 시 애플리케이션에 영향이 있을 수 있으므로 쓰지 말라고 하는데, 사실 컬럼명까지 바꿀 일이 거의 없음

#### join buffer 를 이용한 JOIN (Using join buffer)
- driven table 의 full table scan 이나 index full scan 을 피할 수 없다면, driving table 에서 읽은 레코드르 메모리에 캐시 -> drive table 과 메모리 캐시를 join