---
title: "Tổng hợp các giao thức tầng ứng dụng thường gặp: HTTP, WebSocket, SMTP, FTP, SSH, DNS, v.v."
description: "Tổng hợp các khái niệm cốt lõi và trường hợp sử dụng điển hình của các giao thức tầng ứng dụng thường gặp, tập trung so sánh mô hình giao tiếp và giới hạn khả năng của HTTP với WebSocket."
category: Computer Basics
tag:
  - Computer Network
head:
  - - meta
    - name: keywords
      content: giao thức tầng ứng dụng,HTTP,WebSocket,DNS,SMTP,FTP,đặc điểm,trường hợp sử dụng
---

<!-- @include: @article-header.snippet.md -->

Có rất nhiều giao thức tầng ứng dụng. Các tên HTTP, WebSocket, SMTP, POP3/IMAP, FTP, Telnet, SSH, RTP, DNS cũng thường xuất hiện cùng nhau.

Bạn không cần học đến chi tiết triển khai của từng giao thức, nhưng nếu chỉ nhớ tên giao thức thì rất dễ nhầm lẫn giữa các điểm như “mục đích sử dụng, giao thức truyền tải bên dưới, trường hợp sử dụng điển hình”.

Bài viết này chủ yếu trả lời một số câu hỏi:

1. HTTP, WebSocket, SMTP, FTP, SSH, DNS và các giao thức khác lần lượt giải quyết vấn đề gì?
2. Các giao thức này thường dựa trên TCP hay UDP, cổng thường dùng và trường hợp sử dụng là gì?
3. Những giao thức nào dễ bị nhầm lẫn nhất, cần phân biệt như thế nào trong phỏng vấn và thực tế?

## HTTP: Giao thức truyền siêu văn bản

**Giao thức truyền siêu văn bản (HTTP, HyperText Transfer Protocol)** là một giao thức tầng ứng dụng dùng để truyền siêu văn bản và nội dung đa phương tiện. Trường hợp sử dụng phổ biến nhất là giao tiếp giữa trình duyệt Web và Web server.

![Tổng quan về HTTP: Giao thức truyền siêu văn bản](https://oss.javaguide.cn/github/javaguide/cs-basics/network/http-overview.png)

Khi truy cập một trang Web trong trình duyệt, trình duyệt gửi HTTP request đến server, server xử lý rồi trả về HTTP response. HTML, CSS, JavaScript, hình ảnh, video và nhiều tài nguyên khác trên trang thường được tải qua HTTP.

HTTP sử dụng mô hình client-server. Client gửi HTTP Request (request), server trả về HTTP Response (response). Toàn bộ quá trình được minh họa như hình dưới đây.

![Giao thức HTTP](https://oss.javaguide.cn/github/javaguide/450px-HTTP-Header.png)

Cần lưu ý rằng HTTP là giao thức tầng ứng dụng, bản thân nó không trực tiếp phụ trách truyền tải đáng tin cậy. Các phiên bản HTTP khác nhau cũng không hoàn toàn dựa trên cùng một giao thức tầng dưới:

- **HTTP/1.1**: dựa trên TCP.
- **HTTP/2**: thường cũng dựa trên TCP, nhưng bổ sung khả năng multiplexing, nén header và các tính năng khác.
- **HTTP/3**: dựa trên QUIC, còn QUIC dựa trên UDP, chủ yếu nhằm giảm overhead khi thiết lập connection và giảm ảnh hưởng của head-of-line blocking do TCP gây ra.

Trong HTTP/1.1, Keep-Alive được bật mặc định, tức là connection lâu dài. Nhờ đó, cùng một connection TCP có thể được tái sử dụng cho nhiều HTTP request, tránh phải thiết lập lại connection TCP cho mỗi request, từ đó giảm chi phí của TCP three-way handshake.

Xét từ góc độ tái sử dụng connection, Keep-Alive của HTTP/1.1 giải quyết vấn đề “tái sử dụng cùng một connection TCP cho nhiều request”, nhưng việc xử lý request trên cùng connection vẫn có thể chịu ảnh hưởng của head-of-line blocking.

HTTP/2 áp dụng multiplexing trên một connection TCP, cho phép truyền song song nhiều request và response, giảm head-of-line blocking ở tầng HTTP. Tuy nhiên, vì tầng bên dưới vẫn là TCP, khi một packet TCP bị mất, dữ liệu trên toàn bộ connection vẫn bị ảnh hưởng.

HTTP/3 dựa trên QUIC. QUIC triển khai multiplexing và truyền tải đáng tin cậy trên UDP. Các stream độc lập với nhau, giúp giảm vấn đề head-of-line blocking ở tầng TCP.

Ngoài ra, HTTP là một **giao thức stateless**. Server không tự nhiên ghi nhớ “request trước đó do ai gửi và đang ở trạng thái nào”. Vì vậy, trong phát triển Web thực tế, thường cần dựa vào các cơ chế như Cookie, Session, Token (bao gồm JWT) để duy trì trạng thái đăng nhập và trạng thái session của người dùng.

## WebSocket: Giao thức giao tiếp full-duplex

**WebSocket** là một giao thức full-duplex dựa trên connection TCP. Client và server có thể đồng thời gửi và nhận dữ liệu trên cùng một connection.

![Tổng quan về WebSocket: Giao thức giao tiếp full-duplex](https://oss.javaguide.cn/github/javaguide/cs-basics/network/websocket-overview.png)

Đặc điểm điển hình của nó là: **sau khi connection được thiết lập, server cũng có thể chủ động push message đến client**. Điều này bù đắp hạn chế của mô hình request-response truyền thống của HTTP trong các trường hợp giao tiếp thời gian thực.

Giao thức WebSocket ra đời năm 2008 và trở thành tiêu chuẩn quốc tế năm 2011. Các trình duyệt phổ biến hiện nay hầu như đều hỗ trợ nó. WebSocket không chỉ dùng trong trình duyệt; nhiều ngôn ngữ lập trình, framework và server cũng cung cấp hỗ trợ tương ứng.

Về bản chất, WebSocket vẫn là một giao thức tầng ứng dụng. Nó thường bắt đầu bằng một HTTP request để yêu cầu nâng cấp giao thức. Sau khi nâng cấp thành công, client và server thiết lập một connection lâu dài, sau đó có thể truyền dữ liệu hai chiều.

![Sơ đồ WebSocket](https://oss.javaguide.cn/github/javaguide/system-design/web-real-time-message-push/1460000042192394.png)

Các trường hợp sử dụng phổ biến của WebSocket gồm:

- Bình luận trực tiếp trên video
- Push message thời gian thực, xem chi tiết tại [Giải thích chi tiết về push message thời gian thực trên Web](https://javaguide.cn/system-design/web-real-time-message-push.html)
- Game đối kháng thời gian thực
- Chỉnh sửa cộng tác nhiều người dùng
- Chăm sóc khách hàng trực tuyến / chat mạng xã hội
- Cập nhật dữ liệu thời gian thực như giá cổ phiếu, tỷ số thể thao

Quy trình hoạt động của WebSocket có thể được chia đơn giản thành các bước sau:

1. Client gửi một HTTP request đến server. Request header chứa các field như `Upgrade: websocket`, `Connection: Upgrade`, `Sec-WebSocket-Key`, biểu thị mong muốn nâng cấp connection hiện tại thành WebSocket.
2. Sau khi nhận request, nếu hỗ trợ WebSocket, server trả về status code HTTP `101 Switching Protocols`. Response header chứa các field như `Upgrade: websocket`, `Connection: Upgrade`, `Sec-WebSocket-Accept`, biểu thị nâng cấp giao thức thành công.
3. Sau khi nâng cấp giao thức, client và server thiết lập một connection WebSocket, hai bên có thể giao tiếp hai chiều.
4. Dữ liệu WebSocket được truyền dưới dạng frame (Frame). Một message hoàn chỉnh có thể được chia thành nhiều frame để gửi đi, sau đó bên nhận lắp ghép lại thành message hoàn chỉnh.
5. Client hoặc server đều có thể chủ động gửi close frame. Sau khi bên kia nhận được, nó cũng trả lời bằng close frame, rồi hai bên đóng connection TCP.

Ngoài ra, connection WebSocket thường được dùng kết hợp với **cơ chế heartbeat**. Ví dụ, định kỳ gửi Ping/Pong frame hoặc gửi gói heartbeat ở tầng nghiệp vụ để kiểm tra connection còn khả dụng hay không, tránh connection rơi vào trạng thái giả chết.

## SMTP: Giao thức truyền thư điện tử đơn giản

**Giao thức truyền thư điện tử đơn giản (SMTP, Simple Mail Transfer Protocol)** là một giao thức tầng ứng dụng dựa trên TCP, chủ yếu dùng để **gửi và chuyển tiếp email**.

![Tổng quan về SMTP: Giao thức truyền thư điện tử đơn giản](https://oss.javaguide.cn/github/javaguide/cs-basics/network/smtp-overview.png)

Cần lưu ý một điểm dễ nhầm lẫn:

**SMTP phụ trách gửi email và chuyển tiếp giữa các mail server; POP3/IMAP phụ trách việc người dùng nhận email từ mail server.**

Nói cách khác, khi email được gửi từ mail server của bạn đến mail server bên nhận, quá trình này thường vẫn dùng SMTP. Còn khi người dùng dùng client để xem email trong hộp thư, thường sử dụng POP3 hoặc IMAP.

![Giao thức SMTP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/what-is-smtp.png)

Các cổng thường liên quan đến SMTP là 25, 465, 587; mục đích của ba cổng này không hoàn toàn giống nhau:

| Cổng | Mục đích thường gặp                    | Mô tả                                                                                                               |
| ---- | -------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| 25   | Chuyển tiếp email giữa các mail server | Chủ yếu dùng để chuyển thư từ MTA đến MTA. Nhiều cloud provider hoặc ISP hạn chế outbound trên cổng 25 để ngăn spam |
| 587  | Client gửi email                       | Cổng Message Submission tiêu chuẩn, thường dùng kết hợp với STARTTLS và authentication                              |
| 465  | Gửi email với TLS ngầm định            | Khi client kết nối, trực tiếp thiết lập kênh mã hóa TLS; nhiều nhà cung cấp dịch vụ email vẫn hỗ trợ                |

### Quy trình gửi email

Ví dụ địa chỉ email của tôi là `<dabai@cszhinan.com>` và tôi muốn gửi email đến `<xiaoma@qq.com>`. Toàn bộ quá trình có thể hiểu đơn giản như sau:

1. Tôi soạn email bằng email client hoặc webmail.
2. Email client dùng giao thức SMTP để submit email đến mail server tương ứng với `cszhinan.com`.
3. Mail server bên gửi tra cứu mail server tương ứng với domain người nhận `qq.com`.
4. Mail server bên gửi tiếp tục dùng SMTP để chuyển email đến mail server QQ.
5. Mail server QQ nhận và lưu email.
6. Người dùng `<xiaoma@qq.com>` đọc email từ mail server QQ qua giao thức POP3 hoặc IMAP.

### Làm thế nào để xác định hộp thư có thực sự tồn tại?

Trong một số trường hợp, chúng ta có thể cần xác định một địa chỉ email có thực sự tồn tại hay không. Cách thường dùng là thăm dò dựa trên SMTP:

1. Tra cứu bản ghi MX tương ứng với domain email để tìm mail server.
2. Thử kết nối đến mail server đích.
3. Dùng lệnh SMTP để mô phỏng quy trình gửi thư.
4. Dựa vào kết quả server trả về để phán đoán địa chỉ email có thể tồn tại hay không.

Tuy nhiên, cách này không phải lúc nào cũng đáng tin cậy.

Để ngăn spam, credential stuffing và rò rỉ thông tin riêng tư, nhiều nhà cung cấp dịch vụ email sẽ chặn việc thăm dò sự tồn tại của email hoặc luôn trả về kết quả không rõ ràng. Vì vậy, thăm dò SMTP chỉ có thể dùng để tham khảo, không thể xác định 100% một email chắc chắn tồn tại hay không tồn tại.

Một số công cụ trực tuyến kiểm tra tính hợp lệ của email:

1. <https://verify-email.org/>
2. <http://tool.chacuo.net/mailverify>
3. <https://www.emailcamel.com/>

## POP3/IMAP: Giao thức nhận email

**POP3 và IMAP đều là các giao thức dùng để nhận email**, cả hai đều là giao thức tầng ứng dụng dựa trên TCP.

![Tổng quan về POP3/IMAP: Giao thức nhận email](https://oss.javaguide.cn/github/javaguide/cs-basics/network/pop3-imap-overview.png)

Cần lưu ý: **SMTP chủ yếu phụ trách gửi và chuyển tiếp email, POP3/IMAP chủ yếu phụ trách việc người dùng đọc email từ mail server.**

POP3 có thiết kế khá đơn giản. Cách dùng phổ biến là tải email từ server xuống local. Nó phù hợp với việc nhận thư trên một thiết bị, nhưng trải nghiệm đồng bộ nhiều thiết bị khá kém.

IMAP là giao thức nhận email hiện đại và phổ biến hơn. Nó hỗ trợ quản lý email ở phía server, có thể đồng bộ trạng thái email như đã đọc, chưa đọc, đã xóa, đã lưu trữ, phân loại thư mục và các trạng thái khác. Vì vậy, nếu đồng thời xem cùng một hộp thư trên điện thoại, máy tính và Web, trải nghiệm với IMAP thường tốt hơn.

So sánh đơn giản:

| Giao thức | Mục đích chính           | Đặc điểm                                                      |
| --------- | ------------------------ | ------------------------------------------------------------- |
| POP3      | Nhận email               | Thiên về tải xuống local, khả năng đồng bộ nhiều thiết bị yếu |
| IMAP      | Nhận và quản lý email    | Hỗ trợ đồng bộ nhiều thiết bị, tìm kiếm, đánh dấu, lưu trữ    |
| SMTP      | Gửi và chuyển tiếp email | Phụ trách tuyến chuyển thư                                    |

## FTP: Giao thức truyền file

**FTP (File Transfer Protocol, giao thức truyền file)** là một giao thức tầng ứng dụng dựa trên TCP, dùng để truyền file giữa client và server.

![Tổng quan về FTP: Giao thức truyền file](https://oss.javaguide.cn/github/javaguide/cs-basics/network/ftp-overview.png)

FTP sử dụng mô hình client-server. Một điểm khá đặc biệt là FTP thường thiết lập hai connection TCP.

> FTP khác với nhiều giao thức tầng ứng dụng khác: nó sử dụng hai connection giữa client và server:
>
> 1. **Control connection**: dùng để truyền lệnh và response, chẳng hạn đăng nhập, chuyển directory, xóa file.
> 2. **Data connection**: dùng để thực sự truyền nội dung file hoặc danh sách directory.

Thiết kế tách riêng lệnh và dữ liệu để truyền giúp lệnh điều khiển và dữ liệu file không ảnh hưởng lẫn nhau.

![Quy trình hoạt động của FTP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/ftp.png)

FTP có hai cách thiết lập data connection là active mode (PORT) và passive mode (PASV):

- **Active mode**: client thông báo cho server qua control connection về cổng mà client đang listen. Sau đó server chủ động kết nối đến cổng này của client để thiết lập data connection. Vì server phải chủ động kết nối đến client, nếu client ở sau NAT hoặc firewall thì connection rất dễ thất bại.
- **Passive mode**: client yêu cầu server mở một data port, sau đó client chủ động kết nối đến data port của server. Vì hướng kết nối vẫn là từ client đến server, cách này dễ đi qua NAT và firewall hơn, nên passive mode được dùng phổ biến hơn trong môi trường production.

Lưu ý: bản thân FTP không an toàn. Mặc định, nó không mã hóa nội dung truyền đi; username, password và dữ liệu file đều có thể bị nghe lén hoặc sửa đổi.

Vì vậy, không nên dùng FTP thông thường khi truyền file nhạy cảm. Có thể chọn:

- **SFTP**: giao thức truyền file an toàn dựa trên SSH.
- **FTPS**: bổ sung mã hóa TLS/SSL trên nền FTP.

SFTP và FTPS có tên tương tự nhưng không phải cùng một giao thức. SFTP dựa trên SSH, còn FTPS là FTP over TLS.

## Telnet: Giao thức đăng nhập từ xa

**Telnet** là một giao thức đăng nhập từ xa dựa trên TCP, cổng mặc định là 23. Nó cho phép người dùng đăng nhập từ xa vào server qua terminal và thực thi lệnh trên máy từ xa.

Vấn đề lớn nhất của Telnet là: **truyền dữ liệu dạng plaintext**.

![Tổng quan về Telnet: Giao thức đăng nhập từ xa](https://oss.javaguide.cn/github/javaguide/cs-basics/network/telnet-overview.png)

Username, password, nội dung lệnh và kết quả trả về đều không được mã hóa. Nếu kẻ tấn công có thể theo dõi traffic mạng, họ có thể nhìn thấy trực tiếp thông tin nhạy cảm.

![Giao thức đăng nhập từ xa Telnet](https://oss.javaguide.cn/github/javaguide/cs-basics/network/Telnet_is_vulnerable_to_eavesdropping-2.png)

Vì vậy, Telnet hiện nay hiếm khi được dùng để quản trị từ xa. Trong môi trường production, SSH thường được dùng để thay thế Telnet.

## SSH: Giao thức mạng an toàn

**SSH (Secure Shell)** là một giao thức mạng an toàn dựa trên TCP, cổng mặc định là 22. Nó cung cấp bảo mật cho đăng nhập từ xa, thực thi lệnh và truyền file thông qua cơ chế mã hóa và authentication.

![Tổng quan về SSH: Giao thức mạng an toàn](https://oss.javaguide.cn/github/javaguide/cs-basics/network/ssh-overview.png)

Công dụng kinh điển nhất của SSH là đăng nhập vào server từ xa:

```bash
ssh user@server_ip
```

Ngoài đăng nhập từ xa, SSH còn hỗ trợ:

- Thực thi lệnh từ xa
- Port forwarding
- Tunnel proxy
- X11 forwarding
- Truyền file an toàn dựa trên SFTP hoặc SCP

SSH sử dụng mô hình client-server. SSH Server lắng nghe request kết nối từ client, SSH Client khởi tạo connection. Hai bên trước hết thương lượng thuật toán mã hóa, sau đó tạo symmetric encryption key dùng cho giao tiếp tiếp theo thông qua key exchange. Nội dung giao tiếp sau đó đều được truyền dưới dạng mã hóa.

![SSH: Giao thức mạng an toàn](https://oss.javaguide.cn/github/javaguide/cs-basics/network/ssh-client-server.png)

Cần lưu ý rằng tính an toàn của SSH không chỉ đến từ truyền tải được mã hóa mà còn đến từ cơ chế authentication. Các phương thức authentication phổ biến gồm:

- Password authentication
- Public key authentication
- Multi-factor authentication

Trong môi trường production, nên ưu tiên public key authentication và tắt đăng nhập bằng password yếu.

## RTP: Giao thức truyền tải thời gian thực

**RTP (Real-time Transport Protocol, giao thức truyền tải thời gian thực)** là một giao thức dùng để truyền dữ liệu thời gian thực như audio và video. Nó thường chạy trên UDP. Trong mô hình phân tầng TCP/IP, tầng phía trên UDP là tầng ứng dụng, nên theo quy tắc phân tầng, RTP được xếp vào tầng ứng dụng. Tuy nhiên, trách nhiệm của nó (sequence number, timestamp, synchronization, quality feedback) gần với chức năng của tầng truyền tải hơn. RFC 3550 cũng nói rằng nó “thường được tích hợp vào xử lý ứng dụng thay vì được triển khai như một tầng độc lập”.

![Tổng quan về RTP: Giao thức truyền tải thời gian thực](https://oss.javaguide.cn/github/javaguide/cs-basics/network/rtp-overview.png)

RTP chủ yếu được dùng trong các trường hợp thời gian thực như cuộc gọi thoại, video conference và livestream. Bản thân nó không đảm bảo truyền tải đáng tin cậy, cũng không đảm bảo dữ liệu đến đúng hạn, mà dùng sequence number, timestamp và các thông tin khác để giúp bên nhận sắp xếp, đồng bộ và điều khiển phát. Mặc dù cũng có cách đóng gói RTP over TCP (như RFC 4571), cách này chủ yếu dùng trong các trường hợp đặc biệt như đi qua firewall hoặc tương thích với protocol stack cụ thể. Trong các trường hợp audio/video thời gian thực thực tế, RTP vẫn chủ yếu dùng UDP.

RTP thường được dùng cùng RTCP:

- **RTP**: phụ trách truyền dữ liệu audio/video thời gian thực.
- **RTCP (RTP Control Protocol)**: phụ trách truyền thông tin điều khiển và thống kê, chẳng hạn tỷ lệ mất packet, latency, jitter.

Trong WebRTC, RTP/RTCP là nền tảng quan trọng của truyền audio/video thời gian thực. WebRTC còn kết hợp mã hóa SRTP, congestion control, jitter buffer, NACK, FEC và các cơ chế khác để nâng cao tính an toàn và chất lượng giao tiếp thời gian thực.

Cần lưu ý rằng bản thân RTP không phụ trách resource reservation và không đảm bảo chất lượng truyền tải thời gian thực. Nó cung cấp nền tảng cho việc truyền media thời gian thực; việc kiểm soát chất lượng cụ thể cần dựa vào sự phối hợp của các cơ chế tầng trên.

## DNS: Hệ thống tên miền

**DNS (Domain Name System, hệ thống tên miền)** dùng để giải quyết vấn đề ánh xạ giữa domain name và địa chỉ IP.

![Tổng quan về DNS: Hệ thống tên miền](https://oss.javaguide.cn/github/javaguide/cs-basics/network/dns-overview.png)

Khi truy cập website, thông thường chúng ta nhập domain name, ví dụ:

```text
www.javaguide.cn
```

Nhưng giao tiếp mạng thực tế cần địa chỉ IP. Tác dụng của DNS là phân giải domain name thành địa chỉ IP tương ứng.

DNS thường sử dụng UDP, cổng mặc định là 53. UDP được ưu tiên vì phần lớn DNS query và response khá nhỏ, không cần TCP three-way handshake nên response nhanh hơn.

Trong đặc tả DNS ban đầu, kích thước DNS message qua UDP bị giới hạn ở 512 byte (không bao gồm IP header và UDP header). Nếu response quá lớn, server đặt cờ truncated, sau đó client thử lại qua TCP.

Sau này, EDNS0 mở rộng giới hạn kích thước message của DNS over UDP, giúp DNS có thể chứa response lớn hơn, chẳng hạn dữ liệu liên quan đến DNSSEC. Tuy nhiên, nếu response vượt quá kích thước UDP đã thương lượng hoặc xảy ra zone transfer (các DNS server đồng bộ dữ liệu của toàn bộ zone, việc phân giải domain thông thường hầu như không kích hoạt trường hợp này), TCP vẫn được sử dụng.

Trong mạng hiện đại còn xuất hiện một số phương án DNS an toàn hơn, chẳng hạn:

- **DoH (DNS over HTTPS)**
- **DoT (DNS over TLS)**

Mục đích của chúng đều là giảm các vấn đề về quyền riêng tư và an toàn do DNS query dạng plaintext gây ra.

## Tổng hợp cổng của các giao thức tầng ứng dụng thường gặp

| Giao thức |                                Cổng mặc định | Giao thức tầng truyền tải | Mục đích chính                         |
| --------- | -------------------------------------------: | ------------------------- | -------------------------------------- |
| HTTP      |                                           80 | TCP                       | Truy cập trang Web                     |
| HTTPS     |                                          443 | TCP / QUIC                | Truy cập Web được mã hóa               |
| WebSocket |                                     80 / 443 | TCP                       | Giao tiếp thời gian thực hai chiều     |
| SMTP      |                               25 / 465 / 587 | TCP                       | Gửi và chuyển tiếp email               |
| POP3      |                                    110 / 995 | TCP                       | Nhận email                             |
| IMAP      |                                    143 / 993 | TCP                       | Nhận và đồng bộ email                  |
| FTP       |                                      20 / 21 | TCP                       | Truyền file                            |
| SSH       |                                           22 | TCP                       | Đăng nhập từ xa và truyền file an toàn |
| Telnet    |                                           23 | TCP                       | Đăng nhập từ xa dạng plaintext         |
| DNS       |                                           53 | UDP / TCP                 | Phân giải domain name                  |
| RTP       | Cổng động (số chẵn), RTCP dùng số lẻ liền kề | Chủ yếu UDP               | Truyền audio/video thời gian thực      |

HTTPS được ghi là TCP / QUIC vì HTTPS truyền thống thường dựa trên TLS over TCP, còn trong trường hợp HTTP/3 thì dựa trên QUIC.

## Tóm tắt

Bài viết này chỉ tổng hợp nhanh các giao thức tầng ứng dụng thường gặp, không đi sâu vào các packet của giao thức và chi tiết triển khai cụ thể.

Khi ôn tập, có thể tập trung ghi nhớ một số điểm dễ nhầm lẫn:

- HTTP là giao thức tầng ứng dụng; HTTP/1.1 và HTTP/2 thường dựa trên TCP, HTTP/3 dựa trên QUIC.
- HTTP/1.1 tái sử dụng connection TCP thông qua Keep-Alive, HTTP/2 thực hiện multiplexing trên một connection TCP, HTTP/3 dựa trên QUIC để giảm head-of-line blocking của TCP.
- WebSocket thiết lập connection thông qua nâng cấp HTTP, sau đó hỗ trợ giao tiếp hai chiều.
- SMTP phụ trách gửi email và chuyển tiếp giữa các mail server, POP3/IMAP phụ trách việc người dùng nhận email.
- Các cổng thường dùng của SMTP gồm 25, 587, 465, lần lượt tương ứng với các trường hợp như chuyển tiếp giữa mail server, client submit và submit với TLS ngầm định.
- FTP có active mode và passive mode; passive mode phổ biến hơn trong môi trường production.
- FTP, SFTP, FTPS không phải cùng một thứ: FTP truyền plaintext, SFTP dựa trên SSH, FTPS dựa trên TLS.
- Telnet truyền plaintext, không phù hợp để quản trị từ xa trong môi trường production; SSH được dùng phổ biến hơn trong thực tế.
- DNS thường dựa trên UDP, nhưng cũng dùng TCP khi response quá lớn, bị truncated hoặc thực hiện zone transfer.
- RTP chạy trên UDP, được xếp vào tầng ứng dụng theo quy tắc phân tầng nhưng có trách nhiệm gần với tầng truyền tải hơn; RTP dùng cổng chẵn, còn RTCP đi kèm dùng cổng lẻ liền kề.

## Tài liệu tham khảo

- _Computer Networks: A Top-Down Approach_ (ấn bản thứ bảy)
- Giới thiệu giao thức RTP: <https://mthli.xyz/rtp-introduction/>
- RFC 6455: The WebSocket Protocol
- RFC 9110: HTTP Semantics
- RFC 8446: TLS 1.3
- RFC 9000: QUIC
- RFC 3550: RTP: A Transport Protocol for Real-Time Applications
- RFC 4571: Framing Real-time Transport Protocol (RTP) and RTP Control Protocol (RTCP) Packets over Connection-Oriented Transport
- RFC 6891: Extension Mechanisms for DNS (EDNS(0))

<!-- @include: @article-footer.snippet.md -->
