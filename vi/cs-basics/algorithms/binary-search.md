---
title: "Tổng hợp câu hỏi phỏng vấn về binary search: biên trái, biên phải, binary search trên đáp án và template Java"
description: "Tổng hợp câu hỏi phỏng vấn về binary search, giải thích có hệ thống về binary search cơ bản, biên trái, biên phải, binary search trên đáp án, template Java viết tay, phân tích độ phức tạp và các bài LeetCode thường gặp."
category: Computer Basics
tag:
  - Algorithms
head:
  - - meta
    - name: keywords
      content: binary search,binary search template,left and right boundaries,binary search on answer,Java binary search,LeetCode binary search,algorithm interview questions
---

Điểm dễ khiến bạn mắc lỗi nhất khi làm binary search không nằm ở ý tưởng mà ở việc xử lý biên. `left`, `right`, `mid`, điều kiện vòng lặp và giá trị trả về, chỉ cần chưa hiểu rõ ý nghĩa của một yếu tố là bạn rất dễ viết thành vòng lặp vô hạn hoặc bỏ sót đáp án.

Khi phỏng vấn, để xác định có thể dùng binary search hay không, trước hết hãy hỏi: **không gian chứa đáp án có tính đơn điệu không**. Array đã sắp xếp chỉ là trường hợp trực quan nhất; các bài về tốc độ nhỏ nhất, capacity nhỏ nhất, số ngày ít nhất cũng có thể thực hiện binary search trên phạm vi đáp án.

## Trọng tâm phỏng vấn

- Viết được template binary search cơ bản.
- Xử lý được biên trái, biên phải.
- Nhận ra được binary search trên đáp án, thay vì chỉ biết tìm số trong array.
- Giải thích được vì sao vòng lặp kết thúc và vì sao không bỏ sót đáp án.
- Nêu được time complexity là `O(logn)`, space complexity thường là `O(1)`.

## Khi nào nghĩ đến binary search?

Đừng hiểu binary search là “chỉ có thể tìm số trong sorted array”. Điều binary search thực sự dựa vào là **tính đơn điệu**.

Tính đơn điệu thường có hai loại:

| Loại            | Ví dụ                                                   | Cách xác định                                                          |
| --------------- | ------------------------------------------------------- | ---------------------------------------------------------------------- |
| Array đơn điệu  | Tìm `target` trong sorted array                         | Sau khi so sánh `nums[mid]` với `target`, có thể loại bỏ một nửa       |
| Đáp án đơn điệu | Tìm tốc độ nhỏ nhất, capacity nhỏ nhất, số ngày ít nhất | Nếu một đáp án khả thi thì đáp án lớn hơn cũng khả thi, hoặc ngược lại |

Ví dụ trong bài “Koko ăn chuối”, tốc độ ăn càng nhanh thì càng dễ ăn hết trong thời gian quy định. Bản thân array không cần có thứ tự; tính đơn điệu nằm ở mối quan hệ giữa “tốc độ” và “có thể ăn hết hay không”.

Khi phỏng vấn, có thể xác định như sau:

1. Đề bài có đang tìm một vị trí, một biên hoặc một giá trị khả thi nhỏ nhất/lớn nhất không?
2. Nếu đoán một đáp án `x`, có thể xác định nó có khả thi trong `O(n)` hoặc độ phức tạp thấp hơn không?
3. Khi `x` tăng hoặc giảm, tính khả thi có thay đổi đơn điệu không?

Nếu trả lời được cả ba câu hỏi, về cơ bản bạn có thể thử dùng binary search.

## Template binary search cơ bản

Phù hợp để tìm một giá trị cụ thể trong array đã sắp xếp:

```java
int binarySearch(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            return mid;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return -1;
}
```

Trong template này, khoảng tìm kiếm là closed interval `[left, right]`, nên điều kiện vòng lặp là `left <= right`. Vì mỗi lần loại bỏ `mid` nên cập nhật thành `mid + 1` hoặc `mid - 1`.

Có thể ghi nhớ template này bằng một câu: **mọi vị trí trong interval đều vẫn có thể là đáp án; khi vòng lặp kết thúc, interval rỗng.**

Ví dụ, tìm `7` trong array `[1, 3, 5, 7, 9]`:

1. `left = 0`, `right = 4`, `mid = 2`, `nums[mid] = 5`, đáp án nằm bên phải.
2. Cập nhật `left = mid + 1 = 3`.
3. `mid = 3`, tìm thấy `7`.

Nếu tìm `6`, cuối cùng sẽ xuất hiện `left > right`, nghĩa là closed interval đã rỗng, trả về `-1`.

## Template biên trái

Tìm vị trí đầu tiên lớn hơn hoặc bằng `target`:

```java
int lowerBound(int[] nums, int target) {
    int left = 0;
    int right = nums.length;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] >= target) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }
    return left;
}
```

Khoảng tìm kiếm của template này là left-closed right-open `[left, right)`. `right` được khởi tạo bằng `nums.length`, giá trị trả về có thể bằng `nums.length`, biểu thị array không có vị trí nào lớn hơn hoặc bằng `target`.

Điểm mấu chốt của template biên trái không phải là “tìm `target`”, mà là “tìm vị trí đầu tiên thỏa điều kiện”. Cách viết này xử lý tự nhiên trường hợp không tồn tại đáp án.

Ví dụ, với array `[1, 2, 2, 2, 4]`, tìm vị trí đầu tiên lớn hơn hoặc bằng `2`:

- Khi `nums[mid] >= 2`, `mid` có thể chính là đáp án, nên không thể loại bỏ `mid`, cập nhật `right = mid`.
- Khi `nums[mid] < 2`, `mid` và mọi vị trí bên trái đều không thể là đáp án, cập nhật `left = mid + 1`.

Khi vòng lặp kết thúc, `left == right`, vị trí này chính là vị trí đầu tiên thỏa điều kiện.

## Template biên phải

Để tìm vị trí cuối cùng nhỏ hơn hoặc bằng `target`, trước tiên có thể tìm vị trí đầu tiên lớn hơn `target`, sau đó trừ 1:

```java
int upperBound(int[] nums, int target) {
    int left = 0;
    int right = nums.length;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] > target) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }
    return left - 1;
}
```

Ưu điểm của cách viết này là chỉ cần ghi nhớ một cách tiếp cận cho cả biên trái và biên phải: tìm vị trí đầu tiên thỏa điều kiện.

Biên phải dễ viết sai, nên chuyển về bài toán biên trái:

- Vị trí cuối cùng nhỏ hơn hoặc bằng `target` = vị trí đầu tiên lớn hơn `target` - 1.
- Vị trí cuối cùng nhỏ hơn `target` = vị trí đầu tiên lớn hơn hoặc bằng `target` - 1.

Như vậy không cần duy trì hai template; khi viết tay trong phỏng vấn sẽ ít dễ sai hơn.

## Binary search trên đáp án

Binary search trên đáp án không tìm phần tử trong array, mà tìm giá trị khả thi nhỏ nhất hoặc lớn nhất trong phạm vi đáp án.

Bài toán điển hình: cho một số pile chứa chuối và tổng thời gian `h`, tìm tốc độ ăn chuối nhỏ nhất. Tốc độ càng nhanh thì càng dễ ăn hết trong `h` giờ, đó chính là tính đơn điệu.

Các bài dạng này thường gồm hai bước:

1. Xác định phạm vi đáp án. Ví dụ tốc độ nhỏ nhất là `1`, lớn nhất không vượt quá số chuối trong pile lớn nhất.
2. Viết hàm `check`. Với một tốc độ cho trước, kiểm tra có thể ăn hết trong `h` giờ hay không.

Cận trên này dựa trên ràng buộc của đề bài: `h >= piles.length`. Vì khi tốc độ bằng kích thước pile lớn nhất, mỗi pile mất nhiều nhất 1 giờ để ăn hết, tổng thời gian không vượt quá số pile.

```java
int minEatingSpeed(int[] piles, int h) {
    int left = 1;
    int right = 0;
    for (int pile : piles) {
        right = Math.max(right, pile);
    }
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (canFinish(piles, h, mid)) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }
    return left;
}

boolean canFinish(int[] piles, int h, int speed) {
    long hours = 0;
    for (int pile : piles) {
        hours += (pile + speed - 1) / speed;
    }
    return hours <= h;
}
```

Vì sao ở đây trả về `left`? Vì vòng lặp luôn tìm “tốc độ khả thi đầu tiên”. Khi `canFinish(mid)` là true, nghĩa là `mid` khả thi, nhưng có thể vẫn có tốc độ nhỏ hơn cũng khả thi, nên thu hẹp biên phải. Cuối cùng, vị trí mà hai biên trùng nhau chính là tốc độ khả thi nhỏ nhất.

Trong binary search trên đáp án, hàm `check` thường quan trọng hơn bản thân binary search. Khi phỏng vấn, nên nói rõ ý nghĩa của `check` trước, sau đó mới viết framework binary search.

## Chọn giữa ba loại binary search

| Mục tiêu                           | Template đề xuất          | Giá trị trả về                                        |
| ---------------------------------- | ------------------------- | ----------------------------------------------------- |
| Tìm index của `target`             | Binary search cơ bản      | Tìm thấy thì trả về index, không tìm thấy trả về `-1` |
| Tìm vị trí đầu tiên thỏa điều kiện | Biên trái                 | Trả về `left`, có thể bằng độ dài array               |
| Tìm đáp án khả thi nhỏ nhất        | Binary search trên đáp án | Trả về `left` cuối cùng                               |

Nếu đề bài có các cụm từ “đầu tiên”, “cuối cùng”, “khả thi nhỏ nhất”, “khả thi lớn nhất”, đừng vội viết binary search cơ bản; trước tiên hãy xác định xem đó có phải bài toán biên hay không.

## Quy trình viết thủ công khi phỏng vấn

Code của bài binary search không dài; trong phỏng vấn, điều thường bị hỏi sâu hơn là “vì sao bạn dám loại bỏ một nửa”. Khi viết tay, có thể theo thứ tự này:

1. Trước tiên nói rõ search space: đang tìm trong index của array hay trong phạm vi đáp án.
2. Sau đó nói rõ tính đơn điệu: vì sao có thể loại bỏ một phía của `mid`.
3. Xác định rõ ý nghĩa interval: closed interval `[left, right]` hay left-closed right-open `[left, right)`.
4. Viết quy tắc cập nhật: `mid` còn có thể là đáp án hay không quyết định viết `right = mid` hay `right = mid - 1`.
5. Cuối cùng nói rõ giá trị trả về: khi vòng lặp kết thúc, `left`, `right` lần lượt biểu thị điều gì.

Một câu hỏi tự kiểm tra rất hữu ích là: **khi `nums[mid]` vừa thỏa điều kiện, tôi có loại bỏ mất đáp án có thể đúng không?** Trong binary search biên trái và binary search trên đáp án, `mid` thường vẫn có thể là đáp án, nên không được tùy tiện viết thành `right = mid - 1`.

## Phân tích chi tiết bài tiêu biểu: tìm vị trí đầu tiên và cuối cùng

[34. Tìm vị trí đầu tiên và cuối cùng của phần tử trong array đã sắp xếp](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/) là bài tiêu biểu về binary search tìm biên. Đề bài yêu cầu trả về vị trí bắt đầu và kết thúc của `target`; nếu không tồn tại thì trả về `[-1, -1]`.

Không nên viết bài này thành “sau khi tìm thấy một `target` thì quét sang hai bên”. Cách đó có thể vượt qua một số test, nhưng trong trường hợp xấu nhất sẽ suy biến thành `O(n)`. Cách chắc chắn hơn là thực hiện hai lần tìm biên:

- Lần đầu tìm vị trí đầu tiên lớn hơn hoặc bằng `target`.
- Lần thứ hai tìm vị trí đầu tiên lớn hơn `target`, sau đó trừ 1.

Hai helper method dưới đây giống các template ở trên; ở đây giữ lại code đầy đủ để tiện đối chiếu ý nghĩa của giá trị trả về với logic chính.

```java
int[] searchRange(int[] nums, int target) {
    int left = lowerBound(nums, target);
    if (left == nums.length || nums[left] != target) {
        return new int[] {-1, -1};
    }
    int right = upperBound(nums, target) - 1;
    return new int[] {left, right};
}

int lowerBound(int[] nums, int target) {
    int left = 0;
    int right = nums.length;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] >= target) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }
    return left;
}

int upperBound(int[] nums, int target) {
    int left = 0;
    int right = nums.length;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] > target) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }
    return left;
}
```

Trong phỏng vấn, câu hỏi tiếp theo thường gặp là: nếu array chỉ gồm `target` thì sao? Nếu `target` không tồn tại nhưng đáng lẽ phải được chèn vào giữa thì sao? Thực ra cả hai câu hỏi đều kiểm tra ý nghĩa của giá trị trả về. `lowerBound` trả về vị trí đầu tiên thỏa điều kiện, không đảm bảo phần tử tại vị trí đó bằng `target`, nên trước khi trả về cần kiểm tra lại một lần.

## Minh họa quá trình và các ví dụ biên

Lấy template biên trái làm ví dụ, tìm vị trí đầu tiên lớn hơn hoặc bằng `2` trong array `[1, 2, 2, 2, 4]`:

| Lần      | `left` | `right` | `mid` | Kiểm tra        | Bước tiếp theo |
| -------- | ------ | ------- | ----- | --------------- | -------------- |
| 1        | 0      | 5       | 2     | `nums[2] >= 2`  | `right = 2`    |
| 2        | 0      | 2       | 1     | `nums[1] >= 2`  | `right = 1`    |
| 3        | 0      | 1       | 0     | `nums[0] < 2`   | `left = 1`     |
| Kết thúc | 1      | 1       | -     | `left == right` | Trả về 1       |

Trước khi viết tay, nên xem qua một lượt vài ví dụ biên:

| Input       | Mục tiêu          | Kết quả mong đợi                                     |
| ----------- | ----------------- | ---------------------------------------------------- |
| `[]`        | `1`               | Trả về `-1` hoặc vị trí chèn `0`, tùy yêu cầu đề bài |
| `[1]`       | `1`               | Tìm thấy phần tử duy nhất                            |
| `[1, 1, 1]` | Biên trái của `1` | Trả về `0`                                           |
| `[1, 3, 5]` | Biên trái của `4` | Trả về `2`                                           |
| `[1, 3, 5]` | Biên trái của `6` | Trả về `3`                                           |

Cách viết sai thường gặp:

```java
while (left < right) {
    int mid = (left + right) / 2;
    if (nums[mid] >= target) {
        right = mid - 1; // Sai: mid có thể chính là biên trái, không thể loại bỏ trực tiếp
    } else {
        left = mid + 1;
    }
}
```

Trong bài toán biên trái, khi `nums[mid] >= target`, `mid` vẫn có thể là đáp án, nên phải viết `right = mid`.

## Lỗi thường gặp

- `mid = (left + right) / 2` có thể xảy ra integer overflow; khuyến nghị viết thành `left + (right - left) / 2`.
- Không được trộn lẫn cách cập nhật của closed interval và left-closed right-open interval.
- Khi tìm biên, sau khi tìm thấy `target` thường không thể trả về ngay, mà còn phải tiếp tục thu hẹp interval.
- Với binary search trên đáp án, trước tiên phải chứng minh tính đơn điệu, không thể thấy có “giá trị nhỏ nhất” là áp dụng máy móc.
- Trong các hàm kiểm tra như `canFinish`, có thể cần `long` để tránh overflow khi cộng dồn.

## Tự kiểm tra với các câu hỏi thường gặp

- `left < right` và `left <= right` khác nhau thế nào?
- Vì sao binary search có độ phức tạp `O(logn)`?
- Khi tìm biên trái, vì sao sau khi tìm thấy phải di chuyển `right`?
- Binary search trên đáp án là gì? Nó khác binary search thông thường thế nào?
- Binary search có nhất thiết yêu cầu array phải được sắp xếp không?

## Bài tập đề xuất

- [704. Binary search](https://leetcode.cn/problems/binary-search/)
- [35. Search insert position](https://leetcode.cn/problems/search-insert-position/)
- [34. Tìm vị trí đầu tiên và cuối cùng của phần tử trong array đã sắp xếp](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)
- [875. Koko ăn chuối](https://leetcode.cn/problems/koko-eating-bananas/)
- [1011. Năng lực vận chuyển package trong D ngày](https://leetcode.cn/problems/capacity-to-ship-packages-within-d-days/)

<!-- @include: @article-footer.snippet.md -->
