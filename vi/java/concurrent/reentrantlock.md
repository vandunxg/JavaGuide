---
title: Nguyên lý và ứng dụng của AQS qua cách triển khai ReentrantLock
description: "Phân tích chuyên sâu nguyên lý ReentrantLock và AQS: giải thích chi tiết cách triển khai reentrant lock ReentrantLock, sự khác biệt giữa fair lock và unfair lock, quy trình lock và unlock dựa trên AQS, so sánh performance với synchronized."
category: Java
tag:
  - Java Concurrency
head:
  - - meta
    - name: keywords
      content: ReentrantLock,AQS,fair lock,unfair lock,reentrant lock,lock unlock,nguyên lý ReentrantLock,so sánh synchronized
---

> Bài viết được đăng lại từ: <https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html>
>
> Tác giả: Meituan Technical Team

Phần lớn các lớp đồng bộ trong Java (Semaphore, ReentrantLock, v.v.) đều được triển khai dựa trên AbstractQueuedSynchronizer (gọi tắt là AQS). AQS là một framework đơn giản, cung cấp chức năng quản lý atomic đối với trạng thái đồng bộ, block và đánh thức thread, cùng mô hình queue.

Bài viết sẽ đi dần từ application layer xuống principle layer, đồng thời phân tích chuyên sâu kiến thức về exclusive lock liên quan đến AQS thông qua các đặc tính cơ bản của ReentrantLock và mối liên hệ giữa ReentrantLock với AQS. Bài viết cũng sử dụng hình thức hỏi đáp để giúp bạn hiểu AQS. Do giới hạn về độ dài, bài viết này chủ yếu trình bày logic của exclusive lock và Sync Queue trong AQS, không trình bày phần shared lock và Condition Queue (trọng tâm của bài viết là phân tích nguyên lý AQS, ReentrantLock chỉ được giới thiệu ngắn gọn; nếu quan tâm, bạn có thể đọc source code của ReentrantLock).

> Phân tích source code trong bài viết dựa trên JDK 8. Cách triển khai bên trong AQS tiếp tục được phát triển: trong JDK 11 vẫn có thể thấy các field và method chính được đề cập trong bài viết, còn field của node cùng cách enqueue và waiting trong JDK 17 và version hiện tại đã có thay đổi tương đối lớn. Các ý tưởng cốt lõi về trạng thái đồng bộ, wait queue và việc acquire/release resource vẫn có thể dùng làm nền tảng để tìm hiểu.

## 1 ReentrantLock

### 1.1 Tổng quan đặc tính của ReentrantLock

ReentrantLock nghĩa là reentrant lock, tức một thread có thể lock lặp lại một critical resource. Để giúp bạn hiểu rõ hơn về đặc tính của ReentrantLock, trước hết chúng ta so sánh ReentrantLock với Synchronized thường dùng; các đặc tính như sau (phần màu xanh là nội dung được phân tích chính trong bài viết này):

![](https://p0.meituan.net/travelcube/412d294ff5535bbcddc0d979b2a339e6102264.png)

Dưới đây là phần so sánh trực quan hơn bằng pseudocode:

```java
// **************************Cách sử dụng Synchronized**************************
// 1.Dùng cho code block
synchronized (this) {}
// 2.Dùng cho object
synchronized (object) {}
// 3.Dùng cho method
public synchronized void test () {}
// 4.Reentrant
for (int i = 0; i < 100; i++) {
  synchronized (this) {}
}
// **************************Cách sử dụng ReentrantLock**************************
public void test () throws Exception {
  // 1.Khởi tạo và chọn fair lock hoặc unfair lock
  ReentrantLock lock = new ReentrantLock(true);
  // 2.Có thể dùng cho code block
  lock.lock();
  try {
    // 3.Hỗ trợ nhiều cách lock, khá linh hoạt; có đặc tính reentrant
    if (lock.tryLock(100, TimeUnit.MILLISECONDS)) {
      try {
        // Logic thực thi sau khi lấy lock lần thứ hai
      } finally {
        // Mỗi lần lock thành công phải tương ứng với một lần release
        lock.unlock();
      }
    }
  } finally {
    lock.unlock();
  }
}
```

### 1.2 Mối liên hệ giữa ReentrantLock và AQS

Qua phần trên, chúng ta đã biết ReentrantLock hỗ trợ fair lock và unfair lock (để phân tích nguyên lý của fair lock và unfair lock, có thể tham khảo bài viết 《[Chuyện về “lock” trong Java không thể không nói](https://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=2651749434&idx=3&sn=5ffa63ad47fe166f2f1a9f604ed10091&chksm=bd12a5778a652c61509d9e718ab086ff27ad8768586ea9b38c3dcf9e017a8e49bcae3df9bcc8&scene=38#wechat_redirect)》), đồng thời tầng dưới của ReentrantLock được triển khai bởi AQS. Vậy ReentrantLock liên kết với AQS thông qua fair lock và unfair lock như thế nào? Chúng ta sẽ tập trung tìm hiểu mối quan hệ giữa chúng với AQS từ quy trình lock của hai loại lock này (mối liên hệ với AQS trong quy trình lock khá rõ ràng, quy trình unlock sẽ được giới thiệu sau).

Quy trình lock trong source code của unfair lock như sau:

```java
// java.util.concurrent.locks.ReentrantLock#NonfairSync

// Unfair lock
static final class NonfairSync extends Sync {
  ...
  final void lock() {
    if (compareAndSetState(0, 1))
      setExclusiveOwnerThread(Thread.currentThread());
    else
      acquire(1);
    }
  ...
}
```

Ý nghĩa của đoạn code này là:

- Nếu dùng CAS để set variable State (trạng thái đồng bộ) thành công, tức lấy lock thành công, thì set thread hiện tại thành exclusive thread.
- Nếu dùng CAS để set variable State (trạng thái đồng bộ) thất bại, tức lấy lock thất bại, thì đi vào method Acquire để xử lý tiếp.

Bước đầu tiên khá dễ hiểu, nhưng sau khi lấy lock thất bại ở bước thứ hai thì strategy xử lý tiếp theo là gì? Có thể có các suy nghĩ sau:

- Quy trình tiếp theo sau khi một thread lấy lock thất bại là gì? Có hai khả năng:

(1) Set kết quả lấy lock của thread hiện tại thành thất bại, kết thúc quy trình lấy lock. Cách thiết kế này sẽ làm giảm mạnh concurrency của hệ thống, không đáp ứng nhu cầu thực tế. Vì vậy cần quy trình dưới đây, tức quy trình xử lý của framework AQS.

(2) Có một cơ chế xếp hàng chờ nào đó, thread tiếp tục waiting và vẫn giữ khả năng lấy lock, quy trình lấy lock vẫn tiếp tục.

- Với trường hợp thứ hai của vấn đề 1, vì đã nói đến cơ chế xếp hàng chờ thì nhất định sẽ hình thành một queue nào đó; queue này dùng data structure gì?
- Thread đang trong cơ chế xếp hàng chờ có thể có cơ hội lấy lock vào lúc nào?
- Nếu thread đang trong cơ chế xếp hàng chờ mãi không thể lấy lock thì có phải tiếp tục waiting không, hay có strategy khác để giải quyết vấn đề này?

Với những vấn đề của unfair lock, hãy xem tiếp cách fair lock lấy lock trong source code:

```java
// java.util.concurrent.locks.ReentrantLock#FairSync

static final class FairSync extends Sync {
  ...
  final void lock() {
    acquire(1);
  }
  ...
}
```

Nhìn vào đoạn code này, có thể chúng ta sẽ thắc mắc: function Lock lock bằng method Acquire, nhưng cụ thể nó lock như thế nào?

Kết hợp quy trình lock của fair lock và unfair lock, tuy quy trình có một số điểm khác nhau nhưng đều gọi method Acquire, còn method Acquire là method cốt lõi trong AQS, class cha của FairSync và UnfairSync.

Đối với các vấn đề đã nêu ở trên, thực ra không thể giải đáp trong source code của class ReentrantLock; đáp án nằm trong class chứa method Acquire, tức AbstractQueuedSynchronizer, cũng chính là trọng tâm của bài viết này — AQS. Tiếp theo chúng ta sẽ giới thiệu chi tiết AQS và mối liên hệ giữa ReentrantLock với AQS (đáp án cho các vấn đề liên quan sẽ được giải đáp ở mục 2.3.5).

## 2 AQS

Trước hết, hãy dùng sơ đồ kiến trúc dưới đây để hiểu tổng thể framework AQS:

![](https://p1.meituan.net/travelcube/82077ccf14127a87b77cefd1ccf562d3253591.png)

- Trong hình trên, phần có màu là Method, phần không có màu là Attribution.
- Nhìn chung, framework AQS được chia thành năm tầng, từ trên xuống dưới đi từ nông đến sâu, từ API mà AQS expose ra bên ngoài đến data cơ sở ở tầng dưới.
- Khi một custom synchronizer được tích hợp, chỉ cần override một phần method cần thiết ở tầng thứ nhất, không cần quan tâm đến flow triển khai cụ thể ở tầng dưới. Khi custom synchronizer thực hiện lock hoặc unlock, trước hết đi qua API ở tầng thứ nhất để vào method bên trong AQS, sau đó qua tầng thứ hai để acquire lock; tiếp theo, đối với flow lấy lock thất bại, đi vào phần xử lý wait queue ở tầng thứ ba và thứ tư. Các cách xử lý này đều phụ thuộc vào tầng cung cấp data cơ sở ở tầng thứ năm.

Tiếp theo chúng ta sẽ phân tích framework AQS lần lượt từ tổng thể đến chi tiết, từ flow đến method; quá trình phân tích chính như sau:

![](https://p1.meituan.net/travelcube/d2f7f7fffdc30d85d17b44266c3ab05323338.png)

### 2.1 Tổng quan nguyên lý

Ý tưởng cốt lõi của AQS là: nếu shared resource được request đang rảnh, thì set thread đang request resource thành worker thread hợp lệ và set shared resource thành trạng thái locked; nếu shared resource đang bị chiếm dụng, cần một cơ chế block, waiting và wakeup nhất định để bảo đảm việc phân phối lock. Cơ chế này chủ yếu được triển khai bằng một biến thể của CLH queue, đưa các thread tạm thời chưa lấy được lock vào queue.

CLH: queue Craig, Landin and Hagersten, là một singly linked list; queue trong AQS là một virtual doubly linked queue (FIFO) dạng biến thể của CLH, AQS triển khai việc phân phối lock bằng cách đóng gói mỗi thread request shared resource thành một node.

Sơ đồ nguyên lý chính như sau:

![](https://p0.meituan.net/travelcube/7132e4cef44c26f62835b197b239147b18062.png)

AQS sử dụng một member variable kiểu int có tính Volatile để biểu thị trạng thái đồng bộ, dùng FIFO queue bên trong để thực hiện việc xếp hàng acquire resource, và dùng CAS để hoàn tất việc thay đổi giá trị State.

#### 2.1.1 Data structure của AQS

Trước hết xem data structure cơ bản nhất trong AQS — Node; Node chính là node trong queue dạng biến thể CLH ở trên.

![](https://p1.meituan.net/travelcube/960271cf2b5c8a185eed23e98b72c75538637.png)

Giải thích ý nghĩa của một số method và attribute:

| Method và attribute | Ý nghĩa                                                                                                                                 |
| :------------------ | :-------------------------------------------------------------------------------------------------------------------------------------- |
| waitStatus          | Trạng thái của node hiện tại trong queue                                                                                                |
| thread              | Thread nằm tại node đó                                                                                                                  |
| prev                | Pointer trỏ đến node trước                                                                                                              |
| predecessor         | Trả về node trước, nếu không có thì throw npe                                                                                           |
| nextWaiter          | Trỏ đến node tiếp theo đang ở trạng thái CONDITION (bài viết này không trình bày Condition Queue nên không giới thiệu thêm pointer này) |
| next                | Pointer trỏ đến node sau                                                                                                                |

Hai mode lock của thread:

| Mode      | Ý nghĩa                                 |
| :-------- | :-------------------------------------- |
| SHARED    | Thread waiting lock theo shared mode    |
| EXCLUSIVE | Thread waiting lock theo exclusive mode |

waitStatus có các enum value sau:

| Enum      | Ý nghĩa                                                                       |
| :-------- | :---------------------------------------------------------------------------- |
| 0         | Giá trị mặc định khi một Node được initialize                                 |
| CANCELLED | Bằng 1, biểu thị request lấy lock của thread đã bị cancel                     |
| CONDITION | Bằng -2, biểu thị node đang trong wait queue, thread của node đang chờ wakeup |
| PROPAGATE | Bằng -3, field này chỉ được sử dụng khi thread hiện tại ở trạng thái SHARED   |
| SIGNAL    | Bằng -1, biểu thị thread đã sẵn sàng, chỉ chờ resource release                |

#### 2.1.2 Trạng thái đồng bộ State

Sau khi hiểu data structure, tiếp theo hãy tìm hiểu trạng thái đồng bộ State của AQS. AQS duy trì một field tên là state, nghĩa là trạng thái đồng bộ, được đánh dấu bằng Volatile, dùng để thể hiện tình trạng lock của critical resource hiện tại.

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

private volatile int state;
```

Dưới đây là một số method truy cập field này:

| Tên method                                                         | Mô tả                 |
| :----------------------------------------------------------------- | :-------------------- |
| protected final int getState()                                     | Lấy giá trị State     |
| protected final void setState(int newState)                        | Set giá trị State     |
| protected final boolean compareAndSetState(int expect, int update) | Update State bằng CAS |

Các method này đều được đánh dấu bằng Final, nghĩa là subclass không thể override chúng. Chúng ta có thể triển khai exclusive mode và shared mode của nhiều thread (quy trình lock) bằng cách thay đổi trạng thái đồng bộ được biểu thị bởi field State.

![](https://p0.meituan.net/travelcube/27605d483e8935da683a93be015713f331378.png)

![](https://p0.meituan.net/travelcube/3f1e1a44f5b7d77000ba4f9476189b2e32806.png)

Đối với custom synchronizer, cần custom cách acquire và release trạng thái đồng bộ, tức API layer ở tầng thứ nhất trong sơ đồ kiến trúc AQS.

### 2.2 Method quan trọng của AQS và mối liên hệ với ReentrantLock

Từ sơ đồ kiến trúc có thể biết AQS cung cấp nhiều Protected method để triển khai custom synchronizer. Các method liên quan trong custom synchronizer cũng chỉ nhằm thay đổi field State để triển khai exclusive mode hoặc shared mode cho nhiều thread. Custom synchronizer cần triển khai các method sau (các method ReentrantLock cần triển khai như dưới đây, không phải tất cả):

| Tên method                                  | Mô tả                                                                                                                                                                                          |
| :------------------------------------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| protected boolean isHeldExclusively()       | Thread hiện tại có đang độc chiếm resource không. Chỉ cần triển khai khi dùng Condition.                                                                                                       |
| protected boolean tryAcquire(int arg)       | Exclusive mode. arg là số lần lấy lock, thử acquire resource; thành công trả về True, thất bại trả về False.                                                                                   |
| protected boolean tryRelease(int arg)       | Exclusive mode. arg là số lần release lock, thử release resource; thành công trả về True, thất bại trả về False.                                                                               |
| protected int tryAcquireShared(int arg)     | Shared mode. arg là số lần lấy lock, thử acquire resource. Số âm biểu thị thất bại; 0 biểu thị thành công nhưng không còn resource khả dụng; số dương biểu thị thành công và vẫn còn resource. |
| protected boolean tryReleaseShared(int arg) | Shared mode. arg là số lần release lock, thử release resource; nếu sau khi release cho phép wakeup wait node tiếp theo thì trả về True, ngược lại trả về False.                                |

Thông thường, custom synchronizer hoặc là exclusive mode hoặc là shared mode, và chỉ cần triển khai một trong hai nhóm tryAcquire-tryRelease hoặc tryAcquireShared-tryReleaseShared. AQS cũng hỗ trợ custom synchronizer đồng thời triển khai cả exclusive mode và shared mode, ví dụ ReentrantReadWriteLock. ReentrantLock là exclusive lock nên triển khai tryAcquire-tryRelease.

Lấy unfair lock làm ví dụ, phần này chủ yếu trình bày mối liên hệ giữa unfair lock với các method của AQS; tác dụng của từng method cốt lõi sẽ được giải thích chi tiết ở phần sau.

![](https://p1.meituan.net/travelcube/b8b53a70984668bc68653efe9531573e78636.png)

> 🐛 Đính chính (tham khảo: [issue#1761](https://github.com/Snailclimb/JavaGuide/issues/1761)): một lỗi nhỏ trong hình, sau khi (AQS) CAS sửa shared resource State thành công thì phải là lấy lock thành công (unfair lock).
>
> Source code tương ứng như sau:
>
> ```java
> final boolean nonfairTryAcquire(int acquires) {
>          final Thread current = Thread.currentThread();//Lấy thread hiện tại
>          int c = getState();
>          if (c == 0) {
>              if (compareAndSetState(0, acquires)) {//CAS tranh lock
>                  setExclusiveOwnerThread(current);//Set thread hiện tại thành exclusive thread
>                  return true;//Tranh lock thành công
>              }
>          }
>          else if (current == getExclusiveOwnerThread()) {
>              int nextc = c + acquires;
>              if (nextc < 0) // overflow
>                  throw new Error("Maximum lock count exceeded");
>              setState(nextc);
>              return true;
>          }
>          return false;
>      }
> ```

Để giúp bạn hiểu interaction giữa method của ReentrantLock và AQS, lấy unfair lock làm ví dụ, chúng ta tách riêng flow interaction của lock và unlock để nhấn mạnh, qua đó thuận tiện cho việc hiểu nội dung phía sau.

![](https://p1.meituan.net/travelcube/7aadb272069d871bdee8bf3a218eed8136919.png)

Lock:

- Thực hiện lock thông qua method Lock của ReentrantLock.
- Sẽ gọi method Lock của inner class Sync. Vì Sync#lock là abstract method, tùy fair lock hoặc unfair lock được chọn khi khởi tạo ReentrantLock mà thực thi method Lock của inner class tương ứng; về bản chất đều sẽ thực thi method Acquire của AQS.
- Method Acquire của AQS sẽ thực thi method tryAcquire, nhưng vì tryAcquire cần do custom synchronizer triển khai nên sẽ thực thi method tryAcquire trong ReentrantLock. Vì ReentrantLock triển khai method tryAcquire thông qua inner class fair lock và unfair lock nên sẽ thực thi tryAcquire khác nhau tùy loại lock.
- tryAcquire là logic lấy lock; sau khi lấy lock thất bại sẽ thực thi logic tiếp theo của framework AQS, không liên quan đến custom synchronizer của ReentrantLock.

Unlock:

- Thực hiện unlock thông qua method Unlock của ReentrantLock.
- Unlock sẽ gọi method Release của inner class Sync; method này kế thừa từ AQS.
- Release sẽ gọi method tryRelease. tryRelease cần được custom synchronizer triển khai, và tryRelease chỉ được triển khai trong Sync của ReentrantLock, vì vậy có thể thấy quy trình release lock không phân biệt fair lock hay unfair lock.
- Sau khi release thành công, toàn bộ việc xử lý do framework AQS hoàn thành, không liên quan đến custom synchronizer.

Qua mô tả trên, có thể tổng kết mapping giữa các method cốt lõi ở API layer khi ReentrantLock lock và unlock.

![](https://p0.meituan.net/travelcube/f30c631c8ebbf820d3e8fcb6eee3c0ef18748.png)

## 3 Tìm hiểu AQS qua ReentrantLock

Fair lock và unfair lock trong ReentrantLock có tầng dưới giống nhau; phần này lấy unfair lock làm ví dụ phân tích.

Trong unfair lock có một đoạn code như sau:

```java
// java.util.concurrent.locks.ReentrantLock

static final class NonfairSync extends Sync {
  ...
  final void lock() {
    if (compareAndSetState(0, 1))
      setExclusiveOwnerThread(Thread.currentThread());
    else
      acquire(1);
  }
  ...
}
```

Hãy xem Acquire này được viết như thế nào:

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

public final void acquire(int arg) {
  if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
    selfInterrupt();
}
```

Tiếp theo xem method tryAcquire:

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

protected boolean tryAcquire(int arg) {
  throw new UnsupportedOperationException();
}
```

Có thể thấy đây chỉ là triển khai đơn giản của AQS; method triển khai việc lấy lock cụ thể do fair lock và unfair lock tương ứng tự triển khai (lấy ReentrantLock làm ví dụ). Nếu method này trả về True, nghĩa là thread hiện tại lấy lock thành công và không cần thực thi tiếp; nếu lấy lock thất bại thì cần thêm vào wait queue. Tiếp theo sẽ giải thích chi tiết thread được thêm vào wait queue khi nào và như thế nào.

### 3.1 Thread tham gia wait queue

#### 3.1.1 Thời điểm tham gia queue

Khi thực thi Acquire(1), sẽ lấy lock thông qua tryAcquire. Trong trường hợp này, nếu lấy lock thất bại thì sẽ gọi addWaiter để thêm vào wait queue.

#### 3.1.2 Cách tham gia queue

Sau khi lấy lock thất bại, sẽ thực thi addWaiter(Node.EXCLUSIVE) để thêm vào wait queue; method triển khai cụ thể như sau:

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

private Node addWaiter(Node mode) {
  Node node = new Node(Thread.currentThread(), mode);
  // Try the fast path of enq; backup to full enq on failure
  Node pred = tail;
  if (pred != null) {
    node.prev = pred;
    if (compareAndSetTail(pred, node)) {
      pred.next = node;
      return node;
    }
  }
  enq(node);
  return node;
}
private final boolean compareAndSetTail(Node expect, Node update) {
  return unsafe.compareAndSwapObject(this, tailOffset, expect, update);
}
```

Flow chính như sau:

- Tạo một node mới bằng thread hiện tại và lock mode.
- Pointer Pred trỏ đến tail node Tail.
- Cho Prev pointer của Node trong New trỏ đến Pred.
- Hoàn tất việc set tail node thông qua method compareAndSetTail. Method này chủ yếu so sánh tailOffset với Expect; nếu địa chỉ Node của tailOffset và Node của Expect giống nhau thì set giá trị Tail thành giá trị của Update.

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

static {
  try {
    stateOffset = unsafe.objectFieldOffset(AbstractQueuedSynchronizer.class.getDeclaredField("state"));
    headOffset = unsafe.objectFieldOffset(AbstractQueuedSynchronizer.class.getDeclaredField("head"));
    tailOffset = unsafe.objectFieldOffset(AbstractQueuedSynchronizer.class.getDeclaredField("tail"));
    waitStatusOffset = unsafe.objectFieldOffset(Node.class.getDeclaredField("waitStatus"));
    nextOffset = unsafe.objectFieldOffset(Node.class.getDeclaredField("next"));
  } catch (Exception ex) {
    throw new Error(ex);
  }
}
```

Từ static code block của AQS có thể thấy các thao tác đều lấy offset của attribute của một object trong memory so với object đó. Nhờ offset này, chúng ta có thể tìm attribute trong memory của object. tailOffset là offset tương ứng với tail, vì vậy lúc này Node được new sẽ trở thành tail node của queue hiện tại. Đồng thời, vì đây là doubly linked list nên cũng cần cho node trước trỏ đến tail node.

- Nếu pointer Pred là Null (cho biết trong wait queue không có element), hoặc vị trí mà pointer Pred hiện tại trỏ đến khác vị trí Tail trỏ đến (cho biết đã bị thread khác sửa), thì cần xem method Enq.

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

private Node enq(final Node node) {
  for (;;) {
    Node t = tail;
    if (t == null) { // Must initialize
      if (compareAndSetHead(new Node()))
        tail = head;
    } else {
      node.prev = t;
      if (compareAndSetTail(t, node)) {
        t.next = node;
        return t;
      }
    }
  }
}
```

Nếu chưa được initialize thì cần initialize một head node. Tuy nhiên cần lưu ý head node được initialize không phải node của thread hiện tại, mà là node được tạo bằng constructor không tham số. Nếu đã initialize hoặc do concurrency khiến queue có element thì thực hiện giống method trước. Thực ra addWaiter chính là thao tác thêm tail node vào doubly linked list; cần lưu ý head node của doubly linked list là head node được tạo bằng constructor không tham số.

Tóm lại, flow tổng quát khi thread lấy lock như sau:

1. Khi chưa có thread nào lấy được lock, thread 1 lấy lock thành công.

2. Thread 2 request lock, nhưng lock đang bị thread 1 chiếm giữ.

![img](https://p0.meituan.net/travelcube/e9e385c3c68f62c67c8d62ab0adb613921117.png)

3. Nếu có thêm thread muốn lấy lock thì lần lượt xếp hàng về phía sau trong queue.

Quay lại đoạn code trên, hasQueuedPredecessors là method để fair lock kiểm tra khi lock xem trong wait queue có valid node hay không. Nếu trả về False, cho biết thread hiện tại có thể tranh shared resource; nếu trả về True, cho biết trong queue có valid node, thread hiện tại phải tham gia wait queue.

```java
// java.util.concurrent.locks.ReentrantLock

public final boolean hasQueuedPredecessors() {
  // The correctness of this depends on head being initialized
  // before tail and on head.next being accurate if the current
  // thread is first in queue.
  Node t = tail; // Read fields in reverse initialization order
  Node h = head;
  Node s;
  return h != t && ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

Đến đây, hãy hiểu vì sao h != t && ((s = h.next) == null || s.thread != Thread.currentThread()); phải kiểm tra node sau head. Data được lưu trong node đầu tiên là gì?

> Trong doubly linked list, node đầu tiên là virtual node, thực ra không lưu thông tin nào mà chỉ dùng để giữ chỗ. Node đầu tiên thực sự có data bắt đầu từ node thứ hai. Khi h != t: nếu (s = h.next) == null, wait queue đang được thread initialize nhưng mới chỉ thực hiện đến bước Tail trỏ đến Head, chưa cho Head trỏ đến Tail; lúc này queue có element nên cần trả về True (chi tiết xem phần phân tích code bên dưới). Nếu (s = h.next) != null, cho biết lúc này queue có ít nhất một valid node. Nếu s.thread == Thread.currentThread(), cho biết thread trong valid node đầu tiên của wait queue giống thread hiện tại, nên thread hiện tại có thể acquire resource; nếu s.thread != Thread.currentThread(), cho biết thread trong valid node đầu tiên của wait queue khác thread hiện tại, thread hiện tại phải tham gia wait queue.

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer#enq

if (t == null) { // Must initialize
  if (compareAndSetHead(new Node()))
    tail = head;
} else {
  node.prev = t;
  if (compareAndSetTail(t, node)) {
    t.next = node;
    return t;
  }
}
```

Node enqueue không phải atomic operation nên có thể xuất hiện trạng thái head != tail trong thời gian ngắn; lúc này Tail trỏ đến node cuối cùng, còn Tail trỏ đến Head. Nếu Head chưa trỏ đến Tail (có thể thấy ở dòng 5, 6, 7), thì trong trường hợp này cũng cần thêm thread tương ứng vào queue. Vì vậy đoạn code này dùng để giải quyết vấn đề concurrency trong trường hợp cực đoan.

#### 3.1.3 Thời điểm thread trong wait queue dequeue

Quay lại source code ban đầu:

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

public final void acquire(int arg) {
  if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
    selfInterrupt();
}
```

Phần trên đã giải thích method addWaiter; method này thực ra đưa thread tương ứng vào doubly linked queue dưới dạng data structure Node và trả về một Node chứa thread đó. Node này sẽ được truyền làm parameter vào method acquireQueued. Method acquireQueued có thể thực hiện thao tác “lấy lock” với các thread đang xếp hàng.

Nhìn chung, sau khi thread lấy lock thất bại, nó sẽ được đưa vào wait queue; `acquireQueued` sẽ cho thread tiếp tục waiting và thử lấy lock cho đến khi thành công. `acquire(int)` là cách acquire không thể bị interrupt: khi bị interrupt trong thời gian waiting, nó sẽ ghi nhận trạng thái interrupt trước, sau khi lấy lock thành công mới khôi phục bằng `selfInterrupt()`, chứ không cancel lần acquire này.

Tiếp theo chúng ta phân tích source code acquireQueued theo hai hướng “khi nào dequeue?” và “dequeue như thế nào?”:

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

final boolean acquireQueued(final Node node, int arg) {
  // Đánh dấu đã lấy resource thành công hay chưa
  boolean failed = true;
  try {
    // Đánh dấu trong quá trình waiting có từng bị interrupt hay không
    boolean interrupted = false;
    // Bắt đầu spin, hoặc lấy lock hoặc bị interrupt
    for (;;) {
      // Lấy predecessor của node hiện tại
      final Node p = node.predecessor();
      // Nếu p là head node, cho biết node hiện tại ở đầu hàng đợi data thực; thử lấy lock (đừng quên head node là virtual node)
      if (p == head && tryAcquire(arg)) {
        // Lấy lock thành công, di chuyển head pointer đến node hiện tại
        setHead(node);
        p.next = null; // help GC
        failed = false;
        return interrupted;
      }
      // p là head node nhưng hiện tại chưa lấy được lock (có thể bị unfair lock tranh trước), hoặc p không phải head node; lúc này cần xác định node hiện tại có nên bị block hay không (điều kiện block: waitStatus của predecessor là -1), tránh vòng lặp vô hạn gây lãng phí resource. Hai method cụ thể sẽ được phân tích kỹ bên dưới.
      if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
        interrupted = true;
    }
  } finally {
    if (failed)
      cancelAcquire(node);
  }
}
```

Lưu ý: method setHead set node hiện tại thành virtual node nhưng không sửa waitStatus, vì waitStatus là data luôn cần được sử dụng.

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

private void setHead(Node node) {
  head = node;
  node.thread = null;
  node.prev = null;
}

// java.util.concurrent.locks.AbstractQueuedSynchronizer

// Dựa vào predecessor để phán đoán thread hiện tại có nên bị block hay không
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
  // Lấy trạng thái node của predecessor
  int ws = pred.waitStatus;
  // Cho biết predecessor đang ở trạng thái wakeup
  if (ws == Node.SIGNAL)
    return true;
  // Qua enum value có thể biết waitStatus>0 là trạng thái cancel
  if (ws > 0) {
    do {
      // Lặp để tìm về phía trước, loại cancel node khỏi queue
      node.prev = pred = pred.prev;
    } while (pred.waitStatus > 0);
    pred.next = node;
  } else {
    // Set trạng thái waiting của node trước thành SIGNAL
    compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
  }
  return false;
}
```

parkAndCheckInterrupt chủ yếu dùng để suspend thread hiện tại, block call stack và trả về interrupt status của thread hiện tại.

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```

Flow của method trên như sơ đồ sau:

![](https://p0.meituan.net/travelcube/c124b76dcbefb9bdc778458064703d1135485.png)

Từ hình trên có thể thấy điều kiện thoát khỏi loop hiện tại là “predecessor là head node và thread hiện tại lấy lock thành công”. Để tránh lãng phí CPU do infinite loop, chúng ta sẽ kiểm tra trạng thái của predecessor để quyết định có suspend thread hiện tại hay không; flow suspend cụ thể được biểu diễn bằng sơ đồ sau (flow shouldParkAfterFailedAcquire):

![](https://p0.meituan.net/travelcube/9af16e2481ad85f38ca322a225ae737535740.png)

Sau khi giải đáp nghi vấn về việc release node khỏi queue, lại xuất hiện các vấn đề mới:

- Cancel node trong shouldParkAfterFailedAcquire được tạo ra như thế nào? waitStatus của một node được set thành -1 vào lúc nào?
- Node được release và thông báo đến thread đang suspend vào thời điểm nào?

### 3.2 Tạo node ở trạng thái CANCELLED

Đoạn Finally trong method acquireQueued:

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

final boolean acquireQueued(final Node node, int arg) {
  boolean failed = true;
  try {
    ...
    for (;;) {
      final Node p = node.predecessor();
      if (p == head && tryAcquire(arg)) {
        ...
        failed = false;
        ...
      }
      ...
  } finally {
    if (failed)
      cancelAcquire(node);
    }
}
```

Thông qua method cancelAcquire, trạng thái của Node được đánh dấu thành CANCELLED. Tiếp theo chúng ta phân tích nguyên lý của method này theo từng dòng:

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

private void cancelAcquire(Node node) {
  // Lọc node không hợp lệ
  if (node == null)
    return;
  // Set node này không liên kết với thread nào, tức virtual node
  node.thread = null;
  Node pred = node.prev;
  // Bỏ qua node ở trạng thái cancel thông qua predecessor
  while (pred.waitStatus > 0)
    node.prev = pred = pred.prev;
  // Lấy successor của predecessor đã được lọc
  Node predNext = pred.next;
  // Set trạng thái node hiện tại thành CANCELLED
  node.waitStatus = Node.CANCELLED;
  // Nếu node hiện tại là tail node, set node đầu tiên không ở trạng thái cancel tính từ cuối về trước thành tail node
  // Nếu update thất bại thì đi vào else; nếu update thành công thì set successor của tail thành null
  if (node == tail && compareAndSetTail(node, pred)) {
    compareAndSetNext(pred, predNext, null);
  } else {
    int ws;
    // Nếu node hiện tại không phải successor của head: 1. kiểm tra predecessor của node hiện tại có phải SIGNAL không; 2. nếu không, set predecessor thành SIGNAL và xem có thành công không
    // Nếu một trong hai điều kiện 1 và 2 là true, tiếp tục kiểm tra thread của node hiện tại có phải null không
    // Nếu tất cả điều kiện trên đều thỏa mãn, cho successor của predecessor của node hiện tại trỏ đến successor của node hiện tại
    if (pred != head && ((ws = pred.waitStatus) == Node.SIGNAL || (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) && pred.thread != null) {
      Node next = node.next;
      if (next != null && next.waitStatus <= 0)
        compareAndSetNext(pred, predNext, next);
    } else {
      // Nếu node hiện tại là successor của head, hoặc điều kiện trên không thỏa mãn, wakeup successor của node hiện tại
      unparkSuccessor(node);
    }
    node.next = node; // help GC
  }
}
```

Flow hiện tại:

- Lấy predecessor của node hiện tại. Nếu trạng thái của predecessor là CANCELLED thì duyệt liên tục về phía trước, tìm node đầu tiên có waitStatus <= 0, liên kết Pred tìm được với Node hiện tại và set Node hiện tại thành CANCELLED.
- Xét ba trường hợp sau tùy vị trí của node hiện tại:

(1) Node hiện tại là tail node.

(2) Node hiện tại là successor của Head.

(3) Node hiện tại không phải successor của Head cũng không phải tail node.

Theo mục thứ hai ở trên, hãy phân tích flow của từng trường hợp.

Node hiện tại là tail node.

![](https://p1.meituan.net/travelcube/b845211ced57561c24f79d56194949e822049.png)

Node hiện tại là successor của Head.

![](https://p1.meituan.net/travelcube/ab89bfec875846e5028a4f8fead32b7117975.png)

Node hiện tại không phải successor của Head cũng không phải tail node.

![](https://p0.meituan.net/travelcube/45d0d9e4a6897eddadc4397cf53d6cd522452.png)

Qua flow trên, chúng ta đã hiểu sơ bộ việc tạo và thay đổi trạng thái của CANCELLED node. Nhưng vì sao mọi thay đổi đều thao tác trên Next pointer mà không thao tác trên Prev pointer? Khi nào sẽ thao tác trên Prev pointer?

> Khi thực thi cancelAcquire, predecessor của node hiện tại có thể đã rời queue (đã thực thi method shouldParkAfterFailedAcquire trong khối Try). Nếu sửa Prev pointer lúc này thì có thể khiến Prev trỏ đến một Node khác đã bị remove khỏi queue, vì vậy thay đổi Prev pointer ở đây không an toàn. Trong method shouldParkAfterFailedAcquire có thực thi đoạn code dưới đây, thực ra là xử lý Prev pointer. shouldParkAfterFailedAcquire chỉ được thực thi khi lấy lock thất bại; sau khi vào method này, cho biết shared resource đã được acquire, các node trước node hiện tại sẽ không thay đổi, vì vậy lúc này thay đổi Prev pointer tương đối an toàn.
>
> ```java
> do {
>   node.prev = pred = pred.prev;
> } while (pred.waitStatus > 0);
> ```

### 3.3 Cách unlock

Chúng ta đã phân tích flow cơ bản trong quá trình lock; tiếp theo phân tích flow cơ bản của unlock. Vì ReentrantLock không phân biệt fair lock và unfair lock khi unlock, nên xem trực tiếp source code unlock:

```java
// java.util.concurrent.locks.ReentrantLock

public void unlock() {
  sync.release(1);
}
```

Có thể thấy nơi thực sự release lock được hoàn thành thông qua framework.

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

public final boolean release(int arg) {
  if (tryRelease(arg)) {
    Node h = head;
    if (h != null && h.waitStatus != 0)
      unparkSuccessor(h);
    return true;
  }
  return false;
}
```

Trong ReentrantLock, class cha Sync của fair lock và unfair lock định nghĩa cơ chế release reentrant lock.

```java
// java.util.concurrent.locks.ReentrantLock.Sync

// Method trả về việc lock hiện tại có còn được thread nào nắm giữ hay không
protected final boolean tryRelease(int releases) {
  // Giảm số lần reentrant
  int c = getState() - releases;
  // Thread hiện tại không phải thread đang giữ lock, throw exception
  if (Thread.currentThread() != getExclusiveOwnerThread())
    throw new IllegalMonitorStateException();
  boolean free = false;
  // Nếu thread đang giữ lock đã release toàn bộ, set thread sở hữu exclusive lock hiện tại thành null và update state
  if (c == 0) {
    free = true;
    setExclusiveOwnerThread(null);
  }
  setState(c);
  return free;
}
```

Hãy giải thích source code dưới đây:

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

public final boolean release(int arg) {
  // Nếu tryRelease do custom synchronizer định nghĩa ở trên trả về true, cho biết lock này không còn bị thread nào nắm giữ
  if (tryRelease(arg)) {
    // Lấy head node
    Node h = head;
    // Head node không null và waitStatus của head node không phải trạng thái node được initialize, bỏ trạng thái suspend của thread
    if (h != null && h.waitStatus != 0)
      unparkSuccessor(h);
    return true;
  }
  return false;
}
```

Vì sao điều kiện kiểm tra là h != null && h.waitStatus != 0?

> h == null: Head chưa được initialize. Ở trạng thái ban đầu, head == null; khi node đầu tiên enqueue, Head sẽ được initialize thành một virtual node. Vì vậy nếu chưa kịp enqueue thì có thể xuất hiện head == null.
>
> h != null && waitStatus == 0 cho biết thread tương ứng với successor node vẫn đang chạy, không cần wakeup.
>
> h != null && waitStatus < 0 cho biết successor node có thể đang bị block, cần wakeup.

Tiếp theo xem method unparkSuccessor:

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

private void unparkSuccessor(Node node) {
  // Lấy waitStatus của head node
  int ws = node.waitStatus;
  if (ws < 0)
    compareAndSetWaitStatus(node, ws, 0);
  // Lấy node tiếp theo của node hiện tại
  Node s = node.next;
  // Nếu node tiếp theo là null hoặc node tiếp theo đã bị cancelled, tìm node đầu tiên không bị cancelled của queue
  if (s == null || s.waitStatus > 0) {
    s = null;
    // Bắt đầu từ tail node, tìm về đầu queue, tìm node đầu tiên có waitStatus<0.
    for (Node t = tail; t != null && t != node; t = t.prev)
      if (t.waitStatus <= 0)
        s = t;
  }
  // Nếu node tiếp theo của node hiện tại không null và trạng thái <=0, unpark node hiện tại
  if (s != null)
    LockSupport.unpark(s.thread);
}
```

Vì sao phải tìm node đầu tiên không bị Cancelled từ phía sau về phía trước? Nguyên nhân như sau.

Method addWaiter trước đó:

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

private Node addWaiter(Node mode) {
  Node node = new Node(Thread.currentThread(), mode);
  // Try the fast path of enq; backup to full enq on failure
  Node pred = tail;
  if (pred != null) {
    node.prev = pred;
    if (compareAndSetTail(pred, node)) {
      pred.next = node;
      return node;
    }
  }
  enq(node);
  return node;
}
```

Từ đây có thể thấy node enqueue không phải atomic operation, tức node.prev = pred và compareAndSetTail(pred, node) có thể xem là atomic operation để Tail enqueue, nhưng lúc này pred.next = node chưa được thực thi. Nếu method unparkSuccessor được thực thi đúng lúc đó thì không thể tìm từ trước ra sau, nên cần tìm từ sau về trước. Một nguyên nhân khác là khi tạo node ở trạng thái CANCELLED, Next pointer bị ngắt trước còn Prev pointer chưa bị ngắt, vì vậy cũng bắt buộc phải duyệt từ sau về trước mới có thể duyệt hết tất cả Node.

Tóm lại, nếu tìm từ trước ra sau thì trong trường hợp cực đoan, do atomic operation khi enqueue và thao tác ngắt Next pointer trong quá trình tạo CANCELLED node, có thể không duyệt được toàn bộ node. Vì vậy sau khi wakeup thread tương ứng, thread đó sẽ tiếp tục thực thi. Sau khi tiếp tục thực thi method acquireQueued, interrupt được xử lý như thế nào?

### 3.4 Flow thực thi sau khi khôi phục interrupt

Sau khi wakeup, sẽ thực thi return Thread.interrupted(); function này trả về interrupt status của thread hiện tại và clear status đó.

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

private final boolean parkAndCheckInterrupt() {
  LockSupport.park(this);
  return Thread.interrupted();
}
```

Quay lại code acquireQueued: khi parkAndCheckInterrupt trả về True hoặc False, giá trị của interrupted khác nhau, nhưng đều thực thi vòng lặp tiếp theo. Nếu lúc này lấy lock thành công thì sẽ return interrupted hiện tại.

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

final boolean acquireQueued(final Node node, int arg) {
  boolean failed = true;
  try {
    boolean interrupted = false;
    for (;;) {
      final Node p = node.predecessor();
      if (p == head && tryAcquire(arg)) {
        setHead(node);
        p.next = null; // help GC
        failed = false;
        return interrupted;
      }
      if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
        interrupted = true;
      }
  } finally {
    if (failed)
      cancelAcquire(node);
  }
}
```

Nếu acquireQueued là True thì sẽ thực thi method selfInterrupt.

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

static void selfInterrupt() {
  Thread.currentThread().interrupt();
}
```

Method này thực ra dùng để interrupt thread. Nhưng vì sao sau khi lấy lock lại còn phải interrupt thread? Phần này thuộc kiến thức về cooperative interrupt do Java cung cấp; nếu quan tâm, bạn có thể tìm hiểu thêm. Ở đây chỉ giới thiệu ngắn gọn:

1. Khi thread bị interrupt được wakeup, không biết nguyên nhân wakeup là gì: có thể thread hiện tại bị interrupt trong lúc waiting, cũng có thể được wakeup sau khi lock được release. Vì vậy dùng method Thread.interrupted() để kiểm tra interrupt flag (method này trả về interrupt status của thread hiện tại và set interrupt flag của thread hiện tại thành False), ghi nhận lại; nếu phát hiện thread đã từng bị interrupt thì interrupt thêm một lần nữa.
2. Thread được wakeup trong quá trình waiting resource vẫn sẽ liên tục thử lấy lock sau khi wakeup, cho đến khi tranh được lock. Nói cách khác, trong toàn bộ flow, thread không phản hồi interrupt mà chỉ ghi nhận interrupt. Cuối cùng khi lấy lock thành công và return, nếu thread từng bị interrupt thì cần bổ sung thêm một lần interrupt.

Cách xử lý này chủ yếu sử dụng `runWorker` trong Worder, đơn vị vận hành cơ bản trong thread pool, để kiểm tra và xử lý bổ sung thông qua `Thread.interrupted()`. Nếu quan tâm, bạn có thể xem source code của ThreadPoolExecutor.

### 3.5 Tóm tắt

Ở mục 1.3, chúng ta đã nêu một số vấn đề; bây giờ hãy trả lời chúng.

> Q: Flow tiếp theo sau khi một thread lấy lock thất bại là gì?
>
> A: Có một cơ chế xếp hàng chờ, thread tiếp tục waiting và vẫn giữ khả năng lấy lock, quy trình lấy lock vẫn tiếp tục.
>
> Q: Vì đã nói đến cơ chế xếp hàng chờ thì nhất định sẽ hình thành queue; queue này dùng data structure gì?
>
> A: Là doubly linked queue FIFO dạng biến thể CLH.
>
> Q: Thread đang trong cơ chế xếp hàng chờ có thể có cơ hội lấy lock vào lúc nào?
>
> A: Có thể xem chi tiết ở mục 2.3.1.3.
>
> Q: Nếu thread đang trong cơ chế xếp hàng chờ mãi không thể lấy lock thì có phải tiếp tục waiting không? Hay có strategy khác để giải quyết vấn đề này?
>
> A: Trạng thái node chứa thread sẽ chuyển thành trạng thái cancel; node ở trạng thái cancel sẽ được release khỏi queue, cụ thể xem mục 2.3.2.
>
> Q: Function Lock thực hiện lock thông qua method Acquire, nhưng cụ thể lock như thế nào?
>
> A: Acquire của AQS sẽ gọi method tryAcquire; tryAcquire do từng custom synchronizer triển khai, và hoàn tất quy trình lock thông qua tryAcquire.

## 4 Ứng dụng AQS

### 4.1 Ứng dụng reentrant của ReentrantLock

Tính reentrant của ReentrantLock là một trong những ứng dụng tốt của AQS. Sau khi hiểu các kiến thức trên, chúng ta có thể dễ dàng biết cách ReentrantLock triển khai reentrant. Trong ReentrantLock, dù là fair lock hay unfair lock đều có một đoạn logic.

Fair lock:

```java
// java.util.concurrent.locks.ReentrantLock.FairSync#tryAcquire

if (c == 0) {
  if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) {
    setExclusiveOwnerThread(current);
    return true;
  }
}
else if (current == getExclusiveOwnerThread()) {
  int nextc = c + acquires;
  if (nextc < 0)
    throw new Error("Maximum lock count exceeded");
  setState(nextc);
  return true;
}
```

Unfair lock:

```java
// java.util.concurrent.locks.ReentrantLock.Sync#nonfairTryAcquire

if (c == 0) {
  if (compareAndSetState(0, acquires)){
    setExclusiveOwnerThread(current);
    return true;
  }
}
else if (current == getExclusiveOwnerThread()) {
  int nextc = c + acquires;
  if (nextc < 0) // overflow
    throw new Error("Maximum lock count exceeded");
  setState(nextc);
  return true;
}
```

Từ hai đoạn trên đều có thể thấy có một trạng thái đồng bộ State dùng để kiểm soát toàn bộ tình trạng reentrant. State được đánh dấu bằng Volatile, dùng để bảo đảm visibility và order nhất định.

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

private volatile int state;
```

Tiếp theo xem quy trình chính của field State:

1. Khi State initialize, giá trị là 0, biểu thị chưa có thread nào giữ lock.
2. Khi có thread giữ lock, giá trị sẽ tăng 1 trên cơ sở giá trị cũ; cùng một thread lấy lock nhiều lần thì sẽ tăng nhiều lần, đây chính là khái niệm reentrant.
3. Unlock cũng giảm field này đi 1 cho đến khi về 0; thread đó release lock.

### 4.2 Trường hợp sử dụng trong JUC

Ngoài ứng dụng tính reentrant của ReentrantLock ở trên, AQS với vai trò framework cho concurrent programming còn cung cấp solution tốt cho nhiều công cụ đồng bộ khác. Dưới đây là một số công cụ đồng bộ trong JUC và phần giới thiệu khái quát về các trường hợp sử dụng AQS:

| Công cụ đồng bộ        | Mối liên hệ giữa công cụ đồng bộ và AQS                                                                                                                                                                    |
| :--------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ReentrantLock          | Dùng AQS để lưu số lần lock được giữ lặp lại. Khi một thread lấy lock, ReentrantLock ghi nhận định danh của thread đang lấy lock để kiểm tra việc lấy lặp lại và xử lý exception khi thread sai cố unlock. |
| Semaphore              | Dùng trạng thái đồng bộ của AQS để lưu counter hiện tại của semaphore. tryRelease tăng counter, acquireShared giảm counter.                                                                                |
| CountDownLatch         | Dùng trạng thái đồng bộ của AQS để biểu thị counter. Chỉ khi counter bằng 0 thì mọi thao tác Acquire (method await của CountDownLatch) mới có thể đi qua.                                                  |
| ReentrantReadWriteLock | Dùng 16 bit trong trạng thái đồng bộ của AQS để lưu số lần write lock được giữ, 16 bit còn lại để lưu số lần read lock được giữ.                                                                           |
| ThreadPoolExecutor     | Worker dùng trạng thái đồng bộ của AQS để triển khai việc set exclusive thread variable (tryAcquire và tryRelease).                                                                                        |

### 4.3 Custom synchronizer

Sau khi hiểu nguyên lý cơ bản của AQS, hãy dựa trên các kiến thức AQS ở trên để tự triển khai một công cụ đồng bộ.

```java
public class LeeLock  {

    private static class Sync extends AbstractQueuedSynchronizer {
        @Override
        protected boolean tryAcquire (int arg) {
            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        @Override
        protected boolean tryRelease (int arg) {
            if (getState() == 0 || getExclusiveOwnerThread() != Thread.currentThread()) {
                throw new IllegalMonitorStateException();
            }
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        @Override
        protected boolean isHeldExclusively () {
            return getState() == 1 && getExclusiveOwnerThread() == Thread.currentThread();
        }
    }

    private final Sync sync = new Sync();

    public void lock () {
        sync.acquire(1);
    }

    public void unlock () {
        sync.release(1);
    }
}
```

Hoàn thành một số chức năng đồng bộ thông qua Lock do chính chúng ta định nghĩa.

```java
public class LeeMain {

    static int count = 0;
    static LeeLock leeLock = new LeeLock();

    public static void main (String[] args) throws InterruptedException {

        Runnable runnable = new Runnable() {
            @Override
            public void run () {
                try {
                    leeLock.lock();
                    for (int i = 0; i < 10000; i++) {
                        count++;
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    leeLock.unlock();
                }

            }
        };
        Thread thread1 = new Thread(runnable);
        Thread thread2 = new Thread(runnable);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println(count);
    }
}
```

Kết quả mỗi lần chạy đoạn code trên đều là 20000. Chỉ với vài dòng code đơn giản đã có thể triển khai chức năng đồng bộ; đây chính là sức mạnh của AQS.

## 5 Tổng kết

Trong quá trình phát triển hằng ngày, có rất nhiều trường hợp sử dụng concurrency, nhưng không nhiều người hiểu nguyên lý của framework cơ bản bên trong concurrency. Do giới hạn về độ dài, bài viết chỉ giới thiệu nguyên lý của reentrant lock ReentrantLock và nguyên lý AQS, hy vọng có thể trở thành “viên gạch đầu tiên” giúp bạn tìm hiểu AQS, ReentrantLock và các synchronizer khác.

## Tài liệu tham khảo

- Lea D. The java. util. concurrent synchronizer framework\[J]. Science of Computer Programming, 2005, 58(3): 293-309.
- “Thực chiến lập trình concurrent Java”
- [Chuyện về “lock” trong Java không thể không nói](https://tech.meituan.com/2018/11/15/java-lock.html)

<!-- @include: @article-footer.snippet.md -->
