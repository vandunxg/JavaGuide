---
title: "Giải thích chi tiết đồng bộ dữ liệu từ MySQL sang Elasticsearch: các phương án thường gặp và xử lý consistency"
description: "Giải thích chi tiết các phương án đồng bộ dữ liệu từ MySQL sang Elasticsearch, so sánh double-write ở application layer, đồng bộ định kỳ, Canal, Debezium và Flink CDC, đồng thời giới thiệu đồng bộ toàn bộ, đồng bộ incremental, message không đúng thứ tự, idempotent, xóa, rebuild và đối soát dữ liệu."
category: Database
tag:
  - MySQL
  - Elasticsearch
  - Đồng bộ dữ liệu
head:
  - - meta
    - name: keywords
      content: MySQL đồng bộ Elasticsearch,MySQL đồng bộ ES,đồng bộ MySQL ES,Canal đồng bộ ES,Flink CDC đồng bộ ES,Debezium,CDC,binlog,đồng bộ toàn bộ,đồng bộ incremental,data consistency
---

MySQL mạnh về xử lý transaction, còn Elasticsearch mạnh về full-text search và complex search. Nhiều hệ thống dùng MySQL làm nguồn dữ liệu authoritative, sau đó tập hợp các field cần cho search thành document và ghi vào Elasticsearch.

Sau khi làm như vậy, vấn đề sẽ tập trung ở các khâu sau:

- Khi đưa hệ thống lên production lần đầu, làm thế nào import dữ liệu hiện có trong MySQL?
- Khi MySQL phát sinh thêm, sửa và xóa về sau, làm thế nào liên tục cập nhật ES?
- Khi sync task consume trùng message, không đúng thứ tự hoặc bị gián đoạn, làm thế nào tránh dữ liệu cũ ghi đè dữ liệu mới?
- Khi điều chỉnh cấu trúc index hoặc dữ liệu không nhất quán, làm thế nào rebuild và chuyển đổi êm?

Một phương án sync khả dụng thường phải đồng thời cân nhắc **đồng bộ toàn bộ, đồng bộ incremental, khôi phục sau lỗi và kiểm tra dữ liệu**. Chỉ giải quyết một khâu rất dễ bị mắc kẹt khi đưa lên production hoặc khôi phục sau sự cố.

## Xác định mục tiêu đồng bộ trước

Đồng bộ từ MySQL sang Elasticsearch thường được chia thành hai loại:

- **Đồng bộ toàn bộ**: đọc dữ liệu đã có trong MySQL và xây dựng một ES index hoàn chỉnh. Thường dùng khi đưa hệ thống lên production lần đầu, rebuild index và khắc phục dữ liệu.
- **Đồng bộ incremental**: liên tục capture `INSERT`, `UPDATE`, `DELETE` sau khi MySQL commit, rồi cập nhật ES document tương ứng.

Hai loại này thường được kết hợp. Ví dụ, trước tiên import toàn bộ 10 triệu bản ghi lịch sử, sau đó tiếp tục consume incremental từ binlog position được ghi lại lúc bắt đầu task đồng bộ toàn bộ cho đến khi bắt kịp dữ liệu mới nhất.

Cũng cần lưu ý rằng ghi thành công vào ES không có nghĩa document đã lập tức có thể được search. Elasticsearch dùng `refresh` để document mới ghi có thể được search, vì vậy latency end-to-end ít nhất bao gồm các khâu capture thay đổi, xếp message vào hàng đợi, xử lý dữ liệu, ghi vào ES và `refresh`. Tài liệu chính thức gọi khả năng hiển thị khi search này là [near-real-time search](https://www.elastic.co/docs/manage-data/data-store/near-real-time-search).

![Kết nối giữa đồng bộ toàn bộ và incremental từ MySQL sang ES](https://oss.javaguide.cn/github/javaguide/database/es/mysql-es-full-incremental-sync.webp)

## Xác định sync contract trước khi chọn phương án

Cùng một công cụ sync nhưng đặt trong các data model và yêu cầu consistency khác nhau có thể cho kết quả hoàn toàn khác nhau. Trước khi chọn phương án, hãy ghi rõ các quy ước sau:

- **Nguồn dữ liệu authoritative**: thường lấy MySQL làm chuẩn, ES chỉ lưu search view có thể rebuild. Business không được bỏ qua MySQL để sửa các field authoritative trong ES.
- **Quy tắc tạo document**: xác định những table và field nào tạo thành một ES document, `_id` của document lấy từ đâu, thay đổi ở nhiều table sẽ rebuild những document nào.
- **Semantics của việc xóa**: xóa vật lý, xóa logic và hết hiệu lực theo business lần lượt tương ứng với ES delete, giữ lại cờ xóa hay tạo lại document.
- **Thứ tự và version**: xác định partition key và nguồn version của cùng một document, tránh retry, concurrency và nhiều chain khiến value cũ ghi đè value mới.
- **Mục tiêu latency và recovery**: quy ước latency tối đa được phép trong điều kiện bình thường, sau khi chain gián đoạn mất bao lâu để bắt kịp, khi binlog hoặc thời gian lưu giữ của MQ đã hết thì khởi tạo lại như thế nào.
- **Kiểu field và giá trị rỗng**: xác định amount, số nguyên unsigned, datetime, enum và empty string được ghi vào ES như thế nào; phân biệt field bị thiếu, `null` và giá trị rỗng.
- **Ngân sách tài nguyên database nguồn**: đánh giá số connection, concurrency và disk I/O của việc scan toàn bộ, xác định đọc từ primary hay read replica, đồng thời đặt điều kiện giới hạn tốc độ và tạm dừng cho sync task.
- **Quy trình thay đổi Schema**: khi thêm field, sửa type hoặc điều chỉnh analyzer, ai cập nhật mapping, application, sync task và index template được release theo thứ tự nào.

Việc nghiệm thu cũng không thể chỉ kiểm tra “số lượng bản ghi toàn bộ bằng nhau”. Ít nhất phải bao phủ việc thêm, sửa, xóa vật lý, xóa logic, message trùng và message không đúng thứ tự; đồng thời chủ động tạm dừng ES hoặc sync task để quan sát sau khi khôi phục có thể bắt kịp hay không. Việc nạp lại toàn bộ vào index mới, bắt kịp incremental, kiểm tra dữ liệu, chuyển alias và rollback cũng phải được thực hiện đầy đủ một lần. Mỗi mục trong sync contract đều phải tương ứng với một test case hoặc metric giám sát.

## So sánh các phương án thường gặp

| Phương án                        | Đồng bộ toàn bộ | Đồng bộ incremental        | Ưu điểm chính                                                                   | Vấn đề chính                                                   | Trường hợp sử dụng                                              |
| -------------------------------- | --------------- | -------------------------- | ------------------------------------------------------------------------------- | -------------------------------------------------------------- | --------------------------------------------------------------- |
| Double-write ở application layer | Không hỗ trợ    | Hỗ trợ                     | Dễ triển khai, chain ngắn                                                       | Không thể atomic commit, can thiệp vào business, dễ sai thứ tự | Business đơn giản, data nhỏ, chấp nhận không nhất quán tạm thời |
| Local message table + MQ         | Không hỗ trợ    | Hỗ trợ                     | Update business và lưu event có thể cùng transaction                            | Vẫn cần import toàn bộ, component gửi và consume               | Application đã dùng event-driven architecture                   |
| Đồng bộ định kỳ                  | Hỗ trợ          | Hỗ trợ scan theo điều kiện | Đơn giản, chi phí vận hành thấp                                                 | Latency cao, việc xóa và cửa sổ scan cần xử lý thêm            | Data nhỏ, tần suất update thấp                                  |
| Canal + consumer                 | Cần xử lý riêng | Hỗ trợ                     | Ecosystem trưởng thành, dễ tích hợp MQ và custom consumer                       | Cần duy trì position, consumer và kết nối với đồng bộ toàn bộ  | Đã có Java/MQ infrastructure để đồng bộ incremental             |
| Debezium + Kafka                 | Hỗ trợ snapshot | Hỗ trợ                     | Ecosystem Kafka Connect hoàn thiện, format event thống nhất                     | Nhiều component, phụ thuộc Kafka Connect                       | Đã có Kafka Connect hoặc nền tảng CDC nhiều data source         |
| Flink CDC                        | Hỗ trợ          | Hỗ trợ                     | Tự động kết nối đồng bộ toàn bộ và incremental, phù hợp transformation phức tạp | Chi phí deploy và quản lý state cao hơn                        | Data lớn, cần stream processing hoặc chuyển đổi nhiều table     |

“Hỗ trợ” trong bảng chỉ cho biết tool cung cấp capability tương ứng, không có nghĩa sau khi tích hợp thì business tự nhiên đạt yêu cầu consistency. Hiệu quả cuối cùng còn phụ thuộc vào cách ghi ES, thứ tự message, Checkpoint, retry strategy và thiết kế index.

## Double-write ở application layer

Double-write ở application layer là sau khi business code update MySQL, nó tiếp tục gọi Elasticsearch API để update index.

![Double-write đồng bộ ở application layer](https://oss.javaguide.cn/github/javaguide/database/es/es-mq-synchronization-synchronous-double-write.png)

Giả sử thứ tự ghi là “MySQL trước, ES sau”, các kết quả thường gặp như sau:

| Tình huống                                 | MySQL      | Elasticsearch    | Kết quả                           |
| ------------------------------------------ | ---------- | ---------------- | --------------------------------- |
| Cả hai lần ghi đều thành công              | Thành công | Thành công       | Dữ liệu nhất quán                 |
| Ghi MySQL thất bại                         | Rollback   | Không thực hiện  | Dữ liệu nhất quán                 |
| Application crash sau khi MySQL thành công | Thành công | Chưa thực hiện   | Dữ liệu không nhất quán           |
| MySQL thành công, ghi ES thất bại          | Thành công | Thất bại         | Dữ liệu không nhất quán           |
| ES đã ghi nhưng client timeout             | Thành công | Không rõ kết quả | Khi retry phải bảo đảm idempotent |

Transaction của MySQL không thể rollback việc ghi đã commit vào Elasticsearch, và Elasticsearch cũng không thể cùng MySQL tạo thành một local transaction thông thường. Vì vậy, double-write đồng bộ thường chỉ có thể hướng tới eventual consistency.

Double-write trực tiếp còn có hai vấn đề dễ bị bỏ sót.

Thứ nhất là performance và availability. Business request phải chờ thêm một lần gọi ES, ES chập chờn cũng ảnh hưởng đến main flow. Nếu đổi sang gửi bất đồng bộ bằng thread pool, `CompletableFuture` hoặc MQ thông thường, request latency sẽ giảm, nhưng khoảng trống “MySQL đã commit, process crash khi message chưa được gửi” vẫn tồn tại.

Thứ hai là sai thứ tự do concurrency. Giả sử cùng một product record lần lượt được update thành `v2`, `v3`, thứ tự hai ES request đến nơi có thể bị đảo ngược, cuối cùng khiến `v2` cũ ghi đè `v3`. Chỉ chuyển việc ghi sang bất đồng bộ không tự động giải quyết vấn đề thứ tự.

### Dùng local message table để bù khoảng trống transaction

Khi yêu cầu về độ tin cậy của việc sync cao hơn, có thể dùng **Transactional Outbox (local message table)**:

1. Trong cùng một MySQL transaction, update business table và insert một outbox event.
2. Một delivery program độc lập hoặc CDC component đọc outbox table rồi gửi event đến MQ.
3. ES consumer xử lý message, sau khi thành công thì ghi lại kết quả consume; nếu thất bại thì retry hoặc đưa vào dead-letter queue.

Cách này bảo đảm “update business data” và “lưu event chờ gửi” cùng thành công hoặc cùng rollback. Debezium cũng cung cấp [Outbox Event Router](https://debezium.io/documentation/reference/stable/transformations/outbox-event-router.html) chuyên dụng để capture và transform outbox event.

Local message table giải quyết vấn đề mất event, nhưng vẫn phải xử lý consume trùng, thứ tự message của cùng business primary key, khởi tạo dữ liệu toàn bộ và đối soát dữ liệu.

## Đồng bộ định kỳ

Nếu data không lớn, tần suất update thấp và business chấp nhận latency từ vài phút đến thậm chí vài ngày, scheduled task thường đã đủ dùng. Blog cá nhân, knowledge base nội bộ và chức năng search của hệ thống quản trị nhỏ là các trường hợp điển hình.

![Đồng bộ định kỳ](https://oss.javaguide.cn/github/javaguide/database/es/es-mq-synchronization-scheduled-task.png)

Đồng bộ định kỳ không nhất thiết lần nào cũng phải scan toàn bộ, mà có thể query incremental theo thời gian update:

```sql
SELECT id, title, content, updated_at
FROM article
WHERE (
        updated_at > :last_updated_at
        OR (updated_at = :last_updated_at AND id > :last_id)
      )
  AND updated_at <= :task_cutoff
ORDER BY updated_at, id
LIMIT :batch_size;
```

Khi task bắt đầu, trước tiên cố định `task_cutoff`; sau khi xử lý xong mỗi batch thì lưu composite cursor gồm `last_updated_at` và `last_id`. Batch tiếp theo tiếp tục dùng cùng thời điểm cutoff cho đến khi xử lý xong toàn bộ dữ liệu của lượt này. Cách này vừa tránh deep pagination, vừa không bỏ sót dữ liệu vì nhiều record có cùng `updated_at`.

Xóa vật lý sẽ không để lại `updated_at`, vì vậy đồng bộ incremental định kỳ thường phải kết hợp với field xóa logic, bảng record đã xóa hoặc kiểm tra toàn bộ định kỳ. Nếu không, dữ liệu đã xóa trong MySQL có thể tồn tại lâu dài trong ES.

Khi dùng Spring Task, có thể viết task chạy một lần vào 0 giờ thứ Hai hằng tuần như sau:

```java
@Scheduled(cron = "0 0 0 ? * MON", zone = "Asia/Shanghai")
public void rebuildArticleIndex() {
    // Đọc MySQL theo trang, ghi hàng loạt vào ES index mới
}
```

### Dùng index alias khi rebuild toàn bộ

Không nên vừa xóa vừa ghi trên index cũ mà user đang query. Cách an toàn hơn là:

1. Tạo index mới có version, chẳng hạn `articles_v2`, đồng thời cấu hình trước `settings` và `mappings`.
2. Ghi toàn bộ dữ liệu MySQL vào index mới, hoàn tất kiểm tra số lượng, nội dung lấy mẫu và kết quả query.
3. Bù các thay đổi incremental phát sinh trong thời gian task đồng bộ toàn bộ chạy.
4. Dùng `_aliases` API để chuyển business alias từ index cũ sang index mới một cách atomic.
5. Giữ index cũ trong một khoảng thời gian, sau khi xác nhận không có vấn đề mới dọn dẹp.

```http
POST /_aliases
{
  "actions": [
    { "remove": { "index": "articles_v1", "alias": "articles", "must_exist": true } },
    { "add": { "index": "articles_v2", "alias": "articles" } }
  ]
}
```

[Aliases API](https://www.elastic.co/guide/en/elasticsearch/reference/current/aliases.html) của Elasticsearch có thể thực hiện nhiều thay đổi alias trong một atomic operation. Tuy nhiên, chuyển alias chỉ giải quyết việc chuyển read traffic êm; nếu trong thời gian rebuild vẫn có business write, vẫn phải bù các thay đổi trong khoảng thời gian này bằng cách replay binlog, ghi vào hai index hoặc tạm dừng write.

## CDC dựa trên binlog

Phương án CDC (Change Data Capture, capture thay đổi dữ liệu) đọc MySQL binlog và chuyển các thay đổi dữ liệu đã commit thành event mà downstream có thể consume. Phương án này không cần duy trì logic ghi ES trong từng business interface, hiện cũng là cách đồng bộ incremental phổ biến hơn.

Trước khi tích hợp cần kiểm tra các cấu hình sau:

- MySQL đã bật binlog và cấp cho CDC account các quyền replication và đọc table cần thiết.
- Dùng format binlog mà tool hỗ trợ. Các tool thay đổi theo row như Canal, Debezium và Flink CDC thường yêu cầu `binlog_format=ROW`; Debezium còn yêu cầu rõ `binlog_row_image=FULL`, các tool khác cũng phải kiểm tra cấu hình này theo version đang dùng. `binlog_format` của MySQL 8.4 mặc định là `ROW`, nhưng môi trường cũ và cloud database vẫn phải lấy cấu hình thực tế làm chuẩn.
- Thời gian lưu giữ binlog bao phủ cửa sổ khôi phục sự cố dài nhất. Khi task dừng quá lâu và binlog cần dùng đã bị dọn, thường chỉ có thể thực hiện lại snapshot hoặc đồng bộ toàn bộ.
- Mỗi CDC client dùng `server_id` duy nhất; khi chuyển đổi high availability còn phải kiểm tra GTID, khôi phục position và cấu hình binlog của replica đang kết nối.
- CDC account chỉ được cấp các quyền replication, metadata và đọc target table theo yêu cầu trong tài liệu của tool; ES account giới hạn trong phạm vi write của target index hoặc alias; không ghi credential trực tiếp vào task config được commit vào codebase.

### Canal

[Canal](https://github.com/alibaba/canal) mô phỏng giao thức tương tác của MySQL Replica, request binlog từ MySQL và parse thành các event thay đổi có cấu trúc.

![Nguyên lý hoạt động của Canal](https://oss.javaguide.cn/github/javaguide/open-source-project/canal-overview.png)

Trách nhiệm cốt lõi của Canal Server là subscribe và parse incremental log. Downstream có thể dùng Canal Client để consume trực tiếp, hoặc để Canal gửi message vào Kafka, RocketMQ và các MQ khác, sau đó sync service update ES.

![Đồng bộ dữ liệu qua MQ bằng Canal](https://oss.javaguide.cn/github/javaguide/database/es/es-mq-synchronization-canal-with-mq.png)

Đưa MQ vào chủ yếu có các tác dụng sau:

- Tách Canal và ES consumer, khi ES tạm thời không khả dụng thì thay đổi có thể tạm tích tụ trong MQ.
- Cùng một business primary key có thể được route vào cùng một partition, thuận tiện duy trì thứ tự cục bộ.
- Consumer có thể gộp message và gọi ES Bulk API, giảm chi phí request.
- Sau khi giữ message trong một khoảng thời gian nhất định, có thể replay dữ liệu thất bại hoặc rebuild downstream.

MQ không tự động mang lại consistency. Consumer vẫn phải xác nhận khi nào commit message, retry khi thất bại như thế nào, retry có idempotent hay không và ai sửa dữ liệu dead letter.

Canal Server chủ yếu phụ trách subscribe incremental. Dữ liệu ban đầu có thể được import bằng pagination task tự phát triển hoặc batch sync tool đã kiểm tra compatibility. [Bản open source của DataX](https://github.com/alibaba/DataX/tree/master/elasticsearchwriter) có cung cấp ElasticsearchWriter, nhưng README chính thức vẫn ghi rõ mới chỉ test trên Elasticsearch 5.x; trước khi tích hợp với ES 7/8 bắt buộc phải kiểm tra mapping type, client protocol và tham số ghi. Canal Client Adapter cũng từng cung cấp capability ETL cho một số downstream; nó có phù hợp với version ES hiện tại và mapping phức tạp hay không cũng phải kiểm tra theo version thực tế. Khi đồng bộ toàn bộ và incremental do hai task đảm nhiệm, điều quan trọng nhất là ghi lại và kết nối đúng binlog position.

### Flink CDC

MySQL Source của Flink CDC có thể đọc snapshot của table trước, sau đó tiếp tục consume binlog. Nó chia table thành nhiều Chunk theo shard key và đọc snapshot song song; trước và sau khi đọc mỗi Chunk, nó ghi lại position `LOW`, `HIGH` của binlog, sau đó hợp nhất các thay đổi phát sinh trong khoảng thời gian này để có kết quả Chunk nhất quán.

Thuật toán incremental snapshot này có một số đặc điểm:

- Giai đoạn snapshot có thể được đọc song song.
- Checkpoint được thực hiện theo Chunk; sau khi thất bại có thể khôi phục từ progress đã hoàn thành.
- Không cần giữ global read lock lấy bằng `FLUSH TABLES WITH READ LOCK` trong toàn bộ giai đoạn snapshot.
- Sau khi snapshot toàn bộ kết thúc, tự động tiếp tục đọc binlog, giảm công việc nối tiếp position thủ công.

![Kết nối incremental snapshot với binlog của Flink CDC](https://oss.javaguide.cn/github/javaguide/database/es/flink-cdc-incremental-snapshot.webp)

Tính đến Flink CDC 3.6, tài liệu chính thức cung cấp [MySQL Pipeline Connector](https://nightlies.apache.org/flink/flink-cdc-docs-release-3.6/docs/connectors/pipeline-connectors/mysql/) và [Elasticsearch Pipeline Connector](https://nightlies.apache.org/flink/flink-cdc-docs-release-3.6/docs/connectors/pipeline-connectors/elasticsearch/), có thể trực tiếp mô tả Pipeline từ MySQL đến ES. ES Sink không tự động tạo index, trước khi dùng vẫn phải chuẩn bị index, mapping và các template liên quan.

Exactly-once trong tài liệu Flink CDC chủ yếu mô tả semantics xử lý của Source trong giai đoạn đọc snapshot và binlog. Tài liệu Elasticsearch Sink chính thức của Flink đưa ra delivery guarantee cơ bản là at-least-once: Checkpoint sẽ chờ các ES request đã gửi hoàn tất, nhưng sau khi khôi phục lỗi vẫn có thể replay request. Dùng document ID cố định và `index/upsert` idempotent có thể khiến việc ghi trùng rơi vào cùng một document; nếu cùng document còn có thể sai thứ tự thì cần kết hợp thêm version từ source. Exactly-once của Source không thể trực tiếp suy ra exactly-once end-to-end từ MySQL đến ES.

### Debezium

MySQL Connector của Debezium thường chạy trên Kafka Connect. Khi khởi động lần đầu, nó có thể thực hiện consistency snapshot trước, sau đó liên tục gửi row-level change event tương ứng với binlog position của snapshot vào Kafka Topic. Về sau cũng có thể trigger incremental snapshot theo từng block khi cần.

Nếu team đã dùng Kafka Connect hoặc chuẩn bị đưa thay đổi từ nhiều database vào Kafka thống nhất, Debezium sẽ thuận tiện hơn so với tự phát triển parser binlog và event format. Khi ghi vào ES vẫn cần Elasticsearch Sink Connector hoặc custom consumer; delete event, Topic compaction, Schema change và quy tắc transform message đều phải được thiết kế riêng.

Có thể tham khảo quy trình snapshot và các mục config cụ thể trong [tài liệu chính thức Debezium MySQL Connector](https://debezium.io/documentation/reference/stable/connectors/mysql.html).

## Các vấn đề bắt buộc xử lý trong production

Việc chọn tool chỉ quyết định thay đổi đến downstream bằng cách nào. Dữ liệu có giữ được tính đúng hay không còn phụ thuộc vào các chi tiết triển khai sau.

![Các biện pháp bảo đảm consistency khi đồng bộ MySQL sang ES](https://oss.javaguide.cn/github/javaguide/database/es/mysql-es-consistency-guardrails.webp)

### 1. Dùng primary key MySQL làm ES document ID

Khi sync nên dùng business primary key ổn định và duy nhất làm ES `_id`. Khi thực hiện lại `index` hoặc upsert cùng một dữ liệu, cùng một document sẽ được update, nhờ đó consumer dễ triển khai idempotent hơn.

Nếu để ES tự tạo `_id`, retry message có thể ghi ra nhiều document trùng lặp, khi xóa cũng rất khó tìm đúng target.

### 2. Bảo đảm thứ tự event của cùng một document

Thứ tự toàn cục thường có chi phí rất cao, đồng bộ MySQL sang ES đa số chỉ cần bảo đảm event của cùng một document ID có thứ tự. Cách thường dùng là chọn MQ partition theo business primary key, để event của cùng một primary key vào cùng partition và được xử lý tuần tự.

Ngay cả như vậy, rebalance, retry khi thất bại và nhiều sync chain vẫn có thể tạo ra event không đúng thứ tự. Với trường hợp không thể chấp nhận value cũ ghi đè value mới, có thể mang business version tăng đơn điệu trong event và ES document, rồi loại bỏ version cũ trước khi ghi. Version phải đến từ nguồn thứ tự đáng tin cậy, không thể trực tiếp dùng local time do nhiều machine tự tạo để thay thế.

Nếu version ở source có thể biểu diễn bằng `long` không âm, `index` và `delete` operation của [ES Bulk API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-bulk) hỗ trợ thiết lập `version` và `version_type=external`: chỉ write có version cao hơn mới có thể ghi đè document hiện tại. Delete event cũng phải mang cùng bộ version, nếu không update cũ đến muộn vẫn có thể ghi lại document đã xóa. GTID, tên file binlog và position thường không thể dùng trực tiếp làm external version của ES nếu chưa chuyển đổi; trước khi triển khai cần định nghĩa quy tắc so sánh ổn định.

### 3. Xử lý xóa đúng cách

`INSERT` và `UPDATE` cuối cùng có thể thống nhất thành upsert, còn `DELETE` bắt buộc phải được chuyển rõ ràng thành ES delete operation. Nếu business dùng xóa logic, cũng phải thống nhất là xóa ES document hay giữ document và ghi `deleted=true`.

Nếu sync task filter dữ liệu theo các field thay đổi như `status`, `is_deleted`, còn phải xử lý trường hợp “trước đây thỏa điều kiện nhưng sau khi update thì không còn”. Filter thông thường chỉ loại bỏ update event mới, không tự động thêm delete cho document cũ trong ES. Có thể giữ field trạng thái và filter khi query, hoặc nhận biết state transition rồi tạo delete event rõ ràng.

Document tổng hợp từ nhiều table đặc biệt dễ bỏ sót việc xóa. Ví dụ product document chứa field từ product table, brand table và category table; sau khi xóa một category, không thể chỉ xóa “ES document tương ứng với category ID”, mà phải tìm các product bị ảnh hưởng và rebuild document của chúng.

### 4. Xử lý document từ nhiều table

Binlog ghi lại thay đổi của các row trong table, còn ES thường dùng document denormalized. Một product document có thể kết hợp dữ liệu từ nhiều table như product, brand, inventory và tag.

Khi CDC capture việc brand name thay đổi, cần biết những product nào đang tham chiếu brand đó. Có ba cách xử lý thường gặp:

- Consumer query MySQL theo change event, rồi assemble lại toàn bộ document bị ảnh hưởng.
- Duy trì wide table phù hợp cho search trong MySQL trước, sau đó sync wide table.
- Dùng stream processing engine như Flink để duy trì related state và tạo downstream document.

Cách thứ nhất dễ triển khai nhưng phải kiểm soát concurrency và cache khi query lại MySQL; cách thứ hai làm tăng chi phí duy trì wide table; cách thứ ba phù hợp với trường hợp data và logic liên kết phức tạp hơn, nhưng state, Checkpoint và nâng cấp job đều cần vận hành chuyên biệt.

### 5. Batch write cũng phải kiểm tra kết quả từng item

Elasticsearch [Bulk API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html) có thể thực hiện nhiều `index`, `update` và `delete` trong một request, giảm chi phí network và xử lý request.

Trong một Bulk request có thể chỉ một phần document thất bại. Consumer không được chỉ kiểm tra HTTP status code mà phải kiểm tra từng `items`, chỉ retry các lỗi tạm thời như network timeout, 429 và một số lỗi 5xx. Các vấn đề như mapping conflict và field format error thường không có ý nghĩa nếu tiếp tục retry; cần ghi lại message gốc và chuyển vào dead-letter queue hoặc quy trình sửa thủ công.

Bulk batch cũng không có kích thước cố định phù hợp với mọi hệ thống. Số lượng item, số byte, concurrency và tần suất refresh cần được điều chỉnh dựa trên document size, số lượng shard, ES write queue và kết quả load test.

Khi tốc độ ghi ES thấp hơn tốc độ thay đổi upstream, cần tạo backpressure lên upstream: giới hạn số Bulk request đang xử lý và concurrency của consumer, khi cần thì tạm dừng consume hoặc giảm tốc độ đọc của Source. Không dùng unbounded memory queue để hấp thụ backlog; nó chỉ biến write bottleneck của ES thành memory failure của sync process. Khi dùng MQ, cũng phải bảo đảm thời gian lưu message bao phủ thời gian backlog dài nhất dự kiến.

### 6. Quản lý mapping và thay đổi DDL

MySQL thêm field không có nghĩa ES mapping nên tự động thêm field cùng tên. Dynamic mapping có thể nhận diện date, number hoặc string thành type sai, cũng có thể gây mapping explosion vì số lượng field mất kiểm soát.

Field mapping phải được thiết kế theo cách query, không thể chỉ chuyển đổi máy móc theo MySQL type:

- Tên, title và description cần full-text search nên dùng `text`; khi còn cần sort, aggregation hoặc filter chính xác thì thêm sub-field `keyword`. Status code, tag, email và các structured string thường dùng trực tiếp `keyword`.
- Không chuyển amount vô điều kiện thành `float` hoặc `double`. Có thể giữ integer của đơn vị tiền tệ nhỏ nhất, hoặc dùng `scaled_float` theo độ chính xác cho phép; cả hai cách đều phải cố định quy tắc quy đổi trong sync contract.
- Giới hạn trên của MySQL `BIGINT UNSIGNED` vượt quá ES `long`, giá trị lớn phải dùng `unsigned_long` hoặc chuyển thành `keyword`, không được âm thầm overflow trong quá trình serialize.
- MySQL `TIMESTAMP` chuyển đổi qua lại theo session timezone và UTC, còn `DATETIME` không tự động chuyển đổi tương tự. Sync chain phải cố định session timezone của source và format ngày của ES, đồng thời dùng các sample ở nhiều timezone và quanh thời điểm daylight saving time để kiểm tra kết quả.
- Phân biệt field bị thiếu, `null`, empty string và empty array. Việc chúng có tham gia `exists` query, sort, aggregation và partial update hay không cần được xử lý thống nhất trước khi ghi.

Trong production, nên dùng explicit mapping và index template. Khi field type hoặc analyzer thay đổi không tương thích, hãy tạo index mới và rebuild toàn bộ, sau đó chuyển bằng alias. CDC task cũng phải xác định cách xử lý DDL event của MySQL và thứ tự release của code, index template và sync task.

### 7. Monitor latency, failure và thời gian lưu giữ binlog

Ít nhất nên theo dõi các metric sau:

- Khoảng cách giữa consumer position hiện tại và binlog position mới nhất của MySQL.
- Lượng message backlog trong MQ và thời gian chờ của message cũ nhất.
- Tỷ lệ thành công của ES Bulk, nguyên nhân thất bại từng item, số lần retry và số lượng dead letter.
- Tốc độ đọc, tốc độ ghi và lượng dữ liệu còn lại của task đồng bộ toàn bộ.
- Thời gian lưu giữ binlog còn lại có lớn hơn thời gian backlog hiện tại và thời gian khôi phục sự cố hay không.
- Trạng thái ES cluster, write rejection, disk watermark và refresh latency.

Ngưỡng cảnh báo nên được đặt theo mục tiêu latency của business và baseline từ load test. Việc viết “latency vượt 60 giây” hoặc “tỷ lệ thất bại vượt 1%” thành ngưỡng chung cho mọi hệ thống thường không có cơ sở thực tế.

### 8. Đối soát định kỳ và giữ khả năng rebuild

Message không bị backlog không có nghĩa dữ liệu chắc chắn nhất quán. Quy tắc sync sai, bỏ sót xử lý xóa, mapping conflict và việc sửa ES thủ công đều có thể gây sai lệch âm thầm.

Có thể thực hiện đối soát theo nhiều tầng:

1. So sánh tổng số record hợp lệ của MySQL và ES để nhanh chóng phát hiện chênh lệch rõ ràng.
2. Thống kê số lượng và summary theo khoảng primary key hoặc business partition để thu hẹp phạm vi vấn đề.
3. So sánh mẫu các field quan trọng hoặc tính hash cho document sau khi normalize.
4. Giao các primary key khác biệt cho repair task để tạo lại document tương ứng, tránh sửa ES trực tiếp bằng tay.

Cho dù incremental chain hằng ngày đáng tin cậy đến đâu, vẫn nên giữ khả năng “tạo index mới bằng một lệnh, nạp lại toàn bộ, bắt kịp incremental, kiểm tra và chuyển alias”. Khả năng này vừa dùng để sửa dữ liệu, vừa dùng khi nâng cấp mapping và analyzer.

### 9. Chuẩn bị runbook khôi phục sự cố

Đợi đến khi alert phát ra rồi mới quyết định “nên retry hay nên rebuild” rất dễ làm sự cố lan rộng. Trước khi đưa lên production, cần viết sẵn action khôi phục cho các sự cố thường gặp và diễn tập để xác nhận position, Checkpoint, MQ Offset và index alias đều hoạt động như dự kiến.

| Tình huống sự cố                               | Cách xử lý                                                                                                                                      | Không nên làm trực tiếp                                                        |
| ---------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| ES tạm thời không khả dụng hoặc trả về 429/5xx | Tạm dừng hoặc giới hạn tốc độ ghi, giữ message chưa xác nhận, retry theo backoff strategy, sau khi khôi phục quan sát backlog có giảm hay không | Commit consumer position khi chưa ghi thành công                               |
| Mapping conflict hoặc field format error       | Dừng retry vô hiệu, ghi event gốc và nguyên nhân thất bại, sửa template hoặc quy tắc transform rồi replay có mục tiêu                           | Retry vô hạn cùng một lỗi có tính xác định                                     |
| CDC task restart nhưng state vẫn dùng được     | Khôi phục từ Checkpoint, binlog position hoặc MQ Offset đã lưu, đồng thời dựa vào idempotent write để khử trùng lặp                             | Xóa state rồi khởi động trực tiếp từ position mới nhất                         |
| Binlog hoặc message cần dùng đã hết hạn        | Thực hiện lại consistency snapshot hoặc rebuild toàn bộ, rồi thiết lập lại kết nối giữa đồng bộ toàn bộ và incremental                          | Nhảy đến position mới nhất, tiếp tục chạy và tuyên bố dữ liệu đã nhất quán     |
| ES index hỏng hoặc quy tắc thay đổi toàn bộ    | Tạo index mới, nạp lại toàn bộ, bắt kịp incremental và kiểm tra, sau đó chuyển alias                                                            | Vừa xóa vừa bổ sung trên index cũ mà không giữ điểm fallback cho query traffic |

Runbook cũng phải ghi lại người phụ trách, quyền cần thiết, vị trí của state file hoặc Checkpoint, entry point để replay dead letter và điều kiện rollback. Sau khi diễn tập, kiểm tra trong quá trình khôi phục có xuất hiện áp lực tăng đột biến lên source database, ES write rejection, binlog lại hết hạn hoặc cùng một batch bị replay lặp đi lặp lại hay không.

## Chọn như thế nào?

- Data nhỏ, update không thường xuyên, chấp nhận latency cao: ưu tiên đồng bộ incremental định kỳ, kết hợp rebuild toàn bộ theo chu kỳ.
- Business đã có domain event và MQ đáng tin cậy: có thể dùng local message table + MQ, sync service consume event để update ES.
- Nhu cầu chính là thu nhận ổn định các thay đổi incremental của MySQL, team quen Java và MQ: Canal + MQ + custom consumer thường phù hợp, phần toàn bộ cần thiết kế riêng.
- Đã có Kafka Connect platform hoặc cần thống nhất tích hợp nhiều database: có thể cân nhắc Debezium.
- Data lớn, muốn tự động kết nối đồng bộ toàn bộ và incremental hoặc cần stream cleaning, chuyển đổi nhiều table: có thể cân nhắc Flink CDC.

Cuối cùng, khi chọn phương án không nên chỉ so sánh throughput của tool. Trước tiên xác định search latency mà business có thể chấp nhận, thời gian khôi phục sự cố, số lượng component deploy, độ phức tạp của document nhiều table và năng lực vận hành của team, sau đó load test bằng data size và document structure gần với production.

## Tài liệu tham khảo

- [MySQL 8.4: Thiết lập format binary log](https://dev.mysql.com/doc/refman/8.4/en/binary-log-setting.html)
- [Tài liệu dự án Canal](https://github.com/alibaba/canal)
- [Flink CDC 3.6: MySQL Pipeline Connector](https://nightlies.apache.org/flink/flink-cdc-docs-release-3.6/docs/connectors/pipeline-connectors/mysql/)
- [Flink CDC 3.6: Elasticsearch Pipeline Connector](https://nightlies.apache.org/flink/flink-cdc-docs-release-3.6/docs/connectors/pipeline-connectors/elasticsearch/)
- [Flink: Semantics khôi phục sự cố của Elasticsearch Sink](https://nightlies.apache.org/flink/flink-docs-stable/docs/connectors/datastream/elasticsearch/)
- [Debezium MySQL Connector](https://debezium.io/documentation/reference/stable/connectors/mysql.html)
- [Debezium Outbox Event Router](https://debezium.io/documentation/reference/stable/transformations/outbox-event-router.html)
- [Elasticsearch Bulk API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html)
- [Elasticsearch Aliases](https://www.elastic.co/guide/en/elasticsearch/reference/current/aliases.html)
- [Kiểu field của Elasticsearch](https://www.elastic.co/docs/reference/elasticsearch/mapping-reference/field-data-types)
- [Kiểu field số của Elasticsearch](https://www.elastic.co/docs/reference/elasticsearch/mapping-reference/number)
- [MySQL 8.4: Kiểu DATE, DATETIME và TIMESTAMP](https://dev.mysql.com/doc/refman/8.4/en/datetime.html)
- [Spring Framework: Task Execution and Scheduling](https://docs.spring.io/spring-framework/reference/integration/scheduling.html)

<!-- @include: @article-footer.snippet.md -->
