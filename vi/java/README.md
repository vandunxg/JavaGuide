---
title: "Hệ thống kiến thức Java: Java Basics, Collection, Concurrency, JVM, IO và tính năng mới"
description: Lộ trình học và ôn phỏng vấn Java backend, bao quát Java Basics, source code Collection, Concurrency, JVM, IO/NIO và tính năng mới, phù hợp với sinh viên mới tốt nghiệp và người đi làm.
category: Java
tag:
  - Java
  - Java Basics
  - Java Interview
sitemap:
  changefreq: weekly
  priority: 0.95
head:
  - - meta
    - name: keywords
      content: Java,Java Basics,Java Collection,Java Concurrency,JVM,Java IO,Java NIO,Java New Features,Java Interview Questions,Java Backend Interview
---

<!-- @include: @small-advertisement.snippet.md -->

Hệ thống kiến thức Java này dành cho việc học Java backend và ôn phỏng vấn, sắp xếp các bài viết Java trên website theo thứ tự “cú pháp cơ bản -> collection -> lập trình concurrency -> IO/NIO -> JVM -> tính năng mới”.

Nếu ít thời gian, bạn nên đọc trước các bài tổng hợp câu hỏi phỏng vấn về Java Basics, Collection, Concurrency và JVM để nắm nhanh các câu hỏi thường gặp; nếu muốn củng cố kiến thức cơ bản có hệ thống, hãy đọc theo thứ tự chuyên đề bên dưới.

## Dành cho ai

- Backend developer đang học Java có hệ thống.
- Người chuẩn bị phỏng vấn Java backend khi tìm việc sau tốt nghiệp, chuyển việc hoặc ứng tuyển vào công ty vừa và lớn.
- Độc giả muốn ôn tập liền mạch Java Basics, Collection, Concurrency, JVM, IO và các tính năng mới.
- Kỹ sư đã viết dự án Java nhưng chưa hiểu có hệ thống về nguyên lý bên trong, thiết kế source code và thực tiễn kỹ thuật.

## Trọng tâm học

- Cú pháp cơ bản Java, lập trình hướng đối tượng, exception, generic, reflection, proxy, serialization và các cơ chế cốt lõi khác.
- Phạm vi sử dụng, cách triển khai trong source code và câu hỏi phỏng vấn thường gặp về List, Map, Queue và các collection hỗ trợ concurrency.
- thread, lock, JMM, CAS, AQS, thread pool, CompletableFuture và virtual thread trong Java.
- Vùng nhớ JVM, class loading, garbage collection, cấu hình tham số, công cụ monitoring và xử lý sự cố trên production.
- BIO, NIO, AIO, các mô hình IO và design pattern về IO như decorator, adapter.
- Các tính năng mới quan trọng từ Java 8 đến Java 26, cùng những tính năng thực sự ảnh hưởng đến việc phát triển hằng ngày.

## Thứ tự đọc đề xuất

1. [Chuyên đề Java Basics](./basis/): Nắm vững cú pháp, lập trình hướng đối tượng, generic, reflection, proxy, serialization và các kiến thức cơ bản khác.
2. [Chuyên đề Java Collection](./collection/): Hiểu cách sử dụng và source code của các container thường dùng như ArrayList, LinkedList, HashMap, ConcurrentHashMap.
3. [Chuyên đề lập trình Java Concurrency](./concurrent/): Học có hệ thống về thread, lock, JMM, CAS, AQS, thread pool và các công cụ concurrency.
4. [Chuyên đề JVM](./jvm/): Hiểu vùng nhớ, class loading, garbage collection, tham số JVM và xử lý sự cố trên production.
5. [Chuyên đề Java IO](./io/): Bổ sung BIO, NIO, AIO, Reactor, I/O multiplexing và các design pattern về IO.
6. [Chuyên đề tính năng mới của Java](./new-features/): Tổng hợp theo phiên bản các tính năng quan trọng như Lambda, Stream, modularization, var, Record, virtual thread.

## Bài viết cốt lõi

### Java Basics

- [Chuyên đề Java Basics](./basis/): Trình bày từ cú pháp cơ bản đến các cơ chế cốt lõi và các câu hỏi phỏng vấn Java thường gặp.
- [Tổng hợp câu hỏi phỏng vấn Java Basics (phần 1)](./basis/java-basic-questions-01.md): Bao quát đặc điểm ngôn ngữ Java, cú pháp cơ bản, lập trình hướng đối tượng và các class thường dùng.
- [Tổng hợp câu hỏi phỏng vấn Java Basics (phần 2)](./basis/java-basic-questions-02.md): Tiếp tục hệ thống hóa exception, generic, reflection, annotation và các chi tiết thường gặp.
- [Tổng hợp câu hỏi phỏng vấn Java Basics (phần 3)](./basis/java-basic-questions-03.md): Bổ sung kiến thức nền tảng nâng cao và các điểm dễ mắc lỗi.
- [Giải thích chi tiết về pass-by-value trong Java](./basis/why-there-only-value-passing-in-java.md): Làm rõ mối quan hệ giữa pass-by-value, reference variable và việc thay đổi object.
- [Giải thích chi tiết về Java serialization](./basis/serialization.md): Hiểu cơ chế serialization, serialVersionUID, rủi ro bảo mật và các phương án thay thế.
- [Giải thích chi tiết về cơ chế Java reflection](./basis/reflection.md) và [Giải thích chi tiết về Java proxy pattern](./basis/proxy.md): Nắm vững các cơ chế thường gặp bên trong framework.

### Java Collection

- [Chuyên đề Java Collection](./collection/): Tổng hợp collection framework, các lưu ý khi sử dụng và phân tích source code thường gặp.
- [Tổng hợp câu hỏi phỏng vấn Java Collection (phần 1)](./collection/java-collection-questions-01.md) và [Tổng hợp câu hỏi phỏng vấn Java Collection (phần 2)](./collection/java-collection-questions-02.md): Bao quát các câu hỏi thường gặp về List, Set, Map, Queue và concurrent collection.
- [Tổng hợp lưu ý khi sử dụng Java Collection](./collection/java-collection-precautions-for-use.md): Tổng hợp các lưu ý liên quan đến kiểm tra collection rỗng, duyệt, mở rộng, thread safety và performance.
- [Phân tích source code ArrayList](./collection/arraylist-source-code.md), [Phân tích source code HashMap](./collection/hashmap-source-code.md), [Phân tích source code ConcurrentHashMap](./collection/concurrent-hash-map-source-code.md): Hiểu các lựa chọn thiết kế của container thường dùng qua source code.

### Java Concurrency

- [Chuyên đề lập trình Java Concurrency](./concurrent/): Tập trung vào thread, lock, memory model, thread pool và các công cụ concurrency.
- [Tổng hợp câu hỏi phỏng vấn Java Concurrency (phần 1)](./concurrent/java-concurrent-questions-01.md), [Tổng hợp câu hỏi phỏng vấn Java Concurrency (phần 2)](./concurrent/java-concurrent-questions-02.md), [Tổng hợp câu hỏi phỏng vấn Java Concurrency (phần 3)](./concurrent/java-concurrent-questions-03.md): Xây dựng danh sách câu hỏi phỏng vấn về concurrency.
- [Giải thích chi tiết về JMM (Java Memory Model)](./concurrent/jmm.md): Hiểu visibility, atomicity, ordering và happens-before.
- [Giải thích chi tiết về CAS](./concurrent/cas.md), [Giải thích chi tiết về AQS](./concurrent/aqs.md), [Giải thích chi tiết về Java thread pool](./concurrent/java-thread-pool-summary.md): Nắm vững các chủ đề quan trọng về cơ chế bên trong concurrency.
- [Tổng hợp các câu hỏi thường gặp về virtual thread](./concurrent/virtual-thread.md): Hiểu ảnh hưởng của Project Loom đến concurrency model.

### JVM và IO

- [Chuyên đề JVM](./jvm/): Tập trung vào memory, class loading, GC, tham số, công cụ và xử lý sự cố trên production.
- [Giải thích chi tiết về vùng nhớ Java (trọng tâm)](./jvm/memory-area.md): Hiểu program counter, virtual machine stack, native method stack, heap và method area.
- [Giải thích chi tiết về garbage collection của JVM (trọng tâm)](./jvm/jvm-garbage-collection.md): Hiểu cách xác định object còn sống, các thuật toán garbage collection và các garbage collector phổ biến.
- [Giải thích chi tiết về quá trình class loading](./jvm/class-loading-process.md) và [Giải thích chi tiết về class loader (trọng tâm)](./jvm/classloader.md): Nắm vững vòng đời class và mô hình parent delegation.
- [Chuyên đề Java IO](./io/): Trình bày từ BIO, NIO, AIO đến mô hình IO và design pattern về IO.
- [Tổng hợp kiến thức cơ bản về Java IO](./io/io-basis.md), [Tổng hợp kiến thức cốt lõi về Java NIO](./io/nio-basis.md), [Giải thích chi tiết về mô hình Java IO](./io/io-model.md): Bổ sung kiến thức nền tảng cần thiết cho network programming và middleware.

### Tính năng mới của Java

- [Chuyên đề tính năng mới của Java](./new-features/): Tổng hợp theo phiên bản các tính năng quan trọng về ngôn ngữ, standard library và JVM kể từ Java 8.
- [Thực hành các tính năng mới của Java 8](./new-features/java8-common-new-features.md): Nắm vững Lambda, Stream, Optional, default method của interface và Date API mới.
- [Tổng quan tính năng mới của Java 11 (quan trọng)](./new-features/java11.md), [Tổng quan tính năng mới của Java 17 (quan trọng)](./new-features/java17.md), [Tổng quan tính năng mới của Java 21 (quan trọng)](./new-features/java21.md): Ưu tiên các tính năng được hỗ trợ lâu dài trong các phiên bản LTS.

## Câu hỏi thường gặp

- Vì sao Java chỉ truyền giá trị? Điều gì thực sự xảy ra khi reference của object được truyền làm tham số?
- `String`, `StringBuilder`, `StringBuffer` khác nhau như thế nào?
- Mối quan hệ giữa `equals()` và `hashCode()` là gì?
- Nên chọn `ArrayList` hay `LinkedList`? Vì sao `HashMap` không thread-safe?
- `ConcurrentHashMap` đã thay đổi như thế nào giữa JDK 7 và JDK 8?
- `synchronized` và `ReentrantLock` khác nhau như thế nào?
- JMM đảm bảo visibility, ordering và atomicity như thế nào?
- Các tham số cốt lõi của thread pool nên được cấu hình như thế nào? Vì sao không nên sử dụng trực tiếp `Executors`?
- Vùng nhớ JVM được chia như thế nào? Những vùng nào có thể xảy ra OOM?
- G1, ZGC, Shenandoah lần lượt phù hợp với những trường hợp nào?
- BIO, NIO, AIO khác nhau như thế nào? Reactor model giải quyết vấn đề gì?
- Trong Java 8, 11, 17, 21, những tính năng mới nào đáng nắm vững nhất?

## Chuyên đề liên quan

- [Kiến thức cơ bản về máy tính](../cs-basics/)
- [Thiết kế hệ thống](../system-design/)
- [Cơ sở dữ liệu](../database/)
- [Hệ thống kiến thức về hệ thống phân tán](../distributed-system/)
- [Hệ thống kiến thức về hiệu năng cao](../high-performance/)

<!-- @include: @article-footer.snippet.md -->
