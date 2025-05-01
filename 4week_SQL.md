## **📌 Week 4: CTE, GROUP_CONCAT()**

---

### ** 주요 개념**

- **CTE, GROUP_CONCAT()**:
    - `WITH RECURSIVE`
    - `GROUP_CONCAT()`
 

### **✅ GROUP_CONCAT() 학습 및 문제 풀이**

GROUP_CONCAT()  
GROUP BY와 함께 사용되어, 그룹 내에서 NULL이 아닌 값들을 ' , ' 로 연결한 문자열을 반환한다.

```sql
GROUP_CONCAT([DISTINCT] expr [,expr ...]
             [ORDER BY {unsigned_integer | col_name | expr}
                 [ASC | DESC] [,col_name ...]]
             [SEPARATOR str_val])
```

-NULL이 아닌 값이 하나도 없으면 NULL을 반환한다.  
-DISTINCT 절을 사용해 중복 값을 제거할 수 있다.  
-SEPARATOR ' ' 을 통해 구분자를 ' , ' 외의 값으로 지정할 수 있다.  
-결과는 시스템 변수 group_concat_max_len에 의해 주어지는 최대 길이로 잘린다.  
기본값은 1024이며, 더 크게 설정할 수 있지만 반환값의 실질적인 최대 길이는 max_allowed_packet 값에 의해 제한된다.  
런타임에서 group_concat_max_len 값을 변경하는 문법은 다음과 같으며, val은 부호 없는 정수이다:
```sql
SET [GLOBAL | SESSION] group_concat_max_len = val;
```
-결과 타입은 group_concat_max_len이 512 이하일 경우 VARCHAR 또는 VARBINARY이며, 그 외에는 TEXT 또는 BLOB이다.


📝 **문제 풀이**:

- 🔗 [programmers - 우유와 요거트가 담긴 장바구니](https://school.programmers.co.kr/learn/courses/30/lessons/62284) **`GROUP_CONCAT()`**
    
    ```jsx
    조건:
    - WITH 구문을 사용해 가상의 테이블을 먼저 만들고,
    - GROUP_CONCAT() 을 활용하여 상품명을 하나의 문자열로 합친 후,
    - 그 문자열을 활용해 Milk와 Yogurt가 모두 존재하는 장바구니만 필터링해야 합니다.
    ```
```sql
WITH CartItems AS (
    SELECT 
        CART_ID, 
        GROUP_CONCAT(CONCAT('|',NAME,'|')) AS items
    FROM CART_PRODUCTS
    GROUP BY CART_ID
)

SELECT CART_ID
FROM CartItems
WHERE items LIKE '%|Milk|%' AND items LIKE '%|Yogurt|%';
```


- 🔗 [programmers - 언어별 개발자 분류하기](https://school.programmers.co.kr/learn/courses/30/lessons/276036) **(OPTIONAL)**는 2주차에 `HAVING`을 공부할 때 풀어본 문제입니다.
    
    해당 문제 상황을 참고하여, **각 개발자가 보유한 기술 목록과 기술 카테고리를 요약하여 출력하는 쿼리**를 작성해주세요.**`GROUP_CONCAT()`**
    
    출력 결과는 다음과 같아야 합니다.
    
    ![4-2.PNG](attachment:079e9ecb-3a6d-4dea-aea8-60981d09d411:4-2.png)

```sql
SELECT
    DEVELOPERS.ID,
    DEVELOPERS.EMAIL,
    GROUP_CONCAT(SKILLCODES.NAME) AS NAMES, 
    GROUP_CONCAT(DISTINCT SKILLCODES.CATEGORY) AS CATEGORIES
FROM SKILLCODES
JOIN DEVELOPERS
WHERE SKILLCODES.CODE & DEVELOPERS.SKILL_CODE > 0
GROUP BY DEVELOPERS.ID, DEVELOPERS.EMAIL
```



### **✅ CTE(`WITH RECURSIVE`) 학습 및 문제 풀이**

재귀 공통 테이블 표현식은 자신의 이름을 참조하는 하위 쿼리를 포함하는 표현식이다.

ex)
```sql
WITH RECURSIVE cte (n) AS (
  SELECT 1
  UNION ALL
  SELECT n + 1 FROM cte WHERE n < 5
)
SELECT * FROM cte;
```

WITH 절은 CTE 중 하나라도 자신을 참조하는 경우 반드시 WITH RECURSIVE로 시작해야 한다. 

재귀 CTE의 하위 쿼리 구조는 다음과 같다:

SELECT ... — 초기 행 집합을 반환
UNION ALL
SELECT ... — 추가 행 집합을 반환 (자기 자신을 참조하여 재귀)


첫 번째 SELECT는 초기 행을 반환하며 CTE 이름을 참조하지 않는다.
두 번째 SELECT는 CTE 이름을 FROM 절에서 참조하여 추가 행을 생성하며, 더 이상 새 행이 생성되지 않으면 재귀가 종료된다.

재귀 SELECT의 각 반복은 이전 반복에서 생성된 행만을 대상으로 동작한다.

CTE 결과 열의 데이터 타입은 비재귀 SELECT 부분의 열 타입에서 유추되며, 모든 열은 NULL 허용이다.
타입 결정 시 재귀 SELECT는 무시된다.

UNION DISTINCT를 사용할 경우 중복 행이 제거된다. 이는 무한 루프를 방지하는 데 유용하다.

재귀 SELECT가 비재귀 SELECT보다 넓은 열 값을 생성할 경우,
비재귀 SELECT에서 열 너비를 넓혀야 데이터 잘림(truncation) 을 방지할 수 있다.

ex)
열 넓이가 고정되어 결과가 모두 'abc'로 고정됨.
```sql
  SELECT 1 AS n, 'abc' AS str
  UNION ALL
  SELECT n + 1, CONCAT(str, str) FROM cte WHERE n < 3
```

문자열이 적절히 확장 가능
```sql
  SELECT 1 AS n, CAST('abc' AS CHAR(20)) AS str
  UNION ALL
  SELECT n + 1, CONCAT(str, str) FROM cte WHERE n < 3
```

재귀 SELECT 부분에는 다음과 같은 요소를 사용할 수 없다
: 집계 함수 / 윈도우 함수 / GROUP BY / ORDER BY / DISTINCT

재귀 SELECT는 CTE를 오직 한 번만, 오직 FROM 절에서만 참조해야 한다.
다른 테이블과 JOIN은 가능하지만, LEFT JOIN의 우측에 CTE가 위치할 수는 없다.


<최적화 및 성능 관련 사항>
재귀 SELECT는 EXPLAIN 실행 계획의 Extra 열에 Recursive로 표시된다.
EXPLAIN의 비용(cost) 정보는 반복 1회당 비용이며, 전체 반복 수는 예측할 수 없다.
결과 행 수가 많으면, CTE 결과를 저장하는 임시 테이블이 메모리에서 디스크로 이동할 수 있으며, 
이 경우 성능 저하가 발생할 수 있다.
이 경우 max_heap_table_size 또는 tmp_table_size 값을 조정하면 성능 향상에 도움이 될 수 있다.



📝 **문제 풀이**:

- 🔗 [programmers - 입양 시각 구하기(2)](https://school.programmers.co.kr/learn/courses/30/lessons/59413) **`WITH RECURSIVE`**

```sql
WITH RECURSIVE hours AS (
  SELECT 0 AS hour
  UNION ALL
  SELECT hour + 1 FROM hours WHERE hour < 23
)

SELECT 
  h.hour AS HOUR,
  COUNT(a.ANIMAL_ID) AS COUNT
FROM hours h
LEFT JOIN ANIMAL_OUTS a
  ON HOUR(a.DATETIME) = h.hour
GROUP BY h.hour
ORDER BY h.hour;
```




    

