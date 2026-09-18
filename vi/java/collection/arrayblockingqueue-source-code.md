---
title: "Phân tích source code ArrayBlockingQueue"
description: "Giải thích chuyên sâu source code ArrayBlockingQueue: triển khai blocking queue có giới hạn, ứng dụng của mô hình producer-consumer, cơ chế kiểm soát concurrency ReentrantLock+Condition và cơ chế work queue của thread pool."
category: Java
tag:
  - Java Collections
head:
  - - meta
    - name: keywords
      content: ArrayBlockingQueue source code, blocking queue, bounded queue, producer-consumer pattern, ReentrantLock, Condition, thread pool work queue
---

## Giới thiệu về blocking queue

### Lịch sử của blocking queue

Lịch sử của blocking queue trong Java có thể bắt nguồn từ phiên bản JDK1.5. Khi đó nền tảng Java bổ sung `java.util.concurrent`, tức package JUC thường được nhắc đến, trong đó có nhiều công cụ điều khiển quy trình concurrency, container concurrency, atomic class... Blocking queue được thảo luận trong bài viết này cũng nằm trong đó.

Để giải quyết vấn đề chia sẻ dữ liệu giữa nhiều thread trong kịch bản high concurrency, phiên bản JDK1.5 xuất hiện `ArrayBlockingQueue` và `LinkedBlockingQueue`. Đây là các container concurrency được triển khai theo mô hình producer-consumer. Trong đó, `ArrayBlockingQueue` là bounded queue, tức sau khi số phần tử đạt giới hạn, lần thêm tiếp theo sẽ bị block hoặc ném exception. Còn `LinkedBlockingQueue` là queue được tạo thành từ linked list. Chính vì đặc điểm của linked list, `LinkedBlockingQueue` không có nhiều ràng buộc như `ArrayBlockingQueue` khi thêm phần tử, nên việc queue có giới hạn hay không là tùy chọn (lưu ý, unbounded ở đây không có nghĩa là có thể thêm số lượng phần tử bất kỳ, mà nghĩa là kích thước queue mặc định là `Integer.MAX_VALUE`, gần như vô hạn).

`SynchronousQueue` và `DelayQueue` cũng được đưa vào trong JDK 1.5; JDK 1.7 bổ sung thêm interface `TransferQueue` hỗ trợ thao tác transfer.

### Tư tưởng của blocking queue

Blocking queue là mô hình producer-consumer điển hình, có thể thực hiện các việc sau:

1. Khi dữ liệu trong blocking queue rỗng, tất cả consumer thread sẽ bị block và chờ queue không rỗng.
2. Khi producer đưa dữ liệu vào queue, queue sẽ thông báo queue không rỗng cho consumer, lúc này consumer có thể vào lấy dữ liệu.
3. Khi blocking queue đầy và không thể chứa phần tử mới vì consumer xử lý quá chậm hoặc producer lưu phần tử quá nhanh, producer sẽ bị block và chờ queue không đầy để tiếp tục lưu phần tử.
4. Khi consumer lấy một phần tử khỏi queue, queue sẽ thông báo queue không đầy cho producer, producer có thể tiếp tục đưa dữ liệu vào.

Tóm lại, blocking queue dựa trên hai điều kiện không rỗng và không đầy để thực hiện tương tác giữa producer và consumer. Mặc dù quy trình tương tác và cơ chế chờ-thông báo này khá phức tạp, may mắn là Doug Lea đã ẩn các chi tiết của blocking queue, chúng ta chỉ cần gọi các API như `put`, `take`, `offer`, `poll` để thực hiện việc sản xuất và tiêu thụ giữa nhiều thread.

Điều này cũng khiến blocking queue được sử dụng rộng rãi trong phát triển đa thread. Ví dụ phổ biến nhất là thread pool: từ source code có thể thấy khi core thread không thể xử lý task kịp thời, các task này sẽ được đưa vào `workQueue`.

```java
public ThreadPoolExecutor(int corePoolSize,
                            int maximumPoolSize,
                            long keepAliveTime,
                            TimeUnit unit,
                            BlockingQueue<Runnable> workQueue,
                            ThreadFactory threadFactory,
                            RejectedExecutionHandler handler) {// ...}
```

## Các method thường gặp và kiểm thử ArrayBlockingQueue

Sau khi tìm hiểu sơ lược lịch sử của blocking queue, chúng ta bắt đầu thảo luận trọng tâm về container concurrency được giới thiệu trong bài viết này: `ArrayBlockingQueue`. Để hiểu sâu hơn về `ArrayBlockingQueue` ở các phần sau, trước hết hãy tìm hiểu cách sử dụng `ArrayBlockingQueue` qua một vài ví dụ dưới đây.

Hãy xem ví dụ đầu tiên. Ở đây chúng ta dùng hai thread lần lượt mô phỏng producer và consumer. Sau khi khởi động, producer dùng method `put` để tạo 10 phần tử cho consumer tiêu thụ. Khi số phần tử trong queue đạt giới hạn 5 đã thiết lập, method `put` sẽ bị block.

Tương tự, consumer dùng method `take` để tiêu thụ phần tử. Khi queue rỗng, method `take` sẽ block consumer thread. Để đảm bảo consumer thoát kịp thời sau khi tiêu thụ xong 10 phần tử, tác giả dùng `CountDownLatch` để điều khiển việc kết thúc consumer. Producer ở đây chỉ tạo 10 phần tử. Sau khi consumer tiêu thụ xong 10 phần tử, nó gọi `CountDownLatch`, tất cả thread sẽ dừng.

```java
public class ProducerConsumerExample {

    public static void main(String[] args) throws InterruptedException {

        // Tạo một ArrayBlockingQueue có kích thước 5
        ArrayBlockingQueue<Integer> queue = new ArrayBlockingQueue<>(5);

        // Tạo producer thread
        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 10; i++) {
                    // Thêm phần tử vào queue, nếu queue đầy thì block chờ
                    queue.put(i);
                    System.out.println("Producer thêm phần tử: " + i);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        });

        CountDownLatch countDownLatch = new CountDownLatch(1);

        // Tạo consumer thread
        Thread consumer = new Thread(() -> {
            try {
                int count = 0;
                while (true) {

                    // Lấy phần tử khỏi queue, nếu queue rỗng thì block chờ
                    int element = queue.take();
                    System.out.println("Consumer lấy phần tử: " + element);
                    ++count;
                    if (count == 10) {
                        break;
                    }
                }

                countDownLatch.countDown();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        });

        // Khởi động thread
        producer.start();
        consumer.start();

        // Chờ thread kết thúc
        producer.join();
        consumer.join();

        countDownLatch.await();

        producer.interrupt();
        consumer.interrupt();
    }

}
```

Kết quả output của code như sau. Có thể thấy chỉ sau khi producer đưa phần tử vào queue thì consumer mới có thể tiêu thụ. Điều đó có nghĩa là khi queue không có dữ liệu, consumer sẽ bị block và chờ queue khác rỗng rồi mới tiếp tục tiêu thụ.

```cpp
Producer thêm phần tử: 1
Producer thêm phần tử: 2
Consumer lấy phần tử: 1
Consumer lấy phần tử: 2
Producer thêm phần tử: 3
Consumer lấy phần tử: 3
Producer thêm phần tử: 4
Producer thêm phần tử: 5
Consumer lấy phần tử: 4
Producer thêm phần tử: 6
Consumer lấy phần tử: 5
Producer thêm phần tử: 7
Producer thêm phần tử: 8
Producer thêm phần tử: 9
Producer thêm phần tử: 10
Consumer lấy phần tử: 6
Consumer lấy phần tử: 7
Consumer lấy phần tử: 8
Consumer lấy phần tử: 9
Consumer lấy phần tử: 10
```

Sau khi tìm hiểu hai method thêm và lấy có block là `put`, `take`, chúng ta xem tiếp hai method thêm vào queue và lấy khỏi queue không block là `offer` và `poll`.

Như bên dưới, chúng ta thiết lập một blocking queue có kích thước 3, thử lưu 4 phần tử vào queue bằng method `offer`, sau đó thử lấy 4 lần từ queue bằng `poll`.

```cpp
public class OfferPollExample {

    public static void main(String[] args) {
        // Tạo một ArrayBlockingQueue có kích thước 3
        ArrayBlockingQueue<String> queue = new ArrayBlockingQueue<>(3);

        // Thêm phần tử vào queue
        System.out.println(queue.offer("A"));
        System.out.println(queue.offer("B"));
        System.out.println(queue.offer("C"));

        // Thử thêm phần tử vào queue, nhưng queue đã đầy, trả về false
        System.out.println(queue.offer("D"));

        // Lấy phần tử khỏi queue
        System.out.println(queue.poll());
        System.out.println(queue.poll());
        System.out.println(queue.poll());

        // Thử lấy phần tử khỏi queue, nhưng queue đã rỗng, trả về null
        System.out.println(queue.poll());
    }

}
```

Kết quả output cuối cùng của code như sau. Có thể thấy vì kích thước queue là 3 nên kết quả của 3 lần lưu đầu tiên là `true`; đến lần lưu thứ 4, vì queue đã đầy nên kết quả trả về là `false`. Đây cũng là lý do các method `poll` tiếp theo chỉ lấy được giá trị của 3 phần tử.

```cpp
true
true
true
false
A
B
C
null
```

Sau khi tìm hiểu thao tác thêm và lấy dạng block cũng như non-block, hãy xem một thao tác khá đặc biệt của blocking queue. Trong một số trường hợp, chúng ta muốn lưu kết quả của blocking queue vào list một lần rồi thực hiện thao tác theo batch. Khi đó có thể dùng method `drainTo` của blocking queue. Method này lưu tất cả phần tử trong queue vào list trong một lần. Nếu queue có phần tử và lưu thành công vào list, `drainTo` trả về số phần tử đã chuyển vào list; ngược lại, nếu queue rỗng, `drainTo` trả về 0 ngay lập tức.

```java
public class DrainToExample {

    public static void main(String[] args) {
        // Tạo một ArrayBlockingQueue có kích thước 5
        ArrayBlockingQueue<Integer> queue = new ArrayBlockingQueue<>(5);

        // Thêm phần tử vào queue
        queue.add(1);
        queue.add(2);
        queue.add(3);
        queue.add(4);
        queue.add(5);

        // Tạo một List để lưu các phần tử lấy khỏi queue
        List<Integer> list = new ArrayList<>();

        // Lấy tất cả phần tử khỏi queue và thêm vào List
        queue.drainTo(list);

        // In các phần tử trong List
        System.out.println(list);
    }

}
```

Kết quả output của code như sau:

```cpp
[1, 2, 3, 4, 5]
```

## Phân tích source code ArrayBlockingQueue

Đến đây chúng ta đã có ấn tượng cơ bản về cách sử dụng blocking queue, tiếp theo có thể tìm hiểu sâu hơn về cơ chế hoạt động của `ArrayBlockingQueue`.

### Thiết kế tổng thể

Trước khi tìm hiểu chi tiết cụ thể của `ArrayBlockingQueue`, hãy xem class diagram của `ArrayBlockingQueue`.

![Class diagram ArrayBlockingQueue](https://oss.javaguide.cn/github/javaguide/java/collection/arrayblockingqueue-class-diagram.png)

Từ hình có thể thấy `ArrayBlockingQueue` triển khai interface `BlockingQueue` của blocking queue. Có thể dễ dàng đoán rằng sau khi triển khai interface `BlockingQueue`, `ArrayBlockingQueue` có các hành vi thao tác thường gặp của blocking queue.

Đồng thời, `ArrayBlockingQueue` còn kế thừa abstract class `AbstractQueue`. Abstract class này kế thừa `AbstractCollection` và `Queue`. Từ đặc điểm và ngữ nghĩa của các abstract class, cũng có thể đoán rằng quan hệ kế thừa này giúp `ArrayBlockingQueue` có các thao tác thường gặp của queue.

Vậy có thể đưa ra kết luận như sau: thông qua việc kế thừa `AbstractQueue`, class nhận được template của toàn bộ thao tác queue và framework tổng thể của thao tác thêm và lấy phần tử. Sau đó `ArrayBlockingQueue` lấy được các thao tác thường gặp của blocking queue thông qua việc triển khai `BlockingQueue`, rồi triển khai các thao tác này và điền chi tiết vào template method của `AbstractQueue`. Nhờ đó, `ArrayBlockingQueue` trở thành một blocking queue hoàn chỉnh.

Để kiểm chứng điều này, hãy đi sâu vào source code. Trước tiên xem `AbstractQueue`. Từ quan hệ kế thừa của class, có thể suy ra khái quát rằng class này lấy được các method thao tác thường gặp của collection thông qua `AbstractCollection`, rồi lấy các đặc điểm của queue thông qua interface `Queue`.

```java
public abstract class AbstractQueue<E>
    extends AbstractCollection<E>
    implements Queue<E> {
       //...
}
```

Các thao tác với collection không ngoài việc thêm, xóa, sửa, tìm kiếm, nên hãy bắt đầu từ method thêm. Từ source code có thể thấy class triển khai method `add` của `AbstractCollection`, logic bên trong như sau:

1. Gọi method `offer` nhận được từ interface `Queue`; nếu `offer` thành công thì trả về `true`.
2. Nếu `offer` thất bại, nghĩa là thêm phần tử vào queue thất bại, lập tức ném exception.

```java
public boolean add(E e) {
  if (offer(e))
      return true;
  else
      throw new IllegalStateException("Queue full");
}
```

Trong `AbstractQueue` không có triển khai cho `offer` của `Queue`. Rõ ràng mục đích của cách làm này là định nghĩa logic cốt lõi của `add`, còn chi tiết của `offer` được giao cho subclass, tức `ArrayBlockingQueue`, triển khai.

Đến đây phần phân tích abstract class `AbstractQueue` kết thúc. Tiếp theo hãy xem interface quan trọng khác được triển khai trong `ArrayBlockingQueue`: `BlockingQueue`.

Mở `BlockingQueue`, có thể thấy interface này cũng kế thừa interface `Queue`, nghĩa là nó có toàn bộ hành vi của queue. Đồng thời, nó còn định nghĩa các method cần tự triển khai.

```java
public interface BlockingQueue<E> extends Queue<E> {

     // Thêm phần tử vào queue thành công trả về true, ngược lại ném IllegalStateException
    boolean add(E e);

     // Thêm phần tử vào queue thành công trả về true, ngược lại trả về false
    boolean offer(E e);

     // Nếu thêm phần tử thành công thì trả về ngay; nếu queue đầy, thread sẽ bị block vì phần tử không thể thêm vào.
     // Do trong thời gian block thread có thể bị interrupt nên signature của method ném InterruptedException
    void put(E e) throws InterruptedException;

   // Giống method trước, nhưng khi queue đầy chỉ block trong thời lượng timeout với đơn vị unit.
   // Nếu không thêm thành công trong thời gian chờ thì trả về false ngay.
    boolean offer(E e, long timeout, TimeUnit unit)
        throws InterruptedException;

    // Lấy một phần tử ở đầu queue; nếu queue rỗng thì block chờ.
    // Vì thread sẽ bị block nên method có thể bị interrupt, do đó signature định nghĩa InterruptedException
    E take() throws InterruptedException;

      // Lấy và trả về phần tử ở đầu queue. Nếu queue hiện tại rỗng thì block trong thời lượng timeout với đơn vị unit.
      // Nếu không có phần tử trong khoảng thời gian này thì trả về null ngay.
    E poll(long timeout, TimeUnit unit)
        throws InterruptedException;

      // Lấy số phần tử còn lại trong queue
    int remainingCapacity();

     // Xóa object được chỉ định, thành công trả về true, ngược lại trả về false.
    boolean remove(Object o);

    // Kiểm tra queue có chứa phần tử được chỉ định hay không
    public boolean contains(Object o);

     // Lưu toàn bộ phần tử trong queue vào collection được chỉ định
    int drainTo(Collection<? super E> c);

    // Chuyển maxElements phần tử vào collection
    int drainTo(Collection<? super E> c, int maxElements);
}
```

Sau khi hiểu các thao tác thường gặp của `BlockingQueue`, chúng ta biết rằng `ArrayBlockingQueue` triển khai rồi override các method của `BlockingQueue`, sau đó điền chúng vào các method của `AbstractQueue`. Nhờ vậy chúng ta biết method `offer` được gọi trong method `add` của `AbstractQueue` ở phần trên được triển khai ở đâu.

```java
public boolean add(E e) {
  // offer của AbstractQueue đến từ offer bên dưới, do ArrayBlockingQueue triển khai và override từ BlockingQueue
  if (offer(e))
      return true;
  else
      throw new IllegalStateException("Queue full");
}
```

### Khởi tạo

Trước khi tìm hiểu chi tiết `ArrayBlockingQueue`, hãy xem constructor để hiểu quy trình khởi tạo. Từ source code có thể thấy `ArrayBlockingQueue` có 3 constructor, trong đó constructor cốt lõi nhất là constructor bên dưới.

```java
// capacity biểu thị capacity ban đầu của queue, fair biểu thị tính fairness của lock
public ArrayBlockingQueue(int capacity, boolean fair) {
  // Nếu kích thước queue được thiết lập nhỏ hơn 0 thì ném IllegalArgumentException ngay
  if (capacity <= 0)
      throw new IllegalArgumentException();
  // Khởi tạo một array để lưu phần tử của queue
  this.items = new Object[capacity];
  // Tạo lock điều khiển quy trình của blocking queue
  lock = new ReentrantLock(fair);
  // Dùng lock tạo hai condition để điều khiển việc producer và consumer thao tác với queue
  notEmpty = lock.newCondition();
  notFull =  lock.newCondition();
}
```

Trong constructor này có hai member variable cốt lõi là `notEmpty` (không rỗng) và `notFull` (không đầy), cần đặc biệt lưu ý. Chúng là yếu tố then chốt để producer và consumer làm việc theo thứ tự. Tác giả sẽ giải thích chi tiết điều này trong phần phân tích source code sau; ở đây chỉ cần hiểu sơ lược việc khởi tạo blocking queue.

Hai constructor còn lại đều dựa trên constructor trên. Trong trường hợp mặc định, chúng ta dùng constructor dưới đây, nghĩa là `ArrayBlockingQueue` dùng unfair lock. Tức là sau khi được thông báo, các producer hoặc consumer thread tranh lock một cách ngẫu nhiên.

```java
 public ArrayBlockingQueue(int capacity) {
        this(capacity, false);
    }
```

Một constructor khác ít được dùng hơn cung cấp thêm tham số `Collection` sau khi khởi tạo capacity và fairness của lock. Từ source code có thể thấy constructor này lưu trực tiếp các phần tử của collection truyền từ bên ngoài vào blocking queue trong lúc khởi tạo.

```java
public ArrayBlockingQueue(int capacity, boolean fair,
                              Collection<? extends E> c) {
  // Khởi tạo capacity và fairness của lock
  this(capacity, fair);

  final ReentrantLock lock = this.lock;
  // Lock và lưu các phần tử trong c vào array bên dưới của ArrayBlockingQueue
  lock.lock();
  try {
      int i = 0;
      try {
                // Duyệt và thêm phần tử vào array
          for (E e : c) {
              checkNotNull(e);
              items[i++] = e;
          }
      } catch (ArrayIndexOutOfBoundsException ex) {
          throw new IllegalArgumentException();
      }
      // Ghi lại capacity hiện tại của queue
      count = i;
                      // Cập nhật vị trí tiếp theo mà method put, offer hoặc add sẽ thêm vào array bên dưới của queue
      putIndex = (i == capacity) ? 0 : i;
  } finally {
      // Giải phóng lock sau khi duyệt xong
      lock.unlock();
  }
}
```

### Lấy và thêm phần tử dạng block

Việc lấy và thêm phần tử dạng block của `ArrayBlockingQueue` tương ứng với mô hình producer-consumer. Mặc dù nó cũng hỗ trợ lấy và thêm phần tử dạng non-block (ví dụ method `poll()` và `offer(E e)`, sẽ giới thiệu ở phần sau), nhưng thông thường không dùng chúng.

Các method lấy và thêm phần tử dạng block của `ArrayBlockingQueue` là:

- `put(E e)`: chèn phần tử vào queue. Nếu queue đầy, method sẽ block liên tục cho đến khi queue có chỗ trống hoặc thread bị interrupt.
- `take()`: lấy và xóa phần tử ở đầu queue. Nếu queue rỗng, method sẽ block liên tục cho đến khi queue không rỗng hoặc thread bị interrupt.

Điểm then chốt trong triển khai hai method này là hai condition object `notEmpty` (không rỗng) và `notFull` (không đầy), đã được nhắc đến trong constructor ở phần trên.

Tiếp theo, tác giả dùng hai hình để giúp mọi người hiểu cách hai condition này được sử dụng trong blocking queue.

![Condition không rỗng của ArrayBlockingQueue](https://oss.javaguide.cn/github/javaguide/java/collection/ArrayBlockingQueue-notEmpty-take.png)

Giả sử consumer trong code khởi động trước. Khi phát hiện queue không có dữ liệu, condition không rỗng sẽ suspend thread này, tức suspend cho đến khi condition không rỗng. Sau đó quyền thực thi CPU chuyển sang producer. Producer phát hiện queue có thể lưu dữ liệu, nên đưa dữ liệu vào queue và thông báo condition không rỗng. Lúc này consumer được đánh thức để vào queue lấy giá trị bằng các method như `take`.

![Condition không đầy của ArrayBlockingQueue](https://oss.javaguide.cn/github/javaguide/java/collection/ArrayBlockingQueue-notFull-put.png)

Trong quá trình thực thi tiếp theo, tốc độ producer sản xuất lớn hơn rất nhiều so với tốc độ consumer tiêu thụ. Vì vậy sau khi lấp đầy queue, producer lại thử lưu dữ liệu vào queue, phát hiện queue đã đầy, nên blocking queue suspend thread hiện tại và chờ queue không đầy. Sau đó consumer nắm quyền thực thi CPU và thực hiện tiêu thụ, nhờ vậy queue có thể lưu dữ liệu mới. Queue phát thông báo không đầy, lúc này producer đang suspend sẽ chờ đến khi có quyền thực thi CPU rồi thử lưu dữ liệu vào queue lần nữa.

Sau khi tìm hiểu sơ lược quy trình tương tác của blocking queue dựa trên hai condition, hãy xem source code của method `put` và `take`.

```java
public void put(E e) throws InterruptedException {
    // Đảm bảo phần tử được chèn không phải null
    checkNotNull(e);
    // Lock
    final ReentrantLock lock = this.lock;
    // Dùng lockInterruptibly() thay vì lock() để có thể phản hồi thao tác interrupt.
    // Nếu bị interrupt trong lúc chờ lấy lock, method sẽ ném exception InterruptedException.
    lock.lockInterruptibly();
    try {
            // Nếu count bằng độ dài array thì queue đã đầy, thread hiện tại sẽ được suspend và đưa vào AQS queue,
            // chờ queue không đầy rồi chèn phần tử (condition không đầy).
       // Trong thời gian chờ, lock được giải phóng để thread khác tiếp tục thao tác với queue.
        while (count == items.length)
            notFull.await();
           // Nếu queue có thể lưu phần tử thì gọi enqueue để thêm phần tử vào queue
        enqueue(e);
    } finally {
        // Giải phóng lock
        lock.unlock();
    }
}
```

Bên trong method `put` gọi method `enqueue` để thực hiện việc thêm phần tử vào queue. Hãy tiếp tục đi sâu vào chi tiết triển khai method `enqueue`:

```java
private void enqueue(E x) {
   // Lấy array bên dưới của queue
    final Object[] items = this.items;
    // Đặt giá trị x được truyền vào tại vị trí putIndex
    items[putIndex] = x;
    // Cập nhật putIndex, nếu putIndex bằng độ dài array thì cập nhật thành 0
    if (++putIndex == items.length)
        putIndex = 0;
    // Tăng độ dài queue lên 1
    count++;
    // Thông báo queue không rỗng, các thread bị block vì lấy phần tử có thể tiếp tục làm việc
    notEmpty.signal();
}
```

Từ source code có thể thấy logic thêm phần tử vào queue là thêm một phần tử mới vào array, các bước tổng thể như sau:

1. Lấy array `items` bên dưới của `ArrayBlockingQueue`.
2. Lưu phần tử tại vị trí `putIndex`.
3. Cập nhật `putIndex` sang vị trí tiếp theo. Nếu `putIndex` bằng độ dài queue, nghĩa là `putIndex` đã đến cuối array, lần chèn tiếp theo cần bắt đầu từ 0. (`ArrayBlockingQueue` dùng tư tưởng circular queue, tức tái sử dụng một array theo vòng từ đầu đến cuối.)
4. Cập nhật giá trị `count`, biểu thị độ dài queue hiện tại tăng 1.
5. Gọi `notEmpty.signal()` để thông báo queue không rỗng, consumer có thể lấy giá trị khỏi queue.

Đến đây chúng ta đã hiểu quy trình của method `put`. Để hiểu đầy đủ hơn về thiết kế mô hình producer-consumer của `ArrayBlockingQueue`, hãy tiếp tục xem method `take` dùng để lấy phần tử dạng block khỏi queue.

```java
public E take() throws InterruptedException {
       // Lấy lock
     final ReentrantLock lock = this.lock;
     lock.lockInterruptibly();
     try {
             // Nếu số phần tử trong queue bằng 0, suspend thread hiện tại và đưa vào AQS queue,
             // chờ queue không rỗng rồi lấy và xóa phần tử (condition không rỗng)
         while (count == 0)
             notEmpty.await();
            // Nếu queue không rỗng thì gọi dequeue để lấy phần tử
         return dequeue();
     } finally {
          // Giải phóng lock
         lock.unlock();
     }
}
```

Sau khi hiểu method `put`, việc xem method `take` trở nên rất đơn giản. Logic cốt lõi của chúng hoàn toàn ngược nhau. Ví dụ, method `put` chờ queue không đầy rồi chèn phần tử khi queue đầy (condition không đầy), còn method `take` chờ queue không rỗng rồi lấy và xóa phần tử (condition không rỗng).

Bên trong method `take` gọi method `dequeue` để thực hiện việc lấy phần tử khỏi queue. Logic cốt lõi của nó cũng ngược với method `enqueue`.

```java
private E dequeue() {
  // Lấy array bên dưới của blocking queue
  final Object[] items = this.items;
  @SuppressWarnings("unchecked")
  // Lấy phần tử tại vị trí takeIndex khỏi queue
  E x = (E) items[takeIndex];
  // Đặt takeIndex về null
  items[takeIndex] = null;
  // Di chuyển takeIndex về sau, nếu bằng độ dài array thì cập nhật thành 0
  if (++takeIndex == items.length)
      takeIndex = 0;
  // Giảm độ dài queue đi 1
  count--;
  if (itrs != null)
      itrs.elementDequeued();
  // Thông báo cho các thread bị suspend rằng queue hiện tại không đầy và có thể tiếp tục lưu phần tử
  notFull.signal();
  return x;
}
```

Vì các bước của method `dequeue` (lấy phần tử khỏi queue) và method `enqueue` (thêm phần tử vào queue) đã giới thiệu ở trên về cơ bản tương tự nhau, phần này không lặp lại.

Để giúp dễ hiểu, tác giả vẽ một hình riêng mô tả cách hai condition object `notEmpty` (không rỗng) và `notFull` (không đầy) điều khiển việc thêm và lấy của `ArrayBlockingQueue`.

![Condition không rỗng và không đầy của ArrayBlockingQueue](https://oss.javaguide.cn/github/javaguide/java/collection/ArrayBlockingQueue-notEmpty-notFull.png)

- **Consumer**: sau khi consumer lấy một phần tử khỏi queue bằng thao tác `take` hoặc `poll`, nó thông báo queue không đầy. Khi đó các producer đang chờ queue không đầy sẽ được đánh thức và chờ lấy time slice CPU để thực hiện thao tác thêm vào queue.
- **Producer**: sau khi producer lưu phần tử vào queue, nó thông báo queue không rỗng. Khi đó consumer được đánh thức và chờ lấy time slice CPU để thử lấy phần tử. Hai condition object lặp lại quy trình này để tạo thành một vòng lặp, điều khiển việc thêm và lấy giữa nhiều thread.

### Lấy và thêm phần tử dạng non-block

Các method lấy và thêm phần tử dạng non-block của `ArrayBlockingQueue` là:

- `offer(E e)`: chèn phần tử vào cuối queue. Nếu queue đầy, method trả về `false` ngay mà không chờ và block thread.
- `poll()`: lấy và xóa phần tử ở đầu queue. Nếu queue rỗng, method trả về `null` ngay mà không chờ và block thread.
- `add(E e)`: chèn phần tử vào cuối queue. Nếu queue đầy thì ném exception `IllegalStateException`; bên dưới dựa trên method `offer(E e)`.
- `remove()`: xóa phần tử ở đầu queue. Nếu queue rỗng thì ném exception `NoSuchElementException`; bên dưới dựa trên `poll()`.
- `peek()`: lấy nhưng không xóa phần tử ở đầu queue. Nếu queue rỗng, method trả về `null` ngay mà không chờ và block thread.

Trước tiên xem method `offer`. Logic của nó gần giống `put`, điểm khác duy nhất là khi thêm phần tử thất bại, thread hiện tại không bị block mà trả về `false` ngay.

```java
public boolean offer(E e) {
        // Đảm bảo phần tử được chèn không phải null
        checkNotNull(e);
        // Lấy lock
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
             // Queue đầy thì trả về false ngay
            if (count == items.length)
                return false;
            else {
                // Ngược lại thêm phần tử vào queue và trả về true ngay
                enqueue(e);
                return true;
            }
        } finally {
            // Giải phóng lock
            lock.unlock();
        }
    }
```

Method `poll` cũng tương tự: khi lấy phần tử thất bại, nó trả về rỗng ngay và không block thread đang lấy phần tử.

```java
public E poll() {
        final ReentrantLock lock = this.lock;
        // Lock
        lock.lock();
        try {
            // Nếu queue rỗng thì trả về null ngay, ngược lại lấy phần tử khỏi queue và trả về giá trị
            return (count == 0) ? null : dequeue();
        } finally {
            lock.unlock();
        }
    }
```

Method `add` thực chất chỉ bọc thêm một lớp quanh `offer`, như code dưới đây. Có thể thấy `add` gọi `offer` không giới hạn thời gian; nếu thêm phần tử thất bại thì ném exception ngay.

```java
public boolean add(E e) {
        return super.add(e);
}


public boolean add(E e) {
        // Gọi method offer, nếu thất bại thì ném exception ngay
        if (offer(e))
            return true;
        else
            throw new IllegalStateException("Queue full");
}
```

Method `remove` cũng tương tự, gọi `poll`; nếu trả về `null` thì nghĩa là queue không có phần tử và ném exception ngay.

```java
public E remove() {
        E x = poll();
        if (x != null)
            return x;
        else
            throw new NoSuchElementException();
}
```

Logic của method `peek()` cũng rất đơn giản, bên trong gọi method `itemAt`.

```java
public E peek() {
        // Lock
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            // Khi queue rỗng thì trả về null
            return itemAt(takeIndex);
        } finally {
            // Giải phóng lock
            lock.unlock();
        }
    }

// Trả về phần tử tại vị trí được chỉ định trong queue
@SuppressWarnings("unchecked")
final E itemAt(int i) {
    return (E) items[i];
}
```

### Lấy và thêm phần tử dạng block trong thời gian timeout được chỉ định

Dựa trên việc lấy và thêm phần tử dạng non-block bằng `offer(E e)` và `poll()`, nhà thiết kế cung cấp `offer(E e, long timeout, TimeUnit unit)` và `poll(long timeout, TimeUnit unit)` có thời gian chờ, dùng để thêm và lấy phần tử dạng block trong thời gian timeout được chỉ định.

```java
 public boolean offer(E e, long timeout, TimeUnit unit)
        throws InterruptedException {

        checkNotNull(e);
        long nanos = unit.toNanos(timeout);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
        // Queue đầy, đi vào vòng lặp
            while (count == items.length) {
            // Nếu hết thời gian mà queue vẫn đầy thì trả về false ngay
                if (nanos <= 0)
                    return false;
                 // Block trong thời gian nanos, chờ queue không đầy
                nanos = notFull.awaitNanos(nanos);
            }
            enqueue(e);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

Có thể thấy khi queue đầy, method `offer` có timeout sẽ chờ trong khoảng thời gian người dùng truyền vào. Nếu vẫn không thể lưu phần tử trong thời gian quy định thì trả về `false` ngay.

```java
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
        long nanos = unit.toNanos(timeout);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
          // Queue rỗng, lặp để chờ; nếu hết thời gian mà vẫn rỗng thì trả về null ngay
            while (count == 0) {
                if (nanos <= 0)
                    return null;
                nanos = notEmpty.awaitNanos(nanos);
            }
            return dequeue();
        } finally {
            lock.unlock();
        }
    }
```

Tương tự, `poll` có timeout cũng hoạt động như vậy: nếu queue rỗng thì chờ trong thời gian quy định; nếu hết thời gian mà vẫn rỗng thì trả về `null`.

### Kiểm tra phần tử có tồn tại hay không

`ArrayBlockingQueue` cung cấp `contains(Object o)` để kiểm tra phần tử được chỉ định có tồn tại trong queue hay không.

```java
public boolean contains(Object o) {
    // Nếu phần tử mục tiêu là null thì trả về false ngay
    if (o == null) return false;
    // Lấy array phần tử hiện tại của queue
    final Object[] items = this.items;
    // Lock
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        // Nếu queue không rỗng
        if (count > 0) {
            final int putIndex = this.putIndex;
            // Duyệt từ đầu queue
            int i = takeIndex;
            do {
                if (o.equals(items[i]))
                    return true;
                if (++i == items.length)
                    i = 0;
            } while (i != putIndex);
        }
        return false;
    } finally {
        // Giải phóng lock
        lock.unlock();
    }
}
```

## So sánh các method lấy và thêm phần tử của ArrayBlockingQueue

Để giúp hiểu rõ hơn về `ArrayBlockingQueue`, hãy so sánh các method lấy và thêm phần tử đã đề cập ở trên.

Thêm phần tử:

| Method                                    | Cách xử lý khi queue đầy                                                       | Giá trị trả về |
| ----------------------------------------- | ------------------------------------------------------------------------------ | -------------- |
| `put(E e)`                                | Thread bị block cho đến khi interrupt hoặc được đánh thức                      | void           |
| `offer(E e)`                              | Trả về false ngay                                                              | boolean        |
| `offer(E e, long timeout, TimeUnit unit)` | Block trong timeout chỉ định, quá thời gian mà chưa thêm được thì trả về false | boolean        |
| `add(E e)`                                | Ném exception `IllegalStateException` ngay                                     | boolean        |

Lấy/xóa phần tử:

| Method                              | Cách xử lý khi queue rỗng                                               | Giá trị trả về |
| ----------------------------------- | ----------------------------------------------------------------------- | -------------- |
| `take()`                            | Thread bị block cho đến khi interrupt hoặc được đánh thức               | E              |
| `poll()`                            | Trả về null                                                             | E              |
| `poll(long timeout, TimeUnit unit)` | Block trong timeout chỉ định, quá thời gian mà vẫn rỗng thì trả về null | E              |
| `peek()`                            | Trả về null                                                             | E              |
| `remove()`                          | Ném exception `NoSuchElementException` ngay                             | boolean        |

![](https://oss.javaguide.cn/github/javaguide/java/collection/ArrayBlockingQueue-get-add-element-methods.png)

## Câu hỏi phỏng vấn liên quan đến ArrayBlockingQueue

### ArrayBlockingQueue là gì? Đặc điểm của nó là gì?

`ArrayBlockingQueue` là class triển khai bounded queue của interface `BlockingQueue`, thường dùng để chia sẻ dữ liệu giữa nhiều thread, bên dưới dùng array để triển khai, đúng như tên gọi.

Capacity của `ArrayBlockingQueue` có giới hạn; sau khi tạo, capacity không thể thay đổi.

Để đảm bảo thread safety, `ArrayBlockingQueue` dùng reentrant lock `ReentrantLock` để kiểm soát concurrency. Cả thao tác chèn và thao tác đọc đều cần lấy lock mới có thể thực hiện. Ngoài ra, nó hỗ trợ hai cơ chế truy cập lock là fair và unfair, mặc định là unfair lock.

Mặc dù tên là blocking queue, `ArrayBlockingQueue` cũng hỗ trợ lấy và thêm phần tử dạng non-block (ví dụ method `poll()` và `offer(E e)`). Khi queue đầy, `offer(E e)` trả về `false`; khi queue rỗng, `poll()` trả về `null`.

### Sự khác nhau giữa ArrayBlockingQueue và LinkedBlockingQueue là gì?

`ArrayBlockingQueue` và `LinkedBlockingQueue` là hai triển khai blocking queue thường dùng trong package concurrency của Java, cả hai đều thread-safe. Tuy nhiên, giữa chúng có các điểm khác nhau sau:

- Triển khai bên dưới: `ArrayBlockingQueue` dựa trên array, còn `LinkedBlockingQueue` dựa trên linked list.
- Có giới hạn hay không: `ArrayBlockingQueue` là bounded queue, phải chỉ định capacity khi tạo. Khi tạo `LinkedBlockingQueue` có thể không chỉ định capacity, mặc định là `Integer.MAX_VALUE`, tức unbounded. Tuy nhiên cũng có thể chỉ định kích thước queue để biến nó thành bounded queue.
- Lock có tách biệt hay không: lock trong `ArrayBlockingQueue` không tách biệt, tức producer và consumer dùng cùng một lock; lock trong `LinkedBlockingQueue` tách biệt, producer dùng `putLock`, consumer dùng `takeLock`, nhờ đó giảm tranh chấp lock giữa producer thread và consumer thread.
- Mức sử dụng memory: `ArrayBlockingQueue` cần cấp phát trước memory cho array, còn `LinkedBlockingQueue` cấp phát động memory cho node của linked list. Điều này có nghĩa là `ArrayBlockingQueue` chiếm một lượng memory nhất định ngay khi tạo, và memory được cấp phát thường lớn hơn memory thực tế sử dụng; còn `LinkedBlockingQueue` dần chiếm memory theo số phần tử tăng lên.

### Sự khác nhau giữa ArrayBlockingQueue và ConcurrentLinkedQueue là gì?

`ArrayBlockingQueue` và `ConcurrentLinkedQueue` là hai triển khai queue thường dùng trong package concurrency của Java, cả hai đều thread-safe. Tuy nhiên, giữa chúng có các điểm khác nhau sau:

- Triển khai bên dưới: `ArrayBlockingQueue` dựa trên array, còn `ConcurrentLinkedQueue` dựa trên linked list.
- Có giới hạn hay không: `ArrayBlockingQueue` là bounded queue, phải chỉ định capacity khi tạo; còn `ConcurrentLinkedQueue` là unbounded queue và có thể tăng capacity động.
- Có block hay không: `ArrayBlockingQueue` hỗ trợ cả hai cách lấy và thêm phần tử dạng block và non-block (thường chỉ dùng cách thứ nhất); `ConcurrentLinkedQueue` là unbounded và chỉ hỗ trợ lấy, thêm phần tử dạng non-block.

### Nguyên lý triển khai của ArrayBlockingQueue là gì?

Nguyên lý triển khai của `ArrayBlockingQueue` chủ yếu gồm các điểm sau (ở đây lấy việc lấy và thêm phần tử dạng block làm ví dụ):

- Bên trong `ArrayBlockingQueue` duy trì một array có độ dài cố định để lưu phần tử.
- Dùng object lock `ReentrantLock` để đồng bộ hóa thao tác đọc và ghi, tức dùng cơ chế lock để đảm bảo thread safety.
- Dùng `Condition` để thực hiện thao tác chờ và đánh thức giữa các thread.

Sau đây giải thích chi tiết hơn cách thực hiện chờ và đánh thức giữa các thread (không cần ghi nhớ method cụ thể, khi phỏng vấn chỉ cần trả lời các ý chính):

- Khi queue đầy, producer thread gọi method `notFull.await()` để chờ, chờ queue không đầy rồi chèn phần tử (condition không đầy).
- Khi queue rỗng, consumer thread gọi method `notEmpty.await()` để chờ, chờ queue không rỗng rồi tiêu thụ (condition không rỗng).
- Khi có phần tử mới được thêm, producer thread gọi method `notEmpty.signal()` để đánh thức consumer thread đang chờ tiêu thụ.
- Khi có phần tử được lấy khỏi queue, consumer thread gọi method `notFull.signal()` để đánh thức producer thread đang chờ chèn phần tử.

Bổ sung về interface `Condition`:

> `Condition` chỉ có từ sau JDK1.5 và có tính linh hoạt cao. Ví dụ, nó có thể thực hiện chức năng thông báo theo nhiều hướng, tức là có thể tạo nhiều instance `Condition` (object monitor) trong một object `Lock`. **Thread object có thể đăng ký vào `Condition` được chỉ định, từ đó có thể thông báo thread có chọn lọc, linh hoạt hơn trong việc điều phối thread. Khi dùng method `notify()/notifyAll()` để thông báo, thread được thông báo do JVM lựa chọn. Dùng class `ReentrantLock` kết hợp với instance `Condition` có thể thực hiện “thông báo có chọn lọc”.** Tính năng này rất quan trọng và được interface `Condition` cung cấp mặc định. Còn keyword `synchronized` tương đương với việc toàn bộ object `Lock` chỉ có một instance `Condition`, mọi thread đều đăng ký vào cùng instance đó. Nếu thực thi method `notifyAll()`, tất cả thread đang ở trạng thái chờ sẽ được thông báo, gây ra vấn đề hiệu năng lớn. Trong khi đó, method `signalAll()` của instance `Condition` chỉ đánh thức tất cả thread đang chờ đã đăng ký trong instance `Condition` đó.

## Tài liệu tham khảo

- Series Tìm hiểu chuyên sâu Java | Giải thích chi tiết cách dùng BlockingQueue: <https://juejin.cn/post/6999798721269465102>
- Giải thích từ cơ bản đến chuyên sâu về blocking queue BlockingQueue và triển khai điển hình ArrayBlockingQueue: <https://zhuanlan.zhihu.com/p/539619957>
- Tổng hợp kiến thức concurrency: nguyên lý bên dưới và thực chiến của ArrayBlockingQueue: <https://zhuanlan.zhihu.com/p/339662987>
<!-- @include: @article-footer.snippet.md -->
