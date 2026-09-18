---
title: "Giải thích chi tiết về RPC (Remote Procedure Call): nguyên lý, quy trình gọi, serialization protocol và lựa chọn framework"
category: Phân tán
description: "Giải thích cơ bản về RPC (Remote Procedure Call), trình bày nguyên lý cốt lõi, quy trình gọi, dynamic proxy, serialization protocol, truyền tải mạng, service discovery và lựa chọn các framework thường gặp như Dubbo, gRPC, Thrift."
tag:
  - RPC
head:
  - - meta
    - name: keywords
      content: RPC,nguyên lý RPC,Remote Procedure Call,dynamic proxy,serialization,service discovery,Dubbo,gRPC,Thrift,giao tiếp microservice,câu hỏi phỏng vấn RPC
---

Bài viết này giới thiệu ngắn gọn một số khái niệm cơ bản liên quan đến RPC.

Xét trong distributed system, RPC giải quyết vấn đề **các service gọi lẫn nhau như thế nào**. Request từ bên ngoài thường đi qua [API gateway](../api-gateway.md) trước khi vào hệ thống. Sau đó, các service bên trong tiếp tục phối hợp với nhau qua RPC, HTTP Client, message queue và các phương thức khác. Nếu muốn tìm hiểu chi tiết về service governance của RPC framework trưởng thành như Dubbo, bạn có thể đọc tiếp [Tổng hợp câu hỏi phỏng vấn Dubbo](./dubbo.md).

## RPC là gì?

![Tổng quan về RPC](https://oss.javaguide.cn/github/javaguide/distributed-system/rpc/rpc-overview.png)

**RPC (Remote Procedure Call)** nghĩa là gọi thủ tục từ xa. Qua tên gọi có thể thấy RPC tập trung vào remote call thay vì local call.

**Tại sao cần RPC?** Vì các method do service trên hai server khác nhau cung cấp không nằm trong cùng một memory space, nên cần network programming để truyền các parameter cần cho method call. Kết quả method call cũng cần được nhận qua network programming. Tuy nhiên, nếu tự viết network programming để thực hiện quy trình này thì khối lượng công việc rất lớn, vì phải xử lý cả phương thức truyền tải tầng dưới (TCP hay UDP), serialization và nhiều vấn đề khác.

**RPC giúp chúng ta làm gì?** Nói đơn giản, RPC giúp gọi method của một service trên remote computer, với cách sử dụng đơn giản như gọi local method. Hơn nữa, bạn không cần biết các chi tiết cụ thể của network programming ở tầng dưới.

Ví dụ: hai service A và B được deploy trên hai machine khác nhau. Nếu service A muốn gọi một method của service B, nó có thể thực hiện qua RPC.

Tóm lại: **RPC xuất hiện để giúp việc gọi remote method đơn giản như gọi local method.**

## Nguyên lý của RPC là gì?

Để dễ hiểu nguyên lý RPC, có thể xem các chức năng cốt lõi của RPC được triển khai qua 5 phần sau:

1. **Client (service consumer)**: phía gọi remote method.
1. **Client Stub**: đây thực chất là một proxy class. Proxy class chủ yếu truyền thông tin về method, class, method parameter mà bạn gọi đến server.
1. **Network transport**: truyền thông tin của method được gọi, chẳng hạn parameter, đến server; sau khi server thực thi xong, truyền kết quả trả về qua network. Có nhiều cách triển khai network transport, chẳng hạn Socket cơ bản hoặc Netty có performance và khả năng đóng gói tốt hơn (khuyến nghị).
1. **Server Stub**: Stub này không phải proxy class. Cách gọi “server Stub” có thể gây hiểu nhầm; về bản chất, đây là class nhận request thực thi method từ client, gọi method tương ứng rồi trả kết quả về client.
1. **Server (service provider)**: phía cung cấp remote method.

Sơ đồ nguyên lý cụ thể như sau. Phần sau sẽ nối các bước lại để trình bày toàn bộ quy trình RPC.

![Sơ đồ nguyên lý RPC](https://oss.javaguide.cn/github/javaguide/distributed-system/rpc/rpc-principle.png)

1. Service consumer (client) gọi remote service theo cách gọi local.
1. Client Stub nhận lời gọi, sau đó lắp ghép method, parameter và các thông tin khác thành message body có thể truyền qua network (serialization): `RpcRequest`.
1. Client Stub tìm địa chỉ remote service rồi gửi message đến service provider.
1. Server Stub nhận message và deserialize thành Java object: `RpcRequest`.
1. Server Stub gọi local method dựa trên class, method, method parameter và các thông tin khác trong `RpcRequest`.
1. Server Stub nhận kết quả thực thi method, lắp ghép thành message body có thể truyền qua network: `RpcResponse` (serialization), rồi gửi đến consumer.
1. Client Stub nhận message và deserialize thành Java object: `RpcResponse`, từ đó nhận được kết quả cuối cùng. Hết!

Sau phần trình bày trên, hy vọng bạn đã hiểu nguyên lý RPC.

Dù nội dung không dài, phần cốt lõi của RPC framework về cơ bản đã được giải thích rõ. Các chi tiết kỹ thuật bên trên sẽ được giới thiệu trong những chương sau.

**Cuối cùng, về nguyên lý RPC, bạn không chỉ cần hiểu mà còn phải tự vẽ và trình bày lại cho người khác. Vì trong phỏng vấn, câu hỏi này gần như luôn xuất hiện khi interviewer hỏi về RPC.**

## Có những RPC framework thường gặp nào?

Ở đây, RPC framework là framework cho phép client gọi trực tiếp method của server, đơn giản như gọi local method, chẳng hạn Dubbo, Motan và gRPC được giới thiệu dưới đây. Nếu cần làm việc với HTTP protocol, parse và đóng gói HTTP request/response, thì các framework như vậy không được xem là “RPC framework”, ví dụ Feign.

### Dubbo

![](https://oss.javaguide.cn/github/javaguide/distributed-system/rpc/image-20220716111053081.png)

Apache Dubbo là một microservice framework, cung cấp các giải pháp như RPC communication hiệu năng cao, traffic governance và observability cho việc triển khai microservice quy mô lớn,
bao phủ SDK implementation cho nhiều ngôn ngữ như Java và Golang.

Dubbo cung cấp gần như toàn bộ năng lực service governance, từ service definition, service discovery, service communication đến traffic control. Dubbo cũng hỗ trợ Triple protocol (thế hệ RPC communication protocol tiếp theo được định nghĩa trên HTTP/2), application-level service discovery, Dubbo Mesh (Dubbo3 mang đến nhiều tính năng mới thân thiện với cloud-native) và các tính năng khác.

![Dubbo3](https://oss.javaguide.cn/github/javaguide/distributed-system/rpc/image-20220716111545343.png)

Dubbo do Alibaba open source, sau đó gia nhập Apache. Chính nhờ sự xuất hiện của Dubbo mà ngày càng nhiều công ty bắt đầu sử dụng và chấp nhận distributed architecture.

Dubbo là một trong những open-source project nội địa khá xuất sắc, source code của nó cũng rất đáng để học và đọc!

- GitHub: [https://github.com/apache/incubator-dubbo](https://github.com/apache/incubator-dubbo "https://github.com/apache/incubator-dubbo")
- Trang chủ: <https://dubbo.apache.org/zh/>

### Motan

Motan là RPC framework do Sina Weibo open source. Theo thông tin được chia sẻ, framework này đang hỗ trợ hàng trăm tỷ lượt gọi tại Sina Weibo. Tuy nhiên, tác giả hiếm khi thấy công ty nào sử dụng và tài liệu trên mạng cũng khá ít.

Nhiều người thích so sánh Motan với Dubbo vì cả hai đều do các công ty lớn trong nước open source. Sau khi tham khảo nhiều tài liệu và xem qua source code, tác giả nhận thấy: **Motan giống một phiên bản tinh gọn của Dubbo hơn; có thể nó đã tham khảo tư tưởng của Dubbo, nhưng thiết kế của Motan tinh gọn hơn và chức năng thuần túy hơn.**

Tuy nhiên, tôi không khuyến nghị sử dụng Motan trong project thực tế. Nếu công ty bạn đang sử dụng Motan, vẫn nên cân nhắc Dubbo vì community activity và ecosystem của Dubbo tốt hơn nhiều.

- Tìm hiểu thiết kế RPC framework qua Motan: [http://kriszhang.com/motan-rpc-impl/](http://kriszhang.com/motan-rpc-impl/ "http://kriszhang.com/motan-rpc-impl/")
- Tài liệu Motan: [https://github.com/weibocom/motan/wiki/zh_overview](https://github.com/weibocom/motan/wiki/zh_overview "https://github.com/weibocom/motan/wiki/zh_overview")

### gRPC

![](https://oss.javaguide.cn/github/javaguide/distributed-system/rpc/2843b10d-0c2f-4b7e-9c3e-ea4466792a8b.png)

gRPC là một open-source RPC framework hiệu năng cao, đa dụng do Google open source. Framework này chủ yếu hướng đến phát triển mobile application và được thiết kế dựa trên HTTP/2 protocol standard (hỗ trợ bidirectional stream, message header compression và các tính năng khác, giúp tiết kiệm bandwidth hơn), phát triển dựa trên ProtoBuf serialization protocol và hỗ trợ nhiều ngôn ngữ lập trình.

**ProtoBuf là gì?** [ProtoBuf (Protocol Buffer)](https://github.com/protocolbuffers/protobuf) là một data format linh hoạt và hiệu quả hơn, có thể dùng trong communication protocol, data storage và các lĩnh vực khác. ProtoBuf cơ bản hỗ trợ mọi programming language phổ biến và không phụ thuộc platform. Tuy nhiên, việc dùng ProtoBuf để định nghĩa interface và data type khá rườm rà, đây là một điểm hạn chế nhỏ.

![](https://oss.javaguide.cn/github/javaguide/distributed-system/rpc/image-20220716104304033.png)

Phải nói rằng thiết kế communication layer của gRPC rất xuất sắc. Cải tiến communication layer của [Dubbo-go 3.0](https://dubbogo.github.io/) chủ yếu tham khảo gRPC.

Tuy nhiên, thiết kế của gRPC khiến framework này gần như không có service governance capability. Nếu muốn giải quyết vấn đề đó, bạn cần phụ thuộc vào các component khác, chẳng hạn PolarisMesh của Tencent.

- GitHub: [https://github.com/grpc/grpc](https://github.com/grpc/grpc "https://github.com/grpc/grpc")
- Trang chủ: [https://grpc.io/](https://grpc.io/ "https://grpc.io/")

### Thrift

Apache Thrift là cross-language RPC communication framework do Facebook open source, hiện đã được trao tặng cho Apache Foundation quản lý. Nhờ tính năng cross-language và performance xuất sắc, Thrift được sử dụng tại nhiều công ty Internet. Những công ty có năng lực thậm chí còn dựa trên Thrift để phát triển một distributed service framework, bổ sung các chức năng như service registration và service discovery.

`Thrift` hỗ trợ nhiều **programming language** khác nhau, gồm `C++`, `Java`, `Python`, `PHP`, `Ruby`... (nhiều hơn số ngôn ngữ được gRPC hỗ trợ).

- Trang chủ: [https://thrift.apache.org/](https://thrift.apache.org/ "https://thrift.apache.org/")
- Giới thiệu ngắn về Thrift: [https://www.jianshu.com/p/8f25d057a5a9](https://www.jianshu.com/p/8f25d057a5a9 "https://www.jianshu.com/p/8f25d057a5a9")

### Tổng kết

gRPC và Thrift đều hỗ trợ cross-language RPC call, nhưng chúng chỉ cung cấp các chức năng cơ bản nhất của RPC framework, thiếu sự hỗ trợ của một loạt service component và service governance capability đi kèm.

Xét về mức độ hoàn thiện chức năng, ecosystem và community activity, Dubbo là lựa chọn xuất sắc nhất. Dubbo cũng có nhiều case thành công trong nước, chẳng hạn Dangdang và Didi, là một RPC framework trưởng thành, ổn định và đã được kiểm chứng trong production. Quan trọng nhất, bạn có thể tìm thấy rất nhiều tài liệu tham khảo về Dubbo, nên learning cost tương đối thấp.

Hình dưới đây minh họa ecosystem của Dubbo.

![](https://oss.javaguide.cn/github/javaguide/distributed-system/rpc/eee98ff2-8e06-4628-a42b-d30ffcd2831e.png)

Dubbo cũng là một component trong Spring Cloud Alibaba.

![](https://oss.javaguide.cn/github/javaguide/distributed-system/rpc/0d195dae-72bc-4956-8451-3eaf6dd11cbd.png)

Tuy nhiên, Dubbo và Motan chủ yếu được dùng cho Java. Dù hiện nay Dubbo và Motan cũng tương thích với một số ngôn ngữ, không khuyến nghị lựa chọn như vậy. Nếu cần gọi giữa nhiều ngôn ngữ, bạn có thể cân nhắc gRPC.

Tóm lại, nếu technology stack backend của bạn là Java và đang phân vân chọn RPC framework nào, tôi khuyến nghị cân nhắc Dubbo.

## Thiết kế và triển khai một RPC framework như thế nào?

**"Tự viết RPC framework"** là một ebook nội bộ trong [Knowledge Planet](https://javaguide.cn/about-the-author/zhishixingqiu-two-years.html) của tác giả. Tác giả đã viết 12 bài để trình bày cách triển khai một RPC framework đơn giản từ đầu dựa trên Netty+Kyro+Zookeeper.

Dù quy mô nhỏ nhưng đầy đủ các thành phần cần thiết, project code có comment chi tiết, cấu trúc rõ ràng và tích hợp quy chuẩn code structure của Check Style, rất phù hợp để đọc và học.

**Tổng quan nội dung**:

![](https://oss.javaguide.cn/github/javaguide/image-20220308100605485.png)

## Đã có HTTP protocol, tại sao vẫn cần RPC?

Để xem câu trả lời chi tiết cho vấn đề này, hãy đọc bài viết [Đã có HTTP protocol, tại sao vẫn cần RPC?](../../cs-basics/network/http-vs-rpc.md).

<!-- @include: @article-footer.snippet.md -->
