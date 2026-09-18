---
title: "Chuyên đề MySQL: index, transaction, log, backup và restore, MVCC và tối ưu performance"
description: Lộ trình học MySQL và performance optimization cho phỏng vấn, bao quát index, index mất hiệu lực, transaction isolation level, MVCC, binlog, redo log, undo log, backup và restore, data synchronization, execution plan và SQL optimization.
category: Database
tag:
  - MySQL
  - Database
  - Phỏng vấn backend
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: MySQL,MySQL interview questions,MySQL index,index mất hiệu lực,transaction isolation level,MVCC,binlog,redo log,undo log,MySQL backup,MySQL restore,MySQL sync ES,Canal,Flink CDC,execution plan,SQL execution process,MySQL performance optimization,backend interview
---

MySQL là một trong những relational database được dùng phổ biến nhất trong backend development, đồng thời là chuyên đề thường bị hỏi sâu nhất trong các buổi phỏng vấn database. Khi học MySQL, nên triển khai xoay quanh các mạch chính: index làm truy vấn nhanh hơn như thế nào, transaction bảo đảm consistency ra sao, log bảo đảm recovery và replication thế nào, dữ liệu được đồng bộ tới các hệ thống không đồng nhất như search ra sao, execution plan định vị slow SQL như thế nào.

## Dành cho ai

- Backend developer muốn học có hệ thống về nguyên lý MySQL và performance optimization.
- Người chuẩn bị các câu hỏi phỏng vấn liên quan đến MySQL index, transaction, MVCC, log và execution plan.
- Người đã có thể viết SQL thông thường nhưng chưa quen với việc phân tích slow SQL, thiết kế index và xử lý vấn đề transaction.
- Engineer cần xử lý performance MySQL, data consistency, data synchronization không đồng nhất và vấn đề thiết kế field trong project.

## Trọng tâm học

- B+ tree index, clustered index, secondary index, covering index và truy ngược về table lần lượt là gì?
- Những cách viết SQL nào sẽ khiến index mất hiệu lực, và làm thế nào để xác minh thông qua execution plan?
- Transaction isolation level của MySQL ảnh hưởng đến dirty read, non-repeatable read và phantom read như thế nào?
- InnoDB thực hiện snapshot read thông qua MVCC, undo log và Read View như thế nào?
- binlog, redo log và undo log lần lượt giải quyết vấn đề nào trong replication, crash recovery và transaction rollback?
- MySQL khôi phục đến một thời điểm cụ thể thông qua full backup, binlog và PITR như thế nào?
- Có những phương án nào để đồng bộ dữ liệu MySQL sang Elasticsearch? Xử lý full sync, incremental sync, out-of-order data và data reconciliation như thế nào?
- Nên định vị slow SQL theo từng lớp từ table creation, index, cách viết SQL và execution plan như thế nào?

## Thứ tự đọc đề xuất

1. [Tổng hợp câu hỏi phỏng vấn MySQL thường gặp](./mysql-questions-01.md): trước tiên lập danh sách các vấn đề thường gặp về MySQL.
2. [Giải thích chi tiết về MySQL index](./mysql-index.md), [Tổng hợp các trường hợp MySQL index mất hiệu lực](./mysql-index-invalidation.md): tìm hiểu nguyên lý index và các trường hợp thường gặp khiến index mất hiệu lực.
3. [Giải thích chi tiết về MySQL transaction isolation level](./transaction-isolation-level.md), [Cách InnoDB storage engine triển khai MVCC](./innodb-implementation-of-mvcc.md): nắm vững transaction và consistent read.
4. [Giải thích chi tiết về ba log lớn của MySQL](./mysql-logs.md), [Giải thích chi tiết về MySQL backup và restore](./mysql-backup-and-restore.md), [Giải thích chi tiết về đồng bộ dữ liệu MySQL sang Elasticsearch](./mysql-to-elasticsearch-sync.md): tìm hiểu vai trò của binlog trong commit, recovery, replication và data synchronization không đồng nhất.
5. [Quy trình thực thi SQL statement trong MySQL](./how-sql-executed-in-mysql.md): tìm hiểu connector, parser, optimizer, executor và storage engine phối hợp với nhau như thế nào.
6. [Phân tích MySQL execution plan](./mysql-query-execution-plan.md), [Tổng hợp đề xuất về quy chuẩn tối ưu performance cao cho MySQL](./mysql-high-performance-optimization-specification-recommendations.md): đưa nguyên lý vào slow SQL và quy chuẩn engineering.

## Bài viết cốt lõi

### Tổng quan và quy chuẩn

- [Tổng hợp câu hỏi phỏng vấn MySQL thường gặp](./mysql-questions-01.md): kết nối các nội dung thường gặp như index, transaction, lock, log, storage engine và SQL optimization.
- [Tổng hợp đề xuất về quy chuẩn tối ưu performance cao cho MySQL](./mysql-high-performance-optimization-specification-recommendations.md): tổng hợp đề xuất tối ưu từ góc độ table creation, field, index, SQL, transaction và quy chuẩn development.
- [Ghi chú học MySQL một nghìn dòng](./a-thousand-lines-of-mysql-study-notes.md): phù hợp để bổ sung những phần còn thiếu và nhanh chóng ôn lại các điểm kiến thức thường gặp về MySQL.

### Index và execution plan

- [Giải thích chi tiết về MySQL index](./mysql-index.md): tìm hiểu data structure của index, clustered index, secondary index, covering index, leftmost prefix và thiết kế index.
- [Tổng hợp các trường hợp MySQL index mất hiệu lực](./mysql-index-invalidation.md): tổng hợp các cách viết thường khiến index mất hiệu lực và hướng điều tra.
- [Index MySQL mất hiệu lực do implicit conversion](./index-invalidation-caused-by-implicit-conversion.md): tập trung vào vấn đề index mất hiệu lực do implicit type conversion.
- [Phân tích MySQL execution plan](./mysql-query-execution-plan.md): nắm vững các field quan trọng như `type`, `key`, `rows`, `Extra` của `EXPLAIN`.

### Transaction, MVCC, log và data synchronization

- [Giải thích chi tiết về MySQL transaction isolation level](./transaction-isolation-level.md): tìm hiểu read uncommitted, read committed, repeatable read, serializable và các bất thường khi đọc đồng thời.
- [Cách InnoDB storage engine triển khai MVCC](./innodb-implementation-of-mvcc.md): tìm hiểu hidden field, undo log, Read View và việc xác định visibility.
- [Giải thích chi tiết về ba log lớn của MySQL](./mysql-logs.md): tìm hiểu vai trò, thời điểm ghi và two-phase commit của binlog, redo log và undo log.
- [Giải thích chi tiết về MySQL backup và restore](./mysql-backup-and-restore.md): tìm hiểu mysqldump, XtraBackup, binlog, PITR, RTO/RPO và diễn tập restore.
- [Giải thích chi tiết về đồng bộ dữ liệu MySQL sang Elasticsearch](./mysql-to-elasticsearch-sync.md): so sánh double write ở application layer, đồng bộ định kỳ, Canal, Debezium và Flink CDC, đồng thời tìm hiểu các vấn đề về full sync, incremental sync và eventual consistency.

### Quy trình thực thi và chi tiết engineering

- [Quy trình thực thi SQL statement trong MySQL](./how-sql-executed-in-mysql.md): tìm hiểu connector, query cache, parser, optimizer, executor và storage engine phối hợp với nhau như thế nào.
- [Giải thích chi tiết về MySQL query cache](./mysql-query-cache.md): tìm hiểu cách query cache hoạt động, nguyên nhân mất hiệu lực và bối cảnh bị loại bỏ.
- [Auto-increment primary key của MySQL có nhất thiết liên tục không?](./mysql-auto-increment-primary-key-continuous.md): tìm hiểu mối quan hệ giữa việc phân bổ auto-increment value, rollback, batch insert và tính liên tục của primary key.
- [Đề xuất lựa chọn date type của MySQL](./some-thoughts-on-database-storage-time.md): so sánh các trường hợp sử dụng của những type như `DATE`, `DATETIME` và `TIMESTAMP`.

## Câu hỏi thường gặp

- Tại sao MySQL khuyến nghị sử dụng B+ tree index?
- Clustered index và secondary index khác nhau như thế nào? Truy ngược về table và covering index là gì?
- Nguyên tắc leftmost prefix là gì? Những trường hợp nào sẽ khiến index mất hiệu lực?
- Làm thế nào để thông qua `EXPLAIN` đánh giá một SQL có sử dụng index phù hợp hay không?
- Bốn transaction isolation level của MySQL lần lượt giải quyết vấn đề gì?
- MVCC được triển khai như thế nào? Read View có những field quan trọng nào?
- binlog, redo log và undo log khác nhau như thế nào? Two-phase commit giải quyết vấn đề gì?
- MySQL thực hiện point-in-time recovery thông qua full backup và binlog như thế nào?
- Có những phương án nào để đồng bộ dữ liệu MySQL sang Elasticsearch? Làm thế nào để bảo đảm idempotent và xử lý message out-of-order?
- Nên bắt đầu tối ưu slow SQL từ những khía cạnh nào?
- Tại sao auto-increment primary key không nhất thiết liên tục?
- Nên lựa chọn `DATETIME` và `TIMESTAMP` như thế nào?

## Chuyên đề liên quan

- [Hệ thống kiến thức Database](../)
- [Chuyên đề SQL](../sql/)
- [Hệ thống kiến thức về high-performance](../../high-performance/)
- [Hệ thống kiến thức về high-availability](../../high-availability/)

<!-- @include: @article-footer.snippet.md -->
