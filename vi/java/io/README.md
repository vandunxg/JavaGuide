---
title: "Chuyên đề Java IO: BIO, NIO, AIO, IO model và design pattern"
description: "Lộ trình học Java IO và NIO, bao quát BIO, NIO, AIO, blocking/non-blocking, synchronous/asynchronous, I/O multiplexing, mô hình Reactor và các design pattern trong IO."
category: Java
tag:
  - Java
  - Java IO
  - Java Interview Questions
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Java IO,Java NIO,BIO,NIO,AIO,IO model,I/O multiplexing,Reactor,Selector,Channel,Buffer,Java IO Interview Questions
---

Java IO là nền tảng quan trọng để hiểu việc đọc/ghi file, lập trình mạng, Netty, RPC framework và server hiệu năng cao. Khi học IO, bạn nên đồng thời hiểu Java API, IO model của hệ điều hành và các design pattern phổ biến để có thể kết nối BIO, NIO, AIO, Selector, Channel, Buffer và Reactor.

## Dành cho ai

- Backend developer muốn học Java IO/NIO một cách hệ thống.
- Người đang ôn các câu hỏi phỏng vấn liên quan đến BIO, NIO, AIO, I/O multiplexing và Reactor.
- Người muốn học tiếp các framework và thành phần giao tiếp mạng như Netty, RPC, message queue và database driver.
- Engineer dễ nhầm lẫn giữa blocking/non-blocking, synchronous/asynchronous và các thành phần Selector, Channel, Buffer.

## Trọng tâm học

- Hệ thống IO stream của Java, byte stream, character stream, buffered stream và các thao tác file thường gặp.
- Ứng dụng của các design pattern như Decorator pattern và Adapter pattern trong IO.
- Khác biệt giữa các model, trường hợp sử dụng và ưu/nhược điểm của BIO, NIO, AIO.
- Synchronous/asynchronous, blocking/non-blocking, I/O multiplexing, Reactor và Proactor.
- Mối quan hệ phối hợp giữa Buffer, Channel và Selector, cùng vai trò của chúng trong lập trình mạng.

## Thứ tự đọc đề xuất

1. [Tổng hợp kiến thức cơ bản về Java IO](./io-basis.md): trước hết nắm hệ thống IO stream, các class thường dùng và kiến thức cơ bản về đọc ghi file.
2. [Tổng hợp design pattern trong Java IO](./io-design-patterns.md): hiểu cách Decorator pattern, Adapter pattern và các design pattern khác được áp dụng trong Java IO API.
3. [Giải thích chi tiết Java IO model](./io-model.md): làm rõ BIO, NIO, AIO, synchronous/asynchronous, blocking/non-blocking và I/O multiplexing.
4. [Tổng hợp kiến thức cốt lõi về Java NIO](./nio-basis.md): học sâu về Buffer, Channel, Selector và NIO programming model.

## Bài viết cốt lõi

- [Tổng hợp kiến thức cơ bản về Java IO](./io-basis.md): giới thiệu một cách hệ thống về byte stream, character stream, buffered stream, random access file và các class IO thường gặp.
- [Tổng hợp design pattern trong Java IO](./io-design-patterns.md): giải thích ứng dụng của Decorator pattern, Adapter pattern và các design pattern khác trong IO.
- [Giải thích chi tiết Java IO model](./io-model.md): phân biệt BIO, NIO, AIO, synchronous/asynchronous, blocking/non-blocking và I/O multiplexing.
- [Tổng hợp kiến thức cốt lõi về Java NIO](./nio-basis.md): tìm hiểu Buffer, Channel, Selector, SelectionKey và lập trình server bằng NIO.

## Câu hỏi thường gặp

- Byte stream và character stream khác nhau như thế nào? Khi nào nên sử dụng buffered stream?
- Vì sao Java IO sử dụng Decorator pattern rộng rãi?
- BIO, NIO, AIO khác nhau như thế nào?
- Synchronous và asynchronous, blocking và non-blocking lần lượt có nghĩa là gì?
- I/O multiplexing giải quyết vấn đề gì?
- `select`, `poll`, `epoll` khác nhau như thế nào?
- Reactor model là gì? Nó khác Proactor như thế nào?
- Trong NIO, Buffer, Channel và Selector lần lượt đảm nhiệm vai trò gì?
- Vì sao Netty được xây dựng dựa trên NIO thay vì sử dụng trực tiếp BIO truyền thống?

## Chuyên đề liên quan

- [Hệ thống kiến thức Java](../)
- [Chuyên đề Java concurrency](../concurrent/)
- [Chuyên đề JVM](../jvm/)
- [Mạng máy tính](../../cs-basics/network/)
- [Netty](../../system-design/framework/netty.md)

<!-- @include: @article-footer.snippet.md -->
