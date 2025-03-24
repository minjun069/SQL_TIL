1. 주요 개념 정리 - 윈도우 함수 Window Functions

##14.20.2. Window Function Concepts and Syntax  
윈도우 함수: 한 행을 기준으로 그 주변의 다른 행들과 함께 계산하는 함수  
-모든 행을 유지하면서 기준 행과 주변 행들을 연산하는 작업을 할 수 있게 해주는 함수  
-일반적인 통합 연산은 하나의 행을 결과로 내는 반면, 윈도우 함수는 각 행에 대응하는 결과를 낸다.  
-현재 행(current row, 윈도우 함수가 수행되는 대상 행)에 관련된 행들(OVER로 지정된 행들)은 연산을 위한 윈도우로 구성된다.  
-OVER을 통해 윈도우 함수가 수행된다.  

OVER: 윈도우 함수에게 어떤 범위에서 어떻게 계산할지 알려주는 역할을 하는 부분

OVER절은 두 형태로 사용하여 윈도우 함수의 처리 과정을 정의할 수 있다.  
-OVER (window spec) : 윈도우가 OVER절 내에 직접 정의됨  
-OVER window_name : 이름 붙여진 윈도우가 쿼리의 다른 곳에서 정의되어 참조로 제공됨  
이 경우 WINDOW window_name AS (window spec) 형식으로 쿼리의 다른 곳에서 윈도우를 정의한다.  

OVER (window_spec) 형식  
OVER([window_name] [partition_clause] [order_clause] [frame_clause])_모두 옵션사항  
-window_name: 단독으로 사용될 경우, 그 이름으로 정의된 윈도우 대로 적용된다.  
다른 절과 같이 사용될 경우, 그 이름으로 정의된 윈도우가 뒤의 절(PARTITION BY 등)에 의해 수정되어 적용된다. 단, 이미 윈도우 정의 시에 적용된 기준이 있을 경우 에러가 발생한다.  
-partition_clause: 그룹화할 기준 정의, 생략될 경우 전체 쿼리가 그룹화된다.  
-order_clause: 연산 순서 지정  
-frame_clause: partition의 부분 집합인 frame을 정의한다.  

윈도우 함수는 SELECT와 ORDER BY 절에서만 사용할 수 있다.  
윈도우 함수의 실행 순서는 FROM, WHERE, GROUP BY, HAVING 다음, SELECT, DISTINCT, ORDER BY, LIMIT 이전이다.  
OVER 절은 많은 집계함수를 지원하고, MYSQL은 집계함수가 아닌 함수도 일부 지원한다.  

WINDOW window_name AS (window spec) 로 윈도우 정의  
-HAVING과 ORDER BY 절 사이에 위치하여 정의된다.  
-window_spec에서 다른 윈도우를 지정하여 사용할 수 있지만, 순환 형식으로는 사용할 수 없다.  
ex 허용되는 경우: WINDOW w1 AS (w2), w2 AS (), w3 AS (w1)  
ex 허용되지 않는 경우: WINDOW w1 AS (w2), w2 AS (w3), w3 AS (w1)

##14.20.1. Window Function Descriptions (nonaggregate window functions)  
-null_treatment: NULL 값을 처리하는 방식 지정. 옵션사항 (RESPECT NULLS, IGNORE NULLS)

1) CUME DIST() *over_clause*  
: 현재 행의 값보다 작거나 같은 파티션 값들의 누적분포를 반환한다.  
-윈도우 내 파티션의 순서 상 현재 행보다 작거나 같은 값을 갖는 행의 수를 총 행 수로 나눈 값을 나타낸다.  
-값의 범위는 0~1이다.  
-순서의 기준은 ORDER BY에서 사용된 열을 통해 정해진다. ORDER BY가 없을 경우, 값은 1로 통일된다.  
-사용법이 비슷한 함수: ROW_NUMBER(), PERCENT_RANK()  

2) DENSE RANK() *over_clause*  
: 파티션 내 현재 행의 순위를 반환한다.  
-같은 값일 경우 같은 순위로 취급한다.  
-같은 값을 같은 순위로 매긴 후, 다음 순위는 같은 순위의 개수에 따라 건너뛰지 않고 연속한 값의 순위를 반환한다.  
-순서의 기준은 ORDER BY에서 사용된 열을 통해 정해진다. ORDER BY가 없을 경우, 모든 행이 같은 것으로 취급된다.  
-사용법이 비슷한 함수: RANK() 같은 값을 같은 순위로 매긴 후, 다음 순위는 같은 순위의 개수에 따라 건너뛰고 순위를 반환

3) FIRST VALUE(expr) [null_treatment] *over_clause*  
: 윈도우 프레임의 첫번째 행에서 expr의 값을 반환한다.  
-사용법이 비슷한 함수: LAST_VALUE(expr), NTH_VALUE(expr, n)  

4) LAG(expr [, N [, default]]) [null_treatment] *over_clause*  
: 파티션 내에서 현재 행의 이전 행에서 expr 값을 반환한다.  
-N: 몇 행 이전 값을 가져올지 (디폴트 값은 1). 0일 경우 현재 행의 expr 값 반환.  
-default: 이전 행이 없다면 대신 넣을 값 (디폴트 값은 NULL)  
-사용법이 비슷한 함수: LEAD(expr [, N [, default]]) [null_treatment] *over_clause*

-이전 행과의 차이 또는 합을 계산하기 위해 자주 사용된다.  
ex)  
SELECT   
  val,   
  LAG(val) OVER w AS 'lag',  
  val - LAG(val) OVER w AS 'lag diff'  
FROM numbers  
WINDOW w AS (ORDER BY t);  

5) NTILE(N)  
: 파티션을 N개의 그룹으로 나눈후 각 행을 각 그룹에 배치하여 현재 행이 속한 그룹의 숫자를 반환한다.  
-ORDER BY와 함께 사용되어야 한다.


##14.19.1. Aggregate Function Descriptions  
-집계 함수는 주로 GROUP BY와 함께 사용된다.  
-GROUP BY를 사용하지 않을 경우, 모든 행에 대해 연산한다.  
-따로 명시되지 않은 한, 집계 함수는 NULL값을 무시한다.


-수치형 데이터에 대해 분산과 표준편차 함수는 DOUBLE 자료형을 반환한다.  
-수치형 데이터에 대해 SUM, AVG 함수는 DECIMAL 자료형을 반환한다.  
-SUM, AVG 함수는 날짜/시간 자료형 변수에 작동하지 않는다. 이를 해결하기 위해, 수치 단위로 변환한 후 사용한다.  
ex)  
SELECT SEC_TO_TIME(SUM(TIME_TO_SEC(time_col))) FROM tbl_name;  
SELECT FROM_DAYS(SUM(TO_DAYS(date_col))) FROM tbl_name;

집계함수 목록  
AVG([DISTINCT] expr)  
SUM([DISTINCT] expr)

STDDEV(expr) (= STD(expr), STDDEV POP()): 모집단 표준편차 반환  
STDDEV SAMP(): 표본 표준편차 반환  
VAR POP() (= VARIANCE()): 모집단 분산 반환  
VAR SAMP(): 표본 분산 반환

MAX([DISTINCT] expr)  
MIN([DISTINCT] expr)

COUNT(expr)  
COUTN(DISTINCT expr)

BIT AND(expr): 이진수의 각 자리 별로 AND 연산  
BIT OR(expr): 이진수의 각 자리별로 OR 연산  
BIT XOR(expr): 이진수의 각 자리별로 1의 개수가 홀수면 1, 짝수면 0 (배타적 논리합을 누적 적)

JSON ARRAYAGGG(col or expr): 각 행의 값들을 하나의 배열 [a, b, ...]로 합쳐 반환  
JSON OBJECTAGG(key_col, value_col): 두 열의 값들을 하나의 쌍으로 묶은 배열{key1: value1, key2: value2, ...}을 반환. 만약 key값이 중복된 경우, 마지막 key-value 쌍만을 남김.  
-단순 JSON OBJECTAGG() 사용 시의 key-value의 순서는 비결정적이다.  
-key-value 순서 정렬을 위해 OVER(ORDER BY key or value)를 사용할 수 있다.


2. 문제 풀이
- 🔗 [LeetCode - Rank Scores](https://leetcode.com/problems/rank-scores/description/) `DENSE_RANK()`

SELECT score, DENSE_RANK() OVER (ORDER BY score DESC) AS 'rank'
FROM scores

- 🔗 [Solvesql - 다음날도 서울숲의 미세먼지 농도는 나쁨 😢](https://solvesql.com/problems/bad-finedust-measure/) `LEAD()`

SELECT *
FROM (
  SELECT 
    measured_at AS today, 
    LEAD(measured_at, 1) OVER(ORDER BY measured_at) AS next_day, 
    pm10, 
    LEAD(pm10, 1) OVER(ORDER BY measured_at) AS next_pm10
  FROM measurements
) AS sub
WHERE next_pm10 > pm10

- 🔗 [programmers - 그룹별 조건에 맞는 식당 목록 출력하기](https://school.programmers.co.kr/learn/courses/30/lessons/131124) (도전!!)
WITH 
join_table AS (
    SELECT member_name, review_text, review_date
    FROM rest_review rr
    JOIN member_profile mp ON rr.member_id = mp.member_id
),
ranked AS (
  SELECT 
    member_name,
    DENSE_RANK() OVER (ORDER BY COUNT(*) DESC) AS rk
  FROM join_table
  GROUP BY member_name
)

SELECT 
    j.member_name, 
    j.review_text, 
    DATE_FORMAT(j.review_date, '%Y-%m-%d') AS review_date
FROM join_table j
    JOIN ranked r ON j.member_name = r.member_name
WHERE r.rk = 1
ORDER BY review_date, review_text
