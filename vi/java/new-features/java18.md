---
title: Tổng quan các tính năng mới của Java 18
description: "Tổng quan các cập nhật và tính năng preview của JDK 18, tìm hiểu những cải tiến mà API mới mang lại."
category: Java
tag:
  - Java New Features
head:
  - - meta
    - name: keywords
      content: Java 18,JDK18,tính năng preview,cập nhật API,JEP
---

Java 18 được phát hành chính thức vào ngày 22 tháng 3 năm 2022, không phải là phiên bản hỗ trợ dài hạn.

JDK 18 có tổng cộng 8 tính năng mới. Bài viết này chọn một số tính năng mới quan trọng để giới thiệu chi tiết:

- [JEP 400: UTF-8 by Default (UTF-8 làm character set mặc định)](https://openjdk.java.net/jeps/400)
- [JEP 408: Simple Web Server (Web server tĩnh đơn giản)](https://openjdk.java.net/jeps/408)
- [JEP 413: Code Snippets in Java API Documentation (code snippet trong tài liệu API)](https://openjdk.java.net/jeps/413)
- [JEP 416: Reimplement Core Reflection with Method Handles (tái triển khai Core Reflection bằng Method Handles)](https://openjdk.java.net/jeps/416)
- [JEP 417: Vector API (Third Incubator) (Vector API, incubator lần thứ ba)](https://openjdk.java.net/jeps/417)
- [JEP 418: Internet-Address Resolution SPI (SPI phân giải địa chỉ Internet)](https://openjdk.java.net/jeps/418)
- [JEP 419: Foreign Function & Memory API (Second Incubator) (Foreign Function & Memory API, incubator lần thứ hai)](https://openjdk.java.net/jeps/419)

Hình dưới đây cho biết số lượng tính năng mới và thời điểm cập nhật của từng phiên bản từ JDK 8 đến JDK 25:

![Số lượng tính năng mới và thời điểm cập nhật của từng phiên bản từ JDK 8 đến JDK 25](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

Bài đọc liên quan:

- [Tài liệu OpenJDK Java 18](https://openjdk.java.net/projects/jdk/18/)
- [IntelliJ IDEA | Hỗ trợ tính năng Java 18](https://mp.weixin.qq.com/s/PocFKR9z9u7-YCZHsrA5kQ)

## JEP 400: UTF-8 by Default (UTF-8 làm character set mặc định, finalized)

Cuối cùng, JDK đã đặt UTF-8 làm character set mặc định.

Trong Java 17 và các phiên bản trước, character set mặc định chỉ được xác định khi JVM chạy, phụ thuộc vào hệ điều hành, locale và các yếu tố khác, nên tồn tại rủi ro tiềm ẩn. Ví dụ, một chương trình Java in văn bản ra console có thể chạy bình thường trên Mac nhưng lại hiển thị ký tự lỗi trên Windows nếu bạn không tự thay đổi character set.

## JEP 408: Simple Web Server (Web server tĩnh đơn giản, finalized)

Kể từ Java 18, bạn có thể dùng lệnh `jwebserver` để khởi động một Web server tĩnh đơn giản.

```bash
$ jwebserver
Binding to loopback by default. For all interfaces use "-b 0.0.0.0" or "-b ::".
Serving /cwd and subdirectories on 127.0.0.1 port 8000
URL: http://127.0.0.1:8000/
```

Server này không hỗ trợ CGI và Servlet, chỉ phục vụ file tĩnh.

## JEP 413: Code Snippets in Java API Documentation (code snippet trong tài liệu API, finalized)

Trước Java 18, nếu muốn đưa code snippet vào Javadoc, bạn có thể dùng `<pre>{@code ...}</pre>`.

```java
<pre>{@code
    lines of source code
}</pre>
```

Cách tạo code snippet bằng `<pre>{@code ...}</pre>` cho kết quả hiển thị tương đối bình thường.

Từ Java 18, bạn có thể dùng tag `@snippet` để thực hiện việc này.

```java
/**
 * The following code shows how to use {@code Optional.isPresent}:
 * {@snippet :
 * if (v.isPresent()) {
 *     System.out.println("v: " + v.get());
 * }
 * }
 */
```

Cách dùng `@snippet` cho kết quả tốt hơn và thuận tiện hơn.

## JEP 416: Reimplement Core Reflection with Method Handles (tái triển khai core reflection bằng Method Handles, finalized)

Java 18 cải thiện logic triển khai của `java.lang.reflect.Method` và `Constructor`, giúp cải thiện performance và tốc độ. Thay đổi này không sửa các API liên quan. Điều đó có nghĩa là trong quá trình phát triển, bạn không cần sửa code reflection liên quan mà vẫn có thể sử dụng reflection với performance tốt hơn.

OpenJDK cung cấp kết quả benchmark về performance của reflection trong triển khai mới và cũ.

![Kết quả benchmark về performance của reflection trong triển khai mới và cũ](https://oss.javaguide.cn/github/javaguide/java/new-features/JEP416Benchmark.png)

## JEP 417: Vector API (Vector API, incubator lần thứ ba)

Vector API lần đầu được đề xuất trong [JEP 338](https://openjdk.java.net/jeps/338), sau đó được tích hợp vào Java 16 dưới dạng [incubator API](http://openjdk.java.net/jeps/11). Vòng incubator thứ hai được đề xuất trong [JEP 414](https://openjdk.java.net/jeps/414) và tích hợp vào Java 17. Vòng incubator thứ ba được đề xuất trong [JEP 417](https://openjdk.java.net/jeps/417) và tích hợp vào Java 18. Vòng thứ tư được đề xuất trong [JEP 426](https://openjdk.java.net/jeps/426) và tích hợp vào Java 19.

Phép tính vector gồm một chuỗi thao tác trên vector. Vector API dùng để biểu đạt phép tính vector, phép tính này có thể được compile một cách đáng tin cậy thành các instruction vector tối ưu cho kiến trúc CPU được hỗ trợ tại runtime, từ đó đạt performance tốt hơn phép tính scalar tương đương.

Vector API hướng đến việc cung cấp cho người dùng một cách biểu đạt đơn giản, dễ dùng và độc lập với platform cho nhiều dạng phép tính vector.

Đây là phép tính scalar đơn giản trên các phần tử của array:

```java
void scalarComputation(float[] a, float[] b, float[] c) {
   for (int i = 0; i < a.length; i++) {
        c[i] = (a[i] * a[i] + b[i] * b[i]) * -1.0f;
   }
}
```

Đây là phép tính vector tương đương sử dụng Vector API:

```java
static final VectorSpecies<Float> SPECIES = FloatVector.SPECIES_PREFERRED;

void vectorComputation(float[] a, float[] b, float[] c) {
    int i = 0;
    int upperBound = SPECIES.loopBound(a.length);
    for (; i < upperBound; i += SPECIES.length()) {
        // FloatVector va, vb, vc;
        var va = FloatVector.fromArray(SPECIES, a, i);
        var vb = FloatVector.fromArray(SPECIES, b, i);
        var vc = va.mul(va)
                   .add(vb.mul(vb))
                   .neg();
        vc.intoArray(c, i);
    }
    for (; i < a.length; i++) {
        c[i] = (a[i] * a[i] + b[i] * b[i]) * -1.0f;
    }
}

```

Trong JDK 18, performance của Vector API tiếp tục được tối ưu.

## JEP 418: Internet-Address Resolution SPI (SPI phân giải địa chỉ Internet, finalized)

Java 18 định nghĩa một SPI (service-provider interface) hoàn toàn mới, chủ yếu dùng để phân giải tên host và địa chỉ, để `java.net.InetAddress` có thể sử dụng resolver của bên thứ ba ngoài platform.

## JEP 419: Foreign Function & Memory API (Foreign Function & Memory API, incubator lần thứ hai)

Chương trình Java có thể dùng API này để tương tác với code và data bên ngoài Java runtime. Bằng cách gọi foreign function hiệu quả (tức code bên ngoài JVM) và truy cập foreign memory an toàn (tức memory không do JVM quản lý), API này cho phép chương trình Java gọi native library và xử lý native data mà an toàn và ổn định hơn JNI.

Foreign Function & Memory API trải qua vòng incubator đầu tiên trong Java 17, được đề xuất trong [JEP 412](https://openjdk.java.net/jeps/412). Vòng incubator thứ hai được đề xuất trong [JEP 419](https://openjdk.org/jeps/419) và tích hợp vào Java 18. Phiên bản preview được đề xuất trong [JEP 424](https://openjdk.org/jeps/424) và tích hợp vào Java 19.

Trong [Tổng quan các tính năng mới của Java 19](./java19.md), tôi đã giới thiệu chi tiết về Foreign Function & Memory API, nên ở đây sẽ không giới thiệu thêm.

<!-- @include: @article-footer.snippet.md -->
