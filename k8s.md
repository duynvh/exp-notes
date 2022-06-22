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

### Daemon Set
- Đảm bảo những Nodes chạy 1 bản copy của Pod bởi Daemon Set
- Scheduler ko quản lí những Pods này mà để việc này cho Daemon Set

- Nếu 1 Node được thêm vào cluster thì DaemonSet Controller sẽ tạo 1 Pod trên nó.

- Những chiến lược (strategies) cập nhật:
	- *Rolling Update*: đây là strategy mặc định, những Pods cũ sẽ được tự động kill và Pods mới sẽ được tạo, 1 Pod mỗi Node.
	- *on Delete*: những Pods cũ phải được xóa thủ công để cho Pods mới được tạo

- Problem có thể xảy ra: *DaemonSet muốn tạo Pods trên Node nhưng Node này ko có đủ resource nữa*
-> Khi đó thì Pod sẽ bị stuck ở `Pending` state.

- Solution:
	- List tất cả Nodes trong cluster: `kubectl get nodes`
	- Tìm những Nodes có Pods của bạn: `kubectl get pods -l name=my-app -o wide -n my-ns`
	- So sánh danh sách trên để tìm ra Node gặp vấn đề
	- Xóa thủ công Pods ko được kiểm soát bởi DaemonSet
	
---

### Service
- Dùng để nối IP và 1 unique DNS name cho một group các pods 
- Cho phép truy cập 1 ứng dụng (trên nhiều pods) với 1 IP duy nhất.

- Các loại services:
	- *ClusterIP*: expose service trong nội bộ cluster, chỉ truy cập được bên trong cluster.
	- *NodePort*: exposes trên mỗi IP của Node, có thể truy cập từ bên ngoài của cluster
	- *Load Balancer*: Tạo ra 1 external Load balancer, cho phép truy cập từ bên ngoài, gán một external IP cố định (chỉ để quản lí cluster và nếu cloud support).
	- *Headless*: không thực sự là một service type, nhưng hữu ích để làm 1 interface cho những cơ chế service discovery khác, expose tất cả replicas như là 1 DNS entry.
	
---

### Ingress
- Cho phép từ bên ngoài truy cập vào cluster của bạn
- Tránh tạo Load Balancer cho mỗi service
- Một external endpoint (bảo mật vs SSL/TLS)

- Một Ingress được implement bởi 1 third party:
	- một Ingress Controller: có thể extend specs để support thêm nhiều tính năng
- Hợp nhất nhiều routes trong 1 single resource.

---

### RBAC

- Những phương thức để phân quyền dựa trên resources trong k8s
- Roles: permissions trong 1 namespace
- ClusterRole: permissions trong cluster
- RoleBinding: link một role và 1 subject, dùng để phân quyền trong 1 role nào đó tới một hoặc một tập user trong namespace.
- ClusterRoleBinding: tương tự ở trên

---

### Node and Pod Affinity
- Có thể chỉ định việc Pod chỉ chạy trên những Node có label A, B.
- Có thể chỉ định rule để 2 Pod phải chạy cùng 1 Node.
- Có thể chỉ định rule để các Pod ko đc run trên cùng Node.

---

### Node
- Có thể pause Node để k nhận workload nữa, ko schedule Pod vào Node nữa ...

---

### Debugging/Troubleshooting

1. Show thêm thông tin về pod
```
kubectl get pods
kubectl describe pods my-pod
```

2. Khi Pod bị stuck ở Pending status
- Solution:
	- Có thể resource limit > cluster capacities
	- Xóa Pods và dọn dẹp những tài nguyên ko sử dụng
	- Thêm Node capacities
	- Thêm nhiều Node
	
3. Xem error message của cluster
- Hiển thị tất cả event trong namespace
```
kubectl get events --sort-by=.metadata.creationTimestamp -n my-ns
```

- Hiển thị tất cả warning messages
```
kubectl get events --field-selector type=Warning
```

4. Pod stuck ở Waiting/ImagePullBackOff status
- Cần đặt câu hỏi:
	- Image name, tag, URL có đúng chưa?
	- Image có tồn tại trong registry không?
	- Bạn có thể pull image đó ko?
	- Kubernetes có quyền pull image ko?
	- ImagePullSecret có được config cho secured registry chưa?

5. Kubectl Debug
- Chạy một container nữa bên cạnh Pod cần debug
- Trong container này có thể chạy `curl`, `wget`, ... bên trong mt k8s
- Yêu cầu: cần active feature EphemeralContainer=true
```
kubectl alpha debug -it my-pod --image=busybox --target=my-pod --container=my-debug-container
```

6. Pod bị restart quá nhiều lần
- Nếu Pod dùng quá nhiều Memory thì có thể `OOM Killer` có thể hủy nó.
- Solution: define request and limit memory

7. Nếu Pod ko thể start
- Khi mà bạn link Pod vs ConfigMap hoặc Secret thì cần chú ý tạo linked resources trước cái name mà bạn dùng cho linked resources.

8. Pod đang running những ko hiểu sao lại ko hoạt động?
- Có thể đơn giản là vô xem logs
```
kubectl logs my-pod
```

9. Container cứ restart mãi
- LivenessProbe dùng để báo rằng container đang healthy, nhưng nếu thiếu config gì ở container sẽ làm nó nhầm là container unhealthy
- Có thể vấn đề là:
	- Probe target có truy cập được không?
	- Application cần quá nhiều thời gian để chạy
	
10. Muốn truy cập Pod mà k cần LB
- Có thể mount 1 tunnel giữa Pod và máy bạn
```
kubectl port-forward my-pod 8080
```
- Và mount tunnel trực tiếp tới Service
```
kubectl port-forward svc/my-svc 5050:5555
```
- Có thể thử curl ở local
```
curl localhost:8080/my-api
```

---
## Termination of Pods and Containers
- Khi một pod được terminate thì nó sẽ gửi đến container một TERM signal.
- Và nếu container có xử lí signal này hoặc xử lí hook `PreStop` thì thời gian xử lí tối đa dựa theo setting `terminationGracePeriodSeconds`, nếu quá thời gian setting thì pod cũng sẽ bị terminate chứ không đợi xử lí xong.
- Trong lúc pod đang ở trạng thái terminating (graceful shutdown) thì control plane sẽ ko trỏ endpoint tới những pod đó nữa.
- References:
	- https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/
	- https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination 
