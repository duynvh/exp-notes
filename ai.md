# Artificial Intelligent

### Algorithms
- Depth-First Search (DFS)
- Breadth-First Search (BFS)
- Greedy best-first search (GBFS): sẽ tìm node gần vs đích nhất trước, dựa vào 1 heuristic function h(n)
- A* search: sẽ tìm node có giá trị g(n) + h(n) nhỏ nhất, trong đó g(n) là cost để tới được node, h(n) là estimate cost để từ node tới goal.
- A* chỉ tối ưu khi mà
	- h(n) có thể chấp nhận được (nghĩa là không bao giờ cao hơn true cost) và
	- h(n) là nhất quán (với mỗi node n và node kế vị n' với step cost c, thì h(n) <= h(n')+c)
	
- Minimax
	- Alpha beta pruning để tối ưu minimax
	- Depth-limited minimax: để giải quyết vấn đề khi số lượng possible action quá lớn (vd như cờ vua)
		- phải dùng thêm evaluation function để tính trạng thái của game ở một state nào đó.
		
