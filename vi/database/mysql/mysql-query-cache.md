---
title: "Giải thích chi tiết Query Cache của MySQL"
description: "Phân tích chuyên sâu nguyên lý hoạt động, quản lý cấu hình và ưu nhược điểm của Query Cache MySQL, giải thích vì sao MySQL 8.0 loại bỏ tính năng Query Cache và các đề xuất thực tiễn tốt nhất trong môi trường production."
category: Database
tag:
  - MySQL
head:
  - - meta
    - name: keywords
      content: "MySQL Query Cache,Query Cache,cơ chế cache MySQL,cache invalidation,MySQL 8.0,tối ưu hiệu năng truy vấn,quản lý bộ nhớ MySQL"
---

Cache là một phương thức tối ưu hiệu năng hệ thống hiệu quả và thiết thực. Dù là hệ điều hành, các loại phần mềm ứng dụng hay Web service, chúng đều sử dụng rộng rãi cơ chế cache.

Tuy nhiên, các DBA có kinh nghiệm đều khuyên nên tắt Query Cache (cache truy vấn) tích hợp sẵn của MySQL trong môi trường production. Hơn nữa, từ MySQL 5.7.20, Query Cache đã bị deprecated mặc định. Đến MySQL 8.0 trở đi, tính năng Query Cache thậm chí đã bị xóa hoàn toàn.

Vậy tại sao lại như vậy? Query Cache thực sự vô dụng đến thế sao?

Hãy bắt đầu bài viết với một số câu hỏi sau.

- Query Cache của MySQL là gì? Phạm vi áp dụng?
- Quy tắc cache của MySQL là gì?
- Ưu nhược điểm của cache MySQL là gì?
- Cache MySQL ảnh hưởng thế nào đến hiệu năng?

## Giới thiệu Query Cache của MySQL

Kiến trúc MySQL như hình dưới đây:

![](https://oss.javaguide.cn/github/javaguide/mysql/mysql-architecture.png)

Để tăng tốc độ phản hồi của các câu truy vấn hoàn toàn giống nhau, MySQL Server sẽ thực hiện phép tính Hash trên câu truy vấn để nhận được một giá trị Hash. MySQL Server không xử lý SQL theo bất kỳ cách nào, SQL phải hoàn toàn giống nhau thì giá trị Hash mới giống nhau. Sau khi nhận được giá trị Hash, MySQL dùng giá trị Hash đó để đối chiếu kết quả của câu truy vấn trong Query Cache.

- Nếu khớp (cache hit), result set của truy vấn sẽ được trả trực tiếp về client, không cần parse hay thực thi truy vấn nữa.
- Nếu không khớp (cache miss), giá trị Hash và result set sẽ được lưu vào Query Cache để sử dụng về sau.

> Nói cách khác, **sau khi một câu truy vấn (select) đến MySQL Server, nó sẽ được kiểm tra trong Query Cache trước. Nếu đã từng được thực thi, result set sẽ được trả trực tiếp về client.**

![](https://oss.javaguide.cn/javaguide/13526879-3037b144ed09eb88.png)

## Quản lý và cấu hình Query Cache của MySQL

Có thể dùng lệnh `show variables like '%query_cache%'` để xem thông tin liên quan đến Query Cache.

Trước phiên bản 8.0, thông tin được in ra có thể như sau:

```bash
mysql> show variables like '%query_cache%';
+------------------------------+---------+
| Variable_name                | Value   |
+------------------------------+---------+
| have_query_cache             | YES     |
| query_cache_limit            | 1048576 |
| query_cache_min_res_unit     | 4096    |
| query_cache_size             | 599040  |
| query_cache_type             | ON      |
| query_cache_wlock_invalidate | OFF     |
+------------------------------+---------+
6 rows in set (0.02 sec)
```

Từ phiên bản 8.0 trở đi, thông tin được in ra như sau:

```bash
mysql> show variables like '%query_cache%';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| have_query_cache | NO    |
+------------------+-------+
1 row in set (0.01 sec)
```

Sau đây, chúng ta giải thích thông tin được in ra bởi lệnh `show variables like '%query_cache%';` trước phiên bản 8.0.

- **`have_query_cache`:** MySQL Server đó có hỗ trợ Query Cache hay không. Nếu là YES thì có hỗ trợ, nếu không thì không hỗ trợ.
- **`query_cache_limit`:** Kích thước result set lớn nhất được Query Cache của MySQL cache. Khi result set lớn hơn giá trị này, nó sẽ không được cache.
- **`query_cache_min_res_unit`:** Kích thước block nhỏ nhất (byte) được Query Cache cấp phát. Khi truy vấn được thực hiện, MySQL lưu result set vào Query Cache. Tuy nhiên, nếu result set cần lưu khá lớn và vượt quá giá trị `query_cache_min_res_unit`, MySQL sẽ vừa lấy result vừa lưu dữ liệu. Nói cách khác, một truy vấn có thể cần thực hiện thao tác cấp phát memory nhiều lần. Điều chỉnh `query_cache_min_res_unit` phù hợp có thể tối ưu memory.
- **`query_cache_size`:** Dung lượng memory được cấp phát để cache result set, đơn vị là byte và giá trị phải là bội số nguyên của 1024. Tài liệu chính thức MySQL 5.7 cho biết giá trị mặc định là `1048576` (1 MB), đặt bằng 0 sẽ disable Query Cache. Giá trị mặc định giữa các bản minor version có thể khác nhau, nên chỉ định rõ trong file cấu hình thay vì phụ thuộc vào hành vi mặc định.
- **`query_cache_type`:** Thiết lập loại Query Cache, mặc định là ON. Đặt giá trị GLOBAL có thể thiết lập loại cho mọi client connection tiếp theo. Client có thể đặt giá trị SESSION để ảnh hưởng đến việc sử dụng Query Cache của chính mình.
- **`query_cache_wlock_invalidate`:** Khi một table bị lock, có trả về dữ liệu trong cache hay không. Mặc định đang tắt, trong môi trường production thường nên giữ cấu hình mặc định này.

Các giá trị khả dụng của `query_cache_type` (`query_cache_type` là dynamic variable trong MySQL 5.6/5.7, **nhưng có điều kiện tiên quyết**: nếu instance khởi động với `query_cache_type=0`, server sẽ bỏ qua việc cấp phát exclusive lock của Query Cache. Khi đó, sửa động bằng `SET GLOBAL` sẽ báo lỗi, bắt buộc phải sửa file cấu hình và restart; nếu lúc khởi động giá trị khác 0 thì có thể dùng `SET GLOBAL query_cache_type=N` để có hiệu lực online mà không cần restart):

- 0 hoặc OFF: tắt tính năng Query Cache.
- 1 hoặc ON: bật tính năng Query Cache, nhưng không cache các truy vấn bắt đầu bằng `Select SQL_NO_CACHE`.
- 2 hoặc DEMAND: bật tính năng Query Cache, nhưng chỉ cache các truy vấn bắt đầu bằng `Select SQL_CACHE`.

**Khuyến nghị**:

- Không nên đặt `query_cache_size` quá lớn. Dung lượng quá lớn không chỉ chiếm chỗ của các cấu trúc memory khác trong instance mà còn làm tăng chi phí tìm kiếm trong cache. Nên căn cứ vào cấu hình instance, đặt giá trị ban đầu trong khoảng 10MB đến 100MB, sau đó điều chỉnh theo tình hình sử dụng thực tế.
- Nên disable Query Cache bằng cách đặt `query_cache_size` bằng 0, thay vì chỉ phụ thuộc vào `query_cache_type`. Cả hai đều là dynamic variable, nhưng `query_cache_size=0` sẽ bỏ qua hoàn toàn việc cấp phát memory cho cache và đường dẫn kiểm tra, nên disable triệt để hơn.

Trước phiên bản 8.0, thêm cấu hình sau vào `my.cnf` rồi restart MySQL để bật Query Cache:

```properties
query_cache_type=1
query_cache_size=614400
```

Hoặc khi instance khởi động với `query_cache_type` khác 0, có thể bật Query Cache online bằng các lệnh sau (nếu giá trị lúc khởi động là 0 thì lệnh sẽ báo lỗi, cần sửa file cấu hình rồi restart):

```sql
set global query_cache_type=1;
set global query_cache_size=614400;
```

Có thể dùng ba câu SQL sau để dọn cache thủ công:

- `flush query cache;`: dọn các memory fragment của Query Cache.
- `reset query cache;`: xóa toàn bộ query khỏi Query Cache.
- `flush tables;` đóng tất cả table đang mở, đồng thời thao tác này sẽ xóa nội dung trong Query Cache.

## Cơ chế cache của MySQL

### Quy tắc cache

- Query Cache lưu câu truy vấn và result set vào memory (thường dưới dạng key-value; trong đó Key là giá trị Hash được tính từ text của câu truy vấn, Database hiện tại, charset của client, protocol version và các tham số môi trường khác, còn Value là result set của truy vấn), lần sau sẽ lấy trực tiếp từ memory.
- Kết quả cache được chia sẻ giữa các session, nên một client có thể sử dụng kết quả cache của client khác.
- SQL phải hoàn toàn giống nhau thì mới cache hit (chữ hoa chữ thường, khoảng trắng, Database được sử dụng, protocol version, charset và các yếu tố khác đều phải giống nhau). Khi kiểm tra Query Cache, MySQL Server không xử lý SQL theo bất kỳ cách nào mà sử dụng chính xác câu truy vấn do client truyền đến.
- Không cache result set của subquery trong truy vấn, chỉ cache result set cuối cùng của truy vấn.
- Các function không xác định sẽ không bao giờ được cache, chẳng hạn `now()`, `curdate()`, `last_insert_id()`, `rand()` và các function khác.
- Không cache các truy vấn tạo ra warning (Warnings).
- Khi result set vượt quá `query_cache_limit` (mặc định 1 MB), nó sẽ không được cache.
- Nếu truy vấn chứa bất kỳ user-defined function, stored function, user variable, temporary table hoặc system table trong MySQL nào, kết quả truy vấn cũng sẽ không được cache.
- Sau khi cache được tạo, hệ thống Query Cache của MySQL sẽ theo dõi từng table liên quan trong truy vấn. Nếu các table này (dữ liệu hoặc cấu trúc) thay đổi, toàn bộ dữ liệu cache liên quan đến table đó sẽ mất hiệu lực.
- Query Cache của MySQL gần như không có tác dụng trong môi trường sharding. Nguyên nhân là truy vấn thường được middleware (như ShardingSphere, MyCat) route đến các MySQL instance khác nhau, mỗi instance duy trì Query Cache độc lập. Khi route, middleware thường rewrite SQL (thêm điều kiện shard key và các điều kiện khác), khiến giá trị Hash của câu truy vấn sau khi rewrite không giống câu truy vấn ban đầu, nên cache không thể hit.
- Không cache các truy vấn sử dụng `SQL_NO_CACHE`.
- ……

Ví dụ về option `SELECT` của Query Cache:

```sql
SELECT SQL_CACHE id, name FROM customer;# sẽ được cache
SELECT SQL_NO_CACHE id, name FROM customer;# sẽ không được cache
```

### Quản lý memory trong cơ chế cache

Query Cache được lưu hoàn toàn trong memory, nên trước khi cấu hình và sử dụng, chúng ta cần hiểu cách nó sử dụng memory.

Query Cache của MySQL sử dụng kỹ thuật memory pool để tự quản lý việc giải phóng và cấp phát memory, thay vì thông qua hệ điều hành. Đơn vị cơ bản mà memory pool sử dụng là block có độ dài thay đổi, dùng để lưu thông tin như type, size, data. Các block của một result set được nối với nhau bằng linked list. Độ dài ngắn nhất của block là `query_cache_min_res_unit`.

Khi server khởi động, nó khởi tạo memory cần thiết cho cache thành một free block hoàn chỉnh. Khi truy vấn bắt đầu trả về kết quả, do chưa thể biết trước result set hoàn chỉnh lớn bao nhiêu, MySQL trước tiên xin memory pool một data block cơ sở có kích thước `query_cache_min_res_unit`. Nếu result set vượt quá capacity của block đó, trong quá trình tạo kết quả, hệ thống sẽ tiếp tục xin các data block mới theo nhu cầu và nối chúng bằng linked list.

Để cấp phát memory block, trước tiên cần lock memory block, nên thao tác này rất chậm. MySQL sẽ cố gắng tránh thao tác này bằng cách chọn memory block nhỏ nhất có thể; nếu không đủ thì tiếp tục xin thêm, nếu còn dư sau khi lưu xong thì giải phóng phần dư.

Khi việc đọc ghi concurrent diễn ra, các cache block có kích thước khác nhau bị giải phóng không theo thứ tự và ngẫu nhiên. Kết hợp với phần memory nhỏ còn thừa khi cấp phát (nhỏ hơn `query_cache_min_res_unit`) không thể được tái sử dụng, memory pool sẽ nhanh chóng tạo ra một lượng lớn free block không liên tục (tương tự external fragment ở tầng hệ điều hành), từ đó kích hoạt thao tác memory compaction thường xuyên hơn và làm phát sinh chi phí.

## Ưu nhược điểm của Query Cache MySQL

**Ưu điểm:**

- Query Cache được xử lý sau khi MySQL nhận yêu cầu truy vấn từ client và xác thực quyền truy vấn, nhưng trước khi parse SQL truy vấn. Nói cách khác, sau khi MySQL nhận SQL truy vấn từ client, nó chỉ cần xác thực quyền tương ứng rồi tìm kết quả qua Query Cache; thậm chí không cần đi qua module Optimizer để phân tích và tối ưu execution plan, cũng không cần tương tác với bất kỳ storage engine nào.
- Vì Query Cache dựa trên memory và trả kết quả truy vấn tương ứng trực tiếp từ memory, nó giảm đáng kể disk I/O và CPU calculation. **Tuy nhiên, ưu điểm này chỉ đúng trong các scenario tĩnh có concurrency thấp và read nhiều write ít**; trong môi trường multi-core high concurrency, cạnh tranh gay gắt trên global mutex `LOCK_query_cache` sẽ khiến nhiều thread phải chờ lock (có thể thấy `Waiting for query cache lock` qua `SHOW PROCESSLIST`), khiến TPS/QPS thực tế ngược lại giảm mạnh.

**Nhược điểm:**

- MySQL sẽ tính Hash cho mọi truy vấn loại SELECT nhận được, sau đó tìm xem kết quả cache của truy vấn này có tồn tại hay không. Dù chi phí CPU của việc tính Hash và tìm kiếm bản thân không đáng kể, Query Cache ở tầng dưới phụ thuộc vào một global mutex duy nhất (`LOCK_query_cache`) để bảo đảm an toàn khi concurrent. Khi có high concurrency, hàng nghìn hàng vạn câu truy vấn đồng thời tranh mutex để kiểm tra hoặc ghi cache, khiến lock conflict và chi phí thread context switch cực kỳ nghiêm trọng trở thành performance bottleneck chí mạng.
- Vấn đề cache invalidation của Query Cache. Nếu table thay đổi thường xuyên, tỷ lệ cache invalidation sẽ rất cao. Việc table thay đổi không chỉ là dữ liệu trong table thay đổi, mà còn bao gồm mọi thay đổi về cấu trúc table hoặc index.
- Các truy vấn khác nhau nhưng có cùng kết quả đều được cache, dẫn đến tiêu thụ quá mức tài nguyên memory. Query Cache coi việc khác nhau về chữ hoa chữ thường, khoảng trắng hoặc comment trong câu truy vấn là các truy vấn khác nhau (vì giá trị Hash của chúng khác nhau).
- Việc thiết lập các system variable liên quan không hợp lý sẽ tạo ra nhiều memory fragment, từ đó khiến Query Cache thường xuyên phải dọn memory.

## Ảnh hưởng của Query Cache MySQL đến hiệu năng

Bật Query Cache trong MySQL Server sẽ tạo thêm chi phí cho cả thao tác đọc và ghi của database:

- **Thao tác đọc cần giữ lock để kiểm tra**: trước khi bắt đầu truy vấn đọc, phải kiểm tra cache hit, việc này cần lấy shared lock `LOCK_query_cache`. Khi high concurrency, nhiều read request đồng thời tranh lock sẽ phải xếp hàng.
- **Chi phí ghi cache**: nếu read query có thể được cache, sau khi thực thi cần ghi kết quả vào cache, bao gồm thao tác cấp phát memory và nối linked list, đồng thời cũng cần giữ lock.
- **Thao tác ghi kích hoạt invalidation toàn cục**: khi ghi dữ liệu vào table, phải làm mất hiệu lực toàn bộ cache của table đó. Việc này cần lấy exclusive lock để quét toàn bộ cache area; `query_cache_size` càng lớn thì thời gian giữ lock càng dài. Thiết kế global mutex duy nhất của Query Cache khiến thao tác ghi block mọi read/write request khác. Đây cũng là nguyên nhân hàng đầu khiến MySQL 8.0 loại bỏ tính năng này.
- **Long transaction của InnoDB làm vấn đề nghiêm trọng hơn**: với đặc tính MVCC, cache liên quan không thể sử dụng trước khi transaction commit. Long transaction không chỉ làm giảm cache hit rate, mà exclusive lock do thao tác ghi kích hoạt còn block việc đọc cache của **các table không liên quan khác**.

Có thể dùng lệnh sau để xem tình hình sử dụng Query Cache và đánh giá có đáng bật hay không:

```sql
SHOW STATUS LIKE 'Qcache%';
```

Giải thích các metric chính:

| Status variable        | Ý nghĩa                                                                                                           |
| :--------------------- | :---------------------------------------------------------------------------------------------------------------- |
| `Qcache_hits`          | Số lần cache hit                                                                                                  |
| `Qcache_inserts`       | Số truy vấn được ghi vào cache                                                                                    |
| `Qcache_not_cached`    | Số truy vấn không được cache (không thể cache hoặc cache miss)                                                    |
| `Qcache_lowmem_prunes` | Số cache entry bị evict do thiếu memory; tăng liên tục cho thấy cache thiếu dung lượng hoặc fragment nghiêm trọng |
| `Qcache_free_memory`   | Dung lượng memory trống còn lại của cache (byte)                                                                  |

Công thức tham khảo cho cache hit rate:

```
cache hit rate = Qcache_hits / (Qcache_hits + Qcache_inserts + Qcache_not_cached)
```

Nếu cache hit rate thấp hơn 50% trong thời gian dài, điều đó cho thấy workload không phù hợp với Query Cache và nên disable. Ngoài ra, cần theo dõi tỷ lệ giữa `Qcache_lowmem_prunes` và `Qcache_inserts`: nếu tỷ lệ này cực cao, nghĩa là dữ liệu vừa được ghi vào cache nhanh chóng bị loại bỏ do memory fragment hoặc thiếu dung lượng. Khi đó, bật cache chỉ mang lại tác động tiêu cực. Khi `Qcache_lowmem_prunes` tiếp tục tăng, có thể thực thi `FLUSH QUERY CACHE` để compact memory fragment, hoặc giảm phù hợp giá trị `query_cache_min_res_unit`.

## Tổng kết

Query Cache trong MySQL tuy có thể nâng cao hiệu năng truy vấn của database, nhưng bản thân cơ chế Query Cache cũng tạo thêm chi phí quản lý. Sau mỗi truy vấn đều phải thực hiện một thao tác cache, và sau khi mất hiệu lực còn phải hủy cache.

Query Cache là một cơ chế cache chỉ phù hợp với một số ít trường hợp. Nếu ứng dụng của bạn ít cập nhật database, Query Cache sẽ phát huy tác dụng đáng kể. Một ví dụ điển hình là hệ thống blog: blog thường được cập nhật tương đối chậm, các table dữ liệu tương đối ổn định và ít thay đổi, nên Query Cache sẽ có tác dụng rõ rệt.

Tóm tắt các scenario phù hợp với Query Cache:

- Dữ liệu trong table ít thay đổi, tương đối tĩnh.
- Query (SELECT) có độ lặp lại cao.
- Result set của query nhỏ hơn 1 MB.

Với một hệ thống cập nhật thường xuyên, Query Cache có tác dụng rất nhỏ; trong một số trường hợp, bật Query Cache còn làm hiệu năng giảm.

Tóm tắt các scenario không phù hợp với Query Cache:

- Dữ liệu, cấu trúc table hoặc index thay đổi thường xuyên.
- Có ít query lặp lại.
- Result set của query lớn.

《High Performance MySQL》 viết như sau:

> Theo kinh nghiệm của chúng tôi, trong môi trường chịu tải high concurrency, Query Cache sẽ khiến hiệu năng hệ thống giảm, thậm chí bị treo. Nếu bạn nhất định phải sử dụng Query Cache, đừng đặt memory quá lớn và chỉ sử dụng khi đã xác định rõ lợi ích (database ít thay đổi nội dung).

**Đúng là như vậy! Trong project thực tế, thường nên sử dụng local cache (chẳng hạn Caffeine) hoặc distributed cache (chẳng hạn Redis), hiệu năng tốt hơn và cũng phổ dụng hơn.**

## Tài liệu tham khảo

- 《High Performance MySQL》
- Cơ chế cache MySQL: <https://zhuanlan.zhihu.com/p/55947158>
- Cấu hình và sử dụng Query Cache của RDS MySQL - Tài liệu Alibaba Cloud RDS:<https://help.aliyun.com/document_detail/41717.html>
- 8.10.3 The MySQL Query Cache - Tài liệu chính thức MySQL:<https://dev.mysql.com/doc/refman/5.7/en/query-cache.html>

<!-- @include: @article-footer.snippet.md -->
