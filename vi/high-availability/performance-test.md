---
title: "Nhập môn performance testing và stress testing: RPS, QPS, TPS, P99 và công cụ load testing"
description: "Hướng dẫn nhập môn performance testing, giải thích RPS, QPS, TPS, RT, P90/P99/P999, Little's Law, đánh giá capacity, Hockey Stick Curve, backpressure và kiểm chứng self-healing, cùng load testing, stress testing, stability testing và các công cụ JMeter, Gatling, k6, Locust, ab, wrk."
category: High Availability
icon: "mdi:speedometer"
tag:
  - Performance Testing
  - Stress Testing
head:
  - - meta
    - name: keywords
      content: performance testing,stress testing,load testing,stability testing,RPS,QPS,TPS,RT response time,P99,capacity assessment,concurrency,throughput,backpressure,Little's Law,JMeter,Gatling,k6,Locust,ab load testing,wrk load testing,performance testing interview questions
---

Performance testing thường do đội ngũ kiểm thử chuyên nghiệp phụ trách. Vậy tại sao developer vẫn cần học? Hiểu các chỉ số, loại hình và công cụ của performance testing giúp bạn viết chương trình có performance tốt hơn. Ngoài ra, nếu developer biết performance testing, đây cũng là điểm cộng đáng kể cho hồ sơ năng lực.

Bài viết kết hợp kinh nghiệm thực tế của tác giả với kiến thức tham khảo từ đội ngũ kiểm thử và một số sách hay. Hy vọng bài viết hữu ích với bạn.

Hằng ngày, mọi người nói “load testing” nhưng nhiều khi đang gọi chung các công việc liên quan đến performance testing: dùng công cụ mô phỏng request của người dùng, quan sát throughput, response time, error rate và resource usage của hệ thống dưới các mức tải khác nhau. Khi thực hiện load testing, trước hết hãy nói rõ lần này cần kiểm chứng điều gì.

## Góc nhìn của các vai trò khác nhau về performance website

### Người dùng

Khi mở một website, người dùng quan tâm nhất điều gì? Tất nhiên là **tốc độ phản hồi của website**. Ví dụ, sau khi click trang chủ Taobao, người dùng phải chờ bao lâu để nội dung trang chủ hiện ra; sau khi click nút gửi order, phải chờ bao lâu mới nhận được kết quả.

Vì vậy, khi trải nghiệm hệ thống, người dùng thường đánh giá performance của website dựa trên tốc độ phản hồi.

### Developer

Người dùng và developer đều quan tâm đến tốc độ. Tốc độ này thực chất là **tốc độ hệ thống xử lý request của người dùng**.

Thông thường, developer khó đánh giá trực quan performance của website. Ta thường dựa vào kiến trúc hiện tại và hạ tầng của website để ước lượng sơ bộ, chẳng hạn:

1. Kiến trúc của project có phải distributed không?
2. Có sử dụng cache và message queue không?
3. Nghiệp vụ có concurrency cao đã được xử lý đặc biệt chưa?
4. Thiết kế database có hợp lý không?
5. Algorithm của hệ thống còn cần tối ưu không?
6. Hệ thống có vấn đề memory leak không?
7. Project sử dụng Redis cache có dung lượng bao nhiêu? Performance của server thế nào? Dùng hard disk cơ học hay SSD?
8. …

### Tester

Tester thường dùng công cụ performance testing để kiểm thử và lập một bảng báo cáo. Bảng này có thể bao gồm các nội dung quan trọng sau:

1. Response time;
2. Request success rate;
3. Throughput;
4. …

### Nhân viên vận hành

Nhân viên vận hành thường đánh giá performance của website dựa trên **hạ tầng và resource utilization**. Ví dụ, việc sử dụng resource của server có hợp lý không, database có bị lạm dụng không. Tất nhiên đây là vai trò vận hành theo cách truyền thống; từ khi DevOps phát triển, người chỉ làm vận hành thuần túy không còn nhiều. Ở đây tạm thời vẫn giữ vai trò này.

## Những điểm cần lưu ý khi performance testing

Hầu như không có nhiều bài viết đề cập vấn đề này khi nói về performance testing. Mọi người thường chỉ nói cách thực hiện performance testing và các chỉ số performance testing.

### Xác định rõ mục tiêu load testing

Trước khi load testing, hãy xác định rõ mục tiêu. Nếu không, rất dễ rơi vào tình trạng “tool đã chạy, report cũng có, nhưng không biết kết luận là gì”.

Có ba mục tiêu thường gặp:

1. **Đánh giá capacity trước khi go-live**: kiểm chứng machine specification, số lượng instance, cấu hình database và chiến lược cache hiện tại có đáp ứng được traffic dự kiến không.
2. **Kiểm chứng hiệu quả sau tối ưu**: xác nhận sau khi điều chỉnh slow SQL, cache, thread pool, tham số JVM hoặc kiến trúc, throughput, RT và error rate có thực sự tốt hơn không.
3. **Xác định bottleneck**: tăng tải từng bước, quan sát CPU, memory, GC, database connection pool, thread pool, MQ backlog và các chỉ số khác để tìm ra khâu đầu tiên không chịu nổi tải.

Ba mục tiêu này tương ứng với các phương pháp load testing khác nhau. Đánh giá capacity chú trọng peak level và safety margin; kiểm chứng tối ưu chú trọng so sánh trước và sau; xác định bottleneck cần monitoring và distributed tracing chi tiết hơn.

### Hiểu business scenario của hệ thống

**Trước khi performance testing, bạn càng cần hiểu business scenario hiện tại của hệ thống.** Nếu chưa hiểu đủ sâu về nghiệp vụ, ta rất dễ mắc sai lầm khi quá tập trung vào một hướng kiểm thử, từ đó bỏ qua những khu vực cần performance testing hơn.

Ví dụ, hệ thống có chức năng gửi email cho người dùng. Sau khi cấu hình email thành công, người dùng chỉ cần nhập email tương ứng là có thể gửi; hệ thống xử lý khoảng hàng chục nghìn request gửi email mỗi ngày. Nhiều người thấy vậy sẽ lập tức dùng tool để kiểm thử API gửi email. Tuy nhiên, gửi email có thể không phải bottleneck của hệ thống hiện tại. Khi có nhiều người dùng hệ thống để gửi email, thậm chí nhiều người cùng gửi, thì user management có thể mới là bottleneck.

### Dữ liệu lịch sử rất hữu ích

Dữ liệu lịch sử mà hệ thống hiện tại để lại rất quan trọng. Thông thường, ta có thể dựa vào dữ liệu lịch sử để bước đầu xác định API nào được gọi nhiều hơn và service nào chịu tải lớn nhất. Từ đó, ta có thể thực hiện performance testing và phân tích chi tiết hơn ở những nơi này.

Ngoài ra, những nơi này cũng giống như điểm yếu của hệ thống. Tối ưu tốt chúng sẽ giúp hệ thống cải thiện đáng kể.

### Request model nên gần với traffic thực tế nhất có thể

Load testing một API đơn lẻ chỉ cho biết performance của API đó trong điều kiện hiện tại, không thể trực tiếp đại diện cho capacity thực tế của hệ thống. Khi truy cập hệ thống, người dùng thật thường đăng nhập, duyệt, truy vấn, đặt order, thanh toán hoặc submit form theo trình tự, tỷ lệ và data dependency khác nhau giữa các API.

Vì vậy, script load testing nên được thiết kế theo business flow thực tế: API tần suất cao được gọi nhiều hơn, API tần suất thấp được gọi ít hơn; hệ thống đọc nhiều ghi ít không nên đặt tỷ lệ API ghi quá cao; với API cần authentication, inventory hoặc order state transition, cũng cần chuẩn bị trước test data hợp lệ. Nếu không, QPS đo được có thể rất cao nhưng khi traffic thực tế đổ vào production vẫn có thể xảy ra sự cố.

### Chuẩn bị cơ chế dừng trước

Đặc biệt với load testing trên production, trước khi bắt đầu phải chuẩn bị cơ chế dừng và các protection threshold, chẳng hạn maximum QPS, maximum concurrency, error rate threshold, P99 RT threshold, CPU/memory threshold và database connection pool threshold.

Khi hệ thống xuất hiện nhiều lỗi 5xx, RT liên tục tăng vọt, backlog của queue không giảm hoặc database connection bị cạn kiệt, phải có khả năng nhanh chóng dừng load testing hoặc tự động circuit breaker. Mục tiêu của load testing là phát hiện vấn đề, không phải đánh sập production service.

## Các chỉ số performance thường gặp

Khi load testing, không thể xem các chỉ số một cách riêng lẻ. Chỉ khi đặt RT, throughput và concurrency cạnh nhau, ta mới biết hệ thống đang xử lý request bình thường hay đã bắt đầu xếp hàng.

![Quan hệ giữa QPS, concurrency và RT](https://oss.javaguide.cn/github/javaguide/high-availability/performance-test-metrics-relation.png)

### Response time

Trước hết cần phân biệt vị trí quan sát response time. Với load testing API backend, **response time RT (Response Time)** thường là khoảng thời gian từ khi load testing client gửi request đến khi nhận response, bao gồm network transmission, gateway, server-side processing và việc trả response. Với performance của frontend page, trải nghiệm người dùng còn chịu ảnh hưởng của HTML parsing, JS execution, resource loading, rendering và interaction latency. Các vấn đề này phù hợp hơn với những tool như Lighthouse, Chrome DevTools và WebPageTest.

Average RT chỉ cho biết xu hướng tổng thể. Trải nghiệm người dùng dễ bị ảnh hưởng hơn bởi các request long-tail như P95 và P99. API load testing và production monitoring thường đồng thời theo dõi **P50, P90, P95, P99**; với core flow và khi số lượng request đủ lớn, có thể theo dõi thêm **P999**.

Ví dụ, P99 = 500ms nghĩa là 99% request trả về trong 500ms. 1% slow request còn lại có thể đến từ Cache Miss, slow SQL hoặc GC STW; chúng chiếm giữ connection và worker thread trong thời gian dài. Khi concurrency cao, thread pool bị lấp đầy, request mới bắt đầu xếp hàng hoặc bị reject; nếu upstream tiếp tục timeout và retry, traffic còn bị khuếch đại. Nhiều response cực nhanh sẽ kéo average xuống, khiến vấn đề long-tail bị average RT che lấp.

Dưới đây chỉ là ví dụ về target cho một API online có latency thấp, không phải tiêu chuẩn áp dụng cho mọi hệ thống:

| Percentile | Target ví dụ | Mô tả                                             |
| ---------- | ------------ | ------------------------------------------------- |
| P90        | < 200ms      | Trải nghiệm bình thường của phần lớn người dùng   |
| P99        | < 500ms      | Trải nghiệm long-tail và rủi ro chiếm dụng thread |
| P999       | < 1s         | Long-tail cực đoan của core flow có traffic cao   |

Threshold tốt nhất nên được suy ra từ business SLO, yêu cầu trải nghiệm người dùng, timeout của upstream, năng lực của downstream dependency và historical baseline. Ngoài ra, P999 khá nhạy với số lượng sample. Nếu một lần test chỉ có vài nghìn request, P999 rất dễ bị ảnh hưởng bởi dao động ngẫu nhiên; nếu QPS của API vốn thấp, ý nghĩa thống kê của P999 cũng hạn chế. Với API loại này, có thể xem P99 trước hoặc kéo dài thời gian load testing để thu thập thêm sample.

> **Failure mode**: Khi dao động ngẫu nhiên của downstream khiến P999 tăng vọt, nếu upstream không có timeout, rate limiting và circuit breaker hợp lý, request sẽ chiếm giữ connection, thread hoặc coroutine trong thời gian dài. Nếu tiếp tục retry, có thể xảy ra cạn kiệt thread pool, cạn kiệt connection pool, backlog của queue tăng, error rate tăng; chỉ trong trường hợp nghiêm trọng mới có thể trigger OOM.

### Concurrency

**Concurrency là số lượng request hệ thống đang xử lý tại cùng một thời điểm.**

Concurrency phản ánh **khả năng chịu tải** của hệ thống. Cần phân biệt các khái niệm sau:

- **Concurrent user**: số lượng user online cùng lúc.
- **Concurrent request**: số lượng request hệ thống đang xử lý tại cùng một thời điểm.
- **Maximum concurrency**: số lượng concurrent request tối đa mà hệ thống có thể chịu được; vượt quá giá trị này, performance của hệ thống có thể suy giảm hoặc crash.

### RPS, QPS và TPS

- **RPS (Requests Per Second)**: số lượng HTTP/RPC request server xử lý mỗi giây, phù hợp hơn với API load testing.
- **QPS (Queries Per Second)**: số query server xử lý mỗi giây, thường dùng cho database query và search query, đồng thời cũng thường được dùng rộng để chỉ request throughput.
- **TPS (Transactions Per Second)**: số business transaction server hoàn tất mỗi giây, chẳng hạn một lần đặt order, một lần thanh toán hoặc một lần đăng ký.

> QPS vs TPS: TPS thiên về business semantics hơn. Ví dụ, một lần đặt order có thể tính là 1 TPS, nhưng phía sau có thể gồm 4 HTTP/RPC request: query inventory, tạo order, trừ inventory và gọi payment. Với backend service, đây có thể là 4 RPS và còn tạo ra nhiều database QPS hơn.

### Throughput

**Throughput** là số lượng request hệ thống xử lý trong một đơn vị thời gian. RPS, QPS và TPS đều là các chỉ số định lượng thường dùng.

**Little's Law (định luật Little)**: trong steady state khi hệ thống chưa bão hòa, `average in-flight request ≈ throughput × average response time`. Concurrency ở đây là số request trung bình đang được xử lý hoặc xếp hàng trong hệ thống, không phải số user online; RT dùng average response time, không phải P99; throughput dùng average completion rate của request.

Ví dụ, QPS trung bình là 500, RT trung bình là 200ms (0.2s), số request in-flight trung bình trong hệ thống xấp xỉ `500 × 0.2 = 100`.

Công thức này chỉ dùng được trong vùng response tuyến tính. Khi tiếp tục tăng concurrency, CPU scheduling, lock contention và thời gian xếp hàng cùng tăng mạnh; RT thường tăng phi tuyến, throughput đạt điểm uốn rồi đi vào plateau hoặc giảm xuống. “Hockey Stick Curve” trong hình minh họa hiện tượng này: sau điểm uốn, QPS không tăng mà giảm, nghĩa là hệ thống đã đi vào vùng phi tuyến.

![Hockey Stick Curve giữa QPS và concurrency](https://oss.javaguide.cn/github/javaguide/high-availability/performance-test-hockey-stick-curve.png)

Vì vậy, production capacity không thể chỉ dựa vào công thức để suy ra, mà còn phải được kiểm chứng bằng full-link load testing để xác định giới hạn thực tế.

## Các chỉ số activity của hệ thống

### PV (Page View)

**Lượt truy cập**, tức số page view hoặc click, dùng để đo số web page người dùng truy cập. Trong một khoảng thời gian thống kê, mỗi lần user mở hoặc refresh một page sẽ ghi nhận 1 lần; mở hoặc refresh cùng một page nhiều lần thì page view được cộng dồn. PV được thống kê từ góc độ số lần mở hoặc refresh web page.

### UV (Unique Visitor)

**Unique visitor**, dùng để thống kê số user truy cập một website trong 1 ngày. Một visitor truy cập website nhiều lần trong 1 ngày chỉ được tính là 1 unique visitor. UV được thống kê từ góc độ từng user.

### DAU (Daily Active User)

**Số người dùng hoạt động hằng ngày**, tức số user đăng nhập hoặc sử dụng product trong một ngày (đã loại trùng).

### MAU (Monthly Active Users)

**Số người dùng hoạt động hằng tháng**, tức số user đăng nhập hoặc sử dụng product trong một tháng (đã loại trùng).

### Ví dụ tính toán thực tế

> **Đánh giá production capacity**: Chỉ dùng DAU nhân với một hệ số cố định để ước tính peak rất dễ đánh giá thấp traffic của campaign. Các scenario như flash sale đúng giờ hoặc mở bán lớn thường có peak khác hẳn traffic trung bình ngày. Có thể dùng request volume trung bình ngày trong lịch sử để ước tính QPS trung bình ngày, sau đó kết hợp với peak multiplier của nghiệp vụ (chẳng hạn 3～10 lần trung bình ngày, tùy nghiệp vụ) để có peak QPS sơ bộ. Cuối cùng, dùng Little's Law để suy ra concurrency mục tiêu và cộng thêm safety margin 1.5～2 lần. Kết quả này chỉ nên là điểm bắt đầu của target load testing; cuối cùng vẫn cần thực hiện **full-link load testing** (kết hợp ghi và replay traffic thực tế, chẳng hạn [GoReplay](https://goreplay.org/)) để nắm throughput thực tế, thay vì chỉ dừng ở ước tính trên giấy.

## Phân loại performance testing

| Loại test          | Mục đích                                                           | Cách làm thường gặp                                                             |
| ------------------ | ------------------------------------------------------------------ | ------------------------------------------------------------------------------- |
| **Benchmark**      | Thiết lập performance baseline để tiện so sánh trước sau           | Cố định environment, script và data                                             |
| **Load Test**      | Kiểm chứng hệ thống có đạt yêu cầu dưới expected load không        | Tăng tải từng bước theo traffic model ước tính đến target level                 |
| **Capacity Test**  | Tìm giới hạn tải hiện tại của hệ thống                             | Tăng tải từng bước, quan sát throughput inflection point và resource bottleneck |
| **Stress Test**    | Quan sát degradation, protection và recovery sau khi vượt giới hạn | Vượt expected load, kiểm chứng rate limiting, circuit breaker và self-healing   |
| **Soak/Endurance** | Kiểm chứng stability khi chạy trong thời gian dài                  | Chạy liên tục trong thời gian dài với tải gần traffic thực tế                   |
| **Spike Test**     | Kiểm chứng khả năng chịu traffic tăng đột biến                     | Tăng tải nhanh trong thời gian ngắn rồi nhanh chóng hạ xuống, quan sát recovery |

![Phân loại performance testing: phân biệt các mức stress](https://oss.javaguide.cn/github/javaguide/high-availability/performance-test-type-boundaries.webp)

Điểm khác nhau chính của các loại test này là mức tải được tăng đến đâu: load testing xem target traffic có chịu được không; capacity testing tiếp tục tăng tải để tìm inflection point; stress testing vượt expected load để xem hệ thống fail và recover như thế nào.

### Benchmark

Benchmark nhấn mạnh việc cố định environment, script, data và parameter để tiện so sánh thay đổi trước và sau khi tối ưu. Ví dụ, chạy cùng một API trên cùng một machine trước và sau khi thay đổi index, rồi xem average RT, P99, throughput và CPU level có thay đổi không.

### Load testing

Load testing kiểm chứng hệ thống có đạt yêu cầu dưới traffic model dự kiến không. Ví dụ, dựa trên tỷ lệ lịch sử production để kết hợp các request login, query, đặt order và payment, tăng tải từng bước đến target QPS, rồi xem response time, error rate và resource usage có đáp ứng yêu cầu không.

### Capacity testing

Capacity testing tiếp tục tăng request load cho đến khi một resource nào đó của server bão hòa, hoặc response time và error rate của hệ thống không còn đạt yêu cầu. Loại test này chú trọng hơn đến capacity limit dưới kiến trúc và resource configuration hiện tại.

### Stress testing

Stress testing cố ý đẩy traffic vượt design level. Ở giai đoạn này, có thể chấp nhận fail fast, chẳng hạn trả về 429 hoặc 503; nhưng không thể chấp nhận request treo mãi, connection không được giải phóng, thread pool xếp hàng, queue tăng liên tục trong khi upstream vẫn tiếp tục retry. Khi test, cần xem các protection như token bucket, semaphore, queue length limit và circuit breaker/degradation có thực sự hoạt động không; sau khi rút traffic, connection, thread, queue và throughput có trở về mức bình thường không.

### Stability testing

Stability testing chủ yếu quan sát xu hướng trong thời gian dài. Hệ thống có thể hoàn toàn bình thường lúc mới khởi động nhưng chỉ bộc lộ vấn đề sau vài giờ chạy; những vấn đề này thường được phát hiện ở đây. Có thể chạy traffic gần với tỷ lệ production trong vài giờ, một ngày hoặc lâu hơn, tập trung theo dõi xu hướng: heap memory có chỉ tăng mà không giảm không, Full GC có xuất hiện dày hơn không, connection pool có bị lấp đầy trong thời gian dài không, số thread có liên tục tăng không, MQ hoặc local queue có tích tụ ngày càng nhiều không, error rate có tăng dần không. 7×24 giờ chỉ là cách làm thường gặp trong scenario yêu cầu cao, không phải đáp án cố định.

### Spike testing

Spike testing phù hợp để mô phỏng traffic đột ngột tràn vào khi bắt đầu campaign, flash sale đúng giờ hoặc push message. Khi load testing, trước hết kéo traffic lên cao trong thời gian rất ngắn, sau đó hạ về mức bình thường; quan sát rate limiting, queue và degradation có được trigger như dự kiến không, cũng như sau khi traffic giảm, RT, error rate, thread pool và queue có giảm xuống không.

## Full-link load testing

Load testing một API đơn lẻ chỉ cho biết API đó có thể đạt throughput bao nhiêu với script hiện tại. Khi hệ thống phức tạp xảy ra sự cố, bottleneck thường không nằm ở entry API: gateway authentication không có vấn đề nhưng inventory service bắt đầu xếp hàng; order API vẫn trả về nhưng MQ đã backlog; CPU của application không cao nhưng database connection pool đã bị lấp đầy. Một lần đặt order có thể đi qua gateway, authentication, product, inventory, order, payment, message queue, cache và database. Bất kỳ khâu nào chạm bottleneck trước đều sẽ phản ánh lên request của người dùng.

![Full-link load testing: dựng guardrail trước](https://oss.javaguide.cn/github/javaguide/high-availability/full-link-performance-test-guardrails.webp)

Khi thực hiện full-link load testing, traffic đi vào từ system entry rồi đi theo business flow thực tế qua các bước tiếp theo; mọi service, component và infrastructure đều chịu tải. Cách này gần với traffic production hơn load testing API đơn lẻ, nhưng cũng dễ gây ảnh hưởng nhầm đến data và service thật hơn. Vì vậy, trước hết cần chuẩn bị:

1. **Tách biệt test data**: sử dụng test account, test marker, shadow table hoặc shadow database để tránh làm nhiễm order, payment, inventory và các core data thật.
2. **Nhận diện test traffic**: để gateway, service, log, monitoring và distributed tracing đều nhận diện được test traffic, thuận tiện cho việc thống kê và giảm thiệt hại nhanh chóng.
3. **Monitoring đầy đủ trên toàn flow**: đồng thời quan sát application metrics, machine metrics, database metrics, cache metrics, MQ backlog, slow SQL, GC và trạng thái thread pool.
4. **Circuit breaker và rate limiting khả dụng**: trước load testing, xác nhận rate limiting, degradation, circuit breaker, queue length limit và các protection mechanism khác đã có hiệu lực.
5. **Có thể review kết quả load testing**: ghi lại concurrency, QPS, P90/P99, error rate, resource level và bottleneck point của từng vòng load testing để tiện so sánh với kết quả sau tối ưu.

Nếu nghiệp vụ chưa đến mức cần platform full-link load testing trên production, trước hết dùng JMeter, Gatling, wrk và các tool khác để làm rõ script của core flow. Chỉ cần có thể xác định bottleneck, bảo vệ data và dừng nhanh là đã bao phủ phần lớn scenario đánh giá capacity và xác định bottleneck. Khi nghiệp vụ phức tạp đến mức cần load testing trên production thường xuyên, hãy cân nhắc xây dựng platform chuyên dụng.

## Các công cụ performance testing thường dùng

### Công cụ thường dùng cho backend

Vì system design có liên quan đến vấn đề system performance, interviewer rất có thể hỏi trong buổi phỏng vấn: **Bạn thực hiện performance testing như thế nào?**

Một số công cụ performance testing thường dùng:

| Tool           | Language/script                             | Đặc điểm                                                                                    | Scenario phù hợp                                              |
| -------------- | ------------------------------------------- | ------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| **JMeter**     | Java, GUI + CLI                             | Tính năng đầy đủ, GUI phù hợp để viết và debug script, load testing chính thức nên dùng CLI | Enterprise scenario phức tạp, API flow load testing           |
| **Gatling**    | Java, JavaScript, TypeScript, Scala, Kotlin | Code-driven, report và CI integration thân thiện                                            | Team developer duy trì script load testing, CI/CD integration |
| **k6**         | JavaScript script, Go engine                | Metrics, threshold và CI integration thân thiện                                             | API load testing, SLO regression                              |
| **Locust**     | Python                                      | Viết user behavior bằng Python, hỗ trợ distributed extension                                | Python team, modeling user behavior phức tạp                  |
| **wrk**        | C, hỗ trợ Lua script                        | Nhẹ và performance cao                                                                      | HTTP API high-concurrency benchmark                           |
| **ab**         | C                                           | Đơn giản, nhẹ, đi kèm Apache                                                                | Smoke test nhanh hoặc benchmark sơ bộ                         |
| **LoadRunner** | Commercial suite                            | Hỗ trợ protocol và năng lực quản lý mạnh                                                    | Enterprise large-scale testing                                |

Nếu tôi nhớ không nhầm, ngoài **LoadRunner**, các tool phía trên đều có bản open source hoặc miễn phí.

**Gợi ý chọn tool:**

- **Kiểm chứng nhanh**: dùng `ab` hoặc `wrk` để load testing API đơn giản.
- **Scenario phức tạp**: dùng `JMeter`, hỗ trợ record script, parameterization, assertion và các tính năng khác.
- **Code-driven**: dùng `Gatling` hoặc `k6`, phù hợp để developer duy trì script và tích hợp CI.
- **Modeling user behavior phức tạp**: dùng `Locust`, phù hợp để mô tả user behavior bằng Python.

### Công cụ thường dùng cho frontend

| Tool                            | Mục đích chính                                                                                    |
| ------------------------------- | ------------------------------------------------------------------------------------------------- |
| **Chrome DevTools Performance** | Record và phân tích performance runtime của page, long task, JS execution và rendering bottleneck |
| **Lighthouse**                  | Tự động audit page performance, accessibility, best practice, SEO và tạo report                   |
| **WebPageTest**                 | Kiểm thử trải nghiệm page loading từ góc nhìn browser thật, network hoặc khu vực khác nhau        |
| **Fiddler/Charles**             | Bắt packet, sửa packet và debug HTTP request                                                      |

## Các chiến lược tối ưu performance thường gặp

Trước khi tối ưu, hãy làm rõ request flow: entry gateway, application thread pool, cache, database, MQ và external API; nơi nào xếp hàng trước thì kiểm tra nơi đó trước.

Dưới đây là một số câu hỏi tôi thường tự hỏi khi tối ưu performance:

| Hướng tối ưu     | Hạng mục kiểm tra                                                                                       |
| ---------------- | ------------------------------------------------------------------------------------------------------- |
| **Cache**        | Hệ thống có cần cache không? Hot data đã được cache chưa?                                               |
| **Architecture** | Bản thân system architecture có vấn đề không? Có cần read/write splitting hoặc database sharding không? |
| **Concurrency**  | Hệ thống có nơi nào deadlock không? Lock granularity có hợp lý không?                                   |
| **Memory**       | Hệ thống có memory leak không? GC có thường xuyên không?                                                |
| **Database**     | Việc sử dụng database index có hợp lý không? Có slow SQL không?                                         |
| **Algorithm**    | Time complexity của core algorithm có thể tối ưu không?                                                 |
| **IO**           | Có network call không cần thiết không? Có thể thực hiện batch operation không?                          |

<!-- @include: @article-footer.snippet.md -->
