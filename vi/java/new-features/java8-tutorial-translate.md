---
title: "Bản dịch tiếng Việt 《Hướng dẫn Java 8》"
description: "Dịch và hệ thống hóa tutorial Java 8, bao quát Lambda, method reference, default method của interface, Stream và các tính năng mới khác cùng code ví dụ."
category: Java
tag:
  - Java New Features
head:
  - - meta
    - name: keywords
      content: Java 8,Hướng dẫn,Lambda,Method References,Default Methods,Stream API,Functional Interfaces,Date/Time API
---

# 《Hướng dẫn Java 8》 bản dịch tiếng Việt

JDK 8 được phát hành vào ngày 18 tháng 3 năm 2014. Đây là một phiên bản LTS (Long-Term Support), đồng thời là một trong những phiên bản quan trọng nhất trong lịch sử Java. Tính đến nay, đã có năm phiên bản LTS là JDK 8, JDK 11, JDK 17, JDK 21 và JDK 25.

JDK 8 đã bổ sung nhiều tính năng mới quan trọng. Bài viết này sẽ chọn một số tính năng tiêu biểu để giới thiệu chi tiết:

- Lambda expression
- Method reference
- Default method của interface
- Stream API
- Functional interface
- Class Optional
- Date/Time API
- Cải tiến annotation

Hình dưới đây thể hiện số lượng tính năng mới và thời điểm cập nhật của từng phiên bản từ JDK 8 đến JDK 24:

![](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

Khi Java 8 ngày càng phổ biến, nhiều người nhắc rằng Java 8 cũng là một chủ đề thường được hỏi trong phỏng vấn. Theo yêu cầu của mọi người và nhu cầu thực tế, tôi dự định tổng hợp phần kiến thức này. Ban đầu tôi định tự tổng hợp, sau đó tìm thấy một repository liên quan trên GitHub, địa chỉ:
[https://github.com/winterbe/java8-tutorial](https://github.com/winterbe/java8-tutorial). Repository này viết bằng tiếng Anh. Tôi đã dịch, đồng thời bổ sung và chỉnh sửa một phần nội dung. Dưới đây là phần chính của bài viết.

---

Chào mừng bạn đến với phần giới thiệu về Java 8. Tutorial này sẽ hướng dẫn bạn từng bước qua tất cả tính năng mới của ngôn ngữ. Dựa trên các ví dụ code ngắn gọn, bạn sẽ học cách sử dụng default method của interface, lambda expression, method reference và repeatable annotation. Cuối bài viết, bạn sẽ nắm được các thay đổi mới nhất của API như stream, functional interface, các mở rộng của class Map và Date API mới. Không có những đoạn văn dài khô khan, chỉ có nhiều đoạn code kèm chú thích.

## Default method của interface (Default Methods for Interfaces)

Java 8 cho phép thêm implementation của method không abstract vào interface bằng keyword `default`. Tính năng này còn được gọi là [virtual extension method](http://stackoverflow.com/a/24102730).

Ví dụ đầu tiên:

```java
interface Formula{

    double calculate(int a);

    default double sqrt(int a) {
        return Math.sqrt(a);
    }

}
```

Ngoài abstract method `calculate` dùng để tính công thức của interface, interface `Formula` còn định nghĩa default method `sqrt`. Class triển khai interface này chỉ cần triển khai abstract method `calculate`. Có thể sử dụng trực tiếp default method `sqrt`. Bạn cũng có thể trực tiếp tạo đối tượng thông qua interface rồi triển khai các default method trong interface. Hãy xem cách này qua code:

```java
public class Main {

  public static void main(String[] args) {
    // Truy cập interface thông qua anonymous inner class
    Formula formula = new Formula() {
        @Override
        public double calculate(int a) {
            return sqrt(a * 100);
        }
    };

    System.out.println(formula.calculate(100));     // 100.0
    System.out.println(formula.sqrt(16));           // 4.0

  }

}
```

`formula` được triển khai dưới dạng anonymous object. Code này rất dễ hiểu: chỉ 6 dòng code đã thực hiện phép tính `sqrt(a * 100)`. Ở phần tiếp theo, chúng ta sẽ thấy trong Java 8 có một cách tốt và tiện lợi hơn để triển khai object chỉ có một method.

**Ghi chú của người dịch:** Dù là abstract class hay interface, bạn đều có thể truy cập thông qua anonymous inner class. Không thể trực tiếp tạo object từ abstract class hoặc interface. Với cách truy cập interface thông qua anonymous inner class ở trên, có thể hiểu như sau: một inner class triển khai abstract method trong interface và trả về một object của inner class, sau đó reference của interface trỏ đến object này.

## Lambda expression (Lambda expressions)

Trước hết hãy xem cách sắp xếp chuỗi trong các phiên bản Java cũ:

```java
List<String> names = Arrays.asList("peter", "anna", "mike", "xenia");

Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return b.compareTo(a);
    }
});
```

Chỉ cần truyền một object List và một comparator vào static method `Collections.sort` để sắp xếp theo thứ tự chỉ định. Cách làm thông thường là tạo một comparator object dạng anonymous rồi truyền nó vào method `sort`.

Trong Java 8, bạn không cần dùng cách tạo anonymous object truyền thống này nữa. Java 8 cung cấp cú pháp ngắn gọn hơn là lambda expression:

```java
Collections.sort(names, (String a, String b) -> {
    return b.compareTo(a);
});
```

Có thể thấy code đã ngắn và dễ đọc hơn, nhưng thực tế còn có thể viết ngắn hơn:

```java
Collections.sort(names, (String a, String b) -> b.compareTo(a));
```

Với thân hàm chỉ có một dòng code, bạn có thể bỏ dấu ngoặc nhọn `{}` và keyword `return`. Tuy nhiên vẫn có thể viết ngắn hơn nữa:

```java
names.sort((a, b) -> b.compareTo(a));
```

Bản thân class List đã có method `sort`. Ngoài ra, Java compiler có thể tự suy luận kiểu của tham số, nên bạn không cần viết lại kiểu. Tiếp theo hãy xem lambda expression còn có cách sử dụng nào khác.

## Functional interface (Functional Interfaces)

**Ghi chú của người dịch:** Phần giải thích trong bản gốc chưa thật rõ ràng nên đã được chỉnh sửa!

Các nhà thiết kế ngôn ngữ Java đã dành nhiều công sức để tìm cách hỗ trợ Lambda thân thiện cho các function hiện có. Cuối cùng, họ đưa ra khái niệm functional interface. **Functional interface là interface chỉ chứa duy nhất một abstract method, nhưng có thể có nhiều method không abstract (tức default method được nói ở trên).** Những interface như vậy có thể làm target type của lambda expression. `java.lang.Runnable` và `java.util.concurrent.Callable` là hai ví dụ điển hình về functional interface. Java 8 bổ sung annotation đặc biệt `@FunctionalInterface`, nhưng annotation này thường không bắt buộc. Chỉ cần interface thỏa định nghĩa functional interface, Java compiler có thể dùng nó làm target type của lambda expression. Thông thường nên khai báo annotation `@FunctionalInterface` trên interface. Khi compiler phát hiện interface được đánh dấu không đáp ứng yêu cầu của functional interface, compiler sẽ báo lỗi như hình dưới đây.

![Annotation @FunctionalInterface](https://oss.javaguide.cn/github/javaguide/java/@FunctionalInterface.png)

Ví dụ:

```java
@FunctionalInterface
public interface Converter<F, T> {
  T convert(F from);
}
```

```java
    // TODO Chuyển chuỗi số thành kiểu số nguyên
    Converter<String, Integer> converter = (from) -> Integer.valueOf(from);
    Integer converted = converter.convert("123");
    System.out.println(converted.getClass()); //class java.lang.Integer
```

**Ghi chú của người dịch:** Phần lớn functional interface không cần tự viết. Java 8 đã cung cấp sẵn, các interface này nằm trong package `java.util.function`.

## Method và constructor reference (Method and Constructor References)

Code ở phần trước còn có thể biểu diễn bằng static method reference:

```java
    Converter<String, Integer> converter = Integer::valueOf;
    Integer converted = converter.convert("123");
    System.out.println(converted.getClass());   //class java.lang.Integer
```

Java 8 cho phép truyền reference của method hoặc constructor thông qua keyword `::`. Ví dụ trên cho thấy cách reference static method. Ngoài ra, chúng ta cũng có thể reference method của object:

```java
class Something {
    String startsWith(String s) {
        return String.valueOf(s.charAt(0));
    }
}
```

```java
Something something = new Something();
Converter<String, String> converter = something::startsWith;
String converted = converter.convert("Java");
System.out.println(converted);    // "J"
```

Tiếp theo hãy xem cách dùng keyword `::` để reference constructor. Trước hết, ta định nghĩa một class đơn giản có nhiều constructor:

```java
class Person {
    String firstName;
    String lastName;

    Person() {}

    Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
}
```

Tiếp theo, ta chỉ định một factory interface dùng để tạo object Person:

```java
interface PersonFactory<P extends Person> {
    P create(String firstName, String lastName);
}
```

Thay vì tự triển khai một factory hoàn chỉnh, ta liên kết chúng bằng constructor reference:

```java
PersonFactory<Person> personFactory = Person::new;
Person person = personFactory.create("Peter", "Parker");
```

Chỉ cần dùng `Person::new` để lấy reference đến constructor của class Person. Java compiler sẽ tự chọn constructor phù hợp dựa trên kiểu tham số của method `PersonFactory.create`.

## Scope của lambda expression (Lambda Scopes)

### Truy cập local variable

Bạn có thể trực tiếp truy cập local variable bên ngoài trong lambda expression:

```java
final int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);

stringConverter.convert(2);     // 3
```

Tuy nhiên, khác với anonymous object, biến `num` ở đây không cần khai báo là `final`, code vẫn đúng:

```java
int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);

stringConverter.convert(2);     // 3
```

Tuy vậy, `num` không được sửa bởi code phía sau (tức là ngầm mang ngữ nghĩa `final`). Ví dụ dưới đây sẽ không compile:

```java
int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);
num = 3;//Không được phép cố sửa num trong lambda expression.
```

### Truy cập field và static variable

Khác với local variable, trong lambda expression chúng ta có quyền đọc và ghi cả instance field lẫn static variable. Hành vi này giống với anonymous object.

```java
class Lambda4 {
    static int outerStaticNum;
    int outerNum;

    void testScopes() {
        Converter<Integer, String> stringConverter1 = (from) -> {
            outerNum = 23;
            return String.valueOf(from);
        };

        Converter<Integer, String> stringConverter2 = (from) -> {
            outerStaticNum = 72;
            return String.valueOf(from);
        };
    }
}
```

### Truy cập default method của interface

Bạn còn nhớ ví dụ về `formula` ở phần đầu không? Interface `Formula` định nghĩa default method `sqrt`, và mọi instance `formula` chứa anonymous object đều có thể truy cập method này. Điều đó không áp dụng được cho lambda expression.

Không thể truy cập default method từ lambda expression, vì vậy code sau không thể compile:

```java
Formula formula = (a) -> sqrt(a * 100);
```

## Functional interface tích hợp sẵn (Built-in Functional Interfaces)

API JDK 1.8 chứa nhiều functional interface tích hợp sẵn. Một số interface vốn đã phổ biến trong các phiên bản Java cũ như `Comparator` hoặc `Runnable` cũng được bổ sung annotation `@FunctionalInterface` để có thể dùng trong lambda expression.

Tuy nhiên, Java 8 API cũng cung cấp nhiều functional interface hoàn toàn mới để giúp công việc lập trình thuận tiện hơn. Một số interface bắt nguồn từ thư viện [Google Guava](https://code.google.com/p/guava-libraries/). Dù đã quen thuộc với các interface này, bạn vẫn nên xem cách chúng được mở rộng để dùng với lambda.

### Predicate

Interface Predicate là functional interface nhận một tham số và trả về giá trị boolean, dùng để **đánh giá điều kiện**. Interface này chứa nhiều default method để kết hợp Predicate thành logic phức tạp hơn (chẳng hạn như AND, OR, NOT):

**Ghi chú của người dịch:** Source code của interface Predicate như sau:

```java
package java.util.function;
import java.util.Objects;

@FunctionalInterface
public interface Predicate<T> {

    // Method này nhận một kiểu đầu vào và trả về giá trị boolean, dùng để đánh giá.
    boolean test(T t);

    // Method and tương tự toán tử quan hệ "&&", chỉ trả về true khi cả hai vế đều đúng.
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }
    // Tương tự toán tử "!", đảo ngược kết quả đánh giá.
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }
    // Method or tương tự toán tử quan hệ "||", trả về true khi ít nhất một vế đúng.
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }
   // Method này nhận một object Object và trả về kiểu Predicate, dùng để kiểm tra method test thứ nhất có giống (equal) method test thứ hai hay không.
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
```

Ví dụ:

```java
Predicate<String> predicate = (s) -> s.length() > 0;

predicate.test("foo");              // true
predicate.negate().test("foo");     // false

Predicate<Boolean> nonNull = Objects::nonNull;
Predicate<Boolean> isNull = Objects::isNull;

Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNotEmpty = isEmpty.negate();
```

### Function

Interface Function nhận một tham số và tạo ra kết quả. Default method có thể dùng để nối nhiều function với nhau (`compose`, `andThen`):

**Ghi chú của người dịch:** Source code của interface Function như sau:

```java

package java.util.function;

import java.util.Objects;

@FunctionalInterface
public interface Function<T, R> {

    // Áp dụng object Function vào tham số đầu vào rồi trả về kết quả tính toán.
    R apply(T t);
    // Kết hợp hai Function và trả về một Function có thể thực hiện chức năng của cả hai object Function.
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }
    //
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

```java
Function<String, Integer> toInteger = Integer::valueOf;
Function<String, String> backToString = toInteger.andThen(String::valueOf);
backToString.apply("123");     // "123"
```

### Supplier

Interface Supplier tạo ra kết quả thuộc kiểu generic được chỉ định. Khác với interface Function, interface Supplier không nhận tham số.

```java
Supplier<Person> personSupplier = Person::new;
personSupplier.get();   // new Person
```

### Consumer

Interface Consumer biểu diễn một thao tác thực hiện trên một tham số đầu vào duy nhất.

```java
Consumer<Person> greeter = (p) -> System.out.println("Hello, " + p.firstName);
greeter.accept(new Person("Luke", "Skywalker"));
```

### Comparator

Comparator là interface kinh điển trong Java cũ. Java 8 bổ sung nhiều default method trên nền tảng đó:

```java
Comparator<Person> comparator = (p1, p2) -> p1.firstName.compareTo(p2.firstName);

Person p1 = new Person("John", "Doe");
Person p2 = new Person("Alice", "Wonderland");

comparator.compare(p1, p2);             // > 0
comparator.reversed().compare(p1, p2);  // < 0
```

## Optional

Optional không phải functional interface mà là một container dùng để biểu diễn rõ ràng trường hợp "có thể không có giá trị". Sử dụng đúng cách có thể giảm một phần việc kiểm tra `null` thủ công, nhưng không đảm bảo chương trình sẽ không còn `NullPointerException`. Đây là một khái niệm quan trọng ở phần tiếp theo. Hãy cùng tìm hiểu nhanh cách Optional hoạt động.

Optional là một container đơn giản: hoặc chứa một giá trị khác `null`, hoặc rỗng. Trong trường hợp giá trị trả về có thể biểu diễn kết quả không tồn tại, có thể trả về Optional thay vì dùng `null` để biểu diễn không có kết quả.

Ghi chú của người dịch: Đã bổ sung tác dụng của từng method trong ví dụ.

```java
// of(): tạo một Optional cho giá trị khác null
Optional<String> optional = Optional.of("bam");
// isPresent(): trả về true nếu giá trị tồn tại, ngược lại trả về false
optional.isPresent();           // true
// get(): trả về giá trị nếu Optional có giá trị, ngược lại ném NoSuchElementException
optional.get();                 // "bam"
// orElse(): trả về giá trị nếu có, ngược lại trả về giá trị khác được chỉ định
optional.orElse("fallback");    // "bam"
// ifPresent(): gọi consumer nếu instance Optional có giá trị, ngược lại không xử lý gì
optional.ifPresent((s) -> System.out.println(s.charAt(0)));     // "b"
```

Khuyến nghị đọc: [[Java8] Cách sử dụng Optional đúng](https://blog.kaaass.net/archives/764)

## Stream (Stream)

`java.util.stream.Stream` biểu diễn một chuỗi thao tác có thể lần lượt thực hiện trên một tập hợp phần tử. Thao tác Stream được chia thành intermediate operation và terminal operation. Terminal operation trả về kết quả tính toán của một kiểu cụ thể, còn intermediate operation trả về chính Stream, nhờ đó bạn có thể nối nhiều thao tác theo thứ tự. Stream có thể được tạo từ nhiều data source như collection, array và generator; bản thân Map không có method `stream()`, nhưng có thể tạo stream từ view key, value hoặc entry. Thao tác Stream có thể thực hiện tuần tự hoặc song song.

Trước hết hãy xem cách sử dụng Stream. Đầu tiên tạo List dữ liệu cần dùng trong code ví dụ:

```java
List<String> stringList = new ArrayList<>();
stringList.add("ddd2");
stringList.add("aaa2");
stringList.add("bbb1");
stringList.add("aaa1");
stringList.add("bbb3");
stringList.add("ccc");
stringList.add("bbb2");
stringList.add("ddd1");
```

Java 8 mở rộng các collection class, cho phép tạo Stream thông qua `Collection.stream()` hoặc `Collection.parallelStream()`. Các phần sau sẽ giải thích chi tiết những thao tác Stream thường dùng:

### Filter (lọc)

Filter lọc thông qua một interface predicate và chỉ giữ lại các phần tử thỏa điều kiện. Đây là **intermediate operation**, vì vậy có thể áp dụng các thao tác Stream khác lên kết quả sau khi lọc (chẳng hạn `forEach`). `forEach` cần một function để thực hiện lần lượt trên các phần tử đã lọc. `forEach` là terminal operation, nên không thể thực hiện thao tác Stream khác sau `forEach`.

```java
        // Kiểm thử Filter (lọc)
        stringList
                .stream()
                .filter((s) -> s.startsWith("a"))
                .forEach(System.out::println);//aaa2 aaa1
```

`forEach` được thiết kế cho Lambda, giữ phong cách ngắn gọn nhất. Ngoài ra, bản thân lambda expression có thể tái sử dụng nên rất tiện lợi.

### Sorted (sắp xếp)

Sắp xếp là một **intermediate operation**, trả về Stream đã được sắp xếp. **Nếu không chỉ định Comparator tùy chỉnh thì Comparator mặc định sẽ được sử dụng.**

```java
        // Kiểm thử Sort (sắp xếp)
        stringList
                .stream()
                .sorted()
                .filter((s) -> s.startsWith("a"))
                .forEach(System.out::println);// aaa1 aaa2
```

Cần lưu ý rằng thao tác sắp xếp chỉ tạo ra một Stream đã được sắp xếp, không ảnh hưởng đến data source ban đầu. Sau khi sắp xếp, dữ liệu gốc `stringList` không bị thay đổi:

```java
    System.out.println(stringList);// ddd2, aaa2, bbb1, aaa1, bbb3, ccc, bbb2, ddd1
```

### Map (ánh xạ)

Intermediate operation `map` lần lượt chuyển các phần tử thành object khác theo interface Function được chỉ định.

Ví dụ dưới đây cho thấy cách chuyển chuỗi thành chuỗi viết hoa. Bạn cũng có thể dùng `map` để chuyển object thành kiểu khác. Kiểu Stream mà `map` trả về được quyết định bởi kiểu giá trị trả về của function truyền vào `map`.

```java
        // Kiểm thử thao tác Map
        stringList
                .stream()
                .map(String::toUpperCase)
                .sorted((a, b) -> b.compareTo(a))
                .forEach(System.out::println);// "DDD2", "DDD1", "CCC", "BBB3", "BBB2", "BBB1", "AAA2", "AAA1"
```

### Match (khớp)

Stream cung cấp nhiều thao tác match, cho phép kiểm tra Predicate đã chỉ định có khớp với toàn bộ Stream hay không. Tất cả thao tác match đều là **terminal operation** và trả về giá trị kiểu boolean.

```java
        // Kiểm thử thao tác Match (khớp)
        boolean anyStartsWithA =
                stringList
                        .stream()
                        .anyMatch((s) -> s.startsWith("a"));
        System.out.println(anyStartsWithA);      // true

        boolean allStartsWithA =
                stringList
                        .stream()
                        .allMatch((s) -> s.startsWith("a"));

        System.out.println(allStartsWithA);      // false

        boolean noneStartsWithZ =
                stringList
                        .stream()
                        .noneMatch((s) -> s.startsWith("z"));

        System.out.println(noneStartsWithZ);      // true
```

### Count (đếm)

Đếm là một **terminal operation**, trả về số lượng phần tử trong Stream. **Kiểu của giá trị trả về là long.**

```java
      // Kiểm thử thao tác Count (đếm)
        long startsWithB =
                stringList
                        .stream()
                        .filter((s) -> s.startsWith("b"))
                        .count();
        System.out.println(startsWithB);    // 3
```

### Reduce (rút gọn)

Đây là một **terminal operation**, cho phép rút gọn nhiều phần tử trong stream thành một phần tử thông qua function được chỉ định. Kết quả sau khi rút gọn được biểu diễn bằng interface Optional:

```java
        Optional<String> reduced =
                stringList
                        .stream()
                        .sorted()
                        .reduce((s1, s2) -> s1 + "#" + s2);

        reduced.ifPresent(System.out::println);//aaa1#aaa2#bbb1#bbb2#bbb3#ccc#ddd1#ddd2
```

**Ghi chú của người dịch:** Tác dụng chính của method này là kết hợp các phần tử Stream. Method cung cấp một giá trị bắt đầu (seed), sau đó kết hợp nó với phần tử thứ nhất, thứ hai, thứ n của Stream theo quy tắc tính toán (BinaryOperator). Theo nghĩa này, phép nối chuỗi và các phép sum, min, max, average trên số đều là các trường hợp đặc biệt của reduce. Ví dụ, sum của Stream tương đương với `Integer sum = integers.reduce(0, (a, b) -> a+b);`. Cũng có trường hợp không có giá trị bắt đầu; khi đó hai phần tử đầu tiên của Stream sẽ được kết hợp và kết quả trả về là Optional.

```java
// Nối chuỗi, concat = "ABCD"
String concat = Stream.of("A", "B", "C", "D").reduce("", String::concat);
// Tìm giá trị nhỏ nhất, minValue = -3.0
double minValue = Stream.of(-1.5, 1.0, -3.0, -2.0).reduce(Double.MAX_VALUE, Double::min);
// Tính tổng, sumValue = 10, có giá trị bắt đầu
int sumValue = Stream.of(1, 2, 3, 4).reduce(0, Integer::sum);
// Tính tổng, sumValue = 10, không có giá trị bắt đầu
sumValue = Stream.of(1, 2, 3, 4).reduce(Integer::sum).get();
// Lọc và nối chuỗi, concat = "ace"
concat = Stream.of("a", "B", "c", "D", "e", "F").
 filter(x -> x.compareTo("Z") > 0).
 reduce("", String::concat);
```

Trong code trên, chẳng hạn `reduce()` ở ví dụ đầu tiên có tham số thứ nhất (chuỗi rỗng) là giá trị bắt đầu, tham số thứ hai (`String::concat`) là BinaryOperator. Các `reduce()` có giá trị bắt đầu như vậy đều trả về object cụ thể. Với `reduce()` ở ví dụ thứ tư không có giá trị bắt đầu, do có thể không đủ phần tử nên kết quả trả về là Optional. Hãy lưu ý sự khác biệt này. Xem thêm: [IBM: Giải thích chi tiết Streams API trong Java 8](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/index.html)

## Parallel Stream (parallel stream)

Như đã nói ở trên, Stream có hai loại tuần tự và song song. Các thao tác trên Stream tuần tự được hoàn thành lần lượt trong một thread, còn Stream song song được thực hiện đồng thời trên nhiều thread.

Ví dụ dưới đây cho thấy cách cải thiện performance bằng parallel Stream.

Trước hết, ta tạo một bảng lớn không có phần tử trùng lặp:

```java
int max = 1000000;
List<String> values = new ArrayList<>(max);
for (int i = 0; i < max; i++) {
    UUID uuid = UUID.randomUUID();
    values.add(uuid.toString());
}
```

Ta lần lượt sắp xếp bằng hai cách tuần tự và song song, rồi so sánh thời gian sử dụng.

### Sequential Sort (sắp xếp tuần tự)

```java
// Sắp xếp tuần tự
long t0 = System.nanoTime();
long count = Arrays.stream(list.stream().sorted().toArray()).count();
System.out.println(count);

long t1 = System.nanoTime();

long millis = TimeUnit.NANOSECONDS.toMillis(t1 - t0);
System.out.println(String.format("sequential sort took: %d ms", millis));
```

```plain
1000000
sequential sort took: 709 ms // Thời gian sắp xếp tuần tự
```

### Parallel Sort (sắp xếp song song)

```java
// Sắp xếp song song
long t0 = System.nanoTime();

long count = Arrays.stream(list.parallelStream().sorted().toArray()).count();
System.out.println(count);

long t1 = System.nanoTime();

long millis = TimeUnit.NANOSECONDS.toMillis(t1 - t0);
System.out.println(String.format("parallel sort took: %d ms", millis));

```

```java
1000000
parallel sort took: 475 ms // Thời gian sắp xếp song song
```

Hai đoạn code trên gần như giống nhau, nhưng phiên bản song song nhanh hơn khoảng 50%. Thay đổi duy nhất cần làm là đổi `stream()` thành `parallelStream()`.

## Map

Như đã nói ở trên, kiểu Map không hỗ trợ stream, nhưng Map cung cấp một số method mới hữu ích để xử lý các tác vụ thường ngày. Bản thân interface Map không có method `stream()` để sử dụng, nhưng bạn có thể tạo stream riêng trên key, value hoặc thông qua `map.keySet().stream()`, `map.values().stream()` và `map.entrySet().stream()`.

Ngoài ra, Map hỗ trợ nhiều method mới và hữu ích để thực hiện các tác vụ phổ biến.

```java
Map<Integer, String> map = new HashMap<>();

for (int i = 0; i < 10; i++) {
    map.putIfAbsent(i, "val" + i);
}

map.forEach((id, val) -> System.out.println(val));//val0 val1 val2 val3 val4 val5 val6 val7 val8 val9
```

`putIfAbsent` ngăn chúng ta phải viết thêm code khi kiểm tra null; `forEach` nhận một consumer để thao tác trên từng phần tử trong map.

Ví dụ này cho thấy cách dùng function để tính toán trên map:

```java
map.computeIfPresent(3, (num, val) -> val + num);
map.get(3);             // val33

map.computeIfPresent(9, (num, val) -> null);
map.containsKey(9);     // false

map.computeIfAbsent(23, num -> "val" + num);
map.containsKey(23);    // true

map.computeIfAbsent(3, num -> "bam");
map.get(3);             // val33
```

Tiếp theo là cách xóa một entry trong Map khi cả key và value đều khớp:

```java
map.remove(3, "val3");
map.get(3);             // val33
map.remove(3, "val33");
map.get(3);             // null
```

Một method hữu ích khác:

```java
map.getOrDefault(42, "not found");  // not found
```

Việc merge các phần tử trong Map cũng trở nên dễ dàng:

```java
map.merge(9, "val9", (value, newValue) -> value.concat(newValue));
map.get(9);             // val9
map.merge(9, "concat", (value, newValue) -> value.concat(newValue));
map.get(9);             // val9concat
```

`merge` sẽ chèn nếu key chưa tồn tại; nếu đã tồn tại, method thực hiện merge trên value tương ứng với key cũ rồi chèn lại vào map.

## Date API (API liên quan đến ngày tháng)

Java 8 chứa một Date và Time API hoàn toàn mới trong package `java.time`. Date API mới tương tự thư viện Joda-Time, nhưng hai bên không giống nhau. Các ví dụ sau bao quát những phần quan trọng nhất của API mới. Người dịch đã tham khảo các tài liệu liên quan và chỉnh sửa phần lớn nội dung này.

**Ghi chú của người dịch (tổng hợp):**

- Class Clock cung cấp method truy cập ngày và giờ hiện tại. Clock nhạy với timezone và có thể dùng để lấy số mili giây hiện tại. Một thời điểm cụ thể cũng có thể được biểu diễn bằng class `Instant`. Class `Instant` cũng có thể dùng để tạo object `java.util.Date` của phiên bản cũ.

- Trong API mới, timezone được biểu diễn bằng ZoneId. Có thể dễ dàng lấy timezone bằng static method `of`. Abstract class `ZoneId` (trong package `java.time`) biểu diễn một zone identifier. Class này có static method `getAvailableZoneIds`, trả về tất cả zone identifier.

- JDK 1.8 bổ sung các class như LocalDate và LocalDateTime để xử lý ngày tháng, đồng thời bổ sung class mới DateTimeFormatter để giải quyết vấn đề format ngày tháng. Có thể dùng Instant thay cho Date, LocalDateTime thay cho Calendar và DateTimeFormatter thay cho SimpleDateFormat.

### Clock

Class Clock cung cấp method truy cập ngày và giờ hiện tại. Clock nhạy với timezone và có thể dùng để lấy số mili giây hiện tại. Một thời điểm cụ thể cũng có thể được biểu diễn bằng class `Instant`. Class `Instant` cũng có thể dùng để tạo object `java.util.Date` của phiên bản cũ.

```java
Clock clock = Clock.systemDefaultZone();
long millis = clock.millis();
System.out.println(millis);//1552379579043
Instant instant = clock.instant();
System.out.println(instant);
Date legacyDate = Date.from(instant); //2019-03-12T08:46:42.588Z
System.out.println(legacyDate);//Tue Mar 12 16:32:59 CST 2019
```

### Timezone (timezone)

Trong API mới, timezone được biểu diễn bằng ZoneId. Có thể dễ dàng lấy timezone bằng static method `of`. Abstract class `ZoneId` (trong package `java.time`) biểu diễn một zone identifier. Class này có static method `getAvailableZoneIds`, trả về tất cả zone identifier.

```java
// In tất cả zone identifier
System.out.println(ZoneId.getAvailableZoneIds());

ZoneId zone1 = ZoneId.of("Europe/Berlin");
ZoneId zone2 = ZoneId.of("Brazil/East");
System.out.println(zone1.getRules());// ZoneRules[currentStandardOffset=+01:00]
System.out.println(zone2.getRules());// ZoneRules[currentStandardOffset=-03:00]
```

### LocalTime (giờ địa phương)

LocalTime định nghĩa một thời gian không có thông tin timezone, chẳng hạn 10 giờ tối hoặc 17:30:15. Ví dụ dưới đây dùng timezone được tạo ở phần trước để tạo hai LocalTime. Sau đó so sánh thời gian và tính chênh lệch giữa hai thời điểm theo giờ và phút:

```java
LocalTime now1 = LocalTime.now(zone1);
LocalTime now2 = LocalTime.now(zone2);
System.out.println(now1.isBefore(now2));  // false

long hoursBetween = ChronoUnit.HOURS.between(now1, now2);
long minutesBetween = ChronoUnit.MINUTES.between(now1, now2);

System.out.println(hoursBetween);       // -3
System.out.println(minutesBetween);     // -239
```

LocalTime cung cấp nhiều factory method để đơn giản hóa việc tạo object, bao gồm parse chuỗi thời gian.

```java
LocalTime late = LocalTime.of(23, 59, 59);
System.out.println(late);       // 23:59:59
DateTimeFormatter germanFormatter =
    DateTimeFormatter
        .ofLocalizedTime(FormatStyle.SHORT)
        .withLocale(Locale.GERMAN);

LocalTime leetTime = LocalTime.parse("13:37", germanFormatter);
System.out.println(leetTime);   // 13:37
```

### LocalDate (ngày địa phương)

LocalDate biểu diễn một ngày cụ thể, chẳng hạn 2014-03-11. Giá trị của object này là immutable và cách sử dụng gần giống LocalTime. Ví dụ dưới đây cho thấy cách cộng trừ ngày, tháng và năm cho object Date. Cũng cần lưu ý rằng các object này immutable, thao tác luôn trả về một instance mới.

```java
LocalDate today = LocalDate.now();// Lấy ngày hiện tại
System.out.println("Today's date: " + today);//2019-03-12
LocalDate tomorrow = today.plus(1, ChronoUnit.DAYS);
System.out.println("Tomorrow's date: " + tomorrow);//2019-03-13
LocalDate yesterday = tomorrow.minusDays(2);
System.out.println("Yesterday's date: " + yesterday);//2019-03-11
LocalDate independenceDay = LocalDate.of(2019, Month.MARCH, 12);
DayOfWeek dayOfWeek = independenceDay.getDayOfWeek();
System.out.println("Today's day of week: " + dayOfWeek);//TUESDAY
```

Parse một LocalDate từ chuỗi cũng đơn giản như parse LocalTime. Dưới đây là ví dụ dùng `DateTimeFormatter` để parse chuỗi:

```java
    String str1 = "2014==04==12 01 giờ 06 phút 09 giây";
        // Định nghĩa formatter dùng để parse theo chuỗi ngày và giờ cần parse
        DateTimeFormatter fomatter1 = DateTimeFormatter
                .ofPattern("yyyy==MM==dd HH giờ mm phút ss giây");

        LocalDateTime dt1 = LocalDateTime.parse(str1, fomatter1);
        System.out.println(dt1); // In 2014-04-12T01:06:09

        String str2 = "2014$$$Tháng tư$$$13 20 giờ";
        DateTimeFormatter fomatter2 = DateTimeFormatter
                .ofPattern("yyy$$$MMM$$$dd HH giờ");
        LocalDateTime dt2 = LocalDateTime.parse(str2, fomatter2);
        System.out.println(dt2); // In 2014-04-13T20:00

```

Tiếp theo là một ví dụ sử dụng `DateTimeFormatter` để format ngày:

```java
LocalDateTime rightNow=LocalDateTime.now();
String date=DateTimeFormatter.ISO_LOCAL_DATE_TIME.format(rightNow);
System.out.println(date);//2019-03-12T16:26:48.29
DateTimeFormatter formatter=DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
System.out.println(formatter.format(rightNow));//2019-03-12 16:26:48
```

**🐛 Sửa lỗi (xem [issue#1157](https://github.com/Snailclimb/JavaGuide/issues/1157)):** Khi dùng `YYYY` để hiển thị năm, kết quả là năm thuộc tuần chứa thời điểm hiện tại, nên có thể sai ở tuần giao giữa hai năm. Thông thường nên dùng `yyyy` để hiển thị năm chính xác.

Ví dụ ngày hiển thị sai do giao giữa hai năm:

```java
LocalDateTime rightNow = LocalDateTime.of(2020, 12, 31, 12, 0, 0);
String date= DateTimeFormatter.ISO_LOCAL_DATE_TIME.format(rightNow);
// 2020-12-31T12:00:00
System.out.println(date);
DateTimeFormatter formatterOfYYYY = DateTimeFormatter.ofPattern("YYYY-MM-dd HH:mm:ss");
// 2021-12-31 12:00:00
System.out.println(formatterOfYYYY.format(rightNow));

DateTimeFormatter formatterOfYyyy = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
// 2020-12-31 12:00:00
System.out.println(formatterOfYyyy.format(rightNow));
```

Hình dưới đây cho thấy lỗi cụ thể rõ hơn. IDEA cũng đã đưa ra gợi ý thông minh, ưu tiên dùng `yyyy` thay vì `YYYY`.

![](https://oss.javaguide.cn/github/javaguide/java/new-features/2021042717491413.png)

### LocalDateTime (ngày giờ địa phương)

LocalDateTime biểu diễn đồng thời thời gian và ngày tháng, tương đương với việc kết hợp nội dung của hai phần trước vào một object. LocalDateTime, LocalTime và LocalDate đều là immutable. LocalDateTime cung cấp một số method để truy cập các field cụ thể.

```java
LocalDateTime sylvester = LocalDateTime.of(2014, Month.DECEMBER, 31, 23, 59, 59);

DayOfWeek dayOfWeek = sylvester.getDayOfWeek();
System.out.println(dayOfWeek);      // WEDNESDAY

Month month = sylvester.getMonth();
System.out.println(month);          // DECEMBER

long minuteOfDay = sylvester.getLong(ChronoField.MINUTE_OF_DAY);
System.out.println(minuteOfDay);    // 1439
```

Chỉ cần bổ sung thông tin timezone là có thể chuyển nó thành một object Instant biểu diễn một thời điểm. Object Instant có thể dễ dàng chuyển thành `java.util.Date` kiểu cũ.

```java
Instant instant = sylvester
        .atZone(ZoneId.systemDefault())
        .toInstant();

Date legacyDate = Date.from(instant);
System.out.println(legacyDate);     // Wed Dec 31 23:59:59 CET 2014
```

Format LocalDateTime cũng giống format thời gian và ngày tháng. Ngoài các format được định nghĩa sẵn, chúng ta cũng có thể tự định nghĩa format:

```java
DateTimeFormatter formatter =
    DateTimeFormatter
        .ofPattern("MMM dd, yyyy - HH:mm");
LocalDateTime parsed = LocalDateTime.parse("Nov 03, 2014 - 07:13", formatter);
String string = formatter.format(parsed);
System.out.println(string);     // Nov 03, 2014 - 07:13
```

Khác với `java.text.NumberFormat`, DateTimeFormatter của phiên bản mới là immutable nên thread-safe.
Thông tin chi tiết về format ngày giờ có tại [đây](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html).

## Annotation (Annotations)

Java 8 hỗ trợ repeatable annotation. Hãy xem một ví dụ để hiểu ý nghĩa của tính năng này.
Trước hết, định nghĩa một annotation wrapper Hints để chứa một nhóm annotation Hint cụ thể:

```java
@Retention(RetentionPolicy.RUNTIME)
@interface Hints {
    Hint[] value();
}
@Repeatable(Hints.class)
@interface Hint {
    String value();
}
```

Java 8 cho phép dùng annotation cùng một kiểu nhiều lần, chỉ cần đánh dấu annotation đó bằng `@Repeatable`.

Ví dụ 1: Dùng class wrapper làm container để lưu nhiều annotation (cách cũ)

```java
@Hints({@Hint("hint1"), @Hint("hint2")})
class Person {}
```

Ví dụ 2: Dùng repeatable annotation (cách mới)

```java
@Hint("hint1")
@Hint("hint2")
class Person {}
```

Trong ví dụ thứ hai, Java compiler sẽ ngầm định nghĩa annotation `@Hints` giúp bạn. Hiểu điều này sẽ hữu ích khi dùng reflection để lấy thông tin:

```java
Hint hint = Person.class.getAnnotation(Hint.class);
System.out.println(hint);                   // null
Hints hints1 = Person.class.getAnnotation(Hints.class);
System.out.println(hints1.value().length);  // 2

Hint[] hints2 = Person.class.getAnnotationsByType(Hint.class);
System.out.println(hints2.length);          // 2
```

Dù không định nghĩa annotation `@Hints` trên class `Person`, chúng ta vẫn có thể lấy annotation `@Hints` bằng `getAnnotation(Hints.class)`. Cách thuận tiện hơn là dùng `getAnnotationsByType` để lấy trực tiếp tất cả annotation `@Hint`.
Ngoài ra, annotation của Java 8 còn được mở rộng thêm cho hai target mới:

```java
@Target({ElementType.TYPE_PARAMETER, ElementType.TYPE_USE})
@interface MyAnnotation {}
```

## Tiếp theo nên đọc gì?

Phần giới thiệu về tính năng mới của Java 8 kết thúc tại đây. Chắc chắn vẫn còn nhiều tính năng đang chờ được khám phá. JDK 1.8 còn nhiều thành phần hữu ích khác như `Arrays.parallelSort`, `StampedLock` và `CompletableFuture`.

<!-- @include: @article-footer.snippet.md -->
