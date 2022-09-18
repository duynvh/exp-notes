# Page fault

**Page**
Là một block (segment) lưu trữ dữ liệu

**Page fault**
Là một sự gián đoạn (interruption) xảy ra khi một chương trình truy cập vào một memory block không có trên RAM. Việc này khiến cho OS phải tìm trong virtual memory để nó có thể được lấy từ disk (SSD hoặc HDD) đưa lên RAM.

![[Pasted image 20220916145822.png]]

> 1. Đầu tiên thì internal table (thường là process control block) của process xác định reference đó là valid hay invalid memory access
> 2. Nếu reference invalid thì sẽ terminate process đó. Còn trong trường hợp valid thì nếu không thể truy cập page đó thì sẽ lấy nó từ storage (backing store)
> 3. Sau đó chúng ta sẽ đến free frame list để tìm free frame
> 4. Bây giờ thì disk operation sẽ được schedule để đọc page vào allocated frame mới được cấp
> 5. Khi đọc từ disk xong thì internal table đã được modified và được keep với process, và page table chỉ ra là page bây giờ đã có trên memory.
> 6. Bây giờ chúng ta sẽ restart instruction mà bị interrupt bởi trap. Giờ process có thể access page đó như bình thường.

Trong khi page fault là những background processes trên Window, MacOS hay Linux thì một vài cái có thể gây ra issues.

Valid page fault là bình thường và cần thiết để tăng amount của memory available cho những chương tình trên các OS dùng virtual memory.

Invalid page fault là lỗi được generate bởi OS, xảy ra khi chương trình cố gắng gọi hoặc tạo một segment hay block mà memory access của nó không tồn tại. Nếu chương trình không thể tìm 1 địa chỉ mới thì OS có thể sẽ terminate chương trình đó -> crash.

**References**
- https://www.computerhope.com/jargon/p/pagefaul.htm
- https://www.computerhope.com/jargon/i/ipf.htm
- https://en.wikipedia.org/wiki/Page_fault
- https://www.javatpoint.com/page-fault-handling-in-operating-system
- https://www.studytonight.com/operating-system/page-fault-in-operating-system