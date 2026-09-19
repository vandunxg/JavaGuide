---
title: Tổng quan tính năng mới trong Java 10
description: Tổng quan các cập nhật chính của JDK 10, tập trung giới thiệu suy luận kiểu biến cục bộ với var và các cải tiến khác của nền tảng.
category: Java
tag:
  - Tính năng mới của Java
head:
  - - meta
    - name: keywords
      content: Java 10,JDK10,var suy luận kiểu biến cục bộ,cải tiến garbage collection,hiệu năng
---

**Java 10** được phát hành vào ngày 20 tháng 3 năm 2018. Đây là phiên bản không phải LTS (Long-Term Support), Oracle chỉ cung cấp hỗ trợ trong sáu tháng.

Hình dưới đây thể hiện số lượng tính năng mới và thời điểm cập nhật của từng phiên bản từ JDK 8 đến JDK 25:

![Số lượng tính năng mới và thời điểm cập nhật của từng phiên bản từ JDK 8 đến JDK 25](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

Bài viết này sẽ chọn một số tính năng mới quan trọng để giới thiệu chi tiết:

- [JEP 286: Local-Variable Type Inference (Suy luận kiểu biến cục bộ)](https://openjdk.org/jeps/286)
- [JEP 304: Garbage-Collector Interface (Interface cho garbage collector)](https://openjdk.org/jeps/304)
- [JEP 307: Parallel Full GC for G1 (Full GC song song cho G1)](https://openjdk.org/jeps/307)
- [JEP 310: Application Class-Data Sharing (Chia sẻ dữ liệu class ứng dụng)](https://openjdk.org/jeps/310)
- [JEP 317: Experimental Java-Based JIT Compiler (JIT compiler thử nghiệm dựa trên Java)](https://openjdk.org/jeps/317)

## JEP 286: Local-Variable Type Inference

Do nhiều Java developer mong muốn Java hỗ trợ suy luận kiểu biến cục bộ, tính năng này đã xuất hiện trong Java 10, có thể nói là đáp ứng mong đợi của số đông!

Java 10 cung cấp keyword `var` để khai báo biến cục bộ.

```java
var id = 0;
var codefx = new URL("https://mp.weixin.qq.com/");
var list = new ArrayList<>();
var list = List.of(1, 2, 3);
var map = new HashMap<String, String>();
var p = Paths.of("src/test/java/Java9FeaturesTest.java");
var numbers = List.of("a", "b", "c");
for (var n : numbers)
    System.out.print(n+ " ");
```

`var` chỉ có thể được dùng trong khai báo biến cục bộ có initializer. Nó cũng có thể được dùng cho biến cục bộ trong vòng lặp `for` cơ bản hoặc nâng cao, cũng như biến resource của `try`-with-resources. Nó không thể được dùng cho field, tham số method hoặc kiểu trả về.

```java
var count = null; //❌Không thể biên dịch, không thể khai báo là null
var r = () -> Math.random();//❌Không thể biên dịch, không thể khai báo là biểu thức Lambda
var array = {1, 2, 3};//❌Không thể biên dịch, không thể khai báo array
```

`var` không làm thay đổi sự thật rằng Java là một ngôn ngữ kiểu tĩnh; compiler chịu trách nhiệm suy luận kiểu.

Ngoài ra, Scala và Kotlin đã có keyword `val` (một tổ hợp tương đương `final var`).

## JEP 304: Garbage-Collector Interface

Trong cấu trúc JDK thời kỳ đầu, các component tạo nên implementation của garbage collector (GC) nằm rải rác ở nhiều phần trong codebase. Java 10 đã tách source code của các garbage collector khác nhau bằng cách giới thiệu một interface thuần túy cho garbage collector.

## JEP 307: Parallel Full GC for G1

Từ Java 9, G1 đã trở thành garbage collector mặc định. G1 được thiết kế là một garbage collector có latency thấp, nhằm tránh thực hiện Full GC. Tuy nhiên, Full GC của G1 trong Java 9 vẫn dùng một thread để hoàn tất thuật toán mark-sweep, điều này có thể khiến garbage collector trigger Full GC khi không thể thu hồi memory.

Để giảm thời gian pause của ứng dụng do Full GC, từ Java 10, Full GC của G1 chuyển sang dùng nhiều thread worker song song để thực hiện mark, sweep và compact. Thay đổi này rút ngắn thời gian pause của Full GC, nhưng không trực tiếp làm giảm số lần trigger Full GC.

## JEP 310: **Chia sẻ dữ liệu class ứng dụng (mở rộng tính năng CDS)**

Java 5 đã giới thiệu cơ chế chia sẻ dữ liệu class (Class Data Sharing, viết tắt là CDS), cho phép tiền xử lý một nhóm system class thành shared archive để memory mapping trong runtime, từ đó giảm thời gian khởi động của chương trình Java và memory usage của nhiều JVM. AppCDS, cho phép thêm application class vào shared archive, trước đây chỉ được cung cấp như một tính năng thương mại trong Oracle JDK.

Trên cơ sở tính năng CDS hiện có, Java 10 tiếp tục mở rộng và mở AppCDS, cho phép đưa application class vào shared archive. Quy trình điển hình là trước tiên tạo danh sách application class, sau đó tạo shared archive dựa trên danh sách class; khi khởi động lần sau, archive được load thông qua memory mapping. Bản thân văn bản chứa danh sách class không phải là cache được JVM load trực tiếp trong các lần khởi động sau.

## JEP 317: **JIT compiler thử nghiệm dựa trên Java**

Graal là một JIT compiler được viết bằng ngôn ngữ Java, đồng thời là nền tảng của AOT (Ahead-of-Time) compiler thử nghiệm được giới thiệu trong JDK 9.

Oracle HotSpot VM đi kèm hai JIT compiler được triển khai bằng C++: C1 và C2. Trong Java 10 (Linux/x64, macOS/x64), theo mặc định HotSpot vẫn sử dụng C2, nhưng có thể thay C2 bằng Graal bằng cách thêm các tham số `-XX:+UnlockExperimentalVMOptions -XX:+UseJVMCICompiler` vào lệnh java.

## Cải tiến API

Không phải mọi thay đổi API đều được phát hành thông qua JEP (Java Enhancement Proposal).

Trong quy trình phát triển JDK, **JEP** thường được dùng cho các thay đổi lớn, chẳng hạn như giới thiệu tính năng ngôn ngữ mới (như `var`), cơ chế JVM mới (như ZGC) hoặc refactor library quy mô lớn. Những thay đổi như thêm một vài static method vào class hiện có, chẳng hạn `List.copyOf()`, thường được xem là bảo trì library thông thường. Chúng được JDK developer trực tiếp submit và review thông qua ticket của **JBS (JDK Bug System)**, sau đó được phát hành trực tiếp cùng phiên bản.

### Cải tiến Collection

`List`, `Set`, `Map` cung cấp static method `copyOf()` để trả về một bản copy immutable của collection đầu vào.

```java
static <E> List<E> copyOf(Collection<? extends E> coll) {
    return ImmutableCollections.listCopy(coll);
}
```

Collection được tạo bằng `copyOf()` là immutable collection, không thể thực hiện các thao tác thêm, xoá, thay thế, sort và các thao tác khác, nếu không sẽ phát sinh exception `java.lang.UnsupportedOperationException`. IDEA cũng sẽ hiển thị gợi ý tương ứng.

![Collection được tạo bằng `copyOf()` là immutable collection](https://oss.javaguide.cn/java-guide-blog/image-20210816154125579.png)

Ngoài ra, `java.util.stream.Collectors` được bổ sung các static method để thu thập các phần tử trong stream thành immutable collection.

```java
var list = new ArrayList<>();
list.stream().collect(Collectors.toUnmodifiableList());
list.stream().collect(Collectors.toUnmodifiableSet());
```

### Cải tiến Optional

`Optional` bổ sung method `orElseThrow()` không có tham số. Đây là phiên bản rút gọn của `orElseThrow(Supplier<? extends X> exceptionSupplier)` có tham số; khi không có value, method này mặc định throw exception NoSuchElementException.

```java
Optional<String> optional = Optional.empty();
String result = optional.orElseThrow();
```

## Khác

- **Thread-local handshake**: Trong Java 10, cơ chế quản lý thread đưa vào khái niệm safepoint của JVM, cho phép thực hiện callback của thread mà không cần chạy safepoint toàn cục của JVM. Callback có thể được thực hiện bởi chính thread hoặc JVM thread, đồng thời thread vẫn ở trạng thái blocking. Cách này giúp có thể stop một thread riêng lẻ, thay vì chỉ có thể bật hoặc stop toàn bộ thread.
- **Phân bổ heap trên thiết bị lưu trữ thay thế**: Java 10 cho phép JVM sử dụng heap phù hợp với các loại cơ chế lưu trữ khác nhau và phân bổ heap memory trên các thiết bị memory tùy chọn.
- ……

## Tham khảo

- Java 10 Features and Enhancements : <https://howtodoinjava.com/java10/java10-features/>

- Guide to Java10 : <https://www.baeldung.com/java-10-overview>

- 4 Class Data Sharing : <https://docs.oracle.com/javase/10/vm/class-data-sharing.htm#JSJVM-GUID-7EAA3411-8CF0-4D19-BD05-DF5E1780AA91>

<!-- @include: @article-footer.snippet.md -->
