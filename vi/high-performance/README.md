---
title: "Knowledge system về high-performance system: CDN, load balancing, database optimization, cache và message queue"
description: "Lộ trình học về phỏng vấn và system design high-performance system, bao quát CDN, load balancing, read/write separation, database sharding, hot/cold data separation, deep pagination, SQL optimization, message queue và các MQ phổ biến."
category: High-performance
tag:
  - High-performance
  - System design
  - Backend interview
sitemap:
  changefreq: weekly
  priority: 0.95
head:
  - - meta
    - name: keywords
      content: high-performance system,high-performance system design,high-performance interview questions,CDN,load balancing,read/write separation,database sharding,hot/cold data separation,deep pagination,SQL optimization,message queue,Kafka,RocketMQ,RabbitMQ,Disruptor,backend interview
---

<!-- @include: @small-advertisement.snippet.md -->

Tài liệu **knowledge system về high-performance system** này hướng đến việc học backend, system design và ôn tập phỏng vấn; xoay quanh “giảm latency, tăng throughput, giảm peak, giảm áp lực database, tối ưu data access path” để tổng hợp các bài viết về high-performance của website này.

Nếu thời gian hạn chế, nên xem trước [Tổng hợp câu hỏi phỏng vấn về high-performance system design](./high-performance-system-interview-questions.md) để nhanh chóng xây dựng danh sách vấn đề thường gặp; nếu muốn bổ sung nền tảng có hệ thống, có thể tiến hành theo thứ tự đọc dưới đây.

Khi học phần nội dung này, không nên chỉ ghi nhớ tên các giải pháp như “thêm cache, thêm MQ, sharding database”. Tối ưu high-performance giống một bài phân tích toàn bộ chain hơn: request đi vào từ phía user, đi qua CDN, load balancing, application service, cache, database và message queue; mỗi đoạn đều có thể trở thành bottleneck. Chỉ khi giải thích rõ bottleneck nằm ở đâu, tại sao chọn giải pháp này và sẽ phát sinh vấn đề mới nào thì mới thực sự nắm vững.

## Dành cho ai

- Backend developer đang học có hệ thống về high-performance system design.
- Bạn đang chuẩn bị cho phỏng vấn backend tại trường, thị trường việc làm hoặc các công ty lớn.
- Engineer muốn bổ sung năng lực engineering về CDN, load balancing, database optimization, message queue và các nội dung khác.
- Reader đã gặp các vấn đề như slow SQL, deep pagination, hot traffic, message backlog, database pressure nhưng thiếu giải pháp có hệ thống.

## Trọng tâm học

- High-performance system thực sự đang tối ưu latency, throughput, resource utilization hay user experience?
- CDN, load balancing, cache, database optimization và message queue lần lượt giải quyết bottleneck nào trong chain?
- Read/write separation, database sharding, hot/cold data separation và deep pagination optimization lần lượt phù hợp với những scenario nào?
- Sự khác biệt về positioning và lựa chọn giữa Kafka, RocketMQ, RabbitMQ và Disruptor là gì?
- Trong phỏng vấn, làm thế nào trả lời câu hỏi high-performance theo “bottleneck identification -> solution selection -> trade-off analysis -> implementation risks”?

## Mạch trả lời phỏng vấn

Khi trả lời câu hỏi system design về high-performance, có thể triển khai theo mạch sau:

1. **Xác nhận mục tiêu trước**: QPS, RT, P99, data volume, tỷ lệ đọc/ghi, yêu cầu consistency, ràng buộc chi phí.
2. **Xác định bottleneck tiếp theo**: ingress bandwidth, application thread pool, slow SQL, lock contention, cache hit rate, MQ backlog, downstream dependencies.
3. **Sau đó chọn giải pháp**: tầng ingress dùng CDN/load balancing, tầng application dùng cache/rate limiting/asynchronous processing, tầng data dùng index/read/write separation/database sharding/hot/cold data separation, tầng san bằng peak dùng MQ.
4. **Cuối cùng nói về trade-off**: complexity, consistency risks, operational costs, rollback plan và monitoring metrics do giải pháp mang lại.

Trong phỏng vấn, điều tối kỵ là vừa bắt đầu đã chất đống technical terms. Ví dụ “truy vấn order chậm” không nhất thiết phải database sharding, có thể chỉ là thiếu index, deep pagination, quá nhiều historical data hoặc việc truy vấn của các merchant hot bị tập trung. Trước hết hãy hỏi rõ scenario rồi mới đưa ra giải pháp, câu trả lời sẽ vững vàng hơn nhiều.

## Thứ tự đọc đề xuất

1. [Tổng hợp câu hỏi phỏng vấn về high-performance system design](./high-performance-system-interview-questions.md): trước hết xây dựng danh sách các vấn đề thường gặp về cache, database, message queue, load balancing và những nội dung khác.
2. [Giải thích chi tiết nguyên lý hoạt động của CDN](./cdn.md) và [Giải thích chi tiết nguyên lý và thuật toán load balancing](./load-balancing.md): tìm hiểu ingress traffic và request distribution.
3. [Giải thích chi tiết về read/write separation và database sharding](./read-and-write-separation-and-library-subtable.md), [Tổng hợp các phương pháp SQL optimization thường gặp](./sql-optimization.md), [Giới thiệu deep pagination và đề xuất optimization](./deep-pagination-optimization.md): bổ sung mạch chính về database performance optimization.
4. [Tổng hợp câu hỏi phỏng vấn về message queue](./message-queue/message-queue-interview-questions.md): xây dựng danh sách vấn đề về reliability, idempotency, ordering, backlog và lựa chọn sản phẩm.
5. [Tổng hợp kiến thức nền tảng về message queue](./message-queue/message-queue.md): tìm hiểu asynchronous processing, decoupling, giảm peak, message reliability, ordering và idempotency.
6. Sau đó đi sâu theo tech stack vào [Tổng hợp các vấn đề thường gặp về Kafka](./message-queue/kafka-questions-01.md), [Tổng hợp các vấn đề thường gặp về RocketMQ](./message-queue/rocketmq-questions.md), [Tổng hợp các vấn đề thường gặp về RabbitMQ](./message-queue/rabbitmq-questions.md).

## Bài viết cốt lõi

### Ingress traffic và request distribution

- [Giải thích chi tiết nguyên lý hoạt động của CDN](./cdn.md): tìm hiểu GSLB scheduling, cache strategy, preheating và refresh, hit rate optimization và hotlink protection.
- [Giải thích chi tiết nguyên lý và thuật toán load balancing](./load-balancing.md): tìm hiểu layer 4/layer 7 load balancing, server-side/client-side load balancing và các scheduling algorithm thường gặp.

### Database và data access optimization

- [Giải thích chi tiết về read/write separation và database sharding](./read-and-write-separation-and-library-subtable.md): tìm hiểu master-slave replication, read/write separation, vertical splitting, horizontal splitting và các vấn đề sau database sharding.
- [Giải thích chi tiết về hot/cold data separation](./data-cold-hot-separation.md): tìm hiểu cách xác định hot/cold data, tiered storage, consistency của data migration và cold data query optimization.
- [Tổng hợp các phương pháp SQL optimization thường gặp](./sql-optimization.md): hệ thống hóa các phương pháp thực tế như xác định slow SQL, index optimization, query rewriting và pagination optimization.
- [Giới thiệu deep pagination và đề xuất optimization](./deep-pagination-optimization.md): tìm hiểu vấn đề performance của deep pagination, cũng như các giải pháp range query, subquery optimization, delayed association và covering index.

### Message queue và asynchronous peak reduction

- [Chuyên đề message queue](./message-queue/): từ kiến thức nền tảng về message queue đến giới hạn sử dụng của Kafka, RocketMQ, RabbitMQ và Disruptor.
- [Tổng hợp câu hỏi phỏng vấn về message queue](./message-queue/message-queue-interview-questions.md): ôn tập reliability, consumption semantics, các sản phẩm phổ biến và các câu hỏi đào sâu về project theo message lifecycle.
- [Tổng hợp kiến thức nền tảng về message queue](./message-queue/message-queue.md): tìm hiểu application scenarios, message model, message reliability, ordering, idempotency và xử lý backlog.
- [Tổng hợp các vấn đề thường gặp về Kafka](./message-queue/kafka-questions-01.md): nắm vững Kafka architecture, high-performance principles, message reliability, ordering và Rebalance.
- [Tổng hợp các vấn đề thường gặp về RocketMQ](./message-queue/rocketmq-questions.md): tìm hiểu RocketMQ architecture, message types, storage mechanism, reliability và các tính năng mới của 5.x.
- [Tổng hợp các vấn đề thường gặp về RabbitMQ](./message-queue/rabbitmq-questions.md): tìm hiểu AMQP, Exchange types, acknowledgement mechanism, dead letter queue, delayed queue, Quorum Queue và Streams.
- [Tổng hợp các vấn đề thường gặp về Disruptor](./message-queue/disruptor-questions.md): tìm hiểu RingBuffer, Sequencer, WaitStrategy, lock-free design và cache line padding.

## Vấn đề thường gặp

- Khi tối ưu high-performance system, trước tiên nên xác định những metrics nào?
- CDN và load balancing lần lượt giải quyết vấn đề gì?
- Sự khác biệt giữa layer 4 load balancing và layer 7 load balancing là gì?
- Read/write separation sẽ mang lại những vấn đề consistency nào? Xử lý master-slave replication lag thế nào?
- Sau database sharding, xử lý distributed ID, cross-database JOIN và distributed transaction thế nào?
- Tại sao deep pagination chậm? Có những giải pháp optimization nào?
- Message queue làm thế nào để bảo đảm message không bị mất, không bị trùng và không bị sai thứ tự?
- Lựa chọn giữa Kafka, RocketMQ và RabbitMQ thế nào?
- Nên xác định và xử lý message backlog thế nào?

## Chuyên đề liên quan

- [Knowledge system về high-availability system](../high-availability/)
- [Knowledge system về distributed system](../distributed-system/)
- [Database](../database/)
- [System design](../system-design/)

<!-- @include: @article-footer.snippet.md -->
