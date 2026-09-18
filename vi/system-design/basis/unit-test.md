---
title: Unit test là gì? Nên thực hiện thế nào?
description: Hướng dẫn nhập môn unit test, bao gồm khái niệm unit test, kỹ thuật Mock và Stub, test pyramid và cách sử dụng framework kiểm thử JUnit.
category: Chất lượng code
head:
  - - meta
    - name: keywords
      content: Unit Testing,Mock,Stub,Fake,test pyramid,testability,TDD,JUnit
---

> Bài viết này được biên soạn và hoàn thiện lại từ bài viết [Bàn về lý do viết unit test - Kiến Bàn Phím - 2016](https://www.jianshu.com/p/fa41fb80d2b8).

## Unit test là gì?

Wikipedia giới thiệu về unit test như sau:

> Trong lập trình máy tính, unit test (Unit Testing) là công việc kiểm thử tính đúng đắn của các module chương trình (đơn vị nhỏ nhất trong thiết kế phần mềm).
>
> Một unit là **bộ phận nhỏ nhất có thể kiểm thử** của ứng dụng. Trong lập trình thủ tục, một unit là một chương trình, hàm, thủ tục riêng lẻ, v.v.; với lập trình hướng đối tượng, unit nhỏ nhất là method, bao gồm method trong base class (superclass), abstract class hoặc derived class (subclass).

Do mỗi unit có logic độc lập, khi thực hiện unit test, để cô lập các dependency bên ngoài và đảm bảo các dependency này không ảnh hưởng đến việc xác minh logic, chúng ta thường sử dụng Fake, Stub và Mock.

Để hiểu các khái niệm Fake, Mock và Stub, bạn có thể xem bài viết [Làm rõ các khái niệm Fakes, Mocks và Stubs trong kiểm thử - Vương Hạ Mời Nguyệt Hùng - 2018](https://zhuanlan.zhihu.com/p/26942686).

## Vì sao cần unit test?

### Bảo vệ quá trình refactor

Trong bài viết [refactor](./refactoring.md), tôi đã viết:

> Unit test có thể mang lại sự tự tin cho việc refactor và giảm chi phí refactor. Chúng ta cần coi trọng unit test như code production.

Mỗi developer đều sẽ trải qua refactor. Việc code bị hỏng sau refactor không hề hiếm; rất có thể bạn chỉ sửa một method rất đơn giản nhưng đã khiến hệ thống xuất hiện một lỗi nghiêm trọng.

Unit test có thể kịp thời phát hiện một phần lỗi logic, giảm rủi ro phát sinh regression do refactor. Viết xong một class thì bổ sung unit test cho class đó; viết class thứ hai thì tiếp tục bổ sung unit test... Tuy nhiên, việc mỗi class vượt qua test riêng lẻ không có nghĩa là khi kết hợp chúng lại chắc chắn không có vấn đề. Các vấn đề về interface contract, configuration, database, network và transaction vẫn cần được phát hiện thông qua integration test, end-to-end test và các phương thức khác.

### Nâng cao chất lượng code

Do mỗi unit có logic độc lập, khi thực hiện unit test cần cô lập dependency bên ngoài để đảm bảo các dependency này không ảnh hưởng đến việc xác minh logic. Việc tách các dependency sẽ thúc đẩy dự án phân tách component, sắp xếp quan hệ dependency của dự án và giảm coupling của code ở mức độ lớn hơn. Code được viết như vậy sẽ dễ bảo trì, dễ mở rộng hơn, từ đó nâng cao chất lượng code.

### Giảm bug

Một cỗ máy được cấu thành từ nhiều linh kiện nhỏ. Nếu một linh kiện bị hỏng, máy sẽ gặp sự cố khi vận hành. Chỉ khi đảm bảo mỗi linh kiện đều đúng specification theo bản thiết kế thì máy mới có thể vận hành bình thường.

Một dự án có thể thực hiện unit test sẽ chia business và function thành các bộ phận có quy mô nhỏ hơn, logic độc lập, gọi là unit. Mục tiêu của unit test là xác minh mỗi unit có hoạt động theo kỳ vọng hay không, từ đó giảm xác suất xuất hiện lỗi logic cục bộ và regression. Việc toàn bộ project có thể chạy đúng hay không vẫn cần các tầng test khác cùng xác minh.

### Nhanh chóng định vị bug

Nếu chương trình có bug, sau khi chạy toàn bộ unit test, bạn có thể nhanh chóng định vị đoạn code thực thi tương ứng bằng cách tìm test không pass. Sau khi sửa code, chạy unit test tương ứng; nếu vẫn không pass thì tiếp tục sửa và chạy test... cho đến khi **test pass**.

### Continuous integration phụ thuộc vào automated test

Continuous integration cần các automated test nhanh và đáng tin cậy. Sau khi service continuous integration tự động build code mới, nó sẽ tự động chạy unit test, integration test và các test khác để phát hiện lỗi code. Unit test thường là tầng phản hồi nhanh nhất trong số đó.

## Ai buộc bạn viết unit test?

### Yêu cầu từ leader

Một số leader giàu kinh nghiệm ít nhiều đều yêu cầu team viết unit test. Với những teammate đã có kinh nghiệm làm việc nhất định, yêu cầu này khá hợp lý; còn với người ít kinh nghiệm hoặc sinh viên mới tốt nghiệp thì có lẽ sẽ khổ sở: code còn chưa viết tốt mà đã phải viết unit test, are you kidding me?

Đào tạo người mới cách sử dụng unit test là một nhiệm vụ khó khăn. Người mới chưa hình thành phong cách code, cũng chưa biết unit test quan trọng đến mức nào. Việc bắt buộc viết unit test sẽ khiến họ bối rối và không thể viết code theo cách nghĩ của mình.

### Các cao thủ đều viết unit test

Nhiều project open source nổi tiếng trên thế giới có số lượng unit test rất lớn. Ví dụ như [retrofit](https://link.jianshu.com?t=https://github.com/square/retrofit/tree/master/retrofit/src/test/java/retrofit2), [okhttp](https://link.jianshu.com?t=https://github.com/square/okhttp/tree/master/okhttp-tests/src/test/java/okhttp3), [butterknife](https://link.jianshu.com?t=https://github.com/JakeWharton/butterknife/tree/master/butterknife-compiler/src/test/java/butterknife)... Các cao thủ nước ngoài đều viết unit test, chúng ta cũng viết thôi!

Nhiều độc giả có suy nghĩ như vậy và ban đầu tràn đầy nhiệt huyết. Nhưng khi thực sự phải viết unit test cho project của mình, họ lại gặp vô vàn khó khăn; nguyên nhân lớn là project không thân thiện với unit test. Cuối cùng, họ chỉ có thể viết unit test cho một số utility class không quá quan trọng. Lâu dần, nguyện vọng tốt đẹp ban đầu cũng không thành hiện thực.

### Giữ thể diện

Đều là những người đã có vài năm kinh nghiệm mà ngày nào cũng bị đồng nghiệp kiểm thử đuổi theo để báo bug, không thấy ngại sao? Dành thêm một chút thời gian viết unit test, đảm bảo không có bug cấp thấp, lại có thể thể hiện phong thái cao thủ, vậy thì tại sao không làm?

### Chột dạ

Tác giả cũng là người không quá tin tưởng code của mình, luôn cảm thấy ở đâu đó có thể đột nhiên xuất hiện một bug kỳ lạ, cũng sợ người khác vô tình sửa code của mình (hội chứng hoang tưởng bị hại), rồi thấp thỏm lo lắng khi version mới được đưa lên production... Dành một chút thời gian viết unit test, thỉnh thoảng chạy test để đảm bảo logic ban đầu không có vấn đề, ít nhất cũng có thể ngủ ngon hơn.

## TDD (Test-Driven Development)

### TDD là gì?

TDD là Test-Driven Development (phát triển hướng kiểm thử), một practice và kỹ thuật cốt lõi của agile development, đồng thời cũng là một phương pháp luận thiết kế.

Nguyên lý của TDD là viết code test case trước khi phát triển code chức năng, sau đó viết code chức năng tương ứng với test case để code có thể pass.

Nhịp độ của TDD: “Red - Green - Refactor”.

![](https://static001.geekbang.org/resource/image/09/7f/090e1fc6aff08b4aa66376f776c2337f.png)

Do TDD yêu cầu rất cao ở developer và khác với tư duy phát triển truyền thống nên việc triển khai khá khó khăn.

Trong mắt nhiều người, TDD không thực tế. Một mặt, họ không hiểu ý nghĩa của việc test “dẫn dắt” development; nhưng quan trọng hơn là họ hiếm khi phân rã task. Phân rã task là điểm then chốt để thực hiện TDD tốt. Chỉ khi phân rã task đến mức có thể test thì mới có thể viết test một cách có mục tiêu.

### Phân tích ưu, nhược điểm của TDD

Test-Driven Development có cả ưu điểm và nhược điểm. Vì mỗi test case đều xuất phát từ requirement, hay nói cách khác là phân rã một requirement lớn thành một số requirement nhỏ để viết test case, nên sau khi viết test case, code thực thi do developer viết phải đáp ứng test case. Nếu test không pass thì sửa code thực thi cho đến khi test case pass.

**Ưu điểm**:

1. Giúp bạn sắp xếp requirement và hệ thống hóa tư duy;
2. Giúp bạn thiết kế interface hợp lý hơn (nếu chỉ tưởng tượng thì rất dễ thiết kế ra thứ rất tệ);
3. Giảm xác suất xuất hiện bug trong code;
4. Nâng cao hiệu suất development (với điều kiện sử dụng TDD đúng cách và thành thạo).

**Nhược điểm**:

1. Rất ít người có thể sử dụng TDD tốt; nhìn có vẻ đơn giản nhưng thực tế ngưỡng yêu cầu rất cao;
2. Thông thường cần đầu tư nhiều resource development hơn (thời gian và công sức);
3. Vì test case được viết trước khi thiết kế code nên rất có thể sẽ hạn chế thiết kế tổng thể code của developer;
4. Có thể gây ra sự bất mãn ở developer. Tôi cho rằng đây là điểm khá nghiêm trọng, dù unit test mang lại cho chúng ta rất nhiều lợi ích, bởi không phải ai cũng thích unit test.

Đọc thêm: [Mở TDD theo đúng cách như thế nào? - Trần Thiên - 2017](https://zhuanlan.zhihu.com/p/24997923).

## Nên chọn framework unit test và công cụ Mock như thế nào?

Đối với unit test, JUnit và Spock là các testing framework; còn Mockito, PowerMock, JMockit, TestableMock chủ yếu được dùng để tạo Mock và các test double khác.

JUnit gần như là lựa chọn mặc định, nhưng không cung cấp khả năng Mock nên thường cần kết hợp thêm các công cụ Mock như Mockito. Spock đồng thời cung cấp testing specification và khả năng Mock.

Nên chọn JUnit kết hợp với Mockito hay chọn Spock? Ở đây tôi thực hiện một số phân tích so sánh đơn giản:

- Spock 2.4 có thể sử dụng `SpyStatic()` kết hợp với Mock Maker hỗ trợ static method để mock static method của Java và Groovy; Mockito từ phiên bản 3.4.0 cũng hỗ trợ static method. Bạn có thể xem [ghi chú phát hành Spock 2.4](https://spockframework.org/spock/docs/2.4/release_notes.html) và [tài liệu chính thức của Mockito](https://javadoc.io/doc/org.mockito/mockito-core/latest/org.mockito/org/mockito/Mockito.html).
- Spock dựa trên Groovy, code test viết ra rõ ràng và dễ đọc hơn, tương đối quy củ (có sẵn quy ước cấu trúc test phổ biến given-when-then). Mockito không có quy ước cấu trúc cụ thể, project team cần tự thống nhất một quy ước hoặc tuân thủ các practice tốt về code test. Thông thường, với cùng một test case, code của Spock ngắn gọn hơn.
- Mockito được sử dụng rộng rãi hơn, ổn định và đáng tin cậy. Ngoài ra, Mockito là công cụ Mock được tích hợp mặc định trong SpringBoot Test.

JUnit kết hợp với Mockito và Spock đều là những lựa chọn rất tốt. Tương đối mà nói, JUnit kết hợp với Mockito có tính phù hợp cao hơn.

## Tổng kết

Unit test thực sự mang lại cho bạn rất nhiều lợi ích, nhưng không thể cảm nhận ngay lập tức. Cũng giống như mua bảo hiểm bệnh hiểm nghèo: đóng rất nhiều phí bảo hiểm nhưng nếu không bệnh không đau thì mười mấy năm, thậm chí vài chục năm cũng không dùng đến; tốt nhất là cả đời không cần yêu cầu bồi thường, sức khỏe bình an mới là điều quan trọng nhất. Unit test cũng vậy: viết test có thể mang lại sự yên tâm và là một hình thức bảo vệ code; nếu có bug thì phát hiện càng sớm càng tốt, không có bug thì càng tốt. Không thể nói rằng “viết nhiều unit test như vậy mà cuối cùng không phát hiện được bug, thật lãng phí thời gian”, đúng không?

Dưới đây là một số đề xuất cá nhân về unit test:

> - Code càng quan trọng thì càng phải viết unit test;
> - Nếu code không thể thực hiện unit test, hãy suy nghĩ nhiều hơn về cách cải thiện thay vì từ bỏ;
> - Vừa viết business code vừa viết unit test, không phải đợi hoàn thành toàn bộ feature mới viết;
> - Suy nghĩ nhiều hơn về cách cải thiện và đơn giản hóa code test.
> - Code test cần được refactor hoặc sửa đổi cùng với sự phát triển của code production. Nếu không thể giữ code test sạch sẽ thì code sẽ ngày càng khó sửa.

Là một programmer giàu kinh nghiệm, viết unit test chủ yếu là **chịu trách nhiệm với code của chính mình**. Code có test case sẽ dễ đọc hơn đối với người khác; sau này khi người khác tiếp quản code của bạn, họ cũng có thể yên tâm sửa đổi.

**Thực hành bằng cách gõ nhiều code và trao đổi nhiều với các engineer có kinh nghiệm viết unit test**, bạn sẽ nhận ra lợi ích từ việc viết unit test còn lớn hơn.

<!-- @include: @article-footer.snippet.md -->
