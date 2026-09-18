---
title: "Tổng hợp câu hỏi phỏng vấn về LRU cache: hash table, doubly linked list và LinkedHashMap"
description: "Tổng hợp câu hỏi phỏng vấn về LRU cache, giải thích chính sách eviction LRU, cách triển khai bằng hash table kết hợp doubly linked list, cách viết Java LinkedHashMap, độ phức tạp và các trường hợp sử dụng cache."
category: Computer Fundamentals
tag:
  - Data Structures
head:
  - - meta
    - name: keywords
      content: "LRU cache,LRU,cache eviction,hash table,doubly linked list,LinkedHashMap,Java LRU,page replacement,data structure interview questions"
---

LRU là viết tắt của Least Recently Used, nghĩa là ít được sử dụng gần đây nhất. Khi dung lượng cache đầy, dữ liệu lâu nhất chưa được truy cập sẽ bị loại bỏ.

Trong phỏng vấn, việc tự viết LRU xuất hiện rất thường xuyên vì nó kết hợp hash table và doubly linked list: hash table chịu trách nhiệm tìm node trong `O(1)`, còn doubly linked list chịu trách nhiệm di chuyển node và xóa node cuối trong `O(1)`.

Tổng quan nội dung bài viết:

1. LRU cache là gì?
2. Vì sao LRU phù hợp làm chính sách cache eviction?
3. Vì sao cần hash table + doubly linked list?
4. Tự viết `get` và `put` như thế nào?
5. Java `LinkedHashMap` triển khai LRU như thế nào?
6. LRU trong hệ thống thực tế còn phải cân nhắc điều gì?

![LRU cache duy trì thứ tự truy cập gần đây bằng hash table và doubly linked list](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/lru-cache.png)

## LRU cache là gì?

Mâu thuẫn cốt lõi của cache là: không gian có giới hạn, nhưng vẫn muốn giữ trong memory những dữ liệu “sẽ còn được truy cập trong tương lai” nhiều nhất có thể.

Vấn đề là chương trình không biết tương lai. LRU dùng “đã được truy cập gần đây” để xấp xỉ dự đoán “có thể sẽ được truy cập tiếp”. Nếu một dữ liệu vừa được truy cập, khả năng cao nó sẽ được truy cập lại; nếu một dữ liệu đã lâu không được truy cập, khi cache đầy thì nó sẽ được ưu tiên loại bỏ.

Ví dụ, một cache có dung lượng 2 được truy cập theo thứ tự:

```text
put(1, 1)
put(2, 2)
get(1)
put(3, 3)
```

Trước `put(3, 3)`, cache chứa `1` và `2`. Mặc dù `1` được thêm vào sớm hơn, nó vừa được `get(1)` truy cập, nên dữ liệu ít được sử dụng gần đây nhất là `2`; cuối cùng `2` sẽ bị loại bỏ.

Ví dụ này cũng cho thấy một điểm: LRU không xem “ai được thêm vào sớm nhất”, mà xem “ai đã lâu nhất không được truy cập”.

## Vì sao cần chính sách cache eviction?

Cache không có dung lượng vô hạn. Dù là cache trong memory cục bộ, Redis, database Buffer Pool hay page cache của operating system, tất cả đều phải đối mặt với giới hạn dung lượng.

Sau khi dung lượng đầy, nếu không có chính sách eviction thì chỉ có thể từ chối dữ liệu mới hoặc xóa dữ liệu ngẫu nhiên. Xóa ngẫu nhiên đương nhiên đơn giản, nhưng có thể xóa nhầm dữ liệu đang được truy cập thường xuyên, khiến hit rate giảm. LRU sử dụng thứ tự thời gian truy cập để đưa ra phán đoán theo kinh nghiệm: dữ liệu lâu không được chạm tới thường có xác suất được truy cập lại trong thời gian ngắn thấp hơn.

Có thể so sánh đơn giản các chính sách eviction thường gặp:

| Chính sách | Tiêu chí loại bỏ                    | Đặc điểm                                                                                             |
| ---------- | ----------------------------------- | ---------------------------------------------------------------------------------------------------- |
| FIFO       | Dữ liệu vào cache sớm nhất          | Dễ triển khai nhưng không quan tâm dữ liệu có được truy cập thường xuyên sau đó hay không            |
| LRU        | Dữ liệu lâu nhất chưa được truy cập | Phù hợp với access pattern có tính cục bộ theo thời gian                                             |
| LFU        | Dữ liệu có số lần truy cập ít nhất  | Phù hợp với trường hợp có hot data ổn định trong thời gian dài, nhưng cần duy trì thông tin tần suất |
| TTL        | Thời gian hết hạn                   | Phù hợp với dữ liệu có thời hạn rõ ràng, không tương đương với capacity eviction                     |

LRU thường được đưa vào phỏng vấn vì vừa có bối cảnh kỹ thuật thực tế, vừa kiểm tra tốt khả năng kết hợp các data structure.

## Trọng tâm phỏng vấn

- Giải thích rõ vì sao chỉ dùng hash table hoặc chỉ dùng linked list đều không đủ.
- Viết được `get` và `put`.
- Giải thích được head và tail của doubly linked list lần lượt đại diện cho điều gì.
- Xử lý được các trường hợp biên như dung lượng đầy, cập nhật key đã tồn tại và xóa node cuối.
- Biết Java `LinkedHashMap` có thể triển khai LRU.

## Duy trì thứ tự truy cập của LRU như thế nào?

Trong LRU cache, mỗi lần truy cập đều làm thay đổi mức độ mới của dữ liệu.

Thông thường, ta quy ước:

- Phần đầu linked list biểu thị dữ liệu được sử dụng gần đây nhất.
- Phần cuối linked list biểu thị dữ liệu lâu nhất chưa được sử dụng.
- Sau khi `get(key)` hit, di chuyển node tương ứng lên đầu.
- Nếu `put(key, value)` nhận một key mới, chèn node mới vào đầu.
- Nếu `put(key, value)` nhận một key đã tồn tại, sau khi cập nhật value cũng phải di chuyển node lên đầu.
- Khi dung lượng vượt giới hạn, xóa node ở cuối.

Điểm dễ bị bỏ sót nhất ở đây là `get()`. Nhiều bạn cho rằng `get()` chỉ đọc dữ liệu, không nên thay đổi structure. Nhưng với LRU, đọc cũng là một lần truy cập; chỉ cần hit cache thì “thời gian sử dụng gần đây nhất” của key đó đã trở nên mới hơn.

## Thiết kế data structure

| Thành phần               | Tác dụng                                                          |
| ------------------------ | ----------------------------------------------------------------- |
| `HashMap<Integer, Node>` | Nhanh chóng tìm node trong linked list theo key                   |
| Doubly linked list       | Sắp xếp theo thời gian truy cập, đầu là mới nhất, cuối là cũ nhất |
| Node head và tail giả    | Đơn giản hóa các trường hợp biên khi chèn và xóa                  |

Sau khi truy cập một key, cần di chuyển nó lên đầu linked list. Khi chèn key mới cũng đặt nó ở đầu. Khi vượt quá dung lượng, xóa node ngay trước tail.

Vì sao nhất thiết phải phối hợp hai structure?

Chỉ dùng hash table thì có thể tìm value trong `O(1)`, nhưng không biết key nào lâu nhất chưa được sử dụng. Vẫn phải duy trì thêm thứ tự truy cập.

Chỉ dùng linked list thì có thể duy trì thứ tự truy cập, node ở cuối chính là node cần loại bỏ. Nhưng mỗi lần tìm node theo key phải quét từ đầu đến cuối, độ phức tạp là `O(n)`.

Hash table + doubly linked list vừa đủ để bù đắp điểm yếu của nhau:

- Hash table giúp định vị trực tiếp node trong linked list theo key.
- Doubly linked list giúp di chuyển node và xóa node cuối nhanh chóng.
- Node đồng thời lưu key và value để khi loại bỏ node cuối, có thể xóa key tương ứng khỏi hash table.

## Các bước tự viết trong phỏng vấn

LRU có nhiều chi tiết trong code, nên không nên bắt đầu bằng việc viết cả class. Trong phỏng vấn, trước hết có thể tách các thao tác:

1. Xác định thứ tự linked list: đầu biểu thị dữ liệu được sử dụng gần đây nhất, cuối biểu thị dữ liệu lâu nhất chưa được sử dụng.
2. Định nghĩa `get`: không tìm thấy thì trả về `-1`, tìm thấy thì di chuyển lên đầu.
3. Định nghĩa `put`: key đã tồn tại thì cập nhật value và di chuyển lên đầu; key mới thì chèn vào đầu.
4. Cuối cùng xử lý eviction: sau khi vượt quá dung lượng, xóa node ngay trước tail và xóa nó khỏi hash table.
5. Đóng gói các thao tác của linked list thành `addToHead`, `remove`, `moveToHead`, `removeTail`.

Ưu điểm của cách viết này là `get` và `put` chỉ kết hợp một vài thao tác cơ bản của linked list, không lặp lại việc sửa pointer trong flow chính nên xác suất mắc lỗi thấp hơn nhiều.

## Vì sao dùng doubly linked list?

Để di chuyển một node lên đầu, trước hết cần tách nó khỏi vị trí hiện tại, sau đó chèn nó ngay sau head node.

Nếu dùng singly linked list, khi xóa node hiện tại phải biết node đứng trước nó. Ngay cả khi hash table có thể tìm trực tiếp node hiện tại, vẫn không tìm được node phía trước; cuối cùng vẫn phải duyệt từ đầu.

Node của doubly linked list đồng thời có `prev` và `next`. Để xóa một node bất kỳ, chỉ cần sửa bốn pointer:

```java
node.prev.next = node.next;
node.next.prev = node.prev;
```

Đây là lý do cốt lõi LRU cần doubly linked list: không chỉ phải xóa node cuối, mà khi `get()` hit và khi `put()` cập nhật key đã tồn tại, còn phải di chuyển node bất kỳ lên đầu.

Node head và tail giả cũng rất quan trọng. Với hai sentinel node `head` và `tail`, việc chèn đầu, xóa cuối và xử lý linked list rỗng đều có thể dùng cùng một bộ code, không cần viết các phán đoán `null` ở nhiều nơi.

## Tự viết LRU

```java
class LRUCache {
    private final int capacity;
    private final Map<Integer, Node> map = new HashMap<>();
    private final Node head = new Node(0, 0);
    private final Node tail = new Node(0, 0);

    LRUCache(int capacity) {
        this.capacity = capacity;
        head.next = tail;
        tail.prev = head;
    }

    int get(int key) {
        Node node = map.get(key);
        if (node == null) {
            return -1;
        }
        moveToHead(node);
        return node.value;
    }

    void put(int key, int value) {
        Node node = map.get(key);
        if (node != null) {
            node.value = value;
            moveToHead(node);
            return;
        }
        Node newNode = new Node(key, value);
        map.put(key, newNode);
        addToHead(newNode);
        if (map.size() > capacity) {
            Node removed = removeTail();
            map.remove(removed.key);
        }
    }

    private void moveToHead(Node node) {
        remove(node);
        addToHead(node);
    }

    private void addToHead(Node node) {
        node.prev = head;
        node.next = head.next;
        head.next.prev = node;
        head.next = node;
    }

    private void remove(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private Node removeTail() {
        Node node = tail.prev;
        remove(node);
        return node;
    }

    private static class Node {
        int key;
        int value;
        Node prev;
        Node next;

        Node(int key, int value) {
            this.key = key;
            this.value = value;
        }
    }
}
```

Độ phức tạp thời gian của `get` và `put` đều là `O(1)`, độ phức tạp không gian là `O(capacity)`.

## Minh họa quá trình thao tác

Giả sử dung lượng là 2, thực hiện theo thứ tự:

```text
put(1, 1)
put(2, 2)
get(1)
put(3, 3)
```

Trạng thái linked list thay đổi như sau, bên trái biểu thị dữ liệu được sử dụng gần đây nhất:

| Thao tác           | Trạng thái linked list | Giải thích                                 |
| ------------------ | ---------------------- | ------------------------------------------ |
| Trạng thái ban đầu | Rỗng                   | Head và tail giả nối với nhau, cache rỗng  |
| `put(1, 1)`        | `1`                    | Node mới được chèn vào đầu                 |
| `put(2, 2)`        | `2 -> 1`               | `2` là dữ liệu được sử dụng gần đây nhất   |
| `get(1)`           | `1 -> 2`               | Sau khi truy cập `1`, di chuyển nó lên đầu |
| `put(3, 3)`        | `3 -> 1`               | Vượt quá dung lượng, loại bỏ `2` ở cuối    |

Bảng này giúp kiểm tra hai điểm: truy cập node đã tồn tại phải cập nhật thứ tự sử dụng; khi loại bỏ node, phải xóa node ở cuối, tức node lâu nhất chưa được sử dụng, chứ không phải node mới được thêm vào.

## Triển khai bằng LinkedHashMap

Java `LinkedHashMap` hỗ trợ duy trì các phần tử theo thứ tự truy cập:

```java
class LRUCacheWithLinkedHashMap extends LinkedHashMap<Integer, Integer> {
    private final int capacity;

    LRUCacheWithLinkedHashMap(int capacity) {
        super(capacity, 0.75f, true);
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
        return size() > capacity;
    }
}
```

Tham số thứ ba `accessOrder` trong constructor được đặt thành `true`, biểu thị sắp xếp theo thứ tự truy cập thay vì thứ tự chèn.

Tài liệu chính thức của `LinkedHashMap` cũng đề cập riêng rằng chế độ access-order này phù hợp để xây dựng LRU cache. Có hai điểm cần nhớ:

1. Khi `accessOrder = false`, thứ tự duyệt là thứ tự chèn; khi `accessOrder = true`, thứ tự duyệt là thứ tự truy cập.
2. `removeEldestEntry()` được gọi sau khi chèn mapping mới. Khi trả về `true`, entry cũ nhất sẽ bị xóa.

Tuy nhiên, `LinkedHashMap` không thread-safe. Nếu muốn dùng trực tiếp nó làm local cache trong môi trường nhiều thread, cần tự thêm lock hoặc chọn một cache library trưởng thành.

## LRU có quan hệ gì với Redis?

Khi dùng Redis làm cache, sau khi memory đạt giới hạn `maxmemory`, cũng cần thực hiện chính sách eviction. Redis hỗ trợ các chính sách như `allkeys-lru`, `volatile-lru`:

- `allkeys-lru`: loại bỏ key ít được sử dụng gần đây nhất trong toàn bộ các key.
- `volatile-lru`: chỉ loại bỏ key ít được sử dụng gần đây nhất trong các key đã được đặt thời gian hết hạn.

Tuy nhiên, LRU của Redis không phải là “LRU chính xác” trong bài tự viết khi phỏng vấn. Để tiết kiệm memory và CPU, Redis sử dụng approximate LRU: lấy mẫu ngẫu nhiên một nhóm nhỏ key, rồi chọn key lâu nhất chưa được truy cập để loại bỏ. Có thể điều chỉnh số lượng mẫu bằng `maxmemory-samples`.

Điều này cũng giúp chúng ta hiểu sự khác biệt giữa hệ thống thực tế và bài phỏng vấn: bài phỏng vấn thường yêu cầu dùng hash table + doubly linked list để triển khai LRU chính xác; hệ thống thực tế sẽ cân bằng giữa memory, throughput, concurrency và hit rate.

## Trường hợp sử dụng trong hệ thống

- Cache eviction cục bộ.
- Page replacement của operating system.
- Cache dữ liệu hot.
- Cache kết quả có dung lượng nhỏ trong gateway, client SDK hoặc middleware.

Trong hệ thống thực tế còn phải cân nhắc thread safety, thời gian hết hạn, memory tối đa, metrics và callback eviction. LRU tự viết trong phỏng vấn chủ yếu kiểm tra khả năng kết hợp data structure, không cần đưa tất cả những yếu tố này vào code.

Nếu là local cache trong dự án Java, nhiều trường hợp sẽ không tự viết LRU mà dùng trực tiếp các cache library như Caffeine. Lý do cũng rất thực tế: cache trong hệ thống không chỉ cần eviction theo dung lượng mà còn phải xử lý expiration, concurrency, loading, metrics, async refresh và weight khác nhau của từng entry. Tài liệu chính thức của Caffeine chia eviction thành nhiều loại như theo size, theo time và theo reference.

## Câu hỏi phỏng vấn mở rộng

| Câu hỏi mở rộng                         | Trọng tâm trả lời                                                                                                               |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Vì sao không dùng riêng `HashMap`?      | `HashMap` có thể tìm value nhưng không biết key nào lâu nhất chưa được sử dụng                                                  |
| Vì sao không dùng riêng linked list?    | Linked list duy trì được thứ tự nhưng tìm node theo key cần `O(n)`                                                              |
| Vì sao phải dùng doubly linked list?    | Khi xóa node bất kỳ cần đồng thời nối node trước và sau; singly linked list không thể tìm node trước trong `O(1)`               |
| Vì sao phải dùng node head và tail giả? | Thống nhất logic chèn xóa của linked list rỗng, node đầu và node cuối, giảm các phán đoán rẽ nhánh                              |
| `LinkedHashMap` triển khai LRU thế nào? | Bật `accessOrder` khi khởi tạo, override `removeEldestEntry` để kiểm soát dung lượng                                            |
| Cache thực tế còn phải cân nhắc gì?     | Thread safety, thời gian hết hạn, memory tối đa, callback eviction, metrics hit rate và các vấn đề hệ thống như cache breakdown |

## Điểm dễ sai

- Khi cập nhật key đã tồn tại, cũng phải di chuyển nó lên đầu.
- Sau khi xóa node cuối, đừng quên xóa key khỏi hash table.
- Khi xóa node trong doubly linked list, phải đồng thời sửa cả hai pointer trước và sau.
- Node head và tail giả giúp giảm việc kiểm tra trường hợp biên của linked list rỗng.
- `accessOrder` của `LinkedHashMap` phải được đặt thành `true`.

## Tự kiểm tra bằng các câu hỏi thường gặp

- Vì sao LRU cần hash table và doubly linked list phối hợp?
- Vì sao thao tác `get` cũng phải di chuyển node?
- Khi cập nhật key đã tồn tại, vì sao không thể chỉ sửa value?
- Sau khi loại bỏ node cuối, vì sao còn phải xóa key khỏi hash table?
- Thứ tự chèn và thứ tự truy cập của `LinkedHashMap` khác nhau như thế nào?

## Bài tập đề xuất

- [146. LRU cache](https://leetcode.cn/problems/lru-cache/)
- [460. LFU cache](https://leetcode.cn/problems/lfu-cache/)

## Tài liệu tham khảo

- [Java SE 17 API: LinkedHashMap](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/LinkedHashMap.html)
- [Redis Docs: Key eviction](https://redis.io/docs/latest/develop/reference/eviction/)
- [Operating Systems: Three Easy Pieces](https://pages.cs.wisc.edu/~remzi/OSTEP/)
- [Caffeine Wiki: Eviction](https://github.com/ben-manes/caffeine/wiki/Eviction)

<!-- @include: @article-footer.snippet.md -->
