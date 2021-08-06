# Software design

Talk: youtube.com/watch?v=7PGCvpJl_0o

All about complexity

1. Có 2 loại complexity:
- Essential complexity: bởi vì sự phức tạp này đến từ nghiệp vụ, yêu cầu của ứng dụng, hệ thống nên không thể lượt bỏ được.
- Accidental complexity: do trong quá trình phát triển, design chúng ta tự thêm vào để giải quyết vấn đề

2. 5 thành phần của complexity
- Độ quan trọng từ cao xuống thấp:
	- Shared mutalble state
	- Side effects
	- Dependencies
	- Control flow
	- Code size
3. 2 cách để quản lí complexity
- Reduce complexity
- Isolate complexity

4. OOP
- Only isolate complexity.

5. Use Functional programming to reduce complexity
- Core của functional programming là chúng ta sẽ tập trung vào `data flow` thay vì ~~`control flow`~~.

- Functional programming deal with values; imperative programming deals with objects - *Alex Stepanov*.

- Cách để kết hợp Functional Programming và Imperative Programming là `Function Core Imperative Shell` (tham khảo ở https://mokacoding.com/blog/functional-core-reactive-shell/)
