---
title: "Giải thích chi tiết về Syntactic Sugar của Java"
description: "Phân tích chuyên sâu nguyên lý của Java Syntactic Sugar: cơ chế triển khai ở compile-time của auto boxing/unboxing, type erasure, for-each, varargs, enum, Lambda và các Syntactic Sugar khác, tránh các nhầm lẫn khi sử dụng."
category: Java
tag:
  - Java Basics
head:
  - - meta
    - name: keywords
      content: "Java Syntactic Sugar,auto boxing/unboxing,type erasure,for-each loop,varargs,enum,inner class,Lambda expression,nguyên lý Syntactic Sugar"
---

> Tác giả: Hollis
>
> Bài gốc: <https://mp.weixin.qq.com/s/o4XdEMq1DL-nBS-f8Za5Aw>

Syntactic Sugar là một chủ đề thường được hỏi trong các buổi phỏng vấn Java tại các công ty lớn.

Bài viết này đi sâu vào bytecode và class file từ góc độ nguyên lý biên dịch của Java, lần lượt làm rõ nguyên lý và cách sử dụng Syntactic Sugar trong Java, giúp bạn vừa học cách sử dụng Syntactic Sugar, vừa hiểu nguyên lý phía sau chúng.

## Syntactic Sugar là gì?

**Syntactic Sugar** còn được gọi là syntax sugar, là một thuật ngữ do nhà khoa học máy tính người Anh Peter.J.Landin đặt ra. Thuật ngữ này chỉ một dạng cú pháp được thêm vào ngôn ngữ máy tính; nó không ảnh hưởng đến chức năng của ngôn ngữ nhưng giúp lập trình viên sử dụng thuận tiện hơn. Nói ngắn gọn, Syntactic Sugar làm cho chương trình ngắn gọn hơn và dễ đọc hơn.

![](https://oss.javaguide.cn/github/javaguide/java/basis/syntactic-sugar/image-20220818175953954.png)

> Điều thú vị là trong lĩnh vực lập trình, ngoài syntax sugar còn có syntax salt và syntax saccharin; do giới hạn dung lượng nên ở đây không mở rộng thêm.

Hầu như mọi ngôn ngữ lập trình quen thuộc đều có Syntactic Sugar. Tác giả cho rằng số lượng Syntactic Sugar là một trong những tiêu chí đánh giá một ngôn ngữ có đủ mạnh hay không. Nhiều người nói Java là một “ngôn ngữ ít đường”, nhưng thực ra từ Java 7, nhiều syntax sugar đã liên tục được thêm vào ngôn ngữ Java, chủ yếu được phát triển trong dự án “Project Coin”. Dù hiện nay vẫn có người cho rằng Java là ngôn ngữ ít đường, Java sẽ tiếp tục phát triển theo hướng “nhiều đường” trong tương lai.

## Java có những Syntactic Sugar thường gặp nào?

Như đã đề cập, Syntactic Sugar chủ yếu tồn tại để giúp lập trình viên sử dụng thuận tiện hơn. Tuy nhiên, **Java Virtual Machine không hỗ trợ những Syntactic Sugar này. Chúng sẽ được chuyển về các cấu trúc cú pháp cơ bản trong giai đoạn biên dịch; quá trình này được gọi là desugar.**

Nói đến biên dịch, chắc hẳn mọi người đều biết trong ngôn ngữ Java, lệnh `javac` có thể biên dịch source file có hậu tố `.java` thành bytecode có hậu tố `.class`, có thể chạy trên Java Virtual Machine. Nếu xem source code của `com.sun.tools.javac.main.JavaCompiler`, bạn sẽ thấy trong `compile()` có một bước gọi `desugar()`; method này thực hiện việc desugar.

Các Syntactic Sugar được sử dụng phổ biến nhất trong Java chủ yếu gồm generic, varargs, conditional compilation, auto boxing/unboxing và inner class. Bài viết này chủ yếu phân tích nguyên lý phía sau các Syntactic Sugar đó, từng bước bóc lớp syntax sugar để xem bản chất của chúng.

Ở đây chúng ta sẽ dùng [decompile](https://mp.weixin.qq.com/s?__biz=MzI3NzE0NjcwMg==&mid=2650120609&idx=1&sn=5659f96310963ad57d55b48cee63c788&chksm=f36bbc80c41c3596a1e4bf9501c6280481f1b9e06d07af354474e6f3ed366fef016df673a7ba&scene=21#wechat_redirect); bạn có thể decompile class file trực tuyến bằng [Decompilers online](http://www.javadecompilers.com/).

### `switch` hỗ trợ `String` và enum

Như đã đề cập, từ Java 7, Syntactic Sugar trong ngôn ngữ Java dần phong phú hơn; một tính năng quan trọng là từ Java 7, `switch` bắt đầu hỗ trợ `String`.

Trước hết, hãy nói qua: trong Java thời kỳ đầu, `switch` hỗ trợ `byte`, `short`, `char`, `int` và các wrapper type tương ứng, nhưng không hỗ trợ `boolean`, `long`, `float`, `double`. `char` biểu diễn UTF-16 code unit, không phải kiểu ASCII. Sau đó, Java bổ sung hỗ trợ cho enum, `String` và các reference type khác.

Tiếp theo, hãy xem `switch` hỗ trợ `String` như thế nào qua đoạn code sau:

```java
public class switchDemoString {
    public static void main(String[] args) {
        String str = "world";
        switch (str) {
        case "hello":
            System.out.println("hello");
            break;
        case "world":
            System.out.println("world");
            break;
        default:
            break;
        }
    }
}
```

Nội dung sau khi decompile:

```java
public class switchDemoString
{
    public switchDemoString()
    {
    }
    public static void main(String args[])
    {
        String str = "world";
        String s;
        switch((s = str).hashCode())
        {
        default:
            break;
        case 99162322:
            if(s.equals("hello"))
                System.out.println("hello");
            break;
        case 113318802:
            if(s.equals("world"))
                System.out.println("world");
            break;
        }
    }
}
```

Từ code do phiên bản `javac` cụ thể này tạo ra và decompile, có thể thấy **`switch` trên string được chuyển thành cơ chế dispatch bằng `hashCode()` và kiểm tra bằng `equals()`.** Đây là strategy triển khai của compiler, không phải dạng bytecode bắt buộc do JLS quy định.

Quan sát kỹ có thể thấy giá trị thực sự được đưa vào `switch` là hash value, sau đó dùng method `equals` để so sánh nhằm kiểm tra an toàn. Việc kiểm tra này là cần thiết vì hash có thể xảy ra collision. Vì vậy hiệu năng của cách này thấp hơn `switch` bằng enum hoặc hằng số số nguyên thuần túy, nhưng cũng không quá tệ.

### Generic

Chúng ta đều biết nhiều ngôn ngữ hỗ trợ generic, nhưng không phải ai cũng biết rằng các compiler khác nhau xử lý generic theo những cách khác nhau. Thông thường, compiler có hai cách xử lý generic: `Code specialization` và `Code sharing`. C++ và C# sử dụng cơ chế `Code specialization`, còn Java sử dụng cơ chế `Code sharing`.

> Cách `Code sharing` tạo một biểu diễn bytecode duy nhất cho mỗi generic type, đồng thời ánh xạ các instance của generic type đó vào biểu diễn bytecode duy nhất này. Việc ánh xạ nhiều generic class instance vào một biểu diễn bytecode duy nhất được thực hiện thông qua type erasure.

Nói cách khác, **đối với Java Virtual Machine, nó hoàn toàn không nhận biết được syntax như `Map<String, String> map`. Cần thực hiện desugar bằng type erasure trong giai đoạn compile.**

Quá trình chính của type erasure như sau: 1. thay thế mọi generic parameter bằng type ở bound ngoài cùng bên trái (type của parent cao nhất); 2. loại bỏ toàn bộ type parameter.

Code sau:

```java
Map<String, String> map = new HashMap<String, String>();
map.put("name", "hollis");
map.put("wechat", "Hollis");
map.put("blog", "www.hollischuang.com");
```

Sau khi desugar sẽ trở thành:

```java
Map map = new HashMap();
map.put("name", "hollis");
map.put("wechat", "Hollis");
map.put("blog", "www.hollischuang.com");
```

Code sau:

```java
public static <A extends Comparable<A>> A max(Collection<A> xs) {
    Iterator<A> xi = xs.iterator();
    A w = xi.next();
    while (xi.hasNext()) {
        A x = xi.next();
        if (w.compareTo(x) < 0)
            w = x;
    }
    return w;
}
```

Sau khi type erasure sẽ trở thành:

```java
 public static Comparable max(Collection xs){
    Iterator xi = xs.iterator();
    Comparable w = (Comparable)xi.next();
    while(xi.hasNext())
    {
        Comparable x = (Comparable)xi.next();
        if(w.compareTo(x) < 0)
            w = x;
    }
    return w;
}
```

**Trong Virtual Machine không có generic, chỉ có class và method thông thường. Type parameter của mọi generic class đều bị xóa trong quá trình compile; generic class không có `Class` object riêng. Ví dụ không tồn tại `List<String>.class` hay `List<Integer>.class`, mà chỉ có `List.class`.**

### Auto boxing và unboxing

Auto boxing là việc Java tự động chuyển giá trị của primitive type thành object tương ứng, ví dụ chuyển biến kiểu `int` thành object `Integer`; quá trình ngược lại, chuyển object `Integer` thành giá trị kiểu `int`, được gọi là unboxing. Vì boxing và unboxing ở đây được thực hiện tự động nên được gọi là auto boxing và unboxing. Wrapper class tương ứng với primitive type `byte`, `short`, `char`, `int`, `long`, `float`, `double` và `boolean` lần lượt là `Byte`, `Short`, `Character`, `Integer`, `Long`, `Float`, `Double`, `Boolean`.

Trước hết, hãy xem code auto boxing:

```java
 public static void main(String[] args) {
    int i = 10;
    Integer n = i;
}
```

Code sau khi decompile:

```java
public static void main(String args[])
{
    int i = 10;
    Integer n = Integer.valueOf(i);
}
```

Tiếp theo là code auto unboxing:

```java
public static void main(String[] args) {

    Integer i = 10;
    int n = i;
}
```

Code sau khi decompile:

```java
public static void main(String args[])
{
    Integer i = Integer.valueOf(10);
    int n = i.intValue();
}
```

Từ nội dung sau khi decompile có thể thấy khi boxing, Java tự động gọi method `valueOf(int)` của `Integer`; khi unboxing, Java tự động gọi method `intValue` của `Integer`.

Vì vậy, **quá trình boxing được thực hiện bằng cách gọi method `valueOf` của wrapper, còn quá trình unboxing được thực hiện bằng cách gọi method `xxxValue` của wrapper.**

### Varargs

Varargs (`variable arguments`) là một tính năng được đưa vào Java 1.5. Nó cho phép một method nhận một số lượng giá trị bất kỳ làm tham số.

Hãy xem đoạn code varargs dưới đây; method `print` nhận các tham số biến đổi:

```java
public static void main(String[] args)
    {
        print("Holis", "Kênh công khai:Hollis", "Blog:www.hollischuang.com", "QQ:907607222");
    }

public static void print(String... strs)
{
    for (int i = 0; i < strs.length; i++)
    {
        System.out.println(strs[i]);
    }
}
```

Code sau khi decompile:

```java
 public static void main(String args[])
{
    print(new String[] {
        "Holis", "Kênh công khai:Hollis", "Blog:www.hollischuang.com", "QQ:907607222"
    });
}

public static transient void print(String strs[])
{
    for(int i = 0; i < strs.length; i++)
        System.out.println(strs[i]);

}
```

Code sau khi decompile cho thấy khi được sử dụng, varargs trước tiên tạo một array có độ dài bằng số lượng argument thực tế được truyền khi gọi method, sau đó đưa toàn bộ parameter value vào array này và truyền array đó làm parameter cho method được gọi. (Lưu ý: `transient` chỉ có ý nghĩa khi dùng làm modifier cho member variable. Việc “modifier method” ở đây là do trong javassist, cùng một value được dùng để biểu diễn cả `transient` và `vararg`, xem [tại đây](https://github.com/jboss-javassist/javassist/blob/7302b8b0a09f04d344a26ebe57f29f3db43f2a3e/src/main/javassist/bytecode/AccessFlag.java#L32).)

### Enum

Java SE5 cung cấp một type mới, enum type của Java. Keyword `enum` có thể tạo một type mới từ một tập hợp hữu hạn các value có tên, và những value có tên này có thể được sử dụng như các thành phần thông thường của chương trình. Đây là một tính năng rất hữu ích.

Để xem source code, trước hết phải có một class. Vậy enum type thực chất là class nào? Có phải là `enum` không? Câu trả lời rõ ràng là không. `enum` cũng giống `class`, chỉ là một keyword chứ không phải class. Vậy enum do class nào quản lý? Hãy viết một enum đơn giản:

```java
public enum t {
    SPRING,SUMMER;
}
```

Sau đó dùng decompile để xem đoạn code này được triển khai như thế nào. Nội dung sau khi decompile:

```java
// Tên nhị phân của enum vẫn là t; Java phân biệt chữ hoa chữ thường
public final class t extends Enum
{
    private t(String s, int i)
    {
        super(s, i);
    }
    public static t[] values()
    {
        t at[];
        int i;
        t at1[];
        System.arraycopy(at = ENUM$VALUES, 0, at1 = new t[i = at.length], 0, i);
        return at1;
    }

    public static t valueOf(String s)
    {
        return (t) Enum.valueOf(t.class, s);
    }

    public static final t SPRING;
    public static final t SUMMER;
    private static final t ENUM$VALUES[];
    static
    {
        SPRING = new t("SPRING", 0);
        SUMMER = new t("SUMMER", 1);
        ENUM$VALUES = (new t[] {
            SPRING, SUMMER
        });
    }
}
```

Từ code sau khi decompile có thể thấy `public final class t extends Enum`, cho biết class này kế thừa class `Enum`; enum này không chứa constant-specific class body nên nó ngầm định là `final`.

**Enum class được định nghĩa bằng `enum` sẽ trực tiếp kế thừa `Enum`, vì vậy không thể explicit kế thừa class khác và cũng không thể được class thông thường kế thừa. Enum class không có constant-specific class body sẽ ngầm định là `final`; chỉ cần có enum constant khai báo constant-specific class body, enum class sẽ ngầm định là `sealed`, còn các class body riêng đó tương ứng với các anonymous subclass được cấp quyền.**

### Inner class

Inner class còn được gọi là nested class; có thể hiểu inner class như một member thông thường của outer class.

**Inner class cũng là Syntactic Sugar vì nó chỉ là một khái niệm ở compile-time. Trong `outer.java` định nghĩa một inner class `inner`; sau khi compile thành công sẽ tạo ra hai `.class` file hoàn toàn khác nhau là `outer.class` và `outer$inner.class`. Tuy nhiên, JLS nghiêm cấm nested class có cùng simple name với bất kỳ enclosing class hoặc interface nào.**

```java
public class OuterClass {
    private String userName;

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public static void main(String[] args) {

    }

    class InnerClass{
        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }
}
```

Sau khi biên dịch code trên sẽ tạo ra hai class file: `OuterClass$InnerClass.class`, `OuterClass.class`. Khi thử decompile file `OuterClass.class`, dòng lệnh sẽ in nội dung sau: `Parsing OuterClass.class...Parsing inner class OuterClass$InnerClass.class... Generating OuterClass.jad`. Nó sẽ decompile cả hai file rồi tạo một file `OuterClass.jad`. Nội dung file như sau:

```java
public class OuterClass
{
    class InnerClass
    {
        public String getName()
        {
            return name;
        }
        public void setName(String name)
        {
            this.name = name;
        }
        private String name;
        final OuterClass this$0;

        InnerClass()
        {
            this.this$0 = OuterClass.this;
            super();
        }
    }

    public OuterClass()
    {
    }
    public String getUserName()
    {
        return userName;
    }
    public void setUserName(String userName){
        this.userName = userName;
    }
    public static void main(String args1[])
    {
    }
    private String userName;
}
```

**Vì sao inner class có thể sử dụng thuộc tính `private` của outer class**:

Trong `InnerClass`, thêm một method để in thuộc tính `userName` của outer class:

```java
// Lược bỏ các thuộc tính khác
public class OuterClass {
    private String userName;
    ......
    class InnerClass{
    ......
        public void printOut(){
            System.out.println("Username from OuterClass:"+userName);
        }
    }
}

// Khi đó, dùng lệnh javap -p để decompile OuterClass:
public classOuterClass {
    private String userName;
    ......
    static String access$000(OuterClass);
}
// Khi đó, kết quả decompile của InnerClass:
class OuterClass$InnerClass {
    final OuterClass this$0;
    ......
    public void printOut();
}

```

Thực tế, sau khi biên dịch, bên trong `inner` instance thường có một tham chiếu trỏ tới `outer` instance là `this$0`. Trong class file được tạo bởi JDK 10 và các phiên bản cũ hơn, compiler thường dùng synthetic access method tương tự `access$000` để triển khai việc truy cập thành viên `private` giữa các nested class, vì vậy method `printOut()` sau khi decompile đại khái như sau. Từ JDK 11, nest-based access control được đưa vào; các class trong cùng một nest có thể trực tiếp truy cập thành viên `private` của nhau, nên thường không còn cần các synthetic access method này:

```java
public void printOut() {
    System.out.println("Username from OuterClass:" + OuterClass.access$000(this.this$0));
}
```

Bổ sung:

1. Trong output `javac` điển hình của JDK 10 và các phiên bản cũ hơn, anonymous inner class, local inner class và static inner class cũng có thể lấy thuộc tính `private` thông qua synthetic access method; từ JDK 11 thường sử dụng nest-based access control.
2. Static inner class không có tham chiếu `this$0`.
3. Anonymous inner class và local inner class dùng bản sao của biến cục bộ; sau khi biến được khởi tạo thì không thể sửa nó. Ví dụ:

```java
public class OuterClass {
    private String userName;

    public void test(){
        // Sau khi i được khởi tạo bằng 1 thì không thể sửa nó
        int i=1;
        class Inner{
            public void printName(){
                System.out.println(userName);
                System.out.println(i);
            }
        }
    }
}
```

Sau khi decompile:

```java
// Kết quả decompile Inner bằng lệnh javap
// i được copy vào inner class và là final
class OuterClass$1Inner {
  final int val$i;
  final OuterClass this$0;
  OuterClass$1Inner();
  public void printName();
}

```

### Conditional compilation

Thông thường, mọi dòng code trong chương trình đều tham gia biên dịch. Nhưng đôi khi, để tối ưu code, chúng ta chỉ muốn biên dịch một phần nội dung. Khi đó cần thêm điều kiện vào chương trình để compiler chỉ biên dịch code thỏa mãn điều kiện và loại bỏ code không thỏa mãn. Đây là conditional compilation.

Ví dụ trong C hoặc CPP, có thể thực hiện conditional compilation bằng preprocessor statement. Trong Java cũng có thể thực hiện conditional compilation. Hãy xem một đoạn code:

```java
public class ConditionalCompilation {
    public static void main(String[] args) {
        final boolean DEBUG = true;
        if(DEBUG) {
            System.out.println("Hello, DEBUG!");
        }

        final boolean ONLINE = false;

        if(ONLINE){
            System.out.println("Hello, ONLINE!");
        }
    }
}
```

Code sau khi decompile:

```java
public class ConditionalCompilation
{

    public ConditionalCompilation()
    {
    }

    public static void main(String args[])
    {
        boolean DEBUG = true;
        System.out.println("Hello, DEBUG!");
        boolean ONLINE = false;
    }
}
```

Trước hết có thể thấy trong code sau khi decompile không có `System.out.println("Hello, ONLINE!");`; đây chính là conditional compilation. Khi `if(ONLINE)` là false, compiler không biên dịch code bên trong nó.

Vì vậy, **conditional compilation trong cú pháp Java được thực hiện bằng statement `if` có condition là constant. Nguyên lý của nó cũng là Syntactic Sugar của Java. Dựa vào giá trị true/false của condition trong `if`, compiler trực tiếp loại bỏ code block có branch là false. Conditional compilation theo cách này phải được thực hiện trong method body, không thể thực hiện trên structure của toàn bộ Java class hoặc thuộc tính của class; so với conditional compilation của C/C++, đây thực sự là một hạn chế. Java không đưa conditional compilation vào ngay từ khi thiết kế ngôn ngữ; dù có hạn chế, vẫn tốt hơn là không có.**

### Assertion

Trong Java, keyword `assert` được đưa vào từ JAVA SE 1.4. Để tránh lỗi với code Java phiên bản cũ đã sử dụng keyword `assert`, Java mặc định không bật assertion check khi chạy (khi đó mọi assertion statement đều bị bỏ qua). Muốn bật assertion check, cần dùng switch `-enableassertions` hoặc `-ea`.

Hãy xem đoạn code có assertion:

```java
public class AssertTest {
    public static void main(String args[]) {
        int a = 1;
        int b = 1;
        assert a == b;
        System.out.println("Kênh công khai: Hollis");
        assert a != b : "Hollis";
        System.out.println("Blog: www.hollischuang.com");
    }
}
```

Code sau khi decompile:

```java
public class AssertTest {
   public AssertTest()
    {
    }
    public static void main(String args[])
{
    int a = 1;
    int b = 1;
    if(!$assertionsDisabled && a != b)
        throw new AssertionError();
    System.out.println("Kênh công khai: Hollis");
    if(!$assertionsDisabled && a == b)
    {
        throw new AssertionError("Hollis");
    } else
    {
        System.out.println("Blog: www.hollischuang.com");
        return;
    }
}

static final boolean $assertionsDisabled = !com/hollis/suguar/AssertTest.desiredAssertionStatus();

}
```

Rõ ràng code sau khi decompile phức tạp hơn code do chúng ta tự viết rất nhiều. Vì vậy, dùng Syntactic Sugar `assert` giúp tiết kiệm nhiều code. **Thực chất, triển khai tầng dưới của assertion là statement `if`: nếu kết quả assertion là true thì không làm gì và chương trình tiếp tục chạy; nếu kết quả assertion là false thì chương trình ném `AssertionError` để ngắt việc thực thi.** `-enableassertions` sẽ thiết lập value của field `$assertionsDisabled`.

### Numeric literal

Trong Java 7, numeric literal, dù là integer hay floating-point number, đều cho phép chèn tùy ý số lượng underscore giữa các chữ số. Những underscore này không ảnh hưởng đến value của literal mà chỉ giúp dễ đọc hơn.

Ví dụ:

```java
public class Test {
    public static void main(String... args) {
        int i = 10_000;
        System.out.println(i);
    }
}
```

Sau khi decompile:

```java
public class Test
{
  public static void main(String[] args)
  {
    int i = 10000;
    System.out.println(i);
  }
}
```

Không thấy `_` trong kết quả decompile vì nó chỉ là một phần của literal syntax trong source code, không ảnh hưởng đến number và không được ghi vào class file. **Compiler phải nhận diện và kiểm tra vị trí của `_`, sau đó ghi value của literal vào bytecode.**

### For-each

Enhanced for loop (`for-each`) chắc hẳn không còn xa lạ; nó thường được sử dụng trong phát triển hằng ngày và cần viết ít code hơn for loop. Vậy Syntactic Sugar này được triển khai như thế nào?

```java
public static void main(String... args) {
    String[] strs = {"Hollis", "Kênh công khai: Hollis", "Blog: www.hollischuang.com"};
    for (String s : strs) {
        System.out.println(s);
    }
    List<String> strList = ImmutableList.of("Hollis", "Kênh công khai: Hollis", "Blog: www.hollischuang.com");
    for (String s : strList) {
        System.out.println(s);
    }
}
```

Code sau khi decompile:

```java
public static transient void main(String args[])
{
    String strs[] = {
        "Hollis", "Kênh công khai: Hollis", "Blog: www.hollischuang.com"
    };
    String args1[] = strs;
    int i = args1.length;
    for(int j = 0; j < i; j++)
    {
        String s = args1[j];
        System.out.println(s);
    }

    List strList = ImmutableList.of("Hollis", "Kênh công khai: Hollis", "Blog: www.hollischuang.com");
    String s;
    for(Iterator iterator = strList.iterator(); iterator.hasNext(); System.out.println(s))
        s = (String)iterator.next();

}
```

Code rất đơn giản: **nguyên lý triển khai của for-each thực chất là sử dụng for loop thông thường và iterator.**

### Try-with-resources

Trong Java, với các tài nguyên tốn kém như IO stream dùng để thao tác với file và database connection, sau khi sử dụng phải kịp thời đóng chúng bằng method `close`; nếu không, tài nguyên sẽ luôn ở trạng thái mở và có thể gây ra các vấn đề như memory leak.

Cách thường dùng để đóng tài nguyên là giải phóng trong block `finally`, tức gọi method `close`. Ví dụ, chúng ta thường viết code như sau:

```java
public static void main(String[] args) {
    BufferedReader br = null;
    try {
        String line;
        br = new BufferedReader(new FileReader("d:\\hollischuang.xml"));
        while ((line = br.readLine()) != null) {
            System.out.println(line);
        }
    } catch (IOException e) {
        // handle exception
    } finally {
        try {
            if (br != null) {
                br.close();
            }
        } catch (IOException ex) {
            // handle exception
        }
    }
}
```

Từ Java 7, JDK cung cấp một cách tốt hơn để đóng tài nguyên: dùng statement `try-with-resources`. Viết lại code trên như sau:

```java
public static void main(String... args) {
    try (BufferedReader br = new BufferedReader(new FileReader("d:\\ hollischuang.xml"))) {
        String line;
        while ((line = br.readLine()) != null) {
            System.out.println(line);
        }
    } catch (IOException e) {
        // handle exception
    }
}
```

Đây đúng là một cải tiến lớn. Trước đây tôi thường dùng `IOUtils` để đóng stream, không viết nhiều code trong `finally`, nhưng Syntactic Sugar mới này trông có vẻ thanh lịch hơn nhiều. Hãy xem phần bên trong:

```java
public static transient void main(String args[])
    {
        BufferedReader br;
        Throwable throwable;
        br = new BufferedReader(new FileReader("d:\\ hollischuang.xml"));
        throwable = null;
        String line;
        try
        {
            while((line = br.readLine()) != null)
                System.out.println(line);
        }
        catch(Throwable throwable2)
        {
            throwable = throwable2;
            throw throwable2;
        }
        finally
        {
            if(br != null)
                if(throwable != null)
                    try
                    {
                        br.close();
                    }
                    catch(Throwable throwable1)
                    {
                        throwable.addSuppressed(throwable1);
                    }
                else
                    br.close();
        }
    }
}
```

**Nguyên lý phía sau cũng rất đơn giản: các thao tác đóng tài nguyên mà chúng ta không viết đã được compiler thực hiện thay. Điều này một lần nữa khẳng định tác dụng của Syntactic Sugar là giúp lập trình viên sử dụng thuận tiện hơn, nhưng cuối cùng vẫn phải chuyển thành cú pháp mà compiler nhận biết.**

### Lambda expression

Về lambda expression, có người có thể nghi ngờ vì trên Internet có người nói nó không phải Syntactic Sugar. Thực ra cần đính chính cách nói này. **Lambda expression không phải Syntactic Sugar của anonymous inner class, nhưng nó cũng là một Syntactic Sugar. Cách triển khai thực tế dựa vào một số API liên quan đến lambda được cung cấp ở tầng dưới của JVM.**

Trước hết, hãy xem một lambda expression đơn giản, duyệt một list:

```java
public static void main(String... args) {
    List<String> strList = ImmutableList.of("Hollis", "Kênh công khai: Hollis", "Blog: www.hollischuang.com");

    strList.forEach( s -> { System.out.println(s); } );
}
```

Vì sao nói nó không phải Syntactic Sugar của inner class? Như đã nói ở phần inner class, sau khi biên dịch inner class sẽ có hai class file, nhưng class chứa lambda expression sau khi biên dịch chỉ tạo ra một file.

Code sau khi decompile:

```java
public static /* varargs */ void main(String ... args) {
    ImmutableList strList = ImmutableList.of((Object)"Hollis", (Object)"Kênh công khai: Hollis", (Object)"Blog: www.hollischuang.com");
    strList.forEach((Consumer<String>)LambdaMetafactory.metafactory(null, null, null, (Ljava/lang/Object;)V, lambda$main$0(java.lang.String ), (Ljava/lang/String;)V)());
}

private static /* synthetic */ void lambda$main$0(String s) {
    System.out.println(s);
}
```

Có thể thấy trong method `forEach`, thực tế gọi method `java.lang.invoke.LambdaMetafactory#metafactory`; parameter thứ tư của method này là `implMethod`, chỉ định method implementation. Có thể thấy ở đây thực tế gọi method `lambda$main$0` để xuất kết quả.

Tiếp theo là ví dụ phức tạp hơn một chút: trước hết lọc `List`, sau đó xuất kết quả:

```java
public static void main(String... args) {
    List<String> strList = ImmutableList.of("Hollis", "Kênh công khai: Hollis", "Blog: www.hollischuang.com");

    List HollisList = strList.stream().filter(string -> string.contains("Hollis")).collect(Collectors.toList());

    HollisList.forEach( s -> { System.out.println(s); } );
}
```

Code sau khi decompile:

```java
public static /* varargs */ void main(String ... args) {
    ImmutableList strList = ImmutableList.of((Object)"Hollis", (Object)"Kênh công khai: Hollis", (Object)"Blog: www.hollischuang.com");
    List<Object> HollisList = strList.stream().filter((Predicate<String>)LambdaMetafactory.metafactory(null, null, null, (Ljava/lang/Object;)Z, lambda$main$0(java.lang.String ), (Ljava/lang/String;)Z)()).collect(Collectors.toList());
    HollisList.forEach((Consumer<Object>)LambdaMetafactory.metafactory(null, null, null, (Ljava/lang/Object;)V, lambda$main$1(java.lang.Object ), (Ljava/lang/Object;)V)());
}

private static /* synthetic */ void lambda$main$1(Object s) {
    System.out.println(s);
}

private static /* synthetic */ boolean lambda$main$0(String string) {
    return string.contains("Hollis");
}
```

Hai lambda expression lần lượt gọi method `lambda$main$1` và `lambda$main$0`.

**Vì vậy, cách triển khai lambda expression thực tế dựa vào một số API tầng dưới. Trong giai đoạn biên dịch, compiler sẽ desugar lambda expression thành cách gọi các API bên trong.**

## Các vấn đề có thể gặp

### Generic

**1. Khi generic gặp overload**

```java
public class GenericTypes {

    public static void method(List<String> list) {
        System.out.println("invoke method(List<String> list)");
    }

    public static void method(List<Integer> list) {
        System.out.println("invoke method(List<Integer> list)");
    }
}
```

Đoạn code trên có hai method overload vì parameter type khác nhau: một là `List<String>`, một là `List<Integer>`. Tuy nhiên, code này không thể biên dịch thành công. Như đã nói, sau khi biên dịch, parameter `List<Integer>` và `List<String>` đều bị type erasure, trở thành cùng raw type `List`; thao tác erase khiến chữ ký của hai method hoàn toàn giống nhau.

**2. Khi generic gặp `catch`**

Generic type parameter không thể được dùng trong statement `catch` của Java exception handling vì exception handling do JVM thực hiện tại runtime. Do type information đã bị xóa, JVM không thể phân biệt hai exception type `MyException<String>` và `MyException<Integer>`.

**3. Khi generic chứa static variable**

```java
public class StaticTest{
    public static void main(String[] args){
        GT<Integer> gti = new GT<Integer>();
        gti.var=1;
        GT<String> gts = new GT<String>();
        gts.var=2;
        System.out.println(gti.var);
    }
}
class GT<T>{
    public static int var=0;
    public void nothing(T x){}
}
```

Kết quả in ra của code trên là: 2!

Một số bạn có thể nhầm rằng generic class là các class khác nhau, tương ứng với các bytecode khác nhau. Thực tế, do type erasure, mọi generic class instance đều liên kết với cùng một biểu diễn bytecode; static variable của generic class được dùng chung. `GT<Integer>.var` và `GT<String>.var` trong ví dụ trên thực chất là cùng một biến.

### Auto boxing và unboxing

**So sánh tính bằng nhau của object**

```java
public static void main(String[] args) {
    Integer a = 1000;
    Integer b = 1000;
    Integer c = 100;
    Integer d = 100;
    System.out.println("a == b is " + (a == b));
    System.out.println(("c == d is " + (c == d)));
}
```

Kết quả output:

```plain
a == b is false
c == d is true
```

Trong Java 5, một feature mới được đưa vào thao tác với `Integer` để tiết kiệm memory và cải thiện performance. Integer object được cache và reuse bằng cách sử dụng cùng một object reference.

> Áp dụng cho khoảng integer từ -128 đến +127.
>
> Chỉ áp dụng cho auto boxing, không áp dụng khi tạo object bằng constructor.

### Enhanced for loop

```java
for (Student stu : students) {
    if (stu.getId() == 2)
        students.remove(stu);
}
```

Code này sẽ ném exception `ConcurrentModificationException`.

Ở đây liên quan đến cơ chế **fail-fast (báo lỗi nhanh)** của collection. Lấy `ArrayList` làm ví dụ, bên trong nó duy trì một counter `modCount`; mỗi lần cấu trúc collection bị thay đổi (chẳng hạn add hoặc delete), counter này tăng lên. Khi tạo `Iterator`, `modCount` hiện tại được ghi lại thành `expectedModCount`. Mỗi lần gọi `next()`, `Iterator` kiểm tra `modCount` có bằng `expectedModCount` hay không. Nếu không bằng, nghĩa là collection đã bị sửa bằng cách khác trong lúc duyệt, nên sẽ ném exception `java.util.ConcurrentModificationException`.

Vì vậy, khi `Iterator` hoạt động, object đang được duyệt không được phép thay đổi. Tuy nhiên, bạn có thể dùng method `remove()` của chính `Iterator` để xóa object. Method `Iterator.remove()` sẽ cập nhật đồng thời `expectedModCount` sau khi xóa element, từ đó tránh kích hoạt exception này.

## Tổng kết

Phần trên đã giới thiệu 12 Syntactic Sugar thường dùng trong Java. Syntactic Sugar chỉ là một dạng cú pháp được cung cấp để lập trình viên phát triển thuận tiện hơn. Tuy nhiên, cú pháp này chỉ lập trình viên nhận biết được. Muốn được thực thi, nó phải được desugar, tức chuyển thành cú pháp mà JVM nhận biết. Khi desugar những Syntactic Sugar này, bạn sẽ thấy các cú pháp thuận tiện thường dùng hằng ngày thực chất đều được cấu thành từ những cú pháp khác đơn giản hơn.

Nhờ các Syntactic Sugar này, năng suất phát triển hằng ngày có thể được cải thiện đáng kể, nhưng cũng cần tránh lạm dụng. Tốt nhất nên tìm hiểu nguyên lý trước khi sử dụng để tránh gặp vấn đề.

<!-- @include: @article-footer.snippet.md -->
