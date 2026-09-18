---
title: "Chuyên đề Spring & Spring Boot: IoC, AOP, transaction, auto-configuration, annotation thường dùng và source code"
description: "Lộ trình học và phỏng vấn Spring, Spring Boot, bao quát IoC, AOP, Bean lifecycle, transaction, auto-configuration, annotation thường dùng, design pattern, @Async, source code và các câu hỏi phỏng vấn thường gặp."
category: Framework
tag:
  - Spring
  - Spring Boot
  - Phỏng vấn backend
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: "Spring,Spring Boot,câu hỏi phỏng vấn Spring,câu hỏi phỏng vấn Spring Boot,IoC,AOP,Bean lifecycle,Spring transaction,Spring auto-configuration,annotation thường dùng của Spring,Spring source code,câu hỏi phỏng vấn Java backend"
---

Spring là một trong những infrastructure cốt lõi nhất của Java backend. Học Spring không nên chỉ học thuộc annotation, mà còn phải hiểu IoC, AOP, Bean lifecycle, transaction, auto-configuration, design pattern và các extension point thường gặp.

Spring Boot tiếp tục tích hợp configuration, dependency management, auto-configuration và năng lực observability trong production, giúp phát triển application nhanh hơn nhưng cũng dễ khiến người học bỏ qua nguyên lý bên trong.

Nếu thời gian khá hạn hẹp, bạn có thể xem trước [Tổng hợp câu hỏi phỏng vấn Spring thường gặp](https://interview.javaguide.cn/system-design/spring.html), sau đó quay lại chuyên đề này để bổ sung đầy đủ các chi tiết về IoC, AOP, transaction và auto-configuration.

## Dành cho ai

- Java backend developer đang học Spring, Spring MVC, Spring Boot một cách hệ thống.
- Bạn đang chuẩn bị các câu hỏi phỏng vấn thường gặp về Spring, Spring Boot.
- Bạn đã dùng Spring Boot để phát triển project nhưng chưa hiểu sâu về IoC, AOP, transaction và auto-configuration.
- Engineer muốn hiểu infrastructure nền tảng của backend từ góc độ nguyên lý của framework.

## Trọng tâm học

- Spring IoC giải quyết vấn đề tạo object và quản lý dependency, AOP giải quyết vấn đề tái sử dụng cross-cutting logic.
- Bean lifecycle, scope, circular dependency và extension point là những điểm then chốt để hiểu Spring container.
- Với Spring transaction, cần nắm vững propagation behavior, isolation level, rollback rule và các trường hợp mất hiệu lực.
- Cốt lõi của Spring Boot auto-configuration nằm ở conditional configuration, configuration binding và hệ thống Starter.
- Học annotation không thể chỉ học thuộc công dụng, mà còn phải biết năng lực của container tương ứng phía sau nó.
- Spring source code và design pattern phù hợp để đào sâu hiểu biết, không nên ngay từ đầu đã cố đọc kỹ từng chi tiết source code.

## Thứ tự đọc đề xuất

1. [Tổng hợp câu hỏi phỏng vấn Spring thường gặp](./spring-knowledge-and-questions-summary.md): trước tiên lập danh sách các vấn đề thường gặp về Spring.
2. [Giải thích chi tiết IoC & AOP (hiểu nhanh)](./ioc-and-aop.md): hiểu hai khái niệm nền tảng cốt lõi nhất của Spring.
3. [Tổng hợp annotation thường dùng của Spring & Spring MVC & Spring Boot](./spring-common-annotations.md): liên hệ annotation thường dùng với năng lực của container.
4. [Giải thích chi tiết Spring transaction](./spring-transaction.md): tập trung nắm vững transaction propagation, isolation level, rollback rule và các trường hợp mất hiệu lực.
5. [Giải thích chi tiết nguyên lý Spring Boot auto-configuration](./spring-boot-auto-assembly-principles.md): hiểu vì sao Spring Boot có thể hoạt động ngay sau khi cài đặt.
6. Sau đó, tùy nhu cầu đọc thêm [Giải thích chi tiết design pattern trong Spring](./spring-design-patterns-summary.md), [Phân tích nguyên lý annotation Async](./async.md) và [Đọc hiểu source code cốt lõi của Spring Boot](./springboot-source-code.md).

## Bài viết cốt lõi

- [Tổng hợp câu hỏi phỏng vấn Spring thường gặp](./spring-knowledge-and-questions-summary.md): bao quát các kiến thức cốt lõi của Spring như IoC container, nguyên lý AOP, Bean lifecycle và dependency injection.
- [Tổng hợp câu hỏi phỏng vấn Spring Boot thường gặp](./springboot-knowledge-and-questions-summary.md): bao quát nguyên lý auto-configuration, cơ chế Starter, việc tải file configuration và giám sát bằng Actuator.
- [Giải thích chi tiết IoC & AOP (hiểu nhanh)](./ioc-and-aop.md): giải thích inversion of control, dependency injection, aspect-oriented programming và cơ chế triển khai dynamic proxy.
- [Tổng hợp annotation thường dùng của Spring & Spring MVC & Spring Boot](./spring-common-annotations.md): hệ thống hóa các annotation thường dùng như `@Autowired`, `@Component`, `@RequestMapping`.
- [Giải thích chi tiết Spring transaction](./spring-transaction.md): bao quát `@Transactional`, transaction propagation, isolation level, các trường hợp transaction mất hiệu lực và rollback rule.
- [Giải thích chi tiết nguyên lý Spring Boot auto-configuration](./spring-boot-auto-assembly-principles.md): phân tích `@EnableAutoConfiguration`, cơ chế tải SpringFactories và conditional annotation.
- [Giải thích chi tiết design pattern trong Spring](./spring-design-patterns-summary.md): hiểu việc áp dụng factory pattern, proxy pattern, singleton pattern, template method và các pattern khác trong Spring.
- [Phân tích nguyên lý annotation Async](./async.md): hiểu configuration của asynchronous task, thiết lập thread pool và cơ chế `@EnableAsync`.
- [Đọc hiểu source code cốt lõi của Spring Boot](./springboot-source-code.md): hiểu startup flow, cơ chế auto-configuration và SpringApplication từ góc độ source code.

## Câu hỏi thường gặp

- IoC là gì? DI là gì?
- Spring AOP có quan hệ gì với dynamic proxy?
- Bean của Spring có lifecycle như thế nào?
- Spring giải quyết circular dependency như thế nào? Những circular dependency nào không thể giải quyết?
- `@Autowired` và `@Resource` khác nhau thế nào?
- Spring transaction có những propagation behavior nào?
- Những trường hợp mất hiệu lực thường gặp của `@Transactional` là gì?
- Quy trình Spring Boot auto-configuration như thế nào?
- Starter có tác dụng gì? Làm thế nào để tự định nghĩa một Starter?
- Spring sử dụng những design pattern nào?
- Vì sao `@Async` đôi khi không có hiệu lực?
- Nên bắt đầu đọc Spring source code từ những entry point nào?

## Chuyên đề liên quan

- [Hệ thống kiến thức system design](../../)
- [Chuyên đề nền tảng system design](../../basis/)
- [Tổng hợp câu hỏi phỏng vấn design pattern thường gặp](../../design-pattern.md)
- [Tổng hợp câu hỏi phỏng vấn MyBatis thường gặp](../mybatis/mybatis-interview.md)
- [Hệ thống kiến thức distributed system](../../../distributed-system/)

<!-- @include: @article-footer.snippet.md -->
