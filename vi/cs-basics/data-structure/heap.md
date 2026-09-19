---
title: Giải thích chi tiết về heap (max heap, min heap và priority queue)
description: Phân tích tính chất và thao tác của heap, tìm hiểu cách triển khai priority queue, ưu thế về performance của heap sort, độ phức tạp của thao tác insert/delete và các trường hợp sử dụng thực tế.
category: Computer Science Basics
tag:
  - Data Structure
head:
  - - meta
    - name: keywords
      content: heap,max heap,min heap,priority queue,heapify,sift up,sift down,heap sort
---

## Heap là gì

Heap là một tree thỏa mãn các điều kiện sau:

Giá trị của mỗi node trong heap lớn hơn hoặc bằng (hoặc nhỏ hơn hoặc bằng) giá trị của mọi node trong subtree. Nói cách khác, giá trị của bất kỳ node nào cũng lớn hơn hoặc bằng (hoặc nhỏ hơn hoặc bằng) giá trị của mọi child node.

> Bạn có thể hình dung heap (max heap) như một công ty rất công bằng: người có năng lực sẽ làm leader, không có chuyện người yếu làm leader; những người dưới quyền chắc chắn không thể giỏi hơn leader. Cách hình dung này giúp bạn nắm được các thao tác của heap ở phần sau.

**!!!Lưu ý đặc biệt:**

- Nhiều blog cho rằng heap là complete binary tree, nhưng thực tế không phải vậy, **heap không nhất thiết là complete binary tree**. Chỉ vì thuận tiện cho việc lưu trữ và index, chúng ta thường biểu diễn heap dưới dạng complete binary tree. Trên thực tế, Fibonacci heap và binomial heap nổi tiếng không phải là complete binary tree, thậm chí còn không phải là binary tree.
- (Binary) heap là một array, có thể được xem như một **complete binary tree gần đúng**. — _Introduction to Algorithms_, phiên bản thứ ba

Bạn có thể thử xác định các hình dưới đây có phải là heap hay không.

![Ví dụ kiểm tra có thỏa mãn tính chất heap hay không](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-1.png)

Hình 1 và hình 2 là heap. Hình 1 là max heap, mỗi node đều lớn hơn mọi node trong subtree. Hình 2 là min heap, mỗi node đều nhỏ hơn mọi node trong subtree.

Hình 3 không phải heap. Trong hình 3, root node 1 nhỏ hơn 2 và 15, nhưng 15 lại lớn hơn 3; 19 lớn hơn 5, nên không thỏa mãn tính chất của heap.

## Công dụng của heap

Khi chỉ quan tâm đến phần tử lớn nhất hoặc nhỏ nhất trong toàn bộ dữ liệu, cần nhiều lần lấy phần tử lớn nhất hoặc nhỏ nhất, hay nhiều lần insert hoặc delete dữ liệu, bạn có thể sử dụng heap.

Bạn có thể nghĩ đến ordered array. Khi khởi tạo ordered array, time complexity là `O(nlog(n))`; khi tìm phần tử lớn nhất hoặc nhỏ nhất, time complexity đều là `O(1)`. Tuy nhiên, khi update (insert hoặc delete) dữ liệu, time complexity là `O(n)`. Dù dùng binary search có time complexity `O(log(n))` để tìm vị trí cần insert hoặc delete, việc di chuyển dữ liệu vẫn cần `O(n)`.

**So với ordered array, ưu thế chính của heap là performance của thao tác insert và delete tốt hơn.** Vì heap được triển khai dựa trên complete binary tree, khi insert và delete dữ liệu, chỉ cần di chuyển node lên xuống trong binary tree, time complexity là `O(log(n))`, hiệu quả hơn `O(n)` của ordered array.

Tuy nhiên, cần lưu ý: khi dùng Floyd build-heap method và thực hiện sift down lần lượt từ non-leaf node cuối cùng, time complexity là `O(n)`; nếu bắt đầu từ empty heap và insert lần lượt n phần tử, time complexity là `O(nlogn)`.

## Phân loại heap

Heap được chia thành **max heap** và **min heap**. Điểm khác nhau giữa hai loại nằm ở cách sắp xếp các node.

- **Max heap**: giá trị của mỗi node trong heap lớn hơn hoặc bằng giá trị của mọi node trong subtree
- **Min heap**: giá trị của mỗi node trong heap nhỏ hơn hoặc bằng giá trị của mọi node trong subtree

Như hình dưới đây, hình 1 là max heap, hình 2 là min heap.

![Ví dụ về max heap và min heap](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-2.png)

## Lưu trữ heap

Khi giới thiệu về tree, chúng ta đã nói rằng nhờ các tính chất ưu việt của complete binary tree, dùng array để lưu binary tree vừa tiết kiệm không gian, vừa thuận tiện cho việc index (nếu số thứ tự của root node là 1, thì với mọi node i trong tree, số thứ tự của left child node là `2*i`, right child node là `2*i+1`).

Để thuận tiện cho việc lưu trữ và index, (binary) heap có thể được lưu trữ dưới dạng complete binary tree. Cách lưu trữ như hình dưới đây:

![Minh họa thứ tự lưu trữ heap trong array](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-storage.png)

## Các thao tác của heap

Các thao tác update của heap chủ yếu gồm hai loại: **insert element** và **delete heap top**. Cần nắm vững và hiểu rõ quy trình thực hiện.

> Trước khi đi vào nội dung chính, hãy nhắc lại một lần nữa: heap là một công ty công bằng, người có năng lực tự nhiên sẽ đến vị trí phù hợp với năng lực của mình.

### Insert element

> Insert element: một nhân viên mới gia nhập công ty; lúc đầu phải làm từ cấp cơ sở.

**1. Đặt element cần insert vào cuối**

![Insert element vào heap: element mới được đặt ở cuối array](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-insert-1.png)

> Người có năng lực sẽ dần được thăng chức và tăng lương; vàng thì rồi sẽ tỏa sáng!

**2. Từ dưới lên trên, nếu parent node nhỏ hơn element đó thì đổi chỗ element với parent node, cho đến khi không thể đổi chỗ**

![Insert element vào heap: so sánh element mới với parent node và sift up](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-insert-2.png)

![Insert element vào heap: khôi phục tính chất heap sau khi sift up](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-insert-3.png)

### Delete heap top

Theo tính chất của heap, heap top của max heap là phần tử lớn nhất, còn heap top của min heap là phần tử nhỏ nhất. Khi cần tìm phần tử lớn nhất hoặc nhỏ nhất nhiều lần, có thể dùng heap để thực hiện.

Sau khi delete heap top, để duy trì tính chất của heap, cần điều chỉnh cấu trúc heap. Quá trình này được gọi là “**heapify**”. Có hai hướng điều chỉnh phổ biến:

- Heapify từ dưới lên: khi insert element, element mới di chuyển lên từ cuối, còn gọi là sift up.
- Heapify từ trên xuống: khi delete heap top, element cuối di chuyển từ heap top xuống dưới, còn gọi là sift down.

#### Quá trình đưa hole lên chưa hoàn chỉnh

> Trong công ty heap, có thể xảy ra chuyện leader nghỉ việc. Sau khi leader nghỉ việc, vị trí của leader sẽ bị bỏ trống.

Trước tiên, hãy xem một cách làm dễ nghĩ đến nhưng chưa hoàn chỉnh: trực tiếp delete heap top, để trống vị trí có index 1 trong array.

![Delete heap top: xóa root node](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-delete-top-1.png)

> Vậy ai sẽ thay thế vị trí đó? Tất nhiên là cấp dưới trực tiếp của leader. Ai có năng lực hơn thì để người đó lên.

So sánh left child node và right child node của root node, tức các phần tử trong array tại index 2 và 3, rồi đưa phần tử lớn hơn vào vị trí của root node (index 1).

![Delete heap top: đưa child node lớn hơn lên root node](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-delete-top-2.png)

> Lúc này lại có một vị trí bị bỏ trống; vẫn như cũ, ai có năng lực thì người đó lên.

Liên tục so sánh left child node và right child node của vị trí bị bỏ trống, đưa node lớn hơn vào vị trí trống cho đến khi đến đáy heap.

![Delete heap top: để lại vị trí trống sau khi heapify từ dưới lên](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-delete-top-3.png)

Lúc này, dù child node lớn hơn đã di chuyển lên theo đường đi, nhưng bên trong array vẫn còn vị trí trống, không còn thỏa mãn cấu trúc của complete binary tree, vì vậy không thể xem đây là một lần delete heap top hoàn chỉnh và hợp lệ. Cách chuẩn là dùng element cuối để lấp vào heap top trước, sau đó điều chỉnh nó xuống dưới.

#### Heapify từ trên xuống

Heapify từ trên xuống có thể được mô tả bằng cụm từ “đá chìm xuống biển”. Việc đầu tiên là nhấc viên đá lên rồi ném xuống từ mặt biển. Viên đá này chính là element cuối của heap; chúng ta di chuyển element cuối lên heap top.

![Delete heap top: di chuyển element cuối lên heap top](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-delete-top-4.png)

Sau đó để viên đá này chìm xuống đáy biển, liên tục so sánh giá trị của nó với left child node và right child node, đổi chỗ với child node lớn hơn cho đến khi không thể đổi chỗ.

![Delete heap top: điều chỉnh heap top xuống dưới](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-delete-top-5.png)

![Delete heap top: hoàn tất heapify từ trên xuống](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-delete-top-6.png)

### Tổng hợp thao tác của heap

- **Insert element**: trước tiên đặt element vào cuối array, sau đó heapify từ dưới lên để element cuối sift up
- **Delete heap top**: đổi chỗ heap top với element cuối, thu nhỏ kích thước heap, sau đó điều chỉnh từ heap top xuống dưới cho đến khi khôi phục tính chất của heap.

## Heap sort

Quy trình heap sort gồm hai bước:

- Bước đầu tiên là build heap, biến một array chưa được sắp xếp thành heap
- Bước thứ hai là sort, lấy heap top ra, sau đó heapify các element còn lại; lặp lại cho đến khi lấy hết tất cả element.

### Build heap

Nếu đã hiểu quy trình heapify, bạn sẽ dễ nắm được quy trình build heap hơn. Quy trình build heap chính là heapify từ trên xuống cho tất cả non-leaf node.

Trước tiên cần biết non-leaf node là những node nào. Parent node của node cuối cùng và các node đứng trước nó đều là non-leaf node. Nói cách khác, nếu số node là n, chúng ta cần heapify từ trên xuống (đưa xuống đáy) các node từ n/2 đến 1.

Quy trình cụ thể như hình dưới đây:

![Quy trình build heap: complete binary tree tương ứng với array ban đầu không có thứ tự](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-build-1.png)

Trừu tượng hóa array ban đầu chưa được sắp xếp thành một tree. Số node trong hình là 6, nên node 4, 5, 6 là leaf node, node 1, 2, 3 là non-leaf node. Vì vậy cần heapify từ trên xuống các node 1-3. Lưu ý, thứ tự heapify là từ sau ra trước, bắt đầu từ node 3 cho đến node 1.

Kết quả heapify node 3:

![Quy trình build heap: node 3 hoàn tất sift down](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-build-2.png)

Kết quả heapify node 2:

![Quy trình build heap: node 2 hoàn tất sift down](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-build-3.png)

Kết quả heapify node 1:

![Quy trình build heap: node 1 hoàn tất sift down và hình thành max heap](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-build-4.png)

Đến đây, tree tương ứng với array đã trở thành max heap, build heap hoàn tất!

### Sort

Vì heap top là phần tử lớn nhất, nên chỉ cần lặp lại việc lấy heap top, đặt phần tử lớn nhất này vào cuối array, rồi heapify các element còn lại.

Bây giờ hãy suy nghĩ về hai câu hỏi:

- Sau khi delete heap top, cần thực hiện heapify từ trên xuống (đưa xuống đáy) hay heapify từ dưới lên (sift up)?
- Đặt heap top đã lấy ra ở đâu, có cần tạo array mới không?

Trước tiên trả lời câu hỏi thứ nhất: cần thực hiện heapify từ trên xuống (đưa xuống đáy). Bước đầu của heapify này là di chuyển element cuối lên heap top, khi đó vị trí cuối sẽ trống. Vì số element trong heap đã giảm, vị trí này sẽ không được dùng lại, nên có thể đặt element đã lấy ra vào cuối.

Bạn đã nhận ra chưa? Thực ra đây chính là một thao tác swap: đổi chỗ heap top và element cuối, từ đó gộp việc lấy heap top với bước đầu của heapify (đưa element cuối vào vị trí root node).

Quy trình chi tiết như hình dưới đây:

Lấy element đầu tiên và heapify:

![Quy trình heap sort: lấy heap top và heapify ở vòng 1](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-sort-1.png)

Lấy element thứ hai và heapify:

![Quy trình heap sort: lấy heap top và heapify ở vòng 2](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-sort-2.png)

Lấy element thứ ba và heapify:

![Quy trình heap sort: lấy heap top và heapify ở vòng 3](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-sort-3.png)

Lấy element thứ tư và heapify:

![Quy trình heap sort: lấy heap top và heapify ở vòng 4](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-sort-4.png)

Lấy element thứ năm và heapify:

![Quy trình heap sort: lấy heap top và heapify ở vòng 5](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-sort-5.png)

Lấy element thứ sáu và heapify:

![Quy trình heap sort: tất cả element đã được sort](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-sort-6.png)

Heap sort hoàn tất!

## Trọng tâm ôn tập phỏng vấn

Trong phỏng vấn, heap thường được hỏi cùng với priority queue, Top K, scheduled task và delayed queue.

| Thao tác        | Time complexity | Mô tả                                                                |
| --------------- | --------------- | -------------------------------------------------------------------- |
| Xem heap top    | `O(1)`          | Heap top của max heap là lớn nhất, heap top của min heap là nhỏ nhất |
| Insert element  | `O(logn)`       | Sift up sau khi insert vào cuối                                      |
| Delete heap top | `O(logn)`       | Sift down sau khi đưa element cuối lên heap top                      |
| Build heap      | `O(n)`          | Sift down bắt đầu từ non-leaf node cuối cùng                         |
| Heap sort       | `O(nlogn)`      | In-place sort nhưng không stable                                     |

Trong Java, `PriorityQueue` mặc định là min heap:

```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
PriorityQueue<Integer> maxHeap = new PriorityQueue<>((a, b) -> Integer.compare(b, a));
```

Không nên viết thành `b - a`, vì với các giá trị integer cực trị, phép tính có thể overflow và dẫn đến kết quả so sánh sai.

Các lựa chọn thường gặp cho bài toán Top K:

- Tìm phần tử lớn thứ K trong trường hợp tổng quát hoặc data stream: duy trì min heap có size K, time complexity là `O(nlogk)`, space complexity là `O(k)`.
- Tìm phần tử lớn thứ K trong array một lần và yêu cầu linear time: dùng quickselect, time complexity trung bình là `O(n)`.
- Tìm K phần tử có frequency cao nhất: trước tiên dùng hash table để đếm, sau đó dùng min heap giữ lại K phần tử có frequency cao nhất.
- Median của data stream: dùng một max heap duy trì nửa nhỏ hơn, một min heap duy trì nửa lớn hơn.

Cần lưu ý, min heap là cách giải tổng quát để tìm phần tử lớn thứ K, nhưng không đáp ứng yêu cầu complexity của mọi bài toán. Ví dụ, [LeetCode 215. Phần tử lớn thứ K trong array](https://leetcode.cn/problems/kth-largest-element-in-an-array/) yêu cầu dùng thuật toán có time complexity `O(n)`, thông thường nên dùng randomized quickselect, time complexity kỳ vọng là `O(n)`; nếu yêu cầu nghiêm ngặt worst-case time complexity là `O(n)`, cần dùng thuật toán BFPRT. Có thể tham khảo so sánh các phương án cụ thể trong [Tổng hợp câu hỏi phỏng vấn về bài toán Top K](../algorithms/top-k.md#快排分区思路).

## Mẫu code Java

Bài toán tìm phần tử lớn thứ K có thể dùng min heap có size K. Heap top luôn là phần tử nhỏ nhất trong K phần tử lớn nhất hiện tại. Nếu element mới lớn hơn heap top thì thay thế heap top. Cách viết này có time complexity `O(nlogk)`, space complexity `O(k)`, phù hợp với Top K tổng quát và các trường hợp data stream.

```java
int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> heap = new PriorityQueue<>();
    for (int num : nums) {
        if (heap.size() < k) {
            heap.offer(num);
        } else if (num > heap.peek()) {
            heap.poll();
            heap.offer(num);
        }
    }
    return heap.peek();
}
```

K phần tử có frequency cao nhất thường dùng cách “đếm bằng hash table + min heap”:

```java
int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> count = new HashMap<>();
    for (int num : nums) {
        count.put(num, count.getOrDefault(num, 0) + 1);
    }
    PriorityQueue<int[]> heap = new PriorityQueue<>((a, b) -> Integer.compare(a[1], b[1]));
    for (Map.Entry<Integer, Integer> entry : count.entrySet()) {
        heap.offer(new int[] {entry.getKey(), entry.getValue()});
        if (heap.size() > k) {
            heap.poll();
        }
    }
    int[] ans = new int[k];
    for (int i = k - 1; i >= 0; i--) {
        ans[i] = heap.poll()[0];
    }
    return ans;
}
```

## Minh họa quy trình và các ví dụ biên

Khi duy trì min heap có size K, có thể xem heap là một “candidate pool”:

```text
1. Candidate pool chưa đầy: đưa element vào ngay.
2. Candidate pool đã đầy, element mới <= heap top: không lọt vào top K, bỏ qua.
3. Candidate pool đã đầy, element mới > heap top: lấy heap top ra, đưa element mới vào.
4. Sau khi duyệt xong, heap top là phần tử lớn thứ K.
```

Nên kiểm tra trước một số ví dụ biên:

- `k == 1`: tìm phần tử lớn nhất.
- `k == nums.length`: tìm phần tử nhỏ nhất.
- Array có phần tử trùng lặp: phần tử lớn thứ K thường được tính theo vị trí trong thứ tự sort, không phải phần tử khác biệt thứ K.
- Không nên viết comparator là `b - a`, vì giá trị cực trị có thể overflow.

## Bài tập đề xuất

- [215. Phần tử lớn thứ K trong array](https://leetcode.cn/problems/kth-largest-element-in-an-array/) (đề bài yêu cầu time complexity `O(n)`, ưu tiên luyện quickselect)
- [347. K phần tử có frequency cao nhất](https://leetcode.cn/problems/top-k-frequent-elements/)
- [703. Phần tử lớn thứ K trong data stream](https://leetcode.cn/problems/kth-largest-element-in-a-stream/)
- [295. Median của data stream](https://leetcode.cn/problems/find-median-from-data-stream/)

<!-- @include: @article-footer.snippet.md -->
