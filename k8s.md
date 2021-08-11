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




