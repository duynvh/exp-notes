# Service mesh

- Nguồn: https://buoyant.io/service-mesh-manifesto/

![Service mesh architecture](https://buoyant.io/images/manifesto/diag1.png)

- Một vài khái niệm:
	- Service mesh chia là 2 phần là control plan và data plane
	- Control plane là những process để quản lí, còn data plan là những service mesh's proxies gắn với app.
	- Proxy: xử lí các outgoing calls
	- Reverse proxy: xử lí incoming calls
	- Service mesh proxies: hoạt động giống như vừa là "proxies" vừa là "reverse proxies". Khác với những proxies khác như API gateways hay ingress proxies, vì những proxies này chỉ tập trung vào các calls từ bên ngoài vào trong cluster.
	- Control plane cung cấp những thứ mà data plane cần như là service discovery, TLS certificate, metrics aggregations, ...
	![Control vs Data Plane](https://buoyant.io/images/manifesto/control-plane.png)
	
- Service mesh sinh ra để phục vụ cho service-to-service calls, nên chỉ phù hợp nếu hệ thống của bạn được xây dựng dựa trên nhiều services.

- Vì service mesh sẽ yêu cầu rất nhiều proxies, và với mỗi call bây giờ sẽ phải đi qua thêm 2 proxies nữa (ở client chiều ra, và ở server chiều vào), nên proxies phải đảm bảo:
	- Phải nhanh
	- Phải nhỏ và nhẹ
	- Tự động

### Tại sao phải dùng service mesh?
- Đầu tiên là cost để deploy những cái proxies này được cải tiến và gần như không đáng kể
- Thứ hai và quan trọng nhất là đây là một cách rất tốt để thêm logic vào hệ thống. Và không chỉ những tính năng được cung cấp, mà còn vì bạn có thể thêm vào mà không thay đổi vì hệ thống.

> that, in a multi-service system, regardless of what individual services actually do, the traffic between them is an ideal insertion point for functionality.

- Một vài tính năng chính trên HTTP calls, chia làm 3 loại:
	- `Reliability features`: request retries, timeouts, canaries (traffic splitting/shifting) (liên quan đến deployment, chia traffic từ từ tới những endpoint mới deploy trước khi tắt endpoint ver cũ) ...
	- `Observability features`: Tổng hợp success rate, latencies, request volume cho mỗi service, hoặc là những route cụ thể, ...
	- `Security features`: TLS, access control, ...

- Lí do vì sao nên dùng service mesh thay vì implement trong ứng dụng là 
```
The service mesh gives you features that are critical for running modern server-side software in a way that’s uniform across your stack and decoupled from application code.
```
"Service mesh cung cấp những tính năng cần thiết cho một phần mềm server-side đang chạy một cách đồng nhất với stack của bạn và tách biệt với application code"


