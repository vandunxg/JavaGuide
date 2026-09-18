---
title: Hướng dẫn kinh nghiệm dự án
description: "Hướng dẫn kinh nghiệm dự án: dành cho ứng viên chưa có dự án hoặc dự án chưa nổi bật, đưa ra phương pháp và gợi ý lựa chọn để có kinh nghiệm dự án thực chiến, đồng thời giải thích cách tạo điểm nổi bật cho dự án, cách đánh giá lại và trình bày, từ đó nâng cao sức cạnh tranh của CV và phỏng vấn."
category: Chuẩn bị phỏng vấn
icon: "mdi:projector-screen-outline"
head:
  - - meta
    - name: keywords
      content: Kinh nghiệm dự án,Dự án tuyển dụng campus,Dự án thực chiến,Điểm nổi bật dự án,Mô tả dự án trong CV,Dự án backend,Chuẩn bị dự án phỏng vấn,Đánh giá lại dự án
---

::: tip Lưu ý
Bài viết này trích từ **[《Java Interview Guide》](../zhuanlan/java-mian-shi-zhi-bei.md)**. Đây là chuyên mục hướng dẫn bạn chuẩn bị phỏng vấn hiệu quả hơn, bổ trợ cho JavaGuide, bao quát các chủ đề lý thuyết thường gặp (system design, framework phổ biến, distributed system, high concurrency …… ), cùng các bài chia sẻ kinh nghiệm phỏng vấn chất lượng.
:::

## Làm gì khi chưa có kinh nghiệm dự án?

Chưa có kinh nghiệm dự án là vấn đề mà phần lớn sinh viên mới tốt nghiệp sẽ gặp phải. Thậm chí, nhiều lập trình viên đã có kinh nghiệm làm việc nhưng không hài lòng với dự án mình từng làm ở công ty, cũng muốn tìm một dự án có hàm lượng kỹ thuật cao hơn để thực hiện.

Dưới đây là một vài cách có vẻ đáng tin cậy để có kinh nghiệm dự án, hy vọng có thể giúp ích cho bạn.

### Video/chuyên mục dự án thực chiến

Tìm trên Internet một video hoặc chuyên mục về dự án thực chiến phù hợp với năng lực và nhu cầu tìm việc của bạn, rồi làm theo giảng viên.

Bạn có thể tìm video/chuyên mục dự án thực chiến phù hợp qua các kênh như Mooc, Bilibili, Lagou, Geek Time, các trung tâm đào tạo (chẳng hạn Heima, Shang Silicon Valley).

![Khóa học thực chiến trên Mooc](https://oss.javaguide.cn/javamianshizhibei/mukewangzhiazhanke.png)

Hãy cố gắng chọn một dự án phù hợp với mình. Không nhất thiết phải làm dự án distributed hoặc microservice; với phần lớn mọi người, làm tốt một dự án chạy trên một máy đã là rất đáng ghi nhận.

Tôi từng phỏng vấn nhiều ứng viên. Nhìn vào CV thì có vẻ họ có kinh nghiệm với dự án microservice, nhưng chỉ cần hỏi tùy ý hai câu là biết ngay dự án đó vốn không phải do họ tự làm, hoặc khi làm họ hoàn toàn không suy nghĩ nghiêm túc. Tình huống này sẽ để lại ấn tượng rất xấu.

Trong **[《Java Interview Guide》](../zhuanlan/java-mian-shi-zhi-bei.md)**, phần “Chuẩn bị phỏng vấn”, tôi cũng từng nói:

> Theo tôi, không nhất thiết phải làm dự án microservice hoặc distributed, vì chưa chắc có lợi cho phỏng vấn. Dự án microservice hoặc distributed liên quan đến quá nhiều kiến thức, người bình thường rất khó hiểu thấu đáo. Hơn nữa, với sinh viên mới tốt nghiệp, những dự án này thực ra hơi quá sức. Dù bạn làm xong, nhiều interviewer vẫn sẽ cho rằng đó không phải dự án bạn tự hoàn thành.
>
> Thực ra, bạn làm một dự án đơn thể thật tốt cũng rất ổn; về nâng cao năng lực cá nhân, nó không hề thua kém việc làm dự án microservice hoặc distributed. Làm thế nào để đạt đến mức thật tốt? Tạm không nói đến chất lượng code; quan trọng hơn là bạn nên cố gắng tạo một số điểm nổi bật cho dự án (chẳng hạn bạn đã nâng performance của dự án như thế nào, giải quyết một vấn đề tồn tại trong dự án ra sao). Thành quả đạt được từ dự án nên được định lượng hết sức có thể, ví dụ tôi đã dùng công nghệ xxx để giải quyết vấn đề xxx, đưa QPS của hệ thống từ xxx lên xxx.

Trong quá trình làm theo giảng viên, bạn nhất định phải có suy nghĩ riêng, không nên học qua loa. Với nhiều kiến thức, phần giải thích của người khác có thể chỉ cần đủ để hoàn thành dự án; nếu muốn hiểu thêm, bạn phải tự đào sâu các kiến thức quan trọng.

### Dự án thực chiến mã nguồn mở

Trên GitHub hoặc Gitee có rất nhiều dự án mã nguồn mở dạng thực chiến. Bạn có thể chọn một dự án để nghiên cứu; để hiểu dự án sâu hơn, ngoài việc nắm rõ code hiện có, bạn còn có thể cải tiến hoặc bổ sung tính năng cho dự án.

Bạn có thể tham khảo các dự án mã nguồn mở thực chiến được đề xuất trong [Các dự án mã nguồn mở Java chất lượng cao](https://javaguide.cn/open-source-project/practical-project.html "Các dự án mã nguồn mở Java chất lượng cao"). Chất lượng của chúng đều rất cao, loại hình dự án cũng khá đầy đủ, bao gồm hệ thống blog/forum, hệ thống thi và luyện bài, hệ thống thương mại điện tử, hệ thống quản lý quyền, scaffold phát triển nhanh và nhiều công cụ tự xây khác.

![Các dự án thực chiến Java mã nguồn mở chất lượng cao](https://oss.javaguide.cn/javamianshizhibei/javaguide-practical-project.png)

Hãy luôn nhớ: **không chỉ phải làm, mà còn phải cải tiến và hoàn thiện.** Dù là video/chuyên mục dự án thực chiến hay dự án mã nguồn mở thực chiến, chắc chắn đều có nhiều điểm có thể hoàn thiện và cải tiến.

### Tự làm từ đầu

Tự bắt tay làm một sản phẩm bạn muốn hoàn thành; khi gặp điều chưa biết thì học ngay lúc đó và áp dụng ngay.

Cách này yêu cầu khá cao. Tôi khuyên bạn chỉ nên áp dụng sau khi đã có kinh nghiệm với một dự án. Nếu chưa từng làm dự án, tốt hơn hết vẫn nên nghiêm túc áp dụng hai cách ở trên.

### Tham gia các cuộc thi do các công ty lớn tổ chức

Nếu đạt giải trong các cuộc thi này, giá trị thực tế của dự án sẽ rất cao. Dù không đạt giải cũng không sao, bạn vẫn có thể ghi vào CV.

![Cuộc thi Alibaba Cloud Tianchi](https://oss.javaguide.cn/xingqiu/up-673f598477242691900a1e72c5d8b26df2c.png)

### Tham gia dự án thực tế

Thông thường, bạn có thể tiếp cận việc phát triển dự án thực tế của doanh nghiệp qua các cách sau:

1. Dự án do giảng viên nhận;
2. Dự án freelance tự nhận;
3. Dự án tiếp xúc được trong quá trình thực tập/làm việc.

Dự án do giảng viên nhận và dự án freelance tự nhận thường thiên về nghiệp vụ, ít khi liên quan đến tối ưu performance. Trong trường hợp này, bạn có thể cân nhắc cải tiến dự án. Đừng ngại tốn thời gian; chỉ cần tập trung làm tốt một việc trong một khoảng thời gian, chẳng hạn cải tiến data model của dự án, đưa cache vào để tăng tốc độ truy cập, v.v.

Dự án tiếp xúc được trong quá trình thực tập/làm việc cũng tương tự. Nếu gặp dự án thiên về nghiệp vụ, bạn vẫn nên tự cải tiến và tối ưu dự án ngoài giờ.

Tốt nhất là bạn thực sự tối ưu dự án, vì bản thân việc này cũng giúp nâng cao năng lực cá nhân. Nếu thực sự không có thời gian thực hành thì cũng không sao, chỉ cần hiểu thật kỹ phương pháp tối ưu dự án, đồng thời chuẩn bị trước một số vấn đề có thể gặp trong phỏng vấn.

## Có đề xuất dự án nào khá ổn không?

Trong phần “Chuẩn bị phỏng vấn” của **[《Java Interview Guide》](../zhuanlan/java-mian-shi-zhi-bei.md)** có một bài viết chuyên tổng hợp một số dự án thực chiến chất lượng cao, bao gồm đề xuất về dự án nghiệp vụ, dự án xây công cụ, Lab của các khóa học công khai nước ngoài và tutorial dự án thực chiến dạng video. Đây là tài liệu rất phù hợp để học hoặc dùng làm kinh nghiệm dự án.

![Đề xuất dự án thực chiến Java chất lượng cao](https://oss.javaguide.cn/javamianshizhibei/project-experience-guide.png)

Bài viết này đề xuất hơn 15 dự án thực chiến, gồm cả dự án nghiệp vụ và dự án xây công cụ, dự án mã nguồn mở và tutorial dạng video. Với những bạn tham gia tuyển dụng campus, tôi khuyên nên làm một dự án nghiệp vụ kết hợp với một dự án xây công cụ.

## Dự án tôi làm theo video có bị interviewer chê không?

Nhiều sinh viên mới tốt nghiệp đều làm dự án theo video, và phần lớn interviewer đều biết rõ điều này.

Không loại trừ việc một số interviewer thực sự không chấp nhận cách này, điều đó còn tùy người. Tuy nhiên, tôi tin phần lớn interviewer có thể thông cảm; dù sao khi còn ở trường, thực tế bạn không có nhiều cách để có được kinh nghiệm dự án thực tế.

Phần lớn kinh nghiệm dự án của sinh viên mới tốt nghiệp đều đến từ việc tự tìm trên Internet, hoặc giống như bạn, mua khóa học trả phí rồi làm theo; chỉ một số rất ít là dự án thực tế. Việc bạn muốn làm một dự án thực chiến vốn có mục đích tốt và thực sự có thể học được điều gì đó. Tuy nhiên, điều quan trọng là bạn đã nắm được bao nhiêu. Điều tối kỵ khi xem video là tiếp nhận một cách thụ động; hãy tự cải tiến và suy nghĩ nhiều hơn! Ngay cả dự án làm theo video cũng có thể được tối ưu!

**Nếu thực sự muốn học được điều gì đó, bạn không nên chỉ hoàn thành dự án và chạy được nó, mà còn phải tự thử tối ưu!**

Dưới đây là một vài điểm tối ưu tương đối dễ thực hiện:

1. **Xử lý exception toàn cục**: Nhiều dự án làm chưa tốt phần này. Bạn có thể tham khảo bài viết [《Đóng gói đơn giản một cơ chế xử lý exception toàn cục Spring Boot thanh lịch bằng enum!》](https://mp.weixin.qq.com/s/Y4Q4yWRqKG_lw0GLUsY2qw) của tôi để tối ưu.
2. **Tối ưu lựa chọn công nghệ của dự án**: Chẳng hạn, nơi đang dùng Guava để làm cache local có thể đổi sang **Caffeine**. Caffeine có performance tốt hơn về nhiều mặt! Hoặc hãy xem tầng Controller có đang chứa quá nhiều logic nghiệp vụ hay không.
3. **Về database**: Thiết kế database có thể tối ưu không? index đã được sử dụng đúng chưa? Câu lệnh SQL có thể tối ưu không? Có cần read-write splitting không?
4. **Cache**: Dự án có dữ liệu nào thường xuyên được truy cập không? Có nên thêm cache để tăng tốc độ phản hồi không?
5. **Security**: Dự án có tồn tại vấn đề security không?
6. ……

Ngoài ra, tôi từng chia sẻ trên cộng đồng các ví dụ thực tế về những hướng tối ưu performance thường gặp, liên quan đến multi-threading, asynchronous, index, cache và các hướng khác. Rất khuyến khích bạn xem: <https://t.zsxq.com/06EqfeMZZ> .

Cuối cùng, **xin giới thiệu thêm một mẹo tối ưu code bằng IDEA, cực kỳ hữu ích!**

Phân tích code của bạn: nhấp chuột phải vào dự án -> Analyze->Inspect Code

![](https://oss.javaguide.cn/xingqiu/up-651672bce128025a135c1536cd5dc00532e.png)

Sau khi quét xong, IDEA sẽ đưa ra một số code smell có thể tồn tại, chẳng hạn vấn đề về naming.

![](https://oss.javaguide.cn/xingqiu/up-05c83b319941995b07c8020fddc57f26037.png)

Ngoài ra, bạn còn có thể tùy chỉnh các rule kiểm tra.

![](https://oss.javaguide.cn/xingqiu/up-6b618ad3bad0bc3f76e6066d90c8cd2f255.png)

## Chuẩn bị đào sâu dự án sau khi hoàn thành như thế nào?

Hoàn thành dự án mới chỉ giải quyết vấn đề “có dự án hay không”. Trong technical interview, interviewer sẽ tiếp tục hỏi về kiến trúc dự án, trách nhiệm cá nhân, lựa chọn công nghệ, core flow, chỉ số performance và sự cố production. Bạn nên chuẩn bị hai phiên bản giới thiệu cho mỗi dự án trọng điểm trong CV: một phiên bản 30 giây và một phiên bản 3 phút, sau đó lần lượt tự hỏi sâu về database, cache, message queue và thread pool mà mình đã sử dụng.

Phương pháp chuẩn bị cụ thể có thể tham khảo: [《Trình bày dự án backend trong phỏng vấn như thế nào? Từ giới thiệu dự án đến khó khăn kỹ thuật và đánh giá lại sự cố》](./backend-project-interview-guide.md). Hệ thống order trong bài viết chỉ dùng để minh họa cấu trúc câu trả lời; trách nhiệm và chỉ số của dự án vẫn phải được thay bằng tư liệu thực tế của bạn.

## Chuẩn bị dự án AI Agent như thế nào?

Nếu chuẩn bị dự án AI Agent, ngoài việc trình bày rõ mình đã làm gì, bạn còn phải trả lời được vì sao sử dụng Agent, request được luân chuyển như thế nào, cách đảm bảo an toàn cho thao tác ghi của tool, cũng như đã xác định và khắc phục một lần thất bại ra sao. Bạn có thể đọc tiếp [《Trình bày dự án Agent trong phỏng vấn như thế nào? Từ kiến trúc hệ thống, lựa chọn công nghệ đến đánh giá lại Badcase》](../ai/interview-questions/agent-project-interview-guide.md). Case trong bài viết chỉ có thể dùng làm tham khảo để tổ chức câu trả lời; trách nhiệm và chỉ số của dự án vẫn phải dựa trên trải nghiệm thực tế của bạn.
