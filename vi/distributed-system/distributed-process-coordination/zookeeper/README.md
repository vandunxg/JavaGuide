---
title: "Chuyên đề ZooKeeper: khái niệm cốt lõi, ZNode, Watcher, ZAB, triển khai cluster, Curator và distributed lock"
description: "Lộ trình học ZooKeeper và distributed coordination cho phỏng vấn, bao quát ZNode, loại node, Watcher, ACL, giao thức ZAB, Leader election, triển khai cluster, Curator và distributed lock."
category: Hệ thống phân tán
tag:
  - ZooKeeper
  - distributed coordination
  - distributed lock
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: ZooKeeper,ZooKeeper nhập môn,ZooKeeper nâng cao,ZNode,Watcher,giao thức ZAB,Leader election,Curator,distributed coordination,distributed lock,service registry,phỏng vấn backend
---

ZooKeeper là một thành phần distributed coordination kinh điển, thường được dùng làm service registry, quản lý cấu hình, distributed lock, Leader election và quản lý metadata của cluster. Khi học ZooKeeper, bạn có thể nắm mô hình dữ liệu và Watcher trước, sau đó tìm hiểu giao thức ZAB, Leader election và thực tiễn triển khai.

## Dành cho ai

- Backend developer muốn học ZooKeeper một cách hệ thống.
- Người chuẩn bị câu hỏi phỏng vấn về ZooKeeper, service registry, distributed lock và distributed coordination.
- Người đã dùng ZooKeeper hoặc Curator nhưng chưa thật thành thạo ZNode, Watcher, ZAB và Leader election.
- Kỹ sư cần so sánh ZooKeeper, Eureka, Nacos và các service registry, thành phần coordination khác trong triển khai thực tế.

## Trọng tâm học

- Vì sao data model của ZooKeeper là ZNode dạng cây?
- Temporary node, sequential node và cơ chế Watcher phù hợp giải quyết những vấn đề nào?
- ZooKeeper xử lý message broadcast, đồng bộ master-slave và crash recovery thông qua giao thức ZAB như thế nào?
- Vì sao cluster ZooKeeper thường được khuyến nghị triển khai số node lẻ?
- Curator đơn giản hóa việc phát triển client ZooKeeper nguyên bản và triển khai distributed lock như thế nào?

## Quan hệ với các bài viết khác

[Giải thích chi tiết về distributed coordination](../../protocol/centralized-and-decentralized.md) trình bày trước các vấn đề chung như Leader, Quorum, split-brain và lease; chuyên đề ZooKeeper áp dụng các vấn đề này vào ZNode, Watcher, Session và ZAB.

[Giải thích chi tiết về giao thức ZAB](../../protocol/zab.md) chỉ trình bày atomic broadcast và crash recovery của ZooKeeper, không đi sâu vào ZNode, lệnh Curator và trường hợp sử dụng. Nếu muốn xem lệnh thực tế và mã client, bạn có thể đọc tiếp [Tutorial thực hành ZooKeeper](./zookeeper-in-action.md).

## Thứ tự đọc đề xuất

1. [Giải thích chi tiết về distributed coordination](../../protocol/centralized-and-decentralized.md): trước hết hiểu các vấn đề chung như distributed coordination, Leader, Quorum, split-brain và lease.
2. [Hướng dẫn nhập môn ZooKeeper](./zookeeper-intro.md): tiếp theo tìm hiểu ZNode, Watcher, ACL và các trường hợp sử dụng điển hình.
3. [Giải thích nâng cao về ZooKeeper](./zookeeper-plus.md): tiếp tục bổ sung kiến thức về ZAB, Leader election, chiến lược triển khai cluster, nguyên tắc số node lẻ và cơ chế session.
4. [Tutorial thực hành ZooKeeper](./zookeeper-in-action.md): cuối cùng thực hành với Docker, zkCli, four-letter command và Curator Java client.

## Bài viết cốt lõi

- [Hướng dẫn nhập môn ZooKeeper](./zookeeper-intro.md): giải thích các khái niệm cốt lõi của ZooKeeper, data model ZNode, loại node, cơ chế theo dõi Watcher, kiểm soát quyền ACL và các trường hợp sử dụng điển hình.
- [Giải thích nâng cao về ZooKeeper](./zookeeper-plus.md): tìm hiểu sâu về giao thức ZAB, Leader election, chiến lược triển khai cluster, nguyên tắc số node lẻ, session management và so sánh với các service registry như Eureka, Nacos.
- [Tutorial thực hành ZooKeeper](./zookeeper-in-action.md): bao quát cài đặt và triển khai bằng Docker, lệnh thường dùng của zkCli, four-letter command, thao tác CRUD bằng Curator Java client và ví dụ distributed lock.

## Câu hỏi thường gặp

- ZooKeeper phù hợp giải quyết những vấn đề distributed coordination nào?
- ZNode có những loại nào? Vì sao temporary sequential node thường được dùng cho distributed lock?
- Vì sao Watcher chỉ được kích hoạt một lần? Khi sử dụng cần lưu ý gì?
- Giao thức ZAB có quan hệ gì với Raft và Paxos?
- Vì sao cluster ZooKeeper thường được triển khai số node lẻ?
- ZooKeeper khác Eureka và Nacos như thế nào khi làm service registry?
- So với native ZooKeeper client, Curator giải quyết những khó khăn nào trong phát triển?

## Chuyên đề liên quan

- [Hệ thống kiến thức về distributed system](../../)
- [Chuyên đề về lý thuyết, thuật toán và protocol distributed](../../protocol/)
- [Giải thích chi tiết về distributed coordination](../../protocol/centralized-and-decentralized.md)
- [Giải thích chi tiết về giao thức ZAB](../../protocol/zab.md)
- [Giải thích chi tiết về các phương án triển khai distributed lock](../../distributed-lock-implementations.md)
- [Giải thích chi tiết về distributed configuration center](../../distributed-configuration-center.md)

<!-- @include: @article-footer.snippet.md -->
