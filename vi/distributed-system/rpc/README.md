---
title: "Chuyên đề RPC (Remote Procedure Call): nguyên lý, quy trình gọi, serialization, service discovery, Dubbo và gRPC"
description: Lộ trình học RPC và remote call cho phỏng vấn, bao quát nguyên lý RPC, quy trình gọi, dynamic proxy, serialization, network transmission, service discovery, load balancing, Dubbo, gRPC và so sánh với HTTP.
category: Phân tán
tag:
  - RPC
  - Dubbo
  - Phân tán
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: RPC,nguyên lý RPC,RPC framework,Dubbo,gRPC,Thrift,remote procedure call,service discovery,dynamic proxy,serialization protocol,load balancing,khác biệt HTTP và RPC,phỏng vấn backend
---

RPC (Remote Procedure Call, gọi thủ tục từ xa) là một trong những cách gọi service phổ biến nhất trong distributed system. RPC hướng tới việc giúp gọi service từ xa tự nhiên như gọi local method, đồng thời đóng gói các chi tiết phức tạp như network communication, serialization, service discovery, load balancing, timeout retry và fault-tolerant governance.

## Dành cho ai

- Backend developer muốn hiểu nguyên lý hoạt động của RPC framework.
- Người đang chuẩn bị câu hỏi phỏng vấn liên quan đến RPC, Dubbo và microservice call.
- Người đã dùng Feign, Dubbo, gRPC nhưng chưa nắm chắc quy trình gọi bên trong và service governance.
- Engineer cần quyết định service call nội bộ nên dùng HTTP hay RPC.

## Trọng tâm học

- RPC là mô hình remote call, không phải một protocol cụ thể.
- Một RPC call thường trải qua các bước như dynamic proxy, serialization, network transmission, service discovery, load balancing và deserialization kết quả.
- Dubbo, gRPC, Thrift và các framework khác có trọng tâm khác nhau; không thể chỉ dùng “performance tốt hơn” để khái quát RPC.
- HTTP và RPC không loại trừ lẫn nhau; gRPC là ví dụ điển hình về RPC chạy trên HTTP/2.
- Trong phỏng vấn, cần trả lời theo các lớp: “Vì sao cần RPC, RPC hoạt động thế nào, framework governance service ra sao, chọn framework thế nào”.

## Quan hệ với các bài viết khác về distributed system

RPC chủ yếu giải quyết vấn đề **các service gọi nhau thế nào**. RPC thường xuất hiện cùng [API gateway](../api-gateway.md), nhưng vị trí của chúng khác nhau: gateway phụ trách external traffic entry và governance thống nhất, còn RPC thường dùng cho các service gọi nhau bên trong.

RPC framework thường còn phụ thuộc vào service registration and discovery, load balancing, timeout retry và cluster fault tolerance. Muốn hoàn thiện chuỗi kiến thức này, có thể đọc chuyên đề này cùng [Giải thích chi tiết về distributed configuration center](../distributed-configuration-center.md) và [Chuyên đề ZooKeeper](../distributed-process-coordination/zookeeper/).

## Thứ tự đọc đề xuất

1. [Có HTTP protocol rồi, tại sao vẫn cần RPC?](../../cs-basics/network/http-vs-rpc.md): trước tiên làm rõ quan hệ giữa HTTP và RPC.
2. [Giải thích chi tiết về RPC (Remote Procedure Call)](./rpc-intro.md): sau đó hệ thống hóa quy trình gọi RPC, các component cốt lõi và cách chọn framework.
3. [Tổng hợp câu hỏi phỏng vấn Dubbo](./dubbo.md): cuối cùng đặt RPC vào một framework trưởng thành như Dubbo để xem service governance và thực tiễn production.

## Bài viết cốt lõi

- [Giải thích chi tiết về RPC (Remote Procedure Call)](./rpc-intro.md): tìm hiểu nguyên lý cơ bản của RPC từ các góc độ quy trình gọi, dynamic proxy, serialization protocol, network transmission và service discovery.
- [Tổng hợp câu hỏi phỏng vấn Dubbo](./dubbo.md): kết nối nguyên lý kiến trúc Dubbo, service exposure và reference, cơ chế mở rộng SPI, load balancing, cluster fault tolerance, registry center và các vấn đề production.
- [Có HTTP protocol rồi, tại sao vẫn cần RPC?](../../cs-basics/network/http-vs-rpc.md): giải thích quan hệ giữa HTTP và RPC, tránh hiểu đơn giản RPC là “một protocol cao cấp hơn HTTP”.

## Câu hỏi thường gặp

- Một RPC call hoàn chỉnh có quy trình thế nào?
- Tại sao RPC framework thường dùng dynamic proxy?
- Các serialization protocol phổ biến của RPC là gì, chọn thế nào?
- Quy trình service exposure và service reference của Dubbo diễn ra thế nào?
- SPI, load balancing và cluster fault tolerance của Dubbo được triển khai ra sao?
- HTTP và RPC khác nhau thế nào? Tại sao service call nội bộ thường dùng RPC?
- Tại sao gRPC được gọi là RPC, và tại sao nó dựa trên HTTP/2?
- Nên đánh giá những khía cạnh nào khi chọn HTTP hay RPC cho service call nội bộ?

## Chuyên đề liên quan

- [Hệ thống kiến thức về distributed system](../)
- [Giải thích chi tiết về API gateway](../api-gateway.md)
- [Tổng hợp câu hỏi phỏng vấn Spring Cloud Gateway](../spring-cloud-gateway-questions.md)
- [Computer network](../../cs-basics/network/)

<!-- @include: @article-footer.snippet.md -->
