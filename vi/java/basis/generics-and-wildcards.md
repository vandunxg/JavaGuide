---
title: "Giải thích chi tiết về Generics & Wildcards"
description: "Phân tích toàn diện Java Generics và Wildcards: hiểu sâu cơ chế type erasure, cách dùng upper/lower bounded wildcards và nguyên tắc PECS, nắm vững kỹ năng cốt lõi của generic programming."
category: Java
tag:
  - Java Basics
head:
  - - meta
    - name: keywords
      content: "Java Generics,Wildcards,type erasure,generic bounds,nguyên tắc PECS,generic method,upper/lower bounded wildcards,generic interface"
---

## Generics

### Generics là gì? Có tác dụng gì?

**Java Generics** là một tính năng mới được giới thiệu trong JDK 5. Việc sử dụng tham số generic có thể cải thiện khả năng đọc và tính ổn định của code. **Nếu không có chú thích đặc biệt, các hành vi dưới đây tuân theo Java 8.**

Compiler có thể kiểm tra tham số generic, đồng thời tham số generic cho phép chỉ định kiểu object được truyền vào. Ví dụ, dòng code `ArrayList<Person> persons = new ArrayList<Person>()` chỉ rõ rằng `ArrayList` đó chỉ có thể nhận object kiểu `Person`; nếu truyền kiểu khác vào sẽ báo lỗi (từ JDK 7 có thể viết `new ArrayList<>()`, compiler sẽ suy luận tham số kiểu).

```java
ArrayList<E> extends AbstractList<E>
```

Ngoài ra, `List` raw có kiểu trả về là `Object`, cần chuyển kiểu thủ công mới có thể sử dụng; sau khi dùng Generics, compiler sẽ tự động chuyển kiểu.

### Có những cách sử dụng Generics nào?

Generics thường có ba cách sử dụng: **generic class**, **generic interface**, **generic method**.

**1. Generic class**:

```java
// T ở đây có thể viết thành bất kỳ identifier nào; các type parameter thường gặp như T, E, K, V được dùng để biểu thị generic
// Khi khởi tạo generic class, bắt buộc phải chỉ định kiểu cụ thể của T
public class Generic<T>{

    private T key;

    public Generic(T key) {
        this.key = key;
    }

    public T getKey(){
        return key;
    }
}
```

Khởi tạo generic class:

```java
Generic<Integer> genericInteger = new Generic<Integer>(123456);
// Từ JDK 7 có thể viết: new Generic<>(123456)
```

**2. Generic interface**:

```java
public interface Generator<T> {
    public T method();
}
```

Implement generic interface nhưng không chỉ định kiểu:

```java
class GeneratorImpl<T> implements Generator<T>{
    @Override
    public T method() {
        return null;
    }
}
```

Implement generic interface và chỉ định kiểu:

```java
class GeneratorImpl implements Generator<String> {
    @Override
    public String method() {
        return "hello";
    }
}
```

**3. Generic method**:

```java
   public static < E > void printArray( E[] inputArray )
   {
         for ( E element : inputArray ){
            System.out.printf( "%s ", element );
         }
         System.out.println();
    }
```

Cách sử dụng:

```java
// Tạo các array khác kiểu: Integer, Double và Character
Integer[] intArray = { 1, 2, 3 };
String[] stringArray = { "Hello", "World" };
printArray( intArray  );
printArray( stringArray  );
```

### Generics được sử dụng ở đâu trong project?

- Interface tự định nghĩa cho kết quả trả về dùng chung `CommonResult<T>` có thể chỉ định động kiểu dữ liệu của kết quả theo kiểu trả về cụ thể thông qua tham số `T`
- Định nghĩa class xử lý `Excel` là `ExcelUtil<T>` để chỉ định động kiểu dữ liệu được export bởi `Excel`
- Xây dựng utility class cho collection (tham khảo các method `sort`, `binarySearch` trong `Collections`).
- ……

### Cơ chế type erasure của Generics là gì? Tại sao phải xóa kiểu?

**Java Generics được triển khai thông qua type erasure: generic instance không giữ lại type argument cụ thể khi runtime, nhưng class file vẫn có thể giữ thông tin khai báo generic trong các attribute như `Signature` và có thể đọc thông qua reflection API.**

Trong quá trình compile, compiler sẽ tự động xóa `T` của generic thành `Object`, hoặc xóa `T extends xxx` thành kiểu giới hạn `xxx`.

Type erasure giúp code generic duy trì khả năng tương thích với Java library và binary code được tạo trước khi Generics xuất hiện. Compiler duy trì type safety và ngữ nghĩa polymorphism thông qua các type conversion cần thiết và bridge method.

Điều này có thể hơi trừu tượng, hãy xem một ví dụ:

```java
List<Integer> list = new ArrayList<>();

list.add(12);
//1. Thêm trực tiếp trong quá trình compile sẽ báo lỗi
list.add("a");
Class<? extends List> clazz = list.getClass();
Method add = clazz.getDeclaredMethod("add", Object.class);
//2. Có thể thêm thông qua reflection trong runtime
add.invoke(list, "kl");

System.out.println(list)
```

Xem thêm một ví dụ: do vấn đề type erasure, overload method dưới đây sẽ báo lỗi.

```java
public void print(List<String> list)  { }
public void print(List<Integer> list) { }
```

![Vấn đề của type erasure](https://oss.javaguide.cn/github/javaguide/java/basis/generics-runtime-erasure.png)

Nguyên nhân rất đơn giản: sau khi type erasure, `List<String>` và `List<Integer>` đều trở thành `List` sau khi compile.

**Nếu compiler phải xóa generic, vậy tại sao vẫn dùng Generics? Dùng `Object` thay thế không được sao?**

Câu hỏi này thực chất gián tiếp kiểm tra tác dụng của Generics:

- Dùng Generics cho phép kiểm tra kiểu trong quá trình compile.

- Khi dùng kiểu `Object`, cần thêm type casting thủ công, làm giảm khả năng đọc code và tăng xác suất xảy ra lỗi.

- Generics có thể dùng self-bounded type như `T extends Comparable`.

### Bridge method là gì?

Bridge method (`Bridge Method`) được dùng để đảm bảo polymorphism khi kế thừa generic class.

```java
class Node<T> {
    public T data;
    public Node(T data) { this.data = data; }
    public void setData(T data) {
        System.out.println("Node.setData");
        this.data = data;
    }
}

class MyNode extends Node<Integer> {
    public MyNode(Integer data) { super(data); }

    // Sau khi type erasure, Node<T> trở thành setData(Object data), nhưng subclass MyNode không override method này, nên compiler sẽ thêm bridge method để đảm bảo polymorphism
    public void setData(Object data) {
        setData((Integer) data);
    }

    public void setData(Integer data) {
        System.out.println("MyNode.setData");
        super.setData(data);
    }
}
```

⚠️**Lưu ý**: bridge method do compiler tự động tạo ra, không viết thủ công.

### Generics có những hạn chế nào? Tại sao?

Các hạn chế của Generics thường do cơ chế type erasure gây ra. Sau khi bị xóa thành `Object`, không thể thực hiện kiểm tra kiểu.

- Có thể khai báo biến kiểu `T`, nhưng không thể trực tiếp khởi tạo type parameter thông qua `new T()`.
- Generic parameter không thể là primitive type. Vì primitive type không phải subclass của `Object`, cần dùng reference type tương ứng với primitive type để thay thế.
- Không thể khởi tạo array của generic parameter. Sau khi bị xóa thành `Object`, không thể thực hiện kiểm tra kiểu.
- Không thể khởi tạo generic array.
- Generics không thể dùng `instanceof` để kiểm tra type parameter `T` trong runtime; sau khi type erasure, `getClass()` cũng không thể phân biệt các generic type argument khác nhau (chẳng hạn `List<String>` và `List<Integer>` đều nhận được `List.class`).
- Không thể implement cùng một interface với hai generic parameter khác nhau; sau khi type erasure, bridge method của nhiều superclass sẽ xung đột.
- `static` context của class không thể tham chiếu type parameter do class đó khai báo, nhưng static generic method có thể khai báo và sử dụng type parameter của riêng nó.
- ……

### Code dưới đây có compile được không, tại sao?

```java
public final class Algorithm {
    public static <T> T max(T x, T y) {
        return x > y ? x : y;
    }
}
```

Không thể compile, vì `x` và `y` đều sẽ bị xóa thành kiểu `Object`, mà `Object` không thể dùng với phép so sánh `>`.

```java
public class Singleton<T> {

    public static T getInstance() {
        if (instance == null)
            instance = new Singleton<T>();

        return instance;
    }

    private static T instance = null;
}
```

Không thể compile, vì static field và static method của class không thể tham chiếu type parameter `T` do class đó khai báo. Static method có thể khai báo type parameter của riêng nó, ví dụ `public static <T> T getInstance()`.

## Wildcards

### Wildcard là gì? Có tác dụng gì?

Generic type là cố định, nên trong một số trường hợp sử dụng không đủ linh hoạt; khi đó wildcard xuất hiện! Wildcard cho phép type parameter thay đổi, dùng để giải quyết vấn đề generic type không hỗ trợ covariance.

Ví dụ:

```java
// Giới hạn kiểu là subclass của Person
<? extends Person>
// Giới hạn kiểu là superclass của Manager
<? super Manager>
```

### Wildcard khác gì với generic T thường dùng?

- `T` có thể dùng để khai báo biến hoặc constant, còn `?` thì không.
- `T` thường dùng để khai báo generic class hoặc method, còn wildcard `?` thường dùng trong code gọi generic method và formal parameter.
- Trong quá trình compile, `T` sẽ bị xóa thành kiểu giới hạn hoặc `Object`. Wildcard `?` sẽ được compiler "capture" thành một kiểu cụ thể nhưng chưa biết trong method (capture), nên không thể ghi phần tử nào ngoài `null` vào `List<?>`, nhưng có thể phối hợp với generic method.

### Unbounded wildcard là gì?

Unbounded wildcard có thể nhận dữ liệu generic type bất kỳ, dùng để triển khai các method đơn giản không phụ thuộc vào type parameter cụ thể, có thể capture kiểu của parameter rồi giao cho generic method xử lý.

```java
void testMethod(Person<?> p) {
  // Generic method tự xử lý
}
```

**`List<?>` và `List` có khác nhau không?** Tất nhiên là có!

- `List<?> list` biểu thị rằng element type của `list` là **một kiểu chưa biết nhưng cố định nào đó** (tức là **tồn tại một kiểu `T`, và `list` là `List<T>`**), vì vậy compiler không cho phép thêm bất kỳ element nào ngoài `null` vào đó để tránh không an toàn về kiểu.
- `List list` là raw type, sẽ bỏ qua một phần việc kiểm tra generic type và không tương đương với `List<Object>`. Việc thêm element vào đó thường tạo ra unchecked warning và có thể khiến lỗi kiểu bị trì hoãn đến runtime.

```java
List<?> list = new ArrayList<>();
list.add("sss");//báo lỗi
List list2 = new ArrayList<>();
list2.add("sss");//thông báo cảnh báo
```

### Upper bounded wildcard là gì? Lower bounded wildcard là gì?

Khi sử dụng Generics, có thể giới hạn upper bound và lower bound cho generic type argument được truyền vào, chẳng hạn: **type argument chỉ được truyền vào là superclass hoặc subclass của một kiểu nhất định**.

**Upper bounded wildcard `extends`** biểu thị rằng type argument phải là kiểu được chỉ định hoặc subclass của kiểu đó.

Ví dụ:

```java
// Giới hạn bắt buộc là subclass của class Person
<? extends Person>
```

Có thể thiết lập nhiều type bound, đồng thời cũng có thể giới hạn kiểu `T`.

```java
<T extends T1 & T2>
<T extends XXX>
```

**Lower bounded wildcard `super`** biểu thị rằng type argument phải là kiểu được chỉ định hoặc superclass của kiểu đó.

Ví dụ:

```java
//  Giới hạn bắt buộc là superclass của class Employee
List<? super Employee>
```

**`? extends xxx` và `? super xxx` khác nhau như thế nào?**

Phạm vi type argument mà hai loại này nhận là khác nhau. Với `List<? extends Xxx>`, có thể đọc phần tử dưới dạng `Xxx`, nhưng ngoài `null` thì không thể ghi an toàn vào đó; với `List<? super Xxx>`, có thể ghi `Xxx` và các subclass của nó, còn kết quả đọc chỉ có thể được xem an toàn là `Object`.

**Nguyên tắc PECS (Producer Extends, Consumer Super)**: khi **lấy** element từ data structure, dùng `extends` (producer, Producer); khi **ghi** element vào data structure, dùng `super` (consumer, Consumer). Ví dụ: `List<? extends Number>` chỉ có thể đọc `Number` từ đó, không thể ghi vào; `List<? super Integer>` có thể ghi `Integer` và các subclass của nó, khi đọc sẽ nhận được `Object`. `Collections.copy(List<? super T> dest, List<? extends T> src)` là cách sử dụng điển hình: đọc từ `src`, ghi vào `dest`.

**`T extends xxx` và `? extends xxx` khác nhau như thế nào?**

`T extends xxx` dùng để khai báo type parameter có upper bound, sau khi type erasure sẽ trở thành `xxx`; `? extends xxx` dùng cho wildcard type argument trong parameterized type, có thể xuất hiện ở field, local variable, method parameter và return type.

**`Class<?>` và `Class` khác nhau như thế nào?**

Dùng trực tiếp `Class` sẽ có type warning, còn dùng `Class<?>` thì không, vì `Class` là một generic class và việc nhận raw type sẽ tạo ra warning.

### Code dưới đây có compile được không, tại sao?

```java
class Shape { /* ... */ }
class Circle extends Shape { /* ... */ }
class Rectangle extends Shape { /* ... */ }

class Node<T> { /* ... */ }

Node<Circle> nc = new Node<>();
Node<Shape>  ns = nc;
```

Không thể, vì `Node<Circle>` không phải subclass của `Node<Shape>`.

```java
class Shape { /* ... */ }
class Circle extends Shape { /* ... */ }
class Rectangle extends Shape { /* ... */ }

class Node<T> { /* ... */ }
class ChildNode<T> extends Node<T>{

}
ChildNode<Circle> nc = new ChildNode<>();
Node<Circle>  ns = nc;
```

Có thể compile, `ChildNode<Circle>` là subclass của `Node<Circle>`.

```java
public static void print(List<? extends Number> list) {
    for (Number n : list)
        System.out.print(n + " ");
    System.out.println();
}
```

Có thể compile, `List<? extends Number>` có thể lấy element ra, nhưng không thể gọi `add()` để thêm element.

## Tham khảo

- Tài liệu chính thức của Java: https://docs.oracle.com/javase/tutorial/java/generics/index.html
- Java Basics: Hiểu rõ Generics trong một bài viết: https://www.cnblogs.com/XiiX/p/14719568.html
