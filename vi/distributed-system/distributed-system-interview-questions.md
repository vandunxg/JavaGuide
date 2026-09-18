---
title: "Tổng hợp câu hỏi phỏng vấn hệ thống phân tán mới nhất 2026: CAP, Raft, RPC, distributed lock, transaction và ID"
description: "Tổng hợp câu hỏi phỏng vấn và lộ trình ôn tập hệ thống phân tán mới nhất 2026, bao quát CAP, BASE, centralized và decentralized, Paxos, Raft, ZAB, Gossip, consistent hashing, RPC, API gateway, distributed ID, distributed lock, distributed transaction, configuration center và ZooKeeper cùng các trọng tâm thường gặp."
category: Hệ thống phân tán
tag:
  - Hệ thống phân tán
  - Câu hỏi phỏng vấn
  - Thiết kế hệ thống
head:
  - - meta
    - name: keywords
      content: câu hỏi phỏng vấn hệ thống phân tán,câu hỏi phỏng vấn distributed system,centralized,decentralized,câu hỏi phỏng vấn CAP,câu hỏi phỏng vấn BASE,câu hỏi phỏng vấn RPC,câu hỏi phỏng vấn API gateway,câu hỏi phỏng vấn distributed lock,câu hỏi phỏng vấn distributed transaction,câu hỏi phỏng vấn distributed ID,câu hỏi phỏng vấn ZooKeeper,câu hỏi phỏng vấn Raft,câu hỏi phỏng vấn Paxos
---

Phỏng vấn hệ thống phân tán hiếm khi yêu cầu bạn học thuộc riêng một định nghĩa CAP. Người phỏng vấn thường bắt đầu từ một vấn đề nghiệp vụ rồi hỏi tiếp: Vì sao service phải tách thành nhiều node? Sau khi network timeout thì có thể retry không? Nếu lock hết hạn sớm thì phải làm sao? Dữ liệu giữa các service được giữ consistency bằng cách nào?

Bài viết này là cổng ôn tập cho chuyên đề hệ thống phân tán của JavaGuide, được sắp xếp thành bốn phần: lý thuyết phân tán, RPC và gateway, distributed ID/lock/transaction, configuration center và ZooKeeper. Mỗi phần chỉ liệt kê những vấn đề cần nắm khi ôn tập; câu trả lời và chi tiết triển khai nằm trong các bài viết chuyên đề tương ứng.

Nếu thời gian khá gấp, bạn có thể xem trước [Tổng hợp câu hỏi phỏng vấn hệ thống phân tán thường gặp](https://interview.javaguide.cn/distributed-system/distributed-system.html), đánh dấu những câu chưa biết, rồi quay lại bài này để bổ sung nguyên lý và chi tiết engineering.

## Khi ôn tập, trước tiên cần nắm những vấn đề nào?

| Module                        | Nội dung cần trình bày rõ ràng                                                                                       | Hướng hỏi tiếp thường gặp                                                                                  |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| Lý thuyết phân tán            | Khi node và network đều không đáng tin cậy, hệ thống cân bằng giữa consistency và availability như thế nào           | CAP, BASE, consensus algorithm, Gossip, consistent hashing                                                 |
| RPC và gateway                | Request đi vào hệ thống như thế nào, các service gọi nhau ra sao                                                     | service registration and discovery, load balancing, serialization, timeout retry, routing và rate limiting |
| ID, lock và transaction       | Khi nhiều node đồng thời đọc ghi dữ liệu, kiểm soát uniqueness, mutual exclusion và business consistency như thế nào | Snowflake, Redis lock, 2PC, TCC, transactional message, Saga                                               |
| Configuration và coordination | Node phát hiện thay đổi cấu hình và phối hợp làm việc như thế nào                                                    | config push, version management, ZooKeeper node, Watcher, Leader election                                  |

Khi trả lời các câu hỏi này, đừng chỉ dừng ở “có những phương án nào”. Ít nhất cần nói rõ điều kiện chọn phương án, chuyện gì xảy ra khi thất bại, và trong project đã chuẩn bị những biện pháp fallback nào.

## Lý thuyết và algorithm phân tán

Trước tiên, hãy hình dung một scenario thường gặp nhất: cùng một dữ liệu được lưu thành nhiều replica, node có thể bị down, network giữa các node cũng có thể bị gián đoạn. CAP, BASE và consensus algorithm đều thảo luận về lựa chọn và ràng buộc trong scenario này.

![Cơ chế giao tiếp của hệ thống phân tán: centralized vs decentralized](https://oss.javaguide.cn/github/javaguide/distributed-system/protocol/gossip-centralized-vs-decentralized.png)

Nội dung liên quan:

- [Giải thích CAP và BASE](https://javaguide.cn/distributed-system/protocol/cap-and-base-theorem.html)
- [Giải thích chi tiết về distributed coordination](https://javaguide.cn/distributed-system/protocol/centralized-and-decentralized.html)
- [Giải thích thuật toán Paxos](https://javaguide.cn/distributed-system/protocol/paxos-algorithm.html)
- [Giải thích thuật toán Raft](https://javaguide.cn/distributed-system/protocol/raft-algorithm.html)
- [Giải thích chi tiết giao thức ZAB](https://javaguide.cn/distributed-system/protocol/zab.html)
- [Giải thích chi tiết giao thức Gossip](https://javaguide.cn/distributed-system/protocol/gossip-protocol.html)
- [Giải thích chi tiết thuật toán consistent hashing](https://javaguide.cn/distributed-system/protocol/consistent-hashing.html)

Câu hỏi phỏng vấn thường gặp:

- CAP có phải là “chọn hai trong ba” không? Vì sao nói P về cơ bản không thể né tránh trong hệ thống phân tán?
- BASE và ACID khác nhau như thế nào? Eventual consistency được triển khai ra sao?
- Centralized và decentralized khác nhau như thế nào? Cần hiểu Leader single point, split-brain, majority và Gossip ra sao?
- Paxos, Raft và ZAB lần lượt giải quyết vấn đề gì? Vì sao Raft dễ hiểu và triển khai hơn?
- Vì sao giao thức Gossip phù hợp với service discovery và state propagation?
- Consistent hashing giải quyết vấn đề gì? Virtual node có tác dụng gì?

Phần này cần có khả năng liên kết để trả lời: Sau khi xảy ra network partition, hệ thống còn cung cấp được năng lực nào; khi nhiều replica xuất hiện bất đồng, ai quyết định kết quả cuối cùng; sau khi node tăng giảm, dữ liệu được phân phối lại như thế nào. Chỉ học thuộc các bước của algorithm thường chưa đủ khi bị hỏi sâu theo scenario.

## RPC và API gateway

RPC xử lý việc gọi giữa các service, còn API gateway tiếp nhận request bên ngoài. Khi ôn tập, có thể lần theo một request từ trên xuống: request trước tiên đi qua gateway để thực hiện authentication, routing hoặc rate limiting, sau đó vào service cụ thể; các service tiếp tục gọi nhau thông qua RPC, còn service registration and discovery, load balancing, serialization và timeout control đều phát huy tác dụng trên chuỗi này.

Sơ đồ API gateway như sau:

![Sơ đồ gateway](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/api-gateway-overview.png)

Sơ đồ RPC như sau:

![Tổng quan RPC](https://oss.javaguide.cn/github/javaguide/distributed-system/rpc/rpc-overview.png)

Nội dung liên quan:

- [Tổng hợp câu hỏi phỏng vấn microservice](https://javaguide.cn/distributed-system/microservices-interview-questions.html)
- [Tổng hợp câu hỏi phỏng vấn thường gặp về RPC cơ bản](https://javaguide.cn/distributed-system/rpc/rpc-intro.html)
- [Tổng hợp câu hỏi phỏng vấn thường gặp về Dubbo](https://javaguide.cn/distributed-system/rpc/dubbo.html)
- [RPC và HTTP khác nhau như thế nào?](https://javaguide.cn/distributed-system/rpc/http&rpc.html)
- [Tổng hợp kiến thức cơ bản về API gateway](https://javaguide.cn/distributed-system/api-gateway.html)
- [Tổng hợp vấn đề thường gặp về Spring Cloud Gateway](https://javaguide.cn/distributed-system/spring-cloud-gateway-questions.html)

Câu hỏi phỏng vấn thường gặp:

- RPC và HTTP khác nhau như thế nào? Vì sao gọi nội bộ giữa các service thường dùng RPC?
- Một framework RPC thường gồm những module cốt lõi nào?
- Service registration and discovery, load balancing, serialization và timeout retry lần lượt giải quyết vấn đề gì?
- Ranh giới giữa API gateway, Nginx, load balancer và BFF là gì?
- Spring Cloud Gateway hoàn thành routing match như thế nào? Filter chain, rate limiting và circuit breaker lần lượt có hiệu lực ở giai đoạn nào?

Nhóm câu hỏi này nên được trả lời kết hợp với call chain. Đặc biệt với timeout và retry, không thể chỉ nói “thất bại thì retry”: cách xử lý read request và write request khác nhau, retry nhiều tầng còn có thể khuếch đại traffic, còn write request trước tiên phải cân nhắc idempotent và kết quả không chắc chắn.

## Distributed ID, lock và transaction

Cách tạo order number, cách nhiều instance tranh giành cùng một resource, và cách xử lý partial success khi một nghiệp vụ đi qua nhiều service lần lượt tương ứng với distributed ID, distributed lock và distributed transaction. Chúng thường được hỏi tiếp cùng với project trong các buổi phỏng vấn backend, vì vậy cần dành thêm thời gian ôn tập.

Nội dung liên quan:

- [Giới thiệu distributed ID và tổng hợp các phương án triển khai](https://javaguide.cn/distributed-system/distributed-id.html)
- [Hướng dẫn thiết kế distributed ID](https://javaguide.cn/distributed-system/distributed-id-design.html)
- [Giới thiệu distributed lock](https://javaguide.cn/distributed-system/distributed-lock.html)
- [Tổng hợp các phương án triển khai distributed lock thường gặp](https://javaguide.cn/distributed-system/distributed-lock-implementations.html)
- [Tổng hợp các phương án giải quyết distributed transaction](https://javaguide.cn/distributed-system/distributed-transaction.html)

Câu hỏi phỏng vấn thường gặp:

- Vì sao sau khi sharding database không thể tiếp tục phụ thuộc vào database auto-increment ID?
- UUID, database segment, Redis và Snowflake có những ưu / nhược điểm gì?
- Vì sao distributed lock của Redis cần đặt expiration time? Vì sao phải dùng Lua để bảo đảm tính atomic khi release lock?
- Redisson watchdog giải quyết vấn đề gì? Trong những trường hợp nào lock vẫn có thể mất hiệu lực?
- 2PC, TCC, local message table, transactional message và Saga lần lượt phù hợp với scenario nào?

Khi chuẩn bị câu trả lời, hãy đưa cả tình huống exception vào phương án: clock rollback có ảnh hưởng đến ID không, sau khi lock hết hạn owner ban đầu còn đang thực thi hay không, xử lý việc message được deliver lặp lại như thế nào, sau khi thao tác compensation thất bại thì ai tiếp quản. Khác biệt giữa các phương án engineering thường nằm ở những điểm này.

## Configuration center và ZooKeeper

Configuration center thường được hỏi cùng với configuration change, client push/pull và version management; ZooKeeper thường xuất hiện trong các vấn đề như service registration and discovery, distributed coordination và Leader election.

Nếu CV và project không sử dụng ZooKeeper, có thể hạ mức ưu tiên ôn tập phần này. Chỉ cần CV ghi ZooKeeper thì phải chuẩn bị về ephemeral node, sequential node, Watcher, session mechanism và các ứng dụng điển hình, không thể chỉ trả lời “nó có thể dùng làm distributed lock”.

Nội dung liên quan:

- [Tổng hợp câu hỏi phỏng vấn configuration center phân tán](https://javaguide.cn/distributed-system/distributed-configuration-center.html)
- [Tổng hợp các khái niệm liên quan đến ZooKeeper (nhập môn)](https://javaguide.cn/distributed-system/distributed-process-coordination/zookeeper/zookeeper-intro.html)
- [Tổng hợp các khái niệm liên quan đến ZooKeeper (nâng cao)](https://javaguide.cn/distributed-system/distributed-process-coordination/zookeeper/zookeeper-plus.html)

Câu hỏi phỏng vấn thường gặp:

- Vì sao configuration center không thể chỉ là một kho lưu trữ configuration file?
- Configuration change được push như thế nào? Làm sao bảo đảm client nhận được configuration mới?
- Ephemeral node, sequential node và Watcher của ZooKeeper lần lượt giải quyết được vấn đề gì?
- Vì sao ZooKeeper phù hợp để thực hiện distributed coordination?
- Mối quan hệ giữa ZooKeeper với service registry và configuration center là gì?

## Sắp xếp việc ôn tập theo thời gian chuẩn bị

| Thời gian còn lại | Đề xuất sắp xếp                                                                                                                                                                                                            | Mục tiêu ôn tập                                                                         |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| 1~2 ngày          | Đọc nhanh một lượt [Tổng hợp câu hỏi phỏng vấn hệ thống phân tán thường gặp](https://interview.javaguide.cn/distributed-system/distributed-system.html), ưu tiên bổ sung distributed ID, lock, transaction, RPC và gateway | Có thể trả lời đầy đủ các vấn đề thường gặp, biết hạn chế chính của từng phương án      |
| 3~7 ngày          | Ngoài các câu hỏi thường gặp, bổ sung CAP/BASE, Raft, consistent hashing, đồng thời vẽ call chain từ gateway đến RPC service                                                                                               | Có thể giải thích trade-off phía sau phương án, không dừng ở định nghĩa khi bị hỏi tiếp |
| Trên 1 tuần       | Đọc các bài chuyên đề theo thứ tự của bài viết này, sau đó kết hợp với project của mình để tổng hợp các case exception và cơ sở chọn phương án                                                                             | Có thể trình bày từ ràng buộc nghiệp vụ đến phương án, xử lý lỗi và cách mở rộng        |

Với các vị trí tuyển dụng người đã có kinh nghiệm và trung cao cấp, bạn còn phải đưa câu trả lời trở về project của mình. Khi người phỏng vấn hỏi “Vì sao dùng distributed lock của Redis?”, ưu / nhược điểm chung chỉ là phần mở đầu, sau đó họ thường hỏi: Lock bảo vệ resource nào, concurrency bao nhiêu, timeout và renewal được xử lý ra sao, làm thế nào phát hiện release thất bại. Với những phương án chưa từng sử dụng thực tế, chỉ cần trả lời theo kết quả học tập và tìm hiểu, không được bịa số liệu project.

<!-- @include: @article-footer.snippet.md -->
