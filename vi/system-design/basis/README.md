---
title: "Chuyên đề cơ sở system design: RESTful API, kỹ nghệ phần mềm, đặt tên code, refactoring và unit test"
description: "Lộ trình học phỏng vấn và kiến thức nền tảng về engineering backend, bao quát RESTful API, kỹ nghệ phần mềm, đặt tên code, refactoring và unit test."
category: System design
tag:
  - System design
  - Engineering basics
  - RESTful API
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: cơ sở system design,RESTful API,kỹ nghệ phần mềm,đặt tên code,refactoring,unit test,JUnit,kiến thức nền tảng backend,phỏng vấn backend
---

Chuyên đề cơ sở system design tập trung vào những năng lực engineering dễ bị bỏ qua nhất nhưng có ảnh hưởng lâu dài nhất trong phát triển backend: thiết kế API, đặt tên code, nhận diện code smell, viết test và tổ chức quy trình từ yêu cầu đến bàn giao.

Những nội dung này thoạt nhìn không “hardcore” như cache, message queue hay database sharding, nhưng chúng quyết định hệ thống có thể được team duy trì lâu dài hay không.

## Dành cho ai

- Developer muốn bổ sung có hệ thống kiến thức nền tảng về engineering backend.
- Bạn đọc chuẩn bị câu hỏi phỏng vấn về RESTful API, chất lượng code và unit test.
- Bạn đọc đã có thể hoàn thành việc phát triển nghiệp vụ nhưng muốn nâng cao khả năng thiết kế API và tính maintainable của code.
- Engineer cần xây dựng quy chuẩn API, quy chuẩn đặt tên hoặc quy chuẩn test cho team.

## Trọng tâm học

- Cốt lõi của RESTful API không phải là template URL cố định, mà là thiết kế API rõ ràng, nhất quán và có thể tiến hóa xoay quanh resource.
- Software engineering quan tâm đến toàn bộ quy trình từ yêu cầu, thiết kế, phát triển, test, bàn giao đến bảo trì.
- Tên gọi tốt giúp giảm chi phí giao tiếp, còn tên gọi tệ sẽ khiến hệ thống ngày càng khó thay đổi.
- Refactoring không phải là “viết lại”, mà là cải thiện cấu trúc bên trong trong khi giữ nguyên behavior bên ngoài.
- Unit test phải phục vụ chất lượng code và sự tự tin khi thay đổi, không chỉ chạy theo con số coverage.

## Thứ tự đọc đề xuất

1. [Hướng dẫn ngắn về software engineering](./software-engineering.md): trước hết tìm hiểu quy trình phát triển phần mềm, development model và các khái niệm cơ bản về engineering.
2. [Hướng dẫn ngắn về RestFul API](./RESTfulAPI.md): tìm hiểu resource path, HTTP method, status code và quy chuẩn thiết kế API.
3. [Hướng dẫn đặt tên code](./naming.md): xây dựng thói quen đặt tên rõ ràng, nhất quán và dễ đọc.
4. [Hướng dẫn refactoring code](./refactoring.md): nhận diện code smell, tìm hiểu các nguyên tắc và kỹ thuật refactoring thường gặp.
5. [Unit test thực sự là gì? Nên thực hiện thế nào?](./unit-test.md): cuối cùng dùng test để nâng cao độ an toàn khi thay đổi code.

## Bài viết cốt lõi

- [Hướng dẫn ngắn về software engineering](./software-engineering.md): hệ thống hóa các khái niệm cơ bản về khủng hoảng phần mềm, mô hình quy trình phát triển phần mềm, waterfall model và agile development.
- [Hướng dẫn ngắn về RestFul API](./RESTfulAPI.md): giải thích các nguyên tắc của REST architecture, thiết kế resource path, cách sử dụng HTTP method và quy chuẩn status code.
- [Hướng dẫn đặt tên code](./naming.md): tổng hợp nguyên tắc và kỹ thuật đặt tên variable, method, class, giúp nâng cao tính dễ đọc và maintainable của code.
- [Hướng dẫn refactoring code](./refactoring.md): giải thích định nghĩa refactoring, nguyên tắc refactoring, cách nhận diện code smell và các kỹ thuật refactoring thường dùng.
- [Unit test thực sự là gì? Nên thực hiện thế nào?](./unit-test.md): bao quát khái niệm unit test, Mock và Stub, test pyramid cùng kiến thức cơ bản về JUnit.

## Câu hỏi thường gặp

- REST và RESTful API khác nhau thế nào?
- Resource path trong RESTful API nên được đặt tên thế nào?
- GET, POST, PUT, PATCH và DELETE lần lượt phù hợp với trường hợp sử dụng nào?
- Vì sao software engineering không chỉ là viết code?
- Đặt tên code tốt nên tuân theo những nguyên tắc nào?
- Code smell là gì? Khi nào nên refactoring?
- Refactoring và viết lại khác nhau thế nào?
- Unit test, integration test và end-to-end test khác nhau thế nào?
- Mock và Stub lần lượt giải quyết vấn đề gì?
- Coverage của unit test càng cao có phải càng tốt không?

## Chuyên đề liên quan

- [Hệ thống kiến thức về system design](../)
- [Tổng hợp câu hỏi phỏng vấn design pattern thường gặp](../design-pattern.md)
- [Chuyên đề Spring & Spring Boot](../framework/spring/)
- [Các bài viết kỹ thuật chất lượng cao](../../high-quality-technical-articles/)

<!-- @include: @article-footer.snippet.md -->
