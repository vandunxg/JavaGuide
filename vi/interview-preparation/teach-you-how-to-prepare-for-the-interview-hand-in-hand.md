---
title: Làm thế nào để chuẩn bị phỏng vấn Java hiệu quả?
description: "Làm thế nào để chuẩn bị phỏng vấn Java hiệu quả: từ học tập theo định hướng tìm việc, lập danh sách kỹ năng đến tối ưu CV và tăng tốc trước phỏng vấn, cung cấp phương pháp chuẩn bị có hệ thống, giúp bạn tránh đường vòng và nâng cao tỷ lệ đậu phỏng vấn."
category: Knowledge Planet
icon: "mdi:map-marker-path"
head:
  - - meta
    - name: keywords
      content: chuẩn bị phỏng vấn Java, chuẩn bị phỏng vấn hiệu quả, học tập theo định hướng tìm việc, tăng tốc trước phỏng vấn, tối ưu CV, chuẩn bị dự án, tuyển dụng sinh viên, backend Java
---

::: tip Lưu ý
Bài viết này trích từ **[《Cẩm nang phỏng vấn Java》](../zhuanlan/java-mian-shi-zhi-bei.md)**. Đây là chuyên mục hướng dẫn bạn chuẩn bị phỏng vấn hiệu quả hơn, bổ trợ cho JavaGuide, bao gồm các chủ đề phỏng vấn thường gặp (system design, framework phổ biến, distributed system, high concurrency ……​), kinh nghiệm phỏng vấn chất lượng và nhiều nội dung khác.
:::

Bên cạnh bạn có người bạn nào như thế này không: năng lực lập trình mạnh hơn bạn nhưng kết quả tìm việc lại không bằng bạn? Thực ra **giỏi kỹ thuật ≠ đậu phỏng vấn** —— phỏng vấn ngày nay từ lâu đã không còn là “biết viết code là đủ”, đi phỏng vấn mà không chuẩn bị thì phần lớn là “đâm đầu vào họng súng”.

Phần lớn chúng ta là những developer bình thường, không có bài báo tại hội nghị hàng đầu hay giải thưởng cuộc thi làm chỗ dựa. Trước thực tế “phỏng vấn thì chế rocket, đi làm thì vặn ốc vít”, chúng ta chỉ có thể dựa vào sự chuẩn bị vững chắc để bứt phá. Nhưng chuẩn bị phỏng vấn không có nghĩa là dùng mánh khóe hay học thuộc lòng câu hỏi phỏng vấn. **Nhất định không được mang tâm lý may rủi khi phỏng vấn. Muốn rèn sắt thì bản thân phải cứng!** Đừng nghĩ rằng chỉ cần đọc vài bài kinh nghiệm phỏng vấn, vài bài giải thích câu hỏi phỏng vấn là có thể đậu. Nhất định phải bình tâm học chuyên sâu!

Bài viết này sẽ đứng trên góc nhìn tổng quan để giúp bạn hiểu cách developer chuẩn bị phỏng vấn một cách có hệ thống: từ học theo định hướng tìm việc đến tối ưu CV và tăng tốc trước phỏng vấn, giúp bạn tránh đường vòng và hiệu quả giành được offer mong muốn.

## Học theo định hướng tìm việc càng sớm càng tốt

Tôi khá khuyến nghị các bạn còn đang đi học nên bắt đầu học theo định hướng tìm việc càng sớm càng tốt.

**Cách này có định hướng hơn, nhiều khả năng giảm thời gian bạn rơi vào trạng thái mơ hồ, đồng thời giúp bạn tránh được nhiều đường vòng.**

Tuy nhiên! Đừng hiểu “học theo định hướng tìm việc” thành “vậy là tôi không cần học các môn cơ sở máy tính trên lớp nữa”!

Trong nhiều lần chia sẻ trước đây, tôi đã nhấn mạnh: **nhất định phải học nghiêm túc kiến thức cơ sở máy tính! Operating system, computer organization, computer network thực sự không phải những môn học vô dụng trong thực tế!!!**

Bạn sẽ thấy chúng được dùng trong phỏng vấn ở các công ty lớn, và sau này đi làm cũng sẽ dùng đến. Tôi nêu hai ví dụ riêng nhé!

- **Trong phỏng vấn**: Các buổi phỏng vấn kỹ thuật ở những công ty lớn như ByteDance, Tencent và bài thi viết của gần như mọi công ty đều hỏi các vấn đề liên quan đến operating system.
- **Trong công việc**: Khi sử dụng cache trong thực tế, xét ở tầng software, tư tưởng cache bắt nguồn từ sự không tương xứng giữa tốc độ database, Redis (middleware trong memory) và memory cục bộ; trong thiết kế cấu trúc phân tầng lưu trữ của máy tính, chúng ta cũng thấy vấn đề tương tự và việc sử dụng tư tưởng cache: memory giải quyết vấn đề tốc độ truy cập disk quá chậm, CPU dùng three-level cache để giảm chênh lệch tốc độ giữa register và memory. Chúng đều đối mặt với cùng một vấn đề (tốc độ không tương xứng) và cùng một tư tưởng, vì vậy các biện pháp tối ưu performance của cache mà những người tiên phong trong lĩnh vực máy tính áp dụng khi thiết kế cấu trúc phân tầng lưu trữ cũng phù hợp với việc tối ưu performance của cache ở tầng software.

**Học theo định hướng tìm việc như thế nào?** Nói đơn giản là: dựa trên yêu cầu tuyển dụng để tổng hợp một danh sách kỹ năng cho vị trí mục tiêu, sau đó học tập và nâng cao theo danh sách kỹ năng này.

1. Trước tiên bạn phải làm rõ mình muốn tìm công việc gì
2. Sau đó tổng hợp một danh sách kỹ năng dựa trên yêu cầu của vị trí tuyển dụng
3. Viết CV hoàn chỉnh dựa trên danh sách kỹ năng
4. Cuối cùng học tập và nâng cao theo yêu cầu của CV.

Đây thực ra cũng là cách vận dụng tư tưởng **bắt đầu từ kết quả**.

**Bắt đầu từ kết quả là gì?** Nói đơn giản, bắt đầu từ kết quả nghĩa là đứng trên góc nhìn của kết quả để suy xét vấn đề, xuất phát từ kết quả và xác định những việc cần làm dựa trên kết quả đó.

Bạn sẽ thấy tư tưởng **bắt đầu từ kết quả** có thể được áp dụng trong gần như mọi lĩnh vực.

## Nắm được thời điểm vàng để nộp CV

Trước phỏng vấn, chắc chắn bạn phải tìm hiểu rõ thời gian cụ thể của đợt tuyển dụng mùa xuân và mùa thu.

Người ta thường nói tháng 3 vàng tháng 4 bạc, tháng 9 vàng tháng 10 bạc; bỏ lỡ thời gian này thì nhiều công ty sẽ không còn HC.

**Tuyển dụng mùa thu thường bắt đầu từ tháng 7 và kéo dài đến khoảng cuối tháng 9.**

**Tuyển dụng mùa xuân thường bắt đầu từ tháng 3 và kéo dài đến khoảng cuối tháng 4.**

Đến giữa tháng 9 (tuyển dụng mùa thu)/giữa tháng 3 (tuyển dụng mùa xuân), nhiều công ty, đặc biệt là các công ty lớn, rất có thể đã không còn HC. Phỏng vấn thường bắt đầu từ ít nhất 3 vòng, một số công ty lớn như Alibaba, ByteDance có thể có 5 vòng phỏng vấn. **Nếu phỏng vấn thất bại thì cũng không sao; nếu thể hiện không tốt ở một vòng nào đó cũng không sao, hãy điều chỉnh tâm lý. Bạn đâu chỉ có một lựa chọn, đúng không? Bạn còn có thể ứng tuyển rất nhiều công ty! Hãy điều chỉnh tâm lý.** Năm nay, do ảnh hưởng của dịch bệnh, một số công ty có thể vẫn tiếp tục tập trung phỏng vấn online. Cũng do ảnh hưởng của dịch bệnh, tìm việc có thể khó hơn những năm trước (ảnh hưởng đến các công ty lớn ít hơn).

## Biết cách tìm thông tin tuyển dụng

Dưới đây là những kênh phổ biến để tìm thông tin tuyển dụng:

- **Website chính thức/tài khoản công khai của doanh nghiệp mục tiêu**: kênh lấy thông tin tuyển dụng nhanh và chính thống nhất.
- **Website tuyển dụng**: [BOSS Zhipin](https://www.zhipin.com/), [Zhaopin](https://www.zhaopin.com/), [Lagou](https://www.lagou.com/)……​.
- **Nowcoder**: Mỗi mùa tuyển dụng mùa thu/mùa xuân, rất nhiều công ty đăng thông tin tuyển dụng trên Nowcoder, đồng thời cũng có nhiều nhân viên công ty đăng bài giới thiệu nội bộ tại đây. Địa chỉ: <https://www.nowcoder.com/jobs/recommend/campus> .
- **WonderCV**: WonderCV hiện đã tổng hợp các cổng tuyển dụng trong campus của nhiều doanh nghiệp lớn, địa chỉ: <https://www.wondercv.com/jobs/. Nếu bạn ứng tuyển campus, chỉ cần nhấp vào “đăng ký online campus” là có thể chuyển thẳng đến trang tổng hợp cổng tuyển dụng campus của các doanh nghiệp lớn.>
- **Bạn bè quen biết**: Nếu bạn có bạn bè đang làm việc tại doanh nghiệp mục tiêu, bạn có thể hỏi họ về thông tin tuyển dụng và nhờ họ giới thiệu nội bộ.
- **Buổi giới thiệu tuyển dụng**: Đây cũng là một kênh khá tốt, tuy nhiên doanh nghiệp tốt thường chỉ đến những trường tốt. Bạn có thể chú ý lịch tổ chức của doanh nghiệp mình quan tâm hoặc trực tiếp đến một trường tốt để tham gia. Khi còn ứng tuyển campus, tôi từng tham gia vài buổi giới thiệu. Tuy nhiên, tôi học ở Kinh Châu, nơi đó không có nhiều trường tốt nên thường không có công ty đến tổ chức. Vì vậy lúc đó tôi trực tiếp đến Vũ Hán, tham gia vài buổi giới thiệu ở Đại học Công nghệ Vũ Hán và Đại học Khoa học Kỹ thuật Hoa Trung. Nhìn chung cảm nhận vẫn rất tốt!
- **Khác**: website thông tin việc làm của trường, diễn đàn trường, nhóm QQ của lớp hoặc khóa.

Nếu ứng tuyển campus, bạn nên lấy website chính thức làm chuẩn; nếu có buổi giới thiệu thì càng tốt. Nếu ứng tuyển người đã đi làm, hãy chú ý thêm thông tin vị trí trên các website tuyển dụng như BOSS Zhipin và Lagou.

Bất kể ứng tuyển campus hay ứng tuyển người đã đi làm, nếu tìm được cơ hội giới thiệu nội bộ đáng tin cậy thì xác suất có cơ hội phỏng vấn vẫn rất cao. Hơn nữa, bạn có thể nhờ người giới thiệu nội bộ đưa ra một số đề xuất có định hướng. Có nhiều cách tìm cơ hội giới thiệu nội bộ, ưu tiên bạn bè và bạn học quen thuộc; ngoài ra cũng có thể chú ý thông tin giới thiệu nội bộ trên cộng đồng trao đổi kỹ thuật và tài khoản công khai.

Thông thường bạn chỉ được ứng tuyển một vị trí, nhưng cũng có rất ít trường hợp ứng tuyển hai vị trí ở các bộ phận khác nhau. Việc này có lẽ không ảnh hưởng, nhưng kết quả phỏng vấn trước đó của bạn có thể được ghi lại. Nói cách khác, ngay cả khi bạn ứng tuyển thành công hai vị trí, nếu thất bại ở vòng phỏng vấn của vị trí đầu tiên thì vị trí thứ hai cũng sẽ bị ảnh hưởng, thậm chí rất có thể bị Pass trực tiếp.

## Dành thêm thời gian hoàn thiện CV

Nhất định, nhất định, nhất định phải coi trọng CV! Các bạn! Ít nhất hãy dành 2~3 ngày để chuyên tâm hoàn thiện CV.

Gần đây tôi đã xem rất nhiều CV nhưng rất ít bản khiến tôi hài lòng. Tôi lấy một bản để phân tích đơn giản (hoan nghênh bổ sung ở phần bình luận).

**1. Phần giới thiệu cá nhân không có nhiều thông tin hữu ích.**

![](https://oss.javaguide.cn/github/javaguide/interview-preparation/format,png.png)

Nếu có technical blog, GitHub và thành tích đạt giải khi còn đi học thì nên viết vào đây nếu có thể. Bạn có thể tham khảo template bên dưới 👇 để chỉnh sửa:

![](https://oss.javaguide.cn/github/javaguide/interview-preparation/format,png-20230309224235808.png)

**2. Kinh nghiệm dự án quá sơ sài, hoàn toàn không có chất lượng**

![](https://oss.javaguide.cn/github/javaguide/interview-preparation/format,png-20230309224240305.png)

Mỗi kinh nghiệm dự án thực sự chỉ có thể mô tả bằng một hai câu thôi sao? Hay là bạn không muốn viết? Hay là vì không tự làm nên không dám viết nhiều?

Nếu có dự án, bước đầu tiên trong technical interview thường là interviewer yêu cầu bạn tự giới thiệu dự án. Bạn có thể suy xét từ một số hướng sau:

1. Cảm nhận của bạn về thiết kế tổng thể của dự án (interviewer có thể yêu cầu bạn vẽ system architecture diagram)
2. Trong dự án này bạn phụ trách gì, đã làm gì, đảm nhiệm vai trò gì.
3. Bạn học được gì từ dự án này, đã sử dụng những công nghệ nào, học được cách sử dụng những công nghệ mới nào.
4. Bạn đã từng giải quyết vấn đề nào trong dự án này chưa? Đã giải quyết như thế nào? Đạt được gì?
5. Dự án của bạn sử dụng những công nghệ nào? Bạn đã thực sự hiểu rõ các công nghệ đó chưa? Ví dụ, trong kinh nghiệm dự án bạn sử dụng Seata để thực hiện distributed transaction, vậy bạn nên chuẩn bị trước các vấn đề liên quan đến Seata, chẳng hạn Seata hỗ trợ những configuration center nào, transaction group của Seata được thực hiện ra sao, Seata hỗ trợ những transaction mode nào và lựa chọn thế nào?
6. Bạn từng mắc lỗi gì trong dự án này, cuối cùng đã khắc phục ra sao?

**3. Đối với chuyên ngành máy tính, hoàn toàn không cần viết chứng chỉ cấp độ máy tính 2 vì chứng chỉ này không có giá trị.**

![](https://oss.javaguide.cn/github/javaguide/interview-preparation/format,png-20230309224247261.png)

**4. Phần giới thiệu kỹ năng có vấn đề quá lớn.**

![](https://oss.javaguide.cn/github/javaguide/interview-preparation/93da1096fb02e19071ba13b4f6a7471c.png)

- Tốt nhất nên chuẩn hóa chữ hoa chữ thường của tên kỹ thuật, chẳng hạn java -> Java, spring boot -> Spring Boot. Tuy một số interviewer không để ý, nhưng nhiều interviewer rất coi trọng chi tiết này.
- Phần giới thiệu kỹ năng quá tạp, không có điểm nổi bật. Không cần biết mọi thứ, chỉ cần làm tốt một lĩnh vực là được!
- Với một số kỹ năng của Java backend như Spring Boot, mức độ quen thuộc chỉ dừng ở biết qua, không thể đáp ứng yêu cầu của doanh nghiệp.

Để xem hướng dẫn viết CV chi tiết cho developer, tham khảo: [CV developer rốt cuộc nên viết thế nào?](https://javaguide.cn/interview-preparation/resume-guide.html).

## Mức độ phù hợp với vị trí rất quan trọng

Tuyển dụng campus thường khá linh hoạt với hướng nghiên cứu trong kinh nghiệm dự án của bạn. Ngay cả khi kinh nghiệm dự án không liên quan đến nghiệp vụ cụ thể của công ty, ảnh hưởng thực tế cũng không lớn.

Với tuyển dụng người đã đi làm thì khác. Dù sao công ty cũng muốn tuyển người có thể bắt tay làm việc ngay, nếu bạn có kinh nghiệm liên quan thì công ty sẽ đỡ việc hơn. Tuyển dụng người đã đi làm thường coi trọng kinh nghiệm làm việc trước đây và kinh nghiệm dự án của bạn. Khi HR sàng lọc CV, họ sẽ dựa trên hai nhóm thông tin này để đánh giá bạn có đáp ứng yêu cầu tuyển dụng hay không. Ví dụ bạn ứng tuyển công ty thương mại điện tử nhưng trước đây không có kinh nghiệm làm việc và kinh nghiệm dự án liên quan đến thương mại điện tử, HR rất có thể sẽ Pass CV của bạn ngay khi sàng lọc.

Tuy nhiên, điều này cũng không tuyệt đối. Khi tuyển dụng, một số công ty coi trọng hơn kinh nghiệm trước đây của bạn và ít chú ý đến mức độ phù hợp với vị trí. Kinh nghiệm làm việc tại công ty tốt và kinh nghiệm dự án nổi bật đều là điểm cộng. Những công ty này tin rằng nếu bạn đã làm tốt trong một lĩnh vực nào đó (chẳng hạn thương mại điện tử, thanh toán), thì hẳn bạn cũng có thể nhanh chóng trở thành chuyên gia trong lĩnh vực khác (chẳng hạn nền tảng streaming, phần mềm mạng xã hội). “Lĩnh vực” ở đây không phải lĩnh vực kỹ thuật mà thiên về hướng nghiệp vụ. Chuyển ngang giữa các lĩnh vực kỹ thuật (chẳng hạn backend chuyển sang thuật toán, backend chuyển sang big data) để tìm việc, nếu không có kinh nghiệm liên quan thì gần như không thể tìm được. Ngay cả khi tìm được, rất có thể bạn sẽ phải đối mặt với việc HR ép mức lương.

## Chuẩn bị trước cho technical interview

Trước phỏng vấn nhất định phải chuẩn bị trước các câu hỏi phỏng vấn thường gặp, tức các câu hỏi lý thuyết thường gặp:

- Những knowledge point nào có thể xuất hiện trong phỏng vấn, knowledge point nào là trọng tâm.
- Những vấn đề nào thường được hỏi trong phỏng vấn, bạn nên trả lời thế nào. (Cực kỳ không khuyến nghị học thuộc lòng. Thứ nhất: bằng cách học thuộc, bạn có thể nhớ được bao nhiêu và nhớ được bao lâu? Thứ hai: rất khó kiên trì học theo cách học thuộc câu hỏi!)

Trọng tâm ôn tập cho phỏng vấn Java backend hãy xem bài viết này: [Tổng hợp trọng tâm phỏng vấn Java (quan trọng)](https://javaguide.cn/interview-preparation/key-points-of-interview.html).

Các loại công ty khác nhau có trọng tâm yêu cầu kỹ năng khác nhau. Chẳng hạn Tencent, ByteDance có thể coi trọng hơn kiến thức cơ sở máy tính như network và operating system; Alibaba, Meituan có thể coi trọng hơn kinh nghiệm dự án và năng lực thực chiến.

Nhất định đừng mang suy nghĩ rằng việc kiểm tra câu hỏi lý thuyết thường gặp hoặc câu hỏi cơ sở không có nhiều ý nghĩa. Nếu ôn tập với suy nghĩ này thì hiệu quả có thể không tốt. Thực tế, theo tôi chúng vẫn rất có ý nghĩa; câu hỏi lý thuyết thường gặp hoặc kiến thức cơ sở cũng thường được dùng trong phát triển hằng ngày. Ví dụ, với thread pool, nếu bạn không hiểu rejection policy, cấu hình core parameters và những thứ tương tự, khi sử dụng thread pool trong dự án thực tế có thể sẽ không hiểu rõ cách dùng, dễ phát sinh vấn đề. Hơn nữa, các câu hỏi cơ sở như vậy thực ra là phần dễ chuẩn bị nhất; những nội dung như nguyên lý bên trong, system design, câu hỏi tình huống và đào sâu vào dự án của bạn mới là phần khó nhất!

Tài liệu câu hỏi lý thuyết thường gặp đầu tiên nên chọn [《Cẩm nang phỏng vấn Java》](https://javaguide.cn/zhuanlan/java-mian-shi-zhi-bei.html) (kết hợp với JavaGuide, nội dung sẽ được cập nhật và hoàn thiện theo tình hình phỏng vấn mỗi năm) và [JavaGuide](https://javaguide.cn/). Trong đó không chỉ có các câu hỏi lý thuyết do tác giả tự viết mà còn có nhiều kiến thức thực tế hữu ích cho phát triển. Ngoài tài liệu của tôi, bạn cũng có thể tìm trên mạng một số bài viết và video chất lượng khác để xem.

![Tổng quan nội dung 《Cẩm nang phỏng vấn Java》](https://oss.javaguide.cn/javamianshizhibei/javamianshizhibei-content-overview.png)

## Chuẩn bị trước việc giải thuật toán trực tiếp

Rõ ràng là các buổi phỏng vấn tuyển dụng campus trong nước hiện nay ngày càng coi trọng thuật toán, đặc biệt ở những công ty lớn như ByteDance và Tencent. Bài thi viết khi tuyển dụng campus của phần lớn công ty đều có câu hỏi thuật toán; nếu tỷ lệ AC thấp thì cơ bản là bị loại.

Với tuyển dụng người đã đi làm, phỏng vấn thuật toán cũng vẫn có. Tuy nhiên, interviewer có thể coi trọng hơn năng lực engineering và kinh nghiệm dự án của bạn. Nếu các mặt khác của bạn đều rất tốt nhưng thuật toán yếu thì chưa chắc bị loại. Dù vậy, vẫn khuyến nghị bạn luyện một số bài thuật toán để tránh biến nó thành điểm yếu trong phỏng vấn.

Tuyển dụng người đã đi làm thường đưa cho bạn một bài toán thuật toán ở cuối technical interview để bạn giải.

Về cách chuẩn bị phỏng vấn thuật toán, phần chuẩn bị phỏng vấn của [《Cẩm nang phỏng vấn Java》](https://javaguide.cn/zhuanlan/java-mian-shi-zhi-bei.html) có giới thiệu chi tiết.

![Phần chuẩn bị phỏng vấn trong 《Cẩm nang phỏng vấn Java》](https://oss.javaguide.cn/javamianshizhibei/preparation-for-interview.png)

## Chuẩn bị trước phần tự giới thiệu

Tự giới thiệu thường là lần giao tiếp trực tiếp chính thức đầu tiên giữa bạn và interviewer. Hãy thử đặt mình vào vị trí của interviewer: nếu bạn là interviewer, bạn muốn nghe ứng viên giới thiệu bản thân thế nào? Chắc chắn không phải là nói xã giao rằng mình thích lập trình, thường dành nhiều thời gian học tập, sở thích là chơi bóng đúng không?

Theo tôi, một phần tự giới thiệu tốt ít nhất nên có những yếu tố sau:

- Dùng lời lẽ súc tích nói rõ tech stack chính và lĩnh vực mình giỏi;
- Tập trung vào điểm mạnh và nơi mình sở trường;
- Làm nổi bật năng lực của bản thân, chẳng hạn năng lực định vị bug của bạn đặc biệt tốt;

Nói đơn giản là dùng ngôn ngữ súc tích để làm nổi bật điểm mạnh của mình, tức là tự giới thiệu bản thân!

- Nếu từng thực tập ở công ty lớn thì kinh nghiệm thực tập tương ứng là điểm mạnh.
- Nếu từng tham gia cuộc thi kỹ thuật thì kinh nghiệm thi đấu là điểm mạnh.
- Nếu đã tiếp xúc với việc phát triển dự án cấp doanh nghiệp từ thời đại học và có nhiều kinh nghiệm thực chiến thì kinh nghiệm dự án đó là điểm mạnh.
- ……

Hãy lấy ví dụ từ hai góc độ tuyển dụng người đã đi làm và tuyển dụng campus! Hai ví dụ dưới đây chỉ mang tính tham khảo. Tự giới thiệu không cần học thuộc lòng; chỉ cần nhớ các ý chính, khi phỏng vấn ứng biến theo tình hình công ty cũng không sao. Ngoài ra, trên mạng thường khuyến nghị chuẩn bị hai bản tự giới thiệu: một bản nói với HR, chủ yếu kể những kinh nghiệm làm nổi bật bản thân, lướt qua các kỹ thuật lập trình biết dùng; bản còn lại nói với interviewer kỹ thuật, chủ yếu trình bày chi tiết kỹ thuật và kinh nghiệm dự án.

**Tuyển dụng người đã đi làm:**

> Chào interviewer! Tôi tên là Duxiu. Hiện tôi có 1 năm rưỡi kinh nghiệm làm việc, thành thạo các framework như Spring, MyBatis, hiểu nguyên lý bên trong của Java như JVM tuning và có nhiều kinh nghiệm phát triển distributed system. Tôi rời công ty trước vì muốn được rèn luyện thêm về kỹ thuật. Ở công ty trước, tôi tham gia phát triển một hệ thống giao dịch điện tử phân tán, phụ trách xây dựng architecture cơ sở cho toàn bộ dự án và giải quyết vấn đề database ban đầu cùng một số bảng liên quan quá lớn bằng sharding. Hiện website này hỗ trợ tối đa 100.000 người truy cập đồng thời. Ngoài giờ làm, tôi dùng thời gian rảnh viết một RPC framework đơn giản. Framework này sử dụng Netty để network communication, hiện tôi đã open source dự án trên GitHub và nhận được 2k Star! Về sở thích, tôi khá thích tổng hợp và chia sẻ kiến thức đã học qua blog, hiện đã là tác giả được chứng nhận trên nhiều nền tảng blog. Trong cuộc sống, tôi là người khá tích cực và lạc quan, thường thư giãn bằng cách vận động và chơi bóng. Tôi luôn rất mong muốn gia nhập quý công ty, tôi rất yêu thích văn hóa và technical atmosphere của quý công ty, mong được làm việc cùng bạn!

**Tuyển dụng campus:**

> Chào interviewer! Tôi tên là Xiuer. Khi học đại học, tôi chủ yếu tận dụng thời gian ngoài giờ để học Java và các framework như Spring, MyBatis. Trong thời gian học, tôi từng tham gia phát triển một hệ thống thi. Hệ thống này chủ yếu sử dụng ba framework Spring, MyBatis và shiro. Trong đó tôi đảm nhiệm phát triển backend, chủ yếu phụ trách xây dựng module quản lý quyền. Ngoài ra, khi học đại học tôi từng tham gia một cuộc thi lập trình software; hệ thống đặt đồ ăn online do tôi và team thực hiện đã đạt hạng nhì. Tôi cũng tận dụng thời gian rảnh viết một RPC framework đơn giản. Framework này sử dụng Netty để network communication, hiện tôi đã open source dự án trên GitHub và nhận được 2k Star! Về sở thích, tôi khá thích tổng hợp và chia sẻ kiến thức đã học qua blog, hiện đã là tác giả được chứng nhận trên nhiều nền tảng blog. Trong cuộc sống, tôi là người khá tích cực và lạc quan, thường thư giãn bằng cách vận động và chơi bóng. Tôi luôn rất mong muốn gia nhập quý công ty, tôi rất yêu thích văn hóa và technical atmosphere của quý công ty, mong được làm việc cùng bạn!

## Giảm phàn nàn

Giống như technical interview hiện nay, mọi người đều nói cạnh tranh quá khốc liệt và phàn nàn rằng phỏng vấn hiện tại khó kinh khủng. Nhưng chỉ phàn nàn thì có ích gì? Bạn nói với những ứng viên khác: “Mọi người đừng luyện Leetcode nữa! Cũng đừng chuẩn bị câu hỏi phỏng vấn về high concurrency và high availability nữa! Hiện giờ cạnh tranh đã khốc liệt như vậy rồi!”

Có ai nghe bạn không? **Bạn không chuẩn bị phỏng vấn nhưng người khác sẽ chuẩn bị! Bạn ngốc thật hay sao? Hay bạn thực sự giỏi đến mức không cần chuẩn bị phỏng vấn?**

Vì vậy, bước đầu tiên khi chuẩn bị phỏng vấn Java là cố gắng giảm phàn nàn. Sau khi những lời phàn nàn xuất hiện quá nhiều, chúng sẽ ảnh hưởng rất lớn đến bản thân, khiến bạn trở nên cực kỳ lo lắng.

## Kịp thời review sau phỏng vấn

Nếu thất bại, đừng nản lòng; nếu đậu, đừng quá vui mừng. Phỏng vấn và công việc thực tế là hai chuyện khác nhau. Có thể nhiều người không đậu phỏng vấn lại có năng lực làm việc mạnh hơn bạn rất nhiều, và ngược lại.

Phỏng vấn giống như một hành trình hoàn toàn mới, thất bại và chiến thắng đều là chuyện bình thường. Vì vậy, khuyên mọi người đừng nản lòng hay mất ý chí vì thất bại trong phỏng vấn. Cũng đừng tự mãn vì đậu phỏng vấn; tương lai tốt đẹp hơn đang chờ bạn, hãy tiếp tục cố gắng!

## Tổng kết

Bài viết này hơi dài. Nếu bài viết chỉ giúp bạn nhớ được 7 câu, hãy nhớ 7 câu dưới đây:

1. Nhất định phải chuẩn bị phỏng vấn trước! Technical interview khác với lập trình, giỏi lập trình không có nghĩa chắc chắn đậu technical interview.
2. Nhất định không được mang tâm lý may rủi khi phỏng vấn. Muốn rèn sắt thì bản thân phải cứng! Đừng nghĩ rằng chỉ cần đọc vài bài kinh nghiệm phỏng vấn, vài bài giải thích câu hỏi phỏng vấn là có thể đậu. Nhất định phải bình tâm học chuyên sâu! Đặc biệt nếu mục tiêu là công ty lớn thì càng phải đào sâu nguyên lý!
3. Khuyến nghị sinh viên đại học bắt đầu học theo định hướng tìm việc càng sớm càng tốt. Cách này có định hướng hơn, nhiều khả năng giảm thời gian bạn rơi vào trạng thái mơ hồ, đồng thời giúp bạn tránh được nhiều đường vòng. Tuy nhiên, đừng hiểu “học theo định hướng tìm việc” thành “vậy là tôi không cần học các môn cơ sở máy tính trên lớp nữa”!
4. Nhất định đừng mang suy nghĩ rằng việc kiểm tra câu hỏi lý thuyết thường gặp hoặc câu hỏi cơ sở không có nhiều ý nghĩa. Nếu ôn tập với suy nghĩ này thì hiệu quả có thể không tốt. Thực tế, theo tôi chúng vẫn rất có ý nghĩa; câu hỏi lý thuyết thường gặp hoặc kiến thức cơ sở cũng thường được dùng trong phát triển hằng ngày. Ví dụ, với thread pool, nếu bạn không hiểu rejection policy, cấu hình core parameters và những thứ tương tự, khi sử dụng thread pool trong dự án thực tế có thể sẽ không hiểu rõ cách dùng, dễ phát sinh vấn đề.
5. Giải thuật toán trực tiếp là tiêu chuẩn trong technical interview hiện nay, hãy chuẩn bị càng sớm càng tốt!
6. Mức độ phù hợp với vị trí rất quan trọng. Tuyển dụng campus thường khá linh hoạt với hướng nghiên cứu trong kinh nghiệm dự án của bạn. Ngay cả khi kinh nghiệm dự án không liên quan đến nghiệp vụ cụ thể của công ty, ảnh hưởng thực tế cũng không lớn. Với tuyển dụng người đã đi làm thì khác. Dù sao công ty cũng muốn tuyển người có thể bắt tay làm việc ngay, nếu bạn có kinh nghiệm liên quan thì công ty sẽ đỡ việc hơn.

7. Kịp thời review sau phỏng vấn. Phỏng vấn giống như một hành trình hoàn toàn mới, thất bại và chiến thắng đều là chuyện bình thường. Vì vậy, khuyên mọi người đừng nản lòng hay mất ý chí vì thất bại trong phỏng vấn. Cũng đừng tự mãn vì đậu phỏng vấn; tương lai tốt đẹp hơn đang chờ bạn, hãy tiếp tục cố gắng!
