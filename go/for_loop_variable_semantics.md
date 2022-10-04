# For loop variable semantics
Current Go ver 1.19

```go
var all []*Item
for _, item := range items {
	all = append(all, &item)
}

for _, i := range all {
	fmt.Println(*i)
}

// 3
// 3
// 3
```

```

Hiện tượng ở trên xảy ra là vì biến item (hay i) là `per-loop` chứ không phải `per-iteration`, do đó `&item` là giống nhau ở mỗi lần lặp, và `item` được ghi đè lại mỗi lần lặp. Có thể fix được lỗi ở trên bằng cách sau:

```go
var all []*Item
for _, item := range items {
	item := item
	all = append(all, &item)
}

for _, i := range all {
	fmt.Println(*i)
}

// 1
// 2
// 3
```

Per-loop có nghĩa là với mỗi vòng lặp, thì chỉ có một biến được khởi tạo ra, và mỗi lần lặp thì sẽ được gán lại giá trị, nên trong trường hợp ở trên, khi dùng `&item` thì chỉ có một biến item, nên &item sẽ là không đổi, và giá trị của item thì thay đổi sau mỗi lần lặp, nên mảng `all` chứa 3 phần từ đều là pointer đến item. Và khi vòng lặp kết thúc, kết quả ta thu được là mảng chứa pointer đến cùng 1 giá trị, chính là giá trị cuối cùng trước khi kết thúc vòng lặp.

Và lỗi này cũng xảy ra tương tự với vòng for 3-clause, (và cả trong trường hợp `closure`)
```go
var prints []func()
for i := 1; i <= 3; i++ {
	prints = append(prints, func() { fmt.Println(i) })
}
for _, print := range prints {
	print()
}

// 4
// 4
// 4
```

Reference:
- https://github.com/golang/go/discussions/56010