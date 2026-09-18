---
title: "Giải thích chi tiết về SSO (đăng nhập một lần)"
description: "Giải thích chi tiết nguyên lý SSO (đăng nhập một lần), bao quát thiết kế trung tâm xác thực tập trung, giao thức CAS, triển khai đăng nhập cross-domain và cơ chế đồng bộ trạng thái đăng nhập."
category: System design
tag:
  - Security
head:
  - - meta
    - name: keywords
      content: SSO,đăng nhập một lần,xác thực tập trung,trạng thái đăng nhập,ticket,TGT,ST,giao thức CAS,đăng nhập cross-domain
---

> Bài viết này được cho phép đăng lại từ: <https://ken.io/note/sso-design-implement> Tác giả: ken.io

## Giới thiệu SSO

### SSO là gì?

Tên đầy đủ trong tiếng Anh của SSO là Single Sign On, tức đăng nhập một lần. SSO cho phép người dùng chỉ cần đăng nhập một lần trong nhiều hệ thống ứng dụng là có thể truy cập tất cả các hệ thống ứng dụng tin cậy lẫn nhau.

Ví dụ, sau khi bạn đăng nhập vào trung tâm tài khoản NetEase (<https://reg.163.com/> ) thì khi truy cập các website sau, bạn đều đã ở trạng thái đăng nhập.

- NetEase Live [https://v.163.com](https://v.163.com/)
- NetEase Blog [https://blog.163.com](https://blog.163.com/)
- NetEase Huatian [https://love.163.com](https://love.163.com/)
- NetEase Kaola [https://www.kaola.com](https://www.kaola.com/)
- NetEase Lofter [http://www.lofter.com](http://www.lofter.com/)

### SSO có lợi ích gì?

1. **Góc độ người dùng**: Người dùng có thể đăng nhập một lần và sử dụng nhiều lần, không cần ghi nhớ nhiều bộ username và password, tiết kiệm công sức.
2. **Góc độ quản trị viên hệ thống**: Quản trị viên chỉ cần duy trì một trung tâm tài khoản thống nhất, thuận tiện.
3. **Góc độ phát triển hệ thống mới**: Khi phát triển hệ thống mới, chỉ cần kết nối trực tiếp với trung tâm tài khoản thống nhất, giúp đơn giản hóa quy trình phát triển và tiết kiệm thời gian.

## Thiết kế và triển khai SSO

Bài viết này chủ yếu nhằm thảo luận cách thiết kế và triển khai một hệ thống SSO.

Dưới đây là các chức năng cốt lõi cần triển khai:

- Đăng nhập một lần
- Đăng xuất một lần
- Hỗ trợ đăng nhập một lần cross-domain
- Hỗ trợ đăng xuất một lần cross-domain

### Ứng dụng cốt lõi và dependency

![Thiết kế đăng nhập một lần (SSO)](https://oss.javaguide.cn/github/javaguide/system-design/security/sso/sso-system.png-kblb.png)

| Ứng dụng/module/object             | Mô tả                                                                      |
| ---------------------------------- | -------------------------------------------------------------------------- |
| Website frontend                   | Website yêu cầu đăng nhập                                                  |
| Website SSO - đăng nhập            | Cung cấp trang đăng nhập                                                   |
| Website SSO - đăng xuất            | Cung cấp lối vào đăng xuất                                                 |
| Service SSO - đăng nhập            | Cung cấp service đăng nhập                                                 |
| Service SSO - trạng thái đăng nhập | Cung cấp service kiểm tra trạng thái đăng nhập/tra cứu thông tin đăng nhập |
| Service SSO - đăng xuất            | Cung cấp service đăng xuất cho người dùng                                  |
| Database                           | Lưu trữ thông tin tài khoản người dùng                                     |
| Cache                              | Lưu trữ thông tin đăng nhập của người dùng, thường sử dụng Redis           |

### Lưu trữ và kiểm tra trạng thái đăng nhập của người dùng

Các Web framework phổ biến thường triển khai Session bằng cách tạo một SessionId và lưu trong Cookie của browser. Sau đó, nội dung Session được lưu trong memory phía server. [ken.io](https://ken.io/) cũng từng đề cập đến cách làm này trong bài viết [nguyên lý hoạt động của Session](https://ken.io/note/session-principle-skill). Về tổng thể, bài viết này cũng mượn ý tưởng đó.

Sau khi người dùng đăng nhập thành công, website SSO thiết lập session đăng nhập của riêng mình. Dấu hiệu nhận diện session trong browser cần được lưu trong Cookie có thuộc tính `Secure`, `HttpOnly` và `SameSite` phù hợp; Cookie nên chỉ có tác dụng với host hiện tại, không nên mở rộng trực tiếp đến toàn bộ parent domain chỉ để chia sẻ trạng thái đăng nhập. App di động nên sử dụng system browser để hoàn tất quy trình authorization tiêu chuẩn, đồng thời lưu credential cần thiết trong secure storage như Keychain, Keystore. Bài viết này chủ yếu thảo luận về SSO dựa trên Web.

Khi người dùng truy cập trang yêu cầu đăng nhập, client gửi AuthToken đến service SSO để kiểm tra trạng thái đăng nhập/nhận thông tin đăng nhập của người dùng.

Đối với việc lưu trữ thông tin đăng nhập, nên sử dụng Redis và Redis cluster để lưu trữ thông tin đăng nhập, vừa có thể bảo đảm high availability, vừa có thể mở rộng tuyến tính. Đồng thời, cách này cũng giúp service SSO đáp ứng nhu cầu load balancing/scalability.

| Object              | Mô tả                                                                                                                                                                                                                                                                 |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| AuthToken           | Identifier entropy cao, không thể dự đoán, được tạo bằng số ngẫu nhiên an toàn về mặt mật mã; đồng thời thiết lập cơ chế hết hạn, rotation và revoke. Không sử dụng UUID version có thể dự đoán, cũng không tự mã hóa UserName + timestamp rồi dùng làm session token |
| Thông tin đăng nhập | Thường cache UserId và UserName                                                                                                                                                                                                                                       |

### Đăng nhập/kiểm tra đăng nhập của người dùng

**Sơ đồ trình tự đăng nhập**

![Thiết kế hệ thống SSO - sơ đồ trình tự đăng nhập](https://oss.javaguide.cn/github/javaguide/system-design/security/sso/sso-login-sequence.png-kbrb.png)

Sơ đồ trên minh họa cách chia sẻ AuthToken giữa nhiều subdomain thông qua parent-domain Cookie. Với hệ thống mới, không nên đặt `Domain` của authentication Cookie thành `.test.com`: cách này khiến cùng một credential được gửi đến tất cả subdomain phù hợp; bất kỳ subdomain yếu, không còn sử dụng hoặc bị chiếm quyền nào cũng có thể làm mở rộng attack surface.

Cách an toàn hơn là để website SSO và từng website nghiệp vụ lần lượt duy trì session host-only. Khi website nghiệp vụ cần đăng nhập, nó chuyển hướng đến website SSO. Website SSO dùng Cookie của mình để xác định người dùng đã đăng nhập hay chưa, sau đó trả kết quả xác thực về website nghiệp vụ thông qua authorization code chỉ dùng một lần và có thời hạn ngắn. Backend của website nghiệp vụ dùng code này để đổi lấy thông tin người dùng và thiết lập session cục bộ.

**Lấy thông tin đăng nhập/kiểm tra trạng thái đăng nhập**

![Thiết kế hệ thống SSO - lấy thông tin đăng nhập/kiểm tra trạng thái đăng nhập](https://oss.javaguide.cn/github/javaguide/system-design/security/sso/sso-logincheck-sequence.png-kbrb.png)

### Đăng xuất của người dùng

Đăng xuất SSO không chỉ đơn giản là xóa một Cookie:

1. Server SSO revoke central login session và các refresh authorization liên quan.
2. Website SSO xóa session Cookie của mình.
3. Tùy theo protocol và rủi ro nghiệp vụ, thông qua front-channel hoặc back-channel thông báo cho các website nghiệp vụ dọn dẹp session cục bộ, đồng thời xử lý các trường hợp thông báo thất bại hoặc website offline.

**Sơ đồ trình tự đăng xuất**

![Thiết kế hệ thống SSO - đăng xuất người dùng](https://oss.javaguide.cn/github/javaguide/system-design/security/sso/sso-logout-sequence.png-kbrb.png)

### Đăng nhập, đăng xuất cross-domain

SSO cross-domain không nên cố giải quyết vấn đề đọc/ghi Cookie cross-domain, mà nên thiết lập session riêng cho từng website thông qua browser redirect tiêu chuẩn và token exchange phía backend. Một lựa chọn phổ biến là OpenID Connect Authorization Code Flow.

Ý tưởng cốt lõi để giải quyết vấn đề cross-domain là:

- Sau khi đăng nhập hoàn tất, service SSO chỉ trả authorization code chỉ dùng một lần và có thời hạn ngắn đến callback address đã đăng ký trước và được đối chiếu nghiêm ngặt. Backend của website nghiệp vụ sử dụng authorization code này để đổi lấy kết quả xác thực và thiết lập session riêng, không sao chép cùng một Bearer Token dài hạn giữa nhiều domain.
- Authorization request cần kiểm tra `state`; khi sử dụng OpenID Connect, còn phải kiểm tra Issuer, Audience và signature, đồng thời đối chiếu giá trị của `nonce` nếu sử dụng. Khi public client sử dụng Authorization Code Flow, client cũng cần sử dụng PKCE.
- Cross-site logout sử dụng thông báo front-channel hoặc back-channel được định nghĩa trong protocol, đồng thời cho phép website nghiệp vụ hội tụ trạng thái thông qua session expiration, token revocation và revalidation khi thông báo thất bại.

**Đăng nhập cross-domain (domain chính đã đăng nhập)**

![Thiết kế hệ thống SSO - đăng nhập cross-domain (domain chính đã đăng nhập)](https://oss.javaguide.cn/github/javaguide/system-design/security/sso/sso-crossdomain-login-loggedin-sequence.png-kbrb.png)

**Đăng nhập cross-domain (domain chính chưa đăng nhập)**

![Thiết kế hệ thống SSO - đăng nhập cross-domain (domain chính chưa đăng nhập)](https://oss.javaguide.cn/github/javaguide/system-design/security/sso/sso-crossdomain-login-unlogin-sequence.png-kbrb.png)

**Đăng xuất cross-domain**

![Thiết kế hệ thống SSO - đăng xuất cross-domain](https://oss.javaguide.cn/github/javaguide/system-design/security/sso/sso-crossdomain-logout-sequence.png-kbrb.png)

Các sơ đồ trình tự trên đến từ phương án được đăng lại ban đầu, chủ yếu dùng để giúp hiểu quan hệ chuyển hướng đăng nhập và thông báo. Những chi tiết như truyền trực tiếp AuthToken và chia sẻ parent-domain Cookie không nên được dùng làm cơ sở triển khai cho hệ thống mới. Hệ thống mới cần tuân theo các quy định bảo mật hiện hành của OpenID Connect/OAuth protocol đã chọn.

## Giải thích

- Về phương án: Phương án thiết kế lần này chủ yếu cung cấp ý tưởng triển khai. Đăng nhập của người dùng APP không nên chỉ thêm một “APP signature” tùy chỉnh rồi xem đó là security solution. Nên sử dụng system browser để hoàn tất OpenID Connect/OAuth Authorization Code Flow và sử dụng PKCE; không thể xem APP là môi trường tin cậy có thể bảo quản client secret vĩnh viễn.
- Về sơ đồ trình tự: Sơ đồ trình tự không bao gồm mọi tình huống, mà chỉ liệt kê các tình huống cốt lõi/chính. Ngoài ra, những message không ảnh hưởng đến việc hiểu ý tưởng sẽ được lược bỏ nếu có thể.

## Tham khảo

- [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0-18.html)
- [RFC 9700: Best Current Practice for OAuth 2.0 Security](https://www.rfc-editor.org/rfc/rfc9700.html)

<!-- @include: @article-footer.snippet.md -->
