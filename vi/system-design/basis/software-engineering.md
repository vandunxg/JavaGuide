---
title: Hướng dẫn ngắn gọn về Software Engineering
description: Giải thích chi tiết kiến thức cơ bản về Software Engineering, bao quát software crisis, software development process, Waterfall Model, Agile Development và các khái niệm cốt lõi khác của Software Engineering.
category: System Design
head:
  - - meta
    - name: keywords
      content: Software Engineering,software crisis,software development process,Waterfall Model,Agile Development,requirements analysis,software life cycle,engineering methods
---

Hầu hết người làm software development đều bỏ qua một số khái niệm cơ bản và nền tảng nhất trong software development. Tuy nhiên, những khái niệm này lại rất quan trọng đối với software development, giống như nền móng của software development. Đây cũng là lý do tôi viết bài này.

## Software Engineering là gì?

Năm 1968, NATO (Tổ chức Hiệp ước Bắc Đại Tây Dương) đưa ra thuật ngữ **software crisis** (**Software crisis**). Cùng năm đó, để giải quyết vấn đề software crisis, khái niệm “**Software Engineering**” ra đời. Một ngành học có tên Software Engineering cũng từ đó phát triển.

Theo thời gian, ngành Software Engineering đã không ngừng hoàn thiện qua nhiều vòng. Một số nội dung cốt lõi, chẳng hạn như software development model, ngày càng phong phú và thiết thực hơn!

**Software crisis là gì?**

Nói đơn giản, software crisis mô tả một khó khăn trong software development thời kỳ đó: chúng ta rất khó phát triển phần mềm chất lượng cao một cách hiệu quả.

Dijkstra (tác giả của thuật toán Dijkstra) cũng từng đề cập đến software crisis trong bài phát biểu nhận giải Turing năm 1972. Ông nói như sau: “Nguyên nhân chính dẫn đến software crisis là máy móc đã trở nên mạnh hơn nhiều bậc độ lớn! Nói thẳng ra: khi không có máy móc, lập trình hoàn toàn không có vấn đề gì. Khi có những máy tính yếu, lập trình trở thành một vấn đề vừa phải; còn hiện nay, khi có những máy tính khổng lồ, lập trình cũng trở thành một vấn đề khổng lồ”.

**Nói nhiều như vậy, rốt cuộc Software Engineering là gì?**

Engineering là việc áp dụng lý thuyết vào thực tiễn để giải quyết vấn đề thực tế. Software Engineering chính là việc áp dụng tư duy engineering vào software development.

Trên đây là định nghĩa của tôi về Software Engineering. Hãy cùng xem một định nghĩa có tính thẩm quyền hơn. IEEE Std 610.12-1990, _IEEE Standard Glossary of Software Engineering Terminology_, định nghĩa như sau: (1) Áp dụng các phương pháp có tính hệ thống, có quy chuẩn và có thể định lượng vào việc phát triển, vận hành và bảo trì phần mềm, tức áp dụng phương pháp engineering vào phần mềm. (2) Nghiên cứu các phương pháp được nêu ở (1).

Tóm lại, mục tiêu cuối cùng của Software Engineering là: **tạo ra phần mềm tốt hơn, dễ bảo trì hơn với mức tiêu hao tài nguyên thấp hơn.**

## Software Development Process

[Wikipedia định nghĩa software development process](https://zh.wikipedia.org/wiki/%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91%E8%BF%87%E7%A8%8B) như sau:

> Software development process (tiếng Anh: software development process), hay software process (tiếng Anh: software process), là software development life cycle của việc phát triển phần mềm. Các giai đoạn của nó thực hiện việc xác định và phân tích yêu cầu, thiết kế, triển khai, kiểm thử, bàn giao và bảo trì phần mềm. Software process là các bước cần tuân theo khi phát triển và xây dựng hệ thống, đồng thời là lộ trình của software development.

- Requirements analysis: Phân tích yêu cầu của người dùng và xây dựng logical model.
- Software design: Thiết kế software architecture dựa trên kết quả của requirements analysis.
- Coding: Viết source code để chương trình chạy được.
- Testing: Xác định test case và viết test report.
- Delivery: Bàn giao phần mềm đã hoàn thiện cho khách hàng.
- Maintenance: Bảo trì phần mềm, chẳng hạn như sửa bug và hoàn thiện tính năng.

Software development process chỉ định nghĩa một số quy trình có thể liên quan đến software development ở mức khá khái quát.

Software development model định nghĩa cụ thể hơn về software development process và cung cấp nền tảng lý thuyết vững chắc cho process này.

## Software Development Model

Có nhiều loại software development model và method, chẳng hạn như Waterfall Model, Rapid Prototype Model, V-model, W-model và Agile Development. Trong đó, tiêu biểu nhất vẫn là **Waterfall Model** và **Agile Development**.

**Waterfall Model** định nghĩa một software development life cycle hoàn chỉnh và thể hiện đầy đủ software life cycle.

![](https://oss.javaguide.cn/github/javaguide/system-design/schedule-task/up-264f2750a3d30366e36c375ec3a30ec2775.png)

**Agile Development** không phải là một software development model đơn lẻ, mà là một tập hợp các giá trị và nguyên tắc. Những method cụ thể thường gặp gồm Scrum, Extreme Programming (XP) và Kanban. Agile Development nhấn mạnh cá nhân và tương tác, working software, hợp tác với khách hàng và khả năng phản hồi thay đổi; đồng thời liên tục thu thập phản hồi thông qua iteration và frequent delivery.

**Continuous Integration**, **Refactoring**, **Small Release**, **Pair Programming** và **Test-driven Development** là các technical practice thường gặp trong Extreme Programming; **Daily Stand-up** thường xuất hiện trong Scrum. Đây không phải là các practice bắt buộc trong mọi Agile method. Agile Development cũng không đồng nghĩa với ít tài liệu hơn, mà chú trọng hơn vào working software, đồng thời vẫn giữ lại những tài liệu cần thiết và có giá trị.

## Các strategy cơ bản trong Software Development

### Software Reuse

Khi xây dựng phần mềm mới, chúng ta không cần bắt đầu từ con số không. Bằng cách tái sử dụng các vật liệu có sẵn như những “wheel” đã tồn tại (framework, third-party library...), design pattern, design principle..., chúng ta có thể xây dựng phần mềm đáp ứng yêu cầu nhanh hơn.

Các open-source project mà chúng ta thường tiếp xúc là ví dụ tốt nhất. Tôi nghĩ rằng nếu không có open source, việc xây dựng một phần mềm đáp ứng yêu cầu sẽ tiêu tốn nhiều công sức và thời gian hơn hiện nay rất nhiều!

### Chia để trị

Trong quá trình xây dựng phần mềm, chúng ta sẽ gặp nhiều vấn đề. Có thể phân rã những vấn đề tương đối phức tạp thành các vấn đề nhỏ hơn, rồi lần lượt giải quyết từng vấn đề.

Tôi sẽ trình bày về vấn đề này qua Domain Driven Design (viết tắt là DDD), một software design method khá phổ biến hiện nay.

Trong Domain Driven Design, một khái niệm rất quan trọng là **Domain**, chính là vấn đề chúng ta cần giải quyết. Trong Domain Driven Design, việc cần làm là phân rã một Domain (vấn đề) lớn thành một số Domain nhỏ (subdomain).

Ngoài ra, chia để trị cũng là một tư tưởng thường dùng trong algorithm, tương ứng với divide and conquer algorithm. Nếu muốn tìm hiểu về divide and conquer algorithm, bạn có thể xem [_Design and Analysis of Algorithms_](https://www.coursera.org/learn/algorithms) của Đại học Bắc Kinh.

### Phát triển từng bước

Software development là một quá trình phát triển từng bước. Chúng ta cần liên tục thực hiện iterative incremental development để cuối cùng bàn giao sản phẩm phù hợp với giá trị mà khách hàng cần.

Ở đây bổ sung một khái niệm rất quan trọng trong lĩnh vực software development: **MVP (Minimum Viable Product)**.

Minimum Viable Product là version sản phẩm dùng ít nguồn lực nhất để thu được nhiều hiểu biết đã được khách hàng xác nhận nhất. Mục tiêu là nhanh chóng xác minh các giả định then chốt; sản phẩm này không nhất thiết phải đáp ứng đầy đủ nhu cầu của khách hàng. Hình ảnh dưới đây thể hiện tư tưởng này rất rõ.

![](https://oss.javaguide.cn/github/javaguide/system-design/schedule-task/up-a99961ff7725106c0592abca845d555568a.png)

Nhờ Minimum Viable Product, chúng ta cũng có thể tiến hành market analysis sớm hơn. Điều này rất hữu ích trong quá trình khám phá những điều chưa chắc chắn của sản phẩm và có thể định hướng hiệu quả cho bước tiếp theo.

### Tối ưu và đánh đổi

Software development là một quá trình liên tục tối ưu và cải tiến. Mọi phần mềm đều có nhiều điểm có thể tối ưu, không thể đạt đến sự hoàn hảo. Chúng ta cần không ngừng cải thiện và nâng cao chất lượng phần mềm.

Tuy nhiên, cũng không nên rơi vào vòng luẩn quẩn này. Cần học cách đánh đổi, nâng cao chất lượng phần mềm hiện có bằng cách hiệu quả nhất trong phạm vi nguồn lực hữu hạn.

## Tài liệu tham khảo

- [IEEE Std 610.12-1990: IEEE Standard Glossary of Software Engineering Terminology](https://standards.ieee.org/ieee/610.12/855/)
- [Manifesto for Agile Software Development](https://agilemanifesto.org/)
- [Minimum Viable Product: a guide - Eric Ries](https://www.startuplessonslearned.com/2009/08/minimum-viable-product-guide.html)
- Các khái niệm cơ bản về Software Engineering - Khoa Software Engineering, Đại học Thanh Hoa, Liu Qiang: <https://www.xuetangx.com/course/THU08091000367>
- Software development process - Wikipedia: [Software development process](https://zh.wikipedia.org/wiki/软件开发过程)

<!-- @include: @article-footer.snippet.md -->
