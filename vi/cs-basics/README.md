---
title: "Hệ thống kiến thức nền tảng máy tính: mạng máy tính, hệ điều hành, cấu trúc dữ liệu và thuật toán"
description: Lộ trình học và phỏng vấn kiến thức nền tảng máy tính, bao quát mạng máy tính, hệ điều hành, cấu trúc dữ liệu, thuật toán, Linux, TCP/IP, HTTP, DNS và các nội dung khác, phù hợp để ôn tập phỏng vấn khi mới tốt nghiệp và đã đi làm.
icon: "mdi:desktop-classic"
sitemap:
  changefreq: weekly
  priority: 0.95
head:
  - - meta
    - name: keywords
      content: Kiến thức nền tảng máy tính,Tổng hợp kiến thức nền tảng máy tính,Câu hỏi phỏng vấn kiến thức nền tảng máy tính,Mạng máy tính,Câu hỏi phỏng vấn mạng máy tính,Hệ điều hành,Câu hỏi phỏng vấn hệ điều hành,Cấu trúc dữ liệu,Câu hỏi phỏng vấn cấu trúc dữ liệu,Thuật toán,Câu hỏi phỏng vấn thuật toán,Linux,TCP/IP,HTTP,DNS,Phỏng vấn backend,Phỏng vấn Java,Câu hỏi lý thuyết
  - - meta
    - property: og:title
      content: "Hệ thống kiến thức nền tảng máy tính: mạng máy tính, hệ điều hành, cấu trúc dữ liệu và thuật toán"
  - - meta
    - property: og:description
      content: Hệ thống hóa kiến thức nền tảng máy tính như mạng máy tính, hệ điều hành, cấu trúc dữ liệu và thuật toán, phù hợp để backend developer ôn tập phỏng vấn khi mới tốt nghiệp hoặc đã đi làm.
---

<!-- @include: @small-advertisement.snippet.md -->

**Hệ thống kiến thức nền tảng máy tính** này hướng đến việc học backend và ôn tập phỏng vấn, sắp xếp các bài viết về kiến thức nền tảng máy tính trên website theo thứ tự “mạng máy tính -> hệ điều hành -> cấu trúc dữ liệu -> thuật toán”.

Nếu có ít thời gian, bạn nên đọc trước [Tổng hợp câu hỏi phỏng vấn mạng máy tính thường gặp](./network/other-network-questions.md) và [Tổng hợp câu hỏi phỏng vấn hệ điều hành thường gặp](./operating-system/operating-system-basic-questions-01.md) để nhanh chóng xây dựng danh sách câu hỏi thường gặp; nếu muốn củng cố kiến thức nền tảng một cách có hệ thống, bạn có thể học theo thứ tự chuyên đề bên dưới.

Toàn bộ website có **hơn 300 hình minh họa kỹ thuật**, dùng hình ảnh để giải thích rõ các khái niệm trừu tượng, thay vì chỉ có những đoạn văn khô khan.

![Tổng quan nội dung tổng hợp kiến thức nền tảng máy tính](https://oss.javaguide.cn/github/javaguide/cs-basics/network/cs-basics-overview.png)

## Dành cho ai

- Backend developer đang củng cố kiến thức nền tảng máy tính một cách có hệ thống.
- Người chuẩn bị phỏng vấn backend khi mới tốt nghiệp, đã đi làm hoặc tại các công ty công nghệ vừa và lớn.
- Độc giả muốn kết nối mạng máy tính, hệ điều hành, cấu trúc dữ liệu và thuật toán thành một hệ thống kiến thức hoàn chỉnh.
- Kỹ sư đã từng viết code nghiệp vụ nhưng chưa vững các kiến thức nền tảng như TCP/IP, HTTP, process và thread, memory management, tree và graph, sorting.

## Trọng tâm học

- Với mạng máy tính, cần tập trung hiểu mô hình phân tầng, TCP/UDP, HTTP/HTTPS, DNS, ARP, NAT và các vấn đề network security thường gặp.
- Với hệ điều hành, cần tập trung hiểu process và thread, lock và synchronization, memory management, virtual memory, zero-copy, I/O multiplexing, file system, kiến thức Linux cơ bản và cách sử dụng Shell.
- Với cấu trúc dữ liệu, cần tập trung hiểu đặc điểm và trường hợp sử dụng của array, linked list, stack, queue, hash table, tree, graph, heap, Trie, union-find, skip list, red-black tree, Bloom filter và LRU.
- Với thuật toán, cần tập trung hiểu phân tích complexity, binary search, two pointers, sliding window, DFS/BFS, backtracking, dynamic programming, greedy, Top K, sorting, string, linked list và các bài LeetCode thường gặp.
- Khi phỏng vấn, cần có thể kết nối “khái niệm -> nguyên lý -> so sánh -> trường hợp sử dụng -> câu hỏi thường gặp” thành một câu trả lời hoàn chỉnh.

## Thứ tự đọc đề xuất

1. [Chuyên đề mạng máy tính](./network/): Bắt đầu từ mô hình phân tầng, HTTP, TCP, DNS và các câu hỏi phỏng vấn mạng máy tính thường gặp để xây dựng nhận thức tổng thể về giao tiếp mạng.
2. [Chuyên đề hệ điều hành](./operating-system/): Hiểu process và thread, memory, file system, Linux và Shell, làm nền tảng cho lập trình concurrency, JVM và database.
3. [Chuyên đề cấu trúc dữ liệu](./data-structure/): Nắm vững linear table, hash table, tree, graph, heap, Trie, union-find, skip list, red-black tree, Bloom filter, LRU và các cấu trúc thường gặp khác.
4. [Chuyên đề thuật toán](./algorithms/): Luyện tập kết hợp phân tích complexity, template thuật toán cốt lõi và các bài LeetCode thường gặp.
5. Quay lại làm câu hỏi phỏng vấn để bổ sung những phần còn thiếu: tập trung ôn lại các câu hỏi thường gặp về mạng máy tính và hệ điều hành, sau đó luyện lại các bài cấu trúc dữ liệu và thuật toán theo từng dạng.

Nếu công ty mục tiêu coi trọng thuật toán, bạn nên kết hợp bước 3 và bước 4 khi ôn tập: học một cấu trúc dữ liệu trước, sau đó luyện các dạng bài tương ứng. Ví dụ, sau khi học [hash table](./data-structure/hash-table.md), hãy luyện bài Two Sum và prefix sum; sau khi học [heap](./data-structure/heap.md), hãy luyện [Top K](./algorithms/top-k.md); sau khi học [tree](./data-structure/tree.md) và [graph](./data-structure/graph.md), hãy tập trung luyện [DFS và BFS](./algorithms/dfs-bfs.md), [backtracking](./algorithms/backtracking.md) và [dynamic programming](./algorithms/dynamic-programming.md).

## Bài viết cốt lõi

### Mạng máy tính

- [Chuyên đề mạng máy tính](./network/): Hệ thống hóa kiến thức cốt lõi và các câu hỏi phỏng vấn thường gặp về mạng máy tính theo từng tầng giao thức.
- [Tổng hợp câu hỏi phỏng vấn mạng máy tính thường gặp (phần 1)](./network/other-network-questions.md): Bao quát các vấn đề nền tảng như mô hình OSI/TCP-IP, HTTP, HTTPS, DNS.
- [Tổng hợp câu hỏi phỏng vấn mạng máy tính thường gặp (phần 2)](./network/other-network-questions2.md): Tiếp tục bổ sung các vấn đề thường gặp như TCP, UDP, network security và Socket.
- [Giải thích chi tiết mô hình bảy tầng OSI và mô hình bốn tầng TCP/IP](./network/osi-and-tcp-ip-model.md): Xây dựng nhận thức về phân tầng mạng và trách nhiệm của các giao thức.
- [Điều gì thực sự xảy ra từ lúc nhập URL đến khi trang được hiển thị?](./network/the-whole-process-of-accessing-web-pages.md): Kết nối DNS, TCP, HTTP, browser rendering và các kiến thức liên quan thông qua một request hoàn chỉnh.
- [HTTP vs HTTPS](./network/http-vs-https.md), [HTTP 1.0 vs HTTP 1.1](./network/http1.0-vs-http1.1.md), [Tổng hợp các status code thường gặp của HTTP](./network/http-status-codes.md): Tập trung hiểu các nội dung thường gặp về HTTP.
- [TCP three-way handshake và four-way handshake](./network/tcp-connection-and-disconnection.md), [Cơ chế bảo đảm tính tin cậy khi truyền TCP](./network/tcp-reliability-guarantee.md): Nắm vững cơ chế quản lý connection và reliable transmission cốt lõi nhất của TCP.

### Hệ điều hành

- [Chuyên đề hệ điều hành](./operating-system/): Trình bày từ kiến thức cơ bản về hệ điều hành đến các câu hỏi thường gặp về Linux.
- [Tổng hợp câu hỏi phỏng vấn hệ điều hành thường gặp (phần 1)](./operating-system/operating-system-basic-questions-01.md): Bao quát các vấn đề về nền tảng hệ điều hành, process và thread, deadlock, memory management.
- [Tổng hợp câu hỏi phỏng vấn hệ điều hành thường gặp (phần 2)](./operating-system/operating-system-basic-questions-02.md): Tiếp tục hệ thống hóa các nội dung phỏng vấn về file system, I/O, Linux.
- [Giải thích chi tiết về process và thread: khác biệt, trạng thái, giao tiếp, context switch và virtual thread](./operating-system/process-and-thread.md): Làm rõ resource boundary, state transition, context switch của process và thread, cùng Java virtual thread.
- [Giải thích chi tiết về inter-process communication (IPC): pipe, message queue, shared memory, Socket và Binder](./operating-system/ipc.md): So sánh các cơ chế IPC như pipe, message queue, shared memory, Socket và Binder.
- [Giải thích chi tiết về lock và synchronization của hệ điều hành: mutex, semaphore, condition variable, spinlock và futex](./operating-system/os-lock-and-sync.md): Làm rõ critical section, mutex, semaphore, condition variable, spinlock và futex.
- [Giải thích chi tiết về memory management của hệ điều hành: paging, segmentation, page replacement, Swap và OOM](./operating-system/memory-management.md): Làm rõ memory allocation, memory fragmentation, page table, TLB, page replacement, Swap và OOM.
- [Giải thích chi tiết về virtual memory: address translation, TLB, page fault và page replacement](./operating-system/virtual-memory.md): Làm rõ paging, page table, TLB, page fault và page replacement.
- [Giải thích chi tiết về file system của hệ điều hành: inode, VFS, Page Cache và cơ chế log](./operating-system/file-system.md): Làm rõ inode, dentry, file descriptor, VFS, Page Cache và cơ chế log.
- [Giải thích chi tiết về I/O multiplexing: nguyên lý và khác biệt của select, poll, epoll](./operating-system/io-multiplexing.md): Làm rõ nguyên lý triển khai, khác biệt về performance và trường hợp sử dụng của select, poll, epoll.
- [Giải thích chi tiết về zero-copy: mmap, sendfile và splice](./operating-system/zero-copy.md): Làm rõ đường đi của dữ liệu khi copy trong I/O truyền thống, mmap, sendfile, splice và ứng dụng trong thực tế.
- [Tổng hợp kiến thức nền tảng Linux](./operating-system/linux-intro.md): Nắm vững thư mục Linux, file permission, command thường dùng và kiến thức cơ bản về system.
- [Tổng hợp kiến thức nền tảng lập trình Shell](./operating-system/shell-intro.md): Bổ sung khả năng viết script, sử dụng variable, flow control và command thường dùng.

### Cấu trúc dữ liệu

- [Chuyên đề cấu trúc dữ liệu](./data-structure/): Hệ thống hóa các cấu trúc dữ liệu thường gặp và hình minh họa theo từng loại cấu trúc.
- [Giải thích chi tiết về linear data structure](./data-structure/linear-data-structure.md): Hiểu đặc điểm lưu trữ và complexity thao tác của array, linked list, stack, queue.
- [Tổng hợp câu hỏi phỏng vấn hash table](./data-structure/hash-table.md): Hiểu hash function, hash collision, mở rộng và mối liên hệ với Java `HashMap`.
- [Giải thích chi tiết về tree structure](./data-structure/tree.md): Nắm vững các tree structure thường gặp như binary tree, binary search tree, AVL, B tree, B+ tree.
- [Giải thích chi tiết về graph](./data-structure/graph.md): Hiểu cách biểu diễn graph, DFS, BFS và các thuật toán cơ bản như shortest path.
- [Giải thích chi tiết về heap](./data-structure/heap.md), [Giải thích chi tiết về red-black tree](./data-structure/red-black-tree.md), [Giải thích chi tiết về Bloom filter](./data-structure/bloom-filter.md): Bổ sung các cấu trúc thường gặp trong engineering và các nội dung phỏng vấn quan trọng.
- [Tổng hợp câu hỏi phỏng vấn Trie prefix tree](./data-structure/trie.md), [Tổng hợp câu hỏi phỏng vấn union-find](./data-structure/union-find.md), [Tổng hợp câu hỏi phỏng vấn skip list](./data-structure/skip-list.md), [Tổng hợp câu hỏi phỏng vấn LRU cache](./data-structure/lru-cache.md): Bổ sung các trường hợp sử dụng thường gặp về string set, connectivity, Redis ZSet và cache eviction.

Khi ôn tập cấu trúc dữ liệu, bạn có thể đồng thời xem lại các chuyên đề về Java và database: array/linked list/hash table tương ứng với [Java Collection](../java/collection/), B+ tree tương ứng với [MySQL index](../database/mysql/mysql-index.md), skip list tương ứng với [Redis skip list](../database/redis/redis-skiplist.md), LRU và Bloom filter tương ứng với các trường hợp sử dụng cache.

### Thuật toán

- [Chuyên đề thuật toán](./algorithms/): Hệ thống hóa các tư duy thuật toán thường gặp, các bài LeetCode thường gặp và các thuật toán sorting kinh điển.
- [Hướng dẫn phỏng vấn về time complexity và space complexity](./algorithms/complexity-analysis.md): Nắm vững Big O, complexity của recursion và các cách hiểu sai complexity thường gặp.
- [Tổng hợp câu hỏi phỏng vấn binary search](./algorithms/binary-search.md), [Tổng hợp câu hỏi phỏng vấn two pointers và sliding window](./algorithms/two-pointers-and-sliding-window.md): Nắm vững các template viết tay thường gặp nhất trong bài array, string và linked list.
- [Tổng hợp câu hỏi phỏng vấn DFS và BFS](./algorithms/dfs-bfs.md), [Tổng hợp câu hỏi phỏng vấn backtracking](./algorithms/backtracking.md): Nắm vững tree search, graph search, matrix search, permutation, combination và path enumeration.
- [Tổng hợp câu hỏi phỏng vấn dynamic programming](./algorithms/dynamic-programming.md), [Tổng hợp câu hỏi phỏng vấn greedy algorithm](./algorithms/greedy.md), [Tổng hợp câu hỏi phỏng vấn bài toán Top K](./algorithms/top-k.md): Bổ sung các dạng bài liên quan đến optimal value, interval greedy, heap và data stream.
- [Tổng hợp các tư duy thuật toán kinh điển](./algorithms/classical-algorithm-problems-recommendations.md): Bao quát các tư duy thường gặp như binary search, two pointers, sliding window, backtracking và dynamic programming.
- [Đề xuất các bài LeetCode kinh điển về cấu trúc dữ liệu thường gặp](./algorithms/common-data-structures-leetcode-recommendations.md): Hệ thống hóa lộ trình luyện bài theo từng loại cấu trúc dữ liệu.
- [Một số bài toán thuật toán về string thường gặp](./algorithms/string-algorithm-problems.md), [Một số bài toán thuật toán về linked list thường gặp](./algorithms/linkedlist-algorithm-problems.md): Tập trung luyện các dạng bài thường gặp.
- [Một số bài lập trình trong Sword Offer](./algorithms/the-sword-refers-to-offer.md), [Tổng hợp mười thuật toán sorting kinh điển](./algorithms/10-classical-sorting-algorithms.md): Phù hợp để ôn lại trước phỏng vấn.

Khi luyện bài thuật toán, bạn nên viết thật vững template của từng dạng trước, sau đó làm theo danh sách bài: [Tổng hợp các tư duy thuật toán kinh điển](./algorithms/classical-algorithm-problems-recommendations.md) phù hợp để luyện theo dạng bài, còn [Đề xuất các bài LeetCode kinh điển về cấu trúc dữ liệu thường gặp](./algorithms/common-data-structures-leetcode-recommendations.md) phù hợp để luyện theo cấu trúc.

## Câu hỏi thường gặp

- Mô hình bảy tầng OSI và mô hình bốn tầng TCP/IP lần lượt là gì? Mỗi tầng giải quyết vấn đề nào?
- Từ lúc nhập URL đến khi trang được hiển thị, những bước nào diễn ra ở giữa?
- HTTP và HTTPS khác nhau như thế nào? Vì sao HTTPS an toàn hơn?
- TCP three-way handshake và four-way handshake lần lượt giải quyết vấn đề gì? Vì sao TIME_WAIT tồn tại?
- TCP bảo đảm reliable transmission như thế nào? Nên chọn TCP hay UDP?
- Process và thread khác nhau như thế nào? Deadlock là gì và làm thế nào để tránh?
- Memory management, virtual memory, paging và segmentation của hệ điều hành lần lượt là gì?
- Array, linked list, stack, queue, tree, graph và heap lần lượt phù hợp với trường hợp sử dụng nào?
- Hash table, red-black tree, B+ tree, skip list, Bloom filter và LRU thường được dùng ở đâu trong engineering?
- Khi luyện bài thuật toán, làm thế nào để xây dựng template giải bài theo từng dạng?

## Chuyên đề liên quan

- [Hệ thống kiến thức Java](../java/)
- [Hệ thống kiến thức database](../database/)
- [Hệ thống kiến thức distributed system](../distributed-system/)
- [Hệ thống kiến thức high-performance](../high-performance/)
- [System Design](../system-design/)
- [Đề xuất sách về kiến thức nền tảng máy tính](../books/cs-basics.md)

<!-- @include: @article-footer.snippet.md -->
