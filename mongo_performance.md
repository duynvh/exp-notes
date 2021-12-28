### MongoDB Performance

Series:
- Benchmark: https://www.mongodb.com/blog/post/performance-best-practices-benchmarking
- Hardware and OS Configuration: https://www.mongodb.com/blog/post/performance-best-practices-hardware-and-os-configuration
- Transactions and Read / Write Concerns: https://www.mongodb.com/blog/post/performance-best-practices-transactions-and-read--write-concerns
- Sharding: https://www.mongodb.com/blog/post/performance-best-practices-sharding
- Indexing: https://www.mongodb.com/blog/post/performance-best-practices-indexing
- Query Patterns and Profiling: https://www.mongodb.com/blog/post/performance-best-practices-query-patterns-and-profiling
- MongoDB Data Modeling and Memory Sizing: https://www.mongodb.com/blog/post/performance-best-practices-mongodb-data-modeling-and-memory-sizing

- Terms:
	- `working set`: là RAM sizing dành cho ứng dụng để chứa những data được truy cập thường xuyên hoặc indexes.

---
<details>
	<summary>
		<b>1. Data modeling and memory sizing</b>
	</summary>

## Data modeling
- Nếu biết được những query pattern của ứng dụng để design data mode và lựa chọn `index` tương ứng thì việc query sẽ hiệu quả hơn và tăng throughput khi insert và update, và xa hơn là giúp việc phân bổ workload trong một sharded cluster hiệu quả hơn.

- Một vấn đề quan trọng trong việc data model với Mongo chính là tạo quan hệ giữa những data với nhau.

### Embedding
- Dữ liệu quan hệ 1:1 hoặc 1:n, đều có thể được nhúng vào trong cùng 1 document, vì khi những data này được truy cập cùng nhau thì việc lưu trữ chúng trên 1 single document là tối ưu. 
- Embedding nhìn chung sẽ làm tăng hiệu năng của read operation nhờ khả năng request và retrieve data trên 1 single internal database operation.
- Và khi update data thì có thể dùng 1 atomic write operation vì nó chỉ xảy ra trên 1 document (write trên 1 document là atomic - được Mongo hỗ trợ).

- Tuy nhiên, không phải tất cả quan hệ 1:1, hay 1:n đều dùng embedding.
- Những trường hợp nên dùng `referencing`:
	- Document được read thường xuyên những chứa những data hiếm khi được truy cập. Thì nếu dùng embedding trong trường hợp này sẽ làm tăng in-memory requirements(working set) của collection.
	- Một phần của document được update thường xuyên và growing size, trong khi phần còn lại thì tương đối static.
	- Document size của Mongo tối đa là 16MB.
	
### Referencing
- Giúp giải quyết những vấn đề ở trên, và thường được dùng để xử lí những trường hợp quan hệ n:n.
- Tuy nhiên việc này sẽ làm cho ứng dụng của chúng ta cần thêm những follow-up queries để resolve reference, và dĩ nhiên là cần thêm round-trip tới server, hoặc là cần thêm một `joining` operation dùng `$lookup` trong aggregation pipeline.

## Memory sizing:
	
- Đảm bảo working set phải fit trong RAM
- Bên cạnh việc data modeling thì yếu tố thứ hai cần xem xét để tối ưu performance là sizing working set.
	
- Giống như hầu hết DB thì Mongo sẽ hoạt động tốt nhất nếu working set của ứng dụng (indexes và những data được truy cập thường xuyên) fit trong memory.
- Những improvement khác sẽ không thực sự hiệu quả nếu như không đủ RAM. Nếu bạn còn quan tâm đến cost thay vì chỉ về performance thì nên cân nhất việc sử dụng fast SSD thì sẽ tiết kiệm hơn (dĩ nhiên ko thể xịn như xài nhiều RAM nhé).
	
- Khi working set fit trên RAM, thì việc đọc từ disk sẽ chậm.
- Nếu working set vượt quá lượng RAM thì nên cân nhắc sử dụng một instance khác lớn hơn.
</details>

---
<details>
	<summary>
			2. Query pattern and profiling
	</summary>
	
Một số lưu ý trước khi đi vào việc profiling:
- Dùng driver mới nhất
- Tránh việc tạo những document lớn,
- Chỉ update trên những field cụ thể cần update, ko update cả document, như vậy sẽ giảm được db overhead.
- Update nhiều element của array với 1 operation.
- Dùng replica set tag để có được read isolation cho các tác vụ Analytic.
- Profile query với lệnh explain
	
</details>

---
<details>
	<summary>
		3. Indexing
	</summary>

Thay vì phải scan qua tất cả documents trong collection để tìm kiếm, thì nếu có index, và index đúng thì việc tìm kiếm sẽ tối ưu hơn rất nhiều.
	
- Compound indexes: index được tạo bởi nhiều hơn 1 field
- Trong compound indexes nên follow theo ESR Rule:
	- Đầu tiên, thêm những field cần so sánh bằng (Equality) trong query vào trước
	- Những field tiếp theo phản ánh thứ tự (Sort) search
	- Cuối cùng là những field thể hiện khoảng (Range) được truy cập.
- Sử dụng `Covered Queries`: là những câu query trả trực tiếp kết quả từ index mà ko cần thông qua source documents.
- Đối với những field có số lượng unique value nhỏ thì nên dùng compound index để tăng số unique values.
- Xóa những index không được dùng tới, vì index sẽ tốn tài nguyên RAM và disk, khi document được update thì phải cập nhật index nên sẽ tốn CPU, disk I/O.
- Wildcard indexes
- Text index
- Partial index
- Multi-key indexes
- Tránh dùng case insensitive regex mà hãy dùng `Case intensitive index`
</detail>

---
<details>
	<summary>
		4. Transaction
	</summary>
- Đối với single document thì update là một atomic operation, ngay cả khi update nhiều item trong 1 field
- Từ phiên bản 4.0 thì MongoDB đã hỗ trợ multi-document ACID Transactions. Và một số lưu ý đối với transaction để đạt hiệu năng cũng như hiệu quả cao khi sử dụng:
	- Runtime limit của transaction mặc định là 60s, có thể điều chỉnh tùy nhu cầu
	- Số operations trong một transaction, tốt nhất là không quá 1000 operations, nếu nhiều hơn thì nên chia việc xử lí thành nhiều batches
	- Xử lí exception, nên xử lí ở application logic để catch và retry khi transaction bị abort vì những temporary exceptions.
	- Giảm write latency trong 1 số trường hợp, ví dụ nếu chạy 10 update độc lập thì mỗi update phải đợi một replication round trip, nhưng nếu gom 10 update vào 1 transaction thì chúng sẽ được replicated cùng nhau tại thời điểm transaction được commit và latency có thể giảm đến 10 lần.
</detail>
