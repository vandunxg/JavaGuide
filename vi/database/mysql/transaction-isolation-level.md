---
title: Giải thích chi tiết về transaction isolation level của MySQL
description: Giải thích chi tiết về đặc điểm và khác biệt của bốn transaction isolation level trong MySQL (READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE), phân tích các vấn đề concurrency như dirty read, non-repeatable read, phantom read, cùng cách InnoDB giải quyết phantom read thông qua MVCC và cơ chế lock.
category: Database
tag:
  - MySQL
head:
  - - meta
    - name: keywords
      content: MySQL transaction isolation level, READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE, dirty read, non-repeatable read, phantom read, MVCC, gap lock
---

> Bài viết do [SnailClimb](https://github.com/Snailclimb) và [guang19](https://github.com/guang19) cùng hoàn thành.

Để xem phần tổng quan cơ bản về transaction, hãy đọc bài viết này: [Tổng hợp các kiến thức thường gặp và câu hỏi phỏng vấn về MySQL](./mysql-questions-01.md#MySQL-事务)

## Tổng hợp transaction isolation level

SQL standard định nghĩa bốn transaction isolation level để cân bằng giữa tính isolation (Isolation) của transaction và hiệu năng khi concurrency. Level càng cao thì data consistency càng tốt, nhưng hiệu năng khi concurrency có thể càng thấp. Bốn level này là:

- **READ-UNCOMMITTED (đọc dữ liệu chưa commit)**: isolation level thấp nhất, cho phép đọc các thay đổi dữ liệu chưa được commit, có thể dẫn đến dirty read, phantom read hoặc non-repeatable read. Level này hiếm khi được sử dụng trong thực tế vì khả năng đảm bảo data consistency quá yếu.
- **READ-COMMITTED (đọc dữ liệu đã commit)**: cho phép đọc dữ liệu đã được commit bởi transaction chạy đồng thời, có thể ngăn dirty read, nhưng phantom read hoặc non-repeatable read vẫn có thể xảy ra. Đây là isolation level mặc định của phần lớn database (như Oracle, SQL Server).
- **REPEATABLE-READ (đọc lặp lại)**: kết quả của nhiều lần đọc cùng một field đều nhất quán, trừ khi dữ liệu được chính transaction hiện tại sửa đổi; có thể ngăn dirty read và non-repeatable read, nhưng phantom read vẫn có thể xảy ra. Isolation level mặc định của storage engine InnoDB trong MySQL chính là REPEATABLE READ. Ngoài ra, ở level này, InnoDB dùng cơ chế MVCC (multi-version concurrency control) và Next-Key Locks (gap lock + row lock) để giải quyết phantom read ở mức độ lớn.
- **SERIALIZABLE (có thể tuần tự hóa)**: isolation level cao nhất, đáp ứng đầy đủ yêu cầu isolation của ACID. Tất cả transaction lần lượt thực thi, vì vậy các transaction hoàn toàn không thể ảnh hưởng lẫn nhau; nói cách khác, level này có thể ngăn dirty read, non-repeatable read và phantom read.

| Transaction isolation level | Dirty Read | Non-Repeatable Read | Phantom Read               |
| --------------------------- | ---------- | ------------------- | -------------------------- |
| READ UNCOMMITTED            | √          | √                   | √                          |
| READ COMMITTED              | ×          | √                   | √                          |
| REPEATABLE READ             | ×          | ×                   | √ (standard) / ≈× (InnoDB) |
| SERIALIZABLE                | ×          | ×                   | ×                          |

**Kiểm tra level mặc định:**

Isolation level mặc định của storage engine InnoDB trong MySQL là **REPEATABLE READ**. Có thể xem bằng lệnh sau:

- Trước MySQL 8.0: `SELECT @@tx_isolation;`
- Từ MySQL 8.0 trở đi: `SELECT @@transaction_isolation;`

```bash
mysql> SELECT @@transaction_isolation;
+-------------------------+
| @@transaction_isolation |
+-------------------------+
| REPEATABLE-READ         |
+-------------------------+
```

**Cách InnoDB xử lý phantom read ở REPEATABLE READ:**

Theo định nghĩa của SQL standard về isolation level, REPEATABLE READ không thể ngăn phantom read. Tuy nhiên, cách triển khai của InnoDB tránh phantom read ở mức độ lớn thông qua các cơ chế sau:

- **Snapshot Read**: các câu lệnh **SELECT** thông thường được thực hiện thông qua cơ chế **MVCC**. Khi transaction bắt đầu, một snapshot dữ liệu được tạo; các snapshot read tiếp theo đều đọc version dữ liệu này, nhờ đó tránh nhìn thấy các row mới được transaction khác insert (phantom read) hoặc các row bị transaction khác sửa đổi (non-repeatable read).
- **Current Read**: các thao tác như `SELECT ... FOR UPDATE`, `SELECT ... LOCK IN SHARE MODE`, `INSERT`, `UPDATE`, `DELETE`. InnoDB dùng **Next-Key Lock** để lock các index record được quét và range (gap) giữa chúng, ngăn transaction khác insert record mới trong range này, từ đó tránh phantom read. Next-Key Lock là sự kết hợp giữa row lock (Record Lock) và gap lock (Gap Lock).

Cần lưu ý rằng mặc dù thường cho rằng isolation level càng cao thì concurrency càng kém, storage engine InnoDB đã tối ưu level REPEATABLE READ thông qua cơ chế MVCC. Trong nhiều trường hợp chỉ đọc hoặc đọc nhiều, ghi ít, performance của nó **có thể không khác biệt đáng kể so với READ COMMITTED**. Tuy nhiên, trong trường hợp ghi nhiều và xung đột concurrency cao, cơ chế gap lock của RR có thể tạo ra nhiều lock wait hơn RC.

Ngoài ra, trong một số trường hợp cụ thể, chẳng hạn distributed transaction yêu cầu consistency nghiêm ngặt (XA Transactions), InnoDB có thể yêu cầu hoặc khuyến nghị sử dụng isolation level SERIALIZABLE để bảo đảm data consistency toàn cục.

Sách 《Chuyên sâu về MySQL: Storage engine InnoDB (ấn bản 2)》, chương 7.7, viết như sau:

> Storage engine InnoDB hỗ trợ XA transaction và dùng XA transaction để hỗ trợ triển khai distributed transaction. Distributed transaction là transaction cho phép nhiều transactional resources độc lập tham gia vào một global transaction. Transaction resource thường là relational database system, nhưng cũng có thể là các loại resource khác. Global transaction yêu cầu mọi transaction tham gia hoặc cùng commit, hoặc cùng rollback; đây là yêu cầu cao hơn so với yêu cầu ACID ban đầu của transaction. Ngoài ra, khi sử dụng distributed transaction, transaction isolation level của storage engine InnoDB phải được đặt thành SERIALIZABLE.

## Minh họa thực tế

Trong phần dưới đây, tôi sẽ sử dụng 2 phiên command line MySQL để mô phỏng vấn đề dirty read khi nhiều thread (nhiều transaction) thao tác trên cùng một phần dữ liệu.

Trong cấu hình mặc định của command line MySQL, các transaction đều được tự động commit, nghĩa là sau khi thực thi câu lệnh SQL, thao tác COMMIT sẽ được thực hiện ngay. Nếu muốn bật transaction một cách tường minh, cần sử dụng lệnh: `START TRANSACTION`.

Có thể thiết lập isolation level bằng lệnh sau.

```sql
SET [SESSION|GLOBAL] TRANSACTION ISOLATION LEVEL [READ UNCOMMITTED|READ COMMITTED|REPEATABLE READ|SERIALIZABLE]
```

Tiếp theo, hãy xem một số câu lệnh điều khiển concurrency được sử dụng trong phần thực hành bên dưới:

- `START TRANSACTION` |`BEGIN`: bật transaction một cách tường minh.
- `COMMIT`: commit transaction, khiến mọi thay đổi đối với database trở thành vĩnh viễn.
- `ROLLBACK`: kết thúc transaction của user và hoàn tác mọi thay đổi chưa commit.

### Dirty read (đọc chưa commit)

![](<https://oss.javaguide.cn/github/javaguide/2019-31-1%E8%84%8F%E8%AF%BB(%E8%AF%BB%E6%9C%AA%E6%8F%90%E4%BA%A4)%E5%AE%9E%E4%BE%8B.jpg>)

### Ngăn dirty read (đọc đã commit)

![](https://oss.javaguide.cn/github/javaguide/2019-31-2%E8%AF%BB%E5%B7%B2%E6%8F%90%E4%BA%A4%E5%AE%9E%E4%BE%8B.jpg)

### Non-repeatable read

Vẫn là hình minh họa cho READ COMMITTED ở trên. Mặc dù đã tránh được READ UNCOMMITTED, nhưng lại xuất hiện vấn đề non-repeatable read khi một transaction chưa kết thúc.

![](https://oss.javaguide.cn/github/javaguide/2019-32-1%E4%B8%8D%E5%8F%AF%E9%87%8D%E5%A4%8D%E8%AF%BB%E5%AE%9E%E4%BE%8B.jpg)

### Repeatable read

![](https://oss.javaguide.cn/github/javaguide/2019-33-2%E5%8F%AF%E9%87%8D%E5%A4%8D%E8%AF%BB.jpg)

### Phantom read

#### Minh họa phantom read xảy ra

![](https://oss.javaguide.cn/github/javaguide/phantom_read.png)

Ở lần truy vấn đầu tiên các record có salary bằng 500, SQL script 1 chỉ có một record. SQL script 2 insert một record có salary bằng 500 và commit; sau đó, trong cùng transaction, SQL script 1 lại dùng current read để truy vấn và phát hiện có hai record có salary bằng 500. Đây chính là phantom read.

Lưu ý: về bản chất, ví dụ này cho thấy ngữ nghĩa của snapshot read lần đầu và current read lần hai khác nhau. Ở RR, MVCC có thể bảo đảm snapshot read không xuất hiện phantom read, còn Next-Key Lock có thể ràng buộc current read; nhưng khi trộn snapshot read và current read trong cùng một transaction, kết quả của hai lần đọc có thể khác nhau.

#### Cách giải quyết phantom read

Có nhiều cách giải quyết phantom read, nhưng tư tưởng cốt lõi là khi một transaction thao tác trên dữ liệu của một table, transaction khác không được phép thêm hoặc xóa dữ liệu trong table đó. Các cách giải quyết phantom read chủ yếu gồm:

1. Điều chỉnh transaction isolation level thành `SERIALIZABLE`.
2. Ở transaction isolation level repeatable read, thêm table lock cho table mà transaction thao tác.
3. Ở transaction isolation level repeatable read, thêm `Next-key Lock（Record Lock+Gap Lock）` cho table mà transaction thao tác.

### Tham khảo

- 《Chuyên sâu về MySQL: Storage engine InnoDB》
- <https://dev.MySQL.com/doc/refman/5.7/en/>
- [MySQL lock: Bảy câu hỏi đào sâu bản chất](https://tech.youzan.com/seven-questions-about-the-lock-of-MySQL/)
- [Mối quan hệ giữa transaction isolation level và lock trong InnoDB](https://tech.meituan.com/2014/08/20/innodb-lock.html)

<!-- @include: @article-footer.snippet.md -->
