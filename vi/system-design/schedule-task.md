---
title: Giải thích chi tiết về tác vụ định kỳ trong Java
category: System Design
icon: "mdi:clock-outline"
description: "Giải thích có hệ thống về tác vụ định kỳ và tác vụ trễ trong Java: Timer, ScheduledThreadPoolExecutor, DelayQueue, time wheel, Spring @Scheduled (biểu thức Cron), cùng so sánh lựa chọn và trường hợp sử dụng của các framework lập lịch tác vụ phân tán như Quartz, XXL-JOB, ElasticJob, PowerJob (hủy đơn hàng quá hạn, backup định kỳ, thu thập dữ liệu định kỳ)."
head:
  - - meta
    - name: keywords
      content: "tác vụ định kỳ,Quartz,Elastic-Job,XXL-JOB,PowerJob"
---

## Vì sao cần tác vụ định kỳ?

Hãy xem một số trường hợp nghiệp vụ rất thường gặp:

1. Một hệ thống cần backup dữ liệu lúc 1 giờ sáng.
2. Trên một nền tảng thương mại điện tử, nếu người dùng chưa thanh toán sau nửa giờ kể từ khi đặt hàng thì cần tự động hủy đơn hàng.
3. Một nền tảng tổng hợp nội dung cần thu thập dữ liệu động từ một website nào đó sau mỗi 10 phút để sử dụng cho mục đích riêng.
4. Một nền tảng blog hỗ trợ gửi bài viết theo lịch.
5. Một nền tảng quỹ cần định kỳ tính lợi nhuận trong ngày của người dùng mỗi tối và đẩy dữ liệu mới nhất cho họ.
6. …

Những trường hợp này thường yêu cầu thực hiện một việc tại một thời điểm cụ thể, tức là thực hiện một việc theo lịch hoặc sau một khoảng trễ.

- Tác vụ định kỳ: thực hiện một tác vụ cụ thể tại thời điểm được chỉ định, chẳng hạn 8 giờ sáng mỗi ngày, 3 giờ chiều thứ Hai hằng tuần. Tác vụ định kỳ có thể dùng để thực hiện các công việc lặp lại như backup dữ liệu, dọn dẹp log, tạo báo cáo.
- Tác vụ trễ: thực hiện một tác vụ cụ thể sau một khoảng trễ nhất định, chẳng hạn sau 10 phút, sau 3 giờ. Tác vụ trễ có thể dùng cho các công việc bất đồng bộ như hủy đơn hàng, đẩy thông báo, thu hồi red packet.

Dù phạm vi áp dụng của hai loại có khác nhau, tư tưởng cốt lõi đều là sắp xếp thời điểm thực hiện tác vụ vào một thời điểm nào đó trong tương lai để đạt hiệu quả lập lịch mong muốn.

## Tác vụ định kỳ trên một máy

### Timer

`java.util.Timer` là một cách triển khai tác vụ định kỳ đã được JDK hỗ trợ từ JDK 1.3.

Bên trong, `Timer` sử dụng một class có tên `TaskQueue` để lưu trữ các tác vụ định kỳ. Đây là một priority queue được triển khai dựa trên min-heap. `TaskQueue` sắp xếp các tác vụ theo khoảng cách đến thời điểm thực hiện tiếp theo, bảo đảm tác vụ ở đỉnh heap được thực hiện trước. Vì vậy, khi cần thực hiện tác vụ, mỗi lần chỉ cần lấy tác vụ ở đỉnh heap ra để chạy!

Cách sử dụng `Timer` khá đơn giản. Với cách dưới đây, ta có thể tạo một tác vụ định kỳ được thực hiện sau 1 giây.

```java
// Code ví dụ:
TimerTask task = new TimerTask() {
    public void run() {
        System.out.println("Thời gian hiện tại: " + new Date() + "\n" +
                "Tên thread: " + Thread.currentThread().getName());
    }
};
System.out.println("Thời gian hiện tại: " + new Date() + "\n" +
        "Tên thread: " + Thread.currentThread().getName());
Timer timer = new Timer("Timer");
long delay = 1000L;
timer.schedule(task, delay);


// Output:
Thời gian hiện tại: Fri May 28 15:18:47 CST 2021
Tên thread: main
Thời gian hiện tại: Fri May 28 15:18:48 CST 2021
Tên thread: Timer
```

Tuy nhiên, nó có khá nhiều nhược điểm. Mỗi `Timer` chỉ sử dụng một thread nền để thực hiện tuần tự tất cả tác vụ, nên nếu một tác vụ chạy quá lâu, các tác vụ khác sẽ bị trì hoãn. Nếu `TimerTask#run()` ném ra runtime exception hoặc error không được bắt, thread thực thi duy nhất của `Timer` sẽ kết thúc và các tác vụ sau đó cũng không thể tiếp tục được lập lịch. Không thể mô tả chính xác hành vi này chỉ bằng câu “`Timer` chỉ bắt `InterruptedException`”.

`Timer` có một đoạn chú thích như sau:

```JAVA
 * This class does not offer real-time guarantees: it schedules
 * tasks using the <tt>Object.wait(long)</tt> method.
 *Java 5.0 introduced the {@code java.util.concurrent} package and
 * one of the concurrency utilities therein is the {@link
 * java.util.concurrent.ScheduledThreadPoolExecutor
 * ScheduledThreadPoolExecutor} which is a thread pool for repeatedly
 * executing tasks at a given rate or delay.  It is effectively a more
 * versatile replacement for the {@code Timer}/{@code TimerTask}
 * combination, as it allows multiple service threads, accepts various
 * time units, and doesn't require subclassing {@code TimerTask} (just
 * implement {@code Runnable}).  Configuring {@code
 * ScheduledThreadPoolExecutor} with one thread makes it equivalent to
 * {@code Timer}.
```

Ý chính là: `ScheduledThreadPoolExecutor` hỗ trợ thực hiện tác vụ định kỳ bằng nhiều thread và có chức năng mạnh hơn, là lựa chọn thay thế cho `Timer`.

### ScheduledExecutorService

`ScheduledExecutorService` là một interface, có nhiều class triển khai. Class thường được sử dụng là `ScheduledThreadPoolExecutor`.

![](https://oss.javaguide.cn/javaguide/20210607154324712.png)

Bản thân `ScheduledThreadPoolExecutor` là một thread pool, hỗ trợ thực hiện tác vụ đồng thời. Bên trong, nó sử dụng `DelayedWorkQueue` làm task queue.

```java
// Code ví dụ:
TimerTask repeatedTask = new TimerTask() {
    @SneakyThrows
    public void run() {
        System.out.println("Thời gian hiện tại: " + new Date() + "\n" +
                "Tên thread: " + Thread.currentThread().getName());
    }
};
System.out.println("Thời gian hiện tại: " + new Date() + "\n" +
        "Tên thread: " + Thread.currentThread().getName());
ScheduledExecutorService executor = Executors.newScheduledThreadPool(3);
long delay  = 1000L;
long period = 1000L;
executor.scheduleAtFixedRate(repeatedTask, delay, period, TimeUnit.MILLISECONDS);
Thread.sleep(delay + period * 5);
executor.shutdown();
// Output:
Thời gian hiện tại: Fri May 28 15:40:46 CST 2021
Tên thread: main
Thời gian hiện tại: Fri May 28 15:40:47 CST 2021
Tên thread: pool-1-thread-1
Thời gian hiện tại: Fri May 28 15:40:48 CST 2021
Tên thread: pool-1-thread-1
Thời gian hiện tại: Fri May 28 15:40:49 CST 2021
Tên thread: pool-1-thread-2
Thời gian hiện tại: Fri May 28 15:40:50 CST 2021
Tên thread: pool-1-thread-2
Thời gian hiện tại: Fri May 28 15:40:51 CST 2021
Tên thread: pool-1-thread-2
Thời gian hiện tại: Fri May 28 15:40:52 CST 2021
Tên thread: pool-1-thread-2
```

Dù sử dụng `Timer` hay `ScheduledExecutorService`, ta đều không thể dùng biểu thức Cron để chỉ định thời điểm cụ thể thực hiện tác vụ.

### DelayQueue

`DelayQueue` là delay queue được JUC (`java.util.concurrent)`) cung cấp, dùng để triển khai tác vụ trễ, chẳng hạn hủy đơn hàng ngay nếu sau 15 phút kể từ khi đặt hàng mà chưa thanh toán. Nó là một loại `BlockingQueue`, bên dưới là một unbounded queue được triển khai dựa trên `PriorityQueue` và có tính thread-safe. Có thể tham khảo bài viết [Phân tích source code PriorityQueue](https://javaguide.cn/java/collection/priorityqueue-source-code.html) do tác giả viết.

![Các class triển khai của BlockingQueue](https://oss.javaguide.cn/github/javaguide/java/collection/blocking-queue-hierarchy.png)

`DelayQueue` và `Timer/TimerTask` đều có thể làm nền tảng cho việc lập lịch trễ. `DelayQueue` dùng priority queue để quản lý các phần tử triển khai interface `Delayed`; chỉ những phần tử đã hết thời gian trễ mới có thể được lấy ra, nhưng bản thân nó không chịu trách nhiệm tạo thread để thực hiện tác vụ. Thông thường vẫn cần viết consumer loop và chọn executor phù hợp. `Timer` thì có sẵn một thread thực thi. Cả hai đều cho phép tiếp tục thêm tác vụ sau khi tạo, đồng thời hỗ trợ hủy hoặc xóa tác vụ. Nhận định “`Timer` chỉ có thể chỉ định tác vụ khi khởi tạo” là không đúng.

Để biết chi tiết về `DelayQueue`, hãy tham khảo bài viết [Phân tích source code `DelayQueue`](https://javaguide.cn/java/collection/delayqueue-source-code.html).

### Spring Task

Ta chỉ cần dùng annotation `@Scheduled` do Spring cung cấp để định nghĩa tác vụ định kỳ, rất tiện lợi!

```java
/**
 * cron: sử dụng biểu thức Cron. Chạy ở giây 1 và 2 của mỗi phút
 */
@Scheduled(cron = "1-2 * * * * ? ")
public void reportCurrentTimeWithCronExpression() {
  log.info("Cron Expression: The time is now {}", dateFormat.format(new Date()));
}

```

Khi còn học đại học, một dự án enterprise sử dụng SSM mà tôi thực hiện cũng dùng Spring Task để làm tác vụ định kỳ.

Ngoài ra, Spring Task còn hỗ trợ **biểu thức Cron**. Biểu thức Cron chủ yếu được dùng trong hệ thống job định kỳ (tác vụ định kỳ) để định nghĩa thời điểm hoặc tần suất thực hiện, rất mạnh. Bạn có thể dùng biểu thức Cron để thiết lập thời điểm thực hiện tác vụ định kỳ mỗi ngày hoặc mỗi tháng. Nếu học về tác vụ định kỳ, nhất định phải chú ý đến biểu thức Cron. Có thể tham khảo một [trình tạo biểu thức Cron trực tuyến](http://cron.qqe2.com/) .

Tuy nhiên, cơ chế lập lịch có sẵn của Spring chỉ hỗ trợ một máy và chức năng tương đối đơn giản. Trước đây tôi đã viết bài ["Tìm hiểu Schedule Tasks trong Spring Boot trong 5 phút"](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485563&idx=1&sn=7419341f04036a10b141b74624a3f8c9&chksm=cea247b0f9d5cea6440759e6d49b4e77d06f4c99470243a10c1463834e873ca90266413fbc92&token=2133161636&lang=zh_CN#rd), bạn có thể tham khảo nếu chưa rõ.

Spring cung cấp abstraction lập lịch thông qua `TaskScheduler`, không cố định chỉ một cách triển khai bên dưới. `ThreadPoolTaskScheduler` thường ủy thác cho `ScheduledExecutorService`; trong các môi trường như Jakarta EE cũng có thể sử dụng scheduler do container quản lý. Bản thân `@Scheduled` không cung cấp cơ chế điều phối cluster; khi một ứng dụng được deploy trên nhiều instance, mỗi instance đều có thể kích hoạt tác vụ.

**Tổng hợp ưu / nhược điểm:**

- Ưu điểm: đơn giản, nhẹ, hỗ trợ biểu thức Cron
- Nhược điểm: chức năng đơn giản

### Time wheel

Kafka, Dubbo, ZooKeeper, Netty, Caffeine, Akka đều có triển khai time wheel.

Nói đơn giản, time wheel là một queue dạng vòng (bên dưới thường được triển khai dựa trên array), mỗi phần tử (time slot) trong queue đều có thể lưu một danh sách tác vụ định kỳ.

Mỗi time slot trong time wheel đại diện cho khoảng thời gian cơ bản, hay còn gọi là độ chính xác thời gian của time wheel. Giả sử thời gian tiến thêm một time slot mỗi giây, độ chính xác cao nhất của time wheel là 1 giây (tức là 3 s và 3.9 s sẽ nằm trong cùng một time slot).

Hình dưới là một time wheel có 12 time slot, quay hết một vòng cần 12 s. Khi cần tạo tác vụ định kỳ thực hiện sau 3 s, chỉ cần đặt tác vụ vào time slot có index 3. Khi cần tạo tác vụ định kỳ thực hiện sau 9 s, chỉ cần đặt tác vụ vào time slot có index 9.

![](https://oss.javaguide.cn/github/javaguide/system-design/schedule-task/one-layers-of-time-wheel.png)

Vậy nếu cần tạo tác vụ định kỳ thực hiện sau 13 s thì sao? Khi đó có thể đưa vào khái niệm **số vòng / số round**. Tác vụ vẫn nằm ở time slot có index 1, đồng thời ghi lại số round còn lại cần chờ; chỉ sau khi đi hết một vòng và thêm 1 s thì tác vụ mới được thực hiện. Quy ước về việc “time slot hiện tại có được tính vào số round hay không” có thể khác nhau giữa các cách triển khai, không nên tách khỏi cách triển khai cụ thể để cố định thành 2 vòng.

Ngoài cách tăng số round, còn có một loại **time wheel nhiều tầng** (giống đồng hồ đeo tay), và Kafka sử dụng phương án này.

Hãy xem ví dụ về time wheel trong hình dưới để dễ hiểu hơn.

![](https://oss.javaguide.cn/github/javaguide/system-design/schedule-task/three-layers-of-time-wheel.png)

Time wheel trong hình trên (ms -> s) có độ chính xác thời gian ở tầng 1 là 1, tầng 2 là 20, tầng 3 là 400. Giả sử cần thêm tác vụ A thực hiện sau 350 s (thời điểm hiện tại là 0 s), tác vụ này sẽ được đặt ở tầng 2 (vì khoảng thời gian của tầng 2 là 20\*20=400>350), tại time slot thứ 350/20=17.

Sau khi tầng 1 quay 17 vòng, 340 s đã trôi qua, lúc này con trỏ của tầng 2 đi đến time slot thứ 17. Khi đó, các tác vụ trong time slot thứ 17 của tầng 2 sẽ được chuyển xuống tầng 1.

Lúc này tác vụ A sẽ được thực hiện sau 10 s, nên nó sẽ được chuyển đến time slot thứ 10 của tầng 1.

Việc di chuyển giữa các tầng còn được gọi là nâng cấp / hạ cấp của time wheel. Chỉ cần hình dung theo đồng hồ đeo tay là được!

**Time wheel khá phù hợp với trường hợp cần quản lý số lượng lớn timer. Trong cách triển khai điển hình với số tầng và số slot mỗi tầng có giới hạn, chi phí xác định slot, chèn và tiến con trỏ thường có thể gần O(1); nhưng chi phí thực thi thực tế tại một tick còn phụ thuộc vào số tác vụ hết hạn, việc di chuyển dây chuyền và cách triển khai cụ thể.**

## Tác vụ định kỳ phân tán

### Redis

Redis có thể được dùng để làm tác vụ trễ. Về cơ bản, có hai phương án triển khai chức năng tác vụ trễ dựa trên Redis:

1. Lắng nghe sự kiện hết hạn của Redis
2. Delay queue tích hợp của Redisson

Tôi đã trình bày chi tiết phần này trong bài [Thiết kế hệ thống và câu hỏi tình huống thường gặp trong phỏng vấn backend](https://javaguide.cn/zhuanlan/back-end-interview-high-frequency-system-design-and-scenario-questions.html). Nếu cần, bạn có thể vào Xingqiu để đọc và học. Vì nội dung khá dài nên ở đây không lặp lại.

![Thiết kế hệ thống và câu hỏi tình huống thường gặp trong phỏng vấn backend](https://oss.javaguide.cn/xingqiu/back-end-interview-high-frequency-system-design-and-scenario-questions-fengmian.png)

### MQ

Phần lớn message queue, chẳng hạn RocketMQ, RabbitMQ, đều hỗ trợ message định kỳ / message trễ. Về bản chất, message định kỳ và message trễ giống nhau: server dựa trên thời điểm định kỳ được thiết lập cho message để chuyển message cho consumer xử lý tại một thời điểm cố định.

Tuy nhiên, trước khi sử dụng message định kỳ của MQ, nhất định phải xem rõ giới hạn của sản phẩm và version cụ thể. Chẳng hạn, implementation chính thức của RocketMQ 4.x cung cấp 18 mức trễ cố định, tối đa 2 giờ; RocketMQ 5.x chuyển sang thiết lập thời điểm gửi bằng Unix timestamp ở mức millisecond, mặc định cho phép phạm vi định kỳ tối đa 24 giờ. Không thể gộp cơ chế của hai version này thành cùng một giới hạn.

**Tổng hợp ưu / nhược điểm:**

- **Ưu điểm**: có thể tích hợp với Spring, hỗ trợ phân tán, hỗ trợ cluster, performance khá tốt
- **Nhược điểm**: chức năng hạn chế, thiếu linh hoạt, cần bảo đảm độ tin cậy của message

## Framework lập lịch tác vụ phân tán

Nếu cần các tính năng nâng cao như hỗ trợ sharding tác vụ và high availability trong môi trường phân tán, ta cần dùng framework lập lịch tác vụ phân tán.

Thông thường, việc thực hiện một tác vụ định kỳ phân tán thường liên quan đến các vai trò sau:

- **Tác vụ**: trước hết chắc chắn phải có tác vụ cần thực hiện. Tác vụ này là logic nghiệp vụ cụ thể, chẳng hạn gửi bài viết theo lịch.
- **Scheduler**: tiếp theo là scheduling center. Scheduling center chủ yếu chịu trách nhiệm quản lý tác vụ và phân phối tác vụ cho executor.
- **Executor**: cuối cùng là executor. Executor nhận tác vụ do scheduler phân công và thực hiện.

### Quartz

Một framework lập lịch tác vụ mã nguồn mở rất phổ biến, được viết hoàn toàn bằng Java. Có thể nói Quartz là framework đi đầu hoặc tiêu chuẩn tham khảo trong lĩnh vực tác vụ định kỳ Java. Các framework lập lịch tác vụ khác về cơ bản đều được phát triển dựa trên Quartz, chẳng hạn `elastic-job` của Dangdang.com là một giải pháp lập lịch phân tán được phát triển mở rộng từ Quartz.

Sử dụng Quartz có thể dễ dàng tích hợp với Spring, đồng thời hỗ trợ thêm tác vụ động và cluster. Tuy nhiên, Quartz cũng khá khó sử dụng, API rườm rà.

Ngoài ra, Quartz không có sẵn UI quản trị, nhưng có thể dùng project mã nguồn mở [quartzui](https://github.com/zhaopeiym/quartzui) để giải quyết vấn đề này.

Quartz cũng hỗ trợ tác vụ phân tán. Tuy nhiên, nó thực hiện ở tầng database thông qua cơ chế lock của database, có nhiều nhược điểm như mức độ xâm nhập hệ thống cao, tải giữa các node không cân bằng. Có phần giống một hệ thống phân tán giả.

**Tổng hợp ưu / nhược điểm:**

- Ưu điểm: có thể tích hợp với Spring, đồng thời hỗ trợ thêm tác vụ động và cluster.
- Nhược điểm: hỗ trợ phân tán không tốt, không hỗ trợ quản lý tác vụ trực quan, khó sử dụng (so với các framework cùng loại khác)

### Elastic-Job

ElasticJob ban đầu do Dangdang.com phát hành mã nguồn mở. Trong lịch sử, nó từng được chia thành hai subproject là ElasticJob-Lite và ElasticJob-Cloud. Cách phân loại này và sự phụ thuộc vào Mesos của ElasticJob-Cloud thuộc bối cảnh version cũ, không nên dùng làm bảng lựa chọn hiện tại. Hiện nay, tài liệu chính thức mô tả ElasticJob là một giải pháp nhẹ, phi tập trung, cung cấp sharding tác vụ phân tán; registry center hỗ trợ ZooKeeper và etcd.

`ElasticJob` hỗ trợ sharding và high availability cho tác vụ trong môi trường phân tán, cùng các chức năng như quản lý tác vụ trực quan.

![](https://oss.javaguide.cn/github/javaguide/system-design/schedule-task/elasticjob-feature-list.png)

Dưới đây là sơ đồ kiến trúc của ElasticJob-Lite thời kỳ đầu, sử dụng ZooKeeper làm registry center, dùng để tìm hiểu tư tưởng lập lịch phi tập trung; version hiện tại còn hỗ trợ etcd, không thể xem các component trong hình là cách deploy duy nhất.

![Thiết kế kiến trúc của ElasticJob-Lite](https://oss.javaguide.cn/github/javaguide/system-design/schedule-task/elasticjob-lite-architecture-design.png)

Trong cách deploy này, ElasticJob không thiết lập scheduling service tập trung mà sử dụng ZooKeeper để điều phối sharding tác vụ trên các node. Version hiện tại cũng có thể chọn etcd làm registry center.

Trong Elastic-Job, việc lập lịch định kỳ đều do executor tự kích hoạt. Thiết kế này còn được gọi là thiết kế phi tập trung (việc lập lịch và xử lý đều do từng executor tự hoàn thành).

Spring Boot Starter chính thức hiện tại đăng ký tác vụ thông qua Spring Bean và file cấu hình, không cung cấp annotation `@ElasticJobConf`. Dưới đây lấy ZooKeeper làm registry center:

```java
@Component
public class TestJob implements SimpleJob {
    @Override
    public void execute(ShardingContext context) {
        System.out.printf("Tên tác vụ: %s, tổng số shard: %d, tham số shard hiện tại: %s%n",
                context.getJobName(),
                context.getShardingTotalCount(),
                context.getShardingParameter());
    }
}
```

```yaml
elasticjob:
  regCenter:
    serverLists: localhost:2181
    namespace: elasticjob-demo
  jobs:
    dayJob:
      elasticJobClass: com.example.job.TestJob
      cron: 0/10 * * * * ?
      shardingTotalCount: 2
      shardingItemParameters: 0=AAAA,1=BBBB
```

Thuộc tính cấu hình và loại registry center sẽ thay đổi theo sự phát triển của version. Khi tích hợp, cần lấy [tài liệu Spring Boot Starter](https://shardingsphere.apache.org/elasticjob/current/en/user-manual/usage/job-api/spring-boot-starter/) và [tài liệu registry center](https://shardingsphere.apache.org/elasticjob/current/en/user-manual/configuration/registry-center/) của version đang sử dụng làm chuẩn.

**Địa chỉ liên quan:**

- Địa chỉ GitHub: <https://github.com/apache/shardingsphere-elasticjob>.
- Website chính thức: <https://shardingsphere.apache.org/elasticjob/index_zh.html> .

**Tổng hợp ưu / nhược điểm:**

- Ưu điểm: có thể tích hợp với Spring, hỗ trợ phân tán, hỗ trợ cluster, performance khá tốt, hỗ trợ quản lý tác vụ trực quan
- Nhược điểm: cần deploy thêm registry center như ZooKeeper hoặc etcd, làm tăng độ phức tạp và chi phí bảo trì hệ thống

### XXL-JOB

`XXL-JOB` được phát hành mã nguồn mở vào năm 2015, là một framework lập lịch tác vụ phân tán nhẹ chất lượng tốt, hỗ trợ quản lý tác vụ trực quan, scale out / scale in linh hoạt, retry và cảnh báo khi tác vụ thất bại, sharding tác vụ, v.v.

![](https://oss.javaguide.cn/github/javaguide/system-design/schedule-task/xxljob-feature-list.png)

Theo giới thiệu trên website chính thức của `XXL-JOB`, nó đã giải quyết nhiều nhược điểm của Quartz.

> Quartz là framework nổi bật trong lĩnh vực lập lịch job mã nguồn mở và là lựa chọn hàng đầu cho việc lập lịch job. Tuy nhiên, trong môi trường cluster, Quartz sử dụng API để quản lý tác vụ, từ đó có thể tránh các vấn đề trên, nhưng vẫn tồn tại các vấn đề sau:
>
> - Vấn đề 1: thao tác với tác vụ bằng API không thân thiện;
> - Vấn đề 2: cần persist QuartzJobBean nghiệp vụ vào bảng dữ liệu bên dưới, mức độ xâm nhập hệ thống khá nghiêm trọng.
> - Vấn đề 3: logic lập lịch và QuartzJobBean bị gắn vào cùng một project. Khi số lượng tác vụ lập lịch tăng dần và logic của tác vụ lập lịch ngày càng nặng, performance của hệ thống lập lịch sẽ bị giới hạn rất nhiều bởi nghiệp vụ;
> - Vấn đề 4: tầng bên dưới của Quartz lấy DB lock theo kiểu “chiếm dụng” và để node chiếm được lock chịu trách nhiệm chạy tác vụ, dẫn đến tải giữa các node chênh lệch rất lớn; trong khi XXL-JOB chạy tác vụ theo kiểu “phân phối phối hợp” thông qua executor, tận dụng đầy đủ ưu thế của cluster và cân bằng tải giữa các node.
>
> XXL-JOB đã khắc phục các nhược điểm trên của Quartz.

Thiết kế kiến trúc của `XXL-JOB` như hình dưới:

![](https://oss.javaguide.cn/github/javaguide/system-design/schedule-task/xxljob-architecture-design-v2.1.0.png)

Như hình trên, `XXL-JOB` gồm hai phần chính là **scheduling center** và **executor**. Scheduling center chủ yếu chịu trách nhiệm quản lý tác vụ, quản lý executor và quản lý log. Executor chủ yếu nhận tín hiệu lập lịch và xử lý. Ngoài ra, khi scheduling center lập lịch tác vụ, nó sử dụng RPC tự phát triển để thực hiện.

Khác với thiết kế phi tập trung của Elastic-Job, thiết kế này của `XXL-JOB` còn được gọi là thiết kế tập trung (scheduling center lập lịch cho nhiều executor thực hiện tác vụ).

Tương tự `Quartz`, `XXL-JOB` cũng dựa trên database lock để lập lịch tác vụ nên tồn tại bottleneck về performance. Tuy nhiên, thông thường nếu số lượng tác vụ không quá lớn thì ảnh hưởng không đáng kể và có thể đáp ứng yêu cầu của phần lớn công ty.

Ở version hiện tại, nên sử dụng `@XxlJob` trên method `void` không có tham số trong Spring Bean. Tham số tác vụ, log và kết quả thực hiện được xử lý thông qua `XxlJobHelper`; cách viết dùng `@JobHandler`, nhận `String` và trả về `ReturnT` thuộc API cũ.

```java
@Component
public class MyApiJobHandler {

    @XxlJob("myApiJobHandler")
    public void execute() {
        String param = XxlJobHelper.getJobParam();
        XxlJobHelper.log("Tham số tác vụ: {}", param);
        // Thực hiện logic nghiệp vụ; kết quả mặc định là thành công, khi thất bại có thể gọi XxlJobHelper.handleFail(...)
    }
}
```

![](https://oss.javaguide.cn/github/javaguide/system-design/schedule-task/xxljob-admin-task-management.png)

**Địa chỉ liên quan:**

- Địa chỉ GitHub: <https://github.com/xuxueli/xxl-job/>.
- Giới thiệu chính thức: <https://www.xuxueli.com/xxl-job/> .

**Tổng hợp ưu / nhược điểm:**

- Ưu điểm: dùng được ngay (chi phí học tương đối thấp), tích hợp với Spring, hỗ trợ phân tán, hỗ trợ cluster, hỗ trợ quản lý tác vụ trực quan.
- Nhược điểm: không hỗ trợ thêm tác vụ động (nếu nhất định muốn tạo tác vụ động thì vẫn hỗ trợ, xem [xxl-job issue277](https://github.com/xuxueli/xxl-job/issues/277)).

### PowerJob

Đây là một framework lập lịch tác vụ phân tán rất đáng chú ý, một ngôi sao mới trong lĩnh vực lập lịch tác vụ phân tán. Hiện nay, nhiều công ty đã tích hợp, chẳng hạn OPPO, JD.com, ZTO và Cisco.

Câu chuyện ra đời của framework này cũng khá thú vị. Tác giả PowerJob từng thực tập tại Alibaba. Khi đó Alibaba sử dụng SchedulerX do nội bộ tự phát triển (sản phẩm trả phí của Alibaba Cloud). Sau khi kết thúc kỳ thực tập, tác giả PowerJob rời Alibaba. Tác giả nghĩ đến việc tự phát triển một SchedulerX để phòng trường hợp một ngày nào đó SchedulerX không đáp ứng được nhu cầu, và PowerJob đã ra đời như vậy.

Để biết thêm câu chuyện về PowerJob, bạn có thể xem video của tác giả PowerJob ["Tôi và middleware lập lịch tác vụ của tôi"](https://www.bilibili.com/video/BV1SK411A7F3/). Tóm tắt đơn giản là: “Game không còn thú vị nữa, tôi sẽ giương cao ngọn cờ của framework lập lịch và tính toán phân tán thế hệ mới!”.

Vì SchedulerX là sản phẩm tính phí bằng nhân dân tệ nên ở đây tôi không giới thiệu quá nhiều. PowerJob cũng đã so sánh nó với Quartz, XXL-JOB và SchedulerX. Bảng dưới đây là so sánh tính năng của phía dự án, không phải benchmark độc lập; performance và capacity vẫn cần được kiểm chứng dựa trên version, database, quy mô deploy và tải nghiệp vụ.

|                              | QuartZ                                               | xxl-job                                                  | SchedulerX 2.0                                                         | PowerJob                                                                                |
| ---------------------------- | ---------------------------------------------------- | -------------------------------------------------------- | ---------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| Loại định kỳ                 | CRON                                                 | CRON                                                     | CRON, tần suất cố định, độ trễ cố định, OpenAPI                        | **CRON, tần suất cố định, độ trễ cố định, OpenAPI**                                     |
| Loại tác vụ                  | Java tích hợp sẵn                                    | Java tích hợp sẵn, GLUE Java, script Shell, Python, v.v. | Java tích hợp sẵn, Java bên ngoài (FatJar), script Shell, Python, v.v. | **Java tích hợp sẵn, Java bên ngoài (container), script Shell, Python, v.v.**           |
| Tính toán phân tán           | Không                                                | Sharding tĩnh                                            | Sharding động MapReduce                                                | **Sharding động MapReduce**                                                             |
| Quản trị tác vụ online       | Không hỗ trợ                                         | Hỗ trợ                                                   | Hỗ trợ                                                                 | **Hỗ trợ**                                                                              |
| Log hiển thị trực tuyến      | Không hỗ trợ                                         | Hỗ trợ                                                   | Không hỗ trợ                                                           | **Hỗ trợ**                                                                              |
| Cách lập lịch và performance | Dựa trên database lock, có bottleneck performance    | Dựa trên database lock, có bottleneck performance        | Không rõ                                                               | **Phía dự án cho biết sử dụng thiết kế không lock, capacity thực tế cần benchmark tải** |
| Monitoring cảnh báo          | Không                                                | Email                                                    | SMS                                                                    | **WebHook, email, DingTalk và extension tùy chỉnh**                                     |
| Dependency hệ thống          | Database quan hệ được JDBC hỗ trợ (MySQL, Oracle...) | MySQL                                                    | Nhân dân tệ                                                            | **Database quan hệ bất kỳ được Spring Data Jpa hỗ trợ (MySQL, Oracle...)**              |
| Workflow DAG                 | Không hỗ trợ                                         | Không hỗ trợ                                             | Hỗ trợ                                                                 | **Hỗ trợ**                                                                              |

## Tổng kết các phương án tác vụ định kỳ

Các giải pháp phổ biến cho tác vụ định kỳ trên một máy gồm `Timer`, `ScheduledExecutorService`, `DelayQueue`, Spring Task và time wheel. Với tác vụ định kỳ hoặc tác vụ trễ Java thông thường, thường nên ưu tiên `ScheduledExecutorService` hoặc Spring Task; time wheel phù hợp hơn với trường hợp có nhiều timer, chấp nhận đánh đổi một phần độ chính xác thời gian để lấy performance lập lịch, không phải lựa chọn mặc định tốt nhất cho mọi tác vụ trên một máy.

Redis và MQ tuy có thể thực hiện trigger trễ phân tán, nhưng thường không cung cấp đầy đủ orchestration tác vụ, sharding, bù trừ khi thất bại và quản lý trực quan. Semantics giao hàng phổ biến của MQ đáng tin cậy là “at-least-once”; khi timeout, retry hoặc failover có thể xuất hiện việc gửi trùng, vì vậy consumer bắt buộc phải xử lý idempotent. Tác vụ định kỳ cần scheduler liên tục tạo event trigger mới, không liên quan trực tiếp đến việc một message có thể được “consume nhiều lần” hay không. MQ vẫn rất phù hợp với trigger trễ một lần như hủy đơn hàng quá hạn, đồng thời có thể dùng để tách rời scheduling và execution.

Bất kể chọn phương án nào, trước khi đưa lên production cần làm rõ các semantics sau:

- Tác vụ có idempotent không, xử lý việc thực hiện trùng như thế nào;
- Sau khi ứng dụng dừng hoặc bỏ lỡ thời điểm thực hiện, sẽ thực hiện bù, bỏ qua hay gộp thực hiện;
- Chiến lược timeout, retry, backoff, số lần thử tối đa và dead letter / bù trừ thủ công;
- Cách xử lý timezone, daylight saving time, clock rollback và sai lệch clock giữa các node;
- Cách tránh lập lịch trùng ngoài dự kiến trong cluster, đồng thời thiết lập monitoring và cảnh báo cho delay, tỷ lệ thành công, retry và backlog.

Quartz, Elastic-Job, XXL-JOB và PowerJob là các framework chuyên dùng cho lập lịch phân tán. Chúng cung cấp chức năng tác vụ định kỳ phân tán đầy đủ và mạnh hơn, phù hợp hơn để thực hiện tác vụ định kỳ. Ngoài Quartz, ba framework còn lại đều hỗ trợ quản lý tác vụ trực quan.

XXL-JOB ra mắt năm 2015, có ngưỡng sử dụng tương đối thấp và dùng lập lịch tập trung; ElasticJob dùng lập lịch phi tập trung, đồng thời điều phối sharding tác vụ thông qua ZooKeeper hoặc etcd. Kiến trúc và dependency vận hành của hai framework khác nhau, không thể từ đó kết luận trực tiếp framework nào “performance tốt hơn”. Khi lựa chọn, cần benchmark tải theo version thực tế, số lượng tác vụ, tần suất trigger, quy mô sharding, yêu cầu khôi phục khi lỗi và tải của database hoặc registry center. Các framework khác như PowerJob cũng cần được kiểm chứng theo cùng các tiêu chí, không nên chỉ dựa vào bảng so sánh tính năng của phía dự án để kết luận.

Bài viết này không giới thiệu việc sử dụng thực tế, nhưng điều đó không có nghĩa sử dụng thực tế không quan trọng. Trước khi viết bài, tôi đã tự tay viết Demo tương ứng. Tôi từng sử dụng Quartz từ thời đại học. Tuy nhiên, khi đó tôi dùng Spring. Để trải nghiệm tốt hơn, tôi cũng tự trải nghiệm thực tế trên Spring Boot. Nếu chưa từng sử dụng thực tế một framework nào đó mà đã trực tiếp nói framework đó không tốt thì lập luận sẽ không thuyết phục.

<!-- @include: @article-footer.snippet.md -->
