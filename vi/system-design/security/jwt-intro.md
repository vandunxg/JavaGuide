---
title: JWT là gì? Giải thích chi tiết cấu trúc, phân tích và xác thực đăng nhập JWT
description: JWT (JSON Web Token) là gì? Bài viết giải thích qua ví dụ cấu trúc gồm ba phần Header, Payload, Signature, phân tích Base64Url, xác minh chữ ký, thời gian hết hạn và quy trình xác thực đăng nhập, đồng thời chỉ ra các rủi ro bảo mật thường gặp.
category: System Design
tag:
  - Security
head:
  - - meta
    - name: keywords
      content: JWT là gì,phân tích JWT,JWT Token,xác thực JWT,JSON Web Token,xác thực Token,stateless,Header Payload Signature,thuật toán chữ ký,xác thực đăng nhập,CSRF
---

<!-- @include: @article-header.snippet.md -->

## JWT là gì?

JWT (JSON Web Token) là một định dạng biểu diễn claim nhỏ gọn, an toàn với URL, được định nghĩa trong [RFC 7519](https://datatracker.ietf.org/doc/html/rfc7519). Sau khi đăng nhập thành công, server có thể ghi các claim như định danh người dùng, phạm vi quyền và thời gian hết hạn vào JWT; client mang JWT trong các request tiếp theo, server xác minh chữ ký và các claim liên quan rồi mới quyết định có cho phép hay không.

JWT có thể mang các claim cần thiết cho việc xác thực, vì vậy server không nhất thiết phải lưu trạng thái phiên như mô hình Session truyền thống. Tuy nhiên, các nhu cầu như thu hồi token, thay đổi quyền và chủ động đăng xuất vẫn có thể cần trạng thái phía server; không thể chỉ dựa vào việc "sử dụng JWT" mà kết luận hệ thống hoàn toàn stateless.

Header và Payload của JWT chỉ được encode bằng Base64Url, người có token đều có thể decode chúng, vì vậy không được ghi thông tin nhạy cảm như mật khẩu, số căn cước vào Payload. Chữ ký dùng để kiểm tra nội dung có bị chỉnh sửa hay không, không có tác dụng mã hóa nội dung.

Nếu client đặt JWT dưới dạng Bearer Token trong `Authorization` Header, trình duyệt sẽ không tự động đính kèm nó như Cookie, nhờ đó có thể giảm rủi ro CSRF truyền thống. Tuy nhiên, điều này phụ thuộc vào cách truyền và lưu credential, không phải bản thân định dạng JWT; nếu đặt JWT trong Cookie thì vẫn cần cơ chế bảo vệ CSRF.

[Phân tích ưu và nhược điểm của JWT](./advantages-and-disadvantages-of-jwt.md) giới thiệu chi tiết ưu điểm và hạn chế của việc dùng JWT để xác thực danh tính.

Dưới đây là định nghĩa về JWT trong [RFC 7519](https://datatracker.ietf.org/doc/html/rfc7519).

> JSON Web Token (JWT) is a compact, URL-safe means of representing claims to be transferred between two parties. The claims in a JWT are encoded as a JSON object that is used as the payload of a JSON Web Signature (JWS) structure or as the plaintext of a JSON Web Encryption (JWE) structure, enabling the claims to be digitally signed or integrity protected with a Message Authentication Code (MAC) and/or encrypted. ——[JSON Web Token (JWT)](https://datatracker.ietf.org/doc/html/rfc7519)

## JWT gồm những phần nào?

![Cấu tạo JWT](https://oss.javaguide.cn/javaguide/system-design/jwt/jwt-composition.png)

JWT thường gồm ba phần được encode bằng Base64Url và phân cách bằng `.`:

- **Header (phần đầu)**: mô tả metadata của JWT, gồm loại token và thuật toán chữ ký. Header trở thành phần thứ nhất của JWT sau khi được encode bằng Base64Url.
- **Payload (tải trọng)**: lưu các claim cần truyền, chẳng hạn `sub` (subject, chủ thể), `jti` (JWT ID). Payload trở thành phần thứ hai của JWT sau khi được encode bằng Base64Url.
- **Signature (chữ ký)**: được tính dựa trên Header, Payload đã encode, thuật toán chữ ký và signing key. HS256 sử dụng shared key, còn các thuật toán bất đối xứng như RS256, ES256 sử dụng private key để ký và public key để xác minh.

JWT thường có dạng: `xxxxx.yyyyy.zzzzz`.

Ví dụ:

```plain
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

Bạn có thể decode JWT mẫu trên [jwt.io](https://jwt.io/); sau khi decode sẽ thấy ba phần Header, Payload và Signature. Token thực tế trong môi trường production có thể chứa định danh và thông tin quyền của người dùng, không được sao chép chúng vào công cụ trực tuyến của bên thứ ba.

Header và Payload đều là dữ liệu JSON, còn Signature được tính từ Header, Payload đã encode và signing key.

![](https://oss.javaguide.cn/javaguide/system-design/jwt/jwt.io.png)

### Phân tích JWT và xác minh JWT khác nhau thế nào?

Phân tích JWT chỉ decode Header và Payload bằng Base64Url, không cần key. Bất kỳ ai có token đều có thể thực hiện việc phân tích, vì vậy kết quả phân tích không chứng minh token đáng tin cậy.

Xác minh JWT sử dụng thuật toán và key được chỉ định để kiểm tra Signature, đồng thời cần kiểm tra các claim như `exp`, `nbf`, `iss`, `aud`. Chỉ khi chữ ký và toàn bộ claim theo yêu cầu nghiệp vụ đều vượt qua kiểm tra, server mới có thể tin cậy thông tin danh tính và quyền trong token.

### Header

Header thường gồm hai phần:

- `typ` (Type): loại token, tức JWT.
- `alg` (Algorithm): thuật toán chữ ký, chẳng hạn HS256.

Ví dụ:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Header ở dạng JSON trở thành phần thứ nhất của JWT sau khi được encode bằng Base64Url.

### Payload

Payload cũng là dữ liệu JSON, chứa các Claims (claim).

Claims được chia thành ba loại:

- **Registered Claims (claim đã đăng ký)**: một số claim được định nghĩa sẵn, nên sử dụng nhưng không bắt buộc.
- **Public Claims (claim công khai)**: claim do bên phát hành JWT tự định nghĩa; để tránh xung đột, nên định nghĩa chúng trong [IANA JSON Web Token Registry](https://www.iana.org/assignments/jwt/jwt.xhtml).
- **Private Claims (claim riêng tư)**: claim do bên phát hành JWT tự định nghĩa theo nhu cầu của dự án, phù hợp hơn với các trường hợp sử dụng thực tế.

Dưới đây là một số claim đã đăng ký thường gặp:

- `iss` (issuer): bên phát hành JWT.
- `iat` (issued at time): thời điểm phát hành JWT.
- `sub` (subject): chủ thể của JWT.
- `aud` (audience): bên nhận JWT.
- `exp` (expiration time): thời gian hết hạn của JWT.
- `nbf` (not before time): thời điểm JWT bắt đầu có hiệu lực; JWT có thời điểm sớm hơn thời điểm này không được chấp nhận xử lý.
- `jti` (JWT ID): định danh duy nhất của JWT.

Ví dụ:

```json
{
  "uid": "ff1212f5-d8d1-4496-bf41-d2dda73de19a",
  "sub": "1234567890",
  "name": "John Doe",
  "exp": 15323232,
  "iat": 1516239022,
  "scope": ["admin", "user"]
}
```

Phần Payload mặc định không được mã hóa, **tuyệt đối không lưu thông tin riêng tư trong Payload!!!**

Payload ở dạng JSON trở thành phần thứ hai của JWT sau khi được encode bằng Base64Url.

### Signature

Phần Signature là chữ ký của hai phần trước, có tác dụng ngăn JWT (chủ yếu là payload) bị chỉnh sửa.

Việc tạo chữ ký này cần:

- Header + Payload.
- Signing key được lưu ở server. Khi dùng thuật toán bất đối xứng, không được để lộ private key dùng để ký.
- Thuật toán chữ ký.

Công thức tính chữ ký:

```plain
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

Sau khi tính chữ ký, ghép Header, Payload và Signature thành một chuỗi; các phần được phân cách bằng "dấu chấm" (`.`), chuỗi này chính là JWT.

## Xác thực danh tính dựa trên JWT như thế nào?

Trong ứng dụng xác thực danh tính dựa trên JWT, server tạo JWT từ Payload, Header và key rồi gửi JWT cho client. Client cần lưu token một cách an toàn, phù hợp với hình thức ứng dụng và threat model; các request gửi sau đó sẽ mang token này.

![Sơ đồ xác thực danh tính bằng JWT](https://oss.javaguide.cn/github/javaguide/system-design/jwt/jwt-authentication%20process.png)

Các bước đơn giản hóa:

1. Người dùng gửi username, password và captcha cho server để đăng nhập hệ thống;
2. Nếu username, password và captcha của người dùng được xác minh chính xác, server trả về Token đã ký, tức JWT;
3. Client nhận Token rồi lưu an toàn; ứng dụng trình duyệt có thể dùng BFF để giữ token ở server, hoặc sử dụng Cookie được bảo vệ tùy trường hợp;
4. Mỗi lần gửi request đến backend sau đó, người dùng đều đặt JWT này trong Header;
5. Server kiểm tra JWT và lấy thông tin liên quan đến người dùng từ đó.

Hai khuyến nghị:

1. Không mặc định lưu JWT trong `localStorage` hoặc `sessionStorage`. Mọi script độc hại trên trang cùng origin đều có thể đọc Web Storage; chỉ một lỗ hổng XSS cũng có thể làm lộ token. Khi dùng Cookie, nên đặt thuộc tính `HttpOnly`, `Secure` và `SameSite` phù hợp, đồng thời thực hiện đầy đủ cơ chế bảo vệ CSRF.
2. Cách phổ biến để mang JWT trong phương án không dùng Cookie là đặt nó trong trường `Authorization` của HTTP Header (`Authorization: Bearer Token`).

**[spring-security-jwt-guide](https://github.com/Snailclimb/spring-security-jwt-guide)** là một ví dụ đơn giản về xác thực danh tính bằng JWT, bạn có thể tham khảo nếu quan tâm.

## Làm thế nào để ngăn JWT bị chỉnh sửa?

Với chữ ký đã được kiểm tra chính xác, ngay cả khi JWT bị lộ hoặc bị chặn bắt, kẻ tấn công cũng không thể chỉnh sửa Header hoặc Payload rồi tạo chữ ký hợp lệ nếu không biết signing key. Tuy nhiên, chữ ký không cung cấp tính bảo mật thông tin và cũng không ngăn được kẻ tấn công replay JWT hợp lệ đã bị đánh cắp.

Vì sao lại như vậy? Sau khi nhận JWT, server sẽ phân tích Header, Payload và Signature chứa trong đó. Server tạo lại một Signature dựa trên Header, Payload và key. Server so sánh Signature mới tạo với Signature trong JWT; nếu giống nhau thì Header và Payload chưa bị chỉnh sửa.

Tuy nhiên, nếu key của server cũng bị lộ, kẻ tấn công có thể chỉnh sửa Header và Payload rồi tạo lại một Signature hợp lệ.

Signing key phải được bảo quản cẩn thận và cần có cơ chế rotation và thu hồi.

## Làm thế nào để tăng cường bảo mật cho JWT?

1. Sử dụng thư viện open source đã được kiểm chứng, không tự triển khai logic mã hóa, giải mã và kiểm tra JWT.
2. Server cố định tập thuật toán được phép sử dụng, không trực tiếp tin lựa chọn thuật toán xác minh `alg` trong Header của JWT; key HMAC phải có độ ngẫu nhiên và độ dài đủ lớn.
3. Xác minh mọi claim liên quan đến ứng dụng hiện tại, bao gồm `iss`, `aud`, `exp` và `nbf`, đồng thời đặt giới hạn rõ ràng cho sai lệch clock được cho phép.
4. Với JWT có các mục đích khác nhau như ID Token và Access Token, sử dụng `typ` tường minh cùng các quy tắc kiểm tra loại trừ lẫn nhau để ngăn một token bị thay thế vào bối cảnh khác.
5. Tuyệt đối không lưu thông tin riêng tư trong Payload chưa được mã hóa, đồng thời không coi Claim đã nhận nhưng chưa được xác minh là input đáng tin cậy.
6. Chọn cách lưu token an toàn theo loại client, giới hạn thời gian hiệu lực, phạm vi quyền và bên nhận token; với trường hợp rủi ro cao, cần cân nhắc thêm việc thu hồi, phát hiện replay hoặc ràng buộc người gửi.
7. Key phải được bảo quản cẩn thận và hỗ trợ rotation. Có thể tham khảo các yêu cầu bảo mật đầy đủ hơn trong [RFC 8725: JSON Web Token Best Current Practices](https://www.rfc-editor.org/rfc/rfc8725.html).

<!-- @include: @article-footer.snippet.md -->
