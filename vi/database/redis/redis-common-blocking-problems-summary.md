---
title: "Tổng hợp các nguyên nhân gây blocking thường gặp trong Redis"
description: "Tổng hợp toàn diện các nguyên nhân blocking thường gặp trong Redis, gồm lệnh có độ phức tạp O(n), thao tác với bigkey, ghi AOF xuống đĩa, tạo snapshot RDB, đồng bộ master-replica và các trường hợp khác, giúp bạn chẩn đoán và phòng tránh vấn đề performance của Redis."
category: Database
tag:
  - Redis
head:
  - - meta
    - name: keywords
      content: Redis blocking,Redis performance,O(n) command,bigkey,AOF fsync,RDB snapshot,master-replica synchronization,memory limit
---

> Bài viết được biên soạn và hoàn thiện dựa trên: <https://mp.weixin.qq.com/s/0Nqfq_eQrUb12QH6eBbHXA>, tác giả: A Q nói code

Bài viết tổng hợp chi tiết những trường hợp có thể khiến Redis bị blocking. Đây cũng là các yếu tố quan trọng ảnh hưởng đến performance của Redis, vì vậy bạn cần đặc biệt lưu ý khi sử dụng Redis!

## Lệnh O(n)

Phần lớn lệnh trong Redis có độ phức tạp thời gian O(1), nhưng cũng có một số ít lệnh có độ phức tạp thời gian O(n), chẳng hạn:

- `KEYS *`: trả về tất cả key khớp pattern.
- `HGETALL`: trả về toàn bộ cặp key-value trong một Hash.
- `LRANGE`: trả về các phần tử trong phạm vi chỉ định của List.
- `SMEMBERS`: trả về toàn bộ phần tử trong Set.
- `SINTER`/`SUNION`/`SDIFF`: tính intersection/union/difference của nhiều Set.
- …

Do các lệnh này có độ phức tạp thời gian O(n), đôi khi chúng cũng thực hiện full table scan. Khi n tăng, thời gian thực thi sẽ dài hơn, từ đó khiến client bị blocking. Tuy nhiên, không có nghĩa là không được sử dụng các lệnh này; bạn cần xác định rõ giá trị của N. Nếu cần duyệt, bạn có thể dùng `HSCAN`, `SSCAN`, `ZSCAN` để thay thế.

Ngoài các lệnh có độ phức tạp thời gian O(n) có thể gây blocking, còn có một số lệnh có độ phức tạp thời gian từ O(N) trở lên, chẳng hạn:

- `ZRANGE`/`ZREVRANGE`: trả về toàn bộ phần tử trong phạm vi rank được chỉ định của Sorted Set. Độ phức tạp thời gian là O(log(n)+m), trong đó n là số lượng toàn bộ phần tử, m là số lượng phần tử được trả về. Khi m và n đều rất lớn, độ phức tạp thời gian O(n) sẽ nhỏ hơn.
- `ZREMRANGEBYRANK`/`ZREMRANGEBYSCORE`: xóa toàn bộ phần tử trong phạm vi rank/phạm vi score được chỉ định của Sorted Set. Độ phức tạp thời gian là O(log(n)+m), trong đó n là số lượng toàn bộ phần tử, m là số lượng phần tử bị xóa. Khi m và n đều rất lớn, độ phức tạp thời gian O(n) sẽ nhỏ hơn.
- …

## `SAVE` tạo snapshot RDB

Redis cung cấp hai lệnh để tạo file snapshot RDB:

- `save`: thao tác lưu đồng bộ, sẽ blocking main thread của Redis;
- `bgsave`: fork một process con để thực thi, không blocking main thread của Redis, đây là tùy chọn mặc định.

Theo mặc định, cấu hình Redis sử dụng lệnh `bgsave`. Nếu dùng thủ công lệnh `save` để tạo file snapshot RDB, main thread sẽ bị blocking.

## AOF

### Blocking khi ghi log AOF

Cơ chế persistence AOF của Redis ghi log sau khi thực thi lệnh, khác với cơ sở dữ liệu quan hệ (chẳng hạn MySQL) thường ghi log trước khi thực thi lệnh (để thuận tiện cho việc khôi phục khi gặp sự cố).

![Quy trình ghi log AOF](https://oss.javaguide.cn/github/javaguide/database/redis/redis-aof-write-log-disc.png)

**Tại sao log được ghi sau khi thực thi lệnh?**

- Tránh overhead kiểm tra bổ sung, vì việc ghi log AOF không kiểm tra cú pháp của lệnh;
- Ghi log sau khi thực thi lệnh sẽ không blocking việc thực thi lệnh hiện tại.

Cách này cũng mang lại rủi ro (tôi cũng đã đề cập khi giới thiệu persistence AOF trước đó):

- Nếu Redis bị crash ngay sau khi thực thi lệnh, thay đổi tương ứng có thể bị mất;
- **Có thể blocking việc thực thi các lệnh tiếp theo (ghi log AOF được thực hiện trong main thread của Redis)**.

### Blocking khi fsync AOF

Sau khi bật persistence AOF, mỗi khi thực thi một lệnh làm thay đổi dữ liệu trong Redis, Redis sẽ ghi lệnh đó vào AOF buffer `server.aof_buf`, sau đó, dựa trên cấu hình `appendfsync`, quyết định thời điểm đồng bộ buffer này vào file AOF trên disk.

Trong file cấu hình Redis có ba chế độ persistence AOF khác nhau (chiến lược `fsync`):

1. `appendfsync always`: sau khi main thread gọi `write` để thực hiện thao tác ghi, **main thread** lập tức gọi hàm `fsync` để đồng bộ file AOF (fsync xuống disk), sau khi `fsync` hoàn tất thì thread trả về. Chiến lược `always` do **main thread trực tiếp thực hiện fsync**, không phải background thread. Cách này an toàn nhất về dữ liệu, nhưng mỗi thao tác ghi đều đồng bộ và blocking main thread, làm giảm nghiêm trọng performance của Redis (`write` + `fsync`).
2. `appendfsync everysec`: sau khi main thread gọi `write` để thực hiện thao tác ghi, thao tác ghi trả về ngay lập tức; background thread (`aof_fsync` thread) gọi hàm `fsync` (system call) mỗi giây một lần để đồng bộ file AOF (`write` + `fsync`, khoảng thời gian giữa các lần `fsync` là 1 giây).
3. `appendfsync no`: sau khi main thread gọi `write` để thực hiện thao tác ghi, lệnh trả về ngay lập tức và để OS quyết định thời điểm đồng bộ. Trên Linux thường là 30 giây một lần (`write` nhưng không `fsync`, thời điểm `fsync` do OS quyết định).

Khi background thread (`aof_fsync` thread) gọi hàm `fsync` để đồng bộ file AOF, nó phải chờ cho đến khi ghi xong. Khi disk chịu tải quá lớn, thao tác `fsync` có thể bị blocking, đồng thời main thread gọi hàm `write` cũng sẽ bị blocking. Chỉ sau khi `fsync` hoàn tất, `write` do main thread thực hiện mới có thể trả về thành công.

Bạn có thể xem phần giới thiệu chi tiết về quy trình hoạt động của AOF tại: [Giải thích chi tiết về cơ chế persistence của Redis](./redis-persistence.md), nội dung này giúp hiểu rõ blocking khi fsync AOF.

### Blocking khi AOF rewrite

1. Khi thực thi lệnh `BGREWRITEAOF`, Redis fork một process con để rewrite file. Trong thời gian process con tạo file AOF mới, Redis server duy trì một AOF rewrite buffer để ghi lại mọi lệnh ghi được server thực thi.
2. Sau khi process con hoàn tất việc tạo file AOF mới, server sẽ append toàn bộ nội dung trong rewrite buffer vào cuối file AOF mới, để trạng thái database trong file AOF mới nhất quán với trạng thái database hiện tại.
3. Cuối cùng, server dùng file AOF mới thay thế file AOF cũ, từ đó hoàn tất thao tác AOF rewrite.

Blocking xuất hiện ở bước 2, khi dữ liệu mới trong buffer được ghi vào file mới sẽ phát sinh **blocking**.

Đọc thêm: [Phân tích vấn đề blocking khi AOF rewrite](https://cloud.tencent.com/developer/article/1633077).

## Bigkey

Nếu value tương ứng với một key chiếm dụng lượng memory tương đối lớn, key đó có thể được xem là bigkey. Cụ thể, lớn đến mức nào thì được xem là lớn? Có một tiêu chuẩn tham khảo không hoàn toàn chính xác:

- value kiểu string vượt quá 1MB;
- value của kiểu composite (List, Hash, Set, Sorted Set, v.v.) chứa hơn 5000 phần tử (đối với value kiểu composite, nhiều phần tử hơn không nhất thiết chiếm nhiều memory hơn).

Các vấn đề blocking do bigkey gây ra:

- Client timeout do blocking: Redis xử lý lệnh theo single thread, vì vậy thao tác trên bigkey có thể mất nhiều thời gian và blocking Redis. Nhìn từ phía client, đây là tình trạng không nhận được phản hồi trong thời gian dài.
- Gây network blocking: mỗi lần lấy bigkey sẽ tạo ra lượng network traffic lớn. Nếu một key có kích thước 1 MB và tốc độ truy cập là 1000 lần/giây, mỗi giây sẽ tạo ra 1000MB traffic, đây là thảm họa đối với server dùng network card 1Gbps thông thường.
- Blocking worker thread: nếu dùng `del` để xóa bigkey, worker thread sẽ bị blocking và không thể xử lý các lệnh tiếp theo.

### Tìm bigkey

Khi dùng tham số `--bigkeys` tích hợp sẵn của Redis để tìm bigkey, tốt nhất nên thực thi lệnh này trên replica, vì thực thi trên master sẽ khiến master bị **blocking**.

- Bạn cũng có thể dùng lệnh `SCAN` để tìm bigkey;
- Phân tích file RDB để tìm bigkey. Cách này yêu cầu Redis sử dụng persistence RDB. Trên Internet có sẵn các công cụ:
  - `redis-rdb-tools`: công cụ viết bằng Python dùng để phân tích file snapshot RDB của Redis;
  - `rdb_bigkeys`: công cụ viết bằng Go dùng để phân tích file snapshot RDB của Redis, có performance tốt hơn.

### Xóa bigkey

Bản chất của thao tác xóa là giải phóng vùng memory mà cặp key-value chiếm dụng.

Giải phóng memory chỉ là bước đầu tiên. Để quản lý vùng memory hiệu quả hơn, khi application giải phóng memory, **OS cần chèn memory block đã giải phóng vào linked list của các free memory block** để quản lý và phân bổ lại về sau. Bản thân quá trình này cần một khoảng thời gian nhất định và sẽ **blocking application đang thực hiện giải phóng memory**.

Vì vậy, nếu giải phóng một lượng lớn memory cùng lúc, thời gian thao tác trên linked list của free memory block sẽ tăng, từ đó gây blocking main thread của Redis. Nếu main thread bị blocking, mọi request khác có thể timeout; số lượng timeout tăng dần sẽ làm cạn kiệt Redis connection và gây ra nhiều exception.

Khi xóa bigkey, nên chia thành nhiều đợt và sử dụng thao tác xóa bất đồng bộ.

## Xóa database

Xóa database cũng tương tự việc xóa bigkey ở trên. `flushdb`, `flushall` cũng liên quan đến việc xóa và giải phóng toàn bộ cặp key-value, nên đây cũng là các điểm có thể khiến Redis bị blocking.

## Mở rộng cluster

Redis cluster có thể dynamic scale out và scale in node. Hiện tại quá trình này vẫn ở trạng thái bán tự động và cần con người can thiệp.

Khi scale out hoặc scale in, cần thực hiện data migration. Để đảm bảo consistency trong quá trình migration, Redis thực hiện mọi thao tác migration theo kiểu synchronous.

Khi thực hiện migration, Redis ở hai phía đều sẽ rơi vào trạng thái blocking trong khoảng thời gian khác nhau. Với key nhỏ, khoảng thời gian này có thể xem như không đáng kể, nhưng nếu lượng memory key sử dụng quá lớn, trong trường hợp nghiêm trọng có thể kích hoạt failover trong cluster, gây ra chuyển đổi không cần thiết.

## Swap (memory swap)

**Swap là gì?** Swap dịch theo nghĩa đen là "trao đổi". Trong Linux, Swap thường được gọi là memory swap hoặc swap partition. Nó tương tự virtual memory trong Windows: khi memory không đủ, một phần không gian trên disk được virtualize thành memory để sử dụng, từ đó giải quyết tình trạng thiếu memory. Vì vậy, tác dụng của swap partition là hy sinh disk để tăng memory, giải quyết vấn đề VPS không đủ memory hoặc bị đầy.

Swap cực kỳ nguy hiểm đối với Redis. Một tiền đề quan trọng để Redis đảm bảo performance cao là toàn bộ dữ liệu phải nằm trong memory. Nếu OS swap một phần memory Redis đang sử dụng ra disk, do tốc độ đọc ghi giữa memory và disk chênh lệch vài bậc độ lớn, performance của Redis sau khi xảy ra swap sẽ giảm mạnh.

Cách kiểm tra Redis có xảy ra Swap:

1. Tra cứu process ID của Redis

```bash
redis-cli -p 6383 info server | grep process_id
process_id: 4476
```

2. Dựa vào process ID để tra cứu thông tin memory swap

```bash
cat /proc/4476/smaps | grep Swap
Swap: 0kB
Swap: 0kB
Swap: 4kB
Swap: 0kB
Swap: 0kB
.....
```

Nếu lượng swap đều là 0KB hoặc một vài giá trị là 4KB thì bình thường.

Cách phòng tránh memory swap:

- Đảm bảo máy có đủ memory khả dụng;
- Đảm bảo mọi Redis instance đều thiết lập memory khả dụng tối đa (`maxmemory`), ngăn memory Redis tăng không kiểm soát trong trường hợp cực đoan;
- Hạ priority sử dụng swap của hệ thống, chẳng hạn `echo 10 > /proc/sys/vm/swappiness`.

## CPU contention

Redis là application CPU-intensive điển hình, không nên deploy cùng các service khác cũng CPU-intensive trên multi-core CPU. Khi process khác tiêu thụ CPU quá mức, throughput của Redis sẽ bị ảnh hưởng nghiêm trọng.

Có thể dùng `redis-cli --stat` để lấy thông tin sử dụng Redis hiện tại. Dùng lệnh `top` để lấy thông tin như mức sử dụng CPU của process; dùng thông tin thống kê từ `info commandstats` để phân tích thời gian overhead bất hợp lý của lệnh, kiểm tra xem nguyên nhân có phải do độ phức tạp thuật toán cao hoặc tối ưu memory quá mức hay không.

## Vấn đề network

Các vấn đề network như connection bị từ chối, network latency, soft interrupt của network card cũng có thể khiến Redis bị blocking.

## Tham khảo

- Phân tích và tổng hợp 6 nhóm trường hợp Redis bị blocking: <https://mp.weixin.qq.com/s/eaZCEtTjTuEmXfUubVHjew>
- Ghi chú phát triển và vận hành Redis - cơn ác mộng của Redis - blocking: <https://mp.weixin.qq.com/s/TDbpz9oLH6ifVv6ewqgSgA>

<!-- @include: @article-footer.snippet.md -->
