---
title: Tổng hợp các khái niệm cốt lõi của Git
description: Tổng hợp các khái niệm cốt lõi và workflow của Git, bao quát branch và merge, quản lý commit và giải quyết conflict, hỗ trợ cộng tác nhóm và nâng cao chất lượng code.
category: Công cụ phát triển
tag:
  - Git
head:
  - - meta
    - name: keywords
      content: Git,version control,distributed,branch,commit,merge,conflict resolution,workflow
---

## Version control

### Version control là gì

Version control là hệ thống ghi lại những thay đổi về nội dung của một hoặc nhiều file, để sau này tra cứu các revision cụ thể. Ngoài source code của project, bạn có thể dùng version control cho mọi loại file.

### Vì sao cần version control

Nhờ có nó, bạn có thể đưa một file về trạng thái trước đó, thậm chí đưa cả project về trạng thái tại một thời điểm trong quá khứ. Bạn có thể so sánh chi tiết thay đổi của file, tìm ra ai là người sửa phần nào lần cuối, từ đó tìm nguyên nhân gây ra vấn đề bất thường, cũng như biết ai đã báo cáo một bug chức năng và báo cáo vào thời điểm nào.

### Local version control system

Nhiều người có thói quen lưu các version khác nhau bằng cách copy toàn bộ thư mục project, hoặc đổi tên và thêm thời gian backup để phân biệt. Ưu điểm duy nhất của cách này là đơn giản, nhưng nó rất dễ gây lỗi. Đôi khi bạn có thể nhầm lẫn working directory hiện tại, vô tình ghi nhầm file hoặc ghi đè lên file ngoài ý muốn.

Để giải quyết vấn đề này, từ rất lâu trước đây người ta đã phát triển nhiều local version control system. Phần lớn sử dụng một database đơn giản để ghi lại các khác biệt trong những lần cập nhật file.

![Local version control system](https://oss.javaguide.cn/github/javaguide/tools/git/%E6%9C%AC%E5%9C%B0%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6%E7%B3%BB%E7%BB%9F.png)

### Centralized version control system

Sau đó, người ta lại gặp một vấn đề: làm thế nào để các developer trên những system khác nhau có thể cộng tác? Vì vậy, Centralized Version Control Systems (gọi tắt là CVCS) ra đời.

Centralized version control system có một server quản lý tập trung duy nhất, lưu các revision của toàn bộ file. Những người cộng tác kết nối tới server này thông qua client để lấy file mới nhất hoặc submit update.

![Centralized version control system](https://oss.javaguide.cn/github/javaguide/tools/git/%E9%9B%86%E4%B8%AD%E5%8C%96%E7%9A%84%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6%E7%B3%BB%E7%BB%9F.png)

Cách này tuy giải quyết được hạn chế khiến local version control system không thể cho các developer trên những system khác nhau cộng tác, nhưng vẫn tồn tại các vấn đề sau:

- **Single point of failure:** Nếu central server bị down, những người khác không thể sử dụng; nếu disk của central database bị hỏng mà không có backup, bạn sẽ mất toàn bộ data. Local version control system cũng có vấn đề tương tự: chỉ cần toàn bộ lịch sử của project được lưu ở một nơi duy nhất là có nguy cơ mất tất cả history update.
- **Phải có network mới làm việc được:** Bị ảnh hưởng bởi tình trạng network và bandwidth.

### Distributed version control system

Vì vậy, Distributed Version Control System (gọi tắt là DVCS) ra đời. Git là một distributed version control system điển hình.

Các system loại này không chỉ lấy snapshot mới nhất của file từ client, mà còn mirror toàn bộ code repository về local. Nhờ vậy, nếu bất kỳ server nào dùng cho việc cộng tác bị lỗi, bạn vẫn có thể dùng bất kỳ local repository nào đã được mirror để khôi phục sau đó. Vì mỗi lần clone thực chất là một lần backup đầy đủ code repository.

![Distributed version control system](https://oss.javaguide.cn/github/javaguide/tools/git/%E5%88%86%E5%B8%83%E5%BC%8F%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6%E7%B3%BB%E7%BB%9F.png)

Distributed version control system có thể hoạt động mà không cần network, vì máy tính của mỗi người đều có version repository đầy đủ. Sau khi sửa một file, bạn chỉ cần push thay đổi của mình cho người khác. Tuy nhiên, trong thực tế sử dụng distributed version control system, người ta hiếm khi push thay đổi trực tiếp, mà thường dùng một server đóng vai trò “central server”. Server này chỉ dùng để thuận tiện cho việc “trao đổi” thay đổi của mọi người. Không có nó, mọi người vẫn làm việc bình thường, chỉ là trao đổi thay đổi sẽ bất tiện hơn.

Ưu điểm của distributed version control system không chỉ đơn giản là không cần network; ở phần sau, chúng ta còn thấy các tính năng như branch management cực kỳ mạnh mẽ của Git.

## Làm quen với Git

### Lược sử Git

Khi đó, team Linux kernel sử dụng distributed version control system BitKeeper để quản lý và bảo trì code. Tuy nhiên, sau đó quan hệ hợp tác giữa công ty thương mại phát triển BitKeeper và Linux kernel open source community kết thúc, công ty thu hồi quyền sử dụng BitKeeper miễn phí của Linux kernel community. Linux open source community, đặc biệt là Linus Torvalds, dựa trên kinh nghiệm và bài học khi sử dụng BitKeeper để phát triển version system của riêng mình, đồng thời thực hiện nhiều cải tiến cho version control system mới.

### Khác biệt chính giữa Git và các version management system khác

Git có sự khác biệt rất lớn với các version control system khác trong cách lưu trữ và xử lý nhiều loại thông tin. Mặc dù hình thức command khi thao tác khá tương tự, việc hiểu những khác biệt này sẽ giúp tránh nhầm lẫn trong quá trình sử dụng.

Sau đây, chúng ta chủ yếu nói về một khác biệt quan trọng giữa Git và các version management system khác: **cách xử lý data**.

**Git trực tiếp ghi lại snapshot thay vì so sánh difference. Phần sau tôi sẽ giải thích chi tiết sự khác nhau giữa hai cách này.**

Phần lớn version control system (CVS, Subversion, Perforce, Bazaar, v.v.) lưu trữ thông tin dưới dạng danh sách thay đổi của file. Các system này **coi thông tin được lưu là một nhóm file cơ bản và các difference tích lũy dần theo thời gian của từng file.**

Nguyên lý cụ thể như hình dưới đây. Thực ra cách hiểu khá đơn giản: mỗi khi chúng ta commit update một file, system sẽ ghi lại file đó đã được update những gì, biểu diễn bằng ký hiệu gia số Δ (Delta).

![](https://oss.javaguide.cn/github/javaguide/tools/git/2019-3deltas.png)

**Làm thế nào để lấy được version cuối cùng của một file?**

Rất đơn giản, dựa trên kiến thức toán học cơ bản ở cấp trung học, chúng ta chỉ cần cộng các file gốc với những phần tăng thêm này.

**Cách này có vấn đề gì?**

Ví dụ, nếu có rất nhiều phần tăng thêm, việc lấy file cuối cùng có tốn thời gian và performance không?

Git không xử lý hoặc lưu data theo cách trên. Ngược lại, Git giống như coi data là một nhóm snapshot của một file system nhỏ. Mỗi lần bạn commit update hoặc lưu trạng thái project trong Git, Git chủ yếu tạo snapshot của toàn bộ file tại thời điểm đó và lưu index của snapshot này. Để đạt hiệu quả, nếu file không thay đổi, Git không lưu lại file đó lần nữa mà chỉ giữ một link trỏ tới file đã lưu trước đó. Git xử lý data giống như một **snapshot stream**.

![](https://oss.javaguide.cn/github/javaguide/tools/git/2019-3snapshots.png)

### Ba trạng thái của Git

Git có ba trạng thái, file của bạn có thể ở một trong số đó:

1. **Đã commit (committed):** Data đã được lưu an toàn trong local database.
2. **Đã sửa đổi (modified):** File đã được sửa nhưng chưa được lưu vào database.
3. **Đã staged (staged):** Đánh dấu version hiện tại của một file đã sửa để file đó được đưa vào snapshot của commit tiếp theo.

Từ đó dẫn đến khái niệm về ba working area của project Git: **Git repository (.git directory)**, **Working Directory** và **Staging Area**.

![](https://oss.javaguide.cn/github/javaguide/tools/git/2019-3areas.png)

**Git workflow cơ bản như sau:**

1. Sửa file trong working directory.
2. Stage file, đưa snapshot của file vào staging area.
3. Commit update, tìm file trong staging area và lưu snapshot vĩnh viễn vào Git repository directory.

## Quick start với Git

### Lấy Git repository

Có hai cách để lấy Git project repository.

1. Khởi tạo repository trong directory hiện có: đi vào project directory và chạy command `git init`; command này sẽ tạo một subdirectory tên `.git`.
2. Clone một Git repository hiện có từ server: `git clone [url]`. Nếu muốn tự đặt tên local directory, có thể dùng `git clone [url] directoryname`.

### Ghi lại từng update vào repository

1. **Kiểm tra trạng thái file hiện tại**: `git status`
2. **Đề xuất thay đổi (thêm chúng vào staging area):** `git add filename` (đối với file cụ thể), `git add .` (mọi thay đổi trong directory hiện tại), `git add *.txt` (hỗ trợ wildcard, mọi file `.txt`).
3. **Ignore file:** file `.gitignore`
4. **Commit update:** `git commit -m "commit message"`. Trước mỗi lần chuẩn bị commit, hãy dùng `git status` để kiểm tra mọi thứ đã được stage chưa.
5. **Bỏ qua cách update thông qua staging area:** `git commit -a -m "commit message"`. Khi `git commit` có option `-a`, Git sẽ tự động stage và commit tất cả file đã được track, qua đó bỏ qua bước `git add`.
6. **Remove file:** `git rm filename` (remove khỏi staging area, sau đó commit.)
7. **Rename file:** `git mv README.md README` (command này tương đương với việc kết hợp ba command `mv README.md README`, `git rm README.md`, `git add README`.)

### Một Git commit message tốt

Một Git commit message tốt như sau:

```plain
Tiêu đề: Dùng dòng này để mô tả và giải thích commit lần này

Phần body có thể chỉ gồm vài dòng để bổ sung chi tiết giải thích commit, tốt nhất là đưa ra background liên quan hoặc giải thích commit này sửa và giải quyết vấn đề gì.

Phần body cũng có thể gồm vài đoạn, nhưng cần chú ý ngắt dòng và không viết câu quá dài. Như vậy khi dùng "git log", phần thụt lề sẽ trông đẹp hơn.
```

Phần mô tả của commit title nên rõ ràng nhất có thể và cố gắng tóm tắt trong một câu. Nhờ vậy các tool xem Git log liên quan có thể hiển thị thuận tiện hơn và người khác cũng dễ đọc hơn.

### Push thay đổi lên remote repository

- Nếu bạn chưa clone repository hiện có và muốn kết nối repository của mình với một remote server, có thể dùng command sau để thêm: `git remote add origin <server>`. Ví dụ, để liên kết local repository với repository đã tạo trên GitHub, có thể viết: `git remote add origin https://github.com/Snailclimb/test.git`.
- Commit các thay đổi này lên remote repository: `git push origin main`. `main` ở đây có thể thay bằng bất kỳ branch nào bạn muốn push. Default branch của nhiều project cũ vẫn có tên `master`; hãy căn cứ vào repository thực tế.

  Như vậy, bạn có thể push thay đổi lên server đã thêm.

### Remove và rename remote repository

- Rename test thành test1: `git remote rename test test1`
- Remove remote repository test1: `git remote rm test1`

### Xem commit history

Sau khi commit một số update hoặc clone một project, có thể bạn sẽ muốn xem lại commit history. Tool đơn giản và hiệu quả nhất để hoàn thành việc này là command `git log`. `git log` liệt kê toàn bộ update theo thời gian commit, update gần nhất nằm ở trên cùng.

**Có thể thêm một số parameter để xem nội dung mình muốn:**

Chỉ xem commit record của một người:

```shell
git log --author=bob
```

### Undo operation

Đôi khi sau khi commit, bạn mới phát hiện đã bỏ sót vài file chưa add hoặc viết sai commit message. Khi đó, có thể chạy commit command với option `--amend` để thử commit lại:

```shell
git commit --amend
```

Unstage file:

```shell
git restore --staged filename
```

Git version cũ cũng thường gặp cách viết sau:

```shell
git reset filename
```

Undo thay đổi của file:

```shell
git restore filename
```

Git version cũ cũng thường gặp cách viết sau:

```shell
git checkout -- filename
```

Nếu muốn discard toàn bộ thay đổi và commit ở local, có thể lấy version history mới nhất từ server rồi trỏ local main branch tới đó:

```shell
git fetch origin
git reset --hard origin/main
```

Lưu ý: `git reset --hard` sẽ discard thay đổi local chưa commit; trước khi thực hiện, nhất định phải xác nhận không có nội dung cần giữ lại. Nếu default branch của project cũ là `master`, hãy đổi command tương ứng thành `git reset --hard origin/master`.

### Branch

Branch dùng để cô lập các task phát triển khác nhau. Phát triển feature hoặc sửa bug trên branch khác, sau khi hoàn thành thì merge trở lại main branch. Hiện nay default branch của nhiều repository có tên `main`; trong project cũ cũng thường thấy `master`.

Thông thường, khi phát triển feature mới hoặc sửa một bug khẩn cấp, chúng ta sẽ chọn tạo branch. Phát triển một branch hay nhiều branch tốt hơn còn tùy vào tình huống cụ thể.

Tạo branch tên test

```shell
git branch test
```

Chuyển branch hiện tại sang `test` (khi chuyển branch, Git sẽ reset working directory để nó giống như trạng thái ở commit cuối cùng trên branch đó. Git tự động add, remove và sửa file nhằm đảm bảo working directory hiện tại giống với trạng thái của branch ở commit cuối cùng).

```shell
git switch test
```

Git version cũ cũng thường gặp cách viết `git checkout test`.

![](https://oss.javaguide.cn/github/javaguide/tools/git/2019-3%E5%88%87%E6%8D%A2%E5%88%86%E6%94%AF.png)

Bạn cũng có thể tạo branch và chuyển sang branch đó trực tiếp:

```shell
git switch -c feature_x
```

Git version cũ cũng thường gặp cách viết `git checkout -b feature_x`.

Chuyển sang main branch

```shell
git switch main
```

Merge branch (có thể xảy ra conflict)

```shell
git merge test
```

Xóa branch vừa tạo

```shell
git branch -d feature_x
```

Push branch lên remote repository (sau khi push thành công, người khác có thể nhìn thấy):

```shell
git push origin feature_x
```

## Đề xuất tài liệu học

**Tool học thông qua demo online:**

“Bổ sung, từ [issue729](https://github.com/Snailclimb/JavaGuide/issues/729)” Learn Git Branching <https://oschina.gitee.io/learn-git-branching/>. Website này thuận tiện cho việc demo các thao tác Git cơ bản, giải thích rất rõ tác dụng và kết quả của từng command cơ bản.

**Đề xuất đọc:**

- [Tutorial Git nhập môn bằng hình ảnh (15.000 chữ, 40 hình)](https://www.cnblogs.com/anding/p/16987769.html): Một bài viết rất tâm huyết, nội dung đầy đủ và có hình minh họa chi tiết, cực kỳ đề xuất!
- [Git - Hướng dẫn ngắn gọn](https://rogerdudler.github.io/git-guide/index.zh.html): Bao quát các thao tác Git thường gặp, rất rõ ràng.
- [Git bằng hình ảnh](https://marklodato.github.io/visual-git-guide/index-zh-cn.html): Minh họa các command thường dùng nhất trong Git. Nếu đã hiểu sơ bộ nguyên lý hoạt động của Git, bài viết này có thể giúp bạn hiểu sâu hơn.
- [Nhập môn Git, khỉ cũng hiểu](https://backlog.com/git-tutorial/cn/intro/intro1_1.html): Cách giải thích thú vị.
- [Pro Git book](https://git-scm.com/book/zh/v2): Một cuốn sách Git ở nước ngoài, được dịch ra nhiều ngôn ngữ và có chất lượng rất cao.

<!-- @include: @article-footer.snippet.md -->
