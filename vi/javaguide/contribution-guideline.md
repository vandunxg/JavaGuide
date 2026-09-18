---
title: Hướng dẫn đóng góp
description: Hướng dẫn đóng góp cho dự án mã nguồn mở JavaGuide, giải thích quy trình tham gia bảo trì dự án, gửi PR và trở thành Contributor.
category: Tìm hiểu dự án
icon: "mdi:compass-outline"
---

Xin chào, tôi là Guide! Chào mừng bạn đến với “phòng thí nghiệm mã nguồn mở” của JavaGuide.

Tham gia bảo trì dự án mã nguồn mở không chỉ là một lần thực hành kỹ thuật, mà còn là quá trình “đóng góp trở lại cho cộng đồng kỹ thuật”.

Tại đây, từng dòng chữ và code của bạn đều sẽ được hàng trăm nghìn developer trên toàn thế giới nhìn thấy.

## Vì sao nên tham gia bảo trì JavaGuide?

Nhiều bạn cho rằng cộng đồng mã nguồn mở có ngưỡng tham gia cao, nhưng thực ra không phải vậy. Lợi ích từ việc tham gia bảo trì JavaGuide rất thiết thực:

1. **Nắm chắc kiến thức**: Trong quá trình sửa lỗi hoặc hoàn thiện nội dung, bạn sẽ buộc mình phải “học xuyên thấu”, kiểu ghi nhớ này sâu sắc hơn nhiều so với việc học thuộc lòng các câu trả lời phỏng vấn khuôn mẫu.
2. **Bảo chứng sức ảnh hưởng**: JavaGuide đã sắp đạt 160k Star. Nếu `PR` của bạn được chấp nhận, tên của bạn sẽ được lưu lại vĩnh viễn trong danh sách `Contributor`. Khi tìm việc hoặc phỏng vấn, đây là một **“minh chứng thực chiến mã nguồn mở”** rất thuyết phục.
3. **Phần thưởng hiện vật**: Tôi sẽ thỉnh thoảng gửi tai nghe, bàn phím cơ và các merchandise công nghệ khác cho những bạn đóng góp thường xuyên, thậm chí còn có cả phần thưởng tiền mặt trực tiếp.

## Có thể đóng góp theo những hướng nào?

Bạn có thể dựa vào quỹ thời gian của mình để chọn một trong ba hướng đóng góp sau:

- **Sửa lỗi (cơ bản)**: Phát hiện lỗi chính tả, dùng sai dấu câu hoặc format code lộn xộn trong tài liệu. Đây là loại đóng góp đơn giản nhất, nhưng cũng rất đáng quý.
- **Hoàn thiện (nâng cao)**: Refactor câu trả lời cho các câu hỏi phỏng vấn hiện có. Ví dụ, logic của một bài viết bị đứt đoạn hoặc thiếu phân tích về các tính năng kỹ thuật mới nhất.
- **Bổ sung (chuyên gia)**: Dựa trên xu hướng phỏng vấn mới nhất của các công ty lớn, bổ sung phần giải thích chi tiết cho các câu hỏi phỏng vấn thường gặp hoặc phân tích chuyên sâu các kiến thức quan trọng.

## Làm thế nào để gửi đóng góp một cách thuận lợi?

### Chế độ tối giản: nhấp vào “Chỉnh sửa trang này” (bắt đầu trong 3 phút)

Ở **góc dưới bên trái** của mỗi trang trên site đều có nút **「Chỉnh sửa trang này」**.

1. **Nhấp để chuyển trang**: Truy cập trực tiếp giao diện chỉnh sửa trực tuyến của GitHub.
2. **Chỉnh sửa trực tuyến**: Sửa nội dung trực tiếp trong trình duyệt, không cần thực hiện các bước rắc rối của `git clone`.
3. **Gửi yêu cầu**: Điền thông tin commit (Commit Message), nhấp gửi để tự động tạo `Pull Request`.

Cách này phù hợp nhất để sửa lỗi đánh máy hoặc tối ưu nội dung ở phạm vi nhỏ.

![](https://oss.javaguide.cn/github/javaguide/about/javaguide-contribution-edit-page.png)

### Chế độ nâng cao: Fork + PR (quy trình mã nguồn mở tiêu chuẩn)

Nếu muốn refactor trên diện rộng hoặc bổ sung nội dung mới, bạn nên sử dụng GitHub workflow tiêu chuẩn:

1. **Fork repository**: Nhấp vào `Fork` ở góc trên bên phải của [repository gốc](https://github.com/Snailclimb/JavaGuide) để sao chép JavaGuide vào tài khoản của bạn.
2. **Phát triển local**: Bạn có thể clone dự án về local, tự do chỉnh sửa và viết nội dung. Sau khi hoàn tất việc chỉnh sửa hoặc viết nội dung, chỉ cần commit trực tiếp lên repository bản sao.
3. **Tạo PR**: Sau khi commit xong, nhấp vào `New Pull Request` để yêu cầu merge các thay đổi của bạn vào branch chính của JavaGuide.

![](https://oss.javaguide.cn/github/javaguide/about/javaguide-contribution-pr.png)

Kỹ năng liên quan đến Git rất quan trọng, bạn nên thành thạo trước khi chính thức đi làm.

Tôi đã viết hai bài viết liên quan, bạn nên tham khảo:

- [Tổng hợp các khái niệm cốt lõi của Git](https://javaguide.cn/tools/git/git-intro.html)
- [Tổng hợp các mẹo hữu ích về Github](https://javaguide.cn/tools/git/github-tips.html)

### Gửi Issue để bắt đầu thảo luận

Nếu phát hiện một số chỗ cần cải thiện nhưng tạm thời chưa có thời gian viết code, hoặc muốn đề xuất bổ sung một chuyên đề, hãy trực tiếp mở thảo luận bằng **Issue**.

Mẫu đề xuất:

> **Tiêu đề**: Đề xuất bổ sung hướng dẫn so sánh và lựa chọn giải pháp ghi kép nhất quán giữa Redis và database
>
> **Mô tả nội dung**: Tính nhất quán của cache là một điểm khó và quan trọng trong phỏng vấn cũng như thực chiến; tài liệu hiện chưa có phần so sánh các giải pháp một cách hệ thống. Đề xuất bổ sung:
>
> 1. **So sánh giải pháp**: So sánh chi tiết ưu, nhược điểm của các giải pháp “cập nhật database trước rồi xóa cache”, “xóa kép có độ trễ”, “xóa bất đồng bộ thông qua binlog” và các giải pháp khác.
> 2. **Phân tích các tình huống cực đoan**: Phân tích cách bảo đảm consistency tối đa trong trường hợp replication bị trễ hoặc mạng chập chờn.
>
> **Ý định nhận xử lý**: Tôi đã nghiên cứu sâu về lĩnh vực này và biên soạn một bảng so sánh cùng sơ đồ quy trình, hy vọng có thể đóng góp cho JavaGuide.

## Yêu cầu đóng góp

### Format là năng suất hàng đầu

Giữa tiếng Trung và tiếng Anh cần có khoảng trắng, dấu câu phải đúng quy chuẩn. Tham khảo (chỉ cần đọc một bài bất kỳ):

- [Hướng dẫn trình bày nội dung tiếng Trung](https://github.com/sparanoid/chinese-copywriting-guidelines)
- [Hướng dẫn phong cách viết tài liệu kỹ thuật tiếng Trung](https://github.com/yikeke/zh-style-guide/)
- [Quy tắc trình bày nội dung tiếng Trung - Dawner](https://dawner.top/posts/chinese-copywriting-rules/)
- [Hướng dẫn trình bày tiếng Trung dành cho mọi người - Zhihu](https://zhuanlan.zhihu.com/p/20506092)

### Nội dung nguyên bản

Bạn có thể tham khảo và học hỏi từ bài viết của người khác, nhưng **nhất định, nhất định, nhất định không được copy-paste**!

Điều bạn cần làm không phải là “người vận chuyển thông tin”, mà là “bộ lọc kiến thức”. Hãy diễn đạt bằng lời của mình, cố gắng viết dễ hiểu hơn người khác và làm nổi bật trọng tâm. Chỉ cách diễn đạt “xuyên thấu” như vậy mới là có trách nhiệm lớn nhất với độc giả.

## Lời kết

Mã nguồn mở không phải là cuộc chiến đơn độc của một người, mà là hành trình tiến về phía trước của một tập thể. **Chuẩn bị phỏng vấn Java, hãy chọn JavaGuide!** Hy vọng được thấy tên bạn trong danh sách Contributor.
