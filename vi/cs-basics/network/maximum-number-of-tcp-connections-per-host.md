---
title: Một host chỉ có thể duy trì tối đa 65535 TCP connection sao?
description: "Giải thích một host có thể duy trì bao nhiêu TCP connection từ các góc độ TCP four-tuple, ephemeral port, file descriptor, memory, TIME_WAIT và NAT."
category: Computer Basics
tag:
  - Computer Network
head:
  - - meta
    - name: keywords
      content: TCP connection count,65535,TCP four-tuple,TIME_WAIT,ephemeral port,file descriptor,NAT
---

Một host nhiều nhất chỉ có thể duy trì 65535 TCP connection sao? Tiểu G nói kết luận trước: **không phải**.

Con số `65535` xuất phát từ phạm vi port. Các field source port và destination port trong TCP header đều dài 16 bit, có thể biểu diễn `0~65535`, tổng cộng 2^16 = 65536 giá trị. **65535 là port number lớn nhất, không phải giới hạn số connection.**

Nhưng số TCP connection và số port không phải là một. Muốn hiểu rõ vấn đề này, trước hết cần biết TCP connection được định danh như thế nào.

## TCP connection được phân biệt bằng four-tuple

TCP connection không được định danh duy nhất bằng “local port”, mà bằng four-tuple:

```text
Source IP, source port, destination IP, destination port
```

Chỉ cần four-tuple khác nhau, kernel có thể nhận diện chúng là các connection khác nhau.

Để tránh nhầm lẫn, dưới đây thống nhất viết four-tuple từ góc nhìn **client khởi tạo connection**: `(client IP, client port, server IP, server port)`.

Giả sử server có IP là `192.168.1.100`, đang listen trên port `8080`:

![TCP connection được phân biệt bằng four-tuple và giới hạn thực tế](https://oss.javaguide.cn/github/javaguide/cs-basics/network/maximum-number-of-tcp-connections-per-host-tcp-four-tuple-and-server-connection.png)

- Client A (`10.0.0.1:50000`) kết nối đến → four-tuple `(10.0.0.1, 50000, 192.168.1.100, 8080)`
- Client A (`10.0.0.1:50001`) kết nối lần nữa → four-tuple `(10.0.0.1, 50001, 192.168.1.100, 8080)`
- Client B (`10.0.0.2:50000`) kết nối đến → four-tuple `(10.0.0.2, 50000, 192.168.1.100, 8080)`

Ba connection có server IP và port không đổi, nhưng vì client IP hoặc port khác nhau nên các four-tuple khác nhau, do đó đây là ba connection độc lập.

Có một điểm dễ nhầm: server port `8080` chỉ có **một listening socket**, nhưng sau mỗi lần `accept()`, kernel tạo một **connected socket** mới và phân biệt bằng four-tuple. Vì vậy, nhiều connection dùng chung một server port hoàn toàn không xung đột.

## Vì sao server có thể vượt quá 65535?

Giả sử Web service listen trên `192.168.1.100:443`, server IP và port cố định nhưng client IP và port thay đổi. Ví dụ, server port của `(10.0.0.1, 50001, 192.168.1.100, 443)` và `(10.0.0.2, 50001, 192.168.1.100, 443)` đều là 443, nhưng four-tuple khác nhau nên đây là hai TCP connection khác nhau.

Chỉ xét tổ hợp four-tuple IPv4, sau khi cố định server IP và port, về lý thuyết client IP có `2^32` khả năng, client port có `2^16` khả năng, nên số tổ hợp lý thuyết rất lớn.

Giới hạn thực tế đến từ resource và configuration.

## Giới hạn thực sự là gì?

**1. File descriptor (File Descriptor, FD) và memory.**

Trong Linux, socket cũng là file. Với application process, mỗi connection đã được thiết lập sau `accept()` thường tương ứng với một socket FD; **bản thân listening socket cũng chiếm một FD**. Connection chưa được `accept()` sẽ tạm thời nằm trong kernel queue, không nên đơn giản tính tất cả là FD mà application đang giữ.

Khi số file mà process có thể mở không đủ, lỗi thường gặp là `Too many open files`.

Mỗi TCP connection đều cần kernel duy trì socket, TCP control block, send buffer, receive buffer và các data khác. Khi connection idle, chi phí tương đối nhỏ; khi có data được gửi nhận, buffer và application object cũng tiếp tục chiếm memory.

Không nên học thuộc rằng “một connection chiếm bao nhiêu KB”. Giá trị này chịu ảnh hưởng của kernel version, socket option, buffer size và tình hình gửi nhận của business.

**2. Handshake queue và tốc độ accept.**

Linux thực tế duy trì hai queue:

- **SYN queue (half-connection queue)**: connection đã nhận SYN, gửi SYN-ACK nhưng chưa hoàn tất three-way handshake, chịu giới hạn của `tcp_max_syn_backlog`; kích thước thực tế còn được tính cùng `somaxconn` và `listen()` backlog.
- **accept queue (full-connection queue)**: connection đã hoàn tất handshake và đang chờ application `accept()`, giới hạn là `min(listen(fd, backlog), net.core.somaxconn)`.

Chúng ảnh hưởng đến **việc xếp hàng và loại bỏ trong giai đoạn thiết lập connection**, không phải giới hạn đơn giản của tổng số connection ESTABLISHED.

Khi half-connection queue bị tràn, Linux có thể bật cơ chế SYN Cookie: server mã hóa thông tin cần thiết vào sequence number của SYN-ACK, không lưu full half-connection state ở local; sau khi nhận ACK hợp lệ mới dựng lại thông tin connection. SYN Cookie là biện pháp bảo vệ, không phải biện pháp mở rộng capacity.

Khi full-connection queue bị tràn, hành vi phụ thuộc vào `tcp_abort_on_overflow`: với giá trị mặc định `0`, server drop ACK do client gửi để client retransmit, server có cơ hội retransmit SYN-ACK; với giá trị `1`, server reply RST trực tiếp để fail nhanh. Production environment thường giữ giá trị mặc định `0` để tránh từ chối nhầm connection hợp lệ. Có thể dùng `ss -ltn` để điều tra full-connection queue bị tràn: nếu Recv-Q trong thời gian dài gần Send-Q, nghĩa là accept chưa đủ kịp thời; cần kiểm tra application thread pool có bị block không hoặc backlog configuration có quá nhỏ không.

**3. CPU, NIC và năng lực xử lý business.**

Long connection idle chủ yếu kiểm tra memory, FD limit, kernel connection table và connection keepalive strategy; connection active còn tạo thêm áp lực từ system call, encryption/decryption, protocol parsing, thread scheduling và NIC interrupt.

## Vì sao client dễ chạm giới hạn port hơn?

![Bottleneck của client direct connection và NAT gateway](https://oss.javaguide.cn/github/javaguide/cs-basics/network/maximum-number-of-tcp-connections-per-host-client-and-nat-port-restriction.png)

Server không bị giới hạn 65535, nhưng khi client truy cập cùng một target, ephemeral port có thể cạn trước.

Ví dụ client cố định là `192.168.1.10`, liên tục kết nối đến `10.0.0.1:443`. Lúc này destination IP, destination port và source IP đều cố định, chỉ còn source port có thể thay đổi. Khi source port dùng hết, không thể tạo four-tuple mới.

Có thể xem ephemeral port range do Linux tự động cấp phát như sau:

```bash
sysctl net.ipv4.ip_local_port_range
```

Trên Mac có thể xem như sau:

![Xem ephemeral port range tự động cấp phát trên Mac](https://oss.javaguide.cn/github/javaguide/cs-basics/network/check-automatic-temporary-port-range-mac.jpg)

Nhiều Linux environment mặc định có ephemeral port range là `32768 60999`, khoảng 2,8 vạn port; giá trị thực tế lấy theo output của `sysctl net.ipv4.ip_local_port_range`, và không phải toàn bộ `0~65535` đều được tự động dùng làm ephemeral port.

Khi thấy `Cannot assign requested address` / `EADDRNOTAVAIL`, nhiều lần `connect` fail và target `IP:Port` tập trung, cần nghi ngờ ephemeral port đã cạn hoặc `TIME_WAIT` bị tích tụ.

## Tầng NAT gateway cũng có thể chạm giới hạn trước

Một tình huống khác dễ bị bỏ qua: nhiều machine trong internal network không truy cập Internet trực tiếp mà đi qua NAT gateway trước.

NAT chuyển đổi internal address thành public address. Ví dụ khi machine nội bộ `192.168.1.10:50000` truy cập external service, NAT có thể đổi thành `203.0.113.1:40000` và ghi lại mapping này ở local. Khi response packet quay về, NAT chuyển tiếp lại cho machine nội bộ ban đầu dựa trên mapping.

Nếu nhiều machine nội bộ dùng chung một public IP và tập trung truy cập **cùng một external `IP:Port`**, số source port public khả dụng ở phía NAT sẽ trở thành yếu tố giới hạn. Nếu target phân tán, không gian port reuse sẽ lớn hơn. Thiếu port chỉ là một loại vấn đề; connection tracking table, CPU và memory của NAT device cũng có thể chạm bottleneck trước.

Vì vậy, khi điều tra vấn đề connection count, đừng chỉ nhìn client và server, mà cũng phải kiểm tra NAT gateway ở giữa chain.

Các metric thường dùng để điều tra phía NAT gồm: tỷ lệ sử dụng NAT connection tracking table, tỷ lệ sử dụng SNAT port, số connection từ một public IP đến một target, cùng CPU, memory, packet loss và connection creation rate của NAT device. Nếu NAT thực sự trở thành bottleneck, có thể cân nhắc tăng public IP, tách outbound gateway hoặc thực hiện connection reuse.

## TIME_WAIT ảnh hưởng đến connection count như thế nào?

![TIME_WAIT chiếm local port và ảnh hưởng đến số connection có thể thiết lập](https://oss.javaguide.cn/github/javaguide/ai/context-engineering/how-time-wait-affects-number-of-connections.png)

Trong tình huống điển hình, **bên chủ động close connection sẽ vào `TIME_WAIT`** vì sau khi gửi ACK cuối cùng cần chờ một khoảng thời gian để ngăn việc ACK cuối cùng bị mất và old packet ảnh hưởng đến connection sau đó. (Trong simultaneous close, hai bên đều vào TIME_WAIT, nhưng phần lớn tình huống thường gặp hằng ngày là trường hợp trước.)

Vấn đề là `TIME_WAIT` khiến connection tương ứng không thể tùy ý reuse trong một khoảng thời gian. Với client tạo short connection tần suất cao đến cùng một target, ephemeral port khả dụng sẽ bị nhiều `TIME_WAIT` tiêu thụ, do đó dễ chạm port limit hơn.

Đây cũng là lý do high concurrency **được khuyến nghị ưu tiên dùng connection pool và HTTP keep-alive**, giảm việc tạo short connection từ gốc.

Nói đến connection reuse, nhiều người không phân biệt được TCP Keepalive và HTTP Keep-Alive, nhưng thực ra chúng giải quyết các vấn đề hoàn toàn khác nhau.

Nói đơn giản: HTTP Keep-Alive quản lý “một connection được dùng tối đa bao lâu, phục vụ bao nhiêu request”, còn TCP Keepalive quản lý “nếu lâu không có data, kiểm tra xem peer có biến mất không”. Hai cơ chế không ảnh hưởng lẫn nhau và cũng không thể thay thế nhau. Xem giải thích chi tiết trong bài viết này: [TCP Keepalive và HTTP Keep-Alive khác nhau như thế nào?](./tcp-keepalive-vs-http-keepalive.md).

Đối với kernel parameter, đừng thấy nhiều `TIME_WAIT` là vội sửa ngay.

`tcp_tw_reuse` cần được xem xét cùng kernel version, business scenario và bằng chứng thực tế về port exhaustion, không phù hợp để coi là mục tối ưu vạn năng. `tcp_tw_recycle` càng không nên đụng đến, vì đã bị gỡ từ Linux 4.12.

Cũng đừng nghĩ đến việc dọn TIME_WAIT. Đây không phải thứ bẩn cần dọn, mà là cơ chế bình thường trong TCP protocol.

Khi thấy số lượng `TIME_WAIT` lớn, phản ứng đầu tiên nên là quay lại business chain để tìm vấn đề: có phải liên tục tạo short connection không? Connection pool có hoạt động không? HTTP keep-alive đã bật chưa? Client có chủ động ngắt connection sau mỗi request không?

Một lỗi rất thường gặp trong production environment là **connection pool chưa được cấu hình đúng, cuối cùng làm cạn ephemeral port**.

Ví dụ:

- HTTP client chưa bật keep-alive và cũng không dùng connection pool, mỗi request đều tạo connection mới rồi đóng ngay, khiến `TIME_WAIT` nhanh chóng tích tụ.
- Cấu hình max connection của connection pool hoặc số connection trên mỗi target address quá nhỏ, khiến connection liên tục được tạo và hủy.
- DNS cuối cùng resolve về một IP duy nhất, target request quá tập trung, trong four-tuple về cơ bản chỉ còn source port thay đổi, dễ làm port bị lấp đầy hơn.

Khi điều tra loại vấn đề này, hãy ưu tiên sửa connection reuse. Sau khi xác nhận connection pool, keep-alive, timeout và close strategy đều không có vấn đề, mới cân nhắc mở rộng ephemeral port range hoặc tăng source IP. Đừng vừa bắt đầu đã sửa kernel parameter.

![Flow điều tra vấn đề TIME_WAIT và CLOSE_WAIT](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-time-wait-close-wait-troubleshooting-flowchart.png)

Khi điều tra, có thể dùng `ss -ant` để thống kê số lượng từng TCP state; dùng `ss -ant state time-wait | awk 'NR>1 {print $5}' | sort | uniq -c | sort -nr | head` để xem TIME_WAIT tập trung ở target nào; dùng `ss -ltn` để xem tình trạng tích tụ của accept queue trên listening socket. Nếu TIME_WAIT tập trung ở một remote service, hãy kiểm tra short connection và connection pool; nếu CLOSE_WAIT tập trung ở một local process, ưu tiên kiểm tra application code có close connection đúng cách hay không.

## Quay lại câu hỏi

Một host nhiều nhất chỉ có thể duy trì 65535 TCP connection sao?

Câu trả lời là: không thể hiểu như vậy.

`65535` tương ứng với port range, không phải giới hạn số TCP connection. TCP connection được phân biệt bằng four-tuple: source IP, source port, destination IP, destination port. Khi server listen trên cùng một port, chỉ cần client IP hoặc client port khác nhau thì connection có thể tiếp tục tăng.

Tuy nhiên, phân biệt được về mặt lý thuyết không có nghĩa machine chắc chắn chịu được. Số connection thực tế thường bị giới hạn bởi các resource như file descriptor, memory, CPU, NIC, application processing capability và handshake queue. Nếu client thường xuyên tạo short connection đến cùng một target, còn gặp áp lực từ ephemeral port và `TIME_WAIT`; nếu đi qua NAT ở giữa, còn phải xem NAT gateway có chịu được hay không.

Tiểu G tóm tắt lại thành một câu: **Số connection phía server chủ yếu phụ thuộc vào resource của machine, client kết nối đến cùng một target chủ yếu phụ thuộc vào ephemeral port, còn khi có NAT ở giữa thì phải xem thêm NAT gateway. `65535` chỉ là giới hạn của port number, không phải giới hạn của mọi TCP connection.**
