# Một vài pattern phổ biến trong MongoDB

Thường thì mọi người sẽ được học về SQL và cách tổ chức dữ liệu trong các hệ quản trị cơ sở dữ liệu quan hệ hơn là trong các database dạng NoSQL như MongoDB. Vì thế, nên đôi khi chúng ta chưa tận dụng hết được sức mạnh cũng như tối ưu được hiệu năng của MongoDB.

Thông qua loạt bài viết về các pattern design DB trên https://www.mongodb.com/developer/how-to/ chúng ta sẽ biết được cách sử dụng MongoDB tốt hơn.

Let's go.

1. Polymorphic pattern (cấu trúc đa hình):
- Pattern này được sử dụng khi bạn có những documents có nhiều điểm giống nhau hơn là khác nhau. Và cũng rất phù hợp với việc chúng ra muốn đưa nhiều documents về chung 1 colleciton.

- Khi mà tất cả các documents trong 1 collection có vẻ giống nhau, nhưng không phải hoàn toàn (về mặt cấu trúc) thì khi đó chúng ta đang dùng Polymorphic Pattern.

- Việc này giúp chúng ta:
	- Truy vấn dữ liệu chỉ từ 1 collection
	- Gom dữ liệu dựa trên truy vấn mà chúng ta muốn sử dụng giúp tăng performace. (thay vì phải truy vấn trên nhiều collections)

- Reference: https://www.mongodb.com/developer/how-to/polymorphic-pattern/

2. Attribute Pattern (cấu trúc thuộc tính)
- Pattern này rất phù hợp với các trường hợp sau:
	- Chúng ta có những document lớn với những field giống nhau, nhưng có 1 tập con những field có chung những đặc tính và chúng ta muốn sắp xếp hoặc truy xuất trên những tập đó
	- Những field chúng ta cần sắp xếp chỉ được tìm thấy trong 1 tập con documents.
	- Hoặc cả hai trường gợp trên.

- Về mặt hiệu năng, để optimize việc tìm kiếm thì chúng ta cần đánh index trên tất cả những tập con đó. Và việc tạo tất cả những index đó có thể giảm performance. Vì thế Attribute Pattern là giải pháp tốt cho những trường hợp này.

- Với Attribute Pattern chúng ta có thể move tập con thông tin này vào một mảng chứa các sub-document dạng key-value và giảm số lượng index cần đánh. 
- Bằng việc sử dụng Pattern này chúng ta có thể tổ chức những documents với những đặc tính chung và tính trước được những trường hợp đặc biệt. 

- Reference: https://www.mongodb.com/developer/how-to/attribute-pattern/