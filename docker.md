# Docker

Bài viết này mình sẽ tạm xem như bạn đọc đã nắm được những concept căn bản của docker, nếu như chưa thì có thể xem docs trên trang chủ https://docs.docker.com/ nhé!

Hôm nay mình sẽ điểm qua một vài điều, khiến docker trở thành một phần rất hữu ích trong quá trình development cũng như lúc operations.

1. Image giống như là 1 cái bánh nhiều lớp
- Image trong docker được tạo bởi nhiều layers.
- Layer là read-only/immutable.

- Và những layer này có thể được share giữa nhiều image khác nhau, vì thế mọi người có thể để ý là cùng một image nếu ta chỉnh sửa một chút trong `Dockerfile` và build lại thì thời gian build sẽ nhanh hơn rất nhiều, vì nó có thể tái sử dụng lại các layer không bị thay đổi.

- Không phải tất cả step trong Dockerfile đều tạo layer, chỉ có các command `RUN`, `ADD`, `COPY` mới tạo layer, những step còn lại chỉ là những layer trung gian.

- Có thể list tất cả layer của image qua lệnh sau:
```
docker history my-image --no-trunc
```

- Và các layer xếp chồng lên nhau nên hãy cố gắng tạo file `Dockerfile` theo step từ ít thay đổi đến thay đổi nhiều (dĩ nhiên là vẫn phải tuân theo flow để có thể chạy được 1 container hoàn chỉnh nhé).

- Những layer phụ thuộc bên thứ ba sẽ không cần phải download lại nếu nó đã có ở local.

2. Có thể build image từ 1 Dockerfile trên github mà không cần clone về
```
docker build https://github.com/scraly/gcp-cloud-run-demo.git#main -f helloworld/Dockerfile
```

3. Giới hạn push image lên 1 register
- Mặc định khi push 1 image lên một container register bất kì thì mặc định sẽ push 5 layer cùng lúc.

- Có thể setting bằng options `--max-concurrent-uploads`.

4. docker run = docker create + docker start

5. options
- `-i/--interactive`: cho phép bạn tương tác / chạy những câu lệnh trong container
- `t/--tty`: attach 1 cái remote console tới container

-> Với 2 options này thì container sẽ không bị terminate

- `-d/-d=true`: detach mode, local terminal sẽ ko được attach vào container đang chạy 

- `-P`: publish all exposed ports

6. Docker copy file
- Dùng lệnh sau để copy file từ container ra máy host:
```
docker cp container:xx myfile
```

- Dùng lệnh sau để copy file từ máy host vào container:
```
docker cp myfile container:xx
```

7. Docker stop
- Dùng để stop 1 running container
- Sẽ thực hiện các bước sau:
```
1. Gửi SIGTERM signal
2. Chờ 10s (mặc định)
3. Gửi SIGKILL signal
```

8. Volume
- Volume ko làm tăng size của container

- Volume được lưu ở `/var/lib/docker/volumes`

- Có thể attach volume của container đang chạy vào 1 container khác
```
docker run -it --volumes-from my-vol my-other-container /bin/bash
```

- Có thể set permission cho volume là read-only
```
docker run -d -v nginx-vol:/usr/share/nginx/html:ro --name nginx-vol nginx:lastest
```

9. Events
- Real-time events from server
- Mặc định thì 1000 log events được hiển thị
- Có thể hiển thị theo thời gian

- Hiển thị event trong vòng 5 phút
```
docker events --since '5m'

// Show in JSON
docker events --format '{{json .}}'

// Show stop event
docker events --filter 'event=stop'
```

10. Tìm image trên CLI
```
docker search ubuntu

// search with filter more than 10 stars and limit 5
docker search --filter=stars=10 --limit=5 debian

// List and format response
docker search --format "{{.Name}}\t{{.StarCount}}\t{{.IsOfficial}}" nginx
```


11. Scan image
- Scan một local image
```
docker scan my-image
```

- Scan chi tiết
```
docker scan my-image --file Dockerfile

// Hiển thị dependencies
docker scan my-image --dependency-tree

// Chỉ hiện thị vulnerabilities trong code
docker scan my-image --exclude-base
```

12. Command để remove tất cả images, container, volume, network
```
docker system prune -a
```

13. Giới hạn memory và cpu
- Mặc định container sẽ có thể dùng tất cả memory có sẵn của máy host.

- Khi bạn define limitation cho memory thì container có thể `swap` một lượng tương ứng
```
--memory 128
// memory: 128 và swap: 128 -> thực tế mất 256MB

docker run -m 128m my-image

// no limit for swap
docker run -m 128m --memory-swap -1 my-image
```

- *Best practice*: Set max memory và swap limit bằng nhau -> khi đó container sẽ ko swap.

- --memory sẽ giới hạn memory sử dụng trong container, nhưng con số này không bằng vs lượng memory khi mà chúng ta chạy lệnh `free` trong container.


14. OOM
- OOM: out of memory killer
- OOM sẽ xảy ra khi không còn memory trong system có thể dùng
- Khi đó sẽ kill những process nào tốn nhiều memory nhất cho đến khi hệ thống được khôi phục lại đủ resource để chạy tiếp
- Có thể dùng lệnh sau để deactive OOM
```
docker run -m 128m --oom-killer-disable my-image
```

15. CPU limitations
- Mặc định thì container sẽ không giới hạn CPU
```
// limit 1 cpu
docker run --cpus 1 my-image

// limit 50% of available CPU
docker run --cpus .5 my-image

// chia container sử dụng những cpu vật lí khác nhau, bắt đầu từ 0
docker run --cpuset-cpus 1,2 my-image // list 1,2
docker run --cpuset-cpus 1-3 my-image // range 1 to 3

```

16. Tips build Dockerfile

- CMD: có 2 cách dùng: string syntax và JSON syntax
- Nếu dùng string syntax thì câu lệnh sẽ được thực thi bởi shell, nhưng ko phải tất cả image đều có shell nên có thể dẫn đến lỗi.
```
CMD ./hello
```
- Vì vậy chúng ta nên dùng JSON syntax để tránh những lỗi như thế này
```
CMD ["./hello"]
```

- Chỉ định ít nhất 1 CMD hoặc ENTRYPOINT trong Dockerfile

- Nếu chỉ dùng CMD: định nghĩa executable và parameter cho container, và khi đó ENTRYPOINT mặc định là `/bin/sh -c`

- Nếu dùng ENTRYPOINT vs CMD thì 
	- ENTRYPOINT sẽ định nghĩa executable
	- CMD sẽ định nghĩa parameters
	```
	ENTRYPOINT ["./redis-tools"]
	CMD ["--help"]
	```

- Dùng `ARG` để tránh hardcode trong Dockerfile
```
ARG ENV=prod
...

// docker command
docker build --build-arg ENV=dev .
```

17. Debug with docker

```
// Logs
docker logs my-container -f

// Exec
docker exec -it my-container bash
$ tail -f /var/log/access_log

// list published port
docker port my-container

// display running process
docker top my-container

// get memory & cpu usage
docker stats my-container

// Get ip of container
docker inspect --format '{{NetworkSettings.IPAddress}}' my-container

// View detail
docker inspect my-container

// Run with debug mode
docker run -D xxx

```