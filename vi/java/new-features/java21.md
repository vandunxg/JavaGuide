---
title: "Tính năng mới trong Java 21 (JDK 21): virtual thread, Generational ZGC và Sequenced Collections"
description: "Giải thích chi tiết các tính năng mới trong Java 21 (JDK 21), bao gồm virtual thread, Generational ZGC, Sequenced Collections, record pattern, pattern matching cho switch và chu kỳ hỗ trợ LTS, đồng thời giới thiệu tính năng String Templates dạng preview đã bị rút lại."
category: Java
tag:
  - Java New Features
head:
  - - meta
    - name: keywords
      content: Java 21,JDK 21,JDK21 new features,Java21 new features,LTS,virtual thread,Sequenced Collections,Generational ZGC,record pattern,pattern matching cho switch,String Templates,Foreign Function & Memory API
---

Java 21 (JDK 21) được phát hành chính thức vào ngày 19 tháng 9 năm 2023, là phiên bản hỗ trợ dài hạn (LTS) được Oracle công nhận.

Theo lộ trình hỗ trợ Java SE được Oracle cập nhật vào tháng 4 năm 2026, Premier Support của Oracle JDK 21 kéo dài đến tháng 9 năm 2028, còn Extended Support kéo dài đến tháng 9 năm 2031. Chu kỳ cập nhật miễn phí và hỗ trợ thương mại của các bản phân phối JDK khác nhau có thể khác nhau.

JDK 21 có tổng cộng 15 tính năng mới. Bài viết này sẽ chọn một số tính năng mới quan trọng để giới thiệu chi tiết:

- [JEP 430: String Templates](https://openjdk.org/jeps/430) (String Templates) (preview)
- [JEP 431: Sequenced Collections](https://openjdk.org/jeps/431) (Sequenced Collections)
- [JEP 439: Generational ZGC](https://openjdk.org/jeps/439) (Generational ZGC)
- [JEP 440: Record Patterns](https://openjdk.org/jeps/440) (record pattern)
- [JEP 441: Pattern Matching for switch](https://openjdk.org/jeps/441) (pattern matching cho switch)
- [JEP 442: Foreign Function & Memory API](https://openjdk.org/jeps/442) (Foreign Function & Memory API) (preview lần thứ ba)
- [JEP 443: Unnamed Patterns and Variables](https://openjdk.org/jeps/443) (unnamed pattern và variable) (preview)
- [JEP 444: Virtual Threads](https://openjdk.org/jeps/444) (virtual thread)
- [JEP 445: Unnamed Classes and Instance Main Methods](https://openjdk.org/jeps/445) (unnamed class và instance main method) (preview)

Nếu chủ yếu quan tâm đến việc nâng cấp trong production, bạn có thể xem trước Sequenced Collections, Generational ZGC và virtual thread. String Templates chỉ từng có bản preview trong JDK 21 và 22; proposal sau đó đã bị rút lại, nên JDK hiện tại không còn cung cấp API và syntax này.

Hình dưới đây cho biết số lượng tính năng mới và thời điểm cập nhật của từng phiên bản từ JDK 8 đến JDK 24:

![](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

## JEP 430: String Templates (String Templates, preview)

String Templates là tính năng preview trong JDK 21. Tính năng này được preview lần thứ hai trong JDK 22, sau đó bị rút lại, vì vậy JDK hiện tại không còn cung cấp API và syntax này.

String Templates cung cấp một cách ngắn gọn, trực quan hơn để xây dựng string động. Syntax preview của JDK 21 sử dụng `\{expression}` làm embedded expression và để template processor xử lý template. Expression hỗ trợ local variable, field static hoặc non-static, method call và kết quả tính toán.

Trên thực tế, String Templates tồn tại trong hầu hết ngôn ngữ lập trình:

```typescript
"Greetings {{ name }}!";  //Angular
`Greetings ${ name }!`;    //Typescript
$"Greetings { name }!"    //Visual basic
f"Greetings { name }!"    //Python
```

Trước khi Java có String Templates, chúng ta thường dùng string concatenation hoặc phương thức format để xây dựng string:

```java
//concatenation
message = "Greetings " + name + "!";

//String.format()
message = String.format("Greetings %s!", name);  //concatenation

//MessageFormat
message = new MessageFormat("Greetings {0}!").format(name);

//StringBuilder
message = new StringBuilder().append("Greetings ").append(name).append("!").toString();
```

Các cách này ít nhiều đều có một số nhược điểm, chẳng hạn khó đọc, dài dòng và phức tạp.

Java sử dụng String Templates để nối string, cho phép nhúng expression trực tiếp vào string mà không cần xử lý thêm:

```java
String message = STR."Greetings \{name}!";
```

Trong template expression ở trên:

- STR là template processor.
- `\{name}` là expression; khi runtime, các expression này sẽ được thay thế bằng giá trị của variable tương ứng.

Java hiện hỗ trợ ba template processor:

- STR: tự động thực hiện string interpolation, tức thay mỗi embedded expression trong template bằng giá trị của nó (chuyển thành string).
- FMT: tương tự STR, nhưng còn có thể nhận format specifier. Các format specifier này nằm bên trái embedded expression để kiểm soát kiểu hiển thị của output.
- RAW: không tự động xử lý String Templates như template processor STR và FMT, mà trả về một đối tượng `StringTemplate`. Đối tượng này chứa thông tin về text và expression trong template.

```java
String name = "Lokesh";

//STR
String message = STR."Greetings \{name}.";

//FMT
String message = FMT."Greetings %-12s\{name}.";

//RAW
StringTemplate st = RAW."Greetings \{name}.";
String message = STR.process(st);
```

Ngoài ba template processor có sẵn trong JDK, bạn còn có thể triển khai interface `StringTemplate.Processor` để tạo template processor riêng, chỉ cần kế thừa interface `StringTemplate.Processor`, sau đó triển khai method `process`.

Chúng ta có thể dùng local variable, field static/non-static hoặc thậm chí method làm embedded expression:

```java
//variable
message = STR."Greetings \{name}!";

//method
message = STR."Greetings \{getName()}!";

//field
message = STR."Greetings \{this.name}!";
```

Cũng có thể thực hiện phép tính trong expression và in kết quả:

```java
int x = 10, y = 20;
String s = STR."\{x} + \{y} = \{x + y}";  //"10 + 20 = 30"
```

Để tăng khả năng đọc, chúng ta có thể tách embedded expression thành nhiều dòng:

```java
String time = STR."The current time is \{
    //sample comment - current time in HH:mm:ss
    DateTimeFormatter
      .ofPattern("HH:mm:ss")
      .format(LocalTime.now())
  }.";
```

## JEP 431: Sequenced Collections (Sequenced Collections)

JDK 21 giới thiệu một nhóm interface collection mới: **Sequenced Collections**. Các collection này có thứ tự duyệt xác định (encounter order), đồng thời cung cấp method truy cập phần tử đầu và cuối collection cũng như lấy reverse view.

Sequenced Collections gồm ba interface sau:

- [`SequencedCollection`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/SequencedCollection.html)
- [`SequencedSet`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/SequencedSet.html)
- [`SequencedMap`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/SequencedMap.html)

Interface `SequencedCollection` kế thừa interface `Collection`, cung cấp method truy cập, thêm hoặc xóa phần tử ở hai đầu collection và lấy reverse view của collection.

```java
interface SequencedCollection<E> extends Collection<E> {

  // New Method

  SequencedCollection<E> reversed();

  // Promoted methods from Deque<E>

  void addFirst(E);
  void addLast(E);

  E getFirst();
  E getLast();

  E removeFirst();
  E removeLast();
}
```

Interface `List` và `Deque` kế thừa interface `SequencedCollection`.

Dưới đây dùng `ArrayList` để minh họa hiệu quả sử dụng thực tế:

```java
ArrayList<Integer> arrayList = new ArrayList<>();

arrayList.add(1);   // List contains: [1]

arrayList.addFirst(0);  // List contains: [0, 1]
arrayList.addLast(2);   // List contains: [0, 1, 2]

Integer firstElement = arrayList.getFirst();  // 0
Integer lastElement = arrayList.getLast();  // 2

List<Integer> reversed = arrayList.reversed();
System.out.println(reversed); // Prints [2, 1, 0]
```

Interface `SequencedSet` trực tiếp kế thừa interface `SequencedCollection` và override method `reversed()`.

```java
interface SequencedSet<E> extends SequencedCollection<E>, Set<E> {

    SequencedSet<E> reversed();
}
```

Interface `SortedSet` kế thừa interface `SequencedSet`, còn `LinkedHashSet` triển khai interface `SequencedSet`.

Dưới đây dùng `LinkedHashSet` để minh họa hiệu quả sử dụng thực tế:

```java
LinkedHashSet<Integer> linkedHashSet = new LinkedHashSet<>(List.of(1, 2, 3));

Integer firstElement = linkedHashSet.getFirst();   // 1
Integer lastElement = linkedHashSet.getLast();    // 3

linkedHashSet.addFirst(0);  //List contains: [0, 1, 2, 3]
linkedHashSet.addLast(4);   //List contains: [0, 1, 2, 3, 4]

System.out.println(linkedHashSet.reversed());   //Prints [4, 3, 2, 1, 0]
```

Interface `SequencedMap` kế thừa interface `Map`, cung cấp method truy cập, thêm hoặc xóa key-value pair ở hai đầu collection; lấy `SequencedSet` chứa key, `SequencedCollection` chứa value, `SequencedSet` chứa entry (key-value pair); và lấy reverse view của collection.

```java
interface SequencedMap<K,V> extends Map<K,V> {

  // New Methods

  SequencedMap<K,V> reversed();

  SequencedSet<K> sequencedKeySet();
  SequencedCollection<V> sequencedValues();
  SequencedSet<Entry<K,V>> sequencedEntrySet();

  V putFirst(K, V);
  V putLast(K, V);


  // Promoted Methods from NavigableMap<K, V>

  Entry<K, V> firstEntry();
  Entry<K, V> lastEntry();

  Entry<K, V> pollFirstEntry();
  Entry<K, V> pollLastEntry();
}
```

Interface `SortedMap` kế thừa interface `SequencedMap`, còn `LinkedHashMap` triển khai interface `SequencedMap`.

Dưới đây dùng `LinkedHashMap` để minh họa hiệu quả sử dụng thực tế:

```java
LinkedHashMap<Integer, String> map = new LinkedHashMap<>();

map.put(1, "One");
map.put(2, "Two");
map.put(3, "Three");

map.firstEntry();   //1=One
map.lastEntry();    //3=Three

System.out.println(map);  //{1=One, 2=Two, 3=Three}

Map.Entry<Integer, String> first = map.pollFirstEntry();   //1=One
Map.Entry<Integer, String> last = map.pollLastEntry();    //3=Three

System.out.println(map);  //{2=Two}

map.putFirst(1, "One");     //{1=One, 2=Two}
map.putLast(3, "Three");    //{1=One, 2=Two, 3=Three}

System.out.println(map);  //{1=One, 2=Two, 3=Three}
System.out.println(map.reversed());   //{3=Three, 2=Two, 1=One}
```

## JEP 439: Generational ZGC (Generational ZGC)

Trong JDK 21, ZGC được mở rộng tính năng và bổ sung chức năng Generational GC. Tuy nhiên, chức năng này mặc định bị tắt và cần được bật bằng cấu hình:

```bash
// Bật Generational ZGC
java -XX:+UseZGC -XX:+ZGenerational ...
```

Trong các phiên bản tương lai, official sẽ đặt ZGenerational làm giá trị mặc định, tức mặc định bật Generational GC của ZGC. Ở các phiên bản xa hơn, non-generational ZGC sẽ bị loại bỏ.

> In a future release we intend to make Generational ZGC the default, at which point -XX:-ZGenerational will select non-generational ZGC. In an even later release we intend to remove non-generational ZGC, at which point the ZGenerational option will become obsolete.
>
> Trong một phiên bản tương lai, chúng tôi dự định đặt Generational ZGC làm tùy chọn mặc định. Khi đó, -XX:-ZGenerational sẽ chọn non-generational ZGC. Ở một phiên bản xa hơn, chúng tôi dự định loại bỏ non-generational ZGC. Khi đó, tùy chọn ZGenerational sẽ trở nên obsolete.

Trong khi vẫn duy trì mục tiêu pause thấp của ZGC, Generational ZGC chủ yếu giảm rủi ro pause do allocation, giảm heap memory cần thiết và tăng throughput bằng cách thu hồi young object thường xuyên hơn.

## JEP 440: Record Patterns (record pattern)

Record pattern được preview lần đầu trong Java 19, do [JEP 405](https://openjdk.org/jeps/405) đề xuất. Trong JDK 20, đây là preview lần thứ hai, do [JEP 432](https://openjdk.org/jeps/432) đề xuất. Cuối cùng, record pattern đã chính thức trở thành tính năng trong JDK 21.

[Tổng quan tính năng mới trong Java 20](./java20.md) đã giới thiệu chi tiết về record pattern, nên phần này không lặp lại.

## JEP 441: Pattern Matching for switch (pattern matching cho switch)

Tăng cường expression và statement `switch` trong Java, cho phép sử dụng pattern trong case label. Khi pattern match, code tương ứng với case label sẽ được thực thi.

Trong code dưới đây, expression `switch` sử dụng type pattern để match.

```java
static String formatterPatternSwitch(Object obj) {
    return switch (obj) {
        case Integer i -> String.format("int %d", i);
        case Long l    -> String.format("long %d", l);
        case Double d  -> String.format("double %f", d);
        case String s  -> String.format("String %s", s);
        default        -> obj.toString();
    };
}
```

## JEP 442: Foreign Function & Memory API (Foreign Function & Memory API, preview lần thứ ba)

Java program có thể sử dụng API này để tương tác với code và data bên ngoài Java runtime. Bằng cách gọi hiệu quả external function (tức code bên ngoài JVM) và truy cập an toàn external memory (tức memory không do JVM quản lý), API này cho phép Java program gọi native library và xử lý native data mà không nguy hiểm và mong manh như JNI.

Foreign Function & Memory API trải qua vòng incubator đầu tiên trong Java 17, do [JEP 412](https://openjdk.java.net/jeps/412) đề xuất. Trong Java 18, API trải qua vòng incubator thứ hai, do [JEP 419](https://openjdk.org/jeps/419) đề xuất. Trong Java 19, đây là preview lần đầu, do [JEP 424](https://openjdk.org/jeps/424) đề xuất. Trong JDK 20, đây là preview lần thứ hai, do [JEP 434](https://openjdk.org/jeps/434) đề xuất. Trong JDK 21, đây là preview lần thứ ba, do [JEP 442](https://openjdk.org/jeps/442) đề xuất.

Trong [Tổng quan tính năng mới trong Java 19](./java19.md), tôi đã giới thiệu chi tiết về Foreign Function & Memory API, nên phần này không giới thiệu thêm.

## JEP 443: Unnamed Patterns and Variables (unnamed pattern và variable, preview)

Unnamed pattern và variable cho phép dùng dấu gạch dưới `_` để biểu thị variable không tên và component không được sử dụng trong pattern matching, nhằm tăng khả năng đọc và bảo trì code.

Trường hợp điển hình của unnamed variable là statement `try-with-resources`, exception variable trong mệnh đề `catch` và vòng lặp `for`. Khi không cần sử dụng variable, có thể dùng dấu gạch dưới `_` thay thế để biểu thị rõ variable không được sử dụng.

```java
try (var _ = ScopedContext.acquire()) {
  // No use of acquired resource
}
try { ... }
catch (Exception _) { ... }
catch (Throwable _) { ... }

for (int i = 0, _ = runOnce(); i < arr.length; i++) {
  ...
}
```

Unnamed pattern là một pattern vô điều kiện và không bind bất kỳ value nào. Unnamed pattern variable xuất hiện trong type pattern.

```java
if (r instanceof ColoredPoint(_, Color c)) { ... c ... }

switch (b) {
    case Box(RedBall _), Box(BlueBall _) -> processBox(b);
    case Box(GreenBall _)                -> stopProcessing();
    case Box(_)                          -> pickAnotherBox();
}
```

## JEP 444: Virtual Threads (virtual thread)

Virtual thread là một cập nhật lớn, cần đặc biệt chú ý!

Virtual thread được preview lần đầu trong Java 19, do [JEP 425](https://openjdk.org/jeps/425) đề xuất. Trong JDK 20, đây là preview lần thứ hai. Cuối cùng, virtual thread đã chính thức trở thành tính năng trong JDK 21.

[Tổng quan tính năng mới trong Java 20](./java20.md) đã giới thiệu chi tiết về virtual thread, nên phần này không lặp lại.

## JEP 445: Unnamed Classes and Instance Main Methods (unnamed class và instance main method, preview)

Tính năng này chủ yếu đơn giản hóa khai báo method `main`. Với người mới học Java, khai báo method `main` này đưa vào quá nhiều khái niệm syntax của Java, không thuận lợi cho việc nhanh chóng làm quen.

Định nghĩa method `main` trước khi sử dụng tính năng này:

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

Định nghĩa method `main` sau khi sử dụng tính năng mới này:

```java
class HelloWorld {
    void main() {
        System.out.println("Hello, World!");
    }
}
```

Tinh giản hơn nữa (unnamed class cho phép không định nghĩa tên class):

```java
void main() {
   System.out.println("Hello, World!");
}
```

## Tham khảo

- Java 21 String Templates: <https://howtodoinjava.com/java/java-string-templates/>
- Java 21 Sequenced Collections: <https://howtodoinjava.com/java/sequenced-collections/>
