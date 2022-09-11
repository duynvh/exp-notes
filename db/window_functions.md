# Window functions

Định nghĩa:
https://www.sqlite.org/windowfunctions.html

Là những SQL function mà input values là một hoặc nhiều rows trong results set của lệnh SELECT
Cách để phân biệt `wndow function` với những SQL function khác chính là `OVER` clause, nếu một function có `OVER` clause thì đó là window function, còn không thì là aggregate hoặc là scalar function.

Window function không thể đi với `DISTINCT` và chỉ được dùng trong result set và `ORDER BY` trong lệnh SELECT.

1. **LAG()**
Cho phép bạn lấy giá trị của một số row trước hoặc sau row hiện tại
Ví dụ: lấy ra danh sách tiền chi tiêu mỗi tháng, và chênh lệch so với tháng trước
Cú pháp: 
```sql
LAG(<expression>[,offset[, default_value]]) OVER ( 
	PARTITION BY expr,... ORDER BY expr [ASC|DESC],... 
)
```
với 
- `expression` là giá trị sẽ trả về lấy từ row được tính từ `offset` trong partittion hoặc result set
- `offset` là số lượng row trở về trước so với row hiện tại

2. **NTILE()**
Dùng để chia data thành n `buckets` theo partition hoặc result set
Ví dụ: có một danh sách điểm thi của lớp học có 40, thì dùng NTILE(10) để chia điểm thi thành 10 nhóm dựa trên điểm thi từ thấp đến cao, thì mỗi nhóm sẽ có 4 học sinh (40 / 10).
Cú pháp
```sql
NTILE(expression) OVER ( 
	PARTITION BY expression1, expression2,...
	ORDER BY expression1 [ASC | DESC]expression2,
)
```
với 
- `expression` là 1 số dương để chỉ số bucket cần chia