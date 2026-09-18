---
title: "Lý thuyết, thuật toán và giao thức phân tán: CAP, BASE, tập trung và phi tập trung, Paxos, Raft và Consistent Hashing"
description: "Lộ trình học lý thuyết và giao thức phân tán, bao quát CAP, BASE, tập trung và phi tập trung, bài toán các vị tướng Byzantine, Paxos, Raft, ZAB, Gossip, Consistent Hashing và các nội dung khác; giúp hiểu các vấn đề về consistency, fault tolerance, consensus, lan truyền trạng thái và phân bố dữ liệu."
category: Distributed
tag:
  - Distributed Systems Theory
  - Distributed Protocols and Algorithms
  - Consensus Algorithms
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: distributed systems theory,distributed algorithms,distributed protocols,centralized,decentralized,CAP,BASE,Byzantine generals problem,Paxos,Raft,ZAB,Gossip,consistent hashing,consensus algorithms,Byzantine fault tolerance,BFT,eventual consistency,distributed systems interview questions
---

Lý thuyết, thuật toán và giao thức phân tán là nền tảng để hiểu distributed system. Khi học phần này, không nên chỉ học thuộc kết luận; quan trọng hơn là hiểu sự đánh đổi giữa consistency, availability, fault tolerance, consensus, performance và độ phức tạp khi triển khai trong thực tế của các phương án khác nhau.

## Dành cho ai

- Backend developer muốn hiểu một cách hệ thống về distributed consistency, consensus algorithm và phân bố dữ liệu.
- Những bạn chuẩn bị phỏng vấn về distributed system, system design và backend architecture.
- Những người chỉ dừng ở mức “đã học thuộc” các khái niệm như CAP, BASE, Paxos, Raft, ZAB, Gossip.
- Kỹ sư cần hiểu cơ chế bên trong của ZooKeeper, Redis Cluster, distributed cache và service discovery.

## Trọng tâm học

- CAP và BASE không phải khẩu hiệu; chúng tương ứng với những đánh đổi thực tế nào trong engineering?
- Centralized và decentralized đang giải quyết vấn đề gì? Leader, majority và Gossip lần lượt phù hợp với những trường hợp nào?
- Paxos, Raft và ZAB đều giải quyết bài toán consensus; tại sao độ phức tạp khi triển khai và trường hợp sử dụng của chúng lại khác nhau?
- Tại sao Gossip phù hợp với việc lan truyền trạng thái giữa số lượng lớn node, và nó khác gì với strong consistency protocol?
- Consistent Hashing làm giảm chi phí di chuyển dữ liệu khi mở rộng hoặc thu hẹp node như thế nào?
- Trong phỏng vấn, làm thế nào để trình bày rõ một protocol theo “bối cảnh vấn đề -> ý tưởng cốt lõi -> quy trình -> ưu / nhược điểm -> trường hợp sử dụng”?

## Các bài viết trong nhóm này bổ trợ cho nhau như thế nào?

[Giải thích chi tiết CAP theorem và BASE theory](./cap-and-base-theorem.md) chịu trách nhiệm giải thích nhóm đánh đổi “consistency, availability, partition tolerance” trong distributed system. [Giải thích chi tiết distributed coordination](./centralized-and-decentralized.md) tiếp tục đi sâu vào các trường hợp engineering, thảo luận ai là người đưa ra quyết định, tại sao Leader cũ có thể gây ra vấn đề và khi nào nên dùng Gossip.

Paxos, Raft và ZAB thuộc tuyến consensus protocol. Nên đọc [Giải thích chi tiết thuật toán Raft](./raft-algorithm.md) trước, sau đó đọc [Giải thích chi tiết thuật toán Paxos](./paxos-algorithm.md) và [Giải thích chi tiết giao thức ZAB](./zab.md). Raft phù hợp hơn cho người mới bắt đầu, Paxos gần với lý thuyết kinh điển hơn, còn ZAB tương ứng với atomic broadcast và crash recovery của ZooKeeper.

[Giải thích chi tiết Gossip protocol](./gossip-protocol.md) và [Giải thích chi tiết thuật toán Consistent Hashing](./consistent-hashing.md) không phụ trách thứ tự ghi strong consistency; chúng lần lượt giải quyết vấn đề lan truyền trạng thái và ánh xạ dữ liệu. Đặt hai bài này sau các consensus protocol sẽ giúp tránh nhầm lẫn giữa “lan truyền trạng thái” và “đạt được consensus”.

## Thứ tự đọc đề xuất

1. [Giải thích chi tiết CAP theorem và BASE theory](./cap-and-base-theorem.md): trước tiên xây dựng góc nhìn về sự đánh đổi giữa consistency, availability và partition tolerance.
2. [Giải thích chi tiết distributed coordination](./centralized-and-decentralized.md): đặt Leader, Quorum, split-brain, Lease, Fencing Token và Gossip trên cùng một mạch nội dung.
3. [Giải thích chi tiết bài toán các vị tướng Byzantine](./byzantine-generals-problem.md): hiểu node độc hại, message mâu thuẫn, yêu cầu `3m + 1` node và giới hạn fault tolerance của BFT.
4. [Giải thích chi tiết thuật toán Raft](./raft-algorithm.md): làm quen với consensus algorithm qua Leader election và log replication tương đối dễ hiểu.
5. [Giải thích chi tiết thuật toán Paxos](./paxos-algorithm.md): hiểu vai trò, các phase và điểm khó của consensus algorithm kinh điển.
6. [Giải thích chi tiết giao thức ZAB](./zab.md): đưa consensus algorithm vào bối cảnh message broadcast và crash recovery của ZooKeeper.
7. [Giải thích chi tiết Gossip protocol](./gossip-protocol.md) và [Giải thích chi tiết thuật toán Consistent Hashing](./consistent-hashing.md): hiểu việc lan truyền trạng thái và phân bố dữ liệu trong các system quy mô lớn.

## Bài viết cốt lõi

### Consistency và distributed theory

- [Giải thích chi tiết CAP theorem và BASE theory](./cap-and-base-theorem.md): hiểu sự đánh đổi giữa consistency, availability và partition tolerance, cũng như ý nghĩa engineering của BASE theory và eventual consistency.
- [Giải thích chi tiết distributed coordination](./centralized-and-decentralized.md): hiểu tại sao distributed system cần coordination, cũng như Leader/Quorum, Gossip, Lease và Fencing Token lần lượt xử lý các vấn đề coordination như thế nào.
- [Giải thích chi tiết bài toán các vị tướng Byzantine](./byzantine-generals-problem.md): hiểu khi tồn tại node độc hại hoặc node bất thường, các node bình thường làm thế nào để đạt được cùng một kết quả.

### Consensus algorithm

- [Giải thích chi tiết thuật toán Paxos](./paxos-algorithm.md): hiểu vai trò của Proposer, Acceptor, Learner, quy trình hai phase và tối ưu hóa Multi-Paxos.
- [Giải thích chi tiết thuật toán Raft](./raft-algorithm.md): hiểu Leader election, log replication, các ràng buộc về safety, thay đổi member và khác biệt so với Paxos.
- [Giải thích chi tiết giao thức ZAB](./zab.md): hiểu ZooKeeper Atomic Broadcast, message broadcast, crash recovery, ZXID và transaction log.

### Lan truyền và phân bố dữ liệu

- [Giải thích chi tiết Gossip protocol](./gossip-protocol.md): hiểu anti-entropy, rumor propagation, mô hình Push/Pull, SWIM protocol và eventual consistency.
- [Giải thích chi tiết thuật toán Consistent Hashing](./consistent-hashing.md): hiểu hash ring, virtual node, mở rộng và thu hẹp node, data skew và ứng dụng trong distributed cache.

## Câu hỏi thường gặp

- C, A và P trong CAP lần lượt có nghĩa là gì? Tại sao partition tolerance thường không thể loại bỏ?
- BASE theory và eventual consistency giải quyết vấn đề gì?
- Centralized design và decentralized design khác nhau như thế nào? Tại sao có Leader không nhất thiết đồng nghĩa với single point?
- Tại sao Paxos khó hiểu? Basic Paxos và Multi-Paxos khác nhau như thế nào?
- Tại sao Raft dễ triển khai trong engineering hơn? Leader election và log replication bảo đảm safety như thế nào?
- ZAB và Raft có những điểm tương đồng và khác biệt nào?
- Gossip protocol phù hợp với những trường hợp nào? Tại sao nó thường không cung cấp strong consistency?
- Tại sao Consistent Hashing cần virtual node?

## Chuyên đề liên quan

- [Hệ thống kiến thức về distributed system](../)
- [Chuyên đề ZooKeeper](../distributed-process-coordination/zookeeper/)
- [Giải thích chi tiết distributed configuration center](../distributed-configuration-center.md)
- [Giải thích chi tiết phương án tạo distributed ID](../distributed-id.md)

<!-- @include: @article-footer.snippet.md -->
