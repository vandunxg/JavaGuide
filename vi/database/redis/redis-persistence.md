---
title: Giải thích chi tiết cơ chế persistence của Redis
description: Phân tích nguyên lý hoạt động, cách cấu hình và ưu nhược điểm của ba cơ chế persistence Redis gồm RDB snapshot, AOF log và persistence hỗn hợp, giúp bạn chọn chiến lược persistence phù hợp với business scenario.
category: Database
tag:
  - Redis
head:
  - - meta
    - name: keywords
      content: Redis persistence,RDB,AOF,persistence hỗn hợp,bgsave,khôi phục dữ liệu,Redis backup,fork child process
---

Khi sử dụng cache, thường cần persistence dữ liệu trong memory, tức là ghi dữ liệu trong memory vào disk. Mục đích chủ yếu là tái sử dụng dữ liệu sau này (chẳng hạn restart máy, khôi phục dữ liệu sau khi máy gặp sự cố), hoặc đồng bộ dữ liệu (chẳng hạn master và replica trong Redis cluster đồng bộ dữ liệu qua file RDB).

Một điểm rất quan trọng khiến Redis khác Memcached là Redis hỗ trợ persistence, đồng thời hỗ trợ 3 phương thức:

- Snapshot (snapshotting, RDB)
- Append-only file (AOF)
- Persistence hỗn hợp giữa RDB và AOF (bổ sung từ Redis 4.0)

Địa chỉ tài liệu chính thức: <https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/> .

![](https://oss.javaguide.cn/github/javaguide/database/redis/redis4.0-persitence.png)

**Bài viết này dựa trên Redis phiên bản 7.0+**. Cơ chế persistence giữa các phiên bản có khác biệt quan trọng, hãy xác nhận phiên bản Redis trước khi sử dụng:

| Phiên bản      | Phương thức persistence mặc định | Tính năng quan trọng                        |
| -------------- | -------------------------------- | ------------------------------------------- |
| **Redis 4.0**  | RDB                              | Đưa vào persistence hỗn hợp RDB+AOF         |
| **Redis 6.0**  | RDB                              | AOF vẫn cần bật thủ công                    |
| **Redis 7.0**  | RDB                              | Đưa vào Multi-Part AOF                      |
| **Redis 7.2+** | RDB                              | Tiếp tục tối ưu performance của persistence |

**Khác biệt hành vi quan trọng**:

- **Memory sử dụng bởi AOF rewrite**: trước Redis 7.0, dữ liệu tăng thêm trong thời gian rewrite cần được giữ trong memory; từ 7.0+ vấn đề này được giải quyết bằng Multi-Part AOF.
- **Persistence hỗn hợp**: Redis 4.0-6.x cần bật thủ công, Redis 7.0+ bật mặc định.

Kiểm tra phiên bản Redis:

```bash
redis-cli INFO server | grep redis_version
# Ví dụ output: redis_version:7.0.12
```

Hình dưới đây thể hiện quy trình đầy đủ của cơ chế persistence Redis, bao gồm nội dung cốt lõi của bài viết:

![Quy trình đầy đủ của cơ chế persistence Redis](https://oss.javaguide.cn/github/javaguide/database/redis/redis-persistence-flow.png)

## RDB persistence

### RDB persistence là gì?

Redis có thể tạo snapshot để lấy bản sao dữ liệu được lưu trong memory tại **một thời điểm nhất định**. Sau khi tạo snapshot, Redis có thể backup snapshot, copy snapshot sang server khác để tạo replica có cùng dữ liệu (cấu trúc master-replica của Redis, chủ yếu dùng để nâng cao performance Redis), hoặc giữ snapshot tại chỗ để sử dụng khi restart server.

Snapshot persistence là phương thức persistence mặc định của Redis. Trong file cấu hình `redis.conf` mặc định có cấu hình sau:

```clojure
# Cấu hình mặc định của Redis 7.0 (dạng một dòng)
save 3600 1 300 100 60 10000

# Ý nghĩa từng điều kiện:
# - Trong 3600 giây (1 giờ) có ít nhất 1 key thay đổi
# - Trong 300 giây (5 phút) có ít nhất 100 key thay đổi
# - Trong 60 giây (1 phút) có ít nhất 10000 key thay đổi

# Tương đương dạng nhiều dòng ở phiên bản cũ:
# save 3600 1
# save 300 100
# save 60 10000
```

### Tạo RDB snapshot có block main thread không?

Redis cung cấp hai command để tạo file RDB snapshot:

- `save`: thao tác save đồng bộ, sẽ block main thread của Redis;
- `bgsave`: fork một child process để thực thi.

> Ở đây nói main thread của Redis thay vì main process chủ yếu vì sau khi khởi động, Redis hoàn thành công việc chính bằng cách single-thread. Nếu mô tả là main process của Redis cũng không sai.

#### Phân tích chi phí performance của fork

Mặc dù `bgsave` được thực thi trong child process và không block main thread xử lý command request, nhưng **bản thân thao tác fork là blocking**, đồng thời tạo thêm memory overhead (các giá trị trong bảng dưới chỉ mang tính tham khảo; số thực tế chịu ảnh hưởng của performance CPU, mức fragmentation của memory, system load và các yếu tố khác):

| Kích thước dataset | Độ trễ fork | Memory sử dụng thêm     | Mức độ rủi ro |
| ------------------ | ----------- | ----------------------- | ------------- |
| < 1GB              | < 10ms      | ~10MB (copy page table) | Thấp          |
| 1-10GB             | 10-100ms    | 10-100MB                | Trung bình    |
| 10-50GB            | 100ms-1s    | 100-500MB               | Cao           |
| > 50GB             | > 1s        | > 500MB                 | Rất cao       |

> Bài viết dùng `bgsave` của RDB để giải thích ảnh hưởng performance của fork, nhưng **cơ chế tương tự cũng áp dụng cho AOF rewrite (command `BGREWRITEAOF`)**. AOF rewrite cũng cần fork child process, đồng thời đối mặt với độ trễ fork, COW memory overhead và rủi ro THP. Trong môi trường production, dù là RDB hay AOF rewrite, đều cần chú ý các performance metric liên quan đến fork.

#### Cơ chế Copy-on-Write (COW)

- Sau fork, child process chia sẻ memory page của parent process (page chuẩn 4KB)
- Khi parent process hoặc child process sửa memory page, kernel copy page đó (Copy-on-Write)
- Dataset lớn + write load cao sẽ dẫn tới copy nhiều page, ảnh hưởng performance

#### Vấn đề memory avalanche do THP (Transparent Huge Pages)

Các Linux distribution mặc định bật **THP (Transparent Huge Pages, huge page trong suốt)** với kích thước 2MB. THP làm tăng xác suất huge page bị COW, **trong trường hợp xấu nhất**, nếu memory được gộp thành huge page 2MB, dù client chỉ sửa 10 byte dữ liệu, kernel vẫn copy toàn bộ memory page 2MB, khiến COW memory overhead **phóng đại 512 lần** (2MB / 4KB = 512).

**Hành vi thực tế**: kernel không bắt buộc toàn bộ memory sử dụng huge page 2MB, mà sẽ quyết định động có gộp hay không tùy tình huống. Chỉ sau khi THP gộp thành công thành huge page, thao tác sửa mới trigger COW 2MB. Tuy nhiên, trong tình huống tải ghi đồng thời cao, điều này vẫn làm tăng đáng kể memory consumption, có thể làm cạn memory của host trong chốc lát và trigger **OOM Killer buộc dừng process Redis**.

**Cách kiểm tra**:

```bash
cat /sys/kernel/mm/transparent_hugepage/enabled
# Output [always] madvise never cho biết đã bật (nguy hiểm!)
# Nên output always madvise [never]
```

**Giải pháp**: thêm `echo never > /sys/kernel/mm/transparent_hugepage/enabled` vào Redis startup script, hoặc dùng `redis-server --disable-thp yes` (Redis 6.0+ hỗ trợ).

**Cảnh báo khi khởi động**: khi Redis phát hiện THP đang bật, startup log sẽ ghi `WARNING you have Transparent Huge Pages (THP) support enabled in your kernel`, cần xử lý ngay.

#### Khuyến nghị cho môi trường production

```bash
# 1. Monitor metric rủi ro fork
redis-cli INFO memory | grep -E "(used_memory|used_memory_rss)"

# Ví dụ output:
# used_memory:1073741824
# used_memory_rss:1226833920
# used_memory_rss_human:1.14G

# Tính tỷ lệ RSS/USED, khi fork nên < 2
# Nếu gần hoặc vượt 2, nghĩa là rủi ro fork cao

# 2. Đặt giới hạn maxmemory để giới hạn memory Redis, dành chỗ cho fork
# Đặt trong redis.conf:
# maxmemory 8gb
# maxmemory-policy allkeys-lru

# 3. Tránh trigger BGSAVE thủ công vào giờ cao điểm
# Để Redis tự trigger theo rule cấu hình

# 4. Cân nhắc kiến trúc master-replica + persistence trên replica
# Chuyển thao tác persistence sang replica để tránh fork overhead trên master
```

**Cảnh báo giám sát**:

- `rdb_last_bgsave_time_sec`: thời gian bgsave lần trước, nên < 5s
- `rdb_last_cow_size`: kích thước COW memory của fork lần trước, nên < 10% `used_memory`

## AOF persistence

### AOF persistence là gì?

So với snapshot persistence, AOF persistence có tính real-time tốt hơn. Mặc định Redis chưa bật persistence theo phương thức AOF (append only file), có thể bật bằng parameter `appendonly`:

> **Mô tả phiên bản**: Redis mặc định dùng RDB persistence. Nếu cần dùng AOF, phải đặt thủ công `appendonly yes`. Redis 7.0 đưa vào cơ chế Multi-Part AOF để tối ưu AOF performance, nhưng không thay đổi phương thức persistence mặc định.

```bash
appendonly yes
```

Sau khi bật AOF persistence, mỗi khi thực thi một command làm thay đổi dữ liệu trong Redis, Redis sẽ ghi command đó vào AOF buffer `server.aof_buf`, sau đó ghi vào AOF file (lúc này vẫn nằm trong system kernel buffer, chưa sync xuống disk), cuối cùng dựa vào cấu hình `fsync` để quyết định khi nào đồng bộ dữ liệu trong system kernel buffer xuống disk.

Chỉ khi đã sync xuống disk mới được xem là persistence hoàn tất; nếu không vẫn tồn tại rủi ro mất dữ liệu. Chẳng hạn dữ liệu trong system kernel buffer chưa sync mà máy lưu trữ đã crash, phần dữ liệu đó sẽ bị mất.

Vị trí lưu AOF file giống vị trí RDB file, đều được đặt bằng parameter `dir`; tên file mặc định là `appendonly.aof`.

### Quy trình cơ bản của AOF là gì?

Chức năng AOF persistence có thể chia đơn giản thành 5 bước:

1. **Append command (append)**: mọi write command được nối vào AOF buffer.
2. **Ghi file (write)**: ghi dữ liệu trong AOF buffer vào AOF file. Bước này cần gọi function `write` (system call); sau khi `write` ghi dữ liệu vào system kernel buffer thì return trực tiếp (delayed write). Chú ý!!! Lúc này dữ liệu chưa được sync xuống disk.
3. **Sync file (fsync)**: đây mới là phần cốt lõi của persistence! Dựa trên strategy `appendfsync` trong file `redis.conf`, Redis gọi function `fsync` (system call) vào các thời điểm khác nhau. `fsync` thao tác trên một file, force sync file xuống disk (ghi dữ liệu trong kernel buffer của file vào disk), `fsync` block cho tới khi ghi xong disk mới return, bảo đảm persistence hoàn tất.
4. **Rewrite file (rewrite)**: khi AOF file ngày càng lớn, cần định kỳ rewrite AOF file để nén.
5. **Load khi restart (load)**: khi Redis restart, có thể load AOF file để khôi phục dữ liệu.

> Linux cung cấp trực tiếp một số function để truy cập và điều khiển file, device; các function này được gọi là **system call (syscall)**.

Giải thích lại một số Linux system call được nhắc tới ở trên:

- `write`: sau khi ghi vào system kernel buffer thì return trực tiếp (chỉ ghi vào buffer), không sync ngay xuống disk. Dù nâng cao efficiency nhưng cũng tạo rủi ro mất dữ liệu. **Thao tác sync disk phụ thuộc vào dirty page writeback strategy của Linux kernel (Dirty Page Writeback)**, chủ yếu chịu ảnh hưởng của các parameter sau:
  - `/proc/sys/vm/dirty_expire_centisecs`: thời gian dirty page hết hạn (mặc định 30 giây)
  - `/proc/sys/vm/dirty_writeback_centisecs`: chu kỳ đánh thức của kernel writeback thread (mặc định 5 giây)
  - System memory pressure: khi thiếu memory, sync sẽ được trigger tích cực hơn
- **Điều này nghĩa là trong mode `appendfsync no`, khi crash lượng dữ liệu có thể mất là không kiểm soát và không thể dự đoán**, phụ thuộc thời điểm kernel sync lần trước.
- `fsync`: `fsync` dùng để force flush system kernel buffer (sync xuống disk), bảo đảm chỉ return sau khi thao tác ghi disk kết thúc.

Sơ đồ flow AOF:

![Flow cơ bản của AOF](https://oss.javaguide.cn/github/javaguide/database/redis/aof-work-process.png)

### Có những phương thức AOF persistence nào?

Trong file cấu hình Redis có ba phương thức AOF persistence khác nhau (strategy `fsync`), lần lượt là:

1. `appendfsync always`: sau khi main thread gọi `write` để thực hiện thao tác ghi, lập tức gọi function `fsync` để sync AOF file (flush xuống disk); trong thời gian đó main thread block, chỉ return sau khi `fsync` ghi toàn bộ dữ liệu xuống disk. Strategy `always` do **main thread trực tiếp thực thi fsync**, không phải background thread. Cách này an toàn dữ liệu nhất, về lý thuyết không mất dữ liệu. Nhưng vì mỗi write operation đều sync và block main thread nên performance cực kỳ kém.
2. `appendfsync everysec`: sau khi main thread gọi `write` để ghi thì return ngay; background thread (`aof_fsync` thread) gọi function `fsync` (system call) mỗi giây một lần để sync AOF file (`write` + `fsync`, khoảng cách `fsync` là 1 giây). Cách này gần như không ảnh hưởng performance main thread. Đây là sự cân bằng tốt giữa performance và data safety. Tuy nhiên khi Redis crash bất thường, thông thường có thể mất dữ liệu trong 1 giây gần nhất.

> **Thực tế production (mất dữ liệu 2 giây và rủi ro block)**:
>
> "Tối đa mất 1 giây" là tình huống lý tưởng. Khi disk I/O bận, background fsync thực thi quá lâu, main thread sẽ kiểm tra thời điểm hoàn tất fsync lần trước trong lúc thực thi write command. Nếu thời gian từ lần fsync thành công trước đó vượt quá 2 giây, main thread sẽ bị **force block** để bảo vệ memory không bị đầy (logic kiểm tra block `aof_background_fsync` trong source code Redis `aof.c`).
>
> Vì vậy, **trong trường hợp crash cực đoan, có thể mất tối đa 2 giây dữ liệu**, đồng thời disk jitter sẽ trực tiếp làm P99 latency của Redis tăng vọt.
>
> **Metric bắt buộc phải theo dõi**: `redis-cli INFO persistence | grep aof_delayed_fsync` (ghi lại số lần tích lũy main thread bị fsync block; chỉ có field này khi đã bật AOF).

3. `appendfsync no`: sau khi main thread gọi `write` để ghi thì return ngay, để operating system quyết định khi nào sync; trên Linux thường là mỗi 30 giây (`write` nhưng không `fsync`, thời điểm `fsync` do operating system quyết định). Cách này có performance tốt nhất vì tránh block do `fsync`. Nhưng data safety kém nhất; khi crash lượng dữ liệu mất không kiểm soát, phụ thuộc thời điểm operating system sync lần trước.

Có thể thấy: **khác biệt chính của 3 phương thức persistence này nằm ở thời điểm `fsync` sync AOF file (flush xuống disk)**.

Để cân bằng data safety và write performance, có thể cân nhắc option `appendfsync everysec`, để Redis sync AOF file mỗi giây một lần, ảnh hưởng tới performance Redis tương đối nhỏ. Thông thường, ngay cả khi system crash, user nhiều nhất chỉ mất dữ liệu phát sinh trong một giây. Khi disk bận ghi, Redis cũng sẽ giảm tốc độ một cách phù hợp để thích ứng với write speed tối đa của disk.

> ⚠️ **Lưu ý**: khi disk I/O bottleneck nghiêm trọng, main thread Redis có thể block tới 2 giây do chờ fsync; trong thời gian đó data loss window tăng lên 2 giây. Môi trường production nên theo dõi metric `aof_delayed_fsync` để đánh giá disk health.

Từ Redis 7.0.0, Redis sử dụng cơ chế **Multi-Part AOF**. Như tên gọi, Multi-Part AOF chia AOF file ban đầu thành nhiều AOF file. Trong Multi-Part AOF, AOF file được chia thành ba loại:

- BASE: base AOF file, thường được child process tạo ra qua rewrite; tối đa chỉ có một file.
- INCR: incremental AOF file, thường được tạo khi AOFRW bắt đầu thực thi; có thể có nhiều file.
- HISTORY: history AOF file, được tạo từ sự thay đổi của BASE và INCR AOF. Mỗi khi AOFRW hoàn tất thành công, BASE và INCR tương ứng trước AOFRW lần này sẽ chuyển thành HISTORY; AOF loại HISTORY sẽ được Redis tự động xóa.

Multi-Part AOF không phải trọng tâm, chỉ cần hiểu; có thể xem bài [Thiết kế và triển khai Multi Part AOF của Redis 7.0](https://zhuanlan.zhihu.com/p/467217082) của Alibaba Developer để biết chi tiết.

**Issue liên quan**: [Phương thức AOF của Redis #783](https://github.com/Snailclimb/JavaGuide/issues/783).

### Tại sao AOF ghi log sau khi thực thi command?

Cơ sở dữ liệu quan hệ (chẳng hạn MySQL) thường ghi log trước khi thực thi command (thuận tiện cho việc recovery khi xảy ra lỗi), còn AOF persistence của Redis ghi log sau khi thực thi command.

![Quy trình ghi log của AOF](https://oss.javaguide.cn/github/javaguide/database/redis/redis-aof-write-log-disc.png)

**Tại sao ghi log sau khi thực thi command?**

- Tránh check overhead bổ sung, AOF log không syntax check command;
- Ghi sau khi command thực thi xong sẽ không block command execution hiện tại.

Cách này cũng mang đến rủi ro (phần giới thiệu AOF persistence phía trước cũng đã đề cập):

- Nếu Redis crash ngay sau khi thực thi command, thay đổi tương ứng có thể bị mất;
- Có thể block việc thực thi các command khác tiếp theo (AOF log được ghi trong main thread Redis).

### Bạn biết gì về AOF rewrite?

Khi AOF quá lớn, Redis có thể tự động rewrite AOF trong background để tạo AOF file mới. AOF file mới này có database state giống AOF file cũ nhưng kích thước nhỏ hơn.

![AOF rewrite](https://oss.javaguide.cn/github/javaguide/database/redis/aof-rewrite.png)

> AOF rewrite là một tên gọi dễ gây nhầm lẫn. Chức năng này được thực hiện bằng cách đọc key-value trong database; program không cần đọc, phân tích hay ghi AOF file hiện tại.

Vì AOF rewrite thực hiện rất nhiều write operation, để tránh ảnh hưởng việc Redis xử lý command request bình thường, Redis đặt AOF rewrite program vào child process.

Trong thời gian AOF file rewrite, Redis còn duy trì một **AOF rewrite buffer**. Buffer này ghi lại toàn bộ write command server thực thi trong thời gian child process tạo AOF file mới. Sau khi child process tạo xong AOF file mới, server append toàn bộ nội dung trong rewrite buffer vào cuối AOF file mới, khiến database state được lưu trong AOF file mới nhất quán với database state hiện tại. Cuối cùng server dùng AOF file mới thay thế file cũ, hoàn tất AOF file rewrite.

Để bật AOF rewrite, có thể gọi command `BGREWRITEAOF` để thực thi thủ công, hoặc đặt hai configuration item dưới đây để program tự quyết định thời điểm trigger:

- `auto-aof-rewrite-min-size`: nếu kích thước AOF file nhỏ hơn giá trị này thì không trigger AOF rewrite. Giá trị mặc định là 64 MB;
- `auto-aof-rewrite-percentage`: tỷ lệ giữa kích thước AOF hiện tại (`aof_current_size`) và kích thước AOF ở lần rewrite trước (`aof_base_size`) khi thực thi AOF rewrite. Nếu AOF file hiện tại tăng thêm tới tỷ lệ phần trăm này thì trigger AOF rewrite. Đặt giá trị này thành 0 sẽ disable AOF rewrite tự động. Giá trị mặc định là 100.

**Ranh giới thất bại và các tình huống rủi ro của AOF rewrite**:

Mặc dù AOF rewrite được thực thi trong child process, vẫn có các rủi ro sau cần biết:

| Scenario rủi ro          | Ảnh hưởng                            | Điều kiện trigger             | Biện pháp xử lý                                   |
| ------------------------ | ------------------------------------ | ----------------------------- | ------------------------------------------------- |
| **fork thất bại**        | Không tạo được rewrite child process | Thiếu memory, giới hạn system | Monitor memory usage, đặt `maxmemory`             |
| **Disk đầy**             | Ghi AOF file mới thất bại            | Data tăng nhanh trong rewrite | Monitor disk usage (`df -h`), alert threshold 70% |
| **Hết inode**            | Không tạo được file mới              | System có quá nhiều file nhỏ  | Monitor inode usage (`df -i`), dọn temporary file |
| **Timestamp quay ngược** | Quản lý Multi-Part AOF hỗn loạn      | Vấn đề đồng bộ clock của VM   | Cấu hình NTP service, đặt `aof-timestamp-enabled` |
| **SIGTERM signal**       | Rewrite bị interrupt                 | Operator restart thủ công     | Cấu hình graceful shutdown (`shutdown-timeout`)   |

**Khuyến nghị monitoring trong môi trường production**:

```bash
# Monitor trạng thái AOF rewrite
redis-cli INFO persistence | grep aof_rewrite_in_progress

# Monitor tốc độ tăng kích thước AOF file
redis-cli INFO persistence | grep aof_current_size
redis-cli INFO persistence | grep aof_base_size

# Kiểm tra disk và inode usage
df -h /var/lib/redis
df -i /var/lib/redis

# Đặt incremental fsync strategy trong thời gian AOF rewrite (Redis 7.0+)
# aof-rewrite-incremental-sync yes
```

Trước Redis 7.0, nếu có write command trong thời gian rewrite, AOF có thể sử dụng rất nhiều memory; toàn bộ write command đến trong thời gian rewrite sẽ được ghi xuống disk hai lần.

Sau Redis 7.0, cơ chế AOF rewrite đã được tối ưu. Phần nội dung dưới đây trích từ bài [Nhìn về quá khứ và tương lai của Redis qua việc phát hành Redis 7.0](https://mp.weixin.qq.com/s/RnoPPL7jiFSKkx3G4p57Pg) của Alibaba Developer.

> Cách xử lý incremental data trong AOF rewrite luôn là một vấn đề. Trước đây, incremental data phát sinh trong thời gian ghi phải được giữ trong memory; sau khi ghi xong mới ghi phần incremental data này vào AOF file mới để bảo đảm data integrity. Có thể thấy AOF write tiêu tốn thêm memory và disk I/O; đây cũng là điểm khó của Redis AOF write. Mặc dù trước đó đã có nhiều lần cải tiến, vấn đề cốt lõi về resource consumption vẫn chưa được giải quyết.
>
> Redis Enterprise Edition của Alibaba Cloud ban đầu cũng gặp vấn đề này. Sau nhiều vòng iteration development nội bộ, họ đã triển khai cơ chế Multi-part AOF để giải quyết, đồng thời đóng góp cho community và phát hành cùng phiên bản 7.0 lần này. Cụ thể, dùng các file độc lập để lưu base (full data) + inc (incremental data), giải quyết triệt để việc lãng phí memory và I/O resource, đồng thời hỗ trợ lưu trữ và quản lý AOF file lịch sử. Kết hợp với thông tin time trong AOF file còn có thể thực hiện PITR khôi phục theo thời điểm (Alibaba Cloud Enterprise Edition Tair đã hỗ trợ), giúp tăng thêm data reliability của Redis, đáp ứng nhu cầu rollback dữ liệu của user.

**Issue liên quan**: [Mô tả AOF rewrite không chính xác #1439](https://github.com/Snailclimb/JavaGuide/issues/1439).

### Làm thế nào để kiểm tra data integrity của AOF file?

**Kết luận cốt lõi**: AOF file thuần **không có checksum**, chỉ verify bằng cách parse từng command; CRC64 checksum chỉ tồn tại trong **phần RDB** của hybrid persistence file.

#### Pure AOF mode: không checksum, chỉ parse syntax

Pure AOF file không tính CRC64 checksum cho toàn file hay từng command, mà verify validity bằng cách parse lần lượt các command trong file.

**Tại sao không có checksum?**

AOF là text log được append với tần suất cao. Nếu mỗi lần append command đều phải tính lại CRC64 checksum của toàn file, CPU và disk I/O của main thread sẽ bị ảnh hưởng nghiêm trọng. Vì vậy Redis chọn cách nhẹ hơn: khi load lúc restart, lần lượt đọc và parse command syntax.

Nếu phát hiện syntax error trong quá trình parse (chẳng hạn command không đầy đủ, format sai), Redis sẽ terminate việc load và báo error.

> **Disaster recovery khi file bị truncate ở cuối (tự động recovery)**:
>
> Khi mất điện đột ngột hoặc bị terminate bằng `kill -9`, command cuối cùng của AOF file rất dễ ghi không đầy đủ (chỉ ghi một nửa). Hành vi recovery lúc này do configuration `aof-load-truncated` quyết định:
>
> | Giá trị config   | Hành vi                                                                                                      | Tình huống phù hợp                                                          |
> | ---------------- | ------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------- |
> | `yes` (mặc định) | Redis tự động bỏ command không đầy đủ ở cuối file, tiếp tục startup và in warning trong log                  | Khuyến nghị cho production, cho phép mất ít dữ liệu để đổi lấy availability |
> | `no`             | Redis từ chối startup và báo error ngay, buộc phải dùng tool `redis-check-aof` để xác nhận và repair dữ liệu | Tình huống như tài chính yêu cầu data integrity rất cao                     |
>
> **Kiểm tra khả năng recovery khi bị truncate**:
>
> ```bash
> # Mô phỏng scenario mất điện: append garbage data vô nghĩa vào AOF file
> echo "truncated garbage data" >> /var/lib/redis/appendonly.aof
>
> # Restart Redis (khi aof-load-truncated=yes sẽ tự động recovery)
> redis-server /path/to/redis.conf
> # Log output: # Bad file format reading the append only file: make a backup of your AOF file, then use ./redis-check-aof --fix <filename>
> ```

**Failure mode**: nếu phần giữa AOF file (không phải phần cuối) bị ghi garbage do disk silent corruption, cơ chế tự động truncate không có tác dụng; Redis sẽ crash trực tiếp và từ chối phục vụ. Khi đó cần dùng tool `redis-check-aof --fix` để repair.

**Nguyên lý hoạt động của `redis-check-aof`**:

- **Giai đoạn detect**: lần lượt đọc command theo AOF file format, kiểm tra số lượng command argument, độ dài argument string và các thông tin khác, cung cấp vị trí file của command lỗi/không đầy đủ
- **Giai đoạn repair**: truncate nội dung file phía sau từ vị trí lỗi (**lưu ý: toàn bộ dữ liệu sau vị trí truncate sẽ mất**), file gốc được backup thành `appendonly.aof.broken`

#### Hybrid persistence mode: strategy checksum theo từng phần

Trong **hybrid persistence mode** (được đưa vào từ Redis 4.0), AOF file dùng strategy checksum "theo từng phần":

```
┌─────────────────────────────────────────────────────────┐
│              Cấu trúc hybrid persistence file           │
├─────────────────────────────────────────────────────────┤
│  Phần RDB snapshot (binary) ← CRC64 checksum bảo vệ phần này │
│  ├── Header "REDIS"                                     │
│  ├── Database number, key-value...                      │
│  ├── EOF flag                                            │
│  └── CRC64 checksum (8 byte) ← ranh giới checksum ở đây │
├─────────────────────────────────────────────────────────┤
│  Phần AOF incremental (text) ← không checksum, chỉ parse syntax │
│  ├── *3\r\n$3\r\nSET\r\n...                              │
│  └── ...                                                  │
└─────────────────────────────────────────────────────────┘
```

- **Phần RDB snapshot**: bắt đầu bằng chuỗi cố định `REDIS`, lưu snapshot dữ liệu memory tại một thời điểm, kèm CRC64 checksum ở cuối snapshot data. Checksum này **nằm chính xác ở cuối RDB data block**, chỉ bảo đảm integrity của binary snapshot này.
- **Phần AOF incremental**: nối ngay sau RDB snapshot, ghi lại incremental write command. Phần này **vẫn không có checksum**, dùng cách parse syntax từng command giống pure AOF.

**Quy trình kiểm tra khi load**:

1. Redis trước hết verify phần RDB snapshot: tính CRC64 checksum của data phần này rồi so sánh với checksum đã lưu. Nếu không khớp, Redis từ chối startup.
2. Sau khi verify phần RDB thành công, lần lượt parse AOF incremental command. Nếu parse lỗi thì dừng load các command tiếp theo (nhưng lúc này RDB snapshot data đã load thành công).

#### Mô tả configuration item

| Configuration item   | Scope                                                      | Mô tả                                                                                  |
| -------------------- | ---------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| `rdbchecksum`        | RDB file, phần RDB của hybrid persistence                  | Kiểm soát có tính CRC64 checksum hay không; không có tác dụng với pure AOF incremental |
| `aof-load-truncated` | Pure AOF file, phần AOF incremental của hybrid persistence | Kiểm soát có tự động bỏ phần truncate ở cuối và tiếp tục startup hay không             |

**Manual patch** (dành cho advanced user):

- Nếu không muốn repair AOF file bằng cách truncate, có thể thử manual patch
- Dùng text editor mở AOF file (plain text), thủ công xóa hoặc sửa command lỗi
- Phù hợp với tình huống cụ thể khi đã biết rõ vị trí lỗi

## Tối ưu trong phiên bản mới

### Redis 4.0 đã tối ưu cơ chế persistence như thế nào?

Vì RDB và AOF đều có ưu điểm riêng, từ Redis 4.0, Redis bắt đầu hỗ trợ hybrid persistence giữa RDB và AOF.

#### Mô tả cấu hình

```bash
# Bật AOF
appendonly yes

# Bật hybrid persistence (Redis 7.0+ bật mặc định)
aof-use-rdb-preamble yes

# Tối ưu điều kiện trigger rewrite
auto-aof-rewrite-percentage 100   # Trigger khi kích thước AOF file tăng 100% so với sau lần rewrite trước
auto-aof-rewrite-min-size 64mb    # Chỉ trigger rewrite khi AOF file đạt ít nhất 64MB
```

**Khác biệt giữa các phiên bản**:

- **Redis 4.0-6.x**: hybrid persistence tắt mặc định, cần cấu hình thủ công `aof-use-rdb-preamble yes`
- **Redis 7.0+**: hybrid persistence **bật mặc định**, không cần config thêm

#### Nguyên lý hoạt động

Nếu bật hybrid persistence, khi AOF rewrite, Redis ghi trực tiếp dữ liệu RDB vào đầu AOF file. Ưu điểm là kết hợp được ưu điểm của RDB và AOF, load nhanh đồng thời tránh mất quá nhiều dữ liệu.

**Cấu trúc hybrid persistence file**:

```
┌───────────────────┐
│   RDB Header      │ ← Binary snapshot (format nén)
│   REDIS0009       │
│   ...             │
├───────────────────┤
│   AOF Log Entries │ ← Command format text
│   *3\r\n$3\r\nSET\r\n$5\r\nkey01\r\n...
│   INCR counter    │
│   ...             │
└───────────────────┘
```

**Quy trình hoạt động cốt lõi**:

1. **Giai đoạn xử lý write**:

   - Client thực thi write command (`SET/INCR` và các command khác)
   - Redis lập tức update memory data
   - Append command vào AOF buffer (text format)

2. **Giai đoạn trigger persistence**:

   - AOF file đạt threshold (mặc định 64MB) hoặc tăng 100%
   - Trigger AOF rewrite (`BGREWRITEAOF`)

3. **Giai đoạn build file**:

   - Child process ghi memory data hiện tại vào đầu AOF file mới theo RDB format
   - Parent process tiếp tục xử lý write command, incremental data được ghi vào rewrite buffer
   - Sau khi rewrite xong, append incremental command trong rewrite buffer vào cuối AOF file mới

4. **Giai đoạn recovery data**:
   - Khi Redis startup, ưu tiên load phần RDB (nhanh chóng recovery dữ liệu cơ sở)
   - Sau đó lần lượt replay AOF incremental command (recovery dữ liệu mới nhất)

#### So sánh

| Metric               | Pure RDB               | Pure AOF              | Hybrid persistence     |
| -------------------- | ---------------------- | --------------------- | ---------------------- |
| **Recovery speed**   | Nhanh (tính bằng giây) | Chậm (tính bằng phút) | Nhanh (tính bằng giây) |
| **Data loss window** | Tính bằng phút         | ≤2 giây               | ≤2 giây                |
| **File size**        | Nhỏ (nén)              | Lớn (text log)        | Trung bình             |
| **Ảnh hưởng write**  | Thấp                   | Cao                   | Trung bình             |
| **Readability**      | Kém (binary)           | Tốt (text)            | Kém (phần RDB)         |

**Benchmark data** (dataset 1GB, SSD):

- Pure AOF recovery: 30-60 giây
- Hybrid persistence recovery: 2-5 giây (**nhanh hơn 5-10 lần**)

**Nhược điểm của hybrid persistence**:

- Phần RDB trong AOF file là định dạng nén, không còn là AOF format nên readability kém.
- Cần thêm CPU để compress và decompress RDB.

#### Vấn đề thường gặp và giải pháp

**1. Kiểm tra cấu hình**:

```bash
# Cách 1: kiểm tra file header (output REDIS cho biết đã bật hybrid persistence)
head -c 5 appendonly.aof

# Cách 2: kiểm tra bằng CLI
redis-cli CONFIG GET aof-use-rdb-preamble
# Output: 1) "aof-use-rdb-preamble"
#         2) "yes"
```

**2. Recovery khi file bị hỏng**:

**Mô tả tool**:

| Tool                | Nguyên lý hoạt động                                                                        | Detect error                                                       | Chức năng repair                                                             |
| ------------------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------ | ---------------------------------------------------------------------------- |
| **redis-check-aof** | Lần lượt đọc command theo AOF file format, kiểm tra số argument, độ dài argument string... | Kiểm tra correctness và integrity của command, cung cấp vị trí lỗi | ✅ **Hỗ trợ repair**: truncate content phía sau vị trí lỗi hoặc manual patch |
| **redis-check-rdb** | Lần lượt đọc file header, data và file footer theo RDB file format                         | Kiểm tra correctness trong lúc đọc và báo error                    | ❌ **Không hỗ trợ repair**: chỉ detect vấn đề, cần manual repair             |

**Các bước recovery**:

```bash
# Bước 1: detect vấn đề AOF file
redis-check-aof appendonly.aof
# Output vị trí và nguyên nhân lỗi

# Bước 2: repair AOF file (truncate từ vị trí lỗi)
redis-check-aof --fix appendonly.aof
# AOF file gốc được backup thành appendonly.aof.broken

# Bước 3: kiểm tra phần RDB
redis-check-rdb appendonly.aof
# Chỉ detect, không hỗ trợ parameter --fix

# Bước 4: nếu phần RDB có vấn đề, cần manual repair hoặc bỏ toàn bộ file
# Option A: manual repair (cần hiểu RDB binary format)
# Option B: xóa hybrid persistence file, chỉ dùng pure RDB hoặc pure AOF để recovery

# Bước 5: startup Redis
redis-server --appendonly yes --appendfilename appendonly.aof
```

> **⚠️ Lưu ý quan trọng**:
>
> - **AOF file**: `redis-check-aof --fix` sẽ truncate file từ vị trí lỗi, **mất toàn bộ dữ liệu sau vị trí truncate**
> - **RDB file**: `redis-check-rdb` **không hỗ trợ repair**; nếu phần RDB bị hỏng, toàn bộ hybrid persistence file không thể recovery, chỉ có thể dựa vào backup hoặc pure AOF file
> - **Manual repair**: nếu bắt buộc repair phần RDB, cần dùng hex editor (chẳng hạn `hexdump`, `xxd`) để sửa binary format thủ công

#### Khuyến nghị production config

```bash
# Ví dụ production config đầy đủ
appendonly yes
aof-use-rdb-preamble yes

# Tối ưu performance
aof-rewrite-incremental-fsync yes   # Incremental fsync, giảm peak disk I/O
# Scenario nhạy với latency (khuyến nghị yes)
no-appendfsync-on-rewrite yes       # Tạm dừng fsync trong rewrite để tránh block
# Scenario ưu tiên data safety (khuyến nghị no)
no-appendfsync-on-rewrite no        # Vẫn thực thi fsync trong rewrite, có thể block nhưng an toàn hơn

# Khuyến nghị capacity planning:
# - Dành 2x memory làm disk space
# - Giữ một AOF file < 16GB
# - Monitor metric aof_delayed_fsync
```

Địa chỉ tài liệu chính thức: <https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/>

![](https://oss.javaguide.cn/github/javaguide/database/redis/redis4.0-persitence.png)

### Redis 7.0 đã tối ưu cơ chế persistence như thế nào?

Vì trong quá trình AOF rewrite tồn tại vấn đề incremental data được buffer trong memory và double write xuống disk, từ Redis 7.0, Redis bắt đầu hỗ trợ Multi-Part AOF (bật mặc định, có thể chỉ định directory bằng configuration item `appenddirname`).

Nếu bật Multi-Part AOF, AOF file được chia thành base file (tối đa một file, initial full snapshot, có thể ở RDB hoặc AOF format) và nhiều incr file (incremental command log), mọi part được manifest file track. Ưu điểm là loại bỏ memory buffer overhead và double I/O write trong rewrite, nâng cao performance và giảm nguy cơ fsync block. Do file structure tách biệt, INCR file giữ read-only trước rewrite, copy từng file tương đối an toàn; nhưng backup cross-file vẫn cần pause rewrite, flow backup tổng thể phức tạp hơn single-file AOF, đồng thời với dataset cực lớn vẫn có thể cần monitor resource.

> **Rủi ro single point of failure cốt lõi: manifest file bị hỏng**
>
> Multi-Part AOF phụ thuộc **manifest file** để track và quản lý mọi `base/incr/history` file; đây là core metadata của toàn bộ incremental log system. Nếu manifest file bị hỏng hoặc mất:
>
> | Scenario rủi ro                  | Ảnh hưởng                                                                                     | Độ khó recovery                         |
> | -------------------------------- | --------------------------------------------------------------------------------------------- | --------------------------------------- |
> | **Manifest silent corruption**   | Redis không thể nhận diện và load AOF file chính xác khi startup, database không thể recovery | Rất cao (cần rebuild manifest thủ công) |
> | **Manifest mất do disk failure** | Dù base/incr file còn đầy đủ, Redis cũng không thể reconstruct file dependency                | Rất cao (cần can thiệp thủ công)        |
>
> **Biện pháp giảm thiểu**:
>
> ```bash
> # 1. Backup manifest file (quan trọng ngang data file)
> cp /var/lib/redis/appendonlydir/appendonly.aof.manifest /backup/
>
> # 2. Monitor disk health (phát hiện failure sớm)
> smartctl -a /dev/sda | grep -E "SMART overall-health self-assessment|Media_Errors"
>
> # 3. Định kỳ verify manifest integrity (Redis tự động check khi startup)
> redis-check-aof /var/lib/redis/appendonlydir/appendonly.aof.manifest
> ```
>
> **Redis chưa cung cấp tool repair tự động**, môi trường production bắt buộc đưa manifest file vào backup strategy; mức độ quan trọng tương đương chính RDB/AOF data file.

## Các metric giám sát môi trường production

### Persistence performance metric

```bash
# Metric liên quan đến RDB
redis-cli INFO persistence | grep rdb_last_bgsave_time_sec
# Khuyến nghị: < 5s. Vượt 5s cho biết dataset quá lớn hoặc disk I/O bottleneck

redis-cli INFO persistence | grep rdb_last_cow_size
# Khuyến nghị: < 10% used_memory. Vượt giá trị này cho biết COW memory overhead của fork lớn

redis-cli INFO memory | grep used_memory_rss
redis-cli INFO memory | grep used_memory
# Tính: used_memory_rss / used_memory, khi fork nên < 2

# Metric liên quan đến AOF
redis-cli INFO persistence | grep aof_rewrite_in_progress
# Kỳ vọng: 0 (không rewrite) hoặc 1 (đang rewrite)

redis-cli INFO persistence | grep aof_current_size
redis-cli INFO persistence | grep aof_base_size
# Monitor growth rate, tránh rewrite quá thường xuyên

redis-cli INFO persistence | grep aof_buffer_length
# Khuyến nghị: < 4MB. Quá lớn nghĩa là write speed của main thread nhanh hơn fsync speed
```

### System resource monitoring

```bash
# Disk usage và I/O wait
iostat -x 1 5 | grep dm-0
# Chú ý: %util (I/O usage), await (average wait time)

# Disk space (dành space để rewrite tạo file mới)
df -h /var/lib/redis
# Khuyến nghị: usage < 70%

# Inode usage (scenario có nhiều file nhỏ)
df -i /var/lib/redis
# Khuyến nghị: usage < 90%

# Memory usage
free -h
# Khuyến nghị: dành ít nhất 20% free memory cho fork
```

### Khuyến nghị alert rule

> **Mô tả nguồn metric**:
>
> - **Redis metric**: lấy qua `redis-cli INFO` hoặc Redis exporter (chẳng hạn `redis_rss_memory`, `aof_current_size`)
> - **Node-level metric**: lấy qua node_exporter hoặc system command (chẳng hạn `disk_usage`, system memory, CPU usage)
>
> Các alert rule dưới đây giả định dùng monitoring stack Prometheus + Redis exporter + node_exporter.

```yaml
alert_rules:
  # ── Redis persistence alert liên quan ─────────────────────────────
  - name: "RedisHighMemFragmentation"
    expr: redis_memory_rss_bytes / redis_memory_used_bytes > 2
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Redis memory fragmentation rate quá cao, rủi ro fork COW tăng"
      description: >
        mem_fragmentation_ratio của instance {{ $labels.instance }} = {{ $value | humanize }},
        vượt threshold 2. Fragmentation rate quá cao nghĩa là số physical page OS thực tế cấp phát nhiều hơn đáng kể so với thống kê của Redis,
        sau khi BGSAVE / BGREWRITEAOF trigger fork, số page cần COW copy sẽ tăng đáng kể,
        trong write load cao có thể khiến memory tăng vọt, rủi ro OOM tăng.
        Khuyến nghị thực thi MEMORY PURGE hoặc restart instance vào giờ thấp điểm để giảm fragmentation.

  - name: "RedisAofGrowthTooFast"
    expr: deriv(redis_aof_current_size_bytes[5m]) * 60 > 10485760
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Tốc độ ghi Redis AOF file quá cao"
      description: >
        Tốc độ tăng AOF của instance {{ $labels.instance }} vượt 10 MB/min
        (hiện khoảng {{ $value | humanize1024 }}B/min).
        Write speed cao sẽ liên tục trigger auto-aof-rewrite, làm tăng disk I/O pressure,
        đồng thời có thể tạo write amplification. Khuyến nghị kiểm tra business có tồn tại quá nhiều small command storm hoặc full scan kiểu KEYS hay không.

  - name: "RedisAofFsyncDelayed"
    expr: rate(redis_aof_delayed_fsync_total[5m]) > 0
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: "Redis AOF fsync bị delay, main thread response bị cản trở"
      description: >
        Instance {{ $labels.instance }} liên tục có aof_delayed_fsync tăng,
        main thread bị block do chờ AOF fsync hoàn tất, trực tiếp làm P99 command response kém đi.
        Nguyên nhân thường gặp: ① disk I/O bandwidth bão hòa; ② appendfsync đặt là always;
        ③ dùng chung disk với process I/O cao khác. Khuyến nghị chuyển sang everysec hoặc migrate sang disk độc lập.

  # ── Node-level resource alert ────────────────────────────────────
  - name: "RedisDiskUsageHigh"
    expr: >
      (1 - node_filesystem_avail_bytes{mountpoint="/var/lib/redis"}
         / node_filesystem_size_bytes{mountpoint="/var/lib/redis"}) * 100 > 70
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Redis data disk usage vượt 70%"
      description: >
        Mount point /var/lib/redis hiện có usage {{ $value | humanize }}%.
        Trong AOF rewrite sẽ tạm thời tạo file mới, cần dành khoảng 1.5x AOF size hiện tại,
        thiếu disk sẽ khiến rewrite thất bại và trigger Redis error log "MISCONF".
        RDB bgsave cũng tương tự.
      remediation: >
        1. Xóa RDB snapshot hết hạn và AOF file lịch sử;
        2. Tăng auto-aof-rewrite-min-size để giảm rewrite frequency;
        3. Mở rộng disk hoặc migrate data directory sang partition lớn hơn.
```

## Chọn RDB hay AOF như thế nào?

Về ưu nhược điểm của RDB và AOF, trên website chính thức cũng có phần mô tả khá chi tiết [Redis persistence](https://redis.io/docs/manual/persistence/). Dưới đây là phần tổng hợp ngắn gọn, kết hợp với cách hiểu cá nhân.

**Điểm RDB tốt hơn AOF**:

- **File compact, phù hợp backup và disaster recovery**: nội dung RDB file là binary data đã compressed, lưu dataset tại một thời điểm, file rất nhỏ, cực kỳ phù hợp để backup và disaster recovery. AOF file lưu từng write command, tương tự MySQL binlog log, thường lớn hơn RDB file rất nhiều. Khi AOF quá lớn, Redis có thể tự động rewrite AOF trong background; AOF file mới và AOF file cũ lưu cùng database state nhưng kích thước nhỏ hơn. Tuy nhiên trước Redis 7.0, nếu có write command trong thời gian rewrite, AOF có thể dùng nhiều memory; toàn bộ write command đến trong thời gian rewrite sẽ được ghi xuống disk hai lần.
- **Recovery speed nhanh**: dùng RDB file để recovery data chỉ cần trực tiếp parse và restore data, không cần thực thi từng command nên rất nhanh. AOF cần lần lượt thực thi từng write command nên rất chậm. Nói cách khác, khi recovery dataset lớn, RDB nhanh hơn AOF.
- **Ưu điểm trong master-replica replication**: trên replica, RDB hỗ trợ **partial resynchronization** sau restart và failover. Replica có thể dùng RDB snapshot để nhanh chóng sync tới state của master tại một thời điểm, không cần full sync.
- **Performance overhead nhỏ**: RDB tối đa hóa performance Redis, vì công việc persistence duy nhất parent process Redis cần làm là fork child process; child process hoàn thành toàn bộ công việc còn lại. Parent process không bao giờ thực hiện disk I/O hay thao tác tương tự.

**Điểm AOF tốt hơn RDB**:

- **Data safety cao hơn, hỗ trợ persistence cấp giây**: data safety của RDB không bằng AOF, không thể persistence data realtime hoặc theo giây. Quá trình tạo RDB file khá nặng; dù child process BGSAVE ghi RDB file không block main thread, nó vẫn ảnh hưởng CPU và memory resource của machine, trường hợp nghiêm trọng thậm chí có thể làm Redis service crash. AOF hỗ trợ data loss cấp giây (phụ thuộc `fsync` strategy; nếu là `everysec`, thông thường tối đa mất 1 giây data; nhưng khi disk I/O bận có thể mất 2 giây và main thread bị block), chỉ append command vào AOF file nên nhẹ hơn.
- **Version compatibility tốt**: RDB file được lưu theo binary format cụ thể, và có nhiều phiên bản RDB trong quá trình Redis phát triển, vì vậy Redis service phiên bản cũ có thể không tương thích với RDB format của phiên bản mới.
- **Readability và khả năng thao tác cao**: AOF chứa log của mọi operation theo format dễ hiểu và parse. Bạn có thể dễ dàng export AOF file để phân tích, cũng có thể trực tiếp thao tác AOF file để xử lý một số vấn đề. Chẳng hạn nếu vô tình thực thi command `FLUSHALL` làm xóa toàn bộ dữ liệu, chỉ cần AOF file chưa rewrite, xóa command mới nhất rồi restart là có thể recovery state trước đó.
- **Append log không có rủi ro corruption**: AOF log là append log, không seek, cũng không có vấn đề corruption do mất điện. Ngay cả khi log kết thúc bằng một command ghi dở dang vì lý do nào đó (disk đầy hoặc nguyên nhân khác), tool `redis-check-aof` vẫn có thể repair dễ dàng.

**Ảnh hưởng của sự phát triển phiên bản tới việc lựa chọn**:

| Phiên bản     | Cải tiến chính                                      | Ảnh hưởng tới AOF                                                             | Ý nghĩa với việc lựa chọn                                                                  |
| ------------- | --------------------------------------------------- | ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| **Redis 4.0** | Đưa vào hybrid persistence (`aof-use-rdb-preamble`) | Base file dùng RDB format khi AOF rewrite, recovery speed tăng 5-10 lần       | Giảm vấn đề pure AOF load chậm, nhưng vẫn cần chú ý memory và I/O overhead trong rewrite   |
| **Redis 7.0** | Đưa vào Multi-Part AOF                              | Loại bỏ triệt để double write trong rewrite, memory và I/O overhead giảm mạnh | Dùng AOF độc lập khả thi hơn trong production, nhưng vấn đề fork block vẫn chưa giải quyết |

**Vấn đề cốt lõi chưa giải quyết**:

- **Fork block**: dù là RDB bgsave hay AOF rewrite, bản thân fork đều block main thread (dataset càng lớn, thời gian block càng dài)
- **Khuyến nghị chính thức**: tài liệu chính thức Redis đến nay vẫn khuyến nghị **bật đồng thời RDB và AOF**; RDB làm cold backup bổ sung, ứng phó các scenario cực đoan như AOF file hỏng hoặc write error

**Tương tác giữa AOF và RDB**:

Khi đồng thời bật AOF và RDB persistence:

- **Tránh đồng thời thực hiện heavy I/O operation**: Redis 2.4+ bảo đảm tránh trigger AOF rewrite khi RDB snapshot đang thực hiện, hoặc cho phép BGSAVE trong thời gian AOF rewrite. Điều này ngăn hai Redis background process đồng thời thực hiện disk I/O nặng.
- **Lịch AOF rewrite**: khi snapshot đang thực hiện mà user explicit request log rewrite (dùng BGREWRITEAOF), server return OK status code, cho user biết operation đã được schedule; rewrite bắt đầu sau khi snapshot hoàn tất.
- **Ưu tiên recovery khi restart**: nếu đồng thời bật AOF và RDB persistence rồi Redis restart, **AOF file sẽ được dùng để rebuild original dataset**, vì nó được bảo đảm là đầy đủ nhất.

**Khuyến nghị lựa chọn**:

| Scenario                                         | Giải pháp đề xuất                                                                           | Mô tả                                                                                      |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| **Pure cache (có thể mất)**                      | **Tắt persistence** hoặc chỉ RDB (tần suất thấp)                                            | Tắt hoàn toàn có overhead nhỏ nhất; nếu cần cold backup thì giữ RDB tần suất thấp          |
| **Data importance trung bình** (session, config) | **RDB + AOF hybrid persistence** (Redis 4.0+)                                               | RDB tăng tốc recovery, AOF bổ sung incremental, `everysec` tối đa mất 1s                   |
| **Data importance cao** (business core data)     | **RDB + AOF (MP-AOF, Redis 7.0+)**, và Redis là cache layer chứ không phải storage duy nhất | MP-AOF giảm rewrite overhead; persistence thực sự do primary database (MySQL...) đảm nhiệm |
| **Master-replica architecture**                  | **Master tắt persistence, replica bật AOF**                                                 | Master cấm cấu hình auto restart, tránh empty dataset ghi đè replica                       |

## Tài liệu tham khảo

- 《Redis Design and Implementation》
- Redis persistence - Tài liệu chính thức Redis: <https://redis.io/docs/management/persistence/>
- The difference between AOF and RDB persistence: <https://www.sobyte.net/post/2022-04/redis-rdb-and-aof/>
- Giải thích chi tiết Redis AOF persistence - Li Xiaobing: <http://remcarpediem.net/article/376c55d8/>
- Redis RDB và AOF persistence · Analyze: <https://wingsxdu.com/posts/database/redis/rdb-and-aof/>

<!-- @include: @article-footer.snippet.md -->
