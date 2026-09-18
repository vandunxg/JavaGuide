---
title: Giải thích chi tiết các design pattern trong Spring
description: Giải thích chi tiết các design pattern của framework Spring, bao quát việc áp dụng factory pattern, proxy pattern, singleton pattern, template method và các pattern khác trong source code Spring.
category: Framework
tag:
  - Spring
head:
  - - meta
    - name: keywords
      content: Design pattern trong Spring, factory pattern, proxy pattern, template method, singleton, strategy pattern, adapter pattern, source code Spring
---

“JDK sử dụng những design pattern nào? Spring sử dụng những design pattern nào?” là hai câu hỏi khá thường gặp trong phỏng vấn.

Tôi đã tìm kiếm trên Internet một thời gian và nhận thấy các bài giải thích về design pattern trong Spring gần như đều giống nhau, hơn nữa phần lớn đã khá cũ. Vì vậy, tôi đã dành vài ngày để tự tổng hợp lại.

Do năng lực cá nhân có hạn, nếu bài viết có bất kỳ lỗi nào, bạn có thể góp ý. Do dung lượng bài viết có hạn, phần giải thích về design pattern và một số source code chỉ được đề cập sơ lược. Mục đích chính của bài viết là ôn lại các design pattern trong Spring.

## Inversion of Control (IoC) và Dependency Injection (DI)

**IoC (Inversion of Control, đảo ngược quyền kiểm soát)** là một khái niệm rất quan trọng trong Spring. IoC không phải là một kỹ thuật mà là một tư tưởng thiết kế nhằm giảm coupling. Mục đích chính của IoC là dựa vào “bên thứ ba” (IoC container trong Spring) để giảm coupling giữa các object có quan hệ dependency (IoC container quản lý object, bạn chỉ cần sử dụng), từ đó giảm mức độ coupling giữa các đoạn code.

**IoC là một nguyên tắc, không phải một pattern; các pattern dưới đây (nhưng không chỉ giới hạn ở chúng) đã triển khai nguyên tắc IoC.**

![ioc-patterns](https://oss.javaguide.cn/github/javaguide/ioc-patterns.png)

**Spring IoC container giống như một factory. Khi cần tạo một object, bạn chỉ cần cấu hình file cấu hình/annotation, hoàn toàn không cần quan tâm object được tạo ra như thế nào.** IoC container chịu trách nhiệm tạo object, kết nối các object, cấu hình các object và quản lý toàn bộ lifecycle của chúng từ khi tạo ra cho đến khi bị hủy hoàn toàn.

Trong dự án thực tế, nếu một class Service có hàng trăm, thậm chí hàng nghìn class làm dependency bên dưới và chúng ta cần khởi tạo Service này, có thể mỗi lần bạn phải hiểu rõ constructor của toàn bộ class dependency của Service. Điều này có thể khiến bạn phát điên. Nếu sử dụng IoC, bạn chỉ cần cấu hình rồi reference ở nơi cần dùng, giúp tăng đáng kể khả năng maintain của dự án và giảm độ khó khi phát triển.

> Về cách hiểu Spring IoC, bạn nên xem câu trả lời trên Zhihu này: <https://www.zhihu.com/question/23277575/answer/169698662>, nội dung rất hay.

**Hiểu Inversion of Control như thế nào?** Hãy xem một ví dụ: “Object a phụ thuộc vào object b. Khi object a cần sử dụng object b, nó phải tự tạo object b. Nhưng sau khi hệ thống đưa IoC container vào, object a và object b không còn liên hệ trực tiếp với nhau. Lúc này, khi object a cần sử dụng object b, chúng ta có thể chỉ định IoC container tạo một object b rồi inject vào object a.” Quá trình object a nhận dependency object b chuyển từ hành vi chủ động thành hành vi bị động, quyền kiểm soát bị đảo ngược. Đây là nguồn gốc tên gọi Inversion of Control.

**DI (Dependency Injection) là một design pattern dùng để triển khai Inversion of Control. Dependency Injection là truyền instance variable vào một object.**

## Factory Design Pattern

Spring sử dụng factory pattern để tạo bean object thông qua `BeanFactory` hoặc `ApplicationContext`.

**So sánh hai loại:**

- `BeanFactory`: cung cấp năng lực nền tảng của Spring IoC container. Khi sử dụng trực tiếp `BeanFactory` cơ bản, container thường không chủ động pre-instantiate toàn bộ singleton Bean mà tạo Bean khi được request lần đầu.
- `ApplicationContext`: mở rộng `BeanFactory`, bổ sung các năng lực như publish event, internationalization và load resource; đồng thời mặc định pre-instantiate singleton Bean không lazy trong giai đoạn container khởi động. `prototype` Bean và Bean được đánh dấu lazy sẽ không vì vậy mà được tạo toàn bộ một lần.

Ba implementation phổ biến của `ApplicationContext`:

1. `ClassPathXmlApplicationContext`: coi file context là resource trên classpath.
2. `FileSystemXmlApplicationContext`: load thông tin định nghĩa context từ file XML trong file system.
3. `XmlWebApplicationContext`: load thông tin định nghĩa context từ file XML trong hệ thống Web.

Example:

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.FileSystemXmlApplicationContext;

public class App {
  public static void main(String[] args) {
    ApplicationContext context = new FileSystemXmlApplicationContext(
        "C:/work/IOC Containers/springframework.applicationcontext/src/main/resources/bean-factory-config.xml");

    HelloApplicationContext obj = (HelloApplicationContext) context.getBean("helloApplicationContext");
    obj.getMsg();
  }
}
```

## Singleton Design Pattern

Trong hệ thống, có một số object mà thực tế chúng ta chỉ cần một instance, chẳng hạn thread pool, cache, dialog, registry, log object và object đóng vai trò driver cho các thiết bị như printer, card đồ họa. Thực tế, nhóm object này chỉ nên có một instance. Nếu tạo nhiều instance, có thể phát sinh các vấn đề như hành vi bất thường của chương trình, sử dụng quá nhiều resource hoặc kết quả không nhất quán.

**Lợi ích của singleton pattern**:

- Với các object được sử dụng thường xuyên, có thể bỏ qua thời gian tạo object; với các object nặng, đây là khoản overhead hệ thống đáng kể.
- Do số lần thực hiện thao tác `new` giảm, tần suất sử dụng memory hệ thống cũng giảm, từ đó giảm áp lực lên GC và rút ngắn thời gian GC pause.

**Scope mặc định của bean trong Spring là singleton.** Ngoài scope singleton, bean trong Spring còn có các scope sau:

- **prototype**: mỗi lần lấy sẽ tạo một bean instance mới. Nói cách khác, gọi `getBean()` liên tiếp hai lần sẽ nhận được hai Bean instance khác nhau.
- **request** (chỉ dùng được trong Web application): mỗi HTTP request sẽ tạo một bean mới (request bean), bean này chỉ có hiệu lực trong HTTP request hiện tại.
- **session** (chỉ dùng được trong Web application): mỗi HTTP request đến từ một session mới sẽ tạo một bean mới (session bean), bean này chỉ có hiệu lực trong HTTP session hiện tại.
- **application** (chỉ dùng được trong Web application): mỗi `ServletContext` tương ứng với một Bean instance, bean này chỉ có hiệu lực trong lifecycle của Web application hiện tại. Spring phiên bản cũ từng cung cấp scope `globalSession` riêng cho Portlet application; scope này không thuộc danh sách scope tiêu chuẩn hiện tại.
- **websocket** (chỉ dùng được trong Web application): mỗi WebSocket session tạo một bean mới.

Spring triển khai singleton pattern bằng cách dùng `ConcurrentHashMap` theo một cách đặc biệt để xây dựng singleton registry.

Code cốt lõi để Spring triển khai singleton như sau:

```java
// Dùng ConcurrentHashMap (thread-safe) để triển khai singleton registry
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(64);

public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
        Assert.notNull(beanName, "'beanName' must not be null");
        synchronized (this.singletonObjects) {
            // Kiểm tra instance có tồn tại trong cache hay không
            Object singletonObject = this.singletonObjects.get(beanName);
            if (singletonObject == null) {
                //...lược bỏ nhiều code
                try {
                    singletonObject = singletonFactory.getObject();
                }
                //...lược bỏ nhiều code
                // Nếu instance object chưa tồn tại, đăng ký nó vào singleton registry.
                addSingleton(beanName, singletonObject);
            }
            return (singletonObject != NULL_OBJECT ? singletonObject : null);
        }
    }
    // Thêm object vào singleton registry
    protected void addSingleton(String beanName, Object singletonObject) {
            synchronized (this.singletonObjects) {
                this.singletonObjects.put(beanName, (singletonObject != null ? singletonObject : NULL_OBJECT));

            }
        }
}
```

**Singleton Bean có vấn đề về thread-safety không?**

Phần lớn thời gian, chúng ta không sử dụng multi-threading trong dự án nên ít người quan tâm đến vấn đề này. Singleton Bean có vấn đề về thread chủ yếu vì khi nhiều thread thao tác trên cùng một object sẽ xảy ra resource competition.

Có hai cách giải quyết phổ biến:

1. Hạn chế định nghĩa member variable mutable trong Bean.
2. Định nghĩa một member variable `ThreadLocal` trong class, lưu các member variable mutable cần thiết vào `ThreadLocal` (đây là cách được khuyến nghị).

Tuy nhiên, phần lớn Bean thực tế đều stateless (không có instance variable), chẳng hạn Dao và Service. Trong trường hợp này, Bean là thread-safe.

## Proxy Design Pattern

### Ứng dụng proxy pattern trong AOP

**AOP (Aspect-Oriented Programming, lập trình hướng aspect)** có thể đóng gói những logic hoặc trách nhiệm không liên quan đến business nhưng được các business module cùng gọi (chẳng hạn transaction processing, log management, permission control), giúp giảm code trùng lặp trong hệ thống, giảm coupling giữa các module và có lợi cho khả năng mở rộng, maintain sau này.

Spring AOP dựa trên dynamic proxy. Nếu object cần proxy implement một interface, Spring AOP sẽ sử dụng **JDK Proxy** để tạo proxy object. Với object không implement interface, không thể dùng JDK Proxy để proxy; khi đó Spring AOP sẽ dùng **Cglib** tạo subclass của object được proxy làm proxy, như hình dưới đây:

![SpringAOPProcess](https://oss.javaguide.cn/github/javaguide/SpringAOPProcess.jpg)

Tất nhiên, bạn cũng có thể sử dụng AspectJ. Spring AOP đã tích hợp AspectJ, và AspectJ có thể được xem là framework AOP hoàn chỉnh nhất trong hệ sinh thái Java.

Sau khi sử dụng AOP, chúng ta có thể abstract một số chức năng dùng chung và sử dụng trực tiếp ở nơi cần. Điều này giúp đơn giản hóa đáng kể lượng code. Khi cần thêm chức năng mới cũng thuận tiện hơn, qua đó tăng khả năng mở rộng của hệ thống. Chức năng log, transaction management và nhiều trường hợp khác đều sử dụng AOP.

### Spring AOP và AspectJ AOP khác nhau như thế nào?

**Spring AOP là runtime enhancement, còn AspectJ là compile-time enhancement.** Spring AOP dựa trên proxy (Proxying), còn AspectJ dựa trên bytecode manipulation (Bytecode Manipulation).

Spring AOP đã tích hợp AspectJ, và AspectJ có thể được xem là framework AOP hoàn chỉnh nhất trong hệ sinh thái Java. AspectJ mạnh hơn Spring AOP về chức năng, nhưng Spring AOP đơn giản hơn tương đối.

Nếu số lượng aspect ít, chênh lệch performance giữa hai loại không lớn. Tuy nhiên, khi có quá nhiều aspect, tốt nhất nên chọn AspectJ vì nó nhanh hơn Spring AOP rất nhiều.

## Template Method

Template method pattern là một behavioral design pattern. Pattern này định nghĩa skeleton của algorithm trong một operation và trì hoãn một số bước cho subclass. Template method cho phép subclass định nghĩa lại cách triển khai của một số bước cụ thể trong algorithm mà không thay đổi cấu trúc của algorithm.

```java
public abstract class Template {
    // Đây là template method của chúng ta
    public final void TemplateMethod(){
        PrimitiveOperation1();
        PrimitiveOperation2();
        PrimitiveOperation3();
    }

    protected void  PrimitiveOperation1(){
        // Class hiện tại triển khai
    }

    // Method do subclass triển khai
    protected abstract void PrimitiveOperation2();
    protected abstract void PrimitiveOperation3();

}
public class TemplateImpl extends Template {

    @Override
    public void PrimitiveOperation2() {
        // Class hiện tại triển khai
    }

    @Override
    public void PrimitiveOperation3() {
        // Class hiện tại triển khai
    }
}

```

Trong Spring, các class thao tác với database có tên kết thúc bằng Template như `JdbcTemplate`, `HibernateTemplate` sử dụng template pattern. Thông thường, chúng ta dùng inheritance để triển khai template pattern, nhưng Spring không dùng cách này mà kết hợp Callback pattern với template method pattern. Cách này vừa đạt được hiệu quả tái sử dụng code, vừa tăng tính linh hoạt.

## Observer Pattern

Observer pattern là một object behavioral pattern. Pattern này thể hiện quan hệ dependency giữa các object: khi một object thay đổi, tất cả object phụ thuộc vào object đó cũng phản ứng. Event-driven model của Spring là một ứng dụng kinh điển của observer pattern. Event-driven model của Spring rất hữu ích và có thể giảm coupling cho code trong nhiều tình huống. Chẳng hạn, mỗi lần thêm product, chúng ta cần cập nhật lại product index; khi đó có thể dùng observer pattern để giải quyết vấn đề này.

### Ba role trong event-driven model của Spring

#### Event role

`ApplicationEvent` (trong package `org.springframework.context`) đóng vai trò event. Đây là một abstract class, kế thừa `java.util.EventObject` và implement interface `java.io.Serializable`.

Spring mặc định có các event sau. Chúng đều là implementation của `ApplicationContextEvent` (kế thừa từ `ApplicationContextEvent`):

- `ContextStartedEvent`: event được trigger sau khi `ApplicationContext` khởi động;
- `ContextStoppedEvent`: event được trigger sau khi `ApplicationContext` dừng;
- `ContextRefreshedEvent`: event được trigger sau khi `ApplicationContext` khởi tạo hoặc refresh hoàn tất;
- `ContextClosedEvent`: event được trigger sau khi `ApplicationContext` đóng.

![ApplicationEvent-Subclass](https://oss.javaguide.cn/github/javaguide/ApplicationEvent-Subclass.png)

#### Event listener role

`ApplicationListener` đóng vai trò event listener. Đây là một interface, chỉ định nghĩa method `onApplicationEvent()` để xử lý `ApplicationEvent`. Source code của interface `ApplicationListener` như sau. Có thể thấy event trong interface chỉ cần implement `ApplicationEvent` là được. Vì vậy, trong Spring, chỉ cần implement method `onApplicationEvent()` của interface `ApplicationListener` là có thể hoàn tất việc lắng nghe event.

```java
package org.springframework.context;
import java.util.EventListener;
@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
    void onApplicationEvent(E var1);
}
```

#### Event publisher role

`ApplicationEventPublisher` đóng vai trò event publisher. Đây cũng là một interface.

```java
@FunctionalInterface
public interface ApplicationEventPublisher {
    default void publishEvent(ApplicationEvent event) {
        this.publishEvent((Object)event);
    }

    void publishEvent(Object var1);
}
```

Method `publishEvent()` của interface `ApplicationEventPublisher` được implement trong class `AbstractApplicationContext`. Khi đọc implementation của method này, bạn sẽ nhận ra event thực tế được broadcast thông qua `ApplicationEventMulticaster`. Nội dung cụ thể khá dài nên không phân tích ở đây; sau này có thể sẽ có một bài viết riêng đề cập đến vấn đề này.

### Tóm tắt event flow của Spring

1. Định nghĩa một event: implement một class kế thừa `ApplicationEvent` và viết constructor tương ứng;
2. Định nghĩa một event listener: implement interface `ApplicationListener`, override method `onApplicationEvent()`;
3. Dùng event publisher để publish message: có thể publish message thông qua method `publishEvent()` của `ApplicationEventPublisher`.

Example:

```java
// Định nghĩa một event, kế thừa ApplicationEvent và viết constructor tương ứng
public class DemoEvent extends ApplicationEvent{
    private static final long serialVersionUID = 1L;

    private String message;

    public DemoEvent(Object source,String message){
        super(source);
        this.message = message;
    }

    public String getMessage() {
         return message;
          }


// Định nghĩa một event listener, implement interface ApplicationListener và override method onApplicationEvent();
@Component
public class DemoListener implements ApplicationListener<DemoEvent>{

    // Dùng onApplicationEvent để nhận message
    @Override
    public void onApplicationEvent(DemoEvent event) {
        String msg = event.getMessage();
        System.out.println("Message nhận được là: "+msg);
    }

}
// Publish event, có thể dùng method publishEvent() của ApplicationEventPublisher để publish message.
@Component
public class DemoPublisher {

    @Autowired
    ApplicationContext applicationContext;

    public void publish(String message){
        // Publish event
        applicationContext.publishEvent(new DemoEvent(this, message));
    }
}

```

Khi gọi method `publish()` của `DemoPublisher`, chẳng hạn `demoPublisher.publish("Xin chào")`, console sẽ in ra: `Message nhận được là: Xin chào`.

## Adapter Pattern

Adapter pattern chuyển một interface thành interface khác mà client mong muốn. Adapter pattern cho phép các class có interface không tương thích cùng làm việc với nhau.

### Adapter pattern trong Spring AOP

Chúng ta biết implementation của Spring AOP dựa trên proxy pattern, nhưng enhancement hoặc advice của Spring AOP sử dụng adapter pattern. Interface liên quan là `AdvisorAdapter`.

Các loại Advice thường dùng: `BeforeAdvice` (trước khi gọi method mục tiêu, before advice), `AfterAdvice` (sau khi gọi method mục tiêu, after advice), `AfterReturningAdvice` (sau khi method mục tiêu thực thi xong, trước khi return), v.v. Mỗi loại Advice đều có interceptor tương ứng: `MethodBeforeAdviceInterceptor`, `AfterReturningAdviceInterceptor`, `ThrowsAdviceInterceptor`, v.v.

Các advice được Spring định nghĩa sẵn phải thông qua adapter tương ứng để được adapt thành object thuộc kiểu interface `MethodInterceptor` (method interceptor). Ví dụ: `MethodBeforeAdviceAdapter` gọi method `getInterceptor` để adapt `MethodBeforeAdvice` thành `MethodBeforeAdviceInterceptor`.

### Adapter pattern trong Spring MVC

Trong Spring MVC, `DispatcherServlet` gọi `HandlerMapping` dựa trên request information để phân tích `Handler` tương ứng với request. Sau khi phân tích được `Handler` (hay `Controller` như cách chúng ta thường gọi), việc xử lý bắt đầu do adapter `HandlerAdapter` thực hiện. `HandlerAdapter` là expected interface, các adapter implementation cụ thể dùng để adapt target class, còn `Controller` là class cần được adapt.

**Tại sao cần sử dụng adapter pattern trong Spring MVC?**

Spring MVC có rất nhiều loại `Controller`; các loại `Controller` khác nhau xử lý request bằng các method khác nhau. Nếu không sử dụng adapter pattern, `DispatcherServlet` phải trực tiếp lấy `Controller` tương ứng và tự phán đoán type, giống đoạn code dưới đây:

```java
if(mappedHandler.getHandler() instanceof MultiActionController){
   ((MultiActionController)mappedHandler.getHandler()).xxx
}else if(mappedHandler.getHandler() instanceof XXX){
    ...
}else if(...){
   ...
}
```

Nếu thêm một loại `Controller`, phải thêm một câu lệnh điều kiện vào đoạn code trên. Cách này khiến chương trình khó maintain và vi phạm Open-Closed Principle trong design pattern: mở cho extension, đóng cho modification.

## Decorator Pattern

Decorator pattern có thể dynamically thêm một số thuộc tính hoặc hành vi vào object. So với inheritance, decorator pattern linh hoạt hơn. Nói đơn giản, khi cần sửa chức năng hiện có nhưng không muốn trực tiếp sửa code hiện có, hãy thiết kế một Decorator bọc bên ngoài code đó. Trong JDK cũng có nhiều nơi sử dụng decorator pattern, chẳng hạn họ `InputStream`: các subclass như `FileInputStream` (đọc file), `BufferedInputStream` (thêm cache, giúp tốc độ đọc file tăng đáng kể) của class `InputStream` đều mở rộng chức năng mà không sửa code của `InputStream`.

![Minh họa decorator pattern](https://oss.javaguide.cn/github/javaguide/Decorator.jpg)

## Tổng kết

Spring framework đã sử dụng những design pattern nào?

- **Factory design pattern**: Spring sử dụng factory pattern để tạo bean object thông qua `BeanFactory` và `ApplicationContext`.
- **Proxy design pattern**: implementation của chức năng Spring AOP.
- **Singleton design pattern**: Bean trong Spring mặc định đều là singleton.
- **Template method pattern**: các class thao tác với database có tên kết thúc bằng Template như `jdbcTemplate`, `hibernateTemplate` trong Spring sử dụng template pattern.
- **Observer pattern**: event-driven model của Spring là một ứng dụng kinh điển của observer pattern.
- **Adapter pattern**: enhancement hoặc advice của Spring AOP sử dụng adapter pattern; Spring MVC cũng sử dụng adapter pattern để adapt `Controller`.
- ……

## Tài liệu tham khảo

- 《Spring Internals》
- <https://blog.eduonix.com/java-programming-2/learn-design-patterns-used-spring-framework/>
- <https://www.tutorialsteacher.com/ioc/inversion-of-control>
- <https://design-patterns.readthedocs.io/zh_CN/latest/behavioral_patterns/observer.html>
- <https://juejin.im/post/5a8eb261f265da4e9e307230>
- <https://juejin.im/post/5ba28986f265da0abc2b6084>

<!-- @include: @article-footer.snippet.md -->
