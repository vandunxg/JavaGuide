---
title: "Tổng hợp câu hỏi phỏng vấn về Backtracking: combination, permutation, subset, pruning và template Java"
description: "Tổng hợp câu hỏi phỏng vấn về Backtracking, giải thích cách nhận diện dạng bài Backtracking, template combination, template permutation, template subset, loại bỏ trùng lặp, pruning, phân tích độ phức tạp và các bài thường gặp trên LeetCode."
category: Computer Basics
tag:
  - Algorithms
head:
  - - meta
    - name: keywords
      content: backtracking,backtracking template,combination,permutation,subset,N-Queens,pruning,Java backtracking,LeetCode backtracking,algorithm interview questions
---

Đặc điểm của bài toán Backtracking rất rõ ràng: đề bài yêu cầu bạn tìm tất cả phương án, tất cả đường đi, tất cả combination, hoặc thử các lựa chọn trong một tập hợp. Nó rất giống DFS, điểm khác biệt là Backtracking nhấn mạnh hơn vào “chọn -> đệ quy -> hủy lựa chọn”.

Khi viết Backtracking trong phỏng vấn, điều quan trọng nhất là trước tiên phải nêu rõ ý nghĩa của hàm đệ quy. Khi đã xác định rõ ý nghĩa của hàm, tham số, điều kiện kết thúc và thao tác hủy lựa chọn sẽ ít bị rối hơn.

## Trọng tâm phỏng vấn

- Có thể viết template cho ba dạng: combination, permutation và subset.
- Có thể giải thích tác dụng của `path`, `startIndex`, `used`.
- Có thể xác định từ đề bài xem có cần loại bỏ trùng lặp hay không.
- Có thể thực hiện pruning đơn giản để tránh tìm kiếm vô ích.
- Có thể nói rõ độ phức tạp liên quan đến số lượng kết quả.

## Suy nghĩ về bài Backtracking như thế nào?

Có thể bắt đầu bằng cách vẽ bài toán Backtracking thành một “cây lựa chọn”. Mỗi tầng của cây đại diện cho một lần lựa chọn, node gốc đại diện cho trạng thái chưa chọn, node lá đại diện cho một phương án hoàn chỉnh.

Trước khi viết code, hãy trả lời 4 câu hỏi:

1. Path là gì? Thông thường là các phần tử đã chọn, trong code gọi là `path`.
2. Danh sách lựa chọn là gì? Những phần tử nào hiện vẫn có thể chọn.
3. Điều kiện kết thúc là gì? Khi nào thêm `path` vào kết quả.
4. Có cần pruning không? Những lựa chọn nào chắc chắn không thể tạo ra đáp án hợp lệ.

“Hủy lựa chọn” trong template Backtracking không phải là hình thức. Vì `path` được dùng lại, sau khi thử xong nhánh hiện tại phải khôi phục trạng thái để nhánh tiếp theo sử dụng.

## Template combination

Combination không quan tâm đến thứ tự, thường dùng `startIndex` để kiểm soát tầng tiếp theo bắt đầu từ đâu:

```java
List<List<Integer>> combine(int n, int k) {
    List<List<Integer>> ans = new ArrayList<>();
    backtrack(1, n, k, new ArrayList<>(), ans);
    return ans;
}

void backtrack(int start, int n, int k, List<Integer> path, List<List<Integer>> ans) {
    if (path.size() == k) {
        ans.add(new ArrayList<>(path));
        return;
    }
    for (int i = start; i <= n; i++) {
        path.add(i);
        backtrack(i + 1, n, k, path, ans);
        path.remove(path.size() - 1);
    }
}
```

Bài combination không quan tâm đến thứ tự, vì vậy `[1, 2]` và `[2, 1]` là cùng một kết quả. Tác dụng của `start` là đảm bảo các lần chọn sau chỉ chọn những số nằm sau vị trí hiện tại, tránh trùng lặp.

Nếu cần chọn `k` số từ `1..n`, còn có thể thực hiện pruning:

```java
for (int i = start; i <= n - (k - path.size()) + 1; i++) {
    // ...
}
```

Ý nghĩa là: nếu bắt đầu từ `i` mà số lượng số còn lại không đủ để hoàn thành đủ `k` số, thì không cần tiếp tục liệt kê.

## Template permutation

Permutation quan tâm đến thứ tự, thường dùng `used` để đánh dấu phần tử đã được chọn hay chưa:

```java
List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> ans = new ArrayList<>();
    boolean[] used = new boolean[nums.length];
    backtrack(nums, used, new ArrayList<>(), ans);
    return ans;
}

void backtrack(int[] nums, boolean[] used, List<Integer> path, List<List<Integer>> ans) {
    if (path.size() == nums.length) {
        ans.add(new ArrayList<>(path));
        return;
    }
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) {
            continue;
        }
        used[i] = true;
        path.add(nums[i]);
        backtrack(nums, used, path, ans);
        path.remove(path.size() - 1);
        used[i] = false;
    }
}
```

Bài permutation quan tâm đến thứ tự, vì vậy ở mỗi tầng đều có thể chọn từ tất cả các số, chỉ là không thể sử dụng lại cùng một phần tử. `used[i]` biểu thị phần tử `nums[i]` đã được chọn trong `path` hiện tại hay chưa.

Nếu mảng có các số trùng nhau, việc loại bỏ permutation trùng lặp dễ viết sai hơn combination. Thông thường trước tiên sắp xếp mảng, sau đó ở cùng một tầng bỏ qua trường hợp “phần tử trùng trước đó vẫn chưa được sử dụng”:

```java
if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) {
    continue;
}
```

Câu lệnh này cố định thứ tự lựa chọn của các phần tử trùng nhau ở cùng một tầng, tránh tạo ra permutation trùng lặp.

## Template subset

Trong bài subset, thông thường mỗi node đều là một đáp án:

```java
List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> ans = new ArrayList<>();
    backtrack(0, nums, new ArrayList<>(), ans);
    return ans;
}

void backtrack(int start, int[] nums, List<Integer> path, List<List<Integer>> ans) {
    ans.add(new ArrayList<>(path));
    for (int i = start; i < nums.length; i++) {
        path.add(nums[i]);
        backtrack(i + 1, nums, path, ans);
        path.remove(path.size() - 1);
    }
}
```

Bài subset rất giống bài combination, nhưng không chỉ thu thập kết quả khi đạt độ dài cố định, mà thu thập một lần ở mỗi node. Vì `path` có độ dài bất kỳ đều có thể là một subset.

Nếu đề bài yêu cầu loại bỏ trùng lặp, chẳng hạn input `[1, 2, 2]`, vẫn trước tiên sắp xếp mảng, sau đó bỏ qua phần tử trùng lặp ở cùng một tầng:

```java
if (i > start && nums[i] == nums[i - 1]) {
    continue;
}
```

## Loại bỏ trùng lặp như thế nào?

Nếu input có phần tử trùng lặp, thông thường trước tiên sắp xếp mảng, sau đó chọn chiến lược loại bỏ trùng lặp dựa trên dạng bài:

- Với các bài như subset và combination, chọn theo các index tăng dần, bỏ qua phần tử trùng lặp ở cùng một tầng, chẳng hạn `i > start && nums[i] == nums[i - 1]`.
- Với các bài như permutation, mỗi tầng đều có thể quét từ đầu, thường còn phải kết hợp với `used[]` để tránh sử dụng lại cùng một vị trí.
- Điều kiện loại bỏ trùng lặp phải phân biệt “lựa chọn trùng lặp ở cùng một tầng” và “sử dụng lặp lại trên cùng một path”. Trường hợp trước tạo ra đáp án trùng lặp, trường hợp sau có thể chính là lựa chọn được đề bài cho phép.

## Minh họa quá trình và ví dụ biên

Lấy bài toán combination với `n = 3, k = 2` làm ví dụ, cây lựa chọn có thể đơn giản hóa như sau:

| Lựa chọn ở tầng thứ nhất | Có thể chọn ở tầng thứ hai | Kết quả tạo ra                   |
| ------------------------ | -------------------------- | -------------------------------- |
| Chọn 1                   | 2, 3                       | `[1, 2]`, `[1, 3]`               |
| Chọn 2                   | 3                          | `[2, 3]`                         |
| Chọn 3                   | Không có                   | Không đủ 2 số, thực hiện pruning |

Khi làm bài Backtracking, nên kiểm tra các trường hợp biên sau:

| Input                | Trọng tâm kiểm tra                                   |
| -------------------- | ---------------------------------------------------- |
| Mảng rỗng            | Bài subset thường phải trả về `[[]]`                 |
| `k = 0`              | Bài combination có trả về combination rỗng hay không |
| Có phần tử trùng lặp | Đã sắp xếp và loại bỏ trùng lặp cùng tầng hay chưa   |
| Chỉ có một kết quả   | Có copy `path` đúng cách hay không                   |

Cách viết sai thường gặp:

```java
ans.add(path); // Sai: path sẽ tiếp tục thay đổi ở phía sau
```

Cần viết thành:

```java
ans.add(new ArrayList<>(path));
```

`path` trong Backtracking là object được dùng lại, nếu không copy thì các list trong kết quả đều sẽ bị thay đổi theo các lần đệ quy sau.

## Điểm dễ sai

- Khi thêm vào kết quả phải copy `path`, không được đưa trực tiếp reference vào.
- Combination dùng `startIndex`, permutation dùng `used`, không được viết lẫn lộn.
- Khi loại bỏ trùng lặp, thông thường phải sắp xếp mảng trước.
- Điều kiện pruning không được làm ảnh hưởng đến đáp án đúng.
- Độ phức tạp của Backtracking thường cùng bậc với số lượng kết quả, không nên tùy tiện viết là `O(n)`.

## Bài tập đề xuất

- [77. Combination](https://leetcode.cn/problems/combinations/)
- [78. Subset](https://leetcode.cn/problems/subsets/)
- [46. Permutation](https://leetcode.cn/problems/permutations/)
- [39. Combination Sum](https://leetcode.cn/problems/combination-sum/)
- [51. N-Queens](https://leetcode.cn/problems/n-queens/)

<!-- @include: @article-footer.snippet.md -->
