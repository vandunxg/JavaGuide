---
title: Chia sẻ kinh nghiệm làm backend hai năm tại Didi và Toutiao
description: "Chia sẻ kinh nghiệm làm backend hai năm tại Didi và Toutiao: hệ thống hóa các khái niệm cốt lõi, vấn đề thường gặp và điểm thực tiễn xoay quanh kiến thức kỹ thuật và phỏng vấn, giúp bạn học tập và chuẩn bị phỏng vấn hiệu quả."
category: Tuyển tập bài viết kỹ thuật
tag:
  - Kinh nghiệm cá nhân
head:
  - - meta
    - name: keywords
      content: kinh nghiệm làm việc tại Didi,kinh nghiệm làm việc tại Toutiao,phát triển backend,trưởng thành kỹ thuật,kinh nghiệm công sở,suy ngẫm sâu,tổng kết tích lũy,chủ động đảm nhận
---

> **Lời giới thiệu**: Một bài chia sẻ kinh nghiệm làm việc rất thiết thực, đọc xong chắc chắn sẽ thấy hữu ích!
>
> **Tổng quan nội dung**:
>
> - Hãy học cách suy ngẫm sâu, tổng kết và tích lũy; đây là điều tôi cho là quan trọng và có ý nghĩa nhất.
> - Chủ động học hỏi, duy trì nhiệt huyết kỹ thuật. Nếu chúng ta chủ động học hỏi, duy trì năng lực kỹ thuật và nền tảng kiến thức tăng tương ứng với số năm làm việc, vậy đến 35 tuổi còn gì phải lo lắng? Một người giỏi như vậy chắc cũng được các công ty tranh nhau tuyển dụng nhỉ?
> - Để hoàn thành công việc và tạo ra giá trị cho công ty, tôi cho rằng hai chữ quan trọng nhất là chủ động: chủ động đảm nhận nhiệm vụ, chủ động giao tiếp trao đổi, chủ động thúc đẩy tiến độ dự án, chủ động điều phối nguồn lực, chủ động phản hồi lên cấp trên, chủ động tạo ảnh hưởng, v.v.
> - Hãy dạn dĩ hơn, chủ động tìm người trao đổi để nhanh chóng hòa nhập; điều tối kỵ là gặp vấn đề nhưng không nói, tự cô lập mình.
> - Muốn nịnh thì cứ nịnh, không muốn nịnh cũng không cần mỉa mai người khác, Respect Greatness.
> - Luôn sẵn sàng, có kỹ thuật trong tay thì chẳng có gì đáng sợ; ngày nào làm việc không vui thì cứ nghỉ việc chuyển công ty.
> - Thường xuyên chủ động tổng kết và tích lũy, trao đổi nhiều hơn với người khác để hình thành methodology.
> - …
>
> **Địa chỉ bài gốc**: <https://www.nowcoder.com/discuss/351805>

Trước hết xin nói sơ qua về bối cảnh: tôi tốt nghiệp đại học và thạc sĩ tại một trường 985 không mấy tên tuổi, năm 2017 tốt nghiệp rồi gia nhập Didi; khi đó lúc tìm việc tôi cũng cùng mọi người chiến đấu tại Nowcoder. Nửa cuối năm nay tôi chuyển việc sang Toutiao, vẫn luôn làm các công việc liên quan đến phát triển backend. Trước đây tôi chưa từng thực tập, có thể xem là đã có hai năm rưỡi kinh nghiệm làm việc. Trong hai năm rưỡi đó, tôi được thăng chức một lần, chuyển sang một công ty khác, từng có những ngày vui vẻ mãn nguyện, cũng từng có những ngày hoang mang chật vật, nhưng nhìn chung vẫn thuận lợi chuyển từ một người mới non nớt nơi công sở thành một nhân viên kỳ cựu chuyên “đánh cá”. Trong quá trình này, tôi tổng kết được một số kinh nghiệm “đánh cá” khá thiết thực; có điều tự mình lĩnh ngộ, có điều học được qua trao đổi với người khác, nay chia sẻ cùng mọi người.

## Học cách suy ngẫm sâu, tổng kết và tích lũy

**Điều đầu tiên tôi muốn nói là hãy học cách suy ngẫm sâu, tổng kết và tích lũy; đây là điều tôi cho là quan trọng và có ý nghĩa nhất.**

**Trước hết nói về suy ngẫm sâu.** Trong giới lập trình viên, ta thường nghe những lời như: _“Công việc của tôi chẳng có chút hàm lượng kỹ thuật nào, mỗi ngày chỉ CRUD, rồi viết vài đoạn if-else, cái này thì có thể giúp tôi học được gì?”_

Không bàn đến một phần những lời châm biếm và đùa cợt, đây có thể đúng là suy nghĩ thật của một số bạn; ít nhất trước đây tôi cũng từng nghĩ như vậy. Sau này, cùng với việc tích lũy kinh nghiệm làm việc và trao đổi với một số bạn có level cao, tôi nhận ra suy nghĩ đó thực ra hoàn toàn sai. Quan niệm “không có gì để học” xuất hiện về cơ bản là kết quả của sự lười biếng trong tư duy. **Bất cứ việc nhỏ nào trông có vẻ không đáng kể, chỉ cần suy ngẫm sâu, đào sâu thêm một chút theo chiều dọc hoặc mở rộng thêm theo chiều ngang, đều đủ để khiến người ta đắm mình trong đại dương tri thức.**

Lấy một ví dụ. Có lần một bạn nói với tôi rằng tuần này một service bị OOM, điều tra cả tuần mới phát hiện có chỗ viết defer sai, sửa vài dòng code rồi deploy để khắc phục, đến báo cáo tuần cũng không biết viết gì. Có lẽ mọi người cũng từng gặp tình huống như vậy, đây là một ví dụ khá điển hình. Thực ra riêng việc debug đã là một quá trình phát hiện vấn đề, điều tra vấn đề và giải quyết vấn đề, bao gồm nhiều bước như trigger, định vị, tái hiện, tìm root cause, sửa lỗi và retrospective. Đã dành cả tuần cho việc này thì chắc chắn đã có quá trình không ngừng thử nghiệm và sửa sai; trong đó có rất nhiều chỗ để suy ngẫm. Chẳng hạn về định vị, đã thu hẹp phạm vi như thế nào? Đã đi qua những đường vòng nào? Đã dùng những công cụ phân tích nào? Chẳng hạn về root cause, ít nhất có thể nghiên cứu OOM của Linux, OOM của K8s, memory management của Go, cơ chế defer, nguyên lý function closure, v.v. Nếu thực sự không liên quan đến những vấn đề này mà vẫn mất cả tuần để làm, vậy retrospective hẳn sẽ có rất nhiều điều để suy ngẫm; đưa ra vài chục WHY cũng không thành vấn đề nhỉ...

**Tiếp theo nói về tổng kết và tích lũy.** Tôi cho rằng đây cũng là điểm mà phần lớn lập trình viên còn thiếu: chỉ cắm đầu làm việc thì có thể làm một việc rất tốt, nhưng hầu như không bao giờ trừu tượng hóa và tổng kết. Vì vậy, dù đã làm việc vài năm, kiến thức nắm được vẫn chỉ là vài mảnh rời rạc, không thành hệ thống; không chỉ dễ quên mà còn khiến tầm nhìn bị hạn hẹp, cách nhìn vấn đề bị giới hạn. Kịp thời tổng kết và tích lũy là điều rất quan trọng, đây là quá trình từ thuật đến đạo, giúp góc nhìn khi xem xét vấn đề rộng hơn và ở tầng cao hơn. Khi gặp vấn đề cùng loại, ta có thể dựa trên methodology đã tổng kết để thúc đẩy và giải quyết một cách có hệ thống, theo từng tầng.

Vẫn lấy một ví dụ. Làm backend service, hôm nay tối ưu 1G memory, ngày mai tối ưu 50% thời gian đọc ghi, vậy có thể tổng kết về performance optimization không? Ở application layer, có thể quản lý các bên sử dụng kết nối với service, rà soát tính hợp lý trong việc truy cập của họ; ở architecture layer, có thể dùng cache, preprocessing, read-write separation, asynchronous, parallel, v.v.; ở code layer, còn có thể làm nhiều việc hơn như resource pooling, object reuse, lock-free design, tách large key, xử lý trì hoãn, nén encoding, tuning GC và nhiều thực tiễn performance cao theo từng ngôn ngữ... Lần sau gặp tình huống cần performance optimization, lập tức có thể áp dụng cả một bộ tư duy; phần còn lại chỉ là công cụ và thực hành.

Cũng có bạn nói rằng tôi chỉ tranh luận với PM và làm requirement mỗi ngày, đâu có làm performance optimization. Tạm không bàn xem có thể làm performance optimization hay không, chỉ riêng việc làm business requirement cũng có chỗ để tổng kết. Chẳng hạn, đã từng suy nghĩ về cách xây dựng system chưa? Năng lực cốt lõi của system, boundary của system, bottleneck của system, phân tầng và tách service, service governance, những vấn đề này đã từng suy nghĩ chưa? Mỗi ngày thảo luận requirement với PM, vậy một người làm kỹ thuật nên rèn product thinking và dẫn dắt hướng đi của product như thế nào, làm sao để architecture đi trước business; những vấn đề này cũng có thể suy nghĩ và tổng kết mà. Cứ nghĩ xem, ngay cả việc tiếp nhận và bảo trì code tệ của người khác, Martin Fowler cũng có thể phát triển thành cả một bộ lý thuyết refactoring, lại còn khiến nó trông cao siêu như vậy; chúng ta thực sự cũng không cần tự coi nhẹ công việc của mình...

Vì vậy: **Học tập và trưởng thành là một quá trình tự thúc đẩy. Nếu cảm thấy không còn gì để học, phần lớn không phải thật sự không có gì để học, mà là vì bản thân quá lười; không chỉ lười hành động mà còn lười tư duy. Có thể viết thêm các bài viết kỹ thuật, chia sẻ nhiều hơn, ép bản thân suy nghĩ và tổng kết; dù sao nếu bài viết chưa đủ chiều sâu thì cũng ngại công khai chia sẻ.**

## Chủ động học hỏi, duy trì nhiệt huyết kỹ thuật

Hai năm gần đây, trong giới Internet lan truyền rộng rãi một nỗi lo gọi là hiện tượng lập trình viên 35 tuổi, đại ý là làm trong ngành lập trình đến 35 tuổi thì cơ bản chỉ còn chờ bị sa thải. Không thể phủ nhận ngành Internet ở điểm này thực sự không bằng các nghề trong hệ thống như công chức. Tuy nhiên, lập trình viên 35 tuổi trong vấn đề này không phải là 35 tuổi theo nghĩa sinh học tuyệt đối, mà nên chỉ những lập trình viên đã làm việc hơn mười năm nhưng không khác mấy so với người mới làm hai ba năm. Công việc sau đó cơ bản là ăn vào vốn cũ, không chủ động học hỏi và nạp thêm kiến thức; 35 tuổi cũng gần như 25 tuổi, nhưng lại không còn khao khát học tập và trưởng thành như năm 25 tuổi, ngược lại còn phải lo thêm nhiều việc vụn vặt của gia đình, yêu cầu lương thường cũng cao hơn; trong mắt doanh nghiệp, quả thực không có nhiều sức cạnh tranh.

**Nếu chúng ta chủ động học hỏi, duy trì năng lực kỹ thuật và nền tảng kiến thức tăng tương ứng với số năm làm việc, vậy đến 35 tuổi còn gì phải lo lắng? Một người giỏi như vậy chắc cũng được các công ty tranh nhau tuyển dụng nhỉ?** Nhưng **học tập thực ra là một quá trình đi ngược bản năng con người, nên cần ép bản thân bước ra khỏi vùng an nhàn, chủ động học hỏi và duy trì nhiệt huyết kỹ thuật.** Ở Didi có một câu đại ý rằng: **chủ động bước ra khỏi vùng thoải mái; khi cảm thấy chật vật và áp lực thì thường đó là bóng tối trước bình minh, cũng là lúc trưởng thành nhanh nhất. Ngược lại, nếu cảm thấy mỗi ngày đều quá nhàn nhã, công việc chỉ là làm cho đủ thời gian, có lẽ thực sự đang bị luộc như ếch trong nước ấm rồi.**

Khoảng thời gian vừa tốt nghiệp thường còn khá nhiều thời gian rảnh, chính là lúc tốt để nỗ lực học kỹ thuật. Tận dụng khoảng thời gian này để củng cố nền tảng, hình thành thói quen học tập tốt và duy trì thái độ học tập tích cực sẽ đem lại lợi ích cả đời. Về cách học hiệu quả cao, trên mạng có nhiều người giỏi viết các bài chia sẻ như vậy; sau khi vào công ty, trên mạng nội bộ cũng có thể tìm thấy nhiều chia sẻ tương tự, nên tôi không nói thêm.

**_Có thể tham gia các nhóm học tập và cộng đồng kỹ thuật, cả trong và ngoài công ty, đồng thời quan tâm đến các công nghệ tiên tiến._**

## Chủ động đảm nhận, kịp thời trao đổi và phản hồi

Hai phần đầu vẫn xuất phát từ góc độ cá nhân, hy vọng mọi người có thể nâng cao năng lực cá nhân và duy trì năng lực cạnh tranh cốt lõi. Nhưng xét từ góc độ công ty, công ty tuyển nhân viên vào làm thì điều quan trọng nhất là để nhân viên tạo ra business value và phục vụ công ty. Dù với nhân viên mới tốt nghiệp thường có một hệ thống đào tạo nhất định, trên thực tế công ty quả thực không có nghĩa vụ giúp chúng ta trưởng thành.

**Để hoàn thành công việc và tạo ra giá trị cho công ty, tôi cho rằng hai chữ quan trọng nhất là chủ động: chủ động đảm nhận nhiệm vụ, chủ động giao tiếp trao đổi, chủ động thúc đẩy tiến độ dự án, chủ động điều phối nguồn lực, chủ động phản hồi lên cấp trên, chủ động tạo ảnh hưởng, v.v.**

Khi mới vào làm, cơ bản là leader giao nhiệm vụ gì thì tôi làm tốt phần việc của mình, sau đó làm việc riêng; gần như chưa bao giờ chủ động trao đổi với người khác hay chủ động suy nghĩ về những ý tưởng có thể giúp dự án phát triển. Tôi cứ nghĩ chỉ cần hoàn thành công việc của mình với chất lượng và số lượng đảm bảo là được, sau này mới nhận ra làm như vậy thực ra hoàn toàn chưa đủ, đó chỉ là yêu cầu cơ bản nhất. Còn có những bạn làm theo cách leader chỉ cần đồng bộ hướng đi gần đây, các việc tiếp theo cơ bản không cần leader phải bận tâm nữa; nếu tôi là leader, tôi cũng thích những bạn như vậy. Sau khi vào làm, ta thường nghe một cụm từ là owner consciousness, đại khái chính là ý này.

Trong quá trình đó, một điểm quan trọng khác là kịp thời trao đổi và phản hồi lên cấp trên. Nếu tiến độ dự án không thuận lợi hoặc gặp vấn đề gì, hãy kịp thời đồng bộ với leader; nếu chưa chắc chắn về technical solution, có thể thảo luận với leader; nếu không điều phối được một số resource, có thể nhờ leader giúp, đừng quá e ngại và nghĩ rằng những việc này sẽ gây phiền; leader vốn là để làm những việc đó.. Nếu tiến độ dự án khá thuận lợi, thực sự không cần leader can thiệp, thì cũng cần kịp thời phản hồi tiến độ dự án và lợi ích đạt được, nêu ra ý tưởng của mình để cùng thảo luận, hỏi leader về đề xuất đối với tiến độ hiện tại và còn điểm nào cần cải thiện để loại bỏ sai lệch thông tin. Làm những việc này một mặt là tận dụng hợp lý các resource của leader, mặt khác cũng giúp leader biết được khối lượng công việc của mình và nắm được tổng thể dự án; dù sao leader cũng có leader và cũng phải báo cáo. Có lẽ đây được xem là upward management mà mọi người khá phản cảm, cũng hơi có “chất” ấy; thực ra tôi cũng làm chưa tốt. Nhưng ít nhất, đừng nhận một task rồi cắm đầu làm, thậm chí tách biệt với mọi người, cả tháng không đồng bộ với leader, nghĩ rằng sẽ tung ra một đòn bất ngờ; như vậy cơ bản là hết cơ hội.

**Nhất định phải chủ động; có thể bắt đầu bằng việc ép bản thân phát biểu trong nhiều hoàn cảnh công khai, gặp vấn đề hoặc có ý tưởng thì kịp thời one-one.**

Ngoài những điểm trên, tôi cho rằng còn một số điểm nhỏ cũng khá quan trọng, liệt kê bên dưới:

## Việc đầu tiên là xây dựng niềm tin

Dù là tuyển dụng từ trường hay tuyển dụng xã hội, việc đầu tiên sau khi vào làm là vô cùng quan trọng, trực tiếp quyết định ấn tượng đầu tiên của leader và đồng nghiệp về bạn. Việc đầu tiên cần làm sau khi vào công ty nhất định phải làm thật tốt, ít nhất phải hoàn thành thuận lợi và không được xảy ra sự cố trên production. Mục đích của việc này là xây dựng niềm tin, để team cảm thấy bạn ít nhất là người đáng tin cậy. Nếu làm tốt việc này, những việc sau sẽ khá thuận lợi. Nếu làm hỏng việc này, có thể một số leader vẫn cho cơ hội thứ hai; nhưng nếu lại làm không tốt thì sau đó sẽ rất khó khăn. Điều này càng quan trọng hơn với tuyển dụng xã hội.

Khi mới vào làm, chưa quen tech stack của công ty, business phức tạp khó định hướng, áp lực quả thực khá lớn. Lúc này một mặt cần tự mình bỏ ra nhiều công sức hơn, mặt khác cần trao đổi nhiều hơn với các bạn trong team, không hiểu thì hỏi. **Tôi cho rằng cách học hiệu quả nhất không phải là đọc sách hay xem video học tập, mà là trực tiếp tìm đúng người để trao đổi; để người khác giải thích một lần thì cơ bản đã hiểu hết, hiệu quả nhanh hơn nhiều so với đọc tài liệu và xem code. Không chỉ bỏ qua được quá trình lọc thông tin vô ích, bạn còn hiểu được lịch sử phát triển của business. Tất nhiên, điều này cần một số kỹ năng giao tiếp nhất định, dù sao đồng nghiệp cũng rất bận.**

**Hãy dạn dĩ hơn, chủ động tìm người trao đổi để nhanh chóng hòa nhập; điều tối kỵ là gặp vấn đề nhưng không nói, tự cô lập mình.**

## Vượt quá kỳ vọng

Phạm vi của cụm từ “vượt quá kỳ vọng” rất rộng. Chẳng hạn, leader bảo bạn trực tuần và giải đáp câu hỏi của mọi người trong user group; kết quả không chỉ giải đáp vấn đề mà còn thu thập và phân loại các câu hỏi, sau đó làm một chatbot hỏi đáp thông minh để giải phóng nhân lực trực tuần, đây có thể xem là vượt quá kỳ vọng. Chẳng hạn, leader bảo bạn làm một tool nhỏ cho team vận hành; kết quả bạn xây dựng cả một chuỗi tool, thậm chí phát triển thành một platform và trở thành một project hoàn chỉnh, đây cũng được xem là vượt quá kỳ vọng. Vượt quá kỳ vọng đòi hỏi chúng ta có năng lực làm lớn một việc, tức là nghĩ đến những điều leader chưa nghĩ tới, đồng thời tạo ra giá trị thực tế và thu được business value. Năng lực này thực ra cũng khá quan trọng. Trong công việc, tôi nhận thấy có người có thể làm một việc nhỏ ngày càng lớn, còn có người thì ngược lại; vì vậy những bạn có năng lực đổi mới và thường xuyên vượt quá kỳ vọng rõ ràng sẽ có không gian phát triển lớn hơn.

**Điều này thực ra phụ thuộc khá nhiều vào năng lực cá nhân; hiện tại tôi chưa nghĩ ra đường tắt nào hay, hãy suy nghĩ thêm một bước.**

## Tư duy có hệ thống, xây dựng system một cách hệ thống

Câu này được tôi tổng kết khi thăng chức, đại ý là khi xây dựng system cần có tầm nhìn tổng thể, không bị giới hạn ở một điểm nhỏ nào đó, mà nên có năng lực lập kế hoạch tốt và roadmap phát triển rõ ràng. Chẳng hạn hôm nay thêm một monitoring, ngày mai thêm một alert; những việc này không nên trở thành từng ốc đảo riêng lẻ, mà phải là một bước nhỏ trong giai đoạn một của việc xây dựng stability. Công việc cần làm trong giai đoạn một của việc xây dựng stability là cấu hình alert và rà soát monitoring, bao gồm machine monitoring, system monitoring, business monitoring, data monitoring, v.v.; dự kiến thu được XXX value. Công việc này còn có roadmap tiếp theo: giai đoạn hai của việc xây dựng stability sẽ làm capacity planning và tích hợp load testing; giai đoạn ba sẽ làm các buổi diễn tập downgrade và disaster recovery đa active; giai đoạn bốn sẽ làm... Cảm giác đem lại là người này suy nghĩ rất toàn diện, làm việc có hệ thống và có kế hoạch.

**Thường xuyên chủ động tổng kết và tích lũy, trao đổi nhiều hơn với người khác để hình thành methodology.**

## Nâng cao năng lực soft skill

Năng lực soft skill ở đây thực ra muốn nói đến năng lực về PPT, giao tiếp, trình bày, quản lý thời gian, design, tài liệu, v.v. Nói thật, tôi cho rằng khi đó mình được thăng chức là vì làm PPT tốt hơn một chút... Có thể bình thường mọi người không mấy quan tâm đến những năng lực này, trước đây tôi cũng không coi trọng, cho rằng khá đơn giản, đến lúc dùng thì cứ trực tiếp làm là được; nhưng thực tế có thể không đơn giản như tưởng tượng. Chẳng hạn, công việc PPT + thuyết trình + bảo vệ khi thăng chức thực ra chứa rất nhiều suy nghĩ về chi tiết: chọn nội dung như thế nào, thiết kế bố cục ra sao, dẫn dắt cảm xúc của người nghe thế nào, trả lời câu hỏi của hội đồng ra sao, v.v. Khi thăng chức, tôi từng thấy nhiều bạn sắp xếp nội dung PPT lộn xộn, quá trình thuyết trình cũng không trôi chảy tự nhiên. Dù thực sự đã làm rất nhiều việc, họ lại thiếu nhiều trong cách trình bày, thuộc kiểu biết làm nhưng không biết nói; nếu gặp hội đồng đánh giá từ bộ phận khác không hiểu tình hình thực tế, việc chịu thiệt là điều có thể đoán trước.

**_Mạng nội bộ của công ty thường có một số khóa đào tạo soft skill, có thể tìm cơ hội để chủ động rèn luyện._**

Những chia sẻ trên nhìn chung đều khá tích cực, nhưng xã hội không phải lúc nào cũng tốt đẹp như vậy.. Nội dung bên dưới có xu hướng tiêu cực; những bạn có quan điểm sống đặc biệt đúng đắn hoặc cảm thấy không thoải mái nên bỏ qua.

## Nịnh nọt thật sự cũng có cái hay

Trước khi vào làm, tôi rất phản cảm với chuyện nịnh nọt; lý do ban đầu tôi muốn gia nhập các công ty Internet là vì nghĩ rằng ở đó không có quá nhiều chuyện đối nhân xử thế, nhưng sự thật chứng minh tôi đã sai... Vài ngày sau khi vào làm, đại leader gửi một tin nhắn trong group của phòng ban, ngay sau đó hàng chục tin nhắn kèm biểu tượng ngón tay cái lập tức nối tiếp: đã học hỏi, like, rất hay, xuất sắc. Cảnh tượng ấy nói là cờ đỏ tung bay, trống chiêng vang trời, pháo nổ rộn ràng cũng không quá lời. Ngoài việc kinh ngạc trước năng lực tiếp nhận và tốc độ xử lý thông tin siêu mạnh của mọi người, tôi còn phát hiện ra rằng ngay cả nịnh nọt cũng có đội hình: leader của phòng ban cấp một gửi tin, vài leader của phòng ban cấp hai theo sau, tiếp đó là các trưởng nhóm, cuối cùng là màn cuồng hoan của mọi người. Có lúc tôi nghi ngờ tốc độ nịnh nọt quyết định triển vọng phát triển nghề nghiệp (đúng vậy, hiện giờ tôi không còn nghi ngờ nữa).

Thành thật mà nói, đến giờ tôi vẫn chưa quen nịnh nọt trong group, nhưng cũng không còn phản cảm, có thể nói đã xem việc này như một trò vui. Không phải tôi thiếu tài ăn nói hay năng lực đó (thực tế cũng chẳng cần tài ăn nói gì, mọi người đều đơn giản và trực tiếp); trong một số hoàn cảnh, vì cần khuấy động không khí, tôi cũng có thể nói những lời ngọt ngào, thậm chí sắp xếp cả những lời khen hoa mỹ trích từ thơ văn cổ cho leader. Mà là tôi phát hiện leader trực tiếp của mình cũng không mấy khi nịnh nọt trong group, nên việc ngoài mặt không công khai nịnh nọt của tôi thực ra lại là thuận theo sở thích của leader trong âm thầm...

Nhưng chỉ cần nắm đúng mức độ, chuyện nịnh nọt nhìn chung vẫn có cái hay; cùng lắm là vô ích, ít nhất cũng chẳng có hại gì. Năng lực mọi người gần như nhau, mỗi cơ hội nịnh nọt trong group là một cơ hội để xuất hiện; theo cách nói của một đồng nghiệp, đây gọi là xây dựng ảnh hưởng kỹ thuật cá nhân...

**Muốn nịnh thì cứ nịnh, không muốn nịnh cũng không cần mỉa mai người khác, Respect Greatness.**

## Thực chiến tranh cãi và đổ lỗi không bao giờ vắng mặt

Ở đâu có con người, ở đó có giang hồ. Dù phần lớn những người làm kỹ thuật không quá thâm sâu, những chuyện đau đầu như tranh cãi, đổ lỗi, tranh công và giành việc về cơ bản cũng không vắng mặt; thậm chí tôi từng thấy có người công khai gửi email hàng loạt để tranh cãi... Chủ đề này liên quan đến một số thông tin nhạy cảm nên tôi không nói thêm, hơn nữa người ở cấp bậc thấp như chúng ta cũng không thường gặp những chuyện này. Chỉ muốn nhắc mọi người một câu: đi làm sớm muộn cũng sẽ hóng được chuyện ở phương diện này, đến lúc đó hãy để ý một chút.

**Hãy chú ý một chút; chúng ta không bắt nạt người khác, nhưng cũng không thể dễ dàng để người khác bắt nạt mình.**

## Đừng để những lời hứa hẹn viển vông che mờ mắt

Nói thật, cá nhân tôi khá phản cảm với những hành vi rót canh gà, kích động tinh thần, nói về ước mơ và kể chuyện phấn đấu; năm 9102 sắp kết thúc rồi mà bộ \*\*\*trị này vẫn còn thịnh hành, thật không biết nên thấy nực cười hay đáng buồn. Tất nhiên, bản thân những từ này không có vấn đề gì, nhưng những điều đó nên xuất phát từ tự thúc đẩy, không nên trở thành một kiểu push mạnh từ bên ngoài. Câu “Tôi phải nỗ lực phấn đấu” theo tôi là bình thường, nhưng câu “Bạn phải nỗ lực phấn đấu” thì ít nhiều có vẻ kỳ lạ; nỗ lực phấn đấu để cổ đông công ty làm giàu sao? Đặc biệt khi tiền không được trả đủ, những hành vi này chẳng khác nào giở trò lưu manh. Chúng ta cần giữ nhận thức tỉnh táo trước những màn vẽ bánh của leader, phân tích lý trí và đưa ra quyết định. Chẳng hạn khi cảm thấy tiền lương chưa đủ (hoặc level quá thấp, tương tự), có thể có một số tình huống sau:

1. leader chưa chú ý đến thực tế là lương của bạn thấp
2. leader biết sự thật này, nhưng không biết nhu cầu tăng lương của bạn mạnh đến mức nào
3. leader biết bạn có nhu cầu tăng lương, nhưng cho rằng năng lực của bạn chưa đủ
4. leader biết bạn có nhu cầu tăng lương và năng lực của bạn cũng đủ, nhưng không muốn tăng lương cho bạn
5. leader muốn tăng lương cho bạn, cũng đã phản hồi lên cấp trên và cố gắng xin, nhưng không có resource

Lúc này việc chúng ta cần làm là phản hồi lên cấp trên và trao đổi xác nhận với leader. Nếu là trường hợp 1 và 2, trao đổi có thể loại bỏ sai lệch thông tin. Nếu là trường hợp 3, cần thảo luận tùy tình huống. Nếu là trường hợp 4 và 5, đã có thể cân nhắc rút lui. Với những chuyện này cũng không cần phàn nàn; phàn nàn không giải quyết được vấn đề nào. Việc chúng ta cần làm là nỗ lực nâng cao năng lực cá nhân, duy trì năng lực cạnh tranh cá nhân, chờ một thời điểm thích hợp rồi chuyển việc là xong.

**Luôn sẵn sàng, có kỹ thuật trong tay thì chẳng có gì đáng sợ; ngày nào làm việc không vui thì cứ nghỉ việc chuyển công ty.**

## Học cách “đóng gói”

Nói thẳng ra, điều này có nghĩa là phải biết “chém”. Tôi không nhớ đã đọc ở đâu rằng biết nói, biết viết và giỏi làm là ba yêu cầu lớn đối với người đi làm. Biết nói rất quan trọng; biết nói mới có thể xin được project, kéo được resource và tuyển được người. Cùng một việc, người khác nhau nói ra sẽ có hiệu quả hoàn toàn khác nhau. Chẳng hạn tôi làm một tool nhỏ rồi deploy, tôi chỉ có thể nói sự thật cơ bản; nhưng để leader mô tả thì sẽ thành: xây dựng một đầu mối tool XXX, cải tiến hệ sinh thái hoàn chỉnh XXX, hình thành business closed loop XXX. Anh bạn, tôi phục rồi, đưa hết tiền cho bạn cũng được. Theo quan sát của tôi, công ty Internet nào cũng có vài từ như đầu mối, hệ sinh thái, closed loop, đồng bộ, rà soát, iteration, owner consciousness, v.v. Việc chúng ta cần làm là đọc kỹ và học thuộc toàn văn, à không, ghi nhớ và sử dụng thành thạo.

Đây là cách “đóng gói” sự việc; “đóng gói” con người cũng tương tự, đặc biệt trong những hoàn cảnh thi cử như thăng chức và phỏng vấn, nơi quy trình ngắn và chỉ có một cơ hội, nên việc “đóng gói” càng quan trọng. Tôi không nói thêm về thăng chức và phỏng vấn, vì trong đó có quá nhiều đạo và thuật.. Những tình huống dưới đây được chắt lọc từ cuộc trò chuyện với interviewer của một công ty trong quá trình phỏng vấn, mọi người có thể cảm nhận một chút:

1. Đằng sau chúng tôi là một thị trường trị giá bốn năm trăm tỷ USD...
2. Tôi từng phụ trách một system có lượng truy cập mỗi ngày ở cấp trăm tỷ...
3. Làm việc hai năm mà đạt được mức độ này là khá tốt...
4. Môi trường kỹ thuật của quý công ty khá tốt, triển vọng phát triển business cũng rất rộng mở...
5. À, hai bên cũng như nhau...
6. Ừm, ngưỡng mộ đã lâu...

Cuộc đời như một vở kịch, tất cả đều dựa vào kỹ năng diễn xuất

**Có thể xem nhiều PPT của leader, nghe nhiều hơn các buổi báo cáo và thuyết trình lên cấp trên của sếp.**

## Lựa chọn hay nỗ lực quan trọng hơn?

Còn phải hỏi sao, đương nhiên là lựa chọn. Trước một lựa chọn hoàn hảo, nỗ lực chẳng đáng một xu; một người bạn cấp ba nhiều năm không liên lạc với tôi năm nay đã gõ chuông tại Quảng trường Thời đại... Nhưng những trường hợp như vậy quá ít, chi phí ngẫu nhiên để đưa ra lựa chọn hoàn hảo quá cao và tính bất định quá lớn. Với phần lớn các bạn mới tốt nghiệp, khả năng phán đoán về ngành chưa đủ trưởng thành, cũng chưa đủ chính xác khi đánh giá năng lực bản thân và độ khó của việc khởi nghiệp; lúc này rủ vài người đi khởi nghiệp có vẻ rủi ro quá cao. Tôi cho rằng một con đường ổn thỏa hơn là trước tiên gia nhập một công ty có quy mô hơi lớn, tìm một leader tốt, dựa vào người giỏi để nâng cao năng lực cá nhân. Platform tốt cộng với người dẫn dắt tốt, lại thêm nỗ lực cá nhân, tốc độ cất cánh như vậy là đủ rồi. Sau khi tích lũy được một số mối quan hệ và vốn, hiểu sâu về thị trường và nhu cầu, đồng thời tự tin hơn, có thể cân nhắc chuyện khởi nghiệp.

## Lời cuối

Vốn còn muốn chia sẻ một số câu chuyện trong cuộc sống, nhưng nhận ra bài đã dài như vậy rồi, nên tạm dừng ở đây. Một số tổng kết và đề xuất được viết ở trên thực ra tôi cũng chưa làm tốt, vẫn cần tiếp tục cố gắng và cùng mọi người khích lệ lẫn nhau. Ngoài ra, do góc nhìn cá nhân có hạn, một số quan điểm trong đó cũng không đảm bảo là phổ quát hay chính xác; có thể sau vài năm làm việc nữa những quan điểm này sẽ thay đổi. Hoan nghênh mọi người trao đổi với tôi~ (đổ lỗi thành công)

Cuối cùng chúc mọi người đều tìm được công việc mình yêu thích, vui vẻ làm việc, hạnh phúc trong cuộc sống, thỏa sức phát huy và đạt được nhiều thành tựu.

<!-- @include: @article-footer.snippet.md -->
