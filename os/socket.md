# Socket

**Socket là gì?**
Đầu tiên thì chúng ta cần biết một điều là trên Internet, 2 thiết bị muốn nói chuyện (giao tiếp) được với nhau thì cần phải biết được `địa chỉ IP` của nhau.

Một thiết bị thì có thể chạy nhiều ứng dụng, nên cần thêm 1 cái gì đó để chỉ rõ ra là cần nói chuyện vs ứng dụng nào, lúc này chúng ta sẽ cần tới port. Nôm na là `port` dùng để xác định ứng dụng hoặc service chạy trên máy tính. 

Và socket được định nghĩa là sự kết hợp giữa địa chỉ IP và port.
Cụ thể là `{SRC-IP, SRC-PORT, DEST-IP, DEST-PORT, PROTOCOL}`

**Socket được dùng làm gì?**
Khi tạo một kết nối giữa 2 máy tính với nhau thì chúng ta xài đến `socket`
Một đầu của connection sẽ có một socket.
Ví dụ:
Your PC – **IP1**+port 60200 ——– Google IP2 +port **80** (standard port)

Sự kết hợp IP1+60200 = socket trên máy client, và **IP2 + port 80** = destination socket trên Google server.

**Connection là gì?**

Một flow 2 chiều giữa client và server gọi là 1 connection
Ví dụ:
Client Src IP(1.1.1.1) Client Port(12345)  TCP --------->  Server Src IP(2.2.2.2) Server Port (80)
Server Src IP(2.2.2.2) Server Port(80)      TCP ---------->Client Src IP(1.1.1.1.) Client Port (12345)

Thì ở trên chính là một `connection` khi bạn xem xét cả 2 flow client đến server và từ server về client.
Với flow 1 chiều như client -> server thì ta sẽ gọi là một `TCP socket`

**Khi ta nói client gửi request tới server thì bản chất bên dưới là gì?**
Chúng ta sẽ nói về 2 tình huống để hiểu rõ cách hoạt động của việc client kết nối đến server như thế nào nhé? (sẽ bỏ qua những yếu tố khác mà chúng ta chỉ tập trung vào socket nhé)

Đầu tiên là một tình huống đơn giản, khi một client gọi request đến server thì nó sẽ cần làm gì?
- Phải biết được địa chỉ IP của server, và port của service cần request để khởi tạo socket
- Sau khi đã có thông tin trên, thì kèm với thông tin địa chỉ IP của máy host (client) và một port được assign bất kì bởi OS, client có thể khởi tạo được một socket đến server
- Và dựa trên socket này thì client có thể gửi những requested packet đến server

Vậy trong trường hợp, client tạo 2 request riêng biệt đến cùng một service trên server thì sao
- Tương tự như ở trên thì ở phía client, đối với mỗi request thì sẽ có một port khác nhau được assign để tạo socket riêng
- Còn phía server, server cũng giữ 2 socket với 2 request này (vì mỗi socket được phân biệt bởi `port` khác nhau của client), service process của server có thể liên kết với nhiều socket. Và server có thể biết request nào ứng với socket nào, thì nó hoàn toàn có thể biết cách để xử lí và phản hồi request vào socket tương ứng (dựa vào thông tin trong request header chẳng hạn). Vì thế cho nên ở phía server, thì khi có nhiều request đến nó không cần phải mở thêm port để xử lí, điều này sẽ làm lãng phí resource của server.

References:
- http://www.steves-internet-guide.com/tcpip-ports-sockets/
- https://community.cisco.com/t5/application-networking/difference-between-session-connections-socket/td-p/2417074
- https://stackoverflow.com/questions/3329641/how-do-multiple-clients-connect-simultaneously-to-one-port-say-80-on-a-server#:~:text=Irrespective%20of%20stateful%20or%20stateless,sockets%20differ%20by%20SRC%2DPORT%20.