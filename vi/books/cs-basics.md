---
title: Những cuốn sách kinh điển không thể bỏ qua về Computer Science Basics
description: Gợi ý sách về Computer Science Basics, tổng hợp giáo trình kinh điển và tài nguyên học tập về Operating System, Computer Network, Algorithms and Data Structures, Compiler Principles và các môn cốt lõi khác.
category: Sách Computer Science
icon: "mdi:desktop-classic"
head:
  - - meta
    - name: keywords
      content: Tuyển chọn sách Computer Science Basics
---

Vì nhiều bạn khá thích xem video, nên phần này không chỉ gợi ý sách mà còn giới thiệu thêm một số video hướng dẫn và Project của các trường đại học mà mình thấy hữu ích.

## Operating System

**Vì sao cần học Operating System?**

**Xét về việc nâng cao năng lực cá nhân**, nhiều tư tưởng và thuật toán kinh điển trong Operating System đều có thể được tìm thấy trong các công cụ hoặc framework mà bạn sử dụng khi phát triển hằng ngày. Chẳng hạn, cache mà hệ thống sử dụng (ví dụ Redis) khá giống với cache tốc độ cao của Operating System. CPU có nhiều loại cache tốc độ cao, nhưng phần lớn đều nhằm giải quyết vấn đề tốc độ xử lý của CPU và bộ nhớ không tương đương nhau. Ta cũng có thể xem bộ nhớ là cache tốc độ cao của bộ nhớ ngoài: khi chương trình chạy, dữ liệu từ bộ nhớ ngoài được sao chép vào bộ nhớ; do tốc độ xử lý của bộ nhớ cao hơn nhiều so với bộ nhớ ngoài nên tốc độ xử lý được cải thiện. Tương tự, Redis cache được dùng để giải quyết sự chênh lệch giữa tốc độ xử lý của chương trình và tốc độ truy cập database quan hệ thông thường. Cache tốc độ cao thường dựa trên nguyên lý locality (2-8 principle) và thuật toán loại bỏ tương ứng để đảm bảo dữ liệu trong cache là dữ liệu thường được truy cập. Redis cache mà chúng ta thường dùng cũng nhiều khi áp dụng 2-8 principle; nhiều thuật toán loại bỏ tương tự các thuật toán trong Operating System. Đã nói đến 2-8 principle thì không thể không nhắc đến hit rate, khái niệm dùng chung cho mọi loại cache. Nói đơn giản, đó là tỷ lệ dữ liệu bạn cần truy cập có thể được tìm thấy trực tiếp trong cache. Hit rate cao thường cho thấy thiết kế cache hợp lý và tốc độ xử lý của hệ thống cũng nhanh hơn.

**Xét từ góc độ phỏng vấn**, đặc biệt là tuyển dụng sinh viên mới tốt nghiệp, kiến thức về Operating System được kiểm tra rất nhiều.

**Nói ngắn gọn, học Operating System có thể nâng cao chiều sâu tư duy và khả năng hiểu công nghệ; kiến thức về Operating System cũng là phần bắt buộc khi phỏng vấn.**

Nếu muốn học Operating System một cách hệ thống, cuốn sách chuyên sâu và có tính nền tảng nhất là **[《Nhập môn Operating System》](https://book.douban.com/subject/33463930/)**. Bạn có thể đọc thêm **[《Hiểu sâu Computer Systems》](https://book.douban.com/subject/1230413/)** để hiểu sâu hơn về bản chất của Computer Systems, rất đáng đọc!

![](https://oss.javaguide.cn/github/javaguide/booksimage-20201012191645919.png)

Ngoài ra, một cuốn sách Operating System trong nước mới xuất bản năm ngoái cũng rất hay: **[《Operating System hiện đại: Nguyên lý và triển khai》](https://book.douban.com/subject/35208251/)** (tác phẩm của đội ngũ thầy Hạ và thầy Trần, rất đáng giới thiệu).

![](https://oss.javaguide.cn/github/javaguide/books/20210406132050845.png)

Nếu thích thực hành và không quá hứng thú với kiến thức lý thuyết, bạn nên xem **[《30 ngày tự viết Operating System》](https://book.douban.com/subject/11530329/)**; cuốn sách này sẽ hướng dẫn từng bước cách viết một Operating System.

Học trên giấy cuối cùng vẫn thấy nông, muốn hiểu rõ việc này phải tự mình thực hành! Rất khuyến khích các bạn học Computer Science thực hành thật nhiều!!!

![](https://oss.javaguide.cn/github/javaguide/booksimage-20220409123802972.png)

Một số sách liên quan khác:

- **[《Tự mình viết Operating System》](https://book.douban.com/subject/1422377/)**: Không chỉ phân tích chi tiết nền tảng của nguyên lý Operating System, sách còn dùng nhiều ví dụ code để hướng dẫn từng bước cách viết một framework Operating System có các chức năng cơ bản bằng ngôn ngữ C và Assembly.
- **[《Operating System hiện đại》](https://book.douban.com/subject/3852290/)**: Nội dung khá tốt, nhưng bản dịch ở mức trung bình. Nếu đọc kỹ cuốn này, bạn nên làm hết bài tập cuối chương.
- **[《Khôi phục chân tướng Operating System》](https://book.douban.com/subject/26745156/)**: Tác giả tốt nghiệp Đại học Bắc Kinh và từng là kỹ sư vận hành cấp cao tại Baidu. Vì từng phải học lại môn Operating System khi còn học đại học, sau đó tác giả đã nghiên cứu sâu về Operating System và viết nên cuốn sách này.
- **[《Khám phá chuyên sâu Linux Operating System》](https://book.douban.com/subject/25743846/)**: Theo nội dung cuốn sách, bạn sẽ có nhận thức rõ ràng về cách tạo ra một hệ thống GNU/Linux hoàn chỉnh.
- **[《Thiết kế và triển khai Operating System》](https://book.douban.com/subject/2044818/)**: Giáo trình giảng dạy có tính nền tảng về Operating System.
- **[《Orange'S: Một Operating System được triển khai》](https://book.douban.com/subject/3735649/)**: Bắt đầu từ code boot sector chỉ có hai mươi dòng, sách từng bước trình bày quá trình hoàn thiện một framework Operating System. Đọc cùng 《Thiết kế và triển khai Operating System》 sẽ hiệu quả hơn!

Nếu thích xem video, bạn nên xem khóa học trực tuyến [《Operating System》](https://www.icourse163.org/course/HIT-1002531008) do thầy Lý Trị Quân của Đại học Công nghệ Cáp Nhĩ Tân giảng dạy; chất lượng vượt trội so với nhiều khóa học cấp quốc gia.

Đề cương khóa học như sau:

![Đề cương khóa học](https://oss.javaguide.cn/github/javaguide/books/image-20220414144527747.png)

Khóa học chủ yếu giới thiệu sáu module cơ bản trong một Operating System: quản lý CPU, quản lý bộ nhớ, quản lý thiết bị ngoại vi, quản lý disk và file system, user interface và module khởi động.

Khóa học khá khó, đặc biệt là lab sau mỗi chương. Nếu muốn thực sự hiểu nguyên lý bên trong của Operating System, bạn nên cố gắng làm các lab tương ứng. Đúng như thầy Lý Trị Quân đã nói: “Học trên giấy cuối cùng vẫn thấy nông, muốn hiểu rõ việc này phải tự mình thực hành”.

![](https://oss.javaguide.cn/github/javaguide/books/image-20220414145210679.png)

Nếu có thể tự hoàn thành vài lab, mình tin rằng mức độ hiểu Operating System của bạn sẽ tăng lên đáng kể. Tất nhiên, nếu chỉ muốn ôn cấp tốc cho phỏng vấn thì không cần làm lab.

Nói thật lòng, mình rất thích các bài giảng của thầy Lý Trị Quân. Mình cho rằng thầy là một giảng viên giỏi hiếm có trong nước. Thầy hiểu khoảng cách giữa giáo trình trong nước và nước ngoài nằm ở đâu, cũng hiểu khoảng cách giữa sinh viên trong nước và nước ngoài nằm ở đâu; thầy đang nỗ lực thu hẹp khoảng cách đó bằng cách riêng của mình. Chân thành cảm ơn và mong chờ khóa học tiếp theo của thầy Lý Trị Quân.

![](https://oss.javaguide.cn/github/javaguide/books/image-20220414145249714.png)

Ngoài ra, khóa học nước ngoài [《Hiểu sâu Computer Systems》](https://www.bilibili.com/video/av31289365?from=search&seid=16298868573410423104) dưới đây cũng khá hay.

![](https://oss.javaguide.cn/github/javaguide/booksimage-20201204140653318.png)

## Computer Network

Computer Network là một môn chuyên ngành có tính hệ thống cao; giáo trình Computer Network của các trường đại học danh tiếng nhìn chung đã khá hoàn thiện.

Muốn học tốt Computer Network, trước hết cần hiểu OSI seven-layer model hoặc TCP/IP five-layer model, gồm Application Layer (Application Layer, Presentation Layer, Session Layer), Transport Layer, Network Layer, Data Link Layer và Physical Layer.

![OSI seven-layer model](https://oss.javaguide.cn/github/javaguide/booksosi%E4%B8%83%E5%B1%82%E6%A8%A1%E5%9E%8B2.png)

Về môn học này, sách tham khảo đầu tiên được đặc biệt khuyến nghị là **《Computer Network — Phương pháp top-down》** của Nhà xuất bản Công nghiệp Cơ khí. Mục lục sách rõ ràng, giải thích lần lượt theo TCP/IP five-layer model, đồng thời thảo luận chi tiết về các công nghệ liên quan đến từng layer. Về cơ bản, đề cương giảng dạy môn học này ở các trường đại học chính là mục lục của cuốn sách.

![](https://oss.javaguide.cn/github/javaguide/booksimage-20220409123250570.png)

Nếu thấy cuốn sách trên hơi khô khan, mình đặc biệt khuyến nghị hai cuốn sách thú vị về network dưới đây:

- [《HTTP qua hình ảnh》](https://book.douban.com/subject/25863515/ "HTTP qua hình ảnh"): Giải thích HTTP theo cách như truyện tranh, rất thú vị và không gây khô khan, đồng thời bao quát phần lớn kiến thức HTTP thường gặp. Do giới hạn về độ dài, nội dung có thể chưa toàn diện. Tuy nhiên, nếu không chuyên nghiên cứu về network mà chỉ muốn tìm hiểu kiến thức HTTP, đọc cuốn này nhìn chung là đủ.
- [《Network được kết nối như thế nào》](https://book.douban.com/subject/26941639/ "Network được kết nối như thế nào"): Theo dõi toàn bộ quá trình từ lúc nhập địa chỉ trên browser đến khi hiển thị nội dung trang web; sách dùng hình ảnh kèm nội dung để giải thích toàn cảnh network, đồng thời tập trung giới thiệu cách các thiết bị và software network thực tế hoạt động.

![](https://oss.javaguide.cn/github/javaguide/booksimage-20201011215144139.png)

Ngoài kiến thức lý thuyết, một điểm rất quan trọng khi học Computer Network là: “**thực hành**”. Điều này cũng giống như việc lập trình.

Trên GitHub có một số bài thực hành/Project Computer Network của các trường đại học danh tiếng:

- [Bài thực hành Computer Network của Đại học Công nghệ Cáp Nhĩ Tân](https://github.com/rccoder/HIT-Computer-Network)
- [Bài tập lập trình và tài liệu, lời giải thực hành Wireshark cho 《Computer Network — Phương pháp top-down (bản gốc, phiên bản 6)》](https://github.com/moranzcw/Computer-Networking-A-Top-Down-Approach-NOTES)
- [Project cuối kỳ Computer Network, phòng chat viết bằng Python](https://github.com/KevinWang15/network-pj-chatroom)
- [Khóa học Computer Network của CMU](https://computer-networks.github.io/sp19/lectures.html)

Biết rằng nhiều bạn thích vừa xem video vừa học, nên mình giới thiệu thêm vài video giảng về Computer Network rất chất lượng.

**1. [Khóa học Computer Network của Đại học Công nghệ Cáp Nhĩ Tân](http://www.icourse163.org/course/HIT-154005)**: Khóa học cấp quốc gia, đến nay đã được mở 10 lần. Khóa học nhận được đánh giá rất cao, nên đặc biệt khuyến nghị bạn xem!

![](https://oss.javaguide.cn/github/javaguide/booksimage-20201218141241911.png)

**2. [Computer Network của Wangdao dành cho kỳ thi cao học](https://www.bilibili.com/video/BV19E411D78Q?from=search&seid=17198507506906312317)**: Rất phù hợp với các bạn học Computer Science chuẩn bị thi cao học! Video này hiện đã nhận hơn 16 nghìn lượt thích trên Bilibili.

![](https://oss.javaguide.cn/github/javaguide/booksimage-20201218141652837.png)

## Algorithms

Trước tiên là ba cuốn sách nhập môn. Bất kỳ cuốn nào trong ba cuốn này cũng đều rất phù hợp để bắt đầu học.

1. [《Cuốn sách Algorithms đầu tiên của tôi》](https://book.douban.com/subject/30357170/)
2. [《Algorithms qua hình ảnh》](https://book.douban.com/subject/26979890/)
3. [《Aha! Algorithms》](https://book.douban.com/subject/25894685/)

![](https://oss.javaguide.cn/java-guide-blog/image-20210327104418851.png)

Cá nhân mình nghiêng về **[《Cuốn sách Algorithms đầu tiên của tôi》](https://book.douban.com/subject/30357170/)** hơn. Dù điểm Douban của sách hơi thấp hơn hai cuốn còn lại, mình cho rằng hình minh họa và phần giải thích của sách là tốt nhất trong ba cuốn; vấn đề rõ ràng duy nhất là không có ví dụ code. Tuy nhiên, mình không nghĩ điều đó ảnh hưởng đến việc đây là một cuốn sách Algorithms hay. Mục tiêu của ba cuốn nhập môn này vốn không phải dùng code để giúp bạn giỏi Algorithms đến đâu, mà là làm một cuốn sách nhập môn tốt để đưa bạn bước vào thế giới học Algorithms.

Tiếp theo là một số cuốn sách Algorithms kinh điển hơn.

**[《Algorithms》](https://book.douban.com/subject/19952400/)**

![](https://oss.javaguide.cn/github/javaguide/booksimage-20220409123422140.png)

Nội dung cuốn sách rất rõ ràng, dễ hiểu, phù hợp với người mới bắt đầu về cấu trúc dữ liệu và Algorithms. Sách giới thiệu các cấu trúc dữ liệu và Algorithms thường dùng!

Mình từng được một giảng viên đặc biệt giới thiệu cuốn này khi học năm hai đại học! Khi đó mình cũng mua một cuốn để ở ký túc xá, đến lúc tốt nghiệp thì đọc được hơn một nửa. Vì nội dung thực sự quá nhiều! Ngoài ra, sách còn cung cấp code Java chi tiết, rất phù hợp với những bạn học Java; có thể nói đây là một trong những cuốn sách bắt buộc phải có của lập trình viên Java.

> **Những cuốn sách dưới đây đều là kinh điển của kinh điển, nhưng cũng khá khó đọc; không cần giải thích nhiều, cứ gọi là sách thần thánh là được!**
>
> **Nếu chỉ chuẩn bị cho phỏng vấn Algorithms, bạn không nên đọc những cuốn sách dưới đây.**

**[《Programming Pearls》](https://book.douban.com/subject/3227098/)**

![](https://oss.javaguide.cn/github/javaguide/booksimage-20220409145334093.png)

Một tác phẩm kinh điển được các cao thủ Algorithms từng giành hạng nhất, hạng nhì ACM đặc biệt khuyến nghị. Tác giả cuốn sách cũng rất xuất sắc; James Gosling, cha đẻ của Java, từng là học trò của ông.

Nhiều người nói cuốn sách này không dạy bạn Algorithms cụ thể, mà dạy một cách tư duy khi lập trình. Cách tư duy này không chỉ áp dụng trong lập trình mà còn phù hợp với các lĩnh vực khác.

**[《The Algorithm Design Manual》](https://book.douban.com/subject/4048566/)**

![](https://oss.javaguide.cn/github/javaguide/booksimage-20220409145411049.png)

Đây là cuốn sách Algorithms được dự án tự học Computer Science nổi tiếng trên GitHub [Teach Yourself Computer Science](https://link.zhihu.com/?target=https%3A//teachyourselfcs.com/) đặc biệt khuyến nghị.

Một số sách thần thánh tương tự còn có [《Introduction to Algorithms》](https://book.douban.com/subject/20432061/) và [《The Art of Computer Programming (Volume 1)》](https://book.douban.com/subject/1130500/) .

**Nếu muốn chuẩn bị cho phỏng vấn, những cuốn sách dưới đây có thể sẽ hữu ích!**

**[《Sword Offer》](https://book.douban.com/subject/6966465/)**

![](https://oss.javaguide.cn/github/javaguide/booksimage-20220409145506482.png)

Cuốn sách phỏng vấn này bao quát nhiều câu hỏi phỏng vấn Algorithms kinh điển. Nếu chuẩn bị phỏng vấn vào các công ty lớn, nhất định không nên bỏ qua.

Phần phân tích các bài toán lập trình Algorithms tương ứng với 《Sword Offer》: [CodingInterviews](https://link.zhihu.com/?target=https%3A//github.com/gatieme/CodingInterviews) .

**[《Cẩm nang phỏng vấn code dành cho lập trình viên (bản 2)》](https://book.douban.com/subject/30422021/)**

![](https://oss.javaguide.cn/github/javaguide/booksimage-20220409145622758.png)

Phần lớn bài toán trong 《Cẩm nang phỏng vấn code dành cho lập trình viên (bản 2)》 khó hơn nhiều so với 《Sword Offer》, phạm vi câu hỏi cũng toàn diện hơn 《Sword Offer》. Toàn bộ sách có gần 300 câu hỏi phỏng vấn code kinh điển từng xuất hiện trong thực tế.

Về video, mình khuyến nghị khóa học cấp quốc gia của Đại học Bắc Kinh — **[《Lập trình và Algorithms (2): Cơ sở Algorithms》](https://www.icourse163.org/course/PKU-1001894005)**; nội dung được giảng rất tốt!

![](https://oss.javaguide.cn/github/javaguide/books/22ce4a17dc0c40f6a3e0d58002261b7a.png)

Khóa học giới thiệu bảy Algorithms phổ quát cơ bản: liệt kê, tìm kiếm nhị phân, đệ quy, chia để trị, dynamic programming, tìm kiếm và tham lam. Việc giải quyết nhiều bài toán Algorithms phức tạp đều có thể cần đến các tư tưởng cơ bản này. Ngoài ra, một số bài mẫu trong khóa học có độ khó tương đương các bài trung bình trong kỳ thi lập trình sinh viên quốc tế ACM. Nếu có thể giải quyết những bài này, năng lực Algorithms của bạn sẽ vượt qua phần lớn sinh viên Computer Science bậc cử nhân.

## Data Structures

Thực ra, nhiều cuốn sách Algorithms được đề cập ở trên (ví dụ **《Algorithms》** và **《Introduction to Algorithms》**) đã giới thiệu chi tiết các cấu trúc dữ liệu thường dùng.

Ở đây mình bổ sung thêm một số sách cơ bản liên quan đến cấu trúc dữ liệu.

**[《Nói chuyện về Data Structures》](https://book.douban.com/subject/6424904/)**

![](https://oss.javaguide.cn/github/javaguide/booksimage-20220409145803440.png)

Đây là sách nhập môn, cách viết khá dễ hiểu, phù hợp với những bạn chưa có nền tảng về cấu trúc dữ liệu hoặc chưa học tốt cấu trúc dữ liệu.

**[《Phân tích Data Structures và Algorithms: Mô tả bằng ngôn ngữ Java》](https://book.douban.com/subject/3351237/)**

![](https://oss.javaguide.cn/github/javaguide/booksimage-20220409145823973.png)

Chất lượng rất cao, giới thiệu các cấu trúc dữ liệu và Algorithms thường dùng.

Các sách tương tự còn có **[《Phân tích Data Structures và Algorithms: Mô tả bằng ngôn ngữ C》](https://book.douban.com/subject/1139426/)** và **[《Phân tích Data Structures và Algorithms: Mô tả bằng C++》](https://book.douban.com/subject/1971825/)**

![](https://oss.javaguide.cn/github/javaguide/books/d9c450ccc5224a5fba77f4fa937f7b9c.png)

Về video, bạn nên xem khóa học cấp quốc gia **[《Data Structures》](https://www.icourse163.org/course/ZJU-93001#/info)** của Đại học Chiết Giang.

Phần Data Structures do cô Lão Lão giảng rất tuyệt! Tuy nhiên, khóa học vẫn có một số phần khó, đặc biệt là bài tập sau mỗi chương.

## Các môn nền tảng Computer Science

Toán và tiếng Anh là các môn đại cương, thường có thể học xong trong hai năm nhất và hai năm hai đại học; từ năm hai đến năm ba mới dần tiếp xúc với các môn chuyên ngành. Là những môn đầu tiên mà nhiều học sinh trung học học khi vào đại học, các môn đại cương là bước chuyển tiếp từ bậc trung học sang bậc cử nhân. Xét về tầm quan trọng đối với nghề nghiệp, chúng không quan trọng bằng các môn chuyên ngành, nhưng lại có vị trí rất quan trọng trong kế hoạch học tập và sinh hoạt ở bậc cử nhân. Do số lượng môn đại cương nhiều và số tín chỉ lớn, chúng chiếm phần lớn GPA ở bậc cử nhân, ảnh hưởng đến thứ hạng chuyên ngành trong hai năm đầu, đồng thời ảnh hưởng đến việc phân bổ suất miễn thi cao học sau khi kết thúc năm ba, tức cơ hội học cao học. Xét từ góc độ học tiếp, với những bạn muốn học cao học hoặc tiến sĩ, hai môn nền tảng là Toán và tiếng Anh vẫn rất hữu ích.

### Toán

#### Calculus (Toán cao cấp)

Calculus, hay còn gọi là toán cao cấp, là nỗi đau trong lòng vô số tân sinh viên năm nhất. May mắn là việc kiểm tra ở đại học không quá khắt khe; muốn đạt điểm cao cuối kỳ cũng không đến mức phải cày bài dữ dội như thời trung học. Tầm quan trọng của Calculus đối với sinh viên Computer Science chủ yếu thể hiện ở phép biến đổi hàm trong Computer Graphics, Algorithms gradient trong Machine Learning và các lĩnh vực như Signal Processing.

Hệ thống kiến thức của Calculus gồm hai phần là vi phân và tích phân. Thông thường sẽ học vi phân trước rồi đến tích phân; một số trường chia toán cao cấp thành hai học kỳ. Vi phân là phiên bản nâng cao của đạo hàm ở trung học, khá thân thiện với tân sinh viên năm nhất. Tích phân vừa hay là phép toán ngược của vi phân, nhưng tư duy này khá mới với tân sinh viên năm nhất nên có thể chưa tiếp thu được ngay. Tuy nhiên, môn học này được giảng dạy ở tất cả các trường đại học; phần lớn trường danh tiếng đều có khóa học trực tuyến đi kèm và giáo trình cũng được biên soạn rất tốt. Kết hợp khóa học trực tuyến với giáo trình để học kỹ, bạn nhất định sẽ không bỏ sót môn này.

Về sách, mình khuyến nghị 《The Princeton Companion to Calculus》. Sách giải thích chi tiết những nội dung như nền tảng Calculus, giới hạn, tính liên tục, vi phân, ứng dụng của đạo hàm, tích phân, chuỗi vô hạn, chuỗi Taylor và chuỗi lũy thừa.

![](https://oss.javaguide.cn/github/javaguide/booksimage-20220409155056751.png)

#### Linear Algebra (Đại số tuyến tính)

Cách tư duy của Linear Algebra phức tạp hơn một chút. Môn học định nghĩa một thế giới toán học hoàn toàn mới; mọi ký hiệu và định lý đều mới. Cách duy nhất có thể thử để hiểu có lẽ là dùng hình học để hiểu Linear Algebra. Vì Linear Algebra có mối liên hệ không thể tách rời với hình học, chẳng hạn cơ sở lý thuyết của biến đổi không gian chính là Linear Algebra, nên trên mạng có rất nhiều tài nguyên “học Linear Algebra bằng trực quan hóa”, giúp hiểu ý nghĩa của Linear Algebra và ghi nhớ công thức.

![](https://oss.javaguide.cn/github/javaguide/booksimage-20220409153940473.png)

Về sách, mình khuyến nghị **[《Hướng dẫn học Linear Algebra》](https://book.douban.com/subject/26390093/)** của thầy Lý Thượng Chí thuộc Đại học Khoa học và Công nghệ Trung Quốc.

![](https://oss.javaguide.cn/github/javaguide/booksimage-20220409155325251.png)

#### Probability Theory and Mathematical Statistics

Với các bạn học Computer Science, Probability Theory có thể hữu ích hơn Mathematical Statistics. Một số trường chỉ mở môn Probability Theory, một số trường cũng dạy Mathematical Statistics nhưng chỉ ở mức sơ lược. Lộ trình học Probability Theory tương tự Calculus: từng công thức đi kèm ví dụ, không trừu tượng như Linear Algebra và gần gũi với đời sống hơn. Trong bối cảnh việc làm hiện nay, sinh viên chuyên ngành Probability Theory and Mathematical Statistics có lẽ là những người dễ tìm việc nhất trong nhóm ngành Toán; họ thường làm công việc phân tích dữ liệu sau khi đi làm. Vì vậy, **môn học này thực sự là môn học tiên quyết quan trọng của Data Analysis, và tầm quan trọng của Probability Theory trong Machine Learning cũng là điều không cần bàn cãi.**

Về sách, mình khuyến nghị **[《Giáo trình Probability Theory and Mathematical Statistics》](https://book.douban.com/subject/34897672/)**. Sách gồm tám chương; bốn chương đầu về Probability Theory, chủ yếu trình bày các phân phối xác suất và tính chất của chúng; bốn chương sau về Mathematical Statistics, chủ yếu trình bày các phương pháp ước lượng tham số và kiểm định giả thuyết.

![](https://oss.javaguide.cn/github/javaguide/booksimage-20220409155738505.png)

#### Discrete Mathematics (Set Theory, Graph Theory, Modern Algebra, v.v.)

Discrete Mathematics là môn Toán dành riêng cho Computer Science, nhưng trên thực tế, với những bạn tốt nghiệp cử nhân rồi đi làm, Discrete Mathematics vẫn chưa phát huy được vai trò rất lớn của nó. Vai trò của Discrete Mathematics chủ yếu nằm trong các lĩnh vực như nghiên cứu graph; tính lý thuyết rất cao, nên những bạn muốn học cao học cần nắm thật vững.

### Tiếng Anh

Tiếng Anh có thể xem là một kỹ năng khá linh hoạt ở đại học. Có người nói “tiếng Anh càng giỏi thì càng có lợi cho sự phát triển cá nhân”; điều này không sai, nhưng với một số bạn có mục tiêu phát triển rõ ràng, kỹ năng tiếng Anh có thể không nằm trong danh sách kỹ năng của họ. Những điều tiếp theo chỉ dành cho các bạn học Computer Science.

Ở bậc cử nhân, môn tiếng Anh thường chỉ được mở trong hai năm đầu. Bạn nên nhớ rằng **nếu muốn nâng cao trình độ tiếng Anh chỉ bằng các tiết học tiếng Anh, hãy từ bỏ ý định đó.** Trình độ tiếng Anh được cải thiện hoàn toàn nhờ tích lũy, luyện tập hằng ngày và làm bài có mục tiêu.

**Nhất định phải đỗ kỳ thi tiếng Anh CET-4 và CET-6.** Đây là kỹ năng bắt buộc; phần lớn vị trí việc làm đều xem trình độ CET-4 và CET-6, ít nhất phải vượt qua. CET-4 chỉ khó hơn tiếng Anh trung học một chút; nhiều bạn có thể mắc ở CET-6, nên cần luyện tập có mục tiêu vì trong thời gian đại học có quá ít cơ hội tiếp xúc với tiếng Anh. Một tiết tiếng Anh mỗi học kỳ không đủ để duy trì trình độ tiếng Anh. Với những bạn đến từ vùng xa và có nền tảng tiếng Anh trung học yếu, thi CET-4 và CET-6 sẽ càng khó hơn. Bạn nên tập trung luyện đề thi các năm trước trước kỳ thi, kết hợp học thuộc các từ vựng tần suất cao. Để vượt qua CET-4 và CET-6 chỉ cần 425 điểm, mức điểm này khá dễ đạt. Những bạn khá hơn có thể thử đạt 500 điểm; nếu đạt 600 điểm thì trình độ rất tốt, và đó sẽ là một điểm sáng trong CV.

Kỳ thi IELTS và TOEFL chỉ dành cho những bạn muốn đi du học hoặc ứng tuyển vào vị trí có yêu cầu đặc biệt về tiếng Anh. Không dễ vượt qua IELTS và TOEFL nếu thi mà không chuẩn bị; bỏ tiền tham gia một lớp học thêm ngoài trường đáng tin cậy có thể là lựa chọn tốt hơn.

Với các bạn học Computer Science, năng lực tiếng Anh vẫn khá quan trọng. Dù khi ứng tuyển, việc không có điểm IELTS hoặc TOEFL thường không phải lý do loại hồ sơ, nhưng ít nhất bạn cần có khả năng:

- **Sử dụng thành thạo software, system có giao diện tiếng Anh**
- **Đọc các blog và solution cho bug trên Internet mà không gặp trở ngại**
- **Đọc tài liệu nghiên cứu bằng tiếng Anh thành thạo**
- **Có khả năng viết paper bằng tiếng Anh ở mức nhất định**

Suy cho cùng, ngôn ngữ máy tính là ngôn ngữ ký tự; trong bốn kỹ năng nghe, nói, đọc, viết, yêu cầu tối thiểu phải đáp ứng **đọc và viết** thì đâu có quá đáng, đúng không?

### Compiler Principles

So với các môn chuyên ngành đã giới thiệu ở trên, Compiler Principles có vẻ không quan trọng bằng. Tầm quan trọng của Compiler Principles chủ yếu thể hiện ở:

- Phát triển ngôn ngữ tầng dưới, engine hoặc ngôn ngữ cấp cao như MySQL, Java
- Phát triển Operating System hoặc embedded system
- Tư duy về lexical analysis, syntax, semantics và automata

**Môn học tiên quyết quan trọng của Compiler Principles là Formal Languages and Automata. Tư duy automata được ứng dụng quan trọng trong lexical analysis; sau khi học môn này, bạn sẽ nhận ra sự hữu ích kỳ diệu của Algorithms automata trong nhiều trường hợp.**

Nhìn chung, môn học này tương đối không quan trọng đối với sự phát triển nghề nghiệp của lập trình viên, nhưng xét về độ khó, học môn này có thể củng cố khá tốt tư duy lập trình. Về tài nguyên học tập, ngoài slide trên lớp, bạn có thể dùng cuốn 《Compiler Principles》 làm sách tham khảo để hỗ trợ những phần khó tự học (đây là “Dragon Book” mà mọi người thường nhắc đến; muốn đọc hết vẫn cần khá nhiều nỗ lực).

![](https://oss.javaguide.cn/github/javaguide/books/20210406152148373.png)

Một số sách khác:

- **[《Compiler Principles hiện đại》](https://book.douban.com/subject/30191414/)**: Sách nhập môn Compiler Principles.
- **[《Thiết kế compiler》](https://book.douban.com/subject/20436488/)**: Bao quát toàn bộ chủ đề của compiler từ frontend đến backend.

Những cuốn sách mình giới thiệu ở trên vẫn khá khó, thực sự rất khó kiên trì đọc hết. Ở đây đặc biệt khuyến nghị [khóa học video Compiler Principles của Đại học Công nghệ Cáp Nhĩ Tân](https://www.icourse163.org/course/HIT-1002123007), thực sự rất hay và cũng là khóa học cấp quốc gia; quan trọng hơn là do một nữ giảng viên xinh đẹp, dịu dàng giảng dạy!

![](https://oss.javaguide.cn/github/javaguide/books/20210406152847824.png)
