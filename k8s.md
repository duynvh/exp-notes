# K8s

1. Master node là 1 single point of failture của k8s
- Nó là nơi lưu trữ dữ liệu dạng key-value
- Lưu trữ và replicate trạng thái của cluster

2. API Server
- Là một control plane frontend để các công cụ khác làm việc vs k8s thông qua command, ...
- Có thể scale horizontally (mở rộng theo chiều ngang)

3. Scheduler
- Dùng để tìm ra Node thích hợp nhất để tạo Pod mới

- Những Node đang có sẽ được chọn lọc dựa trên những yêu cầu của Scheduler

4. Controller Manager

- Có nhiệm vụ thay đổi để move từ trạng thái hiện tại sang trạng thái mong muốn.

- Nó sẽ thực hiện một số controller process:
	- Node Controller: Kiểm tra `Nodes` và thực hiện action khi chúng bị down.
	- Replication Controller: Đảm bảo số lượng Pods mong muốn luôn up và available
	- Endpoint Controller: phân bổ `endpoint` object (nhiệm vụ connect Services và Pods)
	- Service Account và Token Controller: tạo những tài khoản mặc định và API access tokens cho những namespaces mới.
	
5. Kubelet

- Quản lí trạng thái thực thi trên mỗi Node
- Kiểm tra tất cả các container của Node có healthy ko
- Có thể tạo và xóa Pods
- Mỗi `Node` thì có 1 instance của `Kubelet`

6. Proxy

- Chạy trên mỗi Node
- Allow network communication cho Pods

---

### Kubeconfig
- Có thể load file config từ:
	- chỉ định `--kubeconfig` flag
	- set biến môi trường `KUBECONFIG`
	- hoặc mặc định ở `$HOME/.kube/config
	
---

### Namespaces
- Dùng để isolate cho mỗi project, mỗi team, mỗi tập các component.
- Resource name phải unique trong 1 namespace
- Không phải tất cả resource đều nằm trong namespace, vd: Node, PV, ...
- Secret chỉ thuộc về 1 namespace, pod ở ns khác không thể truy cập
- Một vài ns đặc biệt:
	- default: mặc định
	- kube-system: cho những object được tạo bởi k8s
	- kube-public: dùng chủ yếu cho cluster, và trường hợp một resource nào đó cần public
	- kube-node-lease: Dùng cho heartbeat Node's lease object
	
---

### Liveness and Readiness Probe

- Liveness dùng để check xem pod đó còn sống ko, nếu ko thì sẽ restart pod.

-Readiness dùng để kiểm tra khi nào pod sẵn sàng để nhận traffic, nếu chưa thì sẽ ko link service tới pod.

---

### Docker registry
```
$ kubectl create secret docker-registry registry-secret
--docker-server=<host>:8500
--docker-username=<user_name>
--docker-password=<user_password>
--docker-email=<user_email>

// usage
....
spec:
	containers:
	- name: my-container
	  image: my-image
	  imagePullPolicy: Always
	imagePullSecrets:
	- name: registry-secret
```

- k8s sẽ dùng config này để biết nên dùng secret nào để access vào image registry (pull những image private)


---

