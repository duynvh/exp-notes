# Why shift is slower than pop?

Hiểu một cách đơn giản thì 
- `shift` phải thay đổi index của tất cả các phần tử trong array nên sẽ có complexity là O(n)
- `pop` chỉ đơn giản là giảm length của array xuống 1 là xong nên sẽ có complexity là O(1)

Tương tự đối với `unshift` và `push`
Nhưng với `push` thì chúng ta cần chú ý một chút về cách 1 array được init trong js. Lúc init thì array sẽ được khởi tạo với một default size, trước khi reach default size thì `push` là O(1) nhưng nếu vượt quá thì engine sẽ phải tạo một contigous block khác có size gấp đôi size cũ rồi copy từng phần tử sang nên sẽ có complexity là O(n)