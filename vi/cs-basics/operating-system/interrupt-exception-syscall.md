---
title: "Giải thích chi tiết về interrupt, exception và system call: từ kernel entry đến Page Fault"
description: "Tổng hợp câu hỏi phỏng vấn thường gặp về interrupt, exception và system call, dùng read() làm manh mối để giải thích hardware interrupt, synchronous exception, system call, signal, clock interrupt, Page Fault và mối quan hệ giữa thread context switch."
category: Computer Science Basics
tag:
  - Operating System
  - Linux
  - System Call
head:
  - - meta
    - name: keywords
      content: interrupt,exception,system call,trap,signal,user mode,kernel mode,context switch,clock interrupt,Page Fault,SIGSEGV,read,Operating System interview questions
---

System call đi từ user mode vào kernel mode chỉ là điểm bắt đầu để hiểu đường đi này. Xem sâu hơn qua một lần `read()`, ta sẽ gặp một số vấn đề liên quan chặt chẽ:

- `read()` đi vào kernel bằng cách nào?
- Vì sao clock interrupt có thể khiến thread đang chạy dừng lại?
- Vì sao Page Fault đôi khi là hành vi bình thường, nhưng đôi khi lại trở thành `SIGSEGV`?
- System call đã vào kernel thì có nhất thiết xảy ra thread context switch không?

Có thể xâu chuỗi các vấn đề này qua một lần gọi `read(fd, buf, count)`. User program thực thi `read()`, glibc đặt system call number và các tham số vào các thanh ghi đã quy ước, CPU thực thi `syscall` để vào kernel. Kernel kiểm tra fd, địa chỉ buffer, trạng thái file, sau đó quyết định lấy dữ liệu từ Page Cache, file system, socket buffer hoặc device driver.

Nếu dữ liệu đã sẵn sàng, kernel copy dữ liệu về user buffer và `read()` nhanh chóng trả về. Khi dữ liệu chưa sẵn sàng, thread hiện tại có thể sleep; sau khi disk I/O hoàn tất hoặc network card nhận được dữ liệu, hardware interrupt đi vào kernel, rồi kernel đánh thức thread trong waiting queue. Thread được schedule lên CPU lần nữa, khi đó `read()` mới tiếp tục trả về.

Sau khi một I/O thông thường chạy, system call, interrupt, exception và scheduling thường nối liền với nhau.

## Các loại event đi vào kernel

Khi CPU thực thi user program bình thường, program counter và logic jump quyết định instruction tiếp theo. Event từ thiết bị ngoại vi, lỗi của instruction hiện tại hoặc yêu cầu service từ kernel do user program chủ động đưa ra đều khiến control flow chuyển vào kernel. CSAPP gọi các trường hợp thoát khỏi instruction flow bình thường này là Exceptional Control Flow, đồng thời phân biệt interrupt, trap, fault và abort thành các loại khác nhau.

Có thể phân loại các entry point thường gặp theo nguồn:

- **Interrupt**: đến từ hardware bên ngoài và không có quan hệ trực tiếp với instruction hiện tại. Network card nhận packet, disk I/O hoàn tất, timer đến thời điểm đều thuộc loại này.
- **Trap**: program chủ động thực thi special instruction để vào kernel. System call là trap thường gặp nhất.
- **Fault**: gặp vấn đề khi thực thi instruction hiện tại, nhưng kernel có thể sửa chữa. Ví dụ điển hình là Page Fault; sau khi sửa xong, instruction gây ra fault sẽ được thực thi lại.
- **Abort**: processor phát hiện lỗi nghiêm trọng khó khôi phục, thường không quay lại instruction flow ban đầu.

![Sơ đồ quan hệ giữa interrupt, exception và system call](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/ecf-kernel-entry-map.webp)

Từ `trap` được dùng không hoàn toàn giống nhau trong các tài liệu khác nhau. Trong ngữ cảnh CSAPP, nó thường chỉ synchronous exception do program chủ động kích hoạt, chẳng hạn system call; còn RISC-V định nghĩa trap là việc chuyển control flow do exception hoặc interrupt gây ra. Trong bài này, khi nhắc đến “trap / system call”, ta dùng nghĩa hẹp thứ nhất; khi liên quan đến RISC-V sẽ nói riêng.

## Mối quan hệ giữa interrupt, exception, system call và signal

Bốn khái niệm này dễ bị nhầm vì chúng không nằm ở cùng một layer.

Hardware interrupt, synchronous exception và system call mô tả lý do CPU đi vào kernel; còn signal là software notification do kernel gửi đến process hoặc thread. Signal có thể xuất phát từ hardware exception đã được chuyển đổi, hoặc do process khác, terminal hay timer tạo ra.

Trước hết, hãy đặt các khái niệm này vào một bảng:

| Khái niệm             | Nguồn kích hoạt                                                                    | Đồng bộ/bất đồng bộ                                 | Bên xử lý                                                       | Kết quả thường gặp                                                   |
| --------------------- | ---------------------------------------------------------------------------------- | --------------------------------------------------- | --------------------------------------------------------------- | -------------------------------------------------------------------- |
| Hardware interrupt    | Thiết bị ngoại vi hoặc timer                                                       | Bất đồng bộ                                         | Kernel interrupt handler                                        | Xử lý event của device, đánh thức waiting task, kích hoạt scheduling |
| Synchronous exception | Quá trình thực thi instruction hiện tại                                            | Đồng bộ                                             | Kernel exception handler                                        | Retry sau khi sửa, chuyển thành signal, terminate process            |
| System call           | Trap do user program chủ động kích hoạt bằng các instruction như `syscall`/`ecall` | Đồng bộ                                             | Kernel system call entry                                        | Trả về kết quả, error code, block để chờ resource                    |
| Signal                | Notification do kernel hoặc process gửi                                            | Thường bất đồng bộ, cũng có thể do exception gây ra | Default action của target process hoặc user mode signal handler | Ignore, terminate, pause, continue, execute handler                  |

Đồng bộ/bất đồng bộ trong bảng này xét xem event có được instruction hiện tại dẫn đến hay không. Chia cho 0, instruction bất hợp lệ, Page Fault và system call đều liên quan đến instruction đang thực thi, nên là synchronous event. Hardware interrupt đến từ thiết bị ngoại vi hoặc timer; khi CPU đang chạy Java thread, network card cũng có thể vừa nhận packet, việc này không liên quan trực tiếp đến dòng user code hiện tại, nên là asynchronous event.

Blocking và non-blocking là một chiều khác. Bước `read()` đi vào kernel là synchronous, nhưng sau khi vào kernel, nếu resource chưa sẵn sàng thì blocking fd khiến thread sleep; fd được thiết lập `O_NONBLOCK` có thể trả về `EAGAIN` ngay.

Hardware interrupt giống như thiết bị ngoại vi gõ vào CPU một lần. Ví dụ CPU đang chạy Java thread của bạn, clock interrupt đến, CPU chạy xong instruction hiện tại rồi đi vào interrupt entry của kernel. Kernel cập nhật clock, thống kê runtime, và khi cần thì để scheduler giao CPU cho thread khác. Trong code của bạn không hề viết thao tác nhường CPU, nhưng thread vẫn có thể bị preempt.

Exception xuất hiện trên instruction hiện tại. Chia cho 0, thực thi instruction bất hợp lệ, truy cập địa chỉ không có permission đều là vấn đề do instruction này kích hoạt. Page Fault cũng thuộc loại này: process truy cập một virtual address, page table tạm thời chưa có mapping hợp lệ, CPU chỉ có thể bàn giao context cho kernel. Synchronous exception không có nghĩa là chắc chắn sửa được; khi kernel không thể sửa, nó vẫn sẽ deliver signal hoặc terminate process.

System call là việc user program chủ động nhờ kernel hỗ trợ. User mode program không thể trực tiếp đọc disk, sửa page table hoặc thao tác với network card, nên `read()`, `write()`, `fork()`, `mmap()` của glibc cuối cùng đều phải đi qua system call interface do kernel cung cấp.

Signal không thuộc cơ chế CPU entry. Nó là notification do kernel gửi cho process hoặc thread. Truy cập memory bất hợp lệ có thể trước tiên kích hoạt Page Fault; kernel phát hiện không thể sửa, sau đó deliver `SIGSEGV` cho process. Khi người dùng nhấn `Ctrl+C`, terminal driver sẽ khiến kernel gửi `SIGINT` cho foreground process group. Process khác cũng có thể gọi `kill()` để gửi signal.

Các chiều dễ bị nhầm có thể tách riêng như sau:

| Chiều                          | Vấn đề cần quan tâm                                          | Ví dụ                                                                            |
| ------------------------------ | ------------------------------------------------------------ | -------------------------------------------------------------------------------- |
| Synchronous/asynchronous event | Event có được instruction hiện tại trực tiếp kích hoạt không | Page Fault là synchronous exception; network interrupt là asynchronous interrupt |
| Blocking/non-blocking I/O      | Khi resource chưa sẵn sàng, thread có chờ không              | Blocking `read()` sẽ sleep; non-blocking `read()` có thể trả về `EAGAIN`         |
| Chuyển user mode/kernel mode   | Có đi vào kernel để thực thi privileged code không           | `syscall`, Page Fault và hardware interrupt đều đi vào kernel                    |
| Thread context switch          | CPU có chuyển từ thread này sang thread khác không           | Có thể xảy ra khi block, preempt hoặc schedule                                   |

Signal handler cũng không phải kernel function. Kernel thường kiểm tra signal đang chờ trước khi từ kernel mode trở về user mode; nếu cần thực thi handler, kernel chuẩn bị user stack, register và trampoline, sau đó để thread trở về user mode và thực thi handler. Vì vậy, signal handler thường không được chèn thực thi ngay giữa hai machine instruction bất kỳ, mà chờ thread từ kernel mode trở về user mode hoặc được đánh thức khỏi interruptible wait, rồi mới vào handler theo sắp xếp của kernel. Với multithreaded program cần chú ý thêm một bước: signal gửi cho process không nhất thiết do thread mà bạn dự đoán xử lý; kernel sẽ chọn một thread không block signal đó.

Sau khi event được xử lý, nơi quay lại cũng khác nhau:

| Loại               | Thường quay lại đâu sau khi xử lý                                                             |
| ------------------ | --------------------------------------------------------------------------------------------- |
| Interrupt          | Quay lại vị trí bị ngắt để tiếp tục thực thi, hoặc schedule thread khác                       |
| Trap / system call | Thường tiếp tục thực thi sau instruction gây trap; trừ các trường hợp như system call restart |
| Fault / Page Fault | Thực thi lại instruction gây ra fault sau khi sửa xong                                        |
| Abort              | Thường không quay lại program ban đầu                                                         |

Bảng này chỉ mô tả path phổ biến nhất. Trong system thực tế, kernel còn có thể deliver signal, restart system call, chuyển sang thread khác hoặc terminate process ngay.

## Chuyển user mode/kernel mode và context switch

Điểm khác nhau giữa user mode và kernel mode nằm ở privilege level của CPU. User mode không thể thực thi privileged instruction và không thể tùy ý truy cập kernel address space; kernel mode có thể quản lý page table, device, interrupt controller và scheduler.

Đi từ user mode vào kernel mode không phải là một function call thông thường. CPU và kernel phải lưu đủ context information, nếu không sau đó sẽ không biết phải quay lại instruction nào của user program.

Trên x86-64, 64-bit system call thường đi qua instruction `syscall`; exception và external interrupt phần lớn đi qua entry đã cấu hình trong IDT. Một số exception sẽ push error code, một số khác thì không; các entry đặc biệt như NMI và Double Fault còn có thể dùng IST stack.

Bài này dùng Linux x86-64 làm ví dụ, nên chủ yếu viết `syscall`. Architecture khác hoặc ABI cũ có thể dùng các entry instruction như `int 0x80`, `sysenter`, `ecall`, `svc`; quy ước register cũng khác.

Bản thân instruction `syscall` thực hiện khá ít việc. Nó đặt return address và flag register vào `RCX`, `R11`, nhưng không lưu đầy đủ register context như function call thông thường, cũng không tự động chuyển sang kernel stack. Linux entry assembly còn phải tiếp tục hoàn thành việc đổi stack, `swapgs`, lưu register và các công việc khác.

Cũng cần phân biệt chuyển user mode/kernel mode với thread context switch:

- Chuyển user mode/kernel mode: CPU đi từ privilege level thấp vào privilege level cao, thực thi kernel code rồi trở về user mode.
- Context switch: scheduler chuyển CPU từ một thread hoặc process sang execution entity khác.

System call chắc chắn đi vào kernel, nhưng không nhất thiết chuyển sang thread khác. Các call như `getpid()` thường trả về rất nhanh và thread hiện tại vẫn tiếp tục chạy. Nếu `read()` phải chờ dữ liệu, kernel có thể suspend thread hiện tại rồi schedule thread khác. Ngoài ra, một số time-related interface có thể hoàn thành ở user mode nhờ vDSO; chẳng hạn trong một số architecture và configuration, `clock_gettime()` và `gettimeofday()` có thể đọc data page do kernel map cho user mode, không nhất thiết lần nào cũng thật sự đi vào kernel.

![Sơ đồ so sánh chuyển user mode/kernel mode và context switch](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/kernel-mode-vs-context-switch.webp)

## Path system call của `read()`

Lấy `read(fd, buf, count)` trên Linux x86-64 làm ví dụ, business code thường gọi wrapper function của glibc chứ không tự viết assembly.

![Sơ đồ flow system call của read](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/read-syscall-path.webp)

glibc đặt system call number vào `rax` và các tham số vào các register đã quy ước. Các tham số system call trên x86-64 lần lượt nằm trong `rdi`, `rsi`, `rdx`, `r10`, `r8`, `r9`.

Sau khi CPU thực thi `syscall`, nó sẽ nhảy đến entry đã được kernel cấu hình theo quy ước của architecture. Linux entry code lưu register state cần dùng về sau, rồi dispatch đến handler tương ứng với `read` dựa trên system call number. Kernel kiểm tra fd, permission và các tham số khác; khi thật sự copy dữ liệu vào user buffer, vấn đề địa chỉ vẫn có thể gây ra `EFAULT`.

Nếu target là regular file, path sẽ đi qua VFS và file system cụ thể, ưu tiên lấy dữ liệu từ Page Cache. Nếu target là socket, kernel kiểm tra receive buffer có dữ liệu hay chưa. Khi dữ liệu đã sẵn sàng, kernel copy dữ liệu vào `buf` do user truyền vào.

Khi kernel truy cập user buffer, nó cũng có thể kích hoạt Page Fault. Linux ghi lại các điểm user memory access có khả năng fault trong exception table; nếu fault xảy ra tại vị trí có thể sửa, kernel sẽ nhảy đến fixup code tương ứng, chuyển kết quả thành lỗi như `-EFAULT` thay vì làm kernel crash trực tiếp.

Ví dụ, nếu truyền `buf` của `read()` thành một địa chỉ rõ ràng không thể ghi, system call thường không kéo kernel chết theo mà trả về `-1` và đặt `errno` thành `EFAULT`.

```c
#include <errno.h>
#include <fcntl.h>
#include <stdio.h>
#include <unistd.h>

int main(void) {
    int fd = open("/dev/zero", O_RDONLY);
    char *p = (char *)1;
    ssize_t n = read(fd, p, 1); // n == -1, errno == EFAULT
    printf("n=%zd errno=%d\n", n, errno);
}
```

Mô tả về `EFAULT` của `read(2)` chính là user buffer không nằm trong address space có thể truy cập. Đối chiếu với kernel path, vấn đề nằm ở bước kernel copy dữ liệu trở lại user buffer. Tài liệu exception table của x86 cũng dùng `get_user()` làm ví dụ: instruction truy cập user memory có thể fault sẽ đi theo cặp với một đoạn fixup code; sau khi Page Fault xảy ra, kernel tra được cặp địa chỉ này, đổi return value thành `-EFAULT`, rồi nhảy đến fixup path để hoàn tất.

Khi system call trả về, `read()` thành công trả về số byte thực tế đã đọc; giá trị này có thể nhỏ hơn `count` và không được xem là lỗi. Khi thất bại, kernel trả về negative error code, còn wrapper function của glibc thường chuyển nó thành `-1` và set `errno`. Khi user buffer không thể truy cập, có thể nhận `EFAULT`; khi bị signal interrupt trong thời gian blocking wait, có thể nhận `EINTR`.

Khi viết production code, không nên mặc định một lần `read()` hoặc đọc đủ hoặc thất bại. Trong thời gian blocking system call chờ đợi, nếu thread nhận signal và thực thi handler, system call có thể trả về `EINTR`; nếu trước khi signal đến đã đọc được một phần dữ liệu, `read()` cũng có thể trả về trực tiếp số byte đã đọc thay vì thất bại. Nếu dùng `SA_RESTART` khi cài đặt handler, một phần blocking system call sẽ tự động restart sau khi handler return. Có restart hay không phụ thuộc vào loại interface và cấu hình signal handling.

## Clock interrupt và preemption

Operating system có preemption không thể trông chờ mọi program chủ động nhường CPU. Sách giáo khoa thường đơn giản hóa path này thành việc kernel cấu hình timer, hardware định kỳ tạo interrupt. Linux hiện đại hỗ trợ tickless, nên machine thực tế không nhất thiết luôn tạo scheduling tick theo tần số cố định, nhưng timer interrupt vẫn là model nền tảng để hiểu preemption.

Giả sử thread A đang chạy ở user mode. Khi timer đến thời điểm, CPU đi vào clock interrupt handler của kernel. Kernel cập nhật runtime của thread hiện tại và kiểm tra có cần schedule hay không. Nếu không cần schedule, sau khi xử lý xong sẽ quay lại A và A tiếp tục chạy; nếu cần schedule, kernel lưu execution context của A, chọn thread B, chuyển sang kernel stack và register context của B, cuối cùng từ kernel trở về vị trí user mode của B.

Khi OSTEP giải thích Limited Direct Execution, nó cũng triển khai theo chain này: timer interrupt trước tiên khiến hardware và kernel lưu user register của process hiện tại, kernel sau đó gọi switch routine để lưu context của process cũ và khôi phục context của process mới, cuối cùng thông qua return-from-trap để quay về process mới.

Path này chắc chắn xảy ra interrupt, nhưng có xảy ra context switch hay không còn phụ thuộc scheduler có chọn thread khác hay không.

Interrupt handler của hardware thường phải xử lý nhanh và thoát nhanh, không thể tùy ý block wait như process context thông thường. Công việc nặng hơn sẽ được trì hoãn đến softirq, work queue hoặc kernel thread để xử lý.

Network card nhận packet là một ví dụ phổ biến. Path cơ bản của Linux NAPI là: device trước tiên dùng hardware interrupt để thông báo cho host, driver schedule NAPI trong interrupt handler, phần xử lý packet tiếp theo thường chạy trong softirq context. Sau khi driver schedule NAPI, nó thường tiếp tục giữ IRQ ở trạng thái masked cho đến khi NAPI polling kết thúc, vì trong khoảng thời gian này tiếp tục nhận hardware interrupt là không cần thiết. Khi khối lượng xử lý quá lớn hoặc softirq bị trì hoãn, việc xử lý cũng có thể do kernel thread như `ksoftirqd` tiếp tục thực hiện. Khi thấy `ksoftirqd` hoặc `%si` tăng cao trong thời gian dài ở production, cần liên tưởng đến xử lý network packet, áp lực softirq và interrupt affinity. Hardware interrupt chỉ phụ trách treo công việc tiếp theo; khi batch process packet, execution đã chuyển sang softirq hoặc kernel thread context.

## Path bình thường và path lỗi của Page Fault

Tên Page Fault dễ khiến người ta nghĩ rằng program đã xảy ra lỗi. Thực tế, nó chỉ cho biết khi CPU thực hiện address translation hoặc permission check, page table entry hiện tại không thể trực tiếp hoàn tất access này.

Có một số trường hợp thường gặp:

1. Page chưa được cấp physical memory, chẳng hạn heap page được lazy allocation lần đầu bị truy cập;
2. Page nằm trong file hoặc Swap nhưng hiện chưa resident trong memory;
3. COW page bị ghi, page table tạm thời đánh dấu read-only và kernel cần copy một bản;
4. Permission truy cập không đúng, chẳng hạn user mode truy cập kernel page, ghi vào read-only page hoặc execute page không cho phép execute;
5. Địa chỉ hoàn toàn không thuộc virtual address region hợp lệ của process.

![Sơ đồ các nhánh xử lý Page Fault](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/page-fault-branching.webp)

Khi xử lý Page Fault, kernel trước hết xem địa chỉ có nằm trong VMA hợp lệ của process hay không, sau đó kiểm tra access type và permission có phù hợp không.

Page Fault hợp lệ có thể sửa được. Kernel cấp physical page, đọc page từ file, swap page từ Swap vào hoặc xử lý COW, cập nhật page table rồi trả về. CPU sẽ thực thi lại instruction gây ra exception. Page Fault loại này có thể là minor fault hoặc major fault; khác biệt nằm ở việc có cần I/O thực tế hay không.

Nếu địa chỉ không thuộc VMA hợp lệ nào hoặc cách truy cập vi phạm page-level permission, kernel thường deliver `SIGSEGV` cho thread hiện tại. Truy cập vùng gần `NULL`, ghi vào read-only mapping đều thuộc loại này. Out-of-bounds access trong C/C++ không đảm bảo kích hoạt Page Fault: nếu địa chỉ đích vẫn nằm trong page đã mapped và permission cho phép, program có thể chỉ phá hỏng dữ liệu lân cận. Chỉ khi địa chỉ vượt biên rơi vào vùng chưa mapped hoặc vi phạm page permission, hardware mới dùng Page Fault để bàn giao vấn đề cho kernel.

COW fork và lazy allocation của xv6 rất phù hợp để giúp hiểu điểm này. Process cha và con trước tiên chia sẻ read-only page; process nào ghi thì process đó kích hoạt page fault, kernel copy page rồi cho phép tiếp tục ghi. Khi process mở rộng address space, kernel có thể trước tiên chỉ ghi nhận range, đến lần access đầu tiên mới cấp physical page. Cả hai scenario đều dùng Page Fault để trì hoãn công việc.

`userfaultfd(2)` có thể làm một ví dụ nâng cao: sau khi user mode đăng ký một vùng memory, các page fault loại missing, minor hoặc write-protect có thể trở thành event trên fd; thread kích hoạt fault trước tiên sẽ block, sau khi một user mode thread khác bổ sung page, tiếp tục hoặc gỡ write protection thì thread đó mới tiếp tục. Cơ chế này thường được dùng cho virtual machine migration, lazy loading và dirty page tracking, nhưng backend business thông thường hiếm khi dùng trực tiếp.

## Chi phí của system call

System call nặng hơn function call thông thường. Function call thông thường vẫn ở user mode, truyền tham số theo ABI, lưu context cần thiết rồi hoàn thành việc jump và return; system call còn phải chuyển sang kernel mode, đi qua entry code để lưu context, thực hiện permission check, và có thể truy cập page table, file object, device driver hoặc waiting queue. Trước khi từ kernel trở về user mode, kernel còn có thể kiểm tra signal đang chờ, preemption, scheduling flag và các state khác.

Tuy nhiên, không thể đánh đồng chi phí của mọi system call. Với call như `getpid()`, phần lớn chi phí nằm ở việc vào và ra khỏi kernel; với `read()` gặp disk I/O, chi phí chính nằm ở việc chờ device và copy dữ liệu. Lock triển khai dựa trên futex khi không có contention thường chỉ thực hiện user mode atomic operation, không gọi `futex(2)`; khi xảy ra contention, cần thread sleep hoặc wakeup thì mới đi vào kernel thông qua `futex(2)`.

Trong engineering, không nên hy sinh correctness chỉ để giảm một lần system call. Các tối ưu phổ biến hơn là batch và giảm thời gian chờ vô nghĩa: buffered I/O, đọc ghi nhiều dữ liệu hơn mỗi lần, I/O multiplexing, `sendfile()`, `mmap()`, `io_uring`; tùy scenario, chúng giảm chi phí mode switch, copy hoặc wait.

## Điểm cần nêu khi phỏng vấn

Khi trả lời “mối quan hệ giữa interrupt, exception và system call là gì”, có thể trình bày theo nguồn của entry point:

> CPU thực thi bình thường theo instruction flow. Event từ thiết bị ngoại vi, lỗi của instruction hiện tại và yêu cầu service từ kernel do user program chủ động đưa ra đều khiến control flow đi vào kernel. Hardware interrupt đến từ external device và là asynchronous; synchronous exception do instruction hiện tại kích hoạt; còn system call là trap do program chủ động kích hoạt bằng các instruction như `syscall`, `ecall`, nên cũng là synchronous event. Sau khi kernel xử lý xong, CPU có thể quay lại program ban đầu để tiếp tục thực thi, cũng có thể schedule thread khác hoặc deliver signal cho process.

Khi trả lời “system call có flow như thế nào”, có thể lấy `read()` làm trọng tâm:

> Wrapper function của glibc đặt system call number và các tham số vào register đã quy ước, rồi thực thi `syscall`. CPU trước tiên đi vào kernel entry theo quy ước của architecture; Linux entry code tiếp tục lưu register state cần dùng, dispatch đến handler tương ứng dựa trên system call number, kiểm tra tham số và permission, rồi thực thi logic của VFS, network, memory management và các phần khác. Khi trả về, kết quả được đặt vào register; khi xảy ra lỗi, glibc thường chuyển thành `-1` và `errno`. Nếu call phải chờ I/O, thread sẽ block, sau đó device interrupt đánh thức thread.

Khi trả lời “điểm khác nhau giữa Page Fault và illegal access”, hãy nắm flow phân nhánh trong kernel:

> Page Fault chỉ cho biết CPU phát hiện address translation hoặc permission check của access này không thể hoàn tất. Kernel sẽ xác định địa chỉ và permission có hợp lệ không. Page Fault hợp lệ có thể sửa, chẳng hạn cấp anonymous page, đọc page từ file hoặc Swap, xử lý COW, rồi thực thi lại instruction gây exception; illegal access không thể sửa, thường deliver `SIGSEGV` và process terminate theo default action.

Chuyển user mode/kernel mode và thread context switch không phải cùng một việc. Một system call thực sự chắc chắn đi vào kernel, nhưng CPU chỉ switch thread khi scheduler chọn execution entity khác, chẳng hạn thread hiện tại block, chủ động nhường CPU hoặc bị preempt.
