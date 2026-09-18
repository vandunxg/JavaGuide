---
title: Giải thích chi tiết về optimistic lock và pessimistic lock
description: "So sánh chuyên sâu optimistic lock và pessimistic lock: giải thích chi tiết cách triển khai pessimistic lock bằng synchronized/ReentrantLock, cơ chế optimistic lock bằng CAS/số phiên bản, phân tích trường hợp sử dụng, so sánh hiệu năng và đề xuất lựa chọn."
category: Java
tag:
  - Java Concurrency
head:
  - - meta
    - name: keywords
      content: optimistic lock,pessimistic lock,synchronized,ReentrantLock,CAS,cơ chế số phiên bản,kiểm soát concurrency,tối ưu lock
---

Nếu đối chiếu pessimistic lock (Pessimistic Lock) và optimistic lock (Optimistic Lock) với đời sống thực, pessimistic lock giống một người khá bi quan (cũng có thể nói là luôn lo xa), luôn giả định tình huống xấu nhất để tránh vấn đề xảy ra. Optimistic lock giống một người khá lạc quan, luôn giả định tình huống tốt nhất và nhanh chóng giải quyết vấn đề trước khi nó xảy ra.

## Pessimistic lock là gì?

Pessimistic lock luôn giả định tình huống xấu nhất, cho rằng mỗi lần shared resource được truy cập sẽ xảy ra vấn đề (ví dụ shared data bị sửa đổi), nên mỗi lần lấy resource đều lock. Khi đó, thread khác muốn lấy resource này sẽ bị blocking cho đến khi lock được owner trước đó release. Nói cách khác, **shared resource mỗi lần chỉ cho một thread sử dụng, các thread khác bị blocking; sau khi dùng xong mới chuyển resource cho thread khác**.

Các exclusive lock như `synchronized` và `ReentrantLock` trong Java là hiện thực của tư tưởng pessimistic lock.

```java
public void performSynchronisedTask() {
    synchronized (this) {
        // Các thao tác cần đồng bộ
    }
}

private Lock lock = new ReentrantLock();
lock.lock();
try {
   // Các thao tác cần đồng bộ
} finally {
    lock.unlock();
}
```

Trong trường hợp high concurrency, lock contention gay gắt sẽ khiến thread bị blocking. Một lượng lớn thread bị blocking sẽ dẫn đến context switch của hệ thống, làm tăng chi phí hiệu năng của hệ thống. Ngoài ra, pessimistic lock cũng có thể phát sinh deadlock (khi thứ tự thread lấy lock không phù hợp), ảnh hưởng đến việc thực thi bình thường của code.

## Optimistic lock là gì?

Optimistic lock luôn giả định tình huống tốt nhất, cho rằng mỗi lần shared resource được truy cập sẽ không xảy ra vấn đề. Thread có thể liên tục thực thi, không cần lock và cũng không cần chờ; chỉ khi submit thay đổi mới xác minh resource tương ứng (tức data) có bị thread khác sửa đổi hay chưa (cách cụ thể có thể dùng cơ chế số phiên bản hoặc thuật toán CAS).

Các atomic variable class trong package `java.util.concurrent.atomic` của Java (ví dụ `AtomicInteger`, `LongAdder`) sử dụng một cách triển khai optimistic lock là **CAS**.
![Tổng quan các atomic class của JUC](https://oss.javaguide.cn/github/javaguide/java/JUC%E5%8E%9F%E5%AD%90%E7%B1%BB%E6%A6%82%E8%A7%88-20230814005211968.png)

```java
// Trong trường hợp high concurrency, LongAdder có hiệu năng tốt hơn AtomicInteger và AtomicLong
// Đổi lại, nó tiêu tốn nhiều memory space hơn (đổi space lấy time)
LongAdder sum = new LongAdder();
sum.increment();
```

Trong trường hợp high concurrency, so với pessimistic lock, optimistic lock không có thread bị blocking do lock contention và cũng không gặp deadlock, nên thường có hiệu năng tốt hơn. Tuy nhiên, nếu conflict xảy ra thường xuyên (tỷ lệ write rất cao), thao tác sẽ thường xuyên thất bại và retry, điều này cũng ảnh hưởng nghiêm trọng đến hiệu năng và khiến CPU tăng vọt.

`LongAdder` phân tán update vào nhiều unit nội bộ khi contention gay gắt, từ đó giảm xác suất mọi thread tranh chấp cùng một value, nhưng update bên trong vẫn có thể dùng CAS và retry, không loại bỏ hoàn toàn contention. Ngoài ra, `sum()` cũng không trả về snapshot nhất quán nguyên tử với các update đồng thời.

Về mặt lý thuyết:

- Pessimistic lock thường được dùng nhiều trong trường hợp write nhiều (nhiều write, contention gay gắt), nhờ đó tránh việc thất bại và retry thường xuyên ảnh hưởng đến hiệu năng. Tuy nhiên, với các semantic đặc thù chỉ cần cộng dồn, cũng có thể cân nhắc giải pháp như `LongAdder` để giảm xác suất retry bằng cách phân tán contention; vẫn cần cân nhắc theo tình huống thực tế.
- Optimistic lock thường được dùng nhiều trong trường hợp write ít (nhiều read, contention ít), nhờ đó tránh việc lock thường xuyên ảnh hưởng đến hiệu năng. Tuy nhiên, optimistic lock chủ yếu nhắm đến một shared variable (tham khảo các atomic variable class trong package `java.util.concurrent.atomic`).

## Triển khai optimistic lock như thế nào?

Optimistic lock thường được triển khai bằng cơ chế số phiên bản hoặc thuật toán CAS. CAS được dùng nhiều hơn, vì vậy cần đặc biệt lưu ý.

### Cơ chế số phiên bản

Thông thường, thêm một field số phiên bản data `version` vào data table để biểu thị số lần data được sửa đổi. Khi data được sửa đổi, giá trị `version` tăng một. Khi thread A muốn update data value, thread này đồng thời đọc giá trị data và `version`. Khi submit update, chỉ update nếu giá trị version vừa đọc bằng với giá trị `version` hiện tại trong database; nếu không thì retry thao tác update cho đến khi thành công.

**Ví dụ đơn giản**: giả sử data table thông tin account trong database có một field version, giá trị hiện tại là 1; còn field số dư account (`balance`) hiện là \$100.

1. Operator A đọc data tại thời điểm này (`version`=1), rồi trừ $50 từ số dư account ($100-$50).
2. Trong lúc operator A thao tác, operator B cũng đọc thông tin user này (`version`=1), rồi trừ $20 từ số dư account ($100-$20).
3. Operator A hoàn tất việc sửa đổi, gửi số phiên bản data (`version`=1) cùng số dư sau khi trừ (`balance`=\$50) lên database để update. Vì version của data submit bằng version hiện tại của record trong database, data được update và `version` của record trong database được cập nhật thành 2.
4. Operator B hoàn tất thao tác và cũng cố submit version (`version`=1) lên database với data (`balance`=\$80). Tuy nhiên, khi đối chiếu version của record trong database, operator B phát hiện version data submit là 1 còn version hiện tại của record trong database là 2, không thỏa mãn chiến lược optimistic lock “version submit phải bằng version hiện tại mới được thực hiện update”, nên submit của operator B bị từ chối.

Nhờ vậy, tránh được khả năng kết quả sửa đổi dựa trên data cũ với `version`=1 của operator B ghi đè kết quả thao tác của operator A.

### Thuật toán CAS

CAS là viết tắt của **Compare And Swap (so sánh và hoán đổi)**, được dùng để triển khai optimistic lock và được ứng dụng rộng rãi trong nhiều framework. Tư tưởng của CAS rất đơn giản: so sánh một expected value với value của variable cần update, chỉ update khi hai value bằng nhau.

CAS là một atomic operation, bên dưới phụ thuộc vào một atomic instruction của CPU.

> **Atomic operation** là operation nhỏ nhất không thể tách rời, nghĩa là một khi operation bắt đầu thì không thể bị ngắt cho đến khi hoàn tất.

CAS liên quan đến ba operand:

- **V**: value của variable cần update (Var)
- **E**: expected value (Expected)
- **N**: new value dự định ghi vào (New)

Khi và chỉ khi value của V bằng E, CAS mới dùng atomic operation để update value của V bằng new value N. Nếu không bằng, nghĩa là thread khác đã update V, thread hiện tại từ bỏ update.

**Ví dụ đơn giản**: thread A muốn sửa value của variable i thành 6, value ban đầu của i là 1 (V = 1, E=1, N=6, giả sử không tồn tại vấn đề ABA).

1. So sánh i với 1; nếu bằng nhau thì nghĩa là chưa bị thread khác sửa đổi và có thể set thành 6.
2. So sánh i với 1; nếu không bằng thì nghĩa là đã bị thread khác sửa đổi, thread hiện tại từ bỏ update và thao tác CAS thất bại.

Khi nhiều thread đồng thời dùng thao tác CAS trên một variable, chỉ một thread thắng và update thành công; các thread còn lại đều thất bại. Tuy nhiên, thread thất bại không bị suspend mà chỉ được thông báo thất bại và được phép thử lại; đương nhiên thread thất bại cũng có thể từ bỏ thao tác.

Để tìm hiểu thêm về CAS, bạn có thể đọc bài viết này do độc giả viết: [Giải thích chi tiết về CAS](./cas.md), trong đó trình bày chi tiết cách triển khai CAS trong Java và một số vấn đề của CAS.

## Tổng kết

Bài viết đã giải thích chi tiết khái niệm optimistic lock và pessimistic lock cùng các cách triển khai phổ biến của optimistic lock:

- Pessimistic lock dựa trên giả định bi quan rằng shared resource sẽ conflict trong mỗi lần truy cập, nên mỗi thao tác đều lock. Cơ chế lock này khiến các thread khác bị blocking cho đến khi lock được release. `synchronized` và `ReentrantLock` trong Java là các cách triển khai tiêu biểu của pessimistic lock. Dù pessimistic lock có thể ngăn data race hiệu quả, trong trường hợp high concurrency nó sẽ khiến thread bị blocking, context switch diễn ra thường xuyên, từ đó ảnh hưởng đến hiệu năng hệ thống và cũng có thể gây deadlock.
- Optimistic lock dựa trên giả định lạc quan rằng shared resource sẽ không conflict trong mỗi lần truy cập, nên không cần lock mà chỉ cần xác minh data có bị thread khác sửa đổi khi submit thay đổi hay không. Các class như `AtomicInteger` và `LongAdder` trong Java triển khai optimistic lock bằng thuật toán CAS (Compare-And-Swap). Optimistic lock tránh được thread bị blocking và deadlock, có hiệu năng tốt trong trường hợp nhiều read, ít write. Nhưng khi thao tác write thường xuyên, nó có thể dẫn đến nhiều retry và thất bại, từ đó ảnh hưởng đến hiệu năng.
- Optimistic lock chủ yếu được triển khai bằng cơ chế số phiên bản hoặc thuật toán CAS. Cơ chế số phiên bản đảm bảo data nhất quán bằng cách so sánh version, còn CAS thực hiện atomic operation bằng CPU instruction, trực tiếp so sánh và hoán đổi value của variable.

Pessimistic lock và optimistic lock đều có ưu, nhược điểm riêng và phù hợp với các trường hợp sử dụng khác nhau. Trong quá trình phát triển thực tế, lựa chọn cơ chế lock phù hợp có thể nâng cao hiệu năng concurrency và tính ổn định của hệ thống một cách hiệu quả.

## Tham khảo

- 《78 bài giảng cốt lõi về lập trình concurrent trong Java》
- Pessimistic lock, optimistic lock, reentrant lock, spin lock, biased lock, lightweight/heavyweight lock, read-write lock và các loại lock cùng cách triển khai trong Java dễ hiểu!: <https://zhuanlan.zhihu.com/p/71156910>

<!-- @include: @article-footer.snippet.md -->
