# Mongo Indexing

References:
- https://dzone.com/articles/effective-mongodb-indexing-part-1
- https://dzone.com/articles/effective-mongodb-indexing-part-2
- https://stackoverflow.com/questions/33921125/when-are-mongodb-indexes-updated

- Bất kì một write operation nào thay đổi những index field đều sẽ update index lại index của document đó.
- Nếu update một document làm nó lớn hơn size đã được cấp thì MongoDB sẽ update tất cả những index trong document đó như là một phần của lệnh update.

-> Do nó nếu như ứng dụng của bạn là write-heavy thì việc tạo quá nhiều index sẽ ảnh hưởng đến performance.

- Mongo dùng B-Tree làm index
- B-Tree hỗ trợ tìm kiếm và tìm kiếm theo range.
- Cost để update lại B-Tree sẽ lớn nếu ko còn free space trong block (phải split index để tạo block mới và move data từ 1 block sang block mới).

1. Index Selectivity
- Mức độ chọn lọc của một thuộc tính hay một group thuộc tính là một thước đo quen thuộc về độ hữu ích của index trên những thuộc tính đó.

- Document hay index có tính chọn lọc (selective) nếu chúng có số lượng giá trị unique lớn hoặc ít giá trị trùng.

- Ví dụ ngày tháng năm sinh sẽ selective hơn là giới tính.

- Lí do selective index hiệu quả hơn vì chúng sẽ point trực tiếp tới những giá trị cụ thể thay vì là 1 list giá trị.

2. Concatenated Indexes
- Đơn giản là index kết hợp nhiều thuộc tính
- Việc này giúp cho index sẽ selective hơn so với single key index.
- Concatenated indexes có thể được dùng trong query ngay cả khi không có tất cả key được đánh index xuất hiện trong query đó.
- Chỉ cần `initial` hay `leading` attribute được dùng thì concatenated index đã phát huy tác dụng rồi.

> Leading attributes là những thuộc tính xuất hiện sớm nhất trong định nghĩa index

### Index hay không Index?

- Khi nào thì việc dùng index sẽ hiệu quả hơn là collection scan? Việc này còn phụ thuộc vào nhiều yếu tố:
	- `Caching effects`: lấy từ index sẽ có hit rates tốt hơn trong Mongo cache so với full collection scan. Nhưng nếu collection được đưa hết vào cache thì collection scan sẽ có tốc độ gần như index.
	- `Document size`: phần lớn thì 1 document sẽ được lấy chỉ với 1 single IO, nên size của document không ảnh hưởng đến performance. Nhưng document size lớn dẫn đến collection size lớn nên sẽ làm tăng IO cần cho collection scan.
	- `Data distribution`: nếu document được lưu trữ theo thứ tự của thuộc tính được index thì sẽ có hiệu năng cao hơn, vì khi đó thì index sẽ phải đi qua ít block hơn. Dữ liệu được lưu trữ theo thứ tự thường được gọi là `highly clustered`.
	
- Nếu data phân bổ random thì collection scan sẽ nhanh hơn index scan nếu hơn 8% collection được get ra. Tuy nhiên nếu data `highly clustered` index scan sẽ nhanh hơn collection scan tới 95%.

- Một vài lưu ý như sau:
	- Nếu tất cả document hoặc lượng lớn document cần được truy cập thì full collection scan là cách nhanh nhất.
	- Nếu cần lấy single document từ một collection lớn thì index trên attribute cần lấy là ngon nhất.
	- Còn ở giữa thì rất khó để đoán được.
	
### Overriding Optimizer:

- Mặc định optimizer của Mongo sẽ ưu tiên chọn sử dụng index scan.
- Trong trường hợp cần query tất cả trong collection hoặc là query số lượng lớn document thì có thể thêm option sau để sử dụng collection scan.
```
db.getSiblingDB("test").getCollection("messages").find(
   {
      text: 'xin chào mình là Duy'   
   }
).hint({$natural: 1})
```
- Tham khảo thêm `hint` ở [đây](https://docs.mongodb.com/manual/reference/method/cursor.hint/)

### Indexes for Sorting:
- Không phải tất cả trường hợp sort dùng index đều tốt, cũng giống như trường hợp get document thì sẽ có những trường hợp sau:
	- Nếu chỉ tìm một vài document, thì index sẽ nhanh hơn collection scan
	- Nhưng nếu sort tất cả document của collection thì sẽ thấy giảm thời gian sort nhưng tăng lượng data phải lấy.
- Đối với sort thì dùng index sẽ lấy first document nhanh hơn collection scan, nhưng nếu lấy last document thì collection scan nhanh hơn
- Mặc định memory cho sorting là 128M, nếu muốn tăng thì phải set thêm `internalQueryExecMaxBlockingSortBytes`. Xem thêm ở [đây](https://stackoverflow.com/questions/56157659/how-to-set-internalqueryexecmaxblockingsortbytes-in-mongo-conf)

### Using indexes for Joins
- Có thể dùng aggregate để join 2 collections.
- Khi dùng aggregate để join thì nên lưu ý đánh index cho `foreignField` nếu ko thì mỗi lần tìm thì Mongo sẽ phải thực hiện 1 collection scan.

### Effect of Indexes on Insert/Update/Delete
- Tránh tạo index trên những attributes được thường xuyên update.