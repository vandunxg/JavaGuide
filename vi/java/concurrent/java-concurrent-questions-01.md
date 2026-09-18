---
title: Tổng hợp câu hỏi phỏng vấn Java Concurrency thường gặp (Phần 1)
description: "Câu hỏi phỏng vấn nền tảng về lập trình đồng thời trong Java: giải thích sâu về khác biệt giữa thread và process, các cách tạo multi-thread, trạng thái vòng đời của thread, bốn điều kiện và cách phòng tránh deadlock, khái niệm concurrency và parallelism cùng các kiến thức cốt lõi khác."
category: Java
tag:
  - Java Concurrency
head:
  - - meta
    - name: keywords
      content: Java Concurrency,thread và process,multi-thread,deadlock,vòng đời thread,lập trình đồng thời,câu hỏi phỏng vấn Java,cách tạo thread
---

## Thread

### ⭐️ Thread và process là gì?

#### Process là gì?

Process là một lần thực thi của chương trình, là đơn vị cơ bản để hệ thống chạy chương trình, vì vậy process mang tính động. Hệ thống chạy một chương trình chính là quá trình một process được tạo, chạy rồi kết thúc.

Trong Java, khi khởi động hàm main thì thực chất là khởi động một process JVM, còn thread chứa hàm main là một thread trong process này, cũng được gọi là main thread.

Như hình dưới đây, trong Windows, bằng cách xem Task Manager, chúng ta có thể thấy rõ các process hiện đang chạy trên Windows (việc chạy các file `.exe`).

![Ví dụ về process - Windows](https://oss.javaguide.cn/github/javaguide/java/%E8%BF%9B%E7%A8%8B%E7%A4%BA%E4%BE%8B%E5%9B%BE%E7%89%87-Windows.png)

#### Thread là gì?

Thread tương tự process, nhưng thread là một đơn vị thực thi nhỏ hơn process. Một process có thể tạo ra nhiều thread trong quá trình thực thi. Khác với process, nhiều thread cùng loại chia sẻ tài nguyên **heap** và **method area** của process, nhưng mỗi thread có **program counter**, **virtual machine stack** và **native method stack** riêng. Vì vậy, khi hệ thống tạo một thread hoặc chuyển đổi giữa các thread, chi phí sẽ nhỏ hơn nhiều so với process. Cũng vì lý do đó, thread còn được gọi là process nhẹ.

Chương trình Java vốn đã là chương trình multi-thread. Chúng ta có thể dùng JMX để xem một chương trình Java thông thường có những thread nào, code như sau.

```java
public class MultiThread {
	public static void main(String[] args) {
		// Lấy MXBean quản lý thread của Java
	ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
		// Không cần lấy thông tin monitor và synchronizer đồng bộ, chỉ lấy thông tin thread và stack của thread
		ThreadInfo[] threadInfos = threadMXBean.dumpAllThreads(false, false);
		// Duyệt thông tin thread, chỉ in thông tin thread ID và tên thread
		for (ThreadInfo threadInfo : threadInfos) {
			System.out.println("[" + threadInfo.getThreadId() + "] " + threadInfo.getThreadName());
		}
	}
}
```

Chương trình trên cho output như sau (nội dung output có thể khác, không cần quá bận tâm đến tác dụng của từng thread bên dưới, chỉ cần biết main thread thực thi phương thức main):

```plain
[5] Attach Listener //Thêm sự kiện
[4] Signal Dispatcher // Thread phân phối và xử lý tín hiệu cho JVM
[3] Finalizer // Thread gọi phương thức finalize của object
[2] Reference Handler // Thread xóa reference
[1] main // main thread, entry point của chương trình
```

Từ output trên có thể thấy: **một chương trình Java chạy đồng thời main thread và nhiều thread khác**.

### Thread Java và thread của hệ điều hành khác nhau thế nào?

Các JDK đời đầu từng dùng green thread (Green Threads) để triển khai thread cấp user. Sau đó, trong HotSpot, thread truyền thống được tạo bằng `new Thread()` dùng platform thread (Platform Thread), platform thread thường ánh xạ theo tỷ lệ 1:1 tới thread của hệ điều hành và do hệ điều hành điều phối. Virtual thread (Virtual Thread) được Java 21 chính thức giới thiệu lại do JVM điều phối; một lượng lớn virtual thread có thể dùng chung ít platform thread làm thread mang, vì vậy không thể tiếp tục đồng nhất mọi thread Java với thread của hệ điều hành.

Ở trên chúng ta đã nhắc đến user thread và kernel thread. Vì nhiều độc giả chưa hiểu rõ sự khác nhau giữa hai loại này, dưới đây là phần giới thiệu ngắn:

- User thread: thread do chương trình ở user space quản lý và điều phối, chạy trong user space (dành riêng cho application).
- Kernel thread: thread do kernel của hệ điều hành quản lý và điều phối, chạy trong kernel space (chỉ chương trình kernel mới có thể truy cập).

Tóm tắt ngắn gọn sự khác nhau và đặc điểm của user thread và kernel thread: user-level thread thường được runtime điều phối trong user space, chi phí tạo và chuyển đổi thấp hơn; khả năng tận dụng multi-core phụ thuộc vào mô hình ánh xạ giữa user thread và kernel thread, mô hình many-to-one không thể tận dụng multi-core để chạy song song, còn mô hình many-to-many thì có thể. Kernel thread do hệ điều hành điều phối, chi phí tạo và chuyển đổi thường cao hơn, nhưng có thể trực tiếp tận dụng multi-core.

Tóm tắt mối quan hệ giữa thread Java và thread hệ điều hành trong một câu: **platform thread thường ánh xạ tới thread hệ điều hành, còn virtual thread do JVM điều phối và được gắn vào platform thread để thực thi**.

Thread model là cách liên kết giữa user thread và kernel thread. Có ba thread model thường gặp:

1. One-to-one (một user thread tương ứng với một kernel thread)
2. Many-to-one (nhiều user thread ánh xạ tới một kernel thread)
3. Many-to-many (nhiều user thread ánh xạ tới nhiều kernel thread)

![Ba thread model thường gặp](https://oss.javaguide.cn/github/javaguide/java/concurrent/three-types-of-thread-models.png)

Trong các hệ điều hành phổ biến như Windows và Linux, platform thread của HotSpot thường dùng mô hình one-to-one, tức là một platform thread tương ứng với một thread hệ điều hành. Virtual thread không dùng ánh xạ one-to-one này, mà được JVM điều phối thực thi trên một nhóm platform thread.

### ⭐️ Hãy mô tả ngắn gọn mối quan hệ, khác biệt, ưu và nhược điểm của thread và process

Hình dưới đây là vùng nhớ Java. Từ góc nhìn JVM, chúng ta hãy nói về mối quan hệ giữa thread và process.

![Vùng dữ liệu runtime của Java (sau JDK1.8)](https://oss.javaguide.cn/github/javaguide/java/jvm/java-runtime-data-areas-jdk1.8.png)

Từ hình trên có thể thấy: một process có thể có nhiều thread, nhiều thread chia sẻ tài nguyên **heap** và **method area (Metaspace sau JDK1.8)** của process, nhưng mỗi thread có **program counter**, **virtual machine stack** và **native method stack** riêng.

**Tóm tắt:** Thread là đơn vị chạy nhỏ hơn được chia ra từ process. Khác biệt lớn nhất giữa thread và process là về cơ bản các process độc lập với nhau, còn các thread thì không nhất thiết như vậy, vì các thread trong cùng một process rất dễ ảnh hưởng lẫn nhau. Chi phí thực thi thread thấp, nhưng không có lợi cho việc quản lý và bảo vệ tài nguyên; process thì ngược lại.

Dưới đây là phần mở rộng của kiến thức này!

Hãy suy nghĩ về câu hỏi sau: tại sao **program counter**, **virtual machine stack** và **native method stack** là private theo thread? Tại sao heap và method area được các thread chia sẻ?

#### Tại sao program counter là private?

Program counter chủ yếu có hai tác dụng sau:

1. Bytecode interpreter thay đổi program counter để lần lượt đọc instruction, từ đó thực hiện điều khiển luồng code, chẳng hạn như thực thi tuần tự, lựa chọn, vòng lặp và xử lý exception.
2. Trong trường hợp multi-thread, program counter dùng để ghi lại vị trí thread hiện đang thực thi, nhờ đó khi thread được chuyển về có thể biết lần trước thread đó đã chạy đến đâu.

Cần lưu ý rằng nếu thực thi native method thì program counter ghi địa chỉ undefined; chỉ khi thực thi code Java, program counter mới ghi địa chỉ của instruction tiếp theo.

Vì vậy, program counter là private chủ yếu để **sau khi chuyển thread có thể khôi phục đúng vị trí thực thi**.

#### Tại sao virtual machine stack và native method stack là private?

- **Virtual machine stack:** Trước khi mỗi Java method thực thi, một stack frame được tạo để lưu local variable table, operand stack, reference tới constant pool và các thông tin khác. Quá trình từ lúc gọi method đến khi thực thi xong tương ứng với quá trình một stack frame được push và pop trong virtual machine stack.
- **Native method stack:** Tác dụng rất giống virtual machine stack, điểm khác là: **virtual machine stack phục vụ việc virtual machine thực thi Java method (tức bytecode), còn native method stack phục vụ Native method mà virtual machine sử dụng.** Trong HotSpot virtual machine, hai stack này được hợp nhất.

Vì vậy, để **đảm bảo local variable trong thread không bị thread khác truy cập**, virtual machine stack và native method stack là private theo thread.

#### Tìm hiểu ngắn gọn về heap và method area

Heap và method area là tài nguyên được mọi thread chia sẻ. Heap là vùng memory lớn nhất trong process, chủ yếu dùng để lưu object mới tạo (gần như mọi object đều được cấp phát memory tại đây). Method area chủ yếu dùng để lưu class information, constant, static variable đã được load, code sau khi được JIT compiler compile và các data khác.

### Tạo thread như thế nào?

Nói chung, có nhiều cách tạo thread, chẳng hạn như kế thừa class `Thread`, implement interface `Runnable`, implement interface `Callable`, dùng thread pool, dùng class `CompletableFuture` và nhiều cách khác.

Tuy nhiên, thực chất các cách này không trực tiếp tạo ra thread. Nói chính xác hơn, đây đều là những cách dùng multi-thread trong code Java.

Nói nghiêm ngặt, Java chỉ có một cách tạo thread, đó là tạo bằng `new Thread().start()`. Bất kể dùng cách nào, cuối cùng vẫn dựa trên `new Thread().start()`.

### ⭐️ Hãy nói về vòng đời và trạng thái của thread

Tại một thời điểm cụ thể trong vòng đời chạy, thread Java chỉ có thể ở một trong 6 trạng thái sau:

- NEW: trạng thái ban đầu, thread đã được tạo nhưng chưa gọi `start()`.
- RUNNABLE: trạng thái có thể chạy, thread đã được gọi `start()` và đang chờ chạy.
- BLOCKED: trạng thái bị block, cần chờ lock được giải phóng.
- WAITING: trạng thái waiting, biểu thị thread cần chờ thread khác thực hiện một hành động cụ thể (notify hoặc interrupt).
- TIMED_WAITING: trạng thái waiting có timeout, có thể tự return sau khoảng thời gian chỉ định thay vì chờ mãi như WAITING.
- TERMINATED: trạng thái kết thúc, biểu thị thread đã chạy xong.

Trong vòng đời, thread không cố định ở một trạng thái mà chuyển đổi giữa các trạng thái khác nhau theo quá trình thực thi code.

Sơ đồ chuyển đổi trạng thái thread Java (nguồn hình: [Đính chính | Ba lỗi về trạng thái thread trong 《Nghệ thuật lập trình đồng thời》](https://mp.weixin.qq.com/s/0UTyrJpRKaKhkhHcQtXAiA)):

![Sơ đồ chuyển đổi trạng thái thread Java](https://oss.javaguide.cn/github/javaguide/java/concurrent/640.png)

Như hình trên, sau khi được tạo, thread ở trạng thái **NEW (mới tạo)**. Sau khi gọi method `start()`, thread bắt đầu chạy và ở trạng thái **READY (có thể chạy)**. Thread ở trạng thái có thể chạy sẽ chuyển sang trạng thái **RUNNING (đang chạy)** sau khi nhận được time slice của CPU.

> Ở cấp hệ điều hành, thread có trạng thái READY và RUNNING; còn ở cấp JVM, chỉ có thể thấy trạng thái RUNNABLE (nguồn hình: [HowToDoInJava](https://howtodoinJava.com/ "HowToDoInJava"): [Java Thread Life Cycle and Thread States](https://howtodoinJava.com/Java/multi-threading/Java-thread-life-cycle-and-thread-states/ "Java Thread Life Cycle and Thread States")), vì vậy hệ thống Java thường gọi chung hai trạng thái này là trạng thái **RUNNABLE (đang chạy)**.
>
> **Tại sao JVM không phân biệt hai trạng thái này?** `Thread.State` của Java mô tả trạng thái thread ở cấp JVM, không dùng để phản ánh toàn bộ trạng thái nội bộ của scheduler hệ điều hành. Chính sách scheduling, độ dài time slice và việc có dùng cơ chế round-robin hay không đều do hệ điều hành và cấu hình quyết định, không thể cố định khái quát thành cơ chế scheduling round-robin 10～20 ms.

![RUNNABLE-VS-RUNNING](https://oss.javaguide.cn/github/javaguide/java/RUNNABLE-VS-RUNNING.png)

- Sau khi thread thực thi method `wait()`, thread chuyển sang trạng thái **WAITING (waiting)**. Thread ở trạng thái waiting cần thông báo từ thread khác mới có thể trở lại trạng thái running.
- Trạng thái **TIMED_WAITING (waiting có timeout)** tương đương trạng thái waiting nhưng có thêm giới hạn timeout. Ví dụ, method `sleep（long millis）` hoặc `wait（long millis）` có thể đưa thread vào trạng thái TIMED_WAITING. Khi hết thời gian timeout, thread sẽ trở về trạng thái RUNNABLE.
- Khi thread đi vào method/block `synchronized`, hoặc sau khi gọi `wait` rồi được `notify` để vào lại method/block `synchronized` nhưng lock đang bị thread khác chiếm, thread sẽ chuyển sang trạng thái **BLOCKED (bị block)**.
- Sau khi thực thi xong method `run()`, thread sẽ chuyển sang trạng thái **TERMINATED (kết thúc)**.

### Context switch của thread là gì?

Trong quá trình thực thi, thread có điều kiện và trạng thái chạy riêng (còn gọi là context), chẳng hạn như program counter và thông tin stack đã nói ở trên. Khi xảy ra các tình huống sau, thread sẽ rời khỏi trạng thái đang chiếm CPU.

- Chủ động nhường CPU, chẳng hạn gọi `sleep()`, `wait()`.
- Hết time slice, vì hệ điều hành phải ngăn một thread hoặc process chiếm CPU quá lâu khiến các thread hoặc process khác bị starvation.
- Gọi system interrupt dạng blocking, chẳng hạn request I/O khiến thread bị block.
- Bị terminate hoặc kết thúc chạy.

Ba trường hợp đầu đều gây ra thread switch. Thread switch nghĩa là phải lưu context của thread hiện tại để khôi phục khi thread đó lần sau chiếm CPU, đồng thời load context của thread tiếp theo sẽ chiếm CPU. Đây chính là **context switch**.

Context switch là chức năng cơ bản của hệ điều hành hiện đại. Vì mỗi lần đều cần lưu và khôi phục information, CPU, memory và các system resource khác phải xử lý, nên hiệu suất sẽ bị hao tổn nhất định. Nếu chuyển đổi quá thường xuyên, hiệu suất tổng thể sẽ thấp.

### So sánh method Thread#sleep() và Object#wait()

**Điểm chung:** Cả hai đều có thể tạm dừng việc thực thi của thread.

**Điểm khác:**

- **Method `sleep()` không release lock, còn method `wait()` release lock.**
- `wait()` thường được dùng cho tương tác/truyền thông giữa các thread, còn `sleep()` thường được dùng để tạm dừng thực thi.
- Sau khi gọi method `wait()`, thread không tự tỉnh lại mà cần thread khác gọi `notify()` hoặc `notifyAll()` trên cùng object. Sau khi method `sleep()` thực thi xong, thread tự tỉnh lại; ngoài ra, thread cũng tự tỉnh lại sau khi `wait(long timeout)` hết timeout.
- `sleep()` là static native method của class `Thread`, còn `wait()` là native method của class `Object`. Tại sao lại thiết kế như vậy? Câu hỏi tiếp theo sẽ giải thích.

### Tại sao method wait() không được định nghĩa trong Thread?

`wait()` cho phép thread đã lấy object lock đi vào trạng thái waiting và tự release object lock mà thread hiện đang chiếm. Mỗi object (`Object`) đều có object lock. Vì muốn release object lock mà thread hiện tại đang chiếm và đưa thread vào trạng thái WAITING, tự nhiên phải thao tác trên object tương ứng (`Object`) thay vì thread hiện tại (`Thread`).

Câu hỏi tương tự: **Tại sao method `sleep()` được định nghĩa trong `Thread`?**

Vì `sleep()` khiến thread hiện tại tạm dừng thực thi, không liên quan đến object class và cũng không cần lấy object lock.

### Có thể gọi trực tiếp method run của class Thread không?

Đây là một câu hỏi phỏng vấn multi-thread Java rất kinh điển và thường xuyên được hỏi trong phỏng vấn. Câu hỏi rất đơn giản nhưng nhiều người lại không trả lời được!

Khi tạo một `Thread`, thread đi vào trạng thái mới tạo. Gọi method `start()` sẽ khởi động một thread và đưa thread vào trạng thái ready. Sau khi được cấp time slice, thread có thể bắt đầu chạy. `start()` thực hiện công việc chuẩn bị tương ứng cho thread rồi tự động thực thi nội dung method `run()`, đây mới là hoạt động multi-thread thực sự. Nhưng nếu thực thi trực tiếp method `run()`, method `run()` sẽ được coi như một method thông thường và chạy trên thread gọi method đó, nên đây không phải hoạt động multi-thread.

**Tóm tắt: Chỉ gọi method `start()` mới có thể khởi động thread và đưa thread vào trạng thái ready; nếu thực thi trực tiếp method `run()` thì sẽ không chạy theo cách multi-thread.**

## Multi-thread

### Khác biệt giữa concurrency và parallelism

- **Concurrency:** Hai hoặc nhiều job thực thi trong cùng một **khoảng thời gian**.
- **Parallelism:** Hai hoặc nhiều job thực thi tại cùng một **thời điểm**.

Điểm quan trọng nhất là: có thực thi **đồng thời** hay không.

### Khác biệt giữa synchronous và asynchronous

- **Synchronous:** Sau khi phát ra một call, trước khi nhận được kết quả thì call đó không thể return mà phải tiếp tục chờ.
- **Asynchronous:** Sau khi phát ra call, không cần chờ kết quả return, call đó return ngay.

### ⭐️ Tại sao cần dùng multi-thread?

Trước hết, xét từ góc độ tổng thể:

- **Từ tầng dưới của computer:** Thread có thể được ví như process nhẹ, là đơn vị nhỏ nhất để chương trình thực thi; chi phí chuyển đổi và scheduling giữa các thread nhỏ hơn rất nhiều so với process. Ngoài ra, thời đại multi-core CPU có nghĩa là nhiều thread có thể chạy đồng thời, giúp giảm chi phí context switch của thread.
- **Từ xu hướng phát triển Internet hiện đại:** Các system hiện nay thường yêu cầu concurrency ở mức hàng triệu, thậm chí hàng chục triệu. Lập trình concurrent bằng multi-thread chính là nền tảng để phát triển system high-concurrency; tận dụng tốt cơ chế multi-thread có thể nâng cao đáng kể concurrency và performance tổng thể của system.

Tiếp tục đi sâu từ tầng dưới của computer:

- **Thời đại single-core:** Trong thời đại single-core, multi-thread chủ yếu nhằm nâng cao hiệu suất process đơn tận dụng CPU và system I/O. Giả sử chỉ chạy một process Java, khi request I/O, nếu process Java chỉ có một thread thì khi thread này bị I/O block, toàn bộ process cũng bị block. CPU và thiết bị I/O chỉ có một bên đang chạy, nên có thể nói đơn giản rằng hiệu suất tổng thể của system chỉ là 50%. Khi dùng multi-thread, một thread bị I/O block thì các thread khác vẫn có thể tiếp tục dùng CPU, từ đó nâng cao hiệu suất tổng thể của process Java khi tận dụng system resource.
- **Thời đại multi-core:** Trong thời đại multi-core, multi-thread chủ yếu nhằm nâng cao khả năng process tận dụng multi-core CPU. Ví dụ, nếu cần tính một task phức tạp mà chỉ dùng một thread thì dù system có bao nhiêu CPU core cũng chỉ một CPU core được sử dụng. Nếu tạo nhiều thread, các thread này có thể được ánh xạ đến nhiều CPU core ở tầng dưới để thực thi. Khi các thread trong task không tranh chấp resource, hiệu suất thực thi task sẽ tăng đáng kể, xấp xỉ (thời gian thực thi trên single-core / số CPU core).

### ⭐️ Single-core CPU có hỗ trợ multi-thread Java không?

Single-core CPU có hỗ trợ multi-thread Java. Hệ điều hành dùng cơ chế round-robin theo time slice để phân phối thời gian CPU cho các thread khác nhau. Dù single-core CPU chỉ có thể thực thi một task mỗi lần, việc chuyển đổi nhanh giữa nhiều thread khiến user cảm thấy nhiều task đang diễn ra đồng thời.

Nhân tiện, hãy nói sơ qua về cách Java scheduling thread.

Hệ điều hành chủ yếu dùng hai cách scheduling thread để quản lý việc thực thi multi-thread:

- **Preemptive Scheduling:** Hệ điều hành quyết định khi nào tạm dừng thread đang chạy và chuyển sang thread khác. Việc chuyển đổi thường được kích hoạt bởi system clock interrupt (round-robin theo time slice) hoặc event ưu tiên cao khác (chẳng hạn I/O operation hoàn tất). Cách này có chi phí context switch, nhưng tính công bằng và hiệu suất tận dụng CPU tốt hơn, ít bị block.
- **Cooperative Scheduling:** Sau khi thực thi xong, thread chủ động thông báo cho system chuyển sang thread khác. Cách này có thể giảm chi phí performance do context switch, nhưng tính công bằng kém hơn và dễ bị block.

Cách Java dùng để scheduling thread là preemptive. Nói cách khác, bản thân JVM không phụ trách scheduling thread mà ủy thác việc scheduling thread cho hệ điều hành. Hệ điều hành thường dựa trên thread priority và time slice để scheduling việc thực thi thread; thread có priority cao thường có nhiều cơ hội nhận time slice CPU hơn.

### ⭐️ Chạy nhiều thread trên single-core CPU có chắc chắn hiệu suất cao hơn không?

Hiệu suất chạy đồng thời nhiều thread trên single-core CPU có cao hơn hay không phụ thuộc vào loại thread và tính chất task. Nói chung có hai loại thread:

1. **CPU-intensive:** Thread CPU-intensive chủ yếu thực hiện tính toán và xử lý logic, cần chiếm nhiều CPU resource.
2. **I/O-intensive:** Thread I/O-intensive chủ yếu thực hiện thao tác input/output, chẳng hạn đọc ghi file và network communication, cần chờ thiết bị I/O phản hồi nhưng không chiếm quá nhiều CPU resource.

Trên single-core CPU, tại một thời điểm chỉ có một thread đang chạy, các thread khác phải chờ CPU phân phối time slice. Nếu thread là CPU-intensive, nhiều thread chạy đồng thời sẽ gây thread switch thường xuyên, tăng system overhead và giảm hiệu suất. Nếu thread là I/O-intensive, nhiều thread chạy đồng thời có thể tận dụng thời gian CPU rảnh trong lúc chờ I/O, từ đó nâng cao hiệu suất.

Vì vậy, với single-core CPU, nếu task là CPU-intensive thì tạo quá nhiều thread sẽ ảnh hưởng hiệu suất; nếu task là I/O-intensive thì tạo nhiều thread sẽ nâng cao hiệu suất. Tất nhiên, số lượng "nhiều" cũng phải vừa phải, không được vượt quá giới hạn system có thể chịu.

### Dùng multi-thread có thể gây ra vấn đề gì?

Mục đích của lập trình concurrency là nâng cao hiệu suất thực thi chương trình và từ đó tăng tốc độ chạy chương trình, nhưng lập trình concurrency không phải lúc nào cũng làm chương trình chạy nhanh hơn; ngoài ra còn có thể gặp nhiều vấn đề như memory leak, deadlock, thread không an toàn và các vấn đề khác.

### Hiểu thread-safe và thread-unsafe như thế nào?

Thread-safe và thread-unsafe mô tả việc có thể đảm bảo tính đúng đắn và nhất quán khi truy cập cùng một data trong môi trường multi-thread hay không.

- Thread-safe nghĩa là trong môi trường multi-thread, với cùng một data, dù có bao nhiêu thread truy cập đồng thời cũng có thể đảm bảo tính đúng đắn và nhất quán của data.
- Thread-unsafe nghĩa là trong môi trường multi-thread, khi nhiều thread đồng thời truy cập cùng một data, có thể dẫn tới data lộn xộn, sai hoặc mất.

## ⭐️ Deadlock

### Thread deadlock là gì?

Thread deadlock mô tả tình huống nhiều thread đồng thời bị block, một hoặc tất cả các thread trong số đó đang chờ một resource được release. Vì thread bị block vô thời hạn nên chương trình không thể kết thúc bình thường.

Như hình dưới đây, thread A đang giữ resource 2, thread B đang giữ resource 1, cả hai đồng thời muốn request resource của nhau nên hai thread chờ lẫn nhau và đi vào trạng thái deadlock.

![Minh họa tình huống deadlock: thread A giữ resource1 và chờ resource2, thread B giữ resource2 và chờ resource1, hình thành vòng chờ khép kín](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/dead-lock-deadlock-scenario.png)

Ví dụ dưới đây minh họa thread deadlock; code mô phỏng tình huống deadlock trong hình trên (nguồn code từ 《Vẻ đẹp của lập trình concurrency》):

```java
public class DeadLockDemo {
    private static Object resource1 = new Object();//resource 1
    private static Object resource2 = new Object();//resource 2

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (resource1) {
                System.out.println(Thread.currentThread() + "get resource1");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource2");
                synchronized (resource2) {
                    System.out.println(Thread.currentThread() + "get resource2");
                }
            }
        }, "Thread 1").start();

        new Thread(() -> {
            synchronized (resource2) {
                System.out.println(Thread.currentThread() + "get resource2");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource1");
                synchronized (resource1) {
                    System.out.println(Thread.currentThread() + "get resource1");
                }
            }
        }, "Thread 2").start();
    }
}
```

Output

```plain
Thread[Thread 1,5,main]get resource1
Thread[Thread 2,5,main]get resource2
Thread[Thread 1,5,main]waiting get resource2
Thread[Thread 2,5,main]waiting get resource1
```

Thread A lấy monitor lock của `resource1` thông qua `synchronized (resource1)`, sau đó dùng `Thread.sleep(1000);` để thread A ngủ 1 giây, nhằm cho thread B có cơ hội chạy và lấy monitor lock của `resource2`. Sau khi kết thúc sleep, thread A và thread B đều bắt đầu request resource của đối phương, rồi hai thread rơi vào trạng thái chờ lẫn nhau, từ đó tạo ra deadlock.

Ví dụ trên thỏa mãn bốn điều kiện cần để deadlock xảy ra:

1. **Mutual exclusion:** Tại một thời điểm, resource đó chỉ do một thread chiếm giữ.
2. **Hold and wait:** Khi một thread bị block vì request resource, thread đó vẫn giữ các resource đã nhận được.
3. **No preemption:** Resource mà thread đã nhận được không thể bị thread khác cưỡng chế lấy đi trước khi dùng xong; chỉ thread đó mới release resource sau khi dùng xong.
4. **Circular wait:** Giữa một số thread hình thành quan hệ chờ resource theo vòng khép kín, đầu và cuối nối với nhau.

### Phát hiện deadlock như thế nào?

- Dùng `jstack <pid>` hoặc `jcmd <pid> Thread.print -l` để xem thread stack và thông tin concurrent lock. Nếu phát hiện deadlock ở cấp Java, output sẽ liệt kê các thread liên quan cùng lock mà chúng đang giữ và chờ. `jmap` chủ yếu dùng để xem thông tin heap hoặc tạo heap dump, không phải công cụ chẩn đoán thread deadlock.
- Dùng các công cụ như VisualVM và JConsole để kiểm tra.

Dưới đây dùng công cụ JConsole làm ví dụ minh họa.

Trước hết, tìm thư mục bin của JDK, tìm jconsole rồi double-click để mở.

![jconsole](https://oss.javaguide.cn/github/javaguide/java/concurrent/jdk-home-bin-jconsole.png)

Với user MAC, có thể dùng `/usr/libexec/java_home -V` để xem thư mục cài JDK. Sau khi tìm được, dùng `open . + đường dẫn thư mục` để mở. Ví dụ, path JDK trên máy tôi là:

```bash
 open . /Users/guide/Library/Java/JavaVirtualMachines/corretto-1.8.0_252/Contents/Home
```

Sau khi mở jconsole, connect tới chương trình tương ứng, sau đó vào giao diện thread và chọn phát hiện deadlock!

![Phát hiện deadlock bằng jconsole](https://oss.javaguide.cn/github/javaguide/java/concurrent/jconsole-check-deadlock.png)

![jconsole phát hiện deadlock](https://oss.javaguide.cn/github/javaguide/java/concurrent/jconsole-check-deadlock-done.png)

### Phòng ngừa và tránh thread deadlock như thế nào?

**Phòng ngừa deadlock như thế nào?** Chỉ cần phá vỡ các điều kiện cần để deadlock xảy ra:

1. **Phá vỡ hold and wait:** Request tất cả resource một lần.
2. **Phá vỡ no preemption:** Khi thread đang giữ một phần resource tiếp tục request resource khác nhưng không request được, thread có thể chủ động release các resource đang giữ.
3. **Phá vỡ circular wait:** Phòng ngừa bằng cách request resource theo thứ tự. Request resource theo một thứ tự nhất định, khi release thì release theo thứ tự ngược lại, từ đó phá vỡ circular wait.

**Tránh deadlock như thế nào?**

Tránh deadlock là khi phân phối resource, dùng algorithm (chẳng hạn Banker’s algorithm) để tính toán và đánh giá việc phân phối resource, đưa system vào trạng thái an toàn.

> **Trạng thái an toàn** là trạng thái system có thể phân phối resource cần thiết cho mỗi thread theo một thứ tự tiến hành nhất định của các thread (P1, P2, P3……Pn), cho đến khi đáp ứng nhu cầu resource tối đa của từng thread, để mọi thread đều có thể hoàn tất thuận lợi. Sequence `<P1、P2、P3.....Pn>` được gọi là safe sequence.

Chúng ta sửa code của thread 2 như dưới đây thì deadlock sẽ không xảy ra.

```java
new Thread(() -> {
            synchronized (resource1) {
                System.out.println(Thread.currentThread() + "get resource1");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource2");
                synchronized (resource2) {
                    System.out.println(Thread.currentThread() + "get resource2");
                }
            }
        }, "Thread 2").start();
```

Output:

```plain
Thread[Thread 1,5,main]get resource1
Thread[Thread 1,5,main]waiting get resource2
Thread[Thread 1,5,main]get resource2
Thread[Thread 2,5,main]get resource1
Thread[Thread 2,5,main]waiting get resource2
Thread[Thread 2,5,main]get resource2

Process finished with exit code 0
```

Hãy phân tích vì sao code trên tránh được deadlock.

Thread 1 lấy monitor lock của `resource1` trước, lúc này thread 2 không thể lấy được lock đó. Sau đó thread 1 lấy monitor lock của `resource2` và có thể lấy thành công. Tiếp theo thread 1 release việc chiếm giữ monitor lock của `resource1` và `resource2`, thread 2 lấy được lock và có thể thực thi. Như vậy, điều kiện circular wait bị phá vỡ nên deadlock được tránh.

<!-- @include: @article-footer.snippet.md -->
