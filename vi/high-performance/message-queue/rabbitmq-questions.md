---
title: Tổng hợp các câu hỏi thường gặp về RabbitMQ
description: "Tổng hợp câu hỏi phỏng vấn và kiến thức thường gặp về RabbitMQ, bao quát giao thức AMQP, các loại Exchange (Direct/Topic/Fanout), cơ chế xác nhận message, dead-letter queue, delay queue, priority queue, Quorum Queue, Streams và các nội dung khác."
category: High Performance
tag:
  - Message Queue
head:
  - - meta
    - name: keywords
      content: RabbitMQ,AMQP protocol,Exchange types,message confirmation,dead-letter queue,delay queue,priority queue,RabbitMQ cluster,message queue interview
---

Hiện nay không thể chỉ chuẩn bị RabbitMQ theo đáp án cũ kiểu “Exchange + Queue”. RabbitMQ 4.0 đã loại bỏ mirror queue; khi cần replication và high availability, trọng tâm là Quorum Queue; khi cần kiểu lưu trữ log, replay lịch sử hoặc xử lý lượng message tồn đọng lớn, hãy đánh giá Streams.

Bài viết này giới thiệu RabbitMQ 4.x mới nhất làm trọng tâm, đồng thời giữ lại vấn đề mirror queue vẫn có thể gặp trong các cluster 3.x cũ. Hãy tập trung vào bốn điểm: mô hình AMQP hoạt động thế nào, Exchange định tuyến ra sao, bảo đảm độ tin cậy của message thế nào, và nên chọn Classic Queue, Quorum Queue hay Streams ra sao.

## RabbitMQ là gì?

RabbitMQ là message middleware mã nguồn mở được viết bằng Erlang. Giao thức được dùng phổ biến nhất là AMQP 0-9-1; ngoài ra còn hỗ trợ AMQP 1.0, MQTT, STOMP và các giao thức khác.

RabbitMQ đóng vai trò Broker trong hệ thống: producer gửi message đến Exchange, Exchange định tuyến message đến Queue theo các quy tắc binding, sau đó consumer lấy message từ Queue. Ưu thế chính của RabbitMQ nằm ở khả năng routing, cơ chế xác nhận, hệ thống plugin và hệ sinh thái client đa ngôn ngữ.

## Đặc điểm của RabbitMQ

- **Độ tin cậy**: hỗ trợ message persistence, manual Ack của consumer, Publisher Confirms, dead-letter Exchange và các cơ chế khác.
- **Routing linh hoạt**: message đi vào Exchange trước, sau đó Exchange gửi message đến Queue dựa trên routing key và binding. Các trường hợp routing phổ biến chỉ cần Exchange tích hợp sẵn; trường hợp phức tạp có thể kết hợp nhiều Exchange hoặc dùng plugin.
- **Khả năng mở rộng**: nhiều node RabbitMQ có thể tạo thành cluster; khả năng tạo replica của Queue do loại Queue quyết định, không thể chỉ nhìn vào hai chữ “cluster”.
- **High availability**: Quorum Queue replication dữ liệu dựa trên giao thức Raft; Streams cũng là cấu trúc dữ liệu có replication, phù hợp hơn với log và replay.
- **Nhiều giao thức**: ngoài hỗ trợ native AMQP, RabbitMQ còn hỗ trợ nhiều giao thức message middleware như STOMP, MQTT.
- **Client đa ngôn ngữ**: RabbitMQ hỗ trợ gần như mọi ngôn ngữ phổ biến như Java, Python, Ruby, PHP, C#, JavaScript.
- **Giao diện quản trị**: Management UI có thể xem Queue, connection, Channel, Exchange, trạng thái node và các chỉ số vận hành phổ biến.
- **Cơ chế plugin**: RabbitMQ mở rộng các khả năng như MQTT, STOMP và monitoring bằng Prometheus thông qua plugin.

## Các khái niệm quan trọng của RabbitMQ

Nhìn tổng thể, RabbitMQ là mô hình producer và consumer, chủ yếu chịu trách nhiệm nhận, lưu trữ và chuyển tiếp message. Có thể hình dung quá trình truyền message như sau: khi bạn gửi một bưu kiện đến bưu điện, bưu điện tạm lưu bưu kiện rồi cuối cùng chuyển thư đến người nhận qua nhân viên giao thư. RabbitMQ giống một hệ thống gồm bưu điện, hòm thư và nhân viên giao thư. Ở góc độ thuật ngữ máy tính, mô hình RabbitMQ giống mô hình Exchange hơn.

Kiến trúc tổng thể của RabbitMQ như sau:

![Sơ đồ kiến trúc cốt lõi và luồng vòng đời message của RabbitMQ 4.0](https://oss.javaguide.cn/github/javaguide/high-performance/rabbitmq/rabbitmq-core-architecture-and-message-lifecycle-flow.png)

Dưới đây là một số đối tượng chính.

### Producer và Consumer

- **Producer**: bên tạo message (người gửi thư)
- **Consumer**: bên tiêu thụ message (người nhận thư)

Message thường gồm 2 phần: **message header** (hay còn gọi là nhãn Label) và **message body**. Message body cũng có thể gọi là **payload**, không có cấu trúc mà hệ thống phải hiểu; còn message header gồm một loạt thuộc tính tùy chọn như routing-key (routing key), priority (mức ưu tiên so với các message khác), delivery-mode (cho biết message có thể cần được lưu trữ bền vững) và các thuộc tính khác. Sau khi producer giao message cho RabbitMQ, RabbitMQ gửi message đến các Consumer quan tâm dựa trên message header.

### Exchange

Trong RabbitMQ, message không được gửi trực tiếp vào **Queue** mà phải đi qua lớp **Exchange**. **Exchange** phân phối message đến các **Queue** tương ứng.

**Exchange** nhận message do producer gửi rồi routing message đến một hoặc nhiều Queue. Nếu không routing được, message có thể được trả về producer, đi vào Alternate Exchange hoặc bị loại bỏ trực tiếp, tùy thuộc vào publish parameters và cấu hình Exchange.

**RabbitMQ có 4 loại Exchange, mỗi loại tương ứng với một routing strategy khác nhau**: **direct**, **fanout**, **topic** và **headers**. Strategy chuyển tiếp message của mỗi loại Exchange khác nhau. Nội dung này sẽ được giới thiệu trong phần **Exchange Types**.

> Lưu ý: AMQP 0-9-1 có một Default Exchange, là một direct Exchange được khai báo sẵn, có tên là chuỗi rỗng `""`. Khi tạo Exchange cho nghiệp vụ, cần chỉ rõ loại Exchange.

Khi producer gửi message cho Exchange, thường sẽ chỉ định một **RoutingKey** để chỉ ra quy tắc routing message. **RoutingKey phải được dùng kết hợp với loại Exchange và BindingKey thì mới có hiệu lực cuối cùng**.

RabbitMQ liên kết **Exchange** với **Queue** thông qua **Binding**. Khi binding, thường sẽ chỉ định một **BindingKey**; RabbitMQ dựa vào đó để quyết định message đi vào Queue nào. Có thể hiểu một binding là một routing rule; Exchange và Queue có thể có quan hệ nhiều-nhiều.

Khi producer gửi message đến Exchange, thường message sẽ kèm RoutingKey. Khi BindingKey và RoutingKey khớp theo loại Exchange hiện tại, message được routing đến Queue tương ứng. Một Exchange có thể bind nhiều Queue, và nhiều binding cũng có thể dùng cùng một BindingKey. BindingKey có tham gia matching hay không phụ thuộc vào loại Exchange. Ví dụ, fanout Exchange bỏ qua BindingKey và gửi message đến mọi Queue đã binding.

### Queue

**Queue** dùng để lưu message cho đến khi gửi đến consumer. Queue là container của message và cũng là điểm cuối của message. Một message có thể được đưa vào một hoặc nhiều Queue. Message nằm trong Queue, chờ consumer kết nối đến Queue để lấy ra.

Trong kiến trúc truyền thống, **RabbitMQ** chỉ lưu message trong **Queue**, trái với message middleware như **Kafka**. Kafka lưu message ở lớp logic **topic**, còn logic Queue tương ứng chỉ là định danh offset trong file lưu trữ thực tế của topic. Producer của RabbitMQ tạo message rồi cuối cùng gửi message vào Queue; consumer có thể lấy và xử lý message từ Queue.

> RabbitMQ bắt đầu cung cấp Streams từ phiên bản 3.9. Streams dùng append-only log: message không bị xóa khỏi log sau khi được consume, consumer có thể replay message lịch sử theo offset. Các trường hợp như event sourcing, phân phối log và lượng message tồn đọng lớn phù hợp hơn với Streams; còn task bất đồng bộ thông thường vẫn nên ưu tiên Classic Queue hoặc Quorum Queue.

**Nhiều consumer có thể subscribe cùng một Queue**; message trong Queue được phân phối cho một trong các consumer đó xử lý, không phải mỗi consumer đều nhận một bản.

> Lưu ý: hiệu quả phân phối thực tế chịu ảnh hưởng của `prefetch_count`. Trong AMQP 0-9-1, `prefetch_count=0` nghĩa là không giới hạn số message chưa xác nhận; consumer có thể nhận rất nhiều message trong một lần. Khi thời gian xử lý nghiệp vụ không ổn định, nên đặt prefetch phù hợp cho consumer để tránh message dồn hết ở local của một consumer.

**RabbitMQ** không hỗ trợ broadcast consumption ở cấp Queue. Nếu muốn mỗi consumer đều nhận một bản message, cách thường dùng là chuẩn bị Queue riêng cho từng consumer rồi bind các Queue đó vào cùng một fanout hoặc topic Exchange.

### Broker (service node của message middleware)

Với RabbitMQ, một RabbitMQ Broker có thể được hiểu đơn giản là một service node hoặc một service instance của RabbitMQ. Trong phần lớn trường hợp, cũng có thể xem một RabbitMQ Broker là một server RabbitMQ.

### Exchange Types

Các Exchange Type thường dùng của RabbitMQ gồm **fanout**, **direct**, **topic** và **headers** (đặc tả AMQP còn đề cập hai Exchange Type là system và custom; ở đây không mô tả).

![So sánh bốn loại Exchange của RabbitMQ](https://oss.javaguide.cn/github/javaguide/high-performance/rabbitmq/rabbitmq-exchange-types.png)

| Loại    | Routing rule                                                 | Trường hợp thường gặp                                                               |
| ------- | ------------------------------------------------------------ | ----------------------------------------------------------------------------------- |
| fanout  | Bỏ qua RoutingKey, gửi đến mọi Queue đã binding              | Refresh configuration, invalidate cache, phân phối log đồng thời đến nhiều consumer |
| direct  | BindingKey và RoutingKey khớp hoàn toàn                      | Phân phối chính xác theo level, loại nghiệp vụ hoặc tên service                     |
| topic   | RoutingKey được tách theo `.`, BindingKey hỗ trợ `*` và `#`  | Lọc nhiều cấp theo khu vực, module nghiệp vụ hoặc loại event                        |
| headers | Matching theo message headers, hỗ trợ `x-match=all` và `any` | Ít dùng; khi có thể biểu đạt bằng topic thì thường không chọn headers               |

Hai wildcard của topic dễ bị nhầm: `*` chỉ khớp một từ, `#` có thể khớp không hoặc nhiều từ. Ví dụ `order.china.*` khớp `order.china.beijing` nhưng không khớp `order.china.beijing.created`; `#.client.#` có thể khớp `com.rabbitmq.client`.

## AMQP là gì?

RabbitMQ ban đầu được xây dựng xoay quanh AMQP 0-9-1. Các khái niệm producer, Exchange, Queue, binding và routing key đều xuất phát từ mô hình AMQP. RabbitMQ cũng hỗ trợ các giao thức như AMQP 1.0, MQTT và STOMP; chi tiết chức năng khi kết nối bằng mỗi giao thức sẽ khác nhau.

Các khái niệm Exchange, Exchange Type, Queue, binding, routing key trong RabbitMQ đều tuân theo **các khái niệm tương ứng** trong giao thức AMQP.

> Từ RabbitMQ 4.0, AMQP 1.0 được hỗ trợ native và bật mặc định, không còn phụ thuộc vào plugin AMQP 1.0 cũ để chuyển đổi giao thức. AMQP 0-9-1 vẫn tiếp tục được hỗ trợ; hệ sinh thái Java, Spring AMQP hiện vẫn chủ yếu xoay quanh giao thức này. Việc dự án mới có chọn AMQP 1.0 hay không phụ thuộc vào mức độ hoàn thiện của client library, nhu cầu interoperability và mức hỗ trợ thực tế của RabbitMQ đối với AMQP 1.0.

**Trong AMQP 0-9-1 cần phân biệt AMQ model và protocol layer**:

- **AMQ model**: định nghĩa các object cốt lõi như Exchange, Queue, binding và cách message được routing từ producer đến consumer.
- **Functional Layer**: định nghĩa các protocol command được nhóm theo logical class, như exchange, queue, basic, tx.
- **Transport Layer**: chịu trách nhiệm xử lý frame, tái sử dụng Channel, heartbeat, xử lý lỗi, biểu diễn dữ liệu và các công việc khác.

**Ba thành phần chính của mô hình AMQP**:

- **Exchange**: component trong message broker server dùng để routing message đến Queue.
- **Queue**: cấu trúc dữ liệu dùng để lưu message, nằm trên disk hoặc trong memory.
- **Binding**: một tập hợp rule cho Exchange biết cần gửi message đến Queue nào.

## Producer và Consumer hoạt động thế nào?

**Producer**:

- Message producer là bên gửi message.
- Message thường gồm hai phần: **message body** (payload) và **message header** (Label/Headers).

**Consumer**:

- Consume message, tức là bên nhận message.
- Consumer kết nối đến RabbitMQ server và subscribe Queue. Nghiệp vụ thường xử lý message body; routing key, headers và delivery tag là metadata của routing hoặc delivery, có thể dùng cho log, idempotency và xử lý xác nhận, nhưng không nên xem chúng là message body của nghiệp vụ.

## Broker service node, Queue và Exchange là gì?

- **Broker**: có thể xem là service node của RabbitMQ. Thông thường một Broker có thể xem là một server RabbitMQ.
- **Queue**: object bên trong RabbitMQ, dùng để lưu message. Nhiều consumer có thể subscribe cùng một Queue; khi đó message trong Queue được chia đều (round-robin) cho nhiều consumer xử lý.
- **Exchange**: producer gửi message đến Exchange, Exchange routing message đến một hoặc nhiều Queue. Khi không routing được, message sẽ được trả về producer hoặc bị loại bỏ trực tiếp.

## Dead-letter queue là gì? Nguyên nhân nào tạo ra dead letter?

DLX là viết tắt của `Dead-Letter-Exchange` (dead-letter Exchange). Khi message trong một Queue trở thành dead message, nó có thể được gửi lại đến một Exchange khác. Exchange này là DLX, còn Queue bind với DLX được gọi là dead-letter queue.

**Các nguyên nhân thường gặp tạo ra dead letter**:

- Message bị reject (`Basic.Reject` hoặc `Basic.Nack`) và `requeue = false`.
- Message hết hạn TTL.
- Queue đạt giới hạn length và message bị loại bỏ.
- Trong Quorum Queue, số lần message được trả lại vượt quá `delivery-limit`.

## Delay queue là gì? RabbitMQ triển khai delay queue thế nào?

Delay queue lưu delay message: message đã được gửi đến RabbitMQ nhưng nghiệp vụ muốn consumer chỉ nhận nó sau một khoảng thời gian.

RabbitMQ bản thân không có delay queue. Để triển khai delay message, thường có hai cách:

1. Dùng TTL + DLX. Message trước tiên đi vào Queue có TTL, sau khi hết hạn được gửi đến dead-letter Exchange rồi đi vào Queue xử lý thực sự. Nhược điểm là dễ bị ảnh hưởng bởi việc block ở đầu Queue; nếu tạo một nhóm Queue cho mỗi thời gian delay, chi phí bảo trì cũng tăng.
2. Dùng plugin `rabbitmq-delayed-message-exchange`. Plugin cung cấp Exchange `x-delayed-message`, cho phép thiết lập thời gian delay theo từng message. README chính thức định vị plugin cho delay ở mức giây, phút, giờ, tối đa một hoặc hai ngày; nếu cần scheduling ở mức ngày, tuần, tháng, hoặc cần dồn 100 nghìn hay hàng triệu delay message, nên dùng hệ thống lưu trữ và scheduling bên ngoài.

Nói cách khác, delay message phổ biến của RabbitMQ không phải native capability của Queue thông thường. TTL + DLX và delay plugin đều có thể thực hiện việc này, nhưng cần cân nhắc quy mô delay và yêu cầu về khả năng khôi phục.

## Priority queue là gì?

RabbitMQ hỗ trợ priority của message, không phân biệt priority của Queue. Priority queue nghĩa là trong cùng một Queue, message được gửi theo priority; message có priority cao được đưa đến consumer sớm hơn.

Classic Queue có thể khai báo priority queue thông qua parameter `x-max-priority`; Quorum Queue cũng hỗ trợ priority. Cần lưu ý rằng nếu tốc độ consume luôn cao hơn tốc độ produce, Queue không bị tồn đọng message thì priority rất khó thể hiện.

## RabbitMQ có những work mode nào?

- Simple mode
- Work mode
- Pub/sub mode
- Routing mode
- Topic mode

## Message được truyền trong RabbitMQ thế nào?

Do chi phí tạo và hủy TCP connection khá lớn (three-way handshake, slow start...), đồng thời số connection đồng thời cũng bị giới hạn bởi tài nguyên hệ thống, RabbitMQ dùng Channel để tái sử dụng TCP connection. Channel là virtual communication channel được xây dựng trên TCP connection.

> Lưu ý:
>
> - Một TCP connection có thể mang nhiều Channel, nhưng tài liệu chính thức khuyến nghị không quá 100-200 Channel mỗi connection.
> - Mỗi Channel có số hiệu riêng nhưng dùng chung flow control của TCP connection.
> - **Channel không thread-safe**, multi-thread nên dùng các Channel instance khác nhau.

## Làm thế nào để bảo đảm độ tin cậy của message?

![Toàn cảnh độ tin cậy của message và kiến trúc Queue trong RabbitMQ 4.0](https://oss.javaguide.cn/github/javaguide/high-performance/rabbitmq/rabbitmq-message-reliability-and-queue-architecture-overview.png)

Message có thể gặp lỗi ở ba giai đoạn: từ producer đến Broker, trong thời gian Broker lưu trữ và từ Broker đến consumer.

**1. Từ producer đến Broker**

Ở phía producer thường phải xử lý đồng thời hai việc: “Broker đã nhận message hay chưa” và “message đã được routing thành công đến Queue hay chưa”:

- **Publisher Confirms**: xác nhận message đã được Broker nhận hay chưa. Với persistent message được routing đến persistent Queue, confirm sẽ chờ đến khi message được lưu bền vững; với Quorum Queue, confirm sẽ chờ đến khi đa số replica chấp nhận.

  ```java
  channel.confirmSelect();
  channel.addConfirmListener((sequenceNumber, multiple) -> {
      // Broker đã xử lý message tương ứng với sequence number này
  }, (sequenceNumber, multiple) -> {
      // Broker không thể xử lý message này, ghi log và retry theo business policy
  });
  ```

- **mandatory + Return Listener**: khi message đến Exchange nhưng không có Queue khớp, producer có thể nhận được return.

  ```java
  channel.basicPublish("exchange", "routingKey",
      true,  // mandatory=true
      null,
      messageBody);

  channel.addReturnListener((replyCode, replyText, exchange, routingKey, properties, body) -> {
      // Message đã đến Exchange nhưng không routing đến Queue nào
      log.error("Message returned: {}", replyText);
  });
  ```

Chỉ bật Confirm là chưa đủ. Message routing thất bại vẫn có thể nhận `basic.ack`, vì Broker đã xử lý lần publish này; muốn bảo đảm message đi vào business Queue thì phải dùng mandatory return hoặc Alternate Exchange.

- **Transaction mechanism** (không khuyến nghị): synchronous blocking, throughput thường kém hơn Publisher Confirms nhiều.
  - Lưu ý: transaction mechanism và Confirm mechanism loại trừ lẫn nhau, không thể dùng đồng thời.

**2. Trong thời gian Broker lưu trữ**

- `delivery_mode=2`: xử lý message dưới dạng persistent message.
- `durable=true`: metadata của Queue có thể được khôi phục sau khi Broker restart.
- Replicated Queue: trong RabbitMQ 4.x, mirror queue đã bị loại bỏ; khi cần replication, chủ yếu cân nhắc Quorum Queue hoặc Streams.

Chỉ thiết lập persistence không có nghĩa message chắc chắn không bị mất. Producer còn phải chờ Publisher Confirm, consumer cũng phải dùng manual Ack; nếu không, các tình huống như Broker crash, connection bị ngắt hoặc consume thất bại vẫn có thể gây mất dữ liệu hoặc consume trùng.

**3. Từ Broker đến consumer**

- **Manual Ack**: `basicAck(deliveryTag, multiple)`, chỉ xác nhận sau khi consume thành công.
- **Retry mechanism**: khi consume thất bại, có thể dùng `basicNack` hoặc `basicReject`, sau đó quyết định có `requeue` hay không tùy loại exception.
- **Dead-letter queue**: sau khi đạt số lần retry tối đa hoặc bị reject, message được routing đến DLQ để cảnh báo, bù dữ liệu hoặc xử lý thủ công về sau.
- **Bảo đảm idempotency**: thực hiện ở business layer để tránh dữ liệu không nhất quán do consume trùng. Về các cách triển khai idempotency cụ thể, xem bài viết [Tổng hợp các giải pháp idempotency cho interface](https://javaguide.cn/high-availability/idempotency.html).

> Lưu ý: Alternate Exchange cũng có thể xử lý routing failure. Sau khi cấu hình Alternate Exchange, message không routing được sẽ được chuyển đến đó; chỉ khi Alternate Exchange cũng không routing được và message đã đặt mandatory thì producer mới nhận được return.

## Làm thế nào để bảo đảm thứ tự của message trong RabbitMQ?

FIFO của RabbitMQ chỉ đúng trong một Queue và còn chịu ảnh hưởng của số consumer, prefetch, retry và requeue. Có ba cách xử lý phổ biến:

**1. Single Consumer mode**: một Queue chỉ bind một consumer. Cách này dễ bảo đảm thứ tự nhất, nhưng throughput cũng dễ trở thành bottleneck nhất.

**2. Ordered partitioning**: hash theo business key (như order ID) đến các Queue khác nhau, mỗi Queue do một consumer độc lập xử lý. Khi cùng một business key luôn đi vào cùng một Queue, có thể tăng throughput mà vẫn giữ thứ tự cục bộ.

Partitioning có hai vấn đề: scale Queue sẽ làm thay đổi kết quả hash, khiến message mới và cũ của cùng một business key có thể đi vào các Queue khác nhau; sau khi consume thất bại và requeue, thứ tự delivery tiếp theo cũng có thể thay đổi. Với nghiệp vụ yêu cầu thứ tự chặt, nên thêm state machine, version hoặc unique constraint trong business table, không chỉ dựa vào thứ tự delivery của MQ.

**3. Xếp hàng bên trong consumer**: một consumer nhận message trước, sau đó phân phối theo business key đến local memory queue và Worker thread. Cách này phải tự xử lý việc dồn message trong memory, mất message khi process crash, thời điểm Ack và backpressure; cần thận trọng khi dùng trong production.

## Làm thế nào để bảo đảm high availability cho RabbitMQ?

High availability của RabbitMQ cần được xem xét ở hai tầng: cluster chỉ giúp nhiều node cùng quản lý metadata và connection; message trong Queue có được replication hay không phụ thuộc vào loại Queue.

Trong RabbitMQ 4.x, Classic Queue là Queue không replication; mirror queue đã bị loại bỏ; các cấu trúc dữ liệu có replication chủ yếu là Quorum Queue và Streams. Chỉ khi còn sử dụng cluster 3.x cũ mới gặp vấn đề bảo trì mirror queue.

Network partition cũng cần được xử lý riêng. Cluster thông thường và mirror queue cũ đều có thể bị ảnh hưởng bởi network instability. Strategy phổ biến là cấu hình `cluster_partition_handling = pause_minority`, cho minority node tạm dừng service để tránh hai phía tiếp tục ghi độc lập. Quorum Queue dùng Raft nên consistency rõ ràng hơn, nhưng cũng cần đa số replica khả dụng; không thể hiểu rằng nó “có thể tiếp tục service khi gặp mọi lỗi network”.

**Single-node mode**

Dành cho demo; thường chỉ là khởi động RabbitMQ trên máy local để thử, không ai dùng single-node mode trong production.

**Cluster mode thông thường**

Nghĩa là khởi động nhiều RabbitMQ instance trên nhiều máy, mỗi máy chạy một instance. Queue bạn tạo chỉ nằm trên một RabbitMQ instance, nhưng mỗi instance đều đồng bộ metadata của Queue (có thể hiểu metadata là một số thông tin cấu hình của Queue; thông qua metadata có thể tìm được instance đang chứa Queue).

Khi consume, nếu kết nối đến một instance khác, instance đó thực tế sẽ lấy dữ liệu từ instance chứa Queue. Cách này chủ yếu dùng để tăng throughput, cho phép nhiều node trong cluster phục vụ thao tác đọc và ghi của một Queue.

**Mirror cluster mode** (Classic Queue Mirroring, đã bị loại bỏ)

Mirror queue đã bị loại bỏ trong RabbitMQ 4.0. RabbitMQ 3.8 giới thiệu Quorum Queue làm giải pháp thay thế; phiên bản 3.13 vẫn có thể dùng mirror queue nhưng tính năng này đã deprecated. Không nên chọn mirror queue cho dự án mới.

Đây là giải pháp high availability của các phiên bản RabbitMQ trước đây. Khác với cluster mode thông thường, trong mirror cluster mode, cả Queue metadata lẫn message trong Queue đều tồn tại trên nhiều instance. Mỗi RabbitMQ node có một mirror hoàn chỉnh của Queue, chứa toàn bộ dữ liệu của Queue. Mỗi lần ghi message vào Queue, message sẽ tự động được đồng bộ đến Queue trên nhiều instance.

Cách hoạt động đại khái như sau:

- Queue master node nhận message rồi đồng bộ đến N mirror node.
- Khi master node crash, mirror node cũ nhất được nâng cấp thành master node.
- Thêm policy trong management console để chỉ định dữ liệu được đồng bộ đến tất cả node hoặc một số node cụ thể.

Ưu điểm:

- Khi bất kỳ máy nào crash, các node khác vẫn chứa đầy đủ dữ liệu của Queue.
- Consumer có thể chuyển sang node khác để tiếp tục consume.

Nhược điểm:

- Chi phí performance lớn vì message phải được đồng bộ đến mọi máy.
- Áp lực và mức tiêu thụ network bandwidth cao.
- Không phải kiến trúc distributed thực sự mà là master-slave replication.

**Quorum Queue** (3.8+)

Quorum Queue là replicated Queue dựa trên giao thức Raft, phù hợp với Queue tồn tại lâu dài và yêu cầu cao về an toàn dữ liệu:

- **Dựa trên giao thức Raft**: thực hiện consistency thông qua replication log và election.
- **Quorum write**: cần đa số node xác nhận (N/2 + 1) mới xem là ghi thành công.
- **Replication semantics rõ ràng hơn**: dễ xử lý master node failure và network partition hơn mirror queue.
- **Trường hợp sử dụng**: business flow như order, payment và inventory deduction, nơi không thể dễ dàng để mất message.

Quorum Queue không phù hợp với mọi trường hợp. Với temporary Queue, Queue được tạo và xóa thường xuyên, yêu cầu latency cực thấp, lượng message tồn đọng lớn trong thời gian dài (đặc biệt từ hàng triệu message trở lên) và fanout quy mô lớn, cần ưu tiên đánh giá Classic Queue hoặc Streams. Tài liệu chính thức cũng khuyến nghị thực hiện upgrade và failure drill bất kể số lượng Queue.

**Cách khai báo ở client**:

Java:

```java
Map<String, Object> args = new HashMap<>();
args.put("x-queue-type", "quorum");
channel.queueDeclare("my-queue", true, false, false, args);
```

Python:

```python
channel.queue_declare(
    queue='my-queue',
    durable=True,
    arguments={'x-queue-type': 'quorum'}
)
```

> `x-queue-type` phải được cung cấp khi khai báo Queue, không thể sửa sau đó bằng Policy. Sau khi loại Queue đã được xác định, không thể trực tiếp đổi classic queue hiện có thành quorum queue.

## Giải quyết vấn đề message queue bị delay và hết hạn thế nào?

RabbitMQ có thể thiết lập thời gian hết hạn của message (TTL). Nếu message nằm trong Queue lâu hơn TTL, message sẽ hết hạn; khi đã cấu hình DLX, message sẽ đi vào dead-letter Exchange, nếu không thì bị loại bỏ.

Nếu dữ liệu có thể khôi phục từ database hoặc source khác, có thể dùng batch re-import để bù dữ liệu:

1. Trong giờ cao điểm, trước tiên loại bỏ dữ liệu không thể xử lý kịp để giữ availability của hệ thống.
2. Trong giờ thấp điểm, viết chương trình tạm thời để query dữ liệu thiếu từ database.
3. Gửi lại dữ liệu đã query vào MQ để consumer xử lý bù.

**Ví dụ**:

- Giả sử có 10.000 order đang tồn đọng chưa được xử lý trong MQ.
- Trong đó 1.000 order bị loại bỏ do hết hạn TTL.
- Cách xử lý: query 1.000 order này từ database rồi gửi lại vào MQ để bù dữ liệu.

Cách này có điều kiện tiên quyết: database phải có đầy đủ dữ liệu lịch sử, quá trình consume bù phải bảo đảm idempotency, và lượng message tồn đọng phải có monitoring alert. Nếu không, chương trình bù tạm thời rất dễ xử lý lại dữ liệu trùng hoặc dữ liệu bẩn.

## Production cần quan tâm những chỉ số nào?

**1. Memory watermark**

- Monitoring tỷ lệ sử dụng `rabbitmq_memory_limit`.
- Ngưỡng cảnh báo: watermark mặc định là 0.4 (40%).
- Ảnh hưởng: khi đạt watermark, RabbitMQ sẽ block publish connection; việc ghi của producer chậm lại, thậm chí dừng hẳn.
- Cấu hình đề xuất:

  ```ini
  vm_memory_high_watermark.relative = 0.4
  vm_memory_high_watermark_paging_ratio = 0.5
  ```

**2. Mức tiêu thụ file descriptor**

- Monitoring tỷ lệ sử dụng File Descriptors.
- Khi số connection tăng đột biến, việc cạn file descriptor sẽ khiến connection mới thất bại; trường hợp nghiêm trọng còn ảnh hưởng đến độ ổn định của node.
- Với môi trường có nhiều connection, cần tăng giới hạn hệ thống trước, ví dụ `ulimit -n 100000`.

**3. Tốc độ tạo và hủy Channel**

- Monitoring tốc độ tạo và hủy Channel.
- Tạo và hủy Channel với tần suất cao gây thêm chi phí CPU và Erlang process.
- Nên tái sử dụng Channel nhưng cũng không nên dồn vô hạn trên một connection; thông thường kiểm soát trong khoảng vài chục đến một hoặc hai trăm.

**4. Độ sâu message tồn đọng**

- Monitoring số lượng message trong Queue và Consumer Lag.
- Ngưỡng cảnh báo: do nghiệp vụ định nghĩa (ví dụ > 10.000 message).
- Công cụ: RabbitMQ Management UI, Prometheus + Grafana.

**5. Disk space và I/O**

- Monitoring disk space còn lại và IOPS.
- Ngưỡng cảnh báo: cảnh báo khi disk space còn lại < 20%.
- Quorum Queue yêu cầu disk I/O cao hơn; nên dùng NVMe SSD.

## Những hiểu lầm thường gặp khi sử dụng RabbitMQ

**Hiểu lầm 1: Mọi Queue đều dùng Quorum Queue**

Quorum Queue replication message đến nhiều replica và xác nhận ghi theo semantics của Raft. Nó phù hợp với Queue có độ tin cậy cao, nhưng không phù hợp với temporary Queue, latency cực thấp, nhiều Queue có vòng đời ngắn, lượng message tồn đọng rất lớn và fanout quy mô lớn. Khi ưu tiên throughput, có thể đánh giá non-replicated Classic Queue hoặc Streams; khi ưu tiên reliability, hãy cân nhắc Quorum Queue.

**Hiểu lầm 2: Prefetch Count càng lớn càng tốt**

Prefetch quá lớn khiến consumer lấy trước rất nhiều message. Queue phía server trông có vẻ không tồn đọng, nhưng message đều bị kẹt ở trạng thái Unacked tại local của consumer, consumer khác không thể nhận được, đồng thời memory của client cũng có thể bị làm cạn.

Có thể bắt đầu bằng một giá trị thận trọng rồi điều chỉnh qua load test dựa trên thời gian xử lý và throughput:

```java
channel.basicQos(20);
```

**Hiểu lầm 3: Delay queue plugin có thể dùng thay cho hệ thống scheduled task**

`rabbitmq-delayed-message-exchange` phù hợp với delay ngắn, không phù hợp với scheduling dài hạn, cũng không phù hợp với việc dồn 100 nghìn hoặc hàng triệu delay message. Với delay task quy mô lớn, nên lưu trạng thái scheduling trong database, Redis, timing wheel hoặc scheduling system, rồi mới gửi message vào MQ khi đến thời điểm.

**Hiểu lầm 4: Network partition sẽ không xảy ra trong môi trường của chúng ta**

Triển khai cross-data-center, cross-availability-zone và việc switch network chập chờn đều có thể gây network partition. Cluster thông thường và mirror queue cũ cần cấu hình strategy khôi phục partition; khi cần replicated Queue, nên ưu tiên đánh giá Quorum Queue, đồng thời xác nhận đa số replica có thể nằm trong topology availability zone đáng tin cậy.

**Hiểu lầm 5: Bật transaction mechanism thì không cần Publisher Confirms**

Transaction mechanism là synchronous blocking mode; tài liệu chính thức cũng nói rõ cơ chế này làm throughput giảm đáng kể. Phía producer thường ưu tiên Publisher Confirms, kết hợp mandatory return hoặc Alternate Exchange để xử lý routing failure.
