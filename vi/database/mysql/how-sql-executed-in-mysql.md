---
title: Quy trình thực thi câu lệnh SQL trong MySQL
description: Giải thích chi tiết toàn bộ quy trình thực thi câu lệnh SQL trong MySQL, từ xác thực danh tính tại connector, query cache, phân tích cú pháp tại analyzer, optimizer tạo execution plan đến executor gọi storage engine.
category: Database
tag:
  - MySQL
head:
  - - meta
    - name: keywords
      content: quy trình thực thi MySQL,quy trình thực thi SQL,connector,analyzer,optimizer,executor,Server layer,storage engine,InnoDB
---

> Bài viết này do [Mộc Mộc Tượng](https://github.com/kinglaw1204) đóng góp.

Bài viết này sẽ phân tích quy trình thực thi một câu lệnh SQL trong MySQL, bao gồm cách truy vấn SQL lưu chuyển bên trong MySQL và cách hoàn tất việc cập nhật bằng câu lệnh SQL.

Trước khi phân tích, trước hết hãy xem qua kiến trúc cơ bản của MySQL. Biết MySQL gồm những component nào và vai trò của chúng sẽ giúp bạn hiểu và giải quyết các vấn đề này.

## I. Phân tích kiến trúc cơ bản của MySQL

### 1.1 Tổng quan kiến trúc cơ bản của MySQL

Hình dưới đây là sơ đồ kiến trúc đơn giản của MySQL. Qua đó, bạn có thể thấy rõ câu lệnh SQL của người dùng được thực thi bên trong MySQL như thế nào.

Trước tiên, hãy giới thiệu ngắn gọn vai trò cơ bản của một số component trong hình để giúp bạn hiểu sơ đồ này. Vai trò của các component sẽ được giới thiệu chi tiết trong mục 1.2.

- **Connector:** thực hiện xác thực danh tính và xử lý quyền (khi đăng nhập MySQL).
- **Query cache:** khi thực thi câu lệnh truy vấn, trước tiên sẽ kiểm tra query cache (đã bị loại bỏ từ MySQL 8.0 vì tính năng này không thực sự hữu ích).
- **Analyzer:** nếu không hit cache, câu lệnh SQL sẽ đi qua analyzer. Nói đơn giản, analyzer trước tiên sẽ xem câu lệnh SQL của bạn muốn làm gì, sau đó kiểm tra cú pháp của câu lệnh có chính xác hay không.
- **Optimizer:** thực thi theo phương án mà MySQL cho là tối ưu nhất.
- **Executor:** thực thi câu lệnh, sau đó trả dữ liệu từ storage engine về. -

![](https://oss.javaguide.cn/javaguide/13526879-3037b144ed09eb88.png)

Nói đơn giản, MySQL chủ yếu được chia thành Server layer và storage engine layer:

- **Server layer:** chủ yếu bao gồm connector, query cache, analyzer, optimizer, executor... Tất cả chức năng dùng chung giữa các storage engine đều được triển khai ở layer này, chẳng hạn stored procedure, trigger, view, function và một module log dùng chung là binlog.
- **Storage engine:** chủ yếu chịu trách nhiệm lưu trữ và đọc dữ liệu. Nó sử dụng kiến trúc dạng plugin có thể thay thế, hỗ trợ nhiều storage engine như InnoDB, MyISAM, Memory... Trong đó, InnoDB có module log riêng là redo log. **Storage engine được sử dụng phổ biến nhất hiện nay là InnoDB; từ MySQL 5.5, nó đã được chọn làm storage engine mặc định.**

### 1.2 Giới thiệu các component cơ bản của Server layer

#### 1) Connector

Connector chủ yếu phụ trách các chức năng liên quan đến xác thực danh tính và quyền, giống như một người bảo vệ có cấp bậc rất cao.

Connector chủ yếu chịu trách nhiệm cho việc người dùng đăng nhập database và xác thực danh tính, bao gồm kiểm tra tài khoản, mật khẩu, quyền... Nếu tài khoản và mật khẩu đã được xác thực, connector sẽ tra cứu toàn bộ quyền của người dùng trong bảng quyền. Sau đó, mọi logic kiểm tra quyền trên connection này đều dựa vào dữ liệu quyền được đọc tại thời điểm đó. Nói cách khác, miễn là connection này chưa bị ngắt, người dùng sẽ không bị ảnh hưởng ngay cả khi administrator thay đổi quyền của họ.

#### 2) Query cache (đã bị loại bỏ từ MySQL 8.0)

Query cache chủ yếu dùng để cache các câu lệnh SELECT và result set của những câu lệnh đó.

Sau khi connection được thiết lập, khi thực thi câu lệnh truy vấn, trước tiên sẽ kiểm tra query cache. MySQL trước tiên kiểm tra câu lệnh SQL này đã từng được thực thi hay chưa, rồi lưu dưới dạng key-value trong memory. Key là câu lệnh truy vấn, value là result set. Nếu cache key khớp, result set sẽ được trả trực tiếp về client. Nếu không hit, các thao tác tiếp theo sẽ được thực hiện, sau đó kết quả cũng được cache để thuận tiện cho lần gọi tiếp theo. Tất nhiên, khi thực sự thực thi truy vấn từ cache, hệ thống vẫn kiểm tra quyền của người dùng và họ có quyền truy vấn bảng đó hay không.

Không nên sử dụng query cache cho truy vấn MySQL vì trong các tình huống thực tế, query cache có thể bị vô hiệu hóa rất thường xuyên. Ví dụ, nếu bạn cập nhật một bảng, toàn bộ query cache của bảng đó sẽ bị xóa. Với dữ liệu không thường xuyên cập nhật, sử dụng cache vẫn có thể chấp nhận được.

Vì vậy, trong hầu hết trường hợp, không nên sử dụng query cache.

Từ MySQL 8.0, tính năng query cache đã bị xóa. MySQL cũng cho rằng tính năng này ít được sử dụng trong các tình huống ứng dụng thực tế nên đã loại bỏ hoàn toàn.

#### 3) Analyzer

Nếu MySQL không hit cache, câu lệnh sẽ đi vào analyzer. Analyzer chủ yếu dùng để phân tích câu lệnh SQL dùng để làm gì, và cũng được chia thành một số bước:

**Bước thứ nhất, lexical analysis:** một câu lệnh SQL gồm nhiều string. Trước tiên, hệ thống trích xuất keyword như `select`, table cần truy vấn, tên field, điều kiện truy vấn... Sau khi hoàn tất các thao tác này, hệ thống chuyển sang bước thứ hai.

**Bước thứ hai, syntax analysis:** chủ yếu kiểm tra SQL bạn nhập có chính xác và có phù hợp với cú pháp MySQL hay không.

Sau khi hoàn tất hai bước này, MySQL đã sẵn sàng bắt đầu thực thi. Nhưng thực thi như thế nào để đạt kết quả tốt nhất? Lúc này cần đến optimizer.

#### 4) Optimizer

Vai trò của optimizer là chọn execution plan mà nó cho là tối ưu để thực thi. Ví dụ, khi có nhiều index thì nên chọn index nào, khi truy vấn nhiều bảng thì nên chọn thứ tự join như thế nào...

Có thể nói, sau khi đi qua optimizer, cách thực thi cụ thể của câu lệnh đã được quyết định.

#### 5) Executor

Sau khi execution plan được chọn, MySQL chuẩn bị bắt đầu thực thi. Trước hết, hệ thống kiểm tra người dùng có quyền hay không. Nếu không có quyền, hệ thống trả về error message; nếu có quyền, hệ thống gọi API của engine và trả về kết quả thực thi.

## II. Phân tích câu lệnh

### 2.1 Câu lệnh truy vấn

Sau những phần trên, một câu lệnh SQL thực sự được thực thi như thế nào? SQL có thể được chia thành hai loại: truy vấn và cập nhật (thêm, sửa, xóa). Trước tiên, hãy phân tích câu lệnh truy vấn. Câu lệnh như sau:

```sql
select * from tb_student  A where A.age='18' and A.name=' Trương Tam ';
```

Dựa trên phần giải thích ở trên, hãy phân tích quy trình thực thi câu lệnh này:

- Trước tiên, connector thực hiện xác thực danh tính và lấy quyền (nếu xác thực thất bại thì từ chối trực tiếp). Trước MySQL 8.0, sau khi xác thực thành công, hệ thống sẽ kiểm tra query cache, dùng câu lệnh SQL này làm key để tra cứu trong memory xem có kết quả hay không. Nếu có thì trả về kết quả trong cache; nếu không thì thực hiện bước tiếp theo.
- Analyzer thực hiện lexical analysis, trích xuất các thành phần quan trọng của câu lệnh SQL. Ví dụ, từ câu lệnh trên trích xuất được đây là truy vấn `select`, table cần truy vấn là `tb_student`, cần truy vấn tất cả column và điều kiện truy vấn là `id='1'`. Sau đó, hệ thống kiểm tra câu lệnh SQL có lỗi cú pháp hay không, chẳng hạn keyword có chính xác hay không. Nếu không có vấn đề thì thực hiện bước tiếp theo.
- Tiếp theo, optimizer xác định execution plan. Với câu lệnh SQL trên, có hai execution plan: a. trước tiên tìm các sinh viên có tên là “Trương Tam” trong bảng sinh viên, sau đó kiểm tra tuổi có phải 18 hay không; b. trước tiên tìm các sinh viên 18 tuổi, sau đó tìm những sinh viên có tên là “Trương Tam”. Optimizer chọn plan có hiệu suất thực thi tốt hơn theo thuật toán tối ưu của mình (theo optimizer, đôi khi plan được chọn chưa chắc là tốt nhất). Sau khi execution plan được xác định, hệ thống chuẩn bị bắt đầu thực thi.

- Kiểm tra quyền. Nếu không có quyền thì trả về error message; nếu có quyền thì gọi API của database engine và trả về execution result của engine.

### 2.2 Câu lệnh cập nhật

Trên đây là quy trình thực thi một câu lệnh truy vấn SQL. Tiếp theo, hãy xem một câu lệnh cập nhật được thực thi như thế nào. Câu lệnh SQL như sau:

```plain
update tb_student A set A.age='19' where A.name=' Trương Tam ';
```

Hãy thử sửa tuổi của Trương Tam. Trong database thực tế chắc chắn sẽ không đặt field là tuổi, nếu không sẽ bị người phụ trách kỹ thuật đánh. Về cơ bản, câu lệnh này cũng đi theo flow truy vấn ở trên, chỉ khác là khi thực thi cập nhật chắc chắn phải ghi log, từ đó đưa module log vào flow. Module log đi kèm MySQL là **binlog (archive log)**, mọi storage engine đều có thể sử dụng. Storage engine InnoDB được sử dụng phổ biến còn có một module log là **redo log (redo log)**. Hãy lấy InnoDB làm ví dụ để phân tích flow thực thi câu lệnh này. Flow như sau:

- Trước tiên tìm bản ghi của Trương Tam. Query cache sẽ không được sử dụng vì quy tắc thiết kế của query cache là chỉ phục vụ các câu lệnh truy vấn.
- Sau đó lấy câu lệnh truy vấn, đổi `age` thành 19, gọi API của engine để ghi row này. Engine InnoDB lưu dữ liệu trong memory, đồng thời ghi redo log. Lúc này redo log chuyển sang trạng thái prepare, sau đó báo cho executor rằng đã thực thi xong và có thể commit bất cứ lúc nào.
- Sau khi nhận được thông báo, executor ghi binlog rồi xóa query cache của bảng. Xóa ở thời điểm này đảm bảo các câu lệnh SELECT tiếp theo không đọc cache cũ — vì transaction sắp commit, dữ liệu sắp chuyển sang trạng thái mới nhất, thời điểm invalidation của cache vừa khớp với thời điểm dữ liệu thực sự được cập nhật.
- Executor gọi API của engine, commit redo log sang trạng thái commit.
- Hoàn tất cập nhật.

**Chắc hẳn sẽ có bạn thắc mắc: tại sao phải dùng hai module log, dùng một module log không được sao?**

Đó là vì ban đầu MySQL chưa có engine InnoDB (engine InnoDB do một công ty khác tích hợp vào MySQL dưới dạng plugin). Engine đi kèm MySQL là MyISAM. Tuy nhiên, redo log là tính năng riêng của engine InnoDB, các storage engine khác đều không có, khiến hệ thống không có khả năng crash-safe (khả năng crash-safe nghĩa là ngay cả khi database restart bất thường, các record đã commit trước đó cũng không bị mất). Binlog chỉ có thể dùng để archive.

Không phải không thể chỉ dùng một module log, mà là engine InnoDB hỗ trợ transaction thông qua redo log. Vậy sẽ có bạn hỏi tiếp: đã dùng hai module log thì có thể không phức tạp như vậy không? Tại sao redo log phải có trạng thái prepare? Hãy dùng phương pháp phản chứng để giải thích vì sao phải làm như vậy.

- **Ghi redo log và commit trực tiếp trước, sau đó ghi binlog:** giả sử sau khi ghi xong redo log thì máy bị dừng, binlog chưa được ghi. Sau khi máy restart, máy sẽ khôi phục dữ liệu thông qua redo log, nhưng binlog không ghi lại dữ liệu này. Khi backup máy sau đó, record này sẽ bị mất, đồng thời việc đồng bộ master-slave cũng mất record này.
- **Ghi binlog trước, sau đó ghi redo log:** giả sử binlog đã được ghi xong thì máy restart bất thường. Vì chưa có redo log nên máy không thể khôi phục record này, nhưng binlog lại có record. Tương tự trường hợp trên, điều này sẽ tạo ra tình trạng dữ liệu không nhất quán.

Nếu sử dụng phương thức two-phase commit của redo log thì sẽ khác. Trước tiên ghi xong redo log và đánh dấu là prepare, ngay sau đó ghi xong binlog rồi đánh dấu redo log là commit. Cách này có thể ngăn các vấn đề nêu trên, từ đó đảm bảo tính nhất quán của dữ liệu.

Vậy nếu xảy ra một tình huống cực đoan thì sao? Giả sử redo log đang ở trạng thái prepare và binlog cũng đã ghi xong, sau đó xảy ra restart bất thường thì điều gì sẽ xảy ra?

Điều này phụ thuộc vào cơ chế xử lý của MySQL. Quy trình xử lý của MySQL như sau:

- Kiểm tra redo log có ở trạng thái commit hay không. Nếu có, nghĩa là binlog chắc chắn đã flush xuống disk thành công, nên commit ngay.
- Nếu redo log chỉ ở trạng thái prepare nhưng chưa ở trạng thái commit, hệ thống sẽ dùng XID của transaction để kiểm tra trong binlog xem transaction đó đã flush xuống disk thành công hay chưa. Nếu đã thành công thì commit redo log, nếu chưa thì rollback transaction.

Như vậy, vấn đề nhất quán dữ liệu được giải quyết.

## III. Tổng kết

- MySQL chủ yếu được chia thành Server layer và engine layer. Server layer chủ yếu bao gồm connector, query cache, analyzer, optimizer, executor, đồng thời có một module log (binlog). Module log này có thể được dùng chung bởi mọi storage engine, còn redo log chỉ có ở InnoDB.
- Engine layer sử dụng kiến trúc dạng plugin, hiện chủ yếu bao gồm MyISAM, InnoDB, Memory...
- Flow thực thi câu lệnh truy vấn như sau: kiểm tra quyền (nếu hit cache) ---> query cache ---> analyzer ---> optimizer ---> kiểm tra quyền ---> executor ---> engine
- Flow thực thi câu lệnh cập nhật như sau: analyzer ----> kiểm tra quyền ----> executor ---> engine ---> redo log (trạng thái prepare) ---> binlog ---> redo log (trạng thái commit)

## IV. Tài liệu tham khảo

- 《45 bài giảng thực chiến về MySQL》
- Sổ tay tham khảo MySQL 5.6:<https://dev.MySQL.com/doc/refman/5.6/en/>

<!-- @include: @article-footer.snippet.md -->
