---
title: TCP Keepalive và HTTP Keep-Alive khác nhau như thế nào?
description: So sánh tầng giao thức, tác dụng cốt lõi, hành vi mặc định, cách thu hồi và trường hợp sử dụng điển hình của TCP Keepalive và HTTP Keep-Alive, đồng thời giải thích tiến trình của các cơ chế liên quan đến Keep-Alive trong HTTP/1.0, HTTP/1.1, HTTP/2 và HTTP/3.
category: Computer Basics
tag:
  - Computer Networking
head:
  - - meta
    - name: keywords
      content: TCP Keepalive,HTTP Keep-Alive,Keep-Alive,long connection,short connection,TCP keepalive,HTTP persistent connection,HTTP/1.0,HTTP/1.1,HTTP/2,HTTP/3,QUIC,UDP,SO_KEEPALIVE
---

Xin chào, tôi là Tiểu G. So sánh TCP Keepalive và HTTP Keep-Alive thường xuất hiện trong các câu hỏi phỏng vấn kỹ thuật. Bài viết này sẽ giải thích chi tiết.

Nói đơn giản, hai cơ chế này chỉ có hậu tố giống nhau nhưng hoàn toàn không phải một thứ, vì một cơ chế nằm ở application layer, còn cơ chế kia nằm ở transport layer, tức là không cùng tầng:

- **HTTP Keep-Alive** là cơ chế ở application layer, giải quyết vấn đề: một TCP connection có thể được nhiều HTTP request tái sử dụng hay không, để không phải thực hiện handshake lại sau mỗi request.
- **TCP Keepalive** là cơ chế ở transport layer, giải quyết vấn đề: khi một TCP connection không có dữ liệu qua lại trong thời gian dài, làm thế nào xác định peer còn hoạt động hay không và có cần thu hồi resource mà connection đang chiếm hay không.

![Các tầng của HTTP và TCP trong mô hình bốn tầng TCP/IP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-ip-4-model.png)

Một cơ chế quản lý việc “có tái sử dụng connection hay không”, cơ chế kia quản lý việc “connection còn sống hay không”. Tầng giao thức và mục đích đều khác nhau, chỉ là tên bị trùng.

Hãy tìm hiểu riêng từng cơ chế.

## HTTP Keep-Alive là gì?

Trước hết hãy nói về vấn đề. Hành vi mặc định của HTTP 1.0 là mỗi TCP connection chỉ phục vụ một HTTP request và response. Sau khi server gửi response, nó lập tức yêu cầu đóng connection, client cũng đóng theo, TCP connection bị ngắt ở cả hai chiều.

![Tổng quan về HTTP, Hypertext Transfer Protocol](https://oss.javaguide.cn/github/javaguide/cs-basics/network/http-overview.png)

Khi mở một trang web, có thể cần tải hàng chục resource như HTML, CSS, JS và image. Nếu mỗi resource đều tạo rồi hủy một connection riêng, riêng chi phí cho three-way handshake và four-way handshake đã không nhỏ, hiệu suất sử dụng TCP connection cũng rất thấp.

Vấn đề này rất rõ ràng: **Vì sao một TCP connection không thể phục vụ nhiều HTTP request?**

Vì vậy HTTP đưa vào header `Connection`. Lấy HTTP/1.0 làm ví dụ, client có thể thêm header này vào request:

```
Connection: Keep-Alive
```

Nếu server cũng xác nhận header này trong response, điều đó có nghĩa hai bên đều đồng ý rằng TCP connection được sử dụng cho HTTP transaction này là một **persistent connection**, tức là chưa đóng connection sau khi request/response kết thúc; các HTTP request khác về sau vẫn có thể tiếp tục tái sử dụng connection này, cho đến khi connection idle timeout, đạt giới hạn số request hoặc bị một bên chủ động đóng.

**Hành vi mặc định của Keep-Alive khác nhau giữa các phiên bản HTTP:**

![Hành vi mặc định của Keep-Alive khác nhau giữa các phiên bản HTTP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/different-http-versions-have-different-default-keep-alive-behaviors.png)

- **HTTP 1.0**: mặc định là short connection. Muốn sử dụng persistent connection, request header phải ghi rõ `Connection: Keep-Alive`, đồng thời server cũng phải thêm header này vào response thì mới có hiệu lực.
- **HTTP 1.1**: mặc định đã là persistent connection, không cần khai báo thêm. Nếu muốn đóng connection sau khi request kết thúc, cần chỉ định rõ `Connection: close`. Đây cũng là lý do HTTP/1.1 giảm đáng kể chi phí thiết lập TCP connection và đóng connection so với HTTP/1.0.
- **HTTP/2**: HTTP/2 không còn sử dụng cách “một connection xử lý tuần tự nhiều request” của HTTP/1.x, mà đưa vào cơ chế multiplexing, nghĩa là một TCP connection có thể đồng thời chạy nhiều Stream, request và response có thể truyền xen kẽ, không còn block lẫn nhau, giải quyết vấn đề head-of-line blocking ở application layer của HTTP/1.1. Tuy nhiên, HTTP/2 vẫn chạy trên một TCP connection duy nhất. Khi TCP bên dưới xảy ra packet loss, dữ liệu phía sau vẫn phải chờ retransmission, nên vẫn chịu ảnh hưởng của head-of-line blocking ở TCP layer. Cách điều khiển connection dựa trên HTTP/1.x này không còn ý nghĩa trong HTTP/2. Nói chính xác hơn, các connection-specific header như `Connection`, `Keep-Alive`, `Transfer-Encoding` bị cấm sử dụng trong HTTP/2; message chứa các header này sẽ bị xem là không hợp lệ.
- **HTTP/3**: HTTP/3 dựa trên QUIC và chạy trên UDP, không còn phụ thuộc vào TCP connection, cũng không sử dụng cách điều khiển connection `Connection: Keep-Alive` của HTTP/1.x. QUIC tự quản lý connection, keepalive và multiplexing, đồng thời giảm ảnh hưởng của head-of-line blocking ở transport layer.

Tóm tắt trong một câu: HTTP/1.0 cần Keep-Alive được chỉ định rõ, HTTP/1.1 mặc định tái sử dụng connection, HTTP/2 nâng cấp từ “tái sử dụng connection” thành “multiplexing trên một TCP connection”, còn HTTP/3 chuyển hẳn sang quản lý connection dựa trên QUIC.

## Đóng và thu hồi HTTP persistent connection như thế nào?

Persistent connection nâng cao hiệu suất sử dụng TCP, nhưng cũng mang đến một vấn đề mới: client mở một trang, TCP connection đã được thiết lập, nhưng người dùng lại bỏ đó không dùng. Connection này cứ idle, server không thể chờ vô hạn, nhưng cũng không thể hoàn toàn trông chờ client tự giác đóng connection.

Nếu các idle connection kiểu này tích tụ quá nhiều, resource TCB (TCP Control Block) của server sẽ bị chiếm dụng vô ích.

Cách giải quyết của HTTP là đưa hai parameter vào header `Keep-Alive`:

```
Keep-Alive: timeout=5, max=10
```

- **timeout=5**: khi connection idle quá 5 giây, server có thể đóng connection.
- **max=10**: connection này phục vụ tối đa 10 HTTP request; khi đạt giới hạn, connection sẽ bị đóng bắt buộc.

Có một điểm dễ bị bỏ qua: **khi đạt ngưỡng timeout hoặc max, server có thể đóng connection bất kể client lúc đó có online hay không.** Nếu client vừa khéo tái sử dụng connection cũ này để gửi request mới, có thể gặp tình huống connection đã bị đóng, request thất bại và cần retry.

Nói cách khác, việc thu hồi idle connection của HTTP Keep-Alive thường do cấu hình server chi phối. Client đương nhiên có thể chủ động đóng connection, nhưng server sẽ không luôn chờ client tự quyết định.

Trong cấu hình Web server thực tế, các parameter này do server quyết định. Ví dụ, giá trị mặc định của `keepalive_timeout` trong Nginx là 75 giây, giá trị mặc định của `keepalive_requests` là 1000 (trước Nginx 1.19.10, giá trị mặc định là 100).

## TCP Keepalive là gì?

Vấn đề TCP Keepalive cần giải quyết hoàn toàn khác: nó không quan tâm connection có chạy HTTP request hay không, mà quan tâm **peer thực sự còn hoạt động hay không**.

Hãy xét một tình huống: client và server thiết lập một TCP connection, nhưng client đột ngột mất điện, dây mạng bị rút hoặc hệ thống bị crash. Lúc này server hoàn toàn không biết phía bên kia đã biến mất, vì TCP không giống một cuộc gọi điện thoại, không có “tín hiệu bận”. Connection này trở thành một dead connection ở trạng thái “half-open”, chiếm dụng vô ích resource TCB trong memory của server.

TCP Keepalive được dùng để phát hiện tình huống này. Quy trình hoạt động như sau:

![Nguyên lý hoạt động của TCP Keepalive](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-keepalive-vs-http-keepalive-tcp-keepalive-working-principle.png)

1. Nếu một TCP connection không có dữ liệu qua lại trong một khoảng thời gian (mặc định **7200 giây, tức 2 giờ**), kernel sẽ tự động gửi một **probe packet** đến peer.
2. Nếu peer vẫn online bình thường, nó sẽ reply một ACK, sau đó timer được reset và chờ thêm 2 giờ.
3. Nếu peer không reply, cứ mỗi **75 giây** sẽ gửi lại một probe packet, retry tối đa **9 lần**.
4. Nếu cả 9 lần đều không có reply, kernel xác định connection đã chết, gửi RST để đóng connection và giải phóng resource.

Ba parameter này tương ứng với các configuration của kernel trên Linux:

| Parameter              | Ý nghĩa                                                      | Giá trị mặc định trên Linux |
| ---------------------- | ------------------------------------------------------------ | --------------------------- |
| `tcp_keepalive_time`   | Sau khi connection idle bao lâu thì bắt đầu gửi probe packet | 7200 giây (2 giờ)           |
| `tcp_keepalive_intvl`  | Khoảng thời gian giữa hai probe packet                       | 75 giây                     |
| `tcp_keepalive_probes` | Số lần probe packet được gửi tối đa                          | 9 lần                       |

macOS sử dụng network stack theo phong cách họ BSD, không có `net.ipv4.*`, mà tương ứng là `net.inet.tcp.*`.

![Xem các parameter TCP Keepalive trên Mac](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-keepalive-parameters.jpg)

Theo giá trị mặc định, tính từ lúc connection bắt đầu idle đến khi cuối cùng bị xác định là đã chết, thời gian chờ tối đa là **7200 + 75 × 9 = 7875 giây**, khoảng 2 giờ 11 phút.

Có thể dùng `sysctl` để xem và sửa đổi:

```bash
sysctl net.ipv4.tcp_keepalive_time
sysctl net.ipv4.tcp_keepalive_intvl
sysctl net.ipv4.tcp_keepalive_probes
```

Còn một vấn đề rất dễ mắc phải: **TCP Keepalive mặc định đang tắt.** Ứng dụng phải bật rõ tùy chọn `SO_KEEPALIVE` khi tạo socket, nếu không kernel sẽ không gửi probe packet. Điều này được quy định rõ trong RFC 1122: Keepalive là một tính năng tùy chọn và theo mặc định không được bật.

Sau khi hiểu nguyên lý hoạt động, tính chất của TCP Keepalive trở nên rất rõ ràng: đây là một **cơ chế thu hồi resource “nhẹ nhàng”**. Nó chỉ thu hồi resource sau khi xác nhận peer offline. Chỉ cần peer vẫn online và còn có thể reply ACK, connection này sẽ tiếp tục được duy trì, timer được reset và chờ thêm 2 giờ. Khi peer online, server không có cách nào đóng connection này chỉ bằng TCP Keepalive.

Điều này tạo nên sự tương phản rõ rệt với việc HTTP Keep-Alive “đến thời gian là đóng, không quan tâm bạn có online hay không”.

## Sau khi TCP Keepalive gửi probe, có những tình huống nào xảy ra?

Sau khi kernel gửi probe packet, tùy theo trạng thái thực tế của peer, kết quả có thể khác nhau:

![Cơ chế probe của TCP Keepalive](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-keepalive-vs-http-keepalive-tcp-keepalive-detection-mechanism.png)

**1. Peer vẫn online bình thường**

Peer nhận probe packet và TCP stack reply một ACK. Bên gửi nhận được ACK, reset idle timer về `tcp_keepalive_time` rồi tiếp tục chờ. Connection không bị đóng.

**2. Peer từng crash nhưng đã restart**

Peer vẫn online, nhưng vì đã restart nên kernel không còn context của connection này. Sau khi nhận probe packet, TCP stack của peer sẽ reply một RST (vì nó không nhận ra connection này). Bên gửi nhận RST và lập tức đóng connection.

**3. Peer crash và chưa khôi phục, hoặc network không thể truy cập**

Probe packet được gửi đi nhưng không nhận được reply nào. Bên gửi retry sau mỗi `tcp_keepalive_intvl` giây; nếu liên tiếp `tcp_keepalive_probes` lần không có phản hồi, nó xác định connection đã chết, kernel đóng connection và giải phóng resource.

Tình huống thứ 3 cũng bao gồm một số vấn đề do intermediate network device gây ra. Ví dụ, NAT gateway thường có cơ chế session timeout. Nếu một connection không truyền dữ liệu trong thời gian dài, NAT table entry sẽ bị xóa. Các probe packet gửi sau đó không thể đến peer, kết quả cũng giống như peer bị crash: đều không nhận được reply và cuối cùng timeout rồi đóng connection.

## TCP Keepalive có những giới hạn nào?

TCP Keepalive ở đây là cơ chế probe keep-alive của TCP layer, không phải cơ chế tái sử dụng connection của HTTP Keep-Alive. Nó có thể phát hiện dead connection, nhưng trong production environment thường không đủ nếu chỉ dựa vào nó, vì một số lý do:

**Probe mặc định quá chậm.** Lấy configuration mặc định của Linux làm ví dụ, connection phải idle 7200 giây mới bắt đầu probe; Windows cũng có keep-alive timeout mặc định là 2 giờ. Mức thời gian này quá dài đối với phần lớn connection của dịch vụ online. `net.ipv4.tcp_keepalive_*` của Linux là system default, ảnh hưởng đến các connection chưa được thiết lập riêng; nếu ứng dụng cần phân biệt policy theo từng connection, có thể thiết lập `TCP_KEEPIDLE`, `TCP_KEEPINTVL`, `TCP_KEEPCNT` trên từng socket ở các platform hỗ trợ. Tuy nhiên, không nên dùng các option này làm giải pháp chung cho mọi platform; cụ thể còn phụ thuộc vào việc operating system và language runtime có expose chúng hay không.

**Chỉ phát hiện connection còn sống, không phát hiện application health.** Probe packet TCP Keepalive do kernel gửi; TCP stack của peer nhận được sẽ trực tiếp reply ACK, application layer hoàn toàn không tham gia. Vì vậy nó chỉ cho biết kernel của peer còn có thể nhận packet và trả ACK, không cho biết thread pool của ứng dụng, event loop, database connection pool hay business dependency của peer có hoạt động bình thường hay không. Đây là điểm mù lớn nhất của nó.

**Dễ xác định nhầm đối tượng khi đi qua intermediate layer.** Nếu giữa client và server có NAT, layer-4 load balancer hoặc reverse proxy, trước hết cần xem TCP connection có bị intermediate layer terminate hay không. Nếu intermediate layer chỉ thực hiện NAT/connection tracking, khoảng thời gian Keepalive cần nhỏ hơn idle timeout của intermediate device thì mới có thể tránh table entry bị xóa; nếu intermediate layer terminate TCP connection, thứ backend phát hiện chỉ là connection từ backend đến intermediate layer còn sống hay không, không có nghĩa client thực sự vẫn còn hoạt động.

**Cách triển khai và giá trị mặc định khác nhau giữa các operating system.** Ví dụ, Linux mặc định bắt đầu probe sau 7200 giây, interval là 75 giây và tối đa 9 lần; Windows cũng có timeout mặc định là 2 giờ, nhưng interval mặc định là 1 giây, số probe từ Windows Vista trở đi cố định là 10 và không thể thay đổi; macOS sử dụng network stack theo phong cách họ BSD, không có nhóm sysctl `net.ipv4.*` của Linux, các parameter liên quan thường nằm dưới `net.inet.tcp.*`. Dùng TCP Keepalive để health check connection cross-platform rất khó đảm bảo tính nhất quán; tốt nhất nên xác nhận tên parameter, unit và giá trị mặc định cụ thể bằng cách đo trên hệ thống mục tiêu.

**Không tác động trực tiếp đến HTTP/3/QUIC.** Với HTTP/3/QUIC connection thực sự, TCP Keepalive không tham gia kiểm tra connection còn sống; nhưng nếu client fallback về HTTP/1.1 hoặc HTTP/2 vì UDP bị chặn hay các nguyên nhân khác, TCP connection sau khi fallback vẫn có thể chịu ảnh hưởng của TCP Keepalive. Connection liveness và timeout của HTTP/3 do QUIC xử lý; ví dụ QUIC có idle timeout, khi cần có thể gửi PING frame để liveness testing; khi HTTP/3 layer đóng connection, nó cũng có thể dùng GOAWAY để hỗ trợ graceful close.

Vì vậy trong thực tế, TCP Keepalive thường đóng vai trò biện pháp dự phòng, giúp bạn dọn dẹp những connection chắc chắn đã chết. Nếu cần health check nhanh hơn, chi tiết hơn và có thể nhận biết trạng thái application layer, bạn vẫn cần tự thực hiện heartbeat ở application layer, chẳng hạn WebSocket Ping/Pong, gRPC keepalive ping hoặc heartbeat protocol do business tự định nghĩa.

Application-layer heartbeat cũng không phải càng thường xuyên càng tốt. Heartbeat interval quá ngắn sẽ làm tăng số packet, áp lực lên server timer và xác suất nhận định nhầm trên network yếu; interval quá dài lại khiến việc phát hiện sự cố bị chậm. Cấu hình thực tế cần được quyết định dựa trên quy mô connection, NAT/LB idle timeout và thời gian phát hiện sự cố mà business có thể chấp nhận.

## Tổng hợp so sánh TCP Keepalive và HTTP Keep-Alive

| Tiêu chí so sánh                     | HTTP Keep-Alive                                                                           | TCP Keepalive                                                                                           |
| ------------------------------------ | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| **Tầng giao thức**                   | Application layer (HTTP protocol)                                                         | Transport layer (TCP protocol)                                                                          |
| **Vấn đề giải quyết**                | Tái sử dụng TCP connection, giảm chi phí thiết lập, đóng connection và slow start lặp lại | Probe TCP connection idle trong thời gian dài, giải phóng resource connection sau khi peer mất kết nối  |
| **Hành vi mặc định**                 | HTTP/1.0 mặc định short connection; HTTP/1.1 mặc định persistent connection               | Mặc định tắt, ứng dụng cần bật rõ `SO_KEEPALIVE`                                                        |
| **Mức độ điều khiển**                | Do HTTP client, Web server hoặc proxy điều khiển theo policy của connection               | Do kernel của operating system điều khiển, một số platform cũng cho phép điều chỉnh theo từng socket    |
| **Parameter thường gặp**             | `Connection`, `Keep-Alive: timeout/max`, server timeout configuration                     | `tcp_keepalive_time/intvl/probes` hoặc parameter tương ứng của platform                                 |
| **Điều kiện đóng**                   | Đạt idle timeout, giới hạn số request hoặc một bên chủ động đóng                          | Sau khi connection idle, gửi probe packet; chỉ đóng khi không có response nhiều lần hoặc nhận RST       |
| **Khi peer online**                  | Server vẫn có thể chủ động thu hồi idle connection theo cấu hình                          | Miễn kernel của peer còn reply ACK, connection thường tiếp tục được duy trì                             |
| **Có thể thay thế heartbeat không**  | Không thể xác định business health, chỉ quản lý việc tái sử dụng HTTP connection          | Không thể xác định thread pool của ứng dụng, event loop và business dependency có bình thường hay không |
| **Ảnh hưởng của intermediate layer** | Proxy và gateway có thể độc lập quản lý hai đoạn HTTP/TCP connection trước và sau         | NAT/LB/reverse proxy có thể khiến thứ bạn probe chỉ là một đoạn TCP connection                          |
| **Quan hệ với HTTP/2/3**             | HTTP/2 vô hiệu hóa connection-specific header; HTTP/3/QUIC không sử dụng cơ chế này       | Chỉ áp dụng cho TCP; HTTP/3/QUIC connection thực sự không chịu ảnh hưởng trực tiếp                      |

Nếu nhìn từ góc độ “ai quyết định đóng connection”, cách tiếp cận của hai cơ chế hoàn toàn trái ngược:

HTTP Keep-Alive là “chủ động thu hồi”: khi server đạt timeout hoặc giới hạn số request, nó có thể đóng connection theo configuration của mình mà không cần probe xem peer có online hay không. Đây là một cách thu hồi resource tương đối chủ động.

TCP Keepalive là “bị động thu hồi”: trước hết nó phải gửi probe packet để hỏi “bạn còn đó không?”. Chỉ cần peer online và có thể reply ACK, server chỉ có thể tiếp tục duy trì connection và refresh timer. Chỉ sau khi xác nhận peer đã biến mất thì mới có thể giải phóng resource. Đây là một chiến lược thu hồi nhẹ nhàng.

Trong dự án thực tế, hai cơ chế thường đồng thời hoạt động và mỗi cơ chế quản lý phần của mình. HTTP Keep-Alive quản lý “một connection được sử dụng tối đa bao lâu và phục vụ bao nhiêu request”, còn TCP Keepalive quản lý “nếu lâu không có dữ liệu thì kiểm tra peer có biến mất hay không”. Hai cơ chế không can thiệp lẫn nhau và cũng không thể thay thế cho nhau.
