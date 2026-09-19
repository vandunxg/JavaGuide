---
title: Phân tích execution plan của MySQL
description: Giải thích chi tiết ý nghĩa các cột trong execution plan EXPLAIN của MySQL, gồm id, select_type, type, key, rows, Extra và các trường quan trọng khác, giúp bạn phân tích điểm nghẽn performance của SQL và tối ưu đúng trọng tâm.
category: Database
tag:
  - MySQL
head:
  - - meta
    - name: keywords
      content: execution plan MySQL,EXPLAIN,query optimizer,phân tích performance SQL,index usage,type access,Extra,tối ưu slow query
---

Bước đầu tiên để tối ưu SQL là đọc hiểu execution plan SQL. Bài viết này trình bày các kiến thức liên quan đến execution plan `EXPLAIN` của MySQL.

> **Ghi chú phiên bản**: Nội dung bài viết dựa trên MySQL 5.7+ và 8.0+. Các cột `filtered` và `partitions` khả dụng trên MySQL 5.7+, còn tính năng `EXPLAIN ANALYZE` và Hash Join yêu cầu MySQL 8.0.18+ và 8.0.20+.

## Execution plan là gì?

**Execution plan** là cách thực thi cụ thể của một câu lệnh SQL sau khi được **MySQL query optimizer** tối ưu.

Execution plan thường được dùng để phân tích và tối ưu performance SQL. Thông qua kết quả của `EXPLAIN`, bạn có thể biết thứ tự query các table, loại thao tác truy cập dữ liệu, index nào có thể được sử dụng, index nào thực tế được sử dụng, mỗi table có bao nhiêu row được đọc và các thông tin khác.

## Lấy execution plan như thế nào?

MySQL cung cấp câu lệnh `EXPLAIN` để lấy thông tin liên quan đến execution plan.

Cần lưu ý rằng câu lệnh `EXPLAIN` tiêu chuẩn không thực sự thực thi query, mà phân tích query thông qua query optimizer, tìm ra phương án query tối ưu và hiển thị thông tin tương ứng.

MySQL 8.0.18 giới thiệu `EXPLAIN ANALYZE`, tính năng này **thực sự thực thi** query và xuất ra thời gian thực tế cùng số row của từng bước. Dữ liệu này đáng tin cậy hơn dữ liệu ước tính của `EXPLAIN` tiêu chuẩn, phù hợp để kiểm tra chuyên sâu slow query trong môi trường test:

```sql
mysql> EXPLAIN ANALYZE SELECT * FROM users WHERE age = 25\G
*************************** 1. row ***************************
EXPLAIN: -> Covering index lookup on users using idx_age_score_name (age=25)
(cost=1.52 rows=12) (actual time=0.0272..0.0344 rows=12 loops=1)
```

Ngoài ra, `EXPLAIN FORMAT=JSON` có thể xuất dữ liệu cost model của optimizer (`query_cost`), phản ánh sát hơn cost của từng bước so với dạng bảng, đặc biệt hữu ích khi tối ưu JOIN nhiều table hoặc subquery:

```sql
mysql> EXPLAIN FORMAT=JSON SELECT * FROM users WHERE age = 25\G
*************************** 1. row ***************************
EXPLAIN: {
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "1.52"
    },
    "table": {
      "table_name": "users",
      "access_type": "ref",
      "key": "idx_age_score_name",
      "rows_examined_per_scan": 12,
      "filtered": "100.00",
      "using_index": true
    }
  }
}
```

Execution plan `EXPLAIN` hỗ trợ các câu lệnh `SELECT`, `DELETE`, `INSERT`, `REPLACE` và `UPDATE`. Thông thường, nó được dùng chủ yếu để phân tích query `SELECT`. Cách sử dụng rất đơn giản, cú pháp như sau:

```sql
EXPLAIN SELECT query statement;
```

Cùng xem execution plan của một query:

**Ví dụ 1: Query một table (dùng index)**

```sql
-- Cấu trúc table: users(id, age, score, name, address), composite index idx_age_score_name(age, score, name)
mysql> EXPLAIN SELECT * FROM users WHERE age = 25;
+----+-------------+-------+------------+------+---------------------+---------------------+---------+-------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys       | key                 | key_len | ref   | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------------+---------------------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | users | NULL       | ref  | idx_age_score_name  | idx_age_score_name  | 5       | const |   12 |   100.00 | Using index |
+----+-------------+-------+------------+------+---------------------+---------------------+---------+-------+------+----------+-------------+
```

**Ví dụ 2: Query UNION (trường hợp id là NULL)**

```sql
mysql> EXPLAIN SELECT * FROM users WHERE id = 1 UNION SELECT * FROM users WHERE id = 2;
+----+--------------+------------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type  | table      | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+--------------+------------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
|  1 | PRIMARY      | users      | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
|  2 | UNION        | users      | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
|  3 | UNION RESULT | <union1,2> | NULL       | ALL   | NULL          | NULL    | NULL    | NULL  | NULL |     NULL | Using temporary |
+----+--------------+------------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
```

Có thể thấy kết quả execution plan có tổng cộng 12 cột. Ý nghĩa của từng cột được tổng hợp trong bảng sau:

| **Tên cột**   | **Ý nghĩa**                                                                               |
| ------------- | ----------------------------------------------------------------------------------------- |
| id            | Identifier thứ tự của query `SELECT`                                                      |
| select_type   | Loại query tương ứng với keyword `SELECT`                                                 |
| table         | Tên table được sử dụng                                                                    |
| partitions    | Partition khớp; với table không partition, giá trị là `NULL`                              |
| type          | Phương thức truy cập table                                                                |
| possible_keys | Index có khả năng được sử dụng                                                            |
| key           | Index thực tế được sử dụng                                                                |
| key_len       | Độ dài của index được chọn                                                                |
| ref           | Khi query bằng điều kiện equality trên index, column hoặc constant được so sánh với index |
| rows          | Số row dự kiến phải đọc                                                                   |
| filtered      | Tỷ lệ phần trăm record còn lại sau khi lọc theo điều kiện của table                       |
| Extra         | Thông tin bổ sung                                                                         |

## Phân tích kết quả EXPLAIN như thế nào?

Để phân tích kết quả thực thi của câu lệnh `EXPLAIN`, cần hiểu các field quan trọng trong execution plan.

### id

`SELECT` identifier, dùng để xác định thứ tự thực thi của từng câu lệnh `SELECT`.

Quy tắc đọc cột `id`:

- **id giống nhau**: Thực thi lần lượt từ trên xuống dưới (thường xuất hiện trong trường hợp JOIN nhiều table).
- **id khác nhau**: Giá trị id càng lớn thì execution priority càng cao (subquery được thực thi trước query bên ngoài).
- **id là NULL**: Cho biết đây là result set của UNION RESULT hoặc DERIVED table, không cần thực thi query riêng.

**Ví dụ**:

```sql
mysql> EXPLAIN SELECT * FROM users WHERE id = 1
    -> UNION
    -> SELECT * FROM users WHERE id = 2\G
*************************** 1. row ***************************
           id: 1
  select_type: PRIMARY
        table: users
         type: const
*************************** 2. row ***************************
           id: 2
  select_type: UNION
        table: users
         type: const
*************************** 3. row ***************************
           id: NULL
  select_type: UNION RESULT
        table: <union1,2>
         type: ALL
        Extra: Using temporary
```

`id = NULL` ở row thứ ba, table = `<union1,2>`, cho biết đây là kết quả hợp nhất của hai query trước đó.

### select_type

Loại query, chủ yếu dùng để phân biệt query thường, UNION, subquery và các query phức tạp khác. Các giá trị thường gặp:

- **SIMPLE**: Query đơn giản, không chứa UNION hoặc subquery.
- **PRIMARY**: Nếu query chứa subquery hoặc phần khác, `SELECT` ở query bên ngoài sẽ được đánh dấu là PRIMARY.
- **SUBQUERY**: `SELECT` đầu tiên trong subquery.
- **UNION**: `SELECT` xuất hiện sau UNION trong câu lệnh UNION.
- **DERIVED**: Subquery xuất hiện trong FROM sẽ được đánh dấu là DERIVED.
- **UNION RESULT**: Kết quả của query UNION.

### table

Tên table được sử dụng trong query. Mỗi row có table tương ứng. Ngoài table thông thường, tên table cũng có thể là các giá trị sau:

- **`<unionM,N>`**: Row này tham chiếu đến kết quả UNION của các row có id là M và N;
- **`<derivedN>`**: Row này tham chiếu đến kết quả derived table do query có id là N tạo ra. Derived table có thể được tạo từ subquery trong câu lệnh FROM.
- **`<subqueryN>`**: Row này tham chiếu đến kết quả materialized subquery do query có id là N tạo ra.

### type (quan trọng)

Loại thực thi query, mô tả cách query được thực thi. **Thứ tự từ tốt nhất đến kém nhất là**:

`system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL`

**Quy tắc kinh nghiệm để đánh giá performance**:

- **Tốt** (ít nhất nên đạt): `system`, `const`, `eq_ref`, `ref`, `range`
- **Cần chú ý**: `index_merge`, `index` (full index scan, vẫn có rủi ro performance khi lượng data lớn)
- **Cần tối ưu**: `ALL` (full table scan)

**Lưu ý**: Thứ tự này phản ánh **hiệu suất truy cập một table**, không đại diện cho performance tổng thể của query. Ví dụ, `type=ref` kèm nhiều lần đọc lại table có thể chậm hơn `type=index` sử dụng covering index.

Ý nghĩa cụ thể của một số type thường gặp:

- **system**: Table chỉ có một record (hoặc là table rỗng), storage engine có thể thống kê chính xác số row. Áp dụng cho các engine như MyISAM, Memory và InnoDB (khi table chỉ có 1 row, InnoDB sẽ tối ưu thành const). Đây là trường hợp đặc biệt của access type const.
- **const**: Table có tối đa một record khớp, có thể tìm thấy record đó chỉ bằng một lần query. Thường dùng khi tất cả field của primary key hoặc unique index được dùng làm điều kiện query.
- **eq_ref**: Khi query JOIN, row của table trước đó chỉ có đúng một row tương ứng trong table hiện tại. Đây là cách JOIN tốt nhất ngoài system và const, thường dùng khi tất cả field của primary key hoặc unique non-null index được dùng làm điều kiện JOIN (đảm bảo nghiêm ngặt match one-to-one).
- **ref**: Dùng index thường làm điều kiện query, kết quả có thể tìm thấy nhiều row phù hợp (khác với eq_ref ở chỗ một driving row có thể match nhiều driven row).
- **index_merge**: Khi mệnh đề WHERE chứa nhiều điều kiện range và mỗi điều kiện có thể dùng một index khác nhau, MySQL sẽ hợp nhất kết quả scan của nhiều index. Cột key liệt kê các index được sử dụng, cột Extra hiển thị thuật toán hợp nhất:

  - `Using union(...)`: Lấy union của nhiều kết quả index (điều kiện OR).
  - `Using sort_union(...)`: Sắp xếp kết quả index trước rồi lấy union (điều kiện OR, các column của index không được sắp xếp).
  - `Using intersection(...)`: Lấy intersection của nhiều kết quả index (điều kiện AND).

  **Ví dụ**:

  ```sql
  -- Điều kiện OR kích hoạt index merge union
  EXPLAIN SELECT * FROM employees WHERE emp_no = 10001 OR dept_no = 'd001';
  -- Extra: Using union(PRIMARY,dept_no_index)
  ```

- **range**: Thực hiện range query trên column của index; cột key trong execution plan cho biết index nào được sử dụng.
- **index**: Full Index Scan, query duyệt toàn bộ index tree. Tương tự ALL (full table scan) nhưng thường có cost thấp hơn: record trong index nhỏ hơn nhiều so với full row, nên cần ít I/O page hơn để đọc cùng số row; nếu đồng thời thỏa điều kiện covering index thì còn có thể tránh đọc lại table. Tuy nhiên, trên table cực lớn (từ hàng trăm triệu row trở lên), full index scan vẫn có thể tạo ra lượng I/O lớn. Không nên bỏ qua cost này chỉ vì type có cấp độ cao hơn ALL.
- **ALL**: Full table scan.

### possible_keys

Cột possible_keys cho biết các index mà MySQL có thể sử dụng khi thực thi query. Nếu cột này là NULL, nghĩa là không có index nào có khả năng được sử dụng. Khi đó, cần kiểm tra các column được dùng trong câu lệnh WHERE, xem có thể cải thiện performance query bằng cách thêm index cho một hoặc nhiều column đó hay không.

### key (quan trọng)

Cột key cho biết index MySQL thực tế đã sử dụng. Nếu là NULL thì nghĩa là không sử dụng index.

### key_len

Cột key_len cho biết độ dài tối đa của index MySQL thực tế sử dụng; khi dùng composite index, giá trị này có thể là tổng độ dài của nhiều column. Nếu vẫn đáp ứng yêu cầu, giá trị càng ngắn càng tốt. Nếu cột key hiển thị NULL thì cột key_len cũng hiển thị NULL.

### rows

Cột rows cho biết số row **ước tính** cần đọc để tìm record cần thiết, dựa trên thống kê của table và lựa chọn index. Giá trị càng nhỏ càng tốt.

Cần lưu ý rằng đây là giá trị ước tính, không phải giá trị chính xác. Thống kê của InnoDB dựa trên việc lấy mẫu ngẫu nhiên các index page:

- Số page được lấy mẫu do `innodb_stats_persistent_sample_pages` kiểm soát (mặc định 20 page).
- Khi data của table thường xuyên thay đổi hoặc sau khi import hàng loạt, độ lệch giữa giá trị ước tính và số row thực tế có thể đạt 10%～50% hoặc lớn hơn.
- **Bẫy table nhỏ**: Khi số row của table rất ít (chẳng hạn < 100 row), optimizer có thể bỏ qua index và chọn full table scan vì cost ước tính của full table scan thấp hơn.

**Cách xác minh**:

```sql
-- Số row ước tính trong execution plan
mysql> EXPLAIN SELECT * FROM users WHERE age = 25\G
rows: 12

-- Số row thực tế (lưu ý: thận trọng khi dùng COUNT(*) trên table lớn)
mysql> SELECT COUNT(*) FROM users WHERE age = 25;
+----------+
| COUNT(*) |
+----------+
|       12 |
+----------+
```

Khi execution plan không khớp với performance thực tế, có thể thực thi `ANALYZE TABLE` để lấy mẫu lại, sau đó quan sát thay đổi của execution plan.

### filtered

Cột filtered cho biết tỷ lệ record **ước tính** còn lại sau khi các row do storage engine trả về được lọc bằng điều kiện WHERE ở Server layer (phần trăm, 0～100). Công thức tính là: `filtered = (số row sau khi lọc điều kiện / số row do storage engine trả về) × 100`.

**Quy tắc đọc**:

- Khi `filtered = 100`: Tất cả row do storage engine trả về đều thỏa điều kiện WHERE (trường hợp lý tưởng).
- Khi `filtered < 100`: Một phần row bị Server layer loại bỏ, cho biết index chưa bao phủ toàn bộ điều kiện query.
- **Trường hợp JOIN**: Optimizer dùng `rows × (filtered / 100)` để ước tính số row mà table hiện tại truyền cho table tiếp theo (fan-out).

Field này đặc biệt quan trọng trong trường hợp JOIN nhiều table: fan-out càng lớn thì driving table cần match càng nhiều row của driven table. Vì vậy, khi giá trị `filtered` rất thấp, hiệu quả filter tốt; còn khi `rows` lớn nhưng `filtered` lại không cao, đó là tín hiệu của bottleneck performance tiềm ẩn. Khi đó nên ưu tiên dùng index condition pushdown (ICP) hoặc index phù hợp hơn để giảm fan-out.

### Extra (quan trọng)

Cột này chứa thông tin bổ sung về cách MySQL parse query. Thông qua thông tin này, có thể hiểu rõ hơn MySQL thực sự thực thi query như thế nào. Các giá trị thường gặp:

- **Using filesort**: MySQL không thể dùng index để hoàn thành yêu cầu sort của ORDER BY hoặc GROUP BY, nên cần thực hiện thêm một lần sort sau khi trả về result set. Khi kích thước result set nằm trong `sort_buffer_size`, việc sort được thực hiện trong memory; nếu vượt quá thì dùng temporary disk file. "filesort" là tên còn sót lại từ trước, không có nghĩa là chắc chắn phát sinh disk I/O.
- **Using temporary**: MySQL cần tạo temporary table để lưu kết quả query, thường gặp trong ORDER BY và GROUP BY.
- **Using index**: Cho biết query sử dụng covering index, không cần đọc lại table, hiệu quả query rất cao.
- **Using index condition**: Cho biết query optimizer đã chọn sử dụng tính năng index condition pushdown.
- **Using where**: Server layer của MySQL áp dụng filter WHERE bổ sung lên các row do storage engine trả về. Ngay cả khi đã sử dụng index (chẳng hạn `type=ref`), nếu index chỉ đáp ứng được một phần điều kiện query thì các điều kiện còn lại vẫn phải được filter ở Server layer; khi đó cũng sẽ xuất hiện `Using where`.
- **Using join buffer (Block Nested Loop)**: Khi query JOIN, nếu driven table không sử dụng index, MySQL sẽ đọc data của driving table vào join buffer trước, sau đó duyệt driven table để match (độ phức tạp O(N×M)).
- **Using join buffer (hash join)**: MySQL giới thiệu thuật toán Hash Join trong phiên bản 8.0.18, **chỉ dùng cho equi-join** (chẳng hạn `t1.id = t2.id`); từ 8.0.20, thuật toán này mặc định thay thế BNL. Độ phức tạp của Hash Join là O(N) ở giai đoạn build + O(M) ở giai đoạn probe, hiệu quả hơn O(N×M) của BNL.

  **Các trường hợp ngoại lệ** (vẫn quay về BNL):

  - Non-equi-join (chẳng hạn `t1.id > t2.id`).
  - Điều kiện JOIN chứa function hoặc expression.
  - Driven table có index khả dụng (khi đó sẽ dùng Index Nested Loop).

Cần lưu ý rằng khi cột Extra chứa Using filesort hoặc Using temporary, performance của MySQL có thể gặp vấn đề, vì vậy nên cố gắng tránh các trường hợp này.

## Tham khảo

- <https://dev.mysql.com/doc/refman/8.0/en/explain-output.html>
- <https://dev.mysql.com/doc/refman/8.0/en/explain.html>
- <https://juejin.cn/post/6953444668973514789>

<!-- @include: @article-footer.snippet.md -->
