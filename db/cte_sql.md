# CTE in SQL

## CTE là gì
Common Table Express (CTE) là một temporary named result set được tạo từ một câu SQL để dùng lại cho những lần `SELECT`, `DELETE`, `INSERT` hoặc `UPDATE` sau.

Syntax của CTE:
```sql
WITH temp_name_1 AS (SELECT ...), 
	temp_name_2 AS (SELECT ...), ... 
<Query> 
```
- Kết quả của câu lệnh SELECT trong dấu `(...)` sẽ được lưu trữ là một CTE.
- Sau khi có được CTE thì có thể dùng nó trong câu query tiếp theo bằng cách reference theo tên của CTE.
- Vòng đời của CTE sẽ đi theo main query, nếu main query kết thúc thì CTE cũng sẽ biến mất.
- CTE temp_name_2 có thể sử dụng CTE temp_name_1 trong câu SELECT của nó.

Ví dụ:
```sql
WITH average_salary AS (
	SELECT role, avg(salary) as avg_salary
	FROM offers
	GROUP BY role
)
SELECT * FROM average_salary
```

Nếu không xài CTE thì chúng ta có thể dùng sub-queries để phục vụ mục đích tương tự. Nhưng CTE vẫn được prefer hơn vì:
- Giúp cho query dễ đọc và dễ hiểu hơn.
- CTE có thể reuse
- CTE có hỗ trợ recursive

Có một vài trường hợp phải sử dụng subquery như việc sử dụng ở trong `WHERE`, có thể xem thêm chi tiết ở đây: https://learnsql.com/blog/sql-subquery-cte-difference/

Lợi ích của việc sử dụng CTE:
- Tránh sử dụng sub-queries
- Tránh tạo các table hoặc view không cần thiết
- Làm cho code dễ đọc và dễ maintain hơn

## Recursive CTE

Recursive CTE là một dạng đặc biệt của nhiều CTE lồng nhau, nó tự reference đến chính nó trong một expression.

Cú pháp của recursive CTE cũng tương tự nhưng nó sẽ sử dụng những operator đặc biệt để có thể tự reference.
```sql
WITH RECURSIVE cte_name AS (
    cte_query_definition (the anchor member)
 
    UNION ALL
 
    cte_query_definition (the recursive member)
)
 
SELECT *
FROM   cte_name;
```

R-CTE sẽ chạy trên một dữ liệu dạng hierarchy, và nó kết thúc khi duyệt hết trên các levels.

Để sử dụng R-CTE thì chúng ta sẽ cần xác định anchor, parent, child. Một ví dụ về việc dùng R-CTE để lấy danh sách employee kèm level tương ứng, và biết được nhân viên đó thuộc quyền quản lí của ai.
```sql
WITH RECURSIVE company_hierarchy AS (
  SELECT    id,
            first_name,
            last_name,
            boss_id,
        0 AS hierarchy_level
  FROM employees
  WHERE boss_id IS NULL
 
  UNION ALL
   
  SELECT    e.id,
            e.first_name,
            e.last_name,
            e.boss_id,
        hierarchy_level + 1
  FROM employees e, company_hierarchy ch
  WHERE e.boss_id = ch.id
)
 
SELECT   ch.first_name AS employee_first_name,
       ch.last_name AS employee_last_name,
       e.first_name AS boss_first_name,
       e.last_name AS boss_last_name,
       hierarchy_level
FROM company_hierarchy ch
LEFT JOIN employees e
ON ch.boss_id = e.id
ORDER BY ch.hierarchy_level, ch.boss_id;
```

## References
- https://learnsql.com/blog/what-is-cte/
- https://learnsql.com/blog/sql-recursive-cte/
- https://learnsql.com/blog/sql-subquery-cte-difference/