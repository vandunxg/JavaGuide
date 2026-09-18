---
title: Lập trình viên học công nghệ mới nhanh như thế nào
description: "Lập trình viên học công nghệ mới nhanh như thế nào: tổng hợp các khái niệm then chốt, câu hỏi thường gặp và điểm thực hành xoay quanh kiến thức công nghệ và phỏng vấn, giúp bạn học hiệu quả và chuẩn bị phỏng vấn."
category: Tuyển tập bài viết kỹ thuật
tag:
  - Chiến lược nâng cấp
head:
  - - meta
    - name: keywords
      content: phương pháp học công nghệ, học công nghệ nhanh, tài liệu chính thức, phỏng vấn kỹ thuật, câu hỏi phỏng vấn dạng học thuộc, thống nhất giữa biết và làm, kỹ năng học tập
---

> **Lời giới thiệu**: Đây là một bài viết trong phần Chiến lược nâng cấp của [《Cẩm nang phỏng vấn Java》](https://javaguide.cn/zhuanlan/java-mian-shi-zhi-bei.html), chia sẻ quan điểm của tôi về cách nhanh chóng học một công nghệ mới.
>
> ![Phần Chiến lược nâng cấp của 《Cẩm nang phỏng vấn Java》](https://oss.javaguide.cn/javamianshizhibei/training-strategy-articles.png)

Nhiều khi, do yêu cầu công việc, chúng ta cần nhanh chóng học một công nghệ nào đó để áp dụng vào project. Hoặc công ty mà chúng ta muốn ứng tuyển yêu cầu một công nghệ trước đây chưa từng tiếp xúc, nên để chuẩn bị cho phỏng vấn, chúng ta cần nhanh chóng nắm vững công nghệ đó.

Là một lập trình viên hoàn toàn tự học, trong bài viết này tôi sẽ trao đổi ngắn gọn quan điểm của mình về cách nhanh chóng học một công nghệ nào đó.

Khi học bất kỳ công nghệ nào, bạn nhất định phải hiểu rõ trước công nghệ đó dùng để giải quyết vấn đề gì. Trước khi học sâu công nghệ này, bạn nhất định phải tìm hiểu nó từ góc nhìn tổng thể, suy nghĩ xem nó gồm những module nào, cung cấp những chức năng gì, so với các công nghệ cùng loại thì có ưu điểm nào.

Ví dụ, khi học Spring, thông qua tài liệu chính thức của Spring, bạn có thể biết được những cập nhật công nghệ mới nhất của Spring, Spring gồm những module nào và Spring có thể giúp bạn giải quyết vấn đề gì.

![](https://oss.javaguide.cn/github/javaguide/system-design/web-real-time-message-push/20210506110341207.png)

Ví dụ khác, khi học message queue, trước tiên tôi sẽ tìm hiểu message queue thường đóng vai trò gì trong hệ thống và giúp chúng ta giải quyết vấn đề gì. Có rất nhiều loại message queue; khi tìm hiểu cụ thể một message queue nào đó, tôi sẽ so sánh nó với các message queue mình đã học. Chẳng hạn khi học RocketMQ, trước tiên tôi sẽ so sánh nó với message queue đầu tiên mình từng học là ActiveMQ, suy nghĩ xem RocketMQ đã cải tiến những gì so với ActiveMQ, giải quyết những vấn đề nào của ActiveMQ, hai bên có những điểm tương đồng nào và khác biệt nào.

**Cách hiệu quả và nhanh nhất để học một công nghệ là kết nối công nghệ đó với những công nghệ bạn đã học trước đây, tạo thành một mạng lưới.**

Sau đó, tôi khuyên bạn trước tiên hãy xem tutorial trong tài liệu chính thức, chạy thử các Demo liên quan và làm một số project nhỏ.

Tuy nhiên, tài liệu chính thức thường bằng tiếng Anh; thông thường chỉ các project nội địa và một số ít project nước ngoài cung cấp tài liệu bằng tiếng Trung. Hơn nữa, phần giới thiệu trong tài liệu chính thức thường khá sơ lược, không phù hợp lắm để người mới bắt đầu dùng làm tài liệu học tập.

Nếu bạn không hiểu rõ tài liệu trên website chính thức, bạn cũng có thể tìm kiếm một số blog hoặc video chất lượng cao bằng các keyword liên quan. **Đừng bao giờ vừa bắt đầu đã nghĩ đến việc phải hiểu rõ nguyên lý của công nghệ này.**

Chẳng hạn khi học Spring framework, sau khi hiểu rõ vấn đề Spring framework giải quyết, tôi khuyên bạn không nên bắt đầu ngay bằng việc nghiên cứu nguyên lý hoặc source code của Spring framework, mà trước tiên hãy trải nghiệm thực tế các chức năng cốt lõi IoC (Inverse of Control: đảo ngược điều khiển) và AOP (Aspect-Oriented Programming: lập trình hướng khía cạnh) do Spring framework cung cấp, dùng Spring framework viết một số Demo, thậm chí dùng Spring framework làm một số project nhỏ.

Nói ngắn gọn, **trước khi nghiên cứu nguyên lý của công nghệ, trước tiên phải hiểu rõ cách sử dụng công nghệ đó.**

Quá trình học từng bước như vậy có thể dần giúp bạn xây dựng hứng thú học tập, đạt được cảm giác thành tựu tức thời và tránh bị nản lòng khi trực tiếp nghiên cứu kiến thức mang tính nguyên lý.

**Khi nghiên cứu nguyên lý của một công nghệ, để tránh nội dung quá trừu tượng, chúng ta cũng có thể thực hành.**

Ví dụ, khi học nguyên lý của Tomcat, chúng ta phát hiện thread pool tùy chỉnh của Tomcat khá thú vị, vậy chúng ta cũng có thể tự viết một thread pool tùy chỉnh. Tương tự, khi học nguyên lý của Dubbo, chúng ta có thể tự xây dựng một RPC framework đơn giản.

Ngoài ra, công nghệ cần dùng trong project và công nghệ cần dùng trong phỏng vấn thực ra vẫn có một số khác biệt.

Nếu bạn học một công nghệ nào đó để sử dụng trong project thực tế, trọng tâm của bạn là học cách sử dụng và best practice của công nghệ đó, tìm hiểu những vấn đề có thể gặp phải trong quá trình sử dụng. Mục tiêu cuối cùng của bạn là công nghệ này mang lại hiệu quả thực tế cho project, hơn nữa hiệu quả đó phải là tích cực.

Nếu bạn học một công nghệ nào đó chỉ để phỏng vấn, trọng tâm của bạn nên đặt vào một số câu hỏi thường gặp nhất về công nghệ đó trong phỏng vấn, tức là những câu hỏi phỏng vấn dạng học thuộc mà chúng ta thường nói đến.

Nhiều người vừa nhắc đến các câu hỏi phỏng vấn dạng học thuộc là tỏ vẻ khinh thường. Theo tôi, nếu bạn không học thuộc một cách máy móc mà suy nghĩ về bản chất của những câu hỏi phỏng vấn này, thì trong quá trình chuẩn bị các câu hỏi đó, bạn vẫn có thể hiểu sâu hơn về công nghệ này.

Cuối cùng, điều quan trọng nhất và cũng khó nhất vẫn là **thống nhất giữa biết và làm! Thống nhất giữa biết và làm! Thống nhất giữa biết và làm!** Dù là lập trình hay lĩnh vực khác, điều quan trọng nhất không phải là bạn biết bao nhiêu, mà là cố gắng thống nhất giữa điều mình biết và điều mình làm.
