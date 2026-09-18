---
title: "Tổng hợp câu hỏi phỏng vấn về union-find: path compression, tính liên thông và Java template"
description: "Tổng hợp câu hỏi phỏng vấn về union-find, giải thích Union Find, find, union, path compression, union by size, tính liên thông, cycle detection, số lượng tỉnh và các bài LeetCode thường gặp."
category: Computer Basics
tag:
  - Data Structures
head:
  - - meta
    - name: keywords
      content: union-find,Union Find,path compression,union by size,connectivity,graph algorithms,cycle detection,number of provinces,Java union-find,LeetCode
---

Union-find chuyên giải quyết các vấn đề về “phân nhóm” và “tính liên thông”. Hai phần tử có thuộc cùng một nhóm không? Sau khi hợp nhất hai set thì còn bao nhiêu connected components? Thêm một edge vào graph có tạo thành cycle không? Tất cả đều có thể được xử lý bằng union-find.

Trong phỏng vấn, code của nó không dài, nhưng nếu viết `find` không tốt thì sẽ ảnh hưởng trực tiếp đến complexity.

Tổng quan nội dung:

1. Union-find là gì?
2. Union-find biểu diễn set bằng array như thế nào?
3. `find`, `union`, `connected` lần lượt làm gì?
4. Vì sao path compression và union by size có thể tăng tốc?
5. Union-find phù hợp với những vấn đề về tính liên thông nào?

![Forest structure của các connected components được biểu diễn bằng parent pointer trong union-find](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/union-find.png)

## Union-find là gì?

Union-find (Disjoint Set Union, DSU, còn gọi là Union Find) duy trì một nhóm các set không giao nhau. Nó đặc biệt giỏi trả lời hai loại vấn đề:

1. **Truy vấn**: hiện tại hai phần tử có thuộc cùng một set không?
2. **Hợp nhất**: hợp nhất hai set chứa hai phần tử thành một set.

Nó không quan tâm đến cấu trúc đầy đủ bên trong set, cũng không quan tâm giữa hai node cụ thể đã đi qua những edge nào. Ví dụ trong quan hệ xã hội, union-find có thể nhanh chóng cho bạn biết A và B có thuộc cùng một network quan hệ không; nhưng nó không cho bạn biết shortest path từ A đến B là gì.

Đây cũng là điểm khác nhau giữa union-find và BFS/DFS: BFS/DFS giống như mỗi lần đều search trực tiếp dọc theo graph; còn union-find duy trì quan hệ liên thông trong quá trình hợp nhất, để các truy vấn sau đó chỉ cần kiểm tra representative node của hai phần tử có giống nhau không.

## Union-find biểu diễn set như thế nào?

Union-find thường dùng một array `parent` để biểu diễn một forest gồm nhiều tree:

- `parent[x]` biểu thị parent node của phần tử `x`.
- Nếu `parent[x] == x`, nghĩa là `x` là root node của set tương ứng.
- Một set chỉ cần dùng root node làm representative.

Khi khởi tạo, mỗi phần tử là một set riêng biệt, nên parent node của mỗi phần tử chính là nó:

```text
parent[0] = 0
parent[1] = 1
parent[2] = 2
...
```

Sau khi thực hiện `union(0, 1)`, có thể gắn root node của `1` vào bên dưới root node của `0`. Khi đó `0` và `1` thuộc cùng một set. Tiếp tục thực hiện `union(1, 2)`, dù truyền vào `1` và `2`, nhưng thứ thực sự được hợp nhất là root node của `1` và root node của `2`.

Vì vậy, điểm mấu chốt trong union-find không phải là “parent node hiện tại của node là ai”, mà là “đi dọc theo các parent node lên trên, cuối cùng root node là ai”. `find(x)` thực hiện chính việc này.

## Ba thao tác cốt lõi

Các thao tác thường gặp của union-find có thể khái quát thành ba thao tác:

| Thao tác          | Tác dụng                                                          |
| ----------------- | ----------------------------------------------------------------- |
| `find(x)`         | Tìm representative node của set chứa `x`, cũng chính là root node |
| `union(a, b)`     | Hợp nhất hai set chứa `a` và `b`                                  |
| `connected(a, b)` | Kiểm tra representative node của `a` và `b` có giống nhau không   |

Nếu root node của hai phần tử giống nhau, nghĩa là chúng đã thuộc cùng một set; nếu root node khác nhau, `union` sẽ gắn một root node vào bên dưới root node còn lại.

## Trọng tâm phỏng vấn

- Có thể viết `find` và `union`.
- Có thể giải thích tác dụng của path compression.
- Có thể dùng union-find để đếm connected components.
- Có thể xử lý cycle detection trong graph, bài toán friend circle, số lượng tỉnh và quan hệ đẳng thức.
- Có thể giải thích union-find phù hợp với dynamic merge nhưng không phù hợp với việc xóa thường xuyên.

## Từ Quick Find đến Quick Union

Khi tìm hiểu union-find, có thể xem trước hai phiên bản cực đoan:

- **Quick Find**: array lưu trực tiếp mã set mà mỗi phần tử thuộc về. Truy vấn hai phần tử có cùng nhóm rất nhanh, nhưng khi hợp nhất hai set thì phải quét toàn bộ array để sửa mã set.
- **Quick Union**: array lưu parent node và dùng root node để đại diện cho set. Khi hợp nhất chỉ cần sửa parent pointer của một root node, nhưng nếu tree quá cao thì `find` sẽ chậm hơn.

Trong phỏng vấn và làm bài, phiên bản tối ưu của Quick Union thường được dùng: **path compression + union by size/rank**.

- **Path compression**: mỗi lần `find(x)`, gắn trực tiếp các node trên đường đi vào bên dưới root node, để những lần truy vấn sau nhanh hơn.
- **Union by size**: khi hợp nhất hai set, gắn tree nhỏ vào bên dưới tree lớn để hạn chế chiều cao của tree.

Kết hợp hai tối ưu này có thể đưa nhiều thao tác của union-find xuống thời gian rất gần hằng số.

## Template cơ bản

```java
class UnionFind {
    private final int[] parent;
    private final int[] size;
    private int count;

    UnionFind(int n) {
        parent = new int[n];
        size = new int[n];
        count = n;
        for (int i = 0; i < n; i++) {
            parent[i] = i;
            size[i] = 1;
        }
    }

    int find(int x) {
        if (parent[x] != x) {
            parent[x] = find(parent[x]);
        }
        return parent[x];
    }

    boolean union(int a, int b) {
        int rootA = find(a);
        int rootB = find(b);
        if (rootA == rootB) {
            return false;
        }
        if (size[rootA] < size[rootB]) {
            parent[rootA] = rootB;
            size[rootB] += size[rootA];
        } else {
            parent[rootB] = rootA;
            size[rootA] += size[rootB];
        }
        count--;
        return true;
    }

    boolean connected(int a, int b) {
        return find(a) == find(b);
    }

    int count() {
        return count;
    }
}
```

`parent[x]` biểu thị parent node của `x`. Parent node của root node là chính nó. Path compression sẽ khiến các node trên đường tìm kiếm được gắn trực tiếp vào bên dưới root node, giúp những lần truy vấn sau nhanh hơn.

Template này có hai chi tiết đáng xem riêng:

1. `parent[x] = find(parent[x])` trong `find()` là path compression. Sau khi recursion trả về root node, nó đồng thời nối trực tiếp `x` với root node.
2. `union()` dùng `size` để quyết định gắn node nào vào bên dưới node nào, đây là union by size. Cách này giúp giảm mức tăng chiều cao của tree.

`count` biểu thị số connected components hiện còn. Mỗi lần `union()` thực sự hợp nhất hai set vốn không liên thông, `count` mới giảm 1; nếu hai phần tử vốn đã liên thông thì không được giảm lặp lại.

## Complexity

Sau khi dùng path compression và union by size, amortized complexity của một thao tác union-find là `O(α(n))`, trong đó `α(n)` là inverse Ackermann function, tăng cực kỳ chậm. Trong phỏng vấn thực tế, thường chỉ cần nói “thời gian gần như hằng số”.

Space complexity là `O(n)`, chủ yếu đến từ hai array `parent` và `size`.

## Trường hợp sử dụng điển hình

| Trường hợp sử dụng                       | Cách xử lý                                                                                    |
| ---------------------------------------- | --------------------------------------------------------------------------------------------- |
| Kiểm tra hai node có liên thông không    | So sánh `find(a)` và `find(b)`                                                                |
| Hợp nhất hai set                         | `union(a, b)`                                                                                 |
| Đếm số connected components              | Khởi tạo bằng `n`, mỗi lần hợp nhất thành công thì giảm 1                                     |
| Kiểm tra undirected graph có cycle không | Nếu hai đầu của một edge đã liên thông, thêm edge sẽ tạo cycle                                |
| Phương trình đẳng thức                   | Hợp nhất các quan hệ bằng nhau trước, sau đó kiểm tra các quan hệ khác nhau có xung đột không |

Union-find đặc biệt phù hợp với những vấn đề “các quan hệ liên tục được hợp nhất và cần truy vấn có cùng nhóm hay không”, chẳng hạn số lượng tỉnh, kết nối dư thừa, hợp nhất account, và thuật toán Kruskal trong minimum spanning tree.

Tuy nhiên, union-find không giỏi xử lý việc xóa quan hệ. Vì một khi hai set đã được hợp nhất, thông tin về những edge nào bên trong khiến chúng liên thông thường đã bị nén lại. Sau khi xóa một edge, không thể xác định set còn liên thông hay không chỉ bằng cách sửa đơn giản array `parent`.

## Điểm dễ sai

- Trong `find`, phải trả về root node, không phải parent node.
- Khi thực hiện path compression, không được bỏ mất giá trị trả về của recursion.
- Khi `union`, chỉ khi hai set vốn không liên thông thì số connected components mới giảm 1.
- Union-find phù hợp với việc hợp nhất, không giỏi xử lý xóa quan hệ.
- Bài toán grid hai chiều cần ánh xạ `(i, j)` thành mã một chiều, ví dụ `i * cols + j`.

## Bài tập đề xuất

- [547. Number of Provinces](https://leetcode.cn/problems/number-of-provinces/)
- [684. Redundant Connection](https://leetcode.cn/problems/redundant-connection/)
- [990. Satisfiability of Equality Equations](https://leetcode.cn/problems/satisfiability-of-equality-equations/)
- [1319. Number of Operations to Make Network Connected](https://leetcode.cn/problems/number-of-operations-to-make-network-connected/)
- [200. Number of Islands](https://leetcode.cn/problems/number-of-islands/)

## Tài liệu tham khảo

- [Algorithms, 4th Edition: Union-Find](https://algs4.cs.princeton.edu/15uf/)
- [Algorithms, 4th Edition: WeightedQuickUnionPathCompressionUF](https://algs4.cs.princeton.edu/15uf/WeightedQuickUnionPathCompressionUF.java.html)
- [CP-Algorithms: Disjoint Set Union](https://cp-algorithms.com/data_structures/disjoint_set_union.html)

<!-- @include: @article-footer.snippet.md -->
