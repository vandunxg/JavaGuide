---
title: Tổng hợp câu hỏi phỏng vấn SQL thường gặp (1)
description: Phần đầu của tổng hợp câu hỏi phỏng vấn SQL thường gặp, bao gồm các thao tác truy vấn cơ bản như SELECT truy xuất dữ liệu, WHERE lọc điều kiện, ORDER BY sắp xếp, DISTINCT loại bỏ trùng lặp, LIMIT phân trang và phân tích đề thi Nowcoder.
category: Database
tag:
  - Database Basics
  - SQL
head:
  - - meta
    - name: keywords
      content: câu hỏi phỏng vấn SQL, truy vấn SELECT, điều kiện WHERE, sắp xếp ORDER BY, loại bỏ trùng lặp DISTINCT, phân trang LIMIT, SQL Basics
---

> Nguồn câu hỏi: [Nowcoder - SQL cần biết và thành thạo](https://www.nowcoder.com/exam/oj?page=1&tab=SQL%E7%AF%87&topicId=298)

## Truy xuất dữ liệu

`SELECT` được dùng để truy vấn dữ liệu từ database.

### Truy xuất tất cả ID từ bảng Customers

Có bảng `Customers` như sau:

| cust_id |
| ------- |
| A       |
| B       |
| C       |

Viết câu lệnh SQL để truy xuất tất cả `cust_id` từ bảng `Customers`.

Đáp án:

```sql
SELECT cust_id
FROM Customers
```

### Truy xuất và liệt kê danh sách sản phẩm đã đặt

Bảng `OrderItems` có cột `prod_id` không rỗng, đại diện cho id sản phẩm và chứa tất cả sản phẩm đã đặt (một số sản phẩm được đặt nhiều lần).

| prod_id |
| ------- |
| a1      |
| a2      |
| a3      |
| a4      |
| a5      |
| a6      |
| a7      |

Viết câu lệnh SQL để truy xuất và liệt kê danh sách không trùng lặp của tất cả sản phẩm đã đặt (`prod_id`).

Đáp án:

```sql
SELECT DISTINCT prod_id
FROM OrderItems
```

Điểm cần biết: `DISTINCT` dùng để trả về các giá trị duy nhất trong cột.

### Truy xuất tất cả cột

Hiện có bảng `Customers` (bảng có cột `cust_id` đại diện cho id khách hàng và `cust_name` đại diện cho tên khách hàng).

| cust_id | cust_name |
| ------- | --------- |
| a1      | andy      |
| a2      | ben       |
| a3      | tony      |
| a4      | tom       |
| a5      | an        |
| a6      | lee       |
| a7      | hex       |

Cần viết câu lệnh SQL để truy xuất tất cả cột.

Đáp án:

```sql
SELECT cust_id, cust_name
FROM Customers
```

## Truy xuất dữ liệu có sắp xếp

`ORDER BY` dùng để sắp xếp result set theo một hoặc nhiều cột. Mặc định, các bản ghi được sắp xếp theo thứ tự tăng dần. Nếu cần sắp xếp theo thứ tự giảm dần, có thể dùng keyword `DESC`.

### Truy xuất và sắp xếp tên khách hàng

Có bảng `Customers`, trong đó `cust_id` đại diện cho id khách hàng và `cust_name` đại diện cho tên khách hàng.

| cust_id | cust_name |
| ------- | --------- |
| a1      | andy      |
| a2      | ben       |
| a3      | tony      |
| a4      | tom       |
| a5      | an        |
| a6      | lee       |
| a7      | hex       |

Truy xuất tất cả tên khách hàng (`cust_name`) từ `Customers` và hiển thị kết quả theo thứ tự từ Z đến A.

Đáp án:

```sql
SELECT cust_name
FROM Customers
ORDER BY cust_name DESC
```

### Sắp xếp theo ID khách hàng và ngày

Có bảng `Orders`:

| cust_id | order_num | order_date          |
| ------- | --------- | ------------------- |
| andy    | aaaa      | 2021-01-01 00:00:00 |
| andy    | bbbb      | 2021-01-01 12:00:00 |
| bob     | cccc      | 2021-01-10 12:00:00 |
| dick    | dddd      | 2021-01-11 00:00:00 |

Viết câu lệnh SQL để truy xuất ID khách hàng (`cust_id`) và mã đơn hàng (`order_num`) từ bảng `Orders`, trước tiên sắp xếp kết quả theo ID khách hàng, sau đó sắp xếp giảm dần theo ngày đặt hàng.

Đáp án:

```sql
# Sắp xếp theo tên cột
# Lưu ý: là order_date giảm dần, không phải order_num
SELECT cust_id, order_num
FROM Orders
ORDER BY cust_id,order_date DESC
```

Điểm cần biết: khi `order by` sắp xếp theo nhiều cột, cột được ưu tiên sắp xếp trước đặt ở phía trước. Các cột khác nhau có thể dùng quy tắc sắp xếp khác nhau.

### Sắp xếp theo số lượng và giá

Giả sử có bảng `OrderItems`:

| quantity | item_price |
| -------- | ---------- |
| 1        | 100        |
| 10       | 1003       |
| 2        | 500        |

Viết câu lệnh SQL để hiển thị số lượng (`quantity`) và giá (`item_price`) trong bảng `OrderItems`, đồng thời sắp xếp theo số lượng từ nhiều đến ít và giá từ cao đến thấp.

Đáp án:

```sql
SELECT quantity, item_price
FROM OrderItems
ORDER BY quantity DESC,item_price DESC
```

### Kiểm tra câu lệnh SQL

Có bảng `Vendors`:

| vend_name   |
| ----------- |
| Hai Di Lao  |
| Xiaolongkan |
| Dalongyi    |

Câu lệnh SQL dưới đây có vấn đề không? Hãy sửa để câu lệnh chạy đúng và trả về kết quả theo thứ tự giảm dần của `vend_name`.

```sql
SELECT vend_name,
FROM Vendors
ORDER vend_name DESC
```

Sau khi sửa:

```sql
SELECT vend_name
FROM Vendors
ORDER BY vend_name DESC
```

Điểm cần biết:

- Dấu phẩy dùng để phân cách các cột.
- `ORDER BY` có `BY`, cần viết đầy đủ và đúng vị trí.

## Lọc dữ liệu

`WHERE` có thể lọc dữ liệu được trả về.

Các toán tử sau có thể được dùng trong mệnh đề `WHERE`:

| Toán tử | Mô tả                                                                          |
| :------ | :----------------------------------------------------------------------------- |
| =       | Bằng                                                                           |
| <>      | Khác. **Chú thích:** Trong một số phiên bản SQL, toán tử này có thể viết là != |
| >       | Lớn hơn                                                                        |
| <       | Nhỏ hơn                                                                        |
| >=      | Lớn hơn hoặc bằng                                                              |
| <=      | Nhỏ hơn hoặc bằng                                                              |
| BETWEEN | Trong một phạm vi                                                              |
| LIKE    | Tìm kiếm một pattern                                                           |
| IN      | Chỉ định nhiều giá trị có thể có cho một cột                                   |

### Trả về sản phẩm có giá cố định

Có bảng `Products`:

| prod_id | prod_name      | prod_price |
| ------- | -------------- | ---------- |
| a0018   | sockets        | 9.49       |
| a0019   | iphone13       | 600        |
| b0018   | gucci t-shirts | 1000       |

【Bài toán】Truy xuất ID sản phẩm (`prod_id`) và tên sản phẩm (`prod_name`) từ bảng `Products`, chỉ trả về sản phẩm có giá 9.49 đô la.

Đáp án:

```sql
SELECT prod_id, prod_name
FROM Products
WHERE prod_price = 9.49
```

### Trả về sản phẩm có giá cao hơn

Có bảng `Products`:

| prod_id | prod_name      | prod_price |
| ------- | -------------- | ---------- |
| a0018   | sockets        | 9.49       |
| a0019   | iphone13       | 600        |
| b0019   | gucci t-shirts | 1000       |

【Bài toán】Viết câu lệnh SQL để truy xuất ID sản phẩm (`prod_id`) và tên sản phẩm (`prod_name`) từ bảng `Products`, chỉ trả về sản phẩm có giá từ 9 đô la trở lên.

Đáp án:

```sql
SELECT prod_id, prod_name
FROM Products
WHERE prod_price >= 9
```

### Trả về sản phẩm và sắp xếp theo giá

Có bảng `Products`:

| prod_id | prod_name | prod_price |
| ------- | --------- | ---------- |
| a0011   | egg       | 3          |
| a0019   | sockets   | 4          |
| b0019   | coffee    | 15         |

【Bài toán】Viết câu lệnh SQL để trả về tên (`prod_name`) và giá (`prod_price`) của tất cả sản phẩm trong bảng `Products` có giá từ 3 đến 6 đô la, sau đó sắp xếp kết quả theo giá.

Đáp án:

```sql
SELECT prod_name, prod_price
FROM Products
WHERE prod_price BETWEEN 3 AND 6
ORDER BY prod_price

# Hoặc
SELECT prod_name, prod_price
FROM Products
WHERE prod_price >= 3 AND prod_price <= 6
ORDER BY prod_price
```

### Trả về nhiều sản phẩm hơn

Bảng `OrderItems` có mã đơn hàng `order_num` và số lượng sản phẩm `quantity`.

| order_num | quantity |
| --------- | -------- |
| a1        | 105      |
| a2        | 1100     |
| a2        | 200      |
| a4        | 1121     |
| a5        | 10       |
| a2        | 19       |
| a7        | 5        |

【Bài toán】Truy xuất tất cả mã đơn hàng khác nhau và không trùng lặp (`order_num`) từ bảng `OrderItems`, trong đó mỗi đơn hàng phải bao gồm từ 100 sản phẩm trở lên.

Đáp án:

```sql
SELECT order_num
FROM OrderItems
GROUP BY order_num
HAVING SUM(quantity) >= 100
```

## Lọc dữ liệu nâng cao

Các toán tử `AND` và `OR` dùng để lọc bản ghi dựa trên nhiều hơn một điều kiện và có thể kết hợp với nhau. `AND` yêu cầu cả 2 điều kiện đều đúng, còn `OR` chỉ cần một trong 2 điều kiện đúng.

### Truy xuất tên nhà cung cấp

Bảng `Vendors` có các trường tên nhà cung cấp (`vend_name`), quốc gia của nhà cung cấp (`vend_country`), bang của nhà cung cấp (`vend_state`).

| vend_name | vend_country | vend_state |
| --------- | ------------ | ---------- |
| apple     | USA          | CA         |
| vivo      | CNA          | shenzhen   |
| huawei    | CNA          | xian       |

【Bài toán】Viết câu lệnh SQL để truy xuất tên nhà cung cấp (`vend_name`) từ bảng `Vendors`, chỉ trả về nhà cung cấp ở bang California (cần lọc theo quốc gia [USA] và bang [CA], vì có thể các quốc gia khác cũng có một CA).

Đáp án:

```sql
SELECT vend_name
FROM Vendors
WHERE vend_country = 'USA' AND vend_state = 'CA'
```

### Truy xuất và liệt kê danh sách sản phẩm đã đặt

Bảng `OrderItems` chứa tất cả sản phẩm đã đặt (một số sản phẩm được đặt nhiều lần).

| prod_id | order_num | quantity |
| ------- | --------- | -------- |
| BR01    | a1        | 105      |
| BR02    | a2        | 1100     |
| BR02    | a2        | 200      |
| BR03    | a4        | 1121     |
| BR017   | a5        | 10       |
| BR02    | a2        | 19       |
| BR017   | a7        | 5        |

【Bài toán】Viết câu lệnh SQL để tìm tất cả đơn hàng đã đặt ít nhất 100 sản phẩm `BR01`, `BR02` hoặc `BR03`. Cần trả về mã đơn hàng (`order_num`), ID sản phẩm (`prod_id`) và số lượng (`quantity`) của bảng `OrderItems`, đồng thời lọc theo ID sản phẩm và số lượng.

Đáp án:

```sql
SELECT order_num, prod_id, quantity
FROM OrderItems
WHERE prod_id IN ('BR01', 'BR02', 'BR03') AND quantity >= 100
```

### Trả về tên và giá của tất cả sản phẩm có giá từ 3 đến 6 đô la

Có bảng `Products`:

| prod_id | prod_name | prod_price |
| ------- | --------- | ---------- |
| a0011   | egg       | 3          |
| a0019   | sockets   | 4          |
| b0019   | coffee    | 15         |

【Bài toán】Viết câu lệnh SQL để trả về tên (`prod_name`) và giá (`prod_price`) của tất cả sản phẩm có giá từ 3 đến 6 đô la, sử dụng toán tử AND, sau đó sắp xếp kết quả theo giá tăng dần.

Đáp án:

```sql
SELECT prod_name, prod_price
FROM Products
WHERE prod_price >= 3 and prod_price <= 6
ORDER BY prod_price
```

### Kiểm tra câu lệnh SQL

Bảng nhà cung cấp `Vendors` có các trường tên nhà cung cấp `vend_name`, quốc gia của nhà cung cấp `vend_country`, tỉnh/bang của nhà cung cấp `vend_state`.

| vend_name | vend_country | vend_state |
| --------- | ------------ | ---------- |
| apple     | USA          | CA         |
| vivo      | CNA          | shenzhen   |
| huawei    | CNA          | xian       |

【Bài toán】Sửa SQL dưới đây để trả về kết quả đúng.

```sql
SELECT vend_name
FROM Vendors
ORDER BY vend_name
WHERE vend_country = 'USA' AND vend_state = 'CA';
```

Sau khi sửa:

```sql
SELECT vend_name
FROM Vendors
WHERE vend_country = 'USA' AND vend_state = 'CA'
ORDER BY vend_name
```

Câu lệnh `ORDER BY` phải đặt sau `WHERE`.

## Lọc bằng wildcard

Wildcard trong SQL phải được dùng cùng toán tử `LIKE`.

Trong SQL, có thể sử dụng các wildcard sau:

| Wildcard                         | Mô tả                                                |
| :------------------------------- | :--------------------------------------------------- |
| `%`                              | Đại diện cho 0 hoặc nhiều ký tự                      |
| `_`                              | Chỉ thay thế một ký tự                               |
| `[charlist]`                     | Bất kỳ ký tự đơn nào trong danh sách ký tự           |
| `[^charlist]` hoặc `[!charlist]` | Bất kỳ ký tự đơn nào không nằm trong danh sách ký tự |

### Truy xuất tên và mô tả sản phẩm (1)

Bảng `Products` như sau:

| prod_name | prod_desc      |
| --------- | -------------- |
| a0011     | usb            |
| a0019     | iphone13       |
| b0019     | gucci t-shirts |
| c0019     | gucci toy      |
| d0019     | lego toy       |

【Bài toán】Viết câu lệnh SQL để truy xuất tên sản phẩm (`prod_name`) và mô tả (`prod_desc`) từ bảng `Products`, chỉ trả về tên sản phẩm có chứa từ `toy` trong mô tả.

Đáp án:

```sql
SELECT prod_name, prod_desc
FROM Products
WHERE prod_desc LIKE '%toy%'
```

### Truy xuất tên và mô tả sản phẩm (2)

Bảng `Products` như sau:

| prod_name | prod_desc      |
| --------- | -------------- |
| a0011     | usb            |
| a0019     | iphone13       |
| b0019     | gucci t-shirts |
| c0019     | gucci toy      |
| d0019     | lego toy       |

【Bài toán】Viết câu lệnh SQL để truy xuất tên sản phẩm (`prod_name`) và mô tả (`prod_desc`) từ bảng `Products`, chỉ trả về sản phẩm không xuất hiện từ `toy` trong mô tả, cuối cùng sắp xếp kết quả theo “tên sản phẩm”.

Đáp án:

```sql
SELECT prod_name, prod_desc
FROM Products
WHERE prod_desc NOT LIKE '%toy%'
ORDER BY prod_name
```

### Truy xuất tên và mô tả sản phẩm (3)

Bảng `Products` như sau:

| prod_name | prod_desc        |
| --------- | ---------------- |
| a0011     | usb              |
| a0019     | iphone13         |
| b0019     | gucci t-shirts   |
| c0019     | gucci toy        |
| d0019     | lego carrots toy |

【Bài toán】Viết câu lệnh SQL để truy xuất tên sản phẩm (`prod_name`) và mô tả (`prod_desc`) từ bảng `Products`, chỉ trả về sản phẩm có cả `toy` và `carrots` trong mô tả. Có nhiều cách thực hiện, nhưng với bài này hãy sử dụng `AND` và hai phép so sánh `LIKE`.

Đáp án:

```sql
SELECT prod_name, prod_desc
FROM Products
WHERE prod_desc LIKE '%toy%' AND prod_desc LIKE "%carrots%"
```

### Truy xuất tên và mô tả sản phẩm (4)

Bảng `Products` như sau:

| prod_name | prod_desc        |
| --------- | ---------------- |
| a0011     | usb              |
| a0019     | iphone13         |
| b0019     | gucci t-shirts   |
| c0019     | gucci toy        |
| d0019     | lego toy carrots |

【Bài toán】Viết câu lệnh SQL để truy xuất tên sản phẩm (`prod_name`) và mô tả (`prod_desc`) từ bảng `Products`, chỉ trả về sản phẩm trong đó `toy` và `carrots` xuất hiện đồng thời theo **đúng thứ tự** trong mô tả. Gợi ý: chỉ cần dùng `LIKE` với ba ký hiệu `%`.

Đáp án:

```sql
SELECT prod_name, prod_desc
FROM Products
WHERE prod_desc LIKE '%toy%carrots%'
```

## Tạo calculated field

### Alias

Một cách dùng phổ biến của alias là đổi tên cột trong kết quả truy vấn (để đáp ứng yêu cầu của báo cáo hoặc khách hàng). Có bảng `Vendors` đại diện cho thông tin nhà cung cấp, trong đó `vend_id` là id nhà cung cấp, `vend_name` là tên nhà cung cấp, `vend_address` là địa chỉ nhà cung cấp và `vend_city` là thành phố của nhà cung cấp.

| vend_id | vend_name     | vend_address | vend_city |
| ------- | ------------- | ------------ | --------- |
| a001    | tencent cloud | address1     | shenzhen  |
| a002    | huawei cloud  | address2     | dongguan  |
| a003    | aliyun cloud  | address3     | hangzhou  |
| a003    | netease cloud | address4     | guangzhou |

【Bài toán】Viết câu lệnh SQL để truy xuất `vend_id`, `vend_name`, `vend_address` và `vend_city` từ bảng `Vendors`, đổi tên `vend_name` thành `vname`, `vend_city` thành `vcity`, `vend_address` thành `vaddress`, đồng thời sắp xếp kết quả tăng dần theo tên nhà cung cấp.

Đáp án:

```sql
SELECT vend_id, vend_name AS vname, vend_address AS vaddress, vend_city AS vcity
FROM Vendors
ORDER BY vname
# Có thể bỏ qua as
SELECT vend_id, vend_name vname, vend_address vaddress, vend_city vcity
FROM Vendors
ORDER BY vname
```

### Giảm giá

Cửa hàng mẫu đang có chương trình giảm giá, tất cả sản phẩm đều giảm 10%. Bảng `Products` chứa id sản phẩm `prod_id` và giá sản phẩm `prod_price`.

【Bài toán】Viết câu lệnh SQL để trả về `prod_id`, `prod_price` và `sale_price` từ bảng `Products`. `sale_price` là một calculated field chứa giá khuyến mãi. Gợi ý: có thể nhân với 0.9 để nhận 90% giá gốc (tức giảm 10%).

Đáp án:

```sql
SELECT prod_id, prod_price, prod_price * 0.9 AS sale_price
FROM Products
```

Lưu ý: `sale_price` là tên của kết quả tính toán, không phải tên cột có sẵn.

## Dùng function để xử lý dữ liệu

### Tên đăng nhập của khách hàng

Cửa hàng của chúng ta đã đi vào hoạt động và đang tạo tài khoản khách hàng. Tất cả người dùng đều cần tên đăng nhập, tên đăng nhập mặc định là tổ hợp tên và thành phố nơi người dùng sống.

Cho bảng `Customers` như sau:

| cust_id | cust_name | cust_contact | cust_city |
| ------- | --------- | ------------ | --------- |
| a1      | Andy Li   | Andy Li      | Oak Park  |
| a2      | Ben Liu   | Ben Liu      | Oak Park  |
| a3      | Tony Dai  | Tony Dai     | Oak Park  |
| a4      | Tom Chen  | Tom Chen     | Oak Park  |
| a5      | An Li     | An Li        | Oak Park  |
| a6      | Lee Chen  | Lee Chen     | Oak Park  |
| a7      | Hex Liu   | Hex Liu      | Oak Park  |

【Bài toán】Viết câu lệnh SQL để trả về ID khách hàng (`cust_id`), tên khách hàng (`cust_name`) và tên đăng nhập (`user_login`), trong đó tên đăng nhập toàn bộ là chữ in hoa và được tạo từ hai ký tự đầu trong thông tin liên hệ của khách hàng (`cust_contact`) cùng ba ký tự đầu của thành phố (`cust_city`). Gợi ý: cần dùng function, phép nối chuỗi và alias.

Đáp án:

```sql
SELECT cust_id, cust_name, UPPER(CONCAT(SUBSTRING(cust_contact, 1, 2), SUBSTRING(cust_city, 1, 3))) AS user_login
FROM Customers
```

Điểm cần biết:

- Function cắt chuỗi `SUBSTRING()`: cắt chuỗi, `substring(str ,n ,m)` (n biểu thị vị trí bắt đầu cắt, m biểu thị số ký tự cần cắt) nghĩa là trả về m ký tự được cắt từ ký tự thứ n của chuỗi str.
- Function nối chuỗi `CONCAT()`: nối hai hoặc nhiều chuỗi thành một chuỗi, `select concat(A,B)`: nối chuỗi A và B.

- Function viết hoa `UPPER()`: chuyển chuỗi được chỉ định thành chữ in hoa.

### Trả về mã và ngày của tất cả đơn hàng trong tháng 1 năm 2020

Bảng đơn hàng `Orders` như sau:

| order_num | order_date          |
| --------- | ------------------- |
| a0001     | 2020-01-01 00:00:00 |
| a0002     | 2020-01-02 00:00:00 |
| a0003     | 2020-01-01 12:00:00 |
| a0004     | 2020-02-01 00:00:00 |
| a0005     | 2020-03-01 00:00:00 |

【Bài toán】Viết câu lệnh SQL để trả về mã đơn hàng (`order_num`) và ngày đặt hàng (`order_date`) của tất cả đơn hàng trong tháng 1 năm 2020, đồng thời sắp xếp tăng dần theo ngày đặt hàng.

Đáp án:

```sql
SELECT order_num, order_date
FROM Orders
WHERE month(order_date) = '01' AND YEAR(order_date) = '2020'
ORDER BY order_date
```

Cũng có thể dùng wildcard:

```sql
SELECT order_num, order_date
FROM Orders
WHERE order_date LIKE '2020-01%'
ORDER BY order_date
```

Điểm cần biết:

- Định dạng ngày: `YYYY-MM-DD`
- Định dạng thời gian: `HH:MM:SS`

Các function thường dùng để xử lý ngày và thời gian:

| Function        | Mô tả                                             |
| --------------- | ------------------------------------------------- |
| `ADDDATE()`     | Cộng thêm một khoảng thời gian (ngày, tuần, v.v.) |
| `ADDTIME()`     | Thêm một khoảng thời gian (giờ, phút, v.v.)       |
| `CURDATE()`     | Trả về ngày hiện tại                              |
| `CURTIME()`     | Trả về thời gian hiện tại                         |
| `DATE()`        | Trả về phần ngày của datetime                     |
| `DATEDIFF`      | Tính chênh lệch giữa hai ngày                     |
| `DATE_FORMAT()` | Trả về chuỗi ngày hoặc thời gian đã format        |
| `DAY()`         | Trả về phần ngày trong một ngày                   |
| `DAYOFWEEK()`   | Trả về thứ trong tuần tương ứng với một ngày      |
| `HOUR()`        | Trả về phần giờ của một thời gian                 |
| `MINUTE()`      | Trả về phần phút của một thời gian                |
| `MONTH()`       | Trả về phần tháng của một ngày                    |
| `NOW()`         | Trả về ngày và thời gian hiện tại                 |
| `SECOND()`      | Trả về phần giây của một thời gian                |
| `TIME()`        | Trả về phần thời gian của datetime                |
| `YEAR()`        | Trả về phần năm của một ngày                      |

## Tổng hợp dữ liệu

Các function liên quan đến tổng hợp dữ liệu:

| Function  | Mô tả                                 |
| --------- | ------------------------------------- |
| `AVG()`   | Trả về giá trị trung bình của một cột |
| `COUNT()` | Trả về số dòng của một cột            |
| `MAX()`   | Trả về giá trị lớn nhất của một cột   |
| `MIN()`   | Trả về giá trị nhỏ nhất của một cột   |
| `SUM()`   | Trả về tổng các giá trị của một cột   |

### Xác định tổng số sản phẩm đã bán

Bảng `OrderItems` đại diện cho các sản phẩm đã bán, `quantity` đại diện cho số lượng sản phẩm đã bán.

| quantity |
| -------- |
| 10       |
| 100      |
| 1000     |
| 10001    |
| 2        |
| 15       |

【Bài toán】Viết câu lệnh SQL để xác định tổng số sản phẩm đã bán.

Đáp án:

```sql
SELECT Sum(quantity) AS items_ordered
FROM OrderItems
```

### Xác định tổng số sản phẩm BR01 đã bán

Bảng `OrderItems` đại diện cho các sản phẩm đã bán, `quantity` đại diện cho số lượng sản phẩm đã bán và `prod_id` là ID sản phẩm.

| quantity | prod_id |
| -------- | ------- |
| 10       | AR01    |
| 100      | AR10    |
| 1000     | BR01    |
| 10001    | BR010   |

【Bài toán】Sửa câu lệnh đã tạo để xác định tổng số sản phẩm (`prod_id`) `BR01` đã bán.

Đáp án:

```sql
SELECT Sum(quantity) AS items_ordered
FROM OrderItems
WHERE prod_id = 'BR01'
```

### Xác định giá của sản phẩm đắt nhất trong bảng Products có giá không quá 10 đô la

Bảng `Products` như sau, `prod_price` đại diện cho giá sản phẩm.

| prod_price |
| ---------- |
| 9.49       |
| 600        |
| 1000       |

【Bài toán】Viết câu lệnh SQL để xác định giá (`prod_price`) của sản phẩm đắt nhất trong bảng `Products` có giá không quá 10 đô la. Đặt tên calculated field là `max_price`.

Đáp án:

```sql
SELECT Max(prod_price) AS max_price
FROM Products
WHERE prod_price <= 10
```

## Nhóm dữ liệu

`GROUP BY`:

- Mệnh đề `GROUP BY` nhóm các bản ghi thành các dòng tổng hợp.
- `GROUP BY` trả về một bản ghi cho mỗi nhóm.
- `GROUP BY` thường đi kèm các aggregate function như `COUNT`, `MAX`, `SUM`, `AVG`.
- `GROUP BY` có thể nhóm theo một hoặc nhiều cột.
- Sau khi `GROUP BY` theo trường nhóm, `ORDER BY` có thể sắp xếp theo trường tổng hợp.

`HAVING`:

- `HAVING` dùng để lọc kết quả đã tổng hợp của `GROUP BY`.
- `HAVING` phải được dùng cùng `GROUP BY`.
- `WHERE` và `HAVING` có thể cùng xuất hiện trong một query.

`HAVING` và `WHERE`:

- `WHERE`: lọc các dòng được chỉ định, phía sau không thể thêm aggregate function (group function).
- `HAVING`: lọc nhóm, phải được dùng cùng `GROUP BY`, không thể dùng riêng.

### Trả về số dòng của từng mã đơn hàng

Bảng `OrderItems` chứa từng sản phẩm của mỗi đơn hàng.

| order_num |
| --------- |
| a002      |
| a002      |
| a002      |
| a004      |
| a007      |

【Bài toán】Viết câu lệnh SQL để trả về số dòng (`order_lines`) của từng mã đơn hàng (`order_num`) và sắp xếp kết quả tăng dần theo `order_lines`.

Đáp án:

```sql
SELECT order_num, Count(order_num) AS order_lines
FROM OrderItems
GROUP BY order_num
ORDER BY order_lines
```

Điểm cần biết:

1. `count(*)`, `count(tên_cột)` đều dùng được; điểm khác nhau là `count(tên_cột)` đếm số dòng không phải NULL.
2. `order by` thực thi cuối cùng nên có thể dùng column alias.
3. Khi aggregate theo nhóm, nhất định không được quên thêm `group by`, nếu không sẽ chỉ có một dòng kết quả.

### Sản phẩm có chi phí thấp nhất của mỗi nhà cung cấp

Có bảng `Products`, trong đó trường `prod_price` đại diện cho giá sản phẩm và `vend_id` đại diện cho id nhà cung cấp.

| vend_id | prod_price |
| ------- | ---------- |
| a0011   | 100        |
| a0019   | 0.1        |
| b0019   | 1000       |
| b0019   | 6980       |
| b0019   | 20         |

【Bài toán】Viết câu lệnh SQL để trả về một trường tên `cheapest_item`, trường này chứa sản phẩm có chi phí thấp nhất của mỗi nhà cung cấp (sử dụng `prod_price` trong bảng `Products`), sau đó sắp xếp kết quả tăng dần từ chi phí thấp nhất đến cao nhất.

Đáp án:

```sql
SELECT vend_id, Min(prod_price) AS cheapest_item
FROM Products
GROUP BY vend_id
ORDER BY cheapest_item
```

### Trả về mã của tất cả đơn hàng có tổng số lượng không nhỏ hơn 100

`OrderItems` đại diện cho bảng sản phẩm đơn hàng, bao gồm mã đơn hàng `order_num` và số lượng sản phẩm `quantity`.

| order_num | quantity |
| --------- | -------- |
| a1        | 105      |
| a2        | 1100     |
| a2        | 200      |
| a4        | 1121     |
| a5        | 10       |
| a2        | 19       |
| a7        | 5        |

【Bài toán】Viết câu lệnh SQL để trả về tất cả mã đơn hàng có tổng số lượng sản phẩm không nhỏ hơn 100, cuối cùng sắp xếp kết quả tăng dần theo mã đơn hàng.

Đáp án:

```sql
# Aggregate trực tiếp
SELECT order_num
FROM OrderItems
GROUP BY order_num
HAVING Sum(quantity) >= 100
ORDER BY order_num

# Subquery
SELECT a.order_num
FROM (SELECT order_num, Sum(quantity) AS sum_num
    FROM OrderItems
    GROUP BY order_num
    HAVING sum_num >= 100) a
ORDER BY a.order_num
```

Điểm cần biết:

- `where`: lọc các dòng được chỉ định, phía sau không thể thêm aggregate function (group function).
- `having`: lọc nhóm, dùng cùng `group by`, không thể dùng riêng.

### Tính tổng

Bảng `OrderItems` đại diện cho thông tin đơn hàng, bao gồm các trường: mã đơn hàng `order_num`, giá bán sản phẩm `item_price` và số lượng sản phẩm `quantity`.

| order_num | item_price | quantity |
| --------- | ---------- | -------- |
| a1        | 10         | 105      |
| a2        | 1          | 1100     |
| a2        | 1          | 200      |
| a4        | 2          | 1121     |
| a5        | 5          | 10       |
| a2        | 1          | 19       |
| a7        | 7          | 5        |

【Bài toán】Viết câu lệnh SQL để aggregate theo mã đơn hàng, trả về tất cả mã đơn hàng có tổng giá không nhỏ hơn 1000, cuối cùng sắp xếp kết quả tăng dần theo mã đơn hàng.

Gợi ý: tổng giá = `item_price` nhân với `quantity`.

Đáp án:

```sql
SELECT order_num, Sum(item_price * quantity) AS total_price
FROM OrderItems
GROUP BY order_num
HAVING total_price >= 1000
ORDER BY order_num
```

### Kiểm tra câu lệnh SQL

Bảng `OrderItems` có mã đơn hàng `order_num`.

| order_num |
| --------- |
| a002      |
| a002      |
| a002      |
| a004      |
| a007      |

【Bài toán】Sửa đoạn code dưới đây cho đúng rồi thực thi.

```sql
SELECT order_num, COUNT(*) AS items
FROM OrderItems
GROUP BY items
HAVING COUNT(*) >= 3
ORDER BY items, order_num;
```

Sau khi sửa:

```sql
SELECT order_num, COUNT(*) AS items
FROM OrderItems
GROUP BY order_num
HAVING items >= 3
ORDER BY items, order_num;
```

## Dùng subquery

Subquery là một SQL query được lồng trong query lớn hơn, còn gọi là inner query hoặc inner select. Câu lệnh chứa subquery cũng được gọi là outer query hoặc outer select. Nói đơn giản, subquery là việc dùng kết quả của một query `SELECT` (subquery) làm nguồn dữ liệu hoặc điều kiện kiểm tra của một SQL statement khác (main query).

Subquery có thể được nhúng trong các statement `SELECT`, `INSERT`, `UPDATE` và `DELETE`, đồng thời có thể dùng cùng các toán tử `=`, `<`, `>`, `IN`, `BETWEEN`, `EXISTS`, v.v.

Subquery thường được dùng trong mệnh đề `WHERE` và mệnh đề `FROM`:

- Khi dùng trong mệnh đề `WHERE`, tùy toán tử mà subquery có thể trả về dữ liệu một dòng một cột, nhiều dòng một cột hoặc một dòng nhiều cột. Subquery cần trả về giá trị có thể làm điều kiện truy vấn của mệnh đề WHERE.
- Khi dùng trong mệnh đề `FROM`, subquery thường trả về dữ liệu nhiều dòng nhiều cột, tương đương một temporary table để phù hợp với quy tắc phần sau `FROM` phải là một bảng. Cách này có thể thực hiện query kết hợp nhiều bảng.

> Lưu ý: Database MySQL bắt đầu hỗ trợ subquery từ phiên bản 4.1, các phiên bản cũ hơn không hỗ trợ.

Cú pháp cơ bản của subquery dùng trong mệnh đề `WHERE`:

```sql
SELECT column_name [, column_name ]
FROM table1 [, table2 ]
WHERE column_name operator
(SELECT column_name [, column_name ]
FROM table1 [, table2 ]
[WHERE])
```

- Subquery cần đặt trong dấu ngoặc `( )`.
- `operator` biểu thị toán tử dùng cho mệnh đề `WHERE`, có thể là toán tử so sánh (như `=`, `<`, `>`, `<>`, v.v.) hoặc toán tử logic (như `IN`, `NOT IN`, `EXISTS`, `NOT EXISTS`, v.v.), tùy theo nhu cầu.

Cú pháp cơ bản của subquery dùng trong mệnh đề `FROM`:

```sql
SELECT column_name [, column_name ]
FROM (SELECT column_name [, column_name ]
      FROM table1 [, table2 ]
      [WHERE]) AS temp_table_name [, ...]
[JOIN type JOIN table_name ON condition]
WHERE condition;
```

- Kết quả của subquery dùng cho `FROM` tương đương một temporary table, vì vậy cần dùng keyword AS để đặt tên cho temporary table đó.
- Subquery cần đặt trong dấu ngoặc `( )`.
- Có thể chỉ định nhiều tên temporary table và dùng statement `JOIN` để nối các bảng này.

### Trả về danh sách khách hàng mua sản phẩm có giá từ 10 đô la trở lên

Bảng `OrderItems` đại diện cho bảng sản phẩm đơn hàng, có các trường mã đơn hàng `order_num` và giá đơn hàng `item_price`; bảng `Orders` đại diện cho bảng thông tin đơn hàng, có ID khách hàng `cust_id` và mã đơn hàng `order_num`.

Bảng `OrderItems`:

| order_num | item_price |
| --------- | ---------- |
| a1        | 10         |
| a2        | 1          |
| a2        | 1          |
| a4        | 2          |
| a5        | 5          |
| a2        | 1          |
| a7        | 7          |

Bảng `Orders`:

| order_num | cust_id |
| --------- | ------- |
| a1        | cust10  |
| a2        | cust1   |
| a2        | cust1   |
| a4        | cust2   |
| a5        | cust5   |
| a2        | cust1   |
| a7        | cust7   |

【Bài toán】Dùng subquery để trả về danh sách khách hàng mua sản phẩm có giá từ 10 đô la trở lên, không cần sắp xếp kết quả.

Đáp án:

```sql
SELECT cust_id
FROM Orders
WHERE order_num IN (SELECT DISTINCT order_num
    FROM OrderItems
    where item_price >= 10)
```

### Xác định các đơn hàng đã mua sản phẩm có prod_id là BR01 (1)

Bảng `OrderItems` đại diện cho bảng thông tin sản phẩm đơn hàng, `prod_id` là id sản phẩm; bảng `Orders` đại diện cho bảng đơn hàng, có `cust_id` là id khách hàng và ngày đặt hàng `order_date`.

Bảng `OrderItems`:

| prod_id | order_num |
| ------- | --------- |
| BR01    | a0001     |
| BR01    | a0002     |
| BR02    | a0003     |
| BR02    | a0013     |

Bảng `Orders`:

| order_num | cust_id | order_date          |
| --------- | ------- | ------------------- |
| a0001     | cust10  | 2022-01-01 00:00:00 |
| a0002     | cust1   | 2022-01-01 00:01:00 |
| a0003     | cust1   | 2022-01-02 00:00:00 |
| a0013     | cust2   | 2022-01-01 00:20:00 |

【Bài toán】Viết câu lệnh SQL, sử dụng subquery để xác định các đơn hàng (trong `OrderItems`) đã mua sản phẩm có `prod_id` là `BR01`, sau đó trả về ID khách hàng (`cust_id`) và ngày đặt hàng (`order_date`) tương ứng với từng sản phẩm từ bảng `Orders`, sắp xếp kết quả tăng dần theo ngày đặt hàng.

Đáp án:

```sql
# Cách 1: subquery
SELECT cust_id,order_date
FROM Orders
WHERE order_num IN
    (SELECT order_num
     FROM OrderItems
     WHERE prod_id = 'BR01' )
ORDER BY order_date;

# Cách 2: nối bảng
SELECT b.cust_id, b.order_date
FROM OrderItems a,Orders b
WHERE a.order_num = b.order_num AND a.prod_id = 'BR01'
ORDER BY order_date
```

### Trả về email của tất cả khách hàng mua sản phẩm có prod_id là BR01 (1)

Bạn muốn biết ngày đặt sản phẩm BR01. Có bảng `OrderItems` đại diện cho bảng thông tin sản phẩm đơn hàng, `prod_id` là id sản phẩm; bảng `Orders` đại diện cho bảng đơn hàng, có `cust_id` là id khách hàng và ngày đặt hàng `order_date`; bảng `Customers` có email khách hàng `cust_email` và id khách hàng `cust_id`.

Bảng `OrderItems`:

| prod_id | order_num |
| ------- | --------- |
| BR01    | a0001     |
| BR01    | a0002     |
| BR02    | a0003     |
| BR02    | a0013     |

Bảng `Orders`:

| order_num | cust_id | order_date          |
| --------- | ------- | ------------------- |
| a0001     | cust10  | 2022-01-01 00:00:00 |
| a0002     | cust1   | 2022-01-01 00:01:00 |
| a0003     | cust1   | 2022-01-02 00:00:00 |
| a0013     | cust2   | 2022-01-01 00:20:00 |

Bảng `Customers` đại diện cho thông tin khách hàng, `cust_id` là id khách hàng, `cust_email` là email khách hàng.

| cust_id | cust_email        |
| ------- | ----------------- |
| cust10  | <cust10@cust.com> |
| cust1   | <cust1@cust.com>  |
| cust2   | <cust2@cust.com>  |

【Bài toán】Trả về email của tất cả khách hàng mua sản phẩm có `prod_id` là BR01 (`cust_email` trong bảng `Customers`), không cần sắp xếp kết quả.

Gợi ý: Bài này liên quan đến statement `SELECT`, trong đó phần trong cùng trả về `order_num` từ bảng `OrderItems`, phần ở giữa trả về `cust_id` từ bảng `Customers`.

Đáp án:

```sql
# Cách 1: subquery
SELECT cust_email
FROM Customers
WHERE cust_id IN (SELECT cust_id
    FROM Orders
    WHERE order_num IN (SELECT order_num
        FROM OrderItems
        WHERE prod_id = 'BR01'))

# Cách 2: nối bảng (inner join)
SELECT c.cust_email
FROM OrderItems a,Orders b,Customers c
WHERE a.order_num = b.order_num AND b.cust_id = c.cust_id AND a.prod_id = 'BR01'

# Cách 3: nối bảng (left join)
SELECT c.cust_email
FROM Orders a LEFT JOIN
  OrderItems b ON a.order_num = b.order_num LEFT JOIN
  Customers c ON a.cust_id = c.cust_id
WHERE b.prod_id = 'BR01'
```

### Tổng tiền của các đơn hàng khác nhau theo từng khách hàng

Cần một danh sách ID khách hàng, trong đó bao gồm tổng số tiền họ đã đặt hàng.

Bảng `OrderItems` đại diện cho thông tin đơn hàng, có mã đơn hàng `order_num`, giá bán sản phẩm `item_price` và số lượng sản phẩm `quantity`.

| order_num | item_price | quantity |
| --------- | ---------- | -------- |
| a0001     | 10         | 105      |
| a0002     | 1          | 1100     |
| a0002     | 1          | 200      |
| a0013     | 2          | 1121     |
| a0003     | 5          | 10       |
| a0003     | 1          | 19       |
| a0003     | 7          | 5        |

Bảng `Orders` có mã đơn hàng `order_num` và id khách hàng `cust_id`.

| order_num | cust_id |
| --------- | ------- |
| a0001     | cust10  |
| a0002     | cust1   |
| a0003     | cust1   |
| a0013     | cust2   |

【Bài toán】Viết câu lệnh SQL để trả về ID khách hàng (`cust_id` trong bảng `Orders`) và dùng subquery để trả về `total_ordered`, qua đó trả về tổng tiền đơn hàng của mỗi khách hàng; sắp xếp kết quả theo số tiền giảm dần.

Đáp án:

```sql
# Cách 1: subquery
SELECT o.cust_id, SUM(tb.total_ordered) AS `total_ordered`
FROM (SELECT order_num, SUM(item_price * quantity) AS total_ordered
    FROM OrderItems
    GROUP BY order_num) AS tb,
  Orders o
WHERE tb.order_num = o.order_num
GROUP BY o.cust_id
ORDER BY total_ordered DESC;

# Cách 2: nối bảng
SELECT b.cust_id, Sum(a.quantity * a.item_price) AS total_ordered
FROM OrderItems a,Orders b
WHERE a.order_num = b.order_num
GROUP BY cust_id
ORDER BY total_ordered DESC
```

Có thể tham khảo giới thiệu chi tiết về cách 1 tại: [issue#2402: lỗi và cách sửa của cách 1](https://github.com/Snailclimb/JavaGuide/issues/2402).

### Truy xuất tất cả tên sản phẩm và tổng số lượng bán tương ứng từ bảng Products

Truy xuất tất cả tên sản phẩm `prod_name` và id sản phẩm `prod_id` trong bảng `Products`.

| prod_id | prod_name |
| ------- | --------- |
| a0001   | egg       |
| a0002   | sockets   |
| a0013   | coffee    |
| a0003   | cola      |

`OrderItems` đại diện cho bảng sản phẩm đơn hàng, gồm sản phẩm đơn hàng `prod_id` và số lượng đã bán `quantity`.

| prod_id | quantity |
| ------- | -------- |
| a0001   | 105      |
| a0002   | 1100     |
| a0002   | 200      |
| a0013   | 1121     |
| a0003   | 10       |
| a0003   | 19       |
| a0003   | 5        |

【Bài toán】Viết câu lệnh SQL để truy xuất tất cả tên sản phẩm (`prod_name`) từ bảng `Products` và một calculated column tên `quant_sold`, trong đó chứa tổng số sản phẩm đã bán (dùng subquery và `SUM(quantity)` trên bảng `OrderItems`).

Đáp án:

```sql
# Cách 1: subquery
SELECT p.prod_name, tb.quant_sold
FROM (SELECT prod_id, Sum(quantity) AS quant_sold
    FROM OrderItems
    GROUP BY prod_id) AS tb,
  Products p
WHERE tb.prod_id = p.prod_id

# Cách 2: nối bảng
SELECT p.prod_name, Sum(o.quantity) AS quant_sold
FROM Products p,
  OrderItems o
WHERE p.prod_id = o.prod_id
GROUP BY p.prod_name (ở đây không thể dùng p.prod_id, sẽ báo lỗi)
```

## Nối bảng

JOIN có nghĩa là “nối”. Đúng như tên gọi, mệnh đề SQL JOIN dùng để kết hợp hai hoặc nhiều bảng thành một query.

Khi nối bảng, cần chọn một trường trong mỗi bảng và so sánh giá trị của các trường đó; hai bản ghi có giá trị giống nhau sẽ được gộp thành một. **Bản chất của việc nối bảng là gộp các bản ghi của những bảng khác nhau thành một bảng mới. Tất nhiên, bảng mới này chỉ là tạm thời và chỉ tồn tại trong thời gian query hiện tại.**

Cú pháp cơ bản để dùng `JOIN` nối hai bảng:

```sql
SELECT table1.column1, table2.column2...
FROM table1
JOIN table2
ON table1.common_column1 = table2.common_column2;
```

`table1.common_column1 = table2.common_column2` là điều kiện nối, chỉ các bản ghi thỏa mãn điều kiện này mới được gộp thành một dòng. Có thể dùng nhiều toán tử để nối bảng, chẳng hạn =, >, <, <>, <=, >=, !=, `between`, `like` hoặc `not`, nhưng phổ biến nhất là =.

Khi hai bảng có trường trùng tên, để database engine phân biệt trường thuộc bảng nào, khi viết tên trường trùng nhau cần thêm tên bảng. Nếu tên trường là duy nhất trong hai bảng thì có thể không dùng định dạng trên mà chỉ viết tên trường.

Ngoài ra, nếu tên trường liên kết của hai bảng giống nhau, có thể dùng mệnh đề `USING` thay cho `ON`, ví dụ:

```sql
# join....on
SELECT c.cust_name, o.order_num
FROM Customers c
INNER JOIN Orders o
ON c.cust_id = o.cust_id
ORDER BY c.cust_name

# Nếu tên trường liên kết của hai bảng giống nhau, cũng có thể dùng mệnh đề USING: JOIN....USING()
SELECT c.cust_name, o.order_num
FROM Customers c
INNER JOIN Orders o
USING(cust_id)
ORDER BY c.cust_name
```

**Sự khác nhau giữa `ON` và `WHERE`:**

- Khi nối bảng, SQL tạo một temporary table theo điều kiện nối. `ON` là điều kiện nối, quyết định việc tạo temporary table.
- `WHERE` lọc dữ liệu trong temporary table sau khi temporary table được tạo để tạo result set cuối cùng; ở thời điểm này không còn JOIN-ON nữa.

Tóm lại: **SQL trước tiên tạo một temporary table dựa trên ON, sau đó lọc temporary table dựa trên WHERE.**

SQL cho phép thêm một số keyword bổ nghĩa ở bên trái `JOIN` để tạo các kiểu nối khác nhau như bảng dưới đây:

| Kiểu nối                                        | Mô tả                                                                                                                           |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| INNER JOIN nối trong                            | (Kiểu nối mặc định) Chỉ trả về dòng khi cả hai bảng đều có bản ghi thỏa mãn điều kiện.                                          |
| LEFT JOIN / LEFT OUTER JOIN nối trái (ngoài)    | Trả về tất cả dòng trong bảng trái, kể cả khi bảng phải không có dòng thỏa mãn điều kiện.                                       |
| RIGHT JOIN / RIGHT OUTER JOIN nối phải (ngoài)  | Trả về tất cả dòng trong bảng phải, kể cả khi bảng trái không có dòng thỏa mãn điều kiện.                                       |
| FULL JOIN / FULL OUTER JOIN nối toàn bộ (ngoài) | Trả về dòng nếu ít nhất một trong hai bảng có bản ghi thỏa mãn điều kiện.                                                       |
| SELF JOIN                                       | Nối một bảng với chính nó như thể bảng đó là hai bảng. Để phân biệt hai bảng, cần đổi tên ít nhất một bảng trong SQL statement. |
| CROSS JOIN                                      | Nối chéo, trả về tích Cartesian của các record set từ hai hoặc nhiều bảng được nối.                                             |

Hình dưới đây minh họa 7 cách dùng liên quan đến LEFT JOIN, RIGHT JOIN, INNER JOIN và OUTER JOIN.

![](https://oss.javaguide.cn/github/javaguide/csdn/d1794312b448516831369f869814ab39.png)

Nếu không thêm modifier nào mà chỉ viết `JOIN`, mặc định sẽ là `INNER JOIN`.

Với `INNER JOIN`, còn có một cách viết ngầm gọi là “**inner join ngầm**”, tức là không dùng keyword `INNER JOIN` mà dùng statement `WHERE` để thực hiện chức năng inner join.

```sql
# Inner join ngầm
SELECT c.cust_name, o.order_num
FROM Customers c,Orders o
WHERE c.cust_id = o.cust_id
ORDER BY c.cust_name

# Inner join tường minh
SELECT c.cust_name, o.order_num
FROM Customers c
INNER JOIN Orders o
USING(cust_id)
ORDER BY c.cust_name;
```

### Trả về tên khách hàng và mã đơn hàng tương ứng

Bảng `Customers` có các trường tên khách hàng `cust_name` và id khách hàng `cust_id`.

| cust_id  | cust_name |
| -------- | --------- |
| cust10   | andy      |
| cust1    | ben       |
| cust2    | tony      |
| cust22   | tom       |
| cust221  | an        |
| cust2217 | hex       |

Bảng thông tin đơn hàng `Orders` có mã đơn hàng `order_num` và id khách hàng `cust_id`.

| order_num | cust_id  |
| --------- | -------- |
| a1        | cust10   |
| a2        | cust1    |
| a3        | cust2    |
| a4        | cust22   |
| a5        | cust221  |
| a7        | cust2217 |

【Bài toán】Viết câu lệnh SQL để trả về tên khách hàng (`cust_name`) trong bảng `Customers` và mã đơn hàng tương ứng (`order_num`) trong bảng `Orders`, sau đó sắp xếp tăng dần theo tên khách hàng rồi theo mã đơn hàng. Có thể thử hai cách viết: một cách dùng cú pháp equi-join đơn giản và một cách dùng `INNER JOIN`.

Đáp án:

```sql
# Inner join ngầm
SELECT c.cust_name, o.order_num
FROM Customers c,Orders o
WHERE c.cust_id = o.cust_id
ORDER BY c.cust_name,o.order_num

# Inner join tường minh
SELECT c.cust_name, o.order_num
FROM Customers c
INNER JOIN Orders o
USING(cust_id)
ORDER BY c.cust_name,o.order_num;
```

### Trả về tên khách hàng, mã đơn hàng tương ứng và tổng giá của mỗi đơn hàng

Bảng `Customers` có các trường tên khách hàng `cust_name` và id khách hàng `cust_id`.

| cust_id  | cust_name |
| -------- | --------- |
| cust10   | andy      |
| cust1    | ben       |
| cust2    | tony      |
| cust22   | tom       |
| cust221  | an        |
| cust2217 | hex       |

Bảng thông tin đơn hàng `Orders` có mã đơn hàng `order_num` và id khách hàng `cust_id`.

| order_num | cust_id  |
| --------- | -------- |
| a1        | cust10   |
| a2        | cust1    |
| a3        | cust2    |
| a4        | cust22   |
| a5        | cust221  |
| a7        | cust2217 |

Bảng `OrderItems` có mã đơn hàng sản phẩm `order_num`, số lượng sản phẩm `quantity` và giá sản phẩm `item_price`.

| order_num | quantity | item_price |
| --------- | -------- | ---------- |
| a1        | 1000     | 10         |
| a2        | 200      | 10         |
| a3        | 10       | 15         |
| a4        | 25       | 50         |
| a5        | 15       | 25         |
| a7        | 7        | 7          |

【Bài toán】Ngoài tên khách hàng và mã đơn hàng, hãy thêm cột thứ ba `OrderTotal` chứa tổng giá của mỗi đơn hàng, sau đó sắp xếp tăng dần theo tên khách hàng rồi theo mã đơn hàng.

```sql
# Cú pháp equi-join đơn giản
SELECT c.cust_name, o.order_num, SUM(quantity * item_price) AS OrderTotal
FROM Customers c,Orders o,OrderItems oi
WHERE c.cust_id = o.cust_id AND o.order_num = oi.order_num
GROUP BY c.cust_name, o.order_num
ORDER BY c.cust_name, o.order_num
```

Lưu ý, có thể có bạn sẽ viết như sau:

```sql
SELECT c.cust_name, o.order_num, SUM(quantity * item_price) AS OrderTotal
FROM Customers c,Orders o,OrderItems oi
WHERE c.cust_id = o.cust_id AND o.order_num = oi.order_num
GROUP BY c.cust_name
ORDER BY c.cust_name,o.order_num
```

Cách này sai! Chỉ `GROUP BY` theo `cust_name` đúng với yêu cầu, nhưng không hợp lệ về cú pháp `GROUP BY`.

Trong statement `SELECT`, nếu không có `GROUP BY`, `cust_name` và `order_num` sẽ trả về nhiều giá trị, còn `SUM(quantity * item_price)` chỉ trả về một giá trị. Dùng `GROUP BY cust_name` giúp `cust_name` và `SUM(quantity * item_price)` tương ứng một-một, hay nói cách khác là **aggregate**; tương tự, cũng cần aggregate theo `order_num`.

> **Tóm lại, các trường trong select hoặc phải cùng aggregate, hoặc đều không aggregate.**

### Xác định các đơn hàng đã mua sản phẩm có prod_id là BR01 (2)

Bảng `OrderItems` đại diện cho bảng thông tin sản phẩm đơn hàng, `prod_id` là id sản phẩm; bảng `Orders` đại diện cho bảng đơn hàng, có `cust_id` là id khách hàng và ngày đặt hàng `order_date`.

Bảng `OrderItems`:

| prod_id | order_num |
| ------- | --------- |
| BR01    | a0001     |
| BR01    | a0002     |
| BR02    | a0003     |
| BR02    | a0013     |

Bảng `Orders`:

| order_num | cust_id | order_date          |
| --------- | ------- | ------------------- |
| a0001     | cust10  | 2022-01-01 00:00:00 |
| a0002     | cust1   | 2022-01-01 00:01:00 |
| a0003     | cust1   | 2022-01-02 00:00:00 |
| a0013     | cust2   | 2022-01-01 00:20:00 |

【Bài toán】Viết câu lệnh SQL, sử dụng subquery để xác định các đơn hàng (trong `OrderItems`) đã mua sản phẩm có `prod_id` là `BR01`, sau đó trả về ID khách hàng (`cust_id`) và ngày đặt hàng (`order_date`) tương ứng với từng sản phẩm từ bảng `Orders`, sắp xếp kết quả tăng dần theo ngày đặt hàng.

Gợi ý: lần này dùng phép nối và cú pháp equi-join đơn giản.

```sql
# Cách 1: subquery
SELECT cust_id, order_date
FROM Orders
WHERE order_num IN (SELECT order_num
    FROM OrderItems
    WHERE prod_id = 'BR01')
ORDER BY order_date

# Cách 2: nối bảng inner join
SELECT cust_id, order_date
FROM Orders o INNER JOIN
  (SELECT order_num
    FROM OrderItems
    WHERE prod_id = 'BR01') tb ON o.order_num = tb.order_num
ORDER BY order_date

# Cách 3: phiên bản rút gọn của cách 2
SELECT cust_id, order_date
FROM Orders
INNER JOIN OrderItems USING(order_num)
WHERE OrderItems.prod_id = 'BR01'
ORDER BY order_date
```

### Trả về email của tất cả khách hàng mua sản phẩm có prod_id là BR01 (2)

Có bảng `OrderItems` đại diện cho bảng thông tin sản phẩm đơn hàng, `prod_id` là id sản phẩm; bảng `Orders` đại diện cho bảng đơn hàng, có `cust_id` là id khách hàng và ngày đặt hàng `order_date`; bảng `Customers` có `cust_email` là email khách hàng và `cust_id` là id khách hàng.

Bảng `OrderItems`:

| prod_id | order_num |
| ------- | --------- |
| BR01    | a0001     |
| BR01    | a0002     |
| BR02    | a0003     |
| BR02    | a0013     |

Bảng `Orders`:

| order_num | cust_id | order_date          |
| --------- | ------- | ------------------- |
| a0001     | cust10  | 2022-01-01 00:00:00 |
| a0002     | cust1   | 2022-01-01 00:01:00 |
| a0003     | cust1   | 2022-01-02 00:00:00 |
| a0013     | cust2   | 2022-01-01 00:20:00 |

Bảng `Customers` đại diện cho thông tin khách hàng, `cust_id` là id khách hàng, `cust_email` là email khách hàng.

| cust_id | cust_email        |
| ------- | ----------------- |
| cust10  | <cust10@cust.com> |
| cust1   | <cust1@cust.com>  |
| cust2   | <cust2@cust.com>  |

【Bài toán】Trả về email của tất cả khách hàng mua sản phẩm có `prod_id` là BR01 (`cust_email` trong bảng `Customers`), không cần sắp xếp kết quả.

Gợi ý: có liên quan đến statement `SELECT`, trong đó phần trong cùng trả về `order_num` từ bảng `OrderItems`, phần ở giữa trả về `cust_id` từ bảng `Customers`, nhưng bắt buộc dùng cú pháp `INNER JOIN`.

```sql
SELECT cust_email
FROM Customers
INNER JOIN Orders using(cust_id)
INNER JOIN OrderItems using(order_num)
WHERE OrderItems.prod_id = 'BR01'
```

### Một cách khác để xác định khách hàng tốt nhất (2)

Bảng `OrderItems` đại diện cho thông tin đơn hàng. Một cách khác để xác định khách hàng tốt nhất là xem họ đã chi bao nhiêu tiền. Bảng `OrderItems` có mã đơn hàng `order_num`, giá bán sản phẩm `item_price` và số lượng sản phẩm `quantity`.

| order_num | item_price | quantity |
| --------- | ---------- | -------- |
| a1        | 10         | 105      |
| a2        | 1          | 1100     |
| a2        | 1          | 200      |
| a4        | 2          | 1121     |
| a5        | 5          | 10       |
| a2        | 1          | 19       |
| a7        | 7          | 5        |

Bảng `Orders` có các trường mã đơn hàng `order_num` và id khách hàng `cust_id`.

| order_num | cust_id  |
| --------- | -------- |
| a1        | cust10   |
| a2        | cust1    |
| a3        | cust2    |
| a4        | cust22   |
| a5        | cust221  |
| a7        | cust2217 |

Bảng khách hàng `Customers` có các trường id khách hàng `cust_id` và tên khách hàng `cust_name`.

| cust_id  | cust_name |
| -------- | --------- |
| cust10   | andy      |
| cust1    | ben       |
| cust2    | tony      |
| cust22   | tom       |
| cust221  | an        |
| cust2217 | hex       |

【Bài toán】Viết câu lệnh SQL để trả về tên khách hàng và tổng tiền của họ, tính từ các mã đơn hàng (`order_num`) trong bảng `OrderItems`, chỉ với tổng giá không nhỏ hơn 1000.

Gợi ý: cần tính tổng (`item_price` nhân với `quantity`). Sắp xếp kết quả theo tổng tiền và dùng cú pháp `INNER JOIN`.

```sql
SELECT cust_name, SUM(item_price * quantity) AS total_price
FROM Customers
INNER JOIN Orders USING(cust_id)
INNER JOIN OrderItems USING(order_num)
GROUP BY cust_name
HAVING total_price >= 1000
ORDER BY total_price
```

## Tạo phép nối nâng cao

### Truy xuất tên và tất cả mã đơn hàng của từng khách hàng (1)

Bảng `Customers` đại diện cho thông tin khách hàng, có id khách hàng `cust_id` và tên khách hàng `cust_name`.

| cust_id  | cust_name |
| -------- | --------- |
| cust10   | andy      |
| cust1    | ben       |
| cust2    | tony      |
| cust22   | tom       |
| cust221  | an        |
| cust2217 | hex       |

Bảng `Orders` đại diện cho thông tin đơn hàng, có mã đơn hàng `order_num` và id khách hàng `cust_id`.

| order_num | cust_id  |
| --------- | -------- |
| a1        | cust10   |
| a2        | cust1    |
| a3        | cust2    |
| a4        | cust22   |
| a5        | cust221  |
| a7        | cust2217 |

【Bài toán】Dùng `INNER JOIN` để viết câu lệnh SQL, truy xuất tên của từng khách hàng (`cust_name` trong bảng `Customers`) và tất cả mã đơn hàng (`order_num` trong bảng `Orders`), cuối cùng trả về theo thứ tự tăng dần của tên khách hàng `cust_name`.

```sql
SELECT cust_name, order_num
FROM Customers
INNER JOIN Orders
USING(cust_id)
ORDER BY cust_name
```

### Truy xuất tên và tất cả mã đơn hàng của từng khách hàng (2)

Bảng `Orders` đại diện cho thông tin đơn hàng, có mã đơn hàng `order_num` và id khách hàng `cust_id`.

| order_num | cust_id  |
| --------- | -------- |
| a1        | cust10   |
| a2        | cust1    |
| a3        | cust2    |
| a4        | cust22   |
| a5        | cust221  |
| a7        | cust2217 |

Bảng `Customers` đại diện cho thông tin khách hàng, có id khách hàng `cust_id` và tên khách hàng `cust_name`.

| cust_id  | cust_name |
| -------- | --------- |
| cust10   | andy      |
| cust1    | ben       |
| cust2    | tony      |
| cust22   | tom       |
| cust221  | an        |
| cust2217 | hex       |
| cust40   | ace       |

【Bài toán】Truy xuất tên của từng khách hàng (`cust_name` trong bảng `Customers`) và tất cả mã đơn hàng (`order_num` trong bảng `Orders`), liệt kê tất cả khách hàng kể cả khách hàng chưa từng đặt hàng. Cuối cùng trả về theo thứ tự tăng dần của tên khách hàng `cust_name`.

```sql
SELECT cust_name, order_num
FROM Customers
LEFT JOIN Orders
USING(cust_id)
ORDER BY cust_name
```

### Trả về tên sản phẩm và mã đơn hàng liên quan

Bảng `Products` là bảng thông tin sản phẩm, có id sản phẩm `prod_id` và tên sản phẩm `prod_name`.

| prod_id | prod_name |
| ------- | --------- |
| a0001   | egg       |
| a0002   | sockets   |
| a0013   | coffee    |
| a0003   | cola      |
| a0023   | soda      |

Bảng `OrderItems` là bảng thông tin đơn hàng, có mã đơn hàng `order_num` và id sản phẩm `prod_id`.

| prod_id | order_num |
| ------- | --------- |
| a0001   | a105      |
| a0002   | a1100     |
| a0002   | a200      |
| a0013   | a1121     |
| a0003   | a10       |
| a0003   | a19       |
| a0003   | a5        |

【Bài toán】Dùng phép nối ngoài (left join, right join, full join) để nối bảng `Products` và `OrderItems`, trả về danh sách tên sản phẩm (`prod_name`) cùng mã đơn hàng liên quan (`order_num`), rồi sắp xếp tăng dần theo tên sản phẩm.

```sql
SELECT prod_name, order_num
FROM Products
LEFT JOIN OrderItems
USING(prod_id)
ORDER BY prod_name
```

### Trả về tên sản phẩm và tổng số đơn hàng của từng sản phẩm

Bảng `Products` là bảng thông tin sản phẩm, có id sản phẩm `prod_id` và tên sản phẩm `prod_name`.

| prod_id | prod_name |
| ------- | --------- |
| a0001   | egg       |
| a0002   | sockets   |
| a0013   | coffee    |
| a0003   | cola      |
| a0023   | soda      |

Bảng `OrderItems` là bảng thông tin đơn hàng, có mã đơn hàng `order_num` và id sản phẩm `prod_id`.

| prod_id | order_num |
| ------- | --------- |
| a0001   | a105      |
| a0002   | a1100     |
| a0002   | a200      |
| a0013   | a1121     |
| a0003   | a10       |
| a0003   | a19       |
| a0003   | a5        |

【Bài toán】Dùng OUTER JOIN để nối bảng `Products` và `OrderItems`, trả về tên sản phẩm (`prod_name`) và tổng số đơn hàng của từng sản phẩm (không phải mã đơn hàng), sau đó sắp xếp tăng dần theo tên sản phẩm.

```sql
SELECT prod_name, COUNT(order_num) AS orders
FROM Products
LEFT JOIN OrderItems
USING(prod_id)
GROUP BY prod_name
ORDER BY prod_name
```

### Liệt kê nhà cung cấp và số lượng sản phẩm họ cung cấp

Có bảng `Vendors` chứa `vend_id` (id nhà cung cấp).

| vend_id |
| ------- |
| a0002   |
| a0013   |
| a0003   |
| a0010   |

Có bảng `Products` chứa `vend_id` (id nhà cung cấp) và `prod_id` (id sản phẩm được cung cấp).

| vend_id | prod_id              |
| ------- | -------------------- |
| a0001   | egg                  |
| a0002   | prod_id_iphone       |
| a00113  | prod_id_tea          |
| a0003   | prod_id_vivo phone   |
| a0010   | prod_id_huawei phone |

【Bài toán】Liệt kê nhà cung cấp (`vend_id` trong bảng `Vendors`) và số lượng sản phẩm họ cung cấp, bao gồm cả nhà cung cấp không có sản phẩm. Cần dùng OUTER JOIN và aggregate function `COUNT()` để tính số lượng sản phẩm của từng nhà cung cấp trong bảng `Products`, cuối cùng sắp xếp tăng dần theo `vend_id`.

Lưu ý: cột `vend_id` xuất hiện trong nhiều bảng, vì vậy mỗi lần tham chiếu cần ghi đầy đủ định danh.

```sql
SELECT v.vend_id, COUNT(prod_id) AS prod_id
FROM Vendors v
LEFT JOIN Products p
USING(vend_id)
GROUP BY v.vend_id
ORDER BY v.vend_id
```

## Kết hợp query

Toán tử `UNION` kết hợp kết quả của hai hoặc nhiều query và tạo một result set chứa các dòng được truy xuất từ những query tham gia `UNION`.

Quy tắc cơ bản của `UNION`:

- Số lượng và thứ tự cột của tất cả query phải giống nhau.
- Kiểu dữ liệu của các cột thuộc những bảng liên quan trong mỗi query phải giống nhau hoặc tương thích.
- Thông thường, tên cột được trả về lấy từ query đầu tiên.

Mặc định, toán tử `UNION` chọn các giá trị khác nhau. Nếu cho phép giá trị trùng lặp, hãy dùng `UNION ALL`.

```sql
SELECT column_name(s) FROM table1
UNION ALL
SELECT column_name(s) FROM table2;
```

Tên cột trong result set của `UNION` luôn bằng tên cột trong statement `SELECT` đầu tiên của `UNION`.

`JOIN` và `UNION`:

- Các cột của bảng được nối trong `JOIN` có thể khác nhau, nhưng số lượng và thứ tự cột của tất cả query trong `UNION` phải giống nhau.
- `UNION` xếp các dòng của các query theo chiều dọc, còn `JOIN` xếp các cột của các query theo chiều ngang, tức là tạo thành một tích Cartesian.

### Kết hợp hai statement SELECT (1)

Bảng `OrderItems` chứa thông tin sản phẩm đơn hàng, trường `prod_id` đại diện cho id sản phẩm và `quantity` đại diện cho số lượng sản phẩm.

| prod_id | quantity |
| ------- | -------- |
| a0001   | 105      |
| a0002   | 100      |
| a0002   | 200      |
| a0013   | 1121     |
| a0003   | 10       |
| a0003   | 19       |
| a0003   | 5        |
| BNBG    | 10002    |

【Bài toán】Kết hợp hai statement `SELECT` để truy xuất ID sản phẩm (`prod_id`) và `quantity` từ bảng `OrderItems`. Một statement `SELECT` lọc các dòng có số lượng bằng 100, statement còn lại lọc sản phẩm có ID bắt đầu bằng BNBG, cuối cùng sắp xếp kết quả tăng dần theo ID sản phẩm.

```sql
SELECT prod_id, quantity
FROM OrderItems
WHERE quantity = 100
UNION
SELECT prod_id, quantity
FROM OrderItems
WHERE prod_id LIKE 'BNBG%'
ORDER BY prod_id;
```

> **Lưu ý**: Khi dùng `ORDER BY` trong query `UNION`, chỉ được dùng một lần sau statement `SELECT` cuối cùng; nó sẽ sắp xếp toàn bộ result set kết hợp.

### Kết hợp hai statement SELECT (2)

Bảng `OrderItems` chứa thông tin sản phẩm đơn hàng, trường `prod_id` đại diện cho id sản phẩm và `quantity` đại diện cho số lượng sản phẩm.

| prod_id | quantity |
| ------- | -------- |
| a0001   | 105      |
| a0002   | 100      |
| a0002   | 200      |
| a0013   | 1121     |
| a0003   | 10       |
| a0003   | 19       |
| a0003   | 5        |
| BNBG    | 10002    |

【Bài toán】Kết hợp hai statement `SELECT` để truy xuất ID sản phẩm (`prod_id`) và `quantity` từ bảng `OrderItems`. Một statement `SELECT` lọc các dòng có số lượng bằng 100, statement còn lại lọc sản phẩm có ID bắt đầu bằng BNBG, cuối cùng sắp xếp kết quả tăng dần theo ID sản phẩm. Lưu ý: **lần này chỉ dùng một statement SELECT.**

Đáp án:

Chỉ dùng một statement `SELECT` thì sử dụng `OR` thay cho `UNION`.

```sql
SELECT prod_id, quantity
FROM OrderItems
WHERE quantity = 100 OR prod_id LIKE 'BNBG%'
ORDER BY prod_id;
```

### Kết hợp tên sản phẩm trong bảng Products và tên khách hàng trong bảng Customers

Bảng `Products` có trường `prod_name` đại diện cho tên sản phẩm.

| prod_name |
| --------- |
| flower    |
| rice      |
| ring      |
| umbrella  |

Bảng Customers đại diện cho thông tin khách hàng, `cust_name` đại diện cho tên khách hàng.

| cust_name |
| --------- |
| andy      |
| ben       |
| tony      |
| tom       |
| an        |
| lee       |
| hex       |

【Bài toán】Viết câu lệnh SQL để kết hợp và trả về tên sản phẩm (`prod_name`) trong bảng `Products` cùng tên khách hàng (`cust_name`) trong bảng `Customers`, sau đó sắp xếp kết quả tăng dần theo tên sản phẩm.

```sql
# Tên cột trong result set của UNION luôn bằng tên cột trong statement SELECT đầu tiên của UNION.
SELECT prod_name
FROM Products
UNION
SELECT cust_name
FROM Customers
ORDER BY prod_name
```

### Kiểm tra câu lệnh SQL

Bảng `Customers` có các trường tên khách hàng `cust_name`, thông tin liên hệ khách hàng `cust_contact`, bang của khách hàng `cust_state` và email khách hàng `cust_email`.

| cust_name | cust_contact | cust_state | cust_email        |
| --------- | ------------ | ---------- | ----------------- |
| cust10    | 8695192      | MI         | <cust10@cust.com> |
| cust1     | 8695193      | MI         | <cust1@cust.com>  |
| cust2     | 8695194      | IL         | <cust2@cust.com>  |

【Bài toán】Sửa câu SQL sai dưới đây.

```sql
SELECT cust_name, cust_contact, cust_email
FROM Customers
WHERE cust_state = 'MI'
ORDER BY cust_name;
UNION
SELECT cust_name, cust_contact, cust_email
FROM Customers
WHERE cust_state = 'IL'ORDER BY cust_name;
```

Sau khi sửa:

```sql
SELECT cust_name, cust_contact, cust_email
FROM Customers
WHERE cust_state = 'MI'
UNION
SELECT cust_name, cust_contact, cust_email
FROM Customers
WHERE cust_state = 'IL'
ORDER BY cust_name;
```

Khi dùng `UNION` để kết hợp các query, chỉ được dùng một mệnh đề `ORDER BY`, và nó phải nằm sau statement `SELECT` cuối cùng.

Hoặc có thể dùng `or` trực tiếp:

```sql
SELECT cust_name, cust_contact, cust_email
FROM Customers
WHERE cust_state = 'MI' or cust_state = 'IL'
ORDER BY cust_name;
```

<!-- @include: @article-footer.snippet.md -->
