---
title: Giải thích chi tiết 3 kiểu dữ liệu đặc biệt của Redis
description: Giải thích chi tiết cách sử dụng và trường hợp áp dụng của ba kiểu dữ liệu đặc biệt Bitmap, HyperLogLog, GEO của Redis, bao gồm các trường hợp điển hình như thống kê check-in, thống kê UV và người ở gần.
category: Database
tag:
  - Redis
head:
  - - meta
    - name: keywords
      content: Redis kiểu dữ liệu đặc biệt,Bitmap,HyperLogLog,GEO,Bitmap,thống kê cardinality,vị trí địa lý,thống kê check-in,thống kê UV
---

Ngoài 5 kiểu dữ liệu cơ bản, Redis còn hỗ trợ 3 kiểu dữ liệu đặc biệt: Bitmap, HyperLogLog và GEO.

## Bitmap (Bitmap)

### Giới thiệu

Theo giới thiệu trên trang web chính thức:

> Bitmaps are not an actual data type, but a set of bit-oriented operations defined on the String type which is treated like a bit vector. Since strings are binary safe blobs and their maximum length is 512 MB, they are suitable to set up to 2^32 different bits.
>
> Bitmap không phải là một kiểu dữ liệu thực tế trong Redis, mà là một tập hợp các thao tác hướng bit được định nghĩa trên kiểu String và được xem như một vector bit. Vì string là các blob an toàn nhị phân, có độ dài tối đa 512 MB, chúng phù hợp để thiết lập tối đa 2^32 bit khác nhau.

Bitmap lưu trữ các số nhị phân liên tiếp (0 và 1). Với Bitmap, chỉ cần một bit để biểu diễn giá trị hoặc trạng thái tương ứng của một phần tử, còn key chính là phần tử đó. 8 bit có thể tạo thành một byte, vì vậy bản thân Bitmap tiết kiệm đáng kể không gian lưu trữ.

Bạn có thể xem Bitmap như một array lưu trữ các số nhị phân (0 và 1), chỉ số của mỗi phần tử trong array được gọi là offset.

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220720194154133.png)

### Lệnh thường dùng

| Lệnh                                  | Giới thiệu                                                              |
| ------------------------------------- | ----------------------------------------------------------------------- |
| SETBIT key offset value               | Thiết lập giá trị tại vị trí offset được chỉ định                       |
| GETBIT key offset                     | Lấy giá trị tại vị trí offset được chỉ định                             |
| BITCOUNT key start end                | Lấy số phần tử có giá trị bằng 1 giữa start và end                      |
| BITOP operation destkey key1 key2 ... | Thực hiện phép tính trên một hoặc nhiều Bitmap, gồm AND, OR, XOR và NOT |

**Minh họa thao tác cơ bản với Bitmap**:

```bash
# SETBIT trả về giá trị của bit trước đó (mặc định là 0), ở đây sẽ tạo 7 bit
> SETBIT mykey 7 1
(integer) 0
> SETBIT mykey 7 0
(integer) 1
> GETBIT mykey 7
(integer) 0
> SETBIT mykey 6 1
(integer) 0
> SETBIT mykey 8 1
(integer) 0
# Dùng bitcount để thống kê số bit đã được đặt thành 1.
> BITCOUNT mykey
(integer) 2
```

### Trường hợp áp dụng

**Các trường hợp cần lưu thông tin trạng thái (0/1 là đủ để biểu diễn)**

- Ví dụ: tình trạng check-in của người dùng, tình trạng người dùng hoạt động, thống kê hành vi người dùng (chẳng hạn đã thích một video nào đó hay chưa).
- Lệnh liên quan: `SETBIT`, `GETBIT`, `BITCOUNT`, `BITOP`.

## HyperLogLog (thống kê cardinality)

### Giới thiệu

HyperLogLog là một thuật toán xác suất đếm cardinality nổi tiếng, được tối ưu và cải tiến từ LogLog Counting (LLC), không phải tính năng riêng của Redis. Redis chỉ triển khai thuật toán này và cung cấp một số API có thể dùng ngay.

HyperLogLog do Redis cung cấp chiếm không gian cực kỳ nhỏ, chỉ cần 12k không gian là có thể lưu gần `2^64` phần tử khác nhau. Điều này thực sự ấn tượng, đây chính là sức hấp dẫn của toán học! Ngoài ra, Redis đã tối ưu cấu trúc lưu trữ của HyperLogLog và sử dụng hai cách đếm:

- **Ma trận thưa**: chiếm rất ít không gian khi số lượng phần tử được đếm còn nhỏ.
- **Ma trận dày**: chiếm 12k không gian khi số lượng phần tử được đếm đạt đến một ngưỡng nhất định.

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220721091424563.png)

Để tiết kiệm bộ nhớ, thuật toán xác suất đếm cardinality không lưu trực tiếp metadata, mà ước tính giá trị cardinality (số phần tử trong tập hợp) thông qua một phương pháp thống kê xác suất nhất định. Vì vậy, kết quả đếm của HyperLogLog không phải là giá trị chính xác và có sai số nhất định (sai số chuẩn là `0.81%`).

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220720194154133.png)

Cách sử dụng HyperLogLog rất đơn giản, nhưng nguyên lý lại rất phức tạp. Bạn có thể xem nguyên lý của HyperLogLog và cách triển khai trong Redis tại bài viết này: [Giải thích nguyên lý của thuật toán HyperLogLog và cách Redis áp dụng nó](https://juejin.cn/post/6844903785744056333).

Tiếp theo là một công cụ giúp hiểu nguyên lý của HyperLogLog: [Sketch of the Day: HyperLogLog — Cornerstone of a Big Data Infrastructure](http://content.research.neustar.biz/blog/hll.html).

Ngoài HyperLogLog, Redis còn cung cấp các cấu trúc dữ liệu xác suất khác. Địa chỉ tài liệu chính thức tương ứng: <https://redis.io/docs/data-types/probabilistic/>.

### Lệnh thường dùng

Các lệnh liên quan đến HyperLogLog rất ít, thường dùng nhất chỉ có 3 lệnh.

| Lệnh                                      | Giới thiệu                                                                                       |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------ |
| PFADD key element1 element2 ...           | Thêm một hoặc nhiều phần tử vào HyperLogLog                                                      |
| PFCOUNT key1 key2                         | Lấy số lượng duy nhất của một hoặc nhiều HyperLogLog                                             |
| PFMERGE destkey sourcekey1 sourcekey2 ... | Gộp nhiều HyperLogLog vào destkey; destkey kết hợp các nguồn để tính số lượng duy nhất tương ứng |

**Minh họa thao tác cơ bản với HyperLogLog**:

```bash
> PFADD hll foo bar zap
(integer) 1
> PFADD hll zap zap zap
(integer) 0
> PFADD hll foo bar
(integer) 0
> PFCOUNT hll
(integer) 3
> PFADD some-other-hll 1 2 3
(integer) 1
> PFCOUNT hll some-other-hll
(integer) 6
> PFMERGE desthll hll some-other-hll
"OK"
> PFCOUNT desthll
(integer) 6
```

### Trường hợp áp dụng

**Các trường hợp đếm số lượng cực lớn (từ cấp triệu, chục triệu trở lên)**

- Ví dụ: thống kê số lượng IP truy cập website phổ biến theo ngày/tuần/tháng, thống kê UV của bài đăng phổ biến.
- Lệnh liên quan: `PFADD`, `PFCOUNT`.

## Geospatial (vị trí địa lý)

### Giới thiệu

Geospatial index (chỉ mục không gian địa lý, gọi tắt là GEO) chủ yếu được dùng để lưu trữ thông tin vị trí địa lý và được triển khai dựa trên Sorted Set.

Với GEO, chúng ta có thể dễ dàng tính khoảng cách giữa hai vị trí, lấy các phần tử ở gần một vị trí được chỉ định và thực hiện các chức năng khác.

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220720194359494.png)

### Lệnh thường dùng

| Lệnh                                             | Giới thiệu                                                                                                                                       |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| GEOADD key longitude1 latitude1 member1 ...      | Thêm thông tin kinh độ, vĩ độ tương ứng của một hoặc nhiều phần tử vào GEO                                                                       |
| GEOPOS key member1 member2 ...                   | Trả về thông tin kinh độ, vĩ độ của các phần tử đã cho                                                                                           |
| GEODIST key member1 member2 M/KM/FT/MI           | Trả về khoảng cách giữa hai phần tử đã cho                                                                                                       |
| GEORADIUS key longitude latitude radius distance | Lấy các phần tử khác trong phạm vi distance quanh vị trí được chỉ định, hỗ trợ các tham số ASC (gần đến xa), DESC (xa đến gần), Count (số lượng) |
| GEORADIUSBYMEMBER key member radius distance     | Tương tự lệnh GEORADIUS, chỉ khác là điểm trung tâm tham chiếu là một phần tử trong GEO                                                          |

**Thao tác cơ bản**:

```bash
> GEOADD personLocation 116.33 39.89 user1 116.34 39.90 user2 116.35 39.88 user3
3
> GEOPOS personLocation user1
116.3299986720085144
39.89000061669732844
> GEODIST personLocation user1 user2 km
1.4018
```

Khi dùng công cụ trực quan hóa Redis để xem `personLocation`, đúng như dự đoán, cấu trúc bên dưới chính là Sorted Set.

Dữ liệu kinh độ, vĩ độ của thông tin vị trí địa lý được lưu trong GEO được chuyển đổi thành một số nguyên thông qua thuật toán GeoHash. Số nguyên này được dùng làm score (tham số trọng số) của Sorted Set.

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220721201545147.png)

**Lấy các phần tử khác trong phạm vi của vị trí được chỉ định**:

```bash
> GEORADIUS personLocation 116.33 39.87 3 km
user3
user1
> GEORADIUS personLocation 116.33 39.87 2 km
> GEORADIUS personLocation 116.33 39.87 5 km
user3
user1
user2
> GEORADIUSBYMEMBER personLocation user1 5 km
user3
user1
user2
> GEORADIUSBYMEMBER personLocation user1 2 km
user1
user2
```

Bạn có thể xem bài viết của Alibaba này để tìm hiểu nguyên lý bên dưới của lệnh `GEORADIUS`: [Redis thực sự triển khai chức năng “người ở gần” như thế nào?](https://juejin.cn/post/6844903966061363207).

**Xóa phần tử**:

GEO được triển khai dựa trên Sorted Set, nên bạn có thể dùng các lệnh liên quan đến Sorted Set cho GEO.

```bash
> ZREM personLocation user1
1
> ZRANGE personLocation 0 -1
user3
user2
> ZSCORE personLocation user2
4069879562983946
```

### Trường hợp áp dụng

**Các trường hợp cần quản lý và sử dụng dữ liệu không gian địa lý**

- Ví dụ: người ở gần.
- Lệnh liên quan: `GEOADD`, `GEORADIUS`, `GEORADIUSBYMEMBER`.

## Tổng kết

| Kiểu dữ liệu     | Mô tả                                                                                                                                                                                                                                                                                                                                               |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Bitmap           | Bạn có thể xem Bitmap như một array lưu trữ các số nhị phân (0 và 1), chỉ số của mỗi phần tử trong array được gọi là offset. Với Bitmap, chỉ cần một bit để biểu diễn giá trị hoặc trạng thái tương ứng của một phần tử, còn key chính là phần tử đó. 8 bit có thể tạo thành một byte, vì vậy bản thân Bitmap tiết kiệm đáng kể không gian lưu trữ. |
| HyperLogLog      | HyperLogLog do Redis cung cấp chiếm không gian cực kỳ nhỏ, chỉ cần 12k không gian là có thể lưu gần `2^64` phần tử khác nhau. Tuy nhiên, kết quả đếm của HyperLogLog không phải là giá trị chính xác và có sai số nhất định (sai số chuẩn là `0.81%`).                                                                                              |
| Geospatial index | Geospatial index (chỉ mục không gian địa lý, gọi tắt là GEO) chủ yếu được dùng để lưu trữ thông tin vị trí địa lý và được triển khai dựa trên Sorted Set.                                                                                                                                                                                           |

## Tham khảo

- Redis Data Structures: <https://redis.com/redis-enterprise/data-structures/>.
- 《Redis: Khám phá chuyên sâu: nguyên lý cốt lõi và thực tiễn áp dụng》 mục 1.6 — HyperLogLog: dùng ít tài nguyên để đạt hiệu quả lớn
- Bloom filter, Bitmap, HyperLogLog: <https://hogwartsrico.github.io/2020/06/08/BloomFilter-HyperLogLog-BitMap/index.html>

<!-- @include: @article-footer.snippet.md -->
