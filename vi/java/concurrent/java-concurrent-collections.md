---
title: Tổng hợp các concurrent container thường gặp trong Java
description: "Tổng hợp toàn diện về concurrent container trong Java: giải thích chi tiết đặc điểm, trường hợp sử dụng và so sánh performance của các container thread-safe thuộc JUC như ConcurrentHashMap/CopyOnWriteArrayList/BlockingQueue."
category: Java
tag:
  - Java Concurrency
head:
  - - meta
    - name: keywords
      content: Java concurrent container,ConcurrentHashMap,CopyOnWriteArrayList,BlockingQueue,ConcurrentLinkedQueue,thread-safe container
---

Phần lớn các container này do JDK cung cấp nằm trong package `java.util.concurrent`.

- **`ConcurrentHashMap`** : `HashMap` thread-safe
- **`CopyOnWriteArrayList`** : `List` thread-safe, performance rất tốt trong trường hợp đọc nhiều ghi ít, tốt hơn `Vector` rất nhiều.
- **`ConcurrentLinkedQueue`** : concurrent queue hiệu quả, được triển khai bằng linked list. Có thể xem đây là `LinkedList` thread-safe, đây là một non-blocking queue.
- **`BlockingQueue`** : Đây là một interface, bên trong JDK triển khai interface này bằng linked list, array và các cách khác. Đây là blocking queue, rất phù hợp để làm kênh chia sẻ dữ liệu.
- **`ConcurrentSkipListMap`** : Bản triển khai của skip list. Đây là một Map, sử dụng cấu trúc dữ liệu skip list để tìm kiếm nhanh.

## ConcurrentHashMap

Như đã biết, `HashMap` không thread-safe. Nếu sử dụng trong môi trường concurrent, một cách thường gặp là dùng phương thức `Collections.synchronizedMap()` để bọc `HashMap`, biến nó thành thread-safe. Tuy nhiên, cách này đồng bộ các truy cập concurrent giữa các thread bằng một global lock, gây ra bottleneck performance nghiêm trọng, đặc biệt trong môi trường high concurrency.

Để giải quyết vấn đề này, `ConcurrentHashMap` ra đời. Đây là phiên bản thread-safe của `HashMap`, cung cấp khả năng xử lý concurrent hiệu quả hơn.

Trong JDK1.7, `ConcurrentHashMap` chia bucket array thành các segment (`Segment`, segmented lock). Mỗi lock chỉ khóa một phần dữ liệu trong container (như hình minh họa bên dưới). Khi nhiều thread truy cập dữ liệu thuộc các segment khác nhau trong container, sẽ không xảy ra lock contention, nhờ đó nâng cao tỷ lệ truy cập concurrent.

![Cấu trúc lưu trữ ConcurrentHashMap trong Java 7](https://oss.javaguide.cn/github/javaguide/java/collection/java7_concurrenthashmap.png)

Đến JDK1.8, `ConcurrentHashMap` loại bỏ segmented lock `Segment`, sử dụng `Node + CAS + synchronized` để đảm bảo an toàn concurrent. Cấu trúc dữ liệu tương tự cấu trúc của `HashMap` 1.8: array + linked list/red-black tree. Trong Java 8, khi độ dài linked list vượt quá ngưỡng nhất định (8), linked list (độ phức tạp time complexity khi tìm kiếm là O(N)) được chuyển thành red-black tree (độ phức tạp time complexity khi tìm kiếm là O(log(N))).

Trong Java 8, lock của thao tác update có granularity nhỏ hơn: khi cần lock, `synchronized` khóa node đầu của bucket tương ứng; các thao tác update trên các bucket khác nhau thường có thể thực hiện concurrent. Thao tác read thường không cần lock và cũng có thể thực hiện concurrent với thao tác update.

![Cấu trúc lưu trữ ConcurrentHashMap trong Java 8](https://oss.javaguide.cn/github/javaguide/java/collection/java8_concurrenthashmap.png)

Để xem phần giới thiệu chi tiết về `ConcurrentHashMap`, hãy đọc bài viết: [Phân tích source code `ConcurrentHashMap`](./../collection/concurrent-hash-map-source-code.md).

## CopyOnWriteArrayList

Trước khi `CopyOnWriteArrayList` được đưa vào JDK 1.5, ngoài `Vector` ra đời khá sớm, cũng có thể bọc `List` thông thường bằng `Collections.synchronizedList()` để có khả năng truy cập đồng bộ. Các phương thức thêm, xóa, sửa, tìm kiếm của `Vector` về cơ bản đều có `synchronized`; một lần gọi phương thức đơn lẻ có tính thread-safe, nhưng các thao tác kết hợp vẫn cần đồng bộ bổ sung.

JDK1.5 đưa package `Java.util.concurrent` (JUC) vào, trong đó cung cấp nhiều container thread-safe có performance concurrent tốt. Bản triển khai `List` thread-safe duy nhất là `CopyOnWriteArrayList`.

Trong phần lớn business scenario, số lần read thường lớn hơn rất nhiều số lần write. Vì thao tác read không sửa dữ liệu hiện có, việc lock cho mỗi lần read thực chất là lãng phí resource. Ngược lại, nên cho phép nhiều thread cùng truy cập dữ liệu bên trong `List`, vì điều này an toàn đối với thao tác read.

Tư tưởng này rất tương tự thiết kế của read-write lock `ReentrantReadWriteLock`: read-read không mutual exclusion, read-write mutual exclusion, write-write mutual exclusion (chỉ read-read không mutual exclusion). `CopyOnWriteArrayList` hiện thực tư tưởng này ở mức cao hơn. Để tối ưu performance của thao tác read, các thao tác read trong `CopyOnWriteArrayList` hoàn toàn không cần lock. Đặc biệt, thao tác write cũng không block thao tác read, chỉ write-write mutual exclusion. Nhờ vậy, performance của thao tác read có thể được nâng cao đáng kể.

Điểm cốt lõi giúp `CopyOnWriteArrayList` thread-safe là nó sử dụng strategy **copy-on-write**, thể hiện ngay trong tên `CopyOnWriteArrayList`.

Khi cần sửa nội dung của `CopyOnWriteArrayList` (các thao tác như `add`, `set`, `remove`), không sửa trực tiếp array ban đầu mà trước tiên tạo một bản sao của array bên dưới, sửa trên array bản sao, sau khi sửa xong mới gán array đã sửa trở lại. Nhờ vậy, thao tác write không ảnh hưởng đến thao tác read.

Để xem phần giới thiệu chi tiết về `CopyOnWriteArrayList`, hãy đọc bài viết: [Phân tích source code `CopyOnWriteArrayList`](./../collection/copyonwritearraylist-source-code.md).

## ConcurrentLinkedQueue

`Queue` thread-safe do Java cung cấp có thể chia thành **blocking queue** và **non-blocking queue**. Ví dụ điển hình của blocking queue là `BlockingQueue`, còn ví dụ điển hình của non-blocking queue là `ConcurrentLinkedQueue`. Trong ứng dụng thực tế, cần chọn blocking queue hoặc non-blocking queue tùy nhu cầu. **Blocking queue có thể được triển khai bằng lock, còn non-blocking queue có thể được triển khai bằng thao tác CAS.**

Nhìn từ tên có thể thấy queue `ConcurrentLinkedQueue` sử dụng linked list làm cấu trúc dữ liệu. `ConcurrentLinkedQueue` có thể xem là queue có performance tốt nhất trong môi trường high concurrency. Sở dĩ nó có performance tốt là nhờ phần triển khai phức tạp bên trong.

Không phân tích source code bên trong `ConcurrentLinkedQueue` ở đây; chỉ cần biết `ConcurrentLinkedQueue` chủ yếu sử dụng CAS non-blocking algorithm để đảm bảo thread-safe.

`ConcurrentLinkedQueue` phù hợp với trường hợp yêu cầu performance tương đối cao, đồng thời có nhiều thread cùng read và write queue. Nói cách khác, nếu cost của việc lock queue cao thì phù hợp dùng `ConcurrentLinkedQueue` không lock để thay thế.

## BlockingQueue

### Giới thiệu về BlockingQueue

Ở trên đã đề cập `ConcurrentLinkedQueue` là non-blocking queue có performance cao. Tiếp theo là blocking queue `BlockingQueue`. Blocking queue (`BlockingQueue`) được sử dụng rộng rãi trong bài toán “producer-consumer”, vì `BlockingQueue` cung cấp các phương thức insert và remove có thể block. Khi queue container đầy, producer thread sẽ bị block cho đến khi queue không còn đầy; khi queue container rỗng, consumer thread sẽ bị block cho đến khi queue không còn rỗng.

`BlockingQueue` là một interface kế thừa từ `Queue`, vì vậy các class triển khai nó cũng có thể được sử dụng như bản triển khai của `Queue`; còn `Queue` lại kế thừa interface `Collection`. Dưới đây là các class triển khai liên quan của `BlockingQueue`:

![Các class triển khai của BlockingQueue](https://oss.javaguide.cn/github/javaguide/java/51622268.jpg)

Sau đây chủ yếu giới thiệu 3 class triển khai `BlockingQueue` thường gặp: `ArrayBlockingQueue`, `LinkedBlockingQueue`, `PriorityBlockingQueue`.

### ArrayBlockingQueue

`ArrayBlockingQueue` là class triển khai bounded queue của interface `BlockingQueue`, bên dưới sử dụng array.

```java
public class ArrayBlockingQueue<E>
extends AbstractQueue<E>
implements BlockingQueue<E>, Serializable{}
```

Sau khi được tạo, capacity của `ArrayBlockingQueue` không thể thay đổi. Cơ chế kiểm soát concurrent sử dụng reentrant lock `ReentrantLock`; cả thao tác insert và read đều phải lấy được lock mới có thể thực hiện. Khi queue đầy, việc thử đưa element vào queue sẽ khiến thao tác bị block; việc thử lấy một element từ queue rỗng cũng block tương tự.

Theo mặc định, `ArrayBlockingQueue` không đảm bảo tính fairness khi các thread đang chờ truy cập queue. Sau khi bật fair strategy, khi có contention, queue sẽ cấp cơ hội truy cập cho producer hoặc consumer đang chờ theo thứ tự FIFO. Điều này mô tả thứ tự chờ trong queue, không đồng nghĩa với việc OS strict schedule thread theo thời gian thực. Ở non-fair mode, thread bị block trong thời gian dài vẫn có thể không truy cập queue kịp thời. Fair strategy thường làm giảm throughput. Nếu cần, có thể dùng code sau:

```java
private static ArrayBlockingQueue<Integer> blockingQueue = new ArrayBlockingQueue<Integer>(10,true);
```

### LinkedBlockingQueue

`LinkedBlockingQueue` là blocking queue được triển khai bên dưới dựa trên **singly linked list**. Có thể dùng nó như unbounded queue hoặc bounded queue, đồng thời đáp ứng đặc tính FIFO. So với `ArrayBlockingQueue`, nó có throughput cao hơn. Để tránh capacity của `LinkedBlockingQueue` tăng nhanh và tiêu tốn nhiều memory, khi tạo object `LinkedBlockingQueue` thường nên chỉ định size. Nếu không chỉ định, capacity bằng `Integer.MAX_VALUE`.

**Các constructor liên quan:**

```java
    /**
     * Unbounded queue theo một nghĩa nào đó
     * Creates a {@code LinkedBlockingQueue} with a capacity of
     * {@link Integer#MAX_VALUE}.
     */
    public LinkedBlockingQueue() {
        this(Integer.MAX_VALUE);
    }

    /**
     * Bounded queue
     * Creates a {@code LinkedBlockingQueue} with the given (fixed) capacity.
     *
     * @param capacity the capacity of this queue
     * @throws IllegalArgumentException if {@code capacity} is not greater
     *         than zero
     */
    public LinkedBlockingQueue(int capacity) {
        if (capacity <= 0) throw new IllegalArgumentException();
        this.capacity = capacity;
        last = head = new Node<E>(null);
    }
```

### PriorityBlockingQueue

`PriorityBlockingQueue` là unbounded blocking queue có hỗ trợ priority. Theo mặc định, element được sort theo natural order. Ngoài ra, có thể chỉ định rule sort bằng cách tự triển khai phương thức `compareTo()` trong class, hoặc chỉ định `Comparator` thông qua constructor khi khởi tạo.

Cơ chế kiểm soát concurrent của `PriorityBlockingQueue` sử dụng reentrant lock `ReentrantLock`. Queue là unbounded queue (`ArrayBlockingQueue` là bounded queue; `LinkedBlockingQueue` cũng có thể chỉ định capacity tối đa của queue bằng cách truyền `capacity` vào constructor, nhưng `PriorityBlockingQueue` chỉ có thể chỉ định initial queue size; về sau khi insert element, **nếu không đủ space thì queue sẽ tự động mở rộng**).

Nói đơn giản, đây là phiên bản thread-safe của `PriorityQueue`. Không thể insert giá trị null; đồng thời object được insert vào queue phải có thể so sánh kích thước (comparable), nếu không sẽ báo exception `ClassCastException`. Thao tác insert `put` của nó không block vì đây là unbounded queue (thao tác `take` sẽ block khi queue rỗng).

**Bài viết đề xuất:** [Đọc hiểu Java concurrent queue BlockingQueue](https://javadoop.com/post/java-concurrent-queue)

## ConcurrentSkipListMap

> Phần nội dung dưới đây tham khảo chuyên mục [Vẻ đẹp của cấu trúc dữ liệu và thuật toán](https://time.geekbang.org/column/intro/126?code=zl3GYeAsRI4rEJIBNu5B/km7LSZsPDlGWQEpAYw5Vu0=&utm_term=SPoster "Vẻ đẹp của cấu trúc dữ liệu và thuật toán") của Geek Time, cùng sách _Thiết kế chương trình Java high concurrency trong thực tế_.

Để giới thiệu `ConcurrentSkipListMap`, trước tiên hãy cùng tìm hiểu đơn giản về skip list.

Với một singly linked list, ngay cả khi linked list đã được sort, nếu muốn tìm một dữ liệu trong đó thì vẫn chỉ có thể duyệt linked list từ đầu đến cuối, nên performance đương nhiên rất thấp. Skip list thì khác. Đây là một cấu trúc dữ liệu có thể dùng để tìm kiếm nhanh, hơi giống balanced tree. Cả hai đều có thể tìm kiếm element nhanh. Tuy nhiên, điểm khác biệt quan trọng là việc insert và delete trên balanced tree rất có thể khiến balanced tree phải điều chỉnh toàn cục một lần. Còn insert và delete trên skip list chỉ cần thao tác trên một phần của toàn bộ cấu trúc dữ liệu. Lợi ích của điều này là: trong high concurrency, cần global lock để đảm bảo thread-safe cho toàn bộ balanced tree; còn với skip list, chỉ cần partial lock. Nhờ vậy, trong môi trường high concurrency có thể đạt performance tốt hơn. Về performance query, time complexity của skip list cũng là **O(logn)**, vì vậy JDK sử dụng skip list để triển khai một Map trong concurrent data structure.

Bản chất của skip list là duy trì đồng thời nhiều linked list, và các linked list được phân tầng.

![Skip list với index cấp 2](https://oss.javaguide.cn/github/javaguide/java/93666217.jpg)

Linked list ở tầng thấp nhất duy trì toàn bộ element trong skip list; linked list ở mỗi tầng phía trên là một subset của tầng ngay bên dưới.

Các element trong mọi linked list của skip list đều được sort. Khi tìm kiếm, có thể bắt đầu từ linked list cấp cao nhất. Khi phát hiện element cần tìm nhỏ hơn successor của node đang truy cập (hoặc successor là null), quá trình sẽ chuyển xuống linked list ở tầng tiếp theo để tiếp tục tìm. Điều này có nghĩa là trong quá trình tìm kiếm, việc search được thực hiện theo kiểu nhảy qua các node. Như hình trên minh họa, hãy tìm element 18 trong skip list.

![Tìm element 18 trong skip list](https://oss.javaguide.cn/github/javaguide/java/32005738.jpg)

Khi tìm 18, trước đây cần duyệt 18 lần, còn hiện tại chỉ cần 7 lần. Khi độ dài linked list tương đối lớn, việc xây dựng index giúp nâng cao rõ rệt performance tìm kiếm.

Từ những điều trên có thể dễ dàng thấy rằng, **skip list là một algorithm dùng space để đổi lấy time.**

Một điểm khác giữa việc sử dụng skip list để triển khai `Map` và sử dụng hash algorithm để triển khai `Map` là: hash không lưu thứ tự của element, còn toàn bộ element trong skip list đều được sort. Vì vậy, khi traverse skip list, sẽ nhận được kết quả có thứ tự. Do đó, nếu ứng dụng cần ordering thì skip list là lựa chọn phù hợp nhất. Class triển khai cấu trúc dữ liệu này trong JDK là `ConcurrentSkipListMap`.

## Tham khảo

- _Thiết kế chương trình Java high concurrency trong thực tế_
- <https://javadoop.com/post/java-concurrent-queue>
- <https://juejin.im/post/5aeebd02518825672f19c546>

<!-- @include: @article-footer.snippet.md -->
