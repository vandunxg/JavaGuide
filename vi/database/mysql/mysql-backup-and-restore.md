---
title: "Giải thích chi tiết về backup và restore MySQL: mysqldump, XtraBackup, binlog và PITR"
description: "Giải thích chi tiết về backup và restore MySQL, giới thiệu mysqldump, MySQL Shell, Percona XtraBackup, binlog, PITR, RTO/RPO, diễn tập restore và các hiểu lầm thường gặp về backup."
category: Database
tag:
  - MySQL
  - Backup và Restore
head:
  - - meta
    - name: keywords
      content: MySQL backup,MySQL restore,mysqldump,mysqlbinlog,MySQL Shell,Percona XtraBackup,binlog,PITR,full backup,incremental backup,logical backup,physical backup,RTO,RPO
---

Sự cố database đáng sợ nhất trong production nhiều khi không phải là process MySQL bị crash.

Process bị crash còn có thể restart, primary database bị crash còn có thể chuyển sang replica. Điều thực sự rắc rối là dữ liệu bị xóa, bị script lỗi sửa nhầm, disk bị hỏng hoặc phát hiện thiếu một số table trong lúc migration. Khi đó, master-slave replication, redo log và undo log đều không đủ dùng; thứ thường có thể cứu vãn tình huống là backup và restore.

Bài viết này chỉ nói về backup và restore MySQL, không đi sâu vào PostgreSQL, Redis hay sản phẩm backup của các nhà cung cấp cloud. Các command chủ yếu được đối chiếu theo MySQL 8.4 LTS, đồng thời tham khảo tài liệu MySQL 9.7 hiện hành tính đến 2026-06-25; tên parameter, yêu cầu permission và khả năng tương thích của tool có thể thay đổi giữa các version, vì vậy trước khi triển khai production nhất định phải kiểm tra theo `mysqldump --help`, `mysqlbinlog --help` và tài liệu của tool trong chính môi trường của bạn.

## Backup thực sự giải quyết vấn đề gì?

Trước tiên, hãy tách vài khái niệm dễ bị trộn lẫn:

- **Crash recovery**: Sau khi MySQL crash bất thường, InnoDB dựa vào redo log, undo log và các cơ chế khác để khôi phục database về trạng thái nhất quán. Đây là vấn đề nhất quán của storage engine sau khi process crash hoặc máy restart.
- **Master-slave replication / chuyển đổi sang hệ thống high availability**: Khi primary database không khả dụng, chuyển traffic sang replica hoặc primary database mới. Đây là vấn đề service availability, nhưng các write sai thường cũng được đồng bộ sang replica.
- **Backup và restore**: Khôi phục từ một bản sao dữ liệu tại thời điểm trước đó, sau đó replay binlog đến một thời điểm cụ thể. Đây là cách xử lý việc mất dữ liệu, xóa hoặc sửa nhầm, disk hỏng, migration giữa các môi trường và lưu trữ phục vụ audit.

Master-slave replication không thể thay thế backup.

Nếu một lệnh `DROP TABLE` thực thi thành công trên primary database, nhiều khả năng nó cũng được đồng bộ sang replica. Replication càng nhanh thì lỗi lan truyền càng nhanh. Giá trị của backup nằm ở việc giữ lại một trạng thái độc lập trong lịch sử, cho bạn cơ hội quay về trước khi sự cố xảy ra.

## RTO và RPO quyết định chiến lược backup

Không nên chỉ hỏi chiến lược backup là “mỗi ngày backup mấy lần”. Hai câu hỏi thực tế hơn là:

- **RPO (Recovery Point Objective)**: Có thể chấp nhận mất dữ liệu tối đa trong bao lâu?
- **RTO (Recovery Time Objective)**: Có thể chấp nhận service được khôi phục trong tối đa bao lâu?

Nếu business có thể chấp nhận mất dữ liệu 1 ngày, full backup mỗi ngày một lần có lẽ là đủ. Nếu dữ liệu order, payment, inventory chỉ được phép mất tối đa vài phút, chỉ full backup là chưa đủ; cần giữ lại binlog để incremental restore. Nếu database có vài trăm GB, việc restore file SQL có thể chạy rất lâu, trong khi RTO yêu cầu khôi phục trong 30 phút, thì cần nghiêm túc cân nhắc physical backup, warm-up replica, diễn tập restore và quy trình chuyển đổi.

Một tổ hợp thường gặp là:

- Thực hiện full backup mỗi ngày hoặc mỗi tuần một lần.
- Bật binlog và giữ log đủ lâu theo yêu cầu RPO.
- Không đặt file backup và binlog trên cùng một disk hoặc trong cùng một failure domain.
- Định kỳ restore backup lên một máy mới và ghi lại thời gian thực tế.

Giữ binlog 30 ngày không có nghĩa RPO chắc chắn là vài giây. Có thể restore đến đâu thực sự phụ thuộc vào binlog cuối cùng vừa đầy đủ, vừa đọc được, vừa được copy sang failure domain độc lập. Trong production còn phải xem `sync_binlog`, `innodb_flush_log_at_trx_commit`, độ trễ archive binlog, process archive có từng bị gián đoạn hay không và chuỗi file binlog có liên tục hay không. MySQL 8.4 mặc định `sync_binlog=1`, `innodb_flush_log_at_trx_commit=1`; hai giá trị này thiên về an toàn hơn. Nếu giảm mức đảm bảo durability vì performance, phải tính khả năng mất các transaction gần nhất vào RPO.

Vì vậy, hệ thống yêu cầu RPO ở mức phút, thậm chí giây, không chỉ cần monitor “trên disk còn binlog của bao nhiêu ngày”, mà còn phải monitor “binlog mới nhất đã archive an toàn cách hiện tại bao lâu”.

Có một giới hạn rất thực tế: tốc độ restore liên quan đến data volume, performance của disk, số lượng index, bandwidth mạng và phương thức import. Khi chưa có dữ liệu diễn tập, RTO chỉ là mong muốn chứ chưa thể là cam kết.

## Có những phương thức backup nào?

Backup MySQL thường có ba nhóm phân loại, mỗi nhóm trả lời một câu hỏi khác nhau.

Theo việc database có cung cấp service trong khi backup hay không:

| Loại        | Mô tả                                                                   | Trường hợp sử dụng                                                            |
| ----------- | ----------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| Cold backup | Dừng MySQL rồi copy các data file                                       | Hệ thống nhỏ, có maintenance window đủ dài                                    |
| Warm backup | Backup khi MySQL đang chạy nhưng có thể lock hoặc ảnh hưởng đến write   | Yêu cầu availability ở mức vừa phải, chấp nhận ảnh hưởng trong thời gian ngắn |
| Hot backup  | Backup khi MySQL đang chạy, cố gắng không block read/write của business | Production database, database lớn, maintenance window rất ngắn                |

Theo nội dung của file backup:

| Loại            | Mô tả                                        | Tool tiêu biểu                              |
| --------------- | -------------------------------------------- | ------------------------------------------- |
| Logical backup  | Export nội dung logic như SQL, CSV           | `mysqldump`, MySQL Shell dump utilities     |
| Physical backup | Copy các file vật lý như data file, log file | Percona XtraBackup, MySQL Enterprise Backup |

Theo phạm vi backup:

| Loại                | Mô tả                                                   |
| ------------------- | ------------------------------------------------------- |
| Full backup         | Backup toàn bộ dữ liệu tại một thời điểm                |
| Incremental backup  | Backup dữ liệu hoặc log đã thay đổi sau backup trước đó |
| Differential backup | Backup dữ liệu đã thay đổi sau lần full backup trước đó |

Trong phỏng vấn thường hỏi các cách phân loại này, nhưng trong production điều thực sự cần đạt được là: file backup có restore được dữ liệu mà business cần hay không. Phân loại chỉ là ngôn ngữ để chọn tool; restore thành công mới là kết quả.

## Dùng mysqldump để logical backup

`mysqldump` là logical backup tool đi kèm MySQL. Nó export một loạt câu lệnh SQL, có thể dùng các câu lệnh này để dựng lại cấu trúc database/table và dữ liệu của table. Ưu điểm của nó là đơn giản, phổ biến, dễ xem và cũng phù hợp với migration giữa các môi trường. Nhược điểm cũng rất rõ: khi data volume lớn, backup chậm và restore còn chậm hơn, vì quá trình restore phải thực thi lại SQL, ghi dữ liệu và tạo index.

Tài liệu chính thức của MySQL cũng nhắc rõ rằng `mysqldump` không phải là giải pháp nhanh cho backup và restore quy mô lớn; khi data volume tăng, physical backup thường phù hợp hơn.

Có thể viết command backup toàn bộ database InnoDB theo hướng production như sau:

```bash
mysqldump \
  --host=127.0.0.1 \
  --user=backup_user \
  --password \
  --all-databases \
  --single-transaction \
  --routines \
  --events \
  --triggers \
  --source-data=2 \
  > mysql-full-backup.sql
```

Một vài parameter cần được giải thích riêng:

- `--single-transaction`: Mở một transaction đọc nhất quán trước khi bắt đầu backup, phù hợp với table InnoDB. Thông thường nó không cần lock table trong suốt thời gian export, nhưng command trong bài còn dùng `--source-data=2`, nên `mysqldump` sẽ tạm thời lấy global read lock ở giai đoạn khởi động để đồng bộ consistency snapshot với binlog position. Vì vậy, cách nói chính xác hơn là “không block write của business trong thời gian dài”, chứ không phải “hoàn toàn không lock”. Nếu có MyISAM, MEMORY hoặc table non-transaction khác, các table này vẫn có thể thay đổi trong lúc backup.
- `--routines` và `--events`: Export cả stored procedure, function và event. Tài liệu MySQL 8.4 nêu rõ các definition liên quan nằm trong data dictionary; `--all-databases` không có nghĩa là tự động kèm các object này.
- `--triggers`: Trigger mặc định sẽ được export; ghi rõ parameter này giúp mục đích của backup script dễ hiểu hơn.
- `--source-data=2`: Ghi tên và position của file binlog hiện tại vào file dump, đồng thời giữ lại dưới dạng SQL comment. `--master-data` thường gặp trong tài liệu cũ hiện là deprecated alias của `--source-data` trong tài liệu mới.

Nếu chỉ backup một database:

```bash
mysqldump \
  --host=127.0.0.1 \
  --user=backup_user \
  --password \
  --single-transaction \
  --routines \
  --events \
  --triggers \
  --source-data=2 \
  order_db \
  > order_db.sql
```

Cách viết này chủ yếu export các object trong `order_db`, không tự động ghi `CREATE DATABASE order_db` và `USE order_db` thành một script tạo database hoàn chỉnh. Khi restore, hoặc phải tạo database trước, hoặc lúc backup dùng `--databases order_db`:

```bash
mysqldump \
  --host=127.0.0.1 \
  --user=backup_user \
  --password \
  --single-transaction \
  --routines \
  --events \
  --triggers \
  --source-data=2 \
  --databases order_db \
  > order_db.sql
```

`--databases` sẽ thêm `CREATE DATABASE` và `USE` vào dump. Nếu chỉ muốn import dữ liệu vào một database cùng tên đã tồn tại, vẫn có thể dùng cách viết trước, nhưng command restore phải chỉ rõ database đích.

Nếu file backup rất lớn, có thể nén trực tiếp:

```bash
mysqldump \
  --host=127.0.0.1 \
  --user=backup_user \
  --password \
  --single-transaction \
  --routines \
  --events \
  --triggers \
  order_db | gzip > order_db.sql.gz
```

Khi restore, thực thi:

```bash
mysql --host=127.0.0.1 --user=root --password < mysql-full-backup.sql
```

File đã nén có thể restore như sau:

```bash
mysql --host=127.0.0.1 --user=root --password \
  -e "CREATE DATABASE IF NOT EXISTS order_db"

gunzip -c order_db.sql.gz | mysql --host=127.0.0.1 --user=root --password order_db
```

`mysqldump` có một số lỗi thường gặp:

- Permission của user backup không đủ, khiến view, trigger, stored procedure và event không được export đầy đủ.
- Khi dùng `--single-transaction`, nếu thực thi `ALTER TABLE`, `DROP TABLE`, `TRUNCATE TABLE` hoặc DDL tương tự trong lúc backup, backup có thể fail hoặc nội dung không đúng như mong đợi.
- Không ghi lại binlog position, khiến về sau không thể tiếp tục point-in-time restore từ full backup.
- Khi restore backup vào môi trường đã bật GTID, không nên phụ thuộc vào `--set-gtid-purged=AUTO` mặc định. Tài liệu chính thức nêu rằng khi source bật GTID, theo mặc định dump có thể ghi `SET @@GLOBAL.gtid_purged` và `SET @@SESSION.sql_log_bin=0`; dump của một số database cũng có thể chứa GTID của các transaction thuộc database khác trong `gtid_executed` của source instance. Khi import vào test database, temporary database hoặc target instance đã có lịch sử GTID, thường cần đánh giá rõ có dùng `--set-gtid-purged=OFF` hay không; nếu tạo replication node mới thì có thể cần giữ lại GTID. Điều quan trọng là phải lựa chọn rõ ràng, không sao chép nguyên giá trị mặc định.
- `--source-data=2` ghi lại binlog position ở cấp instance, không phải incremental log “chỉ thuộc về database này”. PITR cho một database đơn lẻ phức tạp hơn PITR cho toàn instance, đặc biệt phải cân nhắc cross-database transaction, stored procedure, trigger và `binlog_format`; không thể đơn giản replay nguyên binlog của toàn instance vào database đích.

Với database nhỏ, môi trường test, migration giữa các version hoặc export một lượng dữ liệu nhỏ, `mysqldump` rất tiện. Nếu production database hàng trăm GB vẫn chỉ dựa vào nó để restore, thời gian restore thường sẽ khiến mọi người khó chịu.

## MySQL Shell Dump Utilities: logical backup song song

Nếu muốn giữ khả năng migration của logical backup nhưng thấy dump và load bằng một thread của `mysqldump` quá chậm, có thể xem xét MySQL Shell Dump Utilities.

MySQL Shell cung cấp một số function `util`:

```javascript
util.dumpInstance("/backup/mysql/instance", { threads: 8 });
util.dumpSchemas(["order_db"], "/backup/mysql/order_db", { threads: 8 });
util.dumpTables("order_db", ["orders", "order_item"], "/backup/mysql/orders", {
  threads: 8,
});
util.loadDump("/backup/mysql/order_db", { threads: 8 });
```

Nó vẫn được định vị là logical backup, nhưng hỗ trợ dump/load song song, compression, thông tin tiến độ, checksum và output đến object storage. Trong tài liệu chính thức, giá trị mặc định của `threads` là 4; có thể tăng tùy theo tải của instance, network và khả năng ghi của database đích.

Tuy nhiên, không nên hiểu nhầm nó là sản phẩm thay thế physical backup. Nó vẫn export logical object và data file; khi restore cũng phải dựng lại table, index và object. Nếu database lớn và RTO rất gấp, physical backup thường vẫn chắc chắn hơn.

## Dùng binlog để incremental restore

Full backup chỉ có thể restore đến thời điểm backup. Muốn restore đến một thời điểm muộn hơn, cần dựa vào binlog.

binlog ghi lại các event làm thay đổi dữ liệu, đồng thời là nền tảng của master-slave replication và point-in-time restore. PITR (Point-in-Time Recovery) thường bắt đầu bằng việc restore một full backup, sau đó replay log từ binlog position tương ứng với backup cho đến thời gian hoặc position đích.

Một ví dụ đơn giản:

1. Lúc 02:00 sáng thực hiện full backup, file dump ghi `binlog.000120` và position `154`.
2. Lúc 10:21 sáng có người xóa nhầm một nhóm dữ liệu của table `order_item`.
3. Khi restore, trước tiên import full backup lúc 02:00.
4. Sau đó dùng `mysqlbinlog` replay `binlog.000120` và các log tiếp theo, dừng lại trước câu lệnh xóa nhầm.

Có thể restore theo thời gian như sau:

```bash
mysqlbinlog \
  --start-position=154 \
  --stop-datetime="2026-06-25 10:20:59" \
  binlog.000120 binlog.000121 \
  | mysql --binary-mode --host=127.0.0.1 --user=root --password
```

Dừng theo thời gian phù hợp để nhanh chóng thu hẹp phạm vi, nhưng không thể coi đó là thời gian nghiệp vụ tuyệt đối chính xác. `mysqlbinlog` sẽ diễn giải `--stop-datetime` theo timezone local của máy thực thi command và dừng khi gặp event đầu tiên có timestamp lớn hơn hoặc bằng giá trị đích. Khi restore production, cách chắc chắn hơn thường là trước tiên export một đoạn binlog theo khoảng thời gian để kiểm tra, tìm event hoặc transaction boundary tương ứng với thao tác sai, sau đó dùng `--stop-position` để replay chính thức.

Nếu đã tìm chính xác event position, dùng position đáng tin cậy hơn thời gian:

```bash
mysqlbinlog \
  --start-position=154 \
  --stop-position=987654 \
  binlog.000120 \
  | mysql --binary-mode --host=127.0.0.1 --user=root --password
```

Khi chỉ định nhiều file binlog cùng lúc, `--start-position` chỉ tác động lên file đầu tiên trong command, `--stop-position` chỉ tác động lên file cuối cùng; các file ở giữa sẽ được xử lý đầy đủ. Position là byte offset trong file binlog, không phải số thứ tự của event, và phải nằm trên một event boundary hợp lệ.

`mysql --binary-mode` không phải parameter thừa. Tài liệu chính thức đề cập rằng nếu output binlog chứa ký tự null như `\0`, không thêm `--binary-mode` có thể khiến client `mysql` không parse đúng.

MySQL hiện đại thường dùng binlog format `ROW`. Format này ghi lại row change, không nhất thiết hiển thị trực tiếp SQL gốc. Khi điều tra thao tác sai, có thể dùng cách dưới đây để decode row event thành comment dễ đọc:

```bash
mysqlbinlog --base64-output=DECODE-ROWS -vv binlog.000120 > binlog.000120.readable.sql
```

Lưu ý, output này chủ yếu dùng để kiểm tra thủ công. Tài liệu chính thức cũng nhắc rằng nếu muốn thực thi lại output của `mysqlbinlog`, `--base64-output=AUTO` mặc định mới là hành vi an toàn; không nên dùng các mode như `DECODE-ROWS` để replay chính thức.

Incremental restore bằng binlog có một số điều kiện:

- MySQL phải được bật binary logging từ trước.
- Thời gian giữ binlog phải bao phủ yêu cầu RPO.
- Full backup phải tìm được file binlog và position bắt đầu, hoặc có thông tin GTID.
- Trước khi restore, tốt nhất nên verify trên isolated instance, không replay binlog chưa chắc chắn trực tiếp vào production database.

Bản thân file binlog cũng cần được backup. Tài liệu chính thức của MySQL đưa ra một cách backup liên tục binlog như sau:

```bash
mysqlbinlog \
  --read-from-remote-server \
  --host=127.0.0.1 \
  --user=binlog_backup \
  --password \
  --raw \
  --stop-never \
  --connection-server-id=330610 \
  --result-file=/backup/mysql/binlog/ \
  binlog.000999
```

Command này sẽ lấy binlog ở định dạng binary nguyên bản và tiếp tục chờ event mới sau khi đến cuối file log mới nhất hiện tại. Nó khác với replica: khi connection bị ngắt, nó không tự reconnect như replica, vì vậy production script còn phải kết hợp process supervisor, alert và resume từ điểm dừng.

Khi pull binlog liên tục còn có một số điểm dễ bỏ sót:

- Account đọc binlog từ xa cần permission liên quan đến replication. Tài liệu MySQL 8.4 vẫn ghi permission `REPLICATION SLAVE` cho `--read-from-remote-server`; permission model và yêu cầu có thể khác giữa các version, nên khi tạo account phải xác nhận theo tài liệu của version hiện tại.
- `--stop-never` khiến `mysqlbinlog` duy trì connection đến source bằng một server ID; trong production nên cấu hình rõ `--connection-server-id` để tránh xung đột với replica hoặc process `mysqlbinlog` khác.
- Mode `--raw` mặc định dùng file cùng tên với binlog của source để ghi vào current directory; nếu file đã tồn tại thì sẽ bị overwrite. Dùng `--result-file` để chỉ định directory hoặc prefix độc lập, đồng thời monitor permission và capacity của directory.
- Nếu source đã bật binlog encryption, bản copy được `mysqlbinlog` pull về vẫn được lưu ở phía backup dưới dạng unencrypted. Cần dùng TLS cho đường truyền, đồng thời encryption, access control và delete protection cho backup directory.

## Dùng XtraBackup để physical backup

Logical backup export SQL, còn physical backup copy data file và log file liên quan của MySQL. Data volume càng lớn thì physical backup càng có lợi: backup và restore gần với việc copy file hơn, không cần replay từng `INSERT` với số lượng lớn.

Percona XtraBackup là physical backup tool open source thường dùng trong thực tế MySQL. Nó có thể backup dữ liệu của storage engine như InnoDB / XtraDB trong lúc MySQL đang chạy, thường dùng cho hot backup trong production. Tuy nhiên, “hot backup” chủ yếu áp dụng cho transactional engine như InnoDB; tài liệu Percona cũng nêu rằng khi copy dữ liệu non-InnoDB, các table InnoDB sẽ bị lock. Instance sử dụng nhiều storage engine cần được đánh giá ảnh hưởng riêng.

Một quy trình full backup tối thiểu như sau:

```bash
xtrabackup --backup --target-dir=/data/backups/mysql/base
```

Sau khi backup xong không thể dùng ngay để start; trước tiên phải prepare để data file đạt trạng thái nhất quán:

```bash
xtrabackup --prepare --target-dir=/data/backups/mysql/base
```

`--prepare` sẽ apply redo / undo để các file trong backup directory tạo thành snapshot nhất quán. Không được interrupt bước này; nếu sau backup còn phải tiếp tục merge incremental, cần dùng `--apply-log-only` theo tài liệu Percona để giữ trạng thái trước khi rollback transaction chưa commit.

Khi restore, phải dừng MySQL và bảo đảm `datadir` đích rỗng:

```bash
systemctl stop mysqld

mv /var/lib/mysql /var/lib/mysql.bak.$(date +%F-%H%M%S)
install -d -o mysql -g mysql /var/lib/mysql

xtrabackup --copy-back --target-dir=/data/backups/mysql/base

chown -R mysql:mysql /var/lib/mysql

systemctl start mysqld
```

Các command này trông không phức tạp, nhưng vấn đề dễ xảy ra nhất thực sự là compatibility giữa các version. Tài liệu Percona viết rất rõ: XtraBackup 8.4 chỉ hỗ trợ MySQL 8.4 và Percona Server for MySQL 8.4, không hỗ trợ MySQL 8.0 hoặc 9.x; với MySQL 8.0 cần xem phạm vi hỗ trợ của dòng XtraBackup 8.0. Trong production không được dùng “version gần giống nhau” để phán đoán có thể backup và restore hay không.

XtraBackup cũng hỗ trợ incremental backup, chẳng hạn tiếp tục backup dữ liệu thay đổi dựa trên một full backup:

```bash
xtrabackup \
  --backup \
  --target-dir=/data/backups/mysql/inc1 \
  --incremental-basedir=/data/backups/mysql/base
```

Thứ tự prepare, cách merge và các bước restore của incremental chain dễ viết sai hơn. Trước khi team diễn tập thành thạo, không nên ngay lập tức trông cậy vào một incremental chain phức tạp để restore. Cách chắc chắn hơn là bảo đảm trước tiên “physical full backup + binlog” có thể restore, sau đó tùy theo data volume và áp lực về window mà bổ sung incremental backup.

Nếu sau khi restore XtraBackup còn phải tiếp tục PITR, không được đoán điểm bắt đầu. `xtrabackup_binlog_info` trong backup directory ghi lại file binlog và position tại thời điểm backup:

```bash
cat /data/backups/mysql/base/xtrabackup_binlog_info
```

Sau khi restore physical full backup, bắt đầu từ position mà file này cung cấp và dùng `mysqlbinlog` để replay các log tiếp theo.

## Chọn logical backup hay physical backup thế nào?

Có thể lựa chọn theo data volume, mục tiêu restore và năng lực vận hành.

| Trường hợp                                                                   | Cách phù hợp hơn                                                        |
| ---------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| Database nhỏ, test database, export một table, migration giữa các môi trường | `mysqldump`                                                             |
| Cần xem hoặc chỉnh sửa thủ công nội dung backup                              | `mysqldump`                                                             |
| Logical migration quy mô vừa, muốn import/export song song                   | MySQL Shell Dump Utilities                                              |
| Database lớn, restore window ngắn, chủ yếu là table InnoDB                   | XtraBackup hoặc MySQL Enterprise Backup                                 |
| Cần point-in-time restore                                                    | Full backup + binlog                                                    |
| Cloud database instance                                                      | Ưu tiên snapshot / PITR của nhà cung cấp cloud, sau đó export để verify |

Không cần làm phức tạp việc lựa chọn tool. Database nhỏ dùng `mysqldump` không có vấn đề gì; script đơn giản và cũng dễ điều tra khi có lỗi. Khi data volume tăng, vấn đề restore file SQL chậm sẽ ngày càng rõ, lúc đó chuyển sang physical backup thực tế hơn.

Phương án tệ nhất thường là chỉ thực hiện một loại backup và không bao giờ verify restore.

## Nên diễn tập restore thế nào?

Backup script chạy thành công chỉ chứng minh đã tạo ra file. Chỉ restore drill mới trả lời được file có dùng được hay không.

Một buổi diễn tập tối thiểu có thể theo quy trình này:

1. Chuẩn bị một máy isolated, cài MySQL cùng major version.
2. Pull full backup gần nhất và binlog tương ứng, xác nhận encryption key, account, certificate và cách truy cập object storage đều hoạt động.
3. Restore full backup và ghi lại thời gian.
4. Replay binlog đến thời điểm chỉ định và ghi lại thời gian.
5. Verify số lượng database và table quan trọng, SQL nghiệp vụ quan trọng, stored procedure, event, trigger, account, role và permission.
6. Kiểm tra `charset`, `collation`, `time_zone`, `sql_mode`, read-only switch và network isolation, bảo đảm restore instance không bị traffic nghiệp vụ thật kết nối nhầm.
7. Dùng application kết nối tới restore instance và chạy một nhóm read-only smoke API.
8. Ghi lại RTO thực tế của lần diễn tập, thời điểm có thể restore đến, bước thất bại và các thao tác thủ công.

Không nên chỉ kiểm tra MySQL có start được hay không. Ít nhất cần kiểm tra một số nhóm dữ liệu:

```sql
-- Số row của table quan trọng
SELECT COUNT(*) FROM order_db.orders;

-- Thời gian ghi gần nhất
SELECT MAX(created_at) FROM order_db.orders;

-- Stored procedure và function
SHOW PROCEDURE STATUS WHERE Db = 'order_db';
SHOW FUNCTION STATUS WHERE Db = 'order_db';

-- Event
SHOW EVENTS FROM order_db;

-- Trigger
SHOW TRIGGERS FROM order_db;

-- Account, role và permission
SELECT user, host FROM mysql.user;
SHOW GRANTS FOR 'app_user'@'%';

-- Parameter môi trường quan trọng
SELECT
  @@character_set_server,
  @@collation_server,
  @@time_zone,
  @@sql_mode,
  @@read_only,
  @@super_read_only;
```

Nếu business có reconciliation table, transaction log table hoặc inventory table, cần ưu tiên verify các table này. Mục tiêu của restore drill rất cụ thể: sớm phát hiện các vấn đề như “backup thiếu object, binlog thiếu file, không restore được permission, thời gian import vượt xa dự kiến”, vốn sẽ nghiêm trọng hơn trong sự cố.

## Các hiểu lầm thường gặp

**Hiểu lầm 1: Có replica thì không cần backup.**

Replica có thể tiếp nhận traffic read/write, nhưng không ngăn được việc xóa hoặc sửa nhầm. Sau khi SQL sai được đồng bộ sang, replica cũng chuyển thành trạng thái sai. Replica có độ trễ có thể tranh thủ thêm một chút thời gian xử lý, nhưng vẫn không thể thay thế offline backup.

**Hiểu lầm 2: Chỉ backup data, không backup binlog.**

Cách này nhiều nhất chỉ restore được đến thời điểm full backup. Các write từ sau full backup đến trước khi xảy ra sự cố sẽ không thể tìm lại, RPO bị kéo dài theo chu kỳ backup.

**Hiểu lầm 3: Đặt file backup và database trên cùng một máy.**

Khi disk hỏng, phòng máy gặp sự cố hoặc directory bị xóa nhầm, data và backup có thể cùng biến mất. Ít nhất phải copy sang storage độc lập; với business quan trọng còn cần cân nhắc khác data center hoặc object storage.

**Hiểu lầm 4: Backup script không có alert khi fail.**

Có file cũ trong backup directory không có nghĩa lần backup gần nhất thành công. Script nên kiểm tra exit code, file size, thời gian tạo và checksum, đồng thời notify người phụ trách khi fail.

**Hiểu lầm 5: Không bao giờ restore.**

Backup không có restore drill thường có vẻ ít tốn công nhất, nhưng lại đắt nhất khi có sự cố. Các bước restore càng lâu không chạy thì càng dễ bị chặn bởi version, permission, path, disk space và parameter của tool.

**Hiểu lầm 6: Checksum hợp lệ nghĩa là restore được.**

Checksum chỉ cho biết file nhiều khả năng không bị hỏng trong quá trình copy và lưu trữ, không có nghĩa SQL sẽ import thuận lợi, physical backup sẽ start được, object permission đã đầy đủ hoặc chuỗi binlog liên tục. Chỉ restore drill mới chứng minh được năng lực restore.

**Hiểu lầm 7: Ai cũng có thể sửa hoặc xóa file backup.**

Nếu backup dùng chung permission với production account, hoặc script vận hành thông thường có thể trực tiếp overwrite, delete, thì khi gặp xóa nhầm, ransomware hay script bug, backup có thể cùng mất tác dụng. Business quan trọng ít nhất phải có một bản copy cross-account, cross-failure-domain, có version hoặc immutable policy, đồng thời hạn chế permission xóa.

## Một phương án cơ bản có thể triển khai

Nếu chưa có phương án sẵn, có thể bắt đầu từ cách này:

- Database business chủ yếu là InnoDB, data volume không lớn: mỗi ngày full backup bằng `mysqldump --single-transaction --routines --events --triggers --source-data=2`, giữ từ 7 đến 30 ngày, thời gian giữ binlog bao phủ cùng window.
- Khi data volume tăng: thực hiện XtraBackup full backup mỗi ngày hoặc mỗi tuần, quyết định có bổ sung incremental backup hay không dựa trên write volume của business, đồng thời backup binlog riêng.
- Sau khi backup được ghi xuống disk, tính checksum rồi copy sang storage độc lập; business quan trọng nên giữ thêm một bản copy được mã hóa, cross-account, có version hoặc immutable policy.
- Monitor thời điểm full backup mới nhất còn dùng được, thời điểm binlog mới nhất đã archive, tính liên tục của file binlog và trạng thái process archive.
- Thực hiện restore drill ít nhất mỗi tháng một lần; với business cốt lõi, thực hiện thêm một lần trước đợt promotion lớn, migration hoặc nâng cấp version.
- Viết một restore runbook, bao gồm người phụ trách, vị trí backup, cách decrypt, command restore, cách tìm binlog start position, môi trường restore isolated, SQL verify và hướng dẫn rollback.

Các chu kỳ trên chỉ là điểm bắt đầu, không phải đáp án tiêu chuẩn. Dữ liệu tài chính, order, payment và y tế có yêu cầu RPO/RTO nghiêm ngặt hơn; database báo cáo nội bộ và phân tích log thường có thể nới lỏng. Chiến lược backup nên được quyết định theo tổn thất của business, không phải theo template trên mạng.

## Tài liệu tham khảo

- [MySQL Reference Manual: mysqldump](https://dev.mysql.com/doc/refman/8.4/en/mysqldump.html)
- [MySQL Reference Manual: mysqlbinlog](https://dev.mysql.com/doc/refman/8.4/en/mysqlbinlog.html)
- [MySQL Reference Manual: Using mysqlbinlog to Back Up Binary Log Files](https://dev.mysql.com/doc/refman/8.4/en/mysqlbinlog-backup.html)
- [MySQL Reference Manual: Point-in-Time Recovery](https://dev.mysql.com/doc/refman/8.4/en/point-in-time-recovery-binlog.html)
- [MySQL Reference Manual: Binary Logging Options and Variables](https://dev.mysql.com/doc/refman/8.4/en/replication-options-binary-log.html)
- [MySQL Shell 8.4: Instance Dump Utility, Schema Dump Utility, and Table Dump Utility](https://dev.mysql.com/doc/mysql-shell/8.4/en/mysql-shell-utilities-dump-instance-schema.html)
- [MySQL Shell 8.4: Dump Loading Utility](https://dev.mysql.com/doc/mysql-shell/8.4/en/mysql-shell-utilities-load-dump.html)
- [MySQL Reference Manual: MySQL Releases: Innovation and LTS](https://dev.mysql.com/doc/refman/8.4/en/mysql-releases.html)
- [Percona XtraBackup 8.4 Documentation](https://docs.percona.com/percona-xtrabackup/8.4/index.html)
- [Percona XtraBackup 8.4: Prepare a full backup](https://docs.percona.com/percona-xtrabackup/8.4/prepare-full-backup.html)
- [Percona XtraBackup 8.4: Index of files created by Percona XtraBackup](https://docs.percona.com/percona-xtrabackup/8.4/xtrabackup-files.html)
- [Percona XtraBackup 8.0 Supported Versions](https://docs.percona.com/percona-xtrabackup/8.0/supported-versions.html)

<!-- @include: @article-footer.snippet.md -->
