---
title: Phân tích ưu và nhược điểm của xác thực JWT
description: Phân tích chuyên sâu ưu và nhược điểm của xác thực JWT, giải thích các vấn đề như JWT không thể chủ động vô hiệu hóa, gia hạn Token và các giải pháp tương ứng.
category: System Design
tag:
  - Security
head:
  - - meta
    - name: keywords
      content: JWT, xác thực Token, xác thực stateless, nhược điểm của JWT, refresh token, vô hiệu hóa khi đăng xuất, rủi ro bảo mật, phương án thay thế
---

Trong các buổi phỏng vấn tuyển dụng sinh viên mới tốt nghiệp, phần lớn ứng viên sử dụng JWT cho việc xác thực và đăng nhập. Khi được hỏi các câu hỏi khái niệm về JWT và lý do sử dụng JWT, họ cơ bản đều có thể trả lời một vài ý. Tuy nhiên, khi được hỏi về một số vấn đề tồn tại của JWT và giải pháp, chỉ một phần nhỏ ứng viên trả lời khá tốt.

JWT không phải là viên đạn bạc và cũng có nhiều thiếu sót, nhiều lúc không phải lựa chọn tối ưu. Trong bài viết này, chúng ta sẽ cùng tìm hiểu ưu, nhược điểm của xác thực JWT và cách giải quyết các vấn đề thường gặp, đồng thời xem vì sao nhiều người không còn khuyến nghị sử dụng JWT nữa.

Để xem phần giới thiệu các khái niệm cơ bản về JWT, hãy đọc bài viết tôi đã viết: [Giải thích chi tiết các khái niệm cơ bản về JWT](https://javaguide.cn/system-design/security/jwt-intro.html).

## Ưu điểm của JWT

So với phương thức xác thực bằng Session, sử dụng JWT để xác thực danh tính chủ yếu có 4 ưu điểm sau.

### Stateless

Bản thân JWT chứa toàn bộ thông tin cần thiết cho việc xác thực danh tính, vì vậy server không cần lưu trữ thông tin JWT. Điều này rõ ràng làm tăng availability và scalability của hệ thống, đồng thời giảm đáng kể áp lực cho server.

Tuy nhiên, chính vì JWT stateless nên nó cũng dẫn đến nhược điểm lớn nhất: **không thể kiểm soát!**

Ví dụ, nếu muốn vô hiệu hóa một JWT hoặc thay đổi quyền của nó trong thời hạn hiệu lực, thay đổi đó sẽ không có hiệu lực ngay mà thường phải chờ đến khi JWT hết hạn. Tương tự, khi người dùng Logout, JWT vẫn còn hiệu lực. Trừ khi bổ sung logic xử lý ở backend, chẳng hạn lưu các JWT đã vô hiệu hóa, để backend xác minh JWT còn hiệu lực hay không trước khi xử lý. Cách giải quyết cụ thể sẽ được giới thiệu chi tiết ở phần sau; ở đây chỉ nhắc sơ qua.

### Sử dụng Authorization Header có thể giảm rủi ro CSRF truyền thống

**CSRF (Cross Site Request Forgery)** thường được dịch là **giả mạo yêu cầu liên trang**, thuộc lĩnh vực tấn công mạng. So với các hình thức tấn công bảo mật như SQL injection và XSS, CSRF không được biết đến rộng rãi bằng. Tuy nhiên, đây thực sự là rủi ro bảo mật phải được cân nhắc khi phát triển hệ thống. Ngay cả Gmail, sản phẩm được xem là tiêu chuẩn kỹ thuật trong ngành của Google, cũng từng bị phát hiện lỗ hổng CSRF vào năm 2007, gây thiệt hại lớn cho người dùng Gmail.

**Vậy giả mạo yêu cầu liên trang thực chất là gì?** Nói đơn giản, đó là dùng danh tính của bạn để thực hiện những việc không tốt (gửi các yêu cầu gây bất lợi cho bạn, chẳng hạn chuyển tiền trái phép).

Ví dụ đơn giản: Tiểu Tráng đăng nhập vào một ngân hàng trực tuyến, sau đó vào khu vực bài đăng của ngân hàng và thấy một liên kết dưới một bài đăng có nội dung “Đầu tư thông minh, lợi nhuận hàng năm trên 10.000”. Vì tò mò, Tiểu Tráng nhấp vào liên kết và phát hiện tài khoản bị trừ 10.000 nhân dân tệ. Chuyện gì đã xảy ra? Hóa ra hacker đã giấu một request trong liên kết. Request này trực tiếp lợi dụng danh tính của Tiểu Tráng để gửi yêu cầu chuyển tiền đến ngân hàng, tức là gửi request đến ngân hàng thông qua Cookie của bạn.

```html
<a href="http://www.mybank.com/Transfer?bankId=11&money=10000"
  >Đầu tư thông minh, lợi nhuận hàng năm trên 10.000</a
>
```

Tấn công CSRF truyền thống lợi dụng đặc điểm trình duyệt tự động đính kèm thông tin xác thực. Thông tin xác thực phổ biến nhất là `SessionID` trong Cookie. Ngay cả khi kẻ tấn công không thể đọc `SessionID`, chúng vẫn có thể dụ trình duyệt mang nó gửi request đến site mục tiêu.

Ngoài ra, không nhất thiết phải nhấp vào liên kết mới có thể thực hiện cuộc tấn công. Trong nhiều trường hợp, chỉ cần bạn mở một trang nào đó là tấn công CSRF đã xảy ra.

```html
<img src="http://www.mybank.com/Transfer?bankId=11&money=10000" />
```

**Vậy tại sao khi sử dụng JWT, người ta thường nói rủi ro CSRF thấp hơn?**

Nếu client sử dụng JWT dưới dạng Bearer Token và đặt rõ nó vào HTTP `Authorization` Header, trình duyệt sẽ không tự động đính kèm nó vào request cross-site như Cookie. Vì vậy, cách này có thể giảm rủi ro CSRF truyền thống. Yếu tố phát huy tác dụng ở đây là cách truyền thông tin xác thực, không phải bản thân định dạng dữ liệu JWT.

Tuy nhiên, không nên vì vậy mà mặc định lưu JWT vào `localStorage`. Mọi script độc hại trong trang cùng origin đều có thể đọc Web Storage; chỉ một lỗ hổng XSS cũng có thể khiến access token hoặc refresh token bị đánh cắp trực tiếp. Ứng dụng trình duyệt cần lựa chọn phương án dựa trên threat model, chẳng hạn sử dụng Cookie có `HttpOnly`, `Secure` và thuộc tính `SameSite` phù hợp, hoặc dùng Backend For Frontend (BFF) để giữ token ở server.

Nếu sử dụng Cookie để lưu thông tin xác thực đăng nhập, cần đồng thời triển khai tốt cơ chế phòng chống CSRF, chẳng hạn CSRF Token, kiểm tra `Origin`/`Referer` và Cookie `SameSite`. `SameSite` thường nên được dùng như một lớp phòng thủ chiều sâu, không thể trong mọi cách triển khai tự thay thế CSRF Token.

Việc phòng chống XSS không thể dựa vào một “bộ lọc chuỗi đáng ngờ” dùng chung. Cách đáng tin cậy hơn là mã hóa đúng theo từng context khác nhau khi xuất dữ liệu ra HTML, attribute, JavaScript, CSS, URL; khi thực sự cho phép người dùng gửi HTML, hãy sử dụng thư viện HTML sanitizer trưởng thành được cập nhật liên tục; sau đó dùng các cơ chế như CSP để cung cấp thêm lớp phòng thủ chiều sâu.

### Phù hợp với ứng dụng mobile

Nếu sử dụng Session để xác thực danh tính, cần lưu một phần thông tin ở server và phương thức này phụ thuộc vào Cookie (cần Cookie để lưu `SessionId`), vì vậy không phù hợp với ứng dụng mobile.

Tuy nhiên, xác thực bằng JWT không có vấn đề này, vì chỉ cần client có thể lưu trữ JWT là có thể sử dụng; hơn nữa JWT còn có thể được sử dụng giữa nhiều ngôn ngữ.

> Vì sao sử dụng Session để xác thực danh tính lại không phù hợp với mobile?
>
> 1. Quản lý trạng thái: Session dựa trên việc quản lý trạng thái ở server, trong khi ứng dụng mobile thường là stateless. Kết nối của thiết bị mobile có thể không ổn định hoặc bị gián đoạn, nên khó duy trì trạng thái phiên lâu dài. Nếu sử dụng Session để xác thực danh tính, ứng dụng mobile phải thường xuyên duy trì phiên với server, làm tăng chi phí mạng và độ phức tạp;
> 2. Tính tương thích: Ứng dụng mobile thường hướng tới nhiều nền tảng như iOS, Android và Web. Cách quản lý và lưu trữ Session của mỗi nền tảng có thể khác nhau, dẫn đến vấn đề tương thích giữa các nền tảng;
> 3. Tính bảo mật: Thiết bị mobile thường hoạt động trong môi trường mạng không đáng tin cậy, tồn tại rủi ro rò rỉ dữ liệu và bị tấn công. Lưu thông tin phiên nhạy cảm trên thiết bị mobile làm tăng nguy cơ bị tấn công.

### Thân thiện với Single Sign-On

Nếu sử dụng Session để xác thực danh tính, để triển khai Single Sign-On, cần lưu thông tin Session của người dùng trên một máy tính, đồng thời còn gặp vấn đề Cookie cross-domain thường thấy. Nhưng nếu xác thực bằng JWT, JWT được lưu ở client nên không gặp các vấn đề này.

## Các vấn đề thường gặp và cách giải quyết trong xác thực JWT

### JWT vẫn còn hiệu lực trong các trường hợp đăng xuất và tương tự

Các trường hợp cụ thể tương tự gồm:

- Đăng xuất;
- Đổi mật khẩu;
- Server thay đổi quyền hoặc role của một người dùng;
- Tài khoản người dùng bị khóa/xóa;
- Người dùng bị server buộc đăng xuất;
- Người dùng bị đẩy ra khỏi hệ thống;
- ……

Vấn đề này không tồn tại trong phương thức xác thực bằng Session, vì khi gặp trường hợp như vậy, server chỉ cần xóa bản ghi Session tương ứng. Nhưng phương thức xác thực bằng JWT lại khó giải quyết hơn. Như đã nói, một khi JWT được phát hành, nếu backend không bổ sung logic khác thì JWT vẫn có hiệu lực cho đến trước thời điểm hết hạn.

Vậy giải quyết vấn đề này như thế nào? Sau khi tham khảo nhiều tài liệu, tôi tóm tắt 4 phương án dưới đây:

**1. Lưu JWT vào database**

Lưu JWT còn hiệu lực vào database, trong đó nên ưu tiên database in-memory như Redis. Nếu cần vô hiệu hóa một JWT, chỉ cần xóa JWT đó khỏi Redis. Tuy nhiên, cách này khiến mỗi lần sử dụng JWT đều phải truy vấn Redis để kiểm tra JWT có tồn tại hay không, đồng thời đi ngược lại nguyên tắc stateless của JWT.

**2. Cơ chế blacklist**

Tương tự phương án trên, dùng database in-memory như Redis để duy trì một blacklist. Nếu muốn vô hiệu hóa một JWT, chỉ cần đưa JWT đó vào **blacklist**. Sau đó, mỗi request sử dụng JWT đều phải kiểm tra JWT đó có nằm trong blacklist hay không.

Cốt lõi của hai phương án đầu là lưu trữ các JWT còn hiệu lực hoặc đưa JWT được chỉ định vào blacklist.

Mặc dù hai phương án này đều đi ngược lại nguyên tắc stateless của JWT, trong các dự án thực tế, chúng ta thường vẫn sử dụng chúng.

**3. Thay đổi secret (Secret)**

Tạo một secret riêng cho mỗi người dùng. Nếu muốn vô hiệu hóa một JWT, chỉ cần thay đổi secret tương ứng của người dùng. Tuy nhiên, so với hai phương án đầu vốn đưa database in-memory vào hệ thống, cách này gây ra rủi ro lớn hơn:

- Nếu service là distributed, mỗi lần phát hành JWT mới đều phải đồng bộ secret trên nhiều máy. Khi đó, cần lưu secret trong database hoặc service bên ngoài, nên không còn khác Session là bao.
- Nếu người dùng đồng thời mở hệ thống trên hai trình duyệt hoặc cả trên điện thoại, khi đăng xuất tài khoản ở một nơi thì các nơi khác cũng phải đăng nhập lại, đây là điều không mong muốn.

**4. Giữ thời hạn hiệu lực của token ngắn và thường xuyên rotation**

Đây là một phương án rất đơn giản. Tuy nhiên, trạng thái đăng nhập của người dùng sẽ không được duy trì lâu dài và người dùng phải đăng nhập thường xuyên.

Ngoài ra, vấn đề JWT vẫn còn hiệu lực sau khi đổi mật khẩu tương đối dễ giải quyết. Một cách mà tôi cho là khá tốt là: **dùng hash của mật khẩu người dùng để ký JWT. Vì vậy, nếu mật khẩu thay đổi, mọi token trước đó sẽ tự động không thể được xác minh.**

### Vấn đề gia hạn JWT

Thời hạn hiệu lực của JWT thường được khuyến nghị đặt không quá dài. Vậy sau khi JWT hết hạn thì xác thực thế nào, làm sao refresh JWT một cách động để người dùng không phải đăng nhập lại thường xuyên?

Trước hết, hãy xem cách làm phổ biến trong xác thực bằng Session: **giả sử thời hạn hiệu lực của Session là 30 phút, nếu người dùng có truy cập trong 30 phút đó thì kéo dài thời hạn hiệu lực của Session thêm 30 phút.**

Với xác thực JWT, giải quyết vấn đề gia hạn như thế nào? Sau khi tham khảo nhiều tài liệu, tôi tóm tắt 4 phương án dưới đây:

**1. Cách làm tương tự xác thực bằng Session (không khuyến nghị)**

Phương án này đáp ứng phần lớn trường hợp. Giả sử server đặt thời hạn hiệu lực của JWT là 30 phút. Mỗi lần kiểm tra, nếu phát hiện JWT sắp hết hạn, server sẽ tạo JWT mới và trả cho client. Mỗi request, client kiểm tra JWT mới và cũ; nếu không giống nhau thì cập nhật JWT cục bộ. Vấn đề của cách làm này là client chỉ cập nhật JWT khi gần hết hạn, không thật sự thân thiện với client.

**2. Mỗi request đều trả về JWT mới (không khuyến nghị)**

Ý tưởng của phương án này rất đơn giản, nhưng chi phí sẽ khá lớn, đặc biệt khi server phải lưu trữ và duy trì JWT.

**3. Đặt thời hạn hiệu lực của JWT đến nửa đêm (không khuyến nghị)**

Đây là một phương án thỏa hiệp, bảo đảm phần lớn người dùng có thể đăng nhập bình thường vào ban ngày, phù hợp với các hệ thống không yêu cầu cao về bảo mật.

**4. Sử dụng access token ngắn hạn và refresh token dài hạn (khuyến nghị)**

Một token là access token ngắn hạn, chẳng hạn hết hạn sau nửa giờ; token còn lại là refresh token có vòng đời dài hơn, chỉ dùng để lấy access token mới. Không nhất thiết cả hai đều phải ở định dạng JWT. Refresh token có quyền cao và thời gian tồn tại dài, là thông tin xác thực mà kẻ tấn công ưu tiên đánh cắp; không được cho rằng nó “khó bị rò rỉ” chỉ vì ít được sử dụng.

Sau khi client đăng nhập, mỗi lần truy cập sẽ mang theo access token. Khi access token hết hạn, client dùng refresh token được bảo vệ để đổi lấy access token mới. Ứng dụng trình duyệt không nên mặc định đặt refresh token vào `localStorage`; có thể dùng BFF hoặc Cookie được bảo vệ để giảm rủi ro token bị script đọc trực tiếp.

Nhược điểm của phương án này:

- Cần client phối hợp;
- Khi người dùng đăng xuất, đổi mật khẩu hoặc xảy ra sự kiện bảo mật khác, cần thu hồi quyền refresh tương ứng;
- Trong quá trình request lại để lấy JWT, sẽ có một khoảng thời gian ngắn JWT không khả dụng (có thể đặt timer ở client, khi accessJWT sắp hết hạn thì chủ động dùng refreshJWT để lấy accessJWT mới);
- Với public client, authorization server cần sử dụng refresh token rotation và phát hiện việc replay token cũ, hoặc sử dụng refresh token có sender constraint. Refresh token cũng cần được ràng buộc với client, authorization scope và resource server, đồng thời đặt thời gian hết hạn khi không hoạt động.

### JWT có kích thước quá lớn

Cấu trúc JWT phức tạp (Header, Payload và Signature), chứa nhiều thông tin bổ sung hơn và còn cần được mã hóa bằng Base64Url, khiến JWT có kích thước khá lớn và làm tăng chi phí truyền dữ liệu qua mạng.

Thành phần của JWT:

![Cấu tạo JWT](https://oss.javaguide.cn/javaguide/system-design/jwt/jwt-composition.png)

Ví dụ JWT:

```plain
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

Cách giải quyết:

- Hạn chế thông tin trong JWT Payload (phần dữ liệu), chỉ giữ lại thông tin người dùng và quyền cần thiết.
- Trước khi truyền JWT, dùng thuật toán nén (như GZIP) để nén JWT nhằm giảm kích thước.
- Trong một số trường hợp, Token truyền thống có thể phù hợp hơn. Token truyền thống thường chỉ là một định danh duy nhất; thông tin tương ứng (ví dụ user ID, thời hạn hết hạn của Token, thông tin quyền) được lưu ở server và thường được lưu bằng Redis.

## Tổng kết

Một ưu điểm rất quan trọng của JWT là stateless, nhưng trên thực tế, nếu muốn sử dụng JWT một cách hợp lý cho xác thực và đăng nhập trong dự án thực tế thì vẫn cần lưu thông tin JWT.

JWT cũng không phải là viên đạn bạc và có nhiều thiếu sót. Việc chọn JWT hay Session vẫn phải dựa trên nhu cầu cụ thể của dự án. Tuyệt đối không nên mù quáng ca ngợi JWT rồi xem thường các phương án xác thực danh tính khác.

Ngoài ra, không dùng JWT mà sử dụng Token thông thường (ID được tạo ngẫu nhiên, không chứa thông tin cụ thể) kết hợp với Redis để xác thực danh tính cũng là một lựa chọn khả thi.

## Tài liệu tham khảo

- RFC 9700 - Best Current Practice for OAuth 2.0 Security: <https://www.rfc-editor.org/rfc/rfc9700.html>
- OWASP Session Management Cheat Sheet: <https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html>
- OWASP Cross Site Scripting Prevention Cheat Sheet: <https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html>
- Phân tích JWT cực kỳ chi tiết: <https://learnku.com/articles/17883>
- How to log out when using JWT: <https://medium.com/devgorilla/how-to-log-out-when-using-jwt-a8c7823e8a6>
- CSRF protection with JSON Web JWTs: <https://medium.com/@agungsantoso/csrf-protection-with-json-web-JWTs-83e0f2fcbcc>
- Invalidating JSON Web JWTs: <https://stackoverflow.com/questions/21978658/invalidating-json-web-JWTs>

<!-- @include: @article-footer.snippet.md -->
