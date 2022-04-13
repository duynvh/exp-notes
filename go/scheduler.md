# Scheduling in Go

Design và behavior của Go scheduler cho phép những ứng dụng Go chạy multithreaded một cách hiệu quả và tối ưu.
Việc này nhờ vào sự tương đồng về cơ chế của Go Scheduler với Scheduler của hệ điều hành (OS Scheduler).

## OS Scheduler
Một chương trình mà chúng ta viết ra bản chất là một tập các lệnh (machine instructions) được thực thi tuần tự. Để làm được việc này thì sinh ra một khái niệm gọi là `Thread`.

Thread sẽ được sinh ra và thực thi tập lệnh này tuần tự, và nó sẽ thực hiện việc này đến khi không còn lệnh nào nữa.

Mỗi chương trình khi chạy sẽ tạo ra một `Process`, một process sẽ khởi tạo thread. Thread có thể tạo ra các thread khác. Và mỗi thread sẽ chạy độc lập và được những quyết định scheduling sẽ được thực hiện ở Thread level, chứ không phải Process level.

OS Scheduler có trách nhiệm đảm bảo sẽ không có core nào ngồi không nếu có Thread nào đó cần thực thi. Và việc này làm chúng ta có cảm giác như là các Thread đang được thực thi cùng lúc 🙄, nhưng thực chất là chúng đang được thực thi các Thread từ ưu tiên cao xuống thấp. Scheduler phải đảm bảo thời gian thực thi cho các Thread, dù là ưu tiên cao hay thấp và cần giảm thiểu độ trễ nhiều nhất có thể bằng cách đưa ra những quyết định nhanh và chính xác.

## Executing Instructions
Program counter (PC) hay còn gọi là instruction pointer(IP) giúp cho Thread biết được next instruction cần thực thi là lệnh nào. Trong hầu hết các vi xử lí thì PC sẽ trỏ tới next instruction thay vì instruction hiện tại.

## Thread States
Thread có 3 trạng thái: `Waiting`, `Runnable` hoặc `Executing`

**Waiting**: Thread lúc này đang dừng lại và chờ một vài thứ để tiếp tục, có thể là chờ phần cứng(disk, network), chờ OS(system calls) hoặc synchronization call(atomic, mutexes).

**Runnable**: Thread cần thời gian trên 1 core để thực thi tập lệnh được gán. Nếu có nhiều Thread cần thời gian để thực thì thì chúng sẽ tốn nhiều thời gian để hơn, đồng thời thì thời gian dành cho mỗi Thread cũng bị rút ngắn.

**Executing**: Thread lúc này đang ở trên core và đang thực thi tập lệnh của nó.

## Types of work
**CPU-Bound**: những việc này sẽ không bao giờ đặt Thread vào trạng thái `waiting`, nó chỉ đơn thuần là tính toán, ví dụ như tính một triệu phép cộng.

**IO-Bound**: là những việc sẽ đưa Thread vào trạng thái `waiting`, bao gồm những việc như là truy cập một resource nào đó thông qua network, hoặc là tạo một system call tới OS. Ví dụ như là khi truy cập Database chính là IO-Bound, trong bài này thì chúng ta gộp luôn những synchronization event như mutexes hay atomic vào loại này luôn vì nó cũng sẽ đưa thread vào trạng thái waiting.

## Context Switching






















