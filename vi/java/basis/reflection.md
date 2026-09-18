---
title: "Giải thích chi tiết cơ chế reflection trong Java"
description: "Giải thích chuyên sâu nguyên lý và cách sử dụng reflection trong Java: nắm vững các API cốt lõi Class, Method, Field, hiểu cách reflection được sử dụng trong các framework như Spring, MyBatis, đồng thời tìm hiểu cách triển khai dynamic proxy."
category: Java
tag:
  - Java Basics
head:
  - - meta
    - name: keywords
      content: Java reflection, cơ chế reflection, class Class, method Method, field Field, dynamic proxy, nguyên lý framework, thao tác runtime
---

## Reflection là gì?

Nếu bạn từng tìm hiểu nguyên lý bên trong của framework hoặc tự viết framework, chắc hẳn bạn không còn xa lạ với khái niệm reflection.

Reflection được gọi là linh hồn của framework chủ yếu vì nó cho phép chúng ta phân tích class và thực thi method trong class tại runtime.

Thông qua reflection, bạn có thể lấy thông tin về field, method, constructor và các thành phần khác của class, đồng thời gọi hoặc đọc/ghi chúng khi access control và ranh giới module cho phép. Phạm vi của các API khác nhau cũng khác nhau. Ví dụ, `getMethods()` trả về các method public có thể truy cập, còn `getDeclaredMethods()` trả về các method do class hiện tại khai báo nhưng không bao gồm method kế thừa.

## Bạn biết các trường hợp sử dụng reflection?

Trong phần lớn thời gian, chúng ta viết business code nên ít khi gặp trường hợp phải trực tiếp sử dụng cơ chế reflection.

Tuy nhiên, điều đó không có nghĩa reflection không hữu ích. Ngược lại, chính nhờ reflection mà bạn có thể sử dụng nhiều framework một cách dễ dàng. Các framework như Spring/Spring Boot, MyBatis và nhiều framework khác đều sử dụng rộng rãi cơ chế reflection.

**Các framework này cũng sử dụng rộng rãi dynamic proxy, còn việc triển khai dynamic proxy lại phụ thuộc vào reflection.**

Ví dụ dưới đây là code mẫu triển khai dynamic proxy bằng JDK, trong đó sử dụng class reflection `Method` để gọi method được chỉ định.

```java
public class DebugInvocationHandler implements InvocationHandler {
    /**
     * Đối tượng thực trong proxy class
     */
    private final Object target;

    public DebugInvocationHandler(Object target) {
        this.target = target;
    }


    public Object invoke(Object proxy, Method method, Object[] args) throws InvocationTargetException, IllegalAccessException {
        System.out.println("before method " + method.getName());
        Object result = method.invoke(target, args);
        System.out.println("after method " + method.getName());
        return result;
    }
}

```

Ngoài ra, một công cụ quan trọng trong Java là **annotation** cũng được triển khai bằng reflection.

Tại sao khi sử dụng Spring, chỉ một annotation `@Component` đã có thể khai báo một class là Spring Bean? Tại sao bạn có thể đọc được giá trị trong file cấu hình thông qua annotation `@Value`? Rốt cuộc chúng hoạt động như thế nào?

Tất cả là nhờ bạn có thể phân tích class dựa trên reflection, sau đó lấy annotation trên class/property/method/tham số của method. Sau khi lấy được annotation, bạn có thể xử lý tiếp.

## Hãy nói về ưu và nhược điểm của cơ chế reflection

**Ưu điểm**: Giúp code của chúng ta linh hoạt hơn và tạo thuận lợi cho việc cung cấp các chức năng có sẵn cho nhiều framework.

**Nhược điểm**: Cho phép phân tích và thao tác với class tại runtime, nhưng điều này cũng làm tăng vấn đề an toàn. Ví dụ, có thể bỏ qua kiểm tra an toàn của generic parameter (kiểm tra an toàn của generic parameter diễn ra tại thời điểm compile). Ngoài ra, performance của reflection cũng kém hơn một chút, nhưng với framework thì ảnh hưởng thực tế không đáng kể. Bài đọc liên quan: [Java Reflection: Why is it so slow?](https://stackoverflow.com/questions/1392351/java-reflection-why-is-it-so-slow)

## Thực hành reflection

### Bốn cách lấy Class object

Nếu muốn lấy động những thông tin này, chúng ta cần dựa vào Class object. Class object cho chương trình đang chạy biết thông tin như method, variable của một class. Java cung cấp bốn cách lấy Class object:

**1. Khi biết class cụ thể, có thể sử dụng:**

```java
Class alunbarClass = TargetObject.class;
```

Cách này phù hợp với trường hợp đã biết type cụ thể tại thời điểm compile. Việc lấy chính class literal không kích hoạt quá trình khởi tạo class.

**2. Truyền full path của class vào `Class.forName()` để lấy:**

```java
Class alunbarClass1 = Class.forName("cn.javaguide.TargetObject");
```

**3. Lấy thông qua object instance `instance.getClass()`:**

```java
TargetObject o = new TargetObject();
Class alunbarClass2 = o.getClass();
```

**4. Truyền path của class vào `xxxClassLoader.loadClass()` thông qua class loader để lấy:**

```java
ClassLoader.getSystemClassLoader().loadClass("cn.javaguide.TargetObject");
```

Lấy Class object thông qua class loader sẽ không thực hiện khởi tạo, tức là không thực hiện một loạt bước bao gồm khởi tạo. Static code block và static object sẽ không được thực thi.

### Một số thao tác cơ bản với reflection

1. Tạo class `TargetObject` mà chúng ta muốn thao tác bằng reflection.

```java
package cn.javaguide;

public class TargetObject {
    private String value;

    public TargetObject() {
        value = "JavaGuide";
    }

    public void publicMethod(String s) {
        System.out.println("I love " + s);
    }

    private void privateMethod() {
        System.out.println("value is " + value);
    }
}
```

2. Sử dụng reflection để thao tác với method và property của class này.

```java
package cn.javaguide;

import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class Main {
    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InstantiationException, InvocationTargetException, NoSuchFieldException {
        /**
         * Lấy Class object của class TargetObject và tạo instance của class TargetObject
         */
        Class<?> targetClass = Class.forName("cn.javaguide.TargetObject");
        TargetObject targetObject = (TargetObject) targetClass.getDeclaredConstructor().newInstance();
        /**
         * Lấy tất cả method được định nghĩa trong class TargetObject
         */
        Method[] methods = targetClass.getDeclaredMethods();
        for (Method method : methods) {
            System.out.println(method.getName());
        }

        /**
         * Lấy và gọi method được chỉ định
         */
        Method publicMethod = targetClass.getDeclaredMethod("publicMethod",
                String.class);

        publicMethod.invoke(targetObject, "JavaGuide");

        /**
         * Lấy parameter được chỉ định và sửa parameter
         */
        Field field = targetClass.getDeclaredField("value");
        // Hủy kiểm tra an toàn để sửa parameter trong class
        field.setAccessible(true);
        field.set(targetObject, "JavaGuide");

        /**
         * Gọi private method
         */
        Method privateMethod = targetClass.getDeclaredMethod("privateMethod");
        // Hủy kiểm tra an toàn để gọi private method
        privateMethod.setAccessible(true);
        privateMethod.invoke(targetObject);
    }
}

```

Kết quả output:

```plain
publicMethod
privateMethod
I love JavaGuide
value is JavaGuide
```

**Lưu ý**: Có độc giả cho biết code trên khi chạy sẽ ném exception `ClassNotFoundException`. Nguyên nhân cụ thể là bạn chưa thay package name trong đoạn code dưới đây bằng package chứa `TargetObject` do bạn tạo.
Bạn có thể tham khảo bài viết [này](https://www.cnblogs.com/chanshuyi/p/head_first_of_reflection.html).

```java
Class<?> targetClass = Class.forName("cn.javaguide.TargetObject");
```

<!-- @include: @article-footer.snippet.md -->
