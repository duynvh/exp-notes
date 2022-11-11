# Dính bẫy với `conn.Read`
Việc setup một TCP server trong Go khá đơn giản, chúng ta chỉ cần vài dòng code là có thể có một dead simple server rồi
```go
port := 6379  
l, err := net.Listen("tcp", fmt.Sprintf(":%d", port))  
if err != nil {  
   log.Printf("failed to bind to port %d\n", port)  
   os.Exit(1)  
}  
  
conn, err := l.Accept()  
if err != nil {  
   log.Println("error accept connection", err.Error())  
   os.Exit(1)  
}  

defer conn.Close()
for {  
   var input []byte  
   n, err := conn.Read(input)  
   if err != nil {  
      log.Println("error when read input from client", err.Error())  
      break  
   }  
   if n == 0 {  
      break  
   }  
  
   _, err = conn.Write(buildRESPString("PONG"))  
   if err != nil {  
      log.Println("failed to write to client connection", err.Error())  
   }
}
```
Nhìn sơ qua thì ta thấy chương trình không có vấn đề gì cả, lúc chạy lên cũng không có lỗi.
Giờ ta sẽ dùng một tcp client để connect tới server này nhé!
```bash
netcat localhost 6379
```

Chúng ta sẽ thấy connection sẽ được init và close ngay lập tức. Lí do là gì nhỉ?
Để hiểu rõ hơn thì mình debug và thấy chương trình sẽ chạy vào dòng này
```go
...
if n == 0 {  
  break  
} 
...
```
Nhưng mình thậm chí còn chưa gửi 1 message nào từ client, và theo mình biết thì đáng ra hàm `conn.Read()` phải block đến khi nhận được request từ client thì mới đúng chứ.

Và sau một hồi lùng sục thì mình thấy được implement của hàm `Read` là như này:
```go
func (c *conn) Read(b []byte) (int, error) {  
   if !c.ok() {  
      return 0, syscall.EINVAL  
   }  
   n, err := c.fd.Read(b)  
   if err != nil && err != io.EOF {  
      err = &OpError{Op: "read", Net: c.fd.net, Source: c.fd.laddr, Addr: c.fd.raddr, Err: err}  
   }   
   return n, err  
}
```

Ở đây thì vẫn chúng ta vẫn chưa hiểu được mình sai ở đâu 🥲, mình lại tìm tiếp vào hàm `c.fd.Read(b)`
```go
func (fd *netFD) Read(p []byte) (n int, err error) {  
   n, err = fd.pfd.Read(p)  
   runtime.KeepAlive(fd)  
   return n, wrapSyscallError(readSyscallName, err)  
}
```

Rồi lại đi tiếp vào `fd.pfd.Read(p)`
```go
func (fd *FD) Read(p []byte) (int, error) {  
   if err := fd.readLock(); err != nil {  
      return 0, err  
   }  
   defer fd.readUnlock()  
   if len(p) == 0 {  
      // If the caller wanted a zero byte read, return immediately  
      // without trying (but after acquiring the readLock).      // Otherwise syscall.Read returns 0, nil which looks like      // io.EOF.      // TODO(bradfitz): make it wait for readability? (Issue 15735)      
      return 0, nil  
   }
   ...
```

Và mọi điều đã được sáng tỏ, rõ là hàm Read sẽ block, nhưng đó là khi input `[]byte` chúng ta truyền vào phải có len > 0, nếu như ở đoạn code trên, khi khởi tạo `var input []byte` thì hàm Read sẽ return ngay lập tức, và dĩ nhiên là tcp server của chúng ta sẽ hoạt động sai bét.

Khi biết được vấn đề rồi thì chúng ta đơn giản là thay đổi một chút là mọi thứ sẽ work well
```go
...
for {  
   // initialize input with length 
   input := make([]byte, 1024)
   n, err := conn.Read(input)  
   if err != nil {  
      log.Println("error when read input from client", err.Error())  
      break  
   }  
   if n == 0 {  
      break  
   }  
  
   _, err = conn.Write(buildRESPString("PONG"))  
   if err != nil {  
      log.Println("failed to write to client connection", err.Error())  
   }
}
...
```
