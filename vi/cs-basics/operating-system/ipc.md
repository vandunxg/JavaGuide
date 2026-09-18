---
title: "Giải thích chi tiết về IPC giữa các process: pipe, message queue, shared memory, Socket và Binder"
description: "Tổng hợp kiến thức thường gặp về IPC, bắt đầu từ isolation của process address space, giải thích rõ các trade-off trong thiết kế của pipe, message queue, shared memory, semaphore, signal, Socket, Android Binder và IPC của microkernel."
category: Computer Basics
tag:
  - Operating System
  - Linux
  - IPC
head:
  - - meta
    - name: keywords
      content: "IPC giữa các process,IPC,Linux IPC,pipe,FIFO,message queue,shared memory,semaphore,signal,Socket,Unix Domain Socket,Android Binder,IPC của microkernel,câu hỏi phỏng vấn Operating System"
---

Ý tưởng trực giác nhất khi hai process muốn trao đổi một đoạn dữ liệu là: process A ghi dữ liệu vào memory của nó, sau đó process B đọc trực tiếp là được.

Tuy nhiên, cách này không thể thực hiện trong operating system. Mỗi process có virtual address space độc lập; địa chỉ `0x7f...` trong process A và địa chỉ `0x7f...` trong process B không trỏ tới cùng một vùng memory. Các process ở user space không thể tùy ý truy cập memory của nhau, nếu không isolation về quyền cũng trở nên vô nghĩa.

Vì vậy, **IPC (Inter-Process Communication, giao tiếp giữa các process)** không thể bỏ qua operating system.

Đừng nghĩ quá phức tạp, tôi thường xem IPC gồm ba việc: **truyền data thế nào, đồng bộ control flow thế nào, đặt tên và kiểm tra quyền thế nào**. Chỉ ghi nhớ các tên như “pipe, message queue, shared memory” rất dễ học xong rồi quên.

![Isolation của process address space khiến giao tiếp giữa các process phải nhờ cơ chế IPC do kernel cung cấp](https://oss.javaguide.cn/github/javaguide/java/new-features/ipc-why-ipc.png)

## IPC thực sự giải quyết vấn đề gì?

![IPC cần đồng thời giải quyết việc truyền data, đồng bộ, định danh, định tuyến và kiểm tra quyền](https://oss.javaguide.cn/github/javaguide/java/new-features/what-problem-does-ipc-solve.png)

**IPC trước hết giải quyết việc data đi qua thế nào.** Pipe và byte-stream Socket truyền các byte liên tục, message boundary do application quy ước; message queue, datagram Socket và Binder transaction truyền từng message nên vốn đã có boundary; shared memory cho phép nhiều process map cùng một vùng physical memory, sau khi map xong thì đọc ghi vùng shared không cần mỗi lần đều trap vào kernel.

**Nó cũng phải giải quyết vấn đề synchronization.** Shared memory chỉ giải quyết việc “nhìn thấy cùng một bản data”, không giải quyết việc “ai ghi trước, ai đọc sau”. Nếu nhiều process cùng sửa một ring queue mà không có mutex, semaphore, futex hoặc condition variable, data sẽ nhanh chóng bị rối.

**Naming và permission cũng không thể thiếu.** Anonymous pipe thiết lập quan hệ nhờ file descriptor được kế thừa sau `fork`; FIFO dựa vào filesystem path; System V IPC dựa vào key và kernel object ID; Unix Domain Socket có thể bind path, cũng có thể dùng abstract namespace của Linux; Android Binder nhờ Service Manager map service name tới Binder reference.

## Pipe: byte stream, đơn giản nhưng ít boundary

Pipe là loại IPC dễ gặp nhất. Trong Shell, `ps aux | grep java`, ký tự `|` ở giữa chính là việc nối standard output của process trước với standard input của process sau.

Trong Linux, gọi `pipe()` sẽ nhận được hai file descriptor: một read end và một write end. Sau khi parent process tạo pipe rồi gọi `fork()`, child process sẽ kế thừa các file descriptor này, nhờ đó parent và child có thể dùng cùng một pipe để truyền data. Anonymous pipe không có name, thường được dùng giữa các process có quan hệ parent-child.

![Pipe truyền byte stream một chiều giữa parent process và child process thông qua kernel buffer](https://oss.javaguide.cn/github/javaguide/java/new-features/ipc-pipe-flow.png)

Pipe là byte stream một chiều. POSIX chỉ yêu cầu nó một chiều, giao tiếp hai chiều thường cần tạo hai pipe; nó không hiểu message boundary, write end ghi 3 lần không có nghĩa read end cũng đọc 3 lần; buffer nằm trong kernel, khi đầy thì blocking write sẽ sleep, còn non-blocking write có thể trả về `EAGAIN`; pipe cũng không phải regular file nên không thể dùng `lseek()` để định vị ngẫu nhiên.

Trên Linux còn có một con số dễ được hỏi: `PIPE_BUF` là 4096 byte. Với pipe hoặc FIFO, khi một lần `write()` không vượt quá `PIPE_BUF`, kernel bảo đảm lần ghi này không bị đan xen với data của writer khác; ở blocking mode, thao tác có thể chờ buffer có chỗ trống, còn ở non-blocking mode, nếu không đủ chỗ sẽ trả về `EAGAIN`. Ghi vượt quá `PIPE_BUF` có thể bị tách nhỏ hoặc bị đan xen với data của writer khác. Bảo đảm này không có nghĩa pipe có message boundary.

Named pipe (FIFO) dùng `mkfifo` để tạo một special file trong filesystem; hai process không có quan hệ parent-child chỉ cần mở nó theo path là có thể giao tiếp. FIFO có path name, nhưng data không được ghi vào disk; path chỉ là entry để định danh, data thực tế vẫn nằm trong kernel buffer.

Pipe phù hợp để nối các command-line tool hoặc truyền ít data giữa parent và child process. Nó không phù hợp với protocol phức tạp và việc truyền object lớn. Nếu phải thêm length prefix, checksum, sequence number trên byte stream, trong nhiều trường hợp chuyển sang Socket hoặc message queue sẽ tốt hơn.

## Message queue: kernel lưu message, application ít phải xử lý việc chia gói

Message queue chia data thành từng message rồi lưu vào kernel object. Sender gọi `msgsnd()` hoặc `mq_send()` để đưa message vào queue, receiver gọi `msgrcv()` hoặc `mq_receive()` để lấy ra. Interface của System V message queue và POSIX message queue khác nhau, nhưng đều thuộc nhóm giải pháp “kernel sở hữu queue, process đọc ghi theo message”.

So với pipe, ưu điểm trực tiếp nhất của message queue là **message có boundary**. System V message queue hỗ trợ nhận theo message type; POSIX message queue hỗ trợ priority. Trên Linux, `sysconf(_SC_MQ_PRIO_MAX)` thường trả về 32768, trong khi POSIX standard chỉ yêu cầu hỗ trợ ít nhất khoảng từ 0 đến 31.

Chi phí cũng rất rõ ràng: khi gửi, data trong application buffer được copy vào kernel queue; khi nhận, nó lại được copy từ kernel queue về user buffer của receiver. Bản thân queue cũng chịu giới hạn của kernel parameter, chẳng hạn POSIX message queue có các giới hạn `/proc/sys/fs/mqueue/msg_max`, `msgsize_max`.

Vì vậy, message queue phù hợp để truyền các message nhỏ có cấu trúc, chẳng hạn task notification, state event và control command. Nó không phù hợp để truyền ảnh lớn, audio/video frame hoặc serialized object quá lớn. Message queue trong Linux IPC cũng không phải message middleware như Kafka hay RocketMQ: nó không có persistent log, consumer group và replication giữa các machine.

## Shared memory: ít copy nhưng phải tự xử lý synchronization

Ý tưởng của shared memory rất trực tiếp: cho nhiều process map cùng một vùng physical memory vào virtual address space riêng của mỗi process. Sau khi mapping được thiết lập, process A ghi vào vùng memory này thì process B có thể đọc được dữ liệu cập nhật.

Trên Linux có hai nhóm interface thường gặp: System V shared memory dùng `shmget()`, `shmat()`, `shmdt()`, `shmctl()`; POSIX shared memory dùng `shm_open()` để tạo object, `ftruncate()` để đặt size, sau đó dùng `mmap()` để map vào process address space.

![Shared memory cho phép nhiều process map cùng một vùng physical memory nhưng vẫn cần semaphore, futex và các synchronization mechanism khác](https://oss.javaguide.cn/github/javaguide/java/new-features/ipc-shared-memory.png)

Shared memory nhanh vì data path ngắn. Các cách như pipe, message queue và Socket thường phải giao data cho kernel trước, sau đó kernel mới giao cho process khác; sau khi hoàn tất mapping, process đọc ghi cùng một physical page, bản thân data không cần liên tục di chuyển giữa user space và kernel space. Các scenario trao đổi lượng data lớn trên cùng một machine như thu thập log, xử lý audio/video và database cache mới phù hợp để dùng nó.

Tuy nhiên, map cùng một vùng memory chỉ giải quyết vấn đề “có thể nhìn thấy hay không”, không giải quyết vấn đề “khi nào được đọc” và “ai được ghi”.

Lấy shared ring queue làm ví dụ, producer thường ghi data rồi cập nhật `tail`, consumer dựa vào `head` và `tail` để xác định có data mới hay không. Nếu producer chưa ghi xong một record mà đã cập nhật `tail`, consumer có thể lập tức đọc phải một sản phẩm dở dang. Shared memory không thể tự giải quyết vấn đề này; cần thiết kế đồng thời write order, visibility và wake-up mechanism: cách đơn giản có thể dùng inter-process mutex và POSIX semaphore; khi ưu tiên performance, có thể dùng `eventfd`, `futex`, atomic variable và memory barrier.

Còn một chi tiết rất dễ mắc lỗi: không đặt trực tiếp process pointer vào shared memory. Cùng một shared memory có thể được map tại `0x7000...` trong process A nhưng tại `0x5000...` trong process B; address A ghi vào, sau khi B lấy được phần lớn sẽ không có ý nghĩa. Cách thường gặp hơn trong engineering là lưu offset, array index hoặc ngay từ đầu quy ước một fixed-layout structure.

Vì vậy, không thể chỉ nhìn vào số lần copy để đánh giá shared memory. Performance tổng thể còn chịu ảnh hưởng của cache coherence, lock contention, memory barrier, wake-up mechanism và data layout. Nó phù hợp khi data volume lớn, hai bên cùng ở một machine và chấp nhận xử lý nghiêm túc synchronization cùng memory layout; nếu chỉ truyền vài state field hoặc một control command, message queue, pipe và Unix Domain Socket lại dễ dùng hơn.

## Semaphore và signal khác nhau thế nào?

Semaphore thường xuất hiện cùng shared memory, nhưng không chịu trách nhiệm truyền business data. Nó giống một counter hơn, dùng để kiểm soát có bao nhiêu process được vào một critical section, hoặc thông báo cho bên kia rằng “hiện đã có data để đọc”. POSIX semaphore có thể là named hoặc unnamed; `sem_post()` tăng counter lên một, `sem_wait()` thử giảm counter đi một, khi counter bằng 0 thì caller sẽ block để chờ.

Signal giống một asynchronous event notification hơn: `SIGINT` biểu thị terminal interrupt, `SIGTERM` biểu thị yêu cầu process exit, `SIGCHLD` biểu thị state của child process thay đổi. Linux hỗ trợ standard signal và real-time signal. Signal mang được ít information, handler cũng chịu giới hạn async-signal-safe. Trong production code, người ta thường để signal handler chỉ sửa một flag `volatile sig_atomic_t`, hoặc dùng `write(2)` vốn async-signal-safe để ghi một byte vào self-pipe, ghi một giá trị counter `uint64_t` vào eventfd, rồi để main loop xử lý thống nhất.

## Socket: dùng được cả trong cùng machine và giữa các machine

Socket không chỉ dùng cho network communication mà còn có thể làm IPC trên cùng machine.

Nếu hai process ở khác machine, về cơ bản phải dùng network Socket như TCP/UDP. Nếu hai process ở cùng một machine, có thể dùng Unix Domain Socket. Address family của nó là `AF_UNIX` hoặc `AF_LOCAL`, hỗ trợ các type như `SOCK_STREAM`, `SOCK_DGRAM`, `SOCK_SEQPACKET`.

Interface của Unix Domain Socket gần với network Socket, hỗ trợ giao tiếp giữa các process không có quan hệ parent-child; trên Linux có thể bind filesystem path hoặc dùng abstract namespace. Nó còn có thể truyền file descriptor nhờ ancillary data của `sendmsg()` và `SCM_RIGHTS`. Về identity của peer, Unix Socket dạng connection thường dùng `SO_PEERCRED` để lấy pid, uid, gid; trong datagram scenario cũng có thể kết hợp `SO_PASSCRED` và `SCM_CREDENTIALS` để credential đi cùng message.

Nếu được hỏi “chọn pipe hay Unix Domain Socket thế nào”, có thể trả lời: byte stream đơn giản giữa parent và child process thì pipe là đủ; nếu cần giao tiếp hai chiều, request-response, truyền fd và server listen thì Unix Domain Socket phù hợp hơn; nếu cần giao tiếp giữa các machine thì chuyển sang TCP/UDP hoặc RPC framework ở tầng cao hơn.

## Android Binder: biến IPC thành system service call

![Android Binder đóng gói IPC thành system service call thông qua AIDL, Parcel, Binder driver và Service Manager](https://oss.javaguide.cn/github/javaguide/java/new-features/android-binder-turning-ipc-into-system-service-calls.png)

IPC điển hình nhất trong Android là Binder. Việc application gọi system service, Service ở các process khác nhau tương tác và remote interface do AIDL tạo ra đều không thể thiếu nó ở tầng dưới.

Có một số điểm thiết kế của Binder đáng được xem riêng. AIDL giúp client và server thống nhất interface, Android toolchain tạo code encode/decode parameter và proxy; client có cảm giác đang gọi local method, nhưng thực tế sẽ đóng gói parameter thành Parcel rồi giao cho Binder driver thực hiện cross-process transaction. Service Manager trong system sẽ đăng ký với Binder driver làm context manager, chịu trách nhiệm duy trì mapping từ service name tới Binder reference.

Binder transaction có thể mang Binder object, handle, fd và các special object khác. Việc truyền fd giúp Binder phối hợp với shared memory: Binder truyền control message và handle, còn data lớn đặt trong shared memory. Tài liệu AIDL chính thức của Android cũng nhắc rằng remote call được dispatch vào service process từ Binder thread pool do platform quản lý, vì vậy service implementation phải tính đến thread safety.

Binder cũng không phải channel để nhồi object lớn. Tài liệu về `TransactionTooLargeException` của Android nói rõ: Binder transaction buffer hiện là 1 MB và được các transaction đang thực hiện trong process dùng chung. Bản thân exception cũng chỉ là một phán đoán heuristic: client không thể biết chính xác failure xảy ra ở giai đoạn gửi request hay trả response. Cách ổn định hơn là để Binder truyền request nhỏ, kết quả phân trang, fd hoặc resource identifier.

## Vì sao microkernel đặc biệt chú trọng IPC?

Monolithic kernel như Linux đặt rất nhiều capability, gồm filesystem, network protocol stack và driver, trong kernel. Microkernel sẽ chuyển càng nhiều service càng tốt ra các user-space process, chẳng hạn filesystem service, driver service và network service. Isolation tốt hơn, nhưng IPC sẽ trở nên rất thường xuyên.

Application đọc một file, trong monolithic kernel có thể chủ yếu chỉ là một system call vào kernel; trong microkernel, nó có thể phải giao tiếp nhiều lần với filesystem service và block device service. IPC chậm hơn một chút thì toàn bộ system cũng chậm theo.

Vì vậy, trong các paper và implementation của microkernel, tối ưu IPC luôn là trọng tâm.

Thiết kế tiêu biểu của Mach là port. Có thể hiểu port là message queue và capability handle được kernel bảo vệ: task phải giữ một port right thì mới có thể gửi hoặc nhận message tới object tương ứng. Họ L4 cố gắng làm IPC thông dụng thật ngắn: short message truyền parameter qua register, synchronous IPC dùng phong cách rendezvous, direct process switch để tránh một số path phải đi vòng qua scheduler. LRPC (Lightweight Remote Procedure Call) cũng làm cùng một việc: giảm chi phí thread, buffer và scheduling trong các invocation giữa protection domain trên cùng machine.

## Chọn IPC phổ biến thế nào?

Khi chọn giải pháp, đừng chỉ hỏi “cái nào nhanh nhất”. Câu hỏi tốt hơn là: data lớn đến đâu? Có cần message boundary không? Hai bên giao tiếp có quan hệ parent-child không? Có cần request-response hai chiều không? Có cần giao tiếp giữa các machine không? Có cần permission check và nhận diện identity không?

| IPC method         | Data form                               | Giữ message boundary?  | Phù hợp với data lớn?                        | Cross-machine?              | Scenario điển hình                                            |
| ------------------ | --------------------------------------- | ---------------------- | -------------------------------------------- | --------------------------- | ------------------------------------------------------------- |
| Anonymous pipe     | Byte stream                             | Không                  | Không phù hợp                                | Không                       | Parent-child process, Shell pipe                              |
| FIFO               | Byte stream                             | Không                  | Không phù hợp                                | Không                       | Giao tiếp đơn giản giữa process không có quan hệ parent-child |
| Message queue      | Message                                 | Có                     | Không phù hợp                                | Không                       | Control command, state event                                  |
| Shared memory      | Shared region                           | Application định nghĩa | Phù hợp                                      | Không                       | Trao đổi data lớn trên cùng machine                           |
| Unix Domain Socket | Byte stream, datagram, sequenced packet | Tùy type               | Trung bình                                   | Không                       | Server listen trên cùng machine, truyền fd                    |
| TCP/UDP Socket     | Byte stream hoặc datagram               | Tùy protocol           | Tùy protocol và implementation               | Có                          | Giao tiếp giữa các machine                                    |
| Binder             | Transaction, object reference, fd       | Có                     | Không phù hợp để truyền object lớn trực tiếp | Không, Android cùng machine | System service call của Android                               |

![So sánh các phương thức IPC phổ biến về data form, message boundary, truyền data lớn và khả năng giao tiếp giữa các machine](https://oss.javaguide.cn/github/javaguide/java/new-features/ipc-ipc-comparison.png)

Giữa parent và child process, nếu chỉ truyền ít byte stream thì pipe là đủ; process không có quan hệ parent-child cần request-response hai chiều sẽ thuận tiện hơn với Unix Domain Socket; event nhỏ có cấu trúc có thể dùng message queue; data lớn nên ưu tiên shared memory kết hợp synchronization notification; giao tiếp giữa các machine giao cho TCP/UDP hoặc RPC ở tầng cao hơn; cross-process call trong Android application thì thường đi qua Binder.

![Chọn phương thức IPC phù hợp dựa trên data volume, message boundary, khả năng giao tiếp giữa các machine và quan hệ giữa các process](https://oss.javaguide.cn/github/javaguide/java/new-features/ipc-ipc-selection.png)

## Trả lời về IPC trong phỏng vấn thế nào?

Có thể trả lời như sau: theo mặc định, process không thể truy cập trực tiếp user-space address space của nhau, vì vậy IPC hoặc để kernel thay mặt nhận và gửi data, hoặc để kernel tạo một object hoặc memory mapping có thể được share.

Nhìn theo hướng này, pipe, FIFO và Socket chủ yếu truyền byte stream, còn message boundary thường do application protocol xử lý; message queue giữ message boundary, phù hợp với task message, state change và control command tương đối nhỏ; shared memory map cùng một nhóm physical page cho nhiều process, phù hợp với việc trao đổi data lớn trên cùng machine, nhưng synchronization và memory layout phải tự xử lý; signal thiên về event notification, còn semaphore, mutex và `futex` chủ yếu phối hợp với shared data để synchronization. Android Binder có thể được xem là local RPC/transaction channel hướng tới system service, thường dùng cho cross-process service call.

Khi thực sự lựa chọn, vấn đề không nằm ở việc tên nào quen thuộc hơn mà ở data volume, message boundary, communication scope, quan hệ giữa hai bên và permission check. Ví dụ, nối các command giữa parent và child process thì pipe là đủ; local service cần request-response hai chiều và muốn truyền fd thì Unix Domain Socket phù hợp hơn; giao tiếp giữa các machine thì cân nhắc TCP/UDP hoặc RPC ở tầng trên.

Nếu được hỏi tiếp “vì sao shared memory vẫn cần semaphore”, có thể trả lời: shared memory chỉ giúp hai process nhìn thấy cùng một vùng data, không bảo đảm access order. Ai ghi trước, ai đọc sau, đang ghi dở có được đọc hay không đều phải nhờ semaphore, inter-process mutex, `futex`, `eventfd` và các mechanism khác để ràng buộc.

Nếu được hỏi “vì sao Binder không phù hợp để truyền object lớn”, có thể bổ sung giới hạn 1 MB của Binder transaction buffer trong tài liệu chính thức của Android, đồng thời nói rõ buffer này được các transaction đang thực hiện trong process dùng chung. Binder phù hợp hơn để truyền method parameter, return value, object reference và fd; data lớn nên được chia nhỏ, phân trang hoặc truyền qua shared memory.

Khi ghi nhớ IPC, đừng học thuộc nó như một chuỗi thuật ngữ; trước hết hãy hỏi: data có lớn không? Có cần giữ message boundary không? Hai bên giao tiếp có cùng machine không? Có cần request-response hai chiều không? Ai quản lý synchronization và permission? Trả lời xong các câu hỏi này thì về cơ bản phương án cũng đã rõ.
