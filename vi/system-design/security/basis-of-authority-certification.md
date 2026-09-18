---
title: "Giải thích chi tiết các khái niệm cơ bản về Authentication và Authorization"
description: "Giải thích chi tiết các khái niệm cơ bản về Authentication và Authorization, phân biệt Authentication và Authorization, Session, Token, OAuth2 và các kiến thức cốt lõi khác."
category: System Design
tag:
  - Security
head:
  - - meta
    - name: keywords
      content: authentication,authorization,Authentication,Authorization,Session,Token,OAuth2,access control,security basics
---

## Sự khác biệt giữa Authentication và Authorization là gì?

Đây là vấn đề mà hầu hết mọi người đều nhầm lẫn. Trước hết, hãy bắt đầu bằng cách phân biệt cách đọc hai thuật ngữ này. Nhiều người thường đọc nhầm chúng, vì vậy bạn nên tra cách đọc và ý nghĩa cụ thể của hai từ này trước.

Nói đơn giản:

- **Authentication:** Bạn là ai.
- **Authorization:** Bạn có quyền làm gì.

Nói chính thức hơn một chút:

- **Authentication** là thông tin xác thực dùng để kiểm tra danh tính của bạn (ví dụ username/user ID và password). Nhờ thông tin này, hệ thống biết bạn chính là bạn, tức là hệ thống có người dùng này. Vì vậy, Authentication còn được gọi là xác thực danh tính/người dùng.
- **Authorization** xảy ra sau **Authentication**. Chỉ cần nhìn vào ý nghĩa của từ này là có thể hiểu: nó chủ yếu quản lý quyền truy cập hệ thống. Ví dụ, một số tài nguyên chỉ cho người có quyền cụ thể như admin truy cập; một số thao tác trên tài nguyên hệ thống như xóa, thêm và cập nhật cũng chỉ người được chỉ định mới có quyền thực hiện.

Authentication:

![Đăng nhập bằng Authentication](https://oss.javaguide.cn/github/javaguide/system-design/security/authentication-login.png)

Authorization:

![Không có quyền](https://oss.javaguide.cn/github/javaguide/system-design/security/20210604161032412.png)

Hai cơ chế này thường được kết hợp trong hệ thống để bảo vệ tính an toàn của hệ thống.

## Bạn có biết mô hình RBAC không?

Mô hình kiểm soát truy cập thường được dùng nhất trong kiểm soát quyền hệ thống là **mô hình RBAC**.

**RBAC là gì?** RBAC là viết tắt của kiểm soát truy cập dựa trên vai trò (Role-Based Access Control). Đây là cách cấp quyền bằng việc liên kết quyền với vai trò, đồng thời liên kết vai trò với người dùng.

Nói đơn giản: một người dùng có thể sở hữu nhiều vai trò, mỗi vai trò lại có thể được phân bổ nhiều quyền, từ đó hình thành mô hình cấp quyền “người dùng - vai trò - quyền”. Trong mô hình này, người dùng và vai trò, vai trò và quyền tạo thành các quan hệ nhiều-nhiều.

![Sơ đồ mô hình quyền RBAC](https://oss.javaguide.cn/github/javaguide/system-design/security/design-of-authority-system/rbac.png)

Trong mô hình quyền RBAC, quyền được liên kết với vai trò. Người dùng nhận được quyền của vai trò bằng cách trở thành thành viên có vai trò cụ thể, nhờ đó việc quản lý quyền được đơn giản hóa đáng kể.

Để triển khai mô hình quyền RBAC, thiết kế cơ sở dữ liệu thường gặp như sau (tổng cộng 5 bảng, trong đó 2 bảng dùng để thiết lập liên kết giữa các bảng):

![](https://oss.javaguide.cn/2020-11/%E6%95%B0%E6%8D%AE%E5%BA%93%E8%AE%BE%E8%AE%A1-%E6%9D%83%E9%99%90.png)

Thông qua mô hình quyền này, ta có thể tạo các vai trò khác nhau và phân bổ phạm vi quyền khác nhau (menu) cho từng vai trò.

![](https://oss.javaguide.cn/github/javaguide/books%E6%9D%83%E9%99%90%E7%AE%A1%E7%90%86%E6%A8%A1%E5%9D%97.png)

Thông thường, nếu hệ thống có yêu cầu kiểm soát quyền tương đối nghiêm ngặt, người ta sẽ chọn mô hình RBAC để kiểm soát quyền.

## `Cookie` là gì? Tác dụng của `Cookie` là gì?

![](https://oss.javaguide.cn/github/javaguide/system-design/security/cookie-sessionId.png)

`Cookie` và `Session` đều là cách theo dõi danh tính người dùng trình duyệt, nhưng trường hợp sử dụng của chúng không hoàn toàn giống nhau.

Wikipedia định nghĩa `Cookie` như sau:

> `Cookies` là dữ liệu (thường được mã hóa) mà một số website lưu trên thiết bị đầu cuối cục bộ của người dùng để nhận diện danh tính người dùng.

Nói đơn giản: **`Cookie` được lưu ở client, thường dùng để lưu thông tin người dùng**.

Dưới đây là một số trường hợp sử dụng `Cookie`:

1. Lưu thông tin người dùng đã đăng nhập trong `Cookie`, để lần sau truy cập website, trang có thể tự động điền một số thông tin đăng nhập cơ bản. Ngoài ra, `Cookie` còn có thể lưu tùy chọn người dùng, theme và các thông tin cài đặt khác.
2. Dùng `Cookie` để lưu `SessionId` hoặc `Token`. Khi gửi request đến backend, request mang theo `Cookie`, nhờ đó backend có thể lấy được `Session` hoặc `Token`. Vì giao thức HTTP là stateless nên cách này giúp ghi nhận trạng thái hiện tại của người dùng.
3. `Cookie` cũng có thể dùng để ghi nhận và phân tích hành vi người dùng. Ví dụ, khi mua sắm trực tuyến, vì giao thức HTTP không có trạng thái, nếu server muốn biết bạn ở lại một trang bao lâu hoặc đã xem những sản phẩm nào, một cách triển khai thường dùng là lưu các thông tin này trong `Cookie`.
4. …

## Sử dụng `Cookie` trong dự án như thế nào?

Ở đây lấy dự án Spring Boot làm ví dụ.

**1) Thiết lập `Cookie` và trả về client**

```java
@GetMapping("/change-username")
public String setCookie(HttpServletResponse response) {
    // Tạo một cookie
    Cookie cookie = new Cookie("username", "Jovan");
    // Thiết lập thời gian hết hạn của cookie
    cookie.setMaxAge(7 * 24 * 60 * 60); // expires in 7 days
    // Thêm vào response
    response.addCookie(cookie);

    return "Username is changed!";
}
```

**2) Dùng annotation `@CookieValue` do Spring cung cấp để lấy giá trị của một cookie cụ thể**

```java
@GetMapping("/")
public String readCookie(@CookieValue(value = "username", defaultValue = "Atta") String username) {
    return "Hey! My username is " + username;
}
```

**3) Đọc tất cả giá trị `Cookie`**

```java
@GetMapping("/all-cookies")
public String readAllCookies(HttpServletRequest request) {

    Cookie[] cookies = request.getCookies();
    if (cookies != null) {
        return Arrays.stream(cookies)
                .map(c -> c.getName() + "=" + c.getValue()).collect(Collectors.joining(", "));
    }

    return "No cookies";
}
```

Bạn có thể xem thêm nội dung về cách sử dụng `Cookie` trong Spring Boot tại bài viết này: [How to use cookies in Spring Boot](https://attacomsian.com/blog/cookies-spring-boot).

## `Cookie` khác `Session` như thế nào?

**Tác dụng chính của `Session` là ghi nhận trạng thái người dùng ở server.** Một trường hợp điển hình là giỏ hàng. Khi bạn thêm sản phẩm vào giỏ hàng, hệ thống không biết thao tác đó do người dùng nào thực hiện vì giao thức HTTP là stateless. Sau khi server tạo một `Session` riêng cho người dùng cụ thể, nó có thể nhận diện và theo dõi người dùng đó.

Dữ liệu `Cookie` được lưu ở client (phía trình duyệt), còn dữ liệu `Session` được lưu ở server. Xét tương đối, `Session` an toàn hơn. Nếu dùng `Cookie` để lưu thông tin nhạy cảm, không nên ghi trực tiếp thông tin đó vào `Cookie`; tốt nhất là mã hóa thông tin `Cookie`, sau đó giải mã ở server khi cần dùng.

**Vậy xác thực bằng `Session` như thế nào?**

## Xác thực bằng phương án Session-Cookie như thế nào?

Nhiều khi chúng ta nhận diện người dùng cụ thể thông qua `SessionID`, và `SessionID` thường được lưu trong Redis. Ví dụ:

1. Người dùng đăng nhập hệ thống thành công, sau đó client nhận được `Cookie` chứa `SessionID`.
2. Khi người dùng gửi request đến backend, request mang theo `SessionID`, nhờ đó backend biết trạng thái danh tính của bạn.

Quy trình chi tiết hơn của cách xác thực này như sau:

![](https://oss.javaguide.cn/github/javaguide/system-design/security/session-cookie-authentication-process.png)

1. Người dùng gửi username, password và mã xác nhận đến server để đăng nhập hệ thống.
2. Sau khi xác thực thành công, server tạo và lưu một đối tượng `Session` riêng cho người dùng (có thể hiểu là một vùng nhớ trên server, lưu dữ liệu trạng thái của người dùng như giỏ hàng, thông tin đăng nhập...), đồng thời cấp cho `Session` này một `SessionID` duy nhất.
3. Server gửi `SessionID` đến trình duyệt của người dùng thông qua chỉ thị `Set-Cookie` trong HTTP response header.
4. Sau khi nhận `SessionID`, trình duyệt lưu nó ở local dưới dạng `Cookie`. Khi người dùng vẫn đang đăng nhập, mỗi lần gửi request đến server đó, trình duyệt đều tự động đính kèm `Cookie` chứa `SessionID`.
5. Sau khi nhận request, server lấy `SessionID` từ `Cookie` để tìm đối tượng `Session` đã lưu trước đó, từ đó biết đây là người dùng nào và trạng thái trước đó của họ.

Khi sử dụng `Session`, cần lưu ý các điểm sau:

- **Hỗ trợ `Cookie` ở client**: Các chức năng cốt lõi phụ thuộc vào `Session` cần bảo đảm trình duyệt của người dùng đã bật `Cookie`.
- **Quản lý thời hạn `Session`**: Thiết lập thời hạn `Session` hợp lý để cân bằng giữa an toàn và trải nghiệm người dùng.
- **An toàn của `Session ID`**: Thiết lập flag `HttpOnly` cho `Cookie` chứa `SessionID` để ngăn script phía client (như JavaScript) đánh cắp nó; thiết lập flag `Secure` để bảo đảm `SessionID` chỉ được truyền qua kết nối HTTPS, từ đó tăng tính an toàn.

Ngoài ra, Spring Session cung cấp cơ chế quản lý thông tin session của người dùng trên nhiều ứng dụng hoặc instance. Nếu muốn tìm hiểu chi tiết, bạn có thể xem các bài viết hữu ích sau:

- [Getting Started with Spring Session](https://codeboje.de/spring-Session-tutorial/)
- [Guide to Spring Session](https://www.baeldung.com/spring-Session)
- [Sticky Sessions with Spring Session & Redis](https://medium.com/@gvnix/sticky-Sessions-with-spring-Session-redis-bdc6f7438cc3)

## Phương án Session-Cookie hoạt động thế nào khi có nhiều node server?

Phương án Session-Cookie là cách xác thực danh tính rất tốt trong môi trường monolith. Tuy nhiên, khi server được mở rộng theo chiều ngang thành nhiều node, phương án Session-Cookie sẽ gặp thách thức.

Ví dụ: giả sử triển khai hai bản sao của cùng một service là A và B. Khi người dùng đăng nhập lần đầu, Nginx dùng cơ chế load balancing để chuyển request của người dùng đến server A, lúc này thông tin `Session` của người dùng được lưu ở server A. Đến lần truy cập thứ hai, Nginx lại định tuyến request đến server B. Vì server B không lưu thông tin `Session` của người dùng nên người dùng phải đăng nhập lại.

**Làm thế nào để tránh tình huống trên?**

Có một số phương án để tham khảo:

1. Phân bổ tất cả request của một người dùng cho cùng một server xử lý bằng một chiến lược hash cụ thể. Khi đó, mỗi server lưu thông tin `Session` của một phần người dùng. Nếu server bị dừng, toàn bộ thông tin `Session` mà nó lưu sẽ mất hoàn toàn.
2. Thông tin `Session` được lưu ở mỗi server đều được đồng bộ với nhau, tức là mỗi server đều lưu toàn bộ thông tin `Session`. Mỗi khi thông tin `Session` ở một server thay đổi, ta đồng bộ nó sang các server khác. Chi phí của phương án này rất lớn và chi phí đồng bộ càng tăng khi số node càng nhiều.
3. Dùng riêng một data node mà tất cả server đều có thể truy cập (ví dụ cache) để lưu thông tin `Session`. Để bảo đảm high availability, data node nên tránh trở thành single point.
4. Spring Session là một project dùng để quản lý session giữa nhiều server. Nó có thể tích hợp với nhiều backend storage (như Redis, MongoDB...), từ đó triển khai quản lý session phân tán. Thông qua Spring Session, dữ liệu session có thể được lưu trong storage bên ngoài dùng chung, nhằm đồng bộ và chia sẻ session giữa các server.

## Không có `Cookie` thì `Session` còn dùng được không?

Đây là một câu hỏi phỏng vấn kinh điển!

Thông thường, `SessionID` được lưu trong `Cookie`. Nếu dùng phương án lưu `SessionID` bằng `Cookie` mà client vô hiệu hóa `Cookie`, `Session` sẽ không thể hoạt động bình thường.

Tuy nhiên, server-side `Session` không đồng nghĩa với việc bắt buộc phải dùng `Cookie`. Client không phải trình duyệt có thể mang thông tin xác thực session trong request header theo quy ước rõ ràng. Tuy vậy, không nên đặt `SessionID` vào URL: ngay cả khi được mã hóa, nó vẫn là thông tin xác thực có thể sử dụng trực tiếp và có thể bị lộ trong lịch sử trình duyệt, access log, hệ thống monitoring và Referer. URL rewriting chỉ phù hợp với các trường hợp legacy cần tương thích bắt buộc, không nên dùng làm phương án đăng nhập cho hệ thống mới.

## Vì sao cần đặc biệt chú ý đến CSRF khi xác thực dựa trên `Cookie`?

**CSRF (Cross Site Request Forgery)** thường được dịch là **giả mạo request cross-site**. Vậy **giả mạo request cross-site** là gì? Nói đơn giản, đó là dùng danh tính của bạn để gửi những request gây bất lợi cho bạn. Ví dụ:

Tiểu Cường đăng nhập một ngân hàng trực tuyến, sau đó vào khu vực bài viết của ngân hàng và thấy bên dưới một bài viết có đường link ghi “Đầu tư tài chính khoa học, lợi nhuận hằng năm vượt 10.000”. Vì tò mò, Tiểu Cường nhấp vào link đó, kết quả tài khoản bị trừ 10.000. Chuyện gì đã xảy ra? Hóa ra hacker đã giấu một request trong link, request này trực tiếp lợi dụng danh tính của Tiểu Cường để gửi yêu cầu chuyển tiền đến ngân hàng, tức là gửi request đến ngân hàng thông qua `Cookie` của bạn.

```html
<a href="http://www.mybank.com/Transfer?bankId=11&money=10000"
  >Đầu tư tài chính khoa học, lợi nhuận hằng năm vượt 10.000</a
>
```

Như đã đề cập ở trên, khi xác thực bằng `Session`, chúng ta thường dùng `Cookie` để lưu `SessionId`. Sau khi đăng nhập, trình duyệt tự động đính kèm nó vào các request phù hợp với phạm vi của `Cookie`, còn server nhận diện người dùng thông qua `SessionId`. Nếu kẻ tấn công trực tiếp đánh cắp `SessionId`, đó là chiếm quyền session; còn CSRF thường không yêu cầu kẻ tấn công đọc `Cookie`, mà lợi dụng việc trình duyệt tự động gửi kèm `Cookie`.

Trong xác thực bằng `Session`, `SessionId` trong `Cookie` được trình duyệt gửi đến server. Dựa vào đặc điểm này, kẻ tấn công có thể khiến người dùng vô tình nhấp vào link tấn công để đạt được mục đích.

Nếu client dùng Token làm Bearer Token và đưa rõ ràng vào `Authorization` Header, trình duyệt sẽ không tự động đính kèm nó vào request cross-site như với `Cookie`, nhờ đó có thể giảm rủi ro CSRF truyền thống. Điều có tác dụng ở đây là cách mang thông tin xác thực, không phải bản thân định dạng Token hay JWT.

Không nên vì vậy mà mặc định lưu Token vào `localStorage` hoặc `sessionStorage`. Script độc hại trên trang cùng origin có thể đọc Web Storage, và chỉ một lỗ hổng XSS cũng có thể làm lộ Token trực tiếp. Ứng dụng trình duyệt có thể tùy tình huống chọn Backend For Frontend (BFF), hoặc dùng `Cookie` được thiết lập các thuộc tính `HttpOnly`, `Secure` và `SameSite` phù hợp; khi dùng `Cookie`, cũng nên kết hợp CSRF Token, kiểm tra `Origin`/`Referer` và các cơ chế khác.

![](https://oss.javaguide.cn/github/javaguide/system-design/security/20210615161108272.png)

Cần lưu ý rằng dù là `Cookie` hay `Token`, bản thân cơ chế xác thực đều không thể ngăn **tấn công script cross-site (Cross Site Scripting) XSS**. `HttpOnly` có thể giảm rủi ro script đọc trực tiếp `Cookie`, nhưng XSS vẫn có thể gửi request dưới danh tính người dùng. Vì vậy, vẫn cần encoding output đúng cách, sanitize HTML khi cần và các lớp phòng vệ chuyên sâu như CSP.

> Tên viết tắt của tấn công script cross-site (Cross Site Scripting) là CSS, nhưng sẽ bị nhầm với tên viết tắt của cascading style sheet (Cascading Style Sheets, CSS). Vì vậy, có người viết tắt tấn công script cross-site là XSS.

Trong XSS, kẻ tấn công dùng nhiều cách để chèn mã độc vào trang của người dùng khác. Từ đó, chúng có thể dùng script để đánh cắp thông tin như `Cookie`.

Khuyến nghị đọc: [Làm thế nào để ngăn chặn tấn công CSRF? — Đội ngũ kỹ thuật Meituan](https://tech.meituan.com/2018/10/11/fe-security-csrf.html)

Bạn cũng có thể tham khảo các thực hành an toàn:

- [OWASP Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- [OWASP Cross-Site Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)

## JWT là gì? JWT gồm những phần nào?

[Giải thích chi tiết các khái niệm cơ bản về JWT](./jwt-intro.md)

## Xác thực dựa trên JWT như thế nào? Làm thế nào để ngăn JWT bị giả mạo?

[Giải thích chi tiết các khái niệm cơ bản về JWT](./jwt-intro.md)

## SSO là gì?

SSO (Single Sign On), tức đăng nhập một lần, nghĩa là người dùng chỉ cần đăng nhập một trong nhiều subsystem là có quyền truy cập các hệ thống liên quan khác. Ví dụ, sau khi đăng nhập vào hệ thống tài chính của một sàn thương mại điện tử, ta cũng đồng thời đăng nhập thành công vào các subsystem như siêu thị, mua sắm quốc tế và thực phẩm tươi sống của sàn đó.

![Sơ đồ SSO](https://oss.javaguide.cn/github/javaguide/system-design/security/sso.png)

## SSO có lợi ích gì?

- **Góc độ người dùng**: Người dùng có thể đăng nhập một lần và sử dụng nhiều lần, không cần ghi nhớ nhiều bộ username và password, thuận tiện hơn.
- **Góc độ quản trị viên hệ thống**: Quản trị viên chỉ cần duy trì một account center thống nhất, rất thuận tiện.
- **Góc độ phát triển hệ thống mới**: Khi phát triển hệ thống mới, chỉ cần tích hợp trực tiếp với account center thống nhất, giúp đơn giản hóa quy trình phát triển và tiết kiệm thời gian.

## Thiết kế và triển khai một hệ thống SSO như thế nào?

[Giải thích chi tiết về SSO](./sso-intro.md)

## OAuth 2.0 là gì?

OAuth là một authorization protocol tiêu chuẩn của ngành, chủ yếu dùng để cấp cho ứng dụng bên thứ ba một số quyền hạn chế. OAuth 2.0 là thiết kế lại hoàn toàn OAuth 1.0, nhanh hơn và dễ triển khai hơn; OAuth 1.0 đã bị loại bỏ. Xem chi tiết tại: [rfc6749](https://tools.ietf.org/html/rfc6749).

Thực chất, đây là một cơ chế cấp quyền. Mục đích cuối cùng là cấp cho ứng dụng bên thứ ba một Token có thời hạn, để ứng dụng bên thứ ba dùng Token đó lấy các tài nguyên liên quan.

Trường hợp sử dụng phổ biến của OAuth 2.0 là đăng nhập bên thứ ba. Khi website tích hợp đăng nhập bên thứ ba, thông thường website sẽ dùng protocol OAuth 2.0.

Ngoài ra, OAuth 2.0 hiện cũng thường được dùng trong thanh toán (thanh toán qua WeChat, Alipay) và các developer platform (developer platform của WeChat, Alibaba...).

Dưới đây là sơ đồ [đăng nhập bên thứ ba bằng Slack OAuth 2.0](https://api.slack.com/legacy/oauth):

![](https://oss.javaguide.cn/github/javaguide/system-design/security/20210615151716340.png)

**Khuyến nghị đọc:**

- [Giải thích đơn giản về OAuth 2.0](http://www.ruanyifeng.com/blog/2019/04/oauth_design.html)
- [Hiểu protocol OAuth 2.0 là gì trong 10 phút](https://deepzz.com/post/what-is-oauth2-protocol.html)
- [Bốn phương thức của OAuth 2.0](http://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html)
- [Hướng dẫn ví dụ đăng nhập bên thứ ba bằng GitHub OAuth](http://www.ruanyifeng.com/blog/2019/04/github-oauth.html)

## Tham khảo

- Không dùng JWT để thay thế quản lý session: Tìm hiểu toàn diện về Token, JWT, OAuth, SAML, SSO: <https://zhuanlan.zhihu.com/p/38942172>
- Introduction to JSON Web Tokens: <https://jwt.io/introduction>
- JSON Web Token Claims: <https://auth0.com/docs/secure/tokens/json-web-tokens/json-web-token-claims>

<!-- @include: @article-footer.snippet.md -->
