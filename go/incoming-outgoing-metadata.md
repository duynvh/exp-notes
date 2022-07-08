# Metadata trong incoming and outgoing context

Note này để giúp bạn định nghĩa được incoming và outgoing metadata và cách sử dụng chúng như thế nào

Ví dụ: 

Bạn đang viết một gRPC client cần gửi request lên server và muốn gửi kèm token trong metadata. Lúc này thì cần attach token vào metadata trong context của outgoing request
```go
ctx := metadata.AppendToOutgoingContext(ctx, "authorization", "token")
```

Còn ở phía server, để extract metadata từ request của client thì cần lấy từ incoming request
```go
md, ok := metadata.FromIncomingContext(ctx)
```

Link chi tiết về package `metadata` ở [đây](https://pkg.go.dev/google.golang.org/grpc/metadata?utm_source=godoc)

Ok vậy giờ cứ follow the rule này để xài in hay out cho khỏe nhá!
