# Process and thread

Trả lời những câu hỏi sau:
- Process là gì? Thread là gì?
- Process và thread khác nhau như thế nào?
- OS xử lí multi thread thế nào?
- OS xử lí multi process thế nào?
- Minh họa thread, process khi chạy 1 ứng dụng

1. Định nghĩa
- Process:
	- Process được tạo ra khi có một chương trình cần được chạy, hệ điều hành dùng process để cung cấp cho chương trình những gì nó cần để thực thi, ví dụ memory, disk space, ...
	- Process được scheduled(lên lịch) trong CPU để thực thi.
	- Process có thể tạo được nhiều process con khác.
	- Process tốn thời gian để terminate, và nó hoàn toàn độc lập với các process khác do không chia sẻ memory.
	- Ví dụ như khi có một đoạn code Nodejs trong file `index.js`, đây được gọi là program(chương trình), khi chúng ta chạy nó bằng lệnh `node index.js`, lúc đó nó trở thành một process.
	- Một chương trình có thể tạo nhiều process bằng cách run nhiều lần, ví dụ như khi mở một file `.exe` nhiều lần, thì khi đó nhiều process sẽ được tạo ra.
	- Process trong memory sẽ trông như sau:
	![process](https://github.com/duysmile/exp-notes/blob/master/images/process-in-mem.png?raw=true)
		- Text:
		- Stack:
		- Data:
		- Heap:
	

### Source:
- https://developer.chrome.com/blog/inside-browser-part1/?fbclid=IwAR2Gj8pyL3z1d_RzNlY3kolN8100eF8ZbVG2zTuxFq1RN4bxbDVWhkY11AY
- https://linuxhint.com/process-vs-thread-linux/
- https://www.guru99.com/difference-between-process-and-thread.html#1
- https://www.geeksforgeeks.org/introduction-of-process-management