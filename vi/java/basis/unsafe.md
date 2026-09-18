---
title: "Giải thích chi tiết về lớp ma thuật Unsafe của Java"
description: "Phân tích chuyên sâu về lớp ma thuật Unsafe của Java: giới thiệu thao tác bộ nhớ trực tiếp, thao tác atomic CAS, khởi tạo object và các khả năng tầng thấp, giúp hiểu nguyên lý triển khai và rủi ro khi sử dụng các công cụ concurrent trong JUC."
category: Java
tag:
  - Java Basics
head:
  - - meta
    - name: keywords
      content: lớp Unsafe,thao tác bộ nhớ,thao tác atomic CAS,bộ nhớ off-heap,bộ nhớ trực tiếp,sun.misc.Unsafe,triển khai tầng dưới của JUC
---

> Bài viết này được biên soạn và hoàn thiện từ hai bài viết xuất sắc dưới đây:
>
> - [Lớp ma thuật Java: Phân tích ứng dụng của Unsafe - Nhóm kỹ thuật Meituan - 2019](https://tech.meituan.com/2019/02/14/talk-about-java-magic-class-unsafe.html)
> - [Unsafe, con dao hai lưỡi của Java: Giải thích chi tiết về lớp Unsafe - Nhóm Mã Nông - 2021](https://xie.infoq.cn/article/8b6ed4195e475bfb32dacc5cb)

<!-- markdownlint-disable MD024 -->

Nếu đã đọc source code của JUC, bạn chắc chắn sẽ nhận thấy rất nhiều công cụ concurrent gọi một lớp có tên là `Unsafe`.

Vậy lớp này chủ yếu dùng để làm gì? Có những trường hợp sử dụng nào? Bài viết này sẽ giúp bạn hiểu rõ!

## Giới thiệu về Unsafe

`Unsafe` là một lớp nằm trong package `sun.misc`, chủ yếu cung cấp các method thực hiện những thao tác tầng thấp, không an toàn, chẳng hạn truy cập trực tiếp tài nguyên memory của hệ thống, tự quản lý tài nguyên memory. Các method này đóng vai trò quan trọng trong việc nâng cao hiệu suất chạy của Java và tăng khả năng thao tác với tài nguyên tầng dưới của ngôn ngữ Java. Tuy nhiên, vì lớp `Unsafe` khiến ngôn ngữ Java có khả năng thao tác không gian memory tương tự pointer trong ngôn ngữ C, nên không thể phủ nhận rằng nó cũng làm tăng rủi ro phát sinh các vấn đề liên quan đến pointer. Việc sử dụng lớp `Unsafe` quá mức hoặc không đúng cách trong chương trình sẽ làm tăng xác suất xảy ra lỗi, khiến ngôn ngữ vốn an toàn như Java trở nên không còn “an toàn”. Vì vậy, cần hết sức thận trọng khi sử dụng `Unsafe`.

Ngoài ra, việc triển khai các chức năng do `Unsafe` cung cấp cần dựa vào native method. Bạn có thể xem native method là các method được viết bằng ngôn ngữ lập trình khác và được sử dụng trong Java. Native method được khai báo bằng keyword **`native`**; trong code Java chỉ khai báo method signature, còn phần triển khai cụ thể được giao cho **native code**.

![](https://oss.javaguide.cn/github/javaguide/java/basis/unsafe/image-20220717115231125.png)

**Tại sao cần sử dụng native method?**

1. Cần sử dụng các tính năng phụ thuộc hệ điều hành mà Java không có. Để vừa thực hiện cross-platform vừa kiểm soát tầng dưới, Java cần mượn sức mạnh của ngôn ngữ khác.
2. Với một số chức năng có sẵn đã được ngôn ngữ khác triển khai, Java có thể gọi trực tiếp.
3. Khi chương trình nhạy cảm với thời gian hoặc yêu cầu hiệu suất rất cao, cần sử dụng ngôn ngữ ở tầng thấp hơn, chẳng hạn C/C++ hoặc thậm chí assembly.

Nhiều công cụ concurrent trong package JUC gọi native method khi triển khai cơ chế concurrent. Nhờ đó, chúng phá vỡ ranh giới runtime của Java và có thể tiếp cận một số chức năng tầng dưới của hệ điều hành. Cùng một native method có thể được triển khai theo những cách khác nhau trên các hệ điều hành khác nhau, nhưng điều đó trong suốt với người sử dụng và cuối cùng đều cho cùng một kết quả.

## Tạo Unsafe

Một phần source code của `sun.misc.Unsafe` như sau:

```java
public final class Unsafe {
  // Object singleton
  private static final Unsafe theUnsafe;
  ......
  private Unsafe() {
  }
  @CallerSensitive
  public static Unsafe getUnsafe() {
    Class var0 = Reflection.getCallerClass();
    // Chỉ hợp lệ khi được BootstrapClassLoader của bootstrap class loader tải
    if(!VM.isSystemDomainLoader(var0.getClassLoader())) {
      throw new SecurityException("Unsafe");
    } else {
      return theUnsafe;
    }
  }
}
```

Lớp `Unsafe` được triển khai theo singleton, cung cấp static method `getUnsafe` để lấy instance `Unsafe`. Nhìn qua, method này có vẻ dùng được để lấy instance `Unsafe`. Tuy nhiên, khi gọi trực tiếp static method này, một exception `SecurityException` sẽ được ném ra:

```bash
Exception in thread "main" java.lang.SecurityException: Unsafe
 at sun.misc.Unsafe.getUnsafe(Unsafe.java:90)
 at com.cn.test.GetUnsafeTest.main(GetUnsafeTest.java:12)
```

**Tại sao method `public static` không thể được gọi trực tiếp?**

Đó là vì trong method `getUnsafe`, `classLoader` của caller sẽ được kiểm tra để xác định class hiện tại có do `Bootstrap classLoader` tải hay không. Nếu không phải thì một exception `SecurityException` sẽ được ném ra. Nói cách khác, chỉ class được bootstrap class loader tải mới có thể gọi các method trong lớp Unsafe, nhằm ngăn các method này bị gọi trong code không đáng tin cậy.

**Tại sao việc sử dụng lớp Unsafe lại bị giới hạn nghiêm ngặt như vậy?**

Các chức năng do `Unsafe` cung cấp quá thấp (chẳng hạn truy cập trực tiếp tài nguyên memory của hệ thống, tự quản lý tài nguyên memory), nên cũng tiềm ẩn rủi ro bảo mật lớn. Nếu sử dụng không đúng cách, rất dễ phát sinh vấn đề nghiêm trọng.

**Nếu muốn sử dụng lớp `Unsafe` thì lấy instance của nó bằng cách nào?**

Dưới đây là hai cách khả thi.

1. Dùng reflection để lấy object singleton `theUnsafe` đã được khởi tạo trong lớp Unsafe.

```java
private static Unsafe reflectGetUnsafe() {
    try {
      Field field = Unsafe.class.getDeclaredField("theUnsafe");
      field.setAccessible(true);
      return (Unsafe) field.get(null);
    } catch (Exception e) {
      log.error(e.getMessage(), e);
      return null;
    }
}
```

2. Dựa trên điều kiện giới hạn của method `getUnsafe`, dùng command Java `-Xbootclasspath/a` để thêm path jar chứa class A gọi các method liên quan đến Unsafe vào bootstrap path mặc định. Khi đó A được bootstrap class loader tải và có thể lấy instance Unsafe một cách an toàn thông qua method `Unsafe.getUnsafe`.

```bash
java -Xbootclasspath/a: ${path}   // path là đường dẫn jar chứa class gọi các method liên quan đến Unsafe
```

## Chức năng của Unsafe

Nhìn chung, các chức năng do lớp `Unsafe` triển khai có thể được chia thành 8 nhóm sau:

1. Thao tác memory
2. Memory barrier
3. Thao tác object
4. Thao tác data
5. Thao tác CAS
6. Điều phối thread
7. Thao tác Class
8. Thông tin hệ thống

### Thao tác memory

#### Giới thiệu

Nếu bạn từng viết chương trình bằng C hoặc C++, chắc chắn bạn không xa lạ với thao tác memory. Còn trong Java, không được phép thao tác trực tiếp với memory; việc cấp phát và thu hồi memory của object đều do JVM tự triển khai. Tuy nhiên, `Unsafe` cung cấp các API sau để thao tác trực tiếp với memory:

```java
// Cấp phát không gian native mới
public native long allocateMemory(long bytes);
// Điều chỉnh lại kích thước không gian memory
public native long reallocateMemory(long address, long bytes);
// Đặt memory thành giá trị chỉ định
public native void setMemory(Object o, long offset, long bytes, byte value);
// Sao chép memory
public native void copyMemory(Object srcBase, long srcOffset,Object destBase, long destOffset,long bytes);
// Xóa memory
public native void freeMemory(long address);
```

Dùng code sau để kiểm thử:

```java
private void memoryTest() {
    int size = 4;
    // 1. Cấp phát memory ban đầu
    long oldAddr = unsafe.allocateMemory(size);
    System.out.println("Initial address: " + oldAddr);

    // 2. Ghi dữ liệu vào memory ban đầu
    unsafe.putInt(oldAddr, 16843009); // Ghi 0x01010101
    System.out.println("Value at oldAddr: " + unsafe.getInt(oldAddr));

    // 3. Cấp phát lại memory
    long newAddr = unsafe.reallocateMemory(oldAddr, size * 2);
    System.out.println("New address: " + newAddr);

    // 4. reallocateMemory đã sao chép dữ liệu từ oldAddr sang newAddr
    // Vì vậy 4 byte đầu tiên của newAddr phải giống nội dung của oldAddr
    System.out.println("Value at newAddr (first 4 bytes): " + unsafe.getInt(newAddr));

    // Điểm quan trọng: mọi thao tác sau đó phải dựa trên newAddr, oldAddr đã hết hiệu lực!
    try {
        // 5. Ghi dữ liệu mới vào nửa sau của block memory mới
        unsafe.putInt(newAddr + size, 33686018); // Ghi 0x02020202

        // 6. Đọc giá trị long có đủ 8 byte
        System.out.println("Value at newAddr (full 8 bytes): " + unsafe.getLong(newAddr));

    } finally {
        // 7. Chỉ giải phóng địa chỉ memory còn hiệu lực cuối cùng
        unsafe.freeMemory(newAddr);
        // Nếu thử freeMemory(oldAddr), sẽ gây ra lỗi double free!
    }
}
```

Trước hết hãy xem output:

```plain
Initial address: 140467048086752
Value at oldAddr: 16843009
New address: 140467048086752
Value at newAddr (first 4 bytes): 16843009
Value at newAddr (full 8 bytes): 144680345659310337
```

Hành vi của `reallocateMemory` tương tự hàm realloc trong ngôn ngữ C. Nó sẽ cố gắng mở rộng hoặc thu hẹp memory block mà không di chuyển dữ liệu. Hành vi này chủ yếu có hai trường hợp:

1. **Mở rộng tại chỗ**: Nếu phía sau memory block hiện tại có đủ không gian trống liên tiếp, `reallocateMemory` sẽ mở rộng memory ngay tại địa chỉ ban đầu và trả về địa chỉ ban đầu.
2. **Mở rộng sang vị trí khác**: Nếu không đủ không gian phía sau memory block hiện tại, nó sẽ tìm một vùng memory mới đủ lớn, sao chép dữ liệu cũ sang đó, giải phóng địa chỉ memory cũ rồi trả về địa chỉ mới.

**Dựa trên kết quả chạy lần này, có thể phân tích như sau:**

**Bước một: Cấp phát ban đầu và ghi dữ liệu**

- `unsafe.allocateMemory(size)` cấp phát 4 byte memory off-heap, địa chỉ là `140467048086752`.
- `unsafe.putInt(oldAddr, 16843009)` ghi giá trị int `16843009` vào địa chỉ đó, biểu diễn hexadecimal là `0x01010101`. `getInt` đọc đúng giá trị, chứng minh việc ghi thành công.

**Bước hai: Mở rộng memory tại chỗ**

- `long newAddr = unsafe.reallocateMemory(oldAddr, size * 2)` thử mở rộng memory block lên 8 byte.
- Quan sát output New address: `140467048086752`, ta thấy giá trị của `newAddr` và `oldAddr` **hoàn toàn giống nhau**.
- Điều này cho thấy thao tác lần này đã thực hiện “mở rộng tại chỗ”. Hệ thống tìm thấy đủ không gian sau địa chỉ ban đầu `140467048086752` và trực tiếp mở rộng memory block lên 8 byte. Trong quá trình này, địa chỉ cũ `oldAddr` vẫn còn hiệu lực và chính là `newAddr`, dữ liệu cũng không bị di chuyển.

**Bước ba: Xác minh dữ liệu và ghi dữ liệu mới**

- `unsafe.getInt(newAddr)` đọc lại 4 byte đầu tiên, kết quả vẫn là `16843009`, xác nhận dữ liệu cũ còn nguyên vẹn.
- `unsafe.putInt(newAddr + size, 33686018)` ghi giá trị int mới `33686018` vào 4 byte phía sau phần memory được mở rộng (offset là 4), giá trị hexadecimal là `0x02020202`.

**Bước bốn: Đọc dữ liệu đầy đủ**

- `unsafe.getLong(newAddr)` đọc một giá trị long (8 byte) từ địa chỉ bắt đầu. Lúc này 8 byte trong memory là sự ghép nối của `0x01010101` (địa chỉ thấp) và `0x02020202` (địa chỉ cao).
- Trên máy có byte order little-endian, 8 byte này được diễn giải thành số hexadecimal `0x0202020201010101`.
- Chuyển số hexadecimal này sang decimal cho kết quả chính là `144680345659310337`. Điều này giải thích hoàn toàn output cuối cùng.

**Bước năm: Giải phóng memory an toàn**

- Trong block `finally`, `unsafe.freeMemory(newAddr)` giải phóng an toàn memory block 8 byte.
- Vì lần này là mở rộng tại chỗ (`oldAddr == newAddr`), nên ngay cả khi vô tình gọi thêm `freeMemory(oldAddr)` cũng sẽ gây lỗi nghiêm trọng do giải phóng lần hai.

#### Ứng dụng điển hình

`DirectByteBuffer` là một class quan trọng Java dùng để triển khai memory off-heap, thường được dùng làm buffer pool trong quá trình giao tiếp và được sử dụng rộng rãi trong các framework NIO như Netty, MINA. Logic tạo, sử dụng và hủy `DirectByteBuffer` đều được triển khai thông qua API memory off-heap do Unsafe cung cấp.

**Tại sao cần sử dụng memory off-heap?**

- Giảm pause do garbage collection. Vì memory off-heap do hệ điều hành quản lý trực tiếp thay vì JVM, nên khi sử dụng memory off-heap, có thể giữ quy mô memory trong heap nhỏ hơn. Nhờ đó giảm ảnh hưởng của pause khi GC đến ứng dụng.
- Nâng cao hiệu suất thao tác I/O của chương trình. Trong quá trình giao tiếp I/O thường tồn tại thao tác sao chép dữ liệu từ memory trong heap sang memory off-heap. Với dữ liệu tạm có thời gian sống ngắn và cần sao chép giữa các vùng memory thường xuyên, nên lưu trữ trong memory off-heap.

Dưới đây là constructor của `DirectByteBuffer`. Khi tạo `DirectByteBuffer`, nó dùng `Unsafe.allocateMemory` để cấp phát memory, `Unsafe.setMemory` để khởi tạo memory, sau đó tạo object `Cleaner` để theo dõi garbage collection của object `DirectByteBuffer`. Nhờ đó, khi `DirectByteBuffer` bị garbage collection, memory off-heap đã cấp phát cũng được giải phóng.

```java
DirectByteBuffer(int cap) {                   // package-private

    super(-1, 0, cap, cap);
    boolean pa = VM.isDirectMemoryPageAligned();
    int ps = Bits.pageSize();
    long size = Math.max(1L, (long)cap + (pa ? ps : 0));
    Bits.reserveMemory(size, cap);

    long base = 0;
    try {
        // Cấp phát memory và trả về địa chỉ cơ sở
        base = unsafe.allocateMemory(size);
    } catch (OutOfMemoryError x) {
        Bits.unreserveMemory(size, cap);
        throw x;
    }
    // Khởi tạo memory
    unsafe.setMemory(base, size, (byte) 0);
    if (pa && (base % ps != 0)) {
        // Làm tròn lên biên page
        address = base + ps - (base & (ps - 1));
    } else {
        address = base;
    }
    // Theo dõi garbage collection của object DirectByteBuffer để giải phóng memory off-heap
    cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
    att = null;
}
```

### Memory barrier

#### Giới thiệu

Trước khi giới thiệu memory barrier, cần biết rằng compiler và CPU có thể reorder code trong điều kiện vẫn đảm bảo kết quả output của chương trình nhất quán, qua đó nâng cao hiệu suất từ góc độ tối ưu instruction. Tuy nhiên, instruction reordering có thể gây ra kết quả không mong muốn, khiến dữ liệu trong CPU cache và memory không nhất quán. Memory barrier (`Memory Barrier`) ngăn instruction ở hai phía barrier bị reorder, từ đó tránh các tối ưu không chính xác của compiler và hardware.

Ở tầng hardware, memory barrier là instruction do CPU cung cấp để ngăn code bị reorder. Cách triển khai memory barrier có thể khác nhau trên các hardware platform khác nhau. Trong Java 8, ba function memory barrier đã được bổ sung. Chúng che giấu khác biệt ở tầng dưới của hệ điều hành, cho phép định nghĩa trong code và giao cho JVM thống nhất việc sinh instruction memory barrier để triển khai chức năng memory barrier.

`Unsafe` cung cấp ba method liên quan đến memory barrier sau:

```java
// Memory barrier, cấm reorder thao tác load. Thao tác load trước barrier không thể bị reorder ra sau barrier; thao tác load sau barrier không thể bị reorder ra trước barrier
public native void loadFence();
// Memory barrier, cấm reorder thao tác store. Thao tác store trước barrier không thể bị reorder ra sau barrier; thao tác store sau barrier không thể bị reorder ra trước barrier
public native void storeFence();
// Memory barrier, cấm reorder thao tác load và store
public native void fullFence();
```

Memory barrier ràng buộc việc reorder và semantics về visibility giữa các lần truy cập memory được chỉ định, không tương đương với “xóa CPU cache” hoặc bắt buộc đọc lại toàn bộ dữ liệu từ main memory. `loadFence()` chỉ cung cấp ràng buộc ordering ở phía đọc, không thể một mình thiết lập quan hệ `happens-before` trong Java Memory Model giữa hai lần truy cập field thông thường.

Vì vậy, không thể dùng “field `boolean` thông thường + `loadFence()`” để thay thế `volatile` nhằm đảm bảo visibility giữa các thread. Shared flag nên sử dụng `volatile`, lock hoặc cơ chế synchronization như `VarHandle` có pattern truy cập đọc/ghi tương ứng; nếu không, code vẫn tồn tại data race và thread đọc không được đảm bảo sẽ quan sát thấy giá trị đã ghi.

#### Ứng dụng điển hình

Trong Java 8, một cơ chế lock mới là `StampedLock` đã được giới thiệu. Có thể xem đây là phiên bản cải tiến của read-write lock. `StampedLock` cung cấp triển khai optimistic read lock. Optimistic read lock này gần giống thao tác lock-free, hoàn toàn không block thread ghi lấy write lock, từ đó giảm hiện tượng thread ghi bị “starvation” khi đọc nhiều, ghi ít. Vì optimistic read lock do `StampedLock` cung cấp không block việc thread ghi lấy read lock, nên khi thread load shared variable từ main memory vào working memory, có thể phát sinh vấn đề dữ liệu không nhất quán.

Để giải quyết vấn đề này, method `validate` của `StampedLock` thêm một load memory barrier thông qua method `loadFence` của `Unsafe`.

```java
public boolean validate(long stamp) {
   U.loadFence();
   return (stamp & SBITS) == (state & SBITS);
}
```

### Thao tác object

#### Giới thiệu

**Ví dụ**

```java
import sun.misc.Unsafe;
import java.lang.reflect.Field;

public class Main {

    private int value;

    public static void main(String[] args) throws Exception{
        Unsafe unsafe = reflectGetUnsafe();
        assert unsafe != null;
        long offset = unsafe.objectFieldOffset(Main.class.getDeclaredField("value"));
        Main main = new Main();
        System.out.println("value before putInt: " + main.value);
        unsafe.putInt(main, offset, 42);
        System.out.println("value after putInt: " + main.value);
  System.out.println("value after putInt: " + unsafe.getInt(main, offset));
    }

    private static Unsafe reflectGetUnsafe() {
        try {
            Field field = Unsafe.class.getDeclaredField("theUnsafe");
            field.setAccessible(true);
            return (Unsafe) field.get(null);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

}
```

Output:

```plain
value before putInt: 0
value after putInt: 42
value after putInt: 42
```

**Thuộc tính object**

Việc lấy offset memory của member field trong object và sửa giá trị field đã được kiểm thử trong ví dụ trên. Ngoài các method `putInt`, `getInt` đã nêu, Unsafe cung cấp method `put` và `get` cho toàn bộ 8 kiểu dữ liệu primitive và `Object`. Mọi method `put` đều có thể vượt qua quyền truy cập để sửa trực tiếp dữ liệu trong memory. Đọc comment trong source code của OpenJDK cho thấy thao tác đọc/ghi kiểu dữ liệu primitive và `Object` hơi khác nhau: kiểu dữ liệu primitive thao tác trực tiếp trên giá trị thuộc tính (`value`), còn thao tác với `Object` dựa trên giá trị tham chiếu (`reference value`). Dưới đây là các method đọc/ghi `Object`:

```java
// Lấy một object reference tại offset được chỉ định của object
public native Object getObject(Object o, long offset);
// Ghi một object reference tại offset được chỉ định của object
public native void putObject(Object o, long offset, Object x);
```

Ngoài đọc/ghi thông thường thuộc tính object, `Unsafe` còn cung cấp method **đọc/ghi volatile** và **ghi có thứ tự**. Phạm vi của method đọc/ghi `volatile` giống đọc/ghi thông thường, bao gồm toàn bộ kiểu dữ liệu primitive và kiểu `Object`. Lấy kiểu `int` làm ví dụ:

```java
// Đọc giá trị int tại offset được chỉ định của object, hỗ trợ semantics volatile load
public native int getIntVolatile(Object o, long offset);
// Ghi một int tại offset được chỉ định của object, hỗ trợ semantics volatile store
public native void putIntVolatile(Object o, long offset, int x);
```

So với đọc/ghi thông thường, đọc/ghi `volatile` có chi phí cao hơn vì cần đảm bảo visibility và ordering. Khi thực hiện thao tác `get`, giá trị thuộc tính sẽ được lấy bắt buộc từ main memory; khi dùng method `put` để đặt giá trị thuộc tính, giá trị sẽ được cập nhật bắt buộc vào main memory, qua đó đảm bảo các thay đổi này có thể được thread khác nhìn thấy.

Các method ghi có thứ tự gồm ba method sau:

```java
public native void putOrderedObject(Object o, long offset, Object x);
public native void putOrderedInt(Object o, long offset, int x);
public native void putOrderedLong(Object o, long offset, long x);
```

Chi phí ghi có thứ tự thấp hơn `volatile` tương đối nhiều vì nó chỉ đảm bảo ordering khi ghi, không đảm bảo visibility. Nói cách khác, không thể đảm bảo giá trị do một thread ghi sẽ lập tức được thread khác nhìn thấy. Để hiểu sự khác biệt này, cần bổ sung thêm kiến thức về memory barrier. Trước hết, cần hiểu hai khái niệm instruction:

- `Load`: Sao chép dữ liệu trong main memory vào cache của processor
- `Store`: Flush dữ liệu trong cache của processor vào main memory

Điểm khác nhau giữa ghi tuần tự và ghi `volatile` là memory barrier được thêm khi ghi tuần tự thuộc loại `StoreStore`, còn memory barrier được thêm khi ghi `volatile` thuộc loại `StoreLoad`, như hình dưới đây:

![](https://oss.javaguide.cn/github/javaguide/java/basis/unsafe/image-20220717144834132.png)

Trong method ghi có thứ tự, sử dụng barrier `StoreStore`. Barrier này đảm bảo `Store1` lập tức flush dữ liệu vào memory, thao tác này xảy ra trước `Store2` và các thao tác lưu trữ tiếp theo. Còn khi ghi `volatile`, sử dụng barrier `StoreLoad`. Barrier này đảm bảo `Store1` lập tức flush dữ liệu vào memory, thao tác này xảy ra trước `Load2` và các instruction load tiếp theo. Đồng thời, barrier `StoreLoad` khiến mọi instruction truy cập memory trước barrier, bao gồm instruction lưu trữ và truy cập, phải hoàn thành rồi mới thực hiện instruction truy cập memory sau barrier.

Tóm lại, trong ba loại method ghi trên, hiệu suất ghi giảm dần theo thứ tự `put`, `putOrder`, `putVolatile`.

**Khởi tạo object**

Method `allocateInstance` của `Unsafe` cho phép khởi tạo object theo cách không thông thường. Trước hết, định nghĩa một entity class và gán giá trị cho member field trong constructor:

```java
@Data
public class A {
    private int b;
    public A(){
        this.b =1;
    }
}
```

So sánh các cách tạo object khác nhau dựa trên constructor, reflection và method `Unsafe`:

```java
public void objTest() throws Exception{
    A a1=new A();
    System.out.println(a1.getB());
    A a2 = A.class.newInstance();
    System.out.println(a2.getB());
    A a3= (A) unsafe.allocateInstance(A.class);
    System.out.println(a3.getB());
}
```

Kết quả lần lượt là 1, 1, 0. Điều này cho thấy khi tạo object bằng method `allocateInstance`, constructor của class sẽ không được gọi. Khi tạo object theo cách này, chỉ cần `Class` object. Vì vậy, nếu muốn bỏ qua giai đoạn khởi tạo object hoặc bỏ qua kiểm tra an toàn của constructor, có thể dùng cách này. Trong ví dụ trên, nếu đổi constructor của class A thành `private`, sẽ không thể tạo object thông qua constructor và reflection (có thể tạo object sau khi gọi `setAccessible` trên constructor object), nhưng method `allocateInstance` vẫn hoạt động.

#### Ứng dụng điển hình

- **Cách khởi tạo object thông thường**: Về bản chất, cách tạo object thường dùng đều tạo object thông qua cơ chế new. Tuy nhiên, cơ chế new có một đặc điểm: khi class chỉ cung cấp constructor có tham số và không khai báo tường minh constructor không tham số, phải dùng constructor có tham số để tạo object; khi dùng constructor có tham số, phải truyền đủ số lượng tham số tương ứng mới hoàn tất việc khởi tạo object.
- **Cách khởi tạo không thông thường**: `Unsafe` cung cấp method `allocateInstance`, chỉ cần `Class` object là có thể tạo instance của class này, đồng thời không cần gọi constructor, code khởi tạo hay kiểm tra an toàn của JVM. Nó ức chế việc kiểm tra modifier, nghĩa là kể cả constructor được khai báo `private` cũng có thể khởi tạo thông qua method này; chỉ cần cung cấp class object là có thể tạo object tương ứng. Nhờ đặc tính này, `allocateInstance` được sử dụng trong `java.lang.invoke`, Objenesis (cung cấp cách tạo object bỏ qua constructor của class) và Gson (dùng khi deserialization).

### Thao tác array

#### Giới thiệu

Kết hợp hai method `arrayBaseOffset` và `arrayIndexScale` có thể định vị vị trí của từng phần tử trong array ở memory.

```java
// Trả về offset của phần tử đầu tiên trong array
public native int arrayBaseOffset(Class<?> arrayClass);
// Trả về kích thước một phần tử trong array
public native int arrayIndexScale(Class<?> arrayClass);
```

#### Ứng dụng điển hình

Hai method liên quan đến thao tác dữ liệu này được ứng dụng điển hình trong `AtomicIntegerArray` thuộc package `java.util.concurrent.atomic` (có thể thực hiện thao tác atomic trên từng phần tử của array `Integer`). Như source code `AtomicIntegerArray` trong hình dưới đây, nó dùng `arrayBaseOffset` và `arrayIndexScale` của `Unsafe` để lần lượt lấy offset của phần tử đầu tiên trong array là `base` và hệ số kích thước của một phần tử là `scale`. Các thao tác atomic liên quan về sau đều dựa vào hai giá trị này để định vị phần tử trong array. Method `getAndAdd` ở hình thứ hai lấy offset của một phần tử array thông qua method `checkedByteOffset`, sau đó thực hiện thao tác atomic bằng CAS.

![](https://oss.javaguide.cn/github/javaguide/java/basis/unsafe/image-20220717144927257.png)

### Thao tác CAS

#### Giới thiệu

Phần này chủ yếu giới thiệu các method liên quan đến CAS.

```java
/**
  *  CAS
  * @param o         Object chứa field cần sửa
  * @param offset    Offset của field trong object
  * @param expected  Giá trị kỳ vọng
  * @param update    Giá trị cập nhật
  * @return          true | false
  */
public final native boolean compareAndSwapObject(Object o, long offset,  Object expected, Object update);

public final native boolean compareAndSwapInt(Object o, long offset, int expected,int update);

public final native boolean compareAndSwapLong(Object o, long offset, long expected, long update);
```

**CAS là gì?** CAS là viết tắt của Compare And Swap, nghĩa là so sánh và trao đổi. Đây là một kỹ thuật thường dùng khi triển khai concurrent algorithm. Thao tác CAS gồm ba toán hạng: memory location, giá trị gốc kỳ vọng và giá trị mới. Khi thực hiện CAS, nếu giá trị tại memory location giống giá trị kỳ vọng thì cập nhật nó thành giá trị mới theo cách atomic; nếu không thì không cập nhật. HotSpot ánh xạ các thao tác liên quan thành atomic primitive do platform đích cung cấp; trên x86 thường dùng `cmpxchg`, còn các kiến trúc processor khác có thể dùng instruction hoặc chuỗi instruction khác nhau.

#### Ứng dụng điển hình

Các công cụ concurrent trong package JUC sử dụng rất nhiều thao tác CAS. Các bài viết giới thiệu `synchronized` và `AQS` trước đó cũng nhiều lần đề cập đến CAS. Với vai trò optimistic lock, CAS được sử dụng rộng rãi trong các công cụ concurrent. Lớp `Unsafe` cung cấp các method `compareAndSwapObject`, `compareAndSwapInt`, `compareAndSwapLong` để thực hiện thao tác CAS trên kiểu `Object`, `int`, `long`. Lấy method `compareAndSwapInt` làm ví dụ:

```java
public final native boolean compareAndSwapInt(Object o, long offset,int expected,int x);
```

Trong các tham số, `o` là object cần cập nhật, `offset` là offset của field kiểu integer trong object `o`. Nếu giá trị của field này giống `expected`, giá trị field sẽ được đặt thành giá trị mới `x`. Thao tác cập nhật này không thể bị interrupt, tức là một thao tác atomic. Dưới đây là ví dụ sử dụng `compareAndSwapInt`:

```java
private volatile int a;
public static void main(String[] args){
    CasTest casTest=new CasTest();
    new Thread(()->{
        for (int i = 1; i < 5; i++) {
            casTest.increment(i);
            System.out.print(casTest.a+" ");
        }
    }).start();
    new Thread(()->{
        for (int i = 5 ; i <10 ; i++) {
            casTest.increment(i);
            System.out.print(casTest.a+" ");
        }
    }).start();
}

private void increment(int x){
    while (true){
        try {
            long fieldOffset = unsafe.objectFieldOffset(CasTest.class.getDeclaredField("a"));
            if (unsafe.compareAndSwapInt(this,fieldOffset,x-1,x))
                break;
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }
    }
}
```

Chạy code sẽ lần lượt output:

```plain
1 2 3 4 5 6 7 8 9
```

Nếu dán đoạn code trên vào IDE để chạy, bạn sẽ nhận thấy không thể nhận được output mong muốn. Một người dùng đã chỉ ra vấn đề này trên Github: [issue#2650](https://github.com/Snailclimb/JavaGuide/issues/2650). Dưới đây là code đã sửa:

```java
// Đóng gói thao tác tăng và in trong một method có tính atomic cao hơn
private void incrementAndPrint(int targetValue) {
    while (true) {
        int currentValue = a; // Đọc giá trị hiện tại của a
        // Nếu giá trị hiện tại đã đạt hoặc vượt giá trị đích, nghĩa là thread khác đã xử lý, bỏ qua
        if (currentValue >= targetValue) {
            return;
        }
        // Thử thao tác CAS: nếu giá trị hiện tại bằng targetValue - 1 thì đặt thành targetValue theo cách atomic
        if (currentValue == targetValue - 1) {
          if (unsafe.compareAndSwapInt(this, fieldOffset, currentValue, targetValue)) {
              // In ngay sau khi CAS thành công, đảm bảo in đúng giá trị được đặt lần này
              System.out.print(targetValue + " ");
              return;
          }
        }
        // CAS thất bại, đọc lại và thử lại
    }
}
```

Trong ví dụ trên, ta tạo hai thread cùng thử sửa shared variable `a`. Khi mỗi thread gọi method `incrementAndPrint(targetValue)`:

1. Trước hết đọc giá trị hiện tại `currentValue` của `a`.
2. Kiểm tra xem `currentValue` có bằng `targetValue - 1` (giá trị ngay trước đó theo kỳ vọng) hay không.
3. Nếu điều kiện thỏa mãn, gọi `unsafe.compareAndSwapInt()` để thử cập nhật `a` từ `currentValue` thành `targetValue`.
4. Nếu thao tác CAS thành công (trả về true), in `targetValue` rồi thoát loop.
5. Nếu thao tác CAS thất bại, nghĩa là có thread khác đang cạnh tranh đồng thời. Khi đó `currentValue` sẽ được đọc lại và thử lại cho đến khi thành công.

Cơ chế này đảm bảo mỗi số (từ 1 đến 9) chỉ được set thành công và in một lần, đồng thời thực hiện theo thứ tự.

![](https://oss.javaguide.cn/github/javaguide/java/basis/unsafe/image-20220717144939826.png)

Cần lưu ý:

1. **Logic spin:** Bản thân method `compareAndSwapInt` chỉ thực hiện một lần so sánh và trao đổi rồi lập tức trả về kết quả. Vì vậy, để đảm bảo thao tác cuối cùng thành công (khi giá trị phù hợp với kỳ vọng), cần triển khai logic spin tường minh trong code (chẳng hạn loop `while(true)`), liên tục thử cho đến khi thao tác CAS thành công.
2. **Triển khai của `AtomicInteger`:** Bên trong JDK, class `java.util.concurrent.atomic.AtomicInteger` chính là sử dụng thao tác CAS và logic spin tương tự để triển khai tính atomic cho các method như `getAndIncrement()`, `compareAndSet()`. Thông thường, dùng trực tiếp `AtomicInteger` an toàn hơn và được khuyến nghị hơn vì nó đóng gói sự phức tạp ở tầng dưới.
3. **Vấn đề ABA:** Bản thân thao tác CAS tồn tại vấn đề ABA (một giá trị đổi từ A thành B rồi quay lại A; khi CAS kiểm tra, nó sẽ cho rằng giá trị chưa từng thay đổi). Trong một số trường hợp, nếu lịch sử thay đổi của giá trị quan trọng, có thể cần dùng `AtomicStampedReference` để giải quyết. Tuy nhiên, trong tình huống tăng đơn giản của ví dụ này, vấn đề ABA thường không gây ảnh hưởng.
4. **Mức tiêu thụ CPU:** Spin trong thời gian dài sẽ tiêu thụ tài nguyên CPU. Khi cạnh tranh gay gắt hoặc điều kiện không thỏa mãn trong thời gian dài, có thể cân nhắc thêm backoff strategy phức tạp hơn (chẳng hạn `Thread.sleep()` hoặc `LockSupport.parkNanos()`) để tối ưu.

### Điều phối thread

#### Giới thiệu

Hiện tại, các method chính trong `Unsafe` liên quan trực tiếp đến điều phối thread là `park` và `unpark`. Các method `monitorEnter`, `monitorExit`, `tryMonitorEnter` trong lịch sử đã bị xóa từ JDK 9.

```java
// Hủy block thread
public native void unpark(Object thread);
// Block thread
public native void park(boolean isAbsolute, long time);
```

Các method `park`, `unpark` có thể thực hiện suspend và resume thread. Suspend một thread được thực hiện thông qua method `park`; sau khi gọi method `park`, thread sẽ bị block cho đến khi timeout hoặc xuất hiện điều kiện như interrupt. `unpark` có thể kết thúc trạng thái suspend của một thread và khôi phục thread về trạng thái bình thường.

Ba method liên quan đến `monitor` chỉ phù hợp để giới thiệu implementation cũ; code JDK hiện tại không thể gọi chúng nữa. Khi cần object monitor, nên sử dụng statement `synchronized` của ngôn ngữ Java hoặc lock và synchronizer trong `java.util.concurrent`.

#### Ứng dụng điển hình

Class cốt lõi của Java lock và synchronizer framework là `AbstractQueuedSynchronizer` (AQS), sử dụng `LockSupport.park()` và `LockSupport.unpark()` để block và wake up thread. Còn method `park`, `unpark` của `LockSupport` thực tế được triển khai bằng cách gọi method `park`, `unpark` của `Unsafe`.

```java
public static void park(Object blocker) {
    Thread t = Thread.currentThread();
    setBlocker(t, blocker);
    UNSAFE.park(false, 0L);
    setBlocker(t, null);
}
public static void unpark(Thread thread) {
    if (thread != null)
        UNSAFE.unpark(thread);
}
```

Method `park` của `LockSupport` sẽ gọi method `park` của `Unsafe` ở tầng dưới. `park` có thể return vì có permit khả dụng, thread khác gọi `unpark`, thread bị interrupt hoặc return không vì lý do cụ thể; biến thể có timeout cũng return sau khi timeout. Vì vậy, logic block phụ thuộc điều kiện cần kiểm tra lại điều kiện trong loop. Ví dụ dưới đây minh họa trường hợp thread khác gọi `unpark`:

```java
public static void main(String[] args) {
    Thread mainThread = Thread.currentThread();
    new Thread(()->{
        try {
            TimeUnit.SECONDS.sleep(5);
            System.out.println("subThread try to unpark mainThread");
            unsafe.unpark(mainThread);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }).start();

    System.out.println("park main mainThread");
    unsafe.park(false,0L);
    System.out.println("unpark mainThread success");
}
```

Output của chương trình:

```plain
park main mainThread
subThread try to unpark mainThread
unpark mainThread success
```

Flow chạy của chương trình khá dễ hiểu. Sau khi sub thread bắt đầu chạy, nó sleep trước để đảm bảo main thread có thể gọi method `park` và tự block. Sau 5 giây sleep, sub thread gọi method `unpark` để wake up main thread, khiến main thread tiếp tục thực thi các câu lệnh phía sau. Toàn bộ flow như hình dưới đây:

![](https://oss.javaguide.cn/github/javaguide/java/basis/unsafe/image-20220717144950116.png)

### Thao tác Class

#### Giới thiệu

Các thao tác liên quan đến `Class` của `Unsafe` chủ yếu gồm thao tác load class và thao tác static variable.

**Các method liên quan đến đọc static field**

> Ghi chú version: `shouldBeInitialized` và `ensureClassInitialized` đã bị xóa khỏi `sun.misc.Unsafe` trong JDK 22; API thay thế tiêu chuẩn là `MethodHandles.Lookup.ensureInitialized`, được giới thiệu từ JDK 15. Code liên quan bên dưới chỉ áp dụng cho JDK version cũ hơn.

```java
// Lấy offset của static field
public native long staticFieldOffset(Field f);
// Lấy object pointer của static field
public native Object staticFieldBase(Field f);
// Xác định class có cần khởi tạo hay không (dùng để kiểm tra trước khi lấy static field của class)
public native boolean shouldBeInitialized(Class<?> c);
```

Tạo một class chứa static field để kiểm thử:

```java
@Data
public class User {
    public static String name="Hydra";
    int age;
}
private void staticTest() throws Exception {
    User user=new User();
    // Cũng có thể dùng các câu lệnh dưới đây để trigger class initialization
    // 1.
    // unsafe.ensureClassInitialized(User.class);
    // 2.
    // System.out.println(User.name);
    System.out.println(unsafe.shouldBeInitialized(User.class));
    Field sexField = User.class.getDeclaredField("name");
    long fieldOffset = unsafe.staticFieldOffset(sexField);
    Object fieldBase = unsafe.staticFieldBase(sexField);
    Object object = unsafe.getObject(fieldBase, fieldOffset);
    System.out.println(object);
}
```

Kết quả chạy:

```plain
false
Hydra
```

Trong thao tác object của `Unsafe`, ta đã học cách dùng method `objectFieldOffset` để lấy offset của thuộc tính object và dựa trên đó đọc/ghi giá trị variable. Tuy nhiên, method này không áp dụng cho static field trong class. Khi đó cần dùng method `staticFieldOffset`. Trong code trên, chỉ khi lấy object `Field` mới cần đến `Class`; khi lấy thuộc tính static variable thì không còn phụ thuộc vào `Class` nữa.

Trong code trên, trước hết tạo một object `User`. Đó là vì nếu class chưa được khởi tạo thì static field của nó cũng chưa được khởi tạo, field lấy được cuối cùng sẽ là `null`. Vì vậy, trước khi lấy static field, cần gọi method `shouldBeInitialized` để xác định class có cần được khởi tạo trước khi lấy hay không. Nếu xóa câu lệnh tạo object `User`, kết quả chạy sẽ đổi thành:

```plain
true
null
```

**Method `defineClass` cho phép chương trình tạo động một class trong runtime**

> Ghi chú version: `sun.misc.Unsafe.defineClass` đã bị xóa từ JDK 11. Từ JDK 9 trở đi, có thể dùng `MethodHandles.Lookup.defineClass` tùy theo yêu cầu kiểm soát quyền truy cập.

```java
public native Class<?> defineClass(String name, byte[] b, int off, int len, ClassLoader loader,ProtectionDomain protectionDomain);
```

Trong quá trình sử dụng thực tế, chỉ cần truyền vào byte array, index của byte bắt đầu và độ dài byte cần đọc. Theo mặc định, class loader (`ClassLoader`) và protection domain (`ProtectionDomain`) lấy từ instance gọi method này. Ví dụ dưới đây triển khai chức năng decompile file class sau khi sinh:

```java
private static void defineTest() {
    String fileName="F:\\workspace\\unsafe-test\\target\\classes\\com\\cn\\model\\User.class";
    File file = new File(fileName);
    try(FileInputStream fis = new FileInputStream(file)) {
        byte[] content=new byte[(int)file.length()];
        fis.read(content);
        Class clazz = unsafe.defineClass(null, content, 0, content.length, null, null);
        Object o = clazz.getDeclaredConstructor().newInstance();
        Object age = clazz.getMethod("getAge").invoke(o, null);
        System.out.println(age);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

Trong đoạn code lịch sử trên, trước hết đọc một file `class` rồi chuyển nó thành byte array thông qua file stream, sau đó dùng `defineClass` để tạo động class và khởi tạo object. Class được định nghĩa theo cách này vẫn phải trải qua kiểm tra format file class, bytecode verification và các ràng buộc load tương ứng của JVM, không bỏ qua toàn bộ kiểm tra an toàn.

![](https://oss.javaguide.cn/github/javaguide/java/basis/unsafe/image-20220717145000710.png)

Các version Unsafe cũ cũng từng cung cấp method `defineAnonymousClass`:

```java
public native Class<?> defineAnonymousClass(Class<?> hostClass, byte[] data, Object[] cpPatches);
```

Method này có thể dùng để tạo động anonymous class, nhưng đã bị xóa từ JDK 17. `MethodHandles.Lookup.defineHiddenClass`, được giới thiệu trong JDK 15, là API thay thế được hỗ trợ. Implementation cụ thể của lambda thuộc về chi tiết triển khai của JDK; trong version hiện tại không thể còn mô tả rằng nó phụ thuộc vào `Unsafe.defineAnonymousClass` đã bị xóa.

#### Ứng dụng điển hình

Các version JDK trong lịch sử từng dùng `Unsafe.defineAnonymousClass` để hỗ trợ một phần implementation của dynamic language; JDK hiện tại dùng cơ chế hidden class và các cơ chế khác, không còn cung cấp method Unsafe này.

### Thông tin hệ thống

#### Giới thiệu

Phần này gồm hai method lấy thông tin liên quan đến hệ thống.

```java
// Trả về kích thước system pointer. Giá trị trả về là 4 (system 32-bit) hoặc 8 (system 64-bit).
public native int addressSize();
// Kích thước memory page, giá trị này là lũy thừa của 2.
public native int pageSize();
```

#### Ứng dụng điển hình

Hai method này có ít trường hợp sử dụng. Trong class `java.nio.Bits`, khi dùng `pageCount` để tính số lượng memory page cần thiết, method `pageSize` được gọi để lấy kích thước memory page. Ngoài ra, khi dùng method `copySwapMemory` để sao chép memory, method `addressSize` được gọi để kiểm tra trường hợp system 32-bit.

## Tổng kết

Trong bài viết này, chúng ta đã giới thiệu khái niệm cơ bản, nguyên lý hoạt động và một phần API lịch sử của `Unsafe`. Cần lưu ý rằng `sun.misc.Unsafe` là internal API không được hỗ trợ, nhiều method đã bị xóa trong các version JDK khác nhau. JDK 23 đã đánh dấu các method truy cập memory của nó là sẽ bị xóa, từ JDK 24 sẽ mặc định đưa ra runtime warning ở lần gọi đầu tiên. Code mới nên ưu tiên API tiêu chuẩn: dùng `VarHandle` để truy cập field và array trong heap, dùng Foreign Function and Memory API (`MemorySegment` và các API khác) để truy cập memory off-heap, dùng `java.util.concurrent` để synchronization giữa các thread.

<!-- @include: @article-footer.snippet.md -->
