---
title: Tổng hợp các câu hỏi thường gặp về virtual thread
description: "Giải thích chi tiết về virtual thread trong Java 21: làm rõ vị trí, nguyên lý scheduling, khác biệt với platform thread, trường hợp sử dụng, cách tạo, giới hạn hiệu năng, cách tích hợp vào Spring Boot và các lưu ý thực tiễn của Virtual Threads."
category: Java
tag:
  - Java Concurrency
head:
  - - meta
    - name: keywords
      content: Java Virtual Thread,Virtual Threads,Project Loom,tính năng mới Java 21,lightweight thread,coroutine,nguyên lý virtual thread
---

<!-- @include: @article-header.snippet.md -->

Một request Web đi vào, code cần truy vấn database, gọi API từ xa, đọc ghi file. Với cách viết synchronous truyền thống, request này sẽ chiếm một platform thread, dù phần lớn thời gian đang chờ I/O.

Thread pool có thể giảm chi phí tạo thread, nhưng không thay đổi được một thực tế: số lượng platform thread vẫn bị giới hạn bởi số lượng thread của hệ điều hành, bộ nhớ và chi phí scheduling. Khi số request đồng thời tiếp tục tăng, các thread trong thread pool sẽ nhanh chóng bị task đang xếp hàng chiếm hết, throughput nhanh chóng chạm giới hạn.

Virtual thread ra đời để giải quyết vấn đề này. Nó cho phép tiếp tục dùng code synchronous đơn giản, đồng thời không để các task chờ I/O chiếm platform thread đắt giá trong thời gian dài.

## Virtual thread là gì?

Virtual thread (Virtual Thread) là một lightweight thread được Java 21 chính thức giới thiệu, đồng thời là một implementation của `java.lang.Thread`. Nó do JDK quản lý và scheduling, thay vì liên kết một-một trực tiếp với một thread của hệ điều hành.

Platform thread (Platform Thread) thường là một lớp wrapper mỏng quanh thread của hệ điều hành. Khi chạy, một platform thread sẽ chiếm một thread của hệ điều hành trong toàn bộ vòng đời. Virtual thread thì khác: khi chạy code Java, nó cần được mount lên một platform thread; khi thực hiện thao tác blocking có thể suspend, JDK có thể unmount nó khỏi platform thread, để platform thread đó thực thi virtual thread khác.

Vì vậy, số lượng virtual thread có thể lớn hơn rất nhiều số lượng platform thread. Tài liệu chính thức dùng virtual memory để so sánh: hệ điều hành ánh xạ một lượng lớn virtual address vào lượng physical memory hữu hạn, còn Java runtime ánh xạ một lượng lớn virtual thread vào số ít platform thread.

Một số điểm chính của virtual thread:

- Virtual thread vẫn là `Thread`, hỗ trợ `ThreadLocal`, interrupt, exception stack, debugging và quan sát bằng JFR.
- Virtual thread phù hợp với task có nhiều thời gian blocking chờ đợi, chẳng hạn HTTP call, database query, truy cập message queue, file hoặc network I/O.
- Virtual thread không phải execution unit CPU nhanh hơn và sẽ không làm code thuần computation chạy nhanh hơn.
- Virtual thread rất rẻ, thông thường nên áp dụng mô hình “mỗi task một virtual thread”, thay vì pool và reuse như platform thread.

## Virtual thread có quan hệ gì với platform thread?

Trong Java, quan hệ giữa virtual thread, platform thread và thread của hệ điều hành đại khái như sau:

![Quan hệ giữa virtual thread, platform thread và kernel thread của hệ thống](https://oss.javaguide.cn/github/javaguide/java/new-features/virtual-threads-platform-threads-kernel-threads-relationship.png)

Trong các hệ điều hành phổ biến như Windows, Linux, platform thread của HotSpot JVM thường dùng mô hình thread một-một, tức là một platform thread tương ứng với một thread của hệ điều hành. Sau khi virtual thread được đưa vào, JDK thêm một lớp scheduling phía trên platform thread:

- Virtual thread là nơi chứa task; `Thread.currentThread()` mà code nghiệp vụ nhìn thấy trả về chính virtual thread.
- Platform thread là vật mang (Carrier Thread) của virtual thread, chịu trách nhiệm thực thi code Java trong virtual thread.
- Hệ điều hành vẫn chỉ scheduling platform thread và không biết virtual thread tồn tại.

Khi bắt đầu thực thi, một virtual thread được scheduler của JDK mount lên một platform thread. Khi thực thi đến điểm blocking như I/O, `BlockingQueue.take()`, `Future.get()` có hỗ trợ suspend, virtual thread có thể unmount, giải phóng platform thread để tiếp tục thực thi virtual thread khác. Sau khi thao tác blocking sẵn sàng, virtual thread lại được submit về scheduler, mount lên một platform thread để tiếp tục thực thi.

Quá trình mount và unmount này trong suốt với code nghiệp vụ. Code bạn viết vẫn là code synchronous thông thường:

```java
String body = httpClient.send(request, BodyHandlers.ofString()).body();
Result result = repository.query(body);
return service.handle(result);
```

Nếu các call này phát sinh blocking bên trong, virtual thread có thể tự suspend; nếu đổi sang platform thread, thread đó sẽ liên tục chiếm thread tương ứng của hệ điều hành.

## Project Loom có quan hệ gì với virtual thread?

Project Loom là project trong OpenJDK nhằm cải tiến mô hình concurrency của Java, còn virtual thread là một trong những thành quả quan trọng nhất của Loom. Virtual thread lần lượt được preview trong JDK 19, JDK 20, cuối cùng được chính thức đưa vào JDK 21 thông qua [JEP 444](https://openjdk.org/jeps/444).

Loom không chỉ bổ sung một API lightweight thread. Nó còn thúc đẩy việc điều chỉnh các năng lực đi kèm của JDK như blocking I/O, debugging, JFR và thread dump, giúp phong cách lập trình thread-per-request truyền thống một lần nữa có khả năng mở rộng trong các scenario I/O concurrency cao.

Đây cũng là điểm khác biệt quan trọng giữa virtual thread và “coroutine library” thông thường: virtual thread được đưa vào thread model của Java platform. Debugger, Profiler, JFR và thread dump đều có thể hiểu nó theo đơn vị thread, thay vì tách call chain nghiệp vụ thành một loạt callback stage.

## Virtual thread giải quyết vấn đề gì?

Nhiều chương trình server vốn phù hợp với mô hình “mỗi request một thread”. Ưu điểm của nó rất rõ: code thực thi tuần tự, exception có thể được throw dọc theo call stack, debugger có thể theo dõi từng bước, thread dump cũng cho thấy request đang bị kẹt ở đâu.

Vấn đề là platform thread quá đắt.

Giả sử một API có thời gian xử lý trung bình 50ms, hệ thống cần đạt 2000 QPS thì theo ước tính sơ bộ bằng Little's Law, cần xử lý đồng thời khoảng 100 request. Nếu thời gian xử lý trung bình của API tăng lên 500ms, cùng mức 2000 QPS sẽ cần khoảng 1000 request concurrent. Khi mỗi request chiếm một platform thread, số lượng thread rất dễ trở thành bottleneck trước cả CPU, network bandwidth, database connection và các resource khác.

Lập trình asynchronous và Reactive có thể giải phóng thread khỏi việc chờ I/O, nhưng cái giá cũng rất rõ: call chain bị tách thành callback, chain `CompletableFuture` hoặc reactive pipeline; xử lý exception, debugging, flame graph và thread context đều trở nên phức tạp hơn.

Virtual thread cố gắng giữ lại khả năng đọc của code synchronous, đồng thời giảm chi phí chiếm platform thread trong lúc blocking chờ đợi. Thứ nó chủ yếu cải thiện là throughput và khả năng tiếp nhận concurrency, không phải tốc độ thực thi của một request.

## Virtual thread phù hợp với những scenario nào?

Virtual thread phù hợp nhất với các task có đặc điểm sau:

- Số lượng task concurrent rất lớn, thường ở mức hàng nghìn hoặc hàng chục nghìn.
- Task dành phần lớn thời gian chờ I/O, chẳng hạn database, Redis, HTTP/RPC, message queue, file và network read/write.
- Code hiện tại chủ yếu theo mô hình synchronous blocking và không muốn chuyển sang async chain phức tạp để tăng khả năng mở rộng.
- Muốn giữ call stack truyền thống để thuận tiện cho debugging, phân tích load test và xử lý sự cố production.

Các scenario điển hình gồm:

- Gọi database và external HTTP service trong API Spring MVC / Servlet.
- Task nền gọi hàng loạt API của bên thứ ba.
- Gateway hoặc service aggregation gọi concurrent nhiều downstream service.
- Logic consume message có database write hoặc remote call dạng blocking.

Virtual thread không phù hợp để làm task CPU-intensive “chạy nhanh hơn”. Nếu task chủ yếu là tính hash, nén image, sort array lớn hoặc chạy rule engine phức tạp, sau khi số lượng thread vượt số core CPU, throughput thường sẽ không tiếp tục tăng. Với công việc CPU-intensive, vẫn nên tập trung vào algorithm, data structure, batch processing, parallel stream, thread pool chuyên dụng cho computation hoặc tối ưu native.

## Tạo virtual thread như thế nào?

Trong JDK 21 có bốn cách tạo thường gặp.

### Sử dụng `Thread.startVirtualThread()`

Phù hợp để khởi động một virtual thread đơn giản:

```java
public class VirtualThreadDemo {
  public static void main(String[] args) throws InterruptedException {
    Thread thread = Thread.startVirtualThread(() -> {
      System.out.println(Thread.currentThread());
    });

    thread.join();
  }
}
```

Cần lưu ý virtual thread là daemon thread. Nếu method `main` không chờ nó kết thúc, JVM có thể thoát ngay, khiến task chưa kịp thực thi xong.

### Sử dụng `Thread.ofVirtual()`

`Thread.ofVirtual()` trả về một `Thread.Builder.OfVirtual`, có thể thiết lập tên thread, đồng thời lựa chọn start ngay sau khi tạo hoặc chưa start:

```java
public class VirtualThreadDemo {
  public static void main(String[] args) throws InterruptedException {
    Thread unstarted = Thread.ofVirtual()
        .name("order-query")
        .unstarted(() -> System.out.println("query order"));

    unstarted.start();
    unstarted.join();

    Thread started = Thread.ofVirtual()
        .name("payment-query")
        .start(() -> System.out.println("query payment"));

    started.join();
  }
}
```

### Sử dụng `ThreadFactory`

Nếu muốn thống nhất cách đặt tên thread hoặc giao thread factory cho framework sử dụng, có thể tạo virtual thread thông qua `ThreadFactory`:

```java
import java.util.concurrent.ThreadFactory;

public class VirtualThreadDemo {
  public static void main(String[] args) throws InterruptedException {
    ThreadFactory factory = Thread.ofVirtual()
        .name("worker-", 0)
        .factory();

    Thread thread = factory.newThread(() -> {
      System.out.println(Thread.currentThread().getName());
    });

    thread.start();
    thread.join();
  }
}
```

### Sử dụng `Executors.newVirtualThreadPerTaskExecutor()`

Trong phát triển nghiệp vụ, đây là cách phổ biến nhất. Nó tạo một virtual thread mới cho mỗi task được submit:

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class VirtualThreadDemo {
  public static void main(String[] args) throws Exception {
    try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
      Future<String> future = executor.submit(() -> {
        return "hello virtual thread";
      });

      System.out.println(future.get());
    }
  }
}
```

`ExecutorService` ở đây không phải thread pool theo nghĩa truyền thống. Nó không duy trì một nhóm virtual thread cố định để reuse, mà tạo một virtual thread mới cho mỗi task. Khi `try-with-resources` kết thúc, `close()` sẽ được gọi và chờ các task đã submit hoàn tất.

## Có nên pool virtual thread không?

Không nên pool virtual thread.

Mục tiêu chính của thread pool là reuse platform thread đắt giá và đồng thời giới hạn concurrency. Bản thân virtual thread không phải resource khan hiếm, việc pool chúng thường không có ý nghĩa, thậm chí còn đưa mô hình “mỗi task một thread” quay lại cách nghĩ cũ.

Nếu mục tiêu thực tế là giới hạn concurrency khi truy cập một resource nào đó, nên giới hạn resource thay vì giới hạn số lượng virtual thread. Ví dụ, một hệ thống cũ chỉ chịu được tối đa 20 request concurrent thì có thể dùng `Semaphore` để kiểm soát concurrency:

```java
import java.util.concurrent.Semaphore;

public class OldServiceClient {
  private static final Semaphore LIMIT = new Semaphore(20);

  public String call() throws InterruptedException {
    LIMIT.acquire();
    try {
      return doCall();
    } finally {
      LIMIT.release();
    }
  }

  private String doCall() {
    return "ok";
  }
}
```

Nếu bottleneck là database connection thì điều chỉnh kích thước connection pool; nếu bottleneck là downstream interface rate limiting thì thực hiện rate limiting, circuit breaker và retry backoff. Virtual thread có thể khiến việc chờ đợi trở nên rẻ hơn, nhưng không thể làm database connection, capacity của downstream, CPU và memory trở nên vô hạn.

## So sánh hiệu năng giữa virtual thread và platform thread

Kết luận trước: virtual thread không phải “thread chạy nhanh hơn”, mà là “thread có thể tạo rất nhiều và có chi phí blocking thấp hơn”. Nó thường có thể cải thiện throughput của service I/O-intensive, nhưng không giảm thời gian của bản thân một database query hoặc một HTTP call.

Ví dụ dưới đây mô phỏng 10,000 task blocking trong 1 giây:

```java
import java.time.Duration;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.stream.IntStream;

public class VirtualThreadCompareDemo {
  public static void main(String[] args) {
    long start = System.currentTimeMillis();

    try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
      IntStream.range(0, 10_000).forEach(i -> {
        executor.submit(() -> {
          Thread.sleep(Duration.ofSeconds(1));
          return i;
        });
      });
    }

    System.out.println("cost: " + (System.currentTimeMillis() - start) + "ms");
  }
}
```

Nếu đổi sang `Executors.newFixedThreadPool(200)`, cùng một thời điểm nhiều nhất chỉ có 200 task đang thực thi, 10,000 task sẽ được xử lý theo từng batch. Ước tính sơ bộ mỗi batch mất 1 giây, tổng thời gian gần 50 giây. Version virtual thread có thể đưa 10,000 task này vào trạng thái chờ gần như đồng thời; platform thread được giải phóng trong lúc chờ, nên tổng thời gian gần với thời gian chờ của một task hơn.

Ví dụ này chỉ cho thấy virtual thread phù hợp với “blocking wait”, không phải benchmark nghiêm ngặt. Service thực tế còn phụ thuộc vào database connection pool, HTTP client connection pool, downstream rate limiting, GC, object allocation, lock contention, container CPU quota và các yếu tố khác.

## Nguyên lý bên trong của virtual thread là gì?

Có thể tách quá trình thực thi virtual thread thành ba phần: scheduling, mount/unmount và quản lý stack.

### Scheduling

Platform thread phụ thuộc vào scheduling của hệ điều hành. Virtual thread được scheduler của JDK scheduling, sau đó do platform thread mang và thực thi. JEP 444 chỉ ra rằng scheduler của virtual thread là một `ForkJoinPool` work-stealing sử dụng chế độ FIFO; nó không phải pool giống common pool được parallel stream sử dụng.

Theo mặc định, mức parallelism của scheduler liên quan đến số processor khả dụng và có thể điều chỉnh bằng các system property sau:

- `jdk.virtualThreadScheduler.parallelism`: parallelism mục tiêu của scheduler.
- `jdk.virtualThreadScheduler.maxPoolSize`: giới hạn platform thread mà scheduler có thể mở rộng.

Phần lớn hệ thống nghiệp vụ không cần thay đổi hai parameter này. Nên ưu tiên kiểm tra connection pool, rate limiting, lock và blocking point; thông thường hiệu quả sẽ cao hơn điều chỉnh parameter của scheduler.

### Mount và unmount

Khi thực thi code Java, virtual thread sẽ mount lên một platform thread. Khi gặp thao tác blocking có hỗ trợ suspend, virtual thread có thể lưu trạng thái thực thi hiện tại rồi unmount, để platform thread tiếp tục phục vụ virtual thread khác.

Các thao tác blocking phổ biến của JDK đã được điều chỉnh cho virtual thread. Ví dụ network I/O, `BlockingQueue`, `Future.get()` khi blocking trong virtual thread thường không chiếm platform thread bên dưới trong thời gian dài.

Không phải mọi blocking đều có thể unmount. Trong JDK 21 đến JDK 23, virtual thread bị blocking trong block hoặc method `synchronized` sẽ gặp Pinning, tức là bị cố định trên carrier thread. [JEP 491](https://openjdk.org/jeps/491) của JDK 24 đã cải thiện điểm này, giúp virtual thread khi blocking trong `synchronized` cũng giải phóng được platform thread bên dưới, loại bỏ phần lớn scenario Pinning do `synchronized` gây ra. Khi gọi native method hoặc code liên quan đến Foreign Function & Memory API, vẫn cần chú ý rủi ro Pinning còn lại.

### Quản lý stack

Platform thread thường sử dụng stack có kích thước cố định của thread hệ điều hành. Stack của virtual thread được lưu dưới dạng stack chunk object trong Java heap và có thể tăng giảm theo quá trình thực thi. Đây cũng là một trong những nguyên nhân quan trọng giúp có thể tạo số lượng lớn virtual thread.

Tuy nhiên, điều này không có nghĩa virtual thread không tốn memory. Mỗi virtual thread vẫn là một object, đồng thời có chi phí memory cho stack chunk, local variable, `ThreadLocal` và các thành phần khác. Hàng triệu virtual thread không miễn phí, chỉ là thực tế hơn nhiều so với hàng triệu platform thread.

## Pinning là gì?

Có thể hiểu Pinning là tình trạng “virtual thread tạm thời không thể unmount khỏi carrier thread”. Sau khi virtual thread bị cố định trên một platform thread, trong thời gian blocking nó sẽ kéo theo việc chiếm thread của hệ điều hành bên dưới, khiến khả năng mở rộng cũng giảm theo.

Trong JDK 21 đến JDK 23, scenario Pinning điển hình nhất là virtual thread thực hiện blocking I/O bên trong block hoặc method `synchronized`.

```java
public synchronized String load() throws IOException {
  return remoteClient.get("/config"); // Trong JDK 21-23, khi blocking ở đây carrier thread có thể bị cố định
}
```

`synchronized` ngắn và chỉ thao tác trên memory thường không đáng ngại. Điều thực sự cần chú ý là việc giữ lock và thực hiện I/O chậm trên đường đi thường xuyên, chẳng hạn truy vấn database, gọi remote interface hoặc đọc file lớn trong khi đang giữ object lock.

Nếu sử dụng JDK 21 đến JDK 23, có thể cân nhắc:

- Tránh thực hiện I/O chậm bên trong `synchronized`.
- Với scenario lock blocking thường xuyên, sử dụng `ReentrantLock` và release lock trong `try/finally`.
- Dùng JFR để quan sát event `jdk.VirtualThreadPinned`.
- Tạm thời dùng `-Djdk.tracePinnedThreads=full` để định vị call stack bị cố định.

Nếu sử dụng JDK 24 hoặc cao hơn, vấn đề Pinning chính do `synchronized` gây ra đã được JEP 491 giải quyết. Việc chọn `synchronized` hay `java.util.concurrent.locks` có thể quay lại đánh giá dựa trên semantics của code, khả năng maintain và năng lực của lock.

## Các lưu ý khi sử dụng virtual thread

### Đừng xem virtual thread là công cụ tăng tốc CPU

Virtual thread cải thiện khả năng tiếp nhận concurrency của task dạng chờ đợi. CPU-intensive task cuối cùng vẫn phải tranh chấp CPU time slice; dù số lượng virtual thread có nhiều đến đâu cũng không thể vượt qua giới hạn vật lý của số core CPU.

### Đừng dùng tư duy thread pool để giới hạn virtual thread

Không nên tạo virtual thread pool có số lượng cố định. Khi cần giới hạn concurrency, hãy dùng `Semaphore`, connection pool, rate limiter hoặc giới hạn capacity của queue để giới hạn resource cụ thể.

### Cẩn thận khi `ThreadLocal` cache object lớn

Bản Java 21 chính thức đảm bảo virtual thread hỗ trợ `ThreadLocal`, điều này có lợi cho việc tương thích với code cũ và framework. Tuy nhiên, không nên dùng `ThreadLocal` để cache object lớn trong mỗi virtual thread.

Trước đây trong thread pool, một `ThreadLocal<SimpleDateFormat>` có thể chỉ tương ứng với vài chục hoặc vài trăm platform thread. Sau khi migrate sang virtual thread, nếu mỗi task có một virtual thread, cách viết tương tự có thể biến thành mỗi task tạo một cache object, khiến áp lực memory và allocation tăng lên.

Nếu chỉ truyền request context, user ID hoặc Trace ID thì thường không có vấn đề lớn. Nếu cache database connection, array lớn, formatter phức tạp hoặc client object thì cần đánh giá lại. JDK 25 đưa Scoped Values thành tính năng chính thức thông qua JEP 506, phù hợp hơn để truyền context immutable giữa một lượng lớn virtual thread.

### Virtual thread không loại bỏ vấn đề thread safety

Virtual thread khiến việc tạo thread rẻ hơn, đồng nghĩa bạn dễ chạy đồng thời một lượng lớn task concurrent hơn. Data race trước đây chưa lộ ra vì thread pool nhỏ có thể dễ xuất hiện hơn sau khi chuyển sang virtual thread.

Vẫn cần tuân thủ các nguyên tắc cơ bản của concurrent programming: shared mutable state phải được lock hoặc isolate; database connection, session object và client không thread-safe không được nhiều virtual thread sử dụng đồng thời một cách tùy tiện.

### Chú ý connection pool và capacity của downstream

Sau khi nhiều service migrate sang virtual thread, bottleneck đầu tiên thường không còn là business thread pool mà là database connection pool, HTTP connection pool, số lượng Redis connection hoặc downstream rate limiting.

Đây không phải vấn đề của virtual thread. Virtual thread chỉ giúp nhiều task có cơ hội tiến hành đồng thời hơn; shared resource thực sự vẫn phải được quản lý theo capacity. Khi load test nên đồng thời theo dõi:

- QPS, response time và error rate của application.
- Active connection, wait queue và số lần timeout của database connection pool.
- HTTP client connection pool và downstream 429/5xx.
- CPU, heap memory, GC và tốc độ object allocation.
- Event virtual thread và lock contention trong JFR.

### Đừng trộn quá nhiều mô hình asynchronous

Virtual thread phù hợp nhất với code synchronous blocking. Với hệ thống đã được viết asynchronous end-to-end bằng Reactive/WebFlux/Netty, việc bật virtual thread chưa chắc mang lại lợi ích rõ rệt.

Việc trộn các model còn phức tạp hơn: outer layer là virtual thread, inner layer lại dùng nhiều async callback và thread pool; khi điều tra sự cố có thể phải đồng thời đối mặt với virtual thread, event loop, business thread pool và connection pool, tức nhiều loại context. Khi migrate, tốt hơn nên thử nghiệm trước một blocking chain synchronous, thay vì thay thế toàn bộ hệ thống bằng một lần.

## Bật virtual thread trong Spring Boot như thế nào?

Spring Boot bắt đầu cung cấp công tắc tương đối trực tiếp từ phiên bản 3.2. Khi sử dụng Java 21 hoặc cao hơn, có thể bật trong configuration:

```properties
spring.threads.virtual.enabled=true
```

Tài liệu chính thức của Spring Boot cũng nêu một số điểm thực tiễn:

- Sau khi bật virtual thread, một số property cấu hình kích thước thread pool truyền thống không còn có hiệu lực theo cách cũ, vì scheduling của virtual thread phụ thuộc vào platform thread pool trong phạm vi JVM.
- Virtual thread là daemon thread. Nếu application cần các background task như `@Scheduled` để giữ JVM sống, nên thiết lập `spring.main.keep-alive=true`.
- Hiện tại Spring Boot khuyến nghị Java 24 hoặc cao hơn để có trải nghiệm virtual thread tốt hơn, chủ yếu nhờ các cải tiến về Pinning.

Một configuration đơn giản như sau:

```yaml
spring:
  threads:
    virtual:
      enabled: true
  main:
    keep-alive: true
```

Sau khi bật, không có nghĩa mọi API đều chạy nhanh hơn. Nó có khả năng cải thiện nhiều hơn với API synchronous blocking, chờ I/O rõ rệt và concurrency cao. Nếu API chủ yếu tốn thời gian ở CPU, lock contention, bản thân slow SQL hoặc downstream rate limiting, virtual thread chỉ khiến vấn đề lộ ra sớm hơn.

## Điều tra vấn đề virtual thread như thế nào?

JDK đã bổ sung khá nhiều năng lực quan sát cho virtual thread.

### Dùng `jcmd` để export thread dump

`jstack` truyền thống không thật sự phù hợp khi phải xử lý hàng nghìn virtual thread. JDK cung cấp năng lực thread dump mới:

```bash
jcmd <pid> Thread.dump_to_file -format=json thread-dump.json
```

Cũng có thể export ở dạng text:

```bash
jcmd <pid> Thread.dump_to_file -format=text thread-dump.txt
```

Định dạng JSON phù hợp hơn cho tool phân tích, đặc biệt khi có rất nhiều virtual thread.

### Dùng JFR để quan sát event virtual thread

Các event liên quan đến virtual thread trong JFR gồm:

- `jdk.VirtualThreadStart`
- `jdk.VirtualThreadEnd`
- `jdk.VirtualThreadPinned`
- `jdk.VirtualThreadSubmitFailed`

Trong đó `jdk.VirtualThreadPinned` rất hữu ích khi điều tra Pinning. Sau JDK 24, phần lớn Pinning liên quan đến `synchronized` đã được giải quyết, nhưng các scenario còn lại như native/FFM vẫn có thể được quan sát qua JFR.

### Tạm thời bật Pinning stack trace

Trong JDK 21 đến JDK 23, có thể tạm thời dùng:

```bash
-Djdk.tracePinnedThreads=full
```

Khi virtual thread blocking và bị cố định, nó sẽ in call stack, phù hợp để định vị vấn đề trong môi trường local hoặc test. Sau JEP 491 của JDK 24, các scenario Pinning chính liên quan đến `synchronized` đã được cải thiện; với boundary còn lại như native/FFM, vẫn nên kết hợp JFR và thread dump để đánh giá.

## Các câu hỏi phỏng vấn thường gặp về virtual thread

### Virtual thread khác platform thread như thế nào?

Platform thread thường tương ứng một-một với thread của hệ điều hành, có chi phí tạo và context switch cao hơn, đồng thời số lượng có hạn. Virtual thread do JDK scheduling, có thể ánh xạ một lượng lớn virtual thread vào số ít platform thread. Khi blocking chờ I/O, virtual thread thường có thể unmount khỏi carrier thread, để platform thread tiếp tục thực thi virtual thread khác.

### Vì sao virtual thread phù hợp với task I/O-intensive?

Task I/O-intensive dành phần lớn thời gian chờ resource bên ngoài. Khi platform thread chờ, nó chiếm thread của hệ điều hành; khi virtual thread chờ, nó có thể tự suspend và giải phóng carrier thread. Nhờ vậy, cùng số lượng platform thread có thể đảm nhận nhiều task concurrent hơn.

### Virtual thread có phù hợp với task CPU-intensive không?

Không nên xem nó là công cụ tăng tốc CPU. Task CPU-intensive cần CPU time thực tế; sau khi số thread vượt số core, nó chỉ làm tăng tranh chấp scheduling. Virtual thread có thể cải thiện việc chờ trong concurrency cao, nhưng không làm một computation task đơn lẻ chạy nhanh hơn.

### Virtual thread có cần pool không?

Không cần và cũng không nên. Virtual thread rẻ, nên được tạo theo task. Khi cần giới hạn concurrency, hãy giới hạn resource cụ thể như database connection pool, HTTP connection pool, `Semaphore` hoặc rate limiter, thay vì pool virtual thread.

### Virtual thread có giống coroutine không?

Cả hai đều thuộc cách tiếp cận lightweight concurrency, nhưng Java virtual thread là implementation của `java.lang.Thread` và được đưa vào thread model vốn có của Java. Code nghiệp vụ không cần viết `async/await` và cũng không cần tự gọi yield. So với Go goroutine, virtual thread nhấn mạnh hơn vào khả năng tương thích với thread API, debugging tool và phong cách code blocking sẵn có của Java. Với developer, nó giống “thread rẻ hơn rất nhiều”.

### Sau khi dùng virtual thread có còn cần Reactive programming không?

Tùy scenario. Nhiều API server synchronous blocking có thể dùng virtual thread để có khả năng đọc tốt hơn và throughput đủ cao, không cần viết callback chain phức tạp chỉ để giải phóng thread. Nhưng Reactive vẫn phù hợp với stream processing, backpressure, event-driven, long connection và hệ thống đã asynchronous end-to-end. Virtual thread không phải silver bullet thay thế mọi model asynchronous.

### `synchronized` trong JDK 21 còn dùng được không?

Có thể dùng, nhưng cần chú ý boundary. Trong JDK 21 đến JDK 23, virtual thread thực hiện blocking operation bên trong `synchronized` có thể gặp Pinning. Với synchronization trên memory ngắn, vấn đề không lớn; trên đường đi thường xuyên, không nên giữ lock `synchronized` để thực hiện I/O chậm. Sau cải tiến của JEP 491 trong JDK 24, Pinning liên quan đến `synchronized` về cơ bản đã được giải quyết.

## Tài liệu tham khảo

- [JEP 444: Virtual Threads](https://openjdk.org/jeps/444)
- [Oracle Java 21 Documentation: Virtual Threads](https://docs.oracle.com/en/java/javase/21/core/virtual-threads.html)
- [JEP 491: Synchronize Virtual Threads without Pinning](https://openjdk.org/jeps/491)
- [JEP 506: Scoped Values](https://openjdk.org/jeps/506)
- [Spring Boot Reference Documentation: Virtual threads](https://docs.spring.io/spring-boot/reference/features/spring-application.html#features.spring-application.virtual-threads)
- [Spring Blog: Embracing Virtual Threads](https://spring.io/blog/2022/10/11/embracing-virtual-threads/)
- [Inside Java: Managing Throughput with Virtual Threads](https://inside.java/2024/02/04/sip094/)
- [Quarkus Blog: When Quarkus meets Virtual Threads](https://quarkus.io/blog/virtual-thread-1/)

<!-- @include: @article-footer.snippet.md -->
