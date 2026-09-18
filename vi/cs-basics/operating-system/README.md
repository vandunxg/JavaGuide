---
title: "Chuyên đề Operating System: process, thread, memory management, file system, I/O multiplexing, Linux và Shell"
description: "Lộ trình học và câu hỏi phỏng vấn Operating System, bao quát process, thread, inter-process communication, lock và synchronization mechanism, deadlock, virtual memory, zero-copy, I/O multiplexing, file system, Linux Basics, Shell programming và các câu hỏi phỏng vấn Operating System thường gặp."
category: Computer Basics
tag:
  - Operating System
  - Linux
  - Shell
sidebar: false
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Operating System,câu hỏi phỏng vấn Operating System,process,thread,inter-process communication,IPC,lock và synchronization,mutex,semaphore,condition variable,futex,deadlock,memory management,virtual memory,zero-copy,I/O multiplexing,select,poll,epoll,file system,Linux,Shell,câu hỏi phỏng vấn backend
---

Đây là **chuyên đề Operating System** dành cho việc học backend và ôn tập phỏng vấn, tổng hợp các nội dung liên quan đến nền tảng Operating System, process và thread, inter-process communication, lock và synchronization, memory management, virtual memory, zero-copy, I/O multiplexing, file system, Linux và Shell.

## Dành cho ai

- Backend developer đang học có hệ thống nền tảng Operating System.
- Người chuẩn bị câu hỏi phỏng vấn Operating System cho tuyển dụng campus, tuyển dụng xã hội và các công ty công nghệ vừa/lớn.
- Độc giả chỉ biết học thuộc rời rạc về process và thread, deadlock, memory management, Linux command.
- Kỹ sư muốn xây nền tảng cho Java Concurrency, JVM, database và network programming.

## Trọng tâm học

- Operating System chịu trách nhiệm quản lý CPU, memory, file, I/O và process, là nền tảng để hiểu cơ chế vận hành của phần mềm tầng trên.
- Process, thread và inter-process communication là các khái niệm nền tảng của concurrent programming, performance phía server và troubleshooting.
- Lock và synchronization, deadlock, context switch, scheduling là các chủ đề thường gặp trong phỏng vấn.
- Memory management, virtual memory, paging, page replacement giúp hiểu JVM, database và cache.
- Zero-copy và I/O multiplexing giúp hiểu các component hiệu năng cao như Kafka, RocketMQ, Redis, Nginx, Netty.
- Linux và Shell là các năng lực thường dùng trong backend development, deploy, troubleshooting và automation script.

## Thứ tự đọc đề xuất

1. [Tổng hợp câu hỏi phỏng vấn Operating System thường gặp (phần 1)](./operating-system-basic-questions-01.md): trước tiên xây dựng danh sách các vấn đề thường gặp về nền tảng Operating System, process và thread, deadlock, memory management.
2. [Tổng hợp câu hỏi phỏng vấn Operating System thường gặp (phần 2)](./operating-system-basic-questions-02.md): tiếp tục bổ sung các vấn đề về file system, I/O, Linux.
3. [Giải thích chi tiết process và thread: khác biệt, trạng thái, communication, context switch và virtual thread](./process-and-thread.md): tìm hiểu có hệ thống về process, thread, PCB/TCB, fork/exec/wait, thread model và context switch.
4. [Giải thích chi tiết interrupt, exception và system call: từ kernel entry đến page fault](./interrupt-exception-syscall.md): dùng `read()` làm manh mối để kết nối hardware interrupt, synchronous exception, system call, signal, page fault và thread switch.
5. [Giải thích chi tiết CPU scheduling và system load](./cpu-scheduling-and-load.md): tìm hiểu scheduling algorithm, CFS/EEVDF, load average, CPU usage và cách troubleshooting trên production.
6. [Giải thích chi tiết inter-process communication (IPC): pipe, message queue, shared memory, Socket và Binder](./ipc.md): so sánh các phương án IPC như pipe, message queue, shared memory, semaphore, Socket, Binder.
7. [Giải thích chi tiết lock và synchronization mechanism của Operating System: mutex, semaphore, condition variable, spinlock và futex](./os-lock-and-sync.md): tìm hiểu ranh giới trách nhiệm của critical section, mutex, semaphore, condition variable, spinlock và futex.
8. [Giải thích chi tiết deadlock: bốn điều kiện cần, troubleshooting Java deadlock và xử lý database deadlock](./dead-lock.md): làm rõ deadlock wait cycle, bốn điều kiện cần, troubleshooting Java thread deadlock và database transaction retry.
9. [Giải thích chi tiết memory management của Operating System: paging, segmentation, page replacement, Swap và OOM](./memory-management.md): tìm hiểu memory allocation, fragmentation, page table, page reclaim và OOM.
10. [Giải thích chi tiết virtual memory: address translation, TLB, page fault và page replacement](./virtual-memory.md): kết nối paging, page table, TLB, page fault và page replacement.
11. [Giải thích chi tiết file system của Operating System: inode, VFS, Page Cache và journaling mechanism](./file-system.md): tìm hiểu file, directory, inode, VFS, Page Cache và journal recovery.
12. [Giải thích chi tiết I/O multiplexing: nguyên lý và khác biệt của select, poll, epoll](./io-multiplexing.md): tìm hiểu cơ chế kernel phía sau việc một thread xử lý số lượng connection lớn.
13. [Giải thích chi tiết zero-copy: mmap, sendfile và splice](./zero-copy.md): làm rõ đường đi của dữ liệu khi copy trong traditional I/O, mmap, sendfile, splice và trường hợp sử dụng.
14. [Tổng hợp kiến thức cơ bản Linux](./linux-intro.md): nắm cấu trúc directory, file permission, command thường dùng và năng lực troubleshooting cơ bản.
15. [Tổng hợp kiến thức cơ bản Shell programming](./shell-intro.md): học variable, condition, loop, function và cách viết script thường dùng.

## Bài viết cốt lõi

- [Tổng hợp câu hỏi phỏng vấn Operating System thường gặp (phần 1)](./operating-system-basic-questions-01.md): bao quát các vấn đề thường gặp về nền tảng Operating System, process và thread, deadlock, memory management.
- [Tổng hợp câu hỏi phỏng vấn Operating System thường gặp (phần 2)](./operating-system-basic-questions-02.md): tiếp tục hệ thống hóa các điểm kiến thức về file system, I/O, multiplexing, Linux.
- [Giải thích chi tiết process và thread: khác biệt, trạng thái, communication, context switch và virtual thread](./process-and-thread.md): làm rõ ranh giới resource của process và thread, state transition, cơ chế tạo process của Linux và Java virtual thread.
- [Giải thích chi tiết interrupt, exception và system call: từ kernel entry đến page fault](./interrupt-exception-syscall.md): làm rõ mối quan hệ giữa hardware interrupt, synchronous exception, system call, signal và page fault.
- [Giải thích chi tiết CPU scheduling và system load](./cpu-scheduling-and-load.md): làm rõ task scheduling, CFS/EEVDF, load average, CPU usage và command troubleshooting thường dùng.
- [Giải thích chi tiết inter-process communication (IPC): pipe, message queue, shared memory, Socket và Binder](./ipc.md): làm rõ nguyên lý, ưu nhược điểm và cách lựa chọn các phương thức IPC thường gặp.
- [Giải thích chi tiết lock và synchronization mechanism của Operating System: mutex, semaphore, condition variable, spinlock và futex](./os-lock-and-sync.md): làm rõ critical section, mutex, semaphore, condition variable, spinlock, futex, memory ordering và context của kernel lock.
- [Giải thích chi tiết deadlock: bốn điều kiện cần, troubleshooting Java deadlock và xử lý database deadlock](./dead-lock.md): làm rõ điều kiện hình thành deadlock, resource allocation graph, công cụ troubleshooting Java, deadlock detection của database và retry strategy ở application layer.
- [Giải thích chi tiết memory management của Operating System: paging, segmentation, page replacement, Swap và OOM](./memory-management.md): làm rõ VSZ/RSS/PSS, contiguous allocation, memory fragmentation, buddy system, page table, TLB, page fault, page reclaim và OOM.
- [Giải thích chi tiết virtual memory: address translation, TLB, page fault và page replacement](./virtual-memory.md): làm rõ virtual address, physical address, paging, multi-level page table, TLB, page fault và page replacement algorithm.
- [Giải thích chi tiết file system của Operating System: inode, VFS, Page Cache và journaling mechanism](./file-system.md): làm rõ file, directory, inode, dentry, file descriptor, VFS, Page Cache, fsync và journaling mechanism.
- [Giải thích chi tiết I/O multiplexing: nguyên lý và khác biệt của select, poll, epoll](./io-multiplexing.md): làm rõ hai giai đoạn của network I/O, năm I/O model và khác biệt giữa select, poll, epoll.
- [Giải thích chi tiết zero-copy: mmap, sendfile và splice](./zero-copy.md): làm rõ zero-copy thực sự loại bỏ phần nào, cùng các ứng dụng điển hình trong Java NIO, Kafka, RocketMQ.
- [Tổng hợp kiến thức cơ bản Linux](./linux-intro.md): giới thiệu Linux directory tree, file permission, command thường dùng, user và process management.
- [Tổng hợp kiến thức cơ bản Shell programming](./shell-intro.md): giới thiệu Shell variable, conditional judgment, loop, function, text processing và thực hành script.

## Câu hỏi thường gặp

- Process và thread khác nhau thế nào? Thread chia sẻ những resource nào?
- Inter-process communication có những phương thức nào? Mỗi phương thức phù hợp với trường hợp sử dụng nào?
- Context switch là gì? Context switch thường xuyên gây ảnh hưởng gì?
- Mutex, semaphore, condition variable, spinlock và futex lần lượt giải quyết vấn đề gì?
- Điều kiện cần để deadlock xảy ra là gì? Troubleshooting deadlock trong Java và database như thế nào?
- Virtual memory là gì? Paging và segmentation khác nhau thế nào?
- TLB, page fault và page replacement lần lượt giải quyết vấn đề gì?
- Có những page replacement algorithm nào? Page fault là gì?
- Vì sao zero-copy nhanh? mmap, sendfile, splice khác nhau thế nào?
- inode, hard link, soft link của file system lần lượt là gì?
- select, poll, epoll khác nhau thế nào?
- Hiểu file permission của Linux như thế nào? Có những command troubleshooting thường dùng nào?
- Shell script phù hợp để giải quyết những vấn đề automation nào?

## Chuyên đề liên quan

- [Hệ thống kiến thức Computer Basics](../)
- [Chuyên đề Computer Network](../network/)
- [Chuyên đề Data Structure](../data-structure/)
- [Java Concurrency](../../java/concurrent/java-concurrent-questions-01.md)
- [Giải thích chi tiết vùng nhớ JVM](../../java/jvm/memory-area.md)

<!-- @include: @article-footer.snippet.md -->
