---
title: Chiến lược phát triển kỹ thuật của lập trình viên
description: "Chiến lược phát triển kỹ thuật của lập trình viên: hệ thống hóa các khái niệm then chốt, vấn đề thường gặp và điểm thực hành xoay quanh kiến thức kỹ thuật và tổng hợp phỏng vấn, giúp bạn học tập hiệu quả và chuẩn bị phỏng vấn."
category: Tuyển tập bài viết kỹ thuật
author: Bobo Micro-course
tag:
  - Chiến lược nâng cấp
head:
  - - meta
    - name: keywords
      content: chiến lược phát triển kỹ thuật,phát triển lập trình viên,kim tự tháp học tập,luyện tập có chủ đích,chuyên gia kỹ thuật,hoạch định nghề nghiệp,hoạch định mười năm,liên tục tạo ra giá trị
---

> **Lời giới thiệu**: Đây là một bài viết rất hay của thầy Bobo, không chỉ hữu ích cho việc phát triển kỹ thuật mà còn áp dụng được trong các lĩnh vực khác! Nên đọc đi đọc lại để hình thành chiến lược phát triển kỹ thuật của riêng bạn.
>
> **Địa chỉ bài gốc:** <https://mp.weixin.qq.com/s/YrN8T67s801-MRo01lCHXA>

## 1. Mở đầu

Trong nhóm trao đổi kỹ thuật trên WeChat của Bobo, học viên thường hỏi về cách người làm kỹ thuật nên học tập và phát triển thế nào. Dù chỉ trao đổi qua WeChat, tôi vẫn có thể cảm nhận được sự lo lắng của các bạn.

**Vì sao người làm kỹ thuật lại lo lắng?** Nói thẳng ra, đó là vì thiếu can đảm và tầm nhìn hạn hẹp. "Can đảm" là lòng dũng cảm; người lo lắng thường sợ sự bất định của tương lai. "Nhận thức" là hiểu biết; người lo lắng thường không nhìn rõ thế giới xung quanh, cũng không nhìn rõ bản thân và con đường phù hợp với mình. "Tầm nhìn" cũng có thể gọi là chí hướng; người dễ lo lắng thường có tầm nhìn hẹp và chí hướng nhỏ. Nhìn từ góc độ strategy và management, đó là do nhận thức chưa đủ về bản thân và thế giới xung quanh, chưa có một chiến lược học tập và phát triển rõ ràng, dài hạn, cũng chưa có kế hoạch mục tiêu theo từng giai đoạn có thể thực thi cùng sự thực hiện nghiêm túc.

Vì có quá nhiều học viên hỏi những vấn đề tương tự nên tôi bắt đầu thấy hơi phiền. Để tránh phải trả lời lặp lại, tôi đã chuyên tâm tổng hợp và sắp xếp bài viết dài này, cố gắng trả lời thống nhất các loại câu hỏi trên. Nếu sau này vẫn có học viên hỏi tương tự, tôi sẽ hướng dẫn họ đọc bài viết này, rồi dành ba tháng, một năm, thậm chí lâu hơn để suy nghĩ và trả lời một câu hỏi: **Chiến lược phát triển kỹ thuật của bạn rốt cuộc là gì?** Nếu đã suy nghĩ rõ ràng và có câu trả lời rõ ràng, khả thi, xin chúc mừng: bạn chỉ cần thực hiện từng bước, hoàn toàn không cần lo lắng. Việc đạt được mục tiêu chiến lược và tạo ra thành tựu chỉ còn là vấn đề thời gian; nếu không, bạn vẫn cần không ngừng rèn luyện và suy nghĩ, nhất định phải làm rõ vấn đề lớn của cuộc đời này!

Sau đây, hãy cùng xem các chuyên gia kỹ thuật hàng đầu trong ngành đã làm như thế nào.

## 2. Học chiến lược phát triển từ các chuyên gia kỹ thuật hàng đầu

Chúng ta biết software design có Design Pattern, thực ra sự phát triển của người làm kỹ thuật cũng có Growth Pattern. Bobo thường xem quá trình phát triển của các chuyên gia kỹ thuật hàng đầu trên Linkedin, tìm hiểu Growth Pattern trong đó, từ đó gợi mở việc xây dựng chiến lược phát triển kỹ thuật của riêng mình.

Đương nhiên, rất ít chuyên gia kỹ thuật hàng đầu nói rõ cho bạn chiến lược phát triển kỹ thuật của họ và kế hoạch triển khai chi tiết cho từng năm. Nhưng điều đó không cản trở chúng ta truy nguyên chiến lược phát triển kỹ thuật của họ qua quá trình đã trải qua và kết quả tạo ra. Trên thực tế, **người làm kỹ thuật càng giỏi thì chiến lược và con đường phát triển kỹ thuật của họ càng rõ ràng, chúng ta càng dễ tìm ra một số mô hình thành công từ đó.**

### 2.1 Trường hợp chuyên gia system performance

Phần lớn developer trong nước rất thích tối ưu system performance, có người thậm chí không thể nói ba câu mà không nhắc đến high performance/high concurrency. Tuy nhiên, số người thực sự đi sâu vào lĩnh vực này và đạt trình độ chuyên gia lại rất ít.

Chuyên gia kỹ thuật hàng đầu mà tôi đặc biệt muốn giới thiệu là **Brendan Gregg**. Ông là tác giả cuốn sách kinh điển trong lĩnh vực system performance là 《System Performance: Enterprise and the Cloud》 (bản tiếng Trung [《Đỉnh cao hiệu năng: Thấu hiểu hệ thống, doanh nghiệp và cloud computing》](https://www.amazon.cn/dp/B08GC261P9)), đồng thời là tác giả của [Flame Graph, công cụ phân tích performance nổi tiếng](https://github.com/brendangregg/FlameGraph).

Trước đây Brendan Gregg là performance architect cấp cao của Netflix, làm việc tại Netflix gần 7 năm. Tháng 4 năm 2022, ông rời Netflix để đến Intel, đảm nhiệm vị trí Fellow.

![](https://oss.javaguide.cn/github/javaguide/high-quality-technical-articles/cdb11ce2f1c3a69fd19e922a7f5f59bf.png)

Nhìn chung, ông đã đào sâu trong lĩnh vực system performance hơn 10 năm. [Quá trình làm việc trước đây của Brendan Gregg](https://www.linkedin.com/in/brendangregg/) có thể xem trên Linkedin. Trong 10 năm đó, ngoài sách, Brendan Gregg còn tạo ra hơn một trăm tài liệu kỹ thuật liên quan đến system performance, video/PPT thuyết trình cùng nhiều công cụ và phần mềm khác nhau. Tất cả nội dung liên quan đều được chia sẻ rất ngăn nắp trên [technical blog của ông](http://www.brendangregg.com/). Có thể nói ông là một chuyên gia kỹ thuật hàng đầu có năng suất rất cao.

![Công cụ performance](https://oss.javaguide.cn/github/javaguide/high-quality-technical-articles/format,png-20230309231802218.png)

Hình trên đến từ cuốn sách mới 《BPF Performance Tools: Linux System and Application Observability》 của Brendan Gregg. Có thể thấy Brendan Gregg đã đào sâu mức độ nắm vững lĩnh vực system performance đến mọi ngóc ngách của hardware, operating system và application. Có thể nói ông bao quát 360 độ, không có điểm mù; toàn bộ computer system gần như trong suốt đối với ông. Theo Bobo, Brendan Gregg thực sự là một nhân vật tầm cỡ thế giới, chuyên gia hàng đầu của lĩnh vực system performance.

### 2.2 Trường hợp từ open source đến enterprise

Chuyên gia kỹ thuật hàng đầu thứ hai tôi muốn chia sẻ là **Jay Kreps**. Ông là founder/architect của Kafka, message middleware open source nổi tiếng, đồng thời là co-founder và CEO của Confluent. Confluent là một công ty kỹ thuật phát triển sản phẩm và dịch vụ enterprise xoay quanh Kafka.

Từ [quá trình làm việc của Jay Kreps trên Linkedin](https://www.linkedin.com/in/jaykreps/), có thể thấy trước đây Jay Kreps đã làm việc tại Linkedin hơn 7 năm (2007.6 ~ 2014. 9), từ senior engineer, engineering manager cho đến principal staff engineer. Kafka đại khái được Jay Kreps khởi xướng tại Linkedin vào năm 2010 dưới dạng một project, nhằm giải quyết vấn đề thu thập, lưu trữ và tiêu thụ dữ liệu lớn trong nội bộ Linkedin. Sau đó, ông và team của mình luôn tập trung hoàn thiện Kafka, open source (đầu năm 2011) và xây dựng hệ sinh thái cộng đồng.

Đến cuối năm 2014, Kafka đã rất thành công trong cộng đồng và có một nhóm user tương đối lớn. Vì vậy, Jay Kreps cùng một số tác giả ban đầu rời Linkedin, thành lập [Confluent](https://tech.163.com/14/1107/18/AAFG92LD00094ODU.html), bắt đầu con đường cung cấp dịch vụ enterprise cho Kafka và các sản phẩm xung quanh. Tháng 4 năm 2020, Confluent đã nhận được khoản đầu tư Series E trị giá 250 triệu USD, định giá công ty đạt 4,5 tỷ USD. Từ khi Kafka ra đời đến nay, Jay Kreps đã đầu tư trọn vẹn khoảng 10 năm vào sản phẩm và công ty này.

![Bộ ba founder của Confluent](https://oss.javaguide.cn/github/javaguide/high-quality-technical-articles/format,png-20230309231805796.png)

Hình trên là bộ ba founder của Confluent, một tổ hợp rất thú vị: một người Trung Quốc (bên trái), một người Ấn Độ (bên phải), còn Jay Kreps ở giữa là người Mỹ.

Lý do tôi có ấn tượng đặc biệt sâu sắc với Kafka và Jay Kreps là vì nửa cuối năm 2012, tôi cũng chuyên làm việc về thu thập dữ liệu lớn tại bộ phận framework của Ctrip, và từng phát triển một sản phẩm Log Collector + Agent có chức năng tương tự Kafka. Tôi nhớ cùng thời điểm đó có không dưới 4 sản phẩm open source cùng loại: Facebook Scribe, Apache Chukwa, Apache Flume và Apache Kafka. Nhìn lại hiện nay, chỉ Kafka phát triển tốt nhất đến hiện tại. Điều này không thể tách rời sự tập trung và đầu tư liên tục của founder, tất nhiên cũng không thể tách rời tầm nhìn kỹ thuật lớn của một số founder.

Khi đó, tôi gần như chưa có khái niệm về strategic thinking, vẫn đang ở giai đoạn **muốn học mọi công nghệ, cho rằng làm càng nhiều project thì càng giỏi**. Sau nửa năm làm thu thập dữ liệu, tôi quay sang làm những project "thú vị hơn" khác (nhìn từ khía cạnh này cũng có thể thấy tầm nhìn kỹ thuật của tôi khi ấy rất hạn hẹp). Trong thời gian đó, tôi lần lượt theo dõi một số động thái khởi nghiệp của Jay, nhưng không ngờ ông có thể phát triển Confluent đến quy mô hiện tại. Bây giờ nhìn lại, thực ra từ mười năm trước Jay Kreps đã có suy nghĩ chiến lược tương đối rõ ràng về sự phát triển kỹ thuật của mình, đồng thời có tầm nhìn kỹ thuật lớn và một số tố chất cần thiết để tạo thành công. Jay Kreps và Kafka đã cho tôi một bài học sinh động về technical strategy và thực tiễn.

### 2.3 Trường hợp technical media influencer

Đến đây, một số bạn có thể phản bác: Bobo, những chuyên gia hàng đầu mà bạn nói đều có nền tảng học vấn tốt, nền tảng vững chắc và xuất phát điểm cao, nên họ mới thành công hơn. Thực ra không phải vậy. Sau đây tôi muốn giới thiệu một technical media influencer tên là Brad Traversy. Bạn có thể xem [hồ sơ Linkedin của ông](https://www.linkedin.com/in/bradtraversy/). Background của ông khá bình thường, trình độ học vấn gần như chỉ là một community college không chính quy (tương đương cao đẳng), không có kinh nghiệm làm việc tại các công ty lớn chính quy, và một số công việc ít ỏi đều là làm website thuê ngoài.

![](https://oss.javaguide.cn/github/javaguide/high-quality-technical-articles/30d6d67dc6dd5f9251f2f01af4de53fc.png)

Nhưng!!! Hiện nay Brad Traversy là một influencer lớn trong lĩnh vực technical media. [Kênh của ông trên Youtube](https://www.youtube.com/c/TraversyMedia) hiện có hơn 1,38 triệu subscriber, trong 10 năm đã tạo ra hơn 800 video hướng dẫn liên quan đến Web development và programming. Brad Traversy cũng là một giảng viên thành công trên [Udemy](https://www.udemy.com/user/brad-traversy/), hiện đã tạo tổng cộng 19 khóa học trên Udemy, số học viên mua khóa học gần 420 nghìn.

![](https://oss.javaguide.cn/github/javaguide/high-quality-technical-articles/160b0bc4f689413757b9b5e2448f940b.png)

Hiện nay Brad Traversy là freelancer, thu nhập từ quảng cáo Youtube và các khóa học Udemy khá tốt.

Là một technical media influencer như vậy, thật khó tưởng tượng thời trẻ ông từng mang những nhãn mác: thiếu niên hư, nghiện rượu, hút thuốc, dùng ma túy, xăm mình, vào tù...

Mãi đến khi người con đầu tiên ra đời sau khi kết hôn, ông mới bắt đầu gánh vác trách nhiệm và thay đổi. Sau đó, nhờ niềm đam mê mãnh liệt với kỹ thuật, ông bắt đầu liên tục tạo ra các khóa học miễn phí trên nền tảng Youtube. Từ đó, ông tìm thấy mục tiêu chiến lược phù hợp với mình, rồi cuộc đời bắt đầu có nhiều thay đổi tích cực... Nếu bạn quan tâm đến quá khứ của Brad Traversy, nên xem video tự sự [《My Struggles & Success》](https://www.youtube.com/watch?v=zA9krklwADI) của ông trên Youtube.

![My Struggles & Success](https://oss.javaguide.cn/github/javaguide/high-quality-technical-articles/format,png-20230309231830686.png)

Tôi đã xem sơ qua [toàn bộ video của Brad Traversy trên Youtube](https://www.youtube.com/c/TraversyMedia/videos). Tổng cộng 800+ video trong 10 năm, trung bình hơn 80 video mỗi năm. Video đầu tiên được đăng vào tháng 8 năm 2010. Trong vài năm đầu gần như không có subscriber; đến tháng 1 năm 2017, số subscriber mới đạt 50k, tức cách thời điểm bắt đầu gần 6 năm. Tháng 10 năm 2017, số subscriber tăng vọt lên 200k; tháng 3 năm 2018 đạt 300k. Đến tháng 1 năm 2021, số subscriber đạt 1,38 triệu. Có thể cho rằng từ năm 2017, tức sau khi tích lũy 6～7 năm, số subscriber của ông bắt đầu xuất hiện điểm ngoặt. **Nếu vẽ những dữ liệu này ra, đó sẽ là một đường cong lãi kép rất đẹp.**

### 2.4 Tổng kết các trường hợp

Brendan Gregg, Jay Kreps và Brad Traversy đi theo những con đường kỹ thuật khác nhau, nhưng thành công của họ có điểm chung, hay nói cách khác là có mô hình chung:

**1. Tìm được mục tiêu chiến lược dài hạn phù hợp với bản thân.**

- Brendan Gregg: trở thành chuyên gia hàng đầu trong lĩnh vực system performance
- Jay Kreps: xây dựng công ty cung cấp enterprise service dựa trên message queue open source Kafka và đưa công ty lên sàn
- Brad Traversy: trở thành influencer lớn và giảng viên khóa học trong lĩnh vực technical media, lấy đó làm nghề nghiệp của mình

**2. Tập trung đào sâu một lĩnh vực (hoặc một số ít lĩnh vực liên quan) (Niche), giữ vững định hướng, không tùy tiện chuyển lĩnh vực.**

- Brendan Gregg: lĩnh vực system performance
- Jay Kreps: message middleware/real-time computing + khởi nghiệp
- Brad Traversy: lĩnh vực technical media/giảng dạy, hướng Web development + programming language

**3. Đầu tư dài hạn, cả ba đều liên tục đầu tư trong 10 năm.**

**4. Kế hoạch chi tiết theo năm + tạo ra giá trị liên tục, có thể đo lường (Persistent & Measurable Value Output).**

- Brendan Gregg: ngoài output công việc hằng ngày tại công ty, mỗi năm tạo ra hơn 10 tài liệu kỹ thuật và video thuyết trình, trung bình tạo ra 2,5 open source tool. Trong 10 năm đã xuất bản tổng cộng 2 cuốn sách, trong đó 《System Performance》 đã được cập nhật đến phiên bản thứ hai.
- Jay Kreps: nhìn chung có open source product + output từ công ty, xuất bản 1 cuốn sách, mỗi năm phát hành một số phiên bản Kafka và các sản phẩm xung quanh.
- Brad Traversy: mỗi năm có video miễn phí trên Youtube (trung bình hơn 80 video) + khóa học video trả phí trên Udemy (trung bình 1,5 khóa học).

**5. Bắt đầu từ đích đến là một khác biệt lớn giữa người giỏi và người bình thường.**

Người bình thường thường đi đến đâu tính đến đó, ít khi hoạch định dài hạn. Người giỏi thường có mục tiêu lớn trước, sau đó dùng phương pháp suy ngược để cụ thể hóa mục tiêu lớn thành kế hoạch triển khai chi tiết theo năm/tháng/tuần. Brendan Gregg, Jay Kreps và Brad Traversy đều là những điển hình của việc bắt đầu từ đích đến.

![Bắt đầu từ đích đến](https://oss.javaguide.cn/github/javaguide/high-quality-technical-articles/format,png-20230309231833871.png)

Trên đây đã tổng kết Growth Pattern của một số chuyên gia kỹ thuật hàng đầu. Một trọng điểm là sự phát triển của những người này đều được thúc đẩy bởi **việc liên tục tạo ra output có giá trị (Persistent Valuable Output)**. Vì sao output liên tục lại quan trọng đến vậy? Điều này phải bắt đầu từ kim tự tháp học tập dưới đây.

## 3. Kim tự tháp học tập và luyện tập có chủ đích

![Kim tự tháp học tập](https://oss.javaguide.cn/github/javaguide/high-quality-technical-articles/format,png-20230309231836811.png)

Kim tự tháp học tập là kết quả nghiên cứu của National Training Laboratories ở bang Maine, Hoa Kỳ. Mô hình này cho rằng:

> 1. Sau khi nghe giảng trên lớp như thường lệ, tỷ lệ lưu giữ nội dung học tập trung bình chỉ khoảng 5%;
> 2. Tỷ lệ lưu giữ trung bình của việc đọc sách chỉ khoảng 10%;
> 3. Các khóa học kết hợp học tập với hiệu quả nghe nhìn có tỷ lệ lưu giữ trung bình khoảng 20%;
> 4. Tỷ lệ lưu giữ trung bình sau khi giáo viên trực tiếp làm thí nghiệm trình diễn khoảng 30%;
> 5. Tỷ lệ lưu giữ trung bình của thảo luận nhóm (đặc biệt sau tranh biện) có thể đạt 50%;
> 6. Sau khi thực sự áp dụng điều đã học vào thực tiễn, tỷ lệ lưu giữ trung bình có thể đạt 75%;
> 7. Trên cơ sở thực tiễn, hệ thống hóa điều đã học rồi truyền đạt lại cho người khác, tỷ lệ lưu giữ trung bình có thể đạt 90%.

Bảy phương pháp học tập liệt kê ở trên, bốn phương pháp đầu được gọi là **học tập thụ động**, ba phương pháp sau được gọi là **học tập chủ động**.

Hãy lấy việc học bơi làm phép so sánh: học tập thụ động tương đương với việc bạn xem người khác bơi, còn học tập chủ động là bạn phải tự xuống nước bơi. Chúng ta biết các hoạt động như bơi hoặc chạy cần đốt cháy calorie của cơ thể, từ đó mới đạt hiệu quả rèn luyện cơ thể và tăng cơ (cơ bắp là kết quả của việc đốt cháy calorie). Nếu bạn chỉ xem người khác bơi mà không thực sự tự bơi, cơ bắp sẽ không phát triển. Tương tự, học tập chủ động cũng cần đốt cháy calorie của não bộ, từ đó mới đạt hiệu quả rèn luyện não bộ và phát triển "cơ bắp" của não.

Chúng ta cũng biết việc đốt cháy calorie của cơ thể thường khiến người ta cảm thấy không thoải mái. Nếu đốt calorie của cơ thể khiến người ta thấy dễ chịu thì có lẽ trên thế giới đã không có những người béo. Tương tự, đốt calorie của não bộ cũng khiến người ta cảm thấy không thoải mái, căng thẳng, toát mồ hôi hoặc nói năng lộn xộn. Nếu đốt calorie của não bộ khiến người ta thấy dễ chịu thì có lẽ ai trên thế giới cũng thông minh và có thể phát huy tối đa tiềm năng. Tất nhiên, những cảm giác không thoải mái này chỉ là ngắn hạn; về lâu dài, chúng khiến bạn khỏe mạnh và thông minh hơn. Bobo luôn cho rằng **thể chất bẩm sinh giữa người với người thực ra không khác nhau nhiều, nhưng thể chất và năng lực sau này có sự khác biệt. Những khác biệt này phần lớn do chất lượng, tần suất và cường độ rèn luyện cơ thể cùng não bộ về sau tạo ra.**

Sau khi hiểu đạo lý này, người trưởng thành về tâm trí và có tính kỷ luật sẽ liên tục **luyện tập có chủ đích**. Việc luyện tập có chủ đích bao gồm rèn luyện cơ thể. Ví dụ, hiện nay Bobo kiên trì chạy 3 km, đi bộ 3 km mỗi ngày, mỗi ngày thực hiện 60 lần sit-up, plank 5 phút, v.v.; mỗi ngày duy trì để cơ thể đốt cháy một lượng calorie nhất định. Luyện tập có chủ đích cũng bao gồm rèn luyện não bộ. Ví dụ, hiện nay Bobo mỗi ngày làm project và viết code (rèn luyện não + tay), trung bình mỗi ngày tạo ra video miễn phí dài 10 phút trên Bilibili (rèn luyện não + khả năng diễn đạt bằng lời), định kỳ tổng kết và tạo ra bài viết trên tài khoản công chúng (rèn luyện não + khả năng diễn đạt bằng chữ), ngoài ra mỗi ngày chơi bóng thăng bằng khoảng nửa giờ (hình dưới) hoặc game Tomb Raider (rèn luyện tiểu não + tay), duy trì để não bộ đốt cháy một lượng calorie nhất định mỗi ngày và giữ cường độ nhất định (cảm giác không thoải mái vừa phải).

![Game bóng thăng bằng](https://oss.javaguide.cn/github/javaguide/high-quality-technical-articles/format,png-20230309231839985.png)

Về nguyên lý chuyên môn và methodology của luyện tập có chủ đích, nên đọc cuốn sách 《Luyện tập có chủ đích》.

![Luyện tập có chủ đích](https://oss.javaguide.cn/github/javaguide/high-quality-technical-articles/format,png-20230309231842735.png)

Hãy lưu ý: nếu thường ngày bạn chưa bao giờ tập tạ, đột nhiên tập tạ vào một ngày nào đó sẽ rất khó thích nghi, thậm chí bị thương. Rèn luyện não bộ cũng vậy. Nếu bạn chưa từng tạo video, lúc mới bắt đầu sẽ rất khó thích nghi và chất lượng video tạo ra sẽ rất kém. Nhưng không sao, mọi hình thức luyện tập đều là quá trình tiến dần từng bước và không ngừng củng cố. Sau khi "cơ bắp" ở các vùng não liên quan phát triển, bạn sẽ dần bước vào vòng tuần hoàn tích cực; về sau mọi việc ngày càng thuận lợi và các "cơ bắp" liên quan ngày càng phát triển. Vì vậy, cũng giống như tập thể hình, rèn luyện não bộ không thể gặp khó khăn là bỏ cuộc, mà cần luyện tập có chủ đích một cách tuần tự (Incremental) và liên tục (Persistent).

Sau khi hiểu kim tự tháp học tập và luyện tập có chủ đích, hãy nhìn lại cách làm của Brendan Gregg, Jay Kreps và Brad Traversy. Việc học tập và phát triển của họ đều được xây dựng trên nền tảng liên tục tạo ra output có giá trị. Những output đó là kết quả của luyện tập có chủ đích và đốt calorie của não bộ. Output của họ hoặc dựa trên thực tiễn, chẳng hạn open source project Kafka và công ty Confluent của Jay Kreps; hoặc trên cơ sở thực tiễn, hệ thống hóa rồi truyền đạt cho người khác, chẳng hạn PPT/video thuyết trình kỹ thuật và sách của Brendan Gregg, video hướng dẫn của Brad Traversy, v.v. Nói cách khác, họ luôn học tập chủ động và hiệu quả ở tầng 5～7 của kim tự tháp học tập. Hơn nữa, output học tập của họ còn được user sử dụng, có customer value; có user thì có feedback và measurement. Hãy nhớ rằng việc học có feedback và measurement còn được gọi là closed-loop learning, có thể không ngừng cải tiến và nâng cao; ngược lại, việc học không có feedback và measurement thì không thể cải tiến và nâng cao.

Bây giờ, bạn cũng nên hiểu rằng khoe danh sách sách hay khoe skill map là việc rất đơn giản, đọc sách hay tham gia khóa học cũng không khó. Nhưng yêu cầu bạn đưa ra chiến lược phát triển kỹ thuật tổng thể trong 5～10 năm, sau đó dựa trên chiến lược này đưa ra kế hoạch triển khai chi tiết từng năm (đặc biệt là kế hoạch output), rồi nghiêm túc thực hiện đúng kế hoạch, quả thực là việc rất khó. Điều này cần rất nhiều rèn luyện thực tiễn và suy nghĩ sâu sắc, cần đốt cháy rất nhiều calorie của não bộ! Nhưng đây là quy luật tiến hóa do tự nhiên đặt ra. Phát triển thành một chuyên gia kỹ thuật thực thụ cũng giống như trở thành một vận động viên hạng nhất, cần trao đổi bằng lượng calorie tương xứng thông qua việc đốt cháy. Phát triển thành một chuyên gia kỹ thuật thực thụ cũng cần trao đổi bằng giá trị xã hội tương xứng được tạo ra; chỉ như vậy xã hội mới có thể tiến hóa bình thường. Bạn thúc đẩy xã hội tiến hóa thì xã hội mới hồi đáp bạn. Nếu không như vậy, xã hội không thể tiến hóa bình thường.

## 4. Sự hình thành tư duy chiến lược

![Chu kỳ suy nghĩ và điểm cơ hội](https://oss.javaguide.cn/p3-juejin/dc87167f53b243d49f9f4e8c7fe530a1~tplv-k3u1fbpfcp-zoom-1.png)

Khi sinh viên mới tốt nghiệp bắt đầu làm việc tại doanh nghiệp, suy nghĩ thường được tính theo ngày/tuần/tháng. Phần lớn chỉ là hôm nay học công nghệ nào, ngày mai học ngôn ngữ nào, rất ít khi suy nghĩ về mục tiêu một năm hoặc lâu hơn. Đây là giai đoạn mơ hồ, trước mắt tối đen và không nhìn thấy đường; năng lực cũng như xác suất nắm bắt cơ hội đều rất thấp.

Sau ba năm làm việc, người có năng lực lĩnh hội tốt thường lấy một năm làm chu kỳ suy nghĩ, xây dựng và thực hiện một số kế hoạch hằng năm. Đây là giai đoạn tin vào thiên phú và so tài năng lực, có thể nắm bắt một số cơ hội nhỏ.

Sau năm năm làm việc, một số người có năng lực lĩnh hội tốt sẽ hình thành sự can đảm và tầm nhìn nhất định. Họ xây dựng và thực hiện kế hoạch theo chu kỳ 3～5 năm, bắt đầu chủ động bố trí để nắm bắt một số cơ hội quy mô vừa.

Sau mười năm làm việc, người có năng lực lĩnh hội cao sẽ nhìn thấy những thay đổi của mô hình và quy luật, chẳng hạn nhận ra mô hình phát triển của ngành và mô hình phát triển của nhân tài. Từ đó, tư duy chiến lược bắt đầu hình thành. Sau đó, họ xây dựng và thực hiện kế hoạch chiến lược của mình theo chu kỳ 5～10 năm, bắt đầu chủ động bố trí để nắm bắt một số cơ hội vừa và lớn. Brendan Gregg, Jay Kreps và Brad Traversy đều thuộc giai đoạn này.

Tất nhiên, cũng có một số rất ít tinh hoa của thời đại còn giỏi hơn, có thể nhìn thấu thời đại và bản tính con người. Họ suy nghĩ theo đơn vị cả đời hoặc lâu hơn. Những siêu nhân này không nằm trong phạm vi thảo luận của bài viết.

## 5. Đề xuất

**1. Xây dựng và hoạch định chiến lược theo chu kỳ 5～10 năm.**

Hiện nay, sinh viên đại học thường tốt nghiệp ở tuổi 22～23. Sau mười năm làm việc, tức khoảng 32～33 tuổi, bạn cũng đã quan sát suốt mười năm và lẽ ra phải có sự lĩnh hội tương đối sâu sắc về bản thân và thế giới xung quanh (ngành và lĩnh vực của bạn). **Nếu đến tuổi này mà bạn vẫn mơ hồ, hôm nay nắm thứ này, ngày mai nắm thứ khác, thì chỉ có thể nói rằng can đảm và tầm nhìn của bạn khá thấp.** Trong bối cảnh cạnh tranh khốc liệt của ngành IT hiện nay, việc bị cho thôi việc ở tuổi 35 có thể đã ở ngay trước mắt.

Khi đã có strategic thinking, bạn nên xây dựng và hoạch định chiến lược theo chu kỳ 5～10 năm. Lấy các chuyên gia hàng đầu như Brendan Gregg, Jay Kreps và Brad Traversy làm ví dụ, **nếu thực sự muốn tạo ra thành tựu trong đời, chu kỳ đầu tư thường phải là mười năm.** Từ năm 33 tuổi, bạn đại khái có 3 chu kỳ mười năm, vì sau 60 tuổi, người bình thường thường mắt đã mờ, khó làm được việc lớn. Nếu năng lực lĩnh hội kém hơn một chút và đến 40 tuổi mới bắt đầu hoạch định, bạn đại khái còn 2 chu kỳ mười năm. Nếu hoạch định tốt, 2～3 chu kỳ mười năm này có thể giúp bạn tạo dựng một sự nghiệp không nhỏ. Nếu không, rất có thể cả đời bạn không tạo dựng được sự nghiệp gì, hoặc luôn giúp người khác đạt được sự nghiệp của họ.

**2. Tập trung năng lượng vào việc của mình.**

Xét việc thời gian trong đời có thể làm nên sự nghiệp chỉ là 2～3 chu kỳ mười năm, bạn sẽ nhận ra cuộc đời thực ra rất ngắn. Khi đó, bạn sẽ dồn toàn bộ năng lượng vào việc thực hiện chiến lược mười năm, không còn thời gian lãng phí vào những cuộc trò chuyện phiếm hay tranh luận vô ích trên mạng.

**3. Kế hoạch triển khai chi tiết, đặc biệt là kế hoạch output.**

Sau khi có định hướng chiến lược mười năm, bước tiếp theo là kế hoạch triển khai chi tiết từng năm, đặc biệt là kế hoạch output. Những kế hoạch này chủ yếu nên hoạt động ở tầng 5/6/7 của kim tự tháp học tập. **Output phải là kết quả của luyện tập có chủ đích và đốt calorie; mỗi ngày hãy để cơ thể và não bộ duy trì việc đốt cháy một lượng calorie nhất định.**

**4. Tạo ra sản phẩm có giá trị để hình thành feedback tích cực.**

Output phải có customer value, giúp bản thân học tập (bản thân phát triển và tiến hóa), đồng thời hữu ích cho người khác (thúc đẩy xã hội phát triển và tiến hóa). Như vậy, bạn có thể nhận được **feedback và measurement từ user**, hình thành một closed loop, liên tục cải tiến và nâng cao việc học.

**5. Ít chính là nhiều.**

Đào sâu một lĩnh vực (hoặc một số ít lĩnh vực liên quan). Mọi kế hoạch chi tiết phải xoay quanh chiến lược của bạn. Kiềm chế ham muốn trong lòng, đừng tham quá nhiều và mất tập trung, đừng để thế giới ồn ào đánh lạc hướng.

**6. Phải viết ra cả định hướng chiến lược và kế hoạch chi tiết, định kỳ review để tối ưu.**

**7. Phải giữ vững định hướng và nỗ lực liên tục.**

"Khúc tắc toàn, uổng tắc trực": việc thực hiện chiến lược không thể đi theo đường thẳng. Định hướng chiến lược và kế hoạch chi tiết thường cần điều chỉnh theo nhu cầu, đặc biệt ở giai đoạn đầu, nhưng cuối cùng phải hội tụ. Nếu cứ liên tục thay đổi mà không hội tụ, đó là thiếu sự kiên định chiến lược và là một vấn đề lớn cần suy nghĩ, giải quyết.

Có thể tham khảo chiến lược phát triển của người khác, nhưng đừng cố tình bắt chước. Bạn có màu sắc riêng, **bạn nên trở thành phiên bản độc nhất của chính mình.**

Sau khi định hướng chiến lược và kế hoạch chi tiết đã rõ ràng, việc tiếp theo là thực hiện từng bước, kiên định suốt mười năm như một ngày.

**8. Chậm chính là nhanh.**

Việc đạt được mục tiêu chiến lược cũng giống như trồng cây, cần được nuôi dưỡng và phát triển theo thời gian. Hãy nhớ **chậm chính là nhanh.** Khi lo lắng và bối rối, hãy lẩm nhẩm như tụng kinh lời dạy trong 《Truyền Tập Lục》 của Vương Dương Minh:

> Lập chí dụng công, như chủng thụ nhiên. Phương kỳ căn nha, do vị hữu cán; cập kỳ hữu cán, thượng vị hữu chi; chi nhi hậu diệp, diệp nhi hậu hoa thực. Sơ chủng căn thời, chỉ quản tài bồi quán dật. Vật tác chi tưởng, vật tác hoa tưởng, vật tác thực tưởng. Huyền tưởng hà ích? Đãn bất vong tài bồi chi công, phạ một hữu chi diệp hoa thực?
>
> Bản dịch:
>
> Thực hiện mục tiêu chiến lược cũng giống như trồng cây. Ban đầu chỉ là một mầm rễ nhỏ, thân cây còn chưa mọc; khi thân cây mọc lên, cành lá mới dần dần phát triển; cành mọc rồi mới có thể ra hoa và kết quả. Khi mới trồng cây, chỉ cần chăm sóc và tưới nước, đừng mãi băn khoăn bao giờ cành mọc, bao giờ hoa nở, bao giờ quả kết. Băn khoăn thì có ích gì? Chỉ cần kiên trì đầu tư chăm sóc, còn sợ không có cành lá, hoa quả sao?

<!-- @include: @article-footer.snippet.md -->
