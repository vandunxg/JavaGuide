---
title: "Tổng hợp Java keyword"
description: "Tổng hợp có hệ thống các Java keyword thường dùng: giải thích chi tiết cách dùng và điểm khác nhau của các keyword như final, static, this, super, volatile, transient, synchronized, giúp Java developer nắm vững syntax cốt lõi."
category: Java
tag:
  - Java Basics
head:
  - - meta
    - name: keywords
      content: Java keywords,final keyword,static keyword,this keyword,super keyword,volatile,transient,synchronized
---

# Tổng hợp keyword final, static, this, super

## Keyword final

**Keyword final có nghĩa là cuối cùng, không thể sửa đổi, được dùng làm modifier cho class, method và variable, với các đặc điểm sau:**

1. Class được modifier bởi final không thể được kế thừa; mọi member method trong final class sẽ được ngầm định chỉ định là final method;

2. Method được modifier bởi final không thể bị override;

3. Variable được modifier bởi final chỉ có thể được gán một lần. Nếu là variable có primitive data type, value của nó không thể thay đổi sau khi khởi tạo; nếu là variable kiểu reference, sau khi khởi tạo không thể trỏ tới object khác, nhưng bản thân object được reference vẫn có thể thay đổi. Chỉ final variable thỏa điều kiện “constant variable” của JLS mới là compile-time constant.

Giải thích: Có hai lý do sử dụng final method:

1. Khóa method lại để ngăn mọi subclass sửa đổi ý nghĩa của nó;
2. Hiệu năng. Trong các phiên bản Java thời kỳ đầu, final method được chuyển thành inline call. Tuy nhiên, nếu method quá lớn, có thể không thấy bất kỳ cải thiện performance nào từ inline call (các phiên bản Java hiện nay không còn cần dùng final method cho những tối ưu hóa này).

## Keyword static

**Keyword static chủ yếu có bốn trường hợp sử dụng sau:**

1. **Modifier member variable và member method:** Member được modifier bởi static thuộc về class, không thuộc về một object cụ thể nào của class đó, được mọi object trong class chia sẻ, có thể và nên được gọi thông qua tên class. Vị trí lưu trữ cụ thể của static variable là chi tiết triển khai của JVM; với HotSpot từ JDK 8 trở đi, class metadata nằm trong metaspace của native memory, còn static variable của class nằm trên Java heap. Cú pháp gọi: `ClassName.staticVariableName` `ClassName.staticMethod()`
2. **Static code block:** Static code block được định nghĩa bên ngoài method, bên trong class; static code block thực thi trước non-static code block (static code block -> non-static code block -> constructor). Dù class tạo bao nhiêu object, static code block cũng chỉ thực thi một lần.
3. **Static inner class (class được modifier bởi static chỉ có thể là inner class):** Giữa static inner class và non-static inner class có một khác biệt lớn nhất: sau khi compile xong, non-static inner class ngầm lưu một reference trỏ tới outer class tạo ra nó, còn static inner class thì không. Không có reference này nghĩa là: 1. Việc tạo nó không cần phụ thuộc vào việc tạo outer class. 2. Nó không thể sử dụng bất kỳ non-static member variable và method nào của outer class.
4. **Static import (dùng để import static resource trong class, là tính năng mới từ 1.5):** Format là `import static`. Hai keyword này khi dùng cùng nhau có thể chỉ định import static resource cụ thể trong một class; không cần dùng tên class để gọi static member trong class mà có thể dùng trực tiếp static member variable và member method của class.

## Keyword this

Keyword this dùng để tham chiếu đến instance hiện tại của class. Ví dụ:

```java
class Manager {
    Employees[] employees;
    void manageEmployees() {
        int totalEmp = this.employees.length;
        System.out.println("Total employees: " + totalEmp);
        this.report();
    }
    void report() { }
}
```

Trong ví dụ trên, keyword this được dùng ở hai nơi:

- this.employees.length: truy cập variable của instance hiện tại của class Manager.
- this.report(): gọi method của instance hiện tại của class Manager.

Keyword này là tùy chọn, nghĩa là ví dụ trên vẫn hoạt động như cũ nếu không dùng keyword này. Tuy nhiên, dùng keyword này có thể giúp code dễ đọc hoặc dễ hiểu hơn.

## Keyword super

Keyword super dùng để truy cập variable và method của superclass từ subclass. Ví dụ:

```java
public class Super {
    protected int number;
    protected void showNumber() {
        System.out.println("number = " + number);
    }
}
public class Sub extends Super {
    void bar() {
        super.number = 10;
        super.showNumber();
    }
}
```

Trong ví dụ trên, class Sub truy cập member variable number của superclass và gọi method `showNumber()` của superclass Super.

**Lưu ý khi sử dụng this và super:**

- Khi dùng `super()` trong constructor để gọi constructor khác của superclass, câu lệnh này phải ở dòng đầu tiên của constructor, nếu không compiler sẽ báo lỗi. Tương tự, khi dùng `this()` để gọi constructor khác trong class hiện tại, cũng phải đặt ở dòng đầu tiên.
- this và super không thể dùng trong static method.

**Giải thích đơn giản:**

Member được modifier bởi static thuộc về class; static context không có current instance, nên không thể dùng `this`. `super` cũng không phải là một reference độc lập trỏ tới “superclass object”, mà là một dạng syntax bị giới hạn dùng để truy cập member của superclass hoặc gọi superclass constructor, vì vậy cũng không thể dùng trong static context.

## Tham khảo

- <https://www.codejava.net/java-core/the-java-language/java-keywords>
- <https://blog.csdn.net/u013393958/article/details/79881037>

# Giải thích chi tiết keyword static

## Keyword static chủ yếu có bốn trường hợp sử dụng sau

1. Modifier member variable và member method
2. Static code block
3. Dùng làm modifier cho class (chỉ có thể modifier inner class)
4. Static import (dùng để import static resource trong class, là tính năng mới từ 1.5)

### Modifier member variable và member method (thường dùng)

Member được modifier bởi static thuộc về class, không thuộc về một object cụ thể nào của class đó, được mọi object trong class chia sẻ, có thể và nên được gọi thông qua tên class. Vị trí lưu trữ cụ thể của static variable là chi tiết triển khai của JVM.

Method area cũng giống Java heap, là runtime data area được mọi thread chia sẻ. JVM specification quy định nơi này lưu thông tin cấu trúc của mỗi class, chẳng hạn runtime constant pool, field và method data, cùng code của method và constructor. Layout lưu trữ cụ thể do JVM implementation quyết định.

Trong HotSpot của JDK 7 trở về trước, method area chủ yếu được triển khai bằng permanent generation, nhưng method area và permanent generation không tương đương. JDK 8 đã loại bỏ permanent generation: class metadata được chuyển sang metaspace trong native memory, còn string constant và static variable của class nằm trên Java heap.

Cú pháp gọi:

- `ClassName.staticVariableName`
- `ClassName.staticMethod()`

Nếu variable hoặc method được modifier bởi private, điều đó có nghĩa attribute hoặc method đó chỉ có thể được truy cập bên trong class, không thể được truy cập bên ngoài class.

Phương thức kiểm thử:

```java
public class StaticBean {
    String name;
    // Static variable
    static int age;
    public StaticBean(String name) {
        this.name = name;
    }
    // Static method
    static void sayHello() {
        System.out.println("Hello i am java");
    }
    @Override
    public String toString() {
        return "StaticBean{"+
                "name=" + name + ",age=" + age +
                "}";
    }
}
```

```java
public class StaticDemo {
    public static void main(String[] args) {
        StaticBean staticBean = new StaticBean("1");
        StaticBean staticBean2 = new StaticBean("2");
        StaticBean staticBean3 = new StaticBean("3");
        StaticBean staticBean4 = new StaticBean("4");
        StaticBean.age = 33;
        System.out.println(staticBean + " " + staticBean2 + " " + staticBean3 + " " + staticBean4);
        //StaticBean{name=1,age=33} StaticBean{name=2,age=33} StaticBean{name=3,age=33} StaticBean{name=4,age=33}
        StaticBean.sayHello();//Hello i am java
    }
}
```

### Static code block

Static code block được định nghĩa bên ngoài method, bên trong class; static code block thực thi trước non-static code block (static code block -> non-static code block -> constructor). Dù class tạo bao nhiêu object, static code block cũng chỉ thực thi một lần.

Format của static code block là:

```plain
static {
statement body;
}
```

Một class có thể có nhiều static code block, vị trí có thể tùy ý đặt, và chúng không nằm trong bất kỳ method body nào. Static code block thực thi khi class initialization; nếu có nhiều block, JVM sẽ thực thi lần lượt theo thứ tự xuất hiện trong class, mỗi block chỉ thực thi một lần. Class loading có thể xảy ra trước initialization.

![](https://oss.javaguide.cn/github/javaguide/88531075.jpg)

Static code block có thể gán value cho static variable được định nghĩa sau nó, nhưng không thể truy cập variable đó.

### Static inner class

Giữa static inner class và non-static inner class có một khác biệt lớn nhất: sau khi compile xong, non-static inner class ngầm lưu một reference trỏ tới outer class tạo ra nó, còn static inner class thì không. Không có reference này nghĩa là:

1. Việc tạo nó không cần phụ thuộc vào việc tạo outer class.
2. Nó không thể sử dụng bất kỳ non-static member variable và method nào của outer class.

Ví dụ (static inner class triển khai singleton pattern)

```java
public class Singleton {
    // Khai báo là private để tránh gọi default constructor tạo object
    private Singleton() {
    }
   // Khai báo là private để cho biết static inner class chỉ có thể được truy cập trong class Singleton
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }
    public static Singleton getUniqueInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```

Chỉ khi gọi `getUniqueInstance()` và lần đầu chủ động sử dụng `SingletonHolder.INSTANCE` thì `SingletonHolder` mới được khởi tạo; lúc này `INSTANCE` cũng được khởi tạo. JVM có thể load `SingletonHolder` sớm hơn, nhưng sẽ không vì vậy mà thực thi static initialization của nó, đồng thời bảo đảm class này chỉ được khởi tạo một lần.

Cách này không chỉ có ưu điểm lazy initialization mà còn được JVM hỗ trợ về thread safety.

### Static import

Cú pháp là: import static

Hai keyword này khi dùng cùng nhau có thể chỉ định import static resource cụ thể trong một class; không cần dùng tên class để gọi static member trong class mà có thể dùng trực tiếp static member variable và member method của class.

```java
  // Import toàn bộ static resource trong Math; lúc này có thể dùng trực tiếp static method bên trong mà không cần gọi qua tên class
  // Nếu chỉ muốn import một static method, chỉ cần thay * bằng tên method tương ứng
  import static java.lang.Math.*;//Thay bằng import static java.lang.Math.max; để chỉ định import một static method duy nhất là max
public class Demo {
  public static void main(String[] args) {
    int max = max(1,2);
    System.out.println(max);
  }
}
```

## Nội dung bổ sung

### Static method và non-static method

Static method thuộc về chính class, non-static method thuộc về từng object được tạo từ class đó. Nếu operation do method thực hiện không phụ thuộc vào các variable và method của class, hãy khai báo method đó là static (điều này giúp giảm footprint của chương trình). Nếu không, method nên là non-static.

Example

```java
class Foo {
    int i;
    public Foo(int i) {
       this.i = i;
    }
    public static String method1() {
       return "An example string that doesn't depend on i (an instance variable)";
    }
    public int method2() {
       return this.i + 1;  //Depends on i
    }
}
```

Có thể gọi static method như sau: `Foo.method1()`. Nếu thử dùng cách này để gọi method2 thì sẽ thất bại. Nhưng cách sau có thể thực hiện được:

```java
Foo bar = new Foo(1);
bar.method2();
```

Tóm lại:

- Khi gọi static method từ bên ngoài, có thể dùng cách “tên class.tên method” hoặc cách “tên object.tên method”. Instance method chỉ có cách sau. Nói cách khác, gọi static method không cần tạo object.
- Khi static method truy cập member của class hiện tại, chỉ được phép truy cập static member (tức static member variable và static method), không được truy cập instance member variable và instance method; instance method không có giới hạn này.

### Static code block `static{}` và non-static code block `{}` (instance initialization block)

Điểm giống nhau: đều có thể định nghĩa nhiều block trong class; nhiều static code block trong cùng một class được đưa vào quá trình class initialization theo thứ tự văn bản, nhiều instance initialization block được đưa vào quá trình instance initialization theo thứ tự văn bản.

Điểm khác nhau: static code block thực thi một lần khi class initialization, thời điểm trigger không nhất thiết là lần đầu `new`; non-static code block (instance initialization block) thực thi mỗi lần khởi tạo instance mới, và được đưa cùng instance field initializer vào quá trình instance initialization theo thứ tự văn bản. Chúng chạy sau khi superclass constructor hoàn tất và trước khi thực thi các câu lệnh tiếp theo của constructor; nếu constructor ủy quyền cho constructor khác trong cùng class thông qua `this(...)`, quá trình initialization này do constructor thực sự gọi superclass constructor trong delegation chain hoàn tất. Bare code block trong method thông thường chỉ là local code block, không phải instance initialization block.

> **🐛 Đính chính (xem [issue #677](https://github.com/Snailclimb/JavaGuide/issues/677))**: Static code block có thể thực thi khi lần đầu `new` object, nhưng không nhất thiết chỉ thực thi trong lần `new` đầu tiên. Ví dụ, khi tạo Class object thông qua `Class.forName("ClassDemo")`, nó cũng được thực thi; tức là cả `new` và `Class.forName("ClassDemo")` đều thực thi static code block.
> Trong trường hợp thông thường, nếu một số code, chẳng hạn việc khởi tạo các variable hoặc object thường dùng trong project, phải được thực thi khi project khởi động thì cần dùng static code block; đây là code được chủ động thực thi. Nếu muốn thiết kế method trong class có thể được gọi mà không cần tạo object, chẳng hạn class `Arrays`, class `Character`, class `String`, thì cần dùng static method. Điểm khác nhau là static code block được tự động thực thi, còn static method chỉ thực thi khi được gọi.

Ví dụ:

```java
public class Test {
    public Test() {
        System.out.print("Default constructor! --");
    }
    // Non-static code block
    {
        System.out.print("Non-static code block! --");
    }
    // Static code block
    static {
        System.out.print("Static code block! --");
    }
    private static void test() {
        System.out.print("Content in static method! --");
        {
            System.out.print("Code block in static method! --");
        }
    }
    public static void main(String[] args) {
        Test test = new Test();
        Test.test();//Static code block! --Content in static method! --Code block in static method! --
    }
}
```

Output của code trên:

```plain
Static code block! --Non-static code block! --Default constructor! --Content in static method! --Code block in static method! --
```

Khi chỉ thực thi `Test.test()`, output là:

```plain
Static code block! --Content in static method! --Code block in static method! --
```

Khi chỉ thực thi `Test test = new Test()`, output là:

```plain
Static code block! --Non-static code block! --Default constructor! --
```

Khác biệt giữa non-static code block và constructor là: non-static code block dùng để khởi tạo thống nhất cho mọi object, còn constructor dùng để khởi tạo object tương ứng. Vì có thể có nhiều constructor nên constructor nào được chạy sẽ tạo ra object tương ứng, nhưng dù tạo object nào thì cũng luôn thực thi code block chung trước. Nói cách khác, nội dung khởi tạo chung của các object khác nhau được định nghĩa trong code block.

### Tham khảo

- <https://blog.csdn.net/chen13579867831/article/details/78995480>
- <https://www.cnblogs.com/chenssy/p/3388487.html>
- <https://www.cnblogs.com/Qian123/p/5713440.html>

<!-- @include: @article-footer.snippet.md -->
