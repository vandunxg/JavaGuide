---
title: Tổng hợp câu hỏi phỏng vấn Computer Networks thường gặp (Phần 1)
description: Tổng hợp các câu hỏi phỏng vấn Computer Networks thường gặp (Phần 1), bao quát network layer, các phiên bản HTTP, HTTPS, DNS, WebSocket, SSE và PING.
category: Computer Basics
tag:
  - Computer Networks
head:
  - - meta
    - name: keywords
      content: Computer Networks interview questions,TCP/IP four-layer model,HTTP interview,HTTPS vs HTTP,HTTP/1.1 vs HTTP/2,HTTP/3 QUIC,TCP three-way handshake,UDP differences,DNS resolution,WebSocket vs SSE,GET vs POST,application-layer protocols,network layering,head-of-line blocking,PING command,ARP protocol
---

<!-- markdownlint-disable MD033 -->

Computer Networks là chủ đề thường xuyên xuất hiện trong phỏng vấn backend và phỏng vấn tuyển dụng sinh viên mới tốt nghiệp. Đặc biệt, các vấn đề như **network layering của TCP/IP, HTTP, HTTPS, DNS, WebSocket và TCP three-way handshake** gần như xuyên suốt những tình huống phát triển thực tế như “từ lúc nhập URL đến khi trang được hiển thị”, “vì sao API chậm” và “vì sao kết nối thất bại”.

Bài viết "Tổng hợp câu hỏi phỏng vấn Computer Networks thường gặp (Phần 1)" bắt đầu từ network layering model, sau đó hệ thống hóa các kiến thức cốt lõi liên quan đến application layer và HTTP. Bài phù hợp để ôn tập có hệ thống kiến thức cơ bản về Computer Networks, đồng thời cũng có thể dùng làm checklist tra cứu nhanh trước khi phỏng vấn Java backend, backend development và Computer Basics.

## Kiến thức cơ bản về Computer Networks

### Network layer model

#### Mô hình OSI 7 layer là gì? Vai trò của từng layer là gì?

**OSI 7-layer model** là một network layering model do International Organization for Standardization đề xuất. Cấu trúc tổng thể và chức năng của từng layer được thể hiện trong hình sau:

![Phân chia chức năng của từng layer trong OSI 7-layer model](https://oss.javaguide.cn/github/javaguide/cs-basics/network/osi-7-model.png)

Mỗi layer tập trung làm một việc, đồng thời đều cần sử dụng chức năng do layer bên dưới cung cấp. Ví dụ, transport layer cần sử dụng chức năng routing và addressing do network layer cung cấp thì mới biết phải truyền dữ liệu đến đâu.

**Kiến trúc 7 layer của OSI có khái niệm rõ ràng và lý thuyết hoàn chỉnh, nhưng khá phức tạp, thiếu tính thực tiễn, hơn nữa một số chức năng bị lặp lại ở nhiều layer.**

Hình trên có thể hơi trừu tượng, hãy xem thêm một hình ảnh sinh động hơn. Tôi thấy hình này trên một website nước ngoài, rất ấn tượng!

![OSI 7-layer model 2](https://oss.javaguide.cn/github/javaguide/osi七层模型2.png)

#### ⭐️ Mô hình TCP/IP 4 layer là gì? Vai trò của từng layer là gì?

**TCP/IP 4-layer model** là một model hiện đang được sử dụng rộng rãi. Có thể xem TCP/IP model là phiên bản rút gọn của OSI 7-layer model, gồm 4 layer sau:

1. Application layer
2. Transport layer
3. Network layer
4. Network interface layer

Cần lưu ý rằng không thể ghép nối hoàn toàn chính xác TCP/IP 4-layer model với OSI 7-layer model, nhưng có thể đối chiếu đơn giản như hình dưới đây:

![Quan hệ đối chiếu giữa TCP/IP 4-layer model và OSI model](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-ip-4-model.png)

Để xem giới thiệu chi tiết về vai trò của từng layer, hãy đọc bài [Giải thích chi tiết OSI và TCP/IP network layering model (Cơ bản)](https://javaguide.cn/cs-basics/network/osi-and-tcp-ip-model.html).

#### Vì sao network cần được chia layer?

Nói về layering, trước tiên hãy xét một chương trình backend được phát triển bằng framework mà chúng ta thường sử dụng. Thông thường, theo nguyên tắc mỗi layer làm một việc khác nhau, chúng ta chia hệ thống thành ba layer (hệ thống phức tạp có thể có nhiều layer hơn):

1. Repository (thao tác database)
2. Service (thao tác nghiệp vụ)
3. Controller (trao đổi dữ liệu frontend và backend)

**Hệ thống phức tạp cần được chia layer vì mỗi layer cần tập trung vào một nhóm công việc. Lý do network được chia layer cũng tương tự: mỗi layer chỉ tập trung làm một nhóm công việc.**

Quay lại câu hỏi: “Vì sao network cần được chia layer?”. Theo tôi, chủ yếu có 3 lý do:

1. **Các layer độc lập với nhau**: Các layer độc lập với nhau, không cần quan tâm layer khác được triển khai như thế nào, chỉ cần biết cách gọi các chức năng mà layer bên dưới cung cấp (có thể hiểu đơn giản là gọi interface). **Điều này cũng giống nguyên tắc chia layer hệ thống khi phát triển.**
2. **Tăng tính linh hoạt và khả năng thay thế**: Mỗi layer có thể sử dụng công nghệ phù hợp nhất để triển khai, chỉ cần đảm bảo chức năng cung cấp và quy tắc của interface được expose không thay đổi. Đồng thời, mỗi layer có thể được sửa đổi hoặc thay thế theo nhu cầu mà không ảnh hưởng đến cấu trúc của toàn bộ network. **Điều này tương ứng với nguyên tắc high cohesion và low coupling thường được yêu cầu khi phát triển hệ thống.**
3. **Chia vấn đề lớn thành vấn đề nhỏ**: Layering có thể phân rã các vấn đề network phức tạp thành nhiều vấn đề nhỏ hơn, có ranh giới rõ ràng và đơn giản để xử lý. Nhờ đó, hệ thống Computer Networks phức tạp trở nên dễ thiết kế, triển khai và chuẩn hóa hơn. **Điều này tương ứng với việc khi phát triển, chúng ta thường phân rã chức năng hệ thống, rồi phân rã vấn đề phức tạp thành các vấn đề nhỏ hơn, dễ hiểu hơn; các vấn đề nhỏ này có boundary (mục tiêu và interface) được định nghĩa rõ hơn.**

Tôi nhớ đến một câu nói rất nổi tiếng trong thế giới máy tính và muốn chia sẻ ở đây:

> Mọi vấn đề trong lĩnh vực computer science đều có thể được giải quyết bằng cách thêm một intermediate layer; toàn bộ hệ thống máy tính được thiết kế theo cấu trúc layer nghiêm ngặt từ trên xuống dưới.

### Các network protocol thường gặp

#### Các protocol thường gặp ở application layer là gì?

![Các protocol thường gặp ở application layer](https://oss.javaguide.cn/github/javaguide/cs-basics/network/application-layer-protocol.png)

- **HTTP (Hypertext Transfer Protocol)**: Là một application-layer protocol dùng để truyền hypertext và nội dung multimedia, chủ yếu được thiết kế cho giao tiếp giữa Web client và server. HTTP/1.x và HTTP/2 thường dựa trên TCP, còn HTTP/3 chạy trên QUIC dựa trên UDP.
- **SMTP (Simple Mail Transfer Protocol)**: Dựa trên TCP, là protocol dùng để gửi email. Lưu ý ⚠️: SMTP chỉ phụ trách gửi mail, không phải nhận mail. Muốn nhận mail từ mail server, cần sử dụng POP3 hoặc IMAP.
- **POP3/IMAP (mail receiving protocols)**: Dựa trên TCP, cả hai đều phụ trách nhận mail. IMAP mới hơn POP3, có chức năng và performance mạnh hơn. IMAP hỗ trợ các chức năng nâng cao như tìm kiếm, đánh dấu, phân loại và archive mail, đồng thời có thể đồng bộ trạng thái mail trên nhiều thiết bị. Gần như mọi email client và server hiện đại đều hỗ trợ IMAP.
- **FTP (File Transfer Protocol)**: Dựa trên TCP, là protocol dùng để truyền file giữa các máy tính và che giấu sự khác biệt về operating system cũng như cách lưu trữ file. Lưu ý ⚠️: FTP là protocol không an toàn vì không mã hóa dữ liệu trong quá trình truyền. Khi truyền dữ liệu nhạy cảm, nên sử dụng protocol an toàn hơn như SFTP.
- **Telnet (remote login protocol)**: Dựa trên TCP, dùng để login vào server khác thông qua một terminal. Một trong những nhược điểm lớn nhất của Telnet là mọi dữ liệu, bao gồm username và password, đều được gửi dưới dạng plaintext, tiềm ẩn rủi ro bảo mật. Đây là lý do chính khiến Telnet ngày nay ít được sử dụng, thay vào đó là một network protocol rất an toàn có tên SSH.
- **SSH (Secure Shell Protocol)**: Dựa trên TCP, thực hiện các hoạt động như truy cập và truyền file an toàn thông qua cơ chế encryption và authentication.
- **RTP (Real-time Transport Protocol)**: Thường dựa trên UDP nhưng cũng hỗ trợ TCP. RTP cung cấp chức năng truyền dữ liệu real-time end-to-end, nhưng không bao gồm resource reservation và không đảm bảo chất lượng truyền real-time; các chức năng này do WebRTC thực hiện.
- **DNS (Domain Name System)**: Thường dựa trên UDP (port 53), dùng để giải quyết vấn đề mapping giữa domain name và IP address. Khi response quá lớn hoặc thực hiện zone transfer, DNS sẽ chuyển sang TCP.

Để xem giới thiệu chi tiết về các protocol này, hãy đọc bài [Tổng hợp các application-layer protocol thường gặp (Application layer)](./application-layer-protocol.md).

#### Các protocol thường gặp ở transport layer là gì?

![Các protocol thường gặp ở transport layer](https://oss.javaguide.cn/github/javaguide/cs-basics/network/transport-layer-protocol.png)

- **TCP (Transmission Control Protocol)**: Cung cấp dịch vụ truyền dữ liệu **connection-oriented**, **reliable**.
- **UDP (User Datagram Protocol)**: Cung cấp dịch vụ truyền dữ liệu **connectionless**, **best-effort** (không đảm bảo độ tin cậy của việc truyền dữ liệu), đơn giản và hiệu quả.

#### Các protocol thường gặp ở network layer là gì?

![Các protocol thường gặp ở network layer](./images/network-model/nerwork-layer-protocol.png)

- **IP (Internet Protocol)**: Một trong các protocol quan trọng nhất trong TCP/IP, thuộc network layer, chủ yếu dùng để định nghĩa format của packet, routing và addressing packet để packet có thể truyền qua network và đến đúng destination. Hiện nay IP chủ yếu gồm hai loại: IPv4 trước đây và IPv6 mới hơn. Cả hai đều đang được sử dụng, nhưng IPv6 đã được đề xuất để thay thế IPv4.
- **ARP (Address Resolution Protocol)**: ARP giải quyết vấn đề chuyển đổi giữa network-layer address và link-layer address. Vì trong quá trình truyền vật lý, một IP datagram luôn cần biết next hop (destination tiếp theo về mặt vật lý) phải đi đâu, nhưng IP address là logical address còn MAC address mới là physical address, nên ARP giải quyết một số vấn đề khi chuyển IP address thành MAC address.
- **ICMP (Internet Control Message Protocol)**: Protocol dùng để truyền network state và error message, thường được dùng cho network diagnosis và troubleshooting. Ví dụ, công cụ Ping sử dụng ICMP để kiểm tra network connectivity.
- **NAT (Network Address Translation)**: Scenario sử dụng NAT đúng như tên gọi của nó là network address translation, được áp dụng trong quá trình chuyển đổi address từ internal network sang external network. Cụ thể, trong một subnet nhỏ (LAN), các host sử dụng IP address thuộc cùng một LAN; nhưng bên ngoài LAN đó, trong wide area network (WAN), cần một IP address thống nhất để định danh vị trí của LAN trên toàn Internet.
- **OSPF (Open Shortest Path First)**: Một internal gateway protocol (Interior Gateway Protocol, IGP), đồng thời là dynamic routing protocol được sử dụng rộng rãi. OSPF dựa trên link-state algorithm, cân nhắc các yếu tố như bandwidth và latency để chọn path tốt nhất.
- **RIP (Routing Information Protocol)**: Một internal gateway protocol (Interior Gateway Protocol, IGP), đồng thời là dynamic routing protocol, dựa trên distance-vector algorithm, sử dụng số hop cố định làm metric và chọn path có ít hop nhất làm path tốt nhất.
- **BGP (Border Gateway Protocol)**: Routing protocol dùng để trao đổi network-layer reachability information (Network Layer Reachability Information, NLRI) giữa các routing domain, có tính linh hoạt và khả năng mở rộng cao.

## HTTP

### ⭐️ Từ lúc nhập URL đến khi trang được hiển thị đã xảy ra những gì? (Rất quan trọng)

> Câu hỏi tương tự: Khi mở một trang web, toàn bộ quá trình sẽ sử dụng những protocol nào?

Hãy xem một hình trước (nguồn từ "Minh họa HTTP"):

<img src="https://oss.javaguide.cn/github/javaguide/url%E8%BE%93%E5%85%A5%E5%88%B0%E5%B1%95%E7%A4%BA%E5%87%BA%E6%9D%A5%E7%9A%84%E8%BF%87%E7%A8%8B.jpg" alt="Toàn bộ quy trình từ lúc nhập URL đến khi trang được hiển thị" style="zoom:50%" />

Cần lưu ý một lỗi trong hình trên: phải là OSPF chứ không phải OPSF. OSPF (Open Shortest Path First, ospf) là open shortest path first protocol, một routing protocol do Internet Engineering Task Force phát triển.

Nhìn chung, quá trình gồm các bước sau:

1. Nhập URL của trang web cần truy cập vào browser.
2. Browser lấy IP address tương ứng với domain name thông qua DNS protocol.
3. Dựa trên IP address và port number, browser gửi request thiết lập TCP connection đến target server.
4. Browser gửi HTTP request message đến server trên TCP connection để yêu cầu nội dung trang web.
5. Sau khi nhận HTTP request message, server xử lý request và trả HTTP response message về browser.
6. Sau khi nhận HTTP response message, browser parse HTML code trong response body, render structure và style của trang; đồng thời dựa trên URL của các resource khác trong HTML (như image, CSS, JS...) để gửi HTTP request tiếp theo lấy nội dung các resource này cho đến khi trang được load và hiển thị hoàn chỉnh.
7. Khi không cần giao tiếp với server, browser có thể chủ động đóng TCP connection hoặc chờ server đóng connection.

Có thể xem phần giới thiệu chi tiết trong bài [Toàn bộ quá trình truy cập trang web (Liên kết kiến thức)](https://javaguide.cn/cs-basics/network/the-whole-process-of-accessing-web-pages.html) (rất nên đọc).

### ⭐️ HTTP status code gồm những loại nào?

HTTP status code dùng để mô tả kết quả của HTTP request, ví dụ 2xx có nghĩa request đã được xử lý thành công.

![HTTP status code thường gặp](https://oss.javaguide.cn/github/javaguide/cs-basics/network/http-status-code.png)

Để xem phần tổng hợp chi tiết hơn về HTTP status code, hãy đọc bài [Tổng hợp HTTP status code thường gặp (Application layer)](https://javaguide.cn/cs-basics/network/http-status-codes.html).

### Các field thường gặp trong HTTP Header là gì?

| Tên field trong request header | Mô tả                                                                                                                                                                                                                                        | Ví dụ                                                                            |
| :----------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------- |
| Accept                         | Loại nội dung response có thể chấp nhận (Content-Types).                                                                                                                                                                                     | Accept: text/plain                                                               |
| Accept-Charset                 | Character set được chấp nhận                                                                                                                                                                                                                 | Accept-Charset: utf-8                                                            |
| Accept-Datetime                | Version được biểu diễn theo thời gian mà client có thể chấp nhận                                                                                                                                                                             | Accept-Datetime: Thu, 31 May 2007 20:35:00 GMT                                   |
| Accept-Encoding                | Danh sách encoding có thể chấp nhận. Tham khảo HTTP compression.                                                                                                                                                                             | Accept-Encoding: gzip, deflate                                                   |
| Accept-Language                | Danh sách ngôn ngữ tự nhiên của nội dung response có thể chấp nhận.                                                                                                                                                                          | Accept-Language: en-US                                                           |
| Authorization                  | Thông tin authentication dùng cho HTTP                                                                                                                                                                                                       | Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==                                |
| Cache-Control                  | Dùng để chỉ định instruction mà mọi cơ chế cache trong request/response chain này phải tuân thủ                                                                                                                                              | Cache-Control: no-cache                                                          |
| Connection                     | Loại connection mà browser muốn ưu tiên sử dụng                                                                                                                                                                                              | Connection: keep-alive                                                           |
| Content-Length                 | Độ dài request body được biểu diễn bằng octet array (byte 8 bit)                                                                                                                                                                             | Content-Length: 348                                                              |
| Content-MD5                    | Giá trị binary MD5 hash của request body, được encode bằng Base64                                                                                                                                                                            | Content-MD5: Q2hlY2sgSW50ZWdyaXR5IQ==                                            |
| Content-Type                   | Loại multimedia của request body (dùng trong POST và PUT request)                                                                                                                                                                            | Content-Type: application/x-www-form-urlencoded                                  |
| Cookie                         | HTTP Cookie trước đó được server gửi thông qua Set-Cookie (mô tả bên dưới)                                                                                                                                                                   | Cookie: $Version=1; Skin=new;                                                    |
| Date                           | Date và time gửi message (theo format “HTTP date” được định nghĩa trong RFC 7231)                                                                                                                                                            | Date: Tue, 15 Nov 1994 08:12:31 GMT                                              |
| Expect                         | Cho biết client yêu cầu server thực hiện hành vi cụ thể                                                                                                                                                                                      | Expect: 100-continue                                                             |
| From                           | Email address của user gửi request này                                                                                                                                                                                                       | From: `user@example.com`                                                         |
| Host                           | Domain của server (dùng cho virtual host) và port của TCP mà server đang listen. Nếu port được yêu cầu là standard port của service tương ứng, có thể bỏ port number.                                                                        | Host: en.wikipedia.org                                                           |
| If-Match                       | Chỉ thực hiện operation tương ứng khi entity do client cung cấp khớp với entity tương ứng trên server. Chủ yếu được dùng trong các method như PUT: chỉ update resource khi resource chưa bị sửa kể từ lần user update resource đó trước đây. | If-Match: "737060cd8c284d8af7ad3082f209582d"                                     |
| If-Modified-Since              | Cho phép server trả status code `304 Not Modified` khi resource được yêu cầu chưa bị sửa kể từ date chỉ định                                                                                                                                 | If-Modified-Since: Sat, 29 Oct 1994 19:43:31 GMT                                 |
| If-None-Match                  | Cho phép server trả status code `304 Not Modified` khi ETag của resource được yêu cầu chưa thay đổi                                                                                                                                          | If-None-Match: "737060cd8c284d8af7ad3082f209582d"                                |
| If-Range                       | Nếu entity chưa bị sửa, gửi phần còn thiếu; nếu không, gửi toàn bộ entity mới                                                                                                                                                                | If-Range: "737060cd8c284d8af7ad3082f209582d"                                     |
| If-Unmodified-Since            | Chỉ gửi response khi entity chưa bị sửa kể từ một thời điểm cụ thể                                                                                                                                                                           | If-Unmodified-Since: Sat, 29 Oct 1994 19:43:31 GMT                               |
| Max-Forwards                   | Giới hạn số lần message có thể được proxy và gateway forward.                                                                                                                                                                                | Max-Forwards: 10                                                                 |
| Origin                         | Gửi request cho cross-origin resource sharing.                                                                                                                                                                                               | `Origin: http://www.example-social-network.com`                                  |
| Pragma                         | Liên quan đến implementation cụ thể; các field này có thể tạo ra nhiều effect tại bất kỳ thời điểm nào trong request/response chain.                                                                                                         | Pragma: no-cache                                                                 |
| Proxy-Authorization            | Thông tin authentication dùng để authentication với proxy.                                                                                                                                                                                   | Proxy-Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==                          |
| Range                          | Chỉ yêu cầu một phần entity. Byte offset bắt đầu từ 0. Tham khảo byte serving.                                                                                                                                                               | Range: bytes=500-999                                                             |
| Referer                        | Cho biết page trước đó browser đã truy cập; chính một link trên page đó đã đưa browser đến page đang được yêu cầu.                                                                                                                           | `Referer: http://en.wikipedia.org/wiki/Main_Page`                                |
| TE                             | Encoding mà browser dự kiến chấp nhận: có thể sử dụng giá trị trong field Transfer-Encoding của response header;                                                                                                                             | TE: trailers, deflate                                                            |
| Upgrade                        | Yêu cầu server upgrade lên protocol khác.                                                                                                                                                                                                    | Upgrade: HTTP/2.0, SHTTP/1.3, IRC/6.9, RTA/x11                                   |
| User-Agent                     | Chuỗi nhận diện browser                                                                                                                                                                                                                      | User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:12.0) Gecko/20100101 Firefox/21.0 |
| Via                            | Cho server biết request này được gửi qua những proxy nào.                                                                                                                                                                                    | Via: 1.0 fred, 1.1 example.com (Apache/1.1)                                      |
| Warning                        | Warning chung, cho biết entity content body có thể tồn tại lỗi.                                                                                                                                                                              | Warning: 199 Miscellaneous warning                                               |

### ⭐️ HTTP và HTTPS khác nhau như thế nào? (Quan trọng)

![So sánh HTTP và HTTPS](https://oss.javaguide.cn/github/javaguide/cs-basics/network/http-vs-https.png)

- **Port number**: HTTP mặc định là 80, HTTPS mặc định là 443.
- **URL prefix**: URL prefix của HTTP là `http://`, URL prefix của HTTPS là `https://`.
- **Security và phương thức truyền**: Nếu không sử dụng TLS, HTTP mặc định không cung cấp confidentiality, integrity và peer authentication. HTTPS sử dụng TLS để bảo vệ HTTP; HTTP/1.1 và HTTP/2 thường sử dụng TLS over TCP, HTTP/3 sử dụng QUIC tích hợp TLS 1.3. TLS handshake phụ trách authentication peer và thiết lập traffic key, dữ liệu tiếp theo được bảo vệ bởi symmetric AEAD algorithm. Certificate chủ yếu dùng để authentication identity, không thể nói một cách khái quát rằng “certificate encrypt symmetric key”.
- **SEO (Search Engine Optimization)**: Search engine thường ưu tiên website sử dụng HTTPS vì HTTPS cung cấp security và bảo vệ privacy người dùng tốt hơn. Website sử dụng HTTPS có thể được hiển thị ưu tiên trong search result, từ đó ảnh hưởng đến SEO.

Để xem phần so sánh chi tiết hơn giữa HTTP và HTTPS, hãy đọc bài [HTTP vs HTTPS (Application layer)](https://javaguide.cn/cs-basics/network/http-vs-https.html).

### RSA và ECDHE trong HTTPS handshake khác nhau ở đâu? (Application layer)

Điểm khác biệt cốt lõi giữa RSA và ECDHE là: **key material của session là “được truyền đi” hay “được negotiate”.**

Trong static RSA handshake của TLS 1.2, client tạo `PreMasterSecret`, dùng RSA public key trong certificate của server để encrypt rồi gửi cho server; server dùng RSA private key để decrypt. Vấn đề là nếu attacker lưu lại handshake traffic lúc đó, sau này private key của server bị lộ thì có thể giải mã ngược session key trong lịch sử, vì vậy nó không có forward secrecy.

ECDHE không truyền shared secret trực tiếp. Client và server mỗi bên tạo một key pair tạm thời, sau khi exchange temporary public key, hai bên tính ra cùng một shared secret ở local. Private key của server certificate chủ yếu dùng để tạo chữ ký phục vụ authentication, chứng minh temporary parameter chưa bị man-in-the-middle thay thế, chứ không dùng để decrypt session key.

Tóm lại trong một câu: **RSA là client encrypt secret rồi gửi sang; ECDHE là hai bên dùng temporary key để negotiate secret. ECDHE hỗ trợ forward secrecy, vì vậy trở thành hướng chủ đạo của HTTPS hiện đại.**

Giới thiệu chi tiết: [RSA và ECDHE trong HTTPS handshake khác nhau ở đâu? (Application layer)](./https-rsa-vs-ecdhe)

### ⭐️ Có HTTP rồi, vì sao vẫn cần RPC?

HTTP và RPC không phải mối quan hệ bên này thay thế bên kia, cũng không phải vấn đề bên nào cao cấp hơn.

HTTP có thể gọi service, RPC cũng có thể gọi service. Khác biệt thực sự nằm ở việc bạn muốn xem remote call là một lần “truy cập resource” hay một lần “gọi method”.

Nếu là external API, chẳng hạn Web, App hoặc tích hợp hệ thống bên thứ ba, HTTP thường phù hợp hơn. Nó phổ biến, dễ debug, chi phí tích hợp thấp; người khác có thể dùng Postman hoặc curl để test.
Nếu là internal service call, đặc biệt khi có nhiều service, call chain dài, interface được gọi thường xuyên, đồng thời phải cân nhắc service discovery, timeout, retry, load balancing và distributed tracing, RPC sẽ thuận tiện hơn. RPC không chỉ đơn thuần để nhanh hơn, mà còn xử lý cùng lúc nhiều vấn đề phức tạp trong internal service call.

Vì vậy, đừng tiếp tục ghi nhớ đơn giản rằng “HTTP dùng bên ngoài, RPC dùng bên trong”.

Câu này có thể giúp người mới bắt đầu, nhưng khi thực sự làm project, vẫn cần xem xét đối tượng được gọi, team infrastructure, chi phí troubleshooting, yêu cầu performance và chi phí maintenance sau này.

Project không lớn, dùng HTTP đã chạy ổn định thì đừng cố dùng RPC chỉ vì “trông có vẻ microservice hơn”.

Nếu internal call ngày càng phức tạp, HTTP SDK, gateway, monitoring, retry được bổ sung ngày càng nhiều, khi đó có thể nghiêm túc cân nhắc RPC.

Một câu: **HTTP không yếu đến vậy, RPC cũng không thần kỳ đến vậy. Chọn cái nào chủ yếu phụ thuộc vào việc nó có thể giải quyết vấn đề hiện tại với chi phí thấp hơn hay không.**

Giới thiệu chi tiết: [⭐️Có HTTP rồi, vì sao vẫn cần RPC?](./http-vs-rpc.md)

### HTTP/1.0 và HTTP/1.1 khác nhau như thế nào?

![So sánh HTTP/1.0 và HTTP/1.1](https://oss.javaguide.cn/github/javaguide/cs-basics/network/http1.0-vs-http1.1.png)

- **Phương thức connection**: HTTP/1.0 là short connection, HTTP/1.1 hỗ trợ persistent connection. Persistent connection và short connection của HTTP thực chất là persistent connection và short connection của TCP.
- **Response status code**: HTTP/1.1 bổ sung rất nhiều status code; riêng error response status code đã thêm 24 loại. Ví dụ, `100 (Continue)` cho phép client xác nhận server sẵn sàng tiếp tục nhận request body lớn trước khi gửi; `206 (Partial Content)` là status code của range request; `409 (Conflict)` là khi request xung đột với quy định của resource hiện tại; `410 (Gone)` là khi target resource không còn khả dụng, trạng thái này nhiều khả năng là vĩnh viễn và server không biết forwarding address nào còn dùng được.
- **Cơ chế cache**: HTTP/1.0 chủ yếu dùng If-Modified-Since,Expires trong Header làm tiêu chí xác định cache; HTTP/1.1 bổ sung nhiều cache-control strategy hơn, chẳng hạn Entity tag, If-Unmodified-Since, If-Match, If-None-Match và nhiều cache header khác để kiểm soát cache strategy.
- **Bandwidth**: HTTP/1.0 có một số trường hợp lãng phí bandwidth, ví dụ client chỉ cần một phần object nhưng server lại gửi toàn bộ object, đồng thời không hỗ trợ resume download. HTTP/1.1 bổ sung range header trong request header, cho phép chỉ request một phần resource, response code là 206 (Partial Content), giúp developer chủ động sử dụng bandwidth và connection hiệu quả hơn.
- **Xử lý Host header (Host Header)**: HTTP/1.1 bổ sung Host header field, cho phép host nhiều domain trên cùng một IP address, từ đó hỗ trợ virtual host. HTTP/1.0 không có Host header field nên không thể thực hiện virtual host.

Để xem phần so sánh chi tiết hơn giữa HTTP/1.0 và HTTP/1.1, hãy đọc bài [HTTP/1.0 vs HTTP/1.1 (Application layer)](https://javaguide.cn/cs-basics/network/http1.0-vs-http1.1.html).

### ⭐️ HTTP/1.1 và HTTP/2.0 khác nhau như thế nào?

![So sánh HTTP/1.0 và HTTP/1.1](https://oss.javaguide.cn/github/javaguide/cs-basics/network/http1.1-vs-http2.0.png)

- **Multiplexing**: HTTP/2.0 có thể truyền đồng thời nhiều request và response trên cùng một connection (có thể xem là phiên bản nâng cấp của persistent connection trong HTTP/1.1), không ảnh hưởng lẫn nhau. HTTP/1.1 sử dụng phương thức tuần tự, mỗi request và response cần một connection riêng, đồng thời browser bị giới hạn 6-8 TCP connection để kiểm soát resource. Điều này khiến HTTP/2.0 hiệu quả hơn khi xử lý nhiều request, giảm network latency và cải thiện performance.
- **Binary Frames**: HTTP/2.0 truyền dữ liệu bằng binary frame, còn HTTP/1.1 dùng message dạng text. Binary frame gọn và hiệu quả hơn, giảm lượng data truyền và bandwidth tiêu thụ.
- **Head-of-line blocking**: HTTP/2 sử dụng multiplexing, cho phép nhiều request và response truyền song song, xen kẽ trên một TCP connection, giải quyết head-of-line blocking ở application layer của HTTP/1.1; tuy nhiên HTTP/2 vẫn chịu ảnh hưởng của head-of-line blocking ở TCP layer.
- **Header compression**: HTTP/1.1 hỗ trợ compression cho `Body` nhưng không hỗ trợ compression cho `Header`. HTTP/2.0 hỗ trợ compression cho `Header`, sử dụng HPACK algorithm được thiết kế riêng cho Header compression, giảm network overhead.
- **Server Push**: HTTP/2.0 hỗ trợ server push, khi client request một resource, server có thể push các resource liên quan khác cho client cùng lúc, từ đó giảm số request và latency của client. HTTP/1.1 cần client tự gửi request để lấy resource liên quan.

Hình minh họa hiệu quả multiplexing của HTTP/2.0 (nguồn hình: [HTTP/2 For Web Developers](https://blog.cloudflare.com/http-2-for-web-developers/)):

![Multiplexing nhiều request và response trên một TCP connection trong HTTP/2](https://oss.javaguide.cn/github/javaguide/cs-basics/network/http2.0-multiplexing.png)

Có thể thấy cơ chế multiplexing của HTTP/2 cho phép nhiều request và response share một TCP connection, tránh việc phải thiết lập nhiều parallel connection khi HTTP/1.1 xử lý concurrent request, giảm overhead do thiết lập và maintenance connection lặp lại. Trong HTTP/1.1, dù hỗ trợ persistent connection, browser thường thiết lập nhiều parallel connection cho cùng một domain để giảm head-of-line blocking.

### HTTP/2.0 và HTTP/3.0 khác nhau như thế nào?

![So sánh HTTP/2.0 và HTTP/3.0](https://oss.javaguide.cn/github/javaguide/cs-basics/network/http2.0-vs-http3.0.png)

- **Transport protocol**: HTTP/2 dựa trên TCP, còn HTTP/3 mapping HTTP semantics vào QUIC. QUIC xây dựng trên UDP, thực hiện reliable delivery, congestion control, flow control và TLS 1.3 security protection ở transport layer.
- **Thiết lập connection**: HTTPS connection của HTTP/2 cần thiết lập TCP connection trước rồi mới hoàn tất TLS handshake; HTTP/3 kết hợp negotiate transport parameter và TLS 1.3 handshake trong quá trình QUIC establish connection. QUIC connection mới thường sử dụng 1-RTT; 0-RTT chỉ áp dụng khi client có connection state trước đó để resume, đồng thời early data có rủi ro replay. Khi so sánh latency cũng cần thống nhất cùng một metric, chẳng hạn “thời điểm có thể gửi request đầu tiên” hoặc “thời điểm nhận byte đầu tiên”.
- **Header compression**: HTTP/2.0 sử dụng HPACK algorithm để header compression, còn HTTP/3.0 sử dụng QPACK header compression algorithm hiệu quả hơn.
- **Head-of-line blocking**: Nhiều stream của HTTP/2 multiplex trên cùng một TCP connection, packet loss của TCP sẽ block mọi stream trên connection này. QUIC cung cấp reliable ordered delivery độc lập giữa các stream; khi một stream mất data, stream đó sẽ chờ khôi phục data bị thiếu, nhưng thông thường không ngăn các stream khác tiếp tục.
- **Connection migration**: QUIC dùng Connection ID độc lập với IP/port four-tuple để định danh connection. Connection ID không cố định ở 64 bit: QUIC v1 có thể sử dụng Connection ID có độ dài từ zero đến 20 byte, endpoint còn có thể cấp phát và retire nhiều Connection ID. Sau khi network address thay đổi, endpoint có thể verify path mới và duy trì cùng một logical connection.
- **Error recovery**: HTTP/3.0 có error recovery mechanism tốt hơn. Khi xảy ra các vấn đề network như packet loss và latency, nó có thể recovery và retransmit nhanh hơn. HTTP/2.0 thì cần dựa vào error recovery mechanism của TCP.
- **Security**: HTTP/2 thường dùng TLS để bảo vệ HTTP header và data payload, nhưng IP header, TCP header và outer field của TLS record layer vẫn có thể nhìn thấy. QUIC dùng key được derive từ TLS để bảo vệ packet payload, đồng thời header protection cho packet number và một phần first-byte field; IP/UDP header và một phần QUIC header field vẫn có thể nhìn thấy, các message như Version Negotiation cũng không có cùng mức cryptographic protection, vì vậy không thể nói QUIC encrypt toàn bộ mọi header của packet.

So sánh protocol stack của HTTP/1.0, HTTP/2.0 và HTTP/3.0:

![So sánh protocol stack HTTP/1, HTTP/2 và HTTP/3](https://oss.javaguide.cn/github/javaguide/cs-basics/network/http-3-implementation.png)

Dưới đây là hình so sánh chi tiết hơn giữa HTTP/2.0 và HTTP/3.0:

![Hình so sánh chi tiết HTTP/2.0 và HTTP/3.0](https://oss.javaguide.cn/github/javaguide/cs-basics/network/http2-and-http3-stacks-comparison.png)

Có thể thấy từ hình trên:

- **HTTP/2.0**: Sử dụng TCP làm transport protocol, sử dụng HPACK để header compression, dựa vào TLS để thực hiện encryption.
- **HTTP/3.0**: Sử dụng QUIC dựa trên UDP, sử dụng QPACK hiệu quả hơn để header compression, tích hợp trực tiếp TLS trong QUIC. QUIC có các đặc tính như connection migration, congestion control và congestion avoidance, flow control.

Để xem phần giới thiệu chi tiết hơn về quá trình tiến hóa từ HTTP/1.0 -> HTTP/3.0, khuyến nghị đọc [Tối ưu engineering từ HTTP1 đến HTTP3](https://dbwu.tech/posts/http_evolution/).

### Head-of-line blocking của HTTP/1.1 và HTTP/2.0 khác nhau như thế nào?

Nguyên nhân chính gây head-of-line blocking trong HTTP/1.1 là không có multiplexing:

- Trong một TCP connection, request và response của resource được xử lý theo thứ tự. Nếu một resource lớn (như file lớn) đang được truyền, resource nhỏ phía sau (như CSS file nhỏ hơn) phải chờ resource phía trước truyền xong mới được gửi.
- Nếu browser cần load đồng thời nhiều resource (như nhiều CSS, JS file), nó thường mở nhiều parallel TCP connection (thường giới hạn ở 6 connection). Nhưng mỗi connection vẫn bị giới hạn bởi cơ chế request-response tuần tự, do đó vẫn xảy ra **head-of-line blocking ở application layer**.

Dù HTTP/2.0 bổ sung multiplexing, cho phép nhiều request và response truyền song song, xen kẽ trên một TCP connection, giải quyết **head-of-line blocking ở application layer của HTTP/1.1**, HTTP/2.0 vẫn chịu ảnh hưởng của **head-of-line blocking ở TCP layer**:

- HTTP/2.0 chia mỗi resource thành các block nhỏ thông qua frame mechanism và cấp stream ID duy nhất cho mỗi resource, nhờ đó data của nhiều resource có thể truyền xen kẽ trên cùng một TCP connection.
- TCP là transport-layer protocol, yêu cầu data được deliver theo thứ tự. Nếu một packet bị mất trong quá trình truyền, kể cả các packet sau đã đến nơi, vẫn phải chờ packet bị mất được retransmit rồi mới có thể tiếp tục xử lý. Tính tuần tự ở transport layer này dẫn đến **head-of-line blocking ở TCP layer**.
- Ví dụ, nếu một TCP packet của HTTP/2 chứa data của nhiều resource (chẳng hạn JS và CSS) bị mất, toàn bộ resource data trong các packet sau phải chờ packet bị mất được retransmit, khiến mọi stream đều bị block.

Cuối cùng, hãy dùng bảng sau để tổng hợp và bổ sung:

| **Khía cạnh**          | **Head-of-line blocking của HTTP/1.1**                             | **Head-of-line blocking của HTTP/2.0**                                                         |
| ---------------------- | ------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------- |
| **Layer**              | Application layer (hạn chế của HTTP protocol)                      | Transport layer (hạn chế của TCP protocol)                                                     |
| **Nguyên nhân gốc**    | Không có multiplexing, request và response phải truyền theo thứ tự | TCP yêu cầu packet được deliver theo thứ tự, packet loss sẽ block toàn bộ connection           |
| **Phạm vi ảnh hưởng**  | Một HTTP request/response block request/response phía sau.         | Một TCP packet loss ảnh hưởng đến mọi HTTP/2.0 stream (phụ thuộc cùng TCP connection bên dưới) |
| **Cách giảm thiểu**    | Mở nhiều parallel TCP connection                                   | Giảm packet loss hoặc sử dụng QUIC dựa trên UDP                                                |
| **Scenario ảnh hưởng** | Xảy ra mỗi lần, đặc biệt khi file lớn block file nhỏ.              | Dễ xảy ra hơn trong môi trường network có packet loss rate cao.                                |

### ⭐️ HTTP là protocol không lưu state, vậy lưu user state như thế nào?

Bản thân HTTP protocol là **stateless**. Điều này có nghĩa là trong trường hợp mặc định, server không thể phân biệt hai request liên tiếp có đến từ cùng một user hay không, cũng không biết operation trước đó của cùng user là gì. Điều này giống như một waiter “hay quên”: mỗi lần bạn nói chuyện, người đó không biết bạn là ai và cũng không biết trước đó bạn đã gọi món gì.

Nhưng trong Web application thực tế, chẳng hạn các scenario mua sắm online và user login, rõ ràng chúng ta cần ghi nhớ state của user (ví dụ sản phẩm trong shopping cart và thông tin login của user). Để giải quyết vấn đề này, có một số cơ chế thường dùng sau:

**Phương án 1: Session kết hợp Cookie (cách phổ biến):**

![Quy trình Session kết hợp Cookie để lưu trạng thái login của user](https://oss.javaguide.cn/github/javaguide/system-design/security/session-cookie-authentication-process.png)

Đây có thể nói là phương pháp kinh điển và phổ biến nhất. Quy trình cơ bản như sau:

1. User gửi username, password và verification code đến server để login vào system.
2. Sau khi xác thực thành công, server tạo và lưu một Session object riêng cho user này (có thể hiểu là một vùng memory trên server, lưu state data của user như shopping cart và thông tin login), đồng thời cấp một `SessionID` duy nhất cho Session.
3. Server gửi `SessionID` đến browser của user thông qua instruction `Set-Cookie` trong HTTP response header.
4. Sau khi nhận `SessionID`, browser lưu nó local dưới dạng Cookie. Khi user duy trì trạng thái login, mỗi lần gửi request đến server đó, browser tự động đính kèm Cookie chứa `SessionID`.
5. Sau khi nhận request, server lấy `SessionID` từ Cookie để tìm Session object đã lưu trước đó, từ đó biết user nào và state trước đó của user.

Khi sử dụng Session, cần chú ý các điểm sau:

- **Cookie support ở client**: Chức năng cốt lõi phụ thuộc vào Session cần đảm bảo browser của user đã bật Cookie.
- **Session expiration management**: Đặt thời gian expire của Session hợp lý, cân bằng security và user experience.
- **Session ID security**: Đặt flag `HttpOnly` cho Cookie chứa `SessionID` để ngăn client script (như JavaScript) đánh cắp nó; đặt flag Secure để đảm bảo `SessionID` chỉ được truyền trên HTTPS connection, tăng security.

Session data bản thân được lưu ở server-side. Các cách lưu trữ thường gặp gồm:

- **Server memory**: Dễ triển khai, tốc độ truy cập nhanh, nhưng data bị mất khi server restart và không thuận lợi cho load balancing giữa nhiều server. Cách này phù hợp với scenario nghiệp vụ đơn giản, số lượng user không lớn.
- **Database (như MySQL, PostgreSQL)**: Data được lưu bền vững, nhưng performance read/write tương đối thấp, thường không sử dụng cách này.
- **Distributed cache (như Redis)**: Performance cao, hỗ trợ distributed deployment, hiện là giải pháp rất phổ biến trong application quy mô lớn.

**Phương án 2: Khi Cookie bị disable: URL rewriting (URL Rewriting)**

Nếu browser của user disable Cookie hoặc trong một số trường hợp không tiện dùng Cookie, URL rewriting là một phương án thay thế. Cách này đính kèm trực tiếp `SessionID` vào cuối URL dưới dạng parameter. Ví dụ: <http://www.example.com/page?sessionid=xxxxxx>. Server-side sẽ parse parameter `sessionid` trong URL để lấy `SessionID`, từ đó tìm Session data tương ứng.

Phương pháp này thường không được sử dụng vì có các nhược điểm sau:

- URL dài hơn và không đẹp;
- `SessionID` lộ trong URL, security thấp (dễ bị copy, share hoặc ghi vào log);
- Có thể không thân thiện với SEO.

**Phương án 3: Token-based authentication (như JWT - JSON Web Tokens)**

Đây là phương thức stateless authentication ngày càng phổ biến, đặc biệt phù hợp với kiến trúc frontend và backend tách rời và microservice.

![Sơ đồ authentication bằng JWT](https://oss.javaguide.cn/github/javaguide/system-design/jwt/jwt-authentication%20process.png)

Lấy JWT làm ví dụ (Token thông thường cũng tương tự), các bước rút gọn như sau:

1. User gửi username, password và verification code đến server để login vào system.
2. Nếu username, password và verification code được kiểm tra chính xác, server sẽ trả về Token đã được ký, chính là JWT.
3. Client tự lưu Token sau khi nhận (ví dụ trong `localStorage` của browser).
4. Sau này, mỗi lần gửi request đến backend, user đính kèm JWT này trong Header.
5. Server kiểm tra JWT và lấy thông tin liên quan đến user từ đó.

Có thể xem giới thiệu chi tiết về JWT trong hai bài viết sau:

- [Giải thích chi tiết khái niệm cơ bản về JWT](https://javaguide.cn/system-design/security/jwt-intro.html)
- [Phân tích ưu, nhược điểm của JWT identity authentication](https://javaguide.cn/system-design/security/advantages-and-disadvantages-of-jwt.html)

Tóm lại, dù bản thân HTTP là stateless, chúng ta vẫn có thể theo dõi và quản lý user state hiệu quả trong Web application thông qua các cơ chế như Cookie + Session, URL rewriting hoặc Token. Trong đó, **Cookie + Session là phương thức truyền thống và được sử dụng rộng rãi nhất, còn Token-based authentication ngày càng phổ biến trong Web application hiện đại.**

### URI và URL khác nhau như thế nào?

- URI (Uniform Resource Identifier) là một identifier thống nhất, có thể định danh duy nhất một resource.
- URL (Uniform Resource Locator) là một locator thống nhất, có thể cung cấp path của resource đó. Đây là một URI cụ thể, tức URL vừa có thể dùng để định danh resource vừa chỉ ra cách locate resource.

URI có vai trò giống số căn cước, còn URL giống địa chỉ nhà hơn. URL là một URI cụ thể, không chỉ định danh duy nhất resource mà còn cung cấp thông tin để locate resource.

### Cookie và Session khác nhau như thế nào?

Nói chính xác hơn, câu hỏi này thuộc phạm vi authentication và authorization. Có thể tìm câu trả lời chi tiết trong bài [Giải thích chi tiết khái niệm cơ bản về authentication và authorization](https://javaguide.cn/system-design/security/basis-of-authority-certification.html).

### ⭐️ Sự khác nhau giữa GET và POST

Câu hỏi này được thảo luận rất sôi nổi trên Zhihu, địa chỉ: <https://www.zhihu.com/question/28586791>.

GET và POST là hai request method thường dùng trong HTTP protocol, có đặc điểm và cách sử dụng khác nhau tùy scenario và mục đích. Nhìn chung, có thể phân biệt hai method từ các khía cạnh sau (chỉ cần nắm rõ sự khác biệt về semantics của hai method là trọng tâm):

- Semantics (khác biệt chính): GET thường dùng để lấy hoặc query resource, còn POST thường dùng để create hoặc modify resource.
- Idempotent: GET request là idempotent, tức thực hiện lặp lại nhiều lần không làm thay đổi state của resource; POST request không idempotent, tức mỗi lần thực hiện có thể tạo kết quả khác nhau hoặc ảnh hưởng đến state của resource.
- Format: Parameter của GET request thường đặt trong URL, tạo thành query string (querystring); parameter của POST request thường đặt trong request body (body), có thể dùng nhiều encoding format như application/x-www-form-urlencoded, multipart/form-data và application/json. Độ dài URL của GET request bị giới hạn bởi browser và server, còn kích thước body của POST request không có giới hạn rõ ràng. Tuy nhiên, trên thực tế GET request cũng có thể dùng body để truyền data, chỉ là không khuyến nghị vì có thể dẫn đến vấn đề compatibility hoặc semantics.
- Cache: Vì GET request là idempotent, nó có thể được browser hoặc intermediate node khác (như proxy và gateway) cache để cải thiện performance và efficiency. POST request không phù hợp để cache vì có thể có side effect, mỗi lần thực hiện có thể cần response real-time.
- Security: Nếu sử dụng HTTP protocol, cả GET request và POST request đều không an toàn vì bản thân HTTP truyền data dưới dạng plaintext; bắt buộc phải dùng HTTPS để encrypt data. So với POST request, GET request dễ làm lộ sensitive data hơn vì parameter thường đặt trong URL.

Nhắc lại, điểm chính là nắm rõ sự khác biệt về semantics của hai method; trong quá trình sử dụng thực tế, cũng phân biệt dùng GET hay POST dựa trên semantics. Tuy nhiên, một số project dùng POST cho mọi request; điều này không cố định, miễn team thống nhất là được.

## WebSocket

### WebSocket là gì?

WebSocket là một full-duplex communication protocol dựa trên TCP connection, tức client và server có thể đồng thời gửi và nhận data.

WebSocket ra đời năm 2008 và trở thành international standard vào năm 2011. Gần như mọi phiên bản mới của browser phổ biến đều hỗ trợ protocol này. Tuy nhiên, WebSocket không chỉ dùng trong browser-based application; nhiều programming language, framework và server cũng cung cấp WebSocket support.

WebSocket về bản chất là application-layer protocol, dùng để bù đắp thiếu sót của HTTP về khả năng persistent communication. Client và server chỉ cần một lần handshake là có thể trực tiếp tạo persistent connection giữa hai bên và thực hiện bidirectional data transfer.

![WebSocket thiết lập persistent connection để thực hiện bidirectional communication](https://oss.javaguide.cn/github/javaguide/system-design/web-real-time-message-push/1460000042192394.png)

Các application scenario thường gặp của WebSocket:

- Video danmaku
- Real-time message push, xem chi tiết trong bài [Giải thích chi tiết Web real-time message push](https://javaguide.cn/system-design/web-real-time-message-push.html)
- Real-time game battle
- Multi-user collaborative editing
- Social chat
- …

### ⭐️ WebSocket và HTTP khác nhau như thế nào?

WebSocket và HTTP đều là application-layer protocol dựa trên TCP, đều có thể truyền data trong network.

Các điểm khác biệt chính như sau:

- WebSocket là bidirectional real-time communication protocol, còn HTTP là unidirectional communication protocol. Hơn nữa, communication trong HTTP chỉ có thể do client khởi xướng, server không thể chủ động thông báo cho client.
- WebSocket sử dụng `ws://` hoặc `wss://` (protocol được encrypt bằng SSL/TLS, tương tự quan hệ giữa HTTP và HTTPS) làm protocol prefix; HTTP sử dụng `http://` hoặc `https://` làm protocol prefix.
- WebSocket hỗ trợ extension, user có thể mở rộng protocol để triển khai một số subprotocol tùy chỉnh, chẳng hạn hỗ trợ compression và encryption.
- Format data của WebSocket communication tương đối nhẹ, packet header dùng để điều khiển protocol nhỏ hơn, network overhead thấp; còn mỗi HTTP communication đều phải mang full header nên network overhead lớn hơn (HTTP/2.0 dùng binary frame để truyền data và hỗ trợ header compression, giảm network overhead).

### WebSocket hoạt động như thế nào?

Quá trình hoạt động của WebSocket gồm các bước sau:

1. Client gửi HTTP request đến server; request header chứa các field như `Upgrade: websocket` và `Sec-WebSocket-Key`, cho biết muốn upgrade protocol thành WebSocket;
2. Sau khi nhận request, server thực hiện operation upgrade protocol. Nếu hỗ trợ WebSocket, server trả HTTP status code 101; response header chứa các field như `Connection: Upgrade` và `Sec-WebSocket-Accept: xxx`, cho biết upgrade thành công sang WebSocket protocol.
3. Client và server thiết lập WebSocket connection để thực hiện bidirectional data transfer. Data được truyền dưới dạng frame. Mỗi message của WebSocket có thể được chia thành nhiều data frame (đơn vị nhỏ nhất). Sender chia message thành nhiều frame gửi đến receiver, receiver nhận message frame rồi lắp ráp các frame liên quan thành message hoàn chỉnh.
4. Client hoặc server có thể chủ động gửi close frame để biểu thị muốn đóng connection. Bên còn lại sau khi nhận sẽ trả về một close frame, sau đó hai bên đóng TCP connection.

Ngoài ra, sau khi thiết lập WebSocket connection, heartbeat mechanism được sử dụng để duy trì tính ổn định và trạng thái hoạt động của connection.

### ⭐️ WebSocket khác short polling và long polling như thế nào?

Ba phương thức này đều nhằm giải quyết vấn đề “**client làm thế nào để lấy data mới nhất từ server kịp thời và thực hiện real-time update**”. Cách triển khai, efficiency và real-time performance của chúng khác nhau đáng kể.

**1. Short polling**

- **Nguyên lý**: Client gửi HTTP request theo khoảng thời gian cố định (ví dụ 5 giây) để hỏi server có data mới hay không. Server lập tức response sau khi nhận request.
- **Ưu điểm**: Dễ triển khai, compatibility tốt, có thể dùng HTTP request thông thường.
- **Nhược điểm**:
  - **Real-time performance trung bình**: Message có thể đến giữa hai lần polling, user phải chờ request tiếp theo mới biết.
  - **Lãng phí resource lớn**: Liên tục thiết lập/đóng connection, hơn nữa phần lớn request đều nhận được “không có message mới”, làm tăng đáng kể áp lực lên server và network.

**2. Long polling**

- **Nguyên lý**: Sau khi client gửi request, nếu server tạm thời không có data mới thì giữ connection cho đến khi có data mới hoặc timeout mới response. Client lập tức gửi request tiếp theo sau khi nhận response, tạo “pseudo-real-time”.
- **Ưu điểm**:
  - **Real-time performance khá tốt**: Có data mới là có thể push ngay, không cần chờ request định kỳ tiếp theo.
  - **Giảm empty response**: Giảm empty response vô ích và cải thiện efficiency.
- **Nhược điểm**:
  - **Tốn nhiều server resource**: Cần duy trì số lượng lớn connection trong thời gian dài, tiêu tốn server thread/connection.
  - **Lãng phí resource lớn**: Sau mỗi response vẫn cần thiết lập lại connection, hơn nữa vẫn dựa trên HTTP unidirectional request-response mechanism.

**3. WebSocket**

- **Nguyên lý**: Client và server thiết lập một persistent TCP connection sau một lần HTTP Upgrade handshake. Sau đó, hai bên có thể chủ động gửi data bất kỳ lúc nào, thực hiện communication thực sự full-duplex với latency thấp.
- **Ưu điểm**:
  - **Real-time performance mạnh**: Data có thể được gửi và nhận hai chiều ngay lập tức, latency cực thấp.
  - **Resource efficiency cao**: Connection được duy trì, không cần liên tục thiết lập/đóng, giảm resource tiêu thụ.
  - **Chức năng mạnh**: Hỗ trợ server chủ động push message và client chủ động khởi tạo communication.
- **Nhược điểm**:
  - **Hạn chế sử dụng**: Cần cả server và client hỗ trợ WebSocket protocol. Có yêu cầu nhất định về connection management (như heartbeat keep-alive và reconnect khi disconnect).
  - **Triển khai phức tạp**: Triển khai phức tạp hơn short polling và long polling.

![So sánh phương thức communication của WebSocket, short polling và long polling](https://oss.javaguide.cn/github/javaguide/system-design/web-real-time-message-push/1460000042192394.png)

### ⭐️ SSE và WebSocket khác nhau như thế nào?

SSE (Server-Sent Events) và WebSocket đều dùng để server push message real-time đến browser, giúp nội dung trang tự động update mà không cần user refresh thủ công. Dù mục tiêu tương tự, chúng có một số khác biệt quan trọng về cách hoạt động và scenario phù hợp:

1. **Phương thức communication:**
   - **SSE:** **Unidirectional communication**. Chỉ server mới có thể gửi data đến client (browser). Client không thể gửi data đến server qua cùng connection (cần khởi tạo HTTP request mới).
   - **WebSocket:** **Bidirectional communication (full-duplex)**. Client và server có thể gửi message cho nhau bất kỳ lúc nào, thực hiện real-time interaction thực sự.
2. **Protocol bên dưới:**
   - **SSE:** Dựa trên **HTTP/HTTPS protocol chuẩn**. Về bản chất là một HTTP request “persistent connection”, server giữ connection mở và liên tục gửi event stream. Không cần server hoặc protocol support đặc biệt, có thể sử dụng HTTP infrastructure hiện có.
   - **WebSocket:** Sử dụng **protocol độc lập `ws://` hoặc `wss://`**. Cần thông qua một HTTP "Upgrade" request cụ thể để thiết lập connection, đồng thời server phải hỗ trợ rõ ràng WebSocket protocol để xử lý connection và message frame.
3. **Độ phức tạp và chi phí triển khai:**
   - **SSE:** **Tương đối đơn giản**, chủ yếu xử lý ở server-side. Browser có EventSource API chuẩn, dễ sử dụng. Chi phí development và maintenance thấp.
   - **WebSocket:** **Phức tạp hơn một chút**. Server cần xử lý riêng WebSocket connection và protocol, client cũng cần sử dụng WebSocket API. Nếu cần cân nhắc compatibility, heartbeat và reconnect, chi phí development sẽ cao hơn.
4. **Reconnect khi disconnect:**
   - **SSE:** **Browser hỗ trợ native**. EventSource API cung cấp cơ chế tự động reconnect khi disconnect.
   - **WebSocket:** **Cần tự triển khai**. Developer phải tự viết logic để phát hiện disconnect và thử reconnect.
5. **Loại data:**
   - **SSE:** **Chủ yếu được thiết kế để truyền text** (UTF-8 encoding). Nếu cần truyền binary data, trước tiên cần chuyển thành text bằng encoding như Base64.
   - **WebSocket:** **Native support truyền text và binary data**, không cần encoding bổ sung.

Để mang lại user experience tốt hơn và tận dụng đặc tính đơn giản, hiệu quả, dựa trên HTTP chuẩn, **Server-Sent Events (SSE) hiện là lựa chọn phổ biến, thậm chí có thể nói là tiêu chuẩn, để các large language model API (như OpenAI, DeepSeek...) triển khai streaming response**.

Lấy DeepSeek làm ví dụ, chúng ta gửi một request rồi mở browser console để kiểm tra:

![DeepSeek response header](https://oss.javaguide.cn/github/javaguide/cs-basics/network/deepseek-sse.png)

![DeepSeek SSE response liên tục truyền event bằng text/event-stream](https://oss.javaguide.cn/github/javaguide/cs-basics/network/deepseek-sse-eventstream.png)

Có thể thấy response header chứa `text/event-stream`, cho biết thực sự đang sử dụng SSE. Hơn nữa, response data cũng thực sự được truyền liên tục theo block.

## PING

### PING command có tác dụng gì?

PING command là một network diagnosis tool thường dùng để kiểm tra connectivity và network latency giữa các host trong network.

Hãy xem một ví dụ đơn giản, chúng ta PING Baidu.

```bash
# Gửi 4 PING request packet đến www.baidu.com
❯ ping -c 4 www.baidu.com

PING www.a.shifen.com (14.119.104.189): 56 data bytes
64 bytes from 14.119.104.189: icmp_seq=0 ttl=54 time=27.867 ms
64 bytes from 14.119.104.189: icmp_seq=1 ttl=54 time=28.732 ms
64 bytes from 14.119.104.189: icmp_seq=2 ttl=54 time=27.571 ms
64 bytes from 14.119.104.189: icmp_seq=3 ttl=54 time=27.581 ms

--- www.a.shifen.com ping statistics ---
4 packets transmitted, 4 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 27.571/27.938/28.732/0.474 ms
```

Output của PING command thường gồm các phần thông tin sau:

1. **ICMP Echo Request (request message) information**: Sequence number, TTL (Time to Live) value.
2. **Domain name hoặc IP address của target host**: Dòng đầu tiên của output result.
3. **Round-trip time (RTT, Round-Trip Time)**: Tổng thời gian từ lúc gửi ICMP Echo Request (request message) đến khi nhận ICMP Echo Reply (response message), dùng để đo latency của network connection.
4. **Statistics**: Bao gồm số lượng ICMP request packet đã gửi, số lượng ICMP response packet đã nhận, packet loss rate, minimum, average, maximum và standard deviation của round-trip time (RTT).

Nếu target host không thể trả response chính xác cho PING, điều đó cho thấy connectivity giữa hai host có vấn đề (một số host hoặc network administrator có thể disable reply cho ICMP request, điều này cũng khiến không nhận được response chính xác). Nếu round-trip time (RTT) quá cao, điều đó cho thấy network latency quá cao.

### PING command hoạt động theo nguyên lý nào?

PING dựa trên **ICMP (Internet Control Message Protocol)** ở network layer, nguyên lý chính là gửi và nhận ICMP message trên network.

ICMP message chứa type field để xác định loại message. Có nhiều loại ICMP message, nhưng nhìn chung có thể chia thành hai nhóm:

- **Query message type**: Gửi request đến target host và chờ nhận response.
- **Error message type**: Gửi error message đến source host để báo cáo tình trạng lỗi trong network.

ICMP Echo Request (type 8) và ICMP Echo Reply (type 0) mà PING sử dụng đều thuộc query message type.

- PING command gửi ICMP Echo Request đến target host.
- Nếu connectivity giữa hai host bình thường, target host sẽ trả ICMP Echo Reply tương ứng.

### ⭐️ Ping được thì TCP chắc chắn kết nối được không?

Kết luận trước: **Không.**

Ping sử dụng ICMP (network layer), TCP connection sử dụng TCP (transport layer). Hai loại traffic có thể đi qua cùng một network path, nhưng intermediate device sẽ xử lý riêng theo protocol type, port, connection state và security policy. Ping được chỉ có thể chứng minh path của ICMP Echo có thể round-trip, không có nghĩa TCP port của target chắc chắn reachable.

![Khác biệt path giữa ICMP và TCP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/can-ping-but-tcp-may-not-connect-icmp-and-tcp-path-differences.png)

Một số nguyên nhân thường gặp:

- **Security policy của firewall khác nhau**: Nhiều network device cho phép ICMP (thuận tiện cho vận hành và giám sát), nhưng rule cho TCP port chặt chẽ hơn, có thể chỉ mở `22`, `80`, `443` và chặn mọi port khác.
- **Service chưa start hoặc port chưa listen**: Host có thể reply ICMP, nhưng Nginx chưa start hoặc MySQL chưa listen, nên Ping được nhưng TCP không kết nối được.
- **Có NAT / load balancing / security device ở giữa**: Đằng sau public IP có thể không phải một real server; ICMP response có thể đến từ intermediate device, không thể đồng nhất trực tiếp với việc backend service khả dụng.
- **HTTPS còn có thể bị kẹt ở SNI của TLS handshake**: TCP three-way handshake có thể thành công, nhưng SNI trong `ClientHello` bị intermediate device nhận diện và block, khiến connection reset hoặc bị treo.

Ngược lại cũng đúng: **Ping không được không có nghĩa TCP chắc chắn không kết nối được**. Một số server hoặc cloud security group disable ICMP trực tiếp, nhưng business port vẫn hoạt động bình thường.

Đề xuất troubleshooting: kiểm tra DNS trước (nếu dùng domain), sau đó dùng `ping` kiểm tra ICMP, tiếp đến dùng `nc` test port, cuối cùng dùng `curl` hoặc `openssl s_client` kiểm tra HTTPS / TLS. Đừng kết luận quá sớm chỉ dựa trên một command.

![Các layer troubleshooting HTTPS connection](https://oss.javaguide.cn/github/javaguide/cs-basics/network/can-ping-but-tcp-may-not-connect-https-connection-troubleshooting-layers.png)

Giới thiệu chi tiết: [Ping được thì TCP chắc chắn kết nối được không?](./can-ping-but-tcp-may-not-connect.md)

## DNS

### DNS có tác dụng gì?

DNS (Domain Name System) là hệ thống tên miền, protocol quan trọng đầu tiên được sử dụng sau khi user truy cập URL bằng browser. DNS cần giải quyết **vấn đề mapping giữa domain name và IP address**.

![Tổng quan hệ thống DNS resolve domain name thành IP address](https://oss.javaguide.cn/github/javaguide/cs-basics/network/dns-overview.png)

Trên một máy tính, có thể tồn tại browser DNS cache, operating system DNS cache và router DNS cache. Nếu không tìm thấy trong tất cả cache trên, DNS mới được sử dụng.

Thiết kế DNS hiện nay sử dụng distributed, hierarchical database structure. **DNS là application-layer protocol, có thể chạy trên UDP hoặc TCP, port là 53.**

### Có những DNS server nào? Có bao nhiêu root server?

DNS có thể được mô tả từ hai góc độ. Hierarchical authority gồm root, top-level domain và authoritative server của zone cụ thể; phía query gồm các role như stub resolver, recursive resolver và forwarder. Cùng một phần mềm hoặc cùng một server cũng có thể đảm nhận nhiều role, vì vậy các nhóm này không loại trừ lẫn nhau và cũng không bao quát mọi trường hợp.

- Root DNS server cung cấp thông tin referral đến top-level domain server cho bên query.
- Top-level domain DNS server thường trả về thông tin referral của authoritative server cho target domain.
- Authoritative DNS server lưu data của một hoặc nhiều zone và đưa ra authoritative answer cho query trong các zone đó.
- Recursive resolver nhận query từ client, trước tiên kiểm tra cache, khi cần sẽ lần lượt query root, TLD và authoritative server. Đây là role ở phía query, không phải một layer trong DNS authority hierarchy.

Về mặt logic, root server system có 13 root server identifier được đặt tên, từ `a.root-servers.net` đến `m.root-servers.net`, do 12 tổ chức độc lập vận hành. Mỗi identifier có thể được deploy thành nhiều physical instance thông qua Anycast; số lượng và địa điểm instance liên tục thay đổi, cần dựa vào [Root-Servers.org](https://root-servers.org/) để xem data real-time. Không thể hiểu 13 logical identifier là trên toàn cầu chỉ có 13 physical server.

### ⭐️ DNS resolve diễn ra như thế nào?

Toàn bộ quá trình có khá nhiều bước, tôi đã viết riêng một bài để giới thiệu chi tiết: [Giải thích chi tiết DNS (Domain Name System) (Application layer)](https://javaguide.cn/cs-basics/network/dns.html).

### Bạn biết DNS hijacking không? Ứng phó như thế nào?

DNS hijacking là một network attack, thông qua việc sửa DNS server resolution result khiến domain mà user truy cập trỏ đến IP address sai, từ đó khiến user không thể truy cập website bình thường hoặc bị dẫn đến website độc hại. DNS hijacking đôi khi còn được gọi là DNS redirection, DNS spoofing hoặc DNS poisoning.

## Tham khảo

- "Minh họa HTTP"
- "Computer Networks: A Top-Down Approach" (phiên bản thứ bảy)
- Giải thích chi tiết HTTP/2.0 và HTTPS protocol: <https://juejin.cn/post/7034668672262242318>
- Toàn bộ HTTP request header | HTTP Request Headers: <https://www.flysnow.org/tools/table/http-request-headers/>
- HTTP1, HTTP2, HTTP3: <https://juejin.cn/post/6855470356657307662>
- Đánh giá HTTP/3 như thế nào? - Câu trả lời của Che Xiaopang - Zhihu: <https://www.zhihu.com/question/302412059/answer/533223530>

<!-- @include: @article-footer.snippet.md -->
