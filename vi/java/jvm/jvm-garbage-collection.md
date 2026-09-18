---
title: "Giải thích chi tiết về JVM garbage collection (trọng điểm)"
description: "Giải thích chi tiết về JVM garbage collection: trình bày toàn diện các GC algorithm (mark-sweep, copying, mark-compact), cơ chế generational collection, các garbage collector thường dùng (Serial, Parallel, CMS, G1, ZGC, Shenandoah) và thực tiễn GC tuning."
category: Java
tag:
  - JVM
head:
  - - meta
    - name: keywords
      content: JVM garbage collection,GC algorithm,garbage collector,generational collection,mark-sweep,copying algorithm,G1 GC,ZGC,Shenandoah,GC tuning
---

> Nếu không có chú thích đặc biệt, nội dung đều nói về HotSpot VM.
>
> Bài viết này được tổng hợp và bổ sung dựa trên cuốn _Hiểu sâu JVM: Tính năng nâng cao và thực tiễn tốt nhất của JVM_.
>
> Các câu hỏi phỏng vấn thường gặp:
>
> - Làm thế nào xác định một object đã chết (hai cách).
> - Giới thiệu ngắn gọn về strong reference, soft reference, weak reference, phantom reference (sự khác nhau giữa phantom reference với soft reference và weak reference, lợi ích của soft reference).
> - Làm thế nào xác định một constant là constant không còn sử dụng.
> - Làm thế nào xác định một class là class không còn sử dụng.
> - Garbage collection có những algorithm nào, đặc điểm của từng algorithm?
> - Tại sao HotSpot phải chia thành young generation và old generation?
> - Có những garbage collector thường gặp nào?
> - Giới thiệu CMS và G1 collector.
> - Minor GC và Full GC khác nhau như thế nào?

## Lời nói đầu

Khi cần điều tra các vấn đề memory overflow khác nhau, hoặc khi garbage collection trở thành bottleneck khiến hệ thống không thể đạt mức concurrency cao hơn, chúng ta cần thực hiện việc monitoring và tuning cần thiết cho những kỹ thuật “tự động hóa” này.

## Cấu trúc cơ bản của heap

Java automatic memory management chủ yếu xử lý việc thu hồi và cấp phát memory cho object. Đồng thời, chức năng cốt lõi nhất của Java automatic memory management là cấp phát và thu hồi object trong **heap**.

Java heap là khu vực chính do garbage collector quản lý, vì vậy còn được gọi là **GC heap (Garbage Collected Heap)**.

Xét từ góc độ garbage collection, vì các collector hiện nay về cơ bản đều sử dụng generational garbage collection algorithm, Java heap được chia thành một số khu vực khác nhau, để có thể chọn garbage collection algorithm phù hợp theo đặc điểm của từng khu vực.

Trong HotSpot của JDK 7 và các phiên bản cũ hơn, GC thường được giới thiệu theo ba phần dưới đây; trong đó permanent generation là implementation của method area, không thuộc Java heap:

1. Memory của young generation (Young Generation)
2. Old generation (Old Generation)
3. Permanent generation (Permanent Generation)

Eden area, hai Survivor area S0 và S1 trong hình dưới đều thuộc young generation, tầng ở giữa thuộc old generation, còn tầng dưới cùng thuộc permanent generation.

![Cấu trúc heap](https://oss.javaguide.cn/github/javaguide/java/jvm/hotspot-heap-structure.png)

**JDK 8 đã loại bỏ PermGen (permanent generation), class metadata được chuyển sang Metaspace (meta space) sử dụng native memory**.

Bạn có thể xem lại bài [Giải thích chi tiết về Java memory area](./memory-area.md) để tìm hiểu chi tiết hơn về cấu trúc heap.

## Nguyên tắc cấp phát và thu hồi memory

### Object được ưu tiên cấp phát trong Eden area

Trong phần lớn trường hợp, object được cấp phát trong Eden area của young generation. Khi Eden area không còn đủ không gian để cấp phát, VM sẽ thực hiện một lần Minor GC. Sau đây là một thử nghiệm thực tế.

Code thử nghiệm:

```java
public class GCTest {
  public static void main(String[] args) {
    byte[] allocation1, allocation2;
    allocation1 = new byte[30900*1024];
  }
}
```

Chạy theo cách sau:
![](https://oss.javaguide.cn/github/javaguide/java/jvm/25178350.png)

Parameter được thêm vào: `-XX:+PrintGCDetails`
![](https://oss.javaguide.cn/github/javaguide/java/jvm/run-with-PrintGCDetails.png)

Kết quả chạy (phần mô tả bằng chữ màu đỏ không chính xác, đáng ra phải tương ứng với permanent generation của JDK1.7):

![](https://oss.javaguide.cn/github/javaguide/java/jvm/28954286.jpg)

Từ hình trên có thể thấy memory của Eden area gần như đã được cấp phát hoàn toàn (ngay cả khi chương trình không làm gì, young generation vẫn sử dụng hơn 2000 k memory).

Nếu tiếp tục cấp phát memory cho `allocation2` thì điều gì sẽ xảy ra?

```java
allocation2 = new byte[900*1024];
```

![](https://oss.javaguide.cn/github/javaguide/java/jvm/28128785.jpg)

Khi cấp phát memory cho `allocation2`, memory của Eden area gần như đã được cấp phát hết.

Khi Eden area không còn đủ không gian để cấp phát, VM sẽ thực hiện một lần Minor GC. Trong thời gian GC, VM lại phát hiện `allocation1` không thể chứa trong Survivor space, nên buộc phải dùng **promotion guarantee mechanism** để chuyển object của young generation sang old generation trước. Không gian trong old generation đủ để chứa `allocation1`, nên sẽ không xảy ra Full GC. Sau Minor GC, nếu các object được cấp phát tiếp theo có thể nằm trong Eden area thì chúng vẫn được cấp phát memory trong Eden area. Có thể thực thi code sau để kiểm chứng:

```java
public class GCTest {

  public static void main(String[] args) {
    byte[] allocation1, allocation2,allocation3,allocation4,allocation5;
    allocation1 = new byte[32000*1024];
    allocation2 = new byte[1000*1024];
    allocation3 = new byte[1000*1024];
    allocation4 = new byte[1000*1024];
    allocation5 = new byte[1000*1024];
  }
}

```

### Large object đi thẳng vào old generation

Large object là object cần một lượng lớn không gian memory liên tục (ví dụ: string, array).

Việc large object đi thẳng vào old generation được VM quyết định động, phụ thuộc vào garbage collector và parameter liên quan đang được sử dụng. Đây là một chiến lược optimization nhằm tránh đưa large object vào young generation, từ đó giảm frequency và cost của garbage collection trong young generation.

- G1 garbage collector xem object có kích thước đạt hoặc vượt quá một nửa Region là Humongous Object, và cấp phát trực tiếp object đó vào các Humongous Region liên tục thuộc old generation. Kích thước Region có thể được thiết lập qua `-XX:G1HeapRegionSize`.
- Việc các collector khác có đưa large object trực tiếp vào old generation hay không, và trong điều kiện nào, phụ thuộc vào collector cụ thể và phiên bản JDK, không thể khái quát bằng một threshold chung.

### Object tồn tại lâu sẽ đi vào old generation

Vì VM sử dụng tư tưởng generational collection để quản lý memory, khi thu hồi memory cần nhận biết object nào nên nằm trong young generation và object nào nên nằm trong old generation. Để làm được điều này, VM cấp cho mỗi object một object age (Age) counter.

Trong phần lớn trường hợp, object trước hết được cấp phát trong Eden area. Nếu object được sinh ra trong Eden và vẫn tồn tại sau lần Minor GC đầu tiên, đồng thời có thể được Survivor chứa, object sẽ được chuyển vào Survivor space (s0 hoặc s1) và age của object được đặt thành 1 (sau khi chuyển từ Eden area -> Survivor area, age ban đầu của object là 1).

Mỗi lần vượt qua một Minor GC trong Survivor, age của object tăng thêm 1. Khi age đạt promotion threshold, object sẽ được promote vào old generation. `-XX:MaxTenuringThreshold` dùng để thiết lập threshold age tối đa để object được promote, giá trị mặc định phụ thuộc vào garbage collector. Ví dụ, trong JDK 8, giá trị mặc định của Parallel GC là 15, của CMS là 6; promotion threshold thực tế cũng có thể được JVM điều chỉnh động.

> Hiệu chỉnh ([issue552](https://github.com/Snailclimb/JavaGuide/issues/552)): “Khi HotSpot duyệt tất cả object, nó cộng dồn kích thước mà các object chiếm dụng theo age từ nhỏ đến lớn. Khi kích thước của một age nào đó sau khi cộng dồn vượt quá 50% Survivor area (giá trị mặc định là 50%, có thể thiết lập bằng `-XX:TargetSurvivorRatio=percent`, xem [issue1199](https://github.com/Snailclimb/JavaGuide/issues/1199)), nó lấy giá trị nhỏ hơn giữa age này và MaxTenuringThreshold làm promotion age threshold mới”.
>
> Trích dẫn từ tài liệu chính thức của jdk8: <https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html>.
>
> ![](https://oss.javaguide.cn/java-guide-blog/image-20210523201742303.png)

> **Code tính age động như sau:**
>
> ```c++
> uint ageTable::compute_tenuring_threshold(size_t survivor_capacity) {
> //survivor_capacity là kích thước của survivor space
> size_t desired_survivor_size = (size_t)((((double)survivor_capacity)*TargetSurvivorRatio)/100);
> size_t total = 0;
> uint age = 1;
> while (age < table_size) {
> //sizes array là kích thước object của từng age
> total += sizes[age];
> if (total > desired_survivor_size) {
> break;
> }
> age++;
> }
> uint result = age < MaxTenuringThreshold ? age : MaxTenuringThreshold;
> ...
> }
>
> ```

Bổ sung thêm ([issue672](https://github.com/Snailclimb/JavaGuide/issues/672)): **Nguồn gốc của nhận định promotion age mặc định là 15 phần lớn đến từ cuốn _Hiểu sâu JVM_.**
Nếu đọc [các VM parameter liên quan](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html) trên website Oracle, bạn sẽ thấy phần mô tả của `-XX:MaxTenuringThreshold=threshold`:

> **Sets the maximum tenuring threshold for use in adaptive GC sizing. The largest value is 15. The default value is 15 for the parallel (throughput) collector, and 6 for the CMS collector. Promotion age mặc định không phải lúc nào cũng là 15, cần phân biệt theo garbage collector; CMS là 6.**

### Các khu vực chủ yếu thực hiện GC

Ông Chu Chí Minh đã viết như sau ở trang P92 trong lần xuất bản thứ hai của cuốn _Hiểu sâu JVM_:

> ~~_“GC của old generation (Major GC/Full GC), là GC xảy ra trong old generation...”_~~

Nhận định trên đã được hiệu chỉnh trong lần xuất bản thứ ba của _Hiểu sâu JVM_. Cảm ơn câu trả lời của R:

![Câu trả lời của R](https://oss.javaguide.cn/github/javaguide/java/jvm/rf-hotspot-vm-gc.png)

**Tóm lại:**

Xét implementation của HotSpot VM, GC trong đó thực tế chỉ được phân loại chính xác thành hai loại lớn:

Partial GC:

- Young generation collection (Minor GC / Young GC): chỉ thực hiện garbage collection cho young generation;
- Old generation collection (Major GC / Old GC): chỉ thực hiện garbage collection cho old generation. Cần lưu ý rằng trong một số ngữ cảnh, Major GC cũng được dùng để chỉ whole-heap collection;
- Mixed GC: thực hiện garbage collection cho toàn bộ young generation và một phần old generation.

Full GC: thực hiện collection cho toàn bộ Java heap; việc đồng thời thu hồi class metadata trong method area hay không phụ thuộc vào collector, phiên bản JDK và điều kiện class unloading.

### Promotion guarantee của memory space

Promotion guarantee của memory space nhằm bảo đảm trước Minor GC, bản thân old generation vẫn còn không gian trống để chứa toàn bộ object của young generation.

Mô tả về promotion guarantee của memory space trong chương thứ ba của _Hiểu sâu JVM_ như sau:

> Trước JDK 6 Update 24, trước khi xảy ra Minor GC, VM phải kiểm tra xem contiguous space khả dụng lớn nhất của old generation có lớn hơn tổng memory của toàn bộ object trong young generation hay không. Nếu điều kiện này đúng, lần Minor GC đó có thể được bảo đảm là an toàn. Nếu không, VM sẽ kiểm tra giá trị của parameter `-XX:HandlePromotionFailure` có cho phép promotion guarantee failure (Handle Promotion Failure) hay không; nếu cho phép, VM tiếp tục kiểm tra xem contiguous space khả dụng lớn nhất của old generation có lớn hơn kích thước trung bình của các object từng được promote vào old generation hay không. Nếu lớn hơn, VM sẽ thử thực hiện một lần Minor GC, dù lần Minor GC này có rủi ro; nếu nhỏ hơn, hoặc `-XX: HandlePromotionFailure` không cho phép mạo hiểm, khi đó phải chuyển sang thực hiện một lần Full GC.
>
> Sau JDK 6 Update 24, quy tắc thay đổi thành: chỉ cần contiguous space của old generation lớn hơn tổng kích thước object trong young generation hoặc kích thước trung bình của các object từng được promote thì sẽ thực hiện Minor GC, nếu không sẽ thực hiện Full GC.

## Cách xác định object đã chết

Heap gần như chứa tất cả object instance. Bước đầu tiên trước khi garbage collection heap là xác định object nào đã chết (tức object không thể được sử dụng qua bất kỳ cách nào nữa).

### Reference counting algorithm

Thêm một reference counter vào object:

- Mỗi khi có một nơi reference đến object, counter tăng 1;
- Khi reference mất hiệu lực, counter giảm 1;
- Object có counter bằng 0 tại bất kỳ thời điểm nào đều không thể được sử dụng nữa.

**Cách này có implementation đơn giản, nhưng hiện nay các Java VM phổ biến không chọn algorithm này để quản lý lifecycle của object. Một trong những nguyên nhân quan trọng là algorithm không thể tự giải quyết vấn đề circular reference giữa các object.**

![Circular reference giữa các object](https://oss.javaguide.cn/github/javaguide/java/jvm/object-circular-reference.png)

Vấn đề các object reference lẫn nhau như sau: ngoài việc `objA` và `objB` reference lẫn nhau, giữa hai object này không còn reference nào khác. Tuy nhiên, vì reference lẫn nhau nên reference counter của cả hai đều khác 0. Do đó reference counting algorithm không thể thông báo cho GC collector thu hồi chúng.

```java
public class ReferenceCountingGc {
    Object instance = null;
    public static void main(String[] args) {
        ReferenceCountingGc objA = new ReferenceCountingGc();
        ReferenceCountingGc objB = new ReferenceCountingGc();
        objA.instance = objB;
        objB.instance = objA;
        objA = null;
        objB = null;
    }
}
```

### Reachability analysis algorithm

Tư tưởng cơ bản của algorithm này là lấy một loạt object được gọi là **“GC Roots”** làm điểm bắt đầu, rồi tìm kiếm xuống dưới từ các node này. Đường đi mà node đi qua được gọi là reference chain. Khi không có reference chain nào nối object với GC Roots, điều đó chứng minh object không thể sử dụng và cần được thu hồi.

Các `Object 6 ~ Object 10` trong hình dưới dù có quan hệ reference với nhau, nhưng không reachable từ GC Roots, nên là các object cần được thu hồi.

![Reachability analysis algorithm](https://oss.javaguide.cn/github/javaguide/java/jvm/jvm-gc-roots.png)

**Những object nào có thể làm GC Roots?**

- Object được reference trong VM stack (local variable table trong stack frame)
- Object được reference trong native method stack (native method)
- Object được reference bởi class static attribute trong method area
- Object được reference bởi constant trong method area
- Tất cả object đang được synchronous lock nắm giữ
- Object được reference bởi JNI (Java Native Interface)

**Object có thể được thu hồi có đồng nghĩa chắc chắn sẽ được thu hồi không?**

Đối với object đã override method `finalize()` nhưng method này chưa từng được thực thi, sau khi xác nhận object không reachable, HotSpot có thể đưa finalization reference tương ứng vào pending queue để Finalizer thread xử lý bất đồng bộ. Nếu object thiết lập lại liên kết với object trên reference chain trong method `finalize()`, nó có thể tạm thời thoát khỏi việc bị thu hồi; nếu không, garbage collection tiếp theo sẽ xác nhận lại trạng thái có thể thu hồi của object. Quy trình này thường được khái quát là “two marking”, nhưng không có nghĩa garbage collector sẽ đồng bộ chờ method `finalize()` thực thi, cũng không bảo đảm method này nhất định được gọi.

> Method `finalize` trong class `Object` từ lâu đã bị xem là một thiết kế tồi, trở thành gánh nặng của Java language và ảnh hưởng đến tính an toàn cũng như performance của GC. `Object.finalize()` bị deprecated từ JDK 9, còn JEP 421 đánh dấu cơ chế finalization là sẽ bị loại bỏ trong JDK 18. Code mới không nên phụ thuộc vào nó.
>
> Tham khảo:
>
> - [JEP 421: Deprecate Finalization for Removal](https://openjdk.java.net/jeps/421)
> - [Đã đến lúc quên method finalize](https://mp.weixin.qq.com/s/LW-paZAMD08DP_3-XCUxmg)

### Tổng hợp các loại reference

Dù xác định số lượng reference của object bằng reference counting algorithm, hay xác định reference chain của object có reachable bằng reachability analysis algorithm, việc phán đoán object còn sống đều liên quan đến “reference”.

Trước JDK1.2, định nghĩa reference trong Java khá truyền thống: nếu giá trị được lưu trong data có type reference biểu thị địa chỉ bắt đầu của một vùng memory khác, vùng memory đó được gọi là một reference.

Sau JDK 1.2, Java mở rộng khái niệm reference, chia reference thành bốn loại: strong reference, soft reference, weak reference và phantom reference (độ mạnh của reference giảm dần). Strong reference là phép gán reference thông thường xuất hiện phổ biến trong code; các class được định nghĩa tương ứng cho soft reference, weak reference và phantom reference trong JDK lần lượt là `SoftReference`, `WeakReference`, `PhantomReference`.

![Tổng hợp các loại Java reference](https://oss.javaguide.cn/github/javaguide/java/jvm/java-reference-type.png)

**1. Strong reference (StrongReference)**

Strong reference thực chất là phép gán reference phổ biến trong code, cũng là loại reference được sử dụng phổ biến nhất. Code như sau:

```java
String strongReference = new String("abc");
```

Nếu object vẫn có thể được truy cập qua strong reference, nó giống như **vật dụng thiết yếu trong cuộc sống**, garbage collector sẽ không thu hồi object. Khi không gian memory không đủ, Java VM thà ném lỗi `OutOfMemoryError` khiến chương trình kết thúc bất thường, chứ không tùy tiện thu hồi object có strong reachability để giải quyết vấn đề thiếu memory.

**2. Soft reference (SoftReference)**

Nếu object chỉ có soft reference, nó giống như **vật dụng có cũng được, không có cũng được trong cuộc sống**. Code soft reference như sau:

```java
// --- Ví dụ 1 ---
String str = new String("abc");
SoftReference<String> softReference1 = new SoftReference<>(str);
str = null; // loại bỏ strong reference

// --- Ví dụ 2 ---
SoftReference<String> softReference2 = new SoftReference<>(new String("def")); // anonymous object
```

Soft reference object có thể bị thu hồi khi memory pressure lớn, nhưng JVM không bảo đảm chỉ dọn dẹp chúng khi memory không đủ. Bảo đảm duy nhất là: trước khi ném `OutOfMemoryError`, tất cả object chỉ reachable thông qua soft reference chắc chắn sẽ được dọn dẹp. Miễn là garbage collector chưa thu hồi object, chương trình vẫn có thể sử dụng object đó. Soft reference có thể dùng để implement memory-sensitive cache.

Soft reference có thể được sử dụng cùng một reference queue (`ReferenceQueue`). Sau khi garbage collector xóa soft reference, nó sẽ đưa soft reference đã đăng ký với reference queue vào queue tương ứng cùng lúc hoặc sau đó. Việc reference được enqueue cho biết garbage collector đã phát hiện thay đổi reachability tương ứng, không dùng để chứng minh memory mà object chiếm dụng đã được giải phóng vật lý.

**3. Weak reference (WeakReference)**

Nếu object chỉ có weak reference, nó giống như **vật dụng có cũng được, không có cũng được trong cuộc sống**. Code weak reference như sau:

```java
// --- Ví dụ 1 ---
String str = new String("abc");
WeakReference<String> weakReference1 = new WeakReference<>(str);
str = null; // loại bỏ strong reference

// --- Ví dụ 2 ---
WeakReference<String> weakReference2 = new WeakReference<>(new String("abc")); // anonymous object
```

Điểm khác nhau giữa weak reference và soft reference là object chỉ có weak reference có lifecycle ngắn hơn. Khi garbage collector xác định một object chỉ weakly reachable, nó sẽ atomic clear weak reference trỏ đến object đó. Tuy nhiên, thao tác clear chỉ được thực hiện khi garbage collection xảy ra, nên không bảo đảm object sẽ được thu hồi ngay sau khi trở thành weakly reachable.

Weak reference có thể được sử dụng cùng một reference queue (`ReferenceQueue`). Sau khi garbage collector xóa weak reference, nó sẽ đưa weak reference đã đăng ký với reference queue vào queue tương ứng cùng lúc hoặc sau đó.

**4. Phantom reference (PhantomReference)**

“Phantom reference” đúng như tên gọi, chỉ tồn tại trên danh nghĩa. Khác với các loại reference khác, phantom reference không ngăn garbage collector thu hồi object mà nó trỏ đến. Code phantom reference như sau:

```java
// --- Ví dụ 1 ---
String str = new String("abc");
ReferenceQueue queue = new ReferenceQueue();
// tạo phantom reference, bắt buộc phải liên kết với một reference queue
PhantomReference phantomReference1 = new PhantomReference(str, queue);
str = null; // loại bỏ strong reference

// --- Ví dụ 2 ---
PhantomReference phantomReference2 = new PhantomReference(new String("abc"), queue); // anonymous object
```

**Phantom reference chủ yếu dùng để nhận thông báo khi reachability của object thay đổi, đồng thời phối hợp với reference queue để sắp xếp công việc dọn dẹp.**

**Một điểm khác giữa phantom reference với soft reference và weak reference là:** phantom reference thường được sử dụng cùng reference queue (`ReferenceQueue`). Sau khi garbage collector xác định object đã chuyển sang trạng thái phantom reachable, nó sẽ clear phantom reference liên quan và đưa phantom reference đã đăng ký với reference queue vào queue cùng lúc hoặc sau đó. `PhantomReference.get()` luôn trả về `null`, chương trình không thể lấy lại object thông qua phantom reference; phantom reference chủ yếu dùng để sắp xếp công việc dọn dẹp sau khi object không còn có thể được truy cập.

Cần đặc biệt lưu ý: soft reference chủ yếu dùng để implement cache nhạy với memory, nhưng không thể bảo đảm ứng dụng không xảy ra `OutOfMemoryError`; memory overflow còn có thể do resource ngoài heap cạn kiệt và các nguyên nhân khác.

### Làm thế nào xác định một constant là constant không còn sử dụng?

String constant pool chủ yếu thu hồi các constant không còn sử dụng. Vậy làm thế nào xác định một constant không còn sử dụng?

~~**Từ JDK1.7 trở đi, JVM đã chuyển runtime constant pool ra khỏi method area và mở một khu vực trong Java heap (Heap) để lưu runtime constant pool.**~~

> **🐛 Hiệu chỉnh (xem: [issue747](https://github.com/Snailclimb/JavaGuide/issues/747), [reference](https://blog.csdn.net/q5706503/article/details/84640762))**:
>
> 1. **Trước JDK1.7, logic của runtime constant pool bao gồm string constant pool được lưu trong method area; khi đó implementation method area của HotSpot VM là permanent generation.**
> 2. **Trong JDK1.7, string constant pool được chuyển từ method area vào heap. Ở đây không nói đến runtime constant pool, tức string constant pool được chuyển riêng vào heap, còn phần còn lại của runtime constant pool vẫn ở method area, cũng chính là permanent generation trong HotSpot.**
> 3. **Trong JDK1.8, HotSpot loại bỏ permanent generation và thay bằng Metaspace. Khi đó string constant pool vẫn ở heap, runtime constant pool vẫn ở method area, chỉ là implementation của method area chuyển từ permanent generation thành Metaspace.**

Giả sử trong string constant pool có string `"abc"`. Nếu hiện tại không có String object nào reference đến string constant này, điều đó có nghĩa constant `"abc"` là constant không còn sử dụng. Nếu lúc này xảy ra garbage collection và có nhu cầu, `"abc"` sẽ được hệ thống dọn khỏi constant pool.

### Làm thế nào xác định một class là class không còn sử dụng?

Method area chủ yếu thu hồi các class không còn sử dụng. Vậy làm thế nào xác định một class là class không còn sử dụng?

Điều kiện xác định một constant là “constant không còn sử dụng” tương đối đơn giản, còn điều kiện xác định một class là “class không còn sử dụng” khắt khe hơn nhiều. Class phải đồng thời thỏa mãn 3 điều kiện dưới đây mới được xem là **“class không còn sử dụng”**:

- Tất cả instance của class đó đã được thu hồi, tức Java heap không còn instance nào của class đó.
- `ClassLoader` đã load class đó cũng đã được thu hồi.
- Object `java.lang.Class` tương ứng với class đó không được reference ở bất kỳ đâu, không thể truy cập method của class qua reflection ở bất kỳ đâu.

> Cần đặc biệt cẩn thận với điều kiện thứ 3 trong JNI: global reference `jclass` được tạo qua `NewGlobalRef` cũng ngăn class bị unload. Một lỗi thường gặp là JNI layer cache reflection result: `jmethodID`/`jfieldID` chỉ được bảo đảm hợp lệ khi class vẫn ở trạng thái loaded, vì vậy chương trình chạy lâu thường tạo global reference cho `jclass` để giữ class sống. Điều này cũng có nghĩa chỉ cần global reference này chưa được release, class tương ứng sẽ không bao giờ được unload (xem [issue#2908](https://github.com/Snailclimb/JavaGuide/issues/2908)).

VM có thể thu hồi class không còn sử dụng thỏa mãn 3 điều kiện trên. Ở đây chỉ nói là “có thể”, chứ không giống object: class không được sử dụng sẽ không nhất định bị thu hồi.

## Garbage collection algorithm

### Mark-sweep algorithm

Mark-sweep (Mark-and-Sweep) algorithm gồm giai đoạn “mark (đánh dấu)” và “sweep (dọn dẹp)”: trước hết đánh dấu tất cả object không cần thu hồi, sau khi đánh dấu xong thì thu hồi thống nhất tất cả object không được đánh dấu.

Đây là collection algorithm cơ bản nhất, các algorithm về sau đều được cải tiến từ những hạn chế của algorithm này. Garbage collection algorithm này gây ra hai vấn đề rõ ràng:

1. **Vấn đề performance**: cả hai bước mark và sweep đều không có performance cao.
2. **Vấn đề space**: sau mark-sweep sẽ tạo ra một lượng lớn memory fragmentation không liên tục.

![Mark-sweep algorithm](https://oss.javaguide.cn/github/javaguide/java/jvm/mark-and-sweep-garbage-collection-algorithm.png)

Về việc cụ thể nên đánh dấu object có thể thu hồi (object không reachable) hay object không thể thu hồi (object reachable), có nhiều cách nói khác nhau; thực ra cả hai đều không sai, cá nhân tôi thiên về cách nói thứ hai hơn.

Nếu hiểu theo cách thứ nhất, toàn bộ quy trình mark-sweep đại khái như sau:

1. Khi một object được tạo, cấp cho object một mark bit, giả sử là 0 (false);
2. Trong giai đoạn mark, đặt mark bit của tất cả reachable object (hoặc object mà user có thể reference) thành 1 (true);
3. Trong giai đoạn scan, các object có mark bit bằng 0 (false) sẽ bị thu hồi.

### Copying algorithm

Để giải quyết vấn đề performance và memory fragmentation của mark-sweep algorithm, copying (Copying) collection algorithm ra đời. Algorithm chia memory thành hai phần có kích thước bằng nhau, mỗi lần chỉ sử dụng một phần. Khi memory của phần này được sử dụng hết, các object còn sống sẽ được copy sang phần còn lại, sau đó dọn dẹp toàn bộ space đã sử dụng trong một lần. Nhờ vậy, mỗi lần thu hồi memory sẽ thu hồi một nửa memory area.

![Copying algorithm](https://oss.javaguide.cn/github/javaguide/java/jvm/copying-garbage-collection-algorithm.png)

Dù đã cải tiến mark-sweep algorithm, copying algorithm vẫn có các vấn đề sau:

- **Memory khả dụng nhỏ đi**: memory khả dụng giảm còn một nửa ban đầu.
- **Không phù hợp với old generation**: nếu số lượng object còn sống tương đối lớn, performance của việc copy sẽ rất kém.

### Mark-compact algorithm

Mark-compact (Mark-and-Compact) algorithm là một mark algorithm được đề xuất dựa trên đặc điểm của old generation. Giai đoạn mark vẫn giống mark-sweep algorithm, nhưng các bước tiếp theo không trực tiếp thu hồi object có thể thu hồi, mà di chuyển tất cả object còn sống về một đầu, sau đó trực tiếp dọn dẹp memory bên ngoài ranh giới của đầu đó.

![Mark-compact algorithm](https://oss.javaguide.cn/github/javaguide/java/jvm/mark-and-compact-garbage-collection-algorithm.png)

Vì có thêm bước compact nên performance cũng không cao, phù hợp với old generation, nơi frequency garbage collection không quá cao.

### Generational collection algorithm

Generational garbage collector kinh điển chia memory thành một số phần dựa trên thời gian tồn tại khác nhau của object. Thông thường Java heap được chia thành young generation và old generation, nhờ đó có thể chọn garbage collection algorithm phù hợp theo đặc điểm của từng generation. Cần lưu ý garbage collector không phải lúc nào cũng sử dụng thiết kế generational, ví dụ ZGC thời kỳ đầu là non-generational collector.

Ví dụ, trong young generation, mỗi lần collection có rất nhiều object chết, nên có thể chọn copying algorithm; chỉ cần trả một lượng nhỏ cost để copy object là có thể hoàn thành mỗi lần garbage collection. Xác suất object trong old generation còn sống tương đối cao, đồng thời không có space bổ sung để thực hiện promotion guarantee, nên bắt buộc phải chọn mark-sweep hoặc mark-compact algorithm để garbage collection.

**Câu hỏi phỏng vấn mở rộng:** Tại sao HotSpot phải chia thành young generation và old generation?

Hãy trả lời dựa trên phần giới thiệu về generational collection algorithm ở trên.

## Garbage collector

**Nếu collection algorithm là phương pháp luận của memory reclamation, thì garbage collector là implementation cụ thể của memory reclamation.**

Dù so sánh các collector, mục đích không phải chọn ra một collector tốt nhất. Vì cho đến nay chưa có garbage collector tốt nhất, càng không có garbage collector vạn năng, **điều chúng ta có thể làm là chọn garbage collector phù hợp theo application scenario cụ thể**. Hãy thử nghĩ: nếu tồn tại một collector hoàn hảo, phù hợp với mọi nơi và mọi scenario, HotSpot VM đã không cần implement nhiều garbage collector khác nhau như vậy.

Garbage collector mặc định của Oracle/OpenJDK HotSpot trong môi trường Server VM điển hình (lựa chọn thực tế còn chịu ảnh hưởng của platform và runtime environment, có thể dùng command `java -XX:+PrintCommandLineFlags -version` để kiểm tra):

- JDK 8: Parallel Scavenge (young generation) + Parallel Old (old generation)
- JDK 9 trở đi: G1

### Serial collector

Serial (serial) collector là garbage collector cơ bản nhất và có lịch sử lâu đời nhất. Nhìn vào tên là biết collector này là single-threaded collector. Ý nghĩa của **“single-threaded”** không chỉ là nó chỉ sử dụng một garbage collection thread để hoàn thành công việc garbage collection, mà quan trọng hơn là khi thực hiện garbage collection, nó phải tạm dừng tất cả working thread khác (**"Stop The World"**) cho đến khi collection kết thúc.

**Young generation sử dụng mark-copy algorithm, old generation sử dụng mark-compact algorithm.**

![Serial collector](https://oss.javaguide.cn/github/javaguide/java/jvm/serial-garbage-collector.png)

Các nhà thiết kế VM dĩ nhiên biết trải nghiệm người dùng kém do Stop The World gây ra, nên trong thiết kế các garbage collector tiếp theo, pause time không ngừng được rút ngắn (vẫn còn pause; quá trình tìm kiếm garbage collector ưu việt nhất vẫn đang tiếp tục).

Nhưng Serial collector có ưu điểm hơn các garbage collector khác không? Tất nhiên là có, đó là **đơn giản và hiệu quả (so với single-thread của các collector khác)**. Vì không có overhead tương tác giữa các thread, Serial collector tự nhiên có thể đạt single-thread collection efficiency rất cao. Serial collector là lựa chọn khá tốt cho VM chạy ở Client mode.

### ParNew collector

ParNew collector thực chất là phiên bản multi-thread của Serial collector. Ngoài việc sử dụng multi-thread để thực hiện garbage collection, các hành vi còn lại (control parameter, collection algorithm, reclamation strategy, v.v.) hoàn toàn giống Serial collector.

ParNew chỉ phụ trách young generation và sử dụng mark-copy algorithm; nó thường phối hợp với CMS collector phụ trách old generation.

![ParNew collector ](https://oss.javaguide.cn/github/javaguide/java/jvm/parnew-garbage-collector.png)

Trong các phiên bản JDK vẫn hỗ trợ CMS, ParNew là partner ở young generation của CMS; CMS đã bị loại bỏ trong JDK 14, vì vậy cặp collector này chỉ áp dụng cho HotSpot phiên bản cũ.

**Bổ sung về khái niệm parallel và concurrent:**

- **Parallel**: nhiều garbage collection thread làm việc song song, nhưng user thread vẫn ở trạng thái chờ.

- **Concurrent**: user thread và garbage collection thread thực thi đồng thời (nhưng không nhất thiết parallel, có thể thực thi luân phiên), user program tiếp tục chạy, còn garbage collector chạy trên một CPU khác.

### Parallel Scavenge collector

Parallel Scavenge collector cũng là multi-thread collector sử dụng mark-copy algorithm, nhìn gần như giống ParNew. **Vậy điểm đặc biệt của nó là gì?**

```bash
-XX:+UseParallelGC

    Sử dụng Parallel collector (mặc định kết hợp với Parallel Old trong JDK 8)

-XX:+UseParallelOldGC

    Sử dụng Parallel collector + parallel ở old generation
```

Parallel Scavenge collector tập trung vào throughput (sử dụng CPU hiệu quả). Garbage collector như CMS tập trung nhiều hơn vào pause time của user thread (cải thiện user experience). Throughput là tỷ lệ giữa thời gian CPU dùng để chạy user code và tổng thời gian CPU tiêu thụ. Parallel Scavenge collector cung cấp nhiều parameter để user tìm pause time phù hợp nhất hoặc throughput tối đa. Nếu chưa hiểu rõ về hoạt động của collector và gặp khó khăn khi optimization thủ công, sử dụng Parallel Scavenge collector kết hợp adaptive tuning strategy, giao việc optimization memory management cho VM hoàn thành cũng là một lựa chọn tốt.

**Young generation sử dụng mark-copy algorithm, old generation sử dụng mark-compact algorithm.**

![Sơ đồ hoạt động của Parallel Old collector](https://oss.javaguide.cn/github/javaguide/java/jvm/parallel-scavenge-garbage-collector.png)

**Đây là collector mặc định của JDK1.8.**

Dùng command `java -XX:+PrintCommandLineFlags -version` để kiểm tra.

```bash
-XX:InitialHeapSize=262921408 -XX:MaxHeapSize=4206742528 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseParallelGC
java version "1.8.0_211"
Java(TM) SE Runtime Environment (build 1.8.0_211-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.211-b12, mixed mode)
```

JDK1.8 mặc định sử dụng Parallel Scavenge + Parallel Old. Nếu chỉ định parameter `-XX:+UseParallelGC`, mặc định cũng chỉ định `-XX:+UseParallelOldGC`; có thể dùng `-XX:-UseParallelOldGC` để disable chức năng này.

### Serial Old collector

**Phiên bản old generation của Serial collector**, cũng là một single-threaded collector. Nó chủ yếu có hai công dụng: một là kết hợp với Parallel Scavenge collector trong JDK1.5 và các phiên bản trước đó; hai là làm phương án dự phòng cho CMS collector.

![Serial collector](https://oss.javaguide.cn/github/javaguide/java/jvm/serial-garbage-collector.png)

### Parallel Old collector

**Phiên bản old generation của Parallel Scavenge collector.** Collector này sử dụng multi-thread và mark-compact algorithm. Trong các trường hợp chú trọng throughput và resource CPU, có thể ưu tiên cân nhắc Parallel Scavenge collector và Parallel Old collector.

![Sơ đồ hoạt động của Parallel Old collector](https://oss.javaguide.cn/github/javaguide/java/jvm/parallel-scavenge-garbage-collector.png)

### CMS collector

**CMS (Concurrent Mark Sweep) collector là collector hướng đến mục tiêu đạt pause time ngắn nhất khi reclamation. Nó rất phù hợp với các application chú trọng user experience.**

**CMS (Concurrent Mark Sweep) collector là concurrent collector đầu tiên của HotSpot VM theo đúng nghĩa, lần đầu tiên thực hiện việc cho garbage collection thread và user thread làm việc (về cơ bản) đồng thời.**

Có thể thấy từ hai từ **Mark Sweep** trong tên, CMS collector là implementation của **mark-sweep algorithm**. Quy trình hoạt động của nó phức tạp hơn các garbage collector trước đó, gồm bốn bước:

- **Initial mark:** pause ngắn, đánh dấu object (root object) reference trực tiếp với root;
- **Concurrent mark:** đồng thời bật GC và user thread, sử dụng một closure structure để ghi nhận reachable object. Tuy nhiên khi giai đoạn này kết thúc, closure structure không thể bảo đảm chứa toàn bộ reachable object hiện tại. Vì user thread có thể liên tục cập nhật reference field, GC thread không thể bảo đảm tính realtime của reachability analysis. Do đó algorithm này sẽ theo dõi và ghi nhận các nơi xảy ra reference update.
- **Remark:** giai đoạn remark nhằm hiệu chỉnh record đánh dấu của những object có mark thay đổi trong concurrent mark do user program vẫn tiếp tục chạy. Pause time của giai đoạn này thường dài hơn initial mark một chút, nhưng ngắn hơn rất nhiều so với concurrent mark.
- **Concurrent sweep:** bật user thread, đồng thời GC thread bắt đầu sweep các khu vực chưa được mark.

![CMS collector](https://oss.javaguide.cn/github/javaguide/java/jvm/cms-garbage-collector.png)

Chỉ nhìn vào tên cũng có thể thấy đây là một garbage collector ưu tú. Ưu điểm chính: **concurrent collection, low pause**. Tuy nhiên nó có ba nhược điểm rõ ràng:

- **Nhạy với resource CPU;**
- **Không thể xử lý floating garbage;**
- **Collection algorithm mà nó sử dụng, tức mark-sweep algorithm, sẽ tạo ra lượng lớn space fragmentation khi collection kết thúc.**

**CMS garbage collector đã bị đánh dấu deprecated trong Java 9 và bị loại bỏ trong Java 14.**

### G1 collector

**G1 (Garbage-First) là garbage collector hướng đến server, chủ yếu dành cho máy có nhiều processor và memory dung lượng lớn. Nó có đặc điểm throughput cao, đồng thời có xác suất rất cao đáp ứng yêu cầu về GC pause time.**

G1 được xem là một đặc điểm tiến hóa quan trọng của HotSpot VM trong JDK1.7. Nó có các đặc điểm sau:

- **Parallel và concurrent**: G1 tận dụng đầy đủ ưu thế phần cứng trong môi trường CPU, multi-core, sử dụng nhiều CPU (CPU hoặc CPU core) để rút ngắn Stop-The-World pause time. Một số GC action mà các collector khác vốn cần pause Java thread để thực hiện, G1 collector vẫn có thể cho Java program tiếp tục chạy bằng cách thực hiện concurrent.
- **Generational collection**: dù G1 có thể tự quản lý toàn bộ GC heap mà không cần phối hợp với collector khác, nó vẫn giữ lại khái niệm generational.
- **Space integration**: khác với mark-sweep algorithm của CMS, xét tổng thể G1 là collector dựa trên mark-compact algorithm; xét cục bộ thì dựa trên mark-copy algorithm.
- **Pause có thể dự đoán**: đây là một ưu điểm lớn khác của G1 so với CMS. Giảm pause time là điểm cùng quan tâm của G1 và CMS. G1 xây dựng prediction model dựa trên pause time target do user thiết lập và chọn collection set, nhưng target này là soft target, không bảo đảm mỗi pause đều không vượt quá giá trị chỉ định.

Quy trình hoạt động của G1 collector đại khái gồm các bước sau:

- **Initial mark**: pause ngắn (Stop-The-World, STW), đánh dấu object có thể được reference trực tiếp từ GC Roots, tức đánh dấu tất cả active object reachable trực tiếp.
- **Concurrent mark**: chạy concurrent với application, đánh dấu tất cả reachable object. Giai đoạn này có thể kéo dài, phụ thuộc vào kích thước heap và số lượng object.
- **Final mark**: pause ngắn (STW), xử lý một lượng nhỏ reference change còn lại sau khi concurrent mark kết thúc.
- **Evacuation**: dựa trên kết quả mark, chọn region có reclamation value cao, copy object còn sống sang region mới, thu hồi memory của region cũ. Giai đoạn này gồm một hoặc nhiều pause (STW), tùy vào độ phức tạp của reclamation.

![G1 collector](https://oss.javaguide.cn/github/javaguide/java/jvm/g1-garbage-collector.png)

**G1 collector duy trì một priority list ở background. Mỗi lần, dựa trên collection time được cho phép, nó ưu tiên chọn Region có reclamation value lớn nhất (đây là nguồn gốc tên Garbage-First)**. Cách chia memory space thành các Region và thu hồi region theo priority này bảo đảm G1 collector đạt collection efficiency cao nhất có thể trong thời gian giới hạn (chia nhỏ memory thành nhiều phần).

**Từ JDK9, G1 garbage collector trở thành garbage collector mặc định.**

### ZGC collector

Tương tự ParNew và G1, ZGC cũng sử dụng mark-copy algorithm, nhưng ZGC đã cải tiến đáng kể algorithm này.

ZGC có thể kiểm soát pause time trong phạm vi vài millisecond, pause time không bị ảnh hưởng bởi kích thước heap, tình trạng Stop The World xuất hiện ít hơn, nhưng phải đánh đổi một phần throughput. ZGC hỗ trợ heap tối đa 16TB.

ZGC được giới thiệu trong Java11 và ở giai đoạn thử nghiệm. Sau nhiều lần lặp qua các phiên bản, liên tục hoàn thiện và sửa lỗi, ZGC đã có thể được sử dụng chính thức trong Java15.

Tuy nhiên, garbage collector mặc định vẫn là G1. Có thể enable ZGC bằng parameter sau:

```bash
java -XX:+UseZGC className
```

Java 21 giới thiệu generational ZGC. Từ Java 23, generational mode trở thành mode mặc định của ZGC; Java 24 lại loại bỏ non-generational mode.

Có thể enable generational ZGC bằng parameter sau:

```bash
java -XX:+UseZGC className
```

Trong Java 21 và 22, có thể dùng thêm `-XX:+ZGenerational` để enable generational mode; parameter này đã deprecated trong Java 24.

Để tìm hiểu chi tiết về ZGC collector, nên xem các bài viết sau:

- [Phân tích ZGC từ góc độ các thế hệ GC algorithm - JD Technology](https://mp.weixin.qq.com/s/ExkB40cq1_Z0ooDzXn7CVw)
- [Khám phá và thực tiễn về thế hệ mới của garbage collector ZGC - Meituan Technology Team](https://tech.meituan.com/2020/08/06/new-zgc-practice-in-meituan.html)
- [Giải thích chi tiết G1&ZGC, JVM garbage collector theo hướng phỏng vấn - Alibaba Cloud Developer](https://mp.weixin.qq.com/s/Ywj3XMws0IIK-kiUllN87Q)

### Shenandoah collector

Tương tự ZGC, Shenandoah cũng là collector hướng đến low latency, do Red Hat chủ trì phát triển. Shenandoah được đưa vào dưới dạng experimental feature trong Java 12 qua [JEP 189](https://openjdk.org/jeps/189), và trở thành official feature từ Java 15 ([JEP 379](https://openjdk.org/jeps/379)).

Tư tưởng cốt lõi của Shenandoah là **concurrent compaction**: phần lớn công việc compact heap diễn ra đồng thời với application thread, pause time có thể được kiểm soát ở mức dưới millisecond và không bị ảnh hưởng bởi kích thước heap. Shenandoah chủ yếu dùng **forwarding pointer (Brooks Pointer)** và **read barrier** để việc di chuyển object trong concurrent compaction trở nên transparent với application thread.

Cần lưu ý, Shenandoah thời kỳ đầu giống ZGC thời kỳ đầu, là non-generational collector và phải trả một phần cost về throughput. Generational Shenandoah vẫn đang được phát triển (như [JEP 535](https://openjdk.org/jeps/535), [JEP 521](https://openjdk.org/jeps/521)), đáng để tiếp tục theo dõi.

Có thể enable Shenandoah bằng parameter sau:

```bash
java -XX:+UseShenandoahGC className
```

> Lưu ý: Shenandoah không nằm trong danh sách support mặc định của Oracle JDK, chủ yếu được cung cấp cùng OpenJDK (như các distribution của Red Hat, Amazon Corretto, v.v.). Trước khi sử dụng, cần xác nhận distribution JDK đang dùng có bao gồm và enable Shenandoah mặc định hay không (`-XX:+PrintFlagsFinal -version | grep UseShenandoahGC`).

## Tham khảo

- _Hiểu sâu JVM: Tính năng nâng cao và thực tiễn tốt nhất của JVM (lần xuất bản thứ hai)_
- The Java® Virtual Machine Specification - Java SE 8 Edition: <https://docs.oracle.com/javase/specs/jvms/se8/html/index.html>

<!-- @include: @article-footer.snippet.md -->
