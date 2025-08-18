# PART 2 -2 SQL 활용
>## 서브쿼리

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
1) 연관 서브쿼리 : 메인쿼리의 컬럼 포함됨, 단독 실행 가능
    ```sql
    SELECT emp_name, salary
    FROM emp e
    WHERE salary > (
        SELECT AVG(salary)
        FROM emp
        WHERE dept_id = e.dept_id);
    ```
2) 비연관 서브쿼리  : 메인쿼리의 컬럼이 없음, 단독 실행 불가능
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

>## 뷰 

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
>## 집합 연산자
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

>## 그룹함수
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
