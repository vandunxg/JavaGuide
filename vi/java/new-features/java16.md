---
title: Tổng quan các tính năng mới của Java 16
description: "Giới thiệu các cập nhật về ngôn ngữ và nền tảng của JDK 16, bao gồm record class và các thay đổi JEP khác."
category: Java
tag:
  - Java New Features
head:
  - - meta
    - name: keywords
      content: Java 16,JDK16,cải tiến record class,API mới,JEP,performance
---

Java 16 được phát hành chính thức vào ngày 16 tháng 3 năm 2021, là phiên bản không hỗ trợ dài hạn (LTS).

JDK 16 có tổng cộng 17 tính năng mới. Bài viết này chọn một số tính năng mới quan trọng để giới thiệu chi tiết:

- [JEP 338: Vector API (Incubator) (Vector API, đợt incubator đầu tiên)](https://openjdk.java.net/jeps/338)
- [JEP 376: ZGC: Concurrent Thread-Stack Processing (ZGC xử lý đồng thời thread stack)](https://openjdk.java.net/jeps/376)
- [JEP 387: Elastic Metaspace (Metaspace co giãn)](https://openjdk.java.net/jeps/387)
- [JEP 390: Warnings for Value-Based Classes (cảnh báo cho value-based class)](https://openjdk.java.net/jeps/390)
- [JEP 394: Pattern Matching for instanceof (pattern matching của instanceof, chính thức)](https://openjdk.java.net/jeps/394)
- [JEP 395: Records (record class, chính thức)](https://openjdk.java.net/jeps/395)
- [JEP 396: Strongly Encapsulate JDK Internals by Default (mặc định áp dụng encapsulation nghiêm ngặt cho thành phần nội bộ JDK)](https://openjdk.java.net/jeps/396)
- [JEP 397: Sealed Classes (sealed class, đợt preview thứ hai)](https://openjdk.java.net/jeps/397)

Hình dưới đây thể hiện số lượng tính năng mới và thời điểm cập nhật của từng phiên bản từ JDK 8 đến JDK 25:

![Số lượng tính năng mới và thời điểm cập nhật của từng phiên bản từ JDK 8 đến JDK 25](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

Đọc thêm: [Tài liệu OpenJDK Java 16](https://openjdk.java.net/projects/jdk/16/).

## JEP 338: Vector API (Vector API, đợt incubator đầu tiên)

Vector API lần đầu được đề xuất trong [JEP 338](https://openjdk.java.net/jeps/338) và được tích hợp vào Java 16 dưới dạng [incubator API](http://openjdk.java.net/jeps/11). Đợt incubator thứ hai được đề xuất trong [JEP 414](https://openjdk.java.net/jeps/414) và tích hợp vào Java 17, đợt incubator thứ ba được đề xuất trong [JEP 417](https://openjdk.java.net/jeps/417) và tích hợp vào Java 18, đợt thứ tư được đề xuất trong [JEP 426](https://openjdk.java.net/jeps/426) và tích hợp vào Java 19.

Incubator API này cung cấp phiên bản ban đầu của API để biểu diễn một số phép tính vector. Các phép tính này được biên dịch đáng tin cậy tại runtime thành các instruction vector phần cứng tối ưu trên kiến trúc CPU được hỗ trợ, nhờ đó đạt performance tốt hơn phép tính scalar tương đương và tận dụng đầy đủ kỹ thuật Single Instruction Multiple Data (SIMD), một loại instruction có thể sử dụng trên hầu hết CPU hiện đại. Dù HotSpot hỗ trợ vectorization tự động, tập hợp các phép toán scalar có thể chuyển đổi còn hạn chế và dễ bị ảnh hưởng bởi thay đổi code. API này giúp developer dễ dàng viết các thuật toán vector có performance cao và portable bằng Java.

Trong [Tổng quan các tính năng mới của Java 18](./java18.md), tôi đã giới thiệu chi tiết về Vector API nên sẽ không giới thiệu thêm ở đây.

## JEP 347: Enable C++ 14 Language Features (bật các tính năng ngôn ngữ C++ 14)

Java 16 cho phép sử dụng các tính năng ngôn ngữ C++ 14 trong source code C++ của JDK, đồng thời cung cấp hướng dẫn cụ thể về những tính năng có thể sử dụng trong code HotSpot.

Trong Java 15, các tính năng ngôn ngữ được sử dụng trong code C++ của JDK chỉ giới hạn ở tiêu chuẩn ngôn ngữ C++98/03. Điều này yêu cầu cập nhật phiên bản tối thiểu được chấp nhận của compiler trên nhiều platform.

## JEP 376: ZGC: Concurrent Thread-Stack Processing (ZGC xử lý đồng thời thread stack)

Java 16 chuyển việc xử lý thread stack của ZGC từ safe point sang một giai đoạn concurrent, cho phép thời gian tạm dừng tại safe point của GC chỉ còn vài mili giây ngay cả trên heap lớn. Việc loại bỏ nguồn latency cuối cùng trong garbage collector ZGC có thể cải thiện đáng kể performance và hiệu quả của ứng dụng.

## JEP 387: Elastic Metaspace (Metaspace co giãn)

Kể từ khi Metaspace được giới thiệu, theo phản hồi nhận được, Metaspace thường chiếm quá nhiều memory ngoài heap, gây lãng phí memory. Tính năng Metaspace co giãn có thể trả phần memory metadata của class HotSpot chưa sử dụng, tức memory trong metaspace, về operating system nhanh hơn, từ đó giảm dung lượng Metaspace.

Ngoài ra, đề xuất này còn đơn giản hóa code của Metaspace để giảm chi phí bảo trì.

## JEP 390: Warnings for Value-Based Classes (cảnh báo cho value-based class)

> Phần giới thiệu dưới đây được trích từ: [Thực hành | Phân tích cú pháp mới của Java 16](https://xie.infoq.cn/article/8304c894c4e38318d38ceb116), bài viết gốc rất hay và đáng đọc.

Ngay từ Java 9, các nhà thiết kế Java đã nâng cấp annotation `@Deprecated`, bổ sung hai element mới là `since` và `forRemoval`. Element `since` dùng để chỉ định phiên bản mà API bị đánh dấu bằng annotation `@Deprecated`, còn `forRemoval` làm rõ hơn ngữ nghĩa của việc deprecated: nếu `forRemoval=true`, điều đó có nghĩa API chắc chắn sẽ bị xóa trong phiên bản tương lai và developer nên dùng API mới để thay thế. Cách này tránh gây hiểu nhầm như trước (trước Java 9, API được đánh dấu bằng annotation `@Deprecated` có thể mang nhiều ngữ nghĩa, chẳng hạn có rủi ro khi sử dụng, có thể phát sinh lỗi tương thích trong tương lai, có thể bị xóa trong phiên bản tương lai hoặc nên dùng phương án thay thế tốt hơn).

Quan sát kỹ các wrapper class của primitive type (ví dụ: `java.lang.Integer`, `java.lang.Double`), có thể thấy constructor của chúng đều đã được đánh dấu bằng annotation `@Deprecated(since="9", forRemoval = true)`. Điều này có nghĩa các constructor này dự kiến sẽ bị xóa trong tương lai, không nên tiếp tục dùng cách viết như `new Integer(10)` trong chương trình (nên dùng `Integer a = 10` hoặc method `Integer.valueOf()`). Nếu tiếp tục sử dụng, compiler sẽ sinh cảnh báo `'Integer(int)' is deprecated and marked for removal`. Ngoài ra, các wrapper type này, cũng như `java.util.Optional` và `java.time.LocalDateTime`, đã được chỉ định là value-based class, không được nhầm lẫn với value type trong Project Valhalla.

Tiếp theo, khi sử dụng instance của value-based class trong khối `synchronized`, compiler của JDK 16 sẽ sinh cảnh báo. HotSpot cũng có thể ghi log hoặc ngăn chặn kiểu đồng bộ hóa này tại runtime thông qua diagnostic option. Ngay cả khi không bật diagnostic tại runtime, cũng không nên dùng instance của value-based class làm lock, ví dụ:

```java
private Integer count = 0;

public void inc() {
    synchronized (count) {
        count++;
    }
}
```

`Integer` là immutable object. `count++` sẽ thực hiện unboxing, tăng một đơn vị rồi boxing lại, đồng thời đổi field reference sang một instance `Integer` khác. Vì vậy, các thread khác nhau có thể lock các object khác nhau, không thể bảo đảm mutual exclusion ổn định cho phép toán kết hợp này. Nếu cần tự tăng đồng thời, nên dùng một lock object cố định hoặc `AtomicInteger`.

## JEP 392: Packaging Tool (công cụ đóng gói, chính thức)

Trong Java 14, JEP 343 giới thiệu công cụ đóng gói với command `jpackage`. Trong Java 15, công cụ này tiếp tục ở giai đoạn incubator, còn trong Java 16 cuối cùng đã trở thành tính năng chính thức.

Công cụ đóng gói này cho phép đóng gói ứng dụng Java tự chứa. Nó hỗ trợ các định dạng đóng gói native, cung cấp trải nghiệm cài đặt tự nhiên cho người dùng cuối. Các định dạng gồm msi và exe trên Windows, pkg và dmg trên macOS, cùng deb và rpm trên Linux. Công cụ cũng cho phép chỉ định parameter khi khởi động trong lúc đóng gói, có thể được gọi trực tiếp từ command line hoặc gọi theo cách lập trình thông qua ToolProvider API. Lưu ý tên module của jpackage đã đổi từ jdk.incubator.jpackage thành jdk.jpackage. Điều này sẽ cải thiện trải nghiệm của người dùng cuối khi cài đặt ứng dụng và đơn giản hóa việc deploy theo mô hình “app store”.

Để xem cách sử dụng thực tế công cụ đóng gói này, bạn có thể xem video [Playing with Java 16 jpackage](https://www.youtube.com/watch?v=KahYIVzRIkQ) (cần VPN).

## JEP 393: Foreign Memory Access API (Foreign Memory Access API, đợt incubator thứ ba)

Foreign Memory Access API được giới thiệu để cho phép chương trình Java truy cập an toàn và hiệu quả vào memory bên ngoài heap Java.

Java 14 ([JEP 370](https://openjdk.org/jeps/370)) lần đầu đưa Foreign Memory Access API vào giai đoạn incubator, Java 15 thực hiện đợt incubator thứ hai ([JEP 383](https://openjdk.org/jeps/383)), còn Java 16 thực hiện đợt incubator thứ ba.

Mục tiêu của việc giới thiệu Foreign Memory Access API:

- Tổng quát: một API duy nhất phải có khả năng thao tác với nhiều loại memory bên ngoài như native memory, persistent memory và heap memory.
- An toàn: bất kể thao tác với loại memory nào, API không được phá vỡ tính an toàn của JVM.
- Kiểm soát: có thể tự do lựa chọn cách giải phóng memory (explicit, implicit, v.v.).
- Dễ sử dụng: với chương trình cần truy cập memory bên ngoài, API nên cung cấp một giải pháp dễ dùng, đủ để thay thế `sun.misc.Unsafe`.

## JEP 394: Pattern Matching for instanceof (pattern matching của instanceof, chính thức)

| Phiên bản JDK | Loại cập nhật     | JEP                                     | Nội dung cập nhật                                                       |
| ------------- | ----------------- | --------------------------------------- | ----------------------------------------------------------------------- |
| Java SE 14    | preview           | [JEP 305](https://openjdk.org/jeps/305) | Lần đầu giới thiệu pattern matching của instanceof.                     |
| Java SE 15    | Second Preview    | [JEP 375](https://openjdk.org/jeps/375) | Không thay đổi so với phiên bản trước, tiếp tục thu thập thêm phản hồi. |
| Java SE 16    | Permanent Release | [JEP 394](https://openjdk.org/jeps/394) | Pattern variable không còn mặc định là final.                           |

Từ Java 16, bạn có thể thay đổi giá trị của pattern variable trong `instanceof`.

```java
// Old code
if (o instanceof String) {
    String s = (String)o;
    ... use s ...
}

// New code
if (o instanceof String s) {
    ... use s ...
}
```

## JEP 395: Records (record class, chính thức)

Lịch sử thay đổi của record type:

| Phiên bản JDK | Loại cập nhật     | JEP                                          | Nội dung cập nhật                                                                                         |
| ------------- | ----------------- | -------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| Java SE 14    | Preview           | [JEP 359](https://openjdk.java.net/jeps/359) | Giới thiệu keyword `record`; `record` cung cấp cú pháp ngắn gọn để định nghĩa data immutable trong class. |
| Java SE 15    | Second Preview    | [JEP 384](https://openjdk.org/jeps/384)      | Hỗ trợ sử dụng `record` trong method cục bộ và interface.                                                 |
| Java SE 16    | Permanent Release | [JEP 395](https://openjdk.org/jeps/395)      | Non-static inner class có thể định nghĩa static member không phải constant.                               |

Từ Java SE 16, non-static inner class có thể định nghĩa static member không phải constant.

```java
public class Outer {
  class Inner {
    static int age;
  }
}
```

> Trước JDK 16, nếu viết code như trên, IDE sẽ thông báo static field age không thể được định nghĩa trong non-static inner class, trừ khi được khởi tạo bằng constant expression. (The field age cannot be declared static in a non-static inner type, unless initialized with a constant expression)

## JEP 396: Strongly Encapsulate JDK Internals by Default (mặc định áp dụng encapsulation nghiêm ngặt cho thành phần nội bộ JDK)

Tính năng này mặc định áp dụng encapsulation nghiêm ngặt cho tất cả thành phần bên trong JDK, ngoại trừ các internal API quan trọng như `sun.misc.Unsafe`. Theo mặc định, code truy cập internal API của JDK vốn biên dịch thành công trên phiên bản cũ có thể không còn hoạt động. Developer được khuyến khích chuyển từ việc sử dụng thành phần bên trong sang API chuẩn, để họ và user có thể nâng cấp liền mạch lên các phiên bản Java tương lai. Encapsulation nghiêm ngặt được điều khiển bởi launcher option `–illegal-access` từ JDK 9; đến JDK 15, mặc định được đổi thành warning; từ JDK 16, mặc định là deny. Hiện tại vẫn có thể dùng một command-line option để nới lỏng encapsulation đối với tất cả package, nhưng trong tương lai chỉ có thể dùng `–add-opens` để mở các package cụ thể.

## JEP 397: Sealed Classes (sealed class, đợt preview thứ hai)

Sealed class được đề xuất preview bởi [JEP 360](https://openjdk.java.net/jeps/360) và tích hợp vào Java 15. Trong JDK 16, sealed class được cải tiến với việc kiểm tra reference chặt chẽ hơn và quan hệ inheritance của sealed class, đồng thời được [JEP 397](https://openjdk.java.net/jeps/397) đề xuất preview lần nữa.

Trong [Tổng quan các tính năng mới của Java 14 & 15](./java14-15.md), tôi đã giới thiệu chi tiết về sealed class nên sẽ không giới thiệu thêm ở đây.

## Các tối ưu hóa và cải tiến khác

- **JEP 380: Unix-Domain socket channel**: Unix-domain socket vốn là tính năng của hầu hết platform Unix, nay cũng được hỗ trợ trên Windows 10 và Windows Server 2019. Tính năng này bổ sung hỗ trợ Unix-domain (AF_UNIX) socket cho API socket channel và server socket channel của package java.nio.channels. Nó mở rộng cơ chế channel kế thừa để hỗ trợ Unix-domain socket channel và server socket channel. Unix-domain socket được dùng cho inter-process communication (IPC) trên cùng một host. Chúng phần lớn tương tự TCP/IP, khác ở chỗ socket được định danh bằng pathname của file system thay vì địa chỉ Internet Protocol (IP) và port. Với inter-process communication cục bộ, Unix-domain socket an toàn và hiệu quả hơn kết nối loopback TCP/IP.
- **JEP 389: Foreign Linker API (incubate):** Incubator API này cung cấp tính năng truy cập native code bằng Java thuần, có static type. API sẽ đơn giản hóa đáng kể quá trình binding native library vốn phức tạp và dễ xảy ra lỗi. Java 1.1 đã hỗ trợ gọi native method thông qua Java Native Interface (JNI), nhưng cách này không dễ sử dụng. Java developer có thể binding native library cụ thể cho từng tác vụ. API cũng cung cấp hỗ trợ foreign function mà không cần bất kỳ JNI glue code trung gian nào.
- **JEP 357: Migration từ Mercurial sang Git**: Trước đây, source code OpenJDK được quản lý bằng công cụ version control Mercurial, nay đã chuyển sang Git.
- **JEP 369: Migration sang GitHub**: Tương ứng với thay đổi trong JEP 357 từ Mercurial sang Git, sau khi chuyển version control sang Git, cộng đồng OpenJDK đã chọn lưu trữ Git repository trên GitHub. Tuy nhiên, việc migration chỉ áp dụng cho JDK 11 và các phiên bản cao hơn.
- **JEP 386: Port Alpine Linux**: Alpine Linux là một Linux distribution độc lập, phi thương mại, có kích thước rất nhỏ: một container không cần quá 8 MB dung lượng, bản cài đặt tối thiểu trên disk chỉ cần khoảng 130 MB và rất đơn giản, đồng thời vẫn bảo đảm tính an toàn. Đề xuất này port JDK sang Alpine Linux. Vì Alpine Linux là Linux distribution nhẹ dựa trên musl lib, các Linux distribution khác dùng musl lib trên kiến trúc x64 và AArch64 cũng được hỗ trợ.
- **JEP 388: Port Windows/AArch64**: Trọng tâm của các JEP này không phải bản thân công việc port, mà là tích hợp chúng vào main repository của JDK; JEP 386 port JDK sang Alpine Linux và các distribution khác sử dụng musl làm C library chính trên x64. Ngoài ra, JEP 388 port JDK sang Windows AArch64 (ARM64).

## Tài liệu tham khảo

- [Java Language Changes](https://docs.oracle.com/en/java/javase/16/language/java-language-changes.html)
- [Consolidated JDK 16 Release Notes](https://www.oracle.com/java/technologies/javase/16all-relnotes.html)
- [Java 16 được phát hành chính thức, phân tích từng tính năng mới](https://www.infoq.cn/article/IAkwhx7i9V7G8zLVEd4L)
- [Thực hành | Phân tích cú pháp mới của Java 16](https://xie.infoq.cn/article/8304c894c4e38318d38ceb116) (bài viết rất hay)

<!-- @include: @article-footer.snippet.md -->
