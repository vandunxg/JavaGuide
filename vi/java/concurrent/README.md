---
title: "Chuyên đề Java Concurrency: thread, lock, JMM, CAS, AQS, thread pool và virtual thread"
description: "Lộ trình học và phỏng vấn Java Concurrency, bao quát thread, lock, synchronized, ReentrantLock, JMM, CAS, AQS, ThreadLocal, thread pool, CompletableFuture và virtual thread."
category: Java
tag:
  - Java
  - Java Concurrency
  - Java Interview
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Java Concurrency,Java lock,synchronized,ReentrantLock,JMM,CAS,AQS,ThreadLocal,thread pool,CompletableFuture,concurrent collections,Atomic,virtual thread
---

Java Concurrency là một trong những module quan trọng nhất, đồng thời cũng dễ gây nhầm lẫn nhất trong phát triển backend và phỏng vấn. Học concurrency không thể chỉ học thuộc API; cần xâu chuỗi vòng đời thread, cơ chế lock, memory model, atomic operation, thread pool và các lớp công cụ concurrency để hiểu.

## Dành cho ai

- Backend developer muốn học Java Concurrency có hệ thống.
- Bạn đọc đang chuẩn bị câu hỏi phỏng vấn về thread, lock, JMM, CAS, AQS, thread pool, v.v.
- Bạn đọc đã dùng multithreading trong project nhưng chưa nắm chắc các vấn đề như deadlock, tham số thread pool và rò rỉ ThreadLocal.
- Engineer muốn hiểu cách vận dụng concurrent collections, CompletableFuture và virtual thread trong thực tế.

## Trọng tâm học

- Tạo thread, vòng đời, context switch, thread safety và các vấn đề concurrency thường gặp.
- Giới hạn áp dụng của `synchronized`, `volatile`, `ReentrantLock`, mutual exclusion lock, read-write lock, optimistic lock và pessimistic lock.
- JMM, happens-before, instruction reordering, visibility, atomicity và ordering.
- Cơ chế bên trong của CAS, Atomic class, AQS, concurrent collections và blocking queue.
- Các tham số cốt lõi, rejection policy, task queue, cấu hình và thực tiễn production của thread pool.
- Cách sử dụng và rủi ro của CompletableFuture, ThreadLocal, virtual thread trong project thực tế.

## Thứ tự đọc đề xuất

1. [Tổng hợp câu hỏi phỏng vấn Java Concurrency thường gặp (phần 1)](./java-concurrent-questions-01.md): Trước tiên lập danh sách câu hỏi cơ bản về thread, lock và thread safety.
2. [Tổng hợp câu hỏi phỏng vấn Java Concurrency thường gặp (phần 2)](./java-concurrent-questions-02.md) và [Tổng hợp câu hỏi phỏng vấn Java Concurrency thường gặp (phần 3)](./java-concurrent-questions-03.md): Tiếp tục bổ sung JMM, CAS, AQS, thread pool và các công cụ concurrency.
3. [Giải thích chi tiết về Java lock](./java-lock.md), [Giải thích chi tiết về optimistic lock và pessimistic lock](./optimistic-lock-and-pessimistic-lock.md), [Giải thích chi tiết về CAS](./cas.md), [Giải thích chi tiết về JMM (Java Memory Model)](./jmm.md): Trước tiên nắm hệ thống lock, sau đó hiểu ngữ nghĩa bên trong của concurrency control.
4. [Giải thích chi tiết về AQS](./aqs.md), [Tìm hiểu nguyên lý và ứng dụng của AQS qua implementation của ReentrantLock](./reentrantlock.md): Đi sâu tìm hiểu Java lock và synchronizer.
5. [Giải thích chi tiết về Java thread pool](./java-thread-pool-summary.md) và [Best practice cho Java thread pool](./java-thread-pool-best-practices.md): Nắm vững hạ tầng concurrency cơ bản được dùng phổ biến nhất trong production.

## Bài viết cốt lõi

### Câu hỏi phỏng vấn về concurrency

- [Tổng hợp câu hỏi phỏng vấn Java Concurrency thường gặp (phần 1)](./java-concurrent-questions-01.md): Bao quát kiến thức cơ bản về thread, thread safety, lock và các vấn đề concurrency thường gặp.
- [Tổng hợp câu hỏi phỏng vấn Java Concurrency thường gặp (phần 2)](./java-concurrent-questions-02.md): Tiếp tục hệ thống hóa các kiến thức cốt lõi như JMM, volatile, CAS, AQS.
- [Tổng hợp câu hỏi phỏng vấn Java Concurrency thường gặp (phần 3)](./java-concurrent-questions-03.md): Bổ sung thread pool, các lớp công cụ concurrency, CompletableFuture và virtual thread.

### Lock, memory model và synchronizer

- [Giải thích chi tiết về Java lock](./java-lock.md): Từ mutual exclusion lock, read-write lock, spin lock đến `synchronized`, `ReentrantLock` và AQS, xây dựng hệ thống Java lock.
- [Giải thích chi tiết về optimistic lock và pessimistic lock](./optimistic-lock-and-pessimistic-lock.md): Hiểu các chiến lược xử lý xung đột concurrency khác nhau.
- [Giải thích chi tiết về CAS](./cas.md): Hiểu compare-and-swap, vấn đề ABA và chi phí spin.
- [Giải thích chi tiết về JMM (Java Memory Model)](./jmm.md): Nắm vững visibility, atomicity, ordering và happens-before.
- [Giải thích chi tiết về AQS](./aqs.md): Hiểu synchronization queue, exclusive/shared mode và bên trong các synchronizer thường gặp.
- [Tìm hiểu nguyên lý và ứng dụng của AQS qua implementation của ReentrantLock](./reentrantlock.md): Tìm hiểu sâu hơn về AQS thông qua ReentrantLock.

### Công cụ concurrency và thực tiễn áp dụng

- [Giải thích chi tiết về Java thread pool](./java-thread-pool-summary.md): Hiểu các tham số cốt lõi, task queue, rejection policy và execution flow.
- [Best practice cho Java thread pool](./java-thread-pool-best-practices.md): Tổng hợp các đề xuất về thread pool isolation, cấu hình và monitoring trong môi trường production.
- [Tổng hợp các concurrent collections thường gặp của Java](./java-concurrent-collections.md): Hệ thống hóa các concurrent collection như ConcurrentHashMap, CopyOnWriteArrayList và BlockingQueue.
- [Tổng hợp các Atomic class](./atomic-classes.md): Hiểu cách atomic update các primitive type, array, reference và field.
- [Giải thích chi tiết về ThreadLocal](./threadlocal.md): Hiểu thread-local variable, ThreadLocalMap và rủi ro memory leak.
- [Giải thích chi tiết về CompletableFuture](./completablefuture-intro.md): Nắm vững async orchestration, exception handling và cách sử dụng thread pool.
- [Tổng hợp các câu hỏi thường gặp về virtual thread](./virtual-thread.md): Hiểu vai trò, trường hợp sử dụng và giới hạn của virtual thread.

## Câu hỏi thường gặp

- Thread và process khác nhau như thế nào? Thread có những trạng thái nào?
- Thread safety là gì? Làm thế nào để chẩn đoán deadlock?
- Mutual exclusion lock, read-write lock, spin lock khác nhau như thế nào?
- `synchronized` và `ReentrantLock` khác nhau như thế nào?
- `volatile` có đảm bảo atomicity không? Nó giải quyết vấn đề gì?
- JMM là gì? Quy tắc happens-before có tác dụng gì?
- CAS có ưu và nhược điểm gì? Giải quyết vấn đề ABA như thế nào?
- Tư tưởng cốt lõi của AQS là gì? Những lớp công cụ nào dựa trên AQS?
- Cấu hình các tham số cốt lõi của thread pool như thế nào? Chọn rejection policy ra sao?
- Vì sao không khuyến nghị trực tiếp sử dụng `Executors` để tạo thread pool?
- Vì sao `ThreadLocal` có thể gây memory leak?
- Thread pool mặc định của CompletableFuture có rủi ro gì?
- Virtual thread có phù hợp với CPU-intensive task không?

## Chuyên đề liên quan

- [Hệ thống kiến thức Java](../)
- [Chuyên đề Java Collection](../collection/)
- [Chuyên đề JVM](../jvm/)
- [Chuyên đề Java IO](../io/)
- [Hệ điều hành](../../cs-basics/operating-system/)

<!-- @include: @article-footer.snippet.md -->
