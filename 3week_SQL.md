## **📌 Week 3: CASE문 & 비교, 논리 연산자 활용**

---

### **1️⃣ 주요 개념**

- **CASE문 & 논리 연산자 활용**:
    - `CASE WHEN`
    - `IF()`, `IFNULL()`, `NULLIF()`
    - `COALESCE()`


14.5 Flow Control Functions

# CASE 문
(CASE END로 끝나는 CASE statement와는 다른 사용법)

✅ 형식
형식 1)
```sql
CASE value
  WHEN compare_value1 THEN result1
  WHEN compare_value2 THEN result2
  ELSE default_result
END
```

주어진 value를 compare_value들과 비교해서
처음으로 일치하는 경우에 해당하는 result를 반환
아무것도 일치하지 않으면 ELSE절 결과를 반환, ELSE가 없으면 NULL 반환

형식 2)
```sql
CASE
  WHEN condition1 THEN result1
  WHEN condition2 THEN result2
  ELSE default_result
END
```

처음으로 참이 되는 조건에 해당하는 result를 반환
아무것도 일치하지 않으면 ELSE절 결과를 반환, ELSE가 없으면 NULL 반환

✅ 결과 타입 결정 규칙
CASE의 결과는 하나의 열처럼 동작해야 하기 때문에, 최종 결과 타입을 하나로 강제 통합한다.
최종 결과 타입은 모든 결과값들의 타입을 종합해서 결정된다.

📌 1. 숫자 타입 (Numeric)
모든 타입이 숫자이면 결과도 숫자 타입

우선순위:
하나라도 DOUBLE 타입이면 → 결과는 DOUBLE
하나라도 DECIMAL 타입이면 → 결과는 DECIMAL
그렇지 않으면 → 결과는 정수 타입

정수 타입끼리일 때
모두 signed이거나 모두 unsigned이면 → 가장 큰 정수 타입
signed와 unsigned가 섞이면 → 결과는 signed BIGINT
단, unsigned BIGINT와 signed 정수가 섞이면 → 결과는 DECIMAL(정밀도 충분, 소수점 없음)

📌 2. 기타 타입
|타입 종류|	결과 타입|
|--|--|
|모두 BIT|	결과도 BIT|
|BIT + 다른 타입|	BIT를 BIGINT처럼 처리|
|모두 YEAR|	결과도 YEAR|
|YEAR + 다른 타입|	YEAR를 INT처럼 처리|
|모두 문자열 (CHAR, VARCHAR)|	결과는 VARCHAR (가장 긴 문자열 길이 기준)|
|문자열 + 바이너리 (BINARY)|	결과는 VARBINARY|
|SET, ENUM|	결과는 VARCHAR처럼 처리|
|모두 JSON|	결과는 JSON|
|모두 날짜/시간 (DATE, TIME, TIMESTAMP)|	각각 유지 (DATE, TIME, TIMESTAMP)|
|날짜/시간 타입이 섞이면|	결과는 DATETIME|
|모두 GEOMETRY|	결과는 GEOMETRY|
|하나라도 BLOB| 포함	결과는 BLOB|
|그 외 섞인 타입|	결과는 VARCHAR|

📌 3. NULL 리터럴은 무시됨
CASE 안에 NULL이 있어도 타입 결정에는 영향 없음

# IF 문

✅ 형식
IF(expr1, expr2, expr3)

expr1이 참(TRUE) 이면 → expr2를 반환
expr1이 거짓(FALSE) 이면 → expr3를 반환

참의 기준:
expr1이 0이 아니고(<> 0)
expr1이 NULL이 아니면 참으로 간주

✅ 주의할 점
**IF 함수(IF())**와 **IF 문(IF statement)**는 다르다.
→ 여기서는 IF 함수를 다루고 있음.

✅ 결과 타입 결정 규칙
IF 함수 결과도 CASE와 마찬가지로 일관된 하나의 타입이 되어야 한다.

|상황	|결과 타입|
|--|--|
|expr2나 expr3 중 하나가 NULL|	다른 하나의 타입이 결과 타입이 됨|
|둘 중 하나가 문자열|	결과 타입은 문자열|
|둘 다 문자열|	둘 중 하나라도 대소문자 구분이면 결과도 대소문자 구분|
|둘 중 하나가 실수(floating-point)|	결과는 실수 타입|
|둘 중 하나가 정수(integer)|	결과는 정수 타입|


# IFNULL 문

✅ 형식
IFNULL(expr1, expr2)
expr1이 NULL이 아니면 → expr1을 반환
expr1이 NULL이면 → expr2를 반환

※ 1/0처럼 에러가 나는 계산식도 IFNULL로 감싸면 NULL로 처리되어 안전하게 넘어갈 수 있다.

✅ 결과 타입 결정 규칙
expr1과 expr2 둘 중 더 일반적인 타입이 결과 타입이 된다.

일반성 순서:
1. 문자열 (STRING)
2. 실수형 (REAL)
3. 정수형 (INTEGER)


# NULLIF 문

✅ 형식
NULLIF(expr1, expr2)
expr1 = expr2 가 참(TRUE) 이면 → NULL을 반환
그렇지 않으면 → expr1을 그대로 반환

CASE WHEN expr1 = expr2 THEN NULL ELSE expr1 END 과 같은 의미다.

✅ 결과 타입 규칙
반환 타입은 항상 첫 번째 인자(expr1)의 타입을 따른다.

✅ 주의 사항
주의사항 1)
expr1이 함수 호출, 서브쿼리, 복잡한 계산인 경우,
expr1 = expr2를 비교할 때와, 다시 expr1을 결과로 반환하는 경우의 expr1 값이 달라질 수 있어
의도치 않은 결과가 나타날 수 있다.

주의사항 2)
expr1과 expr2의 문자셋이 서로 다른 경우, 
expr1의 문자들이 expr2의 문자셋/Collation 안에 있다면,
expr2의 문자셋/Collation을 사용해서 비교한다.

주의사항 3)
시스템 변수(system variable)와 같이 특별한 문자셋/Collation을 가진 값과 비교할 때,
문자셋 충돌 오류(Illegal mix of collations) 가 발생할 수 있다.
이 경우 CAST() 함수를 사용해서 명시적으로 문자셋과 Collation을 맞춰줘야 한다.



14.4.2 Comparison Functions and Operators
| 이름 | 설명 |
|------|------|
| > | 크다 연산자 |
| >= | 크거나 같다 연산자 |
| < | 작다 연산자 |
| <>, != | 같지 않다 연산자 |
| <= | 작거나 같다 연산자 |
| <=> | NULL-안전 동등(equal) 연산자 |
| = | 같다 연산자 |

| 이름 | 설명 |
|------|------|
| BETWEEN ... AND ... | 값이 특정 범위 안에 있는지 확인 |
| NOT BETWEEN ... AND ... | 값이 특정 범위 밖에 있는지 확인 |
|------|------|
| EXISTS() | 서브쿼리 결과에 행이 존재하는지 확인 |
| NOT EXISTS() | 서브쿼리 결과에 행이 없는지 확인 |
|------|------|
| GREATEST() | 가장 큰 값을 반환 |
| LEAST() | 가장 작은 값을 반환 |
|------|------|
| IN() | 값이 집합 안에 있는지 확인 |
| NOT IN() | 값이 집합 안에 없는지 확인 |
|------|------|
| IS | 값을 불리언(boolean)과 비교 |
| IS NOT | 값을 불리언(boolean)과 비교 (NOT) |
| IS NOT NULL | NULL이 아닌지 테스트 |
| IS NULL | NULL인지 테스트_표준 SQL |
| ISNULL() | 인자가 NULL인지 테스트_MYSQL 전용 함수 |
|------|------|
| LIKE | 간단한 패턴 매칭 |
| NOT LIKE | 패턴 매칭 부정 |
|------|------|
| COALESCE() | 첫 번째 NULL이 아닌 값을 반환 |
| INTERVAL(expr, val1, val2, ...) | 첫 번째 인자보다 작은 값의 인덱스를 반환 |
| STRCMP() | 두 문자열을 비교 |

IS
값이 TRUE, FALSE, UNKNOWN 인지를 판단

LIKE  
: 문자열의 부분 일치 여부를 %, _ 와 함께 확인할 때 사용.
ex)
SELECT 'apple' LIKE 'a%'; -- 결과: 1 (a로 시작하면 참)

INTERVAL(expr, val1, val2, ...)
: expr과 val1, val2, ... 를 순서대로 비교해서 expr보다 작은 값들의 개수를 반환한다.
비교는 정렬된 상태라고 가정하고 수행된다.

STRCMP(str1, str2)
두 문자열을 사전순(lexicographical order) 으로 비교한다.

두 문자열이 같음 → 0  
str1이 str2보다 작음 → < 0   (내부 구현에 따라 음수가 나옴)
str1이 str2보다 큼 → > 0   (내부 구현에 따라 양수가 나옴)


✅ 비교 연산(Comparison Operations) 정리
-비교 연산 결과는 항상 1 (참 TRUE), 0 (거짓 FALSE), 또는 NULL 중 하나가 된다.
-비교 연산은 숫자와 문자열 모두에 대해 동작한다.
-필요한 경우, 문자열은 숫자로, 숫자는 문자열로 자동 변환해서 비교가 수행된다.

-결과가 1/0/NULL이 아닌 함수도 있다
: LEAST(), GREATEST() 같은 함수는 가장 작은 값이나 가장 큰 값을 직접 반환한다.
이 함수들의 비교 규칙은 따로 정의돼 있다.


✅ 관계형 비교 연산자(Relational Comparison Operators)
=	>	<	>=	<=	<>  !=	

다음 비교 연산자는 스칼라 값뿐 아니라 행(Row) 값도 비교할 수 있다.
이런 연산자들은 행 서브쿼리(Row Subquery)에서도 사용할 수 있다.


✅ 타입 변환 관련 함수
-비교를 위해 명시적으로 타입을 변환하려면 CAST() 함수를 사용할 수 있다.
-문자열의 문자셋을 바꾸려면 CONVERT() 함수를 사용할 수 있다.

✅ 문자열 비교 기본 설정
기본적으로 문자열 비교는 대소문자 구분 없이(case-insensitive) 이루어진다.
기본 문자셋은 utf8mb4다.

---

### **1️⃣ 📝문제 풀이**

- 🔗 [HackerRank - 삼각형 종류 분류하기](https://www.hackerrank.com/challenges/what-type-of-triangle/problem) `CASE문`
```sql
SELECT CASE 
    WHEN A = B AND B = C THEN 'Equilateral'
    WHEN A + B <= C OR A + C <= B OR C + B <= A THEN 'Not A Triangle'
    WHEN A = B OR B = C OR C = A THEN 'Isosceles'
    ELSE 'Scalene'
END
 
FROM TRIANGLES
```

- 🔗 [LeetCode - find-customer-referee](https://leetcode.com/problems/find-customer-referee/description/) `IS NULL`
```sql
SELECT name
FROM Customer
WHERE referee_id != 2 or referee_id IS NULL
```
