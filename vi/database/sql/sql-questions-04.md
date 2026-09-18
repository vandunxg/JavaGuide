---
title: "Tổng hợp câu hỏi phỏng vấn SQL thường gặp (4)"
description: "Phần bốn tổng hợp câu hỏi phỏng vấn SQL thường gặp, giải thích chi tiết cách dùng và trường hợp sử dụng của các window function ROW_NUMBER, RANK, DENSE_RANK, NTILE, LAG, LEAD trong MySQL 8.0."
category: Database
tag:
  - Database Basics
  - SQL
head:
  - - meta
    - name: keywords
      content: "câu hỏi phỏng vấn SQL, window function, ROW_NUMBER, RANK, DENSE_RANK, NTILE, LAG, LEAD, MySQL 8.0"
---

> Nguồn đề bài: [Nowcoder - Thử thách SQL nâng cao](https://www.nowcoder.com/exam/oj?page=1&tab=SQL%E7%AF%87&topicId=240)

Bạn có thể quyết định có bỏ qua các câu hỏi tương đối khó hoặc khó hay không, tùy theo tình hình thực tế và nhu cầu phỏng vấn của bản thân.

## Window function chuyên dụng

MySQL 8.0 bổ sung hỗ trợ window function. Dưới đây là các window function thường gặp trong MySQL và cách sử dụng:

1. `ROW_NUMBER()`: Phân bổ một giá trị số nguyên duy nhất cho mỗi dòng trong tập kết quả truy vấn.

```sql
SELECT col1, col2, ROW_NUMBER() OVER (ORDER BY col1) AS row_num
FROM table;
```

2. `RANK()`: Tính thứ hạng của mỗi dòng trong kết quả sắp xếp.

```sql
SELECT col1, col2, RANK() OVER (ORDER BY col1 DESC) AS ranking
FROM table;
```

3. `DENSE_RANK()`: Tính thứ hạng của mỗi dòng trong kết quả sắp xếp và giữ nguyên các thứ hạng giống nhau.

```sql
SELECT col1, col2, DENSE_RANK() OVER (ORDER BY col1 DESC) AS ranking
FROM table;
```

4. `NTILE(n)`: Chia kết quả thành n bucket tương đối đồng đều và phân bổ một mã định danh cho mỗi bucket.

```sql
SELECT col1, col2, NTILE(4) OVER (ORDER BY col1) AS bucket
FROM table;
```

5. `SUM()`, `AVG()`, `COUNT()`, `MIN()`, `MAX()`: Các hàm aggregate này cũng có thể kết hợp với window function để tính tổng, giá trị trung bình, số lượng, giá trị nhỏ nhất và lớn nhất của cột được chỉ định trong window.

```sql
SELECT col1, col2, SUM(col1) OVER () AS sum_col
FROM table;
```

6. `LEAD()` và `LAG()`: Hàm LEAD dùng để lấy giá trị của dòng cách dòng hiện tại một offset về sau, còn hàm LAG dùng để lấy giá trị của dòng cách dòng hiện tại một offset về trước.

```sql
SELECT col1, col2, LEAD(col1, 1) OVER (ORDER BY col1) AS next_col1,
                  LAG(col1, 1) OVER (ORDER BY col1) AS prev_col1
FROM table;
```

7. `FIRST_VALUE()` và `LAST_VALUE()`: Hàm FIRST_VALUE dùng để lấy giá trị đầu tiên của cột được chỉ định trong window, còn hàm LAST_VALUE dùng để lấy giá trị cuối cùng của cột được chỉ định trong window.

```sql
SELECT col1, col2, FIRST_VALUE(col2) OVER (PARTITION BY col1 ORDER BY col2) AS first_val,
                  LAST_VALUE(col2) OVER (PARTITION BY col1 ORDER BY col2) AS last_val
FROM table;
```

Window function thường cần kết hợp với mệnh đề OVER để định nghĩa kích thước window, quy tắc sắp xếp và cách phân nhóm.

### Ba người đứng đầu về điểm trong mỗi loại đề thi

**Mô tả**:

Hiện có bảng thông tin đề thi `examination_info` (`exam_id` ID đề thi, `tag` loại đề thi, `difficulty` độ khó đề thi, `duration` thời lượng thi, `release_time` thời gian phát hành):

| id  | exam_id | tag       | difficulty | duration | release_time        |
| --- | ------- | --------- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL       | hard       | 60       | 2021-09-01 06:00:00 |
| 2   | 9002    | SQL       | hard       | 60       | 2021-09-01 06:00:00 |
| 3   | 9003    | Algorithm | medium     | 80       | 2021-09-01 10:00:00 |

Bảng ghi chép trả lời đề thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` thời gian bắt đầu trả lời, `submit_time` thời gian nộp bài, score điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2021-09-01 09:01:01 | 2021-09-01 09:31:00 | 78     |
| 2   | 1002 | 9001    | 2021-09-01 09:01:01 | 2021-09-01 09:31:00 | 81     |
| 3   | 1002 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 81     |
| 4   | 1003 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:40:01 | 86     |
| 5   | 1003 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:51 | 89     |
| 6   | 1004 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:30:01 | 85     |
| 7   | 1005 | 9003    | 2021-09-01 12:01:01 | 2021-09-01 12:31:02 | 85     |
| 8   | 1006 | 9003    | 2021-09-07 10:01:01 | 2021-09-07 10:21:01 | 84     |
| 9   | 1003 | 9003    | 2021-09-08 12:01:01 | 2021-09-08 12:11:01 | 40     |
| 10  | 1003 | 9002    | 2021-09-01 14:01:01 | (NULL)              | (NULL) |

Tìm 3 người có điểm cao nhất trong mỗi loại đề thi. Nếu điểm cao nhất của hai người giống nhau, chọn người có điểm thấp nhất lớn hơn; nếu vẫn giống nhau thì chọn người có uid lớn hơn. Kết quả theo dữ liệu mẫu như sau:

| tid       | uid  | ranking |
| --------- | ---- | ------- |
| SQL       | 1003 | 1       |
| SQL       | 1004 | 2       |
| SQL       | 1002 | 3       |
| Algorithm | 1005 | 1       |
| Algorithm | 1006 | 2       |
| Algorithm | 1003 | 3       |

**Giải thích**: Các đề thi có bản ghi điểm trả lời là SQL và Algorithm. Với đề SQL, người dùng 1001, 1002, 1003, 1004 có điểm trả lời; điểm cao nhất lần lượt là 81, 81, 89, 85, điểm thấp nhất lần lượt là 78, 81, 86, 40. Vì vậy, xếp hạng trước theo điểm cao nhất, sau đó theo điểm thấp nhất, rồi lấy ba người đầu là 1003, 1004, 1002.

**Đáp án**:

```sql
SELECT tag,
       UID,
       ranking
FROM
  (SELECT b.tag AS tag,
          a.uid AS UID,
          ROW_NUMBER() OVER (PARTITION BY b.tag
                             ORDER BY b.tag,
                                      max(a.score) DESC,
                                      min(a.score) DESC,
                                      a.uid DESC) AS ranking
   FROM exam_record a
   LEFT JOIN examination_info b ON a.exam_id = b.exam_id
   GROUP BY b.tag,
            a.uid) t
WHERE ranking <= 3
```

### Độ chênh thời gian của lần làm nhanh thứ hai và chậm thứ hai lớn hơn một nửa thời lượng đề thi (khá khó)

**Mô tả**:

Hiện có bảng thông tin đề thi `examination_info` (`exam_id` ID đề thi, `tag` loại đề thi, `difficulty` độ khó đề thi, `duration` thời lượng thi, `release_time` thời gian phát hành):

| id  | exam_id | tag       | difficulty | duration | release_time        |
| --- | ------- | --------- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL       | hard       | 60       | 2021-09-01 06:00:00 |
| 2   | 9002    | C++       | hard       | 60       | 2021-09-01 06:00:00 |
| 3   | 9003    | Algorithm | medium     | 80       | 2021-09-01 10:00:00 |

Bảng ghi chép trả lời đề thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` thời gian bắt đầu trả lời, `submit_time` thời gian nộp bài, `score` điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2021-09-01 09:01:01 | 2021-09-01 09:51:01 | 78     |
| 2   | 1001 | 9002    | 2021-09-01 09:01:01 | 2021-09-01 09:31:00 | 81     |
| 3   | 1002 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 81     |
| 4   | 1003 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:59:01 | 86     |
| 5   | 1003 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:51 | 89     |
| 6   | 1004 | 9002    | 2021-09-01 19:01:01 | 2021-09-01 19:30:01 | 85     |
| 7   | 1005 | 9001    | 2021-09-01 12:01:01 | 2021-09-01 12:31:02 | 85     |
| 8   | 1006 | 9001    | 2021-09-07 10:02:01 | 2021-09-07 10:21:01 | 84     |
| 9   | 1003 | 9001    | 2021-09-08 12:01:01 | 2021-09-08 12:11:01 | 40     |
| 10  | 1003 | 9002    | 2021-09-01 14:01:01 | (NULL)              | (NULL) |
| 11  | 1005 | 9001    | 2021-09-01 14:01:01 | (NULL)              | (NULL) |
| 12  | 1003 | 9003    | 2021-09-08 15:01:01 | (NULL)              | (NULL) |

Tìm thông tin các đề thi có độ chênh giữa thời gian làm nhanh thứ hai và chậm thứ hai lớn hơn một nửa thời lượng đề thi, sắp xếp theo ID đề thi giảm dần. Kết quả theo dữ liệu mẫu như sau:

| exam_id | duration | release_time        |
| ------- | -------- | ------------------- |
| 9001    | 60       | 2021-09-01 06:00:00 |

**Giải thích**: Thời gian làm đề 9001 lần lượt là 50 phút, 58 phút, 30 phút 1 giây, 19 phút, 10 phút. Độ chênh giữa thời gian nhanh thứ hai và chậm thứ hai là 50 phút - 19 phút = 31 phút. Thời lượng đề thi là 60 phút, nên thỏa mãn điều kiện lớn hơn một nửa thời lượng đề thi. Kết quả gồm ID đề thi, thời lượng và thời gian phát hành.

**Cách làm:**

Bước đầu tiên, tìm thứ hạng xuôi và ngược của thời gian hoàn thành mỗi đề thi, tức là bảng a;

Bước thứ hai, tạo inner join với bảng thông tin đề thi b, nhóm theo ID đề thi, dùng `having` để lọc dữ liệu có thứ hạng thứ hai, chuyển giây thành phút để so sánh, cuối cùng sắp xếp giảm dần theo ID đề thi.

**Đáp án**:

```sql
SELECT a.exam_id,
       b.duration,
       b.release_time
FROM
  (SELECT exam_id,
          row_number() OVER (PARTITION BY exam_id
                             ORDER BY timestampdiff(SECOND, start_time, submit_time) DESC) rn1,
          row_number() OVER (PARTITION BY exam_id
                            ORDER BY timestampdiff(SECOND, start_time, submit_time) ASC) rn2,
                                              timestampdiff(SECOND, start_time, submit_time) timex
   FROM exam_record
   WHERE score IS NOT NULL ) a
INNER JOIN examination_info b ON a.exam_id = b.exam_id
GROUP BY a.exam_id
HAVING (max(IF (rn1 = 2, a.timex, 0))- max(IF (rn2 = 2, a.timex, 0)))/ 60 > b.duration / 2
ORDER BY a.exam_id DESC
```

### Window thời gian lớn nhất của hai lần trả lời đề thi liên tiếp (khá khó)

**Mô tả**

Hiện có bảng ghi chép trả lời đề thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` thời gian bắt đầu trả lời, `submit_time` thời gian nộp bài, `score` điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score |
| --- | ---- | ------- | ------------------- | ------------------- | ----- |
| 1   | 1006 | 9003    | 2021-09-07 10:01:01 | 2021-09-07 10:21:02 | 84    |
| 2   | 1006 | 9001    | 2021-09-01 12:11:01 | 2021-09-01 12:31:01 | 89    |
| 3   | 1006 | 9002    | 2021-09-06 10:01:01 | 2021-09-06 10:21:01 | 81    |
| 4   | 1005 | 9002    | 2021-09-05 10:01:01 | 2021-09-05 10:21:01 | 81    |
| 5   | 1005 | 9001    | 2021-09-05 10:31:01 | 2021-09-05 10:51:01 | 81    |

Trong số những người đã làm đề ít nhất hai ngày trong năm 2021, hãy tính window thời gian lớn nhất giữa hai lần làm đề liên tiếp trong năm đó là `days_window`, sau đó dựa trên lịch sử của năm đó tính trung bình người đó sẽ làm bao nhiêu đề trong `days_window` ngày. Sắp xếp giảm dần theo window thời gian lớn nhất và số đề trung bình. Kết quả theo dữ liệu mẫu như sau:

| uid  | days_window | avg_exam_cnt |
| ---- | ----------- | ------------ |
| 1006 | 6           | 2.57         |

**Giải thích**: Người dùng 1006 lần lượt làm đề vào các ngày 20210901, 20210906, 20210907. Window thời gian lớn nhất giữa hai lần làm liên tiếp là 6 ngày (từ ngày 1 đến ngày 6). Trong 7 ngày từ ngày 1 đến ngày 7, người này làm tổng cộng 3 đề, trung bình mỗi ngày làm 3/7 = 0.428571 đề, nên trong 6 ngày trung bình sẽ làm 0.428571 \* 6 = 2.57 đề (giữ lại hai chữ số thập phân); người dùng 1005 làm hai đề trong ngày 20210905 nhưng chỉ có bản ghi trả lời của một ngày nên bị lọc.

**Cách làm:**

Phần giải thích trên gợi ý rằng cần loại trùng các bản ghi trả lời, nhưng tuyệt đối không được loại trùng! Loại trùng sẽ không vượt qua test case. Lưu ý giới hạn thời gian là năm 2021;

Ngoài ra, cần chú ý thời gian chênh lệch phải cộng thêm 1 ngày; đồng thời cần lưu ý rằng ==chưa nộp bài cũng được tính==!!!! (Nói chung cảm giác đề này mô tả không rõ, chất lượng ra đề không tốt lắm.)

**Đáp án**:

```sql
SELECT UID,
       max(datediff(next_time, start_time)) + 1 AS days_window,
       round(count(start_time)/(datediff(max(start_time), min(start_time))+ 1) * (max(datediff(next_time, start_time))+ 1), 2) AS avg_exam_cnt
FROM
  (SELECT UID,
          start_time,
          lead(start_time, 1) OVER (PARTITION BY UID
                                    ORDER BY start_time) AS next_time
   FROM exam_record
   WHERE YEAR (start_time) = '2021' ) a
GROUP BY UID
HAVING count(DISTINCT date(start_time)) > 1
ORDER BY days_window DESC,
         avg_exam_cnt DESC
```

### Tình hình hoàn thành của người dùng không có bài chưa hoàn thành trong ba tháng gần nhất

**Mô tả**:

Hiện có bảng ghi chép trả lời đề thi `exam_record` (`uid`: ID người dùng, `exam_id`: ID đề thi, `start_time`: thời gian bắt đầu trả lời, `submit_time`: thời gian nộp bài, nếu trống thì nghĩa là chưa hoàn thành, `score`: điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1006 | 9003    | 2021-09-06 10:01:01 | 2021-09-06 10:21:02 | 84     |
| 2   | 1006 | 9001    | 2021-08-02 12:11:01 | 2021-08-02 12:31:01 | 89     |
| 3   | 1006 | 9002    | 2021-06-06 10:01:01 | 2021-06-06 10:21:01 | 81     |
| 4   | 1006 | 9002    | 2021-05-06 10:01:01 | 2021-05-06 10:21:01 | 81     |
| 5   | 1006 | 9001    | 2021-05-01 12:01:01 | (NULL)              | (NULL) |
| 6   | 1001 | 9001    | 2021-09-05 10:31:01 | 2021-09-05 10:51:01 | 81     |
| 7   | 1001 | 9003    | 2021-08-01 09:01:01 | 2021-08-01 09:51:11 | 78     |
| 8   | 1001 | 9002    | 2021-07-01 09:01:01 | 2021-07-01 09:31:00 | 81     |
| 9   | 1001 | 9002    | 2021-07-01 12:01:01 | 2021-07-01 12:31:01 | 81     |
| 10  | 1001 | 9002    | 2021-07-01 12:01:01 | (NULL)              | (NULL) |

Tìm số đề đã hoàn thành của những người mà trong ba tháng gần nhất có bản ghi trả lời đề thi không có đề nào ở trạng thái chưa hoàn thành, sau đó xếp hạng giảm dần theo số đề đã hoàn thành và ID người dùng. Kết quả theo dữ liệu mẫu như sau:

| uid  | exam_complete_cnt |
| ---- | ----------------- |
| 1006 | 3                 |

**Giải thích**: Ba tháng gần nhất có bản ghi làm đề của người dùng 1006 là 202109, 202108, 202106; số đề đã làm là 3 và đều hoàn thành. Ba tháng gần nhất có bản ghi làm đề của người dùng 1001 là 202109, 202108, 202107; số đề đã làm là 5, số đề hoàn thành là 4 vì có đề chưa hoàn thành, nên bị lọc.

**Cách làm:**

1. `Tìm số đề đã hoàn thành của những người mà trong ba tháng gần nhất có bản ghi trả lời đề thi không có đề nào ở trạng thái chưa hoàn thành`: Trước hết cần nhóm theo người dùng.
2. Ba tháng gần nhất có thể dùng thứ hạng liên tiếp, sắp xếp giảm dần và lấy thứ hạng <= 3.
3. Thống kê số lần trả lời.
4. Kết hợp các điều kiện còn lại.
5. Sắp xếp.

**Đáp án**:

```sql
SELECT UID,
       count(score) exam_complete_cnt
FROM
  (SELECT *, DENSE_RANK() OVER (PARTITION BY UID
                             ORDER BY date_format(start_time, '%Y%m') DESC) dr
   FROM exam_record) t1
WHERE dr <= 3
GROUP BY UID
HAVING count(dr)= count(score)
ORDER BY exam_complete_cnt DESC,
         UID DESC
```

### Tình hình làm bài trong ba tháng gần nhất của 50% người dùng có tỷ lệ chưa hoàn thành cao (khó)

**Mô tả**:

Hiện có bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` nickname, `achievement` điểm thành tích, `level` cấp độ, `job` hướng nghề nghiệp, `register_time` thời gian đăng ký):

| id  | uid  | nick_name | achievement | level | job       | register_time       |
| --- | ---- | --------- | ----------- | ----- | --------- | ------------------- |
| 1   | 1001 | User 1    | 3200        | 7     | Algorithm | 2020-01-01 10:00:00 |
| 2   | 1002 | User 2    | 2500        | 6     | Algorithm | 2020-01-01 10:00:00 |
| 3   | 1003 | User 3    | 2200        | 5     | Algorithm | 2020-01-01 10:00:00 |

Bảng thông tin đề thi `examination_info` (`exam_id` ID đề thi, `tag` loại đề thi, `difficulty` độ khó đề thi, `duration` thời lượng thi, `release_time` thời gian phát hành):

| id  | exam_id | tag       | difficulty | duration | release_time        |
| --- | ------- | --------- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL       | hard       | 60       | 2020-01-01 10:00:00 |
| 2   | 9002    | SQL       | hard       | 80       | 2020-01-01 10:00:00 |
| 3   | 9003    | Algorithm | hard       | 80       | 2020-01-01 10:00:00 |
| 4   | 9004    | PYTHON    | medium     | 70       | 2020-01-01 10:00:00 |

Bảng ghi chép trả lời đề thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` thời gian bắt đầu trả lời, `submit_time` thời gian nộp bài, `score` điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score |
| --- | ---- | ------- | ------------------- | ------------------- | ----- |
| 1   | 1001 | 9001    | 2020-01-01 09:01:01 | 2020-01-01 09:21:59 | 90    |
| 15  | 1002 | 9001    | 2020-01-01 18:01:01 | 2020-01-01 18:59:02 | 90    |
| 13  | 1001 | 9001    | 2020-01-02 10:01:01 | 2020-01-02 10:31:01 | 89    |
| 2   | 1002 | 9001    | 2020-01-20 10:01:01 |                     |       |
| 3   | 1002 | 9001    | 2020-02-01 12:11:01 |                     |       |
| 5   | 1001 | 9001    | 2020-03-01 12:01:01 |                     |       |
| 6   | 1002 | 9001    | 2020-03-01 12:01:01 | 2020-03-01 12:41:01 | 90    |
| 4   | 1003 | 9001    | 2020-03-01 19:01:01 |                     |       |
| 7   | 1002 | 9001    | 2020-05-02 19:01:01 | 2020-05-02 19:32:00 | 90    |
| 14  | 1001 | 9002    | 2020-01-01 12:11:01 |                     |       |
| 8   | 1001 | 9002    | 2020-01-02 19:01:01 | 2020-01-02 19:59:01 | 69    |
| 9   | 1001 | 9002    | 2020-02-02 12:01:01 | 2020-02-02 12:20:01 | 99    |
| 10  | 1002 | 9002    | 2020-02-02 12:01:01 |                     |       |
| 11  | 1002 | 9002    | 2020-02-02 12:01:01 | 2020-02-02 12:43:01 | 81    |
| 12  | 1002 | 9002    | 2020-03-02 12:11:01 |                     |       |
| 17  | 1001 | 9002    | 2020-05-05 18:01:01 |                     |       |
| 16  | 1002 | 9003    | 2020-05-06 12:01:01 |                     |       |

Hãy thống kê số bài làm và số bài hoàn thành mỗi tháng trong ba tháng gần nhất có bản ghi trả lời đề thi của người dùng cấp 6 và cấp 7 thuộc 50% người dùng có tỷ lệ chưa hoàn thành đề SQL cao. Sắp xếp tăng dần theo ID người dùng và tháng.

Kết quả theo dữ liệu mẫu như sau:

| uid  | start_month | total_cnt | complete_cnt |
| ---- | ----------- | --------- | ------------ |
| 1002 | 202002      | 3         | 1            |
| 1002 | 202003      | 2         | 1            |
| 1002 | 202005      | 2         | 1            |

Giải thích: Số bài chưa hoàn thành, tổng số bài đã làm và tỷ lệ chưa hoàn thành đề SQL của từng người dùng như sau:

| uid  | incomplete_cnt | total_cnt | incomplete_rate |
| ---- | -------------- | --------- | --------------- |
| 1001 | 3              | 7         | 0.4286          |
| 1002 | 4              | 8         | 0.5000          |
| 1003 | 1              | 1         | 1.0000          |

1001, 1002, 1003 lần lượt ở vị trí 1.0, 0.5, 0.0, nên 50% người dùng có tỷ lệ cao hơn (vị trí <= 0.5) là 1002, 1003;

1003 không ở cấp 6 hoặc 7;

Ba tháng gần nhất có bản ghi trả lời đề thi là 202005, 202003, 202002;

Trong ba tháng này, số bài làm của 1002 lần lượt là 3, 2, 2; số bài hoàn thành lần lượt là 1, 1, 1.

**Cách làm:**

Lưu ý: Bài này cần tính tất cả số lần trả lời và số lần hoàn thành; loại đề SQL chỉ dùng để giới hạn thứ hạng tỷ lệ chưa hoàn thành, còn người dùng cấp 6, 7 chỉ dùng để giới hạn bản ghi làm bài.

Trước hết tính thứ hạng tỷ lệ chưa hoàn thành:

```sql
SELECT UID,
       count(submit_time IS NULL
             OR NULL)/ count(start_time) AS num,
       PERCENT_RANK() OVER (
                            ORDER BY count(submit_time IS NULL
                                           OR NULL)/ count(start_time)) AS ranking
FROM exam_record
LEFT JOIN examination_info USING (exam_id)
WHERE tag = 'SQL'
GROUP BY UID
```

Tiếp theo lấy bản ghi luyện tập trong ba tháng gần nhất:

```sql
SELECT UID,
       date_format(start_time, '%Y%m') AS month_d,
       submit_time,
       exam_id,
       dense_rank() OVER (PARTITION BY UID
                          ORDER BY date_format(start_time, '%Y%m') DESC) AS ranking
FROM exam_record
LEFT JOIN user_info USING (UID)
WHERE LEVEL IN (6,7)
```

**Đáp án**:

```sql
SELECT t1.uid,
       t1.month_d,
       count(*) AS total_cnt,
       count(t1.submit_time) AS complete_cnt
FROM-- Tính thứ hạng tỷ lệ chưa hoàn thành

  (SELECT UID,
          count(submit_time IS NULL OR NULL)/ count(start_time) AS num,
          PERCENT_RANK() OVER (
                               ORDER BY count(submit_time IS NULL OR NULL)/ count(start_time)) AS ranking
   FROM exam_record
   LEFT JOIN examination_info USING (exam_id)
   WHERE tag = 'SQL'
   GROUP BY UID) t
INNER JOIN
  (-- Lấy bản ghi luyện tập trong ba tháng gần nhất
 SELECT UID,
        date_format(start_time, '%Y%m') AS month_d,
        submit_time,
        exam_id,
        dense_rank() OVER (PARTITION BY UID
                           ORDER BY date_format(start_time, '%Y%m') DESC) AS ranking
   FROM exam_record
   LEFT JOIN user_info USING (UID)
   WHERE LEVEL IN (6,7) ) t1 USING (UID)
WHERE t1.ranking <= 3 AND t.ranking >= 0.5 -- Dùng điều kiện để tìm bản ghi phù hợp

GROUP BY t1.uid,
         t1.month_d
ORDER BY t1.uid,
         t1.month_d
```

### Tỷ lệ tăng trưởng và thay đổi thứ hạng số đề hoàn thành so với năm 2020 (khó)

**Mô tả**:

Hiện có bảng thông tin đề thi `examination_info` (`exam_id` ID đề thi, `tag` loại đề thi, `difficulty` độ khó đề thi, `duration` thời lượng thi, `release_time` thời gian phát hành):

| id  | exam_id | tag       | difficulty | duration | release_time        |
| --- | ------- | --------- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL       | hard       | 60       | 2021-01-01 10:00:00 |
| 2   | 9002    | C++       | hard       | 80       | 2021-01-01 10:00:00 |
| 3   | 9003    | Algorithm | hard       | 80       | 2021-01-01 10:00:00 |
| 4   | 9004    | PYTHON    | medium     | 70       | 2021-01-01 10:00:00 |

Bảng ghi chép trả lời đề thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` thời gian bắt đầu trả lời, `submit_time` thời gian nộp bài, `score` điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score |
| --- | ---- | ------- | ------------------- | ------------------- | ----- |
| 1   | 1001 | 9001    | 2020-08-02 10:01:01 | 2020-08-02 10:31:01 | 89    |
| 2   | 1002 | 9001    | 2020-04-01 18:01:01 | 2020-04-01 18:59:02 | 90    |
| 3   | 1001 | 9001    | 2020-04-01 09:01:01 | 2020-04-01 09:21:59 | 80    |
| 5   | 1002 | 9001    | 2021-03-02 19:01:01 | 2021-03-02 19:32:00 | 20    |
| 8   | 1003 | 9001    | 2021-05-02 12:01:01 | 2021-05-02 12:31:01 | 98    |
| 13  | 1003 | 9001    | 2020-01-02 10:01:01 | 2020-01-02 10:31:01 | 89    |
| 9   | 1001 | 9002    | 2020-02-02 12:01:01 | 2020-02-02 12:20:01 | 99    |
| 10  | 1002 | 9002    | 2021-02-02 12:01:01 | 2020-02-02 12:43:01 | 81    |
| 11  | 1001 | 9002    | 2020-01-02 19:01:01 | 2020-01-02 19:59:01 | 69    |
| 16  | 1002 | 9002    | 2020-02-02 12:01:01 |                     |       |
| 17  | 1002 | 9002    | 2020-03-02 12:11:01 |                     |       |
| 18  | 1001 | 9002    | 2021-05-05 18:01:01 |                     |       |
| 4   | 1002 | 9003    | 2021-01-20 10:01:01 | 2021-01-20 10:10:01 | 81    |
| 6   | 1001 | 9003    | 2021-04-02 19:01:01 | 2021-04-02 19:40:01 | 89    |
| 15  | 1002 | 9003    | 2021-01-01 18:01:01 | 2021-01-01 18:59:02 | 90    |
| 7   | 1004 | 9004    | 2020-05-02 12:01:01 | 2020-05-02 12:20:01 | 99    |
| 12  | 1001 | 9004    | 2021-09-02 12:11:01 |                     |       |
| 14  | 1002 | 9004    | 2020-01-01 12:11:01 | 2020-01-01 12:31:01 | 83    |

Hãy tính tỷ lệ tăng trưởng số lần hoàn thành của từng loại đề thi trong nửa đầu năm 2021 so với cùng kỳ nửa đầu năm 2020 (định dạng phần trăm, giữ lại 1 chữ số thập phân), cùng thay đổi thứ hạng số lần hoàn thành; xuất theo thứ tự giảm dần của tỷ lệ tăng trưởng và thứ hạng năm 2021.

Kết quả theo dữ liệu mẫu như sau:

| tag | exam_cnt_20 | exam_cnt_21 | growth_rate | exam_cnt_rank_20 | exam_cnt_rank_21 | rank_delta |
| --- | ----------- | ----------- | ----------- | ---------------- | ---------------- | ---------- |
| SQL | 3           | 2           | -33.3%      | 1                | 2                | 1          |

Giải thích: Trong nửa đầu năm 2020 có 3 tag có bản ghi trả lời đã hoàn thành, lần lượt là C++, SQL, PYTHON; số lần hoàn thành lần lượt là 3, 3, 2, thứ hạng số lần hoàn thành là 1, 1 (đồng hạng), 3;

Trong nửa đầu năm 2021 có 2 tag có bản ghi trả lời đã hoàn thành, lần lượt là Algorithm, SQL; số lần hoàn thành lần lượt là 3, 2, thứ hạng số lần hoàn thành là 1, 2, cụ thể như sau:

| tag       | start_year | exam_cnt | exam_cnt_rank |
| --------- | ---------- | -------- | ------------- |
| C++       | 2020       | 3        | 1             |
| SQL       | 2020       | 3        | 1             |
| PYTHON    | 2020       | 2        | 3             |
| Algorithm | 2021       | 3        | 1             |
| SQL       | 2021       | 2        | 2             |

Vì vậy, chỉ có tag SQL đủ điều kiện xuất kết quả so sánh cùng kỳ: từ năm 2020 đến năm 2021, số lần hoàn thành giảm từ 3 xuống 2, giảm 33.3% (giữ lại 1 chữ số thập phân); thứ hạng từ 1 xuống 2, lùi 1 hạng.

**Cách làm:**

Điểm khó của bài này là kiểu dữ liệu integer dài không được sinh dấu âm, cần dùng hàm cast để chuyển kiểu dữ liệu thành signed.

Ngoài ra, dùng `công thức tính tỷ lệ tăng trưởng: (exam_cnt_21-exam_cnt_20)/exam_cnt_20`.

Thay đổi thứ hạng số lần hoàn thành (thứ hạng năm 2021 tăng hoặc giảm bao nhiêu so với năm 2020)

Công thức tính: `exam_cnt_rank_21 - exam_cnt_rank_20`

Trong MySQL, hàm `CAST()` dùng để chuyển kiểu dữ liệu của một biểu thức thành kiểu dữ liệu khác. Cú pháp cơ bản như sau:

```sql
CAST(expression AS data_type)

-- Chuyển một chuỗi thành số nguyên
SELECT CAST('123' AS INT);
```

Ví dụ không được trình bày lần lượt ở đây vì hàm này khá đơn giản.

**Đáp án**:

```sql
SELECT
  tag,
  exam_cnt_20,
  exam_cnt_21,
  concat(
    round(
      100 * (exam_cnt_21 - exam_cnt_20) / exam_cnt_20,
      1
    ),
    '%'
  ) AS growth_rate,
  exam_cnt_rank_20,
  exam_cnt_rank_21,
  cast(exam_cnt_rank_21 AS signed) - cast(exam_cnt_rank_20 AS signed) AS rank_delta
FROM
  (
    # Số lần hoàn thành và thứ hạng số lần hoàn thành của từng loại đề thi trong nửa đầu năm 2020 và 2021
    SELECT
      tag,
      count(
        IF (
          date_format(start_time, '%Y%m%d') BETWEEN '20200101'
          AND '20200630',
          start_time,
          NULL
        )
      ) AS exam_cnt_20,
      count(
        IF (
          substring(start_time, 1, 10) BETWEEN '2021-01-01'
          AND '2021-06-30',
          start_time,
          NULL
        )
      ) AS exam_cnt_21,
      rank() over (
        ORDER BY
          count(
            IF (
              date_format(start_time, '%Y%m%d') BETWEEN '20200101'
              AND '20200630',
              start_time,
              NULL
            )
          ) DESC
      ) AS exam_cnt_rank_20,
      rank() over (
        ORDER BY
          count(
            IF (
              substring(start_time, 1, 10) BETWEEN '2021-01-01'
              AND '2021-06-30',
              start_time,
              NULL
            )
          ) DESC
      ) AS exam_cnt_rank_21
    FROM
      examination_info
      JOIN exam_record USING (exam_id)
    WHERE
      submit_time IS NOT NULL
    GROUP BY
      tag
  ) main
WHERE
  exam_cnt_21 * exam_cnt_20 <> 0
ORDER BY
  growth_rate DESC,
  exam_cnt_rank_21 DESC
```

## Aggregate window function

### Chuẩn hóa min-max điểm đề thi

**Mô tả**:

Hiện có bảng thông tin đề thi `examination_info` (`exam_id` ID đề thi, `tag` loại đề thi, `difficulty` độ khó đề thi, `duration` thời lượng thi, `release_time` thời gian phát hành):

| id  | exam_id | tag       | difficulty | duration | release_time        |
| --- | ------- | --------- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL       | hard       | 60       | 2020-01-01 10:00:00 |
| 2   | 9002    | C++       | hard       | 80       | 2020-01-01 10:00:00 |
| 3   | 9003    | Algorithm | hard       | 80       | 2020-01-01 10:00:00 |
| 4   | 9004    | PYTHON    | medium     | 70       | 2020-01-01 10:00:00 |

Bảng ghi chép trả lời đề thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` thời gian bắt đầu trả lời, `submit_time` thời gian nộp bài, `score` điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 6   | 1003 | 9001    | 2020-01-02 12:01:01 | 2020-01-02 12:31:01 | 68     |
| 9   | 1001 | 9001    | 2020-01-02 10:01:01 | 2020-01-02 10:31:01 | 89     |
| 1   | 1001 | 9001    | 2020-01-01 09:01:01 | 2020-01-01 09:21:01 | 90     |
| 12  | 1002 | 9002    | 2021-05-05 18:01:01 | (NULL)              | (NULL) |
| 3   | 1004 | 9002    | 2020-01-01 12:01:01 | 2020-01-01 12:11:01 | 60     |
| 2   | 1003 | 9002    | 2020-01-01 19:01:01 | 2020-01-01 19:30:01 | 75     |
| 7   | 1001 | 9002    | 2020-01-02 12:01:01 | 2020-01-02 12:43:01 | 81     |
| 10  | 1002 | 9002    | 2020-01-01 12:11:01 | 2020-01-01 12:31:01 | 83     |
| 4   | 1003 | 9002    | 2020-01-01 12:01:01 | 2020-01-01 12:41:01 | 90     |
| 5   | 1002 | 9002    | 2020-01-02 19:01:01 | 2020-01-02 19:32:00 | 90     |
| 11  | 1002 | 9004    | 2021-09-06 12:01:01 | (NULL)              | (NULL) |
| 8   | 1001 | 9005    | 2020-01-02 12:11:01 | (NULL)              | (NULL) |

Trong tính toán dữ liệu vật lý và thống kê, có một khái niệm gọi là chuẩn hóa min-max, còn gọi là chuẩn hóa độ lệch. Đây là phép biến đổi tuyến tính dữ liệu gốc, ánh xạ giá trị kết quả vào khoảng [0 - 1].

Công thức chuyển đổi:

![](https://oss.javaguide.cn/github/javaguide/database/sql/29A377601170AB822322431FCDF7EDFE.png)

Hãy thực hiện chuẩn hóa min-max điểm của người dùng đối với các đề thi có độ khó cao trong từng bản ghi trả lời đề thi, sau đó scale về khoảng [0,100], rồi xuất ID người dùng, ID đề thi và giá trị trung bình của điểm sau chuẩn hóa; cuối cùng sắp xếp tăng dần theo ID đề thi và giảm dần theo điểm chuẩn hóa. (Lưu ý: Khoảng điểm mặc định là [0,100]. Nếu một bản ghi trả lời đề thi chỉ có một điểm, không cần dùng công thức; sau khi chuẩn hóa và scale, điểm vẫn là điểm ban đầu.)

Kết quả theo dữ liệu mẫu như sau:

| uid  | exam_id | avg_new_score |
| ---- | ------- | ------------- |
| 1001 | 9001    | 98            |
| 1003 | 9001    | 0             |
| 1002 | 9002    | 88            |
| 1003 | 9002    | 75            |
| 1001 | 9002    | 70            |
| 1004 | 9002    | 0             |

Giải thích: Các đề thi có độ khó cao là 9001, 9002, 9003;

Có 3 bản ghi trả lời đề 9001, điểm lần lượt là 68, 89, 90. Theo công thức đã cho, điểm sau chuẩn hóa là 0, 95, 100. Hai điểm sau cùng đều do người dùng 1001 trả lời, nên điểm mới của người dùng 1001 đối với đề 9001 là (95+100)/2 ≈ 98 (chỉ giữ phần nguyên); điểm mới của người dùng 1003 đối với đề 9001 là 0. Cuối cùng, kết quả được sắp xếp tăng dần theo ID đề thi và giảm dần theo điểm chuẩn hóa.

**Cách làm:**

Điểm cần lưu ý:

1. Với các đề thi có độ khó cao, theo điểm của từng loại đề thi, dùng window function max/min (col) over() để tìm giá trị lớn nhất và nhỏ nhất trong từng nhóm, sau đó tính theo công thức chuẩn hóa; khoảng scale là [0,100], tức là min_max\*100.
2. Nếu một loại đề thi chỉ có một điểm, không cần dùng công thức chuẩn hóa; vì chỉ có một điểm nên max_score=min_score=score, kết quả sau công thức có thể trở thành 0.
3. Cuối cùng, nhóm kết quả theo uid, exam_id để tính giá trị trung bình sau chuẩn hóa; cần lọc score là NULL.

Cuối cùng, hãy đọc kỹ công thức ở trên (nói thật, bài này nhìn khá rắc rối).

**Đáp án**:

```sql
SELECT
  uid,
  exam_id,
  round(sum(min_max) / count(score), 0) AS avg_new_score
FROM
  (
    SELECT
      *,
      IF (
        max_score = min_score,
        score,
        (score - min_score) / (max_score - min_score) * 100
      ) AS min_max
    FROM
      (
        SELECT
          uid,
          a.exam_id,
          score,
          max(score) over (PARTITION BY a.exam_id) AS max_score,
          min(score) over (PARTITION BY a.exam_id) AS min_score
        FROM
          exam_record a
          LEFT JOIN examination_info b USING (exam_id)
        WHERE
          difficulty = 'hard'
      ) t
    WHERE
      score IS NOT NULL
  ) t1
GROUP BY
  uid,
  exam_id
ORDER BY
  exam_id ASC,
  avg_new_score DESC;
```

### Số lần trả lời mỗi đề thi mỗi tháng và tổng số lần trả lời tính đến tháng đó

**Mô tả:**

Hiện có bảng ghi chép trả lời đề thi `exam_record` (uid ID người dùng, exam_id ID đề thi, start_time thời gian bắt đầu trả lời, submit_time thời gian nộp bài, score điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-01 09:01:01 | 2020-01-01 09:21:59 | 90     |
| 2   | 1002 | 9001    | 2020-01-20 10:01:01 | 2020-01-20 10:10:01 | 89     |
| 3   | 1002 | 9001    | 2020-02-01 12:11:01 | 2020-02-01 12:31:01 | 83     |
| 4   | 1003 | 9001    | 2020-03-01 19:01:01 | 2020-03-01 19:30:01 | 75     |
| 5   | 1004 | 9001    | 2020-03-01 12:01:01 | 2020-03-01 12:11:01 | 60     |
| 6   | 1003 | 9001    | 2020-03-01 12:01:01 | 2020-03-01 12:41:01 | 90     |
| 7   | 1002 | 9001    | 2020-05-02 19:01:01 | 2020-05-02 19:32:00 | 90     |
| 8   | 1001 | 9002    | 2020-01-02 19:01:01 | 2020-01-02 19:59:01 | 69     |
| 9   | 1004 | 9002    | 2020-02-02 12:01:01 | 2020-02-02 12:20:01 | 99     |
| 10  | 1003 | 9002    | 2020-02-02 12:01:01 | 2020-02-02 12:31:01 | 68     |
| 11  | 1001 | 9002    | 2020-02-02 12:01:01 | 2020-02-02 12:43:01 | 81     |
| 12  | 1001 | 9002    | 2020-03-02 12:11:01 | (NULL)              | (NULL) |

Hãy xuất số lần trả lời mỗi đề thi mỗi tháng và tổng số lần trả lời tính đến tháng đó.
Kết quả theo dữ liệu mẫu như sau:

| exam_id | start_month | month_cnt | cum_exam_cnt |
| ------- | ----------- | --------- | ------------ |
| 9001    | 202001      | 2         | 2            |
| 9001    | 202002      | 1         | 3            |
| 9001    | 202003      | 3         | 6            |
| 9001    | 202005      | 1         | 7            |
| 9002    | 202001      | 1         | 1            |
| 9002    | 202002      | 3         | 4            |
| 9002    | 202003      | 1         | 5            |

Giải thích: Đề 9001 có bản ghi trả lời trong tổng cộng 4 tháng 202001, 202002, 202003, 202005; số lần trả lời mỗi tháng lần lượt là 2, 1, 3, 1, tổng số lần trả lời tích lũy tính đến từng tháng lần lượt là 2, 3, 6, 7.

**Cách làm:**

Bài này có hai điểm then chốt: thống kê tổng số lần trả lời tính đến tháng đó; xuất số lần trả lời mỗi đề thi mỗi tháng và tổng số lần trả lời tính đến tháng đó.

Điểm mấu chốt là `**sum(count(*)) over(partition by exam_id order by date_format(start_time,'%Y%m'))**`.

**Đáp án**:

```sql
SELECT exam_id,
       date_format(start_time, '%Y%m') AS start_month,
       count(*) AS month_cnt,
       sum(count(*)) OVER (PARTITION BY exam_id
                           ORDER BY date_format(start_time, '%Y%m')) AS cum_exam_cnt
FROM exam_record
GROUP BY exam_id,
         start_month
```

### Tình hình trả lời mỗi tháng và tính đến tháng đó (khá khó)

**Mô tả**: Hiện có bảng ghi chép trả lời đề thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` thời gian bắt đầu trả lời, `submit_time` thời gian nộp bài, `score` điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-01 09:01:01 | 2020-01-01 09:21:59 | 90     |
| 2   | 1002 | 9001    | 2020-01-20 10:01:01 | 2020-01-20 10:10:01 | 89     |
| 3   | 1002 | 9001    | 2020-02-01 12:11:01 | 2020-02-01 12:31:01 | 83     |
| 4   | 1003 | 9001    | 2020-03-01 19:01:01 | 2020-03-01 19:30:01 | 75     |
| 5   | 1004 | 9001    | 2020-03-01 12:01:01 | 2020-03-01 12:11:01 | 60     |
| 6   | 1003 | 9001    | 2020-03-01 12:01:01 | 2020-03-01 12:41:01 | 90     |
| 7   | 1002 | 9001    | 2020-05-02 19:01:01 | 2020-05-02 19:32:00 | 90     |
| 8   | 1001 | 9002    | 2020-01-02 19:01:01 | 2020-01-02 19:59:01 | 69     |
| 9   | 1004 | 9002    | 2020-02-02 12:01:01 | 2020-02-02 12:20:01 | 99     |
| 10  | 1003 | 9002    | 2020-02-02 12:01:01 | 2020-02-02 12:31:01 | 68     |
| 11  | 1001 | 9002    | 2020-01-02 19:01:01 | 2020-02-02 12:43:01 | 81     |
| 12  | 1001 | 9002    | 2020-03-02 12:11:01 | (NULL)              | (NULL) |

Hãy xuất số người dùng hoạt động tháng, số người dùng mới trong tháng, số người dùng mới tối đa trong một tháng tính đến tháng đó và số người dùng tích lũy tính đến tháng đó trong các bản ghi trả lời đề thi mỗi tháng kể từ khi có bản ghi trả lời của người dùng. Kết quả sắp xếp tăng dần theo tháng.

Kết quả theo dữ liệu mẫu như sau:

| start_month | mau | month_add_uv | max_month_add_uv | cum_sum_uv |
| ----------- | --- | ------------ | ---------------- | ---------- |
| 202001      | 2   | 2            | 2                | 2          |
| 202002      | 4   | 2            | 2                | 4          |
| 202003      | 3   | 0            | 2                | 4          |
| 202005      | 1   | 0            | 2                | 4          |

| month  | 1001 | 1002 | 1003 | 1004 |
| ------ | ---- | ---- | ---- | ---- |
| 202001 | 1    | 1    |      |      |
| 202002 | 1    | 1    | 1    | 1    |
| 202003 | 1    |      | 1    | 1    |
| 202005 |      | 1    |      |      |

Từ ma trận trên có thể thấy tháng 1 năm 2020 có 2 người dùng hoạt động (mau=2), số người dùng mới trong tháng là 2;

Tháng 2 năm 2020 có 4 người dùng hoạt động, số người dùng mới trong tháng là 2, số người dùng mới tối đa trong một tháng là 2, số người dùng tích lũy hiện tại là 4.

**Cách làm:**

Điểm khó:

1. Cách tính số người dùng mới mỗi tháng.

2. Tình hình trả lời tính đến tháng đó.

Quy trình đại khái:

(1) Thống kê tháng đăng nhập đầu tiên của mỗi người dùng bằng `min()`.

(2) Thống kê số người dùng hoạt động và số người dùng mới mỗi tháng: trước hết lấy tháng đăng nhập đầu tiên của mỗi người dùng, sau đó nhóm theo tháng đăng nhập đầu tiên và tính tổng để ra số người dùng mới trong tháng đó.

(3) Thống kê số người dùng mới tối đa trong một tháng và số người dùng tích lũy tính đến tháng đó, cuối cùng xuất theo thứ tự tăng dần của tháng.

**Đáp án**:

```sql
-- Số người dùng mới tối đa trong một tháng và số người dùng tích lũy tính đến tháng đó, xuất theo thứ tự tăng dần của tháng
SELECT
	start_month,
	mau,
	month_add_uv,
	max( month_add_uv ) over ( ORDER BY start_month ),
	sum( month_add_uv ) over ( ORDER BY start_month )
FROM
	(
	-- Thống kê số người dùng hoạt động và số người dùng mới mỗi tháng
	SELECT
		date_format( a.start_time, '%Y%m' ) AS start_month,
		count( DISTINCT a.uid ) AS mau,
		count( DISTINCT b.uid ) AS month_add_uv
	FROM
		exam_record a
		LEFT JOIN (
         -- Thống kê tháng đăng nhập đầu tiên của mỗi người dùng
		SELECT uid, min( date_format( start_time, '%Y%m' )) AS first_month FROM exam_record GROUP BY uid ) b ON date_format( a.start_time, '%Y%m' ) = b.first_month
	GROUP BY
		start_month
	) main
ORDER BY
	start_month
```

<!-- @include: @article-footer.snippet.md -->
