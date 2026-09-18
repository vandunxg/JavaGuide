---
title: "Chuyên đề message queue: xử lý asynchronous, giảm đỉnh tải, reliability, tính thứ tự, Kafka, RocketMQ và RabbitMQ"
description: "Lộ trình học và câu hỏi phỏng vấn message queue, bao quát xử lý asynchronous, decoupling ứng dụng, giảm đỉnh tải, độ tin cậy của message, idempotent, tính thứ tự, backlog, Kafka, RocketMQ và RabbitMQ."
category: High Performance
tag:
  - message queue
  - Kafka
  - RocketMQ
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: "message queue,câu hỏi phỏng vấn message queue,Kafka,RocketMQ,RabbitMQ,Disruptor,độ tin cậy của message,message idempotent,tính thứ tự của message,message backlog,xử lý asynchronous,giảm đỉnh tải,backend interview"
---

Message queue là middleware rất phổ biến trong các hệ thống high-performance và high-availability, chủ yếu dùng để xử lý asynchronous, decoupling ứng dụng, giảm đỉnh tải và buffer traffic. Khi học message queue, không thể chỉ ghi nhớ các tính năng của Kafka, RocketMQ, RabbitMQ mà còn phải hiểu độ tin cậy, tính thứ tự, idempotent, cách xử lý backlog và lựa chọn công nghệ.

Nếu bạn chuẩn bị phỏng vấn, nên đọc trước [Tổng hợp câu hỏi phỏng vấn message queue](./message-queue-interview-questions.md), ôn các câu hỏi MQ trong toàn bộ flow production, storage, consumption, confirmation và compensation, sau đó tìm hiểu sâu các bài chuyên đề tương ứng với middleware được dùng trong project.

## Dành cho ai

- Backend developer muốn học message queue một cách có hệ thống.
- Người chuẩn bị câu hỏi phỏng vấn về Kafka, RocketMQ, RabbitMQ và độ tin cậy của message.
- Độc giả đã sử dụng MQ trong project nhưng chưa quen với việc xử lý message bị mất, consume lặp, consume theo thứ tự và backlog.
- Engineer cần lựa chọn công nghệ giữa Kafka, RocketMQ, RabbitMQ và Disruptor.

## Trọng tâm học

- Message queue giải quyết các vấn đề asynchronous, decoupling, giảm đỉnh tải và buffering; không phải flow nào cũng nên đưa MQ vào.
- Độ tin cậy của message cần được xem xét riêng ở producer, Broker, consumer và business idempotent.
- Tính thứ tự của message thường cần được thiết kế đồng thời từ Topic, queue, partition, consumer thread và business Key.
- Backlog không phải vấn đề đơn lẻ, có thể bắt nguồn từ năng lực consumption, dependency downstream, chính sách rate limiting và data skew.
- Kafka, RocketMQ, RabbitMQ và Disruptor có định vị khác nhau; khi lựa chọn cần kết hợp throughput, latency, message model, ecosystem và chi phí vận hành.

## Thứ tự đọc đề xuất

1. [Tổng hợp câu hỏi phỏng vấn message queue](./message-queue-interview-questions.md): trước tiên lập danh sách vấn đề về reliability, idempotent, thứ tự, backlog và lựa chọn công nghệ.
2. [Tổng hợp kiến thức cơ bản về message queue](./message-queue.md): tìm hiểu model chung, trường hợp sử dụng và vấn đề thường gặp của MQ.
3. [Tổng hợp câu hỏi thường gặp về Kafka](./kafka-questions-01.md): tìm hiểu high-throughput log stream, partition, replica, Consumer Group và Rebalance.
4. [Tổng hợp câu hỏi thường gặp về RocketMQ](./rocketmq-questions.md): tìm hiểu các trường hợp sử dụng business message, transaction message, scheduled message, ordered message và message storage.
5. [Tổng hợp câu hỏi thường gặp về RabbitMQ](./rabbitmq-questions.md): tìm hiểu AMQP, Exchange, message confirmation, dead-letter queue và delayed queue.
6. [Tổng hợp câu hỏi thường gặp về Disruptor](./disruptor-questions.md): tìm hiểu high-performance in-memory queue, lock-free design và các trường hợp low-latency.

## Bài viết cốt lõi

- [Tổng hợp câu hỏi phỏng vấn message queue](./message-queue-interview-questions.md): kết nối các câu hỏi phổ biến về high frequency, nguyên lý middleware phổ biến, lựa chọn công nghệ và câu hỏi đào sâu project theo message lifecycle.
- [Tổng hợp kiến thức cơ bản về message queue](./message-queue.md): giải thích có hệ thống về trường hợp sử dụng, message model, độ tin cậy, idempotent, tính thứ tự, xử lý backlog và lựa chọn công nghệ.
- [Tổng hợp câu hỏi thường gặp về Kafka](./kafka-questions-01.md): bao quát Broker, Topic, Partition, Consumer Group, zero-copy, sequential write, ACK, ISR và Rebalance.
- [Tổng hợp câu hỏi thường gặp về RocketMQ](./rocketmq-questions.md): bao quát NameServer, Broker, Proxy, ordinary message, ordered message, transaction message, scheduled message và storage mechanism.
- [Tổng hợp câu hỏi thường gặp về RabbitMQ](./rabbitmq-questions.md): bao quát AMQP, loại Exchange, cơ chế confirmation, dead-letter queue, delayed queue, priority queue và high-availability cluster.
- [Tổng hợp câu hỏi thường gặp về Disruptor](./disruptor-questions.md): bao quát RingBuffer, Sequencer, WaitStrategy, lock-free design, cache line padding và preallocated memory.

## Câu hỏi thường gặp

- Vì sao cần sử dụng message queue? Những trường hợp nào không phù hợp để đưa MQ vào?
- Làm thế nào để bảo đảm message không bị mất?
- Xử lý consume lặp và business idempotent như thế nào?
- Làm thế nào để bảo đảm tính thứ tự của message?
- Làm thế nào để điều tra và xử lý backlog?
- Vì sao Kafka có throughput cao? zero-copy và sequential write lần lượt có tác dụng gì?
- Transaction message của RocketMQ hoạt động như thế nào?
- Exchange của RabbitMQ có những loại nào? Mỗi loại phù hợp với trường hợp nào?
- Nên lựa chọn giữa Kafka, RocketMQ và RabbitMQ như thế nào?

## Tra nhanh khi lựa chọn

| Trường hợp sử dụng                    | Lựa chọn thường gặp | Trọng tâm cần quan tâm                                |
| ------------------------------------- | ------------------- | ----------------------------------------------------- |
| Log, tracking data, stream processing | Kafka, Pulsar       | Throughput, partition scaling, ecosystem              |
| Order, transaction, business event    | RocketMQ            | Transaction message, delayed message, ordered message |
| Routing linh hoạt, tích hợp nhẹ       | RabbitMQ            | Exchange, confirmation mechanism, queue type          |
| Xử lý event low-latency trong process | Disruptor           | RingBuffer, WaitStrategy, lock-free design            |

Khi lựa chọn, đừng chỉ so sánh throughput. Kinh nghiệm vận hành của team, tech stack hiện có, message semantic, yêu cầu latency, thời gian lưu message và việc có cần replay hay không đều ảnh hưởng đến lựa chọn cuối cùng.

## Chuyên đề liên quan

- [Hệ thống kiến thức high-performance](../)
- [Hệ thống kiến thức high-availability](../../high-availability/)
- [Hệ thống kiến thức distributed system](../../distributed-system/)
- [System design](../../system-design/)

<!-- @include: @article-footer.snippet.md -->
