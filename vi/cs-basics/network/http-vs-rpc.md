---
title: "Có HTTP protocol, tại sao vẫn cần RPC? So sánh HTTP và RPC"
category: Computer Basics
description: "Phân tích sâu sự khác biệt bản chất giữa HTTP và RPC, giải thích cách lựa chọn giao tiếp trong microservice. Bao quát các kiến thức cốt lõi như performance của serialization, connection reuse, gRPC, RESTful và service governance."
head:
  - - meta
    - name: keywords
      content: HTTP,RPC,khác biệt HTTP và RPC,giao tiếp microservice,RPC protocol,TCP communication,serialization protocol,RESTful,gRPC,Dubbo,Protobuf,service call,remote call,HTTP protocol,lựa chọn microservice
---

Xin chào, tôi là Tiểu G. Năm hai đại học, vào học kỳ hai, khi xem khóa học miễn phí của Heima, lần đầu tôi tiếp xúc với RPC và lúc đó khá bối rối.

HTTP interface không phải đã gọi được rồi sao?

Frontend gọi backend dùng HTTP, server gọi server cũng có thể dùng HTTP. Viết một interface `/user/getById`, truyền vào user ID và trả về thông tin user, như vậy chẳng phải cũng hoàn thành remote call sao?

Vậy tại sao còn phải làm thêm một RPC để tăng chi phí học tập? Chẳng phải là tự làm khó mình sao!

Điều dễ khiến mọi người nhầm lẫn hơn là nhiều bài viết rất thích đặt HTTP và RPC cạnh nhau để so sánh, như thể chúng là hai protocol cùng một tầng. Đọc xong, bạn có thể chỉ nhớ vài câu: **HTTP hướng đến resource, RPC hướng đến method; HTTP hướng ra bên ngoài, RPC hướng vào bên trong; RPC có performance tốt hơn.**

Những câu này không hoàn toàn sai, nhưng quá sơ lược.

Đến khi thực sự làm project, bạn vẫn sẽ gặp các vấn đề: **Dùng HTTP có được không? Dùng RPC có phải over-design không? gRPC rõ ràng dựa trên HTTP/2, tại sao lại nói nó là RPC?**

Bài viết này sẽ làm rõ vấn đề đó.

## RPC không phải một protocol cụ thể nào

Đây là một ngộ nhận thường gặp. Trước khi bắt đầu các phần tiếp theo, rất cần nói qua điểm này.

**HTTP là một protocol. Còn RPC không phải một protocol cụ thể nào, mà giống một cách gọi hơn.**

RPC là viết tắt của Remote Procedure Call, nghĩa là gọi procedure từ xa. Vấn đề nó muốn giải quyết rất đơn giản: **khi gọi remote service, cố gắng để bạn gọi nó giống như gọi local method.**

![RPC ẩn các chi tiết remote call thông qua local proxy](https://oss.javaguide.cn/github/javaguide/distributed-system/rpc/rpc-overview.png)

Ví dụ gọi user service trong code local:

```java
User user = userService.getUser(1001);
```

Nếu `userService` nằm trong process hiện tại, đây chỉ là một lần gọi method thông thường.

Nhưng nếu user service được deploy trên một machine khác, mọi chuyện sẽ phức tạp hơn. Bạn phải gửi network request, truyền method name và parameter, serialize data, xử lý timeout, failure, retry, sau đó lấy response về rồi deserialize.

Điều RPC framework muốn làm là đóng gói các rắc rối này hết mức có thể. Code phía caller trông vẫn như sau:

```java
User user = userService.getUser(1001);
```

Nhưng bên dưới đã hoàn tất network communication, serialization, service addressing và trả về result.

Vì vậy, cách nói chính xác hơn không phải là “HTTP và RPC, cái nào mạnh hơn”, mà là:

**HTTP là một application-layer protocol, RPC là một remote-call model.**

![HTTP: Tổng quan về Hypertext Transfer Protocol](https://oss.javaguide.cn/github/javaguide/cs-basics/network/http-overview.png)

Xét về implementation, RPC có thể có nhiều dạng. Dubbo là RPC framework, Thrift là RPC framework, gRPC cũng là RPC framework. Tài liệu chính thức của gRPC cũng nói rất trực tiếp: client có thể gọi method của ứng dụng server trên một machine khác giống như gọi local object; server định nghĩa các method có thể gọi từ xa cùng parameter và return type.

![Dubbo3 sử dụng Triple protocol đồng thời hỗ trợ HTTP/1, HTTP/2 và HTTP/3](https://oss.javaguide.cn/github/javaguide/distributed-system/rpc/image-20220716111545343.png)

Điều này giải thích một điểm rất dễ khiến người ta rối: **gRPC là RPC, nhưng nó dựa trên HTTP/2.**

Nó không đối lập với HTTP, mà cung cấp RPC call trên nền HTTP/2.

Trên GitHub của gRPC có riêng một bài viết [gRPC over HTTP2 RPC protocol dựa trên HTTP2](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md) giới thiệu chi tiết:

![gRPC over HTTP2 - RPC protocol dựa trên HTTP2](https://oss.javaguide.cn/github/javaguide/distributed-system/rpc/grpc-over-http2-github.png)

## **Chỉ có TCP vẫn chưa đủ**

Để hiểu sự khác biệt giữa HTTP và RPC, tốt nhất hãy xem thêm một tầng bên dưới.

Nhiều bạn biết HTTP dựa trên TCP, RPC cũng thường dựa trên TCP, nên sẽ nghĩ: vậy tôi dùng thẳng TCP chẳng phải được rồi sao?

Về lý thuyết thì được, nhưng thực tế rất phiền phức.

TCP chịu trách nhiệm về reliable transport, nó truyền một chuỗi byte stream liên tục. Nó không quan tâm business message của bạn bắt đầu từ đâu và kết thúc ở đâu.

Ví dụ client gửi liên tiếp hai request:

```text
getUser:1001
getOrder:8888
```

Server nhận được có thể không phải là hai message ngay ngắn, mà là một đoạn byte stream. Bạn phải tự xác định: message thứ nhất kết thúc ở đâu, message thứ hai bắt đầu từ đâu. Đồng thời còn phải cân nhắc các vấn đề như half packet, packet sticking, encoding, timeout, error code, request ID.

![Ranh giới message của TCP và UDP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-udp-byte-stream-tcp-udp-message-boundary.png)

Đó là lý do application-layer protocol nhất định phải định nghĩa message format.

HTTP định nghĩa một format dùng chung: request line, Header, Body, status code, v.v. MDN cũng định nghĩa HTTP rất rõ: đây là application-layer protocol, ban đầu được dùng cho communication giữa browser và Web server, nhưng cũng có thể dùng cho communication giữa các machine và API access.

RPC framework cũng sẽ định nghĩa message format của riêng mình. Chỉ là nó thường không thiết kế xoay quanh URL và resource, mà xoay quanh service, method, parameter và return value.

Nói thẳng ra, HTTP và RPC đều đang giải quyết một vấn đề:

**Hai process cách nhau qua network, làm thế nào diễn đạt rõ một business call.**

Chỉ là cách model hóa của chúng khác nhau.

## **HTTP giống truy cập resource hơn, RPC giống gọi method hơn**

Cách viết HTTP / REST thường như sau:

```http
GET /users/1001
POST /orders
PUT /orders/888/status
DELETE /comments/9527
```

Mental model của nó là resource.

`/users/1001` là một user resource, `GET` biểu thị việc đọc nó; `POST /orders` biểu thị việc tạo order; `PUT /orders/888/status` biểu thị việc sửa order status.

Cách này rất phù hợp để mở API ra bên ngoài.

Vì nó generic, dễ hiểu và dễ debug. Browser có thể truy cập, Postman có thể gọi, curl có thể test, gateway cũng dễ xử lý. Khi cung cấp interface cho bên thứ ba, chỉ cần để bên đó tích hợp theo tài liệu HTTP thì rào cản khá thấp.

Cách viết RPC giống như sau hơn:

```java
userService.getUser(1001);
orderService.createOrder(request);
inventoryService.deductStock(skuId, count);
```

Mental model của nó là method call.

Caller quan tâm hơn đến các câu hỏi: tôi cần gọi service nào? Method nào? Truyền parameter gì? Trả về object nào?

![Sơ đồ nguyên lý RPC](https://oss.javaguide.cn/github/javaguide/distributed-system/rpc/rpc-principle.png)

Điều này gần với thói quen viết code của Java backend hơn. Đặc biệt khi gọi nội bộ giữa các microservice, các service vốn đã phối hợp với nhau xoay quanh business method, chẳng hạn tạo order, trừ inventory, truy vấn balance, kiểm tra permission. RPC biểu đạt quan hệ gọi này trực tiếp hơn.

Vì vậy, khác biệt lớn nhất giữa HTTP và RPC không phải là một cái gọi được còn cái kia thì không.

Cả hai đều gọi được.

Khác biệt nằm ở chỗ: **bạn model hóa remote interaction thành một lần resource access hay một lần method call.**

## **Tại sao RPC phổ biến hơn trong nội bộ company?**

HTTP đương nhiên có thể dùng để gọi internal service.

Nhiều company dùng toàn HTTP cho các service nội bộ và vẫn chạy tốt. Đặc biệt khi quy mô service không lớn, call chain không phức tạp, HTTP còn đơn giản hơn.

Nhưng khi số lượng service tăng lên, ưu điểm của RPC sẽ dần trở nên rõ ràng.

**Thay đổi rõ ràng đầu tiên là: caller không muốn quan tâm đối phương đang ở machine nào.**

Khi viết business code, tốt nhất bạn chỉ quan tâm “tôi muốn gọi user service”, thay vì quan tâm user service có bao nhiêu machine, IP là gì, machine nào vừa shutdown, machine nào có weight cao.

Điều này cần đến service discovery.

Mô tả của tài liệu chính thức Dubbo về service discovery rất điển hình: Provider đăng ký address vào registry, Consumer đọc và subscribe danh sách address từ registry; khi address thay đổi, registry thông báo cho consumer. Dubbo hỗ trợ các registry phổ biến như Nacos, Consul và ZooKeeper.

![Các role cốt lõi trong kiến trúc Dubbo](https://oss.javaguide.cn/%E6%BA%90%E7%A0%81/dubbo/dubbo-relation.jpg)

Những capability này đương nhiên cũng có thể thực hiện bằng HTTP. Bạn có thể tự ghép một bộ gồm registry, gateway, load balancing và SDK.

Nhưng RPC framework thường đưa trực tiếp những thứ này vào service-call system.

Caller viết service interface, bên dưới tự động hoàn tất service discovery, load balancing, connection management và timeout control. Business code không cần ghép URL ở khắp nơi.

**Thay đổi thứ hai là: interface contract trở nên quan trọng hơn.**

HTTP + JSON rất linh hoạt, nhưng sự linh hoạt cũng có nghĩa là dễ lỏng lẻo.

Field name đổi, type đổi, thêm một enum value, caller có thể chỉ phát hiện lỗi khi runtime. Nếu interface document không được cập nhật kịp thời, lúc integration test sẽ rất khổ.

RPC framework thường dùng contract chặt chẽ hơn để ràng buộc hai bên. Lấy gRPC làm ví dụ, nó thường dùng Protocol Buffers làm interface definition language và message exchange format. Tài liệu chính thức Protocol Buffers cũng nói rõ đây là một cơ chế serialization cho structured data, độc lập với language và platform, có thể mở rộng; có thể dùng `.proto` để định nghĩa structure và generate code cho các language khác nhau.

Lợi ích mang lại là thay đổi interface dễ bị phát hiện hơn ở giai đoạn code generation và compilation.

Đương nhiên, contract chặt không có nghĩa là sẽ không xảy ra sự cố.

Cách compatible field, client version cũ xử lý thế nào, field mới có thể xóa hay không, enum có thể sửa hay không, tất cả vẫn cần được thiết kế cẩn thận. Chỉ là so với việc “mọi người thống nhất với nhau về JSON field”, IDL sẽ cứng rắn hơn một chút.

**Thay đổi thứ ba là: các internal call có tần suất cao sẽ quan tâm hơn đến processing efficiency của machine.**

Ưu điểm của HTTP + JSON là readability cao, con người đọc thấy dễ chịu. Nhưng khi machine xử lý, đây không phải cách tiết kiệm nhất. Field name, text format và parsing cost đều tạo ra overhead bổ sung.

RPC framework thường dùng binary serialization, chẳng hạn Protobuf và Thrift. Kích thước nhỏ hơn, parsing cũng phù hợp với machine processing hơn.

Nhưng không thể khẳng định tuyệt đối ở điểm này.

Câu “RPC chắc chắn nhanh hơn HTTP” không chặt chẽ. HTTP/2, connection reuse, compression, các JSON library khác nhau và network environment khác nhau đều ảnh hưởng đến kết quả. Bản thân gRPC cũng dựa trên HTTP/2, ưu điểm của nó không thể giải thích hết bằng một câu “nó không phải HTTP”.

Cách nói chắc chắn hơn là:

**Trong các scenario service inter-call có tần suất cao, RPC framework thường triển khai các capability như serialization, connection reuse, timeout, retry, load balancing và distributed tracing sát với nhu cầu gọi internal service hơn.**

Đây mới là lý do nó phổ biến trong company.

## **Giá trị của RPC không chỉ là “gọi nhanh hơn một chút”**

Khi nói về RPC, nhiều người thích tập trung vào performance.

Performance đương nhiên quan trọng, nhưng theo tôi giá trị lớn hơn của RPC là service governance.

Sau khi một internal call thực sự được đưa lên production, mọi việc không chỉ đơn giản là gửi request rồi nhận response. Rất nhanh bạn sẽ gặp hàng loạt vấn đề:

- Thời gian timeout của call này nên đặt bao nhiêu? Khi failure có nên retry không? Retry có gây ra duplicate charge không?
- Khi downstream service bị down, upstream có nên degrade không?
- Interface nào gần đây có error rate tăng?
- Một user request đi qua bao nhiêu service?

Nếu tất cả vấn đề này đều do business code xử lý, mọi thứ sẽ nhanh chóng trở nên rối loạn.

RPC framework thường gắn với các capability governance, chẳng hạn timeout control, load balancing, service discovery, circuit breaking và degradation, distributed tracing, call statistics. Phần giới thiệu chính thức của gRPC cũng nhắc đến các capability có thể cắm như load balancing, Tracing, health check và authentication.

HTTP cũng có thể làm những việc này.

Nhiều company dùng API Gateway, service mesh, HTTP SDK, interceptor và distributed tracing component để bổ sung. Làm tốt thì cũng không có vấn đề.

Vì vậy, đừng hiểu RPC là “thứ cao cấp hơn HTTP”. Nó giống việc sắp xếp lại một loạt vấn đề thường gặp trong internal service call theo hướng “remote method call”.

## **Vậy HTTP không phù hợp với internal call sao?**

Không phải vậy. Nếu quy mô service không lớn, team cũng không đông, dùng HTTP ngược lại còn đỡ lo hơn.

Ví dụ một hệ thống quản trị backend được tách thành vài service, tần suất call cũng không cao. Bạn dùng Spring Boot viết vài REST interface, kết hợp với OpenAPI document, unified error code, gateway authentication và log tracing là hoàn toàn đủ.

Cố ép dùng RPC có thể còn mang đến thêm cost, không có ý nghĩa.

Bạn phải đưa vào registry, maintain IDL, xử lý code generation, training team, đồng thời giải quyết vấn đề local debugging và gateway forwarding. Khi chỉ có vài service, call chain cũng không phức tạp, những cost này chưa chắc đã đáng.

HTTP phù hợp với các scenario sau:

- API mở ra bên ngoài, chẳng hạn Web, App và tích hợp với bên thứ ba;
- Team coi trọng tính generic và sự thuận tiện khi debug hơn;
- Tần suất service call không cao;
- Chưa có RPC infrastructure trưởng thành;
- Đã có hệ thống unified HTTP gateway, SDK, rate limiting, authentication và monitoring.

Có một cách phán đoán rất đơn giản, chia sẻ với bạn:

**Nếu hệ thống của bạn đã chạy ổn định bằng HTTP và cũng không có pain point rõ ràng về call governance, thì không cần đổi sang RPC chỉ vì muốn “có chất microservice hơn”.**

Technology selection không phải là dán nhãn.

Giải quyết vấn đề một cách ổn định mới quan trọng hơn.

## **Tại sao gRPC dễ khiến người ta rối?**

gRPC thường khiến mọi người nhầm lẫn vì nó đồng thời nằm trên hai concept.

**Một mặt, nó là RPC framework.**

Bạn định nghĩa service và method, generate code cho client và server, sau đó gọi remote service như gọi method.

**Mặt khác, nó dựa trên HTTP/2 để truyền tải.**

![gRPC over HTTP2 - RPC protocol dựa trên HTTP2](https://oss.javaguide.cn/github/javaguide/distributed-system/rpc/grpc-over-http2-github.png)

Vì vậy, bạn không thể đơn giản hiểu nó là “thứ đối lập với HTTP”.

Cách nói chính xác hơn là: **gRPC dùng HTTP/2 để transport, mặc định dùng Protobuf làm IDL và message serialization format, sau đó dùng RPC model để tổ chức việc gọi.**

Cần lưu ý, Protobuf là lựa chọn kết hợp mặc định phổ biến nhất của gRPC, nhưng không phải bản thân định nghĩa của gRPC. Ở protocol layer, gRPC cho phép `application/grpc+proto`, `application/grpc+json` hoặc custom encoding.

Còn một điểm thường bị bỏ qua: trong response gRPC thông thường, HTTP layer thường là `:status: 200`, còn kết quả call thực sự nằm trong `grpc-status` và `grpc-message` ở HTTP/2 Trailers.

Điều này tạo ra một khác biệt rất thực tế khi troubleshooting.

Khi xem HTTP interface, chúng ta thường xem HTTP status code trước. `200` về cơ bản biểu thị request thành công, `404` biểu thị resource không tồn tại, `500` biểu thị server exception.

Nhưng khi xem gRPC, không thể chỉ nhìn HTTP status code. HTTP là 200 không có nghĩa business call của RPC lần này chắc chắn thành công, mà còn phải xem tiếp `grpc-status`.

Điều này cũng tạo ra một vấn đề engineering: gateway, load balancing, proxy và Service Mesh có support HTTP/2 Trailers chính xác hay không sẽ ảnh hưởng trực tiếp đến gRPC call. Nếu một component trong chain xử lý Trailers không tốt, vấn đề sẽ rất khó phát hiện.

Vì vậy, gRPC không đơn giản là “HTTP/2 + Protobuf”.

Ở tầng HTTP, nó chạy trên HTTP/2.

Về encoding, mặc định kết hợp với Protobuf, nhưng protocol cho phép encoding khác.

Về trải nghiệm gọi, nó giúp bạn gọi remote service giống như gọi local method.

Về việc trả status, nó lại dùng HTTP/2 Trailers để mang kết quả RPC call.

Các thành phần này chồng lên nhau mới là lý do nó dễ khiến người ta rối.

## **Khi lựa chọn thực tế, đừng hỏi cái nào cao cấp hơn**

Tôi khuyên bạn nên lựa chọn theo quan hệ gọi:

- Nếu là browser, mobile client hoặc third-party system gọi, hãy ưu tiên HTTP. Lý do rất đơn giản: generic, cost tích hợp thấp và có nhiều debug tool. Interface đối ngoại sợ nhất là đối phương không tích hợp được. HTTP có ưu thế quá rõ ràng ở điểm này.
- Nếu là các microservice nội bộ gọi nhau với tần suất cao, có thể cân nhắc RPC. Đặc biệt khi số lượng service nhiều, số lượng interface nhiều, call chain phức tạp và yêu cầu về timeout, retry, service registration and discovery, distributed tracing, load balancing đều cao, RPC framework sẽ giúp bỏ đi rất nhiều công việc lặp lại.
- Nếu team đã có HTTP infrastructure trưởng thành thì cũng không cần cố dùng RPC. Ví dụ đã có unified gateway, service discovery, SDK, distributed tracing, rate limiting và circuit breaking, mọi người cũng đã quen dùng HTTP, thì tiếp tục dùng HTTP không có vấn đề.

Nếu muốn dùng gRPC, cần suy nghĩ trước vài vấn đề:

- Browser không thể trực tiếp dùng standard gRPC như backend service, thường cần gRPC-Web hoặc proxy layer;
- Gateway và load balancing có support hay không; local debugging có thuận tiện không;
- Team có chấp nhận `.proto` và code generation hay không;
- Khi troubleshooting trên production, binary message có làm tăng cost tìm hiểu hay không.

gRPC rất mạnh, nhưng không phải không có cost.

Điểm này cần nói rõ trước.

## Một số ngộ nhận thường gặp

### HTTP và RPC, cái nào có performance tốt hơn?

Không thể kết luận một chiều.

Nếu lấy HTTP/1.1 + JSON so với gRPC dựa trên HTTP/2 + Protobuf, trong scenario internal call tần suất cao, vế sau thường tiết kiệm hơn.

Nhưng nếu thay implementation, kết quả có thể khác.

Message size, serialization method, connection reuse, compression, framework implementation và network environment đều ảnh hưởng đến kết quả. Muốn so sánh thực sự, nên dùng interface, data volume và deployment environment của chính bạn để performance test, thay vì học thuộc câu “RPC nhanh hơn”.

### RPC có phải chỉ có thể chạy trên TCP?

Không phải.

RPC là call model, không phải transport protocol. Nó có thể dựa trên TCP, cũng có thể dựa trên HTTP/2. gRPC chính là một ví dụ rất điển hình.

### REST và RPC có loại trừ lẫn nhau không?

Không hoàn toàn loại trừ.

REST thiên về resource modeling, RPC thiên về method call. Trong project thực tế, chúng thường được dùng kết hợp: external interface dùng REST, internal service dùng RPC. Điều này hoàn toàn bình thường.

### Có HTTP/2 rồi thì còn cần RPC không?

HTTP/2 đưa vào HTTP layer các capability như frame, stream, multiplexing và header compression, nâng cao mức tận dụng concurrency trên cùng một TCP connection.

Nhưng nó không tự động giúp bạn định nghĩa service interface, không tự động generate client code, cũng không tự động giải quyết service discovery, timeout retry, call governance và version contract.

Còn một khác biệt rất dễ bị bỏ qua: call mode.

HTTP API thông thường phần lớn là hỏi và trả lời một lần. Ngoài Unary call phổ biến nhất, gRPC còn native support server stream, client stream và bidirectional stream. Tài liệu chính thức gRPC cũng liệt kê rõ bốn call mode là Unary, Server streaming, Client streaming và Bidirectional streaming.

Ví dụ các scenario như subscribe log, push tiến độ long-running task, upload theo batch và real-time sync sẽ tự nhiên hơn khi dùng streaming. Tất nhiên bạn cũng có thể dùng SSE, WebSocket hoặc tự đóng gói dựa trên HTTP/2, nhưng như vậy tương đương với việc lại bổ sung phần capability mà RPC framework đã làm sẵn.

Vì vậy HTTP/2 rất quan trọng, nhưng nó không phải toàn bộ RPC framework.

### gRPC có phải bằng HTTP/2 + Protobuf không?

Không phải.

Câu này chỉ có thể dùng để giúp người mới nhanh chóng tạo ấn tượng ban đầu, không thể xem là định nghĩa nghiêm ngặt.

Cách nói chính xác hơn là: gRPC dựa trên HTTP/2 để mang RPC call, mặc định dùng Protobuf để mô tả interface và message, nhưng protocol bản thân cho phép JSON hoặc custom encoding; đồng thời nó còn định nghĩa một bộ quy tắc hoàn chỉnh gồm request path, Content-Type, Length-Prefixed-Message, `grpc-status` trong Trailers, v.v.

Vì vậy gRPC không chỉ đơn giản là thay một serialization format, mà là một RPC call protocol và một bộ engineering convention.

## Cuối cùng

HTTP và RPC không phải quan hệ một bên thay thế bên kia, cũng không phải vấn đề cái nào cao cấp hơn.

HTTP có thể gọi service, RPC cũng có thể gọi service. Khác biệt thực sự nằm ở chỗ bạn muốn xem remote call là một lần “resource access” hay một lần “method call”.

Nếu là external interface, chẳng hạn Web, App hoặc tích hợp với third-party system, HTTP thường phù hợp hơn. Nó generic, dễ debug, cost tích hợp thấp; đối phương có thể dùng Postman hoặc curl để test.
Nếu là internal service inter-call, đặc biệt khi có nhiều service, call chain dài, interface được gọi thường xuyên và còn phải cân nhắc các vấn đề như service discovery, timeout, retry, load balancing và distributed tracing, RPC sẽ thuận tiện hơn. Nó không chỉ nhằm tăng tốc, mà còn xử lý cùng lúc nhiều việc phiền phức trong internal service call.

Vì vậy, đừng đơn giản học thuộc câu “HTTP hướng ra bên ngoài, RPC hướng vào bên trong” nữa.

Câu này có thể giúp nhập môn, nhưng khi thực sự làm project, vẫn phải xem đối tượng gọi, infrastructure của team, cost troubleshooting, yêu cầu performance và cost maintain về sau.

Quy mô system không lớn, dùng HTTP đã chạy ổn định thì đừng cố dùng RPC chỉ để “trông có vẻ microservice hơn”.

Khi internal call ngày càng phức tạp, HTTP SDK, gateway, monitoring, retry và những thứ khác được bổ sung ngày càng nhiều, bạn có thể nghiêm túc cân nhắc RPC.

Một câu: **HTTP không yếu đến vậy, RPC cũng không thần kỳ đến thế. Chọn cái nào chủ yếu phụ thuộc vào việc nó có thể giải quyết vấn đề hiện tại của bạn với cost thấp hơn hay không.**
