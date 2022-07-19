# Process in OS

Process được tạo ra khi có một chương trình cần được chạy, hệ điều hành dùng process để cung cấp cho chương trình những gì nó cần để thực thi, ví dụ memory, disk space, ...

Process được scheduled(lên lịch) trong CPU để thực thi.

Process có thể tạo được nhiều process con khác.

Process tốn thời gian để terminate, và nó hoàn toàn độc lập với các process khác do không chia sẻ memory.
- Ví dụ như khi có một đoạn code Nodejs trong file `index.js`, đây được gọi là program(chương trình), khi chúng ta chạy nó bằng lệnh `node index.js`, lúc đó nó trở thành một process.
- Một chương trình có thể tạo nhiều process bằng cách run nhiều lần, ví dụ như khi mở một file `.exe` nhiều lần, thì khi đó nhiều process sẽ được tạo ra.
 
Process trong memory sẽ trông như sau:

![process](https://media.geeksforgeeks.org/wp-content/cdn-uploads/gq/2015/06/process.png)

	- Text: chứa thông tin hoạt động hiện tại được thể hiện bằng giá trị của `Program Counter`.
	- Stack: chứa những thông tin tạm, ví dụ như function parameters(tham số của hàm), return addresses(địa chỉ trả về), và local variables.
	- Data: chứa global variable.
	- Heap: bộ nhớ được cấp phát động cho process trong runtime.
 
Những thuộc tính của một Process:
- process id: định danh của process được cấp bởi OS
- process state: trạng thái của process, có thể là ready, running, ...
- CPU registeres: giống như Program Counter(CPU registers phải được lưu và có thể restored khi mà process swap in hoặc out khỏi CPU).
- Account information
- I/O status information: ví dụ như những device được cấp phát cho process, ...
- CPU scheduling information

→ tất cả những thuộc tính trên được gọi là `context` của process
- Mỗi process có một `process control block` (PCB) của riêng nó, và những thuộc tính trên là một phần của PCB.

### Context Switching
Process dùng để save context của một process và load context của một process khác gọi là `Context Switching`.

Khi nào thì xảy ra Context switching?
1. Khi một process có priority cao hơn vào trạng thái ready
2. Khi có ngắt (interupt) xảy ra
3. Khi User mode và Kernel mode switch
4. Khi CPU scheduling được sử dụng

Process switching sẽ gọi một mode switch (user và kernel modes). Context switching chỉ xảy ra ở kernel mode.

### Source:
- https://developer.chrome.com/blog/inside-browser-part1/?fbclid=IwAR2Gj8pyL3z1d_RzNlY3kolN8100eF8ZbVG2zTuxFq1RN4bxbDVWhkY11AY
- https://linuxhint.com/process-vs-thread-linux/
- https://www.guru99.com/difference-between-process-and-thread.html#1
- https://www.geeksforgeeks.org/introduction-of-process-management
- https://levelup.gitconnected.com/difference-between-process-and-thread-479986d15bb6
- https://betterprogramming.pub/pitfalls-of-multithreaded-programs-and-achieving-thread-safety-using-go-1ac80c8d8106
