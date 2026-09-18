---
title: "Tổng hợp kiến thức cơ bản về message queue"
description: "Bài viết hệ thống hóa kiến thức cốt lõi về message queue, bao gồm các trường hợp sử dụng (xử lý bất đồng bộ/tách biệt hệ thống/giảm đỉnh tải), mô hình message (point-to-point/publish-subscribe), cách bảo đảm message không bị mất, tính idempotent và thứ tự của message, cách xử lý message backlog cùng các vấn đề thường gặp khác, cũng như so sánh lựa chọn công nghệ Kafka, RocketMQ và RabbitMQ."
category: High Performance
tag:
  - message queue
head:
  - - meta
    - name: keywords
      content: "message queue,MQ,xử lý bất đồng bộ,tách biệt hệ thống,giảm đỉnh tải,message bị mất,message idempotent,thứ tự message,Kafka,RocketMQ,RabbitMQ"
---

::: tip

Message queue trong bài viết này chủ yếu chỉ distributed message queue.

:::

“RabbitMQ?” “Kafka?” “RocketMQ?”... Trong quá trình học tập và phát triển hằng ngày, chúng ta thường nghe đến keyword message queue. Tôi cũng đã đề cập đến khái niệm này trong nhiều bài viết. Có thể bạn là người đã thành thạo message queue, hoặc là người mới chưa hiểu message queue. Dù bạn đã biết hay chưa, bài viết này sẽ giúp bạn nắm được một số lý thuyết cơ bản về message queue.

Nếu bạn đã có kinh nghiệm, bài viết có thể giúp bạn biết thêm một số khái niệm quan trọng về message queue mà trước đây chưa chú ý. Nếu bạn là người mới, đây sẽ là viên gạch đầu tiên mở cánh cửa đến với message queue.

## Message queue là gì?

Có thể xem message queue là một container lưu trữ message. Khi cần sử dụng message, ta chỉ việc lấy message ra khỏi container. Vì Queue là một cấu trúc dữ liệu vào trước ra trước, message cũng được consume theo thứ tự.

![](https://oss.javaguide.cn/github/javaguide/high-performance/message-queue/message-queue-small.png)

Hai bên tham gia truyền message được gọi là **producer** và **consumer**. Producer chịu trách nhiệm gửi message, consumer chịu trách nhiệm xử lý message.

![Mô hình publish/subscribe (Pub/Sub)](https://oss.javaguide.cn/github/javaguide/high-performance/message-queue/message-queue-pub-sub-model.png)

Một phương thức rất quan trọng để giao tiếp giữa các process trong operating system là message queue. Message queue được đề cập ở đây có đôi chút khác biệt: nó chủ yếu chỉ việc giao tiếp giữa các service và giữa các component/module bên trong hệ thống, thuộc loại **middleware**.

Wikipedia giới thiệu middleware như sau:

> Middleware (tiếng Anh: Middleware), còn được gọi là middleware hoặc tầng trung gian, là một loại software cung cấp kết nối giữa system software và application software, giúp các thành phần software giao tiếp với nhau. Application software có thể dùng middleware để chia sẻ thông tin và resource giữa các technical architecture khác nhau. Middleware nằm trên operating system của client-server, quản lý resource tính toán và network communication.

Nói đơn giản: **middleware là loại software phục vụ application software; application software phục vụ user, còn user sẽ không tiếp xúc hoặc sử dụng middleware.**

Ngoài message queue, các middleware thường gặp còn có RPC framework, distributed component, HTTP server, task scheduling framework, configuration center, công cụ sharding database và công cụ data migration.

Có thể tham khảo phần trả lời của Taobao Technology thuộc Alibaba để xem giới thiệu chi tiết hơn về middleware: <https://www.zhihu.com/question/19730582/answer/1663627873> .

Cùng với sự phát triển của distributed system và microservice system, message queue có nhiều không gian phát huy hơn trong system design. Sử dụng message queue có thể giảm coupling của hệ thống, thực hiện task bất đồng bộ và hiệu quả giảm đỉnh traffic, là một component quan trọng trong distributed system và microservice system.

## Message queue có tác dụng gì?

Thông thường, message queue chủ yếu mang lại ba lợi ích sau cho hệ thống:

1. Xử lý bất đồng bộ
2. Giảm đỉnh tải/giới hạn traffic
3. Giảm coupling của hệ thống

Ngoài ba điểm này, message queue còn có các trường hợp sử dụng khác như thực hiện distributed transaction, bảo đảm thứ tự và xử lý data stream.

Nếu được interviewer hỏi câu này, thông thường là vì trong CV của bạn có đề cập đến message queue. Khi đó, nên kết hợp với project của chính bạn để trả lời.

### Xử lý bất đồng bộ

![Cải thiện performance hệ thống bằng xử lý bất đồng bộ](https://oss.javaguide.cn/github/javaguide/Asynchronous-message-queue.png)

Với các thao tác tốn thời gian trong user request, hãy dùng message queue để xử lý bất đồng bộ: gửi message tương ứng vào message queue rồi trả về ngay, từ đó giảm response time và cải thiện user experience. Sau đó, hệ thống mới consume message.

Sau khi data của user request được ghi vào message queue, response được trả ngay cho user, nhưng các thao tác tiếp theo như business validation và ghi database có thể thất bại. Vì vậy, **sau khi dùng message queue để xử lý bất đồng bộ, cần điều chỉnh business flow cho phù hợp**. Ví dụ, sau khi user đặt order, dữ liệu order được ghi vào message queue thì không thể lập tức trả về rằng order đã đặt thành công. Cần chờ process consumer của order trong message queue xử lý xong order, thậm chí sau khi xuất kho, rồi mới thông báo order thành công cho user qua email hoặc SMS để tránh tranh chấp giao dịch. Cách này tương tự việc đặt vé tàu hoặc vé xem phim trên điện thoại.

### Giảm đỉnh tải/giới hạn traffic

**Trước tiên lưu các transaction message do concurrency cao trong thời gian ngắn tạo ra vào message queue, sau đó backend service consume các message này dần theo năng lực của mình, nhờ đó tránh làm backend service bị quá tải trực tiếp.**

Ví dụ: trong các hoạt động flash sale và promotion của e-commerce, sử dụng message queue hợp lý có thể chống lại hiệu quả tác động của lượng order lớn đổ vào hệ thống ngay khi promotion bắt đầu. Như hình dưới đây:

![Giảm đỉnh tải](https://oss.javaguide.cn/github/javaguide/%E5%89%8A%E5%B3%B0-%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97.png)

### Giảm coupling của hệ thống

Sử dụng message queue còn có thể giảm coupling của hệ thống. Nếu các module không gọi trực tiếp lẫn nhau, việc thêm hoặc sửa một module sẽ ít ảnh hưởng đến các module khác, nhờ đó khả năng mở rộng của hệ thống tốt hơn.

Producer (client) gửi message vào message queue, consumer (server) xử lý message. Hệ thống cần consume chỉ việc lấy message từ message queue để consume, không cần coupling với hệ thống khác, qua đó rõ ràng cũng cải thiện khả năng mở rộng của hệ thống.

![Mô hình publish/subscribe (Pub/Sub)](https://oss.javaguide.cn/github/javaguide/high-performance/message-queue/message-queue-pub-sub-model.png)

**Message queue hoạt động theo publish-subscribe: message sender (producer) publish message, một hoặc nhiều message receiver (consumer) subscribe message.** Như hình trên cho thấy, **message sender (producer) và message receiver (consumer) không coupling trực tiếp với nhau**. Message sender gửi message vào distributed message queue rồi kết thúc xử lý message; message receiver lấy message từ distributed message queue để xử lý tiếp mà không cần biết message đến từ đâu. **Với business mới, chỉ cần quan tâm đến loại message đó là có thể subscribe message; hệ thống và business cũ không bị ảnh hưởng, từ đó thực hiện system design có khả năng mở rộng cho website.**

Ví dụ, hệ thống shopping mall được chia thành các service như user, order, finance, warehouse, notification, logistics và risk control. Sau khi user hoàn tất đặt order, cần gọi các service finance (trừ tiền), warehouse (quản lý tồn kho), logistics (giao hàng), notification (thông báo giao hàng cho user) và risk control (đánh giá rủi ro). Sau khi sử dụng message queue, thao tác đặt order và các thao tác trừ tiền, giao hàng, thông báo tiếp theo được tách biệt: khi đặt order xong, gửi một message vào message queue, nơi nào cần thì subscribe và consume message đó.

![](https://oss.javaguide.cn/github/javaguide/high-performance/message-queue/message-queue-decouple-mall-example.png)

Ngoài ra, để tránh mất message do message queue server bị down, message đã gửi thành công vào message queue sẽ được lưu trên producer server và chỉ bị xóa sau khi thực sự được consumer server xử lý. Khi message queue server down, producer server sẽ chọn server khác trong cluster distributed message queue để publish message.

**Ghi chú:** Đừng cho rằng message queue chỉ có thể hoạt động theo publish-subscribe. Chỉ là trong business context đặc thù là tách biệt hệ thống này, ta dùng publish-subscribe. Ngoài publish-subscribe còn có point-to-point subscribe (một message chỉ có một consumer), nhưng publish-subscribe được dùng phổ biến hơn. Hai message model này do JMS cung cấp; AMQP còn cung cấp thêm 5 message model khác.

### Thực hiện distributed transaction

Một trong các solution cho distributed transaction là MQ transaction.

RocketMQ, Kafka, Pulsar và QMQ đều cung cấp chức năng liên quan đến transaction. Transaction cho phép event stream application định nghĩa toàn bộ quá trình consume, process và produce message thành một atomic operation.

Xem chi tiết trong bài viết [Giải thích chi tiết về distributed transaction (có phí)](https://javaguide.cn/distributed-system/distributed-transaction.html).

![Giải thích chi tiết về distributed transaction - MQ transaction](https://oss.javaguide.cn/github/javaguide/csdn/07b338324a7d8894b8aef4b659b76d92.png)

### Bảo đảm thứ tự

Trong nhiều trường hợp sử dụng, thứ tự xử lý data rất quan trọng. Message queue bảo đảm data được xử lý theo thứ tự nhất định, phù hợp với các trường hợp yêu cầu nghiêm ngặt về thứ tự data. Phần lớn message queue như RocketMQ, RabbitMQ, Pulsar và Kafka đều hỗ trợ ordered message.

### Xử lý delay/schedule

Sau khi gửi message, message không được consume ngay mà sẽ được chỉ định một thời điểm; đến thời điểm đó mới consume. Phần lớn message queue như RocketMQ, RabbitMQ và Pulsar đều hỗ trợ scheduled/delayed message.

![](https://oss.javaguide.cn/github/javaguide/tools/docker/rocketmq-schedule-message.png)

### Instant messaging

MQTT (Message Queuing Telemetry Transport) là một communication protocol nhẹ, dùng publish/subscribe, rất phù hợp với các application như IoT cần hoạt động trong môi trường network băng thông thấp, latency cao hoặc không đáng tin cậy. MQTT hỗ trợ truyền instant message và vẫn duy trì communication ổn định ngay cả khi điều kiện network kém.

RabbitMQ tích hợp MQTT plugin để triển khai chức năng MQTT (mặc định không bật, cần bật thủ công).

### Xử lý data stream

Với lượng data stream lớn do distributed system tạo ra, chẳng hạn business log, monitoring data và user behavior, message queue có thể thu thập data theo thời gian thực hoặc theo batch, rồi đưa vào big data processing engine để quản lý và xử lý data stream hiệu quả.

## Sử dụng message queue sẽ mang lại những vấn đề gì?

- **Giảm system availability:** System availability giảm ở một mức độ nhất định. Vì sao? Trước khi thêm MQ, bạn không cần cân nhắc các trường hợp message bị mất hoặc MQ bị down. Nhưng sau khi đưa MQ vào, bạn phải cân nhắc các vấn đề này.
- **Tăng system complexity:** Sau khi thêm MQ, bạn cần bảo đảm message không bị consume lặp, xử lý trường hợp message bị mất, bảo đảm thứ tự truyền message và các vấn đề khác.
- **Vấn đề consistency:** Như đã nói ở trên, message queue có thể thực hiện bất đồng bộ, và tính bất đồng bộ do message queue mang lại thực sự có thể cải thiện response speed của hệ thống. Nhưng nếu consumer thực sự của message không consume đúng thì sao? Điều đó sẽ dẫn đến data không consistency.

Khi đánh giá một flow có phù hợp để đưa MQ vào hay không, có thể xem các câu hỏi sau:

- Bên gọi có bắt buộc phải nhận kết quả xử lý của downstream ngay không? Nếu bắt buộc, RPC có thể phù hợp hơn.
- Business có chấp nhận eventual consistency không? Nếu không, bất đồng bộ hóa sẽ mang lại thêm chi phí compensation.
- Peak traffic có vượt rõ rệt khả năng xử lý của downstream không? Nếu có, MQ có thể buffer và giảm đỉnh tải.
- Sau khi downstream thất bại có cần retry, compensation và can thiệp thủ công không? Nếu cần, MQ có thể lưu giữ recovery flow.
- Team có năng lực vận hành, monitoring và xử lý sự cố MQ không? Nếu không, complexity có thể lớn hơn lợi ích.

Nói đơn giản, MQ phù hợp với các trường hợp “có thể hoàn thành muộn hơn nhưng không được mất, có thể khôi phục và cần tách biệt hệ thống”, không phù hợp để vô thức chuyển mọi synchronous flow thành asynchronous.

## Làm thế nào để bảo đảm reliability của message?

Phải tách reliability của message theo từng chặng, không nên chỉ nói “bật persistence”:

1. **Giai đoạn producer gửi message:** Bật send confirmation và retry khi gửi thất bại; với business cốt lõi, có thể lưu local message table hoặc transaction message để tránh local transaction thành công nhưng message chưa được gửi.
2. **Giai đoạn Broker lưu trữ:** Message phải được persist; Topic/Queue quan trọng cần cấu hình replica hoặc high-availability queue. Chính sách flush và replica confirmation phải phù hợp với yêu cầu reliability của business.
3. **Giai đoạn consumer xử lý:** Chỉ ACK hoặc commit offset sau khi business xử lý thành công; khi xử lý thất bại, cần retry, đưa vào dead-letter queue hoặc đưa vào compensation flow.
4. **Giai đoạn business dự phòng:** Dùng reconciliation task, compensation task, alert và xử lý thủ công để bao quát các exception cực đoan.

Chi tiết triển khai của mỗi MQ khác nhau, nhưng tư duy đều giống nhau: **confirmation, persistence, retry, idempotent và compensation**.

## Xử lý consume lặp và idempotent như thế nào?

Trong production environment, thường rất khó bảo đảm message tuyệt đối chỉ được consume một lần. Semantic phổ biến hơn là “at-least-once delivery”, nghĩa là message có thể bị lặp và consumer phải làm idempotent.

Các solution idempotent thường gặp:

- **Unique index:** Dùng business unique key như order number, payment number hoặc message ID để ngăn ghi lặp.
- **State machine:** Chỉ cho phép state chuyển theo hướng hợp lệ, chẳng hạn order chỉ có thể chuyển từ “đã thanh toán” sang “đã giao hàng”.
- **Consumption record table:** Ghi lại trạng thái xử lý message; message lặp thì bỏ qua trực tiếp.
- **Redis deduplication:** Phù hợp để loại trùng trong một time window ngắn, nhưng cần chú ý expiration time và rủi ro persistence.

Điểm cốt lõi của idempotent là sử dụng business unique key, không phụ thuộc vào message ID do MQ tự tạo. Vì cùng một business event khi retry, compensation hoặc gửi lại có thể tạo ra message ID khác nhau.

## Xử lý message backlog như thế nào?

Khi message backlog, trước tiên hãy xác định nguyên nhân rồi mới quyết định có scale hay không:

- **Production tăng đột biến:** Activity, promotion lớn, crawler hoặc upstream exception khiến tốc độ ghi message tăng vọt.
- **Consumer chậm đi:** Slow SQL, external API chậm, lock contention, thread pool không đủ hoặc batch quá nhỏ.
- **Không đủ partition hoặc queue:** Instance consumer đã tăng nhưng cùng một queue/partition vẫn chỉ cho phép số consumer hữu hạn xử lý song song.
- **Broker exception:** Disk, network, Controller/NameServer hoặc cluster replication gặp vấn đề.

Thứ tự xử lý thường là: trước tiên giảm thiệt hại bằng rate limiting, sau đó scale consumer và partition, tiếp theo xác định logic consume chậm, cuối cùng xử lý batch tạm thời hoặc replay backlog cũ. Đừng vừa bắt đầu đã chỉ nói “thêm consumer”. Nếu số queue không đủ hoặc logic consume là tuần tự, thêm instance cũng không tăng throughput.

## JMS và AMQP

### JMS là gì?

JMS (JAVA Message Service, Java Message Service) là message service của Java. Client của JMS có thể truyền message bất đồng bộ thông qua JMS service. **JMS (JAVA Message Service, Java Message Service) API là một standard, hay nói cách khác là một specification, cho message service**, cho phép application component tạo, gửi, nhận và đọc message trên nền tảng JavaEE. JMS giúp distributed communication có coupling thấp hơn, message service đáng tin cậy hơn và có tính bất đồng bộ.

JMS định nghĩa năm format khác nhau cho message body cùng các loại message được gọi, cho phép gửi và nhận data dưới nhiều dạng khác nhau:

- `StreamMessage`: data stream của các giá trị nguyên thủy Java
- `MapMessage`: một tập hợp các cặp name-value
- `TextMessage`: một string object
- `ObjectMessage`: một Java object đã được serialize
- `BytesMessage`: một data stream dạng byte

**ActiveMQ (đã bị loại bỏ) được triển khai dựa trên JMS specification.**

### Hai message model của JMS

#### Mô hình point-to-point (P2P)

![Mô hình queue](https://oss.javaguide.cn/github/javaguide/high-performance/message-queue/message-queue-queue-model.png)

Dùng **Queue** làm carrier cho message communication; phù hợp với **producer-consumer model**, một message chỉ có thể được một consumer sử dụng. Message chưa được consume được giữ trong queue cho đến khi được consume hoặc hết thời gian chờ. Ví dụ: nếu producer gửi 100 message và có hai consumer consume, thông thường hai consumer sẽ lần lượt consume mỗi bên một nửa theo thứ tự message được gửi (tức là một message cho bạn, một message cho tôi).

#### Mô hình publish/subscribe (Pub/Sub)

![Mô hình publish/subscribe (Pub/Sub)](https://oss.javaguide.cn/github/javaguide/high-performance/message-queue/message-queue-pub-sub-model.png)

Mô hình publish/subscribe (Pub/Sub) dùng **Topic** làm carrier cho message communication, tương tự **broadcast model**. Publisher publish một message, message đó được truyền qua Topic đến tất cả subscriber.

### AMQP là gì?

AMQP, tức Advanced Message Queuing Protocol, là một application-layer standard cung cấp message service thống nhất, hay **Advanced Message Queuing Protocol** (binary application-layer protocol). Đây là một open standard của application-layer protocol, được thiết kế cho middleware hướng message và tương thích với JMS. Client và message middleware dựa trên protocol này có thể truyền message mà không bị giới hạn bởi việc client/middleware có cùng product hay khác programming language.

**RabbitMQ được triển khai dựa trên AMQP protocol.**

### JMS và AMQP

|         Tiêu chí          | JMS                                               | AMQP                                                                                                                                                                                                                                       |
| :-----------------------: | :------------------------------------------------ | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|        Định nghĩa         | Java API                                          | Protocol                                                                                                                                                                                                                                   |
|      Cross-language       | Không                                             | Có                                                                                                                                                                                                                                         |
|      Cross-platform       | Không                                             | Có                                                                                                                                                                                                                                         |
| Message model được hỗ trợ | Cung cấp hai message model: ①Peer-2-Peer;②Pub/sub | Cung cấp năm message model: ①direct exchange; ②fanout exchange; ③topic exchange; ④headers exchange; ⑤system exchange. Về bản chất, bốn loại sau không khác nhiều so với JMS pub/sub, chỉ được phân chia chi tiết hơn về routing mechanism. |
| Message type được hỗ trợ  | Hỗ trợ nhiều message type, như đã nêu ở trên      | `byte[]` (binary)                                                                                                                                                                                                                          |

**Tóm tắt:**

- AMQP định nghĩa protocol ở wire level cho message, còn JMS định nghĩa API specification. Trong Java ecosystem, nhiều client có thể tương tác qua JMS mà không cần sửa code application, nhưng khả năng cross-platform kém hơn. AMQP có sẵn đặc tính cross-platform và cross-language.
- JMS hỗ trợ các message type phức tạp như `TextMessage` và `MapMessage`; AMQP chỉ hỗ trợ message type `byte[]` (có thể serialize type phức tạp rồi gửi).
- Nhờ routing algorithm do Exchange cung cấp, AMQP có thể cung cấp nhiều cách routing để truyền message vào message queue, còn JMS chỉ hỗ trợ hai cách là Queue và Topic/Subscribe.

## Sự khác nhau giữa RPC và message queue

RPC và message queue đều là component quan trọng trong distributed microservice system. Dưới đây là so sánh đơn giản:

- **Về mục đích:** RPC chủ yếu giải quyết vấn đề remote communication giữa hai service mà không cần hiểu cơ chế communication của network bên dưới. RPC giúp gọi method của một service trên remote computer, quy trình đơn giản như gọi local method. Message queue chủ yếu dùng để giảm coupling hệ thống, thực hiện task bất đồng bộ và hiệu quả giảm đỉnh traffic.
- **Về communication method:** RPC là direct two-way network communication; message queue là one-way network communication có thêm carrier ở giữa.
- **Về architecture:** Message queue cần lưu message, còn RPC không yêu cầu điều này vì RPC là direct two-way network communication.
- **Về tính kịp thời của request processing:** Call qua RPC thường được xử lý ngay; message lưu trong message queue không nhất thiết được xử lý ngay.

Về bản chất, RPC và message queue là hai implementation mechanism khác nhau của network communication, mục đích khác nhau, tuyệt đối không được đánh đồng.

## Lựa chọn công nghệ distributed message queue

### Các message queue thường gặp là gì?

#### Kafka

![](https://oss.javaguide.cn/github/javaguide/high-performance/message-queue/kafka-logo.png)

Kafka là một distributed stream processing platform do LinkedIn open source, đã trở thành Apache top-level project. Ban đầu Kafka được dùng để xử lý lượng log lớn, sau đó dần phát triển thành một high-performance message queue có đầy đủ tính năng.

Stream processing platform có ba chức năng cốt lõi:

1. **Message queue:** Publish và subscribe message stream. Chức năng này tương tự message queue, cũng là lý do Kafka được xếp vào nhóm message queue.
2. **Lưu trữ message stream bền vững và có fault tolerance:** Kafka persist message vào disk, hiệu quả tránh rủi ro message bị mất.
3. **Stream processing platform:** Xử lý khi publish message; Kafka cung cấp một stream processing library hoàn chỉnh.

Kafka là một distributed system gồm server và client giao tiếp qua high-performance TCP protocol. Kafka có thể được deploy trên bare-metal hardware, virtual machine và container trong local environment hoặc cloud environment.

Trước Kafka 2.8, điểm bị phàn nàn nhiều nhất của Kafka là phụ thuộc nặng vào Zookeeper để quản lý metadata và high availability của cluster. Sau Kafka 2.8, Kafka đưa vào KRaft mode dựa trên Raft protocol, không còn phụ thuộc vào Zookeeper, đơn giản hóa đáng kể architecture của Kafka và cho phép sử dụng Kafka theo cách nhẹ hơn.

Tuy nhiên, cần lưu ý: việc migrate cluster cũ từ ZooKeeper mode sang KRaft mode phải thực hiện theo migration flow chính thức, không thể chỉ sửa config rồi restart. Với cluster mới, nên ưu tiên deploy theo mode được khuyến nghị hiện tại của official documentation.

![](https://oss.javaguide.cn/github/javaguide/high-performance/message-queue/kafka3.3.1-kraft-production-ready.png)

Kafka official website: <http://kafka.apache.org/>

Kafka release history (có thể nhìn trực tiếp để biết project còn được maintain hay không): <https://kafka.apache.org/downloads>

#### RocketMQ

![](https://oss.javaguide.cn/github/javaguide/high-performance/message-queue/rocketmq-logo.png)

RocketMQ là một real-time data processing platform cloud-native cho “message, event và stream” do Alibaba open source, lấy cảm hứng từ Kafka và đã trở thành Apache top-level project.

Các đặc tính cốt lõi của RocketMQ (trích từ official website):

- Cloud-native: sinh ra và phát triển trên cloud, scale up/down đàn hồi không giới hạn, thân thiện với K8s.
- High throughput: bảo đảm throughput cấp trillion, đồng thời đáp ứng các scenario microservice và big data.
- Stream processing: cung cấp stream computing engine nhẹ, có khả năng mở rộng cao, performance cao và nhiều tính năng.
- Cấp độ tài chính: stability cấp tài chính, được dùng rộng rãi trong transaction core flow.
- Architecture tối giản: không phụ thuộc external, architecture Shared-nothing.
- Ecosystem thân thiện: kết nối liền mạch với ecosystem xung quanh như microservice, real-time computing và data lake.

Theo giới thiệu trên official website:

> Kể từ khi ra đời, Apache RocketMQ được nhiều doanh nghiệp, developer và cloud vendor sử dụng rộng rãi nhờ architecture đơn giản, business feature phong phú và khả năng mở rộng rất mạnh. Sau hơn mười năm được kiểm chứng trong các scenario quy mô lớn, RocketMQ đã trở thành solution ưu tiên được ngành công nhận cho business message có reliability cấp tài chính, được dùng rộng rãi trong các business scenario thuộc internet, big data, mobile internet và IoT.

RocketMQ official website: <https://rocketmq.apache.org/> (tài liệu rất chi tiết, khuyến nghị đọc)

RocketMQ release history (có thể nhìn trực tiếp để biết project còn được maintain hay không): <https://github.com/apache/rocketmq/releases>

#### RabbitMQ

![](https://oss.javaguide.cn/github/javaguide/high-performance/message-queue/rabbitmq-logo.png)

RabbitMQ là message middleware triển khai bằng ngôn ngữ Erlang, sử dụng AMQP (Advanced Message Queuing Protocol), ban đầu xuất phát từ financial system và dùng để lưu chuyển tiếp message trong distributed system.

Ngày nay RabbitMQ được ngày càng nhiều người công nhận. Điều này gắn liền với performance nổi bật về ease of use, extensibility, reliability và high availability. Các đặc điểm cụ thể của RabbitMQ có thể tóm tắt như sau:

- **Reliability:** RabbitMQ dùng các mechanism như persistence, transfer confirmation và publish confirmation để bảo đảm reliability của message.
- **Routing linh hoạt:** Trước khi message vào queue, message được route qua exchange. Với các routing feature điển hình, RabbitMQ đã cung cấp một số exchange tích hợp để triển khai. Với routing feature phức tạp hơn, có thể bind nhiều exchange với nhau hoặc dùng plugin mechanism để tự triển khai exchange. Nội dung này sẽ được giới thiệu chi tiết khi nói về core concept của RabbitMQ.
- **Extensibility:** Nhiều RabbitMQ node có thể tạo thành cluster, đồng thời có thể scale node trong cluster động theo business thực tế.
- **High availability:** Trong RabbitMQ 4.x, Classic Queue không còn dùng mirrored queue để thực hiện high availability; với scenario yêu cầu reliability cao, nên ưu tiên Quorum Queue hoặc Streams.
- **Hỗ trợ nhiều protocol:** Ngoài native support cho AMQP, RabbitMQ còn hỗ trợ nhiều message middleware protocol như STOMP và MQTT.
- **Client đa ngôn ngữ:** RabbitMQ hỗ trợ gần như mọi ngôn ngữ phổ biến như Java, Python, Ruby, PHP, C#, JavaScript.
- **Management UI dễ dùng:** RabbitMQ cung cấp UI dễ dùng để user monitoring và quản lý message, node trong cluster. Khi cài RabbitMQ, management UI đã được tích hợp sẵn.
- **Plugin mechanism:** RabbitMQ cung cấp nhiều plugin để mở rộng từ nhiều khía cạnh, và cũng có thể tự viết plugin. Cơ chế này khá giống SPI mechanism của Dubbo.

RabbitMQ official website: <https://www.rabbitmq.com/> .

RabbitMQ release history (có thể nhìn trực tiếp để biết project còn được maintain hay không): <https://www.rabbitmq.com/news.html>

#### Pulsar

![](https://oss.javaguide.cn/github/javaguide/high-performance/message-queue/pulsar-logo.png)

Pulsar là next-generation cloud-native distributed message stream platform, ban đầu do Yahoo phát triển và đã trở thành Apache top-level project.

Pulsar kết hợp message, storage và lightweight functional computing, dùng architecture tách biệt compute và storage, hỗ trợ multi-tenant, persistent storage và data replication cross-region giữa nhiều data center. Pulsar có các đặc tính stream data storage như strong consistency, high throughput, low latency và high extensibility, được xem là solution tốt nhất cho truyền, lưu trữ và tính toán real-time message stream trong cloud-native era.

Các feature chính của Pulsar (trích từ official website):

- Là next-generation cloud-native distributed message stream platform.
- Một instance Pulsar native hỗ trợ nhiều cluster, có thể replication message liền mạch giữa các cluster qua nhiều data center.
- Publish latency và end-to-end latency cực thấp.
- Có thể scale liền mạch lên hơn một triệu topic.
- Client API đơn giản, hỗ trợ Java, Go, Python và C++.
- Nhiều subscribe mode cho Topic (exclusive, shared và failover).
- Cơ chế persistent message storage do Apache BookKeeper cung cấp bảo đảm message delivery.
- Pulsar Functions là lightweight serverless computing framework để xử lý data stream native.
- Pulsar IO là serverless connector framework dựa trên Pulsar Functions, giúp data dễ dàng đi vào và đi ra Apache Pulsar.
- Hierarchical storage có thể unload data từ hot storage sang cold/long-term storage (như S3 và GCS) khi data cũ.

Pulsar official website: <https://pulsar.apache.org/>

Pulsar release history (có thể nhìn trực tiếp để biết project còn được maintain hay không): <https://github.com/apache/pulsar/releases>

#### ActiveMQ

Hiện đã bị loại bỏ, không khuyến nghị sử dụng và không khuyến nghị học.

### Lựa chọn như thế nào?

> Tham khảo “Đột phá phỏng vấn Java Engineer mùa 1 - thầy Zhong Hua Shi Shan”.

| Tiêu chí        | Khái quát                                                                                                                                                                                                                                                                                                            |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Throughput      | Throughput hàng chục nghìn của ActiveMQ và RabbitMQ (ActiveMQ có performance kém nhất) thấp hơn một bậc độ lớn so với RocketMQ và Kafka đạt cấp trăm nghìn hoặc thậm chí triệu.                                                                                                                                      |
| Availability    | Đều có thể thực hiện high availability. ActiveMQ và RabbitMQ dựa trên master-slave architecture để thực hiện high availability. RocketMQ dựa trên distributed architecture. Kafka cũng là distributed system, mỗi data có nhiều replica; một số ít machine down sẽ không làm mất data hoặc khiến system unavailable. |
| Timeliness      | RabbitMQ phát triển dựa trên Erlang nên năng lực concurrency rất mạnh, performance cực tốt và latency rất thấp, đạt cấp microsecond; các hệ thống khác đều ở cấp millisecond.                                                                                                                                        |
| Feature support | Pulsar có feature toàn diện hơn, hỗ trợ multi-tenant, nhiều consumption mode và persistence mode, là next-generation cloud-native distributed message stream platform.                                                                                                                                               |
| Message loss    | Khả năng mất message của ActiveMQ và RabbitMQ rất thấp; về lý thuyết Kafka, RocketMQ và Pulsar có thể đạt mức mất 0 message.                                                                                                                                                                                         |

**Tóm tắt:**

- Community của ActiveMQ tương đối mature, nhưng hiện tại performance của ActiveMQ khá kém và version iteration rất chậm, không khuyến nghị sử dụng, đã bị loại bỏ.
- Throughput của RabbitMQ tuy kém hơn Kafka, RocketMQ và Pulsar, nhưng vì phát triển dựa trên Erlang nên năng lực concurrency rất mạnh, performance cực tốt và latency rất thấp, đạt cấp microsecond. Tuy nhiên, cũng vì RabbitMQ phát triển dựa trên Erlang nên ở Trung Quốc có rất ít công ty đủ năng lực nghiên cứu và customize ở cấp source code Erlang. Nếu business scenario không yêu cầu concurrency quá cao (cấp trăm nghìn hoặc triệu), RabbitMQ có thể là lựa chọn đầu tiên trong các message queue này.
- RocketMQ và Pulsar hỗ trợ strong consistency, phù hợp với các scenario yêu cầu consistency của message cao.
- RocketMQ do Alibaba phát triển, là open source project trong Java ecosystem. Có thể đọc trực tiếp source code để customize MQ cho công ty; đồng thời RocketMQ đã được kiểm chứng thực tế trong business scenario của Alibaba.
- Đặc điểm của Kafka rất rõ: chỉ cung cấp một số core feature ít nhưng có throughput cực cao, latency cấp millisecond, availability và reliability cực cao, đồng thời distributed architecture có thể scale tùy ý. Kafka phù hợp nhất khi số lượng topic ít để bảo đảm throughput cực cao. Nhược điểm duy nhất của Kafka là message có thể bị consume lặp, gây ảnh hưởng rất nhẹ đến data accuracy. Trong big data và log collection, ảnh hưởng nhỏ này có thể bỏ qua; đặc tính này vốn phù hợp với real-time computing và log collection trong big data. Nếu là scenario real-time computing hoặc log collection trong big data, dùng Kafka là standard của ngành, community rất active và gần như là de facto standard của lĩnh vực này trên toàn thế giới.

Khi lựa chọn, có thể trực tiếp hơn:

| Scenario                               | Lựa chọn thường gặp | Điểm cần chú ý                                        |
| -------------------------------------- | ------------------- | ----------------------------------------------------- |
| Log, tracking data, stream processing  | Kafka, Pulsar       | Throughput, partition scaling, ecosystem              |
| Transaction, order, business event     | RocketMQ            | Transaction message, delayed message, ordered message |
| Routing linh hoạt, tích hợp đơn giản   | RabbitMQ            | Exchange routing, confirmation mechanism, queue type  |
| Event flow hiệu năng cao trong process | Disruptor           | Low latency, lock-free, không distributed             |

Không có MQ nào tốt nhất tuyệt đối, chỉ có MQ phù hợp hơn với team, business semantic và năng lực vận hành hiện tại.

## Tham khảo

- “Technology Architecture của website quy mô lớn”
- KRaft: Apache Kafka Without ZooKeeper: <https://developer.confluent.io/learn/kraft/>
- Message queue được sử dụng trong những scenario nào?: <https://mp.weixin.qq.com/s/4V1jI6RylJr7Jr9JsQe73A>

<!-- @include: @article-footer.snippet.md -->
