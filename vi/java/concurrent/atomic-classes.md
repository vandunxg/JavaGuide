---
title: Tổng hợp Atomic atomic classes
description: "Giải thích chi tiết Java atomic classes: tổng hợp toàn diện hệ thống Atomic atomic classes trong gói JUC, các class thường dùng như AtomicInteger/AtomicLong/AtomicReference, triển khai thread-safe dựa trên CAS, trường hợp sử dụng và ưu thế về performance."
category: Java
tag:
  - Java Concurrency
head:
  - - meta
    - name: keywords
      content: Atomic atomic classes,AtomicInteger,AtomicLong,AtomicReference,CAS atomic operation,gói JUC concurrency,cách sử dụng atomic classes
---

## Giới thiệu về Atomic atomic classes

`Atomic` dịch sang tiếng Việt có nghĩa là “nguyên tử”. Trong hóa học, nguyên tử là đơn vị nhỏ nhất cấu tạo nên vật chất và không thể bị phân chia trong phản ứng hóa học. Trong lập trình, `Atomic` chỉ một operation có tính atomic, tức operation đó không thể bị phân chia hoặc ngắt giữa chừng. Ngay cả khi được thực thi đồng thời bởi nhiều thread, operation đó hoặc hoàn tất toàn bộ, hoặc không thực thi; các thread khác sẽ không nhìn thấy trạng thái hoàn tất một phần.

Nói đơn giản, atomic class là class có đặc trưng operation atomic.

Các atomic class `Atomic` trong package `java.util.concurrent.atomic` cung cấp một cách thread-safe để thao tác với một biến đơn lẻ.

Các class `Atomic` dựa trên optimistic lock CAS (Compare-And-Swap, so sánh và trao đổi) để bảo đảm tính atomic cho method, không cần sử dụng cơ chế lock truyền thống (chẳng hạn block `synchronized` hoặc `ReentrantLock`).

Bài viết này chỉ giới thiệu khái niệm về Atomic atomic classes; để biết nguyên lý triển khai cụ thể, bạn có thể đọc bài viết [Giải thích chi tiết về CAS](./cas.md) do tác giả viết.

![Tổng quan về JUC atomic classes](https://oss.javaguide.cn/github/javaguide/java/JUC%E5%8E%9F%E5%AD%90%E7%B1%BB%E6%A6%82%E8%A7%88.png)

Dựa trên kiểu dữ liệu được thao tác, có thể chia các atomic class trong gói JUC thành 4 nhóm:

**1. Basic types**

Cập nhật basic type theo cách atomic

- `AtomicInteger`: atomic class kiểu integer
- `AtomicLong`: atomic class kiểu long
- `AtomicBoolean`: atomic class kiểu boolean

**2. Array types**

Cập nhật một phần tử trong array theo cách atomic

- `AtomicIntegerArray`: atomic class cho array kiểu integer
- `AtomicLongArray`: atomic class cho array kiểu long
- `AtomicReferenceArray`: atomic class cho array kiểu reference

**3. Reference types**

- `AtomicReference`: atomic class kiểu reference
- `AtomicMarkableReference`: cập nhật atomic một reference có mark. Class này liên kết mark boolean với reference, có thể phát hiện sự thay đổi giữa hai trạng thái do business quy ước; tuy nhiên, mark một bit không thể ghi lại số lần thay đổi version tùy ý.
- `AtomicStampedReference`: cập nhật atomic một reference có version number. Class này liên kết một giá trị integer với reference, có thể dùng để cập nhật atomic data và version number của data, đồng thời giải quyết vấn đề ABA có thể xảy ra khi dùng CAS để cập nhật atomic.

So với nó, `AtomicStampedReference` dùng version number kiểu integer, phù hợp hơn để phát hiện reference đã trải qua nhiều lần thay đổi giữa hai lần đọc hay chưa.

**4. Atomic classes cho việc cập nhật thuộc tính của object**

- `AtomicIntegerFieldUpdater`: updater để cập nhật atomic field kiểu integer
- `AtomicLongFieldUpdater`: updater để cập nhật atomic field kiểu long
- `AtomicReferenceFieldUpdater`: updater để cập nhật atomic field kiểu reference

## Atomic class cho basic type

Cập nhật basic type theo cách atomic

- `AtomicInteger`: atomic class kiểu integer
- `AtomicLong`: atomic class kiểu long
- `AtomicBoolean`: atomic class kiểu boolean

Ba class trên cung cấp các method gần như giống nhau, vì vậy ở đây dùng `AtomicInteger` làm ví dụ để giới thiệu.

**Các method thường dùng của class `AtomicInteger`**:

```java
public final int get() // Lấy giá trị hiện tại
public final int getAndSet(int newValue)// Lấy giá trị hiện tại và đặt giá trị mới
public final int getAndIncrement()// Lấy giá trị hiện tại và tăng dần
public final int getAndDecrement() // Lấy giá trị hiện tại và giảm dần
public final int getAndAdd(int delta) // Lấy giá trị hiện tại và cộng thêm giá trị dự kiến
boolean compareAndSet(int expect, int update) // Nếu giá trị đầu vào bằng giá trị dự kiến, đặt giá trị đó thành giá trị đầu vào (update) theo cách atomic
public final void lazySet(int newValue)// Cuối cùng đặt thành newValue; lazySet cung cấp semantic yếu hơn method set, có thể khiến các thread khác vẫn đọc được giá trị cũ trong một khoảng thời gian ngắn sau đó, nhưng có thể hiệu quả hơn.
```

**Ví dụ sử dụng class `AtomicInteger`**:

```java
// Khởi tạo object AtomicInteger với giá trị ban đầu là 0
AtomicInteger atomicInt = new AtomicInteger(0);

// Dùng method getAndSet để lấy giá trị hiện tại và đặt giá trị mới là 3
int tempValue = atomicInt.getAndSet(3);
System.out.println("tempValue: " + tempValue + "; atomicInt: " + atomicInt);

// Dùng method getAndIncrement để lấy giá trị hiện tại và tăng 1
tempValue = atomicInt.getAndIncrement();
System.out.println("tempValue: " + tempValue + "; atomicInt: " + atomicInt);

// Dùng method getAndAdd để lấy giá trị hiện tại và tăng thêm giá trị chỉ định là 5
tempValue = atomicInt.getAndAdd(5);
System.out.println("tempValue: " + tempValue + "; atomicInt: " + atomicInt);

// Dùng method compareAndSet để cập nhật có điều kiện theo cách atomic, giá trị kỳ vọng là 9, giá trị cập nhật là 10
boolean updateSuccess = atomicInt.compareAndSet(9, 10);
System.out.println("Update Success: " + updateSuccess + "; atomicInt: " + atomicInt);

// Lấy giá trị hiện tại
int currentValue = atomicInt.get();
System.out.println("Current value: " + currentValue);

// Dùng method lazySet để đặt giá trị mới là 15
atomicInt.lazySet(15);
System.out.println("After lazySet, atomicInt: " + atomicInt);
```

Output:

```java
tempValue: 0; atomicInt: 3
tempValue: 3; atomicInt: 4
tempValue: 4; atomicInt: 9
Update Success: true; atomicInt: 10
Current value: 10
After lazySet, atomicInt: 15
```

## Atomic class cho array type

Cập nhật một phần tử trong array theo cách atomic

- `AtomicIntegerArray`: atomic class cho array kiểu integer
- `AtomicLongArray`: atomic class cho array kiểu long
- `AtomicReferenceArray`: atomic class cho array kiểu reference

Ba class trên cung cấp các method gần như giống nhau, vì vậy ở đây dùng `AtomicIntegerArray` làm ví dụ để giới thiệu.

**Các method thường dùng của class `AtomicIntegerArray`**:

```java
public final int get(int i) // Lấy giá trị phần tử tại vị trí index=i
public final int getAndSet(int i, int newValue)// Trả về giá trị hiện tại tại vị trí index=i và đặt nó thành giá trị mới: newValue
public final int getAndIncrement(int i)// Lấy giá trị phần tử tại vị trí index=i và tăng phần tử tại vị trí đó
public final int getAndDecrement(int i) // Lấy giá trị phần tử tại vị trí index=i và giảm phần tử tại vị trí đó
public final int getAndAdd(int i, int delta) // Lấy giá trị phần tử tại vị trí index=i và cộng thêm giá trị dự kiến
boolean compareAndSet(int i, int expect, int update) // Nếu giá trị đầu vào bằng giá trị dự kiến, đặt giá trị phần tử tại vị trí index=i thành giá trị đầu vào (update) theo cách atomic
public final void lazySet(int i, int newValue)// Cuối cùng đặt phần tử tại vị trí index=i thành newValue; sau khi dùng lazySet, các thread khác có thể vẫn đọc được giá trị cũ trong một khoảng thời gian ngắn sau đó.
```

**Ví dụ sử dụng class `AtomicIntegerArray`**:

```java
int[] nums = {1, 2, 3, 4, 5, 6};
// Tạo AtomicIntegerArray
AtomicIntegerArray atomicArray = new AtomicIntegerArray(nums);

// In giá trị ban đầu trong AtomicIntegerArray
System.out.println("Initial values in AtomicIntegerArray:");
for (int j = 0; j < nums.length; j++) {
    System.out.print("Index " + j + ": " + atomicArray.get(j) + " ");
}

// Dùng method getAndSet để đặt giá trị tại index 0 thành 2 và trả về giá trị cũ
int tempValue = atomicArray.getAndSet(0, 2);
System.out.println("\nAfter getAndSet(0, 2):");
System.out.println("Returned value: " + tempValue);
for (int j = 0; j < atomicArray.length(); j++) {
    System.out.print("Index " + j + ": " + atomicArray.get(j) + " ");
}

// Dùng method getAndIncrement để tăng giá trị tại index 0 lên 1 và trả về giá trị cũ
tempValue = atomicArray.getAndIncrement(0);
System.out.println("\nAfter getAndIncrement(0):");
System.out.println("Returned value: " + tempValue);
for (int j = 0; j < atomicArray.length(); j++) {
    System.out.print("Index " + j + ": " + atomicArray.get(j) + " ");
}

// Dùng method getAndAdd để tăng giá trị tại index 0 thêm 5 và trả về giá trị cũ
tempValue = atomicArray.getAndAdd(0, 5);
System.out.println("\nAfter getAndAdd(0, 5):");
System.out.println("Returned value: " + tempValue);
for (int j = 0; j < atomicArray.length(); j++) {
    System.out.print("Index " + j + ": " + atomicArray.get(j) + " ");
}
```

Output:

```plain
Initial values in AtomicIntegerArray:
Index 0: 1 Index 1: 2 Index 2: 3 Index 3: 4 Index 4: 5 Index 5: 6
After getAndSet(0, 2):
Returned value: 1
Index 0: 2 Index 1: 2 Index 2: 3 Index 3: 4 Index 4: 5 Index 5: 6
After getAndIncrement(0):
Returned value: 2
Index 0: 3 Index 1: 2 Index 2: 3 Index 3: 4 Index 4: 5 Index 5: 6
After getAndAdd(0, 5):
Returned value: 3
Index 0: 8 Index 1: 2 Index 2: 3 Index 3: 4 Index 4: 5 Index 5: 6
```

## Atomic class cho reference type

Atomic class cho basic type chỉ có thể cập nhật một biến. Nếu cần cập nhật atomic nhiều biến, cần dùng atomic class cho reference type.

- `AtomicReference`: atomic class kiểu reference
- `AtomicStampedReference`: cập nhật atomic một reference có version number. Class này liên kết một giá trị integer với reference, có thể dùng để cập nhật atomic data và version number của data, đồng thời giải quyết vấn đề ABA có thể xảy ra khi dùng CAS để cập nhật atomic.
- `AtomicMarkableReference`: cập nhật atomic một reference có mark. Class này liên kết mark boolean với reference, ~~cũng có thể giải quyết vấn đề ABA có thể xảy ra khi dùng CAS để cập nhật atomic.~~

Ba class trên cung cấp các method gần như giống nhau, vì vậy ở đây dùng `AtomicReference` làm ví dụ để giới thiệu.

**Ví dụ sử dụng class `AtomicReference`**:

```java
// Class Person
class Person {
    private String name;
    private int age;
    // Bỏ qua getter/setter và toString
}


// Tạo object AtomicReference và đặt giá trị ban đầu
AtomicReference<Person> ar = new AtomicReference<>(new Person("SnailClimb", 22));

// In giá trị ban đầu
System.out.println("Initial Person: " + ar.get().toString());

// Cập nhật giá trị
Person updatePerson = new Person("Daisy", 20);
ar.compareAndSet(ar.get(), updatePerson);

// In giá trị sau khi cập nhật
System.out.println("Updated Person: " + ar.get().toString());

// Thử cập nhật lần nữa
Person anotherUpdatePerson = new Person("John", 30);
boolean isUpdated = ar.compareAndSet(updatePerson, anotherUpdatePerson);

// In việc cập nhật có thành công hay không và giá trị cuối cùng
System.out.println("Second Update Success: " + isUpdated);
System.out.println("Final Person: " + ar.get().toString());
```

Output:

```plain
Initial Person: Person{name='SnailClimb', age=22}
Updated Person: Person{name='Daisy', age=20}
Second Update Success: true
Final Person: Person{name='John', age=30}
```

**Ví dụ sử dụng class `AtomicStampedReference`**:

```java
// Tạo một object AtomicStampedReference, giá trị ban đầu là "SnailClimb", version number ban đầu là 1
AtomicStampedReference<String> asr = new AtomicStampedReference<>("SnailClimb", 1);

// In giá trị ban đầu và version number
int[] initialStamp = new int[1];
String initialRef = asr.get(initialStamp);
System.out.println("Initial Reference: " + initialRef + ", Initial Stamp: " + initialStamp[0]);

// Cập nhật giá trị và version number
int oldStamp = initialStamp[0];
String oldRef = initialRef;
String newRef = "Daisy";
int newStamp = oldStamp + 1;

boolean isUpdated = asr.compareAndSet(oldRef, newRef, oldStamp, newStamp);
System.out.println("Update Success: " + isUpdated);

// In giá trị và version number sau khi cập nhật
int[] updatedStamp = new int[1];
String updatedRef = asr.get(updatedStamp);
System.out.println("Updated Reference: " + updatedRef + ", Updated Stamp: " + updatedStamp[0]);

// Thử cập nhật bằng version number sai
boolean isUpdatedWithWrongStamp = asr.compareAndSet(newRef, "John", oldStamp, newStamp + 1);
System.out.println("Update with Wrong Stamp Success: " + isUpdatedWithWrongStamp);

// In giá trị và version number cuối cùng
int[] finalStamp = new int[1];
String finalRef = asr.get(finalStamp);
System.out.println("Final Reference: " + finalRef + ", Final Stamp: " + finalStamp[0]);
```

Output như sau:

```plain
Initial Reference: SnailClimb, Initial Stamp: 1
Update Success: true
Updated Reference: Daisy, Updated Stamp: 2
Update with Wrong Stamp Success: false
Final Reference: Daisy, Final Stamp: 2
```

**Ví dụ sử dụng class `AtomicMarkableReference`**:

```java
// Tạo một object AtomicMarkableReference, giá trị ban đầu là "SnailClimb", mark ban đầu là false
AtomicMarkableReference<String> amr = new AtomicMarkableReference<>("SnailClimb", false);

// In giá trị ban đầu và mark
boolean[] initialMark = new boolean[1];
String initialRef = amr.get(initialMark);
System.out.println("Initial Reference: " + initialRef + ", Initial Mark: " + initialMark[0]);

// Cập nhật giá trị và mark
String oldRef = initialRef;
String newRef = "Daisy";
boolean oldMark = initialMark[0];
boolean newMark = true;

boolean isUpdated = amr.compareAndSet(oldRef, newRef, oldMark, newMark);
System.out.println("Update Success: " + isUpdated);

// In giá trị và mark sau khi cập nhật
boolean[] updatedMark = new boolean[1];
String updatedRef = amr.get(updatedMark);
System.out.println("Updated Reference: " + updatedRef + ", Updated Mark: " + updatedMark[0]);

// Thử cập nhật bằng mark sai
boolean isUpdatedWithWrongMark = amr.compareAndSet(newRef, "John", oldMark, !newMark);
System.out.println("Update with Wrong Mark Success: " + isUpdatedWithWrongMark);

// In giá trị và mark cuối cùng
boolean[] finalMark = new boolean[1];
String finalRef = amr.get(finalMark);
System.out.println("Final Reference: " + finalRef + ", Final Mark: " + finalMark[0]);
```

Output như sau:

```plain
Initial Reference: SnailClimb, Initial Mark: false
Update Success: true
Updated Reference: Daisy, Updated Mark: true
Update with Wrong Mark Success: false
Final Reference: Daisy, Final Mark: true
```

## Atomic class cho việc cập nhật thuộc tính của object

Khi cần cập nhật atomic một field trong một class, cần dùng atomic class cho việc cập nhật thuộc tính của object.

- `AtomicIntegerFieldUpdater`: updater để cập nhật atomic field kiểu integer
- `AtomicLongFieldUpdater`: updater để cập nhật atomic field kiểu long
- `AtomicReferenceFieldUpdater`: updater để cập nhật atomic field kiểu reference

Để cập nhật atomic thuộc tính của object cần hai bước. Bước đầu tiên, vì các atomic class cho việc cập nhật thuộc tính của object đều là abstract class, nên mỗi lần sử dụng phải dùng static method `newUpdater()` để tạo updater, đồng thời chỉ định class và thuộc tính cần cập nhật. Bước thứ hai, target field phải được bổ sung `volatile` và khớp với type của updater: lần lượt là `int`, `long` hoặc reference type; đồng thời không được là field `static` hoặc `final`.

Ba class trên cung cấp các method gần như giống nhau, vì vậy ở đây dùng `AtomicIntegerFieldUpdater` làm ví dụ để giới thiệu.

**Ví dụ sử dụng class `AtomicIntegerFieldUpdater`**:

```java
// Class Person
class Person {
    private String name;
    // Để sử dụng AtomicIntegerFieldUpdater, field phải là volatile int
    volatile int age;
    // Bỏ qua getter/setter và toString
}

// Tạo object AtomicIntegerFieldUpdater
AtomicIntegerFieldUpdater<Person> ageUpdater = AtomicIntegerFieldUpdater.newUpdater(Person.class, "age");

// Tạo object Person
Person person = new Person("SnailClimb", 22);

// In giá trị ban đầu
System.out.println("Initial Person: " + person);

// Cập nhật field age
ageUpdater.incrementAndGet(person); // Tăng dần
System.out.println("After Increment: " + person);

ageUpdater.addAndGet(person, 5); // Tăng thêm 5
System.out.println("After Adding 5: " + person);

ageUpdater.compareAndSet(person, 28, 30); // Nếu giá trị hiện tại là 28 thì đặt thành 30
System.out.println("After Compare and Set (28 to 30): " + person);

// Thử cập nhật bằng giá trị so sánh sai
boolean isUpdated = ageUpdater.compareAndSet(person, 28, 35); // Lần này phải thất bại
System.out.println("Compare and Set (28 to 35) Success: " + isUpdated);
System.out.println("Final Person: " + person);
```

Output:

```plain
Initial Person: Name: SnailClimb, Age: 22
After Increment: Name: SnailClimb, Age: 23
After Adding 5: Name: SnailClimb, Age: 28
After Compare and Set (28 to 30): Name: SnailClimb, Age: 30
Compare and Set (28 to 35) Success: false
Final Person: Name: SnailClimb, Age: 30
```

## Tài liệu tham khảo

- 《Nghệ thuật lập trình concurrent trong Java》

<!-- @include: @article-footer.snippet.md -->
