---
title: Tổng hợp HTTP status code thường gặp (tầng ứng dụng)
description: Tổng hợp ý nghĩa và trường hợp sử dụng của các HTTP status code thường gặp, nhấn mạnh những điểm dễ nhầm như 201/204, giúp nâng cao hiệu quả thiết kế và debug API.
category: Computer Science Basics
tag:
  - Computer Networking
head:
  - - meta
    - name: keywords
      content: HTTP status code,2xx,3xx,4xx,5xx,redirect,mã lỗi,201 Created,204 No Content
---

HTTP status code là bản tóm tắt kết quả xử lý do server trả về cho client. Nhìn vào một status code, về cơ bản bạn có thể phán đoán request thành công, được redirect, gặp lỗi ở client hay gặp lỗi ở server.

Status code trông chỉ là những con số, nhưng nhiều status code rất dễ bị nhầm lẫn, chẳng hạn 301 và 302, 401 và 403, 500 và 502, 201 và 204.

Bài viết này chủ yếu trả lời một số câu hỏi:

1. 1xx, 2xx, 3xx, 4xx, 5xx lần lượt đại diện cho loại kết quả nào?
2. Các status code thành công thường gặp như 200, 201, 204 khác nhau thế nào?
3. Các lỗi client thường gặp như 400, 401, 403, 404 nên được hiểu thế nào?
4. Các lỗi server thường gặp như 500, 502, 503, 504 thường có ý nghĩa gì?

![HTTP status code thường gặp](https://oss.javaguide.cn/github/javaguide/cs-basics/network/http-status-code.png)

### 1xx Informational (mã trạng thái Informational)

So với các nhóm status code khác, bạn hầu như không gặp 1xx trong thực tế, nên ở đây bỏ qua.

### 2xx Success (mã trạng thái thành công)

- **200 OK**: Request được xử lý thành công. Ví dụ, gửi một HTTP request truy vấn dữ liệu người dùng đến server, server trả về chính xác dữ liệu người dùng. Đây là một trong những HTTP status code chúng ta thường gặp nhất.
- **201 Created**: Request được xử lý thành công và một ~~resource mới~~ được tạo trên server. Ví dụ, tạo một người dùng mới thông qua POST request.
- **202 Accepted**: Server đã nhận request nhưng chưa xử lý. Ví dụ, gửi một request cần server mất nhiều thời gian xử lý (như tạo report, export Excel), server đã nhận request nhưng chưa xử lý xong.
- **204 No Content**: Server đã xử lý request thành công nhưng không trả về nội dung nào. Ví dụ, gửi request xóa một người dùng, server xử lý thao tác xóa thành công nhưng không trả về nội dung nào.

🐛 Đính chính (tham khảo [issue#2458](https://github.com/Snailclimb/JavaGuide/issues/2458)): xét chính xác hơn, status code 201 Created là tạo một hoặc nhiều resource mới, có thể tham khảo: <https://httpwg.org/specs/rfc9110.html#status.201>.

![Định nghĩa status code 201 Created trong RFC 9110](https://oss.javaguide.cn/github/javaguide/cs-basics/network/rfc9110-201-created.png)

Ở đây cần nói riêng về status code 204, vì trong học tập/công việc hằng ngày bạn không gặp nó quá thường xuyên.

[Mô tả về status code 204 trong HTTP RFC 2616](https://tools.ietf.org/html/rfc2616#section-10.2.5) như sau:

> The server has fulfilled the request but does not need to return an
> entity-body, and might want to return updated metainformation. The
> response MAY include new or updated metainformation in the form of
> entity-headers, which if present SHOULD be associated with the
> requested variant.
>
> If the client is a user agent, it SHOULD NOT change its document view
> from that which caused the request to be sent. This response is
> primarily intended to allow input for actions to take place without
> causing a change to the user agent's active document view, although
> any new or updated metainformation SHOULD be applied to the document
> currently in the user agent's active view.
>
> The 204 response MUST NOT include a message-body, and thus is always
> terminated by the first empty line after the header fields.

Nói đơn giản, status code 204 mô tả trường hợp sau khi gửi HTTP request đến server, bạn chỉ quan tâm kết quả xử lý có thành công hay không. Nói cách khác, thứ bạn cần chỉ là một kết quả: true/false.

Lấy một ví dụ: bạn muốn theo đuổi một cô gái, bạn hỏi cô ấy: "Mình có thể theo đuổi bạn không?", cô ấy trả lời: "Được!". Coi cô gái này là server thì bạn sẽ dễ hiểu status code 204.

### 3xx Redirection (mã trạng thái redirect)

- **301 Moved Permanently**: Resource bị redirect vĩnh viễn. Ví dụ, website của bạn đổi địa chỉ.
- **302 Found**: Resource bị redirect tạm thời. Ví dụ, một số resource trên website của bạn tạm thời được chuyển sang địa chỉ khác.

### 4xx Client Error (mã trạng thái lỗi client)

- **400 Bad Request**: HTTP request được gửi đi có vấn đề. Ví dụ, tham số request không hợp lệ hoặc HTTP method không đúng.
- **401 Unauthorized**: Chưa authentication nhưng lại request resource yêu cầu authentication mới được truy cập.
- **403 Forbidden**: Từ chối trực tiếp HTTP request và không xử lý. Thường dùng cho các request bất hợp pháp.
- **404 Not Found**: Không tìm thấy resource bạn request trên server. Ví dụ, bạn request thông tin của một người dùng nhưng server không tìm thấy người dùng được chỉ định.
- **409 Conflict**: Resource trong request xung đột với trạng thái hiện tại của server, nên request không thể được xử lý.

### 5xx Server Error (mã trạng thái lỗi server)

- **500 Internal Server Error**: Server gặp vấn đề (thường do server phát sinh bug). Ví dụ, server đột nhiên ném exception khi xử lý request, nhưng exception không được xử lý đúng cách trên server.
- **502 Bad Gateway**: Gateway của chúng ta chuyển tiếp request đến server, nhưng server lại trả về một response lỗi.

### Tham khảo

- <https://www.restapitutorial.com/httpstatuscodes.html>
- <https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status>
- <https://en.wikipedia.org/wiki/List_of_HTTP_status_codes>
- <https://segmentfault.com/a/1190000018264501>

<!-- @include: @article-footer.snippet.md -->
