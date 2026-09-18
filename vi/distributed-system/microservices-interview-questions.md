---
title: "Tổng hợp câu hỏi phỏng vấn microservices mới nhất 2026: phân tách service, giao tiếp, consistency dữ liệu và observability"
description: "Câu hỏi phỏng vấn microservices và lộ trình ôn tập mới nhất 2026, bao quát lựa chọn monolith và microservices, phân tách service, RPC, message queue, service discovery, API gateway, configuration center, database decomposition, distributed transaction, fault tolerance, observability và phương án migration."
category: Distributed
tag:
  - Microservices
  - Distributed Systems
  - Interview Questions
head:
  - - meta
    - name: keywords
      content: microservices interview questions,microservices architecture,service decomposition,service discovery,API gateway,service communication,database decomposition,distributed transaction,Saga,Outbox,circuit breaker,rate limiting,observability,microservices migration
---

Phỏng vấn microservices hiếm khi chỉ dừng ở câu hỏi “microservices là gì”. Interviewer thường bắt đầu từ một lần phân tách architecture rồi hỏi tiếp: Vì sao service được chia như vậy? Xử lý thế nào khi lời gọi liên service thất bại? Sau khi nhiều service tự quản lý dữ liệu, làm thế nào để query và bảo đảm consistency? Còn việc đưa service lên production, scale và khôi phục khi có sự cố thì sao?

Bài viết này là cổng ôn tập nội dung microservices của JavaGuide, tổ chức theo các chủ đề phân tách architecture, service communication, consistency dữ liệu, stability và observability. Bài viết không lặp lại toàn bộ câu trả lời, mà giúp bạn xác định phạm vi ôn tập và kết nối kiến thức rải rác trong các chuyên đề distributed system, high availability, message queue và security.

Nếu thời gian khá gấp, bạn có thể xem trước [Tổng hợp câu hỏi phỏng vấn microservices thường gặp](https://interview.javaguide.cn/distributed-system/microservices-interview-questions.html) dạng ôn tập cấp tốc, đánh dấu những câu chưa biết rồi quay lại các chuyên đề tương ứng trong bài này để bổ sung chi tiết.

## Khi ôn tập, trước hết cần nắm những vấn đề nào?

| Module                          | Nội dung cần trình bày rõ                                                                         | Hướng hỏi thêm thường gặp                                                                          |
| ------------------------------- | ------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Architecture và phân tách       | Vì sao phân tách, phân tách theo gì, phân tách đến granularity nào                                | Monolith, cluster, distributed system, bounded context, service boundary, distributed monolith     |
| Communication và infrastructure | Các service gọi và tìm thấy nhau thế nào, request bên ngoài đi vào system ra sao                  | REST, RPC, message queue, service registry và discovery, API gateway, configuration center         |
| Data và consistency             | Sau khi mỗi service tự quản lý dữ liệu, xử lý cross-service query và business transaction thế nào | Database per Service, aggregate query, Saga, transactional message, Outbox, idempotent             |
| Stability và delivery           | Kiểm soát local failure ra sao, release và scale service an toàn thế nào                          | Timeout, retry, circuit breaker, rate limiting, isolation, probe, graceful shutdown, compatibility |
| Observability và security       | Khi có vấn đề, định vị call chain thế nào; truyền identity và permission qua các service ra sao   | Log, metric, tracing, alert, gateway authentication, server-side authorization                     |

Khi trả lời câu hỏi về microservices, tên component chỉ là một phần của phương án. Bạn còn phải nêu căn cứ phân tách, quyền sở hữu dữ liệu, cách xử lý khi thất bại, cũng như việc team có đủ năng lực độc lập phát triển, deploy và vận hành các service này hay không.

## Nền tảng microservices và phân tách service

Microservices là một kiểu architecture tổ chức service theo business capability. Một system có nhiều process không đồng nghĩa với service boundary hợp lý; nếu nhiều service phải cùng sửa, cùng release và còn trực tiếp dùng chung database table, kết quả thường là một distributed monolith có chi phí vận hành cao hơn.

![Từ monolith đến distributed e-commerce](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/monolith-to-distributed-ecommerce.webp)

Nội dung liên quan:

- [Nhập môn distributed system](./distributed-system-intro.md)
- [Tổng hợp câu hỏi phỏng vấn distributed system](./distributed-system-interview-questions.md)

Câu hỏi phỏng vấn thường gặp:

- Monolith architecture, cluster, distributed system và microservices khác nhau thế nào?
- Microservices mang lại những lợi ích gì? Network call, consistency dữ liệu và chi phí vận hành làm tăng bao nhiêu độ phức tạp?
- Những team và giai đoạn business nào không phù hợp để áp dụng microservices ngay?
- Nên phân tách service theo business capability, domain boundary hay database table?
- Bounded context có quan hệ thế nào với service boundary?
- Phân tách service quá nhỏ sẽ phát sinh những vấn đề gì?
- Distributed monolith là gì? Làm thế nào để xác định system đã rơi vào trạng thái này?

Phân tách service cần đồng thời xét đến thay đổi business, quyền sở hữu dữ liệu và trách nhiệm của team. Tách mỗi database table thành một service thường khiến một business operation phải đi qua rất nhiều network call; chỉ phân tách theo cơ cấu tổ chức lại dễ để lại service boundary thiếu tự nhiên sau khi team thay đổi.

## Service communication và infrastructure

Sau khi tách service, lời gọi trong process trước đây sẽ trở thành network communication. Synchronous call cần xử lý timeout và kết quả không chắc chắn; asynchronous message cần xử lý duplicate, thứ tự và eventual consistency. Service discovery, gateway và configuration center lần lượt hỗ trợ việc định địa chỉ, làm điểm vào cho traffic và thay đổi configuration khi số lượng service tăng lên.

![Quy trình RPC call và các capability cốt lõi](https://oss.javaguide.cn/github/javaguide/distributed-system/rpc/rpc-overview.png)

![Trách nhiệm và vị trí deploy của API gateway](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/api-gateway-overview.png)

Nội dung liên quan:

- [Tổng hợp kiến thức nền tảng RPC](./rpc/rpc-intro.md)
- [HTTP và RPC khác nhau thế nào?](../cs-basics/network/http-vs-rpc.md)
- [Tổng hợp câu hỏi phỏng vấn message queue](../high-performance/message-queue/message-queue-interview-questions.md)
- [Tổng hợp kiến thức nền tảng API gateway](./api-gateway.md)
- [Tổng hợp câu hỏi thường gặp về Spring Cloud Gateway](./spring-cloud-gateway-questions.md)
- [Tổng hợp câu hỏi phỏng vấn distributed configuration center](./distributed-configuration-center.md)

Câu hỏi phỏng vấn thường gặp:

- Giữa các microservices nên chọn REST, RPC hay message queue?
- Synchronous call chain quá dài sẽ khuếch đại latency và failure như thế nào?
- Vì sao một RPC framework cần service discovery, load balancing, serialization và cơ chế timeout?
- Client-side service discovery và server-side service discovery khác nhau thế nào?
- API gateway nên phụ trách những phần nào trong routing, authentication, rate limiting và protocol conversion?
- Gateway, Nginx, load balancer và BFF lần lượt giải quyết vấn đề gì?
- Vì sao configuration center không thể chỉ là một thư mục configuration file từ xa?
- Configuration change được gray release thế nào? Khi client không lấy được configuration thì khởi động và chạy ra sao?

Synchronous và asynchronous communication có thể cùng tồn tại. User query thường cần trả về đồng bộ; notification, point và data synchronization sau khi tạo order có thể dùng message queue. Việc lựa chọn nên xuất phát từ yêu cầu về business timeliness, failure handling và consistency, không nên chỉ quyết định theo framework mà team quen thuộc.

## Data decomposition và consistency

Việc các service phát triển độc lập thường đòi hỏi quyền sở hữu dữ liệu cũng phải rõ ràng. Nhiều service cùng trực tiếp đọc và ghi một table tuy giảm được interface call, nhưng bất kỳ bên nào thay đổi schema hoặc semantics của dữ liệu cũng có thể ảnh hưởng service khác; service cũng khó thực sự release độc lập.

![Distributed transaction hình thành từ order service và inventory service](https://oss.javaguide.cn/github/javaguide/distributed-system/distributed-transaction/distributed-transaction-with-two-services.png)

Nội dung liên quan:

- [Tổng hợp giải pháp distributed transaction](./distributed-transaction.md)
- [Tổng hợp câu hỏi phỏng vấn message queue](../high-performance/message-queue/message-queue-interview-questions.md)
- [Tổng hợp giải pháp idempotent cho interface](../high-availability/idempotency.md)

Câu hỏi phỏng vấn thường gặp:

- Vì sao nhấn mạnh mỗi microservice phải sở hữu dữ liệu do mình phụ trách?
- Nhiều service dùng chung database table sẽ tạo ra những coupling nào?
- Cross-service query nên dùng API aggregation, read model, search engine hay data synchronization?
- Business cross-service nên lựa chọn giữa strong consistency và eventual consistency thế nào?
- 2PC, TCC, Saga, transactional message và local message table lần lượt phù hợp với trường hợp nào?
- Transactional Outbox xử lý thế nào tình huống “database ghi thành công nhưng gửi message thất bại”?
- Khi compensation operation thất bại hoặc chạy lặp, làm thế nào để bảo đảm business state đúng?
- Vì sao consumer vẫn cần idempotent?

Giải pháp distributed transaction phải được thảo luận cùng business action. Việc reserve inventory có thể rollback, SMS đã gửi thì không thể thu hồi, còn external payment chưa chắc hỗ trợ tham gia local transaction. Compensation không đồng nghĩa với khôi phục về old value trên phương diện kỹ thuật; nó phải phù hợp với state tiếp theo được business cho phép.

## Stability, release và scale

Microservices phân tán một request qua nhiều node, nên tần suất local failure sẽ tăng theo số bước gọi. Timeout giới hạn thời gian chờ, retry xử lý lỗi tạm thời, circuit breaker ngăn tiếp tục gọi downstream đang liên tục bất thường, còn rate limiting và isolation bảo vệ resource hữu hạn; các cơ chế này cần được cấu hình trên cùng một call chain.

![State machine của circuit breaker](https://oss.javaguide.cn/github/javaguide/high-availability/fallback-and-circuit-breaker-fuse-state-machine.png)

Nội dung liên quan:

- [Tổng hợp câu hỏi phỏng vấn về system design high availability](../high-availability/high-availability-system-interview-questions.md)
- [Giải thích chi tiết về timeout và retry](../high-availability/timeout-and-retry.md)
- [Giải thích chi tiết về rate limiting service](../high-availability/limit-request.md)
- [Giải thích chi tiết về fallback và circuit breaker](../high-availability/fallback-and-circuit-breaker.md)
- [Giải thích chi tiết về redundancy design](../high-availability/redundancy.md)

Câu hỏi phỏng vấn thường gặp:

- Timeout, retry, circuit breaker, rate limiting, fallback và isolation lần lượt giải quyết vấn đề gì?
- Nên thiết lập timeout thế nào dựa trên upstream Deadline và latency distribution của downstream?
- Vì sao nhiều layer service cùng retry có thể tạo ra retry storm?
- Liveness, Readiness và Startup probe lần lượt nên kiểm tra gì?
- Làm thế nào để service graceful shutdown, tránh ngắt trực tiếp request đang xử lý?
- Làm thế nào để API và message format duy trì backward compatibility?
- Khi gray release xảy ra bất thường, traffic và data rollback thế nào?
- Trước khi scale consumer hoặc service instance, vì sao cần kiểm tra partition, connection pool và capacity của downstream?

Health check không nên nhồi mọi dependency vào một probe. Khi database chậm tạm thời, nếu mọi instance cùng restart vì Liveness fail, dependency failure ban đầu sẽ tiếp tục lan rộng. Probe, traffic removal, connection draining và thứ tự application shutdown cần được design cùng nhau.

## Observability và security

Một request trong monolith application thường chỉ đi qua một process, còn microservices call có thể đi qua gateway, nhiều business service, cache, database và message queue. Log, metric và distributed tracing cần dùng request identifier thống nhất để liên kết; nếu không, bạn chỉ thấy từng service báo lỗi riêng lẻ mà không thể khôi phục toàn bộ process.

Nội dung liên quan:

- [Hệ thống kiến thức high availability](../high-availability/)
- [Chuyên đề authentication, authorization và data security](../system-design/security/)
- [Tổng hợp kiến thức nền tảng API gateway](./api-gateway.md)

Câu hỏi phỏng vấn thường gặp:

- Log, metric và distributed tracing lần lượt phù hợp để định vị vấn đề nào?
- Microservices nên monitor những metric nào về request volume, error rate, latency, resource và business?
- TraceId được truyền tiếp qua HTTP, RPC và message queue thế nào?
- Authentication nên đặt ở gateway hay business service?
- Sau khi gateway hoàn tất identity verification, vì sao business service vẫn phải thực hiện resource-level authorization?
- Các service truyền identity cho nhau thế nào và ngăn việc bypass gateway để truy cập trực tiếp internal interface ra sao?

Gateway phù hợp để thực hiện Token validation thống nhất và access control ở mức thô; order ownership, tenant scope và data permission vẫn phải do service sở hữu business data đánh giá. Chỉ ẩn button ở frontend hoặc chỉ kiểm tra role tại gateway không thể bao phủ việc vượt quyền ở mức object và direct call giữa các service.

## Migration microservices và cách trả lời về phương án

Migration từ monolith sang microservices thường bắt đầu với module thay đổi thường xuyên, chịu áp lực capacity rõ rệt hoặc có ranh giới team tương đối rõ. Trong thời gian migration, system mới và cũ sẽ cùng tồn tại; data synchronization, traffic switching, interface compatibility và rollback cần được design trước, quan trọng hơn việc “tách ra bao nhiêu service”.

Câu hỏi phỏng vấn thường gặp:

- Làm thế nào để nhận diện business module phù hợp nhất để ưu tiên tách ra?
- Trong thời gian service mới và cũ cùng tồn tại, nên migration traffic và data thế nào?
- Làm thế nào để tránh việc rewrite một lần khiến delivery cycle và risk mất kiểm soát?
- Khi design một phương án microservices, cần trình bày những nội dung nào?

Khi trả lời một câu hỏi microservices design hoàn chỉnh, có thể triển khai theo thứ tự: business goal, service boundary, interface và message, quyền sở hữu dữ liệu, consistency, stability, observability, security, release và migration. Khi đề bài không cung cấp traffic, quy mô team và yêu cầu consistency, trước tiên nên xác nhận các constraint này với interviewer, rồi mới quyết định có phân tách hay không và dùng component nào.

## Ôn tập theo thời gian chuẩn bị

| Thời gian còn lại | Đề xuất                                                                                                                                                                                                                                                 | Mục tiêu ôn tập                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| 1～2 ngày         | Đọc một lượt [Tổng hợp câu hỏi phỏng vấn microservices thường gặp](https://interview.javaguide.cn/distributed-system/microservices-interview-questions.html), ưu tiên bổ sung service decomposition, communication, data consistency và fault tolerance | Trình bày rõ lợi ích, chi phí của microservices và một call chain hoàn chỉnh                |
| 3～7 ngày         | Bổ sung RPC, gateway, configuration center, distributed transaction, message queue, timeout và circuit breaker; đồng thời vẽ các service và thay đổi dữ liệu đi qua một request order hoặc payment                                                      | Trả lời được failure path, trade-off của phương án và các câu hỏi infrastructure thường gặp |
| Trên 1 tuần       | Dựa trên project để tổng hợp một lần service decomposition hoặc architecture design, bổ sung interface compatibility, gray release, monitoring và alerting, capacity và rollback plan                                                                   | Trình bày được từ business constraint đến service, data, delivery và operation              |

<!-- @include: @article-footer.snippet.md -->
