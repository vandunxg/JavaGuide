---
title: Sách kinh điển về database nên đọc
description: Gợi ý sách database, các sách kinh điển về MySQL, PostgreSQL, Redis và những database khác, bao quát tutorial nhập môn, phân tích nguyên lý, tối ưu performance và các nội dung khác.
category: Sách máy tính
icon: "mdi:database-outline"
head:
  - - meta
    - name: keywords
      content: Tuyển chọn sách database
---

## Database cơ bản

Nếu thấy sách về database khá khô khan và khó tự duy trì việc học, trước tiên bạn có thể xem một số video hay. [《Nguyên lý hệ thống database》](https://www.icourse163.org/course/BNU-1002842007) của Đại học Sư phạm Bắc Kinh và [《Hệ thống database (phần 2): Quản lý và kỹ thuật》](https://www.icourse163.org/course/HIT-1001578001) của Đại học Công nghiệp Cáp Nhĩ Tân đều rất tốt.

Giảng viên của khóa học [《Nguyên lý hệ thống database》](https://www.icourse163.org/course/BNU-1002842007) giảng rất chi tiết. Bài tập ở mỗi phần cũng bám sát kiến thức đã học, sau đó còn có nhiều bài thực hành đi kèm.

![](https://oss.javaguide.cn/github/javaguide/books/up-e113c726a41874ef5fb19f7ac14e38e16ce.png)

Nếu thích thực hành hơn và không hứng thú với kiến thức lý thuyết, bạn nên xem [《Cách phát triển một database đơn giản》](https://cstack.github.io/db_tutorial/). Project này sẽ hướng dẫn từng bước cách viết một database đơn giản.

![](https://oss.javaguide.cn/github/javaguide/books/up-11de8cb239aa7201cc8d78fa28928b9ec7d.png)

Trên GitHub cũng đã có người dùng Java triển khai một database đơn giản, phần giới thiệu khá chi tiết. Nếu quan tâm, bạn có thể xem tại đây: [https://github.com/alchemystar/Freedom](https://github.com/alchemystar/Freedom).

Ngoài project viết bằng Java này, project **[db_tutorial](https://github.com/cstack/db_tutorial)** được một tác giả nước ngoài viết bằng ngôn ngữ C, bạn cũng có thể xem qua.

**Chỉ cần tận dụng tốt search engine, bạn có thể tìm thấy toy database được triển khai bằng đủ loại ngôn ngữ.**

![](https://oss.javaguide.cn/github/javaguide/books/up-d32d853f847633ac7ed0efdecf56be1f1d2.png)

**Học trên giấy mãi vẫn thấy nông, muốn hiểu rõ việc này thì phải tự thực hành! Đặc biệt khuyến nghị các bạn học ngành CS nhất định phải thực hành thật nhiều!!!**

### 《Khái niệm hệ thống database》

Cuốn [《Khái niệm hệ thống database》](https://book.douban.com/subject/10548379/) bao quát toàn bộ các khái niệm về hệ thống database, có hệ thống kiến thức rõ ràng và là giáo trình kinh điển để học hệ thống database! Đây không phải sách tham khảo!

![](https://oss.javaguide.cn/github/javaguide/booksimage-20220409150441742.png)

### 《Triển khai hệ thống database》

Nếu cũng muốn nghiên cứu nguyên lý bên trong MySQL, bạn có thể đọc trước [《Triển khai hệ thống database》](https://book.douban.com/subject/4838430/).

![](https://oss.javaguide.cn/github/javaguide/books/database-system-implementation.png)

Dù là MySQL hay Oracle, kiến trúc tổng thể của chúng khá giống nhau. Điểm khác biệt nằm ở cách triển khai bên trong, chẳng hạn như cấu trúc dữ liệu của index database, cách triển khai storage engine, v.v.

Một số đoạn trong cuốn sách này được dịch khá gượng. Nếu có khả năng đọc bản tiếng Anh thì vẫn nên bắt đầu với bản tiếng Anh.

《Triển khai hệ thống database》 là giáo trình của Stanford. Ngoài ra còn có [《Giáo trình nhập môn hệ thống database》](https://book.douban.com/subject/3923575/) là khóa học nền tảng, có thể giúp bạn nhập môn database.

## MySQL

Dữ liệu trên website hoặc APP đều cần được lưu trữ bằng database.

Trong phát triển project của doanh nghiệp, MySQL được sử dụng khá phổ biến. Nếu muốn học MySQL, bạn có thể đọc 3 cuốn sách dưới đây:

- **[《MySQL cần biết và thành thạo》](https://book.douban.com/subject/3354490/)**: Rất mỏng! Rất phù hợp cho người mới học MySQL, là giáo trình nhập môn rất tốt.
- **[《MySQL hiệu năng cao》](https://book.douban.com/subject/23008813/)**: Tác phẩm kinh điển trong lĩnh vực MySQL! Học MySQL nhất định phải đọc! Đây là nội dung nâng cao, chủ yếu hướng dẫn cách sử dụng MySQL tốt hơn. Sách có cả lý thuyết và thực hành! Nếu không có thời gian đọc hết, tôi khuyên bạn nhất định phải đọc kỹ chương 5 (Tạo index hiệu năng cao) và chương 6 (Tối ưu performance truy vấn).
- **[《Nội tình kỹ thuật MySQL》](https://book.douban.com/subject/24708143/)**: Nếu muốn hiểu sâu về storage engine của MySQL, đọc cuốn này chắc chắn không sai!

![](https://oss.javaguide.cn/github/javaguide/books/up-3d31e762933f9e50cc7170b2ebd8433917b.png)

Về video, bạn có thể xem [《Video tutorial database MySQL》](https://www.bilibili.com/video/BV1fx411X7BD) của Động lực Nút. Video này về cơ bản đã giới thiệu hết một số kiến thức nhập môn liên quan đến MySQL.

Ngoài ra, đặc biệt đề xuất cuốn **[《MySQL vận hành như thế nào》](https://book.douban.com/subject/35231266/)**. Nội dung rất phù hợp để chuẩn bị phỏng vấn. Sách giải thích rất chi tiết nhưng không khô khan, nội dung cũng rất chất lượng!

![](https://oss.javaguide.cn/github/javaguide/csdn/20210703120643370.png)

## PostgreSQL

Giống MySQL, PostgreSQL cũng là một relational database mã nguồn mở, miễn phí và mạnh mẽ. Slogan của PostgreSQL là “**relational database mã nguồn mở tiên tiến nhất thế giới**”.

![](https://oss.javaguide.cn/github/javaguide/books/image-20220702144954370.png)

Trong vài năm gần đây, do nhiều tính năng mới của PostgreSQL quá xuất sắc, ngày càng có nhiều project sử dụng PostgreSQL thay cho MySQL.

Nếu vẫn đang phân vân có nên thử PostgreSQL hay không, bạn nên xem chủ đề Zhihu này: [PostgreSQL so với MySQL, ưu thế nằm ở đâu? - Zhihu](https://www.zhihu.com/question/20010554).

### 《PostgreSQL Guide: Khám phá nội tình》

Cuốn [《PostgreSQL Guide: Khám phá nội tình》](https://book.douban.com/subject/33477094/) chủ yếu giới thiệu nguyên lý hoạt động bên trong PostgreSQL, bao gồm tổ chức logic và triển khai vật lý của các database object, cũng như kiến trúc process và memory.

Khi mới đi làm, tôi từng cần dùng PostgreSQL và đã đọc khoảng 1/3 nội dung. Cảm giác khá ổn.

![](https://oss.javaguide.cn/github/javaguide/books/PostgreSQL-Guide.png)

### 《Nội tình kỹ thuật PostgreSQL: Khám phá chuyên sâu về tối ưu truy vấn》

Cuốn [《Nội tình kỹ thuật PostgreSQL: Khám phá chuyên sâu về tối ưu truy vấn》](https://book.douban.com/subject/30256561/) chủ yếu trình bày chi tiết triển khai một số kỹ thuật của PostgreSQL trong tối ưu truy vấn, giúp bạn hiểu sâu hơn về query optimizer của PostgreSQL.

![《Nội tình kỹ thuật PostgreSQL: Khám phá chuyên sâu về tối ưu truy vấn》](https://oss.javaguide.cn/github/javaguide/books/PostgreSQL-TechnologyInsider.png)

## Redis

**Redis là một database được phát triển bằng ngôn ngữ C**, nhưng khác với database truyền thống ở chỗ **dữ liệu của Redis nằm trong memory**, tức là đây là một in-memory database nên tốc độ đọc ghi rất nhanh. Vì vậy Redis được sử dụng rộng rãi cho cache.

Nếu muốn học Redis, bạn nhất định nên đọc hai cuốn sách dưới đây:

- [《Thiết kế và triển khai Redis》](https://book.douban.com/subject/25900156/): Chủ yếu là nội dung về lý thuyết Redis, khá toàn diện. Trước đây tôi từng viết một bài [《7 năm trước, 24 tuổi, xuất bản một cuốn sách thần về Redis》](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247507030&idx=1&sn=0a5fd669413991b30163ab6f5834a4ad&chksm=cea1939df9d61a8b93925fae92f4cee0838c449534e60731cfaf533369831192e296780b32a6&token=709354671&lang=zh_CN&scene=21#wechat_redirect) để giới thiệu cuốn sách này.
- [《Nguyên lý cốt lõi và thực tiễn Redis》](https://book.douban.com/subject/26612779/): Chủ yếu phân tích các điểm kiến thức quan trọng của Redis dựa trên source code, chẳng hạn như nhiều data structure và tính năng nâng cao.

![《Thiết kế và triển khai Redis》 và 《Thiết kế và triển khai Redis》](https://oss.javaguide.cn/github/javaguide/books/redis-books.png)

Ngoài ra, cuốn [《Phát triển và vận hành Redis》](https://book.douban.com/subject/26971561/) cũng rất tốt, vừa có giới thiệu cơ bản vừa có chia sẻ kinh nghiệm phát triển và vận hành thực tế.

![《Phát triển và vận hành Redis》](https://oss.javaguide.cn/github/javaguide/books/redis-kaifa-yu-yunwei.png)
