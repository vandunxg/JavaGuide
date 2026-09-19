---
title: "Tổng hợp câu hỏi phỏng vấn về DFS và BFS: template tìm kiếm trên cây, đồ thị, ma trận và đường đi ngắn nhất"
description: Tổng hợp câu hỏi phỏng vấn về DFS và BFS, giải thích tìm kiếm theo chiều sâu, tìm kiếm theo chiều rộng, duyệt cây, duyệt đồ thị, tìm kiếm ma trận, duyệt theo tầng, đường đi ngắn nhất và template Java.
category: Computer Science Basics
tag:
  - Algorithms
head:
  - - meta
    - name: keywords
      content: DFS,BFS,depth-first search,breadth-first search,tree traversal,graph traversal,matrix search,level-order traversal,shortest path,Java DFS,Java BFS,LeetCode
---

DFS và BFS là nền tảng của các bài toán về cây, đồ thị và ma trận. Trong phỏng vấn, người phỏng vấn sẽ không chỉ hỏi “DFS là gì”, mà thường đưa cho bạn một bài toán đảo, bài toán phụ thuộc khóa học, số bước ngắn nhất hoặc duyệt cây nhị phân theo tầng, rồi yêu cầu bạn chọn cách tìm kiếm và xử lý biên.

Một cách phán đoán đơn giản: khi cần đi đến tận cùng, liệt kê các đường đi hoặc xử lý các thành phần liên thông, hãy ưu tiên nghĩ đến DFS; khi cần tiến hành theo tầng hoặc tìm số bước ngắn nhất, hãy ưu tiên nghĩ đến BFS.

## Trọng tâm phỏng vấn

- Có thể viết DFS đệ quy và BFS dùng queue.
- Có thể nêu độ phức tạp của tìm kiếm cây và đồ thị.
- Có thể xử lý `visited` để tránh truy cập lặp và vòng lặp vô hạn.
- Có thể phân biệt “duyệt tất cả node” và “tìm số bước ngắn nhất”.
- Có thể chuyển bài toán ma trận thành tìm kiếm đồ thị.

## Chọn DFS hay BFS như thế nào?

DFS và BFS đều có thể duyệt node, nhưng mỗi cách có ưu thế tự nhiên khác nhau.

| Mục tiêu                                          | Cách thường dùng      | Lý do                                              |
| ------------------------------------------------- | --------------------- | -------------------------------------------------- |
| Duyệt tất cả node                                 | DFS hoặc BFS đều được | Chỉ cần không truy cập lặp                         |
| Tìm diện tích thành phần liên thông               | DFS thuận tiện hơn    | Mở rộng bằng đệ quy, code ngắn                     |
| Tìm số bước ngắn nhất trong đồ thị không trọng số | BFS                   | Tiến hành theo tầng, lần đầu đến đích là ngắn nhất |
| Liệt kê tất cả đường đi                           | DFS                   | Đường đi tự nhiên nằm trong stack đệ quy           |
| Duyệt cây nhị phân theo tầng                      | BFS                   | Queue vừa khớp để xử lý theo tầng                  |

Nếu trong đề xuất hiện “ít nhất bao nhiêu bước”, “đường đi ngắn nhất”, “lan rộng đến mọi vị trí”, trước tiên hãy nghĩ đến BFS. Nếu xuất hiện “tất cả phương án”, “có tồn tại một đường đi hay không”, “kích thước thành phần liên thông”, trước tiên hãy nghĩ đến DFS.

## Template DFS

Cách viết thường gặp của DFS trên ma trận:

```java
void dfs(char[][] grid, int i, int j) {
    if (i < 0 || i >= grid.length || j < 0 || j >= grid[0].length) {
        return;
    }
    if (grid[i][j] != '1') {
        return;
    }
    grid[i][j] = '2';
    dfs(grid, i + 1, j);
    dfs(grid, i - 1, j);
    dfs(grid, i, j + 1);
    dfs(grid, i, j - 1);
}
```

Ở đây, ta đổi trực tiếp phần đất đã truy cập thành `'2'`, tương đương với việc dùng mảng ban đầu làm cờ đánh dấu truy cập. Nếu đề bài không cho phép sửa input, hãy tạo riêng `boolean[][] visited`.

Cần xác định rõ ý nghĩa của hàm đệ quy DFS trước. Có thể giải thích đoạn code trên như sau: bắt đầu từ `(i, j)`, đánh dấu toàn bộ phần đất liên thông với nó.

Ý nghĩa này quyết định thứ tự code:

1. Nếu vượt biên thì trả về ngay.
2. Nếu ô hiện tại không phải đất thì trả về ngay.
3. Đánh dấu ô hiện tại để tránh truy cập lặp.
4. Tiếp tục truy cập 4 hướng trên, dưới, trái, phải.

Nhiều bug DFS xuất phát từ việc viết bước 3 quá muộn. Nếu đệ quy sang các node lân cận trước rồi mới đánh dấu node hiện tại, có thể xảy ra đệ quy qua lại giữa hai ô kề nhau.

## Template BFS

BFS phù hợp với duyệt theo tầng và số bước ngắn nhất. Template dưới đây quy ước input là ma trận hình chữ nhật khác rỗng, trong đó `0` biểu thị có thể đi qua, `1` biểu thị vật cản; hàm trả về số bước ngắn nhất từ điểm bắt đầu đến điểm đích, trả về `-1` nếu tọa độ vượt biên hoặc không thể đến đích:

```java
int bfs(int[][] grid, int startX, int startY, int targetX, int targetY) {
    if (grid == null || grid.length == 0 || grid[0].length == 0) {
        return -1;
    }
    int rows = grid.length;
    int columns = grid[0].length;
    if (startX < 0 || startX >= rows || startY < 0 || startY >= columns
            || targetX < 0 || targetX >= rows || targetY < 0 || targetY >= columns
            || grid[startX][startY] == 1 || grid[targetX][targetY] == 1) {
        return -1;
    }
    int[][] dirs = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
    Queue<int[]> queue = new ArrayDeque<>();
    queue.offer(new int[] {startX, startY});
    boolean[][] visited = new boolean[rows][columns];
    visited[startX][startY] = true;
    int step = 0;
    while (!queue.isEmpty()) {
        int size = queue.size();
        for (int k = 0; k < size; k++) {
            int[] cur = queue.poll();
            if (cur[0] == targetX && cur[1] == targetY) {
                return step;
            }
            for (int[] dir : dirs) {
                int x = cur[0] + dir[0];
                int y = cur[1] + dir[1];
                if (x < 0 || x >= rows || y < 0 || y >= columns
                        || visited[x][y] || grid[x][y] == 1) {
                    continue;
                }
                visited[x][y] = true;
                queue.offer(new int[] {x, y});
            }
        }
        step++;
    }
    return -1;
}
```

Đoạn code này trả về số tầng hiện tại khi node đích lần đầu được lấy ra khỏi queue, thay vì chờ queue rỗng. Điều kiện để đi qua trong từng bài cụ thể có thể không phải là `0` và `1`, cần điều chỉnh theo đề bài.

Điểm mấu chốt của BFS là “xử lý theo tầng”. Ban đầu queue chứa các node tầng 0, mỗi vòng lấy kích thước queue hiện tại làm `size` và chỉ xử lý các node của tầng này; các node mới được mở rộng từ chúng thuộc tầng tiếp theo.

Vì sao BFS trên đồ thị không trọng số có thể tìm đường đi ngắn nhất? Vì chi phí của mỗi cạnh bằng nhau. Khi BFS lần đầu đến một node, chắc chắn nó đã dùng số cạnh ít nhất. Về sau, dù vẫn có thể đến node đó lần nữa, đường đi cũng không ngắn hơn, nên có thể đánh dấu đã truy cập ngay.

BFS đa nguồn cũng rất phổ biến. Ví dụ trong bài “Cam bị thối”, tất cả quả cam thối bắt đầu lan rộng cùng lúc. Cách làm là đưa tất cả quả cam thối ban đầu vào queue trước, sau đó lan rộng theo từng tầng.

## Khác biệt giữa tìm kiếm cây và tìm kiếm đồ thị

Cây không có vòng, nên nhiều trường hợp không cần `visited`. Đồ thị có thể có vòng, vì vậy phải tính đến việc truy cập lặp.

| Bối cảnh                  | Có thường dùng `visited` không? | Giải thích                                                          |
| ------------------------- | ------------------------------- | ------------------------------------------------------------------- |
| Duyệt đệ quy cây nhị phân | Thường không                    | Node không có cạnh quay về node cha                                 |
| Duyệt đồ thị vô hướng     | Cần                             | Nếu không, hai node sẽ truy cập lẫn nhau                            |
| Duyệt đồ thị có hướng     | Thường cần                      | Có thể tồn tại vòng                                                 |
| Tìm kiếm ma trận          | Cần                             | Có thể đi từ các hướng trên, dưới, trái, phải quay lại điểm ban đầu |

## Độ phức tạp

Trong tìm kiếm đồ thị, thường dùng `V` biểu thị số đỉnh và `E` biểu thị số cạnh. Khi lưu bằng adjacency list, độ phức tạp thời gian của DFS và BFS thường là `O(V + E)`, độ phức tạp không gian là `O(V)`.

Nếu kích thước ma trận là `m * n`, mỗi ô được truy cập nhiều nhất một lần, độ phức tạp thời gian của tìm kiếm ma trận là `O(mn)`, không gian dành cho cờ đánh dấu truy cập hoặc queue trong trường hợp xấu nhất cũng là `O(mn)`.

## Chuyển bài toán ma trận thành đồ thị như thế nào?

Mỗi ô trong ma trận có thể được xem là một node trong đồ thị. 4 hướng trên, dưới, trái, phải chính là các cạnh đi ra từ node đó.

Mảng hướng thường dùng:

```java
int[][] dirs = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
```

Khi duyệt các node lân cận, chỉ cần làm 3 việc:

1. Tính tọa độ mới.
2. Kiểm tra có vượt biên hay không.
3. Kiểm tra đã truy cập hay chưa, hoặc có phù hợp với yêu cầu đề bài hay không.

Nếu đề bài cho phép di chuyển theo đường chéo, chỉ cần mở rộng mảng hướng thành 8 hướng. Không nên viết thủ công 4 đoạn lời gọi đệ quy gần như giống nhau trong code, vì dùng mảng hướng sẽ ít bỏ sót điều kiện hơn.

## Minh họa quá trình và các trường hợp biên

Lấy số lượng đảo làm ví dụ: khi gặp một ô đất chưa được truy cập, bắt đầu DFS/BFS từ ô đó để đánh dấu toàn bộ hòn đảo.

| Bước | Thao tác                      | Mục đích                              |
| ---- | ----------------------------- | ------------------------------------- |
| 1    | Quét ma trận, tìm một `1`     | Phát hiện một hòn đảo mới             |
| 2    | Tăng số lượng đảo lên 1       | Ghi nhận thành phần liên thông        |
| 3    | DFS/BFS từ ô hiện tại         | Đánh dấu toàn bộ phần đất của đảo này |
| 4    | Tiếp tục quét các ô tiếp theo | Tránh đếm lặp cùng một hòn đảo        |

Khi kiểm tra tìm kiếm ma trận, nên chú ý các trường hợp biên sau:

| Input                       | Trọng tâm                                                                        |
| --------------------------- | -------------------------------------------------------------------------------- |
| Ma trận rỗng                | Có kiểm tra độ dài hàng và cột trước hay không                                   |
| Toàn là nước                | Kết quả phải là 0                                                                |
| Toàn là đất                 | Chỉ có thể đếm thành 1 thành phần liên thông                                     |
| Chỉ kề nhau theo đường chéo | Nếu đề bài chỉ cho phép trên, dưới, trái, phải thì không được tính là liên thông |

Cách viết sai thường gặp:

```java
void dfs(char[][] grid, int i, int j) {
    dfs(grid, i + 1, j);
    grid[i][j] = '2'; // Sai: đánh dấu quá muộn, có thể đệ quy qua lại
}
```

Phải hoàn thành việc đánh dấu truy cập trước khi đệ quy mở rộng sang các node lân cận. Trong đồ thị và ma trận, chỉ cần tồn tại cạnh quay lại hoặc truy cập lẫn nhau giữa các ô kề nhau, việc đánh dấu quá muộn có thể gây truy cập lặp, thậm chí tràn stack.

## Điểm dễ sai

- Đánh dấu truy cập ngay khi đưa node vào queue trong BFS để tránh cùng một node bị đưa vào queue lặp lại.
- Độ sâu đệ quy DFS quá lớn có thể gây tràn stack; trong phỏng vấn, có thể nói rằng ta có thể đổi sang stack tường minh.
- Với bài toán ma trận, kiểm tra vượt biên trước rồi mới truy cập mảng.
- Với đồ thị vô hướng, cần chú ý vấn đề đi từ node con quay lại node cha.
- Khi tìm số bước ngắn nhất, việc đếm số tầng của BFS phải gắn với kích thước tầng hiện tại của queue.

## Bài tập đề xuất

- [102. Duyệt cây nhị phân theo tầng](https://leetcode.cn/problems/binary-tree-level-order-traversal/)
- [200. Số lượng đảo](https://leetcode.cn/problems/number-of-islands/)
- [695. Diện tích lớn nhất của đảo](https://leetcode.cn/problems/max-area-of-island/)
- [994. Cam bị thối](https://leetcode.cn/problems/rotting-oranges/)
- [207. Lịch học](https://leetcode.cn/problems/course-schedule/)

<!-- @include: @article-footer.snippet.md -->
