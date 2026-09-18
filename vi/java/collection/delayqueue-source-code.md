---
title: Phân tích source code DelayQueue
description: "Giải thích chuyên sâu source code DelayQueue: nguyên lý triển khai delay queue, cách sử dụng interface Delayed, lập lịch tác vụ trễ, các trường hợp sử dụng như hủy đơn hàng quá hạn và thiết kế thread-safe dựa trên PriorityQueue."
category: Java
tag:
  - Java Collection
head:
  - - meta
    - name: keywords
      content: source code DelayQueue, delay queue, interface Delayed, delayed task, scheduled task, order timeout, triển khai PriorityQueue
---

## Giới thiệu về DelayQueue

`DelayQueue` là delay queue do package JUC (`java.util.concurrent`) cung cấp, dùng để triển khai delayed task, chẳng hạn tự động hủy đơn hàng sau 15 phút nếu chưa thanh toán. Đây là một loại `BlockingQueue`, bên dưới là một unbounded queue dựa trên `PriorityQueue` và có tính thread-safe. Bạn có thể tham khảo bài viết [Phân tích source code PriorityQueue](./priorityqueue-source-code.md) do tác giả biên soạn.

![Các lớp triển khai của BlockingQueue](https://oss.javaguide.cn/github/javaguide/java/collection/blocking-queue-hierarchy.png)

Các phần tử được lưu trong `DelayQueue` phải triển khai interface `Delayed`, đồng thời cần override phương thức `getDelay()` (tính xem đã đến hạn hay chưa).

```java
public interface Delayed extends Comparable<Delayed> {
    long getDelay(TimeUnit unit);
}
```

Theo mặc định, `DelayQueue` sắp xếp task theo thứ tự tăng dần của thời điểm đến hạn. Chỉ khi phần tử hết hạn (`getDelay()` trả về giá trị nhỏ hơn hoặc bằng 0), phần tử đó mới có thể được lấy ra khỏi queue.

`DelayQueue` được giới thiệu lần đầu trong Java 5, là một blocking queue vô hạn và thread-safe.

## Ví dụ các trường hợp sử dụng phổ biến của DelayQueue

Ở đây, ta muốn task chuyển sang trạng thái có thể lấy ra theo độ trễ đã định. Ví dụ, tạo 3 task với độ trễ lần lượt là 1s, 2s, 3s; dù được thêm không theo thứ tự, task hết hạn sớm nhất vẫn sẽ chuyển sang trạng thái có thể lấy ra trước.

![Delayed task](https://oss.javaguide.cn/github/javaguide/java/collection/delayed-task.png)

Ta có thể dùng `DelayQueue` để triển khai việc này. Trước tiên, ta cần implement `Delayed` để tạo `DelayedTask`, triển khai phương thức `getDelay()` và phép so sánh priority `compareTo`.

```java
/**
 * Delayed task
 */
public class DelayedTask implements Delayed {
    /**
     * Thời điểm task đến hạn
     */
    private long executeTime;
    /**
     * Task
     */
    private Runnable task;

    public DelayedTask(long delay, Runnable task) {
        this.executeTime = System.currentTimeMillis() + delay;
        this.task = task;
    }

    /**
     * Xem task hiện tại còn bao lâu thì đến hạn
     * @param unit
     * @return
     */
    @Override
    public long getDelay(TimeUnit unit) {
        return unit.convert(executeTime - System.currentTimeMillis(), TimeUnit.MILLISECONDS);
    }

    /**
     * Delay queue cần enqueue theo thứ tự tăng dần của thời điểm đến hạn,
     * nên cần triển khai compareTo để so sánh thời điểm đến hạn
     * @param o
     * @return
     */
    @Override
    public int compareTo(Delayed o) {
        return Long.compare(this.executeTime, ((DelayedTask) o).executeTime);
    }

    public void execute() {
        task.run();
    }
}
```

Sau khi hoàn tất việc đóng gói task, cách sử dụng trở nên rất đơn giản: đặt thời gian đến hạn rồi thêm task vào delay queue.

```java
// Tạo delay queue và thêm task
DelayQueue < DelayedTask > delayQueue = new DelayQueue < > ();

// Lần lượt thêm các task đến hạn sau 1s, 2s, 3s
delayQueue.add(new DelayedTask(2000, () -> System.out.println("Task 2")));
delayQueue.add(new DelayedTask(1000, () -> System.out.println("Task 1")));
delayQueue.add(new DelayedTask(3000, () -> System.out.println("Task 3")));

// Lấy task ra và thực thi
while (!delayQueue.isEmpty()) {
  // Blocking để lấy task đến hạn sớm nhất
  DelayedTask task = delayQueue.take();
  if (task != null) {
    task.execute();
  }
}
```

Từ kết quả output có thể thấy, dù task đến hạn sau 2s được thêm trước, task đến hạn sau 1s `Task1` vẫn được thực thi trước.

```java
Task 1
Task 2
Task 3
```

## Phân tích source code DelayQueue

Ở đây lấy JDK1.8 làm ví dụ để phân tích source code cốt lõi bên dưới của `DelayQueue`.

Định nghĩa class `DelayQueue` như sau:

```java
public class DelayQueue<E extends Delayed> extends AbstractQueue<E> implements BlockingQueue<E>
{
  //...
}
```

`DelayQueue` kế thừa class `AbstractQueue` và triển khai interface `BlockingQueue`.

![Class diagram DelayQueue](https://oss.javaguide.cn/github/javaguide/java/collection/delayqueue-class-diagram.png)

### Các member variable cốt lõi

4 member variable cốt lõi của `DelayQueue` như sau:

```java
// Reentrant lock, yếu tố then chốt để đảm bảo thread-safe
private final transient ReentrantLock lock = new ReentrantLock();
// Collection lưu trữ dữ liệu ở tầng dưới của delay queue, đảm bảo các phần tử được sắp xếp tăng dần theo thời điểm đến hạn
private final PriorityQueue<E> q = new PriorityQueue<E>();

// Trỏ đến thread có priority cao nhất đang chuẩn bị thực thi
private Thread leader = null;
// Triển khai tương tác chờ và đánh thức giữa nhiều thread
private final Condition available = lock.newCondition();
```

- `lock`: Như đã biết, việc lưu và lấy trong `DelayQueue` là thread-safe. Vì vậy, để đảm bảo thread-safe khi lưu và lấy phần tử, ta cần lock trong lúc thao tác. `DelayQueue` dựa trên exclusive lock `ReentrantLock` để đảm bảo thread-safe cho các thao tác này.
- `q`: Delay queue yêu cầu các phần tử được sắp xếp tăng dần theo thời điểm đến hạn, nên khi thêm phần tử chắc chắn cần sắp xếp theo priority. Vì vậy, việc lưu và lấy phần tử bên dưới `DelayQueue` đều được quản lý thông qua member variable `q` của priority queue `PriorityQueue` này.
- `leader`: Task của delay queue chỉ thực thi sau khi đến hạn; task chưa đến hạn phải chờ. Để đảm bảo task có priority cao nhất được thực thi ngay khi đến hạn, nhà thiết kế dùng `leader` để quản lý delayed task. Chỉ thread mà `leader` trỏ tới mới có quyền timed wait cho đến khi task đến hạn rồi thực thi; các thread còn lại chỉ có thể chờ vô thời hạn, đến khi thread `leader` xử lý xong delayed task hiện tại thì đánh thức chúng.
- `available`: Tương tác chờ và đánh thức được đề cập khi nói về thread `leader` ở trên được triển khai thông qua `available`. Ví dụ, khi thread 1 cố lấy task từ một `DelayQueue` rỗng, `available` sẽ đưa nó vào waiting queue. Khi một thread thêm delayed task, nó sẽ đánh thức thread đang chờ bằng phương thức `signal` của `available`.

### Constructor

So với các concurrent container khác, constructor của delay queue khá đơn giản. Nó chỉ có 2 constructor. Vì mọi member variable đã được khởi tạo đầy đủ khi class được load, constructor mặc định không làm gì cả. Constructor còn lại nhận một đối tượng `Collection`, gọi phương thức `addAll()` để lưu các phần tử của collection vào priority queue `q`.

```java
public DelayQueue() {}

public DelayQueue(Collection<? extends E> c) {
    this.addAll(c);
}
```

### Thêm phần tử

Các phương thức thêm của `DelayQueue`, dù là `add`, `put` hay `offer`, về bản chất đều chỉ gọi `offer`, nên để hiểu logic thêm của delay queue, ta chỉ cần đọc phương thức `offer`.

Logic tổng thể của phương thức `offer`:

1. Thử lấy `lock`.
2. Nếu lock thành công, gọi phương thức `offer` của `q` để lưu phần tử vào priority queue.
3. Gọi phương thức `peek` để xem phần tử đầu queue hiện tại có phải phần tử vừa enqueue hay không. Nếu đúng, điều đó cho thấy phần tử hiện tại là task sắp đến hạn (tức phần tử có priority cao nhất), nên đặt `leader` về null và thông báo cho các thread đang bị block vì gọi các phương thức như `take` khi queue rỗng đến tranh chấp phần tử.
4. Hoàn tất các bước trên và release `lock`.
5. Trả về true.

Source code như sau, kèm comment chi tiết:

```java
public boolean offer(E e) {
    // Thử lấy lock
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        // Nếu lock thành công, gọi offer của q để lưu phần tử vào priority queue
        q.offer(e);
        // Gọi peek để xem phần tử đầu queue hiện tại có phải phần tử vừa enqueue hay không; nếu đúng, đây là task sắp đến hạn (tức phần tử có priority cao nhất)
        if (q.peek() == e) {
            // Đặt leader về null, thông báo cho thread bị block khi gọi phương thức lấy phần tử đến tranh chấp task này
            leader = null;
            available.signal();
        }
        return true;
    } finally {
        // Hoàn tất các bước trên và release lock
        lock.unlock();
    }
}
```

### Lấy phần tử

Các cách lấy phần tử trong `DelayQueue` gồm blocking và non-blocking. Trước tiên, hãy xem phương thức lấy blocking có logic phức tạp hơn là `take`. Để người đọc dễ hình dung toàn bộ flow lấy blocking, ở đây dùng ví dụ 3 thread đồng thời lấy phần tử để giải thích flow hoạt động của `take`.

> Để hiểu nội dung bên dưới, bạn cần kiến thức liên quan đến AQS. Khuyến nghị đọc 2 bài viết sau:
>
> - [Giải thích AQS bằng hình ảnh và văn bản, cùng xem source code AQS... (bài khá dài)](https://xie.infoq.cn/article/5a3cc0b709012d40cb9f41986)
> - [Đã đọc xong AQS thì không thể thiếu nguyên lý Condition!](https://xie.infoq.cn/article/0223d5e5f19726b36b084b10d)

1. Trước tiên, 3 thread sẽ thử lấy reentrant lock `lock`. Giả sử hiện tại có 3 thread lần lượt là t1, t2, t3; sau đó t1 lấy được lock, còn t2 và t3 không lấy được lock nên được đưa vào waiting queue.

![](https://oss.javaguide.cn/github/javaguide/java/collection/delayqueue-take-0.png)

2. Ngay sau đó, t1 bắt đầu logic lấy phần tử.

3. Trước tiên thread t1 kiểm tra phần tử đầu của queue `DelayQueue` có tồn tại hay không.

4. Nếu phần tử rỗng, điều đó cho thấy queue hiện không có phần tử nào, nên t1 bị block và được lưu vào queue `conditionWaiter`.

![](https://oss.javaguide.cn/github/javaguide/java/collection/delayqueue-take-1.png)

Lưu ý, sau khi gọi `await`, t1 sẽ release lock `lock`. Nếu `DelayQueue` tiếp tục rỗng, t2 và t3 cũng sẽ thực hiện logic tương tự t1 và đi vào queue `conditionWaiter`.

![](https://oss.javaguide.cn/github/javaguide/java/collection/delayqueue-take-2.png)

Nếu queue có phần tử, kiểm tra task hiện tại đã đến hạn hay chưa. Nếu task đã đến hạn, trả về trực tiếp. Nếu task chưa đến hạn, kiểm tra thread `leader` hiện tại (thread duy nhất có thể timed wait và lấy phần tử trong `DelayQueue`) có rỗng hay không. Nếu không rỗng, điều đó cho thấy `leader` hiện đang chờ một phần tử có priority cao hơn phần tử hiện tại đến hạn, nên thread t1 chỉ có thể gọi `await` để chờ vô thời hạn, đợi đến khi `leader` lấy phần tử rồi đánh thức nó. Ngược lại, nếu thread `leader` rỗng, đặt thread hiện tại làm leader và đi vào trạng thái chờ có thời hạn; khi hết thời gian chờ thì lấy phần tử ra và trả về.

Sau khi hoàn tất logic lấy blocking, source code như sau, bạn có thể tự tham khảo:

```java
public E take() throws InterruptedException {
    // Thử lấy reentrant lock, đặt state của AQS bên dưới thành 1 và đặt thành exclusive lock
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        for (;;) {
            // Xem phần tử đầu tiên của queue
            E first = q.peek();
            // Nếu rỗng, đưa thread hiện tại vào waiting queue của ConditionObject, đặt state của AQS bên dưới thành 0, biểu thị release lock và đi vào trạng thái chờ vô thời hạn
            if (first == null)
                available.await();
            else {
                // Nếu phần tử không rỗng, xem còn bao lâu thì phần tử hiện tại đến hạn
                long delay = first.getDelay(NANOSECONDS);
                // Nếu nhỏ hơn hoặc bằng 0 thì đã đến hạn, trả về trực tiếp
                if (delay <= 0)
                    return q.poll();
                // Nếu lớn hơn 0 thì task chưa đến hạn; trước tiên cần release reference tới phần tử này
                first = null; // don't retain ref while waiting
                // Kiểm tra leader có rỗng hay không; nếu không rỗng, có thread đang làm leader và chờ một task đến hạn, nên thread hiện tại chờ vô thời hạn
                if (leader != null)
                    available.await();
                else {
                    // Ngược lại, đặt thread hiện tại làm leader
                    Thread thisThread = Thread.currentThread();
                    leader = thisThread;
                    try {
                        // Và đi vào trạng thái chờ có thời hạn
                        available.awaitNanos(delay);
                    } finally {
                        // Khi task đến hạn, release reference tới leader và vào vòng lặp tiếp theo để return task
                        if (leader == thisThread)
                            leader = null;
                    }
                }
            }
        }
    } finally {
        // Logic kết thúc: khi leader là null và queue có task, đánh thức thread đang chờ lấy phần tử.
        if (leader == null && q.peek() != null)
            available.signal();
        // Release lock
        lock.unlock();
    }
}
```

Tiếp theo, hãy xem phương thức lấy non-blocking `poll`; logic đơn giản hơn nhiều, gồm các bước sau:

1. Thử lấy reentrant lock.
2. Xem phần tử đầu tiên của queue và kiểm tra phần tử có rỗng hay không.
3. Nếu phần tử rỗng hoặc chưa đến hạn thì trả về null trực tiếp.
4. Nếu phần tử không rỗng và đã đến hạn thì gọi `poll` để trả về trực tiếp.
5. Release reentrant lock `lock`.

Source code như sau, bạn có thể tự tham khảo source code và comment:

```java
public E poll() {
    // Thử lấy reentrant lock
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        // Xem phần tử đầu tiên của queue và kiểm tra phần tử có rỗng hay không
        E first = q.peek();

        // Nếu phần tử rỗng hoặc chưa đến hạn thì trả về null trực tiếp
        if (first == null || first.getDelay(NANOSECONDS) > 0)
            return null;
        else
            // Nếu phần tử không rỗng và đã đến hạn thì gọi poll để trả về trực tiếp
            return q.poll();
    } finally {
        // Release reentrant lock lock
        lock.unlock();
    }
}
```

### Xem phần tử

Khi lấy phần tử ở trên, phương thức `peek` được gọi. Đúng như tên gọi, `peek` chỉ xem phần tử trong queue, gồm 4 bước:

1. Lock.
2. Gọi phương thức `peek` của priority queue `q` để xem phần tử ở index 0.
3. Release lock.
4. Trả về phần tử.

```java
public E peek() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return q.peek();
    } finally {
        lock.unlock();
    }
}
```

## Các câu hỏi phỏng vấn phổ biến về DelayQueue

### Nguyên lý triển khai của DelayQueue là gì?

Bên dưới `DelayQueue` dùng priority queue `PriorityQueue` để lưu trữ phần tử, còn `PriorityQueue` áp dụng tư tưởng binary min-heap để đảm bảo phần tử có giá trị nhỏ hơn đứng trước. Nhờ đó, việc quản lý priority của delayed task trong `DelayQueue` trở nên rất thuận tiện. Đồng thời, để đảm bảo thread-safe, `DelayQueue` sử dụng reentrant lock `ReentrantLock`, đảm bảo tại một thời điểm chỉ có một thread có thể thao tác với delay queue. Cuối cùng, để triển khai hiệu quả việc chờ và đánh thức giữa nhiều thread, `DelayQueue` còn sử dụng `Condition`, thực hiện việc chờ và đánh thức giữa nhiều thread thông qua các phương thức `await` và `signal` của `Condition`.

### DelayQueue có thread-safe không?

`DelayQueue` là thread-safe. Nó dùng `ReentrantLock` để triển khai truy cập độc quyền và dùng `Condition` để triển khai thao tác chờ, đánh thức giữa các thread, từ đó đảm bảo tính an toàn và tin cậy trong môi trường nhiều thread.

### DelayQueue có những trường hợp sử dụng nào?

`DelayQueue` thường được dùng để triển khai scheduled task và xóa cache hết hạn. Trong scheduled task, cần đóng gói task cần thực thi thành delayed task object rồi thêm vào `DelayQueue`; phần tử đầu queue đã hết hạn có thể được lấy ra, còn thời điểm task thực sự được thực thi vẫn phụ thuộc vào scheduling của consumer thread. Với trường hợp cache hết hạn, sau khi dữ liệu được cache vào memory, ta có thể đóng gói key của cache thành một delayed deletion task rồi thêm vào `DelayQueue`. Khi dữ liệu hết hạn, lấy key của task và xóa key đó khỏi memory.

### Interface Delayed trong DelayQueue có tác dụng gì?

Interface `Delayed` định nghĩa thời gian delay còn lại của phần tử (`getDelay`) và quy tắc so sánh giữa các phần tử (interface này kế thừa interface `Comparable`). Nếu muốn phần tử được lưu vào `DelayQueue`, bắt buộc phải triển khai phương thức `getDelay()` và `compareTo()` của interface `Delayed`; nếu không, `DelayQueue` không thể biết thời gian còn lại của task và so sánh priority giữa các task.

### DelayQueue khác Timer/TimerTask như thế nào?

`DelayQueue` và `Timer/TimerTask` đều có thể dùng để triển khai scheduled task, nhưng cách triển khai khác nhau. `DelayQueue` dựa trên priority queue, chỉ chịu trách nhiệm lưu delayed element và cho phép lấy ra sau khi đến hạn; còn `Timer/TimerTask` thực thi task theo thứ tự bằng một background thread duy nhất. Nếu một task thực thi quá lâu, nó sẽ ảnh hưởng đến việc thực thi các task khác. Cả hai đều hỗ trợ thêm task trong thời gian chạy, nhưng `DelayQueue` còn có thể trực tiếp xóa delayed element trong queue.

## Tài liệu tham khảo

- 《Hiểu sâu về lập trình high concurrency: Công nghệ cốt lõi của JDK》:
- Nói một lần về 6 cách triển khai delay queue trong Java (đến interviewer cũng phải thuyết phục):<https://www.jb51.net/article/186192.htm>
- Sơ đồ source code DelayQueue (java 8) — những điều thú vị về delay queue: <https://blog.csdn.net/every__day/article/details/113810985>
<!-- @include: @article-footer.snippet.md -->
