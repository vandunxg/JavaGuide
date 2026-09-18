---
title: "Tổng kết bốn năm làm việc tại Tencent sau khi gia nhập qua tuyển dụng tại trường"
description: "Tổng kết bốn năm làm việc tại Tencent sau khi gia nhập qua tuyển dụng tại trường: hệ thống hóa các khái niệm then chốt, vấn đề thường gặp và điểm cần lưu ý trong thực tiễn xoay quanh kiến thức kỹ thuật và phỏng vấn, giúp bạn học tập và chuẩn bị phỏng vấn hiệu quả."
category: Bộ sưu tập bài viết kỹ thuật đặc sắc
author: pioneeryi
tag:
  - Kinh nghiệm cá nhân
head:
  - - meta
    - name: keywords
      content: Kinh nghiệm làm việc tại Tencent, tổng kết bốn năm, đánh giá hiệu suất, đo lường EPC, văn hóa phe cánh, phát triển nghề nghiệp, trưởng thành kỹ thuật, môi trường công sở Internet
---

Lập trình viên là một nghề có tính lưu động rất cao: thường xuyên có những gương mặt mới xuất hiện, cũng thường xuyên có những gương mặt cũ rời đi; có người chủ động rời đi, cũng có người bị buộc thôi việc.

Thêm vào đó, vài năm gần đây cạnh tranh ngày càng gay gắt, phải làm nhiều việc hơn nhưng nhận lại ít hơn, ngành Internet dường như cũng không còn hấp dẫn đến vậy.

Trạng thái người đến người đi, biến động không ngừng, thực ra tôi cũng đã quen từ lâu.

Lối thoát duy nhất của người đi làm không gì khác ngoài việc trau dồi kỹ năng chuyên môn, nâng cao năng lực cạnh tranh cốt lõi của bản thân. Như vậy, dù có thay đổi gì, đi đến đâu, bạn vẫn có thể tự nuôi sống mình.

Hôm nay tôi chia sẻ câu chuyện của một blogger đã gia nhập Tencent qua tuyển dụng tại trường, rồi rời đi sau bốn năm làm việc.

Còn về lý do rời đi, tôi cũng không rõ. Có thể là đã có lựa chọn tốt hơn, hoặc cảm thấy công việc hiện tại không còn giúp bản thân tiến bộ nhiều.

**“Tôi” trong phần dưới đây là chính tác giả bài viết.**

> Địa chỉ bài gốc: <https://zhuanlan.zhihu.com/p/602517682>

Sau khi tốt nghiệp cao học, tôi luôn làm việc tại Tencent, chẳng mấy chốc đã qua bốn năm. Bản thân tôi vốn không có thói quen chủ động tổng kết; trước đây chỉ mải chạy về phía trước mà quên dừng lại suy ngẫm và tổng kết. Tôi nhớ từng đọc một tài liệu quy hoạch nghề nghiệp nói rằng ba năm là một giai đoạn, năm năm là một giai đoạn. Hiện tại vừa đúng bốn năm, lại vừa rời Tencent, nên đã đến lúc làm một bản tổng kết.

Trước tiên, hãy đánh giá đơn giản về bốn năm này: cá nhân tôi cho rằng mình không hoàn toàn lãng phí hay phụ lòng quãng thời gian ấy. Vì sao lại nói vậy? Bởi tôi nhận ra việc so sánh với người khác dường như không có nhiều ý nghĩa: có rất nhiều người làm tốt hơn tôi, cũng không ít người làm kém hơn tôi. Nói cho cùng, tôi chỉ là một người hết sức bình thường, tài năng không nổi bật, kỹ năng không vượt trội. Hãy chấp nhận sự bình thường của bản thân, rồi chỉ cần xem những gì mình làm có khiến mình hài lòng hay không.

Sau đây tôi sẽ nói cụ thể vài điểm. Tôi chủ yếu muốn bàn về công việc, hiệu suất, EPC, cách nhìn về người trong phe thân cận, cuối cùng là những gì đã đạt được.

## Tình hình công việc

Tôi chưa từng chuyển vị trí trong Tencent, nhưng các project đã làm cũng khá đa dạng, gồm: BUGLY, distributed call chain (Huskie), hệ thống crowdsourcing (SOHO), hệ thống đo lường EPC. Một số project hướng ra bên ngoài, một số là hệ thống nội bộ, có thể mọi người chưa biết đến. Tôi khá biết ơn những trải nghiệm với các project này: có cả hệ thống thuần business lẫn hệ thống thiên về framework, nhờ đó tôi đã học được khá nhiều kiến thức.

Tiếp theo, hãy giới thiệu ngắn gọn từng project, dù sao mỗi project đều đã tiêu tốn rất nhiều tâm sức:

BUGLY là một hệ thống báo cáo Crash qua mạng từ phía client, được rất nhiều APP tích hợp. Huskie là project theo dõi distributed call chain được xây dựng dựa trên zipkin. SOHO là một hệ thống crowdsourcing, chủ yếu đưa các nhiệm vụ chuẩn hóa dữ liệu và thu thập giọng nói ra cộng đồng để mọi người thực hiện. Hệ thống đo lường EPC là hệ thống đo lường hiệu quả phát triển, chủ yếu dùng để đo lường hiệu quả phát triển. Ở đây tôi muốn nói về cách hiểu và nhận thức của mình đối với việc phát triển business. Có thể nhiều người cũng giống tôi lúc đầu, từng thắc mắc: làm business cả ngày thì phát triển thế nào? Nói cách khác, làm CRUD cả ngày thì phát triển thế nào? Ban đầu tôi cũng có thắc mắc như vậy, sau đó tôi đã thay đổi quan niệm.

Tôi cho rằng độ phức tạp của một hệ thống có thể được chia sơ lược thành độ phức tạp kỹ thuật và độ phức tạp business. Với hệ thống business, độ phức tạp business thường cao hơn; còn với hệ thống framework, độ phức tạp kỹ thuật thường cao hơn. Giải quyết hai loại phức tạp này đều là một thách thức lớn.

Hệ thống crowdsourcing tôi từng làm trước đây chứa đủ loại business logic được xử lý qua lại, và đó chính là độ phức tạp business cao. Để giải quyết vấn đề này, chúng tôi bắt đầu khám phá và thực hành Domain-Driven Design (DDD), quả thực đã mang lại một số trợ giúp, giúp hệ thống không còn hỗn loạn như trước. Đồng thời, tôi cảm thấy những hiểu biết của mình về DDD trong quá trình này đã giúp ích cho việc phân chia, thiết kế và phát triển các project về sau.

Tất nhiên DDD không phải silver bullet, tôi cũng không định đề cao nó quá mức. Chỉ là sau khi hiểu về nó, đôi khi khi thiết kế và phát triển, tôi có thể đổi sang một cách suy nghĩ khác.

Có thể thấy rằng làm business hằng ngày cho tốt thực ra cũng không dễ. Nếu có thể khám phá và thực hành nhiều hơn, đưa những phương pháp, tư tưởng hoặc kiến trúc tốt vào áp dụng, điều đó sẽ có ích cho cả bản thân lẫn business.

## Tình hình hiệu suất

Tôi làm việc tại Tencent bốn năm. Tencent đánh giá mỗi nửa năm một lần, tổng cộng tám lần. Nhìn lại, kết quả hiệu suất trong bốn năm lần lượt là: ba sao, ba sao, năm sao, ba sao, năm sao, bốn sao, bốn sao, ba sao. Thống kê cho thấy các mức bốn và năm sao chiếm đúng một nửa.

![](https://oss.javaguide.cn/github/javaguide/high-quality-technical-articles/640.png)

PS: May mà trước đây còn có cúp, nếu không thì chẳng còn gì để lưu niệm. (Có vẻ hiện tại Tencent không còn phát nữa.)

Tôi nhớ khá rõ hai lần đạt năm sao. Lần đầu là vào năm thứ hai đi làm. Năm đó tôi làm project crowdsourcing. Vì bản thân project không quá khó, nên tôi dành một phần sức lực cho việc xây dựng nền tảng của team: giúp team xây dựng project scaffolding cho java và golang, đồng thời thực hiện vài buổi chia sẻ kỹ thuật trong trung tâm. Cuối cùng Leader cho rằng biểu hiện của tôi khá nổi bật nên đã trao năm sao. Xem ra chủ động hơn sẽ có lợi cho cả cá nhân và team, cuối cùng cũng có thể nhận được một số phần thưởng.

Lần đạt năm sao thứ hai liên quan đến EPC. Kể một chuyện khá buồn cười: mãi sau này tôi mới biết, vào giai đoạn đầu của project, khi giám đốc đi báo cáo và demo hệ thống cho cấp trên, các chỉ số mất rất lâu mới tải xong. Giám đốc ngượng ngùng nói rằng hệ thống đang được tối ưu. Một thời gian sau, khi đi báo cáo và demo lần nữa, kết quả vẫn rất lúng túng vì phải chờ rất lâu mới hiển thị được; giám đốc đành nói rằng vẫn đang tối ưu. Không ngờ tôi từng khiến giám đốc mất mặt như vậy, haha. Được rồi, nói về kết quả: cuối cùng tôi tự viết một query engine để thay thế Mondrian, sau đó tình trạng lúng túng như vậy không bao giờ xuất hiện nữa. Cùng với đó, tôi cũng nhận được sự khích lệ bằng một kết quả hiệu suất tốt. Khi làm project đo lường EPC, tôi cảm thấy mình trưởng thành rất nhiều, chẳng hạn như khả năng chịu áp lực. Khi xây dựng một hệ thống từ con số 0, sẽ có một quá trình trước tiên phải chống đỡ được rồi mới tối ưu. Ngoài ra, nếu project của bạn quan trọng, đặc biệt là project liên quan đến dữ liệu, thì bất kỳ vấn đề nhỏ nào cũng có thể khiến bạn căng thẳng, phải tìm mọi cách để giảm rủi ro và sự cố. Một cảm nhận khác nữa là trước đây, trong các project, tôi phần lớn là developer; còn với hệ thống này, tôi là Owner và người phụ trách. Khi làm Owner của một hệ thống, bạn phải luôn chịu trách nhiệm, đồng thời còn phải suy nghĩ về quy hoạch và hướng đi của hệ thống, phân bổ requirement hợp lý và kiểm soát tiến độ. Trải nghiệm vai trò hoàn toàn khác trước đây.

## Nói về EPC

Nhiều người chửi hoặc chế giễu EPC. Là một trong những developer cốt lõi của platform đo lường, tôi muốn nói về một cách nhìn khách quan.

Thực ra mục đích ban đầu của EPC là tốt: thông qua các chỉ số hiệu quả phát triển toàn diện và đa chiều, đo lường chất lượng của từng khâu trong hiệu quả phát triển, từ đó phản hồi ngược cho business và nâng cao hiệu quả phát triển. Tuy nhiên, trong quá trình thực tiễn cuối cùng mới phát hiện rằng điều kiện khách quan chưa đáp ứng được (công cụ vẫn chưa được xây dựng tốt). Hơn nữa, việc một mực theo đuổi dữ liệu chỉ số khiến những người bên dưới tìm mọi cách làm cho chỉ số đẹp hơn, cuối cùng đi ngược lại mục đích ban đầu.

Vì sao lại nói EPC tốt? Nếu tìm hiểu kỹ về EPC, bạn sẽ phát hiện đây là một hệ thống đo lường chỉ số khá hoàn chỉnh và tương đối tiên tiến. Nó bao phủ các khâu requirement, code, defect, testing, continuous integration, vận hành và deploy.

Ngoài ra, dù trong quá trình này có một số người và một số business gian lận, phần lớn business vẫn đã thay đổi. Chẳng hạn, phản hồi từ phía Weishi là code trước đây viết rất tệ, nhưng sau khi có EPC, chất lượng code đã tốt hơn nhiều. Dù cuối cùng Weishi vẫn sụp đổ, khi một tòa nhà sắp sụp thì EPC không thể cứu được, và cũng càng không thể trách EPC về việc đó.

## Nói về người trong phe thân cận

Mọi người đều nói Tencent thịnh hành văn hóa phe cánh. Nhưng thực ra tôi thấy công ty nào cũng như vậy. Điều này cũng phù hợp với quy luật cơ bản của sự vật: con người chỉ tin những người mình tin tưởng và quen thuộc. Là một người lãnh đạo, chẳng lẽ bạn sẽ giao việc quan trọng cho người mình không quen biết sao?

Thực ra tôi cũng không biết mình có được xem là người trong phe thân cận hay không. Trên Maimai từng có người hỏi “làm sao biết mình có phải người trong phe thân cận hay không”, bên dưới có một câu trả lời mà tôi thấy rất sâu sắc: nếu bạn không biết mình có phải hay không, thì bạn không phải. Haha, nói vậy thì có lẽ tôi không phải.

Nhưng mặt khác, về sau tôi phụ trách những việc rất quan trọng trong team, có lẽ là việc cũng rất quan trọng trong trung tâm. Tôi một mình phụ trách một hướng, báo cáo trực tiếp với giám đốc, dường như lại hơi giống.

Trên mạng cũng có cách nói khác rất thẳng: có phải người trong phe thân cận hay không thì cứ nhìn xem tiền có đến tay hay không. Nói vậy cũng có lý. Khi ở cấp 7, tôi đã được phát cổ phiếu, tự cảm thấy tình hình khá tốt. Khi đó tôi nghĩ rằng nếu không có gì bất ngờ, con đường tài chính và phát triển sau này của mình có phải sẽ thuận buồm xuôi gió hay không. Không có gì bất ngờ lại chính là đã có bất ngờ: năm sau, EPC không đạt kỳ vọng, tổng giám đốc bộ phận và giám đốc đều bị thay đổi, trung tâm có một giám đốc mới.

Được rồi, lại phải xây dựng lòng tin từ đầu. Về sau, việc có phải người trong phe thân cận hay không đã không còn quan trọng, vì môi trường chung không tốt, lại thêm cắt giảm nhân sự, gần như mọi người đều lần lượt rời đi, dù chủ động hay bị động.

Tóm lại, sự tồn tại của phe thân cận thực ra cũng có nguyên do. Làm thế nào để trở thành người trong phe thân cận? Thực ra tôi cũng không biết. Tuy nhiên, tôi cho rằng thay vì suy nghĩ làm thế nào để trở thành người trong phe thân cận, chi bằng suy nghĩ làm thế nào thể hiện giá trị và năng lực của bản thân. Khi người khác nhận ra giá trị và năng lực của bạn, tự nhiên sẽ có nhiều cơ hội hơn dành cho bạn. Có cơ hội rồi, chỉ cần nắm bắt được thì sẽ có thêm nhiều đãi ngộ.

## Nói thêm về những gì đạt được

Đạt được, thế nào mới gọi là đạt được? Theo tôi, dù là vật chất, kỹ năng, cấp bậc bên ngoài, hay cảm nhận và nhận thức bên trong, tất cả đều có thể xem là thành quả.

Trước tiên nói về một số thứ có thể định lượng, tôi cho rằng gồm:

- Về cấp bậc, tôi đã thăng lên cấp 9, trở thành kỹ sư cấp cao. Dù mọi người đều nói cấp bậc tại Tencent đã bị hạ thấp, nhưng bản thân có năng lực của một kỹ sư cấp cao hay không thì chính mình biết rõ. Tôi cảm thấy qua những nỗ lực trong vài năm qua, mình đã đạt đến trạng thái mà khi đó tôi cho rằng một kỹ sư cấp cao cần đạt được;
- Về hiệu suất, tự đánh giá thì tôi không phải người quá thiên về cạnh tranh, hay nói cách khác, không cạnh tranh chỉ vì muốn cạnh tranh. Nhưng nếu đã xác định phải làm tốt, tôi cho rằng Owner awareness và thái độ chịu trách nhiệm của mình vẫn khá ổn. Cuối cùng, kết quả hiệu suất trong bốn năm tại Tencent cũng tạm chấp nhận được. Tiếp theo là một số soft skill khác:

**1. Khả năng viết tài liệu**

Với lập trình viên, khả năng viết tài liệu thực ra là một năng lực rất quan trọng. Thực ra tôi cũng không cảm thấy khả năng viết tài liệu của mình tốt đến đâu, nhưng cả hai giám đốc trước sau đều nói tài liệu của tôi khá tốt. Xem ra tôi có thể đang ở trên mức trung bình.

**2. Xác định rõ hướng đi**

Cuối cùng, nói về một thành quả trừu tượng hơn nhưng tôi cho là có giá trị nhất: tôi dần xác định rõ hướng đi và con đường sau này, đó là làm phát triển dữ liệu.

Thực ra, tìm thấy và xác định một mục tiêu rất khó. Những người xung quanh có mục tiêu và hướng đi rõ ràng không nhiều, phần lớn đều mơ hồ.

Một thời gian trước, khi trò chuyện với một người về quy hoạch nghề nghiệp, chúng tôi nói rằng có thể suy nghĩ từ hai góc độ:

- Chọn một hướng business, chẳng hạn như thương mại điện tử hoặc quảng cáo, không ngừng tích lũy kiến thức trong lĩnh vực business và kỹ năng liên quan đến business. Cùng với kinh nghiệm tích lũy ngày càng nhiều, cuối cùng bạn sẽ trở thành chuyên gia trong lĩnh vực đó.
- Đi sâu vào một hướng kỹ thuật, không ngừng nghiên cứu kỹ thuật tầng dưới. Như vậy bạn sẽ có cơ hội trở thành chuyên gia về kỹ thuật đó. Thành thật mà nói, dù tôi từng nghiên cứu chuyên sâu và thực hành Domain-Driven Design, cũng từng dùng nó để modeling và giải quyết một số vấn đề business phức tạp, nhưng từ tận đáy lòng, tôi thích nghiên cứu kỹ thuật hơn. Đồng thời, tôi cũng rất hứng thú với big data. Vì vậy, tôi đã quyết định hướng đi sau này của mình là làm công việc liên quan đến dữ liệu.

Bốn năm tại Tencent là trải nghiệm công việc đầu tiên của tôi. Tôi đã gặp nhiều người tài giỏi và học được rất nhiều điều. Cuối cùng tôi chủ động rời đi, cũng có thể xem là rời đi một cách đàng hoàng (dù đã bỏ lỡ một gói đãi ngộ lớn). Dù sao tôi vẫn cảm ơn Tencent.

<!-- @include: @article-footer.snippet.md -->
