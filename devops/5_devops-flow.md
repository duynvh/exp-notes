# Plan > Code > Build > Testing > Release > Deploy > Operate > Monitor >

### 1.Plan
- Lên kế hoạch để làm tính năng mới hay fix bugs trong sprint tiếp theo.
- DevOps có thể tham gia và học được những thứ đã xảy ra trong quá trình phát triển. Có thể hỗ trợ team Dev trong những quyết định của họ với infrastructure hiện tại team DevOps đã build hoặc hướng team theo một con đường tốt hơn, xem team developer như là khách hàng.

### 2.Code
- Khi xong quá trình Plan, thì sẽ bắt đầu viết code.
- Trong quá trình này thì DevOps có thể giúp team Dev hiểu hơn về infrastructure, biết đang có những service nào và cách tốt nhất để giao tiếp giữa các services.
- Và cuối cùng thì merge code vào repository.

### 3.Build
- Đây là giai đoạn đầu tiên DevOps sẽ bắt đầu những quy trình tự động.
- Ở đây code sẽ được lấy và build phụ thuộc vào ngôn ngữ biên dịch hay phiên dịch, hay là một image được build từ code sử dụng CI/CD pipeline.

### 4.Testing
- Để đảm bảo những cập nhật mới không gây ra lỗi mới hoặc không break những thứ đã hoạt động.
- Những phần test này được team development viết và DevOps sẽ chạy những test này để hạn chế tối đa lỗi ở production

### 5.Release
- Một khi các test pass hết thì chúng ta sẽ tiến hành release, điều này phụ thuộc kiểu ứng dụng mà bạn đang phát triển.
- Quá trình release có thể đơn giản là đẩy code lên Github hay là biên dịch code hoặc là build một docker image rồi push lên registry, nơi mà có thể được truy cập bởi những server production.

### 6.Deploy
- Đây là giai đoạn đưa code lên production

### 7.Operate
- Đây là giai đoạn sản phẩm sẽ được sử dụng để phục vụ cho business của bạn.
- DevOps có thể sẽ phải chuẩn bị cho những trường hợp cần auto scaling khi lượng truy cập tăng, ...
- Team DevOps sẽ liên tục nhận feedback để biết những gì đang xảy ra trên production.

### 8.Monitor
- Tất cả những phần trên sẽ cần đến monitoring, đặc biệt là các vấn đề liên quan đến auto-scaling.
- Bạn sẽ không thể biết được chuyện gì xảy ra nếu không theo dõi những thông số cơ bản như memory, disk, CPU, api endpoint, response time, ...
- Và đặc biệt là logs, đây là thứ duy nhất giúp cho developer có thể theo dõi hệ thống mà không cần truy cập vào production systems.