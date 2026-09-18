---
title: Bảy lời khuyên dành cho developer muốn phát triển lên cấp độ cao hơn
description: "Bảy lời khuyên dành cho developer muốn phát triển lên cấp độ cao hơn: hệ thống hóa các khái niệm then chốt, vấn đề thường gặp và điểm thực hành xoay quanh kiến thức kỹ thuật và tổng hợp phỏng vấn, giúp bạn học tập và chuẩn bị phỏng vấn hiệu quả."
category: Tuyển tập bài viết kỹ thuật chọn lọc
author: Kaito
tag:
  - Bí kíp nâng cấp
head:
  - - meta
    - name: keywords
      content: developer phát triển,developer cấp cao,review yêu cầu,nền tảng kỹ thuật,tối ưu performance,xử lý sự cố production,tổng hợp,phát triển sự nghiệp
---

> **Lời giới thiệu**: Nếu một programmer bình thường muốn phát triển thành senior programmer, thậm chí expert hoặc cấp độ cao hơn, cần chú ý tăng cường những mặt nào? Anh Phi, chủ tài khoản công khai “Rèn luyện nền tảng kỹ thuật cho developer”, đã đưa ra bảy lời khuyên thiết thực trong bài viết này.
>
> **Tổng quan nội dung**:
>
> 1. Chủ động tăng cường năng lực review yêu cầu
> 2. Chủ động suy nghĩ về efficiency
> 3. Tăng cường nền tảng kỹ thuật
> 4. Suy nghĩ về performance
> 5. Coi trọng production
> 6. Quan tâm đến bức tranh tổng thể
> 7. Năng lực tổng hợp và đúc kết
>
> **Địa chỉ bài viết gốc**: <https://mp.weixin.qq.com/s/8lMGzBzXine-NAsqEaIE4g>

### Lời khuyên 1: Chủ động tăng cường năng lực review yêu cầu

Trước hết, hãy bắt đầu từ việc review yêu cầu. Trong các công ty Internet, review yêu cầu là điểm bắt đầu chính của công việc phát triển.

Đối với programmer phổ thông, thông thường họ sẽ dựa trên chi tiết yêu cầu do product manager đưa ra để bắt đầu suy nghĩ xem nên triển khai chức năng này thế nào, chi phí phát triển ước tính mất bao lâu. Họ tự xem mình là người phiên dịch giữa yêu cầu và code. Họ hiếm khi suy nghĩ về tính hợp lý của yêu cầu hay giá trị của việc mình làm, cũng không quan tâm hoặc đặt câu hỏi.

Còn với programmer cấp cao, họ sẽ không ngay từ đầu sa vào chi tiết mà sẽ xuất phát nhiều hơn từ bản thân sản phẩm, hỏi product manager tại sao phải làm chi tiết này, mục đích là gì. Nói cách khác, họ sẽ xem xét trước xem yêu cầu đó có hợp lý hay không.

Nếu yêu cầu thực sự không hợp lý thì sẽ tiến hành PK: hoặc điều chỉnh yêu cầu, hoặc loại bỏ nó. Tuy nhiên, cần lưu ý rằng PK và điều chỉnh yêu cầu không chỉ là cắt giảm yêu cầu, mà còn có một hướng khác, đó là tăng cường yêu cầu.

Do thiếu nền tảng kỹ thuật, các bạn product rất có thể chưa suy nghĩ đủ toàn diện. Khi đó, nếu bạn có ý tưởng tốt hơn, hoàn toàn có thể đề xuất và thêm vào yêu cầu để làm cho yêu cầu trở nên có giá trị hơn.

Tóm lại, programmer cấp cao sẽ không nhất nhất phát triển theo tài liệu yêu cầu của product manager, mà **luôn xuất phát từ góc độ có lợi cho business để suy nghĩ, rồi loại bỏ, sửa đổi hoặc bổ sung yêu cầu của product manager.**

Công việc này thoạt nhìn có vẻ không liên quan đến phát triển, nhưng chỉ như vậy mới bảo đảm mọi công việc phát triển tiếp theo đều có giá trị, thay vì làm một đống việc vô ích. Làm quá nhiều việc vô ích sẽ làm giảm mạnh cảm giác thành tựu của developer.

Vì vậy, **programmer phổ thông muốn phát triển thành developer cấp cao hơn nhất định phải tăng cường năng lực review yêu cầu**.

### Lời khuyên 2: Chủ động suy nghĩ về efficiency

Programmer phổ thông thường viết code theo từng bước: có việc thì làm, không có việc thì ngồi chờ. Họ hiếm khi suy nghĩ sâu xem tại sao code hiện tại lại được viết như vậy, cách viết này có lợi ích gì, chỗ nào tồn tại bottleneck, và liệu mình có thể tối ưu nó thêm không.

Còn programmer cao cấp hơn sẽ không giới hạn mình ở việc phát triển xong công việc đang có. Họ sẽ chủ động suy nghĩ xem phương thức phát triển hiện tại có chưa đủ tốt hay không. Vậy mình có thể làm gì đó để nâng cao efficiency hay không?

Ví dụ nhỏ, sáu năm trước khi tiếp nhận một project, tôi phát hiện mỗi tháng nhân viên vận hành tìm tôi bốn lần, nhờ tôi gửi một push notification. Cô ấy nói các developer trước đây đều hỗ trợ như vậy. Xử lý yêu cầu này rất đơn giản, chỉ cần sửa hai dòng rồi deploy là xong. Nhưng thật phiền, hãy thử tưởng tượng bạn đang tập trung viết code thì cô ấy lại đến tìm bạn, khiến toàn bộ mạch suy nghĩ bị gián đoạn. Hơn nữa, thao tác thường xuyên trên production vốn đã đưa vào những rủi ro không chắc chắn; nếu một ngày nào đó lỡ tay thao tác sai thì production sẽ gặp sự cố.

Cách làm của tôi là dành riêng một tuần để xây dựng cho cô ấy một operations backend. Từ đó, mọi push notification đều có thể được cô ấy thao tác trực tiếp trên backend. Tôi cũng có thêm thời gian để làm những việc có giá trị hơn.

Vì vậy, **lời khuyên thứ hai là hãy chủ động suy nghĩ xem trong công việc hiện tại, chỗ nào còn có thể cải thiện efficiency; khi đã nghĩ ra thì chủ động cải tiến nó!**

### Lời khuyên 3: Tăng cường nền tảng kỹ thuật

Nền tảng kỹ thuật gồm những gì? Tôi nghĩ độc giả của “Rèn luyện nền tảng kỹ thuật cho developer” chắc hẳn rất quen thuộc: đó chính là những kiến thức cơ bản như operating system, network mà mọi người từng học ở trường.

Programmer phổ thông sẽ nghĩ: những kiến thức cơ bản này tôi biết hết rồi, dù sao tôi cũng đã học hẳn bốn năm đại học. Sau khi đi làm, họ sẽ không chủ động quay lại để nâng cao hiểu biết chuyên sâu về những nền tảng này.

Programmer cấp cao hiểu rất rõ chút kiến thức mình học năm xưa còn quá hời hợt. Ngoài giờ làm, họ cũng sẽ nghiên cứu sâu về các hướng như Linux và cách triển khai bên trong của network.

Trên thực tế, các chuyên gia kỹ thuật trong ngành Internet phần lớn trở thành chuyên gia là nhờ có hiểu biết rất sâu về những nền tảng này. Chính nền tảng kỹ thuật vững chắc đã thúc đẩy họ phát triển thành chuyên gia.

Tôi khó tin rằng một developer không hiểu tầng bên trong, chỉ biết CURD, chỉ biết dùng framework của người khác, sau này có thể phát triển thành chuyên gia theo hướng kỹ thuật.

Vì vậy, **cũng nên thường xuyên rèn luyện nền tảng kỹ thuật ở tầng bên trong**. Nếu bạn không biết rèn luyện thế nào, hãy kiên trì theo dõi tài khoản công khai “Rèn luyện nền tảng kỹ thuật cho developer”.

### Lời khuyên 4: Suy nghĩ về performance

Programmer phổ thông thường phát triển xong yêu cầu là không quan tâm nữa; chỉ cần yêu cầu được triển khai, test đã thông qua thì có thể bàn giao. Họ chưa từng nghĩ sau này traffic sẽ lớn đến đâu, cũng không biết service của mình có thể hỗ trợ bao nhiêu QPS.

Còn programmer cấp cao thường quan tâm đến performance của code mình viết.

Khi review yêu cầu, họ thường ước tính sơ bộ traffic request sẽ lớn đến đâu. Sau đó, ngay ở giai đoạn design, họ sẽ dựa trên quy mô này để thiết kế solution đáp ứng yêu cầu performance.

Trước khi deploy, họ cũng sẽ tiến hành performance test để kiểm tra performance có đạt kỳ vọng hay không. Nếu có vấn đề về performance thì bottleneck nằm ở đâu, có thể tối ưu thế nào.

Vì vậy, **lời khuyên thứ tư là nhất định phải chủ động quan tâm nhiều hơn đến performance của business mình phụ trách, đồng thời thường xuyên tối ưu và cải tiến**. Tôi cho rằng lời khuyên này cực kỳ quan trọng. Nhưng để làm được điều đó, bạn cần có nền tảng kỹ thuật vững chắc; nếu ngay cả cách network hoạt động bạn cũng không rõ thì nói gì đến tối ưu!

### Lời khuyên 5: Coi trọng production

Programmer phổ thông thường ít quan tâm đến những việc trên production. Trong danh sách server mình nắm chỉ có máy dev và máy deploy; có bao nhiêu máy production, traffic bao nhiêu, gần đây có dao động hay không, có thể họ đều không rõ.

Còn programmer cấp cao hiểu sâu sắc rằng nếu có điều kiện thì nên thường xuyên quan sát service production của mình, xem code đang chạy thế nào, có error log nào không. Khi request đạt peak, CPU và memory tiêu thụ ra sao. Tình hình tiêu thụ network port thế nào, có cần điều chỉnh một số tham số cấu hình hay không.

Khi performance không được như mong đợi, họ có thể quay lại suy nghĩ về solution cải thiện performance, rồi phát triển và deploy lại.

Bạn sẽ nhận ra rằng khi có sự cố trên production, những người khẩn cấp ra tuyến đầu xử lý đều là các programmer cấp cao hơn.

Vì vậy, **lời khuyên thứ năm của anh Phi là hãy thường xuyên quan sát tình hình vận hành trên production**. Chỉ khi thường xuyên quan tâm đến production, bạn mới có thể đảm nhận trọng trách nhanh chóng xử lý sự cố production khi xảy ra.

### Lời khuyên 6: Quan tâm đến bức tranh tổng thể

Programmer phổ thông được phân cho module nào thì làm module đó, tự đặt ra một ranh giới rất nhỏ cho công việc của mình, mọi ánh nhìn đều tập trung trong chiếc khung nhỏ ấy.

Programmer cấp cao sẽ làm quen và tìm hiểu tất cả module project trong team, dù đó không phải phần mình phụ trách. Những người có tư duy này thường phát triển nhanh nhất cả về kỹ thuật lẫn business. Những người được thăng cấp hoặc đề bạt vào vị trí cao hơn thường là kiểu người này.

Thậm chí, một số người ở cấp độ cao hơn không chỉ giới hạn tầm nhìn trong team mà còn quan tâm đến business và technology stack của các team khác trong công ty, thậm chí cả trong ngành. Viết đến đây, tôi nhớ đến câu Trương Nhất Minh từng nói: đừng đặt ranh giới cho công việc của mình.

Vì vậy, **bạn nên có tư duy tổng thể; không chỉ module mình phụ trách, mà thực ra nên quan tâm đến toàn bộ project**. Đừng đến việc các thành viên trong chính group của mình đang làm gì cũng không biết.

### Lời khuyên 7: Năng lực tổng hợp và đúc kết

Programmer phổ thông thường làm xong việc là cho qua, hiếm khi quay lại tổng hợp và đúc kết về kỹ thuật hay business của mình.

Còn programmer cấp cao thường sẽ tổng kết sau khi hoàn thành một việc tương đối lớn, làm một file PPT, viết một bài blog hoặc ghi chép lại theo cách nào đó. Như vậy vừa là sự tổng hợp công việc của bản thân, vừa có thể chia sẻ với các thành viên khác, thúc đẩy team cùng phát triển.

<!-- @include: @article-footer.snippet.md -->
