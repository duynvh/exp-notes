# TestMain 
Hôm nay thì mình cần phải viết test trong Go
Sau khi walkthrough một vài file test có sẵn trong project thì mình thắc mắc một điều:
`Làm sao mà mấy cái test này chạy được nhỉ?`
Thế là bài viết này ra đời.

Đầu tiên thì scenario của mình là mình mới thêm vài cái API CRUD, và requirement là sau khi implement xong hết logic và manual test thì phải viết test cho những API này

Problem ở đây là phải init một đống dependencies đi kèm để run được server và viết API test, mình check code thì chả thấy đoạn này ở đâu, mà sao những file này có thể chạy được ta? Thì ra tất cả phần setup này đã nằm ở func `TestMain`

Và mình lên gg search, đi tìm docs, cuối cùng thì đã có câu trả lời
https://pkg.go.dev/testing#hdr-Main
http://cs-guy.com/blog/2015/01/test-main/

```go
func TestMain(m *testing.M) {
    setup()
    code := m.Run() 
    shutdown()
    os.Exit(code)
}
```

Hiểu một cách đơn giản thì khi chạy bất cứ file test nào thì hàm này cũng sẽ được trigger chạy trước, và chúng ta có thể chạy `go test` để go tool generate một main program để chạy những test function cho package, nhưng nếu có hàm `TestMain` thì go tool sẽ generate code vào trong hàm này.