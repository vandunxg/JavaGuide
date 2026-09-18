---
title: "Giải thích chi tiết các giải pháp distributed transaction: XA, AT, TCC, Saga, local message table và transaction message"
category: Distributed
description: "Giải thích chi tiết các giải pháp distributed transaction, bao quát nguyên lý, ưu nhược điểm, trường hợp sử dụng và trọng tâm phỏng vấn của 2PC, 3PC, XA, Seata AT, TCC, Saga, local message table, transaction message của RocketMQ, best-effort notification và các giải pháp khác."
tag:
  - Distributed Transaction
head:
  - - meta
    - name: keywords
      content: distributed transaction,2PC,3PC,XA,Seata AT,TCC,Saga,local message table,transaction message,RocketMQ,best-effort notification,eventual consistency,compensating transaction,distributed transaction interview questions
---

**Trên mạng đã có rất nhiều bài viết về distributed transaction, tại sao vẫn cần viết thêm một bài?**

1. Thứ nhất, tôi thấy phần lớn bài viết khá khó hiểu, không phù hợp với những bạn chưa có nhiều kinh nghiệm. Mục tiêu của bài này là giúp cả những bạn chưa có nhiều kinh nghiệm làm việc cũng thực sự hiểu được distributed transaction.
2. Thứ hai, tôi thấy phần lớn bài viết chưa đủ chi tiết, bỏ qua nhiều khái niệm quan trọng liên quan đến distributed transaction.

Trước khi trao đổi về distributed transaction, hãy cùng ôn lại một số khái niệm liên quan đến transaction.

> **Thông tin phiên bản**: Nội dung liên quan đến Seata trong bài dựa trên tài liệu 1.7+ / 2.x, transaction message của RocketMQ dựa trên tài liệu 4.9+ / 5.x. Tham số mặc định, API và cách deploy có thể khác nhau giữa các phiên bản; khi áp dụng, hãy ưu tiên tài liệu của phiên bản thực tế trong dự án.

Bài viết này tập trung vào cách phối hợp commit hoặc compensation cho việc ghi dữ liệu xuyên service. Trước khi đọc, bạn nên hiểu các đánh đổi trong [Giải thích chi tiết CAP theorem và BASE theory](./protocol/cap-and-base-theorem.md), cùng vấn đề mutual exclusion và lease trong [distributed lock](./distributed-lock.md); sau khi đọc xong, bạn có thể xem lại trong từng nghiệp vụ cụ thể trạng thái nào cần ràng buộc chặt, trạng thái nào có thể đạt eventual consistency.

## Transaction

Hãy hình dung một tình huống cần chèn nhiều dữ liệu có liên quan vào database, nhưng quá trình này có thể gặp các vấn đề sau:

- Database đột nhiên dừng hoạt động giữa chừng vì một nguyên nhân nào đó.
- Client đột nhiên không thể kết nối đến database vì vấn đề network.
- Khi truy cập database đồng thời, nhiều thread cùng ghi vào database và ghi đè thay đổi của nhau.
- …

Bất kỳ vấn đề nào ở trên cũng có thể khiến dữ liệu không nhất quán. Để bảo đảm tính nhất quán của dữ liệu, system phải xử lý được các vấn đề này. Transaction là cơ chế đầu tiên được trừu tượng hóa để đơn giản hóa việc xử lý. Khái niệm transaction bắt nguồn từ database và hiện đã trở thành một khái niệm phổ biến.

**Transaction là gì?** Nói ngắn gọn, **transaction là một nhóm thao tác logic, hoặc tất cả cùng thực thi, hoặc không thao tác nào được thực thi.**

Ví dụ kinh điển và thường được nhắc đến nhất của transaction là chuyển tiền. Giả sử Tiểu Minh muốn chuyển 1.000 đồng cho Tiểu Hồng, việc chuyển tiền này gồm hai thao tác quan trọng và cả hai phải cùng thành công hoặc cùng thất bại.

1. Giảm số dư của Tiểu Minh 1.000 đồng.
2. Tăng số dư của Tiểu Hồng 1.000 đồng.

Transaction xem hai thao tác này là một tổng thể logic. Các thao tác trong tổng thể này hoặc cùng thành công, hoặc cùng thất bại. Nhờ vậy sẽ không xảy ra tình trạng số dư của Tiểu Minh bị giảm nhưng số dư của Tiểu Hồng lại không tăng.

![](https://oss.javaguide.cn/github/javaguide/mysql/%E4%BA%8B%E5%8A%A1%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

## Database transaction

Trong phần lớn trường hợp, khi nói về transaction mà không chỉ rõ **distributed transaction**, người ta thường đang nói đến **database transaction**.

Database transaction là loại transaction chúng ta thường gặp nhất trong công việc hằng ngày. Nếu project của bạn dùng monolithic architecture, transaction bạn tiếp xúc thường là database transaction.

**Database transaction có tác dụng gì?**

Nói đơn giản, database transaction bảo đảm nhiều thao tác trên database, tức các câu lệnh SQL, tạo thành một tổng thể logic. Các thao tác database tạo thành tổng thể logic này tuân theo nguyên tắc: **hoặc tất cả cùng thực thi thành công, hoặc không thao tác nào được thực thi**.

```sql
# Bắt đầu một transaction
START TRANSACTION;
# Nhiều câu lệnh SQL
SQL1,SQL2...
## Commit transaction
COMMIT;
```

![Minh họa database transaction](https://oss.javaguide.cn/github/javaguide/mysql/%E6%95%B0%E6%8D%AE%E5%BA%93%E4%BA%8B%E5%8A%A1%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

Ngoài ra, transaction của relational database, chẳng hạn `MySQL`, `SQL Server`, `Oracle`, đều có các đặc tính **ACID**:

![ACID](https://oss.javaguide.cn/github/javaguide/mysql/ACID.png)

1. **Tính nguyên tử** (`Atomicity`): transaction là đơn vị thực thi nhỏ nhất và không được phép chia tách. Tính nguyên tử bảo đảm thao tác hoặc hoàn tất toàn bộ, hoặc hoàn toàn không có tác dụng.
2. **Tính nhất quán** (`Consistency`): trước và sau khi thực thi transaction, dữ liệu vẫn nhất quán. Ví dụ trong nghiệp vụ chuyển tiền, dù transaction thành công hay thất bại, tổng số tiền của người chuyển và người nhận phải không đổi.
3. **Tính cô lập** (`Isolation`): khi truy cập database đồng thời, transaction của một user không bị transaction khác can thiệp; database giữa các transaction đồng thời là độc lập.
4. **Tính bền vững** (`Durability`): sau khi transaction được commit, thay đổi của nó đối với dữ liệu trong database là bền vững; ngay cả khi database gặp sự cố, thay đổi đó cũng không bị ảnh hưởng.

🌈 Có một điểm cần bổ sung: **chỉ sau khi bảo đảm tính bền vững, tính nguyên tử và tính cô lập của transaction thì tính nhất quán mới được bảo đảm. Nói cách khác, A, I, D là phương tiện, còn C là mục tiêu!** Có lẽ cũng như tôi, nhiều người đã bị khái niệm ACID đánh lạc hướng trong thời gian dài! Tôi cũng chỉ làm rõ điều này sau khi xem khóa học công khai [《Khóa học software architecture của Châu Chí Minh》](https://time.geekbang.org/opencourse/intro/100064201) (hãy đọc thêm sách hay!!!).

![AID->C](https://oss.javaguide.cn/github/javaguide/mysql/AID-%3EC.png)

Ngoài ra, tác giả của DDIA, tức [《Designing Data-Intensive Application (Thiết kế hệ thống ứng dụng xử lý nhiều dữ liệu)》](https://book.douban.com/subject/30329536/), cũng viết như sau trong cuốn sách này:

> Atomicity, isolation, and durability are properties of the database, whereas consis‐
> tency (in the ACID sense) is a property of the application. The application may rely
> on the database’s atomicity and isolation properties in order to achieve consistency,
> but it’s not up to the database alone.
>
> Ý nghĩa khi dịch là: tính nguyên tử, tính cô lập và tính bền vững là thuộc tính của database, còn tính nhất quán (theo nghĩa ACID) là thuộc tính của application. Application có thể dựa vào tính nguyên tử và tính cô lập của database để đạt được tính nhất quán, nhưng điều này không chỉ phụ thuộc vào database. Vì vậy, chữ C không thuộc về ACID.

Cuốn 《Designing Data-Intensive Application (Thiết kế hệ thống ứng dụng xử lý nhiều dữ liệu)》 rất đáng đọc và có thể đọc nhiều lần! Gần 90% người dùng Douban đánh giá năm sao sau khi đọc cuốn sách này. Bản dịch tiếng Trung đã được mở mã nguồn trên GitHub, tại: [https://github.com/Vonng/ddia](https://github.com/Vonng/ddia).

![](https://img-blog.csdnimg.cn/20210526162552353.png)

**Nguyên lý triển khai database transaction là gì?**

Ở đây, chúng ta lấy engine InnoDB của MySQL làm ví dụ để nói ngắn gọn.

Engine InnoDB của MySQL dùng **redo log (redo log)** để bảo đảm **tính bền vững** của transaction, dùng **undo log (undo log)** để bảo đảm **tính nguyên tử** của transaction. Engine InnoDB của MySQL dùng **cơ chế lock**, **MVCC** và các phương thức khác để bảo đảm tính cô lập của transaction (isolation level mặc định được hỗ trợ là **`REPEATABLE-READ`**).

## Distributed transaction

Trong microservice architecture, một system được tách thành nhiều microservice nhỏ. Mỗi microservice có thể nằm trên một machine khác nhau và mỗi microservice có thể có một database riêng để sử dụng. Trong tình huống này, một nhóm thao tác có thể liên quan đến nhiều microservice và nhiều database. Ví dụ, trong e-commerce system, việc tạo một order thường liên quan đến order service (tăng số order lên một), inventory service (giảm inventory đi một), v.v. Các service này có database riêng để sử dụng.

![Minh họa distributed transaction](https://oss.javaguide.cn/github/javaguide/distributed-system/distributed-transaction/distributed-transaction-with-two-services.png)

**Vậy làm thế nào để bảo đảm nhóm thao tác này hoặc cùng thực thi thành công, hoặc cùng thất bại?**

Chỉ dựa vào database transaction thì không đủ! Lúc này cần đưa vào khái niệm **distributed transaction**!

Thực tế, mọi tình huống xuyên database đều cần dùng distributed transaction. Ví dụ khi performance của một database đạt bottleneck hoặc lượng dữ liệu quá lớn, chúng ta cần **sharding database**. Sau khi sharding, các table trong cùng một database được phân bố trên nhiều database khác nhau. Nếu một thao tác liên quan đến nhiều database, transaction tích hợp sẵn của database sẽ không còn đáp ứng được yêu cầu.

Nói ngắn gọn, **mục tiêu cuối cùng của distributed transaction là bảo đảm tính nhất quán của dữ liệu trong nhiều database có liên quan trong system!**

Distributed transaction cũng là transaction, nên về lý thuyết phải tuân theo bốn đặc tính ACID. Tuy nhiên, xét đến performance, availability và các yếu tố khác, thường không thể đáp ứng đầy đủ ACID mà chỉ có thể chọn một giải pháp cân bằng hơn.

Đối với distributed transaction, một số lý thuyết mới đã ra đời.

## Lý thuyết nền tảng của distributed transaction

### CAP theory và BASE theory

CAP theory và BASE theory là kiến thức nền tảng để hiểu các đánh đổi của distributed transaction. Khi ghi dữ liệu xuyên service, system thường không thể đồng thời theo đuổi consistency mạnh, availability liên tục và độ phức tạp thấp. Cuối cùng, quyết định mang tính engineering sẽ là bước nào bắt buộc phải có ràng buộc mạnh, bước nào có thể compensation để hội tụ.

Ở đây không trình bày đầy đủ định nghĩa CAP và BASE. Bạn nên đọc trước [Giải thích chi tiết CAP theorem và BASE theory](./protocol/cap-and-base-theorem.md). Nếu muốn hiểu sâu hơn vì sao Leader/Quorum, split-brain, Lease và Fencing Token ảnh hưởng đến lock, transaction và configuration center, bạn có thể đọc tiếp [Giải thích chi tiết distributed coordination](./protocol/centralized-and-decentralized.md).

### 3 level của consistency

Có thể chia yêu cầu về consistency của system thành 3 level sau:

- **Strong consistency**: system ghi gì thì đọc ra đúng cái đó.
- **Weak consistency**: không nhất thiết đọc được giá trị mới nhất vừa ghi, cũng không bảo đảm sau bao lâu sẽ đọc được dữ liệu mới nhất; chỉ cố gắng bảo đảm dữ liệu đạt trạng thái nhất quán tại một thời điểm nào đó.
- **Eventual consistency**: phiên bản nâng cấp của weak consistency. System bảo đảm dữ liệu đạt trạng thái nhất quán trong một khoảng thời gian nhất định.

Ngoài 3 level consistency phổ biến trên, còn có các consistency model như read-write consistency và causal consistency. Bạn có thể tham khảo bài luận [《Operational Characterization of Weak Memory Consistency Models》](https://es.cs.uni-kl.de/publications/datarsg/Senf13.pdf). Vì các consistency model này ít gặp trong công việc hằng ngày nên ở đây không trình bày thêm (bản thân tôi cũng không thực sự hiểu rõ 😅).

Trong thực tế, eventual consistency được dùng phổ biến hơn; nhưng với các nghiệp vụ yêu cầu consistency nghiêm ngặt như chuyển khoản ngân hàng, vẫn phải bảo đảm strong consistency.

### Flexible transaction

Điều quan trọng nhất của internet application là bảo đảm high availability. Distributed system không thể sử dụng trong vài giây cũng có thể gây tổn thất rất lớn. Trong bối cảnh này, một số chuyên gia đã dựa trên CAP theory và BASE theory để đề xuất khái niệm **flexible transaction**. **Flexible transaction theo đuổi eventual consistency.**

Thực tế, flexible transaction chính là **BASE theory + business practice**. Mục tiêu của flexible transaction là dựa trên đặc tính của nghiệp vụ để bảo đảm eventual consistency của dữ liệu system bằng phương thức phù hợp. **TCC**, **Saga**, **MQ transaction**, **local message table** đều là flexible transaction.

### Rigid transaction

Đối lập với flexible transaction là **rigid transaction**. Như đã nói, **flexible transaction theo đuổi eventual consistency**. Tương ứng, rigid transaction theo đuổi **strong consistency**. **2PC** và **3PC** thuộc rigid transaction.

![Tổng hợp các giải pháp distributed transaction](https://oss.javaguide.cn/github/javaguide/distributed-system/distributed-transaction/distributed-transaction-solution-summary.png)

## Giải pháp distributed transaction

Có nhiều giải pháp distributed transaction, chẳng hạn **XA / 2PC**, **3PC**, **AT**, **TCC**, **Saga**, **local message table**, **MQ transaction** (Kafka và RocketMQ đều cung cấp chức năng liên quan đến transaction), **best-effort notification**, v.v.

2PC và 3PC là các giải pháp không xâm lấn business code. XA specification là tiêu chuẩn distributed transaction processing (DTP, Distributed Transaction Processing) do tổ chức X/Open định nghĩa, quy định interface giữa TM và RM và thường hoàn tất commit thông qua 2PC. TCC và Saga là các giải pháp xâm lấn business code, AT nằm giữa XA và TCC, MQ transaction phụ thuộc vào tình huống sử dụng message queue, còn local message table và best-effort notification chủ yếu theo đuổi eventual consistency và không hỗ trợ rollback tự động.

Các giải pháp này có trường hợp sử dụng khác nhau. Cần chọn giải pháp phù hợp cho project theo từng tình huống cụ thể.

Một cách chọn đơn giản:

- **Không thể chấp nhận eventual consistency**: ưu tiên xem xét XA / 2PC, phù hợp với các transaction ngắn cần strong consistency như tài chính và kế toán, nhưng phải chấp nhận chi phí về performance và availability.
- **Muốn ít thay đổi business**: có thể đánh giá Seata AT, local message table hoặc MQ transaction. AT ít xâm lấn business code, nhưng yêu cầu về cấu trúc table database, phạm vi hỗ trợ SQL và global lock.
- **Có thể chấp nhận business code bị xâm lấn và chain ngắn**: có thể cân nhắc TCC. Các tình huống điển hình là đóng băng account, reserve inventory, lock coupon và các nghiệp vụ cần reserve resource.
- **Chain dài, nhiều bước**: có thể cân nhắc Saga. Các tình huống điển hình là order fulfillment, travel booking và approval flow.
- **Chỉ cần thông báo để phía bên kia hoàn tất sau cùng**: có thể cân nhắc best-effort notification, chẳng hạn payment callback, thông báo trạng thái logistics và phát point.

Trước khi giới thiệu 2PC và 3PC, hãy giới thiệu một số role liên quan đến 2PC và 3PC (thành phần role trong XA specification):

![](https://oss.javaguide.cn/github/javaguide/distributed-system/distributed-transaction/xa-specification-roles.png)

- **AP (Application Program)**: chính application.
- **RM (Resource Manager)**: resource manager, cũng là participant của transaction; phần lớn trường hợp là database (phần sau sẽ lấy relational database làm ví dụ). Một distributed transaction thường liên quan đến nhiều RM.
- **TM (Transaction Manager)**: transaction manager, chịu trách nhiệm quản lý global transaction, phân bổ định danh duy nhất cho transaction, giám sát tiến độ thực thi transaction và chịu trách nhiệm commit, rollback, failure recovery của transaction.

### 2PC (Two-Phase Commit protocol)

![](https://oss.javaguide.cn/github/javaguide/distributed-system/distributed-transaction/2pc-work-flow.png)

Ý nghĩa của ba ký tự trong 2PC (Two-Phase Commit):

- **2** -> chỉ 2 phase của transaction commit.
- **P** -> Prepare (prepare phase).
- **C** -> Commit (commit phase).

2PC chia quá trình commit transaction thành 2 phase: **prepare phase** và **commit phase**.

#### Prepare phase

Cốt lõi của prepare phase là “hỏi” participant của transaction xem thao tác database transaction local có thành công hay không.

Quy trình của prepare phase:

1. **Transaction coordinator/manager (gọi tắt là TM ở phần sau)** gửi message đến tất cả **transaction participant (gọi tắt là RM ở phần sau)** để hỏi: “Bạn có thể thực thi thao tác transaction không?”, rồi chờ phản hồi.
2. Sau khi nhận message, **RM** bắt đầu thực thi thao tác chuẩn bị của database transaction local, chẳng hạn ghi redo log/undo log; **lúc này transaction chưa được commit**.
3. Nếu **RM** thực thi thành công thao tác database transaction local, RM trả lời “Yes” để cho biết đã sẵn sàng; nếu không, trả lời “No” để cho biết chưa sẵn sàng.

#### Commit phase

Cốt lõi của commit phase là “hỏi” participant của transaction xem commit transaction local có thành công hay không.

Khi tất cả participant của transaction đều ở trạng thái “ready”:

1. **TM** gửi message đến tất cả participant: “Các bạn có thể commit transaction!” (**Commit message**).
2. Sau khi nhận **Commit message**, **RM** thực thi thao tác **commit database transaction local**. Sau khi hoàn tất, RM **giải phóng resource đã chiếm dụng trong toàn bộ thời gian của transaction**.
3. **RM** trả lời: “Transaction đã commit” (**ACK message**).
4. Sau khi **TM** nhận **ACK message** của tất cả **transaction participant**, toàn bộ quá trình distributed transaction chính thức kết thúc.

![Minh họa 2PC - ready](https://oss.javaguide.cn/github/javaguide/distributed-system/distributed-transaction/distributed-transaction-2pc-ready.png)

Khi bất kỳ participant nào của transaction ở trạng thái “not ready”:

1. **TM** gửi message đến tất cả participant: “Các bạn có thể thực thi rollback!” (**Rollback message**).
2. Sau khi nhận **Rollback message**, **RM** thực thi **rollback database transaction local**, sau đó **giải phóng resource đã chiếm dụng trong toàn bộ thời gian của transaction**.
3. **RM** trả lời: “Transaction đã rollback” (**ACK message**).
4. Sau khi **TM** nhận **ACK message** của tất cả **RM**, transaction bị dừng.

![Minh họa 2PC - not ready](https://oss.javaguide.cn/github/javaguide/distributed-system/distributed-transaction/distributed-transaction-2pc-not-ready.png)

#### Tổng kết

Tóm tắt một số điểm quan trọng trong 2 phase của **2PC**:

1. Mục đích chính của **prepare phase** là kiểm tra xem **RM** có thể thực thi thao tác **database transaction local** hay không (!!!chú ý: bước này không commit transaction).
2. Trong **commit phase**, **TM** quyết định thực thi commit hay rollback transaction dựa trên message của **RM** trong **prepare phase**.
3. Sau **commit phase**, distributed transaction hiện tại chắc chắn kết thúc.

**Ưu điểm của 2PC:**

- Mô hình lý thuyết đơn giản, dễ hiểu và triển khai.
- Các database phổ biến như MySQL InnoDB, Oracle và PostgreSQL thường hỗ trợ XA, có thể làm RM trong 2PC và được TM bên ngoài điều phối commit hoặc rollback.

**Đánh đổi của 2PC:**

- Mục tiêu thiết kế của 2PC là strong consistency của dữ liệu. Tuy nhiên trong triển khai thực tế, do network partition, TM dừng hoạt động, RM timeout và các tình huống cực đoan khác, vẫn có thể xảy ra dữ liệu không nhất quán hoặc transaction bị block trong thời gian dài. 2PC không đồng nghĩa với “strong consistency tuyệt đối một cách tự nhiên”.

**Các vấn đề của 2PC:**

- **Synchronous blocking**: participant của transaction sẽ luôn chiếm resource liên quan trước khi chính thức commit transaction. Ví dụ Tiểu Minh chuyển tiền cho Tiểu Hồng, nếu transaction khác cũng muốn thao tác với Tiểu Minh hoặc Tiểu Hồng thì transaction đó sẽ bị block.
- **Dữ liệu không nhất quán**: vấn đề network hoặc TM dừng hoạt động đều có thể khiến dữ liệu không nhất quán. Ví dụ ở phase 2 (commit phase), nếu một phần network gặp vấn đề khiến một số participant không nhận được Commit/Rollback message thì dữ liệu sẽ không nhất quán.
- **Single point**: TM cũng là một role rất quan trọng. Nếu TM dừng hoạt động sau khi hoàn tất prepare phase, participant của transaction sẽ bị kẹt mãi ở commit phase.

### XA mode

XA có thể được hiểu là việc chuẩn hóa 2PC ở tầng resource như database. 2PC là commit protocol, còn XA là interface specification của DTP do X/Open định nghĩa, quy định cách TM điều phối nhiều RM cùng tham gia một global transaction.

Quy trình điển hình:

1. AP khởi tạo global transaction, TM chịu trách nhiệm tạo global transaction context.
2. Mỗi RM, chẳng hạn các database khác nhau, thực thi transaction local nhưng chưa thực sự commit.
3. TM gọi `prepare` trên từng RM. Khi tất cả RM prepare thành công, TM gọi `commit`; nếu không thì gọi `rollback`.

Ưu điểm của XA là ít xâm lấn business code, database tự chịu trách nhiệm về isolation và khả năng rollback của transaction, phù hợp với transaction ngắn, strong consistency và áp lực concurrency không quá cao. Nhược điểm cũng rõ ràng: resource database bị chiếm dụng trong thời gian dài trong transaction, performance và availability đều dễ bị ảnh hưởng.

Seata bắt đầu hỗ trợ XA mode từ version 1.2. Seata XA mode tận dụng hỗ trợ của database, message service và các resource khác đối với XA protocol để quản lý branch transaction. Tính nhất quán tổng thể mạnh hơn, nhưng throughput thường không bằng các flexible solution như AT, TCC và Saga.

### 3PC (Three-Phase Commit protocol)

![](https://oss.javaguide.cn/github/javaguide/distributed-system/distributed-transaction/3pc-work-flow.png)

3PC là phiên bản tối ưu hóa dựa trên 2PC. Nó tách **Prepare phase** của 2PC thành hai phase độc lập: CanCommit (chỉ hỏi có thể commit hay không, không thực thi thao tác chuẩn bị của transaction) và PreCommit (thực thi thao tác chuẩn bị của transaction, ghi redo/undo log). Kết hợp với DoCommit cuối cùng, tổng cộng có ba phase:

1. CanCommit (hỏi có thể commit hay không).
2. PreCommit (thực thi thao tác chuẩn bị của transaction).
3. DoCommit (thực sự commit).

![Minh họa 3PC - ready](https://oss.javaguide.cn/github/javaguide/distributed-system/distributed-transaction/distributed-transaction-3pc-ready.png)

#### Prepare phase (CanCommit)

Trong prepare phase, RM không thực thi thao tác transaction. TM chỉ gửi **prepare request** đến RM và hỏi một số thông tin, chẳng hạn participant của transaction có thể thực thi thao tác database transaction local hay không. RM trả lời “Yes”, “No” hoặc timeout mà không trả lời.

#### PreCommit phase

Nếu tất cả RM trả lời “Yes” trong prepare phase, TM gửi **PreCommit message (pre-commit request)** đến tất cả RM. Sau khi nhận message, RM thực thi thao tác chuẩn bị của database transaction local, chẳng hạn ghi redo log/undo log.

Nếu bất kỳ RM nào trả lời “NO” trong prepare phase hoặc timeout mà không trả lời, TM gửi **Abort message (abort request)** đến tất cả RM. Sau khi nhận message, RM dừng transaction ngay. Việc này không gây thiệt hại lớn cho RM vì về bản chất RM vẫn chưa thực sự làm gì.

Nếu RM thực thi thao tác chuẩn bị của transaction thành công thì trả về “YES”; nếu không thì trả về “No” (cơ hội đổi ý cuối cùng).

TM và RM đều đưa timeout mechanism vào pre-commit phase. Nếu **participant** không nhận được PreCommit message từ TM hoặc TM không nhận được trạng thái kết quả thực thi trước do participant trả về, transaction sẽ bị dừng sau khi vượt quá thời gian chờ, nhờ đó tránh transaction bị block.

#### DoCommit phase

**DoCommit phase** bắt đầu thực hiện commit transaction thực sự.

Nếu tất cả RM trả lời “YES” trong pre-commit phase, TM gửi **DoCommit message (request thực thi commit transaction)** đến tất cả RM. Sau khi nhận message, RM commit database transaction local và giải phóng resource đã chiếm dụng sau khi hoàn tất. Khi transaction commit thành công, RM trả về “YES”.

Nếu bất kỳ RM nào trả lời “NO” trong pre-commit phase hoặc timeout mà không trả lời, TM gửi **Abort message (abort request)** đến tất cả RM. Sau khi nhận message, RM rollback transaction, giải phóng resource và dừng transaction này.

Nếu RM không nhận được DoCommit message từ TM trong thời gian đã đặt, RM cho rằng TM có thể đã gặp sự cố và sẽ trực tiếp commit transaction.

Chỉ cần tất cả RM trả về `Yes` trong pre-commit phase thì sau khi vào phase 3, transaction rất có khả năng thực thi thành công.

Tuy nhiên cần đặc biệt chú ý: **mặc định commit sau khi RM timeout là “con dao hai lưỡi” của 3PC**. Cơ chế này giảm vấn đề RM bị block vĩnh viễn do TM dừng hoạt động trong 2PC, nhưng cũng tạo ra rủi ro không nhất quán mới. Ví dụ TM ban đầu quyết định Abort, nhưng chỉ một phần RM nhận được Abort message, các RM khác mặc định commit do timeout, khiến cùng một transaction có trạng thái “một phần commit, một phần rollback” trên các RM khác nhau. Đây cũng là lý do quan trọng khiến 3PC hiếm khi được triển khai thực sự trong thực tế.

#### Tổng kết

**Ngoài việc tiếp tục tách Prepare phase của 2PC, 3PC còn cải tiến những gì?**

3PC đồng thời đưa **timeout mechanism** vào TM và RM. Nếu TM không nhận được message của RM trong một khoảng thời gian nhất định, nó mặc định thất bại và dừng transaction; nếu RM không nhận được chỉ thị tiếp theo của TM trong thời gian dài, nó sẽ chọn dừng hoặc commit tùy phase hiện tại, nhằm tránh resource bị block lâu dài.

Tuy nhiên, 3PC không giải quyết hoàn hảo vấn đề blocking của 2PC mà còn đưa vào các vấn đề mới như performance kém hơn và vẫn có thể không nhất quán dữ liệu. Vì vậy, 3PC không được ứng dụng rộng rãi trên thực tế. Hướng phổ biến hơn trong engineering là dùng consensus protocol như Paxos / Raft (thường kết hợp với replicated state machine) để giải quyết single point của coordinator và vấn đề nhất quán state, thay vì trực tiếp dùng 3PC.

### TCC (Compensating transaction)

TCC là một flexible transaction solution khá phổ biến hiện nay. Chuyên gia database Pat Helland đã thảo luận tư tưởng tránh distributed transaction truyền thống và chuyển sang business compensation trong bài luận [《Life beyond Distributed Transactions: an Apostate’s Opinion》](https://www.ics.uci.edu/~cs223/papers/cidr07p15.pdf) công bố năm 2007. Nếu quan tâm, bạn có thể đọc bài luận này.

Nói đơn giản, TCC là viết tắt của Try, Confirm và Cancel, gồm ba phase:

1. **Try phase**: thử thực thi. Hoàn tất business check và reserve các business resource cần thiết.
2. **Confirm phase**: xác nhận thực thi. Khi Try phase của tất cả transaction participant thực thi thành công, Confirm sẽ được thực thi. Confirm phase xử lý business resource đã reserve trong Try phase. Nếu không, Cancel sẽ được thực thi.
3. **Cancel phase**: hủy thực thi và giải phóng business resource đã reserve trong Try phase.

Mỗi phase do business code kiểm soát. Nhờ đó có thể tránh long transaction và giữ lock lâu ở tầng database, nhưng đổi lại business code bị xâm lấn mạnh hơn. Developer phải tự xử lý resource reservation, confirmation, cancellation, retry và exception compensation.

Hãy lấy tình huống chuyển tiền làm ví dụ:

1. **Try phase**: trong tình huống chuyển tiền, Try cần kiểm tra số dư account có đủ hay không; resource được reserve là số tiền chuyển.
2. **Confirm phase**: nếu Try phase thực thi thành công, Confirm phase sẽ thực thi thao tác trừ tiền thực sự.
3. **Cancel phase**: giải phóng số tiền chuyển đã reserve trong Try phase.

Thông thường khi dùng `TCC` mode, cần tự triển khai ba method `try`, `confirm`, `cancel` để đạt eventual consistency.

Trong trường hợp bình thường, các method `try`, `confirm` sẽ được thực thi.

![](https://oss.javaguide.cn/github/javaguide/distributed-system/distributed-transaction/distributed-transaction-tcc-confirm.png)

Khi xảy ra exception, các method `try`, `cancel` sẽ được thực thi.

![](https://oss.javaguide.cn/github/javaguide/distributed-system/distributed-transaction/distributed-transaction-tcc-cancel.png)

Nếu xảy ra vấn đề ở Try phase thì có thể thực thi Cancel. **Vậy nếu Confirm hoặc Cancel phase thất bại thì phải làm sao?**

TCC ghi transaction log và persist transaction log vào một storage medium nào đó, chẳng hạn local file, relational database hoặc ZooKeeper. Transaction log chứa trạng thái thực thi transaction. Dựa vào trạng thái này có thể xác định transaction đã commit thành công hay thất bại, cũng như thất bại ở bước nào. Nếu phát hiện Confirm hoặc Cancel phase thất bại, system sẽ retry và tiếp tục thử thực thi logic của Confirm hoặc Cancel phase. Số lần retry do framework cụ thể quyết định; nếu vẫn chưa thành công sau số lần retry tối đa thì thường cần cảnh báo và chuyển sang quy trình can thiệp thủ công.

Nếu code không có bug đặc biệt, xác suất xảy ra vấn đề ở Confirm hoặc Cancel phase tương đối thấp.

Khi triển khai TCC có ba vấn đề engineering kinh điển:

1. **Idempotency**: Confirm và Cancel có thể bị gọi lặp lại do network timeout, TC retry và các nguyên nhân khác; phải bảo đảm kết quả của nhiều lần thực thi là giống nhau. Cách thường dùng là duy trì transaction state table trong database và kiểm tra state trước mỗi lần thực thi.
2. **Empty rollback**: Try request không thực sự đến RM do vấn đề network, nhưng TM đã khởi tạo Cancel. Lúc này Cancel đối mặt với một transaction “chưa từng Try”, cần nhận diện và trả về thành công trực tiếp để tránh thực thi nhầm logic rollback.
3. **Hanging**: Cancel đến RM trước Try, sau đó Try mới đến muộn. Nếu Try tiếp tục reserve resource, resource này có thể không bao giờ có ai Confirm/Cancel. Cách giải quyết là kiểm tra trong Try xem transaction đã từng bị Cancel hay chưa; nếu đã Cancel thì từ chối thực thi Try.

**Transaction log có bị xóa không?** Có. Nếu transaction commit thành công (không throw exception), có thể xóa transaction log tương ứng để tiết kiệm resource.

**TCC mode không cần dựa vào hỗ trợ transaction của underlying data resource, nhưng cần tự triển khai nhiều code hơn**, nên là một distributed solution **xâm lấn business code**.

Tư tưởng của TCC transaction model tương tự 2PC. Tôi vẽ một hình đơn giản để so sánh hai mô hình.

![So sánh 2PC và TCC](https://oss.javaguide.cn/github/javaguide/distributed-system/distributed-transaction/2pc-vs-tcc.png)

**TCC khác 2PC/3PC như thế nào?**

- 2PC/3PC dựa vào transaction ở tầng database hoặc storage resource, còn TCC chủ yếu thực hiện bằng cách sửa business code.
- 2PC/3PC không xâm lấn business code, còn TCC xâm lấn business code.
- 2PC/3PC theo đuổi strong consistency và giữ database lock trong toàn bộ quá trình two-phase commit. TCC theo đuổi eventual consistency và không giữ lock của các business resource trong suốt thời gian đó.

Đối với việc triển khai TCC, trong ngành cũng có một số open-source framework tốt. Cách các framework triển khai TCC có thể hơi khác nhau, nhưng tư tưởng nhìn chung giống nhau.

1. **[ByteTCC](https://github.com/liuyangming/ByteTCC)**: implementation của distributed transaction manager dựa trên cơ chế Try-Confirm-Cancel (TCC). Bài đọc thêm: [Một số suy nghĩ về cách triển khai TCC distributed transaction framework](https://www.bytesoft.org/how-to-impl-tcc/).
2. **[Seata](https://seata.apache.org/zh-cn/)**: Seata là một distributed transaction solution open source, đồng thời hỗ trợ bốn mode AT, TCC, Saga và XA. Ở đây nói đến TCC mode của Seata.
3. **[Hmily](https://gitee.com/dromara/hmily)**: flexible distributed transaction solution cấp độ tài chính. Khi chọn cho project mới, nên đồng thời đánh giá mức độ hoạt động của community và các alternative như Seata.

### AT mode (Automatic compensation)

AT (Automatic Transaction) mode là một trong các mode cốt lõi của Seata, nhằm cung cấp eventual consistency với mức thay đổi business code thấp nhất có thể. Có thể hiểu đây là giải pháp “automatic compensation”: business vẫn commit transaction local như bình thường, còn framework sẽ ghi lại thông tin cần thiết để rollback.

AT mode gồm hai phase chính:

1. **Phase 1**: business SQL thực thi và commit transaction local như bình thường. Đồng thời, data source proxy của Seata parse SQL, ghi before image / after image vào table `undo_log` và đăng ký branch transaction với TC.
2. **Commit phase 2**: nếu global transaction commit, TC thông báo RM xóa `undo_log` bất đồng bộ.
3. **Rollback phase 2**: nếu global transaction rollback, RM tạo compensation SQL ngược dựa trên `undo_log` và khôi phục dữ liệu về before image.

Ưu điểm của AT mode là ít xâm lấn business, phù hợp với các tình huống CRUD thông thường dựa trên relational database; nhược điểm là có yêu cầu về loại SQL, primary key của table, global lock, isolation level và các yếu tố khác, không phù hợp với mọi SQL phức tạp và tình huống xuyên non-relational resource.

### TCC và Saga

| Dimension                  | TCC                                                                   | Saga                                                                  |
| -------------------------- | --------------------------------------------------------------------- | --------------------------------------------------------------------- |
| Xử lý resource             | Reserve hoặc freeze resource trong Try phase                          | Mỗi local transaction commit trực tiếp                                |
| Isolation                  | Dùng business reservation để tạo “pseudo-isolation”                   | Isolation yếu, kết quả đã commit có thể bị transaction khác nhìn thấy |
| Business code bị xâm lấn   | Cao, cần ba nhóm logic Try/Confirm/Cancel                             | Cao, cần thao tác forward và compensation cho từng bước               |
| Transaction length phù hợp | Phù hợp hơn với chain ngắn và tình huống resource reservation rõ ràng | Phù hợp hơn với chain dài và flow nhiều bước                          |
| Tình huống điển hình       | Chuyển tiền, freeze inventory, lock coupon                            | Order fulfillment, travel booking, approval flow                      |

### MQ transaction

RocketMQ, Kafka, Pulsar và QMQ đều cung cấp chức năng liên quan đến transaction. Transaction cho phép ứng dụng event stream định nghĩa toàn bộ quá trình consume, process và produce message thành một thao tác nguyên tử.

Ở đây lấy RocketMQ làm ví dụ (hình lấy từ 《Khóa học chuyên sâu về message queue》). Bài đọc thêm: [Tài liệu tham khảo RocketMQ transaction message](https://rocketmq.apache.org/docs/featureBehavior/04transactionmessage/).

![](https://img-blog.csdnimg.cn/2021060810404597.png)

1. Bên gửi MQ, chẳng hạn logistics service, mở một transaction trên message queue rồi gửi một “half message” đến MQ Server/Broker. Trước khi transaction commit, half message không hiển thị với subscriber/consumer của MQ, chẳng hạn third-party notification service.
2. Nếu gửi “half message” thành công, bên gửi MQ bắt đầu thực thi transaction local.
3. Nếu transaction local của bên gửi MQ thực thi thành công, “half message” trở thành normal message và có thể được consume bình thường. Nếu transaction local thất bại, bên gửi MQ sẽ rollback trực tiếp.

Từ quy trình trên có thể thấy transaction message của RocketMQ mượn tư tưởng two-phase commit: trước tiên gửi half message, half message không hiển thị với consumer; sau khi transaction local thực thi thành công thì commit half message, biến nó thành normal message để consumer consume. Đây không phải 2PC theo semantics XA truyền thống mà là một cơ chế do MQ thiết kế để bảo đảm eventual consistency giữa “kết quả transaction local” và “message visibility”.

**Nếu bên gửi MQ thất bại khi commit hoặc rollback transaction message thì phải làm sao?**

Broker trong RocketMQ sẽ định kỳ truy vấn ngược bên gửi MQ để kiểm tra tình trạng thực thi transaction local của transaction này và quyết định commit hoặc rollback transaction dựa trên kết quả truy vấn.

Việc triển khai cơ chế truy vấn ngược transaction phụ thuộc vào interface tương ứng do business code triển khai. Ví dụ, nếu muốn kiểm tra transaction local tạo thông tin logistics có thực thi thành công hay không, chỉ cần query database để xem thông tin logistics tương ứng có tồn tại hay không.

![](https://img-blog.csdnimg.cn/20210608114710962.png)

**Nếu normal message không được consume chính xác thì phải làm sao?**

Nếu consume message thất bại, RocketMQ sẽ tự động retry consume. Nếu message vẫn chưa được consume chính xác sau số lần retry tối đa, RocketMQ sẽ cho rằng message này có vấn đề và đưa nó vào **dead-letter queue**.

![](https://img-blog.csdnimg.cn/20210608120207740.png)

Message vào dead-letter queue thường cần xử lý thủ công và kiểm tra vấn đề bằng tay.

Transaction message của **QMQ** không phức tạp như implementation của RocketMQ vì tận dụng transaction tích hợp sẵn của database. Tư tưởng cốt lõi thực ra là giải pháp **local message table** do eBay đề xuất, tách distributed transaction thành các local transaction để xử lý.

Chúng ta duy trì một local message table để lưu trạng thái gửi message. Thao tác lưu tình trạng gửi message vào local message table và business operation phải commit trong cùng một transaction. Như vậy, business thực thi thành công đồng nghĩa message table cũng ghi thành công.

Sau đó, khởi động riêng một thread để định kỳ polling message table và gửi các message chưa được xử lý đến message middleware.

Sau khi gửi message thành công, cập nhật message state thành success hoặc xóa trực tiếp message.

Điểm khác biệt cốt lõi của hai nhóm giải pháp nằm ở mức độ phụ thuộc vào availability của MQ:

- **RocketMQ transaction message**: transaction local không phụ thuộc vào Broker, nhưng trước khi thực thi transaction local cần gửi half message thành công. Khi Broker không khả dụng, việc gửi half message sẽ thất bại. Application layer cần quyết định fail fast, retry hay degrade sang compensation flow khác; không thể đơn giản hiểu là “toàn bộ application bị dừng”.
- **QMQ / local message table**: message trước tiên được ghi vào database local của business và nằm trong cùng một local transaction với business operation. MQ tạm thời không khả dụng không ảnh hưởng đến việc commit business transaction; sau đó một thread độc lập tiếp tục deliver message.

Vì vậy, local message table có khả năng chịu đựng MQ tạm thời không khả dụng cao hơn, nhưng đổi lại business cần duy trì message table, delivery thread, retry strategy, idempotent consume và reconciliation mechanism. QMQ chỉ đóng gói giải pháp local message table này đầy đủ hơn và dễ sử dụng hơn.

Bài đọc thêm: [Interviewer: Nhược điểm của RocketMQ distributed transaction message?](https://mp.weixin.qq.com/s/cBx1l1zaThN6_808fMl27g)

### Saga

Saga có lịch sử rất lâu đời. Saga transaction theory đã được Hector & Kenneth đề xuất trong bài luận [《Sagas》](https://www.cs.cornell.edu/andru/cs711/2002fa/reading/sagas.pdf) công bố tại ACM năm 1987, sớm hơn cả khi khái niệm distributed transaction ra đời.

Saga là long transaction solution. Tư tưởng cốt lõi là tách long transaction thành nhiều local short transaction (chuỗi local short transaction).

![](https://oss.javaguide.cn/github/javaguide/distributed-system/distributed-transaction/distributed-transaction-saga.png)

- Long transaction —> T1,T2 ~ Tn local short transaction.
- Mỗi short transaction có một compensation action —> C1,C2 ~ Cn.

Hình dưới đây lấy từ [tài liệu kỹ thuật Microsoft — Saga distributed transaction](https://docs.microsoft.com/zh-cn/azure/architecture/reference-architectures/saga/saga).

![](https://img-blog.csdnimg.cn/20210611101344496.png)

Nếu các short transaction T1,T2 ~ Tn đều hoàn tất thuận lợi, toàn bộ transaction cũng kết thúc thuận lợi; nếu không, system sẽ dùng recovery mode.

**Backward recovery:**

- Giới thiệu: nếu short transaction Ti commit thất bại, compensation tất cả transaction đã hoàn tất (liên tục thực thi Ci để compensation cho Ti).
- Thứ tự thực thi: T1, T2, …, Ti (thất bại), Ci (compensation), …, C2, C1.

**Forward recovery:**

- Giới thiệu: nếu short transaction Ti commit thất bại, liên tục retry Ti cho đến khi thành công.
- Thứ tự thực thi: T1, T2, …, Ti (thất bại), Ti (retry) …, Ti+1, …, Tn.

Tương tự TCC, forward operation và compensation operation của Saga đều cần business developer tự triển khai, nên cũng là một distributed solution **xâm lấn business code**. Một điểm khác biệt lớn giữa Saga và TCC là Saga không có thao tác “Try”; local transaction Ti của nó được commit trực tiếp. Vì vậy performance rất cao!

Bản thân compensation operation cũng là business code, nên có thể thất bại do network, external dependency không khả dụng, business rule thay đổi và các nguyên nhân khác. Saga framework thường dùng strategy “retry liên tục + giới hạn số lần retry tối đa + can thiệp thủ công” để xử lý compensation thất bại, vì vậy compensation action phải được thiết kế idempotent. Để tăng khả năng chịu lỗi (chẳng hạn bản thân Saga system cũng có thể dừng hoạt động), cần bảo đảm tất cả short transaction đều được commit hoặc compensation bằng cách ghi log các operation này (Saga log, tương tự cơ chế log của database). Nhờ vậy, sau khi Saga system khôi phục, chúng ta biết short transaction đã thực thi đến đâu hoặc compensation operation đã thực thi đến đâu.

Ngoài ra, vì Saga không reserve resource bằng thao tác “Try” nên không thể bảo đảm isolation. Đây cũng là một nhược điểm lớn của Saga.

Đối với việc triển khai Saga, trong ngành cũng có một số open-source framework tốt. Cách các framework triển khai Saga có thể hơi khác nhau, nhưng tư tưởng nhìn chung giống nhau.

1. **[ServiceComb Pack](https://github.com/apache/servicecomb-pack)**: giải pháp eventual consistency cho dữ liệu của microservice application.
2. **[Seata](https://seata.apache.org/zh-cn/)**: Seata là một distributed transaction solution open source, Saga là một trong các mode được hỗ trợ.

### Best-effort notification

Best-effort notification là một giải pháp eventual consistency nhẹ hơn, thường gặp trong các tình huống như payment callback, thông báo trạng thái logistics và phát point.

Tư tưởng rất đơn giản: sau khi bên khởi tạo hoàn tất transaction local, cố gắng hết sức để thông báo kết quả cho bên nhận. Nếu thông báo thất bại, tiếp tục retry theo khoảng thời gian cố định hoặc exponential backoff; interface của bên nhận phải bảo đảm idempotent. Sau số lần retry tối đa, thông thường sẽ chuyển sang xử lý thủ công hoặc reconciliation compensation.

Best-effort notification có thể được xem là biến thể đơn giản hóa của local message table: không cần đưa MQ vào mà bên khởi tạo trực tiếp retry gọi interface của bên nhận; nhưng reliability và khả năng traffic smoothing thường không bằng giải pháp đầy đủ “local message table + MQ + idempotent consume”.

## Giới thiệu tổng hợp về Seata solution

Seata là một distributed transaction solution one-stop khá phổ biến trong nước, với các role cốt lõi gồm:

- **TC (Transaction Coordinator)**: transaction coordinator, duy trì state của global transaction và branch transaction.
- **TM (Transaction Manager)**: transaction manager, định nghĩa global transaction boundary và chịu trách nhiệm khởi tạo, commit hoặc rollback global transaction.
- **RM (Resource Manager)**: resource manager, quản lý branch transaction resource, đăng ký branch transaction với TC và báo cáo state.

Seata hỗ trợ nhiều transaction mode:

| Mode | Business code bị xâm lấn | Mục tiêu consistency                            | Đặc điểm điển hình                                                  | Trường hợp sử dụng                                     |
| ---- | ------------------------ | ----------------------------------------------- | ------------------------------------------------------------------- | ------------------------------------------------------ |
| AT   | Thấp                     | Eventual consistency                            | Tự động tạo `undo_log`, tự động compensation ở phase 2              | CRUD thông thường trên relational database             |
| TCC  | Cao                      | Eventual consistency / business constraint mạnh | Business triển khai Try/Confirm/Cancel                              | Transaction chain ngắn cần resource reservation        |
| Saga | Cao                      | Eventual consistency                            | Tách long transaction thành local transaction + compensation        | Long flow, orchestration nhiều service                 |
| XA   | Thấp                     | Nghiêng về strong consistency                   | Phụ thuộc khả năng XA của database, thời gian giữ resource lock dài | Transaction ngắn, strong consistency, concurrency thấp |

Vì vậy, không nên đơn giản xếp Seata vào nhóm TCC hoặc Saga framework. Khi chọn lựa thực tế, cần dựa vào mức độ xâm lấn business, khả năng hỗ trợ của database, độ dài chain, yêu cầu performance và yêu cầu consistency để chọn mode phù hợp trong AT / TCC / Saga / XA.

## Bài viết đề xuất

Để thuận tiện cho việc học thêm, dưới đây là một số bài viết hay được chọn lọc để bạn tham khảo.

> **[Phân tích chuyên sâu Saga distributed transaction](https://segmentfault.com/a/1190000041001954)**
>
> **[Bảy giải pháp distributed transaction kinh điển nhất](https://segmentfault.com/a/1190000040321750)**
>
> **[Các cách dùng phổ biến này của distributed transaction đều có vấn đề, hãy xem cách làm đúng](https://segmentfault.com/a/1190000041031586)**

Diệp Đông Phú 👍👍👍👍👍

Viết rất tốt, tổng hợp giải pháp rất toàn diện và chuyên sâu.

> [So sánh 7 distributed transaction solution, vẫn thích Seata open source của Alibaba hơn, thật đáng dùng! (nguyên lý + thực chiến)](https://mp.weixin.qq.com/s/sXVSFqq2UZ6Pwwt7vx7vIA)
>
> Chuyên mục kỹ thuật Mã Viên 🗓️2021-10-25 👍👍👍👍👍
>
> Giới thiệu một số distributed solution chủ đạo hiện nay và distributed solution one-stop Seata open source của Alibaba.

Bài viết không chỉ giới thiệu lý thuyết mà còn thực hành AT mode của Seata.

> **[Thực hành distributed transaction Seata và giải thích open source chi tiết | Bản ghi GIAC](https://www.sofastack.tech/blog/seata-distributed-transaction-deep-dive/)**
>
> Trương Sâm 🗓️2019-07-02 👍👍👍👍👍
>
> Bài viết này là phần chia sẻ của Trương Sâm (tên hiệu: Thiệu Huy), chuyên gia kỹ thuật Ant Group và một trong những người khởi xướng Seata distributed transaction, tại Global Internet Architecture Conference GIAC. Nội dung giới thiệu chi tiết nguyên nhân phát sinh vấn đề distributed transaction và các biện pháp ứng phó của Ant Group (bốn mode AT, TCC, Saga và XA của Seata distributed transaction).

Bài viết có nhiều hình minh họa sinh động giúp chúng ta hiểu vấn đề! Quả thực là một bài viết hàng đầu!

> **[14.000 chữ, 25 hình giúp bạn hoàn toàn nắm được nguyên lý distributed transaction](https://mp.weixin.qq.com/s/qeUfEJFYCfyDjgzDnq_Jdw)**
>
> Mã Hải 🗓️2020-10-30 👍👍👍👍👍
>
> Chủ yếu giới thiệu single data source transaction và multi-data source transaction, các distributed transaction solution phổ biến và implementation của Seata in AT mode.

> **[6 hình giúp bạn hiểu hoàn toàn XA mode của distributed transaction](https://mp.weixin.qq.com/s/Rp8paKc2bQhERBGDKtpMcA)**
>
> Chu Tấn Quân 🗓️2021-04-25 👍👍👍
>
> Một bài viết về cloud-native của Alibaba, chủ yếu giới thiệu XA mode và implementation, optimization của Seata đối với XA mode. Nội dung giới thiệu khá khái quát, cần kết hợp với các bài viết liên quan để tìm hiểu sâu hơn.

## Open-source project về distributed transaction

1. **[Seata](https://seata.apache.org/zh-cn/)**: Seata là một distributed transaction solution open source, hỗ trợ các mode AT, TCC, Saga, XA và các mode khác, hướng đến cung cấp distributed transaction service có performance cao, dễ dùng trong microservice architecture.
2. **[Hmily](https://gitee.com/dromara/hmily)**: Hmily là một flexible distributed transaction solution có performance cao, cấp độ tài chính, chủ yếu cung cấp các solution như TCC và TAC (tự động tạo rollback SQL). Khi chọn cho project mới, nên chú ý mức độ hoạt động của community và đánh giá cùng Seata, DTM và các solution khác.
3. **[Raincat](https://gitee.com/dromara/Raincat)**: distributed transaction middleware sử dụng two-phase commit.
4. **[Myth](https://gitee.com/dromara/myth)**: open-source framework giải quyết distributed transaction bằng message queue, phát triển bằng Java (JDK 1.8), hỗ trợ các RPC framework như Dubbo, Spring Cloud và Motan.

## Tham khảo

- [Consistency protocol của distributed system: 2PC và 3PC - Matt -2018](https://matt33.com/2018/07/08/distribute-system-consistency-protocol/)
- [Dealing Distributed Transactions with 2PC, 3PC, Local Transaction Table with MQs - Adrian -2021](https://masteranyfield.com/2021/07/26/dealing-distributed-transactions-with-2pc-3pc-local-transaction-table-with-mqs/)
- [Làm thế nào để hiểu 3PC đã giải quyết vấn đề blocking của 2PC? - câu hỏi Zhihu](https://www.zhihu.com/question/422691164)
- [Tài liệu chính thức Apache Seata](https://seata.apache.org/docs/overview/what-is-seata/)
- [Seata XA mode](https://seata.apache.org/zh-cn/docs/user/mode/xa)
- [Seata TCC Fence: idempotency, empty rollback và hanging](https://seata.apache.org/blog/seata-tcc-fence)
- [Tài liệu chính thức RocketMQ transaction message](https://rocketmq.apache.org/docs/featureBehavior/04transactionmessage/)
