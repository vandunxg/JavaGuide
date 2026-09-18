---
title: "Giải thích chi tiết nguyên lý auto-configuration của Spring Boot"
description: "Phân tích chuyên sâu nguyên lý auto-configuration của Spring Boot, giải thích chi tiết cơ chế tải @EnableAutoConfiguration, SpringFactories và nguyên lý hoạt động của conditional annotation."
category: Framework
tag:
  - SpringBoot
head:
  - - meta
    - name: keywords
      content: "Spring Boot auto-configuration,AutoConfiguration,EnableAutoConfiguration,SpringFactories,conditional annotation,Starter,nguyên lý Spring Boot"
---

> Tác giả: [Miki-byte-1024](https://github.com/Miki-byte-1024) & [Snailclimb](https://github.com/Snailclimb)

Mỗi khi được hỏi về Spring Boot, interviewer thường rất thích hỏi câu này: “Hãy trình bày nguyên lý auto-configuration của Spring Boot?”.

Theo tôi, có thể trả lời từ các khía cạnh sau:

1. Spring Boot auto-configuration là gì?
2. Spring Boot thực hiện auto-configuration như thế nào? Làm thế nào để tải theo nhu cầu?
3. Làm thế nào để triển khai một Starter?

Do giới hạn dung lượng, bài viết này không đi sâu. Bạn cũng có thể trực tiếp dùng debug để xem source code của phần auto-configuration trong Spring Boot.

## Lời mở đầu

Nếu từng sử dụng Spring, chắc chắn bạn đã từng bị nỗi ám ảnh về việc cấu hình XML “thống trị”. Ngay cả sau khi Spring bổ sung cấu hình dựa trên annotation, khi bật một số tính năng của Spring hoặc thêm dependency bên thứ ba, chúng ta vẫn phải cấu hình tường minh bằng XML hoặc Java.

Ví dụ, khi chưa có Spring Boot, để viết một dịch vụ RestFul Web, trước hết chúng ta còn phải thực hiện cấu hình như sau.

```java
@Configuration
public class RESTConfiguration
{
    @Bean
    public View jsonTemplate() {
        MappingJackson2JsonView view = new MappingJackson2JsonView();
        view.setPrettyPrint(true);
        return view;
    }

    @Bean
    public ViewResolver viewResolver() {
        return new BeanNameViewResolver();
    }
}
```

`spring-servlet.xml`

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context/ http://www.springframework.org/schema/context/spring-context.xsd
    http://www.springframework.org/schema/mvc/ http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <context:component-scan base-package="com.howtodoinjava.demo" />
    <mvc:annotation-driven />

    <!-- JSON Support -->
    <bean name="viewResolver" class="org.springframework.web.servlet.view.BeanNameViewResolver"/>
    <bean name="jsonTemplate" class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>

</beans>
```

Tuy nhiên, với dự án Spring Boot, chúng ta chỉ cần thêm dependency liên quan, không cần cấu hình, rồi khởi động bằng method `main` dưới đây.

```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

Ngoài ra, chúng ta có thể thiết lập dự án thông qua file cấu hình toàn cục `application.properties` hoặc `application.yml` của Spring Boot, chẳng hạn đổi port, cấu hình thuộc tính JPA, v.v.

**Vì sao sử dụng Spring Boot lại thuận tiện đến vậy?** Đó là nhờ auto-configuration. **Có thể nói auto-configuration là core của Spring Boot, vậy chính xác auto-configuration là gì?**

## Spring Boot auto-configuration là gì?

Khi nhắc đến auto-configuration, hiện nay chúng ta thường liên hệ nó với Spring Boot. Tuy nhiên, trên thực tế Spring Framework đã sớm triển khai chức năng này. Spring Boot chỉ tối ưu thêm trên nền tảng đó thông qua SPI.

> Trong Spring Boot 2.6 và các phiên bản cũ hơn, các class auto-configuration chủ yếu được đăng ký thông qua `META-INF/spring.factories` trong các jar bên ngoài. Spring Boot 2.7 bổ sung `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` và đồng thời vẫn tương thích với cách đăng ký cũ; Spring Boot 3.0 loại bỏ hỗ trợ đăng ký class auto-configuration thông qua key `EnableAutoConfiguration` trong `spring.factories`, nhưng các mục đích sử dụng khác của `spring.factories` không bị ảnh hưởng.

Khi chưa có Spring Boot, nếu cần thêm dependency bên thứ ba, chúng ta phải cấu hình thủ công, rất phiền phức. Nhưng trong Spring Boot, chúng ta chỉ cần thêm trực tiếp một starter. Ví dụ, nếu muốn sử dụng redis trong dự án, chỉ cần thêm starter tương ứng vào dự án.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

Sau khi thêm starter, chúng ta có thể sử dụng chức năng do component bên thứ ba cung cấp chỉ bằng một số annotation và cấu hình đơn giản.

Theo tôi, có thể hiểu đơn giản auto-configuration như sau: **thông qua annotation hoặc một số cấu hình đơn giản, thực hiện một chức năng nào đó với sự hỗ trợ của Spring Boot.**

## Spring Boot thực hiện auto-configuration như thế nào?

Trước hết, hãy xem annotation cốt lõi `SpringBootApplication` của Spring Boot.

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
<1.>@SpringBootConfiguration
<2.>@ComponentScan
<3.>@EnableAutoConfiguration
public @interface SpringBootApplication {

}

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration //thực tế nó cũng là một class cấu hình
public @interface SpringBootConfiguration {
}
```

Có thể xem `@SpringBootApplication` là tập hợp của các annotation `@Configuration`, `@EnableAutoConfiguration` và `@ComponentScan`. Theo website chính thức của Spring Boot, tác dụng của ba annotation này lần lượt là:

- `@EnableAutoConfiguration`: bật cơ chế auto-configuration của Spring Boot
- `@Configuration`: cho phép đăng ký thêm bean hoặc import các class cấu hình khác vào context
- `@ComponentScan`: quét các bean được đánh dấu bằng `@Component` (`@Service`, `@Controller`); mặc định annotation sẽ quét tất cả class trong package chứa class khởi động, đồng thời có thể tùy chỉnh để không quét một số bean. Như hình dưới đây, container sẽ loại trừ `TypeExcludeFilter` và `AutoConfigurationExcludeFilter`.

![](https://oss.javaguide.cn/p3-juejin/bcc73490afbe4c6ba62acde6a94ffdfd~tplv-k3u1fbpfcp-watermark.png)

`@EnableAutoConfiguration` là annotation quan trọng để triển khai auto-configuration, vì vậy chúng ta sẽ bắt đầu từ annotation này.

### @EnableAutoConfiguration: annotation cốt lõi để triển khai auto-configuration

`EnableAutoConfiguration` chỉ là một annotation đơn giản; chức năng cốt lõi của auto-configuration thực tế được triển khai thông qua class `AutoConfigurationImportSelector`.

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage //tác dụng: đăng ký tất cả component trong package main vào container
@Import({AutoConfigurationImportSelector.class}) //tải class auto-configuration xxxAutoconfiguration
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

Bây giờ chúng ta sẽ phân tích trọng tâm xem class `AutoConfigurationImportSelector` thực hiện những gì.

### AutoConfigurationImportSelector: tải class auto-configuration

Dưới đây là phần trích source code của Spring Boot 2.1.x dùng làm ví dụ để phân tích `AutoConfigurationImportSelector`. Một số phần triển khai không ảnh hưởng đến flow đã được lược bỏ, nên không thể biên dịch trực tiếp thành một class độc lập. Từ Spring Boot 2.7 trở lên, các class auto-configuration ứng viên chủ yếu được đọc từ file `AutoConfiguration.imports`; cấu trúc source code cụ thể khác với code của phiên bản cũ dưới đây.

Hệ thống kế thừa của class `AutoConfigurationImportSelector` như sau:

```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {

}

public interface DeferredImportSelector extends ImportSelector {

}

public interface ImportSelector {
    String[] selectImports(AnnotationMetadata var1);
}
```

Có thể thấy class `AutoConfigurationImportSelector` triển khai interface `ImportSelector`, đồng thời triển khai method `selectImports` của interface này. Method này chủ yếu dùng để **lấy fully qualified class name của tất cả class thỏa mãn điều kiện; các class này cần được tải vào IoC container**.

```java
private static final String[] NO_IMPORTS = new String[0];

public String[] selectImports(AnnotationMetadata annotationMetadata) {
        // <1>.kiểm tra công tắc auto-configuration đã được bật chưa
        if (!this.isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        } else {
          //<2>.lấy tất cả bean cần được auto-configuration
            AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
            AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(autoConfigurationMetadata, annotationMetadata);
            return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
        }
    }
```

Ở đây chúng ta cần chú ý method `getAutoConfigurationEntry()`. Method này chủ yếu chịu trách nhiệm tải các class auto-configuration.

Chuỗi gọi của method này như sau:

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/3c1200712655443ca4b38500d615bb70~tplv-k3u1fbpfcp-watermark.png)

Bây giờ chúng ta sẽ kết hợp với source code của `getAutoConfigurationEntry()` để phân tích chi tiết.

```java
private static final AutoConfigurationEntry EMPTY_ENTRY = new AutoConfigurationEntry();

AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata, AnnotationMetadata annotationMetadata) {
        //<1>.
        if (!this.isEnabled(annotationMetadata)) {
            return EMPTY_ENTRY;
        } else {
            //<2>.
            AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
            //<3>.
            List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
            //<4>.
            configurations = this.removeDuplicates(configurations);
            Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
            this.checkExcludedClasses(configurations, exclusions);
            configurations.removeAll(exclusions);
            configurations = this.filter(configurations, autoConfigurationMetadata);
            this.fireAutoConfigurationImportEvents(configurations, exclusions);
            return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
        }
    }
```

**Bước 1**:

Kiểm tra công tắc auto-configuration đã được bật chưa. Mặc định là `spring.boot.enableautoconfiguration=true`, có thể thiết lập trong `application.properties` hoặc `application.yml`.

![](https://oss.javaguide.cn/p3-juejin/77aa6a3727ea4392870f5cccd09844ab~tplv-k3u1fbpfcp-watermark.png)

**Bước 2**:

Dùng để lấy `exclude` và `excludeName` trong annotation `EnableAutoConfiguration`.

![](https://oss.javaguide.cn/p3-juejin/3d6ec93bbda1453aa08c52b49516c05a~tplv-k3u1fbpfcp-zoom-1.png)

**Bước 3**

Trong source code Spring Boot 2.1.x được sử dụng ở bài viết này, khi lấy tất cả class cấu hình cần auto-configuration, hệ thống sẽ đọc `META-INF/spring.factories`:

```plain
spring-boot/spring-boot-project/spring-boot-autoconfigure/src/main/resources/META-INF/spring.factories
```

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/58c51920efea4757aa1ec29c6d5f9e36~tplv-k3u1fbpfcp-watermark.png)

Từ hình dưới đây có thể thấy nội dung cấu hình của file này đã được đọc. Tác dụng của `XXXAutoConfiguration` là tải component theo nhu cầu.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/94d6e1a060ac41db97043e1758789026~tplv-k3u1fbpfcp-watermark.png)

Không chỉ resource có cùng tên `META-INF/spring.factories` trong dependency này được đọc; các resource cùng tên trong những jar khác trên classpath cũng được `SpringFactoriesLoader` hợp nhất và đọc. Cần lưu ý, Starter thường chỉ dùng để gom các dependency jar; code auto-configuration và file đăng ký có thể đặt trong module autoconfigure độc lập hoặc gộp cùng Starter, không phải Starter nào cũng bắt buộc phải chứa `spring.factories`.

Vì vậy, bạn có thể thấy rõ Starter của connection pool database druid cho Spring Boot đã tạo file `META-INF/spring.factories`.

Nếu muốn viết auto-configuration cho Spring Boot 2.6 và các phiên bản cũ hơn, cần sử dụng cách đăng ký này; auto-configuration hướng đến Spring Boot 3.x nên chuyển sang dùng `AutoConfiguration.imports`.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/68fa66aeee474b0385f94d23bcfe1745~tplv-k3u1fbpfcp-watermark.png)

**Bước 4**:

Đến đây, interviewer có thể hỏi bạn: “Trong `spring.factories` có nhiều cấu hình như vậy, mỗi lần khởi động có phải tải tất cả không?”.

Rõ ràng, điều đó không thực tế. Khi debug sâu hơn, bạn sẽ phát hiện giá trị của `configurations` đã nhỏ đi.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/267f8231ae2e48d982154140af6437b0~tplv-k3u1fbpfcp-watermark.png)

Bởi vì ở bước này, hệ thống đã thực hiện một lần lọc: chỉ khi tất cả điều kiện trong `@ConditionalOnXXX` đều được thỏa mãn thì class đó mới có hiệu lực.

```java
@Configuration
// kiểm tra các class liên quan: RabbitTemplate và Channel có tồn tại không
// chỉ tải khi chúng tồn tại
@ConditionalOnClass({ RabbitTemplate.class, Channel.class })
@EnableConfigurationProperties(RabbitProperties.class)
@Import(RabbitAnnotationDrivenConfiguration.class)
public class RabbitAutoConfiguration {
}
```

Nếu quan tâm, bạn có thể tìm hiểu chi tiết các conditional annotation do Spring Boot cung cấp:

- `@ConditionalOnBean`: khi container có Bean được chỉ định
- `@ConditionalOnMissingBean`: khi container không có Bean được chỉ định
- `@ConditionalOnSingleCandidate`: khi Bean được chỉ định chỉ có một trong container, hoặc có nhiều Bean nhưng có một Bean được chỉ định là Bean ưu tiên
- `@ConditionalOnClass`: khi class được chỉ định tồn tại trên classpath
- `@ConditionalOnMissingClass`: khi class được chỉ định không tồn tại trên classpath
- `@ConditionalOnProperty`: thuộc tính được chỉ định có giá trị được chỉ định hay không
- `@ConditionalOnResource`: classpath có resource được chỉ định hay không
- `@ConditionalOnExpression`: dùng biểu thức SpEL làm điều kiện
- `@ConditionalOnJava`: dùng phiên bản Java làm điều kiện
- `@ConditionalOnJndi`: khi JNDI tồn tại tại vị trí được chỉ định
- `@ConditionalOnNotWebApplication`: khi dự án hiện tại không phải Web project
- `@ConditionalOnWebApplication`: khi dự án hiện tại là Web project

## Làm thế nào để triển khai một Starter

Chỉ nói mà không thực hành thì không đủ; bây giờ hãy tạo một starter để triển khai thread pool tùy chỉnh.

Bước 1, tạo project `threadpool-spring-boot-starter`.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/1ff0ebe7844f40289eb60213af72c5a6~tplv-k3u1fbpfcp-watermark.png)

Bước 2, thêm dependency liên quan đến Spring Boot.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/5e14254276604f87b261e5a80a354cc0~tplv-k3u1fbpfcp-watermark.png)

Bước 3, tạo `ThreadPoolAutoConfiguration`.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/1843f1d12c5649fba85fd7b4e4a59e39~tplv-k3u1fbpfcp-watermark.png)

Bước 4, đăng ký class auto-configuration. Với Spring Boot 2.6 và các phiên bản cũ hơn, tạo file `META-INF/spring.factories` trong package resources của project `threadpool-spring-boot-starter`; với Spring Boot 2.7 trở lên, sử dụng `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`; khi hướng đến Spring Boot 3.x, class auto-configuration thường được đánh dấu bằng `@AutoConfiguration`.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/97b738321f1542ea8140484d6aaf0728~tplv-k3u1fbpfcp-watermark.png)

Cuối cùng, tạo project mới và thêm `threadpool-spring-boot-starter`.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/edcdd8595a024aba85b6bb20d0e3fed4~tplv-k3u1fbpfcp-watermark.png)

Kiểm thử thành công!

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/9a265eea4de742a6bbdbbaa75f437307~tplv-k3u1fbpfcp-watermark.png)

## Tổng kết

Spring Boot bật auto-configuration thông qua `@EnableAutoConfiguration` và tải các class auto-configuration ứng viên đã được đăng ký trên classpath. Spring Boot 2.6 và các phiên bản cũ hơn chủ yếu đăng ký thông qua `spring.factories`, còn Spring Boot 2.7 trở lên sử dụng `AutoConfiguration.imports`. Các class auto-configuration kết hợp với nhóm annotation `@Conditional` để có hiệu lực theo nhu cầu; tác dụng chính của Starter là gom các dependency thường dùng, không phải một package name cố định bắt buộc để auto-configuration có hiệu lực.

<!-- @include: @article-footer.snippet.md -->
