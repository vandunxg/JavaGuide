---
title: "Tổng hợp câu hỏi phỏng vấn JVM mới nhất năm 2026: vùng nhớ, class loading, garbage collection và điều tra sự cố production"
category: Java
description: "Tổng hợp câu hỏi phỏng vấn JVM mới nhất năm 2026, bao quát runtime memory area, object creation, class file và class loading, thuật toán và collector của garbage collection, tham số JVM, công cụ chẩn đoán JDK, OOM, Full GC liên tục và CPU tăng vọt cùng các trọng tâm thường gặp khác."
tag:
  - Java
  - JVM
  - Câu hỏi phỏng vấn
head:
  - - meta
    - name: keywords
      content: Câu hỏi phỏng vấn JVM,Vùng nhớ Java,Tạo object,class loading,parent delegation,garbage collection,Câu hỏi phỏng vấn GC,G1,ZGC,Tham số JVM,OOM,Full GC,CPU tăng vọt,Tối ưu JVM
---

Phỏng vấn JVM hiếm khi chỉ dừng ở câu hỏi “heap và stack khác nhau thế nào”. Sau khi trả lời về vùng nhớ, interviewer thường hỏi tiếp object được phân bổ thế nào, object nào có thể được thu hồi, vì sao một lần GC lại gây pause, cũng như cách điều tra khi production xuất hiện OOM, Full GC liên tục hoặc CPU tăng vọt.

Bài viết này là cổng ôn tập phỏng vấn cho chuyên đề JVM của JavaGuide. Các câu hỏi được chia thành năm phần: memory và object, class file và class loading, garbage collection, tham số và công cụ chẩn đoán, điều tra sự cố production. Câu trả lời đầy đủ cho từng câu hỏi nằm trong bài viết chuyên đề tương ứng.

Nếu thời gian khá gấp, bạn có thể đọc trước [Tổng hợp câu hỏi phỏng vấn JVM thường gặp](https://interview.javaguide.cn/java/java-jvm.html), đánh dấu những câu chưa trả lời đầy đủ, rồi quay lại bài viết này và các bài chuyên đề để bổ sung chi tiết.

## Khi ôn tập JVM, trước hết cần nắm những câu hỏi nào?

| Module                      | Nội dung cần trình bày rõ                                                              | Hướng hỏi tiếp thường gặp                                                          |
| --------------------------- | -------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| Memory và object            | Runtime data area được phân chia thế nào, object được tạo, lưu trữ và truy cập ra sao  | heap, stack, method area, direct memory, object header, TLAB, OOM                  |
| Class file và class loading | File `.class` đi vào JVM thế nào, class hoàn tất initialization vào thời điểm nào      | constant pool, quá trình loading, class loader, parent delegation, class isolation |
| Garbage collection          | Xác định object còn sống thế nào, memory được thu hồi ra sao, pause phát sinh từ đâu   | GC Roots, loại reference, thuật toán collection, CMS, G1, ZGC, Full GC             |
| Tham số và công cụ          | Chọn tham số theo JDK, collector và hiện tượng runtime thế nào                         | `-Xms`, `-Xmx`, GC log, Heap Dump, jstat, jstack, JFR                              |
| Điều tra production         | Sau khi nhận cảnh báo, giữ lại bằng chứng, thu hẹp phạm vi và xác minh bản sửa thế nào | CPU, Load, OOM, memory leak, GC liên tục, thread bị block                          |

Hầu hết object instance được phân bổ trên heap; thread đang hoạt động, static field và các vị trí khác có thể giữ reference đến object. GC bắt đầu từ GC Roots để xác định object có reachable hay không, sau đó collector cụ thể thực hiện việc thu hồi. Khi ứng dụng tăng memory hoặc bị pause, còn phải kết hợp GC log, thread stack, heap dump và system metrics để phán đoán nguyên nhân, không thể thấy Full GC rồi lập tức tăng kích thước heap.

## JVM memory area và object

Trả lời về runtime data area chỉ nêu tên là chưa đủ. Mỗi area được những thread nào chia sẻ, lưu trữ gì, vòng đời ra sao và có thể ném exception nào đều có thể trở thành câu hỏi tiếp theo. Method area là area logic do đặc tả JVM định nghĩa; permanent generation và metaspace là các implementation của HotSpot trong những version khác nhau. Khi trả lời, không nên gộp chúng thành cùng một khái niệm.

Nội dung liên quan: [Giải thích chi tiết JVM memory area](./memory-area.md)

Câu hỏi phỏng vấn thường gặp:

- Runtime data area của JVM gồm những phần nào? Những area nào là thread-private?
- Program counter, virtual machine stack và native method stack lần lượt có tác dụng gì?
- Một Java method thay đổi thế nào trong stack frame từ lúc được gọi đến khi trả về?
- `StackOverflowError` và `OutOfMemoryError` lần lượt có thể phát sinh thế nào?
- Java heap chủ yếu lưu trữ gì? Có phải mọi object đều chắc chắn được phân bổ trên heap không?
- Method area, permanent generation và metaspace có quan hệ gì? Vì sao permanent generation bị loại bỏ?
- Runtime constant pool và string constant pool khác nhau thế nào? Vị trí của chúng đã thay đổi ra sao?
- Direct memory có thuộc runtime data area của JVM không? Vì sao nó cũng có thể gây OOM?
- HotSpot phải trải qua những bước nào để tạo một object?
- Chọn pointer bump hay free list thế nào? Khi phân bổ object đồng thời, CAS và TLAB lần lượt giải quyết vấn đề gì?
- Object header, instance data và alignment padding lần lượt lưu trữ gì?
- Handle access và direct pointer access đến object khác nhau thế nào? HotSpot chủ yếu sử dụng cách nào?

Khi gặp câu hỏi về OOM, trước hết cần xác nhận phần bị cạn kiệt là Java heap, metaspace, direct memory hay native memory cần cho việc tạo thread. Chỉ trả lời “tăng `-Xmx`” sẽ bỏ sót các nguyên nhân như collection không giới hạn, class loader leak, direct memory không được giải phóng và số lượng thread mất kiểm soát.

## Class file và class loading

Câu hỏi về class loading thường bắt đầu từ “một class được loading thế nào”, sau đó hỏi tiếp về thời điểm initialization, parent delegation và isolation giữa các class loader khác nhau. Phần này vừa có lifecycle trong đặc tả JVM, vừa có implementation cụ thể của HotSpot và JDK class loader. Khi mô tả khác biệt giữa các version, cần nói rõ JDK đang sử dụng.

Nội dung liên quan:

- [Giải thích chi tiết cấu trúc class file](./class-file-structure.md)
- [Giải thích chi tiết quá trình class loading](./class-loading-process.md)
- [Giải thích chi tiết class loader](./classloader.md)

Câu hỏi phỏng vấn thường gặp:

- Class file gồm những phần nào? Magic number, version number và constant pool lần lượt có tác dụng gì?
- Bytecode instruction của method được lưu ở vị trí nào trong class file?
- Một class phải trải qua những phase nào từ loading đến unloading?
- Loading, verification, preparation, resolution và initialization lần lượt thực hiện gì?
- Việc gán giá trị cho static variable trong preparation phase khác gì với initialization phase?
- Những trường hợp nào trigger initialization của class? Truy cập compile-time constant có trigger không?
- `<clinit>` và constructor `<init>` khác nhau thế nào?
- Bootstrap class loader, platform class loader và application class loader lần lượt loading những class nào? Extension class loader của JDK 8 khác gì?
- Parent delegation model là gì? Nó tránh việc core class bị loading hoặc thay thế lặp lại thế nào?
- Parent delegation được thực thi ra sao? Khi parent loader không thể hoàn tất việc loading thì chuyện gì xảy ra?
- Những tình huống nào có thể phá vỡ parent delegation? Thread context class loader giải quyết vấn đề gì?
- Hai class có cùng fully qualified name thì có chắc là cùng một class không?
- Custom class loader thường cần override `findClass()` hay `loadClass()`?

Trả lời về parent delegation chỉ như một chain tìm kiếm cố định hướng lên trên là chưa đầy đủ. Identity của class được xác định đồng thời bởi fully qualified name và class loader đã loading nó. Các tình huống như SPI, class isolation của application server và hot deploy sử dụng những cách loading khác nhau, mục đích cũng không hoàn toàn giống nhau.

## Garbage collection

Câu hỏi về garbage collection cần đi từ việc phân bổ object và xác định object còn sống đến collector cụ thể. Throughput, pause time và memory usage có sự đánh đổi; khi chọn collector còn phải cân nhắc JDK version, kích thước heap và yêu cầu latency của business, không thể chỉ so sánh tên thuật toán.

Nội dung liên quan: [Giải thích chi tiết JVM garbage collection](./jvm-garbage-collection.md)

Câu hỏi phỏng vấn thường gặp:

- Young generation và old generation thường được phân chia thế nào? Object thường được phân bổ và promote ra sao?
- Big object và object sống lâu sẽ đi vào old generation thế nào?
- Space allocation guarantee là gì?
- Vì sao reference counting khó giải quyết circular reference giữa các object?
- Reachability analysis xác định object còn sống thế nào? Những object nào có thể làm GC Roots?
- Strong reference, soft reference, weak reference và phantom reference khác nhau thế nào?
- Mark-sweep, copying và mark-compact algorithm có những ưu, nhược điểm gì?
- Vì sao generational collection cần chọn algorithm khác nhau cho young generation và old generation?
- Minor GC, Major GC và Full GC khác nhau thế nào? Vì sao cần kết hợp collector và log để hiểu những tên gọi này?
- Serial, Parallel, CMS, G1 và ZGC khác nhau thế nào về mục tiêu và trường hợp sử dụng?
- Vì sao CMS tạo ra memory fragmentation và floating garbage?
- G1 kiểm soát pause thế nào thông qua Region, dự đoán giá trị thu hồi và Mixed GC?
- ZGC giảm pause dài thế nào? Low-pause collector phải đánh đổi những gì?
- Những tình huống nào có thể trigger Full GC? Khi Full GC xảy ra liên tục, nên bắt đầu điều tra từ dữ liệu nào?

Event name, memory partition và parameter trong GC log thay đổi theo JDK và collector. Khi phỏng vấn, có thể nêu runtime environment của mình trước, sau đó giải thích nguyên nhân trigger một lần collection, phạm vi collection, phase pause và kết quả. Học thuộc kết luận cố định mà không xét version rất dễ dẫn đến mâu thuẫn khi bị hỏi tiếp.

## JVM parameter và công cụ chẩn đoán

Câu hỏi về parameter kiểm tra cơ sở để đưa ra cấu hình. Kích thước heap, tỷ lệ young generation, collector và parameter log cần được thảo luận cùng memory khi deploy, tốc độ phân bổ object, mục tiêu latency và JDK version. Đặc biệt với G1, thông thường không nên sao chép nguyên xi cấu hình cũ lấy kích thước young generation cố định làm trung tâm.

Nội dung liên quan:

- [Tổng hợp JVM parameter thường dùng](./jvm-parameters-intro.md)
- [Tổng hợp công cụ monitoring và troubleshooting của JDK](./jdk-monitoring-and-troubleshooting-tools.md)

Câu hỏi phỏng vấn thường gặp:

- `-Xms`, `-Xmx`, `-Xmn` và `-Xss` lần lượt điều khiển gì?
- Vì sao application phía server thường đặt `-Xms` và `-Xmx` bằng nhau?
- `-XX:MetaspaceSize` và `-XX:MaxMetaspaceSize` khác nhau thế nào?
- Chọn garbage collector cho application thế nào? Trước khi đổi collector cần thu thập dữ liệu gì?
- Cấu hình GC log trong JDK 8 và JDK 9 trở đi thế nào?
- Khi xảy ra OOM, làm thế nào để tự động tạo Heap Dump? Vì sao cần xác nhận storage space trước?
- `jps`, `jstat`, `jinfo`, `jmap`, `jstack` và `jcmd` lần lượt phù hợp để kiểm tra gì?
- Dùng `jstat` để theo dõi tần suất GC, capacity của từng area và tình trạng object promote thế nào?
- Khi thread bị block lâu, deadlock hoặc CPU tăng vọt, thread dump có thể cung cấp thông tin gì?
- Heap Dump và thread dump khác nhau thế nào? `kill -3` tạo ra loại nào?
- Khi phân tích Heap Dump bằng MAT, Dominator Tree và reference chain đến GC Roots lần lượt có tác dụng gì?
- JFR, JMC và VisualVM phù hợp để quan sát những runtime information nào? Khi collect online cần cân nhắc overhead gì?

Việc tạo Heap Dump, class histogram hoặc sampling tần suất cao đều có thể làm tăng load production; Dump file cũng có thể chứa business data. Trước khi thực thi diagnostic command, cần xác nhận phạm vi ảnh hưởng, disk space và vị trí lưu file. Trước khi restart khẩn cấp, ít nhất phải giữ lại thời điểm cảnh báo, các metric quan trọng, thread stack và GC information cần thiết.

## Điều tra sự cố JVM trong production

Điều tra production thường không có câu trả lời kiểu “một command là định vị được ngay”. Sau khi nhận cảnh báo, trước hết kiểm tra thời gian, phạm vi ảnh hưởng, thay đổi gần đây và system resource, rồi quyết định collect thread stack, GC log hay heap dump. Nếu service đã ảnh hưởng đến user, cần đồng thời sắp xếp việc giảm tác động và giữ bằng chứng; không thể vì muốn thu thập đầy đủ bằng chứng mà để sự cố tiếp tục lan rộng.

Nội dung liên quan: [Điều tra sự cố Java backend trong production](./jvm-in-action.md)

Câu hỏi phỏng vấn thường gặp:

- Khi CPU tăng vọt, làm thế nào đi từ process đến thread và code stack cụ thể?
- CPU không cao nhưng Load cao, nên kiểm tra những thread state và system metric nào?
- Interface phản hồi chậm nhưng CPU usage bình thường, phân biệt lock waiting, I/O, connection pool và latency của downstream thế nào?
- Khi memory liên tục tăng, làm thế nào xác định đó là business cache tăng, object được phân bổ quá nhanh hay memory leak?
- Sau khi Java process OOM, làm thế nào xác định vấn đề xảy ra ở heap, metaspace, direct memory hay native memory?
- Young GC rất thường xuyên nhưng mỗi lần thu hồi rất nhanh, cần tập trung kiểm tra gì?
- Khi Full GC xảy ra thường xuyên, quan sát old generation tăng, object promote và hiệu quả collection thế nào?
- Nên tạo Heap Dump vào thời điểm nào? Trực tiếp thực thi `jmap -dump` trong production có rủi ro gì?
- Queue của thread pool bị dồn lại có quan hệ gì với CPU usage và thread state?
- Sau khi sửa vấn đề JVM, nên so sánh những metric nào để xác nhận thay đổi có hiệu quả?
- Khi phỏng vấn, trình bày thế nào về một câu hỏi điều tra sự cố JVM mà mình chưa từng trực tiếp trải qua?

Kết quả điều tra cần hình thành được evidence chain. Ví dụ khi CPU tăng vọt, trước tiên tìm Java process và thread có CPU cao, chuyển thread ID sang format dùng trong thread dump, rồi kiểm tra nhiều thread stack có liên tục rơi vào cùng một đoạn code hay không. Với vấn đề memory, cần kết hợp GC log, object histogram, Heap Dump và native memory; chỉ xem heap usage của JVM thì không thể bao quát mọi trường hợp OOM.

## Sắp xếp việc ôn tập theo thời gian chuẩn bị

| Thời gian còn lại | Lịch trình đề xuất                                                                                                                                                                                                       | Mục tiêu ôn tập                                                                                |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------- |
| 1～2 ngày         | Đọc một lượt [Tổng hợp câu hỏi phỏng vấn JVM thường gặp](https://interview.javaguide.cn/java/java-jvm.html), ưu tiên bổ sung memory area, object creation, class loading, xác định object còn sống và collector phổ biến | Trả lời được câu hỏi nền tảng thường gặp, biết mỗi kết luận tương ứng với JDK và collector nào |
| 3～7 ngày         | Bổ sung JVM parameter, GC log và công cụ chẩn đoán JDK; lần lượt trình bày bằng lời quy trình điều tra CPU tăng vọt, OOM và Full GC liên tục                                                                             | Chọn được metric và tool từ hiện tượng cảnh báo, đồng thời nêu được rủi ro thao tác            |
| Hơn 1 tuần        | Đọc toàn bộ bài chuyên đề JVM, phân tích một lần GC log, thread dump hoặc Heap Dump trong local hay môi trường test; kết hợp project để ghi lại JDK, collector, cấu hình heap và runtime metric thực tế                  | Kết nối được nguyên lý, cấu hình, hiện tượng runtime và phương pháp xác minh                   |

Vị trí tuyển dụng dành cho người đã đi làm và vị trí trung, cao cấp thường tiếp tục hỏi về cơ sở chọn parameter và bằng chứng điều tra. Nếu chưa từng trực tiếp xử lý sự cố JVM production, có thể trả lời dựa trên quan sát ở môi trường test và tài liệu chuyên đề, nêu rõ mình sẽ thu thập gì trước và thu hẹp phạm vi thế nào; không nên biến case học tập thành sự cố production từng trải qua.

<!-- @include: @article-footer.snippet.md -->
