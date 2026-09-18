---
title: Ping được, TCP nhất định kết nối được sao?
description: Giải thích sự khác biệt về tính liên thông giữa Ping/ICMP và TCP, lý do Ping được không có nghĩa là port có thể truy cập, đồng thời HTTPS cũng có thể bị chặn do SNI bị nhận diện.
category: Computer Basics
tag:
  - Computer Network
head:
  - - meta
    - name: keywords
      content: Ping,ICMP,TCP,three-way handshake,port connectivity,firewall,TLS,SNI,HTTPS
---

Ping được, TCP nhất định kết nối được sao? Tiểu G cho bạn kết luận trước: **Không phải**.

Lúc này bạn có thể sẽ thắc mắc: rõ ràng Ping được, tại sao TCP lại không hoạt động? Nói chính xác hơn, Ping được chỉ cho biết đường đi của ICMP Echo có thể khứ hồi theo policy hiện tại, không có nghĩa là TCP port của mục tiêu nhất định có thể truy cập.

Nói thật, tôi đã nghiêm túc học hết một lượt về network và cũng đọc khá nhiều tài liệu chuyên đề, nhưng lần đầu gặp câu hỏi này trong phỏng vấn, tôi thực sự hơi bối rối.

Câu trả lời thực ra rất đơn giản: **Ping sử dụng ICMP, còn TCP connection sử dụng TCP. Hai bên có thể đi qua cùng một đường mạng, nhưng thiết bị trung gian sẽ xử lý riêng theo loại protocol, port, trạng thái connection và security policy.**

ICMP hoạt động ở network layer, TCP hoạt động ở transport layer, trong protocol stack chúng hoàn toàn không nằm cùng một layer:

![ICMP và TCP nằm ở các layer khác nhau trong TCP/IP protocol stack](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-ip-4-model.png)

## Ping được chỉ cho biết ICMP có phản hồi

![Sự khác biệt giữa đường đi của ICMP và TCP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/can-ping-but-tcp-may-not-connect-icmp-and-tcp-path-differences.png)

Ping dựa trên ICMP (Internet Control Message Protocol, protocol điều khiển thông báo Internet), thực hiện kiểm tra bằng cách gửi và nhận các ICMP packet.

ICMP packet được chia thành hai loại: **query packet** (chẳng hạn Echo Request / Echo Reply được Ping sử dụng, lần lượt có type là 8 và 0) và **error packet** (báo cáo lỗi network, chẳng hạn Destination Unreachable).

Trong các system phổ biến, `ping` mặc định sử dụng Echo để kiểm tra: với IPv4 là ICMP Echo Request / Echo Reply, với IPv6 là ICMPv6 Echo Request / Echo Reply. Nó không xét port, cũng không quan tâm trên máy mục tiêu có service nào đang chạy hay không. Nếu là `tcping`, `hping` hoặc công cụ kiểm tra của cloud vendor thì cần xem loại kiểm tra cụ thể.

Bạn Ping được một máy thì đại khái chỉ có thể cho biết: phép kiểm tra ICMP đã nhận được phản hồi; đường đi của ICMP request và response này có thể thông suốt. Nếu trước target IP có NAT, load balancing, firewall hoặc Anycast routing, phản hồi ICMP có thể đến từ thiết bị trung gian hoặc một edge node, không thể chứng minh port của backend service có thể truy cập.

Nó cũng chỉ cho biết đến mức đó.

TCP có nhiều điều cần kiểm tra hơn. Ví dụ khi truy cập `example.com:443`, client trước tiên phải gửi `SYN`, server trả về `SYN-ACK`, sau đó client trả lại `ACK`. Khi hoàn tất ba bước này, TCP connection mới được xem là đã thiết lập; TLS, HTTP và application authentication ở phía sau vẫn có thể thất bại.

Bất kỳ bước nào ở giữa bị firewall drop, bị security group chặn, hoặc server hoàn toàn không có process listen trên port đó, TCP đều không thể kết nối.

Vì vậy, `ping` có thể dùng để phán đoán ban đầu, nhưng đừng dùng nó để trực tiếp chứng minh TCP không có vấn đề.

## ICMP được allow không có nghĩa TCP cũng được allow

Nhiều network device sẽ allow ICMP vì nó rất tiện cho việc vận hành. Máy có online hay không, latency có cao không, có packet loss rõ ràng hay không, chỉ cần `ping` là có thể xem sơ bộ.

Nhưng rule TCP thường chặt chẽ hơn. Server có thể chỉ mở `22`, `80`, `443`, còn database port, application port và debug port đều không allow.

Vì vậy sẽ xuất hiện tình huống như sau:

```bash
ping 10.0.0.10
# Kết nối được

nc -vz 10.0.0.10 8080
# timeout hoặc refused
```

Điều này không mâu thuẫn, vì ICMP và việc truy cập TCP port không khớp với cùng một bộ rule allow.

Ở đây cũng cần phân biệt hai loại thất bại:

1. `Connection timed out` thường cho biết `SYN` không nhận được phản hồi hợp lệ, có thể do firewall âm thầm drop, vấn đề về route hoặc đường về;
2. `Connection refused` thường cho biết target đã trả về `RST`, nguyên nhân phổ biến là port chưa listen hoặc policy chủ động refuse.

| Hiện tượng             | Giải thích đại khái                                                                             |
| ---------------------- | ----------------------------------------------------------------------------------------------- |
| `Connection refused`   | Thường đã nhận được `RST`, port chưa listen hoặc bị chủ động refuse                             |
| `Connection timed out` | `SYN` không nhận được phản hồi hợp lệ, có thể bị drop, route bất thường hoặc đường về có vấn đề |
| `No route to host`     | Vấn đề liên quan đến route của máy cục bộ, network lân cận hoặc ICMP unreachable                |
| TLS handshake thất bại | TCP có thể đã thông, tiếp tục kiểm tra SNI, certificate, protocol version hoặc proxy policy     |
| HTTP `4xx` / `5xx`     | TCP/TLS đã đi đến application layer, vấn đề nhiều khả năng nằm ở application hoặc gateway layer |

Một trường hợp còn trực tiếp hơn: máy còn hoạt động nhưng service không hoạt động. Host có thể phản hồi ICMP, nhưng Nginx chưa start, hoặc MySQL không listen trên address mà bạn đang kết nối. Ping đương nhiên thông, TCP đương nhiên thất bại.

## Khi có gateway ở giữa, càng không thể chỉ nhìn Ping

Đằng sau public IP thường không phải một real server mà là firewall, NAT gateway, load balancer hoặc security device.

Phản hồi ICMP bạn nhận được có thể đến từ thiết bị chứa VIP, edge node, hoặc cũng có thể được forward đến một backend nào đó; điều này phụ thuộc vào implementation của NAT, load balancing và firewall. Không thể đồng nhất phản hồi ICMP với việc backend application khả dụng.

Nhưng TCP request không đơn giản như vậy. Khi truy cập `public IP:443`, traffic có thể còn phải tiếp tục được forward đến backend machine. Port mapping chưa được cấu hình, backend service bị lỗi, health check thất bại hoặc security group không allow đều có thể khiến TCP bị treo.

Nhìn từ bên ngoài, chỉ có thể nói một câu: IP Ping được nhưng port lại không kết nối được.

Vì vậy khi thực sự troubleshooting, đừng chỉ gõ một lệnh `ping`. Nếu target là domain, trước tiên hãy xem DNS resolution result, đặc biệt là record A / AAAA, CDN routing và sự khác biệt giữa IPv4 / IPv6:

```bash
dig example.com A +short
dig example.com AAAA +short
```

Address family được application sử dụng và address family mà `ping` chọn không nhất thiết giống nhau, `curl` cũng có thể chọn giữa IPv6 / IPv4 theo Happy Eyeballs. Khi cần, có thể dùng `curl -4`, `curl -6` hoặc `curl --resolve` để cố định biến số.

Sau đó kiểm tra port:

```bash
nc -vz example.com 443
```

Nếu port thông, tiếp tục kiểm tra application layer:

```bash
curl -v https://example.com
```

Trong trường hợp HTTPS, cũng có thể xem trực tiếp TLS handshake:

```bash
openssl s_client -connect example.com:443 -servername example.com -brief
```

Khi nhiều domain dùng chung một IP, nên thêm `-servername`; nếu không, bạn có thể nhận certificate mặc định và đưa ra phán đoán sai.

Nếu vẫn chưa rõ, hãy capture packet để xác nhận layer:

```bash
tcpdump -nn host <ip> and port <port>
tcpdump -nn icmp
```

Chỉ nhìn thấy retransmission của `SYN` thường cho biết TCP layer vẫn chưa thông; nếu TCP đã established nhưng TLS bị treo, tiếp tục kiểm tra `ClientHello`, SNI, certificate, proxy và security policy.

## HTTPS cũng có thể bị treo ở SNI

Còn một điểm dễ phán đoán sai: cùng là `443`, cùng là HTTPS, cũng không có nghĩa là nhất định đi qua được.

Nội dung body của HTTPS được encrypt, nhưng trong `ClientHello` ở đầu TLS handshake thường sẽ có SNI (Server Name Indication, TLS extension). SNI dùng để cho server biết “tôi muốn truy cập domain nào”, nhờ đó cùng một IP có thể host nhiều HTTPS site.

Vấn đề là SNI truyền thống thường ở dạng plaintext.

![Quy trình TLS 1.2 ECDHE handshake](https://oss.javaguide.cn/github/javaguide/cs-basics/network/https-rsa-ecdhe-tls-1-2-ecdhe-rsa-handshake-process.png)

Từ hình trên có thể thấy TLS handshake được chia thành nhiều giai đoạn: ClientHello (mang theo SNI và các cipher suite được hỗ trợ) → ServerHello (chọn cipher suite) → certificate → key exchange → hai bên tính toán shared secret → handshake hoàn tất. Thiết bị trung gian không cần decrypt nội dung HTTPS, chỉ cần nhìn qua `ClientHello` là có thể biết bạn muốn truy cập domain nào và xử lý connection này theo domain policy.

Sau đó hệ sinh thái TLS đã đưa vào ECH (Encrypted ClientHello) để encrypt nhiều thông tin hơn trong `ClientHello`, bao gồm SNI thực. Tuy nhiên ECH có hoạt động hay không còn phụ thuộc vào client, server, DNS record `HTTPS` / SVCB và network environment; không thể mặc định rằng mọi HTTPS đều đã ẩn SNI.

Sau khi khớp policy, thiết bị trung gian có thể âm thầm drop, inject `RST`, terminate TLS, trả về trang chặn hoặc khiến connection bị treo ở giai đoạn TLS handshake. Biểu hiện cụ thể phụ thuộc vào implementation của firewall, proxy hoặc security device.

Khi capture packet các vấn đề loại này khá dễ gây nhầm lẫn: TCP three-way handshake có thể đã thành công, connection trông như đã được thiết lập, nhưng sau khi gửi `ClientHello` thì không có phản hồi hoặc nhanh chóng bị reset.

Vì vậy, “TCP thông” và “HTTPS có thể truy cập bình thường” cũng không phải cùng một chuyện. Vế trước kiểm tra three-way handshake, vế sau còn phải kiểm tra TLS handshake, SNI, certificate, proxy và security policy.

## Tóm tắt

`ping` kiểm tra ICMP; TCP cần kiểm tra target port có listen hay không, three-way handshake có hoàn tất được không, thiết bị trung gian có allow hay không; HTTPS còn có thể bị treo ở TLS handshake, đặc biệt là ở bước SNI.

Ngược lại cũng vậy: Ping không được không có nghĩa TCP nhất định không thông. Một số server hoặc cloud security group sẽ trực tiếp disable ICMP, nhưng business port vẫn hoạt động bình thường. Vì vậy khi troubleshooting, đừng kết luận chỉ bằng một command, hãy verify theo từng layer.

Tiểu G thường kiểm tra theo thứ tự này: nếu là domain, trước tiên xem DNS; sau đó dùng `ping` kiểm tra ICMP; tiếp theo dùng `nc` kiểm tra port; cuối cùng dùng `curl` hoặc `openssl s_client` kiểm tra HTTPS/TLS. Đừng để một lần `ping` quá sớm định tính vấn đề.

![Các layer troubleshooting connection HTTPS](https://oss.javaguide.cn/github/javaguide/cs-basics/network/can-ping-but-tcp-may-not-connect-https-connection-troubleshooting-layers.png)
