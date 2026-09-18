---
title: "Tổng hợp câu hỏi phỏng vấn bài toán Top K: heap, phân vùng quicksort, đếm bucket và data stream"
description: "Tổng hợp câu hỏi phỏng vấn bài toán Top K, giải thích phần tử lớn thứ K, K phần tử có tần suất cao nhất, min-heap, phân vùng quicksort, đếm bucket, median của data stream, PriorityQueue và các bài LeetCode thường gặp."
category: Computer Basics
tag:
  - Algorithms
head:
  - - meta
    - name: keywords
      content: TopK,Top K,Kth largest,Top K frequency,heap,min-heap,quicksort partition,bucket counting,PriorityQueue,median of data stream,LeetCode
---

Bài toán Top K rất thường gặp trong phỏng vấn backend, vì vừa có thể kiểm tra thuật toán, vừa dễ liên hệ với các tình huống engineering: bảng xếp hạng, thống kê từ khóa phổ biến, median của data stream, mã lỗi xuất hiện nhiều nhất trong log đều có thể quy về Top K.

Với nhóm bài này, không nên chỉ ghi nhớ một cách viết. Interviewer thường hỏi tiếp: Nếu dữ liệu rất lớn thì phải làm thế nào? Nếu là data stream thì sao? Nếu yêu cầu K phần tử có tần suất cao nhất thì sao? Phương án sẽ thay đổi theo từng điều kiện.

## Trọng tâm phỏng vấn

- Có thể dùng heap để giải bài phần tử lớn thứ K và K phần tử có tần suất cao nhất.
- Có thể giải thích rõ cách chọn min-heap và max-heap.
- Có thể so sánh complexity của heap, phân vùng quicksort và đếm bucket.
- Có thể xử lý tình huống data stream.
- Có thể viết comparator cho Java `PriorityQueue`.

## Chọn phương án cho bài Top K như thế nào?

Trước tiên xem 3 điều kiện:

1. Chỉ cần phần tử thứ K hay cần đầy đủ K phần tử đầu tiên?
2. Dữ liệu được cung cấp một lần hay là data stream liên tục đến?
3. Có cần kết quả có thứ tự hay không?

Nếu chỉ tìm phần tử lớn thứ K trong một array được cung cấp một lần, phân vùng quicksort trung bình nhanh hơn; nếu dữ liệu liên tục đến, duy trì một heap có kích thước K sẽ tự nhiên hơn; nếu đề bài hỏi K phần tử có tần suất cao nhất, trước tiên phải thống kê frequency, sau đó thực hiện Top K trên frequency.

## So sánh các phương án

| Phương án           | Tình huống phù hợp                                            | Time complexity          | Space complexity           |
| ------------------- | ------------------------------------------------------------- | ------------------------ | -------------------------- |
| Sorting             | Dữ liệu không lớn, ưu tiên code đơn giản                      | `O(nlogn)`               | Tùy implementation sorting |
| Min-heap            | Tìm K phần tử lớn nhất hoặc phần tử lớn thứ K                 | `O(nlogk)`               | `O(k)`                     |
| Phân vùng quicksort | Tìm phần tử lớn thứ K, hiệu suất trung bình cao               | Trung bình `O(n)`        | `O(1)` đến `O(logn)`       |
| Đếm bucket          | Phạm vi frequency có giới hạn, K phần tử có tần suất cao nhất | `O(n)`                   | `O(n)`                     |
| Hai heap            | Median của data stream                                        | Mỗi lần insert `O(logn)` | `O(n)`                     |

Trong phỏng vấn, có thể trả lời về trade-off như sau:

- Sorting đơn giản nhất, phù hợp khi dữ liệu không lớn hoặc không theo đuổi complexity tối ưu.
- Heap phù hợp với tình huống K nhỏ hơn n rất nhiều, chỉ cần space `O(k)`.
- Phân vùng quicksort phù hợp để tìm phần tử lớn thứ K trong dữ liệu được cung cấp một lần, trung bình `O(n)`, nhưng có thể suy biến trong trường hợp xấu nhất.
- Đếm bucket phù hợp với bài toán về frequency, đặc biệt khi phạm vi frequency không vượt quá `n`.

## Dùng min-heap để tìm phần tử lớn thứ K

```java
int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> heap = new PriorityQueue<>();
    for (int num : nums) {
        heap.offer(num);
        if (heap.size() > k) {
            heap.poll();
        }
    }
    return heap.peek();
}
```

Heap luôn giữ lại K số lớn nhất hiện tại, phần tử trên đỉnh heap là phần tử nhỏ nhất trong K số này, cũng chính là phần tử lớn thứ K trong toàn bộ array.

Vì sao là min-heap? Vì cần giữ lại K phần tử lớn nhất trong heap. Khi phần tử mới được thêm vào, nếu kích thước heap vượt quá K thì phải loại bỏ phần tử nhỏ nhất trong K + 1 phần tử này. Đỉnh của min-heap chính là giá trị nhỏ nhất.

Nếu tìm phần tử nhỏ thứ K, tư duy sẽ ngược lại: duy trì max-heap có kích thước K, khi vượt quá K thì lấy phần tử lớn nhất ra.

## Giải thích chi tiết bài tiêu biểu: K phần tử có tần suất cao nhất

[347. K phần tử có tần suất cao nhất](https://leetcode.cn/problems/top-k-frequent-elements/) là bài toán frequency phổ biến nhất trong nhóm Top K. Đề bài cho một integer array và một integer `k`, yêu cầu trả về `k` phần tử có frequency xuất hiện cao nhất, thứ tự kết quả thường không quan trọng.

Không nên sorting trực tiếp array ban đầu trong bài này, vì cần so sánh “frequency”, không phải giá trị phần tử. Cách tách ổn định hơn gồm hai bước:

1. Dùng `HashMap` để thống kê số lần xuất hiện của mỗi phần tử.
2. Duy trì một min-heap sắp xếp tăng dần theo frequency, trong heap chỉ giữ lại `k` phần tử có frequency cao nhất hiện tại.

Vì sao vẫn dùng min-heap? Vì sau khi heap đầy, khi phần tử mới được thêm vào, chỉ cần kích thước heap vượt quá `k` thì lấy phần tử có frequency thấp nhất hiện tại ra. Như vậy sau khi duyệt hết các phần tử khác nhau, những gì còn lại trong heap chính là K phần tử có frequency cao nhất.

```java
int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int num : nums) {
        freq.put(num, freq.getOrDefault(num, 0) + 1);
    }
    PriorityQueue<int[]> heap = new PriorityQueue<>(Comparator.comparingInt(a -> a[1]));
    for (Map.Entry<Integer, Integer> entry : freq.entrySet()) {
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

Ở đây heap sắp xếp tăng dần theo frequency, khi kích thước heap vượt quá K thì lấy phần tử có frequency nhỏ nhất ra.

Lấy `nums = [1,1,1,2,2,3]`, `k = 2` làm ví dụ, bảng frequency là `{1=3, 2=2, 3=1}`. Heap lần lượt đưa `1` và `2` vào, sau đó khi đưa `3` vào thì kích thước vượt quá 2, phần tử có frequency thấp nhất là `3` sẽ bị lấy ra, cuối cùng giữ lại `1` và `2`.

Nếu `k` bằng số phần tử khác nhau, cuối cùng heap sẽ giữ lại toàn bộ phần tử; nếu interviewer yêu cầu output theo thứ tự frequency giảm dần thì cuối cùng vẫn cần sorting kết quả bổ sung.

Nếu interviewer yêu cầu khi frequency bằng nhau thì sắp xếp theo kích thước phần tử hoặc thứ tự từ điển, comparator phải thêm quy tắc sorting thứ hai. Ví dụ, bài K từ có tần suất cao nhất thường yêu cầu từ có frequency cao hơn đứng trước, khi frequency bằng nhau thì từ có thứ tự từ điển nhỏ hơn đứng trước.

## Tư duy phân vùng quicksort

Phân vùng quicksort phù hợp để tìm phần tử lớn thứ K, khi không yêu cầu output K phần tử đầu tiên có thứ tự. Tư duy là mỗi lần chia array thành hai phía theo pivot, rồi quyết định tiếp tục tìm ở phía nào dựa trên thứ hạng của pivot. Time complexity trung bình là `O(n)`, nhưng trong trường hợp xấu nhất có thể suy biến thành `O(n^2)`, cách viết thực tế thường chọn pivot ngẫu nhiên.

Ưu điểm của phân vùng quicksort là không cần duy trì heap, time complexity trung bình thấp; hạn chế là phù hợp hơn với dữ liệu trong memory được cung cấp một lần. Nếu data stream liên tục đến hoặc dữ liệu quá lớn không thể đưa toàn bộ vào memory, phương án heap dễ triển khai hơn.

## Tình huống data stream

Với bài toán data stream, không thể sorting lại mỗi khi có một phần tử. Cách làm thường gặp là liên tục duy trì một data structure:

- Phần tử lớn thứ K trong data stream: duy trì min-heap có kích thước K.
- Median của data stream: duy trì hai heap, max-heap bên trái chứa nửa nhỏ hơn, min-heap bên phải chứa nửa lớn hơn.
- Median của sliding window: còn phải xử lý phần tử hết hạn; heap thông thường không thuận tiện để xóa phần tử bất kỳ, thường cần lazy deletion hoặc ordered set.

## Minh họa quy trình và ví dụ biên

Lấy array `[3, 2, 1, 5, 6, 4]` để tìm phần tử lớn thứ 2 làm ví dụ, duy trì min-heap có kích thước 2. Trong bảng, để dễ đọc, các phần tử trong heap được hiển thị theo thứ tự tăng dần của giá trị, không đại diện cho thứ tự array bên trong của Java `PriorityQueue`.

| Phần tử đọc vào | Phần tử ứng viên | Xử lý sau khi vượt quá K   |
| --------------- | ---------------- | -------------------------- |
| 3               | `[3]`            | Không xử lý                |
| 2               | `[2, 3]`         | Không xử lý                |
| 1               | `[1, 2, 3]`      | Lấy 1 ra, giữ lại `[2, 3]` |
| 5               | `[2, 3, 5]`      | Lấy 2 ra, giữ lại `[3, 5]` |
| 6               | `[3, 5, 6]`      | Lấy 3 ra, giữ lại `[5, 6]` |
| 4               | `[4, 5, 6]`      | Lấy 4 ra, giữ lại `[5, 6]` |

Cuối cùng đỉnh heap là `5`, cũng chính là phần tử lớn thứ 2.

Cách viết sai thường gặp:

```java
PriorityQueue<Integer> heap = new PriorityQueue<>((a, b) -> b - a);
```

Comparator này có thể overflow khi gặp giá trị integer cực hạn. Cách viết an toàn hơn là:

```java
PriorityQueue<Integer> heap = new PriorityQueue<>((a, b) -> Integer.compare(b, a));
```

## Điểm dễ nhầm

- Tìm K phần tử lớn nhất thường dùng min-heap, tìm K phần tử nhỏ nhất thường dùng max-heap.
- `PriorityQueue` mặc định là min-heap.
- Với K phần tử có frequency cao nhất, trước tiên phải thống kê frequency, sau đó thực hiện Top K trên frequency.
- Nếu cần output có thứ tự, sau heap hoặc phân vùng quicksort vẫn cần sorting bổ sung.
- Trong tình huống data stream, không thể sorting lại toàn bộ dữ liệu mỗi lần.

## Tự kiểm tra các câu hỏi thường gặp

- Vì sao khi tìm phần tử lớn thứ K thường duy trì min-heap có kích thước K?
- Min-heap và max-heap lần lượt phù hợp với những tình huống Top K nào?
- Time complexity và space complexity của phương án heap và phương án phân vùng quicksort khác nhau như thế nào?
- Vì sao với K phần tử có frequency cao nhất phải thống kê frequency trước?
- Vì sao median của data stream phù hợp với cách duy trì bằng hai heap?

## Bài tập đề xuất

- [215. Phần tử lớn nhất thứ K trong array](https://leetcode.cn/problems/kth-largest-element-in-an-array/)
- [347. K phần tử có tần suất cao nhất](https://leetcode.cn/problems/top-k-frequent-elements/)
- [692. K từ có tần suất cao nhất](https://leetcode.cn/problems/top-k-frequent-words/)
- [703. Phần tử lớn thứ K trong data stream](https://leetcode.cn/problems/kth-largest-element-in-a-stream/)
- [295. Median của data stream](https://leetcode.cn/problems/find-median-from-data-stream/)

<!-- @include: @article-footer.snippet.md -->
