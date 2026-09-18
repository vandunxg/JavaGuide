---
title: Giải thích chi tiết cơ chế Java SPI
description: "Giải thích toàn diện nguyên lý và ứng dụng của cơ chế Java SPI: tìm hiểu cơ chế phát hiện dịch vụ ServiceLoader, ứng dụng SPI trong JDBC/Dubbo/Spring, so sánh với API và các thực hành tốt nhất."
category: Java
tag:
  - Java Basics
head:
  - - meta
    - name: keywords
      content: Java SPI,cơ chế SPI,ServiceLoader,phát hiện dịch vụ,plugin hóa,tải driver JDBC,mở rộng Dubbo,ứng dụng SPI
---

> Bài viết do [Kingshion](https://github.com/jjx0708) đóng góp. Hoan nghênh thêm nhiều bạn tham gia duy trì JavaGuide, đây là một việc rất có ý nghĩa. Xem thông tin chi tiết tại: [Hướng dẫn đóng góp cho JavaGuide](https://javaguide.cn/javaguide/contribution-guideline.html).

Thiết kế hướng đối tượng khuyến khích lập trình giữa các module dựa trên interface thay vì implementation cụ thể, nhằm giảm coupling giữa các module, tuân thủ nguyên lý đảo ngược phụ thuộc và hỗ trợ nguyên lý đóng-mở (mở cho mở rộng, đóng cho sửa đổi). Tuy nhiên, phụ thuộc trực tiếp vào implementation cụ thể khiến phải sửa code khi thay thế implementation, vi phạm nguyên lý đóng-mở. SPI ra đời để giải quyết vấn đề này. Nó cung cấp một cơ chế phát hiện dịch vụ, cho phép chỉ định động implementation cụ thể bên ngoài chương trình. Điều này tương tự tư tưởng đảo ngược quyền kiểm soát (IoC), chuyển quyền kiểm soát việc lắp ráp component ra bên ngoài chương trình.

Cơ chế SPI cũng giải quyết hạn chế do mô hình delegation của class loader trong hệ thống load class của Java gây ra. [Mô hình delegation](https://javaguide.cn/java/jvm/classloader.html) tuy bảo đảm tính an toàn và nhất quán của core library, nhưng cũng hạn chế core library hoặc extension library load các class trên classpath của ứng dụng (thường do bên thứ ba implementation). SPI cho phép core library hoặc extension library định nghĩa service interface, developer bên thứ ba cung cấp và deploy implementation; cơ chế load SPI service sẽ phát hiện và load các implementation này một cách động khi runtime. Ví dụ, JDBC 4.0 và các phiên bản sau sử dụng SPI để tự động phát hiện và load database driver. Developer chỉ cần đặt JAR driver trên classpath mà không cần dùng `Class.forName()` để load driver class một cách tường minh.

## Giới thiệu SPI

### SPI là gì?

SPI là viết tắt của Service Provider Interface, nghĩa đen là “interface của service provider”. Theo cách hiểu của tôi, đây là một interface chuyên cung cấp cho service provider hoặc developer mở rộng chức năng của framework sử dụng.

SPI tách service interface khỏi service implementation cụ thể, decouple service caller và service implementer, từ đó nâng cao khả năng mở rộng và khả năng bảo trì của chương trình. Sửa đổi hoặc thay thế service implementation không cần sửa caller.

Nhiều framework sử dụng cơ chế SPI của Java, chẳng hạn như Spring framework, load database driver, logging interface và implementation mở rộng của Dubbo.

<img src="https://oss.javaguide.cn/github/javaguide/java/basis/spi/22e1830e0b0e4115a882751f6c417857tplv-k3u1fbpfcp-zoom-1.jpeg" style="zoom:50%;" />

### SPI và API khác nhau như thế nào?

**Vậy SPI khác API như thế nào?**

Nhắc đến SPI thì không thể không nói đến API (Application Programming Interface). Theo nghĩa rộng, cả hai đều là interface và rất dễ nhầm lẫn. Trước tiên hãy dùng một hình để giải thích:

![SPI VS API](https://oss.javaguide.cn/github/javaguide/java/basis/spi-vs-api.png)

Thông thường các module giao tiếp với nhau thông qua interface, vì vậy chúng ta đưa một “interface” vào giữa service caller và service implementation (còn gọi là service provider).

- Khi bên implementation cung cấp interface và implementation, chúng ta có thể gọi interface của bên implementation để sử dụng năng lực mà bên implementation cung cấp. Đây là **API**. Trong trường hợp này, interface và implementation đều nằm trong package của bên implementation. Caller gọi chức năng của bên implementation thông qua interface mà không cần quan tâm đến chi tiết implementation cụ thể.
- Khi interface nằm ở phía caller, đây là **SPI**. Bên caller của interface xác định quy tắc interface, sau đó các vendor khác nhau implementation interface này theo quy tắc đó để cung cấp service.

Lấy một ví dụ dễ hiểu: công ty H là một công ty công nghệ, vừa thiết kế một chip mới và hiện cần sản xuất hàng loạt. Trên thị trường có vài công ty sản xuất chip. Khi đó, chỉ cần công ty H xác định tiêu chuẩn sản xuất chip (định nghĩa tiêu chuẩn interface), các công ty chip hợp tác (service provider) sẽ giao chip mang đặc trưng riêng theo tiêu chuẩn (cung cấp các implementation khác nhau nhưng kết quả đưa ra là giống nhau).

## Demo thực tế

SLF4J (Simple Logging Facade for Java) là một logging facade (interface) của Java. Nó có một số implementation cụ thể như Logback, Log4j, Log4j2, v.v. và có thể chuyển đổi. Khi chuyển implementation logging cụ thể, chúng ta không cần sửa code của project, chỉ cần sửa một số dependency pom trong dependency Maven.

![](https://oss.javaguide.cn/github/javaguide/java/basis/spi/image-20220723213306039-165858318917813.png)

Đây là cách triển khai dựa trên cơ chế SPI. Tiếp theo chúng ta sẽ tự triển khai một logging framework đơn giản.

### Service Provider Interface

Tạo một Java project mới `service-provider-interface` với cấu trúc thư mục như sau (chú ý chỉ cần tạo Java project, không cần tạo Maven project. Maven project sẽ liên quan đến một số cấu hình compile; nếu có private repository thì deploy sẽ thuận tiện hơn, nhưng nếu không có, trong quá trình này có thể gặp một số vấn đề khó hiểu):

```plain
│  service-provider-interface.iml
│
├─.idea
│  │  .gitignore
│  │  misc.xml
│  │  modules.xml
│  └─ workspace.xml
│
└─src
    └─edu
        └─jiangxuan
            └─up
                └─spi
                        Logger.java
                        LoggerService.java
                        Main.class
```

Tạo interface `Logger`, đây chính là SPI, tức service provider interface; các service provider phía sau sẽ implementation interface này.

```java
package edu.jiangxuan.up.spi;

public interface Logger {
    void info(String msg);
    void debug(String msg);
}
```

Tiếp theo là class `LoggerService`, chủ yếu cung cấp chức năng cụ thể cho service user (caller). Class này cũng là phần then chốt để triển khai cơ chế Java SPI. Nếu còn thắc mắc, bạn có thể xem tiếp phần sau.

```java
package edu.jiangxuan.up.spi;

import java.util.ArrayList;
import java.util.List;
import java.util.ServiceLoader;

public class LoggerService {
    private static final LoggerService SERVICE = new LoggerService();

    private final Logger logger;

    private final List<Logger> loggerList;

    private LoggerService() {
        ServiceLoader<Logger> loader = ServiceLoader.load(Logger.class);
        List<Logger> list = new ArrayList<>();
        for (Logger log : loader) {
            list.add(log);
        }
        // LoggerList là toàn bộ ServiceProvider
        loggerList = list;
        if (!list.isEmpty()) {
            // Logger chỉ lấy một provider
            logger = list.get(0);
        } else {
            logger = null;
        }
    }

    public static LoggerService getService() {
        return SERVICE;
    }

    public void info(String msg) {
        if (logger == null) {
            System.out.println("Không tìm thấy Logger service provider trong info");
        } else {
            logger.info(msg);
        }
    }

    public void debug(String msg) {
        if (loggerList.isEmpty()) {
            System.out.println("Không tìm thấy Logger service provider trong debug");
        }
        loggerList.forEach(log -> log.debug(msg));
    }
}
```

Tạo class `Main` (service user, caller), khởi động chương trình để xem kết quả.

```java
package org.spi.service;

public class Main {
    public static void main(String[] args) {
        LoggerService service = LoggerService.getService();

        service.info("Hello SPI");
        service.debug("Hello SPI");
    }
}
```

Kết quả chương trình:

> Không tìm thấy Logger service provider trong info
> Không tìm thấy Logger service provider trong debug

Lúc này chúng ta mới chỉ có interface mà chưa cung cấp implementation nào cho interface `Logger`, nên kết quả output không in ra kết quả tương ứng như mong đợi.

Bạn có thể dùng command hoặc trực tiếp dùng IDEA để package toàn bộ chương trình thành một JAR.

### Service Provider

Tiếp theo tạo một project mới để implementation interface `Logger`.

Tạo project `service-provider` với cấu trúc thư mục như sau:

```plain
│  service-provider.iml
│
├─.idea
│  │  .gitignore
│  │  misc.xml
│  │  modules.xml
│  └─ workspace.xml
│
├─lib
│      service-provider-interface.jar
|
└─src
    ├─edu
    │  └─jiangxuan
    │      └─up
    │          └─spi
    │              └─service
    │                      Logback.java
    │
    └─META-INF
        └─services
                edu.jiangxuan.up.spi.Logger

```

Tạo class `Logback`:

```java
package edu.jiangxuan.up.spi.service;

import edu.jiangxuan.up.spi.Logger;

public class Logback implements Logger {
    @Override
    public void info(String s) {
        System.out.println("Logback info ghi log: " + s);
    }

    @Override
    public void debug(String s) {
        System.out.println("Logback debug ghi log: " + s);
    }
}

```

Import JAR của `service-provider-interface` vào project.

Tạo thư mục lib, sau đó copy JAR vào và thêm nó vào project.

![](https://oss.javaguide.cn/github/javaguide/java/basis/spi/523d5e25198444d3b112baf68ce49daetplv-k3u1fbpfcp-watermark.png)

Sau đó nhấn OK.

![](https://oss.javaguide.cn/github/javaguide/java/basis/spi/f4ba0aa71e9b4d509b9159892a220850tplv-k3u1fbpfcp-watermark.png)

Tiếp theo bạn có thể import một số class và method trong JAR vào project, giống như import package của utility class trong JDK.

Implementation interface `Logger`: trong thư mục `src` tạo folder `META-INF/services`, sau đó tạo file `edu.jiangxuan.up.spi.Logger` (full name của SPI interface). Nội dung file là: `edu.jiangxuan.up.spi.service.Logback` (full name của Logback, tức package name + class name của implementation class của SPI).

**Đây là tiêu chuẩn do cơ chế JDK SPI `ServiceLoader` quy ước.**

Trước tiên hãy giải thích khái quát: gọi `ServiceLoader.load()` sẽ tạo service loader. Khi duyệt `ServiceLoader`, loader sẽ định vị và khởi tạo provider theo nhu cầu; khi gọi `stream()`, kết quả là stream của `ServiceLoader.Provider`, có thể kiểm tra type của provider trước thông qua `type()`, chỉ khi gọi `Provider.get()` mới lấy được instance service provider tương ứng. `ServiceLoader` sẽ cache các provider đã load. Đối với provider trên classpath, file cấu hình nằm trong `META-INF/services`; đối với named module từ Java 9 trở đi, còn có thể khai báo quan hệ service thông qua `uses` và `provides ... with ...` trong module descriptor.

Vì vậy có một số yêu cầu quy ước: tên file nhất định phải là full name của interface, nội dung bên trong nhất định phải là full name của implementation class. Có thể có nhiều implementation class, chỉ cần xuống dòng; khi có nhiều implementation class, chúng sẽ được load lần lượt.

Tiếp theo cũng package project `service-provider` thành JAR. JAR này chính là implementation của service provider. Thông thường việc import Maven dependency trong pom cũng tương tự như vậy, chỉ là hiện tại chúng ta chưa publish JAR này lên Maven public repository nên ở nơi cần sử dụng chỉ có thể thêm thủ công vào project.

### Demo kết quả

Để minh họa trực quan hơn, tôi tạo thêm một project chuyên dùng để test: `java-spi-test`.

Sau đó import JAR của interface `Logger`, rồi import JAR của implementation cụ thể.

![](https://oss.javaguide.cn/github/javaguide/java/basis/spi/image-20220723215812708-165858469599214.png)

Tạo method Main để test:

```java
package edu.jiangxuan.up.service;

import edu.jiangxuan.up.spi.LoggerService;

public class TestJavaSPI {
    public static void main(String[] args) {
        LoggerService loggerService = LoggerService.getService();
        loggerService.info("Xin chào");
        loggerService.debug("Kiểm thử cơ chế Java SPI");
    }
}
```

Kết quả chạy như sau:

> Logback info ghi log: Xin chào
> Logback debug ghi log: Kiểm thử cơ chế Java SPI

Điều này cho thấy implementation class trong JAR đã được import và có hiệu lực.

Nếu không import JAR của implementation cụ thể, kết quả chạy chương trình sẽ là:

> Không tìm thấy Logger service provider trong info
> Không tìm thấy Logger service provider trong debug

Thông qua cơ chế SPI, có thể thấy coupling giữa service (`LoggerService`) và service provider rất thấp. Nếu muốn thay đổi implementation, thực ra chỉ cần sửa implementation cụ thể của interface `Logger` trong project `service-provider`, rồi thay một JAR là được. Một project cũng có thể có nhiều implementation. Đây chẳng phải là nguyên lý của SLF4J sao?

Nếu một ngày yêu cầu thay đổi và cần output log vào message queue hoặc thực hiện một thao tác khác, hoàn toàn không cần sửa implementation của Logback. Chỉ cần thêm một service implementation mới (`service-provider`), có thể thêm implementation trong project hiện tại hoặc import JAR service implementation mới từ bên ngoài. Trong service (`LoggerService`), chúng ta có thể chọn một service implementation cụ thể (`service-provider`) để hoàn thành thao tác cần thiết.

Tiếp theo chúng ta sẽ nói cụ thể về nguyên lý then chốt của Java SPI — **ServiceLoader**.

## ServiceLoader

### Implementation cụ thể của ServiceLoader

> Phần dưới đây trình bày implementation `ServiceLoader` của JDK 8. Java 9 trở đi bổ sung module layer, `stream()`, `findFirst()` và cơ chế provider method, nên cấu trúc source code hiện tại đã khác.

Muốn sử dụng cơ chế SPI của Java cần dựa vào `ServiceLoader` để triển khai. Tiếp theo hãy xem `ServiceLoader` được thực hiện cụ thể như thế nào:

`ServiceLoader` là một utility class do JDK cung cấp, nằm trong package `package java.util;`.

```plain
A facility to load implementations of a service.
```

Đây là comment chính thức của JDK: **Một công cụ để load service implementation.**

Xem tiếp, chúng ta nhận thấy class này có kiểu `final`, nên không thể được inheritance hoặc sửa đổi, đồng thời nó implementation interface `Iterable`. Việc implementation iterator giúp chúng ta dễ dàng lấy service implementation tương ứng bằng cách duyệt ở phần sau.

```java
public final class ServiceLoader<S> implements Iterable<S>{ xxx...}
```

Có thể thấy một định nghĩa constant quen thuộc:

`private static final String PREFIX = "META-INF/services/";`

Dưới đây là method `load`: có thể thấy method `load` hỗ trợ hai loại argument sau khi overload:

```java
public static <S> ServiceLoader<S> load(Class<S> service) {
    ClassLoader cl = Thread.currentThread().getContextClassLoader();
    return ServiceLoader.load(service, cl);
}

public static <S> ServiceLoader<S> load(Class<S> service,
                                        ClassLoader loader) {
    return new ServiceLoader<>(service, loader);
}

private ServiceLoader(Class<S> svc, ClassLoader cl) {
    service = Objects.requireNonNull(svc, "Service interface cannot be null");
    loader = (cl == null) ? ClassLoader.getSystemClassLoader() : cl;
    acc = (System.getSecurityManager() != null) ? AccessController.getContext() : null;
    reload();
}

public void reload() {
    providers.clear();
    lookupIterator = new LazyIterator(service, loader);
}
```

Cơ chế giải quyết việc load class bên thứ ba thực ra nằm trong `ClassLoader cl = Thread.currentThread().getContextClassLoader();`; `cl` là **Thread Context ClassLoader**, tức class loader của context thread. Đây là class loader mà mỗi thread nắm giữ. Thiết kế của JDK cho phép application hoặc container (chẳng hạn Web application server) thiết lập class loader này, để core library có thể dùng nó load class của application.

Trong điều kiện mặc định, Thread Context ClassLoader là Application ClassLoader, chịu trách nhiệm load class trên classpath. Khi core library cần load class do application cung cấp, nó có thể dùng Thread Context ClassLoader để hoàn tất. Nhờ vậy, ngay cả code của core library được Bootstrap ClassLoader load cũng có thể load và sử dụng class do Application ClassLoader load.

Theo thứ tự gọi của code, method `reload()` sử dụng inner class `LazyIterator` để thực hiện. Hãy xem tiếp phần dưới.

Sau khi implementation interface `Iterable`, `ServiceLoader` có khả năng duyệt. Khi method `iterator` được gọi, trước tiên nó tìm trong `Provider` cache của `ServiceLoader`; nếu cache không có kết quả thì tìm trong `LazyIterator`.

```java
public Iterator<S> iterator() {
    return new Iterator<S>() {

        Iterator<Map.Entry<String, S>> knownProviders
                = providers.entrySet().iterator();

        public boolean hasNext() {
            if (knownProviders.hasNext())
                return true;
            return lookupIterator.hasNext(); // Gọi LazyIterator
        }

        public S next() {
            if (knownProviders.hasNext())
                return knownProviders.next().getValue();
            return lookupIterator.next(); // Gọi LazyIterator
        }

        public void remove() {
            throw new UnsupportedOperationException();
        }

    };
}
```

Khi gọi `LazyIterator`, implementation cụ thể như sau:

```java
public boolean hasNext() {
    if (acc == null) {
        return hasNextService();
    } else {
        PrivilegedAction<Boolean> action = new PrivilegedAction<Boolean>() {
            public Boolean run() {
                return hasNextService();
            }
        };
        return AccessController.doPrivileged(action, acc);
    }
}

private boolean hasNextService() {
    if (nextName != null) {
        return true;
    }
    if (configs == null) {
        try {
            // Dùng PREFIX (META-INF/services/) và class name để lấy file cấu hình tương ứng, từ đó lấy implementation class cụ thể
            String fullName = PREFIX + service.getName();
            if (loader == null)
                configs = ClassLoader.getSystemResources(fullName);
            else
                configs = loader.getResources(fullName);
        } catch (IOException x) {
            fail(service, "Error locating configuration files", x);
        }
    }
    while ((pending == null) || !pending.hasNext()) {
        if (!configs.hasMoreElements()) {
            return false;
        }
        pending = parse(service, configs.nextElement());
    }
    nextName = pending.next();
    return true;
}


public S next() {
    if (acc == null) {
        return nextService();
    } else {
        PrivilegedAction<S> action = new PrivilegedAction<S>() {
            public S run() {
                return nextService();
            }
        };
        return AccessController.doPrivileged(action, acc);
    }
}

private S nextService() {
    if (!hasNextService())
        throw new NoSuchElementException();
    String cn = nextName;
    nextName = null;
    Class<?> c = null;
    try {
        c = Class.forName(cn, false, loader);
    } catch (ClassNotFoundException x) {
        fail(service,
                "Provider " + cn + " not found");
    }
    if (!service.isAssignableFrom(c)) {
        fail(service,
                "Provider " + cn + " not a subtype");
    }
    try {
        S p = service.cast(c.newInstance());
        providers.put(cn, p);
        return p;
    } catch (Throwable x) {
        fail(service,
                "Provider " + cn + " could not be instantiated",
                x);
    }
    throw new Error();          // Điều này không thể xảy ra
}
```

Có thể nhiều người thấy phần này hơi phức tạp. Không sao, tôi đã triển khai một model `ServiceLoader` đơn giản để minh họa ý tưởng cốt lõi: đọc tên implementation class từ file cấu hình trên classpath và tạo instance. Nó không bao quát đầy đủ các hành vi của implementation JDK như lazy loading, loại bỏ trùng lặp, xử lý lỗi và module provider.

### Tự triển khai một ServiceLoader

Trước tiên tôi đưa code lên:

```java
package edu.jiangxuan.up.service;

import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.lang.reflect.Constructor;
import java.net.URL;
import java.net.URLConnection;
import java.util.ArrayList;
import java.util.Enumeration;
import java.util.List;

public class MyServiceLoader<S> {

    // Class template của interface tương ứng
    private final Class<S> service;

    // Có thể có nhiều implementation class tương ứng, đóng gói bằng List
    private final List<S> providers = new ArrayList<>();

    // Class loader
    private final ClassLoader classLoader;

    // Method cung cấp cho bên ngoài sử dụng; gọi method này để bắt đầu quy trình load implementation tùy chỉnh.
    public static <S> MyServiceLoader<S> load(Class<S> service) {
        return new MyServiceLoader<>(service);
    }

    // Đặt constructor ở phạm vi private
    private MyServiceLoader(Class<S> service) {
        this.service = service;
        this.classLoader = Thread.currentThread().getContextClassLoader();
        doLoad();
    }

    // Method then chốt, logic load implementation class cụ thể
    private void doLoad() {
        try {
            // Đọc các file trong thư mục META-INF/services của mọi JAR; tên file là tên interface, nội dung file là path và full name của implementation class cụ thể
            Enumeration<URL> urls = classLoader.getResources("META-INF/services/" + service.getName());
            // Duyệt lần lượt các file đã lấy được
            while (urls.hasMoreElements()) {
                // Lấy file hiện tại
                URL url = urls.nextElement();
                System.out.println("File = " + url.getPath());
                // Tạo connection
                URLConnection urlConnection = url.openConnection();
                urlConnection.setUseCaches(false);
                // Lấy input stream của file
                InputStream inputStream = urlConnection.getInputStream();
                // Lấy buffer từ input stream của file
                BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
                // Lấy full name của implementation class từ nội dung file
                String className = bufferedReader.readLine();

                while (className != null) {
                    // Lấy instance của implementation class thông qua reflection
                    Class<?> clazz = Class.forName(className, false, classLoader);
                    // Nếu interface được khai báo và implementation class cụ thể thuộc cùng một kiểu (có thể hiểu là một dạng polymorphism của Java, như quan hệ giữa interface và implementation class, hoặc giữa parent class và child class), thì tạo instance
                    if (service.isAssignableFrom(clazz)) {
                        Constructor<? extends S> constructor = (Constructor<? extends S>) clazz.getConstructor();
                        S instance = constructor.newInstance();
                        // Thêm instance object vừa tạo vào danh sách Provider
                        providers.add(instance);
                    }
                    // Tiếp tục đọc implementation class ở dòng tiếp theo; có thể có nhiều implementation class, chỉ cần xuống dòng.
                    className = bufferedReader.readLine();
                }
            }
        } catch (Exception e) {
            System.out.println("Đọc file gặp exception...");
        }
    }

    // Trả về danh sách implementation class cụ thể tương ứng với SPI interface
    public List<S> getProviders() {
        return providers;
    }
}
```

Các thông tin then chốt cơ bản đã được mô tả qua comment trong code.

Quy trình chính là:

1. Dùng utility class URL để tìm file tương ứng trong thư mục `/META-INF/services` của JAR,
2. Đọc tên file để tìm SPI interface tương ứng,
3. Dùng stream `InputStream` để đọc full name của implementation class cụ thể trong file,
4. Dựa trên full name lấy được, trước tiên kiểm tra xem nó có cùng kiểu với SPI interface hay không. Nếu có, dùng reflection để tạo instance object tương ứng,
5. Thêm instance object vừa tạo vào danh sách `Providers`.

## Tổng kết

Không khó để nhận thấy implementation cụ thể của cơ chế SPI cần định vị động và tạo service provider. Provider trên classpath cần được khai báo trong file cấu hình `META-INF/services/`; provider trong named module được khai báo thông qua `provides ... with ...` trong module descriptor.

Ngoài ra, cơ chế SPI được ứng dụng trong nhiều framework: nguyên lý cơ bản của Spring framework cũng tương tự. Dubbo framework cũng cung cấp cơ chế mở rộng SPI tương tự, chỉ là cách triển khai cụ thể cơ chế SPI trong Dubbo và Spring framework có một số khác biệt nhỏ so với cơ chế chúng ta học hôm nay, nhưng nguyên lý tổng thể đều giống nhau. Tin rằng thông qua việc học cơ chế SPI trong JDK, bạn có thể suy ra những trường hợp khác và hiểu sâu hơn về các framework nâng cao khác.

Cơ chế SPI có thể nâng cao đáng kể tính linh hoạt của thiết kế interface, nhưng cũng có một số nhược điểm:

1. `ServiceLoader` sẽ lazy load provider; nếu caller duyệt toàn bộ provider để chọn implementation, vẫn có thể phát sinh overhead bổ sung;
2. Một instance `ServiceLoader` không bảo đảm thread-safe. Nếu muốn share giữa các thread, caller cần thực hiện synchronization. Việc nhiều instance `ServiceLoader` đồng thời gọi `load` không vì thế mà chắc chắn phát sinh xung đột concurrency.

<!-- @include: @article-footer.snippet.md -->
