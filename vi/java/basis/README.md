---
title: "Chuyên đề Java Basics: cú pháp, lập trình hướng đối tượng, generic, reflection, proxy và serialization"
description: "Lộ trình học và câu hỏi phỏng vấn Java Basics, bao quát cú pháp cơ bản, lập trình hướng đối tượng, keyword, truyền giá trị, generic, reflection, proxy, serialization, SPI, Unsafe và syntactic sugar."
category: Java
tag:
  - Java
  - Java Basics
  - Java interview
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Java Basics,Java interview questions,Java keywords,Java pass-by-value,Java generics,Java reflection,Java proxy,Java serialization,Java SPI,Java Unsafe,Java syntactic sugar
---

Java Basics là nền tảng cần có trước khi học collection, concurrency, JVM, Spring và các middleware. Phần này không chỉ để ghi nhớ cú pháp; quan trọng hơn là hiểu object model của Java, truyền tham số, type erasure của generic, lời gọi reflection, dynamic proxy, ranh giới serialization và cơ chế mở rộng của framework.

## Dành cho ai

- Người mới bắt đầu học Java có hệ thống và muốn kết nối cú pháp cơ bản với các cơ chế cốt lõi.
- Bạn đang chuẩn bị cho câu hỏi phỏng vấn Java Basics và muốn nhanh chóng bổ sung những phần còn thiếu.
- Developer đã viết dự án Java nhưng chưa hiểu sâu về các cơ chế như reflection, proxy, generic, SPI và serialization.
- Kỹ sư muốn tiếp tục học source code của collection, concurrency, JVM và Spring, nhưng cần bổ sung kiến thức nền tảng trước.

## Trọng tâm học

- Cú pháp cơ bản, lập trình hướng đối tượng, exception, các class thường dùng, keyword và chi tiết encoding của Java.
- Mối quan hệ giữa truyền giá trị, reference variable, khả năng thay đổi của object và lời gọi method.
- Generic, wildcard, type erasure và ảnh hưởng của chúng đến collection, thiết kế API và reflection.
- Các cơ chế thường gặp ở tầng dưới của framework như reflection, dynamic proxy và SPI.
- Các điểm kiến thức dễ gây lỗi trong dự án và phỏng vấn như serialization, `BigDecimal`, `Unsafe` và syntactic sugar.

## Thứ tự đọc đề xuất

1. [Tổng hợp câu hỏi phỏng vấn Java Basics thường gặp (phần 1)](./java-basic-questions-01.md): đọc qua trước về cú pháp cơ bản, lập trình hướng đối tượng và các class thường dùng của Java.
2. [Tổng hợp câu hỏi phỏng vấn Java Basics thường gặp (phần 2)](./java-basic-questions-02.md) và [Tổng hợp câu hỏi phỏng vấn Java Basics thường gặp (phần 3)](./java-basic-questions-03.md): bổ sung exception, generic, reflection, annotation và các điểm dễ nhầm thường gặp.
3. [Tổng hợp Java keyword](./java-keyword-summary.md) và [Giải thích chi tiết Java pass-by-value](./why-there-only-value-passing-in-java.md): làm rõ những hiểu lầm thường gặp về các khái niệm nền tảng.
4. [Giải thích chi tiết generic & wildcard](./generics-and-wildcards.md), [Giải thích chi tiết cơ chế Java reflection](./reflection.md), [Giải thích chi tiết Java proxy pattern](./proxy.md): hiểu các năng lực thường gặp ở tầng dưới của framework.
5. [Giải thích chi tiết Java serialization](./serialization.md), [Giải thích chi tiết cơ chế Java SPI](./spi.md), [Giải thích chi tiết lớp Unsafe đặc biệt của Java](./unsafe.md): tiếp tục bổ sung kiến thức mở rộng trong thực tiễn dự án và quá trình đọc source code.

## Bài viết cốt lõi

### Câu hỏi phỏng vấn Basics

- [Tổng hợp câu hỏi phỏng vấn Java Basics thường gặp (phần 1)](./java-basic-questions-01.md): bao quát đặc điểm ngôn ngữ Java, cú pháp cơ bản, lập trình hướng đối tượng, các class thường dùng và những điểm dễ nhầm.
- [Tổng hợp câu hỏi phỏng vấn Java Basics thường gặp (phần 2)](./java-basic-questions-02.md): tiếp tục hệ thống hóa exception, generic, reflection, annotation và các năng lực nền tảng khác.
- [Tổng hợp câu hỏi phỏng vấn Java Basics thường gặp (phần 3)](./java-basic-questions-03.md): bổ sung các câu hỏi phỏng vấn Basics thiên về chi tiết và nâng cao hơn.

### Cơ chế ngôn ngữ

- [Tổng hợp Java keyword](./java-keyword-summary.md): giải thích rõ các keyword như `final`, `static`, `this`, `super`.
- [Giải thích chi tiết Java pass-by-value](./why-there-only-value-passing-in-java.md): giải thích vì sao Java chỉ có truyền giá trị và ngữ nghĩa thực tế khi truyền reference variable làm tham số.
- [Giải thích chi tiết generic & wildcard](./generics-and-wildcards.md): tìm hiểu cú pháp generic, wildcard giới hạn trên và giới hạn dưới, type erasure cùng các trường hợp sử dụng thường gặp.
- [Giải thích chi tiết Java syntactic sugar](./syntactic-sugar.md): tìm hiểu compiler xử lý enhanced for, autoboxing và unboxing, enum, Lambda cùng các syntactic sugar khác như thế nào.

### Cơ chế tầng dưới của framework

- [Giải thích chi tiết cơ chế Java reflection](./reflection.md): tìm hiểu object `Class`, lời gọi reflection, chi phí hiệu năng và các trường hợp sử dụng.
- [Giải thích chi tiết Java proxy pattern](./proxy.md): nắm được static proxy, JDK dynamic proxy và CGLIB proxy.
- [Giải thích chi tiết cơ chế Java SPI](./spi.md): tìm hiểu service discovery và cơ chế mở rộng theo plugin.
- [Giải thích chi tiết Java serialization](./serialization.md): tìm hiểu quy trình serialization, `serialVersionUID`, rủi ro bảo mật và các phương án thay thế.

### Chi tiết thực tiễn

- [Giải thích chi tiết BigDecimal](./bigdecimal.md): nắm được các lưu ý về tính tiền, độ chính xác, rounding mode và cách khởi tạo.
- [Java nên dùng long hay BigDecimal cho số tiền?](./money-long-vs-bigdecimal.md): phân biệt trường hợp lưu trữ và tính toán số tiền, nắm được đơn vị nhỏ nhất, rounding, overflow và thiết kế field trong database.
- [Giải thích chi tiết lớp Unsafe đặc biệt của Java](./unsafe.md): tìm hiểu off-heap memory, CAS, offset của field trong object và các năng lực tầng dưới trong source code.

## Câu hỏi thường gặp

- Java truyền giá trị hay truyền tham chiếu? Vì sao field có thể bị thay đổi khi truyền object làm tham số?
- `==` và `equals()` khác nhau như thế nào? Vì sao override `equals()` bắt buộc phải override `hashCode()`?
- Nên chọn `String`, `StringBuilder` hay `StringBuffer` như thế nào?
- `final`, `static`, `this`, `super` lần lượt có tác dụng gì?
- Type erasure của generic là gì? `List<String>` và `List<Integer>` khác nhau như thế nào tại runtime?
- Vì sao reflection chậm? Có những trường hợp sử dụng điển hình nào?
- JDK dynamic proxy và CGLIB dynamic proxy khác nhau như thế nào?
- Vì sao không khuyến nghị sử dụng trực tiếp Java native serialization?
- Vì sao `BigDecimal` được khuyến nghị khởi tạo bằng string?
- Số tiền nên dùng `long` để lưu đơn vị nhỏ nhất hay dùng `BigDecimal`?

## Chuyên đề liên quan

- [Hệ thống kiến thức Java](../)
- [Chuyên đề Java collection](../collection/)
- [Chuyên đề Java concurrency](../concurrent/)
- [Chuyên đề JVM](../jvm/)
- [Spring](../../system-design/framework/spring/)

<!-- @include: @article-footer.snippet.md -->
