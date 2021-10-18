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

<details>
	<summary>**1. Data modeling and memory sizing**</summary>

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

