---
title: "Giải thích chi tiết 5 kiểu dữ liệu cơ bản của Redis"
description: "Giải thích chi tiết cách sử dụng và các trường hợp áp dụng của 5 kiểu dữ liệu cơ bản String, List, Set, Hash, Zset của Redis, phân tích sâu nguyên lý triển khai của các cấu trúc dữ liệu tầng dưới như SDS, skip list, ziplist."
category: Database
tag:
  - Redis
head:
  - - meta
    - name: keywords
      content: Redis data types,String,List,Set,Hash,Zset,SDS,skip list,ziplist,Redis commands
---

Redis có tổng cộng 5 kiểu dữ liệu cơ bản: String (chuỗi), List (danh sách), Set (tập hợp), Hash (bảng băm), Zset (tập hợp có thứ tự).

5 kiểu dữ liệu này được cung cấp trực tiếp cho người dùng, là dạng lưu trữ dữ liệu. Cơ chế triển khai tầng dưới chủ yếu dựa trên 8 cấu trúc dữ liệu sau: chuỗi động đơn giản (SDS), LinkedList (linked list hai chiều), Dict (hash table/từ điển), SkipList (skip list), Intset (tập hợp số nguyên), ZipList (ziplist), QuickList (danh sách nhanh).

Các cấu trúc dữ liệu tầng dưới tương ứng với 5 kiểu dữ liệu cơ bản của Redis được triển khai như bảng sau:

| String | List                         | Hash         | Set         | Zset             |
| :----- | :--------------------------- | :----------- | :---------- | :--------------- |
| SDS    | LinkedList/ZipList/QuickList | Dict,ZipList | Dict,Intset | ZipList,SkipList |

Trước Redis 3.2, cơ chế triển khai tầng dưới của List là LinkedList hoặc ZipList. Từ Redis 3.2 trở đi, Redis đưa vào QuickList, là sự kết hợp giữa LinkedList và ZipList, nên cơ chế triển khai tầng dưới của List chuyển thành QuickList. Từ Redis 7.0, ZipList được thay thế bằng ListPack.

Bạn có thể tìm thấy phần giới thiệu rất chi tiết về các kiểu/cấu trúc dữ liệu của Redis trên website chính thức của Redis:

- [Redis Data Structures](https://redis.com/redis-enterprise/data-structures/)
- [Redis Data types tutorial](https://redis.io/docs/manual/data-types/data-types-tutorial/)

Trong tương lai, khi các phiên bản Redis mới được phát hành, có thể sẽ xuất hiện các cấu trúc dữ liệu mới. Bằng cách tham khảo phần giới thiệu tương ứng trên website chính thức của Redis, bạn luôn có thể tiếp cận thông tin đáng tin cậy nhất.

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220720181630203.png)

## String (chuỗi)

### Giới thiệu

String là kiểu dữ liệu đơn giản nhất và cũng là kiểu dữ liệu được sử dụng phổ biến nhất trong Redis.

String là kiểu dữ liệu binary-safe, có thể dùng để lưu trữ mọi loại dữ liệu như chuỗi, số nguyên, số thực, hình ảnh (dữ liệu ảnh được encode hoặc decode bằng base64, hoặc đường dẫn ảnh), đối tượng sau serialization.

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220719124403897.png)

Mặc dù Redis được viết bằng ngôn ngữ C, Redis không sử dụng cách biểu diễn chuỗi của C mà tự xây dựng một **chuỗi động đơn giản** (Simple Dynamic String, **SDS**). So với chuỗi gốc của C, SDS của Redis không chỉ lưu được dữ liệu dạng văn bản mà còn lưu được dữ liệu nhị phân, đồng thời độ phức tạp khi lấy độ dài chuỗi là O(1) (chuỗi C là O(N)). Ngoài ra, API SDS của Redis an toàn và không gây tràn buffer.

### Lệnh thường dùng

| Lệnh                            | Giới thiệu                                       |
| ------------------------------- | ------------------------------------------------ |
| SET key value                   | Đặt giá trị cho key được chỉ định                |
| SETNX key value                 | Chỉ đặt giá trị cho key khi key chưa tồn tại     |
| GET key                         | Lấy giá trị của key được chỉ định                |
| MSET key1 value1 key2 value2 …… | Đặt giá trị cho một hoặc nhiều key được chỉ định |
| MGET key1 key2 ...              | Lấy giá trị của một hoặc nhiều key được chỉ định |
| STRLEN key                      | Trả về độ dài giá trị chuỗi được lưu trong key   |
| INCR key                        | Tăng giá trị số được lưu trong key lên một       |
| DECR key                        | Giảm giá trị số được lưu trong key xuống một     |
| EXISTS key                      | Kiểm tra key được chỉ định có tồn tại hay không  |
| DEL key (dùng chung)            | Xóa key được chỉ định                            |
| EXPIRE key seconds (dùng chung) | Đặt thời gian hết hạn cho key được chỉ định      |

Để xem thêm các lệnh Redis String và hướng dẫn sử dụng chi tiết, hãy xem phần giới thiệu tương ứng trên website chính thức của Redis: <https://redis.io/commands/?group=string> .

**Thao tác cơ bản**:

```bash
> SET key value
OK
> GET key
"value"
> EXISTS key
(integer) 1
> STRLEN key
(integer) 5
> DEL key
(integer) 1
> GET key
(nil)
```

**Thiết lập hàng loạt**:

```bash
> MSET key1 value1 key2 value2
OK
> MGET key1 key2 # Lấy hàng loạt value tương ứng với nhiều key
1) "value1"
2) "value2"
```

**Bộ đếm (có thể sử dụng khi nội dung chuỗi là số nguyên):**

```bash
> SET number 1
OK
> INCR number # Tăng giá trị số được lưu trong key lên một
(integer) 2
> GET number
"2"
> DECR number # Giảm giá trị số được lưu trong key xuống một
(integer) 1
> GET number
"1"
```

**Đặt thời gian hết hạn (mặc định là không bao giờ hết hạn):**

```bash
> EXPIRE key 60
(integer) 1
> SETEX key 60 value # Đặt giá trị và thời gian hết hạn
OK
> TTL key
(integer) 56
```

### Trường hợp sử dụng

**Trường hợp cần lưu trữ dữ liệu thông thường**

- Ví dụ: cache Session, Token, địa chỉ hình ảnh, đối tượng sau serialization (tiết kiệm bộ nhớ hơn so với lưu bằng Hash).
- Lệnh liên quan: `SET`, `GET`.

**Trường hợp cần đếm**

- Ví dụ: số request của người dùng trong một đơn vị thời gian (có thể dùng cho rate limiting đơn giản), số lượt truy cập trang trong một đơn vị thời gian.
- Lệnh liên quan: `SET`, `GET`, `INCR`, `DECR`.

**Distributed lock**

Có thể dùng lệnh `SETNX key value` để triển khai một distributed lock đơn giản nhất (vẫn tồn tại một số thiếu sót, thường không khuyến nghị triển khai distributed lock theo cách này).

## List (danh sách)

### Giới thiệu

List trong Redis thực chất là cơ chế triển khai của cấu trúc dữ liệu linked list. Bài viết [Cấu trúc dữ liệu tuyến tính: array, linked list, stack, queue](https://javaguide.cn/cs-basics/data-structure/linear-data-structure.html) đã giới thiệu chi tiết về cấu trúc dữ liệu này, nên ở đây không giới thiệu thêm.

Nhiều ngôn ngữ lập trình cấp cao có sẵn cơ chế triển khai linked list, chẳng hạn như `LinkedList` trong Java, nhưng ngôn ngữ C không triển khai linked list, vì vậy Redis đã tự triển khai cấu trúc dữ liệu linked list của mình. List của Redis được triển khai bằng **linked list hai chiều**, tức là hỗ trợ tìm kiếm và duyệt ngược, giúp thao tác thuận tiện hơn nhưng làm phát sinh thêm một phần chi phí bộ nhớ.

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220719124413287.png)

### Lệnh thường dùng

| Lệnh                        | Giới thiệu                                                                |
| --------------------------- | ------------------------------------------------------------------------- |
| RPUSH key value1 value2 ... | Thêm một hoặc nhiều phần tử vào cuối (bên phải) của list được chỉ định    |
| LPUSH key value1 value2 ... | Thêm một hoặc nhiều phần tử vào đầu (bên trái) của list được chỉ định     |
| LSET key index value        | Đặt giá trị tại vị trí index trong list được chỉ định thành value         |
| LPOP key                    | Xóa và lấy phần tử đầu tiên (ngoài cùng bên trái) của list được chỉ định  |
| RPOP key                    | Xóa và lấy phần tử cuối cùng (ngoài cùng bên phải) của list được chỉ định |
| LLEN key                    | Lấy số lượng phần tử trong list                                           |
| LRANGE key start end        | Lấy các phần tử giữa start và end của list                                |

Để xem thêm các lệnh Redis List và hướng dẫn sử dụng chi tiết, hãy xem phần giới thiệu tương ứng trên website chính thức của Redis: <https://redis.io/commands/?group=list> .

**Dùng `RPUSH/LPOP` hoặc `LPUSH/RPOP` để triển khai queue**:

```bash
> RPUSH myList value1
(integer) 1
> RPUSH myList value2 value3
(integer) 3
> LPOP myList
"value1"
> LRANGE myList 0 1
1) "value2"
2) "value3"
> LRANGE myList 0 -1
1) "value2"
2) "value3"
```

**Dùng `RPUSH/RPOP` hoặc `LPUSH/LPOP` để triển khai stack**:

```bash
> RPUSH myList2 value1 value2 value3
(integer) 3
> RPOP myList2 # Lấy phần tử ngoài cùng bên phải của list ra
"value3"
```

Tôi đã vẽ riêng một sơ đồ để mọi người dễ hình dung các lệnh `RPUSH`, `LPOP`, `LPUSH`, `RPOP`:

![](https://oss.javaguide.cn/github/javaguide/database/redis/redis-list.png)

**Dùng `LRANGE` để xem các phần tử trong phạm vi chỉ số tương ứng của list**:

```bash
> RPUSH myList value1 value2 value3
(integer) 3
> LRANGE myList 0 1
1) "value1"
2) "value2"
> LRANGE myList 0 -1
1) "value1"
2) "value2"
3) "value3"
```

Thông qua lệnh `LRANGE`, bạn có thể triển khai truy vấn phân trang dựa trên List với performance rất cao!

**Dùng `LLEN` để xem độ dài linked list**:

```bash
> LLEN myList
(integer) 3
```

### Trường hợp sử dụng

**Hiển thị feed thông tin**

- Ví dụ: bài viết mới nhất, cập nhật mới nhất.
- Lệnh liên quan: `LPUSH`, `LRANGE`.

**Message queue**

`List` có thể dùng làm message queue, chỉ là chức năng quá đơn giản và tồn tại nhiều thiếu sót, không khuyến nghị sử dụng theo cách này.

Tương đối mà nói, cấu trúc dữ liệu `Stream` mới được Redis 5.0 thêm vào phù hợp hơn để làm message queue, nhưng chức năng vẫn rất sơ sài. So với message queue chuyên dụng, nó vẫn còn nhiều thiếu sót, chẳng hạn như khó giải quyết vấn đề mất message và message tồn đọng.

## Hash (hash)

### Giới thiệu

Hash trong Redis là một bảng ánh xạ field-value (cặp key-value) kiểu String, đặc biệt phù hợp để lưu trữ object. Khi thao tác về sau, bạn có thể trực tiếp sửa giá trị của một số field trong object này.

Hash tương tự `HashMap` trước JDK1.8, cách triển khai bên trong cũng tương tự (array + linked list). Tuy nhiên, Hash của Redis đã được tối ưu thêm.

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220719124421703.png)

### Lệnh thường dùng

| Lệnh                                      | Giới thiệu                                                                                             |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| HSET key field value                      | Đặt giá trị của field được chỉ định trong hash table được chỉ định                                     |
| HSETNX key field value                    | Chỉ đặt giá trị của field được chỉ định khi field chưa tồn tại                                         |
| HMSET key field1 value1 field2 value2 ... | Đồng thời đặt một hoặc nhiều cặp field-value vào hash table được chỉ định                              |
| HGET key field                            | Lấy giá trị của field được chỉ định trong hash table được chỉ định                                     |
| HMGET key field1 field2 ...               | Lấy giá trị của một hoặc nhiều field được chỉ định trong hash table được chỉ định                      |
| HGETALL key                               | Lấy toàn bộ cặp key-value trong hash table được chỉ định                                               |
| HEXISTS key field                         | Kiểm tra field được chỉ định trong hash table được chỉ định có tồn tại hay không                       |
| HDEL key field1 field2 ...                | Xóa một hoặc nhiều field của hash table                                                                |
| HLEN key                                  | Lấy số lượng field trong hash table                                                                    |
| HINCRBY key field increment               | Thực hiện phép tính trên field được chỉ định trong hash được chỉ định (số dương là cộng, số âm là trừ) |

Để xem thêm các lệnh Redis Hash và hướng dẫn sử dụng chi tiết, hãy xem phần giới thiệu tương ứng trên website chính thức của Redis: <https://redis.io/commands/?group=hash> .

**Mô phỏng lưu trữ dữ liệu object**:

```bash
> HMSET userInfoKey name "guide" description "dev" age 24
OK
> HEXISTS userInfoKey name # Kiểm tra field được chỉ định trong value tương ứng với key có tồn tại hay không.
(integer) 1
> HGET userInfoKey name # Lấy giá trị của field được chỉ định lưu trong hash table.
"guide"
> HGET userInfoKey age
"24"
> HGETALL userInfoKey # Lấy toàn bộ field và value của key được chỉ định trong hash table
1) "name"
2) "guide"
3) "description"
4) "dev"
5) "age"
6) "24"
> HSET userInfoKey name "GuideGeGe"
> HGET userInfoKey name
"GuideGeGe"
> HINCRBY userInfoKey age 2
(integer) 26
```

### Trường hợp sử dụng

**Trường hợp lưu trữ dữ liệu object**

- Ví dụ: thông tin người dùng, thông tin sản phẩm, thông tin bài viết, thông tin giỏ hàng.
- Lệnh liên quan: `HSET` (đặt giá trị của một field), `HMSET` (đặt giá trị của nhiều field), `HGET` (lấy giá trị của một field), `HMGET` (lấy giá trị của nhiều field).

## Set (tập hợp)

### Giới thiệu

Kiểu Set trong Redis là một tập hợp không có thứ tự. Các phần tử trong tập hợp không có thứ tự trước sau nhưng đều là duy nhất, tương tự `HashSet` trong Java. Khi cần lưu trữ một list nhưng không muốn xuất hiện dữ liệu trùng lặp, Set là một lựa chọn tốt. Ngoài ra, Set cung cấp API quan trọng để kiểm tra một phần tử có nằm trong một Set hay không, điều mà List không thể cung cấp.

Bạn có thể dễ dàng triển khai các phép toán giao, hợp và hiệu trên Set. Chẳng hạn, bạn có thể lưu tất cả người mà một người dùng theo dõi vào một tập hợp, đồng thời lưu tất cả follower của người đó vào một tập hợp khác. Khi đó, Set có thể rất thuận tiện để triển khai các chức năng như theo dõi chung, follower chung và sở thích chung. Đây chính là quá trình tìm giao.

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220719124430264.png)

### Lệnh thường dùng

| Lệnh                                  | Giới thiệu                                                                  |
| ------------------------------------- | --------------------------------------------------------------------------- |
| SADD key member1 member2 ...          | Thêm một hoặc nhiều phần tử vào tập hợp được chỉ định                       |
| SMEMBERS key                          | Lấy toàn bộ phần tử trong tập hợp được chỉ định                             |
| SCARD key                             | Lấy số lượng phần tử trong tập hợp được chỉ định                            |
| SISMEMBER key member                  | Kiểm tra phần tử được chỉ định có nằm trong tập hợp được chỉ định hay không |
| SINTER key1 key2 ...                  | Lấy giao của tất cả tập hợp đã cho                                          |
| SINTERSTORE destination key1 key2 ... | Lưu giao của tất cả tập hợp đã cho vào destination                          |
| SUNION key1 key2 ...                  | Lấy hợp của tất cả tập hợp đã cho                                           |
| SUNIONSTORE destination key1 key2 ... | Lưu hợp của tất cả tập hợp đã cho vào destination                           |
| SDIFF key1 key2 ...                   | Lấy hiệu của tất cả tập hợp đã cho                                          |
| SDIFFSTORE destination key1 key2 ...  | Lưu hiệu của tất cả tập hợp đã cho vào destination                          |
| SPOP key count                        | Xóa ngẫu nhiên và lấy một hoặc nhiều phần tử trong tập hợp được chỉ định    |
| SRANDMEMBER key count                 | Lấy ngẫu nhiên số lượng phần tử được chỉ định trong tập hợp được chỉ định   |

Để xem thêm các lệnh Redis Set và hướng dẫn sử dụng chi tiết, hãy xem phần giới thiệu tương ứng trên website chính thức của Redis: <https://redis.io/commands/?group=set> .

**Thao tác cơ bản**:

```bash
> SADD mySet value1 value2
(integer) 2
> SADD mySet value1 # Không cho phép phần tử trùng lặp nên thêm thất bại
(integer) 0
> SMEMBERS mySet
1) "value1"
2) "value2"
> SCARD mySet
(integer) 2
> SISMEMBER mySet value1
(integer) 1
> SADD mySet2 value2 value3
(integer) 2
```

- `mySet`: `value1`, `value2`.
- `mySet2`: `value2`, `value3`.

**Tìm giao**:

```bash
> SINTERSTORE mySet3 mySet mySet2
(integer) 1
> SMEMBERS mySet3
1) "value2"
```

**Tìm hợp**:

```bash
> SUNION mySet mySet2
1) "value3"
2) "value2"
3) "value1"
```

**Tìm hiệu**:

```bash
> SDIFF mySet mySet2 # Hiệu là tập hợp gồm tất cả phần tử thuộc mySet nhưng không thuộc A
1) "value1"
```

### Trường hợp sử dụng

**Trường hợp dữ liệu cần lưu trữ không được trùng lặp**

- Ví dụ: thống kê UV của website (trong trường hợp dữ liệu rất lớn, `HyperLogLog` phù hợp hơn), lượt thích bài viết, lượt thích cập nhật và các trường hợp khác.
- Lệnh liên quan: `SCARD` (lấy số lượng phần tử trong tập hợp).

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220719073733851.png)

**Trường hợp cần lấy giao, hợp và hiệu của nhiều nguồn dữ liệu**

- Ví dụ: bạn bè chung (giao), follower chung (giao), theo dõi chung (giao), gợi ý bạn bè (hiệu), gợi ý âm nhạc (hiệu), gợi ý tài khoản đăng ký (hiệu + giao) và các trường hợp khác.
- Lệnh liên quan: `SINTER` (giao), `SINTERSTORE` (giao), `SUNION` (hợp), `SUNIONSTORE` (hợp), `SDIFF` (hiệu), `SDIFFSTORE` (hiệu).

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220719074543513.png)

**Trường hợp cần lấy ngẫu nhiên các phần tử trong nguồn dữ liệu**

- Ví dụ: hệ thống xổ số, gọi tên ngẫu nhiên và các trường hợp khác.
- Lệnh liên quan: `SPOP` (lấy và xóa ngẫu nhiên phần tử trong tập hợp, phù hợp với trường hợp không cho phép trúng thưởng trùng lặp), `SRANDMEMBER` (lấy ngẫu nhiên phần tử trong tập hợp, phù hợp với trường hợp cho phép trúng thưởng trùng lặp).

## Sorted Set (tập hợp có thứ tự)

### Giới thiệu

Sorted Set tương tự Set, nhưng so với Set, Sorted Set bổ sung một tham số trọng số `score`, giúp các phần tử trong tập hợp được sắp xếp theo thứ tự dựa trên `score`. Ngoài ra, có thể lấy danh sách phần tử theo phạm vi của `score`. Nó hơi giống sự kết hợp giữa `HashMap` và `TreeSet` trong Java.

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220719124437791.png)

### Lệnh thường dùng

| Lệnh                                          | Giới thiệu                                                                                                                                                              |
| --------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ZADD key score1 member1 score2 member2 ...    | Thêm một hoặc nhiều phần tử vào sorted set được chỉ định                                                                                                                |
| ZCARD KEY                                     | Lấy số lượng phần tử trong sorted set được chỉ định                                                                                                                     |
| ZSCORE key member                             | Lấy giá trị score của phần tử được chỉ định trong sorted set được chỉ định                                                                                              |
| ZINTERSTORE destination numkeys key1 key2 ... | Lưu giao của tất cả sorted set đã cho vào destination, thực hiện phép tổng hợp SUM trên giá trị score tương ứng của các phần tử giống nhau, numkeys là số lượng tập hợp |
| ZUNIONSTORE destination numkeys key1 key2 ... | Tìm hợp, các nội dung khác tương tự ZINTERSTORE                                                                                                                         |
| ZDIFFSTORE destination numkeys key1 key2 ...  | Tìm hiệu, các nội dung khác tương tự ZINTERSTORE                                                                                                                        |
| ZRANGE key start end                          | Lấy các phần tử giữa start và end của sorted set được chỉ định (score từ thấp đến cao)                                                                                  |
| ZREVRANGE key start end                       | Lấy các phần tử giữa start và end của sorted set được chỉ định (score từ cao xuống thấp)                                                                                |
| ZREVRANK key member                           | Lấy thứ hạng của phần tử được chỉ định trong sorted set được chỉ định (sắp xếp theo score từ cao xuống thấp)                                                            |

Để xem thêm các lệnh Redis Sorted Set và hướng dẫn sử dụng chi tiết, hãy xem phần giới thiệu tương ứng trên website chính thức của Redis: <https://redis.io/commands/?group=sorted-set> .

**Thao tác cơ bản**:

```bash
> ZADD myZset 2.0 value1 1.0 value2
(integer) 2
> ZCARD myZset
2
> ZSCORE myZset value1
2.0
> ZRANGE myZset 0 1
1) "value2"
2) "value1"
> ZREVRANGE myZset 0 1
1) "value1"
2) "value2"
> ZADD myZset2 4.0 value2 3.0 value3
(integer) 2

```

- `myZset`: `value1` (2.0), `value2` (1.0).
- `myZset2`: `value2` (4.0), `value3` (3.0).

**Lấy thứ hạng của phần tử được chỉ định**:

```bash
> ZREVRANK myZset value1
0
> ZREVRANK myZset value2
1
```

**Tìm giao**:

```bash
> ZINTERSTORE myZset3 2 myZset myZset2
1
> ZRANGE myZset3 0 1 WITHSCORES
value2
5
```

**Tìm hợp**:

```bash
> ZUNIONSTORE myZset4 2 myZset myZset2
3
> ZRANGE myZset4 0 2 WITHSCORES
value1
2
value3
3
value2
5
```

**Tìm hiệu**:

```bash
> ZDIFF 2 myZset myZset2 WITHSCORES
value1
2
```

### Trường hợp sử dụng

**Trường hợp cần lấy ngẫu nhiên các phần tử từ nguồn dữ liệu rồi sắp xếp theo một trọng số nào đó**

- Ví dụ: các loại bảng xếp hạng như bảng xếp hạng tặng quà trong phòng livestream, bảng xếp hạng số bước chân trên WeChat, bảng xếp hạng cấp bậc trong Honor of Kings, bảng xếp hạng độ hot của chủ đề và nhiều loại khác.
- Lệnh liên quan: `ZRANGE` (sắp xếp từ thấp đến cao), `ZREVRANGE` (sắp xếp từ cao xuống thấp), `ZREVRANK` (thứ hạng của phần tử được chỉ định).

![](https://oss.javaguide.cn/github/javaguide/database/redis/2021060714195385.png)

["Java Interview Compass"](https://javaguide.cn/zhuanlan/java-mian-shi-zhi-bei.html) có một bài viết trong mục "Câu hỏi phỏng vấn kỹ thuật" giới thiệu chi tiết cách dùng Sorted Set để thiết kế và triển khai một bảng xếp hạng.

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220719071115140.png)

**Trường hợp dữ liệu cần lưu trữ có mức độ ưu tiên hoặc mức độ quan trọng**, chẳng hạn như priority task queue.

- Ví dụ: priority task queue.
- Lệnh liên quan: `ZRANGE` (sắp xếp từ thấp đến cao), `ZREVRANGE` (sắp xếp từ cao xuống thấp), `ZREVRANK` (thứ hạng của phần tử được chỉ định).

## Tổng kết

| Kiểu dữ liệu | Mô tả                                                                                                                                                                                                                                                          |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| String       | Kiểu dữ liệu binary-safe, có thể dùng để lưu trữ mọi loại dữ liệu như chuỗi, số nguyên, số thực, hình ảnh (mã hóa hoặc giải mã base64 của hình ảnh, hoặc đường dẫn hình ảnh), object sau serialization.                                                        |
| List         | List của Redis được triển khai bằng linked list hai chiều, hỗ trợ tìm kiếm và duyệt ngược, giúp thao tác thuận tiện hơn nhưng làm phát sinh thêm một phần chi phí memory.                                                                                      |
| Hash         | Bảng ánh xạ field-value (cặp key-value) thuộc kiểu String, đặc biệt phù hợp để lưu trữ object. Khi thao tác về sau, bạn có thể trực tiếp sửa giá trị của một số field trong object này.                                                                        |
| Set          | Tập hợp không có thứ tự, các phần tử trong tập hợp không có thứ tự trước sau nhưng đều là duy nhất, tương tự `HashSet` trong Java.                                                                                                                             |
| Zset         | So với Set, Sorted Set bổ sung tham số trọng số `score`, giúp các phần tử trong tập hợp được sắp xếp theo thứ tự dựa trên `score`. Ngoài ra, có thể lấy list phần tử theo phạm vi của `score`. Nó hơi giống sự kết hợp giữa `HashMap` và `TreeSet` trong Java. |

## Đọc thêm về cấu trúc dữ liệu

Các kiểu dữ liệu Redis sử dụng nhiều cấu trúc dữ liệu cơ bản ở phía sau. Nếu muốn bổ sung kiến thức về cơ chế bên trong dưới góc độ phỏng vấn, bạn có thể đọc kết hợp các bài viết sau:

- [Giải thích chi tiết về cấu trúc dữ liệu tuyến tính](../../cs-basics/data-structure/linear-data-structure.md): hiểu mối quan hệ giữa List, queue và linked list.
- [Tổng hợp câu hỏi phỏng vấn về hash table](../../cs-basics/data-structure/hash-table.md): hiểu cách tìm kiếm và xử lý hash collision của các cấu trúc như Hash, Set.
- [Tổng hợp câu hỏi phỏng vấn về skip list](../../cs-basics/data-structure/skip-list.md): hiểu những đánh đổi trong cấu trúc phía sau khả năng truy vấn theo phạm vi và lấy thứ hạng của Sorted Set.

## Tham khảo

- Redis Data Structures: <https://redis.com/redis-enterprise/data-structures/> .
- Redis Commands: <https://redis.io/commands/> .
- Redis Data types tutorial: <https://redis.io/docs/manual/data-types/data-types-tutorial/> .
- Redis lưu trữ thông tin object bằng Hash hay String: <https://segmentfault.com/a/1190000040032006>

<!-- @include: @article-footer.snippet.md -->
