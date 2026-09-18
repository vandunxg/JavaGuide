---
title: Tổng quan các tính năng mới của Java 9
description: "Giải thích hệ thống module của Java 9 và các cập nhật như jlink, làm rõ ảnh hưởng đến runtime image và việc sử dụng thư viện."
category: Java
tag:
  - Java New Features
head:
  - - meta
    - name: keywords
      content: Java 9,JDK9,module,JPMS,jlink,collection factory method,new API
---

**Java 9** được phát hành vào ngày 21 tháng 9 năm 2017. Là phiên bản mới được phát hành sau Java 8 tới 3 năm rưỡi, Java 9 mang đến nhiều thay đổi lớn. Trong đó, thay đổi quan trọng nhất là việc giới thiệu Java Platform Module System; ngoài ra còn có các thay đổi về collection, `Stream`...

JDK 9 không phải LTS (bản hỗ trợ dài hạn). Các phiên bản LTS hiện được Oracle liệt kê gồm JDK 8, JDK 11, JDK 17, JDK 21 và JDK 25.

Bài viết này sẽ chọn một số tính năng mới quan trọng để giới thiệu chi tiết:

- [JEP 222: Java Shell Tool (JShell)](https://openjdk.org/jeps/222)
- [JEP 261: Module System (Hệ thống module)](https://openjdk.org/jeps/261)
- [JEP 248: G1 Becomes the Default Garbage Collector (G1 trở thành garbage collector mặc định)](https://openjdk.org/jeps/248)
- [JEP 254: Compact Strings (Chuỗi compact)](https://openjdk.org/jeps/254)
- [JEP 193: Variable Handles (Variable handle)](https://openjdk.org/jeps/193)

Hình dưới đây thể hiện số lượng tính năng mới và thời điểm cập nhật của từng phiên bản từ JDK 8 đến JDK 25:

![](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

## JEP 222: Java Shell Tool (JShell)

JShell là một công cụ thực dụng được bổ sung trong Java 9. Nó cung cấp cho Java một công cụ tương tác dòng lệnh theo thời gian thực tương tự Python.

Trong JShell, bạn có thể nhập trực tiếp biểu thức và xem kết quả thực thi.

![](https://oss.javaguide.cn/java-guide-blog/image-20210816083417616.png)

**JShell mang lại những lợi ích nào?**

1. Hạ thấp rào cản để xuất dòng đầu tiên `"Hello World!"` của Java, giúp tăng hứng thú học tập cho người mới.
2. Khi xử lý logic nhỏ hoặc kiểm tra vấn đề đơn giản, JShell hiệu quả hơn IDE (không nhằm thay thế IDE; với việc kiểm tra logic phức tạp, IDE phù hợp hơn, hai công cụ bổ trợ cho nhau).
3. ……

**Code của JShell khác code thông thường có thể compile ở điểm nào?**

1. Sau khi nhập xong một câu lệnh, JShell sẽ compile và thực thi code ở background, sau đó trả về kết quả ngay, không cần người dùng tự chạy `javac` và `java`.
2. JShell hỗ trợ khai báo lại biến; biến được khai báo sau sẽ ghi đè biến được khai báo trước.
3. JShell hỗ trợ các biểu thức độc lập, chẳng hạn phép cộng thông thường `1 + 1`.
4. ……

## JEP 261: Module System (Hệ thống module)

Hệ thống module là một phần của [Jigsaw Project](https://openjdk.java.net/projects/jigsaw/), đưa thực tiễn phát triển theo module vào Java Platform, giúp code có khả năng tái sử dụng tốt hơn!

**Hệ thống module là gì?** Định nghĩa chính thức là:

> A uniquely named, reusable group of related packages, as well as resources (such as images and XML files) and a module descriptor.

Nói đơn giản, bạn có thể xem một module là một tập hợp các package, resource và module descriptor (`module-info.java`) có tên duy nhất và có thể tái sử dụng.

Bất kỳ file jar nào cũng có thể được nâng cấp thành một module chỉ bằng cách thêm module descriptor (`module-info.java`).

![](https://oss.javaguide.cn/java-guide-blog/module-structure.png)

Sau khi đưa vào hệ thống module, JDK được tổ chức lại thành 94 module. Ứng dụng Java có thể sử dụng **[công cụ jlink](http://openjdk.java.net/jeps/282) mới** (Jlink là công cụ dòng lệnh mới được phát hành cùng Java 9. Nó cho phép developer tạo JRE nhẹ, tùy chỉnh cho các ứng dụng Java dựa trên module) để tạo runtime image tùy chỉnh chỉ chứa các module JDK mà ứng dụng phụ thuộc. Nhờ đó, kích thước Java runtime có thể giảm đáng kể.

Chúng ta có thể dùng keyword `exports` để kiểm soát package nào được mở cho bên ngoài sử dụng và các package đó được mở cho module nào.

```java
module my.module {
    // exports công khai tất cả thành viên public của package được chỉ định
    exports com.my.package.name;
}

module my.module {
    // exports ... to export package được chỉ định cho module được chỉ định
    exports com.my.package.name to com.specific.module;
}
```

Nếu muốn tìm hiểu sâu hơn về module trong Java 9, bạn có thể tham khảo các bài viết sau:

- [《Project Jigsaw: Module System Quick-Start Guide》](https://openjdk.java.net/projects/jigsaw/quick-start)
- [《Java 9 Modules: part 1》](https://stacktraceguru.com/java9/module-introduction)
- [Giải mã Java 9 (2. Hệ thống module)](http://www.cnblogs.com/IcanFixIt/p/6947763.html)

## JEP 248: G1 Becomes the Default Garbage Collector (G1 trở thành garbage collector mặc định)

Trong Java 8, garbage collector mặc định là Parallel Scavenge (young generation) + Parallel Old (old generation). Đến Java 9, CMS garbage collector bị loại bỏ, **G1 (Garbage-First Garbage Collector)** trở thành garbage collector mặc định.

G1 được giới thiệu từ Java 7. Sau khi thể hiện hiệu năng xuất sắc qua hai phiên bản, nó trở thành garbage collector mặc định.

## JEP 193: Variable Handles (Variable handle)

Variable handle là tham chiếu tới một biến hoặc một nhóm biến, bao gồm static field, non-static field, phần tử array và các thành phần trong cấu trúc dữ liệu ngoài heap.

Ý nghĩa của variable handle tương tự method handle đã có là `MethodHandle`, được biểu diễn bởi Java class `java.lang.invoke.VarHandle`. Có thể tạo object `VarHandle` bằng các phương thức lookup của instance `java.lang.invoke.MethodHandles.Lookup` và các cách khác.

Sự xuất hiện của `VarHandle` thay thế một phần thao tác của `java.util.concurrent.atomic` và `sun.misc.Unsafe`. Đồng thời, nó cung cấp một loạt thao tác memory barrier tiêu chuẩn để kiểm soát thứ tự memory ở mức chi tiết hơn. Về tính an toàn, khả dụng và performance, nó đều tốt hơn API hiện có.

## Cải tiến API

Không phải mọi thay đổi API đều được phát hành thông qua JEP (Java Enhancement Proposal).

Trong quy trình phát triển JDK: **JEP** thường được dùng cho các thay đổi lớn, chẳng hạn giới thiệu language feature mới, cơ chế JVM mới hoặc refactor library quy mô lớn. Những thao tác như thêm một vài factory method vào class hiện có, chẳng hạn `List.of()`, thường được xem là bảo trì library thông thường. Chúng được developer JDK trực tiếp submit và review thông qua ticket của **JBS (JDK Bug System)**, sau đó được phát hành cùng phiên bản.

### Cải tiến collection

Bổ sung các factory method như `List.of()`, `Set.of()`, `Map.of()` và `Map.ofEntries()` để tạo immutable collection (có phần giống Guava):

```java
List.of("Java", "C++");
Set.of("Java", "C++");
Map.of("Java", 1, "C++", 2);
```

Collection được tạo bằng `of()` là immutable collection, không thể thực hiện thao tác thêm, xóa, thay thế, sort... Nếu không, exception `java.lang.UnsupportedOperationException` sẽ được ném ra.

### Cải tiến Stream

`Stream` được bổ sung các method mới `ofNullable()`, `dropWhile()`, `takeWhile()` và overload của method `iterate()`.

Method `ofNullable()` trong Java 9 có thể tạo `Stream` một phần tử hoặc rỗng từ một giá trị có thể là `null`. Java 8 đã có thể dùng `Stream.empty()` để tạo stream rỗng, nhưng chưa có method tiện lợi để chuyển trực tiếp giá trị nullable thành stream.

```java
Stream<String> stringStream = Stream.ofNullable("Java");
System.out.println(stringStream.count());// 1
Stream<String> nullStream = Stream.ofNullable(null);
System.out.println(nullStream.count());//0
```

Method `takeWhile()` lần lượt lấy các phần tử thỏa điều kiện từ `Stream`, dừng việc lấy khi gặp phần tử không thỏa điều kiện.

```java
List<Integer> integerList = List.of(11, 33, 66, 8, 9, 13);
integerList.stream().takeWhile(x -> x < 50).forEach(System.out::println);// 11 33
```

Method `dropWhile()` có tác dụng ngược với `takeWhile()`.

```java
List<Integer> integerList2 = List.of(11, 33, 66, 8, 9, 13);
integerList2.stream().dropWhile(x -> x < 50).forEach(System.out::println);// 66 8 9 13
```

Overload mới của method `iterate()` cung cấp một tham số `Predicate` (điều kiện kiểm tra) để quyết định thời điểm kết thúc iteration.

```java
public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f) {
}
// overload mới được bổ sung
public static<T> Stream<T> iterate(T seed, Predicate<? super T> hasNext, UnaryOperator<T> next) {

}
```

So sánh cách sử dụng hai method như dưới đây; overload mới của `iterate()` linh hoạt hơn.

```java
// Dùng method iterate() ban đầu để output các số từ 1~10
Stream.iterate(1, i -> i + 1).limit(10).forEach(System.out::println);
// Dùng overload mới của iterate() để output các số từ 1~10
Stream.iterate(1, i -> i <= 10, i -> i + 1).forEach(System.out::println);
```

### Cải tiến Optional

Class `Optional` được bổ sung các method như `ifPresentOrElse()`, `or()` và `stream()`.

Method `ifPresentOrElse()` nhận hai tham số `Consumer` và `Runnable`. Nếu `Optional` không rỗng, nó gọi tham số `Consumer`; nếu rỗng, nó gọi tham số `Runnable`.

```java
public void ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)

Optional<Object> objectOptional = Optional.empty();
objectOptional.ifPresentOrElse(System.out::println, () -> System.out.println("Empty!!!"));// Empty!!!
```

Method `or()` nhận một tham số `Supplier`. Nếu `Optional` rỗng, nó trả về giá trị `Optional` do tham số `Supplier` chỉ định.

```java
public Optional<T> or(Supplier<? extends Optional<? extends T>> supplier)

Optional<Object> objectOptional = Optional.empty();
objectOptional.or(() -> Optional.of("java")).ifPresent(System.out::println);//java
```

### Cải tiến String

Trong Java 8 và các phiên bản trước, `String` luôn được lưu trữ bằng `char[]`. Từ Java 9, implementation của `String` chuyển sang dùng array `byte[]` để lưu trữ chuỗi, giúp tiết kiệm không gian.

```java
public final class String implements java.io.Serializable,Comparable<String>, CharSequence {
    // Annotation @Stable cho biết biến nhiều nhất chỉ bị sửa một lần, được gọi là "ổn định".
    @Stable
    private final byte[] value;
}
```

### Cải tiến interface

Java 9 cho phép sử dụng private method trong interface. Nhờ đó, việc sử dụng interface linh hoạt hơn, phần nào giống một abstract class được đơn giản hóa.

```java
public interface MyInterface {
    private void methodPrivate(){
    }
}
```

### Cải tiến IO

Trước Java 9, chúng ta chỉ có thể khai báo biến trong block `try-with-resources`:

```java
try (Scanner scanner = new Scanner(new File("testRead.txt"));
    PrintWriter writer = new PrintWriter(new File("testWrite.txt"))) {
    // omitted
}
```

Từ Java 9, có thể sử dụng biến effectively-final trong câu lệnh `try-with-resources`.

```java
final Scanner scanner = new Scanner(new File("testRead.txt"));
PrintWriter writer = new PrintWriter(new File("testWrite.txt"));
try (scanner; writer) {
    // omitted
}
```

**Biến effectively-final là gì?** Nói đơn giản, đó là biến không được modifier `final` nhưng chưa từng thay đổi giá trị sau khi khởi tạo.

Như code trên minh họa, dù biến `writer` không được khai báo rõ ràng là `final`, nó không thay đổi sau lần được gán giá trị đầu tiên, nên là một biến effectively-final.

### Process API

Java 9 bổ sung interface `java.lang.ProcessHandle` để quản lý native process, đặc biệt phù hợp để quản lý process chạy trong thời gian dài.

```java
// Lấy process của JVM hiện đang chạy
ProcessHandle currentProcess = ProcessHandle.current();
// Output id của process
System.out.println(currentProcess.pid());
// Output thông tin của process
System.out.println(currentProcess.info());
```

Tổng quan interface `ProcessHandle`:

![](https://oss.javaguide.cn/java-guide-blog/image-20210816104614414.png)

### Các cải tiến API khác

**Reactive Streams**

Trong class `java.util.concurrent.Flow` của Java 9, các interface cốt lõi của đặc tả reactive stream được bổ sung.

`Flow` bao gồm 4 interface cốt lõi là `Flow.Publisher`, `Flow.Subscriber`, `Flow.Subscription` và `Flow.Processor`. Java 9 cũng cung cấp `SubmissionPublisher` làm một implementation của `Flow.Publisher`.

Để tìm hiểu chi tiết hơn về reactive stream trong Java 9, bạn nên đọc bài viết [Giải mã Java 9 (17. Reactive Streams ) - Lin Bản Thác](https://www.cnblogs.com/IcanFixIt/p/7245377.html).

## Khác

- **Cải tiến platform logging API**: Java 9 cho phép cấu hình cùng một logging implementation cho JDK và application. Bổ sung `System.LoggerFinder` để quản lý logging implementation mà JDK sử dụng. JVM chỉ có một instance `LoggerFinder` trên toàn hệ thống trong thời gian runtime. Chúng ta có thể thêm implementation `System.LoggerFinder` của riêng mình để JDK và application sử dụng các logging framework khác như SLF4J.
- **Cải tiến class `CompletableFuture`**: Bổ sung một số method mới (`completeAsync`, `orTimeout`...).
- **Cải tiến Nashorn engine**: Nashorn là JavaScript engine được giới thiệu từ Java 8. Java 9 cải tiến Nashorn, triển khai một số tính năng mới của ES6 (đã bị deprecated trong Java 11).
- **Tính năng mới của I/O stream**: Bổ sung method mới để đọc và copy dữ liệu trong `InputStream`.
- **Cải thiện security performance của application**: Java 9 bổ sung 4 thuật toán hash SHA-3: SHA3-224, SHA3-256, SHA3-384 và SHA3-512.
- **Cải tiến method handle (Method Handle)**: Method handle được giới thiệu từ Java 7. Java 9 bổ sung thêm nhiều static method trong class `java.lang.invoke.MethodHandles` để tạo các loại method handle khác nhau.
- ……

## Tham khảo

- Java version history：<https://en.wikipedia.org/wiki/Java_version_history>
- Release Notes for JDK 9 and JDK 9 Update Releases : <https://www.oracle.com/java/technologies/javase/9-all-relnotes.html>
- 《Phân tích chuyên sâu các tính năng mới của Java》-Geek Time - JShell: Làm thế nào để nhanh chóng kiểm tra vấn đề đơn giản?
- New Features in Java 9: <https://www.baeldung.com/new-java-9>
- Java – Try with Resources：<https://www.baeldung.com/java-try-with-resources>

<!-- @include: @article-footer.snippet.md -->
