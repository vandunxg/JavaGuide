---
title: "Tổng hợp câu hỏi phỏng vấn về Greedy Algorithm: Greedy theo interval, Jump Game và cách chứng minh"
description: "Tổng hợp câu hỏi phỏng vấn về Greedy Algorithm, giải thích cách nhận diện dạng bài greedy, greedy bằng sorting, greedy theo interval, Jump Game, cách chứng minh greedy và các bài LeetCode thường gặp."
category: Computer Science Basics
tag:
  - Algorithms
head:
  - - meta
    - name: keywords
      content: Greedy Algorithm, template Greedy Algorithm, greedy theo interval, greedy bằng sorting, Jump Game, chứng minh greedy, LeetCode Greedy Algorithm, câu hỏi phỏng vấn Algorithms
---

Code của Greedy Algorithm thường không dài, điểm khó nằm ở việc giải thích tại sao lựa chọn hiện tại không ảnh hưởng đến nghiệm tối ưu toàn cục. Trong phỏng vấn, nếu chỉ viết code mà không giải thích chiến lược greedy, bạn rất dễ bị hỏi sâu rồi bí.

Bạn có thể ghi nhớ một cách nhận diện: nếu bài toán có thể giải bằng cách sorting hoặc duy trì một boundary tối ưu hiện tại, mỗi bước đưa ra một lựa chọn cục bộ, và lựa chọn đó không phá hỏng nghiệm tối ưu về sau, bạn có thể thử dùng greedy.

## Trọng tâm phỏng vấn

- Tìm ra chiến lược greedy.
- Dùng exchange argument, phản chứng hoặc trực giác về boundary để giải thích tính hợp lý của chiến lược.
- Xử lý điều kiện traversal sau khi sorting.
- Phân biệt greedy và dynamic programming.

## Suy nghĩ về bài greedy như thế nào?

Điều dễ mắc nhất khi làm bài greedy là “chọn theo cảm giác”. Trước khi viết code, ít nhất phải nói rõ hai điều:

1. Mỗi bước greedy điều gì, chẳng hạn thời gian kết thúc sớm nhất, vị trí xa nhất có thể nhảy tới hiện tại, lợi nhuận hiện tại dương.
2. Tại sao lựa chọn này không khiến phần sau trở nên tệ hơn.

Chứng minh không nhất thiết phải quá hình thức, nhưng phải nêu được sự đánh đổi. Ví dụ trong interval scheduling, chọn interval kết thúc sớm nhất vì nó để lại không gian lựa chọn lớn nhất cho phần sau; nếu chọn một interval kết thúc muộn hơn, số lượng đáp án sẽ không tăng.

## Dạng bài thường gặp

| Dạng bài            | Chiến lược greedy                                    | Bài tiêu biểu                                                      |
| ------------------- | ---------------------------------------------------- | ------------------------------------------------------------------ |
| Bài toán phân phối  | Ưu tiên đáp ứng đối tượng dễ đáp ứng nhất            | Phân phát bánh quy                                                 |
| Mua bán cổ phiếu    | Cộng dồn mọi lợi nhuận dương                         | Thời điểm mua bán cổ phiếu tốt nhất II                             |
| Bài toán nhảy       | Duy trì vị trí xa nhất có thể tới hiện tại           | Jump Game                                                          |
| Bài toán interval   | Sorting theo right endpoint hoặc left endpoint       | Non-overlapping Intervals, dùng ít mũi tên nhất để bắn nổ bóng bay |
| Tái cấu trúc string | Duy trì số lần còn dùng được hoặc vị trí phủ xa nhất | Partition Labels                                                   |

Greedy thường xuất hiện cùng sorting, vì sorting giúp lựa chọn tối ưu hiện tại trở nên rõ ràng. Bài toán interval thường sorting theo left endpoint hoặc right endpoint; bài toán phân phối thường sorting cả nhu cầu và tài nguyên, sau đó matching bằng two pointers.

## Template Jump Game

```java
boolean canJump(int[] nums) {
    int farthest = 0;
    for (int i = 0; i < nums.length; i++) {
        if (i > farthest) {
            return false;
        }
        farthest = Math.max(farthest, i + nums[i]);
    }
    return true;
}
```

`farthest` biểu thị vị trí xa nhất hiện tại có thể tới. Khi traversal đến `i`, nếu `i > farthest`, điều đó có nghĩa là vị trí hiện tại hoàn toàn không thể tới được.

Điểm greedy của bài này là: không quan tâm cụ thể bước nào đã nhảy đến `i`, chỉ quan tâm vị trí xa nhất hiện tại có thể phủ tới. Chỉ cần vị trí hiện tại nằm trong phạm vi phủ, ta có thể dùng nó để tiếp tục cập nhật phạm vi phủ.

“Jump Game II” có thêm yêu cầu về số bước ít nhất. Bài này duy trì hai boundary:

- `curEnd`: vị trí xa nhất mà số bước hiện tại có thể phủ tới.
- `farthest`: vị trí xa nhất có thể tới nếu nhảy thêm một bước trong phạm vi phủ hiện tại.

Khi traversal đến `curEnd`, điều đó có nghĩa là phạm vi của số bước hiện tại đã dùng hết, bắt buộc phải nhảy thêm một bước và cập nhật `curEnd` thành `farthest`.

## Template greedy theo interval

Lấy Non-overlapping Intervals làm ví dụ: sorting theo right endpoint tăng dần, mỗi lần giữ lại interval kết thúc sớm nhất:

```java
int eraseOverlapIntervals(int[][] intervals) {
    if (intervals.length == 0) {
        return 0;
    }
    Arrays.sort(intervals, Comparator.comparingInt(a -> a[1]));
    int count = 1;
    int end = intervals[0][1];
    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] >= end) {
            count++;
            end = intervals[i][1];
        }
    }
    return intervals.length - count;
}
```

Kết thúc càng sớm thì không gian dành cho các interval phía sau càng lớn; đây là lựa chọn cốt lõi của dạng bài này.

Bài toán interval dễ sai nhất ở trường sorting. Một số lựa chọn thường gặp:

- Muốn chọn nhiều interval không overlap nhất: sorting theo right endpoint tăng dần.
- Muốn merge interval: sorting theo left endpoint tăng dần.
- Muốn dùng ít mũi tên nhất để bắn nổ bóng bay: sorting theo right endpoint tăng dần, cố gắng dùng mũi tên hiện tại để phủ nhiều bóng bay hơn.

Nếu khó giải thích một chiến lược greedy, trước tiên hãy dùng sample nhỏ để tìm counterexample. Ví dụ “mỗi lần chọn interval có độ dài ngắn nhất” nghe có vẻ hợp lý, nhưng không đảm bảo chọn được nhiều interval không overlap nhất.

## Giải thích chi tiết bài tiêu biểu: Dùng ít mũi tên nhất để bắn nổ bóng bay

[452. Dùng ít mũi tên nhất để bắn nổ bóng bay](https://leetcode.cn/problems/minimum-number-of-arrows-to-burst-balloons/) là bài tiêu biểu về greedy theo interval. Đề bài cho một tập interval của các bóng bay `[start, end]`; một mũi tên bắn tại tọa độ `x`, chỉ cần `start <= x <= end` thì bóng bay sẽ bị bắn nổ, yêu cầu dùng ít mũi tên nhất để bắn nổ tất cả bóng bay.

Điểm greedy của bài này là: **mỗi lần bắn mũi tên tại right boundary của interval có thể chọn hiện tại**. Trước tiên sorting theo right endpoint tăng dần, đặt mũi tên đầu tiên tại right endpoint của bóng bay đầu tiên. Nếu left endpoint của các bóng bay phía sau `<= arrow`, mũi tên này vẫn có thể phủ nó; nếu left endpoint `> arrow`, mũi tên hiện tại không thể với tới nữa, phải thêm một mũi tên và đặt mũi tên mới tại right endpoint của bóng bay đó.

Trong code cần chú ý hai boundary: array rỗng trả về `0`; comparator khi sorting không nên viết thành `a[1] - b[1]`, vì tọa độ cực trị có thể gây overflow.

```java
int findMinArrowShots(int[][] points) {
    if (points.length == 0) {
        return 0;
    }
    Arrays.sort(points, (a, b) -> Integer.compare(a[1], b[1]));
    int arrows = 1;
    int arrow = points[0][1];
    for (int i = 1; i < points.length; i++) {
        if (points[i][0] > arrow) {
            arrows++;
            arrow = points[i][1];
        }
    }
    return arrows;
}
```

Nếu sample là `[[10,16],[2,8],[1,6],[7,12]]`, sau khi sorting theo right endpoint sẽ là `[1,6]`, `[2,8]`, `[7,12]`, `[10,16]`. Mũi tên đầu tiên đặt tại `6`, có thể phủ hai interval đầu; khi gặp `[7,12]`, left endpoint đã lớn hơn `6`, phải thêm một mũi tên và đặt tại `12`, mũi tên này lại có thể phủ `[10,16]`. Đáp án cuối cùng là `2`.

## Phân biệt greedy và dynamic programming như thế nào?

| Điểm so sánh             | Greedy                                           | Dynamic programming                             |
| ------------------------ | ------------------------------------------------ | ----------------------------------------------- |
| Cách quyết định          | Chọn trực tiếp ở bước hiện tại                   | Phụ thuộc vào nhiều state trước đó              |
| Có xem lại lịch sử không | Thường không xem lại                             | Cần state transition                            |
| Trọng tâm chứng minh     | Lựa chọn hiện tại không phá hỏng tối ưu toàn cục | Optimal substructure và overlapping subproblems |
| Bài thường gặp           | Interval, Jump Game, phân phối                   | Knapsack, subsequence, path                     |

Nếu lựa chọn hiện tại có vẻ hợp lý nhưng chỉ cần đưa ra một counterexample nhỏ là sai, bài toán nhiều khả năng cần DP hoặc search.

## Điểm dễ sai

- Bài greedy thường cần sorting trước, sorting sai field thì đáp án sai.
- Với bài toán interval, phải xem boundary có cho phép bằng nhau không, chẳng hạn `[1,2]` và `[2,3]` có overlap hay không.
- Trong Jump Game II, thời điểm “tăng số bước” liên quan đến boundary phủ hiện tại.
- Phải giải thích được chiến lược greedy, không chỉ nói “mỗi lần chọn phương án tối ưu”.

## Tự kiểm tra với các câu hỏi thường gặp

- Phân biệt greedy và dynamic programming như thế nào?
- Tại sao bài toán interval thường sorting theo right endpoint?
- Tại sao trong Jump Game chỉ cần duy trì vị trí xa nhất có thể tới là đủ?
- Làm thế nào dùng exchange argument hoặc phản chứng để chứng minh chiến lược greedy đúng?
- Khi boundary của interval cho phép bằng nhau, nên viết điều kiện thế nào?

## Bài tập đề xuất

- [455. Phân phát bánh quy](https://leetcode.cn/problems/assign-cookies/)
- [122. Thời điểm mua bán cổ phiếu tốt nhất II](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)
- [55. Jump Game](https://leetcode.cn/problems/jump-game/)
- [45. Jump Game II](https://leetcode.cn/problems/jump-game-ii/)
- [435. Non-overlapping Intervals](https://leetcode.cn/problems/non-overlapping-intervals/)
- [763. Partition Labels](https://leetcode.cn/problems/partition-labels/)

<!-- @include: @article-footer.snippet.md -->
