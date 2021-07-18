# Date in Javascript

Thật ra bài viết này không hẳn là về DateObject trong Javascript, tuy nhiên vì quá lười để nghĩ ra một cái tên `hay ho` thì thôi mình để thời gian để viết nội dung vẫn oke hơn đúng hông :))

Rồi vào luôn nhé mọi người ~~~

## Vấn đề mà mình gặp phải là gì?

Date trong JS thì có tội tình gì mà lại bị mình mang ra "xỉa xói" thế này 😌 thì cái gì cũng có nguyên nhân của nó cả, chứ một người lương thiện như mình thì không có vô cớ mà trách cứ "nó" đâu.

Câu chuyện bắt đầu vào một mùa deadline dí sát đ\*t, mình cùng với teammates đang phải cong đuôi lên đuổi bắt task cho xong kịp tiến độ. Mọi thứ đang rất suôn sẻ, gặp bug nào ta fix bug đó, gặp task nào ta kill task đó, và những gì trên máy của tụi mình đang là rất hoàn hảo (hoặc gần như thế 😎). Được rồi tạo `Pull Request`(PR) thôi nào, hôm nay kết thúc ở đây thôi... Và rồi tụi mình hoàn thành task và kết thúc một ngày dài làm việc ~~.

Mình đùa đấy :)) đời đâu có đẹp như mơ vậy đâu. Tụi mình quyết định test lại xem mọi thứ đã ổn trên môi trường development chưa trước khi khăn gói ra về. Với tâm trạng rất hào hứng thì chưa kịp vui mừng mình đã bị tạt một gáo nước mát lạnh lẽo nghẹn ngào. Một task liên quan đến schedule mặc dù đã setting đúng nhưng chả thấy có job nào chạy cả. Whyyy??? Mình và anh đồng nghiệp cẩn thận dò lại từng dòng code, chạy lại debug trên máy...

Vài phút sau ...

Aaaa đây rồi, fix rồi tạo PR lại thôi nào!!!

Yeah, lần này chắc chắn rồi, không thể sai được ...

Ơ đệt, lại không đúng rồi, hay là mình lại nhầm gì ta 🤔

Mình lại debug tiếp.

Rồi mình lại sửa.

Lại PR.

Lại lỗi nữa ...

🤯🤯🤯

Hơi bế tắc, nên mình quyết định giải quyết xong phải lên viết để kể cho mọi người cùng bế tắc với mình.

Mình cần làm một việc rất đơn giản là tính toán thời gian để chạy một job được hẹn giờ, client sẽ setting giờ để chạy job này.

![UI Setting](https://i.ibb.co/0mth74Q/Screen-Shot-2021-07-19-at-00-15-57.png)

Giao diện khá đơn giản thế này, và cách mình lưu dữ liệu này vào DB là thế này

```
{
	...
	start_date: 07/19/2021,
	stat_time: [{
		hour: 10,
		min: 30,
	}]
	...
}
```

`start_time` là một mảng vì có thể có nhiều mốc thời gian cần chạy schedule (mọi người đừng bảo là sao ko lưu mảng date luôn nhé, vì mình cũng muốn làm vậy cho nhẹ đầu, nhưng sự đã rồi 😭).

Và khi kích hoạt schedule này mình cần phải gắn thời gian và ngày chạy để cho ra một `date` object hoàn chỉnh. Và đây cũng chính là lúc cơn *đau khổ* ập đến.

Éo le một nỗi là như câu chuyện ở trên mình kể, mọi thứ khi chạy ở máy mình thì rất ok.

```
new Date(start_date).setHours(start_time[0].hour, start_time[0].min)
```

Với suy nghĩ rất trong sáng tinh khôi thì dòng lệnh trên đúng như ý nghĩ mà mình muốn, nhưng cuộc đời nào có êm đềm như vậy, mình đã vướng phải trở ngại ngay tức khắc. Đó chính là `múi giờ (timezone)`.

Vì ở client KH muốn setting giờ theo múi giờ hiện tại của họ, ví dụ ở Việt Nam là múi giờ +7, nhưng để đảm bảo tính nhất quán thì DB phải lưu thời gian ở múi giờ +0 (UTC). 

Và đơn giản là cách trên của mình đã sai hoàn toàn, KH muốn set chạy lúc 9h30 ngày 19/7/2021 thì khi get từ DB ra ngày 19/7/2021 00:00:00 (+7) được đổi thành 18/7/2021 17:00:00 (UTC) và lấy giá trị này set giờ sẽ là 9h30 18/7/2021 🤯

Sau đó mình nghĩ ngay tới việc phải đồng bộ timezone thôi, thế là mình phải chuyển đổi ngày được lấy ra từ DB sang múi giờ của KH rồi mới được set giờ phút. Và cách đơn giản nhất là xài lib thôi, mình chọn `moment-timezone` luôn cho lẹ.

Và đoạn code mới ra đời và xịn sò hơn
```
momentTz(start_date).tz("Asia/Ho_Chi_Minh").toDate().setHours(start_time[0].hour, start_time[0].min)
```

Ý tưởng là mình sẽ chuyển sang múi giờ +7 sau đó mới set giờ phút lại, nhưng mình muốn dùng prototype của kiểu Date trong JS nên cần dùng `toDate`.

Thật thông minh, mình nghĩ thế là xong, ổn thỏa hết rồi.

Run thôi nào !!!

Tạch tạch tạch ...

KẾT QUẢ VẪN SAI KHÔNG ĐỔI ~~

Thời gian mình nhận được vẫn như cũ, nhưng why - tại sao mình thay đổi rồi mà vẫn ko đúng được. Rồi mình mất hơn 1 tiếng đồng hồ để mò mẫm đủ các trang, làm sao để set timezone trong JS, nhưng vẫn bế tắc.

Mình quyết định đứng dậy, uống ngụm nước, nhắm mắt và chất vấn bản thân. Cuối cùng, cũng có một ánh sáng lóe lên, có vẻ như múi giờ của máy sẽ ảnh hưởng đến date object trong JS.

Thế là mình đổi múi giờ của máy mình sang `UTC`, và yeah, mình tái hiện được y nguyên lỗi đang gặp phải. 

Và chân lý chính là đây, khi mình dùng `moment-timezone` chuyển từ múi giờ UTC sang +7, nó đã làm rất tốt (mình xin lỗi vì nghi ngờ năng lực của *moment*), nhưng bước chuyển lại Date Object của JS là 1 nước đi ngu ngốc, rất ngu ngốc của mình. Date Object nhận được sẽ lấy múi giờ của hệ thống và lại đưa giờ đã được chuyển về như cũ
```
|18/7/2021 17:00:00 (UTC) |=>| 19/7/2021 00:00:00 (+7)     | => |18/7/2021 17:00:00 (UTC)    |
----------------------------------------------------------------------------------------------------
|        Lấy từ DB        |=>| Dùng moment để set timezone | => |sau khi dùng method toDate()|
```

Khi đã ngộ ra chân lý thì mọi việc còn lại quá đơn giản, mình chọn cách là set giờ dùng method của moment luôn sau đó hẳn đưa về date trong js. Vậy là xong !!!

```
momentTz(start_date).tz("Asia/Ho_Chi_Minh").set('hour', start_time[0].hour).set('minute', start_time[0].min).toDate()
```

### Tóm lại

Chỉ muốn khoe với mọi người là mình giải quyết vấn đề giỏi quá :)) À không nhầm, tóm lại là Date object trong JS sẽ lấy múi giờ dựa trên thông tin timezone trên máy tính đang chạy, và để tính toán giờ thì nên đưa về cùng timezone.

`Bye bye, have a nice day!`
