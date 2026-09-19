---
title: "Tổng hợp câu hỏi phỏng vấn SQL thường gặp (2)"
description: "Phần thứ hai của tổng hợp câu hỏi phỏng vấn SQL thường gặp, giải thích chi tiết các câu lệnh thao tác dữ liệu DML như INSERT, UPDATE, DELETE, bao gồm kỹ thuật chèn hàng loạt, nhập từ bảng khác, chèn kèm cập nhật và các kỹ thuật thực hành khác."
category: Database
tag:
  - Database Basics
  - SQL
head:
  - - meta
    - name: keywords
      content: SQL interview questions,INSERT insert,UPDATE update,DELETE delete,batch insert,REPLACE INTO,data manipulation
---

> Nguồn câu hỏi: [Thử thách nâng cao SQL - Nowcoder](https://www.nowcoder.com/exam/oj?page=1&tab=SQL%E7%AF%87&topicId=240)

## Thao tác thêm, xóa, sửa

Tổng hợp các cách chèn bản ghi bằng SQL:

- **Chèn thông thường (toàn bộ trường)**: `INSERT INTO table_name VALUES (value1, value2, ...)`
- **Chèn thông thường (giới hạn trường)**: `INSERT INTO table_name (column1, column2, ...) VALUES (value1, value2, ...)`
- **Chèn nhiều bản ghi cùng lúc**: `INSERT INTO table_name (column1, column2, ...) VALUES (value1_1, value1_2, ...), (value2_1, value2_2, ...), ...`
- **Nhập từ bảng khác**: `INSERT INTO table_name SELECT * FROM table_name2 [WHERE key=value]`
- **Chèn kèm cập nhật**: `REPLACE INTO table_name VALUES (value1, value2, ...)` (lưu ý, nguyên lý của cách này là khi phát hiện khóa chính hoặc khóa của unique index bị trùng thì xóa bản ghi cũ rồi chèn lại)

### Chèn bản ghi (1)

**Mô tả**: Hệ thống Nowcoder ghi lại thông tin làm bài thi của mỗi người dùng vào bảng `exam_record`. Hiện có chi tiết làm bài của hai người dùng như sau:

- Người dùng 1001 bắt đầu làm bài thi 9001 lúc 22 giờ 11 phút 12 giây ngày 1 tháng 9 năm 2021, nộp bài sau 50 phút và đạt 90 điểm;
- Người dùng 1002 bắt đầu làm bài thi 9002 lúc 7 giờ 1 phút 2 giây ngày 4 tháng 9 năm 2021, sau 10 phút thì rời khỏi nền tảng.

Bảng bản ghi làm bài thi `exam_record` đã được tạo, cấu trúc như sau. Hãy dùng một câu lệnh để chèn hai bản ghi này vào bảng.

| Filed       | Type       | Null | Key | Extra          | Default | Comment           |
| ----------- | ---------- | ---- | --- | -------------- | ------- | ----------------- |
| id          | int(11)    | NO   | PRI | auto_increment | (NULL)  | ID tự tăng        |
| uid         | int(11)    | NO   |     |                | (NULL)  | ID người dùng     |
| exam_id     | int(11)    | NO   |     |                | (NULL)  | ID bài thi        |
| start_time  | datetime   | NO   |     |                | (NULL)  | Thời gian bắt đầu |
| submit_time | datetime   | YES  |     |                | (NULL)  | Thời gian nộp bài |
| score       | tinyint(4) | YES  |     |                | (NULL)  | Điểm              |

**Đáp án**:

```sql
// Có khóa chính tự tăng nên không cần gán thủ công
INSERT INTO exam_record (uid, exam_id, start_time, submit_time, score) VALUES
(1001, 9001, '2021-09-01 22:11:12', '2021-09-01 23:01:12', 90),
(1002, 9002, '2021-09-04 07:01:02', NULL, NULL);
```

### Chèn bản ghi (2)

**Mô tả**: Hiện có một bảng bản ghi làm bài thi `exam_record`, trong đó chứa các bản ghi làm bài của người dùng qua nhiều năm. Do dữ liệu ngày càng nhiều, việc duy trì ngày càng khó khăn, cần tinh gọn nội dung bảng dữ liệu và sao lưu dữ liệu lịch sử.

Bảng `exam_record`:

| Filed       | Type       | Null | Key | Extra          | Default | Comment           |
| ----------- | ---------- | ---- | --- | -------------- | ------- | ----------------- |
| id          | int(11)    | NO   | PRI | auto_increment | (NULL)  | ID tự tăng        |
| uid         | int(11)    | NO   |     |                | (NULL)  | ID người dùng     |
| exam_id     | int(11)    | NO   |     |                | (NULL)  | ID bài thi        |
| start_time  | datetime   | NO   |     |                | (NULL)  | Thời gian bắt đầu |
| submit_time | datetime   | YES  |     |                | (NULL)  | Thời gian nộp bài |
| score       | tinyint(4) | YES  |     |                | (NULL)  | Điểm              |

Đã tạo bảng mới `exam_record_before_2021` để sao lưu các bản ghi làm bài đã hoàn thành trước năm 2021. Cấu trúc giống bảng `exam_record`. Hãy nhập các bản ghi làm bài đã hoàn thành trước năm 2021 vào bảng này.

**Đáp án**:

```sql
INSERT INTO exam_record_before_2021 (uid, exam_id, start_time, submit_time, score)
SELECT uid,exam_id,start_time,submit_time,score
FROM exam_record
WHERE YEAR(submit_time) < 2021;
```

### Chèn bản ghi (3)

**Mô tả**: Hiện có một đề thi SQL độ khó cao có ID 9003, thời lượng một tiếng rưỡi. Hãy chèn thời gian phát hành `2021-01-01 00:00:00` vào bảng thông tin đề thi `examination_info`. Dù đề thi có ID này tồn tại hay không, thao tác vẫn phải chèn thành công. Hãy thử chèn đề thi này.

Bảng thông tin đề thi `examination_info`:

| Filed        | Type        | Null | Key | Extra          | Default | Comment              |
| ------------ | ----------- | ---- | --- | -------------- | ------- | -------------------- |
| id           | int(11)     | NO   | PRI | auto_increment | (NULL)  | ID tự tăng           |
| exam_id      | int(11)     | NO   | UNI |                | (NULL)  | ID bài thi           |
| tag          | varchar(32) | YES  |     |                | (NULL)  | Nhãn phân loại       |
| difficulty   | varchar(8)  | YES  |     |                | (NULL)  | Độ khó               |
| duration     | int(11)     | NO   |     |                | (NULL)  | Thời lượng (số phút) |
| release_time | datetime    | YES  |     |                | (NULL)  | Thời gian phát hành  |

**Đáp án**:

```sql
REPLACE INTO examination_info VALUES
 (NULL, 9003, "SQL", "hard", 90, "2021-01-01 00:00:00");
```

### Cập nhật bản ghi (1)

**Mô tả**: Hiện có một bảng thông tin đề thi `examination_info`, cấu trúc bảng như hình dưới:

| Filed        | Type     | Null | Key | Extra          | Default | Comment             |
| ------------ | -------- | ---- | --- | -------------- | ------- | ------------------- |
| id           | int(11)  | NO   | PRI | auto_increment | (NULL)  | ID tự tăng          |
| exam_id      | int(11)  | NO   | UNI |                | (NULL)  | ID bài thi          |
| tag          | char(32) | YES  |     |                | (NULL)  | Nhãn phân loại      |
| difficulty   | char(8)  | YES  |     |                | (NULL)  | Độ khó              |
| duration     | int(11)  | NO   |     |                | (NULL)  | Thời lượng          |
| release_time | datetime | YES  |     |                | (NULL)  | Thời gian phát hành |

Hãy sửa toàn bộ trường `tag` có giá trị `PYTHON` trong bảng **examination_info** thành `Python`.

**Ý tưởng**: Bài này có hai cách giải. Cách dễ nghĩ nhất là dùng trực tiếp `update + where` để chỉ định điều kiện cập nhật; cách thứ hai là tìm kiếm và thay thế dựa trên trường cần sửa.

**Đáp án 1**:

```sql
UPDATE examination_info SET tag = 'Python' WHERE tag='PYTHON'
```

**Đáp án 2**:

```sql
UPDATE examination_info
SET tag = REPLACE(tag,'PYTHON','Python')

# REPLACE (trường đích, "nội dung cần tìm", "nội dung thay thế")
```

### Cập nhật bản ghi (2)

**Mô tả**: Hiện có một bảng bản ghi làm bài thi `exam_record`, trong đó chứa các bản ghi làm bài của người dùng qua nhiều năm, cấu trúc như bảng dưới: Bảng bản ghi làm bài `exam_record`: **`submit_time`** là **thời gian hoàn thành** (chú ý câu này).

| Filed       | Type       | Null | Key | Extra          | Default | Comment           |
| ----------- | ---------- | ---- | --- | -------------- | ------- | ----------------- |
| id          | int(11)    | NO   | PRI | auto_increment | (NULL)  | ID tự tăng        |
| uid         | int(11)    | NO   |     |                | (NULL)  | ID người dùng     |
| exam_id     | int(11)    | NO   |     |                | (NULL)  | ID bài thi        |
| start_time  | datetime   | NO   |     |                | (NULL)  | Thời gian bắt đầu |
| submit_time | datetime   | YES  |     |                | (NULL)  | Thời gian nộp bài |
| score       | tinyint(4) | YES  |     |                | (NULL)  | Điểm              |

**Yêu cầu**: Hãy chuyển toàn bộ các bản ghi trong bảng `exam_record` bắt đầu làm bài **trước** ngày 1 tháng 9 năm 2021 và **chưa hoàn thành** sang trạng thái được đánh dấu là đã hoàn thành, tức là đổi thời gian hoàn thành thành `'2099-01-01 00:00:00'` và điểm thành `0`.

**Ý tưởng**: Hãy chú ý các từ khóa trong đề bài (đã được tô sáng), điều kiện **trước** một mốc thời gian. Khi đó cần nghĩ ngay đến việc so sánh thời gian. Có thể dùng trực tiếp `xxx_time < "2021-09-01 00:00:00"`, hoặc dùng hàm `date()` để so sánh. Điều kiện thứ hai là **chưa hoàn thành**, tức thời gian hoàn thành là `NULL`, tương ứng với thời gian nộp bài trong đề, nghĩa là `submit_time là NULL`.

**Đáp án**:

```sql
UPDATE exam_record SET submit_time = '2099-01-01 00:00:00', score = 0 WHERE DATE(start_time) < "2021-09-01" AND submit_time IS null
```

### Xóa bản ghi (1)

**Mô tả**: Hiện có một bảng bản ghi làm bài thi `exam_record`, trong đó chứa các bản ghi làm bài của người dùng qua nhiều năm.

Bảng bản ghi làm bài `exam_record`: **`start_time`** là thời gian bắt đầu làm bài, **`submit_time`** là thời gian nộp bài, tức thời gian kết thúc.

| Filed       | Type       | Null | Key | Extra          | Default | Comment           |
| ----------- | ---------- | ---- | --- | -------------- | ------- | ----------------- |
| id          | int(11)    | NO   | PRI | auto_increment | (NULL)  | ID tự tăng        |
| uid         | int(11)    | NO   |     |                | (NULL)  | ID người dùng     |
| exam_id     | int(11)    | NO   |     |                | (NULL)  | ID bài thi        |
| start_time  | datetime   | NO   |     |                | (NULL)  | Thời gian bắt đầu |
| submit_time | datetime   | YES  |     |                | (NULL)  | Thời gian nộp bài |
| score       | tinyint(4) | YES  |     |                | (NULL)  | Điểm              |

**Yêu cầu**: Hãy xóa các bản ghi trong bảng `exam_record` có thời gian làm bài dưới 5 phút và điểm không đạt (điểm đạt là 60 điểm).

**Ý tưởng**: Dù bài này luyện tập thao tác xóa, nhìn kỹ thì nó kiểm tra cách dùng các hàm thời gian. Với phép so sánh số phút được nêu ở đây, các hàm thường dùng là **`TIMEDIFF`** và **`TIMESTAMPDIFF`**. Cách dùng của hai hàm hơi khác nhau, trong đó hàm sau linh hoạt hơn; lựa chọn tùy thói quen.

1.　 `TIMEDIFF`: hiệu giữa hai thời điểm

```sql
TIMEDIFF(time1, time2)
```

Cả hai tham số đều bắt buộc, mỗi tham số là một biểu thức thời gian hoặc ngày giờ. Nếu tham số được chỉ định không hợp lệ hoặc là `NULL`, hàm sẽ trả về `NULL`.

Với bài này, có thể dùng `TIMEDIFF` bên trong hàm `MINUTE`, vì `TIMEDIFF` tính hiệu thời gian; bọc thêm hàm `MINUTE` bên ngoài sẽ tính ra số phút.

2. `TIMESTAMPDIFF`: dùng để tính chênh lệch thời gian giữa hai mốc thời gian

```sql
TIMESTAMPDIFF(unit,datetime_expr1,datetime_expr2)
# Giải thích tham số
# unit: đơn vị chênh lệch thời gian được trả về khi so sánh ngày, các giá trị thường dùng:
SECOND: giây
MINUTE: phút
HOUR: giờ
DAY: ngày
WEEK: tuần
MONTH: tháng
QUARTER: quý
YEAR: năm
# Hàm TIMESTAMPDIFF trả về kết quả datetime_expr2 - datetime_expr1 (nói đơn giản: sau - trước, tức 2 - 1).
# datetime_expr1 và datetime_expr2 có thể là giá trị kiểu DATE hoặc DATETIME
# (nói đơn giản: có thể là "2023-01-01" hoặc "2023-01-01- 00:00:00").
```

Bài này cần so sánh số phút, nên dùng `TIMESTAMPDIFF(MINUTE, thời gian bắt đầu, thời gian kết thúc) < 5`.

**Đáp án**:

```sql
DELETE FROM exam_record WHERE MINUTE (TIMEDIFF(submit_time , start_time)) < 5 AND score < 60
```

```sql
DELETE FROM exam_record WHERE TIMESTAMPDIFF(MINUTE, start_time, submit_time) < 5 AND score < 60
```

### Xóa bản ghi (2)

**Mô tả**: Hiện có một bảng bản ghi làm bài thi `exam_record`, trong đó chứa các bản ghi làm bài của người dùng qua nhiều năm, cấu trúc như bảng dưới:

Bảng bản ghi làm bài `exam_record`: `start_time` là thời gian bắt đầu làm bài, `submit_time` là thời gian nộp bài, tức thời gian kết thúc; nếu chưa hoàn thành thì để trống.

| Filed       | Type       | Null | Key | Extra          | Default | Comment           |
| ----------- | ---------- | :--: | --- | -------------- | ------- | ----------------- |
| id          | int(11)    |  NO  | PRI | auto_increment | (NULL)  | ID tự tăng        |
| uid         | int(11)    |  NO  |     |                | (NULL)  | ID người dùng     |
| exam_id     | int(11)    |  NO  |     |                | (NULL)  | ID bài thi        |
| start_time  | datetime   |  NO  |     |                | (NULL)  | Thời gian bắt đầu |
| submit_time | datetime   | YES  |     |                | (NULL)  | Thời gian nộp bài |
| score       | tinyint(4) | YES  |     |                | (NULL)  | Điểm              |

**Yêu cầu**: Trong các bản ghi của bảng `exam_record` mà **chưa hoàn thành** **hoặc** có thời gian làm bài dưới 5 phút, hãy xóa 3 bản ghi có thời gian bắt đầu làm bài sớm nhất.

**Ý tưởng**: Bài này khá đơn giản, nhưng cần chú ý thông tin trong đề: thời gian kết thúc nếu chưa hoàn thành thì để trống, đây chính là một điều kiện.

Điều kiện còn lại là nhỏ hơn 5 phút, tương tự bài trước, nhưng ở đây là **hoặc**, tức chỉ cần thỏa mãn một trong hai điều kiện. Ngoài ra, bài còn kiểm tra cách dùng `ORDER BY` và `LIMIT`.

**Đáp án**:

```sql
DELETE FROM exam_record WHERE submit_time IS null OR TIMESTAMPDIFF(MINUTE, start_time, submit_time) < 5
ORDER BY start_time
LIMIT 3
# Mặc định là asc, desc là sắp xếp giảm dần
```

### Xóa bản ghi (3)

**Mô tả**: Hiện có một bảng bản ghi làm bài thi `exam_record`, trong đó chứa các bản ghi làm bài của người dùng qua nhiều năm, cấu trúc như bảng dưới:

| Filed       | Type       | Null | Key | Extra          | Default | Comment           |
| ----------- | ---------- | :--: | --- | -------------- | ------- | ----------------- |
| id          | int(11)    |  NO  | PRI | auto_increment | (NULL)  | ID tự tăng        |
| uid         | int(11)    |  NO  |     |                | (NULL)  | ID người dùng     |
| exam_id     | int(11)    |  NO  |     |                | (NULL)  | ID bài thi        |
| start_time  | datetime   |  NO  |     |                | (NULL)  | Thời gian bắt đầu |
| submit_time | datetime   | YES  |     |                | (NULL)  | Thời gian nộp bài |
| score       | tinyint(4) | YES  |     |                | (NULL)  | Điểm              |

**Yêu cầu**: Hãy xóa tất cả bản ghi trong bảng `exam_record`, **đồng thời đặt lại khóa chính tự tăng**.

**Ý tưởng**: Bài này kiểm tra sự khác nhau giữa ba câu lệnh xóa, hãy chú ý phần được tô sáng vì yêu cầu đặt lại khóa chính:

- `DROP`: xóa sạch bảng, xóa cấu trúc bảng, không thể hoàn tác
- `TRUNCATE`: làm trống bảng, không xóa cấu trúc bảng, không thể hoàn tác
- `DELETE`: xóa dữ liệu, có thể hoàn tác

Ở đây chọn `TRUNCATE` vì: `TRUNCATE` chỉ có thể tác động lên bảng; `TRUNCATE` xóa tất cả các hàng trong bảng nhưng giữ nguyên cấu trúc bảng, các constraint, index và những thành phần khác; `TRUNCATE` đặt lại giá trị tự tăng của bảng; sau khi dùng `TRUNCATE`, dung lượng mà bảng và index chiếm dụng sẽ trở về kích thước ban đầu.

Bài này cũng có thể dùng `DELETE`, nhưng sau khi xóa còn phải dùng `ALTER` để thiết lập thủ công giá trị ban đầu của khóa chính.

Tương tự, cũng có thể dùng `DROP`: xóa trực tiếp toàn bộ bảng, bao gồm cả cấu trúc bảng, rồi tạo lại bảng.

**Đáp án**:

```sql
TRUNCATE  exam_record;
```

## Thao tác bảng và index

### Tạo bảng mới

**Mô tả**: Hiện có một bảng thông tin người dùng, trong đó chứa thông tin những người dùng đã đăng ký trên nền tảng qua nhiều năm. Khi nền tảng Nowcoder không ngừng phát triển và số người dùng tăng nhanh, để cung cấp dịch vụ hiệu quả cho những người dùng có mức độ hoạt động cao, cần tách một phần người dùng sang bảng mới.

Bảng thông tin người dùng ban đầu:

| Filed         | Type        | Null | Key | Default           | Extra          | Comment                |
| ------------- | ----------- | ---- | --- | ----------------- | -------------- | ---------------------- |
| id            | int(11)     | NO   | PRI | (NULL)            | auto_increment | ID tự tăng             |
| uid           | int(11)     | NO   | UNI | (NULL)            |                | ID người dùng          |
| nick_name     | varchar(64) | YES  |     | (NULL)            |                | Biệt danh              |
| achievement   | int(11)     | YES  |     | 0                 |                | Giá trị thành tựu      |
| level         | int(11)     | YES  |     | (NULL)            |                | Cấp độ người dùng      |
| job           | varchar(32) | YES  |     | (NULL)            |                | Định hướng nghề nghiệp |
| register_time | datetime    | YES  |     | CURRENT_TIMESTAMP |                | Thời gian đăng ký      |

Với vai trò là một data analyst, hãy **tạo bảng thông tin người dùng chất lượng cao `user_info_vip`**, có cấu trúc giống bảng thông tin người dùng.

Kết quả cần trả về như bảng dưới đây. Hãy viết câu lệnh tạo bảng để ghi lại tất cả giới hạn và mô tả trong bảng.

| Filed         | Type        | Null | Key | Default           | Extra          | Comment                |
| ------------- | ----------- | ---- | --- | ----------------- | -------------- | ---------------------- |
| id            | int(11)     | NO   | PRI | (NULL)            | auto_increment | ID tự tăng             |
| uid           | int(11)     | NO   | UNI | (NULL)            |                | ID người dùng          |
| nick_name     | varchar(64) | YES  |     | (NULL)            |                | Biệt danh              |
| achievement   | int(11)     | YES  |     | 0                 |                | Giá trị thành tựu      |
| level         | int(11)     | YES  |     | (NULL)            |                | Cấp độ người dùng      |
| job           | varchar(32) | YES  |     | (NULL)            |                | Định hướng nghề nghiệp |
| register_time | datetime    | YES  |     | CURRENT_TIMESTAMP |                | Thời gian đăng ký      |

**Ý tưởng**: Nếu đề bài cho tên bảng cũ, có thể dùng trực tiếp `create table new_table as select * from old_table;`. Nhưng bài này không cho tên bảng cũ nên cần tự tạo bảng, chỉ cần chú ý tạo giá trị mặc định và key, tương đối đơn giản. (Lưu ý: nếu thực thi trên Nowcoder, hãy đảm bảo `comment` khớp với `comment` trong đề bài, bao gồm cả chữ hoa chữ thường; nếu không sẽ không được chấp nhận, đồng thời cũng cần đặt đúng charset.)

Đáp án:

```sql
CREATE TABLE IF NOT EXISTS user_info_vip(
    id INT(11) PRIMARY KEY AUTO_INCREMENT COMMENT'ID tự tăng',
    uid INT(11) UNIQUE NOT NULL COMMENT 'ID người dùng',
    nick_name VARCHAR(64) COMMENT'Biệt danh',
    achievement INT(11) DEFAULT 0 COMMENT 'Giá trị thành tựu',
    `level` INT(11) COMMENT 'Cấp độ người dùng',
    job VARCHAR(32) COMMENT 'Định hướng nghề nghiệp',
    register_time DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT 'Thời gian đăng ký'
)CHARACTER SET UTF8
```

### Sửa đổi bảng

**Mô tả**: Hiện có một bảng thông tin người dùng `user_info`, trong đó chứa thông tin những người dùng đã đăng ký trên nền tảng qua nhiều năm.

**Bảng thông tin người dùng `user_info`:**

| Filed         | Type        | Null | Key | Default           | Extra          | Comment                |
| ------------- | ----------- | ---- | --- | ----------------- | -------------- | ---------------------- |
| id            | int(11)     | NO   | PRI | (NULL)            | auto_increment | ID tự tăng             |
| uid           | int(11)     | NO   | UNI | (NULL)            |                | ID người dùng          |
| nick_name     | varchar(64) | YES  |     | (NULL)            |                | Biệt danh              |
| achievement   | int(11)     | YES  |     | 0                 |                | Giá trị thành tựu      |
| level         | int(11)     | YES  |     | (NULL)            |                | Cấp độ người dùng      |
| job           | varchar(32) | YES  |     | (NULL)            |                | Định hướng nghề nghiệp |
| register_time | datetime    | YES  |     | CURRENT_TIMESTAMP |                | Thời gian đăng ký      |

**Yêu cầu:** Trong bảng thông tin người dùng, hãy thêm sau trường `level` một cột `school` có thể lưu tối đa 15 ký tự; đổi tên cột `job` thành `profession`, đồng thời đổi độ dài trường `varchar` thành 10; đặt giá trị mặc định của `achievement` là 0.

**Ý tưởng**: Trước hết cần nắm cách dùng cơ bản của câu lệnh `ALTER`:

- Thêm một cột: `ALTER TABLE table_name ADD COLUMN column_name type 【first | after column_name】;` (`first`: thêm trước một cột; `after` thì ngược lại)
- Sửa kiểu hoặc constraint của cột: `ALTER TABLE table_name MODIFY COLUMN column_name new_type 【new_constraint】;`
- Đổi tên cột: `ALTER TABLE table_name change COLUMN old_column_name new_column_name type;`
- Xóa cột: `ALTER TABLE table_name drop COLUMN column_name;`
- Đổi tên bảng: `ALTER TABLE table_name rename 【to】 new_table_name;`
- Đưa một cột lên cột đầu tiên: `ALTER TABLE table_name MODIFY COLUMN column_name type first;`

Từ khóa `COLUMN` thực ra có thể bỏ qua, nhưng ở đây vẫn liệt kê theo quy chuẩn.

Khi sửa đổi, nếu có nhiều mục cần sửa thì có thể viết cùng nhau, nhưng cần chú ý định dạng.

**Đáp án**:

```sql
ALTER TABLE user_info
    ADD school VARCHAR(15) AFTER level,
    CHANGE job profession VARCHAR(10),
    MODIFY achievement INT(11) DEFAULT 0;
```

### Xóa bảng

**Mô tả**: Hiện có một bảng bản ghi làm bài thi `exam_record`, trong đó chứa các bản ghi làm bài của người dùng qua nhiều năm. Thông thường mỗi năm sẽ tạo một bảng sao lưu cho bảng `exam_record` là `exam_record_{YEAR}`, trong đó `{YEAR}` là năm tương ứng.

Hiện dữ liệu ngày càng nhiều và sắp hết dung lượng lưu trữ, hãy xóa các bảng sao lưu từ lâu (từ năm 2011 đến năm 2014) nếu chúng tồn tại.

**Ý tưởng**: Bài này rất đơn giản, chỉ cần xóa trực tiếp. Nếu thấy phiền, có thể ngăn cách các bảng cần xóa bằng dấu phẩy và viết trên một dòng; chắc chắn sẽ có bạn hỏi: nếu cần xóa rất nhiều bảng thì sao? Nếu cần xóa nhiều bảng, có thể viết script để thực hiện.

**Đáp án**:

```sql
DROP TABLE IF EXISTS exam_record_2011;
DROP TABLE IF EXISTS exam_record_2012;
DROP TABLE IF EXISTS exam_record_2013;
DROP TABLE IF EXISTS exam_record_2014;
```

### Tạo index

**Mô tả**: Hiện có một bảng thông tin đề thi `examination_info`, trong đó chứa thông tin về nhiều loại đề thi. Để truy vấn bảng thuận tiện và nhanh hơn, cần tạo các index sau trong bảng `examination_info`:

Quy tắc: tạo index thường `idx_duration` trên cột `duration`, tạo unique index `uniq_idx_exam_id` trên cột `exam_id`, tạo full-text index `full_idx_tag` trên cột `tag`.

Theo yêu cầu, kết quả trả về như sau:

| examination_info | 0   | PRIMARY          | 1   | id       | A   | 0   |     |     |     | BTREE    |
| ---------------- | --- | ---------------- | --- | -------- | --- | --- | --- | --- | --- | -------- |
| examination_info | 0   | uniq_idx_exam_id | 1   | exam_id  | A   | 0   |     |     | YES | BTREE    |
| examination_info | 1   | idx_duration     | 1   | duration | A   | 0   |     |     |     | BTREE    |
| examination_info | 1   | full_idx_tag     | 1   | tag      |     | 0   |     |     | YES | FULLTEXT |

Ghi chú: Hệ thống sẽ dùng câu lệnh `SHOW INDEX FROM examination_info` để đối chiếu kết quả.

**Ý tưởng**: Trước hết cần nắm các loại index thường gặp:

- B-Tree index: B-Tree (hay còn gọi là cây cân bằng) index là loại index phổ biến và mặc định nhất. Nó phù hợp với nhiều điều kiện truy vấn, có thể nhanh chóng định vị dữ liệu thỏa mãn điều kiện. B-Tree index phù hợp với thao tác tìm kiếm thông thường, hỗ trợ truy vấn bằng giá trị, truy vấn phạm vi và sắp xếp.
- Unique index: Unique index tương tự B-Tree index thông thường, điểm khác biệt là nó yêu cầu giá trị của cột được tạo index phải là duy nhất. Điều này có nghĩa là khi chèn hoặc cập nhật dữ liệu, MySQL sẽ kiểm tra tính duy nhất của cột index.
- Primary key index: Primary key index là một loại unique index đặc biệt, dùng để định danh duy nhất từng hàng dữ liệu trong bảng. Mỗi bảng chỉ có một primary key index, giúp tăng tốc độ truy cập dữ liệu và bảo đảm tính toàn vẹn dữ liệu.
- Full-text index: Full-text index dùng để tìm kiếm toàn văn trong dữ liệu văn bản. Nó hỗ trợ tìm kiếm từ khóa trong trường văn bản, không chỉ tìm kiếm theo giá trị bằng hoặc theo phạm vi đơn giản. Full-text index phù hợp với các trường hợp sử dụng cần tìm kiếm toàn văn.

```sql
-- Ví dụ:
-- Thêm B-Tree index:
    CREATE INDEX index_name (tên index) ON table_name (tên trường);   -- index_name là tên index, các dòng dưới cũng vậy
-- Tạo unique index:
    CREATE UNIQUE INDEX index_name ON table_name (tên trường);
-- Tạo primary key index:
    ALTER TABLE table_name ADD PRIMARY KEY (tên trường);
-- Tạo full-text index:
    ALTER TABLE table_name ADD FULLTEXT INDEX index_name (tên trường);

-- Qua các ví dụ trên có thể thấy cả create và alter đều có thể thêm index
```

Sau khi nắm các kiến thức cơ bản trên, đáp án của bài này cũng đã rõ.

**Đáp án**:

```sql
ALTER TABLE examination_info
    ADD INDEX idx_duration(duration),
    ADD UNIQUE INDEX uniq_idx_exam_id(exam_id),
    ADD FULLTEXT INDEX full_idx_tag(tag);
```

### Xóa index

**Mô tả**: Hãy xóa unique index `uniq_idx_exam_id` và full-text index `full_idx_tag` trên bảng `examination_info`.

**Ý tưởng**: Bài này kiểm tra cú pháp cơ bản để xóa index:

```sql
-- Dùng DROP INDEX để xóa index
DROP INDEX index_name ON table_name;

-- Dùng ALTER TABLE để xóa index
ALTER TABLE employees DROP INDEX idx_email;
```

Cần lưu ý rằng trong MySQL không hỗ trợ thao tác xóa nhiều index cùng lúc. Mỗi lần xóa index chỉ được chỉ định một tên index.

Ngoài ra, lệnh **DROP** cần được sử dụng thận trọng!

**Đáp án**:

```sql
DROP INDEX uniq_idx_exam_id ON examination_info;
DROP INDEX full_idx_tag ON examination_info;
```

<!-- @include: @article-footer.snippet.md -->
