# 7 database concepts

**Ref**: https://betterprogramming.pub/7-database-concepts-you-should-know-about-a6825f46e449

Những concepts này chủ yếu là ở Relational database.

1. Database lưu trữ các transactions - không phải state
- DB sẽ lưu các action dưới dạng `Write Ahead Log (WAL)` và sau khi action được hoàn thành thì log sẽ bị xóa.

- WAL được dùng để khôi phục lại trạng thái của DB khi bi corrupted, dùng để đồng bộ dữ liệu trong replica.

2. Lựa chọn đúng DB rất khó
- Không có DB nào tốt nhất mà chỉ có DB tốt nhất cho ứng dụng của bạn.

- Để lựa chọn 1 DB phù hợp thì có thể cân nhắc những yếu tố sau:
	- Loại data nào sẽ được lưu vào DB? Lưu log hay lưu thông tin user
	- Data được lưu có phức tạp không? Có thể normalize nó dễ không?
	- Kiểu (uniform) data là như thế nào? Những data này thường có chung một schema hay khác nhau hay là lồng nhau nhiều.
	- Thường sẽ read hay write data hơn? Ứng dụng của bạn là read, hay write heavy hay cả hai.
	- Có yêu cầu gì từ business hay môi trường không?
	
3. Chuyển sang đúng DB còn khó hơn nữa
- Không chỉ là đưa dữ liệu từ nơi này sang nơi khác mà nó còn ảnh hưởng tới việc code của bạn có được thiết kế tốt không có có tightly coupled với database hay không.

4. NoSQL không thay thế được SQL, mà chúng bổ trợ cho nhau
- Có những thứ mà NoSQl sẽ làm tốt, và có những thứ mà SQL sẽ làm tốt hơn. 

- Ví dụ như việc lưu trữ time-series data, chúng ta sẽ không muốn dùng MySQL cho việc này dù nó hoàn toàn làm được. Cũng như việc có nên dùng Redis để lưu trữ những dữ liệu có quan hệ phức tạp không, dĩ nhiên là chúng ta vẫn có thể custom để làm được việc đó tuy nhiên nó làm cho mọi thứ phức tạp hơn rất nhiều.

- Và có những yếu tố khác như ngôn ngữ có hỗ trợ không, các tool vận hành, cloud support, ... cũng sẽ ảnh hưởng tới việc chọn DB.

5. Scaling không hề đơn giản
- Để scale thì DB vẫn phải đảm bảo được tính nhất quán, an toàn và tốc độ vẫn phải đảm bảo, nên việc này không hề dễ dàng. Vậy nên việc scale DB thường là `vertical scaling`.

- Một vài DB thì thường hay dùng cách `horizontal scaling`, thì trade-off ở đây chính là tính nhất quán, và chúng ta phải chấp nhận `eventually consistent`.

6. Indexes như một phép màu cho đến khi nó không còn như thế
- Indexes có thể tối ưu việc read data nhưng nó sẽ dẫn đến trade-off cho việc write data. Khi có indexes thì việc update hay create phải đi kèm với việc update lại index, sẽ dẫn đến phải tốn thêm thời gian cho việc xử lí write transacitons.
- Index chỉ là một bảng copy data của column và được sắp xếp lại theo thứ tự.

- Có nghĩa là index sẽ làm tăng disk space và require I/O khi nó cần update.

7. Transactions
- Mỗi khi có một tương tác vs DB thì khi đó nó sẽ tạo ra một transaction. 
- Trong thế giới của DB, thì transaction là một đơn vị atomic work - một action độc lập, rời rạc để thực hiện, và nhận về 2 kết quả là: commit hoặc rollback.

- Một ví dụ là khi `SELECT *` trên một table được dùng bởi tất cả mọi người, thì mỗi khi chạy database sẽ lock cả table trong suốt khoảng thời gian đó để đợi DB gửi kết quả về cho lệnh SELECT này. (`SELECT` cũng là một transaction)
