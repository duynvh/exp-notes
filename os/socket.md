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

**First remember below two rules:**
1.  Primary key of a socket: A socket is identified by `{SRC-IP, SRC-PORT, DEST-IP, DEST-PORT, PROTOCOL}` not by `{SRC-IP, SRC-PORT, DEST-IP, DEST-PORT}` - Protocol is an important part of a socket's definition.
2.  OS Process & Socket mapping: A process can be associated with (can open/can listen to) multiple sockets which might be obvious to many readers.

**Example 1:** Two clients connecting to same server port means: `socket1 {SRC-A, 100, DEST-X,80, TCP}` and `socket2{SRC-B, 100, DEST-X,80, TCP}`. This means host A connects to server X's port 80 and another host B also connects to same server X to the same port 80. Now, how the server handles these two sockets depends on if the server is single threaded or multiple threaded (I'll explain this later). What is important is that one server can listen to multiple sockets simultaneously.

**To answer the original question of the post:**

Irrespective of stateful or stateless protocols, two clients can connect to same server port because for each client we can assign a different socket (as client IP will definitely differ). Same client can also have two sockets connecting to same server port - since such sockets differ by `SRC-PORT`. With all fairness, "Borealid" essentially mentioned the same correct answer but the reference to state-less/full was kind of unnecessary/confusing.

To answer the second part of the question on how a server knows which socket to answer. First understand that for a single server process that is listening to same port, there could be more than one sockets (may be from same client or from different clients). Now as long as a server knows which request is associated with which socket, it can always respond to appropriate client using the same socket. Thus a server never needs to open another port in its own node than the original one on which client initially tried to connect. If any server allocates different server-ports after a socket is bound, then in my opinion the server is wasting its resource and it must be needing the client to connect again to the new port assigned.

**A bit more for completeness:**

**Example 2:** It's a very interesting question: "can two different processes on a server listen to the same port". If you do not consider protocol as one of parameter defining socket then the answer is no. This is so because we can say that in such case, a single client trying to connect to a server-port will not have any mechanism to mention which of the two listening processes the client intends to connect to. This is the same theme asserted by rule (2). However this is WRONG answer because 'protocol' is also a part of the socket definition. Thus two processes in same node can listen to same port only if they are using different protocol. For example two unrelated clients (say one is using TCP and another is using UDP) can connect and communicate to the same server node and to the same port but they must be served by two different server-processes.

**Server Types - single & multiple:**

When a server's processes listening to a port that means multiple sockets can simultaneously connect and communicate with the same server-process. If a server uses only a single child-process to serve all the sockets then the server is called single-process/threaded and if the server uses many sub-processes to serve each socket by one sub-process then the server is called multi-process/threaded server. Note that irrespective of the server's type a server can/should always uses the same initial socket to respond back (no need to allocate another server-port).

Suggested [Books](https://rads.stackoverflow.com/amzn/click/com/0130183806) and rest of the two volumes if you can.

**A Note on Parent/Child Process (in response to query/comment of 'Ioan Alexandru Cucu')**

Wherever I mentioned any concept in relation to two processes say A and B, consider that they are not related by parent child relationship. OS's (especially UNIX) by design allow a child process to inherit all File-descriptors (FD) from parents. Thus all the sockets (in UNIX like OS are also part of FD) that a process A listening to, can be listened by many more processes A1, A2, .. as long as they are related by parent-child relation to A. But an independent process B (i.e. having no parent-child relation to A) cannot listen to same socket. In addition, also note that this rule of disallowing two independent processes to listen to same socket lies on an OS (or its network libraries) and by far it's obeyed by most OS's. However, one can create own OS which can very well violate this restrictions.

Đầu tiên là một tình huống đơn giản, khi một client gọi request đến server thì nó sẽ cần làm gì?
- 

References:
- http://www.steves-internet-guide.com/tcpip-ports-sockets/
- https://community.cisco.com/t5/application-networking/difference-between-session-connections-socket/td-p/2417074
- https://stackoverflow.com/questions/3329641/how-do-multiple-clients-connect-simultaneously-to-one-port-say-80-on-a-server#:~:text=Irrespective%20of%20stateful%20or%20stateless,sockets%20differ%20by%20SRC%2DPORT%20.
- https://medium.com/swlh/looking-under-the-hood-http-over-tcp-sockets-952a944c99da