---
title: Giải thích chi tiết về CAS
description: "Phân tích chuyên sâu về CAS (compare and swap): giải thích chi tiết nguyên lý thao tác atomic của CAS, triển khai bằng class Unsafe, vấn đề ABA và giải pháp, cơ chế spin lock, cùng so sánh hiệu suất với pessimistic lock."
category: Java
tag:
  - Java Concurrency
head:
  - - meta
    - name: keywords
      content: CAS,Compare-And-Swap,thao tác atomic,vấn đề ABA,spin lock,optimistic lock,Unsafe,nguyên lý CAS
---

Bạn có thể đọc bài viết do tác giả biên soạn về optimistic lock, pessimistic lock và các cách triển khai thường gặp của optimistic lock: [Giải thích chi tiết về optimistic lock và pessimistic lock](https://javaguide.cn/java/concurrent/optimistic-lock-and-pessimistic-lock.html).

Bài viết này chủ yếu giới thiệu cách triển khai CAS trong Java và một số vấn đề tồn tại của CAS.

## CAS trong Java được triển khai như thế nào?

Trong Java, một class quan trọng để triển khai thao tác CAS (Compare-And-Swap, compare and swap) là `Unsafe`.

Class `Unsafe` nằm trong package `sun.misc`, cung cấp các thao tác cấp thấp, không an toàn. Do có chức năng mạnh và tiềm ẩn nguy hiểm, class này thường được dùng bên trong JVM hoặc trong các library cần hiệu suất cực cao và khả năng truy cập tầng dưới, không khuyến nghị developer thông thường sử dụng trong application. Bạn có thể đọc bài viết này để xem giới thiệu chi tiết về class `Unsafe`: 📌[Giải thích chi tiết về class ma thuật Unsafe của Java](https://javaguide.cn/java/basis/unsafe.html).

Class `Unsafe` trong package `sun.misc` cung cấp các method `compareAndSwapObject`, `compareAndSwapInt`, `compareAndSwapLong` để triển khai thao tác CAS cho các kiểu `Object`, `int`, `long`:

```java
/**
 * Cập nhật giá trị của field trong object theo cách atomic.
 *
 * @param o        object cần thao tác
 * @param offset   offset memory của field trong object
 * @param expected  giá trị cũ được kỳ vọng
 * @param x        giá trị mới cần thiết lập
 * @return trả về true nếu cập nhật thành công; ngược lại trả về false
 */
boolean compareAndSwapObject(Object o, long offset, Object expected, Object x);

/**
 * Cập nhật giá trị của field trong object có kiểu int theo cách atomic.
 */
boolean compareAndSwapInt(Object o, long offset, int expected, int x);

/**
 * Cập nhật giá trị của field trong object có kiểu long theo cách atomic.
 */
boolean compareAndSwapLong(Object o, long offset, long expected, long x);
```

Các method CAS do `Unsafe` cung cấp trong JDK 8 là method `native`. Code Java dùng chúng để biểu đạt ngữ nghĩa compare and swap atomic; HotSpot thường nhận diện các lời gọi liên quan là hàm nội tại của JVM (intrinsic), sau đó ánh xạ thành instruction atomic được processor mục tiêu hỗ trợ hoặc một cơ chế tương đương. Cách triển khai cụ thể phụ thuộc vào JVM và kiến trúc CPU, nhưng không thể đơn giản khái quát là “gọi inline assembly C++ thông qua JNI”.

Package `java.util.concurrent.atomic` cung cấp một số class dùng cho các thao tác atomic.

![Tổng quan về các class atomic của JUC](https://oss.javaguide.cn/github/javaguide/java/JUC%E5%8E%9F%E5%AD%90%E7%B1%BB%E6%A6%82%E8%A7%88.png)

Bạn có thể đọc bài viết này để xem giới thiệu và cách sử dụng các class Atomic: [Tổng hợp các class Atomic](https://javaguide.cn/java/concurrent/atomic-classes.html).

Các class Atomic dựa vào optimistic lock CAS để bảo đảm tính atomic của method mà không cần dùng cơ chế lock truyền thống như block `synchronized` hoặc `ReentrantLock`.

`AtomicInteger` là một trong các class atomic của Java, chủ yếu dùng để thao tác atomic với biến kiểu `int`. Bản triển khai JDK 8 dưới đây sử dụng các method thao tác atomic cấp thấp do `Unsafe` cung cấp; trong các JDK mới hơn, API liên quan và chi tiết triển khai bên trong có thể khác, còn application code cũng có thể dùng `VarHandle` tiêu chuẩn để biểu đạt ngữ nghĩa truy cập atomic.

Tiếp theo, chúng ta sẽ phân tích source code cốt lõi của `AtomicInteger` (JDK1.8) để giải thích cách Java dùng các method của class `Unsafe` nhằm triển khai thao tác atomic.

Source code cốt lõi của `AtomicInteger` như sau:

```java
// Lấy instance Unsafe
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;

static {
    try {
        // Lấy offset memory của field “value” trong class AtomicInteger
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}
// Bảo đảm visibility của field “value”
private volatile int value;

// Nếu giá trị hiện tại bằng giá trị kỳ vọng, atomic set giá trị thành newValue
// Dùng method Unsafe#compareAndSwapInt để thực hiện CAS
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}

// Atomic cộng delta vào giá trị hiện tại và trả về giá trị cũ
public final int getAndAdd(int delta) {
    return unsafe.getAndAddInt(this, valueOffset, delta);
}

// Atomic cộng 1 vào giá trị hiện tại và trả về giá trị trước khi cộng (giá trị cũ)
// Dùng method Unsafe#getAndAddInt để thực hiện CAS.
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}

// Atomic trừ 1 khỏi giá trị hiện tại và trả về giá trị trước khi trừ (giá trị cũ)
public final int getAndDecrement() {
    return unsafe.getAndAddInt(this, valueOffset, -1);
}
```

Source code của `Unsafe#getAndAddInt`:

```java
// Lấy và tăng giá trị integer theo cách atomic
public final int getAndAddInt(Object o, long offset, int delta) {
    int v;
    do {
        // Lấy giá trị integer tại offset memory offset của object o theo cách volatile
        v = getIntVolatile(o, offset);
    } while (!compareAndSwapInt(o, offset, v, v + delta));
    // Trả về giá trị cũ
    return v;
}
```

Có thể thấy `getAndAddInt` sử dụng vòng lặp `do-while`: khi thao tác `compareAndSwapInt` thất bại, nó sẽ liên tục retry cho đến khi thành công. Nói cách khác, method `getAndAddInt` thử cập nhật giá trị của `value` thông qua method `compareAndSwapInt`; nếu cập nhật thất bại (giá trị hiện tại bị thread khác thay đổi trong lúc đó), nó sẽ lấy lại giá trị hiện tại và thử cập nhật lần nữa cho đến khi thành công.

Do thao tác CAS có thể thất bại vì xung đột concurrent, nó thường được kết hợp với vòng lặp `while` để liên tục retry sau khi thất bại cho đến khi thao tác thành công. Đây chính là **cơ chế spin lock**.

## CAS có những vấn đề nào?

Vấn đề ABA là vấn đề thường gặp nhất của thuật toán CAS.

### Vấn đề ABA

Nếu khi đọc lần đầu, một biến V có giá trị A, và khi chuẩn bị gán giá trị, kiểm tra thấy nó vẫn là A, liệu có thể kết luận rằng giá trị của nó chưa từng bị thread khác thay đổi không? Rõ ràng là không, vì trong khoảng thời gian đó, giá trị có thể đã bị đổi thành giá trị khác rồi đổi lại thành A. Khi đó, thao tác CAS sẽ nhầm tưởng rằng giá trị chưa từng bị thay đổi. Vấn đề này được gọi là **vấn đề “ABA” của thao tác CAS.**

Cách giải quyết vấn đề ABA là thêm **version number hoặc timestamp** vào trước biến. Class `AtomicStampedReference` được Java cung cấp từ JDK 1.5 trở đi dùng để giải quyết vấn đề ABA. Method `compareAndSet()` của class này trước tiên kiểm tra reference hiện tại có bằng reference kỳ vọng hay không, đồng thời stamp hiện tại có bằng stamp kỳ vọng hay không; nếu tất cả đều bằng nhau, nó sẽ atomic set giá trị của reference và stamp thành giá trị update được cung cấp.

```java
public boolean compareAndSet(V   expectedReference,
                             V   newReference,
                             int expectedStamp,
                             int newStamp) {
    Pair<V> current = pair;
    return
        expectedReference == current.reference &&
        expectedStamp == current.stamp &&
        ((newReference == current.reference &&
          newStamp == current.stamp) ||
         casPair(current, Pair.of(newReference, newStamp)));
}
```

### Vòng lặp lâu gây tốn nhiều chi phí

CAS thường dùng thao tác spin để retry, tức là nếu chưa thành công thì liên tục thực thi vòng lặp cho đến khi thành công. Nếu trong thời gian dài vẫn không thành công, nó sẽ khiến CPU phải chịu chi phí thực thi rất lớn.

Nếu JVM hỗ trợ instruction `pause` do processor cung cấp, hiệu suất của thao tác spin sẽ được cải thiện. Instruction `pause` có hai tác dụng quan trọng:

1. **Trì hoãn thực thi instruction trong pipeline**: instruction `pause` có thể trì hoãn việc thực thi instruction, từ đó giảm mức tiêu thụ tài nguyên CPU. Thời gian trì hoãn cụ thể phụ thuộc vào phiên bản triển khai của processor; trên một số processor, thời gian trì hoãn có thể bằng không.
2. **Tránh xung đột thứ tự memory**: khi thoát vòng lặp, instruction `pause` có thể tránh việc pipeline của CPU bị xóa do xung đột thứ tự memory, từ đó nâng cao hiệu suất thực thi của CPU.

### Chỉ bảo đảm thao tác atomic trên một shared variable

Thao tác CAS chỉ có hiệu lực với một shared variable. Khi cần thao tác trên nhiều shared variable, CAS trở nên bất lực. Tuy nhiên, từ JDK 1.5, Java cung cấp class `AtomicReference`, cho phép chúng ta bảo đảm tính atomic giữa các reference object. Bằng cách đóng gói nhiều biến vào một object, chúng ta có thể dùng `AtomicReference` để thực hiện thao tác CAS.

Ngoài cách dùng `AtomicReference`, cũng có thể dùng lock để bảo đảm tính atomic.

## Tổng kết

Trong Java, các API như class atomic và `VarHandle` có thể biểu đạt thao tác CAS; JVM sẽ dựa trên platform mục tiêu để triển khai chúng thành instruction atomic được processor hỗ trợ hoặc một cơ chế tương đương. Cách triển khai cụ thể phụ thuộc vào JVM và kiến trúc CPU, không bị Java specification giới hạn phải dùng JNI hay một cách viết assembly cụ thể nào.

CAS có đặc tính hiệu suất cao và không cần lock, nhưng cũng cần lưu ý các vấn đề như ABA và chi phí lớn khi vòng lặp kéo dài.
