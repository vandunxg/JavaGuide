---
title: "Giải thích chi tiết về Java lock: mutex, read-write lock, spinlock và tối ưu synchronized"
description: "Tổng hợp có hệ thống về cơ chế Java lock: từ mutex, read-write lock, spinlock đến synchronized, ReentrantLock, AQS, StampedLock; làm rõ phân loại lock, nguyên lý triển khai, khác biệt giữa các phiên bản và khuyến nghị lựa chọn."
category: Java
tag:
  - Java Concurrency
head:
  - - meta
    - name: keywords
      content: Java lock,mutex,read-write lock,spinlock,synchronized,ReentrantLock,AQS,StampedLock,CAS,lock upgrade,lock optimization,Java Concurrency
---

Khi học Java Concurrency, các tên gọi liên quan đến lock rất dễ khiến mọi người nhầm lẫn: mutex, read-write lock, spinlock, pessimistic lock, optimistic lock, CAS, AQS, `synchronized`, `ReentrantLock`, `StampedLock`, biased lock, lightweight lock, heavyweight lock.

Các tên gọi này không nằm trên cùng một chiều phân loại.

Có loại nói về “ai có thể vào critical section”, chẳng hạn mutex và read-write lock; có loại nói về “phải chờ thế nào khi không lấy được lock”, chẳng hạn spinlock và blocking lock; có loại nói về “khóa trước khi sửa shared data hay kiểm tra lại khi commit”, chẳng hạn pessimistic lock và optimistic lock; cũng có loại nói về cách HotSpot tối ưu `synchronized` khi mức độ cạnh tranh khác nhau.

Bài viết này trước hết xây dựng hệ quy chiếu về lock, sau đó xem các công cụ lock thường dùng trong Java được triển khai thế nào. Về chi tiết của pessimistic lock, optimistic lock và CAS, trên site đã có hai bài viết giải thích cụ thể: [Giải thích chi tiết về optimistic lock và pessimistic lock](./optimistic-lock-and-pessimistic-lock.md), [Giải thích chi tiết về CAS](./cas.md). Bài viết này chỉ giữ lại ngữ cảnh cần thiết, tập trung vào “xâu chuỗi hệ thống lock thế nào”.

PS: Bài viết chủ yếu lấy HotSpot / OpenJDK làm bối cảnh. Mutex monitor và memory semantics của `synchronized` thuộc tầng đặc tả Java/JVM; object header, Mark Word, lightweight lock, lock inflation thuộc tối ưu triển khai của HotSpot, đặc tả ngôn ngữ Java không cam kết quy trình cố định. Biased lock bị vô hiệu hóa mặc định từ JDK 15 và các tham số liên quan bị deprecated; từ JDK 18, các tham số liên quan đã bị obsoleted; kết luận về virtual thread và `synchronized` pinning cũng cần phân biệt JDK 21~23 với JDK 24+.

Trước tiên, hãy tách các chiều phân loại bằng một bảng:

| Chiều                        | Tên gọi điển hình                                  | Trả lời câu hỏi                             |
| ---------------------------- | -------------------------------------------------- | ------------------------------------------- |
| Cách mutex critical section  | mutex, read-write lock                             | Ai có thể vào critical section              |
| Chiến lược chờ               | spinlock, blocking lock                            | Phải chờ thế nào khi không lấy được lock    |
| Tư duy kiểm soát concurrency | pessimistic lock, optimistic lock                  | Khóa trước rồi sửa hay kiểm tra khi commit  |
| Cơ chế cập nhật atomic       | CAS, Atomic class                                  | Cập nhật một biến không blocking thế nào    |
| Tối ưu triển khai JVM        | lightweight lock, heavyweight lock, lock inflation | HotSpot giảm chi phí `synchronized` thế nào |
| Công cụ Java lock            | `synchronized`, `ReentrantLock`, `StampedLock`     | Cụ thể dùng gì trong code                   |

## Một lock thực sự bảo vệ điều gì?

Lock giải quyết vấn đề critical section. Critical section là đoạn code truy cập shared mutable state và không thể cho nhiều execution unit thực thi xen kẽ tùy ý.

![Sơ đồ minh họa giao thức bảo vệ critical section: nhiều thread truy cập shared state thông qua một cổng lock thống nhất; bỏ qua lock hoặc thay lock object sẽ phá vỡ quan hệ mutex](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/os-lock-critical-section.png)

Ví dụ phép tăng sau đây:

```java
count++;
```

Trong source code chỉ có một dòng, nhưng dòng code này không thể xem là một thao tác không thể tách rời. Thread thường phải đọc `count` trước, sau đó cộng 1, cuối cùng ghi lại. Khi hai thread thực thi đồng thời, cả hai có thể cùng đọc giá trị cũ `0`, mỗi thread tính ra `1`, rồi cuối cùng cùng ghi `1` trở lại. Cả hai thread đều thực hiện phép tăng, nhưng kết quả chỉ tăng một lần.

Cách lock hoạt động rất trực tiếp: lấy lock trước khi vào đoạn code này và giải phóng sau khi thực thi xong. Chỉ cần mọi code truy cập cùng một shared state tuân thủ quy ước dùng cùng một lock, các thao tác đọc ghi vốn có thể xen kẽ sẽ được gộp thành một đoạn thực thi mutex.

Có một câu rất dễ bị bỏ qua: **lock thực sự bảo vệ protocol truy cập object state; bản thân object không tự động an toàn chỉ vì được lock**.

`synchronized (account)` không thần kỳ khiến mọi field của `account` an toàn. Nếu một đoạn code khác bỏ qua lock này và sửa trực tiếp `account.balance`, thread safety vẫn bị phá vỡ. Khóa học về lock của MIT 6.005 cũng nhiều lần nhấn mạnh điểm này: lock phải bảo vệ representation invariant của một data abstraction; tùy tiện tìm một object để bọc vào không thể đảm bảo invariant luôn đúng.

## Mutex: mỗi thời điểm chỉ cho phép một thread vào

Quy tắc của mutex rất đơn giản: tại một thời điểm, nhiều nhất chỉ một thread giữ lock và vào critical section.

Trong Java, `synchronized` và `ReentrantLock` đều có thể được dùng làm mutex:

```java
class Counter {
    private int count;

    public synchronized void increment() {
        count++;
    }

    public synchronized int get() {
        return count;
    }
}
```

Đổi sang `ReentrantLock`, cách viết sẽ dài dòng hơn một chút nhưng có được nhiều quyền kiểm soát hơn:

```java
class Counter {
    private final ReentrantLock lock = new ReentrantLock();
    private int count;

    public void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock();
        }
    }
}
```

Không được bỏ `try/finally`. Hành động giải phóng của `synchronized` do JVM thực hiện giúp bạn; dù code block kết thúc bình thường hay do exception, monitor đều được giải phóng. `ReentrantLock` là explicit API, việc lấy lock và unlock phải tự ghép cặp. Tài liệu `ReentrantLock` của Oracle cũng đưa việc “ngay lập tức vào block `try` sau khi gọi `lock`” làm cách viết được khuyến nghị.

Điểm thực sự khó của mutex không nằm ở cú pháp mà nằm ở lock granularity.

Một lock lớn bao toàn bộ thao tác sẽ dễ quản lý nhất nhưng concurrency thấp; nhiều lock nhỏ lần lượt bảo vệ các data khác nhau có thể cho throughput tốt hơn, nhưng thứ tự lock, deadlock và consistency của state đều khó quản lý hơn. OSTEP cũng đề cập trade-off này khi nói về POSIX mutex: dùng lock khác nhau cho data khác nhau có thể tăng concurrency, nhưng programmer phải biết rõ mỗi lock bảo vệ phần state nào.

## Read-write lock: read-read chia sẻ, write độc quyền

Mutex cũng rất nghiêm ngặt với thao tác đọc: chỉ cần một thread đang đọc thì thread khác cũng không được vào đọc. Nhưng nhiều business object có một đặc điểm: đọc không thay đổi state, nhiều read thread chạy đồng thời không làm hỏng lẫn nhau.

Read-write lock được tạo ra cho tình huống này.

Nó chia việc truy cập thành hai loại:

- Read lock: shared lock, nhiều thread có thể giữ đồng thời.
- Write lock: exclusive lock, chỉ một thread được giữ, đồng thời write lock và read lock là mutex.

Các quy tắc tương ứng cũng rất dễ nhớ:

- Read-read không mutex.
- Read-write mutex.
- Write-write mutex.

Triển khai điển hình trong Java là `ReentrantReadWriteLock`:

```java
class ProfileCache {
    private final ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
    private final Lock readLock = rwLock.readLock();
    private final Lock writeLock = rwLock.writeLock();
    private final Map<Long, String> cache = new HashMap<>();

    public String get(long userId) {
        readLock.lock();
        try {
            return cache.get(userId);
        } finally {
            readLock.unlock();
        }
    }

    public void put(long userId, String profile) {
        writeLock.lock();
        try {
            cache.put(userId, profile);
        } finally {
            writeLock.unlock();
        }
    }
}
```

Read-write lock phù hợp với scenario nhiều đọc ít ghi, thao tác đọc đủ ngắn và data structure khó bị phá vỡ giữa chừng. Nó không phù hợp với mọi cache và cũng không nhất định nhanh hơn mutex. Nếu thao tác ghi thường xuyên, read thread và write thread sẽ liên tục chặn nhau, chi phí duy trì read-write lock ngược lại có thể triệt tiêu lợi ích.

Java còn cung cấp `StampedLock`, hỗ trợ write lock, pessimistic read lock và optimistic read. Optimistic read không thực sự giữ traditional read lock; nó lấy một stamp trước, sau khi đọc xong mới kiểm tra trong thời gian đó có write xảy ra hay không:

```java
class Point {
    private final StampedLock lock = new StampedLock();
    private double x;
    private double y;

    double distanceFromOrigin() {
        long stamp = lock.tryOptimisticRead();
        double currentX = x;
        double currentY = y;
        if (!lock.validate(stamp)) {
            stamp = lock.readLock();
            try {
                currentX = x;
                currentY = y;
            } finally {
                lock.unlockRead(stamp);
            }
        }
        return Math.hypot(currentX, currentY);
    }
}
```

Optimistic read của `StampedLock` có giới hạn: data đọc được có thể tạm thời không nhất quán, vì vậy chỉ phù hợp với scenario đọc ngắn, có thể hoàn tất việc đọc trong local variable và có thể đọc lại để fallback khi `validate` thất bại. Nó cũng khó thay thế trực tiếp `ReentrantReadWriteLock`; đặc biệt cần lưu ý nó không hỗ trợ reentrancy.

## Spinlock: không blocking, chờ tại chỗ một lúc

Khi thread không lấy được lock, thường có hai cách chờ:

- Blocking: suspend thread hiện tại, để operating system đánh thức sau.
- Spin: không suspend thread, lặp trên CPU để kiểm tra lock đã khả dụng chưa.

Spinlock phù hợp với critical section cực ngắn. Chẳng hạn thread đang giữ lock sắp giải phóng lock; nếu waiting thread lập tức blocking, chi phí suspend và wake-up thread có thể cao hơn việc “quay vài vòng tại chỗ” để chờ.

Vấn đề cũng nằm ở đây: spin phải trả chi phí CPU và liên tục chiếm dụng CPU. Nếu lock lâu không được giải phóng hoặc có nhiều waiting thread, spin sẽ lãng phí CPU time vào việc quay rỗng.

Trong Java, có thể dùng CAS để viết một ví dụ spinlock rất nhỏ:

```java
class SpinLock {
    private final AtomicReference<Thread> owner = new AtomicReference<>();

    public void lock() {
        Thread current = Thread.currentThread();
        while (!owner.compareAndSet(null, current)) {
            Thread.onSpinWait();
        }
    }

    public void unlock() {
        Thread current = Thread.currentThread();
        if (!owner.compareAndSet(current, null)) {
            throw new IllegalMonitorStateException();
        }
    }
}
```

Đoạn code này chỉ dùng để giải thích quan hệ giữa “spin + CAS”, không nên lấy trực tiếp làm business lock. Lock thực tế phải cân nhắc reentrancy, fairness, interruption, timeout, giải phóng khi exception, metrics monitoring, waiting queue và các vấn đề khác. JDK đã đóng gói những phức tạp này trong `synchronized`, `ReentrantLock`, AQS synchronizer và Atomic class.

## Vị trí của pessimistic lock, optimistic lock và CAS

Pessimistic lock và optimistic lock mô tả hai tư duy kiểm soát concurrency, không tương ứng với một Java class cố định nào.

Pessimistic lock giả định xung đột có khả năng xảy ra cao, vì vậy lock resource trước rồi mới thao tác. `synchronized`, `ReentrantLock` và `SELECT ... FOR UPDATE` của database đều là những ví dụ phổ biến.

Optimistic lock giả định xung đột không thường xuyên, trước tiên không blocking người khác, đến khi commit thay đổi mới kiểm tra data đã bị sửa hay chưa. Field `version` trong database và CAS trong Java Atomic class đều thuộc hướng này.

CAS (Compare-And-Swap, compare và swap) có thể hiểu là một cách atomic update được hardware hỗ trợ: chỉ khi value trong memory vẫn bằng expected old value thì mới đổi nó thành new value. Nếu không, nghĩa là có người đã sửa trước; thread hiện tại có thể chọn retry, từ bỏ hoặc chuyển sang logic fallback.

CAS thường có ba vấn đề:

- Retry khi thất bại sẽ tiêu tốn CPU; xung đột càng gay gắt thì càng rõ.
- Chỉ xử lý tự nhiên một variable; consistency của nhiều variable cần được thiết kế thêm.
- Vấn đề ABA: value đổi từ A sang B rồi quay lại A, chỉ nhìn value sẽ tưởng rằng nó chưa từng thay đổi.

Có thể giải quyết ABA bằng version number, timestamp hoặc marked reference. Trong Java có các công cụ như `AtomicStampedReference` và `AtomicMarkableReference`, nhưng trong business code, cách thường gặp hơn là để data model tự mang version number.

Nếu mở rộng phần này, nội dung sẽ trùng với các bài viết hiện có. Muốn xem cách triển khai, ví dụ version number, xử lý ABA và source code của Atomic class, có thể đọc tiếp:

- [Giải thích chi tiết về optimistic lock và pessimistic lock](./optimistic-lock-and-pessimistic-lock.md)
- [Giải thích chi tiết về CAS](./cas.md)
- [Tổng hợp Atomic class](./atomic-classes.md)

## Tầng dưới của synchronized: monitor, bytecode và memory semantics

`synchronized` là cơ chế synchronization tích hợp trong ngôn ngữ Java, có thể áp dụng cho instance method, static method và cũng có thể bao quanh code block.

```java
class Account {
    private long balance;

    public synchronized void deposit(long amount) {
        balance += amount;
    }

    public long balance() {
        synchronized (this) {
            return balance;
        }
    }
}
```

Synchronized method và synchronized code block có biểu hiện không hoàn toàn giống nhau ở tầng bytecode:

- Synchronized method dựa vào method access flag `ACC_SYNCHRONIZED`.
- Synchronized code block sẽ sinh ra instruction `monitorenter` và `monitorexit`.

Dù biểu hiện thế nào, semantics đều là vào monitor và thoát monitor. Java Language Specification còn quy định quan hệ happens-before giữa việc giải phóng lock và lần lấy lock tiếp theo: các write của một thread trước khi giải phóng lock sẽ visible với thread lấy cùng lock sau đó.

Đây cũng là điểm khác giữa `synchronized` và lock khái niệm thông thường “chỉ làm mutex”. Nó đồng thời cung cấp mutex và memory visibility. Chỉ bảo vệ critical section nhưng không thiết lập visibility thì thread khác vẫn có thể đọc phải value cũ.

Ngoài ra, `synchronized` có tính reentrant. Khi một thread đã giữ monitor của một object, nó có thể tiếp tục vào code được bảo vệ bởi cùng lock; JVM ghi lại số lần reenter và giảm dần từng lớp khi thoát.

```java
class ReentrantDemo {
    public synchronized void outer() {
        inner();
    }

    public synchronized void inner() {
        // Cùng một thread có thể vào monitor của this lần nữa
    }
}
```

## Tối ưu lock của synchronized: đừng học thuộc “lock upgrade” như một công thức cố định

Nhiều tài liệu giải thích `synchronized` theo chuỗi “unlocked -> biased lock -> lightweight lock -> heavyweight lock”. Manh mối này hữu ích để hiểu tối ưu thời kỳ đầu của HotSpot, nhưng không thể tách rời version.

Sau JDK 6, HotSpot đã tối ưu `synchronized` rất nhiều. Biased lock hướng đến scenario “luôn cùng một thread vào cùng một lock”; lightweight lock hướng đến scenario “cạnh tranh không gay gắt, nhiều thread vào lệch thời điểm”; heavyweight lock sẽ dùng ObjectMonitor, các thread cạnh tranh có thể blocking và wake-up.

Cần ghi nhớ riêng khác biệt giữa các version:

- JDK 6 đến JDK 14: biased lock là một trong những tối ưu phổ biến của HotSpot.
- JDK 15: JEP 374 vô hiệu hóa biased lock theo mặc định và deprecated các parameter liên quan.
- JDK 18: các parameter liên quan đến biased lock bị obsoleted; khi truyền vào chúng sẽ bị bỏ qua và đưa ra warning.
- JDK 21 đến JDK 23: virtual thread có thể pin platform thread khi blocking trong `synchronized`.
- JDK 24: JEP 491 cải thiện sự phối hợp giữa virtual thread và `synchronized`; virtual thread blocking trên `synchronized` có thể giải phóng platform thread bên dưới, giảm vấn đề pinning.

Vì vậy, khi phỏng vấn hoặc viết bài, có thể nói “HotSpot từng giảm chi phí `synchronized` bằng biased lock, lightweight lock và heavyweight lock”, nhưng không nên nói biased lock là default path mà modern JDK nhất định sẽ đi qua.

Về engineering, một kết luận khác quan trọng hơn: câu “không dùng được thì đừng dùng” từ những năm đầu không còn phù hợp với `synchronized` hiện nay. Trong scenario mutex thông thường, nó có cú pháp đơn giản, giải phóng lock an toàn khi exception và JIT optimization trưởng thành. Chỉ khi cần fair lock, interruptible acquisition, timeout acquisition hoặc nhiều condition queue thì chuyển sang `ReentrantLock` mới tự nhiên hơn.

## Kết nối ReentrantLock, Condition và AQS thế nào

`ReentrantLock` cung cấp khả năng kiểm soát chi tiết hơn `synchronized`:

- Có thể chọn fair lock hoặc unfair lock.
- Có thể dùng `lockInterruptibly()` để phản hồi interruption.
- Có thể dùng `tryLock()` hoặc `tryLock(timeout, unit)` để tránh chờ vô hạn.
- Có thể tạo nhiều `Condition` để tách và quản lý các waiting condition khác nhau.

Cách viết điển hình như sau:

```java
class BoundedBuffer<E> {
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();
    private final Queue<E> queue = new ArrayDeque<>();
    private final int capacity;

    BoundedBuffer(int capacity) {
        this.capacity = capacity;
    }

    public void put(E item) throws InterruptedException {
        lock.lockInterruptibly();
        try {
            while (queue.size() == capacity) {
                notFull.await();
            }
            queue.add(item);
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    public E take() throws InterruptedException {
        lock.lockInterruptibly();
        try {
            while (queue.isEmpty()) {
                notEmpty.await();
            }
            E item = queue.remove();
            notFull.signal();
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```

`while` ở đây cũng không thể tùy tiện đổi thành `if`. Sau khi được đánh thức, thread chỉ có thể biết rằng nó “có cơ hội cạnh tranh lock lại và kiểm tra condition”, không có nghĩa condition chắc chắn đã đúng. Spurious wakeup, nhiều waiting thread cạnh tranh và condition bị thread khác tiêu thụ trước đều khiến thread phải kiểm tra lại sau khi thức dậy.

Tầng dưới của `ReentrantLock` dựa vào AQS (AbstractQueuedSynchronizer). Có thể tạm hiểu AQS là một synchronization framework: dùng `state` để biểu thị synchronization state, dùng FIFO queue để quản lý các thread chưa giành được resource, sau đó phối hợp CAS và `LockSupport.park/unpark` để xếp hàng, blocking và wake-up.

Nhiều concurrency tool được xây dựng trên AQS, chẳng hạn `ReentrantLock`, `Semaphore`, `CountDownLatch`, `ReentrantReadWriteLock`. Nếu muốn tiếp tục tách riêng mạch queue, `state`, CAS và blocking/wake-up, hãy đọc [Giải thích chi tiết về AQS](./aqs.md) và [Xem nguyên lý và ứng dụng của AQS từ cách triển khai ReentrantLock](./reentrantlock.md).

## Nên chọn Java lock thế nào?

Khi chọn lock, đừng vội so sánh “cái nào nhanh nhất”. Hiệu năng của lock liên quan đến độ dài critical section, mức độ cạnh tranh, số lượng thread và cách xử lý sau khi thất bại. Trước tiên hãy làm rõ vài câu hỏi: đoạn code này bảo vệ shared state nào? Thời gian giữ lock khoảng bao lâu? Không lấy được lock có thể chờ không? Sau khi chờ thất bại thì return, retry hay báo lỗi ngay?

Nếu chỉ bảo vệ một đoạn state nhỏ trong JVM process, chẳng hạn cập nhật vài field, duy trì một in-memory Map hoặc chuyển đổi object state, `synchronized` thường đã đủ. Nó ngắn gọn khi viết, tự động giải phóng lock lúc thoát code block và giảm rủi ro quên viết `unlock()`. Khi code cần timeout acquisition, interruptible acquisition, fair lock hoặc cần dùng nhiều `Condition` để quản lý các waiting queue khác nhau, chuyển sang `ReentrantLock` sẽ thuận tiện hơn.

Khi đọc nhiều ghi ít, có thể cân nhắc `ReentrantReadWriteLock`. Trọng tâm ở đây là “ghi ít”; chỉ có các read method thôi là chưa đủ. Nếu write operation thường xuyên, read lock và write lock sẽ liên tục chặn nhau, việc duy trì read-write state cũng có chi phí, cuối cùng chưa chắc có lợi hơn một mutex. Optimistic read của `StampedLock` kén scenario hơn: logic đọc phải ngắn, việc đọc phải state giữa chừng cũng không được gây vấn đề lớn, đồng thời phải chấp nhận đọc lại một lần sau khi validation thất bại.

Nếu chỉ cập nhật một counter, state bit hoặc reference, nên ưu tiên xem các công cụ như Atomic class, `LongAdder`, `LongAccumulator`. Chúng phù hợp với atomic update rất ngắn, không phù hợp để nhét cả một business flow vào CAS retry loop. Business flow càng dài, retry khi thất bại càng dễ làm CPU tiêu tốn vào vòng lặp vô hiệu và cũng khó xử lý side effect hơn.

Nếu vấn đề đã vượt qua ranh giới JVM, chẳng hạn nhiều application instance đồng thời sửa cùng một database row, lock trong Java không quản lý được nữa. Khi xung đột không thường xuyên, có thể dùng version number làm optimistic lock; khi xung đột tương đối thường xuyên và phải sửa với strong consistency, thường cần quay về database row lock, `SELECT ... FOR UPDATE`, unique constraint và các cơ chế database tương tự. Khi mở rộng đến mutex giữa các service, cần Redis, ZooKeeper, database hoặc external system khác đảm nhận; không thể trông chờ `synchronized` hay `ReentrantLock`.

Đừng mù quáng theo đuổi lock granularity “nhỏ”. Một lock lớn dễ đảm bảo correctness nhưng throughput có thể bị ảnh hưởng; tách thành nhiều lock nhỏ giúp giảm cạnh tranh, nhưng lock order, deadlock và chi phí troubleshooting đều tăng. Nhiều khi, trước hết dùng một lock rõ ràng để bảo vệ invariant, rồi dựa trên kết quả load test mà tách lock sẽ vững vàng hơn việc ngay từ đầu thiết kế một loạt fine-grained lock.

## Các lỗi thường gặp

**Lock object không ổn định.**

Một số code trông như đã thêm lock nhưng thực tế có thể lock vào các object khác nhau. Nguyên nhân thường gặp là lock object có thể thay đổi, chẳng hạn kết quả nối string, boxed object hoặc field có thể gán lại. Khi thread A vào, nó lock object cũ; khi thread B vào, nó lock object mới; hai bên không ảnh hưởng nhau và critical section bị tách rời.

```java
private Object lock = new Object();

public void update() {
    synchronized (lock) {
        lock = new Object(); // Các thread sau đó có thể lock vào lock khác
    }
}
```

Nếu cần một lock object riêng, thường hãy định nghĩa nó là `private final` và không expose cho code bên ngoài.

**Lock object visible với bên ngoài.**

Đôi khi `synchronized (this)` và `synchronized (SomeClass.class)` không có vấn đề, nhưng code bên ngoài cũng có thể lấy chúng để lock, khiến bạn không kiểm soát được phạm vi lock competition. Đặc biệt cần thận trọng trong library code; thông thường nên dùng private final lock object.

```java
private final Object lock = new Object();
```

**Thực hiện thao tác chậm trong thời gian giữ lock.**

Truy cập database, gọi RPC hoặc ghi file lớn trong khi giữ lock đều kéo dài thời gian chiếm dụng lock. Lock càng bị giữ lâu, waiting thread càng nhiều, rủi ro timeout, cạn thread pool và deadlock càng cao.

**Thứ tự lock không nhất quán.**

Hai thread lần lượt lấy lock theo thứ tự `A -> B` và `B -> A` rất dễ hình thành deadlock. Khi dùng nhiều lock đồng thời, cần đặt ra một thứ tự ổn định trên toàn cục cho resource. Có thể xem phần giới thiệu đầy đủ về deadlock tại [Giải thích chi tiết về deadlock](../../cs-basics/operating-system/dead-lock.md).

**Đánh đồng thread-safe class với composite operation.**

Các thao tác `get`, `put` đơn lẻ của `ConcurrentHashMap` là thread-safe, nhưng “kiểm tra trước xem có tồn tại rồi mới insert” là composite operation, cần dùng atomic method như `computeIfAbsent` hoặc synchronization bổ sung.

```java
// Không khuyến nghị: thread khác có thể insert giữa containsKey và put
if (!map.containsKey(key)) {
    map.put(key, createValue());
}

// Khuyến nghị: giao composite logic cho atomic method của ConcurrentHashMap
map.computeIfAbsent(key, ignored -> createValue());
```

**Chỉ nhìn lock mà không nhìn resource pool.**

Nhiều tình trạng “bị treo” trên production không liên quan đến Java lock deadlock; các thread có thể đều đang chờ database connection, HTTP connection, thread pool queue hoặc response từ external service. Nhìn thấy `WAITING` trong thread stack chỉ cho biết thread đang chờ; để phán đoán deadlock còn phải tìm được wait cycle ổn định.

## Tổng kết

Lock là tên gọi chung của một nhóm công cụ kiểm soát concurrency, đây vẫn là một khái niệm khá rộng.

Mutex và read-write lock trả lời “ai có thể vào critical section”; spinlock và blocking lock trả lời “không lấy được lock thì chờ thế nào”; pessimistic lock và optimistic lock trả lời “xử lý thế nào trước và sau khi xảy ra xung đột”; CAS và Atomic class giải quyết atomic update của một variable; `synchronized`, `ReentrantLock`, `StampedLock` và AQS là cách Java đưa những tư tưởng này vào code.

Khi thực sự viết code, trước tiên hãy tìm shared state và invariant, sau đó quyết định lock bảo vệ gì, lock granularity bao nhiêu, việc chờ có thể bị interrupt hay không và thất bại có thể retry hay không. Tool chỉ là phương tiện; điều thực sự cần bảo vệ là cùng một synchronization protocol: mọi path truy cập shared state đều phải tuân thủ nó.

## Tài liệu tham khảo

- [The Java Language Specification, Chapter 17. Threads and Locks](https://docs.oracle.com/javase/specs/jls/se24/html/jls-17.html)
- [The Java Virtual Machine Specification, monitorenter](https://docs.oracle.com/javase/specs/jvms/se24/html/jvms-6.html#jvms-6.5.monitorenter)
- [Oracle Java API: Lock](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/Lock.html)
- [Oracle Java API: ReentrantLock](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReentrantLock.html)
- [Oracle Java API: StampedLock](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/StampedLock.html)
- [OpenJDK JEP 374: Deprecate and Disable Biased Locking](https://openjdk.org/jeps/374)
- [OpenJDK JEP 491: Synchronize Virtual Threads without Pinning](https://openjdk.org/jeps/491)
- [OSTEP bản tiếng Trung: Locks](https://pages.cs.wisc.edu/~remzi/OSTEP/Chinese/28.pdf)
- [MIT 6.005: Locks and Synchronization](http://web.mit.edu/6.005/www/fa15/classes/23-locks/)
