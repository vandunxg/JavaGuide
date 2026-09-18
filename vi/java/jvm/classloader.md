---
title: Giải thích chi tiết về Class Loader (trọng điểm)
description: "Giải thích chi tiết về Java Class Loader: phân tích sâu cơ chế ClassLoader, nguyên lý mô hình Parent Delegation, Bootstrap Class Loader/Platform Class Loader/Application Class Loader, triển khai Class Loader tùy chỉnh và các trường hợp phá vỡ Parent Delegation."
category: Java
tag:
  - JVM
head:
  - - meta
    - name: keywords
      content: Class Loader,ClassLoader,Parent Delegation Model,quá trình load class,Class Loader tùy chỉnh,phá vỡ Parent Delegation
---

## Ôn lại quá trình load class

Trước khi giới thiệu Class Loader và Parent Delegation Model, hãy cùng ôn lại ngắn gọn quá trình load class.

- Quá trình load class: **load->link->initialize**.
- Quá trình link có thể chia thành ba bước: **verify->prepare->resolve**.

![Quá trình load class](https://oss.javaguide.cn/github/javaguide/java/jvm/class-loading-procedure.png)

Loading là bước đầu tiên của quá trình load class, chủ yếu hoàn thành 3 việc sau:

1. Lấy binary byte stream của class này thông qua full class name.
2. Chuyển đổi cấu trúc lưu trữ tĩnh do byte stream biểu diễn thành cấu trúc dữ liệu runtime của method area.
3. Tạo một đối tượng `Class` đại diện cho class này trong memory, làm entry point để truy cập các dữ liệu trong method area.

## Class Loader

### Giới thiệu về Class Loader

Class Loader xuất hiện từ JDK 1.0, ban đầu chỉ nhằm đáp ứng nhu cầu của Java Applet (đã bị loại bỏ). Sau đó, nó dần trở thành một bộ phận quan trọng trong chương trình Java, trao cho Java khả năng dynamically load class vào JVM và thực thi.

Theo giới thiệu trong tài liệu API chính thức:

> A class loader is an object that is responsible for loading classes. The class ClassLoader is an abstract class. Given the binary name of a class, a class loader should attempt to locate or generate data that constitutes a definition for the class. A typical strategy is to transform the name into a file name and then read a "class file" of that name from a file system.
>
> Every Class object contains a reference to the ClassLoader that defined it.
>
> Class objects for array classes are not created by class loaders, but are created automatically as required by the Java runtime. The class loader for an array class, as returned by Class.getClassLoader() is the same as the class loader for its element type; if the element type is a primitive type, then the array class has no class loader.

Dịch nôm na là:

> Class Loader là một object chịu trách nhiệm load class. `ClassLoader` là một abstract class. Với binary name của một class, Class Loader phải thử định vị hoặc tạo dữ liệu cấu thành định nghĩa của class đó. Một chiến lược điển hình là chuyển name thành file name, sau đó đọc “class file” có name đó từ file system.
>
> Mỗi non-array class hoặc interface đều có một reference trỏ đến `ClassLoader` đã định nghĩa nó. Array class không được tạo thông qua `ClassLoader` mà được JVM tự động tạo khi cần; Class Loader của array class kiểu reference giống với component type, còn array class của primitive type không có Class Loader, `getClassLoader()` trả về `null`.

Từ phần giới thiệu trên có thể thấy:

- Class Loader là một object chịu trách nhiệm load class, dùng để thực hiện bước loading trong quá trình load class.
- Mỗi non-array class hoặc interface đều ghi nhận `ClassLoader` đã định nghĩa nó.
- Array class không được tạo thông qua `ClassLoader` (array class không có binary byte stream tương ứng) mà do JVM trực tiếp tạo; array class kiểu reference sử dụng Class Loader đã định nghĩa component type, còn array class của primitive type không có Class Loader.

```java
class Class<T> {
  ...
  private final ClassLoader classLoader;
  @CallerSensitive
  public ClassLoader getClassLoader() {
     //...
  }
  ...
}
```

Nói đơn giản, **vai trò chính của Class Loader là dynamically load bytecode của Java class (file `.class`) vào JVM (tạo một object `Class` đại diện cho class đó trong memory).** Bytecode có thể được biên dịch từ Java source program (file `.java`) bằng `javac`, cũng có thể được dynamic generate bằng tool hoặc download qua network.

Ngoài load class, Class Loader còn có thể load các resource mà Java application cần như text, image, config, video và các file resource khác. Bài viết này chỉ thảo luận chức năng cốt lõi của nó: load class.

### Quy tắc load của Class Loader

Khi JVM khởi động, nó không load tất cả class cùng một lúc mà dynamically load theo nhu cầu. Nói cách khác, phần lớn class chỉ được load khi thực sự được sử dụng, giúp memory thân thiện hơn.

Khi load class, `ClassLoader#loadClass` trước tiên sẽ thông qua `findLoadedClass` để kiểm tra JVM đã ghi nhận Class Loader hiện tại là initiating loader của class tương ứng với binary name hay chưa. Nếu có thì trả về trực tiếp, nếu không mới tiếp tục delegation hoặc tìm kiếm. Một Class Loader không thể định nghĩa lại class có cùng binary name.

```java
// Các field và method sau đây lấy từ implementation ClassLoader của JDK 8, chỉ dùng để minh họa cách ghi nhận ở version này
public abstract class ClassLoader {
  ...
  private final ClassLoader parent;
  // Các class được Class Loader này load.
  private final Vector<Class<?>> classes = new Vector<>();
  // Được VM gọi để dùng Class Loader này ghi nhận từng class đã load.
  void addClass(Class<?> c) {
        classes.addElement(c);
   }
  ...
}
```

### Tổng kết về Class Loader

Ba Class Loader quan trọng thường gặp trong JDK 8:

1. **`BootstrapClassLoader` (Bootstrap Class Loader)**: Class Loader built-in của virtual machine ở tầng cao nhất, thường được biểu diễn là `null` trong Java API và không có parent loader. Trong HotSpot của JDK 8, nó chủ yếu load runtime core class library (như `rt.jar`) và các class trong path do `-Xbootclasspath` chỉ định.
2. **`ExtensionClassLoader` (Extension Class Loader)**: Chủ yếu chịu trách nhiệm load các jar và class trong thư mục `%JRE_HOME%/lib/ext`, cùng toàn bộ class trong path được system variable `java.ext.dirs` chỉ định.
3. **`AppClassLoader` (Application Class Loader)**: Class Loader hướng đến user, chịu trách nhiệm load toàn bộ jar và class trong classpath của application hiện tại.

> 🌈 Mở rộng:
>
> - **`rt.jar`**: rt là viết tắt của “RunTime”, `rt.jar` là Java base class library, chứa class file của tất cả class được thấy trong Java doc. Nói cách khác, các built-in library thường dùng `java.xxx.*` đều nằm trong đó, chẳng hạn `java.util.*`, `java.io.*`, `java.nio.*`, `java.lang.*`, `java.sql.*`, `java.math.*`.
> - Sau khi Java 9 giới thiệu module system, `rt.jar` và cơ chế extension directory không còn được sử dụng; Extension Class Loader được Platform Class Loader thay thế. Bootstrap, platform và application Class Loader lần lượt định nghĩa các runtime module khác nhau; không thể đơn giản khái quát rằng mọi module ngoài `java.base` đều do Platform Class Loader load.

Ngoài ba loại Class Loader này, user còn có thể thêm Class Loader tùy chỉnh để đáp ứng nhu cầu đặc biệt. Chẳng hạn, có thể encrypt bytecode của Java class (file `.class`), sau đó dùng Class Loader tùy chỉnh để decrypt khi load.

![Sơ đồ quan hệ phân cấp của Class Loader](https://oss.javaguide.cn/github/javaguide/java/jvm/class-loader-parents-delegation-model.png)

Bootstrap Class Loader là built-in của virtual machine, thường được biểu diễn là `null` trong Java API. Platform Class Loader, Application Class Loader và Class Loader tùy chỉnh thông thường đều là instance của `ClassLoader`. Nhờ đó, user có thể tự định nghĩa Class Loader để application quyết định cách lấy class cần thiết.

Mỗi `ClassLoader` có thể lấy `ClassLoader` cha của nó thông qua `getParent()`. Nếu `ClassLoader` lấy được là `null`, thì parent loader của Class Loader đó là `BootstrapClassLoader`.

```java
public abstract class ClassLoader {
  ...
  // Parent loader
  private final ClassLoader parent;
  @CallerSensitive
  public final ClassLoader getParent() {
     //...
  }
  ...
}
```

**Tại sao class do Bootstrap Class Loader định nghĩa gọi `getClassLoader()` lại nhận được `null`?** Vì Bootstrap Class Loader là built-in Class Loader của JVM, Java API quy ước dùng `null` để biểu diễn nó; ngôn ngữ dùng trong implementation cụ thể không thuộc Java specification.

Sau đây là một ví dụ nhỏ lấy `ClassLoader`:

```java
public class PrintClassLoaderTree {

    public static void main(String[] args) {

        ClassLoader classLoader = PrintClassLoaderTree.class.getClassLoader();

        StringBuilder split = new StringBuilder("|--");
        boolean needContinue = true;
        while (needContinue){
            System.out.println(split.toString() + classLoader);
            if(classLoader == null){
                needContinue = false;
            }else{
                classLoader = classLoader.getParent();
                split.insert(0, "\t");
            }
        }
    }

}
```

Kết quả output (JDK 8):

```plain
|--sun.misc.Launcher$AppClassLoader@18b4aac2
    |--sun.misc.Launcher$ExtClassLoader@53bd815b
        |--null
```

Từ kết quả output có thể thấy:

- `ClassLoader` của Java class `PrintClassLoaderTree` do chúng ta viết là `AppClassLoader`;
- Parent `ClassLoader` của `AppClassLoader` là `ExtClassLoader`;
- Parent `ClassLoader` của `ExtClassLoader` là `Bootstrap Class Loader`, vì vậy kết quả output là null.

### Class Loader tùy chỉnh

Như đã nói ở trên, ngoài `BootstrapClassLoader`, các Class Loader khác đều được implement bằng Java và đều kế thừa `java.lang.ClassLoader`. Nếu muốn tự định nghĩa Class Loader, rõ ràng cần kế thừa abstract class `ClassLoader`.

Class `ClassLoader` có hai method quan trọng:

- `protected Class loadClass(String name, boolean resolve)`: load class có binary name được chỉ định và implement Parent Delegation Model. `name` là binary name của class; nếu `resolve` là true thì gọi method `resolveClass(Class<?> c)` để resolve class đó trong quá trình load.
- `protected Class findClass(String name)`: tìm class theo binary name; implementation mặc định trực tiếp throw `ClassNotFoundException`.

Tài liệu API chính thức viết:

> Subclasses of `ClassLoader` are encouraged to override `findClass(String name)`, rather than this method.
>
> Khuyến nghị subclass của `ClassLoader` override method `findClass(String name)` thay vì method `loadClass(String name, boolean resolve)`.

Nếu không muốn phá vỡ Parent Delegation Model, chỉ cần override method `findClass()` trong class `ClassLoader`; class không thể được parent loader load cuối cùng sẽ được load thông qua method này. Tuy nhiên, nếu muốn phá vỡ Parent Delegation Model thì cần override method `loadClass()`.

## Parent Delegation Model

### Giới thiệu về Parent Delegation Model

Có nhiều loại Class Loader. Khi muốn load một class, cụ thể Class Loader nào sẽ load? Đây là lúc cần nhắc đến Parent Delegation Model.

Theo giới thiệu trên website chính thức:

> The ClassLoader class uses a delegation model to search for classes and resources. Each instance of ClassLoader has an associated parent class loader. When requested to find a class or resource, a ClassLoader instance will delegate the search for the class or resource to its parent class loader before attempting to find the class or resource itself. The virtual machine's built-in class loader, called the "bootstrap class loader", does not itself have a parent but may serve as the parent of a ClassLoader instance.

Dịch nôm na là:

> Class `ClassLoader` sử dụng delegation model để tìm kiếm class và resource. Mỗi instance của `ClassLoader` đều có một parent loader tương ứng. Khi cần tìm class hoặc resource, trước khi tự mình thử tìm class hoặc resource đó, instance `ClassLoader` sẽ ủy thác nhiệm vụ tìm kiếm cho parent loader.
> Built-in Class Loader trong virtual machine, được gọi là “bootstrap class loader”, bản thân không có parent loader nhưng có thể làm parent loader của instance `ClassLoader`.

Từ phần giới thiệu trên có thể thấy:

- Class `ClassLoader` sử dụng delegation model để tìm kiếm class và resource.
- Parent Delegation Model yêu cầu mọi Class Loader, ngoại trừ Bootstrap Class Loader ở tầng cao nhất, đều phải có parent loader riêng.
- Trước khi tự mình thử tìm class hoặc resource, instance `ClassLoader` sẽ ủy thác nhiệm vụ tìm kiếm class hoặc resource cho parent loader.

Quan hệ phân cấp giữa các Class Loader được thể hiện trong hình dưới đây được gọi là “**Parent Delegation Model**” của Class Loader.

![Sơ đồ quan hệ phân cấp của Class Loader](https://oss.javaguide.cn/github/javaguide/java/jvm/class-loader-parents-delegation-model.png)

Lưu ý ⚠️: Parent Delegation Model không phải constraint bắt buộc mà chỉ là cách thức được JDK official khuyến nghị. Nếu có nhu cầu đặc biệt và muốn phá vỡ Parent Delegation Model thì vẫn có thể thực hiện; các phương pháp cụ thể sẽ được giới thiệu ở phần sau.

Thực ra, cách dịch “parent” này dễ khiến người khác hiểu lầm; chúng ta thường hiểu “parent” là cha mẹ, nhưng ở đây nó chủ yếu biểu đạt “thế hệ cha mẹ”, chứ không phải thực sự có một `MotherClassLoader` và một `FatherClassLoader`. Cá nhân tôi cho rằng dịch thành Single Parent Delegation Model sẽ tốt hơn, nhưng vì trong nước đã dịch thành Parent Delegation Model và cách gọi này đã được lưu truyền, cứ dùng như vậy cũng không sao, miễn là không bị hiểu lầm.

Ngoài ra, quan hệ parent-child giữa các Class Loader thường không được implement bằng quan hệ inheritance mà thường dùng quan hệ composition để reuse code của parent loader.

```java
public abstract class ClassLoader {
  ...
  // Composition
  private final ClassLoader parent;
  protected ClassLoader(ClassLoader parent) {
       this(checkCreateClassLoader(), parent);
  }
  ...
}
```

Trong object-oriented programming có một design principle rất kinh điển: **composition over inheritance, ưu tiên composition và hạn chế inheritance.**

### Flow thực thi của Parent Delegation Model

Logic chính của Parent Delegation Model tập trung trong `loadClass()` của `java.lang.ClassLoader`. Dưới đây là một đoạn implementation liên quan của JDK 8:

```java
protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException
{
    synchronized (getClassLoadingLock(name)) {
        // Trước tiên, kiểm tra class này đã được load hay chưa
        Class c = findLoadedClass(name);
        if (c == null) {
            // Nếu c là null thì class này chưa được load
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    // Khi parent loader không null, load class này thông qua loadClass của parent loader
                    c = parent.loadClass(name, false);
                } else {
                    // Khi parent loader là null, gọi Bootstrap Class Loader để load class này
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // Khi parent loader khác null không thể tìm thấy class tương ứng thì throw exception
            }

            if (c == null) {
                // Khi parent loader không thể load, gọi method findClass để load class này
                // User có thể override method này để tự định nghĩa Class Loader
                long t1 = System.nanoTime();
                c = findClass(name);

                // Dùng để thống kê thông tin liên quan đến Class Loader
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            // Thực hiện thao tác link trên class
            resolveClass(c);
        }
        return c;
    }
}
```

Mỗi khi một Class Loader nhận được request load, trước tiên nó forward request cho parent loader. Chỉ khi parent loader không tìm thấy class được request thì Class Loader đó mới thử load.

Kết hợp với source code trên, có thể tóm tắt ngắn gọn flow thực thi của Parent Delegation Model như sau:

- Khi load class, system trước tiên kiểm tra class hiện tại đã được load hay chưa. Class đã được load sẽ được trả về trực tiếp, nếu chưa mới thử load (mỗi parent loader đều đi qua flow này một lần).
- Khi load class, Class Loader trước tiên không tự thử load class đó mà ủy thác request cho parent loader thực hiện (gọi method `loadClass()` của parent loader để load class). Như vậy, mọi request cuối cùng đều được chuyển đến Bootstrap Class Loader ở tầng cao nhất.
- Chỉ khi parent loader phản hồi rằng không thể hoàn thành request load này (không tìm thấy class cần thiết trong phạm vi tìm kiếm của nó), child loader mới thử tự load (gọi method `findClass()` của mình để load class).
- Nếu child loader cũng không thể load class này, nó sẽ throw exception `ClassNotFoundException`.

🌈 Mở rộng:

**Quy tắc cụ thể để JVM xác định hai Java class có giống nhau hay không**: JVM không chỉ xem full name của class có giống nhau hay không mà còn xem Class Loader load class đó có giống nhau hay không. Chỉ khi cả hai đều giống nhau thì mới coi hai class là giống nhau. Ngay cả khi hai class bắt nguồn từ cùng một file `Class` và được cùng một virtual machine load, chỉ cần Class Loader load chúng khác nhau thì hai class chắc chắn không giống nhau.

### Lợi ích của Parent Delegation Model

Parent Delegation Model là một bộ phận quan trọng trong cơ chế load Java class. Việc parent loader được ưu tiên giúp code trong cùng một delegation chain reuse type đã được parent loader định nghĩa, đồng thời giảm rủi ro application code giả mạo platform API; nó không thể ngăn các Class Loader độc lập với nhau lần lượt định nghĩa các class trùng name.

JVM phân biệt runtime type dựa trên binary name của class hoặc interface và Class Loader đã định nghĩa nó. Parent Delegation sẽ ưu tiên để platform class do built-in Class Loader tương ứng định nghĩa, nhưng không phải mọi core hoặc platform API đều do Bootstrap Class Loader load: sau JDK 9 còn có Platform Class Loader chịu trách nhiệm định nghĩa platform class.

Ví dụ, JVM sẽ ưu tiên giao request load các core class như `java.lang.Object` cho `BootstrapClassLoader` xử lý; nhưng trên thực tế, `ClassLoader#preDefineClass` còn kiểm tra class name ở giai đoạn definition. Mọi class name bắt đầu bằng `java.` đều bị reject, vì vậy không thể dùng Class Loader tùy chỉnh để giả mạo core class.

Nhiều bạn sẽ nói: “Vậy bypass Parent Delegation Model là được mà?”.

Tuy nhiên, ngay cả khi attacker bypass Parent Delegation Model, Java vẫn có security mechanism ở tầng thấp hơn để bảo vệ core class library. Method `preDefineClass` của `ClassLoader` sẽ kiểm tra class name trước khi define class. Mọi class name bắt đầu bằng `"java."` đều trigger `SecurityException`, ngăn malicious code define hoặc load core class giả mạo.

Source code method `ClassLoader#preDefineClass` trong JDK 8 như sau:

```java
private ProtectionDomain preDefineClass(String name,
                                            ProtectionDomain pd)
    {
        // Kiểm tra class name có hợp lệ hay không
        if (!checkName(name)) {
            throw new NoClassDefFoundError("IllegalName: " + name);
        }

        // Ngăn define class trong package "java.*".
        // Kiểm tra này rất quan trọng đối với security vì ngăn malicious code thay thế core Java class.
        // JDK 9 dùng Platform Class Loader để tăng cường security của method preDefineClass
        if ((name != null) && name.startsWith("java.")) {
            throw new SecurityException
                ("Tên package bị cấm: " +
                 name.substring(0, name.lastIndexOf('.')));
        }

         // Nếu chưa chỉ định ProtectionDomain thì dùng domain mặc định (defaultDomain).
        if (pd == null) {
            pd = defaultDomain;
        }

        if (name != null) {
            checkCerts(name, pd.getCodeSource());
        }

        return pd;
    }
```

JDK 9 giới thiệu Platform Class Loader, có thể lấy nó thông qua `ClassLoader.getPlatformClassLoader()`. Hạn chế package name `java.*` của `defineClass` vẫn tồn tại, nhưng implementation cụ thể khác JDK 8.

### Cách phá vỡ Parent Delegation Model

~~Để tránh cơ chế Parent Delegation, chúng ta có thể tự định nghĩa một Class Loader rồi override `loadClass()`.~~

**🐛 Đính chính (tham khảo [issue871](https://github.com/Snailclimb/JavaGuide/issues/871))**: Khi custom loader, cần kế thừa `ClassLoader`. Nếu không muốn phá vỡ Parent Delegation Model thì chỉ cần override method `findClass()` trong class `ClassLoader`; class không thể được parent loader load cuối cùng sẽ được load thông qua method này. Tuy nhiên, nếu muốn phá vỡ Parent Delegation Model thì cần override method `loadClass()`.

Tại sao override method `loadClass()` lại phá vỡ Parent Delegation Model? Flow thực thi của Parent Delegation Model đã giải thích điều này:

> Khi load class, Class Loader trước tiên không tự thử load class đó mà ủy thác request cho parent loader thực hiện (gọi method `loadClass()` của parent loader để load class).

Sau khi override method `loadClass()`, chúng ta có thể thay đổi flow thực thi truyền thống của Parent Delegation Model. Chẳng hạn, child loader có thể tự thử load class trước khi delegate cho parent loader, hoặc thử load từ nơi khác sau khi parent loader trả về. Quy tắc cụ thể do chúng ta tự implement và customize theo nhu cầu project.

Server Tomcat khá quen thuộc tự định nghĩa Class Loader `WebAppClassLoader` để phá vỡ cơ chế Parent Delegation, nhằm ưu tiên load class trong thư mục Web application trước rồi mới load class trong các thư mục khác. Đây cũng là nguyên lý cụ thể để class giữa các Web application trong Tomcat được isolation.

Cấu trúc phân cấp Class Loader của Tomcat như sau:

![Cấu trúc phân cấp Class Loader của Tomcat](https://oss.javaguide.cn/github/javaguide/java/jvm/tomcat-class-loader-parents-delegation-model.png)

Tomcat hiện đại mặc định sử dụng hierarchy `Bootstrap -> System -> Common -> WebappX`. Vị trí tìm kiếm của từng loader như sau:

- Vị trí tìm kiếm của `Common` được cấu hình bởi `common.loader` trong `$CATALINA_BASE/conf/catalina.properties`, mặc định chủ yếu gồm `$CATALINA_BASE/lib` và `$CATALINA_HOME/lib`.
- Mỗi loader `WebappX` chịu trách nhiệm cho `/WEB-INF/classes` và `/WEB-INF/lib/*.jar` của Web application tương ứng.
- Loader `Server` và `Shared` mặc định không được định nghĩa; chỉ xuất hiện trong hierarchy phức tạp hơn sau khi cấu hình `server.loader` hoặc `shared.loader`.

Từ quan hệ delegation trong hình có thể thấy:

- Các class mà loader `Common` nhìn thấy có thể được Tomcat internal component và mọi Web application dùng chung.
- Nếu cấu hình tường minh, `Server` chỉ hiển thị với Tomcat internal component, còn `Shared` hiển thị với mọi Web application. Cấu hình mặc định không có hai loader này, Web application loader trực tiếp lấy `Common` làm parent loader.
- Mỗi Web application tạo một `WebAppClassLoader` riêng, đồng thời set thread context Class Loader thành `WebAppClassLoader` trong thread khởi động Web application. Các instance `WebAppClassLoader` được isolation với nhau, qua đó thực hiện class isolation giữa các Web application.

Chỉ dựa vào Class Loader tùy chỉnh không thể đáp ứng yêu cầu của một số scenario. Ví dụ, trong một số trường hợp, Class Loader ở tầng cao cần load class mà chỉ loader ở tầng thấp mới có thể load.

Chẳng hạn trong SPI, interface của SPI (như `java.sql.Driver`) do Java core library cung cấp và được `BootstrapClassLoader` load. Implementation của SPI (như `com.mysql.cj.jdbc.Driver`) do third-party vendor cung cấp, được Application Class Loader hoặc Class Loader tùy chỉnh load. Theo mặc định, một class và các dependency của nó do cùng một Class Loader load. Vì vậy, Class Loader load interface của SPI (`BootstrapClassLoader`) cũng sẽ được dùng để load implementation của SPI. Theo Parent Delegation Model, `BootstrapClassLoader` không thể tìm thấy SPI implementation class vì nó không thể delegate cho child loader để thử load.

Cần lưu ý: sau khi JDK 9+ giới thiệu modularization, JDBC API được tách vào module `java.sql`, không còn do `BootstrapClassLoader` trực tiếp load mà do `PlatformClassLoader` load.

```java
public class ClassLoaderTest {
    public static void main(String[] args) throws ClassNotFoundException {
        Class<?> clazz = Class.forName("java.sql.Driver");
        ClassLoader loader = clazz.getClassLoader();
        System.out.println("Loader for java.sql.Driver: " + loader);

        // Trong môi trường .jdks/corretto-1.8.0_442/bin/java là Loader for java.sql.Driver: null

        // Trong môi trường .jdks/jbr-17.0.12/bin/java là Loader for java.sql.Driver: jdk.internal.loader.ClassLoaders$PlatformClassLoader@30f39991
    }
}
```

Một ví dụ khác: giả sử project có jar của Spring. Vì jar này được dùng chung giữa các Web application nên nó sẽ do `SharedClassLoader` load (Web server là Tomcat). Project có một số business class sử dụng Spring, chẳng hạn implement interface do Spring cung cấp hoặc sử dụng annotation do Spring cung cấp. Vì vậy, Class Loader load class của Spring (tức `SharedClassLoader`) cũng sẽ được dùng để load các business class này. Tuy nhiên, business class nằm trong thư mục Web application, không nằm trên load path của `SharedClassLoader`, nên `SharedClassLoader` không thể tìm thấy và load chúng.

Giải quyết vấn đề này thế nào? Khi đó cần dùng **Thread Context Class Loader (`ThreadContextClassLoader`)**.

Với ví dụ Spring, khi Spring cần load business class, nó không dùng Class Loader của mình mà dùng context Class Loader của thread hiện tại. Như đã nói ở trên, mỗi Web application tạo một `WebAppClassLoader` riêng và set thread context Class Loader thành `WebAppClassLoader` trong thread khởi động Web application. Nhờ vậy, Class Loader ở tầng cao (`SharedClassLoader`) có thể mượn child loader (`WebAppClassLoader`) để load business class, phá vỡ cơ chế delegation khi load class của Java và cho phép application sử dụng Class Loader theo hướng ngược lại.

Nguyên lý của Thread Context Class Loader là lưu một Class Loader trong dữ liệu private của thread, bind nó với thread, sau đó lấy ra sử dụng khi cần. Class Loader này thường do application hoặc container (như Tomcat) set.

`Java.lang.Thread` có `getContextClassLoader()` và `setContextClassLoader(ClassLoader cl)` lần lượt dùng để lấy và set thread context Class Loader. Nếu không set bằng `setContextClassLoader(ClassLoader cl)`, thread sẽ kế thừa thread context Class Loader của parent thread.

Code Spring lấy thread context Class Loader như sau:

```java
cl = Thread.currentThread().getContextClassLoader();
```

Bạn có thể tự tìm hiểu sâu hơn về nguyên lý Tomcat phá vỡ Parent Delegation Model. Tài liệu đề xuất: [《Phân tích chuyên sâu Tomcat & Jetty》](http://gk.link/a/10Egr).

## Đọc thêm

- 《Phân tích chuyên sâu Java Virtual Machine》
- Phân tích chuyên sâu nguyên lý Java ClassLoader: <https://blog.csdn.net/xyang81/article/details/7292380>
- Java Class Loader (ClassLoader): <http://gityuan.com/2016/01/24/java-classloader/>
- Class Loaders in Java: <https://www.baeldung.com/java-classloaders>
- Class ClassLoader - tài liệu chính thức Oracle: <https://docs.oracle.com/javase/8/docs/api/java/lang/ClassLoader.html>
- Java ClassLoader khó hiểu, nếu vẫn chưa hiểu thì đã muộn: <https://zhuanlan.zhihu.com/p/51374915>

<!-- @include: @article-footer.snippet.md -->
