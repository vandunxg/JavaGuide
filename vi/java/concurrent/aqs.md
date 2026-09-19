---
title: Giải thích chi tiết AQS
description: "Phân tích chuyên sâu về AQS (AbstractQueuedSynchronizer): nguyên lý cốt lõi, cấu trúc CLH queue, triển khai exclusive lock và shared lock, ứng dụng của các synchronizer như ReentrantLock/Semaphore và cơ chế block/wake-up thread."
category: Java
tag:
  - Java Concurrency
head:
  - - meta
    - name: keywords
      content: AQS,AbstractQueuedSynchronizer,queue synchronizer,exclusive lock,shared lock,CLH queue,nguyên lý triển khai ReentrantLock
---

<!-- markdownlint-disable MD024 -->

## Giới thiệu AQS

Tên đầy đủ của AQS là `AbstractQueuedSynchronizer`, nghĩa là abstract queue synchronizer. Class này nằm trong package `java.util.concurrent.locks`.

![](https://oss.javaguide.cn/github/javaguide/AQS.png)

AQS là một abstract class, chủ yếu dùng để xây dựng lock và synchronizer.

```java
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer implements java.io.Serializable {
}
```

AQS cung cấp một số triển khai chức năng dùng chung để xây dựng lock và synchronizer. Vì vậy, dùng AQS có thể xây dựng đơn giản và hiệu quả nhiều synchronizer được ứng dụng rộng rãi, chẳng hạn `ReentrantLock`, `Semaphore`, `ReentrantReadWriteLock`. Mặc dù `SynchronousQueue` cũng dùng waiting queue và CAS để ghép cặp thread, nó không dựa trên AQS.

## Nguyên lý AQS

> Lưu ý: Phần source code AQS và phân tích trạng thái node dưới đây chủ yếu dựa trên JDK 8. Triển khai bên trong AQS tiếp tục phát triển: trong JDK 11 vẫn có thể thấy các field hoặc method như `waitStatus`, `addWaiter()`, `acquireQueued()` được đề cập trong bài; còn field node và cách enqueue, waiting trong JDK 17 cũng như phiên bản hiện tại đã thay đổi khá nhiều. Tư duy tổng thể xây dựng synchronizer dựa trên synchronization state, waiting queue và template method không thay đổi.

Khi được hỏi về kiến thức concurrency trong phỏng vấn, phần lớn bạn sẽ được hỏi: “Hãy nói về cách bạn hiểu nguyên lý AQS”. Dưới đây là một ví dụ để tham khảo. Phỏng vấn không phải là học thuộc câu trả lời, bạn nhất định phải thêm suy nghĩ của mình; nếu chưa thể thêm suy nghĩ riêng thì ít nhất cũng phải bảo đảm có thể diễn đạt dễ hiểu thay vì đọc thuộc lòng.

### Tìm hiểu nhanh về AQS

Trước khi thực sự giải thích source code AQS, cần có nhận thức tổng thể về AQS. Phần này sẽ bắt đầu bằng một vài câu hỏi để hiểu AQS ở cấp độ tổng thể, biết AQS nằm ở tầng nào trong Java Concurrency; sau đó khi học source code AQS, bạn sẽ hiểu rõ hơn mối quan hệ giữa synchronizer và AQS.

#### AQS có tác dụng gì?

AQS giải quyết vấn đề phức tạp khi developer triển khai synchronizer. Nó cung cấp một framework dùng chung để triển khai nhiều synchronizer, ví dụ **reentrant lock** (`ReentrantLock`), **semaphore** (`Semaphore`) và **countdown latch** (`CountDownLatch`). Bằng cách đóng gói cơ chế synchronization của thread ở tầng dưới, AQS ẩn logic quản lý thread phức tạp, để developer chỉ cần tập trung vào logic synchronization cụ thể.

Nói đơn giản, AQS là một abstract class, cung cấp **execution framework** dùng chung cho synchronizer. Nó định nghĩa **quy trình dùng chung để acquire và release resource**, còn logic acquire resource cụ thể do synchronizer cụ thể triển khai bằng cách override template method. Vì vậy, có thể xem AQS là **“nền móng” cơ bản** của synchronizer, còn synchronizer là **“ứng dụng” cụ thể** được xây dựng trên AQS.

#### Vì sao AQS sử dụng biến thể của CLH lock queue?

CLH lock là một triển khai tối ưu dựa trên **spin lock**.

Trước hết hãy nói về vấn đề của spin lock: spin lock liên tục thực hiện thao tác `compareAndSet` (gọi tắt là `CAS`) trên một atomic variable để thử acquire lock. Trong tình huống concurrency cao, nhiều thread đồng thời tranh chấp cùng một atomic variable, dễ khiến thao tác `CAS` của một thread thất bại trong thời gian dài, từ đó gây ra vấn đề **“starvation”** (một số thread có thể không bao giờ acquire được lock).

CLH lock cải tiến spin lock bằng cách thêm một queue để tổ chức các thread đang cạnh tranh:

- Mỗi thread trở thành một node rồi thêm vào queue, đồng thời spin để theo dõi trạng thái của node thread phía trước thay vì trực tiếp tranh chấp shared variable.
- Các thread xếp hàng theo thứ tự, bảo đảm fairness và tránh vấn đề “starvation”.

AQS (AbstractQueuedSynchronizer) tiếp tục tối ưu trên nền CLH lock, tạo thành **biến thể CLH queue** bên trong. Hai điểm cải tiến chính:

1. **Spin + blocking**: CLH lock dùng spin thuần túy để chờ lock release, nhưng nhiều thao tác spin sẽ chiếm quá nhiều CPU. AQS đưa vào cơ chế kết hợp **spin + blocking**:
   - Nếu thread acquire lock thất bại, trước tiên spin trong thời gian ngắn để thử acquire lock;
   - Nếu vẫn thất bại, thread chuyển sang trạng thái blocking và chờ được wake-up, nhờ đó giảm lãng phí CPU.
2. **Đổi queue một chiều thành queue hai chiều**: CLH lock dùng queue một chiều, node chỉ biết trạng thái của predecessor; khi một node release lock, cần wake-up node phía sau thông qua queue. AQS đổi queue thành **queue hai chiều**, thêm pointer `next`, để node không chỉ biết predecessor mà còn có thể trực tiếp wake-up successor, nhờ đó đơn giản hóa thao tác queue và tăng hiệu quả wake-up.

#### Vì sao AQS có performance khá tốt?

Vì bên trong AQS sử dụng rất nhiều thao tác `CAS`.

Bên trong AQS dùng queue để lưu các node thread đang waiting. Vì queue là shared resource, trong môi trường multi-thread cần bảo đảm truy cập queue được đồng bộ hóa.

Bên trong AQS dùng thao tác `CAS` để đồng bộ hóa quyền truy cập queue. Thao tác `CAS` chủ yếu dùng để bảo đảm concurrency safety cho hai thao tác khởi tạo queue và enqueue thread node. Dù dùng `CAS` để kiểm soát concurrency safety có thể bảo đảm performance khá tốt, nó đồng thời cũng mang đến **độ phức tạp khi coding** khá cao.

#### Vì sao Node trong AQS cần các trạng thái khác nhau?

Trạng thái `waitStatus` trong AQS tương tự một **state machine**, dùng các trạng thái khác nhau để biểu thị ý nghĩa khác nhau của Node và kiểm soát việc chuyển trạng thái theo từng thao tác.

- Trạng thái `0`: sau khi node mới được thêm vào queue, trạng thái ban đầu là `0`.

- Trạng thái `SIGNAL`: khi có node mới được thêm vào queue, trạng thái của predecessor của node mới sẽ được cập nhật từ `0` thành `SIGNAL`, biểu thị sau khi predecessor release lock thì cần wake-up node mới. Khi wake-up successor của node ở trạng thái `SIGNAL`, trạng thái `SIGNAL` sẽ được cập nhật thành `0`. Tức là việc xóa trạng thái `SIGNAL` biểu thị thao tác wake-up đã được thực hiện.

- Trạng thái `CANCELLED`: nếu một node đang chờ acquire lock trong queue nhưng thất bại vì một nguyên nhân nào đó, trạng thái của node sẽ chuyển thành `CANCELLED`, biểu thị đã hủy việc acquire lock. Node ở trạng thái này là bất thường, không thể được wake-up và cũng không thể wake-up successor.

### Tư tưởng cốt lõi của AQS

Tư tưởng cốt lõi của AQS là: nếu shared resource được request đang rảnh, thread hiện tại yêu cầu resource sẽ được chỉ định là thread đang làm việc hợp lệ và shared resource được đặt sang trạng thái locked. Nếu shared resource đang bị chiếm dụng, cần có cơ chế để thread blocking và waiting, cũng như phân phối lock khi được wake-up. AQS triển khai cơ chế này trên cơ sở **CLH lock** (Craig, Landin, and Hagersten locks) sau khi tiếp tục tối ưu.

**CLH lock** cải tiến spin lock, là spin lock dựa trên singly linked list. Trong môi trường multi-thread, các thread request acquire lock được tổ chức thành queue một chiều; mỗi thread waiting spin để truy cập trạng thái của node phía trước, chỉ khi node phía trước release lock thì node hiện tại mới có thể acquire lock. Cấu trúc queue của **CLH lock** như hình dưới.

![Cấu trúc queue của CLH lock](https://oss.javaguide.cn/github/javaguide/open-source-project/clh-lock-queue-structure.png)

**Waiting queue** được dùng trong AQS là biến thể của CLH lock queue (sau đây gọi tắt là CLH variant queue).

CLH variant queue của AQS là queue hai chiều. Các thread tạm thời chưa acquire được lock sẽ được thêm vào queue này. CLH variant queue và CLH lock queue ban đầu chủ yếu khác nhau ở hai điểm:

- Tối ưu từ **spin** thành **spin + blocking**: thao tác spin có performance rất cao, nhưng nhiều thao tác spin sẽ chiếm khá nhiều CPU. Vì vậy trong CLH variant queue, trước tiên thread spin để thử acquire lock, nếu thất bại thì chuyển sang blocking waiting.
- Tối ưu từ **queue một chiều** thành **queue hai chiều**: trong CLH variant queue, các thread waiting sẽ bị block. Sau khi thread phía trước queue release lock, cần wake-up thread phía sau, do đó thêm pointer `next` và trở thành queue hai chiều.

AQS đóng gói mỗi thread request shared resource thành một node (Node) trong CLH variant queue để thực hiện phân phối lock. Trong CLH variant queue, một node biểu thị một thread, lưu reference của thread (`thread`), trạng thái của node hiện tại trong queue (`waitStatus`), predecessor (`prev`) và successor (`next`).

Cấu trúc CLH variant queue trong AQS như hình dưới:

![Cấu trúc CLH variant queue](https://oss.javaguide.cn/github/javaguide/java/concurrent/clh-queue-structure-bianti.png)

Để đọc chi tiết về cấu trúc dữ liệu cốt lõi của AQS - CLH lock, bạn nên xem bài [Cấu trúc dữ liệu cốt lõi của Java AQS - CLH lock - Qunar Technical Salon](https://mp.weixin.qq.com/s/jEx-4XhNGOFdCo4Nou5tqg).

Sơ đồ nguyên lý cốt lõi của AQS (`AbstractQueuedSynchronizer`):

![CLH variant queue](https://oss.javaguide.cn/github/javaguide/java/concurrent/clh-queue-state.png)

AQS sử dụng **member variable `state` kiểu int để biểu thị synchronization state**, đồng thời dùng **FIFO waiting queue của thread** tích hợp sẵn để xếp hàng các thread acquire resource.

Variable `state` được đánh dấu bằng `volatile`, dùng để biểu thị tình trạng acquire của resource trong critical section hiện tại. Tác dụng của `volatile` ở đây không chỉ là bảo đảm visibility, quan trọng hơn là thông qua quy tắc happens-before (write operation của volatile variable xảy ra trước read operation tiếp theo) để ngăn compiler và processor reorder instruction, từ đó bảo đảm tính đúng đắn của lock semantics.

```java
// Shared variable, dùng volatile để bảo đảm thread visibility và ngăn instruction reordering
private volatile int state;
```

Ngoài ra, `state` có thể được thao tác thông qua `getState()`, `setState()` và `compareAndSetState()` có modifier `protected`. Các method này đều có modifier `final`, nên không thể bị override trong subclass.

```java
// Trả về giá trị hiện tại của synchronization state
protected final int getState() {
     return state;
}
// Đặt giá trị synchronization state
protected final void setState(int newState) {
     state = newState;
}
// Đặt synchronization state một cách nguyên tử (thao tác CAS) thành update nếu giá trị hiện tại bằng expect (giá trị kỳ vọng)
protected final boolean compareAndSetState(int expect, int update) {
      return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

Lấy reentrant mutex lock `ReentrantLock` làm ví dụ, bên trong nó duy trì một variable `state` để biểu thị trạng thái lock đang được chiếm dụng. Giá trị ban đầu của `state` là 0, biểu thị lock đang unlocked. Khi thread A gọi method `lock()`, nó sẽ thử acquire lock theo exclusive mode thông qua method `tryAcquire()` và tăng giá trị `state` lên 1. Nếu thành công, thread A acquire được lock. Nếu thất bại, thread A sẽ được thêm vào waiting queue (CLH variant queue) cho đến khi thread khác release lock. Giả sử thread A acquire lock thành công, trước khi release lock, chính thread A có thể tiếp tục acquire lock này (`state` sẽ tăng dần). Đây là biểu hiện của reentrancy: một thread có thể acquire cùng một lock nhiều lần mà không bị block. Tuy nhiên, điều đó cũng có nghĩa là thread phải release lock với số lần bằng số lần acquire thì `state` mới trở về 0, tức lock mới trở lại trạng thái unlocked. Chỉ khi đó các thread khác đang waiting mới có cơ hội acquire lock.

Quá trình thread A thử acquire lock như hình dưới (nguồn hình [Xem nguyên lý và ứng dụng của AQS từ triển khai ReentrantLock - Meituan Technical Team](./reentrantlock.md)):

![Acquire lock ở exclusive mode của AQS](https://oss.javaguide.cn/github/javaguide/java/concurrent/aqs-exclusive-mode-acquire-lock.png)

Tiếp theo lấy countdown timer `CountDownLatch` làm ví dụ. Có thể khởi tạo `state` bằng N, biểu thị cần N lần gọi `countDown()`. N biểu thị số event hoặc số lần count, không bắt buộc bằng số thread; một thread có thể gọi `countDown()` nhiều lần, hoặc nhiều thread lần lượt gọi. Khi `state` trở thành 0, AQS sẽ wake-up các thread bị block trong waiting queue do gọi `await()`, sau đó các thread này có thể tiếp tục thực thi.

### Ý nghĩa trạng thái waitStatus của Node

Trạng thái `waitStatus` trong AQS tương tự **state machine**, dùng các trạng thái khác nhau để biểu thị ý nghĩa khác nhau của Node và kiểm soát việc chuyển trạng thái theo từng thao tác.

| Trạng thái Node | Giá trị | Ý nghĩa                                                                                                                                                                                      |
| --------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CANCELLED`     | 1       | Biểu thị thread đã hủy việc acquire lock. Khi thread bị interrupt hoặc chờ resource timeout trong lúc chờ acquire resource, trạng thái sẽ được cập nhật thành trạng thái này.                |
| `SIGNAL`        | -1      | Biểu thị successor cần node hiện tại wake-up. Sau khi node của thread hiện tại release lock, cần wake-up successor.                                                                          |
| `CONDITION`     | -2      | Biểu thị node đang waiting trên Condition. Sau khi thread khác gọi method `signal()` của Condition, node sẽ được chuyển từ waiting queue sang synchronization queue để chờ acquire resource. |
| `PROPAGATE`     | -3      | Dùng trong shared mode. Trong shared mode có thể xảy ra tình huống thread trong queue không được wake-up, vì vậy thêm trạng thái `PROPAGATE` để giải quyết vấn đề này.                       |
|                 | 0       | Trạng thái ban đầu của node mới được thêm vào queue.                                                                                                                                         |

Trong source code AQS, thường dùng `> 0`, `< 0` để kiểm tra `waitStatus`.

Nếu `waitStatus > 0`, biểu thị node đã hủy việc waiting để acquire resource.

Nếu `waitStatus < 0`, biểu thị trạng thái node là bình thường, tức chưa hủy việc waiting.

Trong đó trạng thái `SIGNAL` là quan trọng nhất. Việc chuyển trạng thái node và thao tác tương ứng như sau:

| Chuyển trạng thái | Thao tác tương ứng                                                                                                                                                                                         |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `0`               | Khi node mới enqueue, trạng thái ban đầu là `0`.                                                                                                                                                           |
| `0 -> SIGNAL`     | Khi node mới enqueue, trạng thái predecessor của nó được cập nhật từ `0` thành `SIGNAL`. Trạng thái `SIGNAL` biểu thị successor của node này cần được wake-up.                                             |
| `SIGNAL -> 0`     | Khi wake-up successor, cần xóa trạng thái của node hiện tại. Thường xảy ra ở node `head`, ví dụ trạng thái node `head` được cập nhật từ `SIGNAL` thành `0`, biểu thị đã wake-up successor của node `head`. |
| `0 -> PROPAGATE`  | AQS đưa vào trạng thái `PROPAGATE` để giải quyết tình huống node thread có thể không được wake-up trong môi trường concurrency (sẽ phân tích trong phần source code acquire resource ở shared mode).       |

### Custom synchronizer

Có thể triển khai custom synchronizer dựa trên AQS. AQS cung cấp 5 template method (template method pattern). Cách thông thường để tạo custom synchronizer như sau (template method pattern là một ứng dụng rất kinh điển):

1. Custom synchronizer kế thừa `AbstractQueuedSynchronizer`.
2. Override các template method mà AQS expose.

**AQS sử dụng template method pattern. Khi tạo custom synchronizer, cần override các hook method dưới đây do AQS cung cấp:**

```java
// Exclusive mode. Thử acquire resource, thành công trả về true, thất bại trả về false.
protected boolean tryAcquire(int)
// Exclusive mode. Thử release resource, thành công trả về true, thất bại trả về false.
protected boolean tryRelease(int)
// Shared mode. Thử acquire resource. Số âm biểu thị thất bại; 0 biểu thị thành công nhưng không còn resource khả dụng; số dương biểu thị thành công và còn resource.
protected int tryAcquireShared(int)
// Shared mode. Thử release resource, thành công trả về true, thất bại trả về false.
protected boolean tryReleaseShared(int)
// Thread hiện tại có đang nắm giữ resource ở exclusive mode không. Chỉ cần triển khai khi dùng condition.
protected boolean isHeldExclusively()
```

**Hook method là gì?** Hook method là method được khai báo trong abstract class, thường có modifier `protected`. Nó có thể là empty method (do subclass triển khai), hoặc method có default implementation. Template design pattern kiểm soát việc triển khai các bước cố định thông qua hook method.

Do giới hạn độ dài, phần này không giải thích chi tiết template method pattern. Nếu chưa hiểu rõ, bạn có thể xem bài viết [Template method pattern được cải tiến bằng Java8 thực sự rất tuyệt!](https://mp.weixin.qq.com/s/zpScSCktFpnSWHWIQem2jg).

Ngoài các hook method nêu trên, những method khác trong class AQS đều là `final`, nên không thể bị class khác override.

### Các mode chia sẻ resource của AQS

AQS định nghĩa hai mode chia sẻ resource: `Exclusive` (exclusive, chỉ một thread có thể thực thi, như `ReentrantLock`) và `Share` (shared, nhiều thread có thể thực thi đồng thời, như `Semaphore`/`CountDownLatch`).

Nói chung, mode của custom synchronizer hoặc là exclusive hoặc là shared, và chỉ cần triển khai một trong hai nhóm `tryAcquire-tryRelease`, `tryAcquireShared-tryReleaseShared`. Tuy nhiên AQS cũng hỗ trợ custom synchronizer triển khai đồng thời cả exclusive và shared mode, như `ReentrantReadWriteLock`.

### So sánh chuyên sâu giữa exclusive mode và shared mode

Phần trên đã giới thiệu sơ lược hai mode chia sẻ resource của AQS. Phần này sẽ so sánh có hệ thống sự khác nhau giữa exclusive mode và shared mode trên nhiều phương diện để hiểu sâu hơn.

#### So sánh đặc tính

| Tiêu chí                         | Exclusive mode (Exclusive)                               | Shared mode (Share)                                                                                                                                                 |
| -------------------------------- | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Concurrency**                  | Tại một thời điểm chỉ một thread acquire được resource   | Tại một thời điểm nhiều thread có thể đồng thời acquire resource                                                                                                    |
| **Entry acquire resource**       | `acquire(int arg)`                                       | `acquireShared(int arg)`                                                                                                                                            |
| **Template method cần override** | `tryAcquire(int)` / `tryRelease(int)`                    | `tryAcquireShared(int)` / `tryReleaseShared(int)`                                                                                                                   |
| **Giá trị trả về tryXxx**        | `boolean`, `true` biểu thị acquire/release thành công    | `int` (khi acquire), số âm biểu thị thất bại, 0 biểu thị thành công nhưng không còn resource, số dương biểu thị thành công và còn resource; `boolean` (khi release) |
| **Wake-up successor**            | Wake-up một successor khi release resource               | Sau khi acquire resource thành công, nếu còn resource sẽ tiếp tục wake-up node phía sau (wake-up propagation)                                                       |
| **Node type marker**             | `Node.EXCLUSIVE` (`null`)                                | `Node.SHARED` (một static `Node` instance)                                                                                                                          |
| **Triển khai điển hình**         | `ReentrantLock`, write lock của `ReentrantReadWriteLock` | `Semaphore`, `CountDownLatch`, read lock của `ReentrantReadWriteLock`                                                                                               |

#### Ý nghĩa của `state` trong các synchronizer khác nhau

`state` trong AQS là một synchronization state variable dùng chung. Mỗi synchronizer gán cho nó một ý nghĩa khác nhau:

| Synchronizer             | Mode               | Ý nghĩa của `state`                                                                                                                        |
| ------------------------ | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `ReentrantLock`          | Exclusive          | Biểu thị số lần reentrant của lock. `state == 0` biểu thị lock rảnh; `state > 0` biểu thị lock đang được hold, giá trị là số lần reentrant |
| `ReentrantReadWriteLock` | Exclusive + shared | 16 bit cao biểu thị số read lock đang được hold (shared), 16 bit thấp biểu thị số lần reentrant của write lock (exclusive)                 |
| `Semaphore`              | Shared             | Biểu thị số permit khả dụng. Mỗi `acquire()` giảm, `release()` tăng                                                                        |
| `CountDownLatch`         | Shared             | Biểu thị count cần waiting. Mỗi `countDown()` giảm 1, khi về 0 thì wake-up mọi thread waiting                                              |

Ví dụ code dưới đây minh họa trực quan sự khác nhau khi sử dụng exclusive mode và shared mode:

```java
import java.util.concurrent.Semaphore;
import java.util.concurrent.locks.ReentrantLock;

public class ExclusiveVsSharedDemo {
    public static void main(String[] args) {
        // Exclusive mode: tại một thời điểm chỉ 1 thread có thể vào critical section
        ReentrantLock lock = new ReentrantLock();

        // Shared mode: tại một thời điểm tối đa 3 thread có thể vào critical section
        Semaphore semaphore = new Semaphore(3);

        // Ví dụ exclusive mode
        Runnable exclusiveTask = () -> {
            lock.lock();
            try {
                System.out.println(Thread.currentThread().getName()
                        + " đã acquire exclusive lock, đang thực thi...");
                Thread.sleep(500);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                lock.unlock();
            }
        };

        // Ví dụ shared mode
        Runnable sharedTask = () -> {
            boolean acquired = false;
            try {
                semaphore.acquire();
                acquired = true;
                System.out.println(Thread.currentThread().getName()
                        + " đã acquire permit, đang thực thi...");
                Thread.sleep(500);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                if (acquired) {
                    semaphore.release();
                }
            }
        };

        System.out.println("=== Exclusive mode (ReentrantLock) ===");
        for (int i = 0; i < 5; i++) {
            new Thread(exclusiveTask, "exclusive-thread-" + i).start();
        }

        try { Thread.sleep(3000); } catch (InterruptedException e) { }

        System.out.println("\n=== Shared mode (Semaphore) ===");
        for (int i = 0; i < 5; i++) {
            new Thread(sharedTask, "shared-thread-" + i).start();
        }
    }
}
```

Chạy code trên có thể quan sát thấy: ở exclusive mode, tại một thời điểm chỉ một thread thực thi, nhưng `ReentrantLock` non-fair mặc định không bảo đảm thread acquire lock đúng theo thứ tự start hoặc waiting; ở shared mode, tối đa 3 thread thực thi đồng thời.

### Phân tích source code acquire resource của AQS (exclusive mode)

Entry method để acquire resource ở exclusive mode trong AQS là `acquire()`, như sau:

```JAVA
// AQS
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

Trong `acquire()`, thread trước tiên thử acquire resource; nếu thất bại, đóng gói thread thành Node rồi thêm vào waiting queue của AQS; sau khi enqueue, thread trong waiting queue sẽ thử acquire resource và bị block. Ba thao tác này tương ứng với các method sau:

- `tryAcquire()`: thử acquire lock (template method), `AQS` không cung cấp triển khai cụ thể mà do subclass triển khai.
- `addWaiter()`: nếu acquire lock thất bại, đóng gói thread hiện tại thành Node rồi thêm vào CLH variant queue của AQS để chờ acquire lock.
- `acquireQueued()`: block thread và gọi `tryAcquire()` để thread trong queue thử acquire lock.

#### Phân tích `tryAcquire()`

Template method `tryAcquire()` tương ứng trong AQS như sau:

```JAVA
// AQS
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}
```

Method `tryAcquire()` là template method do AQS cung cấp nhưng không có default implementation.

Vì vậy khi phân tích method `tryAcquire()`, lấy non-fair lock (exclusive lock) của `ReentrantLock` làm ví dụ. `tryAcquire()` được triển khai bên trong `ReentrantLock` sẽ gọi đến `nonfairTryAcquire()`:

```JAVA
// ReentrantLock
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    // 1. Lấy trạng thái state trong AQS
    int c = getState();
    // 2. Nếu state bằng 0, chứng tỏ lock chưa bị thread khác chiếm dụng
    if (c == 0) {
        // 2.1. Cập nhật state thông qua CAS
        if (compareAndSetState(0, acquires)) {
            // 2.2. Nếu CAS update thành công, đặt lock holder là thread hiện tại
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // 3. Nếu thread hiện tại và thread đang hold lock là cùng một thread, nghĩa là xảy ra reentrant lock
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        // 3.1. Tăng số lần reentrant của lock
        setState(nextc);
        return true;
    }
    // 4. Nếu lock bị thread khác chiếm dụng, trả về false, biểu thị acquire lock thất bại
    return false;
}
```

Bên trong method `nonfairTryAcquire()`, acquire resource được hoàn thành chủ yếu qua hai thao tác cốt lõi:

- Dùng `CAS` update variable `state`. `state == 0` biểu thị resource chưa bị chiếm dụng. `state > 0` biểu thị resource đang bị chiếm dụng, khi đó `state` biểu thị số lần reentrant.
- Dùng `setExclusiveOwnerThread()` đặt thread đang hold resource.

Nếu thread update variable `state` thành công, điều đó biểu thị đã acquire resource, vì vậy chỉ cần đặt thread đang hold resource thành thread hiện tại.

#### Phân tích `addWaiter()`

Sau khi thử acquire resource bằng method `tryAcquire()` thất bại, method `addWaiter()` sẽ được gọi để đóng gói thread hiện tại thành Node rồi thêm vào queue bên trong `AQS`. Code `addWaiter()` như sau:

```JAVA
// AQS
private Node addWaiter(Node mode) {
    // 1. Đóng gói thread hiện tại thành Node.
    Node node = new Node(Thread.currentThread(), mode);
    Node pred = tail;
    // 2. Nếu pred != null, chứng tỏ tail node đã được khởi tạo, có thể trực tiếp thêm Node vào queue.
    if (pred != null) {
        node.prev = pred;
        // 2.1. Dùng CAS để bảo đảm concurrency safety.
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    // 3. Khởi tạo queue và thêm Node mới tạo vào queue.
    enq(node);
    return node;
}
```

**Concurrency safety khi enqueue node:**

Trong method `addWaiter()`, cần thực hiện thao tác **enqueue** Node. Vì chạy trong môi trường multi-thread nên cần dùng thao tác `CAS` để bảo đảm concurrency safety.

Dùng thao tác `CAS` update pointer `tail` trỏ tới Node mới enqueue. `CAS` bảo đảm chỉ một thread có thể sửa pointer `tail` thành công, từ đó bảo đảm concurrency safety khi enqueue Node.

**Khởi tạo queue bên trong AQS:**

Khi thực thi `addWaiter()`, nếu phát hiện `pred == null`, tức pointer `tail` là null, điều đó chứng tỏ queue chưa được khởi tạo. Cần gọi method `enq()` để khởi tạo queue và thêm Node vào queue sau đó. Code như sau:

```JAVA
// AQS
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) {
            // 1. Dùng CAS để bảo đảm concurrency safety khi initialize queue
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            // 2. Giống thao tác enqueue node trong method addWaiter()
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

Trong method `enq()`, queue được khởi tạo; trong quá trình này cũng cần dùng `CAS` để bảo đảm concurrency safety.

Khởi tạo queue gồm hai bước: khởi tạo node `head`, rồi để `tail` trỏ tới node `head`.

**Queue sau khi khởi tạo như hình dưới:**

![](https://oss.javaguide.cn/github/javaguide/java/concurrent/clh-queue-structure-init.png)

#### Phân tích `acquireQueued()`

Để tiện theo dõi, nhắc lại code acquire resource bằng `acquire()` trong `AQS`:

```JAVA
// AQS
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

Trong method `acquire()`, sau khi thêm Node vào queue bằng method `addWaiter()`, method `acquireQueued()` sẽ được gọi. Code như sau:

```JAVA
// AQS: cho các node trong queue thử acquire lock và block thread.
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            // 1. Thử acquire lock.
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            // 2. Kiểm tra thread có thể block không; nếu có thì block thread hiện tại.
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        // 3. Nếu acquire lock thất bại, hủy acquire lock và update trạng thái node thành CANCELLED.
        if (failed)
            cancelAcquire(node);
    }
}
```

Trong method `acquireQueued()`, chủ yếu thực hiện hai việc:

- **Thử acquire resource:** sau khi thread hiện tại enqueue, nếu phát hiện predecessor là node `head`, chứng tỏ thread hiện tại là node waiting đầu tiên trong queue, nên gọi `tryAcquire()` để thử acquire resource.

- **Block thread hiện tại:** nếu thử acquire resource thất bại, cần block thread hiện tại và chờ được wake-up để acquire resource.

**1. Thử acquire resource**

Trong method `acquireQueued()`, thử acquire resource gồm 2 bước:

- `p == head`: biểu thị predecessor của node hiện tại là node `head`. Lúc này node hiện tại là node waiting đầu tiên trong AQS queue.
- `tryAcquire(arg) == true`: biểu thị thread hiện tại thử acquire resource thành công.

Sau khi acquire resource thành công, cần **đưa node của thread hiện tại ra khỏi waiting queue**. Thao tác này là đặt node hiện tại thành node `head` (`head` là virtual node, không tham gia xếp hàng acquire resource).

**2. Block thread hiện tại**

Trong `AQS`, việc wake-up node hiện tại phụ thuộc vào node trước đó. Nếu node trước đó hủy acquire lock, trạng thái sẽ chuyển thành `CANCELLED`; node ở trạng thái `CANCELLED` chưa acquire được lock nên cũng không thể thực hiện unlock để wake-up node hiện tại. Vì vậy trước khi block thread hiện tại, cần bỏ qua các node ở trạng thái `CANCELLED`.

Dùng method `shouldParkAfterFailedAcquire()` để kiểm tra node thread hiện tại có thể block hay không:

```JAVA
// AQS: kiểm tra node thread hiện tại có thể block hay không.
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    // 1. Trạng thái predecessor bình thường, trực tiếp trả về true.
    if (ws == Node.SIGNAL)
        return true;
    // 2. ws > 0 biểu thị trạng thái predecessor bất thường, tức CANCELLED; cần bỏ qua node bất thường.
    if (ws > 0) {
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        // 3. Nếu trạng thái predecessor không phải SIGNAL cũng không phải CANCELLED, đặt trạng thái thành SIGNAL.
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

Logic kiểm tra trong method `shouldParkAfterFailedAcquire()`:

- Nếu phát hiện trạng thái predecessor là `SIGNAL`, có thể block thread hiện tại.
- Nếu phát hiện trạng thái predecessor là `CANCELLED`, cần bỏ qua node ở trạng thái `CANCELLED`.
- Nếu phát hiện trạng thái predecessor không phải `SIGNAL` và cũng không phải `CANCELLED`, chứng tỏ predecessor đang ở trạng thái waiting resource bình thường, nên đặt trạng thái predecessor thành `SIGNAL`, biểu thị predecessor cần wake-up successor.

Sau khi xác định thread hiện tại có thể block, gọi method `parkAndCheckInterrupt()` để block thread hiện tại. Bên trong sử dụng `LockSupport` để triển khai blocking. Ở tầng dưới, `LockSupport` dựa trên class `Unsafe` để block thread, code như sau:

```JAVA
// AQS
private final boolean parkAndCheckInterrupt() {
    // Thread block tại đây
    LockSupport.park(this);
    // Sau khi thread được wake-up, trả về interrupt status của thread
    return Thread.interrupted();
}
```

**Vì sao sau khi thread được wake-up phải trả về interrupt status?**

Trong method `parkAndCheckInterrupt()`, sau khi thực thi `LockSupport.park(this)`, thread sẽ bị block. Code như sau:

```JAVA
// AQS
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    // Sau khi thread được wake-up, cần trả về interrupt status của thread
    return Thread.interrupted();
}
```

Sau khi thread được wake-up, cần thực thi `Thread.interrupted()` để trả về interrupt status của thread. Vì sao?

Điều này liên quan đến cơ chế phối hợp interrupt của thread. Sau khi thread được wake-up, chưa thể xác định nó được wake-up bởi interrupt hay bởi `LockSupport.unpark()`, nên cần dựa vào interrupt status của thread để phán đoán.

**Trong method `acquire()`, vì sao cần gọi `selfInterrupt()`?**

Code method `acquire()` như sau:

```JAVA
// AQS
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

Trong method `acquire()`, khi điều kiện của câu lệnh `if` trả về `true`, `selfInterrupt()` sẽ được gọi. Method này interrupt thread hiện tại. Vì sao cần interrupt thread hiện tại?

Khi điều kiện `if` là `true`, cần `tryAcquire()` trả về `false` và `acquireQueued()` trả về `true`.

Giá trị trả về của method `acquireQueued()` là **interrupt status** sau khi thread được wake-up, được trả về bằng `Thread.interrupted()`. Method này vừa trả về interrupt status vừa clear interrupt status của thread.

Vì vậy nếu điều kiện `if` là `true`, điều đó biểu thị interrupt status của thread là `true`, nhưng sau khi gọi `Thread.interrupted()`, interrupt status của thread đã bị clear thành `false`; do đó cần thực thi lại `selfInterrupt()` để set lại interrupt status của thread.

### Phân tích source code release resource của AQS (exclusive mode)

Entry method để release resource ở exclusive mode trong AQS là `release()`, code như sau:

```JAVA
// AQS
public final boolean release(int arg) {
    // 1. Thử release lock
    if (tryRelease(arg)) {
        Node h = head;
        // 2. Wake-up successor
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

Trong method `release()`, chủ yếu thực hiện hai việc: thử release lock và wake-up successor. Các method tương ứng như sau:

**1. Thử release lock**

Dùng method `tryRelease()` để thử release lock. Đây là template method do custom synchronizer triển khai, vì vậy ở đây vẫn dùng `ReentrantLock` làm ví dụ.

Method `tryRelease()` được triển khai trong `ReentrantLock` như sau:

```JAVA
// ReentrantLock
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    // 1. Kiểm tra thread đang hold lock có phải thread hiện tại không
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    // 2. Nếu state bằng 0, biểu thị thread hiện tại không còn reentrant count. Cập nhật free thành true, biểu thị thread sẽ release lock.
    if (c == 0) {
        free = true;
        // 3. Cập nhật thread đang hold resource thành null
        setExclusiveOwnerThread(null);
    }
    // 4. Cập nhật giá trị state
    setState(c);
    return free;
}
```

Trong method `tryRelease()`, trước tiên tính giá trị `state` sau khi release lock, rồi kiểm tra giá trị `state` có bằng 0 không.

- Nếu `state == 0`, biểu thị thread không còn reentrant count, cập nhật `free = true` và đổi thread đang hold resource thành null, biểu thị thread đã release hoàn toàn lock này.
- Nếu `state != 0`, biểu thị thread vẫn còn reentrant count, nên không cập nhật giá trị `free`; `free` là `false`, biểu thị thread chưa release hoàn toàn lock này.

Sau đó update giá trị `state` và trả về `free`; giá trị `free` biểu thị thread có release hoàn toàn lock hay không.

**2. Wake-up successor**

Nếu `tryRelease()` trả về `true`, biểu thị thread không còn reentrant count và lock đã được release hoàn toàn, vì vậy cần wake-up successor.

Trước khi wake-up successor, cần kiểm tra có thể wake-up successor hay không. Điều kiện là `h != null && h.waitStatus != 0`. Giải thích như sau:

- `h == null`: biểu thị node `head` chưa được initialize, tức queue trong AQS chưa được initialize nên không thể wake-up thread node trong queue.
- `h != null && h.waitStatus == 0`: biểu thị head node vừa được initialize xong (trạng thái initialize của node là 0), thread successor chưa enqueue thành công nên chưa cần wake-up các thread phía sau. (Sau khi successor enqueue, nó sẽ đổi trạng thái predecessor thành `SIGNAL`, biểu thị cần wake-up successor.)
- `h != null && h.waitStatus != 0`: `waitStatus` có thể lớn hơn 0 hoặc nhỏ hơn 0. `> 0` biểu thị node đã hủy việc waiting acquire resource, `< 0` biểu thị node đang ở trạng thái waiting bình thường.

Tiếp theo đi vào method `unparkSuccessor()` để xem cách wake-up successor:

```JAVA
// AQS: tham số node ở đây là head node của queue (virtual head node)
private void unparkSuccessor(Node node) {
    int ws = node.waitStatus;
    // 1. Clear trạng thái head node để chuẩn bị cho wake-up tiếp theo.
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    Node s = node.next;
    // 2. Nếu successor bất thường, duyệt ngược từ tail để tìm node ở trạng thái bình thường rồi wake-up.
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        // 3. Wake-up successor
        LockSupport.unpark(s.thread);
}
```

Trong `unparkSuccessor()`, nếu trạng thái head node `< 0` (trong tình huống bình thường, chỉ cần có successor thì trạng thái head node phải là `SIGNAL`, tức -1), điều đó biểu thị cần wake-up successor. Vì vậy trước tiên clear trạng thái head node, đổi trạng thái thành 0, biểu thị đã thực hiện thao tác wake-up successor.

Nếu `s == null` hoặc `s.waitStatus > 0`, biểu thị successor bất thường. Khi đó không thể wake-up node bất thường mà phải tìm node ở trạng thái bình thường để wake-up.

Vì vậy cần duyệt ngược từ pointer `tail` để tìm node đầu tiên ở trạng thái bình thường (`waitStatus <= 0`) và wake-up node đó.

**Vì sao phải duyệt ngược từ pointer `tail` thay vì duyệt xuôi từ pointer `head` để tìm node ở trạng thái bình thường?**

Hướng duyệt liên quan đến **thao tác enqueue node**. Method enqueue như sau:

```JAVA
// AQS: method enqueue node
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    Node pred = tail;
    if (pred != null) {
        // 1. Update pointer prev trước.
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            // 2. Sau đó mới update pointer next.
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}
```

Trong method `addWaiter()`, enqueue node `node` cần update hai pointer `node.prev` và `pred.next`, nhưng hai thao tác này không phải **atomic operation**. Pointer `node.prev` được update trước, sau đó mới update pointer `pred.next`.

Trong tình huống cực đoan, có thể xảy ra việc node tiếp theo của `head` có trạng thái `CANCELLED`, khi đó node mới enqueue chỉ vừa update pointer `node.prev` nhưng chưa update pointer `pred.next`, như hình dưới:

![](https://oss.javaguide.cn/github/javaguide/java/concurrent/aqs-addWaiter.png)

Nếu duyệt xuôi từ pointer `head`, sẽ không tìm được node mới enqueue, vì vậy cần duyệt ngược từ pointer `tail` để tìm node mới enqueue.

### Minh họa nguyên lý hoạt động của AQS (exclusive mode)

Đến đây đã phân tích xong source code acquire và release resource ở exclusive mode trong AQS. Để có nhận thức rõ hơn về nguyên lý hoạt động AQS và thay đổi trạng thái node, tiếp theo sẽ dùng hình vẽ để tìm hiểu toàn bộ nguyên lý hoạt động AQS.

Vì AQS là công cụ synchronization ở tầng dưới, các method acquire và release resource không cung cấp triển khai cụ thể, nên ở đây dùng `ReentrantLock` để minh họa bằng hình.

Giả sử có tổng cộng 3 thread thử acquire lock, lần lượt là `T1`, `T2` và `T3`.

Giả sử thread `T1` acquire lock trước, thread `T2` xếp hàng chờ acquire lock. Trước khi thread `T2` vào queue, cần initialize queue bên trong AQS. Sau khi initialize, trạng thái node `head` là `0`. Queue bên trong AQS sau khi initialize như hình dưới:

![](https://oss.javaguide.cn/github/javaguide/java/concurrent/aqs-acquire-and-release-process.png)

Lúc này thread `T2` thử acquire lock. Vì thread `T1` đang hold lock, thread `T2` sẽ vào queue chờ acquire lock. Đồng thời trạng thái predecessor (node `head`) được đổi từ `0` thành `SIGNAL`, biểu thị cần wake-up successor của node `head`. Queue bên trong AQS lúc này như hình dưới:

![](https://oss.javaguide.cn/github/javaguide/java/concurrent/aqs-acquire-and-release-process-2.png)

Lúc này thread `T3` thử acquire lock. Vì thread `T1` đang hold lock, thread `T3` sẽ vào queue chờ acquire lock. Đồng thời trạng thái predecessor (node thread `T2`) được đổi từ `0` thành `SIGNAL`, biểu thị node thread `T2` cần wake-up successor. Queue bên trong AQS lúc này như hình dưới:

![](https://oss.javaguide.cn/github/javaguide/java/concurrent/aqs-acquire-and-release-process-3.png)

Giả sử thread `T1` release lock, successor `T2` sẽ được wake-up. Sau khi được wake-up, thread `T2` acquire lock và thoát khỏi waiting queue.

Node thread `T2` thoát waiting queue không phải bằng cách trực tiếp remove khỏi queue, mà bằng cách biến node thread `T2` thành node `head` mới, nhờ đó kết thúc việc waiting acquire resource. Queue bên trong AQS lúc này như sau:

![](https://oss.javaguide.cn/github/javaguide/java/concurrent/aqs-acquire-and-release-process-4.png)

Giả sử thread `T2` release lock, successor `T3` sẽ được wake-up. Sau khi thread `T3` acquire lock, nó cũng thoát waiting queue bằng cách biến node thread `T3` thành node `head`, kết thúc việc waiting acquire resource. Queue bên trong AQS lúc này như sau:

![](https://oss.javaguide.cn/github/javaguide/java/concurrent/aqs-acquire-and-release-process-5.png)

### Phân tích source code acquire resource của AQS (shared mode)

Entry method để acquire resource ở shared mode trong AQS là `acquireShared()`, như sau:

```JAVA
// AQS
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```

Trong method `acquireShared()`, trước tiên sẽ thử acquire shared lock. Nếu acquire thất bại, thread hiện tại được thêm vào queue và block, chờ wake-up rồi thử acquire shared lock. Hai thao tác này tương ứng với `tryAcquireShared()` và `doAcquireShared()`.

Trong đó `tryAcquireShared()` là template method do AQS cung cấp, synchronizer triển khai logic cụ thể. Vì vậy ở đây dùng `Semaphore` làm ví dụ để phân tích cách acquire resource ở shared mode.

#### Phân tích `tryAcquireShared()`

`Semaphore` triển khai fair lock và non-fair lock. Tiếp theo phân tích source code `tryAcquireShared()` với non-fair lock làm ví dụ.

Method `tryAcquireShared()` được override trong `Semaphore` sẽ gọi method `nonfairTryAcquireShared()` bên dưới:

```JAVA
// Semaphore override template method của AQS
protected int tryAcquireShared(int acquires) {
    return nonfairTryAcquireShared(acquires);
}

// Semaphore
final int nonfairTryAcquireShared(int acquires) {
    for (;;) {
        // 1. Lấy số resource khả dụng.
        int available = getState();
        // 2. Tính số resource còn lại.
        int remaining = available - acquires;
        // 3. Nếu số resource còn lại < 0, biểu thị resource không đủ, trả về ngay; nếu CAS update state thành công, biểu thị thread hiện tại đã acquire shared resource, trả về ngay.
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```

Ở shared mode, giá trị `state` trong AQS biểu thị số lượng shared resource.

Trong method `nonfairTryAcquireShared()`, liên tục thử acquire resource trong infinite loop, và thoát loop nếu **số resource còn lại không đủ** hoặc **thread hiện tại acquire resource thành công**. Method trả về **số resource còn lại**. Dựa trên giá trị trả về, có 3 trường hợp:

- **Số resource còn lại > 0**: biểu thị acquire resource thành công và các thread tiếp theo cũng có thể acquire resource thành công.
- **Số resource còn lại = 0**: biểu thị acquire resource thành công nhưng các thread tiếp theo không thể acquire resource thành công.
- **Số resource còn lại < 0**: biểu thị acquire resource thất bại.

#### Phân tích `doAcquireShared()`

Để tiện theo dõi, nhắc lại entry method acquire resource `acquireShared()`:

```JAVA
// AQS
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```

Trong method `acquireShared()`, trước tiên gọi `tryAcquireShared()` để thử acquire resource.

Nếu giá trị trả về của method `< 0`, tức số resource còn lại nhỏ hơn 0, biểu thị thread hiện tại acquire resource thất bại. Vì vậy sẽ vào method `doAcquireShared()`, thêm thread hiện tại vào AQS queue để waiting. Như sau:

```JAVA
// AQS
private void doAcquireShared(int arg) {
    // 1. Thêm thread hiện tại vào queue để waiting.
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                // 2. Nếu thread hiện tại là node đầu tiên của waiting queue, thử acquire resource.
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    // 3. Remove node thread hiện tại khỏi waiting queue và wake-up successor.
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        // 3. Nếu acquire resource thất bại, hủy acquire resource và update trạng thái node thành CANCELLED.
        if (failed)
            cancelAcquire(node);
    }
}
```

Vì thread hiện tại đã thử acquire resource nhưng thất bại, nên trong method `doAcquireShared()` cần đóng gói thread hiện tại thành Node rồi thêm vào queue để waiting.

Điểm khác biệt lớn nhất giữa acquire resource ở **shared mode** và **exclusive mode** là: ở shared mode, số lượng resource có thể lớn hơn 1, tức nhiều thread có thể cùng hold resource.

Vì vậy ở shared mode, sau khi thread được wake-up và acquire resource, nếu phát hiện vẫn còn resource thì nó sẽ thử wake-up thread phía sau để thread đó thử acquire resource. Method `setHeadAndPropagate()` tương ứng như sau:

```JAVA
// AQS
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head;
    // 1. Remove node thread hiện tại khỏi waiting queue.
    setHead(node);
    // 2. Wake-up node waiting phía sau.
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        Node s = node.next;
        if (s == null || s.isShared())
            doReleaseShared();
    }
}
```

Trong method `setHeadAndPropagate()`, wake-up successor cần thỏa mãn một số điều kiện, chủ yếu là 2 điều kiện:

- `propagate > 0`: `propagate` biểu thị số resource còn lại sau khi acquire resource. Nếu `> 0`, có thể wake-up thread phía sau để acquire resource.
- `h.waitStatus < 0`: node `h` là node `head` trước khi thực thi `setHead()`. Khi kiểm tra `head.waitStatus`, dùng `< 0` chủ yếu để xác định trạng thái `head` là `SIGNAL` hoặc `PROPAGATE`. Nếu node `head` là `SIGNAL`, có thể wake-up successor; nếu trạng thái node `head` là `PROPAGATE`, cũng có thể wake-up successor (nhằm giải quyết vấn đề trong môi trường concurrency, sẽ giải thích chi tiết sau).

Điều kiện `if` để **wake-up successor waiting node** trong code hơi phức tạp. Vì sao lại viết như vậy?

```JAVA
if (propagate > 0 || h == null || h.waitStatus < 0 ||
    (h = head) == null || h.waitStatus < 0)
```

- `h == null || h.waitStatus < 0`: `h == null` dùng để tránh null pointer exception. Trong tình huống bình thường, `h` không là `null`, vì trước khi thực thi đến đây, node hiện tại đã enqueue, queue không thể chưa initialize.

  `h.waitStatus < 0` chủ yếu kiểm tra trạng thái node `head` có phải `SIGNAL` hoặc `PROPAGATE` không; dùng trực tiếp `< 0` để kiểm tra sẽ thuận tiện hơn.

- `(h = head) == null || h.waitStatus < 0`: nếu đến đây nghĩa là điều kiện `h.waitStatus < 0` trước đó đã đúng, biểu thị có concurrency.

  Đồng thời có thread khác đang wake-up successor và đã đổi giá trị node `head` từ `SIGNAL` thành `0`. Vì vậy ở đây lấy lại node `head` mới; node `head` lần này là node thread hiện tại được set bằng `setHead()`, sau đó tiếp tục kiểm tra trạng thái `waitStatus`.

Nếu điều kiện `if` đúng, code sẽ đi vào method `doReleaseShared()` để wake-up successor waiting node, như sau:

```JAVA
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        // 1. Queue cần có ít nhất một thread node đang waiting.
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            // 2. Nếu trạng thái node head là SIGNAL, có thể wake-up successor.
            if (ws == Node.SIGNAL) {
                // 2.1 Clear trạng thái SIGNAL của node head và update thành 0. Biểu thị đã wake-up successor của node này.
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;
                // 2.2 Wake-up successor
                unparkSuccessor(h);
            }
            // 3. Nếu trạng thái node head là 0, update thành PROPAGATE để giải quyết vấn đề trong concurrency; phần sau sẽ giải thích chi tiết.
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;
        }
        if (h == head)
            break;
    }
}
```

Trong method `doReleaseShared()`, kiểm tra trạng thái `waitStatus` của node `head` để quyết định thao tác tiếp theo, có hai trường hợp:

- Trạng thái node `head` là `SIGNAL`: biểu thị node `head` có successor cần wake-up, nên dùng thao tác `CAS` đổi trạng thái `SIGNAL` của node `head` thành `0`. Xóa trạng thái `SIGNAL` để biểu thị đã thực hiện wake-up successor của node `head`.
- Trạng thái node `head` là `0`: biểu thị có concurrency, cần đổi `0` thành `PROPAGATE` để bảo đảm thread được wake-up bình thường trong môi trường concurrency.

#### Vì sao cần trạng thái `PROPAGATE`?

Trong AQS, `PROPAGATE` của Node dùng để xử lý vấn đề thread node có thể không được wake-up trong môi trường concurrency. `PROPAGATE` chỉ được dùng trong method `doReleaseShared()`.

**Tiếp theo phân tích qua ví dụ vì sao cần trạng thái `PROPAGATE`.**

Ở shared mode, call chain của các method acquire và release resource như sau:

- Call chain acquire resource của thread: `acquireShared() -> tryAcquireShared() -> thread block chờ wake-up -> tryAcquireShared() -> setHeadAndPropagate() -> nếu (số resource còn lại > 0) || (head.waitStatus < 0) thì wake-up successor`.

- Call chain release resource của thread: `releaseShared() -> tryReleaseShared() -> doReleaseShared()`.

**Nếu khi release resource không đổi trạng thái node `head` từ `0` thành `PROPAGATE`:**

Giả sử có tổng cộng 4 thread thử acquire resource ở shared mode và tổng cộng 2 resource. Ban đầu thread `T3` và `T4` acquire được resource, thread `T1` và `T2` không acquire được nên xếp hàng trong queue.

- Ở thời điểm 1, thread `T1` và `T2` đang trong waiting queue, `T3` và `T4` đang hold resource. Node và trạng thái tương ứng trong waiting queue (giá trị trong ngoặc là `waitStatus` của node):

  `head(-1) -> T1(-1) -> T2(0)`.

- Ở thời điểm 2, thread `T3` release resource, đổi trạng thái node `head` từ `SIGNAL` thành `0` thông qua method `doReleaseShared()`, đồng thời wake-up thread `T1`, sau đó thread `T3` thoát.

  Sau khi được wake-up, thread `T1` acquire resource thông qua `tryAcquireShared()`, nhưng chưa kịp thực thi `setHeadAndPropagate()` để đặt mình thành node `head`. Trạng thái node trong waiting queue lúc này:

  `head(0) -> T1(-1) -> T2(0)`.

- Ở thời điểm 3, thread `T4` release resource. Vì trạng thái node `head` lúc này là `0`, nếu trong `doReleaseShared()` không làm gì khi `ws == 0` (tức không có trạng thái `PROPAGATE`), `T4` không thể wake-up successor của `head`, sau đó thread `T4` thoát.

- Ở thời điểm 4, thread `T1` tiếp tục thực thi method `setHeadAndPropagate()`. Trước tiên lưu `head` cũ (`waitStatus == 0`), sau đó thực thi `setHead(T1)` để đặt mình thành node `head`.

  Lúc này `propagate == 0`, `old head.waitStatus == 0`, hai điều kiện đầu không thỏa mãn. Tuy nhiên vì `setHead()` không reset `waitStatus`, `waitStatus` của `head` mới (tức node `T1` ban đầu) vẫn là `-1` (`SIGNAL`), nên lần kiểm tra thứ hai `(h = head).waitStatus < 0` vẫn đúng và sẽ gọi `doReleaseShared()` để wake-up `T2`.

  Trong thứ tự thời gian này, lần kiểm tra `head` thứ hai thực sự có thể xử lý tình huống. Vậy `PROPAGATE` giải quyết vấn đề gì?

- Ở thời điểm 5 (thứ tự thời gian cực đoan hơn), xét interleaving sau: sau khi thread `T1` thực thi xong `setHead(T1)` và **trước khi kiểm tra `new head.waitStatus`**, `doReleaseShared()` của thread `T4` vừa thực thi `unparkSuccessor()` để wake-up `T2`; `T2` nhanh chóng acquire resource và thực thi `setHead(T2)` để đặt mình thành `head` mới. Vì `T2` ban đầu là node cuối queue nên `waitStatus` của nó là `0`. Khi `T1` tiếp tục đọc `head` mới, nó đọc được node `T2` (`waitStatus == 0`), lần kiểm tra thứ hai lúc này cũng không thể thông qua.

  Trong thứ tự thời gian concurrency cực đoan này, `propagate == 0`, `old head.waitStatus == 0`, `new head.waitStatus == 0`, cả ba điều kiện đều không thỏa mãn nên `T1` không gọi `doReleaseShared()`. Nếu lúc này queue vẫn còn node waiting phía sau, tín hiệu wake-up sẽ bị mất.

Bảng thời gian tương ứng:

| Thời điểm   | Thread T1                                                                                                            | Thread T2                                                                                         | Thread T3                        | Thread T4                                                                                          | Waiting queue                       |
| ----------- | -------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- | -------------------------------- | -------------------------------------------------------------------------------------------------- | ----------------------------------- |
| Thời điểm 1 | Waiting queue                                                                                                        | Waiting queue                                                                                     | Hold resource                    | Hold resource                                                                                      | `head(-1) -> T1(-1) -> T2(0)`       |
| Thời điểm 2 | (Đang thực thi) được wake-up, acquire resource nhưng chưa kịp đặt mình thành node `head`                             | Waiting queue                                                                                     | (Đang thực thi) release resource | Hold resource                                                                                      | `head(0) -> T1(-1) -> T2(0)`        |
| Thời điểm 3 |                                                                                                                      | Waiting queue                                                                                     | Đã thoát                         | (Đang thực thi) release resource. Nhưng trạng thái node `head` là `0`, không thể wake-up successor | `head(0) -> T1(-1) -> T2(0)`        |
| Thời điểm 4 | (Đang thực thi) hoàn tất `setHead(T1)`, chưa kiểm tra `new head.waitStatus`                                          | Waiting queue                                                                                     | Đã thoát                         | Đã thoát                                                                                           | `head(-1, node thread T1) -> T2(0)` |
| Thời điểm 5 |                                                                                                                      | (Đang thực thi) được T4 wake-up, acquire resource, thực thi `setHead(T2)` và trở thành `head` mới | Đã thoát                         | (Đang thực thi) `doReleaseShared()` wake-up T2                                                     | `head(0, node thread T2)`           |
| Thời điểm 6 | (Đang thực thi) đọc `head` mới là node T2, `waitStatus == 0`, lần kiểm tra thứ hai thất bại, không wake-up successor | Đã acquire resource                                                                               | Đã thoát                         | Đã thoát                                                                                           | `head(0, node thread T2)`           |

**Nếu khi thread release resource, đổi trạng thái node `head` từ `0` thành `PROPAGATE`, có thể giải quyết vấn đề concurrency nêu trên như sau:**

Điểm cốt lõi của `PROPAGATE` là: nó sửa trạng thái của **old `head`**, còn reference của old `head` đã được lưu vào local variable `h` ngay đầu method `setHeadAndPropagate()`, nên không bị ảnh hưởng bởi các thay đổi concurrency về sau.

- Ở thời điểm 1~2, tình huống giống như trên:

  Thời điểm 1: `head(-1) -> T1(-1) -> T2(0)`.

  Thời điểm 2: `T3` release resource, trạng thái `head` thành `0` và wake-up `T1`.

- Ở thời điểm 3, thread `T4` release resource. Vì trạng thái node `head` lúc này là `0`, `doReleaseShared()` đổi trạng thái node `head` từ `0` thành `PROPAGATE(-3)`, sau đó thread `T4` thoát. Trạng thái node trong waiting queue lúc này:

  `head(PROPAGATE) -> T1(-1) -> T2(0)`.

- Ở thời điểm 4, thread `T1` tiếp tục thực thi method `setHeadAndPropagate()`. Trước tiên lưu old `head` vào local variable `h`, lúc này `h.waitStatus == PROPAGATE(-3)`. Sau đó thực thi `setHead(T1)` để đặt mình thành node `head`.

- Ở thời điểm 5, ngay cả khi xảy ra interleaving cực đoan giống trước đó (`T4` wake-up `T2`, `T2` trở thành `head` mới), khi `T1` kiểm tra:

  - `propagate > 0` → `0 > 0` → **false**
  - `h == null` → **false**
  - `h.waitStatus < 0` → `PROPAGATE(-3) < 0` → **true**!

  Vì reference của old `head` `h` đã được lưu ở đầu method nên không bị ảnh hưởng bởi `setHead()` và thao tác concurrency về sau. Do đó trạng thái `PROPAGATE` bảo đảm `h.waitStatus < 0` chắc chắn thông qua. Vì vậy thread `T1` sẽ gọi `doReleaseShared()` trong method `setHeadAndPropagate()` để wake-up successor.

Nhờ trạng thái `PROPAGATE`, có thể tránh mất tín hiệu wake-up trong thứ tự thời gian concurrency cực đoan. Bảng thời gian tương ứng:

| Thời điểm   | Thread T1                                                                                                             | Thread T2                                                                                | Thread T3                        | Thread T4                                                                                          | Waiting queue                        |
| ----------- | --------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- | -------------------------------- | -------------------------------------------------------------------------------------------------- | ------------------------------------ |
| Thời điểm 1 | Waiting queue                                                                                                         | Waiting queue                                                                            | Hold resource                    | Hold resource                                                                                      | `head(-1) -> T1(-1) -> T2(0)`        |
| Thời điểm 2 | (Đang thực thi) được wake-up, acquire resource nhưng chưa kịp đặt mình thành node `head`                              | Waiting queue                                                                            | (Đang thực thi) release resource | Hold resource                                                                                      | `head(0) -> T1(-1) -> T2(0)`         |
| Thời điểm 3 | Chưa tiếp tục thực thi                                                                                                | Waiting queue                                                                            | Đã thoát                         | (Đang thực thi) release resource. Lúc này trạng thái node `head` được đổi từ `0` thành `PROPAGATE` | `head(PROPAGATE) -> T1(-1) -> T2(0)` |
| Thời điểm 4 | (Đang thực thi) lưu old `head` (`waitStatus == PROPAGATE`), thực thi `setHead(T1)`                                    | Waiting queue                                                                            | Đã thoát                         | Đã thoát                                                                                           | `head(-1, node thread T1) -> T2(0)`  |
| Thời điểm 5 | (Đang thực thi) kiểm tra old `h.waitStatus < 0` (`PROPAGATE(-3) < 0`) đúng, gọi `doReleaseShared()` wake-up successor | Waiting queue                                                                            | Đã thoát                         | Đã thoát                                                                                           | `head(0, node thread T1) -> T2(0)`   |
| Thời điểm 6 | Đã thoát                                                                                                              | (Đang thực thi) thread `T2` được wake-up, acquire resource và đặt mình thành node `head` | Đã thoát                         | Đã thoát                                                                                           | `head(0, node thread T2)`            |

Tóm lại: trạng thái `PROPAGATE` và lần kiểm tra `head` thứ hai trong `setHeadAndPropagate()` là **hai lớp bảo vệ** của cùng một bug fix trong JDK 7 ([JDK-6801020](https://bugs.openjdk.org/browse/JDK-6801020)). `PROPAGATE` cung cấp bảo đảm đáng tin cậy hơn bằng cách sửa trạng thái old `head`, vì reference của old `head` đã được lưu vào local variable ngay đầu method và không bị thao tác `setHead()` đồng thời thay thế.

### Phân tích source code release resource của AQS (shared mode)

Entry method để release resource ở shared mode trong AQS là `releaseShared()`, code như sau:

```JAVA
// AQS
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```

Method `tryReleaseShared()` là template method do AQS cung cấp. Ở đây cũng dùng `Semaphore` để giải thích:

```JAVA
// Semaphore
protected final boolean tryReleaseShared(int releases) {
    for (;;) {
        int current = getState();
        int next = current + releases;
        if (next < current) // overflow
            throw new Error("Maximum permit count exceeded");
        if (compareAndSetState(current, next))
            return true;
    }
}
```

Trong method `tryReleaseShared()` do `Semaphore` triển khai, liên tục thử release resource trong vòng lặp vô hạn, tức dùng thao tác `CAS` để update giá trị `state`.

Nếu update thành công, chứng tỏ release resource thành công và sẽ đi vào method `doReleaseShared()`.

Method `doReleaseShared()` đã được phân tích chi tiết trong phần acquire resource (shared mode) ở trên, nên không lặp lại ở đây.

### Cơ chế hoạt động của Condition queue

Trong bảng trạng thái `waitStatus` phía trước đã đề cập trạng thái `CONDITION` (giá trị -2), biểu thị node đang waiting trong Condition queue. Phần này giải thích có hệ thống cơ chế hoạt động của Condition queue.

#### Condition là gì?

`Condition` là interface được định nghĩa trong package `java.util.concurrent.locks`. Nó cung cấp cơ chế thread waiting/notification tương tự `Object.wait()` / `Object.notify()`, nhưng mạnh mẽ và linh hoạt hơn. `Condition` phải được sử dụng cùng `Lock`, giống như `wait/notify` phải được sử dụng cùng `synchronized`.

So với `Object` `wait/notify`, ưu điểm chính của `Condition` là:

- **Hỗ trợ nhiều waiting queue**: một `Lock` có thể tạo nhiều instance `Condition`, các thread khác nhau có thể waiting trên các condition khác nhau, thực hiện phối hợp thread tinh vi hơn. `synchronized` chỉ có một waiting queue.
- **Hỗ trợ waiting không phản hồi interrupt**: `Condition` cung cấp method `awaitUninterruptibly()`.
- **Hỗ trợ waiting có timeout**: `Condition` cung cấp method `awaitNanos(long)` và `await(long, TimeUnit)`, có thể đặt deadline waiting.

#### Hai queue trong AQS

Thực tế bên trong AQS duy trì **hai queue**:

1. **Synchronization queue (CLH variant queue)**: là queue hai chiều đã phân tích chi tiết ở trên, dùng để lưu các thread node chờ vì acquire resource thất bại.
2. **Condition queue (Condition Queue)**: là singly linked list, dùng để lưu các thread node chờ do gọi method `Condition.await()`. Mỗi instance `Condition` duy trì một Condition queue độc lập.

Node trong Condition queue dùng pointer `nextWaiter` của `Node` để link tới node tiếp theo, tạo thành singly linked list. Node đầu Condition queue là `firstWaiter`, node cuối là `lastWaiter`.

#### Quy trình hoạt động cốt lõi của Condition

Inner class `ConditionObject` của AQS triển khai interface `Condition`. Các method cốt lõi là `await()` và `signal()`.

**Quy trình hoạt động của method `await()`:**

1. Đóng gói thread hiện tại thành Node (đặt `waitStatus` thành `CONDITION`) và thêm vào cuối Condition queue.
2. Release hoàn toàn lock mà thread hiện tại đang hold (đặt giá trị `state` thành 0), đồng thời lưu giá trị `state` trước khi release.
3. Block thread hiện tại, chờ được wake-up bởi `signal()` hoặc bị interrupt.
4. Sau khi được wake-up, thông qua `acquireQueued()` để vào lại synchronization queue cạnh tranh lock và khôi phục giá trị `state` đã lưu trước đó (số lần reentrant).

**Quy trình hoạt động của method `signal()`:**

1. Kiểm tra thread gọi `signal()` có hold lock hay không (nếu không hold sẽ throw `IllegalMonitorStateException`).
2. Remove node waiting đầu tiên khỏi Condition queue.
3. Đổi `waitStatus` của node đó từ `CONDITION` thành `0`, rồi dùng method `enq()` thêm node vào cuối synchronization queue.
4. Nếu trạng thái predecessor trong synchronization queue bất thường (`CANCELLED`) hoặc CAS đặt trạng thái predecessor thành `SIGNAL` thất bại, trực tiếp wake-up thread đó.

Method `signalAll()` tương tự `signal()`, khác ở chỗ nó chuyển **tất cả** node trong Condition queue sang synchronization queue.

Ví dụ code dưới đây minh họa cách dùng điển hình của `Condition` để triển khai một blocking queue có bound đơn giản:

```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class SimpleBlockingQueue<T> {
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;
    private final ReentrantLock lock = new ReentrantLock();
    // Hai Condition queue khác nhau: lần lượt dùng cho "queue chưa đầy" và "queue không rỗng"
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();

    public SimpleBlockingQueue(int capacity) {
        this.capacity = capacity;
    }

    /**
     * Thêm phần tử vào queue, nếu queue đầy thì waiting.
     */
    public void put(T item) throws InterruptedException {
        lock.lock();
        try {
            // Khi queue đầy, waiting trên condition notFull
            while (queue.size() == capacity) {
                notFull.await();
            }
            queue.offer(item);
             // Sau khi thêm phần tử, signal consumer thread đang waiting trên condition notEmpty
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    /**
     * Lấy phần tử khỏi queue, nếu queue rỗng thì waiting.
     */
    public T take() throws InterruptedException {
        lock.lock();
        try {
            // Khi queue rỗng, waiting trên condition notEmpty
            while (queue.isEmpty()) {
                notEmpty.await();
            }
            T item = queue.poll();
             // Sau khi lấy phần tử, signal producer thread đang waiting trên condition notFull
            notFull.signal();
            return item;
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        SimpleBlockingQueue<Integer> blockingQueue = new SimpleBlockingQueue<>(5);

        // Producer thread
        Thread producer = new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    blockingQueue.put(i);
                    System.out.println("produce: " + i);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Producer");

        // Consumer thread
        Thread consumer = new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    int item = blockingQueue.take();
                    System.out.println("consume: " + item);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Consumer");

        producer.start();
        consumer.start();
    }
}
```

Trong ví dụ trên, `notFull` và `notEmpty` là hai instance `Condition` độc lập, lần lượt duy trì Condition queue riêng. Producer waiting trên `notFull` khi queue đầy, consumer waiting trên `notEmpty` khi queue rỗng. Thiết kế tách biệt các điều kiện chờ này tránh wake-up thread không cần thiết và hiệu quả hơn `synchronized` + `wait/notifyAll`.

#### Phân tích source code cốt lõi của `await()`

```java
// Inner class ConditionObject của AQS
public final void await() throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    // 1. Đóng gói thread hiện tại thành Node và thêm vào Condition queue
    Node node = addConditionWaiter();
    // 2. Release hoàn toàn lock và lưu giá trị state trước khi release
    int savedState = fullyRelease(node);
    int interruptMode = 0;
    // 3. Nếu node chưa nằm trong synchronization queue thì block thread hiện tại
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    // 4. Sau khi wake-up, vào lại synchronization queue để cạnh tranh lock
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null)
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}
```

Method `await()` có hai thao tác cốt lõi:

- `fullyRelease(node)`: release hoàn toàn lock (không chỉ release một lần), để dù thread reentrant lock nhiều lần, các thread khác vẫn có thể acquire lock trong thời gian thread waiting. Sau khi wake-up, khôi phục số lần reentrant trước đó thông qua `acquireQueued(node, savedState)`.
- `isOnSyncQueue(node)`: kiểm tra node đã được chuyển sang synchronization queue chưa. Khi thread khác gọi `signal()`, node được chuyển từ Condition queue sang synchronization queue; lúc đó `isOnSyncQueue()` trả về `true`, thread thoát vòng lặp `while` và bắt đầu cạnh tranh lock.

### Phân tích khác biệt performance giữa fair lock và non-fair lock

Trong phần phân tích source code trước đó, triển khai `tryAcquire()` được giải thích bằng non-fair lock của `ReentrantLock`. Thực tế `ReentrantLock` hỗ trợ đồng thời fair lock và non-fair lock. Phần này phân tích sâu hơn sự khác nhau trong triển khai và ảnh hưởng đến performance của hai loại.

#### Khác biệt ở tầng source code

`ReentrantLock` mặc định dùng non-fair lock, có thể chuyển sang fair lock thông qua constructor parameter:

```java
// Non-fair lock (mặc định)
ReentrantLock unfairLock = new ReentrantLock();
// Fair lock
ReentrantLock fairLock = new ReentrantLock(true);
```

Khác biệt cốt lõi giữa hai loại nằm ở triển khai method `tryAcquire()`. `nonfairTryAcquire()` của non-fair lock đã được phân tích ở trên, tiếp theo xem triển khai của fair lock:

```java
// ReentrantLock.FairSync
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        // Khác biệt cốt lõi: trước tiên gọi hasQueuedPredecessors() để kiểm tra queue có thread waiting lâu hơn không
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
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
    return false;
}
```

**Khác biệt duy nhất** là fair lock thêm một lần kiểm tra `hasQueuedPredecessors()` trước khi CAS sửa `state`:

```java
// AQS
public final boolean hasQueuedPredecessors() {
    Node t = tail;
    Node h = head;
    Node s;
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

Method này dùng để kiểm tra trước thread hiện tại có thread khác đang xếp hàng hay không. Nếu có, thread hiện tại không thể trực tiếp acquire lock mà phải xếp hàng waiting, từ đó bảo đảm tính fairness **FIFO**.

Non-fair lock không có kiểm tra này. Khi lock vừa được release, thread mới đến có thể trực tiếp dùng CAS để “giành” lock, ngay cả khi synchronization queue đã có thread khác đang waiting.

#### So sánh khác biệt performance

| Tiêu chí               | Non-fair lock (mặc định)                                                                                                            | Fair lock                                                                             |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| **Throughput**         | Cao hơn. Thread mới có cơ hội trực tiếp acquire lock, giảm context switch của thread                                                | Thấp hơn. Mọi thread đều phải xếp hàng, tăng overhead context switch                  |
| **Thread starvation**  | Có thể xảy ra. Trong tình huống cực đoan, một số thread không thể acquire lock trong thời gian dài                                  | Thường khó xảy ra hơn, nhưng fair lock không bảo đảm OS thread scheduling             |
| **Context switch**     | Ít hơn. Sau khi thread hold lock release lock, thread mới đến có thể trực tiếp acquire lock mà không cần wake-up thread trong queue | Nhiều hơn. Mỗi lần release lock đều cần wake-up thread tiếp theo trong queue          |
| **Tình huống phù hợp** | Phần lớn tình huống (yêu cầu cao về response time và throughput)                                                                    | Tình huống yêu cầu nghiêm ngặt về fairness (như resource allocation, task scheduling) |

**Vì sao performance của non-fair lock thường tốt hơn?**

Nguyên nhân cốt lõi là **giảm số lần context switch của thread**. Sau khi thread A đang hold lock release lock:

- **Non-fair lock**: nếu vừa lúc thread B đang thử acquire lock (chưa vào synchronization queue), thread B có thể trực tiếp dùng CAS acquire lock và thực thi ngay, bỏ qua overhead wake-up thread trong queue. Thread đang waiting trong queue sau khi được wake-up phát hiện lock đã bị chiếm sẽ block lại; dù nhìn như “lãng phí” một lần wake-up, tổng thể vẫn giảm số lần context switch.
- **Fair lock**: thread B phải xếp ở cuối queue, sau đó wake-up thread ở đầu queue. Từ lúc thread được wake-up đến lúc thực sự bắt đầu execute tồn tại một khoảng **scheduling latency** (thread chuyển từ blocking sang running); trong khoảng latency đó lock ở trạng thái idle, làm giảm hiệu suất sử dụng lock.

Doug Lea chỉ ra trong document của `ReentrantLock` rằng: trong môi trường multi-thread, throughput tổng thể của chương trình dùng fair lock thường thấp hơn chương trình dùng non-fair lock (tức chậm hơn), vì vậy `ReentrantLock` mặc định dùng non-fair mode. Tuy nhiên trong tình huống cần bảo đảm thứ tự xử lý request hoặc tránh thread starvation (như phân phối connection pool), fair lock là lựa chọn tốt hơn.

Ví dụ code dưới đây minh họa sự khác nhau về behavior giữa fair lock và non-fair lock:

```java
import java.util.concurrent.locks.ReentrantLock;

public class FairVsUnfairLockDemo {
    // Lần lượt test fair lock và non-fair lock
    private static void testLock(ReentrantLock lock, String lockType) {
        System.out.println("=== " + lockType + " ===");
        Runnable task = () -> {
            for (int i = 0; i < 2; i++) {
                lock.lock();
                try {
                    System.out.println(Thread.currentThread().getName() + " đã acquire lock");
                } finally {
                    lock.unlock();
                }
            }
        };

        Thread[] threads = new Thread[5];
        for (int i = 0; i < 5; i++) {
            threads[i] = new Thread(task, lockType + "-thread-" + i);
        }
        for (Thread t : threads) {
            t.start();
        }
        for (Thread t : threads) {
            try { t.join(); } catch (InterruptedException e) { }
        }
        System.out.println();
    }

    public static void main(String[] args) {
        // Non-fair lock: cùng một thread có thể acquire lock liên tiếp nhiều lần
        testLock(new ReentrantLock(false), "non-fair lock");

        // Fair lock: khi có cạnh tranh, ưu tiên thread đã waiting lâu hơn acquire lock trước
        testLock(new ReentrantLock(true), "fair lock");
    }
}
```

Chạy code trên thường có thể quan sát thấy: ở non-fair lock mode, cùng một thread dễ acquire lock liên tiếp nhiều lần hơn (vì sau khi release lock nó lập tức cạnh tranh lại, có cơ hội giành lock trước khi thread trong queue được wake-up); khi có thread waiting, fair lock có xu hướng phân phối lock theo thứ tự queue. Tuy nhiên fairness không đồng nghĩa OS scheduling công bằng; nếu thread khác chưa chạy đến điểm waiting, cùng một thread vẫn có thể liên tiếp acquire lock.

## Các synchronizer thường gặp

### Semaphore (semaphore)

#### Giới thiệu

`synchronized` và `ReentrantLock` mỗi lần chỉ cho phép một thread truy cập resource, còn `Semaphore` (semaphore) dùng để kiểm soát số thread đồng thời truy cập resource cụ thể.

Cách dùng `Semaphore` đơn giản. Giả sử có `N(N>5)` thread acquire shared resource trong `Semaphore`; code dưới đây biểu thị trong cùng một thời điểm chỉ 5 trong N thread acquire được shared resource, các thread khác đều block và chỉ thread acquire được shared resource mới có thể thực thi. Khi có thread release shared resource, thread khác đang block mới có thể acquire.

```java
// Số lượng shared resource ban đầu
final Semaphore semaphore = new Semaphore(5);
// Acquire 1 permit
semaphore.acquire();
// Release 1 permit
semaphore.release();
```

Khi số resource ban đầu là 1, `Semaphore` suy biến thành exclusive lock.

`Semaphore` có hai mode:

- **Fair mode:** khi có cạnh tranh, method blocking `acquire` sẽ chọn thread theo FIFO tại điểm enqueue nội bộ; điều này không đồng nghĩa sắp xếp nghiêm ngặt theo thời gian trên wall clock lúc method được gọi. Ngoài ra, `tryAcquire()` không có argument không tuân theo thiết lập fairness và vẫn có thể chen hàng thành công;
- **Non-fair mode:** mode có tính preemptive.

Hai constructor tương ứng của `Semaphore` như sau:

```java
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}

public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
```

**Cả hai constructor đều phải nhận số lượng permit. Constructor thứ hai có thể chỉ định fair mode hoặc non-fair mode; mặc định là non-fair mode.**

`Semaphore` thường dùng trong tình huống resource có giới hạn rõ ràng về số lượt truy cập, ví dụ rate limiting (chỉ áp dụng cho single-machine mode; trong project thực tế nên dùng Redis + Lua để rate limiting).

#### Nguyên lý

`Semaphore` là một triển khai của shared lock. Nó mặc định khởi tạo giá trị `state` của AQS bằng `permits`; có thể hiểu `permits` là số lượng permit, chỉ thread lấy được permit mới có thể thực thi.

Lấy method `acquire` không argument làm ví dụ: khi gọi `semaphore.acquire()`, thread thử acquire permit. Nếu `state > 0`, biểu thị có thể acquire thành công; nếu `state <= 0`, biểu thị số permit không đủ và acquire thất bại.

Nếu có thể acquire thành công (`state > 0`), thread thử dùng CAS để sửa giá trị `state` thành `state=state-1`. Nếu acquire thất bại, nó tạo một Node thêm vào waiting queue và suspend thread hiện tại.

```java
// Acquire 1 permit
public void acquire() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}

// Acquire một hoặc nhiều permit
public void acquire(int permits) throws InterruptedException {
    if (permits < 0) throw new IllegalArgumentException();
    sync.acquireSharedInterruptibly(permits);
}
```

Method `acquireSharedInterruptibly` là default implementation trong `AbstractQueuedSynchronizer`.

```java
// Acquire permit ở shared mode; thành công thì return, thất bại thì thêm vào waiting queue và suspend thread
public final void acquireSharedInterruptibly(int arg)
    throws InterruptedException {
    if (Thread.interrupted())
      throw new InterruptedException();
        // Thử acquire permit; arg là số permit cần acquire. Khi acquire thất bại, tạo node thêm vào waiting queue và suspend thread.
    if (tryAcquireShared(arg) < 0)
      doAcquireSharedInterruptibly(arg);
}
```

Tiếp theo lấy non-fair mode (`NonfairSync`) làm ví dụ để xem triển khai method `tryAcquireShared`.

```java
// Thử acquire resource ở shared mode (resource trong Semaphore là permit):
protected int tryAcquireShared(int acquires) {
    return nonfairTryAcquireShared(acquires);
}

// Acquire permit ở shared mode non-fair
final int nonfairTryAcquireShared(int acquires) {
    for (;;) {
        // Số permit hiện có
        int available = getState();
        /*
         * Thử acquire permit. Khi số permit hiện có nhỏ hơn hoặc bằng 0, trả về số âm, biểu thị acquire thất bại.
         * Chỉ khi số permit hiện có lớn hơn 0 mới có thể acquire thành công; nếu CAS thất bại, loop để lấy giá trị mới nhất và thử acquire.
         */
        int remaining = available - acquires;
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```

Lấy method `release` không argument làm ví dụ: khi gọi `semaphore.release()`, thread thử release permit và dùng CAS sửa giá trị `state` thành `state=state+1`. Sau khi release permit thành công, đồng thời wake-up một thread trong waiting queue. Thread được wake-up sẽ thử sửa lại `state` thành `state=state-1`; nếu `state > 0` thì acquire permit thành công, nếu không thì vào lại waiting queue và block.

```java
// Release 1 permit
public void release() {
    sync.releaseShared(1);
}

// Release một hoặc nhiều permit
public void release(int permits) {
    if (permits < 0) throw new IllegalArgumentException();
    sync.releaseShared(permits);
}
```

Method `releaseShared` là default implementation trong `AbstractQueuedSynchronizer`.

```java
// Release shared lock
// Nếu tryReleaseShared trả về true thì wake-up một hoặc nhiều thread trong waiting queue.
public final boolean releaseShared(int arg) {
    // Release shared lock
    if (tryReleaseShared(arg)) {
      // Release waiting successor của node hiện tại
      doReleaseShared();
      return true;
    }
    return false;
}
```

Method `tryReleaseShared` là method được inner class `Sync` của `Semaphore` override; default implementation trong `AbstractQueuedSynchronizer` chỉ throw exception `UnsupportedOperationException`.

```java
// Một method được override trong inner class Sync
// Thử release resource
protected final boolean tryReleaseShared(int releases) {
    for (;;) {
        int current = getState();
        // Tăng permit khả dụng lên 1
        int next = current + releases;
        if (next < current) // overflow
            throw new Error("Maximum permit count exceeded");
         // CAS sửa giá trị state
        if (compareAndSetState(current, next))
            return true;
    }
}
```

Có thể thấy các method đã đề cập ở trên về cơ bản đều được triển khai thông qua synchronizer `sync`. `Sync` là inner class của `Semaphore`, kế thừa `AbstractQueuedSynchronizer` và override một số method trong đó. Ngoài ra, `Sync` còn có hai subclass là `NonfairSync` (tương ứng non-fair mode) và `FairSync` (tương ứng fair mode).

```java
private static final class Sync extends AbstractQueuedSynchronizer {
  // ...
}
static final class NonfairSync extends Sync {
  // ...
}
static final class FairSync extends Sync {
  // ...
}
```

#### Thực chiến

```java
public class SemaphoreExample {
  // Số request
  private static final int threadCount = 550;

  public static void main(String[] args) throws InterruptedException {
    // Tạo thread pool có số thread cố định (nếu số thread của thread pool ở đây quá ít, bạn sẽ thấy thực thi rất chậm)
    ExecutorService threadPool = Executors.newFixedThreadPool(300);
    // Số permit ban đầu
    final Semaphore semaphore = new Semaphore(20);

    for (int i = 0; i < threadCount; i++) {
      final int threadnum = i;
      threadPool.execute(() -> {// Cách sử dụng Lambda expression
        try {
          semaphore.acquire();// Acquire một permit, nên số thread có thể chạy là 20/1=20
          test(threadnum);
          semaphore.release();// Release một permit
        } catch (InterruptedException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        }

      });
    }
    threadPool.shutdown();
    System.out.println("finish");
  }

  public static void test(int threadnum) throws InterruptedException {
    Thread.sleep(1000);// Mô phỏng thao tác xử lý request tốn thời gian
    System.out.println("threadnum:" + threadnum);
    Thread.sleep(1000);// Mô phỏng thao tác xử lý request tốn thời gian
  }
}
```

Method `acquire()` sẽ block cho đến khi có permit để lấy rồi lấy đi một permit; mỗi method `release` tăng một permit, thao tác này có thể đánh thức một lần gọi `acquire()` đang block. Tuy nhiên thực tế không có object permit cụ thể; `Semaphore` chỉ duy trì số lượng permit có thể lấy. `Semaphore` thường dùng để giới hạn số thread acquire một loại resource nào đó.

Đương nhiên cũng có thể lấy và release nhiều permit cùng lúc, nhưng thường không cần làm vậy:

```java
semaphore.acquire(5);// Acquire 5 permit, nên số thread có thể chạy là 20/5=4
test(threadnum);
semaphore.release(5);// Release 5 permit
```

Ngoài method `acquire()`, một method tương ứng khác cũng thường được dùng là `tryAcquire()`. Method này trả về false ngay nếu không acquire được permit.

[Nội dung bổ sung từ issue645](https://github.com/Snailclimb/JavaGuide/issues/645):

> `Semaphore` được triển khai dựa trên AQS, dùng để kiểm soát số thread truy cập đồng thời, nhưng khái niệm này khác với shared lock. Constructor của `Semaphore` dùng parameter `permits` để initialize variable `state` của AQS; variable này biểu thị số permit khả dụng. Khi thread gọi method `acquire()` để thử acquire permit, `state` sẽ nguyên tử giảm 1. Nếu sau khi giảm 1, `state` lớn hơn hoặc bằng 0, `acquire()` return thành công và thread có thể tiếp tục thực thi. Nếu sau khi giảm 1, `state` nhỏ hơn 0, biểu thị số thread truy cập đồng thời đã đạt giới hạn `permits`; thread đó sẽ được đưa vào waiting queue của AQS và bị block, **không phải spin waiting**. Khi thread khác hoàn thành task và gọi method `release()`, `state` sẽ nguyên tử tăng 1. Thao tác `release()` sẽ wake-up một hoặc nhiều thread đang block trong waiting queue của AQS. Các thread được wake-up sẽ lại thử thao tác `acquire()` để cạnh tranh permit khả dụng. Vì vậy, `Semaphore` giới hạn số thread truy cập đồng thời bằng cách kiểm soát số permit, chứ không phải bằng spin và cơ chế shared lock.

### CountDownLatch (countdown latch)

#### Giới thiệu

`CountDownLatch` cho phép `count` thread block tại một điểm cho đến khi đủ số lần `countDown()` được gọi.

`CountDownLatch` chỉ dùng một lần. Giá trị counter chỉ có thể khởi tạo một lần trong constructor và sau đó không có cơ chế nào set lại; sau khi dùng xong `CountDownLatch`, không thể sử dụng lại.

#### Nguyên lý

`CountDownLatch` là một triển khai của shared lock. Nó mặc định khởi tạo giá trị `state` của AQS bằng `count`, có thể thấy điều này qua constructor của `CountDownLatch`.

```java
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}

private static final class Sync extends AbstractQueuedSynchronizer {
    Sync(int count) {
        setState(count);
    }
  //...
}
```

Khi thread gọi `countDown()`, thực tế method `tryReleaseShared` được dùng để giảm `state` bằng thao tác CAS cho đến khi `state` bằng 0. Khi `state` bằng 0, biểu thị đã đủ số lần gọi `countDown()`, các thread waiting trên `CountDownLatch` sẽ được wake-up và tiếp tục thực thi.

```java
public void countDown() {
    // Sync là inner class của CountDownLatch, kế thừa AbstractQueuedSynchronizer
    sync.releaseShared(1);
}
```

Method `releaseShared` là default implementation trong `AbstractQueuedSynchronizer`.

```java
// Release shared lock
// Nếu tryReleaseShared trả về true thì wake-up một hoặc nhiều thread trong waiting queue.
public final boolean releaseShared(int arg) {
    // Release shared lock
    if (tryReleaseShared(arg)) {
      // Release waiting successor của node hiện tại
      doReleaseShared();
      return true;
    }
    return false;
}
```

Method `tryReleaseShared` là method được inner class `Sync` của `CountDownLatch` override; default implementation trong `AbstractQueuedSynchronizer` chỉ throw exception `UnsupportedOperationException`.

```java
// Decrement state cho đến khi state thành 0;
// Chỉ khi count giảm đến 0 thì countDown mới return true
protected boolean tryReleaseShared(int releases) {
    // Kiểm tra state bằng spin
    for (;;) {
        int c = getState();
        // Nếu state đã là 0 thì trực tiếp return false
        if (c == 0)
            return false;
        // Decrement state
        int nextc = c-1;
        // CAS update giá trị state
        if (compareAndSetState(c, nextc))
            return nextc == 0;
    }
}
```

Lấy method `await` không argument làm ví dụ. Khi gọi `await()`, nếu `state` khác 0 thì chứng tỏ task chưa hoàn thành, `await()` sẽ block, tức các câu lệnh sau `await()` chưa được thực thi (main thread được thêm vào waiting queue, cũng chính là CLH variant queue). Sau đó AQS tiếp tục kiểm tra `state == 0`; nếu `state == 0`, mọi thread waiting sẽ được release và các câu lệnh sau `await()` được thực thi.

```java
// Waiting (cũng có thể gọi là acquire lock)
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
// Waiting có timeout
public boolean await(long timeout, TimeUnit unit)
    throws InterruptedException {
    return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
}
```

Method `acquireSharedInterruptibly` là default implementation trong `AbstractQueuedSynchronizer`.

```java
// Thử acquire lock; thành công thì return, thất bại thì thêm vào waiting queue và suspend thread
public final void acquireSharedInterruptibly(int arg)
    throws InterruptedException {
    if (Thread.interrupted())
      throw new InterruptedException();
        // Thử acquire lock; thành công thì return
    if (tryAcquireShared(arg) < 0)
      // Acquire thất bại, thêm vào waiting queue và suspend thread
      doAcquireSharedInterruptibly(arg);
}
```

Method `tryAcquireShared` là method được inner class `Sync` của `CountDownLatch` override. Tác dụng của nó là kiểm tra giá trị `state` có bằng 0 không; nếu có thì return 1, nếu không thì return -1.

```java
protected int tryAcquireShared(int acquires) {
    return (getState() == 0) ? 1 : -1;
}
```

#### Thực chiến

**Hai cách dùng điển hình của CountDownLatch:**

1. Một thread waiting trước khi bắt đầu chạy cho đến khi n thread chạy xong: initialize counter của `CountDownLatch` bằng n (`new CountDownLatch(n)`), mỗi task thread sau khi thực thi xong giảm counter 1 (`countdownlatch.countDown()`); khi counter trở thành 0, thread đang `await()` trên `CountDownLatch` sẽ được wake-up. Một tình huống ứng dụng điển hình là khi start một service, main thread cần chờ nhiều component load xong rồi mới tiếp tục thực thi.
2. Triển khai tính parallelism tối đa khi nhiều thread bắt đầu thực thi task: lưu ý đây là parallelism, không phải concurrency, nhấn mạnh nhiều thread đồng thời bắt đầu thực thi tại một thời điểm. Tương tự một cuộc đua, đặt nhiều thread tại vạch xuất phát, chờ hiệu lệnh rồi đồng thời chạy. Cách làm là initialize một object `CountDownLatch` shared, initialize counter bằng 1 (`new CountDownLatch(1)`), nhiều thread trước khi bắt đầu task sẽ gọi `coundownlatch.await()`; khi main thread gọi `countDown()`, counter thành 0 và nhiều thread đồng thời được wake-up.

**Ví dụ code CountDownLatch:**

```java
public class CountDownLatchExample {
  // Số request
  private static final int THREAD_COUNT = 550;

  public static void main(String[] args) throws InterruptedException {
    // Tạo thread pool có số thread cố định (nếu số thread của thread pool ở đây quá ít, bạn sẽ thấy thực thi rất chậm)
    // Chỉ dùng để test, trong tình huống thực tế hãy tự gán parameter cho thread pool
    ExecutorService threadPool = Executors.newFixedThreadPool(300);
    final CountDownLatch countDownLatch = new CountDownLatch(THREAD_COUNT);
    for (int i = 0; i < THREAD_COUNT; i++) {
      final int threadNum = i;
      threadPool.execute(() -> {
        try {
          test(threadNum);
        } catch (InterruptedException e) {
          e.printStackTrace();
        } finally {
          // Biểu thị một request đã hoàn thành
          countDownLatch.countDown();
        }

      });
    }
    countDownLatch.await();
    threadPool.shutdown();
    System.out.println("finish");
  }

  public static void test(int threadnum) throws InterruptedException {
    Thread.sleep(1000);
    System.out.println("threadNum:" + threadnum);
    Thread.sleep(1000);
  }
}
```

Trong code trên, số request được định nghĩa là 550. Chỉ sau khi 550 request này được xử lý xong, `System.out.println("finish");` mới được thực thi.

Lần tương tác đầu tiên với `CountDownLatch` là main thread waiting các thread khác. Main thread phải gọi method `CountDownLatch.await()` ngay sau khi start các thread khác. Khi đó thao tác của main thread sẽ block tại method này cho đến khi các thread khác hoàn thành task tương ứng.

N thread khác phải reference object latch, vì chúng cần thông báo cho object `CountDownLatch` rằng task tương ứng đã hoàn thành. Cơ chế thông báo này được thực hiện qua method `CountDownLatch.countDown()`; mỗi lần gọi method này, giá trị count initialize trong constructor giảm 1. Vì vậy sau khi đủ N lần gọi method này, giá trị count bằng 0, main thread có thể tiếp tục thực thi task của mình thông qua method `await()`.

Nói thêm: dùng không đúng method `await()` của `CountDownLatch` rất dễ gây deadlock. Ví dụ nếu đổi vòng lặp `for` trong code trên thành:

```java
for (int i = 0; i < threadCount-1; i++) {
.......
}
```

Khi đó giá trị `count` không thể bằng 0, dẫn đến waiting liên tục.

### CyclicBarrier (cyclic barrier)

#### Giới thiệu

`CyclicBarrier` rất giống `CountDownLatch`. Nó cũng hỗ trợ cơ chế chờ đồng bộ giữa các thread, nhưng chức năng phức tạp và mạnh hơn `CountDownLatch`. Tình huống ứng dụng chủ yếu tương tự `CountDownLatch`.

> `CountDownLatch` được triển khai dựa trên AQS, còn `CyclicBarrier` dựa trên `ReentrantLock` (`ReentrantLock` cũng là AQS synchronizer) và `Condition`.

Nghĩa đen của `CyclicBarrier` là barrier (Barrier) có thể sử dụng lại (Cyclic). Việc nó cần làm là: khi một nhóm thread đến barrier (cũng có thể gọi là synchronization point), các thread bị block cho đến khi thread cuối cùng đến barrier; lúc đó barrier mở và mọi thread bị barrier chặn mới tiếp tục thực thi.

#### Nguyên lý

Bên trong `CyclicBarrier` dùng variable `count` làm counter. Giá trị ban đầu của `count` là giá trị initialize của property `parties`. Mỗi khi một thread đến barrier, counter giảm 1. Khi count bằng 0, biểu thị đây là thread cuối cùng của generation đến barrier, nó sẽ thử thực thi task được truyền vào constructor.

```java
// Số thread bị chặn mỗi lần
private final int parties;
// Counter
private int count;
```

Tiếp theo xem sơ lược qua source code.

1. Constructor mặc định của `CyclicBarrier` là `CyclicBarrier(int parties)`. Parameter biểu thị số thread bị barrier chặn. Mỗi thread gọi method `await()` để thông báo với `CyclicBarrier` rằng mình đã đến barrier, sau đó thread hiện tại bị block.

```java
public CyclicBarrier(int parties) {
    this(parties, null);
}

public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}
```

Trong đó `parties` biểu thị số thread bị barrier chặn. Khi số thread bị chặn đạt giá trị này, barrier mở để mọi thread đi qua.

2. Khi object `CyclicBarrier` gọi method `await()`, thực tế nó gọi method `dowait(false, 0L)`. Method `await()` giống như dựng một barrier để chặn thread; chỉ khi số thread bị chặn đạt giá trị `parties`, barrier mới mở và thread mới có thể tiếp tục thực thi.

```java
public int await() throws InterruptedException, BrokenBarrierException {
  try {
      return dowait(false, 0L);
  } catch (TimeoutException toe) {
      throw new Error(toe); // cannot happen
  }
}
```

Phân tích source code method `dowait(false, 0L)` như sau:

```java
    // Chỉ sau khi số thread hoặc số request đạt count thì code sau await mới được thực thi. Trong ví dụ trên, giá trị count là 5.
    private int count;
    /**
     * Main barrier code, covering the various policies.
     */
    private int dowait(boolean timed, long nanos)
        throws InterruptedException, BrokenBarrierException,
               TimeoutException {
        final ReentrantLock lock = this.lock;
        // Lock
        lock.lock();
        try {
            final Generation g = generation;

            if (g.broken)
                throw new BrokenBarrierException();

            // Nếu thread bị interrupt thì throw exception
            if (Thread.interrupted()) {
                breakBarrier();
                throw new InterruptedException();
            }
            // Decrement count
            int index = --count;
            // Khi count giảm về 0, biểu thị thread cuối cùng đã đến barrier, đạt điều kiện để thực thi code sau method await
            if (index == 0) {  // tripped
                boolean ranAction = false;
                try {
                    final Runnable command = barrierCommand;
                    if (command != null)
                        command.run();
                    ranAction = true;
                    // Reset count về giá trị initialize của property parties
                    // Wake-up các thread waiting trước đó
                    // Bắt đầu lượt thực thi tiếp theo
                    nextGeneration();
                    return 0;
                } finally {
                    if (!ranAction)
                        breakBarrier();
                }
            }

            // loop until tripped, broken, interrupted, or timed out
            for (;;) {
                try {
                    if (!timed)
                        trip.await();
                    else if (nanos > 0L)
                        nanos = trip.awaitNanos(nanos);
                } catch (InterruptedException ie) {
                    if (g == generation && ! g.broken) {
                        breakBarrier();
                        throw ie;
                    } else {
                        // We're about to finish waiting even if we had not
                        // been interrupted, so this interrupt is deemed to
                        // "belong" to subsequent execution.
                        Thread.currentThread().interrupt();
                    }
                }

                if (g.broken)
                    throw new BrokenBarrierException();

                if (g != generation)
                    return index;

                if (timed && nanos <= 0L) {
                    breakBarrier();
                    throw new TimeoutException();
                }
            }
        } finally {
            lock.unlock();
        }
    }
```

#### Thực chiến

Ví dụ 1:

```java
public class CyclicBarrierExample1 {
  // Số request
  private static final int threadCount = 550;
  // Số thread cần synchronize
  private static final CyclicBarrier cyclicBarrier = new CyclicBarrier(5);

  public static void main(String[] args) throws InterruptedException {
    // Tạo thread pool
    ExecutorService threadPool = Executors.newFixedThreadPool(10);

    for (int i = 0; i < threadCount; i++) {
      final int threadNum = i;
      Thread.sleep(1000);
      threadPool.execute(() -> {
        try {
          test(threadNum);
        } catch (InterruptedException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        } catch (BrokenBarrierException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        }
      });
    }
    threadPool.shutdown();
  }

  public static void test(int threadnum) throws InterruptedException, BrokenBarrierException {
    System.out.println("threadnum:" + threadnum + "is ready");
    try {
       /** Chờ 60 giây, bảo đảm các thread con thực thi xong hoàn toàn */
      cyclicBarrier.await(60, TimeUnit.SECONDS);
    } catch (Exception e) {
      System.out.println("-----CyclicBarrierException------");
    }
    System.out.println("threadnum:" + threadnum + "is finish");
  }

}
```

Kết quả chạy như sau:

```plain
threadnum:0is ready
threadnum:1is ready
threadnum:2is ready
threadnum:3is ready
threadnum:4is ready
threadnum:4is finish
threadnum:0is finish
threadnum:1is finish
threadnum:2is finish
threadnum:3is finish
threadnum:5is ready
threadnum:6is ready
threadnum:7is ready
threadnum:8is ready
threadnum:9is ready
threadnum:9is finish
threadnum:5is finish
threadnum:8is finish
threadnum:7is finish
threadnum:6is finish
......
```

Có thể thấy chỉ sau khi số thread, cũng tức số request, đạt 5 như định nghĩa thì code sau method `await()` mới được thực thi.

Ngoài ra, `CyclicBarrier` còn cung cấp constructor nâng cao hơn `CyclicBarrier(int parties, Runnable barrierAction)`, dùng để ưu tiên thực thi `barrierAction` khi thread đến barrier, thuận tiện xử lý các tình huống business phức tạp hơn.

Ví dụ 2:

```java
public class CyclicBarrierExample2 {
  // Số request
  private static final int threadCount = 550;
  // Số thread cần synchronize
  private static final CyclicBarrier cyclicBarrier = new CyclicBarrier(5, () -> {
    System.out.println("------ưu tiên thực thi sau khi số thread đạt yêu cầu------");
  });

  public static void main(String[] args) throws InterruptedException {
    // Tạo thread pool
    ExecutorService threadPool = Executors.newFixedThreadPool(10);

    for (int i = 0; i < threadCount; i++) {
      final int threadNum = i;
      Thread.sleep(1000);
      threadPool.execute(() -> {
        try {
          test(threadNum);
        } catch (InterruptedException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        } catch (BrokenBarrierException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        }
      });
    }
    threadPool.shutdown();
  }

  public static void test(int threadnum) throws InterruptedException, BrokenBarrierException {
    System.out.println("threadnum:" + threadnum + "is ready");
    cyclicBarrier.await();
    System.out.println("threadnum:" + threadnum + "is finish");
  }

}
```

Kết quả chạy như sau:

```plain
threadnum:0is ready
threadnum:1is ready
threadnum:2is ready
threadnum:3is ready
threadnum:4is ready
------ưu tiên thực thi sau khi số thread đạt yêu cầu------
threadnum:4is finish
threadnum:0is finish
threadnum:2is finish
threadnum:1is finish
threadnum:3is finish
threadnum:5is ready
threadnum:6is ready
threadnum:7is ready
threadnum:8is ready
threadnum:9is ready
------ưu tiên thực thi sau khi số thread đạt yêu cầu------
threadnum:9is finish
threadnum:5is finish
threadnum:6is finish
threadnum:8is finish
threadnum:7is finish
......
```

## Tài liệu tham khảo

- Giải thích chi tiết AQS trong Java Concurrency: <https://www.cnblogs.com/waterystone/p/4920797.html>
- Xem nguyên lý và ứng dụng của AQS từ triển khai ReentrantLock: <https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html>

<!-- @include: @article-footer.snippet.md -->

```

```
