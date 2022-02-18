# Redis - RabbitMQ - Kafka

1. RabbitMQ
	- Scale: ~50k msg/s
	- Persistent: có thể có hoặc không tùy setting
	- Có cả one-to-one và one-to-many consumers

2. Kafka
	- Scale: ~1 milion msg/s
	- Persistent: yes
	- Chỉ support one-to-many

3. Redis
	- Scale: ~1 milion msg/s
	- Persistent: no, vì nó là 1 in-memory datastore
	- Hỗ trợ cả one-to-one và one-to-many
	
	- Redis không có persistency nhưng có thể dumps memory xuống disk/DB được.
	
4. Use cases
	- Short-lived messages: Redis
		- Redis rất hiệu quả trong trường hợp sử dụng với các short-lived message, khi đó persistent là không bắt buộc. Redis rất nhanh và sử dụng in-memory nên rất phù hợp với những short retention messages.
	
	- Large amounts of data: Kafka
		- Kafka là một high throughput distributed queue được xây dựng để lưu lượng lớn dữ liệu trong một thời gian dài.
		- Kafka rất lí tưởng cho những trường hợp one-to-many và persistency là bắt buộc
		
	- Complex routing: RabbitMQ
		- RabbitMQ hỗ trợ rất nhiều tính năng và khả năng routing phức tạp. Nhưng bù lại thì message rate không quá lớn khi hỗ trợ việc complex routing communication ( > 10.000 msg/s)
