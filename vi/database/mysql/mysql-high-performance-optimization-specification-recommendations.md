---
title: "Tổng hợp khuyến nghị về quy chuẩn tối ưu hóa hiệu năng MySQL"
description: "Tổng hợp khuyến nghị về quy chuẩn tối ưu hóa hiệu năng MySQL, bao quát quy chuẩn đặt tên database, quy chuẩn thiết kế table, quy chuẩn thiết kế field, quy chuẩn thiết kế index, quy chuẩn viết SQL..., giúp bạn xây dựng hệ thống database hiệu quả và ổn định."
category: Database
tag:
  - MySQL
head:
  - - meta
    - name: keywords
      content: "quy chuẩn tối ưu hóa MySQL,quy chuẩn thiết kế database,thiết kế index,quy chuẩn viết SQL,tối ưu slow query,lựa chọn kiểu field,thiết kế cấu trúc table"
---

> Tác giả: Tingfeng Địa chỉ bài gốc: <https://www.cnblogs.com/huchong/p/10219318.html>.
>
> JavaGuide đã được tác giả cho phép sử dụng và đã hoàn thiện, bổ sung nội dung bài gốc.

## Quy chuẩn đặt tên database

- Tên của mọi database object phải dùng chữ cái thường và phân tách bằng dấu gạch dưới.
- Tên của mọi database object không được dùng từ khóa dành riêng của MySQL (nếu tên table chứa từ khóa, khi truy vấn cần đặt nó trong dấu nháy đơn).
- Việc đặt tên database object phải giúp nhìn tên hiểu nghĩa, tốt nhất không vượt quá 32 ký tự.
- Database và table tạm thời phải có tiền tố `tmp_` và hậu tố là ngày; table backup phải có tiền tố `bak_` và hậu tố là ngày (timestamp).
- Tên và kiểu của các column lưu trữ cùng một loại dữ liệu phải nhất quán (thường được dùng làm column liên kết; nếu kiểu của các column liên kết không nhất quán khi truy vấn, database sẽ tự động thực hiện implicit conversion, khiến index trên column mất hiệu lực và làm giảm hiệu suất truy vấn).

## Quy chuẩn thiết kế cơ bản cho database

### Mọi table phải dùng storage engine InnoDB

Nếu không có yêu cầu đặc biệt (tức là chức năng InnoDB không thể đáp ứng, như columnar storage, lưu trữ dữ liệu không gian...), mọi table phải dùng storage engine InnoDB (trước MySQL 5.5 mặc định dùng MyISAM, từ 5.6 trở đi mặc định là InnoDB).

InnoDB hỗ trợ transaction, row-level lock, khả năng khôi phục tốt hơn và hiệu năng tốt hơn khi concurrency cao.

### Database và table thống nhất dùng character set UTF8

Khả năng tương thích tốt hơn. Character set thống nhất có thể tránh lỗi ký tự do chuyển đổi character set. Việc phải chuyển đổi trước khi so sánh các character set khác nhau sẽ khiến index mất hiệu lực. Nếu database cần lưu trữ emoji, phải dùng character set utf8mb4.

Bạn nên đọc bài viết tôi đã viết: [Giải thích chi tiết về character set MySQL](../character-set.md).

### Mọi table và field đều cần thêm comment

Dùng mệnh đề comment để thêm ghi chú cho table và column, đồng thời duy trì data dictionary ngay từ đầu.

### Cố gắng kiểm soát kích thước dữ liệu của một table, nên giữ trong phạm vi 5 triệu dòng

5 triệu không phải là giới hạn của database MySQL. Kích thước quá lớn sẽ gây nhiều vấn đề khi sửa cấu trúc table, backup và khôi phục.

Có thể dùng các cách như archive dữ liệu lịch sử (áp dụng cho dữ liệu log), sharding database và table (áp dụng cho dữ liệu nghiệp vụ) để kiểm soát kích thước dữ liệu.

### Thận trọng khi sử dụng partitioned table của MySQL

Partitioned table biểu hiện là nhiều file về mặt vật lý, nhưng là một table về mặt logic.

Hãy chọn partition key thận trọng, vì hiệu suất truy vấn xuyên partition có thể còn thấp hơn.

Nên dùng cách tách table vật lý để quản lý dữ liệu lớn.

### Đặt các column thường được dùng cùng nhau vào một table

Tránh thêm các thao tác join.

### Cấm tạo field dự phòng trong table

- Tên của field dự phòng rất khó thể hiện đúng ý nghĩa.
- Không thể xác định kiểu dữ liệu lưu trong field dự phòng, nên không thể chọn kiểu phù hợp.
- Việc sửa kiểu của field dự phòng sẽ lock table.

### Cấm lưu trữ file (chẳng hạn hình ảnh) và các dữ liệu binary lớn trong database

Lưu file trong database sẽ ảnh hưởng nghiêm trọng đến hiệu năng database và tiêu tốn quá nhiều storage.

Các file (chẳng hạn hình ảnh), tức những dữ liệu binary lớn, thường được lưu trên file server, database chỉ lưu thông tin địa chỉ file.

### Không nên bị ràng buộc bởi các database normal forms

Thông thường, khi thiết kế relational database cần đáp ứng Third Normal Form, nhưng để đáp ứng Third Normal Form, chúng ta có thể phải tách thành nhiều table. Khi truy vấn phải thực hiện join nhiều table, đôi khi để nâng cao hiệu suất truy vấn, có thể hạ yêu cầu về normal form và lưu một lượng thông tin redundant nhất định trong table, cách này còn gọi là denormalization. Tuy nhiên, cần chú ý denormalization phải có mức độ phù hợp.

### Cấm thực hiện stress test database trên môi trường production

### Cấm kết nối trực tiếp từ database môi trường development/test đến database production

Rủi ro bảo mật cực kỳ lớn, cần luôn thận trọng với môi trường production!

## Quy chuẩn thiết kế field trong database

### Ưu tiên chọn kiểu dữ liệu nhỏ nhất đáp ứng nhu cầu lưu trữ

Số byte lưu trữ càng nhỏ thì không gian chiếm dụng càng ít và hiệu năng cũng càng tốt.

**a. Một số string có thể chuyển thành kiểu số để lưu trữ, chẳng hạn có thể chuyển địa chỉ IP thành dữ liệu integer.**

Số có tính liên tục, hiệu năng tốt hơn và cũng chiếm ít không gian hơn.

MySQL cung cấp hai method để xử lý địa chỉ IP:

- `INET_ATON()`: chuyển ip thành unsigned integer (4-8 bit);
- `INET_NTOA()`: chuyển IP dạng integer thành địa chỉ.

Trước khi insert dữ liệu, dùng `INET_ATON()` chuyển địa chỉ IP thành integer; khi hiển thị dữ liệu, dùng `INET_NTOA()` chuyển IP dạng integer thành địa chỉ để hiển thị.

**b. Với dữ liệu không âm (như ID tự tăng, IP dạng integer, tuổi), ưu tiên dùng unsigned integer để lưu trữ.**

So với signed, unsigned có thể cung cấp phạm vi lưu trữ lớn gấp đôi:

```sql
SIGNED INT -2147483648~2147483647
UNSIGNED INT 0~4294967295
```

**c. Với kiểu dữ liệu có giá trị nhỏ (chẳng hạn tuổi, trạng thái biểu thị bằng 0/1), ưu tiên dùng kiểu TINYINT.**

### Tránh dùng kiểu dữ liệu TEXT, BLOB; kiểu TEXT thường gặp nhất có thể lưu 64 KB dữ liệu

**a. Nên tách column BLOB hoặc TEXT vào một extension table riêng.**

Temporary table trong memory của MySQL không hỗ trợ các kiểu dữ liệu lớn như TEXT, BLOB. Nếu query chứa các dữ liệu này, khi thực hiện các thao tác như sort sẽ không thể dùng temporary table trong memory mà phải dùng temporary table trên disk. Ngoài ra, với loại dữ liệu này, MySQL vẫn phải thực hiện query lần hai, khiến performance của SQL giảm mạnh, nhưng điều đó không có nghĩa là tuyệt đối không được dùng các kiểu dữ liệu này.

Nếu nhất định phải dùng, nên tách column BLOB hoặc TEXT vào một extension table riêng. Khi query tuyệt đối không dùng `select *` mà chỉ lấy các column cần thiết; nếu không cần dữ liệu của column TEXT thì không query column đó.

**2. Kiểu TEXT hoặc BLOB chỉ có thể dùng prefix index**

Vì MySQL giới hạn độ dài của field trong index, kiểu TEXT chỉ có thể dùng prefix index, đồng thời column TEXT không thể có default value.

### Tránh dùng kiểu ENUM

- Muốn sửa giá trị ENUM phải dùng câu lệnh ALTER.
- Hiệu năng thao tác ORDER BY trên kiểu ENUM thấp, cần thêm thao tác xử lý.
- Kiểu dữ liệu ENUM tồn tại một số giới hạn, chẳng hạn không nên dùng số làm giá trị của ENUM.

Đọc thêm: [Có nên dùng kiểu enum của MySQL không? - Tóm lược kiến trúc - Zhihu](https://www.zhihu.com/question/404422255/answer/1661698499).

### Cố gắng định nghĩa mọi column là NOT NULL

Trừ khi có lý do đặc biệt phải dùng giá trị NULL, nếu không field luôn nên được định nghĩa là NOT NULL.

- Index trên column NULL cần thêm không gian để lưu trữ, nên chiếm nhiều không gian hơn.
- Khi so sánh và tính toán, phải xử lý riêng giá trị NULL.

Đọc thêm: [Chia sẻ kỹ thuật | Lựa chọn default value trong MySQL (rỗng hay NULL)](https://opensource.actionsky.com/20190710-mysql/).

### Tuyệt đối không dùng string để lưu trữ ngày tháng

Với kiểu ngày tháng, tuyệt đối không dùng string để lưu trữ ngày tháng. Có thể cân nhắc DATETIME, TIMESTAMP và timestamp dạng số.

Ba cách này đều có ưu điểm riêng. Lựa chọn cách phù hợp nhất tùy trường hợp thực tế mới là điều quan trọng. Dưới đây là so sánh đơn giản giữa ba cách để bạn lựa chọn đúng kiểu dữ liệu lưu thời gian trong quá trình phát triển thực tế:

| Kiểu              | Không gian lưu trữ | Định dạng ngày tháng           | Phạm vi ngày tháng                                           | Có thông tin timezone không |
| ----------------- | ------------------ | ------------------------------ | ------------------------------------------------------------ | --------------------------- |
| DATETIME          | 5~8 byte           | YYYY-MM-DD hh:mm:ss[.fraction] | 1000-01-01 00:00:00[.000000] ～ 9999-12-31 23:59:59[.999999] | Không                       |
| TIMESTAMP         | 4~7 byte           | YYYY-MM-DD hh:mm:ss[.fraction] | 1970-01-01 00:00:01[.000000] ～ 2038-01-19 03:14:07[.999999] | Có                          |
| Timestamp dạng số | 4 byte             | Toàn số, chẳng hạn 1578707612  | Sau 1970-01-01 00:00:01                                      | Không                       |

Xem giới thiệu chi tiết về lựa chọn kiểu thời gian của MySQL tại bài viết này: [Khuyến nghị lưu trữ dữ liệu kiểu thời gian MySQL](https://javaguide.cn/database/mysql/some-thoughts-on-database-storage-time.html).

### Dữ liệu số tiền liên quan đến tài chính bắt buộc dùng kiểu decimal

- **Floating-point không chính xác**: float, double
- **Floating-point chính xác**: decimal

Kiểu decimal là số floating-point chính xác, không làm mất precision khi tính toán. Không gian chiếm dụng phụ thuộc vào width được định nghĩa. Cứ mỗi 4 byte có thể lưu 9 chữ số và dấu thập phân chiếm 1 byte. Ngoài ra, decimal có thể dùng để lưu integer lớn hơn bigint.

Tuy nhiên, do decimal cần thêm không gian và chi phí tính toán, chỉ nên dùng decimal khi cần tính toán dữ liệu chính xác.

### Một table không nên chứa quá nhiều field

Nếu một table chứa quá nhiều field, có thể cân nhắc tách thành nhiều table, khi cần thì thêm intermediate table để liên kết.

## Quy chuẩn thiết kế index

### Giới hạn số lượng index trên mỗi table, nên giữ không quá 5 index cho mỗi table

Index không phải càng nhiều càng tốt! Index có thể nâng cao hiệu suất nhưng cũng có thể làm giảm hiệu suất.

Index có thể tăng hiệu suất query, nhưng đồng thời cũng làm giảm hiệu suất insert và update, thậm chí trong một số trường hợp còn làm giảm hiệu suất query.

Khi optimize query, MySQL optimizer sẽ dựa trên thông tin thống kê để đánh giá từng index có thể sử dụng, từ đó tạo ra execution plan tốt nhất. Nếu đồng thời có nhiều index có thể dùng cho query, thời gian MySQL optimizer tạo execution plan sẽ tăng, đồng thời performance query cũng giảm.

### Cấm dùng full-text index

Full-text index không phù hợp với trường hợp OLTP.

### Cấm tạo index riêng cho từng column trong table

Trước version 5.6, một SQL chỉ có thể dùng một index của một table; từ 5.6 trở đi, tuy đã có cách optimize bằng Index Merge, nhưng vẫn kém xa query dùng một composite index.

### Mỗi table InnoDB phải có một primary key

InnoDB là một index-organized table: thứ tự logic lưu trữ dữ liệu và thứ tự của index giống nhau. Mỗi table có thể có nhiều index, nhưng thứ tự lưu trữ của table chỉ có thể có một.

InnoDB tổ chức table theo thứ tự của primary key index.

- Không dùng column được update thường xuyên làm primary key, không dùng primary key nhiều column (tương đương composite index).
- Không dùng UUID, MD5, HASH, string column làm primary key (không thể bảo đảm dữ liệu tăng theo thứ tự).
- Nên dùng giá trị ID tự tăng làm primary key.

### Khuyến nghị về các column thường dùng cho index

- Các column xuất hiện trong mệnh đề WHERE của câu lệnh SELECT, UPDATE, DELETE.
- Các field nằm trong ORDER BY, GROUP BY, DISTINCT.
- Không tạo một index cho từng column thuộc mục 1 và 2; thông thường tạo composite index cho các field ở mục 1 và 2 sẽ hiệu quả hơn.
- Các column dùng để join nhiều table.

### Cách chọn thứ tự column trong index

Mục đích tạo index là tìm dữ liệu thông qua index, giảm random IO và tăng performance query. Index lọc được càng ít dữ liệu thì dữ liệu đọc từ disk vào càng ít.

- **Đặt column có độ phân biệt cao nhất ở bên trái của composite index**: Đây là nguyên tắc quan trọng nhất. Độ phân biệt càng cao thì dữ liệu được lọc qua index càng ít, thao tác I/O cũng càng ít. Cách tính độ phân biệt là `count(distinct column) / count(*)`.
- **Đặt column được sử dụng thường xuyên nhất ở bên trái của composite index**: Điều này phù hợp với nguyên tắc leftmost prefix matching. Đặt column điều kiện query thường dùng nhất ở bên trái có thể tận dụng index ở mức tối đa.
- **Độ dài field**: Độ dài field ảnh hưởng rất ít đến non-leaf node của composite index vì node này lưu giá trị của toàn bộ các field trong composite index. Độ dài field chủ yếu ảnh hưởng đến không gian lưu trữ của primary key và các field nằm trong index khác, cũng như kích thước leaf node của các index này. Vì vậy, khi chọn thứ tự column của composite index, độ ưu tiên của độ dài field thấp nhất. Với primary key và các field nằm trong index khác, chọn field ngắn hơn có thể tiết kiệm không gian lưu trữ và nâng cao performance I/O.

### Tránh tạo redundant index và duplicate index (làm tăng thời gian query optimizer tạo execution plan)

- Ví dụ về duplicate index: primary key(id), index(id), unique index(id).
- Ví dụ về redundant index: index(a,b,c), index(a,b), index(a).

### Với các query thường xuyên, ưu tiên cân nhắc dùng covering index

> Covering index: là index chứa tất cả field được query (các field trong where, select, order by, group by).

**Lợi ích của covering index**:

- **Tránh query lần hai trên table InnoDB, tức thao tác quay lại table**: InnoDB lưu trữ theo thứ tự clustered index. Với InnoDB, leaf node của secondary index lưu thông tin primary key của row. Nếu dùng secondary index để query dữ liệu, sau khi tìm được key tương ứng vẫn phải query lần hai thông qua primary key mới lấy được dữ liệu thực sự cần. Trong covering index, có thể lấy toàn bộ dữ liệu từ key value của secondary index, tránh query primary key lần hai (quay lại table), giảm thao tác IO và nâng cao hiệu suất query.
- **Có thể biến random IO thành sequential IO để tăng tốc query**: Vì covering index được lưu theo thứ tự key value, với các range query thiên về IO, số IO sẽ ít hơn nhiều so với đọc ngẫu nhiên từng row từ disk. Do đó, khi truy cập bằng covering index, random read trên disk cũng có thể được chuyển thành sequential IO khi tìm kiếm index.

---

### Quy chuẩn SET index

**Cố gắng tránh dùng foreign key constraint**

- Không khuyến nghị dùng foreign key constraint, nhưng nhất định phải tạo index trên các key liên kết giữa các table.
- Foreign key có thể dùng để bảo đảm referential integrity của dữ liệu, nhưng nên thực hiện ở phía business.
- Foreign key ảnh hưởng đến thao tác ghi của parent table và child table, từ đó làm giảm performance.

## Quy chuẩn phát triển SQL trong database

### Cố gắng không tính toán trong database, phép tính phức tạp cần chuyển sang business application

Cố gắng không tính toán trong database, phép tính phức tạp cần chuyển sang business application. Điều này có thể tránh database chịu tải quá nặng, ảnh hưởng đến performance và stability của database. Vai trò chính của database là lưu trữ và quản lý dữ liệu, không phải xử lý dữ liệu.

### Tối ưu các câu lệnh SQL ảnh hưởng lớn đến performance

Cần tìm các câu lệnh SQL cần optimize nhất. Đó có thể là câu lệnh được sử dụng thường xuyên nhất hoặc câu lệnh có mức cải thiện rõ rệt nhất sau khi optimize. Có thể phát hiện các câu lệnh SQL cần optimize bằng cách truy vấn slow query log của MySQL.

### Tận dụng đầy đủ các index đã tồn tại trên table

Tránh dùng điều kiện query có hai dấu `%`. Ví dụ: `a like '%123%'` (nếu không có `%` ở đầu mà chỉ có `%` ở cuối thì có thể dùng index trên column).

Một SQL chỉ có thể tận dụng một column trong composite index để thực hiện range query. Ví dụ: có composite index trên các column a,b,c, trong điều kiện query có range query trên column a thì index trên column b,c sẽ không được dùng.

Khi định nghĩa composite index, nếu column a cần dùng range search thì phải đặt column a ở bên phải composite index. Dùng left join hoặc not exists để optimize thao tác not in, vì not in cũng thường khiến index mất hiệu lực.

### Cấm dùng SELECT \*; bắt buộc dùng SELECT <danh sách field> để query

- `SELECT *` sẽ tiêu tốn nhiều CPU hơn.
- Các field không cần thiết của `SELECT *` làm tăng mức sử dụng network bandwidth và thời gian truyền dữ liệu, đặc biệt là các field lớn (như varchar, blob, text).
- `SELECT *` không thể tận dụng tối ưu hóa covering index của MySQL optimizer (dựa trên strategy “covering index” của MySQL optimizer, đây là cách optimize query có tốc độ cực nhanh, hiệu quả cực cao và được giới chuyên môn khuyến nghị rộng rãi).
- `SELECT <danh sách field>` có thể giảm ảnh hưởng do thay đổi cấu trúc table.

### Cấm dùng câu lệnh INSERT không có danh sách field

**Không khuyến nghị**:

```sql
insert into t values ('a','b','c');
```

**Khuyến nghị**:

```sql
insert into t(c1,c2,c3) values ('a','b','c');
```

### Nên dùng prepared statement để thực hiện thao tác database

- Prepared statement có thể tái sử dụng các plan này, giảm thời gian cần để compile SQL và giải quyết vấn đề SQL injection do dynamic SQL gây ra.
- Chỉ truyền parameter sẽ hiệu quả hơn truyền câu lệnh SQL.
- Câu lệnh giống nhau có thể parse một lần, sử dụng nhiều lần, nâng cao hiệu suất xử lý.

### Tránh implicit conversion kiểu dữ liệu

Implicit conversion sẽ khiến index mất hiệu lực, ví dụ:

```sql
select name,phone from customer where id = '111';
```

Xem giải thích chi tiết trong bài viết: [Implicit conversion trong MySQL khiến index mất hiệu lực](./index-invalidation-caused-by-implicit-conversion.md).

### Tránh dùng subquery, có thể optimize subquery thành thao tác join

Thông thường, chỉ khi subquery nằm trong mệnh đề in và subquery là SQL đơn giản (không chứa mệnh đề union, group by, order by, limit) thì mới có thể chuyển subquery thành truy vấn liên kết để optimize.

**Lý do subquery có performance kém**: result set của subquery không thể dùng index. Thông thường result set của subquery được lưu vào temporary table, dù là temporary table trong memory hay trên disk cũng đều không có index, nên performance query sẽ bị ảnh hưởng nhất định. Đặc biệt với subquery trả về result set tương đối lớn, ảnh hưởng đến performance query cũng càng lớn. Vì subquery tạo ra nhiều temporary table không có index nên sẽ tiêu tốn quá nhiều tài nguyên CPU và IO, tạo ra nhiều slow query.

### Tránh dùng JOIN với quá nhiều table

Với MySQL, có join cache và kích thước cache có thể được thiết lập bằng parameter join_buffer_size.

Trong MySQL, mỗi table được join thêm trong một SQL sẽ khiến hệ thống cấp phát thêm một join cache. Một SQL liên kết càng nhiều table thì memory chiếm dụng càng lớn.

Nếu chương trình sử dụng nhiều thao tác liên kết nhiều table và join_buffer_size được thiết lập không hợp lý, rất dễ làm server hết memory, ảnh hưởng đến performance và stability của database server.

Đồng thời, thao tác liên kết sẽ tạo ra thao tác temporary table, ảnh hưởng đến hiệu suất query. MySQL cho phép liên kết tối đa 61 table; khuyến nghị không quá 5 table.

### Giảm số lần tương tác với cùng một database

Database phù hợp hơn với việc xử lý batch. Gộp nhiều thao tác giống nhau lại có thể nâng cao hiệu suất xử lý.

### Khi dùng OR để kiểm tra cùng một column, dùng IN thay cho OR

Số lượng giá trị trong IN không nên vượt quá 500. Thao tác IN có thể tận dụng index hiệu quả hơn, còn OR trong đa số trường hợp rất khó tận dụng index.

### Cấm dùng order by rand() để random sort

`order by rand()` sẽ nạp toàn bộ dữ liệu thỏa điều kiện trong table vào memory, sau đó sort toàn bộ dữ liệu trong memory theo giá trị được tạo ngẫu nhiên, đồng thời có thể tạo một giá trị ngẫu nhiên cho từng row. Nếu data set thỏa điều kiện rất lớn, thao tác này sẽ tiêu tốn nhiều tài nguyên CPU, IO và memory.

Nên lấy một giá trị ngẫu nhiên trong chương trình, sau đó lấy dữ liệu từ database.

### Trong mệnh đề WHERE, cấm chuyển đổi và tính toán bằng function trên column

Thực hiện chuyển đổi hoặc tính toán bằng function trên column sẽ khiến không thể dùng index.

**Không khuyến nghị**:

```sql
where date(create_time)='20190101'
```

**Khuyến nghị**:

```sql
where create_time >= '20190101' and create_time < '20190102'
```

### Khi rõ ràng không có giá trị trùng, dùng UNION ALL thay vì UNION

- UNION đưa toàn bộ dữ liệu của hai result set vào temporary table rồi mới thực hiện thao tác loại bỏ duplicate.
- UNION ALL không thực hiện thao tác loại bỏ duplicate trên result set.

### Tách SQL lớn phức tạp thành nhiều SQL nhỏ

- SQL lớn có logic tương đối phức tạp hoặc cần nhiều CPU để tính toán.
- Trong MySQL, một SQL chỉ có thể dùng một CPU để tính toán.
- Sau khi tách SQL, có thể nâng cao hiệu suất xử lý bằng cách thực thi song song.

### Chương trình kết nối các database khác nhau phải dùng account khác nhau, cấm query xuyên database

- Dành dư địa cho việc di chuyển database và sharding database và table.
- Giảm coupling nghiệp vụ.
- Tránh rủi ro bảo mật do quyền quá lớn.

## Quy chuẩn hành vi thao tác database

### Với thao tác ghi batch (UPDATE, DELETE, INSERT) trên 1 triệu row, phải chia thành nhiều batch để thực hiện

**Thao tác khối lượng lớn có thể gây replication lag nghiêm trọng**

Trong môi trường primary-secondary, thao tác khối lượng lớn có thể gây replication lag nghiêm trọng. Thao tác ghi khối lượng lớn thường cần thời gian tương đối dài, và chỉ được thực hiện trên các secondary database sau khi hoàn tất trên primary database, nên sẽ gây độ trễ kéo dài giữa primary database và secondary database.

**Khi binlog ở row format sẽ tạo ra lượng log lớn**

Thao tác ghi khối lượng lớn sẽ tạo ra lượng log lớn, đặc biệt với binary data ở row format. Vì row format ghi lại thay đổi của từng row, nên càng sửa nhiều dữ liệu trong một lần thì lượng log tạo ra càng nhiều, thời gian truyền và khôi phục log cũng càng dài. Đây cũng là một nguyên nhân gây replication lag.

**Tránh tạo thao tác transaction lớn**

Việc sửa dữ liệu khối lượng lớn nhất định diễn ra trong một transaction, khiến lượng lớn dữ liệu trong table bị lock và tạo ra nhiều blocking. Blocking ảnh hưởng rất lớn đến performance MySQL.

Đặc biệt, blocking kéo dài sẽ chiếm toàn bộ connection khả dụng của database, khiến các application khác trong môi trường production không thể kết nối database. Vì vậy, cần chú ý chia batch khi thực hiện thao tác ghi khối lượng lớn.

### Với table lớn, dùng pt-online-schema-change để sửa cấu trúc table

- Tránh replication lag do sửa table lớn gây ra.
- Tránh lock table khi sửa field của table.

Phải thận trọng khi sửa cấu trúc dữ liệu của table lớn, vì thao tác này sẽ gây lock table nghiêm trọng, đặc biệt trong môi trường production thì không thể chấp nhận.

pt-online-schema-change trước hết tạo một table mới có cấu trúc giống table gốc và sửa cấu trúc table trên table mới. Sau đó, nó copy dữ liệu từ table gốc sang table mới, đồng thời thêm một số trigger vào table gốc để copy dữ liệu mới thêm của table gốc sang table mới. Sau khi copy xong toàn bộ dữ liệu row, table mới được đổi tên thành table gốc và table cũ bị xóa. Nhờ vậy, một thao tác DDL ban đầu được tách thành nhiều batch nhỏ.

### Cấm cấp quyền super cho account dùng cho program

- Khi đạt giới hạn connection tối đa, vẫn giữ lại 1 connection cho user có quyền super.
- Quyền super chỉ được dành cho account mà DBA dùng để xử lý vấn đề.

### Account kết nối database của program phải tuân thủ nguyên tắc quyền tối thiểu

- Account database mà program sử dụng chỉ được dùng trong một DB, không được query xuyên database.
- Về nguyên tắc, account mà program sử dụng không được có quyền drop.

## Đọc thêm

- [Quy chuẩn thiết kế MySQL mà người làm kỹ thuật nhất định phải biết, tất cả đều là bài học đau đớn - Alibaba Developer](https://mp.weixin.qq.com/s/XC8e5iuQtfsrEOERffEZ-Q)
- [Trao đổi về 15 mẹo nhỏ khi tạo table database](https://mp.weixin.qq.com/s/NM-aHaW6TXrnO6la6Jfl5A)

<!-- @include: @article-footer.snippet.md -->
