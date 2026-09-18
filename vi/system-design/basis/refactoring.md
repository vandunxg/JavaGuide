---
title: Hướng dẫn refactoring code
description: Hướng dẫn thực hành refactoring code, bao quát định nghĩa refactoring, nguyên tắc refactoring, nhận diện code smell cùng các kỹ thuật refactoring và best practice thường dùng.
category: Chất lượng code
head:
  - - meta
    - name: keywords
      content: refactoring code,kỹ thuật refactoring,nguyên tắc refactoring,Design Patterns,SOLID,code smell,khả năng bảo trì,Unit Testing
---

Thời gian trước, tôi đọc lại cuốn [Refactoring: Cải thiện thiết kế hiện có của code](https://book.douban.com/subject/30468597/) và học hỏi được rất nhiều điều. Vì vậy, tôi viết một bài ngắn để chia sẻ quan điểm của mình về refactoring.

![](https://oss.javaguide.cn/github/javaguide/image-20220311155746549.png)

## Refactoring là gì?

Cuốn sách kinh điển về refactoring [Refactoring: Cải thiện thiết kế hiện có của code](https://book.douban.com/subject/30468597/) đưa ra định nghĩa về refactoring từ hai góc độ:

> - Refactoring (danh từ): Điều chỉnh cấu trúc bên trong của phần mềm nhằm tăng khả năng dễ hiểu và giảm chi phí sửa đổi, trong khi không thay đổi hành vi có thể quan sát của phần mềm.
> - Refactoring (động từ): Sử dụng một loạt kỹ thuật refactoring để điều chỉnh cấu trúc mà không thay đổi hành vi có thể quan sát của phần mềm.

Nói theo cách gần gũi hơn với engineer: **refactoring là điều chỉnh cấu trúc theo từng bước nhỏ, thông qua một loạt thay đổi, để code dễ hiểu và dễ sửa đổi hơn mà không thay đổi hành vi có thể quan sát của phần mềm.** Design Patterns và các nguyên tắc thiết kế phần mềm có thể định hướng cho refactoring, còn automated test là nền tảng bảo đảm quan trọng của refactoring.

Các nguyên tắc thiết kế phần mềm định hướng cách chúng ta tổ chức và chuẩn hóa code. Đồng thời, refactoring cũng nhằm thiết kế phần mềm đáp ứng tối đa các nguyên tắc thiết kế phần mềm.

Cốt lõi của refactoring đúng cách là **bước đi nhất định phải nhỏ; mỗi bước refactoring không được ảnh hưởng đến hoạt động bình thường của phần mềm và có thể dừng bất cứ lúc nào.**

**Một số Design Patterns thường gặp:**

![Các Design Patterns thường gặp](https://oss.javaguide.cn/github/javaguide/system-design/basis/common-design-patterns.png)

Để xem tổng hợp Design Patterns đầy đủ hơn, bạn có thể tham khảo dự án open source **[java-design-patterns](https://github.com/iluwatar/java-design-patterns)**.

**Một số nguyên tắc thiết kế phần mềm thường gặp:**

![Các nguyên tắc thiết kế phần mềm thường gặp](https://oss.javaguide.cn/github/javaguide/system-design/basis/programming-principles.png)

Để xem tổng hợp đầy đủ hơn về các nguyên tắc thiết kế, bạn có thể tham khảo hai dự án open source **[java-design-patterns](https://github.com/iluwatar/java-design-patterns)** và **[hacker-laws-zh](https://github.com/nusr/hacker-laws-zh)**.

## Vì sao cần refactoring?

Khi giới thiệu định nghĩa refactoring ở trên, tôi đã trình bày lợi ích của refactoring từ góc độ khá trừu tượng: mục đích chính của refactoring là giúp code dễ hiểu hơn và giảm chi phí sửa đổi về sau.

Nếu đặt trong một dự án thực tế, refactoring cụ thể có thể mang lại lợi ích gì cho chúng ta?

1. **Giúp code dễ hiểu hơn**: Thông qua các biện pháp như thêm comment, chuẩn hóa quy tắc đặt tên và tối ưu logic, code sẽ dễ được hiểu hơn;
2. **Tránh code bị thoái hóa**: Loại bỏ code có code smell thông qua refactoring;
3. **Hiểu sâu hơn về code**: Quá trình refactoring code sẽ giúp bạn hiểu sâu hơn về một phần code;
4. **Phát hiện bug tiềm ẩn**: Đúng vậy, nhiều bug tiềm ẩn được chúng ta phát hiện trong quá trình refactoring;
5. ……

Sau khi xem những lợi ích của refactoring được giới thiệu ở trên, bạn sẽ nhận ra mục tiêu cuối cùng của refactoring là **nâng cao tốc độ và chất lượng phát triển phần mềm**.

Refactoring không làm chậm tốc độ phát triển phần mềm. Ngược lại, nếu chất lượng code và thiết kế phần mềm kém, tốc độ phát triển sẽ ngày càng chậm khi chúng ta muốn thêm tính năng mới. Cuối cùng, thậm chí chúng ta còn có thể muốn viết lại toàn bộ system.

![](https://oss.javaguide.cn/github/javaguide/bad&good-design.png)

Cuốn sách [Refactoring: Cải thiện thiết kế hiện có của code](https://book.douban.com/subject/30468597/) viết:

> Mục đích duy nhất của refactoring là giúp chúng ta phát triển nhanh hơn và tạo ra nhiều giá trị hơn với ít công sức hơn.

## Tối ưu performance có phải là refactoring không?

Mục đích của refactoring là nâng cao khả năng đọc, khả năng bảo trì và tính linh hoạt của code. Refactoring tập trung vào cấu trúc bên trong của code: làm thế nào để developer dễ hiểu code hơn, làm thế nào để việc phát triển và bảo trì tính năng về sau hiệu quả hơn. Còn tối ưu performance nhằm giúp code chạy nhanh hơn và chiếm ít tài nguyên hơn. Nó tập trung vào biểu hiện bên ngoài của chương trình: làm thế nào để giảm thời gian phản hồi, giảm mức tiêu thụ tài nguyên và nâng cao throughput của hệ thống. Hai việc này có vẻ đối lập, nhưng trên thực tế mục tiêu của chúng thống nhất với nhau, đều nhằm nâng cao chất lượng tổng thể của phần mềm.

Trong quá trình phát triển thực tế, cách làm lý tưởng là trước tiên **bảo đảm khả năng đọc và khả năng bảo trì của code**, sau đó lựa chọn kỹ thuật tối ưu performance phù hợp theo nhu cầu thực tế. Thiết kế phần mềm tốt không phải là mù quáng theo đuổi performance tối đa, mà là tìm được sự cân bằng giữa khả năng bảo trì và performance. Bằng cách này, chúng ta có thể xây dựng system vừa **dễ quản lý** vừa có **performance tốt**.

## Khi nào nên thực hiện refactoring?

Refactoring có thể được thực hiện bất cứ lúc nào trong quá trình phát triển, chỉ cần tùy tình hình mà làm, không cần phân bổ riêng một hoặc hai ngày chỉ để thực hiện refactoring.

### Trước khi commit code

Cuốn sách [Refactoring: Cải thiện thiết kế hiện có của code](https://book.douban.com/subject/30468597/) giới thiệu khái niệm **quy tắc khu cắm trại**:

> Khi lập trình, cần tuân theo quy tắc khu cắm trại: bảo đảm codebase khi bạn rời đi chắc chắn khỏe mạnh hơn lúc bạn đến.

Ý tưởng cốt lõi của khái niệm này thực ra rất đơn giản: trước khi commit code, hãy dành chút thời gian suy nghĩ xem commit lần này làm code của project khỏe mạnh hơn, bị thoái hóa hơn hay không thay đổi gì.

Chỉ khi mỗi thành viên trong team bảo đảm commit của mình không làm code của project thoái hóa hơn, code của project mới phát triển theo hướng lành mạnh.

**Khi rời khỏi khu cắm trại (code của project), đừng để lại rác (code smell)! Hãy cố gắng bảo đảm khu cắm trại sạch sẽ hơn!**

### Sau và trước khi phát triển tính năng mới

Sau khi phát triển một tính năng mới, chúng ta nên nhìn lại xem có điểm nào có thể cải thiện hay không. Trước khi thêm một tính năng mới, chúng ta có thể suy nghĩ xem mình có thể refactor code để việc phát triển tính năng mới dễ dàng hơn hay không.

Việc phát triển một tính năng mới không nên chỉ dừng ở việc xác minh tính năng đã pass; chúng ta cũng nên cố gắng bảo đảm chất lượng code.

Có một phép ẩn dụ về hai chiếc mũ: trước khi phát triển tính năng mới, tôi nhận ra refactoring có thể giúp việc phát triển tính năng mới dễ dàng hơn, nên tôi đội chiếc mũ refactoring. Sau khi refactoring, tôi đổi lại chiếc mũ ban đầu và tiếp tục phát triển. Sau khi hoàn thành tính năng mới, tôi lại nhận ra code của mình khó hiểu, nên tiếp tục đội chiếc mũ refactoring. Trạng thái phát triển tốt là chuyển đổi qua lại giữa refactoring và phát triển tính năng mới như vậy.

![hai chiếc mũ](https://oss.javaguide.cn/github/javaguide/refractor-two-hats.png)

### Sau Code Review

Code Review có thể nâng cao rất hiệu quả chất lượng tổng thể của code. Nó giúp chúng ta phát hiện code smell và những vấn đề có thể tồn tại trong code. Ngoài ra, Code Review còn giúp các developer khác trong team hiểu module business do bạn phụ trách, từ đó tránh hiệu quả rủi ro single point do nhân sự.

Sau một lần Code Review, code của bạn có thể nhận được rất nhiều đề xuất cải thiện.

### Refactoring kiểu nhặt rác

Khi phát hiện code có code smell (rác), nếu không muốn dừng công việc đang làm nhưng cũng không muốn bỏ mặc rác, chúng ta có thể làm như sau:

- Nếu phần rác này dễ refactor, chúng ta có thể refactor ngay.
- Nếu phần rác này không dễ refactor, chúng ta có thể ghi lại trước, rồi quay lại refactor sau khi hoàn thành task hiện tại.

### Khi đọc và tìm hiểu code

Những người làm development hẳn đều rất đồng cảm: chúng ta thường phải đọc code do người khác trong team viết và cũng thường phải đọc code mình đã viết trước đây. Thời gian đọc code thường nhiều hơn rất nhiều so với thời gian viết code.

Khi đọc và tìm hiểu code, nếu phát hiện code smell, chúng ta có thể refactor code đó.

Ví dụ, khi đọc một đoạn code do Trương Tam viết, bạn nhận thấy logic của đoạn code này quá phức tạp và khó hiểu, đồng thời bạn có cách viết tốt hơn, thì có thể refactor logic trong đoạn code của Trương Tam.

## Những lưu ý khi refactoring

### Automated test là lưới bảo vệ cho refactoring

**Automated test có thể tạo sự tự tin cho refactoring và giảm chi phí refactoring. Unit test thường là tầng phản hồi nhanh nhất; integration test, acceptance test và các loại test khác cũng có thể cùng tạo thành lưới bảo vệ. Chúng ta cần coi trọng test code như production code.**

Ngoài ra, cần nhấn mạnh thêm một điểm: continuous integration cũng phụ thuộc vào automated test nhanh và đáng tin cậy. Sau khi service continuous integration tự động build code mới, nó sẽ tự động chạy test để phát hiện lỗi code.

**Thế nào mới được tính là unit test?** Có rất nhiều định nghĩa trên Internet, chúng khá trừu tượng và dễ khiến người đọc bối rối. Theo tôi, định nghĩa về unit test chủ yếu phụ thuộc vào project của bạn; thậm chí một function hoặc một class cũng có thể được xem là một unit. Ví dụ, chúng ta viết một method tính tỷ suất lợi nhuận cổ phiếu cá nhân, rồi viết riêng một unit test để xác minh tính đúng đắn của method đó. Hoặc code có một class chuyên phụ trách data masking, chúng ta cũng viết riêng một unit test cho class này để xác minh việc data masking có đúng như kỳ vọng hay không.

**Unit test cũng cần được refactor hoặc sửa đổi.** Cuốn [Clean Code: Nghệ thuật viết code sạch](https://book.douban.com/subject/4199741/) viết:

> Test code cần được sửa đổi theo sự phát triển của production code. Nếu test không thể duy trì sự sạch sẽ, nó sẽ ngày càng khó sửa đổi.

### Đừng refactor chỉ vì muốn refactor

**Refactoring nhất định phải mang lại giá trị cho project!** Trong một số trường hợp, chúng ta không nên refactor:

- Sau khi học một Design Patterns hoặc best practice nào đó, bất chấp tình hình thực tế của project mà cố tình áp dụng vào project (tránh cargo cult programming);
- Khi tiến độ project khá gấp, refactor code bên dưới của một API được project gọi (sau refactoring, việc project gọi API này cũng không mang lại giá trị gì);
- Viết lại dễ dàng và đỡ tốn công hơn refactoring;
- ……

### Tuân theo phương pháp

Cuốn sách [Refactoring: Cải thiện thiết kế hiện có của code](https://book.douban.com/subject/30468597/) liệt kê một số code smell thường gặp (chẳng hạn như code trùng lặp, function quá dài) và các kỹ thuật refactoring (chẳng hạn như Extract Method, Extract Variable, Extract Class). Chúng ta nên dành thời gian học các kiến thức lý thuyết liên quan đến refactoring và thực hành những lý thuyết này trong code.

## Luyện tập refactoring thế nào?

Ngoài việc luyện tập và nâng cao kỹ năng refactoring trong quá trình refactor code của project, bạn còn có thể sử dụng các cách sau:

- [Tôi suy nghĩ gì khi refactor](https://mp.weixin.qq.com/s/pFaFKMXzNCOuW2SD9Co40g): Bài viết này của Zhuanzhuan Technology tổng hợp các bối cảnh và phương thức refactoring thường gặp.
- [Bài tập thực chiến về refactoring](https://linesh.gitbook.io/refactoring/): Hướng dẫn bạn học refactoring từng bước thông qua một số case nhỏ!
- [Website học Design Patterns + refactoring](https://refactoringguru.cn/): Học online miễn phí về refactoring code, Design Patterns và nguyên tắc SOLID (Single Responsibility, Open-Closed, Liskov Substitution, Interface Segregation và Dependency Inversion).
- [Tutorial refactoring code trong tài liệu chính thức của IDEA](https://www.jetbrains.com/help/idea/refactoring-source-code.html#popular-refactorings): Hướng dẫn cách sử dụng IDEA để thực hiện refactoring.

## Tài liệu tham khảo

- [Đọc lại “Refactoring” - ThoughtWorks Insight - 2020](https://insights.thoughtworks.cn/reread-refactoring/): Giới thiệu chi tiết các điểm chính của refactoring, chẳng hạn như refactoring từng bước nhỏ và refactoring kiểu nhặt rác, chủ yếu tập trung vào các khái niệm refactoring.
- [Các kỹ thuật refactoring code thường gặp - VectorJin - 2021](https://juejin.cn/post/6954378167947624484): Giới thiệu cách thực hiện refactoring từ các góc độ nguyên tắc thiết kế phần mềm, Design Patterns, phân tầng code và quy tắc đặt tên, thiên về thực hành hơn.

<!-- @include: @article-footer.snippet.md -->
