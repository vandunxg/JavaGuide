---
title: "Tổng hợp câu hỏi phỏng vấn Java Basics thường gặp (phần cuối)"
description: "Tổng hợp câu hỏi phỏng vấn Java nâng cao: giải thích chi tiết cơ chế xử lý exception, nguyên lý generic, ứng dụng reflection, cách dùng annotation, cơ chế SPI, serialization, mô hình I/O (BIO/NIO/AIO), syntactic sugar và các kiến thức cốt lõi khác."
category: Java
tag:
  - Java Basics
head:
  - - meta
    - name: keywords
      content: Java exception, generic, reflection, annotation, SPI, serialization, I/O stream, syntactic sugar, try-with-resources, BIO NIO AIO, câu hỏi phỏng vấn Java
---

## Exception

**Tổng quan sơ đồ phân cấp exception của Java**:

![Sơ đồ phân cấp exception của Java](https://oss.javaguide.cn/github/javaguide/java/basis/types-of-exceptions-in-java.png)

### `Exception` và `Error` khác nhau thế nào?

Trong Java, mọi exception đều có tổ tiên chung là class `Throwable` trong package `java.lang`. Class `Throwable` có hai subclass quan trọng:

- **`Exception`**: exception mà bản thân chương trình có thể xử lý, có thể bắt bằng `catch`. `Exception` lại được chia thành Checked Exception (bắt buộc xử lý) và Unchecked Exception (có thể không xử lý).
- **`Error`**: `Error` thuộc nhóm lỗi mà chương trình không thể xử lý, ~~chúng ta không thể bắt bằng `catch`~~ không khuyến nghị bắt bằng `catch`. Ví dụ: lỗi khi JVM chạy (`Virtual MachineError`), lỗi thiếu bộ nhớ JVM (`OutOfMemoryError`), lỗi định nghĩa class (`NoClassDefFoundError`), v.v. Khi các lỗi này xảy ra, JVM thường sẽ chọn kết thúc thread.

### Sự khác nhau giữa `ClassNotFoundException` và `NoClassDefFoundError`

- `ClassNotFoundException` là `Exception`, xảy ra khi không tìm thấy class trong lúc dynamic class loading bằng reflection; đây là tình huống có thể dự đoán và bắt để xử lý.
- `NoClassDefFoundError` là `Error`, cho biết JVM hoặc class loader không tìm thấy định nghĩa class khi cố gắng load class. Ngoài trường hợp thiếu JAR lúc runtime, nó còn có thể do tiếp tục sử dụng class sau khi khởi tạo class thất bại. Lỗi này thường kết thúc thread hiện tại, nhưng không có nghĩa toàn bộ JVM chắc chắn không thể tiếp tục chạy.

### ⭐️ Checked Exception và Unchecked Exception khác nhau thế nào?

**Checked Exception** là exception mà code Java phải xử lý trong quá trình compile. Nếu không được xử lý bằng `catch` hoặc keyword `throws`, code sẽ không thể compile.

Ví dụ đoạn code I/O dưới đây:

![](https://oss.javaguide.cn/github/javaguide/java/basis/checked-exception.png)

Ngoài `RuntimeException` và các subclass của nó, tất cả class `Exception` khác và subclass của chúng đều là checked exception. Các checked exception thường gặp gồm exception liên quan đến I/O, `ClassNotFoundException`, `SQLException`...

**Unchecked Exception** tức **unchecked exception**. Trong quá trình compile code Java, ngay cả khi không xử lý unchecked exception, code vẫn có thể compile bình thường.

`RuntimeException` và các subclass của nó là unchecked exception; theo cách phân loại của JLS, `Error` và các subclass của nó cũng là unchecked exception. Các `RuntimeException` thường gặp:

- `NullPointerException` (lỗi null pointer)
- `IllegalArgumentException` (lỗi đối số, ví dụ kiểu tham số truyền vào method không đúng)
- `NumberFormatException` (lỗi format khi chuyển string thành number, là subclass của `IllegalArgumentException`)
- `ArrayIndexOutOfBoundsException` (lỗi vượt quá giới hạn array)
- `ClassCastException` (lỗi chuyển đổi type)
- `ArithmeticException` (lỗi số học)
- `SecurityException` (lỗi bảo mật, ví dụ không đủ quyền)
- `UnsupportedOperationException` (thao tác không được hỗ trợ, ví dụ tạo trùng một user)
- …

![](https://oss.javaguide.cn/github/javaguide/java/basis/unchecked-exception.png)

### Bạn thiên về sử dụng Checked Exception hay Unchecked Exception hơn?

Mặc định sử dụng Unchecked Exception, chỉ dùng Checked Exception khi cần thiết.

Có thể xem Unchecked Exception (ví dụ `NullPointerException`) là một code bug. Với bug, cách tốt nhất là để nó bộc lộ rồi sửa code, thay vì dùng `try-catch` để che giấu.

Thông thường, chỉ dùng Checked Exception trong một trường hợp: khi exception đó là một phần của business logic và caller bắt buộc phải xử lý nó. Ví dụ exception số dư không đủ. Đây không phải bug mà là một nhánh business bình thường; cần dùng Checked Exception để buộc caller xử lý tình huống này, chẳng hạn nhắc user nạp tiền. Như vậy vừa bảo đảm tính đầy đủ của business logic quan trọng, vừa giữ code ngắn gọn nhất có thể.

### Các method thường dùng của class `Throwable` là gì?

- `String getMessage()`: trả về thông tin chi tiết khi exception xảy ra
- `String toString()`: trả về mô tả ngắn gọn khi exception xảy ra
- `String getLocalizedMessage()`: trả về thông tin bản địa hóa của exception object. Subclass của `Throwable` có thể override method này để tạo thông tin bản địa hóa. Nếu subclass không override method này, thông tin method trả về giống với kết quả của `getMessage()`
- `void printStackTrace()`: in thông tin exception được đóng gói trong object `Throwable` ra console

### Dùng `try-catch-finally` thế nào?

- Block `try`: dùng để chứa code có thể phát sinh exception. Sau nó có thể là không hoặc nhiều block `catch`; nếu không có block `catch` thì bắt buộc phải có block `finally`.
- Block `catch`: dùng để xử lý exception được `try` bắt.
- Block `finally`: các câu lệnh trong block `finally` luôn được thực thi, bất kể exception có được bắt hoặc xử lý hay không. Khi gặp câu lệnh `return` trong block `try` hoặc `catch`, block `finally` sẽ được thực thi trước khi method return.

Ví dụ code:

```java
try {
    System.out.println("Try to do something");
    throw new RuntimeException("RuntimeException");
} catch (Exception e) {
    System.out.println("Catch Exception -> " + e.getMessage());
} finally {
    System.out.println("Finally");
}
```

Output:

```plain
Try to do something
Catch Exception -> RuntimeException
Finally
```

**Lưu ý: Không dùng `return` trong block `finally`!** Khi cả câu lệnh `try` và `finally` đều có `return`, câu lệnh `return` trong block `try` sẽ bị bỏ qua. Nguyên nhân là giá trị trả về của `return` trong câu lệnh `try` trước tiên được tạm lưu vào một biến cục bộ; khi thực thi đến `return` trong câu lệnh `finally`, giá trị của biến này sẽ trở thành giá trị trả về của `return` trong `finally`.

Ví dụ code:

```java
public static void main(String[] args) {
    System.out.println(f(2));
}

public static int f(int value) {
    try {
        return value * value;
    } finally {
        if (value == 2) {
            return 0;
        }
    }
}
```

Output:

```plain
0
```

### Code trong `finally` có luôn được thực thi không?

Không phải lúc nào cũng vậy! Trong một số tình huống, code trong `finally` sẽ không được thực thi.

Ví dụ, nếu JVM bị dừng trước `finally`, code trong `finally` sẽ không được thực thi.

```java
try {
    System.out.println("Try to do something");
    throw new RuntimeException("RuntimeException");
} catch (Exception e) {
    System.out.println("Catch Exception -> " + e.getMessage());
    // Kết thúc JVM đang chạy hiện tại
    System.exit(1);
} finally {
    System.out.println("Finally");
}
```

Output:

```plain
Try to do something
Catch Exception -> RuntimeException
```

Ngoài ra, nếu process của JVM bị buộc dừng, chẳng hạn gọi `Runtime.halt()`, operating system trực tiếp kết thúc process hoặc máy bị mất điện, block `finally` cũng có thể không kịp thực thi. Uncaught exception thông thường dù cuối cùng khiến thread hiện tại kết thúc, trước khi thread kết thúc vẫn thực thi `finally` theo quy tắc của ngôn ngữ.

Issue liên quan: <https://github.com/Snailclimb/JavaGuide/issues/190>.

🧗🏻 Nâng cao: phân tích nguyên lý triển khai đằng sau syntactic sugar `try catch finally` từ góc độ bytecode.

### Dùng `try-with-resources` thay cho `try-catch-finally` thế nào?

1. **Phạm vi áp dụng (định nghĩa resource):** mọi object implement `java.lang.AutoCloseable` hoặc `java.io.Closeable`
2. **Thứ tự đóng resource và thực thi block `finally`:** trong câu lệnh `try-with-resources`, mọi block `catch` hoặc `finally` đều chạy sau khi các resource được khai báo đã đóng

《Effective Java》 chỉ rõ:

> Với resource bắt buộc phải đóng, luôn nên ưu tiên `try-with-resources` thay vì `try-finally`. Code tạo ra sẽ ngắn hơn, rõ ràng hơn, và exception nhận được cũng hữu ích hơn. Câu lệnh `try-with-resources` giúp viết code phải đóng resource dễ hơn; nếu dùng `try-finally` thì gần như không thể làm tốt điều này.

Các resource như `InputStream`, `OutputStream`, `Scanner`, `PrintWriter` trong Java đều cần gọi method `close()` để đóng thủ công. Thông thường, ta dùng câu lệnh `try-catch-finally` để thực hiện yêu cầu này như sau:

```java
// Đọc nội dung file text
Scanner scanner = null;
try {
    scanner = new Scanner(new File("D://read.txt"));
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException e) {
    e.printStackTrace();
} finally {
    if (scanner != null) {
        scanner.close();
    }
}
```

Dùng câu lệnh `try-with-resources` sau Java 7 để cải tiến code trên:

```java
try (Scanner scanner = new Scanner(new File("test.txt"))) {
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException fnfe) {
    fnfe.printStackTrace();
}
```

Khi cần đóng nhiều resource, dùng `try-with-resources` cũng rất đơn giản; nếu vẫn dùng `try-catch-finally` có thể phát sinh nhiều vấn đề.

Có thể khai báo nhiều resource trong block `try-with-resources` bằng cách phân tách chúng bằng dấu chấm phẩy.

```java
try (BufferedInputStream bin = new BufferedInputStream(new FileInputStream(new File("test.txt")));
     BufferedOutputStream bout = new BufferedOutputStream(new FileOutputStream(new File("out.txt")))) {
    int b;
    while ((b = bin.read()) != -1) {
        bout.write(b);
    }
}
catch (IOException e) {
    e.printStackTrace();
}
```

### ⭐️ Có những lưu ý nào khi sử dụng exception?

- Không định nghĩa exception thành static variable, vì điều này khiến thông tin stack trace bị sai. Mỗi lần chủ động throw exception, cần tự new một exception object để throw.
- Thông tin exception được throw phải có ý nghĩa.
- Nên throw exception cụ thể hơn. Ví dụ khi string chuyển thành number bị lỗi format, nên throw `NumberFormatException` thay vì parent class `IllegalArgumentException`.
- Tránh ghi log trùng lặp: nếu nơi bắt exception đã ghi đủ thông tin (gồm type, error message và stack trace), khi throw lại exception này trong business code thì không nên ghi lại cùng error message. Ghi log lặp lại sẽ khiến file log phình to, đồng thời có thể che khuất nguyên nhân thực tế và khiến vấn đề khó theo dõi, giải quyết hơn.
- …

## Generic

### Generic là gì? Có tác dụng gì?

**Java generic (Generics)** là một tính năng mới được giới thiệu trong JDK 5. Dùng generic parameter có thể tăng khả năng đọc hiểu và tính ổn định của code.

Compiler có thể kiểm tra generic parameter, đồng thời generic parameter cho phép chỉ định type của object được truyền vào. Ví dụ dòng `ArrayList<Person> persons = new ArrayList<Person>()` chỉ rõ `ArrayList` này chỉ nhận object kiểu `Person`; nếu truyền object kiểu khác vào sẽ báo lỗi.

```java
ArrayList<E> extends AbstractList<E>
```

Ngoài ra, return type của raw `List` là `Object`, cần tự chuyển type mới có thể sử dụng; sau khi dùng generic, compiler sẽ tự động thực hiện việc chuyển type.

### Có những cách sử dụng generic nào?

Generic thường có ba cách sử dụng: **generic class**, **generic interface**, **generic method**.

**1. Generic class**:

```java
// T ở đây có thể viết tùy ý thành bất kỳ tên định danh nào; các parameter thường gặp như T, E, K, V thường dùng để biểu thị generic
// Khi instantiate generic class, bắt buộc phải chỉ định type cụ thể của T
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

Cách instantiate generic class:

```java
Generic<Integer> genericInteger = new Generic<Integer>(123456);
```

**2. Generic interface**:

```java
public interface Generator<T> {
    public T method();
}
```

Triển khai generic interface, không chỉ định type:

```java
class GeneratorImpl<T> implements Generator<T>{
    @Override
    public T method() {
        return null;
    }
}
```

Triển khai generic interface, chỉ định type:

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
// Tạo array của các type khác nhau: Integer, Double và Character
Integer[] intArray = { 1, 2, 3 };
String[] stringArray = { "Hello", "World" };
printArray( intArray  );
printArray( stringArray  );
```

> Lưu ý: `public static <E> void printArray(E[] inputArray)` là static generic method. Static context không có instance hiện tại, vì vậy không thể tham chiếu type parameter được khai báo ở class; điều này không liên quan đến việc “static method được load trước”. Static method có thể khai báo và sử dụng type parameter riêng `<E>`.

### Generic được dùng ở đâu trong project?

- Interface custom trả về kết quả generic `CommonResult<T>` có thể chỉ định động data type của result theo return type cụ thể thông qua parameter `T`
- Định nghĩa class xử lý `Excel` là `ExcelUtil<T>` để chỉ định động data type được `Excel` export
- Xây dựng collection utility class (tham khảo các method `sort`, `binarySearch` trong `Collections`).
- …

## ⭐️ Reflection

Để xem giải thích chi tiết về reflection, hãy đọc bài viết [Giải thích chi tiết cơ chế Java Reflection](https://javaguide.cn/java/basis/reflection.html).

### Reflection là gì?

Nói đơn giản, Java reflection (Reflection) là khả năng **động lấy thông tin về class và thao tác với class hoặc object (method, field) trong lúc chương trình runtime**.

Thông thường, type của code ta viết đã được xác định lúc compile; cần gọi method nào, truy cập field nào đều rất rõ ràng. Nhưng reflection cho phép đến **runtime** mới tìm hiểu một class có những method, field nào, constructor ra sao, đồng thời động tạo object, gọi method hoặc sửa field khi access control và module boundary cho phép.

Chính khả năng “tự quan sát” và thao tác lúc runtime này khiến reflection trở thành **nền tảng của nhiều framework và library phổ biến**. Nó giúp code linh hoạt hơn và có thể xử lý type chưa biết lúc compile.

### Ưu và nhược điểm của reflection?

**Ưu điểm:**

1. **Tính linh hoạt và tính động:** reflection cho phép chương trình dynamic load class, tạo object, gọi method và truy cập field lúc runtime; từ đó thích ứng và mở rộng hành vi của chương trình theo nhu cầu thực tế (như config file, user input, annotation, v.v.). Nhiều Java framework hiện đại (như Spring, Hibernate, MyBatis) dựa vào tính năng này để triển khai các chức năng cốt lõi như dependency injection (DI), aspect-oriented programming (AOP), object-relational mapping (ORM), xử lý annotation; có thể nói reflection là nền tảng không thể thiếu của việc phát triển framework.
2. **Giảm coupling và tính dùng chung:** thông qua reflection, có thể viết code dùng chung, tái sử dụng và tách rời tốt hơn, giảm dependency giữa các module. Ví dụ có thể dùng reflection để triển khai object copy dùng chung, serialization, Bean utility, v.v.

**Nhược điểm:**

1. **Performance overhead:** thao tác reflection thường chậm hơn gọi trực tiếp trong code. Nguyên nhân gồm dynamic type resolution, tìm method và việc tối ưu của JIT compiler bị hạn chế. Tuy nhiên, với phần lớn trường hợp dùng trong framework, mức suy giảm performance này thường có thể chấp nhận được hoặc bản thân framework sẽ cache để tối ưu.
2. **Vấn đề security:** khi đáp ứng các điều kiện như access check và quan hệ mở module, reflection có thể bypass một phần access check của ngôn ngữ Java (như truy cập field và method `private`), có thể phá vỡ encapsulation. Ngoài ra, reflection còn có thể bypass kiểm tra generic ở compile time, gây rủi ro về type safety. Từ Java 9 trở đi, module boundary có thể từ chối deep reflection access như vậy và throw `InaccessibleObjectException`.
3. **Khả năng đọc và maintainability của code:** lạm dụng reflection khiến code phức tạp, khó hiểu và khó debug. Lỗi thường chỉ bộc lộ lúc runtime, không dễ phát hiện như compile-time error.

Đọc thêm: [Java Reflection: Why is it so slow?](https://stackoverflow.com/questions/1392351/java-reflection-why-is-it-so-slow).

### Các use case của reflection?

Khi viết business code, có thể ta ít trực tiếp làm việc với Java reflection (Reflection). Nhưng có thể bạn chưa nhận ra rằng mình đang tận hưởng sự tiện lợi do reflection mang lại mỗi ngày! **Nhiều framework phổ biến như Spring/Spring Boot, MyBatis đều sử dụng rất nhiều reflection ở bên trong**, nhờ đó chúng mới linh hoạt và mạnh mẽ như vậy.

Dưới đây là một số use case phổ biến để dễ hình dung.

**1. Dependency injection và inversion of control (IoC)**

Các IoC framework tiêu biểu như Spring/Spring Boot sẽ quét các class có annotation cụ thể (như `@Component`, `@Service`, `@Repository`, `@Controller`) lúc startup, dùng reflection để khởi tạo object (Bean), rồi inject dependency bằng reflection (như `@Autowired`, constructor injection, v.v.).

**2. Xử lý annotation**

Annotation bản thân chỉ là một “marker”; phải có thành phần đọc marker đó thì mới biết cần làm gì. Reflection chính là “bộ đọc” đó. Framework dùng reflection để kiểm tra class, method, field có annotation cụ thể hay không, sau đó thực thi logic tương ứng theo thông tin annotation. Ví dụ khi thấy `@Value`, nó dùng reflection đọc nội dung annotation, tìm value tương ứng trong config file, rồi dùng reflection set value cho field.

**3. Dynamic proxy và AOP**

Muốn tự động thêm một số xử lý trước hoặc sau khi gọi method (như ghi log, mở transaction, kiểm tra quyền)? AOP (aspect-oriented programming) được dùng cho việc này, còn dynamic proxy là một cách phổ biến để triển khai AOP. Dynamic proxy có sẵn trong JDK (`Proxy` và `InvocationHandler`) phụ thuộc vào reflection. Khi proxy object gọi method của object thật bên trong, nó thực hiện thông qua `Method.invoke` của reflection.

```java
public class DebugInvocationHandler implements InvocationHandler {
    private final Object target; // object thật

    public DebugInvocationHandler(Object target) { this.target = target; }

    // proxy: proxy object, method: method được gọi, args: các argument của method
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Logic của aspect: trước khi gọi method " + method.getName());
        // Gọi method cùng tên của object thật bằng reflection
        Object result = method.invoke(target, args);
        System.out.println("Logic của aspect: sau khi gọi method " + method.getName());
        return result;
    }
}
```

**4. Object-relational mapping (ORM)**

Các framework như MyBatis, Hibernate có thể giúp bạn tự động biến từng dòng data lấy từ database thành từng Java object. Làm sao chúng biết database field tương ứng với Java field nào? Vẫn là nhờ reflection. Chúng dùng reflection lấy danh sách field của Java class, ghép query result theo name hoặc config, sau đó dùng reflection gọi setter hoặc sửa trực tiếp field value. Ngược lại, khi lưu object vào database, chúng cũng dùng reflection đọc field value để tạo SQL.

## Proxy

Để xem giới thiệu chi tiết về Java proxy, có thể đọc bài viết [Giải thích chi tiết Java Proxy Pattern](https://javaguide.cn/java/basis/proxy.html) của tác giả.

### Triển khai dynamic proxy thế nào?

Dynamic proxy là một design pattern rất mạnh, cho phép **bổ sung chức năng cho method của một class hoặc object mà không sửa source code**.

Trong Java, có hai cách phổ biến nhất để triển khai dynamic proxy: **JDK dynamic proxy** và **CGLIB dynamic proxy**.

**Cách thứ nhất: JDK dynamic proxy**

Đây là cách Java cung cấp chính thức; yêu cầu cốt lõi là target class phải implement một hoặc nhiều interface. Khi runtime, JDK dynamic proxy dùng method `Proxy.newProxyInstance()` để tạo động instance của một proxy class implement các interface này. Proxy class được generate trong memory nên bạn không nhìn thấy file `.java` hoặc `.class` của nó.

Khi gọi bất kỳ method nào của proxy object, lời gọi đó sẽ được chuyển tiếp tới method `invoke` của interface `InvocationHandler` mà ta cung cấp. Trong method `invoke`, có thể thêm enhancement logic trước hoặc sau khi gọi method gốc (target method).

**Cách thứ hai: CGLIB dynamic proxy**

CGLIB là một code generation library bên thứ ba. Nguyên lý của nó hoàn toàn khác JDK: không yêu cầu class được proxy implement interface. Khi runtime, nó dynamic generate subclass của target class làm proxy class (thông qua kỹ thuật thao tác bytecode ASM). Sau đó, nó override mọi method không phải `final`, `private` hoặc `static` trong parent class (tức class được proxy).

Khi gọi bất kỳ method nào của proxy object, lời gọi đó sẽ được method `intercept` của interface `MethodInterceptor` trong CGLIB chặn lại. Tương tự method `invoke` của `InvocationHandler`, có thể thêm enhancement logic trong method `intercept`, trước hoặc sau khi gọi method của parent class gốc.

### Static proxy và dynamic proxy khác nhau thế nào?

Khác biệt cốt lõi giữa static proxy và dynamic proxy nằm ở **thời điểm xác định quan hệ proxy, tính linh hoạt khi triển khai và chi phí bảo trì**.

| Tiêu chí so sánh                 | Static proxy (Static Proxy)                                                                                                                                       | Dynamic proxy (Dynamic Proxy)                                                                                                                               |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Thời điểm xác định quan hệ proxy | Compile time (sau compile tạo file bytecode `.class` cố định)                                                                                                     | Runtime (dynamic generate bytecode của proxy class và load vào JVM)                                                                                         |
| Cách triển khai                  | Tự viết proxy class trước khi compile, thường gọi tới target object bằng composition và delegation                                                                | Không cần tự viết proxy class cụ thể, dùng `Handler`/`Interceptor` đóng gói enhancement logic                                                               |
| Dependency vào interface         | Không bắt buộc; static proxy dựa trên interface thường khiến proxy class và target class tuân theo cùng một interface                                             | JDK dynamic proxy hướng tới interface, CGLIB và các subclass proxy khác hướng tới implementation class có thể kế thừa                                       |
| Số lượng code và maintainability | Nhiều code hơn (càng nhiều target class càng nhiều proxy class), chi phí maintenance cao; khi interface thêm method, target class và proxy class phải sửa đồng bộ | Rất ít code (enhancement logic dùng chung có thể tái sử dụng), maintainability tốt; decoupled với interface, thay đổi interface không ảnh hưởng proxy logic |
| Ưu thế cốt lõi                   | Dễ triển khai, logic trực quan, không phụ thuộc framework bổ sung                                                                                                 | Linh hoạt, tái sử dụng cao, giảm code trùng lặp, thích ứng use case phức tạp                                                                                |
| Use case điển hình               | Decorator pattern đơn giản, nhu cầu enhance một số ít class cố định                                                                                               | Spring AOP, RPC framework (như Dubbo), ORM framework                                                                                                        |

### ⭐️ JDK dynamic proxy và CGLIB dynamic proxy khác nhau thế nào?

1. JDK dynamic proxy là giải pháp chính thức, yêu cầu class được proxy phải implement interface. Nguyên lý là dynamic generate một implementation class của interface để làm proxy. CGLIB là giải pháp bên thứ ba, không cần interface. Nguyên lý là dynamic generate một subclass của class được proxy để làm proxy. Nhưng cũng vì dùng inheritance nên nó không thể proxy class `final`; method được proxy cũng không thể là `final` hoặc `private`.
2. Về performance, trong phần lớn trường hợp JDK dynamic proxy tốt hơn; khi phiên bản JDK được nâng cấp, ưu thế này càng rõ rệt.

### Hãy giới thiệu use case thực tế của dynamic proxy trong framework

Use case điển hình nhất của dynamic proxy là **Spring AOP**.

AOP (Aspect-Oriented Programming: lập trình hướng aspect) có thể đóng gói các logic hoặc trách nhiệm không liên quan đến business nhưng được các business module cùng gọi (như xử lý transaction, quản lý log, kiểm soát quyền), giúp giảm code trùng lặp trong system, giảm coupling giữa các module, đồng thời có lợi cho khả năng mở rộng và maintainability sau này.

Spring AOP dựa trên dynamic proxy. Nếu object cần proxy implement một interface, Spring AOP dùng **JDK Proxy** để tạo proxy object; còn với object không implement interface thì không thể dùng JDK Proxy để proxy. Khi đó Spring AOP dùng **Cglib** generate subclass của object được proxy để làm proxy, như hình dưới đây:

![SpringAOPProcess](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/230ae587a322d6e4d09510161987d346.jpeg)

## Annotation

### Annotation là gì?

`Annotation` là một tính năng mới được giới thiệu từ Java 5, có thể xem là một dạng comment đặc biệt, chủ yếu dùng để gắn lên class, method hoặc variable, cung cấp thông tin cho chương trình sử dụng lúc compile hoặc runtime.

Bản chất annotation là một interface đặc biệt kế thừa `Annotation`:

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {

}

public interface Override extends Annotation{

}
```

JDK cung cấp nhiều annotation built-in (như `@Override`, `@Deprecated`), đồng thời ta cũng có thể custom annotation.

### Có những cách xử lý annotation nào?

Annotation chỉ có hiệu lực sau khi được xử lý; các cách xử lý thường gặp gồm hai loại:

- **Quét trực tiếp lúc compile:** compiler quét và xử lý annotation tương ứng khi compile code Java. Ví dụ method dùng annotation `@Override`, compiler sẽ kiểm tra method hiện tại có override method tương ứng của superclass lúc compile hay không.
- **Xử lý bằng reflection lúc runtime:** các annotation có sẵn trong framework (như `@Value`, `@Component` của Spring framework) đều được xử lý bằng reflection.

## ⭐️ SPI

Để xem giải thích chi tiết về SPI, hãy đọc bài viết [Giải thích chi tiết cơ chế Java SPI](https://javaguide.cn/java/basis/spi.html).

### SPI là gì?

SPI là Service Provider Interface, nghĩa đen là “interface của service provider”. Theo cách hiểu của tôi, đây là một interface được cung cấp riêng cho service provider hoặc developer mở rộng chức năng framework sử dụng.

SPI tách service interface và service implementation cụ thể, decouple service caller với service implementer, giúp tăng khả năng mở rộng và maintainability của chương trình. Sửa hoặc thay thế service implementation không cần sửa caller.

Nhiều framework sử dụng cơ chế SPI của Java, như Spring framework, nạp database driver, logging interface và các extension implementation của Dubbo.

<img src="https://oss.javaguide.cn/github/javaguide/java/basis/spi/22e1830e0b0e4115a882751f6c417857tplv-k3u1fbpfcp-zoom-1.jpeg" style="zoom:50%;" />

### SPI và API khác nhau thế nào?

**Vậy SPI và API khác nhau thế nào?**

Nói đến SPI thì không thể không nói về API (Application Programming Interface). Theo nghĩa rộng, cả hai đều là interface và rất dễ nhầm lẫn. Trước hết hãy dùng một hình để giải thích:

![SPI VS API](https://oss.javaguide.cn/github/javaguide/java/basis/spi-vs-api.png)

Thông thường, các module giao tiếp với nhau thông qua interface, vì vậy ta đưa một “interface” vào giữa service caller và service implementer (còn gọi là service provider).

- Khi implementer cung cấp cả interface và implementation, ta có thể gọi interface của implementer để sử dụng capability mà implementer cung cấp; đó là **API**. Trong trường hợp này, interface và implementation đều nằm trong package của implementer. Caller gọi chức năng của implementer qua interface mà không cần quan tâm chi tiết implementation.
- Khi interface nằm ở phía caller, đó là **SPI**. Caller của interface xác định quy tắc interface, sau đó các vendor khác nhau implement interface theo quy tắc này để cung cấp service.

Ví dụ dễ hiểu: công ty H là một công ty công nghệ, vừa thiết kế một chip mới và hiện cần sản xuất hàng loạt, trong khi thị trường có vài công ty sản xuất chip. Khi công ty H chỉ định tiêu chuẩn sản xuất chip (định nghĩa tiêu chuẩn interface), các công ty chip hợp tác (service provider) sẽ giao chip có đặc điểm riêng theo tiêu chuẩn (cung cấp implementation của các giải pháp khác nhau nhưng cho ra cùng một kết quả).

### Ưu và nhược điểm của SPI?

Cơ chế SPI có thể tăng đáng kể tính linh hoạt của việc thiết kế interface, nhưng cũng có một số nhược điểm, chẳng hạn:

- `ServiceLoader` sẽ định vị và instantiate provider theo nhu cầu; chỉ khi caller duyệt qua toàn bộ provider để chọn implementation thì mới trigger load tất cả provider khả dụng.
- Một instance `ServiceLoader` không bảo đảm thread-safe; giữa các instance khác nhau không có quy tắc “cùng `load` thì chắc chắn conflict”.

## ⭐️ Serialization và deserialization

Để xem giải thích chi tiết về serialization và deserialization, hãy đọc bài viết [Giải thích chi tiết Java Serialization](https://javaguide.cn/java/basis/serialization.html), trong đó có kiến thức và câu hỏi phỏng vấn đầy đủ hơn.

### Serialization là gì? Deserialization là gì?

Khi cần lưu Java object, chẳng hạn lưu Java object vào file hoặc truyền Java object qua network, đều cần dùng serialization.

Nói đơn giản:

- **Serialization**: chuyển data structure hoặc object thành dạng có thể lưu trữ hoặc truyền đi, thường là binary byte stream, cũng có thể là format text như JSON, XML
- **Deserialization**: quá trình chuyển data được tạo ra trong serialization về data structure hoặc object ban đầu

Với ngôn ngữ lập trình hướng object như Java, ta serialize object, tức instance của class sau khi được khởi tạo; nhưng trong C++, một ngôn ngữ bán hướng object, `struct` (struct) định nghĩa data structure type, còn class tương ứng với object type.

Dưới đây là các use case thường gặp của serialization và deserialization:

- Trước khi object được truyền qua network (ví dụ lúc gọi remote method RPC), cần serialize object; sau khi nhận serialized object thì cần deserialize;
- Trước khi lưu object vào file cần serialize; khi đọc object từ file ra cần deserialize;
- Trước khi lưu object vào database (như Redis) cần serialize; khi đọc object từ cache database ra cần deserialize;
- Khi chuyển object thành byte representation cần lưu lâu dài hoặc truyền qua component, thường cần serialization; Java object thông thường sử dụng trong memory JVM thì không cần serialization.

Wikipedia giới thiệu serialization như sau:

> **Serialization** (serialization) trong xử lý data của computer science là quá trình chuyển data structure hoặc trạng thái object thành format có thể sử dụng (ví dụ lưu thành file, lưu trong buffer hoặc gửi qua network), để sau đó có thể khôi phục trạng thái ban đầu trong cùng hoặc một môi trường computer khác. Khi lấy lại byte theo serialization format, có thể dùng nó để tạo bản sao có cùng semantic với object gốc. Với nhiều object, chẳng hạn object phức tạp sử dụng nhiều reference, quá trình dựng lại bằng serialization không dễ dàng. Object serialization trong lập trình hướng object không bao gồm các function mà object gốc liên kết trước đó. Quá trình này còn được gọi là object marshalling. Thao tác ngược lại, trích xuất data structure từ một chuỗi byte, là deserialization (còn gọi là unmarshalling).

Tóm lại: **mục tiêu chính của serialization là chuyển object thành representation phù hợp để truyền qua network hoặc persist vào filesystem, database, cache và các medium khác.**

![](https://oss.javaguide.cn/github/javaguide/a478c74d-2c48-40ae-9374-87aacf05188c.png)

<p style="text-align:right;font-size:13px;color:gray">https://www.corejavaguru.com/java/serialization/interview-questions-1</p>

**Serialization protocol thuộc layer nào trong mô hình TCP/IP 4 layer?**

Ta biết hai bên network communication phải sử dụng và tuân thủ cùng một protocol. Mô hình TCP/IP 4 layer như sau; serialization protocol thuộc layer nào?

1. Application layer
2. Transport layer
3. Network layer
4. Network interface layer

![Mô hình TCP/IP 4 layer](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-ip-4-model.png)

Như hình trên, trong mô hình protocol 7 layer OSI, presentation layer chủ yếu xử lý và chuyển user data của application layer thành binary stream. Ngược lại, nó chuyển binary stream thành user data của application layer. Điều này tương ứng với serialization và deserialization, đúng không?

Vì application layer, presentation layer và session layer trong mô hình protocol 7 layer OSI đều tương ứng với application layer trong mô hình TCP/IP 4 layer, serialization protocol thuộc một phần của application layer trong TCP/IP protocol.

### Nếu có field không muốn serialize thì làm thế nào?

Với variable không muốn serialize, dùng keyword `transient` để khai báo.

Tác dụng của keyword `transient` là ngăn serialize các variable trong instance được đánh dấu bằng keyword này; khi object được deserialize, value của variable được đánh dấu `transient` sẽ không được persist và restore.

Ngoài ra, cần lưu ý một số điểm về `transient`:

- `transient` chỉ có thể đánh dấu variable, không thể đánh dấu class và method.
- Sau deserialization, value của variable được đánh dấu `transient` sẽ được đặt thành default value của type. Ví dụ nếu đánh dấu variable kiểu `int`, kết quả sau deserialization là `0`.
- Vì `static` variable không thuộc bất kỳ object nào, nên dù có được đánh dấu bằng keyword `transient` hay không, nó cũng không được serialize.

### Các serialization protocol thường gặp là gì?

Cơ chế serialization tích hợp trong JDK thường không được dùng vì hiệu suất serialization thấp và có vấn đề security. Các serialization protocol được dùng phổ biến hơn gồm Hessian, Kryo, Protobuf, ProtoStuff; tất cả đều là binary serialization protocol.

Các format như JSON và XML thuộc nhóm serialization dạng text. Dù khả năng đọc tốt hơn nhưng performance kém, nên thường không được chọn.

### Vì sao không khuyến nghị dùng serialization đi kèm JDK?

Ta ít, gần như không bao giờ, trực tiếp dùng cơ chế serialization tích hợp trong JDK, chủ yếu vì các nguyên nhân sau:

- **Không hỗ trợ gọi giữa các ngôn ngữ**: nếu gọi service được viết bằng ngôn ngữ khác thì không hỗ trợ.
- **Performance kém**: so với serialization framework khác, performance thấp hơn; nguyên nhân chính là byte array sau serialization có kích thước lớn, làm tăng chi phí truyền tải.
- **Có vấn đề security**: bản thân serialization và deserialization không có vấn đề. Nhưng khi input của data deserialization có thể bị user kiểm soát, attacker có thể tạo input độc hại để deserialization sinh ra object ngoài dự kiến và thực thi code tùy ý trong quá trình này. Đọc thêm: [Application security: nỗi đau từ lỗ hổng Java deserialization](https://cryin.github.io/blog/secure-development-java-deserialization-vulnerability/).

## I/O

Để xem giải thích chi tiết về I/O, hãy đọc các bài viết dưới đây; chúng có kiến thức và câu hỏi phỏng vấn đầy đủ hơn.

- [Tổng hợp kiến thức cơ bản Java IO](https://javaguide.cn/java/io/io-basis.html)
- [Tổng hợp design pattern Java IO](https://javaguide.cn/java/io/io-design-patterns.html)
- [Giải thích chi tiết mô hình Java IO](https://javaguide.cn/java/io/io-model.html)

### Bạn hiểu Java IO stream không?

IO là `Input/Output`, tức input và output. Quá trình đưa data vào memory của computer là input; ngược lại, quá trình đưa data ra external storage (như database, file, remote host) là output. Quá trình truyền data tương tự dòng nước nên được gọi là IO stream. Trong Java, IO stream được chia thành input stream và output stream; theo cách xử lý data lại chia thành byte stream và character stream.

Hơn 40 class trong Java IO đều được dẫn xuất từ 4 abstract base class sau.

- `InputStream`/`Reader`: base class của mọi input stream; loại trước là byte input stream, loại sau là character input stream.
- `OutputStream`/`Writer`: base class của mọi output stream; loại trước là byte output stream, loại sau là character output stream.

### Vì sao I/O stream phải chia thành byte stream và character stream?

Bản chất câu hỏi là: **dù đọc ghi file hay gửi nhận qua network, đơn vị lưu trữ nhỏ nhất của thông tin đều là byte, vậy tại sao thao tác I/O stream lại chia thành thao tác byte stream và thao tác character stream?**

Theo tôi, chủ yếu có hai nguyên nhân:

- Character stream được JVM chuyển đổi từ byte; quá trình này tương đối tốn thời gian;
- Nếu không biết encoding type, khi dùng byte stream rất dễ phát sinh ký tự lỗi.

### Java IO có những design pattern nào?

Tham khảo câu trả lời: [Tổng hợp design pattern Java IO](https://javaguide.cn/java/io/io-design-patterns.html)

### ⭐️ BIO, NIO và AIO khác nhau thế nào?

Tham khảo câu trả lời: [Giải thích chi tiết mô hình Java IO](https://javaguide.cn/java/io/io-model.html)

## Syntactic sugar

### Syntactic sugar là gì?

**Syntactic sugar** là một dạng cú pháp đặc biệt được ngôn ngữ lập trình thiết kế để developer thuận tiện hơn khi viết chương trình; cú pháp này không ảnh hưởng đến chức năng của ngôn ngữ lập trình. Khi triển khai cùng một chức năng, code viết dựa trên syntactic sugar thường đơn giản, ngắn gọn và dễ đọc hơn.

Ví dụ, `for-each` trong Java là một syntactic sugar thường dùng; nguyên lý thực tế của nó dựa trên vòng lặp for thông thường và iterator.

```java
String[] strs = {"JavaGuide", "Tài khoản chính thức: JavaGuide", "Blog: https://javaguide.cn/"};
for (String s : strs) {
    System.out.println(s);
}
```

Tuy nhiên, JVM thực ra không thể nhận diện syntactic sugar. Để được thực thi đúng, Java syntactic sugar trước hết phải được compiler desugar, tức chuyển thành cú pháp cơ bản mà JVM nhận diện trong giai đoạn compile chương trình. Điều này cũng cho thấy thành phần thực sự hỗ trợ syntactic sugar trong Java là Java compiler chứ không phải JVM. Nếu xem source code của `com.sun.tools.javac.main.JavaCompiler`, bạn sẽ thấy trong `compile()` có một bước gọi `desugar()`; method này chịu trách nhiệm loại bỏ syntactic sugar.

### Java có những syntactic sugar thường gặp nào?

Các syntactic sugar thường dùng nhất trong Java gồm generic, autoboxing/unboxing, varargs, enum, inner class, enhanced for loop, syntax `try-with-resources`, lambda expression, v.v.

Để xem giải thích chi tiết về các syntactic sugar này, hãy đọc bài viết [Giải thích chi tiết Java Syntactic Sugar](./syntactic-sugar.md).

<!-- @include: @article-footer.snippet.md -->
