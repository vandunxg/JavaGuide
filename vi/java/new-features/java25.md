---
title: Tổng quan các tính năng mới của Java 25
description: "Tổng quan các tính năng mới quan trọng và thay đổi preview của JDK 25, tập trung vào concurrency, GC và các cải tiến ngôn ngữ/nền tảng."
category: Java
tag:
  - Tính năng mới của Java
head:
  - - meta
    - name: keywords
      content: Java 25,JDK25,LTS,Scoped Values,Compact Object Headers,Generational Shenandoah,module imports,structured concurrency
---

JDK 25 được phát hành ngày 16 tháng 9 năm 2025 và là một phiên bản rất quan trọng, mang tính cột mốc.

JDK 25 được Oracle và hầu hết các nhà phát hành JDK xếp vào nhóm LTS (bản hỗ trợ dài hạn). Các phiên bản LTS hiện tại của Oracle gồm JDK 8, JDK 11, JDK 17, JDK 21 và JDK 25.

JDK 25 gồm tổng cộng 18 tính năng mới. Bài viết này chọn một số tính năng quan trọng để giới thiệu chi tiết:

- [JEP 506: Scoped Values (giá trị theo phạm vi)](https://openjdk.org/jeps/506)
- [JEP 512: Compact Source Files and Instance Main Methods (source file compact và instance main method)](https://openjdk.org/jeps/512)
- [JEP 519: Compact Object Headers (object header compact)](https://openjdk.org/jeps/519)
- [JEP 521: Generational Shenandoah (Shenandoah GC dạng generational)](https://openjdk.org/jeps/521)
- [JEP 507: Primitive Types in Patterns, instanceof, and switch (pattern matching hỗ trợ primitive type, preview lần thứ ba)](https://openjdk.org/jeps/507)
- [JEP 505: Structured Concurrency (structured concurrency, preview lần thứ năm)](https://openjdk.org/jeps/505)
- [JEP 511: Module Import Declarations (khai báo import module)](https://openjdk.org/jeps/511)
- [JEP 513: Flexible Constructor Bodies (constructor body linh hoạt)](https://openjdk.org/jeps/513)
- [JEP 508: Vector API (Vector API, incubation lần thứ mười)](https://openjdk.org/jeps/508)

Hình dưới đây thể hiện số lượng tính năng mới và thời điểm cập nhật của từng phiên bản từ JDK 8 đến JDK 25:

![](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

## JDK 25

### JEP 506: Scoped Values (giá trị theo phạm vi)

Scoped Values có thể chia sẻ dữ liệu immutable trong một thread và giữa các thread, vượt trội hơn `ThreadLocal`, đặc biệt khi sử dụng nhiều virtual thread.

```java
final static ScopedValue<...> V = ScopedValue.newInstance();

// In some method
ScopedValue.where(V, <value>)
           .run(() -> { ... V.get() ... call methods ... });

// In a method called directly or indirectly from the lambda expression
... V.get() ...
```

Scoped Values cung cấp cơ chế truyền dữ liệu immutable một chiều từ caller đến callee. Các thread con được tạo bằng structured concurrency có thể kế thừa binding của thread cha; việc này không cần sao chép binding, nhờ đó giảm chi phí thời gian và không gian khi nhiều thread chia sẻ context.

### JEP 512: Compact Source Files và Instance Main Methods

Tính năng này lần đầu được đề xuất trong [JEP 445](https://openjdk.org/jeps/445 "JEP 445") (JDK 21), sau đó tiếp tục được cải tiến và hoàn thiện trong JDK 22, JDK 23 và JDK 24, cuối cùng chính thức trở thành tính năng trong JDK 25.

Cải tiến này đơn giản hóa đáng kể các bước viết chương trình Java đơn giản, cho phép đặt class và main method trong cùng một file không có `public class` cấp cao nhất, đồng thời cho phép method `main` trở thành instance method không static.

```java
class HelloWorld {
    void main() {
        System.out.println("Hello, World!");
    }
}
```

Đơn giản hơn nữa:

```java
void main() {
    System.out.println("Hello, World!");
}
```

Đây là một bước tiến lớn nhằm hạ thấp rào cản học Java và nâng cao hiệu quả khi viết chương trình nhỏ, script. Người mới bắt đầu không còn phải hiểu khai báo phức tạp `public static void main(String[] args)`. Với việc tạo prototype nhanh và viết script, điều này cũng khiến Java trở thành lựa chọn hấp dẫn hơn.

### JEP 519: Compact Object Headers

Tính năng này được giới thiệu dưới dạng tính năng thử nghiệm trong [JEP 450](https://openjdk.org/jeps/450 "JEP 450") (JDK 24) và trở thành tính năng chính thức trong JDK 25.

Thông qua việc tối ưu cấu trúc bên trong của object header, trên HotSpot JVM với kiến trúc 64-bit, kích thước object header được giảm từ 96-128 bit (12-16 byte) ban đầu xuống còn 64 bit (8 byte). Kết quả là giảm mức sử dụng heap, tăng mật độ triển khai và cải thiện tính cục bộ của dữ liệu.

Compact Object Headers chưa trở thành layout object header mặc định của JVM, mà cần được bật bằng cấu hình tường minh:

- JDK 24 cần bật bằng tổ hợp command-line argument:
  `$ java -XX:+UnlockExperimentalVMOptions -XX:+UseCompactObjectHeaders ...`;
- Từ JDK 25 chỉ cần `-XX:+UseCompactObjectHeaders` là có thể bật.

### JEP 521: Generational Shenandoah GC

Shenandoah GC được giới thiệu dưới dạng tính năng thử nghiệm trong JDK 12 và trở thành tính năng chính thức trong JDK 15. Tính năng này mặc định bị tắt, có thể bật bằng `-XX:+UseShenandoahGC`.

Shenandoah là garbage collector có pause time thấp do Red Hat chủ trì phát triển, mục tiêu là vẫn duy trì pause time ngắn và tương đối độc lập với kích thước heap ngay cả khi heap lớn.

Shenandoah truyền thống thực hiện concurrent marking và compacting trên toàn bộ heap. Dù pause time cực ngắn, hiệu quả khi xử lý object ở young generation lại không bằng generational GC. Sau khi bổ sung generational, Shenandoah có thể thu gom thường xuyên và hiệu quả hơn lượng lớn object có vòng đời ngắn trong young generation, nhờ đó vừa duy trì pause time cực thấp, vừa có throughput cao hơn và CPU overhead thấp hơn.

Shenandoah GC cần được bật bằng lệnh:

- JDK 24 cần bật bằng tổ hợp command-line argument: `-XX:+UseShenandoahGC -XX:+UnlockExperimentalVMOptions -XX:ShenandoahGCMode=generational`
- Từ JDK 25 chỉ cần `-XX:+UseShenandoahGC -XX:ShenandoahGCMode=generational` là có thể bật.

### JEP 507: Pattern matching hỗ trợ primitive type (preview lần thứ ba)

Tính năng này lần đầu được đề xuất trong [JEP 455](https://openjdk.org/jeps/455 "JEP 455") (JDK 23).

Pattern matching có thể xử lý mọi primitive type (`int`, `double`, `boolean`...) trong câu lệnh `switch` và `instanceof`.

```java
static void test(Object obj) {
    if (obj instanceof int i) {
        System.out.println("Đây là kiểu int: " + i);
    }
}
```

Nhờ đó, có thể thực hiện type matching và conversion đối với primitive type an toàn và ngắn gọn hơn như với object type, tiếp tục loại bỏ template code trong Java.

### JEP 505: Structured Concurrency (preview lần thứ năm)

JDK 19 giới thiệu structured concurrency dưới dạng incubator API. Trong JDK 25, API này đang ở giai đoạn preview lần thứ năm, nhằm đơn giản hóa việc lập trình multi-thread, không nhằm thay thế `java.util.concurrent`.

Structured concurrency xem nhiều task chạy trong các thread khác nhau là một work unit duy nhất, qua đó đơn giản hóa việc xử lý lỗi, nâng cao độ tin cậy và cải thiện observability. Nói cách khác, structured concurrency giữ lại khả năng đọc, bảo trì và quan sát của code single-thread.

API cơ bản của structured concurrency là `StructuredTaskScope`. API này hỗ trợ chia task thành nhiều subtask chạy đồng thời, thực thi chúng trên các thread riêng, và các subtask phải hoàn tất trước khi task chính tiếp tục.

Cách sử dụng cơ bản của `StructuredTaskScope` như sau:

```java
    try (var scope = StructuredTaskScope.open()) {
        // Dùng method fork để tạo thread thực thi subtask
        Subtask<Integer> subtask1 = scope.fork(task1);
        Subtask<String> subtask2 = scope.fork(task2);
        // Chờ thread hoàn tất
        scope.join();
        // Xử lý result, có thể xử lý hoặc rethrow exception
        ... process results/exceptions ...
    } // close
```

Structured concurrency đặc biệt phù hợp với virtual thread, một loại thread nhẹ do JDK triển khai. Nhiều virtual thread có thể dùng chung một OS thread, nhờ đó cho phép tạo số lượng virtual thread rất lớn.

### JEP 511: Module Import Declarations

Tính năng này lần đầu được đề xuất trong [JEP 476](https://openjdk.org/jeps/476 "JEP 476") (JDK 23), sau đó được hoàn thiện trong [JEP 494](https://openjdk.org/jeps/494 "JEP 494") (JDK 24) và chính thức trở thành tính năng trong JDK 25.

Module import declarations cho phép import ngắn gọn toàn bộ exported package của một module trong code Java mà không cần khai báo từng import package. Tính năng này đơn giản hóa việc tái sử dụng thư viện module hóa, đặc biệt khi sử dụng nhiều module, tránh phải viết nhiều import package và giúp lập trình viên truy cập thư viện bên thứ ba cùng các class cơ bản của Java thuận tiện hơn.

Tính năng này đặc biệt hữu ích cho người mới bắt đầu và phát triển prototype, vì không yêu cầu lập trình viên module hóa code của mình mà vẫn tương thích với cách import truyền thống, nâng cao hiệu suất phát triển và khả năng đọc code.

```java
// Import toàn bộ module java.base, có thể truy cập trực tiếp các class như List, Map, Stream mà không cần import package tương ứng mỗi lần
import module java.base;

public class Example {
    public static void main(String[] args) {
        String[] fruits = { "apple", "berry", "citrus" };
        Map<String, String> fruitMap = Stream.of(fruits)
            .collect(Collectors.toMap(
                s -> s.toUpperCase().substring(0, 1),
                Function.identity()));

        System.out.println(fruitMap);
    }
}
```

### JEP 513: Flexible Constructor Bodies

Tính năng này lần đầu được đề xuất trong [JEP 447](https://openjdk.org/jeps/447 "JEP 447") (JDK 22), sau đó trải qua các giai đoạn preview trong [JEP 482 ](https://openjdk.org/jeps/482 "JEP 482 ") (JDK 23) và [JEP 492](https://openjdk.org/jeps/492 "JEP 492") (JDK 24), rồi chính thức trở thành tính năng trong JDK 25.

Java yêu cầu trong constructor, lời gọi `super(...)` hoặc `this(...)` phải xuất hiện ở câu lệnh đầu tiên. Điều này có nghĩa là không thể trực tiếp khởi tạo field trong constructor của subclass trước khi gọi constructor của superclass.

Flexible constructor bodies giải quyết vấn đề này bằng cách cho phép viết statement trong constructor body trước khi gọi `super(..)` hoặc `this(..)`. Các statement này có thể khởi tạo field nhưng không được tham chiếu đến instance đang được khởi tạo. Nhờ đó, khi superclass constructor gọi method của subclass, field của subclass không bị sử dụng khi chưa được khởi tạo đúng cách, tăng độ tin cậy của việc tạo class.

Tính năng này giải quyết hạn chế trước đây của Java syntax đối với việc tổ chức code constructor, cho phép lập trình viên biểu đạt hành vi của constructor tự do và tự nhiên hơn. Ví dụ, có thể trực tiếp thực hiện validation, chuẩn bị và chia sẻ parameter trong constructor mà không cần phụ thuộc vào helper method hoặc constructor phụ trợ, từ đó cải thiện khả năng đọc và bảo trì code.

```java
class Person {
    private final String name;
    private int age;

    public Person(String name, int age) {
        if (age < 0) {
            throw new IllegalArgumentException("Age cannot be negative.");
        }
        this.name = name; // Khởi tạo field trước khi gọi superclass constructor
        this.age = age;
        // ... Code khởi tạo khác
    }
}

class Employee extends Person {
    private final int employeeId;

    public Employee(String name, int age, int employeeId) {
        this.employeeId = employeeId; // Khởi tạo field trước khi gọi superclass constructor
        super(name, age); // Gọi superclass constructor
        // ... Code khởi tạo khác
    }
}
```

### JEP 508: Vector API (incubation lần thứ mười)

Vector computation gồm một chuỗi phép toán trên vector. Vector API dùng để biểu đạt vector computation; tại runtime, computation này có thể được biên dịch đáng tin cậy thành các vector instruction tối ưu trên CPU architecture được hỗ trợ, nhờ đó đạt performance cao hơn scalar computation tương đương.

Mục tiêu của Vector API là cung cấp cho người dùng một cách đơn giản, dễ sử dụng và độc lập với platform để biểu đạt nhiều loại vector computation.

Đây là scalar computation đơn giản trên các phần tử của array:

```java
void scalarComputation(float[] a, float[] b, float[] c) {
   for (int i = 0; i < a.length; i++) {
        c[i] = (a[i] * a[i] + b[i] * b[i]) * -1.0f;
   }
}
```

Đây là vector computation tương đương, sử dụng Vector API:

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

Mặc dù vẫn đang trong giai đoạn incubation, lần lặp thứ mười đã đủ cho thấy tầm quan trọng của API này. Nó cho phép Java viết code trong các lĩnh vực nhạy về performance như scientific computing, machine learning và big data processing với performance gần bằng, thậm chí sánh ngang performance của các ngôn ngữ native như C++. Đây là yếu tố then chốt giúp Java duy trì sức cạnh tranh trong lĩnh vực high-performance computing.
