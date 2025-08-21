# PART 2 -2 SQL 활용
>## 1. 서브쿼리

* 개념 : 하나의 쿼리 안에 또 다른 쿼리. 바깥 쿼리는 메인쿼리.
    * 메인쿼리의 컬럼이 포함된 서브쿼리는 연관 서브쿼리, 포함되지 않은 서브쿼리는 비연관 서브쿼리라고 함
* 위치 
    * select 절 : 스칼라 서브쿼리
    * from : 인라인 뷰
    * where, having : 중첩 서브쿼리
    * ORDER BY, INSERT의 VALUES, 심지어 UPDATE의 SET, DELETE에도 사용 가능



> 예시
#### 1. SELECT 절 → **스칼라 서브쿼리 (Scalar Subquery)**

* **특징**: SELECT 절에서 컬럼처럼 사용되는 서브쿼리
* 반드시 \*\*단일 값(스칼라 값, 한 행 한 컬럼)\*\*을 반환해야 함
* 여러 행/컬럼을 반환하면 오류 발생

```sql
SELECT emp_name,
       (SELECT dept_name 
          FROM dept d 
         WHERE d.dept_id = e.dept_id) AS dept_name
FROM emp e;
```

👉 `dept_name`을 직접 조인하지 않고, SELECT 절에서 **하나의 값만 뽑아내는 쿼리**라서 "스칼라"라고 부름

---

#### 2. FROM 절 → **인라인 뷰 (Inline View)**

* **특징**: FROM 절에 서브쿼리가 들어와서 **마치 임시 테이블처럼 동작**
* 메인 쿼리에서 이 서브쿼리를 하나의 "뷰"처럼 참조할 수 있음

```sql
SELECT dept_id, avg_salary
FROM (
        SELECT dept_id, AVG(salary) AS avg_salary
        FROM emp
        GROUP BY dept_id
     ) tmp
WHERE avg_salary > 5000;
```

👉 `tmp`라는 **임시 테이블**을 만드는 것과 동일 → 그래서 "인라인 뷰"라고 부름

---

#### 3. WHERE / HAVING 절 → **중첩 서브쿼리 (Nested Subquery)**

* **특징**: 조건식 안에 들어가서, 메인쿼리의 행과 비교됨
* 보통 `IN`, `EXISTS`, `=`, `ANY`, `ALL` 등과 함께 쓰임

```sql
-- WHERE 절 예시
SELECT emp_name
FROM emp
WHERE dept_id IN (SELECT dept_id FROM dept WHERE location = 'SEOUL');

-- HAVING 절 예시
SELECT dept_id, AVG(salary)
FROM emp
GROUP BY dept_id
HAVING AVG(salary) > (SELECT AVG(salary) FROM emp);
```

👉 메인쿼리의 **조건을 결정하는 역할**이라 "중첩" 서브쿼리라고 부름

---

#### 🔑 정리

| 위치               | 이름       | 특징               |
| ---------------- | -------- | ---------------- |
| **SELECT**       | 스칼라 서브쿼리 | 단일 값 반환, 컬럼처럼 사용 |
| **FROM**         | 인라인 뷰    | 임시 테이블(뷰)처럼 동작   |
| **WHERE/HAVING** | 중첩 서브쿼리  | 조건식을 결정하는 데 사용   |

### 스칼라 서브쿼리
: 주로 select절에 위치, 컬럼이 올 수 있는 대부분 위치에 사용 가능
* 컬럼 대신 사용되므로 반드시 **하나의 값만을 반환해야함, 아니면 에러**
* 여러 값 반환할려면 서브쿼리 여러개 해야지

### 인라인 뷰
: from 절 등 테이블 명이 올 수 있는 위치에 사용 가능
### 중첩 서브 쿼리

A) WHERE, HAVING절에 사용 가능. 중첩 서브쿼리는 메인 쿼리와 관계에 따라 아래와 같이 구분
1) 연관 서브쿼리 : 메인쿼리의 컬럼 포함됨, 단독 실행 불가
    ```sql
    SELECT emp_name, salary
    FROM emp e
    WHERE salary > (
        SELECT AVG(salary)
        FROM emp
        WHERE dept_id = e.dept_id);
    ```
2) 비연관 서브쿼리  : 메인쿼리의 컬럼이 없음, 단독 실행 가능
    ```sql
    SELECT emp_name, salary
    FROM emp
    WHERE salary > (SELECT AVG(salary) FROM emp);
    ```
B) 반환하는 형태에 따라 3가지로 구분

1) 단일 행 서브쿼리 : 항상 1건 이하의 결과만 반환
    * 단일 행 비교 연산자(=, <, > 등)과 함께 사용
    ```sql
    select ~~~
    from~
    where price = (Select max(price) from product)
    ```
2) 다중 행 서브쿼리 : 2건 이상의 행 반환
    * 다중 행 비교 연산자와 함게 사용(```IN, ALL, ANY, SOME, EXISTS```)
3) 다중 컬럼 서브 쿼리
    * 서브쿼리가 여러 개의 컬럼을 반환하는 경우

    * 메인쿼리에서 복수 컬럼을 동시에 비교할 때 사용

    * 일반적인 단일 컬럼 서브쿼리보다 조건이 더 정교해짐**(하지만 단일행+다중컬럼일 경우 단일행 비교 연산자 사용 가능)**
    ```sql
    SELECT emp_id, emp_name, dept_id, salary
    FROM emp
    WHERE (dept_id, salary) IN (
            SELECT dept_id, MIN(salary)
            FROM emp
            GROUP BY dept_id
        );
    ```

    * 특징
        * (컬럼1, 컬럼2, …) 형태로 묶어서 비교

        * 반드시 서브쿼리에서 반환하는 컬럼 수와 메인쿼리에서 비교하는 컬럼 수가 같아야 함

        | 서브쿼리 유형    | 결과 형태  | 사용 연산자               |
        | ---------- | ------ | -------------------- |
        | 단일 행 서브쿼리  | 1행 1컬럼 | =, <, >, <=, >=, <>  |
        | 다중 행 서브쿼리  | N행 1컬럼 | IN, ANY, ALL, EXISTS |
        | 단일 행 다중 컬럼 | 1행 N컬럼 | = (튜플 비교)            |
        | 다중 행 다중 컬럼 | N행 N컬럼 | IN, EXISTS (튜플 비교)   |



---

# 📌 1. IN

* 서브쿼리 결과 집합 중 **하나라도 일치**하면 참
* 여러 개 값 비교할 때 많이 씀

```sql
SELECT emp_name
FROM emp
WHERE dept_id IN (SELECT dept_id FROM dept WHERE location_id = 1700);
```

👉 `dept_id`가 서브쿼리 결과에 속하면 조회

---

# 📌 2. ANY (또는 SOME)

* 서브쿼리의 여러 결과 중 **하나라도 만족**하면 참
* 보통 비교 연산자와 같이 사용

```sql
SELECT emp_name, salary
FROM emp
WHERE salary > ANY (SELECT salary FROM emp WHERE dept_id = 10);
```

👉 부서 10의 직원 중 **최소 한 명의 급여보다 크면** 조회됨
\= **최소값 기준 비교**

---

# 📌 3. ALL

* 서브쿼리의 모든 결과와 비교해서 **모두 만족**해야 참

```sql
SELECT emp_name, salary
FROM emp
WHERE salary > ALL (SELECT salary FROM emp WHERE dept_id = 10);
```

👉 부서 10의 직원 급여 **전부보다 커야** 조회됨
\= **최댓값 기준 비교**

---

# 📌 4. EXISTS

* 서브쿼리 결과가 **하나라도 존재하면 참**
* 보통 `EXISTS (SELECT 1 ...)` 형태로 많이 씀 (반환값은 중요하지 않음, "존재 여부"만 체크)

```sql
SELECT emp_name
FROM emp e
WHERE EXISTS (SELECT 1
                FROM bonus b
               WHERE b.emp_id = e.emp_id);
```

👉 `emp_id`가 `bonus` 테이블에 존재하는 직원만 조회

---

# 📌 요약 비교

| 연산자       | 의미             | 사용 예시               | 특징            |
| --------- | -------------- | ------------------- | ------------- |
| IN        | 집합 안에 포함되면 참   | `x IN (1,2,3)`      | 단순 포함 여부      |
| ANY(SOME) | 여러 값 중 하나라도 만족 | `x > ANY (서브쿼리)`    | 최소 기준         |
| ALL       | 여러 값 전부 만족     | `x > ALL (서브쿼리)`    | 최대 기준         |
| EXISTS    | 서브쿼리 결과 존재 여부  | `EXISTS (SELECT …)` | 조건 충족 행 존재 여부 |

---

✅ 한 줄 정리:

* **IN** → "값이 집합 안에 있니?"
* **ANY** → "값이 최소 하나랑 비교해도 되니?"
* **ALL** → "모든 값과 비교해도 되니?"
* **EXISTS** → "해당 조건을 만족하는 행이 있니?"

---

>## 2. 뷰 

1) 뷰란?
* SELECT 문을 저장해둔 가상 테이블
* 실제 데이터를 별도로 저장하지 않고, 원본 테이블(기본 테이블, base table)의 데이터를 보여주는 창 같은 개념
* 마치 테이블처럼 사용할 수 있음 (SELECT, JOIN 가능)

2) 특징
* 보안성 : 보안이 필요한 컬럼을 가진 테이블일 경우 해당 컬럼을 제외한 별도의 뷰 생성해 보안 유지
* 독립성 : 테이블 스키마가 변경될 경우 에플리케이션은 변경하지 않고 뷰만 수정
3) 편리성 : 복잡한 쿼리구문을 뷰 이름으로 단축시켜 가독성 높임

3) 뷰 생성 / 사용법

(1) 생성
```sql 
CREATE VIEW emp_dept_vw AS
SELECT e.emp_id, e.emp_name, d.dept_name
FROM emp e
JOIN dept d ON e.dept_id = d.dept_id;
```

👉 emp_dept_vw 라는 뷰 생성

👉 내부적으로는 emp 와 dept JOIN 쿼리 저장

(2) 사용
```sql
SELECT * 
FROM emp_dept_vw
WHERE dept_name = 'SALES';
```

👉 실제 테이블처럼 SELECT 가능

👉 실행 시점에 내부 SELECT 문이 수행되어 결과 반환

(3) 삭제
```DROP VIEW emp_dept_vw;```
>## 3. 집합 연산자
* 각 쿼리의 결과 집합을 가지고 연산을 하는 명령어

1) UNION ALL : 합집합 + 중복행 출력
2) UNION : 합집합, 중복행은 한줄로 출력
3) INTERSECT : 교집합, 중복행은 한줄로 출력
4) MINUS/EXCEPT : 차집합, 중복행은 한줄로 출력

### 1) UNION ALL/UNION

* UNION ALL:  QUERY1, QUERY2의 결과를 그대로 합침, 중복행도 그대로 출력됨
```sql
select * from running_man
union all
select * from infinite_Challenge
```
* UNION : QUERY1, QUERY2의 결과를 그대로 합침, 중복행은 한줄로 출력(중복 제거)
    * 중복된 행이 없을 땐 UNION ALL과 같은 결과를 도출하지만, UNION을 사용시 내부적으로 중복된 행을 찾아 제거하는 과정을 거쳐야 하므로 성능상 불리함

### 2) INTESECT
: QUERY 1, QUERY 2의 결과에서 공통된 부분만 중복을 제거하여 출력
```sql
select * from running_man
intersect
select * from infinite_Challenge
```
* 헤더값(출력되는 칼럼명)은 첫번째 쿼리를 따라간다
### MINUS/EXCEPT
: QUERY 1에서 QUERY 2의 결과를 제거하고 출력
```sql
select * from running_man
EXCEPT -- MINUS
select * from infinite_Challenge
```

>## 4. 그룹함수
: 데이터를 GROUP BY하여 나타낼 수 있는 데이터를 구하는 함수
* 여러 행(Row)을 묶어서 하나의 결과(값)로 나타내는 함수

* 개별 데이터가 아니라, 집단(그룹)에 대해 요약 통계를 내는 함수

* 그래서 GROUP BY 절과 자주 같이 쓰임

1) 집계함수: ```COUNT, SUM, AVG, MAX, MIN```
2) 소계(총계)함수 : ```ROLLUP, CUBE, GROUPING SETS```

### ROLLUP
* 소그룹간 소계 및 총계를 계산하는 함수
    * 위에서 아래로 총계가 쌓임
1) ROLLUP(A): A로 그룹핑, 총합계
2) ROLLUP(A,B) : A,B로 그룹핑, A로 그룹핑, 총합계
3) ROLLUP(A,B,C) : A.B,C로 그룹핑(세부적), A,B로 그룹핑(덜세부적), A로 그룹핑, 총합계

eg)
sales table
| A(지역) | B(지점) | C(판매원) | amt |
| ----- | ----- | ------ | --- |
| 서울    | 강남    | 홍길동    | 100 |
| 서울    | 강남    | 김철수    | 200 |
| 서울    | 종로    | 이영희    | 150 |
| 부산    | 해운대   | 박민수    | 300 |

* GROUP BY ROLLUP(A, B, C) 결과


| A(지역) | B(지점) | C(판매원) | SUM(amt)             |
| ----- | ----- | ------ | -------------------- |
| 서울    | 강남    | 홍길동    | 100                  |
| 서울    | 강남    | 김철수    | 200                  |
| 서울    | 강남    | NULL   | 300 ← 소계 (서울-강남 합계)  |
| 서울    | 종로    | 이영희    | 150                  |
| 서울    | 종로    | NULL   | 150 ← 소계 (서울-종로 합계)  |
| 서울    | NULL  | NULL   | 450 ← 소계 (서울 전체 합계)  |
| 부산    | 해운대   | 박민수    | 300                  |
| 부산    | 해운대   | NULL   | 300 ← 소계 (부산-해운대 합계) |
| 부산    | NULL  | NULL   | 300 ← 소계 (부산 전체 합계)  |
| NULL  | NULL  | NULL   | 750 ← 총합계            |

* **괄호로 묶으면 “그룹핑 단위를 하나의 패키지”처럼 취급합니다.**
📌 1. ROLLUP((A,B), C)
* 묶음 (A,B)를 하나의 컬럼 단위로 간주

* 전개 순서: (A,B), C → (A,B) → ()

    즉 , 결과 그룹:

    * (A, B, C)별 합계

    * (A, B)별 합계

    * 전체 총합계

⚠️ 여기서는 C만 단독으로 집계되지 않음 (왜냐하면 (A,B)는 항상 같이 움직이는 단위라서)

📌 2. ROLLUP(A, (B,C))

* 묶음 (B,C)를 하나의 단위로 취급

*전개 순서: A, (B,C) → A → ()

* 즉, 결과 그룹:

    * (A, B, C)별 합계

    * A별 합계

    * 전체 총합계

⚠️ 여기서는 (B,C)만 단독으로 합계되지 않음

### CUBE
: 소그룹간 총계를 다차원적으로 계산(다차원 집계)
* GROUP BY CUBE(컬럼들) 은 지정한 컬럼들의 가능한 모든 조합을 만들어 집계

* 데이터 분석에서 피벗(pivot) 요약처럼 “모든 차원별 소계”를 자동으로 뽑고 싶을 때 사용

1) CUBE(A)
```sql
SELECT A, SUM(amt)
FROM sales
GROUP BY CUBE(A);
```
* 그룹 조합
    * A별 합계

    *   전체 총합계 즉, ROLLUP(A)와 동일
    * 결과: (A), ()

2) CUBE(A, B)
```sql
SELECT A, B, SUM(amt)
FROM sales
GROUP BY CUBE(A, B);
```


* 그룹 조합

    (A,B)별 합계

    A별 합계

    B별 합계

    전체 총합계

> 즉, 2개 컬럼 → 2² = 4개 조합    

3) CUBE(A, B, C)
```sql
SELECT A, B, C, SUM(amt)
FROM sales
GROUP BY CUBE(A, B, C);
```

 * 그룹 조합

    (A,B,C)

    (A,B)

    (A,C)

    (B,C)

    (A)

    (B)

    (C)

    () (총합계)

> 즉, 3개 컬럼 → 2³ = 8개 조합

📌 ROLLUP vs CUBE 차이

* ROLLUP(A,B,C) → 계층적으로 요약: (A,B,C) → (A,B) → (A) → () (총 4단계)

* CUBE(A,B,C) → 가능한 모든 조합: 총 8개 조합 (2³)

>ROLLUP은 “위계적 요약”, CUBE는 “조합 전체 요약”

### GROUPING SETS
: 특정 항목에 대한 소계를 계산하는 함수이다. 인자 값으로 ROLLUP, CUBE를 사용할 수도 있다.
* ROLLUP, CUBE 는 자동으로 여러 집계를 만들어주지만,

* GROUPING SETS 는 내가 원하는 조합만 직접 지정해서 집계하는 기능이에요.

* 즉, 여러 GROUP BY 절을 UNION 하는 것과 같은 효과를 냅니다.

| A(지역) | B(지점) | C(직원) | amount |
| ----- | ----- | ----- | ------ |
| 서울    | 강남    | 홍길동   | 100    |
| 서울    | 강남    | 김철수   | 200    |
| 서울    | 종로    | 이영희   | 150    |
| 부산    | 해운대   | 박민수   | 300    |


1) GROUPING SETS(A,B) : A로 그룹핑, B로 그룹핑

| A    | B    | SUM(amount) |
| ---- | ---- | ----------- |
| 서울   | NULL | 450         |
| 부산   | NULL | 300         |
| NULL | 강남   | 300         |
| NULL | 종로   | 150         |
| NULL | 해운대  | 300         |

2) GROUPING SETS(A,B,()) : A로 그룹핑, B로 그룹핑, 총합계

| A    | B    | SUM(amount) |
| ---- | ---- | ----------- |
| 서울   | NULL | 450         |
| 부산   | NULL | 300         |
| NULL | 강남   | 300         |
| NULL | 종로   | 150         |
| NULL | 해운대  | 300         |
| NULL | NULL | 750         |

3) GROUPING SETS(A, ROLLUP(B)) : A로 그룹핑, B로 그룹핑, 총합계

| A    | B    | SUM(amount) |
| ---- | ---- | ----------- |
| 서울   | NULL | 450         |
| 부산   | NULL | 300         |
| NULL | 강남   | 300         |
| NULL | 종로   | 150         |
| NULL | 해운대  | 300         |
| NULL | NULL | 750         |
* 결과는 GROUPING SETS(A,B,()) 와 동일
4) GROUPING SETS(A, ROLLUP(B,C)) : A로 그룹핑, B,C로 그룹핑, B로 그룹핑, 총합계

| A    | B    | C    | SUM(amount) |
| ---- | ---- | ---- | ----------- |
| 서울   | NULL | NULL | 450         |
| 부산   | NULL | NULL | 300         |
| NULL | 강남   | 홍길동  | 100         |
| NULL | 강남   | 김철수  | 200         |
| NULL | 강남   | NULL | 300         |
| NULL | 종로   | 이영희  | 150         |
| NULL | 종로   | NULL | 150         |
| NULL | 해운대  | 박민수  | 300         |
| NULL | 해운대  | NULL | 300         |
| NULL | NULL | NULL | 750         |

5) GROUPING SETS(A,B,ROLLUP(C)) : A로 그룹핑, B로 그룹핑, C로 그룹핑, 총합계

| A    | B    | C    | SUM(amount) |
| ---- | ---- | ---- | ----------- |
| 서울   | NULL | NULL | 450         |
| 부산   | NULL | NULL | 300         |
| NULL | 강남   | NULL | 300         |
| NULL | 종로   | NULL | 150         |
| NULL | 해운대  | NULL | 300         |
| NULL | NULL | 홍길동  | 100         |
| NULL | NULL | 김철수  | 200         |
| NULL | NULL | 이영희  | 150         |
| NULL | NULL | 박민수  | 300         |
| NULL | NULL | NULL | 750         |

* ROLLUP 함수는 인수의 순서에 따라 결과가 달라지며, CUBE, GROUPING SETS는 인수의 순서가 바귀어도 같은 결과 출력
    1) CUBE(A,B) → (A,B), (A), (B), ()

    CUBE(B,A) → (B,A), (B), (A), ()
    👉 결과 조합은 동일 (A,B) ≡ (B,A)

    즉, CUBE 는 모든 가능한 조합을 전부 생성하기 때문에 순서가 중요하지 않음


    2) GROUPING SETS(A,B) → (A), (B)

    GROUPING SETS(B,A) → (B), (A)
    👉 같은 집합 결과 (집합은 순서가 없으므로 동일)


#### GROUPING
: ROLLUP, CUBE, GROUPING SETS등과 함께 쓰이며 결과 집합에 나오는 NULL 값이 진짜 데이터 NULL인지, 아니면 집계(소계/총계)를 의미하는 NULL인지 구분하기 위해 쓰는 함수
* ROW를 구분할 수 있게 해줌

```GROUPING(컬럼명)```

* 해당 컬럼이 집계 레벨에서 생긴 NULL 이면 1 반환, 실제 데이터의 NULL 이면 0 반환

eg)
| 지역(A) | 지점(B) | amount |
| ----- | ----- | ------ |
| 서울    | 강남    | 100    |
| 서울    | 종로    | 200    |
| 부산    | 해운대   | 300    |

```sql
SELECT A, B, SUM(amount),
       GROUPING(A) AS gA,
       GROUPING(B) AS gB
FROM sales
GROUP BY ROLLUP(A,B);
```
| A    | B    | SUM(amount) | gA | gB                  |
| ---- | ---- | ----------- | -- | ------------------- |
| 서울   | 강남   | 100         | 0  | 0                   |
| 서울   | 종로   | 200         | 0  | 0                   |
| 서울   | NULL | 300         | 0  | 1  ← B는 집계 NULL     |
| 부산   | 해운대  | 300         | 0  | 0                   |
| 부산   | NULL | 300         | 0  | 1  ← B는 집계 NULL     |
| NULL | NULL | 600         | 1  | 1  ← A,B 모두 집계 NULL |


1) CASE + GROUPING
```SQL
SELECT 
  CASE 
    WHEN GROUPING(dept_id) = 1 AND GROUPING(job_id) = 1 
      THEN '총합계'
    WHEN GROUPING(job_id) = 1 
      THEN dept_id || ' 부서 소계'
    ELSE dept_id || '-' || job_id
  END AS 구분,
  SUM(salary) AS 합계
FROM emp
GROUP BY ROLLUP(dept_id, job_id);
```
| 구분         | 합계    |
| ---------- | ----- |
| 10-CLERK   | 5000  |
| 10-MANAGER | 7000  |
| 10 부서 소계   | 12000 |
| 20-CLERK   | 4000  |
| 20 부서 소계   | 4000  |
| 총합계        | 16000 |


2) DECOD + GROUPING(ORACLE)
```sql
SELECT 
  DECODE(GROUPING(dept_id), 1, '총합계', dept_id) AS 부서,
  DECODE(GROUPING(job_id), 1, '소계', job_id)     AS 직무,
  SUM(salary) AS 합계
FROM emp
GROUP BY ROLLUP(dept_id, job_id);
```
| 부서  | 직무      | 합계    |
| --- | ------- | ----- |
| 10  | CLERK   | 5000  |
| 10  | MANAGER | 7000  |
| 10  | 소계      | 12000 |
| 20  | CLERK   | 4000  |
| 20  | 소계      | 4000  |
| 총합계 | 소계      | 16000 |

>## 5. 윈도우 함수
: 행은 그대로 두고 “부서별 합계/순위/누적값” 같은 요약을 옆 칼럼으로 붙이는 것, "OVER" 키워드와 함께 사용되며 
1) 순위함수
2) 집계 함수
3) 행 순서 함수
4) 비율 함수 
로 나눌 수 있다

* 기본 형태
```
<함수명>(컬럼) OVER (
    PARTITION BY 컬럼    -- 그룹 기준 (옵션) => 그룹 내 순위/집계 함
    ORDER BY 컬럼        -- 정렬 기준 (옵션)
    ROWS/RANGE BETWEEN ... -- 윈도우 프레임 (옵션)
)
```

### 1. 순위 함수 
: RANK() / DENSE_RANK() / ROW_NUMBER() 같은 순위 함수는 “순위를 매길 대상 컬럼”을 OVER 절의 ORDER BY 에서 지정합니다.
* 즉, RANK() 자체는 그냥 순위 계산기일 뿐, 무엇을 기준으로 순위를 매길지는 OVER 절이 결정합니다.

1) RANK : 순위를 매기며 같은 순위 존재시 존재 수만큼 다음 순위 건너뜀
```sql
SELECT emp_name, salary,
       RANK() OVER (ORDER BY salary DESC) AS 급여순위
FROM emp;
```

2) DENSE RANK : 같은 순위가 존재하더라도 다음 순위 건너 뛰지 않고 매김

| dept\_id | emp\_name | salary |
| -------- | --------- | ------ |
| 10       | A         | 5000   |
| 10       | B         | 4000   |
| 10       | C         | 4000   |
| 10       | D         | 3000   |
| 20       | E         | 6000   |
| 20       | F         | 5000   |
| 20       | G         | 5000   |
| 20       | H         | 4000   |

```sql
SELECT dept_id, emp_name, salary,
       DENSE_RANK() OVER ( -- 동점자는 같은 순위, 다음 순위는 건너뛰지 않고 연속
         PARTITION BY dept_id -- PARTITION BY dept_id → 부서별로 나눠서 순위 계산
         ORDER BY salary DESC --ORDER BY salary DESC → 급여 높은 순으로 정렬 
       ) AS 급여순위
FROM emp;
```
| dept\_id | emp\_name | salary | 급여순위 |
| -------- | --------- | ------ | ---- |
| 10       | A         | 5000   | 1    |
| 10       | B         | 4000   | 2    |
| 10       | C         | 4000   | 2    |
| 10       | D         | 3000   | 3    |
| 20       | E         | 6000   | 1    |
| 20       | F         | 5000   | 2    |
| 20       | G         | 5000   | 2    |
| 20       | H         | 4000   | 3    |


3) ROW_NUMBER : 순위를 매기면서 동일 값이라도 각기 다른 순위 부여, 말 그대로 행번호
* 날씨를 내림차순 하면 최신 날짜부터 오래된 날짜로 감
    * 최신 날짜 = 더 큼!!!(숫자가 단순히 더 크잖아 ㅋ)

### 2. 집계 함수
개념 : SUM(), AVG(), COUNT(), MAX(), MIN() 같은 집계 함수는 원래 GROUP BY랑 같이 쓰이죠. 그런데 윈도우 함수(OVER 절) 안에 넣으면,

* 그룹으로 묶지 않고도 "행 단위"로 집계 결과를 같이 볼 수 있습니다.
* ```집계함수(칼럼명) OVER () AS ~```
1) SUM

EG)

| dept\_id | emp\_name | sales |
| -------- | --------- | ----- |
| 10       | A         | 100   |
| 10       | B         | 200   |
| 10       | C         | 300   |
| 20       | D         | 400   |
| 20       | E         | 500   |
| 20       | F         | 600   |

WINDOW 집계함수 사용시
```SQL
SELECT dept_id, emp_name, sales,
       SUM(sales) OVER (PARTITION BY dept_id) AS 부서매출합계,
       AVG(sales) OVER (PARTITION BY dept_id) AS 부서평균매출
FROM sales;
```

| dept\_id | emp\_name | sales | 부서매출합계 | 부서평균매출 |
| -------- | --------- | ----- | ------ | ------ |
| 10       | A         | 100   | 600    | 200    |
| 10       | B         | 200   | 600    | 200    |
| 10       | C         | 300   | 600    | 200    |
| 20       | D         | 400   | 1500   | 500    |
| 20       | E         | 500   | 1500   | 500    |
| 20       | F         | 600   | 1500   | 500    |

* 행 수는 유지, 부서별 매출합계, 평균 나옴

* OVER절 내에 ORDER BY를 써서 누적값을 구할 수 있음
```sql
SUM(sales) OVER (
    PARTITION BY dept_id
    ORDER BY sales
    range BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```
    * ORDER BY : 누적/순차 계산 기준

    * RANGE : 현재 행 기준으로 "얼마나 앞뒤 행을 포함할지" 범위 설정

* UNBOUNDED PRECEDING : 맨 처음 행부터 현재 행까지의 모든 범위를 의미, 보통 누적합계(累積合計) 만들 때 사용

* 이때 SUM하는 컬럼을 ORDER BY절에 명시하면 RANGE~~이거 안해도 댐

2) MAX/MIN
```SQL
SELECT STUDENT_NAME,
    SUBJECT,
    SCORE,
    MAX(SCORE) OVER(PARTITION BY SUBJECT) AS MAX_SCORE
FROM SQLD
```
* 과목별 최대 점수를 받은 사람만 출력하는 쿼리
```sql
SELECT student, subject, score
FROM (
    SELECT student, subject, score,
           MAX(score) OVER (PARTITION BY subject) AS max_score
    FROM scores
) 
WHERE score = max_score;
```

3) AVG
* 과목별 평균 점수 이상을 받은 사람만 출력하는 쿼리
```sql
SELECT student, subject, score
FROM (
    SELECT student, subject, score,
           AVG(score) OVER (PARTITION BY subject) AS avg_score
    FROM scores
) t
WHERE score >= avg_score;
```

4) COUNT : 데이터 건수  구하는 함수
```sql
SELECT STUDENT_NAME,
    SUBJECT,
    SCORE,
    COUNT(*) OVER(PARTITION BY SUBJECT) AS PASS_COUNT
FROM SQLD
WHERE RESULT = "PASS"
```

>### WINDOWING 절을 이용해 집계하는 데이터의 범위를 지정 가능

* 현재 행이란? : 윈도우 함수에서 말하는 “현재 행(Current Row)” 은 단순히 SELECT 결과셋에서 ORDER BY 기준으로 정렬했을 때, 지금 계산 중인 그 행을 뜻합니다.

    📌     윈도우 함수는 행 단위로 실행돼요.
        한 행을 기준으로 “앞으로 몇 행까지 포함할까?” 혹은 “이 값까지 포함할까?”를 판단할 때 현재 행이라는 말을 써요.

 ```sql
    <집계함수>() OVER (
    PARTITION BY ...
    ORDER BY ...
    {ROWS | RANGE | GROUPS}
    BETWEEN <start> AND <end>
    )

```
1. 기준
    (ROWS / RANGE / GROUPS)

    | 기준         | 설명                    | 의미                                  |
    | ---------- | --------------------- | ----------------------------------- |
    | **ROWS**   | 행(Row) 개수 단위          | 현재 행을 중심으로 위·아래 몇 개 “행” 포함          |
    | **RANGE**  | ORDER BY 값의 “값 범위” 단위 | 현재 행의 값 ± 특정 범위까지 포함 (같은 값이면 같이 묶임) |
    | **GROUPS** | ORDER BY의 동일 값 집합 단위  | 같은 값끼리 하나의 그룹으로 묶고 그룹 단위로 범위 설정     |


2. 범위 지정 키워드

    | 키워드                     | 의미                                                |
    | ----------------------- | ------------------------------------------------- |
    | **UNBOUNDED PRECEDING** | 파티션의 맨 앞부터                                        |
    | **N PRECEDING**         | 현재 행 이전 N개 (ROWS 기준) / **N단위 이전 (RANGE/GROUPS 기준)**   |
    | **CURRENT ROW**         | 지금 계산 중인 행 (ROWS) / 현재 값 (RANGE) / 현재 그룹 (GROUPS) |
    | **N FOLLOWING**         | 현재 행 이후 N개 (ROWS 기준) / N단위 이후 (RANGE/GROUPS 기준)   |
    | **UNBOUNDED FOLLOWING** | 파티션의 맨 끝까지                                        |


* COUNT + WINDOWING절을 이용한 범위 지정
    * 범위 내에 현재 행이 포함된다면(BETWWEEN CURRENT ROW 등) 카운트 한다
eg) 과목별로 본인보다 점수가 높거나 같은 건수를 카운트 하는 쿼리
```sql
SELECT STUDENT_NAME,
    SUBJECT,
    SCORE,
    COUNT(*) OVER(PARTITION BY SUBJECT -- PARTITION BY를 쓰면 모든 연산은 그 파티션 안에서만 수행됩니다.
                ORDER BY SCORE DESC
                RANGE UNBOUNDED PRECEDING) AS HIGHER_COUNT -- RANGE → ORDER BY 값이 동일하면 같이 묶어서 처리
FROM SQLD
```
* ```RANGE UNBOUNDED PRECEDING``` = ```RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW```

결과
| STUDENT | SUBJECT | SCORE | CNT |
| ------- | ------- | ----- | --- |
| A       | 수학      | 90    | 1   |
| B       | 수학      | 80    | 2   |
| C       | 수학      | 70    | 3   |
| D       | 영어      | 95    | 1   |
| E       | 영어      | 85    | 2   |

+ 이떄 ```COUNT(*) OVER (PARTITION BY subject)```는 ORDER BY가 없으므로, 누적 개수 X 그냥 파티션 크기 그대로

+ ```COUNT(*) OVER (PARTITION BY subject ORDER BY score)``` : ORDER BY가 들어갔으므로 윈도우 프레임이 "시작 ~ 현재 행"으로 기본 설정됨


eg2) 본인 점수와 5점 이하로 차이나거나 점수가 같은 건수 카운트
```sql
SELECT STUDENT, SUBJECT, SCORE,
       COUNT(*) OVER (
         PARTITION BY SUBJECT
         ORDER BY SCORE
         RANGE BETWEEN 5 PRECEDING AND 5 FOLLOWING
       ) AS CNT_IN_RANGE
FROM scores;
```
* ROWS → 행(row) 단위 => "앞으로 5행, 뒤로 5행"

* RANGE → 값(value) 단위 (ORDER BY 컬럼 기준) => "현재 값 ±5 범위" (즉, 값의 차이로 범위 결정)

### 3. 행 순서 함수

1. FIRST_VALUE : 파티션 가장 선두끝에 위치한 데이터를 구하는 함수
* 과목별로 가장 높은 점수를 구하는 쿼리
```sql
SELECT NAME,
    SUBJECT,
    SCORE,
    FIST_VALUE(SCORE) OVER(PARTITION BY SUBJECT ORDER BY SCORE DESC) AS FIST_VALUE
FROM SQLD
```

2. LAST_VALUE : 파티션 가장 끝에 위치한 데이터 구함
* 이름만 보면 "파티션 마지막 값" 같지만, 실제로는 윈도우 프레임 내 마지막 값을 가져옴.

* 그런데 기본 윈도우 프레임(WINDOWING) DEFAULT값이 ```RANGE UNBOUNDED PRECEDING``` 이라서, 그냥 쓰면 "현재 행 값"이 나옴.
=> windowing 절 명시해야함


```sql
LAST_VALUE(col) OVER (
  ORDER BY col
  ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
)
```

3. LAG/LEAD : 파티션 별로 **"현재 행 기준으로 이전/뒤 행의 값을 가져오는 함수"**입니다. 즉, ORDER BY 기준으로 "앞/뒤를 돌아본다(LAG)" 라고 이해하면 돼요.

```sql
LAG(column, offset, default)
  OVER (PARTITION BY ... ORDER BY ...)
```
column : 참조할 컬럼

offset : 몇 행 이전 값을 가져올지 (기본값 1 → 바로 이전 행)

default : 이전 행이 없을 때 대신 표시할 값 (기본값 NULL)

PARTITION BY : 그룹 단위로 나누어 계산 (없으면 전체 기준)

ORDER BY : 순서를 정의해야 "이전 행"이 정해짐

* LAG : offset만큼 위의 행(앞의 행)을 본다
* LEAD : offset만큼 아래의 행(뒤의 행)을 본다

### 3. 비율 함수
1. RATIO_TO_REPORT : 파티션 별 합계에서 차지하는 비율을 구하는 함수, 0~1사이
* 과목별 score 합계에서 차지하는 비율을 구하는 쿼리
```sql
SELECT NAME,
    SUBJECT,
    SCORE,
    SUM(SCORE) OVER (PARTITION BY SUBJECT) AS SUM
    SCORE/SUM(SCORE) OVER(PARTITION BY SUBJECT) AS "SCORE/SUM",
    RATIO_TO_REPORT(SCORE) OVER(PARTITION BY SUBJECT) AS RATIO_TO_REPORT -- score/sum과 같은 결과가 나옴
FROM SQLD;
```
2. PERCENT_RAN : 해당 파티션의 맨 위 끝 행을 0, 맨 아래 끝 행을 1로 놓고 현재 행이 위치하는 백분위 순위 값을 구함, 0~1사이

3. CUME_DIST : 해당 파티션에서 누적 백분율 구하는 함수, 0~1사이 값 가짐
    * 현재 행까지 누적 건수/전체 건수임

4. NTILE
* NTILE(n) : 결과 집합을 n등분으로 나누는 윈도우 함수, 순서(ORDER BY) 기준으로 데이터를 나눔

* 각 행에 "몇 번째 구간(tile)"에 속하는지를 반환

    👉 즉, 분위수(quartile, percentile) 계산에 자주 쓰입니다.

```sql
NTILE(n) OVER (PARTITION BY col ORDER BY col2)
```
n : 나눌 구간 수 (예: 2 → 반으로 나누기, 4 → 4분위수, 100 → 백분위수)

PARTITION BY : 그룹별로 구간 나누기 (선택사항)

ORDER BY : 정렬 기준 (필수!)

eg) 학생 점수를 4분위수(Quartile)로 나누고 싶을 때:
```sql
SELECT student_name,
       score,
       NTILE(4) OVER (ORDER BY score DESC) AS quartile
FROM exam;
```

좋은 질문이에요 👏 **NTILE**이 실제 데이터에서 애매한 경우(행 수가 나누어떨어지지 않거나 동일 값이 있을 때) 어떻게 동작하는지 정리해드릴게요.

---
## NTILE +@
>1️⃣ 행 수가 **n으로 나누어떨어지지 않을 때**

* `NTILE(n)`은 **가능한 한 균등하게** 나눔.
* 나머지가 생기면, **앞쪽 그룹부터 1명씩 더 배정**.

예: 10명을 3등분 →

* 몫: 10 ÷ 3 = 3명씩
* 나머지: 1명 → 앞쪽 그룹에 추가
  👉 그룹 분배: `4명 / 3명 / 3명`

---

>2️⃣ 동일한 값(동점)이 있을 때

* **값이 같더라도** `ORDER BY`를 기준으로 순서를 매긴 후 그대로 그룹에 배정함.
* 즉, **NTILE은 값 자체가 아니라 행 단위로 분배**하는 함수라서, 같은 점수를 받았더라도 다른 그룹에 들어갈 수 있음.

예: 학생 7명 점수 → `95, 90, 90, 85, 80, 70, 60`
`NTILE(3)` →

| student | score | tile |                      |
| ------- | ----- | ---- | -------------------- |
| A       | 95    | 1    |                      |
| B       | 90    | 1    |                      |
| C       | 90    | 2    | ← 같은 점수지만 다음 그룹에 배정됨 |
| D       | 85    | 2    |                      |
| E       | 80    | 2    |                      |
| F       | 70    | 3    |                      |
| G       | 60    | 3    |                      |

👉 `RANK()`와 달리, **동점이라고 해서 같은 tile 번호를 보장하지 않음**.


>## 6. TOP-N 쿼리
개념 : topp n개의 데이터를 추출하는 쿼리

### 1. ROWNUM()
* ROWNUM = PSEUDO(가짜) 컬럼임 -> 존재 하지 않는 가짜 컬럼이라는 뜻
* index 느낌으로 행 번호를 매겨주는 함수

>왜 WHERE ROWNUM = 5가 안 되는가?

이유는 ROWNUM이 부여되는 방식 때문이에요.
```
WHERE 절을 먼저 실행 → 조건 검사

조건을 만족하는 행이 있으면 그 행에 ROWNUM = 1 부여

그다음 행은 ROWNUM = 2, 이런 식으로 번호 매겨짐
```
    즉, ROWNUM은 조건 필터링이 끝나야 매겨지는 번호라서:

>ROWNUM = 5라고 쓰면 → 처음부터 "5번 행을 가져와야 한다"는 조건인데, 애초에 행이 추가되면서 차례대로 1부터만 부여되기 때문에 5라는 값은 최초엔 절대 만족할 수 없음. 그래서 결과는 항상 NULL이에요.

* 그래서 결국 <, <= 조건은 가능함

✅ 그럼 "5번째 행만" 가져오려면?

ROWNUM만으로는 불가능하고, 서브쿼리 또는 ROW_NUMBER() 윈도우 함수를 써야 해요.


#### 1. FROM 절에 서브쿼리 + ORDER BY
```SQL
SELECT *
FROM (
    SELECT e.*, ROWNUM rn
    FROM (
        SELECT * 
        FROM emp
        ORDER BY score DESC
    ) e
)
WHERE rn <= 5;
```
* 안쪽 서브쿼리에서 먼저 ORDER BY score DESC 실행 → 점수 순으로 정렬된 결과셋 나옴

* 그 결과에 ROWNUM 부여 → 1,2,3,4,5,...

* 바깥쪽에서 WHERE rn <= 5 조건 적용 → 성적순 Top 5 학생 추출 ✅ (정상 동작)

#### 2. 서브쿼리 없이 직접 ROWNUM <= 5, ORDER BY
```SQL
SELECT *
FROM emp
WHERE ROWNUM <= 5
ORDER BY score DESC;
```
* FROM으로 emp 불러옴

* WHERE ROWNUM <= 5 먼저 실행됨 → 앞에서 가져온 5개 행에만 ROWNUM 부여

* 이미 5개 행만 확정됨 → 그다음에 ORDER BY score DESC 적용됨

* 결과: "점수순으로 정렬된 5명"이 아니라, "임의로 뽑힌 5명"을 점수순 정렬 ⚠️



### 2. 윈도우 함수의 순위 함수
: 앞서 나온 윈도우 함수의 row_number, rank, dense rank 등의 함수로 top-N쿼리 작성 가능

1. ROW_NUMBER() : 중복 점수 고려 ❌, 무조건 순서 번호, 같은 점수여도 다른 번호
```sql
SELECT *
FROM (
    SELECT NAME, SUBJECT, SCORE,
           ROW_NUMBER() OVER (PARTITION BY SUBJECT ORDER BY SCORE DESC) AS RN
    FROM STUDENT
)
WHERE RN <= 3;
```
2. RANK() : 중복 점수 고려 ⭕, 순위 건너뜀
```sql
SELECT *
FROM (
    SELECT NAME, SUBJECT, SCORE,
           RANK() OVER (PARTITION BY SUBJECT ORDER BY SCORE DESC) AS RK
    FROM STUDENT
)
WHERE RK <= 3;
```

3. DENSE_RANK() : 중복 점수 고려 ⭕, 순위 연속
```sql
SELECT *
FROM (
    SELECT NAME, SUBJECT, SCORE,
           RANK() OVER (PARTITION BY SUBJECT ORDER BY SCORE DESC) AS RK
    FROM STUDENT
)
WHERE RK <= 3;

```
>## 7. 셀프 조인
개념 : 같은 테이블을 자기 자신과 조인하는 것. from절에 같은 테이블이 두번 이상 등장하기 때문에 alias 표기해야함
* 보통 계층 구조(예: 직원–관리자 관계)나 행 간 비교가 필요할 때 사용합니다.

* 예제 1: 직원과 관리자 찾기
Employee 테이블 구조:

* emp_id

* emp_name

* manager_id (→ 같은 Employee 테이블의 emp_id 참조)
```sql
SELECT e.emp_name AS employee,
       m.emp_name AS manager
FROM Employee e
LEFT JOIN Employee m
  ON e.manager_id = m.emp_id; -- 같은 Employee 테이블을 두 번 불러서 사원과 그의 상사를 연결한 것.  
```
| employee | manager |
| -------- | ------- |
| 철수       | 영희      |
| 영희       | 민수      |
| 민수       | (NULL)  |

* 예제 2: 같은 부서에서 서로 다른 직원 짝짓기
```sql
SELECT e1.emp_name AS emp1,
       e2.emp_name AS emp2,
       e1.dept_id
FROM Employee e1
JOIN Employee e2
  ON e1.dept_id = e2.dept_id
 AND e1.emp_id  < e2.emp_id;
```
결과: 같은 부서 내 직원들을 페어링해서 보여줄 수 있음. (< 조건으로 중복 제거)

* 예제 3 
```

EMPLOYEES 테이블에는 다음과 같은 컬럼이 있다.

employee_id : 직원의 고유 ID

name : 직원 이름

manager_id : 해당 직원의 상사의 employee_id

즉, 한 테이블 안에서 직원과 관리자의 관계를 표현하고 있다 -> self join
```
정답
```sql
SELECT B.employee_id AS manager_id,
       B.name        AS manager_name,
       A.employee_id,
       A.name
FROM employees A
JOIN employees B
  ON A.manager_id = B.employee_id;

```

* A : 직원(Employee) 역할
* B : 관리자(Manager) 역할

* 조건 A.manager_id = B.employee_id
→ 직원의 manager_id와 관리자의 employee_id를 연결
    * a의 매니저 id가 b의 id = a가 직원 역할


1. A.manager_id = B.employee_id

    A = 직원

    B = 관리자

    결과: 직원 → 그 직원의 상사

2. A.employee_id = B.manager_id

    A = 관리자

    B = 직원

    결과: 관리자 → 그 관리자가 관리하는 부하 직원

즉, 조인 조건이 달라지면 어느 쪽이 “상사 테이블” 역할인지도 달라진다.

>## 8. 계층 쿼리
: 테이블 내 계층 구조를 이루는 컬럼이 존재할 경우 계층 쿼리를 이용해 데이터를 출력할 수 있다.
    * self join을 쉽게 표현 가능

좋습니다 🙆 계층 쿼리에서 자주 쓰이는 핵심 키워드들을 하나씩 풀어드릴게요.

---

### LEVEL

* 오라클 계층 쿼리에서 제공하는 **가상 컬럼**
* 현재 행이 \*\*트리에서 몇 번째 단계(깊이)\*\*에 속하는지 보여줌
* 루트 노드 = 1, 그 자식은 2, 그 아래는 3 … 이런 식

예:

```
Steven (LEVEL=1)
  Neena (LEVEL=2)
    Michael (LEVEL=3)
```



### SYS\_CONNECT\_BY\_PATH

* 루트에서 현재 노드까지의 \*\*경로(Path)\*\*를 문자열로 반환
* 구문:

  ```sql
  SYS_CONNECT_BY_PATH(column, 구분자)
  ```
* `구분자`를 기준으로 루트부터 현재 행까지의 계층 경로 표시



### START WITH

* 계층 구조에서 **탐색을 시작할 루트 행**을 지정
* 보통 최상위 노드를 찾을 조건을 넣음 (예: `manager_id IS NULL`)

예:

```sql
START WITH manager_id IS NULL
```

→ 상위 관리자가 없는 CEO부터 시작



### CONNECT BY

* 계층을 따라 내려가면서 **부모-자식 관계를 정의**
* 보통 `CONNECT BY PRIOR parent_column = child_column` 형태

예:

```sql
CONNECT BY PRIOR employee_id = manager_id
```

→ “이전 행의 employee\_id가 현재 행의 manager\_id와 같으면 연결”



### PRIOR

* CONNECT BY 안에서 쓰이며, \*\*부모 노드(이전 단계)\*\*를 가리키는 키워드
* `PRIOR employee_id = manager_id`
  → 부모의 employee\_id = 자식의 manager\_id

반대로 하면:

* `PRIOR manager_id = employee_id`
  → 자식에서 부모로 올라가는 관계 설정 가능

네 좋아요 🙆 이어서 **CONNECT\_BY\_ROOT**, **CONNECT\_BY\_ISLEAF**까지 정리해 드릴게요.



### CONNECT\_BY\_ROOT

* 현재 노드에서 거슬러 올라갔을 때의 **최상위 루트 노드의 값**을 반환합니다.
* 구문:

  ```sql
  CONNECT_BY_ROOT column
  ```
* 보통 "이 직원의 최상위 사장은 누구인가?" 같은 질문에 유용합니다.

예시:

```sql
SELECT employee_id,
       name,
       CONNECT_BY_ROOT name AS top_manager
FROM employees
START WITH manager_id IS NULL
CONNECT BY PRIOR employee_id = manager_id;
```

👉 결과:

| employee\_id | name    | top\_manager |
| ------------ | ------- | ------------ |
| 100          | Steven  | Steven       |
| 101          | Neena   | Steven       |
| 201          | Michael | Steven       |



### CONNECT\_BY\_ISLEAF

* 현재 노드가 \*\*리프(leaf, 더 이상 자식이 없는 말단 노드)\*\*인지 여부를 반환합니다.
* 값:

  * `1` → 리프 노드 (자식 없음)
  * `0` → 리프 아님 (자식 있음)

예시:

```sql
SELECT employee_id,
       name,
       CONNECT_BY_ISLEAF AS is_leaf
FROM employees
START WITH manager_id IS NULL
CONNECT BY PRIOR employee_id = manager_id;
```

👉 결과:

| employee\_id | name    | is\_leaf |
| ------------ | ------- | -------- |
| 100          | Steven  | 0        |
| 101          | Neena   | 0        |
| 201          | Michael | 1        |


### 역방향 전개 : 리프 -> 루트로 올라가게 하려면
```SQL
SELECT employee_id,
       name,
       LEVEL,
       SYS_CONNECT_BY_PATH(name, ' -> ') AS path
FROM employees
START WITH employee_id = 201              -- 리프 노드(말단 직원)부터 시작
CONNECT BY PRIOR manager_id = employee_id; -- 위로 올라가는 규칙
```
    START WITH employee_id = 201 → 말단 직원 Michael부터 시작

    CONNECT BY PRIOR manager_id = employee_id

    여기서 PRIOR manager_id = 부모 행의 manager_id

    employee_id = 현재 행의 employee_id

    즉, 현재 직원의 employee_id가 이전 단계(manager_id)에 해당하면 연결 → 위로 올라감

### 계층 쿼리의 정렬
* order by 쓰지 않는다
* 대신 **ORDER SIBLINGS BY**절 사용하여 같은 레벨끼리 정렬되도록 작성 가능

```sql
SELECT employee_id,
       name,
       manager_id,
       LEVEL
FROM employees
START WITH manager_id IS NULL
CONNECT BY PRIOR employee_id = manager_id
ORDER SIBLINGS BY name;
```
| LEVEL | name      | manager\_id |
| ----- | --------- | ----------- |
| 1     | Steven    | (NULL)      |
| 2     | Alexander | 100         |
| 2     | Lex       | 100         |
| 2     | Neena     | 100         |



>## 9. PIVOT, UNPIVOT 절
### 1. PIVOT
* pivot = 회전하다 : 행이 열로 변환되어 가로가 길쭉한 형태로 변하게 된다

* 기본 문법 
```
SELECT *
FROM (
    SELECT column1, column2, value_column
    FROM some_table
)
PIVOT (
    AGGREGATE_FUNCTION(value_column)
    FOR col IN ( '값1' AS alias1, '값2' AS alias2, ... )
);
```
* value_column : 집계할 값

* AGGREGATE_FUNCTION : SUM, COUNT, AVG 같은 집계 함수

* FOR col IN (...) 
    * col : pivot할 컬럼을 지정
    * IN() : PIVOT할 컬럼 값을 지정



* 예시 

데이터 (employees)

```text
dept    gender  salary
HR      M       3000
HR      F       3200
IT      M       4000
IT      F       4200
IT      M       4100
```

쿼리

```sql
SELECT *
FROM (
    SELECT dept, gender, salary
    FROM employees
)
PIVOT (
    COUNT(salary)
    FOR gender IN ('M' AS male, 'F' AS female)
);
```

결과

```text
dept    male    female
HR      1       1
IT      2       1
```

👉 `gender` 값(M/F)이 열로 바뀌고, `COUNT`로 집계됨.

### pivot 컬럼명 규칙
* 기본적으로 집계함수에 붙은 별칭은 헤더명 끝에 언더바와 함께 표시된다. 집계함수에 별칭이 지정되지 않고 IN절에만 지정되었다면 IN절 별칭만 나온다.

    | 집계함수 별칭 | IN 절 별칭 | 최종 헤더명 형식                       | 예시 컬럼명                 |
    | ------- | ------- | ------------------------------- | ---------------------- |
    | 있음      | 있음      | IN절 별칭 + '\_' + 집계별칭            | male\_CNT, female\_CNT |
    | 없음      | 있음      | IN절 별칭만 표시                      | male, female           |
    | 있음      | 없음      | 집계별칭만 표시                        | CNT                    |
    | 없음      | 없음      | 기본 시스템 생성 이름 (예: COUNT(SALARY)) | COUNT(SALARY)          |

### PIVOT 절에서 필터링
: pivot 절에도 where절로 필터링 가능
### PIVOT과 여러 집계

* 한 번의 PIVOT에서 SUM, AVG 같이 여러 집계를 동시에 낼 수 있음.
```sql
PIVOT (SUM(SALARY) AS SAL, AVG(SALARY) AS AVG
       FOR DEPT_NAME IN ('IT개발팀' AS IT, '클라우드팀' AS CLOUD)) -- IT_SAL, IT_AVG, CLOUD_SAL, CLOUD_AVG 열이 생김.
```
### PIVOT + 여러 기준

* FOR 절에도 컬럼을 추가해 기준 컬럼을 2개 이상 줄수도 있음 → 그룹핑 기준을 늘려 열이 늘어나면서 세분화.
```sql
PIVOT (AVG(SALARY) AS AVG
       FOR (DEPT_NAME, GRADE) 
       IN (('IT개발팀','과장') AS IT1,
           ('IT개발팀','차장') AS IT2,
           ('AI연구팀','과장') AS AI1))
```

### PIVOT에서 여러 행 출력되는 경우

* 기본 개념

    PIVOT은 **행을 열로 회전(pivoting)시키는 것**이지만,
    행을 완전히 하나로 합쳐버리는 게 아니라, **집계 기준이 아닌 다른 컬럼이 남아 있으면 그대로 유지**합니다. 즉

* PIVOT 기준 컬럼(`DEPT_NAME`, `GRADE`) → 열로 변환됨
* 집계 함수(`AVG(SALARY)`) → 값 계산됨
* **인라인 뷰에 포함된 다른 컬럼(`HIRE_DATE` 같은거)** → 그대로 행에 남음

그래서 `HIRE_DATE`가 있으면, 같은 `DEPT_NAME + GRADE` 조합이라도 날짜별로 행이 여러 개 찍히는 겁니다.


* 예시 비교

#### (1) `HIRE_DATE` 포함

```sql
SELECT *
FROM (
    SELECT DEPT_NAME, GRADE, HIRE_DATE, SALARY
    FROM EMP_INFO
)
PIVOT (
    AVG(SALARY) AS AVG
    FOR (DEPT_NAME, GRADE) 
    IN (('IT개발팀','과장') AS IT1,
        ('IT개발팀','차장') AS IT2,
        ('AI연구팀','과장') AS AI1)
);
```

👉 결과:

| HIRE\_DATE | IT1\_AVG | IT2\_AVG | AI1\_AVG |
| ---------- | -------- | -------- | -------- |
| 2020-01-01 | 4500     | (null)   | (null)   |
| 2020-03-15 | (null)   | 5200     | (null)   |
| 2020-05-10 | (null)   | (null)   | 4800     |

➡ `HIRE_DATE`가 남아 있으므로 여러 행 출력됨.



#### (2) `HIRE_DATE` 제거

```sql
SELECT *
FROM (
    SELECT DEPT_NAME, GRADE, SALARY
    FROM EMP_INFO
)
PIVOT (
    AVG(SALARY) AS AVG
    FOR (DEPT_NAME, GRADE) 
    IN (('IT개발팀','과장') AS IT1,
        ('IT개발팀','차장') AS IT2,
        ('AI연구팀','과장') AS AI1)
);
```

👉 결과:

| IT1\_AVG | IT2\_AVG | AI1\_AVG |
| -------- | -------- | -------- |
| 4700     | 5200     | 4800     |

➡ `HIRE_DATE`가 없으니 한 줄로 요약됨.


### 2. UNPIVOT 절
: 열 데이터를 다시 행으로 변환하는 기능. 즉, PIVOT된 애를 다시 원본으로 되돌림.



예시 데이터 (EMP\_INFO\_PIVOT)

```text
GRADE   HR_SAL  IT_SAL  AI_SAL  CLOUD_SAL
과장     7000    8000    7500    (NULL)
대리     6000    (NULL)  (NULL)  6300
사원     (NULL)  5000    (NULL)  5000
차장     (NULL)  9000    9300    9500
```

---

#### UNPIVOT 적용 쿼리

```sql
SELECT GRADE, DEPT_NAME, SALARY
FROM EMP_INFO_PIVOT
UNPIVOT (
    SALARY FOR DEPT_NAME IN (HR_SAL, IT_SAL, AI_SAL, CLOUD_SAL)
)
ORDER BY GRADE;
```
* SELECT절에 있는 DEPT_NAME, SALARY는 실제 emp_info_pivot에 존재하는게 아니라 내가 설정한거
    * SALARY : 기존 집계 된 칼럼
    * DEPT_NAME : 기존 PIVOT할 컬럼 
* emp_info_pivot에 있는 컬럼은 IN절 안에 존재


#### 결과

```text
GRADE   DEPT_NAME   SALARY
과장     HR_SAL     7000
과장     IT_SAL     8000
과장     AI_SAL     7500
대리     HR_SAL     6000
대리     CLOUD_SAL  6300
사원     IT_SAL     5000
사원     CLOUD_SAL  5000
차장     IT_SAL     9000
차장     AI_SAL     9300
차장     CLOUD_SAL  9500
```

👉 `HR_SAL, IT_SAL, AI_SAL, CLOUD_SAL` 컬럼이 행으로 변환됨.
👉 `DEPT_NAME`, `SALARY` 컬럼은 UNPIVOT 구문에서 새로 만들어짐.

---

#### 옵션: ALIAS 지정

```sql
SELECT GRADE, DEPT_NAME, SALARY
FROM EMP_INFO_PIVOT
UNPIVOT (
    SALARY FOR DEPT_NAME IN (
        HR_SAL AS '인사팀',
        IT_SAL AS 'IT개발팀',
        AI_SAL AS 'AI연구팀',
        CLOUD_SAL AS '클라우드팀'
    )
)
ORDER BY GRADE;
```

결과

```text
GRADE   DEPT_NAME   SALARY
과장     인사팀     7000
과장     IT개발팀   8000
과장     AI연구팀   7500
대리     인사팀     6000
대리     클라우드팀 6300
...
```

👉 IN 절에서 별칭을 지정하면 변환된 `DEPT_NAME`에 그대로 반영됨.

### pivot vs unpivot
* pivot: 인라인 뷰, 집계함수, 데이터를 나열한 IN절
* unpivot : 결과 데이터에 출력될 헤더명 네이밍, IN절에는 기존 칼럼명 

#### 옵션: INCLUDE NULLS

* 기본적으로 UNPIVOT은 NULL 값은 제외
* `INCLUDE NULLS` 옵션을 주면 NULL도 행으로 출력됨

```sql
SELECT GRADE, DEPT_NAME, SALARY
FROM EMP_INFO_PIVOT
UNPIVOT INCLUDE NULLS
( SALARY FOR DEPT_NAME IN (HR_SAL, IT_SAL, AI_SAL, CLOUD_SAL) )
ORDER BY GRADE;
```



>## 10 정규 표현식
: 특정 규칙에 맞는 문자열 패턴을 정의하는 식

* 기본 연산자

| 연산자      | 의미                  | 예시                                             |        |                     |
| -------- | ------------------- | ---------------------------------------------- | ------ | ------------------- |
| `.`      | 임의의 한 글자            | `'a.c'` → `abc`, `axc` 매칭                      |        |                     |
| `*`      | 0회 이상 반복            | `'ab*'` → `a`, `ab`, `abb`, `abbb...`          |        |                     |
| `+`      | 1회 이상 반복            | `'ab+'` → `ab`, `abb`, `abbb...` (단 `a` 단독은 X) |        |                     |
| `?`      | 0회 또는 1회 등장         | `'ab?'` → `a`, `ab`                            |        |                     |
| `{m}`    | 정확히 m번 반복           | `'a{3}'` → `aaa`                               |        |                     |
| `{m,n}`  | m\~n번 반복            | `'a{2,4}'` → `aa`, `aaa`, `aaaa`               |        |                     |
| `^`      | 문자열 시작              | `'^abc'` → `abc...`로 시작하는 문자열                  |        |                     |
| `$`      | 문자열 끝               | `'xyz$'` → `...xyz`로 끝나는 문자열                   |        |                     |
| \|       | \|                  | OR 조건                                          | \`'cat | dog'`→`cat`또는`dog\` |
| `[...]`  | 문자 집합(1글자)          | `'[abc]'` → `a` 또는 `b` 또는 `c`                  |        |                     |
| `[^...]` | 제외 문자 집합            | `'[^0-9]'` → 숫자가 아닌 문자                         |        |                     |
### 1. REGEXP_SUBSTR
기본 문법 (Oracle 기준)
```sql
REGEXP_SUBSTR(source_string, pattern [, position [, occurrence [, match_param ]]])
```
| 인자              | 설명                                                                                   |
| --------------- | ------------------------------------------------------------------------------------ |
| `source_string` | 검색 대상 문자열                                                                            |
| `pattern`       | 찾을 정규표현식 패턴                                                                          |
| `position`      | 검색 시작 위치 (기본값 = 1, 즉 문자열 첫 글자부터)                                                     |
| `occurrence`    | 몇 번째 매칭 결과를 반환할지 지정 (기본값 = 1)                                                        |
| `match_param`   | 대소문자 구분 여부, 단일행/다중행 모드 등 옵션 (`'i'` = 대소문자 무시, `'c'` = 대소문자 구분, `'n'` = `.`이 개행 포함 등) |


```sql
select REGEXP_SUBSTR("SQL", "S|L") 
```
-> SQL이란 단어에서 S OR L을 찾아 첫번째로 일치하는 문자를 출력
-> "S"
```sql
select REGEXP_SUBSTR("SQL", "SQL|D") 
```
-> "SQL"

### 패턴을 나타내는 연산자


## 대괄호 `[...]`

* **문자 집합(Character class)**
* 안에 들어있는 문자 중 **하나만 매칭**됨.

예시:

```regex
[abc]   → a, b, c 중 하나
[0-9]   → 숫자 하나 (0 ~ 9)
[sq]l  → sl or ql 중 첫번째로 일치하는 문자열
```

---

## 부정 집합 `[^...]`

* 대괄호 안에서 `^`가 맨 앞에 있으면 **NOT(부정)** 의미
* 즉, 지정된 집합을 제외한 나머지 문자와 매칭

예시:

```regex
[^0-9]  → 숫자가 아닌 문자
[^abc]  → a, b, c 이외의 문자
📌 [^sq]l → sl도 ql도 아닌 l로 끝나는 두글자의 문자열을 찾아 출력한다
```

---

## 하이픈 `[-]`

* 대괄호 안에서는 **범위 지정**에 사용
* `a-z`, `0-9`처럼 연속된 범위를 표현 가능

예시:

```regex
[a-z]   → 소문자 알파벳 하나
[0-9]   → 숫자 하나
[A-F0-9] → 16진수 문자 하나 (0~9, A~F)
```

※ `-`를 문자 그대로 쓰고 싶으면 대괄호 맨 앞이나 맨 뒤에 배치:

```regex
[-AB]   → '-', 'A', 'B' 중 하나
[AB-]   → 'A', 'B', '-' 중 하나
```

---

## 소괄호 `( ... )`

* **그룹(Grouping)**
* OR 조건(`|`)을 묶거나, 반복(`*`, `+`, `{m,n}`) 범위를 지정

예시:

```regex
(cat|dog) → 'cat' 또는 'dog'
(ab)+     → 'ab', 'abab', 'ababab'
```

---

## 중괄호 `{m,n}`

* **반복 횟수 지정**

예시:

```regex
a{3}    → 'aaa'
a{2,4}  → 'aa', 'aaa', 'aaaa'
a{2,}   → 'aa' 이상 반복
```

---

## 정리 표

| 기호       | 의미       | 예시       | 매칭 결과         |                |
| -------- | -------- | -------- | ------------- | -------------- |
| `[...]`  | 문자 집합    | `[abc]`  | a, b, c 중 하나  |                |
| `[^...]` | 부정 문자 집합 | `[^0-9]` | 숫자가 아닌 문자     |                |
| `[-]`    | 범위 지정    | `[a-z]`  | 소문자 알파벳 하나    |                |
| `()`     | 그룹핑      | \`(cat   | dog)\`        | 'cat' 또는 'dog' |
| `{m,n}`  | 반복 횟수    | `a{2,4}` | aa, aaa, aaaa |                |


### POSIX 문자 클래스

| 연산자 (POSIX)     | 의미                            | 동일한 정규식 기호          |
| --------------- | ----------------------------- | ------------------- |
| `[:alnum:]`     | 영문자 + 숫자 (알파벳/숫자)             | `[A-Za-z0-9]`, `\w` |
| `[:alpha:]`     | 영문자 (알파벳)                     | `[A-Za-z]`          |
| `[:digit:]`     | 숫자 (0\~9)                     | `[0-9]`, `\d`       |
| `[:xdigit:]`    | 16진수 숫자 (0-9, A-F, a-f)       | `[0-9A-Fa-f]`       |
| `[:lower:]`     | 소문자 알파벳                       | `[a-z]`             |
| `[:upper:]`     | 대문자 알파벳                       | `[A-Z]`             |
| `[:blank:]`     | 공백(스페이스, 탭)                   | `[ \t]`             |
| `[:space:]`     | 공백 문자 전체 (스페이스, 탭, 개행 등)      | `\s`                |
| `[:punct:]`     | 구두점 문자 (.,;:?! 등)             | `[[:punct:]]`       |

* 구두점 문자: 문장 내 의미를 명확히하거나 문장 구조를 나타내는 기호를 의미하며 특수문자 포함한다

### 문자열 시작과 끝

^ : 문자열 시작

$ : 문자열 끝

^[A-Z] : 대문자로 시작

[A-Z]$ : 대문자로 끝

### 이메일, 전화번호 정규 표현식



```sql
-- 📌 이메일 정규표현식
[[:alnum:]._%+-]+@[[:alnum:].-]+\.[[:alpha:].]{2,}
```

| 기호          | 설명                                             |
| ----------- | ---------------------------------------------- |
| `[ ... ]`   | 문자 집합(Character Class). 안에 있는 어떤 문자든 한 글자에 매칭됨 |
| `[:alnum:]` | POSIX 문자 클래스. 알파벳 대소문자(A-Z, a-z) + 숫자(0-9)     |
| `.`         | 원래는 "임의의 한 문자"를 의미. 하지만 문자 집합 안에서는 마침표(`.`) 자체 |
| `_ % + -`   | 특수문자 그대로 허용됨                                   |
| `+`         | 앞 패턴이 1회 이상 반복됨. 예: `[a-z]+` → 소문자가 하나 이상      |
| `@`         | 리터럴 문자. 실제 `@` 기호와 매칭                          |
| `\.`        | 실제 마침표(`.`)와 매칭. (`.`는 임의 문자이므로 이스케이프 필요)      |
| `{2,}`      | 앞 패턴이 2회 이상 반복. 예: `[A-Z]{2,}` → 대문자 2개 이상     |
| `[:alpha:]` | 알파벳(A–Z, a–z)                                  |

---

```sql
-- 📌 전화번호 정규표현식
010[-]?[0-9]{3,4}[-]?[0-9]{4}
```

| 기호      | 설명                                                 |
| ------- | -------------------------------------------------- |
| `010`   | 문자열 그대로 "010"과 일치                                  |
| `[-]?`  | 문자 집합 `[-]` → 하이픈(`-`) 1개. `?` → 0번 또는 1번 나타날 수 있음 |
| `[0-9]` | 숫자 0부터 9까지 중 한 자리                                  |
| `{3,4}` | 숫자가 3자리 또는 4자리 반복됨 (`123`, `1234`)                 |
| `{4}`   | 숫자 4자리 고정                                          |



### 2. `REGEXP_REPLACE` 함수

문자열 내에서 정규표현식 패턴과 일치하는 부분을 **모두** 찾아 지정한 다른 문자열로 교체한다.
데이터 변환이나 텍스트 필드 내 특정 패턴을 찾아 바꾸는 데 유용하다.

```sql
SELECT REGEXP_REPLACE('My phone number is 010-1234-5678', '[0-9]', '*') AS RESULT1,
       REGEXP_REPLACE('My phone number is 02(123)5678', 
                      '([[:digit:]]{2})([[:digit:]]{3})([[:digit:]]{4})',
                      '(\1) \2-\3') AS RESULT2
FROM DUAL;
```

**결과**

* RESULT1 → `My phone number is ***-****-****`
* RESULT2 → `My phone number is (02) 123-5678`

---

#### 📌 Level Up Test

```sql
SELECT REGEXP_REPLACE('AB12–CD34', '[A-Z]{2}', '**') AS result
FROM DUAL;
```

✅ 정답: `**12–**34`

---

### 3. `REGEXP_INSTR` 함수

문자열에서 정규표현식 패턴과 일치하는 부분의 **위치**를 반환한다.

```sql
SELECT REGEXP_INSTR('apple, banana, cherry', 'banana') AS R1,
       REGEXP_INSTR('My phone number is 123-456-7890', '[0-9]', 1, 2) AS R2,
       REGEXP_INSTR('123-456-7890 is my phone number', '[0-9]', 15, 2) AS R3,
       REGEXP_INSTR('apple, banana, cherry', 'banana', 1, 1, 1) AS R4,
       REGEXP_INSTR('Apple, banana, Cherry', 'cherry', 1, 1, 0, 'i') AS R5
FROM DUAL;
```

해석
```
R1 = 8
'banana'가 "apple, banana, cherry"에서 처음 나타나는 위치는 8번째 문자.

R2 = 18
'My phone number is 123-456-7890'에서 숫자([0-9])를 찾는데, 두 번째로 나오는 숫자의 위치.
첫 번째 숫자는 17번째(1), 두 번째 숫자는 18번째(2).

R3 = 0
'123-456-7890 is my phone number'에서 15번째 문자부터 검색 시작.
하지만 15번째 이후에는 숫자가 두 개 이상 나오지 않음 → 0 반환.

R4 = 14
'apple, banana, cherry'에서 'banana'를 찾되, 첫 번째 매치의 끝 위치를 반환하도록 설정.
'banana'는 8~13번째 문자 → 끝 위치는 14.

R5 = 16
'Apple, banana, Cherry'에서 'cherry'를 찾는데, 'i' 옵션(대소문자 무시).
"Cherry"는 16번째부터 시작 → 결과는 16.
```
---

#### 📌 Level Up Test

```sql
SELECT REGEXP_INSTR('abc123def456ghi', '[0-9]+', 1, 2) AS result
FROM DUAL;
```

✅ 정답: `10`
(두 번째 숫자 시퀀스 `456`의 시작 위치는 10번째)

---

### 4. `REGEXP_COUNT` 함수

문자열 내 정규표현식 패턴이 몇 번 나타나는지 횟수를 반환한다.

```sql
SELECT REGEXP_COUNT('There are 123 apples and 456 oranges', '[0-9]') AS R1,
       REGEXP_COUNT('There are 123 apples and 456 oranges', '[0-9]', 15) AS R2,
       REGEXP_COUNT('apple, banana, cherry, date', ',') AS R3,
       REGEXP_COUNT('Apple, apple, APPLE', 'apple', 1, 'i') AS R4,
       REGEXP_COUNT('Apple, apple, APPLE', 'apple', 1, 'c') AS R5
FROM DUAL;
```


---
해석
```
R1 = 6
'There are 123 apples and 456 oranges'에서 [0-9] 숫자 개수.
123 → 3개, 456 → 3개 → 총 6개.

R2 = 3
같은 문자열에서 15번째 위치부터 숫자 개수 세기.
"123 apples and 456 oranges"에서 15번째 이후엔 "456"만 있음 → 3개.

R3 = 3
'apple, banana, cherry, date'에서 ,의 개수. → 3개.

R4 = 3
'Apple, apple, APPLE'에서 'apple' 찾기, 'i' 옵션으로 대소문자 무시.
"Apple", "apple", "APPLE" → 총 3개.

R5 = 1
같은 문자열에서 'apple' 찾기, 'c' 옵션으로 대소문자 구분., c가 기본값
"apple" (소문자만) → 1개 → 1.
```

#### 📌 Level Up Test

```sql
SELECT REGEXP_COUNT('123123123123123', '123', 1) AS R1,
       REGEXP_COUNT('123123123123123', '123', 3) AS R2
FROM DUAL;
```

✅ 정답: `5, 4cr`

---

### 5. `REGEXP_LIKE` 조건

정규표현식을 사용해 문자열 패턴과 일치 여부를 확인하는 조건식.
데이터를 필터링할 때 사용되며, `LIKE` 연산자보다 강력한 패턴 매칭 가능.

```sql
SELECT FIRST_NAME, LAST_NAME
FROM HR.EMPLOYEES
WHERE REGEXP_LIKE(FIRST_NAME, '^Ste(v|ph)en$');
```
* 해석 
```
^ → 문자열의 시작

Ste → "Ste"로 시작해야 함

(v|ph) → "v" 또는 "ph" 중 하나

en → "en"으로 끝나야 함

$ → 문자열의 끝

즉, Steven 또는 Stephen 만 매칭됩니다.
```

>## 적중 예상 문제

