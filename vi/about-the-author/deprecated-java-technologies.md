---
title: Các công nghệ Java đã lỗi thời, đừng học nữa!
description: Tổng hợp các công nghệ Java đã lỗi thời, không khuyến nghị học JSP, Struts, EJB, Java Applets, SOAP và các công nghệ quá cũ khác, kèm đề xuất giải pháp thay thế hiện đại.
category: Về tác giả
tag:
  - Tản mạn
---

Vài ngày trước, tôi tiện tay trả lời một câu hỏi trên Zhihu: “Học Java đến JSP thì không học tiếp được nữa, phải làm sao?”.

Với tâm lý không muốn người khác đi đường vòng, tôi trả lời rằng: công nghệ đã bị loại bỏ thì đừng học nữa, đồng thời liệt kê một số công nghệ đã bị loại bỏ trong lĩnh vực phát triển Java.

## Các công nghệ Java đã bị loại bỏ

Nội dung nguyên bản câu trả lời của tôi như sau, liệt kê một số công nghệ đã bị loại bỏ trong lĩnh vực phát triển Java:

**JSP**

- **Lý do**: JSP đã lỗi thời, không thể đáp ứng nhu cầu phát triển Web hiện đại; frontend và backend tách rời đã trở thành xu hướng chủ đạo.
- **Giải pháp thay thế**: Template engine (như Thymeleaf, Freemarker) phổ biến hơn trong phát triển full-stack truyền thống; còn trong kiến trúc frontend và backend tách rời, các framework frontend hiện đại như React, Vue, Angular đã thay thế vai trò của JSP.
- **Lưu ý**: Một số dự án cũ của doanh nghiệp nhà nước và doanh nghiệp nhà nước trung ương có thể vẫn đang sử dụng JSP, nhưng tình trạng này ngày càng ít gặp.

**Struts (đặc biệt là 1.x)**

- **Lý do**: Cấu hình phức tạp, hiệu suất phát triển thấp và tồn tại các lỗ hổng bảo mật nghiêm trọng (như lỗ hổng Apache Struts 2 nổi tiếng thế giới). Ngoài ra, việc bảo trì từ cộng đồng không đủ, hệ sinh thái dần thu hẹp.
- **Giải pháp thay thế**: Spring MVC và Spring WebFlux mang lại trải nghiệm phát triển đơn giản hơn, tính năng mạnh hơn cùng sự hỗ trợ hoàn thiện từ cộng đồng, đã hoàn toàn thay thế Struts.

**EJB (Enterprise JavaBeans)**

- **Lý do**: EJB quá phức tạp, chi phí phát triển cao, đường cong học tập dốc và dần bị các framework nhẹ hơn thay thế trong các dự án thực tế.
- **Giải pháp thay thế**: Spring/Spring Boot cung cấp giải pháp phát triển cấp doanh nghiệp đơn giản hơn và mạnh mẽ hơn, gần như đã trở thành tiêu chuẩn thực tế cho phát triển doanh nghiệp Java. Ngoài ra, các framework như Solon của Trung Quốc và Quarkus thân thiện với cloud native cũng rất tốt.

**Java Applets**

- **Lý do**: Các trình duyệt hiện đại (như Chrome, Firefox, Edge) từ lâu đã loại bỏ hoàn toàn hỗ trợ cho Java Applets, đồng thời Applets có các vấn đề bảo mật nghiêm trọng.
- **Giải pháp thay thế**: HTML5, WebAssembly và các framework JavaScript hiện đại (như React, Vue) có thể tạo ra trải nghiệm tương tác an toàn và hiệu quả hơn mà không cần hỗ trợ plugin.

**SOAP / JAX-WS**

- **Lý do**: SOAP và JAX-WS quá phức tạp, format dữ liệu dài dòng (XML), không thân thiện với hiệu suất và hiệu quả phát triển.
- **Giải pháp thay thế**: RESTful API và RPC nhẹ hơn, hiệu quả hơn, là lựa chọn ưu tiên trong kiến trúc microservice hiện đại.

**RMI (Remote Method Invocation)**

- **Lý do**: RMI là một công nghệ gọi từ xa Java thời kỳ đầu, nhưng khả năng tương thích kém, cấu hình phức tạp và hiệu suất thấp.
- **Giải pháp thay thế**: RESTful API và PRC cung cấp giải pháp gọi từ xa đơn giản và hiệu quả hơn, hoàn toàn thay thế RMI.

**Swing / JavaFX**

- **Lý do**: Tỷ trọng của ứng dụng desktop trong lĩnh vực phát triển đã giảm mạnh, Web và mobile đã trở thành xu hướng chủ đạo. Hệ sinh thái của Swing và JavaFX không phong phú bằng các framework đa nền tảng hiện đại.
- **Giải pháp thay thế**: Các framework phát triển desktop đa nền tảng (như Flutter Desktop, Electron) mang lại trải nghiệm hiện đại hơn.
- **Lưu ý**: Một số dự án cũ của doanh nghiệp nhà nước và doanh nghiệp nhà nước trung ương có thể vẫn đang sử dụng Swing / JavaFX, nhưng tình trạng này ngày càng ít gặp.

**Ant**

- **Lý do**: Ant là một công cụ build dựa trên cấu hình XML, khó sử dụng và cấu hình phức tạp.
- **Giải pháp thay thế**: Maven và Gradle cung cấp khả năng quản lý dependency và build project hiệu quả hơn, trở thành lựa chọn ưu tiên của các công cụ build hiện đại.

## Phát ngôn của những kẻ soi mói

Không ngờ phần bình luận quả nhiên xuất hiện một kiểu kẻ soi mói rất phổ biến:

> “Thứ học không phải là công nghệ, mà là tư tưởng. Vậy bò cũng là một công nghệ con người không cần sao? Tại sao vừa sinh ra bạn phải học bò trước? Nếu ngay cả tư tưởng nền tảng cũng không biết mà đã đi học đủ loại framework, cuối cùng chỉ có thể là một kẻ vô dụng chỉ biết CV!”

<img src="https://oss.javaguide.cn/github/javaguide/about-the-author/prattle/deprecated-java-technologies-zhihu-comments.png" style="zoom:50%;" />

Câu này thoạt nhìn có vẻ có lý, nhưng thực tế lại bộc lộ **sự thiếu hiểu biết và cố chấp** của một người.

**Người càng thiếu kiến thức càng tin vào mọi thứ một cách tuyệt đối**, vì họ chưa từng nghiêm túc tìm hiểu những góc nhìn đối lập với quan điểm của mình, đồng thời cũng thiếu nhận thức tổng thể về sự phát triển của công nghệ.

Lấy một ví dụ, khi mới bắt đầu học phát triển backend Java, tôi hoàn toàn không có kinh nghiệm, nên tùy tiện mua một cuốn sách để đọc. Khi đó tôi đọc cuốn **《Java Web tích hợp phát triển: Vua trở lại》** (nơi giấc mơ bắt đầu).

Khi còn học đại học, nhiều nội dung trong cuốn sách này thực ra đã lỗi thời, chẳng hạn sách dành rất nhiều trang để giới thiệu các công nghệ JSP, Struts, Hibernate, EJB và SVN. Tuy vậy, đến tận bây giờ tôi vẫn vô cùng biết ơn cuốn sách này vì đã đưa tôi bước vào cánh cửa phát triển backend Java.

![](https://oss.javaguide.cn/github/javaguide/about-the-author/prattle/java-web-integration-development-king-returns.png)

Cuốn sách này có tổng cộng **1010** trang. Khi đó có thể nói tôi đã học đến mức quên ăn quên ngủ, mất rất nhiều thời gian mới hoàn toàn “nghiền ngẫm” hết cả cuốn sách.

Nhìn lại, nếu khi đó tôi có ý thức tránh học những công nghệ đã bị loại bỏ này, tôi thực sự có thể tiết kiệm rất nhiều thời gian để học những nội dung phổ biến và thiết thực hơn.

Vậy những công nghệ đã bị loại bỏ này có hữu ích không? Nói thật, **chẳng có tác dụng gì, hoàn toàn lãng phí thời gian**.

**Đã phải dành thời gian học, tại sao không học những công nghệ phổ biến hơn và có giá trị thực tế hơn?**

Hiện nay vốn đã có sự cạnh tranh rất gay gắt, dù theo hướng Java hay hướng công nghệ khác, số công nghệ cần học đều rất nhiều.

Nếu muốn hiểu cái gọi là “tư tưởng tầng dưới”, thay vì lãng phí thời gian vào một công nghệ đã không còn giá trị ứng dụng thực tế như JSP, bạn nên học sâu hơn về Servlet, nghiên cứu nguyên lý AOP và IoC của Spring, đồng thời tìm hiểu cơ chế hoạt động của Spring MVC từ góc độ source code.

Những nội dung này không chỉ giúp bạn nắm được các tư tưởng cốt lõi mà còn thực sự phát huy tác dụng trong phát triển thực tế. Chẳng phải như vậy có ý nghĩa hơn việc dành rất nhiều thời gian cho JSP sao?

## Có công ty còn sử dụng thì có cần học không?

Sau khi đăng những quan điểm liên quan đến bài viết này trên [tài khoản công khai](https://mp.weixin.qq.com/s/lf2dXHcrUSU1pn28Ercj0w) của mình, tôi lại nhận được một kiểu phát ngôn khác mà theo tôi là cực kỳ ngu ngốc:

- “JSP tuy rất cũ rồi nhưng vẫn phải học một chút, biết dùng là được, vì nhiều dự án cũ của chúng tôi vẫn còn sử dụng.”
- “Nhiều dự án cũ của doanh nghiệp nhà nước trung ương và doanh nghiệp nhà nước vẫn còn sử dụng, chắc chắn phải học chứ!”

Quan điểm này hoàn toàn là soi mói câu chữ! Nếu theo logic đó, bạn còn cần học Struts2, SVN, JavaFX và các công nghệ lỗi thời khác vì vẫn có công ty sử dụng chúng. Một người bạn đại học của tôi sau khi tốt nghiệp vào làm tại một doanh nghiệp nhà nước ở Vũ Hán, viết JavaFX một năm rồi không chịu nổi và nghỉ việc. Trước đó anh ấy chưa từng tiếp xúc với JavaFX, khi tuyển dụng cũng không bị hỏi về công nghệ này.

Nhất định đừng cho rằng bạn sẽ phải đối mặt với một dự án dùng technology stack lỗi thời. Khi tìm việc, chắc chắn bạn nên tìm bằng technology stack phổ biến, đồng thời cố gắng tìm công việc giúp kỹ năng phát triển và khiến bạn làm việc thoải mái hơn. Nếu thực sự không tìm được công việc phù hợp mà phải bảo trì dự án cũ, đó là chuyện sau này, lúc ấy học đến đâu dùng đến đó cũng được.

**Người mới bắt đầu đã được khuyên mà vẫn cố học công nghệ bị loại bỏ thì đầu óc quả thực có vấn đề, về cơ bản có thể từ bỏ ngành này rồi!**
