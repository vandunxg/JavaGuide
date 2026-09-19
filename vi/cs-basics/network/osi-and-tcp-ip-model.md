---
title: Giải thích chi tiết mô hình 7 tầng OSI và 4 tầng TCP/IP
description: Giải thích chi tiết mô hình phân tầng và trách nhiệm của OSI và TCP/IP, đồng thời so sánh khác biệt và các đánh đổi kỹ thuật qua lịch sử và thực tiễn.
category: Kiến thức máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: OSI 7 tầng,TCP/IP 4 tầng,mô hình phân tầng,phân chia trách nhiệm,protocol stack,so sánh
---

Phân tầng mạng là bản đồ đầu tiên khi học mạng máy tính. Không có bản đồ này, các khái niệm HTTP, TCP, IP, Ethernet và DNS rất dễ bị xếp lẫn vào nhau, khiến ta không phân biệt được khái niệm nào phụ thuộc vào khái niệm nào và khái niệm nào chịu trách nhiệm cho việc gì.

Hai mô hình phân tầng thường gặp là mô hình 7 tầng OSI và mô hình 4 tầng TCP/IP. Mô hình trước phù hợp hơn để xây dựng framework khái niệm, còn mô hình sau gần với việc triển khai thực tế trên Internet hơn.

Bài viết này chủ yếu trả lời một số câu hỏi:

1. Mỗi tầng trong mô hình OSI 7 tầng làm những gì?
2. Mô hình TCP/IP 4 tầng tương ứng với mô hình OSI 7 tầng như thế nào?
3. Vì sao mô hình OSI hoàn chỉnh về mặt lý thuyết nhưng trên thực tế không trở thành implementation chủ đạo của Internet?
4. Khi học một network protocol cụ thể, vì sao cần biết trước protocol đó nằm ở tầng nào?

## Mô hình 7 tầng OSI

**Mô hình 7 tầng OSI** là mô hình phân tầng mạng do Tổ chức Tiêu chuẩn hóa Quốc tế đề xuất. Cấu trúc tổng thể và chức năng mỗi tầng cung cấp được thể hiện trong hình dưới đây:

![Phân chia chức năng của từng tầng trong mô hình OSI 7 tầng](https://oss.javaguide.cn/github/javaguide/cs-basics/network/osi-7-model.png)

Mỗi tầng chỉ tập trung làm một việc, đồng thời đều cần sử dụng chức năng do tầng bên dưới cung cấp. Chẳng hạn, transport layer cần sử dụng chức năng routing và addressing do network layer cung cấp, nhờ đó biết cần truyền dữ liệu đến đâu.

**Kiến trúc 7 tầng của OSI có khái niệm rõ ràng và lý thuyết hoàn chỉnh, nhưng khá phức tạp, thiếu tính thực tiễn; hơn nữa, một số chức năng còn lặp lại ở nhiều tầng.**

Hình trên có thể khá trừu tượng, vì vậy hãy xem thêm một hình ảnh sinh động hơn. Tôi bắt gặp hình ảnh này trên một website nước ngoài và thấy nó rất hay!

![Mô hình OSI 7 tầng 2](https://oss.javaguide.cn/github/javaguide/osi七层模型2.png)

**Đã có mô hình 7 tầng OSI mạnh như vậy, vì sao nó lại không vượt qua được mô hình 4 tầng TCP/IP?**

Đúng là vào thời điểm đó, mô hình 7 tầng OSI nhận được sự ủng hộ của một số công ty lớn, thậm chí cả chính phủ một số quốc gia. Vậy vì sao nó lại thất bại trong bối cảnh đó? Theo tôi, chủ yếu có những nguyên nhân sau:

1. Các chuyên gia OSI thiếu kinh nghiệm thực tế và thiếu động lực thương mại khi hoàn thiện tiêu chuẩn OSI.
2. Các protocol của OSI quá phức tạp khi triển khai, hơn nữa hiệu suất vận hành rất thấp.
3. Chu kỳ xây dựng tiêu chuẩn OSI quá dài, khiến các thiết bị được sản xuất theo tiêu chuẩn OSI không thể kịp thời đi vào thị trường. (Đầu những năm 1990, mặc dù toàn bộ tiêu chuẩn quốc tế OSI đã được xây dựng, Internet dựa trên TCP/IP đã đi trước và vận hành thành công ở phạm vi khá lớn trên toàn cầu.)
4. Việc phân chia các tầng của OSI chưa thật hợp lý, một số chức năng lặp lại ở nhiều tầng.

Mặc dù mô hình 7 tầng OSI đã thất bại, nó vẫn cung cấp nhiều nền tảng lý thuyết hữu ích. Để hiểu rõ hơn về phân tầng mạng, việc học mô hình 7 tầng OSI vẫn rất cần thiết.

Cuối cùng, hãy xem thêm một hình tổng hợp rất hay về mô hình 7 tầng OSI!

![Mối quan hệ tương ứng giữa các tầng của mô hình OSI 7 tầng và TCP/IP 4 tầng](https://oss.javaguide.cn/github/javaguide/cs-basics/network/osi-model-detail.png)

## Mô hình 4 tầng TCP/IP

**Mô hình 4 tầng TCP/IP** hiện được sử dụng rộng rãi. Có thể xem mô hình TCP/IP là phiên bản rút gọn của mô hình 7 tầng OSI, gồm 4 tầng sau:

1. Application layer
2. Transport layer
3. Network layer
4. Network interface layer

Cần lưu ý rằng không thể ghép nối hoàn toàn chính xác mô hình 4 tầng TCP/IP với mô hình 7 tầng OSI. Tuy nhiên, có thể đơn giản hóa việc đối chiếu hai mô hình như hình dưới đây:

![Phân chia chức năng của từng tầng trong mô hình TCP/IP 4 tầng](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-ip-4-model.png)

### Tầng ứng dụng (Application layer)

**Application layer nằm trên transport layer, chủ yếu cung cấp dịch vụ trao đổi thông tin giữa các application trên hai thiết bị đầu cuối. Tầng này định nghĩa format trao đổi thông tin, sau đó message được chuyển cho transport layer để truyền đi.** Đơn vị dữ liệu được trao đổi ở application layer được gọi là message.

![Quá trình phối hợp của mô hình 5 tầng mạng trong một lần truyền dữ liệu](https://oss.javaguide.cn/github/javaguide/cs-basics/network/network-five-layer-sample-diagram.png)

Application layer protocol định nghĩa các quy tắc giao tiếp mạng. Những network application khác nhau cần các application layer protocol khác nhau. Trên Internet có rất nhiều application layer protocol, chẳng hạn HTTP hỗ trợ Web application, SMTP hỗ trợ email, v.v.

**Các protocol thường gặp ở application layer**:

![Các protocol thường gặp ở application layer](https://oss.javaguide.cn/github/javaguide/cs-basics/network/application-layer-protocol.png)

- **HTTP (Hypertext Transfer Protocol, protocol truyền hypertext)**: Dựa trên TCP, là protocol dùng để truyền hypertext và nội dung đa phương tiện, chủ yếu được thiết kế cho giao tiếp giữa Web browser và Web server. Khi dùng browser để duyệt web, trang web được tải thông qua HTTP request.
- **SMTP (Simple Mail Transfer Protocol, protocol gửi email)**: Dựa trên TCP, là protocol dùng để gửi email. Lưu ý ⚠️: SMTP chỉ chịu trách nhiệm gửi email, không chịu trách nhiệm nhận email. Để nhận email từ mail server, cần sử dụng protocol POP3 hoặc IMAP.
- **POP3/IMAP (protocol nhận email)**: Dựa trên TCP, cả hai đều chịu trách nhiệm nhận email. IMAP mới hơn POP3 và mạnh hơn về chức năng cũng như performance. IMAP hỗ trợ các chức năng nâng cao như tìm kiếm, đánh dấu, phân loại, lưu trữ email; đồng thời có thể đồng bộ trạng thái email giữa nhiều thiết bị. Hầu hết email client và server hiện đại đều hỗ trợ IMAP.
- **FTP (File Transfer Protocol, protocol truyền file)**: Dựa trên TCP, là protocol dùng để truyền file giữa các máy tính, có thể che giấu sự khác biệt giữa operating system và cách lưu trữ file. Lưu ý ⚠️: FTP là protocol không an toàn vì không mã hóa dữ liệu trong quá trình truyền. Khi truyền dữ liệu nhạy cảm, nên dùng protocol an toàn hơn như SFTP.
- **Telnet (protocol remote login)**: Dựa trên TCP, được dùng để login vào server khác thông qua terminal. Một trong những nhược điểm lớn nhất của Telnet là toàn bộ dữ liệu, bao gồm username và password, đều được gửi dưới dạng plaintext, tiềm ẩn rủi ro bảo mật. Đây là lý do chính khiến Telnet ngày nay hiếm khi được sử dụng, thay vào đó là network transfer protocol rất an toàn có tên SSH.
- **SSH (Secure Shell Protocol, protocol network transfer an toàn)**: Dựa trên TCP, thực hiện các tác vụ như truy cập an toàn và truyền file thông qua cơ chế mã hóa và authentication.
- **RTP (Real-time Transport Protocol, protocol truyền dữ liệu real-time)**: Thường dựa trên UDP nhưng cũng hỗ trợ TCP. RTP cung cấp chức năng truyền dữ liệu real-time end-to-end, nhưng không bao gồm resource reservation và không đảm bảo chất lượng truyền real-time; các chức năng này do WebRTC thực hiện.
- **DNS (Domain Name System, hệ thống quản lý domain name)**: Thường dựa trên UDP (port 53), được dùng để giải quyết việc mapping giữa domain name và IP address. Khi response quá lớn hoặc thực hiện zone transfer, protocol sẽ chuyển sang dùng TCP.

Để xem phần giới thiệu chi tiết về các protocol này, hãy đọc bài [Tổng hợp protocol thường gặp ở application layer (Application layer)](./application-layer-protocol.md).

### Tầng truyền tải (Transport layer)

**Nhiệm vụ chính của transport layer là cung cấp dịch vụ truyền dữ liệu tổng quát cho giao tiếp giữa các process của hai thiết bị đầu cuối.** Application process sử dụng dịch vụ này để truyền application layer message. “Tổng quát” nghĩa là dịch vụ này không dành riêng cho một network application cụ thể, mà nhiều application có thể dùng chung một transport layer service.

**Các protocol thường gặp ở transport layer**:

![Các protocol thường gặp ở transport layer](https://oss.javaguide.cn/github/javaguide/cs-basics/network/transport-layer-protocol.png)

- **TCP (Transmission Control Protocol, protocol điều khiển truyền dữ liệu)**: Cung cấp dịch vụ truyền dữ liệu **connection-oriented** và **reliable**.
- **UDP (User Datagram Protocol, protocol datagram người dùng)**: Cung cấp dịch vụ truyền dữ liệu **connectionless**, **best effort** (không đảm bảo độ tin cậy khi truyền dữ liệu), đơn giản và hiệu quả.

### Tầng mạng (Network layer)

**Network layer chịu trách nhiệm cung cấp dịch vụ giao tiếp cho các host khác nhau trên packet-switched network.** Khi gửi dữ liệu, network layer đóng gói segment hoặc datagram do transport layer tạo ra thành packet để truyền đi. Trong kiến trúc TCP/IP, vì network layer sử dụng IP protocol nên packet còn được gọi là IP datagram, gọi tắt là datagram.

⚠️ Lưu ý: **Không được nhầm “user datagram UDP” của transport layer với “IP datagram” của network layer.**

**Một nhiệm vụ khác của network layer là chọn route phù hợp, để packet do transport layer của source host truyền xuống có thể đi qua các router và đến destination host.**

Ở đây cần nhấn mạnh rằng chữ “network” trong “network layer” không còn chỉ một network cụ thể như cách chúng ta thường nói, mà là tên của tầng thứ ba trong mô hình kiến trúc mạng máy tính.

Internet được kết nối từ rất nhiều network dị thể (heterogeneous) thông qua router. Network layer protocol được Internet sử dụng là Internet Protocol connectionless và nhiều routing protocol. Vì vậy network layer của Internet còn được gọi là **internet layer** hoặc **IP layer**.

**Các protocol thường gặp ở network layer**:

![Các protocol thường gặp ở network layer](./images/network-model/nerwork-layer-protocol.png)

- **IP (Internet Protocol, protocol Internet)**: Một trong những protocol quan trọng nhất trong TCP/IP protocol. Chức năng chính là định nghĩa format của packet, routing và addressing packet để chúng có thể truyền qua các network và đến đúng destination. Hiện nay IP chủ yếu gồm hai phiên bản: IPv4 trước đây và IPv6 mới hơn. Cả hai protocol đều đang được sử dụng, nhưng IPv6 đã được đề xuất để thay thế IPv4.
- **ARP (Address Resolution Protocol, protocol phân giải address)**: ARP giải quyết việc chuyển đổi giữa network layer address và link layer address. Vì một IP datagram khi truyền vật lý luôn cần biết next hop (destination tiếp theo về mặt vật lý) cần đi đâu, trong khi IP address là logical address còn MAC address mới là physical address, nên ARP giải quyết một số vấn đề trong việc chuyển IP address thành MAC address.
- **ICMP (Internet Control Message Protocol, protocol Internet control message)**: Protocol dùng để truyền network status và error message, thường được dùng cho network diagnosis và troubleshooting. Chẳng hạn, công cụ Ping sử dụng ICMP để kiểm tra network connectivity.
- **NAT (Network Address Translation, protocol network address translation)**: Use case của NAT đúng như tên gọi là network address translation, được dùng trong quá trình chuyển đổi address từ internal network sang external network. Cụ thể, trong một subnet nhỏ (local area network, LAN), các host dùng IP address thuộc cùng một LAN; nhưng bên ngoài LAN đó, trong wide area network (WAN), cần một IP address thống nhất để định danh vị trí của LAN đó trên toàn Internet.
- **OSPF (Open Shortest Path First, protocol ưu tiên đường đi ngắn nhất mở)**: Một internal gateway protocol (Interior Gateway Protocol, IGP), đồng thời là một dynamic routing protocol được sử dụng rộng rãi. OSPF dựa trên link-state algorithm và xem xét các yếu tố như bandwidth, latency để chọn path tốt nhất.
- **RIP (Routing Information Protocol, protocol thông tin routing)**: Một internal gateway protocol (Interior Gateway Protocol, IGP), đồng thời là một dynamic routing protocol. RIP dựa trên distance-vector algorithm, sử dụng số hop cố định làm metric và chọn path có ít hop nhất làm path tốt nhất.
- **BGP (Border Gateway Protocol, protocol border gateway)**: Routing protocol dùng để trao đổi thông tin network layer reachability (Network Layer Reachability Information, NLRI) giữa các routing domain, có tính linh hoạt và khả năng mở rộng cao.

### Tầng giao diện mạng (Network interface layer)

Có thể xem network interface layer là sự kết hợp của data link layer và physical layer.

1. Data link layer thường được gọi tắt là link layer (việc truyền dữ liệu giữa hai host luôn diễn ra từng đoạn trên các link). **Chức năng của data link layer là lắp ráp IP datagram do network layer truyền xuống thành frame, rồi truyền frame trên link giữa hai node liền kề. Mỗi frame bao gồm data và các control information cần thiết như synchronization information, address information, error control, v.v.**
2. **Chức năng của physical layer là thực hiện việc truyền trong suốt bit stream giữa các computer node liền kề, đồng thời che giấu tối đa sự khác biệt giữa các transmission medium và physical device cụ thể.**

Các chức năng và protocol quan trọng của network interface layer được thể hiện trong hình dưới đây:

![Các chức năng và protocol quan trọng của network interface layer](https://oss.javaguide.cn/github/javaguide/cs-basics/network/network-interface-layer-protocol.png)

### Tổng kết

Hãy tóm tắt ngắn gọn các protocol và core technology trong mỗi tầng:

![Tổng quan protocol của các tầng TCP/IP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/network-protocol-overview.png)

**Application layer protocol**:

- HTTP (Hypertext Transfer Protocol, protocol truyền hypertext)
- SMTP (Simple Mail Transfer Protocol, protocol gửi email)
- POP3/IMAP (protocol nhận email)
- FTP (File Transfer Protocol, protocol truyền file)
- Telnet (protocol remote login)
- SSH (Secure Shell Protocol, protocol network transfer an toàn)
- RTP (Real-time Transport Protocol, protocol truyền dữ liệu real-time)
- DNS (Domain Name System, hệ thống quản lý domain name)
- ……

**Transport layer protocol**:

- TCP protocol
  - Cấu trúc segment
  - Truyền dữ liệu đáng tin cậy
  - Flow control
  - Congestion control
- UDP protocol
  - Cấu trúc datagram
  - RDT (Reliable Data Transfer Protocol)

**Network layer protocol**:

- IP (Internet Protocol, protocol Internet)
- ARP (Address Resolution Protocol, protocol phân giải address)
- ICMP protocol (protocol control message, dùng để gửi control message)
- NAT (Network Address Translation, protocol network address translation)
- OSPF (Open Shortest Path First, protocol ưu tiên đường đi ngắn nhất mở)
- RIP (Routing Information Protocol, protocol thông tin routing)
- BGP (Border Gateway Protocol, protocol border gateway)
- ……

**Network interface layer**:

- Kỹ thuật error detection
- Multiplexing protocol (kỹ thuật channel multiplexing)
- CSMA/CD protocol
- MAC protocol
- Ethernet technology
- ……

## Vì sao mạng được phân tầng?

Ở cuối bài viết này, tôi muốn trao đổi về câu hỏi: “Vì sao mạng cần phân tầng?”.

Khi nói đến phân tầng, trước tiên hãy lấy việc sử dụng framework để phát triển một chương trình backend làm ví dụ. Chúng ta thường chia system thành 3 tầng theo nguyên tắc mỗi tầng làm những việc khác nhau (system phức tạp có thể có nhiều tầng hơn):

1. Repository (thao tác database)
2. Service (thao tác nghiệp vụ)
3. Controller (trao đổi dữ liệu giữa frontend và backend)

**System phức tạp cần phân tầng vì mỗi tầng cần tập trung vào một nhóm công việc. Lý do mạng được phân tầng cũng giống như vậy: mỗi tầng chỉ tập trung làm một nhóm công việc.**

Vậy hãy quay lại câu hỏi: “Vì sao mạng cần phân tầng?”. Theo tôi, chủ yếu có 3 nguyên nhân:

1. **Các tầng độc lập với nhau**: Các tầng độc lập với nhau, mỗi tầng không cần quan tâm tầng khác được implement như thế nào, chỉ cần biết cách gọi chức năng mà tầng bên dưới cung cấp (có thể hiểu đơn giản là gọi interface)**. Điều này cũng giống với việc phân tầng system khi development.**
2. **Tăng tính linh hoạt tổng thể**: Mỗi tầng có thể sử dụng technology phù hợp nhất để implement. Chỉ cần đảm bảo chức năng cung cấp và quy tắc của interface được expose không thay đổi là được. **Điều này cũng tương ứng với nguyên tắc high cohesion, low coupling thường được yêu cầu khi phát triển system.**
3. **Chia nhỏ vấn đề lớn**: Phân tầng có thể chia một network problem phức tạp thành nhiều problem nhỏ có ranh giới tương đối rõ ràng và đơn giản để xử lý, giải quyết. Nhờ đó, computer network system phức tạp trở nên dễ design, implement và standardize hơn. **Điều này tương ứng với việc khi development, chúng ta thường chia nhỏ chức năng của system, rồi chia vấn đề phức tạp thành các vấn đề nhỏ hơn, dễ hiểu hơn; những vấn đề nhỏ này có boundary (định nghĩa rõ hơn về mục tiêu và interface).**

Tôi nhớ đến một câu nói rất nổi tiếng trong thế giới computer, xin chia sẻ ở đây:

> Mọi vấn đề trong lĩnh vực computer science đều có thể được giải quyết bằng cách thêm một intermediate layer gián tiếp. Toàn bộ computer system được design theo cấu trúc tầng nghiêm ngặt từ trên xuống dưới.

## Tài liệu tham khảo

- TCP/IP model vs OSI model：<https://fiberbit.com.tw/tcpip-model-vs-osi-model/>
- Data Encapsulation and the TCP/IP Protocol Stack：<https://docs.oracle.com/cd/E19683-01/806-4075/ipov-32/index.html>

<!-- @include: @article-footer.snippet.md -->
