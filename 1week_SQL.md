1. 주요 개념 정리
1) 서브쿼리
15.2.15 Subqueries
서브쿼리란 메인쿼리 내부에 괄호로 묶여 사용되는 쿼리를 말한다.
외부쿼리에 데이터를 제공하는 역할로 사용되기 때문에 select문만 허용된다.

-서브쿼리의 이점
각 구문을 구분하여 구조화할 수 있다.
복잡한 join, union 문 대신 사용하여 가독성을 높일 수 있다.


15.2.15.2 Coparisons Using Subqueries
서브쿼리를 사용하는 가장 일반적인 방법은 비교연산자와 함께 사용하는 것이다.
값과 비교할 때는 서브쿼리가 값을 반환하도록,
행 생성자(Row Constructor)와 비교할 때는 서브쿼리가 같은 개수의 값들을 갖는 행 생성자를 반환하도록 유의한다.


15.2.15.3 Subqueries with ANY, IN or SOME
IN: 특정 값이 서브쿼리 결과 목록에 포함되면 TRUE, 그렇지 않으면 FALSE 반환. 비교 연산자 없이 비교 대상과 함께 사용한다.
ex) SELECT s1 FROM t1 WHERE s1 IN (SELECT s1 FROM t2);

ANY: 서브쿼리 결과 중 하나라도 조건을 만족하면 TRUE, 그렇지 않으면 FALSE 반환. 반드시 비교 연산자와 함께 사용한다.
SOME: ANY와 동일.
ex) SELECT s1 FROM t1 WHERE s1 = ANY (SELECT s1 FROM t2);


15.2.15.4 Subqueries with ALL
ALL: 서브쿼리 결과의 모든 값이 조건을 만족하면 TRUE, 그렇지 않으면 FALSE 반환. 반드시 비교 연산자와 함께 사용한다.
ex) SELECT s1 FROM t1 WHERE s1 > ALL (SELECT s1 FROM t2);

-ALL 사용 시 서브쿼리의 결과 값이 비거나, NULL값을 갖는 경우를 주의해야 하며 이러한 경우를 edge case라 부른다.
-서브쿼리 결과 값이 빈 경우, ALL 연산은 TRUE를 반환한다.
-서브쿼리 결과 값이 NULL인 경우, ALL 연산은 UNKNOWN(NULL)을 반환한다.
-<> ALL 과 NOT IN은 같은 연산을 수행한다.


15.2.15.6. Subqueries with EXISTS or NOT EXISTS
EXISTS: 서브쿼리가 하나 이상의 행을 반환하면 TRUE, 그렇지 않으면 FALSE 반환. 비교 대상, 비교 연산자 없이 단독으로 사용한다.
NOT EXISTS: 서브쿼리가 아무 행도 반환하지 않으면 TRUE, 하나라도 반환하면 FALSE 반환. 비교 대상, 비교 연산자 없이 단독으로 사용한다.

-서브쿼리가 NULL 값을 가진 행을 반환하더라도 하나 이상의 행을 반환한 것으로 간주한다.


15.2.15.10. Subquery Errors
ERROR 1235: LIMIT과 IN/ALL/ANY/SOME을 동시에 포함하는 서브쿼리 사용 시 발생 (MySQL)
ERROR 1241: 여러 열을 반환하는 서브쿼리 사용 시 발생
-행 비교를 위한 경우, 여러 열을 반환하는 서브쿼리를 사용할 수 있지만, 그렇지 않은 경우, 서브쿼리는 단일 값을 반환해야 한다.
ERROR 1242: 비교 연산자만을 사용하여 연산할 경우, 여러 행을 반환하는 서브쿼리 사용 시 발생
ERROR 1093: 업데이트하려는 테이블을 서브쿼리 내에서도 사용 시 발생 (MySQL)



2) CTE(WITH)
15.2.20 WITH (Common Table Expressions)
CTE란 WITH절 하에 ,로 구분된 한 개 이상의 하위 절을 포함한 표현이다.
각 하위 절은 서브쿼리와 그 이름으로 구성된다.
ex)
WITH
  cte1 AS (SELECT a, b FROM table1),
  cte2 AS (SELECT c, d FROM table2)
SELECT b, d FROM cte1 JOIN cte2
WHERE cte1.a = cte2.c;


CTE의 열 이름 정의
- cte name 다음 괄호로 열 이름 지정
WITH cte (col1, col2) AS ...
- 서브쿼리 내에서 as로 열 이름 지정
WITH cte AS
( SELECT 1 AS col1, 2 AS col2 ... )


WITH 절이 허용되는 맥락
- SELECT, UPDATE, DELETE 구문의 시작 부분
- 서브쿼리의 시작 부분
- SELECT 구문을 포함하는 구문의 SELECT 앞 부분
- 동일 수준에 대하여 하나의 WITH구문만이 허용된다.


CTE간 인용
-스스로 인용하는 CTE는 recursive CTE로 정의한다.
-먼저 정의된 CTE를 이후에 정의되는 CTE가 인용할 수 있다.
(서로를 인용하는 mutually-recursive CTE는 예외적으로 사용가능하다.)
-외부 쿼리 단위에서 정의된 CTE를 보다 내부의 쿼리 단위에서 정의되는 CTE가 인용할 수 있다.

-같은 이름을 가진 객체를 참조할 때 (파생 테이블, CTE, 이외 객체) 순으로 탐색한다. 또한 현재 쿼리 단위에서 바깥 쿼리 단위로 확장하며 탐색한다.


파생 테이블(derived table) 과의 비교 
-두 표현 모두 이름을 붙이며, 한 구문 내에서만 존재한다.
-파생 테이블은 쿼리 내에서 한 번만 인용 가능하지만, CTE는 여러 번에 걸쳐 인용 가능하다.
-CTE는 재귀적 사용이 가능하다.
-CTE는 다른 CTE를 인용 가능하다.

CREATE TEMPORARY TABLE 과의 비교
-CTE는 정의, 삭제되는 것이 요구되지 않는다.
-CTE는 테이블 생성 권한이 요구되지 않는다.



2. 문제 풀이

- 많이 주문한 테이블
식사 금액이 테이블 당 평균 식사 금액보다 더 많은 경우를 모두 출력

SELECT * FROM tips
WHERE total_bill > (SELECT AVG(total_bill) FROM tips)


- 레스토랑의 대목
요일별 매출액 합계를 구하고, 매출이 1500 달러 이상인 요일의 결제 내역을 모두 출력

SELECT * FROM tips 
WHERE day IN (
    SELECT day FROM tips 
    GROUP BY day HAVING SUM(total_bill) > 1500
);


- 식품분류별 가장 비싼 식품의 정보 조회하기

서브쿼리
SELECT category, price, product_name 
FROM food_product f
WHERE price = (
    SELECT MAX(price) 
    FROM food_product
    WHERE category IN ('과자', '국', '김치', '식용유')
        AND category = f.category
    GROUP BY category 
)
ORDER BY price DESC


CTE
WITH cte AS (
SELECT category, MAX(price) as max_price FROM food_product
WHERE category IN ('과자', '국', '김치', '식용유')
GROUP BY category )

SELECT category, price, product_name FROM food_product
WHERE (category, price) IN (
    SELECT category, max_price FROM cte
)
ORDER BY price DESC;

서브쿼리는 대응하는 하나의 값을 반환해야하는 구조이기 때문에,
둘 이상의 열을 조건으로 사용하는 경우 CTE가 보다 직관적이고 편하다고 느꼈다.
