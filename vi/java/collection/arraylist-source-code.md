---
title: Phân tích source code ArrayList
description: "Giải thích chuyên sâu về ArrayList: cấu trúc array bên trong, cơ chế mở rộng 1,5 lần, truy cập ngẫu nhiên nhanh bằng RandomAccess, triển khai serialization và so sánh performance với Vector."
category: Java
tag:
  - Java Collection
head:
  - - meta
    - name: keywords
      content: ArrayList source code,ArrayList expansion mechanism,dynamic array,RandomAccess,ArrayList serialization,difference between ArrayList and Vector
---

## Giới thiệu về ArrayList

Bên trong `ArrayList` là một array, tương đương với dynamic array. So với array thông thường trong Java, capacity của nó có thể tăng động. Trước khi thêm nhiều phần tử, application có thể dùng `ensureCapacity` để tăng capacity của instance `ArrayList`. Điều này giúp giảm số lần reallocation khi tăng dần.

`ArrayList` kế thừa `AbstractList`, đồng thời implement các interface `List`, `RandomAccess`, `Cloneable`, `java.io.Serializable`.

```java

public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable{

  }
```

- `List`: cho biết đây là một list, hỗ trợ các thao tác thêm, xóa, tìm kiếm... và có thể truy cập bằng index.
- `RandomAccess`: đây là một marker interface, cho biết các `List` implement interface này hỗ trợ **truy cập ngẫu nhiên nhanh**. Trong `ArrayList`, ta có thể nhanh chóng lấy object tương ứng thông qua index phần tử, đây chính là truy cập ngẫu nhiên nhanh.
- `Cloneable`: cho biết nó hỗ trợ copy thông qua method `clone()`, `ArrayList#clone()` trả về shallow copy.
- `Serializable`: cho biết nó có thể thực hiện serialization, tức là chuyển object thành byte stream để lưu trữ lâu dài hoặc truyền qua network, rất thuận tiện.

![Class diagram ArrayList](https://oss.javaguide.cn/github/javaguide/java/collection/arraylist-class-diagram.png)

### Khác biệt giữa ArrayList và Vector? (Chỉ cần biết)

- `ArrayList` là implementation class chính của `List`, bên trong dùng `Object[]` để lưu trữ, phù hợp với các thao tác tìm kiếm thường xuyên và không thread-safe.
- `Vector` là implementation class lâu đời của `List`, bên trong dùng `Object[]` để lưu trữ và thread-safe.

### ArrayList có thể thêm giá trị null không?

`ArrayList` có thể lưu object thuộc mọi type, bao gồm cả giá trị `null`. Tuy nhiên, không nên thêm giá trị `null` vào `ArrayList`, vì giá trị `null` không có ý nghĩa và khiến code khó maintain; chẳng hạn quên xử lý null sẽ dẫn đến NullPointerException.

Code ví dụ:

```java
ArrayList<String> listOfStrings = new ArrayList<>();
listOfStrings.add(null);
listOfStrings.add("java");
System.out.println(listOfStrings);
```

Output:

```plain
[null, java]
```

### Khác biệt giữa ArrayList và LinkedList?

- **Có đảm bảo thread-safe hay không:** `ArrayList` và `LinkedList` đều không synchronized, tức là không đảm bảo thread-safe;
- **Data structure bên trong:** bên trong `ArrayList` dùng **array `Object[]`**; bên trong `LinkedList` dùng data structure **doubly linked list** (trước JDK1.6 là circular linked list, JDK1.7 đã bỏ tính circular. Hãy chú ý sự khác biệt giữa doubly linked list và doubly circular linked list, phần dưới sẽ giới thiệu cụ thể!);
- **Việc insert và delete có bị ảnh hưởng bởi vị trí phần tử hay không:**
  - `ArrayList` dùng array để lưu trữ, nên time complexity của việc insert và delete phần tử phụ thuộc vào vị trí phần tử. Ví dụ: khi thực thi method `add(E e)`, `ArrayList` mặc định append phần tử được chỉ định vào cuối list, trường hợp này có time complexity là O(1). Nhưng nếu insert và delete phần tử tại vị trí `i` đã chỉ định (`add(int index, E element)`), time complexity là O(n). Vì khi thực hiện các thao tác trên, phần tử tại `i` cùng `(n-i)` phần tử đứng sau nó trong collection đều phải dịch về sau hoặc về trước một vị trí.
  - `LinkedList` dùng linked list để lưu trữ, nên việc insert hoặc delete phần tử ở đầu hay cuối không bị ảnh hưởng bởi vị trí phần tử (`add(E e)`, `addFirst(E e)`, `addLast(E e)`, `removeFirst()`, `removeLast()`), time complexity là O(1). Nếu insert và delete phần tử tại vị trí `i` đã chỉ định (`add(int index, E element)`, `remove(Object o)`, `remove(int index)`), time complexity là O(n), vì cần di chuyển đến vị trí đã chỉ định trước rồi mới insert và delete.
- **Có hỗ trợ truy cập ngẫu nhiên nhanh hay không:** `LinkedList` không hỗ trợ truy cập phần tử ngẫu nhiên hiệu quả, còn `ArrayList` (implement interface `RandomAccess`) thì có. Truy cập ngẫu nhiên nhanh là nhanh chóng lấy object phần tử thông qua số thứ tự của phần tử (tương ứng với method `get(int index)`).
- **Mức sử dụng memory:** phần memory bị lãng phí của `ArrayList` chủ yếu nằm ở việc cuối list dự phòng một phần capacity nhất định, còn chi phí memory của `LinkedList` nằm ở việc mỗi phần tử cần nhiều memory hơn `ArrayList` (vì phải lưu direct successor, direct predecessor và data).

## Đọc source code cốt lõi của ArrayList

Ở đây lấy JDK1.8 làm ví dụ để phân tích source code bên trong `ArrayList`.

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    private static final long serialVersionUID = 8683452581122892189L;

    /**
     * Capacity mặc định ban đầu
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * Array rỗng (dùng cho empty instance).
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    // Shared empty array instance dùng cho empty instance có capacity mặc định.
    // Tách nó khỏi array EMPTY_ELEMENTDATA để biết cần tăng capacity bao nhiêu khi thêm phần tử đầu tiên.
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * Array lưu data của ArrayList
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * Số lượng phần tử mà ArrayList chứa
     */
    private int size;

    /**
     * Constructor có tham số capacity ban đầu (cho phép tự chỉ định kích thước ban đầu của collection khi tạo object ArrayList)
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            // Nếu tham số truyền vào lớn hơn 0, tạo array có kích thước initialCapacity
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            // Nếu tham số truyền vào bằng 0, tạo empty array
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            // Trường hợp khác, throw exception
            throw new IllegalArgumentException("Illegal Capacity: " +
                    initialCapacity);
        }
    }

    /**
     * Constructor không tham số mặc định
      * DEFAULTCAPACITY_EMPTY_ELEMENTDATA có length bằng 0; capacity ban đầu thực tế là empty array, khi thêm phần tử đầu tiên thì array mới có capacity 10
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * Tạo một list chứa các phần tử của collection đã chỉ định, theo thứ tự iterator của collection trả về.
     */
    public ArrayList(Collection<? extends E> c) {
        // Chuyển collection đã chỉ định thành array
        elementData = c.toArray();
        // Nếu length của array elementData khác 0
        if ((size = elementData.length) != 0) {
            // Nếu elementData không phải array type Object (c.toArray có thể trả về array không phải type Object nên cần câu lệnh dưới đây để kiểm tra)
            if (elementData.getClass() != Object[].class)
                // Gán nội dung của array elementData vốn không phải type Object vào array elementData type Object mới
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // Trường hợp khác, thay bằng empty array
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }

    /**
     * Điều chỉnh capacity của instance ArrayList này về size hiện tại của list. Application có thể dùng thao tác này để tối thiểu hóa storage của instance ArrayList.
     */
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
                    ? EMPTY_ELEMENTDATA
                    : Arrays.copyOf(elementData, size);
        }
    }
// Dưới đây là cơ chế mở rộng của ArrayList
// Cơ chế mở rộng của ArrayList cải thiện performance; nếu mỗi lần chỉ tăng một phần tử,
// việc insert thường xuyên sẽ dẫn đến copy thường xuyên, làm giảm performance, còn cơ chế mở rộng của ArrayList tránh được tình huống này.

    /**
     * Nếu cần, tăng capacity của instance ArrayList này để đảm bảo nó có thể chứa ít nhất số lượng phần tử đã chỉ định
     *
     * @param minCapacity capacity tối thiểu cần thiết
     */
    public void ensureCapacity(int minCapacity) {
        // Nếu không phải empty array mặc định thì giá trị của minExpand là 0;
        // nếu là empty array mặc định thì giá trị của minExpand là 10
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
                // Nếu không phải element table mặc định thì có thể dùng kích thước bất kỳ
                ? 0
                // Nếu là empty array mặc định thì nó phải đã có kích thước mặc định
                : DEFAULT_CAPACITY;

        // Nếu capacity tối thiểu lớn hơn capacity hiện có
        if (minCapacity > minExpand) {
            // Đảm bảo capacity đủ theo capacity tối thiểu cần thiết
            ensureExplicitCapacity(minCapacity);
        }
    }


    // Tính capacity cần thiết theo array hiện tại và minCapacity đã cho.
    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        // Nếu array hiện tại là empty array (trạng thái ban đầu), trả về giá trị lớn hơn giữa capacity mặc định và capacity tối thiểu làm capacity cần thiết
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        // Nếu không thì trả về trực tiếp capacity tối thiểu
        return minCapacity;
    }

    // Đảm bảo capacity bên trong đạt capacity tối thiểu đã chỉ định.
    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

    // Kiểm tra có cần mở rộng capacity hay không
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            // Gọi method grow để mở rộng capacity; gọi method này nghĩa là đã bắt đầu mở rộng capacity
            grow(minCapacity);
    }

    /**
     * Kích thước array tối đa có thể allocate
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * Method cốt lõi để mở rộng capacity của ArrayList.
     */
    private void grow(int minCapacity) {
        // oldCapacity là capacity cũ, newCapacity là capacity mới
        int oldCapacity = elementData.length;
        // Dịch phải oldCapacity một bit, hiệu quả tương đương oldCapacity /2,
        // ta biết tốc độ của phép toán bit nhanh hơn rất nhiều so với phép chia, kết quả của toàn bộ biểu thức là cập nhật capacity mới thành 1,5 lần capacity cũ,
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        // Sau đó kiểm tra capacity mới có lớn hơn capacity tối thiểu cần thiết hay không; nếu vẫn nhỏ hơn capacity tối thiểu thì lấy capacity tối thiểu làm capacity mới của array,
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        // Tiếp tục kiểm tra capacity mới có vượt quá capacity tối đa mà ArrayList định nghĩa hay không,
        // nếu vượt quá thì gọi hugeCapacity() để so sánh minCapacity với MAX_ARRAY_SIZE,
        // nếu minCapacity lớn hơn MAX_ARRAY_SIZE thì capacity mới là Integer.MAX_VALUE, nếu không thì capacity mới là MAX_ARRAY_SIZE.
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    // So sánh minCapacity với MAX_ARRAY_SIZE
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
                Integer.MAX_VALUE :
                MAX_ARRAY_SIZE;
    }

    /**
     * Trả về số lượng phần tử trong list này.
     */
    public int size() {
        return size;
    }

    /**
     * Trả về true nếu list này không chứa phần tử nào.
     */
    public boolean isEmpty() {
        // Chú ý sự khác biệt giữa = và ==
        return size == 0;
    }

    /**
     * Trả về true nếu list này chứa phần tử đã chỉ định.
     */
    public boolean contains(Object o) {
        // Method indexOf(): trả về index xuất hiện đầu tiên của phần tử đã chỉ định trong list này, nếu list không chứa phần tử này thì trả về -1
        return indexOf(o) >= 0;
    }

    /**
     * Trả về index xuất hiện đầu tiên của phần tử đã chỉ định trong list này, nếu list không chứa phần tử này thì trả về -1
     */
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i] == null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                // Method equals() dùng để so sánh
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * Trả về index xuất hiện cuối cùng của phần tử đã chỉ định trong list này, nếu list không chứa phần tử thì trả về -1.
     */
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size - 1; i >= 0; i--)
                if (elementData[i] == null)
                    return i;
        } else {
            for (int i = size - 1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * Trả về shallow copy của instance ArrayList này. (Bản thân các phần tử không được copy.)
     */
    public Object clone() {
        try {
            ArrayList<?> v = (ArrayList<?>) super.clone();
            // Chức năng của Arrays.copyOf là copy array, trả về array sau khi copy. Tham số là array được copy và length cần copy
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // Điều này không nên xảy ra vì chúng ta có thể clone
            throw new InternalError(e);
        }
    }

    /**
     * Trả về một array chứa toàn bộ phần tử trong list này theo đúng thứ tự (từ phần tử đầu tiên đến phần tử cuối cùng).
     * Array được trả về là "safe" vì list không giữ reference đến nó.
     * (Nói cách khác, method này phải allocate một array mới.)
     * Vì vậy, caller có thể tự do sửa đổi structure của array được trả về.
     * Lưu ý: nếu phần tử là reference type, việc sửa nội dung của phần tử sẽ ảnh hưởng đến object trong list ban đầu.
     * Method này đóng vai trò cầu nối giữa API dựa trên array và API dựa trên collection.
     */
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }

    /**
     * Trả về một array chứa toàn bộ phần tử trong list này theo đúng thứ tự (từ phần tử đầu tiên đến phần tử cuối cùng);
     * runtime type của array được trả về là runtime type của array đã chỉ định. Nếu list vừa với array đã chỉ định thì trả về array đó.
     * Nếu không, một array mới sẽ được allocate với runtime type của array đã chỉ định và size của list.
     * Nếu list vừa với array đã chỉ định nhưng còn dư chỗ (tức số lượng phần tử trong array lớn hơn số lượng phần tử của list), phần tử trong array ngay sau phần tử cuối của collection sẽ được set thành null.
     * (Chỉ khi caller biết list không chứa phần tử null thì mới có thể xác định size của list.)
     */
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // Tạo array có runtime type mới nhưng chứa nội dung của array ArrayList
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        // Gọi arraycopy() do System cung cấp để copy giữa các array
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }

    // Positional Access Operations

    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }

    /**
     * Trả về phần tử tại vị trí đã chỉ định trong list này.
     */
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }

    /**
     * Thay thế phần tử tại vị trí đã chỉ định trong list này bằng phần tử đã chỉ định.
     */
    public E set(int index, E element) {
        // Kiểm tra giới hạn của index
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        // Trả về phần tử ban đầu ở vị trí này
        return oldValue;
    }

    /**
     * Append phần tử đã chỉ định vào cuối list này.
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        // Ở đây có thể thấy bản chất việc thêm phần tử của ArrayList tương đương với việc gán giá trị cho array
        elementData[size++] = e;
        return true;
    }

    /**
     * Insert phần tử đã chỉ định tại vị trí đã chỉ định trong list này.
     * Trước tiên gọi rangeCheckForAdd để kiểm tra giới hạn của index; sau đó gọi method ensureCapacityInternal để đảm bảo capacity đủ lớn;
     * tiếp theo dịch mọi phần tử từ index trở đi về sau một vị trí; insert element vào vị trí index; cuối cùng tăng size lên 1.
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        // Cần xem implementation arraycopy() dùng để copy giữa các array này, bên dưới sẽ dùng arraycopy() để array tự copy chính nó
        System.arraycopy(elementData, index, elementData, index + 1,
                size - index);
        elementData[index] = element;
        size++;
    }

    /**
     * Xóa phần tử tại vị trí đã chỉ định trong list. Dịch mọi phần tử phía sau sang trái (giảm index của chúng đi một).
     */
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index + 1, elementData, index,
                    numMoved);
        elementData[--size] = null; // clear to let GC do its work
        // Phần tử bị xóa khỏi list
        return oldValue;
    }

    /**
     * Xóa lần xuất hiện đầu tiên của phần tử đã chỉ định khỏi list (nếu có). Nếu list không chứa phần tử này thì không thay đổi.
     * Trả về true nếu list này chứa phần tử đã chỉ định
     */
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

    /*
     * Đây là private method remove, bỏ qua việc kiểm tra giới hạn và không trả về giá trị bị xóa.
     */
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index + 1, elementData, index,
                    numMoved);
        elementData[--size] = null; // Sau khi xóa phần tử, set vị trí này thành null để garbage collector (GC) có thể thu hồi phần tử.
    }

    /**
     * Xóa tất cả phần tử khỏi list.
     */
    public void clear() {
        modCount++;

        // Set giá trị của tất cả phần tử trong array thành null
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }

    /**
     * Append toàn bộ phần tử trong collection đã chỉ định vào cuối list này theo thứ tự do Iterator của collection đã chỉ định trả về.
     */
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * Insert toàn bộ phần tử trong collection đã chỉ định vào list này, bắt đầu từ vị trí đã chỉ định.
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount

        int numMoved = size - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                    numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * Xóa khỏi list này tất cả phần tử có index từ fromIndex (bao gồm fromIndex) đến trước toIndex.
     * Dịch mọi phần tử phía sau sang trái (giảm index của chúng).
     */
    protected void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = size - toIndex;
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                numMoved);

        // clear to let GC do its work
        int newSize = size - (toIndex - fromIndex);
        for (int i = newSize; i < size; i++) {
            elementData[i] = null;
        }
        size = newSize;
    }

    /**
     * Kiểm tra index đã cho có nằm trong phạm vi hay không.
     */
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * Một phiên bản của rangeCheck được add và addAll sử dụng
     */
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * Trả về thông tin chi tiết của IndexOutOfBoundsException
     */
    private String outOfBoundsMsg(int index) {
        return "Index: " + index + ", Size: " + size;
    }

    /**
     * Xóa khỏi list này tất cả phần tử cũng có trong collection đã chỉ định.
     */
    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        // Trả về true nếu list này bị sửa đổi
        return batchRemove(c, false);
    }

    /**
     * Chỉ giữ lại những phần tử trong list này có trong collection đã chỉ định.
     * Nói cách khác, xóa khỏi list này mọi phần tử không có trong collection đã chỉ định.
     */
    public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, true);
    }


    /**
     * Trả về list iterator của các phần tử trong list, bắt đầu từ vị trí đã chỉ định và theo đúng thứ tự.
     * Index đã chỉ định biểu thị phần tử đầu tiên mà lần gọi next ban đầu sẽ trả về. Lần gọi previous ban đầu sẽ trả về phần tử có index nhỏ hơn index đã chỉ định 1 đơn vị.
     * List iterator được trả về là fail-fast.
     */
    public ListIterator<E> listIterator(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException("Index: " + index);
        return new ListItr(index);
    }

    /**
     * Trả về list iterator của list (theo đúng thứ tự).
     * List iterator được trả về là fail-fast.
     */
    public ListIterator<E> listIterator() {
        return new ListItr(0);
    }

    /**
     * Trả về iterator của các phần tử trong list này theo đúng thứ tự.
     * Iterator được trả về là fail-fast.
     */
    public Iterator<E> iterator() {
        return new Itr();
    }
```

## Phân tích cơ chế mở rộng của ArrayList

### Bắt đầu từ constructor của ArrayList

ArrayList có ba cách để khởi tạo, source code của constructor như sau (JDK8):

```java
/**
 * Kích thước capacity mặc định ban đầu
 */
private static final int DEFAULT_CAPACITY = 10;

private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/**
 * Constructor mặc định, tạo một empty list; capacity sẽ là 10 khi thêm phần tử đầu tiên (constructor không tham số)
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

/**
 * Constructor có tham số capacity ban đầu. (User tự chỉ định capacity)
 */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {// capacity ban đầu lớn hơn 0
        // Tạo array có kích thước initialCapacity
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {// capacity ban đầu bằng 0
        // Tạo empty array
        this.elementData = EMPTY_ELEMENTDATA;
    } else {// capacity ban đầu nhỏ hơn 0, throw exception
        throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);
    }
}


/**
 * Tạo list chứa các phần tử của collection đã chỉ định theo thứ tự iterator của collection trả về
 * Nếu collection đã chỉ định là null, throws NullPointerException.
 */
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

Người đọc tinh ý chắc chắn sẽ nhận ra: **khi tạo `ArrayList` bằng constructor không tham số, thực tế giá trị được khởi tạo là một empty array. Chỉ khi thực sự thêm phần tử vào array thì capacity mới được cấp phát. Tức là khi thêm phần tử đầu tiên vào array, capacity của array mới được mở rộng thành 10.** Phần này sẽ được đề cập khi phân tích cơ chế mở rộng của `ArrayList` bên dưới!

> Bổ sung: khi `new` object `ArrayList` bằng constructor không tham số trong JDK6, array `Object[]` có length 10 được tạo trực tiếp trong `elementData`.

### Phân tích từng bước cơ chế mở rộng của ArrayList

Ở đây lấy `ArrayList` được tạo bằng constructor không tham số làm ví dụ để phân tích.

#### Method add

```java
/**
* Append phần tử đã chỉ định vào cuối list này.
*/
public boolean add(E e) {
    // Trước khi thêm phần tử, trước tiên gọi method ensureCapacityInternal
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // Ở đây có thể thấy bản chất việc thêm phần tử của ArrayList tương đương với việc gán giá trị cho array
    elementData[size++] = e;
    return true;
}
```

**Lưu ý**: JDK11 đã loại bỏ hai method `ensureCapacityInternal()` và `ensureExplicitCapacity()`

Source code của method `ensureCapacityInternal` như sau:

```java
// Tính capacity cần thiết theo minCapacity đã cho và các phần tử hiện tại của array.
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    // Nếu array hiện tại là empty array (trạng thái ban đầu), trả về giá trị lớn hơn giữa capacity mặc định và capacity tối thiểu làm capacity cần thiết
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    // Nếu không thì trả về trực tiếp capacity tối thiểu
    return minCapacity;
}

// Đảm bảo capacity bên trong đạt capacity tối thiểu đã chỉ định.
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
```

Method `ensureCapacityInternal` rất đơn giản, bên trong trực tiếp gọi method `ensureExplicitCapacity`:

```java
// Kiểm tra có cần mở rộng capacity hay không
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    // Kiểm tra capacity hiện tại của array có đủ để lưu minCapacity phần tử hay không
    if (minCapacity - elementData.length > 0)
        // Gọi method grow để mở rộng capacity
        grow(minCapacity);
}
```

Hãy cùng phân tích kỹ:

- Khi `add` phần tử thứ 1 vào `ArrayList`, `elementData.length` bằng 0 (vì list vẫn rỗng). Do thực thi method `ensureCapacityInternal()`, lúc này `minCapacity` bằng 10. Khi đó điều kiện `minCapacity - elementData.length > 0` đúng, nên sẽ đi vào method `grow(minCapacity)`.
- Khi `add` phần tử thứ 2, `minCapacity` bằng 2, lúc này `elementData.length` (capacity) sau khi thêm phần tử đầu tiên đã được mở rộng thành `10`. Khi đó điều kiện `minCapacity - elementData.length > 0` không đúng, nên không đi vào (thực thi) method `grow(minCapacity)`.
- Khi thêm phần tử thứ 3, 4... đến phần tử thứ 10, method grow vẫn không được thực thi, capacity của array đều là 10.

Cho đến khi thêm phần tử thứ 11, `minCapacity` (bằng 11) lớn hơn `elementData.length` (bằng 10). Khi đó đi vào method `grow` để mở rộng capacity.

#### Method grow

```java
/**
 * Kích thước array tối đa có thể allocate
 */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

/**
 * Method cốt lõi để mở rộng capacity của ArrayList.
 */
private void grow(int minCapacity) {
    // oldCapacity là capacity cũ, newCapacity là capacity mới
    int oldCapacity = elementData.length;
    // Dịch phải oldCapacity một bit, hiệu quả tương đương oldCapacity /2,
    // ta biết tốc độ của phép toán bit nhanh hơn rất nhiều so với phép chia, kết quả của toàn bộ biểu thức là cập nhật capacity mới thành 1,5 lần capacity cũ,
    int newCapacity = oldCapacity + (oldCapacity >> 1);

    // Sau đó kiểm tra capacity mới có lớn hơn capacity tối thiểu cần thiết hay không; nếu vẫn nhỏ hơn capacity tối thiểu thì lấy capacity tối thiểu làm capacity mới của array,
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;

    // Nếu capacity mới lớn hơn MAX_ARRAY_SIZE, đi vào (thực thi) method `hugeCapacity()` để so sánh minCapacity với MAX_ARRAY_SIZE,
    // nếu minCapacity lớn hơn capacity tối đa thì capacity mới là `Integer.MAX_VALUE`, nếu không thì capacity mới là MAX_ARRAY_SIZE, tức `Integer.MAX_VALUE - 8`.
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);

    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

**`int newCapacity = oldCapacity + (oldCapacity >> 1)`, vì vậy sau mỗi lần mở rộng capacity, capacity của ArrayList sẽ trở thành khoảng 1,5 lần capacity cũ (nếu oldCapacity là số chẵn thì đúng 1,5 lần, nếu là số lẻ thì xấp xỉ 1,5 lần)!** Kết quả khác nhau tùy oldCapacity là số chẵn hay lẻ, ví dụ: 10+10/2 = 15, 33+33/2=49. Nếu là số lẻ thì phần thập phân sẽ bị bỏ.

> `>>` (shift operator): `>>1` dịch phải một bit tương đương với chia cho 2, dịch phải n bit tương đương với chia cho 2 lũy thừa n. Ở đây oldCapacity rõ ràng được dịch phải 1 bit nên tương đương với oldCapacity /2. Với phép tính nhị phân trên data lớn, shift operator nhanh hơn nhiều toán tử thông thường vì chương trình chỉ cần dịch chuyển mà không cần thực hiện phép chia, từ đó cải thiện performance và tiết kiệm resource.

**Hãy tiếp tục tìm hiểu method `grow()` thông qua ví dụ:**

- Khi `add` phần tử thứ 1, `oldCapacity` bằng 0. Sau khi so sánh, điều kiện if thứ nhất đúng, `newCapacity = minCapacity` (bằng 10). Tuy nhiên, điều kiện if thứ hai không đúng, tức `newCapacity` không lớn hơn `MAX_ARRAY_SIZE`, nên không đi vào method `hugeCapacity`. Capacity của array là 10, method `add` return true, size tăng lên 1.
- Khi `add` phần tử thứ 11 và đi vào method `grow`, `newCapacity` bằng 15, lớn hơn `minCapacity` (bằng 11), nên điều kiện if thứ nhất không đúng. Capacity mới không vượt quá giới hạn tối đa của array, nên không đi vào method `hugeCapacity`. Capacity của array mở rộng thành 15, method add return true, size tăng lên 11.
- Tương tự...

**Bổ sung một điểm khá quan trọng nhưng dễ bị bỏ qua:**

- Trong Java, thuộc tính `length` dùng cho array. Ví dụ, khi khai báo một array và muốn biết length của array, ta dùng thuộc tính length.
- Method `length()` dùng cho string. Khi muốn biết length của string, ta dùng method `length()`.
- Method `size()` dùng cho generic collection. Khi muốn biết collection có bao nhiêu phần tử, hãy gọi method này để kiểm tra!

#### Method hugeCapacity()

Từ source code method `grow()` ở trên, ta biết: nếu capacity mới lớn hơn `MAX_ARRAY_SIZE`, method `hugeCapacity()` được gọi để so sánh `minCapacity` với `MAX_ARRAY_SIZE`. Nếu `minCapacity` lớn hơn giới hạn capacity tối đa thì capacity mới là `Integer.MAX_VALUE`, nếu không thì capacity mới là `MAX_ARRAY_SIZE`, tức `Integer.MAX_VALUE - 8`.

```java
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    // So sánh minCapacity với MAX_ARRAY_SIZE
    // Nếu minCapacity lớn, dùng Integer.MAX_VALUE làm kích thước array mới
    // Nếu MAX_ARRAY_SIZE lớn, dùng MAX_ARRAY_SIZE làm kích thước array mới
    // MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

### Method `System.arraycopy()` và `Arrays.copyOf()`

Khi đọc source code, ta sẽ phát hiện `ArrayList` gọi hai method này rất nhiều lần. Ví dụ: thao tác mở rộng capacity được đề cập ở trên, cùng các method `add(int index, E element)`, `toArray()`... đều dùng chúng!

#### Method `System.arraycopy()`

Source code:

```java
    // Ta phát hiện arraycopy là một native method, tiếp theo hãy giải thích ý nghĩa cụ thể của từng tham số
    /**
    *   Copy array
    * @param src array nguồn
    * @param srcPos vị trí bắt đầu trong array nguồn
    * @param dest array đích
    * @param destPos vị trí bắt đầu trong array đích
    * @param length số lượng phần tử array cần copy
    */
    public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```

Ví dụ sử dụng:

```java
    /**
     * Insert phần tử đã chỉ định tại vị trí đã chỉ định trong list này.
     * Trước tiên gọi rangeCheckForAdd để kiểm tra giới hạn của index; sau đó gọi method ensureCapacityInternal để đảm bảo capacity đủ lớn;
     * tiếp theo dịch mọi phần tử từ index trở đi về sau một vị trí; insert element vào vị trí index; cuối cùng tăng size lên 1.
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        // Method arraycopy() thực hiện việc array tự copy chính nó
        // elementData: array nguồn; index: vị trí bắt đầu trong array nguồn; elementData: array đích; index + 1: vị trí bắt đầu trong array đích; size - index: số lượng phần tử array cần copy;
        System.arraycopy(elementData, index, elementData, index + 1, size - index);
        elementData[index] = element;
        size++;
    }
```

Hãy viết một method đơn giản để test:

```java
public class ArraycopyTest {

  public static void main(String[] args) {
    // TODO Auto-generated method stub
    int[] a = new int[10];
    a[0] = 0;
    a[1] = 1;
    a[2] = 2;
    a[3] = 3;
    System.arraycopy(a, 2, a, 3, 3);
    a[2]=99;
    for (int i = 0; i < a.length; i++) {
      System.out.print(a[i] + " ");
    }
  }

}
```

Kết quả:

```plain
0 1 99 2 3 0 0 0 0 0
```

#### Method `Arrays.copyOf()`

Source code:

```java
    public static int[] copyOf(int[] original, int newLength) {
      // Cấp phát một array mới
        int[] copy = new int[newLength];
  // Gọi System.arraycopy để copy data trong array nguồn và trả về array mới
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```

Ví dụ sử dụng:

```java
   /**
     * Trả về một array chứa toàn bộ phần tử trong list này theo đúng thứ tự (từ phần tử đầu tiên đến phần tử cuối cùng); runtime type của array được trả về là runtime type của array đã chỉ định.
     */
    public Object[] toArray() {
    // elementData: array cần copy; size: length cần copy
        return Arrays.copyOf(elementData, size);
    }
```

Theo tôi, `Arrays.copyOf()` chủ yếu được dùng để mở rộng array hiện có. Code test như sau:

```java
public class ArrayscopyOfTest {

  public static void main(String[] args) {
    int[] a = new int[3];
    a[0] = 0;
    a[1] = 1;
    a[2] = 2;
    int[] b = Arrays.copyOf(a, 10);
    System.out.println("b.length"+b.length);
  }
}
```

Kết quả:

```plain
10
```

#### Mối liên hệ và khác biệt giữa hai method

**Mối liên hệ:**

Quan sát source code của hai method, ta có thể thấy bên trong `copyOf()` thực tế gọi `System.arraycopy()`.

**Khác biệt:**

`arraycopy()` cần array đích, copy array ban đầu vào array do bạn tự định nghĩa hoặc vào chính array ban đầu. Đồng thời có thể chọn điểm bắt đầu và length cần copy cũng như vị trí đặt trong array mới. `copyOf()` tự động tạo một array mới bên trong system và trả về array đó.

### Method `ensureCapacity`

Trong source code `ArrayList` có một method `ensureCapacity`, có lẽ bạn đã để ý đến nó. Method này chưa từng được gọi bên trong `ArrayList`, nên rõ ràng nó được cung cấp để user gọi. Vậy method này có tác dụng gì?

```java
    /**
     * Nếu cần, tăng capacity của instance ArrayList này để đảm bảo nó có thể chứa ít nhất số phần tử do tham số minimum capacity chỉ định.
     *
     * @param   minCapacity   capacity tối thiểu cần thiết
     */
    public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }

```

Về mặt lý thuyết, tốt nhất nên dùng method `ensureCapacity` trước khi thêm nhiều phần tử vào `ArrayList` để giảm số lần incremental reallocation.

Hãy dùng code bên dưới để test thực tế hiệu quả của method này:

```java
public class EnsureCapacityTest {
  public static void main(String[] args) {
    ArrayList<Object> list = new ArrayList<Object>();
    final int N = 10000000;
    long startTime = System.currentTimeMillis();
    for (int i = 0; i < N; i++) {
      list.add(i);
    }
    long endTime = System.currentTimeMillis();
    System.out.println("Trước khi dùng method ensureCapacity: "+(endTime - startTime));

  }
}
```

Kết quả chạy:

```plain
Trước khi dùng method ensureCapacity: 2158
```

```java
public class EnsureCapacityTest {
    public static void main(String[] args) {
        ArrayList<Object> list = new ArrayList<Object>();
        final int N = 10000000;
        long startTime1 = System.currentTimeMillis();
        list.ensureCapacity(N);
        for (int i = 0; i < N; i++) {
            list.add(i);
        }
        long endTime1 = System.currentTimeMillis();
        System.out.println("Sau khi dùng method ensureCapacity: "+(endTime1 - startTime1));
    }
}
```

Kết quả chạy:

```plain
Sau khi dùng method ensureCapacity: 1773
```

Từ kết quả chạy, có thể thấy dùng method `ensureCapacity` trước khi thêm nhiều phần tử vào `ArrayList` có thể cải thiện performance. Tuy nhiên, chênh lệch performance này gần như không đáng kể. Hơn nữa, trong project thực tế hầu như không cần thêm nhiều phần tử như vậy vào `ArrayList`.

<!-- @include: @article-footer.snippet.md -->
