---
title: Tổng hợp câu hỏi phỏng vấn Spring thường gặp
description: Giải thích chi tiết các câu hỏi phỏng vấn cốt lõi về Spring Framework, bao quát IoC container, nguyên lý AOP, vòng đời Bean, dependency injection và các kiến thức cốt lõi khác của Spring.
category: Framework
tag:
  - Spring
head:
  - - meta
    - name: keywords
      content: câu hỏi phỏng vấn Spring,Spring Framework,vòng đời Bean,IoC,AOP,dependency injection,transaction,câu hỏi Spring thường gặp
---

Bài viết này chủ yếu muốn giúp bạn hiểu sâu hơn về Spring thông qua một số câu hỏi, vì vậy sẽ không đề cập quá nhiều đến code!

Nhiều câu hỏi dưới đây tôi cũng không chú ý trong quá trình sử dụng Spring, mà phải tạm thời tra cứu nhiều tài liệu và sách để bổ sung. Trên Internet cũng có nhiều bài tổng hợp câu hỏi thường gặp/câu hỏi phỏng vấn Spring, nhưng tôi thấy phần lớn sao chép lẫn nhau, nhiều câu hỏi cũng chưa tốt và một số câu trả lời còn có vấn đề. Vì vậy, tôi đã dành một tuần ngoài giờ để tổng hợp lại, hy vọng bài viết có ích cho bạn.

## Spring Basics

### Spring Framework là gì?

Spring là một Java development framework mã nguồn mở, nhẹ, nhằm nâng cao hiệu quả phát triển của developer và khả năng bảo trì hệ thống.

Thông thường khi nói Spring Framework, chúng ta đều chỉ Spring Framework. Đây là tập hợp của nhiều module, có thể hỗ trợ rất thuận tiện cho việc phát triển, chẳng hạn Spring hỗ trợ IoC (Inversion of Control: đảo ngược quyền kiểm soát) và AOP (Aspect-Oriented Programming: lập trình hướng khía cạnh), truy cập database thuận tiện, tích hợp component bên thứ ba thuận tiện (email, task, scheduling, cache, v.v.), hỗ trợ tốt cho unit test và hỗ trợ phát triển ứng dụng Java RESTful.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/38ef122122de4375abcd27c3de8f60b4.png)

Tư tưởng cốt lõi nhất của Spring là không phát minh lại bánh xe, dùng được ngay sau khi cài đặt và nâng cao hiệu quả phát triển.

Spring dịch ra có nghĩa là mùa xuân, có thể thấy mục tiêu và sứ mệnh của nó là mang mùa xuân đến cho các lập trình viên Java! Cảm động!

🤐 Nói thêm một chút: **Một ngôn ngữ thường cần một ứng dụng chủ lực để trở nên phổ biến, Spring chính là một application framework chủ lực của hệ sinh thái Java.**

Các chức năng cốt lõi Spring cung cấp chủ yếu là IoC và AOP. Khi học Spring, nhất định phải hiểu rõ tư tưởng cốt lõi của IoC và AOP!

- Trang chủ Spring: <https://spring.io/>
- Địa chỉ GitHub: <https://github.com/spring-projects/spring-framework>

### Spring gồm những module nào?

**Phiên bản Spring4.x**:

![Các module chính của Spring4.x](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/jvme0c60b4606711fc4a0b6faf03230247a.png)

**Phiên bản Spring5.x**:

![Các module chính của Spring5.x](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/20200831175708.png)

Trong Spring5.x, component Portlet của module Web đã bị deprecated, đồng thời bổ sung component WebFlux dùng cho xử lý reactive bất đồng bộ.

Quan hệ dependency giữa các module Spring như sau:

![Quan hệ dependency giữa các module Spring](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/20200902100038.png)

#### Core Container

Đây là module cốt lõi, cũng có thể nói là module nền tảng của Spring Framework, chủ yếu cung cấp hỗ trợ cho chức năng IoC dependency injection. Hầu hết các chức năng khác của Spring đều phụ thuộc vào module này, có thể thấy điều đó từ sơ đồ quan hệ dependency giữa các module Spring ở trên.

- **spring-core**: Các utility class cốt lõi của Spring Framework.
- **spring-beans**: Cung cấp hỗ trợ cho việc tạo, cấu hình và quản lý bean.
- **spring-context**: Cung cấp hỗ trợ cho internationalization, event propagation, resource loading và các chức năng khác.
- **spring-expression**: Cung cấp hỗ trợ cho expression language (Spring Expression Language) SpEL, chỉ phụ thuộc vào module core, không phụ thuộc các module khác và có thể sử dụng độc lập.

#### AOP

- **spring-aspects**: Module này cung cấp hỗ trợ tích hợp với AspectJ.
- **spring-aop**: Cung cấp triển khai lập trình hướng khía cạnh.
- **spring-instrument**: Cung cấp chức năng thêm agent cho JVM. Cụ thể, nó cung cấp cho Tomcat một weaving agent, có thể truyền class file cho Tomcat như thể các file này được class loader load. Không hiểu cũng không sao, phạm vi sử dụng của module này rất hạn chế.

#### Data Access/Integration

Danh sách module dưới đây chủ yếu dựa trên Spring Framework 5.x. Spring Framework hiện đại đã loại bỏ một số tích hợp công nghệ cũ, khi sử dụng thực tế cần dựa trên danh sách module chính thức của phiên bản Spring mục tiêu.

- **spring-jdbc**: Cung cấp abstraction JDBC cho database access. Mỗi database có API độc lập để thao tác, còn Java application chỉ cần tương tác với JDBC API, nhờ đó che giấu ảnh hưởng của database.
- **spring-tx**: Cung cấp hỗ trợ cho transaction.
- **spring-orm**: Trong Spring Framework 5.x, cung cấp hỗ trợ cho các công nghệ ORM như Hibernate, JPA; các phiên bản Spring cũ hơn còn từng cung cấp tích hợp iBATIS.
- **spring-oxm**: Cung cấp abstraction OXM (Object-to-XML Mapping). Các phiên bản Spring khác nhau hỗ trợ implementation cụ thể khác nhau, chẳng hạn JAXB; Castor, XMLBeans, JiBX thuộc các tích hợp của phiên bản cũ.
- **spring-jms**: Message service. Từ Spring Framework 4.1, module này còn cung cấp kế thừa cho module spring-messaging.

#### Spring Web

- **spring-web**: Cung cấp một số hỗ trợ cơ bản nhất cho việc triển khai chức năng Web.
- **spring-webmvc**: Cung cấp triển khai Spring MVC.
- **spring-websocket**: Cung cấp hỗ trợ cho WebSocket, cho phép client và server giao tiếp hai chiều.
- **spring-webflux**: Cung cấp hỗ trợ cho WebFlux. WebFlux là Web framework reactive, non-blocking được giới thiệu trong Spring Framework 5.0, có thể chạy trên Netty hoặc trên Servlet container hỗ trợ non-blocking I/O. Ứng dụng có non-blocking end-to-end hay không còn phụ thuộc vào việc data access và các downstream call khác có chứa blocking operation hay không.

#### Messaging

**spring-messaging** là module mới được thêm từ Spring4.0, chủ yếu chịu trách nhiệm tích hợp một số ứng dụng truyền message nền tảng cho Spring Framework.

#### Spring Test

Team Spring khuyến khích test-driven development (TDD). Nhờ inversion of control (IoC), unit test và integration test trở nên đơn giản hơn.

Module test của Spring hỗ trợ khá tốt các test framework phổ biến như JUnit (unit test framework), TestNG (tương tự JUnit), Mockito (chủ yếu dùng để Mock object), PowerMock (giải quyết các vấn đề của Mockito như không thể mock final, static, private method), v.v.

### ⭐️Quan hệ giữa Spring, Spring MVC và Spring Boot là gì?

Nhiều người thường nhầm lẫn giữa Spring, Spring MVC và Spring Boot! Phần này giới thiệu ngắn gọn về ba thành phần này, thực ra rất đơn giản, không có gì cao siêu.

Spring gồm nhiều functional module (đã đề cập ở trên), trong đó quan trọng nhất là module Spring-Core (chủ yếu cung cấp hỗ trợ cho IoC dependency injection). Việc triển khai chức năng của các module khác trong Spring (chẳng hạn Spring MVC) về cơ bản đều cần phụ thuộc vào module này.

Hình dưới đây tương ứng với phiên bản Spring 4.x. Spring 5.0 giới thiệu WebFlux dùng cho xử lý reactive và dần loại bỏ hỗ trợ liên quan đến Portlet; thành phần module của các phiên bản Spring hiện đại cần dựa trên tài liệu chính thức.

![Các module chính của Spring](https://oss.javaguide.cn/github/javaguide/jvme0c60b4606711fc4a0b6faf03230247a.png)

Spring MVC là một module rất quan trọng của Spring, chủ yếu trao cho Spring khả năng nhanh chóng xây dựng Web application theo kiến trúc MVC. MVC là viết tắt của Model, View và Controller, tư tưởng cốt lõi là tổ chức code bằng cách tách business logic, data và display.

![](https://oss.javaguide.cn/java-guide-blog/image-20210809181452421.png)

Khi dùng Spring để phát triển, việc phải cấu hình mọi thứ quá phức tạp, chẳng hạn khi bật một số tính năng Spring cần cấu hình tường minh bằng XML hoặc Java. Vì vậy, Spring Boot ra đời!

Spring hướng tới việc đơn giản hóa phát triển ứng dụng enterprise J2EE. Spring Boot hướng tới việc đơn giản hóa phát triển Spring (giảm file cấu hình, dùng được ngay sau khi cài đặt!).

Spring Boot chỉ đơn giản hóa configuration. Nếu cần xây dựng Web application theo kiến trúc MVC, bạn vẫn cần dùng Spring MVC làm MVC framework; chỉ là Spring Boot giúp đơn giản hóa nhiều configuration của Spring MVC, thực sự dùng được ngay sau khi cài đặt!

## Spring IoC

### ⭐️IoC là gì?

IoC (Inversion of Control) nghĩa là đảo ngược quyền kiểm soát/đảo ngược control. Đây là một tư tưởng chứ không phải một technical implementation. Nó mô tả vấn đề tạo và quản lý object trong lĩnh vực phát triển Java.

Ví dụ: class A hiện đang phụ thuộc vào class B.

- **Cách phát triển truyền thống**: thường tự dùng từ khóa new trong class A để tạo một object B.
- **Cách phát triển theo tư tưởng IoC**: không tạo object bằng từ khóa new, mà nhờ IoC container (Spring Framework) khởi tạo object. Cần object nào thì lấy trực tiếp object đó từ IoC container.

So sánh hai cách phát triển trên: chúng ta “mất đi một quyền” (quyền tạo và quản lý object), nhưng đổi lại nhận được một lợi ích (không cần quan tâm đến hàng loạt việc như tạo và quản lý object).

**Vì sao gọi là đảo ngược quyền kiểm soát?**

- **Control**: quyền tạo object (khởi tạo và quản lý).
- **Inversion**: trao quyền kiểm soát cho môi trường bên ngoài (IoC container).

![Minh họa IoC](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/IoC&Aop-ioc-illustration.png)

### ⭐️IoC giải quyết vấn đề gì?

Tư tưởng IoC là hai bên không phụ thuộc lẫn nhau, resource liên quan do container bên thứ ba quản lý. Điều này có lợi ích gì?

1. Giảm coupling, hay mức độ dependency, giữa các object;
2. Resource dễ quản lý hơn; ví dụ nếu dùng Spring container cung cấp thì rất dễ triển khai singleton.

Ví dụ: có một thao tác với User, phát triển theo cấu trúc hai tầng Service và Dao.

Khi chưa dùng tư tưởng IoC, nếu tầng Service muốn sử dụng implementation cụ thể của tầng Dao, cần dùng từ khóa new trong `UserServiceImpl` để tự tạo implementation cụ thể `UserDaoImpl` của `IUserDao` (không thể trực tiếp new interface).

Cách này hoàn toàn có thể hoạt động, nhưng hãy hình dung tình huống sau:

Trong quá trình phát triển đột nhiên có requirement mới, cần phát triển một implementation cụ thể khác cho interface `IUserDao`. Vì tầng Service phụ thuộc vào implementation cụ thể của `IUserDao`, chúng ta phải sửa object được new trong `UserServiceImpl`. Nếu chỉ có một class tham chiếu implementation cụ thể của `IUserDao` thì có thể chưa đáng ngại, nhưng nếu có rất nhiều nơi tham chiếu implementation cụ thể của `IUserDao`, khi cần thay đổi cách triển khai `IUserDao` thì việc sửa sẽ vô cùng đau đầu.

![IoC&Aop-ioc-illustration-dao-service](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/IoC&Aop-ioc-illustration-dao-service.png)

Theo tư tưởng IoC, chúng ta giao quyền kiểm soát object (tạo và quản lý) cho IoC container. Khi sử dụng, chỉ cần trực tiếp “yêu cầu” IoC container là được.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/IoC&Aop-ioc-illustration-dao.png)

### Spring Bean là gì?

Nói đơn giản, Bean là cách gọi những object do IoC container quản lý.

Chúng ta cần cho IoC container biết những object nào cần quản lý; điều này được định nghĩa thông qua configuration metadata. Configuration metadata có thể là file XML, annotation hoặc Java configuration class.

```xml
<!-- Constructor-arg with 'value' attribute -->
<bean id="..." class="...">
   <constructor-arg value="..."/>
</bean>
```

Hình dưới đây minh họa đơn giản cách IoC container sử dụng configuration metadata để quản lý object.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/062b422bd7ac4d53afd28fb74b2bc94d.png)

Hai package `org.springframework.beans` và `org.springframework.context` là nền tảng của implementation IoC. Nếu muốn nghiên cứu source code liên quan đến IoC, bạn có thể xem qua.

### Những annotation nào dùng để khai báo một class là Bean?

- `@Component`: Annotation thông dụng, có thể đánh dấu bất kỳ class nào là component của `Spring`. Nếu không biết một Bean thuộc layer nào, có thể dùng annotation `@Component` để đánh dấu.
- `@Repository`: Tương ứng với persistence layer, tức Dao layer, chủ yếu dùng cho các thao tác liên quan đến database.
- `@Service`: Tương ứng với service layer, chủ yếu chứa một số logic phức tạp và cần dùng đến Dao layer.
- `@Controller`: Tương ứng với Spring MVC control layer, chủ yếu dùng để nhận request của user, gọi `Service` và trả data về frontend page.

### Sự khác nhau giữa @Component và @Bean là gì?

- Annotation `@Component` áp dụng cho class, còn annotation `@Bean` áp dụng cho method.
- `@Component` thường được tự động phát hiện và auto-assemble vào Spring container thông qua classpath scanning (có thể dùng annotation `@ComponentScan` để định nghĩa path cần scan, tìm các class được đánh dấu cần assemble và tự động assemble vào bean container của Spring). Annotation `@Bean` thường được dùng để định nghĩa bean được tạo ra bên trong method mang annotation này. `@Bean` cho Spring biết đây là instance của một class và trả instance đó khi cần dùng.
- Annotation `@Bean` có tính tùy biến mạnh hơn `@Component`, và nhiều nơi chỉ có thể đăng ký bean bằng annotation `@Bean`. Ví dụ, khi cần assemble class từ third-party library vào `Spring` container thì chỉ có thể thực hiện bằng `@Bean`.

Ví dụ sử dụng annotation `@Bean`:

```java
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }

}
```

Code trên tương đương với cấu hình XML dưới đây:

```xml
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```

Ví dụ dưới đây không thể thực hiện bằng `@Component`.

```java
@Bean
public OneService getService(status) {
    case (status)  {
        when 1:
                return new serviceImpl1();
        when 2:
                return new serviceImpl2();
        when 3:
                return new serviceImpl3();
    }
}
```

### Những annotation nào dùng để inject Bean?

`@Autowired` do Spring cung cấp, cùng với `@Resource` và `@Inject` do Jakarta specification cung cấp, đều có thể dùng để inject Bean.

| Annotation   | Package                                        | Source                                 |
| ------------ | ---------------------------------------------- | -------------------------------------- |
| `@Autowired` | `org.springframework.beans.factory.annotation` | Spring 2.5+                            |
| `@Resource`  | `jakarta.annotation` (Spring 6+)               | Jakarta Annotations / JSR-250          |
| `@Inject`    | `jakarta.inject` (Spring 6+)                   | Jakarta Dependency Injection / JSR-330 |

`@Autowired` và `@Resource` được sử dụng nhiều hơn.

### ⭐️Sự khác nhau giữa @Autowired và @Resource là gì?

`@Autowired` là annotation tích hợp sẵn của Spring, logic inject mặc định là **ưu tiên match theo type (byType), nếu có nhiều Bean cùng type thì tiếp tục lọc theo name (byName)**.

Cụ thể:

1. Ưu tiên tìm Bean phù hợp trong Spring container dựa trên type của interface/class. Nếu chỉ tìm thấy một Bean phù hợp với type thì inject trực tiếp, không cần xét name;
2. Nếu tìm thấy nhiều Bean cùng type (ví dụ một interface có nhiều implementation), sẽ thử match **property name hoặc parameter name** với name của Bean (name mặc định của Bean là class name viết thường chữ cái đầu, trừ khi được chỉ định tường minh bằng `@Bean(name = "...")` hoặc `@Component("...")`).

Khi một interface có nhiều implementation:

- Nếu property name trùng với name của một Bean thì inject Bean đó;
- Nếu property name không trùng với tên của bất kỳ Bean nào thì sẽ ném `NoUniqueBeanDefinitionException`, lúc này cần dùng `@Qualifier` để chỉ định tường minh tên Bean cần inject.

Ví dụ:

```java
// Interface SmsService có hai implementation: SmsServiceImpl1, SmsServiceImpl2 (đều do Spring quản lý)

// Lỗi: byType match nhiều Bean, property name "smsService" không trùng tên mặc định của hai implementation (smsServiceImpl1, smsServiceImpl2)
@Autowired
private SmsService smsService;

// Đúng: property name "smsServiceImpl1" trùng tên mặc định của implementation SmsServiceImpl1
@Autowired
private SmsService smsServiceImpl1;

// Đúng: dùng @Qualifier để chỉ định tường minh tên Bean "smsServiceImpl1"
@Autowired
@Qualifier(value = "smsServiceImpl1")
private SmsService smsService;
```

Trong thực tế phát triển, chúng tôi vẫn khuyến nghị dùng annotation `@Qualifier` để chỉ định tường minh name thay vì phụ thuộc vào name của variable.

`@Resource` bắt nguồn từ specification **JSR-250**. Trong JDK 6 đến JDK 10, `javax.annotation.Resource` được cung cấp kèm JDK; từ JDK 11 cần import API dependency riêng. Project Spring 5/Java EE 8 thường dùng `javax.annotation-api`, còn project Spring 6/Jakarta EE 9 trở lên dùng `jakarta.annotation-api`.

Spring xử lý `@Resource` (trường hợp không có parameter) như sau:

1. **Match theo name (byName):** Mặc định lấy field name làm tên bean để tìm trong container. Nếu tìm thấy Bean có name đó thì inject trực tiếp.
2. **Fallback sang match theo type (byType):** Nếu **không** tìm thấy Bean cùng name, Spring sẽ thử tìm theo **type** của field. **Kết quả match theo type:**
   - **Tìm thấy 1 Bean**: inject thành công.
   - **Tìm thấy 0 Bean**: ném exception (`NoSuchBeanDefinitionException`).
   - **Tìm thấy >1 Bean**: ném exception (`NoUniqueBeanDefinitionException`).

`@Resource` có hai property quan trọng và thường dùng trong phát triển hằng ngày: `name` (name) và `type` (type).

```java
public @interface Resource {
    String name() default "";
    Class<?> type() default Object.class;
}
```

Nếu chỉ chỉ định property `name` thì cách inject là `byName`; nếu chỉ chỉ định property `type` thì cách inject là `byType`; nếu đồng thời chỉ định `name` và `type` (không khuyến nghị) thì cách inject là `byType` + `byName`.

```java
// Lỗi, cả byName và byType đều không match được bean
@Resource
private SmsService smsService;
// Inject đúng bean tương ứng với object SmsServiceImpl1
@Resource
private SmsService smsServiceImpl1;
// Inject đúng bean tương ứng với object SmsServiceImpl1 (khuyến nghị hơn)
@Resource(name = "smsServiceImpl1")
private SmsService smsService;
```

**Tóm tắt:**

- `@Autowired` là annotation do Spring cung cấp, `@Resource` là annotation do Jakarta Annotations/JSR-250 specification cung cấp.
- `Autowired` mặc định inject theo `byType` (match theo type), `@Resource` mặc định inject theo `byName` (match theo name).
- Khi một interface có nhiều implementation, `@Autowired` và `@Resource` đều cần name để match đúng Bean tương ứng. `Autowired` có thể dùng annotation `@Qualifier` để chỉ định tường minh name, còn `@Resource` có thể dùng property `name` để chỉ định tường minh name.
- `@Autowired` hỗ trợ dùng trên constructor, method, field và parameter. `@Resource` chủ yếu dùng để inject trên field và method, không hỗ trợ constructor hoặc parameter.

Xét việc ngữ nghĩa của `@Resource` rõ ràng hơn (ưu tiên name), đồng thời đây là Java standard và giúp giảm coupling với Spring Framework, chúng tôi thường **khuyến nghị dùng `@Resource`** hơn, đặc biệt trong trường hợp cần inject theo name. Tuy nhiên, `@Autowired` kết hợp constructor injection có ưu thế về tính immutable và bắt buộc của dependency injection, cũng là một practice rất tốt.

### Có những cách nào để inject Bean?

Các cách phổ biến của dependency injection (Dependency Injection, DI):

1. Constructor injection: inject dependency thông qua constructor của class.
1. Setter injection: inject dependency thông qua Setter method của class.
1. Field injection: trực tiếp dùng annotation (chẳng hạn `@Autowired` hoặc `@Resource`) trên field của class để inject dependency.

Ví dụ constructor injection:

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    //...
}
```

Ví dụ Setter injection:

```java
@Service
public class UserService {

    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    //...
}
```

Ví dụ Field injection:

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    //...
}
```

### ⭐️Constructor injection hay Setter injection?

Spring có câu trả lời chính thức cho vấn đề này: <https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html#beans-setter-injection>.

Ở đây tôi chủ yếu trích lọc và hoàn thiện đề xuất của Spring.

**Spring chính thức khuyến nghị constructor injection**, ưu điểm của cách inject này như sau:

1. Tính đầy đủ của dependency: bảo đảm mọi dependency bắt buộc được inject ngay khi object được tạo, tránh rủi ro NullPointerException.
2. Tính bất biến: giúp tạo immutable object, nâng cao thread safety.
3. Bảo đảm khởi tạo: component đã được khởi tạo hoàn chỉnh trước khi sử dụng, giảm lỗi tiềm ẩn.
4. Thuận tiện cho test: trong unit test, có thể trực tiếp truyền dependency giả qua constructor mà không cần dựa vào Spring container để inject.

Constructor injection phù hợp với **dependency bắt buộc**, còn **Setter injection** phù hợp hơn với **dependency tùy chọn**, những dependency có thể có default value hoặc được thiết lập động trong vòng đời object. Dù `@Autowired` có thể dùng cho Setter method để xử lý dependency bắt buộc, constructor injection vẫn là lựa chọn tốt hơn.

Trong một số trường hợp (chẳng hạn third-party class không cung cấp Setter method), constructor injection có thể là **lựa chọn duy nhất**.

### ⭐️Các scope của Bean là gì?

Trong Spring thường có các scope của Bean sau:

- **singleton**: Chỉ có một Bean instance duy nhất trong IoC container. Bean trong Spring mặc định đều là singleton, đây là ứng dụng của singleton design pattern.
- **prototype**: Mỗi lần lấy sẽ tạo một Bean instance mới. Nghĩa là gọi `getBean()` hai lần liên tiếp sẽ nhận được hai Bean instance khác nhau.
- **request** (chỉ dùng được trong Web application): Mỗi HTTP request sẽ tạo một Bean mới (request Bean), Bean đó chỉ có hiệu lực trong HTTP request hiện tại.
- **session** (chỉ dùng được trong Web application): Mỗi HTTP request đến từ session mới sẽ tạo một Bean mới (session Bean), Bean đó chỉ có hiệu lực trong HTTP session hiện tại.
- **application/global-session** (chỉ dùng được trong Web application): Mỗi Web application tạo một Bean khi khởi động (application Bean), Bean đó chỉ có hiệu lực trong thời gian application hiện tại chạy.
- **websocket** (chỉ dùng được trong Web application): Mỗi WebSocket session tạo một Bean mới.

**Cấu hình scope của Bean như thế nào?**

Cách xml:

```xml
<bean id="..." class="..." scope="singleton"></bean>
```

Cách annotation:

```java
@Bean
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public Person personPrototype() {
    return new Person();
}
```

### ⭐️Bean có thread-safe không?

Bean trong Spring Framework có thread-safe hay không phụ thuộc vào scope và state của nó.

Ở đây lấy hai scope thường dùng nhất là prototype và singleton làm ví dụ. Hầu hết các trường hợp đều dùng scope mặc định singleton, vì vậy cần tập trung vào scope singleton.

Trong scope prototype, mỗi lần lấy từ container sẽ tạo một Bean instance mới, có thể giảm khả năng chia sẻ ở tầng container, nhưng bản thân scope không bảo đảm thread safety: nếu caller chia sẻ cùng một prototype instance cho nhiều thread thì vẫn có thể xảy ra resource contention. Trong scope singleton, IoC container chỉ có một Bean instance duy nhất, nên dễ xảy ra tranh chấp shared state hơn (phụ thuộc Bean có state hay không).

Ví dụ stateful Bean:

```java
// Định nghĩa một class giỏ hàng, chứa List lưu các sản phẩm trong giỏ hàng của user
@Component
public class ShoppingCart {
    private List<String> items = new ArrayList<>();

    public void addItem(String item) {
        items.add(item);
    }

    public List<String> getItems() {
        return items;
    }
}
```

Tuy nhiên, phần lớn Bean trên thực tế là stateless (không định nghĩa member variable có thể thay đổi) (chẳng hạn Dao, Service), trong trường hợp này Bean là thread-safe.

Ví dụ stateless Bean:

```java
// Định nghĩa một user service chỉ chứa business logic mà không lưu state nào.
@Component
public class UserService {

    public User findUserById(Long id) {
        //...
    }
    //...
}
```

Có ba cách phổ biến để giải quyết vấn đề thread safety của stateful singleton Bean:

1. **Tránh member variable có thể thay đổi**: cố gắng thiết kế Bean thành stateless.
2. **Dùng `ThreadLocal`**: lưu member variable có thể thay đổi trong `ThreadLocal`, bảo đảm độc lập giữa các thread.
3. **Dùng cơ chế synchronization**: sử dụng `synchronized` hoặc `ReentrantLock` để đồng bộ, bảo đảm thread safety.

Lấy `ThreadLocal` làm ví dụ để minh họa tình huống `ThreadLocal` lưu thông tin đăng nhập của user:

```java
public class UserThreadLocal {

    private UserThreadLocal() {}

    private static final ThreadLocal<SysUser> LOCAL = ThreadLocal.withInitial(() -> null);

    public static void put(SysUser sysUser) {
        LOCAL.set(sysUser);
    }

    public static SysUser get() {
        return LOCAL.get();
    }

    public static void remove() {
        LOCAL.remove();
    }
}
```

### ⭐️Bạn có biết vòng đời của Bean không?

1. **Tạo Bean instance**: Bean container trước hết tìm Bean definition trong file cấu hình, sau đó chọn strategy khởi tạo phù hợp (factory method, constructor autowiring hoặc khởi tạo đơn giản), rồi dùng Java reflection API để tạo Bean instance.
2. **Gán/fill thuộc tính Bean**: Thiết lập các property và dependency liên quan cho Bean, chẳng hạn xử lý các annotation `@Autowired`, `@Value`, `@Resource` được đánh dấu trên field hoặc Setter method.
3. **Khởi tạo Bean**:
   - Nếu Bean implement interface `BeanNameAware`, gọi method `setBeanName()`, truyền vào tên của Bean.
   - Nếu Bean implement interface `BeanClassLoaderAware`, gọi `setBeanClassLoader()`, truyền vào instance của `ClassLoader`.
   - Nếu Bean implement interface `BeanFactoryAware`, gọi `setBeanFactory()`, truyền vào instance của `BeanFactory`.
   - Tương tự, nếu implement các interface `*.Aware` khác thì gọi method tương ứng.
   - Nếu có object `BeanPostProcessor` liên quan đến Spring container đã load Bean này, thực thi method `postProcessBeforeInitialization()`.
   - Nếu Bean implement interface `InitializingBean`, thực thi method `afterPropertiesSet()`.
   - Nếu Bean definition trong file cấu hình có property `init-method`, thực thi method được chỉ định.
   - Nếu có object `BeanPostProcessor` liên quan đến Spring container đã load Bean này, thực thi method `postProcessAfterInitialization()`.
4. **Hủy Bean**: Hủy không có nghĩa là lập tức xóa Bean, mà là ghi lại method hủy của Bean trước, sau đó khi cần hủy Bean hoặc container thì gọi các method này để giải phóng resource mà Bean đang giữ.
   - Nếu Bean implement interface `DisposableBean`, thực thi method `destroy()`.
   - Nếu Bean definition trong file cấu hình có property `destroy-method`, thực thi method hủy Bean được chỉ định. Hoặc có thể trực tiếp dùng annotation `@PreDestroy` để đánh dấu method cần thực thi trước khi hủy Bean.

Trong method `doCreateBean()` của `AbstractAutowireCapableBeanFactory` có thể thấy bốn giai đoạn này được thực thi lần lượt:

```java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
    throws BeanCreationException {

    // 1. Tạo Bean instance
    BeanWrapper instanceWrapper = null;
    if (instanceWrapper == null) {
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }

    Object exposedObject = bean;
    try {
        // 2. Gán/fill thuộc tính Bean
        populateBean(beanName, mbd, instanceWrapper);
        // 3. Khởi tạo Bean
        exposedObject = initializeBean(beanName, exposedObject, mbd);
    }

    // 4. Hủy Bean - đăng ký callback interface
    try {
        registerDisposableBeanIfNecessary(beanName, bean, mbd);
    }

    return exposedObject;
}
```

Interface `Aware` cho phép Bean lấy được resource của Spring container.

Các interface `Aware` Spring cung cấp chủ yếu gồm:

1. `BeanNameAware`: inject beanName tương ứng với Bean hiện tại;
2. `BeanClassLoaderAware`: inject ClassLoader dùng để load Bean hiện tại;
3. `BeanFactoryAware`: inject reference đến `BeanFactory` container hiện tại.

Interface `BeanPostProcessor` là extension point mạnh mẽ Spring cung cấp để sửa đổi Bean.

```java
public interface BeanPostProcessor {

	// Xử lý trước khởi tạo
	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

	// Xử lý sau khởi tạo
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

}
```

- `postProcessBeforeInitialization`: thực thi sau khi Bean instance được khởi tạo và property được inject, nhưng trước method `InitializingBean#afterPropertiesSet` và method `init-method` tùy chỉnh;
- `postProcessAfterInitialization`: tương tự phần trên, nhưng thực thi sau method `InitializingBean#afterPropertiesSet` và method `init-method` tùy chỉnh.

`InitializingBean` và `init-method` là extension point Spring cung cấp cho việc khởi tạo Bean.

```java
public interface InitializingBean {
  // Logic khởi tạo
	void afterPropertiesSet() throws Exception;
}
```

Chỉ định method `init-method`, tức chỉ định method khởi tạo:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="demo" class="com.chaycao.Demo" init-method="init"/>

</beans>
```

**Ghi nhớ như thế nào?**

1. Nhìn tổng thể có thể chia đơn giản thành bốn bước: khởi tạo instance —> gán property —> khởi tạo —> hủy.
2. Bước khởi tạo bao gồm khá nhiều bước nhỏ, gồm dependency injection của interface `Aware`, xử lý của `BeanPostProcessor` trước và sau khởi tạo, cùng thao tác khởi tạo của `InitializingBean` và `init-method`.
3. Bước hủy sẽ đăng ký các callback interface liên quan, cuối cùng hủy thông qua `DisposableBean` và `destory-method`.

Cuối cùng, chia sẻ thêm một sơ đồ minh họa rõ ràng ([Cách ghi nhớ vòng đời Spring Bean](https://chaycao.github.io/2020/02/15/如何记忆Spring-Bean的生命周期.html)).

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/spring-bean-lifestyle.png)

## Spring AOP

### Hãy nói về hiểu biết của bạn đối với AOP

AOP (Aspect-Oriented Programming: lập trình hướng khía cạnh) có thể đóng gói các logic hoặc trách nhiệm không liên quan đến business nhưng được nhiều business module cùng gọi (chẳng hạn transaction handling, log management, permission control), giúp giảm code lặp trong hệ thống, giảm coupling giữa các module và có lợi cho khả năng mở rộng, bảo trì sau này.

Spring AOP dựa trên dynamic proxy. Nếu object cần proxy implement một interface nào đó, Spring AOP sẽ dùng **JDK Proxy** để tạo proxy object; còn với object không implement interface, không thể dùng JDK Proxy để proxy, lúc này Spring AOP sẽ dùng **Cglib** tạo subclass của object được proxy làm proxy, như hình dưới đây:

![SpringAOPProcess](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/230ae587a322d6e4d09510161987d346.jpeg)

Đương nhiên bạn cũng có thể dùng **AspectJ**! Spring AOP đã tích hợp AspectJ, AspectJ có thể xem là AOP framework hoàn chỉnh nhất trong hệ sinh thái Java.

Một số thuật ngữ chuyên ngành trong AOP:

| Thuật ngữ     |                                                      Ý nghĩa                                                      |
| :------------ | :---------------------------------------------------------------------------------------------------------------: |
| Target        |                                                Object nhận advice                                                 |
| Proxy         |                          Proxy object được tạo sau khi áp dụng advice vào target object                           |
| JoinPoint     |                   Tất cả method được định nghĩa trong class của target object đều là join point                   |
| Pointcut      | Join point bị aspect intercept/enhance (pointcut nhất định là join point, nhưng join point chưa chắc là pointcut) |
| Advice        |             Logic/code được enhance, tức việc cần làm sau khi intercept join point của target object              |
| Aspect        |                                                 Pointcut + Advice                                                 |
| Weaving (dệt) |                        Quá trình áp dụng advice vào target object, từ đó tạo proxy object                         |

### Sự khác nhau giữa Spring AOP và AspectJ AOP là gì?

| Đặc tính             | Spring AOP                                                                                   | AspectJ                                                           |
| -------------------- | -------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| **Cách enhance**     | Enhance tại runtime (dựa trên dynamic proxy)                                                 | Enhance lúc compile, lúc class load (trực tiếp thao tác bytecode) |
| **Hỗ trợ pointcut**  | Theo method (trong phạm vi Spring Bean, không hỗ trợ method final và staic)                  | Theo method, field, constructor, static method, v.v.              |
| **Performance**      | Phụ thuộc proxy tại runtime, có overhead nhất định, performance thấp hơn khi có nhiều aspect | Không có overhead proxy lúc runtime, performance cao hơn          |
| **Độ phức tạp**      | Đơn giản, dễ dùng, phù hợp với phần lớn scenario                                             | Tính năng mạnh, nhưng tương đối phức tạp                          |
| **Scenario sử dụng** | Nhu cầu AOP tương đối đơn giản trong Spring application                                      | Nhu cầu AOP hiệu năng cao, độ phức tạp cao                        |

**Chọn như thế nào?**

- **Xét về chức năng**: AspectJ hỗ trợ scenario AOP phức tạp hơn, Spring AOP đơn giản và dễ dùng hơn. Nếu cần enhance `final` method, static method, field access, constructor call, hoặc cần áp dụng logic enhance trên object không do Spring quản lý, AspectJ là lựa chọn duy nhất.
- **Xét về performance**: Khi số lượng aspect ít, chênh lệch performance giữa hai bên không lớn; khi có nhiều aspect, AspectJ có performance tốt hơn.

**Tóm tắt trong một câu**: Scenario đơn giản ưu tiên Spring AOP; scenario phức tạp hoặc có yêu cầu performance cao thì chọn AspectJ.

### Những loại advice thường gặp của AOP là gì?

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/aspectj-advice-types.jpg)

- **Before** (before advice): kích hoạt trước khi gọi method của target object
- **After** (after advice): kích hoạt sau khi gọi method của target object
- **AfterReturning** (returning advice): kích hoạt sau khi method của target object hoàn tất và trả về result
- **AfterThrowing** (exception advice): kích hoạt sau khi method của target object ném/phát sinh exception trong quá trình chạy. AfterReturning và AfterThrowing loại trừ lẫn nhau. Nếu method gọi thành công không có exception thì có return value; nếu method ném exception thì không có return value.
- **Around** (around advice): điều khiển việc gọi method của target object bằng code. Around advice có phạm vi thao tác lớn nhất trong tất cả loại advice, vì có thể trực tiếp lấy target object và method cần thực thi, nên có thể tùy ý thực hiện việc trước và sau khi gọi method của target object, thậm chí không gọi method của target object.

### Kiểm soát thứ tự thực thi của nhiều aspect như thế nào?

1. Thông thường dùng annotation `@Order` để trực tiếp định nghĩa thứ tự aspect.

```java
// Giá trị càng nhỏ thì priority càng cao
@Order(3)
@Component
@Aspect
public class LoggingAspect implements Ordered {
```

**2. Implement interface `Ordered` và override method `getOrder`.**

```java
@Component
@Aspect
public class LoggingAspect implements Ordered {

    // ....

    @Override
    public int getOrder() {
        // Giá trị trả về càng nhỏ thì priority càng cao
        return 1;
    }
}
```

## Spring MVC

### Hãy nói về hiểu biết của bạn đối với Spring MVC

MVC là viết tắt của Model, View và Controller, tư tưởng cốt lõi là tổ chức code bằng cách tách business logic, data và display.

![](https://oss.javaguide.cn/java-guide-blog/image-20210809181452421.png)

Trên Internet có nhiều người nói MVC không phải design pattern mà chỉ là software design specification, còn tôi thiên về quan điểm MVC cũng là một trong nhiều design pattern. Trong project **[java-design-patterns](https://github.com/iluwatar/java-design-patterns)** có phần giới thiệu liên quan đến MVC.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/159b3d3e70dd45e6afa81bf06d09264e.png)

Muốn thực sự hiểu Spring MVC, trước hết hãy xem thời kỳ Model 1 và Model 2, khi chưa có Spring MVC.

**Thời kỳ Model 1**

Nhiều bạn học Java backend muộn có thể chưa từng tiếp xúc với phát triển JavaWeb application trong thời kỳ Model 1. Ở chế độ Model1, gần như toàn bộ Web application được tạo từ các JSP page, chỉ dùng một số ít JavaBean để xử lý database connection, access và các thao tác khác.

Trong chế độ này, JSP vừa là control layer (Controller), vừa là presentation layer (View). Rõ ràng chế độ này có nhiều vấn đề. Chẳng hạn control logic và presentation logic trộn lẫn, khiến khả năng tái sử dụng code cực thấp; hoặc frontend và backend phụ thuộc lẫn nhau, khó test, bảo trì và hiệu quả phát triển rất thấp.

![mvc-mode1](https://oss.javaguide.cn/java-guide-blog/mvc-mode1.png)

**Thời kỳ Model 2**

Các bạn từng học Servlet và làm Demo liên quan hẳn biết mô hình phát triển “Java Bean(Model) + JSP (View) + Servlet (Controller)”, đây là mô hình phát triển JavaWeb MVC thời kỳ đầu.

- Model: data liên quan đến hệ thống, tức dao và bean.
- View: hiển thị data trong model, chỉ dùng để hiển thị.
- Controller: nhận request của user, gửi request đến Model, cuối cùng trả data cho JSP và hiển thị cho user.

![](https://oss.javaguide.cn/java-guide-blog/mvc-model2.png)

Chế độ Model2 vẫn còn nhiều vấn đề, mức độ abstraction và encapsulation của Model2 chưa đủ, khi phát triển bằng Model2 không thể tránh khỏi việc phát minh lại bánh xe, làm giảm đáng kể khả năng bảo trì và tái sử dụng của chương trình.

Vì vậy, nhiều MVC framework liên quan đến phát triển JavaWeb lần lượt ra đời, chẳng hạn Struts2, nhưng Struts2 khá nặng.

**Thời kỳ Spring MVC**

Cùng với sự phổ biến của Spring, một development framework nhẹ, Spring MVC framework xuất hiện trong hệ sinh thái Spring. Spring MVC là MVC framework xuất sắc nhất hiện nay. So với Struts2, Spring MVC đơn giản và thuận tiện hơn, hiệu quả phát triển cao hơn, đồng thời tốc độ chạy cũng nhanh hơn.

MVC là một design pattern, còn Spring MVC là một MVC framework rất tốt. Spring MVC giúp phát triển Web layer ngắn gọn hơn và được tích hợp tự nhiên với Spring Framework. Trong Spring MVC, backend project thường được chia thành Service layer (xử lý business), Dao layer (database operation), Entity layer (entity class) và Controller layer (control layer, trả data cho frontend page).

### Các core component của Spring MVC là gì?

Ghi nhớ các component dưới đây cũng là ghi nhớ nguyên lý hoạt động của SpringMVC.

- **`DispatcherServlet`**: **central processor cốt lõi**, chịu trách nhiệm nhận request, phân phối và trả response cho client.
- **`HandlerMapping`**: **handler mapper**, match và tìm `Handler` có thể xử lý dựa trên URL, đồng thời đóng gói interceptor liên quan đến request cùng với `Handler`.
- **`HandlerAdapter`**: **handler adapter**, dựa trên `Handler` do `HandlerMapping` tìm được để adapter và thực thi `Handler` tương ứng;
- **`Handler`**: **request handler**, xử lý request thực tế.
- **`ViewResolver`**: **view resolver**, dựa trên logical view/view do `Handler` trả về để resolve và render view thực tế, sau đó truyền cho `DispatcherServlet` response client.

### ⭐️Bạn có biết nguyên lý hoạt động của SpringMVC không?

**Nguyên lý Spring MVC như hình dưới đây:**

> Tôi không tự vẽ sơ đồ nguyên lý hoạt động của SpringMVC mà trực tiếp tìm trên Internet cho nhanh một sơ đồ rất rõ ràng, không rõ nguồn gốc ban đầu.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/de6d2b213f112297298f3e223bf08f28.png)

**Mô tả flow (quan trọng):**

1. Client (browser) gửi request, `DispatcherServlet` intercept request.
2. `DispatcherServlet` gọi `HandlerMapping` dựa trên thông tin request. `HandlerMapping` match và tìm `Handler` có thể xử lý dựa trên URL (chính là `Controller` mà chúng ta thường nói), đồng thời đóng gói interceptor liên quan đến request cùng với `Handler`.
3. `DispatcherServlet` gọi adapter `HandlerAdapter` để thực thi `Handler`.
4. Sau khi `Handler` xử lý xong request của user, trả một object `ModelAndView` cho `DispatcherServlet`. Như tên gọi, `ModelAndView` chứa model data và thông tin view tương ứng. `Model` là data object trả về, `View` là một `View` ở mức logic.
5. `ViewResolver` tìm `View` thực tế dựa trên logical `View`.
6. `DispaterServlet` truyền `Model` trả về cho `View` (render view).
7. Trả `View` cho requestor (browser).

Flow trên là nguyên lý hoạt động của mô hình phát triển truyền thống (JSP, Thymeleaf, v.v.). Tuy nhiên, cách phát triển chủ đạo hiện nay là tách frontend và backend. Khi đó khái niệm `View` của Spring MVC có một số thay đổi. Vì `View` thường do frontend framework (Vue, React, v.v.) xử lý, backend không còn chịu trách nhiệm render page mà chỉ cung cấp data, do đó:

- Khi tách frontend và backend, backend thường không trả về view cụ thể mà trả về **pure data** (thường ở dạng JSON), frontend chịu trách nhiệm render và hiển thị.
- Phần `View` thường không cần thiết lập trong scenario tách frontend và backend. Controller method của Spring MVC chỉ cần trả data, không còn trả `ModelAndView`, mà trả trực tiếp data, Spring sẽ tự động chuyển thành JSON. Tương ứng, `ViewResolver` cũng không còn được sử dụng.

Thực hiện như thế nào?

- Dùng annotation `@RestController` thay cho annotation `@Controller` truyền thống, như vậy mọi method mặc định trả data dạng JSON thay vì cố gắng resolve view.
- Nếu dùng `@Controller`, có thể kết hợp annotation `@ResponseBody` để trả JSON.

### Xử lý exception tập trung như thế nào?

Khuyến nghị dùng annotation để xử lý exception tập trung, cụ thể sẽ dùng hai annotation `@ControllerAdvice` + `@ExceptionHandler`.

```java
@ControllerAdvice
@ResponseBody
public class GlobalExceptionHandler {

    @ExceptionHandler(BaseException.class)
    public ResponseEntity<?> handleAppException(BaseException ex, HttpServletRequest request) {
      //......
    }

    @ExceptionHandler(value = ResourceNotFoundException.class)
    public ResponseEntity<ErrorReponse> handleResourceNotFoundException(ResourceNotFoundException ex, HttpServletRequest request) {
      //......
    }
}
```

Cách xử lý exception này không tạo AOP proxy cho `Controller`. Sau khi `Controller` method ném exception, Spring MVC tìm method `@ExceptionHandler` có thể xử lý exception thông qua chuỗi xử lý `HandlerExceptionResolver`.

Trong `ExceptionHandlerMethodResolver`, method `getMappedMethod` quyết định exception cụ thể được method nào gắn annotation `@ExceptionHandler` xử lý.

```java
@Nullable
  private Method getMappedMethod(Class<? extends Throwable> exceptionType) {
    List<Class<? extends Throwable>> matches = new ArrayList<>();
    // Tìm mọi thông tin exception có thể xử lý. mappedMethods lưu mapping giữa exception và method xử lý exception
    for (Class<? extends Throwable> mappedException : this.mappedMethods.keySet()) {
      if (mappedException.isAssignableFrom(exceptionType)) {
        matches.add(mappedException);
      }
    }
    // Không rỗng nghĩa là có method xử lý exception
    if (!matches.isEmpty()) {
      // Sort theo mức độ match từ nhỏ đến lớn
      matches.sort(new ExceptionDepthComparator(exceptionType));
      // Trả về method xử lý exception
      return this.mappedMethods.get(matches.get(0));
    }
    else {
      return null;
    }
  }
```

Từ source code có thể thấy: **`getMappedMethod()` trước hết tìm mọi method có thể match để xử lý exception, sau đó sort từ nhỏ đến lớn và cuối cùng lấy method match nhỏ nhất (tức có độ match cao nhất).**

## Spring Framework đã sử dụng những design pattern nào?

> Để xem giới thiệu chi tiết về các design pattern dưới đây, bạn có thể đọc bài [Giải thích chi tiết các design pattern trong Spring](https://javaguide.cn/system-design/framework/spring/spring-design-patterns-summary.html) do tôi viết.

- **Factory design pattern**: Spring dùng factory pattern để tạo bean object thông qua `BeanFactory`, `ApplicationContext`.
- **Proxy design pattern**: Cách triển khai chức năng Spring AOP.
- **Singleton design pattern**: Bean trong Spring mặc định đều là singleton.
- **Template method pattern**: Các class thao tác database có tên kết thúc bằng Template như `jdbcTemplate`, `hibernateTemplate` sử dụng template pattern.
- **Wrapper design pattern**: Project cần kết nối nhiều database, và mỗi client có thể truy cập database khác nhau tùy nhu cầu trong từng request. Pattern này cho phép dynamic switch giữa các data source theo nhu cầu của client.
- **Observer pattern**: Spring event-driven model là một ứng dụng kinh điển của observer pattern.
- **Adapter pattern**: Enhance hoặc advice (Advice) của Spring AOP sử dụng adapter pattern; Spring MVC cũng dùng adapter pattern để adapter `Controller`.
- ……

## ⭐️Circular dependency trong Spring

### Bạn có biết circular dependency trong Spring không, giải quyết như thế nào?

Circular dependency là việc các Bean object tham chiếu vòng tròn, tức hai hoặc nhiều Bean cùng giữ reference đến nhau, chẳng hạn CircularDependencyA → CircularDependencyB → CircularDependencyA.

```java
@Component
public class CircularDependencyA {
    @Autowired
    private CircularDependencyB circB;
}

@Component
public class CircularDependencyB {
    @Autowired
    private CircularDependencyA circA;
}
```

Một object tự phụ thuộc cũng có thể tạo circular dependency, nhưng xác suất rất thấp và thuộc lỗi khi viết code.

```java
@Component
public class CircularDependencyA {
    @Autowired
    private CircularDependencyA circA;
}
```

Spring Framework có thể dùng three-level cache để giải quyết circular dependency của một phần singleton Bean khi inject Setter/field. Circular dependency qua constructor, circular dependency của prototype Bean và các scenario khác không thể dựa vào cơ chế này để giải quyết.

Three-level cache trong Spring thực ra là ba Map:

```java
// Level-one cache
/** Cache of singleton objects: bean name to bean instance. */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

// Level-two cache
/** Cache of early singleton objects: bean name to bean instance. */
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);

// Level-three cache
/** Cache of singleton factories: bean name to ObjectFactory. */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
```

Nói đơn giản, three-level cache của Spring gồm:

1. **Level-one cache (`singletonObjects`)**: Lưu Bean ở trạng thái hoàn chỉnh (đã khởi tạo, fill property và initialization), singleton pool, tồn tại vì “thuộc tính singleton của Spring”. Thông thường chúng ta lấy Bean từ đây, nhưng không phải mọi Bean đều ở singleton pool, chẳng hạn prototype Bean không nằm trong đó.
2. **Level-two cache (`earlySingletonObjects`)**: Lưu Bean đang chuyển tiếp (bán thành phẩm, chưa fill property), tức object do `ObjectFactory` trong level-three cache tạo ra; được dùng cùng level-three cache để tránh mỗi lần gọi `ObjectFactory#getObject()` trong trường hợp AOP lại tạo một proxy object mới.
3. **Level-three cache (`singletonFactories`)**: Lưu `ObjectFactory`. Method `getObject()` của `ObjectFactory` (cuối cùng gọi `getEarlyBeanReference()`) có thể tạo raw Bean object hoặc proxy object (nếu Bean được AOP aspect proxy). Level-three cache chỉ có hiệu lực với singleton Bean.

Tiếp theo là flow Spring tạo Bean:

1. Trước hết tìm trong **level-one cache `singletonObjects`**, nếu tồn tại thì trả về;
2. Nếu không tồn tại hoặc object đang được tạo, tìm trong **level-two cache `earlySingletonObjects`**;
3. Nếu vẫn chưa lấy được, tìm trong **level-three cache `singletonFactories`**, thực thi `getObject()` của `ObjectFacotry` để lấy object. Sau khi lấy thành công, xóa object khỏi level-three cache và thêm vào level-two cache.

Object được lưu trong level-three cache là `ObjectFacoty`:

```java
public interface ObjectFactory<T> {
    T getObject() throws BeansException;
}
```

Khi tạo Bean, nếu cho phép circular dependency, Spring sẽ expose trước Bean object vừa khởi tạo xong nhưng chưa hoàn tất initialization. Việc này được thực hiện bằng method `addSingletonFactory`, thêm một object `ObjectFactory` vào level-three cache:

```java
// AbstractAutowireCapableBeanFactory # doCreateBean #
public abstract class AbstractAutowireCapableBeanFactory ... {
	protected Object doCreateBean(...) {
        //...

        // Hỗ trợ circular dependency: thêm ()->getEarlyBeanReference làm method getObject() của ObjectFactory vào level-three cache
		addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
    }
}
```

Như đã nói trong flow tạo Bean của Spring, nếu không lấy được object từ level-one và level-two cache thì sẽ lấy object từ level-three cache thông qua method `getObject` của `ObjectFactory`.

```java
class A {
    // Sử dụng B
    private B b;
}
class B {
    // Sử dụng A
    private A a;
}
```

Lấy code circular dependency trên làm ví dụ, flow giải quyết circular dependency hoàn chỉnh như sau:

- Khi Spring tạo A xong, phát hiện A phụ thuộc B, nên tiếp tục tạo B; B phụ thuộc A, nên lại tiếp tục tạo A;
- Khi B tạo A, lúc này A phát sinh circular dependency. Vì A chưa hoàn tất initialization nên chắc chắn chưa có A trong **level-one và level-two cache**;
- Khi đó tìm trong level-three cache và gọi method `getObject()` để lấy **object đã expose trước của A**, tức gọi method `getEarlyBeanReference()` được thêm ở trên để tạo **object đã expose trước của A**;
- Sau đó xóa `ObjectFactory` khỏi level-three cache và đưa object đã expose trước vào level-two cache, rồi inject object đã expose trước này vào dependency của B để hỗ trợ circular dependency.

**Chỉ dùng two-level cache có đủ không?** Khi không có AOP, đúng là chỉ cần level-one và level-two cache cũng có thể giải quyết circular dependency. Nhưng khi có AOP, level-three cache rất quan trọng, vì nó bảo đảm dù trong quá trình tạo Bean có nhiều request đến early reference thì vẫn luôn trả về cùng một proxy object, tránh một Bean có nhiều proxy object.

**Tóm tắt cách Spring giải quyết bằng three-level cache:**

Ở phần three-level cache, chủ yếu cần ghi nhớ cách Spring hỗ trợ circular dependency: khi xảy ra circular dependency, lấy `ObjectFactory` được lưu trong **level-three cache `singletonFactories`** và gọi method `getObject()` để lấy object đã expose trước của circular dependency (dù chưa hoàn tất initialization nhưng đã lấy được địa chỉ lưu trữ của object trong heap), sau đó đưa object đã expose trước vào level-two cache. Như vậy khi circular dependency xảy ra sẽ không khởi tạo lặp lại!

### @Lazy có giải quyết được circular dependency không?

`@Lazy` dùng để đánh dấu class có cần lazy load/delayed load hay không, có thể áp dụng trên class, method, constructor, method parameter và member variable.

Spring Boot 2.2 bổ sung **global lazy-loading property**. Sau khi bật, toàn bộ Bean sẽ được đặt thành lazy load và chỉ được tạo khi cần.

Cấu hình global lazy-loading trong file cấu hình:

```properties
# Mặc định là false
spring.main.lazy-initialization=true
```

Cấu hình global lazy-loading bằng code:

```java
SpringApplication springApplication=new SpringApplication(Start.class);
springApplication.setLazyInitialization(true);
springApplication.run(args);
```

Nếu không cần thiết thì cố gắng không dùng global lazy-loading. Global lazy-loading khiến lần đầu Bean được sử dụng sẽ load chậm hơn, đồng thời trì hoãn việc phát hiện vấn đề của application (chỉ xuất hiện khi Bean được khởi tạo).

Nếu Bean không được đánh dấu lazy-loading thì sẽ được tạo và khởi tạo trong quá trình Spring IoC container khởi động. Nếu Bean được đánh dấu lazy-loading thì không lập tức instantiate khi Spring IoC container khởi động, mà chỉ tạo khi lần đầu được request. Điều này giúp giảm thời gian initialization khi application khởi động và cũng có thể dùng để giải quyết circular dependency.

Circular dependency được giải quyết bằng `@Lazy` như thế nào? Ví dụ có hai Bean A và B circular dependency với nhau, có thể thêm `@Lazy` tại injection point B trong A, chẳng hạn constructor parameter `A(@Lazy B b)`. Khi đó thứ được resolve lazy là dependency B, không phải chỉ đơn giản đặt `@Lazy` trên constructor hoặc type của A.

- Trước hết Spring tạo Bean A, khi tạo cần inject property B;
- Vì injection point B trong A được đánh dấu `@Lazy`, Spring tạo một proxy delayed-resolution của B và inject proxy vào A;
- Sau đó bắt đầu instantiate và initialize B. Khi inject property A vào B, lúc này A đã tạo xong nên có thể inject A vào.

Từ flow load trên có thể thấy: điểm mấu chốt để `@Lazy` giải quyết circular dependency là sử dụng proxy object.

- **Không có `@Lazy`**: Khi Spring initialize A trong container, lập tức thử tạo B; trong quá trình tạo B lại thử tạo A, cuối cùng dẫn đến circular dependency (tức đệ quy vô hạn và cuối cùng ném exception).
- **Có `@Lazy`**: Spring không lập tức tạo B mà inject một proxy object của B. Vì lúc này B chưa thực sự initialize nên A có thể initialize thuận lợi. Đến khi instance A thực sự gọi method của B, proxy object mới trigger việc initialize B thật sự.

Proxy tại injection point của `@Lazy` có thể phá vỡ circular dependency chain ở một mức độ nhất định, bao gồm một số scenario constructor injection. Tuy nhiên, đây không phải là loại bỏ circular dependency từ thiết kế; với dependency relationship phức tạp cũng có thể phát sinh vấn đề initialization khó nhận ra hơn. Vì vậy best practice vẫn là cố gắng tránh circular dependency trong thiết kế.

### SpringBoot có cho phép circular dependency xảy ra không?

Trước SpringBoot 2.6.x, circular dependency được cho phép mặc định, nghĩa là code xuất hiện circular dependency thì thông thường cũng không báo lỗi. Từ SpringBoot 2.6.x, Spring chính thức không còn khuyến nghị viết code có circular dependency và đề nghị developer giảm các dependency lẫn nhau không cần thiết khi viết code. Đây cũng là việc chúng ta nên làm nhất: bản thân circular dependency là một khiếm khuyết thiết kế, không nên phụ thuộc quá mức vào Spring mà bỏ qua quy chuẩn và chất lượng code; biết đâu một phiên bản SpringBoot nào đó trong tương lai sẽ cấm hoàn toàn code có circular dependency.

Sau SpringBoot 2.6.x, nếu không muốn refactor code circular dependency thì có thể dùng các cách sau:

- Thiết lập cho phép circular dependency trong global configuration file: `spring.main.allow-circular-references=true`. Cách đơn giản và thô, không khuyến nghị lắm.
- Thêm annotation `@Lazy` vào Bean gây circular dependency, đây là cách tương đối được khuyến nghị. `@Lazy` dùng để đánh dấu class có cần lazy load/delayed load hay không, có thể áp dụng trên class, method, constructor, method parameter và member variable.
- ……

## ⭐️Spring Transaction

Để xem giới thiệu chi tiết về Spring transaction, bạn có thể đọc bài [Giải thích chi tiết Spring transaction](https://javaguide.cn/system-design/framework/spring/spring-transaction.html) do tôi viết.

### Spring quản lý transaction bằng những cách nào?

- **Programmatic transaction**: hard-code trong code (khuyến nghị dùng trong distributed system): quản lý transaction thủ công thông qua `TransactionTemplate` hoặc `TransactionManager`. Transaction scope quá lớn có thể gây timeout vì transaction chưa commit, do đó scope của transaction phải nhỏ hơn lock.
- **Declarative transaction**: cấu hình trong XML configuration file hoặc trực tiếp dựa trên annotation (khuyến nghị dùng cho standalone application hoặc business system đơn giản): thực chất được triển khai thông qua AOP (cách dùng full annotation dựa trên `@Transactional` là phổ biến nhất).

### Spring transaction có những transaction propagation behavior nào?

**Transaction propagation behavior dùng để giải quyết vấn đề transaction khi các business-layer method gọi lẫn nhau.**

Khi một transaction method được transaction method khác gọi, phải chỉ định transaction cần propagate như thế nào. Ví dụ: method có thể tiếp tục chạy trong transaction hiện tại hoặc mở transaction mới và chạy trong transaction riêng của nó.

Các giá trị transaction propagation behavior hợp lệ như sau:

**1. `TransactionDefinition.PROPAGATION_REQUIRED`**

Đây là transaction propagation behavior được dùng nhiều nhất. Annotation `@Transactional` chúng ta thường dùng mặc định cũng sử dụng behavior này. Nếu hiện tại có transaction thì tham gia transaction đó; nếu hiện tại không có transaction thì tạo transaction mới.

**2. `TransactionDefinition.PROPAGATION_REQUIRES_NEW`**

Tạo transaction mới. Nếu hiện tại có transaction thì suspend transaction hiện tại. Nghĩa là bất kể outer method có mở transaction hay không, inner method được đánh dấu `Propagation.REQUIRES_NEW` sẽ mở transaction riêng; các transaction độc lập và không ảnh hưởng lẫn nhau.

**3. `TransactionDefinition.PROPAGATION_NESTED`**

Nếu hiện tại có transaction thì tạo một transaction làm nested transaction của transaction hiện tại để chạy; nếu hiện tại không có transaction thì giá trị này tương đương `TransactionDefinition.PROPAGATION_REQUIRED`.

**4. `TransactionDefinition.PROPAGATION_MANDATORY`**

Nếu hiện tại có transaction thì tham gia transaction đó; nếu hiện tại không có transaction thì ném exception (mandatory: bắt buộc).

Cách này ít được sử dụng.

Ba transaction propagation behavior còn lại cũng là cấu hình hợp lệ, cần hiểu dựa trên việc có outer transaction hay không:

- **`TransactionDefinition.PROPAGATION_SUPPORTS`**: Nếu hiện tại có transaction thì tham gia transaction đó; nếu hiện tại không có transaction thì tiếp tục chạy theo cách non-transactional.
- **`TransactionDefinition.PROPAGATION_NOT_SUPPORTED`**: Chạy theo cách non-transactional; nếu hiện tại có transaction thì suspend transaction hiện tại.
- **`TransactionDefinition.PROPAGATION_NEVER`**: Chạy theo cách non-transactional; nếu hiện tại có transaction thì ném exception.

### Spring transaction có những isolation level nào?

Giống phần transaction propagation behavior, để thuận tiện sử dụng, Spring cũng định nghĩa một enum class tương ứng: `Isolation`.

```java
public enum Isolation {

    DEFAULT(TransactionDefinition.ISOLATION_DEFAULT),
    READ_UNCOMMITTED(TransactionDefinition.ISOLATION_READ_UNCOMMITTED),
    READ_COMMITTED(TransactionDefinition.ISOLATION_READ_COMMITTED),
    REPEATABLE_READ(TransactionDefinition.ISOLATION_REPEATABLE_READ),
    SERIALIZABLE(TransactionDefinition.ISOLATION_SERIALIZABLE);

    private final int value;

    Isolation(int value) {
        this.value = value;
    }

    public int value() {
        return this.value;
    }

}
```

Dưới đây lần lượt giới thiệu từng transaction isolation level:

- **`TransactionDefinition.ISOLATION_DEFAULT`**: Dùng isolation level mặc định của backend database. MySQL mặc định dùng isolation level `REPEATABLE_READ`, Oracle mặc định dùng isolation level `READ_COMMITTED`.
- **`TransactionDefinition.ISOLATION_READ_UNCOMMITTED`**: Isolation level thấp nhất, ít được sử dụng vì cho phép đọc data change chưa commit, **có thể gây dirty read, phantom read hoặc non-repeatable read**.
- **`TransactionDefinition.ISOLATION_READ_COMMITTED`**: Cho phép đọc data đã commit của concurrent transaction, **có thể ngăn dirty read, nhưng phantom read hoặc non-repeatable read vẫn có thể xảy ra**.
- **`TransactionDefinition.ISOLATION_REPEATABLE_READ`**: Kết quả nhiều lần đọc cùng một field đều nhất quán, trừ khi data bị chính transaction hiện tại sửa đổi, **có thể ngăn dirty read và non-repeatable read, nhưng phantom read vẫn có thể xảy ra**.
- **`TransactionDefinition.ISOLATION_SERIALIZABLE`**: Isolation level cao nhất, hoàn toàn tuân thủ isolation level của ACID. Mọi transaction lần lượt thực thi, nhờ đó hoàn toàn không thể gây ảnh hưởng lẫn nhau, tức **level này có thể ngăn dirty read, non-repeatable read và phantom read**. Tuy nhiên nó ảnh hưởng nghiêm trọng đến performance của chương trình, thông thường cũng không dùng level này.

### Bạn có biết annotation @Transactional(rollbackFor = Exception.class) không?

`Exception` được chia thành runtime exception `RuntimeException` và non-runtime exception. Transaction management rất quan trọng đối với enterprise application; ngay cả khi xảy ra exception, nó vẫn có thể bảo đảm tính nhất quán của data.

Khi annotation `@Transactional` áp dụng trên class, mọi public method của class đó đều có transaction property này; đồng thời cũng có thể dùng annotation ở cấp method để override định nghĩa ở cấp class.

Rollback strategy mặc định của annotation `@Transactional` là chỉ rollback transaction khi gặp `RuntimeException` (runtime exception) hoặc `Error`, chứ không rollback `Checked Exception` (checked exception). Điều này là vì Spring cho rằng `RuntimeException` và Error là lỗi không thể dự đoán, còn checked exception là lỗi có thể dự đoán và có thể xử lý bằng business logic.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/spring-transactional-rollbackfor.png)

Nếu muốn thay đổi rollback strategy mặc định, có thể dùng property `rollbackFor` và `noRollbackFor` của annotation `@Transactional` để chỉ định exception nào cần rollback và exception nào không cần rollback. Ví dụ, muốn mọi exception đều rollback transaction thì dùng annotation như sau:

```java
@Transactional(rollbackFor = Exception.class)
public void someMethod() {
// some business logic
}
```

Nếu muốn một số exception cụ thể không rollback transaction, có thể dùng annotation như sau:

```java
@Transactional(noRollbackFor = CustomException.class)
public void someMethod() {
// some business logic
}
```

## Spring Data JPA

Điều quan trọng với JPA là thực hành, phần này chỉ tổng hợp một số ít kiến thức.

### Dùng JPA để một field không được persist trong database như thế nào?

Giả sử có class dưới đây:

```java
@Entity(name="USER")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "ID")
    private Long id;

    @Column(name="USER_NAME")
    private String userName;

    @Column(name="PASSWORD")
    private String password;

    private String secrect;

}
```

Nếu muốn field `secrect` không được persist, tức không được database lưu thì phải làm gì? Có thể dùng một số cách sau:

```java
static String transient1; // không persistent vì là static
final String transient2 = "Satish"; // không persistent vì là final
transient String transient3; // không persistent vì là transient
@Transient
String transient4; // không persistent vì là @Transient
```

Thông thường hai cách sau được dùng nhiều hơn; cá nhân tôi dùng annotation nhiều hơn.

### Chức năng auditing của JPA dùng để làm gì? Có tác dụng gì?

Auditing chủ yếu giúp ghi lại hành vi cụ thể của database operation, chẳng hạn record nào do ai tạo, tạo lúc nào, người sửa cuối cùng là ai và sửa lần cuối lúc nào.

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@MappedSuperclass
@EntityListeners(value = AuditingEntityListener.class)
public abstract class AbstractAuditBase {

    @CreatedDate
    @Column(updatable = false)
    @JsonIgnore
    private Instant createdAt;

    @LastModifiedDate
    @JsonIgnore
    private Instant updatedAt;

    @CreatedBy
    @Column(updatable = false)
    @JsonIgnore
    private String createdBy;

    @LastModifiedBy
    @JsonIgnore
    private String updatedBy;
}
```

- `@CreatedDate`: Cho biết field này là field thời gian tạo, khi entity được insert thì sẽ set value.

- `@CreatedBy`: Cho biết field này là người tạo, khi entity được insert thì sẽ set value.

  `@LastModifiedDate`, `@LastModifiedBy` cũng tương tự.

### Những annotation nào dùng để biểu diễn relationship giữa các entity?

- `@OneToOne`: one-to-one.
- `@ManyToMany`: many-to-many.
- `@OneToMany`: one-to-many.
- `@ManyToOne`: many-to-one.

Dùng `@ManyToOne` và `@OneToMany` cũng có thể biểu diễn relationship many-to-many.

## Spring Security

Điều quan trọng với Spring Security là thực hành, phần này chỉ tổng hợp một số ít kiến thức.

### Có những cách nào để kiểm soát request access permission?

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/image-20220728201854641.png)

- `permitAll()`: Cho phép vô điều kiện mọi hình thức access, bất kể đã login hay chưa.
- `anonymous()`: Cho phép anonymous access, tức chỉ có thể access khi chưa login.
- `denyAll()`: Từ chối vô điều kiện mọi hình thức access.
- `authenticated()`: Chỉ cho phép user đã authenticated access.
- `fullyAuthenticated()`: Chỉ cho phép user đã full authentication access, không chấp nhận anonymous authentication hoặc remember-me authentication.
- `hasRole(String)`: Chỉ cho phép role được chỉ định access.
- `hasAnyRole(String)`: Chỉ định một hoặc nhiều role, user thỏa mãn ít nhất một role thì có thể access.
- `hasAuthority(String)`: Chỉ cho phép user có authority được chỉ định access.
- `hasAnyAuthority(String)`: Chỉ định một hoặc nhiều authority, user thỏa mãn ít nhất một authority thì có thể access.
- `hasIpAddress(String)`: Chỉ cho phép user có IP được chỉ định access.

### hasRole và hasAuthority có khác nhau không?

Có thể xem bài viết của tác giả Song: [hasRole và hasAuthority trong Spring Security có khác nhau không?](https://mp.weixin.qq.com/s/GTNOa2k9_n_H0w24upClRw), bài viết giới thiệu khá chi tiết.

### ⭐️Mã hóa password như thế nào?

Nếu cần lưu sensitive data như password vào database, trước hết cần encode bằng adaptive one-way hash function rồi mới lưu, thay vì dùng reversible encryption.

Spring Security cung cấp nhiều implementation của password encoding algorithm, dùng được ngay sau khi cài đặt. Interface của các implementation này là `PasswordEncoder`; nếu cần custom password encoding scheme thì cũng cần implement interface `PasswordEncoder`.

Interface `PasswordEncoder` có hai abstract method bắt buộc phải implement là `encode()` và `matches()`, cùng một default method `upgradeEncoding()` có thể override khi cần.

```java
public interface PasswordEncoder {
    // Encode một chiều password gốc
    String encode(CharSequence var1);
    // So sánh password gốc với password lưu trong database
    boolean matches(CharSequence var1, String var2);
    // Kiểm tra password đã encode có cần nâng cấp encoding không, mặc định trả về false
    default boolean upgradeEncoding(String encodedPassword) {
        return false;
    }
}
```

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/image-20220728183540954.png)

Spring chính thức khuyến nghị dùng adaptive one-way function có work factor điều chỉnh được và điều chỉnh thời gian verify theo performance của hệ thống, chẳng hạn bcrypt, PBKDF2, scrypt hoặc Argon2.

### Thay đổi encryption algorithm mà hệ thống sử dụng một cách linh hoạt như thế nào?

Nếu trong quá trình phát triển đột nhiên phát hiện encryption algorithm hiện tại không đáp ứng yêu cầu và cần đổi sang algorithm khác thì nên làm gì?

Cách được khuyến nghị là dùng `DelegatingPasswordEncoder` để tương thích với nhiều password encryption scheme khác nhau, đáp ứng các business requirement khác nhau.

Nhìn từ tên gọi cũng có thể thấy `DelegatingPasswordEncoder` thực ra là một proxy class, không phải một encryption algorithm hoàn toàn mới. Nó proxy các implementation của encryption algorithm đã đề cập ở trên. Sau Spring Security 5.0, mặc định password encryption đã dựa trên `DelegatingPasswordEncoder`.

## Tài liệu tham khảo

- 《Bí mật bên trong Spring》
- 《Học chuyên sâu Spring từ con số 0》: <https://juejin.cn/book/6857911863016390663>
- <http://www.cnblogs.com/wmyskxz/p/8820371.html>
- <https://www.journaldev.com/2696/spring-interview-questions-and-answers>
- <https://www.edureka.co/blog/interview-questions/spring-interview-questions/>
- <https://www.cnblogs.com/clwydjgs/p/9317849.html>
- <https://howtodoinjava.com/interview-questions/top-spring-interview-questions-with-answers/>
- <http://www.tomaszezula.com/2014/02/09/spring-series-part-5-component-vs-bean/>
- <https://stackoverflow.com/questions/34172888/difference-between-bean-and-autowired>

<!-- @include: @article-footer.snippet.md -->
