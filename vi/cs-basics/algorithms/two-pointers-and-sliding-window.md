---
title: "Tổng hợp câu hỏi phỏng vấn về two pointers và sliding window: array, linked list, string và các template thường gặp"
description: Tổng hợp câu hỏi phỏng vấn về two pointers và sliding window, giải thích left/right pointer, fast/slow pointer, read/write pointer, fixed window, variable window, template Java và các bài LeetCode thường gặp.
category: Computer Science Basics
tag:
  - Algorithms
head:
  - - meta
    - name: keywords
      content: two pointers,sliding window,fast/slow pointer,left/right pointer,read/write pointer,fixed window,variable window,array algorithm,linked list algorithm,string algorithm,LeetCode
---

Two pointers và sliding window thường được ôn cùng nhau, nhưng vấn đề chúng giải quyết không hoàn toàn giống nhau. Two pointers giống một chiến lược di chuyển hơn, còn sliding window nhấn mạnh việc duy trì trạng thái trong một đoạn liên tiếp.

Một cách phán đoán thực tế: nếu bài toán quan tâm đến quan hệ giữa hai vị trí, hãy nghĩ đến two pointers trước; nếu bài toán quan tâm đến continuous subarray hoặc continuous substring và cần duy trì điều kiện trong window, hãy nghĩ đến sliding window trước.

## Trọng tâm phỏng vấn

- Phân biệt được left/right pointer, fast/slow pointer và read/write pointer.
- Duy trì được count, sum, maximum hoặc trạng thái matching trong sliding window.
- Giải thích được vì sao pointer chỉ di chuyển theo một hướng và time complexity là `O(n)`.
- Xử lý được empty array, single element, duplicate element và boundary khi thu hẹp window.

## Chính xác thì hai cách này khác nhau thế nào?

Two pointers là cách viết rộng hơn: chỉ cần dùng hai pointer phối hợp tiến lên thì đều có thể gọi là two pointers. Sliding window cụ thể hơn, nó duy trì một đoạn liên tiếp `[left, right]`; trong window thường có một nhóm state, chẳng hạn character count, element sum, maximum hoặc matching count.

| Đặc điểm bài toán                                                       | Có khả năng dùng   |
| ----------------------------------------------------------------------- | ------------------ |
| Tìm hai số trong sorted array                                           | Left/right pointer |
| Tìm cycle, middle hoặc node thứ K từ cuối trong linked list             | Fast/slow pointer  |
| Xoá hoặc ghi đè element tại chỗ                                         | Read/write pointer |
| Tìm continuous subarray/substring dài nhất, ngắn nhất hoặc đếm số lượng | Sliding window     |

Khi phỏng vấn, hãy nói rõ ý nghĩa của pointer trước; cách này chắc chắn hơn so với viết code ngay. Ví dụ: “`left` biểu thị boundary bên trái của window, `right` biểu thị character đang được thử thêm vào window”, sau đó việc thu hẹp window sẽ không bị rối.

## Left/right pointer

Left/right pointer thường được dùng với sorted array hoặc bài toán thu hẹp từ hai đầu:

```java
int[] twoSumSorted(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;
    while (left < right) {
        int sum = nums[left] + nums[right];
        if (sum == target) {
            return new int[] {left, right};
        } else if (sum < target) {
            left++;
        } else {
            right--;
        }
    }
    return new int[] {-1, -1};
}
```

Nếu array không có thứ tự, thông thường hãy sort trước rồi dùng left/right pointer. Sau khi sort, cần nhớ time complexity trở thành `O(nlogn)`.

Lý do left/right pointer hoạt động là sau mỗi lần so sánh, ta có thể loại bỏ một phần đáp án. Với bài toán two sum trong sorted array:

- Nếu sum hiện tại quá nhỏ, nghĩa là số mà left pointer trỏ tới quá nhỏ; right pointer đi sang trái chỉ làm sum nhỏ hơn, vì vậy chỉ có thể dịch left pointer sang phải.
- Nếu sum hiện tại quá lớn, nghĩa là số mà right pointer trỏ tới quá lớn; left pointer đi sang phải chỉ làm sum lớn hơn, vì vậy chỉ có thể dịch right pointer sang trái.

Three sum cũng theo cùng một cách, chỉ khác là cố định trước một số rồi thực hiện two sum trong phần còn lại. Điểm khó là loại duplicate: cần loại duplicate ở số cố định, và sau khi left/right pointer tìm thấy đáp án cũng phải bỏ qua các giá trị trùng nhau.

## Fast/slow pointer

Fast/slow pointer thường được dùng với linked list:

```java
boolean hasCycle(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) {
            return true;
        }
    }
    return false;
}
```

Điểm trọng tâm của bài linked list không phải code dài hay ngắn, mà là ý nghĩa của pointer phải ổn định. Thứ tự của `fast != null && fast.next != null` cũng không được đảo ngược.

Fast/slow pointer thường có hai loại chênh lệch tốc độ:

- `fast` đi 2 bước mỗi lần, `slow` đi 1 bước mỗi lần: dùng để detect cycle và tìm middle của linked list.
- Một pointer đi trước `k` bước, sau đó hai pointer cùng đi: dùng để tìm node thứ `k` từ cuối.

Khi tìm node thứ `k` từ cuối, khoảng cách giữa hai pointer luôn là `k` node. Khi pointer phía trước đi đến cuối linked list, pointer phía sau vừa dừng ở vị trí mục tiêu. Khi xoá node thứ `N` từ cuối, thông thường sẽ thêm dummy head để tránh phải xử lý riêng trường hợp xoá head node.

## Read/write pointer

Read/write pointer thường được dùng để modify array tại chỗ:

```java
int removeDuplicates(int[] nums) {
    if (nums.length == 0) {
        return 0;
    }
    int write = 1;
    for (int read = 1; read < nums.length; read++) {
        if (nums[read] != nums[read - 1]) {
            nums[write] = nums[read];
            write++;
        }
    }
    return write;
}
```

`read` chịu trách nhiệm scan array gốc, còn `write` trỏ tới vị trí tiếp theo có thể ghi. Khi phỏng vấn, tốt nhất hãy nói rõ ý nghĩa của hai variable này trước.

Điểm cốt lõi của read/write pointer là “đọc hết array, chỉ ghi các phần cần giữ lại về phía trước”. Dạng bài này thường yêu cầu modify tại chỗ và return length mới, thay vì tạo array mới.

Khi xác định thời điểm ghi, có thể tự hỏi: element mà `read` đang trỏ tới có nên được giữ lại không? Nếu có, ghi nó vào `write`, sau đó `write++`; nếu không, chỉ di chuyển `read`.

## Variable sliding window

Lấy bài “longest substring without repeating characters” làm ví dụ:

```java
int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> count = new HashMap<>();
    int left = 0;
    int ans = 0;
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        count.put(c, count.getOrDefault(c, 0) + 1);
        while (count.get(c) > 1) {
            char d = s.charAt(left);
            count.put(d, count.get(d) - 1);
            left++;
        }
        ans = Math.max(ans, right - left + 1);
    }
    return ans;
}
```

Trong template này, right pointer chịu trách nhiệm mở rộng window, left pointer chịu trách nhiệm thu hẹp window khi window không hợp lệ. Mỗi character đi vào window nhiều nhất một lần và đi ra window một lần, vì vậy time complexity là `O(n)`.

Variable window thường có một nhịp cố định:

1. Right pointer thêm element mới và cập nhật state của window.
2. Khi window không thoả điều kiện, liên tục di chuyển left pointer và đồng thời cập nhật state.
3. Cập nhật đáp án tại vị trí window thoả yêu cầu bài toán.

Thời điểm cập nhật của bài toán tìm dài nhất và tìm ngắn nhất không giống nhau:

- Tìm longest valid window: thường cập nhật đáp án sau khi window trở nên hợp lệ.
- Tìm shortest window thoả điều kiện: thường cập nhật đáp án khi window đã thoả điều kiện, sau đó tiếp tục thu hẹp left boundary.

Ví dụ, trong bài “minimum window substring”, ngay khi window bao phủ đủ target character, cần cập nhật đáp án trước rồi mới thử thu nhỏ window; trong bài “longest substring without repeating characters”, khi window có duplicate character, cần thu hẹp về trạng thái hợp lệ trước rồi mới cập nhật đáp án.

## Fixed sliding window

Fixed window phù hợp với “subarray/substring có độ dài `k`”:

```java
int maxSum(int[] nums, int k) {
    int window = 0;
    for (int i = 0; i < k; i++) {
        window += nums[i];
    }
    int ans = window;
    for (int right = k; right < nums.length; right++) {
        window += nums[right];
        window -= nums[right - k];
        ans = Math.max(ans, window);
    }
    return ans;
}
```

Điểm trọng tâm của fixed window là thêm một element ở bên phải và loại một element ở bên trái.

Fixed window không cần `while` để thu hẹp vì độ dài window luôn cố định. Nó giống một phép thống kê rolling hơn:

- Element mới đi vào window.
- Element cũ rời khỏi window bị loại bỏ.
- Cập nhật đáp án của window hiện tại.

Nếu cần duy trì maximum hoặc minimum trong window, variable thông thường là chưa đủ; thường phải dùng monotonic queue. Ví dụ, trong bài “sliding window maximum”, queue lưu các index có khả năng trở thành maximum, còn phần tử đầu queue là maximum của window hiện tại.

## Quy trình viết code khi phỏng vấn

Với bài two pointers và sliding window, điều đáng sợ nhất khi phỏng vấn là ý nghĩa của pointer thay đổi giữa chừng. Nên viết theo thứ tự sau:

1. Xác định dạng bài trước: thu hẹp hai đầu, fast/slow đuổi nhau, ghi đè tại chỗ hay continuous window.
2. Xác định rõ ý nghĩa của pointer: `left`, `right`, `slow`, `fast`, `write` lần lượt trỏ tới đâu.
3. Xác định state của window: window duy trì sum, count, maximum hay matching count.
4. Xác định điều kiện di chuyển: khi nào right pointer mở rộng, khi nào left pointer thu hẹp.
5. Xác định thời điểm cập nhật đáp án: sau khi hợp lệ thì cập nhật longest, khi thoả điều kiện thì cập nhật shortest.

Một câu để phân biệt longest và shortest: **Bài longest thường sửa window trước rồi mới cập nhật đáp án; bài shortest thường ghi nhận đáp án trước rồi tiếp tục thu hẹp.**

## Giải thích chi tiết bài tiêu biểu: Minimum Window Substring

[76. Minimum Window Substring](https://leetcode.cn/problems/minimum-window-substring/) là một trong những bài kiểm tra nhiều chi tiết nhất về sliding window. Bài toán yêu cầu tìm substring ngắn nhất trong `s` sao cho bao phủ toàn bộ character và số lần xuất hiện tương ứng trong `t`.

Điểm mấu chốt của bài này không phải có biết dùng window hay không, mà là có giải thích rõ được ba count hay không:

- `need`: mỗi character trong target string `t` cần bao nhiêu.
- `window`: mỗi character hiện có bao nhiêu trong window hiện tại.
- `valid`: có bao nhiêu loại character đã đạt đủ số lần cần thiết.

Khi `valid == need.size()`, nghĩa là window hiện tại đã bao phủ `t`; lúc này cần cập nhật đáp án và thử thu hẹp left boundary.

```java
String minWindow(String s, String t) {
    Map<Character, Integer> need = new HashMap<>();
    Map<Character, Integer> window = new HashMap<>();
    for (char c : t.toCharArray()) {
        need.put(c, need.getOrDefault(c, 0) + 1);
    }

    int left = 0;
    int valid = 0;
    int start = 0;
    int minLen = Integer.MAX_VALUE;

    for (int right = 0; right < s.length(); right++) {
        char in = s.charAt(right);
        if (need.containsKey(in)) {
            window.put(in, window.getOrDefault(in, 0) + 1);
            if (window.get(in).equals(need.get(in))) {
                valid++;
            }
        }

        while (valid == need.size()) {
            if (right - left + 1 < minLen) {
                start = left;
                minLen = right - left + 1;
            }
            char out = s.charAt(left);
            left++;
            if (need.containsKey(out)) {
                if (window.get(out).equals(need.get(out))) {
                    valid--;
                }
                window.put(out, window.get(out) - 1);
            }
        }
    }

    return minLen == Integer.MAX_VALUE ? "" : s.substring(start, start + minLen);
}
```

Ở đây có hai điểm dễ viết sai:

- `valid--` phải xảy ra trước khi giảm `window[out]`, vì lúc đó window vẫn vừa đủ thoả điều kiện.
- Cập nhật đáp án phải đặt bên trong `while (valid == need.size())`, vì chỉ khi window hiện tại đã bao phủ `t` thì nó mới đủ điều kiện được so sánh với đáp án ngắn nhất.

## Minh hoạ quá trình và các sample ở boundary

Lấy “longest substring without repeating characters” làm ví dụ, biến đổi window của string `abba` như sau:

| Character của right pointer | Window sau khi thêm | Hợp lệ không | Left pointer di chuyển thế nào                      | Longest hiện tại |
| --------------------------- | ------------------- | ------------ | --------------------------------------------------- | ---------------- |
| `a`                         | `a`                 | Hợp lệ       | Không di chuyển                                     | 1                |
| `b`                         | `ab`                | Hợp lệ       | Không di chuyển                                     | 2                |
| `b`                         | `abb`               | Không hợp lệ | Bỏ `a` nhưng vẫn không hợp lệ, bỏ tiếp `b` đầu tiên | 2                |
| `a`                         | `ba`                | Hợp lệ       | Không di chuyển                                     | 2                |

Với sliding window, ít nhất nên kiểm tra các boundary sau:

| Input                         | Điểm trọng tâm                                  |
| ----------------------------- | ----------------------------------------------- |
| Empty string hoặc empty array | Có return trực tiếp 0 không                     |
| Tất cả character giống nhau   | Left boundary có liên tục thu hẹp không         |
| Không có duplicate character  | Đáp án có thể cập nhật tới toàn bộ length không |
| Window tối ưu ở đầu hoặc cuối | Thời điểm cập nhật đáp án có đúng không         |

Cách viết dễ sai:

```java
if (count.get(c) > 1) {
    left++; // Sai: chỉ di chuyển một lần chưa chắc khôi phục được window hợp lệ
}
```

Khi thu hẹp variable window, thông thường phải dùng `while` cho tới khi window trở lại thoả điều kiện. Chỉ di chuyển một lần sẽ dễ sai với các input như `abba`, `aaabc`.

## Điểm dễ sai

- Với bài two pointers, trước tiên phải xác định rõ ý nghĩa của hai pointer, không vừa viết vừa đoán.
- Trong sliding window, thời điểm cập nhật đáp án phụ thuộc vào việc đề bài hỏi longest hay shortest.
- Khi thu hẹp window, count, sum và matching count trong window đều phải được cập nhật đồng bộ.
- Với fast/slow pointer trong linked list, trước tiên phải kiểm tra `fast` và `fast.next`.
- Với bài three sum, việc loại duplicate sau khi sort phải được xử lý riêng.

## Tự kiểm tra với các câu hỏi thường gặp

- Vì sao bài two pointers thường có time complexity `O(n)`, thay vì `O(n^2)` do hai vòng lặp lồng nhau?
- Vì sao three sum cần sort? Việc loại duplicate xảy ra ở những vị trí nào?
- Khi fast/slow pointer tìm middle của linked list, với độ dài chẵn nên return node giữa trước hay node giữa sau?
- Khi nào sliding window dùng `if` để thu hẹp, khi nào bắt buộc dùng `while` để thu hẹp?
- Thời điểm cập nhật đáp án của longest window và shortest window khác nhau thế nào?

## Bài tập đề xuất

- [26. Remove Duplicates from Sorted Array](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/)
- [15. 3Sum](https://leetcode.cn/problems/3sum/)
- [141. Linked List Cycle](https://leetcode.cn/problems/linked-list-cycle/)
- [3. Longest Substring Without Repeating Characters](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)
- [76. Minimum Window Substring](https://leetcode.cn/problems/minimum-window-substring/)

<!-- @include: @article-footer.snippet.md -->
