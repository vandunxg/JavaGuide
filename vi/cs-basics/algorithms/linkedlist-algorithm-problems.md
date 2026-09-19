---
title: Một số bài toán thuật toán trên linked list thường gặp
description: Tuyển chọn các bài toán linked list phổ biến, kèm hướng giải và cách triển khai, gồm cộng hai số, đảo ngược, phát hiện cycle, nhấn mạnh xử lý trường hợp biên và phân tích complexity.
category: Computer Fundamentals
tag:
  - Algorithms
head:
  - - meta
    - name: keywords
      content: linked list algorithm, cộng hai số, đảo ngược linked list, phát hiện cycle, hợp nhất linked list, phân tích complexity
---

<!-- markdownlint-disable MD024 -->

## 1. Cộng hai số

### Mô tả bài toán

> LeetCode: Cho hai linked list không rỗng biểu diễn hai số nguyên không âm. Các chữ số được lưu theo thứ tự ngược; mỗi node chỉ lưu một chữ số. Hãy cộng hai số và trả về một linked list mới.
>
> Bạn có thể giả định rằng ngoài số 0, hai số này đều không bắt đầu bằng 0.

Ví dụ:

```plain
Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Reason: 342 + 465 = 807
```

### Phân tích bài toán

Địa chỉ lời giải chi tiết chính thức của LeetCode:

<https://leetcode-cn.com/problems/add-two-numbers/solution/>

> Khi cần thao tác với head node, hãy cân nhắc tạo một dummy node và dùng dummy->next để biểu thị head node thực sự. Cách này giúp tránh xử lý trường hợp biên head node rỗng.

Chúng ta dùng một biến để theo dõi carry, rồi mô phỏng việc cộng lần lượt từng chữ số, bắt đầu từ đầu danh sách, nơi chứa chữ số có trọng số thấp nhất.

![Hình 1, trực quan hóa phương pháp cộng hai số: 342 + 465 = 807, mỗi node chứa một chữ số và các chữ số được lưu theo thứ tự ngược.](https://oss.javaguide.cn/github/javaguide/cs-basics/algorithms/34910956.jpg)

### Solution

**Trước hết, chúng ta cộng từ chữ số có trọng số thấp nhất, tức head của list l1 và l2. Hãy nhớ xử lý carry!**

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
//https://leetcode-cn.com/problems/add-two-numbers/description/
class Solution {
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    ListNode dummyHead = new ListNode(0);
    ListNode p = l1, q = l2, curr = dummyHead;
    // carry biểu thị giá trị nhớ
    int carry = 0;
    while (p != null || q != null) {
        int x = (p != null) ? p.val : 0;
        int y = (q != null) ? q.val : 0;
        int sum = carry + x + y;
        // giá trị nhớ
        carry = sum / 10;
        // giá trị của node mới là sum % 10
        curr.next = new ListNode(sum % 10);
        curr = curr.next;
        if (p != null) p = p.next;
        if (q != null) q = q.next;
    }
    if (carry > 0) {
        curr.next = new ListNode(carry);
    }
    return dummyHead.next;
}
}
```

## 2. Đảo ngược linked list

### Mô tả bài toán

> Jianzhi Offer: Nhập một linked list, đảo ngược linked list rồi xuất toàn bộ phần tử.

![Đảo ngược linked list](https://oss.javaguide.cn/github/javaguide/cs-basics/algorithms/81431871.jpg)

### Phân tích bài toán

Nói đơn giản, bài toán này là: làm thế nào để node phía sau trỏ đến node phía trước! Trong code bên dưới, một node next được định nghĩa để lưu node tiếp theo cần đảo ngược, tránh làm linked list bị “đứt”.

### Solution

```java
public class ListNode {
  int val;
  ListNode next = null;

  ListNode(int val) {
    this.val = val;
  }
}
```

```java
/**
 *
 * @author Snailclimb
 * @date 2018-09-19
 * @Description: Đảo ngược singly linked list
 */
public class Solution {

  public ListNode ReverseList(ListNode head) {

    ListNode next = null;
    ListNode pre = null;

    while (head != null) {
      // Lưu node tiếp theo cần đảo ngược
      next = head.next;
      // Node cần đảo ngược trỏ đến node trước đó đã được đảo ngược (lần đảo ngược đầu tiên sẽ trỏ đến null)
      head.next = pre;
      // Đưa node hiện tại lên đầu phần đã đảo ngược
      pre = head;
      // Tiếp tục đi về phía đuôi linked list
      head = next;
    }
    return pre;
  }

}
```

Cách kiểm thử:

```java
  public static void main(String[] args) {

    ListNode a = new ListNode(1);
    ListNode b = new ListNode(2);
    ListNode c = new ListNode(3);
    ListNode d = new ListNode(4);
    ListNode e = new ListNode(5);
    a.next = b;
    b.next = c;
    c.next = d;
    d.next = e;
    new Solution().ReverseList(a);
    while (e != null) {
      System.out.println(e.val);
      e = e.next;
    }
  }
```

Output:

```plain
5
4
3
2
1
```

## 3. Node thứ k tính từ cuối trong linked list

### Mô tả bài toán

> Jianzhi Offer: Nhập một linked list, hãy xuất node thứ k tính từ cuối trong linked list đó.

### Phân tích bài toán

> **Node thứ k tính từ cuối trong linked list cũng chính là node thứ (L - k + 1) tính từ đầu. Chỉ cần nắm được điều này là cơ bản giải quyết được bài toán!**

Trước hết dùng hai node/pointer. node1 chạy trước; sau khi node1 đi qua k - 1 node, node2 bắt đầu chạy. Khi node1 chạy đến cuối, node mà node2 trỏ tới chính là node thứ k tính từ cuối, cũng là node thứ (L - k + 1) tính từ đầu.

### Solution

```java
/*
public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}*/

// Time complexity O(n), chỉ cần duyệt một lần
// https://www.nowcoder.com/practice/529d3ae5a407492994ad2a246518148a?tpId=13&tqId=11167&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
public class Solution {
  public ListNode FindKthToTail(ListNode head, int k) {
    // Nếu linked list rỗng hoặc k nhỏ hơn hoặc bằng 0
    if (head == null || k <= 0) {
      return null;
    }
    // Khai báo hai node cùng trỏ đến head node
    ListNode node1 = head, node2 = head;
    // Ghi lại số lượng node
    int count = 0;
    // Ghi lại giá trị k để sử dụng về sau
    int index = k;
    // Pointer node1 chạy trước và ghi lại số node. Khi node1 chạy qua k - 1 node thì node2 bắt đầu chạy,
    // khi node1 chạy đến cuối, node mà node2 trỏ tới chính là node thứ k tính từ cuối
    while (node1 != null) {
      node1 = node1.next;
      count++;
      if (k < 1) {
        node2 = node2.next;
      }
      k--;
    }
    // Nếu số lượng node nhỏ hơn node thứ k tính từ cuối cần tìm thì trả về null
    if (count < index)
      return null;
    return node2;

  }
}
```

## 4. Xóa node thứ N tính từ cuối trong linked list

> LeetCode: Cho một linked list, xóa node thứ n tính từ cuối và trả về head node của linked list.

**Ví dụ:**

```plain
Cho một linked list: 1->2->3->4->5 và n = 2.

Sau khi xóa node thứ hai tính từ cuối, linked list trở thành 1->2->3->5.

```

**Giải thích:**

Giá trị n được cho là hợp lệ.

**Nâng cao:**

Bạn có thể thử triển khai bằng một lần duyệt không?

Bài toán này có lời giải chi tiết trên LeetCode, hãy tham khảo LeetCode.

### Phân tích bài toán

Chúng ta nhận thấy bài toán này có thể dễ dàng rút gọn thành một bài toán khác: xóa node thứ (L - n + 1) tính từ đầu danh sách, trong đó L là độ dài danh sách. Chỉ cần tìm được độ dài L của danh sách thì bài toán sẽ dễ giải quyết.

![Hình 1. Xóa phần tử thứ L - n + 1 trong danh sách](https://oss.javaguide.cn/github/javaguide/cs-basics/algorithms/94354387.jpg)

### Solution

**Phương pháp duyệt hai lần**

Trước hết, chúng ta thêm một **dummy node** ở đầu danh sách để hỗ trợ. Dummy node giúp đơn giản hóa một số trường hợp đặc biệt, chẳng hạn danh sách chỉ có một node hoặc cần xóa head của danh sách. Trong lần duyệt đầu tiên, chúng ta tìm độ dài L của danh sách. Sau đó đặt một pointer trỏ đến dummy node và di chuyển nó qua danh sách cho đến khi đến node thứ (L - n). **Chúng ta nối lại pointer next của node thứ (L - n) đến node thứ (L - n + 2) để hoàn tất thuật toán.**

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
// https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/description/
public class Solution {
  public ListNode removeNthFromEnd(ListNode head, int n) {
    // Dummy node giúp đơn giản hóa một số trường hợp đặc biệt, chẳng hạn danh sách chỉ có một node hoặc cần xóa head của danh sách
    ListNode dummy = new ListNode(0);
    // Dummy node trỏ đến head node
    dummy.next = head;
    // Lưu độ dài linked list
    int length = 0;
    ListNode len = head;
    while (len != null) {
      length++;
      len = len.next;
    }
    length = length - n;
    ListNode target = dummy;
    // Tìm node ở vị trí L - n
    while (length > 0) {
      target = target.next;
      length--;
    }
    // Nối lại pointer next của node thứ (L - n) đến node thứ (L - n + 2)
    target.next = target.next.next;
    return dummy.next;
  }
}
```

**Nâng cao - phương pháp duyệt một lần:**

> Node thứ n tính từ cuối trong linked list cũng chính là node thứ (L - n + 1) tính từ đầu.

Thực ra phương pháp này dùng cùng ý tưởng với cách tìm “node thứ k tính từ cuối trong linked list” ở bài thứ tư bên trên. **Ý tưởng cơ bản là:** định nghĩa hai node node1 và node2; node1 chạy trước, khi node1 chạy đến node thứ n + 1 thì node2 bắt đầu chạy. Khi node1 chạy đến node cuối cùng, vị trí của node2 chính là node thứ (L - n), trong đó L là tổng độ dài linked list, cũng là node thứ n + 1 tính từ cuối.

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
public class Solution {
  public ListNode removeNthFromEnd(ListNode head, int n) {

    ListNode dummy = new ListNode(0);
    dummy.next = head;
    // Khai báo hai node cùng trỏ đến dummy node
    ListNode node1 = dummy, node2 = dummy;

    // node1 chạy trước; khi node1 chạy đến node thứ n thì node2 bắt đầu chạy
    // Khi node1 chạy đến node cuối cùng, vị trí của node2 là node thứ (L-n), cũng là node thứ n+1 tính từ cuối (L là tổng độ dài linked list)
    while (node1 != null) {
      node1 = node1.next;
      if (n < 1 && node1 != null) {
        node2 = node2.next;
      }
      n--;
    }

    node2.next = node2.next.next;

    return dummy.next;

  }
}
```

## 5. Hợp nhất hai linked list đã sắp xếp

### Mô tả bài toán

> Jianzhi Offer: Nhập hai linked list tăng dần, hãy xuất linked list sau khi hợp nhất. Linked list sau khi hợp nhất phải thỏa mãn quy tắc không giảm.

### Phân tích bài toán

Chúng ta có thể phân tích như sau:

1. Giả sử có hai linked list A và B;
2. So sánh giá trị của head node A1 của A với head node B1 của B. Giả sử A1 nhỏ hơn thì A1 là head node;
3. So sánh A2 với B1. Giả sử B1 nhỏ hơn thì A1 trỏ đến B1;
4. So sánh A2 với B2;
5. Cứ lặp lại như vậy là được, cách này khá dễ hiểu.

Hãy thử triển khai bằng recursion!

### Solution

**Phiên bản recursion:**

```java
/*
public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}*/
//https://www.nowcoder.com/practice/d8b6b4358f774294a89de2a6ac4d9337?tpId=13&tqId=11169&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking
public class Solution {
  public ListNode Merge(ListNode list1, ListNode list2) {
    if (list1 == null) {
      return list2;
    }
    if (list2 == null) {
      return list1;
    }
    if (list1.val <= list2.val) {
      list1.next = Merge(list1.next, list2);
      return list1;
    } else {
      list2.next = Merge(list1, list2.next);
      return list2;
    }
  }
}
```

## Trọng tâm ôn tập phỏng vấn

Code của các bài toán linked list thường không dài, nhưng thứ tự cập nhật pointer rất dễ viết sai. Trước khi phỏng vấn, ít nhất cần nắm vững 4 template: dummy head node, đảo ngược linked list, fast/slow pointer và hợp nhất linked list.

| Template              | Dạng bài phù hợp                                             | Điểm mấu chốt                                                              |
| --------------------- | ------------------------------------------------------------ | -------------------------------------------------------------------------- |
| Dummy head node       | Xóa node, hợp nhất linked list, head node có thể thay đổi    | Trả về `dummy.next`                                                        |
| Đảo ngược linked list | Đảo ngược toàn bộ, đảo ngược một đoạn, đảo ngược từng nhóm K | Lưu `next`, sau đó sửa `cur.next`                                          |
| Fast/slow pointer     | Phát hiện cycle, node thứ K tính từ cuối, node giữa          | Kiểm tra `fast` và `fast.next` trước                                       |
| Hợp nhất linked list  | Hai linked list có thứ tự, K linked list có thứ tự           | Dùng recursion hoặc iteration, chú ý nối phần linked list còn lại vào đuôi |

Nên học thuộc template iteration của thao tác đảo ngược linked list:

```java
ListNode reverseList(ListNode head) {
    ListNode prev = null;
    ListNode cur = head;
    while (cur != null) {
        ListNode next = cur.next;
        cur.next = prev;
        prev = cur;
        cur = next;
    }
    return prev;
}
```

## Minh họa quá trình và ví dụ về trường hợp biên

Khi đảo ngược linked list, điều cốt lõi là lưu `next` trước rồi sửa `cur.next`. Có thể ghi nhớ qua thay đổi của pointer như sau:

```text
Ban đầu: prev = null, cur = head

Mỗi vòng:
next = cur.next
cur.next = prev
prev = cur
cur = next

Kết thúc: cur == null, prev trỏ đến head node mới
```

Với các bài toán như xóa node và hợp nhất linked list, hãy ưu tiên cân nhắc dummy head node:

```java
ListNode dummy = new ListNode(0);
dummy.next = head;
// Thực hiện thao tác thống nhất trên phần linked list sau dummy
return dummy.next;
```

Một số điểm dễ sai:

- Khi xóa node thứ N tính từ cuối, dummy head node giúp xử lý thống nhất trường hợp xóa head node.
- Khi đảo ngược một đoạn, trước hết phải lưu node trước đoạn và node sau đoạn.
- Khi kiểm tra linked list có cycle, điều kiện vòng lặp là `fast != null && fast.next != null`.
- Code hợp nhất linked list bằng recursion ngắn, nhưng khi linked list rất dài có thể có rủi ro tràn recursion stack.
- Cần kiểm tra riêng linked list rỗng, linked list chỉ có một node, xóa head node và xóa node cuối.

<!-- @include: @article-footer.snippet.md -->
