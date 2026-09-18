---
title: Cảm nhận sau nửa năm từ tốt nghiệp đến đi làm
description: Cảm nhận sau nửa năm đi làm của một sinh viên mới tốt nghiệp, giá trị của business code CRUD, tích lũy kỹ thuật nhờ thời gian ngoài giờ làm và chia sẻ trải nghiệm chuyển đổi từ trường học sang môi trường công sở.
category: Đến gần tác giả
tag:
  - Trải nghiệm cá nhân
---

Nếu đã đọc phần giới thiệu trước đây của tôi, bạn sẽ biết tôi là một trong hàng triệu sinh viên mới tốt nghiệp năm 2019. Bài viết này chủ yếu nói về cảm nhận của tôi sau hơn nửa năm đi làm. Trong bài có nhiều cảm nhận chủ quan của riêng tôi, nếu bạn không đồng tình ở điểm nào thì cứ nói thẳng trong phần bình luận, tôi sẽ tôn trọng suy nghĩ của mọi người.

Nói sơ qua về tình hình của tôi! Hiện tại tôi đang làm việc tại một công ty nước ngoài, công việc hằng ngày cũng giống phần lớn mọi người, chủ yếu là development. Từ khi tốt nghiệp đến nay, tôi đã đi làm được hơn nửa năm, đồng thời cũng đã qua thời gian thử việc 6 tháng của công ty. Hiện tại, tôi đã làm hai project thiên về business tại công ty, một project vẫn đang thực hiện. Khó mà tưởng tượng được rằng backend của hai project business tôi làm ở công ty đều không liên quan đến distributed system/microservice, cũng chưa tiếp xúc với việc ứng dụng thực tế các công nghệ khá “cao cấp” như Redis, Kafka trong project.

Project đầu tiên là một project nội bộ của công ty — hệ thống phát triển nhân viên. Bỏ qua cái tên hệ thống phát triển nhân viên, thực tế hệ thống này làm về performance review, chẳng hạn như đánh giá của bạn trong một project team. Công nghệ của project này là Spring Boot + JPA + Spring Security + K8S + Docker + React. Project thứ hai hiện đang thực hiện là một project tích hợp game (cocos), Web admin (Spring Boot + Vue) và mini program (Taro).

Đúng vậy, phần lớn thời gian làm việc của tôi liên quan đến CRUD, hằng ngày tôi cũng viết các page frontend. Trước đây, một người bạn tôi quen nghe nói phần lớn nội dung project tôi làm đều là business code thì rất ngạc nhiên. Cậu ấy cho rằng chỉ viết business code thì không thể tiến bộ? what? Một người mới tốt nghiệp như bạn còn viết business code chưa tốt mà đã nói thế với tôi! Vì vậy, **tôi rất khó hiểu tại sao hiện nay nhiều người còn viết business code chưa tốt lại phản cảm khi nghe đến CRUD? Ít nhất tôi cảm thấy trong thời gian làm việc, chất lượng code của mình đã được nâng lên, năng lực định vị vấn đề đã cải thiện đáng kể, hiểu biết về business cũng sâu hơn, đồng thời tôi đã có thể tự hoàn thành một số phần development frontend.**

Thực ra, cá nhân tôi cho rằng viết tốt business code cũng không dễ như vậy. Trước khi phàn nàn rằng ngày nào mình cũng làm CRUD, hãy xem code CRUD của mình đã viết tốt chưa. Nói cách khác, trong quá trình chỉ viết CRUD, bạn đã hiểu rõ những annotation hoặc class thường dùng nào? Điều này giống như một người chỉ biết những annotation đơn giản nhất như `@Service`, `@Autowired`, `@RestController`... nói rằng mình đã làm chủ Spring Boot vậy.

Không biết từ khi nào mọi người đều cho rằng có kinh nghiệm thực tế sử dụng Redis, MQ thì rất ghê gớm, điều này có thể liên quan đến môi trường interview hiện nay. Bạn cần khác biệt với người khác, nếu muốn vào các công ty lớn thì dường như bắt buộc phải thành thạo những công nghệ này. Được rồi, không phải “dường như” — nói tự tin hơn thì với phần lớn người tìm việc, những công nghệ này mặc định đã là thứ bắt buộc phải có.

**Nói thật, khi còn học đại học, tôi cũng từng rơi vào “mệnh đề giả” này**. Năm hai đại học, vì tham gia một kênh truyền thông của trường thiên về kỹ thuật, tôi mới bắt đầu tiếp xúc với Java. Khi đó, mục đích học Java của chúng tôi là development một hệ thống dùng trong trường. Lúc ấy, trình độ lập trình của tôi mới chỉ ở mức nhập môn, phải mất một thời gian mới nắm được Java Basics. Sau đó, tôi bắt đầu học Android development.

Đến học kỳ một năm ba, tôi mới thực sự xác định hướng đi Java backend và tìm việc development Java backend. Sau khoảng 3 tháng học các kiến thức cơ bản về WEB development, tôi bắt đầu học nội dung về distributed system như Redis, Dubbo. Khi đó, tôi học theo cách đọc sách + xem video + đọc blog. Trong quá trình tự học, tôi đã tự làm hai project hoàn chỉnh bằng cách xem video: một business system thông thường và một distributed system. **Khi ấy tôi tưởng làm xong là mình rất giỏi, cho rằng công việc CRUD thông thường đã không còn phù hợp với trình độ hiện tại của mình nữa. Ha ha! Giờ nhìn lại, tôi khi đó đúng là quá ngây thơ!**

Thế rồi, đến kỳ nghỉ hè năm ba, khi làm project cùng thầy giáo thì vấn đề xuất hiện. Năm ba, chúng tôi làm cùng thầy một hệ thống performance review có độ phức tạp business ở mức trung bình. Công nghệ của project này là: SSM + Shiro + JSP. Khi làm project đó, tôi gặp đủ loại vấn đề; những đoạn code mà tôi tưởng mình biết viết thì lại không viết được, thậm chí viết một CRUD đơn giản cũng mất vài ngày. Vì vậy, khi ấy tôi vừa ôn tập, vừa học, vừa viết code. Dù rất mệt, tôi đã học được nhiều điều và trở nên vững vàng hơn trước kỹ thuật. Tôi cho rằng câu “**project này không còn khả năng bảo trì**” là lời phủ nhận lớn nhất của tôi đối với project mình từng làm.

Công nghệ thay đổi muôn hình vạn trạng, nắm được những thứ cốt lõi nhất mới là điều quan trọng. Vài năm trước, có thể chúng ta còn dùng Spring development theo XML truyền thống; hiện nay, hầu như mọi người đều dùng công cụ development Spring Boot để nâng cao tốc độ development. Chẳng hạn, vài năm trước khi sử dụng message queue, có thể chúng ta còn dùng ActiveMQ, nhưng đến nay hầu như không còn ai dùng nó, các lựa chọn thường dùng hiện tại là Rocket MQ và Kafka. Trong thời đại công nghệ thay đổi nhanh như vậy, bạn không thể học từng framework/tool một.

**Nhiều người mới bắt đầu đã muốn học thông qua việc làm project ngay, đặc biệt là trong công ty, tôi cho rằng đây không phải lựa chọn phù hợp.** Nếu nền tảng Java hoặc Spring Boot của bạn chưa tốt, nên chủ động học trước rồi mới bắt đầu xem video hoặc làm project bằng các cách khác. **Ngoài ra, tôi không hiểu tại sao mọi người đều nói vừa làm project vừa học sẽ cho hiệu quả tốt nhất. Theo tôi, điều này phải có một tiền đề: bạn đã hiểu cơ bản về công nghệ đó, hoặc đã có hiểu biết nhất định về lập trình.**

**Trọng tâm!!! Khi nền tảng của bản thân chưa vững, chỉ đơn thuần làm theo video thì hoàn toàn không có tác dụng. Bạn sẽ nhận ra rằng sau khi xem xong video, đến lúc tự viết code thì lại không biết viết.**

Không biết programmer ở các công ty khác như thế nào? Tôi cảm thấy việc tích lũy kỹ thuật phụ thuộc rất nhiều vào ngày thường. Chỉ dựa vào công việc thì trong phần lớn trường hợp chỉ giúp bạn thành thạo hơn trong việc hoàn thành requirement; tất nhiên, viết nhiều ít nhiều cũng sẽ nâng cao nhận thức của bạn về chất lượng code (với điều kiện bạn có ý thức này).

Ngoài giờ làm, tôi tận dụng thời gian rảnh để học những điều mình muốn học. Một ví dụ trong công việc là project đầu tiên khi mới vào công ty đã sử dụng Spring Security + JWT. Vì lúc đó chưa hiểu rõ công nghệ này, tôi đã dành khoảng một tuần ngoài giờ làm để học, viết một Demo và chia sẻ lên GitHub, địa chỉ GitHub: <https://github.com/Snailclimb/spring-security-jwt-guide>. Nhân dịp này, tôi cũng đã chia sẻ

- [《Một câu hỏi giúp bạn phân biệt rõ Authentication, Authorization và Cookie, Session, Token》](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485626&idx=1&sn=3247aa9000693dd692de8a04ccffeec1&chksm=cea24771f9d5ce675ea0203633a95b68bfe412dc6a9d05f22d221161147b76161d1b470d54b3&token=684071313&lang=zh_CN&scene=21#wechat_redirect)
- [Phân tích ưu nhược điểm của JWT authentication và giải pháp cho các vấn đề thường gặp](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485655&idx=1&sn=583eeeb081ea21a8ec6347c72aa223d6&chksm=cea2471cf9d5ce0aa135f2fb9aa32d98ebb3338292beaccc1aae43d1178b16c0125eb4139ca4&token=1737409938&lang=zh_CN#rd)

Một ví dụ gần đây khác là trong thời gian ở nhà vì dịch viêm phổi, tôi đã tự học Kafka và đang chuẩn bị viết một loạt bài nhập môn. Hiện tôi đã hoàn thành:

1. Kafka nhập môn theo cách dễ hiểu;
2. Cài đặt Kafka và trải nghiệm các chức năng cơ bản;
3. Spring Boot tích hợp Kafka để gửi và nhận message;
4. Một số nội dung về transaction, xử lý message lỗi... khi Spring Boot tích hợp Kafka để gửi và nhận message.

Chưa hoàn thành:

1. Phân tích các tính năng nâng cao của Kafka, chẳng hạn như workflow, vì sao Kafka nhanh...;
2. Đọc và phân tích source code;
3. ……

**Vì vậy, tôi cho rằng việc tích lũy và lắng đọng kỹ thuật phụ thuộc rất nhiều vào thời gian ngoài giờ làm (không tính những người rất giỏi và một số chuyên gia).**

**Chặng đường phía trước còn rất dài. Dù có nhiều năng lượng đến đâu, bạn cũng không thể học hết mọi công nghệ mình muốn; hãy biết lựa chọn, biết thỏa hiệp và dành thời gian giải trí phù hợp.**
