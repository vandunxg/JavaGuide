---
title: "Giải thích chi tiết IoC & AOP (hiểu nhanh)"
description: "Giải thích chi tiết nguyên lý cốt lõi của Spring IoC và AOP, đi sâu vào cơ chế thực hiện Inversion of Control, Dependency Injection, lập trình hướng aspect và dynamic proxy."
category: Framework
tag:
  - Spring
head:
  - - meta
    - name: keywords
      content: IoC,DI,AOP,Spring IoC container,Dependency Injection,Aspect-oriented programming,dynamic proxy,Spring principles
---

Bài viết này sẽ giải thích IoC & AOP dựa trên các câu hỏi sau:

- IoC là gì?
- IoC giải quyết vấn đề gì?
- IoC và DI khác nhau thế nào?
- AOP là gì?
- AOP giải quyết vấn đề gì?
- AOP có những trường hợp sử dụng nào?
- Vì sao AOP được gọi là lập trình hướng aspect?
- Có những cách nào để triển khai AOP?

Trước hết cần nói rõ: IoC & AOP không phải do Spring đề xuất. Thực ra chúng đã tồn tại trước Spring, chỉ là khi đó thiên về lý thuyết hơn. Spring đã triển khai rất tốt hai tư tưởng này ở cấp độ kỹ thuật.

## IoC (Inversion of Control)

### IoC là gì?

IoC (Inversion of Control) nghĩa là đảo ngược quyền kiểm soát. Đây là một tư tưởng, không phải một triển khai kỹ thuật. Nó mô tả vấn đề tạo và quản lý object trong lĩnh vực phát triển Java.

Ví dụ: class A hiện đang phụ thuộc vào class B.

- **Cách phát triển truyền thống**: thường tạo thủ công một object B trong class A bằng keyword `new`.
- **Cách phát triển theo tư tưởng IoC**: không tạo object bằng keyword `new`, mà nhờ IoC container (framework Spring) khởi tạo object. Khi cần object nào, bạn chỉ cần lấy object đó từ IoC container.

So sánh hai cách phát triển trên: chúng ta “mất đi một quyền” (quyền tạo và quản lý object), nhưng đồng thời nhận được một lợi ích (không cần tiếp tục lo việc tạo, quản lý object và một loạt vấn đề liên quan).

**Vì sao gọi là Inversion of Control?**

- **Control**: chỉ quyền tạo object (khởi tạo và quản lý).
- **Inversion**: quyền kiểm soát được giao cho môi trường bên ngoài (IoC container).

![Sơ đồ IoC](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/IoC&Aop-ioc-illustration.png)

### IoC giải quyết vấn đề gì?

Tư tưởng IoC là hai bên không phụ thuộc lẫn nhau, mà để container bên thứ ba quản lý các resource liên quan. Cách này có những lợi ích gì?

1. Giảm mức độ coupling, hay mức độ phụ thuộc, giữa các object;
2. Resource dễ quản lý hơn; ví dụ, nếu sử dụng container Spring cung cấp, bạn có thể dễ dàng triển khai singleton.

Ví dụ: thực hiện thao tác với User bằng cấu trúc hai layer Service và Dao.

Khi chưa sử dụng tư tưởng IoC, nếu layer Service muốn sử dụng implementation cụ thể của layer Dao, cần dùng keyword `new` để tạo thủ công implementation cụ thể `IUserDao` là `UserDaoImpl` trong `UserServiceImpl` (không thể trực tiếp `new` interface).

Cách này hoàn toàn có thể hoạt động, nhưng hãy hình dung tình huống sau:

Trong quá trình phát triển, đột nhiên có một yêu cầu mới: cần xây dựng một implementation cụ thể khác cho interface `IUserDao`. Vì layer Service phụ thuộc vào implementation cụ thể của `IUserDao`, chúng ta cần sửa object được tạo bằng `new` trong `UserServiceImpl`. Nếu chỉ có một class tham chiếu đến implementation cụ thể của `IUserDao` thì việc sửa có thể không quá khó. Nhưng nếu có rất nhiều nơi cùng tham chiếu đến implementation cụ thể của `IUserDao`, khi cần thay đổi implementation của `IUserDao`, việc sửa sẽ rất khó khăn.

![IoC&Aop-ioc-illustration-dao-service](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/IoC&Aop-ioc-illustration-dao-service.png)

Với tư tưởng IoC, chúng ta giao quyền kiểm soát object (tạo và quản lý) cho IoC container quản lý. Khi sử dụng, chúng ta chỉ cần “yêu cầu” object từ IoC container.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/IoC&Aop-ioc-illustration-dao.png)

### IoC và DI có khác nhau không?

IoC (Inversion of Control: đảo ngược quyền kiểm soát) là một ý tưởng thiết kế, hay một pattern nào đó. Ý tưởng thiết kế này là **giao quyền kiểm soát việc tạo object vốn được thực hiện thủ công trong chương trình cho bên thứ ba, chẳng hạn IoC container.** Với framework Spring thường dùng, IoC container thực tế là một Map (key, value), trong Map lưu trữ nhiều object khác nhau. Tuy nhiên, IoC cũng được ứng dụng trong các ngôn ngữ khác, không phải tính năng riêng của Spring.

Cách triển khai IoC phổ biến và hợp lý nhất được gọi là Dependency Injection (viết tắt là DI).

Martin Fowler đã đề cập trong một bài viết rằng nên đổi tên IoC thành DI. Nội dung gốc và địa chỉ bài viết: <https://martinfowler.com/articles/injection.html>.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/martin-fowler-injection.png)

Ý chính của Martin Fowler là IoC quá phổ biến nhưng không thể hiện rõ ý nghĩa, khiến nhiều người bối rối. Vì vậy, dùng DI để gọi chính xác pattern này sẽ phù hợp hơn.

## AOP (Aspect oriented programming)

Phần này không đi sâu vào quá nhiều thuật ngữ chuyên môn; mục tiêu chính là làm rõ tư tưởng AOP.

### AOP là gì?

AOP (Aspect Oriented Programming) nghĩa là lập trình hướng aspect. AOP là một sự tiếp nối của OOP (lập trình hướng object); hai bên bổ sung cho nhau, không đối lập.

Mục đích của AOP là tách các cross-cutting concerns (chẳng hạn ghi log, quản lý transaction, kiểm soát quyền, rate limiting cho API, idempotency cho API và các chức năng khác) khỏi core business logic. Thông qua các kỹ thuật như dynamic proxy và thao tác bytecode, AOP thực hiện việc tái sử dụng và decoupling code, nâng cao khả năng maintain và mở rộng của code. Mục đích của OOP là đóng gói business logic theo thuộc tính và hành vi của object, thông qua các khái niệm như class, object, inheritance và polymorphism để thực hiện modular hóa và phân tầng code (đồng thời cũng có thể tái sử dụng code), nâng cao tính dễ đọc và khả năng maintain của code.

### Vì sao AOP được gọi là lập trình hướng aspect?

AOP được gọi là lập trình hướng aspect vì tư tưởng cốt lõi của nó là tách các cross-cutting concerns khỏi core business logic, hình thành từng **aspect**.

![Sơ đồ lập trình hướng aspect](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/aop-program-execution.jpg)

Sau đây là phần tổng hợp các thuật ngữ quan trọng của AOP (nếu chưa hiểu cũng không sao, bạn có thể tiếp tục đọc):

- **Cross-cutting concerns**: hành vi dùng chung trong nhiều class hoặc object (chẳng hạn ghi log, quản lý transaction, kiểm soát quyền, rate limiting cho API, idempotency cho API và các chức năng khác).
- **Aspect**: class đóng gói cross-cutting concerns; một aspect là một class. Aspect có thể định nghĩa nhiều advice để triển khai chức năng cụ thể.
- **JoinPoint**: một thời điểm cụ thể khi method được gọi hoặc thực thi (chẳng hạn lúc gọi method, lúc exception được ném ra).
- **Advice**: thao tác aspect cần thực hiện tại một join point. Có năm loại advice: Before, After, AfterReturning, AfterThrowing và Around. Bốn loại advice đầu tiên thực thi trước hoặc sau target method, còn Around có thể kiểm soát quá trình thực thi target method.
- **Pointcut**: một expression dùng để khớp những join point cần được aspect enhance. Pointcut có thể được định nghĩa bằng annotation, regular expression, phép toán logic và các cách khác. Ví dụ, `execution(* com.xyz.service..*(..))` khớp các class hoặc interface trong package `com.xyz.service` và các subpackage của nó.
- **Weaving**: quá trình kết nối aspect với target object, tức áp dụng advice vào các join point được pointcut khớp. Có hai thời điểm weaving phổ biến: Compile-Time Weaving (ví dụ AspectJ) và Runtime Weaving (ví dụ AspectJ, Spring AOP).

### AOP có những loại advice phổ biến nào?

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/aspectj-advice-types.jpg)

- **Before**: kích hoạt trước khi method của target object được gọi.
- **After**: kích hoạt sau khi method của target object được gọi.
- **AfterReturning**: kích hoạt sau khi method của target object thực thi xong và trả về giá trị kết quả.
- **AfterThrowing**: kích hoạt sau khi method của target object ném hoặc phát sinh exception trong quá trình chạy. AfterReturning và AfterThrowing loại trừ lẫn nhau. Nếu method gọi thành công và không có exception thì sẽ có giá trị trả về; nếu method ném exception thì sẽ không có giá trị trả về.
- **Around**: điều khiển việc gọi method của target object theo cách lập trình. Around có phạm vi thao tác lớn nhất trong tất cả các loại advice vì có thể trực tiếp lấy target object và method cần thực thi. Do đó, Around có thể thực hiện bất kỳ thao tác nào trước hoặc sau khi gọi method của target object, thậm chí không gọi method của target object.

### AOP giải quyết vấn đề gì?

OOP không thể xử lý tốt một số hành vi dùng chung phân tán trong nhiều class hoặc object (chẳng hạn ghi log, quản lý transaction, kiểm soát quyền, rate limiting cho API, idempotency cho API và các chức năng khác). Những hành vi này thường được gọi là **cross-cutting concerns**. Nếu lặp lại việc triển khai các hành vi này trong từng class hoặc object, code sẽ bị dư thừa, phức tạp và khó maintain.

AOP có thể tách cross-cutting concerns (chẳng hạn ghi log, quản lý transaction, kiểm soát quyền, rate limiting cho API, idempotency cho API và các chức năng khác) khỏi **core concerns (core business logic)**, thực hiện separation of concerns.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/crosscut-logic-and-businesslogic-separation%20%20%20%20%20%20.png)

Lấy ghi log làm ví dụ. Giả sử cần ghi log theo một format thống nhất cho một số method. Trước khi sử dụng AOP, chúng ta phải lần lượt viết logic ghi log cho từng method; toàn bộ đều là logic lặp lại.

```java
public CommonResponse<Object> method1() {
      // Business logic
      xxService.method1();
      // Bỏ qua phần xử lý business logic cụ thể
      // Ghi log
      ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
      HttpServletRequest request = attributes.getRequest();
      // Bỏ qua logic ghi log cụ thể, chẳng hạn lấy nhiều thông tin, ghi vào database...
      return CommonResponse.success();
}

public CommonResponse<Object> method2() {
      // Business logic
      xxService.method2();
      // Bỏ qua phần xử lý business logic cụ thể
      // Ghi log
      ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
      HttpServletRequest request = attributes.getRequest();
      // Bỏ qua logic ghi log cụ thể, chẳng hạn lấy nhiều thông tin, ghi vào database...
      return CommonResponse.success();
}

// ...
```

Sau khi sử dụng AOP, chúng ta có thể đóng gói logic ghi log thành một aspect, sau đó chỉ định những method nào cần thực hiện thao tác ghi log thông qua pointcut và advice.

```java

// Annotation log
@Target({ElementType.PARAMETER,ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Log {

    /**
     * Mô tả
     */
    String description() default "";

    /**
     * Loại method INSERT DELETE UPDATE OTHER
     */
    MethodType methodType() default MethodType.OTHER;
}

// Aspect log
@Component
@Aspect
public class LogAspect {
  // Pointcut, tất cả method được đánh dấu bằng annotation Log
  @Pointcut("@annotation(cn.javaguide.annotation.Log)")
  public void webLog() {
  }

   /**
   * Around advice
   */
  @Around("webLog()")
  public Object doAround(ProceedingJoinPoint joinPoint) throws Throwable {
    // Bỏ qua phần xử lý cụ thể
  }

  // Bỏ qua phần code còn lại
}
```

Như vậy, chỉ cần một annotation là có thể thực hiện ghi log:

```java
@Log(description = "method1",methodType = MethodType.INSERT)
public CommonResponse<Object> method1() {
      // Business logic
      xxService.method1();
      // Bỏ qua phần xử lý business logic cụ thể
      return CommonResponse.success();
}
```

### AOP có những trường hợp sử dụng nào?

- Ghi log: định nghĩa annotation ghi log tùy chỉnh, dùng AOP để thực hiện ghi log chỉ với một dòng code.
- Thống kê performance: dùng AOP để thống kê thời gian thực thi method trước và sau khi target method chạy, thuận tiện cho việc tối ưu và phân tích.
- Quản lý transaction: annotation `@Transactional` cho phép Spring quản lý transaction, chẳng hạn rollback thao tác phát sinh exception, tránh phải lặp lại logic quản lý transaction. Annotation `@Transactional` được triển khai dựa trên AOP.
- Kiểm soát quyền: dùng AOP để kiểm tra user có quyền cần thiết trước khi target method thực thi. Nếu có quyền thì thực thi target method, nếu không thì không thực thi. Ví dụ, SpringSecurity cho phép tùy chỉnh kiểm tra quyền chỉ với một dòng code bằng annotation `@PreAuthorize`.
- Rate limiting cho API: dùng AOP để giới hạn request bằng algorithm rate limiting cụ thể trước khi target method thực thi.
- Quản lý cache: dùng AOP để đọc và cập nhật cache trước và sau khi target method thực thi.
- ……

### Có những cách nào để triển khai AOP?

Các cách triển khai AOP phổ biến gồm dynamic proxy, thao tác bytecode và các cách khác.

Spring AOP dựa trên dynamic proxy. Nếu object cần proxy triển khai một interface nào đó, Spring AOP sẽ dùng **JDK Proxy** để tạo proxy object. Với object không triển khai interface, JDK Proxy không thể proxy object đó. Khi ấy, Spring AOP sẽ dùng CGLIB tạo subclass của object được proxy để làm proxy, như hình dưới đây:

![SpringAOPProcess](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/230ae587a322d6e4d09510161987d346.jpeg)

**Strategy dynamic proxy của Spring Boot và Spring có giống nhau không?** Thực ra là không, nhiều người hiểu sai điểm này.

Trước Spring Boot 2.0, giá trị mặc định của `spring.aop.proxy-target-class` là `false`; khi có user interface, thông thường sử dụng **JDK dynamic proxy**. Nếu target class không có interface khả dụng, Spring AOP vẫn fallback về **CGLIB dynamic proxy**, không chỉ vì target class không triển khai interface mà ném exception. Code auto-configuration AOP của Spring Boot 1.5.x như sau:

```java
@Configuration
@ConditionalOnClass({ EnableAspectJAutoProxy.class, Aspect.class, Advice.class })
@ConditionalOnProperty(prefix = "spring.aop", name = "auto", havingValue = "true", matchIfMissing = true)
public class AopAutoConfiguration {

	@Configuration
	@EnableAspectJAutoProxy(proxyTargetClass = false)
 // Configuration class này chỉ có hiệu lực khi spring.aop.proxy-target-class=false hoặc chưa được cấu hình rõ.
 // Nói cách khác, nếu developer chưa chỉ định rõ strategy proxy, Spring sẽ mặc định load JDK dynamic proxy.
	@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "false", matchIfMissing = true)
	public static class JdkDynamicAutoProxyConfiguration {

	}

	@Configuration
	@EnableAspectJAutoProxy(proxyTargetClass = true)
 // Configuration class này chỉ có hiệu lực khi spring.aop.proxy-target-class=true.
 // Tức là khi developer chỉ định rõ dùng CGLIB dynamic proxy thông qua property, Spring sẽ load configuration class này.
	@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "true", matchIfMissing = false)
	public static class CglibAutoProxyConfiguration {

	}

}
```

Từ Spring Boot 2.0, nếu user không cấu hình gì thì mặc định sử dụng **CGLIB dynamic proxy**. Nếu muốn bắt buộc sử dụng JDK dynamic proxy, có thể thêm `spring.aop.proxy-target-class=false` vào file configuration. Code auto-configuration AOP của Spring Boot 2.0 như sau:

```java
@Configuration
@ConditionalOnClass({ EnableAspectJAutoProxy.class, Aspect.class, Advice.class,
		AnnotatedElement.class })
@ConditionalOnProperty(prefix = "spring.aop", name = "auto", havingValue = "true", matchIfMissing = true)
public class AopAutoConfiguration {

	@Configuration
	@EnableAspectJAutoProxy(proxyTargetClass = false)
 // Configuration class này chỉ có hiệu lực khi spring.aop.proxy-target-class=false.
 // Tức là khi developer chỉ định rõ dùng JDK dynamic proxy thông qua property, Spring sẽ load configuration class này.
	@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "false", matchIfMissing = false)
	public static class JdkDynamicAutoProxyConfiguration {

	}

	@Configuration
	@EnableAspectJAutoProxy(proxyTargetClass = true)
 // Configuration class này chỉ có hiệu lực khi spring.aop.proxy-target-class=true hoặc chưa được cấu hình rõ.
 // Nói cách khác, nếu developer chưa chỉ định rõ strategy proxy, Spring sẽ mặc định load CGLIB proxy.
	@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "true", matchIfMissing = true)
	public static class CglibAutoProxyConfiguration {

	}

}
```

Tất nhiên, bạn cũng có thể sử dụng **AspectJ**! Spring AOP đã tích hợp AspectJ. AspectJ có thể được xem là framework AOP hoàn chỉnh nhất trong hệ sinh thái Java.

**Spring AOP thực hiện enhancement ở runtime, còn AspectJ hỗ trợ weaving tại compile time, sau khi compile và khi class được load.** Spring AOP dựa trên proxy (Proxying), còn AspectJ dựa trên thao tác bytecode (Bytecode Manipulation).

Spring AOP đã tích hợp AspectJ. AspectJ có thể được xem là framework AOP hoàn chỉnh nhất trong hệ sinh thái Java. So với Spring AOP, AspectJ mạnh hơn về tính năng, nhưng Spring AOP tương đối đơn giản hơn.

Nếu aspect của chúng ta ít, chênh lệch performance giữa hai bên không đáng kể. Nhưng khi có quá nhiều aspect, tốt nhất nên chọn AspectJ vì nó nhanh hơn Spring AOP rất nhiều.

## Tham khảo

- AOP in Spring Boot, is it a JDK dynamic proxy or a Cglib dynamic proxy?: <https://www.springcloud.io/post/2022-01/springboot-aop/>
- Spring Proxying Mechanisms: <https://docs.spring.io/spring-framework/reference/core/aop/proxying.html>
