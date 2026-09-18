---
title: Hướng dẫn ngắn gọn về RESTful API
description: Giải thích chi tiết quy tắc thiết kế RESTful API, bao quát nguyên tắc kiến trúc REST, thiết kế đường dẫn tài nguyên, cách sử dụng phương thức HTTP và quy tắc mã trạng thái.
category: Chất lượng code
head:
  - - meta
    - name: keywords
      content: RESTful API,REST,thiết kế API,đường dẫn tài nguyên,phương thức HTTP,mã trạng thái,idempotent,quy ước API
---

Bài viết này giới thiệu ngắn gọn về kiến thức liên quan đến RESTful API mà backend developer cần có.

Trước khi chính thức giới thiệu RESTful API, trước hết chúng ta cần hiểu rõ: **API rốt cuộc là gì?**

## API là gì?

**API (Application Programming Interface)** dịch ra là giao diện lập trình ứng dụng.

Khi phát triển backend, công việc chính của chúng ta là cung cấp API cho frontend hoặc các backend service khác, chẳng hạn API truy vấn dữ liệu người dùng.

![](https://oss.javaguide.cn/github/javaguide/system-design/basis/20210507130629538.png)

Tuy nhiên, API không chỉ đại diện cho interface mà backend system expose; các method do framework cung cấp cũng thuộc phạm vi của API.

Để mọi người dễ hiểu hơn, tôi liệt kê thêm vài ví dụ 🌰:

1. Khi bạn tìm kiếm một sản phẩm nào đó trên một trang thương mại điện tử, frontend của trang đó đã gọi API liên quan đến tìm kiếm sản phẩm do backend cung cấp.
2. Khi dùng JDK để phát triển chương trình Java và muốn đọc input của người dùng, bạn cần sử dụng API liên quan đến IO do JDK cung cấp.
3. ……

Bạn có thể hiểu API là cầu nối giao tiếp giữa các chương trình với nhau, về bản chất chỉ là một function. Ngoài ra, việc sử dụng API cũng có quy tắc; các quy tắc đó, chẳng hạn format của input và output, do bên cung cấp API quy định.

## RESTful API là gì?

**RESTful API** thường còn được gọi là **REST API**, là API được xây dựng dựa trên REST. REST rốt cuộc là gì sẽ được trình bày ở phần sau vì nó liên quan đến khá nhiều khái niệm.

Nếu đọc các bài viết liên quan đến RESTful API, bạn thường thấy khá khó hiểu, chủ yếu vì một số khái niệm liên quan đến REST không dễ nắm bắt. Tuy nhiên, trên thực tế, kiến thức về RESTful API được sử dụng trong phát triển hằng ngày rất đơn giản và dễ khái quát!

Ví dụ, nếu tôi đưa cho bạn hai API dưới đây, bạn có thể lập tức biết chúng dùng để làm gì! Đó chính là điểm mạnh của RESTful API!

```plain
GET    /classes: liệt kê tất cả class
POST   /classes: tạo một class mới
```

**RESTful API giúp bạn nhìn vào URL + HTTP Method là biết URL đó dùng để làm gì; nhìn vào HTTP status code là biết kết quả request ra sao.**

Khi thiết kế API trong quá trình phát triển, ít nhất chúng ta cũng nên đáp ứng các yêu cầu cơ bản nhất của RESTful API, chẳng hạn ưu tiên dùng danh từ trong interface, dùng request `POST` để tạo resource, request `DELETE` để xoá resource, v.v. Ví dụ: `GET /notes/{id}`: lấy thông tin note được chỉ định.

## Giải thích REST

**REST** là viết tắt của `REpresentational State Transfer`. Cụm từ này được dịch là “**chuyển trạng thái biểu diễn**”.

Cách hiểu này khá khó hình dung. Nói trực tiếp, REST mô tả việc client chuyển từ application state này sang application state khác thông qua “biểu diễn” của resource. Nếu vẫn chưa hiểu, hãy tiếp tục đọc; phần giải thích dưới đây chắc chắn sẽ giúp bạn hiểu REST là gì.

Chúng ta sẽ lần lượt giải thích các khái niệm liên quan ở trên để hiểu sâu hơn. Thực ra, bạn không cần nắm rõ các khái niệm dưới đây vẫn có thể đọc phần tiếp theo. Tuy nhiên, nếu muốn trao đổi với người khác về “RESTful API”, tôi vẫn khuyên bạn nên hiểu kỹ chúng!

- **Resource**: Có thể gọi dữ liệu của các đối tượng thực tế là resource. Một resource có thể là một collection hoặc một cá thể đơn lẻ. Ví dụ, classes của chúng ta là resource dạng collection, còn một class cụ thể là resource dạng cá thể. Mỗi loại resource có một URI (Uniform Resource Identifier) tương ứng; nếu cần lấy resource này, chỉ cần truy cập URI đó, chẳng hạn lấy một class cụ thể: `/classes/12`. Ngoài ra, resource cũng có thể chứa sub-resource, chẳng hạn `/classes/{classId}/teachers`: liệt kê thông tin tất cả teacher của một class được chỉ định.
- **Representational**: “Resource” là một thực thể thông tin và có thể có nhiều hình thức biểu hiện bên ngoài. Các hình thức trình bày cụ thể của “resource”, chẳng hạn `json`, `xml`, `image`, `txt`, được gọi là “representation” của nó.
- **State Transfer**: Chắc hẳn lần đầu nhìn thấy cụm từ này bạn sẽ rất bối rối? Nói đơn giản, client chuyển từ application state này sang application state khác thông qua representation của resource cùng các thông tin điều khiển như link trong đó. Việc thực hiện create, delete, update, query bằng các phương thức HTTP cũng có thể làm thay đổi trạng thái resource ở server. Lưu ý: HTTP là stateless protocol, server không cần lưu session state của client giữa hai request.

Từ các giải thích trên, chúng ta có thể tổng kết RESTful architecture như sau:

1. Mỗi URI đại diện cho một loại resource;
2. Client và server truyền cho nhau một representation nào đó của resource, chẳng hạn `json`, `xml`, `image`, `txt`;
3. Client thao tác trên resource ở server thông qua HTTP verb cụ thể, từ đó thực hiện “chuyển trạng thái biểu diễn”.

## Quy tắc RESTful API

![](https://oss.javaguide.cn/github/javaguide/system-design/basis/20210507154007779.png)

### Action

- `GET`: request server lấy resource cụ thể. Ví dụ: `GET /classes` (lấy tất cả class)
- `POST`: yêu cầu target resource xử lý nội dung request theo semantics của chính nó, thường dùng để tạo resource. Ví dụ: `POST /classes` (tạo class)
- `PUT`: tạo hoặc thay thế state hiện tại của target resource (client thường cung cấp resource đầy đủ sau khi cập nhật). Ví dụ: `PUT /classes/12` (cập nhật class có số hiệu 12)
- `DELETE`: xoá liên kết giữa target URI và chức năng của resource hiện tại. Ví dụ: `DELETE /classes/12` (xoá class có số hiệu 12)
- `PATCH`: cập nhật resource trên server (client cung cấp các thuộc tính thay đổi, có thể xem là partial update), ít được dùng hơn nên không nêu ví dụ ở đây.

Trong đó, `GET` là safe và idempotent; `PUT` và `DELETE` là idempotent; `PATCH` mặc định không đảm bảo idempotent.

### Path (đặt tên interface)

Path còn được gọi là “endpoint”, biểu thị URL cụ thể của API. Trong phát triển thực tế, các quy ước phổ biến như sau:

1. **URL của HTTP API dạng resource thường dùng danh từ, danh từ thường ở dạng số nhiều.** Đây là quy ước đặt tên URI phổ biến, không phải ràng buộc bắt buộc của REST. Nếu API khó trừu tượng hoá thành resource, chẳng hạn các thao tác tính toán, dịch, thì cũng có thể dùng động từ. Ví dụ: `GET /calculate?param1=11&param2=33`.
2. **Không dùng chữ hoa, nên dùng dấu gạch ngang `-`, không dùng dấu gạch dưới `_`.** Ví dụ, mã mời nên viết là `invitation-code` thay vì ~~invitation_code~~.
3. **Tận dụng API versioning.** Khi API có thay đổi lớn và không tương thích với version trước, chúng ta có thể version hoá thông qua URL, chẳng hạn `http://api.example.com/v1`, `http://apiv1.example.com`. Version không nhất thiết phải là số; chỉ là số được dùng nhiều nhất. Ngày tháng hoặc mùa cũng có thể làm version identifier, miễn là team dự án thống nhất.
4. **Interface nên ưu tiên dùng danh từ, tránh dùng động từ.** RESTful API thao tác (HTTP Method) trên resource (danh từ), không phải action (động từ).

Talk is cheap! Hãy lấy một ví dụ thực tế để minh hoạ. Giả sử có một API cung cấp thông tin về class, bao gồm cả thông tin student và teacher trong class, path nên được thiết kế như sau.

```plain
GET    /classes: liệt kê tất cả class
POST   /classes: tạo một class mới
GET    /classes/{classId}: lấy thông tin một class được chỉ định
PUT    /classes/{classId}: cập nhật thông tin một class được chỉ định (thường thiên về cập nhật toàn bộ)
PATCH  /classes/{classId}: cập nhật thông tin một class được chỉ định (thường thiên về cập nhật một phần)
DELETE /classes/{classId}: xoá một class
GET    /classes/{classId}/teachers: liệt kê thông tin tất cả teacher của một class được chỉ định
GET    /classes/{classId}/students: liệt kê thông tin tất cả student của một class được chỉ định
DELETE /classes/{classId}/teachers/{ID}: xoá thông tin teacher được chỉ định thuộc một class được chỉ định
```

Phản ví dụ:

```plain
/getAllclasses
/createNewclass
/deleteAllActiveclasses
```

Hãy làm rõ cấu trúc phân cấp của resource. Ví dụ, nếu phạm vi nghiệp vụ là trường học, school sẽ là resource cấp một: `/schools`; teacher: `/schools/{schoolId}/teachers`, student: `/schools/{schoolId}/students` là resource cấp hai.

### Filtering

Nếu cần thêm điều kiện cụ thể khi query, nên dùng dạng URL parameter. Ví dụ, cần query các class có state là active và name là guidegege:

```plain
GET    /classes?state=active&name=guidegege
```

Ví dụ, để thực hiện query phân trang:

```plain
GET    /classes?page=1&size=10 // chỉ định trang 1, mỗi trang 10 dữ liệu
```

### Status Codes

**Phạm vi status code:**

| 2xx: thành công | 3xx: redirect              | 4xx: lỗi client                   | 5xx: lỗi server     |
| --------------- | -------------------------- | --------------------------------- | ------------------- |
| 200 thành công  | 301 redirect vĩnh viễn     | 400 request không hợp lệ          | 500 lỗi server      |
| 201 đã tạo      | 304 resource chưa thay đổi | 401 chưa được cấp quyền           | 502 lỗi gateway     |
|                 |                            | 403 bị từ chối truy cập           | 504 gateway timeout |
|                 |                            | 404 không tìm thấy                |                     |
|                 |                            | 405 method của request không đúng |                     |

## HATEOAS trong REST

> **Trong định nghĩa ban đầu của Fielding về REST, HATEOAS là một phần của ràng buộc uniform interface. Tuy nhiên, trong thực tế, nhiều HTTP/JSON API được gọi là REST API lại không triển khai nó.**

Trên đây là những kiến thức cơ bản nhất về RESTful API và cũng là những điều dễ áp dụng nhất trong quá trình phát triển hằng ngày. HATEOAS yêu cầu dùng Hypermedia để điều khiển application state, tức là cung cấp các thông tin điều khiển như link trong kết quả trả về, để người dùng biết bước tiếp theo cần làm gì mà không cần tra tài liệu.

Ví dụ, khi người dùng gửi request đến root directory của `api.example.com`, sẽ nhận được kết quả như sau:

```javascript
{"link": {
  "rel":   "collection",
  "href":  "https://api.example.com/classes",
  "title": "List of classes",
  "type":  "application/vnd.yourformat+json"
}}
```

Đoạn code trên cho biết trong document có thuộc tính `link`; người dùng đọc thuộc tính này là biết API nào cần gọi tiếp theo. `rel` biểu thị mối quan hệ giữa target resource và context hiện tại, `href` biểu thị path của target resource, `title` biểu thị tiêu đề của link, còn `type` là gợi ý về media type của representation của target resource. Thiết kế `Hypermedia API` như vậy được gọi là [HATEOAS](https://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven).

Trong Spring có một thư viện API tên là HATEOAS, qua đó chúng ta có thể dễ dàng tạo API phù hợp với thiết kế HATEOAS hơn. Bài viết liên quan:

- [Sử dụng HATEOAS trong Spring Boot](https://blog.aisensiy.me/2017/06/04/spring-boot-and-hateoas/)
- [Building REST services with Spring](https://spring.io/guides/tutorials/rest/) (trang web chính thức của Spring)
- [An Intro to Spring HATEOAS](https://www.baeldung.com/spring-hateoas-tutorial)
- [spring-hateoas-examples](https://github.com/spring-projects/spring-hateoas-examples/tree/master/hypermedia)
- [Spring HATEOAS](https://spring.io/projects/spring-hateoas#learn) (trang web chính thức của Spring)

## Tài liệu tham khảo

- [Luận văn của Fielding: Representational State Transfer](https://ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)

- [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)

- [RFC 5789: PATCH Method for HTTP](https://www.rfc-editor.org/rfc/rfc5789.html)

- [RFC 8288: Web Linking](https://www.rfc-editor.org/rfc/rfc8288.html)

- <https://RESTfulapi.net/>

- <https://www.ruanyifeng.com/blog/2014/05/restful_api.html>

- <https://juejin.im/entry/59e460c951882542f578f2f0>

- <https://phauer.com/2016/testing-RESTful-services-java-best-practices/>

- <https://www.seobility.net/en/wiki/REST_API>

- <https://dev.to/duomly/rest-api-vs-graphql-comparison-3j6g>

<!-- @include: @article-footer.snippet.md -->
