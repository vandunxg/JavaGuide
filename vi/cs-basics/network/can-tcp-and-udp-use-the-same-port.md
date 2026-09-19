---
title: TCP và UDP có thể sử dụng cùng một port không?
description: Giải thích rõ TCP và UDP có thể sử dụng cùng một port hay không, port space, quy tắc bind và các ví dụ thường gặp.
category: Computer Basics
tag:
  - Computer Networking
head:
  - - meta
    - name: keywords
      content: TCP,UDP,port,socket,bind,DNS 53,HTTP3,QUIC,UDP 443
---

Trong phỏng vấn, bạn thường gặp câu hỏi này: trên cùng một máy, TCP đã lắng nghe `8080`, vậy UDP có thể lắng nghe tiếp `8080` không?

Kết luận: **Có thể. Namespace bind port của TCP và UDP được phân biệt theo transport layer protocol, nên cùng một số port ở các protocol khác nhau không xung đột.** Một process lắng nghe `TCP/8080`, process khác lắng nghe `UDP/8080`, kernel sẽ phân phối chúng vào các protocol stack tương ứng.

## Port number do ai quản lý?

Port là số hiệu mà transport layer dùng để phân biệt các process ứng dụng. Địa chỉ IP xác định host, port number xác định application cụ thể trên host đó.

Header của TCP và UDP đều có trường source port và destination port, mỗi trường dài 16 bit (16 bits), nên phạm vi port number đều là `0~65535`. Tuy nhiên, port `0` thường có ý nghĩa đặc biệt trong API thực tế, chẳng hạn để hệ thống tự cấp temporary port, nên không phù hợp để xem là port lắng nghe của service thông thường.

**Cùng phạm vi số không có nghĩa là cùng đối tượng bind**. Khi đăng ký service, listening, packet capture, cấu hình firewall và security group rule, thường phải xem transport layer protocol cùng với port, chẳng hạn `TCP/53`, `UDP/53`, `TCP/443`, `UDP/443`.

`TCP/443` và `UDP/443` chỉ giống nhau về số, còn đường xử lý trong protocol stack khác nhau. Sau khi nhận IP packet, kernel trước tiên xem protocol identifier của IP layer: trong IPv4 là trường Protocol, còn trong IPv6 tương ứng là Next Header. Protocol number của TCP là `6`, của UDP là `17`. Trước khi phân phối theo port, kernel đã dựa trên protocol number để chuyển packet đến TCP hoặc UDP protocol stack tương ứng.

![Quy trình phân phối protocol của kernel](https://oss.javaguide.cn/github/javaguide/cs-basics/network/can-tcp-and-udp-use-the-same-port-kernel-protocol-dispatching-process.png)

TCP và UDP đều nằm ở transport layer nhưng khác nhau rất nhiều. Bảng dưới đây so sánh 8 khía cạnh để có cái nhìn tổng thể:

| Đặc tính                | TCP                                                                                                     | UDP                                             |
| ----------------------- | ------------------------------------------------------------------------------------------------------- | ----------------------------------------------- |
| **Tính kết nối**        | Connection-oriented (thiết lập connection bằng three-way handshake, giải phóng bằng four-way handshake) | Connectionless, gửi trực tiếp                   |
| **Độ tin cậy**          | Reliable (sequence number, ACK, retransmission, flow control, congestion control)                       | Unreliable, giao theo cơ chế best effort        |
| **Duy trì state**       | Stateful, duy trì thông tin connection                                                                  | Stateless, gửi xong không duy trì state         |
| **Hiệu suất truyền**    | Thấp hơn (chi phí thiết lập connection, xác nhận và retransmission lớn)                                 | Cao hơn (cấu trúc đơn giản, chi phí thấp)       |
| **Hình thức truyền**    | Byte stream-oriented, không giữ message boundary                                                        | Message-oriented, tự nhiên giữ message boundary |
| **Chi phí header**      | 20~60 byte                                                                                              | Cố định 8 byte                                  |
| **Mô hình giao tiếp**   | Point-to-point (unicast)                                                                                | Hỗ trợ cả unicast, multicast và broadcast       |
| **Ứng dụng thường gặp** | HTTP/HTTPS, FTP, SMTP, SSH                                                                              | DNS, DHCP, SNMP, TFTP, VoIP, video stream       |

Chính vì TCP và UDP là hai protocol độc lập ở transport layer, kernel mới tách chúng ra xử lý trước khi phân phối port.

## Vì sao không xung đột khi bind socket?

Chương trình server thường tạo socket trước, sau đó dùng `bind()` để gắn local IP và port. Một TCP socket bind `8080`, một UDP socket khác cũng bind `8080`, thông thường có thể cùng tồn tại. Khi xác định xung đột, kernel không chỉ xét số port mà còn xét protocol, local address và các thông tin khác.

Với TCP, một connection đã thiết lập thường có thể được định danh bằng four-tuple: source IP, source port, destination IP, destination port. Trong firewall, NAT, packet capture và traffic troubleshooting, người ta cũng thường thêm transport layer protocol vào, gọi là five-tuple:

```text
Protocol, source IP, source port, destination IP, destination port
```

Destination port của hai communication đều có thể là `8080`; chỉ cần protocol khác nhau thì kernel sẽ không xem chúng là cùng một communication. UDP không có state machine của connection như TCP, nhưng khi gửi và nhận data cũng mang theo source IP, source port, destination IP, destination port.

## Thử xác minh đơn giản

Có thể dùng `nc` để thử nhanh. Tuy nhiên, các implementation của `nc` trên các system khác nhau không hoàn toàn giống nhau, cách viết tham số `-l`, `-u` và port cũng có thể khác. Dưới đây là cách viết thường gặp của OpenBSD netcat; nếu lệnh báo lỗi, trước tiên có thể dùng `nc -h` để xem help của máy hiện tại.

Trước tiên khởi động TCP listener:

```bash
nc -l 8000
```

Sau đó khởi động UDP listener:

```bash
nc -u -l 8000
```

Hai lệnh có thể cùng tồn tại. Trên Linux, có thể kiểm tra thêm:

```bash
ss -tulnp | grep 8000
```

Thông thường sẽ thấy một listener `tcp` và một listener `udp`, số port giống nhau nhưng protocol khác nhau.

Nếu muốn tránh khác biệt về tham số của `nc`, cũng có thể dùng code để xác minh: trong Java, `ServerSocket(8000)` và `DatagramSocket(8000)` có thể được tạo đồng thời; trong Go, `net.Listen("tcp", ":8000")` và `net.ListenPacket("udp", ":8000")` cũng có thể cùng tồn tại. Nếu thử lắng nghe lần nữa bằng cùng protocol, thông thường sẽ thấy lỗi address đã được sử dụng.

## Khi nào xảy ra xung đột?

![Khi nào port xảy ra xung đột](https://oss.javaguide.cn/github/javaguide/cs-basics/network/when-does-tcp-conflict-occur.png)

TCP và UDP không xung đột với nhau không có nghĩa là có thể tùy ý bind lặp lại port.

Xung đột thường gặp hơn xảy ra trong **cùng một protocol**. Ví dụ, một process đã bind `0.0.0.0:8080`, thường sẽ bao phủ `8080` trên tất cả IPv4 address của máy; một process khác bind một IPv4 address cụ thể với `TCP/8080` thường sẽ xung đột. Tuy nhiên, hành vi cuối cùng còn chịu ảnh hưởng của thứ tự bind, `SO_REUSEADDR`, `SO_REUSEPORT` và implementation của operating system.

Nếu hai process bind các local IP khác nhau, cùng protocol và cùng port vẫn có thể hoạt động, chẳng hạn `192.168.1.10:8080` và `192.168.1.11:8080` đều là TCP.

Một điểm khác dễ bị bỏ qua: IPv6 wildcard address `[::]:8080` trong một số environment có thể đồng thời nhận IPv6 và IPv4-mapped address; `IPV6_V6ONLY` sẽ ảnh hưởng việc nó có xung đột với IPv4 socket hay không. Khi troubleshooting, có thể dùng `ss -tulnp` để đồng thời xem `0.0.0.0:port` và `[::]:port`.

`SO_REUSEADDR`, `SO_REUSEPORT` cũng thay đổi quy tắc bind, thường được dùng trong các trường hợp restart nhanh, lắng nghe bằng nhiều process và phân bổ tải. Ở đây, G khuyên bạn chỉ cần nhớ:

**TCP và UDP có thể lắng nghe cùng một số port vì khác protocol, không cần `SO_REUSEPORT`. `SO_REUSEADDR` / `SO_REUSEPORT` chủ yếu ảnh hưởng việc reuse address và port, restart nhanh và lắng nghe bằng nhiều process trong cùng một protocol; việc có được phép hay không và phân luồng như thế nào còn phụ thuộc operating system và loại socket cụ thể.**

## Hai ví dụ thực tế

### Vì sao DNS đồng thời sử dụng TCP/UDP 53?

![Ví dụ thực tế DNS và HTTP/3 đồng thời sử dụng port TCP và UDP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/can-tcp-and-udp-use-the-same-port-practical-application-example.png)

DNS là ví dụ kinh điển nhất. Trong IANA registry, service `domain` đồng thời đăng ký `TCP/53` và `UDP/53`; DNS service thực tế cũng thường đồng thời lắng nghe hai port này.

Truy vấn domain hằng ngày thường sử dụng UDP vì query và response khá nhỏ, UDP không cần thiết lập connection nên nhanh. Tuy nhiên, trong các trường hợp sau sẽ chuyển sang TCP: UDP response bị cắt ngắn (vị trí flag `TC` trong DNS header được đặt thành 1, thường xảy ra khi response vượt quá giới hạn độ dài của UDP), zone transfer (Zone Transfer, cần reliable transmission để bảo đảm data integrity), hoặc DNSSEC response quá lớn. Đây không phải là “`UDP/53` đã bị chiếm nên `TCP/53` không thể dùng”, mà là DNS vốn có thể đồng thời sử dụng `53` của hai protocol.

### HTTPS và port 443 của HTTP/3 cũng tương tự

HTTPS truyền thống thường là HTTP/1.1 hoặc HTTP/2 over TLS over TCP, mặc định sử dụng `TCP/443`. HTTP/3 chạy trên QUIC, còn QUIC dựa trên UDP. Browser thường biết server hỗ trợ HTTP/3 thông qua `Alt-Svc` hoặc DNS record `HTTPS`, sau đó thử thiết lập QUIC connection; cách deploy thường gặp là đồng thời mở `TCP/443` và `UDP/443`.

![Implementation của HTTP/3 protocol stack](https://oss.javaguide.cn/github/javaguide/cs-basics/network/http-3-implementation.png)

Điều này không xung đột với `TCP/443` hiện có. Một server hoàn toàn có thể đồng thời cung cấp:

```text
HTTPS (HTTP/1.1, HTTP/2) -> TCP/443
HTTP/3                  -> UDP/443
```

Nhìn từ bên ngoài đều là `443`, nhưng nhìn từ protocol stack thì là hai đường riêng.

Trong production cũng cần chú ý: nếu chỉ cho phép `TCP/443`, HTTP/1.1 và HTTP/2 có thể đều hoạt động bình thường, nhưng HTTP/3 sẽ không hoạt động. Cần kiểm tra riêng `TCP/443` và `UDP/443` trên cloud security group, load balancer, Nginx / gateway và host firewall, sau đó dùng `curl --http3` hoặc browser developer tools để xác nhận protocol đã thực sự chuyển sang HTTP/3 hay chưa.

## Trả lời thế nào trong phỏng vấn?

TCP và UDP có thể sử dụng cùng một số port vì chúng là các protocol khác nhau ở transport layer; kernel trước tiên phân phối theo IP protocol number đến TCP stack hoặc UDP stack, sau đó tìm socket theo address và port trong từng protocol stack tương ứng, vì vậy `TCP/8080` và `UDP/8080` có thể cùng tồn tại.

Xung đột thực sự dễ xảy ra là bind trong cùng một protocol, chẳng hạn hai TCP service thường không thể đồng thời lắng nghe cùng một local IP và port; khi đó mới liên quan đến các socket reuse options như `SO_REUSEADDR`, `SO_REUSEPORT`. Chỉ cần nhớ hai ví dụ là đủ: DNS đồng thời sử dụng `UDP/53` và `TCP/53`; HTTP/3 thường được deploy với `UDP/443`, có thể cùng tồn tại với `TCP/443` của HTTPS truyền thống.
