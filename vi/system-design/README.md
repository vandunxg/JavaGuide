---
title: "Hệ thống kiến thức về System Design: Design Pattern, nền tảng engineering, authentication và authorization, data security cùng các framework phổ biến"
description: "Lộ trình học về System Design và các câu hỏi phỏng vấn, bao quát Design Pattern, RESTful API, software engineering, code refactoring, unit testing, authentication và authorization, data security, Spring và real-time message push."
category: System Design
tag:
  - System Design
  - Design Pattern
  - Phỏng vấn backend
sitemap:
  changefreq: weekly
  priority: 0.95
head:
  - - meta
    - name: keywords
      content: System Design,câu hỏi phỏng vấn System Design,Design Pattern,RESTful API,software engineering,code refactoring,unit testing,authentication và authorization,JWT,SSO,permission system,data security,Spring,Spring Boot,MyBatis,Netty,scheduled tasks,real-time message push,phỏng vấn backend
---

<!-- @include: @small-advertisement.snippet.md -->

Hệ thống **kiến thức về System Design** này dành cho việc học backend, thực hành engineering và ôn tập phỏng vấn, tập hợp các bài viết về System Design trên trang này xoay quanh việc “phân rã yêu cầu nghiệp vụ thành giải pháp engineering ổn định, dễ bảo trì và dễ mở rộng”.

System Design không chỉ là học thuộc các bài toán tình huống như flash sale, short URL hay Feed. Khi đi vào backend, bạn còn cần hiểu các năng lực nền tảng như thiết kế API, code quality, Design Pattern, nguyên lý framework, authentication và authorization, data security, scheduled tasks và real-time message push.

Nếu có ít thời gian, bạn nên đọc trước [Tổng hợp câu hỏi phỏng vấn System Design thường gặp](./system-design-questions.md) để nhanh chóng xây dựng danh sách các vấn đề thường gặp; nếu muốn bổ sung kiến thức nền tảng một cách có hệ thống, bạn có thể tiếp tục theo thứ tự đọc bên dưới.

## Dành cho ai

- Developer đang học System Design backend một cách có hệ thống.
- Các bạn chuẩn bị phỏng vấn backend khi tuyển dụng mới tốt nghiệp, tuyển dụng có kinh nghiệm hoặc tại các công ty công nghệ vừa và lớn.
- Engineer muốn nâng cấp từ “biết viết business code” lên “có thể thiết kế module và giải pháp”.
- Người đọc đã tiếp xúc với Spring, MyBatis, permission system, scheduled tasks và các công nghệ khác nhưng chưa hệ thống hóa kiến thức.

## Trọng tâm học

- Trong phỏng vấn System Design, tổ chức câu trả lời thế nào từ việc làm rõ yêu cầu, quy trình cốt lõi, data model, thiết kế interface, khả năng mở rộng và các điểm rủi ro?
- Các nền tảng engineering như RESTful API, naming, refactoring và unit testing ảnh hưởng thế nào đến chi phí bảo trì lâu dài?
- Design Pattern không phải là học thuộc sơ đồ UML, mà là hiểu cách đóng gói các điểm thay đổi thường gặp.
- Spring, MyBatis và Netty lần lượt giải quyết những độ phức tạp engineering nào?
- Authentication và authorization, JWT, SSO, permission system và data security lần lượt bao phủ những phần nào trong security flow?
- Scheduled tasks và real-time message push có những vấn đề nào về lựa chọn công nghệ và tính ổn định trong production?

## Thứ tự đọc đề xuất

1. [Tổng hợp câu hỏi phỏng vấn System Design thường gặp](./system-design-questions.md): trước tiên xây dựng danh sách các bài toán tình huống và vấn đề thường gặp về System Design.
2. [Chuyên đề nền tảng System Design](./basis/): bổ sung RESTful API, software engineering, code naming, code refactoring và unit testing.
3. [Tổng hợp câu hỏi phỏng vấn Design Pattern thường gặp](./design-pattern.md): hiểu các Design Pattern thường gặp phù hợp để giải quyết những điểm thay đổi nào.
4. [Chuyên đề Spring & Spring Boot](./framework/spring/): nắm được IoC, AOP, transaction, auto-configuration, các annotation thường dùng và Design Pattern của framework.
5. [Chuyên đề authentication, authorization và data security](./security/): hiểu có hệ thống về authentication và authorization, JWT, SSO, permission system, encryption, data masking và data validation.
6. Tùy theo nhu cầu dự án, đọc thêm [Giải thích chi tiết Java scheduled tasks](./schedule-task.md) và [Giải thích chi tiết Web real-time message push](./web-real-time-message-push.md).

## Bài viết cốt lõi

### System Design và nền tảng engineering

- [Tổng hợp câu hỏi phỏng vấn System Design thường gặp](./system-design-questions.md): bao quát các bài toán tình huống System Design như short URL system, flash sale system và xử lý dữ liệu quy mô lớn.
- [Chuyên đề nền tảng System Design](./basis/): trình bày từ RESTful API, software engineering đến code naming, code refactoring và unit testing.
- [Hướng dẫn ngắn về RestFul API](./basis/RESTfulAPI.md): tìm hiểu resource modeling, HTTP method, status code và quy chuẩn thiết kế interface.
- [Hướng dẫn code refactoring](./basis/refactoring.md): tìm hiểu code smell, nguyên tắc refactoring và các phương pháp refactoring thường gặp.
- [Unit testing thực chất là gì? Nên thực hiện thế nào?](./basis/unit-test.md): tìm hiểu unit testing, Mock, Stub, test pyramid và nền tảng JUnit.
- [Tổng hợp câu hỏi phỏng vấn Design Pattern thường gặp](./design-pattern.md): hệ thống hóa các Design Pattern thường gặp như Singleton, Factory, Proxy, Chain of Responsibility, Strategy và Observer.

### Framework phổ biến

- [Chuyên đề Spring & Spring Boot](./framework/spring/): kết nối các vấn đề thường gặp của Spring, Spring MVC và Spring Boot.
- [Tổng hợp câu hỏi phỏng vấn Spring thường gặp](./framework/spring/spring-knowledge-and-questions-summary.md): bao quát các vấn đề cốt lõi như IoC, AOP, vòng đời Bean và dependency injection.
- [Tổng hợp câu hỏi phỏng vấn SpringBoot thường gặp](./framework/spring/springboot-knowledge-and-questions-summary.md): bao quát các vấn đề như auto-configuration, Starter, tải configuration và Actuator.
- [Tổng hợp câu hỏi phỏng vấn MyBatis thường gặp](./framework/mybatis/mybatis-interview.md): tìm hiểu `#{}`, `${}`, dynamic SQL, cache, pagination plugin và Mapper mapping.
- [Tổng hợp câu hỏi phỏng vấn Netty thường gặp](./framework/netty.md): tìm hiểu high-performance network programming, Reactor model, event loop và ChannelPipeline.

### Authentication, authorization và data security

- [Chuyên đề authentication, authorization và data security](./security/): tập trung vào authentication và authorization, JWT, SSO, permission system, encryption, data masking và data validation.
- [Giải thích chi tiết các khái niệm nền tảng về authentication và authorization](./security/basis-of-authority-certification.md): tìm hiểu các khái niệm nền tảng như authentication, authorization, Session, Token và OAuth2.
- [Giải thích chi tiết các khái niệm nền tảng về JWT](./security/jwt-intro.md): tìm hiểu thành phần, chữ ký, quy trình hoạt động và các tình huống login authentication của JWT.
- [Giải thích chi tiết thiết kế permission system](./security/design-of-authority-system.md): tìm hiểu RBAC permission model và thiết kế permission system.
- [Tổng hợp các encryption algorithm thường gặp](./security/encryption-algorithms.md): tìm hiểu symmetric encryption, asymmetric encryption, hash algorithm và các tình huống sử dụng thường gặp.
- [Tổng hợp các phương án data masking](./security/data-desensitization.md): tìm hiểu các quy tắc data masking phổ biến và cách triển khai engineering.

### Task scheduling và message push

- [Giải thích chi tiết Java scheduled tasks](./schedule-task.md): hệ thống hóa Timer, ScheduledThreadPoolExecutor, DelayQueue, time wheel, Spring @Scheduled và distributed task scheduling framework.
- [Giải thích chi tiết Web real-time message push](./web-real-time-message-push.md): tìm hiểu short polling, long polling, SSE, WebSocket và lựa chọn phương án message push.
- [Kiến thức nền tảng J2EE](./J2EE基础知识.md): ôn tập các nền tảng Java Web như Servlet, request forwarding và redirect, Session và Cookie.

## Câu hỏi thường gặp

- Nên trả lời câu hỏi System Design theo cấu trúc nào?
- Trong thiết kế RESTful API, nên sử dụng resource path, HTTP method và status code thế nào?
- Vì sao code naming, refactoring và unit testing ảnh hưởng đến khả năng bảo trì của hệ thống?
- Các Design Pattern thường gặp lần lượt giải quyết vấn đề gì? Chúng được ứng dụng thế nào trong Spring?
- Nên hiểu Spring IoC, AOP, vòng đời Bean và cơ chế transaction thế nào?
- Quy trình cốt lõi của auto-configuration trong Spring Boot là gì?
- `#{}` và `${}` của MyBatis khác nhau thế nào? Cache cấp một và cache cấp hai hoạt động ra sao?
- Authentication và authorization khác nhau thế nào? Session, Token, JWT và OAuth2 lần lượt phù hợp với tình huống nào?
- Nên thiết kế RBAC permission system thế nào?
- Data masking, data validation và encryption algorithm lần lượt giải quyết vấn đề security nào?
- Nên lựa chọn scheduled task framework thế nào? Cần lưu ý những vấn đề nào khi scheduling task phân tán?
- Web real-time message push có những phương án nào? Nên cân nhắc short polling, SSE và WebSocket ra sao?

## Chuyên đề liên quan

- [Hệ thống kiến thức về high-performance](../high-performance/)
- [Hệ thống kiến thức về high-availability](../high-availability/)
- [Hệ thống kiến thức về distributed system](../distributed-system/)
- [Database](../database/)
- [Computer network](../cs-basics/network/other-network-questions.md)

<!-- @include: @article-footer.snippet.md -->
