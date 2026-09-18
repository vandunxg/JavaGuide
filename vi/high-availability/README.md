---
title: "Hệ thống kiến thức về high availability: SLA, rate limiting, circuit breaker, degradation, retry, idempotency, redundancy và kiểm thử tải"
description: "Lộ trình học high availability system cho phỏng vấn và system design, bao quát SLA, chỉ số availability, single point of failure, redundancy, disaster recovery, rate limiting, degradation, circuit breaker, timeout retry, API idempotency, kiểm thử tải và diễn tập sự cố."
category: High availability
tag:
  - High availability
  - System design
  - Backend interview
sitemap:
  changefreq: weekly
  priority: 0.95
head:
  - - meta
    - name: keywords
      content: high availability system, high availability system design, high availability interview questions, SLA, chỉ số availability, rate limiting, circuit breaker, degradation, timeout retry, API idempotency, redundancy design, disaster recovery, performance testing, diễn tập sự cố, backend interview
---

<!-- @include: @small-advertisement.snippet.md -->

Tài liệu **hệ thống kiến thức về high availability** này dành cho việc học backend, system design và ôn phỏng vấn, tổng hợp các bài viết về high availability trên trang xoay quanh việc giảm sự cố, cô lập sự cố, khôi phục nhanh, giảm ảnh hưởng của request lặp và traffic bất thường.

Nếu có ít thời gian, bạn nên đọc trước [Tổng hợp câu hỏi phỏng vấn system design về high availability](./high-availability-system-interview-questions.md) để nhanh chóng xây dựng danh sách câu hỏi thường gặp; nếu muốn bổ sung nền tảng một cách có hệ thống, bạn có thể đọc theo thứ tự bên dưới.

## Dành cho ai

- Backend developer đang học system design về high availability một cách có hệ thống.
- Người chuẩn bị phỏng vấn backend cho kỳ tuyển dụng sinh viên mới tốt nghiệp, tuyển dụng người đã có kinh nghiệm hoặc các công ty vừa và lớn.
- Engineer muốn bổ sung năng lực về rate limiting, degradation, circuit breaker, retry, idempotency, disaster recovery và kiểm thử tải.
- Người đã gặp các vấn đề như API timeout, request lặp, lỗi từ downstream, traffic tăng đột biến, system avalanche nhưng thiếu giải pháp có hệ thống.

## Trọng tâm học

- High availability không có nghĩa là “không xảy ra sự cố”, mà là khi sự cố xảy ra thì có thể kiểm soát, cô lập và khôi phục tối đa.
- SLA, chỉ số availability, RTO và RPO là ngôn ngữ quan trọng để đánh giá thiết kế high availability.
- Rate limiting, degradation, circuit breaker và timeout retry lần lượt giải quyết các vấn đề stability ở những tầng khác nhau.
- Thiết kế idempotency là năng lực nền tảng để xử lý request lặp, auto retry, message được consume lặp và callback thanh toán.
- Redundancy, disaster recovery, performance testing, diễn tập sự cố và gray release quyết định hệ thống có chịu được rủi ro production thực tế hay không.

## Thứ tự đọc đề xuất

1. [Tổng hợp câu hỏi phỏng vấn system design về high availability](./high-availability-system-interview-questions.md): trước hết xây dựng danh sách câu hỏi thường gặp về SLA, rate limiting, circuit breaker, retry, idempotency, disaster recovery và kiểm thử tải.
2. [Giải thích chi tiết về system design của high availability](./high-availability-system-design.md): hiểu có hệ thống phương pháp thiết kế tổng thể của kiến trúc high availability.
3. [Giải thích chi tiết về service rate limiting](./limit-request.md), [Giải thích chi tiết về service degradation và circuit breaker](./fallback-and-circuit-breaker.md), [Giải thích chi tiết về cơ chế timeout và retry](./timeout-and-retry.md): nắm vững bộ ba quản trị stability.
4. [Giải thích chi tiết về thiết kế API idempotency](./idempotency.md): hiểu các scenario như request lặp, business idempotency và callback thanh toán.
5. [Giải thích chi tiết về thiết kế redundancy](./redundancy.md) và [Nhập môn performance testing](./performance-test.md): bổ sung kiến thức về disaster recovery, kiểm thử tải, đánh giá capacity và xác minh sự cố.

## Bài viết cốt lõi

### Tổng quan và lộ trình phỏng vấn

- [Tổng hợp câu hỏi phỏng vấn system design về high availability](./high-availability-system-interview-questions.md): kết nối SLA, single point of failure, rate limiting, degradation, circuit breaker, timeout retry, API idempotency, redundancy, disaster recovery và performance testing.
- [Giải thích chi tiết về system design của high availability](./high-availability-system-design.md): giải thích SLA, bao nhiêu số 9, quản trị single point of failure, high availability của cache, asynchronous traffic peak shaving, gray release và khôi phục sự cố.

### Quản trị stability

- [Giải thích chi tiết về service rate limiting](./limit-request.md): tìm hiểu fixed window, sliding window, token bucket, leaky bucket, Sentinel, Redis Lua và rate limiting của Redisson.
- [Giải thích chi tiết về service degradation và circuit breaker](./fallback-and-circuit-breaker.md): tìm hiểu Fallback, degradation switch, state machine của circuit breaker, hiệu ứng system avalanche, chiến lược isolation và lựa chọn framework.
- [Giải thích chi tiết về cơ chế timeout và retry](./timeout-and-retry.md): tìm hiểu connection timeout, read timeout, exponential backoff, random jitter, retry storm và idempotency.

### Idempotency, disaster recovery và xác minh

- [Giải thích chi tiết về thiết kế API idempotency](./idempotency.md): giải thích idempotency key, Token, unique index, deduplication table, optimistic lock, pessimistic lock, distributed lock và callback thanh toán.
- [Giải thích chi tiết về thiết kế redundancy](./redundancy.md): giải thích hardware redundancy, service redundancy, data redundancy, geographic redundancy, disaster recovery trong cùng thành phố, disaster recovery khác địa điểm và kiến trúc multi-active.
- [Nhập môn performance testing](./performance-test.md): tìm hiểu QPS, TPS, RT, P90/P99/P999, đánh giá capacity, stress testing, stability testing và công cụ kiểm thử tải.

## Câu hỏi thường gặp

- High availability system được định nghĩa availability như thế nào? Mỗi số 9 lần lượt có ý nghĩa gì?
- RTO và RPO khác nhau như thế nào?
- Rate limiting, degradation và circuit breaker lần lượt giải quyết vấn đề gì?
- Token bucket và leaky bucket khác nhau như thế nào? Distributed rate limiting được triển khai ra sao?
- Vì sao timeout và retry có thể gây system avalanche?
- API idempotency có những phương án triển khai nào? Làm thế nào để đảm bảo idempotency cho callback thanh toán?
- Redundancy, disaster recovery, multi-active trong cùng thành phố và multi-active khác địa điểm lần lượt phù hợp với scenario nào?
- Performance testing, stress testing và stability testing khác nhau như thế nào?
- Làm thế nào để xác minh thiết kế high availability có hiệu quả thông qua diễn tập sự cố?

## Chuyên đề liên quan

- [Hệ thống kiến thức về high performance](../high-performance/)
- [Hệ thống kiến thức về distributed system](../distributed-system/)
- [System design](../system-design/)
- [Chuyên đề message queue](../high-performance/message-queue/)

<!-- @include: @article-footer.snippet.md -->
