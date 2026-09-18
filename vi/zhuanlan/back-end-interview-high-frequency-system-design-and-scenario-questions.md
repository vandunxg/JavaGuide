---
title: Câu hỏi phỏng vấn System Design backend thường gặp | Câu hỏi tình huống | Seckill system | Short link system (kèm đáp án)
description: Phân tích các câu hỏi System Design và tình huống thường gặp trong backend interview, bao quát seckill system, short link system, xử lý dữ liệu khổng lồ, distributed ID và hơn 30 câu hỏi phỏng vấn kinh điển, phù hợp để chuẩn bị cho backend interview tại các công ty vừa và lớn.
category: Knowledge Planet
head:
  - - meta
    - name: keywords
      content: câu hỏi phỏng vấn System Design,câu hỏi tình huống,System Design backend,seckill system,short link system,câu hỏi phỏng vấn xử lý dữ liệu khổng lồ,distributed system,câu hỏi phỏng vấn thường gặp,case System Design,câu hỏi tình huống backend
---

## Giới thiệu

**《Các câu hỏi System Design và tình huống thường gặp trong backend interview》** là một booklet nội bộ trong [Knowledge Planet](../about-the-author/zhishixingqiu-two-years.md) của tôi, tổng hợp có hệ thống các case System Design và câu hỏi tình huống thường gặp trong backend interview.

### Vì sao bạn cần booklet này?

Những năm gần đây, các buổi phỏng vấn kỹ thuật trong nước ngày càng cạnh tranh gay gắt. Ngày càng nhiều công ty (Alibaba, Meituan, ByteDance, Tencent...) bắt đầu đánh giá **System Design** và **câu hỏi tình huống** trong phỏng vấn để đánh giá toàn diện hơn năng lực tổng thể của ứng viên, dù là tuyển dụng sinh viên mới tốt nghiệp hay tuyển dụng người đã có kinh nghiệm.

> Nhiều bạn học thuộc lòng các câu hỏi lý thuyết phỏng vấn, nhưng vừa gặp câu hỏi mở như “Làm thế nào để thiết kế một seckill system?” là bối rối.

**Đặc điểm đánh giá System Design và câu hỏi tình huống**:

- ✅ Không có đáp án tiêu chuẩn, trọng tâm là đánh giá quá trình tư duy và năng lực architecture
- ✅ Đánh giá khả năng vận dụng tổng hợp các kỹ thuật như high concurrency, high availability, distributed
- ✅ Đánh giá năng lực giải quyết vấn đề thực tế và kinh nghiệm engineering
- ⚠️ Một buổi phỏng vấn thông thường không chỉ toàn câu hỏi tình huống, thường sẽ xen kẽ 1-2 câu hỏi để đánh giá bạn

Vì vậy, booklet **《Các câu hỏi System Design và tình huống thường gặp trong backend interview》** đã ra đời!

### Booklet này có thể mang lại gì cho bạn?

**1. Điểm cộng trong phỏng vấn**

Nếu trả lời tốt các câu hỏi System Design và tình huống, bạn sẽ để lại ấn tượng rất tốt với interviewer! Chỉ cần chuẩn bị một chút cho dạng câu hỏi này là bạn có thể nổi bật.

**2. Nâng cao tư duy System Design**

Ngay cả khi không chuẩn bị cho phỏng vấn, booklet này vẫn có thể giúp bạn xây dựng framework tư duy System Design và nâng cao năng lực giải quyết vấn đề thực tế.

**3. Tham khảo để áp dụng thực tế**

Nhiều case được đề cập có thể áp dụng trực tiếp vào project của bạn, chẳng hạn như:

- Đăng nhập OAuth của bên thứ ba (WeChat/QQ)
- Cách chính xác để dùng Redis triển khai delayed task
- Thiết kế và triển khai dynamic thread pool
- Nhiều phương án triển khai distributed lock

## Tổng quan nội dung

### 📐 Case System Design

| Chủ đề                                                                  | Kiến thức trọng tâm                                                                        |
| ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| ⭐ **Thiết kế một dynamic thread pool như thế nào?**                    | Điều chỉnh động tham số thread pool, monitoring alert, rejection policy, graceful shutdown |
| **Thiết kế một hệ thống tin nhắn nội bộ như thế nào?**                  | Push message, thống kê số lượng chưa đọc, WebSocket, message queue                         |
| **Thiết kế hệ thống Weibo Feed stream / information feed như thế nào?** | Push/pull model, Timeline, smart recommendation, read/write diffusion, cache strategy      |
| **Thiết kế một bảng xếp hạng như thế nào?**                             | Redis Sorted Set, real-time update, pagination query, sắp xếp dữ liệu khổng lồ             |
| **Một số case System Design điển hình (bổ sung)**                       | Chia sẻ các case tổng hợp như like, coupon, red packet                                     |

### 🎯 Câu hỏi tình huống thường gặp

| Chủ đề                                                                 | Kiến thức trọng tâm                                                                  |
| ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| ⭐ **Triển khai tự động hủy order quá hạn như thế nào?**               | Delayed queue, scheduled task, state machine, bảo đảm idempotency                    |
| **Triển khai delayed task dựa trên Redis như thế nào?**                | Lắng nghe expiration event vs Redisson DelayedQueue, tính kịp thời, reliability      |
| ⭐ **Giải quyết vấn đề upload file lớn như thế nào?**                  | Chunked upload, resumable upload, instant upload, concurrent upload, file validation |
| **Triển khai chức năng xác định vị trí địa lý của IP như thế nào?**    | Lựa chọn IP library, offline library vs online API, performance optimization         |
| **Thống kê UV của website như thế nào?**                               | Khái niệm PV/UV/VV/IP, HyperLogLog, thống kê deduplication                           |
| ⭐ **Một số câu hỏi tình huống backend interview điển hình (bổ sung)** | Các scenario tổng hợp như rate limiting, idempotency, cache penetration              |

### 🔐 Authentication, security và risk control

| Chủ đề                                                              | Kiến thức trọng tâm                                                              |
| ------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| ⭐ **Triển khai masking sensitive word trong project như thế nào?** | Masking strategy, regex matching, performance optimization, dynamic config       |
| ⭐ **Truyền và lưu trữ password an toàn như thế nào?**              | Salted hash, BCrypt, HTTPS, ngăn replay attack                                   |
| **Triển khai đăng nhập OAuth của bên thứ ba như thế nào?**          | OAuth 2.0 protocol, authorization code flow, Token mechanism, JWT                |
| **Thiết kế scenario đăng nhập bằng captcha như thế nào?**           | Tạo captcha, lưu trữ, validation, chống abuse, quản lý thời hạn hiệu lực         |
| **Hạn chế đăng nhập như thế nào sau nhiều lần nhập sai password?**  | Rate limiting strategy, Redis counter, sliding window, distributed rate limiting |

### 📊 Scenario dữ liệu lớn

| Chủ đề                                                                                   | Kiến thức trọng tâm                                                 |
| ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| ⭐ **40 tỷ số QQ, giới hạn 1G memory, làm thế nào để deduplication?**                    | Bitmap, Bloom filter, tư duy divide and conquer, external sorting   |
| ⭐ **DAU hơn 100 triệu, làm thế nào để bảo đảm video được recommendation không bị lặp?** | Bloom filter, Redis Set, deduplication strategy, space optimization |
| ⭐ **Bài toán Top K trên dữ liệu lớn**                                                   | Heap sort, quickselect, divide and conquer, MapReduce               |

### 🔄 Concurrency control và distributed consistency

| Chủ đề                                                                  | Kiến thức trọng tâm                                                              |
| ----------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| **Nhiều rider tranh một order, làm thế nào để bảo đảm không bị trùng?** | Distributed lock, optimistic lock, Redis SETNX, concurrency control              |
| **Xử lý thế nào khi rút tiền thất bại (hoàn order)?**                   | Compensation mechanism, idempotent design, state rollback, reconciliation system |

## Xem trước nội dung

![Các câu hỏi System Design và tình huống thường gặp trong backend interview](https://oss.javaguide.cn/xingqiu/back-end-interview-high-frequency-system-design-and-scenario-questions-fengmian.png)

## Đối tượng phù hợp

- 🎓 **Ứng viên tìm việc sau khi tốt nghiệp**: ứng phó với System Design interview tại các công ty lớn
- 👨‍💻 **Người chuyển việc qua tuyển dụng có kinh nghiệm**: nâng cao năng lực architecture design, nhận được offer tốt hơn
- 🔧 **Engineer junior và mid-level**: học tư duy System Design, nâng cao năng lực giải quyết vấn đề thực tế
- 📚 **Người yêu thích công nghệ**: tìm hiểu nguyên lý thiết kế của các hệ thống thường gặp

<!-- @include: @planet2.snippet.md -->
