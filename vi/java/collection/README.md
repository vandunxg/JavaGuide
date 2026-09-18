---
title: "Chuyên đề Java Collections: List, Map, Queue, concurrent collections và phân tích source code"
description: Lộ trình học câu hỏi phỏng vấn và source code Java Collections, bao quát List, Set, Map, Queue, ArrayList, HashMap, ConcurrentHashMap, blocking queue và các vấn đề thường gặp khi sử dụng Collections.
category: Java
tag:
  - Java
  - Java Collections
  - Câu hỏi phỏng vấn Java
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Java Collections,câu hỏi phỏng vấn Java Collections,ArrayList,LinkedList,HashMap,ConcurrentHashMap,CopyOnWriteArrayList,ArrayBlockingQueue,PriorityQueue,DelayQueue,source code Collections
---

Java Collections là một trong những thư viện cơ bản được sử dụng thường xuyên nhất trong phát triển nghiệp vụ, đồng thời cũng là module thường được hỏi nhất trong các buổi phỏng vấn Java. Khi học Collections, bạn cần biết mỗi container phù hợp với trường hợp sử dụng nào, đồng thời hiểu các lựa chọn thiết kế phía sau việc mở rộng capacity, hash collision, iterator, thread safety và concurrent collections.

## Dành cho ai

- Người muốn nắm vững có hệ thống Java Collections Framework.
- Người chuẩn bị các câu hỏi phỏng vấn liên quan đến List, Map, Queue, concurrent collections và phân tích source code.
- Người thường xuyên sử dụng Collections nhưng chưa nắm rõ các chi tiết như mở rộng capacity, hash collision, fail-fast, thread safety.
- Kỹ sư muốn đọc source code JDK và bắt đầu xây dựng năng lực phân tích source code từ các class Collections thường dùng.

## Trọng tâm học

- Hệ thống interface và định vị các class triển khai thường gặp của List, Set, Map, Queue.
- Cấu trúc dữ liệu bên trong và cơ chế mở rộng capacity của `ArrayList`, `LinkedList`, `HashMap`, `LinkedHashMap`.
- Tư duy bảo đảm thread safety của các concurrent collections như `ConcurrentHashMap`, `CopyOnWriteArrayList`, `ArrayBlockingQueue`.
- Các chi tiết thường gặp như hash collision, chuyển thành red-black tree, fail-fast, xoá bằng iterator, kiểm tra Collections rỗng và ước tính capacity.
- Cách bắt đầu phân tích source code từ bốn góc độ: cấu trúc dữ liệu, field quan trọng, method cốt lõi và kiểm soát concurrency.

## Thứ tự đọc đề xuất

1. [Tổng hợp câu hỏi phỏng vấn Java Collections (phần 1)](./java-collection-questions-01.md): Trước tiên xây dựng danh sách vấn đề về Collections Framework và các container thường gặp.
2. [Tổng hợp câu hỏi phỏng vấn Java Collections (phần 2)](./java-collection-questions-02.md): Tiếp tục bổ sung các chi tiết về Map, Queue, concurrent collections và source code.
3. [Tổng hợp lưu ý khi sử dụng Java Collections](./java-collection-precautions-for-use.md): Nắm được những cách sử dụng trong project dễ gặp lỗi nhất.
4. [Phân tích source code ArrayList](./arraylist-source-code.md), [Phân tích source code LinkedList](./linkedlist-source-code.md), [Phân tích source code HashMap](./hashmap-source-code.md): Bắt đầu đọc source code từ các container được sử dụng thường xuyên nhất.
5. [Phân tích source code ConcurrentHashMap](./concurrent-hash-map-source-code.md), [Phân tích source code CopyOnWriteArrayList](./copyonwritearraylist-source-code.md), [Phân tích source code ArrayBlockingQueue](./arrayblockingqueue-source-code.md): Sau đó tìm hiểu concurrent collections và blocking queue.

## Bài viết cốt lõi

### Câu hỏi phỏng vấn và quy tắc sử dụng Collections

- [Tổng hợp câu hỏi phỏng vấn Java Collections (phần 1)](./java-collection-questions-01.md): Bao quát các vấn đề cơ bản về Collections Framework, List, Set, Map, Queue.
- [Tổng hợp câu hỏi phỏng vấn Java Collections (phần 2)](./java-collection-questions-02.md): Tiếp tục hệ thống hoá hash table, concurrent collections, source code Collections và các lỗi thường gặp.
- [Tổng hợp lưu ý khi sử dụng Java Collections](./java-collection-precautions-for-use.md): Tổng hợp các lưu ý liên quan đến khởi tạo Collections, kiểm tra rỗng, duyệt và xoá, thread safety và performance.

### Source code List và Map

- [Phân tích source code ArrayList](./arraylist-source-code.md): Tìm hiểu dynamic array, mở rộng capacity, truy cập ngẫu nhiên và iterator.
- [Phân tích source code LinkedList](./linkedlist-source-code.md): Tìm hiểu doubly linked list, thao tác đầu cuối và trường hợp sử dụng phù hợp.
- [Phân tích source code HashMap](./hashmap-source-code.md): Tìm hiểu array, linked list, red-black tree, hàm perturbation, mở rộng capacity và treeification.
- [Phân tích source code LinkedHashMap](./linkedhashmap-source-code.md): Tìm hiểu thứ tự truy cập, thứ tự chèn và trường hợp sử dụng LRU.

Nếu chưa quen với các cấu trúc bên trong, bạn có thể đọc trước [Giải thích chi tiết linear data structure](../../cs-basics/data-structure/linear-data-structure.md), [Tổng hợp câu hỏi phỏng vấn hash table](../../cs-basics/data-structure/hash-table.md), [Giải thích chi tiết red-black tree](../../cs-basics/data-structure/red-black-tree.md) và [Tổng hợp câu hỏi phỏng vấn LRU cache](../../cs-basics/data-structure/lru-cache.md), sau đó quay lại đọc source code Collections sẽ thuận lợi hơn.

### Concurrent collections và Queue

- [Phân tích source code ConcurrentHashMap](./concurrent-hash-map-source-code.md): Tìm hiểu sự tiến hoá từ segmented lock đến CAS + synchronized.
- [Phân tích source code CopyOnWriteArrayList](./copyonwritearraylist-source-code.md): Tìm hiểu copy-on-write và trường hợp sử dụng nhiều đọc, ít ghi.
- [Phân tích source code ArrayBlockingQueue](./arrayblockingqueue-source-code.md): Tìm hiểu bounded blocking queue, lock và condition queue.
- [Phân tích source code PriorityQueue (trả phí)](./priorityqueue-source-code.md): Tìm hiểu cấu trúc heap và priority queue.
- [Phân tích source code DelayQueue](./delayqueue-source-code.md): Tìm hiểu delay queue, priority queue và trường hợp sử dụng trong scheduled task.

## Câu hỏi thường gặp

- `ArrayList` và `LinkedList` khác nhau thế nào? Tại sao trong nhiều trường hợp `ArrayList` được khuyến nghị hơn?
- Cấu trúc dữ liệu bên trong của `HashMap` là gì? Khi nào sẽ treeify?
- Tại sao `HashMap` không thread-safe? Có thể xảy ra vấn đề gì khi mở rộng capacity?
- `HashMap` và `ConcurrentHashMap` khác nhau thế nào?
- Cách triển khai `ConcurrentHashMap` trong JDK 7 và JDK 8 thay đổi thế nào?
- Tại sao `CopyOnWriteArrayList` phù hợp với trường hợp nhiều đọc, ít ghi?
- fail-fast và fail-safe khác nhau thế nào?
- Làm thế nào để xoá phần tử an toàn khi duyệt Collections?
- `ArrayBlockingQueue`, `PriorityQueue`, `DelayQueue` lần lượt phù hợp với những trường hợp sử dụng nào?

## Chuyên đề liên quan

- [Hệ thống kiến thức Java](../)
- [Chuyên đề Java Basics](../basis/)
- [Chuyên đề Java Concurrency](../concurrent/)
- [Chuyên đề JVM](../jvm/)
- [Cấu trúc dữ liệu](../../cs-basics/data-structure/)
- [Tổng hợp câu hỏi phỏng vấn hash table](../../cs-basics/data-structure/hash-table.md)
- [Tổng hợp câu hỏi phỏng vấn LRU cache](../../cs-basics/data-structure/lru-cache.md)

<!-- @include: @article-footer.snippet.md -->
