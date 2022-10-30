# Issue with time.After and time.Tick
https://pkg.go.dev/time#After
>After waits for the duration to elapse and then sends the current time on the returned channel. It is equivalent to NewTimer(d).C. The underlying Timer is not recovered by the garbage collector until the timer fires. If efficiency is a concern, use NewTimer instead and call Timer.Stop if the timer is no longer needed.

Trong docs trên trang chủ Go có nói tới việc func time.After sẽ trả về 1 channel time.Time và nó sẽ được fire sau khi đợi đủ thời gian `timeout`. Vấn đề ở đây là bằng một cách nào đó, mà function kết thúc trước khi timer fire như vd sau

```go
timer := time.After(10 * time.Second)

select {
case <-timer:
	// do A
case <-ch:
	// do B
}
```

Nếu hàm chạy vào `do B` thì chương trình sẽ bị leak `timer`

**Giải pháp** tốt nhất là không xài `time.After`, thay vào đó thì chúng ta sẽ dùng `NewTimer` để thực hiện công việc tương tự

```go
timer := time.NewTimer(10 * time.Second)
defer timer.Stop()
timerCh := timer.C

select {
case <-timer.C:
	// do A
case <-ch:
	// do B
}
```

Với cách này thì khi function kết thúc thì timer cũng sẽ được release vì đã gọi `timer.Stop()`

Tương tự vs `time.Tick` thì nên xài `time.NewTicker` và `ticker.Stop` khi cần release.