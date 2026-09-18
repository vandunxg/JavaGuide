---
title: "Tuyển chọn open-source project Java: học tập, thực hành và lựa chọn công cụ"
description: "Đề xuất open-source project Java và lộ trình học qua thực hành, tuyển chọn các technical tutorial, project thực hành, framework, thư viện công cụ, project AI và big data được duy trì liên tục trên GitHub và Gitee."
category: Open-source Projects
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Java open-source projects,Java practical projects,Java project recommendations,Java technical tutorials,Java system design projects,Java libraries,open-source project recommendations,backend projects,resume projects
---

<!-- @include: @small-advertisement.snippet.md -->

**Tuyển chọn open-source project Java** này dành cho việc học Java backend, thực hành project và lựa chọn công cụ, tổng hợp các project Java/JVM vẫn đang được duy trì trên GitHub và Gitee.

Số lượng project không phải càng nhiều càng tốt. Ở đây chú trọng hơn đến mức độ hoạt động của source code, chất lượng tài liệu và khả năng chạy được, đồng thời cố gắng nêu rõ project phù hợp để giải quyết vấn đề gì. Hoan nghênh bạn đề xuất các project bạn đánh giá cao tại [khu vực Issues của JavaGuide](https://github.com/Snailclimb/JavaGuide/issues).

## Điều hướng

| Danh mục                                          | Phạm vi tuyển chọn                                                       | Dùng để làm gì                                            |
| ------------------------------------------------- | ------------------------------------------------------------------------ | --------------------------------------------------------- |
| [Technical tutorial](./tutorial.md)               | Knowledge base, course, open-source book và code example                 | Củng cố nền tảng, bổ sung phần còn thiếu                  |
| [Project thực hành](./practical-project.md)       | Ứng dụng Java/JVM có business function hoàn chỉnh                        | Luyện tập, phát triển tiếp, chuẩn bị kinh nghiệm project  |
| [Project AI](./machine-learning.md)               | Java AI framework, model inference tool và ứng dụng AI                   | Phát triển RAG, Agent và model service                    |
| [Framework và infrastructure](./system-design.md) | Web framework, database, middleware, observability và các component khác | Lựa chọn công nghệ, học source code, system design        |
| [Thư viện công cụ](./tool-library.md)             | Java library được thêm vào project qua Maven/Gradle                      | Giải quyết vấn đề cụ thể khi coding                       |
| [Công cụ phát triển](./tools.md)                  | GUI, CLI, platform kiểm tra và công cụ vận hành chạy độc lập             | Hoàn thiện toolchain phát triển và troubleshooting cục bộ |
| [Project big data](./big-data.md)                 | Project về compute, storage, data integration và lakehouse               | Học hoặc xây dựng nền tảng big data                       |

## Quy tắc tuyển chọn và duy trì

- Ngày kiểm tra của đợt này là **11 tháng 8 năm 2026**; không tuyển chọn các project có lần cập nhật gần nhất trước **11 tháng 5 năm 2026**.
- Với project GitHub, lấy commit gần nhất trên default branch làm chuẩn; với project Gitee, tham khảo thời điểm push gần nhất của repository.
- Khi repository được chuyển, archive hoặc ngừng duy trì, sẽ cập nhật link hoặc loại khỏi danh sách. Version number dùng một lần và thay đổi README không thể hoàn toàn đại diện cho chất lượng project; về sau sẽ tiếp tục đối chiếu với lịch sử release và tình hình xử lý Issue.
- Star chỉ dùng để tham khảo. Project thiếu tài liệu, không thể build, license không rõ ràng hoặc technology stack rõ ràng đã lỗi thời có thể không được tuyển chọn dù có độ phổ biến cao.

## Cách sử dụng danh sách này

- Ở giai đoạn mới học, trước tiên hãy xem technical tutorial, sau đó bắt đầu từ các project có ranh giới rõ ràng như Spring Petclinic, đừng ngay từ đầu đã lao vào hệ thống microservice lớn.
- Khi chuẩn bị kinh nghiệm project, ưu tiên chọn project thực hành có thể chạy cục bộ, business flow hoàn chỉnh và thực sự có thể được bạn cải tạo. Clone và deploy không đồng nghĩa với kinh nghiệm project.
- Khi lựa chọn công nghệ, trước tiên xác định thứ cần là code dependency, tool độc lập hay infrastructure, sau đó vào danh mục tương ứng để tránh so sánh trực tiếp các project ở những layer khác nhau.
- Trước khi đọc source code, hãy xem tài liệu project, cách phân chia module và test, sau đó mang theo một vấn đề cụ thể để lần theo call chain; hiệu quả thường cao hơn đọc từng dòng từ entry point.

## Câu hỏi thường gặp

- Khi tìm project Java để luyện tập, nên ưu tiên xem business project hay infrastructure project?
- Làm thế nào để xác định một open-source project có phù hợp để đưa vào resume hay không?
- Project có số Star cao có nhất định đáng học không?
- Sau khi clone project và chạy được, nên bắt đầu cải tạo từ những phần nào?
- Khi interviewer hỏi về khó khăn của project, làm thế nào để tránh chỉ nói về CRUD?
- Nên lựa chọn thư viện công cụ thế nào? Khi nào không nên thêm dependency mới?
- Muốn nâng cao khả năng đọc source code, nên bắt đầu từ những open-source project nào?

## Chủ đề liên quan

- [Hệ thống kiến thức Java](../java/)
- [System design](../system-design/)
- [Hệ thống kiến thức distributed system](../distributed-system/)
- [Hệ thống kiến thức high-performance system](../high-performance/)
- [Tuyển chọn technical book](../books/)
- [Chuyên mục chất lượng dành riêng cho thành viên](../zhuanlan/)

<!-- @include: @article-footer.snippet.md -->
