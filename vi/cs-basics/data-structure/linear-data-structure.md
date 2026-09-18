---
title: "Giải thích chi tiết về linear data structure (array, linked list, stack, queue)"
description: "Tổng hợp đặc tính và thao tác của array/linked list/stack/queue, kết hợp phân tích complexity và ứng dụng điển hình, nắm được cách lựa chọn và triển khai linear structure."
category: Computer Basics
tag:
  - Data Structure
head:
  - - meta
    - name: keywords
      content: array,linked list,stack,queue,deque,complexity analysis,random access,insertion and deletion
---

# Linear Data Structure

## 1. Array

**Array** là một data structure rất phổ biến. Nó gồm các element cùng kiểu và sử dụng một vùng memory liên tục để lưu trữ.

Ta có thể trực tiếp tính địa chỉ lưu trữ tương ứng của element thông qua index của element đó.

Đặc điểm của array là: **cung cấp random access** và có capacity giới hạn.

```java
Giả sử độ dài của array là n.
Truy cập: O(1) //truy cập element ở vị trí cụ thể
Chèn: O(n) //trường hợp xấu nhất xảy ra khi chèn ở đầu array và phải di chuyển toàn bộ element
Xóa: O(n) //trường hợp xấu nhất xảy ra khi xóa ở đầu array và phải di chuyển mọi element phía sau element đầu tiên
```

![Array](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/array.png)

## 2. Linked List

### 2.1. Giới thiệu về linked list

**Linked list** tuy là một linear list nhưng không lưu trữ data theo thứ tự tuyến tính, mà không sử dụng vùng memory liên tục để lưu trữ data.

Complexity của thao tác chèn và xóa trong linked list là O(1), chỉ cần biết element ngay trước vị trí đích. Tuy nhiên, khi tìm một node hoặc truy cập node ở vị trí cụ thể, complexity là O(n).

Sử dụng linked list có thể khắc phục nhược điểm phải biết trước kích thước data của array. Cấu trúc linked list có thể tận dụng đầy đủ memory của máy tính để thực hiện quản lý memory động một cách linh hoạt. Nhưng linked list không tiết kiệm space; so với array, nó chiếm nhiều space hơn vì mỗi node trong linked list còn lưu pointer trỏ tới node khác. Ngoài ra, linked list không có ưu điểm random access như array.

### 2.2. Phân loại linked list

**Các cách phân loại linked list phổ biến:**

1. Singly linked list
2. Doubly linked list
3. Circular linked list
4. Doubly circular linked list

```java
Giả sử linked list có n element.
Truy cập: O(n) //truy cập element ở vị trí cụ thể
Chèn và xóa: O(1) //phải biết vị trí chèn element
```

#### 2.2.1. Singly linked list

**Singly linked list** chỉ có một hướng, node chỉ có một successor pointer `next` trỏ tới node phía sau. Vì vậy, data structure linked list thường không liên tục trong physical memory. Theo thói quen, ta gọi node đầu tiên là head node; linked list thường có một node `head` không lưu bất kỳ value nào, thông qua head node ta có thể duyệt toàn bộ linked list. Tail node thường trỏ tới `null`.

![Singly linked list](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/single-linkedlist.png)

#### 2.2.2. Circular linked list

**Circular linked list** thực ra là một singly linked list đặc biệt. Khác với singly linked list, tail node của circular linked list không trỏ tới `null` mà trỏ tới head node của linked list.

![Circular linked list](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/circular-linkedlist.png)

#### 2.2.3. Doubly linked list

**Doubly linked list** có hai pointer: một `prev` trỏ tới node trước đó và một `next` trỏ tới node phía sau.

![Doubly linked list](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/bidirectional-linkedlist.png)

#### 2.2.4. Doubly circular linked list

**Doubly circular linked list** có `next` của node cuối cùng trỏ tới `head`, còn `prev` của `head` trỏ tới node cuối cùng, tạo thành một vòng.

![Doubly circular linked list](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/bidirectional-circular-linkedlist.png)

### 2.3. Trường hợp sử dụng

- Nếu cần hỗ trợ random access thì linked list không thể đáp ứng.
- Nếu số lượng data element cần lưu trữ không xác định và thường xuyên cần thêm, xóa data thì linked list phù hợp hơn.
- Nếu số lượng data element cần lưu trữ xác định và không thường xuyên thêm, xóa data thì array phù hợp hơn.

### 2.4. Array vs linked list

- Array hỗ trợ random access, còn linked list thì không.
- Array sử dụng vùng memory liên tục nên thân thiện với cơ chế cache của CPU, còn linked list thì ngược lại.
- Kích thước của array cố định, còn linked list hỗ trợ dynamic resizing một cách tự nhiên. Nếu array được khai báo quá nhỏ, cần cấp phát thêm một vùng memory lớn hơn để lưu các element của array, sau đó copy array cũ vào đó; thao tác này khá tốn thời gian!

## 3. Stack

### 3.1. Giới thiệu về stack

**Stack** chỉ cho phép thêm data (`push`) và loại bỏ data (`pop`) ở một đầu của linear data collection có thứ tự (gọi là top của stack). Vì vậy, nó hoạt động theo nguyên tắc **last in, first out (LIFO, Last In First Out)**. **Trong stack, thao tác push và pop đều xảy ra ở top.**

Stack thường được triển khai bằng array một chiều hoặc linked list. Stack triển khai bằng array được gọi là **sequential stack**, còn stack triển khai bằng linked list được gọi là **linked stack**.

```java
Giả sử stack có n element.
Truy cập: O(n) //trường hợp xấu nhất
Chèn và xóa: O(1) //chèn và xóa element ở top
```

![Minh họa cấu trúc last in, first out của stack](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/stack.png)

### 3.2. Trường hợp sử dụng phổ biến của stack

Khi data cần xử lý chỉ liên quan đến việc chèn và xóa data ở một đầu, đồng thời có đặc tính **last in, first out (LIFO, Last In First Out)**, ta có thể sử dụng data structure stack.

#### 3.2.1. Triển khai chức năng quay lại và tiến tới của browser

Chỉ cần sử dụng hai stack (`Stack1` và `Stack2`) là có thể triển khai chức năng này. Ví dụ, bạn lần lượt xem bốn page 1,2,3,4, ta lần lượt push bốn page 1,2,3,4 vào `Stack1`. Khi muốn quay lại page 2, bạn nhấn nút quay lại; ta lần lượt pop hai page 4,3 khỏi `Stack1`, sau đó push chúng vào `Stack2`. Nếu lại muốn quay về page 3, bạn nhấn nút tiến tới; ta pop page 3 khỏi `Stack2`, sau đó push page này vào `Stack1`. Hình minh họa như sau:

![Triển khai chức năng quay lại và tiến tới của browser bằng hai stack](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/stack-browser-back-forward.png)

#### 3.2.2. Kiểm tra các symbol có xuất hiện theo cặp hay không

> Cho một string chỉ gồm `'('`, `')'`, `'{'`, `'}'`, `'['`, `']'`, hãy xác định string đó có hợp lệ hay không.
>
> String hợp lệ phải thỏa mãn:
>
> 1. Left bracket phải được đóng bằng right bracket cùng kiểu.
> 2. Left bracket phải được đóng theo đúng thứ tự.
>
> Ví dụ `"()"`, `"()[]{}"`, `"{[]}"` đều là string hợp lệ, còn `(]`, `([)]` thì không.

Đây thực tế là một bài toán trên LeetCode, ta có thể dùng `Stack` để giải quyết.

1. Trước hết, lưu quy tắc tương ứng giữa các bracket vào `Map`; điều này không cần bàn cãi.
2. Tạo một stack. Duyệt string: nếu character là left bracket thì thêm trực tiếp vào `stack`; nếu không, so sánh element trên top của `stack` với bracket này, nếu không bằng nhau thì trả về `false`. Sau khi duyệt xong, nếu `stack` rỗng thì trả về `true`.

```java
public boolean isValid(String s){
    // Quy tắc tương ứng giữa các bracket
    HashMap<Character, Character> mappings = new HashMap<Character, Character>();
    mappings.put(')', '(');
    mappings.put('}', '{');
    mappings.put(']', '[');
    Stack<Character> stack = new Stack<Character>();
    char[] chars = s.toCharArray();
    for (int i = 0; i < chars.length; i++) {
        if (mappings.containsKey(chars[i])) {
            char topElement = stack.empty() ? '#' : stack.pop();
            if (topElement != mappings.get(chars[i])) {
                return false;
            }
        } else {
            stack.push(chars[i]);
        }
    }
    return stack.isEmpty();
}
```

#### 3.2.3. Đảo ngược string

Chỉ cần push từng character trong string vào stack rồi pop ra.

#### 3.2.4. Duy trì function call

Function được gọi sau cùng phải hoàn tất thực thi trước, phù hợp với đặc tính **last in, first out (LIFO, Last In First Out)** của stack.
Ví dụ, recursive function call có thể được triển khai bằng stack; mỗi recursive call sẽ push parameter và return address vào stack.

#### 3.2.5. Duyệt theo chiều sâu (DFS)

Trong quá trình depth-first search, stack được dùng để lưu search path, nhằm backtrack về layer trước đó.

### 3.3. Triển khai stack

Stack có thể được triển khai bằng array hoặc linked list. Dù dựa trên array hay linked list, time complexity của push và pop đều là O(1).

Dưới đây, ta dùng array để triển khai một stack có các method cơ bản `push()`, `pop()` (trả về element trên top và pop khỏi stack), `peek()` (trả về element trên top nhưng không pop), `isEmpty()` và `size()`.

> Gợi ý: trước mỗi lần push, hãy kiểm tra capacity của stack có đủ hay không; nếu không đủ thì dùng `Arrays.copyOf()` để mở rộng.

```java
public class MyStack {
    private int[] storage;//array lưu các element trong stack
    private int capacity;//capacity của stack
    private int count;//số lượng element trong stack
    private static final int GROW_FACTOR = 2;

    //Constructor không có capacity ban đầu. Capacity mặc định là 8
    public MyStack() {
        this.capacity = 8;
        this.storage=new int[8];
        this.count = 0;
    }

    //Constructor có capacity ban đầu
    public MyStack(int initialCapacity) {
        if (initialCapacity < 1)
            throw new IllegalArgumentException("Capacity too small.");

        this.capacity = initialCapacity;
        this.storage = new int[initialCapacity];
        this.count = 0;
    }

    //Push vào stack
    public void push(int value) {
        if (count == capacity) {
            ensureCapacity();
        }
        storage[count++] = value;
    }

    //Đảm bảo capacity đủ lớn
    private void ensureCapacity() {
        int newCapacity = capacity * GROW_FACTOR;
        storage = Arrays.copyOf(storage, newCapacity);
        capacity = newCapacity;
    }

    //Trả về element trên top và pop khỏi stack
    public int pop() {
        if (count == 0)
            throw new IllegalArgumentException("Stack is empty.");
        count--;
        return storage[count];
    }

    //Trả về element trên top nhưng không pop khỏi stack
    public int peek() {
        if (count == 0){
            throw new IllegalArgumentException("Stack is empty.");
        }else {
            return storage[count-1];
        }
    }

    //Kiểm tra stack có rỗng hay không
    public boolean isEmpty() {
        return count == 0;
    }

    //Trả về số lượng element trong stack
    public int size() {
        return count;
    }

}
```

Kiểm tra:

```java
MyStack myStack = new MyStack(3);
myStack.push(1);
myStack.push(2);
myStack.push(3);
myStack.push(4);
myStack.push(5);
myStack.push(6);
myStack.push(7);
myStack.push(8);
System.out.println(myStack.peek());//8
System.out.println(myStack.size());//8
for (int i = 0; i < 8; i++) {
    System.out.println(myStack.pop());
}
System.out.println(myStack.isEmpty());//true
myStack.pop();//Error: java.lang.IllegalArgumentException: Stack is empty.
```

## 4. Queue

### 4.1. Giới thiệu về queue

**Queue** là linear list **first in, first out (FIFO, First In, First Out)**. Trong ứng dụng thực tế, queue thường được triển khai bằng linked list hoặc array. Queue triển khai bằng array được gọi là **sequential queue**, còn queue triển khai bằng linked list được gọi là **linked queue**. **Queue chỉ cho phép chèn ở rear, tức enqueue, và xóa ở front, tức dequeue.**

Cách hoạt động của queue tương tự stack; điểm khác biệt duy nhất là queue chỉ cho phép thêm data mới ở rear.

```java
Giả sử queue có n element.
Truy cập: O(n) //trường hợp xấu nhất
Chèn và xóa: O(1) //chèn ở rear và xóa ở front
```

![Queue](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/queue.png)

### 4.2. Phân loại queue

#### 4.2.1. Simple queue

Simple queue là queue phổ biến; mỗi lần thêm element, element đều được thêm vào cuối queue. Simple queue lại được chia thành **sequential queue (triển khai bằng array)** và **linked queue (triển khai bằng linked list)**.

**Sequential queue có vấn đề “false overflow”, tức có vị trí trống nhưng không thể thêm element.**

Giả sử hình dưới đây là một sequential queue. Ta dequeue hai element đầu tiên 1,2 rồi enqueue hai element 7,8. Khi thực hiện enqueue và dequeue, cả `front` và `rear` đều liên tục di chuyển về phía sau. Khi `rear` di chuyển đến cuối, ta không thể thêm data vào queue nữa dù trong array vẫn còn space trống; hiện tượng này là **“false overflow”**. Ngoài vấn đề false overflow, như hình dưới đây, khi thêm element 8, pointer `rear` di chuyển ra ngoài array (out of bounds).

> Để tránh việc front và rear trùng nhau khi chỉ có một element khiến xử lý trở nên phức tạp, ta đưa vào hai pointer: pointer `front` trỏ tới element đầu queue, pointer `rear` trỏ tới vị trí ngay sau element cuối queue. Như vậy, khi `front` bằng `rear`, queue không còn một element mà là queue rỗng. — Trích 《Đại thoại về Data Structure》

![False overflow của sequential queue](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/seq-queue-false-overflow.png)

#### 4.2.2. Circular queue

Circular queue có thể giải quyết vấn đề false overflow và out of bounds của sequential queue. Cách giải quyết là quay lại từ đầu, từ đó tạo thành vòng nối đầu với cuối; đây cũng là nguồn gốc tên gọi circular queue.

Vẫn dùng hình trên làm ví dụ: nếu trỏ pointer `rear` tới vị trí có array index bằng 0 thì sẽ không có vấn đề out of bounds. Khi tiếp tục thêm element vào queue, `rear` sẽ di chuyển về phía sau.

![Circular queue](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/circular-queue.png)

Trong sequential queue, ta nói queue rỗng khi `front==rear`; còn trong circular queue thì không giống vậy, trạng thái này cũng có thể là queue đầy như hình trên. Có hai cách giải quyết:

1. Có thể đặt một flag `flag`: khi `front==rear` và `flag=0` thì queue rỗng; khi `front==rear` và `flag=1` thì queue đầy.
2. Khi queue rỗng, điều kiện là `front==rear`. Khi queue đầy, ta đảm bảo array còn một vị trí trống và `rear` trỏ tới vị trí trống đó. Như hình dưới đây, điều kiện xác định queue đầy là: `(rear+1) % QueueSize==front`.

#### 4.2.3. Deque

**Deque** là queue cho phép chèn và xóa ở cả hai đầu queue, linh hoạt hơn so với simple queue.

Thông thường, ta có thể thực hiện các thao tác `addFirst`, `addLast`, `removeFirst` và `removeLast` trên deque.

#### 4.2.4. Priority queue

**Priority queue** xét từ cấu trúc bên trong thì không phải là linear data structure, nó thường được triển khai bằng heap.

1. Khi mỗi element enqueue, priority queue sẽ chèn element mới vào heap và điều chỉnh heap.
2. Khi dequeue ở đầu queue, priority queue sẽ trả về element trên top của heap và điều chỉnh heap.

Bạn có thể xem phần [Heap](https://javaguide.cn/cs-basics/data-structure/heap.html) để biết cách triển khai heap cụ thể.

Priority queue chỉ đảm bảo đầu queue là element có priority cao nhất (hoặc thấp nhất) hiện tại, không đảm bảo array bên dưới, iterator hoặc toàn bộ collection có thứ tự toàn cục. Sau mỗi lần lấy element đầu queue ra, element có priority tiếp theo mới trở thành đầu queue mới.

Mặc dù priority queue thường được triển khai bằng cấu trúc heap không tuyến tính, nó cung cấp cho người dùng khả năng dequeue theo priority thông qua queue interface. “Priority” ở đây chỉ mô tả thứ tự dequeue, không thể hiểu là tất cả element trong collection sẽ tự động được sắp xếp.

### 4.3. Trường hợp sử dụng phổ biến của queue

Khi cần xử lý data theo một thứ tự nhất định, có thể cân nhắc sử dụng data structure queue.

- **Blocking queue:** Blocking queue có thể được xem là queue có thêm thao tác blocking trên nền queue. Khi queue rỗng, thao tác dequeue bị block; khi queue đầy, thao tác enqueue bị block. Sử dụng blocking queue giúp dễ dàng triển khai mô hình “producer - consumer”.
- **Request/task queue trong thread pool:** Khi thread pool không còn thread rảnh, request cần thread resource cho task mới sẽ được xử lý thế nào? Câu trả lời là các task này sẽ được đưa vào task queue, chờ đến khi thread trong thread pool rảnh rồi lấy task ra khỏi queue để thực thi. Task queue được chia thành unbounded queue (triển khai dựa trên linked list) và bounded queue (triển khai dựa trên array). Đặc điểm của unbounded queue là capacity về lý thuyết không bị giới hạn; task có thể liên tục enqueue cho đến khi resource của system cạn kiệt. Ví dụ: blocking queue `LinkedBlockingQueue` mà `FixedThreadPool` sử dụng có capacity mặc định là `Integer.MAX_VALUE`, vì vậy có thể được xem là “unbounded queue”. Bounded queue thì khác: khi queue đầy, nếu tiếp tục submit task mới, do queue không thể chứa thêm task, thread pool sẽ reject các task này và throw exception `java.util.concurrent.RejectedExecutionException`.
- **Stack:** Deque có thể triển khai toàn bộ chức năng của stack (`push`, `pop` và `peek`), đồng thời các method liên quan đã được định nghĩa trong interface `Deque`. `Stack` chưa bị đánh dấu deprecated, nhưng nó là subclass của `Vector` từ thời kỳ đầu; tài liệu JDK khuyến nghị ưu tiên dùng `Deque` và implementation của nó (như `ArrayDeque`) để thực hiện thao tác stack.
- **Breadth-first search (BFS):** Trong quá trình breadth-first search trên graph, queue được dùng để lưu các node chờ truy cập, bảo đảm các node của graph được duyệt theo thứ tự từng layer.
- Process queue của Linux kernel (xếp theo priority)
- Party trong đời sống thực, playlist trên player;
- Message queue
- Và vân vân…

## Trọng tâm ôn tập phỏng vấn

Linear structure là nền tảng của bài toán algorithm và Java collection. Trong phỏng vấn, array, linked list, stack và queue thường được đặt cạnh nhau để so sánh.

| Structure   | Query             | Insertion/deletion      | Java type điển hình          | Dạng bài thường gặp                                         |
| ----------- | ----------------- | ----------------------- | ---------------------------- | ----------------------------------------------------------- |
| Array       | Theo index `O(1)` | Vị trí giữa `O(n)`      | Array bên dưới `ArrayList`   | Binary search, two pointers, prefix sum                     |
| Linked list | `O(n)`            | `O(1)` khi đã biết node | `LinkedList`                 | Đảo ngược linked list, fast/slow pointer, merge linked list |
| Stack       | Top `O(1)`        | Top `O(1)`              | `ArrayDeque`                 | Matching bracket, monotonic stack, DFS                      |
| Queue       | Front `O(1)`      | Enqueue/dequeue `O(1)`  | `ArrayDeque`, blocking queue | BFS, producer-consumer, xếp task vào queue                  |

Một số điểm hữu ích khi trả lời câu hỏi phỏng vấn:

- Array random access nhanh vì memory liên tục, có thể tính trực tiếp address thông qua base address và index.
- Linked list chèn, xóa nhanh với điều kiện đã lấy được node tại vị trí cần thao tác; nếu còn phải tìm trước thì tổng thể vẫn là `O(n)`.
- Trong Java, không khuyến nghị tiếp tục dùng `Stack`; lựa chọn phổ biến hơn là `Deque`, chẳng hạn `ArrayDeque`.
- Trong engineering, queue không chỉ được dùng cho algorithm BFS mà còn cho task queue của thread pool, message queue, rate limiting, traffic peak smoothing và các trường hợp khác.
- Điểm then chốt của circular queue là phân biệt queue rỗng và queue đầy; cách phổ biến là bỏ trống một vị trí hoặc duy trì riêng số lượng element.

## Bài tập đề xuất

- Array: [704. Binary Search](https://leetcode.cn/problems/binary-search/), [26. Remove Duplicates from Sorted Array](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/)
- Linked list: [206. Reverse Linked List](https://leetcode.cn/problems/reverse-linked-list/), [19. Remove Nth Node From End of List](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)
- Stack: [20. Valid Parentheses](https://leetcode.cn/problems/valid-parentheses/), [739. Daily Temperatures](https://leetcode.cn/problems/daily-temperatures/)
- Queue: [102. Binary Tree Level Order Traversal](https://leetcode.cn/problems/binary-tree-level-order-traversal/), [239. Sliding Window Maximum](https://leetcode.cn/problems/sliding-window-maximum/)

<!-- @include: @article-footer.snippet.md -->
