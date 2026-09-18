---
title: "Tổng hợp câu hỏi phỏng vấn về skip list: multi-level index, range query và Redis ZSet"
description: "Tổng hợp câu hỏi phỏng vấn về skip list, giải thích multi-level index, query, insert, delete, complexity, range query, so sánh với red-black tree và triển khai bên trong Redis ZSet."
category: Computer Science Basics
tag:
  - Data Structures
head:
  - - meta
    - name: keywords
      content: skip list,SkipList,Redis ZSet,sorted set,range query,multi-level index,so sánh red-black tree,câu hỏi phỏng vấn Redis,câu hỏi phỏng vấn data structures
---

Skip list có thể được hiểu là “sorted linked list có multi-level index”. Query trên sorted linked list thông thường cần quét từ đầu đến cuối, complexity là `O(n)`; skip list thêm nhiều tầng index phía trên linked list, khi query có thể nhanh chóng bỏ qua một nhóm node từ tầng cao, sau đó lần lượt đi xuống.

Sorted set ZSet của Redis sử dụng kết hợp skip list và hash table ở tầng dưới, vì vậy skip list thường xuất hiện cùng Redis trong các buổi phỏng vấn backend.

Tổng quan nội dung:

1. Skip list là gì?
2. Vì sao skip list có thể giảm query từ `O(n)` xuống trung bình `O(logn)`?
3. Skip list tìm kiếm, insert và delete như thế nào?
4. Nên so sánh skip list và red-black tree như thế nào?
5. Vì sao Redis ZSet sử dụng skip list?

![Skip list xây dựng multi-level index trên sorted linked list để tăng tốc tìm kiếm](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/skip-list.png)

## Skip list là gì?

Skip list là một data structure dựa trên sorted linked list. Tầng dưới cùng vẫn là một sorted linked list hoàn chỉnh, mọi element đều xuất hiện ở tầng thấp nhất; phía trên linked list tầng dưới, skip list tiếp tục xây dựng một số tầng index thưa hơn.

Có thể hình dung nó như mục lục của một cuốn sách:

- Linked list ở tầng dưới cùng giống như các trang nội dung, thông tin đầy đủ nhất nhưng lật từ trang đầu đến trang cuối rất chậm.
- Index ở tầng trên giống như mục lục, thông tin ít hơn nhưng có thể nhanh chóng chuyển đến vị trí gần mục tiêu.
- Khi query, trước tiên đi sang phải từ index ở tầng cao nhất; khi không thể đi tiếp thì hạ xuống một tầng, cho đến tầng dưới cùng.

Đây là điểm khác biệt lớn nhất giữa skip list và linked list thông thường: linked list thông thường mỗi lần chỉ có thể đi tiếp một bước; skip list có thể bỏ qua nhiều node cùng lúc trong index ở tầng cao.

## Node của skip list trông như thế nào?

Node của linked list thông thường thường chỉ có một pointer `next`, còn node của skip list có nhiều pointer tiến. Nếu một node xuất hiện ở tầng 3 thì nghĩa là nó có pointer tương ứng ở các tầng 0, 1, 2.

Nhìn ở mức trừu tượng, node của skip list có thể được hiểu như sau:

```java
class SkipListNode {
    int value;
    SkipListNode[] forward;
}
```

Ở đây, `forward[i]` biểu thị node tiếp theo mà node hiện tại trỏ tới ở tầng `i`. Trong triển khai thực tế, còn có thể lưu score, member, pointer `backward`, span và các thông tin khác để hỗ trợ rank, duyệt ngược và range query.

## Số tầng được tạo ra như thế nào?

Skip list không duy trì balance bằng rotation hay recoloring, mà dựa vào số tầng ngẫu nhiên.

Khi insert một node mới, thông thường sẽ dùng random function để quyết định node đó có thể lên bao nhiêu tầng. Có thể hình dung quá trình này như tung đồng xu: node chắc chắn xuất hiện ở tầng thấp nhất; nếu lần tung đầu tiên là mặt ngửa thì lên một tầng; tiếp tục là mặt ngửa thì lên thêm một tầng; cho đến khi ra mặt sấp hoặc đạt số tầng tối đa.

Kết quả là: node ở tầng càng cao càng ít, node ở tầng càng thấp càng dày. Trong trường hợp lý tưởng, tầng 1 giữ lại khoảng một nửa số node, tầng 2 lại giữ một nửa số đó, tầng 3 tiếp tục giảm. Dù không balance tuyệt đối, xét theo xác suất thì height sẽ duy trì ở cấp `O(logn)`.

Đây cũng là nguồn gốc của chữ “skip” trong tên skip list: index ở tầng cao cho phép quá trình query bỏ qua từng đoạn element.

## Trọng tâm phỏng vấn

- Giải thích rõ vì sao skip list query nhanh hơn linked list thông thường.
- Mô tả được quy trình cơ bản của find, insert và delete.
- Nêu được query, insert và delete trung bình của skip list là `O(logn)`.
- Giải thích được sự đánh đổi giữa skip list và red-black tree.
- Liên hệ với trường hợp sử dụng range query của Redis ZSet.

## Skip list tìm kiếm như thế nào?

Khi tìm kiếm, bắt đầu từ index ở tầng cao nhất:

1. Nếu node tiếp theo của node hiện tại nhỏ hơn target value thì đi sang phải.
2. Nếu node tiếp theo lớn hơn target value hoặc là `null` thì hạ xuống một tầng.
3. Sau khi xuống tầng thấp nhất thì tiếp tục tìm target node.

Quá trình này hơi giống binary search trên sorted array, nhưng tầng dưới cùng của skip list vẫn là linked list.

Nói chính xác hơn, mỗi tầng của skip list đều là một sorted linked list. Khi query luôn tuân theo một nguyên tắc: đi sang phải được thì đi sang phải, không đi sang phải được thì đi xuống.

Giả sử cần tìm `26`:

1. Bắt đầu từ head node ở tầng cao nhất.
2. Nếu giá trị node bên phải nhỏ hơn `26`, target vẫn còn ở bên phải, có thể tiếp tục dịch phải.
3. Nếu giá trị node bên phải lớn hơn `26`, đi tiếp sang phải sẽ vượt qua target, nên hạ xuống một tầng.
4. Lặp lại quá trình này, cuối cùng xác nhận target có tồn tại ở tầng dưới cùng hay không.

Nếu target không tồn tại, skip list cũng có thể tìm được vị trí mà nó nên được insert: node cuối cùng ở tầng dưới cùng có giá trị nhỏ hơn target chính là predecessor node của vị trí insert.

## Insert và delete

Khi insert một node, trước tiên cần tìm predecessor node của nó ở mỗi tầng, sau đó nối node mới vào. Node mới có thể lên bao nhiêu tầng thường do random function quyết định.

Khi delete node, cũng cần tìm predecessor node ở mỗi tầng, sau đó để pointer bỏ qua target node.

Skip list không duy trì balance bằng rotation mà dùng số tầng ngẫu nhiên để giữ height của index trong phạm vi hợp lý. Đây cũng là lý do nó dễ triển khai hơn red-black tree.

Khi viết code insert thực tế, thường sẽ duy trì một array `update`: `update[i]` biểu thị node mà sau nó node mới sẽ được insert ở tầng `i`. Trong quá trình tìm vị trí insert, các predecessor node này được ghi lại luôn; sau khi có số tầng ngẫu nhiên thì có thể lần lượt sửa pointer.

Delete cũng tương tự: trước tiên tìm predecessor node ở mỗi tầng; nếu node tiếp theo ở tầng đó đúng là target node thì trỏ `forward` của predecessor node tới node tiếp theo của target node.

Vì vậy, insert và delete của skip list không chỉ sửa linked list ở tầng dưới cùng mà còn phải đồng bộ duy trì các tầng index mà target node từng xuất hiện.

## Complexity

| Operation   | Complexity trung bình | Mô tả                                                       |
| ----------- | --------------------- | ----------------------------------------------------------- |
| Find        | `O(logn)`             | Bỏ qua node thông qua multi-level index                     |
| Insert      | `O(logn)`             | Update nhiều tầng pointer sau khi tìm vị trí                |
| Delete      | `O(logn)`             | Ngắt pointer sau khi tìm predecessor                        |
| Range query | `O(logn + k)`         | Định vị start point trước, sau đó lần lượt trả về k element |

Space complexity ở cấp `O(n)`, nhưng sẽ có thêm một số index pointer so với linked list thông thường.

`O(logn)` ở đây là complexity theo nghĩa trung bình, phụ thuộc vào probabilistic balance do số tầng ngẫu nhiên mang lại. Skip list không đưa ra ràng buộc balance nghiêm ngặt ở worst case như red-black tree, nhưng khi random function hoạt động bình thường và parameter được thiết lập hợp lý, performance thường rất ổn định.

Range query là một trường hợp skip list xử lý rất tốt: trước tiên dùng `O(logn)` để định vị start point của range, sau đó duyệt tuần tự `k` result về phía sau theo linked list ở tầng dưới cùng.

## Chọn skip list hay red-black tree?

| Điểm so sánh               | Skip list                                       | Red-black tree                                                    |
| -------------------------- | ----------------------------------------------- | ----------------------------------------------------------------- |
| Cách balance               | Số tầng ngẫu nhiên                              | Rotation và recoloring                                            |
| Độ khó triển khai          | Tương đối trực tiếp                             | Việc fix sau insert và delete phức tạp hơn                        |
| Range query                | Quét theo linked list tầng dưới, rất thuận tiện | Inorder traversal cũng làm được nhưng cách triển khai rắc rối hơn |
| Complexity worst case      | Phụ thuộc tính ngẫu nhiên                       | Có ràng buộc balance nghiêm ngặt                                  |
| Đại diện trong engineering | Redis ZSet                                      | Java `TreeMap`, tree hóa `HashMap`                                |

## Vì sao Redis ZSet sử dụng skip list?

ZSet cần hỗ trợ:

- Query score nhanh theo member.
- Sort theo score.
- Range query theo score.
- Lấy rank.

Hash table phù hợp để query score theo member, còn skip list phù hợp để sort và range query theo score. Sau khi kết hợp, ZSet có thể đồng thời hỗ trợ find nhanh và duyệt có thứ tự.

Cụ thể hơn:

- Thông qua hash table, có thể tìm trực tiếp score tương ứng theo member.
- Thông qua skip list, có thể duy trì thứ tự từ nhỏ đến lớn theo score.
- Khi thực hiện các range query như `ZRANGE`, `ZRANGEBYSCORE`, skip list có thể định vị start point trước, sau đó liên tục trả về result theo linked list.
- Nếu node của skip list duy trì thông tin span thì còn có thể hỗ trợ các thao tác liên quan đến rank.

Cần lưu ý rằng Redis sẽ sử dụng các internal encoding khác nhau tùy theo data size và config để tiết kiệm memory. Trong phỏng vấn, câu “ZSet sử dụng hash table + skip list” thường là đang nói đến core structure của nó khi xử lý sorted set có quy mô lớn.

## Điểm dễ sai

- Skip list không phải array cũng không phải binary tree; tầng dưới cùng của nó là linked list.
- Complexity trung bình của skip list là `O(logn)`, không được bảo đảm bằng balance nghiêm ngặt.
- Range query là thế mạnh của skip list: định vị start point trước, sau đó duyệt theo linked list ở tầng dưới cùng.
- Redis ZSet không chỉ sử dụng skip list mà còn kết hợp với hash table.

## Tự kiểm tra các câu hỏi thường gặp

- Vì sao skip list query nhanh?
- Skip list và red-black tree khác nhau như thế nào?
- Vì sao Redis ZSet không sử dụng red-black tree?
- Số tầng của skip list được quyết định như thế nào?
- Complexity của skip list khi range query là bao nhiêu?

## Tài liệu tham khảo

- [Skip Lists: A Probabilistic Alternative to Balanced Trees](https://dl.acm.org/doi/10.1145/78973.78977)
- [William Pugh: A Skip List Cookbook](https://drum.lib.umd.edu/bitstreams/17176ef8-8330-4a6c-8b75-4cd18c570bec/download)
- [Redis Docs: Sorted Sets](https://redis.io/docs/latest/develop/data-types/sorted-sets/)
- [Redis source code: t_zset.c](https://github.com/redis/redis/blob/unstable/src/t_zset.c)

<!-- @include: @article-footer.snippet.md -->
