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