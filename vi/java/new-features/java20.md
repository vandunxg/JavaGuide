---
title: Tổng quan các tính năng mới của Java 20
description: Tổng hợp các thay đổi về ngôn ngữ và concurrency của JDK 20, tiếp nối các cải tiến liên quan đến virtual thread và pattern matching.
category: Java
tag:
  - Java New Features
head:
  - - meta
    - name: keywords
      content: Java 20,JDK20,Record Patterns preview,cải tiến Virtual Threads,cải tiến ngôn ngữ,JEP
---

JDK 20 được phát hành vào ngày 21 tháng 3 năm 2023, là phiên bản không thuộc nhóm hỗ trợ dài hạn.

Phiên bản tiếp theo là JDK 21, phiên bản LTS được phát hành vào tháng 9 năm 2023.

JDK 20 có tổng cộng 7 tính năng mới. Bài viết này sẽ chọn một số tính năng mới quan trọng để giới thiệu chi tiết:

- [JEP 429: Scoped Values (giá trị theo phạm vi)](https://openjdk.org/jeps/429) (lần ươm tạo thứ nhất)
- [JEP 432: Record Patterns (record pattern)](https://openjdk.org/jeps/432) (lần preview thứ hai)
- [JEP 433: Pattern Matching for switch (pattern matching cho switch)](https://openjdk.org/jeps/433) (lần preview thứ tư)
- [JEP 434: Foreign Function & Memory API (API hàm và bộ nhớ bên ngoài)](https://openjdk.org/jeps/434) (lần preview thứ hai)
- [JEP 436: Virtual Threads (virtual thread)](https://openjdk.org/jeps/436) (lần preview thứ hai)
- [JEP 437: Structured Concurrency (structured concurrency)](https://openjdk.org/jeps/437) (lần ươm tạo thứ hai)
- [JEP 438: Vector API (API vector)](https://openjdk.org/jeps/438) (lần ươm tạo thứ năm)

Hình dưới đây cho thấy số lượng tính năng mới và thời điểm cập nhật của từng phiên bản từ JDK 8 đến JDK 25:

![Số lượng tính năng mới và thời điểm cập nhật của từng phiên bản từ JDK 8 đến JDK 25](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

## JEP 429: Scoped Values (giá trị theo phạm vi, lần ươm tạo thứ nhất)

Scoped Values có thể chia sẻ dữ liệu immutable trong cùng một thread và giữa các thread, ưu việt hơn ThreadLocal, đặc biệt khi sử dụng nhiều virtual thread.

```java
final static ScopedValue<...> V = ScopedValue.newInstance();

// In some method
ScopedValue.where(V, <value>)
           .run(() -> { ... V.get() ... call methods ... });

// In a method called directly or indirectly from the lambda expression
... V.get() ...
```

Scoped Values cho phép chia sẻ dữ liệu an toàn và hiệu quả giữa các component trong chương trình lớn mà không cần truyền qua tham số method.

Để tìm hiểu chi tiết về Scoped Values, bạn nên đọc bài [Câu hỏi thường gặp về Scoped Values](https://www.happycoders.eu/java/scoped-values/).

## JEP 432: Record Patterns (record pattern, lần preview thứ hai)

Record Patterns có thể destructure giá trị của `record`, tức là trích xuất dữ liệu từ Record Class thuận tiện hơn. Ngoài ra, chúng còn có thể được lồng nhau và kết hợp với type pattern để tạo ra cách điều hướng và xử lý dữ liệu mạnh mẽ, mang tính khai báo và có thể kết hợp.

Record Patterns không thể được sử dụng độc lập mà phải kết hợp với `instanceof` hoặc pattern matching của `switch`.

Trước tiên, hãy dùng `instanceof` để minh họa đơn giản.

Định nghĩa một record class đơn giản:

```java
record Shape(String type, long unit){}
```

Trước khi có Record Patterns:

```java
Shape circle = new Shape("Circle", 10);
if (circle instanceof Shape shape) {

  System.out.println("Area of " + shape.type() + " is : " + Math.PI * Math.pow(shape.unit(), 2));
}
```

Sau khi có Record Patterns:

```java
Shape circle = new Shape("Circle", 10);
if (circle instanceof Shape(String type, long unit)) {
  System.out.println("Area of " + type + " is : " + Math.PI * Math.pow(unit, 2));
}
```

Tiếp theo, hãy xem cách kết hợp Record Patterns với `switch`.

Định nghĩa một số class:

```java
interface Shape {}
record Circle(double radius) implements Shape { }
record Square(double side) implements Shape { }
record Rectangle(double length, double width) implements Shape { }
```

Trước khi có Record Patterns:

```java
Shape shape = new Circle(10);
switch (shape) {
    case Circle c:
        System.out.println("The shape is Circle with area: " + Math.PI * c.radius() * c.radius());
        break;

    case Square s:
        System.out.println("The shape is Square with area: " + s.side() * s.side());
        break;

    case Rectangle r:
        System.out.println("The shape is Rectangle with area: " + r.length() * r.width());
        break;

    default:
        System.out.println("Unknown Shape");
        break;
}
```

Sau khi có Record Patterns:

```java
Shape shape = new Circle(10);
switch(shape) {

  case Circle(double radius):
    System.out.println("The shape is Circle with area: " + Math.PI * radius * radius);
    break;

  case Square(double side):
    System.out.println("The shape is Square with area: " + side * side);
    break;

  case Rectangle(double length, double width):
    System.out.println("The shape is Rectangle with area: " + length * width);
    break;

  default:
    System.out.println("Unknown Shape");
    break;
}
```

Record Patterns loại bỏ các thao tác ép kiểu không cần thiết, giúp code ngắn gọn và dễ đọc hơn. Bản thân Record Patterns không loại bỏ mọi rủi ro liên quan đến `null` hoặc `NullPointerException`: `null` không khớp với Record Patterns, và tham chiếu của component trong record vẫn có thể là `null`.

Record Patterns được preview lần đầu trong Java 19, do [JEP 405](https://openjdk.org/jeps/405) đề xuất. Trong JDK 20, đây là lần preview thứ hai, do [JEP 432](https://openjdk.org/jeps/432) đề xuất. Các cải tiến lần này gồm:

- Thêm hỗ trợ suy luận type parameter cho generic Record Patterns.
- Thêm hỗ trợ để Record Patterns xuất hiện trong phần đầu của enhanced `for` statement.
- Xóa hỗ trợ cho named Record Patterns.

**Lưu ý**: Không nhầm lẫn Record Patterns với record class được giới thiệu chính thức trong [JDK16](./java16.md).

## JEP 433: Pattern Matching for switch (pattern matching cho switch, lần preview thứ tư)

Tương tự `instanceof`, `switch` cũng được bổ sung khả năng tự động chuyển đổi khi type matching.

Ví dụ code `instanceof`:

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

Ví dụ code `switch`:

```java
// Old code
static String formatter(Object o) {
    String formatted = "unknown";
    if (o instanceof Integer i) {
        formatted = String.format("int %d", i);
    } else if (o instanceof Long l) {
        formatted = String.format("long %d", l);
    } else if (o instanceof Double d) {
        formatted = String.format("double %f", d);
    } else if (o instanceof String s) {
        formatted = String.format("String %s", s);
    }
    return formatted;
}

// New code
static String formatterPatternSwitch(Object o) {
    return switch (o) {
        case Integer i -> String.format("int %d", i);
        case Long l    -> String.format("long %d", l);
        case Double d  -> String.format("double %f", d);
        case String s  -> String.format("String %s", s);
        default        -> o.toString();
    };
}
```

Pattern matching của `switch` đã được preview lần lượt trong Java 17, Java 18 và Java 19; Java 20 là lần preview thứ tư. Mỗi lần preview về cơ bản đều có một số cải tiến nhỏ, nên ở đây không trình bày chi tiết.

## JEP 434: Foreign Function & Memory API (API hàm và bộ nhớ bên ngoài, lần preview thứ hai)

Java program có thể dùng API này để tương tác với code và dữ liệu bên ngoài Java runtime. Bằng cách gọi hiệu quả các foreign function (tức code bên ngoài JVM) và truy cập an toàn vào foreign memory (tức vùng nhớ không do JVM quản lý), API này cho phép Java program gọi native library và xử lý native data mà không nguy hiểm, mong manh như JNI.

Foreign Function & Memory API được ươm tạo lần đầu trong Java 17, do [JEP 412](https://openjdk.java.net/jeps/412) đề xuất. Trong Java 18, API này được ươm tạo lần thứ hai, do [JEP 419](https://openjdk.org/jeps/419) đề xuất. Trong Java 19, đây là lần preview đầu tiên, do [JEP 424](https://openjdk.org/jeps/424) đề xuất.

Trong JDK 20, đây là lần preview thứ hai, do [JEP 434](https://openjdk.org/jeps/434) đề xuất. Các cải tiến lần này gồm:

- Hợp nhất các abstraction `MemorySegment` và `MemoryAddress`.
- Cải thiện hệ thống phân cấp của `MemoryLayout`.
- Tách `MemorySession` thành `Arena` và `SegmentScope` để hỗ trợ chia sẻ segment xuyên qua ranh giới quản lý.

Trong [Tổng quan các tính năng mới của Java 19](./java19.md), tôi đã giới thiệu chi tiết về Foreign Function & Memory API, nên ở đây không giới thiệu thêm.

## JEP 436: Virtual Threads (virtual thread, lần preview thứ hai)

Virtual thread là thread nhẹ do JDK chứ không phải OS triển khai và được JDK scheduling. Nhiều virtual thread chia sẻ cùng một operating system thread, nên số lượng virtual thread có thể lớn hơn rất nhiều so với số lượng operating system thread.

Trước khi virtual thread được giới thiệu, package `java.lang.Thread` đã hỗ trợ platform thread, tức loại thread mà chúng ta vẫn sử dụng trước khi có virtual thread. JVM scheduler quản lý virtual thread thông qua platform thread (carrier thread). Một platform thread có thể thực thi các virtual thread khác nhau ở những thời điểm khác nhau (nhiều virtual thread được mount trên một platform thread). Khi virtual thread bị block hoặc phải chờ, platform thread có thể chuyển sang thực thi một virtual thread khác.

Mối quan hệ giữa virtual thread, platform thread và system kernel thread được minh họa dưới đây (nguồn hình: [How to Use Java 19 Virtual Threads](https://medium.com/javarevisited/how-to-use-java-19-virtual-threads-c16a32bad5f7)):

![Mối quan hệ giữa virtual thread, platform thread và system kernel thread](https://oss.javaguide.cn/github/javaguide/java/new-features/virtual-threads-platform-threads-kernel-threads-relationship.png)

Nói thêm một chút về mối quan hệ tương ứng giữa platform thread và system kernel thread: Trong các operating system phổ biến như Windows và Linux, Java thread sử dụng thread model one-to-one, tức một platform thread tương ứng với một system kernel thread. Solaris là một ngoại lệ: HotSpot VM trên Solaris hỗ trợ mô hình many-to-many và one-to-one. Bạn có thể tham khảo câu trả lời của R: [Thread model trong JVM là user-level phải không?](https://www.zhihu.com/question/23096638/answer/29617153).

So với platform thread, virtual thread rẻ và nhẹ hơn, có thể hủy ngay sau khi sử dụng, vì vậy không cần reuse hoặc pool chúng. Mỗi task có thể chạy trên một virtual thread riêng. Việc suspend và resume virtual thread thường không cần tạo hoặc chuyển đổi một operating system thread cho từng task, từ đó giảm chi phí tài nguyên thread và scheduling do nhiều task blocking gây ra.

Virtual thread đã được chứng minh là rất hữu ích trong các ngôn ngữ lập trình multi-thread khác, chẳng hạn như Goroutine trong Go và process trong Erlang.

Có một thảo luận về virtual thread của Java 19 trên Zhihu. Nếu quan tâm, bạn có thể xem tại: <https://www.zhihu.com/question/536743167>.

Bạn có thể xem phần giải thích chi tiết và nguyên lý về virtual thread của Java trong các bài viết sau:

- [Nhập môn virtual thread tối giản](https://javaguide.cn/java/concurrent/virtual-thread.html)
- [Java19 chính thức GA! Tìm hiểu cách virtual thread cải thiện đáng kể throughput của hệ thống](https://mp.weixin.qq.com/s/yyApBXxpXxVwttr01Hld6Q)
- [Virtual thread - phân tích source code của VirtualThread](https://www.cnblogs.com/throwable/p/16758997.html)

Virtual thread được preview lần đầu trong Java 19, do [JEP 425](https://openjdk.org/jeps/425) đề xuất. Trong JDK 20, đây là lần preview thứ hai, với một số thay đổi nhỏ nên ở đây không trình bày chi tiết.

Cuối cùng, hãy xem bốn cách tạo virtual thread:

```java
// 1. Tạo thông qua Thread.ofVirtual()
Runnable fn = () -> {
  // your code here
};

Thread thread = Thread.ofVirtual()
                      .start(fn);

// 2. Tạo thông qua Thread.startVirtualThread()
Thread thread = Thread.startVirtualThread(() -> {
  // your code here
});

// 3. Tạo thông qua Executors.newVirtualThreadPerTaskExecutor()
var executorService = Executors.newVirtualThreadPerTaskExecutor();

executorService.submit(() -> {
  // your code here
});

class CustomThread implements Runnable {
  @Override
  public void run() {
    System.out.println("CustomThread run");
  }
}

// 4. Tạo thông qua ThreadFactory
CustomThread customThread = new CustomThread();
// Lấy thread factory class
ThreadFactory factory = Thread.ofVirtual().factory();
// Tạo virtual thread
Thread thread = factory.newThread(customThread);
// Khởi động thread
thread.start();
```

Qua 4 cách tạo virtual thread nêu trên, có thể thấy để hạ thấp rào cản sử dụng virtual thread, JDK cố gắng tái sử dụng `Thread` class hiện có, giúp chuyển đổi sang sử dụng virtual thread một cách thuận lợi.

## JEP 437: Structured Concurrency (structured concurrency, lần ươm tạo thứ hai)

Java 19 giới thiệu structured concurrency, một phương pháp lập trình multi-thread nhằm đơn giản hóa việc lập trình multi-thread thông qua structured concurrency API, không nhằm thay thế `java.util.concurrent` và hiện vẫn đang ở giai đoạn ươm tạo.

Structured concurrency coi nhiều task chạy trong các thread khác nhau là một work unit duy nhất, từ đó đơn giản hóa việc xử lý lỗi, nâng cao độ tin cậy và tăng khả năng quan sát. Nói cách khác, structured concurrency giữ lại khả năng đọc, bảo trì và quan sát của code single-thread.

API cơ bản của structured concurrency là [`StructuredTaskScope`](https://download.java.net/java/early_access/loom/docs/api/jdk.incubator.concurrent/jdk/incubator/concurrent/StructuredTaskScope.html). `StructuredTaskScope` hỗ trợ chia task thành nhiều concurrent subtask, thực thi chúng trong các thread riêng và yêu cầu các subtask hoàn tất trước khi task chính tiếp tục.

Cách sử dụng cơ bản của `StructuredTaskScope` như sau:

```java
    try (var scope = new StructuredTaskScope<Object>()) {
        // Dùng method fork để tạo thread thực thi subtask
        Future<Integer> future1 = scope.fork(task1);
        Future<String> future2 = scope.fork(task2);
        // Chờ thread hoàn tất
        scope.join();
        // Xử lý kết quả có thể bao gồm xử lý hoặc ném lại exception
        ... process results/exceptions ...
    } // close
```

Structured concurrency đặc biệt phù hợp với virtual thread, một loại thread nhẹ do JDK triển khai. Nhiều virtual thread chia sẻ cùng một operating system thread, cho phép tạo ra số lượng virtual thread rất lớn.

Thay đổi duy nhất của structured concurrency trong JDK 20 là cập nhật để các thread được tạo trong task scope, `StructuredTaskScope`, kế thừa Scoped Values. Điều này đơn giản hóa việc chia sẻ dữ liệu immutable giữa các thread. Xem chi tiết tại [JEP 429](https://openjdk.org/jeps/429).

## JEP 438: Vector API (API vector, lần ươm tạo thứ năm)

Vector computation gồm một chuỗi thao tác trên vector. Vector API dùng để biểu đạt vector computation. Trong runtime, computation này có thể được compile đáng tin cậy thành các vector instruction tối ưu trên CPU architecture được hỗ trợ, từ đó đạt performance tốt hơn scalar computation tương đương.

Mục tiêu của Vector API là cung cấp cho người dùng cách biểu đạt đơn giản, dễ sử dụng và không phụ thuộc platform cho nhiều loại vector computation.

Vector API ban đầu do [JEP 338](https://openjdk.java.net/jeps/338) đề xuất và được tích hợp vào Java 16 dưới dạng [incubator API](http://openjdk.java.net/jeps/11). Lần ươm tạo thứ hai do [JEP 414](https://openjdk.java.net/jeps/414) đề xuất và được tích hợp vào Java 17. Lần ươm tạo thứ ba do [JEP 417](https://openjdk.java.net/jeps/417) đề xuất và được tích hợp vào Java 18. Lần thứ tư do [JEP 426](https://openjdk.java.net/jeps/426) đề xuất và được tích hợp vào Java 19.

Lần ươm tạo này trong Java 20 về cơ bản không thay đổi Vector API, chỉ sửa một số lỗi và cải thiện performance. Xem chi tiết tại [JEP 438](https://openjdk.org/jeps/438).

<!-- @include: @article-footer.snippet.md -->
