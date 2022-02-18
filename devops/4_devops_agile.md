# DevOps & Agile

## Khái niệm
- Agile Development: là một phương pháp tập trung vào việc delivery từng kết quả nhỏ một cách nhanh chóng, nhằm release một version mới mỗi tuần hay mỗi tháng. Mục đích cuối cùng là đem lại một trải nghiệm tối ưu cho end-users.

- DevOps: là những practices dựa trên việc kết hợp giữa software development và delivery process, giữa developer và operations. Lợi ích mà DevOps mang lại là đơn giản hóa quá trình phát triển và giảm thiểu thông tin sai lệch.

## Điểm khác nhau
- Đối tượng:
	- Agile tập trung vào việc giao tiếp giữa end-user và developer
	- DevOps thì nhắm đến team developer và team operation
	- Có thể hiểu là Agile tập trung hướng ra bên ngoài, còn DevOps tập trung vào những cái bên trong
- Team
	- Agile thì áp dụng cho developer và project manager
	- DevOps thì nằm ở điểm giao nhau của development, QA và operations, họ tham gia vào tất cả các giao đoạn trong chu trình phát triển sản phẩm, và là một phần của Agile team.
- Những framework được áp dụng
	- Agile thì có Scrum > Kanban > Lean > Extreme > Crystal > Dynamic > Feature-Driven.
	- DevOps thì có những practice như là Infrastructure as Code, Monitoring, Self Healing, End to End test automation
- Feedback
	- Đối với Agile thì feedback chủ yếu là từ end user
	- Còn với DevOps thì feedback đến từ stakeholders và team được ưu tiên hơn.
- Documentation
	- Agile ưu tiên sự linh hoạt và hiệu quả hơn là việc document hay monitor.
	- Còn với DevOps thì project document là một thành phần thiết yếu.
- Risks
	- Rủi ro của Agile đến từ việc linh hoạt trong các phương pháp, nên rất khó để dự đoán và đánh giá
	- Còn với DevOps thì rủi ro nằm ở việc thiếu hiểu biết về các khái niệm và các công cụ phù hợp.
- Tools
	- Agile: JIRA, Trello, Slack, ...
	- DevOps: Jenkins, Github Actions, ...
	
### Lợi ích của việc kết hợp Agile và DevOps
- Linh hoạt trong quản lí và tận dụng sức mạnh của công nghệ
- Các practice trong Agile giúp DevOps team giao tiếp hiệu quả hơn
- Cost cho việc automation là cần thiết để đáp ứng các yêu cầu về deploy nhanh và thường xuyên của Agile
- Và cuối cùng chính là có được một sản phẩm tốt

### Để đạt được hiệu quả thì có thể tham khảo 7 step sau
1. Kết hợp development và operations team
2. Hình thành những team build & run, tất cả những gì liên quan đến development hay operation thì đều được thảo luận với DevOps team.
3. Thay đổi cách tiếp cận với sprints, đánh dấu độ ưu tiên để yêu cầu các task cho DevOps, và các task này cũng có giá trị ngang với development task. Ủng hộ việc trao đổi ý kiến giữa development và operation team về workflow hay các vấn đề có thể xảy ra.
4. Có QA ở tất cả các giai đoạn phát triển
5. Chọn đúng tools cho đúng việc
6. Tự động mọi thứ có thể
7. Đánh giá và kiểm soát bằng các số liệu
