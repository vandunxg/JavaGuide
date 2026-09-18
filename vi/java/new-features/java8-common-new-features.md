---
title: Thực chiến các tính năng mới của Java 8
description: Giải thích thực chiến các tính năng mới cốt lõi của Java 8, bao gồm Lambda, Stream, Optional, Date/Time API và default method của interface.
category: Java
tag:
  - Tính năng mới của Java
head:
  - - meta
    - name: keywords
      content: Java 8,Lambda,Stream API,Optional,Date/Time API,default method,functional interface
---

> Bài viết do [cowbi](https://github.com/cowbi) đóng góp~

<!-- markdownlint-disable MD024 -->

JDK 8 được phát hành vào ngày 18 tháng 3 năm 2014. Đây là một phiên bản LTS (Long-Term Support), đồng thời cũng là một trong những phiên bản được sử dụng rộng rãi trong hệ sinh thái Java trong thời gian dài. Các phiên bản LTS hiện được Oracle liệt kê gồm JDK 8, JDK 11, JDK 17, JDK 21 và JDK 25.

JDK 8 đưa vào nhiều tính năng mới quan trọng. Bài viết này sẽ chọn một số tính năng mới tương đối quan trọng để giới thiệu chi tiết:

- Lambda expression
- Stream API
- Lớp Optional
- Date-Time API
- default method của interface
- functional interface

Hình dưới đây thể hiện số lượng tính năng mới và thời điểm cập nhật do mỗi phiên bản từ JDK 8 đến JDK 24 mang lại:

![](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

Oracle phát hành Java 8 (JDK 1.8) vào năm 2014. Kể từ đó, phiên bản này được sử dụng rộng rãi trong hệ sinh thái Java trong thời gian dài. Nhiều lập trình viên vẫn chưa hiểu đầy đủ một số tính năng mới của nó, đặc biệt là những developer đã quen với các phiên bản trước Java 8, chẳng hạn như tôi.

Để không bị tụt lại quá xa, việc tổng hợp và hệ thống hóa các tính năng mới này vẫn cần thiết. So với jdk.7, nó có nhiều thay đổi hoặc có thể nói là tối ưu hóa, chẳng hạn như `interface` có thể có static method và có method body, điều này đã đảo ngược nhận thức trước đây; cấu trúc dữ liệu của `java.util.HashMap` được bổ sung red-black tree; cùng với Lambda expression đã quá quen thuộc. Bài viết này không thể chia sẻ lần lượt tất cả tính năng mới, mà chỉ liệt kê những tính năng mới thường dùng để giải thích chi tiết. Xem thêm phần giới thiệu về [các tính năng mới của Java 8 trên trang chính thức](https://www.oracle.com/java/technologies/javase/8-whats-new.html).

## Interface

Mục đích ban đầu của `interface` là hướng tới abstraction và nâng cao khả năng mở rộng. Điều này cũng để lại một hạn chế: khi sửa `Interface`, các class triển khai nó cũng phải sửa theo.

Để giải quyết vấn đề việc sửa interface không tương thích với các implementation hiện có, method của `interface` mới có thể được khai báo bằng `default` hoặc `static`, nhờ đó có thể có method body và class triển khai không cần override method này.

Một `interface` có thể có nhiều method được khai báo như vậy. Sự khác biệt giữa hai modifier này chủ yếu cũng là sự khác biệt giữa method thông thường và static method.

1. Method được khai báo bằng `default` là instance method thông thường, có thể gọi bằng `this`, đồng thời có thể được subclass inheritance và override.
2. Method được khai báo bằng `static` được sử dụng giống static method của class thông thường. Tuy nhiên, nó không thể được subclass inheritance mà chỉ có thể gọi bằng `Interface`.

Hãy xem một ví dụ thực tế.

```java
public interface InterfaceNew {
    static void sm() {
        System.out.println("Cách triển khai do interface cung cấp");
    }
    static void sm2() {
        System.out.println("Cách triển khai do interface cung cấp");
    }

    default void def() {
        System.out.println("default method của interface");
    }
    default void def2() {
        System.out.println("default2 method của interface");
    }
    // Class triển khai phải override
    void f();
}

public interface InterfaceNew1 {
    default void def() {
        System.out.println("default method của InterfaceNew1");
    }
}
```

Nếu một class vừa triển khai interface `InterfaceNew` vừa triển khai interface `InterfaceNew1`, cả hai interface đều có `def()`, đồng thời `InterfaceNew` và `InterfaceNew1` không có quan hệ inheritance, thì class đó bắt buộc phải override `def()`. Nếu không, compiler sẽ báo lỗi.

```java
public class InterfaceNewImpl implements InterfaceNew , InterfaceNew1{
    public static void main(String[] args) {
        InterfaceNewImpl interfaceNew = new InterfaceNewImpl();
        interfaceNew.def();
    }

    @Override
    public void def() {
        InterfaceNew1.super.def();
    }

    @Override
    public void f() {
    }
}
```

**Trong Java 8, interface và abstract class khác nhau như thế nào?**

Nhiều bạn cho rằng: “Nếu `interface` cũng có thể có implementation method riêng thì dường như không khác `abstract class` là bao.”

Thực ra chúng vẫn có khác biệt:

1. Sự khác biệt giữa `interface` và `class`, chủ yếu gồm:

   - Interface hỗ trợ multiple inheritance, class chỉ single inheritance
   - Instance method không có body trong interface mặc định là `public abstract`, field mặc định là `public static final`; ngoài ra interface còn có thể khai báo các method như `default`, `static`. Member của abstract class có thể sử dụng nhiều modifier hơn

2. Method của `interface` giống một extension plugin hơn, còn method của `abstract class` được dùng để inheritance.

Như đã đề cập ở đầu, các method có modifier `default` và `static` được bổ sung vào `interface` nhằm giải quyết vấn đề việc sửa interface không tương thích với các implementation hiện có, chứ không nhằm thay thế `abstract class`. Khi sử dụng, nơi nào nên dùng abstract class thì vẫn dùng abstract class, không nên thay thế nó chỉ vì các tính năng mới của interface.

**Hãy nhớ rằng interface luôn khác class.**

## functional interface

**Định nghĩa**: Còn gọi là SAM interface, tức Single Abstract Method interface, là interface có đúng một abstract method nhưng có thể có nhiều non-abstract method.

Trong Java 8 có riêng package `java.util.function` dành cho functional interface. Tất cả interface trong package này đều có annotation `@FunctionalInterface`, cung cấp khả năng functional programming.

Trong các package khác cũng có functional interface. Một số interface không có annotation `@FunctionalInterface`, nhưng chỉ cần phù hợp với định nghĩa functional interface thì vẫn là functional interface, không liên quan đến việc có annotation `@FunctionalInterface` hay không. Annotation này chỉ dùng để bắt buộc chuẩn hóa định nghĩa trong quá trình compile. Functional interface được ứng dụng rộng rãi trong Lambda expression.

## Lambda expression

Tiếp theo là Lambda expression quen thuộc. Đây là tính năng mới quan trọng nhất thúc đẩy việc phát hành Java 8. Kể từ sau generic (`Generics`) và annotation (`Annotation`), đây là thay đổi lớn nhất.

Lambda expression giúp code trở nên ngắn gọn và súc tích hơn, đồng thời cho phép Java hỗ trợ functional programming đơn giản.

> Lambda expression là một anonymous function. Java 8 cho phép truyền function làm parameter vào method.

### Cú pháp

```java
(parameters) -> expression hoặc
(parameters) ->{ statements; }
```

### Thực chiến Lambda

Hãy dùng các ví dụ thường gặp để cảm nhận sự tiện lợi mà Lambda mang lại.

#### Thay thế anonymous inner class

Trước đây, cách duy nhất để truyền parameter động vào method là sử dụng inner class. Ví dụ:

**1. Interface `Runnable`**

```java
new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("The runable now is using!");
            }
}).start();
// Dùng lambda
new Thread(() -> System.out.println("It's a lambda function!")).start();
```

**2. Interface `Comparator`**

```java
List<Integer> strings = Arrays.asList(1, 2, 3);

Collections.sort(strings, new Comparator<Integer>() {
@Override
public int compare(Integer o1, Integer o2) {
    return Integer.compare(o1, o2);}
});

// Lambda
Collections.sort(strings, (Integer o1, Integer o2) -> Integer.compare(o1, o2));
// Tách ra
Comparator<Integer> comparator = (Integer o1, Integer o2) -> Integer.compare(o1, o2);
Collections.sort(strings, comparator);
```

**3. Interface `Listener`**

```java
JButton button = new JButton();
button.addItemListener(new ItemListener() {
@Override
public void itemStateChanged(ItemEvent e) {
   e.getItem();
}
});
// lambda
button.addItemListener(e -> e.getItem());
```

**4. Interface tự định nghĩa**

Ba ví dụ trên là những trường hợp thường gặp nhất trong quá trình phát triển. Qua đó cũng có thể cảm nhận sự tiện lợi và gọn gàng mà Lambda mang lại. Nó chỉ giữ lại code thực sự được sử dụng và lược bỏ toàn bộ code không cần thiết. Vậy nó có yêu cầu gì đối với interface không? Ta nhận thấy các anonymous inner class này chỉ override một method của interface, dĩ nhiên cũng chỉ có một method cần override. Đây chính là **functional interface** đã đề cập ở trên. Nói cách khác, chỉ cần parameter của method là functional interface thì có thể dùng Lambda expression.

```java
@FunctionalInterface
public interface Comparator<T>{
    int compare(T o1, T o2);
}

@FunctionalInterface
public interface Runnable{
    void run();
}
```

Ta tự định nghĩa một functional interface:

```java
@FunctionalInterface
public interface LambdaInterface {
 void f();
}
// Sử dụng
public class LambdaClass {
    public static void forEg() {
        lambdaInterfaceDemo(()-> System.out.println("Functional interface tự định nghĩa"));
    }
    // Parameter là functional interface
    static void lambdaInterfaceDemo(LambdaInterface i){
        i.f();
    }
}
```

#### Lặp collection

```java
void lamndaFor() {
        List<String> strings = Arrays.asList("1", "2", "3");
        // foreach truyền thống
        for (String s : strings) {
            System.out.println(s);
        }
        // foreach bằng Lambda
        strings.forEach((s) -> System.out.println(s));
        // hoặc
        strings.forEach(System.out::println);
     // map
        Map<Integer, String> map = new HashMap<>();
        map.forEach((k,v)->System.out.println(v));
}
```

#### Method reference

Java 8 cho phép sử dụng keyword `::` để truyền method hoặc constructor reference. Dù thế nào, kiểu trả về của expression cũng phải là functional-interface.

```java
public class LambdaClassSuper {
    LambdaInterface sf(){
        return null;
    }
}

public class LambdaClass extends LambdaClassSuper {
    public static LambdaInterface staticF() {
        return null;
    }

    public LambdaInterface f() {
        return null;
    }

    void show() {
        // 1. Gọi static function, kiểu trả về phải là functional-interface
        LambdaInterface t = LambdaClass::staticF;

        // 2. Gọi instance method
        LambdaClass lambdaClass = new LambdaClass();
        LambdaInterface lambdaInterface = lambdaClass::f;

        // 3. Gọi method trên superclass
        LambdaInterface superf = super::sf;

        // 4. Gọi constructor
        LambdaInterface tt = LambdaClassSuper::new;
    }
}
```

#### Truy cập biến

```java
int i = 0;
Collections.sort(strings, (Integer o1, Integer o2) -> o1 - i);
// i =3;
```

Lambda expression có thể tham chiếu đến local variable bên ngoài, nhưng biến đó phải là `final` hoặc effectively final (không được gán lại sau khi khởi tạo). Compiler sẽ không tự động thêm modifier `final` cho biến.

## Stream

Java bổ sung package `java.util.stream`, có nhiều điểm tương tự với các loại stream trước đây. Loại stream được tiếp xúc nhiều nhất trước đây là resource stream, chẳng hạn `java.io.FileInputStream`, dùng stream để input file từ nơi này sang nơi khác. Nó chỉ là công cụ vận chuyển nội dung và không thực hiện _CRUD_ nào trên nội dung file.

`Stream` vẫn không lưu trữ dữ liệu, nhưng khác ở chỗ nó có thể retrieve và xử lý logic dữ liệu collection, bao gồm filter, sort, thống kê, count, v.v. Có thể hình dung nó giống câu lệnh SQL.

Source data của nó có thể là `Collection`, `Array`, v.v. Vì parameter của các method đều là functional interface type nên thường được sử dụng cùng Lambda.

### Loại stream

1. stream tuần tự
2. parallelStream song song, có thể thực thi bằng nhiều thread

### Method thường dùng

Tiếp theo hãy xem các method thường dùng của `java.util.stream.Stream`.

```java
/**
 * Trả về một stream tuần tự
 */
default Stream<E> stream()

/**
 * Trả về một stream song song
 */
default Stream<E> parallelStream()

/**
 * Trả về stream của T
 */
public static<T> Stream<T> of(T t)

/**
 * Trả về một ordered stream có các element là những value được chỉ định.
 */
public static<T> Stream<T> of(T... values) {
    return Arrays.stream(values);
}


/**
 * Filter, trả về stream gồm các element của stream này khớp với predicate đã cho
 */
Stream<T> filter(Predicate<? super T> predicate);

/**
 * Kiểm tra tất cả element của stream này có khớp với predicate được cung cấp hay không.
 */
boolean allMatch(Predicate<? super T> predicate)

/**
 * Kiểm tra có element nào của stream này khớp với predicate được cung cấp hay không.
 */
boolean anyMatch(Predicate<? super T> predicate);

/**
 * Trả về builder của một Stream.
 */
public static<T> Builder<T> builder();

/**
 * Dùng Collector để reduce các element của stream này
 */
<R, A> R collect(Collector<? super T, A, R> collector);

/**
 * Trả về số lượng element trong stream này.
 */
long count();

/**
 * Trả về stream gồm các element khác nhau của stream này (theo Object.equals(Object)).
 */
Stream<T> distinct();

/**
 * Lặp qua
 */
void forEach(Consumer<? super T> action);

/**
 * Dùng để lấy stream với số lượng được chỉ định, độ dài bị cắt không vượt quá maxSize.
 */
Stream<T> limit(long maxSize);

/**
 * Dùng để map mỗi element tới result tương ứng
 */
<R> Stream<R> map(Function<? super T, ? extends R> mapper);

/**
 * Sort theo Comparator được cung cấp.
 */
Stream<T> sorted(Comparator<? super T> comparator);

/**
 * Bỏ n element đầu tiên của stream này và trả về stream mới gồm các element còn lại.
 */
Stream<T> skip(long n);

/**
 * Trả về một array chứa các element của stream này.
 */
Object[] toArray();

/**
 * Dùng generator được cung cấp để trả về một array chứa các element của stream này, nhằm cấp phát array trả về và các array khác cần thiết cho việc thực thi theo partition hoặc resize.
 */
<A> A[] toArray(IntFunction<A[]> generator);

/**
 * Gộp stream
 */
public static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b)
```

### Thực chiến

Bài viết liệt kê cách sử dụng các method tiêu biểu của `Stream`. Để biết thêm cách sử dụng, hãy xem API.

```java
@Test
public void test() {
  List<String> strings = Arrays.asList("abc", "def", "gkh", "abc");
    // Trả về stream thỏa điều kiện
    Stream<String> stringStream = strings.stream().filter(s -> "abc".equals(s));
    // Tính số lượng element trong stream thỏa điều kiện
    long count = stringStream.count();

    // forEach lặp qua -> in element
    strings.stream().forEach(System.out::println);

    // limit lấy stream có 1 element
    Stream<String> limit = strings.stream().limit(1);
    // toArray: ví dụ muốn xem limitStream bên trong, chẳng hạn chuyển thành String[], hoặc lặp qua
    String[] array = limit.toArray(String[]::new);

    // map thao tác trên mỗi element và trả về stream mới
    Stream<String> map = strings.stream().map(s -> s + "22");

    // sorted sort và in
    strings.stream().sorted().forEach(System.out::println);

    // Collectors collect đưa abc vào container
    List<String> collect = strings.stream().filter(string -> "abc".equals(string)).collect(Collectors.toList());
    // Chuyển list thành string, các element được ngăn cách bằng dấu phẩy
    String mergedString = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.joining(","));

    // Thống kê array, chẳng hạn:
    List<Integer> number = Arrays.asList(1, 2, 5, 4);

    IntSummaryStatistics statistics = number.stream().mapToInt((x) -> x).summaryStatistics();
    System.out.println("Số lớn nhất trong list: "+statistics.getMax());
    System.out.println("Số nhỏ nhất trong list: "+statistics.getMin());
    System.out.println("Giá trị trung bình: "+statistics.getAverage());
    System.out.println("Tổng tất cả các số: "+statistics.getSum());

    // concat gộp stream
    List<String> strings2 = Arrays.asList("xyz", "jqx");
    Stream.concat(strings2.stream(),strings.stream()).count();

    // Lưu ý: một Stream chỉ có thể thao tác một lần, nếu không sẽ báo lỗi.
    Stream stream = strings.stream();
    // Sử dụng lần thứ nhất
    stream.limit(2);
    // Sử dụng lần thứ hai
    stream.forEach(System.out::println);
    // Báo lỗi java.lang.IllegalStateException: stream has already been operated upon or closed

    // Có thể gọi liên tiếp trong cùng một pipeline
    strings.stream().limit(2).forEach(System.out::println);
}
```

### Thực thi lazy

Khi thực thi method trả về `Stream`, method không được thực thi ngay mà chỉ thực thi sau khi có một method không trả về `Stream`. Vì lấy được `Stream` chưa có nghĩa là có thể sử dụng trực tiếp, mà cần xử lý nó thành một type thông thường. Có thể hình dung `Stream` ở đây giống binary stream (hai thứ hoàn toàn khác nhau), lấy được cũng không thể đọc hiểu.

Hãy phân tích method `filter` bên dưới.

```java
@Test
public void laziness(){
  List<String> strings = Arrays.asList("abc", "def", "gkh", "abc");
  Stream<Integer> stream = strings.stream().filter(new Predicate() {
      @Override
      public boolean test(Object o) {
        System.out.println("Thực thi Predicate.test");
        return true;
        }
      });

   System.out.println("Thực thi count");
   stream.count();
}
/*-------Kết quả thực thi--------*/
Thực thi count
Thực thi Predicate.test
Thực thi Predicate.test
Thực thi Predicate.test
Thực thi Predicate.test
```

Theo thứ tự thực thi, lẽ ra phải in `Thực thi Predicate.test` 4 lần trước, sau đó mới in `Thực thi count`. Kết quả thực tế hoàn toàn ngược lại. Điều này cho thấy method trong `filter` không được thực thi ngay mà chỉ thực thi sau khi gọi method `count()`.

Các ví dụ trên đều là instance của `Stream` tuần tự. `parallelStream` song song có cách sử dụng giống stream tuần tự. Khác biệt chính là `parallelStream` có thể thực thi bằng nhiều thread, được triển khai dựa trên framework ForkJoin. Khi có thời gian, bạn có thể tìm hiểu framework `ForkJoin` và `ForkJoinPool`. Có thể hiểu đơn giản rằng nó được thực hiện thông qua thread pool, từ đó liên quan đến các vấn đề như thread safety và mức tiêu hao thread. Tiếp theo, hãy trải nghiệm việc thực thi bằng nhiều thread của parallel stream qua code.

```java
@Test
public void parallelStreamTest(){
   List<Integer> numbers = Arrays.asList(1, 2, 5, 4);
   numbers.parallelStream() .forEach(num->System.out.println(Thread.currentThread().getName()+">>"+num));
}
// Kết quả thực thi
main>>5
ForkJoinPool.commonPool-worker-2>>4
ForkJoinPool.commonPool-worker-11>>1
ForkJoinPool.commonPool-worker-9>>2
```

Từ kết quả có thể thấy `for-each` sử dụng nhiều thread.

### Tóm tắt

Từ source code và các instance, có thể tổng kết một số đặc điểm của stream:

1. Thông qua chain programming đơn giản, nó có thể dễ dàng xử lý tiếp dữ liệu sau khi lặp.
2. Parameter của các method đều là functional interface type
3. Một Stream chỉ có thể thao tác một lần, thao tác xong sẽ đóng; tiếp tục sử dụng stream này sẽ báo lỗi.
4. Stream không lưu dữ liệu và không thay đổi source data

## Optional

Trong [phần giới thiệu Optional của Sổ tay phát triển Alibaba](https://share.weiyun.com/ThuqEbD5) có viết:

> Ngăn chặn NPE là phẩm chất cơ bản của lập trình viên. Hãy chú ý các tình huống phát sinh NPE:
>
> 1. Khi return type là primitive type nhưng return object của wrapper type, việc unboxing tự động có thể gây NPE.
>
> Ví dụ sai: `public int f() { return object Integer }`, nếu là `null` thì unboxing tự động ném NPE.
>
> 2. Kết quả query database có thể là `null`.
> 3. Dù element trong collection `isNotEmpty`, data element lấy ra vẫn có thể là `null`.
> 4. Khi remote call trả về object, luôn phải kiểm tra null để ngăn NPE.
> 5. Với data lấy từ Session, nên kiểm tra NPE để tránh null pointer.
> 6. Chained call `obj.getA().getB().getC()`; chuỗi call liên tiếp dễ phát sinh NPE.
>
> Ví dụ đúng: dùng class `Optional` của JDK 8 để ngăn vấn đề NPE.

Ở đây khuyến nghị dùng `Optional` để biểu đạt rõ ràng “có thể không có kết quả”, từ đó giảm một phần rủi ro NPE (`java.lang.NullPointerException`). Optional hoặc chứa một giá trị khác `null`, hoặc rỗng, và không lưu `null` bên trong. Tiếp theo, hãy lần lượt tìm hiểu cách triển khai của `Optional` qua source code.

Giả sử có một class `Zoo` chứa một thuộc tính `Dog`, yêu cầu là lấy `age` của `Dog`.

```java
class Zoo {
   private Dog dog;
}

class Dog {
   private int age;
}
```

Cách truyền thống để giải quyết NPE như sau:

```java
Zoo zoo = getZoo();
if(zoo != null){
   Dog dog = zoo.getDog();
   if(dog != null){
      int age = dog.getAge();
      System.out.println(age);
   }
}
```

Kiểm tra object không null từng lớp. Có người cho rằng cách này xấu và không thanh lịch, nhưng tôi không nghĩ vậy. Ngược lại, tôi thấy nó gọn gàng, dễ đọc và dễ hiểu. Bạn nghĩ sao?

Cách triển khai bằng `Optional` như sau:

```java
Optional.ofNullable(zoo).map(o -> o.getDog()).map(d -> d.getAge()).ifPresent(age ->
    System.out.println(age)
);
```

Có phải ngắn gọn hơn nhiều không?

### Cách tạo một Optional

Trong ví dụ trên, `Optional.ofNullable` là một trong các cách tạo Optional. Trước hết hãy xem ý nghĩa của nó và source code của các method tạo khác.

```java
/**
 * Common instance for {@code empty()}. Object EMPTY toàn cục
 */
private static final Optional<?> EMPTY = new Optional<>();

/**
 * Giá trị được Optional duy trì
 */
private final T value;

/**
 * Nếu value là null thì trả về EMPTY, nếu không thì trả về of(T)
 */
public static <T> Optional<T> ofNullable(T value) {
   return value == null ? empty() : of(value);
}
/**
 * Trả về object EMPTY
 */
public static<T> Optional<T> empty() {
   Optional<T> t = (Optional<T>) EMPTY;
   return t;
}
/**
 * Trả về object Optional
 */
public static <T> Optional<T> of(T value) {
    return new Optional<>(value);
}
/**
 * Constructor private, gán giá trị cho value
 */
private Optional(T value) {
  this.value = Objects.requireNonNull(value);
}
/**
 * Vì vậy nếu value của of(T value) là null thì sẽ ném NullPointerException, dường như chưa giải quyết vấn đề NPE
 */
public static <T> T requireNonNull(T obj) {
  if (obj == null)
         throw new NullPointerException();
  return obj;
}
```

Khác biệt chính giữa method `ofNullable` và method `of` là: khi value là `null`, `ofNullable` trả về Optional rỗng, còn `of` sẽ ném `NullPointerException`. Khi `null` biểu thị “không có giá trị” hợp lệ thì dùng `ofNullable`; khi parameter theo quy ước bắt buộc khác `null` thì có thể dùng `of` để phát hiện lỗi sớm.

**`map()` và `flatMap()` khác nhau như thế nào?**

Cả `map` và `flatMap` đều áp dụng một function lên từng element trong collection, nhưng `map` trả về collection mới, còn `flatMap` map mỗi element thành một collection rồi flatten collection đó.

Trong tình huống thực tế, nếu `map` trả về array thì kết quả cuối cùng là array hai chiều. Dùng `flatMap` nhằm flatten array hai chiều này thành array một chiều.

```java
public class MapAndFlatMapExample {
    public static void main(String[] args) {
        List<String[]> listOfArrays = Arrays.asList(
                new String[]{"apple", "banana", "cherry"},
                new String[]{"orange", "grape", "pear"},
                new String[]{"kiwi", "melon", "pineapple"}
        );

        List<String[]> mapResult = listOfArrays.stream()
                .map(array -> Arrays.stream(array).map(String::toUpperCase).toArray(String[]::new))
                .collect(Collectors.toList());

        System.out.println("Using map:");
        mapResult.forEach(arrays-> System.out.println(Arrays.toString(arrays)));

        List<String> flatMapResult = listOfArrays.stream()
                .flatMap(array -> Arrays.stream(array).map(String::toUpperCase))
                .collect(Collectors.toList());

        System.out.println("Using flatMap:");
        System.out.println(flatMapResult);
    }
}

```

Kết quả thực thi:

```plain
Using map:
[[APPLE, BANANA, CHERRY], [ORANGE, GRAPE, PEAR], [KIWI, MELON, PINEAPPLE]]

Using flatMap:
[APPLE, BANANA, CHERRY, ORANGE, GRAPE, PEAR, KIWI, MELON, PINEAPPLE]
```

Cách hiểu đơn giản nhất là `flatMap()` có thể mở rộng kết quả của `map()`.

Trong `Optional`, khi dùng `map()`, nếu mapping function trả về một giá trị thông thường thì giá trị đó được bọc trong một `Optional` mới. Khi dùng `flatMap`, nếu mapping function trả về một `Optional`, `Optional` được trả về sẽ được flatten và không bị bọc thành `Optional` lồng nhau.

Dưới đây là code ví dụ so sánh:

```java
public static void main(String[] args) {
        int userId = 1;

        // Code sử dụng flatMap
        String cityUsingFlatMap = getUserById(userId)
                .flatMap(OptionalExample::getAddressByUser)
                .map(Address::getCity)
                .orElse("Unknown");

        System.out.println("User's city using flatMap: " + cityUsingFlatMap);

        // Code không sử dụng flatMap
        Optional<Optional<Address>> optionalAddress = getUserById(userId)
                .map(OptionalExample::getAddressByUser);

        String cityWithoutFlatMap;
        if (optionalAddress.isPresent()) {
            Optional<Address> addressOptional = optionalAddress.get();
            if (addressOptional.isPresent()) {
                Address address = addressOptional.get();
                cityWithoutFlatMap = address.getCity();
            } else {
                cityWithoutFlatMap = "Unknown";
            }
        } else {
            cityWithoutFlatMap = "Unknown";
        }

        System.out.println("User's city without flatMap: " + cityWithoutFlatMap);
    }
```

Sử dụng đúng `flatMap` trong `Stream` và `Optional` có thể giảm nhiều code không cần thiết.

### Kiểm tra value có phải null không

```java
/**
 * Kiểm tra value có phải null không
 */
public boolean isPresent() {
    return value != null;
}
/**
 * Nếu value khác null thì thực thi consumer.accept
 */
public void ifPresent(Consumer<? super T> consumer) {
   if (value != null)
    consumer.accept(value);
}
```

### Lấy value

```java
/**
 * Return the value if present, otherwise invoke {@code other} and return
 * the result of that invocation.
 * Nếu value != null thì trả về value, nếu không thì trả về kết quả thực thi của other
 */
public T orElseGet(Supplier<? extends T> other) {
    return value != null ? value : other.get();
}

/**
 * Nếu value != null thì trả về value, nếu không thì trả về T
 */
public T orElse(T other) {
    return value != null ? value : other;
}

/**
 * Nếu value != null thì trả về value, nếu không thì ném exception do parameter trả về
 */
public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
        if (value != null) {
            return value;
        } else {
            throw exceptionSupplier.get();
        }
}
/**
 * Nếu value là null thì ném NoSuchElementException, nếu không thì trả về value.
 */
public T get() {
  if (value == null) {
      throw new NoSuchElementException("No value present");
  }
  return value;
}
```

### Filter value

```java
/**
 * 1. Nếu là empty thì trả về empty
 * 2. Nếu predicate.test(value)==true thì trả về this, nếu không thì trả về empty
 */
public Optional<T> filter(Predicate<? super T> predicate) {
        Objects.requireNonNull(predicate);
        if (!isPresent())
            return this;
        else
            return predicate.test(value) ? this : empty();
}
```

### Tóm tắt

Sau khi xem source code của `Optional`, có thể thấy `of()` yêu cầu parameter khác `null`, `get()` ném `NoSuchElementException` khi Optional rỗng, còn `flatMap()` dùng để flatten mapping result trả về Optional và không phải method cần tránh. Thông thường nên chọn `of()` hoặc `ofNullable()` tùy việc value có được phép thiếu hay không, đồng thời ưu tiên dùng `orElse`, `orElseGet`, `orElseThrow` và các method khác để xử lý giá trị rỗng. Cuối cùng hãy kết hợp sử dụng các method thường gặp của `Optional`.

```java
Optional.ofNullable(zoo).map(o -> o.getDog()).map(d -> d.getAge()).filter(v->v==1).orElse(3);
```

## Date-Time API

Đây là phần bổ sung mạnh mẽ cho `java.util.Date`, giải quyết phần lớn vấn đề của class Date:

1. Không thread-safe
2. Xử lý timezone phức tạp
3. Việc format và tính toán thời gian rườm rà
4. Thiết kế có khiếm khuyết: class Date đồng thời chứa date và time; ngoài ra còn có `java.sql.Date`, dễ gây nhầm lẫn.

Hãy so sánh sự khác biệt giữa `java.util.Date` và Date mới qua các ví dụ thời gian thường dùng. Code đang dùng `java.util.Date` nên được thay đổi.

### Các class chính của java.time

`java.util.Date` vừa chứa date vừa chứa time, còn `java.time` tách riêng hai phần này.

```java
LocalDateTime.class // date + time format: yyyy-MM-ddTHH:mm:ss.SSS
LocalDate.class // date format: yyyy-MM-dd
LocalTime.class // time format: HH:mm:ss
```

### Format

**Trước Java 8:**

```java
public void oldFormat(){
    Date now = new Date();
    // format yyyy-MM-dd
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
    String date  = sdf.format(now);
    System.out.println(String.format("date format : %s", date));

    // format HH:mm:ss
    SimpleDateFormat sdft = new SimpleDateFormat("HH:mm:ss");
    String time = sdft.format(now);
    System.out.println(String.format("time format : %s", time));

    // format yyyy-MM-dd HH:mm:ss
    SimpleDateFormat sdfdt = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    String datetime = sdfdt.format(now);
    System.out.println(String.format("dateTime format : %s", datetime));
}
```

**Từ Java 8:**

```java
public void newFormat(){
    // format yyyy-MM-dd
    LocalDate date = LocalDate.now();
    System.out.println(String.format("date format : %s", date));

    // format HH:mm:ss
    LocalTime time = LocalTime.now().withNano(0);
    System.out.println(String.format("time format : %s", time));

    // format yyyy-MM-dd HH:mm:ss
    LocalDateTime dateTime = LocalDateTime.now();
    DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    String dateTimeStr = dateTime.format(dateTimeFormatter);
    System.out.println(String.format("dateTime format : %s", dateTimeStr));
}
```

### Chuyển string thành date

**Trước Java 8:**

```java
// Đã deprecated
Date date = new Date("2021-01-26");
// Thay bằng
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
Date date1 = sdf.parse("2021-01-26");
```

**Từ Java 8:**

```java
LocalDate date = LocalDate.of(2021, 1, 26);
LocalDate.parse("2021-01-26");

LocalDateTime dateTime = LocalDateTime.of(2021, 1, 26, 12, 12, 22);
LocalDateTime.parse("2021-01-26T12:12:22");

LocalTime time = LocalTime.of(12, 12, 22);
LocalTime.parse("12:12:22");
```

**Trước Java 8**, mọi chuyển đổi đều cần thông qua class `SimpleDateFormat`, còn **từ Java 8** chỉ cần method `of` hoặc `parse` của `LocalDate`, `LocalTime`, `LocalDateTime`.

### Tính toán date

Dưới đây chỉ lấy **date sau một tuần** làm ví dụ. Các đơn vị khác (năm, tháng, ngày, nửa ngày, giờ, v.v.) cũng tương tự. Ngoài ra, các đơn vị này đều được định nghĩa trong enum _java.time.temporal.ChronoUnit_.

**Trước Java 8:**

```java
public void afterDay(){
     // Date sau một tuần
     SimpleDateFormat formatDate = new SimpleDateFormat("yyyy-MM-dd");
     Calendar ca = Calendar.getInstance();
     ca.add(Calendar.DATE, 7);
     Date d = ca.getTime();
     String after = formatDate.format(d);
     System.out.println("Date sau một tuần: " + after);

   // Tính khoảng cách giữa hai date là bao nhiêu ngày; cách tính số năm, số tháng cũng tương tự
     String dates1 = "2021-12-23";
   String dates2 = "2021-02-26";
     SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
     Date date1 = format.parse(dates1);
     Date date2 = format.parse(dates2);
     int day = (int) ((date1.getTime() - date2.getTime()) / (1000 * 3600 * 24));
     System.out.println(dates1 + " và " + dates2 + " cách nhau " + day + " ngày");
     // Kết quả: 2021-02-26 và 2021-12-23 cách nhau 300 ngày
}
```

**Từ Java 8:**

```java
public void pushWeek(){
     // Date sau một tuần
     LocalDate localDate = LocalDate.now();
     // Cách 1
     LocalDate after = localDate.plus(1, ChronoUnit.WEEKS);
     // Cách 2
     LocalDate after2 = localDate.plusWeeks(1);
     System.out.println("Date sau một tuần: " + after);

     // Tính khoảng cách giữa hai date là bao nhiêu ngày, số năm, số tháng
     LocalDate date1 = LocalDate.parse("2021-02-26");
     LocalDate date2 = LocalDate.parse("2021-12-23");
     Period period = Period.between(date1, date2);
     System.out.println("Khoảng cách từ date1 đến date2: "
                + period.getYears() + " năm "
                + period.getMonths() + " tháng "
                + period.getDays() + " ngày");
   // Kết quả in ra là “Khoảng cách từ date1 đến date2: 0 năm 9 tháng 27 ngày”
     // period.getDays() trả về số ngày sau khi bỏ phần năm và tháng, không phải tổng số ngày
     // Nếu muốn lấy chính xác tổng số ngày thì dùng method bên dưới
     long day = date2.toEpochDay() - date1.toEpochDay();
     System.out.println(date1 + " và " + date2 + " cách nhau " + day + " ngày");
     // Kết quả in ra: 2021-02-26 và 2021-12-23 cách nhau 300 ngày
}
```

### Lấy date được chỉ định

Ngoài việc tính date rườm rà, việc lấy một date cụ thể cũng rất bất tiện, chẳng hạn lấy ngày đầu tiên hoặc ngày cuối cùng của tháng hiện tại.

**Trước Java 8:**

```java
public void getDay() {

        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
        // Lấy ngày đầu tiên của tháng hiện tại:
        Calendar c = Calendar.getInstance();
        c.set(Calendar.DAY_OF_MONTH, 1);
        String first = format.format(c.getTime());
        System.out.println("first day:" + first);

        // Lấy ngày cuối cùng của tháng hiện tại
        Calendar ca = Calendar.getInstance();
        ca.set(Calendar.DAY_OF_MONTH, ca.getActualMaximum(Calendar.DAY_OF_MONTH));
        String last = format.format(ca.getTime());
        System.out.println("last day:" + last);

        // Ngày cuối cùng của năm
        Calendar currCal = Calendar.getInstance();
        Calendar calendar = Calendar.getInstance();
        calendar.clear();
        calendar.set(Calendar.YEAR, currCal.get(Calendar.YEAR));
        calendar.roll(Calendar.DAY_OF_YEAR, -1);
        Date time = calendar.getTime();
        System.out.println("last day:" + format.format(time));
}
```

**Từ Java 8:**

```java
public void getDayNew() {
    LocalDate today = LocalDate.now();
    // Lấy ngày đầu tiên của tháng hiện tại:
    LocalDate firstDayOfThisMonth = today.with(TemporalAdjusters.firstDayOfMonth());
    // Lấy ngày cuối cùng của tháng hiện tại
    LocalDate lastDayOfThisMonth = today.with(TemporalAdjusters.lastDayOfMonth());
    // Lấy ngày tiếp theo:
    LocalDate nextDay = lastDayOfThisMonth.plusDays(1);
    // Ngày cuối cùng của năm
    LocalDate lastday = today.with(TemporalAdjusters.lastDayOfYear());
    // Chủ nhật cuối cùng của năm 2021; nếu dùng Calendar thì rất rắc rối.
    LocalDate lastMondayOf2021 = LocalDate.parse("2021-12-31").with(TemporalAdjusters.lastInMonth(DayOfWeek.SUNDAY));
}
```

Trong `java.time.temporal.TemporalAdjusters` còn nhiều thuật toán tiện lợi. Ở đây không giới thiệu API nữa vì chúng đều rất đơn giản, xem là hiểu ngay.

### JDBC và Java 8

Hiện tại quan hệ tương ứng giữa các type thời gian của JDBC và type thời gian của Java 8 là:

1. `Date` ---> `LocalDate`
2. `Time` ---> `LocalTime`
3. `Timestamp` ---> `LocalDateTime`

Trước JDBC 4.2, thường dùng `java.sql.Date`, `java.sql.Time` và `java.sql.Timestamp` để lần lượt biểu diễn các SQL time type này.

### Timezone

> Timezone: Theo phân chia timezone chính thức, cứ mỗi 15° kinh độ là một timezone; toàn cầu có 24 timezone, mỗi timezone chênh nhau 1 giờ. Tuy nhiên, để thuận tiện cho quản lý hành chính, thường một quốc gia hoặc một tỉnh được xếp vào cùng một timezone. Ví dụ, lãnh thổ Trung Quốc trải dài khoảng 5 timezone nhưng thực tế chỉ dùng giờ chuẩn của timezone thứ tám phía Đông, tức giờ Bắc Kinh.

Về bản chất, object `java.util.Date` lưu số millisecond đã trôi qua từ 0 giờ ngày 1 tháng 1 năm 1970 (GMT) đến thời điểm mà Date object biểu diễn. Nói cách khác, dù `new Date` ở timezone nào thì số millisecond được ghi lại vẫn giống nhau và không liên quan đến timezone. Tuy nhiên khi sử dụng, nên chuyển nó thành giờ địa phương, đây là vấn đề internationalization của thời gian. Bản thân `java.util.Date` không hỗ trợ internationalization mà cần nhờ đến `TimeZone`.

```java
// Giờ Bắc Kinh: Wed Jan 27 14:05:29 CST 2021
Date date = new Date();

SimpleDateFormat bjSdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
// Timezone Bắc Kinh
bjSdf.setTimeZone(TimeZone.getTimeZone("Asia/Shanghai"));
System.out.println("Số millisecond: " + date.getTime() + ", giờ Bắc Kinh: " + bjSdf.format(date));

// Timezone Tokyo
SimpleDateFormat tokyoSdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
tokyoSdf.setTimeZone(TimeZone.getTimeZone("Asia/Tokyo"));  // Thiết lập timezone Tokyo
System.out.println("Số millisecond: " + date.getTime() + ", giờ Tokyo: " + tokyoSdf.format(date));

// Nếu in trực tiếp thì sẽ tự động chuyển thành giờ của timezone hiện tại
System.out.println(date);
// Wed Jan 27 14:05:29 CST 2021
```

Trong các tính năng mới, `java.time.ZonedDateTime` được đưa vào để biểu diễn thời gian có timezone. Có thể xem nó là `LocalDateTime + ZoneId`.

```java
// Thời gian của timezone hiện tại
ZonedDateTime zonedDateTime = ZonedDateTime.now();
System.out.println("Thời gian của timezone hiện tại: " + zonedDateTime);

// Giờ Tokyo
ZoneId zoneId = ZoneId.of(ZoneId.SHORT_IDS.get("JST"));
ZonedDateTime tokyoTime = zonedDateTime.withZoneSameInstant(zoneId);
System.out.println("Giờ Tokyo: " + tokyoTime);

// Chuyển ZonedDateTime thành LocalDateTime
LocalDateTime localDateTime = tokyoTime.toLocalDateTime();
System.out.println("Giờ địa phương sau khi chuyển từ giờ Tokyo: " + localDateTime);

// Chuyển LocalDateTime thành ZonedDateTime
ZonedDateTime localZoned = localDateTime.atZone(ZoneId.systemDefault());
System.out.println("Thời gian của timezone địa phương: " + localZoned);

// Kết quả in ra
Thời gian của timezone hiện tại: 2021-01-27T14:43:58.735+08:00[Asia/Shanghai]
Giờ Tokyo: 2021-01-27T15:43:58.735+09:00[Asia/Tokyo]
Giờ địa phương sau khi chuyển từ giờ Tokyo: 2021-01-27T15:43:58.735
Thời gian của timezone địa phương: 2021-01-27T15:43:58.735+08:00[Asia/Shanghai]
```

### Tóm tắt

Qua so sánh Date cũ và mới ở trên, dĩ nhiên đây chỉ là một phần khác biệt về chức năng; các chức năng khác cần tự tìm hiểu thêm. Tóm lại, date-time-api mang lại nhiều lợi ích cho việc thao tác date. Khi gặp thao tác với date trong công việc hằng ngày, ưu tiên đầu tiên là date-time-api; chỉ cân nhắc Date cũ nếu thực sự không giải quyết được.

## Tổng kết

Các tính năng mới của Java 8 đã được tổng hợp gồm:

- Interface & functional Interface
- Lambda
- Stream
- Optional
- Date time-api

Đây đều là những tính năng thường dùng trong quá trình phát triển. Sau khi hệ thống hóa, có thể thấy chúng thực sự hữu ích, nhưng tôi lại chưa áp dụng sớm hơn. Tôi luôn cảm thấy việc học các tính năng mới của Java 8 khá rắc rối nên vẫn sử dụng cách triển khai cũ. Thực ra chỉ cần vài ngày là có thể nắm được các tính năng mới này; một khi đã nắm được, hiệu suất sẽ tăng đáng kể. Việc tăng lương thực ra cũng là tiền trả cho việc học tập; nếu không học, cuối cùng sẽ bị đào thải và khủng hoảng tuổi 35 sẽ đến sớm hơn.

<!-- @include: @article-footer.snippet.md -->
