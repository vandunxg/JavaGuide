---
title: Giải thích chi tiết mô hình Java IO
description: "Giải thích chi tiết mô hình Java IO: phân tích chuyên sâu ba mô hình BIO blocking IO, NIO non-blocking IO, AIO asynchronous IO, cơ chế I/O multiplexing, pattern Reactor/Proactor và phân biệt các khái niệm synchronous, asynchronous, blocking, non-blocking."
category: Java
tag:
  - Java IO
  - Java Basics
head:
  - - meta
    - name: keywords
      content: Java IO,BIO,NIO,AIO,blocking IO,non-blocking IO,I/O multiplexing,Reactor pattern,Proactor pattern
---

Phần mô hình IO thực sự khá khó hiểu vì cần quá nhiều kiến thức về tầng dưới của máy tính. Tôi đã mất khá nhiều thời gian để viết bài này và rất mong có thể trình bày những gì mình biết. Hy vọng bạn sẽ thu hoạch được điều gì đó! Để viết bài này, tôi còn xem lại cuốn 《UNIX Network Programming》, khó thật đấy, trời ơi! Đau lòng~

_Năng lực cá nhân có hạn. Nếu bài viết còn điểm nào cần bổ sung/hoàn thiện/chỉnh sửa, mời bạn chỉ ra trong phần bình luận để cùng tiến bộ!_

## Lời nói đầu

I/O luôn là một nội dung khó hiểu đối với nhiều bạn, trong bài viết này tôi sẽ trình bày những gì mình hiểu về I/O, hy vọng có thể giúp ích cho bạn.

## I/O

### I/O là gì?

I/O (**I**nput/**O**utput) nghĩa là **input/output**.

**Trước tiên, hãy tìm hiểu I/O từ góc độ cấu trúc máy tính.**

Theo kiến trúc Von Neumann, cấu trúc máy tính được chia thành 5 phần lớn: bộ tính toán, bộ điều khiển, bộ nhớ, thiết bị input và thiết bị output.

![Kiến trúc Von Neumann](https://oss.javaguide.cn/github/javaguide/java/io/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9pcy1jbG91ZC5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70.jpeg)

Thiết bị input (chẳng hạn bàn phím) và thiết bị output (chẳng hạn màn hình) đều là thiết bị ngoại vi. Card mạng và ổ cứng vừa có thể là thiết bị input, vừa có thể là thiết bị output.

Thiết bị input đưa dữ liệu vào máy tính, thiết bị output nhận dữ liệu do máy tính đưa ra.

**Xét từ góc độ cấu trúc máy tính, I/O mô tả quá trình giao tiếp giữa hệ thống máy tính và các thiết bị bên ngoài.**

**Tiếp theo, hãy tìm hiểu I/O từ góc độ ứng dụng.**

Theo kiến thức về hệ điều hành đã học ở đại học: để bảo đảm tính ổn định và an toàn của hệ điều hành, không gian địa chỉ của một process được chia thành **user space** và **kernel space**.

Các application chúng ta thường chạy đều hoạt động trong user space, chỉ kernel space mới có thể thực hiện các thao tác liên quan đến tài nguyên ở system mode, chẳng hạn quản lý file, giao tiếp giữa các process, quản lý memory, v.v. Nói cách khác, muốn thực hiện thao tác IO, nhất định phải dựa vào kernel.

Ngoài ra, chương trình trong user space không thể trực tiếp truy cập kernel space.

Khi muốn thực hiện thao tác IO, do không có quyền thực hiện các thao tác này, chương trình chỉ có thể thực hiện system call để yêu cầu hệ điều hành hoàn tất.

Vì vậy, nếu user process muốn thực hiện thao tác IO thì phải thông qua **system call** để gián tiếp truy cập kernel space.

Trong quá trình phát triển hằng ngày, chúng ta tiếp xúc nhiều nhất với **disk IO (đọc và ghi file)** và **network IO (request và response qua mạng)**.

**Xét từ góc độ application, application của chúng ta thực hiện lời gọi IO (system call) đến kernel của hệ điều hành, còn kernel của hệ điều hành chịu trách nhiệm thực hiện thao tác IO cụ thể. Nói cách khác, application thực tế chỉ khởi tạo lời gọi thực hiện thao tác IO; việc thực thi IO cụ thể do kernel của hệ điều hành hoàn tất.**

Lấy thao tác đọc truyền thống làm ví dụ, sau khi application thực hiện lời gọi I/O, thường sẽ trải qua hai bước:

1. Kernel chờ thiết bị I/O chuẩn bị xong dữ liệu.
2. Kernel copy dữ liệu từ kernel space sang user space.

### Có những mô hình IO phổ biến nào?

Trong hệ thống UNIX có tổng cộng 5 mô hình IO: **synchronous blocking I/O**, **synchronous non-blocking I/O**, **I/O multiplexing**, **signal-driven I/O** và **asynchronous I/O**.

Đây cũng là 5 mô hình IO thường được nhắc đến.

## 3 mô hình IO phổ biến trong Java

### BIO (Blocking I/O)

**BIO thuộc mô hình synchronous blocking IO.**

Trong mô hình synchronous blocking IO, sau khi application gọi read, nó sẽ bị block cho đến khi kernel copy dữ liệu vào user space.

![Nguồn hình ảnh: 《Phân tích chuyên sâu Tomcat & Jetty》](https://oss.javaguide.cn/p3-juejin/6a9e704af49b4380bb686f0c96d33b81~tplv-k3u1fbpfcp-watermark.png)

Khi số lượng connection của client không cao thì không có vấn đề gì. Tuy nhiên, khi phải xử lý hàng trăm nghìn hoặc thậm chí hàng triệu connection, mô hình BIO truyền thống không thể đáp ứng. Vì vậy, chúng ta cần một mô hình xử lý I/O hiệu quả hơn để đáp ứng concurrency cao hơn.

### NIO (New I/O)

NIO trong Java được giới thiệu từ Java 1.4, tương ứng với package `java.nio`, cung cấp các abstraction như `Channel`, `Selector`, `Buffer`. Chữ N trong NIO là New. NIO đồng thời cung cấp các channel blocking và non-blocking; trong đó selectable channel (`SelectableChannel`) có thể được cấu hình ở non-blocking mode. Đây là phương thức thao tác I/O theo hướng buffer và dựa trên channel.

Trong Java NIO, cách lập trình mạng dựa trên `SelectableChannel` ở chế độ non-blocking và `Selector` tương ứng với **mô hình I/O multiplexing**. Tuy nhiên, không thể đồng nhất toàn bộ package `java.nio` với I/O multiplexing, vì package này còn bao gồm file I/O, blocking channel và các API khác.

Hãy tiếp tục theo dõi mạch giải thích, bạn sẽ tìm được câu trả lời!

Trước tiên, hãy xem **mô hình synchronous non-blocking IO**.

![Nguồn hình ảnh: 《Phân tích chuyên sâu Tomcat & Jetty》](https://oss.javaguide.cn/p3-juejin/bb174e22dbe04bb79fe3fc126aed0c61~tplv-k3u1fbpfcp-watermark.png)

Trong mô hình synchronous non-blocking IO, application liên tục thực hiện lời gọi read. Trong thời gian chờ dữ liệu được copy từ kernel space sang user space, thread vẫn bị block cho đến khi kernel copy dữ liệu vào user space.

So với mô hình synchronous blocking IO, mô hình synchronous non-blocking IO thực sự đã được cải thiện đáng kể. Thông qua polling, nó tránh việc bị block liên tục.

> Với synchronous non-blocking IO, khi thực hiện một lời gọi read, nếu dữ liệu chưa sẵn sàng thì application không cần block để chờ mà có thể chuyển sang thực hiện một số tác vụ tính toán nhỏ, sau đó nhanh chóng quay lại tiếp tục thực hiện lời gọi read, tức là polling. Việc
> polling này không diễn ra liên tục mà có khoảng nghỉ; tận dụng khoảng nghỉ đó chính là điểm khiến synchronous non-blocking IO hiệu quả hơn synchronous blocking IO.

Tuy nhiên, mô hình IO này cũng tồn tại vấn đề: **việc application liên tục thực hiện system call I/O để polling xem dữ liệu đã sẵn sàng hay chưa tiêu tốn rất nhiều tài nguyên CPU.**

Lúc này, **mô hình I/O multiplexing** xuất hiện.

![](https://oss.javaguide.cn/github/javaguide/java/io/88ff862764024c3b8567367df11df6ab~tplv-k3u1fbpfcp-watermark.png)

Trong mô hình I/O multiplexing, thread trước tiên thực hiện lời gọi select để hỏi kernel xem dữ liệu đã sẵn sàng hay chưa. Sau khi kernel chuẩn bị xong dữ liệu, user thread mới thực hiện lời gọi read. Quá trình gọi read (dữ liệu từ kernel space -> user space) vẫn bị block.

> Các system call hỗ trợ I/O multiplexing hiện nay gồm select, epoll, v.v. Gần như mọi hệ điều hành hiện đều hỗ trợ system call select.
>
> - **Lời gọi select**: system call do kernel cung cấp, hỗ trợ truy vấn trạng thái khả dụng của nhiều system call trong một lần. Gần như mọi hệ điều hành đều hỗ trợ.
> - **Lời gọi epoll**: thuộc kernel Linux 2.6, là phiên bản cải tiến của lời gọi select, tối ưu hiệu suất thực thi I/O.

**Mô hình I/O multiplexing giảm mức tiêu thụ tài nguyên CPU bằng cách giảm các system call không hiệu quả.**

Trong NIO của Java có một khái niệm rất quan trọng là **selector (Selector)**, còn có thể gọi là **multiplexer**. Nhờ đó, chỉ cần một thread cũng có thể quản lý nhiều client connection. Chỉ khi dữ liệu từ client đến, thread mới phục vụ client đó.

![Mối quan hệ giữa Buffer, Channel và Selector](https://oss.javaguide.cn/github/javaguide/java/nio/channel-buffer-selector.png)

### AIO (Asynchronous I/O)

AIO thường chỉ API asynchronous channel được NIO.2 giới thiệu trong Java 7. Ngoài asynchronous channel, NIO.2 còn bao gồm API file system mới.

Thao tác của asynchronous channel sẽ trả về ngay. Caller có thể lấy kết quả thông qua `Future`, hoặc truyền `CompletionHandler` để thực hiện callback sau khi thao tác hoàn tất.

![](https://oss.javaguide.cn/github/javaguide/java/io/3077e72a1af049559e81d18205b56fd7~tplv-k3u1fbpfcp-watermark.png)

Hiện tại, AIO vẫn chưa được ứng dụng rộng rãi. Trước đây Netty cũng từng thử sử dụng AIO, nhưng sau đó đã từ bỏ. Nguyên nhân là sau khi sử dụng AIO, hiệu suất của Netty trên hệ thống Linux không được cải thiện nhiều.

Cuối cùng là một hình minh họa tóm tắt đơn giản về BIO, NIO và AIO trong Java.

![So sánh BIO, NIO và AIO](https://oss.javaguide.cn/github/javaguide/java/nio/bio-aio-nio.png)

## Tài liệu tham khảo

- 《Phân tích chuyên sâu Tomcat & Jetty》
- Cách hoàn tất một IO: <https://llc687.top/126.html>
- Lập trình viên nên hiểu IO như thế này: [https://www.jianshu.com/p/fa7bdc4f3de7](https://www.jianshu.com/p/fa7bdc4f3de7)
- 10 phút hiểu nguyên lý tầng dưới của Java NIO: <https://www.cnblogs.com/crazymakercircle/p/10225159.html>
- Bạn biết gì về mô hình IO | Phần lý thuyết: <https://www.cnblogs.com/sheng-jie/p/how-much-you-know-about-io-models.html>
- 《UNIX Network Programming, Tập 1; Socket Networking API》 mục 6.2 Mô hình IO

<!-- @include: @article-footer.snippet.md -->
