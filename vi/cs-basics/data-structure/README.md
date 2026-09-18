---
title: "Hệ thống kiến thức cấu trúc dữ liệu: array, linked list, hash table, tree, graph, heap và phỏng vấn"
description: Lộ trình ôn tập câu hỏi phỏng vấn cấu trúc dữ liệu, bao quát array, linked list, stack, queue, hash table, tree, graph, heap, Trie, union-find, skip list, red-black tree, Bloom filter, LRU, phân tích complexity và các trường hợp sử dụng trong Java backend.
category: Computer Basics
tag:
  - Data Structures
  - Algorithms
  - Câu hỏi phỏng vấn
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Data Structures,câu hỏi phỏng vấn cấu trúc dữ liệu,lộ trình ôn tập cấu trúc dữ liệu,array,linked list,stack,queue,hash table,HashMap,tree,graph,heap,Trie,union-find,skip list,red-black tree,Bloom filter,LRU,Java Collections,Redis,MySQL index,phỏng vấn backend
---

Hệ thống kiến thức **cấu trúc dữ liệu** này tổ chức nội dung theo phỏng vấn và các trường hợp sử dụng trong Java backend: trước tiên hiểu dữ liệu được lưu trữ thế nào, sau đó tìm hiểu complexity của các thao tác thường gặp, cuối cùng kết nối các cấu trúc với những vấn đề thực tế như Java Collections, MySQL index, Redis, cache và message queue.

Trong phỏng vấn, câu hỏi về cấu trúc dữ liệu hiếm khi chỉ dừng ở “array là gì”. Thường gặp hơn là các câu hỏi tiếp nối: Vì sao array truy vấn nhanh còn linked list thuận tiện cho chèn và xoá? Vì sao `HashMap` cần resize? Vì sao B+ tree phù hợp làm index? Vì sao Bloom filter có thể cho kết quả dương tính giả? Đằng sau những câu hỏi này đều kiểm tra cùng một điều: bạn có hiểu complexity và sự đánh đổi giữa các trường hợp sử dụng do việc lựa chọn cấu trúc mang lại hay không.

Khi chuẩn bị về cấu trúc dữ liệu, không nên chỉ học thuộc định nghĩa. Cách hiệu quả hơn là tách mỗi cấu trúc thành 4 câu hỏi: lưu trữ thế nào, truy vấn thế nào, thay đổi thế nào, phù hợp với trường hợp sử dụng nào. Khi giải thích rõ được 4 câu hỏi này rồi mới làm các bài toán thuật toán tương ứng, hiệu quả sẽ cao hơn nhiều.

## Dành cho ai

- Bạn đang bổ sung nền tảng cấu trúc dữ liệu và chuẩn bị phỏng vấn backend khi tuyển dụng mới tốt nghiệp hoặc đã đi làm.
- Bạn thường bị mắc ở các cấu trúc như array, linked list, tree, graph, heap khi làm bài toán thuật toán.
- Bạn muốn liên hệ cấu trúc dữ liệu với Java Collections, Redis và hệ thống cache.
- Bạn đã xem qua các khái niệm nhưng khi trả lời câu hỏi phỏng vấn thường chỉ dừng ở mức định nghĩa.

## Phỏng vấn cấu trúc dữ liệu hỏi gì

| Điểm khảo sát           | Cách hỏi thường gặp                                                           | Trọng tâm ôn tập                                  |
| ----------------------- | ----------------------------------------------------------------------------- | ------------------------------------------------- |
| Cách lưu trữ            | Sequential storage và linked storage khác nhau thế nào?                       | Tính liên tục của memory, pointer, cache-friendly |
| Complexity của thao tác | Vì sao truy vấn array là O(1), còn truy vấn linked list là O(n)?              | Complexity của truy vấn, chèn, xoá, duyệt         |
| So sánh cấu trúc        | Chọn red-black tree hay AVL tree thế nào? B tree và B+ tree khác nhau ra sao? | Bảng so sánh + trường hợp sử dụng                 |
| Liên hệ thực tế         | HashMap, TreeMap, PriorityQueue, Redis ZSet sử dụng cấu trúc gì?              | Ứng dụng thực tế trong Java/database/cache        |
| Tiếp nối thuật toán     | Viết tree traversal, graph search, Top K, LRU thế nào?                        | Ôn cùng template thuật toán                       |

## Khung trả lời phỏng vấn

Câu hỏi về cấu trúc dữ liệu không nên chỉ được trả lời ở mức “nó là gì”. Cách trình bày vững hơn trong phỏng vấn là triển khai theo mạch sau:

```text
Định nghĩa -> Cách lưu trữ -> Complexity của thao tác thường gặp -> Ưu / nhược điểm -> Trường hợp sử dụng -> Ứng dụng trong Java/Redis/MySQL
```

Lấy hash table làm ví dụ, một câu trả lời đầy đủ có thể được trình bày như sau:

1. Hash table dùng hash function để ánh xạ key vào index của array.
2. Truy vấn, chèn và xoá có complexity trung bình là `O(1)`, nhưng sẽ suy giảm khi collision nghiêm trọng.
3. Có thể xử lý collision bằng các cách như chaining và open addressing.
4. Java `HashMap` sử dụng array + linked list + red-black tree; resize dùng để kiểm soát load factor.
5. Cấu trúc này phù hợp với các trường hợp sử dụng như tìm kiếm nhanh, đếm, loại trùng và index cho cache, nhưng sẽ tiêu tốn thêm không gian.

Cách trả lời này chịu được các câu hỏi tiếp nối tốt hơn câu “truy vấn hash table là O(1)”, vì đồng thời giải thích nguyên lý, complexity và ứng dụng thực tế.

## Thứ tự đọc đề xuất

1. [Giải thích chi tiết linear data structure](./linear-data-structure.md): trước tiên nắm vững array, linked list, stack và queue, hiểu sequential storage và linked storage.
2. [Tổng hợp câu hỏi phỏng vấn hash table](./hash-table.md): hiểu hash function, collision, resize và liên hệ với `HashMap`.
3. [Giải thích chi tiết tree structure](./tree.md): nắm vững binary tree, binary search tree, AVL, B tree, B+ tree và mối liên hệ với MySQL index.
4. [Giải thích chi tiết heap](./heap.md): hiểu priority queue, Top K, heap sort và `PriorityQueue`.
5. [Giải thích chi tiết graph](./graph.md): hiểu cách lưu trữ graph, DFS, BFS, topological sort và điểm bắt đầu của shortest path.
6. [Tổng hợp câu hỏi phỏng vấn Trie prefix tree](./trie.md), [Tổng hợp câu hỏi phỏng vấn union-find](./union-find.md): bổ sung kiến thức về tập hợp chuỗi và các vấn đề connectivity.
7. [Tổng hợp câu hỏi phỏng vấn skip list](./skip-list.md), [Giải thích chi tiết red-black tree](./red-black-tree.md), [Giải thích chi tiết Bloom filter](./bloom-filter.md), [Tổng hợp câu hỏi phỏng vấn LRU cache](./lru-cache.md): ôn tập theo các trường hợp sử dụng với Java Collections, Redis, cache và database.

## Bài viết cốt lõi

| Bài viết                                                                | Trọng tâm                                  | Liên hệ thường gặp                          |
| ----------------------------------------------------------------------- | ------------------------------------------ | ------------------------------------------- |
| [Giải thích chi tiết linear data structure](./linear-data-structure.md) | array, linked list, stack, queue           | `ArrayList`, `LinkedList`, message queue    |
| [Tổng hợp câu hỏi phỏng vấn hash table](./hash-table.md)                | hash function, collision, resize           | `HashMap`, cache, loại trùng                |
| [Giải thích chi tiết tree structure](./tree.md)                         | binary tree, BST, AVL, B tree, B+ tree     | MySQL index, expression tree                |
| [Giải thích chi tiết graph](./graph.md)                                 | adjacency list, adjacency matrix, DFS, BFS | quan hệ phụ thuộc, routing, quan hệ đề xuất |
| [Giải thích chi tiết heap](./heap.md)                                   | max heap, min heap, heap sort              | `PriorityQueue`, Top K, delay queue         |
| [Giải thích chi tiết red-black tree](./red-black-tree.md)               | cân bằng gần đúng, rotation, đổi màu       | `TreeMap`, treeification của `HashMap`      |
| [Giải thích chi tiết Bloom filter](./bloom-filter.md)                   | bit array, hash, false positive            | cache penetration, loại trùng, blacklist    |
| [Tổng hợp câu hỏi phỏng vấn skip list](./skip-list.md)                  | multi-level index, range query             | Redis ZSet                                  |
| [Tổng hợp câu hỏi phỏng vấn LRU cache](./lru-cache.md)                  | hash table + doubly linked list            | local cache, page replacement               |

## Tra nhanh cách chọn cấu trúc

Nhiều câu hỏi về cấu trúc dữ liệu thực chất đang hỏi “vì sao chọn cấu trúc này trong trường hợp sử dụng này, thay vì một cấu trúc khác”. Bảng dưới đây phù hợp để ôn tập nhanh trước phỏng vấn:

| Trường hợp sử dụng                          | Ưu tiên cân nhắc                   | Điểm đánh đổi                                                                                                |
| ------------------------------------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| Truy cập ngẫu nhiên theo index thường xuyên | array, `ArrayList`                 | Truy vấn nhanh, chi phí chèn xoá phần tử ở giữa cao                                                          |
| Thường xuyên chèn xoá ở hai đầu             | deque, linked list                 | Thao tác pointer linh hoạt nhưng truy cập ngẫu nhiên chậm                                                    |
| Nhanh chóng xác định phần tử có tồn tại     | hash table, Bloom filter           | Hash table chính xác nhưng tốn không gian, Bloom filter tiết kiệm không gian nhưng có thể cho false positive |
| Duy trì ordered set và range query          | red-black tree, skip list, B+ tree | Red-black tree phù hợp với ordered set trong memory, skip list phù hợp với range query và triển khai thực tế |
| Xử lý giá trị lớn nhất, nhỏ nhất, Top K     | heap, priority queue               | Chỉ quan tâm cực trị cục bộ, không phù hợp để duyệt toàn bộ theo thứ tự                                      |
| Xác định connectivity và phân nhóm          | union-find                         | Merge và query nhanh nhưng không phù hợp khi thường xuyên xoá quan hệ                                        |
| So khớp prefix, gợi ý tìm kiếm              | Trie                               | Truy vấn liên quan đến độ dài chuỗi nhưng số lượng node có thể lớn                                           |
| Loại bỏ cache                               | LRU, LFU                           | LRU xét lần truy cập gần nhất, LFU xét tần suất truy cập                                                     |

Khi ôn tập, bạn có thể tự hỏi ngược lại: nếu không dùng cấu trúc này thì chậm ở đâu? Sẽ tốn thêm bao nhiêu không gian? Điều kiện biên là gì? Khi suy nghĩ rõ các câu hỏi này, bạn thường có thể xử lý được các câu hỏi tiếp nối trong phỏng vấn.

## Lộ trình ôn tập 7 ngày

| Ngày   | Trọng tâm                               | Hành động đề xuất                                                                |
| ------ | --------------------------------------- | -------------------------------------------------------------------------------- |
| Ngày 1 | array, linked list                      | Viết bảng complexity, tự viết code đảo linked list và xoá node                   |
| Ngày 2 | stack, queue, hash table                | Viết bài matching dấu ngoặc, dùng stack triển khai queue, two sum                |
| Ngày 3 | tree                                    | Viết tree traversal, lowest common ancestor, ôn lại B+ tree                      |
| Ngày 4 | heap                                    | Viết Top K, K phần tử có tần suất cao nhất, hiểu `PriorityQueue`                 |
| Ngày 5 | graph                                   | Viết DFS/BFS, bài toán số lượng đảo, course schedule                             |
| Ngày 6 | red-black tree, skip list, Bloom filter | Tập trung chuẩn bị các câu hỏi tiếp nối về trường hợp sử dụng thực tế            |
| Ngày 7 | LRU và ôn tập tổng hợp                  | Tự viết code LRU, hệ thống hoá complexity và trường hợp sử dụng của mọi cấu trúc |

## Lộ trình ôn tập 30 ngày

| Giai đoạn   | Thời gian      | Mục tiêu                                                                                        |
| ----------- | -------------- | ----------------------------------------------------------------------------------------------- |
| Giai đoạn 1 | Ngày 1 đến 6   | Linear structure và hash table, giải thích rõ complexity và mối liên hệ với Java Collections    |
| Giai đoạn 2 | Ngày 7 đến 13  | Tree, heap, graph, kết hợp DFS/BFS và làm bài Top K                                             |
| Giai đoạn 3 | Ngày 14 đến 20 | Trie, union-find, skip list, red-black tree, bổ sung các cấu trúc nâng cao                      |
| Giai đoạn 4 | Ngày 21 đến 25 | Bloom filter, LRU, câu hỏi về trường hợp sử dụng thực tế, kết nối Redis/MySQL/cache             |
| Giai đoạn 5 | Ngày 26 đến 30 | Ôn lại các bài làm sai và luyện trả lời phỏng vấn, chuẩn bị 2 câu hỏi tiếp nối cho mỗi cấu trúc |

## Tự kiểm tra các câu hỏi thường gặp

- Bố cục memory của array và linked list khác nhau thế nào? Vì sao array truy cập ngẫu nhiên nhanh?
- Chọn `ArrayList` hay `LinkedList` trong Java thế nào?
- Stack và queue lần lượt phù hợp với những trường hợp sử dụng nào? Monotonic stack và monotonic queue giải quyết vấn đề gì?
- Có những cách nào để xử lý hash collision? Vì sao `HashMap` cần resize?
- Binary search tree, AVL tree và red-black tree khác nhau thế nào?
- Vì sao B tree và B+ tree phù hợp với database index?
- Heap và binary tree thông thường khác nhau thế nào? Vì sao Top K thường dùng heap?
- Chọn adjacency list hay adjacency matrix của graph thế nào? Complexity của DFS và BFS là bao nhiêu?
- Vì sao skip list phù hợp với range query? Vì sao Redis sử dụng skip list?
- Vì sao Bloom filter có thể cho false positive? Vì sao việc xoá lại khó?
- Vì sao LRU thường được triển khai bằng hash table kết hợp doubly linked list?

## Chuyên đề liên quan

- [Hệ thống kiến thức Computer Basics](../)
- [Chuyên đề Algorithms](../algorithms/)
- [Đề xuất các bài toán LeetCode kinh điển về cấu trúc dữ liệu](../algorithms/common-data-structures-leetcode-recommendations.md)
- [Java Collections](../../java/collection/java-collection-questions-01.md)
- [Giải thích chi tiết MySQL index](../../database/mysql/mysql-index.md)
- [Tổng hợp câu hỏi phỏng vấn Redis thường gặp](../../database/redis/redis-questions-01.md)
- [Chuẩn bị phỏng vấn](../../interview-preparation/)

<!-- @include: @article-footer.snippet.md -->
