---
title: "Hệ thống kiến thức Database: SQL, MySQL, Redis, MongoDB và Elasticsearch"
description: "Lộ trình học hệ thống kiến thức Database và ôn phỏng vấn, bao quát SQL, index MySQL, transaction, log, MVCC, execution plan, cache Redis, MongoDB và Elasticsearch."
category: Database
tag:
  - Database
  - MySQL
  - Redis
sitemap:
  changefreq: weekly
  priority: 0.95
head:
  - - meta
    - name: keywords
      content: Database,câu hỏi phỏng vấn Database,SQL,MySQL,Redis,MongoDB,Elasticsearch,index MySQL,transaction MySQL,log MySQL,MVCC,cache Redis,persistence Redis,cluster Redis,phỏng vấn backend
---

<!-- @include: @small-advertisement.snippet.md -->

**Hệ thống kiến thức Database** này dành cho việc học backend, thực hành kỹ thuật và ôn tập phỏng vấn. Các bài viết về Database trên website được sắp xếp theo thứ tự “kiến thức cơ bản về Database -> SQL -> MySQL -> Redis -> NoSQL và công cụ tìm kiếm”.

Nếu có ít thời gian, bạn nên đọc trước [Tổng hợp câu hỏi phỏng vấn Database cơ bản](./basis.md), [Tổng hợp kiến thức cơ bản về cú pháp SQL](./sql/sql-syntax-summary.md), [Tổng hợp câu hỏi phỏng vấn MySQL thường gặp](./mysql/mysql-questions-01.md) và [Tổng hợp câu hỏi phỏng vấn Redis thường gặp (phần 1)](./redis/redis-questions-01.md) để nhanh chóng xây dựng danh sách các câu hỏi thường gặp.

## Dành cho ai

- Backend developer đang học một cách có hệ thống kiến thức cơ bản về Database, SQL, MySQL và Redis.
- Người chuẩn bị cho phỏng vấn backend khi ứng tuyển fresher, developer đã có kinh nghiệm hoặc vào các công ty vừa và lớn.
- Engineer muốn bổ sung kiến thức về index, transaction, log, execution plan, cache consistency, persistence và cluster của Redis.
- Người đã từng viết CRUD nghiệp vụ nhưng chưa nắm vững các nguyên lý bên trong Database, tối ưu performance và cách lựa chọn NoSQL.

## Trọng tâm học

- Relational database và NoSQL phù hợp để giải quyết những vấn đề nào, ranh giới lựa chọn thường gặp ở đâu?
- Cần nắm vững query, aggregation, join, subquery và ngữ nghĩa của transaction trong SQL như thế nào?
- Làm thế nào để kết nối index MySQL, transaction isolation, MVCC, ba log chính và execution plan thành một mạch kiến thức?
- Vì sao Redis nhanh, cần hiểu thế nào về các data structure thường dùng, cache strategy, persistence, vấn đề blocking và cơ chế cluster?
- MongoDB, Elasticsearch và các hệ thống NoSQL/search tương tự thường được hỏi những điểm nào trong phỏng vấn backend và lựa chọn kỹ thuật?

## Thứ tự đọc đề xuất

1. [Tổng hợp câu hỏi phỏng vấn Database cơ bản](./basis.md) và [Tổng hợp câu hỏi phỏng vấn NoSQL cơ bản](./nosql.md): trước tiên hiểu phân loại Database, transaction, normalization, các loại NoSQL và trường hợp sử dụng điển hình.
2. [Tổng hợp kiến thức cơ bản về cú pháp SQL](./sql/sql-syntax-summary.md): bổ sung kiến thức cơ bản về query, filter, sort, aggregation, join, subquery, insert, update và delete.
3. [Chuyên đề MySQL](./mysql/): tập trung học index, transaction isolation, MVCC, ba log chính, quy trình thực thi và execution plan.
4. [Chuyên đề Redis](./redis/): tập trung học kiến thức cơ bản về cache, data structure, chiến lược đọc/ghi cache, persistence, vấn đề blocking, memory fragmentation và cluster.
5. [Chuyên đề MongoDB](./mongodb/) và [Tổng hợp câu hỏi phỏng vấn Elasticsearch thường gặp](./elasticsearch/elasticsearch-questions-01.md): bổ sung kiến thức về document database và search engine theo yêu cầu vị trí tuyển dụng.

## Bài viết cốt lõi

### Kiến thức cơ bản về Database và SQL

Phần này phù hợp để hình thành nhận thức tổng quan về Database trước, trọng tâm là hiểu loại Database, ngữ nghĩa transaction, character set, kiến thức cơ bản về SQL và các câu hỏi query thường gặp.

- [Tổng hợp câu hỏi phỏng vấn Database cơ bản](./basis.md): hệ thống hóa khái niệm cơ bản về Database, tính chất của transaction, concurrency control, normalization và các câu hỏi phỏng vấn thường gặp.
- [Tổng hợp câu hỏi phỏng vấn NoSQL cơ bản](./nosql.md): tìm hiểu các loại NoSQL như key-value, document, column family và graph database cùng trường hợp sử dụng phù hợp.
- [Giải thích chi tiết về character set: character set là gì? Sử dụng thế nào?](./character-set.md): tìm hiểu character set, encoding, nguyên nhân lỗi ký tự và thiết lập character set của MySQL.
- [Chuyên đề SQL](./sql/): trình bày từ kiến thức cơ bản về cú pháp SQL đến các câu hỏi phỏng vấn SQL thường gặp.
- [Tổng hợp kiến thức cơ bản về cú pháp SQL](./sql/sql-syntax-summary.md): bao quát query, filter, sort, aggregation, group, join, subquery và sửa đổi dữ liệu.

### MySQL

MySQL là một trong những chủ đề trọng tâm nhất về relational database đối với backend developer. Khi học, nên kết nối “index -> execution plan -> transaction -> MVCC -> log -> tối ưu performance” thành một mạch.

- [Chuyên đề MySQL](./mysql/): kết nối các kiến thức về index MySQL, transaction, MVCC, log, execution plan và tối ưu performance.
- [Tổng hợp câu hỏi phỏng vấn MySQL thường gặp](./mysql/mysql-questions-01.md): nhanh chóng xây dựng danh sách các câu hỏi MySQL thường gặp.
- [Giải thích chi tiết về index MySQL](./mysql/mysql-index.md): tìm hiểu data structure của index, leftmost prefix, covering index, table lookup và nguyên tắc thiết kế index.
- [Giải thích chi tiết về transaction isolation level của MySQL](./mysql/transaction-isolation-level.md): tìm hiểu dirty read, non-repeatable read, phantom read và sự đánh đổi giữa các isolation level khác nhau.
- [Cách InnoDB triển khai MVCC](./mysql/innodb-implementation-of-mvcc.md): tìm hiểu Read View, hidden field, undo log và snapshot read.
- [Giải thích chi tiết về ba log chính của MySQL](./mysql/mysql-logs.md): tìm hiểu vai trò và mối quan hệ của binlog, redo log và undo log.
- [Giải thích chi tiết việc đồng bộ MySQL sang Elasticsearch](./mysql/mysql-to-elasticsearch-sync.md): so sánh double write ở application layer, đồng bộ định kỳ, Canal, Debezium và Flink CDC; tìm hiểu các vấn đề về full sync, incremental sync và data consistency.
- [Phân tích execution plan của MySQL](./mysql/mysql-query-execution-plan.md): nắm vững các field thường gặp của EXPLAIN và điểm bắt đầu phân tích slow SQL.

### Redis

Redis vừa là cache, vừa là middleware thường gặp trong phỏng vấn. Khi học, không nên chỉ học thuộc command mà cần hiểu cache strategy, data structure, persistence, nguyên nhân blocking và cơ chế cluster.

- [Chuyên đề Redis](./redis/): tập trung vào cache, data structure, persistence, cluster, vấn đề blocking và thực hành kỹ thuật.
- [Tổng hợp câu hỏi phỏng vấn cơ bản về cache](./redis/cache-basics.md): tìm hiểu trường hợp sử dụng cache, cache penetration (cache xuyên thấu), cache breakdown (sập cache), cache avalanche và vấn đề consistency.
- [Tổng hợp câu hỏi phỏng vấn Redis thường gặp (phần 1)](./redis/redis-questions-01.md) và [Tổng hợp câu hỏi phỏng vấn Redis thường gặp (phần 2)](./redis/redis-questions-02.md): nhanh chóng xây dựng danh sách các câu hỏi Redis thường gặp.
- [Giải thích chi tiết 5 data type cơ bản của Redis](./redis/redis-data-structures-01.md): tìm hiểu trường hợp sử dụng của String, List, Hash, Set và Sorted Set.
- [Giải thích chi tiết cơ chế persistence của Redis](./redis/redis-persistence.md): tìm hiểu RDB, AOF, AOF rewrite và mixed persistence.
- [Giải thích chi tiết Redis Cluster](./redis/redis-cluster.md): tìm hiểu master-slave replication, Sentinel, Cluster, slot và failover.

### NoSQL và công cụ tìm kiếm

Phần này phù hợp để bổ sung sau khi đã nắm vững relational database và cache, nhằm hiểu ranh giới lựa chọn của document database, search engine và non-relational storage.

- [Chuyên đề MongoDB](./mongodb/): hệ thống hóa document model, index, replica set, sharding, transaction và các câu hỏi phỏng vấn thường gặp về MongoDB.
- [Tổng hợp câu hỏi phỏng vấn MongoDB thường gặp (phần 1)](./mongodb/mongodb-questions-01.md) và [Tổng hợp câu hỏi phỏng vấn MongoDB thường gặp (phần 2)](./mongodb/mongodb-questions-02.md): tìm hiểu khái niệm cốt lõi và thực hành engineering của MongoDB.
- [Tổng hợp câu hỏi phỏng vấn Elasticsearch thường gặp](./elasticsearch/elasticsearch-questions-01.md): tìm hiểu inverted index, sharding, replica, quy trình write/query và các trường hợp sử dụng search.

## Câu hỏi thường gặp

- Relational database và NoSQL khác nhau thế nào? Mỗi loại phù hợp với những trường hợp sử dụng nào?
- ACID của transaction trong Database là gì? Mỗi isolation level giải quyết vấn đề gì?
- Nên hiểu thứ tự thực thi của WHERE, GROUP BY, HAVING và ORDER BY trong SQL như thế nào?
- Vì sao index MySQL có thể tăng tốc query? Trong trường hợp nào index không được sử dụng?
- InnoDB triển khai consistent read không dùng lock thông qua MVCC như thế nào?
- binlog, redo log và undo log lần lượt giải quyết vấn đề gì?
- Làm thế nào để phân tích execution plan của SQL bằng EXPLAIN?
- Vì sao Redis nhanh? Xử lý cache penetration, cache breakdown và cache avalanche thế nào?
- Persistence, master-slave replication, Sentinel và Cluster của Redis lần lượt giải quyết vấn đề gì?
- MongoDB và Elasticsearch lần lượt phù hợp với những business scenario nào?

## Chuyên đề liên quan

- [Hệ thống kiến thức về high-performance](../high-performance/)
- [Hệ thống kiến thức về high-availability](../high-availability/)
- [Hệ thống kiến thức về distributed system](../distributed-system/)
- [System design](../system-design/)

<!-- @include: @article-footer.snippet.md -->
