---
title: Tổng hợp câu hỏi phỏng vấn MongoDB thường gặp (phần trên)
description: Tổng hợp phần trên các câu hỏi phỏng vấn MongoDB thường gặp, giải thích chi tiết các kiến thức cốt lõi như khái niệm cơ bản, cấu trúc lưu trữ, kiểu dữ liệu, replica set high availability, mở rộng theo chiều ngang bằng sharded cluster và các nội dung khác, hỗ trợ chuẩn bị phỏng vấn backend.
category: Database
tag:
  - NoSQL
  - MongoDB
head:
  - - meta
    - name: keywords
      content: câu hỏi phỏng vấn MongoDB,cơ sở dữ liệu document,BSON,replica set,sharded cluster,MongoDB index,WiredTiger,aggregation pipeline
---

> Một phần nhỏ nội dung tham khảo mô tả trong tài liệu chính thức của MongoDB, xin nêu rõ tại đây.

## Kiến thức cơ bản về MongoDB

### MongoDB là gì?

MongoDB là một hệ thống cơ sở dữ liệu NoSQL mã nguồn mở dựa trên **distributed file storage**, được viết bằng **C++**. MongoDB cung cấp phương thức lưu trữ **document-oriented**, thao tác tương đối đơn giản, hỗ trợ mô hình hóa dữ liệu **schema-less**, có thể lưu trữ các kiểu dữ liệu tương đối phức tạp và là một **document database** rất phổ biến.

Trong điều kiện tải cao, MongoDB hỗ trợ mở rộng theo chiều ngang và high availability một cách tự nhiên. Bạn có thể dễ dàng thêm nhiều node/instance để bảo đảm performance và availability của service. Trong nhiều trường hợp, MongoDB có thể thay thế relational database truyền thống hoặc phương thức lưu trữ key/value, nhằm cung cấp giải pháp lưu trữ dữ liệu có khả năng mở rộng, high availability và performance cao cho Web application.

### Cấu trúc lưu trữ của MongoDB là gì?

Cấu trúc lưu trữ của MongoDB khác với relational database truyền thống, chủ yếu gồm ba đơn vị sau:

- **Document**: đơn vị cơ bản nhất trong MongoDB, gồm các cặp key-value BSON, tương tự một row trong relational database.
- **Collection**: một collection có thể chứa nhiều document, tương tự một table trong relational database.
- **Database**: một database có thể chứa nhiều collection. Bạn có thể tạo nhiều database trong MongoDB, tương tự database trong relational database.

Nói cách khác, MongoDB lưu các bản ghi dữ liệu dưới dạng document (cụ thể hơn là [BSON document](https://www.mongodb.com/docs/manual/core/document/#std-label-bson-document-format)). Các document này được tập hợp trong collection, còn database lưu trữ một hoặc nhiều collection document.

**Đối chiếu các thuật ngữ thường gặp giữa SQL và MongoDB**:

| SQL            | MongoDB              |
| -------------- | -------------------- |
| Table          | Collection           |
| Row            | Document             |
| Col            | Field                |
| Primary Key    | Object ID (Objectid) |
| Index          | Index                |
| Embedded Table | Embedded Document    |
| Array          | Array                |

#### Document

Bản ghi trong MongoDB chính là một BSON document, một cấu trúc dữ liệu gồm các cặp key-value, tương tự JSON object và là đơn vị dữ liệu cơ bản trong MongoDB. Giá trị của field có thể bao gồm document khác, array và array document.

![MongoDB document](https://oss.javaguide.cn/github/javaguide/database/mongodb/crud-annotated-document..png)

Key của document là string. Ngoại trừ một số trường hợp đặc biệt, key có thể sử dụng bất kỳ ký tự UTF-8 nào.

- Key không được chứa `\0` (null character). Ký tự này dùng để biểu thị phần kết thúc của key.
- `.` và `$` có ý nghĩa đặc biệt, chỉ được sử dụng trong môi trường cụ thể.
- Key bắt đầu bằng dấu gạch dưới `_` được dành riêng (không bắt buộc nghiêm ngặt).

**BSON [bee·sahn]** là viết tắt của Binary [JSON](http://json.org/), là biểu diễn nhị phân của JSON document. BSON hỗ trợ nhúng document và array vào document và array khác, đồng thời chứa các phần mở rộng cho phép biểu diễn những kiểu dữ liệu không thuộc đặc tả JSON. Có thể tham khảo đặc tả BSON tại [bsonspec.org](http://bsonspec.org/), xem thêm [BSON types](https://www.mongodb.com/docs/manual/reference/bson-types/).

Theo giới thiệu về BJSON trên Wikipedia, tốc độ duyệt BJSON nhanh hơn JSON, đây cũng là lý do chính MongoDB chọn BSON. Tuy nhiên, BJSON cần nhiều storage space hơn.

> So với JSON, BSON hướng đến việc nâng cao hiệu quả lưu trữ và scan. Các phần tử lớn trong BSON document có length field đứng trước để thuận tiện cho việc scan. Trong một số trường hợp, do có length prefix và explicit array index, BSON sử dụng nhiều space hơn JSON.

![Trang chủ BSON](https://oss.javaguide.cn/github/javaguide/database/mongodb/bsonspec.org.png)

#### Collection

MongoDB collection tồn tại trong database và **không có cấu trúc cố định**, tức là **schema-less**. Điều này có nghĩa là bạn có thể insert dữ liệu với format và type khác nhau vào collection. Tuy nhiên, thông thường dữ liệu được insert vào cùng một collection sẽ có một mức độ liên quan nhất định.

![MongoDB collection](https://oss.javaguide.cn/github/javaguide/database/mongodb/crud-annotated-collection.png)

Không cần tạo collection trước. Khi document đầu tiên được insert hoặc index đầu tiên được tạo, nếu collection chưa tồn tại thì một collection mới sẽ được tạo.

Tên collection có thể là bất kỳ string UTF-8 nào thỏa mãn các điều kiện sau:

- Tên collection không được là empty string `""`.
- Tên collection không được chứa `\0` (null character), ký tự biểu thị phần kết thúc của tên collection.
- Tên collection không được bắt đầu bằng `"system."`, đây là prefix dành riêng cho system collection. Ví dụ, collection `system.users` lưu thông tin user của database, collection `system.namespaces` lưu thông tin về tất cả collection trong database.
- Tên collection phải bắt đầu bằng dấu gạch dưới hoặc ký hiệu chữ cái, đồng thời không được chứa `$`.

#### Database

Database dùng để lưu trữ tất cả collection, còn collection dùng để lưu trữ tất cả document. Một MongoDB có thể tạo nhiều database, mỗi database có collection và permission riêng.

MongoDB dành riêng một số database đặc biệt.

- **admin**: database admin chủ yếu lưu root user và role. Ví dụ, table system.users lưu user, table system.roles lưu role. Thông thường không nên thao tác trực tiếp trên database này. Khi thêm một user vào database này và cấp cho user đó role `dbAdminAnyDatabase` trên database admin, user này sẽ tự động kế thừa permission của mọi database. Một số server-side command nhất định cũng chỉ có thể chạy từ database này, chẳng hạn command tắt server.
- **local**: database local không được replicate sang shard khác, vì vậy có thể dùng để lưu bất kỳ collection nào của một server cục bộ. Thông thường không nên dùng local database để lưu dữ liệu và cũng không nên thực hiện thao tác CRUD, vì dữ liệu không thể được backup và restore bình thường.
- **config**: khi MongoDB được cấu hình sharding, database config có thể dùng để lưu thông tin liên quan đến shard.
- **test**: database test được tạo mặc định. Khi kết nối tới service [mongod](https://mongoing.com/docs/reference/program/mongod.html) mà không chỉ định database cụ thể, mặc định sẽ kết nối tới database test.

Tên database có thể là bất kỳ string UTF-8 nào thỏa mãn các điều kiện sau:

- Không được là empty string `""`.
- Không được chứa `' '` (space), `.`, `$`, `/`, `\` và `\0` (null character).
- Nên viết toàn bộ bằng chữ thường.
- Tối đa 64 byte.

Tên database cuối cùng sẽ trở thành tên file trong file system, đó là lý do có nhiều giới hạn như vậy.

### MongoDB có những đặc điểm gì?

- **Bản ghi dữ liệu được lưu dưới dạng document**: bản ghi trong MongoDB chính là một BSON document, một cấu trúc dữ liệu gồm các cặp key-value, tương tự JSON object và là đơn vị dữ liệu cơ bản trong MongoDB.
- **Tự do về schema**: khái niệm collection tương tự table trong MySQL, nhưng không cần định nghĩa schema nào, có thể biểu diễn domain model phức tạp bằng ít data object hơn.
- **Hỗ trợ nhiều phương thức query**: MongoDB query API hỗ trợ thao tác đọc/ghi (CRUD), data aggregation, text search và geospatial query.
- **Hỗ trợ ACID transaction**: NoSQL database thường không hỗ trợ transaction vì phải đánh đổi để có khả năng mở rộng và performance cao. Tuy nhiên, MongoDB là một ngoại lệ. Giống relational database, MongoDB transaction cũng có các đặc tính ACID. MongoDB hỗ trợ atomicity nguyên bản ở cấp single document và cũng có đặc tính của transaction. MongoDB 4.0 bổ sung hỗ trợ multi-document transaction, nhưng chỉ hỗ trợ transaction trong replication set, tức scope của transaction bị giới hạn trong một replica set. MongoDB 4.2 giới thiệu distributed transaction, bổ sung hỗ trợ multi-document transaction trên sharded cluster và hợp nhất với hỗ trợ multi-document transaction sẵn có trên replica set.
- **Binary storage hiệu quả**: document được lưu trong collection dưới dạng các cặp key-value. Key dùng để định danh duy nhất một document, thường có type ObjectId, còn value tồn tại dưới dạng BSON. BSON = Binary JSON, là format được bổ sung một số type và metadata trên nền JSON.
- **Tích hợp data compression**: cần ít resource hơn để lưu cùng một lượng dữ liệu.
- **Hỗ trợ mapreduce**: hoàn thành các task aggregation phức tạp bằng phương pháp divide-and-conquer. Tuy nhiên, từ MongoDB 5.0, map-reduce không còn được MongoDB khuyến nghị sử dụng; phương án thay thế là [aggregation pipeline](https://www.mongodb.com/docs/manual/core/aggregation-pipeline/). Aggregation pipeline cung cấp performance và availability tốt hơn map-reduce.
- **Hỗ trợ nhiều loại index**: MongoDB hỗ trợ nhiều loại index, bao gồm single-field index, compound index, multikey index, hashed index, text index, geospatial index..., mỗi loại index phù hợp với các trường hợp sử dụng khác nhau.
- **Hỗ trợ failover**: cung cấp khả năng tự động khôi phục sau failure. Khi primary node gặp failure, hệ thống tự động bầu một primary node mới từ secondary node, bảo đảm cluster hoạt động bình thường mà client không nhận biết được.
- **Hỗ trợ sharded cluster**: MongoDB hỗ trợ cluster tự động phân tách dữ liệu, giúp cluster lưu trữ nhiều dữ liệu hơn và có performance cao hơn. Khi insert và update dữ liệu, hệ thống có thể tự động route và lưu trữ.
- **Hỗ trợ lưu file lớn**: dung lượng lưu trữ của một document trong MongoDB không được vượt quá 16MB. Với file lớn hơn 16MB, MongoDB cung cấp GridFS để lưu trữ. GridFS chia dữ liệu lớn thành các phần, sau đó lưu những document nhỏ đã được tách vào database.

### MongoDB phù hợp với những application scenario nào?

**Ưu thế của MongoDB nằm ở tính linh hoạt của data model và storage engine, khả năng mở rộng của architecture và hỗ trợ mạnh mẽ cho index.**

Khi chọn MongoDB, cần cân nhắc đầy đủ ưu thế của MongoDB và quyết định dựa trên nhu cầu của project thực tế:

- Khi project phát triển, việc lưu dữ liệu bằng format tương tự JSON (BSON) có đáp ứng nhu cầu của project không? Bản ghi trong MongoDB chính là một BSON document, một cấu trúc dữ liệu gồm các cặp key-value, tương tự JSON object và là đơn vị dữ liệu cơ bản trong MongoDB.
- Có cần lưu trữ lượng dữ liệu lớn không? Có cần mở rộng theo chiều ngang nhanh không? MongoDB hỗ trợ sharded cluster, có thể dễ dàng thêm nhiều node (instance), giúp cluster lưu trữ nhiều dữ liệu hơn và có performance cao hơn.
- Có cần nhiều loại index hơn để đáp ứng nhiều application scenario hơn không? MongoDB hỗ trợ nhiều loại index, bao gồm single-field index, compound index, multikey index, hashed index, text index, geospatial index..., mỗi loại index phù hợp với các trường hợp sử dụng khác nhau.
- ……

## MongoDB storage engine

### MongoDB hỗ trợ những storage engine nào?

Storage engine là component cốt lõi của database, chịu trách nhiệm quản lý cách dữ liệu được lưu trữ trong memory và disk.

Giống MySQL, MongoDB cũng sử dụng **plugin-based storage engine architecture**, hỗ trợ nhiều loại storage engine khác nhau. Mỗi storage engine giải quyết các vấn đề trong những scenario khác nhau. Khi tạo database hoặc collection, có thể chỉ định storage engine.

> Plugin-based storage engine architecture có thể tách rời Server layer và storage engine layer, đồng thời hỗ trợ nhiều storage engine. Chẳng hạn, MySQL vừa hỗ trợ storage engine InnoDB có cấu trúc B-Tree, vừa hỗ trợ storage engine RocksDB có cấu trúc LSM.

Khi storage engine mới xuất hiện, storage engine mặc định là MMAPV1. Từ MongoDB 4.x, storage engine MMAPv1 không còn được hỗ trợ.

Hiện nay chủ yếu có hai storage engine sau:

- **WiredTiger storage engine**: từ MongoDB 3.2, storage engine mặc định là [WiredTiger storage engine](https://www.mongodb.com/docs/manual/core/wiredtiger/). Nó rất phù hợp với phần lớn workload và được khuyến nghị cho deployment mới. WiredTiger cung cấp document-level concurrency model, checkpoint và data compression (sẽ giới thiệu ở phần sau) cùng các tính năng khác.
- **In-Memory storage engine**: [In-Memory storage engine](https://www.mongodb.com/docs/manual/core/inmemory/) có trong MongoDB Enterprise. Storage engine này không lưu document trên disk mà giữ document trong memory để có latency dữ liệu dễ dự đoán hơn.

Ngoài ra, MongoDB 3.0 cung cấp **pluggable storage engine API**, cho phép bên thứ ba phát triển storage engine cho MongoDB, khá tương tự MySQL.

### WiredTiger dựa trên LSM Tree hay B+ Tree?

Hiện nay, phần lớn database storage engine phổ biến được triển khai dựa trên B/B+ Tree hoặc LSM (Log Structured Merge) Tree. Đối với NoSQL database, phần lớn database (chẳng hạn HBase, Cassandra, RocksDB) dựa trên LSM Tree, nhưng MongoDB không giống như vậy.

Như đã nói ở trên, từ MongoDB 3.2, storage engine mặc định là WiredTiger storage engine. Trên website chính thức của WiredTiger, có thể thấy WiredTiger sử dụng B+ Tree làm storage structure:

```plain
WiredTiger maintains a table's data in memory using a data structure called a B-Tree ( B+ Tree to be specific), referring to the nodes of a B-Tree as pages. Internal pages carry only keys. The leaf pages store both keys and values.
```

Ngoài ra, WiredTiger cũng hỗ trợ [LSM (Log Structured Merge)](https://source.wiredtiger.com/3.1.0/lsm.html) Tree làm storage structure. Khi MongoDB sử dụng WiredTiger làm storage engine, mặc định nó dùng B+ Tree.

Nếu muốn tìm hiểu lý do MongoDB sử dụng B+ Tree, có thể xem bài viết này: [【Phản bác chuỗi bài phân tích máy móc】Đừng phân tích thiếu căn cứ nữa, MongoDB sử dụng B+ Tree, không phải B Tree như bạn nghĩ](https://zhuanlan.zhihu.com/p/519658576).

Khi sử dụng B+ Tree, WiredTiger dùng **page** làm đơn vị cơ bản để đọc ghi dữ liệu trên disk. Mỗi node của B+ Tree là một page. Có ba loại page:

- **root page (root node)**: root node của B+ Tree.
- **internal page (internal node)**: intermediate index node không trực tiếp lưu dữ liệu.
- **leaf page (leaf node)**: leaf node trực tiếp lưu dữ liệu, gồm page header, block header và dữ liệu thực tế (key/value). Page header định nghĩa type của page, kích thước payload thực tế và số record trong page; block header định nghĩa checksum của page, vị trí address của block trên disk và các thông tin khác.

Cấu trúc tổng thể như hình dưới đây:

![Cấu trúc tổng thể WiredTiger B+ Tree](https://oss.javaguide.cn/github/javaguide/database/mongodb/mongodb-b-plus-tree-integral-structure.png)

Nếu muốn nghiên cứu sâu về WiredTiger storage engine, nên đọc [loạt bài về WiredTiger storage engine](https://mongoing.com/archives/category/wiredtiger%e5%ad%98%e5%82%a8%e5%bc%95%e6%93%8e%e7%b3%bb%e5%88%97) của cộng đồng MongoDB Trung Quốc.

## MongoDB aggregation

### MongoDB aggregation có tác dụng gì?

Trong project thực tế, chúng ta thường cần gộp nhiều document, thậm chí nhiều collection, để tính toán và phân tích (chẳng hạn tính tổng, lấy giá trị lớn nhất), sau đó trả về kết quả đã tính. Quá trình này được gọi là **aggregation operation**.

Theo tài liệu chính thức, có thể sử dụng aggregation operation để:

- Kết hợp các value từ nhiều document.
- Thực hiện một chuỗi phép tính trên dữ liệu trong collection.
- Phân tích sự thay đổi của dữ liệu theo thời gian.

### MongoDB cung cấp những phương thức thực thi aggregation nào?

MongoDB cung cấp hai phương thức thực thi aggregation:

- **Aggregation pipeline**: phương thức được ưu tiên để thực hiện aggregation operation.
- **Single purpose aggregation methods**: các hàm aggregation cho một mục đích duy nhất, chẳng hạn `count()`, `distinct()`, `estimatedDocumentCount()`.

Phần lớn bài viết còn đề cập đến phương thức aggregation **map-reduce**. Tuy nhiên, từ MongoDB 5.0, map-reduce không còn được MongoDB khuyến nghị sử dụng; phương án thay thế là [aggregation pipeline](https://www.mongodb.com/docs/manual/core/aggregation-pipeline/). Aggregation pipeline cung cấp performance và availability tốt hơn map-reduce.

MongoDB aggregation pipeline gồm nhiều stage. Mỗi stage chuyển đổi document khi document đi qua pipeline. Mỗi stage nhận output của stage trước đó, tiếp tục xử lý dữ liệu rồi gửi dữ liệu đó làm input cho stage tiếp theo.

Quy trình hoạt động của mỗi pipeline:

1. Nhận một chuỗi document dữ liệu raw
2. Thực hiện một chuỗi phép tính trên các document này
3. Output document kết quả cho stage tiếp theo

![Quy trình hoạt động của pipeline](https://oss.javaguide.cn/github/javaguide/database/mongodb/mongodb-aggregation-stage.png)

**Các stage operator thường dùng**:

| Operator  | Mô tả ngắn                                                                                                                              |
| --------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| \$match   | Operator match, dùng để filter collection document                                                                                      |
| \$project | Operator projection, dùng để tái cấu trúc field của từng document, có thể extract, rename hoặc thao tác trên field cũ để thêm field mới |
| \$sort    | Operator sort, dùng để sắp xếp document theo một hoặc nhiều field                                                                       |
| \$limit   | Operator limit, dùng để giới hạn số lượng document trả về                                                                               |
| \$skip    | Operator skip, dùng để bỏ qua số lượng document được chỉ định                                                                           |
| \$count   | Operator count, dùng để thống kê số lượng document                                                                                      |
| \$group   | Operator group, dùng để group collection document                                                                                       |
| \$unwind  | Operator unwind, dùng để tách từng value trong array thành một document riêng                                                           |
| \$lookup  | Operator lookup, dùng để join collection khác trong cùng database và lấy document được chỉ định, tương tự populate                      |

Xem thêm giới thiệu về các operator trong tài liệu chính thức: <https://docs.mongodb.com/manual/reference/operator/aggregation/>

Stage operator được dùng làm phần tử cấp đầu tiên trong array parameter của method `db.collection.aggregate`.

```sql
db.collection.aggregate( [ { stage operator: mô tả }, { stage operator: mô tả }, ... ] )
```

Dưới đây là một ví dụ trong tài liệu chính thức của MongoDB:

```sql
db.orders.aggregate([
   # Stage thứ nhất: stage $match filter document theo field status và truyền document có status bằng "A" đến stage tiếp theo.
    { $match: { status: "A" } },
  # Stage thứ hai: stage $group group document theo field cust_id để tính tổng amount của mỗi giá trị cust_id duy nhất.
    { $group: { _id: "$cust_id", total: { $sum: "$amount" } } }
])
```

## MongoDB transaction

> Việc hiểu nguyên lý của MongoDB transaction khá tốn thời gian, bản thân tôi cũng chưa hiểu thật rõ. Vì vậy, ở đây tôi chỉ giới thiệu đơn giản về MongoDB transaction. Nếu muốn tìm hiểu nguyên lý, bạn có thể tự tìm kiếm và tham khảo tài liệu liên quan.
>
> Dưới đây là một số bài viết được đề xuất để tham khảo:
>
> - [Kiến thức kỹ thuật | Nguyên lý MongoDB transaction](https://mongoing.com/archives/82187)
> - [Thiết kế và triển khai consistency model của MongoDB](https://developer.aliyun.com/article/782494)
> - [Giới thiệu transaction trong tài liệu chính thức của MongoDB](https://www.mongodb.com/docs/upcoming/core/transactions/)

Khi giới thiệu về NoSQL data, chúng ta cũng đã nói rằng NoSQL database thường không hỗ trợ transaction vì phải đánh đổi để có khả năng mở rộng và performance cao. Tuy nhiên, MongoDB là một ngoại lệ và có hỗ trợ transaction.

Giống relational database, MongoDB transaction cũng có các đặc tính ACID:

- **Atomicity** (`Atomicity`): transaction là execution unit nhỏ nhất và không được phép chia tách. Atomicity của transaction bảo đảm action hoặc hoàn thành toàn bộ, hoặc hoàn toàn không có tác dụng;
- **Consistency** (`Consistency`): dữ liệu nhất quán trước và sau khi thực thi transaction. Chẳng hạn trong nghiệp vụ chuyển khoản, bất kể transaction thành công hay thất bại, tổng tiền của người chuyển và người nhận phải không đổi;
- **Isolation** (`Isolation`): khi truy cập database đồng thời, transaction của một user không bị transaction khác can thiệp, database giữa các concurrent transaction là độc lập. WiredTiger storage engine hỗ trợ isolation read-uncommitted, read-committed và snapshot. Khi MongoDB khởi động, mặc định chọn snapshot isolation. Ở các isolation level khác nhau, trong vòng đời của một transaction có thể xuất hiện dirty read, non-repeatable read, phantom read và các hiện tượng khác.
- **Durability** (`Durability`): sau khi transaction được commit, thay đổi của transaction đối với dữ liệu trong database phải được duy trì; ngay cả khi database gặp failure, thay đổi đó cũng không được bị ảnh hưởng.

Bài viết này không trình bày chi tiết về transaction. Nếu quan tâm, bạn có thể xem bài [Tổng hợp câu hỏi phỏng vấn MySQL thường gặp](../mysql/mysql-questions-01.md), trong đó có giới thiệu chi tiết.

MongoDB hỗ trợ atomicity nguyên bản ở cấp single document và cũng có đặc tính của transaction. Khi nói về MongoDB transaction, thông thường là nói đến **multi-document**. MongoDB 4.0 bổ sung hỗ trợ ACID transaction trên nhiều document, nhưng chỉ hỗ trợ ACID transaction trong replication set, tức scope của transaction bị giới hạn trong một replica set. MongoDB 4.2 giới thiệu **distributed transaction**, bổ sung hỗ trợ multi-document transaction trên sharded cluster và hợp nhất với hỗ trợ multi-document transaction sẵn có trên replica set.

Theo tài liệu chính thức:

> Từ MongoDB 4.2, distributed transaction và multi-document transaction trong MongoDB là cùng một khái niệm. Distributed transaction là multi-document transaction trên sharded cluster và replica set. Từ MongoDB 4.2, multi-document transaction (dù trên sharded cluster hay replica set) cũng được gọi là distributed transaction.

Trong phần lớn trường hợp, multi-document transaction có cost performance lớn hơn single-document write. Với phần lớn scenario, [denormalized data model (embedded document và array)](https://www.mongodb.com/docs/upcoming/core/data-model-design/#std-label-data-modeling-embedding) vẫn là lựa chọn tốt nhất. Nói cách khác, data modeling phù hợp có thể giảm nhu cầu sử dụng multi-document transaction xuống mức tối đa.

**Lưu ý**:

- Từ MongoDB 4.2, multi-document transaction hỗ trợ replica set và sharded cluster, trong đó primary node sử dụng WiredTiger storage engine, còn secondary node sử dụng WiredTiger storage engine hoặc In-Memory storage engine. Trong MongoDB 4.0, chỉ replica set sử dụng WiredTiger storage engine mới hỗ trợ transaction.
- Trong MongoDB 4.2 và các version cũ hơn, không thể tạo collection trong transaction. Từ MongoDB 4.4, có thể tạo collection và index trong transaction. Xem chi tiết tại [Tạo collection và index trong transaction](https://www.mongodb.com/docs/upcoming/core/transactions/#std-label-transactions-create-collections-indexes).

## MongoDB data compression

Nhờ WiredTiger storage engine (storage engine mặc định từ MongoDB 3.2), MongoDB hỗ trợ compression cho tất cả collection và index. Compression giảm thiểu storage usage với cost là thêm CPU.

Theo mặc định, WiredTiger sử dụng thuật toán compression [Snappy](https://github.com/google/snappy) (do Google mã nguồn mở, hướng đến tốc độ rất cao và compression hợp lý, compression ratio từ 3 đến 5 lần) để block compression cho tất cả collection và prefix compression cho tất cả index.

Ngoài Snappy, collection còn có các thuật toán compression sau:

- [zlib](https://github.com/madler/zlib): thuật toán compression cao, compression ratio từ 5 đến 7 lần
- [Zstandard](https://github.com/facebook/zstd) (viết tắt là zstd): thuật toán compression nhanh, lossless, mã nguồn mở bởi Facebook. Thuật toán này hướng đến scenario realtime compression ở cấp độ zlib và compression ratio tốt hơn, cung cấp compression rate cao hơn cùng CPU usage thấp hơn, có từ MongoDB 4.2.

WiredTiger log cũng được compression, mặc định cũng sử dụng thuật toán compression Snappy. Nếu log record nhỏ hơn hoặc bằng 128 byte, WiredTiger sẽ không compression record đó.

## Khác biệt giữa Amazon Document và MongoDB

Amazon DocumentDB (tương thích với MongoDB) là một database service nhanh, đáng tin cậy và được quản lý hoàn toàn. Amazon DocumentDB giúp dễ dàng thiết lập, vận hành và mở rộng database tương thích MongoDB trên cloud.

### Operator `$vectorSearch`

Amazon DocumentDB không hỗ trợ `$vectorSearch` như một operator độc lập. Thay vào đó, nó hỗ trợ `vectorSearch` bên trong operator `$search`. Xem thêm [Vector search trên Amazon DocumentDB](https://docs.aws.amazon.com/zh_cn/documentdb/latest/developerguide/vector-search.html).

### `OpCountersCommand`

Behavior của `OpCountersCommand` trong Amazon DocumentDB khác với `opcounters.command` của MongoDB như sau:

- `opcounters.command` của MongoDB tính tất cả command ngoại trừ insert, update và delete, còn `OpCountersCommand` của Amazon DocumentDB cũng loại trừ command `find`.
- Amazon DocumentDB tính internal command (chẳng hạn `getCloudWatchMetricsV2`) vào `OpCountersCommand`.

### Quản lý database và collection

Amazon DocumentDB không hỗ trợ admin database hoặc local database, cũng không hỗ trợ collection `system.*` hoặc `startup_log` của MongoDB.

### `cursormaxTimeMS`

Trong Amazon DocumentDB, `cursor.maxTimeMS` reset counter cho mỗi request. Do đó, nếu chỉ định `maxTimeMS` là 3000MS, query mất 2800MS còn mỗi request `getMore` tiếp theo mất 300MS thì cursor sẽ không timeout. Cursor chỉ timeout khi một operation duy nhất (dù là query hay một request `getMore`) mất nhiều hơn giá trị `maxTimeMS` đã chỉ định. Ngoài ra, scanner kiểm tra execution time của cursor chạy theo chu kỳ 5 phút.

### explain()

Amazon DocumentDB mô phỏng MongoDB 4.0 API trên một database engine chuyên dụng, sử dụng distributed, fault-tolerant và self-healing storage system. Vì vậy, query plan và output của `explain()` có thể khác nhau giữa Amazon DocumentDB và MongoDB. Client muốn kiểm soát query plan có thể dùng operator `$hint` để buộc chọn index ưu tiên.

### Giới hạn tên field

Amazon DocumentDB không hỗ trợ dấu chấm `.` trong tên field của document, chẳng hạn `db.foo.insert({'x.1':1})`.

Amazon DocumentDB cũng không hỗ trợ prefix `$` trong tên field.

Ví dụ, thử command sau trong Amazon DocumentDB hoặc MongoDB:

```shell
rs0:PRIMARY< db.foo.insert({"a":{"$a":1}})
```

MongoDB sẽ trả về kết quả sau:

```shell
WriteResult({ "nInserted" : 1 })
```

Amazon DocumentDB sẽ trả về một error:

```shell
WriteResult({
  "nInserted" : 0,
  "writeError" : {
    "code" : 2,
    "errmsg" : "Document can't have $ prefix field names: $a"
  }
})
```

## Tham khảo

- Tài liệu chính thức MongoDB (tài liệu tham khảo chính, ưu tiên tài liệu chính thức): <https://www.mongodb.com/docs/manual/>
- 《MongoDB Definitive Guide》
- Kiến thức kỹ thuật | Nguyên lý MongoDB transaction - Cộng đồng MongoDB Trung Quốc: <https://mongoing.com/archives/82187>
- Transactions - tài liệu chính thức MongoDB: <https://www.mongodb.com/docs/manual/core/transactions/>
- WiredTiger Storage Engine - tài liệu chính thức MongoDB: <https://www.mongodb.com/docs/manual/core/wiredtiger/>
- Một bài về WiredTiger storage engine: Phân tích cấu trúc dữ liệu cơ bản: <https://mongoing.com/topic/archives-35143>

<!-- @include: @article-footer.snippet.md -->
