---
title: Tổng hợp kiến thức cơ bản về cú pháp SQL
description: Tổng hợp kiến thức cơ bản về cú pháp SQL, giải thích có hệ thống DDL về định nghĩa dữ liệu, DML về thao tác dữ liệu, DQL về truy vấn dữ liệu, DCL về ngôn ngữ điều khiển dữ liệu, bao quát các kiến thức cốt lõi như thao tác bảng, constraint, index, transaction và truy vấn JOIN.
category: Database
tag:
  - Database Basics
  - SQL
head:
  - - meta
    - name: keywords
      content: SQL syntax,DDL,DML,DQL,DCL,CREATE,SELECT,INSERT,UPDATE,DELETE,JOIN,truy vấn kết hợp,subquery
---

> Bài viết này được hoàn thiện dựa trên hai tài liệu sau:
>
> - [Sổ tay cấp tốc về cú pháp SQL](https://juejin.cn/post/6844903790571700231)
> - [Tutorial MySQL toàn diện](https://www.begtut.com/mysql/mysql-tutorial.html)

## Khái niệm cơ bản

### Thuật ngữ cơ sở dữ liệu

- `database` - Container lưu trữ dữ liệu có tổ chức (thường là một file hoặc một nhóm file).
- `table` - Danh sách có cấu trúc của một loại dữ liệu cụ thể.
- `schema` - Thông tin về bố cục và đặc tính của database và table. Schema định nghĩa dữ liệu được lưu trữ trong table như thế nào, bao gồm lưu loại dữ liệu nào, dữ liệu được phân tách ra sao, từng phần thông tin được đặt tên thế nào. Database và table đều có schema.
- `column` - Một field trong table. Mọi table đều gồm một hoặc nhiều column.
- `row` - Một record trong table.
- `primary key` - Một column (hoặc một nhóm column), có giá trị nhận diện duy nhất từng row trong table.

### Cú pháp SQL

SQL (Structured Query Language), SQL tiêu chuẩn do ủy ban tiêu chuẩn ANSI quản lý, vì vậy còn được gọi là ANSI SQL. Mỗi DBMS đều có implementation riêng, chẳng hạn PL/SQL, Transact-SQL.

#### Cấu trúc cú pháp SQL

![](https://oss.javaguide.cn/p3-juejin/cb684d4c75fc430e92aaee226069c7da~tplv-k3u1fbpfcp-zoom-1.png)

Cấu trúc cú pháp SQL bao gồm:

- **`clause`** - Thành phần cấu tạo của statement và query (trong một số trường hợp, các thành phần này là tùy chọn).
- **`expression`** - Có thể tạo ra bất kỳ scalar value nào, hoặc được tạo thành từ column và row của table trong database.
- **`predicate`** - Chỉ định điều kiện cho logic ba giá trị (3VL) (true/false/unknown) hoặc giá trị Boolean cần đánh giá, đồng thời giới hạn hiệu lực của statement và query hoặc thay đổi luồng thực thi của chương trình.
- **`query`** - Truy xuất dữ liệu dựa trên điều kiện cụ thể. Đây là một thành phần quan trọng của SQL.
- **`statement`** - Có thể tác động lâu dài đến schema và dữ liệu, đồng thời có thể điều khiển transaction, luồng thực thi chương trình, connection, session hoặc chẩn đoán của database.

#### Điểm chính của cú pháp SQL

- **SQL statement không phân biệt chữ hoa chữ thường**, nhưng tên database table, column và value có phân biệt hay không còn phụ thuộc vào DBMS cụ thể và cấu hình. Ví dụ: `SELECT`, `select` và `Select` là như nhau.
- **Nhiều SQL statement phải được phân tách bằng dấu chấm phẩy (`;`)**.
- Khi xử lý SQL statement, **mọi khoảng trắng đều bị bỏ qua**.

SQL statement có thể viết trên một dòng hoặc tách thành nhiều dòng.

```sql
-- SQL statement trên một dòng

UPDATE user SET username='robot', password='robot' WHERE username = 'root';

-- SQL statement trên nhiều dòng
UPDATE user
SET username='robot', password='robot'
WHERE username = 'root';
```

SQL hỗ trợ ba loại comment:

```sql
## Comment 1
-- Comment 2
/* Comment 3 */
```

### Phân loại SQL

#### DDL (Data Definition Language)

Data Definition Language (DDL) là ngôn ngữ trong nhóm ngôn ngữ SQL, chịu trách nhiệm định nghĩa cấu trúc dữ liệu và định nghĩa database object.

Chức năng chính của DDL là **định nghĩa database object**.

Các lệnh cốt lõi của DDL là `CREATE`, `ALTER`, `DROP`.

#### DML (Data Manipulation Language)

Data Manipulation Language (DML) là các câu lệnh dùng để thao tác database, thực hiện việc truy cập object và dữ liệu trong database.

Chức năng chính của DML là **truy cập dữ liệu**, vì vậy cú pháp của nó chủ yếu xoay quanh **đọc và ghi database**.

Các lệnh cốt lõi của DML là `INSERT`, `UPDATE`, `DELETE`, `SELECT`. Bốn lệnh này được gọi chung là CRUD (Create, Read, Update, Delete), tức thêm, xóa, sửa, truy vấn.

#### TCL (Transaction Control Language)

Transaction Control Language (TCL) dùng để **quản lý transaction trong database**. Các lệnh này quản lý những thay đổi do DML statement thực hiện. TCL cũng cho phép nhóm các statement thành transaction logic.

Các lệnh cốt lõi của TCL là `COMMIT`, `ROLLBACK`.

#### DCL (Data Control Language)

Data Control Language (DCL) là nhóm lệnh dùng để kiểm soát quyền truy cập dữ liệu. Nó có thể kiểm soát quyền của user account cụ thể đối với các database object như table, view, stored procedure và user-defined function.

Các lệnh cốt lõi của DCL là `GRANT`, `REVOKE`.

DCL chủ yếu **kiểm soát quyền truy cập của user**, vì vậy cách sử dụng lệnh không phức tạp. Các quyền có thể được kiểm soát bằng DCL gồm: `CONNECT`, `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `EXECUTE`, `USAGE`, `REFERENCES`.

Quyền được hỗ trợ cũng khác nhau tùy DBMS và thực thể bảo mật.

**Trước tiên, hãy tìm hiểu cách sử dụng DML statement. Chức năng chính của DML là đọc và ghi database để thực hiện thêm, xóa, sửa, truy vấn.**

## Thêm, xóa, sửa, truy vấn

Thêm, xóa, sửa, truy vấn, còn gọi là CRUD, là các thao tác cơ bản trong những thao tác cơ bản của database.

### Chèn dữ liệu

Statement `INSERT INTO` dùng để chèn record mới vào table.

**Chèn toàn bộ row**

```sql
# Chèn một row
INSERT INTO user
VALUES (10, 'root', 'root', 'xxxx@163.com');
# Chèn nhiều row
INSERT INTO user
VALUES (10, 'root', 'root', 'xxxx@163.com'), (12, 'user1', 'user1', 'xxxx@163.com'), (18, 'user2', 'user2', 'xxxx@163.com');
```

**Chèn một phần row**

```sql
INSERT INTO user(username, password, email)
VALUES ('admin', 'admin', 'xxxx@163.com');
```

**Chèn dữ liệu được truy vấn**

```sql
INSERT INTO user(username)
SELECT name
FROM account;
```

### Cập nhật dữ liệu

Statement `UPDATE` dùng để cập nhật record trong table.

```sql
UPDATE user
SET username='robot', password='robot'
WHERE username = 'root';
```

### Xóa dữ liệu

- Statement `DELETE` dùng để xóa record trong table.
- `TRUNCATE TABLE` có thể làm trống table, tức xóa mọi row. Lưu ý: statement `TRUNCATE` không thuộc cú pháp DML mà thuộc cú pháp DDL.

**Xóa dữ liệu được chỉ định trong table**

```sql
DELETE FROM user
WHERE username = 'robot';
```

**Xóa toàn bộ dữ liệu trong table**

```sql
TRUNCATE TABLE user;
```

### Truy vấn dữ liệu

Statement `SELECT` dùng để truy vấn dữ liệu từ database.

`DISTINCT` dùng để trả về các value khác nhau duy nhất. Nó áp dụng cho tất cả column, nghĩa là chỉ khi value của mọi column đều giống nhau thì mới được xem là giống nhau.

`LIMIT` giới hạn số row trả về. Có thể có hai tham số: tham số đầu tiên là row bắt đầu, tính từ 0; tham số thứ hai là tổng số row trả về.

- `ASC`: tăng dần (mặc định)
- `DESC`: giảm dần

**Truy vấn một column**

```sql
SELECT prod_name
FROM products;
```

**Truy vấn nhiều column**

```sql
SELECT prod_id, prod_name, prod_price
FROM products;
```

**Truy vấn tất cả column**

```sql
SELECT *
FROM products;
```

**Truy vấn các value khác nhau**

```sql
SELECT DISTINCT
vend_id FROM products;
```

**Giới hạn kết quả truy vấn**

```sql
-- Trả về 5 row đầu tiên
SELECT * FROM mytable LIMIT 5;
SELECT * FROM mytable LIMIT 0, 5;
-- Trả về row thứ 3 đến thứ 5
SELECT * FROM mytable LIMIT 2, 3;
```

## Sắp xếp

`order by` dùng để sắp xếp result set theo một hoặc nhiều column. Mặc định record được sắp xếp tăng dần; nếu cần sắp xếp giảm dần, có thể dùng keyword `desc`.

Khi `order by` sắp xếp nhiều column, column được sắp xếp trước đặt ở trước, column được sắp xếp sau đặt ở sau. Các column khác nhau cũng có thể dùng quy tắc sắp xếp khác nhau.

```sql
SELECT * FROM products
ORDER BY prod_price DESC, prod_name ASC;
```

## Nhóm

**`group by`**:

- Clause `group by` nhóm record thành các summary row.
- `group by` trả về một record cho mỗi group.
- `group by` thường đi kèm các aggregate như `count`, `max`, `sum`, `avg`.
- `group by` có thể nhóm theo một hoặc nhiều column.
- Sau khi `group by` sắp xếp theo column dùng để group, `order by` có thể sắp xếp theo summary column.

**Group**

```sql
SELECT cust_name, COUNT(cust_address) AS addr_num
FROM Customers GROUP BY cust_name;
```

**Sắp xếp sau khi group**

```sql
SELECT cust_name, COUNT(cust_address) AS addr_num
FROM Customers GROUP BY cust_name
ORDER BY cust_name DESC;
```

**`having`**:

- `having` dùng để filter kết quả `group by` đã được tổng hợp.
- `having` thường được dùng cùng `group by`.
- `where` và `having` có thể cùng xuất hiện trong một query.

**Dùng WHERE và HAVING để filter dữ liệu**

```sql
SELECT cust_name, COUNT(*) AS NumberOfOrders
FROM Customers
WHERE cust_email IS NOT NULL
GROUP BY cust_name
HAVING COUNT(*) > 1;
```

**`having` và `where`**:

- `where`: filter các row được chỉ định, phía sau không thể thêm aggregate function (group function). `where` đứng trước `group by`.
- `having`: filter group, thường dùng cùng `group by`, không thể dùng độc lập. `having` đứng sau `group by`.

## Subquery

Subquery là SQL query được lồng trong một query lớn hơn, còn được gọi là inner query hoặc inner select. Statement chứa subquery cũng được gọi là outer query hoặc outer select. Nói đơn giản, subquery là việc dùng kết quả của một `select` query (subquery) làm nguồn dữ liệu hoặc điều kiện xét đoán cho một SQL statement khác (main query).

Subquery có thể được nhúng trong statement `SELECT`, `INSERT`, `UPDATE` và `DELETE`, đồng thời có thể dùng cùng các operator `=`, `<`, `>`, `IN`, `BETWEEN`, `EXISTS`.

Subquery thường được dùng sau clause `WHERE` và clause `FROM`:

- Khi dùng cho clause `WHERE`, tùy operator khác nhau, subquery có thể trả về dữ liệu một row một column, nhiều row một column hoặc một row nhiều column. Subquery cần trả về value có thể làm điều kiện query của clause `WHERE`.
- Khi dùng cho clause `FROM`, subquery thường trả về dữ liệu nhiều row nhiều column, tương đương trả về một temporary table, phù hợp với quy tắc phía sau `FROM` phải là table. Cách này có thể thực hiện query kết hợp nhiều table.

> Lưu ý: Database MYSQL chỉ bắt đầu hỗ trợ subquery từ version 4.1, các version cũ không hỗ trợ.

Cú pháp cơ bản của subquery dùng cho clause `WHERE` như sau:

```sql
select column_name [, column_name ]
from   table1 [, table2 ]
where  column_name operator
    (select column_name [, column_name ]
    from table1 [, table2 ]
    [where])
```

- Subquery cần được đặt trong dấu ngoặc `()`.
- `operator` biểu thị operator dùng cho clause where.

Cú pháp cơ bản của subquery dùng cho clause `FROM` như sau:

```sql
select column_name [, column_name ]
from (select column_name [, column_name ]
      from table1 [, table2 ]
      [where]) as temp_table_name
where  condition
```

Kết quả của subquery dùng cho `FROM` tương đương một temporary table, vì vậy cần dùng keyword AS để đặt tên cho temporary table này.

**Subquery lồng trong subquery**

```sql
SELECT cust_name, cust_contact
FROM customers
WHERE cust_id IN (SELECT cust_id
                  FROM orders
                  WHERE order_num IN (SELECT order_num
                                      FROM orderitems
                                      WHERE prod_id = 'RGAN01'));
```

Inner query được thực thi trước parent query để kết quả của inner query có thể được truyền cho outer query. Quy trình thực thi có thể tham khảo hình dưới:

![](https://oss.javaguide.cn/p3-juejin/c439da1f5d4e4b00bdfa4316b933d764~tplv-k3u1fbpfcp-zoom-1.png)

### WHERE

- Clause `WHERE` dùng để filter record, tức thu hẹp phạm vi dữ liệu truy cập.
- Sau `WHERE` là một điều kiện trả về `true` hoặc `false`.
- `WHERE` có thể dùng cùng `SELECT`, `UPDATE` và `DELETE`.
- Các operator có thể dùng trong clause `WHERE`.

| Operator | Mô tả                                                                     |
| -------- | ------------------------------------------------------------------------- |
| =        | Bằng                                                                      |
| <>       | Khác. Lưu ý: trong một số version của SQL, operator này có thể viết là != |
| >        | Lớn hơn                                                                   |
| <        | Nhỏ hơn                                                                   |
| >=       | Lớn hơn hoặc bằng                                                         |
| <=       | Nhỏ hơn hoặc bằng                                                         |
| BETWEEN  | Trong một phạm vi                                                         |
| LIKE     | Tìm kiếm một pattern                                                      |
| IN       | Chỉ định nhiều value có thể có của một column                             |

**Clause `WHERE` trong statement `SELECT`**

```ini
SELECT * FROM Customers
WHERE cust_name = 'Kids Place';
```

**Clause `WHERE` trong statement `UPDATE`**

```ini
UPDATE Customers
SET cust_name = 'Jack Jones'
WHERE cust_name = 'Kids Place';
```

**Clause `WHERE` trong statement `DELETE`**

```ini
DELETE FROM Customers
WHERE cust_name = 'Kids Place';
```

### IN và BETWEEN

- Operator `IN` dùng trong clause `WHERE`, có tác dụng chọn một value bất kỳ trong một số value cụ thể được chỉ định.
- Operator `BETWEEN` dùng trong clause `WHERE`, có tác dụng chọn value nằm trong một phạm vi.

**Ví dụ IN**

```sql
SELECT *
FROM products
WHERE vend_id IN ('DLL01', 'BRS01');
```

**Ví dụ BETWEEN**

```sql
SELECT *
FROM products
WHERE prod_price BETWEEN 3 AND 5;
```

### AND, OR, NOT

- `AND`, `OR`, `NOT` là các lệnh dùng để xử lý logic cho điều kiện filter.
- Độ ưu tiên của `AND` cao hơn `OR`; để làm rõ thứ tự xử lý, có thể dùng `()`.
- Operator `AND` biểu thị cả điều kiện bên trái và bên phải đều phải thỏa mãn.
- Operator `OR` biểu thị chỉ cần một trong hai điều kiện bên trái và bên phải thỏa mãn.
- Operator `NOT` dùng để phủ định một điều kiện.

**Ví dụ AND**

```sql
SELECT prod_id, prod_name, prod_price
FROM products
WHERE vend_id = 'DLL01' AND prod_price <= 4;
```

**Ví dụ OR**

```ini
SELECT prod_id, prod_name, prod_price
FROM products
WHERE vend_id = 'DLL01' OR vend_id = 'BRS01';
```

**Ví dụ NOT**

```sql
SELECT *
FROM products
WHERE prod_price NOT BETWEEN 3 AND 5;
```

### LIKE

- Operator `LIKE` dùng trong clause `WHERE`, có tác dụng xác định string có khớp pattern hay không.
- Chỉ dùng `LIKE` khi field là text value.
- `LIKE` hỗ trợ hai wildcard: `%` và `_`.
- Không nên lạm dụng wildcard; wildcard ở đầu pattern sẽ khiến việc match rất chậm.
- `%` biểu thị một character bất kỳ xuất hiện số lần bất kỳ.
- `_` biểu thị một character bất kỳ xuất hiện một lần.

**Ví dụ %**

```sql
SELECT prod_id, prod_name, prod_price
FROM products
WHERE prod_name LIKE '%bean bag%';
```

**Ví dụ \_**

```sql
SELECT prod_id, prod_name, prod_price
FROM products
WHERE prod_name LIKE '__ inch teddy bear';
```

## JOIN

JOIN có nghĩa là “kết nối”. Đúng như tên gọi, SQL JOIN clause dùng để kết hợp hai hoặc nhiều table để query.

Khi join table, cần chọn một field trong mỗi table và so sánh value của các field này; hai record có value giống nhau sẽ được gộp thành một. **Bản chất của việc join table là gộp record của các table khác nhau lại để tạo thành một table mới. Tất nhiên, table mới này chỉ là temporary table, chỉ tồn tại trong thời gian query hiện tại.**

Cú pháp cơ bản để dùng `JOIN` kết nối hai table như sau:

```sql
select table1.column1, table2.column2...
from table1
join table2
on table1.common_column1 = table2.common_column2;
```

`table1.common_column1 = table2.common_column2` là join condition; chỉ những record thỏa mãn điều kiện này mới được gộp thành một row. Có thể dùng nhiều operator để join table, chẳng hạn =, >, <, <>, <=, >=, !=, `between`, `like` hoặc `not`, nhưng phổ biến nhất là dùng =.

Khi hai table có field trùng tên, để database engine phân biệt field thuộc table nào, khi viết tên field trùng nhau cần thêm tên table. Tất nhiên, nếu tên field là duy nhất trong hai table thì có thể không dùng định dạng trên mà chỉ viết tên field.

Ngoài ra, nếu tên field liên kết của hai table giống nhau, cũng có thể dùng clause `USING` thay cho `ON`, ví dụ:

```sql
# join....on
select c.cust_name, o.order_num
from Customers c
inner join Orders o
on c.cust_id = o.cust_id
order by c.cust_name;

# Nếu tên field liên kết của hai table giống nhau, cũng có thể dùng clause USING: join....using()
select c.cust_name, o.order_num
from Customers c
inner join Orders o
using(cust_id)
order by c.cust_name;
```

**Khác biệt giữa `ON` và `WHERE`**:

- Khi join table, SQL tạo một temporary table mới dựa trên join condition. `ON` chính là join condition, quyết định việc tạo temporary table.
- `WHERE` filter dữ liệu trong temporary table sau khi temporary table được tạo để tạo ra result set cuối cùng; lúc này không còn JOIN-ON.

Tóm lại: **SQL trước tiên tạo một temporary table dựa trên ON, sau đó filter temporary table dựa trên WHERE**.

SQL cho phép thêm một số keyword bổ nghĩa ở bên trái `JOIN` để tạo ra các loại join khác nhau như bảng dưới đây:

| Loại JOIN                                        | Mô tả                                                                                                                                  |
| ------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| INNER JOIN (inner join)                          | (Cách join mặc định) Chỉ trả về row khi cả hai table đều có record thỏa mãn điều kiện.                                                 |
| LEFT JOIN / LEFT OUTER JOIN (left outer join)    | Trả về mọi row trong table bên trái, kể cả khi table bên phải không có row thỏa mãn điều kiện.                                         |
| RIGHT JOIN / RIGHT OUTER JOIN (right outer join) | Trả về mọi row trong table bên phải, kể cả khi table bên trái không có row thỏa mãn điều kiện.                                         |
| FULL JOIN / FULL OUTER JOIN (full outer join)    | Chỉ cần một trong hai table có record thỏa mãn điều kiện thì sẽ trả về row.                                                            |
| SELF JOIN                                        | Join một table với chính nó, như thể table đó là hai table. Để phân biệt hai table, cần đổi tên ít nhất một table trong SQL statement. |
| CROSS JOIN                                       | Cross join, trả về Cartesian product của result set từ hai hoặc nhiều table được join.                                                 |

Hình dưới đây minh họa 7 cách sử dụng liên quan đến LEFT JOIN, RIGHT JOIN, INNER JOIN và OUTER JOIN.

![](https://oss.javaguide.cn/p3-juejin/701670942f0f45d3a3a2187cd04a12ad~tplv-k3u1fbpfcp-zoom-1.png)

Nếu không thêm keyword bổ nghĩa nào, chỉ viết `JOIN`, thì mặc định là `INNER JOIN`.

Với `INNER JOIN`, còn có một cách viết ẩn gọi là “**inner join ẩn**”, tức không dùng keyword `INNER JOIN` mà dùng statement `WHERE` để thực hiện chức năng inner join.

```sql
# Inner join ẩn
select c.cust_name, o.order_num
from Customers c, Orders o
where c.cust_id = o.cust_id
order by c.cust_name;

# Inner join tường minh
select c.cust_name, o.order_num
from Customers c inner join Orders o
using(cust_id)
order by c.cust_name;
```

## UNION

Operator `UNION` kết hợp kết quả của hai hoặc nhiều query và tạo ra một result set chứa các row được truy xuất từ những query tham gia `UNION`.

Quy tắc cơ bản của `UNION`:

- Số lượng column và thứ tự column của tất cả query phải giống nhau.
- Data type của column thuộc table được dùng trong mỗi query phải giống nhau hoặc tương thích.
- Thông thường tên column trả về được lấy từ query đầu tiên.

Mặc định, operator `UNION` chọn các value khác nhau. Nếu cho phép value trùng lặp, hãy dùng `UNION ALL`.

```sql
SELECT column_name(s) FROM table1
UNION ALL
SELECT column_name(s) FROM table2;
```

Tên column trong result set của `UNION` luôn bằng tên column trong statement `SELECT` đầu tiên của `UNION`.

`JOIN` và `UNION`:

- Các column của table được join trong `JOIN` có thể khác nhau, nhưng trong `UNION`, số lượng column và thứ tự column của tất cả query phải giống nhau.
- `UNION` đặt các row sau query cạnh nhau (theo chiều dọc), còn `JOIN` đặt các column sau query cạnh nhau (theo chiều ngang), tức tạo thành Cartesian product.

## Function

Function của các database khác nhau thường không giống nhau, vì vậy không có tính portable. Phần này chủ yếu lấy function của MySQL làm ví dụ.

### Xử lý text

| Function             | Mô tả                                   |
| -------------------- | --------------------------------------- |
| `LEFT()`, `RIGHT()`  | Character ở bên trái hoặc bên phải      |
| `LOWER()`, `UPPER()` | Chuyển thành chữ thường hoặc chữ hoa    |
| `LTRIM()`, `RTRIM()` | Xóa khoảng trắng bên trái hoặc bên phải |
| `LENGTH()`           | Độ dài, tính theo byte                  |
| `SOUNDEX()`          | Chuyển thành giá trị ngữ âm             |

Trong đó, **`SOUNDEX()`** có thể chuyển một string thành pattern alphanumeric mô tả cách phát âm của string đó.

```sql
SELECT *
FROM mytable
WHERE SOUNDEX(col1) = SOUNDEX('apple')
```

### Xử lý ngày và thời gian

- Định dạng ngày: `YYYY-MM-DD`
- Định dạng thời gian: `HH:MM:SS`

| Function        | Mô tả                                         |
| --------------- | --------------------------------------------- |
| `AddDate()`     | Thêm một ngày (ngày, tuần, v.v.)              |
| `AddTime()`     | Thêm một khoảng thời gian (giờ, phút, v.v.)   |
| `CurDate()`     | Trả về ngày hiện tại                          |
| `CurTime()`     | Trả về thời gian hiện tại                     |
| `Date()`        | Trả về phần ngày của datetime                 |
| `DateDiff()`    | Tính chênh lệch giữa hai ngày                 |
| `Date_Add()`    | Function tính toán ngày có tính linh hoạt cao |
| `Date_Format()` | Trả về string ngày hoặc thời gian đã format   |
| `Day()`         | Trả về phần ngày trong một date               |
| `DayOfWeek()`   | Trả về thứ tương ứng với một date             |
| `Hour()`        | Trả về phần giờ trong một time                |
| `Minute()`      | Trả về phần phút trong một time               |
| `Month()`       | Trả về phần tháng trong một date              |
| `Now()`         | Trả về ngày và thời gian hiện tại             |
| `Second()`      | Trả về phần giây trong một time               |
| `Time()`        | Trả về phần time của một datetime             |
| `Year()`        | Trả về phần năm trong một date                |

### Xử lý số

| Function | Mô tả             |
| -------- | ----------------- |
| SIN()    | Sin               |
| COS()    | Cos               |
| TAN()    | Tan               |
| ABS()    | Giá trị tuyệt đối |
| SQRT()   | Căn bậc hai       |
| MOD()    | Phần dư           |
| EXP()    | Hàm mũ            |
| PI()     | Số pi             |
| RAND()   | Số ngẫu nhiên     |

### Tổng hợp

| Function  | Mô tả                                    |
| --------- | ---------------------------------------- |
| `AVG()`   | Trả về giá trị trung bình của một column |
| `COUNT()` | Trả về số row của một column             |
| `MAX()`   | Trả về giá trị lớn nhất của một column   |
| `MIN()`   | Trả về giá trị nhỏ nhất của một column   |
| `SUM()`   | Trả về tổng value của một column         |

`AVG()` sẽ bỏ qua row NULL.

Dùng `DISTINCT` có thể khiến aggregate function chỉ tổng hợp các value khác nhau.

```sql
SELECT AVG(DISTINCT col1) AS avg_col
FROM mytable
```

**Tiếp theo, hãy tìm hiểu cách sử dụng DDL statement. Chức năng chính của DDL là định nghĩa database object (chẳng hạn database, table, view, index).**

## Định nghĩa dữ liệu

### Database (DATABASE)

#### Tạo database

```sql
CREATE DATABASE test;
```

#### Xóa database

```sql
DROP DATABASE test;
```

#### Chọn database

```sql
USE test;
```

### Table (TABLE)

#### Tạo table

**Tạo thông thường**

```sql
CREATE TABLE user (
  id int(10) unsigned NOT NULL COMMENT 'Id',
  username varchar(64) NOT NULL DEFAULT 'default' COMMENT 'Tên người dùng',
  password varchar(64) NOT NULL DEFAULT 'default' COMMENT 'Mật khẩu',
  email varchar(64) NOT NULL DEFAULT 'default' COMMENT 'Email'
) COMMENT='Table người dùng';
```

**Tạo table mới dựa trên table có sẵn**

```sql
CREATE TABLE vip_user AS
SELECT * FROM user;
```

#### Xóa table

```sql
DROP TABLE user;
```

#### Sửa table

**Thêm column**

```sql
ALTER TABLE user
ADD age int(3);
```

**Xóa column**

```sql
ALTER TABLE user
DROP COLUMN age;
```

**Sửa column**

```sql
ALTER TABLE `user`
MODIFY COLUMN age tinyint;
```

**Thêm primary key**

```sql
ALTER TABLE user
ADD PRIMARY KEY (id);
```

**Xóa primary key**

```sql
ALTER TABLE user
DROP PRIMARY KEY;
```

### View (VIEW)

Định nghĩa:

- View là table trực quan dựa trên result set của SQL statement.
- View là virtual table, bản thân nó không chứa dữ liệu nên cũng không thể thực hiện thao tác index trên đó. Thao tác trên view giống thao tác trên table thông thường.

Tác dụng:

- Đơn giản hóa các thao tác SQL phức tạp, chẳng hạn join phức tạp;
- Chỉ sử dụng một phần dữ liệu của table thực;
- Đảm bảo an toàn dữ liệu bằng cách chỉ cấp cho user quyền truy cập view;
- Thay đổi format và cách biểu diễn dữ liệu.

![View MySQL](https://oss.javaguide.cn/p3-juejin/ec4c975296ea4a7097879dac7c353878~tplv-k3u1fbpfcp-zoom-1.jpeg)

#### Tạo view

```sql
CREATE VIEW top_10_user_view AS
SELECT id, username
FROM user
WHERE id < 10;
```

#### Xóa view

```sql
DROP VIEW top_10_user_view;
```

### Index (INDEX)

**Index là một data structure dùng để query và tìm kiếm dữ liệu nhanh, bản chất có thể xem như một data structure đã được sắp xếp.**

Tác dụng của index tương tự mục lục của sách. Ví dụ, khi tra từ điển, nếu không có mục lục thì chỉ có thể tìm từng trang một, tốc độ rất chậm. Nếu có mục lục, chỉ cần tìm vị trí của từ trong mục lục rồi lật trực tiếp đến trang đó.

**Ưu điểm**:

- Dùng index có thể tăng đáng kể tốc độ truy xuất dữ liệu (giảm đáng kể lượng dữ liệu cần truy xuất), đây cũng là lý do quan trọng nhất để tạo index.
- Tạo unique index có thể đảm bảo tính duy nhất của mỗi row dữ liệu trong database table.

**Nhược điểm**:

- Tạo và duy trì index tốn nhiều thời gian. Khi thêm, xóa hoặc sửa dữ liệu trong table, nếu dữ liệu có index thì index cũng cần được cập nhật động, làm giảm hiệu suất thực thi SQL.
- Index cần dùng file vật lý để lưu trữ, đồng thời cũng chiếm một phần dung lượng.

Tuy nhiên, **dùng index nhất định sẽ cải thiện query performance sao?**

Trong phần lớn trường hợp, index query nhanh hơn full table scan. Nhưng nếu lượng dữ liệu trong database không lớn thì dùng index chưa chắc mang lại cải thiện đáng kể.

Để xem phần giải thích chi tiết về index, hãy đọc bài [Giải thích chi tiết về MySQL Index](https://javaguide.cn/database/mysql/mysql-index.html) do tôi viết.

#### Tạo index

```sql
CREATE INDEX user_index
ON user (id);
```

#### Thêm index

```sql
ALTER table user ADD INDEX user_index(id)
```

#### Tạo unique index

```sql
CREATE UNIQUE INDEX user_index
ON user (id);
```

#### Xóa index

```sql
ALTER TABLE user
DROP INDEX user_index;
```

### Constraint

SQL constraint dùng để quy định quy tắc dữ liệu trong table.

Nếu có thao tác dữ liệu vi phạm constraint, thao tác đó sẽ bị constraint ngăn lại.

Constraint có thể được quy định khi tạo table (thông qua statement CREATE TABLE), hoặc sau khi table được tạo (thông qua statement ALTER TABLE).

Các loại constraint:

- `NOT NULL` - Chỉ ra một column không thể lưu value NULL.
- `UNIQUE` - Đảm bảo mỗi row của một column phải có value duy nhất.
- `PRIMARY KEY` - Kết hợp của NOT NULL và UNIQUE. Đảm bảo một column (hoặc kết hợp của hai hay nhiều column) có định danh duy nhất, giúp tìm một record cụ thể trong table dễ dàng và nhanh hơn.
- `FOREIGN KEY` - Đảm bảo referential integrity, tức dữ liệu trong một table khớp với value trong table khác.
- `CHECK` - Đảm bảo value trong column phù hợp với điều kiện được chỉ định.
- `DEFAULT` - Quy định default value khi không gán value cho column.

Dùng constraint khi tạo table:

```sql
CREATE TABLE Users (
  Id INT(10) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT 'Id tự tăng',
  Username VARCHAR(64) NOT NULL UNIQUE DEFAULT 'default' COMMENT 'Tên người dùng',
  Password VARCHAR(64) NOT NULL DEFAULT 'default' COMMENT 'Mật khẩu',
  Email VARCHAR(64) NOT NULL DEFAULT 'default' COMMENT 'Địa chỉ email',
  Enabled TINYINT(4) DEFAULT NULL COMMENT 'Có hiệu lực hay không',
  PRIMARY KEY (Id)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COMMENT='Table người dùng';
```

**Tiếp theo, hãy tìm hiểu cách sử dụng TCL statement. Chức năng chính của TCL là quản lý transaction trong database.**

## Xử lý transaction

Không thể rollback statement `SELECT`; rollback statement `SELECT` cũng không có ý nghĩa. Cũng không thể rollback statement `CREATE` và `DROP`.

**MySQL mặc định là implicit commit**: sau mỗi statement được thực thi, statement đó được xem là một transaction rồi commit. Khi xuất hiện statement `START TRANSACTION`, implicit commit sẽ bị tắt; sau khi statement `COMMIT` hoặc `ROLLBACK` được thực thi, transaction sẽ tự động kết thúc và implicit commit được khôi phục.

Có thể tắt auto commit bằng `set autocommit=0`, transaction chỉ được commit khi `set autocommit=1`; flag `autocommit` áp dụng cho từng connection chứ không áp dụng cho server.

Các lệnh:

- `START TRANSACTION` - Lệnh đánh dấu điểm bắt đầu của transaction.
- `SAVEPOINT` - Lệnh tạo savepoint.
- `ROLLBACK TO` - Lệnh rollback về savepoint được chỉ định; nếu chưa đặt savepoint thì rollback về statement `START TRANSACTION`.
- `COMMIT` - Commit transaction.

```sql
-- Bắt đầu transaction
START TRANSACTION;

-- Thao tác insert A
INSERT INTO `user`
VALUES (1, 'root1', 'root1', 'xxxx@163.com');

-- Tạo savepoint updateA
SAVEPOINT updateA;

-- Thao tác insert B
INSERT INTO `user`
VALUES (2, 'root2', 'root2', 'xxxx@163.com');

-- Rollback về savepoint updateA
ROLLBACK TO updateA;

-- Commit transaction, chỉ thao tác A có hiệu lực
COMMIT;
```

**Tiếp theo, hãy tìm hiểu cách sử dụng DCL statement. Chức năng chính của DCL là kiểm soát quyền truy cập của user.**

## Kiểm soát quyền

Để cấp quyền cho user account, có thể dùng command `GRANT`. Để thu hồi quyền của user, có thể dùng command `REVOKE`. Phần này lấy MySQL làm ví dụ để giới thiệu ứng dụng thực tế của việc kiểm soát quyền.

Cú pháp cấp quyền bằng `GRANT`:

```sql
GRANT privilege,[privilege],.. ON privilege_level
TO user [IDENTIFIED BY password]
[REQUIRE tsl_option]
[WITH [GRANT_OPTION | resource_option]];
```

Giải thích đơn giản:

1. Chỉ định một hoặc nhiều quyền sau keyword `GRANT`. Nếu cấp nhiều quyền cho user, mỗi quyền được phân tách bằng dấu phẩy.
2. `ON privilege_level` xác định cấp độ áp dụng của quyền. MySQL hỗ trợ cấp global (`*.*`), database (`database.*`), table (`database.table`) và cấp column. Nếu dùng cấp quyền column thì sau mỗi quyền phải chỉ định một danh sách column hoặc danh sách column được phân tách bằng dấu phẩy.
3. `user` là user được cấp quyền. Nếu user đã tồn tại, `GRANT` statement sẽ sửa quyền của user đó; nếu không, `GRANT` statement sẽ tạo user mới. Clause tùy chọn `IDENTIFIED BY` cho phép đặt password mới cho user.
4. `REQUIRE tsl_option` chỉ định user có bắt buộc phải kết nối đến database server thông qua connection an toàn như SSL, X059 hay không.
5. Clause tùy chọn `WITH GRANT OPTION` cho phép cấp cho user khác hoặc thu hồi từ user khác những quyền bạn đang có. Ngoài ra, có thể dùng clause `WITH` để phân bổ tài nguyên của MySQL database server, chẳng hạn đặt số connection hoặc statement tối đa mà user có thể sử dụng mỗi giờ. Điều này rất hữu ích trong môi trường dùng chung như shared hosting MySQL.

Cú pháp thu hồi quyền bằng `REVOKE`:

```sql
REVOKE   privilege_type [(column_list)]
        [, priv_type [(column_list)]]...
ON [object_type] privilege_level
FROM user [, user]...
```

Giải thích đơn giản:

1. Chỉ định danh sách quyền cần thu hồi khỏi user sau keyword `REVOKE`. Các quyền cần được phân tách bằng dấu phẩy.
2. Chỉ định cấp độ quyền cần thu hồi trong clause `ON`.
3. Chỉ định user account cần thu hồi quyền trong clause `FROM`.

`GRANT` và `REVOKE` có thể kiểm soát quyền truy cập ở nhiều cấp:

- Toàn bộ server, dùng `GRANT ALL` và `REVOKE ALL`;
- Toàn bộ database, dùng `ON database.*`;
- Table cụ thể, dùng `ON database.table`;
- Column cụ thể;
- Stored procedure cụ thể.

Account mới tạo không có quyền nào. Account được định nghĩa dưới dạng `username@host`; `username@%` dùng default host name. Thông tin account của MySQL được lưu trong database mysql.

```sql
USE mysql;
SELECT user FROM user;
```

Bảng dưới đây mô tả tất cả quyền được phép dùng trong statement `GRANT` và `REVOKE`:

| **Privilege**           | **Mô tả**                                                                                                                          | **Level** |            |             |           |     |     |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | --------- | ---------- | ----------- | --------- | --- | --- |
| **Global**              | Database                                                                                                                           | **Table** | **Column** | **Routine** | **Proxy** |     |     |
| ALL [PRIVILEGES]        | Cấp mọi quyền ở access level được chỉ định, trừ GRANT OPTION                                                                       |           |            |             |           |     |     |
| ALTER                   | Cho phép user dùng statement ALTER TABLE                                                                                           | X         | X          | X           |           |     |     |
| ALTER ROUTINE           | Cho phép user thay đổi hoặc xóa routine đã lưu                                                                                     | X         | X          |             |           | X   |     |
| CREATE                  | Cho phép user tạo database và table                                                                                                | X         | X          | X           |           |     |     |
| CREATE ROUTINE          | Cho phép user tạo routine đã lưu                                                                                                   | X         | X          |             |           |     |     |
| CREATE TABLESPACE       | Cho phép user tạo, thay đổi hoặc xóa tablespace và log file group                                                                  | X         |            |             |           |     |     |
| CREATE TEMPORARY TABLES | Cho phép user dùng CREATE TEMPORARY TABLE để tạo temporary table                                                                   | X         | X          |             |           |     |     |
| CREATE USER             | Cho phép user dùng các statement CREATE USER, DROP USER, RENAME USER và REVOKE ALL PRIVILEGES.                                     | X         |            |             |           |     |     |
| CREATE VIEW             | Cho phép user tạo hoặc sửa view.                                                                                                   | X         | X          | X           |           |     |     |
| DELETE                  | Cho phép user dùng DELETE                                                                                                          | X         | X          | X           |           |     |     |
| DROP                    | Cho phép user xóa database, table và view                                                                                          | X         | X          | X           |           |     |     |
| EVENT                   | Cho phép event sử dụng event scheduler.                                                                                            | X         | X          |             |           |     |     |
| EXECUTE                 | Cho phép user thực thi routine đã lưu                                                                                              | X         | X          | X           |           |     |     |
| FILE                    | Cho phép user đọc mọi file trong database directory.                                                                               | X         |            |             |           |     |     |
| GRANT OPTION            | Cho phép user có quyền cấp hoặc thu hồi quyền của account khác.                                                                    | X         | X          | X           |           | X   | X   |
| INDEX                   | Cho phép user tạo hoặc xóa index.                                                                                                  | X         | X          | X           |           |     |     |
| INSERT                  | Cho phép user dùng statement INSERT                                                                                                | X         | X          | X           | X         |     |     |
| LOCK TABLES             | Cho phép user dùng LOCK TABLES trên table có quyền SELECT                                                                          | X         | X          |             |           |     |     |
| PROCESS                 | Cho phép user dùng statement SHOW PROCESSLIST để xem mọi process.                                                                  | X         |            |             |           |     |     |
| PROXY                   | Bật user proxy.                                                                                                                    |           |            |             |           |     |     |
| REFERENCES              | Cho phép user tạo foreign key                                                                                                      | X         | X          | X           | X         |     |     |
| RELOAD                  | Cho phép user dùng thao tác FLUSH                                                                                                  | X         |            |             |           |     |     |
| REPLICATION CLIENT      | Cho phép user query để xem vị trí của master server hoặc slave server                                                              | X         |            |             |           |     |     |
| REPLICATION SLAVE       | Cho phép user dùng replication slave để đọc binary log event từ master server.                                                     | X         |            |             |           |     |     |
| SELECT                  | Cho phép user dùng statement SELECT                                                                                                | X         | X          | X           | X         |     |     |
| SHOW DATABASES          | Cho phép user hiển thị mọi database                                                                                                | X         |            |             |           |     |     |
| SHOW VIEW               | Cho phép user dùng statement SHOW CREATE VIEW                                                                                      | X         | X          | X           |           |     |     |
| SHUTDOWN                | Cho phép user dùng command mysqladmin shutdown                                                                                     | X         |            |             |           |     |     |
| SUPER                   | Cho phép user thực hiện các thao tác quản trị khác như CHANGE MASTER TO, KILL, PURGE BINARY LOGS, SET GLOBAL và command mysqladmin | X         |            |             |           |     |     |
| TRIGGER                 | Cho phép user dùng thao tác TRIGGER.                                                                                               | X         | X          | X           |           |     |     |
| UPDATE                  | Cho phép user dùng statement UPDATE                                                                                                | X         | X          | X           | X         |     |     |
| USAGE                   | Tương đương “không có quyền”                                                                                                       |           |            |             |           |     |     |

### Tạo account

```sql
CREATE USER myuser IDENTIFIED BY 'mypassword';
```

### Đổi tên account

```sql
UPDATE user SET user='newuser' WHERE user='myuser';
FLUSH PRIVILEGES;
```

### Xóa account

```sql
DROP USER myuser;
```

### Xem quyền

```sql
SHOW GRANTS FOR myuser;
```

### Cấp quyền

```sql
GRANT SELECT, INSERT ON *.* TO myuser;
```

### Xóa quyền

```sql
REVOKE SELECT, INSERT ON *.* FROM myuser;
```

### Đổi password

```sql
SET PASSWORD FOR myuser = 'mypass';
```

## Stored procedure

Stored procedure có thể được xem như một batch xử lý một chuỗi SQL operation. Stored procedure có thể được gọi bởi trigger, stored procedure khác và các application như Java, Python, PHP.

![Stored procedure MySQL](https://oss.javaguide.cn/p3-juejin/60afdc9c9a594f079727ec64a2e698a3~tplv-k3u1fbpfcp-zoom-1.jpeg)

Lợi ích của việc dùng stored procedure:

- Đóng gói code, đảm bảo một mức độ an toàn nhất định;
- Tái sử dụng code;
- Do được compile trước nên có performance cao.

Tạo stored procedure:

- Khi tạo stored procedure trong command line, cần tự định nghĩa delimiter vì command line dùng `;` làm ký hiệu kết thúc, trong khi stored procedure cũng chứa dấu chấm phẩy nên sẽ nhận nhầm các dấu chấm phẩy này là ký hiệu kết thúc, gây lỗi cú pháp.
- Bao gồm ba loại parameter `in`, `out` và `inout`.
- Mọi thao tác gán value cho variable đều cần dùng statement `select into`.
- Mỗi lần chỉ có thể gán value cho một variable, không hỗ trợ thao tác trên collection.

Cần lưu ý: **Alibaba 《Sổ tay phát triển Java》bắt buộc không được dùng stored procedure, vì stored procedure khó debug và mở rộng, đồng thời không có tính portable.**

![](https://oss.javaguide.cn/p3-juejin/93a5e011ade4450ebfa5d82057532a49~tplv-k3u1fbpfcp-zoom-1.png)

Việc có nên dùng trong project hay không còn tùy vào nhu cầu thực tế của project; chỉ cần cân nhắc kỹ ưu nhược điểm.

### Tạo stored procedure

```sql
DROP PROCEDURE IF EXISTS `proc_adder`;
DELIMITER ;;
CREATE DEFINER=`root`@`localhost` PROCEDURE `proc_adder`(IN a int, IN b int, OUT sum int)
BEGIN
    DECLARE c int;
    if a is null then set a = 0;
    end if;

    if b is null then set b = 0;
    end if;

    set sum  = a + b;
END
;;
DELIMITER ;
```

### Sử dụng stored procedure

```less
set @b=5;
call proc_adder(2,@b,@s);
select @s as sum;
```

## Cursor

Cursor là một database query được lưu trên DBMS server. Nó không phải một statement `SELECT`, mà là result set được statement đó truy xuất.

Dùng cursor trong stored procedure có thể duyệt qua result set theo từng bước.

Cursor chủ yếu được dùng trong interactive application, nơi user cần cuộn dữ liệu trên màn hình và duyệt hoặc thay đổi dữ liệu.

Các bước cụ thể để dùng cursor:

- Trước khi dùng cursor, bắt buộc phải declare (định nghĩa) cursor. Quá trình này thực tế chưa truy xuất dữ liệu, mà chỉ định nghĩa statement `SELECT` và các tùy chọn cursor cần dùng.

- Sau khi declare, bắt buộc phải mở cursor để sử dụng. Quá trình này dùng statement SELECT đã định nghĩa ở trên để thực sự truy xuất dữ liệu.

- Với cursor đã được điền dữ liệu, lấy (truy xuất) từng row theo nhu cầu.

- Khi kết thúc việc dùng cursor, bắt buộc phải đóng cursor và nếu có thể thì giải phóng cursor (phụ thuộc vào DBMS cụ thể).

```sql
DELIMITER $
CREATE  PROCEDURE getTotal()
BEGIN
    DECLARE total INT;
    -- Tạo variable nhận dữ liệu cursor
    DECLARE sid INT;
    DECLARE sname VARCHAR(10);
    -- Tạo variable tổng số
    DECLARE sage INT;
    -- Tạo variable đánh dấu kết thúc
    DECLARE done INT DEFAULT false;
    -- Tạo cursor
    DECLARE cur CURSOR FOR SELECT id,name,age from cursor_table where age>30;
    -- Chỉ định giá trị trả về khi vòng lặp cursor kết thúc
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = true;
    SET total = 0;
    OPEN cur;
    FETCH cur INTO sid, sname, sage;
    WHILE(NOT done)
    DO
        SET total = total + 1;
        FETCH cur INTO sid, sname, sage;
    END WHILE;

    CLOSE cur;
    SELECT total;
END $
DELIMITER ;

-- Gọi stored procedure
call getTotal();
```

## Trigger

Trigger là một database object liên quan đến thao tác trên table. Khi event được chỉ định xuất hiện trên table chứa trigger, object đó sẽ được gọi, tức event thao tác trên table sẽ trigger việc thực thi trigger trên table.

Có thể dùng trigger để audit trail, ghi record thay đổi vào một table khác.

Ưu điểm của trigger:

- SQL trigger cung cấp một phương pháp khác để kiểm tra data integrity.
- SQL trigger có thể bắt các lỗi trong business logic ở database layer.
- SQL trigger cung cấp một phương pháp khác để chạy scheduled task. Khi dùng SQL trigger, không cần chờ scheduled task chạy, vì trigger sẽ tự động được gọi trước hoặc sau khi dữ liệu trong table thay đổi.
- SQL trigger hữu ích cho việc audit các thay đổi dữ liệu trong table.

Nhược điểm của trigger:

- SQL trigger chỉ có thể cung cấp validation mở rộng, không thể thay thế mọi validation. Một số validation đơn giản phải được thực hiện ở application layer. Ví dụ, có thể dùng JavaScript để validate input của user ở client, hoặc dùng server-side scripting language như JSP, PHP, ASP.NET, Perl để validate input của user ở server.
- Việc gọi và thực thi SQL trigger từ client application là không visible, vì vậy rất khó tìm hiểu chuyện gì xảy ra ở database layer.
- SQL trigger có thể làm tăng overhead của database server.

MySQL không cho phép dùng statement CALL trong trigger, tức không thể gọi stored procedure.

> Lưu ý: Trong MySQL, dấu chấm phẩy `;` là identifier kết thúc statement. Khi gặp dấu chấm phẩy, MySQL xem đoạn statement đó đã kết thúc và có thể bắt đầu thực thi. Vì vậy, khi interpreter gặp dấu chấm phẩy trong action thực thi trigger, nó sẽ bắt đầu thực thi rồi báo lỗi vì không tìm thấy END tương ứng với BEGIN.
>
> Lúc này sẽ cần command `DELIMITER` (`DELIMITER` có nghĩa là delimiter, separator). Đây là một command không cần identifier kết thúc statement, cú pháp là: `DELIMITER new_delimiter`. `new_delimiter` có thể là một symbol dài một hoặc nhiều ký tự; mặc định là dấu chấm phẩy `;`, nhưng có thể đổi thành symbol khác như `$` - `DELIMITER $`. Sau đó, các statement kết thúc bằng dấu chấm phẩy sẽ không khiến interpreter phản ứng; chỉ khi gặp `$` thì nó mới xem là statement kết thúc. Lưu ý, sau khi dùng xong cần nhớ đổi lại.

Trước MySQL version 5.7.2, mỗi table được định nghĩa tối đa sáu trigger.

- `BEFORE INSERT` - Activate trước khi insert dữ liệu vào table.
- `AFTER INSERT` - Activate sau khi insert dữ liệu vào table.
- `BEFORE UPDATE` - Activate trước khi update dữ liệu trong table.
- `AFTER UPDATE` - Activate sau khi update dữ liệu trong table.
- `BEFORE DELETE` - Activate trước khi xóa dữ liệu khỏi table.
- `AFTER DELETE` - Activate sau khi xóa dữ liệu khỏi table.

Tuy nhiên, từ MySQL version 5.7.2 trở lên, có thể định nghĩa nhiều trigger cho cùng một trigger event và thời điểm thao tác.

**`NEW` và `OLD`**:

- MySQL định nghĩa keyword `NEW` và `OLD` để biểu thị row dữ liệu đã trigger trigger trong table nơi trigger được định nghĩa.
- Trong trigger kiểu `INSERT`, `NEW` biểu thị dữ liệu mới sắp được (`BEFORE`) hoặc đã được (`AFTER`) insert;
- Trong trigger kiểu `UPDATE`, `OLD` biểu thị dữ liệu cũ sắp hoặc đã bị sửa, `NEW` biểu thị dữ liệu mới sắp hoặc đã được sửa thành;
- Trong trigger kiểu `DELETE`, `OLD` biểu thị dữ liệu cũ sắp hoặc đã bị xóa;
- Cách dùng: `NEW.columnName` (`columnName` là tên một column tương ứng trong table).

### Tạo trigger

> Gợi ý: Để hiểu các điểm chính của trigger, trước tiên cần tìm hiểu command tạo trigger.

Command `CREATE TRIGGER` dùng để tạo trigger.

Cú pháp:

```sql
CREATE TRIGGER trigger_name
trigger_time
trigger_event
ON table_name
FOR EACH ROW
BEGIN
  trigger_statements
END;
```

Giải thích:

- `trigger_name`: tên trigger
- `trigger_time` : thời điểm trigger được activate. Giá trị là `BEFORE` hoặc `AFTER`.
- `trigger_event` : event mà trigger lắng nghe. Giá trị là `INSERT`, `UPDATE` hoặc `DELETE`.
- `table_name` : đối tượng mà trigger lắng nghe. Chỉ định table tạo trigger.
- `FOR EACH ROW`: giám sát cấp row, là cách viết cố định của MySQL; DBMS khác sẽ khác.
- `trigger_statements`: action được trigger thực thi. Là danh sách gồm một hoặc nhiều SQL statement; mỗi statement trong danh sách đều phải kết thúc bằng dấu chấm phẩy `;`.

Khi điều kiện trigger được thỏa mãn, action của trigger nằm giữa `BEGIN` và `END` sẽ được thực thi.

Ví dụ:

```sql
DELIMITER $
CREATE TRIGGER `trigger_insert_user`
AFTER INSERT ON `user`
FOR EACH ROW
BEGIN
    INSERT INTO `user_history`(user_id, operate_type, operate_time)
    VALUES (NEW.id, 'add a user',  now());
END $
DELIMITER ;
```

### Xem trigger

```sql
SHOW TRIGGERS;
```

### Xóa trigger

```sql
DROP TRIGGER IF EXISTS trigger_insert_user;
```

## Bài viết đề xuất

- [Lập trình viên backend nên biết: Hướng dẫn tối ưu hiệu suất cao cho SQL! Hơn 35 đề xuất tối ưu, áp dụng ngay!](https://mp.weixin.qq.com/s/I-ZT3zGTNBZ6egS7T09jyQ)
- [Lập trình viên backend nên biết: 30 đề xuất để viết SQL chất lượng cao](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247486461&idx=1&sn=60a22279196d084cc398936fe3b37772&chksm=cea24436f9d5cd20a4fa0e907590f3e700d7378b3f608d7b33bb52cfb96f503b7ccb65a1deed&token=1987003517&lang=zh_CN#rd)

<!-- @include: @article-footer.snippet.md -->
