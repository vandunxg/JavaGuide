---
title: Đề xuất lựa chọn kiểu dữ liệu ngày tháng trong MySQL
description: So sánh chuyên sâu sự khác biệt giữa DATETIME và TIMESTAMP trong MySQL, phân tích xử lý time zone, dung lượng lưu trữ, phạm vi giá trị và đưa ra đề xuất thực tiễn tốt nhất khi lựa chọn kiểu dữ liệu ngày tháng.
category: Database
tag:
  - MySQL
head:
  - - meta
    - name: keywords
      content: Lưu trữ thời gian MySQL,DATETIME,TIMESTAMP,time stamp,xử lý time zone,lựa chọn kiểu dữ liệu ngày tháng,hàm ngày tháng MySQL
---

Trong công việc phát triển phần mềm hằng ngày, lưu trữ thời gian là một nhu cầu cơ bản và thường gặp. Dù là ghi lại thời điểm thao tác dữ liệu, thời điểm phát sinh giao dịch tài chính, thời điểm khởi hành của chuyến đi hay thời điểm người dùng đặt hàng, thông tin thời gian đều gắn chặt với logic nghiệp vụ và chức năng hệ thống. Vì vậy, việc lựa chọn và sử dụng đúng các kiểu dữ liệu ngày giờ của MySQL rất quan trọng; lựa chọn phù hợp hay không thậm chí có thể ảnh hưởng đáng kể đến tính chính xác của nghiệp vụ và tính ổn định của hệ thống.

Bài viết này nhằm giúp developer xem xét lại và hiểu sâu hơn các cách lưu trữ thời gian khác nhau trong MySQL, từ đó đưa ra lựa chọn phù hợp hơn cho từng trường hợp nghiệp vụ của dự án.

## Không dùng string để lưu trữ ngày tháng

Giống như nhiều người mới học database, trong giai đoạn đầu học tập, tác giả cũng từng thử dùng kiểu string (chẳng hạn kiểu VARCHAR) để lưu trữ ngày tháng và thời gian, thậm chí có lúc còn cho rằng đây là một cách đơn giản, trực quan. Dù sao thì định dạng `'YYYY-MM-DD HH:MM:SS'` trông cũng rõ ràng, dễ hiểu.

Tuy nhiên, đây không phải cách làm đúng, chủ yếu vì có hai vấn đề sau:

1. **Hiệu quả sử dụng không gian**: So với các kiểu ngày giờ có sẵn của MySQL, string thường cần nhiều không gian lưu trữ hơn để biểu diễn cùng một thông tin thời gian.
2. **Hiệu quả truy vấn và tính toán thấp**:
   - **Thao tác so sánh phức tạp và kém hiệu quả**: So sánh ngày tháng dựa trên string cần so sánh từng ký tự theo thứ tự từ điển. Cách này không chỉ thiếu trực quan (ví dụ, `'2024-05-01'` sẽ nhỏ hơn `'2024-1-10'`) mà còn kém hiệu quả hơn nhiều so với việc dùng kiểu ngày giờ gốc để so sánh giá trị số hoặc thời điểm.
   - **Khả năng tính toán bị hạn chế**: Không thể trực tiếp sử dụng các hàm ngày giờ phong phú do database cung cấp để tính toán (chẳng hạn tính khoảng cách giữa hai ngày, cộng hoặc trừ ngày tháng), mà phải chuyển đổi định dạng trước, làm tăng độ phức tạp.
   - **Hiệu năng index không tốt**: Index dựa trên string thường kém hiệu quả và linh hoạt hơn index của kiểu ngày giờ gốc khi xử lý truy vấn phạm vi (chẳng hạn tìm dữ liệu trong một khoảng thời gian cụ thể).

## Lựa chọn DATETIME và TIMESTAMP

`DATETIME` và `TIMESTAMP` là hai kiểu dữ liệu rất thường dùng trong MySQL để lưu trữ dữ liệu chứa thông tin ngày tháng và thời gian. Cả hai đều có thể lưu giá trị thời gian chính xác đến giây (MySQL 5.6.4+ hỗ trợ phần giây có độ chính xác cao hơn). Vậy trong ứng dụng thực tế, nên lựa chọn giữa hai kiểu này như thế nào?

Dưới đây, chúng ta sẽ so sánh chúng theo một số khía cạnh quan trọng:

### Thông tin time zone

Kiểu `DATETIME` lưu trữ **giá trị ngày tháng và thời gian theo literal**, bản thân nó **không chứa bất kỳ thông tin time zone nào**. Khi bạn chèn một giá trị `DATETIME`, MySQL sẽ lưu chính xác thời gian bạn cung cấp và không thực hiện chuyển đổi time zone nào.

**Điều này gây ra vấn đề gì?** Nếu ứng dụng cần hỗ trợ nhiều time zone, hoặc time zone của server và client có thể thay đổi, khi dùng `DATETIME`, ứng dụng phải tự xử lý việc chuyển đổi và diễn giải time zone. Nếu xử lý không đúng (chẳng hạn giả định mọi thời gian được lưu đều thuộc cùng một time zone, nhưng môi trường thực tế đã thay đổi), có thể dẫn đến sự nhầm lẫn khi hiển thị hoặc tính toán thời gian.

**`TIMESTAMP` có liên quan đến time zone**. Khi lưu trữ, MySQL sẽ chuyển giá trị thời gian trong time zone của session hiện tại thành UTC (Coordinated Universal Time) để lưu trữ nội bộ. Khi truy vấn trường `TIMESTAMP`, MySQL lại chuyển thời gian UTC đã lưu về time zone được thiết lập cho session hiện tại để hiển thị.

Điều này có nghĩa là khi truy vấn trường `TIMESTAMP` của cùng một bản ghi trong các session có thiết lập time zone khác nhau, bạn có thể thấy các biểu diễn giờ địa phương khác nhau, nhưng chúng đều tương ứng với cùng một thời điểm tuyệt đối (thời gian UTC). Điều này rất hữu ích với các ứng dụng cần hỗ trợ toàn cầu và nhiều time zone.

Hãy cùng xem ví dụ thực tế!

Câu lệnh SQL tạo bảng:

```sql
CREATE TABLE `time_zone_test` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `date_time` datetime DEFAULT NULL,
  `time_stamp` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

Chèn một bản ghi (giả sử time zone của session hiện tại là mặc định của hệ thống, chẳng hạn UTC+0):

```sql
INSERT INTO time_zone_test(date_time,time_stamp) VALUES(NOW(),NOW());
```

Truy vấn dữ liệu (trong session cùng time zone):

```sql
SELECT date_time, time_stamp FROM time_zone_test;
```

Kết quả:

```plain
+---------------------+---------------------+
| date_time           | time_stamp          |
+---------------------+---------------------+
| 2020-01-11 09:53:32 | 2020-01-11 09:53:32 |
+---------------------+---------------------+
```

Bây giờ, thay đổi time zone của session hiện tại thành múi giờ phía Đông 8 (UTC+8):

```sql
SET time_zone = '+8:00';
```

Truy vấn dữ liệu lần nữa:

```bash
# Giá trị TIMESTAMP tự động chuyển thành thời gian UTC+8
+---------------------+---------------------+
| date_time           | time_stamp          |
+---------------------+---------------------+
| 2020-01-11 09:53:32 | 2020-01-11 17:53:32 |
+---------------------+---------------------+
```

**Mở rộng: Các câu lệnh SQL thường dùng để thiết lập time zone của MySQL**

```sql
# Xem time zone của session hiện tại
SELECT @@session.time_zone;
# Thiết lập time zone của session hiện tại
SET time_zone = 'Europe/Helsinki';
SET time_zone = "+00:00";
# Thiết lập time zone toàn cục của database
SELECT @@global.time_zone;
# Thiết lập time zone toàn cục
SET GLOBAL time_zone = '+8:00';
SET GLOBAL time_zone = 'Europe/Helsinki';
```

### Dung lượng sử dụng

Dưới đây là dung lượng lưu trữ mà các kiểu dữ liệu ngày tháng của MySQL sử dụng (tài liệu chính thức: <https://dev.mysql.com/doc/refman/8.0/en/storage-requirements.html>):

![](https://oss.javaguide.cn/github/javaguide/FhRGUVHFK0ujRPNA75f6CuOXQHTE.jpeg)

Trước MySQL 5.6.4, dung lượng lưu trữ của DateTime và TIMESTAMP là cố định, lần lượt là 8 byte và 4 byte. Tuy nhiên, từ MySQL 5.6.4, dung lượng lưu trữ của chúng sẽ thay đổi theo độ chính xác mili giây; DateTime nằm trong khoảng 5~8 byte, còn TIMESTAMP nằm trong khoảng 4~7 byte.

### Phạm vi biểu diễn

Phạm vi thời gian mà `TIMESTAMP` biểu diễn được nhỏ hơn, chỉ đến năm 2038:

- `DATETIME`: `'1000-01-01 00:00:00.000000'` đến `'9999-12-31 23:59:59.999999'`
- `TIMESTAMP`: `'1970-01-01 00:00:01.000000'` UTC đến `'2038-01-19 03:14:07.999999'` UTC

### Performance

Do `TIMESTAMP` cần chuyển đổi giữa UTC và time zone của session hiện tại khi lưu trữ và truy xuất, quá trình này có thể phát sinh chi phí tính toán bổ sung, đặc biệt khi cần gọi interface tầng dưới của hệ điều hành để lấy hoặc xử lý thông tin time zone. Mặc dù database và hệ điều hành hiện đại đã tối ưu hóa việc này, trong một số trường hợp cực đoan có concurrency cao hoặc cực kỳ nhạy cảm với latency, `DATETIME` không cần chuyển đổi time zone nên logic xử lý tương đối đơn giản, trực tiếp và có thể có ưu thế performance nhỏ.

Để có hành vi dễ dự đoán và có thể giảm chi phí chuyển đổi của `TIMESTAMP`, cách được khuyến nghị là quản lý time zone thống nhất ở tầng ứng dụng, hoặc thiết lập tường minh tham số `time_zone` ở cấp kết nối/session của database, thay vì phụ thuộc vào time zone mặc định của server hoặc hệ điều hành.

## Unix time stamp dạng số có phải lựa chọn tốt hơn không?

Ngoài hai kiểu trên, trong thực tế cũng thường dùng kiểu số nguyên (`INT` hoặc `BIGINT`) để lưu trữ cái gọi là “Unix time stamp” (tức tổng số giây hoặc mili giây tính từ 00:00:00 UTC ngày 1 tháng 1 năm 1970 đến thời điểm mục tiêu).

Cách lưu trữ này có một số ưu điểm của kiểu `TIMESTAMP`, đồng thời hiệu quả khi dùng nó để sắp xếp và so sánh ngày tháng cũng cao hơn, việc truyền giữa các hệ thống cũng rất thuận tiện vì chỉ lưu trữ giá trị số. Nhược điểm cũng rất rõ ràng: khả năng dễ đọc của dữ liệu quá kém, bạn không thể trực quan thấy thời gian cụ thể.

Định nghĩa time stamp như sau:

> Định nghĩa của time stamp là tính từ một thời điểm cơ sở, thời điểm cơ sở này là «1970-1-1 00:00:00 +0:00». Bắt đầu từ thời điểm này, dùng số nguyên để biểu diễn và tính theo giây; cùng với thời gian trôi qua, số nguyên thời gian này không ngừng tăng. Như vậy, chỉ cần một giá trị số là có thể biểu diễn thời gian một cách đầy đủ. Hơn nữa, đây là một giá trị tuyệt đối, nghĩa là dù ở bất kỳ nơi nào trên Trái Đất, time stamp biểu thị thời gian này đều giống nhau, giá trị được tạo ra cũng giống nhau. Nó không có khái niệm time zone, nên trong quá trình truyền thời gian giữa các hệ thống cũng không cần chuyển đổi bổ sung; chỉ khi hiển thị cho người dùng mới chuyển thành string thời gian địa phương.

Thao tác thực tế trong database:

```sql
-- Chuyển string ngày giờ thành Unix time stamp (giây)
mysql> SELECT UNIX_TIMESTAMP('2020-01-11 09:53:32');
+---------------------------------------+
| UNIX_TIMESTAMP('2020-01-11 09:53:32') |
+---------------------------------------+
|                            1578707612 |
+---------------------------------------+
1 row in set (0.00 sec)

-- Chuyển Unix time stamp (giây) thành định dạng ngày giờ
mysql> SELECT FROM_UNIXTIME(1578707612);
+---------------------------+
| FROM_UNIXTIME(1578707612) |
+---------------------------+
| 2020-01-11 09:53:32       |
+---------------------------+
1 row in set (0.01 sec)
```

## PostgreSQL không có DATETIME

Vì có độc giả đề cập đến kiểu thời gian của PostgreSQL (PG), phần này bổ sung thêm một số nội dung. Địa chỉ mô tả các kiểu thời gian trong tài liệu chính thức của PG: <https://www.postgresql.org/docs/current/datatype-datetime.html>.

![Tổng hợp các kiểu thời gian PostgreSQL](https://oss.javaguide.cn/github/javaguide/mysql/pg-datetime-types.png)

Có thể thấy PG không có kiểu mang tên `DATETIME`:

- `TIMESTAMP WITHOUT TIME ZONE` của PG gần với `DATETIME` của MySQL nhất về chức năng. Nó lưu trữ ngày tháng và thời gian nhưng không chứa bất kỳ thông tin time zone nào, mà lưu trữ giá trị literal.
- `TIMESTAMP WITH TIME ZONE` (hoặc `TIMESTAMPTZ`) của PG tương đương với `TIMESTAMP` của MySQL. Khi lưu trữ, nó chuyển giá trị đầu vào thành UTC, và khi truy xuất, nó chuyển đổi rồi hiển thị theo time zone của session hiện tại.

Đối với phần lớn ứng dụng cần ghi lại thời điểm xảy ra chính xác, `TIMESTAMPTZ` là lựa chọn được khuyến nghị và vững chắc nhất trong PostgreSQL, vì nó xử lý tốt nhất sự phức tạp của time zone.

## Tổng kết

Rốt cuộc nên lưu trữ thời gian trong MySQL như thế nào? `DATETIME`? `TIMESTAMP`? Hay time stamp dạng số?

Không có một giải pháp vạn năng. Nhiều developer cho rằng time stamp dạng số thực sự tốt, vừa hiệu quả vừa tương thích với nhiều hệ thống, nhưng cũng có nhiều người cho rằng nó không đủ trực quan.

Tác giả của cuốn sách _High Performance MySQL_ khuyến nghị dùng TIMESTAMP, vì biểu diễn thời gian bằng số không đủ trực quan. Dưới đây là nguyên văn:

<img src="https://oss.javaguide.cn/github/javaguide/%E9%AB%98%E6%80%A7%E8%83%BDmysql-%E4%B8%8D%E6%8E%A8%E8%8D%90%E7%94%A8%E6%95%B0%E5%80%BC%E6%97%B6%E9%97%B4%E6%88%B3.jpg" style="zoom:50%;" />

Mỗi cách đều có ưu thế riêng. Lựa chọn cách phù hợp nhất theo trường hợp thực tế mới là điều quan trọng. Dưới đây là phần so sánh đơn giản giữa ba cách này để bạn lựa chọn đúng kiểu dữ liệu lưu trữ thời gian trong quá trình phát triển thực tế:

| Kiểu               | Dung lượng lưu trữ | Định dạng ngày tháng           | Phạm vi ngày tháng                                           | Có thông tin time zone hay không |
| ------------------ | ------------------ | ------------------------------ | ------------------------------------------------------------ | -------------------------------- |
| DATETIME           | 5~8 byte           | YYYY-MM-DD hh:mm:ss[.fraction] | 1000-01-01 00:00:00[.000000] ～ 9999-12-31 23:59:59[.999999] | Không                            |
| TIMESTAMP          | 4~7 byte           | YYYY-MM-DD hh:mm:ss[.fraction] | 1970-01-01 00:00:01[.000000] ～ 2038-01-19 03:14:07[.999999] | Có                               |
| Time stamp dạng số | 4 byte             | Toàn chữ số như 1578707612     | Thời gian sau 1970-01-01 00:00:01                            | Không                            |

**Tóm tắt đề xuất lựa chọn:**

- Ưu thế cốt lõi của `TIMESTAMP` nằm ở khả năng xử lý time zone tích hợp sẵn. Database chịu trách nhiệm lưu trữ UTC và tự động chuyển đổi dựa trên time zone của session, giúp đơn giản hóa việc phát triển ứng dụng cần xử lý nhiều time zone. Nếu ứng dụng cần xử lý nhiều time zone hoặc muốn database tự động quản lý việc chuyển đổi time zone, `TIMESTAMP` là lựa chọn tự nhiên (lưu ý giới hạn phạm vi thời gian, tức vấn đề năm 2038).
- Nếu trường hợp sử dụng không liên quan đến chuyển đổi time zone, hoặc muốn ứng dụng hoàn toàn kiểm soát logic time zone, đồng thời cần biểu diễn thời gian sau năm 2038, `DATETIME` là lựa chọn chắc chắn hơn.
- Nếu đặc biệt quan tâm đến performance khi so sánh, hoặc cần truyền dữ liệu thời gian thường xuyên giữa các hệ thống và có thể chấp nhận hy sinh khả năng dễ đọc (hoặc luôn chuyển đổi ở tầng ứng dụng), time stamp dạng số là một lựa chọn mạnh.

<!-- @include: @article-footer.snippet.md -->
