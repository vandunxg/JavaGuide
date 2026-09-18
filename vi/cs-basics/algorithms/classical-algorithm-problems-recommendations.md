---
title: Tổng hợp tư duy thuật toán kinh điển (kèm đề xuất bài LeetCode)
description: "Tổng hợp các tư duy thuật toán thường gặp như binary search, two pointers, sliding window, DFS/BFS, backtracking, dynamic programming, greedy, divide and conquer, topological sort, union-find, bit manipulation, đồng thời đưa ra dấu hiệu nhận diện dạng bài, template, bài tiêu biểu và trọng tâm ôn tập."
category: Computer Basics
tag:
  - Algorithms
  - LeetCode
  - Interview
head:
  - - meta
    - name: keywords
      content: algorithm ideas,binary search,two pointers,sliding window,DFS,BFS,backtracking,dynamic programming,greedy,divide and conquer,topological sort,union-find,bit manipulation,LeetCode problem recommendations
---

Đừng học thuộc tư duy thuật toán một cách cô lập. Cách hỏi hữu ích hơn trong phỏng vấn là: tín hiệu nào cho thấy nên dùng nó? Điểm nào trong template dễ viết sai nhất? Nếu interviewer thay đổi điều kiện, tôi nên bắt đầu điều chỉnh từ biến hoặc state nào?

Danh sách bài này được tổ chức theo tư duy. Mỗi nhóm đều đưa ra “tín hiệu nhận diện, template thường dùng, bài tiêu biểu, trọng tâm ôn tập”. Số lượng bài được giới hạn ở mức có thể đại diện cho template; hiểu rõ những bài này trước sẽ hiệu quả hơn việc máy móc làm thêm nhiều bài.

## Cách sử dụng danh sách bài

Đừng bắt đầu bằng việc làm hết tất cả bài theo thứ tự. Cách phù hợp hơn cho việc chuẩn bị phỏng vấn là: trước tiên đọc bài viết về template tương ứng, xác nhận bạn có thể tự viết code cốt lõi, sau đó làm “bài bắt buộc”, cuối cùng dùng “bài nâng cao” để kiểm tra trường hợp biên và các biến thể.

| Mục tiêu                          | Hành động đề xuất                                                                                                                                                                             |
| --------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Nhanh chóng xây dựng template     | Trước tiên đọc các bài viết về template thường gặp như [binary search](./binary-search.md), [two pointers và sliding window](./two-pointers-and-sliding-window.md), [DFS/BFS](./dfs-bfs.md)   |
| Bổ sung search và DP              | Tiếp tục đọc [backtracking](./backtracking.md), [dynamic programming](./dynamic-programming.md), tự viết ít nhất 2 bài cơ bản cho mỗi nhóm                                                    |
| Bổ sung thiếu sót trước phỏng vấn | Dùng [greedy](./greedy.md), [bài toán Top K](./top-k.md), [union-find](../data-structure/union-find.md) để bổ sung các biến thể thường gặp                                                    |
| Ôn lại đáp án của bản thân        | Với mỗi bài, ghi lại tín hiệu nhận diện dạng bài, ý nghĩa của biến cốt lõi, độ phức tạp và ví dụ trường hợp biên. Nếu không giải thích rõ được, nghĩa là bạn vẫn chưa thực sự nắm vững bài đó |

## Binary search

| Hạng mục             | Nội dung                                                                                                                                                                                                                          |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Tín hiệu nhận diện   | Array có thứ tự, điều kiện đơn điệu, tìm biên, tìm giá trị khả thi nhỏ nhất hoặc giá trị khả thi lớn nhất                                                                                                                         |
| Template thường dùng | Binary search cơ bản, biên trái, biên phải, binary search trên đáp án                                                                                                                                                             |
| Bài bắt buộc         | [704. Binary search](https://leetcode.cn/problems/binary-search/), [34. Tìm phần tử ở vị trí đầu tiên và cuối cùng trong array đã sắp xếp](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/) |
| Bài nâng cao         | [35. Tìm vị trí chèn](https://leetcode.cn/problems/search-insert-position/), [875. Koko ăn chuối](https://leetcode.cn/problems/koko-eating-bananas/)                                                                              |
| Trọng tâm ôn tập     | Điều kiện vòng lặp, cách tính `mid`, sau khi cập nhật biên có rơi vào vòng lặp vô hạn hay không                                                                                                                                   |

## Two pointers

| Hạng mục             | Nội dung                                                                                                                                                                                                            |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Tín hiệu nhận diện   | Array có thứ tự, sửa tại chỗ, thu hẹp từ hai đầu vào giữa, fast và slow trong linked list                                                                                                                           |
| Template thường dùng | Two pointers trái phải, fast và slow pointers, read và write pointers                                                                                                                                               |
| Bài bắt buộc         | [26. Xóa phần tử trùng trong array đã sắp xếp](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/), [977. Bình phương của array đã sắp xếp](https://leetcode.cn/problems/squares-of-a-sorted-array/) |
| Bài nâng cao         | [15. Tổng của ba số](https://leetcode.cn/problems/3sum/), [142. Linked list vòng II](https://leetcode.cn/problems/linked-list-cycle-ii/)                                                                            |
| Trọng tâm ôn tập     | Ý nghĩa của pointer phải cố định, không được bỏ sót điều kiện loại trùng, với bài linked list hãy vẽ trước 3 node                                                                                                   |

## Sliding window

| Hạng mục             | Nội dung                                                                                                                                                                                                                |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Tín hiệu nhận diện   | Subarray liên tiếp, substring liên tiếp, dài nhất/ngắn nhất, window thỏa mãn một điều kiện nào đó                                                                                                                       |
| Template thường dùng | Window cố định, window biến đổi, counting Map                                                                                                                                                                           |
| Bài bắt buộc         | [3. Substring dài nhất không có ký tự trùng](https://leetcode.cn/problems/longest-substring-without-repeating-characters/), [209. Subarray có độ dài nhỏ nhất](https://leetcode.cn/problems/minimum-size-subarray-sum/) |
| Bài nâng cao         | [76. Substring bao phủ nhỏ nhất](https://leetcode.cn/problems/minimum-window-substring/), [438. Tìm tất cả anagram trong string](https://leetcode.cn/problems/find-all-anagrams-in-a-string/)                           |
| Trọng tâm ôn tập     | Khi nào mở rộng biên phải, khi nào thu hẹp biên trái, duy trì các biến bên trong window như thế nào                                                                                                                     |

## DFS và BFS

| Hạng mục             | Nội dung                                                                                                                                                                     |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Tín hiệu nhận diện   | Duyệt tree, duyệt graph, các thành phần liên thông trong matrix, số bước ngắn nhất, duyệt theo level                                                                         |
| Template thường dùng | DFS đệ quy, DFS mô phỏng bằng stack, BFS bằng queue, BFS theo level                                                                                                          |
| Bài bắt buộc         | [102. Duyệt tree nhị phân theo level](https://leetcode.cn/problems/binary-tree-level-order-traversal/), [200. Số lượng đảo](https://leetcode.cn/problems/number-of-islands/) |
| Bài nâng cao         | [994. Cam thối](https://leetcode.cn/problems/rotting-oranges/), [127. Word ladder](https://leetcode.cn/problems/word-ladder/)                                                |
| Trọng tâm ôn tập     | Đánh dấu đã truy cập, kiểm tra vượt biên, thống kê số level của BFS                                                                                                          |

## Backtracking

| Hạng mục             | Nội dung                                                                                                                        |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Tín hiệu nhận diện   | Liệt kê mọi phương án, lựa chọn path, combination, permutation, subset, ràng buộc bàn cờ                                        |
| Template thường dùng | `path`, danh sách lựa chọn, level đệ quy, hoàn tác lựa chọn                                                                     |
| Bài bắt buộc         | [77. Combination](https://leetcode.cn/problems/combinations/), [78. Subset](https://leetcode.cn/problems/subsets/)              |
| Bài nâng cao         | [39. Tổng combination](https://leetcode.cn/problems/combination-sum/), [51. N quân hậu](https://leetcode.cn/problems/n-queens/) |
| Trọng tâm ôn tập     | Tham số đệ quy đại diện cho điều gì, điều kiện pruning nên đặt trước vòng lặp hay bên trong vòng lặp                            |

## Dynamic programming

| Hạng mục             | Nội dung                                                                                                                                                                                                 |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Tín hiệu nhận diện   | Tìm giá trị tối ưu, số phương án, có thể đạt tới hay không, subsequence, knapsack, gộp interval                                                                                                          |
| Template thường dùng | DP một chiều, DP hai chiều, rolling array, knapsack DP                                                                                                                                                   |
| Bài bắt buộc         | [70. Leo cầu thang](https://leetcode.cn/problems/climbing-stairs/), [322. Đổi tiền](https://leetcode.cn/problems/coin-change/)                                                                           |
| Bài nâng cao         | [300. Subsequence tăng dài nhất](https://leetcode.cn/problems/longest-increasing-subsequence/), [416. Chia thành các subset có tổng bằng nhau](https://leetcode.cn/problems/partition-equal-subset-sum/) |
| Trọng tâm ôn tập     | Ý nghĩa của `dp[i]`, khởi tạo, thứ tự duyệt, có thể nén không gian hay không                                                                                                                             |

## Greedy

| Hạng mục             | Nội dung                                                                                                                                                 |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Tín hiệu nhận diện   | Mỗi bước chọn đối tượng phù hợp nhất hiện tại, thường đi kèm sorting, interval, jump, mua bán                                                            |
| Template thường dùng | Chọn sau khi sorting, duy trì biên xa nhất, gộp/phủ interval                                                                                             |
| Bài bắt buộc         | [455. Phân phát bánh quy](https://leetcode.cn/problems/assign-cookies/), [55. Jump game](https://leetcode.cn/problems/jump-game/)                        |
| Bài nâng cao         | [45. Jump game II](https://leetcode.cn/problems/jump-game-ii/), [435. Interval không giao nhau](https://leetcode.cn/problems/non-overlapping-intervals/) |
| Trọng tâm ôn tập     | Vì sao greedy không sai, phản ví dụ có thể bác bỏ strategy hiện tại hay không                                                                            |

## Divide and conquer

| Hạng mục             | Nội dung                                                                                                                                                                                               |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Tín hiệu nhận diện   | Bài toán có thể tách thành các subproblem cùng loại, kết quả của subproblem có thể được gộp lại                                                                                                        |
| Template thường dùng | Tách đệ quy, giải subproblem, gộp kết quả                                                                                                                                                              |
| Bài bắt buộc         | [108. Chuyển array đã sắp xếp thành binary search tree](https://leetcode.cn/problems/convert-sorted-array-to-binary-search-tree/), [148. Sorting linked list](https://leetcode.cn/problems/sort-list/) |
| Bài nâng cao         | [23. Gộp K linked list tăng dần](https://leetcode.cn/problems/merge-k-sorted-lists/), [215. Phần tử lớn thứ K trong array](https://leetcode.cn/problems/kth-largest-element-in-an-array/)              |
| Trọng tâm ôn tập     | Điểm kết thúc đệ quy, interval trái phải có chồng lên nhau hay không, độ phức tạp khi gộp                                                                                                              |

## Topological sort

| Hạng mục             | Nội dung                                                                                                                                            |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| Tín hiệu nhận diện   | Dependency giữa các khóa học, dependency giữa các task, directed acyclic graph, xác định có thể hoàn thành hay không                                |
| Template thường dùng | Array indegree + queue, hoặc đánh dấu ba màu bằng DFS                                                                                               |
| Bài bắt buộc         | [207. Course schedule](https://leetcode.cn/problems/course-schedule/)                                                                               |
| Bài nâng cao         | [210. Course schedule II](https://leetcode.cn/problems/course-schedule-ii/), [269. Từ điển sao Hỏa](https://leetcode.cn/problems/alien-dictionary/) |
| Trọng tâm ôn tập     | Khi nào giảm indegree, số lượng kết quả có bằng số node hay không                                                                                   |

## Union-find

| Hạng mục             | Nội dung                                                                                                                                                                                            |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Tín hiệu nhận diện   | Tính liên thông, phân nhóm, network bạn bè, cạnh dư thừa, quan hệ đẳng thức                                                                                                                         |
| Template thường dùng | `find`, `union`, path compression, gộp theo kích thước                                                                                                                                              |
| Bài bắt buộc         | [547. Số lượng tỉnh](https://leetcode.cn/problems/number-of-provinces/)                                                                                                                             |
| Bài nâng cao         | [684. Kết nối dư thừa](https://leetcode.cn/problems/redundant-connection/), [990. Tính thỏa mãn của các phương trình đẳng thức](https://leetcode.cn/problems/satisfiability-of-equality-equations/) |
| Trọng tâm ôn tập     | `find` có path compression hay không, khi nào kiểm tra xung đột                                                                                                                                     |

## Bit manipulation

| Hạng mục             | Nội dung                                                                                                                                        |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| Tín hiệu nhận diện   | Chẵn lẻ, có phải lũy thừa của 2 hay không, chỉ xuất hiện một lần, nén state                                                                     |
| Template thường dùng | XOR, phép AND để xóa bit 1 thấp nhất, liệt kê bằng bitmask                                                                                      |
| Bài bắt buộc         | [136. Số chỉ xuất hiện một lần](https://leetcode.cn/problems/single-number/), [231. Lũy thừa của 2](https://leetcode.cn/problems/power-of-two/) |
| Bài nâng cao         | [191. Số lượng bit 1](https://leetcode.cn/problems/number-of-1-bits/), [78. Subset](https://leetcode.cn/problems/subsets/)                      |
| Trọng tâm ôn tập     | Tính chất của XOR, ý nghĩa của `n & (n - 1)`, biểu diễn bit của số âm                                                                           |

## Mục lục ôn tập

Bài viết này chỉ giữ lại các dạng bài kinh điển và đề xuất danh sách bài. Lộ trình luyện nhanh 7 ngày và lộ trình hệ thống 30 ngày được duy trì thống nhất trong [tổng quan ôn tập phỏng vấn thuật toán](./README.md). Nếu sau này điều chỉnh nhịp độ ôn tập, chỉ cần cập nhật trang tổng quan để tránh bảng lộ trình trong nhiều danh sách bài bị lệch nhau.

<!-- @include: @article-footer.snippet.md -->
