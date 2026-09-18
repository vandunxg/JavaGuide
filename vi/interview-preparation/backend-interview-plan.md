---
title: "Kế hoạch chinh phục phỏng vấn Java backend bản mới nhất 2026 (bao quát hệ thống backend tổng quát)"
description: "Kế hoạch ôn tập phỏng vấn Java backend, cung cấp bản rút gọn 4 tuần và bản tiêu chuẩn 8 tuần, bao quát project và CV, Java core, MySQL, Redis, Spring, kiến thức máy tính cơ bản, hệ thống phân tán, high availability và JVM, đồng thời đưa ra sản phẩm đầu ra và phương pháp tự kiểm tra cho từng giai đoạn."
category: Interview Preparation
icon: mdi:star-outline
head:
  - - meta
    - name: keywords
      content: Java backend interview, interview preparation plan, interview guide, câu hỏi lý thuyết, tuyển dụng sinh viên mới tốt nghiệp, tuyển dụng có kinh nghiệm, project experience, Java interview
---

Đọc hết các câu hỏi phỏng vấn trong JavaGuide chỉ mới hoàn thành việc đọc tài liệu. Interviewer thường bắt đầu hỏi từ CV và project, rồi tiếp tục hỏi sâu về Java, MySQL, Redis, Spring hoặc message queue được đề cập trong đó. Chỉ ôn tập theo mục lục knowledge base rất dễ dẫn đến việc đọc rất nhiều bài nhưng đến lúc tự trả lời lại vẫn không biết bắt đầu từ đâu.

Kế hoạch này đặt project và CV lên trước, triển khai kiến thức kỹ thuật theo CV và vị trí mục tiêu. Kế hoạch được chia thành bản rút gọn 4 tuần và bản tiêu chuẩn 8 tuần. Khi không đủ thời gian, hãy bỏ phần mở rộng không liên quan đến vị trí, đừng biến mọi chuyên đề thành việc đọc lướt.

## Chọn 4 tuần hay 8 tuần trước

| Giai đoạn                                  | Bản rút gọn 4 tuần                      | Bản tiêu chuẩn 8 tuần                            |
| ------------------------------------------ | --------------------------------------- | ------------------------------------------------ |
| Kiểm tra ban đầu                           | Ngày 1～2                               | Ngày 1～2                                        |
| Project và CV                              | Thời gian còn lại của tuần 1            | Tuần 1                                           |
| Java, MySQL, Redis                         | Tuần 2                                  | Tuần 2～4                                        |
| Spring và system design                    | Nửa đầu tuần 3                          | Tuần 5                                           |
| Kiến thức máy tính và thuật toán           | Lồng ghép mỗi ngày, chọn theo vị trí    | Tập trung ôn ở tuần 6, luyện thuật toán mỗi ngày |
| Distributed, JVM và xử lý sự cố production | Nửa sau tuần 3 đến tuần 4, chọn theo CV | Tuần 7                                           |
| Mock interview và bổ sung lỗ hổng          | 2 ngày cuối                             | Tuần 8                                           |

Bản 4 tuần phù hợp với người đã học các kiến thức chính và hiện cần ôn tập tập trung; nếu lần đầu học có hệ thống kiến thức Java backend, 8 tuần cũng chỉ là điểm bắt đầu. Nếu thời gian có thể duy trì mỗi ngày chưa đến 2 giờ, hãy ưu tiên bản 8 tuần. Nếu đã bắt đầu ứng tuyển hoặc sắp interview, có thể dùng bản 4 tuần, nhưng phạm vi ôn tập phải thu hẹp theo CV: nếu CV không viết Kafka và mô tả vị trí cũng không yêu cầu, không cần dành hai ba ngày cho chi tiết triển khai message queue.

Đừng để thuật toán đến cuối mới học cấp tốc. Với vị trí có yêu cầu bài thi viết hoặc coding question, hãy duy trì luyện tập từ tuần đầu; với vị trí không kiểm tra thuật toán, hãy dành thời gian này cho project, database và scenario question.

## Mức độ nào mới được xem là biết

“Đã xem qua” và “có thể trả lời khi interview” cách nhau rất xa. Mỗi câu hỏi ít nhất phải trải qua ba tầng sau:

| Tầng                           | Cách tự kiểm tra                                                                                            |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------- |
| Có thể trả lời                 | Không xem tài liệu, nói ra kết luận và keyword trong 30～60 giây                                            |
| Có thể trả lời câu hỏi đào sâu | Tiếp tục giải thích nguyên lý triển khai, điều kiện áp dụng, cách thất bại thường gặp và phương án thay thế |
| Có thể gắn với project         | Nêu project có sử dụng hay không, vì sao chọn như vậy, đã gặp hạn chế gì, xác minh kết quả thế nào          |

Không cần làm ghi chú ôn tập quá phức tạp, chỉ cần giữ bốn cột “câu hỏi, link tài liệu, tầng hiện tại, điểm chưa trả lời tốt”. Nội dung đọc xong trong ngày ít nhất phải được trả lời một lần mà không nhìn tài liệu; nếu không trả lời được thì quay lại tra bài gốc, đừng dùng việc đọc đi đọc lại để thay cho hồi tưởng.

## Giai đoạn 0: Xác định phạm vi trước

Dùng 1～2 ngày để hoàn thành ba việc: xác định vị trí mục tiêu, kiểm tra CV và thực hiện một lần tự kiểm tra tổng quan.

Trước tiên, tìm một vài mô tả vị trí dự định ứng tuyển, ghi lại các kỹ năng lặp lại nhiều lần rồi đối chiếu từng mục với CV. Phạm vi ôn tập chủ yếu đến từ hai nơi: vị trí yêu cầu rõ điều gì và CV chủ động viết gì. Nếu CV xuất hiện “quen thuộc với Redis”, “phụ trách module order”, “sử dụng Kafka xử lý asynchronous task”, thì phía sau phải có câu hỏi tương ứng và chi tiết project để trả lời tiếp.

Ít nhất cần để lại bốn tài liệu trong giai đoạn này:

- Một CV PDF có thể dùng để ứng tuyển.
- Dàn ý tự giới thiệu trong 30～60 giây.
- Một bản tóm tắt project cho mỗi project.
- Một danh sách câu hỏi cần ôn được sắp xếp theo mức độ ưu tiên.

Phương pháp chuẩn bị có thể tham khảo [Làm thế nào để chuẩn bị hiệu quả cho Java interview?](./teach-you-how-to-prepare-for-the-interview-hand-in-hand.md) và [Tổng hợp trọng tâm Java backend interview](./key-points-of-interview.md).

Khi CV chưa hoàn thiện, hãy xem [Hướng dẫn viết CV cho programmer](./resume-guide.md) trước; đừng vừa ôn tập vừa liên tục thêm công nghệ mới vào CV, nếu không phạm vi ôn tập sẽ không ngừng mở rộng.

## Giai đoạn 1: Đào sâu project và CV

Project thường là điểm bắt đầu của các câu hỏi kỹ thuật. Nếu không trình bày rõ project, dù học thuộc nguyên lý của nhiều component cũng rất khó đưa câu trả lời trở lại trải nghiệm của bản thân.

Hãy chuẩn bị các nội dung sau cho mỗi project trọng điểm:

| Nội dung            | Câu hỏi cần trả lời                                                            |
| ------------------- | ------------------------------------------------------------------------------ |
| Bối cảnh nghiệp vụ  | Project phục vụ ai, giải quyết vấn đề gì, core flow là gì                      |
| Trách nhiệm cá nhân | Những API, table, task hoặc module nào do mình phụ trách, tham gia ở mức nào   |
| Request flow        | Một request đi qua những service, cache, database và message queue nào         |
| Lựa chọn công nghệ  | Vì sao dùng giải pháp hiện tại, đã so sánh gì, phải trả giá nào                |
| Điểm khó hoặc sự cố | Hiện tượng là gì, định vị, sửa chữa và xác minh thế nào                        |
| Chỉ số project      | Dữ liệu đến từ production hay test, cách thống kê và điều kiện đối chiếu là gì |
| Phạm vi trách nhiệm | Phần nào do đồng nghiệp hoặc team khác phụ trách                               |

Chuẩn bị hai phiên bản 30 giây và 3 phút cho mỗi project. Bản 30 giây trình bày nghiệp vụ, trách nhiệm và một trọng điểm; bản 3 phút bổ sung core flow, lựa chọn công nghệ và một vấn đề có thể tiếp tục bị hỏi sâu. Đừng học thuộc nguyên văn, chỉ cần nhớ trình tự và keyword. Cách viết cụ thể xem tại [Trình bày project backend khi interview thế nào?](./backend-project-interview-guide.md).

Hãy tiếp tục liệt kê câu hỏi theo từng công nghệ trong project. Ví dụ, nếu sử dụng Redis để cache thông tin sản phẩm, ít nhất phải chuẩn bị về thiết kế Key, chiến lược hết hạn, cache miss, data consistency và cách xử lý khi Redis không khả dụng; nếu viết thread pool, phải giải thích được loại task, tham số core, queue, rejection policy và năng lực chịu tải của downstream.

Kết quả project có thể định lượng, nhưng con số phải có nguồn. Khi không có chỉ số production, có thể đo bổ sung trong môi trường test và ghi rõ cấu hình máy, lượng dữ liệu, mô hình concurrency và thời lượng test. Đừng bịa QPS production cho project luyện tập, cũng đừng viết module chỉ mới xem qua thành phần mình phụ trách.

Không có internship hoặc project chính thức vẫn có thể chuẩn bị. Project hoàn thành theo khóa học, project open source được phát triển tiếp, project môn học và project cuộc thi đều có thể viết; trọng điểm là những thay đổi bản thân đã thực hiện: thêm tính năng, điều chỉnh schema, bổ sung test, sửa defect hoặc so sánh các phương án khác nhau. Có thể tiếp tục đọc [Hướng dẫn về project experience](./project-experience-guide.md), [Không có kinh nghiệm internship khi tuyển dụng sinh viên mới tốt nghiệp thì làm gì?](./internship-experience.md) và [Các project open source Java chất lượng để thực hành](../open-source-project/practical-project.md).

Sau khi hoàn thành giai đoạn này, chọn ngẫu nhiên một project và trả lời bốn câu hỏi sau mà không xem tài liệu:

1. Project này giải quyết vấn đề gì, bạn phụ trách việc gì?
2. Một core request đi qua các bước thế nào?
3. Lựa chọn công nghệ nào đáng giải thích nhất, vì sao?
4. Đã gặp vấn đề gì, kết luận được hỗ trợ bởi bằng chứng nào?

## Giai đoạn 2: Java, MySQL và Redis

Ba phần này có phạm vi rất rộng, không phù hợp để chia thời gian đều nhau. Trước tiên hãy làm một lượt rút câu hỏi ngẫu nhiên; chuyên đề nào chỉ nói được định nghĩa thì bổ sung thời gian cho chuyên đề đó, còn nội dung đã có thể trả lời gắn với project chỉ cần ôn lại.

### Java Basics, collection và concurrency

Trước tiên đọc ba nhóm bài về Java Basics, collection và concurrency:

- [Các câu hỏi phỏng vấn Java Basics thường gặp (phần trên)](../java/basis/java-basic-questions-01.md), [（phần giữa）](../java/basis/java-basic-questions-02.md), [（phần dưới）](../java/basis/java-basic-questions-03.md)
- [Các câu hỏi phỏng vấn Java collection thường gặp (phần trên)](../java/collection/java-collection-questions-01.md), [（phần dưới）](../java/collection/java-collection-questions-02.md)
- [Các câu hỏi phỏng vấn Java concurrency thường gặp (phần trên)](../java/concurrent/java-concurrent-questions-01.md), [（phần giữa）](../java/concurrent/java-concurrent-questions-02.md), [（phần dưới）](../java/concurrent/java-concurrent-questions-03.md)

Phần Basics cần giải thích được các khái niệm thường gặp và hành vi của code; collection tập trung vào lựa chọn, mở rộng capacity, thread safety và cách dùng sai thường gặp; concurrency cần kết nối được thread state, lock, JMM, ThreadLocal, thread pool và asynchronous task. Khi CV liên quan đến concurrent programming, hãy đọc sâu hơn [JMM](../java/concurrent/jmm.md), [Giải thích chi tiết thread pool](../java/concurrent/java-thread-pool-summary.md), [ThreadLocal](../java/concurrent/threadlocal.md), [AQS](../java/concurrent/aqs.md) và [CompletableFuture](../java/concurrent/completablefuture-intro.md).

### MySQL

[Tổng hợp các câu hỏi phỏng vấn MySQL thường gặp](../database/mysql/mysql-questions-01.md) phù hợp làm tuyến chính. Khi đọc đến index, transaction và lock, hãy chuyển sang các bài chuyên đề:

- [Giải thích chi tiết MySQL index](../database/mysql/mysql-index.md)
- [Ba log chính của MySQL](../database/mysql/mysql-logs.md)
- [Transaction isolation level](../database/mysql/transaction-isolation-level.md)
- [Cách InnoDB triển khai MVCC](../database/mysql/innodb-implementation-of-mvcc.md)
- [SQL được thực thi trong MySQL thế nào](../database/mysql/how-sql-executed-in-mysql.md)
- [Phân tích MySQL execution plan](../database/mysql/mysql-query-execution-plan.md)

Đừng dừng ở leftmost matching và index không được sử dụng khi gặp câu hỏi về index. Hãy lấy một SQL trong project và giải thích điều kiện query, phân bố dữ liệu, execution plan, số row được scan cũng như cách sửa cuối cùng. Câu hỏi về transaction cũng phải gắn với code: Vì sao transaction scope quá lớn, những call nào không nên đặt trong transaction, transaction của Spring sẽ thất bại trong những trường hợp nào.

### Redis

Trước tiên đọc [Các câu hỏi phỏng vấn Redis thường gặp (phần trên)](../database/redis/redis-questions-01.md) và [（phần dưới）](../database/redis/redis-questions-02.md), sau đó chọn chuyên đề theo project:

- Data structure: [5 basic data types](../database/redis/redis-data-structures-01.md), [3 special data types](../database/redis/redis-data-structures-02.md), [Skip list](../database/redis/redis-skiplist.md)
- Vấn đề cache: [Cache Basics](../database/redis/cache-basics.md), [Các chiến lược đọc và ghi cache thường dùng](../database/redis/3-commonly-used-cache-read-and-write-strategies.md)
- Vận hành và lưu trữ: [Persistence](../database/redis/redis-persistence.md), [Memory fragmentation](../database/redis/redis-memory-fragmentation.md), [Tổng hợp các nguyên nhân blocking thường gặp](../database/redis/redis-common-blocking-problems-summary.md)
- Cách dùng trong nghiệp vụ: [Redis delayed task](../database/redis/redis-delayed-task.md), [Dùng Redis Stream làm message queue](../database/redis/redis-stream-mq.md)

Khi chuẩn bị Redis, đừng chỉ học thuộc data type. Hãy chọn một cache flow thực tế trong project và thử trình bày đầy đủ: request đọc cache thế nào, sau khi cache miss thì query dữ liệu từ đâu, query được rồi thì ghi ngược lại thế nào, cache hết hạn sau bao lâu, khi Redis gặp sự cố thì nghiệp vụ fallback ra sao.

Sau khi ôn xong, hãy trộn Java, MySQL và Redis để rút câu hỏi, mỗi loại 5 câu. Mỗi câu trước tiên dùng một hai câu để đưa ra kết luận, sau đó tiếp tục tự hỏi hai vòng: “Vì sao?” và “Dùng trong project thế nào?”. Câu nào bị vướng thì quay lại bài tương ứng để bổ sung đúng knowledge point đó, không cần đọc lại cả chương.

## Giai đoạn 3: Spring và system design

### Spring, Spring Boot và MyBatis

Trọng tâm chuẩn bị Spring là các tính năng thực sự sử dụng trong project. Trước tiên đọc [Các câu hỏi phỏng vấn Spring thường gặp](../system-design/framework/spring/spring-knowledge-and-questions-summary.md) và [Các câu hỏi phỏng vấn Spring Boot thường gặp](../system-design/framework/spring/springboot-knowledge-and-questions-summary.md), sau đó bổ sung các chuyên đề sau:

- [IoC và AOP](../system-design/framework/spring/ioc-and-aop.md)
- [Spring transaction](../system-design/framework/spring/spring-transaction.md)
- [Nguyên lý auto-assembly của Spring Boot](../system-design/framework/spring/spring-boot-auto-assembly-principles.md)
- [Design pattern được sử dụng trong Spring](../system-design/framework/spring/spring-design-patterns-summary.md)
- [Các câu hỏi phỏng vấn MyBatis thường gặp](../system-design/framework/mybatis/mybatis-interview.md)

Khi tự kiểm tra, đừng chỉ giải thích ý nghĩa annotation. Hãy dùng project để trình bày Bean được tạo thế nào, AOP được dùng ở đâu, transaction boundary được phân chia thế nào, vì sao một transaction nào đó thất bại, và cuối cùng MyBatis thực thi SQL nào. Nếu project không sử dụng Netty, reactive programming hoặc extension point phức tạp, không cần tạm thời thêm chúng vào CV chỉ để mở rộng phạm vi.

### Authentication, authorization và các vấn đề security thường gặp

Khi CV liên quan đến login, permission hoặc open API, hãy chuẩn bị [Authentication và authorization Basics](../system-design/security/basis-of-authority-certification.md), [JWT](../system-design/security/jwt-intro.md), [SSO](../system-design/security/sso-intro.md) và [Thiết kế permission system](../system-design/security/design-of-authority-system.md). Khi trả lời, hãy nói rõ thông tin authentication được đặt ở đâu, permission được kiểm tra tại vị trí nào, Token hết hiệu lực thế nào, cũng như API ngăn chặn privilege escalation và duplicate submission ra sao.

### System design và scenario question

Với system design question, trước tiên xác nhận requirement và constraint rồi mới bắt đầu vẽ component. Hãy triển khai câu trả lời theo trình tự sau:

1. Làm rõ quy mô user, request volume, latency, availability và consistency requirements.
2. Xác định core business flow, data model và API.
3. Đưa ra phương án nền tảng có thể hoạt động.
4. Bổ sung cache, asynchronous processing, sharding, rate limiting hoặc degradation theo bottleneck.
5. Nêu failure scenario, data consistency, monitoring và capacity verification.

Để nhập môn, hãy xem [Tổng hợp các câu hỏi phỏng vấn system design thường gặp](../system-design/system-design-questions.md), [System design interview question về high performance](../high-performance/high-performance-system-interview-questions.md) và [System design interview question về high availability](../high-availability/high-availability-system-interview-questions.md). Các scenario hoàn chỉnh như short URL, flash sale và xử lý dữ liệu khổng lồ có thể tham khảo [Các system design và scenario question backend interview thường gặp](../zhuanlan/back-end-interview-high-frequency-system-design-and-scenario-questions.md).

Sau khi hoàn thành, chọn hai câu hỏi để trình bày bằng lời mà không xem architecture diagram có sẵn. Lần đầu đưa ra phương án nền tảng; sau khi interviewer tăng traffic, thêm failure hoặc consistency requirement thì điều chỉnh lại, trọng điểm là giải thích rõ vì sao phương án thay đổi.

## Giai đoạn 4: Kiến thức máy tính cơ bản và thuật toán

Độ sâu ôn tập kiến thức máy tính cơ bản do vị trí và quy trình interview quyết định. Nếu có vòng viết, thuật toán hoặc coding bằng tay, cần luyện thuật toán liên tục từ tuần đầu; nếu vị trí chú trọng business development hơn, vẫn phải đảm bảo có thể trả lời về network, operating system và các data structure thường gặp.

### Thuật toán và data structure

Trước tiên dùng [Chuyên đề thuật toán](../cs-basics/algorithms/) để xác định phạm vi, sau đó luyện [Binary search](../cs-basics/algorithms/binary-search.md), [Two pointers và sliding window](../cs-basics/algorithms/two-pointers-and-sliding-window.md), [DFS/BFS](../cs-basics/algorithms/dfs-bfs.md), [Backtracking](../cs-basics/algorithms/backtracking.md), [Dynamic programming](../cs-basics/algorithms/dynamic-programming.md) và [Top K](../cs-basics/algorithms/top-k.md).

Khi luyện bài, hãy giữ lại các bài làm sai và boundary condition, đừng chỉ cố học thuộc template. Ít nhất phải giải thích được time complexity, tự viết các bài thường gặp về linked list, tree traversal, binary search, hash và heap; nếu CV viết một data structure nào đó, còn phải giải thích được vì sao nó phù hợp với scenario hiện tại.

### Computer network và operating system

Về network, trước tiên đọc [Các câu hỏi phỏng vấn computer network thường gặp (phần trên)](../cs-basics/network/other-network-questions.md) và [（phần dưới）](../cs-basics/network/other-network-questions2.md), sau đó tập trung vào [Quy trình từ khi nhập URL đến khi hiển thị trang](../cs-basics/network/the-whole-process-of-accessing-web-pages.md), [HTTP và HTTPS](../cs-basics/network/http-vs-https.md), [TCP three-way handshake và four-way wave](../cs-basics/network/tcp-connection-and-disconnection.md) và [TCP đảm bảo reliable transmission thế nào](../cs-basics/network/tcp-reliability-guarantee.md).

Về operating system, lấy [Các câu hỏi phỏng vấn operating system thường gặp (phần trên)](../cs-basics/operating-system/operating-system-basic-questions-01.md) và [（phần dưới）](../cs-basics/operating-system/operating-system-basic-questions-02.md) làm chính, tập trung kiểm tra process và thread, virtual memory, I/O, deadlock và system call. Đừng chỉ học thuộc định nghĩa, hãy thử liên hệ chúng với Java thread, file I/O, network request, OOM và context switch.

## Giai đoạn 5: Distributed, high performance và high availability

Giai đoạn này đi theo CV và vị trí. Nếu project là monolith và vị trí cũng không yêu cầu distributed, chỉ cần nắm các vấn đề thường gặp; nếu CV viết microservice, message queue, distributed lock hoặc database sharding, phải có khả năng chịu được câu hỏi đào sâu ở các chuyên đề tương ứng.

| Nội dung xuất hiện trong CV hoặc vị trí   | Tài liệu bắt đầu                                                                                                                                                                                                                                                      | Ít nhất cần chuẩn bị đến mức nào                                                         |
| ----------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Microservice, RPC                         | [Câu hỏi phỏng vấn microservice](../distributed-system/microservices-interview-questions.md), [RPC Basics](../distributed-system/rpc/rpc-intro.md)                                                                                                                    | Service được tách thế nào, call timeout và retry ra sao, failure được cô lập thế nào     |
| Gateway, configuration center             | [API Gateway](../distributed-system/api-gateway.md), [Distributed configuration center](../distributed-system/distributed-configuration-center.md)                                                                                                                    | Request routing, authentication, rate limiting, config push và xử lý failure             |
| Distributed ID, lock, transaction         | [Distributed ID](../distributed-system/distributed-id.md), [Distributed lock](../distributed-system/distributed-lock-implementations.md), [Distributed transaction](../distributed-system/distributed-transaction.md)                                                 | Điều kiện lựa chọn, correctness risk, timeout và failure recovery                        |
| Message queue                             | [Câu hỏi phỏng vấn message queue](../high-performance/message-queue/message-queue-interview-questions.md)                                                                                                                                                             | Send failure, duplicate consumption, thứ tự, backlog và capacity của downstream          |
| High concurrency và database optimization | [High-performance system design](../high-performance/high-performance-system-interview-questions.md), [SQL optimization](../high-performance/sql-optimization.md)                                                                                                     | Vị trí bottleneck, capacity limit, cái giá của cache và asynchronous processing          |
| Stability engineering                     | [High-availability system design](../high-availability/high-availability-system-design.md), [Timeout và retry](../high-availability/timeout-and-retry.md), [Rate limiting](../high-availability/limit-request.md), [Idempotency](../high-availability/idempotency.md) | Failure lan truyền thế nào, giảm thiệt hại ra sao, biện pháp tạm thời có tác dụng phụ gì |

CAP, BASE, consistent hashing, Raft và các theory khác dùng để giải thích design cụ thể, không cần tách khỏi project để học thuộc định nghĩa dài. Hãy chọn một distributed solution trong project và trả lời vì sao cần nó, vì sao chọn như vậy, khi failure sẽ thế nào, cũng như chứng minh nó thực sự có hiệu quả ra sao.

## Giai đoạn 6: JVM và xử lý vấn đề production

Nếu CV viết JVM tuning, GC optimization, OOM troubleshooting hoặc vị trí nhấn mạnh việc xử lý vấn đề production, nên đưa giai đoạn này lên trước ngay sau Java concurrency. Với sinh viên mới tốt nghiệp thiếu kinh nghiệm production, ít nhất phải nắm memory area, object reclamation, class loading và tư duy chẩn đoán thường gặp.

Trước tiên dùng [Tổng hợp các câu hỏi phỏng vấn JVM thường gặp](../java/jvm/jvm-interview-questions.md) để liệt kê các câu hỏi cần trả lời, sau đó bổ sung các chuyên đề sau:

- [Java memory area](../java/jvm/memory-area.md)
- [JVM garbage collection](../java/jvm/jvm-garbage-collection.md)
- [Quá trình class loading](../java/jvm/class-loading-process.md) và [class loader](../java/jvm/classloader.md)
- [Công cụ monitoring và troubleshooting của JDK](../java/jvm/jdk-monitoring-and-troubleshooting-tools.md)
- [Xử lý vấn đề production của Java backend](../java/jvm/jvm-in-action.md)

Khi tự kiểm tra, đừng dừng ở “heap chứa object, stack chứa local variable”. Hãy tự đặt một cảnh báo cụ thể, chẳng hạn CPU tăng vọt, Full GC thường xuyên hoặc OOM, rồi giải thích trước tiên cần xác nhận chỉ số nào, bảo toàn hiện trường thế nào, dùng công cụ gì để thu hẹp phạm vi, thao tác nào có thể làm sự cố nghiêm trọng hơn và xác minh thế nào sau khi sửa.

## Sắp xếp việc ôn tập trong một tuần thế nào

Thời gian mỗi ngày có thể chia đại khái thành ba phần: một nửa dùng để đọc và hiểu, một phần tư để trả lời không nhìn tài liệu, thời gian còn lại để luyện cách trình bày project hoặc thuật toán. Đọc bao nhiêu trang trong ngày không quan trọng, ít nhất phải để lại một câu hỏi có thể thuật lại và một điểm vẫn chưa trả lời tốt.

Mỗi tuần sắp xếp một mock interview 30～60 phút. Nhờ đối phương bắt đầu hỏi từ CV, sau khi hỏi sâu về project mới chuyển sang Java, database và scenario question. Khi không có bạn đồng hành, có thể ghi âm hoặc dùng AI để mô phỏng câu hỏi đào sâu, nhưng sau khi trả lời xong vẫn phải quay lại bài viết, code hoặc tài liệu chính thức để đối chiếu fact.

Liên tục thêm tài liệu mới trong quá trình ôn tập rất dễ khiến kế hoạch mất kiểm soát. Mỗi chuyên đề chỉ cần giữ một tài liệu chính và một ít bài chuyên đề; nếu đã xem ba câu trả lời cho cùng một câu hỏi mà vẫn không nói được, nên bắt đầu trả lời không nhìn tài liệu thay vì tiếp tục lưu thêm câu trả lời thứ tư.

## Làm gì trong 1～2 ngày trước interview

Khi sắp interview, đừng mở chuyên đề mới nữa, hãy chốt theo CV và các bài làm sai:

| Việc                   | Cách làm                                                                                                                                     |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| Tự giới thiệu          | Nói một lần bản 30～60 giây, xác nhận trải nghiệm, tech stack và hướng tìm việc thống nhất                                                   |
| Project                | Trình bày một lần bản 30 giây và 3 phút cho mỗi project trọng điểm, lập tức bổ sung tài liệu ở chỗ bị vướng                                  |
| Tech stack trong CV    | Kiểm tra ngẫu nhiên các công nghệ ghi “quen thuộc” hoặc “thành thạo”, xác nhận có thể trả lời nguyên lý, giới hạn và cách dùng trong project |
| Câu hỏi sai thường gặp | Chỉ ôn lại các câu mình làm sai và điểm yếu, không làm lại toàn bộ question bank                                                             |
| Code và thiết bị       | Với online interview, kiểm tra trước network, camera, microphone, chia sẻ màn hình và môi trường lập trình                                   |
| Thông tin vị trí       | Xem lại mô tả vị trí một lần, chuẩn bị project và câu hỏi liên quan nhất đến vị trí                                                          |

Khi hồi hộp ảnh hưởng đến khả năng thể hiện, có thể tham khảo [Quá căng thẳng khi interview thì phải làm gì?](./how-to-handle-interview-nerves.md).

## Sau interview ôn lại thế nào

Sau interview hãy nhanh chóng ghi lại các câu hỏi, không cần cố khôi phục đầy đủ mọi chi tiết. Với mỗi câu chưa trả lời tốt, ghi lại năm mục: câu hỏi, lúc đó đã trả lời thế nào, còn thiếu gì, căn cứ đúng nằm ở đâu, lần sau sẽ trả lời thế nào. Khi bị vướng ở câu hỏi đào sâu về project, còn phải quay lại bản tóm tắt project để bổ sung trách nhiệm, vị trí code, cách thống kê chỉ số hoặc giới hạn của phương án.

Trước interview tiếp theo chỉ xem bản ghi chú này và danh sách câu hỏi ưu tiên cao trước đó. Nội dung mở rộng liên tục vài lần không bị hỏi đến, đồng thời không xuất hiện trong CV và yêu cầu vị trí, có thể hạ mức ưu tiên; câu hỏi lặp lại nhiều lần thì đưa vào danh sách chính. Phạm vi ôn tập sẽ dần thu hẹp theo các interview thực tế.
