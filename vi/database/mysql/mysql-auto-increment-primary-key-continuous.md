---
title: Khóa chính tự tăng của MySQL có nhất thiết liên tục không?
description: Giải thích chi tiết nguyên nhân khóa chính tự tăng của MySQL không liên tục, phân tích cơ chế phân bổ giá trị tự tăng trong các trường hợp xung đột unique key, rollback transaction, insert hàng loạt, cùng cấu hình và ảnh hưởng của chế độ lock tự tăng InnoDB.
category: Database
tag:
  - MySQL
  - Phỏng vấn công ty lớn
head:
  - - meta
    - name: keywords
      content: MySQL tự tăng khóa chính,AUTO_INCREMENT,khóa chính không liên tục,rollback transaction,insert hàng loạt,xung đột unique key,innodb_autoinc_lock_mode
---

> Tác giả: Phi Thiên Tiểu Ngưu Nhục
>
> Bài viết gốc: <https://mp.weixin.qq.com/s/qci10h9rJx_COZbHV3aygQ>

Như ai cũng biết, khóa chính tự tăng có thể giúp clustered index duy trì thứ tự insert tăng dần hết mức có thể, tránh random query, từ đó nâng cao hiệu suất query.

Nhưng trên thực tế, khóa chính tự tăng của MySQL không thể bảo đảm luôn tăng liên tục.

Hãy xem một ví dụ dưới đây. Trước tiên, tạo một table như sau:

![](https://oss.javaguide.cn/p3-juejin/3e6b80ba50cb425386b80924e3da0d23~tplv-k3u1fbpfcp-zoom-1.png)

## Giá trị tự tăng được lưu ở đâu?

Dùng `insert into test_pk values(null, 1, 1)` để insert một row, sau đó thực thi command `show create table` để xem định nghĩa cấu trúc của table:

![](https://oss.javaguide.cn/p3-juejin/c17e46230bd34150966f0d86b2ad5e91~tplv-k3u1fbpfcp-zoom-1.png)

Định nghĩa cấu trúc của table trên được lưu trong file local có hậu tố `.frm`. Có thể tìm thấy file `.frm` này trong thư mục data của thư mục cài đặt MySQL:

![](https://oss.javaguide.cn/p3-juejin/3ec0514dd7be423d80b9e7f2d52f5902~tplv-k3u1fbpfcp-zoom-1.png)

Từ cấu trúc table trên có thể thấy trong định nghĩa table xuất hiện `AUTO_INCREMENT=2`, nghĩa là khi insert dữ liệu lần tiếp theo, nếu cần tự động tạo giá trị tự tăng thì sẽ tạo `id = 2`.

Tuy nhiên, cần lưu ý rằng giá trị tự tăng không được lưu trong định nghĩa table, tức file `.frm`. Các engine khác nhau có chiến lược lưu giá trị tự tăng khác nhau:

1. Giá trị tự tăng của engine MyISAM được lưu trong data file.

2. Giá trị tự tăng của engine InnoDB thực ra được lưu trong memory và không được persistence. Mỗi khi mở table lần đầu, engine sẽ tìm giá trị lớn nhất của giá trị tự tăng `max(id)`, sau đó lấy `max(id)+1` làm giá trị tự tăng hiện tại của table.

Ví dụ: hiện tại row có `id` lớn nhất trong table của chúng ta là 1, `AUTO_INCREMENT=2`, đúng không? Lúc này, chúng ta xóa row có `id=1`, `AUTO_INCREMENT` vẫn là 2.

![](https://oss.javaguide.cn/p3-juejin/61b8dc9155624044a86d91c368b20059~tplv-k3u1fbpfcp-zoom-1.png)

Nhưng nếu lập tức restart MySQL instance, sau khi restart, `AUTO_INCREMENT` của table này sẽ trở thành 1. Nói cách khác, việc restart MySQL có thể thay đổi giá trị `AUTO_INCREMENT` của một table.

![](https://oss.javaguide.cn/p3-juejin/27fdb15375664249a31f88b64e6e5e66~tplv-k3u1fbpfcp-zoom-1.png)

![](https://oss.javaguide.cn/p3-juejin/dee15f93e65d44d384345a03404f3481~tplv-k3u1fbpfcp-zoom-1.png)

Trên đây là thử nghiệm trên phiên bản MySQL 5.x local của tôi. Trên thực tế, **từ MySQL 8.0, record thay đổi của giá trị tự tăng được đặt trong redo log, cung cấp khả năng persistence cho giá trị tự tăng**, nghĩa là thực hiện được việc “nếu xảy ra restart, giá trị tự tăng của table có thể được khôi phục về giá trị trước khi MySQL restart dựa trên redo log”.

Nói cách khác, với ví dụ trên, sau khi restart instance, `AUTO_INCREMENT` của table này vẫn là 2.

Sau khi hiểu giá trị tự tăng của MySQL được lưu ở đâu, chúng ta hãy xem tiếp cơ chế thay đổi giá trị tự tăng, từ đó dẫn đến scenario đầu tiên khiến giá trị tự tăng không liên tục.

## Các scenario khiến giá trị tự tăng không liên tục

### Scenario 1: Giá trị tự tăng không liên tục

Trong MySQL, nếu field `id` được định nghĩa là `AUTO_INCREMENT`, khi insert một row, giá trị tự tăng hoạt động như sau:

- Nếu field `id` được chỉ định là 0, `null` hoặc không chỉ định giá trị khi insert dữ liệu, giá trị `AUTO_INCREMENT` hiện tại của table sẽ được điền vào field tự tăng.
- Nếu field `id` được chỉ định một giá trị cụ thể khi insert dữ liệu, câu lệnh sẽ trực tiếp sử dụng giá trị đã chỉ định.

Quan hệ lớn nhỏ giữa giá trị cần insert và giá trị tự tăng hiện tại sẽ quyết định kết quả thay đổi của giá trị tự tăng. Giả sử giá trị cần insert là `insert_num`, giá trị tự tăng hiện tại là `autoIncrement_num`:

- Nếu `insert_num < autoIncrement_num`, giá trị tự tăng của table không thay đổi.
- Nếu `insert_num >= autoIncrement_num`, cần thay đổi giá trị tự tăng hiện tại thành giá trị tự tăng mới.

Nói cách khác, nếu `id` cần insert là 100, giá trị tự tăng hiện tại là 90, `insert_num >= autoIncrement_num`, thì giá trị tự tăng sẽ được thay đổi thành giá trị tự tăng mới là 101.

Có nhất thiết là như vậy không?

Không hẳn.

Những bạn đã tìm hiểu về distributed id chắc chắn biết rằng, để tránh primary key được tạo bởi hai database bị conflict, có thể thiết lập id tự tăng của một database đều là số lẻ, còn id tự tăng của database kia đều là số chẵn.

Việc dùng số lẻ hay số chẵn được quyết định bởi hai parameter `auto_increment_offset` và `auto_increment_increment`. Hai parameter này lần lượt biểu thị giá trị khởi đầu và step của giá trị tự tăng, giá trị mặc định đều là 1.

Vì vậy, trong ví dụ trên, các bước tạo giá trị tự tăng mới thực tế là: bắt đầu từ `auto_increment_offset`, liên tục cộng thêm `auto_increment_increment` làm step, cho đến khi tìm được giá trị đầu tiên lớn hơn 100, dùng giá trị đó làm giá trị tự tăng mới.

Vì vậy, trong trường hợp này, giá trị tự tăng có thể là 102, 103, v.v., dẫn đến `id` của primary key không liên tục.

Đáng tiếc hơn, ngay cả khi giá trị khởi đầu và step của giá trị tự tăng đều được đặt là 1, `id` của primary key tự tăng cũng không nhất thiết bảo đảm primary key liên tục.

### Scenario 2: Giá trị tự tăng không liên tục

Ví dụ, hiện tại chúng ta insert một record `(null,1,1)` vào table, primary key được tạo là 1, `AUTO_INCREMENT=2`, đúng không?

![](https://oss.javaguide.cn/p3-juejin/c22c4f2cea234c7ea496025eb826c3bc~tplv-k3u1fbpfcp-zoom-1.png)

Lúc này, nếu thực thi thêm một command insert `(null,1,1)`, hiển nhiên sẽ báo lỗi `Duplicate entry`, vì chúng ta đã thiết lập field `a` là unique index:

![](https://oss.javaguide.cn/p3-juejin/c0325e31398d4fa6bb1cbe08ef797b7f~tplv-k3u1fbpfcp-zoom-1.png)

Nhưng bạn sẽ ngạc nhiên khi phát hiện rằng, dù insert thất bại, giá trị tự tăng vẫn tăng từ 2 lên 3!

Tại sao lại như vậy?

Hãy phân tích flow thực thi của câu lệnh insert này:

1. Executor gọi interface của engine InnoDB để chuẩn bị insert một record `(null,1,1)`.
2. InnoDB phát hiện user không chỉ định giá trị của id tự tăng, nên lấy giá trị tự tăng hiện tại 2 của table `test_pk`.
3. Thay record được truyền vào thành `(2,1,1)`.
4. Thay giá trị tự tăng của table thành 3.
5. Tiếp tục thực thi thao tác insert dữ liệu. Do record có `a=1` đã tồn tại, nên báo `Duplicate key error` và trả về.

Có thể thấy thao tác thay đổi giá trị tự tăng diễn ra trước thao tác insert dữ liệu thực sự.

Khi câu lệnh này thực sự được thực thi, do gặp xung đột unique key `a`, row có `id = 2` không insert thành công, nhưng giá trị tự tăng cũng không được thay đổi trở lại. Vì vậy, khi insert row mới sau đó, `id` tự tăng nhận được sẽ là 3. Nói cách khác, primary key tự tăng không liên tục.

Đến đây, chúng ta đã liệt kê hai trường hợp khiến primary key tự tăng không liên tục:

1. Giá trị khởi đầu và step của giá trị tự tăng không được đặt là 1.
2. Xung đột unique key.

Ngoài ra, rollback transaction cũng dẫn đến tình trạng này.

### Scenario 3: Giá trị tự tăng không liên tục

Hiện tại trong table của chúng ta có một record `(1,1,1)`, `AUTO_INCREMENT = 3`:

![](https://oss.javaguide.cn/p3-juejin/6220fcf7dac54299863e43b6fb97de3e~tplv-k3u1fbpfcp-zoom-1.png)

Trước tiên, chúng ta insert một row `(null, 2, 2)`, tức là `(3, 2, 2)`, đồng thời `AUTO_INCREMENT` thay đổi thành 4:

![](https://oss.javaguide.cn/p3-juejin/3f02d46437d643c3b3d9f44a004ab269~tplv-k3u1fbpfcp-zoom-1.png)

Sau đó thực thi đoạn SQL sau:

![](https://oss.javaguide.cn/p3-juejin/faf5ce4a2920469cae697f845be717f5~tplv-k3u1fbpfcp-zoom-1.png)

Mặc dù chúng ta đã insert một record `(null, 3, 3)`, nhưng vì dùng `rollback` để rollback nên trong database không có record này:

![](https://oss.javaguide.cn/p3-juejin/6cb4c02722674dd399939d3d03a431c1~tplv-k3u1fbpfcp-zoom-1.png)

Trong trường hợp rollback transaction này, giá trị tự tăng không rollback theo! Như hình dưới đây, giá trị tự tăng vẫn cố định tăng từ 4 lên 5:

![](https://oss.javaguide.cn/p3-juejin/e6eea1c927424ac7bda34a511ca521ae~tplv-k3u1fbpfcp-zoom-1.png)

Vì vậy, nếu tiếp tục insert một row `(null, 3, 3)`, primary key `id` sẽ được tự động gán là `5`:

![](https://oss.javaguide.cn/p3-juejin/80da69dd13b543c4a32d6ed832a3c568~tplv-k3u1fbpfcp-zoom-1.png)

Vậy tại sao khi xảy ra xung đột unique key hoặc rollback, MySQL không thay đổi giá trị tự tăng của table trở lại? Nếu rollback thì chẳng phải `id` tự tăng sẽ không bị gián đoạn sao?

Trên thực tế, nguyên nhân chính để làm như vậy là nâng cao performance.

Hãy trực tiếp dùng phương pháp phản chứng để kiểm chứng: giả sử MySQL sẽ thay đổi giá trị tự tăng trở lại khi transaction rollback, điều gì sẽ xảy ra?

Hiện có hai transaction A và B thực thi song song. Khi request giá trị tự tăng, để tránh hai transaction request trùng `id` tự tăng, chắc chắn cần lock rồi request lần lượt, đúng không?

1. Giả sử transaction A nhận được `id = 1`, transaction B nhận được `id=2`, lúc này giá trị tự tăng của table `t` là 3, sau đó tiếp tục thực thi.
2. Transaction B commit thành công, nhưng transaction A gặp xung đột unique key, tức row có `id = 1` insert thất bại. Nếu cho phép transaction A rollback `id` tự tăng, tức thay đổi giá trị tự tăng hiện tại của table trở lại 1, sẽ xảy ra tình huống sau: table đã có row với `id = 2`, nhưng giá trị `id` tự tăng hiện tại là 1.
3. Khi các transaction khác tiếp tục thực thi, chúng sẽ request được `id=2`. Lúc này, câu lệnh insert sẽ báo “primary key conflict”.

![](https://oss.javaguide.cn/p3-juejin/5f26f02e60f643c9a7cab88a9f1bdce9~tplv-k3u1fbpfcp-zoom-1.png)

Để giải quyết xung đột primary key này, có hai cách:

1. Trước mỗi lần request `id`, kiểm tra trước xem `id` này đã tồn tại trong table hay chưa. Nếu đã tồn tại thì bỏ qua `id` này.
2. Mở rộng phạm vi lock của `id` tự tăng: phải chờ một transaction thực thi xong và commit thì transaction tiếp theo mới được request `id` tự tăng.

Hiển nhiên, chi phí của hai cách trên đều khá cao và sẽ gây ra vấn đề performance. Xét đến cùng, nguyên nhân nằm ở giả định “cho phép `id` tự tăng rollback”.

Vì vậy, InnoDB từ bỏ thiết kế này: câu lệnh thực thi thất bại cũng không rollback `id` tự tăng. Cũng chính vì vậy, MySQL chỉ bảo đảm `id` tự tăng tăng dần, chứ không bảo đảm liên tục.

Tóm lại, chúng ta đã phân tích ba trường hợp khiến giá trị tự tăng không liên tục. Còn trường hợp thứ tư là insert dữ liệu hàng loạt.

### Scenario 4: Giá trị tự tăng không liên tục

Đối với câu lệnh insert dữ liệu hàng loạt, MySQL có chiến lược request `id` tự tăng theo batch:

1. Trong quá trình thực thi câu lệnh, lần đầu request `id` tự tăng sẽ được phân bổ 1 `id`.
2. Sau khi dùng hết 1 `id`, lần thứ hai câu lệnh này request `id` tự tăng sẽ được phân bổ 2 `id`.
3. Sau khi dùng hết 2 `id`, vẫn là câu lệnh này, lần thứ ba sẽ được phân bổ 4 `id`.
4. Cứ tiếp tục như vậy, mỗi lần cùng một câu lệnh request `id` tự tăng, số lượng `id` tự tăng được phân bổ sẽ gấp đôi lần trước.

Cần lưu ý, insert dữ liệu hàng loạt được nói ở đây không phải là chứa nhiều giá trị `value` trong câu lệnh insert thông thường!!! Vì khi request `id` tự tăng cho loại câu lệnh này, có thể tính chính xác cần bao nhiêu `id`, sau đó request một lần; sau khi request xong thì có thể release lock.

Còn với các loại câu lệnh như `insert … select`, `replace …… select` và `load data`, MySQL không biết chính xác cần request bao nhiêu `id`, nên dùng chiến lược request theo batch này. Dù sao request từng `id` một thực sự quá chậm.

Ví dụ, giả sử hiện tại table của chúng ta có dữ liệu như sau:

![](https://oss.javaguide.cn/p3-juejin/6453cfc107f94e3bb86c95072d443472~tplv-k3u1fbpfcp-zoom-1.png)

Chúng ta tạo một table `test_pk2` có cùng định nghĩa cấu trúc với table `test_pk` hiện tại:

![](https://oss.javaguide.cn/p3-juejin/45248a6dc34f431bba14d434bee2c79e~tplv-k3u1fbpfcp-zoom-1.png)

Sau đó dùng `insert...select` để insert dữ liệu hàng loạt vào table `teset_pk2`:

![](https://oss.javaguide.cn/p3-juejin/c1b061e86bae484694d15ceb703b10ca~tplv-k3u1fbpfcp-zoom-1.png)

Có thể thấy dữ liệu đã được import thành công.

Tiếp theo hãy xem giá trị tự tăng của `test_pk2` là bao nhiêu:

![](https://oss.javaguide.cn/p3-juejin/0ff9039366154c738331d64ebaf88d3b~tplv-k3u1fbpfcp-zoom-1.png)

Theo phân tích trên, giá trị này là 8 chứ không phải 6.

Cụ thể, `insert……select` thực tế đã insert 5 row vào table: `(1,1)`, `(2,2)`, `(3,3)`, `(4,4)`, `(5,5)`. Tuy nhiên, 5 row này được request `id` tự tăng trong 3 lần. Kết hợp với chiến lược request theo batch, số lượng `id` tự tăng nhận được mỗi lần gấp đôi lần trước, nên:

- Lần request đầu tiên nhận được 1 `id`: `id=1`.
- Lần request thứ hai được phân bổ 2 `id`: `id=2` và `id=3`.
- Lần request thứ ba được phân bổ 4 `id`: `id=4`, `id = 5`, `id = 6`, `id=7`.

Vì câu lệnh này thực tế chỉ dùng 5 `id`, nên `id=6` và `id=7` bị lãng phí. Sau đó, khi thực thi `insert into test_pk2 values(null,6,6)`, dữ liệu thực tế được insert là `(8,6,6)`:

![](https://oss.javaguide.cn/p3-juejin/51612fbac3804cff8c5157df21d6e355~tplv-k3u1fbpfcp-zoom-1.png)

## Tóm tắt

Bài viết này tổng hợp 4 trường hợp khiến giá trị tự tăng không liên tục:

1. Giá trị khởi đầu và step của giá trị tự tăng không được đặt là 1.
2. Xung đột unique key.
3. Rollback transaction.
4. Insert hàng loạt (chẳng hạn câu lệnh `insert...select`).

<!-- @include: @article-footer.snippet.md -->
