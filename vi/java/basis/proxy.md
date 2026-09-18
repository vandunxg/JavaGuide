---
title: Giải thích chi tiết Proxy pattern trong Java
description: "Giải thích chi tiết nguyên lý và cách triển khai Proxy pattern trong Java: so sánh sự khác biệt giữa static proxy và dynamic proxy, phân tích cơ chế JDK dynamic proxy và CGLIB proxy, tìm hiểu cách triển khai cross-cutting concern của AOP."
category: Java
tag:
  - Java Basics
head:
  - - meta
    - name: keywords
      content: Proxy pattern trong Java, static proxy, dynamic proxy, JDK dynamic proxy, CGLIB proxy, AOP, design pattern, triển khai proxy
---

## 1. Proxy pattern

Proxy pattern là một design pattern tương đối dễ hiểu. Nói đơn giản, **chúng ta sử dụng proxy object để thay thế việc truy cập real object, nhờ đó có thể cung cấp thêm các thao tác chức năng và mở rộng chức năng của target object mà không cần sửa target object ban đầu.**

**Tác dụng chính của Proxy pattern là mở rộng chức năng của target object. Ví dụ, bạn có thể thêm một số thao tác tùy chỉnh trước hoặc sau khi một method nào đó của target object được thực thi.**

Lấy một ví dụ: cô dâu nhờ dì của mình thay mình xử lý các câu hỏi của chú rể. Mọi câu hỏi cô dâu nhận được đều đã được dì xử lý và lọc trước. Ở đây, dì có thể được xem là proxy object của bạn; hành vi (method) của proxy là nhận và trả lời câu hỏi của chú rể.

![Understanding the Proxy Design Pattern | by Mithun Sasidharan | Medium](https://oss.javaguide.cn/2020-8/1*DjWCgTFm-xqbhbNQVsaWQw.png)

<p style="text-align:right;font-size:13px;color:gray">https://medium.com/@mithunsasidharan/understanding-the-proxy-design-pattern-5e63fe38052a</p>

Proxy pattern có hai cách triển khai là static proxy và dynamic proxy. Trước tiên, hãy xem cách triển khai static proxy.

## 2. Static proxy

Trong static proxy, việc tăng cường từng method của target object đều được thực hiện thủ công (sẽ minh họa cụ thể bằng code ở phần sau), rất kém linh hoạt (ví dụ khi interface thêm method mới, cả target object và proxy object đều phải sửa đổi) và phiền phức (cần viết một proxy class riêng cho từng target class). Trường hợp sử dụng trong thực tế rất ít, hầu như không thấy static proxy trong phát triển hằng ngày.

Ở trên, chúng ta nói về static proxy từ góc độ triển khai và ứng dụng. Từ góc độ JVM, **static proxy biến interface, implementation class, proxy class thành từng file class thực tế ngay trong quá trình compile.**

Các bước triển khai static proxy:

1. Định nghĩa một interface và implementation class của nó;
2. Tạo một proxy class cũng triển khai interface này;
3. Inject target object vào proxy class, sau đó gọi method tương ứng của target class trong method tương ứng của proxy class. Nhờ vậy, chúng ta có thể che giấu việc truy cập target object thông qua proxy class, đồng thời thực hiện các thao tác tùy ý trước và sau khi target method được thực thi.

Sau đây là minh họa bằng code!

**1. Định nghĩa interface gửi SMS**

```java
public interface SmsService {
    String send(String message);
}
```

**2. Triển khai interface gửi SMS**

```java
public class SmsServiceImpl implements SmsService {
    public String send(String message) {
        System.out.println("send message:" + message);
        return message;
    }
}
```

**3. Tạo proxy class và cũng triển khai interface gửi SMS**

```java
public class SmsProxy implements SmsService {

    private final SmsService smsService;

    public SmsProxy(SmsService smsService) {
        this.smsService = smsService;
    }

    @Override
    public String send(String message) {
        // Trước khi gọi method, chúng ta có thể thêm thao tác của mình
        System.out.println("before method send()");
        smsService.send(message);
        // Sau khi gọi method, chúng ta cũng có thể thêm thao tác của mình
        System.out.println("after method send()");
        return null;
    }
}
```

**4. Sử dụng thực tế**

```java
public class Main {
    public static void main(String[] args) {
        SmsService smsService = new SmsServiceImpl();
        SmsProxy smsProxy = new SmsProxy(smsService);
        smsProxy.send("java");
    }
}
```

Sau khi chạy code trên, console in ra:

```bash
before method send()
send message:java
after method send()
```

Có thể thấy qua kết quả output rằng chúng ta đã thêm chức năng vào method `send()` của `SmsServiceImpl`.

## 3. Dynamic proxy

So với static proxy, dynamic proxy linh hoạt hơn. Chúng ta không cần tạo riêng một proxy class cho từng target class, đồng thời cũng không bắt buộc phải triển khai interface; chúng ta có thể proxy trực tiếp implementation class (cơ chế CGLIB dynamic proxy).

**Từ góc độ JVM, dynamic proxy tạo bytecode của class một cách động tại runtime rồi load vào JVM.**

Khi nói đến dynamic proxy, Spring AOP và RPC framework là hai nội dung không thể bỏ qua; việc triển khai của chúng đều phụ thuộc vào dynamic proxy.

**Dynamic proxy được sử dụng tương đối ít trong phát triển hằng ngày, nhưng gần như là kỹ thuật bắt buộc trong framework. Sau khi học dynamic proxy, bạn cũng sẽ hiểu và học nguyên lý của nhiều framework khác dễ dàng hơn.**

Trong Java có nhiều cách triển khai dynamic proxy, chẳng hạn như **JDK dynamic proxy**, **CGLIB dynamic proxy**, v.v.

[guide-rpc-framework](https://github.com/Snailclimb/guide-rpc-framework) sử dụng JDK dynamic proxy, trước tiên hãy xem cách sử dụng JDK dynamic proxy.

Ngoài ra, mặc dù [guide-rpc-framework](https://github.com/Snailclimb/guide-rpc-framework) không sử dụng **CGLIB dynamic proxy**, chúng ta vẫn sẽ giới thiệu ngắn gọn cách sử dụng cũng như so sánh nó với **JDK dynamic proxy**.

### 3.1. Cơ chế JDK dynamic proxy

#### 3.1.1. Giới thiệu

**Trong cơ chế Java dynamic proxy, interface `InvocationHandler` và class `Proxy` là thành phần cốt lõi.**

Method được sử dụng thường xuyên nhất trong class `Proxy` là `newProxyInstance()`. Method này chủ yếu dùng để tạo proxy object.

```java
    public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
    {
        ......
    }
```

Method này có tổng cộng 3 tham số:

1. **loader**: class loader, dùng để load proxy object.
2. **interfaces**: một số interface được class bị proxy triển khai;
3. **h**: object triển khai interface `InvocationHandler`;

Để triển khai dynamic proxy, chúng ta còn phải triển khai `InvocationHandler` nhằm tùy chỉnh logic xử lý. Khi dynamic proxy object gọi một method, lời gọi method này sẽ được chuyển tiếp đến method `invoke` của class triển khai interface `InvocationHandler`.

```java
public interface InvocationHandler {

    /**
     * Khi bạn gọi method thông qua proxy object, thực tế method này sẽ được gọi
     */
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}
```

Method `invoke()` có ba tham số sau:

1. **proxy**: dynamic proxy class
2. **method**: tương ứng với method được gọi bởi proxy class object
3. **args**: các tham số của method hiện tại

Nói cách khác: **khi proxy object được tạo bằng `newProxyInstance()` của class `Proxy` gọi method, thực tế method `invoke()` của class triển khai interface `InvocationHandler` sẽ được gọi.** Bạn có thể tùy chỉnh logic xử lý trong method `invoke()`, chẳng hạn như thực hiện thao tác gì trước và sau khi method được thực thi.

#### 3.1.2. Các bước sử dụng JDK dynamic proxy class

1. Định nghĩa một interface và implementation class của nó;
2. Tùy chỉnh `InvocationHandler` và override method `invoke`; trong method `invoke`, chúng ta sẽ gọi method gốc (method của class bị proxy) và tùy chỉnh một số logic xử lý;
3. Tạo proxy object thông qua method `Proxy.newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h)`;

#### 3.1.3. Ví dụ code

Cách giải thích này có thể hơi trừu tượng và khó hiểu, hãy xem một ví dụ để hình dung!

**1. Định nghĩa interface gửi SMS**

```java
public interface SmsService {
    String send(String message);
}
```

**2. Triển khai interface gửi SMS**

```java
public class SmsServiceImpl implements SmsService {
    public String send(String message) {
        System.out.println("send message:" + message);
        return message;
    }
}
```

**3. Định nghĩa một JDK dynamic proxy class**

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

/**
 * @author shuang.kou
 * @createTime 2020-05-11 11:23:00
 */
public class DebugInvocationHandler implements InvocationHandler {
    /**
     * Real object trong proxy class
     */
    private final Object target;

    public DebugInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws InvocationTargetException, IllegalAccessException {
        // Trước khi gọi method, chúng ta có thể thêm thao tác của mình
        System.out.println("before method " + method.getName());
        Object result = method.invoke(target, args);
        // Sau khi gọi method, chúng ta cũng có thể thêm thao tác của mình
        System.out.println("after method " + method.getName());
        return result;
    }
}

```

Method `invoke()`: khi dynamic proxy object gọi method gốc, cuối cùng thực tế method được gọi là `invoke()`, sau đó method `invoke()` thay chúng ta gọi method gốc của object bị proxy.

**4. Lấy factory class của proxy object**

```java
public class JdkProxyFactory {
    public static Object getProxy(Object target) {
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(), // class loader của target class
                target.getClass().getInterfaces(),  // các interface proxy cần triển khai, có thể chỉ định nhiều interface
                new DebugInvocationHandler(target)   // InvocationHandler tùy chỉnh tương ứng với proxy object
        );
    }
}
```

`getProxy()`: chủ yếu lấy proxy object của một class thông qua method `Proxy.newProxyInstance（）`

**5. Sử dụng thực tế**

```java
SmsService smsService = (SmsService) JdkProxyFactory.getProxy(new SmsServiceImpl());
smsService.send("java");
```

Sau khi chạy code trên, console in ra:

```plain
before method send
send message:java
after method send
```

### 3.2. Cơ chế CGLIB dynamic proxy

#### 3.2.1. Giới thiệu

**Vấn đề nghiêm trọng nhất của JDK dynamic proxy là nó chỉ có thể proxy các class đã triển khai interface.**

**Để giải quyết vấn đề này, chúng ta có thể sử dụng cơ chế CGLIB dynamic proxy.**

[CGLIB](https://github.com/cglib/cglib)(_Code Generation Library_) là một bytecode generation library dựa trên [ASM](http://www.baeldung.com/java-asm), cho phép sửa đổi và tạo bytecode một cách động tại runtime. CGLIB triển khai proxy thông qua inheritance. Nhiều open-source framework nổi tiếng sử dụng [CGLIB](https://github.com/cglib/cglib), chẳng hạn module AOP của Spring: nếu target object triển khai interface thì mặc định sử dụng JDK dynamic proxy, nếu không thì sử dụng CGLIB dynamic proxy.

**Trong cơ chế CGLIB dynamic proxy, interface `MethodInterceptor` và class `Enhancer` là thành phần cốt lõi.**

Bạn cần tùy chỉnh `MethodInterceptor` và override method `intercept`; `intercept` dùng để intercept và tăng cường method của class bị proxy.

```java
public interface MethodInterceptor
extends Callback{
    // Intercept method trong class bị proxy
    public Object intercept(Object obj, java.lang.reflect.Method method, Object[] args,MethodProxy proxy) throws Throwable;
}

```

1. **obj**: object bị proxy (object cần được tăng cường)
2. **method**: method bị intercept (method cần được tăng cường)
3. **args**: các tham số đầu vào của method
4. **proxy**: dùng để gọi method gốc

Bạn có thể lấy class bị proxy một cách động thông qua class `Enhancer`. Khi proxy class gọi method, thực tế method được gọi là method `intercept` trong `MethodInterceptor`.

#### 3.2.2. Các bước sử dụng CGLIB dynamic proxy class

1. Định nghĩa một class;
2. Tùy chỉnh `MethodInterceptor` và override method `intercept`; `intercept` dùng để intercept và tăng cường method của class bị proxy, tương tự method `invoke` trong JDK dynamic proxy;
3. Tạo proxy class thông qua `create()` của class `Enhancer`;

#### 3.2.3. Ví dụ code

Khác với JDK dynamic proxy không cần dependency bổ sung, [CGLIB](https://github.com/cglib/cglib)(_Code Generation Library_) thực tế thuộc một open-source project. Nếu muốn sử dụng nó, bạn cần tự thêm dependency tương ứng.

```xml
<dependency>
  <groupId>cglib</groupId>
  <artifactId>cglib</artifactId>
  <version>3.3.0</version>
</dependency>
```

**1. Triển khai một class gửi SMS thông qua Aliyun**

```java
package github.javaguide.dynamicProxy.cglibDynamicProxy;

public class AliSmsService {
    public String send(String message) {
        System.out.println("send message:" + message);
        return message;
    }
}
```

**2. Tùy chỉnh `MethodInterceptor` (method interceptor)**

```java
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

/**
 * MethodInterceptor tùy chỉnh
 */
public class DebugMethodInterceptor implements MethodInterceptor {


    /**
     * @param o           chính proxy object (chú ý không phải original object; nếu sử dụng method.invoke(o, args) sẽ dẫn đến recursive call)
     * @param method      method bị intercept (method cần được tăng cường)
     * @param args        các tham số đầu vào của method
     * @param methodProxy cơ chế gọi method hiệu năng cao, tránh overhead của reflection
     */
    @Override
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        // Trước khi gọi method, chúng ta có thể thêm thao tác của mình
        System.out.println("before method " + method.getName());
        Object object = methodProxy.invokeSuper(o, args);
        // Sau khi gọi method, chúng ta cũng có thể thêm thao tác của mình
        System.out.println("after method " + method.getName());
        return object;
    }

}
```

**3. Lấy proxy class**

```java
import net.sf.cglib.proxy.Enhancer;

public class CglibProxyFactory {

    public static Object getProxy(Class<?> clazz) {
        // Tạo dynamic proxy enhancement class
        Enhancer enhancer = new Enhancer();
        // Thiết lập class loader
        enhancer.setClassLoader(clazz.getClassLoader());
        // Thiết lập class bị proxy
        enhancer.setSuperclass(clazz);
        // Thiết lập method interceptor
        enhancer.setCallback(new DebugMethodInterceptor());
        // Tạo proxy class
        return enhancer.create();
    }
}
```

**4. Sử dụng thực tế**

```java
AliSmsService aliSmsService = (AliSmsService) CglibProxyFactory.getProxy(AliSmsService.class);
aliSmsService.send("java");
```

Sau khi chạy code trên, console in ra:

```bash
before method send
send message:java
after method send
```

### 3.3. So sánh JDK dynamic proxy và CGLIB dynamic proxy

1. JDK dynamic proxy là cơ chế chính thức, yêu cầu class bị proxy phải triển khai interface. Nguyên lý của nó là tạo động một implementation class của interface để làm proxy. CGLIB là thư viện bên thứ ba, không yêu cầu interface. Nguyên lý của nó là tạo động một subclass của class bị proxy để làm proxy. Nhưng cũng vì sử dụng inheritance nên nó không thể proxy class `final`; method bị proxy cũng không thể là `final` hoặc `private`.
2. Về hiệu năng, trong phần lớn trường hợp JDK dynamic proxy tốt hơn. Khi version JDK được nâng cấp, ưu thế này càng rõ rệt.

## 4. So sánh static proxy và dynamic proxy

Khác biệt cốt lõi giữa static proxy và dynamic proxy nằm ở **thời điểm xác định quan hệ proxy, tính linh hoạt khi triển khai và chi phí bảo trì**.

| Khía cạnh so sánh                | Static proxy (Static Proxy)                                                                                                                                       | Dynamic proxy (Dynamic Proxy)                                                                                                                                     |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Thời điểm xác định quan hệ proxy | Compile time (sau khi compile tạo file bytecode `.class` cố định)                                                                                                 | Runtime (tạo bytecode của proxy class một cách động và load vào JVM)                                                                                              |
| Cách triển khai                  | Viết proxy class thủ công trước khi compile, thường gọi target object thông qua composition và delegation                                                         | Không cần viết proxy class cụ thể thủ công, đóng gói logic tăng cường thông qua `Handler`/`Interceptor`                                                           |
| Phụ thuộc interface              | Không bắt buộc; static proxy dựa trên interface thường khiến proxy class và target class tuân theo cùng một interface                                             | JDK dynamic proxy hướng đến interface, CGLIB và các proxy subclass khác hướng đến implementation class có thể được kế thừa                                        |
| Lượng code và khả năng bảo trì   | Lượng code lớn (càng nhiều target class thì càng nhiều proxy class), chi phí bảo trì cao; khi interface thêm method, target class và proxy class phải sửa đồng bộ | Lượng code rất ít (logic tăng cường dùng chung có thể tái sử dụng), khả năng bảo trì tốt; tách rời khỏi interface, thay đổi interface không ảnh hưởng logic proxy |
| Ưu thế cốt lõi                   | Triển khai đơn giản, logic trực quan, không phụ thuộc framework bổ sung                                                                                           | Tính linh hoạt và khả năng tái sử dụng cao, giảm code lặp, phù hợp với các tình huống phức tạp                                                                    |
| Trường hợp sử dụng điển hình     | Decorator pattern đơn giản, nhu cầu tăng cường cho một số ít class cố định                                                                                        | Spring AOP, RPC framework (chẳng hạn Dubbo), ORM framework                                                                                                        |

## 5. Tổng kết

Bài viết này chủ yếu giới thiệu hai cách triển khai Proxy pattern: static proxy và dynamic proxy. Nội dung bao gồm thực hành static proxy và dynamic proxy, sự khác biệt giữa static proxy và dynamic proxy, cũng như sự khác biệt giữa JDK dynamic proxy và CGLIB dynamic proxy.

Bạn có thể tìm thấy toàn bộ source code được đề cập trong bài viết tại đây: [https://github.com/Snailclimb/guide-rpc-framework-learning/tree/master/src/main/java/github/javaguide/proxy](https://github.com/Snailclimb/guide-rpc-framework-learning/tree/master/src/main/java/github/javaguide/proxy).

<!-- @include: @article-footer.snippet.md -->
