# Page fault

**Page**
Là một block (segment) lưu trữ dữ liệu

**Page fault**
Là một sự gián đoạn (interruption) xảy ra khi một chương trình truy cập vào một memory block không có trên RAM. Việc này khiến cho OS phải tìm trong virtual memory để nó có thể được lấy từ disk (SSD hoặc HDD) đưa lên RAM.

![[Pasted image 20220916145822.png]]
> -   The computer hardware traps to the kernel and program counter (PC) is saved on the stack. Current instruction state information is saved in CPU registers.
> -  An assembly program is started to save the general registers and other volatile information to keep the OS from destroying it.
> -  Operating system finds that a page fault has occurred and tries to find out which virtual page is needed. Some times hardware register contains this required information. If not, the operating system must retrieve PC, fetch instruction and find out what it was doing when the fault occurred.
> - Once virtual address caused page fault is known, system checks to see if address is valid and checks if there is no protection access problem.
> - If the virtual address is valid, the system checks to see if a page frame is free. If no frames are free, the page replacement algorithm is run to remove a page.
> -  If frame selected is dirty, page is scheduled for transfer to disk, context switch takes place, fault process is suspended and another process is made to run until disk transfer is completed.
> - As soon as page frame is clean, operating system looks up disk address where needed page is, schedules disk operation to bring it in.
> -  When disk interrupt indicates page has arrived, page tables are updated to reflect its position, and frame marked as being in normal state.
> - Faulting instruction is backed up to state it had when it began and PC is reset. Faulting is scheduled, operating system returns to routine that called it.
> - Assembly Routine reloads register and other state information, returns to user space to continue execution.

Trong khi page fault là những background processes trên Window, MacOS hay Linux thì một vài cái có thể gây ra issues.

Valid page fault là bình thường và cần thiết để tăng amount của memory available cho những chương tình trên các OS dùng virtual memory.

Invalid page fault là lỗi được generate bởi OS, xảy ra khi chương trình cố gắng gọi hoặc tạo một segment hay block mà memory access của nó không tồn tại. Nếu chương trình không thể tìm 1 địa chỉ mới thì OS có thể sẽ terminate chương trình đó -> crash.

**References**
- https://www.computerhope.com/jargon/p/pagefaul.htm
- https://www.computerhope.com/jargon/i/ipf.htm
- https://en.wikipedia.org/wiki/Page_fault
- https://www.javatpoint.com/page-fault-handling-in-operating-system