---
title: "Chuyên đề Computer Networking: mô hình phân tầng, HTTP, HTTPS, DNS, TCP, UDP, ARP và NAT"
description: Lộ trình học và câu hỏi phỏng vấn Computer Networking, bao quát mô hình phân tầng OSI/TCP-IP, HTTP, HTTPS, DNS, TCP, UDP, ARP, NAT, network security và các câu hỏi phỏng vấn thường gặp.
category: Computer Fundamentals
tag:
  - Computer Networking
  - TCP/IP
  - HTTP
sidebar: false
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Computer Networking,câu hỏi phỏng vấn Computer Networking,mô hình bảy tầng OSI,TCP/IP,HTTP,HTTPS,DNS,TCP,UDP,ARP,NAT,câu hỏi phỏng vấn backend
---

Chuyên đề **Computer Networking** này dành cho việc học backend và ôn tập phỏng vấn, sắp xếp các bài viết về network trên site theo thứ tự “mô hình phân tầng -> application layer -> transport layer -> network layer -> security”.

## Dành cho ai

- Backend developer đang học Computer Networking một cách có hệ thống.
- Người chuẩn bị câu hỏi phỏng vấn Computer Networking khi ứng tuyển fresher hoặc experienced, tại các công ty vừa và lớn.
- Người đọc chỉ học thuộc rời rạc các kiến thức như HTTP, HTTPS, TCP, DNS, Socket.
- Kỹ sư muốn liên hệ kiến thức network với RPC, gateway, load balancing và system design.

## Trọng tâm học

- Giá trị cốt lõi của network layering là tách một vấn đề communication phức tạp; mỗi layer chỉ xử lý trách nhiệm của mình.
- HTTP, HTTPS, DNS là những kiến thức application layer được sử dụng thường xuyên nhất trong phát triển backend.
- Các điểm thường được hỏi về TCP tập trung ở connection management, reliable transmission, congestion control, TIME_WAIT và Keepalive.
- Kiến thức network layer như ARP, NAT giúp hiểu communication trong LAN, truy cập mạng nội bộ và mạng bên ngoài, cũng như troubleshooting.
- Trong phỏng vấn, cần có thể kết hợp một request hoàn chỉnh để nối protocol, connection, encryption, resolution và transmission thành một chuỗi.

## Thứ tự đọc đề xuất

1. [Tổng hợp câu hỏi phỏng vấn Computer Networking thường gặp (phần 1)](./other-network-questions.md) và [Tổng hợp câu hỏi phỏng vấn Computer Networking thường gặp (phần 2)](./other-network-questions2.md): trước tiên xây dựng danh sách các vấn đề thường gặp.
2. [Giải thích chi tiết mô hình bảy tầng OSI và mô hình bốn tầng TCP/IP](./osi-and-tcp-ip-model.md): hiểu network layering và trách nhiệm của từng layer.
3. [Điều gì thực sự xảy ra từ lúc nhập URL đến khi trang được hiển thị?](./the-whole-process-of-accessing-web-pages.md): dùng một chuỗi hoàn chỉnh để liên kết DNS, TCP, HTTP và quá trình xử lý của browser.
4. [HTTP vs HTTPS](./http-vs-https.md), [RSA và ECDHE trong HTTPS handshake](./https-rsa-vs-ecdhe.md), [Tổng hợp HTTP status code thường gặp](./http-status-codes.md): bổ sung các vấn đề thường gặp ở application layer.
5. [TCP three-way handshake và four-way termination](./tcp-connection-and-disconnection.md), [Đảm bảo tính tin cậy khi truyền dữ liệu TCP](./tcp-reliability-guarantee.md), [Giải thích chi tiết TCP TIME_WAIT](./tcp-time-wait.md): tập trung nắm vững TCP.

## Bài viết cốt lõi

### Tổng quan và nền tảng

- [Tổng hợp câu hỏi phỏng vấn Computer Networking thường gặp (phần 1)](./other-network-questions.md): bao quát các vấn đề nền tảng như network model, HTTP, HTTPS, DNS.
- [Tổng hợp câu hỏi phỏng vấn Computer Networking thường gặp (phần 2)](./other-network-questions2.md): tiếp tục hệ thống hóa các vấn đề thường gặp về TCP, UDP, Socket và network security.
- [Giải thích chi tiết mô hình bảy tầng OSI và mô hình bốn tầng TCP/IP](./osi-and-tcp-ip-model.md): hiểu network model, protocol layering và quá trình data encapsulation.
- [Điều gì thực sự xảy ra từ lúc nhập URL đến khi trang được hiển thị?](./the-whole-process-of-accessing-web-pages.md): liên kết các kiến thức network thường gặp bằng một request.

### Application layer

- [Tổng hợp các application layer protocol thường gặp](./application-layer-protocol.md): hệ thống hóa các protocol như HTTP, WebSocket, SMTP, FTP, SSH, DNS.
- [HTTP vs HTTPS](./http-vs-https.md): hiểu HTTPS encryption, certificate, authentication và integrity protection.
- [RSA và ECDHE trong HTTPS handshake](./https-rsa-vs-ecdhe.md): phân biệt các phương thức key exchange và forward secrecy.
- [HTTP 1.0 vs HTTP 1.1](./http1.0-vs-http1.1.md): hiểu sự khác biệt về persistent connection, cache, Host header và các khía cạnh khác.
- [Tổng hợp HTTP status code thường gặp](./http-status-codes.md): nắm được ngữ nghĩa và trường hợp sử dụng của status code từ 1xx đến 5xx.
- [Giải thích chi tiết hệ thống tên miền DNS](./dns.md): hiểu domain resolution, recursive query, iterative query và cache.
- [Có HTTP rồi, tại sao vẫn cần RPC?](./http-vs-rpc.md): làm rõ mối quan hệ giữa HTTP và RPC ở các layer.

### Transport layer, network layer và security

- [TCP three-way handshake và four-way termination](./tcp-connection-and-disconnection.md): nắm được connection establishment, connection termination và các state quan trọng.
- [Đảm bảo tính tin cậy khi truyền dữ liệu TCP](./tcp-reliability-guarantee.md): hiểu sequence number, acknowledgment, retransmission, flow control và congestion control.
- [Giải thích chi tiết TCP TIME_WAIT](./tcp-time-wait.md): hiểu vai trò, ảnh hưởng và giới hạn tối ưu của TIME_WAIT.
- [TCP Keepalive và HTTP Keep-Alive khác nhau thế nào?](./tcp-keepalive-vs-http-keepalive.md): phân biệt keepalive ở transport layer và persistent connection ở application layer.
- [Tại sao TCP hướng theo byte stream còn UDP hướng theo datagram?](./tcp-byte-stream-udp-datagram.md): hiểu sự khác biệt về data boundary giữa TCP và UDP.
- [Giải thích chi tiết protocol ARP](./arp.md), [Giải thích chi tiết protocol NAT](./nat.md), [Tổng hợp các phương thức network attack thường gặp](./network-attack-means.md): bổ sung kiến thức cơ bản về network layer và security.

## Câu hỏi thường gặp

- Mô hình bảy tầng OSI và mô hình bốn tầng TCP/IP khác nhau thế nào?
- Điều gì xảy ra ở phần network từ lúc nhập URL đến khi trang được hiển thị?
- HTTP và HTTPS khác nhau thế nào? Quá trình HTTPS handshake diễn ra ra sao?
- Điểm khác biệt cốt lõi giữa HTTP 1.0, 1.1 và 2.0 là gì?
- Các HTTP status code thường gặp lần lượt biểu thị điều gì?
- Tại sao TCP three-way handshake và four-way termination không thể lược bỏ?
- TCP đảm bảo reliable transmission như thế nào? Congestion control và flow control khác nhau thế nào?
- Tại sao TIME_WAIT tồn tại? Làm thế nào để troubleshooting khi có quá nhiều TIME_WAIT?
- TCP Keepalive và HTTP Keep-Alive khác nhau thế nào?
- DNS, ARP và NAT lần lượt giải quyết vấn đề gì?

## Chuyên đề liên quan

- [Hệ thống kiến thức Computer Fundamentals](../)
- [Chuyên đề Operating System](../operating-system/)
- [Hệ thống kiến thức Distributed System](../../distributed-system/)
- [Chuyên đề RPC](../../distributed-system/rpc/)
- [Hệ thống kiến thức High-Performance](../../high-performance/)

<!-- @include: @article-footer.snippet.md -->
