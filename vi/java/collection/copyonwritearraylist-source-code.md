---
title: Phân tích source code CopyOnWriteArrayList
description: "Giải thích chuyên sâu về CopyOnWriteArrayList: cơ chế Copy-On-Write (COW), trường hợp đọc nhiều ghi ít, triển khai List thread-safe, đảm bảo tính nhất quán snapshot và cân nhắc chi phí bộ nhớ."
category: Java
tag:
  - Java Collections
head:
  - - meta
    - name: keywords
      content: CopyOnWriteArrayList source code, Copy-On-Write COW, thread-safe List, đọc nhiều ghi ít, concurrent container, tính nhất quán snapshot
---

## Giới thiệu về CopyOnWriteArrayList

Trước JDK1.5, nếu muốn sử dụng `List` an toàn trong môi trường concurrent, bạn có thể chọn `Vector` hoặc synchronized wrapper được trả về bởi `Collections.synchronizedList()`. `Vector` là một collection cũ và đã lỗi thời. Hầu hết các method như thêm, xóa, sửa và tìm kiếm của `Vector` đều được thêm `synchronized`. Cách này tuy có thể đảm bảo synchronization, nhưng tương đương với việc đặt một big lock lên toàn bộ `Vector`, khiến mỗi method khi thực thi đều phải lấy lock và dẫn đến performance rất thấp.

JDK1.5 giới thiệu package `java.util.concurrent` (JUC), cung cấp nhiều container thread-safe có performance concurrent tốt. Trong đó, implementation `List` thread-safe duy nhất là `CopyOnWriteArrayList`. Về phần tổng hợp các concurrent container thường gặp trong package `java.util.concurrent`, bạn có thể xem bài viết này: [Tổng hợp các concurrent container thường gặp trong Java](https://javaguide.cn/java/concurrent/java-concurrent-collections.html).

### CopyOnWriteArrayList có gì đặc biệt?

Trong phần lớn business scenario, thao tác đọc thường nhiều hơn đáng kể so với thao tác ghi. Vì thao tác đọc không sửa đổi dữ liệu hiện có, việc lock cho mỗi lần đọc thực ra là một sự lãng phí resource. Ngược lại, chúng ta nên cho phép nhiều thread đồng thời truy cập dữ liệu bên trong `List`, vì thao tác đọc vốn an toàn.

Tư tưởng này khá tương đồng với thiết kế của `ReentrantReadWriteLock`, tức là read-read không mutual exclusion, read-write mutual exclusion, write-write mutual exclusion (chỉ read-read không mutual exclusion). `CopyOnWriteArrayList` hiện thực tư tưởng này ở mức cao hơn. Để phát huy performance của thao tác đọc đến mức tối đa, thao tác đọc trong `CopyOnWriteArrayList` hoàn toàn không cần lock. Đặc biệt hơn, thao tác ghi cũng không block thao tác đọc, chỉ có write-write mới mutual exclusion. Nhờ vậy, performance của thao tác đọc có thể được cải thiện đáng kể.

Core thread-safe của `CopyOnWriteArrayList` nằm ở việc nó sử dụng strategy **Copy-On-Write**, như chính tên `CopyOnWriteArrayList` đã thể hiện.

### Tư tưởng của Copy-On-Write là gì?

“Copy-On-Write” trong tên `CopyOnWriteArrayList` nghĩa là copy khi ghi, viết tắt là COW.

Dưới đây là phần giới thiệu về Copy-On-Write trên Wikipedia, khá dễ hiểu:

> Copy khi ghi (tiếng Anh: Copy-on-write, viết tắt COW) là một strategy tối ưu hóa trong lĩnh vực lập trình máy tính. Tư tưởng cốt lõi là: nếu nhiều caller đồng thời yêu cầu cùng một resource (chẳng hạn dữ liệu lưu trên memory hoặc disk), họ sẽ cùng nhận một pointer trỏ đến cùng resource đó. Chỉ khi một caller cố gắng sửa nội dung resource, system mới thực sự tạo một bản sao riêng (private copy) cho caller đó, còn resource ban đầu mà các caller khác nhìn thấy vẫn không thay đổi. Quá trình này trong suốt với các caller khác. Ưu điểm chính của cách làm này là nếu caller không sửa resource thì sẽ không tạo private copy, vì vậy nhiều caller chỉ thực hiện thao tác đọc có thể dùng chung một resource.

Lấy `CopyOnWriteArrayList` làm ví dụ: khi cần sửa nội dung của `CopyOnWriteArrayList` (các thao tác như `add`, `set`, `remove`), nó không sửa trực tiếp array gốc mà trước tiên tạo một bản sao của array bên dưới, thực hiện sửa trên array bản sao, sau đó gán array đã sửa trở lại. Nhờ vậy có thể đảm bảo thao tác ghi không ảnh hưởng đến thao tác đọc.

Có thể thấy cơ chế Copy-On-Write rất phù hợp với concurrent scenario đọc nhiều ghi ít và có thể cải thiện đáng kể concurrent performance của system.

Tuy nhiên, cơ chế Copy-On-Write không phải silver bullet, nó vẫn có một số nhược điểm:

1. Chi phí bộ nhớ: mỗi thao tác ghi đều cần copy một bản dữ liệu gốc, chiếm thêm memory space. Khi data volume lớn, điều này có thể khiến resource bộ nhớ không đủ.
2. Chi phí thao tác ghi: mỗi thao tác ghi đều cần copy dữ liệu gốc rồi mới sửa và thay thế, nên chi phí thao tác ghi tương đối lớn. Trong scenario ghi thường xuyên, performance có thể bị ảnh hưởng.
3. Tính nhất quán snapshot: iterator giữ snapshot của array bên dưới tại thời điểm được tạo, các thay đổi sau đó sẽ không phản ánh vào iterator này.
4. …

## Phân tích source code CopyOnWriteArrayList

Ở đây lấy JDK1.8 làm ví dụ để phân tích source code cốt lõi bên trong của `CopyOnWriteArrayList`.

Định nghĩa class `CopyOnWriteArrayList` như sau:

```java
public class CopyOnWriteArrayList<E>
extends Object
implements List<E>, RandomAccess, Cloneable, Serializable
{
  //...
}
```

`CopyOnWriteArrayList` implement các interface sau:

- `List` : cho biết đây là một list, hỗ trợ thao tác thêm, xóa, tìm kiếm và có thể truy cập bằng index.
- `RandomAccess`: đây là một marker interface, cho biết collection `List` implement interface này hỗ trợ **random access nhanh**.
- `Cloneable`: cho biết nó hỗ trợ copy thông qua method `clone()`, `CopyOnWriteArrayList#clone()` trả về shallow copy.
- `Serializable` : cho biết nó có thể thực hiện serialization, tức là chuyển object thành byte stream để persistent storage hoặc truyền qua network, rất thuận tiện.

![Class diagram CopyOnWriteArrayList](https://oss.javaguide.cn/github/javaguide/java/collection/copyonwritearraylist-class-diagram.png)

### Khởi tạo

`CopyOnWriteArrayList` có một constructor không tham số và hai constructor có tham số.

```java
// Tạo một CopyOnWriteArrayList rỗng
public CopyOnWriteArrayList() {
    setArray(new Object[0]);
}

// Tạo một CopyOnWriteArrayList chứa các element của collection được chỉ định theo thứ tự iterator của collection
public CopyOnWriteArrayList(Collection<? extends E> c) {
    Object[] elements;
    if (c.getClass() == CopyOnWriteArrayList.class)
        elements = ((CopyOnWriteArrayList<?>)c).getArray();
    else {
        elements = c.toArray();
        // c.toArray đôi khi (không chính xác) không trả về Object[] (xem 6260652)
        if (elements.getClass() != Object[].class)
            elements = Arrays.copyOf(elements, elements.length, Object[].class);
    }
    setArray(elements);
}

// Tạo một list chứa bản copy của array được chỉ định
public CopyOnWriteArrayList(E[] toCopyIn) {
    setArray(Arrays.copyOf(toCopyIn, toCopyIn.length, Object[].class));
}
```

### Chèn element

Method `add()` của `CopyOnWriteArrayList` có ba version:

- `add(E e)`: chèn element vào cuối `CopyOnWriteArrayList`.
- `add(int index, E element)`: chèn element vào vị trí được chỉ định trong `CopyOnWriteArrayList`.
- `addIfAbsent(E e)`: nếu element được chỉ định không tồn tại thì thêm element đó. Trả về true nếu thêm element thành công.

Ở đây dùng `add(E e)` làm ví dụ:

```java
// Chèn element vào cuối CopyOnWriteArrayList
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    // Lock
    lock.lock();
    try {
        // Lấy array ban đầu
        Object[] elements = getArray();
        // Độ dài array ban đầu
        int len = elements.length;
        // Tạo array mới có độ dài +1 và copy các element của array ban đầu sang array mới
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        // Đặt element ở cuối array mới
        newElements[len] = e;
        // array trỏ đến array mới
        setArray(newElements);
        return true;
    } finally {
        // Unlock
        lock.unlock();
    }
}
```

Có thể thấy từ source code trên:

- Bên trong method `add` sử dụng `ReentrantLock` để lock, đảm bảo synchronization và tránh nhiều thread đồng thời thực hiện thao tác ghi. Field lock được đánh dấu `final`, sau khi khởi tạo reference không thể trỏ đến object khác, đồng thời logic release lock được đặt trong `finally` để đảm bảo lock được release.
- `CopyOnWriteArrayList` implement thao tác ghi bằng cách copy array bên dưới: trước tiên tạo một array mới để chứa element được thêm, thực hiện thao tác ghi trên array mới, cuối cùng gán array mới cho reference của array bên dưới để thay thế array cũ. Điều này chứng minh điều đã nói ở trên: core thread-safe của `CopyOnWriteArrayList` nằm ở strategy **Copy-On-Write**.
- Mỗi thao tác ghi đều cần copy array bên dưới thông qua `Arrays.copyOf`, có time complexity O(n) và chiếm thêm memory space. Vì vậy, `CopyOnWriteArrayList` phù hợp với scenario đọc nhiều ghi ít. Khi thao tác ghi không thường xuyên và memory resource dồi dào, nó có thể cải thiện performance của system.
- `CopyOnWriteArrayList` không có thao tác mở rộng capacity thông qua method `grow()` tương tự `ArrayList`.

> Time complexity của method `Arrays.copyOf` là O(n), trong đó n biểu thị độ dài array cần copy. Nguyên lý của method này là trước tiên tạo một array mới, sau đó copy dữ liệu từ source array sang array mới và cuối cùng trả về array mới. Method này copy toàn bộ array nên time complexity tỷ lệ thuận với độ dài array, tức O(n). Cần lưu ý rằng do bên dưới gọi instruction copy cấp system, performance của method này trong ứng dụng thực tế khá tốt. Tuy nhiên, cũng cần kiểm soát lượng dữ liệu copy để tránh memory usage quá cao.

### Đọc element

Thao tác đọc của `CopyOnWriteArrayList` dựa trên array bên trong `array` và không thực sự sửa đổi array này. Vì vậy, thao tác đọc không cần synchronization control hay lock operation, có thể đảm bảo data safety. Với cơ chế này, nhiều thread có thể đồng thời đọc element trong list.

```java
// Array bên dưới, chỉ có thể truy cập thông qua method getArray và setArray
private transient volatile Object[] array;

public E get(int index) {
    return get(getArray(), index);
}

final Object[] getArray() {
    return array;
}

private E get(Object[] a, int index) {
    return (E) a[index];
}
```

Tuy nhiên, method `get` có tính nhất quán yếu (weakly consistent), trong một số trường hợp có thể đọc được value cũ.

Method `get(int index)` thực hiện qua hai bước:

1. Lấy reference của array hiện tại thông qua `getArray()`;
2. Trực tiếp lấy element có index là index từ array.

Quá trình này không lock, nên trong concurrent environment có thể xảy ra tình huống sau:

1. Thread 1 gọi method `get(int index)` để lấy value, bên trong lấy được reference của thuộc tính `array` thông qua method `getArray()`;
2. Thread 2 gọi các method sửa đổi như `add`, `set`, `remove` của `CopyOnWriteArrayList`, bên trong thay đổi giá trị của thuộc tính `array` thông qua method `setArray`;
3. Thread 1 vẫn lấy value từ array `array` cũ.

### Lấy số lượng element trong list

```java
public int size() {
    return getArray().length;
}
```

Mỗi lần copy, array `array` trong `CopyOnWriteArrayList` vừa đủ chứa toàn bộ element, không reserve trước một phần space như `ArrayList`. Vì vậy, `CopyOnWriteArrayList` không có thuộc tính `size`; độ dài array bên dưới của `CopyOnWriteArrayList` chính là số lượng element, nên method `size()` chỉ cần trả về độ dài array.

### Xóa element

`CopyOnWriteArrayList` có tổng cộng 4 method liên quan đến việc xóa element:

1. `remove(int index)`: xóa element tại vị trí được chỉ định trong list. Di chuyển mọi element phía sau sang trái (trừ 1 khỏi index của chúng).
2. `boolean remove(Object o)`: xóa element được chỉ định xuất hiện đầu tiên trong list; trả về false nếu element đó không tồn tại.
3. `boolean removeAll(Collection<?> c)`: xóa khỏi list tất cả element có trong collection được chỉ định.
4. `void clear()`: xóa toàn bộ element trong list.

Ở đây dùng `remove(int index)` làm ví dụ:

```java
public E remove(int index) {
    // Lấy reentrant lock
    final ReentrantLock lock = this.lock;
    // Lock
    lock.lock();
    try {
         // Lấy array hiện tại
        Object[] elements = getArray();
        // Lấy độ dài array hiện tại
        int len = elements.length;
        // Lấy element tại index được chỉ định (old value)
        E oldValue = get(elements, index);
        int numMoved = len - index - 1;
        // Kiểm tra element bị xóa có phải element cuối cùng hay không
        if (numMoved == 0)
             // Nếu xóa element cuối cùng thì trực tiếp copy tất cả element trước element đó sang array mới
            setArray(Arrays.copyOf(elements, len - 1));
        else {
            // Copy theo từng phần, copy các element trước index và sau index + 1 sang array mới
            // Độ dài array mới bằng độ dài array cũ - 1
            Object[] newElements = new Object[len - 1];
            System.arraycopy(elements, 0, newElements, 0, index);
            System.arraycopy(elements, index + 1, newElements, index,
                             numMoved);
            // Gán array mới cho reference array
            setArray(newElements);
        }
        return oldValue;
    } finally {
         // Unlock
        lock.unlock();
    }
}
```

### Kiểm tra element có tồn tại hay không

`CopyOnWriteArrayList` cung cấp hai method dùng để kiểm tra element được chỉ định có nằm trong list hay không:

- `contains(Object o)`: kiểm tra có chứa element được chỉ định hay không.
- `containsAll(Collection<?> c)`: kiểm tra list có chứa toàn bộ element của collection được chỉ định hay không.

```java
// Kiểm tra có chứa element được chỉ định hay không
public boolean contains(Object o) {
    // Lấy array hiện tại
    Object[] elements = getArray();
    // Gọi index để thử tìm element được chỉ định; nếu giá trị trả về lớn hơn hoặc bằng 0 thì trả về true, ngược lại trả về false
    return indexOf(o, elements, 0, elements.length) >= 0;
}

// Kiểm tra có chứa toàn bộ element của collection được chỉ định hay không
public boolean containsAll(Collection<?> c) {
    // Lấy array hiện tại
    Object[] elements = getArray();
    // Lấy độ dài array
    int len = elements.length;
    // Duyệt collection được chỉ định
    for (Object e : c) {
        // Lặp gọi method indexOf để kiểm tra; chỉ cần có một element không được chứa thì trực tiếp trả về false
        if (indexOf(e, elements, 0, len) < 0)
            return false;
    }
    // Biểu thị toàn bộ element đều được chứa hoặc collection được chỉ định là collection rỗng, khi đó trả về true
    return true;
}
```

## Test các method thường dùng của CopyOnWriteArrayList

Code:

```java
// Tạo một object CopyOnWriteArrayList
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();

// Thêm element vào list
list.add("Java");
list.add("Python");
list.add("C++");
System.out.println("List ban đầu: " + list);

// Sử dụng method get để lấy element tại vị trí được chỉ định
System.out.println("Element thứ hai của list là: " + list.get(1));

// Sử dụng method remove để xóa element được chỉ định
boolean result = list.remove("C++");
System.out.println("Kết quả xóa: " + result);
System.out.println("List sau khi xóa element: " + list);

// Sử dụng method set để cập nhật element tại vị trí được chỉ định
list.set(1, "Golang");
System.out.println("List sau khi cập nhật: " + list);

// Sử dụng method add để chèn element tại vị trí được chỉ định
list.add(0, "PHP");
System.out.println("List sau khi chèn element: " + list);

// Sử dụng method size để lấy kích thước list
System.out.println("Kích thước list là: " + list.size());

// Sử dụng method removeAll để xóa tất cả lần xuất hiện của element trong collection được chỉ định
result = list.removeAll(List.of("Java", "Golang"));
System.out.println("Kết quả xóa hàng loạt: " + result);
System.out.println("List sau khi xóa hàng loạt element: " + list);

// Sử dụng method clear để xóa toàn bộ element trong list
list.clear();
System.out.println("List sau khi xóa sạch: " + list);
```

Output:

```plain
List sau khi cập nhật: [Java, Golang]
List sau khi chèn element: [PHP, Java, Golang]
Kích thước list là: 3
Kết quả xóa hàng loạt: true
List sau khi xóa hàng loạt element: [PHP]
List sau khi xóa sạch: []
```

<!-- @include: @article-footer.snippet.md -->
