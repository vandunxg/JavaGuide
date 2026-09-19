---
title: Giải thích chi tiết về phân mảnh bộ nhớ Redis
description: Phân tích chuyên sâu nguyên nhân, cách kiểm tra và phương án tối ưu phân mảnh bộ nhớ Redis, bao gồm cách tính tỷ lệ phân mảnh bộ nhớ, nguyên lý của allocator jemalloc và cấu hình tự động dọn dẹp phân mảnh bộ nhớ.
category: Database
tag:
  - Redis
head:
  - - meta
    - name: keywords
      content: phân mảnh bộ nhớ Redis,tỷ lệ phân mảnh bộ nhớ,jemalloc,cấp phát bộ nhớ,activedefrag,tối ưu bộ nhớ,quản lý bộ nhớ Redis
---

## Phân mảnh bộ nhớ là gì?

Bạn có thể hiểu đơn giản phân mảnh bộ nhớ là những vùng bộ nhớ trống không thể sử dụng.

Ví dụ: hệ điều hành cấp cho bạn một vùng bộ nhớ liên tục 32 byte, nhưng dữ liệu bạn lưu trữ thực tế chỉ cần 24 byte. Nếu 8 byte bộ nhớ dư ra này sau đó không thể được cấp phát để lưu trữ dữ liệu khác, nó có thể được gọi là phân mảnh bộ nhớ.

![Phân mảnh bộ nhớ](https://oss.javaguide.cn/github/javaguide/memory-fragmentation.png)

Phân mảnh bộ nhớ Redis tuy không ảnh hưởng đến performance của Redis nhưng sẽ làm tăng mức tiêu thụ bộ nhớ.

## Vì sao Redis có phân mảnh bộ nhớ?

Có 2 nguyên nhân phổ biến khiến Redis phát sinh phân mảnh bộ nhớ:

**1. Vùng bộ nhớ Redis yêu cầu hệ điều hành cấp khi lưu trữ dữ liệu có thể lớn hơn vùng bộ nhớ thực tế dữ liệu cần.**

Dưới đây là nguyên văn từ tài liệu chính thức của Redis:

> To store user keys, Redis allocates at most as much memory as the `maxmemory` setting enables (however there are small extra allocations possible).

Khi Redis sử dụng hàm `zmalloc` (hàm cấp phát bộ nhớ do Redis tự triển khai) để cấp phát bộ nhớ, ngoài việc cấp phát vùng bộ nhớ có kích thước `size`, nó còn cấp phát thêm vùng bộ nhớ có kích thước `PREFIX_SIZE`.

Mã nguồn của hàm `zmalloc` như sau (địa chỉ mã nguồn: <https://github.com/antirez/redis-tools/blob/master/zmalloc.c>):

```java
void *zmalloc(size_t size) {
   // Cấp phát bộ nhớ có kích thước được chỉ định
   void *ptr = malloc(size+PREFIX_SIZE);
   if (!ptr) zmalloc_oom_handler(size);
#ifdef HAVE_MALLOC_SIZE
   update_zmalloc_stat_alloc(zmalloc_size(ptr));
   return ptr;
#else
   *((size_t*)ptr) = size;
   update_zmalloc_stat_alloc(size+PREFIX_SIZE);
   return (char*)ptr+PREFIX_SIZE;
#endif
}
```

Ngoài ra, Redis có thể sử dụng nhiều allocator để cấp phát bộ nhớ (libc, jemalloc, tcmalloc), mặc định sử dụng [jemalloc](https://github.com/jemalloc/jemalloc). jemalloc cấp phát bộ nhớ theo một loạt kích thước cố định (8 byte, 16 byte, 32 byte...). Các đơn vị bộ nhớ do jemalloc phân chia được minh họa như hình dưới đây:

![Sơ đồ đơn vị bộ nhớ jemalloc](https://oss.javaguide.cn/github/javaguide/database/redis/6803d3929e3e46c1b1c9d0bb9ee8e717.png)

Khi kích thước bộ nhớ chương trình yêu cầu gần nhất với một giá trị cố định, jemalloc sẽ cấp cho chương trình vùng bộ nhớ có kích thước tương ứng. Ví dụ, nếu chương trình cần 17 byte bộ nhớ, jemalloc sẽ cấp trực tiếp 32 byte, dẫn đến lãng phí 15 byte bộ nhớ. Tuy nhiên, jemalloc đã được tối ưu riêng cho vấn đề phân mảnh bộ nhớ nên thông thường sẽ không xảy ra phân mảnh quá mức.

**2. Việc thường xuyên sửa đổi dữ liệu trong Redis cũng tạo ra phân mảnh bộ nhớ.**

Khi một dữ liệu nào đó trong Redis bị xóa, Redis thường không giải phóng ngay bộ nhớ cho hệ điều hành.

Điều này cũng được nêu nguyên văn trong tài liệu chính thức của Redis:

![](https://oss.javaguide.cn/github/javaguide/redis-docs-memory-optimization.png)

Địa chỉ tài liệu: <https://redis.io/topics/memory-optimization>.

## Làm thế nào để xem thông tin phân mảnh bộ nhớ Redis?

Sử dụng lệnh `info memory` để xem thông tin liên quan đến bộ nhớ Redis. Ý nghĩa cụ thể của từng tham số trong hình dưới đây được giải thích chi tiết trong tài liệu chính thức của Redis: <https://redis.io/commands/INFO>.

![](https://oss.javaguide.cn/github/javaguide/redis-info-memory.png)

Công thức tính tỷ lệ phân mảnh bộ nhớ Redis: `mem_fragmentation_ratio` (tỷ lệ phân mảnh bộ nhớ) = `used_memory_rss` (kích thước vùng bộ nhớ vật lý mà hệ điều hành thực tế đã cấp phát cho Redis) / `used_memory` (kích thước vùng bộ nhớ mà allocator của Redis thực tế yêu cầu để lưu trữ dữ liệu)

Nói cách khác, giá trị `mem_fragmentation_ratio` (tỷ lệ phân mảnh bộ nhớ) càng lớn thì tỷ lệ phân mảnh bộ nhớ càng nghiêm trọng.

Tuyệt đối không được nhầm rằng giá trị `used_memory_rss` trừ `used_memory` chính là kích thước phân mảnh bộ nhớ! Giá trị này còn bao gồm chi phí của các process khác, cùng chi phí của shared library, stack và các thành phần khác.

Có thể nhiều bạn sẽ hỏi: “Tỷ lệ phân mảnh bộ nhớ bao nhiêu thì cần dọn dẹp?”.

Thông thường, chúng ta cho rằng chỉ cần dọn dẹp phân mảnh bộ nhớ khi `mem_fragmentation_ratio > 1.5`. `mem_fragmentation_ratio > 1.5` có nghĩa là để lưu trữ 2G dữ liệu thực tế trong Redis, bạn cần hơn 3G bộ nhớ.

Nếu muốn xem nhanh tỷ lệ phân mảnh bộ nhớ, bạn cũng có thể sử dụng lệnh sau:

```bash
> redis-cli -p 6379 info | grep mem_fragmentation_ratio
```

Ngoài ra, tỷ lệ phân mảnh bộ nhớ có thể nhỏ hơn 1. Trường hợp này tôi chưa từng gặp trong quá trình sử dụng hằng ngày. Nếu quan tâm, bạn có thể xem bài viết [Phân tích sự cố | Phải làm gì khi tỷ lệ phân mảnh bộ nhớ Redis quá thấp? - Cộng đồng mã nguồn mở Aikeson](https://mp.weixin.qq.com/s/drlDvp7bfq5jt2M5pTqJCw).

## Làm thế nào để dọn dẹp phân mảnh bộ nhớ Redis?

Từ Redis 4.0-RC3 trở đi, Redis đã tích hợp sẵn tính năng defragmentation, giúp tránh tỷ lệ phân mảnh bộ nhớ quá lớn.

Chỉ cần sử dụng lệnh `config set` để đặt tùy chọn cấu hình `activedefrag` thành `yes`.

```bash
config set activedefrag yes
```

Thời điểm dọn dẹp cụ thể cần được kiểm soát thông qua hai tham số sau:

```bash
# Bắt đầu dọn dẹp khi dung lượng phân mảnh bộ nhớ đạt 500mb
config set active-defrag-ignore-bytes 500mb
# Bắt đầu dọn dẹp khi tỷ lệ phân mảnh bộ nhớ lớn hơn 1.5
config set active-defrag-threshold-lower 50
```

Cơ chế tự động dọn dẹp phân mảnh bộ nhớ của Redis có thể ảnh hưởng đến performance của Redis. Bạn có thể giảm ảnh hưởng đến performance của Redis thông qua hai tham số sau:

```bash
# Tỷ lệ thời gian CPU dành cho việc dọn dẹp phân mảnh bộ nhớ không thấp hơn 20%
config set active-defrag-cycle-min 20
# Tỷ lệ thời gian CPU dành cho việc dọn dẹp phân mảnh bộ nhớ không cao hơn 50%
config set active-defrag-cycle-max 50
```

Ngoài ra, restart node cũng có thể sắp xếp lại phân mảnh bộ nhớ. Nếu Redis của bạn chạy trong cluster theo kiến trúc high availability, có thể chuyển primary node có tỷ lệ phân mảnh quá cao thành replica node để restart an toàn.

## Tham khảo

- Tài liệu chính thức của Redis: <https://redis.io/topics/memory-optimization>
- Công nghệ cốt lõi và thực chiến Redis - Geek Time - Sau khi xóa dữ liệu, vì sao mức sử dụng bộ nhớ vẫn rất cao?: <https://time.geekbang.org/column/article/289140>
- Phân tích mã nguồn Redis - cấp phát bộ nhớ: <<https://shinerio.cc/2020/05/17/redis/Redis> Phân tích mã nguồn - quản lý bộ nhớ>

<!-- @include: @article-footer.snippet.md -->
