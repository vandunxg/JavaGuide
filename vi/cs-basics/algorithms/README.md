---
title: "Chuyên đề thuật toán: lộ trình luyện bài phỏng vấn, template cốt lõi và các bài LeetCode thường gặp"
description: "Lộ trình ôn tập phỏng vấn thuật toán, bao quát phân tích độ phức tạp, binary search, two pointers, sliding window, DFS/BFS, backtracking, dynamic programming, greedy, Top K, string, linked list, sorting và các bài LeetCode thường gặp."
category: Computer Basics
tag:
  - Algorithms
  - LeetCode
  - Interview
sidebar: false
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: algorithms,algorithm interview questions,LeetCode,problem-solving roadmap,binary search,two pointers,sliding window,DFS,BFS,backtracking,dynamic programming,greedy,TopK,sorting algorithms,string algorithms,linked list algorithms,backend interview
---

**Chuyên đề thuật toán** này không sắp xếp kiến thức theo thứ tự giáo trình mà được tổng hợp theo lộ trình luyện bài phỏng vấn thực tế: trước tiên làm rõ độ phức tạp, sau đó nắm các template thường gặp như binary search, two pointers, sliding window, DFS/BFS, backtracking, dynamic programming, greedy, Top K, cuối cùng dùng các bài về string, linked list, sorting và danh sách bài LeetCode để ôn tập.

Càng luyện bài thuật toán về sau, bạn càng dễ rơi vào tình trạng: đã làm không ít bài nhưng chỉ cần thay đổi điều kiện là bị mắc kẹt. Nguyên nhân thường không phải do làm chưa đủ nhiều bài, mà là chưa quy bài toán về đúng template. Điều thực sự hữu ích trong phỏng vấn là: nhìn đề và nhận ra nó thuộc dạng bài nào, trước tiên viết được phiên bản chạy được, sau đó giải thích độ phức tạp và cách xử lý trường hợp biên.

## Dành cho ai

- Người đang chuẩn bị cho các bài thuật toán để ứng tuyển khi mới tốt nghiệp hoặc khi đã đi làm, muốn luyện LeetCode có hệ thống theo từng dạng bài.
- Độc giả đã làm một số bài nhưng khi ôn tập lại không giải thích rõ được “vì sao bài này phải làm như vậy”.
- Backend developer có nền tảng data structure khá tốt nhưng thiếu kinh nghiệm về template thuật toán và xử lý trường hợp biên.
- Kỹ sư chỉ còn 7 đến 30 ngày trước phỏng vấn và cần nhanh chóng lấy lại phong độ giải bài.

## Phỏng vấn thuật toán kiểm tra gì

Phỏng vấn thuật toán thường không chỉ xem bạn có thể AC một bài hay không, mà chủ yếu đánh giá 4 điều:

| Tiêu chí đánh giá     | Biểu hiện cụ thể trong phỏng vấn                                               | Cần làm gì khi ôn tập                                   |
| --------------------- | ------------------------------------------------------------------------------ | ------------------------------------------------------- |
| Nhận diện dạng bài    | Bài này là binary search, sliding window, backtracking hay DP                  | Luyện theo dạng bài, không hoàn toàn luyện ngẫu nhiên   |
| Tính ổn định của code | Trường hợp biên, null pointer, index, điều kiện vòng lặp có đáng tin cậy không | Chuẩn bị 2 đến 3 ví dụ trường hợp biên cho mỗi template |
| Diễn đạt độ phức tạp  | Có thể nói rõ time complexity và space complexity không                        | Viết độ phức tạp sau mỗi bài                            |
| Khả năng chuyển đổi   | Có thể điều chỉnh template khi điều kiện thay đổi không                        | Mỗi dạng bài cần luyện cả bài cơ bản và bài biến thể    |

Nếu chỉ có thể ghi nhớ một câu: **trước tiên xây dựng template theo dạng bài, sau đó dùng các bài tiêu biểu để luyện khả năng chuyển đổi.**

## Thứ tự đọc đề xuất

1. [Hướng dẫn phỏng vấn về time complexity và space complexity](./complexity-analysis.md): trước tiên làm rõ Big O, độ phức tạp đệ quy và các nhầm lẫn thường gặp.
2. [Tổng hợp câu hỏi phỏng vấn về binary search](./binary-search.md): luyện binary search cơ bản, biên trái, biên phải và binary search trên đáp án.
3. [Tổng hợp câu hỏi phỏng vấn về two pointers và sliding window](./two-pointers-and-sliding-window.md): giải quyết các bài thường gặp về array, string và linked list.
4. [Tổng hợp câu hỏi phỏng vấn về DFS và BFS](./dfs-bfs.md): nắm tree, graph, tìm kiếm matrix và duyệt theo level.
5. [Tổng hợp câu hỏi phỏng vấn về backtracking](./backtracking.md): tập trung xử lý các bài về combination, permutation, subset và bàn cờ.
6. [Tổng hợp câu hỏi phỏng vấn về dynamic programming](./dynamic-programming.md): bắt đầu từ state definition và transition equation, không học thuộc bài.
7. [Tổng hợp câu hỏi phỏng vấn về greedy](./greedy.md) và [tổng hợp câu hỏi phỏng vấn về Top K](./top-k.md): bổ sung sorting, greedy, heap, partition của quicksort và bucket counting.
8. [Một số bài thuật toán chuỗi thường gặp](./string-algorithm-problems.md), [Một số bài thuật toán linked list thường gặp](./linkedlist-algorithm-problems.md), [Tổng hợp 10 thuật toán sorting kinh điển](./10-classical-sorting-algorithms.md): ôn tập trước phỏng vấn theo từng chuyên đề.

## Template cốt lõi

| Template            | Dấu hiệu nhận biết                                                                          | Bài viết trọng tâm                                                                                   |
| ------------------- | ------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Binary search       | Sorted, monotonic, giá trị khả thi nhỏ nhất, giá trị khả thi lớn nhất                       | [Tổng hợp câu hỏi phỏng vấn về binary search](./binary-search.md)                                    |
| Two pointers        | In-place modification, thu hẹp từ hai đầu, fast/slow pointer đuổi nhau, định vị linked list | [Tổng hợp câu hỏi phỏng vấn về two pointers và sliding window](./two-pointers-and-sliding-window.md) |
| Sliding window      | Subarray liên tiếp, substring liên tiếp, window dài nhất/ngắn nhất                          | [Tổng hợp câu hỏi phỏng vấn về two pointers và sliding window](./two-pointers-and-sliding-window.md) |
| DFS/BFS             | Duyệt tree, duyệt graph, thành phần liên thông của matrix, số bước ngắn nhất theo level     | [Tổng hợp câu hỏi phỏng vấn về DFS và BFS](./dfs-bfs.md)                                             |
| Backtracking        | Liệt kê mọi phương án, lựa chọn path, combination và permutation, ràng buộc bàn cờ          | [Tổng hợp câu hỏi phỏng vấn về backtracking](./backtracking.md)                                      |
| Dynamic programming | Giá trị tối ưu, đếm, có thể đạt đến hay không, subsequence, knapsack                        | [Tổng hợp câu hỏi phỏng vấn về dynamic programming](./dynamic-programming.md)                        |
| Greedy              | Ở mỗi bước chọn đối tượng phù hợp nhất hiện tại, thường kết hợp với sorting                 | [Tổng hợp câu hỏi phỏng vấn về greedy](./greedy.md)                                                  |
| Top K               | Lớn thứ K, K phần tử có tần suất cao nhất, data stream, priority                            | [Tổng hợp câu hỏi phỏng vấn về Top K](./top-k.md)                                                    |

## Lộ trình luyện nhanh trong 7 ngày

Khi thời gian rất gấp, không nên bắt đầu từ bài khó. Mục tiêu của lộ trình 7 ngày là củng cố template và khả năng tự viết code ổn định:

| Ngày   | Trọng tâm                    | Việc nên làm                                                                                              |
| ------ | ---------------------------- | --------------------------------------------------------------------------------------------------------- |
| Ngày 1 | Độ phức tạp + sorting        | Ôn tập Big O, quicksort, merge sort, heap sort và tính ổn định                                            |
| Ngày 2 | Binary search + two pointers | Viết template biên trái, biên phải, Two Sum, Three Sum, xóa phần tử trùng                                 |
| Ngày 3 | Sliding window + string      | Viết các bài về substring dài nhất không lặp, substring bao phủ nhỏ nhất và palindrome                    |
| Ngày 4 | Linked list                  | Viết reverse linked list, linked list có cycle, xóa node thứ N tính từ cuối                               |
| Ngày 5 | Tree và BFS                  | Viết duyệt preorder, inorder, postorder, duyệt theo level, lowest common ancestor                         |
| Ngày 6 | Backtracking + DP            | Viết subset, combination, coin change, longest increasing subsequence                                     |
| Ngày 7 | Top K + ôn tập               | Viết bài phần tử lớn thứ K, K phần tử có tần suất cao nhất, tổng hợp các bài sai và ví dụ trường hợp biên |

## Lộ trình hệ thống trong 30 ngày

Lộ trình 30 ngày không yêu cầu mỗi ngày phải làm thật nhiều bài. Nhịp độ phù hợp hơn là: mỗi ngày 1 đến 3 bài tiêu biểu, sau mỗi bài viết 5 dòng ôn tập.

| Giai đoạn   | Thời gian      | Mục tiêu                                                                                   |
| ----------- | -------------- | ------------------------------------------------------------------------------------------ |
| Giai đoạn 1 | Ngày 1 đến 5   | Độ phức tạp, array, linked list, stack, queue, đảm bảo có thể tự viết các template cơ bản  |
| Giai đoạn 2 | Ngày 6 đến 12  | Binary search, two pointers, sliding window, string, tập trung luyện trường hợp biên       |
| Giai đoạn 3 | Ngày 13 đến 18 | Tree, graph, DFS/BFS, union-find, xây dựng framework cho bài tìm kiếm                      |
| Giai đoạn 4 | Ngày 19 đến 24 | Backtracking, dynamic programming, greedy, tập trung luyện state definition và pruning     |
| Giai đoạn 5 | Ngày 25 đến 30 | Top K, sorting, bài tổng hợp và ôn tập các bài sai, chuẩn bị phần giải thích khi phỏng vấn |

## Tự kiểm tra các câu hỏi thường gặp

- Vì sao time complexity phải xét số hạng bậc cao nhất? Tính độ phức tạp đệ quy như thế nào?
- Chọn `left < right` hay `left <= right` trong binary search như thế nào?
- Two pointers và sliding window khác nhau thế nào?
- DFS và BFS lần lượt phù hợp với vấn đề nào? Khi nào cần `visited`?
- Backtracking có quan hệ gì với DFS? Nên đặt pruning ở đâu?
- Vì sao dynamic programming khó? Xác định state definition và thứ tự duyệt như thế nào?
- Vì sao greedy cần được chứng minh? Trong phỏng vấn, trả lời đến mức nào là đủ?
- Với Top K, nên chọn heap, partition của quicksort hay bucket counting?
- Tính ổn định, in-place sorting, độ phức tạp best-case/worst-case của sorting algorithm lần lượt là gì?

## Chuyên đề liên quan

- [Hệ thống kiến thức Computer Basics](../)
- [Chuyên đề về data structure](../data-structure/)
- [Đề xuất bài LeetCode kinh điển về những data structure thường gặp](./common-data-structures-leetcode-recommendations.md)
- [Tổng hợp các tư tưởng thuật toán kinh điển](./classical-algorithm-problems-recommendations.md)
- [Java Collection](../../java/collection/java-collection-questions-01.md)
- [Chuẩn bị phỏng vấn](../../interview-preparation/)
- [Đề xuất sách về Computer Basics](../../books/cs-basics.md)

<!-- @include: @article-footer.snippet.md -->
