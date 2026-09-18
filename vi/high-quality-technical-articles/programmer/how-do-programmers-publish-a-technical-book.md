---
title: Làm thế nào lập trình viên xuất bản một cuốn sách kỹ thuật
description: "Làm thế nào lập trình viên xuất bản một cuốn sách kỹ thuật: hệ thống hóa các khái niệm then chốt, vấn đề thường gặp và điểm thực hành xoay quanh kiến thức kỹ thuật và tổng hợp phỏng vấn, giúp bạn học tập hiệu quả và chuẩn bị phỏng vấn."
category: Tuyển tập bài viết kỹ thuật chọn lọc
author: hsm_computer
tag:
  - Lập trình viên
head:
  - - meta
    - name: keywords
      content: lập trình viên xuất bản sách,sách kỹ thuật xuất bản,hợp tác với nhà xuất bản,công ty sách,kỹ năng viết sách,thu nhập nhuận bút,viết kỹ thuật,sách bán chạy
---

> **Lời giới thiệu**: Giới thiệu chi tiết cách lập trình viên nên bắt đầu từ con số 0 để xuất bản cuốn sách của riêng mình.
>
> **Địa chỉ bài gốc**: <https://www.cnblogs.com/JavaArchitect/p/12195219.html>

Khi phỏng vấn hoặc tìm hiểu công việc phụ, nếu có thể thuyết phục người khác bằng cách chứng minh năng lực của mình thì rất có thể sẽ đạt hiệu quả gấp đôi với công sức chỉ bằng một nửa. Làm thế nào để chứng minh năng lực? Thuyết phục nhất chắc chắn là nền tảng làm việc tại công ty lớn, chẳng hạn đảm nhiệm vai trò architect cấp cao tại BAT, khi đó hầu như không cần nói thêm gì nữa.

Tuy nhiên, không phải ai sau khi đi làm cũng lập tức trở thành architect của công ty lớn. Trên con đường tiến bộ, bạn còn có thể chứng minh bản thân thông qua tài khoản công khai, bài viết chuyên mục, số lượng code trên GitHub, xuất bản sách và quay video. So với những cách khác, sách kỹ thuật của riêng bạn được nhà xuất bản cấp quốc gia bảo chứng nên dễ khiến người khác công nhận năng lực hơn; với một số công ty nhỏ, thậm chí cuốn sách của riêng bạn có thể được xem như tấm vé miễn phỏng vấn. Vì vậy, trong bài viết này, tôi sẽ trao đổi với các bạn lập trình viên về những vấn đề liên quan đến việc xuất bản sách kỹ thuật.

## 1. Không phải đợi có năng lực rồi mới viết sách, mà nâng cao năng lực trong quá trình viết sách

Tôi biết không ít bạn bè đã xuất bản cuốn sách đầu tiên trong vòng 3 năm đi làm; một số người xuất sắc thậm chí đã xuất bản sách khi còn đi học.

Trái lại, không ít bạn có thể nghĩ rằng phải đợi tích lũy kỹ thuật đến một mức nhất định rồi mới viết. Thực ra, suy nghĩ này có thể chưa đủ tích cực: vừa viết sách vừa nâng cao kỹ thuật, đồng thời cuốn sách viết ra còn có thể giúp ích cho người khác, hoàn toàn là điều có thể làm được.

Ví dụ, nếu muốn tìm hiểu sâu về data analysis và machine learning bằng Python, bạn có thể học có hệ thống rồi tổng hợp các case về web crawler, data analysis và machine learning đã học trước đó. Dựa trên cách hiểu của mình, hãy trình bày lại theo cách phù hợp với người mới bắt đầu, như vậy là có thể xuất bản thành sách. Loại sách này có thể chưa giúp ích nhiều cho người có kinh nghiệm, nhưng vì có các case nên chắc chắn hữu ích với độc giả mới nhập môn, bởi đây là cách chia sẻ từ trải nghiệm thực tế. Hơn nữa, nếu không có động lực xuất bản sách, quá trình học có thể chỉ dừng ở mức biết sơ qua hoặc không thể toàn tâm học tập; có mục tiêu xuất bản sách sẽ giúp đảm bảo hiệu quả học tập hơn.

## 2. Sách phù hợp với junior developer, senior developer và architect

Như đã đề cập, junior developer phù hợp với sách hướng dẫn qua case. Lấy chủ đề web crawler, data analysis và machine learning bằng Python làm ví dụ: trước tiên có thể tìm vài cuốn sách có sẵn về lĩnh vực này. Có thể nội dung hoặc chương trong mỗi cuốn khác nhau, nhưng nếu đọc kết hợp thì chúng có thể bao quát nội dung của lĩnh vực này. Sau đó tham khảo cách triển khai của sách khác, chẳng hạn một chương viết về web crawler, một chương viết về pandas, một chương viết về matplotlib; khi tổng hợp lại, bạn có thể dùng một số chương để tạo thành một cuốn sách. Tóm lại, sách của người khác có nội dung gì thì không được sao chép, nhưng có thể tham khảo họ viết những điểm kỹ thuật nào.

Sau khi xác định các chương, hãy xác định tiếp các mục trong từng chương. Ví dụ, chương 3 nói về case web crawler, có thể quy định 3.1 nói về khái niệm web crawler, 3.2 nói về cách xây dựng thư viện Scrapy, 3.3 nói về cách phát triển case web crawler bằng Scrapy. Xác định từ chương đến mục theo thứ tự này là có thể định hình khung của một cuốn sách. Vì là sách hướng dẫn qua case, trước tiên cần cung cấp code chạy được, sau đó dùng các case code này để hướng dẫn người khác nhập môn. Case không nhất thiết phải quá sâu, nhưng phải đủ để người mới bắt đầu đọc là hiểu. Sau khi học từng bước theo hệ thống kiến thức bạn đưa ra, họ phải có thể hiểu nội dung chủ đề đó. Đồng thời, sau khi đọc sách, họ có thể chạy được các case web crawler, machine learning do bạn cung cấp, nắm được kiến thức trong lĩnh vực này và thực hiện công việc phát triển cơ bản. Với junior developer, chỉ cần dành chút tâm sức và thời gian thì mục tiêu này không khó đạt được.

Còn với senior developer và architect, ngoài việc viết sách thuần túy về case, bạn còn có thể đưa vào sách kinh nghiệm phát triển đã tổng kết tại các công ty lớn, tức những vấn đề từng gặp. Chẳng hạn khi dùng matplotlib trong Python để hiển thị chú giải, có những kỹ thuật nào khi thiết lập trục tọa độ, thường gặp vấn đề gì trong quá trình thiết lập. Nếu sách chứa nhiều kinh nghiệm như vậy thì giá trị sẽ cao hơn.

Ngoài ra, senior developer và architect còn có thể viết những cuốn sách có hàm lượng kỹ thuật cao hơn, chẳng hạn kinh nghiệm thực tiễn trong các tình huống high concurrency hoặc kinh nghiệm dùng k8s+docker để ứng phó với high concurrency. Trong sách, bạn có thể đưa ra code, đồng thời trình bày phương án triển khai và kỹ thuật triển khai architecture, chẳng hạn cache nên được lựa chọn thế nào trong tình huống high concurrency, làm thế nào để tránh cache breakdown, cache avalanche, làm thế nào để kiểm tra sự cố redis trên production và thiết kế phương án ứng phó sự cố. Ngoài hướng này, bạn còn có thể đi sâu vào chi tiết, chẳng hạn thông qua source code bên trong của dubbo để cho mọi người biết cách cấu hình dubbo hiệu quả và cách kiểm tra khi xảy ra sự cố. Nếu architect hoặc senior developer có một cuốn sách như vậy làm bảo chứng, cộng thêm kinh nghiệm làm việc tại công ty lớn, họ càng có thể tạo dựng danh tiếng cho mình.

## 3. Có thể liên hệ trực tiếp với nhà xuất bản hoặc tìm công ty xuất bản

Trong bài blog [Những vấn đề về công việc phụ của lập trình viên: Trao đổi về xuất bản sách và quay video](https://www.cnblogs.com/JavaArchitect/p/11616906.html), tôi đã trình bày sự khác biệt giữa xuất bản sách thông qua nhà xuất bản và thông qua công ty sách để mọi người tham khảo. Sau khi đọc, mọi người có thể tự quyết định cách xuất bản.

Tuy nhiên, dù chọn cách nào, trước khi xuất bản sách bạn cũng phải làm rõ một số việc; có thể nhân viên của một số công ty xuất bản sách sẽ không chủ động nói, nên bạn cần tự hỏi cho rõ.

- Đối tác hợp tác với bạn là ai? Công ty xuất bản sách hay nhà xuất bản?
- Sách của bạn sẽ được xuất bản ở nhà xuất bản nào? Những nhà xuất bản khá nổi tiếng trong nước là Tsinghua, Posts & Telecom, Electronic Industry và Machinery Industry. Không thể nói các nhà xuất bản khác không tốt, nhưng trong ngành bốn nhà xuất bản này được công nhận nhiều hơn.
- Người trao đổi với bạn là biên tập viên sách có quyền quyết định cuối cùng hay nhân viên của công ty sách? Nói thêm một lần nữa, người có thể quyết định sách có được xuất bản hay không và xác định ý kiến sửa đổi là biên tập viên của nhà xuất bản.

Sau khi so sánh nhà xuất bản và công ty xuất bản sách, đồng thời làm rõ nhiều chi tiết, mọi người có thể tự cân nhắc cách hợp tác. Hơn nữa, thông tin liên hệ của nhà xuất bản và công ty sách đều có trên website chính thức, mọi người có thể tự liên hệ qua email hoặc các cách khác.

## 4. Nếu bị người khác dùng làm đối tượng thử nghiệm hoặc bị thiếu tôn trọng, hãy nhanh chóng giảm thiệt hại

Trước đây tôi từng thấy một công ty xuất bản sách tuyển tác giả cho sách dành cho người mới học Java, đồng thời cũng chủ động liên hệ với nhân viên liên quan; phần lớn phản hồi nhận được là: “Phải viết lại”.

Ví dụ, tôi gửi đề cương đã lập và nhận phản hồi là “phải viết lại”, vì đối phương chưa học Java nhưng sau khi xem đề cương của tôi với tư cách người không có nền tảng, họ thấy không thể học được. Còn phải viết lại thành thế nào thì đối phương cũng không nói rõ được. Tóm lại, họ yêu cầu tôi đưa một đề cương khác; sau khi đưa thêm một bản, kết quả vẫn không đạt. Lần này còn đỡ hơn: họ đưa cho tôi đề cương của vài cuốn sách tương tự để tôi tự xem người khác có điểm gì hay. Tóm lại, họ không đưa ra hoặc không thể đưa ra điểm cần cải thiện cụ thể, mà yêu cầu tôi tự thử nhiều cách cải thiện khác nhau cho đến khi đối phương cảm thấy được.

So với khi trao đổi với các biên tập viên chuyên nghiệp của nhà xuất bản, dù đề cương hay bản thảo có vấn đề, họ vẫn chỉ rõ vấn đề nằm ở đâu và đưa ra ý kiến sửa đổi cụ thể. Tôi không biết cơ cấu tổ chức trong công ty sách ra sao, nhưng nhà xuất bản có bộ phận chuyên trách sách máy tính và các biên tập viên chuyên trách; ý kiến họ đưa ra khá chuyên nghiệp và rất dễ áp dụng khi sửa đổi.

Ngoài ra, tôi thỉnh thoảng thấy nhân viên của các công ty xuất bản sách trên nhiều kênh đăng bản thảo do người khác giao, rồi công khai chỉ ra vấn đề trong đó để mọi người rút kinh nghiệm. Tạm không bàn về động cơ của việc này, và dù nhân viên đó đã che thông tin có thể tiết lộ danh tính tác giả, nhưng tác giả giao bản thảo cho bạn vì tin tưởng; tự ý công khai bản thảo khi chưa được tác giả đồng ý, nói rằng “không coi tác giả ra gì” cũng không phải quá lời. Nếu không, hoàn toàn có thể trao đổi riêng với tác giả thay vì công khai lỗi vô ý của họ trước mọi người.

Khi hợp tác với nhà xuất bản, tôi chưa từng gặp chuyện như vậy; các biên tập viên nhà xuất bản mà tôi biết đều dành cho tác giả sự tôn trọng đầy đủ. Hơn nữa, khi trao đổi với bạn bè và nhiều người bạn làm tại các công ty xuất bản sách, tôi cũng nhận được sự tôn trọng và đối đãi lịch sự. Vì vậy, khi viết sách, đặc biệt là cuốn đầu tiên, nếu gặp tình huống bị dùng làm đối tượng thử nghiệm hoặc cảm thấy qua lời nói và những khía cạnh khác rằng đối phương không coi trọng mình, bạn có thể lập tức giảm thiệt hại. Thực ra cũng không có “thiệt hại” gì: khi trao đổi đề cương và bản thảo hiện tại với biên tập viên nhà xuất bản, lợi ích của bạn có khi còn tăng lên.

## 5. Viết một chương dài 30 trang như thế nào?

Sau khi thống nhất hợp đồng viết với nhà xuất bản, bạn có thể bắt đầu sáng tác. Sách được cấu thành từ các chương; ở đây hãy nói về cách hình thành ý tưởng và viết một chương.

Ví dụ, chương về web crawler dài khoảng 30 trang. Trước tiên xác định các mục và tiểu mục: 3.1 xây dựng môi trường web crawler là một tiểu mục, còn 3.1.1 tải package Python Scrapy là một mục. Trước hết xác định nội dung cần viết; cụ thể với chương web crawler, có thể viết 3.1 xây dựng môi trường, 3.2 các module quan trọng của Scrapy, 3.3 cách phát triển web crawler bằng Scrapy, 3.4 cách chạy sau khi phát triển xong, 3.5 cách đưa thông tin đã crawl vào database. Tất cả đều là các tiểu mục.

Cụ thể hơn về các mục, trong 3.5, mục 3.5.1 viết cách xây dựng môi trường database, 3.5.2 viết cách kết nối database trong Scrapy, 3.5.3 đưa ra case thực tế, 3.5.4 đưa ra các bước chạy và hiệu quả minh họa.

Như vậy có thể xây dựng khung của một chương. Trong mỗi tiểu mục, trước tiên đưa ra code chạy được và có thể làm rõ vấn đề, sau đó giải thích code, viết cách cấu hình code và những vấn đề cần chú ý khi phát triển, khi cần thì dùng bảng và hình để minh họa. Theo cách trình bày này, nhiều nhất 3 tuần có thể hoàn thành một chương; nếu nhanh thì một chương chỉ mất một tuần rưỡi.

Tương tự, một cuốn sách có khoảng 12 chương. Chương đầu tiên có thể nói về cách cài đặt môi trường và cú pháp cơ bản; các chương sau đi từ nông đến sâu, mỗi chương một chủ đề. Ví dụ với sách về Python web crawler, chương hai có thể nói về cú pháp cơ bản, chương ba nói về giao thức http và các điểm kiến thức về web crawler, rồi đi sâu hơn để trình bày đầy đủ các kỹ năng về web crawler, data analysis, data visualization và machine learning.

Tính như vậy, nếu viết cuốn sách đầu tiên, trung bình mỗi tháng hoàn thành 2 chương thì khoảng nửa năm đến 8 tháng có thể hoàn thành một cuốn sách. Cách làm là trước tiên xây dựng hệ thống kiến thức của sách, khi viết từng chương thì xây dựng khung cho một điểm kiến thức; trong các tiểu mục và mục, kết hợp code với giải thích, đi từ đơn giản đến phức tạp, như vậy mọi người có thể hoàn thành cuốn sách đầu tiên của riêng mình.

## 6. Làm thế nào viết một cuốn sách bán được hơn 5.000 bản?

Hiện nay sách giấy thường in 2.500 bản mỗi lần; phần lớn sách chỉ in một lần rồi bán hết. Nếu bán được 5.000 bản thì được xem là được đón nhận; nếu vượt 10.000 bản thì có thể gọi là sách cấp độ “đại thần”. Ở đây tạm không bàn về sách cấp độ “đại thần”, mà chỉ nói về cách viết một cuốn sách bán được hơn 5.000 bản.

1. Tốt nhất nên bám sát xu hướng, chẳng hạn xu hướng hiện nay là full-stack development và machine learning. Muốn tìm xu hướng, hãy đến những nơi như JD.com để xem keyword của các sách bán chạy. Khi thực hiện cụ thể, hãy trao đổi nhiều với biên tập viên nhà xuất bản; tác giả có thể phân tích vấn đề chủ yếu từ góc độ kỹ thuật, còn biên tập viên nhà xuất bản sẽ cân nhắc từ góc độ thị trường.

2. Nếu sách của bạn có thể được tổ chức đào tạo dùng làm giáo trình thì muốn không nổi tiếng cũng khó. Tổ chức đào tạo thường dùng giáo trình như thế nào? Thứ nhất là hướng đến người mới bắt đầu, thứ hai là code đầy đủ, thứ ba là bao quát toàn bộ điểm kiến thức trong lĩnh vực đó. Để đạt được điều này, mọi người có thể trao đổi trực tiếp với biên tập viên nhà xuất bản và hỏi các chi tiết liên quan.

3. Có thể dùng văn phong sinh động, nhưng không được dùng lời văn quá hoa mỹ để che giấu việc nội dung thiếu chiều sâu. Nói cách khác, sách bán chạy nhất định phải có nội dung thiết thực và giải quyết được vấn đề thực tế của người mới bắt đầu. Ví dụ với machine learning bằng Python, hãy viết một cuốn sách dùng case để bao quát các algorithm machine learning thường dùng hiện nay, mỗi chương một algorithm; trong case có các yếu tố như data visualization, data analysis và web crawler. Nếu hiệu quả trực quan còn hấp dẫn thì khả năng cuốn sách bán chạy sẽ rất cao.

4. Tuyệt đối không được làm qua loa. Code chạy được chưa đủ, còn phải cố gắng làm cho code ngắn gọn; phần giải thích nên hướng nhiều đến độc giả. Về nội dung, cần đảm bảo độc giả đọc là hiểu và có thu hoạch. Điểm này có thể khá trừu tượng, nhưng sau khi tự mình viết vài cuốn sách, tôi thực sự cảm nhận rằng làm được rất khó. Tuy nhiên, một khi làm được thì dù sách không bán chạy, ít nhất cũng không làm hại người đọc.

## 7. Tổng kết: Xuất bản sách chỉ là một cột mốc, lập trình viên nên không ngừng tiến bộ

Xuất bản sách không đơn giản, vì không phải ai cũng sẵn sàng dành thời gian và công sức viết sách vào mỗi tối và mỗi cuối tuần trong suốt nửa năm đến 8 tháng. Nhưng xuất bản sách cũng không khó; suy cho cùng, khi đã dành thời gian thì xuất bản sách cũng chỉ là công việc debug code và viết nội dung, nhiều nhất thêm chi phí trao đổi với người khác.

Thực ra thu nhập từ việc xuất bản sách không cao, tính ra mỗi tháng khoảng 3k. Nếu hợp tác với công ty xuất bản sách thì có lẽ còn ít hơn, nhưng dù sao việc này cũng có thể chứng minh năng lực của mình. Tuy nhiên, sau khi xuất bản sách không thể dừng lại ở đó, vì trong các công ty lớn có rất nhiều người giỏi, thậm chí họ không cần dựa vào việc xuất bản sách để chứng minh năng lực.

Vậy làm thế nào để tối đa hóa lợi ích do việc xuất bản sách mang lại? Thứ nhất, bạn có thể dựa vào đó để vào công ty lớn; khi phỏng vấn, có sách của riêng mình chắc chắn là một điểm cộng. Thứ hai, có thể dùng điều này để mở chuyên mục trên các website lớn, quay video hoặc mở tài khoản công khai; dù sao có sự bảo chứng của nhà xuất bản thì người khác sẽ càng tin tưởng năng lực của bạn hơn. Thứ ba, cần tiếp tục dựa vào phương pháp học tập và tinh thần tiến bộ tích lũy khi viết sách để nghiên cứu những kỹ thuật chuyên sâu hơn. Khi có kỹ thuật, bạn không chỉ có thể kiếm nhiều tiền hơn ở công ty lớn mà còn có thể kiếm tiền hiệu quả hơn thông qua các hình thức như đào tạo doanh nghiệp.

<!-- @include: @article-footer.snippet.md -->
