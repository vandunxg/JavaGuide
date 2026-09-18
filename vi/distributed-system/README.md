---
title: "Hệ thống kiến thức hệ thống phân tán: nhập môn, lý thuyết và giao thức, RPC, gateway, lock, transaction và ID"
description: Lộ trình học và phỏng vấn hệ thống phân tán, bao quát nhập môn hệ thống phân tán, tập trung và phi tập trung, CAP, BASE, bài toán các vị tướng Byzantine, Paxos, Raft, ZAB, Gossip, consistent hashing, RPC, API Gateway, distributed lock, distributed transaction và ZooKeeper.
category: Distributed Systems
tag:
  - Distributed Systems
  - System Design
  - Backend Interview
sitemap:
  changefreq: weekly
  priority: 0.95
head:
  - - meta
    - name: keywords
      content: hệ thống phân tán,nhập môn hệ thống phân tán,câu hỏi phỏng vấn hệ thống phân tán,tập trung,phi tập trung,CAP,BASE,bài toán các vị tướng Byzantine,Paxos,Raft,ZAB,Gossip,RPC,Dubbo,API Gateway,distributed ID,distributed lock,distributed transaction,config center,ZooKeeper,phỏng vấn backend
---

<!-- @include: @small-advertisement.snippet.md -->

**Hệ thống kiến thức hệ thống phân tán** này hướng đến việc học backend, system design và ôn tập phỏng vấn, sắp xếp các bài viết về hệ thống phân tán trên site theo thứ tự “nhập môn hệ thống phân tán -> communication và invocation -> service governance -> consistency và coordination -> engineering practice”.

Nếu bạn mới bắt đầu học hệ thống phân tán, nên xem [Nhập môn hệ thống phân tán](./distributed-system-intro.md) trước để xây dựng nhận thức tổng thể; nếu thời gian có hạn, nên xem [Tổng hợp câu hỏi phỏng vấn hệ thống phân tán](./distributed-system-interview-questions.md) trước để nhanh chóng xây dựng danh sách câu hỏi thường gặp. Khi vị trí ứng tuyển yêu cầu rõ về microservice, bạn có thể xem thêm [Tổng hợp câu hỏi phỏng vấn microservice](./microservices-interview-questions.md) để kết nối các vấn đề về tách service, communication, data consistency và stability.

## Dành cho ai

- Backend developer đang học hệ thống phân tán một cách bài bản.
- Bạn đang chuẩn bị cho phỏng vấn backend khi tuyển dụng mới tốt nghiệp, đã đi làm hoặc tại các công ty lớn.
- Engineer muốn bổ sung lý thuyết phân tán, engineering practice và năng lực lựa chọn solution.
- Bạn đã viết business code nhưng chưa nắm vững nguyên lý bên trong của RPC, distributed lock, distributed transaction, config center và các thành phần tương tự.

## Trọng tâm học

- Vì sao hệ thống phân tán cần đánh đổi giữa consistency, availability, performance và complexity?
- CAP, BASE, Paxos, Raft, ZAB, Gossip và consistent hashing lần lượt giải quyết vấn đề gì?
- RPC, API Gateway, config center và service registry lần lượt đảm nhận trách nhiệm gì trong hệ thống microservice?
- Distributed ID, distributed lock và distributed transaction có những solution thường gặp và vấn đề nào trong business thực tế?
- Trong phỏng vấn, làm thế nào để kết nối “concept, principle, scenario, solution comparison, implementation experience” thành một câu trả lời hoàn chỉnh?

## Kết nối các bài viết này như thế nào?

Có thể đọc nhóm bài viết này theo 4 tuyến.

Tuyến thứ nhất là **tuyến lý thuyết**: [Nhập môn hệ thống phân tán](./distributed-system-intro.md) trước tiên giải thích vì sao hệ thống nhiều node trở nên phức tạp, [Giải thích chi tiết định lý CAP và lý thuyết BASE](./protocol/cap-and-base-theorem.md) giải thích khi partition xảy ra thì consistency và availability được đánh đổi ra sao, sau đó [Giải thích chi tiết distributed coordination](./protocol/centralized-and-decentralized.md) đặt Leader, Quorum, Lease, Fencing Token và Gossip trên cùng một tuyến.

Tuyến thứ hai là **tuyến invocation**: [Chuyên đề RPC remote procedure call](./rpc/) giải quyết cách các service gọi lẫn nhau, [Giải thích chi tiết API Gateway](./api-gateway.md) giải quyết cách external traffic đi vào hệ thống, còn [Tổng hợp câu hỏi phỏng vấn Spring Cloud Gateway](./spring-cloud-gateway-questions.md) mở rộng về routing, predicate, filter và rate limiting của Spring Cloud Gateway.

Tuyến thứ ba là **tuyến data consistency**: [Giải thích chi tiết solution tạo distributed ID](./distributed-id.md) giải quyết vấn đề global unique identifier, [Thực chiến thiết kế distributed ID](./distributed-id-design.md) đưa ID vào các business như order number, coupon và short URL, [Distributed lock](./distributed-lock.md) và [Distributed transaction](./distributed-transaction.md) tiếp tục xử lý mutual exclusion giữa các node và consistency của việc ghi giữa các service.

Tuyến thứ tư là **tuyến coordination component**: [Giải thích chi tiết distributed config center](./distributed-configuration-center.md) trình bày config publishing và client disaster recovery, [Chuyên đề ZooKeeper](./distributed-process-coordination/zookeeper/) trình bày concept, ZAB, session, Watcher và thực chiến Curator của coordination component.

## Thứ tự đọc đề xuất

1. [Nhập môn hệ thống phân tán](./distributed-system-intro.md): trước tiên hiểu hệ thống phân tán là gì, và vì sao sau khi tách thành nhiều node lại phát sinh các vấn đề về communication, failure và consistency.
2. [Tổng hợp câu hỏi phỏng vấn hệ thống phân tán](./distributed-system-interview-questions.md): xây dựng danh sách câu hỏi thường gặp, biết những điểm nào thường được hỏi nhất trong phỏng vấn.
3. [Tổng hợp câu hỏi phỏng vấn microservice](./microservices-interview-questions.md): hiểu các vấn đề về tách service, service communication, data consistency, stability và migration.
4. [Giải thích chi tiết distributed coordination](./protocol/centralized-and-decentralized.md): hiểu mối quan hệ giữa Leader, Quorum, Gossip, Lease, split-brain và Fencing Token.
5. [Giải thích chi tiết định lý CAP và lý thuyết BASE](./protocol/cap-and-base-theorem.md): hiểu logic đánh đổi cốt lõi nhất của hệ thống phân tán.
6. [Giải thích chi tiết bài toán các vị tướng Byzantine](./protocol/byzantine-generals-problem.md): hiểu malicious node, failure assumption và ranh giới fault tolerance của BFT trong bài toán consensus.
7. [Giải thích chi tiết RPC remote procedure call](./rpc/rpc-intro.md): nắm được cách các service communication với nhau và những vấn đề engineering mà RPC framework giải quyết.
8. [Giải thích chi tiết solution tạo distributed ID](./distributed-id.md), [Nhập môn distributed lock](./distributed-lock.md), [Giải thích chi tiết solution distributed transaction](./distributed-transaction.md): bổ sung các engineering practice thường gặp.
9. [Hướng dẫn nhập môn ZooKeeper](./distributed-process-coordination/zookeeper/zookeeper-intro.md) và [Giải thích chi tiết distributed config center](./distributed-configuration-center.md): hiểu distributed coordination và config governance.

## Bài viết cốt lõi

### Nền tảng hệ thống phân tán và lý thuyết, giao thức

Phần này phù hợp để xây dựng nhận thức bên trong về hệ thống phân tán trước tiên, tập trung hiểu consistency, availability, partition tolerance, consensus algorithm và data distribution.

- [Nhập môn hệ thống phân tán](./distributed-system-intro.md): hiểu định nghĩa, quá trình tiến hóa architecture, đặc điểm điển hình, các hệ thống thường gặp và learning path của hệ thống phân tán.
- [Tổng hợp câu hỏi phỏng vấn microservice](./microservices-interview-questions.md): kết nối các vấn đề về tách service, RPC và message, data ownership, distributed transaction, fault tolerance, observability và migration.
- [Chuyên đề distributed theory, algorithm và protocol](./protocol/): đặt CAP, BASE, Paxos, Raft, ZAB, Gossip và consistent hashing trên cùng một tuyến học.
- [Giải thích chi tiết distributed coordination](./protocol/centralized-and-decentralized.md): kết nối Leader/Quorum, split-brain, Lease, Fencing Token và Gossip, hiểu hai vấn đề “ai đưa ra quyết định, state được lan truyền thế nào”.
- [Giải thích chi tiết định lý CAP và lý thuyết BASE](./protocol/cap-and-base-theorem.md): hiểu consistency, availability, partition tolerance và eventual consistency.
- [Giải thích chi tiết bài toán các vị tướng Byzantine](./protocol/byzantine-generals-problem.md): hiểu điểm khó của consensus trong bối cảnh malicious node, yêu cầu `3m + 1` node và BFT fault tolerance.
- [Giải thích chi tiết thuật toán Raft](./protocol/raft-algorithm.md): nhập môn Leader election và log replication bằng một consensus algorithm dễ hiểu hơn.
- [Giải thích chi tiết thuật toán Paxos](./protocol/paxos-algorithm.md): bổ sung role, flow và điểm khó của consensus algorithm kinh điển.
- [Giải thích chi tiết protocol ZAB](./protocol/zab.md): hiểu atomic broadcast, crash recovery và transaction log mechanism của ZooKeeper.
- [Giải thích chi tiết protocol Gossip](./protocol/gossip-protocol.md): hiểu information propagation và eventual consistency giữa các node quy mô lớn.
- [Giải thích chi tiết thuật toán consistent hashing](./protocol/consistent-hashing.md): hiểu vấn đề data distribution trong distributed cache, load balancing và database sharding.

### RPC và service invocation

RPC giải quyết complexity trong engineering của remote service invocation, bao gồm serialization, network transmission, service discovery, load balancing, timeout retry và service governance.

- [Chuyên đề RPC](./rpc/): từ RPC basics, Dubbo đến mối quan hệ giữa HTTP và RPC, xây dựng nhận thức đầy đủ về service invocation.
- [Giải thích chi tiết RPC remote procedure call](./rpc/rpc-intro.md): hiểu invocation flow, dynamic proxy, serialization, network transmission và lựa chọn framework của RPC.
- [Tổng hợp câu hỏi phỏng vấn Dubbo](./rpc/dubbo.md): kết nối architecture của Dubbo, service exposure và reference, SPI, load balancing và cluster fault tolerance.
- [Đã có protocol HTTP, vì sao vẫn cần RPC?](../cs-basics/network/http-vs-rpc.md): làm rõ mối quan hệ về layer và ranh giới lựa chọn giữa HTTP và RPC.

### API Gateway và traffic entry

API Gateway phụ trách unified access, routing forwarding, authentication and authorization, rate limiting and circuit breaking, canary release và cross-origin handling; đây là entry layer quan trọng trong hệ thống microservice.

- [Giải thích chi tiết API Gateway](./api-gateway.md): hiểu request routing, authentication and authorization, rate limiting and circuit breaking, load balancing và kiến trúc dual-layer gateway.
- [Tổng hợp câu hỏi phỏng vấn Spring Cloud Gateway](./spring-cloud-gateway-questions.md): nắm vững Predicate, GatewayFilter, GlobalFilter, rate limiting and circuit breaking và các vấn đề production thường gặp.

### Distributed ID, lock và transaction

Phần này thiên về engineering implementation, thường gặp trong các scenario order, payment, flash sale, inventory deduction, data consistency và cross-service collaboration.

- [Giải thích chi tiết solution tạo distributed ID](./distributed-id.md): so sánh UUID, database auto-increment, segment mode, Redis, Snowflake, Leaf, Tinyid và các solution khác.
- [Thực chiến thiết kế distributed ID](./distributed-id-design.md): kết hợp các business scenario như order number, payment code, coupon, one-code payment để hiểu thiết kế business ID.
- [Nhập môn distributed lock](./distributed-lock.md): hiểu mutual exclusion semantics, lock granularity, safe release, timeout renewal và Fencing Token.
- [Giải thích chi tiết các solution triển khai distributed lock](./distributed-lock-implementations.md): so sánh distributed lock của Redis, Redisson, Redlock, ZooKeeper và Curator.
- [Giải thích chi tiết solution distributed transaction](./distributed-transaction.md): hiểu có hệ thống XA, AT, TCC, Saga, local message table, transaction message và best-effort notification.

### Config center và ZooKeeper

Config center và ZooKeeper chủ yếu giải quyết các vấn đề về service configuration, service registration and discovery, distributed coordination và cluster consistency.

- [Giải thích chi tiết distributed config center](./distributed-configuration-center.md): so sánh Apollo, Nacos, Spring Cloud Config và Kubernetes ConfigMap.
- [Chuyên đề ZooKeeper](./distributed-process-coordination/zookeeper/): từ core concept của ZooKeeper đến ZAB, Leader election, Curator và thực chiến distributed lock.
- [Hướng dẫn nhập môn ZooKeeper](./distributed-process-coordination/zookeeper/zookeeper-intro.md): nắm vững ZNode, Watcher, ACL và các use case điển hình.
- [Giải thích chi tiết ZooKeeper nâng cao](./distributed-process-coordination/zookeeper/zookeeper-plus.md): hiểu protocol ZAB, Leader election, cluster deployment và session management.
- [Tutorial thực chiến ZooKeeper](./distributed-process-coordination/zookeeper/zookeeper-in-action.md): hoàn thành thực hành với Docker, zkCli, four-letter commands và Curator.

## Câu hỏi thường gặp

- Vì sao hệ thống phân tán không thể đồng thời đáp ứng hoàn hảo consistency, availability và partition tolerance?
- Mối quan hệ giữa CAP và BASE là gì? Eventual consistency phù hợp với những scenario nào?
- Vì sao hệ thống phân tán cần coordination? Tập trung và phi tập trung lần lượt phù hợp với những scenario nào?
- Paxos, Raft và ZAB khác nhau thế nào, lần lượt được dùng ở đâu?
- RPC và HTTP khác nhau thế nào? Vì sao service invocation nội bộ thường dùng RPC?
- Distributed ID đảm bảo global unique, tăng dần theo xu hướng và business readability bằng cách nào?
- Distributed lock của Redis có những vấn đề nào? Redlock có chắc chắn reliable không?
- Distributed transaction có những solution nào? TCC, Saga và local message table lần lượt phù hợp với scenario nào?
- Config center và service registry khác nhau thế nào? Nên lựa chọn Apollo, Nacos và Spring Cloud Config ra sao?
- Microservice nên được tách thế nào? Khi nào việc tách service sẽ biến thành distributed monolith?
- Các microservice xử lý communication failure, data consistency giữa các service và canary release thế nào?

## Chuyên đề liên quan

- [System Design](../system-design/)
- [Thiết kế hệ thống high availability](../high-availability/)
- [Thiết kế hệ thống high performance](../high-performance/)
- [Computer Network](../cs-basics/network/)

<!-- @include: @article-footer.snippet.md -->
