---
title: Tổng hợp câu hỏi phỏng vấn SQL thường gặp (5)
description: Phần năm của tổng hợp câu hỏi phỏng vấn SQL thường gặp, giải thích chi tiết kỹ thuật xử lý giá trị NULL, bao gồm hàm IFNULL, COALESCE, cũng như dùng CASE WHEN để thống kê theo điều kiện và tính tỷ lệ hoàn thành.
category: Database
tag:
  - Database Basics
  - SQL
head:
  - - meta
    - name: keywords
      content: câu hỏi phỏng vấn SQL,xử lý giá trị NULL,IFNULL,COALESCE,CASE WHEN,thống kê theo điều kiện,tính tỷ lệ hoàn thành
---

> Nguồn đề bài: [Nowcoder - Thử thách SQL nâng cao](https://www.nowcoder.com/exam/oj?page=1&tab=SQL%E7%AF%87&topicId=240)

Các câu hỏi tương đối khó hoặc khó có thể được bỏ qua tùy theo tình hình thực tế và nhu cầu phỏng vấn của bạn.

## Xử lý giá trị rỗng

### Thống kê số lượng và tỷ lệ bài thi chưa hoàn thành

**Mô tả**:

Có bảng ghi chép làm bài `exam_record` (`uid` ID người dùng, `exam_id` ID bài thi, `start_time` thời gian bắt đầu làm bài, `submit_time` thời gian nộp bài, `score` điểm), dữ liệu như sau:

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-02 09:01:01 | 2020-01-02 09:21:01 | 80     |
| 2   | 1001 | 9001    | 2021-05-02 10:01:01 | 2021-05-02 10:30:01 | 81     |
| 3   | 1001 | 9001    | 2021-09-02 12:01:01 | (NULL)              | (NULL) |

Hãy thống kê số lượng `incomplete_cnt` và tỷ lệ `incomplete_rate` bài thi chưa hoàn thành trong các bài thi có trạng thái chưa hoàn thành. Kết quả từ dữ liệu mẫu như sau:

| exam_id | incomplete_cnt | complete_rate |
| ------- | -------------- | ------------- |
| 9001    | 1              | 0.333         |

Giải thích: Bài thi 9001 có 3 bản ghi làm bài, trong đó 2 lần hoàn thành và 1 lần chưa hoàn thành, vì vậy số lượng chưa hoàn thành là 1, tỷ lệ chưa hoàn thành là 0.333 (giữ lại 3 chữ số thập phân).

**Cách làm**:

Bài này chỉ cần chú ý một phần có điều kiện, một phần không có điều kiện; có thể truy vấn hai phần điều kiện riêng rồi hợp nhất, hoặc trực tiếp phán đoán điều kiện trong `select`.

**Đáp án**:

Cách viết 1:

```sql
SELECT
    exam_id,
    (COUNT(*) - COUNT(submit_time)) AS incomplete_cnt,
    ROUND((COUNT(*) - COUNT(submit_time)) / COUNT(*), 3) AS incomplete_rate
FROM
    exam_record
GROUP BY
    exam_id
HAVING
    (COUNT(*) - COUNT(submit_time)) > 0;
```

Dùng `COUNT(*)` để thống kê tổng số bản ghi trong nhóm, `COUNT(submit_time)` chỉ thống kê các bản ghi có trường `submit_time` khác NULL (tức số lần đã hoàn thành). Hiệu của hai giá trị là số lần chưa hoàn thành.

Cách viết 2:

```sql
SELECT
    exam_id,
    COUNT(CASE WHEN submit_time IS NULL THEN 1 END) AS incomplete_cnt,
    ROUND(COUNT(CASE WHEN submit_time IS NULL THEN 1 END) / COUNT(*), 3) AS incomplete_rate
FROM
    exam_record
GROUP BY
    exam_id
HAVING
    COUNT(CASE WHEN submit_time IS NULL THEN 1 END) > 0;
```

Dùng biểu thức `CASE`: khi điều kiện đúng thì trả về một giá trị khác `NULL` (ví dụ 1), nếu không thì trả về `NULL`. Sau đó dùng hàm `COUNT` để thống kê số lượng giá trị khác `NULL`.

Cách viết 3:

```sql
SELECT
    exam_id,
    SUM(submit_time IS NULL) AS incomplete_cnt,
    ROUND(SUM(submit_time IS NULL) / COUNT(*), 3) AS incomplete_rate
FROM
    exam_record
GROUP BY
    exam_id
HAVING
    incomplete_cnt > 0;
```

Dùng hàm `SUM` để tính tổng một biểu thức. Khi `submit_time` là `NULL`, giá trị của biểu thức `(submit_time IS NULL)` là 1 (TRUE), nếu không thì là 0 (FALSE). Cộng các giá trị 1 và 0 lại sẽ thu được số lượng chưa hoàn thành.

### Thời gian làm bài trung bình và điểm trung bình của bài thi khó của người dùng cấp 0

**Mô tả**:

Có bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` nickname, `achievement` điểm thành tích, `level` cấp độ, `job` hướng nghề nghiệp, `register_time` thời gian đăng ký), dữ liệu như sau:

| id  | uid  | nick_name  | achievement | level | job        | register_time       |
| --- | ---- | ---------- | ----------- | ----- | ---------- | ------------------- |
| 1   | 1001 | NiuKe số 1 | 10          | 0     | Thuật toán | 2020-01-01 10:00:00 |
| 2   | 1002 | NiuKe số 2 | 2100        | 6     | Thuật toán | 2020-01-01 10:00:00 |

Bảng thông tin bài thi `examination_info` (`exam_id` ID bài thi, `tag` loại bài thi, `difficulty` độ khó bài thi, `duration` thời lượng thi, `release_time` thời gian phát hành), dữ liệu như sau:

| id  | exam_id | tag        | difficulty | duration | release_time        |
| --- | ------- | ---------- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL        | hard       | 60       | 2020-01-01 10:00:00 |
| 2   | 9002    | SQL        | easy       | 60       | 2020-01-01 10:00:00 |
| 3   | 9004    | Thuật toán | medium     | 80       | 2020-01-01 10:00:00 |

Bảng ghi chép làm bài `exam_record` (`uid` ID người dùng, `exam_id` ID bài thi, `start_time` thời gian bắt đầu làm bài, `submit_time` thời gian nộp bài, `score` điểm), dữ liệu như sau:

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-02 09:01:01 | 2020-01-02 09:21:59 | 80     |
| 2   | 1001 | 9001    | 2021-05-02 10:01:01 | (NULL)              | (NULL) |
| 3   | 1001 | 9002    | 2021-02-02 19:01:01 | 2021-02-02 19:30:01 | 87     |
| 4   | 1001 | 9001    | 2021-06-02 19:01:01 | 2021-06-02 19:32:00 | 20     |
| 5   | 1001 | 9002    | 2021-09-05 19:01:01 | 2021-09-05 19:40:01 | 89     |
| 6   | 1001 | 9002    | 2021-09-01 12:01:01 | (NULL)              | (NULL) |
| 7   | 1002 | 9002    | 2021-05-05 18:01:01 | 2021-05-05 18:59:02 | 90     |

Hãy xuất thời gian làm bài trung bình và điểm trung bình của tất cả bài thi khó đối với mỗi người dùng cấp 0. Với bài chưa hoàn thành, mặc định dùng thời lượng thi tối đa của bài thi và điểm 0. Kết quả từ dữ liệu mẫu như sau:

| uid  | avg_score | avg_time_took |
| ---- | --------- | ------------- |
| 1001 | 33        | 36.7          |

Giải thích: Người dùng cấp 0 là 1001, bài thi khó là 9001. Các bản ghi làm bài 9001 của 1001 có 3 bản ghi, thời gian lần lượt là 20 phút, chưa hoàn thành (dùng thời lượng bài thi 60 phút), 30 phút (chưa đủ 31 phút), điểm lần lượt là 80, chưa hoàn thành (xử lý là 0), 20. Vì vậy thời gian trung bình là 110/3=36.7 (giữ lại một chữ số thập phân), điểm trung bình là 33 (làm tròn lấy phần nguyên).

**Cách làm**: Bài này dùng `IF` là thuận tiện nhất vì có xử lý giá trị NULL. Dĩ nhiên cũng có thể dùng `case when`, về cơ bản tương tự. Điểm khó của bài nằm ở xử lý giá trị rỗng; các điều kiện truy vấn còn lại không quá khó.

**Đáp án**:

```sql
SELECT UID,
       round(avg(new_socre)) AS avg_score,
       round(avg(time_diff), 1) AS avg_time_took
FROM
  (SELECT er.uid,
          IF (er.submit_time IS NOT NULL, TIMESTAMPDIFF(MINUTE, start_time, submit_time), ef.duration) AS time_diff,
          IF (er.submit_time IS NOT NULL,er.score,0) AS new_socre
   FROM exam_record er
   LEFT JOIN user_info uf ON er.uid = uf.uid
   LEFT JOIN examination_info ef ON er.exam_id = ef.exam_id
   WHERE uf.LEVEL = 0 AND ef.difficulty = 'hard' ) t
GROUP BY UID
ORDER BY UID
```

## Câu lệnh điều kiện nâng cao

### Lọc người dùng theo nickname, điểm thành tích và ngày hoạt động được giới hạn (khá khó)

**Mô tả**:

Có bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` nickname, `achievement` điểm thành tích, `level` cấp độ, `job` hướng nghề nghiệp, `register_time` thời gian đăng ký):

| id  | uid  | nick_name      | achievement | level | job        | register_time       |
| --- | ---- | -------------- | ----------- | ----- | ---------- | ------------------- |
| 1   | 1001 | NiuKe số 1     | 1000        | 2     | Thuật toán | 2020-01-01 10:00:00 |
| 2   | 1002 | NiuKe số 2     | 1200        | 3     | Thuật toán | 2020-01-01 10:00:00 |
| 3   | 1003 | Tiến công số 3 | 2200        | 5     | Thuật toán | 2020-01-01 10:00:00 |
| 4   | 1004 | NiuKe số 4     | 2500        | 6     | Thuật toán | 2020-01-01 10:00:00 |
| 5   | 1005 | NiuKe số 5     | 3000        | 7     | C++        | 2020-01-01 10:00:00 |

Bảng ghi chép làm bài `exam_record` (`uid` ID người dùng, `exam_id` ID bài thi, `start_time` thời gian bắt đầu làm bài, `submit_time` thời gian nộp bài, `score` điểm):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-02 09:01:01 | 2020-01-02 09:21:59 | 80     |
| 3   | 1001 | 9002    | 2021-02-02 19:01:01 | 2021-02-02 19:30:01 | 87     |
| 2   | 1001 | 9001    | 2021-05-02 10:01:01 | (NULL)              | (NULL) |
| 4   | 1001 | 9001    | 2021-06-02 19:01:01 | 2021-06-02 19:32:00 | 20     |
| 6   | 1001 | 9002    | 2021-09-01 12:01:01 | (NULL)              | (NULL) |
| 5   | 1001 | 9002    | 2021-09-05 19:01:01 | 2021-09-05 19:40:01 | 89     |
| 11  | 1002 | 9001    | 2020-01-01 12:01:01 | 2020-01-01 12:31:01 | 81     |
| 12  | 1002 | 9002    | 2020-02-01 12:01:01 | 2020-02-01 12:31:01 | 82     |
| 13  | 1002 | 9002    | 2020-02-02 12:11:01 | 2020-02-02 12:31:01 | 83     |
| 7   | 1002 | 9002    | 2021-05-05 18:01:01 | 2021-05-05 18:59:02 | 90     |
| 16  | 1002 | 9001    | 2021-09-06 12:01:01 | 2021-09-06 12:21:01 | 80     |
| 17  | 1002 | 9001    | 2021-09-06 12:01:01 | (NULL)              | (NULL) |
| 18  | 1002 | 9001    | 2021-09-07 12:01:01 | (NULL)              | (NULL) |
| 8   | 1003 | 9003    | 2021-02-06 12:01:01 | (NULL)              | (NULL) |
| 9   | 1003 | 9001    | 2021-09-07 10:01:01 | 2021-09-07 10:31:01 | 89     |
| 10  | 1004 | 9002    | 2021-08-06 12:01:01 | (NULL)              | (NULL) |
| 14  | 1005 | 9001    | 2021-02-01 11:01:01 | 2021-02-01 11:31:01 | 84     |
| 15  | 1006 | 9001    | 2021-02-01 11:01:01 | 2021-02-01 11:31:01 | 84     |

Bảng ghi chép luyện tập `practice_record` (`uid` ID người dùng, `question_id` ID câu hỏi, `submit_time` thời gian nộp bài, `score` điểm):

| id  | uid  | question_id | submit_time         | score |
| --- | ---- | ----------- | ------------------- | ----- |
| 1   | 1001 | 8001        | 2021-08-02 11:41:01 | 60    |
| 2   | 1002 | 8001        | 2021-09-02 19:30:01 | 50    |
| 3   | 1002 | 8001        | 2021-09-02 19:20:01 | 70    |
| 4   | 1002 | 8002        | 2021-09-02 19:38:01 | 70    |
| 5   | 1003 | 8002        | 2021-09-01 19:38:01 | 80    |

Hãy tìm thông tin người dùng có nickname bắt đầu bằng “NiuKe”, kết thúc bằng “số”, điểm thành tích trong khoảng 1200~2500 và lần hoạt động gần nhất (trả lời câu hỏi hoặc làm bài thi) thuộc tháng 9 năm 2021.

Kết quả từ dữ liệu mẫu như sau:

| uid  | nick_name  | achievement |
| ---- | ---------- | ----------- |
| 1002 | NiuKe số 2 | 1200        |

**Giải thích**: Người dùng có nickname bắt đầu bằng “NiuKe”, kết thúc bằng “số” và điểm thành tích trong khoảng 1200~2500 là 1002, 1004;

Hoạt động gần nhất của 1002 ở khu vực bài thi là tháng 9 năm 2021, hoạt động gần nhất ở khu vực câu hỏi cũng là tháng 9 năm 2021; hoạt động gần nhất của 1004 ở khu vực bài thi là tháng 8 năm 2021, khu vực câu hỏi không có hoạt động.

Vì vậy chỉ 1002 thỏa mãn điều kiện cuối cùng.

**Cách làm**:

Trước tiên liệt kê các câu lệnh truy vấn chính theo điều kiện.

Nickname bắt đầu bằng “NiuKe”, kết thúc bằng “số”: `nick_name LIKE "NiuKe%số"`

Điểm thành tích trong khoảng 1200~2500: `achievement BETWEEN 1200 AND 2500`

Điều kiện thứ ba đã giới hạn tháng 9 nên có thể viết trực tiếp: `( date_format( record.submit_time, '%Y%m' )= 202109 OR date_format( pr.submit_time, '%Y%m' )= 202109 )`

**Đáp án**:

```sql
SELECT DISTINCT u_info.uid,
                u_info.nick_name,
                u_info.achievement
FROM user_info u_info
LEFT JOIN exam_record record ON record.uid = u_info.uid
LEFT JOIN practice_record pr ON u_info.uid = pr.uid
WHERE u_info.nick_name LIKE "NiuKe%số"
  AND u_info.achievement BETWEEN 1200
  AND 2500
  AND (date_format(record.submit_time, '%Y%m')= 202109
       OR date_format(pr.submit_time, '%Y%m')= 202109)
GROUP BY u_info.uid
```

### Lọc bản ghi làm bài theo quy tắc nickname và quy tắc bài thi (khá khó)

**Mô tả**:

Có bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` nickname, `achievement` điểm thành tích, `level` cấp độ, `job` hướng nghề nghiệp, `register_time` thời gian đăng ký):

| id  | uid  | nick_name     | achievement | level | job        | register_time       |
| --- | ---- | ------------- | ----------- | ----- | ---------- | ------------------- |
| 1   | 1001 | NiuKe số 1    | 1900        | 2     | Thuật toán | 2020-01-01 10:00:00 |
| 2   | 1002 | NiuKe số 2    | 1200        | 3     | Thuật toán | 2020-01-01 10:00:00 |
| 3   | 1003 | NiuKe số 3 ♂ | 2200        | 5     | Thuật toán | 2020-01-01 10:00:00 |
| 4   | 1004 | NiuKe số 4    | 2500        | 6     | Thuật toán | 2020-01-01 10:00:00 |
| 5   | 1005 | NiuKe số 555  | 2000        | 7     | C++        | 2020-01-01 10:00:00 |
| 6   | 1006 | 666666        | 3000        | 6     | C++        | 2020-01-01 10:00:00 |

Bảng thông tin bài thi `examination_info` (`exam_id` ID bài thi, `tag` loại bài thi, `difficulty` độ khó bài thi, `duration` thời lượng thi, `release_time` thời gian phát hành):

| id  | exam_id | tag | difficulty | duration | release_time        |
| --- | ------- | --- | ---------- | -------- | ------------------- |
| 1   | 9001    | C++ | hard       | 60       | 2020-01-01 10:00:00 |
| 2   | 9002    | c#  | hard       | 80       | 2020-01-01 10:00:00 |
| 3   | 9003    | SQL | medium     | 70       | 2020-01-01 10:00:00 |

Bảng ghi chép làm bài `exam_record` (`uid` ID người dùng, `exam_id` ID bài thi, `start_time` thời gian bắt đầu làm bài, `submit_time` thời gian nộp bài, `score` điểm):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-02 09:01:01 | 2020-01-02 09:21:59 | 80     |
| 2   | 1001 | 9001    | 2021-05-02 10:01:01 | (NULL)              | (NULL) |
| 4   | 1001 | 9001    | 2021-06-02 19:01:01 | 2021-06-02 19:32:00 | 20     |
| 3   | 1001 | 9002    | 2021-02-02 19:01:01 | 2021-02-02 19:30:01 | 87     |
| 5   | 1001 | 9002    | 2021-09-05 19:01:01 | 2021-09-05 19:40:01 | 89     |
| 6   | 1001 | 9002    | 2021-09-01 12:01:01 | (NULL)              | (NULL) |
| 11  | 1002 | 9001    | 2020-01-01 12:01:01 | 2020-01-01 12:31:01 | 81     |
| 16  | 1002 | 9001    | 2021-09-06 12:01:01 | 2021-09-06 12:21:01 | 80     |
| 17  | 1002 | 9001    | 2021-09-06 12:01:01 | (NULL)              | (NULL) |
| 18  | 1002 | 9001    | 2021-09-07 12:01:01 | (NULL)              | (NULL) |
| 7   | 1002 | 9002    | 2021-05-05 18:01:01 | 2021-05-05 18:59:02 | 90     |
| 12  | 1002 | 9002    | 2020-02-01 12:01:01 | 2020-02-01 12:31:01 | 82     |
| 13  | 1002 | 9002    | 2020-02-02 12:11:01 | 2020-02-02 12:31:01 | 83     |
| 9   | 1003 | 9001    | 2021-09-07 10:01:01 | 2021-09-07 10:31:01 | 89     |
| 8   | 1003 | 9003    | 2021-02-06 12:01:01 | (NULL)              | (NULL) |
| 10  | 1004 | 9002    | 2021-08-06 12:01:01 | (NULL)              | (NULL) |
| 14  | 1005 | 9001    | 2021-02-01 11:01:01 | 2021-02-01 11:31:01 | 84     |
| 15  | 1006 | 9001    | 2021-02-01 11:01:01 | 2021-09-01 11:31:01 | 84     |

Hãy tìm ID bài thi đã hoàn thành và điểm trung bình của người dùng có nickname gồm “NiuKe” + chữ số thuần + “số”, hoặc chỉ gồm chữ số, đối với loại bài thi bắt đầu bằng chữ c (như C, C++, c#), rồi sắp xếp tăng dần theo ID người dùng và điểm trung bình. Kết quả từ dữ liệu mẫu như sau:

| uid  | exam_id | avg_score |
| ---- | ------- | --------- |
| 1002 | 9001    | 81        |
| 1002 | 9002    | 85        |
| 1005 | 9001    | 84        |
| 1006 | 9001    | 84        |

Giải thích: Người dùng có nickname thỏa mãn điều kiện là 1002, 1004, 1005, 1006;

Bài thi bắt đầu bằng c là 9001, 9002;

Trong các bản ghi làm bài thỏa mãn điều kiện trên, điểm hoàn thành bài 9001 của 1002 là 81, 80, điểm trung bình là 81 (80.5 làm tròn là 81);

Điểm hoàn thành bài 9002 của 1002 là 90, 82, 83, điểm trung bình là 85;

**Cách làm**:

Vẫn như trước, vì đề bài đã đưa ra các điều kiện nên trước tiên viết từng điều kiện.

Tìm người dùng có nickname gồm “NiuKe” + chữ số thuần + “số”, hoặc chỉ gồm chữ số: Ban đầu có thể viết `nick_name LIKE 'NiuKe%số' OR nick_name REGEXP '^[0-9]+$'`, nhưng nếu trong bảng có “NiuKe H số” thì cũng sẽ khớp.

Vì vậy ở đây cần dùng regex: `nick_name LIKE '^NiuKe[0-9]+số'`

Loại bài thi bắt đầu bằng chữ c: `e_info.tag LIKE 'c%'` hoặc `tag regexp '^c|^C'`; cách thứ nhất cũng có thể khớp chữ C viết hoa.

**Đáp án**:

```sql
SELECT UID,
       exam_id,
       ROUND(AVG(score), 0) avg_score
FROM exam_record
WHERE UID IN
    (SELECT UID
     FROM user_info
     WHERE nick_name RLIKE "^NiuKe[0-9]+số $"
       OR nick_name RLIKE "^[0-9]+$")
  AND exam_id IN
    (SELECT exam_id
     FROM examination_info
     WHERE tag RLIKE "^[cC]")
  AND score IS NOT NULL
GROUP BY UID,exam_id
ORDER BY UID,avg_score;
```

### Xuất các trường hợp khác nhau tùy việc bản ghi chỉ định có tồn tại hay không (khó)

**Mô tả**:

Có bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` nickname, `achievement` điểm thành tích, `level` cấp độ, `job` hướng nghề nghiệp, `register_time` thời gian đăng ký):

| id  | uid  | nick_name      | achievement | level | job        | register_time       |
| --- | ---- | -------------- | ----------- | ----- | ---------- | ------------------- |
| 1   | 1001 | NiuKe số 1     | 19          | 0     | Thuật toán | 2020-01-01 10:00:00 |
| 2   | 1002 | NiuKe số 2     | 1200        | 3     | Thuật toán | 2020-01-01 10:00:00 |
| 3   | 1003 | Tiến công số 3 | 22          | 0     | Thuật toán | 2020-01-01 10:00:00 |
| 4   | 1004 | NiuKe số 4     | 25          | 0     | Thuật toán | 2020-01-01 10:00:00 |
| 5   | 1005 | NiuKe số 555   | 2000        | 7     | C++        | 2020-01-01 10:00:00 |
| 6   | 1006 | 666666         | 3000        | 6     | C++        | 2020-01-01 10:00:00 |

Bảng ghi chép làm bài `exam_record` (`uid` ID người dùng, `exam_id` ID bài thi, `start_time` thời gian bắt đầu làm bài, `submit_time` thời gian nộp bài, `score` điểm):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-02 09:01:01 | 2020-01-02 09:21:59 | 80     |
| 2   | 1001 | 9001    | 2021-05-02 10:01:01 | (NULL)              | (NULL) |
| 3   | 1001 | 9002    | 2021-02-02 19:01:01 | 2021-02-02 19:30:01 | 87     |
| 4   | 1001 | 9002    | 2021-09-01 12:01:01 | (NULL)              | (NULL) |
| 5   | 1001 | 9003    | 2021-09-02 12:01:01 | (NULL)              | (NULL) |
| 6   | 1001 | 9004    | 2021-09-03 12:01:01 | (NULL)              | (NULL) |
| 7   | 1002 | 9001    | 2020-01-01 12:01:01 | 2020-01-01 12:31:01 | 99     |
| 8   | 1002 | 9003    | 2020-02-01 12:01:01 | 2020-02-01 12:31:01 | 82     |
| 9   | 1002 | 9003    | 2020-02-02 12:11:01 | (NULL)              | (NULL) |
| 10  | 1002 | 9002    | 2021-05-05 18:01:01 | (NULL)              | (NULL) |
| 11  | 1002 | 9001    | 2021-09-06 12:01:01 | (NULL)              | (NULL) |
| 12  | 1003 | 9003    | 2021-02-06 12:01:01 | (NULL)              | (NULL) |
| 13  | 1003 | 9001    | 2021-09-07 10:01:01 | 2021-09-07 10:31:01 | 89     |

Hãy lọc dữ liệu trong bảng. Khi có bất kỳ người dùng cấp 0 nào có số bài thi chưa hoàn thành lớn hơn 2, hãy xuất số lượng và tỷ lệ bài thi chưa hoàn thành của từng người dùng cấp 0 (giữ lại 3 chữ số thập phân); nếu không có người dùng nào như vậy, hãy xuất hai chỉ số này của tất cả người dùng có bản ghi làm bài. Kết quả sắp xếp tăng dần theo tỷ lệ chưa hoàn thành.

Kết quả từ dữ liệu mẫu như sau:

| uid  | incomplete_cnt | incomplete_rate |
| ---- | -------------- | --------------- |
| 1004 | 0              | 0.000           |
| 1003 | 1              | 0.500           |
| 1001 | 4              | 0.667           |

**Giải thích**: Người dùng cấp 0 là 1001, 1003, 1004; số bài thi đã làm và số bài chưa hoàn thành của họ lần lượt là: 6:4, 2:1, 0:0;

Người dùng cấp 0 là 1001 có số bài chưa hoàn thành lớn hơn 2, vì vậy xuất số lượng chưa hoàn thành và tỷ lệ chưa hoàn thành của ba người dùng này (1004 chưa từng làm bài thi, tỷ lệ chưa hoàn thành mặc định là 0, giữ lại 3 chữ số thập phân thành 0.000);

Kết quả được sắp xếp tăng dần theo tỷ lệ chưa hoàn thành.

Phụ chú: Nếu 1001 không thỏa mãn điều kiện “số bài thi chưa hoàn thành lớn hơn 2”, cần xuất hai chỉ số này của 1001, 1002, 1003, vì bảng ghi chép làm bài chỉ có bản ghi làm bài của ba người dùng này.

**Cách làm**:

Trước tiên viết SQL có thể thỏa mãn điều kiện **“người dùng cấp 0 có số bài thi chưa hoàn thành lớn hơn 2”**:

```sql
SELECT ui.uid UID
FROM user_info ui
LEFT JOIN exam_record er ON ui.uid = er.uid
WHERE ui.uid IN
    (SELECT ui.uid
     FROM user_info ui
     LEFT JOIN exam_record er ON ui.uid = er.uid
     WHERE er.submit_time IS NULL
       AND ui.LEVEL = 0 )
GROUP BY ui.uid
HAVING sum(IF(er.submit_time IS NULL, 1, 0)) > 2
```

Sau đó viết riêng câu lệnh truy vấn SQL cho hai trường hợp:

Trường hợp 1. Truy vấn tỷ lệ bài thi chưa hoàn thành của người dùng cấp 0 thỏa mãn điều kiện

```sql
SELECT
	 tmp1.uid uid,
	sum(
	IF
	( er.submit_time IS NULL AND er.start_time IS NOT NULL, 1, 0 )) incomplete_cnt,
	round(
		sum(
		IF
		( er.submit_time IS NULL AND er.start_time IS NOT NULL, 1, 0 ))/ count( tmp1.uid ),
		3
	) incomplete_rate
FROM
	(
	SELECT DISTINCT
		ui.uid
	FROM
		user_info ui
		LEFT JOIN exam_record er ON ui.uid = er.uid
	WHERE
		er.submit_time IS NULL
		AND ui.LEVEL = 0
	) tmp1
	LEFT JOIN exam_record er ON tmp1.uid = er.uid
GROUP BY
	tmp1.uid
ORDER BY
	incomplete_rate
```

Trường hợp 2. Truy vấn tỷ lệ bài thi chưa hoàn thành của tất cả người dùng có bản ghi làm bài khi không tồn tại điều kiện yêu cầu

```sql
SELECT
	ui.uid uid,
	sum( CASE WHEN er.submit_time IS NULL AND er.start_time IS NOT NULL THEN 1 ELSE 0 END ) incomplete_cnt,
	round(
		sum(
		IF
		( er.submit_time IS NULL AND er.start_time IS NOT NULL, 1, 0 ))/ count( ui.uid ),
		3
	) incomplete_rate
FROM
	user_info ui
	JOIN exam_record er ON ui.uid = er.uid
GROUP BY
	ui.uid
ORDER BY
	incomplete_rate
```

Ghép lại chính là đáp án:

```sql
WITH host_user AS
  (SELECT ui.uid UID
   FROM user_info ui
   LEFT JOIN exam_record er ON ui.uid = er.uid
   WHERE ui.uid IN
       (SELECT ui.uid
        FROM user_info ui
        LEFT JOIN exam_record er ON ui.uid = er.uid
        WHERE er.submit_time IS NULL
          AND ui.LEVEL = 0 )
   GROUP BY ui.uid
   HAVING sum(IF (er.submit_time IS NULL, 1, 0))> 2),
     tt1 AS
  (SELECT tmp1.uid UID,
                   sum(IF (er.submit_time IS NULL
                           AND er.start_time IS NOT NULL, 1, 0)) incomplete_cnt,
                   round(sum(IF (er.submit_time IS NULL
                                 AND er.start_time IS NOT NULL, 1, 0))/ count(tmp1.uid), 3) incomplete_rate
   FROM
     (SELECT DISTINCT ui.uid
      FROM user_info ui
      LEFT JOIN exam_record er ON ui.uid = er.uid
      WHERE er.submit_time IS NULL
        AND ui.LEVEL = 0 ) tmp1
   LEFT JOIN exam_record er ON tmp1.uid = er.uid
   GROUP BY tmp1.uid
   ORDER BY incomplete_rate),
     tt2 AS
  (SELECT ui.uid UID,
                 sum(CASE
                         WHEN er.submit_time IS NULL
                              AND er.start_time IS NOT NULL THEN 1
                         ELSE 0
                     END) incomplete_cnt,
                 round(sum(IF (er.submit_time IS NULL
                               AND er.start_time IS NOT NULL, 1, 0))/ count(ui.uid), 3) incomplete_rate
   FROM user_info ui
   JOIN exam_record er ON ui.uid = er.uid
   GROUP BY ui.uid
   ORDER BY incomplete_rate)
  (SELECT tt1.*
   FROM tt1
   LEFT JOIN
     (SELECT UID
      FROM host_user) t1 ON 1 = 1
   WHERE t1.uid IS NOT NULL )
UNION ALL
  (SELECT tt2.*
   FROM tt2
   LEFT JOIN
     (SELECT UID
      FROM host_user) t2 ON 1 = 1
   WHERE t2.uid IS NULL)
```

Phiên bản V2 (cải tiến dựa trên phần trên, đáp án ngắn hơn và logic chặt chẽ hơn):

```sql
SELECT
	ui.uid,
	SUM(
	IF
	( start_time IS NOT NULL AND score IS NULL, 1, 0 )) AS incomplete_cnt,#3. Số bài thi chưa hoàn thành
	ROUND( AVG( IF ( start_time IS NOT NULL AND score IS NULL, 1, 0 )), 3 ) AS incomplete_rate #4. Tỷ lệ chưa hoàn thành

FROM
	user_info ui
	LEFT JOIN exam_record USING ( uid )
WHERE
CASE

		WHEN (#1. Khi có bất kỳ người dùng cấp 0 nào có số bài thi chưa hoàn thành lớn hơn 2
		SELECT
			MAX( lv0_incom_cnt )
		FROM
			(
			SELECT
				SUM(
				IF
				( score IS NULL, 1, 0 )) AS lv0_incom_cnt
			FROM
				user_info
				JOIN exam_record USING ( uid )
			WHERE
				LEVEL = 0
			GROUP BY
				uid
			) table1
			)> 2 THEN
			uid IN ( #1.1 Tìm từng người dùng cấp 0
			SELECT uid FROM user_info WHERE LEVEL = 0 ) ELSE uid IN ( #2. Nếu không tồn tại người dùng như vậy, tìm người dùng có bản ghi làm bài
			SELECT DISTINCT uid FROM exam_record )
		END
		GROUP BY
			ui.uid
		ORDER BY
		incomplete_rate #5. Kết quả sắp xếp tăng dần theo tỷ lệ chưa hoàn thành
```

### Tỷ lệ các mức điểm khác nhau theo cấp độ người dùng (khá khó)

**Mô tả**:

Có bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` nickname, `achievement` điểm thành tích, `level` cấp độ, `job` hướng nghề nghiệp, `register_time` thời gian đăng ký):

| id  | uid  | nick_name     | achievement | level | job        | register_time       |
| --- | ---- | ------------- | ----------- | ----- | ---------- | ------------------- |
| 1   | 1001 | NiuKe số 1    | 19          | 0     | Thuật toán | 2020-01-01 10:00:00 |
| 2   | 1002 | NiuKe số 2    | 1200        | 3     | Thuật toán | 2020-01-01 10:00:00 |
| 3   | 1003 | NiuKe số 3 ♂ | 22          | 0     | Thuật toán | 2020-01-01 10:00:00 |
| 4   | 1004 | NiuKe số 4    | 25          | 0     | Thuật toán | 2020-01-01 10:00:00 |
| 5   | 1005 | NiuKe số 555  | 2000        | 7     | C++        | 2020-01-01 10:00:00 |
| 6   | 1006 | 666666        | 3000        | 6     | C++        | 2020-01-01 10:00:00 |

Bảng ghi chép làm bài `exam_record` (`uid` ID người dùng, `exam_id` ID bài thi, `start_time` thời gian bắt đầu làm bài, `submit_time` thời gian nộp bài, `score` điểm):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-02 09:01:01 | 2020-01-02 09:21:59 | 80     |
| 2   | 1001 | 9001    | 2021-05-02 10:01:01 | (NULL)              | (NULL) |
| 3   | 1001 | 9002    | 2021-02-02 19:01:01 | 2021-02-02 19:30:01 | 75     |
| 4   | 1001 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:11:01 | 60     |
| 5   | 1001 | 9003    | 2021-09-02 12:01:01 | 2021-09-02 12:41:01 | 90     |
| 6   | 1001 | 9001    | 2021-06-02 19:01:01 | 2021-06-02 19:32:00 | 20     |
| 7   | 1001 | 9002    | 2021-09-05 19:01:01 | 2021-09-05 19:40:01 | 89     |
| 8   | 1001 | 9004    | 2021-09-03 12:01:01 | (NULL)              | (NULL) |
| 9   | 1002 | 9001    | 2020-01-01 12:01:01 | 2020-01-01 12:31:01 | 99     |
| 10  | 1002 | 9003    | 2020-02-01 12:01:01 | 2020-02-01 12:31:01 | 82     |
| 11  | 1002 | 9003    | 2020-02-02 12:11:01 | 2020-02-02 12:41:01 | 76     |

Để thể hiện định tính kết quả làm bài của người dùng, chúng ta chia điểm bài thi thành bốn mức Xuất sắc, Tốt, Trung bình, Kém theo các mốc [90,75,60] (mốc được tính vào khoảng bên trái). Hãy thống kê tỷ lệ từng mức điểm trong các bài thi đã hoàn thành của người dùng ở các cấp độ khác nhau (giữ lại 3 chữ số thập phân). Người dùng chưa từng hoàn thành bài thi không cần xuất, kết quả sắp xếp giảm dần theo cấp độ người dùng, sau đó giảm dần theo tỷ lệ.

Kết quả từ dữ liệu mẫu như sau:

| level | score_grade | ratio |
| ----- | ----------- | ----- |
| 3     | Tốt         | 0.667 |
| 3     | Xuất sắc    | 0.333 |
| 0     | Tốt         | 0.500 |
| 0     | Trung bình  | 0.167 |
| 0     | Xuất sắc    | 0.167 |
| 0     | Kém         | 0.167 |

Giải thích: Người dùng đã hoàn thành bài thi là 1001, 1002; cấp độ người dùng và mức điểm tương ứng của các bài thi đã hoàn thành như sau:

| uid  | exam_id | score | level | score_grade |
| ---- | ------- | ----- | ----- | ----------- |
| 1001 | 9001    | 80    | 0     | Tốt         |
| 1001 | 9002    | 75    | 0     | Tốt         |
| 1001 | 9002    | 60    | 0     | Trung bình  |
| 1001 | 9003    | 90    | 0     | Xuất sắc    |
| 1001 | 9001    | 20    | 0     | Kém         |
| 1001 | 9002    | 89    | 0     | Tốt         |
| 1002 | 9001    | 99    | 3     | Xuất sắc    |
| 1002 | 9003    | 82    | 3     | Tốt         |
| 1002 | 9003    | 76    | 3     | Tốt         |

Vì vậy tỷ lệ từng mức điểm của người dùng cấp 0 (chỉ có 1001) là: Xuất sắc 1/6, Tốt 1/6, Trung bình 1/6, Kém 3/6; tỷ lệ từng mức điểm của người dùng cấp 3 (chỉ có 1002) là: Xuất sắc 1/3, Tốt 2/3. Kết quả giữ lại 3 chữ số thập phân.

**Cách làm**:

Trước tiên viết điều kiện **“chia điểm bài thi thành bốn mức Xuất sắc, Tốt, Trung bình, Kém theo các mốc [90,75,60]”**, ở đây có thể dùng `case when`.

```sql
CASE
		WHEN a.score >= 90 THEN
		'Xuất sắc'
		WHEN a.score < 90 AND a.score >= 75 THEN
		'Tốt'
		WHEN a.score < 75 AND a.score >= 60 THEN
	'Trung bình' ELSE 'Kém'
END
```

Điểm mấu chốt của bài là phần này, những phần còn lại chỉ là ghép điều kiện.

**Đáp án**:

```sql
SELECT a.LEVEL,
       a.score_grade,
       ROUND(a.cur_count / b.total_num, 3) AS ratio
FROM
  (SELECT b.LEVEL AS LEVEL,
          (CASE
               WHEN a.score >= 90 THEN 'Xuất sắc'
               WHEN a.score < 90
                    AND a.score >= 75 THEN 'Tốt'
               WHEN a.score < 75
                    AND a.score >= 60 THEN 'Trung bình'
               ELSE 'Kém'
           END) AS score_grade,
          count(1) AS cur_count
   FROM exam_record a
   LEFT JOIN user_info b ON a.uid = b.uid
   WHERE a.submit_time IS NOT NULL
   GROUP BY b.LEVEL,
            score_grade) a
LEFT JOIN
  (SELECT b.LEVEL AS LEVEL,
          count(b.LEVEL) AS total_num
   FROM exam_record a
   LEFT JOIN user_info b ON a.uid = b.uid
   WHERE a.submit_time IS NOT NULL
   GROUP BY b.LEVEL) b ON a.LEVEL = b.LEVEL
ORDER BY a.LEVEL DESC,
         ratio DESC
```

## Truy vấn giới hạn

### Ba người đăng ký sớm nhất

**Mô tả**:

Có bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` nickname, `achievement` điểm thành tích, `level` cấp độ, `job` hướng nghề nghiệp, `register_time` thời gian đăng ký):

| id  | uid  | nick_name     | achievement | level | job        | register_time       |
| --- | ---- | ------------- | ----------- | ----- | ---------- | ------------------- |
| 1   | 1001 | NiuKe số 1    | 19          | 0     | Thuật toán | 2020-01-01 10:00:00 |
| 2   | 1002 | NiuKe số 2    | 1200        | 3     | Thuật toán | 2020-02-01 10:00:00 |
| 3   | 1003 | NiuKe số 3 ♂ | 22          | 0     | Thuật toán | 2020-01-02 10:00:00 |
| 4   | 1004 | NiuKe số 4    | 25          | 0     | Thuật toán | 2020-01-02 11:00:00 |
| 5   | 1005 | NiuKe số 555  | 4000        | 7     | C++        | 2020-01-11 10:00:00 |
| 6   | 1006 | 666666        | 3000        | 6     | C++        | 2020-11-01 10:00:00 |

Hãy tìm 3 người có thời gian đăng ký sớm nhất. Kết quả từ dữ liệu mẫu như sau:

| uid  | nick_name  | register_time       |
| ---- | ---------- | ------------------- |
| 1001 | NiuKe số 1 | 2020-01-01 10:00:00 |
| 1003 | NiuKe số 3 | 2020-01-02 10:00:00 |
| 1004 | NiuKe số 4 | 2020-01-02 11:00:00 |

Giải thích: Sau khi sắp xếp theo thời gian đăng ký, chọn ba người đầu tiên và xuất ID người dùng, nickname, thời gian đăng ký.

**Đáp án**:

```sql
SELECT uid, nick_name, register_time
    FROM user_info
    ORDER BY register_time
    LIMIT 3
```

### Danh sách trang thứ ba những người hoàn thành bài thi ngay trong ngày đăng ký (khá khó)

**Mô tả**: Có bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` nickname, `achievement` điểm thành tích, `level` cấp độ, `job` hướng nghề nghiệp, `register_time` thời gian đăng ký):

| id  | uid  | nick_name     | achievement | level | job        | register_time       |
| --- | ---- | ------------- | ----------- | ----- | ---------- | ------------------- |
| 1   | 1001 | NiuKe số 1    | 19          | 0     | Thuật toán | 2020-01-01 10:00:00 |
| 2   | 1002 | NiuKe số 2    | 1200        | 3     | Thuật toán | 2020-01-01 10:00:00 |
| 3   | 1003 | NiuKe số 3 ♂ | 22          | 0     | Thuật toán | 2020-01-01 10:00:00 |
| 4   | 1004 | NiuKe số 4    | 25          | 0     | Thuật toán | 2020-01-01 10:00:00 |
| 5   | 1005 | NiuKe số 555  | 4000        | 7     | Thuật toán | 2020-01-11 10:00:00 |
| 6   | 1006 | NiuKe số 6    | 25          | 0     | Thuật toán | 2020-01-02 11:00:00 |
| 7   | 1007 | NiuKe số 7    | 25          | 0     | Thuật toán | 2020-01-02 11:00:00 |
| 8   | 1008 | NiuKe số 8    | 25          | 0     | Thuật toán | 2020-01-02 11:00:00 |
| 9   | 1009 | NiuKe số 9    | 25          | 0     | Thuật toán | 2020-01-02 11:00:00 |
| 10  | 1010 | NiuKe số 10   | 25          | 0     | Thuật toán | 2020-01-02 11:00:00 |
| 11  | 1011 | 666666        | 3000        | 6     | C++        | 2020-01-02 10:00:00 |

Bảng thông tin bài thi `examination_info` (`exam_id` ID bài thi, `tag` loại bài thi, `difficulty` độ khó bài thi, `duration` thời lượng thi, `release_time` thời gian phát hành):

| id  | exam_id | tag        | difficulty | duration | release_time        |
| --- | ------- | ---------- | ---------- | -------- | ------------------- |
| 1   | 9001    | Thuật toán | hard       | 60       | 2020-01-01 10:00:00 |
| 2   | 9002    | Thuật toán | hard       | 80       | 2020-01-01 10:00:00 |
| 3   | 9003    | SQL        | medium     | 70       | 2020-01-01 10:00:00 |

Bảng ghi chép làm bài `exam_record` (`uid` ID người dùng, `exam_id` ID bài thi, `start_time` thời gian bắt đầu làm bài, `submit_time` thời gian nộp bài, `score` điểm):

| id  | uid  | exam_id | start_time          | submit_time         | score |
| --- | ---- | ------- | ------------------- | ------------------- | ----- |
| 1   | 1001 | 9001    | 2020-01-02 09:01:01 | 2020-01-02 09:21:59 | 80    |
| 2   | 1002 | 9003    | 2020-01-20 10:01:01 | 2020-01-20 10:10:01 | 81    |
| 3   | 1002 | 9002    | 2020-01-01 12:11:01 | 2020-01-01 12:31:01 | 83    |
| 4   | 1003 | 9002    | 2020-01-01 19:01:01 | 2020-01-01 19:30:01 | 75    |
| 5   | 1004 | 9002    | 2020-01-01 12:01:01 | 2020-01-01 12:11:01 | 60    |
| 6   | 1005 | 9002    | 2020-01-01 12:01:01 | 2020-01-01 12:41:01 | 90    |
| 7   | 1006 | 9001    | 2020-01-02 19:01:01 | 2020-01-02 19:32:00 | 20    |
| 8   | 1007 | 9002    | 2020-01-02 19:01:01 | 2020-01-02 19:40:01 | 89    |
| 9   | 1008 | 9003    | 2020-01-02 12:01:01 | 2020-01-02 12:20:01 | 99    |
| 10  | 1008 | 9001    | 2020-01-02 12:01:01 | 2020-01-02 12:31:01 | 98    |
| 11  | 1009 | 9002    | 2020-01-02 12:01:01 | 2020-01-02 12:31:01 | 82    |
| 12  | 1010 | 9002    | 2020-01-02 12:11:01 | 2020-01-02 12:41:01 | 76    |
| 13  | 1011 | 9001    | 2020-01-02 10:01:01 | 2020-01-02 10:31:01 | 89    |

![](https://oss.javaguide.cn/github/javaguide/database/sql/D2B491866B85826119EE3474F10D3636.png)

Hãy tìm những người có hướng nghề nghiệp là kỹ sư thuật toán và đã hoàn thành bài thi loại thuật toán ngay trong ngày đăng ký, rồi xếp hạng theo điểm cao nhất trong tất cả các kỳ thi đã tham gia. Bảng xếp hạng rất dài nên sẽ được phân trang, mỗi trang 3 bản ghi; cần lấy thông tin của người ở trang thứ 3 (trang bắt đầu từ 1).

Kết quả từ dữ liệu mẫu như sau:

| uid  | level | register_time       | max_score |
| ---- | ----- | ------------------- | --------- |
| 1010 | 0     | 2020-01-02 11:00:00 | 76        |
| 1003 | 0     | 2020-01-01 10:00:00 | 75        |
| 1004 | 0     | 2020-01-01 11:00:00 | 60        |

Giải thích: Ngoài 1011, hướng nghề nghiệp của các người dùng khác đều là kỹ sư thuật toán; bài thi loại thuật toán là 9001 và 9002, 11 người dùng đều hoàn thành bài thi loại thuật toán ngay trong ngày đăng ký; khi tính điểm cao nhất của tất cả kỳ thi, chỉ 1002 và 1008 hoàn thành hai kỳ thi, điểm cao nhất của hai kỳ thi của 1002 là 81, của 1008 là 99.

Xếp hạng theo điểm cao nhất như sau:

| uid  | level | register_time       | max_score |
| ---- | ----- | ------------------- | --------- |
| 1008 | 0     | 2020-01-02 11:00:00 | 99        |
| 1005 | 7     | 2020-01-01 10:00:00 | 90        |
| 1007 | 0     | 2020-01-02 11:00:00 | 89        |
| 1002 | 3     | 2020-01-01 10:00:00 | 83        |
| 1009 | 0     | 2020-01-02 11:00:00 | 82        |
| 1001 | 0     | 2020-01-01 10:00:00 | 80        |
| 1010 | 0     | 2020-01-02 11:00:00 | 76        |
| 1003 | 0     | 2020-01-01 10:00:00 | 75        |
| 1004 | 0     | 2020-01-01 11:00:00 | 60        |
| 1006 | 0     | 2020-01-02 11:00:00 | 20        |

Mỗi trang có 3 bản ghi, trang thứ ba là các bản ghi thứ 7~9, nên chỉ cần trả về các dòng của 1010, 1003, 1004.

**Cách làm**:

1. Mỗi trang có ba bản ghi, để lấy thông tin người ở trang thứ ba cần dùng `limit`.

2. Thống kê **thông tin và điểm của từng bản ghi** của những người có hướng nghề nghiệp là kỹ sư thuật toán và đã hoàn thành bài thi loại thuật toán ngay trong ngày đăng ký: trước tiên tìm người dùng thỏa mãn điều kiện, sau đó dùng left join để nối và tìm thông tin cùng điểm của từng bản ghi.

**Đáp án**:

```sql
SELECT t1.uid,
       LEVEL,
       register_time,
       max(score) AS max_score
FROM exam_record t
JOIN examination_info USING (exam_id)
JOIN user_info t1 ON t.uid = t1.uid
AND date(t.submit_time) = date(t1.register_time)
WHERE job = 'Thuật toán'
  AND tag = 'Thuật toán'
GROUP BY t1.uid,
         LEVEL,
         register_time
ORDER BY max_score DESC
LIMIT 6,3
```

## Hàm chuyển đổi văn bản

### Sửa bản ghi bị gộp cột

**Mô tả**: Có bảng thông tin bài thi `examination_info` (`exam_id` ID bài thi, `tag` loại bài thi, `difficulty` độ khó bài thi, `duration` thời lượng thi, `release_time` thời gian phát hành):

| id  | exam_id | tag                  | difficulty | duration | release_time        |
| --- | ------- | -------------------- | ---------- | -------- | ------------------- |
| 1   | 9001    | Thuật toán           | hard       | 60       | 2021-01-01 10:00:00 |
| 2   | 9002    | Thuật toán           | hard       | 80       | 2021-01-01 10:00:00 |
| 3   | 9003    | SQL                  | medium     | 70       | 2021-01-01 10:00:00 |
| 4   | 9004    | Thuật toán,medium,80 |            | 0        | 2021-01-01 10:00:00 |

Người nhập đề đã lỡ nhập đồng thời loại câu hỏi `tag`, độ khó và thời lượng của một số bản ghi vào trường `tag`. Hãy tìm các bản ghi nhập sai này, tách chúng rồi xuất theo đúng loại cột.

Kết quả từ dữ liệu mẫu như sau:

| exam_id | tag        | difficulty | duration |
| ------- | ---------- | ---------- | -------- |
| 9004    | Thuật toán | medium     | 80       |

**Cách làm**:

Trước tiên tìm hiểu hàm cần dùng trong bài này.

Hàm `SUBSTRING_INDEX` dùng để trích xuất một phần của chuỗi theo dấu phân cách được chỉ định. Hàm nhận ba tham số: chuỗi gốc, dấu phân cách và số lượng phần cần trả về.

Cú pháp của hàm `SUBSTRING_INDEX`:

```sql
SUBSTRING_INDEX(str, delimiter, count)
```

- `str`: chuỗi gốc cần tách.
- `delimiter`: chuỗi hoặc ký tự dùng làm dấu phân cách.
- `count`: số lượng phần cần trả về.
  - Nếu `count` lớn hơn 0, trả về `count` phần đầu tính từ bên trái (ranh giới là dấu phân cách).
  - Nếu `count` nhỏ hơn 0, trả về `count` phần đầu tính từ bên phải (ranh giới là dấu phân cách), tức đếm từ phải sang trái.

Dưới đây là một số ví dụ minh họa cách dùng hàm `SUBSTRING_INDEX`:

1. Trích xuất phần đầu tiên của chuỗi:

   ```sql
   SELECT SUBSTRING_INDEX('apple,banana,cherry', ',', 1);
   -- Kết quả: 'apple'
   ```

2. Trích xuất phần cuối cùng của chuỗi:

   ```sql
   SELECT SUBSTRING_INDEX('apple,banana,cherry', ',', -1);
   -- Kết quả: 'cherry'
   ```

3. Trích xuất hai phần đầu tiên của chuỗi:

   ```sql
   SELECT SUBSTRING_INDEX('apple,banana,cherry', ',', 2);
   -- Kết quả: 'apple,banana'
   ```

4. Trích xuất hai phần cuối cùng của chuỗi:

   ```sql
   SELECT SUBSTRING_INDEX('apple,banana,cherry', ',', -2);
   -- Kết quả: 'banana,cherry'
   ```

**Đáp án**:

```sql
SELECT
	exam_id,
	substring_index( tag, ',', 1 ) tag,
	substring_index( substring_index( tag, ',', 2 ), ',',- 1 ) difficulty,
	substring_index( tag, ',',- 1 ) duration
FROM
	examination_info
WHERE
	difficulty = ''
```

### Xử lý nickname quá dài bằng cách cắt ngắn

**Mô tả**: Có bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` nickname, `achievement` điểm thành tích, `level` cấp độ, `job` hướng nghề nghiệp, `register_time` thời gian đăng ký):

| id  | uid  | nick_name               | achievement | level | job        | register_time       |
| --- | ---- | ----------------------- | ----------- | ----- | ---------- | ------------------- |
| 1   | 1001 | NiuKe số 1              | 19          | 0     | Thuật toán | 2020-01-01 10:00:00 |
| 2   | 1002 | NiuKe số 2              | 1200        | 3     | Thuật toán | 2020-01-01 10:00:00 |
| 3   | 1003 | NiuKe số 3 ♂           | 22          | 0     | Thuật toán | 2020-01-01 10:00:00 |
| 4   | 1004 | NiuKe số 4              | 25          | 0     | Thuật toán | 2020-01-01 11:00:00 |
| 5   | 1005 | NiuKe số 5678901234     | 4000        | 7     | Thuật toán | 2020-01-11 10:00:00 |
| 6   | 1006 | NiuKe số 67890123456789 | 25          | 0     | Thuật toán | 2020-01-02 11:00:00 |

Nickname của một số người dùng đặc biệt dài, gây rối kiểu hiển thị trong một số trường hợp. Vì vậy cần chuyển đổi nickname quá dài trước khi xuất. Hãy xuất thông tin người dùng có số ký tự lớn hơn 10; với người có số ký tự lớn hơn 13, xuất 10 ký tự đầu rồi thêm ba dấu chấm: `...`.

Kết quả từ dữ liệu mẫu như sau:

| uid  | nick_name            |
| ---- | -------------------- |
| 1005 | NiuKe số 5678901234  |
| 1006 | NiuKe số 67890123... |

Giải thích: Người dùng có số ký tự lớn hơn 10 là 1005 và 1006, độ dài lần lượt là 13, 17; vì vậy cần cắt nickname của 1006 trước khi xuất.

**Cách làm**:

Bài này liên quan đến việc đếm ký tự. Để tính số ký tự của chuỗi (tức độ dài chuỗi), có thể dùng hàm `LENGTH` hoặc `CHAR_LENGTH`. Hai hàm này khác nhau ở cách xử lý ký tự nhiều byte.

1. Hàm `LENGTH`: trả về số byte của chuỗi cho trước. Với chuỗi chứa ký tự nhiều byte, mỗi ký tự được tính như một byte.

Ví dụ:

```sql
SELECT LENGTH('ê'); -- Kết quả: 2, vì ký tự 'ê' chiếm 2 byte trong UTF-8
```

1. Hàm `CHAR_LENGTH`: trả về số ký tự của chuỗi cho trước. Với chuỗi chứa ký tự nhiều byte, mỗi ký tự được tính là một ký tự.

Ví dụ:

```sql
SELECT CHAR_LENGTH('ê'); -- Kết quả: 1, vì 'ê' chỉ có một ký tự
```

**Đáp án**:

```sql
SELECT
	uid,
CASE

		WHEN CHAR_LENGTH( nick_name ) > 13 THEN
		CONCAT( SUBSTR( nick_name, 1, 10 ), '...' ) ELSE nick_name
	END AS nick_name
FROM
	user_info
WHERE
	CHAR_LENGTH( nick_name ) > 10
GROUP BY
	uid;
```

### Lọc và thống kê khi chữ hoa chữ thường lẫn lộn (khá khó)

**Mô tả**:

Có bảng thông tin bài thi `examination_info` (`exam_id` ID bài thi, `tag` loại bài thi, `difficulty` độ khó bài thi, `duration` thời lượng thi, `release_time` thời gian phát hành):

| id  | exam_id | tag        | difficulty | duration | release_time        |
| --- | ------- | ---------- | ---------- | -------- | ------------------- |
| 1   | 9001    | Thuật toán | hard       | 60       | 2021-01-01 10:00:00 |
| 2   | 9002    | C++        | hard       | 80       | 2021-01-01 10:00:00 |
| 3   | 9003    | C++        | hard       | 80       | 2021-01-01 10:00:00 |
| 4   | 9004    | sql        | medium     | 70       | 2021-01-01 10:00:00 |
| 5   | 9005    | C++        | hard       | 80       | 2021-01-01 10:00:00 |
| 6   | 9006    | C++        | hard       | 80       | 2021-01-01 10:00:00 |
| 7   | 9007    | C++        | hard       | 80       | 2021-01-01 10:00:00 |
| 8   | 9008    | SQL        | medium     | 70       | 2021-01-01 10:00:00 |
| 9   | 9009    | SQL        | medium     | 70       | 2021-01-01 10:00:00 |
| 10  | 9010    | SQL        | medium     | 70       | 2021-01-01 10:00:00 |

Bảng thông tin làm bài `exam_record` (`uid` ID người dùng, `exam_id` ID bài thi, `start_time` thời gian bắt đầu làm bài, `submit_time` thời gian nộp bài, `score` điểm):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-01 09:01:01 | 2020-01-01 09:21:59 | 80     |
| 2   | 1002 | 9003    | 2020-01-20 10:01:01 | 2020-01-20 10:10:01 | 81     |
| 3   | 1002 | 9002    | 2020-02-01 12:11:01 | 2020-02-01 12:31:01 | 83     |
| 4   | 1003 | 9002    | 2020-03-01 19:01:01 | 2020-03-01 19:30:01 | 75     |
| 5   | 1004 | 9002    | 2020-03-01 12:01:01 | 2020-03-01 12:11:01 | 60     |
| 6   | 1005 | 9002    | 2020-03-01 12:01:01 | 2020-03-01 12:41:01 | 90     |
| 7   | 1006 | 9001    | 2020-05-02 19:01:01 | 2020-05-02 19:32:00 | 20     |
| 8   | 1007 | 9003    | 2020-01-02 19:01:01 | 2020-01-02 19:40:01 | 89     |
| 9   | 1008 | 9004    | 2020-02-02 12:01:01 | 2020-02-02 12:20:01 | 99     |
| 10  | 1008 | 9001    | 2020-02-02 12:01:01 | 2020-02-02 12:31:01 | 98     |
| 11  | 1009 | 9002    | 2020-02-02 12:01:01 | 2020-01-02 12:43:01 | 81     |
| 12  | 1010 | 9001    | 2020-01-02 12:11:01 | (NULL)              | (NULL) |
| 13  | 1010 | 9001    | 2020-02-02 12:01:01 | 2020-01-02 10:31:01 | 89     |

Loại bài thi `tag` có thể bị lẫn chữ hoa chữ thường. Trước tiên hãy lọc các loại `tag` có số lần làm bài nhỏ hơn 3, sau đó thống kê số lần làm bài ban đầu tương ứng sau khi chuyển chúng thành chữ hoa.

Nếu `tag` không thay đổi sau khi chuyển đổi thì không xuất kết quả đó.

Kết quả từ dữ liệu mẫu như sau:

| tag | answer_cnt |
| --- | ---------- |
| C++ | 6          |

Giải thích: Các bài thi đã được làm là 9001, 9002, 9003, 9004; loại `tag` và số lần làm bài của chúng như sau:

| exam_id | tag        | answer_cnt |
| ------- | ---------- | ---------- |
| 9001    | Thuật toán | 4          |
| 9002    | C++        | 6          |
| 9003    | c++        | 2          |
| 9004    | sql        | 2          |

Các `tag` có số lần làm bài nhỏ hơn 3 là c++ và sql. Sau khi chuyển thành chữ hoa, chỉ C++ đã có số lần làm bài, vì vậy xuất số lần làm bài sau khi chuyển c++ thành chữ hoa là 6.

**Cách làm**:

Trước hết, bài này hơi không nhất quán: theo dữ liệu mẫu, 9004 chỉ có 1 lần, nhưng ở đây hiển thị là 2 lần.

Xem các hàm chuyển đổi chữ hoa chữ thường:

1. Hàm `UPPER(s)` hoặc `UCASE(s)` chuyển toàn bộ ký tự chữ cái trong chuỗi s thành chữ hoa;

2. Hàm `LOWER(s)` hoặc `LCASE(s)` chuyển toàn bộ ký tự chữ cái trong chuỗi s thành chữ thường.

Điểm khó là nối cùng một bảng để truy vấn các giá trị khác nhau.

**Đáp án**:

```sql
WITH a AS
  (SELECT tag,
          COUNT(start_time) AS answer_cnt
   FROM exam_record er
   JOIN examination_info ei ON er.exam_id = ei.exam_id
   GROUP BY tag)
SELECT a.tag,
       b.answer_cnt
FROM a
INNER JOIN a AS b ON UPPER(a.tag)= b.tag # a viết thường, b viết hoa
AND a.tag != b.tag
WHERE a.answer_cnt < 3;
```

<!-- @include: @article-footer.snippet.md -->
