---
title: "Tổng hợp câu hỏi phỏng vấn SQL thường gặp (3)"
description: "Phần ba tổng hợp câu hỏi phỏng vấn SQL thường gặp, giải thích chi tiết cách dùng các hàm tổng hợp COUNT, SUM, AVG, MAX, MIN, cũng như các kỹ thuật nâng cao như GROUP BY, lọc HAVING và tính trung bình cắt ngọn."
category: Database
tag:
  - Database Basics
  - SQL
head:
  - - meta
    - name: keywords
      content: "Câu hỏi phỏng vấn SQL,hàm tổng hợp,COUNT,SUM,AVG,MAX,MIN,GROUP BY,HAVING,trung bình cắt ngọn"
---

> Nguồn đề: [Nowcoder Challenge - Thử thách SQL nâng cao](https://www.nowcoder.com/exam/oj?page=1&tab=SQL%E7%AF%87&topicId=240)

Với các câu hỏi khó hoặc rất khó, bạn có thể quyết định bỏ qua hay không tùy tình hình thực tế và nhu cầu phỏng vấn.

## Hàm tổng hợp

### Trung bình cắt ngọn điểm bài thi khó thuộc nhóm SQL (khá khó)

**Mô tả**: Nhân viên vận hành của Nowcoder muốn xem tình hình điểm của mọi người trong các bài thi khó thuộc nhóm SQL.

Hãy tính trung bình cắt ngọn điểm của tất cả người dùng đã hoàn thành bài thi khó thuộc nhóm SQL từ bảng dữ liệu `exam_record` (loại bỏ một giá trị lớn nhất và một giá trị nhỏ nhất rồi tính trung bình).

Dữ liệu mẫu: `examination_info` (`exam_id` ID bài thi, `tag` nhóm bài thi, `difficulty` độ khó bài thi, `duration` thời lượng thi, `release_time` thời gian phát hành)

| id  | exam_id | tag        | difficulty | duration | release_time        |
| --- | ------- | ---------- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL        | hard       | 60       | 2020-01-01 10:00:00 |
| 2   | 9002    | Thuật toán | medium     | 80       | 2020-08-02 10:00:00 |

Dữ liệu mẫu: `exam_record` (uid ID người dùng, exam_id ID bài thi, start_time thời gian bắt đầu làm bài, submit_time thời gian nộp bài, score điểm)

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-02 09:01:01 | 2020-01-02 09:21:01 | 80     |
| 2   | 1001 | 9001    | 2021-05-02 10:01:01 | 2021-05-02 10:30:01 | 81     |
| 3   | 1001 | 9001    | 2021-06-02 19:01:01 | 2021-06-02 19:31:01 | 84     |
| 4   | 1001 | 9002    | 2021-09-05 19:01:01 | 2021-09-05 19:40:01 | 89     |
| 5   | 1001 | 9001    | 2021-09-02 12:01:01 | (NULL)              | (NULL) |
| 6   | 1001 | 9002    | 2021-09-01 12:01:01 | (NULL)              | (NULL) |
| 7   | 1002 | 9002    | 2021-02-02 19:01:01 | 2021-02-02 19:30:01 | 87     |
| 8   | 1002 | 9001    | 2021-05-05 18:01:01 | 2021-05-05 18:59:02 | 90     |
| 9   | 1003 | 9001    | 2021-09-07 12:01:01 | 2021-09-07 10:31:01 | 50     |
| 10  | 1004 | 9001    | 2021-09-06 10:01:01 | (NULL)              | (NULL) |

Kết quả truy vấn theo dữ liệu đầu vào:

| tag | difficulty | clip_avg_score |
| --- | ---------- | -------------- |
| SQL | hard       | 81.7           |

Từ bảng `examination_info` có thể biết bài thi 9001 là bài thi SQL độ khó cao. Điểm của các lượt làm bài là [80,81,84,90,50], sau khi bỏ điểm cao nhất và thấp nhất còn [80,81,84], điểm trung bình là 81.6666667, làm tròn đến một chữ số thập phân được 81.7.

**Mô tả đầu vào:**

Dữ liệu đầu vào có ít nhất 3 điểm hợp lệ.

**Cách 1:** Để tìm bài thi SQL độ khó cao, chắc chắn cần liên kết bảng `examination_info`, sau đó tìm bài thi độ khó cao. Từ `examination_info` biết được bài thi SQL độ khó cao có `exam_id` là 9001, vậy tiếp theo dùng `exam_id = 9001` làm điều kiện truy vấn.

Trước hết tìm bài thi 9001: `select * from exam_record where exam_id = 9001`

Sau đó tìm điểm cao nhất: `select max(score) điểm cao nhất from exam_record where exam_id = 9001`

Tiếp theo tìm điểm thấp nhất: `select min(score) điểm thấp nhất from exam_record where exam_id = 9001`

Trong tập kết quả điểm đã truy vấn, để loại bỏ điểm cao nhất và thấp nhất, cách trực quan nhất là dùng NOT IN hoặc NOT EXISTS. Ở đây dùng NOT IN.

Trước hết viết phần chính: `select tag, difficulty, round(avg(score), 1) clip_avg_score from examination_info info INNER JOIN exam_record record`

**Mẹo nhỏ**: Hàm `ROUND()` của MySQL: `ROUND(X)` trả về số nguyên gần X nhất; `ROUND(X,D)` trả về X với giá trị được giữ đến D chữ số sau dấu thập phân, chữ số thứ D được làm tròn.

Sau đó ghép các câu lệnh "rời rạc" ở trên là được. Lưu ý, trong NOT IN cần dùng `UNION ALL` để liên kết hai truy vấn con; dùng `UNION` để đưa kết quả của max và min vào cùng một tập, tạo thành hiệu ứng một cột nhiều dòng.

**Đáp án 1:**

```sql
SELECT tag, difficulty, ROUND(AVG(score), 1) clip_avg_score
\tFROM examination_info info  INNER JOIN exam_record record
\t\tWHERE info.exam_id = record.exam_id
\t\t\tAND  record.exam_id = 9001
\t\t\t\tAND record.score NOT IN(
\t\t\t\t\tSELECT MAX(score)
\t\t\t\t\t\tFROM exam_record
\t\t\t\t\t\t\tWHERE exam_id = 9001
\t\t\t\t\t\t\t\tUNION ALL
\t\t\t\t\tSELECT MIN(score)
\t\t\t\t\t\tFROM exam_record
\t\t\t\t\t\t\tWHERE exam_id = 9001
\t\t\t\t)
```

Đây là cách giải trực quan và dễ nghĩ đến nhất, nhưng vẫn cần cải thiện. Đây là cách dựa vào dữ liệu cụ thể để vượt qua, còn nếu tuân thủ chặt chẽ yêu cầu đề bài thì nên viết như sau:

```sql
SELECT tag,
       difficulty,
       ROUND(AVG(score), 1) clip_avg_score
FROM examination_info info
INNER JOIN exam_record record
WHERE info.exam_id = record.exam_id
  AND record.exam_id =
    (SELECT examination_info.exam_id
     FROM examination_info
     WHERE tag = 'SQL'
       AND difficulty = 'hard' )
  AND record.score NOT IN
    (SELECT MAX(score)
     FROM exam_record
     WHERE exam_id =
         (SELECT examination_info.exam_id
          FROM examination_info
          WHERE tag = 'SQL'
            AND difficulty = 'hard' )
     UNION ALL SELECT MIN(score)
     FROM exam_record
     WHERE exam_id =
         (SELECT examination_info.exam_id
          FROM examination_info
          WHERE tag = 'SQL'
            AND difficulty = 'hard' ) )
```

Tuy nhiên bạn sẽ thấy các câu lệnh lặp lại rất nhiều, vì vậy có thể dùng `WITH` để trích xuất phần dùng chung.

**Giới thiệu mệnh đề `WITH`:**

Mệnh đề `WITH`, còn gọi là Common Table Expression (CTE), là cách định nghĩa bảng tạm trong truy vấn SQL. Nó cho phép tạo một tập kết quả được đặt tên tạm thời trong truy vấn và tham chiếu đến tập kết quả đó trong cùng truy vấn.

Cách dùng cơ bản:

```sql
WITH cte_name (column1, column2, ..., columnN) AS (
    -- Thân truy vấn
    SELECT ...
    FROM ...
    WHERE ...
)
-- Truy vấn chính
SELECT ...
FROM cte_name
WHERE ...
```

Mệnh đề `WITH` gồm các phần sau:

- `cte_name`: Đặt tên cho bảng tạm để có thể tham chiếu trong truy vấn chính.
- `(column1, column2, ..., columnN)`: Không bắt buộc, dùng để chỉ định tên cột của bảng tạm.
- `AS`: Bắt buộc, biểu thị bắt đầu định nghĩa bảng tạm.
- `Thân truy vấn CTE`: Câu lệnh truy vấn thực tế dùng để định nghĩa dữ liệu trong bảng tạm.

Một trong các mục đích chính của mệnh đề `WITH` là tăng khả năng đọc và bảo trì truy vấn, đặc biệt khi có nhiều truy vấn con lồng nhau hoặc cần dùng lại cùng logic truy vấn. Bằng cách đặt logic này trong một bảng tạm có tên, ta có thể tổ chức rõ ràng hơn và loại bỏ code trùng lặp.

Ngoài ra, mệnh đề `WITH` còn có thể thực hiện truy vấn đệ quy trong các truy vấn phức tạp. Truy vấn đệ quy cho phép thực hiện nhiều lần lặp trên cùng một bảng trong một truy vấn duy nhất, từng bước xây dựng tập kết quả. Điều này hữu ích khi xử lý dữ liệu phân cấp, cơ cấu tổ chức và cấu trúc cây.

**Chi tiết nhỏ**: MySQL phiên bản 5.7 trở về trước không hỗ trợ sử dụng alias trực tiếp trong mệnh đề `WITH`.

Đáp án cải tiến:

```sql
WITH t1 AS
  (SELECT record.*,
          info.tag,
          info.difficulty
   FROM exam_record record
   INNER JOIN examination_info info ON record.exam_id = info.exam_id
   WHERE info.tag = "SQL"
     AND info.difficulty = "hard" )
SELECT tag,
       difficulty,
       ROUND(AVG(score), 1)
FROM t1
WHERE score NOT IN
    (SELECT max(score)
     FROM t1
     UNION SELECT min(score)
     FROM t1)
```

**Cách 2:**

- Lọc bài thi SQL độ khó cao: `where tag="SQL" and difficulty="hard"`
- Tính trung bình cắt ngọn: `(tổng - giá trị lớn nhất - giá trị nhỏ nhất) / (tổng số - 2)`:
  - `(sum(score) - max(score) - min(score)) / (count(score) - 2)`
  - Một nhược điểm là nếu có nhiều giá trị lớn nhất và nhỏ nhất thì phương pháp này khó lọc ra. Tuy nhiên đề bài nói rõ là **trung bình sau khi bỏ một giá trị lớn nhất và một giá trị nhỏ nhất**, nên ở đây có thể dùng công thức này.

**Đáp án 2:**

```sql
SELECT info.tag,
       info.difficulty,
       ROUND((SUM(record.score)- MIN(record.score)- MAX(record.score)) / (COUNT(record.score)- 2), 1) AS clip_avg_score
FROM examination_info info,
     exam_record record
WHERE info.exam_id = record.exam_id
  AND info.tag = "SQL"
  AND info.difficulty = "hard";
```

### Thống kê số lượt làm bài

Có một bảng ghi lại lượt làm bài `exam_record`. Hãy thống kê tổng số lượt làm bài `total_pv`, số lượt làm bài đã hoàn thành `complete_pv` và số bài thi đã hoàn thành `complete_exam_cnt`.

Dữ liệu mẫu bảng `exam_record` (uid ID người dùng, `exam_id` ID bài thi, `start_time` thời gian bắt đầu làm bài, `submit_time` thời gian nộp bài, `score` điểm):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-02 09:01:01 | 2020-01-02 09:21:01 | 80     |
| 2   | 1001 | 9001    | 2021-05-02 10:01:01 | 2021-05-02 10:30:01 | 81     |
| 3   | 1001 | 9001    | 2021-06-02 19:01:01 | 2021-06-02 19:31:01 | 84     |
| 4   | 1001 | 9002    | 2021-09-05 19:01:01 | 2021-09-05 19:40:01 | 89     |
| 5   | 1001 | 9001    | 2021-09-02 12:01:01 | (NULL)              | (NULL) |
| 6   | 1001 | 9002    | 2021-09-01 12:01:01 | (NULL)              | (NULL) |
| 7   | 1002 | 9002    | 2021-02-02 19:01:01 | 2021-02-02 19:30:01 | 87     |
| 8   | 1002 | 9001    | 2021-05-05 18:01:01 | 2021-05-05 18:59:02 | 90     |
| 9   | 1003 | 9001    | 2021-09-07 12:01:01 | 2021-09-07 10:31:01 | 50     |
| 10  | 1004 | 9001    | 2021-09-06 10:01:01 | (NULL)              | (NULL) |

Kết quả mẫu:

| total_pv | complete_pv | complete_exam_cnt |
| -------- | ----------- | ----------------- |
| 10       | 7           | 2                 |

Giải thích: Tính đến hiện tại có 10 lượt ghi nhận làm bài, trong đó 7 lượt đã hoàn thành (lượt thoát giữa chừng ở trạng thái chưa hoàn thành, thời gian nộp bài và điểm là NULL), các bài thi đã hoàn thành là 9001 và 9002.

**Cách làm**: Khi thấy yêu cầu thống kê số lượt, chắc chắn trước tiên phải nghĩ đến hàm `COUNT`. Vấn đề là phải viết thế nào để thống kê các bản ghi khác nhau? Có thể giải quyết bằng truy vấn con (bài này cũng có thể viết bằng case when, cách giải tương tự nhưng logic khác). Trước hết hãy tìm hiểu cách dùng cơ bản của `COUNT`.

Cú pháp cơ bản của hàm `COUNT()`:

```sql
COUNT(expression)
```

`expression` có thể là tên cột, biểu thức, hằng số hoặc wildcard. Một số ví dụ dùng thường gặp:

1. Tính số lượng tất cả các dòng trong bảng:

```sql
SELECT COUNT(*) FROM table_name;
```

2. Tính số lượng giá trị không rỗng (khác NULL) của một cột:

```sql
SELECT COUNT(column_name) FROM table_name;
```

3. Tính số dòng thỏa mãn điều kiện:

```sql
SELECT COUNT(*) FROM table_name WHERE condition;
```

4. Kết hợp với `GROUP BY` để tính số dòng của từng nhóm sau khi phân nhóm:

```sql
SELECT column_name, COUNT(*) FROM table_name GROUP BY column_name;
```

5. Tính số tổ hợp duy nhất của các cột:

```sql
SELECT COUNT(DISTINCT column_name1, column_name2) FROM table_name;
```

Khi dùng hàm `COUNT()`, nếu không chỉ định tham số hoặc dùng `COUNT(*)`, hàm sẽ tính số lượng tất cả dòng. Nếu dùng tên cột, hàm chỉ tính số lượng giá trị không rỗng của cột đó.

Ngoài ra, kết quả của hàm `COUNT()` là một giá trị số nguyên. Kể cả khi kết quả bằng 0, hàm cũng không trả về NULL, cần ghi nhớ điều này.

**Đáp án**:

```sql
SELECT
\tcount(*) total_pv,
\t( SELECT count(*) FROM exam_record WHERE submit_time IS NOT NULL ) complete_pv,
\t( SELECT COUNT( DISTINCT exam_id, score IS NOT NULL OR NULL ) FROM exam_record ) complete_exam_cnt
FROM
\texam_record
```

Ở đây cần giải thích kỹ câu `COUNT( DISTINCT exam_id, score IS NOT NULL OR NULL)`: biểu thức `score IS NOT NULL OR NULL` trả về TRUE khi score khác NULL và trả về NULL khi score là NULL. Lưu ý, nếu không thêm `OR NULL`, phần điều kiện sẽ trả về TRUE/FALSE; giá trị FALSE tương ứng với 0.

Bản thân `COUNT` không thể đếm số dòng trên nhiều cột. Việc thêm `DISTINCT` khiến nhiều cột được xem như một tổ hợp duy nhất để có thể đếm số dòng xuất hiện; khi tính `COUNT(DISTINCT ...)`, chỉ các dòng có giá trị khác NULL mới được tính, cũng cần chú ý điều này.

Ngoài ra, qua bài này có thể biết được mẫu thường dùng để thêm điều kiện cho count: `count( điều kiện cột or null)`.

### Điểm thấp nhất không nhỏ hơn điểm trung bình

**Mô tả**: Hãy tìm điểm thấp nhất của người dùng có điểm bài thi SQL không nhỏ hơn điểm trung bình của các bài thi thuộc nhóm đó trong bảng ghi lại lượt làm bài.

Dữ liệu mẫu bảng `exam_record` (uid ID người dùng, `exam_id` ID bài thi, `start_time` thời gian bắt đầu làm bài, `submit_time` thời gian nộp bài, `score` điểm):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-02 09:01:01 | 2020-01-02 09:21:01 | 80     |
| 2   | 1002 | 9001    | 2021-09-05 19:01:01 | 2021-09-05 19:40:01 | 89     |
| 3   | 1002 | 9002    | 2021-09-02 12:01:01 | (NULL)              | (NULL) |
| 4   | 1002 | 9003    | 2021-09-01 12:01:01 | (NULL)              | (NULL) |
| 5   | 1002 | 9001    | 2021-02-02 19:01:01 | 2021-02-02 19:30:01 | 87     |
| 6   | 1002 | 9002    | 2021-05-05 18:01:01 | 2021-05-05 18:59:02 | 90     |
| 7   | 1003 | 9002    | 2021-02-06 12:01:01 | (NULL)              | (NULL) |
| 8   | 1003 | 9003    | 2021-09-07 10:01:01 | 2021-09-07 10:31:01 | 86     |
| 9   | 1004 | 9003    | 2021-09-06 12:01:01 | (NULL)              | (NULL) |

Bảng `examination_info` (`exam_id` ID bài thi, `tag` nhóm bài thi, `difficulty` độ khó bài thi, `duration` thời lượng thi, `release_time` thời gian phát hành)

| id  | exam_id | tag        | difficulty | duration | release_time        |
| --- | ------- | ---------- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL        | hard       | 60       | 2020-01-01 10:00:00 |
| 2   | 9002    | SQL        | easy       | 60       | 2020-02-01 10:00:00 |
| 3   | 9003    | Thuật toán | medium     | 80       | 2020-08-02 10:00:00 |

Dữ liệu đầu ra mẫu:

| min_score_over_avg |
| ------------------ |
| 87                 |

**Giải thích**: Bài thi 9001 và 9002 thuộc nhóm SQL, điểm của các lượt làm hai bài thi này là [80,89,87,90], điểm trung bình là 86.5, điểm nhỏ nhất không thấp hơn điểm trung bình là 87.

**Cách làm**: Nhìn qua dạng bài này quả thực khá phức tạp vì chưa biết bắt đầu từ đâu. Nhưng sau khi đọc kỹ đề, cần biết nắm bắt thông tin then chốt. Ví dụ bài này: `Hãy tìm điểm thấp nhất của người dùng có điểm bài thi SQL không nhỏ hơn điểm trung bình của các bài thi thuộc nhóm đó trong bảng ghi lại lượt làm bài.` Bạn có thể rút ra những thông tin hữu ích nào để làm hướng giải?

Thứ nhất: Tìm điểm bài thi ==SQL==.

Thứ hai: ==Điểm trung bình== của nhóm bài thi đó.

Thứ ba: ==Điểm thấp nhất của người dùng== trong nhóm bài thi đó.

Sau đó “cầu nối” ở giữa là ==không nhỏ hơn==.

Tách các điều kiện, lần lượt hoàn thành:

```sql
-- Tìm điểm có tag là ‘SQL’   【80, 89,87,90】
-- Tính điểm trung bình của nhóm này
select  ROUND(AVG(score), 1) from  examination_info info INNER JOIN exam_record record
\twhere info.exam_id = record.exam_id
\tand tag= 'SQL'
```

Sau đó tìm điểm thấp nhất của nhóm bài thi này, rồi so sánh tập kết quả `【80, 89,87,90】` với điểm trung bình để có đáp án cuối cùng.

**Đáp án**:

```sql
SELECT MIN(score) AS min_score_over_avg
FROM examination_info info
INNER JOIN exam_record record
WHERE info.exam_id = record.exam_id
  AND tag= 'SQL'
  AND score >=
    (SELECT ROUND(AVG(score), 1)
     FROM examination_info info
     INNER JOIN exam_record record
     WHERE info.exam_id = record.exam_id
       AND tag= 'SQL' )
```

Thực ra yêu cầu của dạng bài này nhìn có vẻ khá “vòng vèo”, nhưng chỉ cần sắp xếp lại cẩn thận, tách điều kiện lớn thành các điều kiện nhỏ, rồi ghép chúng lại. Chỉ cần nhớ: **nắm phần chính, sắp xếp các nhánh**, vấn đề sẽ được giải quyết.

## Truy vấn phân nhóm

### Số ngày hoạt động trung bình và số người hoạt động hàng tháng

**Mô tả**: Bản ghi làm bài của người dùng trong khu vực làm bài của Nowcoder được lưu trong bảng `exam_record`, nội dung như sau:

Bảng `exam_record` (uid ID người dùng, `exam_id` ID bài thi, `start_time` thời gian bắt đầu làm bài, `submit_time` thời gian nộp bài, `score` điểm)

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2021-07-02 09:01:01 | 2021-07-02 09:21:01 | 80     |
| 2   | 1002 | 9001    | 2021-09-05 19:01:01 | 2021-09-05 19:40:01 | 81     |
| 3   | 1002 | 9002    | 2021-09-02 12:01:01 | (NULL)              | (NULL) |
| 4   | 1002 | 9003    | 2021-09-01 12:01:01 | (NULL)              | (NULL) |
| 5   | 1002 | 9001    | 2021-07-02 19:01:01 | 2021-07-02 19:30:01 | 82     |
| 6   | 1002 | 9002    | 2021-07-05 18:01:01 | 2021-07-05 18:59:02 | 90     |
| 7   | 1003 | 9002    | 2021-07-06 12:01:01 | (NULL)              | (NULL) |
| 8   | 1003 | 9003    | 2021-09-07 10:01:01 | 2021-09-07 10:31:01 | 86     |
| 9   | 1004 | 9003    | 2021-09-06 12:01:01 | (NULL)              | (NULL) |
| 10  | 1002 | 9003    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 81     |
| 11  | 1005 | 9001    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 88     |
| 12  | 1006 | 9002    | 2021-09-02 12:11:01 | 2021-09-02 12:31:01 | 89     |
| 13  | 1007 | 9002    | 2020-09-02 12:11:01 | 2020-09-02 12:31:01 | 89     |

Hãy tính số ngày hoạt động trung bình hàng tháng `avg_active_days` và số người hoạt động hàng tháng `mau` trong khu vực làm bài thi ở mỗi tháng năm 2021. Kết quả mẫu của dữ liệu trên:

| month  | avg_active_days | mau |
| ------ | --------------- | --- |
| 202107 | 1.50            | 2   |
| 202109 | 1.25            | 4   |

**Giải thích**: Tháng 7 năm 2021 có 2 người hoạt động, tổng cộng hoạt động 3 ngày (1001 hoạt động 1 ngày, 1002 hoạt động 2 ngày), số ngày hoạt động trung bình là 1.5. Tháng 9 năm 2021 có 4 người hoạt động, tổng cộng hoạt động 5 ngày, số ngày hoạt động trung bình là 1.25. Kết quả giữ lại 2 chữ số thập phân.

Lưu ý: hoạt động ở đây có nghĩa là có hành vi ==nộp bài==.

**Cách làm**: Sau khi đọc đề, trước hết chú ý phần được nhấn mạnh. Thông thường khi tính số ngày và số người hoạt động hàng tháng, cần nghĩ ngay đến các hàm ngày tháng. Bài này cũng được tách nhỏ để giải quyết. Trước tiên để tính số người hoạt động chắc chắn cần dùng `COUNT`, nhưng có một điểm dễ sai: người dùng 1002 làm hai bài thi khác nhau trong tháng 9, nên cần loại trùng, nếu không số người hoạt động sẽ sai. Điểm thứ hai là phải biết định dạng ngày tháng. Như bảng trên, đề yêu cầu hiển thị theo định dạng `202107`, nên cần dùng `DATE_FORMAT` để định dạng.

Cách dùng cơ bản:

`DATE_FORMAT(date_value, format)`

- `date_value` là giá trị ngày hoặc thời gian cần định dạng.
- `format` là định dạng ngày hoặc thời gian được chỉ định (giống định dạng ngày tháng trong Java).

**Đáp án**:

```sql
SELECT DATE_FORMAT(submit_time, '%Y%m') MONTH,
                                        round(count(DISTINCT UID, DATE_FORMAT(submit_time, '%Y%m%d')) / count(DISTINCT UID), 2) avg_active_days,
                                        COUNT(DISTINCT UID) mau
FROM exam_record
WHERE YEAR (submit_time) = 2021
GROUP BY MONTH
```

Nói thêm một câu: dùng `COUNT(DISTINCT uid, DATE_FORMAT(submit_time, '%Y%m%d'))` có thể thống kê số lượng giá trị tổ hợp của cột `uid` và `submit_time` sau khi định dạng theo năm, tháng, ngày.

### Tổng số lượt làm bài theo tháng và số lượt làm bài trung bình mỗi ngày

**Mô tả**: Có một bảng ghi lại việc luyện câu hỏi `practice_record`, dữ liệu mẫu như sau:

| id  | uid  | question_id | submit_time         | score |
| --- | ---- | ----------- | ------------------- | ----- |
| 1   | 1001 | 8001        | 2021-08-02 11:41:01 | 60    |
| 2   | 1002 | 8001        | 2021-09-02 19:30:01 | 50    |
| 3   | 1002 | 8001        | 2021-09-02 19:20:01 | 70    |
| 4   | 1002 | 8002        | 2021-09-02 19:38:01 | 70    |
| 5   | 1003 | 8002        | 2021-08-01 19:38:01 | 80    |

Hãy thống kê tổng số lượt luyện câu hỏi theo tháng `month_q_cnt` và số lượt luyện câu hỏi trung bình mỗi ngày `avg_day_q_cnt` của người dùng trong từng tháng năm 2021 (sắp xếp tăng dần theo tháng), cùng với tình hình tổng thể của năm đó. Kết quả mẫu:

| submit_month  | month_q_cnt | avg_day_q_cnt |
| ------------- | ----------- | ------------- |
| 202108        | 2           | 0.065         |
| 202109        | 3           | 0.100         |
| Tổng hợp 2021 | 5           | 0.161         |

**Giải thích**: Tháng 8 năm 2021 có tổng cộng 2 lượt luyện, trung bình mỗi ngày là 2/31=0.065 (giữ 3 chữ số thập phân); tháng 9 năm 2021 có tổng cộng 3 lượt luyện, trung bình mỗi ngày là 3/30=0.100; năm 2021 có tổng cộng 5 lượt luyện (trung bình tổng hợp của năm không có ý nghĩa thực tế, ở đây tính theo 31 ngày: 5/31=0.161).

> Nowcoder đã dùng phiên bản MySQL mới nhất. Nếu khi chạy xuất hiện lỗi `ONLY_FULL_GROUP_BY`, nghĩa là trong phép tổng hợp GROUP BY, nếu cột trong SELECT không xuất hiện trong GROUP BY thì SQL đó không hợp lệ. Vì cột không nằm trong mệnh đề GROUP BY, các cột được truy vấn phải xuất hiện trong GROUP BY nếu không sẽ báo lỗi, hoặc trường đó phải nằm trong hàm tổng hợp.

**Cách làm:**

Nhìn dữ liệu mẫu cần nghĩ ngay đến các hàm liên quan. Ví dụ `submit_month` cần dùng `DATE_FORMAT` để định dạng ngày tháng, sau đó truy vấn số lượt luyện mỗi tháng.

Số lượt luyện mỗi tháng:

```sql
SELECT MONTH ( submit_time ), COUNT( question_id )
FROM
\tpractice_record
GROUP BY
\tMONTH (submit_time)
```

Tiếp theo, cột thứ ba cần dùng hàm `DAY(LAST_DAY(date_value))` để tìm số ngày trong tháng của một ngày cho trước.

Ví dụ:

```sql
SELECT DAY(LAST_DAY('2023-07-08')) AS days_in_month;
-- Kết quả: 31

SELECT DAY(LAST_DAY('2023-02-01')) AS days_in_month;
-- Kết quả: 28 (tháng 2 của năm không nhuận)

SELECT DAY(LAST_DAY(NOW())) AS days_in_current_month;
-- Kết quả: 31 (số ngày của tháng hiện tại)
```

Dùng hàm `LAST_DAY()` để lấy ngày cuối cùng của tháng chứa ngày đã cho, sau đó dùng hàm `DAY()` để trích xuất số ngày của ngày đó. Như vậy có thể lấy được số ngày của tháng chỉ định.

Cần chú ý, hàm `LAST_DAY()` trả về một giá trị ngày, còn hàm `DAY()` dùng để trích xuất phần ngày trong giá trị đó.

Sau khi phân tích như trên, có thể viết đáp án ngay. Bài này phức tạp chủ yếu ở phần xử lý ngày tháng, logic không khó.

**Đáp án**:

```sql
SELECT DATE_FORMAT(submit_time, '%Y%m') submit_month,
       count(question_id) month_q_cnt,
       ROUND(COUNT(question_id) / DAY (LAST_DAY(submit_time)), 3) avg_day_q_cnt
FROM practice_record
WHERE DATE_FORMAT(submit_time, '%Y') = '2021'
GROUP BY submit_month
UNION ALL
SELECT 'Tổng hợp 2021' AS submit_month,
       count(question_id) month_q_cnt,
       ROUND(COUNT(question_id) / 31, 3) avg_day_q_cnt
FROM practice_record
WHERE DATE_FORMAT(submit_time, '%Y') = '2021'
ORDER BY submit_month
```

Trong kết quả mẫu, vì dòng cuối cần đưa ra dữ liệu tổng hợp nên phải dùng `UNION ALL` để thêm vào tập kết quả. Đừng quên sắp xếp ở cuối!

### Người dùng hợp lệ có số bài thi chưa hoàn thành lớn hơn 1 (khá khó)

**Mô tả**: Có bảng ghi lại lượt làm bài `exam_record` (`uid` ID người dùng, `exam_id` ID bài thi, `start_time` thời gian bắt đầu làm bài, `submit_time` thời gian nộp bài, `score` điểm), dữ liệu mẫu như sau:

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2021-07-02 09:01:01 | 2021-07-02 09:21:01 | 80     |
| 2   | 1002 | 9001    | 2021-09-05 19:01:01 | 2021-09-05 19:40:01 | 81     |
| 3   | 1002 | 9002    | 2021-09-02 12:01:01 | (NULL)              | (NULL) |
| 4   | 1002 | 9003    | 2021-09-01 12:01:01 | (NULL)              | (NULL) |
| 5   | 1002 | 9001    | 2021-07-02 19:01:01 | 2021-07-02 19:30:01 | 82     |
| 6   | 1002 | 9002    | 2021-07-05 18:01:01 | 2021-07-05 18:59:02 | 90     |
| 7   | 1003 | 9002    | 2021-07-06 12:01:01 | (NULL)              | (NULL) |
| 8   | 1003 | 9003    | 2021-09-07 10:01:01 | 2021-09-07 10:31:01 | 86     |
| 9   | 1004 | 9003    | 2021-09-06 12:01:01 | (NULL)              | (NULL) |
| 10  | 1002 | 9003    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 81     |
| 11  | 1005 | 9001    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 88     |
| 12  | 1006 | 9002    | 2021-09-02 12:11:01 | 2021-09-02 12:31:01 | 89     |
| 13  | 1007 | 9002    | 2020-09-02 12:11:01 | 2020-09-02 12:31:01 | 89     |

Ngoài ra có bảng thông tin bài thi `examination_info` (`exam_id` ID bài thi, `tag` nhóm bài thi, `difficulty` độ khó bài thi, `duration` thời lượng thi, `release_time` thời gian phát hành), dữ liệu mẫu như sau:

| id  | exam_id | tag        | difficulty | duration | release_time        |
| --- | ------- | ---------- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL        | hard       | 60       | 2020-01-01 10:00:00 |
| 2   | 9002    | SQL        | easy       | 60       | 2020-02-01 10:00:00 |
| 3   | 9003    | Thuật toán | medium     | 80       | 2020-08-02 10:00:00 |

Hãy thống kê dữ liệu của những người dùng hợp lệ có số lượt làm bài thi chưa hoàn thành trong năm 2021 lớn hơn 1. (Người dùng hợp lệ là người có ít nhất 1 lượt làm bài hoàn thành và số lượt chưa hoàn thành nhỏ hơn 5.) Xuất ID người dùng, số lượt làm bài chưa hoàn thành, số lượt làm bài hoàn thành và tập hợp `tag` của các bài thi đã làm; sắp xếp giảm dần theo số lượt chưa hoàn thành. Kết quả mẫu:

| uid  | incomplete_cnt | complete_cnt | detail                                                                            |
| ---- | -------------- | ------------ | --------------------------------------------------------------------------------- |
| 1002 | 2              | 4            | 2021-09-01:Thuật toán;2021-07-02:SQL;2021-09-02:SQL;2021-09-05:SQL;2021-07-05:SQL |

**Giải thích**: Trong các lượt làm bài năm 2021, ngoại trừ 1004, các người dùng khác đều thỏa định nghĩa người dùng hợp lệ. Nhưng chỉ 1002 có số bài thi chưa hoàn thành lớn hơn 1, nên chỉ xuất 1002. `detail` là tập `{ngày:tag}` của các bài thi 1002 đã làm; giữa ngày và tag dùng **:**, giữa nhiều phần tử dùng **;**.

**Cách làm:**

Đọc kỹ đề có thể phân tích: trước hết cần liên kết bảng vì sau đó phải xuất `tag`.

Lọc dữ liệu năm 2021:

```sql
SELECT *
FROM exam_record er
LEFT JOIN examination_info ei ON er.exam_id = ei.exam_id
WHERE YEAR (er.start_time)= 2021
```

Nhóm theo uid, sau đó kiểm tra điều kiện với từng người dùng. Đề yêu cầu `số bài thi hoàn thành ít nhất là 1, số bài thi chưa hoàn thành lớn hơn 1 và nhỏ hơn 5`.

Vậy khi viết SQL, điều kiện phải là: `chưa hoàn thành > 1 and đã hoàn thành >=1 and chưa hoàn thành < 5`.

Vì cuối cùng cần nối chuỗi và ghép các thành phần, có thể dùng hàm `GROUP_CONCAT`. Dưới đây là giới thiệu ngắn về cách dùng hàm này:

Cú pháp cơ bản:

```sql
GROUP_CONCAT([DISTINCT] expr [ORDER BY {unsigned_integer | col_name | expr} [ASC | DESC] [, ...]]             [SEPARATOR sep])
```

- `expr`: Cột hoặc biểu thức cần nối.
- `DISTINCT`: Tham số tùy chọn, dùng để loại trùng. Khi chỉ định `DISTINCT`, cùng một giá trị chỉ xuất hiện một lần.
- `ORDER BY`: Tham số tùy chọn, dùng để sắp xếp các giá trị sau khi nối. Có thể chọn tăng dần (`ASC`) hoặc giảm dần (`DESC`).
- `SEPARATOR sep`: Tham số tùy chọn, dùng để đặt dấu phân cách sau khi nối. (Bài này dùng tham số để đặt dấu `;`.)

Hàm `GROUP_CONCAT()` thường được dùng trong mệnh đề `GROUP BY`, nối các giá trị của một nhóm dòng thành một chuỗi và trả về dưới dạng tổng hợp trong tập kết quả.

**Đáp án**:

```sql
SELECT a.uid,
       SUM(CASE
               WHEN a.submit_time IS NULL THEN 1
           END) AS incomplete_cnt,
       SUM(CASE
               WHEN a.submit_time IS NOT NULL THEN 1
           END) AS complete_cnt,
       GROUP_CONCAT(DISTINCT CONCAT(DATE_FORMAT(a.start_time, '%Y-%m-%d'), ':', b.tag)
                    ORDER BY start_time SEPARATOR ";") AS detail
FROM exam_record a
LEFT JOIN examination_info b ON a.exam_id = b.exam_id
WHERE YEAR (a.start_time)= 2021
GROUP BY a.uid
HAVING incomplete_cnt > 1
AND complete_cnt >= 1
AND incomplete_cnt < 5
ORDER BY incomplete_cnt DESC
```

- `SUM(CASE WHEN a.submit_time IS NULL THEN 1 END)` thống kê số bản ghi chưa hoàn thành của mỗi người dùng.
- `SUM(CASE WHEN a.submit_time IS NOT NULL THEN 1 END)` thống kê số bản ghi đã hoàn thành của mỗi người dùng.
- `GROUP_CONCAT(DISTINCT CONCAT(DATE_FORMAT(a.start_time, '%Y-%m-%d'), ':', b.tag) ORDER BY a.start_time SEPARATOR ';')` nối ngày thi và nhãn của mỗi người dùng thành một chuỗi phân cách bằng dấu chấm phẩy, đồng thời sắp xếp theo thời gian bắt đầu thi.

## Truy vấn con lồng nhau

### Các nhóm bài thi người dùng có số bài thi hoàn thành trung bình mỗi tháng không nhỏ hơn 3 thường làm (khá khó)

**Mô tả**: Có bảng ghi lại lượt làm bài `exam_record` (`uid`: ID người dùng, `exam_id`: ID bài thi, `start_time`: thời gian bắt đầu làm bài, `submit_time`: thời gian nộp bài, nếu không nộp thì là NULL, `score`: điểm), dữ liệu mẫu như sau:

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2021-07-02 09:01:01 | (NULL)              | (NULL) |
| 2   | 1002 | 9003    | 2021-09-01 12:01:01 | 2021-09-01 12:21:01 | 60     |
| 3   | 1002 | 9002    | 2021-09-02 12:01:01 | 2021-09-02 12:31:01 | 70     |
| 4   | 1002 | 9001    | 2021-09-05 19:01:01 | 2021-09-05 19:40:01 | 81     |
| 5   | 1002 | 9002    | 2021-07-06 12:01:01 | (NULL)              | (NULL) |
| 6   | 1003 | 9003    | 2021-09-07 10:01:01 | 2021-09-07 10:31:01 | 86     |
| 7   | 1003 | 9003    | 2021-09-08 12:01:01 | 2021-09-08 12:11:01 | 40     |
| 8   | 1003 | 9001    | 2021-09-08 13:01:01 | (NULL)              | (NULL) |
| 9   | 1003 | 9002    | 2021-09-08 14:01:01 | (NULL)              | (NULL) |
| 10  | 1003 | 9003    | 2021-09-08 15:01:01 | (NULL)              | (NULL) |
| 11  | 1005 | 9001    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 88     |
| 12  | 1005 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 88     |
| 13  | 1005 | 9002    | 2021-09-02 12:11:01 | 2021-09-02 12:31:01 | 89     |

Bảng thông tin bài thi `examination_info` (`exam_id`: ID bài thi, `tag`: nhóm bài thi, `difficulty`: độ khó bài thi, `duration`: thời lượng thi, `release_time`: thời gian phát hành), dữ liệu mẫu như sau:

| id  | exam_id | tag        | difficulty | duration | release_time        |
| --- | ------- | ---------- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL        | hard       | 60       | 2020-01-01 10:00:00 |
| 2   | 9002    | C++        | easy       | 60       | 2020-02-01 10:00:00 |
| 3   | 9003    | Thuật toán | medium     | 80       | 2020-08-02 10:00:00 |

Hãy thống kê các nhóm bài thi mà những người dùng có “số bài thi hoàn thành trung bình mỗi tháng” không nhỏ hơn 3 thường làm, cùng số lượt làm bài; xuất kết quả giảm dần theo số lượt. Kết quả mẫu:

| tag        | tag_cnt |
| ---------- | ------- |
| C++        | 4       |
| SQL        | 2       |
| Thuật toán | 1       |

**Giải thích**: Người dùng 1002 và 1005 đều có 3 bài thi hoàn thành trong tháng 09 năm 2021, các người dùng khác đều nhỏ hơn 3. Sau đó phân bố `tag` của các bài thi mà 1002 và 1005 đã làm, sắp xếp số lượt giảm dần, lần lượt là C++, SQL, Thuật toán.

**Cách làm**: Bài này kiểm tra truy vấn con kết hợp, trọng tâm là `số bài thi hoàn thành trung bình mỗi tháng >= 3`. Tuy nhiên theo ý kiến cá nhân, cách diễn đạt chưa thật rõ; nếu nói trực tiếp là truy vấn tháng 9 thì sẽ dễ hiểu hơn. Không phải tháng nào cũng cần >= 3, cũng không phải tổng số lượt làm bài chia cho số tháng làm bài. Đừng hiểu sai.

Trước tiên truy vấn những người dùng có ít nhất ba lượt làm bài mỗi tháng:

```sql
SELECT UID
FROM exam_record record
GROUP BY UID,
         MONTH (start_time)
HAVING count(submit_time) >= 3
```

Sau bước này có thể đi sâu hơn. Chỉ cần hiểu bước trên (không bị cụm “trung bình mỗi tháng” trong đề làm nhiễu), rồi lồng thêm một truy vấn con để tìm những người dùng nằm trong tập đó, sau đó truy vấn các cột đề yêu cầu. Nhớ sắp xếp!

```sql
SELECT tag,
       count(start_time) AS tag_cnt
FROM exam_record record
INNER JOIN examination_info info ON record.exam_id = info.exam_id
WHERE UID IN
    (SELECT UID
     FROM exam_record record
     GROUP BY UID,
              MONTH (start_time)
     HAVING count(submit_time) >= 3)
GROUP BY tag
ORDER BY tag_cnt DESC
```

### Số người làm bài và điểm trung bình trong ngày phát hành bài thi

**Mô tả**: Có bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` nickname, `achievement` thành tích, `level` cấp độ, `job` hướng nghề nghiệp, `register_time` thời gian đăng ký), dữ liệu mẫu như sau:

| id  | uid  | nick_name  | achievement | level | job        | register_time       |
| --- | ---- | ---------- | ----------- | ----- | ---------- | ------------------- |
| 1   | 1001 | Nowcoder 1 | 3100        | 7     | Thuật toán | 2020-01-01 10:00:00 |
| 2   | 1002 | Nowcoder 2 | 2100        | 6     | Thuật toán | 2020-01-01 10:00:00 |
| 3   | 1003 | Nowcoder 3 | 1500        | 5     | Thuật toán | 2020-01-01 10:00:00 |
| 4   | 1004 | Nowcoder 4 | 1100        | 4     | Thuật toán | 2020-01-01 10:00:00 |
| 5   | 1005 | Nowcoder 5 | 1600        | 6     | C++        | 2020-01-01 10:00:00 |
| 6   | 1006 | Nowcoder 6 | 3000        | 6     | C++        | 2020-01-01 10:00:00 |

**Giải nghĩa**: Người dùng 1001 có nickname Nowcoder 1, thành tích 3100, cấp độ 7, hướng nghề nghiệp là Thuật toán, thời gian đăng ký 2020-01-01 10:00:00.

Bảng thông tin bài thi `examination_info` (`exam_id` ID bài thi, `tag` nhóm bài thi, `difficulty` độ khó bài thi, `duration` thời lượng thi, `release_time` thời gian phát hành), dữ liệu mẫu:

| id  | exam_id | tag        | difficulty | duration | release_time        |
| --- | ------- | ---------- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL        | hard       | 60       | 2021-09-01 06:00:00 |
| 2   | 9002    | C++        | easy       | 60       | 2020-02-01 10:00:00 |
| 3   | 9003    | Thuật toán | medium     | 80       | 2020-08-02 10:00:00 |

Bảng ghi lại lượt làm bài `exam_record` (`uid` ID người dùng, `exam_id` ID bài thi, `start_time` thời gian bắt đầu làm bài, `submit_time` thời gian nộp bài, `score` điểm), dữ liệu mẫu:

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2021-07-02 09:01:01 | 2021-09-01 09:41:01 | 70     |
| 2   | 1002 | 9003    | 2021-09-01 12:01:01 | 2021-09-01 12:21:01 | 60     |
| 3   | 1002 | 9002    | 2021-09-02 12:01:01 | 2021-09-02 12:31:01 | 70     |
| 4   | 1002 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:40:01 | 80     |
| 5   | 1002 | 9003    | 2021-08-01 12:01:01 | 2021-08-01 12:21:01 | 60     |
| 6   | 1002 | 9002    | 2021-08-02 12:01:01 | 2021-08-02 12:31:01 | 70     |
| 7   | 1002 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:40:01 | 85     |
| 8   | 1002 | 9002    | 2021-07-06 12:01:01 | (NULL)              | (NULL) |
| 9   | 1003 | 9002    | 2021-09-07 10:01:01 | 2021-09-07 10:31:01 | 86     |
| 10  | 1003 | 9003    | 2021-09-08 12:01:01 | 2021-09-08 12:11:01 | 40     |
| 11  | 1003 | 9003    | 2021-09-01 13:01:01 | 2021-09-01 13:41:01 | 70     |
| 12  | 1003 | 9001    | 2021-09-08 14:01:01 | (NULL)              | (NULL) |
| 13  | 1003 | 9002    | 2021-09-08 15:01:01 | (NULL)              | (NULL) |
| 14  | 1005 | 9001    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 90     |
| 15  | 1005 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 88     |
| 16  | 1005 | 9002    | 2021-09-02 12:11:01 | 2021-09-02 12:31:01 | 89     |

Hãy tính số người dùng trên cấp độ 5 làm bài `uv` và điểm trung bình `avg_score` trong ngày phát hành của mỗi bài thi thuộc nhóm SQL. Sắp xếp giảm dần theo số người, nếu bằng nhau thì tăng dần theo điểm trung bình. Kết quả mẫu:

| exam_id | uv  | avg_score |
| ------- | --- | --------- |
| 9001    | 3   | 81.3      |

Giải thích: Chỉ có một bài thi thuộc nhóm SQL, ID bài thi là 9001. Trong ngày phát hành (2021-09-01), 1001, 1002, 1003, 1005 đều đã làm bài, nhưng 1003 là người dùng cấp độ 5, ba người còn lại trên cấp độ 5. Điểm của họ là [70,80,85,90], điểm trung bình là 81.3 (giữ một chữ số thập phân).

**Cách làm**: Bài này nhìn có vẻ phức tạp, nhưng chỉ cần lần lượt tách các điều kiện “bên ngoài”, rồi gộp lại là có đáp án. Với truy vấn nhiều bảng, hãy nhớ: đi từ ngoài vào trong, gỡ từng lớp.

Trước hết nối ba bảng và thêm một số điều kiện. Ví dụ đề yêu cầu người dùng có `cấp độ > 5`, có thể truy vấn trước như sau:

```sql
SELECT DISTINCT u_info.uid
FROM examination_info e_info
INNER JOIN exam_record record
INNER JOIN user_info u_info
WHERE e_info.exam_id = record.exam_id
  AND u_info.uid = record.uid
  AND u_info.LEVEL > 5
```

Tiếp theo chú ý yêu cầu: `người dùng làm bài trong ngày phát hành của từng bài thi thuộc nhóm SQL`. Từ khóa ==trong ngày== khiến ta cần nghĩ ngay đến so sánh thời gian.

So sánh ngày phát hành bài thi với ngày bắt đầu thi: `DATE(e_info.release_time) = DATE(record.start_time)`. Không cần lo `submit_time` là null, vì sẽ lọc trong where sau đó.

**Đáp án**:

```sql
SELECT record.exam_id AS exam_id,
       COUNT(DISTINCT u_info.uid) AS uv,
       ROUND(SUM(record.score) / COUNT(u_info.uid), 1) AS avg_score
FROM examination_info e_info
INNER JOIN exam_record record
INNER JOIN user_info u_info
WHERE e_info.exam_id = record.exam_id
  AND u_info.uid = record.uid
  AND DATE (e_info.release_time) = DATE (record.start_time)
  AND submit_time IS NOT NULL
  AND tag = 'SQL'
  AND u_info.LEVEL > 5
GROUP BY record.exam_id
ORDER BY uv DESC,
         avg_score ASC
```

Chú ý phần phân nhóm và sắp xếp cuối cùng: trước tiên sắp xếp theo số người, nếu bằng nhau thì sắp xếp theo điểm trung bình.

### Phân bố cấp độ người dùng của những người có điểm bài thi lớn hơn 80

**Mô tả**:

Có bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` nickname, `achievement` thành tích, `level` cấp độ, `job` hướng nghề nghiệp, `register_time` thời gian đăng ký):

| id  | uid  | nick_name  | achievement | level | job        | register_time       |
| --- | ---- | ---------- | ----------- | ----- | ---------- | ------------------- |
| 1   | 1001 | Nowcoder 1 | 3100        | 7     | Thuật toán | 2020-01-01 10:00:00 |
| 2   | 1002 | Nowcoder 2 | 2100        | 6     | Thuật toán | 2020-01-01 10:00:00 |
| 3   | 1003 | Nowcoder 3 | 1500        | 5     | Thuật toán | 2020-01-01 10:00:00 |
| 4   | 1004 | Nowcoder 4 | 1100        | 4     | Thuật toán | 2020-01-01 10:00:00 |
| 5   | 1005 | Nowcoder 5 | 1600        | 6     | C++        | 2020-01-01 10:00:00 |
| 6   | 1006 | Nowcoder 6 | 3000        | 6     | C++        | 2020-01-01 10:00:00 |

Bảng thông tin bài thi `examination_info` (`exam_id` ID bài thi, `tag` nhóm bài thi, `difficulty` độ khó bài thi, `duration` thời lượng thi, `release_time` thời gian phát hành):

| id  | exam_id | tag        | difficulty | duration | release_time        |
| --- | ------- | ---------- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL        | hard       | 60       | 2021-09-01 06:00:00 |
| 2   | 9002    | C++        | easy       | 60       | 2021-09-01 06:00:00 |
| 3   | 9003    | Thuật toán | medium     | 80       | 2021-09-01 10:00:00 |

Bảng thông tin lượt làm bài `exam_record` (`uid` ID người dùng, `exam_id` ID bài thi, `start_time` thời gian bắt đầu làm bài, `submit_time` thời gian nộp bài, `score` điểm):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2021-09-01 09:01:01 | 2021-09-01 09:41:01 | 79     |
| 2   | 1002 | 9003    | 2021-09-01 12:01:01 | 2021-09-01 12:21:01 | 60     |
| 3   | 1002 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 70     |
| 4   | 1002 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:40:01 | 80     |
| 5   | 1002 | 9003    | 2021-08-01 12:01:01 | 2021-08-01 12:21:01 | 60     |
| 6   | 1002 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 70     |
| 7   | 1002 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:40:01 | 85     |
| 8   | 1002 | 9002    | 2021-09-01 12:01:01 | (NULL)              | (NULL) |
| 9   | 1003 | 9002    | 2021-09-07 10:01:01 | 2021-09-07 10:31:01 | 86     |
| 10  | 1003 | 9003    | 2021-09-08 12:01:01 | 2021-09-08 12:11:01 | 40     |
| 11  | 1003 | 9003    | 2021-09-01 13:01:01 | 2021-09-01 13:41:01 | 81     |
| 12  | 1003 | 9001    | 2021-09-01 14:01:01 | (NULL)              | (NULL) |
| 13  | 1003 | 9002    | 2021-09-08 15:01:01 | (NULL)              | (NULL) |
| 14  | 1005 | 9001    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 90     |
| 15  | 1005 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 88     |
| 16  | 1005 | 9002    | 2021-09-02 12:11:01 | 2021-09-02 12:31:01 | 89     |

Hãy thống kê phân bố cấp độ người dùng của những người có điểm bài thi thuộc nhóm SQL lớn hơn 80, sắp xếp giảm dần theo số lượng (đảm bảo số lượng không trùng nhau). Kết quả mẫu:

| level | level_cnt |
| ----- | --------- |
| 6     | 2         |
| 5     | 1         |

Giải thích: 9001 là bài thi thuộc nhóm SQL. Những người có điểm lớn hơn 80 ở bài thi này gồm 1002, 1003, 1005; trong đó có hai người cấp độ 6 và một người cấp độ 5.

**Cách làm:** Dữ liệu của bài này giống bài trước, chỉ thay đổi điều kiện truy vấn. Hiểu bài trước thì bài này có thể giải quyết rất nhanh.

**Đáp án**:

```sql
SELECT u_info.LEVEL AS LEVEL,
       count(u_info.uid) AS level_cnt
FROM examination_info e_info
INNER JOIN exam_record record
INNER JOIN user_info u_info
WHERE e_info.exam_id = record.exam_id
  AND u_info.uid = record.uid
  AND record.score > 80
  AND submit_time IS NOT NULL
  AND tag = 'SQL'
GROUP BY LEVEL
ORDER BY level_cnt DESC
```

## Truy vấn hợp nhất

### Số người và số lượt làm của từng câu hỏi và từng bài thi

**Mô tả**:

Có bảng ghi lại lượt làm bài `exam_record` (uid ID người dùng, exam_id ID bài thi, start_time thời gian bắt đầu làm bài, submit_time thời gian nộp bài, score điểm):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2021-09-01 09:01:01 | 2021-09-01 09:41:01 | 81     |
| 2   | 1002 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 70     |
| 3   | 1002 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:40:01 | 80     |
| 4   | 1002 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 70     |
| 5   | 1004 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:40:01 | 85     |
| 6   | 1002 | 9002    | 2021-09-01 12:01:01 | (NULL)              | (NULL) |

Bảng luyện câu hỏi `practice_record` (uid ID người dùng, question_id ID câu hỏi, submit_time thời gian nộp, score điểm):

| id  | uid  | question_id | submit_time         | score |
| --- | ---- | ----------- | ------------------- | ----- |
| 1   | 1001 | 8001        | 2021-08-02 11:41:01 | 60    |
| 2   | 1002 | 8001        | 2021-09-02 19:30:01 | 50    |
| 3   | 1002 | 8001        | 2021-09-02 19:20:01 | 70    |
| 4   | 1002 | 8002        | 2021-09-02 19:38:01 | 70    |
| 5   | 1003 | 8001        | 2021-08-02 19:38:01 | 70    |
| 6   | 1003 | 8001        | 2021-08-02 19:48:01 | 90    |
| 7   | 1003 | 8002        | 2021-08-01 19:38:01 | 80    |

Hãy thống kê số người và số lượt làm của từng câu hỏi và từng bài thi, lần lượt hiển thị theo `uv` và `pv` của “bài thi” và “câu hỏi” theo thứ tự giảm dần. Kết quả mẫu:

| tid  | uv  | pv  |
| ---- | --- | --- |
| 9001 | 3   | 3   |
| 9002 | 1   | 3   |
| 8001 | 3   | 5   |
| 8002 | 2   | 2   |

**Giải thích**: “Bài thi” có 3 người làm bài 9001 tổng cộng 3 lượt, 1 người làm bài 9002 tổng cộng 3 lượt; “luyện câu hỏi” có 3 người luyện 8001 tổng cộng 5 lượt, 2 người luyện 8002 tổng cộng 2 lượt.

**Cách làm**: Điểm khó và dễ sai của bài này là vấn đề dùng đồng thời `UNION` và `ORDER BY`.

Có một số trường hợp sau: dùng `union` và nhiều `order by` mà không có dấu ngoặc sẽ báo lỗi!

`order by` trong mệnh đề được nối bằng `union` không có tác dụng.

Ví dụ không thêm dấu ngoặc:

```sql
SELECT exam_id AS tid,
       COUNT(DISTINCT UID) AS uv,
       COUNT(UID) AS pv
FROM exam_record
GROUP BY exam_id
ORDER BY uv DESC,
         pv DESC
UNION
SELECT question_id AS tid,
       COUNT(DISTINCT UID) AS uv,
       COUNT(UID) AS pv
FROM practice_record
GROUP BY question_id
ORDER BY uv DESC,
         pv DESC
```

Sẽ báo lỗi cú pháp ngay. Nếu không có dấu ngoặc thì chỉ được có một `order by`.

Một trường hợp khác là `order by` không có tác dụng, nhưng lại có tác dụng trong mệnh đề con. Cách giải quyết là bọc thêm một tầng truy vấn bên ngoài.

**Đáp án**:

```sql
SELECT *
FROM
  (SELECT exam_id AS tid,
          COUNT(DISTINCT exam_record.uid) uv,
          COUNT(*) pv
   FROM exam_record
   GROUP BY exam_id
   ORDER BY uv DESC, pv DESC) t1
UNION
SELECT *
FROM
  (SELECT question_id AS tid,
          COUNT(DISTINCT practice_record.uid) uv,
          COUNT(*) pv
   FROM practice_record
   GROUP BY question_id
   ORDER BY uv DESC, pv DESC) t2;
```

### Những người đồng thời thỏa mãn hai hoạt động

**Mô tả**: Để khuyến khích nhiều người dùng học tập và tiến bộ qua việc luyện câu hỏi trên nền tảng Nowcoder, chúng tôi thường tặng ưu đãi cho những người dùng vừa tích cực vừa có thành tích tốt. Trước đây có hai hoạt động vận hành: tặng phiếu ưu đãi cho những người luôn đạt ít nhất 85 điểm ở mỗi bài thi (activity1), và những người ít nhất một lần hoàn thành bài thi độ khó cao trong một nửa thời gian với điểm lớn hơn 80 (activity2).

Hiện cần lọc một lần những người thỏa mãn hai hoạt động này để giao cho nhân viên vận hành. Hãy viết một SQL: xuất ID và số hoạt động của những người trong năm 2021 luôn đạt ít nhất 85 điểm ở mỗi bài thi, hoặc ít nhất một lần hoàn thành bài thi độ khó cao trong một nửa thời gian với điểm lớn hơn 80; sắp xếp theo ID người dùng.

Có bảng thông tin bài thi `examination_info` (`exam_id` ID bài thi, `tag` nhóm bài thi, `difficulty` độ khó bài thi, `duration` thời lượng thi, `release_time` thời gian phát hành):

| id  | exam_id | tag        | difficulty | duration | release_time        |
| --- | ------- | ---------- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL        | hard       | 60       | 2021-09-01 06:00:00 |
| 2   | 9002    | C++        | easy       | 60       | 2021-09-01 06:00:00 |
| 3   | 9003    | Thuật toán | medium     | 80       | 2021-09-01 10:00:00 |

Bảng ghi lại lượt làm bài `exam_record` (`uid` ID người dùng, `exam_id` ID bài thi, `start_time` thời gian bắt đầu làm bài, `submit_time` thời gian nộp bài, `score` điểm):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2021-09-01 09:01:01 | 2021-09-01 09:31:00 | 81     |
| 2   | 1002 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 70     |
| 3   | 1003 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:40:01 | **86** |
| 4   | 1003 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 89     |
| 5   | 1004 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:30:01 | 85     |

Dữ liệu đầu ra mẫu:

| uid  | activity  |
| ---- | --------- |
| 1001 | activity2 |
| 1003 | activity1 |
| 1004 | activity1 |
| 1004 | activity2 |

**Giải thích**: Điểm thấp nhất của 1001 là 81, không thỏa hoạt động 1, nhưng người dùng hoàn thành bài thi dài 60 phút trong 29 phút 59 giây với điểm 81, nên thỏa hoạt động 2. Điểm thấp nhất của 1003 là 86, thỏa hoạt động 1; thời gian hoàn thành đều dài hơn một nửa thời lượng bài thi nên không thỏa hoạt động 2. Người dùng 1004 vừa đúng dùng 30 phút để hoàn thành bài thi, đạt 85 điểm, nên thỏa cả hoạt động 1 và 2.

**Cách làm**: Bài này cần phép trừ thời gian, dùng hàm `TIMESTAMPDIFF()` để tính chênh lệch phút giữa hai timestamp.

Cách dùng cơ bản:

Ví dụ:

```sql
TIMESTAMPDIFF(MINUTE, start_time, end_time)
```

Tham số đầu tiên của hàm `TIMESTAMPDIFF()` là đơn vị thời gian. Ở đây chọn `MINUTE` để trả về chênh lệch phút. Tham số thứ hai là timestamp sớm hơn, tham số thứ ba là timestamp muộn hơn. Hàm trả về chênh lệch phút giữa chúng.

Sau khi biết cách dùng hàm này, quay lại yêu cầu của `activity1`: chỉ cần tìm điểm lớn hơn hoặc bằng 85. Trước hết viết phần này để logic sau đó rõ ràng hơn:

```sql
SELECT DISTINCT UID
FROM exam_record
WHERE score >= 85
  AND YEAR (start_time) = '2021'
```

Theo điều kiện 2, tiếp tục viết phần `người hoàn thành bài thi độ khó cao trong một nửa thời gian và có điểm lớn hơn 80`:

```sql
SELECT UID
FROM examination_info info
INNER JOIN exam_record record
WHERE info.exam_id = record.exam_id
  AND (TIMESTAMPDIFF(MINUTE, start_time, submit_time)) < (info.duration / 2)
  AND difficulty = 'hard'
  AND score >= 80
```

Sau đó chỉ cần `UNION` hai phần lại. (Đặc biệt chú ý vấn đề dấu ngoặc và vị trí của `order by`; cách dùng cụ thể đã nói ở phần trước.)

**Đáp án**:

```sql
SELECT DISTINCT UID UID,
                    'activity1' activity
FROM exam_record
WHERE UID not in
    (SELECT UID
     FROM exam_record
     WHERE score<85
       AND YEAR(submit_time) = 2021 )
UNION
SELECT DISTINCT UID UID,
                    'activity2' activity
FROM exam_record e_r
LEFT JOIN examination_info e_i ON e_r.exam_id = e_i.exam_id
WHERE YEAR(submit_time) = 2021
  AND difficulty = 'hard'
  AND TIMESTAMPDIFF(SECOND, start_time, submit_time) <= duration *30
  AND score>80
ORDER BY UID
```

## Truy vấn liên kết

### Số bài thi hoàn thành và số câu hỏi luyện tập của người dùng thỏa điều kiện (khó)

**Mô tả**:

Có bảng thông tin người dùng `user_info` (uid ID người dùng, nick_name nickname, achievement thành tích, level cấp độ, job hướng nghề nghiệp, register_time thời gian đăng ký):

| id  | uid  | nick_name  | achievement | level | job        | register_time       |
| --- | ---- | ---------- | ----------- | ----- | ---------- | ------------------- |
| 1   | 1001 | Nowcoder 1 | 3100        | 7     | Thuật toán | 2020-01-01 10:00:00 |
| 2   | 1002 | Nowcoder 2 | 2300        | 7     | Thuật toán | 2020-01-01 10:00:00 |
| 3   | 1003 | Nowcoder 3 | 2500        | 7     | Thuật toán | 2020-01-01 10:00:00 |
| 4   | 1004 | Nowcoder 4 | 1200        | 5     | Thuật toán | 2020-01-01 10:00:00 |
| 5   | 1005 | Nowcoder 5 | 1600        | 6     | C++        | 2020-01-01 10:00:00 |
| 6   | 1006 | Nowcoder 6 | 2000        | 6     | C++        | 2020-01-01 10:00:00 |

Bảng thông tin bài thi `examination_info` (exam_id ID bài thi, tag nhóm bài thi, difficulty độ khó bài thi, duration thời lượng thi, release_time thời gian phát hành):

| id  | exam_id | tag        | difficulty | duration | release_time        |
| --- | ------- | ---------- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL        | hard       | 60       | 2021-09-01 06:00:00 |
| 2   | 9002    | C++        | hard       | 60       | 2021-09-01 06:00:00 |
| 3   | 9003    | Thuật toán | medium     | 80       | 2021-09-01 10:00:00 |

Bảng ghi lại lượt làm bài `exam_record` (uid ID người dùng, exam_id ID bài thi, start_time thời gian bắt đầu làm bài, submit_time thời gian nộp bài, score điểm):

| id  | uid  | exam_id | start_time          | submit_time         | score |
| --- | ---- | ------- | ------------------- | ------------------- | ----- |
| 1   | 1001 | 9001    | 2021-09-01 09:01:01 | 2021-09-01 09:31:00 | 81    |
| 2   | 1002 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 81    |
| 3   | 1003 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:40:01 | 86    |
| 4   | 1003 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:51 | 89    |
| 5   | 1004 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:30:01 | 85    |
| 6   | 1005 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:02 | 85    |
| 7   | 1006 | 9003    | 2021-09-07 10:01:01 | 2021-09-07 10:21:01 | 84    |
| 8   | 1006 | 9001    | 2021-09-07 10:01:01 | 2021-09-07 10:21:01 | 80    |

Bảng ghi lại việc luyện câu hỏi `practice_record` (uid ID người dùng, question_id ID câu hỏi, submit_time thời gian nộp, score điểm):

| id  | uid  | question_id | submit_time         | score |
| --- | ---- | ----------- | ------------------- | ----- |
| 1   | 1001 | 8001        | 2021-08-02 11:41:01 | 60    |
| 2   | 1002 | 8001        | 2021-09-02 19:30:01 | 50    |
| 3   | 1002 | 8001        | 2021-09-02 19:20:01 | 70    |
| 4   | 1002 | 8002        | 2021-09-02 19:38:01 | 70    |
| 5   | 1004 | 8001        | 2021-08-02 19:38:01 | 70    |
| 6   | 1004 | 8002        | 2021-08-02 19:48:01 | 90    |
| 7   | 1001 | 8002        | 2021-08-02 19:38:01 | 70    |
| 8   | 1004 | 8002        | 2021-08-02 19:48:01 | 90    |
| 9   | 1004 | 8002        | 2021-08-02 19:58:01 | 94    |
| 10  | 1004 | 8003        | 2021-08-02 19:38:01 | 70    |
| 11  | 1004 | 8003        | 2021-08-02 19:48:01 | 90    |
| 12  | 1004 | 8003        | 2021-08-01 19:38:01 | 80    |

Hãy tìm những người dùng nổi bật cấp độ 7, có điểm trung bình bài thi SQL độ khó cao lớn hơn 80; thống kê tổng số lượt hoàn thành bài thi và tổng số lượt luyện câu hỏi trong năm 2021 của họ, chỉ giữ những người có bản ghi hoàn thành bài thi trong năm 2021. Kết quả sắp xếp tăng dần theo số bài thi hoàn thành, giảm dần theo số câu hỏi luyện tập.

Dữ liệu đầu ra mẫu:

| uid  | exam_cnt | question_cnt |
| ---- | -------- | ------------ |
| 1001 | 1        | 2            |
| 1003 | 2        | 0            |

Giải thích: Người dùng 1001, 1003, 1004, 1006 thỏa điểm trung bình bài thi SQL độ khó cao lớn hơn 80, nhưng chỉ 1001 và 1003 là người dùng nổi bật cấp độ 7. 1001 hoàn thành 1 lượt bài thi 9001 và luyện 2 lượt câu hỏi; 1003 hoàn thành 2 lượt bài thi 9001, 9002 nhưng không luyện câu hỏi nào (vì vậy số đếm là 0).

**Cách làm:**

Trước hết lọc sơ bộ các điều kiện, ví dụ tìm những người dùng đã làm bài thi SQL độ khó cao:

```sql
SELECT
\trecord.uid
FROM
\texam_record record
\tINNER JOIN examination_info e_info ON record.exam_id = e_info.exam_id
\tJOIN user_info u_info ON record.uid = u_info.uid
WHERE
\te_info.tag = 'SQL'
\tAND e_info.difficulty = 'hard'
```

Sau đó tiếp tục thêm các điều kiện theo yêu cầu đề bài.

Nhưng ở đây cần chú ý thêm:

Thứ nhất: Không được đặt điều kiện `YEAR(submit_time)= 2021` ở cuối, mà phải đặt trong điều kiện `ON`, vì liên kết trái vẫn trả về toàn bộ dòng của bảng trái, trong đó bảng phải có thể là null. Đặt trong mệnh đề `ON` của điều kiện `JOIN` nhằm bảo đảm khi liên kết hai bảng, chỉ các bản ghi thỏa điều kiện năm mới được liên kết. Như vậy tránh đưa bản ghi của các năm khác vào kết quả. Ví dụ 1001 từng làm bài thi trong năm 2021 nhưng chưa luyện câu hỏi; nếu đặt điều kiện ở cuối sẽ loại người dùng này.

Thứ hai: Phải dùng `COUNT(distinct er.exam_id) exam_cnt, COUNT(distinct pr.id) question_cnt`, cần thêm distinct vì liên kết trái tạo ra nhiều giá trị trùng lặp.

**Đáp án**:

```sql
SELECT er.uid AS UID,
       count(DISTINCT er.exam_id) AS exam_cnt,
       count(DISTINCT pr.id) AS question_cnt
FROM exam_record er
LEFT JOIN practice_record pr ON er.uid = pr.uid
AND YEAR (er.submit_time)= 2021
AND YEAR (pr.submit_time)= 2021
WHERE er.uid IN
    (SELECT er.uid
     FROM exam_record er
     LEFT JOIN examination_info ei ON er.exam_id = ei.exam_id
     LEFT JOIN user_info ui ON er.uid = ui.uid
     WHERE tag = 'SQL'
       AND difficulty = 'hard'
       AND LEVEL = 7
     GROUP BY er.uid
     HAVING avg(score) > 80)
GROUP BY er.uid
ORDER BY exam_cnt,
         question_cnt DESC
```

Có thể bạn sẽ thắc mắc: rõ ràng đã giới hạn điều kiện `tag = 'SQL' AND difficulty = 'hard'`, tại sao người dùng 1003 vẫn truy vấn được hai bản ghi thi, trong đó một bản ghi có `tag` là `C++`? Đó là do đặc điểm của `LEFT JOIN`: kể cả khi không có dòng khớp ở bảng phải, mọi bản ghi của bảng trái vẫn được giữ lại.

### Tình hình hoạt động của từng người dùng cấp độ 6/7 (khó)

**Mô tả**:

Có bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` nickname, `achievement` thành tích, `level` cấp độ, `job` hướng nghề nghiệp, `register_time` thời gian đăng ký):

| id  | uid  | nick_name  | achievement | level | job        | register_time       |
| --- | ---- | ---------- | ----------- | ----- | ---------- | ------------------- |
| 1   | 1001 | Nowcoder 1 | 3100        | 7     | Thuật toán | 2020-01-01 10:00:00 |
| 2   | 1002 | Nowcoder 2 | 2300        | 7     | Thuật toán | 2020-01-01 10:00:00 |
| 3   | 1003 | Nowcoder 3 | 2500        | 7     | Thuật toán | 2020-01-01 10:00:00 |
| 4   | 1004 | Nowcoder 4 | 1200        | 5     | Thuật toán | 2020-01-01 10:00:00 |
| 5   | 1005 | Nowcoder 5 | 1600        | 6     | C++        | 2020-01-01 10:00:00 |
| 6   | 1006 | Nowcoder 6 | 2600        | 7     | C++        | 2020-01-01 10:00:00 |

Bảng thông tin bài thi `examination_info` (`exam_id` ID bài thi, `tag` nhóm bài thi, `difficulty` độ khó bài thi, `duration` thời lượng thi, `release_time` thời gian phát hành):

| id  | exam_id | tag        | difficulty | duration | release_time        |
| --- | ------- | ---------- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL        | hard       | 60       | 2021-09-01 06:00:00 |
| 2   | 9002    | C++        | easy       | 60       | 2021-09-01 06:00:00 |
| 3   | 9003    | Thuật toán | medium     | 80       | 2021-09-01 10:00:00 |

Bảng ghi lại lượt làm bài `exam_record` (`uid` ID người dùng, `exam_id` ID bài thi, `start_time` thời gian bắt đầu làm bài, `submit_time` thời gian nộp bài, `score` điểm):

| uid  | exam_id | start_time          | submit_time         | score  |
| ---- | ------- | ------------------- | ------------------- | ------ |
| 1001 | 9001    | 2021-09-01 09:01:01 | 2021-09-01 09:31:00 | 78     |
| 1001 | 9001    | 2021-09-01 09:01:01 | 2021-09-01 09:31:00 | 81     |
| 1005 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:30:01 | 85     |
| 1005 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:02 | 85     |
| 1006 | 9003    | 2021-09-07 10:01:01 | 2021-09-07 10:21:59 | 84     |
| 1006 | 9001    | 2021-09-07 10:01:01 | 2021-09-07 10:21:01 | 81     |
| 1002 | 9001    | 2020-09-01 13:01:01 | 2020-09-01 13:41:01 | 81     |
| 1005 | 9001    | 2021-09-01 14:01:01 | (NULL)              | (NULL) |

Bảng ghi lại việc luyện câu hỏi `practice_record` (`uid` ID người dùng, `question_id` ID câu hỏi, `submit_time` thời gian nộp, `score` điểm):

| uid  | question_id | submit_time         | score |
| ---- | ----------- | ------------------- | ----- |
| 1001 | 8001        | 2021-08-02 11:41:01 | 60    |
| 1004 | 8001        | 2021-08-02 19:38:01 | 70    |
| 1004 | 8002        | 2021-08-02 19:48:01 | 90    |
| 1001 | 8002        | 2021-08-02 19:38:01 | 70    |
| 1004 | 8002        | 2021-08-02 19:48:01 | 90    |
| 1006 | 8002        | 2021-08-04 19:58:01 | 94    |
| 1006 | 8003        | 2021-08-03 19:38:01 | 70    |
| 1006 | 8003        | 2021-08-02 19:48:01 | 90    |
| 1006 | 8003        | 2020-08-01 19:38:01 | 80    |

Hãy thống kê tổng số tháng hoạt động, số ngày hoạt động năm 2021, số ngày hoạt động làm bài thi năm 2021 và số ngày hoạt động luyện câu hỏi năm 2021 của từng người dùng cấp độ 6/7. Sắp xếp giảm dần theo tổng số tháng hoạt động và số ngày hoạt động năm 2021. Kết quả mẫu:

| uid  | act_month_total | act_days_2021 | act_days_2021_exam |
| ---- | --------------- | ------------- | ------------------ |
| 1006 | 3               | 4             | 1                  |
| 1001 | 2               | 2             | 1                  |
| 1005 | 1               | 1             | 1                  |
| 1002 | 1               | 0             | 0                  |
| 1003 | 0               | 0             | 0                  |

**Giải thích**: Có tổng cộng 5 người dùng cấp độ 6/7. 1006 hoạt động trong ba tháng 202109, 202108, 202008; số ngày hoạt động năm 2021 là 20210907, 20210804, 20210803, 20210802, tổng cộng 4 ngày; trong khu vực làm bài thi năm 2021 hoạt động 1 ngày là 20210907, còn trong khu vực luyện câu hỏi hoạt động 3 ngày.

**Cách làm:**

Điểm mấu chốt của bài này là cách dùng `CASE WHEN THEN`, nếu không sẽ phải viết rất nhiều `left join` vì tạo ra nhiều tập kết quả.

Câu lệnh `CASE WHEN THEN` là một biểu thức điều kiện, dùng để thực hiện thao tác khác nhau hoặc trả về kết quả khác nhau trong SQL tùy theo điều kiện.

Cấu trúc cú pháp:

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ...
    ELSE result
END
```

Trong cấu trúc này có thể thêm nhiều mệnh đề `WHEN` tùy nhu cầu. Sau mỗi mệnh đề `WHEN` là một điều kiện (`condition`) và một kết quả (`result`). Điều kiện có thể là bất kỳ biểu thức logic nào; nếu thỏa điều kiện, kết quả tương ứng sẽ được trả về.

Mệnh đề `ELSE` cuối cùng là tùy chọn, dùng để chỉ định kết quả mặc định khi tất cả điều kiện trước đó đều không thỏa. Nếu không cung cấp mệnh đề `ELSE`, mặc định sẽ trả về `NULL`.

Ví dụ:

```sql
SELECT score,
    CASE
        WHEN score >= 90 THEN 'Xuất sắc'
        WHEN score >= 80 THEN 'Tốt'
        WHEN score >= 60 THEN 'Đạt'
        ELSE 'Không đạt'
    END AS grade
FROM student_scores;
```

Trong ví dụ trên, câu lệnh CASE WHEN THEN trả về cấp độ (`grade`) tùy theo khoảng điểm (`score`) của học sinh. Nếu điểm lớn hơn hoặc bằng 90 thì trả về "Xuất sắc"; nếu điểm lớn hơn hoặc bằng 80 thì trả về "Tốt"; nếu điểm lớn hơn hoặc bằng 60 thì trả về "Đạt"; ngược lại trả về "Không đạt".

Sau khi hiểu cách dùng trên, quay lại bài này: cần liệt kê các số ngày hoạt động khác nhau.

```sql
count(distinct act_month) as act_month_total,
count(distinct case when year(act_time)='2021'then act_day end) as act_days_2021,
count(distinct case when year(act_time)='2021' and tag='exam' then act_day end) as act_days_2021_exam,
count(distinct case when year(act_time)='2021' and tag='question'then act_day end) as act_days_2021_question
```

Ở đây `tag` là nhãn được gán trước để phân biệt truy vấn, tách khu vực thi và khu vực luyện câu hỏi.

Tìm người dùng trong khu vực làm bài thi:

```sql
SELECT
\t\tuid,
\t\texam_id AS ans_id,
\t\tstart_time AS act_time,
\t\tdate_format( start_time, '%Y%m' ) AS act_month,
\t\tdate_format( start_time, '%Y%m%d' ) AS act_day,
\t\t'exam' AS tag
\tFROM
\t\texam_record
```

Tiếp theo là người dùng trong khu vực luyện câu hỏi:

```sql
SELECT
\t\tuid,
\t\tquestion_id AS ans_id,
\t\tsubmit_time AS act_time,
\t\tdate_format( submit_time, '%Y%m' ) AS act_month,
\t\tdate_format( submit_time, '%Y%m%d' ) AS act_day,
\t\t'question' AS tag
\tFROM
\t\tpractice_record
```

Cuối cùng hợp nhất hai kết quả bằng `UNION`, rồi đừng quên sắp xếp kết quả (bài này hơi giống tư tưởng chia để trị).

**Đáp án**:

```sql
SELECT user_info.uid,
       count(DISTINCT act_month) AS act_month_total,
       count(DISTINCT CASE
                          WHEN YEAR (act_time)= '2021' THEN act_day
                      END) AS act_days_2021,
       count(DISTINCT CASE
                          WHEN YEAR (act_time)= '2021'
                               AND tag = 'exam' THEN act_day
                      END) AS act_days_2021_exam,
       count(DISTINCT CASE
                          WHEN YEAR (act_time)= '2021'
                               AND tag = 'question' THEN act_day
                      END) AS act_days_2021_question
FROM
  (SELECT UID,
          exam_id AS ans_id,
          start_time AS act_time,
          date_format(start_time, '%Y%m') AS act_month,
          date_format(start_time, '%Y%m%d') AS act_day,
          'exam' AS tag
   FROM exam_record
   UNION ALL SELECT UID,
                    question_id AS ans_id,
                    submit_time AS act_time,
                    date_format(submit_time, '%Y%m') AS act_month,
                    date_format(submit_time, '%Y%m%d') AS act_day,
                    'question' AS tag
   FROM practice_record) total
RIGHT JOIN user_info ON total.uid = user_info.uid
WHERE user_info.LEVEL IN (6,
                          7)
GROUP BY user_info.uid
ORDER BY act_month_total DESC,
         act_days_2021 DESC
```

<!-- @include: @article-footer.snippet.md -->
