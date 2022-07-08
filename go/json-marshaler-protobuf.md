# JSON Marshaler for protobuf

Câu chuyện là mình đang dùng 1 plugin của `protoc` là `gRPC-gateway` với mục đích là generate ra một proxy server để serve RESTful HTTP API cho một gRPC service.

gRPC Gateway sẽ giúp bạn chỉ cần viết code cho gRPC và nó sẽ tự động generate code để tạo ra RESTful API theo những annotation bạn đã định nghĩa trong proto file.

Bạn có thể tham khảo thêm về gRPC Gateway ở [đây](https://grpc-ecosystem.github.io/grpc-gateway/)

Một ngày mình cần định nghĩa 1 REST API sử dụng type được generate từ proto nhưng không được generate tự động từ plugin. Mình vẫn làm như bình thường cho đến bước này
```go
...
data, err := json.Marshal(resp)
...
writer.Header().Set("Content-Type", "application/json")
_, _ = writer.Write(data)
```

Mình nhận thấy mọi thứ vẫn hoạt động bình thường, ok vậy là xong việc.

Nhưng rồi một ngày, ông anh front-end bảo với mình, sao response trả về data type không giống như khi trước. Mình thấy khá bất ngờ, vì mình xài nguyên đống proto cũ, sao lại khác type được.

Mình kiểm tra đi kiểm tra lại code vẫn không thấy vấn đề nằm đâu. Thế là nhờ phía front-end check xem bị mismatch type chỗ nào, và thế là mình tìm được vấn đề.

Trong protobuf ver 3 thì đối với type int64 khi convert sang json thì nó sẽ được chuyển thành string (xem thêm ở [đây](https://developers.google.com/protocol-buffers/docs/proto3#json)). Đó là lí do mà type của mình không giống như khi xài plugin, vì mình đang parse bằng `json.Marshal` thì nó vẫn follow theo type đã được define thôi mà.

Sau một hồi lục lọi thì mình cũng tìm được chân ái
```go
var jsonMarshaler = jsonpb.Marshaler{EmitDefaults: true, OrigName: true}
...
data, err := jsonMarshaler.MarshalToString(resp)
...
if dataS != "" {
	dataRes = json.RawMessage(data)
}
writer.Header().Set("Content-Type", "application/json")
_, _ = writer.Write(dataRes)
```

Có thể tham khảo thêm về `jsonpb` ở [đây].(https://github.com/golang/protobuf/blob/master/jsonpb/json.go)

Trích dẫn trong source `https://github.com/golang/protobuf/blob/master/jsonpb`

> Package jsonpb provides functionality to marshal and unmarshal between a protocol buffer message and JSON. It follows the specification at https://developers.google.com/protocol-buffers/docs/proto3#json.

Và rồi mình note lại vài dòng này ở đây cho ai cần giống mình nhé!