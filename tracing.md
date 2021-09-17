# Distributed Tracing (DT)

Hiểu một cách đơn giản `Distributed tracing` là cách 1 request chuyển từ service này sang service khác.

- DT về cơ bản là về việc `tracking` và `analyzing` requests khi chúng di chuyển quanh distributed architecture.
- Khác với cách monitoring truyền thống là chỉ tập trung vào một service độc lập, và chi tiết request sẽ ảnh hưởng tới các số liệu tổng hợp.

### Why?
Để trả lời được các câu hỏi sau:
> Những services nào mà request đó đã đi qua?
> Bottlenecks nằm ở đâu?
> Thời gian request mất do network lag khi giao tiếp giữa các services?
> Những gì xảy ra trong mỗi service cho request đó?

-> DT vẫn hữu ích ngay cả khi dùng cho single services, nó vẫn có thể dùng để theo dõi và biết được request của bạn đang gặp vấn đề ở đâu, internal service hay là outbound calls.

### Trace
- Chúng ta sẽ dùng Dapper-style tracers, dựa trên paper của [Google](https://research.google.com/pubs/pub36356.html).
- Những entities chính là `Trace` và `Span`.
	- **Trace**: cover toàn bộ request qua tất cả những services mà nó đi qua. Nó sẽ chứa tất cả *Span* trong request.
	- **Span**: là một logical chunk trong một *Trace*
	- Spans sẽ có nhiều quan hệ cha-con.

- Quan hệ cha-con chi tiết như sau:
	- Một child span sẽ chỉ có 1 parent.
	- Một parent span có thể có nhiều child span.
	- Quan hệ cha-con cho phép chúng ta tạo ra một `trace tree` cho tất cả các span của một request.
	- Trace tree sẽ có 1 root span, là span ko có parent.

- Khi một Trace bắt đầu thì sẽ có một `Trace ID` được tạo và sẽ đi theo request tới bất cứ đâu. Một Span mới được được tạo cho mỗi logical chunk of work trong request, sẽ chứa cùng `Trace ID`, một `Span ID` mới, và có thể có `Parent Span ID`. Và span cũng chưa thời gian bắt đầu và duration.

- Đối với 1 request trong microservices thì thường sẽ có 1 span chung cho cả request, và một child span cho mỗi outbound call tới một service khác, tới DB, ... 

### Span Collectors, Visualization, And Logs
- Span Collectors dùng để collect tất cả span trong distributed system.

Ref: https://medium.com/nikeengineering/hit-the-ground-running-with-distributed-tracing-core-concepts-ff5ad47c7058


