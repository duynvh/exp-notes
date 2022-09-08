# Copylock

**Vấn đề** bắt nguồn từ việc xử lí `concurrent` trong Go
Cùng xem qua ví dụ sau

```go
package main

type X struct {
	lock sync.Mutex
	data interface{}
}

func doSomething(x X) {
	x.lock.Lock()
	defer x.lock.Unlock()

	// do something here
}

func main() {
	x := X{data: "Hello"}

	doSomething(x)
}

```

Mới nhìn qua thì đoạn code trên không có vấn đề gì, nhưng mà nhìn kĩ thì sẽ thấy trong hàm `doSomething` đoạn lock sẽ không có ý nghĩa. Lí do là, khi ta truyền x vào func `doSomething` thì value của x được copy và biến lock cũng được copy nốt, nên khi call `Lock()` bên trong `doSomething` thì original value của x vẫn không có tác động gì, nên cũng không thể đảm bảo không có data race với x

**Giải pháp** là chúng ta có thể update struct thành như sau
```go
type X struct {
	lock *sync.Mutex
	data interface{}
}
```
Khi đó khi truyền vào hàm `doSomething` sẽ copy direct part value của lock, nên khi call `Lock()` thì nó vẫn sẽ affect lên original x.

-> **Bài học là không nên copy những values có chứa types được define trong sync package (https://pkg.go.dev/sync#pkg-overview)**

Và Go cũng cung cấp một tool để phát hiện việc này là `Copy Analyzer` (https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/copylock)

Có thể sử dụng command `go vet .` để kiểm tra xem chương trình của bạn có đang copy lock không nhé!