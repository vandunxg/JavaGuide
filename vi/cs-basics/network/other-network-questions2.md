---
title: Tổng hợp câu hỏi phỏng vấn mạng máy tính thường gặp (phần dưới)
description: Tổng hợp các câu hỏi phỏng vấn mạng máy tính thường gặp (phần dưới), bao quát kiến thức cơ bản như TCP/UDP, quản lý kết nối, truyền dữ liệu tin cậy, HTTP/3, IP, IPv6, NAT và ARP.
category: Kiến thức cơ sở máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: câu hỏi phỏng vấn mạng máy tính,TCP vs UDP,bắt tay ba bước TCP,HTTP/3 QUIC,IPv4 vs IPv6,độ tin cậy TCP,địa chỉ IP,giao thức NAT,giao thức ARP,phỏng vấn tầng transport,câu hỏi thường gặp tầng network,dựa trên giao thức TCP,dựa trên giao thức UDP,head-of-line blocking,đóng kết nối bốn bước
---

Trong các câu hỏi phỏng vấn mạng máy tính, phần dễ bị hỏi sâu vào chi tiết thường tập trung ở các kiến thức thuộc transport layer và network layer như **TCP, UDP, IP, ARP, NAT, IPv4/IPv6**. Ví dụ: Vì sao TCP đáng tin cậy? Vì sao cần bắt tay ba bước và đóng kết nối bốn bước? Vì sao HTTP/3 chuyển sang QUIC dựa trên UDP? Những câu hỏi này không chỉ kiểm tra khái niệm, mà còn kiểm tra sự hiểu biết của bạn về quá trình giao tiếp mạng.

Bài viết 《Tổng hợp câu hỏi phỏng vấn mạng máy tính thường gặp (phần dưới)》 sẽ tập trung hệ thống hóa các nội dung thường gặp trong phỏng vấn backend như TCP và UDP, quản lý kết nối TCP, truyền dữ liệu tin cậy, địa chỉ IP, ARP, NAT, giúp bạn kết nối các trọng tâm của transport layer và network layer.

## TCP và UDP

### ⭐️ Sự khác nhau giữa TCP và UDP (quan trọng)

1. **Có hướng kết nối hay không**:
   - TCP có hướng kết nối. Trước khi truyền dữ liệu, trước tiên phải thiết lập kết nối thông qua “bắt tay ba bước”; sau khi truyền dữ liệu xong, còn cần giải phóng kết nối thông qua “đóng kết nối bốn bước”. Điều này bảo đảm hai bên đã sẵn sàng giao tiếp.
   - UDP không kết nối. Trước khi gửi dữ liệu không cần thiết lập bất kỳ kết nối nào, chỉ cần gửi thẳng packet (datagram) đi.
2. **Có phải là truyền tin tin cậy hay không**:
   - TCP cung cấp dịch vụ truyền dữ liệu tin cậy. TCP sử dụng một loạt cơ chế như sequence number, xác nhận (ACK), retransmission khi timeout, flow control và congestion control để bảo đảm dữ liệu đến đích không lỗi, không mất, không trùng lặp và đúng thứ tự.
   - UDP cung cấp phương thức truyền không tin cậy. UDP cố gắng hết sức để chuyển phát (best-effort delivery), nhưng không bảo đảm dữ liệu nhất định sẽ đến nơi, cũng không bảo đảm thứ tự đến nơi, càng không tự động retransmission. Sau khi nhận packet, bên nhận cũng không chủ động gửi xác nhận.
3. **Có state hay không**:
   - TCP có state. Vì phải bảo đảm độ tin cậy, TCP cần duy trì thông tin state của kết nối ở hai đầu kết nối, chẳng hạn sequence number, window size, dữ liệu nào đã gửi, dữ liệu nào đã nhận xác nhận.
   - UDP stateless. UDP không duy trì state của kết nối. Sau khi bên gửi gửi dữ liệu, nó không còn quan tâm dữ liệu có đến nơi hay không và đến nơi bằng cách nào, vì vậy overhead nhỏ hơn (**rất “tệ bạc”!**).
4. **Hiệu suất truyền**:
   - Vì TCP cần thiết lập kết nối, gửi xác nhận và xử lý retransmission nên overhead lớn, hiệu suất truyền tương đối thấp.
   - UDP có cấu trúc đơn giản, không có cơ chế điều khiển phức tạp, overhead nhỏ, hiệu suất truyền cao hơn và tốc độ nhanh hơn.
5. **Hình thức truyền**:
   - TCP hướng byte stream (Byte Stream). TCP xem dữ liệu do application cung cấp là một chuỗi byte không có cấu trúc, có thể chia nhỏ hoặc gộp dữ liệu.
   - UDP hướng message (Message Oriented). Application cung cấp cho UDP block dữ liệu lớn bao nhiêu thì UDP gửi nguyên như vậy, không chia nhỏ cũng không gộp lại, giữ nguyên ranh giới message của application.
6. **Overhead header**:
   - Header của TCP ít nhất cần 20 byte, nếu chứa field option thì tối đa có thể đạt 60 byte.
   - Header của UDP rất đơn giản, cố định chỉ có 8 byte.
7. **Có cung cấp dịch vụ broadcast hoặc multicast hay không**:
   - TCP chỉ hỗ trợ giao tiếp unicast point-to-point (Point-to-Point).
   - UDP hỗ trợ các phương thức giao tiếp one-to-one (unicast), one-to-many (multicast/Multicast) và one-to-all (broadcast/Broadcast).
8. ……

Để so sánh trực quan hơn, bạn có thể xem bảng dưới đây:

| Đặc tính                | TCP                        | UDP                                          |
| ----------------------- | -------------------------- | -------------------------------------------- |
| **Tính kết nối**        | Có hướng kết nối           | Không kết nối                                |
| **Độ tin cậy**          | Tin cậy                    | Không tin cậy (best-effort)                  |
| **Duy trì state**       | Có state                   | Stateless                                    |
| **Hiệu suất truyền**    | Thấp                       | Cao                                          |
| **Hình thức truyền**    | Hướng byte stream          | Hướng datagram (message)                     |
| **Overhead header**     | 20 - 60 byte               | 8 byte                                       |
| **Mô hình giao tiếp**   | Point-to-point (unicast)   | Unicast, multicast, broadcast                |
| **Ứng dụng thường gặp** | HTTP/HTTPS, FTP, SMTP, SSH | DNS, DHCP, SNMP, TFTP, VoIP, video streaming |

### ⭐️ Khi nào chọn TCP, khi nào chọn UDP?

Chọn TCP hay UDP chủ yếu phụ thuộc vào **mức độ yêu cầu về độ tin cậy của việc truyền dữ liệu, cũng như yêu cầu về tính real-time và hiệu suất**.

Khi **độ chính xác và tính toàn vẹn của dữ liệu đặc biệt quan trọng, tuyệt đối không được xảy ra lỗi**, thông thường nên chọn TCP. Vì TCP cung cấp một bộ cơ chế hoàn chỉnh (bắt tay ba bước, xác nhận, retransmission, flow control, congestion control...) để bảo đảm dữ liệu được chuyển đến nơi một cách tin cậy và đúng thứ tự. Các trường hợp sử dụng điển hình như sau:

- **Duyệt Web (HTTP/HTTPS):** Nội dung trang Web, ảnh, script phải được tải đầy đủ thì mới hiển thị chính xác.
- **Truyền file (FTP, SCP):** Nội dung file không được mất hoặc sai thứ tự bất kỳ byte nào.
- **Gửi và nhận email (SMTP, POP3, IMAP):** Nội dung email cần được chuyển đến đầy đủ và chính xác.
- **Đăng nhập từ xa (SSH, Telnet):** Command và response cần được truyền chính xác.
- ……

Khi **tính real-time, tốc độ và hiệu suất được ưu tiên, đồng thời application có thể chịu được một lượng nhỏ dữ liệu bị mất hoặc sai thứ tự**, thông thường nên chọn UDP. UDP có overhead nhỏ, truyền nhanh, không có quy trình phức tạp để thiết lập kết nối và bảo đảm độ tin cậy. Các trường hợp sử dụng điển hình như sau:

- **Giao tiếp audio/video real-time (VoIP, video conference, livestream):** Thỉnh thoảng mất một hoặc hai packet (có thể khiến hình ảnh hoặc âm thanh bị giật trong thời gian ngắn) thường dễ chấp nhận hơn việc phải chờ retransmission (cơ chế của TCP) gây ra độ trễ dài. Application layer có thể có cơ chế bù riêng.
- **Game online:** Cần truyền nhanh các thông tin như vị trí, state của người chơi, yêu cầu về tính real-time cực kỳ cao; dữ liệu cũ nhanh chóng không còn hữu ích, nên mất một lượng nhỏ dữ liệu thường không ảnh hưởng nhiều.
- **DHCP (Dynamic Host Configuration Protocol):** Khi client yêu cầu IP, bản thân client chưa có IP nên không thể đáp ứng điều kiện tiên quyết để TCP thiết lập kết nối; đồng thời DHCP có nhu cầu broadcast, mô hình tương tác đơn giản và cơ chế reliability tích hợp sẵn.
- **Báo cáo dữ liệu IoT:** Trong một số trường hợp, sensor định kỳ báo cáo dữ liệu; mất một vài data point có thể không ảnh hưởng đến việc phân tích xu hướng tổng thể.
- ……

### HTTP dựa trên TCP hay UDP?

~~**HTTP protocol dựa trên TCP protocol**, vì vậy trước khi gửi HTTP request, trước tiên phải thiết lập TCP connection, tức là phải trải qua bắt tay 3 lần.~~

🐛 Đính chính (tham khảo [issue#1915](https://github.com/Snailclimb/JavaGuide/issues/1915)):

Trước HTTP/3.0, HTTP dựa trên TCP protocol; còn HTTP/3.0 không còn sử dụng TCP mà chuyển sang **QUIC protocol dựa trên UDP**:

- **HTTP/1.x và HTTP/2.0:** Hai version HTTP này đều được xây dựng rõ ràng trên TCP. TCP cung cấp phương thức truyền tin cậy, có hướng kết nối, bảo đảm dữ liệu đến nơi đúng thứ tự và không lỗi, điều này rất quan trọng đối với việc hiển thị chính xác nội dung Web. Trước khi gửi HTTP request, cần dùng bắt tay ba bước của TCP để thiết lập connection.
- **HTTP/3.0:** Đây là một thay đổi lớn. HTTP/3 loại bỏ TCP và chuyển sang sử dụng QUIC protocol, còn QUIC được xây dựng trên UDP.

![So sánh protocol stack HTTP/1, HTTP/2 và HTTP/3](https://oss.javaguide.cn/github/javaguide/cs-basics/network/http-3-implementation.png)

**Vì sao HTTP/3 phải thực hiện thay đổi này? Chủ yếu có hai nguyên nhân:**

1. Giải quyết vấn đề head-of-line blocking (Head-of-Line Blocking, viết tắt: HOL blocking).
2. Giảm độ trễ khi thiết lập connection.

Sau đây là phần giới thiệu chi tiết về hai cải tiến này.

Trong HTTP/2, dù có thể truyền đồng thời nhiều request/response stream trên một TCP connection (multiplexing), nhưng đặc tính của bản thân TCP (bảo đảm đúng thứ tự và độ tin cậy) khiến nếu một TCP packet của một stream bị mất hoặc trễ, toàn bộ TCP connection sẽ bị block để chờ retransmission packet đó. Điều này khiến tất cả HTTP/2 stream trên TCP connection này đều bị ảnh hưởng, ngay cả khi packet của các stream khác đã đến nơi. **QUIC (chạy trên UDP) giải quyết vấn đề này**. Bên trong QUIC có cơ chế multiplexing và flow control riêng. Các request/response stream khác nhau thực sự độc lập ở tầng QUIC. Nếu packet của một stream bị mất, nó chỉ block stream đó, không ảnh hưởng đến các stream khác trên cùng QUIC connection (về bản chất là multiplexing + polling), giúp nâng cao đáng kể hiệu suất truyền đồng thời.

Ngoài việc giải quyết vấn đề head-of-line blocking, HTTP/3.0 còn có thể giảm độ trễ của quá trình handshake. Trong HTTP/2.0, nếu muốn thiết lập một HTTPS connection an toàn, cần trải qua TCP handshake ba bước và TLS handshake:

RTT là thời gian round trip để packet đi từ một đầu đến đầu bên kia rồi quay lại, không phải thời gian truyền một chiều. Khi nói về độ trễ của TCP handshake, phải nêu rõ mốc kết thúc đo: sau khi client gửi SYN, khoảng 1 RTT sau sẽ nhận được SYN-ACK, sau đó có thể gửi ACK cuối cùng và application request; server còn cần thêm độ trễ một chiều để nhận ACK và request đó. Khi so sánh HTTP/2 và HTTP/3, cần thống nhất sử dụng cùng một chỉ số như “khi nào client có thể gửi request đầu tiên” hoặc “khi nào client nhận được byte đầu tiên”.

HTTPS connection của HTTP/2 trước tiên cần thiết lập TCP connection, sau đó hoàn tất TLS handshake. HTTP/3 kết hợp việc thương lượng transport parameters và TLS 1.3 handshake trong quá trình QUIC thiết lập connection. QUIC connection mới thường sử dụng 1-RTT; 0-RTT chỉ áp dụng cho trường hợp khôi phục khi client đang giữ state của connection trước đó, có thể mang early data trong packet đầu tiên, nhưng tồn tại rủi ro replay, chỉ phù hợp với request an toàn khi replay.

Có thể tham khảo hai link dưới đây để xem thêm thông tin liên quan:

- <https://zh.wikipedia.org/zh/HTTP/3>
- <https://datatracker.ietf.org/doc/rfc9114/>

### Vì sao TCP hướng byte stream, còn UDP hướng message?

![Ranh giới message của TCP và UDP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-udp-byte-stream-tcp-udp-message-boundary.png)

TCP hướng byte stream. Dữ liệu do application layer ghi vào sẽ đi vào kernel buffer; TCP chỉ bảo đảm các byte này đến đầu bên kia một cách tin cậy và đúng thứ tự, không bảo đảm một lần `send()` tương ứng với một lần `recv()`, cũng không giữ ranh giới message của application layer. Vì vậy bên nhận có thể đọc được nhiều message trong một lần, hoặc chỉ đọc được nửa message, đây là hiện tượng thường gọi là sticky packet và packet splitting.
![Vì sao TCP xảy ra sticky packet / packet splitting?](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-udp-byte-stream-tcp-sticky-split-causes.png)

UDP hướng message. Mỗi lần application layer giao dữ liệu cho UDP sẽ được gửi dưới dạng một UDP datagram, phía nhận cũng đọc theo datagram, do đó tự nhiên giữ nguyên ranh giới message. Tuy nhiên UDP không bảo đảm dữ liệu đến nơi tin cậy, cũng không bảo đảm thứ tự.

Bản chất của việc giải quyết sticky packet/packet splitting của TCP là application layer protocol tự định nghĩa ranh giới message. Các phương án thường gặp gồm độ dài cố định, delimiter và length header. Trong thực tế, length header được sử dụng phổ biến hơn vì thân thiện hơn với binary protocol và message có độ dài thay đổi, nhưng cần xử lý byte order, giới hạn độ dài tối đa, bộ đệm packet chưa hoàn chỉnh và việc đóng connection bất thường.

![Application layer định nghĩa ranh giới message như thế nào?](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-udp-byte-stream-tcp-message-boundary-solutions.png)

Giới thiệu chi tiết: [Vì sao TCP hướng byte stream, còn UDP hướng message?](./tcp-byte-stream-udp-datagram.md)

### Bạn biết những protocol nào dựa trên TCP/UDP?

TCP (Transmission Control Protocol) và UDP (User Datagram Protocol) là hai protocol cốt lõi của transport layer Internet, cung cấp dịch vụ giao tiếp cơ sở cho nhiều application layer protocol. Dưới đây là một số application layer protocol phổ biến được xây dựng lần lượt trên TCP và UDP:

**Các protocol chạy trên TCP (nhấn mạnh truyền tin cậy, đúng thứ tự):**

| Tên protocol (viết tắt)                    | Tên đầy đủ tiếng Anh               | Công dụng chính                                    | Mô tả và đặc tính                                                                                                                                                                                                 |
| ------------------------------------------ | ---------------------------------- | -------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Hypertext Transfer Protocol (HTTP)         | HyperText Transfer Protocol        | Truyền Web page, hypertext và nội dung multimedia  | **HTTP/1.x và HTTP/2 dựa trên TCP**. Các version đầu không mã hóa, là nền tảng của giao tiếp Web.                                                                                                                 |
| Hypertext Transfer Protocol Secure (HTTPS) | HyperText Transfer Protocol Secure | Truyền Web được mã hóa                             | Sử dụng TLS để bảo vệ HTTP. HTTP/1.1 và HTTP/2 thường dùng TLS over TCP, HTTP/3 sử dụng QUIC tích hợp TLS 1.3.                                                                                                    |
| File Transfer Protocol (FTP)               | File Transfer Protocol             | Truyền file                                        | FTP truyền **plaintext** theo cách truyền thống, không an toàn. Khuyến nghị sử dụng version an toàn là **SFTP (SSH File Transfer Protocol)** hoặc **FTPS (FTP over SSL/TLS)**.                                    |
| Simple Mail Transfer Protocol (SMTP)       | Simple Mail Transfer Protocol      | **Gửi** email                                      | Phụ trách gửi email từ client đến server hoặc chuyển tiếp giữa các mail server. Có thể nâng cấp lên truyền dữ liệu mã hóa bằng **STARTTLS**.                                                                      |
| Post Office Protocol version 3 (POP3)      | Post Office Protocol version 3     | **Nhận** email                                     | Thông thường tải email từ server **về thiết bị local rồi xóa bản sao trên server** (có thể cấu hình giữ lại). **POP3S** là version được mã hóa bằng SSL/TLS.                                                      |
| Internet Message Access Protocol (IMAP)    | Internet Message Access Protocol   | **Nhận và quản lý** email                          | Email được giữ trên server, hỗ trợ đồng bộ state email trên nhiều thiết bị, quản lý folder, tìm kiếm online... **IMAPS** là version được mã hóa bằng SSL/TLS. Là lựa chọn ưu tiên của các dịch vụ email hiện đại. |
| Remote Terminal Protocol (Telnet)          | Teletype Network                   | Đăng nhập terminal từ xa                           | Tất cả dữ liệu (bao gồm password) đều được truyền **plaintext**, độ an toàn cực thấp, về cơ bản đã bị SSH thay thế hoàn toàn.                                                                                     |
| Secure Shell Protocol (SSH)                | Secure Shell                       | Quản trị từ xa an toàn, truyền dữ liệu được mã hóa | Cung cấp đăng nhập từ xa và thực thi command qua kết nối mã hóa, cùng các chức năng như truyền file an toàn (SFTP), là lựa chọn an toàn thay thế Telnet.                                                          |

**Các protocol chạy trên UDP (nhấn mạnh truyền nhanh, overhead thấp):**

| Tên protocol (viết tắt)                    | Tên đầy đủ tiếng Anh                  | Công dụng chính                                         | Mô tả và đặc tính                                                                                                                                                                 |
| ------------------------------------------ | ------------------------------------- | ------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Hypertext Transfer Protocol (HTTP/3)       | HyperText Transfer Protocol version 3 | Truyền Web thế hệ mới                                   | Dựa trên **QUIC** protocol (bản thân QUIC được xây dựng trên UDP), nhằm giảm độ trễ và giảm head-of-line blocking của TCP; khi khôi phục session có thể sử dụng early data 0-RTT. |
| Dynamic Host Configuration Protocol (DHCP) | Dynamic Host Configuration Protocol   | Phân bổ động IP và network configuration                | Client tự động lấy IP, subnet mask, gateway, DNS server và các thông tin khác từ server.                                                                                          |
| Domain Name System (DNS)                   | Domain Name System                    | Phân giải domain name thành IP                          | **Thường sử dụng UDP** để query nhanh. Khi response packet quá lớn hoặc thực hiện zone transfer (AXFR), DNS sẽ **chuyển sang TCP** để bảo đảm tính toàn vẹn dữ liệu.              |
| Real-time Transport Protocol (RTP)         | Real-time Transport Protocol          | Truyền audio/video stream real-time                     | Thường dùng cho VoIP, video conference, livestream... Ưu tiên độ trễ thấp, cho phép mất một lượng nhỏ packet. Thường được sử dụng cùng RTCP.                                      |
| RTP Control Protocol (RTCP)                | RTP Control Protocol                  | Giám sát chất lượng và thông tin control của RTP stream | Hoạt động cùng RTP, cung cấp thông tin thống kê như packet loss, latency, jitter, hỗ trợ flow control và congestion management.                                                   |
| Trivial File Transfer Protocol (TFTP)      | Trivial File Transfer Protocol        | Truyền file đơn giản                                    | Chức năng đơn giản, thường dùng trong các trường hợp truyền file nhỏ như khởi động diskless workstation trong LAN, nâng cấp firmware cho network device.                          |
| Simple Network Management Protocol (SNMP)  | Simple Network Management Protocol    | Giám sát và quản lý network device                      | Cho phép network administrator query và thay đổi thông tin state của network device.                                                                                              |
| Network Time Protocol (NTP)                | Network Time Protocol                 | Đồng bộ clock của máy tính                              | Dùng để đồng bộ thời gian giữa các máy tính trong network, bảo đảm thời gian nhất quán.                                                                                           |

**Tóm lại:**

- **TCP** phù hợp hơn với các application có yêu cầu cao về **độ tin cậy, tính toàn vẹn và tính đúng thứ tự** của dữ liệu, chẳng hạn duyệt Web (HTTP/HTTPS), truyền file (FTP/SFTP), gửi và nhận email (SMTP/POP3/IMAP).
- **UDP** phù hợp hơn với các application yêu cầu cao về **tính real-time và có thể chịu được một lượng nhỏ dữ liệu bị mất**, chẳng hạn phân giải domain (DNS), audio/video real-time (RTP), game online, quản lý network (SNMP)...

### TCP Keepalive và HTTP Keep-Alive khác nhau thế nào?

| Khía cạnh so sánh                    | HTTP Keep-Alive                                                                                        | TCP Keepalive                                                                                                   |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------- |
| **Tầng sở hữu**                      | Tầng application (HTTP protocol)                                                                       | Tầng transport (TCP protocol)                                                                                   |
| **Vấn đề giải quyết**                | Tái sử dụng TCP connection, giảm overhead do thiết lập connection, đóng connection, slow start lặp lại | Thăm dò TCP connection idle trong thời gian dài, giải phóng resource connection sau khi đầu bên kia mất kết nối |
| **Hành vi mặc định**                 | HTTP/1.0 mặc định short connection; HTTP/1.1 mặc định long connection                                  | Mặc định tắt, application cần bật rõ ràng `SO_KEEPALIVE`                                                        |
| **Mức độ điều khiển**                | Do HTTP client, Web server hoặc proxy điều khiển theo policy connection                                | Do kernel của OS điều khiển, một số platform cũng cho phép điều chỉnh theo từng socket                          |
| **Parameter thường gặp**             | `Connection`, `Keep-Alive: timeout/max`, cấu hình timeout của server                                   | `tcp_keepalive_time/intvl/probes` hoặc parameter tương ứng của platform                                         |
| **Trigger đóng**                     | Đạt idle timeout, giới hạn số request hoặc một bên chủ động đóng                                       | Sau khi idle, gửi probe packet; chỉ đóng sau nhiều lần không nhận được response hoặc nhận RST                   |
| **Khi peer online**                  | Server vẫn có thể chủ động thu hồi idle connection theo cấu hình                                       | Chỉ cần kernel của peer có thể phản hồi ACK thì connection thường tiếp tục được duy trì                         |
| **Có thể thay heartbeat không**      | Không thể xác định business có healthy hay không, chỉ có thể quản lý việc tái sử dụng HTTP connection  | Không thể xác định thread pool, event loop hay business dependency có hoạt động bình thường hay không           |
| **Ảnh hưởng của intermediate layer** | Proxy, gateway có thể độc lập quản lý hai đoạn HTTP/TCP connection trước và sau                        | NAT/LB/reverse proxy có thể khiến thứ bạn thăm dò chỉ là một đoạn TCP connection                                |
| **Quan hệ với HTTP/2/3**             | HTTP/2 không dùng connection-level header; HTTP/3/QUIC không sử dụng cơ chế này                        | Chỉ có tác dụng với TCP; HTTP/3/QUIC connection thực sự không bị ảnh hưởng trực tiếp                            |

**Hành vi mặc định của Keep-Alive khác nhau trong các version HTTP:**

![Hành vi mặc định của Keep-Alive khác nhau trong các version HTTP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/different-http-versions-have-different-default-keep-alive-behaviors.png)

Nếu nhìn từ góc độ “ai quyết định đóng connection”, thái độ của hai cơ chế hoàn toàn trái ngược:

HTTP Keep-Alive là “chủ động thu hồi” - khi server đạt timeout hoặc giới hạn số request, server có thể đóng connection theo cấu hình của mình mà không cần thăm dò trước xem peer có online hay không. Đây là một phương thức thu hồi resource tương đối chủ động.

TCP Keepalive là “bị động thu hồi” - trước tiên phải gửi probe packet để hỏi “bạn còn ở đó không?”. Chỉ cần peer online và có thể phản hồi ACK, server chỉ có thể tiếp tục duy trì connection và refresh timer. Chỉ sau khi xác nhận peer đã không còn ở đó thì mới có thể giải phóng resource. Đây là một policy thu hồi nhẹ nhàng.

![Nguyên lý hoạt động của TCP Keepalive](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-keepalive-vs-http-keepalive-tcp-keepalive-working-principle.png)

![Cơ chế thăm dò của TCP Keepalive](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-keepalive-vs-http-keepalive-tcp-keepalive-detection-mechanism.png)

Trong project thực tế, hai cơ chế thường chạy đồng thời, mỗi cơ chế quản lý phần của mình. HTTP Keep-Alive quản lý “một connection được sử dụng tối đa bao lâu, phục vụ bao nhiêu request”; TCP Keepalive quản lý “nếu không có dữ liệu trong thời gian dài thì kiểm tra xem peer đã biến mất chưa”. Hai cơ chế không ảnh hưởng lẫn nhau và cũng không thể thay thế lẫn nhau.

Giới thiệu chi tiết: [TCP Keepalive và HTTP Keep-Alive khác nhau thế nào?](./tcp-keepalive-vs-http-keepalive.md)

### ⭐️ TCP bắt tay ba bước và đóng kết nối bốn bước (rất quan trọng)

**Các câu hỏi phỏng vấn liên quan:**

- Vì sao cần bắt tay ba bước?
- Lần bắt tay thứ 2 đã gửi ACK về rồi, vì sao vẫn phải gửi SYN?
- Vì sao cần đóng kết nối bốn bước?
- Vì sao không thể gộp ACK và FIN do server gửi thành một lần, biến thành đóng kết nối ba bước?
- Nếu ACK của server trong lần đóng kết nối thứ hai không đến được client thì chuyện gì sẽ xảy ra?
- Vì sao sau lần đóng kết nối thứ tư, client cần chờ 2\*MSL (Maximum Segment Lifetime) rồi mới chuyển sang state CLOSED?

**Đáp án tham khảo**: [TCP bắt tay ba bước và đóng kết nối bốn bước (tầng transport)](https://javaguide.cn/cs-basics/network/tcp-connection-and-disconnection.html).

### TCP TIME_WAIT rốt cuộc đang chờ điều gì? Vì sao phải chờ?

**Các câu hỏi phỏng vấn liên quan:**

1. `TIME_WAIT` rốt cuộc đang chờ điều gì?
2. `TIME_WAIT` tích tụ số lượng lớn có thực sự gây ra vấn đề không?
3. Có thể tùy tiện bật `tcp_tw_reuse` không?
4. Phân biệt `TIME_WAIT` và `CLOSE_WAIT` như thế nào?

**Đáp án tham khảo**: [Giải thích chi tiết TCP TIME_WAIT: Vì sao phải chờ, có gây ra vấn đề không, có thể tái sử dụng không?](./tcp-time-wait.md).

### ⭐️ TCP bảo đảm độ tin cậy của việc truyền như thế nào? (quan trọng)

[Bảo đảm độ tin cậy khi truyền TCP (tầng transport)](https://javaguide.cn/cs-basics/network/tcp-reliability-guarantee.html)

### TCP và UDP có thể sử dụng cùng một port không?

Kết luận: **Có thể**. Namespace binding của port TCP và UDP được phân biệt theo transport protocol, cùng một port number ở các protocol khác nhau không xung đột.

Sau khi kernel nhận IP packet, trước tiên kernel xem protocol identifier của IP layer (protocol number của TCP là `6`, của UDP là `17`), phân phối packet cho TCP hoặc UDP protocol stack tương ứng dựa trên protocol number, sau đó phân phối theo address và port trong protocol stack riêng. Vì vậy `TCP/8080` và `UDP/8080` có thể cùng tồn tại, kernel hoàn toàn không coi chúng là cùng một communication.

![Quy trình phân phối protocol của kernel](https://oss.javaguide.cn/github/javaguide/cs-basics/network/can-tcp-and-udp-use-the-same-port-kernel-protocol-dispatching-process.png)

Thứ thực sự dễ xung đột là việc binding trùng lặp trong **cùng một protocol**, chẳng hạn hai TCP service thường không thể cùng listen trên một local IP và port; khi đó mới liên quan đến các socket reuse option như `SO_REUSEADDR`, `SO_REUSEPORT`.

Ví dụ kinh điển: DNS đồng thời sử dụng `UDP/53` (truy vấn thông thường) và `TCP/53` (response quá lớn, zone transfer); HTTP/3 thường được deploy với `UDP/443` (QUIC), có thể cùng tồn tại với `TCP/443` của HTTPS truyền thống.

![Ví dụ thực tế DNS và HTTP/3 đồng thời sử dụng port TCP và UDP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/can-tcp-and-udp-use-the-same-port-practical-application-example.png)

Giới thiệu chi tiết: [TCP và UDP có thể sử dụng cùng một port không?](./can-tcp-and-udp-use-the-same-port.md)

### ⭐️ Một host chỉ duy trì được tối đa 65535 TCP connection sao?

Kết luận: **Không phải**. `65535` là port number lớn nhất, không phải giới hạn số connection.

TCP connection được phân biệt bằng four-tuple: source IP, source port, destination IP, destination port. Chỉ cần four-tuple khác nhau thì kernel sẽ nhận diện là các connection khác nhau. Khi server listen trên cùng một port, chỉ cần client IP hoặc client port khác nhau thì số connection vẫn có thể tiếp tục tăng.

![TCP connection được phân biệt bằng four-tuple và giới hạn thực sự](https://oss.javaguide.cn/github/javaguide/cs-basics/network/maximum-number-of-tcp-connections-per-host-tcp-four-tuple-and-server-connection.png)

Các yếu tố thực sự giới hạn số connection:

- **Server:** Chủ yếu bị giới hạn bởi file descriptor, memory, CPU, network card và năng lực xử lý của application, không phải số port.
- **Client:** Khi kết nối đến cùng một target, source IP và destination IP:Port đều cố định, chỉ còn source port có thể thay đổi, dễ chạm giới hạn ephemeral port hơn (Linux mặc định khoảng 28 nghìn port). Việc tích tụ `TIME_WAIT` sẽ làm vấn đề nghiêm trọng hơn.
- **NAT gateway:** Khi nhiều machine trong mạng nội bộ dùng chung một public IP để truy cập cùng một external target, public source port ở phía NAT cũng trở thành bottleneck.

Trong production, vấn đề thường gặp nhất không phải thiếu port, mà là **connection pool cấu hình không phù hợp khiến short connection liên tục được tạo và hủy**, làm cạn ephemeral port. Khi điều tra, trước tiên hãy kiểm tra connection pool và keep-alive có hoạt động hay không, đừng ngay lập tức sửa kernel parameter.

Giới thiệu chi tiết: [Một host chỉ duy trì được tối đa 65535 TCP connection sao?](./maximum-number-of-tcp-connections-per-host.md)

## IP

### IP protocol có tác dụng gì?

**IP (Internet Protocol)** là một trong những protocol quan trọng nhất trong TCP/IP protocol, thuộc network layer, chủ yếu dùng để định nghĩa format của packet, thực hiện routing và addressing cho packet, để packet có thể truyền qua các network và đến đúng destination.

Hiện tại IP protocol chủ yếu được chia thành hai loại: IPv4 và IPv6, trong đó IPv6 mới hơn. Hiện cả hai protocol đều đang được sử dụng, nhưng protocol sau đã được đề xuất để thay thế protocol trước.

### IP address là gì? IP addressing hoạt động như thế nào?

IP address thường được cấp cho network interface, dùng để nhận diện communication endpoint trong một scope và routing context cụ thể. Một interface có thể có nhiều address, address cũng có thể thay đổi động; private address có thể được tái sử dụng trong các network khác nhau, còn Anycast address cũng có thể được cấp cho nhiều interface. Ví dụ IPv4 address là `192.168.1.1`, ví dụ IPv6 address là `2001:0db8:85a3:0000:0000:8a2e:0370:7334`.

Khi network device gửi IP packet, packet chứa **source IP address** và **destination IP address**. Chúng nhận diện source interface address và destination address được sử dụng trong lần giao tiếp này, không phải identity cố định vĩnh viễn của device.

Network device dựa trên destination IP address để xác định destination của packet, rồi forward packet đến destination network hoặc subnetwork chính xác, từ đó thực hiện giao tiếp giữa các device.

Phương thức addressing dựa trên IP address này là nền tảng của giao tiếp Internet, cho phép packet truyền qua các network khác nhau. Việc address có unique và có thể được global route hay không phụ thuộc vào loại address và scope; không thể khái quát IP address là identity toàn cầu duy nhất của mỗi device.

![IP address giúp packet đến destination](https://oss.javaguide.cn/github/javaguide/cs-basics/network/internet_protocol_ip_address_diagram.png)

### IP address filtering là gì?

**IP address filtering (IP Address Filtering)** nói đơn giản là giới hạn hoặc chặn quyền truy cập của một IP address cụ thể hoặc một dải IP address. Ví dụ, nếu image service của bạn đột nhiên bị một IP address tấn công, bạn có thể cấm IP address đó truy cập image service.

IP address filtering là một biện pháp network security đơn giản. Trong ứng dụng thực tế, nó thường được sử dụng cùng các biện pháp network security khác như authentication, authorization và encryption. Chỉ sử dụng IP address filtering không thể bảo đảm hoàn toàn network security.

### ⭐️ IPv4 và IPv6 khác nhau thế nào?

**IPv4 (Internet Protocol version 4)** là version IP address đang được sử dụng rộng rãi, có format gồm bốn nhóm số được phân tách bằng dấu chấm, chẳng hạn 123.89.46.72. IPv4 sử dụng address 32 bit làm Internet address, nghĩa là có khoảng 4,2 tỷ (2^32) IP address có thể sử dụng.

![IPv4 address biểu diễn address 32 bit bằng format thập phân phân tách bằng dấu chấm](https://oss.javaguide.cn/github/javaguide/cs-basics/network/Figure-1-IPv4Addressformatwithdotteddecimalnotation-29c824f6a451d48d8c27759799f0c995.png)

Số lượng ít như vậy đương nhiên không đủ dùng! Để giải quyết vấn đề cạn kiệt IP address, cách căn bản nhất là sử dụng version IP mới có address space lớn hơn - **IPv6 (Internet Protocol version 6)**. IPv6 address sử dụng format phức tạp hơn, gồm các nhóm số và chữ cái được phân tách bằng một hoặc hai dấu hai chấm, chẳng hạn 2001:0db8:85a3:0000:0000:8a2e:0370:7334. IPv6 sử dụng Internet address 128 bit, nghĩa là có tới 2^128 (39 chữ số bắt đầu bằng 3, thật khủng khiếp) IP address có thể sử dụng.

![IPv6 address biểu diễn address 128 bit bằng format hexadecimal phân tách bằng dấu hai chấm](https://oss.javaguide.cn/github/javaguide/cs-basics/network/Figure-2-IPv6Addressformatwithhexadecimalnotation-7da3a419bd81627a9b2cef3b0efb4940.png)

Ngoài address space lớn hơn, IPv6 còn có các ưu điểm:

- **Stateless Address Autoconfiguration (viết tắt SLAAC):** Host có thể tạo IPv6 address dựa trên prefix do router quảng bá và interface identifier, không cần dựa vào DHCPv6 để phân bổ address. Trước khi sử dụng address thường thực hiện Duplicate Address Detection (DAD), nhưng DAD chỉ kiểm tra xem có trùng trong cùng link hay không, hơn nữa việc kiểm tra không hoàn toàn đáng tin cậy, không thể dựa vào đó để khẳng định address được bảo đảm “globally unique”.
- **NAT (Network Address Translation) trở thành tùy chọn:** Tài nguyên IPv6 address dồi dào, có thể cấp cho mỗi device trên toàn cầu một address độc lập.
- **Cải tiến cấu trúc header:** IPv6 basic header đơn giản hóa một phần xử lý trên routing path thường gặp, nhưng performance thực tế vẫn phụ thuộc vào hardware, extension header, network policy và implementation cụ thể, không thể chỉ dựa vào cấu trúc header để bảo đảm overall performance chắc chắn tăng.
- **Extension header tùy chọn:** Cho phép thêm các extension header khác nhau (Extension Headers) vào IPv6 header để thực hiện các loại function và option khác nhau.
- **ICMPv6 (Internet Control Message Protocol for IPv6):** ICMPv6 trong IPv6 có một số cải tiến so với ICMP trong IPv4, chẳng hạn cải tiến các function như neighbor discovery và Path MTU Discovery, từ đó nâng cao độ tin cậy và performance của network.
- ……

### Làm thế nào để lấy IP thật của client?

Có nhiều phương thức lấy IP thật của client, chủ yếu được chia thành phương thức ở application layer, transport layer và network layer.

**Phương thức ở application layer:**

`X-Forwarded-For` là request header được sử dụng rộng rãi nhưng chưa được standardize trong hệ sinh thái HTTP proxy; cơ chế tương ứng được IETF standardize là HTTP `Forwarded` header. Cả hai đều thuộc HTTP, không thể áp dụng trực tiếp cho các application layer protocol khác như SMTP. Business service cũng không thể vô điều kiện tin tưởng `X-Forwarded-For` do client truyền vào: trusted reverse proxy nên overwrite hoặc normalize giá trị truyền từ bên ngoài, server chỉ parse phần do các proxy đã biết thêm vào.

**Phương thức ở transport layer:**

Sử dụng TCP Options field để mang thông tin source IP thật. Phương thức này áp dụng cho mọi protocol dựa trên TCP, không bị giới hạn bởi application layer. Tuy nhiên, đây không phải chức năng được TCP standard hỗ trợ, do đó cần sửa đổi cả hai phía giao tiếp. Nghĩa là: phía sender cần có khả năng chèn source IP thật vào TCP Options; phía receiver cần có khả năng đọc IP address trong TCP Options.

Cũng có thể truyền client IP và Port bằng Proxy Protocol. Phương thức này có thể sử dụng Nginx hoặc reverse proxy server khác hỗ trợ protocol này để lấy IP thật hoặc parse IP thật ở business server.

**Phương thức ở network layer:**

Tunnel + DSR mode. Phương thức này có thể áp dụng cho mọi protocol, nhưng triển khai tương đối phức tạp và cũng có một số giới hạn, vì vậy trong ứng dụng thực tế thường không sử dụng.

### NAT có tác dụng gì?

**NAT (Network Address Translation)** chủ yếu dùng để chuyển đổi IP address giữa các network khác nhau. NAT cho phép map private IP address (chẳng hạn IP address được sử dụng trong LAN) thành public IP address (IP address được sử dụng trên Internet), hoặc map ngược lại, từ đó giúp nhiều device trong LAN truy cập Internet thông qua một public IP address duy nhất.

NAT không chỉ giảm bớt vấn đề thiếu tài nguyên IPv4 address, mà còn che giấu internal address và topology. Hành vi filtering của nhiều NAT device khiến external traffic không có mapping sẵn khó trực tiếp đến internal host, nhưng thứ quyết định packet inbound nào có thể đi qua là filtering policy, không phải bản thân address translation. NAT không thể thay thế stateful firewall, access control và biện pháp bảo mật trên host.

![NAT thực hiện chuyển đổi IP address](https://oss.javaguide.cn/github/javaguide/cs-basics/network/network-address-translation.png)

Đọc thêm: [Giải thích chi tiết NAT protocol (tầng network)](https://javaguide.cn/cs-basics/network/nat.html).

## ARP

### MAC address là gì?

MAC address có tên đầy đủ là **Media Access Control Address**, dùng để nhận diện interface ở link layer và truyền data frame trong local network. MAC address thuộc về network interface, không phải identity vĩnh viễn của toàn bộ device; một device có thể có nhiều network interface, mỗi interface có thể sử dụng một MAC address khác nhau.

![Mặt sau của router sẽ ghi MAC address](https://oss.javaguide.cn/github/javaguide/cs-basics/network/router-back-will-indicate-mac-address.png)

MAC address cũng thường được gọi là LAN address, physical address hoặc Ethernet address. Khác với IP address dùng để route ở network layer, MAC address chủ yếu được sử dụng trong link hiện tại hoặc broadcast domain.

> Một điểm khác cần biết là không chỉ network resource mới có IP address, network device cũng có IP address, chẳng hạn router. Nhưng xét về cấu trúc, router và các network device khác có vai trò tạo nên một network, hơn nữa thường là internal network, vì vậy IP address chúng sử dụng thường là internal IP. Khi device trong internal network giao tiếp với device bên ngoài internal network, cần sử dụng NAT protocol.

MAC address thường gặp của Ethernet là EUI-48 có 6 byte (48 bit). IEEE phân bổ các address block có kích thước khác nhau như MA-L, MA-M, MA-S, để vendor tiếp tục phân bổ globally administered address; ngoài ra còn có locally administered address, không cần IEEE phân bổ toàn cầu. OS có thể sửa đổi hoặc randomize MAC address, vì vậy address không bảo đảm bất biến vĩnh viễn, và trong các network khác nhau cũng có thể xuất hiện address giống nhau.

Cuối cùng, hãy nhớ MAC address có một address đặc biệt: FF-FF-FF-FF-FF-FF (địa chỉ gồm toàn bit 1), address này biểu thị broadcast address.

### ⭐️ ARP protocol giải quyết vấn đề gì?

ARP protocol, có tên đầy đủ là **Address Resolution Protocol**, giải quyết vấn đề chuyển đổi giữa network layer address và link layer address. Vì khi IP datagram được truyền qua mạng, luôn cần biết next hop (destination tiếp theo trên đường truyền) phải đi đến đâu; nhưng IP address là logical address, còn MAC address mới là physical address. ARP protocol giải quyết một số vấn đề trong việc chuyển IP address thành MAC address.

### Nguyên lý hoạt động của ARP protocol?

[Giải thích chi tiết ARP protocol (tầng network)](https://javaguide.cn/cs-basics/network/arp.html)

## Gợi ý ôn tập

Rất khuyến khích bạn đọc cuốn 《HTTP minh họa》. Cuốn sách này không dài nhưng nội dung rất đầy đủ; dù dùng để nắm một cách có hệ thống một số kiến thức về network hay chỉ đơn thuần để chuẩn bị cho phỏng vấn đều rất hữu ích. Một số bài viết dưới đây chỉ mang tính tham khảo. Khi học môn này vào năm hai đại học, giáo trình chúng tôi sử dụng là 《Computer Network phiên bản 7》 (do Xie Xiren biên soạn); không khuyến nghị bạn đọc giáo trình này, sách rất dày và kiến thức thiên về lý thuyết, không chắc bạn có thể bình tĩnh đọc hết.

## Tham khảo

- 《HTTP minh họa》
- 《Mạng máy tính: Phương pháp tiếp cận từ trên xuống》 (phiên bản 7)
- Internet Protocol (IP) là gì?: <https://www.cloudflare.com/zh-cn/learning/network-layer/internet-protocol/>
- Các phương pháp truyền source IP thật - Geektime: <https://time.geekbang.org/column/article/497864>
- What Is NAT and What Are the Benefits of NAT Firewalls?: <https://community.fs.com/blog/what-is-nat-and-what-are-the-benefits-of-nat-firewalls.html>

<!-- @include: @article-footer.snippet.md -->
