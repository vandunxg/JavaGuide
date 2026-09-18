---
title: Tổng hợp câu hỏi phỏng vấn cơ bản về cache
description: Giải thích chi tiết tư tưởng cốt lõi của cache, sự khác biệt giữa local cache và distributed cache, cùng thiết kế kiến trúc multi-level cache. Bao quát các giải pháp cache phổ biến như Caffeine và Redis, cùng các giải pháp đảm bảo tính nhất quán của cache. Phù hợp với Java developer muốn học thiết kế kiến trúc cache.
category: Database
tag:
  - Redis
head:
  - - meta
    - name: keywords
      content: cache,local cache,distributed cache,multi-level cache,Caffeine,Redis,tính nhất quán của cache,thiết kế hệ thống,Java cache,Guava Cache
---

> **Câu hỏi phỏng vấn liên quan**:
>
> - Vì sao cần dùng cache?
> - Nên triển khai local cache như thế nào?
> - Vì sao cần distributed cache? / Vì sao không dùng local cache trực tiếp?
> - Vì sao cần multi-level cache?
> - Multi-level cache phù hợp với những trường hợp sử dụng nào?

## Tư tưởng cơ bản của cache

Nhiều bạn chỉ biết cache có thể cải thiện performance của hệ thống và giảm **thời gian phản hồi** (Response Time), nhưng chưa hiểu rõ tư tưởng cốt lõi của cache là gì.

Tư tưởng cơ bản của cache thực ra rất đơn giản, đó là áp dụng chiến lược tối ưu performance kinh điển **dùng không gian đổi lấy thời gian**. Nói cách khác, dùng thêm storage để lưu một số dữ liệu có thể được sử dụng hoặc tính toán lại, từ đó giảm thời gian lấy lại hoặc tính toán lại dữ liệu.

Khi nói đến việc dùng không gian đổi lấy thời gian, ngoài cache, bạn còn nghĩ ra ví dụ nào khác không? Dưới đây là một số ví dụ thường gặp:

- **Index**: index là một cấu trúc dữ liệu riêng, tổ chức một số cột hoặc field trong bảng database theo một quy tắc sắp xếp nhất định. Dù chiếm thêm storage, index có thể cải thiện đáng kể hiệu quả truy vấn và giảm chi phí sắp xếp dữ liệu.
- **Dư thừa field trong bảng database**: lưu dư thừa trong cùng một bảng những dữ liệu thường được query kết hợp, để giảm việc query liên kết nhiều bảng, từ đó cải thiện performance query và giảm áp lực cho database.
- **CDN (Content Delivery Network)**: phân phối static resource đến nhiều edge node để truy cập từ vị trí gần nhất, qua đó tăng tốc độ truy cập static resource và giảm tải cho server origin cùng bandwidth.

Khi lập trình, bạn cần học cách khái quát và tổng hợp, kết nối những điều đã học với nhau! Nếu trong buổi phỏng vấn bạn có thể trao đổi được những điều này, interviewer chắc chắn sẽ có ấn tượng tốt về bạn.

Đừng nghĩ cache quá cao siêu. Dù cache thực sự có cost-performance rất cao trong việc cải thiện performance hệ thống, khi học và áp dụng cache, bạn sẽ nhận ra tư tưởng cache cũng được sử dụng rộng rãi trong CPU, operating system và nhiều nơi khác.

Ví dụ, **CPU Cache** lưu dữ liệu trong memory để giải quyết vấn đề tốc độ xử lý của **CPU** không tương xứng với tốc độ truy cập memory; memory lưu dữ liệu trên hard disk để giải quyết vấn đề tốc độ **I/O** của hard disk quá chậm.

![Sơ đồ mô hình CPU Cache](https://oss.javaguide.cn/github/javaguide/java/concurrent/cpu-cache.png)

Một ví dụ khác, để tăng tốc độ chuyển đổi từ virtual address sang physical address, operating system đã đưa **Translation Lookaside Buffer** (**TLB**, còn gọi là fast table) vào trên cơ sở page table.

![Dịch địa chỉ sau khi thêm TLB](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/physical-virtual-address-translation-mmu.png)

Lấy browser được sử dụng hằng ngày làm ví dụ, browser sẽ cache các image hoặc static file đã truy cập (browser cache), nhờ đó tốc độ load sẽ tăng đáng kể khi truy cập lại cùng một page.

![](https://oss.javaguide.cn/github/javaguide/database/redis/chrome-clear-cache.png)

Dữ liệu trong cache được sử dụng khi phát triển hằng ngày thường được lưu trong **RAM** (memory), với tốc độ truy cập cực nhanh. Để tránh mất dữ liệu trong memory sau khi restart hoặc crash, nhiều cache middleware (chẳng hạn **Redis**) cung cấp cơ chế persistence trên disk. So với relational database (chẳng hạn **MySQL**), cache có tốc độ truy cập và khả năng hỗ trợ concurrency cao hơn vài bậc độ lớn. Thêm một lớp cache phía trên database là biện pháp cốt lõi để bảo vệ storage bên dưới và tăng throughput hệ thống.

## Phân loại cache

Tiếp theo, hãy cùng xem cache được sử dụng trong phát triển hằng ngày thường được chia thành những loại nào.

### Local cache

#### Local cache là gì?

Local cache được sử dụng khá nhiều trong nhiều project, đặc biệt là khi dùng monolithic architecture. Nếu data volume không lớn và không có yêu cầu distributed, sử dụng local cache vẫn phù hợp.

Local cache nằm bên trong application. Ưu điểm lớn nhất là application và cache cùng nằm trong một process, nên tốc độ request đến local cache rất nhanh và không phát sinh network overhead.

Sơ đồ monolithic architecture thường gặp như sau: dùng **Nginx** để thực hiện **load balancing**, deploy hai application giống nhau lên server. Hai service dùng chung một database và đều sử dụng local cache.

![Sơ đồ local cache](https://oss.javaguide.cn/github/javaguide/database/redis/local-cache.png)

**Lưu ý:** Khi sử dụng local cache trong cluster mode, phải cân nhắc **load balancing strategy**. Nếu Nginx sử dụng **Round-Robin** mặc định, request của cùng một user có thể ngẫu nhiên rơi vào các machine khác nhau, khiến cache hit rate cực thấp. Giải pháp như sau:

1. **Gateway layer**: sử dụng consistent hashing hoặc Sticky Session để bảo đảm request của cùng một user luôn được chuyển đến cùng một machine.
2. **Application layer**: chỉ sử dụng local cache cho data **“gần như không thay đổi trên toàn hệ thống”** (chẳng hạn configuration dictionary), không dùng cho data theo user.

#### Có những giải pháp local cache nào?

**1. `HashMap` và `ConcurrentHashMap` có sẵn trong JDK.**

`ConcurrentHashMap` có thể được xem là phiên bản thread-safe của `HashMap`. Cả hai đều lưu các key-value pair. Tuy nhiên, trong phần lớn trường hợp, hai loại này không được dùng làm cache vì chỉ cung cấp chức năng cache, không cung cấp các chức năng khác như expiration time. Một cache framework tương đối hoàn chỉnh tối thiểu phải cung cấp ba chức năng: **expiration time**, **eviction mechanism** và **hit rate statistics**.

**2. `Ehcache`, `Guava Cache` và `Spring Cache` là ba local cache framework được sử dụng khá nhiều.**

- So với hai loại còn lại, `Ehcache` nặng hơn. Tuy nhiên, so với `Guava Cache` và `Spring Cache`, `Ehcache` hỗ trợ tích hợp vào Hibernate và MyBatis làm multi-level cache, có thể persistence data cache vào local disk, đồng thời cũng cung cấp giải pháp cluster (khá hạn chế, có thể bỏ qua).
- `Guava Cache` và `Spring Cache` khá giống nhau. `Guava` được sử dụng nhiều hơn `Spring Cache`; nó cung cấp API rất tiện lợi, đồng thời hỗ trợ thiết lập thời gian hiệu lực của cache và các chức năng khác. Implementation bên trong của nó cũng khá gọn gàng, nhiều nơi có tư tưởng tương tự `ConcurrentHashMap`.
- Nếu dùng annotation của `Spring Cache` để implement cache, code sẽ trông rất gọn gàng và elegant, nhưng cũng dễ phát sinh các vấn đề như cache penetration và out of memory.

**3. Caffeine, một lựa chọn mới nổi.**

So với `Guava`, `Caffeine` tốt hơn về nhiều mặt, chẳng hạn performance. Thông thường nên dùng nó để thay thế `Guava`. Ngoài ra, cách sử dụng `Guava` và `Caffeine` cũng rất giống nhau!

Ví dụ code tạo local cache bằng `Caffeine`, sử dụng builder pattern:

```java
// Ví dụ tạo local cache bằng Caffeine
Cache<String, String> cache = Caffeine.newBuilder()
        // Hết hạn sau 60 ngày kể từ khi ghi
        .expireAfterWrite(60, TimeUnit.DAYS)
        // Dung lượng ban đầu
        .initialCapacity(100)
        // Giới hạn số lượng entry tối đa
        .maximumSize(500)
        // Bật chức năng thống kê
        .recordStats()
        .build();
```

#### Local cache có những điểm hạn chế nào?

Ưu điểm của local cache rất rõ ràng: **ít dependency**, **nhẹ**, **đơn giản**, **cost thấp**.

Tuy nhiên, local cache có những hạn chế sau:

- **Local cache gắn chặt với application và không thân thiện với distributed architecture**. Ví dụ, khi cùng một service được deploy trên nhiều machine, cache giữa các service không thể dùng chung vì local cache chỉ tồn tại trên machine hiện tại.
- **Dung lượng local cache bị giới hạn rõ rệt bởi machine nơi service được deploy.** Nếu service của hệ thống hiện tại tiêu thụ nhiều memory, dung lượng còn có thể dùng cho local cache sẽ rất ít.

### Distributed cache

#### Distributed cache là gì?

Có thể xem distributed cache (Distributed Cache) là một service của memory database, có chức năng cuối cùng là cung cấp service cho cache data.

Distributed cache tồn tại độc lập với application, nhiều application có thể cùng sử dụng một distributed cache service.

Hình dưới đây là sơ đồ kiến trúc đơn giản sử dụng distributed cache. Dùng Nginx để thực hiện load balancing, deploy hai application giống nhau lên server. Hai service dùng chung một database và cache.

![Distributed cache](https://oss.javaguide.cn/github/javaguide/database/redis/distributed-cache.png)

Sau khi sử dụng distributed cache, cache service có thể được deploy trên một server riêng. Ngay cả khi cùng một service được deploy trên nhiều machine, chúng vẫn sử dụng chung một cache. Ngoài ra, performance, capacity và chức năng được cung cấp bởi distributed cache service riêng cũng mạnh hơn nhiều.

**Trong thiết kế software system không có silver bullet, việc đưa vào bất kỳ công nghệ nào thường cũng giống như con dao hai lưỡi.** Nếu sử dụng đúng cách, công nghệ có thể mang lại lợi ích lớn cho hệ thống. Nếu không, bạn chỉ tốn công sức mà không thu được kết quả.

Nói đơn giản, việc đưa distributed cache vào hệ thống thường mang lại các vấn đề sau:

- **Độ phức tạp của hệ thống tăng**: sau khi thêm cache, bạn phải duy trì tính nhất quán giữa cache và database, duy trì hot cache, bảo đảm high availability của cache service, v.v.
- **Chi phí phát triển hệ thống thường tăng**: thêm cache đồng nghĩa với việc hệ thống cần một cache service riêng, việc này phát sinh cost tương ứng. Cost này còn khá cao vì tiêu tốn memory quý giá.

#### Có những giải pháp distributed cache nào?

Trong các distributed cache, những lựa chọn lâu đời và được sử dụng nhiều vẫn là **Memcached** và **Redis**. Tuy nhiên, hiện nay hầu như không còn thấy project nào dùng **Memcached** làm cache, mà đều trực tiếp dùng **Redis**.

Memcached từng được sử dụng khá phổ biến khi distributed cache mới bắt đầu phát triển. Sau đó, cùng với sự phát triển của Redis, mọi người dần chuyển sang Redis mạnh hơn.

Một số công ty lớn cũng open source các distributed high-performance KV storage database tương tự Redis, chẳng hạn [Tendis](https://github.com/Tencent/Tendis) do Tencent open source. Tendis sử dụng [RocksDB](https://github.com/facebook/rocksdb), một open source project nổi tiếng, làm storage engine, tương thích 100% với Redis protocol và tất cả data model của Redis 4.0. Về so sánh Redis và Tendis, Tencent từng đăng bài viết chính thức [Redis vs Tendis: Hé lộ kiến trúc hybrid storage nóng-lạnh](https://mp.weixin.qq.com/s/MeYkfOIdnU6LYlsGb24KjQ), bạn có thể tham khảo sơ lược.

Tuy nhiên, từ lịch sử commit Github của project Tendis có thể thấy bản open source của Tendis gần như không còn được maintain và update. Hơn nữa, mức độ quan tâm không cao và số công ty sử dụng cũng ít. Vì vậy, không khuyến nghị dùng Tendis để implement distributed cache.

Hiện nay, hai distributed cache open source sau vẫn là các lựa chọn thay thế Redis được industry công nhận (đều nổi tiếng nhờ tận dụng danh tiếng của Redis):

- [Dragonfly](https://github.com/dragonflydb/dragonfly): memory database được xây dựng cho nhu cầu workload của application hiện đại, tương thích hoàn toàn với API của Redis và Memcached, không cần sửa code khi migration, tự nhận là memory database nhanh nhất thế giới.
- [KeyDB](https://github.com/Snapchat/KeyDB): một high-performance fork của Redis, tập trung vào multi-threading, memory efficiency và high throughput.

Tuy nhiên, cá nhân tôi vẫn khuyến nghị ưu tiên Redis cho distributed cache. Redis đã trải qua kiểm chứng production trong nhiều năm, có ecosystem rất tốt và tài liệu cũng đầy đủ.

### Multi-level cache

#### Multi-level cache là gì? Vì sao cần dùng?

Ở đây chỉ trao đổi đơn giản về giải pháp multi-level cache **local cache + distributed cache**, cũng là cách implement multi-level cache phổ biến nhất.

Đến đây có lẽ nhiều bạn sẽ hỏi: **Đã dùng distributed cache rồi thì vì sao còn cần local cache?**

Mặc dù local cache và distributed cache đều là cache, tốc độ truy cập local cache lớn hơn rất nhiều so với distributed cache, vì truy cập local cache không phát sinh network overhead, như đã đề cập ở trên.

Tuy nhiên, trong điều kiện thông thường, cũng không khuyến nghị sử dụng multi-level cache vì nó làm tăng gánh nặng maintenance (chẳng hạn cần bảo đảm tính nhất quán dữ liệu giữa L1 cache và L2 cache). Hơn nữa, với phần lớn trường hợp sử dụng, hiệu quả cải thiện thực tế không quá lớn.

Dưới đây là hai trường hợp sử dụng phù hợp với multi-level cache:

- Data cache không thường xuyên thay đổi, tương đối ổn định;
- Lưu lượng truy cập data đặc biệt lớn, chẳng hạn trường hợp flash sale.

Trong giải pháp multi-level cache, level cache thứ nhất (L1) sử dụng local memory (chẳng hạn Caffeine), level cache thứ hai (L2) sử dụng distributed cache (chẳng hạn Redis).

![Multi-level cache](https://oss.javaguide.cn/javaguide/database/redis/multilevel-cache.png)

Khi đọc cache data, trước tiên đọc từ L1 và trả về trực tiếp nếu hit; nếu L1 miss thì đọc từ L2. Nếu L2 hit, trước tiên nên ghi data về L1 của instance hiện tại rồi mới trả về kết quả, tránh việc mỗi lần L1 miss sau đó đều phải truy cập L2 lặp lại. Nếu L2 cũng không có data, tiếp tục query database. Sau khi query thành công, ghi data vào cả L1 và L2. Cách này có thể giảm số lần đọc L2 và giảm áp lực cho L2.

Một số open source implementation của multi-level cache được khuyến nghị:

- [J2Cache](https://gitee.com/ld/J2Cache): Java cache framework hai level dựa trên local memory và Redis.
- [JetCache](https://github.com/alibaba/jetcache): cache framework do Alibaba open source, hỗ trợ multi-level cache, tự động refresh distributed cache, TTL và các chức năng khác.

#### Đảm bảo tính nhất quán của multi-level cache như thế nào?

Trong hệ thống multi-level cache, chi phí để bảo đảm strong consistency quá cao. Một số cache framework cung cấp chức năng multi-level cache trong industry về cơ bản đều bảo đảm eventual consistency. Chẳng hạn, có thể sử dụng cơ chế publish/subscribe của Redis, Redis Stream hoặc message queue để bảo đảm khi local cache của một instance thay đổi, các instance khác có thể kịp thời update local cache của chúng nhằm duy trì tính nhất quán của cache.

Giải pháp của Zhengcaiyun Technology là Canal + broadcast message, được giới thiệu đơn giản như sau:

1. DB sửa data: trước tiên sửa data trong database.
2. Trigger update cache thông qua việc lắng nghe Canal message: dùng Canal lắng nghe thao tác thay đổi database, khi phát hiện data thay đổi thì trigger update cache.
3. Đồng bộ Redis cache: với Redis cache, vì trong cluster chỉ dùng chung một bản data nên chỉ cần đồng bộ cache trực tiếp.
4. Đồng bộ local cache: vì local cache nằm trong các JVM instance khác nhau, cần dựa vào cơ chế broadcast message queue (MQ) để broadcast thông báo update đến các business instance, từ đó đồng bộ local cache.

Xem chi tiết: [Thiết kế và thực chiến hệ thống distributed multi-level cache](https://juejin.cn/post/7225634879152570405)

## Đọc thêm về data structure

- [Giải thích chi tiết Bloom Filter](../../cs-basics/data-structure/bloom-filter.md): tìm hiểu cache penetration, false positive rate và khó khăn khi xóa.
- [Tổng hợp câu hỏi phỏng vấn về LRU cache](../../cs-basics/data-structure/lru-cache.md): tìm hiểu eviction strategy của local cache, cách viết `LinkedHashMap` và tư tưởng page replacement.
- [Tổng hợp câu hỏi phỏng vấn về hash table](../../cs-basics/data-structure/hash-table.md): tìm hiểu key mapping của cache, hash collision và resize.

## Tham khảo

- Những câu chuyện về cache: https://tech.meituan.com/2017/03/17/cache-about.html
- Phân tích thiết kế cache của distributed system: https://segmentfault.com/a/1190000041689802
