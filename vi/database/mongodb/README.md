---
title: "Chuyên đề MongoDB: document model, index, replica set, sharding, transaction và các câu hỏi phỏng vấn thường gặp"
description: Lộ trình học MongoDB, phỏng vấn và NoSQL, bao quát document model, collection, index, replica set, sharding, transaction, aggregation, read concern, write concern, storage engine và các câu hỏi phỏng vấn thường gặp.
category: Database
tag:
  - MongoDB
  - NoSQL
  - Phỏng vấn backend
sitemap:
  changefreq: weekly
  priority: 0.85
head:
  - - meta
    - name: keywords
      content: MongoDB,MongoDB interview questions,NoSQL,document database,MongoDB index,replica set,sharding,aggregation,transaction,backend interview
---

MongoDB là một document database điển hình, phù hợp với dữ liệu bán cấu trúc, mô hình field linh hoạt và các trường hợp sử dụng cần lặp nhanh. Khi học MongoDB, nên tập trung hiểu document model, index, replica set, sharding, aggregation và khả năng transaction, thay vì chỉ xem nó đơn giản là “database có thể lưu JSON”.

## Dành cho ai

- Backend developer muốn tìm hiểu các khái niệm cốt lõi của MongoDB và document database.
- Người chuẩn bị các câu hỏi phỏng vấn liên quan đến MongoDB, NoSQL và document model.
- Kỹ sư cần lựa chọn giữa relational database và document database.
- Người đã tiếp xúc với MongoDB nhưng chưa nắm vững index, replica set, sharding và cơ chế transaction.

## Trọng tâm học

- Database, collection, document của MongoDB khác gì với database, table, row trong relational database?
- Document model phù hợp với những cấu trúc dữ liệu nào, khi nào không nên dùng MongoDB?
- Index, aggregation pipeline, replica set và sharding của MongoDB lần lượt giải quyết vấn đề gì?
- Nên hiểu read concern, write concern, transaction và semantics về consistency như thế nào?
- Trong phỏng vấn, trả lời câu hỏi về MongoDB từ các khía cạnh “model, query, index, high availability, scalability, use case” như thế nào?

## Thứ tự đọc đề xuất

1. [Tổng hợp câu hỏi phỏng vấn NoSQL cơ bản](../nosql.md): trước tiên tìm hiểu phân loại NoSQL, trường hợp sử dụng và khác biệt so với relational database.
2. [Tổng hợp câu hỏi phỏng vấn MongoDB (phần 1)](./mongodb-questions-01.md): tìm hiểu khái niệm cơ bản, document model, index và query của MongoDB.
3. [Tổng hợp câu hỏi phỏng vấn MongoDB (phần 2)](./mongodb-questions-02.md): tiếp tục tìm hiểu replica set, sharding, transaction, aggregation và thực tiễn production.
4. Sau đó quay lại [Hệ thống kiến thức Database](../), đặt MongoDB cạnh MySQL, Redis và Elasticsearch để so sánh vị trí của chúng.

## Bài viết cốt lõi

- [Tổng hợp câu hỏi phỏng vấn MongoDB (phần 1)](./mongodb-questions-01.md): bao quát khái niệm cơ bản, document model, collection, query, index và các vấn đề sử dụng thường gặp của MongoDB.
- [Tổng hợp câu hỏi phỏng vấn MongoDB (phần 2)](./mongodb-questions-02.md): bao quát replica set, sharding, transaction, aggregation, read/write concern, storage engine và thực tiễn production.
- [Tổng hợp câu hỏi phỏng vấn NoSQL cơ bản](../nosql.md): giúp hiểu vị trí của MongoDB trong hệ sinh thái NoSQL và sự khác biệt với key-value database, column-family database và graph database.

## Câu hỏi thường gặp

- MongoDB khác MySQL như thế nào?
- Document model của MongoDB phù hợp với những trường hợp sử dụng nào?
- MongoDB có những loại index nào? Thiết kế index như thế nào?
- Replica set của MongoDB đảm bảo high availability như thế nào?
- Sharding của MongoDB giải quyết vấn đề gì? Chọn shard key như thế nào?
- MongoDB có hỗ trợ transaction không? Có phù hợp với các trường hợp sử dụng transaction phức tạp không?
- Aggregation pipeline của MongoDB phù hợp để giải quyết những vấn đề nào?
- Read concern và write concern lần lượt kiểm soát điều gì?
- Những trường hợp nào không phù hợp để sử dụng MongoDB?
- MongoDB, Redis và Elasticsearch khác nhau về vị trí như thế nào?

## Chuyên đề liên quan

- [Hệ thống kiến thức Database](../)
- [Tổng hợp câu hỏi phỏng vấn NoSQL cơ bản](../nosql.md)
- [Chuyên đề MySQL](../mysql/)
- [Chuyên đề Redis](../redis/)

<!-- @include: @article-footer.snippet.md -->
