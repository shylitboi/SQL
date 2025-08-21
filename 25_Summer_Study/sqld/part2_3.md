# PART2-3 관리구문
: 데이터베이스 객체나 사용자, 권한 등을 생성/수정/삭제/관리하기 위한 SQL 구문을 말합니다.
>## DML : DDL에서 정의한대로 데이터 입력, 수정, 삭제 조회

### 1. INSERT
* INSERT 구문은 테이블에 새로운 행(Row)을 추가하는 SQL 문입니다.


기본 문법

```sql
INSERT INTO 테이블명 (컬럼1, 컬럼2, ..., 컬럼N)
VALUES (값1, 값2, ..., 값N);
```

* `테이블명` : 데이터를 넣을 테이블 이름
* `(컬럼1, 컬럼2, …)` : 값을 넣을 컬럼 지정 (생략 가능)
* `VALUES` : 실제로 삽입할 값



(1) 특정 컬럼에 값 삽입

```sql
INSERT INTO employees (employee_id, first_name, last_name, hire_date, job_id, salary)
VALUES (207, 'Steven', 'King', SYSDATE, 'IT_PROG', 6000);
```

👉 employees 테이블에 새로운 직원(207번 Steven King)을 추가

---

(2) 모든 컬럼에 값 삽입

컬럼명을 생략하면, **테이블의 모든 컬럼 순서대로** 값을 넣어야 함.

```sql
INSERT INTO employees
VALUES (208, 'Neena', 'Kochhar', 'NEENA', '515.123.4568', SYSDATE, 'AD_VP', 17000, NULL, NULL, 100, 90);
```
* 순서대로 빠짐없이 나열되어야 하며 순서가 뒤바뀌어 데이터 유형이 맞지 않거나 누락된 데이터가 있어 전체 컬럼 개수와 맞지 않으면 ```error```발생 
---

(3) 여러 행 삽입 (Oracle 11g+ 또는 표준 SQL)

```sql
INSERT ALL
    INTO employees (employee_id, first_name, last_name, job_id, salary)
    VALUES (209, 'Lex', 'De Haan', 'AD_VP', 17000)
    INTO employees (employee_id, first_name, last_name, job_id, salary)
    VALUES (210, 'Alexander', 'Hunold', 'IT_PROG', 9000)
SELECT * FROM dual;
```

---

(4) 주의사항

* ```PK```, `NOT NULL` 제약조건이 있는 컬럼에는 반드시 값을 넣어야 함.
* `UNIQUE` 제약조건이 걸린 컬럼에는 중복된 값 삽입 불가.
* `DEFAULT` 값이 설정된 컬럼은 값을 생략하면 자동으로 기본값이 들어감.

### 2. UPDATE


* `UPDATE` 구문은 **테이블의 기존 데이터 값을 수정**하는 SQL 문입니다.
* 새로운 행을 추가하는 게 아니라, 특정 조건에 맞는 행의 컬럼 값을 바꾸는 역할을 합니다.

- 기본 문법

```sql
UPDATE 테이블명
   SET 컬럼1 = 값1,
       컬럼2 = 값2,
       ...
 WHERE 조건;
```

* `SET` : 바꿀 컬럼과 그 값 지정
* `WHERE` : 어떤 행(Row)을 수정할지 지정
* `WHERE`절을 생략하면 → **테이블 전체 행이 수정됨** (주의!)

(1) 특정 직원의 급여 수정

```sql
UPDATE employees
   SET salary = 6000
 WHERE employee_id = 101;
```

👉 employee\_id = 101 인 직원의 급여를 6000으로 변경

---

(2) 여러 컬럼 동시에 수정

```sql
UPDATE employees
   SET salary = 7200,
       job_id = 'IT_PROG'
 WHERE employee_id = 102;
```

👉 employee\_id = 102 인 직원의 급여와 직무(job\_id)를 동시에 변경

---

(3) 조건 없는 UPDATE (전체 변경)

```sql
UPDATE employees
   SET department_id = 50;
```

👉 employees 테이블의 **모든 행의 부서번호를 50으로 변경**
⚠️ `WHERE` 절 생략 시 매우 주의해야 함!

---

(4) 서브쿼리 이용

```sql
UPDATE employees
   SET department_id = (
       SELECT department_id
       FROM departments
       WHERE department_name = 'IT'
   )
 WHERE job_id = 'IT_PROG';
```

👉 `job_id = 'IT_PROG'` 인 직원들의 부서를 "IT 부서의 ID"로 변경

---

(4) 주의사항

* `WHERE` 절 조건을 잘못 걸면 → 불필요하게 많은 데이터가 바뀔 수 있음
* 무조건 실행 전에 **SELECT로 조건 검증**하는 습관 필요
* `CHECK`, `NOT NULL`, `FOREIGN KEY` 제약조건을 위반하면 에러 발생

### 3. DELETE


* `DELETE` 구문은 **테이블의 기존 데이터 행(Row)을 삭제**하는 SQL 문입니다.
* `UPDATE`가 데이터를 수정하는 거라면, `DELETE`는 해당 데이터를 아예 없애는 역할을 합니다.

* 기본 문법

```sql
DELETE FROM 테이블명
 WHERE 조건;
```

* `WHERE` : 어떤 행을 삭제할지 지정
* `WHERE` 절을 생략하면 → 테이블의 **모든 데이터가 삭제됨** (⚠️ 주의!!)

1) 특정 행 삭제

```sql
DELETE FROM employees
 WHERE employee_id = 101;
```

👉 employee\_id = 101 인 직원의 데이터 삭제

---

(2) 조건에 맞는 여러 행 삭제

```sql
DELETE FROM employees
 WHERE department_id = 50;
```

👉 부서 번호가 50번인 모든 직원 삭제

---

(3) 조건 없는 삭제 (전체 삭제)

```sql
DELETE FROM employees;
```

👉 employees 테이블의 모든 행 삭제 (⚠️ 구조는 남음, `TRUNCATE`와 비교 필요)

---
(4) 주의사항

* **항상 WHERE 절 확인 필수** → 실수로 조건 없이 실행하면 테이블 전체 데이터 삭제됨.
* `DELETE`는 하나하나 행을 지우기 때문에, **대량 삭제 시 성능이 느릴 수 있음**.
* `DELETE`는 롤백(ROLLBACK) 가능 → 트랜잭션 제어와 함께 사용 권장.
* 전체 삭제가 목적이라면, `TRUNCATE`가 더 빠르고 효율적임 (단, ROLLBACK 불가).

---

(5) DELETE vs TRUNCATE 차이

| 구분     | DELETE       | TRUNCATE     |
| ------ | ------------ | ------------ |
| 조건 지정  | 가능 (`WHERE`) | 불가능 (전체 삭제만) |
| 롤백 가능  | ✅ 가능         | ❌ 불가능(별도의 로그 쌓지 않음)        |
| 속도     | 상대적으로 느림     | 매우 빠름        |
| 트리거 발생 | ✅ 발생         | ❌ 발생 안함      |
| 테이블 구조 | 유지           | 유지           |


Eg)
```sql
DELETE FROM SAMPLE1 A
WHERE NOT EXISTS (
    SELECT 1 
      FROM SAMPLE2 B 
     WHERE A.COL3 = B.CO1
)
```
```
🔎 해석
메인 DELETE 대상

SAMPLE1 테이블에서 행(Row)을 삭제하려고 함.

A는 SAMPLE1의 별칭(alias).

NOT EXISTS 서브쿼리

SAMPLE2 테이블(B)에서 조건을 검사.

조건: A.COL3 = B.CO1

즉, SAMPLE1의 COL3 값이 SAMPLE2의 CO1 컬럼에 존재하는지 확인.

NOT EXISTS 조건

만약 SAMPLE1.COL3 값이 SAMPLE2.CO1 안에 존재하지 않는 경우 → 그 행은 삭제됨.
```

* INSERT, UPDATE, DELETE는 명령어 날리고 별도의 커밋 명령어를 실행시켜야 데이터가 반영되며 롤백 가능.
    * MYSSQL의 경우 오토 커밋됨


### 4. MERGE


* `MERGE`는 \*\*INSERT + UPDATE (필요 시 DELETE까지)\*\*를 하나의 문장으로 처리할 수 있는 구문입니다.
* 흔히 **업서트(UPSERT)** 라고 부릅니다.

  * 조건에 맞는 데이터가 이미 있으면 → **UPDATE**
  * 조건에 맞는 데이터가 없으면 → **INSERT**

---

* 기본 문법

```sql
MERGE INTO 대상테이블 A
USING 참조테이블 B
   ON (조인 조건)
 WHEN MATCHED THEN
      UPDATE SET 컬럼 = 값
 WHEN NOT MATCHED THEN
      INSERT (컬럼1, 컬럼2, ...) 
      VALUES (값1, 값2, ...);
```



(1) 단순 UPSERT

```sql
MERGE INTO employees e
USING new_employees n
   ON (e.employee_id = n.employee_id)
 WHEN MATCHED THEN
      UPDATE SET e.salary = n.salary,
                 e.job_id = n.job_id
 WHEN NOT MATCHED THEN
      INSERT (employee_id, first_name, last_name, job_id, salary)
      VALUES (n.employee_id, n.first_name, n.last_name, n.job_id, n.salary);
```

👉 `employees` 테이블에 대해:

* `employee_id`가 존재하면 → 급여와 직무(job\_id) 갱신
* 존재하지 않으면 → 새 직원 삽입
---
(2) DELETE까지 포함한 MERGE

```sql
MERGE INTO target t
USING source s
   ON (t.id = s.id)
 WHEN MATCHED THEN
      UPDATE SET t.value = s.value
      DELETE WHERE s.value IS NULL
 WHEN NOT MATCHED THEN
      INSERT (id, value)
      VALUES (s.id, s.value);
```

👉 조건 충족 시:

* 값이 존재 → UPDATE
* 값이 NULL → DELETE
* 없던 값이면 → INSERT

(4) 장점

* `INSERT`, `UPDATE`, `DELETE`를 따로 실행할 필요 없이 **한 번에 동기화** 가능
* 대용량 데이터 동기화, ETL, 마스터 테이블 업데이트 등에 매우 효율적

(5) 주의사항

* `ON` 절의 조인 조건을 잘못 걸면 → 원치 않는 대량 삽입/수정 발생 가능
* 제약조건(UNIQUE, FK 등)을 고려해야 안전하게 동작
* 경우에 따라 ```WHEN MATCHED THEN``` 혹은 ```WHEN NOT MATHCED THEN``` 중 하나만 사용 가능

#### 📌 MERGE에서의 USING
좋은 질문이에요 👍
`MERGE` 구문에서의 **`USING`** 과 일반적인 **`JOIN ... USING`** 은 문맥은 비슷하지만, 쓰임새와 범위가 조금 다릅니다.

---

## 📌 1. 일반 JOIN에서의 `USING`

```sql
SELECT e.employee_id, d.department_name
  FROM employees e
  JOIN departments d
 USING (department_id);
```

* 여기서의 `USING` 은 **조인 기준 컬럼이 두 테이블에 동일한 이름으로 존재할 때** 쓰는 축약 문법입니다.
* `ON e.department_id = d.department_id` 와 동일한 의미.
* 즉, **JOIN의 컬럼 조건을 지정하는 문법적 편의**입니다.

---

## 📌 2. MERGE에서의 `USING`

```sql
MERGE INTO target t
USING source s
   ON (t.id = s.id)
 WHEN MATCHED THEN
      UPDATE SET ...
 WHEN NOT MATCHED THEN
      INSERT (...);
```

* `USING` 은 \*\*MERGE 시 기준이 되는 "소스 데이터셋"\*\*을 지정하는 절입니다.
* 이때 소스는:

  * 다른 테이블 (`source`)
  * 뷰
  * 서브쿼리 (`SELECT ... FROM ...`)
  * 임시 값(dual 이용)
    도 올 수 있습니다.

즉, `MERGE` 문에서는 `USING`이 \*\*조인하려는 상대 테이블(또는 데이터셋)\*\*을 지정하는 역할을 합니다.

---

## 📌 3. `ON` 절의 역할

* `ON` 절이 바로 **MERGE의 조인 조건**입니다.
* 두 테이블이 어떤 기준으로 매칭되는지 지정해야 합니다.
* 예시:

```sql
MERGE INTO sample1 a
USING sample2 b
   ON (a.col3 = b.col1 AND b.status = 'Y')
```

👉 `sample1.col3` 과 `sample2.col1`이 같고, 동시에 `sample2.status`가 `'Y'`인 경우 매칭


>## TCL
: 트랜잭션을 제어하는 명령어

### 1. 트랜잭션이란
: 죽어도 한세트로 묶일 수 박에 없는 **논리적인 업무 단위**
* 완전 성공 or 완전 실패
* 실패시 ROLLBACK해야 결품이 발생하는 사태 막음

특징
* 원자성: ALL OR NOTHING
* 일관성
* 고립성
* 지속성


### 2. COMMIT


* `COMMIT` 은 **현재까지의 트랜잭션에서 수행된 변경 사항(INSERT, UPDATE, DELETE 등)을 데이터베이스에 영구적으로 반영**하는 명령어입니다.COMMIT을 실행해야 최종족으로 데이터 파일에 기록되고 트랜잭션이 완료된다.
* COMMIT 하지 않으면 메모리까지만 반영이 되는데 메모리는 휘발성이기에 언제든 사라질 수 있고, 다른 사용자는 변경된 값을 조회할 수 없음
* 한 번 `COMMIT` 하면, 해당 변경은 **취소(ROLLBACK) 불가**.


특징

* `COMMIT` 이후에는 **트랜잭션 종료** → 새로운 트랜잭션이 시작됨.
* `DDL`(CREATE, ALTER, DROP 등)은 자동으로 COMMIT이 실행됨.
* 오라클(Oracle) 같은 DB는 `COMMIT` 전까지는 변경이 **undo 영역**에 저장돼 있어 취소 가능.

---
### 3. ROLLBACK
: INSERT, UPDATE, DELETE 후 변경된 내용을 취소하는 명령어. ```SAVEPOINT``` 이후의 작업만 취소할 수 있습니다.
* COMMIT 과 마찬가지로 UPDATE 한 뒤 오랜 시간동안 rollback 하지 않을 경우 LOCK에 걸려 다른 사용자가 변경할 수 없음

### 4. SAVEPOINT
: ROLLBACK 수행시 전체 작업을 되돌리지 않고 일부만 되돌릴 수 있게 하는 기능
* ROLLBACK 뒤에 특정 SAVEPOINT 지정해주면 그 지점까지만 데이터 복구된다


>## DDL 
: 데이터를 정의하는 명령어. 데이터 유형, 크기 등을 설정해줌.
* VAR()에서서는 공백을 문자로 안침 -> "jennie " = "jennie"
* VARCHAR 타입에서는 "jennie" != "jennie "

### 1. CREATE

* `CREATE` 구문은 데이터베이스 안에서 **새로운 객체(Object)를 생성**하는 명령어입니다.
* 생성 가능한 객체에는 **데이터베이스, 사용자, 테이블, 뷰, 인덱스, 시퀀스 등**이 있습니다.

---

>기본 문법 (테이블 생성)

```sql
CREATE TABLE 테이블명 (
    컬럼명 데이터타입 [제약조건], 
    컬럼명 데이터타입 [제약조건], 
    ...
);
```

### 📌 CREATE TABLE 제약조건 종류

#### 1. **NOT NULL**

* 해당 컬럼은 반드시 값이 있어야 함 → `NULL` 입력 불가

```sql
CREATE TABLE employees (
    employee_id NUMBER(5) NOT NULL,
    last_name   VARCHAR2(25) NOT NULL
);
```

👉 `employee_id`, `last_name` 에는 반드시 값이 들어가야 함

---

#### 2. **UNIQUE**

* 컬럼 값이 **중복될 수 없음** (NULL은 허용)

```sql
CREATE TABLE employees (
    email VARCHAR2(50) UNIQUE
);
```

👉 email 컬럼은 중복 불가, 단 `NULL` 값은 여러 개 가능

---

#### 3. **PRIMARY KEY**

* `NOT NULL + UNIQUE` 성격을 동시에 가짐
* 테이블에서 각 행(Row)을 **유일하게 식별**하는 키

```sql
CREATE TABLE employees (
    employee_id NUMBER(5) PRIMARY KEY,
    last_name   VARCHAR2(25) NOT NULL
);
```

👉 `employee_id` 는 반드시 값이 있고, 중복될 수 없음

---

#### 4. **FOREIGN KEY (참조 무결성)**

* 다른 테이블의 기본키(PK)나 유니크 키를 참조하는 키
* 부모 테이블에 없는 값은 입력 불가

```sql
CREATE TABLE employees (
    employee_id   NUMBER(5) PRIMARY KEY,
    department_id NUMBER(5),
    CONSTRAINT fk_dept FOREIGN KEY (department_id)
        REFERENCES departments(department_id)
);
```

👉 employees.department\_id 는 departments.department\_id 값을 반드시 참조해야 함
```
(1) CONSTRAINT fk_dept

제약조건에 이름(fk_dept) 을 부여하는 부분

이렇게 이름을 주면 → 나중에 ALTER TABLE ... DROP CONSTRAINT fk_dept; 같은 명령으로 쉽게 관리 가능

이름 규칙은 보통 fk_테이블명_컬럼명 또는 fk_컬럼명 형태로 지음

여기서는 fk_dept → "부서 외래키"라는 의미

(2) FOREIGN KEY (department_id)

이 테이블(employees)의 department_id 컬럼이 외래키(Foreign Key) 임을 지정

즉, 다른 테이블(부모 테이블)의 특정 컬럼을 참조해야 함

(3) REFERENCES departments(department_id)

외래키가 참조하는 부모 테이블과 컬럼을 지정

departments(department_id) : departments 테이블의 department_id 컬럼이 부모 키

즉, employees.department_id 값은 반드시 departments.department_id에 존재해야 함
```

#### 📌 무결성 관련 옵션 (ON DELETE / ON UPDATE)

외래키(FK)를 정의할 때 보통 이렇게 씁니다:

```sql
CONSTRAINT fk_dept FOREIGN KEY (department_id)
    REFERENCES departments(department_id)
    ON DELETE CASCADE
    ON UPDATE SET NULL;
```

즉, 부모 테이블(`departments`)에서 **행이 삭제되거나 업데이트될 때 자식 테이블(`employees`)에 어떤 영향을 줄지**를 정하는 옵션입니다.

1. **CASCADE**

* 부모 키가 변경/삭제되면 → 자식 테이블도 **같이 변경/삭제**됨

예시:

```sql
ON DELETE CASCADE
```

* 부모 테이블에서 `department_id = 10` 삭제 → 자식 테이블에서 `department_id = 10` 가진 직원도 모두 삭제됨

👉 **연쇄 삭제/수정**

---

2. **SET NULL**

* 부모 키가 변경/삭제되면 → 자식 테이블 FK 값을 **NULL**로 설정

예시:

```sql
ON DELETE SET NULL
```

* `departments` 에서 `department_id = 20` 삭제 → `employees` 의 해당 직원들의 `department_id` 컬럼이 `NULL` 처리됨

👉 자식 행은 남기되, 참조 무결성을 맞추기 위해 값을 NULL 처리

---

3. **SET DEFAULT**

* 부모 키가 변경/삭제되면 → 자식 테이블 FK 값을 **DEFAULT 값**으로 변경

예시:

```sql
ON DELETE SET DEFAULT
```

* 자식 테이블 `department_id` 컬럼에 `DEFAULT 99` 가 설정돼 있다면, 부모 삭제 시 자동으로 `99`로 채워짐

👉 NULL 대신 지정된 기본값으로 처리

---

4. **RESTRICT**

* 부모 키에 대해 삭제/수정을 **제한**
* 즉, 자식이 참조하고 있으면 부모 행을 삭제/수정할 수 없음

예시:

```sql
ON DELETE RESTRICT
```

* 부모 테이블 `department_id = 30` 이 employees에서 참조 중이면 삭제 불가

👉 보수적 옵션 (자식이 있으면 부모 변경 불가)

---

5. **NO ACTION**

* SQL 표준에서 **RESTRICT와 사실상 동일**
* 다만, 제약조건 검사 시점이 조금 다름

  * `NO ACTION` 은 트랜잭션 끝날 때 무결성 위반을 체크
  * `RESTRICT` 는 명령 실행 즉시 체크

👉 대부분의 DBMS에서는 `RESTRICT` 와 `NO ACTION` 을 같은 의미로 취급

---

#### 5. **CHECK**

* 입력되는 값이 지정한 조건을 만족해야 함

```sql
CREATE TABLE employees (
    salary NUMBER(8,2) CHECK (salary > 0),
    job_id VARCHAR2(10) CHECK (job_id IN ('IT_PROG','HR_REP','SA_MAN'))
);
```

👉 salary는 0보다 커야 하고, job\_id는 정해진 값만 가능

---

#### 6. **DEFAULT**

* 값을 넣지 않으면 자동으로 기본값 입력

```sql
CREATE TABLE employees (
    hire_date DATE DEFAULT SYSDATE,
    status    VARCHAR2(10) DEFAULT 'ACTIVE'
);
```

👉 hire\_date는 기본적으로 오늘 날짜, status는 "ACTIVE"



---

(1) 간단한 테이블 생성
규칙

1. 테이블 명은 고유해야한다.
2. 한 테이블 내 컬럼명 또한 고유
3. 컬럼명 뒤에 데이터 유형, 데이터 크기 명시
4. 컬럼은 콤마로 구분
5. 테이블명과 컬럼명은 숫자로 시작 될 수 없음
6. 마지막은 ; 세미콜론으로 끝난다
```sql
CREATE TABLE employees (
    employee_id   NUMBER(5)      PRIMARY KEY,
    first_name    VARCHAR2(20),
    last_name     VARCHAR2(25)   NOT NULL,
    hire_date     DATE           DEFAULT SYSDATE,
    salary        NUMBER(8,2)    CHECK (salary > 0),
    department_id NUMBER(4)
);
```

👉 의미

* `employee_id` → 기본 키 (Primary Key)
* `last_name` → 반드시 값이 있어야 함 (NOT NULL)
* `hire_date` → 기본값은 오늘 날짜
* `salary` → 0보다 커야 함

---

### 2. 뷰(View) 생성

```sql
CREATE VIEW emp_view AS
SELECT employee_id, first_name, salary
  FROM employees
 WHERE department_id = 50;
```

👉 `employees` 테이블의 일부 데이터를 볼 수 있는 가상 테이블 생성

### 3. ALTER


* `ALTER` 구문은 이미 생성된 **데이터베이스 객체(특히 테이블)의 구조를 변경**할 때 사용합니다.
* **컬럼 추가/수정/삭제, 제약조건 추가/삭제, 테이블명 변경 등** 다양한 구조 변경이 가능합니다.

* 기본 문법 

```sql
ALTER TABLE 테이블명 
    [ADD | MODIFY | DROP | RENAME] ... ;
```

(1) 컬럼 추가

```sql
ALTER TABLE employees
ADD email VARCHAR2(50);
```

👉 employees 테이블에 `email` 컬럼 추가
* 추가된 컬럼 위치는 항상 맨 끝, 별도 위치 지정 x
---

(2) 컬럼 수정 : MODIFY
* 데이터 유형, default 값, NOT NULL 제약 조건에 대해 변경 가능
* 컬럼에 저장된 데이터가 없는 경우에만 데이터 유형 변경 가능
* 크기를 줄일 땐 데이터가 줄이고자 하는 크기보다 작아야하며 크기 늘리는 것은 상관 없음

```sql
ALTER TABLE employees
MODIFY last_name VARCHAR2(40) NOT NULL;
```

👉 `last_name` 컬럼 길이를 40으로 변경, NOT NULL 제약 추가

---

(3) 컬럼 삭제

```sql
ALTER TABLE employees
DROP COLUMN email;
```

👉 email 컬럼 삭제
* 한번 삭제한 컬럼은 복구 불가
---

(4) 컬럼 이름 변경

```sql
ALTER TABLE employees
RENAME COLUMN last_name TO surname;
```

👉 `last_name` → `surname` 으로 이름 변경

---

(5) 테이블 이름 변경

```sql
ALTER TABLE employees
RENAME TO staff;
```

👉 테이블 이름을 employees → staff 로 변경

---

(6) 제약조건 추가

```sql
ALTER TABLE employees
ADD CONSTRAINT uq_emp_email UNIQUE (email);
```

👉 email 컬럼에 UNIQUE 제약조건 추가

---

### (7) 제약조건 삭제

```sql
ALTER TABLE employees
DROP CONSTRAINT uq_emp_email;
```

👉 `uq_emp_email` 제약조건 삭제

---

#### 특징

* `ALTER` 는 실행 즉시 **자동 COMMIT** 발생 (ROLLBACK 불가)
* 데이터가 이미 들어있는 테이블의 경우, **컬럼 타입 축소** 같은 변경은 제약이 많음
* 무분별한 ALTER 남용은 **성능 저하 및 무결성 위반** 가능

### 4. DROP TABLE 
* DROP TABLE 은 데이터베이스에서 테이블 자체를 완전히 삭제하는 명령어입니다.

* 테이블 구조 + 데이터 + 제약조건 + 인덱스 모두 삭제됩니다.

* 실행 시 자동 COMMIT 발생 → ROLLBACK 불가 ⚠️


기본 문법
```sql
DROP TABLE 테이블명 [CASCADE CONSTRAINTS] [PURGE];
```

옵션 설명
```CASCADE CONSTRAINTS```

* 다른 테이블에서 외래키(FK)로 참조 중인 경우, 기본적으로는 DROP 불가

* CASCADE CONSTRAINTS 옵션을 주면, 참조하는 제약조건까지 함께 삭제 가능

즉 나는 너와 관계를 끊어내고 사라지겠다


### 5. TRUNCATE TABLE 
: 테이블 모든 데이터 삭제, ROLLBACK 불가능하 DDL로 분류된다...!!!


>## DCL
: DCL은 사용자(User), 권한(Privilege), 롤(Role) 을 관리하는 SQL 구문입니다.
### 1. USER 관련 명령어
#### (1) 사용자 생성

: 내가 사는 집에 룸메가 온다면 맨 처음 할 일은 룸메를 정하고 비밀번호를 부여해주는 일이다

```sql
CREATE USER user_name IDENTIFIED BY password;
```

👉 새로운 사용자 생성

예시:

```sql
CREATE USER scott IDENTIFIED BY tiger;
```

---

#### (2) 사용자 수정

```sql
ALTER USER user_name IDENTIFIED BY new_password;
```

👉 비밀번호 변경

예시:

```sql
ALTER USER scott IDENTIFIED BY newtiger;
```

---

#### (3) 사용자 삭제

```sql
DROP USER user_name CASCADE;
```

👉 사용자를 삭제, `CASCADE` 옵션 시 해당 사용자가 소유한 모든 객체도 함께 삭제

예시:

```sql
DROP USER scott CASCADE;
```

---

### 2. 권한(Privilege) 관련 명령어
: 룸메를 선정하고 비번을 줬다면 그 다음에는 집을 이용할 수 있ㄴ느 권한을 부여해야한다. 권한을 부여하지 않는다면 룸메는 방에 들어가지 못할 것

#### (1) 시스템 권한 부여

```sql
GRANT 권한 TO 사용자;
```

예시:

```sql
GRANT CREATE SESSION TO scott;
GRANT CREATE TABLE TO scott;
```

👉 scott 계정에 로그인 권한과 테이블 생성 권한 부여

---

#### (2) 시스템 권한 회수

```sql
REVOKE 권한 FROM 사용자;
```

예시:

```sql
REVOKE CREATE TABLE FROM scott;
```

👉 scott 계정의 테이블 생성 권한 회수

---

###  3. 롤(Role) 관련 명령어

롤(Role) = 여러 권한을 묶어서 한번에 관리하는 권한 그룹
eg) CREATE_SESSION, CREATE_USER, CREATE_TABLE => CREAT_R

#### (1) 롤 생성

```sql
CREATE ROLE role_name;
```

예시:

```sql
CREATE ROLE manager_role;
```

---

#### (2) 롤에 권한 부여

```sql
GRANT 권한 TO 롤;
```

예시:

```sql
GRANT SELECT, UPDATE ON employees TO manager_role;
```

---

#### (3) 사용자에게 롤 부여

```sql
GRANT 롤 TO 사용자;
```

예시:

```sql
GRANT manager_role TO scott;
```

👉 scott 사용자는 manager\_role이 가진 모든 권한을 상속받음

---

#### (4) 롤 권한 회수

```sql
REVOKE 롤 FROM 사용자;
```

예시:

```sql
REVOKE manager_role FROM scott;
```

---

>## 적중 예상 문제

