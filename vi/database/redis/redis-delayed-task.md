---
title: "Làm thế nào để triển khai delayed task bằng Redis?"
description: "Giải thích chi tiết hai phương án triển khai delayed task bằng Redis: lắng nghe event hết hạn và delay queue của Redisson; phân tích ưu, nhược điểm, vấn đề độ tin cậy và trường hợp sử dụng của từng phương án."
category: Database
tag:
  - Redis
head:
  - - meta
    - name: keywords
      content: "Redis delayed task, delay queue, lắng nghe event hết hạn, Redisson DelayedQueue, order timeout, scheduled task"
---

Redis chỉ có hai phương án chính để triển khai delayed task:

1. Lắng nghe event hết hạn của Redis
2. Delay queue tích hợp sẵn của Redisson

Khi phỏng vấn, trước tiên bạn có thể nói rằng mình đã cân nhắc hai phương án này, nhưng cuối cùng nhận thấy phương án lắng nghe event hết hạn của Redis có nhiều vấn đề, nên chọn DelayedQueue tích hợp sẵn của Redisson.

Interviewer có thể hỏi thêm một số vấn đề liên quan. Phần sau sẽ đề cập đến các vấn đề này, bạn chỉ cần chuẩn bị trước.

Ngoài những vấn đề được giới thiệu dưới đây, bạn cũng nên ôn lại các vấn đề Redis thường gặp, vì interviewer có thể hỏi thêm những vấn đề khác về Redis.

### Nguyên lý triển khai delayed task bằng cách lắng nghe event hết hạn của Redis là gì?

Redis 2.0 giới thiệu tính năng publish-subscribe (pub/sub). Trong pub/sub, có một khái niệm gọi là **channel (kênh)**, tương tự phần nào với **topic (chủ đề)** trong message queue.

pub/sub có hai vai trò: publisher (bên phát) và subscriber (còn gọi là consumer):

- Publisher gửi message tới channel được chỉ định bằng `PUBLISH`.
- Subscriber đăng ký channel mình quan tâm bằng `SUBSCRIBE`. Subscriber có thể đăng ký một hoặc nhiều channel.

![Tính năng publish-subscribe (pub/sub) của Redis](https://oss.javaguide.cn/github/javaguide/database/redis/redis-pub-sub.png)

Trong mode pub/sub, producer cần chỉ định channel nhận message, còn consumer đăng ký channel tương ứng để nhận message.

Redis có nhiều channel mặc định. Redis tự gửi message tới các channel này, thay vì code do chúng ta tự viết. Trong đó, `__keyevent@0__:expired` là một channel mặc định, phụ trách lắng nghe event hết hạn của key. Nói cách khác, sau khi một key hết hạn, Redis sẽ phát một event key hết hạn tới channel `__keyevent@<db>__:expired`.

Chỉ cần lắng nghe channel này, chúng ta có thể nhận message về key đã hết hạn, từ đó triển khai chức năng delayed task.

Tính năng này được Redis gọi chính thức là **keyspace notifications**, dùng để giám sát real-time thay đổi của key và value trong Redis.

### Phương án lắng nghe event hết hạn của Redis để triển khai delayed task có những nhược điểm gì?

**1. Tính kịp thời kém**

Tài liệu chính thức giải thích nguyên nhân của vấn đề này tại: <https://redis.io/docs/manual/keyspace-notifications/#timing-of-expired-events>.

![Event hết hạn của Redis](https://oss.javaguide.cn/github/javaguide/database/redis/redis-timing-of-expired-events.png)

Ý chính là: message event hết hạn được phát khi Redis server xóa key, chứ không được phát ngay sau khi key hết hạn.

Có hai strategy thường dùng để xóa dữ liệu hết hạn:

1. **Lazy deletion**: chỉ kiểm tra dữ liệu hết hạn khi lấy key. Cách này thân thiện nhất với CPU, nhưng có thể khiến quá nhiều key hết hạn chưa được xóa.
2. **Periodic deletion**: định kỳ lấy một batch key để xóa key hết hạn. Đồng thời, Redis giới hạn thời lượng và tần suất thực hiện thao tác xóa ở tầng dưới để giảm ảnh hưởng đến CPU.

Periodic deletion thân thiện hơn với memory, còn lazy deletion thân thiện hơn với CPU. Mỗi cách có ưu điểm riêng, vì vậy Redis sử dụng **periodic deletion + lazy deletion**.

Do đó, có thể xảy ra trường hợp chúng ta đã đặt thời gian hết hạn cho key, nhưng đến thời điểm chỉ định key vẫn chưa bị xóa, dẫn đến chưa phát event hết hạn.

**2. Mất message**

Message trong mode pub/sub của Redis không hỗ trợ persistence, khác với message queue. Trong mode pub/sub của Redis, publisher gửi message tới channel được chỉ định, subscriber lắng nghe channel tương ứng để nhận message. Khi không có subscriber, message sẽ bị loại bỏ trực tiếp và không được lưu trong Redis.

**3. Message bị xử lý lặp khi có nhiều service instance**

Mode pub/sub của Redis hiện chỉ có broadcast mode. Điều này có nghĩa là khi producer phát một message tới channel cụ thể, tất cả consumer đăng ký channel tương ứng đều có thể nhận message đó.

Lúc này, cần chú ý việc nhiều service instance xử lý cùng một message, làm tăng khối lượng phát triển code và độ khó bảo trì.

### Delay queue của Redisson hoạt động theo nguyên lý nào? Có ưu điểm gì?

Redisson là một Redis client mã nguồn mở dành cho Java, cung cấp nhiều tính năng dùng ngay, như nhiều implementation của distributed lock và delay queue.

Chúng ta có thể dùng delay queue RDelayedQueue tích hợp sẵn của Redisson để triển khai delayed task.

Delay queue RDelayedQueue của Redisson được triển khai dựa trên SortedSet của Redis. SortedSet là một ordered set, trong đó mỗi element có thể được đặt một score đại diện cho weight của element đó. Redisson tận dụng đặc tính này, đưa các task cần thực thi sau vào SortedSet và đặt thời gian hết hạn tương ứng làm score cho chúng.

Redisson định kỳ dùng command `zrangebyscore` để quét các element đã hết hạn trong SortedSet, sau đó xóa chúng khỏi SortedSet và thêm vào ready message list. Ready message list là một blocking queue; khi có message được thêm vào, consumer sẽ nhận được thông báo. Cách này tránh việc consumer phải polling toàn bộ SortedSet, từ đó nâng cao hiệu suất thực thi.

So với phương án lắng nghe event hết hạn của Redis để triển khai delayed task, cách này có các ưu điểm sau:

1. **Giảm khả năng mất message**: Message trong DelayedQueue được persist. Ngay cả khi Redis bị down, dựa trên cơ chế persistence, nhiều nhất cũng chỉ mất một lượng nhỏ message, ảnh hưởng không lớn. Tất nhiên, bạn cũng có thể dùng cách scan database làm cơ chế bù.
2. **Không xảy ra duplicate consumption**: Mỗi client đều lấy task từ cùng một queue đích, nên không xảy ra duplicate consumption.

So với delay queue tích hợp sẵn của Redisson, message queue có thể đạt throughput và độ tin cậy cao hơn bằng cách bảo đảm độ tin cậy của message consumption, kiểm soát số lượng producer và consumer cùng các biện pháp khác. Trong project thực tế, nên ưu tiên phương án delayed message của message queue.
