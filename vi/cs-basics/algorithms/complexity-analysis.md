---
title: "Hướng dẫn phỏng vấn về time complexity và space complexity: Big O, recursion complexity và các ngộ nhận thường gặp"
description: Hướng dẫn phỏng vấn về time complexity và space complexity, giải thích có hệ thống về Big O, loop complexity, recursion complexity, space complexity, cách đánh giá input scale và các ngộ nhận thường gặp về complexity trong phỏng vấn thuật toán.
category: Computer Basics
tag:
  - Algorithms
head:
  - - meta
    - name: keywords
      content: time complexity,space complexity,Big O,recursion complexity,loop complexity,algorithm complexity,complexity analysis,algorithm interview questions,LeetCode complexity
---

Complexity analysis là cánh cửa đầu tiên của phỏng vấn thuật toán. Interviewer không nhất thiết yêu cầu bạn viết proof thật chặt chẽ, nhưng sẽ muốn bạn nói rõ: đoạn code này chạy bao nhiêu lượt, dùng thêm bao nhiêu space, input scale tăng lên thì điều gì xảy ra.

Trước hết cần làm rõ một ranh giới: complexity analysis thường xét xu hướng tăng trưởng khi input scale rất lớn, không phải runtime chính xác. `O(n)` không có nghĩa chắc chắn nhanh hơn `O(nlogn)`, vì constant, data scale, cache hit và chi tiết implementation đều ảnh hưởng đến thời gian thực tế. Tuy nhiên, trong phỏng vấn, trước tiên chỉ cần nói rõ growth order theo Big O, sau đó bổ sung một câu về giới hạn của scenario thực tế là đủ.

## Trọng tâm phỏng vấn

- Có thể dựa vào loop, recursion và thao tác trên data structure để xác định time complexity.
- Có thể phân biệt extra space với space do input tự chiếm dụng.
- Có thể nói rõ best-case, worst-case và average complexity lần lượt phù hợp với những algorithm nào.
- Khi gặp code recursion, có thể dùng recursion tree hoặc phân tích input size của subproblem.
- Không mặc định xem `HashMap`, sorting và thao tác trên heap đều là `O(1)`.

## Nói về complexity trong phỏng vấn như thế nào?

Khi trả lời về complexity, đừng chỉ đưa ra một kết luận. Cách nói tốt hơn là “code đã làm gì, vì vậy complexity là bao nhiêu”.

Ví dụ với Two Sum:

```text
Duyệt array một lần, thực hiện một lần query và một lần insert cho mỗi element trong HashMap, thao tác trên hash table có average là O(1), vì vậy time complexity là O(n). Sử dụng thêm một HashMap để lưu mapping từ element đến index, worst-case sẽ lưu n element, vì vậy space complexity là O(n).
```

Câu trả lời này chắc chắn hơn việc chỉ nói `O(n)`, vì đã trình bày cả quá trình suy luận. Nếu interviewer hỏi tiếp về worst-case của hash table, bạn cũng có cơ sở để trả lời.

## Các mức complexity thường gặp

| Complexity | Scenario thường gặp                                                                   | Ghi chú phỏng vấn                                   |
| ---------- | ------------------------------------------------------------------------------------- | --------------------------------------------------- |
| `O(1)`     | Truy cập array theo index, thao tác trên đỉnh stack, query trung bình trên hash table | Worst-case của hash table có thể bị suy biến        |
| `O(logn)`  | Binary search, heapify up/down, query trên balanced tree                              | Mỗi lượt thu nhỏ một phần input scale               |
| `O(n)`     | Duyệt array, linked list hoặc string một lần                                          | Xem có thật sự chỉ quét một lần hay không           |
| `O(nlogn)` | Quicksort average, merge sort, heap sort                                              | Mức thường gặp nhất trong bài sorting               |
| `O(n^2)`   | Nested loop, liệt kê từng cặp                                                         | Cần cảnh giác xem có thể optimize hay không         |
| `O(2^n)`   | Liệt kê subset, một số bài backtracking                                               | Search space của việc liệt kê subset là exponential |
| `O(n!)`    | Permutation đầy đủ, brute force cho traveling salesman                                | Chỉ phù hợp với input scale nhỏ                     |

Thông thường, input scale của bài thuật toán sẽ gợi ý complexity có thể chấp nhận:

| Input scale | Complexity thường có thể chấp nhận           |
| ----------- | -------------------------------------------- |
| `n <= 20`   | Exponential, backtracking, state compression |
| `n <= 100`  | Đôi khi có thể dùng `O(n^3)`                 |
| `n <= 1000` | Thường dùng `O(n^2)`                         |
| `n <= 10^5` | `O(nlogn)` hoặc `O(n)`                       |
| `n >= 10^6` | Thông thường cần gần với `O(n)`              |

Đây không phải quy tắc cứng, nhưng có thể giúp bạn phán đoán trong phỏng vấn xem brute-force solution có thể bị timeout hay không.

## Xác định loop complexity như thế nào?

Với loop thông thường, hãy xem số lần thực thi:

```java
for (int i = 0; i < n; i++) {
    // O(1)
}
```

Đoạn này là `O(n)`.

Với nested loop, không thể chỉ nhìn vào số tầng, mà cần xem số lần thực tế của mỗi tầng:

```java
for (int i = 0; i < n; i++) {
    for (int j = i; j < n; j++) {
        // O(1)
    }
}
```

Số lần chạy của inner loop là `n + (n - 1) + ... + 1`, tức `n(n + 1) / 2`, nên complexity được ghi là `O(n^2)`.

Nếu loop variable tăng gấp đôi sau mỗi lần, thông thường đó là `O(logn)`:

```java
for (int i = 1; i < n; i *= 2) {
    // O(1)
}
```

Một trường hợp khác rất dễ đánh giá nhầm là two pointers:

```java
while (left < n && right < n) {
    if (needMoveRight()) {
        right++;
    } else {
        left++;
    }
}
```

Mặc dù trong `while` có lồng điều kiện, nhưng `left` và `right` đều chỉ tăng đơn điệu, nhiều nhất mỗi biến di chuyển `n` lần, nên complexity tổng thể là `O(n)`, không phải `O(n^2)`.

## Xác định recursion complexity như thế nào?

Khi phân tích recursion complexity, trước tiên có thể xem hai câu hỏi:

1. Mỗi recursion level có bao nhiêu subproblem?
2. Ngoài recursive call, mỗi level còn thực hiện bao nhiêu công việc bổ sung?

Binary search mỗi lần chỉ đi vào một subproblem và scale giảm một nửa:

```java
int binarySearch(int[] nums, int target, int left, int right) {
    if (left > right) {
        return -1;
    }
    int mid = left + (right - left) / 2;
    if (nums[mid] == target) {
        return mid;
    }
    if (nums[mid] < target) {
        return binarySearch(nums, target, mid + 1, right);
    }
    return binarySearch(nums, target, left, mid - 1);
}
```

Recursion depth là `logn`, mỗi level chỉ thực hiện công việc `O(1)`, vì vậy time complexity là `O(logn)`, còn space của recursion stack là `O(logn)`.

Merge sort tách thành hai subproblem ở mỗi level, tổng work của việc merge ở mỗi level là `O(n)`, số level là `logn`, vì vậy time complexity là `O(nlogn)`, còn space của array phụ là `O(n)`.

Hãy xem thêm một counterexample: Fibonacci dùng recursion thông thường.

```java
int fib(int n) {
    if (n <= 1) {
        return n;
    }
    return fib(n - 1) + fib(n - 2);
}
```

Nó không phải `O(n)`, vì mỗi lần lại tiếp tục tách thành hai recursive call, nhiều subproblem bị tính lặp lại, nên time complexity gần với `O(2^n)`. Nếu thêm array memoization, mỗi state chỉ được tính một lần, time complexity sẽ trở thành `O(n)`, space complexity cũng là `O(n)`.

## Xem xét space complexity như thế nào?

Space complexity xét phần space được sử dụng thêm trong quá trình algorithm chạy, các nguồn thường gặp gồm:

- Tạo array, hash table, queue hoặc stack mới.
- Recursion call stack.
- Auxiliary space khi sorting hoặc merging.
- Có tính result set là extra space hay không còn tùy yêu cầu của đề. Trong phỏng vấn, bạn có thể chủ động nói rõ.

Ví dụ, cách viết reverse linked list bằng iteration chỉ dùng vài pointer, space complexity là `O(1)`. Nếu dùng recursion để reverse, dù không tạo array một cách rõ ràng, recursion stack vẫn có depth là `n`, nên space complexity là `O(n)`.

## Các điểm dễ sai thường gặp

- Sorting không miễn phí. Sorting trước rồi dùng two pointers thì time complexity thường ít nhất là `O(nlogn)`.
- Query trên `HashMap` trung bình là `O(1)`, nhưng worst-case thì không phải vậy.
- Recursion dù không tạo collection một cách rõ ràng vẫn có thể sử dụng space cho recursion stack.
- Duyệt matrix hai chiều thường là `O(mn)`, đừng tiện tay viết thành `O(n)`.
- Space của queue trong BFS không phải constant, worst-case có thể lưu rất nhiều node của level tiếp theo.
- Complexity của bài backtracking thường liên quan đến số lượng result, không thể chỉ nhìn vào recursion depth.

## Tự kiểm tra các câu hỏi thường gặp

- Vì sao complexity analysis thường bỏ qua constant?
- `O(n)` chắc chắn nhanh hơn `O(nlogn)` sao?
- Time complexity average và worst-case của quicksort lần lượt là bao nhiêu?
- Tính space complexity của recursive algorithm như thế nào?
- Vì sao time complexity của DFS và BFS thường là `O(V + E)`?
- Vì sao query trên hash table có average là `O(1)`?

## Bài tập đề xuất

- [704. Binary Search](https://leetcode.cn/problems/binary-search/)
- [912. Sort an Array](https://leetcode.cn/problems/sort-an-array/)
- [206. Reverse Linked List](https://leetcode.cn/problems/reverse-linked-list/)
- [200. Number of Islands](https://leetcode.cn/problems/number-of-islands/)

<!-- @include: @article-footer.snippet.md -->
