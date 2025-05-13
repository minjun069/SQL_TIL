📌 Week 5: 정규표현식&비트연산자

✅ 정규표현식 학습 및 문제 풀이
- **정규 표현식 (REGEXP)**
    - `REGEXP_LIKE()`
    - `REGEXP_REPLACE()`
    - `REGEXP_SUBSTR()`
    - `pattern syntax`

| 이름                | 설명                             |
| ----------------- | ------------------------------ |
| REGEXP        | 문자열이 정규 표현식과 일치하는지 검사 |
| NOT REGEXP    | REGEXP의 부정                        | 
| REGEXP\_INSTR()   | 정규 표현식과 일치하는 부분 문자열의 시작 인덱스 반환 |
| REGEXP\_LIKE()    | 문자열이 정규 표현식과 일치하는지 검사          |
| REGEXP\_REPLACE() | 정규 표현식과 일치하는 부분을 치환            |
| REGEXP\_SUBSTR()  | 정규 표현식과 일치하는 부분 문자열 반환         |
| RLIKE             | 문자열이 정규 표현식과 일치하는지 검사          |


🔗 programmers - 서울에 위치한 식당 목록 출력하기 
```sql
SELECT 
    RI.REST_ID, 
    RI.REST_NAME, 
    RI.FOOD_TYPE, 
    RI.FAVORITES, 
    RI.ADDRESS,
    ROUND(AVG(RV.REVIEW_SCORE), 2) AS SCORE
FROM 
    REST_INFO RI
JOIN 
    REST_REVIEW RV ON RI.REST_ID = RV.REST_ID
WHERE 
    RI.ADDRESS REGEXP '^서울'
GROUP BY 
    RI.REST_ID, 
    RI.REST_NAME, 
    RI.FOOD_TYPE, 
    RI.FAVORITES, 
    RI.ADDRESS
ORDER BY 
    SCORE DESC, 
    RI.FAVORITES DESC;
```

✅ 비트연산자 학습 및 문제 풀이
- **비트 연산자**: `&`, `|`, `^`, `<<`, `>>`

| 이름            | 설명            |          |
| ------------- | ------------- | -------- |
| `&`           | 비트 AND 연산     |          |
| `>>`          | 오른쪽 시프트       |          |
| `<<`          | 왼쪽 시프트        |          |
| `^`           | 비트 XOR 연산     |          |
| `BIT_COUNT()` | 1로 설정된 비트의 개수 |          |
| \`            | \`            | 비트 OR 연산 |
| `~`           | 비트 반전 (NOT)   |          |

🔗 programmers - 부모의 형질을 모두 가지는 대장균 찾기 
```sql
SELECT
    E.ID,
    E.GENOTYPE,
    P.GENOTYPE AS PARENT_GENOTYPE
FROM
    ECOLI_DATA E
JOIN
    ECOLI_DATA P ON E.PARENT_ID = P.ID
WHERE
    (E.GENOTYPE & P.GENOTYPE) = P.GENOTYPE
ORDER BY
    E.ID ASC;
```
