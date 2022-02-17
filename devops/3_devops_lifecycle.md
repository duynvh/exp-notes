# DevOps Lifecycle - Application Focused

- Chúng ta sẽ đi qua phần high level của một ứng dụng từ lúc bắt đầu đến khi kết thúc và lặp lại.

1. Development
- Việc lên plan và coding là của developer, nhưng là một DevOps thì bạn có thể đọc code và chọn infrastructure tốt nhất cho ứng dụng.

2. Testing
- Bắt đầu bằng việc QA manually, sau đó DevOps có thể giả lập trên môi trường test để cải thiện hiệu quả.
- Quá trình này sẽ dần được tự động hóa.

3. Integration
- Đây là giai đoạn giữa của DevOps lifecycle, ở đây developer sẽ tạo ra nhiều commit thay đổi thường xuyên, có thể là theo ngày hoặc theo tuần.
- Với mỗi commit thì ứng dụng sẽ phải đi qua phần automated testing để sớm phát hiện issues hoặc bugs.

4. Deployment
- Deploy ứng dụng lên production servers.
- Giai đoạn này chúng ta sẽ cần đến `Application Configuration Management` và `Infrastructure as Code`.

5. Monitoring
- Để đảm bảo trải nghiệm của end users thì chúng ta cần phải theo dõi Application Performance liên tục.
- Việc này giúp cho developer có thể quyết định tốt hơn về việc cải thiện ứng dụng trong những bản release sau.
- Reliability là yếu tố quyết định, chúng ta muốn ứng dụng luôn available và hoạt động tốt.
- Và bên cạnh đó việc quan sát, bảo mật hay quản lí dữ liệu phải được theo dõi thường xuyên và feedback luôn có thể được dùng để cải thiện ứng dụng tốt hơn.
