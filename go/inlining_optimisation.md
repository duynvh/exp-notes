# Inlining Optimisations

1. Inlining là gì?
 - Inlining là việc đưa những functions nhỏ hơn vào trong những caller tương ứng của chúng. Trước đây thì việc tối ưu này thường được làm thủ công, còn bây giờ thì nó là một trong những tối ưu nền tảng được thực hiện tự động trong quá trình compile.

2. Tại sao cần inlining optimisations?
- Inlining is important for two reasons. The first is it removes the overhead of the function call itself. The second is it permits the compiler to more effectively apply other optimisation strategies.
- Inlining quan trọng là vì 2 lí do
	- Đầu tiên là nó xoá bỏ overhead khi call function
	- Thứ hai là nó cho phép compiler áp dụng những optimisation strategies khác hiệu quả hơn

3. Inlining trong Go
- Trong Go thì mặc định compiler sẽ tối ưu inlining cho chúng ta, để test thử xem inlining tối ưu code như thế nào thì chúng ta dùng một pragma `//go:noinline` để ngăn compiler inlining tự động một function
```go
package main

import "testing"

//go:noinline
func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}

var Result int

func BenchmarkMax(b *testing.B) {
    var r int
    for i := 0; i < b.N; i++ {
        r = max(-1, i)
    }
    Result = r
}
```
- Chúng ta sẽ chạy lệnh `go test -bench=.`
> BenchmarkMax-8          509964764                2.023 ns/op
- Mình chạy trên máy MacAir M1 thì cost là 2.023ns, bây giờ thì mình sẽ bỏ đoạn `go:noinline` để xem việc tối ưu inlining sẽ cho kết quả như thế nào
```go
package main

import "testing"

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}

var Result int

func BenchmarkMax(b *testing.B) {
    var r int
    for i := 0; i < b.N; i++ {
        r = max(-1, i)
    }
    Result = r
}
```
- Và kết quả là
> BenchmarkMax-8          1000000000               0.3185 ns/op

- Vậy thì do đâu mà hiệu năng được cải thiện như thế này?
- Đầu tiên là xoá đi function call và associated preamble. Đưa nội dung của hàm `max` vào bên trong caller để giảm số lượng instructions được thực hiện bởi processor và bỏ đi một vài branches.
- Và lúc này thì nội dung của hàm `max` visible đối với compiler nên nó có thể thực hiện thêm một vài tối ưu khác.
- Giờ chúng ta hãy thử làm thủ công việc này, đưa nội dung hàm `max` vào bên trong calller
```go
package main

import "testing"

var Result int

func BenchmarkMax(b *testing.B) {
    var r int
    for i := 0; i < b.N; i++ {
	    if -1 > i {
	        r = -1
	    } else {
		    r = i
	    }
    }
    Result = r
}
```

- Và kết quả là 
> BenchmarkMax-8          1000000000               0.3180 ns/op

- Và khi đã inlining, thì compiler có thể thấy là `-1 > i` sẽ không thể xảy ra (do giá trị khởi tạo i là 0, và sẽ tăng) nên function trên sẽ được improve lần nữa
```go
package main

import "testing"

var Result int

func BenchmarkMax(b *testing.B) {
    var r int
    for i := 0; i < b.N; i++ {
		r = i
	    
    }
    Result = r
}
```

- Và quá trình inlining mà chúng ta nói đến ở đây là `leaf inlining`, là inlining được thực hiện ở bottom của call stack vào caller trực tiếp.
- Còn một loại inlining nữa là `middle-stack inlining`

**Resources:**
- https://dave.cheney.net/2020/04/25/inlining-optimisations-in-go