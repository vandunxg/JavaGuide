---
title: "Giải thích chi tiết về process và thread: khác biệt, trạng thái, giao tiếp, context switch và virtual thread"
description: "Tổng hợp các câu hỏi phỏng vấn thường gặp về process và thread, hệ thống hóa các khái niệm, mô hình tài nguyên, chuyển đổi trạng thái, PCB/TCB, fork/exec/wait, mô hình thread, context switch và mối quan hệ giữa Java thread với virtual thread từ góc nhìn operating system."
category: Computer Basics
tag:
  - Operating System
  - Process và Thread
  - Java Concurrency
head:
  - - meta
    - name: keywords
      content: "process,thread,difference between process and thread,process state,thread state,PCB,TCB,fork,exec,wait,clone,pthread,context switch,thread model,Java virtual thread,operating system interview questions"
---

Process và thread là hai khái niệm cơ bản nhất trong operating system, đồng thời cũng là hai khái niệm dễ bị nhầm lẫn nhất khi học thuộc.

Khi phỏng vấn hỏi về khác biệt giữa chúng, nhiều câu trả lời chỉ dừng ở “process là đơn vị cơ bản để phân bổ tài nguyên, thread là đơn vị cơ bản để CPU lập lịch”. Câu này có thể dùng làm điểm bắt đầu, nhưng chưa đủ.

Tìm hiểu sâu hơn sẽ gặp một loạt câu hỏi cụ thể hơn: Vì sao các process mặc định được cô lập? Thread thực sự chia sẻ những gì? Sau `fork()` những gì giữa process cha và process con giống nhau, những gì đã tách biệt? Vì sao tùy ý gọi `fork()` trong chương trình multi-thread lại gây ra vấn đề? Virtual thread của Java có được xem là thread của operating system không?

Bài viết này sẽ lần lượt triển khai từ những câu hỏi đó. Trước tiên làm rõ ranh giới giữa program, process và thread, sau đó tìm hiểu `fork`, `exec`, `wait`, `clone` trong Linux, cuối cùng quay lại context switch, mô hình thread và virtual thread của Java. Khi đọc, có thể nắm một mạch chính: process thiên về ranh giới tài nguyên và tính cô lập, còn thread thiên về một đường thực thi có thể được lập lịch.

## Program, process và thread lần lượt là gì?

![Mối quan hệ giữa program, process và thread](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/relationship-between-program-process-and-thread.png)

Program là một tập hợp instruction và data được lưu trên đĩa, chẳng hạn một executable file hoặc một JAR package. Nó chưa thực sự chạy mà chỉ là file tĩnh.

Khi operating system load program vào memory, tạo virtual address space, file descriptor table và các tài nguyên cấp process khác cho nó, đồng thời tạo execution context như stack, register context cho thread ban đầu, một phiên chạy của program sẽ trở thành một **process**. Cùng một program có thể được khởi động nhiều lần, tương ứng với nhiều process; chẳng hạn mở đồng thời hai cửa sổ terminal thường là hai process instance khác nhau.

Thread là execution flow trong process. Một process có ít nhất một thread; nhiều thread trong process chia sẻ tài nguyên của process này, nhưng mỗi thread cũng có execution context riêng. Operating system hiện đại thường thực sự lập lịch thread: thread nào đang ở trạng thái có thể chạy thì scheduler có thể phân CPU time slice cho thread đó.

Có thể ghi nhớ định hướng lớn bằng một câu: **process thiên về ranh giới tài nguyên, thread thiên về thực thi và lập lịch.**

Để phán đoán một khái niệm thiên về process hay thread, trước tiên cũng có thể hỏi: Nó mô tả ranh giới tài nguyên hay một execution path? Address space, open file table và permission information thiên về process; stack, register và program counter thiên về thread.

![So sánh khác biệt giữa process và thread bằng nhà máy WeChat](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/wechat-factory-process-thread.png)

Tuy nhiên, câu này chỉ là điểm tựa khi học, không thể xem là chi tiết triển khai của mọi system. Chẳng hạn, bên trong Linux kernel dùng `task_struct` để mô tả scheduling entity; process và thread giống những task có mức độ chia sẻ tài nguyên khác nhau hơn. Tài liệu Windows lại nói rõ thread là đơn vị cơ bản để operating system phân bổ processor time. Tên gọi giữa các system không hoàn toàn giống nhau, nhưng mối quan hệ ở tầng abstraction nhìn chung tương đồng.

## Process sở hữu những tài nguyên nào?

Process không chỉ gồm code đang được thực thi. Một process thường bao gồm những nội dung sau:

- **Virtual address space**: process nhìn thấy một vùng virtual memory liên tục, bên trong có code segment, data segment, heap, stack, memory-mapped region và các thành phần khác.
- **File và handle đã mở**: chẳng hạn file descriptor, Socket, pipe, device handle.
- **Thông tin security và identity**: chẳng hạn user ID, permission, credential, security context.
- **Thông tin liên quan đến scheduling**: priority, CPU time statistic, affinity, state và các thông tin khác.
- **Signal, environment variable, working directory và execution context khác**.

Các process mặc định được cô lập. Một process không thể tùy ý đọc hoặc ghi vào virtual address space của process khác; đây cũng là nền tảng để operating system bảo vệ các program khác nhau. Nếu hai process muốn trao đổi data, chúng cần sử dụng các phương thức IPC như pipe, message queue, shared memory, Socket, file và signal.

Tính cô lập mang lại security, nhưng cũng mang lại cost. Hai process có address space và resource table riêng; việc chuyển đổi, giao tiếp, tạo và hủy đều nặng hơn thread.

## Process có những trạng thái phổ biến nào?

Mô hình năm trạng thái thường gặp trong giáo trình đủ để xử lý phần lớn câu hỏi phỏng vấn:

- **Trạng thái tạo (New)**: process đang được tạo, chưa vào ready queue.
- **Trạng thái sẵn sàng (Ready)**: các điều kiện chạy về cơ bản đã đủ, chỉ còn thiếu CPU.
- **Trạng thái chạy (Running)**: đang thực thi trên CPU.
- **Trạng thái blocked (Blocked/Waiting)**: đang chờ một event nào đó, chẳng hạn I/O hoàn tất, lock được giải phóng hoặc timer đến hạn.
- **Trạng thái kết thúc (Terminated/Exit)**: process kết thúc, operating system thu hồi các tài nguyên liên quan.

![Sơ đồ chuyển đổi trạng thái process](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/state-transition-of-process.png)

Điểm quan trọng của state transition không nằm ở tên gọi mà ở nguyên nhân kích hoạt. Ready state nhận được CPU sẽ trở thành running state; khi time slice dùng hết, process có thể quay lại ready state; phát sinh blocking I/O trong lúc chạy sẽ chuyển sang blocked state; sau khi event đang chờ hoàn tất, process trước tiên quay lại ready state và chờ được scheduling lần tiếp theo.

Một số giáo trình còn thêm **suspended state**. Suspended nhấn mạnh process tạm thời không ở trong memory hoặc bị user/system tạm dừng; blocked nhấn mạnh process đang chờ event. Hai trạng thái này không giống nhau: một process có thể bị blocked nhưng vẫn ở trong memory, cũng có thể bị swap ra external storage rồi ở trạng thái blocked và suspended.

## PCB là gì?

PCB (Process Control Block, process control block) là data structure để operating system quản lý process. Nhiều thông tin trong lúc process chạy không tự nhiên tồn tại rời rạc mà được kernel đặt và duy trì trong các structure tương tự PCB.

PCB thường ghi lại:

- Thông tin định danh process: PID, parent process ID, user ID và các thông tin khác.
- Process state và scheduling information: ready, running, blocked, priority, time statistic.
- CPU context: program counter, stack pointer, general-purpose register và các thông tin khác, giúp chuyển đổi trở lại để tiếp tục thực thi.
- Memory management information: page table, virtual address space, memory mapping.
- Resource information: file đã mở, signal handling, working directory, I/O state và các thông tin khác.

Khi xảy ra context switch, operating system lưu lại execution context như register của execution entity hiện tại, sau đó khôi phục execution context của execution entity tiếp theo. Những structure như PCB/TCB chính là căn cứ để biết “lần sau tiếp tục chạy từ đâu”.

Cách triển khai của Linux có một điểm đặc biệt: nó xem cả process và thread là task; `task_struct` không trực tiếp chứa toàn bộ resource mà dùng pointer trỏ đến các resource structure như memory descriptor, file table và signal handling. Khi nhiều thread thuộc cùng một process, chúng sẽ trỏ đến cùng một nhóm resource structure; các process khác nhau thì trỏ đến resource khác nhau. Đây cũng là lý do hiểu `clone()` rất hữu ích trên Linux.

## fork, exec, wait trong Linux lần lượt làm gì?

![Call chain của fork, exec và wait](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/fork-exec-wait-call-chain.png)

Trong lập trình Unix/Linux, việc tạo process thường không thể bỏ qua ba hành động `fork()`, `exec()`, `wait()`.

`fork()` dùng để tạo process con. Sau khi gọi thành công, process cha và process con tiếp tục thực thi từ cùng một vị trí, chỉ khác return value: process cha nhận PID của process con, process con nhận 0. Process cha và process con có virtual address space độc lập; ngay sau khi tạo, nội dung của chúng trông giống nhau. Các system hiện đại thường kết hợp copy-on-write, chỉ khi một phía ghi vào memory thì kernel mới copy page tương ứng.

Cũng cần chú ý đến file descriptor. Sau `fork()`, file descriptor table của process cha và process con là các bản sao riêng, nhưng fd tương ứng sẽ trỏ đến cùng một open file description, nên file offset, open status flag và các thông tin khác được chia sẻ. Trong thực tế thường kết hợp `FD_CLOEXEC` hoặc `O_CLOEXEC` để tránh sau `exec()` những fd không nên kế thừa bị leak sang program mới.

Nhóm hàm `exec()` dùng để nạp một program khác vào process hiện tại. Nó không tạo process mới mà thay thế user-space content của process hiện tại như code, data, heap, stack bằng program mới. Model thường gặp trong command line là: Shell trước tiên `fork()` process con, sau đó process con `exec()` thành target program.

`wait()`/`waitpid()` dùng để chờ state change của process con và thu hồi state information còn lại trong kernel sau khi process con exit. Nếu process con đã exit nhưng process cha vẫn chưa `wait`, nó sẽ để lại zombie process. Zombie process không còn thực thi code nhưng vẫn chiếm PID và bản ghi exit status.

Khi Shell khởi động external command, call chain thường là: Shell gọi `fork()` tạo process con, process con gọi `exec()` để trở thành target program, process cha dùng `wait()` hoặc `waitpid()` để chờ và thu hồi exit status.

Có một chi tiết dễ bị bỏ qua: sau khi multi-thread process gọi `fork()`, process con chỉ giữ lại thread đã gọi `fork()`. Trạng thái của lock, condition variable, malloc và stdio của các thread khác trong process cha có thể bị copy sang, nhưng các thread tương ứng đã không còn tồn tại. Nói chặt chẽ hơn, sau `fork()` và trước `exec()`, multi-thread program trong process con chỉ nên gọi các hàm async-signal-safe; thực hiện logic phức tạp trong khoảng thời gian này rất dễ gặp lỗi.

## Thread chia sẻ gì và có gì riêng?

![Tài nguyên được chia sẻ và execution context riêng của thread](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/thread-shared-and-private-content.png)

Từ góc nhìn operating system, các thread trong cùng một process chia sẻ phần lớn resource của process, chẳng hạn:

- Code segment, data segment, heap và các memory region khác trong process address space;
- File descriptor đã mở, Socket, working directory;
- Process ID, address space và một phần cấu hình signal handling;
- Global variable và heap object.

Nếu chuyển sang ngữ cảnh Java/JVM, Java thread còn chia sẻ heap, method area/metaspace và các runtime data area khác trong cùng một JVM process. Method area/metaspace không phải khái niệm operating system phổ quát; hiểu chúng ở tầng JVM sẽ phù hợp hơn.

Trong user space của Linux, các thread trong cùng một process khi gọi `getpid()` thường thấy cùng một thread group ID, tức PID của process như cách gọi thông thường; nhưng mỗi thread trong kernel vẫn có task/TID riêng, có thể dùng `gettid()` để phân biệt.

Mỗi thread cũng có nội dung riêng:

- Stack: lưu function call, local variable, return address và các thông tin khác.
- Register và program counter: ghi lại thread đang thực thi đến đâu.
- Thread ID, scheduling priority, thread-local storage (TLS).
- Thread state và một lượng nhỏ context information mà kernel dùng để khôi phục việc thực thi.

Việc chia sẻ giúp giao tiếp giữa các thread rất thuận tiện: một thread ghi data vào object trên heap thì thread khác có thể nhìn thấy ngay. Nhưng chia sẻ cũng mang lại data race: nếu nhiều thread đồng thời đọc ghi cùng một mutable data mà không có cơ chế synchronization như lock, atomic variable, condition variable, kết quả có thể không như dự kiến.

Đây cũng là khác biệt quan trọng giữa thread và process trong thực tế: process crash thường không trực tiếp phá hỏng process khác; một thread trong cùng process ghi vượt giới hạn memory hoặc trigger illegal access thường sẽ kéo theo toàn bộ process.

## TCB là gì?

TCB (Thread Control Block, thread control block) có thể hiểu là control information ở cấp thread. Nó thường ghi lại thread ID, thread state, register context, stack information, priority, thread-local storage và các nội dung khác.

Trong một số giáo trình hoặc cách triển khai hệ thống, PCB và TCB tách biệt: PCB phụ trách tài nguyên cấp process, TCB phụ trách execution context cấp thread. `task_struct` của Linux lại thống nhất scheduling entity thành task, sau đó phân biệt process và thread dựa trên việc resource structure có được chia sẻ hay không. Khi học khái niệm, không cần quá bận tâm đến tên gọi; điều quan trọng là nhìn rõ thông tin nào thuộc ranh giới tài nguyên và thông tin nào thuộc execution context.

## Đã có process thì vì sao vẫn cần thread?

Chủ yếu là để thực hiện concurrency với chi phí thấp hơn trong cùng một application.

Nếu một server cần đồng thời xử lý network read/write, tính toán nghiệp vụ và flush log xuống disk, dùng nhiều process đương nhiên cũng được, nhưng chia sẻ state giữa các process phức tạp, giao tiếp phải đi qua IPC và mức tiêu thụ tài nguyên cũng cao hơn. Khi chuyển sang nhiều thread, chúng có thể trực tiếp chia sẻ heap memory và connection đã mở; chỉ cần synchronization được viết đúng thì chi phí phối hợp thấp hơn nhiều.

Thread cũng có thể nâng cao resource utilization. Trên single-core CPU, khi một thread bị block ở disk hoặc network I/O, thread khác có thể tiếp tục chạy; trên multi-core CPU, nhiều thread có cơ hội thực thi đồng thời trên các core khác nhau. CPU-intensive task và I/O-intensive task có nhu cầu khác nhau về số lượng thread, không thể đơn giản hiểu rằng càng nhiều thread thì càng nhanh.

Thread không phải resource miễn phí. Trong Linux NPTL, nếu soft limit `RLIMIT_STACK` lúc process khởi động không phải `unlimited`, nó sẽ quyết định default stack size của thread mới; `ulimit -s` thường là 8192 KB, vì vậy default thread stack thường gặp là 8 MB. Nếu `RLIMIT_STACK` là `unlimited`, hệ thống sẽ dùng giá trị mặc định phụ thuộc kiến trúc, chẳng hạn đa số kiến trúc là 2 MB. Cũng có thể chỉ định thread stack size thông qua `pthread_attr_setstacksize()`, nhưng không được thấp hơn `PTHREAD_STACK_MIN`; giá trị do Linux man-pages đưa ra là 16384 byte. Ngoài ra, thread còn chịu giới hạn về số lượng PID, `threads-max`, memory và các tài nguyên khác. Trong production system, tùy ý tạo số lượng lớn platform thread thường dẫn đến memory pressure, scheduling overhead và context switch tăng lên.

## Phân biệt user thread, kernel thread và thread model như thế nào?

Xét theo bên chịu trách nhiệm scheduling, thread có thể chia thành user-level thread và kernel-level thread.

**User-level thread** do user-space runtime hoặc thread library quản lý; kernel thường không nhìn thấy các thread này. Ưu điểm là việc tạo và chuyển đổi không nhất thiết cần system call; vấn đề là nếu tất cả user thread chỉ tương ứng với một kernel scheduling entity thì khi một thread phát sinh blocking system call, toàn bộ process có thể bị chặn theo, đồng thời rất khó tận dụng multi-core.

**Kernel-level thread** do operating system kernel tạo và scheduling. Khi một thread blocked, kernel vẫn có thể scheduling các thread khác trong cùng process; nhiều thread cũng có thể thực thi song song trên multi-core. Cái giá phải trả là việc tạo, hủy, block, wake up và chuyển đổi đều cần kernel tham gia.

Có ba thread model phổ biến:

![Ba thread model phổ biến](https://oss.javaguide.cn/github/javaguide/java/new-features/process-and-thread-three-thread-models.png)

| Model        | Ý nghĩa                                          | Ưu điểm                                              | Vấn đề chính                                                                    |
| ------------ | ------------------------------------------------ | ---------------------------------------------------- | ------------------------------------------------------------------------------- |
| Many-to-one  | Nhiều user thread ánh xạ vào một kernel thread   | Chuyển đổi ở user space nhanh, cost triển khai thấp  | Một blocking có thể ảnh hưởng toàn bộ, không tận dụng đầy đủ multi-core         |
| One-to-one   | Một user thread ánh xạ vào một kernel thread     | Tận dụng được multi-core, ảnh hưởng của blocking nhỏ | Số lượng thread bị giới hạn bởi system resource, cost tạo và chuyển đổi cao hơn |
| Many-to-many | Nhiều user thread ánh xạ vào nhiều kernel thread | Cân bằng giữa tính linh hoạt và năng lực parallel    | Runtime và scheduling implementation phức tạp hơn                               |

POSIX thread của Linux và system thread của Windows về cơ bản thuộc one-to-one model. Bên dưới, `pthread_create()` trên Linux sẽ sử dụng `clone()`, trong đó các flag như `CLONE_VM`, `CLONE_FILES`, `CLONE_FS`, `CLONE_THREAD` quyết định những resource nào được chia sẻ. Process và thread trên Linux không phải hai cơ chế tạo hoàn toàn tách biệt mà là khác biệt về resource sharing do tham số của `clone()` mang lại.

## Thread context switch và process context switch khác nhau thế nào?

![So sánh cost của thread context switch và process context switch](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/context-switch-cost-comparison.png)

Context switch là việc CPU chuyển từ execution entity này sang execution entity khác. Operating system cần lưu execution context của entity hiện tại như register, program counter, stack pointer, sau đó khôi phục execution context của entity tiếp theo.

Thread switch và process switch đều có overhead, nhưng process switch thường nặng hơn. Nguyên nhân là process có address space độc lập; khi chuyển đổi có thể phát sinh các chi phí như page table switch, TLB invalidation và suy giảm cache locality. Các thread trong cùng một process chia sẻ address space, nên khi chuyển đổi thường không cần thay toàn bộ memory mapping.

Có thể đơn giản hóa thành hai câu: thread switch trong cùng một process chủ yếu thay execution context riêng của thread như stack, register, program counter; process switch ngoài việc thay execution context còn có thể chuyển address space và gây ảnh hưởng đến TLB cũng như cache locality.

Tuy nhiên, thread switch cũng không thể chỉ được xem là “lưu vài register”. Cross-core migration, lock contention, việc cache line liên tục bị invalidate, hoặc khi số lượng thread lớn hơn nhiều lần số CPU core, thread scheduling vẫn tiêu tốn nhiều CPU. Trong performance analysis, nếu thấy nhiều thời gian bị tiêu tốn cho scheduling, lock wait, system call và context switch, tiếp tục tăng thread thường chỉ khiến tình hình tệ hơn.

## Fiber, coroutine và virtual thread có được tính là thread không?

Fiber và coroutine thường chạy trong user space, được application hoặc runtime scheduling. Thứ mà operating system thực sự scheduling là kernel thread đảm nhiệm chúng, chứ không phải từng fiber hoặc coroutine. Vì vậy, khi chuyển đổi các lightweight execution unit này thường không cần trap vào kernel, cost có thể thấp hơn.

Nhưng chúng không phải “free thread”. Nếu runtime không chuyển blocking I/O thành dạng có thể suspend và resume, một user-space task bị block trên carrier thread sẽ khiến các task khác trên cùng carrier thread cũng bị ảnh hưởng. Ngoài ra, ngôn ngữ, runtime, CPU architecture và độ sâu call stack khác nhau đều ảnh hưởng đến switching cost; không thể xem con số tính bằng nanosecond trong một benchmark cụ thể là kết luận phổ quát.

Virtual thread được Java 21 giới thiệu là một ví dụ điển hình. Nó vẫn là `java.lang.Thread`, nhưng không độc chiếm lâu dài một operating system thread. Runtime của virtual thread sẽ mount nó lên platform thread, platform thread lại tương ứng với underlying system kernel thread; khi virtual thread thực thi loại blocking I/O mà JDK hỗ trợ suspend, JDK có thể unmount nó trước để platform thread này chạy virtual thread khác.

Vì vậy, virtual thread phù hợp với lượng lớn task “chờ I/O”, chẳng hạn high-concurrency request, database access, remote call. Thứ nó nâng cao là concurrency capacity và throughput scalability, không phải làm một đoạn CPU computation code chạy nhanh hơn. CPU-intensive long task vẫn phải xét số lượng CPU core, khối lượng computation và scheduling overhead, không thể vô hạn dồn chúng cho virtual thread.

Mối quan hệ giữa virtual thread, platform thread và system kernel thread:

![Mối quan hệ giữa virtual thread, platform thread và system kernel thread](https://oss.javaguide.cn/github/javaguide/java/new-features/virtual-threads-platform-threads-kernel-threads-relationship.png)

Cũng cần chú ý đến pinning. Lấy Java 21 làm ví dụ, khi virtual thread thực hiện blocking operation trong `synchronized` block/method, native method hoặc foreign function, nó có thể không unmount được khỏi platform thread đang đảm nhiệm nó. Kết quả là platform thread cũng bị chiếm dụng theo và không thể chạy virtual thread khác. Pinning trong thời gian ngắn và với số lượng ít sẽ không khiến program lỗi, nhưng pinning thường xuyên và kéo dài sẽ ảnh hưởng đến scalability. Các JDK về sau đã cải tiến pinning liên quan đến `synchronized`; khi đánh giá thực tế cần căn cứ vào version JDK đang sử dụng. Các boundary như native/foreign call vẫn cần được chú ý thêm.

## Tổng hợp khác biệt giữa process và thread như thế nào?

Trong phỏng vấn có thể trả lời từ 5 góc độ: tài nguyên, scheduling, communication, overhead và reliability.

| Khía cạnh                | Process                                                                    | Thread                                                            |
| ------------------------ | -------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| Định vị cơ bản           | Đơn vị cơ bản để phân bổ và cô lập resource                                | Đơn vị cơ bản để CPU scheduling và thực thi                       |
| Address space            | Độc lập theo mặc định                                                      | Chia sẻ trong cùng process                                        |
| Nội dung riêng           | PID, address space, resource table và các thông tin khác                   | Stack, register, program counter, TLS và các thông tin khác       |
| Cách giao tiếp           | Cần IPC, như pipe, Socket, shared memory                                   | Có thể trực tiếp đọc ghi shared memory nhưng phải synchronization |
| Chi phí tạo/chuyển đổi   | Thường cao hơn                                                             | Thường thấp hơn                                                   |
| Ảnh hưởng khi xảy ra lỗi | Tính cô lập tốt hơn, một process crash thường không ảnh hưởng process khác | Một thread crash có thể khiến toàn bộ process exit                |

Có thể tổ chức một câu trả lời tương đối đầy đủ như sau:

Process là resource container khi program chạy, sở hữu virtual address space độc lập cùng các resource như file và permission; thread là execution flow trong process, nhiều thread chia sẻ resource của process nhưng mỗi thread tự lưu execution context như stack, register và program counter. Process có tính cô lập mạnh hơn, chi phí communication và switching cao hơn; thread phối hợp thuận tiện hơn, chi phí tạo và switching thường nhẹ hơn, nhưng shared memory mang lại vấn đề thread safety, một thread gặp lỗi cũng có thể ảnh hưởng toàn bộ process.

## Những hiểu lầm phổ biến

**Hiểu lầm một: process là parallel, thread là concurrent.**

Concurrency và parallel mô tả quan hệ thực thi, không phải thuộc tính cố định của process/thread. Trên single-core, nhiều process hoặc thread đều chỉ có thể concurrent; trên multi-core, nhiều process hoặc thread đều có thể parallel.

**Hiểu lầm hai: càng nhiều thread thì performance càng tốt.**

Thread phù hợp để che latency do I/O wait, đồng thời có thể tận dụng multi-core; nhưng quá nhiều thread sẽ làm tăng mức sử dụng stack memory, scheduling, lock contention và cache invalidation. CPU-intensive task thường phù hợp hơn với cấu hình thread “xấp xỉ số core”; chỉ I/O-intensive task mới có thể cần nhiều concurrent execution unit hơn.

**Hiểu lầm ba: các process hoàn toàn không thể chia sẻ memory.**

Mặc định cô lập không có nghĩa là không thể chia sẻ. Shared memory chính là một phương thức IPC chuyên dùng để nhiều process map cùng một vùng physical memory, chỉ là lập trình viên phải tự xử lý synchronization và lifecycle.

**Hiểu lầm bốn: Java virtual thread chính là operating system thread.**

Platform thread thường là thin wrapper của OS thread; virtual thread do Java runtime scheduling và được mount lên platform thread để thực thi. Cả hai đều biểu hiện dưới dạng `Thread`, nhưng resource model và cơ chế scheduling khác nhau.
