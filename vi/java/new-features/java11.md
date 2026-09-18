---
title: Tổng quan các tính năng mới của Java 11 (quan trọng)
description: Tóm tắt các cập nhật của JDK 11, tập trung vào HTTP client mới, các cải tiến về chuỗi và những tính năng thực tế khác.
category: Java
tag:
  - Tính năng mới của Java
head:
  - - meta
    - name: keywords
      content: Java 11,JDK11,LTS,HTTP client,String API,tính năng bị loại bỏ
---

Java 11 chính thức được phát hành vào ngày 25 tháng 9 năm 2018, đây là một phiên bản rất quan trọng! Java 11 là phiên bản Long-Term-Support đầu tiên sau Java 8. Theo lộ trình hỗ trợ hiện tại của Oracle, Extended Support của Java 11 sẽ kéo dài đến tháng 1 năm 2032.

Hình dưới đây là timeline hỗ trợ Oracle JDK do Oracle chính thức cung cấp.

![Timeline hỗ trợ Oracle JDK do Oracle chính thức cung cấp](https://oss.javaguide.cn/github/javaguide/java/new-features/4c1611fad59449edbbd6e233690e9fa7.png)

Hình dưới đây thể hiện số lượng tính năng mới và thời điểm cập nhật do mỗi phiên bản từ JDK 8 đến JDK 25 mang lại:

![Số lượng tính năng mới và thời điểm cập nhật do mỗi phiên bản từ JDK 8 đến JDK 25 mang lại](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

Bài viết này sẽ chọn một số tính năng mới quan trọng để giới thiệu chi tiết:

- [JEP 321: HTTP Client (Standard)](https://openjdk.org/jeps/321)
- [JEP 323: Local-Variable Syntax for Lambda Parameters](https://openjdk.org/jeps/323)
- [JEP 330: Launch Single-File Source-Code Programs](https://openjdk.org/jeps/330)
- [JEP 333: ZGC: A Scalable Low-Latency Garbage Collector (Experimental)](https://openjdk.org/jeps/333)

## JEP 321: HTTP Client (HTTP client, bản chuẩn)

Java 11 đã chuẩn hóa HTTP Client API được giới thiệu trong Java 9 và cập nhật trong Java 10. Trong thời gian incubate ở hai phiên bản trước, HTTP Client gần như được viết lại hoàn toàn và hiện hỗ trợ đầy đủ asynchronous non-blocking.

Ngoài ra, trong Java 11, package của HTTP Client được đổi từ `jdk.incubator.http` thành `java.net.http`. API này cung cấp semantics request và response non-blocking thông qua `CompletableFuture`. Cách sử dụng cũng rất đơn giản, như sau:

```java
var request = HttpRequest.newBuilder()
    .uri(URI.create("https://javastack.cn"))
    .GET()
    .build();
var client = HttpClient.newHttpClient();

// Đồng bộ
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println(response.body());

// Bất đồng bộ
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
    .thenApply(HttpResponse::body)
    .thenAccept(System.out::println);
```

## JEP 333: ZGC (garbage collector độ trễ thấp, có khả năng mở rộng, experimental)

**ZGC là Z Garbage Collector**, một garbage collector có khả năng mở rộng và độ trễ thấp.

ZGC được thiết kế chủ yếu để đáp ứng các mục tiêu sau:

- Thời gian dừng GC không quá 10ms
- Có thể xử lý cả heap nhỏ vài trăm MB và heap lớn vài TB
- Throughput của ứng dụng không giảm quá 15% (so với thuật toán thu gom G1)
- Tạo nền tảng thuận tiện để đưa vào các tính năng GC mới và tối ưu bằng colored pointers cùng Load barriers
- Hiện chỉ hỗ trợ nền tảng Linux/x64

ZGC hiện **đang ở giai đoạn experimental**, chỉ hỗ trợ nền tảng Linux/x64. Lưu ý: ZGC trở thành tính năng chính thức trong Java 15 và ZGC phân thế hệ được giới thiệu trong Java 21.

Tương tự ParNew và G1 trong CMS, ZGC cũng sử dụng thuật toán mark-copy, nhưng ZGC đã cải tiến đáng kể thuật toán này.

Trong ZGC, tình huống Stop The World xảy ra ít hơn!

Có thể xem chi tiết tại: [《Khám phá và thực tiễn về garbage collector thế hệ mới ZGC》](https://tech.meituan.com/2020/08/06/new-zgc-practice-in-meituan.html)

## JEP 323: Local-Variable Syntax for Lambda Parameters (cú pháp biến cục bộ cho tham số Lambda)

Từ Java 10, Java đã giới thiệu tính năng quan trọng là suy luận kiểu biến cục bộ. Suy luận kiểu cho phép dùng keyword `var` làm kiểu của biến cục bộ thay vì kiểu thực tế; compiler sẽ suy luận kiểu dựa trên giá trị được gán cho biến.

Trong Java 10, keyword `var` có một số giới hạn:

- Chỉ được dùng cho biến cục bộ
- Bắt buộc phải khởi tạo khi khai báo
- Không thể dùng làm tham số method
- Không thể sử dụng trong biểu thức Lambda

Từ Java 11, developer được phép sử dụng `var` để khai báo tham số trong biểu thức Lambda.

```java
// Hai cách dưới đây tương đương nhau
Consumer<String> consumer = (var i) -> System.out.println(i);
Consumer<String> consumer = (String i) -> System.out.println(i);
```

## JEP 330: Launch Single-File Source-Code Programs (khởi chạy chương trình source code một file)

Điều này có nghĩa là chúng ta có thể chạy source code Java chỉ gồm một file. Tính năng này cho phép dùng Java interpreter để thực thi trực tiếp source code Java. Source code được compile trong memory rồi thực thi bởi interpreter, không cần tạo file `.class` trên disk. Ràng buộc duy nhất là tất cả class liên quan phải được định nghĩa trong cùng một file Java.

Tính năng này đặc biệt hữu ích với người mới học Java và muốn thử các chương trình đơn giản. Nó cũng có thể dùng cùng jshell, qua đó phần nào tăng khả năng sử dụng Java để viết script.

## Cải tiến API

Không phải mọi thay đổi API đều được phát hành thông qua JEP (Java Enhancement Proposal).

Trong quy trình phát triển JDK: **JEP** thường được dùng cho những thay đổi lớn, chẳng hạn như giới thiệu language feature mới (ví dụ `var`), cơ chế JVM mới (ví dụ ZGC) hoặc refactor library quy mô lớn. Những thay đổi như thêm một vài method vào class hiện có, chẳng hạn `String.isBlank()`, thường được xem là hoạt động bảo trì library thông thường. Chúng được developer JDK gửi và review trực tiếp thông qua ticket của **JBS (JDK Bug System)**, sau đó phát hành trực tiếp cùng phiên bản.

### Cải tiến String

Java 11 bổ sung một loạt method xử lý chuỗi:

```java
// Kiểm tra chuỗi có trống hay không
" ".isBlank();//true
// Loại bỏ khoảng trắng ở đầu và cuối chuỗi
" Java ".strip();// "Java"
// Loại bỏ khoảng trắng ở đầu chuỗi
" Java ".stripLeading();   // "Java "
// Loại bỏ khoảng trắng ở cuối chuỗi
" Java ".stripTrailing();  // "Java"
// Lặp chuỗi bao nhiêu lần
"Java".repeat(3);             // "JavaJavaJava"
// Trả về collection các chuỗi được phân tách bởi line terminator.
"A\nB\nC".lines().count();    // 3
"A\nB\nC".lines().collect(Collectors.toList());
```

### Cải tiến Optional

Bổ sung method `isEmpty()` để kiểm tra object `Optional` được chỉ định có trống hay không.

```java
var op = Optional.empty();
System.out.println(op.isEmpty());//Kiểm tra object Optional được chỉ định có trống hay không
```

## Các tính năng mới khác

- **Garbage collector mới Epsilon**: một triển khai GC hoàn toàn thụ động, phân bổ tài nguyên memory có giới hạn, giảm tối đa memory usage và memory throughput latency
- **Heap Profiling overhead thấp**: Java 11 cung cấp phương pháp sampling allocation trên Java heap với overhead thấp, có thể lấy thông tin về các Java object được phân bổ trên heap và truy cập thông tin heap thông qua JVMTI
- **Protocol TLS1.3**: Java 11 bao gồm triển khai specification TLS 1.3 (RFC 8446), thay thế TLS có trong các phiên bản trước, bao gồm TLS 1.2. Đồng thời, Java 11 cũng cải tiến các tính năng TLS khác, chẳng hạn OCSP stapling extension (RFC 6066, RFC 6961), session hash và extended master secret extension (RFC 7627), mang lại nhiều cải tiến về security và performance
- **Flight Recorder (Java Flight Recorder)**: trước đây Flight Recorder là một công cụ profiling của JDK bản thương mại, nhưng trong Java 11, code của công cụ này được đưa vào public codebase để mọi người đều có thể sử dụng.
- ......

## Tham khảo

- JDK 11 Release Notes：<https://www.oracle.com/java/technologies/javase/11-relnote-issues.html>
- Java 11 – Features and Comparison：<https://www.geeksforgeeks.org/java-11-features-and-comparison/>

<!-- @include: @article-footer.snippet.md -->
