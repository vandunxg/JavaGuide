---
title: "Điều tra sự cố production Java backend: CPU, memory, GC, thread pool, database và Redis"
description: "Hướng dẫn điều tra sự cố production Java backend, bao quát xác nhận cảnh báo, giảm thiểu ảnh hưởng và lưu bằng chứng, CPU tăng vọt, OOM, GC thường xuyên, thread pool và connection pool cạn kiệt, SQL chậm, Redis bị block, message queue backlog và tổng kết sự cố."
category: Java
tag:
  - JVM
  - Xử lý sự cố production
  - Tối ưu performance
  - Phỏng vấn Java
sitemap:
  changefreq: monthly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Java điều tra sự cố production,điều tra sự cố JVM,CPU tăng vọt,xử lý OOM,Full GC,thread pool cạn kiệt,connection pool cạn kiệt,SQL chậm,sự cố Redis,MQ backlog
---

Khi cảnh báo vừa vang lên, việc điều tra rất dễ bị dẫn dắt bởi đường biểu diễn nổi bật nhất trên trang monitoring. Thấy CPU cao thì lấy thread stack, thấy Full GC thì kiểm tra heap, vừa deploy xong đã chuẩn bị rollback. Nhưng các hiện tượng này thường truyền dẫn lẫn nhau: SQL chậm có thể giữ database connection, việc chờ connection pool lại giữ các business thread, cuối cùng có thể đồng thời thấy latency, error rate và CPU tăng. Chỉ nhìn một chỉ số rất khó xác định root cause.

Khi sự cố vẫn đang lan rộng, trước tiên hãy kiểm soát ảnh hưởng theo phương án ứng phó. Nếu instance còn dư địa, sau khi xác nhận việc thu thập bằng chứng không tiếp tục làm service quá tải, hãy lưu log, Trace, thread stack và thông tin memory. Restart thường có thể khôi phục tạm thời, nhưng cũng sẽ xóa dấu vết; việc cứ để instance bất thường tiếp tục nhận traffic để lấy Heap Dump cũng có thể làm vấn đề trầm trọng hơn. Tình hình thực tế quyết định nên rút traffic trước, rollback hay lấy bằng chứng trước.

Phần dưới chỉ bàn về các vấn đề thường gặp ở phía ứng dụng Java. Nếu bằng chứng đã chỉ đến việc lập lịch container, kernel, network device hoặc cloud platform, cần chuyển sang monitoring và tài liệu vận hành của nền tảng tương ứng để tiếp tục điều tra; phần này không đi sâu.

::: warning Lưu ý khi thao tác trên production
Trước khi thực thi lệnh chẩn đoán trên production, trước tiên hãy kiểm tra instance còn đang nhận traffic hay không, disk còn bao nhiêu dung lượng và bản thân lệnh có overhead thế nào. Cần kiểm soát tần suất khi liên tục lấy thread stack; Heap Dump, class histogram và GC chủ động có thể gây mức sử dụng CPU, I/O hoặc pause đáng kể. Khi instance đã gần không thể sử dụng, trước tiên hãy rút traffic hoặc giảm thiểu thiệt hại, đừng tiếp tục gánh traffic nghiệp vụ chỉ để giữ dấu vết.
:::

## Sau khi nhận cảnh báo cần xác nhận gì trước?

Sau khi nhận cảnh báo, trước tiên hãy xem phía user có bất thường hay không. Một instance có CPU cao nhưng request vẫn được các instance khác tiếp nhận bình thường không nghiêm trọng như trường hợp error rate của toàn service cũng tăng. Đồng thời cần loại trừ false positive của monitoring, độ trễ thu thập metric và trường hợp downstream timeout khiến thread bị kẹt trong service này.

Hãy đặt request volume, error rate, P95/P99, CPU, memory, GC, thread pool và connection pool trong khoảng mười mấy phút trước và sau cảnh báo lên cùng một biểu đồ, sau đó đối chiếu với thời điểm deploy, thay đổi config, scheduled task và traffic tăng đột biến. Cũng cần đưa cảnh báo của database, Redis, message queue và external interface vào cùng phạm vi. Dựa theo timeline để xác nhận interface và user nào bị ảnh hưởng, bất thường bắt đầu từ khi nào và metric nào thay đổi sớm nhất.

Response time trung bình bình thường cũng không có nghĩa mọi request đều bình thường. Một số ít request cực chậm có thể nằm đúng trên các business flow quan trọng như login, đặt hàng hoặc thanh toán; khi đó P95/P99, timeout rate và business success rate có giá trị tham khảo hơn.

## Cân bằng việc giảm thiểu ảnh hưởng và lưu bằng chứng ra sao?

Khi sự cố vẫn đang lan rộng, trước tiên hãy giảm thiểu ảnh hưởng theo phương án ứng phó. Các biện pháp thường gặp gồm rollback version gần nhất, rút instance bất thường, tắt scheduled task có vấn đề, rate limiting, circuit breaker, degrade tính năng không cốt lõi và scale out. Mỗi biện pháp đều có tác dụng phụ: scale out có thể tiếp tục làm database quá tải, retry sẽ khuếch đại traffic downstream, còn rollback cũng chưa chắc tương thích với data đã thay đổi.

Nếu điều kiện cho phép, hãy lưu lại các thông tin sau trước khi restart hoặc rollback:

- Ảnh chụp màn hình monitoring hoặc khoảng thời gian trước và sau khi cảnh báo xảy ra.
- Application log, access log, GC log và lịch sử thay đổi.
- Một đến ba thread stack cách nhau vài giây, thay vì chỉ giữ một bản.
- Các call chậm, call lỗi và thời gian của upstream/downstream trong Trace.
- CPU process, RSS, heap, số thread, file descriptor và trạng thái network connection.
- Database slow query, lock wait, long transaction, trạng thái connection pool, Redis slow command và MQ consumer backlog.

Ngay cả khi instance đã được rút traffic, cũng đừng lập tức cho rằng có thể an toàn thực hiện Heap Dump. Việc dump heap lớn cần đủ disk space và I/O; một số lệnh còn có thể gây pause kéo dài. Cách thận trọng hơn là cấu hình auto dump khi OOM trong startup parameters từ trước, đồng thời ghi file dump vào directory có dung lượng và permission phù hợp.

```bash
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/path/with/enough/space
```

## Làm thế nào nhanh chóng nhận định vấn đề nằm ở layer nào?

Khi tạm thời chưa có manh mối từ call chain, hãy chọn điểm bắt đầu điều tra theo hiện tượng. Các đối chiếu trong bảng không thể thay thế cho việc xác minh tiếp theo.

| Hiện tượng                                  | Ưu tiên kiểm tra                                                                                  |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| CPU cao, interface cũng chậm                | Hot thread, vòng lặp vô hạn, serialization, lock contention, GC thường xuyên                      |
| CPU không cao, Load cao                     | Disk I/O, uninterruptible sleep, network storage, CPU limit của container                         |
| RSS liên tục tăng, Java heap ổn định        | Direct memory, thread stack, Metaspace, JNI, native library và memory mapping                     |
| Old area vẫn liên tục tăng sau khi GC       | Object có lifecycle dài, cache không giới hạn, collection được giữ lại, listener hoặc ThreadLocal |
| Số thread và queue length tăng              | Downstream chậm, lock wait, xử lý task chậm, thread pool isolation chưa đủ                        |
| Thời gian chờ database connection pool tăng | SQL chậm, long transaction, connection leak, database không đủ capacity                           |
| Redis RT tăng                               | Slow command, big Key, hot Key, network chập chờn, connection pool wait                           |
| MQ Lag liên tục tăng                        | Consumer chậm, partition không đều, retry message bất thường, downstream không đủ capacity        |

Vấn đề cũng có thể lan truyền qua nhiều layer. Một SQL chậm giữ database connection, sau đó business thread block ở connection pool, thread pool queue bắt đầu dồn lại, upstream timeout rồi retry, cuối cùng thấy CPU, latency và error rate cùng tăng. Khi điều tra cần dựa theo timeline để xác nhận metric nào thay đổi đầu tiên.

## Điều tra CPU tăng vọt thế nào?

Trước tiên xác nhận CPU cao đến từ process nào, sau đó tìm thread sử dụng CPU trong process đó. Dưới đây là cách điều tra thường dùng trên Linux:

```bash
# Tìm Java process
jps -l

# Xem mức sử dụng CPU của từng thread trong process
top -H -p <pid>

# Cũng có thể xem CPU usage của thread trong một khoảng thời gian
pidstat -t -p <pid> 1
```

`top -H` hiển thị thread ID ở dạng thập phân, còn `nid` trong thread stack thường dùng dạng thập lục phân. Trước tiên hãy chuyển đổi, sau đó tìm thread tương ứng trong thread stack:

```bash
printf '%x\n' <tid>
jcmd <pid> Thread.print > /tmp/thread-dump.txt
```

Cần lấy liên tiếp vài bản thread stack. Một thread chỉ thực hiện JSON serialization trong một bản stack có thể chỉ vô tình được lấy mẫu đúng lúc; nhiều bản stack đều dừng ở cùng một đoạn code thì mới đáng tiếp tục kiểm tra. Các nguyên nhân thường gặp gồm:

- Vòng lặp hoặc đệ quy không thể thoát.
- Task tính toán như serialization của object lớn, regex backtracking và encryption/decryption.
- Cấp phát nhiều object gây GC thường xuyên.
- Nhiều thread tranh chấp cùng một lock, kèm theo context switch.
- Traffic tăng hoặc retry khuếch đại, CPU chỉ đang xử lý nhiều việc hơn một cách bình thường.

Khi CPU đã gần đạt 100%, đừng đồng thời khởi động nhiều task chẩn đoán có overhead cao. Trước tiên hãy rút bớt traffic hoặc chọn một instance bất thường để lấy bằng chứng, tránh để thao tác chẩn đoán tiếp tục chiếm tài nguyên.

## CPU không cao nhưng Load cao thì làm thế nào?

Linux Load Average thống kê các task đang chạy và đang ở trạng thái uninterruptible sleep. Khi disk, network storage hoặc một số kernel I/O wait nghiêm trọng, CPU usage có thể không cao nhưng Load vẫn liên tục tăng.

```bash
vmstat 1
iostat -xz 1
pidstat -d -p <pid> 1
```

Tập trung quan sát running queue, I/O wait và task bị block trong `vmstat`, cùng device utilization, wait time và queue trong `iostat`. `iostat`, `pidstat` thường do package sysstat cung cấp; cần xác nhận trước chúng có sẵn trong môi trường production hay không.

## Interface rất chậm nhưng CPU không cao thì điều tra thế nào?

Vấn đề này thường xảy ra trong lúc chờ: chờ database connection, chờ downstream response, chờ lock, chờ task trong thread pool hoặc chờ disk I/O.

Trước tiên hãy tìm đoạn tốn thời gian nhất từ Trace. Nếu không có Trace, có thể kết hợp request duration trong access log, database slow query, metric của client connection pool và thread stack để thu hẹp phạm vi. Các trạng thái thường gặp trong thread stack gồm:

- Nhiều thread dừng ở `getConnection()` của database connection pool; kiểm tra connection pool wait, active connection, SQL chậm và long transaction.
- Nhiều thread dừng ở bước đọc response của HTTP client; kiểm tra downstream latency, timeout config và số lần retry.
- Nhiều thread ở trạng thái `BLOCKED`; kiểm tra lock holder và thao tác chậm trong critical section.
- Business thread idle nhưng task queue dồn lại; kiểm tra consumer thread, cách submit task và trạng thái executor.

Tổng thời gian của một request chậm phải khớp với từng giai đoạn. Nếu Trace chỉ ghi nhận thời gian thực thi của business method, còn connection pool wait, thread pool queue và DNS/TLS time của client không được ghi nhận, sẽ xuất hiện một khoảng trống không giải thích được ở giữa; cần bổ sung monitoring hoặc instrumentation tương ứng.

## Memory tăng liên tục và OOM thì điều tra thế nào?

Trước tiên phân biệt phần tăng là Java heap, process RSS hay tổng memory của container. `-Xmx` chỉ giới hạn Java heap; process còn sử dụng Metaspace, Code Cache, thread stack, direct memory, cấu trúc dữ liệu của GC và memory của native library.

Trước tiên xem dữ liệu từ cả JVM và hệ thống:

```bash
jcmd <pid> GC.heap_info
jcmd <pid> VM.flags
jcmd <pid> VM.native_memory summary
```

`VM.native_memory` phụ thuộc vào Native Memory Tracking (NMT), cần bật khi JVM khởi động, ví dụ `-XX:NativeMemoryTracking=summary`. Nó tạo thêm overhead, không thể đợi đến khi sự cố xảy ra mới bật tạm thời.

NMT chủ yếu thống kê native memory do chính JVM/HotSpot quản lý, không thể bao phủ toàn bộ memory allocation của JNI hoặc native library bên thứ ba. Khi RSS liên tục tăng nhưng NMT không có thay đổi tương ứng, vẫn cần tiếp tục điều tra bằng công cụ phân tích OS và native memory.

Các thông tin OOM thường gặp tương ứng với những hướng khác nhau:

| Thông tin OOM                    | Hướng điều tra thường gặp                                                                    |
| -------------------------------- | -------------------------------------------------------------------------------------------- |
| `Java heap space`                | Object lớn, object bị giữ, collection không giới hạn, cache, một query trả về quá nhiều data |
| `GC overhead limit exceeded`     | Heap gần cạn, GC tốn nhiều thời gian nhưng thu hồi được rất ít                               |
| `Metaspace`                      | Dynamic class generation, class loader leak, giới hạn Metaspace quá nhỏ                      |
| `Direct buffer memory`           | NIO direct memory, Netty Buffer, release chậm hoặc giới hạn không hợp lý                     |
| `unable to create native thread` | Quá nhiều thread, process limit, container thiếu memory, stack của một thread quá lớn        |

Có thể dùng MAT, VisualVM và các tool khác để phân tích Heap Dump. Trước tiên xem các object chiếm dụng nhiều nhất, Dominator Tree, reference chain đến GC Roots và class loader đáng ngờ; đừng chỉ kết luận dựa trên số lượng object.

Khi cần dump thủ công, có thể sử dụng:

```bash
jcmd <pid> GC.heap_dump filename=/path/with/enough/space/heap.hprof
```

Lệnh này có thể gây ảnh hưởng đáng kể đến application. Trước khi thực thi cần đánh giá heap size, disk còn trống, I/O và rủi ro pause. Khi production instance vẫn đang nhận traffic, ưu tiên thao tác trên instance bất thường sau khi đã rút traffic. `jmap -dump:live` có thể trigger Full GC, không phù hợp để thực thi trực tiếp như một lệnh không có rủi ro.

## Làm thế nào để phân biệt memory leak với tăng trưởng bình thường?

Quan sát heap usage sau một lần Full GC. Nếu business traffic và quy mô dữ liệu ổn định, Old area vẫn tiếp tục tăng sau nhiều lần Full GC thì mới cần nghi ngờ mạnh việc object không được giải phóng. Cache warm-up, class loading và traffic tăng cũng khiến heap tăng dần sau startup; chúng không nhất thiết là leak.

Khi phân tích cũng cần xem vì sao object bị giữ. Một `Map` rất lớn chỉ cho thấy nó chiếm memory; tiếp tục lần theo reference chain để tìm cache không giới hạn, static collection, listener, ThreadLocal hoặc session lifecycle sai thì mới xác nhận được vị trí cần sửa.

## Young GC thường xuyên thì điều tra thế nào?

Young GC thường xuyên thường cho thấy tốc độ phân bổ object cao hoặc young generation có capacity thấp. Trước tiên xem allocation rate, lượng object sống sau mỗi lần thu hồi và pause time, sau đó đánh giá có cần sửa code hay điều chỉnh parameter hay không.

```bash
jstat -gcutil <pid> 1000 10
jcmd <pid> GC.heap_info
```

Các nguyên nhân thường gặp gồm một query tải nhiều object, interface trả về result quá lớn, log hoặc serialization tạo nhiều temporary object, batch processing không kiểm soát batch size và cấu hình young generation không phù hợp với load.

Điều chỉnh young generation chỉ thay đổi nhịp GC, không thể sửa query không giới hạn và code phân bổ nhiều object. Trước tiên hãy xác nhận object được tạo ra từ đâu thông qua GC log, JFR, allocation sampling hoặc load test. Sau khi đổi parameter, cần so sánh throughput, P95/P99, số lần GC và tổng pause time trong cùng traffic và điều kiện dữ liệu.

## Full GC thường xuyên thì điều tra thế nào?

Trước tiên xác nhận nguyên nhân trigger, usage trước và sau khi thu hồi, cùng pause time từ GC log. Các field log và nguyên nhân trigger của các collector khác nhau không hoàn toàn giống nhau; không thể quy mọi Full GC về “old generation đầy”.

Khi điều tra, tập trung xem:

- Old area có giảm rõ rệt sau khi thu hồi hay không.
- Big object và object được promote có quá nhiều hay không.
- Metaspace có gần chạm giới hạn hay không.
- Có `System.gc()` tường minh hoặc diagnostic tool trigger hay không.
- Heap size, memory limit của container và collector config có phù hợp hay không.
- Trước GC có xuất hiện traffic tăng, batch task hoặc cache refresh hay không.

Nếu mức sử dụng sau khi thu hồi vẫn rất cao, tiếp tục phân tích object bị giữ; nếu hiệu quả thu hồi bình thường nhưng nhanh chóng đầy lại, tập trung kiểm tra allocation rate, promotion rate và heap capacity. Trước khi sửa JVM parameter, hãy xác nhận behavior của application; nếu không, tăng heap chỉ trì hoãn sự cố tiếp theo và có thể làm tăng pause hoặc chi phí dump.

## Thread pool queue dồn lại thì điều tra thế nào?

Vấn đề thread pool không thể chỉ xem số active thread. Ít nhất cần monitoring đồng thời:

- `corePoolSize`, `maximumPoolSize` và số thread hiện tại.
- Số active thread, queue length và queue capacity.
- Task submission rate, completion rate, wait time và execution time.
- Số lần reject và reject policy cụ thể.

Queue liên tục tăng cho thấy tốc độ task được đưa vào cao hơn tốc độ hoàn thành. Nguyên nhân có thể là traffic đột biến, cũng có thể do chính task chậm đi, chẳng hạn database connection wait, downstream timeout hoặc lock contention. Tăng số thread trực tiếp sẽ tăng concurrent access đến database, Redis và external interface, có thể đẩy vấn đề xuống downstream.

Hãy điều tra theo thứ tự dưới đây:

1. Xác nhận business task nào sử dụng thread pool này, có dùng chung với task không liên quan hay không.
2. So sánh submission rate, completion rate, queue time và execution time.
3. Lấy thread stack của các task đang chạy, xác nhận chúng đang tính toán hay chờ.
4. Kiểm tra database, HTTP và Redis connection pool có đồng thời cạn kiệt hay không.
5. Dựa trên mức độ quan trọng của task để quyết định rate limiting, degrade, scale out, isolation hoặc discard.

`CallerRunsPolicy` khiến thread submit task tự thực thi task, có thể tạm thời làm chậm tốc độ submit, nhưng sau khi Web request thread bị task dài chiếm giữ, interface latency cũng sẽ tăng. Nó có phù hợp với thread pool hiện tại hay không cần được đánh giá dựa trên caller và loại task.

## Database connection pool cạn kiệt thì điều tra thế nào?

Khi connection pool cạn kiệt, trước tiên xem active connection, idle connection, thread đang chờ và thời gian lấy connection. Khi nhiều business thread cùng chờ connection, thường tiếp tục kiểm tra theo ba hướng:

- SQL thực thi chậm, connection không thể được trả về trong thời gian dài.
- Transaction scope quá lớn, bao gồm remote call, xử lý file hoặc lượng lớn tính toán.
- Code không đóng connection đúng ở mọi branch, gây connection leak.

Ở phía database, tiếp tục kiểm tra session hiện tại, slow query, lock wait, long transaction và instance resource. MySQL có thể kết hợp `SHOW PROCESSLIST`, slow query log, Performance Schema, `EXPLAIN` và thông tin transaction lock của InnoDB để định vị. Execution plan phụ thuộc vào SQL hiện tại, parameter và data distribution; việc dùng index trong test database không chứng minh production chắc chắn giống vậy.

Trước khi mở rộng connection pool cần xác nhận database còn chịu được nhiều concurrent hơn hay không. Số instance của application nhân với connection pool limit của mỗi instance mới là tổng connection mà database có thể phải đối mặt; sau khi scale out service, tổng số này cũng sẽ tăng theo.

Nội dung liên quan: [Phân tích execution plan của MySQL](../../database/mysql/mysql-query-execution-plan.md), [Tổng hợp tối ưu SQL](../../high-performance/sql-optimization.md).

## Redis chậm đi thì điều tra thế nào?

Khi application truy cập Redis chậm đi, trước tiên phân biệt client wait và server execution. Connection pool cạn kiệt, network chập chờn hoặc vấn đề DNS đều khiến một Redis call chậm đi, ngay cả khi command thực tế chạy rất nhanh.

Ở Redis server, tập trung xem:

- Command latency và slow log.
- CPU, memory, network traffic và số connection.
- Big Key, hot Key, xóa key hết hạn và persistence operation.
- Độ trễ replication master-slave, cluster state và failover.

`SLOWLOG GET` ghi nhận execution time của command trên Redis server, không bao gồm queue và network transmission. Khi tổng thời gian của application rất cao nhưng Slow Log bình thường, tiếp tục kiểm tra connection pool wait, network và client processing.

Khi điều tra big Key có thể dùng các tool như `redis-cli --bigkeys`, nhưng chúng cần scan key space và sẽ tạo thêm load. Trong production nên chọn thời điểm ít traffic, replica hoặc sampling có kiểm soát, đồng thời xác nhận trước ảnh hưởng của command với version và cluster hiện tại. Đừng trực tiếp thực thi `KEYS *` trên production.

Hot Key cần được xác nhận bằng cách kết hợp tần suất truy cập và node load. Chỉ xem size của Key sẽ không tìm được một Key nhỏ có số lần đọc đặc biệt cao; có thể dùng proxy layer, client statistics, monitoring platform hoặc khả năng phân tích hot Key có kiểm soát để lấy bằng chứng.

Nội dung liên quan: [Các câu hỏi phỏng vấn Redis thường gặp](../../database/redis/redis-questions-02.md).

## Message queue bị backlog thì điều tra thế nào?

Trước tiên so sánh production rate và consumption rate, đồng thời xác nhận backlog tập trung ở Topic, queue hoặc partition nào. Sau đó kiểm tra consumer instance còn sống hay không, consumer thread có bị block hay không, processing time của từng message có thay đổi hay không, cùng với việc downstream database và interface có chậm đi hay không.

Các nguyên nhân thường gặp gồm:

- Traffic tăng, production rate vượt capacity thiết kế.
- Một loại message xử lý chậm, làm nghẽn toàn bộ partition hoặc queue.
- Message bất thường liên tục retry, tạo retry storm.
- Partition được phân bổ không đều hoặc consumer thường xuyên Rebalance.
- Commit offset thất bại sau khi consume thành công, gây consume trùng lặp.
- Downstream database, Redis hoặc external interface trở thành bottleneck.

Việc tăng consumer có hiệu quả hay không phụ thuộc vào message queue model. Trong cùng một consumer group của Kafka, parallelism hiệu dụng bị giới hạn bởi số partition; tiếp tục tăng consumer nhưng không có partition để phân bổ sẽ không tăng throughput. Ngay cả khi tăng partition và consumer, cũng cần xác nhận downstream capacity để tránh sau khi consumption khôi phục, toàn bộ traffic backlog dồn xuống database cùng lúc.

Trong thời gian xử lý backlog cũng cần bảo đảm idempotent, theo dõi consumer error và dead letter. Khi cần bỏ qua message bất thường, phải giữ lại message content và nguyên nhân thất bại để bù trừ sau đó, thay vì vứt bỏ trực tiếp.

Nội dung liên quan: [Các vấn đề thường gặp về message queue](../../high-performance/message-queue/message-queue.md).

## Downstream interface bị timeout thì điều tra thế nào?

Trước tiên chia thời gian của một call thành: connection pool wait, DNS, TCP connection establishment, TLS handshake, request sending, server processing và response reading. Các phase khác nhau cần timeout config khác nhau; chỉ đặt một total timeout sẽ khó xác định thời gian đã tiêu tốn ở đâu.

Khi kiểm tra caller, tập trung xem:

- Connection timeout, read timeout và total budget của call chain.
- Active connection, waiting queue và connection reuse của HTTP connection pool.
- Số lần retry, backoff strategy và những error nào trigger retry.
- Circuit breaker state, kết quả degrade và success rate của request đầu tiên.

Khi downstream đã chậm đi, retry ở nhiều layer sẽ nhanh chóng khuếch đại traffic. Gateway, business service và SDK mỗi nơi retry ba lần thì trong trường hợp xấu nhất có thể tạo request volume vượt xa một lần call. Trước tiên hãy giới hạn số tầng retry và total budget, sau đó dựa vào interface có idempotent hay không để quyết định có thể retry hay không.

## Dùng một interface chậm để minh họa toàn bộ quá trình điều tra

Giả sử P99 của order query interface tăng sau một lần deploy version, error rate cũng bắt đầu tăng; hãy điều tra theo thứ tự dưới đây. Ví dụ này nhằm minh họa phương pháp, không đại diện cho một sự cố production thực tế.

1. Bắt đầu từ monitoring để xác nhận vấn đề bắt đầu sau deploy, tập trung ở instance version mới và order query interface.
2. Tạm dừng các lần deploy tiếp theo, rút một phần instance bất thường; error rate giảm, trước tiên kiểm soát phạm vi ảnh hưởng.
3. So sánh Trace của instance mới và cũ, phát hiện phần lớn thời gian nằm ở việc lấy database connection và thực thi query.
4. Monitoring connection pool cho thấy số thread chờ tăng, database xuất hiện một SQL giống nhau trong slow query.
5. Dùng điều kiện query tương ứng với parameter production để xem execution plan, phát hiện điều kiện filter mới thêm khiến composite index cũ không còn lọc hiệu quả, đồng thời phát sinh thêm sort.
6. Sửa query và index trong môi trường test gần với data distribution của production, so sánh số row được scan, P95/P99, CPU và connection usage.
7. Sau khi thay đổi qua giai đoạn canary, tiếp tục quan sát slow query, connection pool wait và business error rate, sau đó từng bước khôi phục traffic.
8. Khi tổng kết, bổ sung performance test cho query scenario tương ứng và các mục monitoring sau deploy.

Trong scenario giả định này, thread pool và connection pool backlog xuất hiện sau SQL chậm; execution plan và slow query record đều chỉ ra SQL là root cause. Nếu ở bước 4 phát hiện SQL chạy nhanh, thời gian chủ yếu nằm ngoài connection pool, cần quay lại các giả thuyết khác để tiếp tục xác minh.

## Làm thế nào để verify sau khi sửa?

Sau khi sửa code hoặc config, vẫn cần xác nhận metric ở phía user đã khôi phục, các biện pháp tạm thời có thể gỡ bỏ và không phát sinh vấn đề data mới.

Khi verify, so sánh cùng một nhóm metric trước, trong và sau sự cố: request volume, P95/P99, error rate, business success rate, CPU, memory, GC, thread pool, connection pool và dependency latency. Khi liên quan đến consume trùng lặp, inventory, order và trạng thái tài chính, cũng cần đối soát data.

Thay đổi performance cần được so sánh trong điều kiện gần tương đương về data volume, traffic model, machine config và cache state. Chỉ chạy load test một lần trên một interface không thể đại diện cho việc toàn bộ business flow đã khôi phục. Cụ thể có thể tham khảo: [Nhập môn performance test và stress test](../../high-availability/performance-test.md).

## Khi tổng kết sự cố cần ghi lại những gì?

Tổng kết cần giữ lại các tài liệu có thể dùng trực tiếp trong sự cố lần sau. Tài liệu ít nhất phải lưu:

- Timeline sự cố, phạm vi ảnh hưởng và biểu hiện phía user.
- Nguyên nhân trực tiếp, điều kiện trigger và lý do sự cố lan rộng.
- Bằng chứng được sử dụng trong quá trình giảm thiểu ảnh hưởng, định vị và sửa lỗi.
- Monitoring, phương án ứng phó hoặc test nào đã không phát huy tác dụng.
- Hạng mục cải tiến tiếp theo, người phụ trách và thời hạn hoàn thành.

“Tăng cường kiểm tra code” rất khó xác minh đã hoàn thành hay chưa. Hạng mục cải tiến cần được cụ thể hóa thành cơ chế, chẳng hạn thêm cảnh báo cho connection pool wait time, giới hạn số row của batch query, bổ sung idempotent test cho message consumption và thêm kiểm tra so sánh interface quan trọng vào deploy workflow.

## Trả lời thế nào về điều tra sự cố production trong phỏng vấn?

Khi interviewer hỏi “xử lý CPU tăng vọt trên production thế nào”, có thể trả lời thứ tự tổng quát trước, sau đó kết hợp với case mình từng làm:

> Trước tiên tôi sẽ xác nhận phạm vi ảnh hưởng, traffic và thay đổi gần đây, đánh giá có cần rút instance, rate limiting hoặc rollback hay không. Nếu điều kiện cho phép, tôi sẽ lưu lại monitoring, log và nhiều bản thread stack. Sau khi xác nhận CPU thực sự đến từ Java process, dùng `top -H` hoặc `pidstat -t` để tìm hot thread, chuyển thread ID sang thập lục phân rồi đối chiếu với `nid` trong thread stack. Sampling liên tục để xác nhận thread trong thời gian dài dừng ở đoạn code nào, sau đó kết hợp GC, Trace và traffic để đánh giá đó là computational hotspot, vòng lặp vô hạn, lock contention hay GC thường xuyên. Sau khi sửa, verify trong cùng scenario, đồng thời bổ sung monitoring và regression test.

Nếu chưa từng xử lý sự cố production thực tế, có thể nói rõ mình đã diễn tập scenario nào trong test environment và đã tham khảo case nào. Đừng nói những bài viết mình từng đọc thành sự cố do chính mình phụ trách.

## Tra nhanh các tool thường dùng

| Tool                      | Công dụng thường gặp                                            | Lưu ý khi sử dụng                                                         |
| ------------------------- | --------------------------------------------------------------- | ------------------------------------------------------------------------- |
| `top`, `pidstat`          | CPU, I/O của process và thread                                  | Trước tiên xác nhận phạm vi quan sát của container và host                |
| `vmstat`, `iostat`        | Running queue, memory và disk của hệ thống                      | Cần kết hợp xu hướng trong một khoảng thời gian                           |
| `jcmd`                    | Thread stack, heap info, JFR, class histogram                   | Một số command có overhead cao, cần đánh giá trước khi thực thi           |
| `jstat`                   | GC và thay đổi của các memory area                              | Sampling liên tục hữu ích hơn dữ liệu tại một thời điểm                   |
| JFR, JMC                  | Phân tích event về CPU, allocation, lock, I/O                   | Cấu hình thu thập cần tính đến overhead trên production                   |
| MAT, VisualVM             | Phân tích Heap Dump                                             | Tập trung xem reference chain và GC Roots                                 |
| Arthas                    | Thời gian thực thi method, thread, class và runtime diagnostics | Cần giới hạn phạm vi khi thực hiện Trace/Watch trên method có traffic cao |
| Trace/monitoring platform | Request chain, error rate, latency và dependency                | Chú ý sampling rate, data masking và time synchronization                 |

## Đọc thêm

- [Tổng hợp tool monitoring và xử lý sự cố JDK](./jdk-monitoring-and-troubleshooting-tools.md)
- [Tổng hợp các tham số JVM quan trọng nhất](./jvm-parameters-intro.md)
- [Giải thích chi tiết JVM garbage collection](./jvm-garbage-collection.md)
- [Giải thích chi tiết Java thread pool](../concurrent/java-thread-pool-summary.md)
- [Giải thích chi tiết deadlock](../../cs-basics/operating-system/dead-lock.md)
- [Hướng dẫn xử lý sự cố Java của Oracle: memory leak](https://docs.oracle.com/en/java/javase/17/troubleshoot/troubleshooting-memory-leaks.html)
- [Điều tra vấn đề latency của Redis](https://redis.io/docs/latest/operate/oss_and_stack/management/optimization/latency/)
- [Redis Slow Log](https://redis.io/docs/latest/commands/slowlog/)
- [Phân tích một sự cố OOM trên production](https://juejin.cn/post/7205141492264976445)
- [Phân tích và xử lý 9 vấn đề CMS GC thường gặp trong Java](https://tech.meituan.com/2020/11/12/java-9-cms-gc.html)

<!-- @include: @article-footer.snippet.md -->
