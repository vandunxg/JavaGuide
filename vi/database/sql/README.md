---
title: "Chuyên đề SQL: Nền tảng cú pháp, truy vấn, tổng hợp, kết nối, truy vấn con và câu hỏi phỏng vấn thường gặp"
description: "Lộ trình học nền tảng SQL và database cho phỏng vấn, bao quát truy vấn SQL, lọc, sắp xếp, tổng hợp, phân nhóm, kết nối, truy vấn con, thêm sửa xoá, constraint, transaction và các câu hỏi phỏng vấn SQL thường gặp."
category: Database
tag:
  - SQL
  - Database
  - Phỏng vấn backend
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: SQL,câu hỏi phỏng vấn SQL,cú pháp SQL,truy vấn SQL,tổng hợp SQL,kết nối SQL,truy vấn con SQL,nền tảng database,phỏng vấn backend
---

SQL là nền tảng database mà backend developer không thể bỏ qua. Dù học tiếp về index của MySQL, execution plan hay tối ưu SQL chậm, bạn đều cần nắm vững các ngữ nghĩa cơ bản như truy vấn, lọc, tổng hợp, kết nối, truy vấn con và sửa dữ liệu.

## Dành cho ai

- Backend developer đang học nền tảng database và cú pháp SQL.
- Người chuẩn bị cho câu hỏi phỏng vấn về nền tảng SQL, bài truy vấn SQL và CRUD database.
- Người đã viết SQL đơn giản nhưng chưa thành thạo JOIN, GROUP BY, HAVING, truy vấn con và thứ tự thực thi.
- Engineer muốn củng cố nền tảng SQL trước khi học index của MySQL và tối ưu SQL.

## Trọng tâm học

- Nên hiểu vai trò và thứ tự thực thi của SELECT, WHERE, ORDER BY, LIMIT, GROUP BY, HAVING như thế nào?
- INNER JOIN, LEFT JOIN, RIGHT JOIN, UNION và truy vấn con lần lượt phù hợp với những trường hợp sử dụng nào?
- Kết hợp hàm tổng hợp, thống kê theo nhóm và lọc theo điều kiện như thế nào?
- Cách viết INSERT, UPDATE, DELETE có những ranh giới nào dễ bị bỏ sót?
- Nên phân tích câu hỏi SQL trong phỏng vấn theo quan hệ giữa các bảng, điều kiện lọc, chiều thống kê tổng hợp và sắp xếp phân trang như thế nào?

## Thứ tự đọc đề xuất

1. [Tổng hợp kiến thức nền tảng cú pháp SQL](./sql-syntax-summary.md): trước tiên nắm có hệ thống cú pháp cơ bản và các thao tác thường gặp của SQL.
2. [Tổng hợp câu hỏi phỏng vấn SQL thường gặp (1)](./sql-questions-01.md), [Tổng hợp câu hỏi phỏng vấn SQL thường gặp (2)](./sql-questions-02.md): luyện tập truy vấn cơ bản, sắp xếp, tổng hợp và các hàm thường gặp.
3. [Tổng hợp câu hỏi phỏng vấn SQL thường gặp (3)](./sql-questions-03.md): tiếp tục bổ sung tư duy về kết nối, truy vấn con và truy vấn phức tạp.
4. [Tổng hợp câu hỏi phỏng vấn SQL thường gặp (4)](./sql-questions-04.md), [Tổng hợp câu hỏi phỏng vấn SQL thường gặp (5)](./sql-questions-05.md): củng cố khả năng phân tích truy vấn qua nhiều bài tập hơn.
5. Sau khi học xong nền tảng SQL, bạn nên tiếp tục đọc [Chuyên đề MySQL](../mysql/) để kết hợp cách viết SQL với index và execution plan.

## Bài viết cốt lõi

- [Tổng hợp kiến thức nền tảng cú pháp SQL](./sql-syntax-summary.md): bao quát cú pháp cơ bản về truy vấn, lọc, sắp xếp, tổng hợp, phân nhóm, kết nối, truy vấn con, thêm, sửa, xoá và constraint.
- [Tổng hợp câu hỏi phỏng vấn SQL thường gặp (1)](./sql-questions-01.md): làm quen với truy vấn, sắp xếp và lọc đơn giản qua các bài cơ bản.
- [Tổng hợp câu hỏi phỏng vấn SQL thường gặp (2)](./sql-questions-02.md): tiếp tục luyện tập cách dùng hàm, xử lý chuỗi, xử lý ngày tháng và các cách viết truy vấn thường gặp.
- [Tổng hợp câu hỏi phỏng vấn SQL thường gặp (3)](./sql-questions-03.md): phù hợp để rèn luyện truy vấn nhiều bảng, thống kê theo nhóm và phân tích truy vấn con.
- [Tổng hợp câu hỏi phỏng vấn SQL thường gặp (4)](./sql-questions-04.md): bổ sung nhiều câu hỏi phỏng vấn SQL thường gặp và hướng giải quyết.
- [Tổng hợp câu hỏi phỏng vấn SQL thường gặp (5)](./sql-questions-05.md): tiếp tục củng cố năng lực vận dụng tổng hợp trong các bài truy vấn SQL.

## Câu hỏi thường gặp

- Thứ tự thực thi logic của câu lệnh truy vấn SQL là gì?
- WHERE và HAVING khác nhau như thế nào?
- INNER JOIN và LEFT JOIN khác nhau như thế nào?
- UNION và UNION ALL khác nhau như thế nào?
- COUNT(\*), COUNT(1), COUNT(tên cột) khác nhau như thế nào?
- Nên lựa chọn truy vấn con hay JOIN như thế nào?
- Vì sao sau GROUP BY chỉ có thể chọn cột phân nhóm hoặc kết quả tổng hợp?
- Truy vấn phân trang có những cách viết thường gặp nào?
- Vì sao UPDATE và DELETE luôn cần thận trọng khi thêm điều kiện lọc?
- Nên phân tích bài SQL dựa trên quan hệ giữa các bảng như thế nào?

## Chuyên đề liên quan

- [Hệ thống kiến thức database](../)
- [Chuyên đề MySQL](../mysql/)
- [Hệ thống kiến thức về high-performance](../../high-performance/)
- [Tổng hợp các phương pháp tối ưu SQL thường gặp](../../high-performance/sql-optimization.md)

<!-- @include: @article-footer.snippet.md -->
