📝 **문제 풀이**: 

- 🔗 [solvesql - 복수 국적 메달 수상한 선수 찾기](https://solvesql.com/problems/multiple-medalist/)
```sql
WITH t AS (
  SELECT r.athlete_id
  FROM records r
  JOIN games g ON r.game_id == g.id
  WHERE (r.medal != "") & (g.year >= 2000)
  GROUP BY athlete_id
  HAVING COUNT(DISTINCT team_id) >= 2
)

SELECT name
FROM athletes
WHERE id IN t
ORDER BY name
```

- 🔗 [solvesql - 온라인 쇼핑몰의 월 별 매출액 집계](https://solvesql.com/problems/shoppingmall-monthly-summary/)
```sql
SELECT 
  strftime('%Y-%m', order_date) as order_month,
  SUM(CASE WHEN o.order_id NOT LIKE 'C%' THEN oi.price*oi.quantity ELSE 0 END) AS ordered_amount,
  SUM(CASE WHEN o.order_id LIKE 'C%' THEN oi.price*oi.quantity ELSE 0 END) AS canceled_amount,
  SUM(oi.price*oi.quantity) AS total_amount
FROM order_items oi
JOIN orders o ON oi.order_id = o.order_id
GROUP BY order_month
```

- 🔗 [solvesql - 세 명이 서로 친구인 관계 찾기](https://solvesql.com/problems/friend-group-of-3/)
```sql
SELECT
  e1.user_a_id AS user_a_id,
  e1.user_b_id AS user_b_id,
  e2.user_b_id AS user_c_id
FROM edges e1
JOIN edges e2 
  ON e1.user_a_id = e2.user_a_id AND e1.user_b_id < e2.user_b_id
JOIN edges e3 
  ON (
    (e3.user_a_id = e1.user_b_id AND e3.user_b_id = e2.user_b_id)
     OR
    (e3.user_b_id = e1.user_b_id AND e3.user_a_id = e2.user_b_id)
  )
WHERE e1.user_a_id < e1.user_b_id AND e1.user_a_id < e2.user_b_id
  AND (e1.user_a_id = 3820 OR e1.user_b_id = 3820 OR e2.user_b_id = 3820)
ORDER BY user_a_id, user_b_id, user_c_id;
```
