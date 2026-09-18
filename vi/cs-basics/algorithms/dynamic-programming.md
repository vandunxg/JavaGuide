---
title: "Tổng hợp câu hỏi phỏng vấn về dynamic programming: state transition, knapsack, subsequence và Java template"
description: "Tổng hợp câu hỏi phỏng vấn về dynamic programming, giải thích state definition, state transition, initialization, thứ tự duyệt, 0-1 knapsack, unbounded knapsack, subsequence, interval DP và các bài LeetCode thường gặp."
category: Computer Basics
tag:
  - Algorithms
head:
  - - meta
    - name: keywords
      content: dynamic programming,DP,state transition,knapsack problem,0-1 knapsack,unbounded knapsack,subsequence,interval DP,Java dynamic programming,LeetCode dynamic programming
---

Dynamic programming khó không phải vì code nhất định dài, mà vì một khi state definition sai thì transition equation, initialization và thứ tự duyệt phía sau cũng sẽ sai theo.

Trong phỏng vấn, đừng vừa bắt đầu đã học thuộc template. Trước tiên hãy tự hỏi hai câu: bài toán này có thể tách thành các bài toán con không? Đáp án hiện tại có phụ thuộc vào các đáp án đã tính trước đó không? Nếu cả hai câu trả lời đều là có, hãy cân nhắc DP.

## Trọng tâm phỏng vấn

- Có thể nói rõ ý nghĩa của `dp[i]` hoặc `dp[i][j]`.
- Có thể viết transition equation.
- Có thể xử lý initialization và thứ tự duyệt.
- Có thể phán đoán có thể nén space hay không.
- Có thể phân biệt các dạng thường gặp như knapsack, subsequence và interval.

## Khi nào nên cân nhắc dynamic programming?

Không phải cứ thấy bài hỏi “giá trị tối ưu” là áp dụng DP. Cách phán đoán đáng tin cậy hơn là xem hai điều kiện:

1. Bài toán có thể tách thành các bài toán cùng dạng với quy mô nhỏ hơn không.
2. Các bài toán con có bị tính lặp lại không.

Ví dụ, Fibonacci sequence, `f(n)` phụ thuộc vào `f(n - 1)` và `f(n - 2)`, trong đó `f(n - 2)` sẽ bị tính lặp lại trong recursion. Lưu các kết quả trung gian này lại chính là DP.

Trong phỏng vấn, bạn có thể bắt đầu từ brute-force recursion, sau đó nói rõ phần nào bị tính lặp, cuối cùng chuyển recursion thành memoization search hoặc tabulation. Quá trình này giúp interviewer tin rằng bạn thực sự hiểu hơn là chỉ học thuộc mảng `dp`.

## 5 bước làm DP

1. Định nghĩa state: `dp[i]` rốt cuộc biểu thị điều gì.
2. Viết transition: state hiện tại được suy ra từ những state nào.
3. Thực hiện initialization: khi không có state trước đó thì đáp án là gì.
4. Xác định thứ tự duyệt: tính state nào trước, state nào sau.
5. Kiểm tra sample: dùng một input nhỏ để tự tính mảng.

Trong đó bước 1 quan trọng nhất. Một khi ý nghĩa của `dp[i]` mơ hồ, code phía sau sẽ biến thành thứ được thử cho đến khi chạy đúng.

Một state definition tốt thường thỏa mãn:

- Có thể bao quát đáp án mà đề bài hỏi.
- Có thể suy ra từ state nhỏ hơn.
- Số chiều ít nhất có thể, nhưng không được vì tiết kiệm space mà làm rối ý nghĩa.

## Ví dụ DP một chiều

Bài toán Climbing Stairs:

```java
int climbStairs(int n) {
    if (n <= 2) {
        return n;
    }
    int prev2 = 1;
    int prev1 = 2;
    for (int i = 3; i <= n; i++) {
        int cur = prev1 + prev2;
        prev2 = prev1;
        prev1 = cur;
    }
    return prev1;
}
```

Ý nghĩa state: có bao nhiêu cách đi đến bậc `i`. Transition equation: `dp[i] = dp[i - 1] + dp[i - 2]`.

Bài này cũng có thể suy ra từ recursion:

```text
Bước cuối cùng để đến bậc i là đi 1 bước từ i-1, hoặc đi 2 bước từ i-2.
```

Vì `dp[i]` chỉ phụ thuộc vào hai state trước đó, có thể nén mảng thành hai biến. Tiền đề để nén space là bạn phải xác nhận sau này sẽ không dùng lại các state cũ.

## Template 0-1 knapsack

Mỗi item chỉ được chọn một lần:

```java
int knapsack01(int[] weights, int[] values, int capacity) {
    int[] dp = new int[capacity + 1];
    for (int i = 0; i < weights.length; i++) {
        for (int j = capacity; j >= weights[i]; j--) {
            dp[j] = Math.max(dp[j], dp[j - weights[i]] + values[i]);
        }
    }
    return dp[capacity];
}
```

Phải duyệt capacity theo thứ tự giảm dần để tránh dùng lặp cùng một item trong một vòng.

Duyệt theo thứ tự giảm dần là điểm dễ bị hỏi nhất trong 0-1 knapsack. Nếu duyệt capacity theo thứ tự tăng dần, khi tính `dp[j]` có thể dùng `dp[j - weight]` vừa được cập nhật trong vòng hiện tại, tương đương với việc chọn cùng một item nhiều lần. Khi đó bài toán đã trở thành unbounded knapsack.

Cách hỏi điển hình về 0-1 knapsack không nhất thiết gọi trực tiếp là knapsack. Ví dụ, bài “có thể chia thành hai subset có tổng bằng nhau không” có thể chuyển thành: có thể chọn một số phần tử từ array sao cho tổng của chúng bằng một nửa tổng toàn bộ không.

## Template unbounded knapsack

Mỗi item có thể được chọn nhiều lần:

```java
int unboundedKnapsack(int[] weights, int[] values, int capacity) {
    int[] dp = new int[capacity + 1];
    for (int i = 0; i < weights.length; i++) {
        for (int j = weights[i]; j <= capacity; j++) {
            dp[j] = Math.max(dp[j], dp[j - weights[i]] + values[i]);
        }
    }
    return dp[capacity];
}
```

Duyệt capacity theo thứ tự tăng dần để cho phép dùng lại item hiện tại.

Trong unbounded knapsack, việc duyệt capacity theo thứ tự tăng dần chính là để cho phép dùng lại item hiện tại. Ví dụ với Coin Change, mỗi loại coin có thể dùng nhiều lần; khi tính số tiền lớn hơn, có thể tiếp tục transition dựa trên state mà coin hiện tại đã tham gia.

Nếu đề hỏi “số combination” hay “số permutation” thì thứ tự duyệt cũng thay đổi:

- Số combination: thường duyệt item trước, sau đó duyệt capacity.
- Số permutation: thường duyệt capacity trước, sau đó duyệt item.

Phần này trong phỏng vấn có thể không bị hỏi quá sâu, nhưng rất quan trọng khi gặp các bài như Coin Change II.

## Các dạng bài thường gặp

| Dạng bài                     | Thiết kế state                                                                        | Bài tiêu biểu |
| ---------------------------- | ------------------------------------------------------------------------------------- | ------------- |
| Climbing Stairs/House Robber | `dp[i]` biểu thị giá trị tối ưu của `i` vị trí đầu tiên                               | 70, 198       |
| Knapsack                     | `dp[j]` biểu thị giá trị tối ưu hoặc số phương án khi capacity bằng `j`               | 416, 518, 322 |
| Subsequence                  | `dp[i]` hoặc `dp[i][j]` biểu thị đáp án kết thúc tại một vị trí hoặc của hai prefix   | 300, 1143     |
| Palindrome                   | `dp[i][j]` biểu thị interval `[i, j]` có thỏa điều kiện hay không hoặc giá trị tối ưu | 647, 516      |
| Path                         | `dp[i][j]` biểu thị đáp án khi đi đến ô `(i, j)`                                      | 62, 64        |

## Chọn memoization search hay tabulation?

Cả hai cách viết đều lưu đáp án của các bài toán con.

| Cách viết          | Đặc điểm                                                | Trường hợp phù hợp                                |
| ------------------ | ------------------------------------------------------- | ------------------------------------------------- |
| Memoization search | Recursion từ target state xuống dưới, tính theo nhu cầu | State transition phức tạp, recursion tự nhiên hơn |
| Tabulation         | Điền bảng từ state nhỏ đến state lớn                    | Thứ tự duyệt rõ ràng, thuận tiện nén space        |

Nếu ban đầu chưa nghĩ rõ thứ tự duyệt, bạn có thể viết memoization search trước. Sau khi quan hệ giữa các state rõ ràng, hãy chuyển thành tabulation. Nhiều tree DP và interval DP sẽ dễ viết đúng hơn khi dùng memoization search.

## Cách trình bày khi viết tay trong phỏng vấn

Không nên bắt đầu trực tiếp từ code khi làm bài DP. Khi viết tay trong phỏng vấn, bạn có thể nói rõ 4 câu sau trước:

1. Mảng `dp` có ý nghĩa gì, đáp án cuối cùng nằm ở vị trí nào.
2. State hiện tại phụ thuộc vào những state cũ nào, vì sao những state cũ đó đã được tính.
3. Vì sao initialization được viết như vậy, đặc biệt `0`, `1` và infinity lần lượt biểu thị điều gì.
4. Vì sao thứ tự duyệt không dùng trước các state chưa tính hoặc các state không được phép dùng lặp.

Nếu không nói rõ được 4 câu này thì code rất có thể được viết dựa vào trí nhớ; khi gặp biến thể sẽ dễ mất phương hướng.

## Phân tích chi tiết bài tiêu biểu: Coin Change

[322. Coin Change](https://leetcode.cn/problems/coin-change/) là một bài rất phù hợp để phỏng vấn về unbounded knapsack. Đề bài cho denomination của coin và target amount, hỏi cần ít nhất bao nhiêu coin để tạo thành target amount; mỗi coin có thể được dùng vô hạn lần.

State definition có thể được trình bày như sau:

```text
dp[j] biểu thị số coin ít nhất cần để tạo thành amount j.
```

Initialization là điểm then chốt của bài này. `dp[0] = 0` biểu thị tạo thành amount 0 không cần coin; các amount khác trước tiên được đặt thành một giá trị lớn không thể đạt tới, biểu thị tạm thời chưa thể đạt.

Code dùng `Arrays.fill`, cần import `java.util.Arrays`.

```java
int coinChange(int[] coins, int amount) {
    int max = amount + 1;
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, max);
    dp[0] = 0;

    for (int coin : coins) {
        for (int j = coin; j <= amount; j++) {
            dp[j] = Math.min(dp[j], dp[j - coin] + 1);
        }
    }

    return dp[amount] == max ? -1 : dp[amount];
}
```

Vì sao phải duyệt capacity theo thứ tự tăng dần? Vì một coin có thể được dùng nhiều lần. Khi tính `dp[j]`, dùng `dp[j - coin]`; nếu `dp[j - coin]` đã được coin hiện tại cập nhật trong vòng này thì có nghĩa là coin hiện tại có thể tiếp tục được dùng, đúng với unbounded knapsack.

Nếu đề biến thành “mỗi loại coin chỉ được dùng một lần” thì phải duyệt capacity theo thứ tự giảm dần. Hướng duyệt không phải vấn đề về format, mà là cách kiểm soát một item có được tham gia lặp lại vào transition hay không.

## So sánh state definition

Với các bài DP, thường không phải không viết được transition mà là chọn sai ý nghĩa state. Một số nhóm state dưới đây trông gần giống nhau nhưng cách viết hoàn toàn khác:

| Dạng bài                       | Ý nghĩa state                                                  | Điểm cần chú ý khi transition                             |
| ------------------------------ | -------------------------------------------------------------- | --------------------------------------------------------- |
| Longest Increasing Subsequence | `dp[i]` biểu thị độ dài LIS kết thúc tại `nums[i]`             | Bắt buộc chọn `nums[i]`, tìm giá trị nhỏ hơn ở phía trước |
| House Robber                   | `dp[i]` biểu thị số tiền lớn nhất của `i` căn nhà đầu tiên     | Trộm hoặc không trộm căn nhà thứ `i`                      |
| Longest Common Subsequence     | `dp[i][j]` biểu thị độ dài LCS của hai prefix                  | So sánh ký tự cuối của hai prefix                         |
| Palindromic Substring          | `dp[i][j]` biểu thị interval `[i, j]` có phải palindrome không | Phụ thuộc vào interval bên trong `[i + 1, j - 1]`         |

Trong phỏng vấn, bạn có thể chủ động nói một câu: `dp[i]` ở đây là “kết thúc tại i”, không phải “giá trị tối ưu trong `i` phần tử đầu tiên”. Câu này giúp tránh nhiều lỗi khi viết bài subsequence.

## Minh họa quá trình và sample biên

Lấy Climbing Stairs làm ví dụ, khi `n = 5`, state thay đổi như sau:

| `i` | `dp[i - 2]` | `dp[i - 1]` | `dp[i]` |
| --- | ----------- | ----------- | ------- |
| 3   | 1           | 2           | 3       |
| 4   | 2           | 3           | 5       |
| 5   | 3           | 5           | 8       |

Bảng này cần được đọc không phải để chú ý bản thân các con số, mà để thấy state chỉ phụ thuộc vào hai vị trí trước đó nên có thể nén thành hai biến.

Với bài DP, nên kiểm tra các boundary sau:

| Input                      | Điểm cần chú ý                                         |
| -------------------------- | ------------------------------------------------------ |
| `n = 0` hoặc array rỗng    | Initialization đã bao quát chưa                        |
| Chỉ có 1 phần tử           | Có truy cập vượt giới hạn `dp[1]` không                |
| Không thể tạo thành target | Giá trị khởi tạo có thể biểu thị “không thể đạt” không |
| Tính số phương án          | Initialization và thứ tự duyệt có đúng không           |

Cách viết dễ sai:

```java
for (int j = weights[i]; j <= capacity; j++) {
    dp[j] = Math.max(dp[j], dp[j - weights[i]] + values[i]); // sai trong 0-1 knapsack
}
```

Trong 0-1 knapsack, phải duyệt capacity theo thứ tự giảm dần; nếu không, state vừa được cập nhật trong vòng hiện tại sẽ bị dùng lại, tương đương với việc chọn cùng một item nhiều lần.

## Điểm dễ sai

- Không được liên tục thay đổi ý nghĩa của `dp`.
- Initialization không phải cứ điền `0` tùy ý, mà phải xem ý nghĩa của state.
- Capacity của 0-1 knapsack duyệt theo thứ tự giảm dần, capacity của unbounded knapsack duyệt theo thứ tự tăng dần.
- Initialization khi tính số phương án khác với khi tính giá trị tối ưu.
- Với bài subsequence, thường phải phân biệt “kết thúc tại i” và “trong `i` phần tử đầu tiên”.

## Tự kiểm tra các câu hỏi thường gặp

- Vì sao bước đầu tiên của DP nhất định phải là định nghĩa state?
- Memoization search và tabulation khác nhau thế nào? Khi nào viết memoization trước sẽ chắc chắn hơn?
- Vì sao capacity của 0-1 knapsack phải được duyệt theo thứ tự giảm dần?
- Vì sao capacity của unbounded knapsack có thể được duyệt theo thứ tự tăng dần?
- Khi `dp[i]` biểu thị “kết thúc tại i” và khi biểu thị “`i` phần tử đầu tiên”, transition khác nhau thế nào?
- Khi tính số lần ít nhất, giá trị lớn nhất và số phương án, initialization lần lượt cần chú ý điều gì?

## Bài tập đề xuất

- [70. Climbing Stairs](https://leetcode.cn/problems/climbing-stairs/)
- [198. House Robber](https://leetcode.cn/problems/house-robber/)
- [322. Coin Change](https://leetcode.cn/problems/coin-change/)
- [416. Partition Equal Subset Sum](https://leetcode.cn/problems/partition-equal-subset-sum/)
- [300. Longest Increasing Subsequence](https://leetcode.cn/problems/longest-increasing-subsequence/)
- [1143. Longest Common Subsequence](https://leetcode.cn/problems/longest-common-subsequence/)

<!-- @include: @article-footer.snippet.md -->
