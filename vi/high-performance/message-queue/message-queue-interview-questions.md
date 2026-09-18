---
title: "Tổng hợp câu hỏi phỏng vấn message queue mới nhất 2026: độ tin cậy, idempotent, thứ tự, backlog và lựa chọn"
description: "Câu hỏi phỏng vấn message queue và lộ trình ôn tập mới nhất 2026, bao quát asynchronous, decoupling, hấp thụ đỉnh lưu lượng, độ tin cậy của message, tiêu thụ trùng lặp, idempotent, message có thứ tự, backlog, Kafka, RocketMQ, RabbitMQ và lựa chọn công nghệ."
category: "Hiệu năng cao"
tag:
  - message queue
  - câu hỏi phỏng vấn
  - hiệu năng cao
head:
  - - meta
    - name: keywords
      content: message queue,câu hỏi phỏng vấn MQ,độ tin cậy của message,tiêu thụ trùng lặp,idempotent của message,message có thứ tự,backlog,Kafka,câu hỏi phỏng vấn RocketMQ,câu hỏi phỏng vấn RabbitMQ,lựa chọn message queue
---

Câu hỏi phỏng vấn về message queue thường bắt đầu từ “Vì sao sử dụng MQ?”, sau đó tiếp tục truy vấn theo vòng đời của một message: Producer có thể retry sau khi gửi timeout không? Broker trả về thành công có nghĩa là message chắc chắn không bị mất không? Consumer xử lý thành công nhưng xác nhận thất bại thì chuyện gì xảy ra? Cần xử lý thế nào với tiêu thụ trùng lặp, sai thứ tự và backlog?

Bài viết này là điểm bắt đầu ôn tập cho chuyên đề message queue của JavaGuide, được sắp xếp thành bốn phần: trường hợp sử dụng, độ tin cậy của message, middleware phổ biến và lựa chọn công nghệ. Mỗi phần chỉ liệt kê những vấn đề cần nắm khi ôn tập; câu trả lời đầy đủ và chi tiết triển khai nằm trong các bài chuyên đề tương ứng.

Nếu thời gian khá gấp, bạn có thể xem trước [Tổng hợp câu hỏi phỏng vấn message queue thường gặp](https://interview.javaguide.cn/high-performance/message-queue-interview-questions.html) bản ôn phỏng vấn cấp tốc, đánh dấu những vấn đề chưa giải thích rõ, rồi quay lại bài này để bổ sung nguyên lý và chi tiết kỹ thuật.

## Khi ôn tập, trước tiên cần nắm những vấn đề nào?

| Module                         | Nội dung cần giải thích rõ                                                                                      | Hướng hỏi thêm thường gặp                                                                           |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| Trường hợp sử dụng và giới hạn | Vì sao sử dụng MQ, những chuỗi nào không phù hợp để chuyển sang asynchronous                                    | asynchronous, decoupling, hấp thụ đỉnh lưu lượng, eventual consistency, RPC và MQ                   |
| Độ tin cậy và semantic         | Mỗi khâu từ sản xuất đến tiêu thụ xác nhận, retry và khôi phục thế nào                                          | mất message, tiêu thụ trùng lặp, idempotent, thứ tự, retry, dead letter, backlog                    |
| Nguyên lý middleware           | MQ phổ biến tổ chức message, replicate dữ liệu và điều phối consumer thế nào                                    | Kafka, RocketMQ, RabbitMQ, partition, replica, consumer group, cơ chế xác nhận                      |
| Lựa chọn và quản trị           | Kết hợp năng lực nghiệp vụ, kinh nghiệm vận hành và mục tiêu xử lý sự cố để chọn, quản lý message queue thế nào | throughput, latency, replay, transaction message, monitoring, mở rộng, disaster recovery, Disruptor |

Khi trả lời câu hỏi về độ tin cậy của message, không thể chỉ nêu một config. Cần giải thích đồng thời việc xác nhận của producer, persistence và replication của Broker, xác nhận của consumer, idempotent ở tầng nghiệp vụ và compensation, reconciliation; bất kỳ khâu nào xử lý không đầy đủ cũng có thể biểu hiện thành mất message, trùng lặp hoặc kết quả không nhất quán.

## Kiến thức cơ bản và giới hạn sử dụng của message queue

Message queue chuyển lời gọi synchronous giữa producer và consumer thành truyền message asynchronous. Luồng chính có thể trả về nhanh hơn, traffic đột biến cũng có thể tạm lưu trong Broker, nhưng hệ thống đồng thời tăng thêm middleware, trạng thái asynchronous và quy trình khôi phục khi xảy ra sự cố.

![Nâng performance hệ thống nhờ xử lý bất đồng bộ](https://oss.javaguide.cn/github/javaguide/Asynchronous-message-queue.png)

Nội dung liên quan:

- [Tổng hợp kiến thức cơ bản về message queue](./message-queue.md)
- [Tổng hợp câu hỏi phỏng vấn thiết kế hệ thống hiệu năng cao](../high-performance-system-interview-questions.md)

Câu hỏi phỏng vấn thường gặp:

- Message queue thực hiện asynchronous, decoupling ứng dụng và hấp thụ đỉnh lưu lượng thế nào?
- Sau khi đưa message queue vào, độ phức tạp hệ thống và data consistency thay đổi ra sao?
- Nghiệp vụ nào phù hợp với xử lý asynchronous? Có thể chuyển thẳng các luồng cần realtime mạnh như kết quả thanh toán, khấu trừ tồn kho sang asynchronous không?
- RPC và message queue khác nhau thế nào về hướng giao tiếp, thời gian phản hồi, cách coupling và xử lý sự cố?
- Mô hình point-to-point và publish-subscribe lần lượt phù hợp với trường hợp nào?
- Vì sao không thể ghi message vào MQ rồi lập tức trả về cho người dùng “nghiệp vụ thành công”?

Khi trả lời “Vì sao sử dụng MQ?”, tốt nhất nên gắn với một flow cụ thể. Ví dụ, sau khi tạo order, bước nào bắt buộc hoàn thành synchronous, notification, điểm tích lũy và task risk control nào có thể xử lý asynchronous; khi consumer thất bại, trạng thái người dùng nhìn thấy liên kết với compensation ở backend thế nào.

## Độ tin cậy của message và ngữ nghĩa tiêu thụ

Có thể trả lời về độ tin cậy của message theo bốn phần: producer, Broker, consumer và xử lý nghiệp vụ. Producer phải xác nhận message đã được tiếp nhận chưa, Broker phải cân nhắc persistence và replica, consumer phải xác nhận sau khi nghiệp vụ hoàn tất, còn phía nghiệp vụ phải xử lý trường hợp trùng lặp và kết quả không xác định.

![Mô hình message queue](https://oss.javaguide.cn/github/javaguide/high-performance/message-queue/message-queue-queue-model.png)

Nội dung liên quan:

- [Tổng hợp kiến thức cơ bản về message queue](./message-queue.md)
- [Tổng hợp câu hỏi thường gặp về Kafka](./kafka-questions-01.md)
- [Tổng hợp câu hỏi thường gặp về RocketMQ](./rocketmq-questions.md)
- [Tổng hợp câu hỏi thường gặp về RabbitMQ](./rabbitmq-questions.md)

Câu hỏi phỏng vấn thường gặp:

- At Most Once, At Least Once và Exactly Once lần lượt có nghĩa gì?
- Message có thể bị mất ở những khâu nào? Mỗi khâu giảm rủi ro thế nào?
- Vì sao At Least Once dễ tạo ra message trùng lặp? Consumer bảo đảm idempotent nghiệp vụ thế nào?
- Vì sao deduplication bằng message ID không thể hoàn toàn thay thế business unique key?
- Bảo đảm message của cùng một order hoặc cùng một user được xử lý theo thứ tự thế nào?
- Sau khi tiêu thụ thất bại, lỗi nào phù hợp để retry, message nào nên đưa vào dead letter queue?
- Khi xảy ra backlog, phân biệt traffic sản xuất tăng đột biến, consumer chậm lại, partition không đủ và downstream gặp sự cố thế nào?

Thứ tự, độ tin cậy và throughput thường ảnh hưởng lẫn nhau. Cố định cùng một business key vào một queue hoặc partition có thể bảo đảm local ordering, nhưng cũng hạn chế parallelism; synchronous flush và nhiều replica hơn có thể giảm rủi ro mất dữ liệu, đồng thời làm tăng latency ghi và chi phí tài nguyên.

## Câu hỏi thường gặp về Kafka

Các câu hỏi thường gặp về Kafka chủ yếu xoay quanh partition, replica, consumer group, độ tin cậy và thiết kế throughput cao. Khi trả lời, cần đặt Producer, Broker, Partition, Replica, Consumer Group và Offset vào cùng một read/write flow.

![Bố cục Partition của Kafka Topic](https://oss.javaguide.cn/github/javaguide/high-performance/message-queue/KafkaTopicPartionsLayout.png)

Nội dung liên quan: [Tổng hợp câu hỏi thường gặp về Kafka](./kafka-questions-01.md)

Câu hỏi phỏng vấn thường gặp:

- Topic, Partition, Replica, Consumer Group và Offset lần lượt đảm nhiệm vai trò gì?
- Vì sao Kafka chỉ bảo đảm ordering trong một Partition? Nên chọn business Key thế nào?
- Quan hệ giữa Leader, Follower, ISR, HW và LEO là gì?
- Nên phối hợp `acks=all`, số replica và `min.insync.replicas` thế nào?
- Vì sao Kafka có throughput cao? Ghi tuần tự, Page Cache, zero-copy, batching và compression lần lượt có tác dụng gì?
- Idempotent producer và transaction của Kafka giải quyết vấn đề gì? Exactly Once có thể bao phủ external database không?
- Vì sao Kafka 4.0 chỉ hỗ trợ KRaft? Nó khác gì với mode ZooKeeper trước đây?
- Rebalance gây ra những ảnh hưởng nào? Giảm rebalance không cần thiết thế nào?

Không nên ghi nhớ tham số Kafka một cách rời rạc. Ví dụ, `acks=all` chờ xác nhận từ các replica trong ISR hiện tại đáp ứng điều kiện, không có nghĩa là mọi replica đã cấu hình đều được flush xuống disk; thời điểm commit Offset sau khi tiêu thụ thành công cũng ảnh hưởng trực tiếp đến rủi ro tiêu thụ trùng lặp và mất message.

## Câu hỏi thường gặp về RocketMQ

Câu hỏi phỏng vấn về RocketMQ thường gần với các trường hợp sử dụng message trong nghiệp vụ hơn; transaction message, delay message, message có thứ tự và lưu trữ message là những vấn đề thường được hỏi thêm. Khi ôn tập, có thể bắt đầu từ cách message đi qua NameServer, Producer, Broker và Consumer, sau đó tìm hiểu CommitLog, ConsumeQueue cùng cơ chế flush và replication.

Nội dung liên quan: [Tổng hợp câu hỏi thường gặp về RocketMQ](./rocketmq-questions.md)

Câu hỏi phỏng vấn thường gặp:

- NameServer, Broker, Producer, Consumer và Proxy lần lượt phụ trách gì?
- Các node NameServer không giao tiếp với nhau, vậy cung cấp khả năng service discovery thế nào?
- CommitLog, ConsumeQueue và IndexFile lần lượt lưu dữ liệu gì?
- Half message, commit, rollback và transaction checkback trong transaction message của RocketMQ phối hợp thế nào?
- Delay message và scheduled message phù hợp với trường hợp nào? Vì sao consumer vẫn phải kiểm tra trạng thái nghiệp vụ?
- RocketMQ bảo đảm message trong cùng một business group có thứ tự thế nào?
- Synchronous flush, asynchronous flush, synchronous replication và asynchronous replication lần lượt ảnh hưởng đến điều gì?
- Sau khi message bị backlog, vì sao tăng consumer chưa chắc nâng được tốc độ xử lý?

Transaction message giải quyết việc điều phối giữa local transaction và gửi message, nhưng không thể thay consumer hoàn thành idempotent và compensation. Kết quả transaction checkback của Broker vẫn là trạng thái local transaction của producer; việc consumer thực thi trùng lặp, downstream thất bại và reconciliation nghiệp vụ cần được xử lý riêng.

## Câu hỏi thường gặp về RabbitMQ

RabbitMQ tập trung kiểm tra routing model của AMQP, cơ chế xác nhận, dead letter và delay queue, cùng độ tin cậy của các loại queue khác nhau. Trước hết phải giải thích rõ quan hệ giữa Exchange, Routing Key, Binding và Queue.

![Kiến trúc cốt lõi và vòng đời message của RabbitMQ](https://oss.javaguide.cn/github/javaguide/high-performance/rabbitmq/rabbitmq-core-architecture-and-message-lifecycle-flow.png)

Nội dung liên quan: [Tổng hợp câu hỏi thường gặp về RabbitMQ](./rabbitmq-questions.md)

Câu hỏi phỏng vấn thường gặp:

- direct, fanout, topic và headers Exchange lần lượt route message thế nào?
- Publisher Confirms có thể xác nhận đến khâu nào? Khi message không thể route đến queue, phát hiện thế nào?
- Chỉ persistence Exchange, Queue và Message có đủ để bảo đảm message không bị mất không?
- Manual ACK, NACK, requeue, retry và dead letter queue phối hợp thế nào?
- Khi dùng TTL cùng dead letter exchange để triển khai delay message, vì sao có thể xảy ra head-of-line blocking?
- Classic Queue, Quorum Queue và Stream lần lượt phù hợp với trường hợp nào?
- Triển khai cluster RabbitMQ trên nhiều node có đồng nghĩa message trong queue đã có replica không?
- Nên monitoring Ready, Unacked, message age, tốc độ xác nhận và trạng thái replica thế nào?

Publisher Confirm chỉ cho biết Broker đã xử lý lần publish này; message có vào đúng business queue hay không còn phải xem routing result. Consumer confirm cũng chỉ cho biết phía consumer đã tiếp nhận kết quả xử lý, còn database nghiệp vụ có được ghi chính xác hay không vẫn cần transaction, idempotent và compensation bảo đảm.

## Lựa chọn message queue và câu hỏi đào sâu về dự án

Khi lựa chọn, trước tiên xác nhận nghiệp vụ cần semantic message nào, sau đó xem team có đủ kinh nghiệm deploy và xử lý sự cố tương ứng không. Chỉ so sánh throughput trên một máy khó đưa ra kết luận đáng tin cậy.

Nội dung liên quan:

- [Tổng hợp kiến thức cơ bản về message queue](./message-queue.md)
- [Tổng hợp câu hỏi thường gặp về Disruptor](./disruptor-questions.md)

Câu hỏi phỏng vấn thường gặp:

- Kafka, RocketMQ và RabbitMQ khác nhau thế nào về message model, throughput, latency, replay và vận hành?
- Log stream, order event, routing phức tạp và task delay lần lượt phù hợp với loại sản phẩm nào?
- Khi công ty đã vận hành ổn định một MQ, trường hợp nào đáng để đưa thêm sản phẩm khác vào?
- Message queue nên monitoring những metric nào về sản xuất, lưu trữ, tiêu thụ và nghiệp vụ?
- Sau khi Broker, consumer hoặc dependency downstream gặp sự cố, hệ thống degrade và khôi phục thế nào?
- Disruptor khác gì với distributed message queue?

Câu hỏi đào sâu về dự án thường đi vào các con số cụ thể và xử lý bất thường: production rate cao nhất, latency tiêu thụ, số partition hoặc queue, thời gian lưu message, số lần retry tối đa, ngưỡng cảnh báo backlog và quy trình khôi phục sau sự cố. Với config và incident chưa từng tham gia, chỉ cần trả lời theo những gì đã học, không bịa số liệu production.

## Sắp xếp việc ôn tập theo thời gian chuẩn bị

| Thời gian còn lại | Lịch đề xuất                                                                                                                                                                                                       | Mục tiêu ôn tập                                                            |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------- |
| 1–2 ngày          | Đọc nhanh [Tổng hợp câu hỏi phỏng vấn message queue thường gặp](https://interview.javaguide.cn/high-performance/message-queue-interview-questions.html), ưu tiên bổ sung độ tin cậy, idempotent, thứ tự và backlog | Có thể trả lời câu hỏi thường gặp theo vòng đời message                    |
| 3–7 ngày          | Ngoài các vấn đề chung, tập trung học một MQ đang dùng thực tế trong dự án, sau đó bổ sung định vị và khác biệt chính của hai sản phẩm còn lại                                                                     | Có thể trả lời câu hỏi về nguyên lý sản phẩm, xử lý bất thường và lựa chọn |
| Trên 1 tuần       | Đọc các bài chuyên đề theo thứ tự trong bài này, vẽ chain production, storage, replication, consumption, confirmation và compensation, đồng thời dựa vào dự án để tổng hợp monitoring và quy trình khôi phục sự cố | Có thể trình bày từ yêu cầu nghiệp vụ đến config, trade-off và vận hành    |

<!-- @include: @article-footer.snippet.md -->
