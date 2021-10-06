### Select N + 1 Problem

Ref: 
- https://www.appdynamics.com/blog/product/common-application-problems-and-how-to-fix-them-the-select-n-1-problem/
- https://medium.com/doctolib/understanding-and-fixing-n-1-query-30623109fe89

Tham khảo:
- https://www.appdynamics.com/blog/product/common-application-problems-and-how-to-fix-them-the-select-n-1-problem/
- 

Đây là một performance anti-pattern. Một ứng dụng bị `N + 1 problem` khi nó phải tạo N + 1 lần gọi database( với N là số lượng object cần fetch). Giống như hầu hết các antipatterns, bản thân nó không nhất thiết là một vấn đề, tuy nhiên trong nhiều trường hợp khi N lớn thì sẽ làm giảm performance đáng kể do việc tạo hàng trăm, hàng nghìn database call trong một business transaction.

> Bạn đang spam database với nhiều query nhỏ, nhanh thay vì một hoặc hai query phức tạp.

Ví dụ:
Thường thì vấn đề nay hay xảy ra khi muốn query trên các table/collection có quan hệ cha con. Ở đây mình sẽ lấy một ví dụ cụ thể là việc lưu trữ comment, những comment reply cho một comment nào đó sẽ giữ quan hệ cha/con.
```
# Lấy list comments
parent_comment_ids = this.model.find({});

# Lấy comments trả lời cho những comment trên
child_comments = this.model.find({
	parent_id: parent_comment_ids,
});
```

Khi không có nhiều parents hoặc children thì việc này không phải quá tệ. Những nếu như nó lên tới hàng ngàn thì mỗi transaction sẽ gọi database tới hàng ngàn lần. Dù là mỗi lần gọi đều rất nhanh, nhưng response time của cả transaction có thể sẽ tới vài giây. Và vấn đề này sẽ rất tệ nếu traffic ngày càng tăng.

#### Cách để phát hiện Select N + 1 Problem
- Khi vấn đề xảy ra thì database vẫn sẽ trông như bình thường - nghĩa là không có bất kì long-running query nào cả, và CPU có thể đạt mức bình thường.
- Nên cách tốt nhất là sử dụng các công cụ theo dõi hiệu năng (performance monitoring) cho phép bạn xem được cụ thể các Business Transaction (user request) và xem code thực thi để biết nó đang tương tác đến DB như thế nào.

#### Solution
- Thường thì chúng ta hay dùng các kiểu ORM(object/relational mapper) nên nó sẽ có cơ chế lazing loading sẽ dẫn đến Select N + 1 problem, tương tự với `mongoose` khi chạy với populate cũng sẽ gặp vấn đề tương tự.
- Do đó để giải quyết vấn đề thì chúng ta sẽ gom những id lại rồi dùng query để get (giống như join 2 tập dữ liệu trên SQL) để giảm số lượng query phải thực hiện, và sau đó có thể dùng application code để mapping.


