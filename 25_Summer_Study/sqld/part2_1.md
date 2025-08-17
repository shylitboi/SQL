# PART 2 -1 SQL 기본
>## 관계형 데이터베이스 개요

1)  데이터베이스 : 용도와 목적에 맞는 데이터 저장
2) 관계형 데이터베이스 RDB : 데이터를 2차원 테이블 형태로 표현 후 각 테이블간 관계를 정의하는 것으로 시작  
    * RDBMS : RDB 관리하는 시스템, 오라클 ,mySQL 등
3) TABLE: RDB의 가본 단위(데이터 모델에서 엔터티에 해당)
4) SQL : RDB에서 데이터를 다루기 위한 언어

>## SELECT

1. select: 데이터 조회시 사용, '*' 사용시 전체 컬럼 조회
    * where절 없으면 전체 행 조회
    * 별칭 : AS 붙이거나 넣지 않아도 됨, 별칭 붙인 경우 테이블명 대신 alias 사용해야함
2. 산술 연산자 : number, date 유형 등에 사용 가능
    * null이 포함된다면 결과값은 Null

3. 합성 연산자 : 문자와 문자 연결시 사용  '||'

>## 함수

### 1) 문자 함수
* CHR(ASCII 코드) : 128개의 문자를 숫자로 표현할 수 있게 정의해 놓은 코드
    ```sql
    CHR(65) -> A
    ```
* LOWER/UPPER : 대/소문자로 변환
* **LTRIM(문자, [특정문자]), []는 옵션** : 특정문자를 따로 명시하지 않으면 왼쪽 공백을 제거, 명시한 경우 문자열 왼쪽부터 특정 문자와 비교하여  특정 문자에 포함되어 있으면 제거
    * RTRIM도 동일
    ```sql
    SELECT RTRIM(LTRIM(' SQ L D  ')) FROM DUAL
    ```
    결과는 'SQ L D'
    ```SQL
    SELECT RTRIM(LTRIM('SQL DEVELOPER', 'S')) FROM DUAL;
    ```
    결과는 'QL DEVELOPER'

    ```SQL
    LTRIM('DATABASE','DE')
    ```
    결과 : "ATABASE"

    ```sql
    RTRIM('SQL','LE')
    ```
    결과 : 'SQ'


* TRIM([위치] [특정 문자] [FROM] 문자열), []는 옵션 : 옵션이 없을 경우 문자열 왼쪽과 오른쪽 공백 제거
    * 옵션이 있는 경우 LEADING(앞) or TRAILING(뒤) or BOTH(앞뒤 동시) 로 지정된 곳 부터 한글자씩 특정 문자와 비교하여 같으면 제거
    * LTRIM, RTIM과 달리 특정문자를 **한글자**만 지정 가능
* SUBSTR(문자열, 시작점, [길이]) []는 옵션
    * 원하는 부분만 잘라서 반환, 길이를 명시하지 않은 경우 시작부터 끝까지 반환
    ```sql
    SUBSTR('블랙핑크제니', 3, 2) # 3번쨰(sql은 1부터 시작) 부터 2만큼
    ```
    결과 : 핑크
* LENGTH(문자열) : 길이 반환, 얘도 1부터 카운트
    ```sql
    length('jennie')
    ```
    결과 = 6

* REPLACE(문자열, 변경 전 문자열, [변경 후 문자열]) []는 옵션
    * 문자열에서 변경전 문자열을 찾아 변경 후 문자열로 바꿈
    * 변경 후 문자열을 주지 않으면 변경 전 문자열 삭제

* LPAD(문자열, 길이, 문자) : 문자열이 길이가 될 때 까지 왼쪽을 특정 문자로 채움
    ```sql
    lpad('JENNIE', 10, 'V')
    ```
    결과 : VVVVJENNIE

### 2) 숫자 함수

* ABS(숫자) : 절대값 반환
* SIGN(숫자) : 부호 반환
    * 양수 = 1, 음수 = -1, 0 = 0

* ROUND(수, [자릿수]) []는 옵션 : 수를 지정된 자릿수```까지``` 반올림
    * 자릿수 명시 하지 않은 경우 기본값은 0으로 반올림된 정수 반환
    * **자릿수가 음수일경우 지정된 정수부를 반올림하여 반환**
    ```sql
    round(163.76, -2) # 200
    ```
    * 0 자릿수는 1의 자리니까 -1은 10의 자리까지, -2는 100의자리까지...
* TRUNC(수, [자릿수]) []는 옵션 : 수를 지정된 자릿수 ```전 까지``` 버림
    ```sql
    trunc(54.29, 1) # 54.2
    trunc(54.29, -1) # 50
* CEIL(수) : 소숫점 이하 수를 올림해 정수 반환

* FLOOR(수) : 소수점 이하의 수를 버림하여 정수 반환


* MOD(수1, 수2) : 수1을 수2로 나눈 나머지 반환, 수2가 0일경우 수1 반환

### 3) 날짜 함수
* SYSDATE : 현재의 연, 월, 일, 시, 분, 초 반환
* EXTRACT(특정 단위 FROM 날자 데이터) : 날짜 데이터에서 특정 단위(YEAR, MONTH, DAY, HOUR, MINUTE, SECOND)만을 출력
    ```sql
    EXTRACT(YEAR FROM SYSDATE) # 2025
    ```

* ADD_MONTHS(날짜 데이터, 특정 개월수) : 날짜 데이터에서 특정 개월 수를 더한 날짜 반환
    * 날짜 이전의 달, 다음달에 기준 날짜의 일자가 존재하지 않으면 해당 월의 마지막 일자 반환
    ```sql
    add_months(DATE '2025-01-31', 1) 
    ```
        * 2/31일이 존재하지 않으므로 2/28일 반환 
* TO_DATE(문자열, 형식) : 날짜 데이터로 변환함
    형식 : 'YYYY-MM_DD'

### 4) 변환 함수
* 명시적 형변환과 암시적 형변환 
    * 명시적 : 변환 함수를 사용하여 데이터 유형 변환을 명시적으로 나타냄
    * 암시적 : 데이터베이스 알아서 변환함
    eg) 조건절에서 VARCHAR유형의 birthday 컬럼을 숫자와 비교할경우 오류가 안나고 내부적으로 알아서 number 유형으로 변환함
        * 하지만 되도록 명시적 형변환을 사용하는 것이 좋다

* TO_NUMBER(문자열) : 문자열 -> 숫자형
* TO_CHAR(수 or 날짜 [포맷]) []는 옵션 : 문자열로 변환
* TO_DATE(문자열, 포맷) : 문자열을 포맷 형식의 날짜형으로 변환
    * YYYY, MM, DD, HH(12시간 시), HH24(24시간 시), MI, SS

### 5) NULL 관련 함수

* NVL(인수1, 인수2) : 인수 1의 값이 null일 경우 인수 2 반환, null이 아닐 경우 인수 1 반환
    ```sql
    NVL(review_score, 0) 
    ```
    * review_score 컬럼의 값이 Null일 경우 0 반환, 아니면 속성값 반환

* NULLIF(인수1, 인수2) : 인수1과 인수2가 같으면 null, 아니면 인수1 반환
    ```sql
    nullif(review_score,0)
    ```
* COALESCE(인수1, 인수2, 인수3,,,) : null이 아닌 최초의 인수 반환

* NVL2(인수1, 인수2, 인수3) : 인수 1이 null이 아니면 인수2 반환, null이면 인수3반환

### 6) CASE - WHEN - END

`CASE WHEN` 구문은 **조건에 따라 다른 값을 반환할 수 있는 조건 분기 문**입니다.

#### 🧾 기본 문법

```sql
CASE
  WHEN 조건1 THEN 결과1
  WHEN 조건2 THEN 결과2
  ...
  ELSE 기본값
END
```

#### ✅ 특징

* 위에서부터 차례로 조건을 평가하고, **가장 먼저 참이 되는 조건의 결과**를 반환합니다.
* `ELSE`는 모든 조건이 거짓일 때 반환되는 기본값입니다 (필수는 아님).

#### 📝 예시

```sql
SELECT 
  NAME,
  SCORE,
  CASE
    WHEN SCORE >= 90 THEN 'A'
    WHEN SCORE >= 80 THEN 'B'
    ELSE 'F'
  END AS GRADE
FROM STUDENTS;
```

* case 구문에서는 else 뒤의 값이 디폴트고 별도의 else가 없을 경우 null이 디폴트

>## WHERE 

### 1) 비교 연산자, 부정 비교 연산자
* 비교 연산자 
    * 논리 연산자의 우선순위는 **() -> NOT -> AND -> OR**
    * 조건식에서 컬럼명은 우측에 위치할 수도 있음
* 부정 비교 : 같지 않다, 크지 않다

    * 같지 않다 : ```!=```, ```<>```, ```^=``` ```not 컬럼명=```
    * 크지 않다 : ```not 컬럼명 >```


* **SQL 연산자**

    1) BETWEEN A AND B : a와 b사이(a,b 포함)
    2) LIKE '문자열'
        ```sql
        WHERE 컬럼 LIKE '패턴'
        ```
        * `%`는 **임의의 길이 문자열**
        * `'서울%'`는 **‘서울’로 시작하는 문자열**을 의미

        ### 와일드 카드
        | 기호  | 설명                         |
        | --- | -------------------------- |
        | `%` | 0개 이상의 임의의 문자 (문자 수 제한 없음) |
        | `_` | 정확히 1개의 임의의 문자             |


        ```sql
        -- '서울'로 시작하는 주소 찾기
        WHERE ADDRESS LIKE '서울%'

        -- '동'으로 끝나는 식당 이름 찾기
        WHERE REST_NAME LIKE '%동'

        -- '김치'를 포함하는 리뷰 텍스트 찾기
        WHERE REVIEW_TEXT LIKE '%김치%'
        ```
        ### NOT LIKE
        ```SQL
        -- '서울'로 시작하지 않는 주소
        WHERE ADDRESS NOT LIKE '서울%'
        ```



        - `'통풍시트'`, `'열선시트'`, `'가죽시트'` 중 **하나 이상의 옵션이 포함된 자동차만 선택**
        - 자동차 종류(`CAR_TYPE`)별로 몇 대 있는지 집계
        - 컬럼명은 `CAR_TYPE`, `CARS`
        - 결과는 자동차 종류 기준으로 오름차순 정렬

        ---

        ## 오답

        ```sql
        SELECT CAR_TYPE, COUNT(*) AS CARS
        FROM CAR_RENTAL_COMPANY_CAR
        WHERE OPTIONS = "통풍시트" OR OPTIONS = "열선시트", OR OPTIONS =  "가죽시트"
        GROUP BY CAR_TYPE
        ````

        ### ❌ 오류 설명

        1. **문법 오류**:
        `"열선시트",` 부분에 \*\*잘못된 쉼표(,)\*\*가 포함되어 SQL 오류 발생

        2. **논리 오류**:
        `OPTIONS = "열선시트"`는 **옵션이 정확히 "열선시트" 하나만** 있어야만 참이 됨
        하지만 `OPTIONS`에는 `"가죽시트,열선시트,후방카메라"`처럼 여러 옵션이 들어 있으므로,
        **정확히 일치하는 경우 외에는 모두 필터링 실패**

        ---

        ## ✅ 해결 방법: `LIKE` 사용

        옵션 문자열에서 **특정 키워드가 포함됐는지**를 확인하려면 `LIKE`로 부분 문자열 검색을 해야 함.

        ```sql
        WHERE OPTIONS LIKE '%통풍시트%'
        OR OPTIONS LIKE '%열선시트%'
        OR OPTIONS LIKE '%가죽시트%'
        ```

        * `%`는 와일드카드: 키워드 앞뒤로 다른 문자가 있어도 일치됨
        * 예: `'가죽시트,열선시트'` → `%열선시트%`에 매칭됨
    3) IN (LIST)
        ```sql
        WHERE D.MCDP_CD IN ('CS', 'GS')
        ```
        * `MCDP_CD`가 `'CS'` 또는 `'GS'`인 행만 선택
        * `IN`을 사용할 경우 **복수의 값 조건**을 간결하게 처리할 수 있음
        * 다음과 같은 방식과 동일한 의미:
        ```sql
        WHERE D.MCDP_CD = 'CS' OR D.MCDP_CD = 'GS'
        ```
    4) IS NULL : null인 행을 조회

* **부정 SQL 연산자**
    1) NOT BETWEEN A AND B : A,B사이가 아니며 a,b 미포함
    2) NOT IN (LIST)
    3) IS NOT NULL
    * **NOT(A AND B) : AND에 NOT이 붙으면 OR 효과가 나옴**
        ```sql
        where not(sal < 300 and sal > 500)
        # where절 괄호를 풀면 ```sal >= 300 or sal <= 500```
    * 반대로 NOT(A OR B) 라면 AND가 나오겠지

* 논리 연산자  
    1) AND 모든 조건이 참이어야 함
    2) OR 하나 이상 조건이 참이어야 함
    3) NOT 참이면 거짓, 거짓이면 참
    eg) title이 supoort 이거나 city가 calgary가 아닌 행 조회 
    ```sql
    where not (title = 'support' or city = 'calgary')
    ```
    =
    ```sql
    where title <> 'support' and city <> 'calgary'
    ```

* 연산자 우선순위
    1) 산술 연산자
    2) 연결 연산자(||)
    3) 비교 연산자
    4) in, like, between, is null
    5) not
    6) and
    7) or


>## GROUP BY, HAVING

### 1) GROUP BY 
* group by + (그룹핑의 기준이 되는) 칼럼명

### 2) 집계 함수

* COUNT(*) :  전체 행 개수 반환
* COUNT(칼럼) : 컬럼값이 null인 행을 제외하고 count
* COUNT(DISTINCT 컬럼) : 컬럼값이 null인 행을 제외하고 중복을 제거한 count 
* SUM(컬럼) : 컬럼값이 null인 행을 제외하고 컬럼값 합계 반환
* AVG(컬럼) : 컬럼값이 null인 행을 제외하고 평균값 반환
* MIN(컬럼) : 최소값
* MAX(컬럼) : 최대값

### 3) having
