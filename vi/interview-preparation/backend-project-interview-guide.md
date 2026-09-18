---
title: "Trình bày project backend trong phỏng vấn thế nào? Từ giới thiệu project đến technical challenge và retrospective sự cố"
description: Hướng dẫn chuẩn bị phỏng vấn project Java backend, trình bày rõ phần giới thiệu project, trách nhiệm cá nhân, core flow, lựa chọn công nghệ, tối ưu performance, sự cố production, metric định lượng và cách chuẩn bị cho các câu hỏi đào sâu thường gặp.
category: Interview Preparation
tag:
  - Java Interview
  - Backend Interview
  - Project Experience
  - Project Deep Dive
sitemap:
  changefreq: monthly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: phỏng vấn project backend,phỏng vấn project Java,giới thiệu project,đào sâu project,lựa chọn công nghệ,điểm khó của project,sự cố production,tối ưu performance,phỏng vấn Java
---

“Hãy giới thiệu project bạn đã làm.”

“Đây là một project microservice phát triển dựa trên Spring Boot, sử dụng MySQL, Redis, Kafka, Elasticsearch……”

Nhiều phần giới thiệu project dừng lại ở đây. Khi interviewer hỏi tiếp “Tại sao lại dùng Kafka?”, bạn rất dễ bị khựng lại. Các câu hỏi phỏng vấn thường gặp về Java, MySQL, Redis có thể học thuộc trước, nhưng câu hỏi đào sâu về project sẽ luôn quay về các ràng buộc nghiệp vụ thực tế, implementation trong code và kết quả verification; học thuộc một bài nói cố định chỉ đủ ứng phó phần mở đầu.

## Interviewer muốn tìm hiểu gì từ project?

Interviewer sẽ xác nhận vài điều từ project: bạn có hiểu business của project và một request được lưu chuyển thế nào không; bạn cụ thể phụ trách code, data và các hệ thống upstream/downstream nào; khi gặp vấn đề, bạn định vị nguyên nhân, so sánh các phương án và verify kết quả ra sao; các công nghệ và metric trong CV có chịu được câu hỏi đào sâu không.

Phỏng vấn tuyển dụng sinh viên mới tốt nghiệp không yêu cầu mọi project phải đạt độ phức tạp của production system lớn. Một monolith do bạn tự làm và có thể trình bày thấu đáo thường đáng tin cậy hơn một project microservice dựng theo tutorial. Phỏng vấn người đã đi làm sẽ tiếp tục hỏi về traffic, capacity, sự cố, gray release, rollback và phối hợp team; khi trả lời, bạn cần đưa ra nhiều bằng chứng production hơn.

## Chuẩn bị một bản nháp project trước buổi phỏng vấn

Với mỗi project trọng tâm trong CV, hãy chuẩn bị riêng một bản nháp; không cần viết thành bài văn, chỉ cần bạn hiểu và có thể nhanh chóng nhớ lại trước buổi phỏng vấn.

| Nội dung cần chuẩn bị | Câu hỏi cần trả lời                                                                                                    |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Bối cảnh business     | Project dành cho ai? Giải quyết vấn đề gì? Business flow quan trọng nhất là gì?                                        |
| Phạm vi system        | Có những module nào? Phụ thuộc vào external system nào? Data đi từ đâu đến đâu?                                        |
| Trách nhiệm cá nhân   | Những requirement, API hoặc module nào do bạn phụ trách? Bạn tham gia ở mức độ nào?                                    |
| Core flow             | Một request đi qua những service, cache, database và message queue nào?                                                |
| Lựa chọn công nghệ    | Tại sao sử dụng phương án hiện tại? Đã so sánh những phương án nào? Đã phải trả giá gì?                                |
| Điểm khó và sự cố     | Đã gặp vấn đề cụ thể nào? Định vị, sửa chữa và verify thế nào?                                                         |
| Metric của project    | Có những record đáng tin cậy nào về traffic, latency, error rate, data volume, resource consumption và kết quả tối ưu? |

Ở cuối bản nháp, hãy dành riêng một mục ghi những phần bạn không tham gia. Ví dụ, bạn thiết kế database table còn deploy và capacity planning do team infrastructure phụ trách thì hãy trả lời đúng thực tế. Interviewer thường chấp nhận phạm vi trách nhiệm có giới hạn, còn rủi ro bịa ra kinh nghiệm tham gia lại lớn hơn.

## Nên trình bày project thế nào?

Cần chuẩn bị cả phiên bản 30 giây và 3 phút. Khi interviewer chỉ muốn nhanh chóng hiểu project, hãy dùng phiên bản ngắn; khi đối phương yêu cầu giới thiệu chi tiết, hãy bổ sung architecture, trách nhiệm và công việc trọng tâm.

### Phiên bản 30 giây

Phiên bản 30 giây chỉ giữ lại bốn nội dung: project giải quyết vấn đề gì, user hoặc business chính là ai, trách nhiệm của bạn, và một công việc bạn đã chuẩn bị để trình bày sâu.

Ví dụ:

> Đây là một order system dành cho nhân viên procurement nội bộ doanh nghiệp, chủ yếu bao phủ tra cứu sản phẩm, đặt hàng, approval và order fulfillment. Trong project, tôi phụ trách hai flow tạo order và đóng order quá hạn, bao gồm thiết kế table, phát triển API, xử lý idempotent và tích hợp monitoring. Công việc tôi dành nhiều thời gian nhất trong project là tối ưu performance của flow tạo order; sau đây tôi có thể trình bày chi tiết cách tôi định vị slow request khi đó.

Đoạn nói này không liệt kê tất cả middleware, nhưng để lại cho interviewer một số điểm có thể hỏi tiếp: trạng thái order, idempotent, đóng order quá hạn và tối ưu performance.

### Phiên bản 3 phút

Phiên bản 3 phút triển khai theo thứ tự sau:

1. Bối cảnh business và user của project.
2. System gồm những module chính nào, core request được lưu chuyển thế nào.
3. Module và phạm vi trách nhiệm bạn phụ trách.
4. Một hoặc hai điểm khó hay kết quả có bằng chứng.

Vẫn lấy order system làm ví dụ:

> System chủ yếu phục vụ nhân viên procurement và finance, phụ trách tra cứu sản phẩm, tạo order, approval, đồng bộ trạng thái thanh toán và tra cứu fulfillment. Backend được tách thành các module order, inventory và approval; khi tạo order, trước tiên sẽ validate request và price, sau đó tạo order và reserve inventory. Sau khi thành công, system gửi message để downstream hoàn thành các task async như thông báo approval.
>
> Tôi phụ trách tạo order và đóng order quá hạn. Tạo order cần xử lý duplicate submission, thiếu inventory và gửi message thất bại; đóng order quá hạn cần tránh đóng những order đã thanh toán. Tôi chủ yếu hoàn thành API, state transition, idempotent và compensation task, đồng thời tích hợp monitoring liên quan. Sau đó, sau một lần release version, tail latency của API tra cứu order tăng lên; tôi tham gia định vị và tối ưu vấn đề. Dựa trên Trace và slow SQL, chúng tôi phát hiện query condition không khớp với composite index, sau khi điều chỉnh index lại thực hiện comparative load test trên cùng data volume.

Đây là một ví dụ, đừng chỉ đổi tên project rồi đưa trách nhiệm và sự cố trong đó vào CV. Project của bạn không có message queue thì hãy nói về synchronous call; nếu không có data load test thực tế, bạn cũng có thể nêu rõ test environment và phương pháp verification, đừng tạm thời bịa ra một QPS.

## Nên trình bày architecture diagram thế nào?

Architecture diagram phù hợp để trình bày theo một request thực tế. User request đi vào từ gateway, đi qua service nào, đọc cache và database nào, task nào đi vào message queue, sau khi thất bại thì xử lý ra sao. Sau khi trình bày main flow, hãy bổ sung thêm một exception flow.

Mỗi khi xuất hiện một component trong architecture, tốt nhất bạn có thể trả lời ba câu hỏi:

- Nó đảm nhiệm công việc gì trong flow này?
- Nếu bỏ nó đi thì chuyện gì xảy ra?
- Khi nó không khả dụng, system xử lý thế nào?

Nếu project sử dụng Redis để cache thông tin sản phẩm, bạn còn phải chuẩn bị về database query sau khi cache miss, cache expiration strategy, hot data, data consistency và cách degrade khi Redis gặp sự cố. Chỉ trả lời “Redis có performance cao nên dùng Redis” thường sẽ nhanh chóng đi vào vùng mù kiến thức.

Đừng vẽ architecture diagram quá lớn. Nhét gateway, service registry, config center, hơn mười microservice và toàn bộ middleware vào một diagram sẽ khiến bạn khó tìm được trọng tâm khi trình bày. Architecture diagram dùng trong phỏng vấn chỉ cần giữ phạm vi project và một core flow; với project phức tạp, bạn có thể chuẩn bị thêm một module diagram hoặc sequence diagram.

## Làm rõ trách nhiệm của mình thế nào?

“Phụ trách phát triển order module” cung cấp rất ít thông tin. Hãy tiếp tục làm rõ phạm vi requirement, phạm vi code và phạm vi phối hợp:

- Phạm vi requirement: tạo order, hủy order, đóng order quá hạn, hay toàn bộ order domain.
- Phạm vi code: bạn tham gia ở mức độ nào vào API, table structure, state machine, scheduled task và message consumer.
- Phạm vi phối hợp: bạn có tham gia solution review không, phối hợp với inventory, payment, test và operation thế nào.
- Phạm vi kết quả: bạn có theo dõi release, gray release, monitoring và maintenance sau đó không.

Với project tuyển dụng sinh viên mới tốt nghiệp, hãy nói thẳng đây là personal project hoặc course project, cũng như phần nào tham khảo tutorial. Nếu bạn từng thêm feature, bổ sung test hoặc sửa table structure trên nền tutorial thì hãy tập trung trình bày những thay đổi đó. Interviewer quan tâm bạn có thực sự tự làm và suy nghĩ hay không, không cần biến personal project thành production system của tập đoàn lớn.

## Trả lời về lựa chọn công nghệ thế nào?

Khi trả lời về lựa chọn công nghệ, hãy trình bày đầy đủ problem, constraint, các phương án ứng viên và kết quả verification.

Giả sử interviewer hỏi: “Tại sao đóng order quá hạn lại dùng delayed message?”

Khi trả lời, cần nêu rõ:

1. Sau khi tạo order cần kiểm tra trạng thái thanh toán vào thời điểm chỉ định; task volume và yêu cầu về độ chính xác của latency là gì.
2. Database scan định kỳ, Redis expiration notification, time wheel và delayed message có những giới hạn nào.
3. Tại sao project hiện tại chọn delayed message; infrastructure hiện có, operation cost và kinh nghiệm của team có ảnh hưởng đến quyết định không.
4. Khi message bị duplicate, delayed, lost hoặc consumer gặp sự cố thì xử lý thế nào.

Không có điều kiện của project thì lựa chọn công nghệ không có đáp án tiêu chuẩn. Với project nhỏ, dùng scheduled task scan các order đang chờ thanh toán, kết hợp index phù hợp và sharding để xử lý, có thể đã đủ dùng; khi team đã có message queue mature và order volume tương đối lớn thì có thể cân nhắc delayed message. Khi trả lời phỏng vấn, cần nói rõ tại sao trong điều kiện hiện tại lại chọn như vậy, đồng thời thừa nhận giới hạn của phương án.

Việc đưa Redis, Kafka hoặc Elasticsearch vào chỉ cho thấy project sử dụng các component này. Chỉ khi có thể giải thích các vấn đề sau mới chứng minh bạn nắm được phần công việc đó:

- Redis cache những data nào, thiết kế key ra sao, xác định expiration time thế nào.
- Thiết kế Topic và partition của Kafka ra sao, xử lý production failure và duplicate consumption thế nào.
- Modeling document trong Elasticsearch thế nào, cập nhật index ra sao, tại sao query result đáng tin cậy.
- Sau khi sharding, chọn shard key thế nào, xử lý scale out và cross-shard query ra sao.

Khi project chưa đạt quy mô tương ứng, có thể trình bày một công nghệ nào đó như nội dung học tập và thử nghiệm, đừng tuyên bố nó đã giải quyết một production bottleneck vốn không tồn tại.

## Nên trình bày điểm khó của project thế nào?

“Thời gian khá gấp” hay “requirement thường xuyên thay đổi” đúng là làm tăng độ khó công việc, nhưng technical interview thường muốn nghe một vấn đề engineering có thể tiếp tục được hỏi sâu.

Hãy tìm tư liệu từ những việc bạn thực sự đã làm, thường gồm:

- Vấn đề correctness: duplicate order, overselling inventory, state disorder, amount precision, data inconsistency.
- Vấn đề performance: slow SQL, cache hit rate giảm, lock contention, thread pool backlog, GC pause.
- Vấn đề stability: dependency timeout, message backlog, connection pool cạn, release failure, traffic tăng đột biến.
- Vấn đề engineering: refactor legacy code, gray migration, tương thích với old data, thay đổi API cross-team.

Ít nhất một điểm khó cần trình bày rõ quy trình sau:

```text
Hiện tượng và ảnh hưởng → Constraint đã biết → Troubleshooting hoặc analysis → So sánh phương án
→ Quá trình triển khai → Kết quả verification → Vấn đề còn lại
```

Ví dụ “API query rất chậm” vẫn chưa phải một điểm khó hoàn chỉnh. Hãy nói tiếp request nào chậm, bắt đầu từ khi nào, P95/P99 thay đổi ra sao, database scan bao nhiêu row, cuối cùng sửa index hay query logic, và verify thế nào trên cùng data volume và traffic. Nếu không lưu lại số liệu khi đó, hãy nêu rõ đã quan sát những metric nào và kết luận dựa trên bằng chứng gì.

## Nên trình bày tối ưu performance thế nào?

Tối ưu performance trong project rất dễ bị hỏi sâu, vì sau câu “thời gian xử lý API giảm từ 2 giây xuống 200 ms” còn rất nhiều câu hỏi:

- 2 giây và 200 ms lần lượt được đo ở đâu?
- Data volume, concurrency và machine configuration có giống nhau không?
- Đang xem average latency hay P95/P99?
- Bottleneck thực sự nằm ở application, database, cache hay downstream service?
- Sau khi tối ưu có tăng rủi ro data consistency và maintenance cost không?

Khi chuẩn bị case tối ưu performance, hãy giữ lại một bộ tài liệu có thể đối chiếu: Trace, execution plan, GC log, load test configuration, monitoring screenshot hoặc test report trước và sau tối ưu. Nếu không tiện mang tài liệu của công ty đi, có thể ghi lại conclusion và quá trình troubleshooting đã được ẩn thông tin nhạy cảm.

Khi trả lời, hãy nối các thông tin này lại:

> API nào xuất hiện vấn đề ở traffic và data volume nào; tôi dựa vào metric nào để thu hẹp vấn đề xuống component nào; đã so sánh những phương án nào, cuối cùng thay đổi gì; sử dụng environment và metric nào để verify; sau khi release tiếp tục theo dõi gì; thay đổi này mang lại những cost nào.

Cache, async và parallel processing thường có thể cải thiện response time, nhưng cũng đưa vào các vấn đề về cache consistency, message reliability, thread safety và downstream pressure. Nêu rõ những cost này sẽ đáng tin cậy hơn việc chỉ nhấn mạnh con số performance.

## Nên trình bày production incident thế nào?

Hãy trả lời một incident theo thứ tự thời gian:

1. **Phát hiện vấn đề**: alert, user feedback hoặc quan sát khi release đã phát hiện điều gì.
2. **Xác nhận ảnh hưởng**: API, user và instance nào bị ảnh hưởng, error rate và latency thay đổi thế nào.
3. **Khẩn cấp giảm ảnh hưởng**: rollback, loại instance khỏi traffic, rate limiting, degrade hoặc tắt feature.
4. **Lưu lại evidence**: log, Trace, thread dump, Heap Dump, GC log và change record.
5. **Định vị root cause**: đã đưa ra những hypothesis nào, loại trừ hướng sai ra sao, cuối cùng dùng evidence nào để xác nhận.
6. **Sửa chữa và verification**: đã thay đổi code hoặc config gì, test, gray release và quan sát thế nào.
7. **Tránh tái diễn**: đã bổ sung monitoring, test, capacity limit hoặc release check nào.

Khi troubleshooting có áp dụng temporary measure, cũng cần nói rõ side effect của nó. Ví dụ restart instance có thể tạm thời khôi phục service nhưng sẽ làm mất hiện trường; scale out mù quáng có thể tiếp tục đẩy pressure sang database. Có thể tham khảo phương pháp troubleshooting hoàn chỉnh tại: [Troubleshooting vấn đề production của Java backend](../java/jvm/jvm-in-action.md).

## Chuẩn bị metric của project thế nào?

Metric cần thể hiện quy mô project, ảnh hưởng của vấn đề hoặc kết quả thay đổi; không cần chất đống con số chỉ để làm project có vẻ lớn.

| Metric                                | Phù hợp để mô tả điều gì                   | Hiểu lầm thường gặp                                                     |
| ------------------------------------- | ------------------------------------------ | ----------------------------------------------------------------------- |
| QPS/TPS                               | Traffic và system throughput               | Chỉ báo peak mà không nói rõ vị trí thống kê và time range              |
| P95/P99                               | Trải nghiệm của slow request               | Chỉ xem average response time                                           |
| Error rate                            | Tình trạng request failure                 | Trộn business rejection với system error                                |
| Data volume                           | Quy mô query, storage và migration         | Chỉ nói tổng volume mà không nói tốc độ tăng trưởng và phân bố hot/cold |
| CPU, memory, GC                       | Resource của application và trạng thái JVM | Chỉ báo số liệu sau tối ưu mà không có điều kiện đối chiếu              |
| Message backlog, connection pool wait | Internal queue của system                  | Chỉ scale out mà không xác nhận năng lực xử lý của downstream           |

Khi project thực tế không có metric sẵn, có thể đo bổ sung trong test environment, nhưng cần nói rõ data đến từ test environment. Machine configuration, data volume, concurrency model và test duration cũng cần được ghi lại cùng nhau; không được giả mạo test number thành production data.

## Chuẩn bị cho các câu hỏi đào sâu thường gặp về project thế nào?

Đặt CV trước mặt và lần lượt đặt câu hỏi theo từng technology stack trong đó. Nhóm câu hỏi sau phù hợp với phần lớn project Java backend:

### Business và architecture

- Business flow quan trọng nhất của project là gì?
- Component nào trong system dễ xảy ra vấn đề nhất?
- Nếu traffic tăng lên gấp vài lần hiện tại, bottleneck có thể xuất hiện đầu tiên ở đâu?
- Chọn monolith hay microservice thế nào? Sau khi tách service đã tăng thêm những cost nào?

### Database và cache

- Core table được thiết kế thế nào? Tại sao chọn primary key và index này?
- Phát hiện slow SQL thế nào? Đã xem những field nào trong execution plan?
- Data nào được đưa vào cache? Xử lý thế nào sau khi cache invalidation?
- Khi cache và database không nhất quán, business có thể chấp nhận trong bao lâu?

### Message và consistency

- Tại sao phải gửi message này, synchronous call có khả thi không?
- Production failure, duplicate processing của consumer và message backlog lần lượt xử lý thế nào?
- Xử lý thế nào khi local transaction commit thành công nhưng gửi message thất bại?
- Idempotent của API phụ thuộc vào business unique identifier nào?

### Concurrency và stability

- Dựa vào đâu để thiết lập thread pool parameter? Chuyện gì xảy ra khi queue đầy?
- Khi downstream API chậm đi, timeout, retry, rate limiting và circuit breaker phối hợp thế nào?
- Khi Redis, database hoặc message queue không khả dụng, project vẫn có thể cung cấp những capability nào?
- Khi release xảy ra vấn đề, rollback thế nào và xác nhận data không bị hỏng ra sao?

Không cần chuẩn bị một standard answer cho mọi câu hỏi. Hãy liên hệ câu hỏi với code, table structure, config và monitoring của mình; khi trả lời sẽ tự nhiên hơn nhiều.

## Trọng tâm chuẩn bị cho tuyển dụng sinh viên mới tốt nghiệp và người đã đi làm khác nhau thế nào?

Project của sinh viên mới tốt nghiệp thường tiếp tục bị hỏi về kiến thức cơ bản và implementation detail. Ví dụ khi bạn sử dụng HashMap, thread pool, Redis, interviewer có thể chuyển từ project sang data structure, concurrency và vấn đề cache. Khi chuẩn bị, cần bảo đảm những công nghệ xuất hiện trong CV đều nắm được nguyên lý cơ bản và có thể tìm thấy code tương ứng.

Phỏng vấn người đã đi làm sẽ quan tâm đến quy mô project và engineering trade-off: ai đưa ra requirement, solution được review thế nào, release được gray release ra sao, incident được xử lý thế nào, metric có được cải thiện không, và phối hợp với team khác thế nào. Chỉ nói implementation trong code thường chưa đủ, còn cần bổ sung solution và phần sau release.

Số năm làm việc càng dài, interviewer càng có thể hỏi sâu “Tại sao khi đó không chọn phương án khác?” và “Nếu làm lại thì sẽ sửa thế nào?”. Với những câu hỏi này, có thể thành thật trình bày điều kiện lịch sử; time, năng lực team, infrastructure và quy mô business khi đó đều có thể ảnh hưởng đến lựa chọn.

## Tránh overstate project thế nào?

Những mô tả sau rất dễ kéo theo các câu hỏi vượt quá phạm vi chuẩn bị:

- Viết project của team thành project do mình phụ trách độc lập.
- Viết một solution chỉ từng đọc thành solution đã được triển khai production.
- Viết kết quả test trên single machine thành production QPS.
- Thêm vào project những middleware thực tế chưa từng sử dụng chỉ để làm project có vẻ phức tạp.
- Chỉ nhớ conclusion do tutorial đưa ra mà chưa xem code và config của project.

Có thể tối ưu cách diễn đạt project, nhưng không được bịa trách nhiệm và kết quả. Nếu một solution chỉ từng được research hoặc làm Demo, hãy nói thẳng: “Cuối cùng production không áp dụng; tôi từng phụ trách verification của solution và conclusion thu được là……”. Câu trả lời như vậy cũng chịu được các câu hỏi đào sâu tiếp theo.

## Tự kiểm tra trước buổi phỏng vấn

Trước buổi phỏng vấn, hãy tìm một tờ giấy trắng và hoàn thành những việc sau mà không xem tài liệu:

- Giới thiệu project lần lượt trong 30 giây và 3 phút.
- Vẽ một core request flow và một failure flow.
- Nói rõ ba feature do bản thân phụ trách và vị trí code tương ứng.
- Chuẩn bị một lựa chọn công nghệ, một vấn đề performance và một incident hoặc defect fix.
- Bổ sung cách thống kê và tài liệu verification cho từng con số trong CV.
- Trả lời “Tại sao sử dụng, thất bại thế nào, quan sát ra sao” với từng middleware.

Nếu một câu hỏi chỉ có thể trả lời bằng định nghĩa component, hãy quay lại code hoặc document của project và kiểm tra thêm một lần. Phần giới thiệu project không cần quá hoa mỹ; chỉ cần interviewer có thể tiếp tục hỏi theo câu trả lời của bạn, còn bạn có detail thực tế để trả lời tiếp, về cơ bản là đủ.

## Đọc thêm

- [Hướng dẫn kinh nghiệm project](./project-experience-guide.md)
- [Hướng dẫn viết CV cho programmer](./resume-guide.md)
- [Troubleshooting vấn đề production của Java backend](../java/jvm/jvm-in-action.md)
- [Câu hỏi phỏng vấn system design hiệu năng cao](../high-performance/high-performance-system-interview-questions.md)
- [Câu hỏi phỏng vấn system design high availability](../high-availability/high-availability-system-interview-questions.md)
- [Nhập môn performance test và load test](../high-availability/performance-test.md)

<!-- @include: @article-footer.snippet.md -->
