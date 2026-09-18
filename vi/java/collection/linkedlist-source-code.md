---
title: LinkedList phân tích source code
description: "Phân tích chuyên sâu source code LinkedList: cấu trúc doubly linked list, triển khai interface Deque, độ phức tạp thời gian O(1) khi chèn xóa đầu cuối, so sánh performance với ArrayList và các trường hợp sử dụng."
category: Java
tag:
  - Java Collections
head:
  - - meta
    - name: keywords
      content: "LinkedList source code,doubly linked list,interface Deque,khác biệt giữa LinkedList và ArrayList,performance chèn xóa,triển khai linked list"
---

<!-- @include: @article-header.snippet.md -->

## Giới thiệu về LinkedList

`LinkedList` là một collection class được triển khai dựa trên doubly linked list, thường được so sánh với `ArrayList`. Phần [Tổng hợp câu hỏi phỏng vấn Java Collections thường gặp (phần 1)](./java-collection-questions-01.md) có giới thiệu chi tiết về sự khác biệt giữa `LinkedList` và `ArrayList`.

![Doubly linked list](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/bidirectional-linkedlist.png)

Tuy nhiên, trong project, thường không sử dụng `LinkedList`. Hầu hết trường hợp cần `LinkedList` đều có thể dùng `ArrayList` thay thế, hơn nữa performance thường tốt hơn! Ngay cả tác giả của `LinkedList`, Joshua Bloch, cũng từng nói rằng ông chưa bao giờ sử dụng `LinkedList`.

![](https://oss.javaguide.cn/github/javaguide/redisimage-20220412110853807.png)

Ngoài ra, đừng mặc định cho rằng `LinkedList` là linked list nên phù hợp nhất cho các trường hợp thêm xóa phần tử. Như đã nói ở trên, `LinkedList` chỉ có độ phức tạp thời gian xấp xỉ O(1) khi chèn hoặc xóa phần tử ở đầu/cuối; trong các trường hợp khác, độ phức tạp thời gian trung bình khi thêm xóa phần tử đều là O(n).

### Độ phức tạp thời gian khi chèn và xóa phần tử trong LinkedList?

- Chèn/xóa ở đầu: chỉ cần sửa pointer của head node là hoàn tất thao tác chèn/xóa, nên độ phức tạp thời gian là O(1).
- Chèn/xóa ở cuối: chỉ cần sửa pointer của tail node là hoàn tất thao tác chèn/xóa, nên độ phức tạp thời gian là O(1).
- Chèn/xóa tại vị trí chỉ định: cần di chuyển đến vị trí chỉ định trước, sau đó sửa pointer của node chỉ định để hoàn tất chèn/xóa. Tuy nhiên, nhờ có pointer đầu và cuối, có thể bắt đầu từ pointer gần hơn, nên cần duyệt trung bình n/4 phần tử, độ phức tạp thời gian là O(n).

### Vì sao LinkedList không thể triển khai interface RandomAccess?

`RandomAccess` là một marker interface, dùng để biểu thị class triển khai interface này hỗ trợ random access (tức là có thể truy cập nhanh phần tử thông qua index). Vì data structure bên dưới của `LinkedList` là linked list, địa chỉ bộ nhớ không liên tục và chỉ có thể định vị bằng pointer, nên không hỗ trợ random access nhanh và không thể triển khai interface `RandomAccess`.

## Phân tích source code LinkedList

Ở đây lấy JDK1.8 làm ví dụ để phân tích source code cốt lõi bên dưới của `LinkedList`.

Định nghĩa class `LinkedList` như sau:

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
  //...
}
```

`LinkedList` kế thừa `AbstractSequentialList`, còn `AbstractSequentialList` lại kế thừa `AbstractList`.

Nếu đã đọc source code của `ArrayList`, chúng ta biết rằng `ArrayList` cũng kế thừa `AbstractList`, vì vậy `LinkedList` sẽ có phần lớn method tương tự `ArrayList`.

`LinkedList` triển khai các interface sau:

- `List`: biểu thị đây là một list, hỗ trợ các thao tác thêm, xóa, tìm kiếm... và có thể truy cập bằng index.
- `Deque`: kế thừa từ interface `Queue`, có đặc tính của deque, hỗ trợ chèn và xóa phần tử từ hai đầu, thuận tiện để triển khai các data structure như stack và queue. Cần lưu ý cách phát âm của `Deque` là "deck" [dɛk], nhiều người thường đọc sai từ này.
- `Cloneable`: biểu thị class có khả năng copy, có thể thực hiện deep copy hoặc shallow copy.
- `Serializable`: biểu thị class có thể thực hiện serialization, tức là chuyển object thành byte stream để lưu trữ lâu dài hoặc truyền qua network, rất thuận tiện.

![Class diagram của LinkedList](https://oss.javaguide.cn/github/javaguide/java/collection/linkedlist--class-diagram.png)

Các phần tử trong `LinkedList` được định nghĩa bằng `Node`:

```java
private static class Node<E> {
    E item;// giá trị node
    Node<E> next; // node tiếp theo được trỏ tới (successor node)
    Node<E> prev; // node trước đó được trỏ tới (predecessor node)

    // Thứ tự các tham số khởi tạo lần lượt là: predecessor node, giá trị của node, successor node
    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

### Khởi tạo

`LinkedList` có một constructor không tham số và một constructor có tham số.

```java
// Tạo một object linked list rỗng
public LinkedList() {
}

// Nhận một collection làm tham số và tạo một linked list có cùng các phần tử với collection truyền vào
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

### Chèn phần tử

Ngoài việc triển khai các method liên quan đến interface `List`, `LinkedList` còn triển khai nhiều method của interface `Deque`, nên có nhiều cách để chèn phần tử.

Ở đây lấy các method chèn liên quan đến interface `List` làm ví dụ để giải thích source code, tương ứng với method `add()`.

Method `add()` có hai phiên bản:

- `add(E e)`: dùng để chèn phần tử ở cuối `LinkedList`, tức là đặt phần tử mới làm phần tử cuối cùng của linked list, độ phức tạp thời gian là O(1).
- `add(int index, E element)`: dùng để chèn phần tử tại vị trí chỉ định. Cách chèn này cần di chuyển đến vị trí chỉ định trước, sau đó sửa pointer của node chỉ định để hoàn tất chèn/xóa, vì vậy cần di chuyển trung bình n/4 phần tử, độ phức tạp thời gian là O(n).

```java
// Chèn phần tử ở cuối linked list
public boolean add(E e) {
    linkLast(e);
    return true;
}

// Chèn phần tử tại vị trí chỉ định của linked list
public void add(int index, E element) {
    // Kiểm tra index có vượt quá phạm vi hay không
    checkPositionIndex(index);

    // Kiểm tra index có phải là vị trí cuối linked list hay không
    if (index == size)
        // Nếu đúng thì gọi trực tiếp method linkLast để chèn node phần tử vào cuối linked list
        linkLast(element);
    else
        // Nếu không thì gọi method linkBefore để chèn nó trước phần tử chỉ định
        linkBefore(element, node(index));
}

// Chèn node phần tử vào cuối linked list
void linkLast(E e) {
    // Gán phần tử cuối cùng (truyền reference) cho node l
    final Node<E> l = last;
    // Tạo node, chỉ định node trước là node cuối last và node sau là null
    final Node<E> newNode = new Node<>(l, e, null);
    // Trỏ reference last đến node mới
    last = newNode;
    // Kiểm tra tail node có rỗng hay không
    // Nếu l là null, nghĩa là đây là lần đầu thêm phần tử
    if (l == null)
        // Nếu là lần đầu thêm, gán first bằng node mới; lúc này linked list chỉ có một phần tử
        first = newNode;
    else
        // Nếu không phải lần đầu thêm, gán node mới cho next của l (phần tử cuối cùng trước khi thêm)
        l.next = newNode;
    size++;
    modCount++;
}

// Chèn phần tử trước phần tử chỉ định
void linkBefore(E e, Node<E> succ) {
    // assert succ != null; assertion rằng succ không phải null
    // Định nghĩa một node lưu reference prev của succ, tức là thông tin node trước đó
    final Node<E> pred = succ.prev;
    // Khởi tạo node và chỉ rõ predecessor và successor
    final Node<E> newNode = new Node<>(pred, e, succ);
    // Trỏ reference prev của node succ đến node mới
    succ.prev = newNode;
    // Kiểm tra predecessor có phải null không; null nghĩa là succ là node đầu tiên
    if (pred == null)
        // Node mới trở thành node đầu tiên
        first = newNode;
    else
        // Trỏ reference successor của predecessor của node succ đến node mới
        pred.next = newNode;
    size++;
    modCount++;
}
```

### Lấy phần tử

`LinkedList` có tổng cộng 3 method liên quan đến việc lấy phần tử:

1. `getFirst()`: lấy phần tử đầu tiên của linked list.
2. `getLast()`: lấy phần tử cuối cùng của linked list.
3. `get(int index)`: lấy phần tử tại vị trí chỉ định của linked list.

```java
// Lấy phần tử đầu tiên của linked list
public E getFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}

// Lấy phần tử cuối cùng của linked list
public E getLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return l.item;
}

// Lấy phần tử tại vị trí chỉ định của linked list
public E get(int index) {
  // Kiểm tra index có vượt phạm vi hay không; nếu vượt phạm vi thì ném exception
  checkElementIndex(index);
  // Trả về phần tử tương ứng với index trong linked list
  return node(index).item;
}
```

Điểm cốt lõi ở đây là method `node(int index)`:

```java
// Trả về node không rỗng tại index chỉ định
Node<E> node(int index) {
    // Assertion rằng index chưa vượt quá phạm vi
    // assert isElementIndex(index);
    // Nếu index nhỏ hơn một nửa size thì tìm từ đầu (tìm về phía sau), ngược lại tìm từ cuối về trước
    if (index < (size >> 1)) {
        Node<E> x = first;
        // Duyệt, lặp để tìm về phía sau cho đến khi i == index
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

Các method như `get(int index)` hoặc `remove(int index)` đều gọi method này bên trong để lấy node tương ứng.

Từ source code của method này có thể thấy method xác định bắt đầu duyệt từ đầu hay cuối linked list bằng cách so sánh index với một nửa size của linked list. Nếu index nhỏ hơn một nửa size thì duyệt từ đầu linked list, ngược lại duyệt từ cuối linked list. Nhờ đó có thể tìm node mục tiêu trong thời gian ngắn hơn, tận dụng đầy đủ đặc tính của doubly linked list để cải thiện hiệu quả.

### Xóa phần tử

`LinkedList` có tổng cộng 5 method liên quan đến việc xóa phần tử:

1. `removeFirst()`: xóa và trả về phần tử đầu tiên của linked list.
2. `removeLast()`: xóa và trả về phần tử cuối cùng của linked list.
3. `remove(E e)`: xóa phần tử chỉ định xuất hiện đầu tiên trong linked list; nếu phần tử không tồn tại thì trả về false.
4. `remove(int index)`: xóa phần tử tại index chỉ định và trả về giá trị của phần tử đó.
5. `void clear()`: loại bỏ tất cả phần tử trong linked list này.

```java
// Xóa và trả về phần tử đầu tiên của linked list
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}

// Xóa và trả về phần tử cuối cùng của linked list
public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}

// Xóa phần tử chỉ định xuất hiện đầu tiên trong linked list; nếu phần tử không tồn tại thì trả về false
public boolean remove(Object o) {
    // Nếu phần tử chỉ định là null, duyệt linked list và xóa phần tử null đầu tiên tìm được
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        // Nếu không phải null, duyệt linked list để tìm node cần xóa
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}

// Xóa phần tử tại vị trí chỉ định của linked list
public E remove(int index) {
    // Kiểm tra index có vượt quá phạm vi hay không; nếu vượt quá thì ném exception
    checkElementIndex(index);
    return unlink(node(index));
}
```

Điểm cốt lõi ở đây là method `unlink(Node<E> x)`:

```java
E unlink(Node<E> x) {
    // Assertion rằng x không phải null
    // assert x != null;
    // Lấy phần tử của node hiện tại (tức node cần xóa)
    final E element = x.item;
    // Lấy node tiếp theo của node hiện tại
    final Node<E> next = x.next;
    // Lấy node trước đó của node hiện tại
    final Node<E> prev = x.prev;

    // Nếu node trước đó là null thì node hiện tại là head node
    if (prev == null) {
        // Trỏ trực tiếp head của linked list đến node tiếp theo của node hiện tại
        first = next;
    } else { // Nếu node trước đó không phải null
        // Trỏ pointer next của node trước đó đến node tiếp theo của node hiện tại
        prev.next = next;
        // Đặt pointer prev của node hiện tại thành null, giúp GC thu hồi
        x.prev = null;
    }

    // Nếu node tiếp theo là null thì node hiện tại là tail node
    if (next == null) {
        // Trỏ trực tiếp tail của linked list đến node trước đó của node hiện tại
        last = prev;
    } else { // Nếu node tiếp theo không phải null
        // Trỏ pointer prev của node tiếp theo đến node trước đó của node hiện tại
        next.prev = prev;
        // Đặt pointer next của node hiện tại thành null, giúp GC thu hồi
        x.next = null;
    }

    // Đặt phần tử của node hiện tại thành null, giúp GC thu hồi
    x.item = null;
    size--;
    modCount++;
    return element;
}
```

Logic của method `unlink()` như sau:

1. Trước hết lấy predecessor và successor của node x cần xóa;
2. Kiểm tra node cần xóa có phải head node hoặc tail node hay không:
   - Nếu x là head node thì trỏ first đến successor node next của x
   - Nếu x là tail node thì trỏ last đến predecessor node prev của x
   - Nếu x không phải head node cũng không phải tail node thì thực hiện bước tiếp theo
3. Trỏ successor của predecessor node của node x cần xóa đến successor next của node cần xóa, ngắt liên kết giữa x và x.prev;
4. Trỏ predecessor của successor node của node x cần xóa đến predecessor prev của node cần xóa, ngắt liên kết giữa x và x.next;
5. Đặt phần tử của node cần xóa thành null và thay đổi độ dài linked list.

Có thể tham khảo hình dưới đây để hiểu (nguồn hình: [Phân tích source code LinkedList (JDK 1.8)](https://www.tianxiaobo.com/2018/01/31/LinkedList-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-JDK-1-8/)):

![Logic của method unlink](https://oss.javaguide.cn/github/javaguide/java/collection/linkedlist-unlink.jpg)

### Duyệt linked list

Khuyến nghị sử dụng vòng lặp `for-each` để duyệt các phần tử trong `LinkedList`; vòng lặp `for-each` cuối cùng sẽ được chuyển thành dạng iterator.

```java
LinkedList<String> list = new LinkedList<>();
list.add("apple");
list.add("banana");
list.add("pear");

for (String fruit : list) {
    System.out.println(fruit);
}
```

Điểm cốt lõi khi duyệt `LinkedList` chính là triển khai iterator của nó.

```java
// Iterator hai chiều
private class ListItr implements ListIterator<E> {
    // Node đã đi qua khi gọi next() hoặc previous() lần gần nhất;
    private Node<E> lastReturned;
    // Node sẽ được duyệt tiếp theo;
    private Node<E> next;
    // Index của node sẽ được duyệt tiếp theo, tức index của successor node của node hiện tại;
    private int nextIndex;
    // Giá trị modification count được kỳ vọng trong lần duyệt hiện tại, dùng để so sánh với modCount của LinkedList và xác định linked list có bị sửa bởi thread khác hay không.
    private int expectedModCount = modCount;
    …………
}
```

Sau đây giới thiệu chi tiết các method cốt lõi trong iterator `ListItr`.

Trước tiên xem iterator theo hướng từ đầu đến cuối:

```java
// Kiểm tra còn node tiếp theo hay không
public boolean hasNext() {
    // Kiểm tra index của node tiếp theo có nhỏ hơn kích thước linked list hay không; nếu có thì vẫn còn phần tử tiếp theo để duyệt
    return nextIndex < size;
}
// Lấy node tiếp theo
public E next() {
    // Kiểm tra linked list có bị sửa trong quá trình duyệt hay không
    checkForComodification();
    // Kiểm tra còn node tiếp theo để duyệt hay không; nếu không thì ném exception NoSuchElementException
    if (!hasNext())
        throw new NoSuchElementException();
    // Trỏ lastReturned đến node hiện tại
    lastReturned = next;
    // Trỏ next đến node tiếp theo
    next = next.next;
    nextIndex++;
    return lastReturned.item;
}
```

Tiếp theo xem iterator theo hướng từ cuối đến đầu:

```java
// Kiểm tra còn node trước đó hay không
public boolean hasPrevious() {
    return nextIndex > 0;
}

// Lấy node trước đó
public E previous() {
    // Kiểm tra linked list có bị sửa trong quá trình duyệt hay không
    checkForComodification();
    // Nếu không có node trước đó thì ném exception
    if (!hasPrevious())
        throw new NoSuchElementException();
    // Trỏ lastReturned và next đến node trước đó
    lastReturned = next = (next == null) ? last : next.prev;
    nextIndex--;
    return lastReturned.item;
}
```

Nếu cần xóa hoặc chèn phần tử, cũng có thể sử dụng iterator để thực hiện.

```java
LinkedList<String> list = new LinkedList<>();
list.add("apple");
list.add(null);
list.add("banana");

// Method removeIf của interface Collection về cơ bản vẫn dựa trên iterator
list.removeIf(Objects::isNull);

for (String fruit : list) {
    System.out.println(fruit);
}
```

Method xóa phần tử tương ứng với iterator như sau:

```java
// Xóa phần tử được trả về ở lần gọi trước từ list
public void remove() {
    // Kiểm tra linked list có bị sửa trong quá trình duyệt hay không
    checkForComodification();
    // Nếu node được trả về ở lần trước là null thì ném exception
    if (lastReturned == null)
        throw new IllegalStateException();

    // Lấy node tiếp theo của node được trả về ở lần trước
    Node<E> lastNext = lastReturned.next;
    // Xóa node được trả về ở lần trước khỏi linked list
    unlink(lastReturned);
    // Sửa pointer
    if (next == lastReturned)
        next = lastNext;
    else
        nextIndex--;
    // Đặt reference của node được trả về ở lần trước thành null, giúp GC thu hồi
    lastReturned = null;
    expectedModCount++;
}
```

## Kiểm thử các method thường dùng của LinkedList

Code:

```java
// Tạo object LinkedList
LinkedList<String> list = new LinkedList<>();

// Thêm phần tử vào cuối linked list
list.add("apple");
list.add("banana");
list.add("pear");
System.out.println("Nội dung linked list: " + list);

// Chèn phần tử tại vị trí chỉ định
list.add(1, "orange");
System.out.println("Nội dung linked list: " + list);

// Lấy phần tử tại vị trí chỉ định
String fruit = list.get(2);
System.out.println("Phần tử tại index 2: " + fruit);

// Sửa phần tử tại vị trí chỉ định
list.set(3, "grape");
System.out.println("Nội dung linked list: " + list);

// Xóa phần tử tại vị trí chỉ định
list.remove(0);
System.out.println("Nội dung linked list: " + list);

// Xóa phần tử chỉ định xuất hiện đầu tiên
list.remove("banana");
System.out.println("Nội dung linked list: " + list);

// Lấy độ dài linked list
int size = list.size();
System.out.println("Độ dài linked list: " + size);

// Xóa toàn bộ linked list
list.clear();
System.out.println("Linked list sau khi xóa toàn bộ: " + list);
```

Kết quả:

```plain
Phần tử tại index 2: banana
Nội dung linked list: [apple, orange, banana, grape]
Nội dung linked list: [orange, banana, grape]
Nội dung linked list: [orange, grape]
Độ dài linked list: 2
Linked list sau khi xóa toàn bộ: []
```

<!-- @include: @article-footer.snippet.md -->
