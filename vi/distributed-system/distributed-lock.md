---
title: "Nhập môn distributed lock: Vì sao cần distributed lock, lock granularity, gia hạn timeout và các trường hợp sử dụng"
category: Distributed
description: "Nhập môn distributed lock, giải thích vì sao cần distributed lock, ngữ nghĩa mutual exclusion, lock granularity, owner token, giải phóng an toàn, gia hạn timeout, Fencing Token và các trường hợp sử dụng điển hình như flash sale, trừ tồn kho."
tag:
  - Distributed lock
head:
  - - meta
    - name: keywords
      content: distributed lock,nhập môn distributed lock,vì sao cần distributed lock,lock granularity,giải phóng an toàn,Fencing Token,flash sale bán vượt tồn kho,trừ tồn kho,câu hỏi phỏng vấn distributed lock
---

Có rất nhiều bài viết về distributed lock trên mạng. Bài này là một phiên bản tương đối ngắn gọn, dễ hiểu; hướng đến phỏng vấn và công việc hằng ngày, trước hết làm rõ các khái niệm và ranh giới thường gặp nhất.

Bài viết trước tiên giới thiệu các khái niệm cơ bản về distributed lock.

## Vì sao cần distributed lock?

Trong môi trường nhiều thread, nếu nhiều thread đồng thời truy cập và sửa cùng một shared resource (ví dụ tồn kho sản phẩm, đơn giao đồ ăn), nhưng không có mutual exclusion, atomic update, optimistic lock hoặc unique constraint để bảo vệ, có thể xảy ra dữ liệu không nhất quán, xử lý trùng lặp, bán vượt tồn kho và các vấn đề khác, ảnh hưởng đến tính đúng đắn và tính ổn định của chương trình.

Ví dụ, giả sử hiện có 100 người dùng tham gia một hoạt động flash sale có thời hạn; mỗi người dùng chỉ được mua 1 sản phẩm, trong khi số lượng sản phẩm chỉ có 3. Nếu không truy cập shared resource theo cơ chế mutual exclusion, có thể xảy ra tình huống sau:

- Nhiều thread như thread 1, 2, 3 cùng đi vào phương thức mua hàng; mỗi thread tương ứng với một người dùng.
- Thread 1 và thread 2 đại diện cho hai người dùng khác nhau. Chúng gần như đồng thời đọc được rằng tồn kho còn 1 sản phẩm, nên đều vượt qua kiểm tra tồn kho, tiếp tục tạo order và trừ tồn kho.
- Thread 1 tiếp tục thực thi, giảm số lượng tồn kho đi 1 rồi trả về thành công.
- Thread 2 cũng tiếp tục thực thi, giảm số lượng tồn kho đi 1 rồi trả về thành công.
- Cuối cùng cả hai request đều thành công, nhưng tồn kho chỉ đủ bán 1 sản phẩm, nên xảy ra bán vượt tồn kho.
- Kiểm tra giới hạn mua và trừ tồn kho là hai ràng buộc khác nhau: giới hạn mua xử lý việc một người dùng mua lặp, còn trừ tồn kho xử lý nhiều người dùng cùng cạnh tranh một lượng tồn kho.

![Shared resource không được truy cập theo mutual exclusion dẫn đến sự cố](https://oss.javaguide.cn/github/javaguide/distributed-system/distributed-lock/oversold-without-locking.png)

Ý tưởng của lock là tuần tự hóa một critical section: tại cùng một thời điểm chỉ cho phép một execution unit đi vào logic này. Cách này có thể giảm xung đột đồng thời, nhưng cũng làm giảm throughput; nếu có thể giải quyết bằng database atomic update, unique constraint, optimistic lock, CAS hoặc tuần tự hóa message thì không nhất thiết phải dùng distributed lock.

Ví dụ, chống bán vượt tồn kho không nhất thiết phải dùng distributed lock: database conditional update `UPDATE stock SET count = count - 1 WHERE sku_id = ? AND count > 0` có thể bảo đảm tồn kho không bị trừ thành số âm; giới hạn mua của người dùng có thể tạo unique index trên `user_id + activity_id`; tạo order có thể dùng idempotency key để chống submit trùng. Trong tình huống high concurrency còn có thể kết hợp Redis pre-deduct inventory, MQ ghi database bất đồng bộ và đối soát bù trừ.

Distributed lock được thảo luận ở đây về bản chất là một giải pháp pessimistic mutual exclusion: lấy lock trước rồi mới vào critical section; nếu không lấy được lock thì chờ, thất bại hoặc retry.

Pessimistic lock luôn giả định tình huống xấu nhất, cho rằng mỗi lần shared resource được truy cập đều có thể phát sinh vấn đề (chẳng hạn shared data bị sửa), nên mỗi lần lấy resource đều lock. Nhờ vậy, thread khác muốn lấy resource sẽ bị block cho đến khi lock được holder trước đó giải phóng. Nói cách khác, **mỗi lần shared resource chỉ được một thread sử dụng, các thread khác bị block; sau khi dùng xong, resource mới được chuyển cho thread khác**.

Với nhiều thread trên một máy, trong Java, chúng ta thường dùng class `ReentrantLock` và keyword `synchronized` có sẵn trong JDK, tức **local lock**, để kiểm soát nhiều thread trong cùng một JVM process truy cập local shared resource.

Dưới đây là sơ đồ minh họa local lock.

![Local lock](https://oss.javaguide.cn/github/javaguide/distributed-system/distributed-lock/jvm-local-lock.png)

Có thể thấy các thread truy cập shared resource theo mutual exclusion; tại cùng một thời điểm chỉ một thread lấy được local lock để truy cập shared resource.

Trong distributed system, các service/client khác nhau thường chạy trên những JVM process độc lập. Nếu nhiều JVM process dùng chung một resource, local lock không thể thực hiện mutual exclusion khi truy cập resource. Khi đó cần đặt trạng thái lock trong một external system mà mọi process đều có thể truy cập, tức **distributed lock**.

Nhìn từ góc độ distributed coordination, distributed lock thực ra trả lời một câu hỏi: tại cùng một thời điểm, ai là owner duy nhất của một resource? Nếu bạn chưa hiểu rõ mối quan hệ giữa Leader/Quorum, Lease và Fencing Token, nên đọc [Giải thích chi tiết về distributed coordination](./protocol/centralized-and-decentralized.md) trước, sau đó quay lại tìm hiểu các implementation cụ thể như Redis, ZooKeeper và etcd sẽ thuận lợi hơn.

Ví dụ: order service của hệ thống được deploy thành 3 instance, cùng cung cấp service ra bên ngoài. Để chống bán vượt tồn kho, thứ cần bảo vệ không chỉ là “kiểm tra tồn kho”, mà là critical section “kiểm tra tồn kho → trừ tồn kho → ghi nhận giao dịch mua/tạo order”; nếu chỉ lock việc query mà không lock việc trừ tồn kho thì vẫn có thể ghi sai do concurrent access. Vì order service nằm trên các JVM process khác nhau, local lock không thể hoạt động đúng trong tình huống này. Cần dùng distributed lock để dù các thread không nằm trong cùng một JVM process, chúng vẫn có thể lấy cùng một lock, từ đó thực hiện mutual exclusion khi truy cập shared resource.

Dưới đây là sơ đồ minh họa distributed lock.

![Distributed lock](https://oss.javaguide.cn/github/javaguide/distributed-system/distributed-lock/distributed-lock.png)

Có thể thấy các thread trong những process độc lập này truy cập shared resource theo mutual exclusion; tại cùng một thời điểm chỉ một thread lấy được distributed lock để truy cập shared resource.

## Distributed lock cần có những điều kiện nào?

Một distributed lock cơ bản nhất cần đáp ứng:

- **Mutual exclusion**: với cùng một lock key tương ứng với cùng một resource, tại cùng một thời điểm chỉ được có một holder hợp lệ. Cần thiết kế lock key theo granularity của resource, chẳng hạn `stock:{skuId}`, `order:{orderId}`, tránh nhồi các resource không liên quan vào một global lock.
- **High availability và chống deadlock**: bản thân lock service cần cố gắng duy trì availability; đồng thời phải có expiration time, session mechanism hoặc lease mechanism để tránh lock không bao giờ được giải phóng sau khi client crash. Tuy nhiên expiration time phải được thiết kế cùng execution time của business và renewal mechanism, nếu không có thể xảy ra lock hết hạn sớm khiến hai client đồng thời đi vào critical section.
- **Giải phóng an toàn**: khi giải phóng lock phải kiểm tra identity của lock holder, chỉ được giải phóng lock do chính mình nắm giữ. Ví dụ với Redis, khi lấy lock ghi một value ngẫu nhiên; khi giải phóng dùng Lua script so sánh value trước rồi mới xóa key.

Ngoài ba điều kiện cơ bản trên, một distributed lock tốt còn cần đáp ứng các điều kiện sau:

- **Reentrant**: không phải mọi trường hợp đều bắt buộc, nhưng nếu cùng một thread/request chain có thể vào cùng một critical section nhiều lần thì cần ghi nhận lock holder và số lần reentrant, tránh tự block chính mình.
- **High performance**: thao tác lấy và giải phóng lock cần hoàn tất nhanh, đồng thời không được gây ảnh hưởng quá lớn đến performance của toàn hệ thống.
- **Ngữ nghĩa lấy lock rõ ràng**: lấy lock có thể là block để chờ, chờ trong thời gian giới hạn hoặc thất bại ngay lập tức. Trong production thường cần đặt thời gian chờ tối đa và retry backoff, không được chờ vô hạn.
- **Renewal mechanism**: cần thiết lập lock TTL dựa trên P99 execution time của business critical section; khi critical section có thể vượt TTL, cần dùng watchdog/lease renewal hoặc rút ngắn critical section.
- **Fencing Token**: các trường hợp nghiêm ngặt hơn còn cần Fencing Token. Mỗi lần lấy lock thành công sẽ tạo một token tăng đơn điệu; downstream resource chỉ chấp nhận write có token lớn hơn, dùng để chặn late write của holder cũ sau khi lock hết hạn.

## Có những cách implementation distributed lock phổ biến nào?

Các giải pháp implementation distributed lock phổ biến:

- Dùng relational database như MySQL để implementation distributed lock.
- Dùng distributed coordination service ZooKeeper để implementation distributed lock.
- Dùng high-performance key-value store (Key-Value Store) như Redis hoặc distributed consistent key-value store như etcd để implementation distributed lock.

Các giải pháp database đại khái có ba loại: insert vào lock table có unique index, row lock dựa trên transaction với `SELECT ... FOR UPDATE`, và named lock như `GET_LOCK()` của MySQL. Chúng đều có thể thực hiện mutual exclusion ở mức độ nhất định, nhưng khác nhau về performance, thời điểm giải phóng, ngữ nghĩa timeout và cách khôi phục khi gặp lỗi.

Không phải database solution không thể xử lý việc lock hết hiệu lực, mà ngữ nghĩa hết hiệu lực và performance thường không tự nhiên bằng các solution như Redis/ZooKeeper/etcd. Ví dụ, lock table có thể thêm field expiration time, nhưng phải xử lý việc lock hết hạn bị client khác giành lấy, clock consistency, cleanup task và transaction isolation; `GET_LOCK()` phụ thuộc vào ngữ nghĩa connection/session của MySQL, không phù hợp với mọi business chain.

Redis lock thường được dùng hơn trong các trường hợp cần high performance, critical section ngắn và cho phép dùng business idempotency để làm lớp bảo vệ cuối; ZooKeeper/etcd phù hợp hơn với các coordination scenario cần session semantics, sequential node, lease và consistency mạnh hơn, nhưng throughput, latency và chi phí vận hành thường cao hơn. Tôi đã viết riêng một bài để giới thiệu chi tiết hai solution Redis và ZooKeeper: [Tổng hợp các solution distributed lock phổ biến](./distributed-lock-implementations.md).

Cuối cùng cần nhắc lại: **distributed lock không phải distributed transaction. Lock chỉ kiểm soát việc concurrent access vào critical section, không bảo đảm database commit chắc chắn thành công, cũng không bảo đảm message sending và order write có tính atomic consistency. Tính nhất quán của business vẫn phải dựa vào local transaction, idempotency, state machine, compensation task và các cơ chế khác.**

## Tổng kết

Bài viết này chủ yếu giới thiệu:

- Công dụng của distributed lock: trong distributed system, các service/client khác nhau thường chạy trên những JVM process độc lập. Nếu nhiều JVM process dùng chung một resource thì local lock không thể thực hiện mutual exclusion khi truy cập resource.
- Các điều kiện distributed lock cần có: mutual exclusion, high availability và chống deadlock, giải phóng an toàn, reentrant, high performance, ngữ nghĩa lấy lock rõ ràng và renewal mechanism. Các trường hợp nghiêm ngặt hơn còn cần kết hợp Fencing Token.
- Các cách implementation distributed lock phổ biến: relational database như MySQL, distributed coordination service ZooKeeper, high-performance key-value store như Redis và distributed consistent key-value store như etcd.

<!-- @include: @article-footer.snippet.md -->
