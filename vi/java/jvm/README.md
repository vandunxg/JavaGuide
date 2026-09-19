---
title: "Chuyên đề JVM: vùng nhớ, class loading, garbage collection, tuning tham số và troubleshooting production"
description: "Lộ trình học phỏng vấn JVM và performance tuning, bao quát vùng nhớ Java, cấu trúc class file, class loading, garbage collection, tham số JVM, công cụ monitoring JDK và troubleshooting production."
category: Java
tag:
  - Java
  - JVM
  - Câu hỏi phỏng vấn Java
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: JVM,câu hỏi phỏng vấn JVM,vùng nhớ Java,class loading,class loader,garbage collection,GC,tham số JVM,công cụ monitoring JDK,OOM,performance tuning
---

JVM là nền tảng cốt lõi mà Java backend không thể bỏ qua. Mục tiêu học JVM không phải là ghi nhớ khái niệm, mà là có thể giải thích object được tạo và thu hồi như thế nào, class được load ra sao, GC ảnh hưởng đến ứng dụng thế nào, tham số được cấu hình ra sao, cũng như cách xử lý các vấn đề production như OOM, GC diễn ra thường xuyên và CPU tăng vọt.

## Dành cho ai

- Java backend developer muốn học JVM một cách có hệ thống.
- Người đang ôn các câu hỏi phỏng vấn liên quan đến vùng nhớ JVM, class loading, GC, tuning tham số và troubleshooting production.
- Độc giả đã từng tham gia bảo trì service production nhưng chưa quen với GC log, heap dump, thread stack và công cụ JDK.
- Kỹ sư muốn tiếp tục đi sâu vào Spring, Netty, middleware hoặc performance optimization.

## Trọng tâm học

- Vùng nhớ runtime của JVM, tạo object, truy cập object và các tình huống OOM.
- Cấu trúc class file, quá trình class loading, class loader và parent delegation model.
- Kiến thức cơ bản về garbage collection, xác định object còn sống, thuật toán garbage collection và các garbage collector phổ biến.
- Tham số JVM, GC log, heap dump, thread stack và các công cụ monitoring, chẩn đoán JDK phổ biến.
- Quy trình cơ bản để xử lý sự cố production: quan sát hiện tượng, thu thập chỉ số, phân tích bằng công cụ, xác định nguyên nhân và kiểm chứng tối ưu hóa.

## Thứ tự đọc đề xuất

1. Khi chuẩn bị phỏng vấn, hãy đọc trước [Tổng hợp câu hỏi phỏng vấn JVM thường gặp](./jvm-interview-questions.md) để tìm ra những phần bạn chưa trả lời đầy đủ; nếu học có hệ thống, bạn có thể bắt đầu từ bước tiếp theo.
2. [Giải thích JVM bằng ngôn ngữ dễ hiểu](./jvm-intro.md): trước tiên có cái nhìn tổng thể về JVM.
3. [Giải thích chi tiết vùng nhớ Java (trọng tâm)](./memory-area.md): hiểu runtime data area và các tình huống OOM thường gặp.
4. [Giải thích chi tiết cấu trúc class file](./class-file-structure.md), [Giải thích chi tiết quá trình class loading](./class-loading-process.md), [Giải thích chi tiết class loader (trọng tâm)](./classloader.md): nắm được quá trình class đi từ `.class` đến object có thể chạy được.
5. [Giải thích chi tiết garbage collection của JVM (trọng tâm)](./jvm-garbage-collection.md): học có hệ thống kiến thức cơ bản về GC, thuật toán và garbage collector.
6. [Tổng hợp các tham số JVM quan trọng nhất](./jvm-parameters-intro.md), [Tổng hợp công cụ monitoring và xử lý sự cố JDK](./jdk-monitoring-and-troubleshooting-tools.md), [Xử lý sự cố production của Java backend](./jvm-in-action.md): bắt đầu với cấu hình tham số và thực tiễn production.

## Bài viết cốt lõi

### Nền tảng JVM và vùng nhớ

- [Tổng hợp câu hỏi phỏng vấn JVM thường gặp](./jvm-interview-questions.md): hệ thống hóa các câu hỏi thường gặp theo vùng nhớ, class loading, GC, tham số, công cụ và xử lý sự cố production.
- [Giải thích JVM bằng ngôn ngữ dễ hiểu](./jvm-intro.md): hiểu vai trò và thành phần của JVM từ góc nhìn tổng thể.
- [Giải thích chi tiết vùng nhớ Java (trọng tâm)](./memory-area.md): trình bày program counter, virtual machine stack, native method stack, heap, method area và direct memory.
- [Giải thích chi tiết cấu trúc class file](./class-file-structure.md): hiểu magic number, version number, constant pool, access flag, field table, method table và attribute table.

### Class loading

- [Giải thích chi tiết quá trình class loading](./class-loading-process.md): hệ thống hóa loading, verification, preparation, resolution, initialization, sử dụng và unloading.
- [Giải thích chi tiết class loader (trọng tâm)](./classloader.md): hiểu bootstrap class loader, extension class loader (JDK 8) / platform class loader (JDK 9+), application class loader và parent delegation model.

### Garbage collection và tuning

- [Giải thích chi tiết garbage collection của JVM (trọng tâm)](./jvm-garbage-collection.md): hiểu cách xác định object còn sống, các loại reference, thuật toán garbage collection, generational collection và các garbage collector phổ biến.
- [Tổng hợp các tham số JVM quan trọng nhất](./jvm-parameters-intro.md): hệ thống hóa các tham số liên quan đến heap size, GC log, OOM dump, garbage collector và chẩn đoán.
- [Tổng hợp công cụ monitoring và xử lý sự cố JDK](./jdk-monitoring-and-troubleshooting-tools.md): giới thiệu các công cụ như jps, jstat, jmap, jstack, jcmd, JConsole, VisualVM, JMC.
- [Xử lý sự cố production của Java backend](./jvm-in-action.md): bắt đầu từ xác nhận cảnh báo, giảm tác động và lưu bằng chứng, rồi lần lượt xử lý CPU, memory, GC, thread pool, connection pool, slow SQL, Redis và message backlog.

## Câu hỏi thường gặp

- Vùng nhớ runtime của JVM được phân chia như thế nào? Những vùng nào là thread-private?
- Heap và method area lần lượt lưu trữ gì? Direct memory có thể OOM không?
- Object được tạo như thế nào? Có những cách nào để định vị object khi truy cập?
- Quá trình class loading có những giai đoạn nào? Giai đoạn initialization được kích hoạt khi nào?
- Parent delegation model là gì? Vì sao cần mô hình này?
- Làm thế nào để xác định object có thể được thu hồi? Strong reference, soft reference, weak reference và phantom reference khác nhau thế nào?
- Minor GC, Major GC và Full GC khác nhau thế nào?
- G1, ZGC và Shenandoah lần lượt phù hợp với những trường hợp sử dụng nào?
- Các tham số JVM thường dùng gồm những gì? Làm thế nào để giữ lại GC log và OOM dump trong production?
- Nên xử lý thế nào khi CPU tăng vọt, Full GC xảy ra thường xuyên hoặc có memory leak?

## Chuyên đề liên quan

- [Hệ thống kiến thức Java](../)
- [Chuyên đề Java Basics](../basis/)
- [Chuyên đề Java Concurrency](../concurrent/)
- [Chuyên đề Java IO](../io/)
- [Hệ điều hành](../../cs-basics/operating-system/)

<!-- @include: @article-footer.snippet.md -->
