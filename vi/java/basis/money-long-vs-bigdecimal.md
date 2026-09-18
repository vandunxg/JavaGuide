---
title: "Nên dùng long hay BigDecimal cho tiền trong Java?"
description: "Hướng dẫn chọn kiểu dữ liệu tiền trong Java: giải thích các trường hợp phù hợp để dùng long lưu đơn vị tiền tệ nhỏ nhất và BigDecimal tính toán chính xác, cùng với việc làm tròn, tràn số, chuyển đổi đơn vị và thiết kế field trong database."
category: Java
tag:
  - Java Basics
  - Java money calculation
head:
  - - meta
    - name: keywords
      content: Java money type,long storing money,Long storing cents,BigDecimal money calculation,money precision,money rounding,DECIMAL,BIGINT
---

Trong phần bình luận của một bài viết thảo luận về kiểu dữ liệu của field tiền, tôi thấy vài câu trả lời hoàn toàn khác nhau: có người nhất quyết dùng `Long` lưu cents, có người nói các trường hợp như lãi suất, tỷ giá phải dùng `BigDecimal`, cũng có người đề cập việc truyền trực tiếp string trong interface.

Những ý kiến này không bàn về cùng một việc. Dùng `Long` lưu cents là nói về cách lưu tiền; dùng `BigDecimal` cho lãi suất và tỷ giá là nói về cách tính tiền; truyền string qua interface thường chỉ là format truyền dữ liệu.

Tiền đã xác định đơn vị nhỏ nhất có thể được lưu bằng `long`. Nếu trong quá trình tính toán cần giữ phần thập phân hoặc cần chỉ định rõ cách làm tròn, hãy dùng `BigDecimal`. Hai kiểu này có thể cùng xuất hiện trong một hệ thống.

Phương án Long được đề cập dưới đây đều chỉ việc dùng số nguyên để lưu đơn vị tiền tệ nhỏ nhất. Khi code Java thực hiện phép tính, thông thường dùng primitive type `long`; chỉ dùng wrapper type `Long` khi cần biểu diễn giá trị null.

| Hạng mục so sánh       | `long`                                                     | `BigDecimal`                                       |
| ---------------------- | ---------------------------------------------------------- | -------------------------------------------------- |
| Cách biểu diễn         | Số nguyên với đơn vị nhỏ nhất cố định                      | Số thập phân có `scale`                            |
| Cách dùng phổ biến     | Tiền order, số dư, tiền ghi sổ đã xác định đơn vị nhỏ nhất | Tính discount, tax, interest, exchange rate        |
| Rủi ro chính           | Nhầm đơn vị, silent overflow, khó mở rộng precision        | Cách khởi tạo, quy tắc rounding, khác biệt `scale` |
| Kiểu database phổ biến | `BIGINT`                                                   | `DECIMAL(p, s)`                                    |

## Vì sao tiền không thể dùng double?

`double` và `float` lưu binary floating-point number. Nhiều số thập phân hữu hạn khi đổi sang binary sẽ trở thành phân số nhị phân lặp vô hạn, chỉ có thể lấy giá trị gần nhất có thể biểu diễn.

```java
double a = 1.0;
double b = 0.9;

System.out.println(a - b);
// 0.09999999999999998

System.out.println(0.1 + 0.1 + 0.1);
// 0.30000000000000004
```

Sai số này không liên quan đến implementation của Java; mọi ngôn ngữ sử dụng binary floating-point number theo IEEE 754 đều gặp phải. Tính tiền thường yêu cầu kết quả tuân theo precision thập phân và quy tắc rounding rõ ràng, nên giá trị xấp xỉ khó đáp ứng yêu cầu này.

`new BigDecimal(0.1)` có thể hiển thị đầy đủ giá trị xấp xỉ được lưu trong `double`:

```java
System.out.println(new BigDecimal(0.1));
// 0.1000000000000000055511151231257827021181583404541015625
```

Đây cũng là lý do không nên khởi tạo money object từ `double`. Một giá trị đã phát sinh sai số sẽ không tự khôi phục thành số thập phân ban đầu khi được chuyển sang `BigDecimal`.

## Khoản tiền nào phù hợp với `long`?

Nếu business quy định tiền CNY luôn chính xác đến cents, `19.99` CNY có thể được lưu thành `1999` cents. Các phép cộng trừ đều thực hiện trên số nguyên nên không phát sinh sai số thập phân.

```java
long priceCents = 1_999L;
long shippingCents = 500L;
long totalCents = Math.addExact(priceCents, shippingCents);
```

`long` phù hợp với các giá trị đã rounding như order amount, account balance và payment amount; trong database có thể dùng `BIGINT`.

Khi lưu trong database, tên field tốt nhất nên chứa unit để dễ nhìn và trực quan hơn:

```sql
CREATE TABLE orders (
    id           BIGINT PRIMARY KEY,
    amount_cents BIGINT NOT NULL
);
```

Nếu dùng `amount`, chỉ nhìn vào giá trị thì không thể biết `amount = 100` nghĩa là 100 CNY hay 100 cents. Đổi thành `amount_cents = 100` thì khác.

**Cần lưu ý gì khi dùng long?**

Lưu thống nhất tiền theo cents cũng cố định precision ở hai chữ số thập phân. Kết quả trung gian của exchange rate, interest, tax hoặc tính phí theo lượng có thể cần bốn, sáu hoặc nhiều chữ số thập phân hơn; những phép tính này không thể tiếp tục tính bằng “cents” một cách gượng ép.

Cũng cần ngăn overflow. `+` và `*` thông thường không báo lỗi sau khi overflow; code tiền có thể dùng `Math.addExact()`, `Math.subtractExact()` và `Math.multiplyExact()`:

```java
long subtotalCents = Math.multiplyExact(unitPriceCents, quantity);
long balanceCents = Math.subtractExact(currentBalanceCents, paymentCents);
```

Phép nhân còn phải kiểm tra kết quả trung gian. Việc amount cuối cùng không vượt quá `Long.MAX_VALUE` không có nghĩa là từng bước trong `unit price × quantity × multiplier` cũng sẽ không overflow.

Hệ thống multi-currency cũng không thể giả định mọi currency đều có hai chữ số thập phân. Khoản tiền ít nhất phải đi kèm currency; số chữ số của đơn vị nhỏ nhất do currency hoặc business rule quyết định, không thể suy ra từ một giá trị `long` riêng lẻ.

## Loại tiền nào phù hợp với BigDecimal?

Discount, tax, interest và quy đổi exchange rate thường tạo ra kết quả trung gian vượt quá đơn vị tiền tệ nhỏ nhất.

`BigDecimal` biểu diễn số thập phân bằng số nguyên có độ chính xác tùy ý và `scale`, có thể giữ lại các giá trị trung gian này rồi rounding tại vị trí do business quy định.

```java
BigDecimal price = new BigDecimal("19.99");
BigDecimal discountRate = new BigDecimal("0.95");

BigDecimal discountedPrice = price.multiply(discountRate);
// 18.9905
```

Hằng số tiền tệ nên được khởi tạo trực tiếp bằng string. Nếu interface truyền đến string thì chuyển thẳng thành `BigDecimal`; nếu field trong database là `DECIMAL` thì mapping thẳng thành `BigDecimal`, không cần chuyển qua `double` ở giữa.

**Nếu `divide()` không chia hết thì phải làm sao?**

Khi đó cần chỉ định số chữ số giữ lại và cách rounding.

Đoạn code dưới đây giữ lại hai chữ số thập phân và dùng `HALF_UP` (làm tròn nửa lên). Nếu gọi trực tiếp `a.divide(b)`, chương trình sẽ throw `ArithmeticException`.

```java
BigDecimal a = new BigDecimal("10");
BigDecimal b = new BigDecimal("3");

System.out.println(a.divide(b, 2, RoundingMode.HALF_UP)); // 3.33
```

Một điểm khác cần lưu ý: `BigDecimal` là immutable class, kết quả phép tính phải được nhận bằng variable mới hoặc gán lại. Lần đầu gọi `add()` mà không nhận giá trị trả về thì `amount` vẫn là `10.00`:

```java
BigDecimal amount = new BigDecimal("10.00");

amount.add(new BigDecimal("2.00"));
System.out.println(amount); // vẫn là 10.00

amount = amount.add(new BigDecimal("2.00"));
System.out.println(amount); // 12.00
```

Để so sánh giá trị tiền, thường dùng `compareTo()`. `equals()` còn so sánh cả `scale`, vì vậy kết quả gọi `equals()` trên `1.0` và `1.00` là `false`:

```java
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("1.00");

System.out.println(a.equals(b));         // false
System.out.println(a.compareTo(b) == 0); // true
```

Khác biệt này cũng ảnh hưởng đến `HashMap` và `HashSet`. Nếu dùng `BigDecimal` làm key, tốt nhất nên thống nhất `scale` trước; nếu không, `1.0` và `1.00` sẽ bị xem là hai key khác nhau.

## Long và BigDecimal có thể dùng cùng nhau không?

Có thể. Lấy unit price `19.99` CNY, mua 3 sản phẩm và discount `0.95` làm ví dụ: dùng `BigDecimal` khi tính toán, sau đó chuyển payment amount thành `long` theo đơn vị cents:

```java
BigDecimal unitPrice = new BigDecimal("19.99");
BigDecimal discountRate = new BigDecimal("0.95");
long quantity = 3L;

BigDecimal payable = unitPrice
        .multiply(BigDecimal.valueOf(quantity))
        .multiply(discountRate)
        .setScale(2, RoundingMode.HALF_UP);

long payableCents = payable
        .movePointRight(2)
        .longValueExact();
```

Đoạn code này tính ra `payable` là `56.97`, `movePointRight(2)` chuyển nó thành `5697`. `longValueExact()` chỉ nhận integer trong phạm vi `long`; chỉ cần còn phần thập phân khác không hoặc giá trị vượt phạm vi là sẽ throw `ArithmeticException`.

Khi chuyển đổi money, không dùng trực tiếp `longValue()` vì nó sẽ bỏ phần thập phân:

```java
long amount = new BigDecimal("19.99").longValue();
System.out.println(amount); // 19
```

Nếu input amount nhiều nhất chỉ có hai chữ số thập phân, cũng có thể dùng `RoundingMode.UNNECESSARY` để kiểm tra:

```java
public static long toCentsExact(BigDecimal amount) {
    return amount
            .setScale(2, RoundingMode.UNNECESSARY)
            .movePointRight(2)
            .longValueExact();
}

public static BigDecimal fromCents(long cents) {
    return BigDecimal.valueOf(cents, 2);
}
```

`19.9` có thể bổ sung thành `19.90`, còn `19.999` sẽ trực tiếp throw exception, không âm thầm truncate hoặc rounding.

Nếu payment interface nhận cents thì truyền `5697`; nếu nhận string theo đơn vị CNY thì truyền `payable.toPlainString()`. Đặt code chuyển đổi ở interface adapter layer; trong quá trình business calculation không nên chuyển đổi qua lại giữa các kiểu và unit.

## Database nên dùng BIGINT hay DECIMAL?

Trong Java, nếu dùng `long` lưu đơn vị nhỏ nhất thì field trong database thường dùng `BIGINT`; nếu Java dùng `BigDecimal` thì field trong database thường dùng `DECIMAL(p, s)`.

```sql
CREATE TABLE settlement_detail (
    id               BIGINT PRIMARY KEY,
    payable_cents    BIGINT        NOT NULL,
    exchange_rate    DECIMAL(18, 8) NOT NULL,
    settlement_amount DECIMAL(18, 2) NOT NULL
);
```

MySQL xếp integer và `DECIMAL` vào nhóm exact-value type. Trong `DECIMAL(18, 2)`, `18` là tổng số chữ số có nghĩa, còn `2` là số chữ số thập phân; nó có thể bao phủ amount của business hay không phải được suy ra từ giá trị lớn nhất, không thể cứ thấy money field là dùng chung một precision.

Không nên phụ thuộc vào việc MySQL tự rounding khi ghi `DECIMAL`. Code Java trước tiên gọi `setScale()` để rounding hoặc validation, sau đó mới ghi kết quả vào database; như vậy giá trị lưu trong database mới khớp với kết quả tính trong chương trình.

Money field có dùng `DEFAULT 0` hay không phụ thuộc vào business meaning. Amount bị thiếu và amount bằng zero không phải lúc nào cũng giống nhau; tùy tiện thêm default value có thể che giấu dữ liệu bị truyền thiếu. `NOT NULL` thường nên được giữ lại, còn default value nên do domain rule quyết định.

## Tổng kết

Tiền đã rounding xong và cố định đơn vị nhỏ nhất phù hợp để lưu bằng `long`; các phép tính như discount, tax, interest và exchange rate cần giữ phần thập phân thì dùng `BigDecimal`. Khi rounding, phải ghi rõ số chữ số giữ lại và `RoundingMode`.

Hai kiểu có thể dùng cùng nhau. Ở giai đoạn tính toán, giữ `BigDecimal`; sau khi xác định payment amount cuối cùng, trước tiên dịch dấu thập phân theo số chữ số của đơn vị nhỏ nhất, rồi dùng `longValueExact()` chuyển thành số nguyên. Tên field phải ghi rõ unit, phép tính integer phải kiểm tra overflow, còn việc chuyển đổi type và unit nên tập trung trong code adapter của interface hoặc database.
