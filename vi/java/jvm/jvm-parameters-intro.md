---
title: "Tổng hợp các tham số JVM quan trọng nhất"
description: "Tổng hợp các tham số JVM thường dùng và cách cấu hình, kết hợp các đề xuất thực tiễn về tối ưu memory và GC."
category: Java
tag:
  - JVM
head:
  - - meta
    - name: keywords
      content: tham số JVM,kích thước heap,kích thước stack,cấu hình GC,tối ưu performance,tham số XX
---

> Bài viết này được JavaGuide dịch từ [https://www.baeldung.com/jvm-parameters](https://www.baeldung.com/jvm-parameters), đồng thời bổ sung và hoàn thiện nhiều nội dung.
> Tài liệu về tham số [https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html)
>
> Phiên bản JDK: chủ yếu là 1.8, đồng thời bổ sung các tham số thường dùng ở các phiên bản mới

Trong bài viết này, bạn sẽ nắm được một số cấu hình tham số thường dùng nhất trong Java Virtual Machine (JVM), từ đó hiểu rõ hơn và tối ưu môi trường chạy của ứng dụng Java.

## Các tham số liên quan đến heap memory

> Java Heap là vùng lớn nhất trong memory do JVM quản lý, **được mọi thread dùng chung** và được tạo khi virtual machine khởi động. **Mục đích duy nhất của vùng memory này là lưu trữ các object instance; gần như mọi object instance và array đều được cấp phát memory trên heap.**

![Các tham số cấu hình thường dùng của vùng memory](./pictures/内存区域常见配置参数.png)

### Thiết lập kích thước heap memory (-Xms và -Xmx)

Dựa trên nhu cầu thực tế của ứng dụng để thiết lập kích thước heap memory ban đầu và tối đa là một trong những thực tiễn thường gặp nhất khi tối ưu performance. **Khuyến nghị thiết lập tường minh hai tham số này và thường nên đặt chúng cùng một giá trị** để tránh chi phí performance do heap memory tự điều chỉnh trong lúc chạy.

Sử dụng các tham số sau để thiết lập:

```bash
-Xms<heap size>[unit]  # Thiết lập kích thước heap ban đầu của JVM
-Xmx<heap size>[unit]  # Thiết lập kích thước heap tối đa của JVM
```

- `<heap size>`: Chỉ định giá trị cụ thể của memory.
- `[unit]`: Chỉ định đơn vị của memory, chẳng hạn g (GB), m (MB), k (KB).

**Ví dụ:** Đặt heap ban đầu và heap tối đa của JVM đều là 4GB:

```bash
-Xms4G -Xmx4G
```

### Thiết lập kích thước memory của Young Generation

Theo [tài liệu chính thức của Oracle](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/sizing.html), sau khi hoàn tất cấu hình tổng memory khả dụng của heap, yếu tố ảnh hưởng lớn thứ hai là tỷ lệ `Young Generation` chiếm trong heap memory. Kích thước mặc định của Young Generation chịu ảnh hưởng của implementation JVM, platform, garbage collector, kích thước heap và cơ chế tự thích ứng; không tồn tại một “mức tối thiểu cố định 1310 MB” phù hợp với mọi môi trường. `NewSize` và `MaxNewSize` lần lượt dùng để giới hạn cận dưới và cận trên của kích thước Young Generation.

Với các generational collector hỗ trợ tham số kích thước Young Generation cố định, có thể thiết lập kích thước memory của Young Generation theo hai cách sau:

**1. Chỉ định bằng `-XX:NewSize` và `-XX:MaxNewSize`**

```bash
-XX:NewSize=<young size>[unit]    # Thiết lập kích thước ban đầu của Young Generation
-XX:MaxNewSize=<young size>[unit] # Thiết lập kích thước tối đa của Young Generation
```

**Ví dụ:** Thiết lập Young Generation tối thiểu 512MB, tối đa 1024MB:

```bash
-XX:NewSize=512m -XX:MaxNewSize=1024m
```

**2. Chỉ định bằng `-Xmn<young size>[unit]`**

**Ví dụ:** Cố định kích thước Young Generation là 512MB:

```bash
-Xmn512m
```

Việc thiết lập tường minh kích thước Young Generation sẽ hạn chế khả năng tự điều chỉnh của garbage collector. [Tài liệu chính thức của Oracle](https://docs.oracle.com/en/java/javase/25/docs/specs/man/java.html#extra-options-for-java) nêu rõ rằng không nên đặt `-Xmn` cho G1; có cần điều chỉnh hay không và điều chỉnh thế nào nên được quyết định dựa trên collector cụ thể và GC log.

Một kinh nghiệm quan trọng trong chiến lược tối ưu GC là:

> Cố gắng để các object mới tạo được cấp phát memory trong Young Generation và được thu hồi, vì chi phí của Minor GC thường thấp hơn nhiều so với Full GC. Phân tích GC log để đánh giá việc phân bổ không gian Young Generation có hợp lý hay không. Nếu nhiều object mới sớm được đưa vào Old Generation (Promotion), có thể điều chỉnh kích thước Young Generation bằng `-Xmn` hoặc `-XX:NewSize/-XX:MaxNewSize` một cách phù hợp, nhằm giảm tối đa tình trạng object đi thẳng vào Old Generation.

Ngoài ra, bạn có thể sử dụng tham số **`-XX:NewRatio=<int>`** để thiết lập **tỷ lệ kích thước memory giữa Old Generation và toàn bộ Young Generation (bao gồm Eden và hai vùng Survivor)**.

Ví dụ, `-XX:NewRatio=2` biểu thị Old Generation : Young Generation = 2 : 1, tức Young Generation chiếm 1/3 tổng kích thước heap. Giá trị mặc định phụ thuộc vào JVM, collector và platform.

```bash
-XX:NewRatio=2
```

### Thiết lập kích thước PermGen/Metaspace

**Từ Java 8, class metadata được chuyển sang native memory. Nếu không thiết lập giới hạn bằng `-XX:MaxMetaspaceSize`, class metadata tăng liên tục có thể tiêu thụ lượng lớn native memory; PermGen bị giới hạn bởi `-XX:MaxPermSize`.**

Trước JDK 1.8, khi PermGen chưa bị loại bỏ hoàn toàn, thường dùng các tham số sau để điều chỉnh kích thước method area:

```bash
-XX:PermSize=N # Kích thước ban đầu của method area (PermGen)
-XX:MaxPermSize=N # Kích thước tối đa của method area (PermGen), vượt quá giá trị này sẽ ném exception OutOfMemoryError:java.lang.OutOfMemoryError: PermGen
```

Tương đối mà nói, garbage collection trong vùng này ít xảy ra hơn, nhưng điều đó không có nghĩa là dữ liệu đã vào method area sẽ “tồn tại vĩnh viễn”.

**Trong JDK 1.8, method area (PermGen của HotSpot) đã bị loại bỏ hoàn toàn (quá trình này đã bắt đầu từ JDK 1.7), thay vào đó là Metaspace; Metaspace sử dụng native memory.**

Dưới đây là một số tham số thường dùng:

```bash
-XX:MetaspaceSize=N # Thiết lập kích thước ban đầu của Metaspace (đây là một hiểu lầm thường gặp, sẽ giải thích ở phần sau)
-XX:MaxMetaspaceSize=N # Thiết lập kích thước tối đa của Metaspace
```

**🐛 Đính chính (tham khảo [issue#1947](https://github.com/Snailclimb/JavaGuide/issues/1947)):**

**1. `-XX:MetaspaceSize` không phải là capacity ban đầu:** Capacity ban đầu của Metaspace không được thiết lập bởi `-XX:MetaspaceSize`. Bất kể cấu hình `-XX:MetaspaceSize` là bao nhiêu, với JVM 64-bit, capacity ban đầu của Metaspace thường là một giá trị nhỏ cố định (tài liệu Oracle đề cập khoảng 12MB đến 20MB, quan sát thực tế khoảng 20.8MB).

Có thể tham khảo nội dung được đề cập trong [Các lưu ý khác](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/considerations.html) của tài liệu chính thức Oracle:

> Chỉ định giá trị cao hơn cho tùy chọn MetaspaceSize để tránh garbage collection sớm do class metadata gây ra. Lượng class metadata được cấp phát cho một ứng dụng phụ thuộc vào ứng dụng, và không có hướng dẫn chung để lựa chọn MetaspaceSize. Kích thước mặc định của MetaspaceSize phụ thuộc vào platform, dao động từ 12 MB đến khoảng 20 MB.
>
> Kích thước mặc định của MetaspaceSize phụ thuộc vào platform, dao động từ 12 MB đến khoảng 20 MB.

Ngoài ra, bạn có thể xem thử nghiệm [Hiểu lầm về tham số JVM MetaspaceSize](https://mp.weixin.qq.com/s/jqfppqqd98DfAJHZhFbmxA).

**2. Mở rộng và GC metadata:** Khi phần memory đã commit của Metaspace đạt ngưỡng high-water tương ứng với `-XX:MetaspaceSize`, JVM sẽ trigger một lần garbage collection để thử unload class và giải phóng class metadata. Sau đó, JVM sẽ động điều chỉnh tăng hoặc giảm ngưỡng trigger GC tiếp theo dựa trên hiệu quả giải phóng; chu kỳ collection cụ thể nào được sử dụng phụ thuộc vào garbage collector và phiên bản JDK, không thể gọi chung là Full GC. Bên trong garbage collector, biến `_capacity_until_GC` được dùng để xác định vùng Metaspace đã đạt ngưỡng hay chưa, với code khởi tạo như sau:

```c
void MetaspaceGC::initialize() {
  // Đặt high-water mark thành MaxMetapaceSize trong quá trình khởi tạo VM vì
  // không thể thực hiện GC trong quá trình khởi tạo.
  _capacity_until_GC = MaxMetaspaceSize;
}
```

**3. Tác dụng của `-XX:MaxMetaspaceSize`:** Nếu không thiết lập tường minh `-XX:MaxMetaspaceSize`, Metaspace mặc định không có giới hạn cố định; class metadata tăng liên tục có thể tiêu thụ lượng lớn native memory. Có thiết lập tham số này hay không và đặt bao nhiêu cần được quyết định dựa trên tình hình sử dụng class metadata và ngân sách native memory của process. Giới hạn quá nhỏ sẽ làm tăng tần suất GC metadata, đồng thời có thể sớm trigger `OutOfMemoryError: Metaspace`; vì vậy không tồn tại một giá trị khuyến nghị phù hợp với mọi ứng dụng.

Đọc thêm: [Đính chính issue: Nếu không chỉ định kích thước MaxMetaspaceSize thì sẽ không làm cạn kiệt memory #1204](https://github.com/Snailclimb/JavaGuide/issues/1204).

## Các tham số liên quan đến garbage collection

### Chọn garbage collector

Việc chọn garbage collector (Garbage Collector, GC) phù hợp rất quan trọng đối với throughput và response latency của ứng dụng. Để tìm hiểu chi tiết về thuật toán và collector garbage collection, có thể xem bài viết [Giải thích chi tiết về garbage collection của JVM (trọng tâm)](https://javaguide.cn/java/jvm/jvm-garbage-collection.html) do tác giả biên soạn.

JVM cung cấp nhiều implementation GC, phù hợp với các trường hợp khác nhau:

- **Serial GC (serial garbage collector):** Thực thi GC bằng một thread, phù hợp với client mode hoặc môi trường CPU đơn core. Tham số: `-XX:+UseSerialGC`.
- **Parallel GC (parallel garbage collector):** Dùng nhiều thread để thực hiện garbage collection ở Young Generation và Old Generation, tập trung vào throughput; là GC mặc định của Server VM trong JDK 8. Khi bật `-XX:+UseParallelGC` trong JDK 8, mặc định sẽ đi kèm Parallel Old. Tham số: `-XX:+UseParallelGC`.
- **CMS GC (Concurrent Mark Sweep collector):** Nhắm đến thời gian pause collection ngắn nhất, phần lớn giai đoạn GC có thể thực thi đồng thời với user thread. Phù hợp với ứng dụng yêu cầu cao về response time. Bị đánh dấu deprecated trong JDK 9 và bị loại bỏ trong JDK 14. Tham số: `-XX:+UseConcMarkSweepGC`.
- **G1 GC (Garbage-First Garbage Collector):** GC mặc định của HotSpot trên các máy điển hình thuộc Server-class từ JDK 9 trở đi; lựa chọn mặc định trong các môi trường khác có thể khác nhau. Nó chia heap thành nhiều Region, cân bằng giữa throughput và pause time, đồng thời cố gắng đáp ứng mục tiêu pause time do người dùng thiết lập. Tham số: `-XX:+UseG1GC`.
- **ZGC:** GC low latency, cần JDK phiên bản mới hơn hỗ trợ. Tham số: `-XX:+UseZGC`.
- **Shenandoah GC:** Một collector low pause khác, khả năng sử dụng phụ thuộc vào bản phân phối JDK cụ thể. Tham số: `-XX:+UseShenandoahGC`.

### Ghi log GC

Trong production hoặc khi troubleshooting vấn đề GC, **nhất định phải bật ghi log GC**. GC log chi tiết là cơ sở then chốt để phân tích và giải quyết vấn đề GC.

Dưới đây là một số tham số GC log của JDK 8:

```bash
# --- Cấu hình cơ bản được khuyến nghị ---
# In thông tin GC chi tiết
-XX:+PrintGCDetails
# In timestamp khi GC xảy ra (tính từ thời điểm JVM khởi động)
# -XX:+PrintGCTimeStamps
# In ngày và thời gian khi GC xảy ra (thường dùng hơn)
-XX:+PrintGCDateStamps
# Chỉ định đường dẫn output của file GC log, %t có thể output timestamp
-Xloggc:/path/to/gc-%t.log

# --- Cấu hình nâng cao được khuyến nghị ---
# In phân bố tuổi của object (giúp đánh giá tình trạng object được promote lên Old Generation)
-XX:+PrintTenuringDistribution
# In thông tin heap trước và sau GC
-XX:+PrintHeapAtGC
# In thông tin xử lý các loại reference (strong/soft/weak/phantom)
-XX:+PrintReferenceGC
# In thời gian pause của ứng dụng (Stop-The-World, STW)
-XX:+PrintGCApplicationStoppedTime

# --- Cấu hình rotation file GC log ---
# Bật rotation file GC log
-XX:+UseGCLogFileRotation
# Thiết lập số lượng file log rotation (ví dụ, giữ lại 14 file gần nhất)
-XX:NumberOfGCLogFiles=14
# Thiết lập kích thước tối đa của mỗi file log (ví dụ, 50MB)
-XX:GCLogFileSize=50M

# --- Cấu hình chẩn đoán hỗ trợ tùy chọn ---
# In thông tin thống kê safepoint (giúp phân tích nguyên nhân STW)
# -XX:+PrintSafepointStatistics
# -XX:PrintSafepointStatisticsCount=1
```

Từ JDK 9 trở đi nên sử dụng unified JVM logging framework `-Xlog`. Ví dụ, cấu hình sau ghi lại GC log chi tiết và bật rotation theo kích thước:

```bash
-Xlog:gc*:file=/path/to/gc-%t.log:time,uptime,level,tags:filecount=14,filesize=50M
```

Không phải tham số cũ nào cũng tiếp tục sử dụng được: một số tham số được mapping, deprecated hoặc không còn được nhận diện. Ví dụ, `PrintGCDetails` tương ứng với `-Xlog:gc*`, `PrintTenuringDistribution` tương ứng với `-Xlog:gc+age*=debug`, `PrintGCApplicationStoppedTime` tương ứng với `-Xlog:safepoint`. Cần lấy tài liệu về command `java` của JDK mục tiêu và output của `java -Xlog:help` làm chuẩn.

## Xử lý OOM

Với các ứng dụng lớn, việc gặp lỗi thiếu memory rất phổ biến và ngược lại có thể khiến ứng dụng crash. Đây là một trường hợp rất quan trọng nhưng khó giải quyết bằng cách tái hiện.

Đó là lý do JVM cung cấp một số tham số giúp dump heap memory ra file vật lý, sau đó có thể dùng file này để tìm memory leak:

```bash
# Tạo file heap dump khi heap Java cạn kiệt và ném OOME
-XX:+HeapDumpOnOutOfMemoryError

# Chỉ định file hoặc thư mục heap dump. Nếu chỉ định thư mục, JVM sẽ dùng tên mặc định java_pid<pid>.hprof
-XX:HeapDumpPath=/data/dumps/

# (Tùy chọn) Thực thi command hoặc script được chỉ định khi xảy ra OOM
# Ví dụ, gửi thông báo cảnh báo hoặc thử restart service (cần thận trọng khi sử dụng)
# -XX:OnOutOfMemoryError="<command> <args>"
# Ví dụ: -XX:OnOutOfMemoryError="sh /path/to/notify.sh"

# (Tùy chọn) Bật kiểm tra giới hạn overhead GC
# Nếu tỷ lệ thời gian GC trên tổng thời gian quá cao (mặc định 98%) và hiệu quả collection rất thấp (mặc định dưới 2% heap),
# sẽ sớm ném OOM để ngăn ứng dụng bị treo lâu trong GC.
-XX:+UseGCOverheadLimit
```

## Các tham số thường dùng khác

- `-server`: Bật rõ ràng Server mode của HotSpot VM. (Trên JVM 64-bit, đây thường là giá trị mặc định.)
- `-XX:+UseStringDeduplication`: (JDK 8u20+) Thử nhận diện các String object có nội dung giống nhau và dùng chung array dữ liệu character bên dưới để giảm memory sử dụng. Trong JDK 8, array bên dưới là `char[]`; các JDK hiện đại bật Compact Strings thường dùng `byte[]`. Tùy chọn này cũng yêu cầu sử dụng collector hỗ trợ string deduplication, chẳng hạn G1.
- `-XX:SurvivorRatio=<ratio>`: Thiết lập tỷ lệ kích thước giữa Eden và một Survivor. Ví dụ, `-XX:SurvivorRatio=8` biểu thị Eden:Survivor = 8:1.
- `-XX:MaxTenuringThreshold=<threshold>`: Thiết lập ngưỡng tuổi tối đa để object từ Young Generation được promote lên Old Generation (sau mỗi lần Minor GC, nếu object còn sống thì tuổi tăng 1). Giá trị mặc định liên quan đến collector và phiên bản JDK; ví dụ, trong tài liệu JDK 8, giá trị mặc định của Parallel GC là 15, còn CMS là 6.
- `-XX:+DisableExplicitGC`: Khiến JVM bỏ qua các yêu cầu GC tường minh như `System.gc()`, nhưng JVM vẫn tự thực hiện GC khi cần. Yêu cầu tường minh không đảm bảo trigger Full GC; có nên bật tham số này hay không cần được đánh giá dựa trên việc dọn dẹp direct memory, hành vi của framework và collector đang sử dụng, không thể khẳng định chung cho mọi trường hợp.
- `-XX:+UseLargePages`: (Cần OS hỗ trợ) Thử sử dụng large memory page (chẳng hạn 2MB thay vì 4KB), có thể cải thiện performance của ứng dụng sử dụng nhiều memory nhưng cần kiểm thử thận trọng.
- `-XX:MinHeapFreeRatio=<percent>` / `-XX:MaxHeapFreeRatio=<percent>`: Kiểm soát phần trăm tối thiểu/tối đa của heap memory còn free sau GC, dùng để tự động điều chỉnh kích thước heap (nếu `-Xms` và `-Xmx` không bằng nhau). Thông thường nên đặt `-Xms` và `-Xmx` giống nhau để tránh chi phí điều chỉnh.

**Lưu ý:** Các tham số sau trong các phiên bản JVM hiện đại có thể đã **deprecated, bị loại bỏ hoặc được bật mặc định nên không cần thiết lập thủ công**:

- `-XX:+UseLWPSynchronization`: Tùy chọn synchronization strategy cũ; JVM hiện đại thường có implementation được tối ưu hơn.
- `-XX:LargePageSizeInBytes`: Thường được tự động xác định bởi `-XX:+UseLargePages` hoặc thông qua cấu hình OS.
- `-XX:+UseStringCache`: Đã bị loại bỏ.
- `-XX:+UseCompressedStrings`: Tùy chọn experimental cũ, đã bị loại bỏ; Compact Strings được giới thiệu từ JDK 9 là một implementation khác.
- `-XX:+OptimizeStringConcat`: Tùy chọn tối ưu string concatenation của HotSpot C2 phiên bản cũ; từ JDK 9, string concatenation chủ yếu được implementation thông qua `invokedynamic` và `StringConcatFactory`, không thể xem hai cơ chế này là cùng một switch.

## Tổng kết

Bài viết cung cấp cho Java developer một hướng dẫn thực tiễn về cấu hình các tham số JVM thường dùng, nhằm giúp người đọc hiểu và tối ưu performance cũng như stability của ứng dụng Java. Các trọng điểm của bài viết gồm:

1. **Cấu hình heap memory:** Có thể thiết lập tường minh heap memory ban đầu và tối đa (`-Xms`, `-Xmx`) dựa trên môi trường deploy (ứng dụng server thường đặt hai giá trị bằng nhau). Việc có cần thiết lập tường minh kích thước Young Generation hay không cần được quyết định dựa trên garbage collector và GC log; G1 thường không được khuyến nghị đặt `-Xmn`.
2. **Quản lý Metaspace (Java 8+):** `-XX:MetaspaceSize` dùng để thiết lập ngưỡng high-water ban đầu trigger GC metadata, không phải capacity ban đầu của Metaspace. `-XX:MaxMetaspaceSize` có thể giới hạn native memory được class metadata sử dụng, nhưng giới hạn cụ thể cần được xác định dựa trên tình hình thực tế của ứng dụng.
3. **Lựa chọn garbage collector và log:** Giới thiệu các trường hợp sử dụng phù hợp của những thuật toán GC khác nhau, đồng thời nhấn mạnh sự cần thiết của việc bật GC log chi tiết trong môi trường production và test để troubleshooting; JDK 8 sử dụng tham số GC log truyền thống, từ JDK 9 trở đi sử dụng `-Xlog`.
4. **Troubleshooting sự cố OOM:** Giải thích cách tự động tạo heap dump khi xảy ra OOM bằng các tham số như `-XX:+HeapDumpOnOutOfMemoryError`, qua đó phục vụ việc phân tích memory leak về sau.
5. **Các tham số khác:** Giới thiệu ngắn gọn một số tham số hữu ích khác như string deduplication, đồng thời chỉ ra tình trạng của một số tham số cũ.

Để xem các trường hợp troubleshooting và tối ưu cụ thể, có thể tham khảo bài viết [Các trường hợp troubleshooting và tối ưu performance JVM trong production](https://javaguide.cn/java/jvm/jvm-in-action.html) do tác giả biên soạn.

<!-- @include: @article-footer.snippet.md -->
