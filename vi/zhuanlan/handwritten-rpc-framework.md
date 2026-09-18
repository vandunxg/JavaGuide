---
title: "Hướng dẫn tự viết framework RPC: Thực chiến Netty + ZooKeeper + SPI"
description: "Tự triển khai từ đầu một framework Java RPC dựa trên Netty, ZooKeeper, dynamic proxy và SPI tùy chỉnh, bao quát communication protocol, serialization, service registration and discovery, load balancing, asynchronous calls, timeout handling, automated testing, cách viết trong CV và chuẩn bị phỏng vấn."
category: Knowledge Planet
star: 5
head:
  - - meta
    - name: keywords
      content: handwritten RPC,RPC framework,RPC thực chiến,Netty RPC,ZooKeeper,Java RPC,custom protocol,RPC serialization,service registration and discovery,consistent hashing,SPI mechanism,CompletableFuture,RPC interview project
---

Khi người phỏng vấn tiếp tục hỏi sâu về một project RPC, câu hỏi sẽ nhanh chóng đi vào phần triển khai cụ thể:

- TCP chỉ là byte stream, xử lý sticky packet và half packet như thế nào?
- Khi một long connection gửi nhiều request, sau khi response quay về làm thế nào để tìm đúng caller tương ứng?
- Sau khi service instance online hoặc offline, client làm thế nào để kịp thời nhận được địa chỉ mới?
- Sau khi request timeout, phải dọn dẹp `CompletableFuture` và Map chứa request đang thực hiện như thế nào, có cần đóng Channel được dùng lại không?
- Khi thêm một cách serialization mới, tại sao không nên tiếp tục thêm branch vào `switch`?

Nếu chỉ nhớ rằng “RPC khiến remote call đơn giản như local call”, bạn sẽ khó trả lời tiếp những câu hỏi này. 《Framework RPC tự viết》 xoay quanh một project Java có thể chạy được, kết nối dynamic proxy, network communication, custom protocol, serialization, registry, load balancing, asynchronous call và exception handling thành một thể thống nhất.

Nếu bạn chưa quen với khái niệm và flow của RPC, có thể đọc trước bài viết miễn phí của JavaGuide: [Giải thích chi tiết về RPC remote procedure call](../distributed-system/rpc/rpc-intro.md).

## Giới thiệu project

[guide-rpc-framework](https://github.com/Snailclimb/guide-rpc-framework) là một framework RPC lightweight hướng đến việc học và thực hành engineering. Project giữ lại phiên bản Socket thời kỳ đầu, đồng thời cung cấp implementation dựa trên Netty, thuận tiện cho việc đối chiếu khác biệt giữa BIO và NIO trong connection, thread và cách xử lý request.

**Địa chỉ project (hoan nghênh Star):**

- GitHub：<https://github.com/Snailclimb/guide-rpc-framework>
- Gitee：<https://gitee.com/SnailClimb/guide-rpc-framework>

Mã nguồn project hoàn toàn open source. Tutorial đi kèm là một booklet nội bộ của JavaGuide Knowledge Planet, hiện gồm 15 bài chính, cùng một bài về cách viết trong CV và giải đáp các câu hỏi thường gặp khi phỏng vấn.

## Một RPC call trải qua những gì

![Kiến trúc tổng thể của Guide RPC Framework](https://oss.javaguide.cn/github/javaguide/distributed-system/rpc/guide-rpc-framework-architecture.webp)

Lấy service interface được inject bằng `@RpcReference` làm ví dụ, một call sẽ trải qua các bước sau:

```text
Local interface call
    ↓
JDK dynamic proxy lắp ráp RpcRequest
    ↓
ZooKeeper service discovery
    ↓
Consistent hashing chọn service instance
    ↓
Thiết lập hoặc reuse Netty Channel
    ↓
Protocol encoding, serialization và compression
    ↓
Server frame splitting, decoding và thực thi target method
    ↓
Response hoàn tất CompletableFuture tương ứng theo requestId
```

Tutorial sẽ không dừng ở mức “client gửi request, server trả về result”. Cách sắp xếp các protocol field, vì sao server không thể thực thi method tốn thời gian trong I/O thread, request dùng chung một timeout budget từ lúc xếp hàng đến khi nhận response như thế nào, tất cả sẽ được giải thích dựa trên code hiện tại.

## Project hiện đã có những năng lực nào

| Module                             | Implementation hiện tại                                                                                                                  |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Network communication              | Hai implementation truyền tải Socket và Netty, Netty hỗ trợ reuse Channel và heartbeat detection                                         |
| Communication protocol             | Header cố định 16 byte, gồm magic number, version, full length, message type, codec number và request number                             |
| Message boundary                   | Frame splitting dựa trên length field, đồng thời giới hạn frame và message body sau khi decompress tối đa 8 MiB                          |
| Serialization và compression       | Hỗ trợ Hessian, Kryo, Protostuff và Gzip, cách serialization mặc định là Hessian                                                         |
| Service registration and discovery | Sử dụng ZooKeeper ephemeral node và CuratorCache, hỗ trợ local address cache và session recovery                                         |
| Load balancing                     | Mặc định sử dụng consistent hashing, mỗi physical node tương ứng với 160 virtual node                                                    |
| Request-response correlation       | Sử dụng `requestId -> CompletableFuture` để quản lý concurrent request, xử lý timeout, cancel, disconnect và cleanup khi close           |
| Invocation method                  | Cung cấp synchronous proxy và asynchronous mirror interface trả về `CompletableFuture`                                                   |
| Extension mechanism                | Named SPI tùy chỉnh, bao phủ serialization, compression, load balancing, registry, service discovery và transport layer                  |
| Spring integration                 | Sử dụng `@RpcScan`, `@RpcService`, `@RpcReference` để hoàn tất scan, publish service và inject proxy                                     |
| Error handling                     | Sử dụng stable status code từ 0 đến 16 để phân biệt parameter error, timeout, resource exhaustion, service unavailable và internal error |
| Automated testing                  | Tổng cộng 82 test, bao phủ configuration, SPI, encoding/decoding, proxy, transport, service handler và ZooKeeper session recovery        |

Đây vẫn là một project phục vụ việc học và thực hành engineering. Automatic retry, circuit breaker, rate limiting, TLS, caller authentication, remote cancellation, monitoring center và bounded task queue phía server vẫn chưa được implement. Tutorial sẽ chỉ rõ trực tiếp các giới hạn này, tránh gán năng lực của một RPC framework trưởng thành vào code hiện tại.

## Tutorial này được trình bày như thế nào

Tutorial trước tiên dùng Socket và JDK dynamic proxy để dựng call chain tối thiểu, sau đó từng bước chuyển sang Netty, custom protocol, pluggable serialization và ZooKeeper. Người đọc có thể thấy mỗi lần refactor giải quyết vấn đề nào, thay vì phải đối mặt ngay với một bộ code đã được lắp ráp sẵn.

Phần phân tích source code được triển khai theo thứ tự call thực tế. Client tạo request như thế nào, service discovery chọn address như thế nào, codec xử lý message như thế nào, server thực thi method như thế nào, response hoàn tất Future ra sao, tất cả đều có thể đối chiếu giữa bài viết và code.

Các chương SPI, load balancing và testing phía sau tiếp tục xử lý extension và failure scenario. Ngoài normal call, tutorial cũng thảo luận về duplicate `requestId`, response đến muộn, request timeout, connection đóng sớm, protocol frame không hợp lệ và ZooKeeper session mất hiệu lực.

Environment, thứ tự khởi động và RPC call đầu tiên cần thiết để chạy project tiếp tục được duy trì trong phần “Quick Start” của project README, không tách riêng thêm một bài trùng lặp.

## Nội dung tutorial đi kèm

### RPC cơ bản và thiết kế framework

1. **01 RPC là gì? Nguyên lý là gì?**: Bắt đầu từ khác biệt giữa local call và remote call, giải thích các thành phần của RPC và quy trình cơ bản của một call.
2. **02 Giới thiệu các RPC framework thường gặp**: So sánh các giải pháp thường gặp như Dubbo, gRPC, Thrift, Motan, tìm hiểu các yếu tố cần quan tâm khi lựa chọn framework.
3. **03 Làm thế nào để tự triển khai một framework RPC?**: Xác định trách nhiệm của các module như proxy, transport, serialization, registry và load balancing.
4. **04 Giới thiệu serialization và lựa chọn serialization protocol**: So sánh JDK serialization, Hessian, Kryo, Protostuff và các giải pháp khác, đồng thời giải thích cách project hiện tại tích hợp chúng.

### Network communication, dynamic proxy và ZooKeeper

1. **05 Thực chiến Socket network communication**: Dùng số component ít nhất để chạy thông remote request, tìm hiểu thread và connection model của phiên bản blocking I/O.
2. **06 Thực chiến Netty network communication**: Tìm hiểu EventLoop, Channel, Pipeline, Handler và frame splitting bằng length field.
3. **07 Thực chiến static proxy, JDK và CGLIB dynamic proxy**: Tìm hiểu proxy object che giấu request assembly và chi tiết network transport như thế nào.
4. **08 Các command ZooKeeper thường dùng và giải thích chi tiết cách sử dụng Curator**: Nắm vững node model, ephemeral node, cơ chế listener và cách sử dụng cơ bản của Curator.

### Phân tích source code của RPC framework

1. **09 Transport module**: Đọc code theo luồng client gửi request, protocol encoding/decoding, Channel reuse, heartbeat và server handler.
2. **10 Registry module**: Phân tích service identifier, service publishing, address cache, node listener và session recovery.
3. **11 Các module khác**: Tiếp tục giải thích proxy, service provider, local service container, Spring annotation integration và exception handling.

### Asynchronous call, extension mechanism và reliability

1. **12 Dùng CompletableFuture để tối ưu việc nhận response từ server**: Implement concurrent request và response correlation, đồng thời giải thích timeout, cancel và resource cleanup.
2. **13 Extension mechanism SPI của RPC framework được implement như thế nào?**: Chuyển từ hard-coded factory sang named SPI, phân tích extension discovery, validation và three-level cache.
3. **14 Thực chiến random load balancing và consistent hashing**: Tìm hiểu trường hợp sử dụng phù hợp của hai algorithm, tập trung phân tích implementation consistent hashing hiện được project sử dụng.
4. **15 Làm thế nào để test một RPC framework?**: Dùng JUnit, Netty EmbeddedChannel và Curator TestingServer để kiểm tra concurrency, timeout, disconnect, invalid frame và session recovery.

## Cách viết trong CV và chuẩn bị phỏng vấn

Project chạy được chỉ là bước đầu. Sau khi ghi vào CV, serialization mặc định, độ dài protocol header, thread model, phạm vi timeout và các năng lực governance còn thiếu hiện tại đều có thể bị hỏi sâu.

《Cách viết trong CV và giải đáp các câu hỏi thường gặp khi phỏng vấn》 đi kèm bao gồm:

- 9 cách viết được đề xuất cho project backend, cùng 6 phiên bản rút gọn phù hợp với CV một trang.
- Các chỉ số hiện có thể kiểm tra trực tiếp, chẳng hạn protocol header 16 byte, giới hạn message 8 MiB, 160 virtual node và 82 test.
- Phần giới thiệu project trong một phút và 18 câu hỏi thường gặp.
- Những mô tả năng lực project dễ viết sai, chẳng hạn mặc định là Hessian, random load balancing chưa được tích hợp vào SPI configuration hiện tại, không có Spring Boot Starter, cũng chưa có retry và circuit breaker hoàn chỉnh.

Phần này không dạy bạn tự bịa số liệu để làm đẹp project. Không có báo cáo stress test thì không viết “100.000 QPS”; chưa implement Spring Boot Starter thì cũng không đổi tên Spring annotation integration rồi đưa vào CV.

## Học xong cần trả lời được những câu hỏi nào

- RPC khác HTTP call như thế nào, tại sao project này chọn custom binary protocol?
- 16 byte của protocol header lần lượt lưu thông tin gì, length field giải quyết sticky packet và half packet như thế nào?
- Tại sao phải đồng thời giới hạn frame size trước khi compress và message body size sau khi decompress?
- Khi nhiều concurrent request reuse một Channel, response tìm đúng `CompletableFuture` tương ứng như thế nào?
- Request timeout bắt đầu được tính từ thời điểm nào, tại sao phải bao phủ cả queueing và service discovery?
- Custom SPI khác JDK `ServiceLoader` và Dubbo SPI như thế nào?
- ZooKeeper ephemeral node, CuratorCache và session recovery lần lượt giải quyết vấn đề gì?
- Tại sao consistent hashing cần virtual node, khi service instance thay đổi thì những request nào sẽ được remap?
- Tại sao server phải đưa việc thực thi business ra khỏi Netty I/O thread, các method trên cùng một Channel có thể chạy song song không?
- Framework này còn thiếu những năng lực nào để có thể dùng trong production, nên bổ sung mục nào trước?

Nếu những câu hỏi này chỉ có thể học thuộc kết luận, bạn nên quay lại các chapter tương ứng và đối chiếu code để đi lại một lượt.

## Phù hợp với ai

- Đã học Java basics, collection, reflection và multi-threading, muốn đưa những kiến thức này vào sử dụng trong một project hoàn chỉnh.
- Đã tiếp xúc với Netty hoặc ZooKeeper nhưng vẫn khá mơ hồ về trách nhiệm của chúng trong RPC call chain.
- Đang chuẩn bị ứng tuyển Java backend cho sinh viên mới tốt nghiệp hoặc người đã có kinh nghiệm, muốn bổ sung một project có thể trình bày sâu về protocol, network và distributed coordination.
- Thường sử dụng các công cụ remote call như Dubbo, Feign, muốn tiếp tục tìm hiểu proxy, protocol, service registration and discovery và response correlation được implement như thế nào.
- Chuẩn bị đọc source code của các framework như Dubbo, Netty, muốn trước tiên xây dựng nhận thức tổng thể bằng một project có quy mô nhỏ hơn.

Lần đầu tiếp xúc với RPC cũng không sao, nhưng tốt nhất bạn đã nắm Java basic syntax, Maven và các khái niệm cơ bản về client/server. Tutorial sẽ không dành quá nhiều dung lượng để lặp lại những kiến thức prerequisite này.

## Thứ tự học đề xuất

1. Theo project README chuẩn bị JDK 25, Maven 3.9 và ZooKeeper 3.9.5, trước tiên chạy thông server và client.
2. Đọc 8 tutorial đầu tiên, tự hoàn thành implementation tối thiểu của Socket, proxy, Netty và ZooKeeper.
3. Đối chiếu source code hiện tại khi đọc tutorial 9–14, ít nhất tự tay cải tạo một extension point, chẳng hạn thêm serializer hoặc load balancing strategy.
4. Chạy `mvn verify`, sau đó bổ sung một test về timeout, disconnect hoặc invalid frame, xác nhận rằng bạn có thể giải thích resource cleanup sau failure.
5. Cuối cùng chỉnh lý CV và chuẩn bị câu hỏi đào sâu. Năng lực chưa tự tay verify thì không ghi vào phần kinh nghiệm project.

## Câu hỏi thường gặp

### Code project có cần trả phí không?

Không cần. Code trong repository GitHub và Gitee hoàn toàn open source miễn phí, không có Pro version.

### Đọc tutorial đi kèm ở đâu?

Tutorial đi kèm là booklet nội bộ của JavaGuide Knowledge Planet, được đọc online qua tài liệu Yuque và không bán riêng cho bên ngoài.

### Tutorial là video hay văn bản?

Tutorial chủ yếu gồm văn bản, phân tích source code và hình minh họa kỹ thuật. Khi đọc, bạn có thể chuyển trực tiếp giữa bài viết và code, đồng thời thuận tiện tìm kiếm một protocol field, class hoặc test scenario nào đó về sau.

### Có thể đưa project trực tiếp vào CV không?

Có, nhưng trước hết cần chạy thông project, đọc hiểu các call chain chính và hoàn thành ít nhất một thay đổi của riêng bạn. Người phỏng vấn thường tiếp tục hỏi về protocol design, Netty thread model, ZooKeeper node, consistent hashing hoặc `CompletableFuture`; chỉ copy mô tả project sẽ rất khó ứng phó.

## Tham gia học

Nếu bạn chỉ muốn đọc source code, hãy truy cập trực tiếp repository open source. Nếu muốn học quy trình implementation theo call chain, đồng thời chuẩn bị cách viết trong CV và các câu hỏi đào sâu về project, bạn có thể tham gia [JavaGuide Knowledge Planet](../about-the-author/zhishixingqiu-two-years.md) để đọc toàn bộ tutorial.

Trong Knowledge Planet còn cung cấp hỏi đáp 1-1, chỉnh sửa CV, tài liệu phỏng vấn Java, system design và scenario question, đọc source code cùng các nội dung khác; bạn có thể sử dụng chúng cùng project RPC này.

<!-- @include: @planet2.snippet.md -->
