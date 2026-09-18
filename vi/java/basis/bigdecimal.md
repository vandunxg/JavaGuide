---
title: Giải thích chi tiết về BigDecimal
description: "Giải thích chi tiết cách sử dụng BigDecimal: giải quyết vấn đề mất độ chính xác của số floating-point, nắm vững phép cộng trừ nhân chia, quy tắc làm tròn của RoundingMode, phương thức so sánh compareTo, phù hợp với các trường hợp cần tính toán độ chính xác cao như tính toán tài chính."
category: Java
tag:
  - Java Basics
head:
  - - meta
    - name: keywords
      content: BigDecimal,độ chính xác số floating-point,phép tính số thập phân,chế độ làm tròn RoundingMode,so sánh BigDecimal,tính toán số tiền,mất độ chính xác
---

《Sổ tay phát triển Java của Alibaba》 có đề cập: “Để tránh mất độ chính xác, có thể sử dụng `BigDecimal` để thực hiện phép tính số floating-point”.

Phép tính số floating-point cũng có nguy cơ mất độ chính xác sao? Đúng vậy!

Ví dụ code:

```java
float a = 2.0f - 1.9f;
float b = 1.8f - 1.7f;
System.out.println(a);// 0.100000024
System.out.println(b);// 0.099999905
System.out.println(a == b);// false
```

**Tại sao khi tính toán với số floating-point `float` hoặc `double` lại có nguy cơ mất độ chính xác?**

Điều này liên quan rất lớn đến cơ chế máy tính lưu số. Như bạn biết, máy tính sử dụng hệ nhị phân, đồng thời số bit dùng để biểu diễn một số là hữu hạn. Nhiều số thập phân khi chuyển sang nhị phân sẽ lặp vô hạn, chỉ có thể làm tròn về một số chữ số hữu hạn, vì vậy tồn tại nguy cơ mất độ chính xác. Tuy nhiên, những giá trị như 0.5 và 0.25 có thể biểu diễn bằng số nhị phân hữu hạn nên vẫn được biểu diễn chính xác.

Ví dụ, 0.2 trong hệ thập phân không thể chuyển chính xác thành số nhị phân:

```java
// Quá trình chuyển 0.2 sang số nhị phân là liên tục nhân với 2 cho đến khi không còn phần thập phân,
// trong quá trình này, các phần nguyên thu được khi xếp từ trên xuống chính là kết quả nhị phân.
0.2 * 2 = 0.4 -> 0
0.4 * 2 = 0.8 -> 0
0.8 * 2 = 1.6 -> 1
0.6 * 2 = 1.2 -> 1
0.2 * 2 = 0.4 -> 0 (xảy ra lặp)
...
```

Để tìm hiểu thêm về số floating-point, bạn nên xem bài viết [Cơ sở hệ thống máy tính (4): Số floating-point](http://kaito-kidd.com/2018/08/08/computer-system-float-point/).

## Giới thiệu về BigDecimal

`BigDecimal` có thể biểu diễn chính xác số thập phân và cung cấp các phép tính cho phép chỉ định rõ độ chính xác và quy tắc làm tròn. Tuy nhiên, khi sử dụng `MathContext` có độ chính xác hữu hạn, thực hiện phép chia cần làm tròn, hoặc chuyển kết quả sang `float`, `double`, việc làm tròn vẫn có thể xảy ra.

Thông thường, hầu hết các trường hợp nghiệp vụ cần kết quả tính toán số thập phân chính xác (chẳng hạn các trường hợp liên quan đến tiền) đều sử dụng `BigDecimal`.

《Sổ tay phát triển Java của Alibaba》 có đề cập: **Khi kiểm tra giá trị bằng nhau giữa các số floating-point, kiểu dữ liệu nguyên thủy không được dùng `==` để so sánh, wrapper type không được dùng `equals` để so sánh.**

![](https://oss.javaguide.cn/javaguide/image-20211213101646884.png)

Nguyên nhân cụ thể đã được giới thiệu chi tiết ở trên nên không nhắc lại ở đây.

Để giải quyết vấn đề mất độ chính xác khi tính toán số floating-point, bạn có thể trực tiếp sử dụng `BigDecimal` để định nghĩa giá trị số thập phân, sau đó thực hiện phép tính số thập phân.

```java
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
BigDecimal c = new BigDecimal("0.8");

BigDecimal x = a.subtract(b);
BigDecimal y = b.subtract(c);

System.out.println(x.compareTo(y));// 0
```

## Các phương thức BigDecimal thường gặp

### Khởi tạo

Khi sử dụng `BigDecimal`, để tránh mất độ chính xác, nên sử dụng constructor `BigDecimal(String val)` hoặc static method `BigDecimal.valueOf(double val)` để tạo object.

《Sổ tay phát triển Java của Alibaba》 cũng đề cập đến nội dung này, như hình dưới đây.

![](https://oss.javaguide.cn/javaguide/image-20211213102222601.png)

### Cộng trừ nhân chia

Phương thức `add` dùng để cộng hai object `BigDecimal`, phương thức `subtract` dùng để trừ hai object `BigDecimal`. Phương thức `multiply` dùng để nhân hai object `BigDecimal`, phương thức `divide` dùng để chia hai object `BigDecimal`.

```java
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
System.out.println(a.add(b));// 1.9
System.out.println(a.subtract(b));// 0.1
System.out.println(a.multiply(b));// 0.90
System.out.println(a.divide(b));// Không thể chia hết, ném ArithmeticException
System.out.println(a.divide(b, 2, RoundingMode.HALF_UP));// 1.11
```

Cần lưu ý rằng phải chọn overload của `divide` tùy theo nghiệp vụ có cho phép làm tròn hay không. Khi yêu cầu kết quả chính xác, có thể sử dụng phiên bản không chỉ định quy tắc làm tròn; khi kết quả không thể biểu diễn chính xác, `ArithmeticException` sẽ được ném ra. Khi cho phép làm tròn, cần chỉ định rõ `scale` và `roundingMode`. `RoundingMode.UNNECESSARY` dùng để assertion rằng kết quả không cần làm tròn; nếu thực tế cần làm tròn thì cũng sẽ ném `ArithmeticException`.

```java
public BigDecimal divide(BigDecimal divisor, int scale, RoundingMode roundingMode) {
    return divide(divisor, scale, roundingMode.oldMode);
}
```

Có khá nhiều quy tắc làm tròn, dưới đây liệt kê một số quy tắc:

```java
public enum RoundingMode {
   // 2.4 -> 3 , 1.6 -> 2
   // -1.6 -> -2 , -2.4 -> -3
   UP(BigDecimal.ROUND_UP),
   // 2.4 -> 2 , 1.6 -> 1
   // -1.6 -> -1 , -2.4 -> -2
   DOWN(BigDecimal.ROUND_DOWN),
   // 2.4 -> 3 , 1.6 -> 2
   // -1.6 -> -1 , -2.4 -> -2
   CEILING(BigDecimal.ROUND_CEILING),
   // 2.5 -> 2 , 1.6 -> 1
   // -1.6 -> -2 , -2.5 -> -3
   FLOOR(BigDecimal.ROUND_FLOOR),
   // 2.4 -> 2 , 1.6 -> 2
   // -1.6 -> -2 , -2.4 -> -2
   HALF_UP(BigDecimal.ROUND_HALF_UP),
   //......
}
```

### So sánh lớn, nhỏ

`a.compareTo(b)`: trả về -1 nếu `a` nhỏ hơn `b`, 0 nếu `a` bằng `b`, 1 nếu `a` lớn hơn `b`.

```java
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
System.out.println(a.compareTo(b));// 1
```

### Giữ lại bao nhiêu chữ số thập phân

Sử dụng phương thức `setScale` để thiết lập số chữ số sau dấu thập phân cần giữ lại và quy tắc làm tròn. Có khá nhiều quy tắc làm tròn, không cần ghi nhớ; IDEA sẽ gợi ý.

```java
BigDecimal m = new BigDecimal("1.255433");
BigDecimal n = m.setScale(3,RoundingMode.HALF_DOWN);
System.out.println(n);// 1.255
```

## Vấn đề so sánh giá trị bằng nhau của BigDecimal

《Sổ tay phát triển Java của Alibaba》 có đề cập:

![](https://oss.javaguide.cn/github/javaguide/java/basis/image-20220714161315993.png)

Ví dụ code cho thấy vấn đề khi `BigDecimal` sử dụng phương thức `equals()` để so sánh giá trị bằng nhau:

```java
BigDecimal a = new BigDecimal("1");
BigDecimal b = new BigDecimal("1.0");
System.out.println(a.equals(b));//false
```

Đó là vì phương thức `equals()` không chỉ so sánh giá trị (value) mà còn so sánh `scale`, trong khi phương thức `compareTo()` sẽ bỏ qua `scale` khi so sánh.

`scale` của 1.0 là 1, `scale` của 1 là 0, vì vậy kết quả của `a.equals(b)` là false.

![](https://oss.javaguide.cn/github/javaguide/java/basis/image-20220714164706390.png)

Phương thức `compareTo()` có thể so sánh giá trị của hai `BigDecimal`; nếu bằng nhau thì trả về 0, nếu số thứ nhất lớn hơn số thứ hai thì trả về 1, ngược lại trả về -1.

```java
BigDecimal a = new BigDecimal("1");
BigDecimal b = new BigDecimal("1.0");
System.out.println(a.compareTo(b));//0
```

## Chia sẻ utility class BigDecimal

Trên Internet có một utility class thao tác với `BigDecimal` được khá nhiều người sử dụng, cung cấp nhiều static method để đơn giản hóa thao tác với `BigDecimal`.

Tôi đã cải tiến một chút utility class này và chia sẻ source code:

```java
import java.math.BigDecimal;
import java.math.RoundingMode;

/**
 * Utility class nhỏ giúp đơn giản hóa phép tính BigDecimal
 */
public class BigDecimalUtil {

    /**
     * Độ chính xác mặc định của phép chia
     */
    private static final int DEF_DIV_SCALE = 10;

    private BigDecimalUtil() {
    }

    /**
     * Thực hiện phép cộng bằng BigDecimal; khi chuyển kết quả thành double vẫn có thể xảy ra làm tròn.
     *
     * @param v1 số bị cộng
     * @param v2 số cộng
     * @return tổng của hai tham số
     */
    public static double add(double v1, double v2) {
        BigDecimal b1 = BigDecimal.valueOf(v1);
        BigDecimal b2 = BigDecimal.valueOf(v2);
        return b1.add(b2).doubleValue();
    }

    /**
     * Thực hiện phép trừ bằng BigDecimal; khi chuyển kết quả thành double vẫn có thể xảy ra làm tròn.
     *
     * @param v1 số bị trừ
     * @param v2 số trừ
     * @return hiệu của hai tham số
     */
    public static double subtract(double v1, double v2) {
        BigDecimal b1 = BigDecimal.valueOf(v1);
        BigDecimal b2 = BigDecimal.valueOf(v2);
        return b1.subtract(b2).doubleValue();
    }

    /**
     * Thực hiện phép nhân bằng BigDecimal; khi chuyển kết quả thành double vẫn có thể xảy ra làm tròn.
     *
     * @param v1 số bị nhân
     * @param v2 số nhân
     * @return tích của hai tham số
     */
    public static double multiply(double v1, double v2) {
        BigDecimal b1 = BigDecimal.valueOf(v1);
        BigDecimal b2 = BigDecimal.valueOf(v2);
        return b1.multiply(b2).doubleValue();
    }

    /**
     * Cung cấp phép chia tương đối chính xác; khi không chia hết, giữ chính xác đến
     * 10 chữ số sau dấu thập phân, các chữ số sau đó được làm tròn theo HALF_EVEN.
     *
     * @param v1 số bị chia
     * @param v2 số chia
     * @return thương của hai tham số
     */
    public static double divide(double v1, double v2) {
        return divide(v1, v2, DEF_DIV_SCALE);
    }

    /**
     * Cung cấp phép chia tương đối chính xác. Khi không chia hết, số chữ số được chỉ định
     * bởi tham số scale, các chữ số sau đó được làm tròn theo HALF_EVEN.
     *
     * @param v1    số bị chia
     * @param v2    số chia
     * @param scale cho biết cần giữ bao nhiêu chữ số sau dấu thập phân.
     * @return thương của hai tham số
     */
    public static double divide(double v1, double v2, int scale) {
        if (scale < 0) {
            throw new IllegalArgumentException(
                    "The scale must be a positive integer or zero");
        }
        BigDecimal b1 = BigDecimal.valueOf(v1);
        BigDecimal b2 = BigDecimal.valueOf(v2);
        return b1.divide(b2, scale, RoundingMode.HALF_EVEN).doubleValue();
    }

    /**
     * Làm tròn đến số chữ số sau dấu thập phân được chỉ định theo quy tắc HALF_EVEN.
     *
     * @param v     số cần làm tròn HALF_EVEN
     * @param scale cần giữ lại bao nhiêu chữ số sau dấu thập phân
     * @return kết quả sau khi làm tròn HALF_EVEN
     */
    public static double round(double v, int scale) {
        if (scale < 0) {
            throw new IllegalArgumentException(
                    "The scale must be a positive integer or zero");
        }
        BigDecimal b = BigDecimal.valueOf(v);
        BigDecimal one = new BigDecimal("1");
        return b.divide(one, scale, RoundingMode.HALF_EVEN).doubleValue();
    }

    /**
     * Chuyển đổi thành float; khi vượt quá độ chính xác hoặc phạm vi của float có thể xảy ra làm tròn hoặc tràn số
     *
     * @param v số cần chuyển đổi
     * @return kết quả chuyển đổi
     */
    public static float convertToFloat(double v) {
        BigDecimal b = BigDecimal.valueOf(v);
        return b.floatValue();
    }

    /**
     * Chuyển đổi thành int, không làm tròn; phần thập phân sẽ bị cắt, khi vượt quá phạm vi sẽ mất các bit cao
     *
     * @param v số cần chuyển đổi
     * @return kết quả chuyển đổi
     */
    public static int convertsToInt(double v) {
        BigDecimal b = BigDecimal.valueOf(v);
        return b.intValue();
    }

    /**
     * Chuyển đổi thành long, không làm tròn; phần thập phân sẽ bị cắt, khi vượt quá phạm vi sẽ mất các bit cao
     *
     * @param v số cần chuyển đổi
     * @return kết quả chuyển đổi
     */
    public static long convertsToLong(double v) {
        BigDecimal b = BigDecimal.valueOf(v);
        return b.longValue();
    }

    /**
     * Trả về giá trị lớn hơn trong hai số
     *
     * @param v1 số thứ nhất cần so sánh
     * @param v2 số thứ hai cần so sánh
     * @return giá trị lớn hơn trong hai số
     */
    public static double returnMax(double v1, double v2) {
        BigDecimal b1 = BigDecimal.valueOf(v1);
        BigDecimal b2 = BigDecimal.valueOf(v2);
        return b1.max(b2).doubleValue();
    }

    /**
     * Trả về giá trị nhỏ hơn trong hai số
     *
     * @param v1 số thứ nhất cần so sánh
     * @param v2 số thứ hai cần so sánh
     * @return giá trị nhỏ hơn trong hai số
     */
    public static double returnMin(double v1, double v2) {
        BigDecimal b1 = BigDecimal.valueOf(v1);
        BigDecimal b2 = BigDecimal.valueOf(v2);
        return b1.min(b2).doubleValue();
    }

    /**
     * So sánh chính xác hai số
     *
     * @param v1 số thứ nhất cần so sánh
     * @param v2 số thứ hai cần so sánh
     * @return trả về 0 nếu hai số bằng nhau, 1 nếu số thứ nhất lớn hơn số thứ hai, ngược lại trả về -1
     */
    public static int compareTo(double v1, double v2) {
        BigDecimal b1 = BigDecimal.valueOf(v1);
        BigDecimal b2 = BigDecimal.valueOf(v2);
        return b1.compareTo(b2);
    }

}
```

Issue liên quan: [Đề xuất đặt quy tắc làm tròn thành RoundingMode.HALF_EVEN, tức là làm tròn theo HALF_EVEN,#2129](https://github.com/Snailclimb/JavaGuide/issues/2129).

![RoundingMode.HALF_EVEN](https://oss.javaguide.cn/github/javaguide/java/basis/RoundingMode.HALF_EVEN.png)

## Tổng kết

Nhiều số thập phân không thể được biểu diễn chính xác bằng số nhị phân hữu hạn, vì vậy khi tính toán với `float` hoặc `double` tồn tại nguy cơ mất độ chính xác.

Tuy nhiên, Java cung cấp `BigDecimal` để thao tác với số floating-point. Cách triển khai của `BigDecimal` sử dụng `BigInteger` (dùng để thao tác với số nguyên lớn); điểm khác biệt là `BigDecimal` bổ sung khái niệm `scale`.

<!-- @include: @article-footer.snippet.md -->
