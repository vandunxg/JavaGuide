---
title: "Giải thích chi tiết về tree structure (binary tree, AVL, B/B+ tree)"
description: "Giải thích có hệ thống các khái niệm cốt lõi và phương pháp traversal của tree và binary tree, kết hợp các chỉ số như height/depth để củng cố nền tảng data structure và tư duy thuật toán."
category: Computer Science Basics
tag:
  - Data Structure
head:
  - - meta
    - name: keywords
      content: tree,binary tree,binary search tree,balanced tree,traversal,preorder,inorder,postorder,level-order,height,depth
---

Tree là một data structure giống với tree trong đời sống (tree bị đảo ngược). Mọi tree không rỗng chỉ có một root node.

Một tree có các đặc điểm sau:

1. Hai node bất kỳ trong một tree được nối với nhau bằng duy nhất một path.
2. Nếu một tree có n node thì chắc chắn có đúng n-1 edge.
3. Một tree không chứa cycle.

Hình dưới đây là một tree, đồng thời là một binary tree.

![Binary tree](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/%E4%BA%8C%E5%8F%89%E6%A0%91-2.png)

Như hình trên, hãy giải thích một số khái niệm thường dùng trong tree:

- **Node**: Mỗi phần tử trong tree đều có thể gọi chung là node.
- **Root node**: Node ở tầng cao nhất, hay node không có parent node. Trong hình trên, node A là root node.
- **Parent node**: Nếu một node chứa child node thì node đó được gọi là parent node của child node. Trong hình trên, node B là parent node của node D và node E.
- **Child node**: Root node của một subtree mà một node chứa được gọi là child node của node đó. Trong hình trên, node D và node E là child node của node B.
- **Sibling node**: Các node có cùng parent node được gọi là sibling node của nhau. Trong hình trên, parent node chung của node D và node E là node B, nên D và E là sibling node.
- **Leaf node**: Node không có child node. Trong hình trên, D, F, H, I đều là leaf node.
- **Height của node**: Số edge trong path dài nhất từ node đó đến leaf node.
- **Depth của node**: Số edge trong path từ root node đến node đó.
- **Level của node**: depth của node + 1.
- **Height của tree**: height của root node.

> Về định nghĩa depth và height của tree, bạn có thể xem câu hỏi này trên Stack Overflow: [What is the difference between tree depth and height?](https://stackoverflow.com/questions/2603692/what-is-the-difference-between-tree-depth-and-height).

## Phân loại binary tree

**Binary tree** là một tree structure mà mỗi node có nhiều nhất hai branch (tức không có node nào có branch degree lớn hơn 2).

Các branch của **binary tree** thường được gọi là “**left subtree**” hoặc “**right subtree**”. Ngoài ra, các branch của **binary tree** có thứ tự trái phải và không thể tùy ý đảo ngược.

Tầng thứ i của **binary tree** có nhiều nhất `2^(i-1)` node. Theo định nghĩa “depth của root node bằng 0” trong bài viết này, binary tree có depth bằng k có nhiều nhất `2^(k+1)-1` node (trường hợp là full binary tree), và ít nhất `k+1` node (trường hợp suy biến thành một chain). Về định nghĩa depth của node, các tài liệu trong nước có những quy ước khác nhau; bài viết này sử dụng [định nghĩa depth của node](<https://zh.wikipedia.org/wiki/%E6%A0%91_(%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84)#/%E6%9C%AF%E8%AF%AD>) của Wikipedia.

![Định nghĩa depth của node trên Wikipedia](https://oss.javaguide.cn/github/javaguide/image-20220119112736158.png)

### Full binary tree

Nếu số node ở mỗi level của một binary tree đều đạt mức tối đa thì binary tree đó là **full binary tree**. Nói cách khác, nếu một binary tree có K level và tổng số node là `2^k -1` thì nó là **full binary tree**. Hình dưới đây minh họa điều đó:

![Full binary tree](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/full-binary-tree.png)

### Complete binary tree

Nếu tất cả level ngoại trừ level cuối đều đầy, đồng thời level cuối đầy hoặc chỉ thiếu liên tiếp một số node ở bên phải, thì binary tree đó là **complete binary tree**.

Bạn có thể hình dung một tree được mở rộng bắt đầu từ root node: chỉ sau khi mở rộng xong left child node mới bắt đầu mở rộng right child node; chỉ sau khi mở rộng xong một level mới tiếp tục mở rộng level tiếp theo. Hình dưới đây minh họa điều đó:

![Complete binary tree](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/complete-binary-tree.png)

Complete binary tree có một tính chất rất hữu ích: **số thứ tự của parent node và child node có quan hệ tương ứng với nhau.**

Có thể bạn đã nhận ra rằng khi giá trị của root node là 1, nếu số thứ tự của parent node là i thì số thứ tự của left child node là 2i, còn số thứ tự của right child node là 2i+1. Tính chất này giúp complete binary tree tiết kiệm đáng kể không gian khi lưu bằng array, đồng thời dùng số thứ tự để tìm parent node và child node của một node. Phần lưu trữ binary tree sẽ giới thiệu chi tiết hơn ở phía sau.

### AVL tree (height-balanced binary search tree)

**AVL tree** là một binary search tree cân bằng theo height và có các tính chất sau:

1. Có thể là một empty tree.
2. Nếu không rỗng, trị tuyệt đối của chênh lệch height giữa hai subtree trái và phải không vượt quá 1, đồng thời hai subtree trái và phải cũng đều là AVL tree.

Red-black tree, scapegoat tree, weight-balanced tree và các tree khác cũng được dùng để tránh binary search tree bị suy biến nghiêm trọng, nhưng điều kiện balance của chúng không giống điều kiện “chênh lệch height giữa left subtree và right subtree không vượt quá 1” của AVL tree. Splay tree đạt được bảo đảm về amortized complexity thông qua việc điều chỉnh sau khi truy cập.

Trước khi giới thiệu balanced binary tree, hãy xem một tree:

![Oblique tree](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/oblique-tree.png)

**Bạn gọi thứ này là tree à???**

Đúng vậy, thứ này thực sự được gọi là tree, chỉ là tree này đã suy biến thành một linked list, và chúng ta gọi nó là **oblique tree**.

**Nếu vậy thì tại sao tôi không dùng linked list luôn?**

Đúng là như vậy.

Bản thân binary tree thông thường không bảo đảm việc query nhanh hơn linked list. Chỉ khi tận dụng tính có thứ tự của binary search tree hoặc các quan hệ index khác, tree structure mới có thể giúp **search** và **update** data hiệu quả hơn; binary search tree chưa được balance trong trường hợp xấu nhất vẫn suy biến thành `O(n)`.

Tuy nhiên, nếu binary search tree suy biến thành linked list thì lợi thế query do structure có thứ tự mang lại khó thể hiện, hiệu năng cũng giảm mạnh. AVL tree sử dụng điều kiện height balance chặt chẽ hơn để tránh tình trạng này: chênh lệch height giữa left subtree và right subtree của mỗi node nhiều nhất là 1, như hình dưới đây:

![Balanced binary tree](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/balanced-binary-tree.png)

## Lưu trữ binary tree

Lưu trữ binary tree chủ yếu được chia thành **linked storage** và **sequential storage**:

### Linked storage

Tương tự linked list, linked storage của binary tree dùng pointer để nối các node lại với nhau và không cần vùng storage liên tục.

Mỗi node gồm ba thuộc tính:

- data. data không nhất thiết là một data đơn lẻ; tùy trường hợp, nó có thể là nhiều data thuộc các type khác nhau.
- Pointer của left node left.
- Pointer của right node right.

Nhưng JAVA không có pointer mà!

Vậy thì dùng reference đến object là được (đừng hỏi tôi tìm object ở đâu).

![Binary tree với linked storage](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/chain-store-binary-tree.png)

### Sequential storage

Sequential storage là lưu trữ bằng array. Mỗi vị trí trong array chỉ lưu data của node, không lưu pointer của left child node và right child node; index của child node được xác định thông qua array index. Số thứ tự của root node là 1. Với mỗi node Node, giả sử node được lưu tại vị trí có array index là i thì left child node được lưu tại vị trí 2i, còn right child node được lưu tại vị trí có array index là 2i+1.

Sequential storage bằng array của một complete binary tree được minh họa như hình dưới đây:

![Sequential storage bằng array của complete binary tree](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/sequential-storage.png)

Bạn có thể thử điền array dùng để lưu binary tree dưới đây, rồi so sánh sự khác nhau với sequential storage của complete binary tree:

![Sequential storage bằng array của non-complete binary tree](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/sequential-storage2.png)

Có thể thấy rằng nếu binary tree cần lưu không phải complete binary tree thì trong array sẽ xuất hiện các khoảng trống, khiến hiệu suất sử dụng memory giảm.

## Traversal binary tree

### Preorder traversal

![Preorder traversal](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/preorder-traversal.png)

Preorder traversal của binary tree là output root node trước, sau đó traverse left subtree, cuối cùng traverse right subtree. Khi traverse left subtree và right subtree cũng tuân theo quy tắc preorder traversal, vì vậy có thể implement preorder traversal bằng recursion.

Code như sau:

```java
public void preOrder(TreeNode root){
    if(root == null){
        return;
    }
    System.out.println(root.data);
    preOrder(root.left);
    preOrder(root.right);
}
```

### Inorder traversal

![Inorder traversal](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/inorder-traversal.png)

Inorder traversal của binary tree là đệ quy thực hiện inorder traversal của left subtree trước, sau đó output value của root node, rồi đệ quy thực hiện inorder traversal của right subtree. Bạn có thể hình dung việc dùng một bàn tay ép phẳng tree: parent node bị ép vào giữa left child node và right child node, như hình dưới đây:

![Inorder traversal của binary tree sau khi ép phẳng](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/inorder-traversal2.png)

Code như sau:

```java
public void inOrder(TreeNode root){
    if(root == null){
        return;
    }
    inOrder(root.left);
    System.out.println(root.data);
    inOrder(root.right);
}
```

### Postorder traversal

![Postorder traversal](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/postorder-traversal.png)

Postorder traversal của binary tree là đệ quy thực hiện postorder traversal của left subtree trước, sau đó đệ quy thực hiện postorder traversal của right subtree, cuối cùng output value của root node.

Code như sau:

```java
public void postOrder(TreeNode root){
    if(root == null){
        return;
    }
    postOrder(root.left);
    postOrder(root.right);
    System.out.println(root.data);
}
```

## Trọng tâm ôn tập phỏng vấn

Trong phỏng vấn về tree structure, câu hỏi thường bắt đầu từ binary tree traversal, sau đó lần lượt hỏi sâu về binary search tree, balanced tree, B tree và B+ tree.

| Structure          | Đặc điểm                                                                       | Câu hỏi thường gặp                                       |
| ------------------ | ------------------------------------------------------------------------------ | -------------------------------------------------------- |
| Binary tree        | Mỗi node có nhiều nhất hai child node                                          | Traversal, path, lowest common ancestor, xây dựng tree   |
| Binary search tree | Left subtree nhỏ hơn root, right subtree lớn hơn root                          | Inorder traversal có thứ tự, suy biến thành linked list  |
| AVL tree           | Height-balanced                                                                | Query nhanh, insert/delete cần rotation thường xuyên hơn |
| Red-black tree     | Gần balanced                                                                   | Java `TreeMap`, tree hóa `HashMap`                       |
| B tree             | Multi-way balanced search tree                                                 | Thân thiện với disk IO                                   |
| B+ tree            | Data thường nằm ở leaf node, các leaf node được nối bằng linked list có thứ tự | MySQL index, range query                                 |

Template traversal binary tree cần có thể tự viết:

```java
void dfs(TreeNode root) {
    if (root == null) {
        return;
    }
    // Vị trí preorder
    dfs(root.left);
    // Vị trí inorder
    dfs(root.right);
    // Vị trí postorder
}
```

Các câu trả lời thường gặp về BST:

- Inorder traversal của binary search tree cho ra một sequence tăng dần.
- Nếu data được insert vốn đã có thứ tự, BST thông thường sẽ suy biến thành linked list.
- AVL tree balance chặt chẽ hơn red-black tree, query ổn định hơn; yêu cầu balance của red-black tree rộng hơn, chi phí điều chỉnh khi insert/delete thấp hơn.
- B+ tree phù hợp với database index: mỗi node có thể lưu nhiều key hơn, height của tree thấp hơn, linked list có thứ tự ở leaf node phù hợp với range query.

Có thể phân loại bài toán thuật toán binary tree trước dựa trên việc “current node đảm nhiệm vai trò gì trong recursion”:

- Nhóm path: current node cần được thêm vào path, sau khi recursion kết thúc thì undo; thường gặp trong path từ root đến leaf và tổng path.
- Nhóm thông tin subtree: left subtree và right subtree trả kết quả trước, current node sau đó merge chúng; thường gặp khi tính height, diameter và balanced binary tree.
- Nhóm phân nhánh hội tụ: left subtree và right subtree lần lượt tìm target, current node xác định có phải điểm hội tụ hay không; thường gặp khi tìm lowest common ancestor.
- Nhóm construction: xác định root node trước, sau đó chia interval của left subtree và right subtree; thường gặp khi xây dựng binary tree từ preorder + inorder.

## Java code template

Level-order traversal là template non-recursive thường gặp nhất trong phỏng vấn binary tree. Nhiều bài như “giá trị lớn nhất của mỗi level”, “zigzag traversal”, “minimum depth” đều có thể biến đổi từ template này.

```java
List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> ans = new ArrayList<>();
    if (root == null) {
        return ans;
    }
    Queue<TreeNode> queue = new ArrayDeque<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int size = queue.size();
        List<Integer> level = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left != null) {
                queue.offer(node.left);
            }
            if (node.right != null) {
                queue.offer(node.right);
            }
        }
        ans.add(level);
    }
    return ans;
}
```

Khi verify BST, không nên chỉ so sánh current node với left child và right child. Cách đúng là truyền upper bound và lower bound cho mỗi subtree:

```java
boolean isValidBST(TreeNode root) {
    return check(root, Long.MIN_VALUE, Long.MAX_VALUE);
}

boolean check(TreeNode node, long lower, long upper) {
    if (node == null) {
        return true;
    }
    if (node.val <= lower || node.val >= upper) {
        return false;
    }
    return check(node.left, lower, node.val) && check(node.right, node.val, upper);
}
```

Lowest common ancestor (LCA) có thể dùng tư duy postorder: left subtree và right subtree tìm target node trước, current node sau đó dựa vào return value để xác định có hội tụ hay không.

```java
TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q) {
        return root;
    }
    TreeNode left = lowestCommonAncestor(root.left, p, q);
    TreeNode right = lowestCommonAncestor(root.right, p, q);
    if (left != null && right != null) {
        return root;
    }
    return left != null ? left : right;
}
```

Ý nghĩa của đoạn code này là: nếu `p` và `q` lần lượt xuất hiện trong left subtree và right subtree thì current node là lowest common ancestor; nếu chỉ xuất hiện ở một bên thì tiếp tục return kết quả của bên đó lên trên.

Khi xây dựng binary tree từ preorder + inorder, phần tử đầu tiên của preorder array là root node; trong inorder array, phần bên trái root node là left subtree, phần bên phải là right subtree. Để tránh phải tìm root node trong array tuyến tính mỗi lần, thông thường trước tiên dùng hash table để ghi lại index trong inorder.

```java
TreeNode buildTree(int[] preorder, int[] inorder) {
    Map<Integer, Integer> index = new HashMap<>();
    for (int i = 0; i < inorder.length; i++) {
        index.put(inorder[i], i);
    }
    return build(preorder, 0, preorder.length - 1, 0, inorder.length - 1, index);
}

TreeNode build(
    int[] preorder,
    int preLeft,
    int preRight,
    int inLeft,
    int inRight,
    Map<Integer, Integer> index
) {
    if (preLeft > preRight) {
        return null;
    }
    int rootVal = preorder[preLeft];
    int rootIndex = index.get(rootVal);
    int leftSize = rootIndex - inLeft;
    TreeNode root = new TreeNode(rootVal);
    root.left = build(preorder, preLeft + 1, preLeft + leftSize, inLeft, rootIndex - 1, index);
    root.right = build(preorder, preLeft + leftSize + 1, preRight, rootIndex + 1, inRight, index);
    return root;
}
```

Điểm dễ sai nhất trong bài toán construction là boundary của interval. Bạn nên viết rõ ý nghĩa của `preLeft/preRight` và `inLeft/inRight` trước, sau đó dựa vào kích thước left subtree `leftSize` để chia preorder array.

## Minh họa quy trình và các boundary sample

Với bài toán binary tree, trước tiên có thể xác định “current node cần làm gì”, rồi quyết định dùng preorder, inorder, postorder hay level-order.

```text
Preorder: xử lý current node trước, sau đó xử lý left subtree và right subtree, phù hợp để copy tree và xây dựng path.
Inorder: left -> root -> right, kết quả inorder trong BST có thứ tự.
Postorder: xử lý left subtree và right subtree trước, sau đó xử lý current node, phù hợp để tính height, diameter và xóa node.
Level-order: tiến hành theo từng level, phù hợp với minimum depth, thống kê theo level và serialization.
```

Bạn nên tự viết và kiểm tra một số boundary sample trước:

- Empty tree: nhiều bài nên return empty list, `0` hoặc `true`.
- Chỉ có một node: cả recursion exit và level-order queue đều phải xử lý được.
- Linked list suy biến: depth của recursion có thể đạt `n`, không được viết nhầm complexity thành `O(logn)`.
- BST có `Integer.MIN_VALUE` / `Integer.MAX_VALUE`: nên dùng `long` cho upper bound và lower bound.
- Trong LCA, một target node là ancestor của target node còn lại: khi gặp target node phải return current node ngay.
- Khi xây dựng tree mà array rỗng: recursive interval sẽ trở thành `preLeft > preRight`, cần return `null`.

## Bài tập đề xuất

- [144. Binary Tree Preorder Traversal](https://leetcode.cn/problems/binary-tree-preorder-traversal/)
- [102. Binary Tree Level Order Traversal](https://leetcode.cn/problems/binary-tree-level-order-traversal/)
- [98. Validate Binary Search Tree](https://leetcode.cn/problems/validate-binary-search-tree/)
- [236. Lowest Common Ancestor of a Binary Tree](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/)
- [105. Construct Binary Tree from Preorder and Inorder Traversal](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

<!-- @include: @article-footer.snippet.md -->
