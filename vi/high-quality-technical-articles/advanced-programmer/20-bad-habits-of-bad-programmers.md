---
title: 20 thói quen xấu của lập trình viên tồi
description: "20 thói quen xấu của lập trình viên tồi: tổng hợp các khái niệm then chốt, vấn đề thường gặp và điểm thực hành xoay quanh kiến thức kỹ thuật và phỏng vấn, giúp bạn học hiệu quả và chuẩn bị tốt cho phỏng vấn."
category: Tuyển tập bài viết kỹ thuật
author: Kaito
tag:
  - Bí kíp nâng cấp
head:
  - - meta
    - name: keywords
      content: thói quen xấu của lập trình viên,quy chuẩn lập trình,comment code,tài liệu kỹ thuật,làm việc nhóm,commit code,tố chất nghề nghiệp,tu dưỡng lập trình
---

> **Lời đề xuất**: Một bài viết của cao thủ Kaito, những lời khuyên rất thực tế!
>
> **Địa chỉ bài gốc:** <https://mp.weixin.qq.com/s/6hUU6SZsxGPWAIIByq93Rw>

Tôi tin chắc bạn từng gặp kiểu lập trình viên này: **dù viết code, viết tài liệu hay giao tiếp với người khác, họ đều tỏ ra rất chuyên nghiệp**. Mỗi lần gặp những người như vậy, tôi đều tự hỏi, rốt cuộc họ làm được thế nào?

Qua thời gian làm việc, dần dần tôi cũng tổng kết được một số kinh nghiệm: họ đều duy trì một số thói quen tốt tưởng như rất nhỏ, nhưng chính những thói quen đó thể hiện tố chất cơ bản của một lập trình viên giỏi.

Nhưng hôm nay chúng ta hãy đổi góc nhìn và xem một lập trình viên tồi có những thói quen xấu nào. Chỉ cần tránh được những vấn đề này, chúng ta có thể dần tiến gần hơn đến một lập trình viên giỏi.

## 1、Viết tên thuật ngữ kỹ thuật không chuẩn

Trong cả CV cá nhân lẫn tài liệu kỹ thuật, tôi thường thấy tên thuật ngữ kỹ thuật được viết không chuẩn, chẳng hạn như JAVA, javascript, python, MySql, Hbase, restful.

Cách viết đúng phải là Java, JavaScript, Python, MySQL, HBase, RESTful. Đừng xem nhẹ vấn đề này, rất có thể nhiều người phỏng vấn sẽ loại CV của bạn chỉ vì điểm này.

## 2、Viết tài liệu, trộn tiếng Trung và tiếng Anh không chuẩn

Dùng dấu câu tiếng Anh khi mô tả bằng tiếng Trung, dùng ký tự full-width cho tiếng Anh và chữ số, không chèn khoảng trắng giữa tiếng Trung với tiếng Anh và chữ số, v.v.

Nhiều người bỏ qua việc thêm một «khoảng trắng» giữa tiếng Trung với tiếng Anh và chữ số, nhưng cách trình bày sẽ dễ đọc hơn. Trước đây, bài viết của tôi luôn tuân theo những chi tiết này.

## 3、Không viết comment cho logic quan trọng, hoặc viết quá dài dòng

Nhiều lập trình viên không viết comment cho code logic phức tạp và quan trọng. Ngoài bản thân họ có thể đọc hiểu logic của code, người khác hoàn toàn không thể hiểu. Hoặc có viết comment nhưng lại quá dài dòng, không có logic.

Logic quan trọng không chỉ cần comment, mà còn phải ngắn gọn, rõ ràng. Code đơn giản đọc là hiểu ngay thì có thể không thêm comment.

## 4、Viết function phức tạp, dài dòng

Một function có hàng trăm dòng, một file có hàng nghìn dòng code, function phức tạp không được tách nhỏ, khiến code ngày càng khó bảo trì, cuối cùng không ai dám sửa.

Vẫn phải tuân thủ các design pattern cơ bản, chẳng hạn như Single Responsibility: một function chỉ làm một việc; Open-Closed Principle: mở cho mở rộng, đóng cho sửa đổi.

Nếu logic của function thực sự phức tạp thì ít nhất cũng phải bảo đảm logic chính đủ rõ ràng.

## 5、Không đọc tài liệu chính thức, chỉ đọc blog rác

Nhiều người gặp vấn đề nhưng không đọc tài liệu chính thức trước, mà lại thích tìm đọc các blog rác. Nội dung của những blog này sao chép lẫn nhau và đầy lỗi.

Thực ra tài liệu chính thức của nhiều phần mềm đã được viết rất tốt, các vấn đề thường gặp đều có thể tìm được lời giải. Đọc kỹ tài liệu chính thức tốt hơn đọc blog rác cả trăm lần, hãy hình thành thói quen đọc tài liệu chính thức.

## 6、Cổ xúy quan điểm kiến thức nền tảng vô dụng

Có người ngày nào cũng chạy theo các project open source và framework đổi mới từng ngày, nhưng lại không chịu dành thời gian đào sâu nguyên lý bên trong. Họ có thể giải quyết các vấn đề thường gặp, nhưng gặp vấn đề sâu hơn một chút thì lại bó tay.

Nhiều thiết kế system architecture cao cấp thực ra đều bắt nguồn từ tầng bên trong. Hãy thử nghĩ xem, những thứ như kiến trúc máy tính, operating system, network protocol đã trải qua bao nhiêu năm tiến hóa mới có hình dạng như hiện nay. Trong quá trình tiến hóa, vô số vấn đề phức tạp đã xuất hiện. Hiểu được cách giải quyết những vấn đề đó thì khi nhìn vào kỹ thuật tầng trên, mọi thứ sẽ trở nên rất đơn giản.

## 7、Thích khoe kỹ thuật

Có người ngày nào cũng treo những thuật ngữ kỹ thuật «cao siêu» bên miệng, sợ người khác không biết mình đã học kỹ thuật cao thâm nào. Họ thích khoe kỹ thuật bằng lời nói, nhưng người khác vừa hỏi chi tiết đã cứng họng.

## 8、Không chấp nhận chất vấn

Khi người khác đặt câu hỏi về solution do mình thiết kế, họ chỉ biết phản bác gay gắt thay vì bình tĩnh phân tích ưu nhược điểm và trao đổi với tâm thế học hỏi.

Những người này học được chút kiến thức đã cho rằng mình rất giỏi, nhưng không nhận ra chẳng qua là do hiểu biết của bản thân còn quá hạn hẹp.

## 9、Protocol API không chuẩn

Thống nhất protocol API với người khác chỉ bằng trao đổi miệng, không cung cấp tài liệu quy chuẩn để mô tả. Thậm chí đến lúc test tích hợp mới phát hiện protocol hoàn toàn khác với những gì đã thống nhất, hoặc đã sửa protocol nhưng không thông báo cho bên tích hợp, khiến trải nghiệm hợp tác rất tệ.

## 10、Gặp vấn đề là tự đâm đầu giải quyết

Đây là vấn đề mà các lập trình viên mới vào nghề rất dễ mắc phải: gặp vấn đề chỉ biết tự đâm đầu giải quyết, đến deadline vẫn không có kết quả, lãnh đạo hỏi mới biết có vấn đề không thể giải quyết.

Phản hồi kịp thời khi gặp vấn đề mới là có trách nhiệm với bản thân và với team.

## 11、Nói thì làm được, viết thì hỏng

Bình thường phương án kỹ thuật được nói hay như rót mật, nhưng hễ yêu cầu viết code thì lại hỏng, điển hình của kiểu mắt cao tay thấp.

## 12、Diễn đạt không có logic, không đứng ở góc độ đối phương

Thảo luận vấn đề nhưng không nói rõ bối cảnh, vừa mở đầu đã nói solution của mình, khiến người khác nghe mà chẳng hiểu gì. Đến khi yêu cầu mô tả lại từ đầu thì bản thân cũng không nói rõ được.

Học cách giao tiếp và diễn đạt là nền tảng của hợp tác.

## 13、Không chủ động suy nghĩ, chỉ biết chìa tay xin

Gặp vấn đề nhưng không tự google, không suy nghĩ đã hỏi người khác, thích làm người chỉ biết chìa tay xin.

Thời gian của mỗi người đều rất quý giá. Mọi người đều thích bạn mang theo suy nghĩ của mình khi đặt câu hỏi hơn. Làm vậy vừa tránh được nhiều vấn đề cấp thấp, vừa nâng cao chất lượng trao đổi.

## 14、Thường xuyên lặp lại lỗi

Sau khi xảy ra vấn đề thì nói lần sau sẽ chú ý, nhưng lần sau vẫn lặp lại vấn đề, đó là thiếu trách nhiệm với bản thân, nói cho cùng là vấn đề thái độ.

## 15、Thêm feature không tính đến khả năng mở rộng

Thêm feature mới chỉ tập trung vào một phần nghiệp vụ nhỏ, không tính đến khả năng mở rộng của toàn bộ system, dẫn đến tình trạng code chắp vá nghiêm trọng.

Hãy học cách phân tích requirement và những thay đổi có thể xảy ra trong tương lai, thiết kế solution tổng quát hơn và giảm cost phát triển về sau.

## 16、Không tự test API, có vấn đề không log

Không tự test API do mình phát triển mà đã phối hợp test với người khác, đến khi có vấn đề lại nói rằng không có log, khiến hiệu quả phối hợp cực kỳ thấp.

## 17、Commit code không chuẩn

Nhiều người commit code nhưng không viết mô tả, hoặc viết mô tả vô nghĩa. Đặc biệt khi chỉ sửa một lượng code nhỏ, việc này sẽ khiến cost truy ngược vấn đề tăng lên.

Đặt ra quy chuẩn commit code giúp bạn không sửa code quá tùy tiện trong mỗi lần commit.

## 18、Tự sửa database production

Kết nối trực tiếp đến database production để sửa dữ liệu, thậm chí có trường hợp quên điều kiện WHERE trong SQL UPDATE / DELETE, gây ra sự cố dữ liệu.

Khi sửa database production, nhất định phải hết sức thận trọng. Khuyến nghị trước khi thao tác hãy nhờ đồng nghiệp review code rồi mới thực hiện.

## 19、Chưa làm rõ requirement đã viết code

Nhiều lập trình viên nhận requirement xong không suy nghĩ nhiều đã bắt đầu viết code. Cách hiểu của họ lệch với requirement, dẫn đến phải làm lại vô ích.

Dành thêm thời gian làm rõ requirement có thể tránh được nhiều vấn đề bất hợp lý.

## 20、Thiết kế quan trọng không viết tài liệu

Thiết kế quan trọng không được ghi lại thành tài liệu, khi bàn giao system cho người khác chỉ mô tả bằng lời, khiến thông tin then chốt bị mất.

Đôi khi, để hiểu một solution design, đọc một tài liệu tốt còn hiệu quả hơn đọc vài trăm dòng code.

## Tổng kết

Bạn mắc bao nhiêu thói quen xấu trên? Hoặc xung quanh bạn có từng gặp người như vậy chưa?

Tôi cho rằng chủ động tránh những vấn đề này từ sớm là việc bắt buộc phải làm để trở thành một lập trình viên giỏi. Tổng hợp lại, những thói quen này chủ yếu thuộc 4 phương diện:

- Tố chất lập trình tốt
- Tâm thế học tập khiêm tốn
- Giao tiếp và diễn đạt tốt
- Chú trọng phối hợp trong team

Kỹ năng chuyên môn của một lập trình viên giỏi có thể rất khó học trong thời gian ngắn, nhưng những tố chất nghề nghiệp cơ bản này có thể rèn được trong thời gian ngắn.

Hy vọng bạn và tôi có lỗi thì sửa, không có lỗi thì lấy đó làm động lực.

<!-- @include: @article-footer.snippet.md -->
