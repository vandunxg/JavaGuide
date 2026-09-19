---
title: Từ khi nhập URL đến khi hiển thị trang, điều gì thực sự xảy ra?
description: Xâu chuỗi toàn bộ quy trình từ khi nhập URL đến khi render trang, bao quát DNS, TCP, HTTP, TLS, ARP, đóng gói dữ liệu và trình duyệt render, hỗ trợ hiểu về phỏng vấn và thực tế.
category: Computer Basics
tag:
  - Computer Network
head:
  - - meta
    - name: keywords
      content: quy trình truy cập trang web,DNS,thiết lập TCP,yêu cầu HTTP,bắt tay TLS,ARP,tải tài nguyên,trình duyệt render,đóng kết nối
---

Từ lúc nhập URL vào thanh địa chỉ đến khi trang hiển thị, quy trình phía sau đi qua nhiều khâu như DNS, TCP, TLS, HTTP, ARP, đóng gói dữ liệu và render của trình duyệt.

Câu hỏi này thường được dùng để kiểm tra hiểu biết tổng thể về mạng, vì nó kết nối các kiến thức ở application layer, transport layer, network layer và link layer. Chỉ học thuộc từng protocol riêng lẻ rất dễ bị rời rạc; đi qua toàn bộ quy trình truy cập trang web một lần sẽ dễ hiểu hơn nhiều.

Bài viết này chủ yếu trả lời một số câu hỏi:

1. Sau khi nhập URL, trình duyệt sẽ xử lý cục bộ những gì trước?
2. Quy trình DNS phân giải domain name diễn ra như thế nào?
3. TCP thiết lập connection như thế nào? Nếu dùng HTTPS, TLS handshake thực hiện những gì?
4. Quy trình tương tác giữa HTTP request và response là gì?
5. Data packet từ host đến server đi qua những lớp đóng gói và chuyển tiếp nào?
6. Sau khi nhận HTML, trình duyệt tiếp tục tải các resource như CSS, JS, ảnh và render trang như thế nào?
7. Sau khi tải trang hoàn tất, connection sẽ được tái sử dụng hoặc đóng như thế nào?

Nhìn chung, mô hình truyền thông mạng có thể được biểu diễn bằng hình dưới đây. Quy trình truy cập trang web chính là quá trình data được đóng gói từng lớp từ application layer đi xuống, truyền qua network vật lý đến đầu bên kia, rồi được tháo gói từng lớp từ dưới lên.

![Sự phối hợp của mô hình mạng năm lớp trong quá trình truy cập trang web](https://oss.javaguide.cn/github/javaguide/cs-basics/network/five-layers.png)

Trước khi bắt đầu, hãy xem nhanh toàn bộ quy trình:

1. **Trình duyệt phân tích URL và kiểm tra cache**: Trình duyệt phân tích các thành phần của URL và kiểm tra HTTP cache xem đã có bản sao hợp lệ của resource này chưa (strong cache, negotiated cache).
2. **DNS resolution**: Trình duyệt thông qua DNS protocol để lấy IP address tương ứng với domain name.
3. **Thiết lập transport connection**: HTTP/1.1 và HTTP/2 thường thiết lập TCP connection trước bằng TCP three-way handshake; HTTP/3 thì thiết lập QUIC connection dựa trên UDP.
4. **Thiết lập secure channel (HTTPS)**: HTTP/1.1 và HTTP/2 thường thực hiện TLS handshake sau khi thiết lập TCP connection; HTTP/3 tích hợp TLS 1.3 trong quá trình thiết lập QUIC connection để hoàn tất key negotiation và server authentication.
5. **Gửi HTTP request**: Trình duyệt gửi HTTP request message đến server trên connection để yêu cầu lấy nội dung trang web.
6. **Server xử lý và trả response**: Sau khi nhận request, server xử lý và trả về HTTP response message.
7. **Trình duyệt phân tích và render**: Trình duyệt phân tích HTML, CSS, thực thi JavaScript và tải các resource khác được tham chiếu trong trang (ảnh, font, v.v.).
8. **Quản lý connection**: Sau khi tải trang hoàn tất, connection được tái sử dụng hoặc đóng theo chính sách keep-alive.

Sau đây sẽ lần lượt trình bày theo quy trình này.

## Bước 1: Phân tích URL và kiểm tra cache

Mở trình duyệt, nhập URL vào thanh địa chỉ rồi nhấn Enter. Việc đầu tiên trình duyệt làm không phải là gửi request, mà là phân tích URL và kiểm tra xem có thể sử dụng trực tiếp local cache hay không.

### URL là gì

URL (Uniform Resource Locator, bộ định vị tài nguyên thống nhất) là địa chỉ duy nhất của tài nguyên trên Internet. Mỗi tài nguyên có thể truy cập trên mạng đều tương ứng với một URL; về lý thuyết, file và URL tương ứng một-một. Trên thực tế cũng có ngoại lệ, chẳng hạn trong trường hợp redirect hoặc CDN, nhiều URL có thể trỏ đến cùng một resource.

### Cấu trúc thành phần của URL

![Cấu trúc thành phần của URL](https://oss.javaguide.cn/github/javaguide/cs-basics/network/URL-parts.png)

Một URL hoàn chỉnh gồm các phần sau:

1. **Protocol** (Scheme): Tiền tố của URL biểu thị protocol được sử dụng; phổ biến nhất là `http` và `https`, ngoài ra còn có `ftp:` dùng để truyền file.
2. **Domain name** (Host): Tên định danh của đích truy cập, cũng có thể dùng trực tiếp IP address. Về bản chất, domain name là phiên bản dễ đọc của IP address.
3. **Port** (Port): Nằm ngay sau domain name, được phân cách bằng dấu hai chấm. HTTP mặc định là 80, HTTPS mặc định là 443; nếu sử dụng port mặc định thì có thể bỏ qua.
4. **Resource path** (Path): Bắt đầu từ `/` đầu tiên, biểu thị vị trí resource trên server. Trong thiết kế ban đầu, path tương ứng với file vật lý trên server; hiện nay thường là virtual path được backend ánh xạ tới route.
5. **Query parameter** (Query): Phần sau `?`, có dạng cặp key-value `key=value`; nhiều parameter được phân cách bằng `&`. Server sẽ trích xuất các parameter này khi phân tích request.
6. **Anchor** (Fragment): Phần sau `#`, dùng để định vị một vị trí trong trang. Anchor **không** được gửi đến server như một phần của request mà chỉ được trình duyệt xử lý cục bộ.

### Kiểm tra browser cache

Sau khi phân tích xong URL, trình duyệt sẽ kiểm tra HTTP cache trước để xem đã có bản sao hợp lệ của resource đó chưa:

1. **Strong cache**: Kiểm tra header `Cache-Control` (chẳng hạn `max-age`) hoặc `Expires` để xác định cache còn trong thời hạn hiệu lực hay không. Nếu còn hiệu lực, dùng cache trực tiếp và bỏ qua toàn bộ network request tiếp theo.
2. **Negotiated cache**: Khi strong cache không được hit, trình duyệt gửi verification request đến server (mang theo `If-Modified-Since` hoặc `If-None-Match`), server xác định resource có thay đổi hay không. Nếu không thay đổi, trả về `304 Not Modified`, trình duyệt tiếp tục dùng local cache; nếu đã thay đổi, trả về `200 OK` và resource mới.

Khi HTTP cache được hit, toàn bộ quy trình truy cập kết thúc tại đây và không cần gửi network request.

### Chuẩn bị phân giải domain name

Nếu HTTP cache không được hit, trình duyệt cần gửi network request đến server, trước tiên phải lấy IP address tương ứng với domain name. Trước khi chính thức gửi DNS query, trình duyệt còn lần lượt kiểm tra:

1. **Browser DNS cache**: Trình duyệt tự duy trì một DNS cache, trước tiên xem có record của domain name đó hay không.
2. **Operating system DNS cache**: Khi browser cache không được hit, truy vấn DNS cache của operating system.
3. **File hosts**: Operating system kiểm tra file `hosts` cục bộ để xem có mapping trực tiếp từ domain name đến IP address hay không. Nếu có thì dùng trực tiếp IP address đó và bỏ qua DNS resolution.

Nếu tất cả đều không được hit, trình duyệt cần thực hiện DNS query đầy đủ.

## Bước 2: DNS resolution

DNS (Domain Name System, hệ thống tên miền) giải quyết **bài toán ánh xạ giữa domain name và IP address**. Domain name chỉ là tên thuận tiện cho con người ghi nhớ; giao tiếp mạng thực sự cần IP address.

### Quy trình DNS resolution

Sau khi nhận domain name, quy trình DNS resolution của trình duyệt thường diễn ra theo các bước sau:

1. **Browser DNS cache**: Trình duyệt tự duy trì một DNS cache, trước tiên kiểm tra cache xem có record của domain name đó và record chưa hết hạn hay không.
2. **Operating system DNS cache**: Khi browser cache không được hit, gửi DNS query request đến operating system. Operating system cũng có DNS cache riêng.
3. **Local DNS server**: Local DNS server do operating system cấu hình (thường do ISP cung cấp hoặc dùng public DNS như `8.8.8.8`, `114.114.114.114`). Nếu local DNS server có cache chưa hết hạn thì trả kết quả trực tiếp.
4. **Recursive/iterative query**: Khi local DNS server không hit cache, nó thay mặt client thực hiện iterative query: trước tiên hỏi root DNS server, sau đó hỏi top-level domain DNS server (chẳng hạn `.com`), cuối cùng hỏi authoritative DNS server để lần lượt lấy IP address mục tiêu.
5. **Trả kết quả và cache**: Sau khi local DNS server lấy được kết quả cuối cùng, nó trả về client đồng thời lưu một bản vào local cache để dùng cho các query sau.

Hình dưới đây minh họa một quy trình iterative query của DNS điển hình:

![Quy trình DNS resolution](https://oss.javaguide.cn/github/javaguide/cs-basics/network/DNS-process.png)

Trong thực tế, local DNS server thường đã cache rất nhiều địa chỉ của TLD server. Phần lớn query không cần bắt đầu từ root server; trường hợp bỏ qua root server và tra cứu trực tiếp TLD rất phổ biến.

> Để biết thêm chi tiết về DNS (cấp bậc DNS server, khác biệt giữa recursive query và iterative query, các loại DNS record, lý do thường dùng UDP, v.v.), bạn có thể tham khảo bài viết [Giải thích chi tiết DNS Domain Name System (Application Layer)](https://javaguide.cn/cs-basics/network/dns.html).

## Bước 3: Thiết lập transport connection

Sau khi lấy được IP address của server mục tiêu, trình duyệt cần thiết lập transport connection với server. Trước tiên, bài viết giới thiệu TCP connection thường được HTTP/1.1 và HTTP/2 sử dụng; nếu negotiation chọn HTTP/3 thì sẽ chuyển sang thiết lập QUIC connection, QUIC được xây dựng trên UDP.

### TCP three-way handshake

Mục đích của TCP three-way handshake là **đồng bộ initial sequence number của hai bên** và **xác nhận đường truyền gửi nhận của hai bên khả dụng**.

![Minh họa TCP three-way handshake](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-shakes-hands-three-times.png)

1. **Handshake lần thứ nhất (SYN)**: Client gửi SYN segment, mang theo initial sequence number của mình `seq=x`, chuyển sang trạng thái `SYN_SENT`.
2. **Handshake lần thứ hai (SYN+ACK)**: Server nhận được rồi trả lời SYN+ACK, mang theo initial sequence number của mình `seq=y`, acknowledgement number `ack=x+1`, chuyển sang trạng thái `SYN_RCVD`.
3. **Handshake lần thứ ba (ACK)**: Client nhận được rồi gửi ACK, acknowledgement number `ack=y+1`; hai bên chuyển sang trạng thái `ESTABLISHED`, connection được thiết lập.

Thiết kế three-way handshake không nhằm mục đích «đi thêm một bước», mà để hai bên đều xác nhận rằng: đối phương có thể nhận data của mình và mình cũng có thể nhận data của đối phương. Two-way handshake không làm được điều này — sau handshake lần thứ hai, server vẫn chưa biết client đã nhận SYN+ACK của mình hay chưa.

> Để xem phân tích chi tiết về three-way handshake, half-connection queue/full-connection queue, SYN Flood protection, v.v., bạn có thể tham khảo [TCP three-way handshake và four-way handshake (Transport Layer)](https://javaguide.cn/cs-basics/network/tcp-connection-and-disconnection.html).

### Nếu là HTTPS: TLS handshake

Nếu sử dụng HTTPS dựa trên TCP, sau khi TCP connection được thiết lập còn phải thực hiện TLS handshake. HTTP/3 tích hợp TLS 1.3 trong quá trình thiết lập QUIC connection. TLS có ba mục tiêu cốt lõi: **encryption** (chống nghe lén), **authentication** (chống giả mạo), **integrity check** (chống sửa đổi).

Quy trình TLS handshake khái quát (lấy TLS 1.2 RSA key exchange làm ví dụ):

1. **Client Hello**: Client gửi TLS version được hỗ trợ, danh sách cipher suite và một random number.
2. **Server Hello**: Server chọn một cipher suite trong số đó, trả về certificate của mình và một random number khác.
3. **Key exchange**: Client xác minh tính hợp lệ của server certificate (thông qua CA signature verification), sau đó tạo pre-master secret (Pre-Master Secret), mã hóa bằng public key của server rồi gửi đến server. Hai bên dựa trên pre-master secret và hai random number đã trao đổi trước đó để tính session key dùng cho symmetric encryption.
4. **Hoàn tất**: Hai bên mã hóa communication bằng session key, handshake kết thúc.

Cần lưu ý rằng quy trình trên mô tả phương thức RSA-based key exchange trong TLS 1.2. HTTPS hiện đại thường dùng ECDHE key exchange; TLS 1.3 cũng có thể dùng PSK hoặc PSK+(EC)DHE trong các trường hợp như session resumption. Shared secret của ECDHE được hai bên tạo ra thông qua temporary key negotiation và không truyền trực tiếp trên network; nếu chỉ có private key của server certificate thì thường không thể decrypt các session trước đó đã dùng temporary key negotiation. TLS 1.3 tiếp tục đơn giản hóa handshake, thường rút gọn full handshake xuống 1-RTT và loại bỏ static RSA key exchange.

Sau khi TLS handshake hoàn tất, các HTTP request và response tiếp theo đều được encrypt bằng symmetric key đã negotiate khi truyền đi. Tính an toàn của HTTPS đến từ TLS layer, không phải từ việc thay đổi bản thân HTTP protocol.

> Để biết phân tích chi tiết về nguyên lý encryption của TLS (asymmetric encryption, symmetric encryption, digital signature, CA certificate), bạn có thể tham khảo [HTTP vs HTTPS (Application Layer)](https://javaguide.cn/cs-basics/network/http-vs-https.html). Để biết khác biệt giữa hai phương thức key exchange RSA và ECDHE, bạn có thể tham khảo [Quy trình HTTPS RSA vs ECDHE handshake](https://javaguide.cn/cs-basics/network/https-rsa-vs-ecdhe.html).

## Bước 4: Gửi HTTP request

Sau khi TCP connection (và TLS channel nếu có) được thiết lập, trình duyệt có thể gửi HTTP request.

### Cấu trúc HTTP request message

Một HTTP/1.1 request message điển hình như sau:

```http
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)
Accept: text/html,application/xhtml+xml
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Connection: keep-alive
Cookie: session_id=abc123
```

Ý nghĩa từng phần:

- **Request line**: `GET /index.html HTTP/1.1` — request method (GET), resource path (`/index.html`), protocol version (HTTP/1.1).
- **Host header**: Chỉ định target hostname. Đây là yêu cầu bắt buộc của HTTP/1.1, vì cùng một server (cùng một IP) có thể host nhiều website thông qua virtual host.
- **Các request header khác**: `User-Agent` (thông tin client), `Accept` (response type có thể chấp nhận), `Accept-Encoding` (compression method được hỗ trợ), `Cookie` (state information được mang theo), v.v.

### Server xử lý request

Sau khi nhận request, server trải qua một loạt xử lý để tạo response:

1. **Nhận request**: Web server (chẳng hạn Nginx, Tomcat) nhận và phân tích HTTP request message.
2. **Định tuyến request**: Dựa trên URL path để route request đến backend processing logic tương ứng (Controller, Servlet, v.v.).
3. **Business processing**: Thực thi business logic cụ thể, có thể liên quan đến database query, đọc cache, gọi service khác, v.v.
4. **Xây dựng response**: Đóng gói kết quả xử lý thành HTTP response message.

### Cấu trúc HTTP response message

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Encoding: gzip
Content-Length: 1256
Cache-Control: max-age=3600
Set-Cookie: session_id=xyz789; Path=/

<!DOCTYPE html>
<html>
...
</html>
```

Ý nghĩa từng phần:

- **Status line**: `HTTP/1.1 200 OK` — protocol version, status code (200), status description.
- **Response header**: `Content-Type` (response body type), `Content-Encoding` (compression method), `Cache-Control` (cache policy), `Set-Cookie` (thiết lập Cookie), v.v.
- **Response body**: Nội dung thực tế được yêu cầu, chẳng hạn HTML document, JSON data, binary data của ảnh, v.v.

Các status code thường gặp:

| Status code | Loại         | Ví dụ thường gặp                                                |
| ----------- | ------------ | --------------------------------------------------------------- |
| 2xx         | Thành công   | 200 OK, 206 Partial Content                                     |
| 3xx         | Redirect     | 301 redirect vĩnh viễn, 302 redirect tạm thời, 304 chưa sửa đổi |
| 4xx         | Client error | 400 Bad Request, 403 Forbidden, 404 Not Found                   |
| 5xx         | Server error | 500 Internal Server Error, 502 Bad Gateway                      |

> Để biết tổng hợp chi tiết về các status code HTTP thường gặp, bạn có thể tham khảo [Tổng hợp status code HTTP thường gặp (Application Layer)](https://javaguide.cn/cs-basics/network/http-status-codes.html).

## Bước 5: Đóng gói và chuyển tiếp data packet

Sau khi HTTP request được gửi từ trình duyệt, dữ liệu không trực tiếp «bay» đến server. Nó cần trải qua quá trình đóng gói từng lớp của protocol stack, rồi được chuyển tiếp từng hop trên network vật lý đến đích.

### Quy trình data encapsulation

HTTP message ở application layer được đóng gói lần lượt qua transport layer, network layer và link layer, cuối cùng trở thành bit stream có thể truyền trên physical medium:

![Tổng quan protocol của các lớp TCP/IP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/network-protocol-overview.png)

Mỗi layer chỉ quan tâm đến header cần thêm và sử dụng service do layer bên dưới cung cấp để truyền data:

- **Transport layer (TCP)**: Thêm source port và destination port, dùng sequence number và acknowledgement number để bảo đảm truyền dữ liệu tin cậy.
- **Network layer (IP)**: Thêm source IP và destination IP, chịu trách nhiệm addressing và routing, quyết định path mà data packet đi từ source đến destination.
- **Link layer**: Thêm source MAC address và destination MAC address, chịu trách nhiệm truyền data frame giữa các node liền kề.

### Routing và forwarding ở network layer

Data packet từ source host đến destination host thường cần đi qua nhiều router trung chuyển. Chức năng cốt lõi của network layer là **routing và forwarding**:

- **Routing**: Xác định path mà packet đi từ source đến destination (do các routing protocol như OSPF, BGP, v.v. tính toán).
- **Forwarding**: Chuyển packet từ input port của router sang output port phù hợp.

Mỗi router duy trì một routing table, tra bảng dựa trên destination IP address để quyết định next hop. Data packet trong network giống như một bưu kiện; mỗi trạm chỉ xem «gửi đến đâu ở trạm tiếp theo» mà không cần quan tâm toàn bộ path.

### ARP protocol: từ IP address đến MAC address

Khi data frame được truyền qua link layer, cần biết MAC address của thiết bị ở next hop chứ không thể chỉ dùng IP address. ARP (Address Resolution Protocol, giao thức phân giải địa chỉ) giải quyết bài toán «đã biết IP address thì lấy MAC address tương ứng như thế nào».

ARP hoạt động theo cách **broadcast query, unicast response**:

1. Host trước tiên tra local ARP cache table để xem đã có MAC address tương ứng với target IP hay chưa.
2. Khi cache không được hit, broadcast một ARP request trong LAN: «IP của ai là xxx.xxx.xxx.xxx? Hãy cho tôi biết MAC address của bạn.»
3. Target device (hoặc router interface) nhận được rồi trả lời MAC address của mình bằng unicast.
4. Bên gửi request nhận response rồi lưu IP-MAC mapping vào ARP cache table; các lần communication sau dùng trực tiếp mapping này.

Nếu target host không nằm trong cùng subnet, host không cần biết MAC address của target cuối cùng mà chỉ cần biết **MAC address của local gateway (router)**. Data packet trước tiên gửi đến gateway, sau đó gateway chuyển tiếp từng hop đến target network.

> Để biết nguyên lý hoạt động chi tiết của ARP (addressing cùng subnet/khác subnet, ARP table, các attack thường gặp), bạn có thể tham khảo [Giải thích chi tiết ARP protocol (Network Layer)](https://javaguide.cn/cs-basics/network/arp.html).

### Network Address Translation (NAT)

Trong phần lớn mạng gia đình và doanh nghiệp, host trong intranet dùng private IP address (chẳng hạn `192.168.x.x`), không thể route trực tiếp trên public network. NAT (Network Address Translation) protocol chịu trách nhiệm chuyển đổi IP address giữa intranet và public network.

Khi intranet host gửi data packet đến public network, NAT device (thường là router) thay source IP address từ private address thành public address và ghi lại port mapping. Khi response packet quay về, NAT dựa trên mapping table để chuyển destination address trở lại private address của intranet host.

## Bước 6: Trình duyệt phân tích và render

Sau khi server trả về HTML response, công việc thực sự của trình duyệt mới bắt đầu. Trình duyệt cần phân tích HTML, xây dựng DOM tree, tải sub-resource, tính toán style, layout và cuối cùng render lên màn hình.

### Phân tích HTML và xây dựng DOM

Sau khi nhận HTML document, trình duyệt phân tích từng dòng từ trên xuống dưới:

1. **Xây dựng DOM tree**: Phân tích HTML tag, tạo document object model (DOM) tree biểu thị cấu trúc trang.
2. **Xây dựng CSSOM tree**: Khi gặp CSS file được `<link>` tham chiếu hoặc tag `<style>`, tải và phân tích CSS, tạo CSS object model (CSSOM) tree biểu thị style rule của trang.
3. **Xây dựng render tree**: Gộp DOM tree và CSSOM tree để tạo render tree. Render tree chỉ chứa node cần hiển thị và thông tin style của chúng (element có `display: none` sẽ không xuất hiện trong render tree).
4. **Layout**: Tính toán vị trí và kích thước của từng node trong render tree.
5. **Paint**: Chuyển render tree sau layout thành pixel trên màn hình.
6. **Composite**: Ghép các layer khác nhau thành khung hình cuối cùng để hiển thị trên màn hình.

### Tải sub-resource

HTML document thường tham chiếu rất nhiều resource bên ngoài:

- **CSS file** (`<link rel="stylesheet">`): Stylesheet thông thường có tính blocking sẽ block rendering; trình duyệt thường chờ CSS load và parse xong mới layout và paint, vì CSS có thể thay đổi layout của element. Có thể thay đổi hành vi này thông qua attribute `media`, dynamic loading hoặc `preload`.
- **JavaScript file** (`<script>`): Mặc định sẽ block HTML parsing, vì JavaScript có thể sửa DOM. Có thể thay đổi hành vi load thông qua attribute `async` hoặc `defer`.
- **Ảnh, font, v.v.**: Không block HTML parsing, nhưng chỉ có thể hiển thị sau khi load xong.

Việc load các sub-resource này sẽ trigger thêm HTTP request. Nếu dùng HTTP/1.1, trình duyệt thường duy trì tối đa 6 TCP connection đồng thời cho cùng một domain name để download resource song song. Cơ chế multiplexing của HTTP/2 cho phép truyền song song nhiều resource trên cùng một TCP connection.

### Thực thi JavaScript

Việc thực thi JavaScript sẽ block HTML parsing. Khi gặp tag `<script>`, trình duyệt tạm dừng xây dựng DOM, trước tiên download và execute script. Nếu script có DOM operation, có thể trigger việc xây dựng lại DOM và page reflow (Reflow) hoặc repaint (Repaint).

Các biện pháp tối ưu thường dùng trong frontend development hiện đại gồm:

- Đặt `<script>` ở cuối `<body>` hoặc dùng attribute `defer` để tránh block DOM parsing.
- Dùng attribute `async` để load bất đồng bộ các script không ảnh hưởng đến DOM.
- Tăng tốc tải static resource thông qua CDN.
- Tận dụng browser cache để giảm request lặp lại.

## Bước 7: Quản lý connection

Sau khi page và resource load xong, connection thường không bị ngắt ngay; cách quản lý connection phụ thuộc vào HTTP version và configuration. HTTP/1.0, HTTP/1.1 và HTTP/2 dưới đây chủ yếu nói về TCP connection; HTTP/3 quản lý QUIC connection và stream.

### HTTP/1.0 short connection

HTTP/1.0 mặc định dùng short connection: sau khi mỗi request-response hoàn tất thì đóng TCP connection. Nếu page tham chiếu 10 external resource, trình duyệt cần thiết lập 10 TCP connection độc lập; mỗi connection đều phải trải qua three-way handshake và four-way handshake, phần lớn thời gian bị tiêu tốn vào việc thiết lập và giải phóng connection.

### HTTP/1.1 long connection (Keep-Alive)

HTTP/1.1 mặc định dùng long connection (Connection: keep-alive). Sau khi một TCP connection được thiết lập, có thể liên tục gửi nhiều request và nhận nhiều response mà không cần handshake lại mỗi lần. Nhờ đó, các sub-resource như CSS, JS và ảnh trong page có thể dùng lại cùng một TCP connection để load.

Long connection không phải là vĩnh viễn. Server thường cấu hình idle timeout (chẳng hạn `KeepAliveTimeout`); nếu không có request mới trong thời gian timeout thì server mới chủ động đóng connection.

### HTTP/2 multiplexing

HTTP/2 bổ sung multiplexing trên nền long connection. Nhiều request và response có thể được truyền đồng thời trên cùng một TCP connection; data được tách thành các frame nhỏ hơn và dùng stream (Stream) để phân biệt chúng thuộc về request nào. Điều này giải quyết head-of-line blocking ở application layer của HTTP/1.1 — request chậm ở phía trước sẽ không chặn request phía sau. Cần lưu ý rằng HTTP/2 vẫn dựa trên TCP; khi xảy ra packet loss ở TCP layer, mọi stream trên cùng connection đều bị ảnh hưởng (head-of-line blocking ở TCP layer). HTTP/3 dựa trên QUIC protocol (UDP) mới tiếp tục giảm nhẹ vấn đề này.

### Đóng connection

Khi connection thực sự cần đóng (do bên chủ động đóng khởi xướng):

1. Bên chủ động đóng gửi FIN, biểu thị mình không còn data cần gửi.
2. Bên bị động đóng trả lời ACK, chuyển sang trạng thái `CLOSE_WAIT`, nhưng vẫn có thể tiếp tục gửi data còn lại.
3. Sau khi gửi xong data, bên bị động đóng cũng gửi FIN.
4. Bên chủ động đóng trả lời ACK cuối cùng, chuyển sang trạng thái `TIME_WAIT`, chờ 2MSL rồi đóng hoàn toàn.

Sự tồn tại của trạng thái `TIME_WAIT` nhằm bảo đảm ACK cuối cùng có thể đến đầu bên kia, đồng thời để các packet cũ còn sót lại trên network tiêu biến, tránh gây nhiễu connection mới tiếp theo.

> Để biết về TCP four-way handshake, ảnh hưởng của TIME_WAIT, điều tra tình trạng CLOSE_WAIT tích tụ, v.v., bạn có thể tham khảo [TCP three-way handshake và four-way handshake (Transport Layer)](https://javaguide.cn/cs-basics/network/tcp-connection-and-disconnection.html).

## Tổng kết quy trình đầy đủ

Kết nối các bước trên lại, quy trình truy cập đầy đủ có thể khái quát như sau:

1. **Nhập URL** → Trình duyệt phân tích các phần của URL, kiểm tra HTTP cache; nếu cần network request thì kiểm tra tiếp file hosts.
2. **DNS resolution** → Lần lượt tra cứu browser cache, operating system cache, local DNS server; khi cần thì iterative query qua root → TLD → authoritative server để lấy target IP address.
3. **Thiết lập transport connection** → HTTP/1.1 và HTTP/2 thường thiết lập TCP connection; HTTP/3 thiết lập QUIC connection.
4. **Thiết lập secure channel (HTTPS)** → HTTPS dựa trên TCP tiếp tục thực hiện TLS handshake; HTTP/3 hoàn tất TLS 1.3 handshake trong quá trình thiết lập QUIC connection.
5. **HTTP request và response** → Trình duyệt gửi request, server xử lý và trả về HTML response.
6. **Data encapsulation và forwarding** → Request message được đóng gói từng lớp theo TCP → IP → link layer, truyền từng hop qua router, switch và các thiết bị trung gian khác đến server; response quay về trình duyệt theo chiều ngược lại.
7. **Trình duyệt render** → Phân tích HTML để xây dựng DOM tree, load sub-resource như CSS/JS/ảnh, xây dựng render tree, layout và paint page.
8. **Quản lý connection** → Tái sử dụng hoặc đóng TCP connection theo HTTP version và configuration.

Truy cập một trang web nhìn có vẻ đơn giản, nhưng thực tế xâu chuỗi gần như toàn bộ protocol cốt lõi của computer network. Hiểu theo quy trình này, các protocol như DNS, TCP, TLS, HTTP, IP, ARP không còn là những kiến thức rời rạc mà trở thành các khâu khác nhau trong một chain hoàn chỉnh.

## Tài liệu tham khảo

1. 《Mạng máy tính (ấn bản 7)》
2. 《HTTP qua hình minh họa》
3. [What really happens when you navigate to a URL](https://stackoverflow.com/questions/2092527/what-really-happens-when-you-navigate-to-a-url)
4. [How browsers work](https://web.dev/howbrowserswork/)

<!-- @include: @article-footer.snippet.md -->
