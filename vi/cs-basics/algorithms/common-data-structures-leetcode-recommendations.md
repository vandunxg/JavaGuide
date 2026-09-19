---
title: Đề xuất các bài LeetCode kinh điển về các cấu trúc dữ liệu thường gặp
description: "Tổng hợp các bài LeetCode thường gặp theo các cấu trúc array, linked list, stack, queue, hash table, tree, graph, heap, Trie, union-find, kèm dạng bài, template, giá trị phỏng vấn và trọng tâm ôn tập."
category: Computer Basics
tag:
  - Algorithms
  - Data Structures
  - LeetCode
head:
  - - meta
    - name: keywords
      content: LeetCode,data structures,array,linked list,stack,queue,hash table,binary tree,graph,heap,Trie,union-find,bài LeetCode,lộ trình luyện bài
---

Khi làm bài về cấu trúc dữ liệu, không nên chỉ luyện từ Easy đến Hard theo độ khó. Cách vững hơn là phân loại dạng bài theo từng cấu trúc: array tập trung vào index và khoảng, linked list tập trung vào pointer, stack và queue tập trung vào ràng buộc thứ tự, tree và graph tập trung vào traversal, heap tập trung vào priority, hash table tập trung vào định vị nhanh.

Danh sách dưới đây giới hạn trong các bài thường gặp khi phỏng vấn và các bài tiêu biểu cho template. Với mỗi nhóm, hãy làm “bài nhất định phải làm” trước, sau đó mới làm “bài nâng cao”. Sau khi hoàn thành bài, ít nhất hãy ghi lại complexity, trường hợp biên và bài này thuộc template nào.

## Cách dùng danh sách bài này

Đừng chỉ ghi nhớ kết luận khi làm bài về cấu trúc dữ liệu. Mỗi khi luyện một nhóm, hãy quay lại tìm hiểu “cách lưu trữ, thao tác cốt lõi, complexity” của cấu trúc tương ứng, rồi mới bắt tay viết bài. Khi đó, nếu interviewer hỏi tiếp về Java Collections, Redis, MySQL index hoặc trường hợp sử dụng của cache, câu trả lời sẽ không chỉ dừng ở mức lời giải.

| Cấu trúc                         | Nên đọc gì trước                                                                                                                                                   | Khi luyện bài cần tập trung vào gì                                        |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------- |
| Array, linked list, stack, queue | [Giải thích chi tiết về linear data structure](../data-structure/linear-data-structure.md), [Two pointers và sliding window](./two-pointers-and-sliding-window.md) | index, cập nhật pointer, thời điểm push/pop                               |
| Hash table                       | [Tổng hợp câu hỏi phỏng vấn hash table](../data-structure/hash-table.md)                                                                                           | thiết kế key, thời điểm cập nhật số đếm, collision và resize              |
| Tree và graph                    | [Giải thích chi tiết tree structure](../data-structure/tree.md), [Giải thích chi tiết graph](../data-structure/graph.md), [DFS và BFS](./dfs-bfs.md)               | giá trị trả về của đệ quy, visited, thống kê số tầng của BFS              |
| Heap và Top K                    | [Giải thích chi tiết heap](../data-structure/heap.md), [Tổng hợp câu hỏi phỏng vấn Top K](./top-k.md)                                                              | kích thước heap, comparator, bối cảnh data stream                         |
| Trie và union-find               | [Tổng hợp câu hỏi phỏng vấn Trie prefix tree](../data-structure/trie.md), [Tổng hợp câu hỏi phỏng vấn union-find](../data-structure/union-find.md)                 | cấu trúc node, đánh dấu kết thúc, path compression, kiểm tra connectivity |
| LRU                              | [Tổng hợp câu hỏi phỏng vấn LRU cache](../data-structure/lru-cache.md)                                                                                             | cách hash table và doubly linked list cùng duy trì O(1)                   |

## Array

| Dạng bài      | Bài nhất định phải làm                                                                                        | Bài nâng cao                                                                                                                                                 | Giá trị phỏng vấn                   | Trọng tâm ôn tập                                        |
| ------------- | ------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------- | ------------------------------------------------------- |
| Binary search | [704. Binary search](https://leetcode.cn/problems/binary-search/)                                             | [34. Tìm vị trí đầu tiên và cuối cùng của phần tử trong sorted array](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/) | Kiểm tra điều kiện vòng lặp và biên | `left <= right`, cập nhật biên trái phải                |
| Sửa tại chỗ   | [26. Xóa phần tử trùng trong sorted array](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/) | [80. Xóa phần tử trùng trong sorted array II](https://leetcode.cn/problems/remove-duplicates-from-sorted-array-ii/)                                          | Kiểm tra cách dùng two pointers     | ý nghĩa slow pointer, thời điểm ghi đè                  |
| Two pointers  | [977. Bình phương các phần tử trong sorted array](https://leetcode.cn/problems/squares-of-a-sorted-array/)    | [15. Three Sum](https://leetcode.cn/problems/3sum/)                                                                                                          | Dạng bài array thường gặp           | loại trùng sau khi sorting, di chuyển pointer trái phải |
| Prefix sum    | [303. Truy vấn tổng theo khoảng - array bất biến](https://leetcode.cn/problems/range-sum-query-immutable/)    | [560. Subarray có tổng bằng K](https://leetcode.cn/problems/subarray-sum-equals-k/)                                                                          | Điểm bắt đầu của bài subarray       | ý nghĩa prefix sum, đếm bằng hash table                 |

## Linked list

| Dạng bài          | Bài nhất định phải làm                                                                                        | Bài nâng cao                                                                                     | Giá trị phỏng vấn              | Trọng tâm ôn tập                       |
| ----------------- | ------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ | ------------------------------ | -------------------------------------- |
| Thao tác cơ bản   | [707. Thiết kế linked list](https://leetcode.cn/problems/design-linked-list/)                                 | [24. Đổi chỗ từng cặp node trong linked list](https://leetcode.cn/problems/swap-nodes-in-pairs/) | Kiểm tra kỹ năng thao tác node | dummy head, thứ tự chèn và xóa         |
| Đảo linked list   | [206. Đảo linked list](https://leetcode.cn/problems/reverse-linked-list/)                                     | [92. Đảo linked list II](https://leetcode.cn/problems/reverse-linked-list-ii/)                   | Bài thường gặp khi viết tay    | thứ tự cập nhật `prev`, `cur`, `next`  |
| Fast-slow pointer | [141. Linked list có chu kỳ](https://leetcode.cn/problems/linked-list-cycle/)                                 | [142. Linked list có chu kỳ II](https://leetcode.cn/problems/linked-list-cycle-ii/)              | Câu hỏi thường được hỏi thêm   | suy ra điểm gặp nhau và điểm vào cycle |
| Xóa node          | [19. Xóa node thứ N tính từ cuối linked list](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/) | [61. Xoay linked list](https://leetcode.cn/problems/rotate-list/)                                | Kiểm tra xử lý biên            | độ dài linked list, xóa head node      |

## Stack và queue

| Dạng bài          | Bài nhất định phải làm                                                                           | Bài nâng cao                                                                                                        | Giá trị phỏng vấn                                 | Trọng tâm ôn tập                                                |
| ----------------- | ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- | --------------------------------------------------------------- |
| Mô phỏng cấu trúc | [232. Dùng stack triển khai queue](https://leetcode.cn/problems/implement-queue-using-stacks/)   | [225. Dùng queue triển khai stack](https://leetcode.cn/problems/implement-stack-using-queues/)                      | Kiểm tra hiểu biết về cấu trúc                    | vai trò của input stack và output stack                         |
| Kiểm tra ngoặc    | [20. Ngoặc hợp lệ](https://leetcode.cn/problems/valid-parentheses/)                              | [394. Decode string](https://leetcode.cn/problems/decode-string/)                                                   | Bài mở đầu về string stack                        | khi nào push, khi nào pop                                       |
| Monotonic stack   | [739. Nhiệt độ hằng ngày](https://leetcode.cn/problems/daily-temperatures/)                      | [84. Hình chữ nhật lớn nhất trong histogram](https://leetcode.cn/problems/largest-rectangle-in-histogram/)          | Dạng bài có tần suất xuất hiện trung bình đến cao | stack duy trì thứ tự tăng hay giảm                              |
| Monotonic queue   | [239. Giá trị lớn nhất của sliding window](https://leetcode.cn/problems/sliding-window-maximum/) | [862. Subarray ngắn nhất có tổng ít nhất là K](https://leetcode.cn/problems/shortest-subarray-with-sum-at-least-k/) | Template thường gặp cho bài Hard                  | phần tử ở đầu queue hết hạn, duy trì tính đơn điệu ở cuối queue |

## Hash table

| Dạng bài          | Bài nhất định phải làm                                                              | Bài nâng cao                                                                                       | Giá trị phỏng vấn                | Trọng tâm ôn tập                                           |
| ----------------- | ----------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- | -------------------------------- | ---------------------------------------------------------- |
| Tìm nhanh         | [1. Two Sum](https://leetcode.cn/problems/two-sum/)                                 | [49. Nhóm anagram](https://leetcode.cn/problems/group-anagrams/)                                   | Nhập môn hash table              | thiết kế key                                               |
| Đếm               | [242. Anagram hợp lệ](https://leetcode.cn/problems/valid-anagram/)                  | [347. K phần tử có tần suất cao nhất](https://leetcode.cn/problems/top-k-frequent-elements/)       | Bài thống kê tần suất thường gặp | khi nào chọn đếm bằng array, khi nào dùng Map              |
| Prefix sum + hash | [560. Subarray có tổng bằng K](https://leetcode.cn/problems/subarray-sum-equals-k/) | [974. Subarray có tổng chia hết cho K](https://leetcode.cn/problems/subarray-sums-divisible-by-k/) | Dạng bài subarray thường gặp     | kiểm tra trước rồi mới thêm, tránh tính cả prefix hiện tại |
| Cấu trúc cache    | [146. LRU cache](https://leetcode.cn/problems/lru-cache/)                           | [460. LFU cache](https://leetcode.cn/problems/lfu-cache/)                                          | Bài thiết kế viết tay            | cách hash table phối hợp với doubly linked list            |

## Binary tree

| Dạng bài               | Bài nhất định phải làm                                                                                                                                | Bài nâng cao                                                                                                                                            | Giá trị phỏng vấn       | Trọng tâm ôn tập                                |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------- | ----------------------------------------------- |
| Traversal              | [144. Preorder traversal của binary tree](https://leetcode.cn/problems/binary-tree-preorder-traversal/)                                               | [102. Level order traversal của binary tree](https://leetcode.cn/problems/binary-tree-level-order-traversal/)                                           | Nền tảng của bài tree   | biên đệ quy, số tầng khi dùng queue             |
| Bài toán path          | [112. Tổng trên path](https://leetcode.cn/problems/path-sum/)                                                                                         | [124. Tổng path lớn nhất trong binary tree](https://leetcode.cn/problems/binary-tree-maximum-path-sum/)                                                 | DFS thường gặp          | tách giá trị trả về và đáp án toàn cục          |
| Xây dựng tree          | [105. Xây dựng binary tree từ preorder và inorder traversal](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/) | [106. Xây dựng binary tree từ inorder và postorder traversal](https://leetcode.cn/problems/construct-binary-tree-from-inorder-and-postorder-traversal/) | Kiểm tra phạm vi đệ quy | không viết sai phạm vi index                    |
| Lowest common ancestor | [236. Lowest common ancestor của binary tree](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/)                                  | [235. Lowest common ancestor của binary search tree](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-search-tree/)                      | Câu hỏi thường gặp      | khác biệt giữa cách giải của tree thường và BST |

## Graph

| Dạng bài         | Bài nhất định phải làm                                                  | Bài nâng cao                                                                        | Giá trị phỏng vấn           | Trọng tâm ôn tập                  |
| ---------------- | ----------------------------------------------------------------------- | ----------------------------------------------------------------------------------- | --------------------------- | --------------------------------- |
| Grid DFS/BFS     | [200. Số lượng đảo](https://leetcode.cn/problems/number-of-islands/)    | [695. Diện tích lớn nhất của đảo](https://leetcode.cn/problems/max-area-of-island/) | Nhập môn graph search       | xử lý vượt biên, đánh dấu visited |
| Topological sort | [207. Course schedule](https://leetcode.cn/problems/course-schedule/)   | [210. Course schedule II](https://leetcode.cn/problems/course-schedule-ii/)         | Bài về quan hệ phụ thuộc    | indegree array, queue             |
| Shortest path    | [994. Cam bị thối](https://leetcode.cn/problems/rotting-oranges/)       | [127. Word ladder](https://leetcode.cn/problems/word-ladder/)                       | Ứng dụng BFS theo level     | thống kê số bước mỗi level        |
| Connectivity     | [547. Số lượng tỉnh](https://leetcode.cn/problems/number-of-provinces/) | [684. Kết nối dư thừa](https://leetcode.cn/problems/redundant-connection/)          | Điểm bắt đầu của union-find | template `find` và `union`        |

## Heap

| Dạng bài          | Bài nhất định phải làm                                                                              | Bài nâng cao                                                                                              | Giá trị phỏng vấn     | Trọng tâm ôn tập                   |
| ----------------- | --------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- | --------------------- | ---------------------------------- |
| Phần tử lớn thứ K | [215. Phần tử lớn thứ K trong array](https://leetcode.cn/problems/kth-largest-element-in-an-array/) | [703. Phần tử lớn thứ K trong data stream](https://leetcode.cn/problems/kth-largest-element-in-a-stream/) | Top K thường gặp      | duy trì kích thước min heap bằng K |
| Thống kê tần suất | [347. K phần tử có tần suất cao nhất](https://leetcode.cn/problems/top-k-frequent-elements/)        | [692. K từ có tần suất cao nhất](https://leetcode.cn/problems/top-k-frequent-words/)                      | Hash table + heap     | cách viết comparator               |
| Hai heap          | [295. Median trong data stream](https://leetcode.cn/problems/find-median-from-data-stream/)         | [480. Median trong sliding window](https://leetcode.cn/problems/sliding-window-median/)                   | Bài thiết kế nâng cao | cân bằng max heap và min heap      |

## Trie và union-find

| Cấu trúc                   | Bài nhất định phải làm                                                        | Bài nâng cao                                                                                                         | Giá trị phỏng vấn                            | Trọng tâm ôn tập                                     |
| -------------------------- | ----------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | ---------------------------------------------------- |
| Trie                       | [208. Cài đặt Trie](https://leetcode.cn/problems/implement-trie-prefix-tree/) | [211. Thêm và tìm kiếm từ](https://leetcode.cn/problems/design-add-and-search-words-data-structure/)                 | Bài về tập hợp các string                    | cấu trúc node, đánh dấu kết thúc                     |
| Trie + DFS                 | [212. Word search II](https://leetcode.cn/problems/word-search-ii/)           | [648. Thay thế từ](https://leetcode.cn/problems/replace-words/)                                                      | Bài có tần suất xuất hiện trung bình đến cao | cắt nhánh theo prefix                                |
| Union-find                 | [547. Số lượng tỉnh](https://leetcode.cn/problems/number-of-provinces/)       | [1319. Số thao tác để kết nối network](https://leetcode.cn/problems/number-of-operations-to-make-network-connected/) | Template connectivity                        | path compression                                     |
| Union-find phát hiện cycle | [684. Kết nối dư thừa](https://leetcode.cn/problems/redundant-connection/)    | [990. Tính thỏa mãn của phương trình bằng nhau](https://leetcode.cn/problems/satisfiability-of-equality-equations/)  | Biến thể thường gặp của bài graph            | gộp các phương trình trước, sau đó kiểm tra xung đột |

## Điểm bắt đầu của lộ trình ôn tập

Bài viết này chỉ giữ lại danh sách bài liên quan đến cấu trúc dữ liệu. Lộ trình ôn tập 7 ngày và 30 ngày được duy trì thống nhất trong [tổng quan ôn tập cấu trúc dữ liệu](../data-structure/README.md), tránh lặp lại cùng một kế hoạch giữa bài danh sách và trang tổng quan.

<!-- @include: @article-footer.snippet.md -->
