---
title: "Tổng quan tính năng mới của Java 26"
description: "Tổng quan các tính năng mới quan trọng và thay đổi đang ở preview của JDK 26, tập trung vào HTTP/3, tối ưu performance GC, AOT cache và cải tiến ngôn ngữ/nền tảng."
category: Java
tag:
  - Java New Features
head:
  - - meta
    - name: keywords
      content: Java 26,JDK26,HTTP/3,G1 GC,AOT cache,lazy constants,structured concurrency,Vector API,pattern matching
---

JDK 26 được phát hành vào ngày 17 tháng 3 năm 2026, đây là một phiên bản không phải LTS (không hỗ trợ dài hạn). Phiên bản LTS trước đó là **JDK 25**, phiên bản LTS tiếp theo dự kiến là **JDK 29**.

JDK 26 có tổng cộng 10 tính năng mới. Bài viết này sẽ chọn một số tính năng mới quan trọng để giới thiệu chi tiết:

- [JEP 517: HTTP/3 for the HTTP Client API (Thêm hỗ trợ HTTP/3 cho HTTP Client API)](https://openjdk.org/jeps/517)
- [JEP 522: G1 GC: Improve Throughput by Reducing Synchronization (Tăng throughput của G1 GC bằng cách giảm synchronization)](https://openjdk.org/jeps/522)
- [JEP 516: Ahead-of-Time Object Caching with Any GC (AOT object cache hỗ trợ mọi GC)](https://openjdk.org/jeps/516)
- [JEP 500: Prepare to Make Final Mean Final (Chuẩn bị làm cho final thực sự immutable)](https://openjdk.org/jeps/500)
- [JEP 526: Lazy Constants (Lazy Constants, preview lần hai)](https://openjdk.org/jeps/526)
- [JEP 525: Structured Concurrency (Structured Concurrency, preview lần sáu)](https://openjdk.org/jeps/525)
- [JEP 530: Primitive Types in Patterns, instanceof, and switch (Pattern matching hỗ trợ primitive type, preview lần bốn)](https://openjdk.org/jeps/530)
- [JEP 524: PEM Encodings of Cryptographic Objects (PEM encoding cho cryptographic object, preview lần hai)](https://openjdk.org/jeps/524)
- [JEP 529: Vector API (Vector API, incubator lần thứ mười một)](https://openjdk.org/jeps/529)
- [JEP 504: Remove the Applet API (Loại bỏ Applet API)](https://openjdk.org/jeps/504)

Hình dưới đây cho biết số lượng tính năng mới và thời điểm cập nhật của từng phiên bản từ JDK 8 đến JDK 25:

![](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

## JEP 517: Thêm hỗ trợ HTTP/3 cho HTTP Client API

JDK 26 chính thức thêm hỗ trợ **HTTP/3** cho API `java.net.http.HttpClient`. Đây là một cập nhật quan trọng đã được mong đợi từ lâu.

**Ưu điểm của HTTP/3**:

- **Dựa trên giao thức QUIC**: HTTP/2 được triển khai dựa trên giao thức TCP, còn HTTP/3 bổ sung giao thức QUIC (Quick UDP Internet Connections) để truyền dữ liệu đáng tin cậy, cung cấp mức bảo mật tương đương TLS/SSL và có latency kết nối, truyền tải thấp hơn. Có thể xem QUIC là phiên bản nâng cấp của UDP, bổ sung nhiều tính năng như encryption, retransmission, v.v.
- **Loại bỏ head-of-line blocking**: HTTP/2 multiplex nhiều request trên một kết nối TCP. Khi xảy ra mất gói, tất cả HTTP request sẽ bị block. Nhờ đặc tính của QUIC, HTTP/3 giải quyết được phần nào vấn đề head-of-line blocking (Head-of-Line blocking, viết tắt: HOL blocking): một kết nối có thể thiết lập nhiều data stream độc lập; khi một data stream mất gói, các data stream khác không bị ảnh hưởng.
- **Thiết lập kết nối nhanh hơn**: HTTP/2 cần trải qua quy trình TCP three-way handshake kinh điển (do thiết lập kết nối HTTPS an toàn còn cần TLS handshake, tổng cộng cần khoảng 3 RTT). Nhờ đặc tính của QUIC (TLS 1.3 hỗ trợ handshake 1 RTT và 0 RTT), việc thiết lập kết nối chỉ cần 0-RTT hoặc 1-RTT. Điều này có nghĩa là trong trường hợp tốt nhất, QUIC không cần thêm round trip time nào để thiết lập kết nối mới.
- **Trải nghiệm mobile tốt hơn**: HTTP/3 hỗ trợ connection migration dựa trên kết nối QUIC. QUIC dùng connection ID có độ dài thay đổi do endpoint chọn để định danh và route kết nối, connection ID cũng có thể thay đổi trong khi kết nối; sau khi xử lý path validation, thay đổi địa chỉ mạng (chẳng hạn chuyển từ Wi-Fi sang mobile data) không cần thiết lập lại toàn bộ kết nối như khi TCP four-tuple thay đổi.

Bạn có thể đọc phần giới thiệu chi tiết trong bài viết: [Tổng hợp câu hỏi phỏng vấn Computer Network thường gặp (Phần 1)](https://javaguide.cn/cs-basics/network/other-network-questions.html) (mô hình phân tầng network, tổng hợp các network protocol thường gặp, HTTP, WebSocket, DNS, v.v.).

**Cách sử dụng**:

Trong JDK 26, protocol mặc định của `HttpClient` vẫn là HTTP/2. Để dùng HTTP/3, cần chỉ định rõ `HTTP_3` trên `HttpClient` hoặc một `HttpRequest`:

```java
HttpClient client = HttpClient.newBuilder()
    .version(HttpClient.Version.HTTP_3)
    .build();

HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://example.com"))
    .build();

// Ưu tiên thử HTTP/3; nếu server không hỗ trợ thì mặc định fallback về HTTP/2 hoặc HTTP/1.1
HttpResponse<String> response = client.send(request,
    HttpResponse.BodyHandlers.ofString());

System.out.println(response.body());
```

Nếu cần chỉ định rõ sử dụng HTTP/3, có thể thiết lập thông qua method `version()`:

```java
// Mặc định ưu tiên sử dụng HTTP/3 cho mọi request
HttpClient client = HttpClient.newBuilder()
    .version(HttpClient.Version.HTTP_3)  // Chỉ định rõ HTTP/3
    .build();

// Thiết lập protocol version ưu tiên cho một HttpRequest
HttpRequest request = HttpRequest.newBuilder(URI.create("https://javaguide.cn/"))
                         .version(HttpClient.Version.HTTP_3)
                         .GET().build();
```

## JEP 522: Tăng throughput của G1 GC bằng cách giảm synchronization

**Từ JDK 9, G1 trở thành garbage collector mặc định.** Nó tìm cách cân bằng giữa latency và throughput. Tuy nhiên, sự cân bằng này đôi khi ảnh hưởng đến performance của application. So với Parallel GC hướng tới throughput, G1 làm việc concurrent với application nhiều hơn để giảm thời gian GC pause. Điều này có nghĩa là application thread phải chia sẻ CPU và phối hợp với GC thread, việc synchronization này làm giảm throughput và tăng latency.

JEP 522 giới thiệu cơ chế **hai card table (Card Table)**:

1. **Card table thứ nhất**: write barrier của application thread không cần synchronization khi cập nhật card table này, giúp code của write barrier đơn giản và nhanh hơn.
2. **Card table thứ hai**: optimizer thread xử lý song song card table ban đầu rỗng này trong background.

Khi G1 phát hiện việc scan card table thứ nhất có thể vượt quá mục tiêu pause time, nó sẽ atomically swap hai card table. Application thread tiếp tục cập nhật card table rỗng (card table thứ hai ban đầu), còn optimizer thread xử lý card table đầy (card table thứ nhất ban đầu), không cần synchronization thêm.

**Cải thiện performance**:

- Trong application **thường xuyên sửa object reference field**, throughput tăng **5-15%**
- Ngay cả application không thường xuyên sửa reference field cũng có thể tăng throughput tối đa **5%** nhờ write barrier được đơn giản hóa (trên x64 giảm từ khoảng 50 instruction xuống chỉ còn 12)
- GC pause time cũng **giảm nhẹ**

**Overhead bộ nhớ**:

Card table thứ hai có cùng kích thước với card table thứ nhất. Mỗi card table cần 0,2% dung lượng Java heap, tức mỗi 1GB heap sẽ dùng thêm khoảng 2MB native memory.

## JEP 516: AOT object cache hỗ trợ mọi GC

Đây là một cột mốc quan trọng của **Project Leyden**, cho phép AOT object cache hoạt động với **mọi garbage collector**.

AOT class data sharing (JEP 483) được giới thiệu trước đó trong JDK 24 chỉ hỗ trợ G1 garbage collector, không thể kết hợp với các GC khác như ZGC. Nguyên nhân là object reference được lưu trong AOT cache sử dụng physical memory address, trong khi memory layout và chiến lược di chuyển object của các GC khác nhau.

JEP 516 thay đổi cách lưu object reference từ **physical memory address** sang **logical index**:

- Lưu cache ở streaming format không phụ thuộc GC
- Bất kỳ GC nào cũng có thể load và parse cache tại runtime
- JVM chuyển logical index thành memory address thực tế khi load

**Lợi ích về performance**:

- **Tối ưu startup time**: giảm đáng kể cold start time của Java application
- **Hỗ trợ ZGC**: ZGC có latency thấp giờ đây cũng có thể hưởng lợi từ việc tăng tốc startup nhờ AOT cache
- **Phù hợp cloud-native**: đặc biệt có giá trị trong các trường hợp nhạy cảm với startup time như microservice và serverless function

## JEP 500: Chuẩn bị làm cho final thực sự immutable

Tính năng này mở đường cho nguyên tắc ưu tiên integrity của Java, chuẩn bị làm cho field `final` thực sự immutable.

Từ JDK 5, để hỗ trợ các trường hợp như serialization, reflection API cho phép sửa một số field `final` thông qua **deep reflection**:

```java
import java.lang.reflect.Field;

class Example {
    private final String name;

    Example(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}

// Sửa field final thông qua reflection
Example example = new Example("Original");
Field field = Example.class.getDeclaredField("name");
field.setAccessible(true);
field.set(example, "Modified");  // Mặc định JDK 26 sẽ thành công và phát cảnh báo sửa field final trái phép
System.out.println(example.getName());  // In ra "Modified"
```

Mặc dù khả năng này được một số framework sử dụng (chẳng hạn serialization library, dependency injection framework và test tool), nó phá vỡ bảo đảm về tính immutable của `final` và cản trở compiler optimization.

Trong JDK 26, khi sửa field `final` thông qua deep reflection, JVM sẽ **phát cảnh báo**. Đây là bước chuẩn bị cho việc mặc định cấm thao tác này trong các phiên bản tương lai.

Với các trường hợp thực sự cần sửa field `final`, JDK 26 cung cấp cơ chế lựa chọn rõ ràng, cho phép developer tiếp tục sử dụng khả năng này trong giai đoạn chuyển tiếp, đồng thời chuẩn bị cho strict mode trong tương lai.

## JEP 526: Lazy Constants (preview lần hai)

Tính năng này được preview lần đầu trong JDK 25 với [JEP 502: Stable Values (preview)](https://openjdk.org/jeps/502); JEP 526 của JDK 26 đổi tên API thành Lazy Constants và tiến hành preview lần thứ hai.

Field `static final` truyền thống thường được khởi tạo ngay khi class initialization, điều này sẽ:

- Tăng startup time.
- Lãng phí memory nếu constant đó chưa từng được sử dụng.
- Cần các pattern lazy initialization phức tạp (chẳng hạn double-checked locking, Holder class pattern).

JEP 526 giới thiệu `LazyConstant<T>`, một object chứa immutable data. JVM xem object này như một constant thực sự để đạt performance tương đương field khai báo là `final`.

```java
// Cách truyền thống: khởi tạo ngay khi class initialization
static final ExpensiveObject TRADITIONAL = new ExpensiveObject();

// Cách mới: chỉ khởi tạo khi truy cập lần đầu
static final LazyConstant<ExpensiveObject> LAZY =
    LazyConstant.of(() -> new ExpensiveObject());

// Khi sử dụng
ExpensiveObject obj = LAZY.get();  // Lúc này mới khởi tạo
```

**Ưu điểm**:

- **Khởi tạo theo nhu cầu**: chỉ khởi tạo khi truy cập lần đầu, cải thiện startup performance.
- **Thread-safe**: có sẵn guarantee về thread safety, không cần tự synchronization.
- **JVM optimization**: JVM có thể tối ưu lazy constant như field `final`.
- **Đơn giản hóa code**: loại bỏ các lazy initialization pattern phức tạp như double-checked locking.

## JEP 525: Structured Concurrency (preview lần sáu)

JDK 19 giới thiệu Structured Concurrency dưới dạng incubator API. Trong JDK 26, API này ở preview lần thứ sáu, nhằm đơn giản hóa lập trình multi-thread, không nhằm thay thế `java.util.concurrent`.

Structured Concurrency xem nhiều task chạy trong các thread khác nhau như một work unit, từ đó đơn giản hóa error handling, tăng reliability và cải thiện observability. Nói cách khác, Structured Concurrency giữ lại readability, maintainability và observability của code single-thread.

API cơ bản của Structured Concurrency là `StructuredTaskScope`. API này hỗ trợ chia task thành nhiều concurrent subtask, thực thi chúng trong thread riêng. Các subtask phải hoàn thành trước khi main/parent task tiếp tục và sẽ bị cancel nếu main/parent task fail.

Cách sử dụng cơ bản của `StructuredTaskScope`:

```java
    try (var scope = StructuredTaskScope.open()) {
        // Dùng method fork để tạo thread thực thi subtask
        Subtask<Integer> subtask1 = scope.fork(task1);
        Subtask<String> subtask2 = scope.fork(task2);
        // Chờ thread hoàn tất
        scope.join();
        // Xử lý kết quả có thể bao gồm xử lý hoặc rethrow exception
        ... process results/exceptions ...
    } // close
```

Structured Concurrency đặc biệt phù hợp với virtual thread, một loại thread nhẹ do JDK triển khai. Nhiều virtual thread chia sẻ cùng một operating system thread, cho phép tạo số lượng virtual thread rất lớn.

**Thay đổi mới trong Java 26**:

- **Cải tiến Joiner**: interface `Joiner` thêm method `onTimeout()`, cho phép trả về kết quả cụ thể khi timeout xảy ra.
- **Tối ưu return type**: `allSuccessfulOrThrow()` giờ trả về trực tiếp danh sách kết quả (`List`) thay vì stream của subtask như trước.
- **Đơn giản hóa API**: đổi tên `anySuccessfulResultOrThrow()` thành `anySuccessfulOrThrow()`.

## JEP 530: Pattern matching hỗ trợ primitive type (preview lần bốn)

Tính năng này lần đầu được đề xuất bởi [JEP 455](https://openjdk.org/jeps/455 "JEP 455") (JDK 23).

Pattern matching có thể xử lý mọi primitive data type (`int`, `double`, `boolean`, v.v.) trong câu lệnh `switch` và `instanceof`:

```java
static void test(Object obj) {
    if (obj instanceof int i) {
        System.out.println("Đây là kiểu int: " + i);
    }
}
```

JDK 26 tiếp tục cải tiến tính năng này:

- Loại bỏ nhiều hạn chế liên quan đến primitive type, giúp pattern matching, `instanceof` và `switch` thống nhất, giàu tính biểu đạt hơn.
- Cải tiến định nghĩa về unconditional exactness.
- Áp dụng kiểm tra dominance nghiêm ngặt hơn trong cấu trúc `switch`, giúp compiler nhận biết và giảm nhiều lỗi code hơn.

Nhờ đó, có thể thực hiện type matching và conversion đối với primitive type an toàn, ngắn gọn hơn như với object type, tiếp tục loại bỏ boilerplate code trong Java.

## JEP 524: PEM encoding cho cryptographic object (preview lần hai)

Tính năng này lần đầu được đề xuất bởi [JEP 518](https://openjdk.org/jeps/518) (JDK 25).

PEM (Privacy-Enhanced Mail) là text format được sử dụng rộng rãi để lưu trữ và truyền cryptographic object như certificate, private key và public key. JEP 524 cung cấp API mới để encode cryptographic object thành PEM format và decode từ PEM format trở lại cryptographic object.

```java
// Encode key thành PEM format
KeyPairGenerator kpg = KeyPairGenerator.getInstance("RSA");
kpg.initialize(2048);
KeyPair keyPair = kpg.generateKeyPair();

// Encode thành PEM
String pemEncoded = PEMEncoder.of().encodeToString(keyPair.getPrivate());

// Decode từ PEM
PrivateKey decodedKey = PEMDecoder.of().decode(pemEncoded, PrivateKey.class);
```

API này giảm rủi ro lỗi, đơn giản hóa yêu cầu compliance, đồng thời cải thiện portability và interoperability của Java application an toàn bằng cách đơn giản hóa việc thiết lập và tích hợp encryption cho nhu cầu enterprise, cloud và regulatory.

## JEP 529: Vector API (incubator lần thứ mười một)

Vector computation gồm một chuỗi operation trên vector. Vector API dùng để biểu đạt vector computation. Tại runtime, computation này có thể được compile đáng tin cậy thành vector instruction tối ưu trên CPU architecture được hỗ trợ, từ đó đạt performance tốt hơn scalar computation tương đương.

Mục tiêu của Vector API là cung cấp cho user cách biểu đạt ngắn gọn, dễ sử dụng, platform-independent cho nhiều loại vector computation.

Đây là scalar computation đơn giản trên các phần tử của array:

```java
void scalarComputation(float[] a, float[] b, float[] c) {
   for (int i = 0; i < a.length; i++) {
        c[i] = (a[i] * a[i] + b[i] * b[i]) * -1.0f;
   }
}
```

Đây là vector computation tương đương sử dụng Vector API:

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

Dù vẫn đang ở giai đoạn incubator, iteration lần thứ mười một đã đủ chứng minh tầm quan trọng của Vector API. Nó giúp Java có thể viết code với performance gần bằng hoặc thậm chí tương đương native language như C++ trong các lĩnh vực nhạy cảm với performance như scientific computing, machine learning, AI inference và big data processing.

## JEP 504: Loại bỏ Applet API

Applet API được đánh dấu deprecated trong JDK 9 và được đánh dấu sắp bị loại bỏ trong JDK 17. Trong JDK 26, Applet API cuối cùng đã được **loại bỏ hoàn toàn**. Đây là một thay đổi đáng mừng!

Điều này có nghĩa là:

- Class `java.applet.Applet` và các class liên quan đã bị xóa.
- Giảm dung lượng cài đặt và source code của JDK.
- Cải thiện performance, stability và security của application.

Công nghệ Applet đã lỗi thời từ lâu, modern Web development đã hoàn toàn chuyển sang các technology stack khác. Loại bỏ legacy API này là bước cần thiết để hiện đại hóa Java platform.

## Tổng kết

Dù JDK 26 là một phiên bản không phải LTS, nó vẫn có một số tính năng quan trọng đáng chú ý:

| Category        | Tính năng                                                                                  |
| --------------- | ------------------------------------------------------------------------------------------ |
| **Network**     | Hỗ trợ HTTP/3                                                                              |
| **Performance** | Tối ưu throughput của G1 GC, AOT cache hỗ trợ mọi GC                                       |
| **Language**    | Pattern matching hỗ trợ primitive type (preview lần bốn), Lazy Constants (preview lần hai) |
| **Concurrency** | Structured Concurrency (preview lần sáu), Vector API (incubator lần thứ mười một)          |
| **Security**    | Làm cho final thực sự immutable, hỗ trợ PEM encoding                                       |
| **Cleanup**     | Loại bỏ Applet API                                                                         |

Oracle sẽ cung cấp các bản cập nhật đến tháng 9 năm 2026, sau đó phiên bản này sẽ được thay thế bởi Oracle JDK 27.
