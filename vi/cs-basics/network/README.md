---
title: "Chuyên đề Mạng máy tính: mô hình phân tầng, HTTP, HTTPS, DNS, TCP, UDP, ARP và NAT"
description: Lộ trình học và câu hỏi phỏng vấn về Mạng máy tính, bao quát mô hình phân tầng OSI/TCP-IP, HTTP, HTTPS, DNS, TCP, UDP, ARP, NAT, network security và các câu hỏi phỏng vấn thường gặp.
category: Kiến thức máy tính cơ bản
tag:
  - Mạng máy tính
  - TCP/IP
  - HTTP
sidebar: false
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Mạng máy tính,câu hỏi phỏng vấn Mạng máy tính,mô hình bảy tầng OSI,TCP/IP,HTTP,HTTPS,DNS,TCP,UDP,ARP,NAT,câu hỏi phỏng vấn backend
---

Chuyên đề **Mạng máy tính** này dành cho việc học backend và ôn tập phỏng vấn, sắp xếp các bài viết về mạng trên site theo thứ tự “mô hình phân tầng -> application layer -> transport layer -> network layer -> security”.

## Dành cho ai

- Backend developer đang học Mạng máy tính một cách có hệ thống.
- Người chuẩn bị cho phỏng vấn network tại các công ty vừa và lớn, dù là ứng viên mới ra trường hay đã có kinh nghiệm.
- Người đọc chỉ học thuộc rời rạc các kiến thức như HTTP, HTTPS, TCP, DNS, Socket.
- Kỹ sư muốn liên hệ kiến thức mạng với RPC, gateway, load balancing và system design.

## Trọng tâm học

- Giá trị cốt lõi của phân tầng mạng là chia nhỏ một vấn đề giao tiếp phức tạp; mỗi tầng chỉ giải quyết phần việc của mình.
- HTTP, HTTPS, DNS là những kiến thức application layer được sử dụng thường xuyên nhất trong phát triển backend.
- Các điểm thường được hỏi về TCP tập trung ở quản lý kết nối, truyền tin cậy, congestion control, TIME_WAIT và Keepalive.
- Kiến thức network layer như ARP, NAT giúp hiểu giao tiếp trong LAN, truy cập mạng nội bộ và mạng bên ngoài, cũng như xử lý sự cố.
- Trong phỏng vấn, cần biết kết hợp một request hoàn chỉnh để xâu chuỗi protocol, connection, encryption, resolution và transmission.

## Thứ tự đọc đề xuất

1. [Tổng hợp câu hỏi phỏng vấn Mạng máy tính thường gặp (phần 1)](./other-network-questions.md) và [Tổng hợp câu hỏi phỏng vấn Mạng máy tính thường gặp (phần 2)](./other-network-questions2.md): trước tiên xây dựng danh sách câu hỏi trọng tâm.
2. [Giải thích chi tiết mô hình bảy tầng OSI và mô hình bốn tầng TCP/IP](./osi-and-tcp-ip-model.md): hiểu cách phân tầng mạng và trách nhiệm của từng tầng.
3. [Điều gì thực sự xảy ra từ lúc nhập URL đến khi trang được hiển thị?](./the-whole-process-of-accessing-web-pages.md): dùng một quy trình hoàn chỉnh để liên kết DNS, TCP, HTTP và quá trình xử lý của trình duyệt.
4. [HTTP vs HTTPS](./http-vs-https.md), [RSA và ECDHE trong HTTPS handshake](./https-rsa-vs-ecdhe.md), [Tổng hợp HTTP status code thường gặp](./http-status-codes.md): bổ sung các vấn đề trọng tâm ở application layer.
5. [TCP three-way handshake và four-way termination](./tcp-connection-and-disconnection.md), [Đảm bảo tính tin cậy khi truyền dữ liệu TCP](./tcp-reliability-guarantee.md), [Giải thích chi tiết TCP TIME_WAIT](./tcp-time-wait.md): tập trung nắm vững TCP.

## Bài viết cốt lõi

### Tổng quan và nền tảng

- [Tổng hợp câu hỏi phỏng vấn Mạng máy tính thường gặp (phần 1)](./other-network-questions.md): bao quát các vấn đề nền tảng như mô hình mạng, HTTP, HTTPS, DNS.
- [Tổng hợp câu hỏi phỏng vấn Mạng máy tính thường gặp (phần 2)](./other-network-questions2.md): tiếp tục hệ thống hóa các vấn đề thường gặp về TCP, UDP, Socket và network security.
- [Giải thích chi tiết mô hình bảy tầng OSI và mô hình bốn tầng TCP/IP](./osi-and-tcp-ip-model.md): hiểu mô hình mạng, phân tầng protocol và quá trình đóng gói dữ liệu.
- [Điều gì thực sự xảy ra từ lúc nhập URL đến khi trang được hiển thị?](./the-whole-process-of-accessing-web-pages.md): xâu chuỗi các kiến thức mạng thường gặp bằng một request.

### Application layer

- [Tổng hợp các protocol thường gặp ở application layer](./application-layer-protocol.md): hệ thống hóa các protocol như HTTP, WebSocket, SMTP, FTP, SSH, DNS.
- [HTTP vs HTTPS](./http-vs-https.md): hiểu HTTPS encryption, certificate, authentication và integrity protection.
- [RSA và ECDHE trong HTTPS handshake](./https-rsa-vs-ecdhe.md): phân biệt các phương thức key exchange và forward secrecy.
- [HTTP 1.0 vs HTTP 1.1](./http1.0-vs-http1.1.md): hiểu sự khác biệt về persistent connection, cache, Host header và các khía cạnh khác.
- [Tổng hợp HTTP status code thường gặp](./http-status-codes.md): nắm được ý nghĩa và trường hợp sử dụng của status code từ 1xx đến 5xx.
- [Giải thích chi tiết hệ thống tên miền DNS](./dns.md): hiểu domain resolution, recursive query, iterative query và cache.
- [Có HTTP rồi, tại sao vẫn cần RPC?](./http-vs-rpc.md): làm rõ mối quan hệ về tầng giữa HTTP và RPC.

### Transport layer, network layer và security

- [TCP three-way handshake và four-way termination](./tcp-connection-and-disconnection.md): nắm được việc thiết lập, ngắt connection và các state quan trọng.
- [Đảm bảo tính tin cậy khi truyền dữ liệu TCP](./tcp-reliability-guarantee.md): hiểu sequence number, acknowledgment, retransmission, flow control và congestion control.
- [Giải thích chi tiết TCP TIME_WAIT](./tcp-time-wait.md): hiểu vai trò, ảnh hưởng và giới hạn tối ưu hóa TIME_WAIT.
- [TCP Keepalive và HTTP Keep-Alive khác nhau thế nào?](./tcp-keepalive-vs-http-keepalive.md): phân biệt keepalive ở transport layer và persistent connection ở application layer.
- [Tại sao TCP là byte stream còn UDP là datagram?](./tcp-byte-stream-udp-datagram.md): hiểu sự khác biệt về ranh giới dữ liệu giữa TCP và UDP.
- [Giải thích chi tiết giao thức ARP](./arp.md), [Giải thích chi tiết giao thức NAT](./nat.md), [Tổng hợp các phương thức network attack thường gặp](./network-attack-means.md): bổ sung kiến thức cơ bản về network layer và security.

## Câu hỏi thường gặp

- Mô hình bảy tầng OSI và mô hình bốn tầng TCP/IP khác nhau thế nào?
- Từ lúc nhập URL đến khi trang được hiển thị, phần network diễn ra thế nào?
- HTTP và HTTPS khác nhau thế nào? Quá trình HTTPS handshake diễn ra ra sao?
- Điểm khác biệt cốt lõi giữa HTTP 1.0, 1.1 và 2.0 là gì?
- Các HTTP status code thường gặp lần lượt biểu thị điều gì?
- Tại sao TCP three-way handshake và four-way termination không thể thiếu?
- TCP đảm bảo truyền tin cậy thế nào? Congestion control và flow control khác nhau thế nào?
- Tại sao TIME_WAIT tồn tại? Làm thế nào để chẩn đoán khi có quá nhiều TIME_WAIT?
- TCP Keepalive và HTTP Keep-Alive khác nhau thế nào?
- DNS, ARP và NAT lần lượt giải quyết vấn đề gì?

## Chuyên đề liên quan

- [Hệ thống kiến thức máy tính cơ bản](../)
- [Chuyên đề Operating System](../operating-system/)
- [Hệ thống kiến thức Distributed System](../../distributed-system/)
- [Chuyên đề RPC](../../distributed-system/rpc/)
- [Hệ thống kiến thức High-Performance](../../high-performance/)

<!-- @include: @article-footer.snippet.md -->
