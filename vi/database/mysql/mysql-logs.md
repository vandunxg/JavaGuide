---
title: Giải thích chi tiết về ba loại log chính của MySQL (binlog, redo log và undo log)
description: "Phân tích chuyên sâu vai trò và nguyên lý của ba loại log chính trong MySQL: binlog, redo log và undo log; giải thích chi tiết cơ chế two-phase commit bảo đảm tính nhất quán dữ liệu và ứng dụng của log trong crash recovery, master-slave replication."
category: Database
tag:
  - MySQL
head:
  - - meta
    - name: keywords
      content: MySQL log,binlog,redo log,undo log,two-phase commit,crash recovery,master-slave replication,WAL,transaction log
---

> Bài viết này do tài khoản công khai A Tinh (lập trình viên) đóng góp; JavaGuide đã bổ sung và hoàn thiện.

## Lời mở đầu

Log của MySQL chủ yếu gồm một số loại: error log, query log, slow query log, transaction log và binary log. Trong đó, quan trọng nhất là binary log binlog (archive log) cùng transaction log gồm redo log (redo log) và undo log (rollback log).

![](https://oss.javaguide.cn/github/javaguide/01.png)

Bài viết này tập trung vào redo log, binlog (archive log), two-phase commit và undo log (rollback log).

## redo log

redo log là loại log đặc thù của storage engine InnoDB, giúp MySQL có khả năng crash recovery.

Ví dụ, khi MySQL instance bị lỗi hoặc dừng đột ngột, lúc khởi động lại, storage engine InnoDB sẽ dùng redo log để khôi phục dữ liệu, bảo đảm tính bền vững và tính toàn vẹn của dữ liệu.

![](https://oss.javaguide.cn/github/javaguide/02.png)

Trong MySQL, dữ liệu được tổ chức theo page. Khi bạn truy vấn một record, hệ thống sẽ tải một page dữ liệu từ disk; page này được gọi là data page và được đưa vào `Buffer Pool`.

Các lần truy vấn sau sẽ tìm trong `Buffer Pool` trước; nếu không tìm thấy mới tải từ disk, nhờ đó giảm chi phí IO và cải thiện performance.

Khi update dữ liệu trong table cũng vậy: nếu phát hiện dữ liệu cần update đang tồn tại trong `Buffer Pool`, hệ thống sẽ update trực tiếp trong `Buffer Pool`.

Sau đó, thay đổi “đã thực hiện trên một data page nào đó” sẽ được ghi vào `redo log buffer`, rồi flush xuống file redo log.

![](https://oss.javaguide.cn/github/javaguide/03.png)

> Lưu ý lỗi trong hình: ở bước 4, câu “xóa redo log buffe rồi flush vào redo log” phải là buffer thay vì buffe.

Trong trường hợp lý tưởng, redo log sẽ được flush ngay khi transaction commit, nhưng trên thực tế, thời điểm flush phụ thuộc vào strategy.

> Mẹo nhỏ: mỗi record redo gồm “số tablespace + số data page + offset + độ dài dữ liệu thay đổi + dữ liệu thay đổi cụ thể”.

### Thời điểm flush

Trong storage engine InnoDB, **redo log buffer** là một vùng memory dùng để tạm lưu redo log. Để bảo đảm tính bền vững của transaction và tính nhất quán của dữ liệu, InnoDB sẽ flush log trong buffer này xuống file redo log trên disk vào những thời điểm nhất định. Các thời điểm đó có thể được tổng hợp thành sáu trường hợp sau:

1. **Khi commit transaction (cốt lõi nhất)**: khi transaction commit, redo log trong log buffer sẽ được flush xuống disk (có thể điều khiển bằng parameter `innodb_flush_log_at_trx_commit`, sẽ nói ở phần sau).
2. **Khi không đủ dung lượng trong redo log buffer**: đây là cơ chế chủ động quản lý capacity của InnoDB, nhằm tránh việc user thread bị block do buffer đầy.
   - Khi dung lượng đã sử dụng của redo log buffer vượt quá **một nửa (50%)** tổng capacity, background thread sẽ **chủ động** flush phần log này xuống disk để giải phóng dung lượng cho các lần ghi log tiếp theo, đây là một optimization “phòng xa”.
   - Nếu buffer bị **ghi đầy hoàn toàn** do transaction lớn hoặc I/O bận, mọi user thread cố ghi log mới sẽ bị **block**, đồng thời bắt buộc thực hiện một lần flush đồng bộ cho đến khi có dung lượng trống. Tình huống này ảnh hưởng đến performance của database và nên được hạn chế.
3. **Khi trigger Checkpoint**: Checkpoint là cơ chế cốt lõi được InnoDB thiết kế nhằm rút ngắn thời gian crash recovery. Khi Checkpoint được trigger, InnoDB cần flush toàn bộ dirty page trước checkpoint này xuống disk. Theo nguyên tắc **Write-Ahead Logging (WAL)**, trước khi data page được ghi xuống disk, redo log tương ứng phải được ghi xuống disk trước. Vì vậy, thao tác Checkpoint chắc chắn sẽ bảo đảm redo log liên quan cũng đã được flush xuống disk.
4. **Background thread flush định kỳ**: InnoDB có một master thread chạy ở background, khoảng mỗi giây thực hiện một lần các task định kỳ, trong đó có flush log trong redo log buffer xuống disk. Cơ chế này là biện pháp bảo đảm persistence chủ yếu khi `innodb_flush_log_at_trx_commit` được đặt là 0 hoặc 2.
5. **Khi server shutdown bình thường**: trong quá trình MySQL server shutdown bình thường, để bảo đảm dữ liệu của mọi transaction đã commit được lưu đầy đủ, InnoDB sẽ thực hiện một lần flush cuối cùng, xóa toàn bộ log còn lại trong redo log buffer và ghi vào file trên disk.
6. **Khi binlog chuyển file**: khi binlog được bật, trong cấu hình double 1 (`innodb_flush_log_at_trx_commit=1` và `sync_binlog=1`) của MySQL, để bảo đảm trạng thái giữa redo log và binlog nhất quán (dùng cho crash recovery hoặc master-slave replication), khi file binlog đầy hoặc khi chuyển file bằng cách thực hiện thủ công `flush logs`, thao tác flush redo log sẽ được trigger.

Tóm lại, InnoDB sẽ flush redo log trong nhiều tình huống để bảo đảm tính bền vững và tính nhất quán của dữ liệu.

Cần chú ý thiết lập strategy flush đúng cho `innodb_flush_log_at_trx_commit`. Tùy strategy flush được cấu hình cho MySQL, sau khi MySQL dừng đột ngột có thể xảy ra mất một lượng nhỏ dữ liệu.

`innodb_flush_log_at_trx_commit` có 3 giá trị, tương ứng với 3 strategy flush:

- **0**: khi đặt là 0, mỗi lần commit transaction sẽ không thực hiện flush. Cách này có performance cao nhất nhưng cũng kém an toàn nhất, vì nếu MySQL gặp lỗi hoặc dừng đột ngột, các transaction trong 1 giây gần nhất có thể bị mất.
- **1**: khi đặt là 1, mỗi lần commit transaction đều thực hiện flush. Cách này có performance thấp nhất nhưng an toàn nhất, vì chỉ cần transaction commit thành công thì record redo log chắc chắn đã ở trên disk và sẽ không mất dữ liệu.
- **2**: khi đặt là 2, mỗi lần commit transaction chỉ ghi nội dung redo log trong log buffer vào page cache (filesystem cache). Page cache chuyên dùng để cache file, file được cache ở đây chính là file redo log. Cách này có performance và độ an toàn nằm giữa hai cách trên.

Strategy flush `innodb_flush_log_at_trx_commit` có giá trị mặc định là 1. Chỉ khi đặt là 1 mới không mất dữ liệu. Để bảo đảm persistence của transaction, chúng ta bắt buộc phải đặt nó là 1.

Ngoài ra, storage engine InnoDB có một background thread. Cứ mỗi `1` giây, thread này ghi nội dung trong `redo log buffer` vào filesystem cache (`page cache`), sau đó gọi `fsync` để flush.

![](https://oss.javaguide.cn/github/javaguide/04.png)

Nói cách khác, record redo log của một transaction chưa commit cũng có thể được flush.

**Tại sao?**

Vì trong quá trình transaction thực thi, record redo log được ghi vào `redo log buffer`, sau đó các record redo log này được background thread flush.

![](https://oss.javaguide.cn/github/javaguide/05.png)

Ngoài thao tác polling mỗi `1` giây của background thread, còn một trường hợp khác: khi dung lượng được sử dụng của `redo log buffer` sắp đạt một nửa `innodb_log_buffer_size`, background thread sẽ chủ động flush.

Dưới đây là flowchart của các strategy flush khác nhau.

#### innodb_flush_log_at_trx_commit=0

![](https://oss.javaguide.cn/github/javaguide/06.png)

Khi là `0`, nếu MySQL gặp lỗi hoặc dừng đột ngột thì có thể mất dữ liệu trong `1` giây.

#### innodb_flush_log_at_trx_commit=1

![](https://oss.javaguide.cn/github/javaguide/07.png)

Khi là `1`, chỉ cần transaction commit thành công thì record redo log chắc chắn ở trên disk và sẽ không mất dữ liệu.

Nếu MySQL gặp lỗi hoặc dừng đột ngột trong thời gian transaction đang thực thi, phần log này sẽ mất, nhưng transaction chưa commit nên log mất cũng không gây tổn thất.

#### innodb_flush_log_at_trx_commit=2

![](https://oss.javaguide.cn/github/javaguide/09.png)

Khi là `2`, chỉ cần transaction commit thành công thì nội dung trong `redo log buffer` chỉ được ghi vào filesystem cache (`page cache`).

Nếu chỉ MySQL gặp lỗi thì sẽ không mất dữ liệu, nhưng nếu máy dừng đột ngột thì có thể mất dữ liệu trong `1` giây.

### Log file group

Redo log được lưu trên disk dưới dạng nhiều file tạo thành một **log file group**; mỗi file `redo` có cùng kích thước.

Ví dụ, có thể cấu hình một group gồm `4` file, mỗi file có kích thước `1GB`, khi đó cả log file group có thể ghi `4G` dữ liệu.

Cơ chế này sử dụng dạng circular array: bắt đầu ghi từ đầu, ghi đến cuối rồi quay lại đầu và tiếp tục ghi vòng, như hình dưới đây.

![](https://oss.javaguide.cn/github/javaguide/10.png)

Trong **log file group** này còn có hai thuộc tính quan trọng là `write pos` và `checkpoint`.

- **write pos** là vị trí ghi hiện tại, vừa ghi vừa tiến về phía sau.
- **checkpoint** là vị trí hiện tại cần xóa, cũng dịch dần về phía sau.

Mỗi lần flush record redo log vào **log file group**, vị trí `write pos` sẽ được cập nhật bằng cách dịch về phía sau.

Mỗi lần MySQL load **log file group** để khôi phục dữ liệu, các record redo log đã được load sẽ bị xóa, đồng thời `checkpoint` được cập nhật bằng cách dịch về phía sau.

Phần còn trống giữa `write pos` và `checkpoint` có thể dùng để ghi record redo log mới.

![](https://oss.javaguide.cn/github/javaguide/11.png)

Nếu `write pos` đuổi kịp `checkpoint`, nghĩa là **log file group** đã đầy. Khi đó không thể ghi record redo log mới nữa, MySQL phải dừng lại, xóa một số record và đẩy `checkpoint` về phía trước.

![](https://oss.javaguide.cn/github/javaguide/12.png)

Lưu ý, từ MySQL 8.0.30, log file group đã có một số thay đổi:

> The innodb_redo_log_capacity variable supersedes the innodb_log_files_in_group and innodb_log_file_size variables, which are deprecated. When the innodb_redo_log_capacity setting is defined, the innodb_log_files_in_group and innodb_log_file_size settings are ignored; otherwise, these settings are used to compute the innodb_redo_log_capacity setting (innodb_log_files_in_group \* innodb_log_file_size = innodb_redo_log_capacity). If none of those variables are set, redo log capacity is set to the innodb_redo_log_capacity default value, which is 104857600 bytes (100MB). The maximum redo log capacity is 128GB.

> Redo log files reside in the #innodb_redo directory in the data directory unless a different directory was specified by the innodb_log_group_home_dir variable. If innodb_log_group_home_dir was defined, the redo log files reside in the #innodb_redo directory in that directory. There are two types of redo log files, ordinary and spare. Ordinary redo log files are those being used. Spare redo log files are those waiting to be used. InnoDB tries to maintain 32 redo log files in total, with each file equal in size to 1/32 \* innodb_redo_log_capacity; however, file sizes may differ for a time after modifying the innodb_redo_log_capacity setting.

Điều này có nghĩa là trước MySQL 8.0.30, có thể dùng `innodb_log_files_in_group` và `innodb_log_file_size` để cấu hình số lượng và kích thước file của log file group. Nhưng từ MySQL 8.0.30 trở đi, hai variable này đã bị deprecated; ngay cả khi được chỉ định, chúng cũng chỉ được dùng để tính giá trị của `innodb_redo_log_capacity`. Số lượng file của log file group được cố định là 32, còn kích thước file là `innodb_redo_log_capacity / 32`.

Để kiểm chứng thay đổi này, có thể thực hiện như sau.

Trước tiên, tạo một file cấu hình với các giá trị của `innodb_log_files_in_group` và `innodb_log_file_size`:

```properties
[mysqld]
innodb_log_file_size = 10485760
innodb_log_files_in_group = 64
```

Khởi động một container MySQL 8.0.32 bằng Docker:

```bash
docker run -d -p 3312:3309 -e MYSQL_ROOT_PASSWORD=your-password -v /path/to/your/conf:/etc/mysql/conf.d --name
MySQL830 mysql:8.0.32
```

Bây giờ hãy xem log khởi động:

```plain
2023-08-03T02:05:11.720357Z 0 [Warning] [MY-013907] [InnoDB] Deprecated configuration parameters innodb_log_file_size and/or innodb_log_files_in_group have been used to compute innodb_redo_log_capacity=671088640. Please use innodb_redo_log_capacity instead.
```

Điều này cũng cho thấy hai biến `innodb_log_files_in_group` và `innodb_log_file_size` được dùng để tính `innodb_redo_log_capacity`, đồng thời đã deprecated.

Tiếp theo, hãy xem số lượng file của log file group là bao nhiêu:

![](./images/redo-log.png)

Có thể thấy vừa đúng 32 file, và kích thước mỗi file là `671088640 / 32 = 20971520`.

Vì vậy, khi sử dụng MySQL 8.0.30 trở đi, nên dùng variable `innodb_redo_log_capacity` để cấu hình log file group.

### Tóm tắt redo log

Trên đây đã trình bày vai trò, thời điểm flush và hình thức lưu trữ của redo log.

Bây giờ hãy suy nghĩ về một câu hỏi: **chỉ cần flush trực tiếp data page sau khi thay đổi là được, vậy redo log còn có tác dụng gì?**

Chẳng phải cả hai đều flush sao? Khác biệt nằm ở đâu?

```java
1 Byte = 8bit
1 KB = 1024 Byte
1 MB = 1024 KB
1 GB = 1024 MB
1 TB = 1024 GB
```

Trên thực tế, kích thước data page là `16KB`, flush khá tốn thời gian, trong khi có thể chỉ thay đổi vài `Byte` dữ liệu trong data page. Có cần flush toàn bộ data page không?

Hơn nữa, flush data page là random write, vì vị trí tương ứng với một data page có thể nằm ở vị trí ngẫu nhiên trong file trên disk, nên performance rất kém.

Nếu ghi redo log, một record có thể chỉ chiếm vài chục `Byte`, chỉ bao gồm số tablespace, số data page, offset trong file trên disk
và giá trị được cập nhật, hơn nữa còn là sequential write nên tốc độ flush rất nhanh.

Vì vậy, ghi nội dung thay đổi dưới dạng redo log có performance vượt xa cách ghi data page, đồng thời giúp database có khả năng xử lý concurrency mạnh hơn.

> Thực ra data page trong memory cũng được flush vào những thời điểm nhất định, được gọi là page merge. Phần này sẽ được giải thích chi tiết khi nói về `Buffer Pool`.

## binlog

redo log là physical log, nội dung ghi lại là “đã thực hiện thay đổi gì trên một data page nào đó”, thuộc storage engine InnoDB.

Còn binlog là logical log, nội dung ghi lại logic nguyên bản của statement, tương tự “tăng field c của record có ID=2 lên 1”, thuộc layer `MySQL Server`.

Bất kể dùng storage engine nào, chỉ cần xảy ra update dữ liệu trong table thì đều tạo ra binlog.

Vậy binlog rốt cuộc dùng để làm gì?

Có thể nói **data backup, primary-standby, master-master và master-slave** của database MySQL đều không thể thiếu binlog; cần dựa vào binlog để đồng bộ dữ liệu và bảo đảm tính nhất quán của dữ liệu.

![](https://oss.javaguide.cn/github/javaguide/01-20220305234724956.png)

binlog ghi lại mọi thao tác logic liên quan đến update dữ liệu và được ghi theo kiểu sequential write.

### Format record

binlog có ba format, có thể chỉ định bằng parameter `binlog_format`.

- **statement**
- **row**
- **mixed**

Khi chỉ định `statement`, nội dung ghi lại là câu lệnh `SQL` nguyên bản. Ví dụ, thực thi `update T set update_time=now() where id=1` thì nội dung ghi lại như sau.

![](https://oss.javaguide.cn/github/javaguide/02-20220305234738688.png)

Khi đồng bộ dữ liệu, hệ thống sẽ thực thi câu lệnh `SQL` được ghi lại. Nhưng có một vấn đề: `update_time=now()` sẽ lấy thời gian hiện tại của hệ thống; thực thi trực tiếp sẽ khiến dữ liệu không nhất quán với database gốc.

Để giải quyết vấn đề này, cần chỉ định là `row`. Nội dung ghi lại không còn chỉ là câu lệnh `SQL` đơn giản mà còn chứa dữ liệu cụ thể của thao tác, như dưới đây.

![](https://oss.javaguide.cn/github/javaguide/03-20220305234742460.png)

Nội dung được ghi ở format `row` không thể hiện thông tin chi tiết, cần dùng tool `mysqlbinlog` để parse.

`update_time=now()` trở thành thời gian cụ thể `update_time=1627112756247`; `@1`, `@2`, `@3` sau điều kiện lần lượt là giá trị nguyên bản của field thứ 1 đến thứ 3 trong record đó (**giả sử table này chỉ có 3 field**).

Như vậy có thể bảo đảm tính nhất quán khi đồng bộ dữ liệu. Thông thường đều chỉ định là `row`, nhờ đó việc recovery và đồng bộ database đáng tin cậy hơn.

Tuy nhiên, format này cần dung lượng lớn hơn để ghi dữ liệu; khi recovery và đồng bộ cũng tiêu tốn nhiều tài nguyên IO hơn, ảnh hưởng đến tốc độ thực thi.

Vì vậy mới có phương án trung gian là chỉ định `mixed`, trong đó nội dung ghi lại là sự kết hợp của hai format trước.

MySQL sẽ xác định câu lệnh `SQL` này có thể gây ra dữ liệu không nhất quán hay không. Nếu có, nó dùng format `row`; nếu không, nó dùng format `statement`.

### Cơ chế ghi

Thời điểm ghi binlog cũng rất đơn giản: trong quá trình transaction thực thi, trước tiên binlog được ghi vào `binlog cache`; khi transaction commit, `binlog cache` mới được ghi vào file binlog.

Vì binlog của một transaction không thể bị tách ra, bất kể transaction lớn đến đâu cũng phải bảo đảm được ghi một lần, nên system sẽ cấp cho mỗi thread một vùng memory làm `binlog cache`.

Có thể dùng parameter `binlog_cache_size` để điều khiển kích thước `binlog cache` của một thread. Nếu dữ liệu vượt quá parameter này, dữ liệu sẽ được tạm lưu trên disk (`Swap`).

Quy trình flush log binlog như sau:

![](https://oss.javaguide.cn/github/javaguide/04-20220305234747840.png)

- **`write` trong hình trên nghĩa là ghi log vào page cache của filesystem, chưa persist dữ liệu xuống disk nên tốc độ khá nhanh.**
- **`fsync` trong hình trên mới là thao tác persist dữ liệu xuống disk.**

Thời điểm `write` và `fsync` có thể được điều khiển bằng parameter `sync_binlog`, mặc định là `1`.

Khi là `0`, mỗi lần commit transaction chỉ thực hiện `write`, còn thời điểm thực hiện `fsync` do system tự quyết định.

![](https://oss.javaguide.cn/github/javaguide/05-20220305234754405.png)

Dù performance được cải thiện, nhưng nếu máy dừng đột ngột thì binlog trong `page cache` sẽ bị mất.

Để an toàn, có thể đặt là `1`, nghĩa là mỗi lần commit transaction đều thực hiện `fsync`, giống với **quy trình flush redo log**.

Cuối cùng còn một cách trung gian khác: có thể đặt là `N(N>1)`, nghĩa là mỗi lần commit transaction đều `write`, nhưng chỉ `fsync` sau khi tích lũy đủ `N` transaction.

![](https://oss.javaguide.cn/github/javaguide/06-20220305234801592.png)

Trong tình huống xuất hiện IO bottleneck, đặt `sync_binlog` thành một giá trị tương đối lớn có thể cải thiện performance.

Tương tự, nếu máy dừng đột ngột thì binlog của `N` transaction gần nhất sẽ bị mất.

## Two-phase commit

redo log (redo log) giúp storage engine InnoDB có khả năng crash recovery.

binlog (archive log) bảo đảm tính nhất quán dữ liệu trong kiến trúc cluster MySQL.

Mặc dù đều là cơ chế bảo đảm persistence, nhưng trọng tâm của chúng khác nhau.

Trong quá trình thực thi statement update, hệ thống sẽ ghi hai loại log là redo log và binlog. Lấy transaction làm đơn vị cơ bản, redo log có thể được ghi liên tục trong quá trình transaction thực thi, còn binlog chỉ được ghi khi commit transaction, vì vậy thời điểm ghi redo log và binlog khác nhau.

![](https://oss.javaguide.cn/github/javaguide/01-20220305234816065.png)

Quay lại vấn đề chính, nếu logic giữa hai log là redo log và binlog không nhất quán thì sẽ xảy ra vấn đề gì?

Lấy statement `update` làm ví dụ. Giả sử record có `id=2` có giá trị field `c` là `0`, update giá trị field `c` thành `1`, câu lệnh `SQL` là `update T set c=1 where id=2`.

Giả sử trong quá trình thực thi, sau khi ghi xong redo log thì xảy ra lỗi khi ghi binlog. Tình huống sẽ thế nào?

![](https://oss.javaguide.cn/github/javaguide/02-20220305234828662.png)

Vì xảy ra lỗi trước khi ghi xong binlog, nên binlog không có record thay đổi tương ứng. Do đó, sau này khi dùng binlog để recovery dữ liệu, lần update này sẽ bị thiếu; giá trị `c` của record được recovery là `0`, trong khi database gốc được khôi phục bằng redo log lại có giá trị `c` là `1`, cuối cùng dữ liệu không nhất quán.

![](https://oss.javaguide.cn/github/javaguide/03-20220305235104445.png)

Để giải quyết vấn đề logic không nhất quán giữa hai log, storage engine InnoDB sử dụng phương án **two-phase commit**.

Nguyên lý rất đơn giản: chia việc ghi redo log thành hai bước là `prepare` và `commit`, đây chính là **two-phase commit**.

![](https://oss.javaguide.cn/github/javaguide/04-20220305234956774.png)

Sau khi sử dụng **two-phase commit**, việc xảy ra lỗi khi ghi binlog cũng không ảnh hưởng, vì khi MySQL recovery dữ liệu dựa trên redo log, nếu phát hiện redo log vẫn ở phase `prepare` và không có log binlog tương ứng thì sẽ rollback transaction đó.

![](https://oss.javaguide.cn/github/javaguide/05-20220305234937243.png)

Xét một tình huống khác: nếu xảy ra lỗi khi redo log chuyển sang phase `commit` thì transaction có rollback không?

![](https://oss.javaguide.cn/github/javaguide/06-20220305234907651.png)

Transaction sẽ không rollback. MySQL sẽ thực hiện logic được khoanh trong hình trên. Mặc dù redo log vẫn ở phase `prepare`, nhưng có thể dùng `id` của transaction để tìm log binlog tương ứng, nên MySQL cho rằng transaction đã hoàn chỉnh và sẽ commit để recovery dữ liệu.

## undo log

> Phần nội dung này do JavaGuide bổ sung:

Mọi thay đổi dữ liệu trong transaction đều được ghi vào undo log. Nếu xảy ra lỗi trong quá trình thực thi transaction hoặc cần thực hiện rollback, MySQL có thể dùng undo log để khôi phục dữ liệu về state trước khi transaction bắt đầu.

undo log là logical log, ghi lại câu lệnh SQL. Ví dụ, nếu transaction thực thi statement DELETE thì undo log sẽ ghi lại một statement INSERT tương ứng. Đồng thời, thông tin của undo log cũng được ghi vào redo log, vì undo log cũng cần được bảo vệ persistence. Ngoài ra, bản thân undo log sẽ được xóa và dọn dẹp. Ví dụ, với thao tác INSERT, sau khi transaction commit có thể xóa undo log; với thao tác UPDATE/DELETE, undo log không bị xóa ngay khi transaction commit mà được thêm vào history list và do background thread purge dọn dẹp.

undo log được ghi theo dạng segment. Mỗi thao tác undo khi được ghi sẽ chiếm một **undo log segment** nằm trong **rollback segment**. Khi transaction bắt đầu, hệ thống cần cấp phát một rollback segment cho nó. Mỗi rollback segment có 1024 undo log segment, giúp quản lý nhu cầu rollback của nhiều transaction concurrency.

Thông thường, **rollback segment header** (thường nằm ở page đầu tiên của rollback segment) phụ trách quản lý rollback segment. Rollback segment header là một phần của rollback segment, thường nằm ở page đầu tiên của rollback segment. **history list** là một phần của rollback segment header, chủ yếu dùng để ghi lại undo log của mọi transaction đã commit nhưng chưa được dọn dẹp (purge). List này giúp purge thread tìm và dọn dẹp các record undo log không còn cần thiết.

Ngoài ra, implementation của **MVCC** phụ thuộc vào **hidden field, Read View và undo log**. Trong implementation nội bộ, InnoDB dựa vào `DB_TRX_ID` của row dữ liệu và `Read View` để xác định visibility của dữ liệu. Nếu không visible, InnoDB tìm version lịch sử trong undo log thông qua `DB_ROLL_PTR` của row dữ liệu. Version dữ liệu mà mỗi transaction đọc được có thể khác nhau. Trong cùng một transaction, user chỉ có thể nhìn thấy các thay đổi đã commit trước khi transaction đó tạo `Read View` và các thay đổi do chính transaction đó thực hiện.

## Tổng kết

> Phần nội dung này do JavaGuide bổ sung:

MySQL InnoDB engine sử dụng **redo log** để bảo đảm **persistence** của transaction, sử dụng **undo log** để bảo đảm **atomicity** của transaction.

**Data backup, primary-standby, master-master và master-slave** của database MySQL đều không thể thiếu binlog; cần dựa vào binlog để đồng bộ dữ liệu và bảo đảm tính nhất quán của dữ liệu.

## Tham khảo

- 《45 bài giảng thực chiến về MySQL》
- 《Hướng dẫn trở thành chuyên gia tối ưu MySQL thực chiến từ con số 0》
- 《MySQL vận hành như thế nào: Tìm hiểu từ tầng bên trong》
- 《Công nghệ MySQL: Storage engine InnoDB》

<!-- @include: @article-footer.snippet.md -->
