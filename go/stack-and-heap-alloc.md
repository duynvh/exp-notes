# Stack and heap allocation

> Source: https://www.youtube.com/watch?v=ZMZpH4yT7M0

### Câu hỏi là: Khi nào thì Go sẽ đặt biến trong Stack, khi nào sẽ để trong Heap?

-> TLDR: Khi nào compiler có thể chứng minh là biến đó không còn được dùng sau khi function return thì nó sẽ để trong stack.

- Trong Go thì mỗi chương trình sẽ có nhiều stack memory (mỗi stack cho một goroutine) và heap memory

- Việc biết biến được lưu ở stack hay heap có quan trọng:
	- Nó không ảnh hưởng tới việc chương trình chạy đúng hay không.
	- Nhưng nó ảnh hưởng tới performance của ứng dụng.
	- Bất cứ gì ở trong Heap được quản lí bởi `Garbage Collector` (GC).
	- GC của Go rất xịn, nhưng nó có độ trễ, và với những ứng dụng cần hiệu năng cao, cần chú ý tới lượng garbage mà bạn đặt trong heap.
	
- Ví dụ 1:
```go
func main() {
	a := 1
	a1 := square(a)
	fmt.Println(a1)
}

func square(x int) int {
	return x * x
}
```
- Giải thích step by step:
	- Ban đầu chương trình sẽ khởi tạo `a = 1` và `a1 = 0` trong stack frame (stack frame là bộ nhớ dùng cho một function)
	- Khi thực thi square function thì sẽ tạo một thêm stack frame chứa biến x
	- Khi thực thi xong và return thì cập nhật `a1` ở `main` và remove stack framce của square, và phần memory đó sẽ được reuse.
	- Khi chạy fmt.Println thì Go lại tạo một stack frame cho việc này

- Ví dụ 2: Stack với pointers
```go
func main() {
	a := 1
	inc(&a)
	fmt.Println(a)
}

func inc(x *int) {
	*x++
}
```
- Giải thích:
	- Tương tự như vd1 thì chương trình vẫn tạo stack framce cho hàm main và cấp phát bộ nhớ cho biến `a`
	- Thay vì stackframe ở function inc lưu giá trị như ở vd1 thì nó lưu địa chỉ của biến x, sau đó thay đổi giá trị mà nó trỏ tới (biến a).

-> Ta có thể thấy là `sharing down typically stays in the stack`, nghĩa là khi truyền biến từ ngoài vào trong thì thường tất cả các biến đó đều nằm trong stack

- Ví dụ 3: trả về một pointer
```go
func main() {
	n := answer()
	fmt.Println(*n/2)
}

func answer() *int {
	x := 2
	return &x
}
```
- Giải thích:
	- Đầu tiên vẫn là stackframe cho main, n sẽ khởi tạo là nil
	- Sau đó tới lượt stackframe cho function answer, biến x được gán giá trị bằng 2
	- Ở đây, khi function answer return, Go đã không còn thu hồi stack frame của nó, nhưng ở main thì biến n vẫn trỏ tới biến x trong answer. Việc này dẫn đến một problem lớn.
	- Thực tế, Go đã xử lí việc này bằng cách đưa x chứa trong Heap. Khi compiler đọc đến function answer, nó biết là không an toàn nếu chứa biến x trong stack, và nó escape x sang heap, việc này xảy ra ở compile time chứ không phải runtime.
	
-> Và kết luận thứ 2 là `sharing up typically escapes to the heap`, nghĩa là khi một biến ở trong một function được truyền ra function cấp cao hơn (bằng pointers) thì thường nó sẽ được compiler đặt trong heap.

- Trên thực tế thì chúng ta không thể chắc chắn được biến sẽ được allocate ở Stack hay Heap, chỉ có compiler biết điều đó. Và nếu muốn biết thì đơn giản chúng ta cần đi "hỏi" nó.

- Trên trang chủ của Go: `golang.org/doc/faq#stack_or_heap`
	- Khi có thể Go compiler sẽ cấp phát biến trong function vào stackframe của function đó
	- Tuy nhiên nếu compiler không thể chứng minh được là biến đó không được tham chiếu sau khi function return, thì compiler phải cấp phát biến ở garbage-collected heap để tránh lỗi `dangling pointer errors`
	
- Và ở đây thì compiler làm một việc gọi là `Escape Analysis` để quyết định việc cấp phát trên.

### Khi nào thì giá trị được khởi tạo trong heap
1. Khi mà giá trị đó có thể được reference sau khi function tạo ra nó return.
2. Khi compiler nhận thấy giá trị đó quá lớn để lưu vừa trong stack.
3. Khi compiler không biết được kích thước của giá trị đó ở compile time.

- Và khi hiểu được những điều này thì ta có thể giải thích vì sao io.Reader interface được định nghĩa
```go
type Reader interface {
	Read (p []byte) (n int, err error)
}
// thay vì
type Reader interface {
	Read (n int) (p []byte, err error)
}
```
- Vì cách thứ hai sẽ có thể phải lưu rất nhiều garbage trong heap.
