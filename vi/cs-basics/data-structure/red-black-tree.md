---
title: "Giải thích chi tiết về red-black tree (properties, rotation, applications)"
description: "Giải thích chuyên sâu năm properties và quá trình rotation, adjustment của red-black tree, giúp hiểu cơ chế self-balancing và ứng dụng trong standard library cùng các cấu trúc index."
category: Computer Science Basics
tag:
  - Data Structure
head:
  - - meta
    - name: keywords
      content: red-black tree,self-balancing,rotation,insertion,deletion,properties,black height,time complexity
---

# Red-black tree

## Giới thiệu red-black tree

Red-black tree là một self-balancing binary search tree. Nó được Rudolf Bayer phát minh vào năm 1972 và khi đó được gọi là balanced binary B-tree (symmetric binary B-trees). Sau đó, vào năm 1978, Leo J. Guibas và Robert Sedgewick đã sửa đổi thành “red-black tree” như ngày nay.

Nhờ đặc tính self-balancing, nó bảo đảm các thao tác như search, insertion và deletion có time complexity `O(logn)` trong trường hợp xấu nhất, với performance ổn định.

Trong JDK, `TreeMap`, `TreeSet` và phần cài đặt bên trong của `HashMap` từ JDK1.8 đều sử dụng red-black tree.

## Vì sao cần red-black tree?

Red-black tree ra đời để giải quyết các nhược điểm của binary search tree.

Binary search tree là một data structure dựa trên phép so sánh. Mỗi node có một key; key của left child nhỏ hơn key của parent, còn key của right child lớn hơn key của parent. Cấu trúc này giúp thực hiện search, insertion và deletion thuận tiện, vì chỉ cần so sánh key giữa các node là có thể xác định vị trí của target node. Tuy nhiên, binary search tree có một vấn đề lớn: hình dạng của nó phụ thuộc vào thứ tự insertion của các node. Nếu các node được insertion theo thứ tự tăng dần hoặc giảm dần, binary search tree sẽ suy biến thành một linear structure, tức linked list. Khi đó, performance của binary search tree giảm mạnh, time complexity chuyển từ `O(logn)` thành `O(n)`.

Red-black tree ra đời để giải quyết nhược điểm của binary search tree, vì trong một số trường hợp binary search tree sẽ suy biến thành một linear structure.

## Đặc điểm của red-black tree

1. Mỗi node chỉ có thể là red hoặc black.
2. Root node luôn có màu black.
3. Mỗi empty child link đều được xem là một NIL leaf node có màu black.
4. Nếu node có màu red thì các child node của nó phải có màu black, tức không xuất hiện các red node liên tiếp.
5. Mỗi path từ một node bất kỳ đến mọi NIL descendant node của nó đều chứa cùng số lượng black node, tức có cùng black height.

Trong mối tương ứng giữa red-black tree và 2-3 tree, một black node cùng các red node nối với nó có thể biểu diễn chung một multi-key node. Đây chỉ là một structural mapping; bản thân node của red-black tree luôn có nhiều nhất hai child node.

Chính các đặc điểm này bảo đảm red-black tree được balance, khiến height của red-black tree không vượt quá `2log(n+1)`.

## Cấu trúc dữ liệu của red-black tree

AVL tree và red-black tree đều là self-balancing binary search tree, còn 2-3 tree là multi-way search tree. Red-black tree có thể tạo structural correspondence với 2-3 tree hoặc 2-3-4 tree, nhưng không thể gọi chung chúng là B-tree. So với AVL tree, điều kiện balance của red-black tree rộng hơn; nó giới hạn height của tree thông qua các color rule và black height constraint.

## Cài đặt cấu trúc red-black tree

```java
public class Node {

    public Class<?> clazz;
    public Integer value;
    public Node parent;
    public Node left;
    public Node right;

    // Thuộc tính cần cho AVL tree
    public int height;
    // Thuộc tính cần cho red-black tree
    public Color color = Color.RED;

}
```

### 1. Left-leaning coloring

![Minh họa quá trình left-leaning coloring của red-black tree](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/red-black-tree-1.png)

- Khi coloring, dựa vào grandparent node của node hiện tại để tìm uncle node.
- Sau đó color parent node thành black, uncle node thành black và grandparent node thành red. Tuy nhiên, việc color grandparent node thành red chỉ là tạm thời; sau thao tác cân bằng height của tree, root node sẽ được color thành black.

### 2. Right-leaning coloring

![Minh họa quá trình right-leaning coloring của red-black tree](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/red-black-tree-2.png)

### 3. Cân bằng bằng left rotation

#### 3.1 Một lần left rotation

![Minh họa cân bằng bằng một lần left rotation của red-black tree](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/red-black-tree-3.png)

#### 3.2 Right rotation + left rotation

![Minh họa cân bằng bằng right rotation + left rotation của red-black tree](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/red-black-tree-4.png)

### 4. Cân bằng bằng right rotation

#### 4.1 Một lần right rotation

![Minh họa cân bằng bằng một lần right rotation của red-black tree](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/red-black-tree-5.png)

#### 4.2 Left rotation + right rotation

![Minh họa cân bằng bằng left rotation + right rotation của red-black tree](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/red-black-tree-6.png)

## Trọng tâm ôn tập phỏng vấn

Trong phỏng vấn về red-black tree, thường không yêu cầu tự viết đầy đủ logic fix-up cho insertion và deletion. Thường gặp hơn là yêu cầu trình bày rõ properties, vì sao nó gần balanced, điểm khác biệt với AVL tree và những nơi được sử dụng trong Java.

| Điểm so sánh          | AVL tree                                    | Red-black tree                                    |
| --------------------- | ------------------------------------------- | ------------------------------------------------- |
| Yêu cầu balance       | Nghiêm ngặt hơn                             | Tương đối rộng hơn                                |
| Performance khi query | Ổn định hơn                                 | Cũng có thể duy trì `O(logn)`                     |
| Insertion và deletion | Có thể cần nhiều rotation và adjustment hơn | Thường cần ít lần adjustment hơn                  |
| Ứng dụng thường gặp   | Search structure có nhiều read, ít write    | `TreeMap`, `TreeSet`, treeification của `HashMap` |

Bạn có thể tổ chức câu trả lời phỏng vấn theo thứ tự sau:

1. Binary search tree thông thường sẽ suy biến thành linked list khi các node được insertion theo thứ tự.
2. Red-black tree giới hạn height thông qua color rule, bảo đảm query, insertion và deletion vẫn có time complexity `O(logn)`.
3. Nó không fully balanced mà chỉ approximately balanced, vì vậy cost của adjustment khi insertion và deletion thấp hơn AVL tree.
4. Trong Java, `TreeMap` và `TreeSet` dựa trên red-black tree; từ JDK 8, khi linked list trong `HashMap` quá dài, nó cũng sẽ treeify thành red-black tree.

Việc treeification của `HashMap` còn phải thỏa mãn điều kiện về capacity, không phải cứ linked list đạt threshold là chắc chắn treeify. Đây là chi tiết thường được hỏi sâu trong các buổi phỏng vấn về Java Collection.

<!-- @include: @article-footer.snippet.md -->
