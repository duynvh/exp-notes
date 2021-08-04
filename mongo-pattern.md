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
	
- Ví dụ: data cho một kì Olympic, thay vì lưu thành tích các vận động viên theo bộ môn, chúng ta có thể lưu chung tất cả trong 1 collectio. Vì các vận động viên căn bản sẽ có những thuộc tính khá giống nhau, sẽ khác nhau ở phần bộ môn và thành tích bộ môn đấy thôi.
```
{
	"name": "Duy Nguyen",
	"nation": "Vietnam",
	...
	"sport": "football",
	"goal": 10,
	"medal": "gold",
},
{
	"name": "Duy Dep trai",
	"nation": "Vietnam",
	...
	"sport": "tennis",
	"event": [{
		"type": "single",
		"medal": "silver",
	}],
}
```

- Reference: https://www.mongodb.com/developer/how-to/polymorphic-pattern/

2. Attribute Pattern (cấu trúc thuộc tính)
- Pattern này rất phù hợp với các trường hợp sau:
	- Chúng ta có những document lớn với những field giống nhau, nhưng có 1 tập con những field có chung những đặc tính và chúng ta muốn sắp xếp hoặc truy xuất trên những tập đó
	- Những field chúng ta cần sắp xếp chỉ được tìm thấy trong 1 tập con documents.
	- Hoặc cả hai trường gợp trên.

- Về mặt hiệu năng, để optimize việc tìm kiếm thì chúng ta cần đánh index trên tất cả những tập con đó. Và việc tạo tất cả những index đó có thể giảm performance. Vì thế Attribute Pattern là giải pháp tốt cho những trường hợp này.

- Với Attribute Pattern chúng ta có thể move tập con thông tin này vào một mảng chứa các sub-document dạng key-value và giảm số lượng index cần đánh. 
- Bằng việc sử dụng Pattern này chúng ta có thể tổ chức những documents với những đặc tính chung và tính trước được những trường hợp đặc biệt. 

- Ví dụ: data dữ liệu số cuộc gọi thành công thất bại của 1 chiến dịch quảng cáo, thay vì phải lưu 2 field là `success`, `fail`, và có thể là thêm nhiều trạng thái khác nếu business thay đổi thì chúng ta có thể gom chúng thành dạng:
```
{
	...
	status: [
		{
			type: "success",
			quantity: 10,
		},
		{
			type: "fail",
			quantity: 2,
		},
	],
}
```

- Reference: https://www.mongodb.com/developer/how-to/attribute-pattern/

3. Bucket Pattern
- Pattern này rất phù hợp khi làm việc với IoT, thống kê real-time, hoặc dữ liệu time-series nói chung. Bằng việc gom nhóm (bucketing) dữ liệu lại với nhau, chúng ta sẽ dễ dàng tổ chức các group data cụ thể, tăng khả năng dự đoán tương lai cũng như khám phá các trend quá khứ nhờ dữ liệu, và tối ưu sử dụng bộ nhớ lưu trữ.

- Vì dữ liệu trong những case này sẽ là rất nhiều, và cập nhật liên tục, nếu như chúng ta lưu trữ như đối vs SQL thì lượng document sẽ rất lớn, dẫn đến tốn bộ nhớ lưu trữ cũng như index size. Bằng việc tận dụng document data model, chúng ta có thể "bucket" data lại theo thời gian, 1 document sẽ lưu dữ liệu trong 1 khoảng thời gian cụ thể (ví dụ 1h, 1 ngày).

- Sử dụng Pattern này chúng ta sẽ giảm được index size, đơn giản truy vấn, và có thể dùng dữ liệu tổng hợp trong 1 document.

- Ví dụ: data gửi lên từ 1 sensor sẽ nhất nhiều và nếu phải lưu trữ theo cách thông thường thì mỗi phút sẽ phải lưu 1 record, như vậy lượng data sẽ rất nhiều, thay vào đó chúng ta sẽ gom data trong 1h vào 1 document, vậy là chúng ta đã tiết kiệm bộ nhớ đi rất nhiều.
```
// before
{
   sensor_id: 12345,
   timestamp: ISODate("2019-01-31T10:00:00.000Z"),
   temperature: 40
}
{
   sensor_id: 12345,
   timestamp: ISODate("2019-01-31T10:01:00.000Z"),
   temperature: 40
}
{
   sensor_id: 12345,
   timestamp: ISODate("2019-01-31T10:02:00.000Z"),
   temperature: 41
}

// after
{
    sensor_id: 12345,
    start_date: ISODate("2019-01-31T10:00:00.000Z"),
    end_date: ISODate("2019-01-31T10:59:59.000Z"),
    measurements: [
       {
       timestamp: ISODate("2019-01-31T10:00:00.000Z"),
       temperature: 40
       },
       {
       timestamp: ISODate("2019-01-31T10:01:00.000Z"),
       temperature: 40
       },
       ...
       {
       timestamp: ISODate("2019-01-31T10:42:00.000Z"),
       temperature: 42
       }
    ],
   transaction_count: 42,
   sum_temperature: 2413
}
```

- Reference: https://www.mongodb.com/developer/how-to/bucket-pattern/

4. Outliner Pattern
- Pattern này giúp chúng ta phòng ngừa đc một vài truy vấn hay một vài documents sẽ dẫn chúng ta tới 1 giải pháp ko tối ưu với phần lớn các use cases.

- Outliner pattern được sử dụng khi có một sự khác biệt đáng kể trong dữ liệu, còn nếu được xem là `bình thường` thì việc thay đổi kiến trúc của app cho chúng sẽ làm giảm hiệu năng cho hầu hết các truy vấn đặc trưng.

- Ví dụ: dữ liệu bán sách, chúng ta có thể hoàn toàn lưu trữ danh sách user đã mua sách trong cùng document thông tin sách, như vậy việc truy vấn sẽ nhanh hơn và đơn giản hơn. Nhưng đối với cuốn sách dạng best seller thì lượng người mua có thể tới hàng triệu, khi đó chúng ta cần tách danh sách người mua này sang một collection khác, và ở document chứa thông tin sách này có 1 field để phân biệt nó với phần lớn document `bình thường` còn lại, như là `has_extras: true`

- Reference: https://www.mongodb.com/developer/how-to/outlier-pattern/

5. Computed Pattern
- Pattern này sử dụng khi chúng ta có những dữ liệu cần được tính toán nhiều lần trong ứng dụng. Và được sử dụng khi dữ liệu của chúng ta có lượng read > write. Ví dụ lượng read là 1000.000/h và write chỉ là 1000/h, sử dụng pattern này giúp việc tính toán giảm bớt 1000 lần.

- Pattern này khá đơn giản, thay vì phải tính toán mỗi khi read data thì chúng ta có thể làm việc này trước, ở background, kết quả có thể không chính xác 100% tại thời điểm read, nhưng bù lại lượng resource tiết kiệm được sẽ rất lớn và nếu dữ liệu không cần độ chính xác tuyệt đối, khi đó chúng ta nên áp dụng pattern này.

- Ví dụ: khi xem dữ liệu 1 bài post trên facebook, chúng ta có thể thấy lượng like và comment của post đó, thì việc đếm số lượng like và comment này chúng ta có thể tính toán từ trước, mỗi khi user like hoặc comment và post và lưu lại như 1 phần thông tin post thay vì mỗi lần truy vấn phải đi count lại số lượng like và comment.

- Reference: https://www.mongodb.com/developer/how-to/computed-pattern/

6. Subset Pattern
- 
