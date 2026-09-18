---
title: Những cuốn sách kinh điển về hệ thống phân tán nên đọc
description: Gợi ý sách về hệ thống phân tán, DDIA, distributed transaction, consensus algorithm, microservice architecture và các sách kinh điển khác, giúp nắm vững kiến thức cốt lõi về thiết kế hệ thống phân tán.
category: Sách máy tính
icon: "mdi:transit-connection-variant"
---

## 《Tìm hiểu sâu về hệ thống phân tán》

![](https://oss.javaguide.cn/github/javaguide/books/deep-understanding-of-distributed-system.png)

**[《Tìm hiểu sâu về hệ thống phân tán》](https://book.douban.com/subject/35794814/)** là một cuốn sách gốc tiếng Trung về hệ thống phân tán được xuất bản năm 2022, chủ yếu trình bày các khái niệm cơ bản, thách thức thường gặp và consensus algorithm trong lĩnh vực hệ thống phân tán.

Tác giả dành nhiều nội dung để giới thiệu consensus algorithm rất quan trọng trong lĩnh vực hệ thống phân tán, đồng thời hướng dẫn bạn dùng ngôn ngữ Go để tự triển khai từ đầu Paxos, consensus algorithm khởi nguồn.

Nói thật, tôi vẫn chưa bắt đầu đọc cuốn sách này. Nhưng! Tôi gần như đã đọc kỹ từng bài viết về hệ thống phân tán trên blog của tác giả. Tác giả bắt đầu lên ý tưởng cho 《Tìm hiểu sâu về hệ thống phân tán》 từ năm 2019, bắt đầu viết vào năm 2020 và mất gần hai năm mới hoàn thành bản thảo.

![](https://oss.javaguide.cn/github/javaguide/books/image-20220706121952258.png)

Tác giả đã viết riêng một bài để giới thiệu câu chuyện phía sau cuốn sách này. Nếu quan tâm, bạn có thể tự xem: <https://zhuanlan.zhihu.com/p/487534882> .

Cuối cùng, đây là địa chỉ repository chứa code và errata của cuốn sách: <https://github.com/tangwz/DistSysDeepDive> .

## 《Thiết kế hệ thống ứng dụng hướng dữ liệu》

![](https://oss.javaguide.cn/github/javaguide/books/ddia.png)

Rất khuyến nghị **[《Thiết kế hệ thống ứng dụng hướng dữ liệu》](https://book.douban.com/subject/30329536/)** (DDIA, Designing Data-Intensive Application), rất đáng đọc nhiều lần! Gần 90% người dùng Douban đã đánh giá năm sao cho cuốn sách này sau khi đọc.

Cuốn sách chủ yếu trình bày distributed database, data partition, transaction, distributed system và các nội dung khác.

Phần lớn các khái niệm được giới thiệu trong sách có thể bạn đã từng nghe qua, nhưng sau khi đọc nội dung trong sách, có thể bạn sẽ chợt hiểu ra: “Ồ! Hóa ra là như vậy! Đây chẳng phải là nguyên lý của một công nghệ nào đó sao?”.

Trước đây tôi từng viết riêng một câu trả lời trên Zhihu để giới thiệu và đề xuất cuốn sách này. Nếu chưa đọc, bạn có thể xem: [Những cuốn sách lập trình nào khiến bạn thấy vô cùng thích thú sau khi đọc?](https://www.zhihu.com/question/50408698/answer/2278198495) . Ngoài ra, nếu bạn thấy cuốn sách này khá khó và không hiểu nhiều đoạn khi đọc, tôi đề xuất [cuốn sách đọc kỹ DDIA theo từng chương](https://ddia.qtmuniao.com) do tác giả của 《Tìm hiểu sâu về hệ thống phân tán》 viết.

## 《Tìm hiểu sâu về distributed transaction》

![](https://oss.javaguide.cn/github/javaguide/books/In-depth-understanding-of-distributed-transactions-xiaoyu.png)

**[《Tìm hiểu sâu về distributed transaction》](https://book.douban.com/subject/35626925/)** có một tác giả là nhà sáng lập gateway Apache ShenYu (incubating), đồng thời là nhà sáng lập các framework distributed transaction như Hmily, RainCat và Myth.

Khi học về distributed transaction, bạn có thể tham khảo cuốn sách này. Dù có một số lỗi nhỏ và một vài chỗ diễn đạt chưa mạch lạc, phần giới thiệu về các giải pháp distributed transaction nhìn chung vẫn khá tốt.

## 《Từ Paxos đến Zookeeper》

![](https://oss.javaguide.cn/github/javaguide/books/image-20211216161350118.png)

**[《Từ Paxos đến Zookeeper》](https://book.douban.com/subject/26292004/)** là một cuốn sách hay giúp bạn nhập môn lý thuyết hệ thống phân tán. Cuốn sách chủ yếu giới thiệu một số consistency protocol phân tán điển hình và các hướng giải quyết vấn đề consistency trong hệ thống phân tán, trong đó tập trung giải thích Paxos và protocol ZAB.

PS: Zookeeper hiện nay không được dùng nhiều, bạn không cần tập trung học, nhưng protocol Paxos và ZAB vẫn rất đáng để nghiên cứu sâu.

## 《Tìm hiểu sâu về consensus algorithm phân tán》

![](https://oss.javaguide.cn/github/javaguide/books/deep-dive-into-distributed-consensus-algorithms.png)

**[《Tìm hiểu sâu về consensus algorithm phân tán》](https://book.douban.com/subject/36335459/)** phân tích chi tiết nguyên lý cốt lõi và implementation detail của các consensus algorithm phân tán phổ biến như Paxos, Raft và Zab. Nếu muốn tìm hiểu về consensus algorithm phân tán, bạn có thể tham khảo phần tổng hợp trong cuốn sách này.

## 《Mẫu thiết kế microservice architecture》

![](https://oss.javaguide.cn/github/javaguide/books/microservices-patterns.png)

**[《Mẫu thiết kế microservice architecture》](https://book.douban.com/subject/33425123/)** có tác giả Chris Richardson, người được bình chọn là một trong mười software architect hàng đầu thế giới và là một trong những người tiên phong của microservice architecture. Cuốn sách tập hợp 44 architecture design pattern đã được kiểm chứng qua thực tiễn, dùng để giải quyết các vấn đề khó như service decomposition, transaction management, query và cross-service communication. Nội dung sách không chỉ vững về lý thuyết mà còn hướng dẫn người đọc từng bước nắm vững cách phát triển và deploy ứng dụng microservice architecture cấp production thông qua nhiều ví dụ code Java.

## 《Kiến trúc Phoenix》

![](https://oss.javaguide.cn/github/javaguide/books/f5bec14d3b404ac4b041d723153658b5.png)

**[《Kiến trúc Phoenix》](https://book.douban.com/subject/35492898/)** là bản tổng hợp kinh nghiệm nhiều năm về architecture và R&D của thầy Chu Chí Minh. Nội dung rất thực tế, có cả chiều sâu lẫn chiều rộng, kết hợp lý thuyết với thực tiễn!

Đúng như phụ đề “Xây dựng hệ thống phân tán lớn đáng tin cậy”, nội dung chính của cuốn sách là: “Làm thế nào để xây dựng một hệ thống phần mềm phân tán lớn đáng tin cậy”, bao gồm các khía cạnh sau:

- Lộ trình tiến hóa của software architecture từ monolith đến microservice rồi đến serverless.
- Những vấn đề architect cần chú ý khi thiết kế architecture và các practice tốt.
- Nền tảng của hệ thống phân tán, chẳng hạn các consensus algorithm phân tán thường gặp như Paxos và Multi Paxos.
- Immutable infrastructure như virtualized container và service mesh.
- Hướng dẫn tránh các bẫy khi tiến tới microservice.

Tôi đã đề xuất cuốn sách này nhiều lần. Xem thêm trong các bài viết trước đây:

- [Một cuốn sách cực hay khác của thầy Chu Chí Minh! Phát hiện kho báu!](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247505254&idx=1&sn=04faf3093d6002354f06fffbfc2954e0&chksm=cea19aadf9d613bbba7ed0e02ccc4a9ef3a30f4d83530e7ad319c2cc69cd1770e43d1d470046&scene=178&cur_album_id=1646812382221926401#rd)
- [Một cuốn sách cực hay khác trong lĩnh vực Java! Thầy Chu Chí Minh đúng là đỉnh!](https://mp.weixin.qq.com/s/9nbzfZGAWM9_qIMp1r6uUQ)

## Khác

- [《Hệ thống phân tán: Khái niệm và thiết kế》](https://book.douban.com/subject/21624776/): thiên về dạng giáo trình, nội dung đầy đủ nhưng không thú vị, có thể dùng làm sách tham khảo;
- [《Nguyên lý và thực tiễn của distributed architecture》](https://book.douban.com/subject/35689350/): xuất bản năm 2021, không được chú ý nhiều, tôi cũng chưa đọc.
