---
title: Làm thế nào để nhanh chóng bắt nhịp công việc khi gia nhập công ty mới
description: "Làm thế nào để nhanh chóng bắt nhịp công việc khi gia nhập công ty mới: hệ thống hóa các khái niệm then chốt, vấn đề thường gặp và điểm thực hành xoay quanh kiến thức kỹ thuật cùng kinh nghiệm phỏng vấn, giúp bạn học tập hiệu quả và chuẩn bị phỏng vấn."
category: Tuyển tập bài viết kỹ thuật chất lượng cao
tag:
  - Công việc
head:
  - - meta
    - name: keywords
      content: gia nhập công ty mới, nhanh chóng hòa nhập, trạng thái làm việc, tìm hiểu nghiệp vụ, làm quen công nghệ, phối hợp nhóm, thích nghi khi chuyển việc, programmer gia nhập công ty
---

> **Lời giới thiệu**: Rất khuyến khích mọi bạn sắp gia nhập hoặc đang làm việc tại công ty đọc bài viết này. Sau khi đọc, bạn có thể tránh được rất nhiều cạm bẫy. Toàn bài có logic rõ ràng và nội dung toàn diện!
>
> **Địa chỉ bài gốc**: <https://www.cnblogs.com/hunternet/p/14675348.html>

![Làm thế nào để nhanh chóng bắt nhịp khi gia nhập công ty mới](https://oss.javaguide.cn/github/javaguide/high-quality-technical-articles/work/%E6%96%B0%E5%85%A5%E8%81%8C%E4%B8%80%E5%AE%B6%E5%85%AC%E5%8F%B8%E5%A6%82%E4%BD%95%E5%BF%AB%E9%80%9F%E8%BF%9B%E5%85%A5%E7%8A%B6%E6%80%81.png)

Mùa nhảy việc cao điểm tháng 3, tháng 4 sắp khép lại. Tôi tin rằng nhiều bạn đã tìm được công việc ưng ý và sắp có hoặc đã có một khởi đầu mới.

Những bạn từng có kinh nghiệm chuyển việc hẳn đều biết, mỗi khi đến một công ty mới, bạn có thể phải đối mặt với nghiệp vụ, công nghệ và đội nhóm mới... Những điều này có thể phá vỡ tư duy làm việc, thói quen code và cách phối hợp trước đây của bạn...

Với công ty, họ lại không thể cho bạn vài tháng để từ từ làm quen. Vì vậy, làm thế nào để nhanh chóng bắt nhịp công việc và sớm phát huy giá trị của bản thân là điều rất quan trọng.

Một số người có thể may mắn: công ty mới có quy trình và cơ chế hoàn thiện, giúp nhân viên mới nhanh chóng bắt nhịp thông qua hình thức kèm cặp, nhiều chương trình đào tạo... Một số người lại không may mắn như vậy. Chẳng hạn, vài năm trước khi chuyển việc vào một công ty lớn, tôi chưa có cơ chế giúp người mới hòa nhập hoàn thiện như hiện nay, lại đúng lúc đội nhóm bận rộn nhất. Ngay chiều ngày đầu tiên đi làm, tôi đã được giao vài vấn đề trên production để kiểm tra, nhưng không có tài liệu hay đào tạo nào. Khi gặp tình huống như vậy, nhiều người có thể khó thích nghi nhanh, cuối cùng không chịu nổi áp lực và nảy sinh ý định nghỉ việc.

![bad175e3a380bea.](https://hunter-picgos.oss-cn-shanghai.aliyuncs.com/picgo/bad175e3a380bea..jpg)

Vậy, **chúng ta nên làm thế nào để nhanh chóng bắt nhịp công việc và thích nghi với nhịp độ làm việc mới?**

Khi bắt đầu công việc mới, trước mắt là cả đống code repository nên nhiều người thường cảm thấy không biết bắt đầu từ đâu. Nhưng nhìn lại kinh nghiệm làm việc và dự án trước đây, chúng ta có thể nhận ra chúng có nhiều điểm tương đồng. Khi bắt đầu một dự án mới, thường sẽ trải qua vài bước: yêu cầu -> thiết kế -> phát triển -> test -> phát hành, lặp đi lặp lại như vậy để hoàn thành hết dự án này đến dự án khác.

![Quy trình dự án](https://oss.javaguide.cn/github/javaguide/high-quality-technical-articles/work/image-20220704191430466.png)

Trong quá trình này, bốn khía cạnh kiến thức là nghiệp vụ, kỹ thuật, dự án và đội nhóm luôn xuyên suốt. Khi gia nhập công ty mới, mục tiêu giai đoạn đầu tiên của chúng ta là có năng lực theo đội nhóm thực hiện dự án. Vì vậy, những kiến thức cần nhanh chóng nắm vững cũng phải bắt đầu từ bốn khía cạnh này.

## Nghiệp vụ

Nhiều người có thể cho rằng là một người làm kỹ thuật thì điều cần hiểu nhất chẳng phải là kỹ thuật sao? Vì vậy, sau khi vào công ty, họ nóng lòng nghiên cứu tài liệu kỹ thuật, kiến trúc hệ thống, thậm chí cầm source code lên đọc ngay. Nếu bạn cũng làm như vậy thì hoàn toàn sai lầm! Ở hầu hết công ty, kỹ thuật tồn tại như một công cụ. Dù rất quan trọng, kỹ thuật cũng tồn tại để phục vụ nghiệp vụ; kỹ thuật giải quyết vấn đề làm thế nào, còn nghiệp vụ cho chúng ta biết làm gì và vì sao phải làm. Một khi tách khỏi nghiệp vụ, sự tồn tại của kỹ thuật sẽ hoàn toàn mất ý nghĩa.

Có hai cách rất quan trọng để tìm hiểu nghiệp vụ.

**Một là dựa vào hỏi**

Nếu đội nhóm bạn gia nhập có cơ chế đào tạo nghiệp vụ hoàn thiện và tài liệu yêu cầu chi tiết, có lẽ bạn không cần hỏi quá nhiều vẫn có thể hiểu nghiệp vụ. Nhưng đó chỉ là tình huống lý tưởng; phần lớn công ty không có điều kiện này. Vì vậy, chúng ta chỉ có thể dựa vào việc hỏi.

Ở đây nhất định phải nói rằng người mới cần có một chút “độ dày da mặt”, không hiểu thì phải hỏi. Tôi từng thấy nhiều người mới vì hướng nội, rụt rè nên luôn ngại hỏi khi gặp thắc mắc. Điều này khiến họ mất rất nhiều thời gian để hòa nhập đội nhóm và đảm nhận trách nhiệm quan trọng hơn. Đừng sợ bị mắng hay bị phản bác; tôi tin rằng tuyệt đại đa số programmer đều rất dễ trao đổi!

**Hai là dựa vào test**

Tôi cho rằng test chắc chắn là cách giúp một người nhanh chóng tìm hiểu nghiệp vụ của đội nhóm. Thông qua test, chúng ta có thể đi qua toàn bộ quy trình của dự án do đội nhóm phụ trách. Nếu gặp chỗ không thể tiếp tục hoặc không hiểu, hãy kịp thời hỏi. Trong quá trình này, chúng ta sẽ tự nhiên nhanh chóng hiểu được quy trình nghiệp vụ cốt lõi.

Trong quá trình tìm hiểu nghiệp vụ, điều cần chú ý là đừng quá chú trọng chi tiết. Mục tiêu của chúng ta trước hết là hiểu tổng thể quy trình nghiệp vụ: chúng ta phục vụ những user nào, cung cấp những service nào...

## Kỹ thuật

Sau khi sơ bộ tìm hiểu xong nghiệp vụ, đã đến lúc tìm hiểu kỹ thuật. Có lẽ bạn đã không thể kiềm chế ý định mở source code, nhưng vẫn phải nhắc bạn một câu: đừng vội.

Lúc này, trước tiên chúng ta nên dựa trên nghiệp vụ đã hiểu và kết hợp kinh nghiệm làm việc trước đây để suy nghĩ: nếu tự mình triển khai hệ thống này thì nên làm thế nào? Bước này rất quan trọng. Sau đó, khi tìm hiểu cách hệ thống được triển khai về mặt kỹ thuật, bạn có thể so sánh điểm khác biệt giữa cách triển khai của mình và hệ thống, vì sao có những khác biệt đó, cách nào tốt hơn, cách nào chưa tốt. Với cách chưa tốt, bạn có thể đưa ra ý kiến; với cách tốt hơn, bạn có thể tiếp thu và học hỏi để áp dụng cho mình!

Tiếp theo, chúng ta sẽ tìm hiểu kỹ thuật, nhưng cũng không nên ngay lập tức lật xem source code. **Nên phân tích hệ thống từng bước, từ tổng quan đến chi tiết, từ ngoài vào trong.**

Trước hết, chúng ta nên tìm hiểu sơ bộ **technology stack mà đội nhóm/dự án sử dụng**: Java hay .NET, hay có nhiều ngôn ngữ cùng tồn tại; dự án tách frontend và backend hay server bao trọn cả hai; database sử dụng là MySQL hay PostgreSQL... Nhờ vậy, chúng ta có thể hình dung nhất định về các công nghệ, framework được sử dụng và phần việc mình phụ trách. Một số người có thể đã tìm hiểu sơ bộ điều này trong lúc phỏng vấn.

Tiếp theo, chúng ta nên tìm hiểu **kiến trúc nghiệp vụ tổng thể của hệ thống**. Đội nhóm mình chủ yếu phụ trách những hệ thống nào, mỗi hệ thống chủ yếu gồm những module nào, tương tác với những hệ thống bên ngoài nào... Tốt nhất nên dùng flowchart hoặc mind map để hệ thống hóa những điều này.

Sau đó, chúng ta cần xem **đội nhóm mình cung cấp những API hoặc service nào ra bên ngoài**. Mỗi API và service cung cấp chức năng gì? Chúng ta có thể tiếp tục test hệ thống của mình. Lúc này, hãy xem quy trình chính gồm những page nào, mỗi page gọi những API backend nào, mỗi API backend tương ứng với code repository nào. (Nếu chỉ làm backend service, có thể xem đội nhóm cung cấp những service nào, có những upstream service nào, mỗi upstream service gọi những service nào của đội nhóm...) Tương tự, chúng ta nên dùng hình vẽ để hệ thống hóa chúng.

Tiếp đó, chúng ta cần tìm hiểu **hệ thống hoặc service của mình phụ thuộc vào những external service nào**, tức là cần những hệ thống bên ngoài nào hỗ trợ. Những service này có thể do đội nhóm khác hoặc công ty khác cung cấp. Lúc này, chúng ta có thể vào code để xem sơ bộ cách tương tác với hệ thống bên ngoài, bao gồm communication framework (REST, RPC), communication protocol...

Ở tầng code, trước hết chúng ta nên tìm hiểu cấu trúc phân tầng của code trong mỗi module: một module được chia thành bao nhiêu tầng, trách nhiệm của mỗi tầng là gì. Hiểu được điều này, chúng ta sẽ có khái niệm ban đầu về toàn bộ thiết kế hệ thống; tiếp đó là cấu trúc thư mục code và vị trí của các file config.

Cuối cùng, chúng ta có thể tìm một ví dụ, chẳng hạn một API hoặc một page, để lần theo đường chạy của code, đi trọn một lượt từ input đến output nhằm kiểm chứng hiểu biết trước đó.

Đến đây, việc tìm hiểu về mặt kỹ thuật có thể tạm dừng. Mục tiêu của chúng ta chỉ là có nhận thức ban đầu về hệ thống; những phần chi tiết hơn sẽ còn rất nhiều thời gian để tìm hiểu sau.

## Dự án và đội nhóm

Ở trên, chúng ta đã nói rằng khi gia nhập công ty mới, mục tiêu giai đoạn đầu tiên là có năng lực theo đội nhóm thực hiện dự án. Tiếp theo, chúng ta cần tìm hiểu dự án vận hành như thế nào.

Chúng ta nên nắm các điểm then chốt trong toàn bộ quá trình từ thiết kế yêu cầu, viết code, đưa code vào repository cho đến phát hành lên production. Chẳng hạn, dự án áp dụng mô hình agile hay waterfall, một iteration kéo dài bao lâu, yêu cầu đến từ đâu và được thể hiện dưới hình thức nào, có requirement review không, quy chuẩn viết code là gì, viết xong thì build thế nào, đưa code vào repository ra sao, có quy chuẩn commit không, bàn giao cho test thế nào, cần chuẩn bị gì trước khi phát hành, sử dụng công cụ phát hành ra sao...

Về dự án, chúng ta chỉ cần quan sát đồng nghiệp hoặc tự mình trải qua một iteration phát triển là có thể đại khái hiểu rõ.

Trong khi tìm hiểu cách dự án vận hành, chúng ta cũng nên tìm hiểu đội nhóm. Tương tự, trước hết nên bắt đầu từ bên ngoài: chúng ta kết nối với những đội nhóm bên ngoài nào, chẳng hạn yêu cầu đến từ đâu, có kết nối với đội nhóm bên ngoài công ty không, những upstream team nào cung cấp service, những downstream team nào là dependency, các đội nhóm trao đổi với nhau như thế nào, cách trao đổi thường dùng là gì...

Tiếp theo là bên trong đội nhóm: đội nhóm có những role nào, trách nhiệm của mỗi người là gì. Nhờ vậy, khi gặp vấn đề, chúng ta có thể biết rõ nên tìm đồng nghiệp tương ứng để nhờ hỗ trợ. Có những hoạt động và cuộc họp định kỳ nào không, chẳng hạn daily stand-up, weekly meeting; có quy tắc bất thành văn nào không; có cơ chế review nội bộ hoặc chia sẻ nào không...

## Tổng kết

Khi gia nhập công ty mới và đối mặt với những thách thức mới trong công việc, việc nhanh chóng bắt nhịp và tạo ra giá trị cho bản thân sẽ đem lại cho bạn một khởi đầu tốt.

Với một programmer, nhanh chóng bắt nhịp công việc trước hết có nghĩa là phải có năng lực theo đội nhóm thực hiện dự án. Ở đây, tôi đứng trên góc nhìn backend development và tổng hợp một số phương pháp, kinh nghiệm từ bốn khía cạnh: nghiệp vụ, kỹ thuật, dự án và đội nhóm.

Nếu bạn có phương pháp hoặc đề xuất hay về cách nhanh chóng bắt nhịp công việc, hãy để lại bình luận.

Cuối cùng, hãy dùng một mind map để ôn lại nội dung bài viết. Nếu thấy bài viết hữu ích, bạn có thể theo dõi public account ở cuối bài. Tôi sẽ thường xuyên chia sẻ một số kinh nghiệm và suy nghĩ trong quá trình trưởng thành của mình để cùng mọi người học hỏi và tiến bộ.

<!-- @include: @article-footer.snippet.md -->
