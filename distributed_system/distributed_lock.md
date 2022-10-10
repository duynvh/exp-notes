# Distributed lock with Redis

TL,DR: sẽ dùng một thuật toán gọi là `Redlock` để implement distributed lock

3 tiêu chí cần đảm bảo khi implement distributed lock
- **Safety property**: Mutual exclusion. Tại một thời điểm thì chỉ có 1 client giữ lock
- **Liveness property A**: Deadlock free. Luôn luôn có thể acquire lock dù là client lấy lock có bị crash hoặc partitioned.
- **Liveness property B**: Fault tolerance. Miễn là đa số Redis nodes còn sống thì client có thể acquire và release locks.

Problem: chúng ta có thể dễ dàng implement lock trên redis với việc `SET` key để acquire lock và `DEL` khi release. Nhưng phương pháp này dễ dàng gặp vấn đề khi sử dụng vs distributed system
Ví dụ về một case race condition với model này:
1. Client A acquire lock trên master
2. Master crash trước khi key được transmit tới replica
3. Replica được promote lên thành master
4. Client B acquire lock trên cùng resource mà A đang giữ lock -> **SAFETY VIOLATION!**

1. Single instance
- Đầu tiên chúng ta cần biết cách implement đúng đối với case sử dụng single instance redis trước khi tìm hiểu về Redlock cho cluster redis.
- Để acquire lock: 
```markdown
    SET resource_name my_random_value NX PX 30000
```
- random value ở đây cần unique giữa các client để đảm bảo client này sẽ không release lock của client khác (trong trường hợp client giữ lock thực hiện action lâu hơn TTL của lock, lúc này lock đã được release tự động và client khác có thể acquire lock trên cùng resource).
- Để release lock thì ta dùng một Lua script
```vbnet
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```
2. Multi instances
- Chúng ta sẽ giả sử là đang có 1 cluster redis chạy N Redis masters.
- Để lấy lock thì client sẽ làm những bước sau: 
	1.  Lấy thời gian hiện tại tính bằng ms
	2.  Cố gắng acquire lock trên tất cả N instance tuần tự, dùng cùng 1 key và random value trên tất cả instance. Trong quá trình thực hiện bước này thì khi setting lock trên mỗi instance, client dùng 1 timeout nhỏ hơn total lock auto-release time để acquire. Ví dụ, nếu auto-release time là 10s, thì timeout nên là nằm trong khoảng 5-50ms. Việc này nhằm ngăn không cho client bị block quá lâu khi cố gắng nói chuyện vs redis node đã down: nếu instance ko available thì tốt nhất nên nói chuyện vs instance tiếp theo ASAP
	3. Client tính toán xem thời gian để acquire lock, bằng cách trừ đi thời gian đã init ở bước 1. Và chỉ khi client có thể acquire lock trên đa số các instance (ít nhất N/2 + 1) và tổng thời gian để acquire lock nhỏ hơn validity time, thì lock mới được acquire thành công
	4. Nếu lock được acquire thì validity time của nó được tính bằng thời gian validity ban đầu trừ cho thời gian đã mất để lấy lock(tính ở bước 3)
	5. Nếu client không acquire được lock vì lí do nào đó (hoặc là không thể lock trên N/2 + 1 instance hoặc là validity time < 0) nó sẽ cố gắng unlock tất cả instance (cả những instance mà nó chưa thể lock)

**Notes** cho https://redis.io/docs/reference/patterns/distributed-locks/#safety-arguments
-  Trong trường hợp 2 client cùng acquire lock tại 1 thời điểm
	- Client 1 acquire lock trên node A, B, C
	- Client 2 acquire lock trên node C, B, A
	- Trường hợp để client 1 và 2 cùng acquire thành công n/2+1 = 2 node là khi Client 1 acquire lock ở C thì lock ở A expire do hết TTL, nếu vậy thì `MIN_VALIDITY=TTL-(T2-T1)-CLOCK_DRIFT` sẽ <0 -> lock sẽ invalid -> sẽ không thể acquire lock cùng lúc trên 1 resource, và safe property được đảm bảo.

Resources:
- https://redis.io/docs/reference/patterns/distributed-locks/#safety-arguments