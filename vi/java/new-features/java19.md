---
title: Tổng quan các tính năng mới của Java 19
description: Giới thiệu các tính năng preview và cập nhật về concurrency của JDK 19, tạo nền tảng cho virtual thread sau này.
category: Java
tag:
  - Java New Features
head:
  - - meta
    - name: keywords
      content: Java 19,JDK19,virtual thread preview,structured concurrency,Foreign Function API,JEP
---

JDK 19 được phát hành chính thức vào ngày 20 tháng 9 năm 2022, không phải là phiên bản hỗ trợ dài hạn.

JDK 19 có tổng cộng 7 tính năng mới. Bài viết này sẽ chọn một số tính năng mới quan trọng để giới thiệu chi tiết:

- [JEP 424: Foreign Function & Memory API (API hàm và bộ nhớ bên ngoài)](https://openjdk.org/jeps/424) (preview)
- [JEP 425: Virtual Threads (virtual thread)](https://openjdk.org/jeps/425) (preview)
- [JEP 426: Vector API](https://openjdk.java.net/jeps/426) (incubator lần thứ tư)
- [JEP 428: Structured Concurrency](https://openjdk.org/jeps/428) (incubator)

Hình dưới đây thể hiện số lượng tính năng mới và thời gian cập nhật do mỗi phiên bản từ JDK 8 đến JDK 25 mang lại:

![Số lượng tính năng mới và thời gian cập nhật do mỗi phiên bản từ JDK 8 đến JDK 25 mang lại](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

## JEP 424: Foreign Function & Memory API (preview)

Chương trình Java có thể sử dụng API này để tương tác với code và dữ liệu bên ngoài Java runtime. Bằng cách gọi hiệu quả external function (tức code bên ngoài JVM) và truy cập an toàn external memory (tức memory không do JVM quản lý), API này cho phép chương trình Java gọi native library và xử lý native data mà không gặp các rủi ro và tính mong manh như JNI.

Foreign Function & Memory API trải qua vòng incubator đầu tiên trong Java 17, do [JEP 412](https://openjdk.java.net/jeps/412) đề xuất. Vòng incubator thứ hai do [JEP 419](https://openjdk.org/jeps/419) đề xuất và được tích hợp vào Java 18; bản preview do [JEP 424](https://openjdk.org/jeps/424) đề xuất và được tích hợp vào Java 19.

Trước khi có Foreign Function & Memory API:

- Java cung cấp một số method thực hiện các thao tác cấp thấp, không an toàn thông qua [`sun.misc.Unsafe`](https://hg.openjdk.java.net/jdk/jdk/file/tip/src/jdk.unsupported/share/classes/sun/misc/Unsafe.java), chẳng hạn truy cập trực tiếp vào system memory resource và tự quản lý memory resource. Class `Unsafe` cho Java khả năng thao tác trên memory space tương tự pointer của C, nhưng cũng làm tăng tính không an toàn của Java; sử dụng class `Unsafe` không đúng sẽ làm tăng khả năng chương trình gặp lỗi.
- Java đã hỗ trợ gọi native method thông qua Java Native Interface (JNI) từ Java 1.1, nhưng JNI không dễ sử dụng. Việc triển khai JNI quá phức tạp, nhiều bước (có thể tham khảo các bước cụ thể trong bài viết này: [Guide to JNI (Java Native Interface)](https://www.baeldung.com/jni)), không chịu sự kiểm soát của cơ chế bảo đảm an toàn ngôn ngữ của JVM và ảnh hưởng đến tính cross-platform của Java. Ngoài ra, performance của JNI cũng không tốt vì JNI method call không thể hưởng lợi từ nhiều JIT optimization phổ biến, như inline. Mặc dù các framework như [JNA](https://github.com/java-native-access/jna), [JNR](https://github.com/jnr/jnr-ffi) và [JavaCPP](https://github.com/bytedeco/javacpp) đã cải tiến JNI, hiệu quả vẫn chưa thật sự lý tưởng.

Foreign Function & Memory API được giới thiệu để giải quyết một số vấn đề khi Java truy cập external function và external memory.

Foreign Function & Memory API (FFM API) định nghĩa các class và interface:

- Phân bổ external memory: `MemorySegment`, `MemoryAddress` và `SegmentAllocator`
- Thao tác và truy cập external memory có cấu trúc: `MemoryLayout`, `VarHandle`
- Kiểm soát việc phân bổ và giải phóng external memory: `MemorySession`
- Gọi external function: `Linker`, `FunctionDescriptor` và `SymbolLookup`

Dưới đây là ví dụ sử dụng FFM API. Đoạn code này lấy method handle của function `radixsort` trong C library, sau đó dùng nó để sắp xếp bốn string trong mảng Java.

```java
// 1. Tìm external function trong path của C library
Linker linker = Linker.nativeLinker();
SymbolLookup stdlib = linker.defaultLookup();
MethodHandle radixSort = linker.downcallHandle(
                             stdlib.lookup("radixsort"), ...);
// 2. Phân bổ memory trên heap để lưu bốn string
String[] javaStrings   = { "mouse", "cat", "dog", "car" };
// 3. Phân bổ off-heap memory để lưu bốn pointer
SegmentAllocator allocator = implicitAllocator();
MemorySegment offHeap  = allocator.allocateArray(ValueLayout.ADDRESS, javaStrings.length);
// 4. Sao chép string từ heap sang off-heap memory
for (int i = 0; i < javaStrings.length; i++) {
    // Phân bổ một string trong off-heap memory, sau đó lưu pointer trỏ đến nó
    MemorySegment cString = allocator.allocateUtf8String(javaStrings[i]);
    offHeap.setAtIndex(ValueLayout.ADDRESS, i, cString);
}
// 5. Sắp xếp dữ liệu trong off-heap memory bằng cách gọi external function
radixSort.invoke(offHeap, javaStrings.length, MemoryAddress.NULL, '\0');
// 6. Sao chép string (đã được sắp xếp lại) từ off-heap memory về heap
for (int i = 0; i < javaStrings.length; i++) {
    MemoryAddress cStringPtr = offHeap.getAtIndex(ValueLayout.ADDRESS, i);
    javaStrings[i] = cStringPtr.getUtf8String(0);
}
assert Arrays.equals(javaStrings, new String[] {"car", "cat", "dog", "mouse"});  // true
```

## JEP 425: Virtual Threads (preview)

Virtual thread là thread nhẹ do JDK chứ không phải OS triển khai. Nhiều virtual thread chia sẻ cùng một OS thread, vì vậy số lượng virtual thread có thể lớn hơn rất nhiều số lượng OS thread.

Virtual thread đã được chứng minh là rất hữu ích trong các ngôn ngữ multithreading khác, chẳng hạn như Goroutine trong Go và process trong Erlang.

Virtual thread thường không cần tạo hoặc chuyển đổi một OS thread cho mỗi task, nhờ đó giảm resource của thread và scheduling overhead do một lượng lớn blocking task gây ra, đồng thời giảm công sức viết, duy trì và quan sát các concurrent application có throughput cao.

Zhihu có một cuộc thảo luận về virtual thread của Java 19, nếu quan tâm bạn có thể xem: <https://www.zhihu.com/question/536743167>.

Có thể xem phần giải thích chi tiết và nguyên lý của Java virtual thread trong các bài viết dưới đây:

- [Nguyên lý và phân tích performance của virtual thread｜DeWu Technology](https://mp.weixin.qq.com/s/vdLXhZdWyxc6K-D3Aj03LA)
- [Java 19 chính thức GA! Xem virtual thread cải thiện đáng kể throughput của system như thế nào](https://mp.weixin.qq.com/s/yyApBXxpXxVwttr01Hld6Q)
- [Virtual thread - Phân tích source code VirtualThread](https://www.cnblogs.com/throwable/p/16758997.html)

## JEP 426: Vector API (incubator lần thứ tư)

Vector API ban đầu do [JEP 338](https://openjdk.java.net/jeps/338) đề xuất và được tích hợp vào Java 16 dưới dạng [incubator API](http://openjdk.java.net/jeps/11). Vòng incubator thứ hai do [JEP 414](https://openjdk.java.net/jeps/414) đề xuất và được tích hợp vào Java 17; vòng incubator thứ ba do [JEP 417](https://openjdk.java.net/jeps/417) đề xuất và được tích hợp vào Java 18; vòng thứ tư do [JEP 426](https://openjdk.java.net/jeps/426) đề xuất và được tích hợp vào Java 19.

Trong [Tổng quan các tính năng mới của Java 18](./java18.md), tôi đã giới thiệu chi tiết về Vector API, nên ở đây không giới thiệu thêm.

## JEP 428: Structured Concurrency (incubator)

JDK 19 giới thiệu Structured Concurrency, một phương pháp lập trình multithreading nhằm đơn giản hóa việc lập trình nhiều thread thông qua Structured Concurrency API, không nhằm thay thế `java.util.concurrent` và hiện đang ở giai đoạn incubator.

Structured Concurrency xem nhiều task chạy trong các thread khác nhau như một work unit duy nhất, qua đó đơn giản hóa việc xử lý lỗi, nâng cao reliability và tăng observability. Nói cách khác, Structured Concurrency giữ lại readability, maintainability và observability của code single-thread.

API cơ bản của Structured Concurrency là [`StructuredTaskScope`](https://download.java.net/java/early_access/loom/docs/api/jdk.incubator.concurrent/jdk/incubator/concurrent/StructuredTaskScope.html). `StructuredTaskScope` hỗ trợ chia task thành nhiều concurrent subtask, thực thi chúng trong thread riêng và yêu cầu subtask phải hoàn thành trước khi main task tiếp tục.

Cách sử dụng cơ bản của `StructuredTaskScope` như sau:

```java
    try (var scope = new StructuredTaskScope<Object>()) {
        // Dùng method fork để tạo thread thực thi subtask
        Future<Integer> future1 = scope.fork(task1);
        Future<String> future2 = scope.fork(task2);
        // Chờ thread hoàn thành
        scope.join();
        // Xử lý result có thể bao gồm xử lý exception hoặc rethrow exception
        ... process results/exceptions ...
    } // close
```

Structured Concurrency rất phù hợp với virtual thread, một loại thread nhẹ do JDK triển khai. Nhiều virtual thread chia sẻ cùng một OS thread, nhờ đó cho phép có rất nhiều virtual thread.

<!-- @include: @article-footer.snippet.md -->
