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
Comming soon...

## References
- https://learnsql.com/blog/what-is-cte/
- https://learnsql.com/blog/sql-subquery-cte-difference/