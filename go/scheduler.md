# Scheduling in Go

Design và behavior của Go scheduler cho phép những ứng dụng Go chạy multithreaded một cách hiệu quả và tối ưu.
Việc này nhờ vào sự tương đồng về cơ chế của Go Scheduler với Scheduler của hệ điều hành (OS Scheduler).

## OS Scheduler

Source: https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part1.html

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
Nếu đang dùng Linux, Mac hay Windows thì bạn đang dùng một hệ điều hành đã có sẵn scheduler.

Và bạn cần lưu ý vài điều:
- Đầu tiên là không thể đoán trước được scheduler sẽ chọn Thread nào, vì nó sẽ phải dựa vào độ ưu tiên cùng với những event(như nhận data từ network) để quyết định chọn gì và khi nào.
- Thứ hai là bạn không thể viết code dựa trên những behavior mà bạn may mắn được trải nghiệm nhưng không thể đảm bảo là nó sẽ luôn đúng. Và rất dễ để bạn nghĩ là 1000 lần mà nó chạy thế này thì chắc chắn là thế rồi, điều này hoàn toàn không đúng đâu nhé.

Việc swap Thread trên 1 core được gọi là `context switch`. Context switch xảy ra khi scheduler lấy một `excecuting` thread ra khỏi 1 core và thay thế bằng một `runnable` thread. Và thread được chọn để chạy sẽ đổi sang trạng thái `executing`. Thread được lấy ra thì có thể về trạng thái `runnable`(nếu nó vẫn còn có thể chạy tiếp) hoặc là về trạng thái `waiting`(nếu nó bị thay thế bởi một IO-bound request)

Context switch được xem là tốn nhiều chi phí, vì nó cần thời gian để swap Thread vào và ra khỏi một core. Thường thì độ trễ của context switching phụ thuộc nhiều yếu tố, và tốn khoảng từ `~1000 -> ~1500 nano giây`, thường thì phần cứng có thể xử lí trung bình khoảng 12 câu lệnh mỗi nano giây, thì context switch có thể tốn tới `~12k -> ~18k lệnh`.

Nếu chương trình của bạn tập trung vào IO-Bound thì context switch sẽ rất có lợi vì khi một thread chuyển sang waiting thì thread khác ở trạng thái runnable sẽ thay thế. Điều này đảm bảo core luôn hoạt động. Không để cho core rảnh rỗi nếu có việc cần làm.

Nhưngg nếu chương trình tập trung chủ yếu vào các tác vụ CPU-Bound thì context switch sẽ ảnh hưởng rất nhiều đến performance. Vì Thread luôn có việc phải làm nên context switch sẽ làm cho công việc của thread bị gián đoạn.

## Less is More
Càng ít thread ở trạng thái runnable nghĩa là sẽ càng ít overhead cho schedule, và nhiều thời dành cho mỗi Thread. Càng nhiều Thread ở trạng thái runnable thì càng ít thời gian cho Thread, và càng ít tác vụ được hoàn thành hơn theo thời gian.

## Find the balance
Để có được throughput tốt nhất cho ứng dụng thì cần tìm được tỉ lệ phù hợp giữa số core và số Thread bạn cần chạy. Để quản lí việc này thường chúng ta sẽ dùng Thread pools.

Khi viết webservice và cần tương tác với database thì con số magic `3 thread trên mỗi core` thường cho throughput tốt nhất. Con số này giảm thiểu độ trễ của context switch và tận dụng tối đa thời gian xử lí trên các core.

## Cache lines
Truy cập dữ liệu trên main memory thường cho độ trễ lớn (~100 -> ~300 clock cycles) nên vi xử lí và core có local cache để giữ cho dữ liệu ở gần với hardware thread cần chúng. Truy cập dữ liệu từ cache tốn ít chi phí hơn (~3 -> ~40 clock cycles). Ngày nay thì một cách để tối ưu hiệu năng là đưa dữ liệu vào các vi xử lí để giảm thiệu độ trễ do truy cập dữ liệu. Việc viết các ứng dụng đa luồng có thay đổi các state cần phải xem xét kĩ càng cơ chế của hệ thống caching.

Dữ liệu được trao đổi giữ vi xử lí và main memory thông qua `cache lines`. Cache line là 64-byte chunk memory được dùng để trao đổi dữ liệu giữ main memory và hệ thống caching.

Khi mà nhiều thread chạy song song cùng truy cập tới 1 dữ liệu hoặc là những dữ liệu gần nhau thì có thể chúng sẽ truy cập dữ liệu trên cùng một cache line. Bất cứ thread nào chạy trên core đều có một bản sao của cùng cache line.

Nếu một thread trên 1 core thay đổi bản sao của cache line thì tất cả các bản sao của cùng cache line đó sẽ bị đánh dấu là `dirty`. Và khi thread nào cần đọc hay ghi vào cache line đã đánh dấu `dirty` đó thì đều cần phải truy cập main memory để lấy bản sao mới của cache line.

Đây được gọi là `cache-coherency problem`. Nên khi viết các ứng dụng đa luồng có thay đổi shared state thì cần phải lưu ý đến caching system.

## Scheduling Decision Scenario
Sau đây là một vài thứ thú vị mà schedule sẽ xem xét khi đưa ra quyết định scheduling.

Context như sau:

"Khi chạy một ứng dụng và main thread được tạo, và xử lí trên core 1. Thread bắt đầu thực thi các câu lệnh, cache line được truy cập vì có dữ liệu được yêu cầu. Thread lúc này quyết định tạo 1 thread mới cho những xử lí đồng thời."

Khi đó thì scheduler sẽ cần trả lời những câu hỏi sau:
1. Có nên context-switch main thread ra khỏi core 1? Việc này có thể cải thiện hiệu năng, vì có thể thread mới sẽ cần cùng dữ liệu với main thread đã được cache. Nhưng main thread sẽ không được thực thi full time.
2. Hay là thread nào đang đợi ở core 1 sẽ phải đợi đến khi main thread thực thi xong? Thread sẽ không được chạy nhưng độ trễ để fetch dữ liệu sẽ không tốn khi nó bắt đầu chạy.
3. Hay là để thread đợi core nào available? Việc này có nghĩa là cache line ở core đó sẽ phải được flushed, retrieved và duplicated, có thể gây ra độ trễ. Tuy nhiên thread sẽ bắt đầu nhanh hơn và main thread vẫn có thể tiếp tục công việc của nó.
	
---
## Go Scheduler

### Startup
Khi chương trình Go start thì nó được cung cấp một Logical processor (P) với mỗi virtual core.

Nếu bạn có một vi xử lí có nhiều hardware threads trên mỗi physical core (Hyper-threading), thì mỗi hardware thread được thể hiện trong Go như một virtual core.

Ví dụ đối với máy core i7, thì vi xử lí có 4 core vật lí (physical core). Core i7 có cơ chế Hyper threading, nên mỗi core vật lí sẽ có 2 hardware threads trên mỗi core. Do đó ở chương trình Go ta sẽ thấy có 8 virtual core để thực thi OS threads song song.

Một số thuật ngữ:
- Processor (P): để thể hiện 1 virtual core
- Machine (M): để thể hiện một OS thread
- Goroutine (G): để chỉ một goroutine

Mỗi P sẽ đi cùng với 1 M, Thread này được quản lí bởi OS, và OS có trách nhiệm đặt Thread này vào Core để xử lí. Việc này có nghĩa là khi chạy một chương trình Go trong ví dụ này thì ta có 8 thread để thực thi công việc, mỗi thread gắn với một P.

Khi một chương trình Go chạy bản chất vẫn là tạo ra một goroutine. Goroutine có thể được xem là thread ở tầng application, và có nhiều nét giống với OS thread. OS thread thì context-switch trên core còn Goroutine thì context-switch trên M (OS Thread).

Cuối cùng là `run queue`. Có 2 loại run queue trong Go scheduler: Global Run Queue (GRQ) và Local Run Queue (LRQ). Mỗi P sẽ có một LRQ để quản lí các Goroutine được assign để thực thi trong context của P. Những Goroutine này sẽ được context-switch trên M được assign cho P. Còn GRQ sẽ chứa những Goroutine chưa được gán cho bất kì P nào. Sẽ có một process để đưa Goroutine từ GRQ vào LRQ để thực thi.

![GRQ, LRQ](https://www.ardanlabs.com/images/goinggo/94_figure2.png)

OS Scheduler thường có sẵn ở OS, gọi là `preemptive scheduler`, chạy ở kernel nên chúng ta không thể dự đoán được cách nó vận hành. Còn Go Scheduler chạy ở user space, ở trên kernel, gọi là `cooperating scheduler`, nghĩa là scheduler cần những sự kiện ở user space được định nghĩa rõ ràng để đưa ra quyết định scheduling.

Go Scheduler có một cái hay là nó trông giống như `preemptive scheduler` vì bạn không thể đoán trước được nó sẽ làm gì, và việc quyết định scheduling không phụ thuộc vào developer mà nó xảy ra ở Go runtime.

### Goroutine States
Goroutines có 3 trạng thái:
- Waiting: lúc này goroutine đã dừng lại và phải đợi một cái gì đó để có thể thực thi tiếp. Có thể là đợi OS (system calls) hoặc là đợi những tác vụ đồng bộ (atomic, mutex)
- Runnable: ở trạng thái này thì goroutine đang đợi được gắn vào một M để thực thi. Nếu có quá nhiều routine ở trạng thái này thì sẽ càng phải đợi lâu hơn, và thời gian để thực thi của mỗi routine cũng ngắn hơn
- Executing: lúc này routine đã được gắn vào M và đang thực thi.

### Context switching
Có 4 kiểu event làm cho scheduler đưa ra quyết định:
- Dùng keyword *go*: tạo ra 1 goroutine, khi đó scheduler có cơ hội đưa ra quyết định schedule.
- Garbage collection: khi GC chạy với tập goroutine của nó, thì những routine này cần M để chạy.Việc này làm cho GC schedule loạn xạ. Tuy nhiên thì scheduler biết được routine đang làm gì và tận dụng điều này để đưa ra những quyết định sáng suốt. Một quyết định sáng suốt là việc context-switch một routine muốn xài heap với những routine khác ko cần dùng tới heap trong quá trình GC.
- System calls: nếu một routine chạy một system call sẽ làm nó block M, đôi khi thì scheduler phải context-switch nó ra khỏi M và context-switch một routine khác vào M đó. Tuy nhiên cũng có trường hợp một M mới được yêu cầu để tiếp tục thực thi những routine đang được xếp hàng đợi trong P.
- Synchronization và Orchestration: nếu những operation atomic, mutex hoặc channel làm routine bị block, thì scheduler có thể context-switch sang một routine mới để chạy. Khi routine có thể chạy lại thì nó được re-queue và context-switch lại.

### Asynchronous System Calls
Với những network-based system calls thì OS có thể xử lí chúng bất đồng bộ nhờ có 1 thành phần là `network poller`. Ở mỗi OS sẽ dùng mỗi loại khác nhau, ví dụ như MacOS dùng kqueue, Linux dùng epoll còn Windows thì là iocp.

Khi dùng network poller để xử lí các networking system calls thì scheduler có thể tránh được việc goroutine sẽ block M khi có những system calls này. Việc này giúp cho M available để thực thi tiếp những routine trong LRQ của P thay vì phải tạo một M mới, giúp giảm tải scheduling trên OS.

Khi một routine tạo ra một networking call thì nó sẽ được đẩy ra network poller để xử lí, khi đó M có thể lấy routine khác đang có trong LRQ để xử lí. Routine xử lí network ban đầu sau khi được network poller xử lí xong thì được push trở lại vào LRQ.

### Synchronous System Calls
Có những system calls không thể chạy bất đồng bộ ví dụ như việc file-based system call, khi đó thì network poller không thể được sử dụng. Lúc này thì goroutine thực hiện system call sẽ block M.

Và Go Scheduler xử lí case này bằng cách:
- Tách M ra khỏi P cùng với Goroutine đang block, một M mới (có thể được tạo mới hoặc lấy lại trong quá trình transition swap trước đó) được gắn vào P và Goroutine khác trong LRQ được context-switch.
- Sau khi blocking system call chạy xong thì Goroutine bị blokc trước đó được thêm vào LRQ lại. Còn M ban đầu có thể được dùng trong tương lai.

### Working Stealing
Go scheduler còn là một work-stealing scheduler. Điều này giúp cho việc scheduling được hiệu quả hơn.

Điều sau cùng bạn mong muốn chính là đưa M vào waiting state vì lúc đó OS sẽ context-switch M khỏi core. Đồng nghĩa với việc P không thể hoàn thành công việc được, dù có goroutine đang ở trạng trái runnable đi nữa cho đến khi M được context-switch trở lại core.

Work stealing còn giúp cân bằng số lượng Goroutine giữa các P để có thể phân bố công việc tốt hơn và hoàn thành chúng hiệu quả hơn.

Ví dụ:
- Một chương trình Go chạy multi-thread với mỗi P có 4 goroutine cần chạy ở LRQ và một routine ở GRQ.
- Nếu một P chạy nhanh hơn cái còn lại, thực hiện xong hết routine thì nó sẽ xem xem là ở P còn lại có routine nào không, nó sẽ lấy *một nửa* số routine về chạy tiếp.
- Nếu một P hoàn thành xong công việc, ở P còn lại cũng không còn routine nào trong LRQ thì nó sẽ xem xem GRQ còn routine nào không nó sẽ lấy về chạy nốt.

### Source: 
- https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part2.html
- https://rakyll.org/scheduler/
- https://granulate.io/deep-dive-into-golang-performance/
- https://go.dev/src/runtime/proc.go
- https://docs.google.com/document/d/1TTj4T2JO42uD5ID9e89oa0sLKhJYD0Y_kqxDv3I3XMw/edit
- https://runi.me/post/go-routine-under-the-hood/
