---
title: Hướng dẫn viết CV cho lập trình viên
description: "Hướng dẫn viết CV cho lập trình viên: giải thích rõ cấu trúc CV, cách viết kinh nghiệm dự án và mô tả kỹ năng từ góc nhìn của quy trình sàng lọc, cung cấp mẫu CV và gợi ý tránh lỗi, giúp bạn tăng tỷ lệ CV được thông qua và giúp người phỏng vấn dễ khai thác điểm nổi bật của bạn hơn."
category: Chuẩn bị phỏng vấn
icon: "mdi:account-tie-outline"
head:
  - - meta
    - name: keywords
      content: CV lập trình viên,CV Java,tối ưu CV,cách viết kinh nghiệm dự án,mẫu CV,CV tuyển dụng sinh viên,CV tuyển dụng người đã đi làm,chuẩn bị phỏng vấn
---

::: tip Gợi ý
Bài viết này trích từ **[《Hướng dẫn phỏng vấn Java》](../zhuanlan/java-mian-shi-zhi-bei.md)**. Đây là một cẩm nang hướng dẫn bạn chuẩn bị phỏng vấn hiệu quả hơn, bao gồm các chủ đề thường gặp (system design, framework phổ biến, distributed system, high concurrency ...), các bài chia sẻ kinh nghiệm phỏng vấn chất lượng và nhiều nội dung khác.
:::

## Lời mở đầu

Một CV tốt có thể đóng vai trò rất quan trọng trong toàn bộ quá trình ứng tuyển và phỏng vấn.

**Vì sao CV lại quan trọng?** Có thể nhìn từ những điểm sau:

**1. CV giống như bộ mặt của chúng ta, ở mức độ rất lớn quyết định bạn có nhận được cơ hội phỏng vấn hay không.**

- Nếu bạn ứng tuyển trực tuyến, CV chắc chắn sẽ được HR sàng lọc. HR có thể chỉ dành khoảng 10 giây để xem một CV rồi quyết định bạn có được vào vòng phỏng vấn hay không.
- Nếu bạn được giới thiệu nội bộ, nếu CV không có ưu thế gì thì dù người giới thiệu có nhiệt tình đến đâu cũng không thể giúp được.

Ngoài ra, ngay cả khi vượt qua vòng sàng lọc đầu tiên và nhận được cơ hội phỏng vấn, trong các vòng sau, người phỏng vấn cũng sẽ dựa vào CV để phán đoán liệu bạn có đáng để họ dành nhiều thời gian phỏng vấn hay không.

**2. Nội dung trên CV quyết định ở mức độ rất lớn trọng tâm câu hỏi của người phỏng vấn.**

- Thông thường, người phỏng vấn sẽ hỏi những thứ bạn ghi là mình biết trên CV (Java Basics, collection, concurrency, MySQL, Redis, Spring, Spring Boot là những nội dung mà hầu như ai cũng được hỏi). Ví dụ, nếu bạn viết mình sử dụng Redis thành thạo, người phỏng vấn nhiều khả năng sẽ hỏi một số vấn đề về Redis; nếu bạn viết mình đã sử dụng message queue trong dự án, người phỏng vấn rất có thể sẽ hỏi nhiều câu liên quan đến message queue.
- Mức độ thành thạo kỹ năng cũng quyết định ở mức độ rất lớn độ sâu câu hỏi của người phỏng vấn.

Viết được một CV tốt mà không phóng đại năng lực của bản thân cũng là một năng lực rất đáng có. Thông thường, người có năng lực kỹ thuật và năng lực học tập tốt cũng viết được CV tốt hơn!

## Mẫu CV

Phong cách của CV thực sự rất rất quan trọng!!! Nếu CV của bạn xấu đến mức không ai muốn nhìn thì người phỏng vấn thực sự cũng không muốn đọc tiếp. Nỗi khổ khi phải xử lý hàng trăm CV mỗi ngày, bạn không hiểu đâu!

Ở đây, tôi khuyến nghị mọi người dùng cú pháp Markdown để viết CV, sau đó chuyển định dạng Markdown sang PDF rồi gửi CV ứng tuyển. Nếu bạn chưa hiểu rõ cú pháp Markdown, có thể dành nửa giờ để xem sơ lược hướng dẫn cú pháp Markdown: <http://www.markdown.cn/>.

Dưới đây là một số mẫu CV khá ổn mà tôi đã thu thập:

- Bộ sưu tập mẫu CV phù hợp với tiếng Trung (khuyến nghị, mã nguồn mở và miễn phí): <https://github.com/dyweb/awesome-resume-for-chinese>
- Muji CV (khuyến nghị, một phần miễn phí): <https://www.mujicv.com/>
- Simple CV (khuyến nghị, một phần miễn phí): <https://easycv.cn/>
- Minimal CV (miễn phí): <https://www.polebrief.com/index>
- Công cụ dàn trang CV Markdown (mã nguồn mở và miễn phí): <https://resume.mdnice.com/>
- Webmaster CV (tính phí, hỗ trợ tạo bằng AI): <https://jianli.chinaz.com/>
- Mẫu CV tùy chỉnh bằng typora+markdown+css: <https://github.com/Snailclimb/typora-markdown-resume>
- Super CV (một phần tính phí): <https://www.wondercv.com/>

Hầu hết các mẫu CV trên chỉ có nội dung dài 1 trang, khó thể hiện đủ lượng thông tin. Nếu bạn không phải là chuyên gia hàng đầu (ví dụ từng đạt giải trong cuộc thi ACM), tôi vẫn khuyên bạn cố gắng viết thêm nội dung có thể làm nổi bật năng lực của mình (CV tuyển dụng sinh viên trong vòng 2 trang, CV tuyển dụng người đã đi làm trong vòng 3 trang; nhớ cô đọng câu chữ, không viết quá nhiều lời thừa).

Tóm lại một số **lưu ý khi dàn trang CV**:

- Cố gắng ngắn gọn, đừng quá màu mè.
- Tên kỹ thuật nên dùng cách viết hoa chuẩn, ví dụ java -> Java, spring boot -> Spring Boot. Một số người phỏng vấn có thể không để ý điều này, nhưng nhiều người sẽ coi trọng chi tiết này.
- Thêm khoảng trắng giữa tiếng Việt và chữ số, tiếng Anh sẽ giúp CV trông dễ chịu hơn.

Ngoài ra, trong Zhishixingqiu còn có các mẫu CV thực tế để tham khảo, địa chỉ: <https://t.zsxq.com/12ypxGNzU> (cần tham gia [Zhishixingqiu](https://javaguide.cn/about-the-author/zhishixingqiu-two-years.html) để nhận được).

![](https://oss.javaguide.cn/javamianshizhibei/image-20230918073550606.png)

## Nội dung CV

### Thông tin cá nhân

- Thông tin cơ bản nhất: họ tên (tên trên căn cước), tuổi, số điện thoại, quê quán, thông tin liên hệ, địa chỉ email
- Điểm cộng tiềm năng: địa chỉ GitHub, địa chỉ blog (nếu blog kỹ thuật và GitHub không có nội dung gì thì không nên viết)

Ví dụ:

![](https://oss.javaguide.cn/zhishixingqiu/20210428212337599.png)

**Có nên đưa ảnh vào CV không?** Nhiều người gặp vấn đề này khi viết CV.

Thực ra có ảnh hay không đều được, ảnh hưởng không lớn, hoàn toàn không cần bận tâm vấn đề này. Trừ khi vị trí bạn ứng tuyển yêu cầu rõ ràng phải có ảnh. Tuy nhiên, nếu muốn đưa ảnh vào thì đừng dùng ảnh đời thường, nên dùng ảnh tương đối nghiêm túc như ảnh thẻ.

### Định hướng ứng tuyển

Bạn muốn ứng tuyển vị trí nào, muốn làm việc ở thành phố nào. Ngoài ra, bạn cũng có thể viết định hướng ứng tuyển trong phần thông tin cá nhân.

Ví dụ:

![](https://oss.javaguide.cn/zhishixingqiu/20210428212410288.png)

### Quá trình học tập

Quá trình học tập cũng không thể thiếu. Qua phần giới thiệu quá trình học tập, bạn cần đảm bảo người phỏng vấn biết được học vấn, chuyên ngành, trường tốt nghiệp và ngày tốt nghiệp của bạn.

Ví dụ:

> Đại học Bách khoa Bắc Kinh, thạc sĩ, Kỹ thuật phần mềm 2019.09 - 2022.01
> Đại học Hồ Nam, cử nhân, Hóa học ứng dụng 2015.09 ~ 2019.06

### Kỹ năng chuyên môn

Trước tiên hãy tự hỏi mình biết những gì, sau đó xem công ty bạn định ứng tuyển cần gì. Thông thường HR có thể không hiểu nhiều về kỹ thuật, nên khi sàng lọc CV, họ có thể chỉ tập trung xem các keyword về kỹ năng chuyên môn. Với những kỹ năng công ty yêu cầu nhưng bạn chưa biết, bạn có thể dành vài ngày học chúng, sau đó ghi trên CV rằng mình hiểu kỹ năng đó.

Dưới đây là danh sách kỹ năng Java backend mới nhất. Bạn có thể điều chỉnh linh hoạt dựa trên tình hình của bản thân và yêu cầu tuyển dụng của vị trí. Tư tưởng cốt lõi là cố gắng đáp ứng mọi yêu cầu kỹ năng của vị trí tuyển dụng.

![Mẫu kỹ năng Java backend](https://oss.javaguide.cn/zhishixingqiu/jinengmuban.png)

Tôi cũng đưa riêng một phần giới thiệu kỹ năng của một bạn mà tôi từng xem qua để cùng tìm vấn đề.

![](https://oss.javaguide.cn/zhishixingqiu/up-a58d644340f8ce5cd32f9963f003abe4233.png)

Phần giới thiệu kỹ năng trong hình trên có những vấn đề sau:

- Tên kỹ thuật nên dùng cách viết hoa chuẩn, ví dụ java -> Java, spring boot -> Spring Boot. Một số người phỏng vấn có thể không để ý điều này, nhưng nhiều người sẽ coi trọng chi tiết này.
- Phần giới thiệu kỹ năng quá tạp, không có điểm nổi bật. Không cần biết mọi thứ, chỉ cần làm tốt một lĩnh vực là được!
- Độ thành thạo một số kỹ năng của Java backend, chẳng hạn Spring Boot, chỉ dừng ở mức hiểu, không đáp ứng được yêu cầu của doanh nghiệp.

### Kinh nghiệm thực tập/kinh nghiệm làm việc (quan trọng)

Kinh nghiệm làm việc dành cho người đã đi làm, kinh nghiệm thực tập dành cho sinh viên.

Kinh nghiệm làm việc nên được giới thiệu theo thứ tự thời gian đảo ngược. Cả kinh nghiệm thực tập và kinh nghiệm làm việc đều cần nêu bật ngắn gọn những việc chính bạn đã làm trong thời gian làm việc.

Ví dụ:

> **Công ty XXX (tháng X năm 201X ~ tháng X năm 201X)**
>
> - **Vị trí**: Kỹ sư Java backend
> - **Nội dung công việc**: Chủ yếu phụ trách XXX

### Kinh nghiệm dự án (quan trọng)

CV có một hoặc hai kinh nghiệm dự án là điều rất bình thường, nhưng số người thực sự có thể trình bày tốt kinh nghiệm dự án cho người phỏng vấn lại rất ít.

Phần giới thiệu kinh nghiệm dự án của nhiều ứng viên thường gặp các vấn đề như quá dài dòng, quá đơn giản, không làm nổi bật điểm mạnh.

Mẫu giới thiệu kinh nghiệm dự án như sau:

> Tên dự án (nên dùng cỡ chữ lớn hơn)
>
> 2017-05~2018-06 Kỹ sư Java backend tại Taobao
>
> - **Mô tả dự án**: Mô tả ngắn gọn dự án dùng để làm gì.
> - **Tech stack**: Đã sử dụng những công nghệ nào (ví dụ Spring Boot + MySQL + Redis + Mybatis-plus + Spring Security + Oauth2)
> - **Nội dung công việc/Trách nhiệm cá nhân**: Mô tả ngắn gọn bạn đã làm gì, giải quyết vấn đề gì và mang lại cải thiện thực chất nào. Làm nổi bật năng lực của bản thân, không kể lại quá bình thường.
> - **Thu hoạch cá nhân (tùy chọn)**: Bạn đã học được gì từ dự án này, đã sử dụng những kỹ thuật nào, học được cách sử dụng những kỹ thuật mới nào. Thông thường có thể không cần viết thu hoạch cá nhân, vì những gì bạn viết trong phần trách nhiệm cá nhân đã thể hiện thu hoạch chính của mình.
> - **Kết quả dự án (tùy chọn)**: Mô tả ngắn gọn dự án đã đạt được thành tích gì.

**1. Kinh nghiệm dự án nên làm nổi bật bạn đã làm gì, đồng thời tóm tắt ngắn gọn tình hình cơ bản của dự án.**

Phần giới thiệu dự án nên được cô đọng trong tối đa hai dòng, không cần giới thiệu quá nhiều nhưng cũng đừng tùy tiện dùng vài chữ để kết thúc.

Ngoài ra, thu hoạch cá nhân và kết quả dự án đều là tùy chọn. Nếu chọn viết thì cũng đừng dành quá nhiều dung lượng cho chúng, hãy nhớ trọng tâm là giới thiệu nội dung công việc/trách nhiệm cá nhân.

**2. Với technical architecture, chỉ cần viết trực tiếp tên kỹ thuật, không cần giới thiệu kỹ thuật đó dùng để làm gì, không có ý nghĩa và thuộc dạng giới thiệu vô ích.**

![](https://oss.javaguide.cn/github/javaguide/interview-preparation/46c92fbc5160e65dd85c451143177144.png)

**3. Cố gắng giảm các phần giới thiệu trách nhiệm cá nhân thuần nghiệp vụ, điều này không thân thiện với phỏng vấn. Hãy cố gắng khai thác thêm điểm nổi bật (khoảng 6-8 phần giới thiệu trách nhiệm cá nhân là vừa đủ, hãy chọn lọc), tốt nhất là thể hiện tố chất tổng hợp của bản thân, chẳng hạn bạn đã phối hợp các thành viên trong nhóm dự án cùng phát triển như thế nào, đã giải quyết một vấn đề hóc búa ra sao, hoặc đã tối ưu performance của một module trong dự án như thế nào.**

Ngay cả khi tính năng hoặc vấn đề không phải do bạn thực hiện hay giải quyết, chỉ cần hiểu rõ và nắm chắc thì bạn vẫn có thể dùng cho mình, chỉ cần trau chuốt phù hợp!

Các điểm nổi bật theo hướng performance cũng tương đối dễ chuẩn bị trước phỏng vấn, nhưng đừng để tất cả đều liên quan đến performance, đó cũng là một cực đoan.

Ngoài ra, thành quả đạt được từ việc tối ưu kỹ thuật nên được định lượng hết mức có thể:

- Dùng kỹ thuật xxx giải quyết vấn đề xxx, QPS của hệ thống tăng từ xxx lên xxx.
- Dùng kỹ thuật xxx tối ưu API xxx, QPS của hệ thống tăng từ xxx lên xxx.
- Dùng kỹ thuật xxx giải quyết vấn đề xxx, tốc độ truy vấn được tối ưu xxx, QPS của hệ thống đạt 100k+.
- Dùng kỹ thuật xxx tối ưu module xxx, response time giảm từ 2s xuống 0.2s.
- ……

Ví dụ giới thiệu trách nhiệm cá nhân (đây chỉ là ví dụ, đừng sao chép nguyên văn, hãy tự viết dựa trên kinh nghiệm dự án của mình, nếu không khi phỏng vấn rất dễ bị hỏi khó):

- Dựa trên Spring Cloud Gateway + Spring Security OAuth2 + JWT triển khai authentication, authorization và permission thống nhất cho microservice, sử dụng mô hình quyền RBAC để thực hiện kiểm soát quyền động.
- Tham gia phát triển module order của dự án, phụ trách các chức năng tạo, xóa, truy vấn order, dựa trên Spring state machine để thực hiện chuyển đổi trạng thái order.
- Đưa Elasticsearch vào các trường hợp tìm kiếm sản phẩm và order, đồng thời triển khai các chức năng đề xuất sản phẩm liên quan và gợi ý tìm kiếm.
- Tích hợp Canal + RabbitMQ để đồng bộ dữ liệu tăng thêm của MySQL (như dữ liệu sản phẩm, order) sang Elasticsearch.
- Sử dụng plugin delayed queue do RabbitMQ chính thức cung cấp để triển khai các trường hợp delayed task như tự động hủy order quá hạn, nhắc nhở coupon hết hạn và xử lý refund.
- Đưa RabbitMQ vào hệ thống push message để thực hiện xử lý bất đồng bộ, peak shaving và decoupling service, tốc độ push tối đa 100k/s, lượng message tối đa trong một ngày là 20 triệu.
- Sử dụng công cụ MAT phân tích file dump để giải quyết vấn đề cảnh báo service timeout hàng loạt sau khi phiên bản mới của service quảng cáo được đưa lên production.
- Điều tra và giải quyết vấn đề deadlock do parent task tính phí và child task anti-fraud sử dụng cùng một thread pool.
- Dựa trên EasyExcel triển khai import/export dữ liệu phân phối quảng cáo, sử dụng MyBatis batch processing để insert dữ liệu, dựa trên task table thực hiện bất đồng bộ.
- Phụ trách phát triển module thống kê user, sử dụng CompletableFuture để load song song thông tin dữ liệu của module thống kê user backend, response time trung bình giảm từ 3.5s xuống 1s.
- Dựa trên Sentinel thực hiện rate limiting và degradation cho các trường hợp cốt lõi (như đăng nhập, đăng ký user, truy vấn địa chỉ nhận hàng), bảo vệ hệ thống và nâng cao trải nghiệm user.
- Dùng Redis+Caffeine làm cache hai cấp cho dữ liệu phổ biến (như trang chủ, blog phổ biến), giải quyết vấn đề cache breakdown và cache penetration, tốc độ truy vấn ở mức mili giây, QPS 300k+.
- Sử dụng CompletableFuture tối ưu module truy vấn shopping cart, orchestration các lời gọi RPC bất đồng bộ để lấy thông tin user, chi tiết sản phẩm, thông tin coupon..., response time giảm từ 2s xuống 0.2s.
- Xây dựng service EasyMock để mô phỏng API của nền tảng bên thứ ba, tạo thuận lợi cho việc tích hợp API trong điều kiện cách ly mạng.
- Dựa trên SkyWalking + Elasticsearch xây dựng hệ thống distributed tracing để thực hiện monitoring toàn bộ chain.

**4. Nếu cảm thấy kỹ thuật trong dự án của mình khá lạc hậu, bạn có thể tự cải tiến trong thời gian riêng. Điều quan trọng là làm cho dự án có điểm nổi bật, dùng cách nào không quan trọng.**

Phần kinh nghiệm dự án rất quan trọng đối với CV. Phần chuẩn bị phỏng vấn của [《Hướng dẫn phỏng vấn Java》](https://javaguide.cn/zhuanlan/java-mian-shi-zhi-bei.html) có vài bài viết về tối ưu kinh nghiệm dự án, khuyến nghị bạn đọc kỹ, chúng sẽ có ích cho bạn.

![](https://oss.javaguide.cn/zhishixingqiu/4e11dbc842054e53ad6c5f0445023eb5~tplv-k3u1fbpfcp-zoom-1.png)

**5. Tránh viết phần giới thiệu trách nhiệm cá nhân chỉ xoay quanh một technical point, điều này rất không nên.**

![](https://oss.javaguide.cn/zhishixingqiu/image-20230424222513028.png)

**6. Tránh mô tả mơ hồ, phần giới thiệu cần cụ thể (kỹ thuật + trường hợp sử dụng + hiệu quả), đồng thời chú ý cô đọng câu chữ (tránh chất đống từ kỹ thuật, lược bỏ mô tả không cần thiết).**

![](https://oss.javaguide.cn/github/javaguide/interview-preparation/project-experience-avoiding-ambiguity-descriptio.png)

### Thành tích, giải thưởng (tùy chọn)

Nếu bạn có thành tích đạt giải trong các cuộc thi có giá trị (ví dụ ACM, cuộc thi Tianchi của Alibaba), nhất định phải viết phần thành tích, giải thưởng này! Ngoài ra, bạn cũng có thể đưa phần thành tích, giải thưởng lên trước một cách phù hợp, đặt ở vị trí dễ thấy hơn.

### Kinh nghiệm hoạt động trường học (tùy chọn)

Nếu có kinh nghiệm hoạt động trường học nổi bật thì viết ngắn gọn, không có thì không cần viết!

### Tự đánh giá

**Tự đánh giá là cách diễn giải về bản thân. Nhất định phải dùng ngôn ngữ súc tích để làm nổi bật đặc điểm và ưu thế của mình, tránh lời thừa!** Đừng nói những điều chung chung như chăm chỉ, chịu khó; người phỏng vấn sẽ thấy phiền khi đọc kiểu tự đánh giá này.

Có thể viết phần tự đánh giá từ những góc độ sau:

- Năng lực viết tài liệu, năng lực học tập, năng lực giao tiếp, năng lực phối hợp nhóm
- Thái độ đối với công việc và tinh thần trách nhiệm cá nhân
- Áp lực công việc có thể chịu đựng và thái độ đối với khó khăn
- Theo đuổi kỹ thuật, theo đuổi chất lượng code
- Kinh nghiệm phát triển hoặc vận hành hệ thống distributed, high concurrency

Ba ví dụ thực tế:

- Năng lực học tập tốt, khi tham gia cuộc thi thiết kế phần mềm cấp quốc gia vào năm ba đại học, đã nhanh chóng học Python và viết một hệ thống crawler có thể cấu hình.
- Có tinh thần phối hợp nhóm, khi tham gia cuộc thi thiết kế phần mềm cấp quốc gia vào năm ba đại học, đã điều phối 5 bạn developer trong nhóm dự án, hỗ trợ các bạn gặp khó khăn khi coding và cuối cùng hoàn thành thuận lợi các chức năng cốt lõi của dự án trong 1 tháng.
- Có kinh nghiệm dự án phong phú, từng dẫn dắt phát triển nhiều dự án cấp doanh nghiệp trong thời gian học đại học.

## Nguyên tắc STAR và FAB

### Nguyên tắc STAR (Situation Task Action Result)

Chắc hẳn mọi người đều từng nghe về nguyên tắc STAR. Khi phỏng vấn, bạn có thể áp dụng nguyên tắc này vào CV và quá trình trao đổi với người phỏng vấn.

Nguyên tắc STAR gồm 4 từ sau (tên của nguyên tắc STAR được tạo từ chữ cái đầu của chúng):

- **Situation:** Tình huống. Sự việc xảy ra trong hoàn cảnh nào?
- **Task:** Nhiệm vụ. Nhiệm vụ của bạn là gì?
- **Action:** Hành động. Bạn đã làm gì?
- **Result:** Kết quả. Kết quả cuối cùng thế nào?

### Nguyên tắc FAB (Feature Advantage Benefit)

Ngoài nguyên tắc STAR, bạn cũng cần biết một nguyên tắc có tên FAB thường được sử dụng trong ngành sales.

Nguyên tắc FAB gồm 3 từ sau (tên của nguyên tắc FAB được tạo từ chữ cái đầu của chúng):

- **Feature:** Đặc điểm/ưu thế của bạn là gì?
- **Advantage:** Bạn tốt hơn người khác ở những điểm nào?
- **Benefit:** Nếu tuyển bạn, bên tuyển dụng sẽ nhận được lợi ích gì?

## Gợi ý

### Tránh quá nhiều trang

Tinh gọn cách diễn đạt, làm nổi bật điểm mạnh. CV tuyển dụng sinh viên không nên dài quá 2 trang, CV tuyển dụng người đã đi làm không nên dài quá 3 trang. Nếu nội dung quá nhiều thì không cần cố ép xuống một trang, chỉ cần giữ dàn trang sạch sẽ, gọn gàng là được.

Tôi đã xem hàng nghìn CV, có một số ít bạn viết CV gần 10 trang, khiến tôi toát mồ hôi.

![CV quá nhiều trang](https://oss.javaguide.cn/zhishixingqiu/image-20230508223646164.png)

### Tránh ngữ nghĩa mơ hồ

Cố gắng tránh cách diễn đạt chủ quan, giảm bớt các tính từ mơ hồ về ngữ nghĩa. Cách diễn đạt cần ngắn gọn, rõ ràng, cấu trúc CV cần mạch lạc.

Ví dụ:

- Cách diễn đạt không tốt: Tôi đóng một vai trò rất quan trọng trong nhóm.
- Cách diễn đạt tốt: Với vai trò technical lead backend, tôi dẫn dắt nhóm hoàn thành việc design và development dự án backend.

### Chú ý phong cách CV

Phong cách CV cũng rất quan trọng, nhất định phải chú ý! Không cần theo đuổi sự màu mè, nhưng phải cố gắng đảm bảo cấu trúc rõ ràng và dễ đọc.

### Khác

- Nhất định phải gửi CV ở định dạng PDF, không dùng Word hoặc định dạng khác. Đây là điều cơ bản nhất!
- Không biết gì thì đừng viết lên CV. Hãy chú ý tính chân thực của CV, trau chuốt phù hợp thì không vấn đề gì.
- Kinh nghiệm làm việc nên được giới thiệu theo thứ tự thời gian đảo ngược, kinh nghiệm thực tập nên đưa nội dung có giá trị nhất lên đầu.
- Việc thể hiện hoàn hảo kinh nghiệm dự án của bản thân là rất quan trọng. Trọng tâm là làm nổi bật bạn đã làm gì (khai thác điểm mạnh), chứ không phải giới thiệu dự án dùng để làm gì.
- Kinh nghiệm dự án nên được sắp xếp theo thứ tự thời gian đảo ngược. Ngoài ra, kinh nghiệm dự án không nằm ở số lượng (chọn 2-3 dự án là đủ), mà nằm ở điểm nổi bật.
- Trong quá trình chuẩn bị phỏng vấn, bạn nên xem những gì đã viết trên CV là trọng tâm, đặc biệt là phần kinh nghiệm dự án và giới thiệu kỹ năng.
- Phỏng vấn và công việc là hai chuyện khác nhau. Người thông minh sẽ dẫn người phỏng vấn vào lĩnh vực mình giỏi, còn những người khác sẽ bị người phỏng vấn dắt mũi. Dù phỏng vấn và công việc là hai chuyện khác nhau, nếu muốn nhận được offer ưng ý thì năng lực của bản thân phải đủ mạnh.

## Chỉnh sửa CV

Cho đến nay, tôi đã giúp ít nhất **6000+** bạn trong cộng đồng cung cấp dịch vụ chỉnh sửa CV miễn phí. Vì thời gian cá nhân có hạn, việc chỉnh sửa CV chỉ dành cho độc giả tham gia cộng đồng. Nếu cần được xem CV, bạn có thể tham gia [**Zhishixingqiu chính thức của JavaGuide**](https://javaguide.cn/about-the-author/zhishixingqiu-two-years.html#%E7%AE%80%E5%8E%86%E4%BF%AE%E6%94%B9) (nhấp vào liên kết để xem giới thiệu chi tiết).

![img](https://oss.javaguide.cn/xingqiu/%E7%AE%80%E5%8E%86%E4%BF%AE%E6%94%B92.jpg)

Mặc dù phí chỉ bằng một phần một trăm so với lớp đào tạo/trại huấn luyện, chất lượng nội dung trong Zhishixingqiu cao hơn, dịch vụ cũng toàn diện hơn, rất phù hợp với những bạn chuẩn bị phỏng vấn Java và học Java.

Dưới đây là một số dịch vụ mà cộng đồng cung cấp (nhấp vào hình bên dưới để xem giới thiệu chi tiết về Zhishixingqiu):

[![Dịch vụ cộng đồng](https://oss.javaguide.cn/xingqiu/xingqiufuwu.png)](../about-the-author/zhishixingqiu-two-years.md)

Ở đây cũng cung cấp một phiếu ưu đãi độc quyền có thời hạn:

![Phiếu ưu đãi 30 tệ của Zhishixingqiu](https://oss.javaguide.cn/xingqiu/xingqiuyouhuijuan-30.jpg)
