### Head of line (HOL) blocking

Đối với giao thức HTTP/1.1 thì `Head of line blocking` là để chỉ việc một client chỉ có một số lượng giới hạn TCP connections tới server (thường là 6 connections cho mỗi hostname) và khi muốn tạo 1 request mới trên một trong những connections đó thì nó phải đợi request trước đó trên cùng connection hoàn tất. Nên nếu một request cần quá nhiều thời gian để xử lý sẽ dẫn đến việc không thể tạo request mới, gây ra hiện tượng blocking ở client.

HTTP/1.1 giới thiệu một tính năng là `Pipelining` cho phép client gửi nhiều HTTP request trên cùng một TCP connection. Tuy nhiên HTTP/1.1 vẫn phải đợi những responses về theo đúng thứ tự nên nó cũng không giải quyết được HOL, và tính năng này cũng không được sử dụng rộng rãi.

HTTP/2(h2) giải quyết vấn đề HOL bằng multiplexing requests (ghép kênh requests) trên cùng một TCP connection, do đó client có thể tạo nhiều request tới server mà không cần phải đợi những request trước đó hoàn tất và responses có thể trả về theo bất cứ thứ tự nào.

Tuy nhiên HTTP/2 vẫn còn gặp một loại HOL khác vì nó chạy trên một TCP connection, và do TCP có cơ chế để tránh bị nghẽn, nên khi có một packet bị lost trong TCP stream, thì sẽ làm tất cả stream phải đợi đến khi package đó được re-transmitted và nhận được.

Và giải pháp cho việc này là chạy HTTP/2 trên UDP và tối ưu việc quản lí nghẽn (congestion), và đó là lí do giao thức `QUIC`ra đời, và nó chính là tương lai của HTTP.
