---
title: "Chuyên đề Redis: cache, data structure, persistence, cluster, blocking và thực tiễn engineering"
description: "Lộ trình học Redis và cache cho phỏng vấn, bao quát cache xuyên thấu (cache penetration), sập cache (cache breakdown), cache avalanche, chiến lược đọc ghi, data structure của Redis, persistence, vấn đề blocking, delayed task và cluster."
category: Database
tag:
  - Redis
  - Cache
  - Phỏng vấn backend
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Redis,câu hỏi phỏng vấn Redis,cache,cache penetration,cache breakdown,cache avalanche,data structure Redis,persistence Redis,cluster Redis,blocking Redis,skip list Redis,delayed task Redis,message queue Redis,phỏng vấn backend
---

Redis là một trong những cache và in-memory data store hiệu năng cao được sử dụng phổ biến nhất trong backend. Khi học Redis, không nên chỉ dừng ở command và data type, mà còn phải hiểu các vấn đề engineering như chiến lược đọc ghi cache, data structure bên trong, persistence, nguyên nhân blocking, quản lý memory, replication và cluster.

## Dành cho ai

- Backend developer muốn học một cách có hệ thống về nguyên lý Redis, thiết kế cache và thực tiễn engineering.
- Bạn đang chuẩn bị các câu hỏi phỏng vấn liên quan đến data structure, persistence, cluster, high availability và cache consistency của Redis.
- Bạn đã sử dụng Redis trong project nhưng chưa thật sự quen với cache exception, blocking, memory fragmentation và cơ chế cluster.
- Engineer cần dùng Redis để triển khai delayed task, message queue, leaderboard, shopping cart và các khả năng khác.

## Trọng tâm học

- Các data structure thường dùng của Redis phù hợp với những business scenario nào, encoding bên trong ảnh hưởng đến performance ra sao?
- Nên thiết kế giải pháp thế nào cho các vấn đề cache penetration, cache breakdown, cache avalanche và cache consistency?
- RDB, AOF, AOF rewrite và hybrid persistence trong Redis khác nhau thế nào?
- Vì sao Redis có thể bị blocking, làm thế nào để xác định slow command, big key, hot key và ảnh hưởng của persistence?
- Master-slave replication, Sentinel và Cluster lần lượt giải quyết vấn đề gì, failover có những quy trình quan trọng nào?
- Khi dùng Redis để triển khai delayed task và message queue, giới hạn khả năng và rủi ro về reliability là gì?

## Thứ tự đọc đề xuất

1. [Tổng hợp câu hỏi phỏng vấn cache cơ bản](./cache-basics.md): trước tiên hiểu các trường hợp sử dụng cache, cache exception và vấn đề consistency.
2. [Tổng hợp câu hỏi phỏng vấn Redis (phần 1)](./redis-questions-01.md), [Tổng hợp câu hỏi phỏng vấn Redis (phần 2)](./redis-questions-02.md): xây dựng danh sách các vấn đề Redis thường gặp.
3. [Giải thích chi tiết 5 data type cơ bản của Redis](./redis-data-structures-01.md), [Giải thích chi tiết 3 data type đặc biệt của Redis](./redis-data-structures-02.md): nắm vững một cách có hệ thống data structure và application scenario.
4. [Giải thích chi tiết 3 chiến lược đọc ghi cache thường dùng](./3-commonly-used-cache-read-and-write-strategies.md), [Giải thích chi tiết cơ chế persistence của Redis](./redis-persistence.md): bổ sung kiến thức về cache consistency và khả năng khôi phục dữ liệu.
5. [Tổng hợp các nguyên nhân blocking thường gặp của Redis](./redis-common-blocking-problems-summary.md), [Giải thích chi tiết Redis Cluster](./redis-cluster.md): hiểu Redis trong môi trường production.

## Bài viết cốt lõi

### Cache cơ bản và chiến lược đọc ghi

- [Tổng hợp câu hỏi phỏng vấn cache cơ bản](./cache-basics.md): giải thích application scenario của cache, cache penetration, cache breakdown, cache avalanche, cache consistency và cache eviction.
- [Giải thích chi tiết 3 chiến lược đọc ghi cache thường dùng](./3-commonly-used-cache-read-and-write-strategies.md): so sánh các chiến lược thường gặp như Cache Aside, Read/Write Through và Write Behind.
- [Tổng hợp câu hỏi phỏng vấn Redis (phần 1)](./redis-questions-01.md) và [Tổng hợp câu hỏi phỏng vấn Redis (phần 2)](./redis-questions-02.md): kết nối các kiến thức về Redis cơ bản, thread model, data structure, persistence, cluster và vấn đề production.

### Data structure và application điển hình

- [Giải thích chi tiết 5 data type cơ bản của Redis](./redis-data-structures-01.md): tìm hiểu data structure bên trong và business scenario của String, List, Hash, Set, Sorted Set.
- [Giải thích chi tiết 3 data type đặc biệt của Redis](./redis-data-structures-02.md): tìm hiểu cách dùng và application scenario phù hợp của Bitmap, HyperLogLog và Geospatial.
- [Vì sao Redis dùng skip list để triển khai ordered set](./redis-skiplist.md): tìm hiểu cấu trúc skip list, complexity của truy vấn và lựa chọn triển khai của Sorted Set.
- [Làm thế nào để triển khai delayed task dựa trên Redis?](./redis-delayed-task.md): so sánh các phương thức triển khai bằng expiration event, Sorted Set và Stream.
- [Làm thế nào để triển khai message queue dựa trên Redis?](./redis-stream-mq.md): tìm hiểu sự khác biệt khi dùng List, Pub/Sub và Stream làm message queue.

### Persistence, memory và cluster

- [Giải thích chi tiết cơ chế persistence của Redis](./redis-persistence.md): giải thích một cách có hệ thống về RDB, AOF, AOF rewrite và hybrid persistence.
- [Giải thích chi tiết memory fragmentation của Redis](./redis-memory-fragmentation.md): tìm hiểu nguyên nhân tạo ra memory fragmentation, cách theo dõi metric và chiến lược dọn dẹp.
- [Tổng hợp các nguyên nhân blocking thường gặp của Redis](./redis-common-blocking-problems-summary.md): tổng hợp các nguồn blocking như slow command, big key, persistence, master-slave synchronization, CPU và network.
- [Giải thích chi tiết Redis Cluster](./redis-cluster.md): tìm hiểu master-slave replication, Sentinel, Cluster, slot migration và failover.

## Câu hỏi thường gặp

- Vì sao Redis nhanh? Vì sao single thread vẫn có thể đáp ứng concurrency cao?
- Các data type thường gặp của Redis lần lượt phù hợp với những business scenario nào?
- Vì sao Sorted Set sử dụng skip list?
- Cache penetration, cache breakdown và cache avalanche khác nhau thế nào, xử lý ra sao?
- Làm thế nào để bảo đảm consistency giữa cache và database?
- RDB và AOF khác nhau thế nào? AOF rewrite giải quyết vấn đề gì?
- Các nguyên nhân blocking thường gặp của Redis là gì? Làm thế nào để kiểm tra big key và slow command?
- Master-slave replication, Sentinel và Cluster của Redis khác nhau thế nào?
- Redis triển khai delayed task thế nào? Rủi ro về reliability nằm ở đâu?
- Stream của Redis có những giới hạn nào so với message queue truyền thống?

## Chuyên đề liên quan

- [Hệ thống kiến thức Database](../)
- [Hệ thống kiến thức High Performance](../../high-performance/)
- [Hệ thống kiến thức High Availability](../../high-availability/)
- [Chuyên đề Message Queue](../../high-performance/message-queue/)

<!-- @include: @article-footer.snippet.md -->
