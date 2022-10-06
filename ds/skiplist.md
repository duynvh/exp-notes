# Skiplist
Problem: Đối với linked list thì có cách nào để tìm kiếm với độ phức tạp nhỏ hơn O(n) không?
**1. Skiplist là gì?**
- ý tưởng của skiplist là tạo ra thêm nhiều layer để chúng ta có thể bỏ qua một vài node trong quá trình tìm kiếm.
- Ví dụ:
![[Pasted image 20221007031946.png]]
- Bài toán là chúng ta muốn tìm node 50. Thay vì như bình thường chúng ta phải duyệt qua các node từ root để tìm tới node có giá trị 50 thì giờ chúng ta tạo thêm một layer nữa nằm ở trên gọi là `express lane` , ta chỉ cần duyệt qua 3 node 10, 30, 57 để xác định được node 50 nằm trong khoảng nào, rồi từ node 30, chúng ta duyệt qua từng node.
- Ta thấy ở đây chúng ta đã skip qua được những node từ 10 -> 30
- Vậy độ phức tạp về thời gian nếu chúng ta xài 2 layers là bao nhiêu? Nếu chúng ta có `n` node 
 ở `normal lance` và `√n` ở `express lane` và chúng ta chia đều normal lane, thì ở mỗi segment của normal lane (mỗi segment bao gồm những node nằm giữa 2 node ở express lane) sẽ có `√n` node. `√n` là phép chia tối ưu vs 2 layers. Với cách sắp xếp này thì số lượng node cần phải duyệt khi searchi sẽ là O(√n). Do vậy với việc dùng thêm O(√n) space, chúng ta giảm được time complexity thành O(√n).

```text
Advantages of Skip List:

-   The skip list is solid and trustworthy.
-   To add a new node to it, it will be inserted extremely quickly. 
-   Easy to implement compared to the hash table and binary search tree
-   The number of nodes in the skip list increases, and the possibility of the worst-case decreases
-   Requires only Θ(logn) time in the average case for all operations.
-   Finding a node in the list is relatively straightforward.

Disadvantages of Skip List:

-   It needs a greater amount of memory than the balanced tree.
-   Reverse search is not permitted.
-   Searching is slower than a linked list
-   Skip lists are not cache-friendly because they don’t optimize the locality of reference
```

**2. Tại sao cần skiplist?**
- Để tìm kiếm, thêm, sửa trong linked list với time complexity nhỏ hơn thành vì phải traversal qua tất cả node trong linked list.

**3. Dùng skiplist cho usecase nào?**
- Một use case phổ biến là phân trang.
- Nếu dùng query với offset và limit, thì query của bạn sẽ ngày càng chậm khi số trang càng tăng, do DB phải walk through hết tất cả offset node.
- Nếu ko dùng offset thì chúng ta phân trang bằng cách nào -> skiplist

**4. Implementation?**

Resources:
- https://www.geeksforgeeks.org/skip-list/
- https://ocw.mit.edu/courses/6-046j-introduction-to-algorithms-sma-5503-fall-2005/resources/lecture-12-skip-lists/
- https://www.cs.cmu.edu/~ckingsf/bioinfo-lectures/skiplists.pdf
- https://github.com/gansidui/skiplist