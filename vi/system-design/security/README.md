---
title: "Chuyên đề Authentication, Authorization và Data Security: JWT, SSO, hệ thống phân quyền, encryption, data desensitization và data validation"
description: "Lộ trình học phỏng vấn về Authentication, Authorization và Data Security, bao quát Session, Token, OAuth2, JWT, SSO, RBAC, encryption algorithm, sensitive word filtering, data desensitization, data validation và password security."
category: System Design
tag:
  - Security
  - Data Security
  - Backend interview
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Authentication,Authorization,Session,Token,OAuth2,JWT,SSO,RBAC,hệ thống phân quyền,encryption algorithm,sensitive word filtering,data desensitization,data validation,password security,backend interview
---

Chuyên đề Authentication, Authorization và Data Security tập trung vào một chuỗi rất cơ bản nhưng có chi phí sai sót cao trong hệ thống backend: người dùng đăng nhập thế nào, danh tính được truyền đi ra sao, quyền được kiểm tra thế nào, dữ liệu nhạy cảm được bảo vệ ra sao, dữ liệu đầu vào được kiểm tra thế nào.

Security không phải là vấn đề có thể được giải quyết chỉ bằng một framework hoặc một annotation. Cần thiết kế đồng thời từ nhiều khâu như Authentication, Authorization, truyền tải, lưu trữ, hiển thị, data validation và audit.

Nếu thời gian khá hạn chế, bạn có thể xem trước [Tổng hợp câu hỏi phỏng vấn thường gặp về Authentication và Authorization](https://interview.javaguide.cn/system-design/authentication-and-authorization-interview-questions.html) và [Tổng hợp câu hỏi phỏng vấn thường gặp về Web Security](https://interview.javaguide.cn/system-design/security-interview-questions.html), sau đó quay lại chuyên đề này để bổ sung đầy đủ nguyên lý và phương án triển khai.

## Dành cho ai

- Developer backend đang học về đăng nhập, Authentication, hệ thống phân quyền và Data Security.
- Bạn đọc đang chuẩn bị câu hỏi phỏng vấn liên quan đến Authentication, Authorization, JWT, SSO, RBAC và data desensitization.
- Engineer cần thiết kế quyền quản trị, hệ thống người dùng, cách hiển thị dữ liệu nhạy cảm hoặc phương án data validation trong dự án.
- Bạn đọc đã sử dụng các framework như Spring Security, Sa-Token, Shiro nhưng muốn bổ sung các khái niệm bên trong.

## Trọng tâm học

- Authentication giải quyết “Bạn là ai”, Authorization giải quyết “Bạn có thể làm gì”; không được đánh đồng hai khái niệm này.
- Session, Token, JWT, OAuth2 và SSO phù hợp với các trường hợp sử dụng khác nhau; không thể chỉ dựa vào “stateless” để đánh giá ưu / nhược điểm.
- Ưu điểm và vấn đề của JWT đều rất rõ ràng; trọng tâm nằm ở kiểm soát thời điểm hết hiệu lực, gia hạn, rủi ro rò rỉ và quản trị phía server.
- Hệ thống phân quyền thường cần mô hình hóa theo các chiều như user, role, permission, resource, organization và data scope.
- Data Security bao gồm cả encryption khi lưu trữ, hiển thị data desensitization, data validation, sensitive word filtering và password security.
- Giải pháp security phải cân nhắc chi phí triển khai, trải nghiệm người dùng, khả năng audit và năng lực xử lý sự cố.

## Thứ tự đọc đề xuất

1. [Giải thích chi tiết các khái niệm cơ bản về Authentication và Authorization](./basis-of-authority-certification.md): trước hết phân biệt các khái niệm Authentication, Authorization, Session, Token, OAuth2.
2. [Giải thích chi tiết các khái niệm cơ bản về JWT](./jwt-intro.md) và [Phân tích ưu / nhược điểm của JWT](./advantages-and-disadvantages-of-jwt.md): tìm hiểu cách JWT hoạt động, ưu điểm và giới hạn.
3. [Giải thích chi tiết SSO (Single Sign-On)](./sso-intro.md): tìm hiểu authentication center thống nhất, đăng nhập xuyên hệ thống và đồng bộ trạng thái đăng nhập.
4. [Giải thích chi tiết thiết kế hệ thống phân quyền](./design-of-authority-system.md): triển khai Authentication và Authorization vào thiết kế hệ thống phân quyền RBAC.
5. [Tổng hợp encryption algorithm phổ biến](./encryption-algorithms.md), [Tổng hợp giải pháp data desensitization](./data-desensitization.md), [Vì sao cả frontend và backend đều phải thực hiện data validation?](./data-validation.md): bổ sung nền tảng Data Security.
6. Sau đó, tùy theo trường hợp sử dụng, đọc [Tổng hợp giải pháp sensitive word filtering](./sentive-words-filter.md) và [Vì sao khi quên password chỉ có thể reset, không thể cho biết password gốc?](./why-password-reset-instead-of-retrieval.md).

## Bài viết cốt lõi

### Authentication và Authorization

- [Giải thích chi tiết các khái niệm cơ bản về Authentication và Authorization](./basis-of-authority-certification.md): trình bày các kiến thức cốt lõi như Authentication, Authorization, Session, Token và OAuth2.
- [Giải thích chi tiết các khái niệm cơ bản về JWT](./jwt-intro.md): trình bày cấu trúc, signing algorithm, nguyên lý hoạt động và ứng dụng JWT trong đăng nhập, Authentication.
- [Phân tích ưu / nhược điểm của JWT](./advantages-and-disadvantages-of-jwt.md): phân tích các vấn đề như JWT không thể chủ động hết hiệu lực, gia hạn Token và các giải pháp.
- [Giải thích chi tiết SSO (Single Sign-On)](./sso-intro.md): trình bày authentication center thống nhất, giao thức CAS, triển khai đăng nhập cross-domain và cơ chế đồng bộ trạng thái đăng nhập.
- [Giải thích chi tiết thiết kế hệ thống phân quyền](./design-of-authority-system.md): trình bày việc mô hình hóa hệ thống phân quyền, access control và thiết kế trang quản trị dựa trên RBAC.

### Data Security

- [Tổng hợp encryption algorithm phổ biến](./encryption-algorithms.md): hệ thống hóa nguyên lý và trường hợp sử dụng của các algorithm như AES, RSA, MD5 và SHA.
- [Tổng hợp giải pháp sensitive word filtering](./sentive-words-filter.md): trình bày quá trình phát triển của algorithm sensitive word filtering từ brute-force matching đến Trie tree, AC automaton và thực tiễn engineering.
- [Tổng hợp giải pháp data desensitization](./data-desensitization.md): trình bày quy tắc và cách triển khai data desensitization cho số điện thoại, số căn cước, số thẻ ngân hàng và các data nhạy cảm khác.
- [Vì sao cả frontend và backend đều phải thực hiện data validation?](./data-validation.md): giải thích tầm quan trọng của parameter validation và permission validation, cũng như cách ngăn việc bypass validation ở frontend.
- [Vì sao khi quên password chỉ có thể reset, không thể cho biết password gốc?](./why-password-reset-instead-of-retrieval.md): giải thích password hash, salt, Bcrypt và an toàn khi truyền password.

## Câu hỏi thường gặp

- Authentication và Authorization khác nhau thế nào?
- Session và Token khác nhau thế nào?
- JWT gồm những phần nào? Chữ ký giải quyết vấn đề gì?
- Vì sao JWT không thể chủ động hết hiệu lực một cách tự nhiên? Có những giải pháp nào?
- Mối quan hệ giữa OAuth2 và JWT là gì?
- Quy trình cốt lõi của SSO (Single Sign-On) là gì?
- Thiết kế RBAC permission model thế nào? Mối quan hệ giữa user, role, permission và resource là gì?
- Symmetric encryption, asymmetric encryption và hash algorithm lần lượt phù hợp với trường hợp sử dụng nào?
- Vì sao password không thể được lưu dưới dạng plaintext? Vì sao khi quên password chỉ có thể reset?
- Nên thực hiện data desensitization ở storage layer, service layer hay presentation layer?
- Vì sao backend bắt buộc phải thực hiện data validation?
- Sensitive word filtering có những phương án triển khai phổ biến nào?

## Chuyên đề liên quan

- [Hệ thống kiến thức về System Design](../)
- [Chuyên đề cơ bản về System Design](../basis/)
- [Chuyên đề Spring & Spring Boot](../framework/spring/)
- [Hệ thống kiến thức về high availability](../../high-availability/)
- [Network Security](../../cs-basics/network/network-attack-means.md)

<!-- @include: @article-footer.snippet.md -->
