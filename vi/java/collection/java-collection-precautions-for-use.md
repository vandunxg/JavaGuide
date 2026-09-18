---
title: Tổng hợp lưu ý khi sử dụng Java Collections
description: "Tổng hợp lưu ý khi sử dụng Java Collections: hệ thống hóa các best practice về kiểm tra collection rỗng, bẫy Arrays.asList, vấn đề subList, lựa chọn concurrent container dựa trên Alibaba Java Coding Guidelines để tránh các lỗi thường gặp."
category: Java
tag:
  - Java Collections
head:
  - - meta
    - name: keywords
      content: Java Collections best practices,kiểm tra collection rỗng,Arrays.asList,subList,concurrent container,lưu ý sử dụng collection,tối ưu performance
---

Bài viết này tổng hợp các lưu ý thường gặp khi sử dụng collection và nguyên lý cụ thể của chúng dựa trên 《Alibaba Java Coding Guidelines》.

Bạn nên đọc kỹ vài lần để tránh gặp những lỗi cơ bản này khi tự viết code.

## Kiểm tra collection rỗng

《Alibaba Java Coding Guidelines》 mô tả như sau:

> **Để kiểm tra các phần tử bên trong mọi collection có rỗng hay không, hãy sử dụng method `isEmpty()`, không sử dụng cách `size()==0`.**

Lý do là method `isEmpty()` dễ đọc hơn và có độ phức tạp thời gian là `O(1)`.

Phần lớn collection mà chúng ta sử dụng có method `size()` với độ phức tạp thời gian cũng là `O(1)`. Tuy nhiên, cũng có nhiều collection có độ phức tạp khác `O(1)`, chẳng hạn `ConcurrentLinkedQueue` trong package `java.util.concurrent`. Method `isEmpty()` của `ConcurrentLinkedQueue` phán đoán thông qua method `first()`. Method `first()` trả về node đầu tiên trong queue có value khác `null` (node có value `null` vì được dùng cho logic xóa mềm trong iterator).

```java
public boolean isEmpty() { return first() == null; }

Node<E> first() {
    restartFromHead:
    for (;;) {
        for (Node<E> h = head, p = h, q;;) {
            boolean hasItem = (p.item != null);
            if (hasItem || (q = p.next) == null) {  // value của node hiện tại khác null hoặc đã đến cuối queue
                updateHead(h, p);  // đặt head thành p
                return hasItem ? p : null;
            }
            else if (p == q) continue restartFromHead;
            else p = q;  // p = p.next
        }
    }
}
```

Vì method `updateHead(h, p)` đều được thực thi khi insert và delete phần tử, nên độ phức tạp thời gian thực thi method này có thể được xem xấp xỉ là `O(1)`. Còn method `size()` cần duyệt toàn bộ linked list, nên độ phức tạp thời gian là `O(n)`.

```java
public int size() {
    int count = 0;
    for (Node<E> p = first(); p != null; p = succ(p))
        if (p.item != null)
            if (++count == Integer.MAX_VALUE)
                break;
    return count;
}
```

Ngoài ra, trong `ConcurrentHashMap` 1.7, độ phức tạp thời gian của method `size()` và method `isEmpty()` cũng khác nhau. `ConcurrentHashMap` 1.7 lưu số lượng phần tử trong mỗi `Segment`; method `size()` cần thống kê số lượng của từng `Segment`, còn `isEmpty()` chỉ cần tìm `Segment` đầu tiên không rỗng. Tuy nhiên, trong `ConcurrentHashMap` 1.8, cả method `size()` và `isEmpty()` đều cần gọi method `sumCount()` để tổng hợp số đếm trong `baseCount` và `CounterCell[]`. Dưới đây là source code của method `sumCount()`:

```java
final long sumCount() {
    CounterCell[] as = counterCells; CounterCell a;
    long sum = baseCount;
    if (as != null)
        for (int i = 0; i < as.length; ++i)
            if ((a = as[i]) != null)
                sum += a.value;
    return sum;
}
```

Trong môi trường concurrent, khi `ConcurrentHashMap` 1.8 phân tán việc cập nhật counter bằng `baseCount` và `CounterCell[]`, nó giảm cạnh tranh thay vì lưu số lượng node trong mỗi `Node`. Trong `ConcurrentHashMap` 1.7, số lượng phần tử được lưu trong mỗi `Segment`; method `size()` cần thống kê số lượng của từng `Segment`, còn `isEmpty()` chỉ cần tìm `Segment` đầu tiên không rỗng.

## Chuyển collection thành Map

《Alibaba Java Coding Guidelines》 mô tả như sau:

> **Khi sử dụng method `toMap()` của class `java.util.stream.Collectors` để chuyển thành collection `Map`, cần đặc biệt chú ý rằng khi value là `null`, exception NPE sẽ được throw.**

```java
class Person {
    private String name;
    private String phoneNumber;
     // getters and setters
}

List<Person> bookList = new ArrayList<>();
bookList.add(new Person("jack","18163138123"));
bookList.add(new Person("martin",null));
// Null pointer exception
bookList.stream().collect(Collectors.toMap(Person::getName, Person::getPhoneNumber));
```

Hãy cùng giải thích nguyên nhân.

Trước tiên, hãy xem method `toMap()` của class `java.util.stream.Collectors`. Có thể thấy bên trong method này gọi method `merge()` của interface `Map`.

```java
public static <T, K, U, M extends Map<K, U>>
Collector<T, ?, M> toMap(Function<? super T, ? extends K> keyMapper,
                            Function<? super T, ? extends U> valueMapper,
                            BinaryOperator<U> mergeFunction,
                            Supplier<M> mapSupplier) {
    BiConsumer<M, T> accumulator
            = (map, element) -> map.merge(keyMapper.apply(element),
                                          valueMapper.apply(element), mergeFunction);
    return new CollectorImpl<>(mapSupplier, accumulator, mapMerger(mergeFunction), CH_ID);
}
```

Method `merge()` của interface `Map` như sau. Đây là default implementation trong interface.

> Nếu bạn chưa hiểu các new feature của Java 8, hãy xem bài viết này: [《Tổng hợp Java8 New Features》](https://mp.weixin.qq.com/s/ojyl7B6PiHaTWADqmUq2rw).

```java
default V merge(K key, V value,
        BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    Objects.requireNonNull(value);
    V oldValue = get(key);
    V newValue = (oldValue == null) ? value :
               remappingFunction.apply(oldValue, value);
    if(newValue == null) {
        remove(key);
    } else {
        put(key, newValue);
    }
    return newValue;
}
```

Method `merge()` trước tiên sẽ gọi method `Objects.requireNonNull()` để kiểm tra value có rỗng hay không.

```java
public static <T> T requireNonNull(T obj) {
    if (obj == null)
        throw new NullPointerException();
    return obj;
}
```

> `Collectors` cũng cung cấp method `toMap()` không cần `mergeFunction`. Tuy nhiên, nếu xảy ra xung đột key, exception `duplicateKeyException` sẽ được throw. Vì vậy, khi sử dụng method `toMap()`, bạn nên luôn truyền `mergeFunction`.

## Duyệt collection

《Alibaba Java Coding Guidelines》 mô tả như sau:

> **Không thực hiện thao tác `remove/add` phần tử trong vòng lặp foreach. Khi remove phần tử, hãy sử dụng `Iterator`; nếu thao tác concurrent, cần lock object `Iterator`.**

Cần lưu ý rằng chỉ lock object `Iterator` không thể ngăn thread khác sửa collection. Lấy wrapper đồng bộ được trả về bởi `Collections.synchronizedXxx()` làm ví dụ: khi duyệt, cần đồng bộ trên collection đã được wrap và bảo đảm mọi truy cập đều được thực hiện thông qua wrapper đó.

Thông qua decompile, bạn sẽ thấy cú pháp foreach ở tầng dưới thực chất vẫn dựa trên `Iterator`. Tuy nhiên, thao tác `remove/add` trực tiếp gọi method của chính collection, chứ không gọi method `remove/add` của `Iterator`.

Điều này khiến `Iterator` bất ngờ phát hiện phần tử của mình đã bị `remove/add`, sau đó throw `ConcurrentModificationException` để thông báo đã xảy ra concurrent modification exception. Đây chính là **fail-fast mechanism** phát sinh trong trạng thái single-thread.

> **fail-fast mechanism**: Khi nhiều thread sửa một fail-fast collection, có thể throw `ConcurrentModificationException`. Ngay cả trong single-thread cũng có thể xảy ra tình huống này, như đã đề cập ở trên.
>
> Đọc thêm: [fail-fast là gì](https://www.cnblogs.com/54chensongxia/p/12470446.html).

Bắt đầu từ Java8, có thể sử dụng method `Collection#removeIf()` để xóa các phần tử thỏa mãn điều kiện cụ thể, ví dụ:

```java
List<Integer> list = new ArrayList<>();
for (int i = 1; i <= 10; ++i) {
    list.add(i);
}
list.removeIf(filter -> filter % 2 == 0); /* xóa toàn bộ số chẵn trong list */
System.out.println(list); /* [1, 3, 5, 7, 9] */
```

Ngoài cách trực tiếp sử dụng `Iterator` để duyệt như trên, bạn còn có thể:

- Sử dụng vòng lặp for thông thường.
- Tùy theo trường hợp sử dụng, dùng các class collection hỗ trợ snapshot iteration hoặc weakly consistent iteration. Ví dụ, iterator của `CopyOnWriteArrayList` dựa trên snapshot, còn iterator của `ConcurrentHashMap` là weakly consistent.
- …

## Loại bỏ phần tử trùng trong collection

《Alibaba Java Coding Guidelines》 mô tả như sau:

> **Có thể tận dụng đặc tính phần tử unique của `Set` để nhanh chóng loại bỏ phần tử trùng trong một collection, tránh dùng `contains()` của `List` để duyệt nhằm loại bỏ phần tử trùng hoặc kiểm tra việc chứa phần tử.**

Ở đây, chúng ta dùng `HashSet` và `ArrayList` làm ví dụ.

```java
// Ví dụ code loại bỏ phần tử trùng bằng Set
public static <T> Set<T> removeDuplicateBySet(List<T> data) {

    if (CollectionUtils.isEmpty(data)) {
        return new HashSet<>();
    }
    return new HashSet<>(data);
}

// Ví dụ code loại bỏ phần tử trùng bằng List
public static <T> List<T> removeDuplicateByList(List<T> data) {

    if (CollectionUtils.isEmpty(data)) {
        return new ArrayList<>();

    }
    List<T> result = new ArrayList<>(data.size());
    for (T current : data) {
        if (!result.contains(current)) {
            result.add(current);
        }
    }
    return result;
}

```

Điểm khác biệt cốt lõi của hai cách nằm ở implementation của method `contains()`.

Method `contains()` của `HashSet` ở tầng dưới phụ thuộc vào method `containsKey()` của `HashMap`, với độ phức tạp thời gian gần `O(1)` (khi không xảy ra hash collision là `O(1)`).

```java
private transient HashMap<E,Object> map;
public boolean contains(Object o) {
    return map.containsKey(o);
}
```

Nếu có N phần tử được insert vào Set, độ phức tạp thời gian sẽ gần `O(n)`.

Method `contains()` của `ArrayList` thực hiện bằng cách duyệt mọi phần tử, nên độ phức tạp thời gian gần `O(n)`.

```java
public boolean contains(Object o) {
    return indexOf(o) >= 0;
}
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}

```

## Chuyển collection thành array

《Alibaba Java Coding Guidelines》 mô tả như sau:

> **Khi sử dụng method chuyển collection thành array, bắt buộc dùng `toArray(T[] array)` của collection và truyền vào một empty array có type hoàn toàn giống với type cần trả về, độ dài bằng 0.**

Tham số của method `toArray(T[] array)` là một generic array. Nếu method `toArray` không truyền tham số, kết quả trả về là một array có type `Object`.

```java
String [] s= new String[]{
    "dog", "lazy", "a", "over", "jumps", "fox", "brown", "quick", "A"
};
List<String> list = Arrays.asList(s);
Collections.reverse(list);
// sẽ báo lỗi nếu không chỉ định type
s=list.toArray(new String[0]);
```

Nhờ JVM optimization, hiện nay việc dùng `new String[0]` làm tham số của method `Collection.toArray()` có performance tốt hơn. `new String[0]` đóng vai trò template để chỉ định type của array trả về, còn `0` giúp tiết kiệm space vì nó chỉ dùng để chỉ rõ type trả về. Xem thêm: <https://shipilev.net/blog/2016/arrays-wisdom-ancients/>

## Chuyển array thành collection

《Alibaba Java Coding Guidelines》 mô tả như sau:

> **Khi sử dụng utility class `Arrays.asList()` để chuyển array thành collection, không được sử dụng các method liên quan đến việc sửa collection; các method `add/remove/clear` của nó sẽ throw exception `UnsupportedOperationException`.**

Trước đây, tôi từng gặp một bẫy tương tự trong một project.

`Arrays.asList()` khá phổ biến trong quá trình development hằng ngày. Chúng ta có thể dùng nó để chuyển một array thành một collection `List`.

```java
String[] myArray = {"Apple", "Banana", "Orange"};
List<String> myList = Arrays.asList(myArray);
// hai câu lệnh trên tương đương với một câu lệnh bên dưới
List<String> myList = Arrays.asList("Apple","Banana", "Orange");
```

Mô tả của JDK source code cho method này:

```java
/**
  *Trả về một list có kích thước cố định được hỗ trợ bởi array đã chỉ định. Method này là cầu nối
  * giữa API dựa trên array và API dựa trên collection, kết hợp với Collection.toArray().
  * List trả về có thể serialization và implement interface RandomAccess.
  */
public static <T> List<T> asList(T... a) {
    return new ArrayList<>(a);
}
```

Sau đây là phần tổng hợp các lưu ý khi sử dụng.

**1. `Arrays.asList()` sẽ không tự động autobox và trải array primitive thành các phần tử của list.**

```java
int[] myArray = {1, 2, 3};
List myList = Arrays.asList(myArray);
System.out.println(myList.size());//1
System.out.println(myList.get(0));//giá trị địa chỉ của array
System.out.println(myList.get(1));//lỗi: ArrayIndexOutOfBoundsException
int[] array = (int[]) myList.get(0);
System.out.println(array[0]);//1
```

Khi truyền vào một array primitive, tham số mà `Arrays.asList()` thực sự nhận được không phải các phần tử trong array mà là chính object array! Khi đó, phần tử duy nhất của `List` chính là array này, từ đó giải thích được đoạn code trên.

Chúng ta có thể giải quyết vấn đề này bằng cách sử dụng array của wrapper type.

```java
Integer[] myArray = {1, 2, 3};
```

**2. Sử dụng các method sửa collection: `add()`, `remove()`, `clear()` sẽ throw exception.**

```java
List myList = Arrays.asList(1, 2, 3);
myList.add(4);//lỗi khi runtime: UnsupportedOperationException
myList.remove(1);//lỗi khi runtime: UnsupportedOperationException
myList.clear();//lỗi khi runtime: UnsupportedOperationException
```

Method `Arrays.asList()` trả về không phải `java.util.ArrayList`, mà là một inner class của `java.util.Arrays`. Inner class này không implement các method sửa collection, hay nói cách khác là không override các method này.

```java
List myList = Arrays.asList(1, 2, 3);
System.out.println(myList.getClass());//class java.util.Arrays$ArrayList
```

Dưới đây là source code đơn giản của `java.util.Arrays$ArrayList`; có thể thấy các method mà class này override.

```java
  private static class ArrayList<E> extends AbstractList<E>
        implements RandomAccess, java.io.Serializable
    {
        ...

        @Override
        public E get(int index) {
          ...
        }

        @Override
        public E set(int index, E element) {
          ...
        }

        @Override
        public int indexOf(Object o) {
          ...
        }

        @Override
        public boolean contains(Object o) {
           ...
        }

        @Override
        public void forEach(Consumer<? super E> action) {
          ...
        }

        @Override
        public void replaceAll(UnaryOperator<E> operator) {
          ...
        }

        @Override
        public void sort(Comparator<? super E> c) {
          ...
        }
    }
```

Tiếp tục xem method `add/remove/clear` của `java.util.AbstractList`, chúng ta sẽ hiểu tại sao chúng throw `UnsupportedOperationException`.

```java
public E remove(int index) {
    throw new UnsupportedOperationException();
}
public boolean add(E e) {
    add(size(), e);
    return true;
}
public void add(int index, E element) {
    throw new UnsupportedOperationException();
}

public void clear() {
    removeRange(0, size());
}
protected void removeRange(int fromIndex, int toIndex) {
    ListIterator<E> it = listIterator(fromIndex);
    for (int i=0, n=toIndex-fromIndex; i<n; i++) {
        it.next();
        it.remove();
    }
}
```

**Vậy làm thế nào để chuyển array thành `ArrayList` đúng cách?**

1. Tự implement utility class

```java
// JDK1.5+
static <T> List<T> arrayToList(final T[] array) {
  final List<T> l = new ArrayList<T>(array.length);

  for (final T s : array) {
    l.add(s);
  }
  return l;
}


Integer [] myArray = { 1, 2, 3 };
System.out.println(arrayToList(myArray).getClass());//class java.util.ArrayList
```

2. Cách đơn giản nhất

```java
List list = new ArrayList<>(Arrays.asList("a", "b", "c"))
```

3. Sử dụng `Stream` của Java8 (khuyến nghị)

```java
Integer [] myArray = { 1, 2, 3 };
List myList = Arrays.stream(myArray).collect(Collectors.toList());
// array primitive cũng có thể chuyển đổi (phụ thuộc vào thao tác autoboxing boxed)
int [] myArray2 = { 1, 2, 3 };
List myList = Arrays.stream(myArray2).boxed().collect(Collectors.toList());
```

4. Sử dụng Guava

Với immutable collection, bạn có thể sử dụng class [`ImmutableList`](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/ImmutableList.java) và các factory method [`of()`](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/ImmutableList.java#L101), [`copyOf()`](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/ImmutableList.java#L225) của nó (tham số không được là `null`):

```java
List<String> il = ImmutableList.of("string", "elements");  // from varargs
List<String> il = ImmutableList.copyOf(aStringArray);      // from array
```

Với mutable collection, bạn có thể sử dụng class [`Lists`](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/Lists.java) và factory method [`newArrayList()`](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/Lists.java#L87) của nó:

```java
List<String> l1 = Lists.newArrayList(anotherListOrCollection);    // from collection
List<String> l2 = Lists.newArrayList(aStringArray);               // from array
List<String> l3 = Lists.newArrayList("or", "string", "elements"); // from varargs
```

5. Sử dụng Apache Commons Collections

```java
List<String> list = new ArrayList<String>();
CollectionUtils.addAll(list, str);
```

6. Sử dụng method `List.of()` của Java9

```java
Integer[] array = {1, 2, 3};
List<Integer> list = List.of(array);
```

<!-- @include: @article-footer.snippet.md -->
