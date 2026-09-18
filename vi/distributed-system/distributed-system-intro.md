---
title: "Giải thích chi tiết về hệ thống phân tán: khái niệm cốt lõi, tiến hóa kiến trúc, đặc điểm điển hình và lộ trình học"
description: "Giải thích nhập môn về hệ thống phân tán, tìm hiểu hệ thống phân tán là gì, vì sao cần hệ thống phân tán, quá trình tiến hóa từ monolith đến kiến trúc phân tán, các đặc điểm điển hình, loại phổ biến, điểm khó cốt lõi và lộ trình học."
category: Distributed Systems
tag:
  - Distributed Systems
  - System Design
head:
  - - meta
    - name: keywords
      content: hệ thống phân tán,hệ thống phân tán là gì,kiến trúc phân tán,microservices,cluster,remote call,distributed consistency,data sharding,replica replication,lộ trình học hệ thống phân tán
---

Khi mới tiếp xúc với hệ thống phân tán, nhiều người sẽ bị dội vào một loạt thuật ngữ: CAP, BASE, Paxos, Raft, distributed lock, distributed transaction.

Những khái niệm này đều không thể tránh, nhưng nếu mới nhập môn đã đi thẳng vào chúng, bạn rất dễ học hệ thống phân tán thành một đống thuật ngữ không liên quan với nhau. Một điểm bắt đầu tốt hơn là một câu hỏi mộc mạc nhưng thực tế hơn: Việc vốn có thể hoàn thành bằng một máy, một process và một database, vì sao về sau phải tách sang nhiều máy? Sau khi tách, vì sao một timeout, một lần retry hay một message bị gửi trùng lại kéo theo nhiều vấn đề thiết kế đến vậy?

Bài viết này trước hết sẽ giải thích rõ “hệ thống phân tán là gì”. Paxos, Raft và distributed transaction sẽ không được suy luận chi tiết; chúng chỉ được đặt lại vào mạch chính để bạn biết đại khái chúng đang giải quyết loại vấn đề nào.

## Hệ thống phân tán là gì?

Cuốn sách 《Distributed Systems: Concepts and Design》 định nghĩa hệ thống phân tán như sau:

> Hệ thống phân tán là một hệ thống trong đó các component phần cứng hoặc phần mềm nằm trên những máy tính được kết nối mạng chỉ giao tiếp và phối hợp hành động thông qua việc truyền message.

Trong engineering, có thể hiểu như sau: **Hệ thống phân tán gồm nhiều computing unit tương đối độc lập. Các computing unit này giao tiếp và phối hợp qua network để cung cấp một service hoàn chỉnh ra bên ngoài.**

Đừng hiểu “độc lập” ở đây là mỗi node đều độc chiếm CPU, memory, disk và operating system. Node có thể là physical machine, virtual machine, container hoặc chỉ là một software process. Hai process trên cùng một machine sẽ share CPU bên dưới và physical memory; container cũng thường share kernel của host machine. Nói chúng độc lập chủ yếu có nghĩa là chúng có execution state và local data riêng, có thể chạy độc lập, cũng có thể bị treo, restart hoặc mất connection riêng.

Rắc rối cũng bắt đầu từ đây.

Các node không share cùng một process address space. Local method call ban đầu sau khi được tách ra có thể trở thành một RPC, một HTTP request, một message delivery hoặc một lần đồng bộ database replica. Network có latency và có thể mất packet; request có thể đã đến server nhưng response bị mất. “Timeout” mà client nhìn thấy không thể chứng minh server chưa thực thi.

Nhiều machine còn mang đến một giới hạn khác: không node nào có thể nhìn thấy trạng thái thực của toàn hệ thống tại một thời điểm. Mỗi node nhìn thấy local state và những message đã nhận. Clock của các node cũng có sai lệch; ngay cả khi dùng NTP để đồng bộ, bạn cũng không thể xem đó là một global clock hoàn toàn nhất quán.

Ví dụ, người dùng click “Gửi đơn hàng”. Trên page đó chỉ là một request, nhưng server có thể đã đi qua gateway, user service, product service, order service, inventory service, coupon service và payment service; đồng thời còn có thể ghi database, gửi message và cập nhật cache. Người dùng nhìn thấy một button, còn backend nhìn thấy một chuỗi phối hợp giữa các node.

![Tổng quan hệ thống phân tán](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/distributed-system-overview.webp)

| Khía cạnh          | Hệ thống single machine hoặc single process | Hệ thống phân tán                                       |
| ------------------ | ------------------------------------------- | ------------------------------------------------------- |
| Cách giao tiếp     | Method call, shared memory                  | RPC, message, network protocol                          |
| Phạm vi failure    | Thường share một failure boundary           | Node có thể failure độc lập                             |
| Góc nhìn thời gian | Chủ yếu dựa vào clock của cùng một machine  | Clock giữa nhiều node có sai lệch                       |
| Quan sát state     | Tương đối dễ quan sát toàn bộ state         | Node thường chỉ có local view                           |
| Xử lý transaction  | Local database transaction phổ biến hơn     | Phối hợp giữa service, compensation hoặc consensus      |
| Cách scale         | Chủ yếu scale up                            | Tăng node theo chiều ngang                              |
| Điều tra vấn đề    | Log và call stack của một process           | Log, metric, distributed tracing và state giữa các node |

## Vì sao cần hệ thống phân tán?

Monolith không hề thấp kém. Với một management backend hoặc một business system có traffic không lớn, dùng một application và một database lại dễ phát triển, deploy và điều tra vấn đề hơn. Nhiều system thực sự gặp vấn đề không phải vì ban đầu dùng monolith, mà vì tách quá sớm khiến complexity tăng trước khi lợi ích xuất hiện.

Hệ thống phân tán thường chỉ trở nên cần thiết sau khi xuất hiện pressure.

**Đầu tiên là pressure về computing.** CPU, memory và disk I/O của một machine đều có giới hạn. Có thể nâng cấu hình machine, nhưng price, hardware specification và single-point risk sẽ cản trở scale up. Phân tán request lên nhiều machine mới là hướng đi mà phần lớn business system sẽ chọn về sau.

**Storage pressure cũng tương tự.** Một order table tăng từ 1 triệu row lên 1 tỷ row sẽ khiến query, backup, index maintenance và failure recovery đều nặng hơn. Nếu tiếp tục dồn toàn bộ data lên một machine, cost và risk sẽ tăng. Data sharding tách các data khác nhau sang các node khác nhau, chẳng hạn shard theo user ID hoặc order ID; replica replication lưu nhiều bản của cùng một data để tăng availability, khả năng disaster recovery và đồng thời chia sẻ một phần read request. Chúng giải quyết vấn đề về capacity và failure, nhưng cũng kéo theo cross-shard query, replica synchronization và data consistency.

**Availability cũng buộc system phải đi theo hướng nhiều node.** Nếu chỉ có một application server, server hỏng thì service dừng; nếu chỉ có một database, disk hỏng hoặc host failure sẽ ảnh hưởng trực tiếp đến business. Nhiều instance, nhiều replica và nhiều availability zone ít nhất có thể giúp system tiếp tục service khi một phần node gặp vấn đề, hoặc giữ lại một phần capability.

Đặt trong business, các động lực phổ biến thường là:

- **Performance không chịu nổi**: single machine không xử lý được nhiều request như vậy, cần nhiều machine chia sẻ computing.
- **Không chứa hết data**: cost của storage, index, backup và recovery trên single machine quá cao, cần shard hoặc replica.
- **Không chống đỡ được failure**: single-point failure ảnh hưởng quá lớn, cần redundancy, failover và degradation.

Cũng có những system không bị capacity của single machine ép buộc, mà bị địa lý, tổ chức và security boundary thúc đẩy. Deploy service và data ở khu vực gần user hơn có thể giảm access latency, đồng thời thuận tiện cho disaster recovery giữa các region; deploy độc lập giữa các team, business domain hoặc security domain có thể tránh việc mọi function share cùng một release window và failure boundary.

Vì vậy, **distributed không chỉ đơn giản là “thêm machine”. Nó giải quyết vấn đề về capacity, availability, isolation và collaboration, đồng thời đưa network, failure, data consistency và cost điều tra vấn đề vào system.**

## Từ e-commerce monolith đến e-commerce phân tán

Giả sử có một e-commerce system thời kỳ đầu, trong đó user, product, order, inventory và payment đều được viết trong một Spring Boot application; data được lưu trong cùng một MySQL instance.

Khi business mới bắt đầu, cấu trúc này rất thoải mái. Một lần đặt hàng chỉ là một local call chain: validate user, query product, trừ inventory, tạo order và bắt đầu payment. Code nằm trong một process, transaction hoàn thành trong một database. Khi có vấn đề, chỉ cần xem log của một application và một database là cơ bản có thể điều tra rõ.

Sau khi traffic tăng, pressure trước tiên sẽ rơi vào một số nơi: lượng query của product detail page cao, lượng write khi tạo order cao, xung đột concurrency khi trừ inventory nhiều, còn payment chain thì không thể tùy tiện failure. Nếu tiếp tục nhồi toàn bộ logic vào một application, bất kỳ module nào chậm cũng có thể kéo cả system lại.

Lúc này nhiều team sẽ scale theo chiều ngang trước: deploy 3 hoặc nhiều application instance hơn, dùng Nginx, gateway hoặc load balancer để phân phối request. Chỉ cần application cố gắng stateless, thêm instance là có thể chia sẻ một phần traffic.

Về sau, system có thể tiếp tục được tách:

- Product service phụ trách product information và hiển thị price;
- Order service phụ trách tạo order và chuyển đổi order state;
- Inventory service phụ trách trừ inventory, rollback inventory và inventory flow;
- Payment service phụ trách kết nối với third-party payment channel;
- Message queue phụ trách truyền bất đồng bộ các event như payment thành công, inventory thay đổi và notification logistics.

![Từ monolith đến e-commerce phân tán](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/monolith-to-distributed-ecommerce.webp)

Lợi ích của việc tách rất trực tiếp. Product service có traffic lớn có thể scale riêng; payment service yêu cầu stability cao có thể thiết kế rate limiting, retry và circuit breaker riêng; inventory service có nhiều concurrency conflict có thể thiết kế data structure và lock strategy chuyên biệt xoay quanh việc trừ inventory.

Rắc rối cũng nhanh chóng xuất hiện.

Order service gọi inventory service để trừ inventory, nếu request timeout thì order service có nên retry không? Lần trừ inventory trước là chưa gửi đi, hay đã trừ thành công nhưng response bị mất? Nếu trừ inventory thành công nhưng tạo order thất bại thì phải bù inventory thế nào? Payment success message bị gửi trùng, order state có bị update trùng không?

Những vấn đề này cũng có thể xuất hiện trong monolith, chỉ là hệ thống phân tán sẽ phóng đại chúng. System không còn chỉ có một process, một memory và một transaction context. Nhiều việc trước đây “tiện tay làm luôn”, sau khi tách ra đều phải thiết kế lại.

## Hệ thống phân tán có những đặc điểm điển hình nào?

Chỉ nhìn số lượng machine là chưa đủ. Những đặc điểm dưới đây mới là nguồn gốc của complexity trong hệ thống phân tán.

### Phối hợp nhiều node

Một request thường cần nhiều node cùng hoàn thành. Request đặt hàng có thể đi qua gateway, order service, inventory service, payment service, message queue và database. Mỗi node chỉ phụ trách một đoạn logic nhỏ; ghép lại mới thành một business chain hoàn chỉnh.

Khi chain dài ra, latency và failure đều bị phóng đại. Thread pool của một node nào đó bị đầy, database slow query hoặc network chập chờn, thứ user nhìn thấy có thể chỉ là “đặt hàng quay mãi”.

### Node thực thi concurrency, không có global view tức thời

Các node chạy đồng thời. Mỗi node chỉ có thể trực tiếp nhìn thấy local state của mình và những message đã nhận, không thể đọc trạng thái thực của toàn system tại một thời điểm.

Vì vậy sẽ xuất hiện những phán đoán có vẻ mâu thuẫn nhưng đều có thể giải thích: một node đã nhận được config mới nhất, node khác vẫn dừng ở version cũ; một node cho rằng Leader vẫn còn sống, node khác vì timeout đã bắt đầu election. Nhiều vấn đề phân tán không nhất thiết do code viết sai, mà do các node khác nhau nhìn thấy thông tin khác nhau ở những thời điểm khác nhau.

### Network communication

Các node phải trao đổi data qua network. Network không giống local memory: nó không đảm bảo request chắc chắn đến nơi và cũng không đảm bảo response trả về trong thời gian dự kiến.

Một remote call có thể xảy ra các tình huống sau:

- Request chưa rời khỏi client;
- Request đã đến server nhưng chưa thực thi xong;
- Server đã thực thi thành công nhưng response chưa đến client;
- Client đã timeout nhưng server vẫn đang tiếp tục thực thi;
- Server thực thi thất bại nhưng error response cũng không trả về thành công.

Timeout, retry, idempotency, circuit breaker và degradation được chuẩn bị cho những tình huống này. Khi remote call đã đi vào main chain, không thể đợi đến khi production báo lỗi mới bổ sung các thiết kế này.

### Partial failure

Trong single machine system, process bị down thì failure boundary tương đối rõ. Trong hệ thống phân tán thường là nửa tốt, nửa xấu: một phần node bình thường, một phần node bất thường; một phần request thành công, một phần request thất bại; service A không truy cập được service B nhưng service C vẫn truy cập được service B.

Một node không response cũng chưa chắc đã down. Network chập chờn, GC pause, thread pool đầy hoặc disk I/O bị kẹt đều có thể khiến nó tạm thời “trông như đã chết”. Nếu system chỉ dựa vào việc “có response hay không” để phán đoán failure thì rất dễ nhận định sai.

### Data replication và data sharding

Khi data scale và availability tăng, system rất dễ đi đến sharding và replication.

Sharding là phân tán data khác nhau sang các node khác nhau. Các rule phổ biến gồm user ID, order ID, region và hash value. Sau khi shard, pressure trên từng node giảm, nhưng cross-shard query, cross-shard transaction và shard scaling sẽ trở nên phức tạp.

Replication là lưu nhiều bản của cùng một data, chẳng hạn MySQL master-slave replication, Redis master-slave replication, Kafka partition replica và ZooKeeper replica nhiều node. Có replica, khi node failure thì dễ tiếp tục service hơn, read request cũng có thể được chia sẻ cho nhiều replica. Cái giá phải trả là replica synchronization có latency: sau khi master node write thành công, slave node có thể vẫn chưa catch up; user vừa write data xong, nếu read request tiếp theo rơi vào replica cũ thì có thể đọc phải value cũ.

![Sharding, replication và consistency](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/sharding-replication-consistency.webp)

### Không có global clock đồng bộ hoàn hảo

Mỗi machine có physical clock riêng, nhưng clock sẽ có sai lệch và drift. Các cơ chế đồng bộ thời gian như NTP và GPS có thể thu nhỏ sai số, nhưng không thể đảm bảo mọi node có time view hoàn toàn giống nhau ở mọi thời điểm.

Hệ thống phân tán hiếm khi chỉ dựa vào wall clock để phán đoán thứ tự trước sau của event. Khi biểu đạt causal relationship, có thể dùng logical clock như Lamport Clock và Vector Clock; khi replication, election và state change, cũng thường dùng term, epoch, version number hoặc monotonically increasing sequence.

Physical time vẫn hữu ích; log, timeout, lease và cache expiration đều không thể thiếu nó. Chỉ là khi dựa vào physical time, cần biết mình chấp nhận được clock error và drift lớn đến mức nào. Logical clock giải quyết vấn đề thứ tự event, nhưng không thể trực tiếp thay thế nhu cầu về physical time như “sau bao lâu thì lock hết hạn”.

## Những hệ thống phân tán phổ biến là gì?

Hệ thống phân tán không phải một loại middleware nào đó, mà là một dạng system. Các loại phổ biến gồm:

| Loại                            | Vấn đề giải quyết                                                                                  | Ví dụ phổ biến                      |
| ------------------------------- | -------------------------------------------------------------------------------------------------- | ----------------------------------- |
| Distributed coordination system | Leader election, configuration management, service discovery, distributed lock                     | ZooKeeper, etcd, Consul             |
| Distributed database            | Data sharding, replica replication, horizontal scaling                                             | TiDB, CockroachDB, Cassandra, HBase |
| Distributed cache               | Multi-node cache, tăng tốc hot data, mở rộng cache capacity                                        | Redis Cluster, Memcached cluster    |
| Distributed message queue       | Asynchronous decoupling, traffic peak smoothing, event-driven                                      | Kafka, RocketMQ, Pulsar             |
| Distributed file/object storage | Lưu file lớn, nhiều replica, read/write throughput cao                                             | HDFS, Ceph, MinIO                   |
| RPC framework                   | Interface definition, serialization, cross-service request/response                                | gRPC, Apache Thrift                 |
| Service governance system       | Service registration/discovery, load balancing, traffic management, circuit breaker, configuration | Dubbo, Spring Cloud                 |

Những system này giải quyết các vấn đề khác nhau nhưng thường xuất hiện cùng nhau trong một business architecture. Một order system có thể dùng Redis làm cache, dùng RocketMQ truyền order event, dùng ZooKeeper hoặc Nacos làm service discovery, dùng MySQL sharding để lưu order, sau đó dùng distributed tracing system để điều tra request đã đi qua những service nào.

Khi học hệ thống phân tán, đừng chỉ chăm chú học thuộc parameter của một middleware. Hãy hỏi nó giải quyết vấn đề gì khi đặt trong system, đồng thời để lại những complexity nào cho business side.

## Hệ thống phân tán, cluster và microservices khác nhau thế nào?

Ba từ này thường bị trộn lẫn, nhưng chúng không chỉ cùng một việc.

**Cluster** nhấn mạnh deployment form. Nhiều machine cùng cung cấp service có thể gọi là một cluster. Ví dụ 3 Nginx instance, 5 Redis node hoặc 3 application instance. Chúng có thể làm cùng một việc, cũng có thể phân công master-slave, sharding hoặc leader election.

**Hệ thống phân tán** nhấn mạnh sự phối hợp giữa các node. Nhiều node giao tiếp qua network, cùng hoàn thành một task và biểu hiện như một whole ra bên ngoài. Cluster có thể là một dạng của hệ thống phân tán, nhưng hệ thống phân tán còn liên quan đến data replication, consistency, fault tolerance, scheduling và coordination.

**Microservices** là một application architecture style. Nó tách business system thành nhiều service được tổ chức xoay quanh business capability; mỗi service có thể được develop, deploy và scale độc lập. Microservices system thường cũng là hệ thống phân tán vì giữa các service phải đi qua network call. Tuy nhiên, hệ thống phân tán không nhất thiết là microservices. Kafka, HDFS và ZooKeeper bản thân đều là hệ thống phân tán, nhưng không phải business microservices.

Cũng có một trường hợp rất phổ biến: một business application vẫn là monolith nhưng phụ thuộc vào Redis Cluster, Kafka, Elasticsearch và MySQL master-slave. Bản thân business application này chưa được tách thành microservices, nhưng nó chạy trên một nhóm distributed infrastructure.

## Hệ thống phân tán khó ở đâu?

Hệ thống phân tán khó không phải vì các khái niệm nghe có vẻ cao siêu, mà vì có quá nhiều tình huống failure. Network khiến kết quả của một operation trở nên không chắc chắn, còn failure độc lập khiến các node khác nhau đồng thời nhìn thấy state khác nhau của system.

Local call và caller nằm trong cùng một process hoặc failure boundary nên execution result tương đối dễ phán đoán. Remote call thêm một lớp không chắc chắn: timeout chỉ cho biết client không nhận được response trong thời gian quy định, không thể chứng minh server chưa thực thi. Request có thể chưa được gửi đi, cũng có thể đã thực thi thành công nhưng response bị mất. Sự khác biệt này ảnh hưởng trực tiếp đến retry strategy.

![Tính không chắc chắn của remote call](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/remote-call-uncertainty.webp)

Ví dụ order service gọi inventory service để trừ inventory, client đặt timeout là 2 giây. Sau 2 giây order service không nhận được response, nó có một số lựa chọn:

- Trực tiếp cho rằng trừ inventory thất bại và hủy order;
- Gọi lại inventory service một lần nữa;
- Query inventory deduction flow để xác nhận request trước có thành công hay không;
- Đặt order ở trạng thái processing trước, sau đó compensation bằng message hoặc scheduled task.

Mỗi lựa chọn đều có cái giá. Hủy order trực tiếp có thể phán đoán sai vì inventory service có lẽ đã trừ thành công; retry trực tiếp có thể trừ inventory trùng; query flow yêu cầu inventory service cung cấp idempotency key và record có thể query; asynchronous compensation khiến user thấy trạng thái “đang xử lý”, product experience cũng phải phối hợp.

Idempotency trở nên quan trọng trong tình huống này. Chỉ cần tồn tại timeout và retry, cùng một business request có thể được xử lý nhiều lần. Server phải nhận diện được “đây là cùng một business operation”, không thể vì client retry mà trừ tiền, trừ inventory hoặc phát coupon trùng. Với remote write operation có side effect, còn phải thiết kế business idempotency key, result query, retry có giới hạn, exponential backoff và random jitter để tránh khi downstream failure thì retry traffic tiếp tục làm quá tải downstream.

Data consistency cũng là một vấn đề tương tự. Trong monolith application, một database transaction có thể đồng thời update order table và inventory table; sau khi tách thành order service và inventory service, order database và inventory database không nằm trong cùng một transaction. Muốn chúng hoặc cùng success hoặc cùng failure thì cần các giải pháp như distributed transaction, transaction message, TCC, Saga và local message table.

Nhiều business cross-service chấp nhận state không nhất quán trong thời gian ngắn, sau đó dùng transaction message, retry, compensation và reconciliation để state của order, inventory và payment cuối cùng thỏa mãn business constraint. Trong engineering, cách này cũng thường được gọi là “eventual consistency”. Nó không cùng một ngữ cảnh với eventual consistency trong consistency model của replica: vế sau nhấn mạnh rằng khi không còn write mới, nhiều data replica cuối cùng sẽ converge; vế trước thiên về asynchronous coordination của cross-service business flow.

Điều tra vấn đề cũng chậm hơn. Một request đi qua 6 service; chỉ cần log của một service nào đó không chuẩn, thiếu distributed tracing hoặc thiết kế error code lộn xộn thì việc định vị sẽ rất vất vả. Trong production environment, khi thiếu observability, rất khó phán đoán request bị kẹt ở node nào và error xuất hiện đầu tiên từ đâu.

## Các capability nền tảng phổ biến của business distributed system

Những capability dưới đây thường xuất hiện trong microservices và online business system. Chúng là phần hỗ trợ engineering, không phải một phần trong định nghĩa hệ thống phân tán. Quan hệ member của service cũng không nhất thiết phải được duy trì bởi một registry độc lập; DNS, static configuration, Gossip hoặc cluster protocol đều có thể được dùng.

**Service discovery và load balancing**: Service instance sẽ scale out, scale in và restart; caller không thể hard-code service address. Registry ghi nhận service instance, load balancing chọn một instance khả dụng để call.

**Timeout, retry và idempotency**: Remote call bắt buộc phải đặt timeout. Retry cần thận trọng, chỉ phù hợp với operation có thể retry và có idempotency protection. Các operation như payment, trừ inventory và phát coupon nhất định phải có business unique ID hoặc idempotency table làm chỗ dự phòng.

**Circuit breaker, rate limiting và degradation**: Khi downstream service chậm hoặc failure, upstream không thể chờ và retry vô hạn; nếu không failure sẽ lan theo call chain. Circuit breaker dùng để fail fast, rate limiting dùng để kiểm soát pressure ở entry point, degradation dùng để giữ main chain.

**Configuration management và dynamic change**: Sau khi số lượng service tăng, configuration không thể chỉ dựa vào việc sửa local file thủ công. Configuration center có thể quản lý configuration tập trung và hỗ trợ gray release, dynamic refresh và rollback.

**Log, metric và distributed tracing**: Log trả lời “đã xảy ra chuyện gì”, metric trả lời “hiện tại có healthy không”, distributed tracing trả lời “một request đã đi qua đâu”. Chỉ khi đặt 3 loại data này cùng nhau mới có cơ sở để điều tra distributed issue.

**Data consistency và compensation mechanism**: Khi write data giữa các node, phải thiết kế trước cách xử lý sau failure. Đó là strong consistency, eventual consistency hay cho phép inconsistency trong thời gian ngắn? Sau failure dựa vào retry, xử lý thủ công, reconciliation để sửa hay business rollback? Không thể đợi production xảy ra lỗi rồi mới bổ sung các vấn đề này.

## Học hệ thống phân tán thế nào?

Khi học hệ thống phân tán, không nên vừa bắt đầu đã học thuộc tên algorithm. Đi từ engineering problem đến theory trước sẽ thuận lợi hơn.

Bước đầu tiên là network communication. HTTP, RPC, TCP, timeout, retry, connection pool và serialization quyết định các service giao tiếp với nhau thế nào. Không có nền tảng này, việc học service governance về sau sẽ rất thiếu chắc chắn.

Bước thứ hai là service decomposition và service governance. Vì sao phải tách service, sau khi tách thì service discovery, load balancing, rate limiting, circuit breaking và distributed tracing thế nào; xử lý version compatibility và gray release ra sao. Phần lớn vấn đề hằng ngày của microservices nằm ở layer này.

Bước thứ ba là bổ sung data layer: replication, sharding, cache, message queue, distributed ID, distributed lock và distributed transaction. Ở đây cần hỏi nhiều hơn về các tình huống bất thường, chẳng hạn phải làm gì khi message bị gửi trùng, cache và database không nhất quán, hoặc lock hết hạn nhưng business vẫn chưa thực thi xong.

Cuối cùng mới xem các theory và protocol như CAP, BASE, centralized và decentralized, Paxos, Raft, ZAB, Gossip và consistent hashing. Khi học những nội dung này, trọng tâm là hiểu vì sao các system như ZooKeeper, etcd, Kafka, Redis Cluster và distributed database lại được thiết kế như vậy.

Ở đây nên đọc trước [Giải thích chi tiết về distributed coordination](./protocol/centralized-and-decentralized.md). Bài viết đặt Leader, Quorum, split-brain, Lease, Fencing Token và Gossip trên cùng một mạch, giúp bạn hiểu trước khi đi vào chi tiết Raft, ZAB và Gossip: “Ai đưa ra quyết định, state được truyền đi thế nào, nếu sai thì sẽ ra sao”.

Cũng cần phân biệt consensus và distributed transaction. Paxos, Raft và ZAB chủ yếu giải quyết việc một nhóm replica đạt consensus về thứ tự log, Leader hoặc state change như thế nào; TCC, Saga và transaction message chủ yếu giải quyết cách phối hợp commit và compensation giữa nhiều business participant. Chúng có thể xuất hiện trong cùng một system nhưng xử lý các vấn đề khác nhau.

Ở giai đoạn nhập môn, giải thích rõ 6 câu hỏi dưới đây sẽ hữu ích hơn việc học thuộc một chuỗi thuật ngữ:

1. Vì sao single machine system phải tách thành nhiều node?
2. Remote call khác local call ở điểm nào?
3. Vì sao trong hệ thống phân tán không thể đơn giản đồng nhất timeout với failure?
4. Vì sao nhiều replica lại dẫn đến vấn đề consistency?
5. Vì sao khi write data giữa các service thường phải cân nhắc idempotency, compensation và eventual consistency?
6. Vì sao một số system cần Leader, còn một số system phù hợp hơn với việc truyền state bằng Gossip?

## Tài liệu tham khảo

- [Hệ thống phân tán là gì, học hệ thống phân tán thế nào](https://www.cnblogs.com/xybaby/p/7787034.html)
- [Những distributed system architecture mã nguồn mở phổ biến hiện nay là gì?](https://www.zhihu.com/question/19832447/answer/91660607)
- [Sơ đồ thiết kế distributed system phổ biến (tổng hợp)](https://www.raychase.net/6364)
- [A brief introduction to distributed systems](https://www.researchgate.net/publication/306241722_A_brief_introduction_to_Distributed_Systems)
- [MIT 6.824 Distributed Systems](https://pdos.csail.mit.edu/6.824/)
- [Time, Clocks, and the Ordering of Events in a Distributed System](https://www.microsoft.com/en-us/research/publication/time-clocks-ordering-events-distributed-system/)
- [Timeouts, retries and backoff with jitter](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/)
- [Eventually Consistent - Revisited](https://www.allthingsdistributed.com/2008/12/eventually_consistent.html)
- [Raft Consensus Algorithm](https://raft.github.io/)
- 《Designing Data-Intensive Applications》
- 《Distributed Systems: Concepts and Design》

<!-- @include: @article-footer.snippet.md -->
