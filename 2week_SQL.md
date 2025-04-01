## **📌 Week 2: 복합 JOIN & GROUP BY + HAVING**

---

### **1️⃣ 주요 개념**

- **복합 JOIN & GROUP BY + HAVING**:
    - `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `FULL OUTER JOIN`
    - `CROSS JOIN`, `SELF JOIN`
    - `GROUP BY`, `HAVING`
- 해당 문법의 개념과 사용 시 주의할 점들을 정리하여 깃허브에 정리해 주세요.

---

### **2️⃣ 과제 안내**

각 개념에 대해 공식 문서를 참고하여 정리하고, 해당 개념을 활용하여 문제를 해결하세요.

### **✅ 복합 JOIN 학습 및 문제 풀이**

📖 **공식 문서 참고**: 🔗 [MySQL 공식 문서 - 조인](https://dev.mysql.com/doc/refman/8.0/en/join.html)

📝 **문제 풀이**: 🔗 [programmers - 저자별 카테고리 별 매출액 집계하기](https://school.programmers.co.kr/learn/courses/30/lessons/144856)

### **✅ GROUP BY + HAVING 학습 및 문제 풀이**

📖 **공식 문서 참고**

- 🔗 [MySQL 공식 문서 - GROUP BY](https://dev.mysql.com/doc/refman/8.0/en/group-by-handling.html) ([`ONLY_FULL_GROUP_BY`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_only_full_group_by) mode 부분은 제외하셔도 됩니다.)
- 🔗 [MySQL 공식 문서 - HAVING](https://dev.mysql.com/doc/refman/8.0/en/select.html) (HAVING절에 관련된 부분만 학습해보세요~)

📝 **문제 풀이**: 🔗 [programmers - 언어별 개발자 분류하기](https://school.programmers.co.kr/learn/courses/30/lessons/276036) (도전)


1. 주요 개념

1.1 복합 JOIN  
15.2.13.2 JOIN Clause  
table reference: FROM절에서 사용할 수 있는 모든 테이블 관련 표현  
(단일 테이블 이름, JOIN 구문, 서브쿼리, 테이블 함수 또는 뷰)

JOIN 사용 시 고려사항  
-table reference : tbl_name AS alias_name  또는 tbl_name alias_name  
-table_subquery : FROM 절에 나오는 서브쿼리는 별칭을 반드시 포함해야 한다.  
-하나의 JOIN에서 참조 가능한 테이블의 개수는 61개 이다.


1.2 GROUP BY + HAVING

14.19.3 MySQL Handling of GROUP BY (mode 제외)

GROUP BY와 비집계(nonaggregate) 컬럼

표준 SQL 규칙  
-SQL-92 이전:  
SELECT, HAVING, ORDER BY에서 사용하는 비집계 컬럼은 반드시 GROUP BY에 포함되어야 함.  
-SQL:1999 이상:  
GROUP BY 컬럼에 함수적 종속(functional dependency) 되어 있다면 예외 허용 가능.  
예: custid가 기본키일 때 name은 종속 → SELECT 가능.

MySQL의 처리 방식  
: ONLY_FULL_GROUP_BY 모드가 켜져 있으면 (기본값), GROUP BY에 포함되지 않은 비집계 컬럼은 허용되지 않음.  
단, 아래 중 하나일 경우 예외 처리:  
-해당 컬럼이 함수적으로 종속되어 있을 경우  
-WHERE절 등으로 값이 하나로 고정된 경우  
-ANY_VALUE() 함수로 명시적으로 "아무거나 써도 됨"이라고 표시한 경우

함수적 종속 (Functional Dependency)  
: 기본키나 유니크 제약 조건이 있는 열에 대해 다른 열이 종속적이면 허용됨  
예: name이 기본키이면 address는 종속됨 → SELECT name, address, MAX(age) ... GROUP BY name 가능

ANY_VALUE() 함수  
: 집계되지 않은 열이 있어도 명시적으로 허용할 때 사용

GROUP BY 없이 집계만 있는 경우, 집계함수만 있는 쿼리에 비집계 컬럼을 SELECT에 쓰면 오류 발생

DISTINCT + ORDER BY 사용 시 제한  
: DISTINCT와 ORDER BY를 같이 쓸 때, ORDER BY 대상 컬럼이 SELECT에 없으면 오류 발생  
(중복 제거 후 정렬 기준을 결정할 수 없기 때문)

GROUP BY에서 허용되는 MySQL 확장  
-GROUP BY 절에 표현식 사용 가능 (ex. FLOOR(value/100))  
-별칭 사용 가능 (ex. GROUP BY id, val)


15.2.13 SELECT Statement (HAVING 절 관련)

GROUP BY의 역할  
: 데이터를 그룹별로 묶어서 집계 함수(COUNT, AVG, MAX, 등)를 적용할 수 있게 한다.  
-GROUP BY에 명시되지 않은 열도 SELECT 절에 포함시킬 수 있음(MySQL 확장 기능).  
-GROUP BY는 기본적으로 오름차순 정렬되며, 정렬 순서를 ORDER BY로 별도 지정 가능.   
-WITH ROLLUP을 붙이면 그룹 집계에 대한 합계(소계, 전체합)를 추가할 수 있음.

HAVING의 역할  
: 그룹핑된 결과에 조건을 적용할 때 사용.

WHERE vs HAVING  
-WHERE은 그룹핑 이전의 개별 행에 대한 조건,  
-HAVING은 그룹핑 이후의 집계값에 대한 조건을 걸 수 있음.  
-HAVING에서는 집계 함수(MAX, COUNT, 등)를 사용할 수 있음.  
-반면 WHERE에서는 집계 함수 사용 불가.

주의 사항  
-HAVING은 최적화가 거의 안 되며, 실행 순서상 거의 마지막에 적용됨 (LIMIT 뒤).  
-HAVING에서 사용하는 열 이름이 SELECT의 별칭(alias)과 겹치면 애매한 경우가 발생할 수 있으므로, 이름 충돌 주의.  
-HAVING에 들어갈 열은 집계 함수이거나, GROUP BY에 포함된 열이어야 SQL 표준에 부합함.

2. 문제 풀이

📝 **문제 풀이**: 🔗 [programmers - 저자별 카테고리 별 매출액 집계하기](https://school.programmers.co.kr/learn/courses/30/lessons/144856)
SELECT 
    A.AUTHOR_ID,
    A.AUTHOR_NAME,
    B.CATEGORY,
    SUM(BS.SALES * B.PRICE) AS TOTAL_SALES
FROM BOOK_SALES BS
JOIN BOOK B ON BS.BOOK_ID = B.BOOK_ID
JOIN AUTHOR A ON B.AUTHOR_ID = A.AUTHOR_ID
WHERE BS.SALES_DATE BETWEEN '2022-01-01' AND '2022-01-31'
GROUP BY A.AUTHOR_ID, A.AUTHOR_NAME, B.CATEGORY
ORDER BY A.AUTHOR_ID ASC, B.CATEGORY DESC;


📝 **문제 풀이**: 🔗 [programmers - 언어별 개발자 분류하기](https://school.programmers.co.kr/learn/courses/30/lessons/276036) (도전)

SELECT
  CASE
    WHEN fe.CODE IS NOT NULL AND py.CODE IS NOT NULL THEN 'A'
    WHEN cs.CODE IS NOT NULL THEN 'B'
    WHEN fe.CODE IS NOT NULL THEN 'C'
  END AS GRADE,
  D.ID,
  D.EMAIL
FROM DEVELOPERS D
-- Front End 스킬
LEFT JOIN SKILLCODES fe
  ON (D.SKILL_CODE & fe.CODE) > 0 AND fe.CATEGORY = 'Front End'
-- Python 스킬
LEFT JOIN SKILLCODES py
  ON (D.SKILL_CODE & py.CODE) > 0 AND py.NAME = 'Python'
-- C# 스킬
LEFT JOIN SKILLCODES cs
  ON (D.SKILL_CODE & cs.CODE) > 0 AND cs.NAME = 'C#'
WHERE
  (fe.CODE IS NOT NULL AND py.CODE IS NOT NULL) -- A
  OR cs.CODE IS NOT NULL                        -- B
  OR fe.CODE IS NOT NULL                        -- C
ORDER BY GRADE ASC, D.ID ASC;

