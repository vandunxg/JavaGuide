---
title: "Tính năng mới của Java 17 (JDK 17): sealed class, pattern matching cho switch và Random API"
description: "Giải thích chi tiết tính năng mới trong Java 17 (JDK 17), bao gồm sealed class, pattern matching cho switch, Random API mới, Foreign Function & Memory API, chu kỳ hỗ trợ LTS và các thay đổi cần lưu ý khi nâng cấp."
category: Java
tag:
  - Java New Features
head:
  - - meta
    - name: keywords
      content: Java 17,JDK 17,tính năng mới JDK 17,tính năng mới Java 17,LTS,sealed class,pattern matching cho switch,Random API,Foreign Function & Memory API,JEP
---

Java 17 (JDK 17) được phát hành chính thức vào ngày 14 tháng 9 năm 2021, là phiên bản hỗ trợ dài hạn (LTS) do Oracle xác định.

Theo lộ trình hỗ trợ Java SE được Oracle cập nhật vào tháng 4 năm 2026, Premier Support của Oracle JDK 17 kéo dài đến tháng 9 năm 2026, còn Extended Support kéo dài đến tháng 9 năm 2029. Chu kỳ cập nhật miễn phí và hỗ trợ thương mại của các bản phân phối JDK khác nhau không giống nhau, vì vậy môi trường production vẫn cần căn cứ vào bản phân phối đang sử dụng thực tế.

![](https://oss.javaguide.cn/github/javaguide/java/new-features/4c1611fad59449edbbd6e233690e9fa7.png)

Khi nâng cấp từ JDK 11 lên JDK 17, bạn nên đặc biệt chú ý đến sealed class, pattern matching cho `switch`, PRNG API mới và các thay đổi về tính tương thích do việc đóng gói mạnh các API nội bộ của JDK. Phiên bản Java tối thiểu của Spring 6.x và Spring Boot 3.x cũng là Java 17.

JDK 17 có tổng cộng 14 tính năng mới. Bài viết này sẽ chọn một số tính năng mới quan trọng để giới thiệu chi tiết:

- [JEP 356: Enhanced Pseudo-Random Number Generators (Bộ sinh số giả ngẫu nhiên nâng cao)](https://openjdk.java.net/jeps/356)
- [JEP 398: Deprecate the Applet API for Removal (Đánh dấu Applet API là deprecated để loại bỏ)](https://openjdk.java.net/jeps/398)
- [JEP 406: Pattern Matching for switch (Pattern matching cho switch, preview)](https://openjdk.java.net/jeps/406)
- [JEP 407: Remove RMI Activation (Loại bỏ cơ chế RMI Activation)](https://openjdk.java.net/jeps/407)
- [JEP 409: Sealed Classes (Sealed class, chính thức)](https://openjdk.java.net/jeps/409)
- [JEP 410: Remove the Experimental AOT and JIT Compiler (Loại bỏ các AOT và JIT compiler thử nghiệm)](https://openjdk.java.net/jeps/410)
- [JEP 411: Deprecate the Security Manager for Removal (Đánh dấu Security Manager là deprecated để loại bỏ)](https://openjdk.java.net/jeps/411)
- [JEP 412: Foreign Function & Memory API (Incubator) (Foreign Function & Memory API, incubator lần thứ nhất)](https://openjdk.java.net/jeps/412)
- [JEP 414: Vector API (Second Incubator) (Vector API, incubator lần thứ hai)](https://openjdk.java.net/jeps/414)

Hình dưới đây thể hiện số lượng tính năng mới và thời gian cập nhật của từng phiên bản từ JDK 8 đến JDK 16:

![](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

Đọc thêm: [Tài liệu OpenJDK Java 17](https://openjdk.java.net/projects/jdk/17/).

## JEP 356: Enhanced Pseudo-Random Number Generators (Bộ sinh số giả ngẫu nhiên nâng cao)

Trước JDK 17, chúng ta có thể dùng `Random`, `ThreadLocalRandom` và `SplittableRandom` để tạo số ngẫu nhiên. Tuy nhiên, cả 3 class này đều có những hạn chế riêng và thiếu hỗ trợ cho các thuật toán giả ngẫu nhiên phổ biến.

Java 17 bổ sung các interface và implementation mới cho bộ sinh số giả ngẫu nhiên (pseudorandom number generator, PRNG, còn gọi là bộ sinh bit ngẫu nhiên xác định), giúp developer dễ dàng chuyển đổi giữa các thuật toán PRNG khác nhau trong ứng dụng.

> [PRNG](https://ctf-wiki.org/crypto/streamcipher/prng/intro/) được dùng để tạo ra dãy số gần với dãy số hoàn toàn ngẫu nhiên. Nói chung, PRNG dựa vào một giá trị ban đầu, còn gọi là seed, để tạo ra dãy số giả ngẫu nhiên tương ứng. Khi seed đã được xác định, các số ngẫu nhiên do PRNG tạo ra hoàn toàn xác định, vì vậy dãy số mà nó tạo ra không phải là ngẫu nhiên thực sự.

Ví dụ sử dụng:

```java
RandomGeneratorFactory<RandomGenerator> l128X256MixRandom = RandomGeneratorFactory.of("L128X256MixRandom");
// Dùng timestamp làm seed cho bộ sinh số ngẫu nhiên
RandomGenerator randomGenerator = l128X256MixRandom.create(System.currentTimeMillis());
// Tạo số ngẫu nhiên
randomGenerator.nextInt(10);
```

## JEP 398: Deprecate the Applet API for Removal (Đánh dấu Applet API là deprecated để loại bỏ)

Applet API được dùng để viết các Java applet chạy trên trình duyệt Web. Nó đã lỗi thời từ nhiều năm trước nên không còn lý do để sử dụng.

Applet API được đánh dấu deprecated từ Java 9 ([JEP 289](https://openjdk.java.net/jeps/289)), nhưng khi đó chưa nhằm mục đích loại bỏ.

## JEP 406: Pattern Matching for switch (Pattern matching cho switch, preview)

Tương tự `instanceof`, `switch` cũng được bổ sung khả năng pattern matching theo type kèm chuyển đổi tự động.

Ví dụ code với `instanceof`:

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

Ví dụ code với `switch`:

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

Việc kiểm tra giá trị `null` cũng được tối ưu.

```java
// Old code
static void testFooBar(String s) {
    if (s == null) {
        System.out.println("oops!");
        return;
    }
    switch (s) {
        case "Foo", "Bar" -> System.out.println("Great");
        default           -> System.out.println("Ok");
    }
}

// New code
static void testFooBar(String s) {
    switch (s) {
        case null         -> System.out.println("Oops");
        case "Foo", "Bar" -> System.out.println("Great");
        default           -> System.out.println("Ok");
    }
}
```

## JEP 407: Remove RMI Activation (Loại bỏ cơ chế RMI Activation)

Loại bỏ cơ chế kích hoạt của Remote Method Invocation (RMI), đồng thời giữ lại các thành phần còn lại của RMI. Cơ chế kích hoạt RMI đã lỗi thời và không còn được sử dụng.

## JEP 409: Sealed Classes (Sealed class)

Sealed class được đề xuất dưới dạng preview bởi [JEP 360](https://openjdk.java.net/jeps/360) và được tích hợp vào Java 15. Trong JDK 16, sealed class được cải tiến (kiểm tra tham chiếu chặt chẽ hơn và quan hệ inheritance của sealed class), sau đó được preview lần hai theo [JEP 397](https://openjdk.java.net/jeps/397).

Trong [Tổng quan tính năng mới của Java 14 & 15](./java14-15.md), sealed class đã được trình bày chi tiết, nên không nhắc lại ở đây.

## JEP 410: Remove the Experimental AOT and JIT Compiler (Loại bỏ các AOT và JIT compiler thử nghiệm)

Trong Java 9, [JEP 295](https://openjdk.java.net/jeps/295) đã giới thiệu AOT compiler thử nghiệm, dùng để biên dịch các class Java thành native code trước khi khởi động virtual machine.

Java 17 loại bỏ các AOT và JIT compiler thử nghiệm vì các compiler này ít được sử dụng kể từ khi ra mắt, trong khi công sức cần thiết để bảo trì lại rất lớn. JVM Compiler Interface (JVMCI) thử nghiệm ở cấp Java vẫn được giữ lại, để developer có thể tiếp tục sử dụng các phiên bản compiler được build bên ngoài cho JIT compilation.

## JEP 411: Deprecate the Security Manager for Removal (Đánh dấu Security Manager là deprecated để loại bỏ)

Đánh dấu Security Manager là deprecated để loại bỏ trong các phiên bản tương lai.

Security Manager có từ Java 1.0. Trong nhiều năm qua, nó chưa bao giờ là phương thức chính để bảo vệ code Java phía client và cũng hiếm khi được dùng để bảo vệ code phía server. Để thúc đẩy Java phát triển, Java 17 đánh dấu Security Manager là deprecated để loại bỏ cùng với Applet API cũ ([JEP 398](https://openjdk.java.net/jeps/398)).

## JEP 412: Foreign Function & Memory API (Foreign Function & Memory API, incubator lần thứ nhất)

Chương trình Java có thể dùng API này để tương tác với code và data bên ngoài Java runtime. Thông qua việc gọi hiệu quả các foreign function (tức code bên ngoài JVM) và truy cập an toàn vào foreign memory (tức memory không do JVM quản lý), API này cho phép chương trình Java gọi native library và xử lý native data mà an toàn hơn và ít mong manh hơn JNI.

Foreign Function & Memory API được đưa vào incubator lần đầu trong Java 17, do [JEP 412](https://openjdk.java.net/jeps/412) đề xuất. Vòng incubator thứ hai do [JEP 419](https://openjdk.org/jeps/419) đề xuất và được tích hợp vào Java 18, còn preview do [JEP 424](https://openjdk.org/jeps/424) đề xuất và được tích hợp vào Java 19.

Trong [Tổng quan tính năng mới của Java 19](./java19.md), Foreign Function & Memory API đã được trình bày chi tiết, nên không nhắc lại ở đây.

## JEP 414: Vector API (Second Incubator) (Vector API, incubator lần thứ hai)

Vector API ban đầu được đề xuất bởi [JEP 338](https://openjdk.java.net/jeps/338) và được tích hợp vào Java 16 dưới dạng [incubator API](http://openjdk.java.net/jeps/11). Vòng incubator thứ hai do [JEP 414](https://openjdk.java.net/jeps/414) đề xuất và được tích hợp vào Java 17, vòng incubator thứ ba do [JEP 417](https://openjdk.java.net/jeps/417) đề xuất và được tích hợp vào Java 18, còn vòng thứ tư do [JEP 426](https://openjdk.java.net/jeps/426) đề xuất và được tích hợp vào Java 19.

Incubator API này cung cấp phiên bản ban đầu của API để biểu diễn một số phép tính vector. Khi runtime, các phép tính này được compile một cách đáng tin cậy thành các instruction phần cứng vector tối ưu trên kiến trúc CPU được hỗ trợ, nhờ đó đạt performance cao hơn so với phép tính scalar tương đương và tận dụng tối đa kỹ thuật Single Instruction, Multiple Data (SIMD), một loại instruction có trên hầu hết CPU hiện đại. Mặc dù HotSpot hỗ trợ vectorization tự động, tập hợp các phép toán scalar có thể chuyển đổi còn hạn chế và dễ bị ảnh hưởng khi code thay đổi. API này giúp developer dễ dàng viết các thuật toán vector portable và có performance cao bằng Java.

Trong [Tổng quan tính năng mới của Java 18](./java18.md), Vector API đã được trình bày chi tiết, nên không nhắc lại ở đây.

<!-- @include: @article-footer.snippet.md -->
