---
title: Kiến thức cơ bản về J2EE
description: "Giải thích chi tiết kiến thức cơ bản về J2EE, bao quát vòng đời Servlet, chuyển tiếp và chuyển hướng request, cơ chế Session và Cookie cùng các khái niệm cốt lõi của Java Web."
category: System Design
head:
  - - meta
    - name: keywords
      content: J2EE,Java Web,Servlet,JSP,HTTP request response,Servlet lifecycle,Session,Cookie
---

## Tổng hợp Servlet

> Lưu ý: J2EE là tên lịch sử, sau đó được đổi tên thành Java EE, tên specification hiện tại là Jakarta EE. Bài viết chủ yếu giới thiệu mô hình lập trình Servlet/JSP truyền thống.

Trong chương trình Java Web, **Servlet** chủ yếu chịu trách nhiệm tiếp nhận request của người dùng `HttpServletRequest`, xử lý trong các method như `doGet()`, `doPost()` và trả response thông qua `HttpServletResponse`. Servlet có thể thiết lập tham số khởi tạo để sử dụng bên trong Servlet. Trong môi trường không phân tán, Servlet container thường sử dụng một instance cho **mỗi khai báo Servlet**; nếu cùng một class Servlet có nhiều khai báo thì vẫn có thể có nhiều instance. Container gọi `init()` khi khởi tạo và gọi `destroy()` khi instance ngừng phục vụ. Servlet có thể được khai báo thông qua `@WebServlet`, `web.xml` hoặc API lập trình; một khai báo Servlet cũng có thể ánh xạ nhiều URL. Container có thể cho nhiều thread đồng thời thực thi method `service()` của cùng một instance, vì vậy không nên lưu trạng thái có thể thay đổi của từng request trong các field của instance.

## Nêu sự khác biệt giữa Servlet và CGI?

### Nhược điểm của CGI

1. Phải khởi động một system process để vận hành chương trình CGI cho mỗi request. Nếu request thường xuyên, việc này sẽ gây overhead lớn.

2. Phải load và chạy một chương trình CGI cho mỗi request, việc này sẽ gây overhead lớn.

3. Phải lặp lại việc viết code xử lý network protocol và encoding; những công việc này đều rất tốn thời gian.

### Ưu điểm của Servlet

1. Chỉ cần khởi động một system process và load một JVM, giảm đáng kể overhead của hệ thống.

2. Khi nhiều request cần xử lý giống nhau, chỉ cần load một class, nhờ đó cũng giảm đáng kể overhead.

3. Tất cả class được load động có thể dùng chung phần xử lý network protocol và request decoding, giảm đáng kể khối lượng công việc.

4. Servlet có thể tương tác trực tiếp với Web server, còn chương trình CGI thông thường thì không. Servlet cũng có thể chia sẻ dữ liệu giữa các chương trình, nhờ đó dễ triển khai các chức năng như connection pool.

Bổ sung: Công ty Sun Microsystems phát hành công nghệ Servlet vào năm 1996 để cạnh tranh với CGI. Servlet là một chương trình Java đặc biệt; một Web application dựa trên Java thường chứa một hoặc nhiều class Servlet. Servlet không thể tự tạo và thực thi mà chạy trong Servlet container. Container truyền request của người dùng cho chương trình Servlet và trả response của Servlet về cho người dùng. Thông thường một Servlet sẽ liên kết với một hoặc nhiều trang JSP. Trước đây CGI thường bị phê phán vì vấn đề overhead hiệu năng, nhưng Fast CGI đã giải quyết vấn đề hiệu suất của CGI từ lâu. Vì vậy khi phỏng vấn, không nên tùy tiện phê phán CGI; trên thực tế nhiều website quen thuộc vẫn sử dụng công nghệ CGI.

Tham khảo: 《Vua trở lại của phát triển tích hợp javaweb》P7

## Interface Servlet có những method nào và tìm hiểu vòng đời Servlet

Interface Servlet định nghĩa 5 method, trong đó **ba method đầu liên quan đến vòng đời Servlet**:

- `void init(ServletConfig config) throws ServletException`
- `void service(ServletRequest req, ServletResponse resp) throws ServletException, java.io.IOException`
- `void destroy()`
- `java.lang.String getServletInfo()`
- `ServletConfig getServletConfig()`

**Vòng đời:** **Vòng đời Servlet bắt đầu sau khi Web container load và khởi tạo instance Servlet**, container chạy **method init()** để khởi tạo Servlet; khi request đến, container gọi **method service()** của Servlet, method service() sẽ gọi các method tương ứng với request như **doGet hoặc doPost** khi cần. Khi server tắt hoặc project bị unload, server sẽ hủy instance Servlet và gọi **method destroy()**. **Method init và destroy chỉ chạy một lần, method service chạy mỗi khi client request Servlet**. Đôi khi Servlet cần sử dụng một số resource phải khởi tạo và hủy, vì vậy có thể đặt code khởi tạo resource trong method init và code hủy resource trong method destroy. Như vậy không cần khởi tạo và hủy resource trong mỗi lần xử lý request của client.

Tham khảo: 《Vua trở lại của phát triển tích hợp javaweb》P81

## Sự khác biệt giữa GET và POST

Câu hỏi này từng được thảo luận khá sôi nổi trên Zhihu, địa chỉ: <https://www.zhihu.com/question/28586791> .

![](https://static001.geekbang.org/infoq/04/0454a5fff1437c32754f1dfcc3881148.png)

GET và POST là hai method request thường dùng trong HTTP protocol, có đặc điểm và cách sử dụng khác nhau tùy bối cảnh và mục đích. Nhìn chung, có thể phân biệt chúng từ các khía cạnh sau:

- Khác biệt về ngữ nghĩa: GET dùng để lấy biểu diễn của target resource, là method an toàn và idempotent. POST dùng để yêu cầu target resource xử lý nội dung request theo ngữ nghĩa của chính resource đó, thường dùng để tạo resource, submit dữ liệu hoặc trigger operation, mặc định không có ngữ nghĩa idempotent.
- Khác biệt về format: Điều kiện truy vấn của GET request thường đặt trong query string của URL, còn POST thường truyền dữ liệu qua request content, có thể dùng các media type như `application/x-www-form-urlencoded`, `multipart/form-data`, `application/json`. HTTP specification không quy định một giới hạn cố định và dùng chung cho URL hoặc POST request content; giới hạn thực tế do browser, server, gateway và các component khác quyết định.
- Khác biệt về cache: GET response mặc định có thể được cache nhưng vẫn chịu ràng buộc của các response header như `Cache-Control`. POST response cũng không tuyệt đối không thể cache, chỉ là server cần chủ động cung cấp thông tin cho phép cache; trong thực tế cách dùng này ít phổ biến.
- Khác biệt về tính an toàn: Cả GET request và POST request đều không tuyệt đối an toàn, vì bản thân HTTP protocol truyền dữ liệu dạng plaintext; URL, header và body đều có thể bị đánh cắp hoặc sửa đổi. Để bảo đảm an toàn, phải dùng HTTPS protocol để mã hóa dữ liệu truyền đi. Tuy nhiên, trong một số trường hợp, GET request dễ làm lộ dữ liệu nhạy cảm hơn POST request, vì parameter của GET request xuất hiện trong URL, còn URL có thể được ghi lại trong browser history, server log, proxy log và các nơi khác. Vì vậy, thông thường nên dùng POST + body để truyền dữ liệu riêng tư.

Điểm quan trọng là chọn GET hoặc POST theo ngữ nghĩa chuẩn của HTTP method. Thiết kế mọi request thành POST tuy khả thi về mặt kỹ thuật nhưng sẽ làm mất các ưu điểm của GET về cache, retry an toàn và ngữ nghĩa tại các middleware; không nên chỉ lấy lý do “team đã thống nhất” làm căn cứ thiết kế.

## Khi nào gọi doGet() và doPost()

Khi thuộc tính `method` của tag Form là `get` thì gọi `doGet()`, khi là `post` thì gọi `doPost()`.

## Sự khác biệt giữa chuyển tiếp (Forward) và chuyển hướng (Redirect)

**Chuyển tiếp là hành vi của server, chuyển hướng là hành vi của client.**

**Chuyển tiếp (Forward)**

Được thực hiện thông qua method `forward(HttpServletRequest request,HttpServletResponse response)` của object RequestDispatcher. Có thể lấy RequestDispatcher thông qua method `getRequestDispatcher()` của HttpServletRequest. Ví dụ code dưới đây chuyển đến trang login_success.jsp.

```java
     request.getRequestDispatcher("login_success.jsp").forward(request, response);
```

**Chuyển hướng (Redirect)** do server trả về status code 3xx và response header `Location`, sau đó client gửi request đến địa chỉ mới. Trong Servlet thường sử dụng `HttpServletResponse#sendRedirect()`, cũng có thể tự thiết lập status code và `Location`. Trong các implementation lịch sử, 301, 302, 303 có thể đổi request tiếp theo sau POST thành GET; khi cần giữ nguyên method của request ban đầu, nên dùng 307 hoặc 308 tùy trường hợp.

1. **Xét theo nội dung hiển thị trên thanh địa chỉ**

   forward là server request resource, server trực tiếp truy cập URL của resource đích, đọc response content của URL đó rồi gửi content này cho browser. Browser hoàn toàn không biết content server gửi đến từ đâu, vì vậy thanh địa chỉ vẫn là địa chỉ ban đầu.
   redirect là server dựa trên logic để gửi một status code, yêu cầu browser request lại địa chỉ đó. Vì vậy thanh địa chỉ hiển thị URL mới.

2. **Xét theo việc chia sẻ dữ liệu**

   forward: trang được chuyển tiếp và trang đích có thể chia sẻ dữ liệu trong request.
   redirect: không thể chia sẻ dữ liệu.

3. **Xét theo nơi sử dụng**

   forward: thường dùng khi user login, chuyển tiếp user đến module tương ứng theo role.
   redirect: thường dùng khi user logout để quay về trang chính hoặc chuyển đến website khác.

4. Xét về hiệu suất

   forward: cao.
   redirect: thấp.

## Tự động refresh (Refresh)

Một số browser hỗ trợ response header không chuẩn `Refresh`, có thể dùng để refresh hoặc chuyển hướng sau một khoảng trễ. Trong Servlet có thể thiết lập thông qua `HttpServletResponse`:

```java
response.setHeader("Refresh", "5; url=http://localhost:8080/servlet/example.htm");
```

Đơn vị của 5 là giây. Vì `Refresh` không phải response header chuẩn của HTTP, không nên dùng nó thay cho chuyển hướng 3xx chuẩn; việc refresh trang cũng có thể được thực hiện ở frontend tùy nhu cầu.

## Servlet và thread safety

**Servlet không thread-safe, đọc ghi đồng thời bởi nhiều thread sẽ gây ra vấn đề dữ liệu không đồng bộ.** Cách giải quyết là cố gắng không định nghĩa thuộc tính name mà định nghĩa riêng biến name bên trong method `doGet()` và `doPost()`. Dù có thể dùng block `synchronized(name){}` để giải quyết vấn đề nhưng sẽ khiến thread phải chờ, đây không phải cách làm hợp lý nhất.
Lưu ý: Đọc ghi đồng thời các thuộc tính của class Servlet bởi nhiều thread sẽ khiến dữ liệu không đồng bộ. Tuy nhiên, nếu chỉ đọc thuộc tính đồng thời mà không ghi thì sẽ không xảy ra vấn đề dữ liệu không đồng bộ. Vì vậy, các thuộc tính chỉ đọc trong Servlet nên được định nghĩa với kiểu `final`.

Tham khảo: 《Vua trở lại của phát triển tích hợp javaweb》P92

## JSP và Servlet có quan hệ gì?

Thực ra vấn đề này đã được trình bày ở trên. Servlet là một chương trình Java đặc biệt, chạy trong JVM của server và có thể dựa vào sự hỗ trợ của server để cung cấp nội dung hiển thị cho browser. Về bản chất, JSP là một dạng đơn giản của Servlet. JSP được server xử lý thành một chương trình Java tương tự Servlet, giúp đơn giản hóa việc tạo nội dung trang. Điểm khác biệt chủ yếu giữa Servlet và JSP là logic ứng dụng của Servlet nằm trong file Java và được tách hoàn toàn khỏi HTML ở presentation layer. Còn với JSP, Java và HTML có thể kết hợp trong một file có extension `.jsp`. Có người nói Servlet là viết HTML trong Java, còn JSP là viết Java code trong HTML; tất nhiên cách nói này khá phiến diện và chưa chính xác. JSP tập trung hơn vào view, Servlet tập trung hơn vào control logic. Trong MVC architecture pattern, JSP phù hợp làm view, còn Servlet phù hợp làm controller.

## Nguyên lý hoạt động của JSP

JSP page được JSP container chuyển thành class implementation của Servlet rồi compile. Với HTTP, class implementation được tạo ra cần implement interface `HttpJspPage`, còn `HttpJspPage` kế thừa `JspPage`. Trong cấu hình compile on demand phổ biến, việc chuyển đổi và compile xảy ra ở request đầu tiên; container cũng có thể precompile JSP, vì vậy đây không phải bước cố định luôn xảy ra ở lần request đầu tiên. Java source và class file được tạo thường lưu trong working directory của container. Dưới đây là ví dụ về trường hợp compile on demand.
Trong project JspLoginDemo có một file Jsp tên login.jsp. Sau khi deploy project lần đầu lên server và truy cập file Jsp này, ta nhận thấy trong directory xuất hiện thêm hai file như hình dưới.
File `.class` chính là Servlet tương ứng với JSP. Sau khi compile xong, class file được chạy để response request của client. Những lần sau khi client truy cập login.jsp, Tomcat không compile lại JSP file mà gọi trực tiếp class file để response request của client.

![Nguyên lý hoạt động của JSP](https://oss.javaguide.cn/github/javaguide/1.jpeg)

Trong chế độ compile on demand, request đầu tiên cần hoàn tất việc chuyển đổi và compile nên thường chậm hơn các request sau. Nếu xóa class file do container tạo, container sẽ compile lại JSP khi cần page đó.

Khi phát triển Web program, thường xuyên cần sửa JSP. Tomcat có thể tự động phát hiện thay đổi của JSP program. Nếu phát hiện JSP source code đã thay đổi, Tomcat sẽ compile lại JSP khi client request JSP ở lần tiếp theo mà không cần restart Tomcat. Tính năng tự động phát hiện này mặc định được bật. Việc kiểm tra thay đổi tiêu tốn một lượng nhỏ thời gian; khi deploy Web application, có thể tắt tính năng này trong `web.xml`.

Tham khảo: 《Vua trở lại của phát triển tích hợp javaweb》P97

## JSP có những built-in object nào và tác dụng của chúng là gì?

[JSP built-in object - CSDN Blog](http://blog.csdn.net/qq_34337272/article/details/64310849)

JSP có 9 built-in object:

- request: đóng gói request của client, trong đó chứa parameter từ GET hoặc POST request;
- response: đóng gói response của server cho client;
- pageContext: thông qua object này có thể lấy các object khác;
- session: object đóng gói user session;
- application: object đóng gói môi trường chạy của server;
- out: object output stream dùng để xuất server response;
- config: object cấu hình của Web application;
- page: bản thân JSP page (tương đương `this` trong chương trình Java);
- exception: object đóng gói exception được page ném ra.

## Các method chính của Request object là gì?

- `setAttribute(String name,Object)`: thiết lập giá trị parameter của request có tên là name.
- `getAttribute(String name)`: trả về giá trị attribute được chỉ định bởi name.
- `getAttributeNames()`: trả về tập hợp tên của tất cả attribute trong request, kết quả là một instance của enumeration.
- `getCookies()`: trả về tất cả Cookie object của client, kết quả là một mảng Cookie.
- `getCharacterEncoding()`: trả về character encoding được request sử dụng.
- `getContentLength()`: trả về số byte của request body; trả về -1 khi không biết độ dài hoặc độ dài vượt phạm vi `int`; request lớn có thể dùng `getContentLengthLong()`.
- `getHeader(String name)`: lấy thông tin header do HTTP protocol định nghĩa.
- `getHeaders(String name)`: trả về tất cả giá trị của request Header có tên được chỉ định, kết quả là một instance của enumeration.
- `getHeaderNames()`: trả về tên của tất cả request Header, kết quả là một instance của enumeration.
- `getInputStream()`: trả về input stream của request, dùng để lấy dữ liệu trong request.
- `getMethod()`: lấy method mà client dùng để truyền dữ liệu đến server.
- `getParameter(String name)`: lấy giá trị parameter có tên được chỉ định bởi name mà client truyền đến server.
- `getParameterNames()`: lấy tất cả tên parameter mà client truyền đến server, kết quả là một instance của enumeration.
- `getParameterValues(String name)`: lấy tất cả giá trị của parameter có tên được chỉ định bởi name.
- `getProtocol()`: lấy tên protocol mà client dựa vào để truyền dữ liệu đến server.
- `getQueryString()`: lấy query string.
- `getRequestURI()`: lấy phần URI path trong request line, từ protocol name đến giữa query string.
- `getRemoteAddr()`: lấy IP address của client.
- `getRemoteHost()`: lấy tên của client.
- `getSession()`: trả về Session liên kết với request; nếu không tồn tại thì tạo mới.
- `getSession(boolean create)`: trả về Session liên kết với request; trả về `null` khi `create` là `false` và Session không tồn tại.
- `getServerName()`: lấy tên server.
- `getServletPath()`: lấy path của script file mà client request.
- `getServerPort()`: lấy port của server.
- `removeAttribute(String name)`: xóa một attribute trong request.

## Sự khác biệt giữa request.getAttribute() và request.getParameter() là gì?

`getParameter()` đọc parameter do client submit cùng request, chẳng hạn query string của URL hoặc form field đã được parse. Method này trả về `String`; khi có nhiều value cùng tên, có thể dùng `getParameterValues()`.

`getAttribute()` đọc object được server code bind vào request hiện tại thông qua `setAttribute()`, kiểu trả về là `Object`. Trong quá trình server request forwarding như `forward`, các component vẫn xử lý cùng một request object nên có thể chia sẻ request attribute; client redirect sẽ tạo request mới và không giữ attribute của request cũ. Quá trình này không phải là container copy một vùng memory giữa các page.

## Sự khác biệt về hành vi của include directive và include là gì?

**include directive:** JSP có thể dùng include directive để include file khác. File được include có thể là JSP file, HTML file hoặc text file. File được include giống như một phần của JSP file đó, được compile và execute đồng thời. Cú pháp như sau:
<%@ include file="địa chỉ URL tương đối của file" %>

**include action:** `<jsp:include>` action element dùng để include file static và dynamic. Action này chèn file được chỉ định vào page đang được tạo. Cú pháp như sau:
<jsp:include page="địa chỉ URL tương đối" flush="true" />

## 9 built-in object, 7 action và 3 directive của JSP

[Tổng hợp 9 built-in object, 7 action và 3 directive của JSP](http://blog.csdn.net/qq_34337272/article/details/64310849)

## Giải thích bốn scope trong JSP

Bốn scope trong JSP gồm page, request, session và application, cụ thể:

- **page** đại diện cho object và attribute liên quan đến một page.
- **request** đại diện cho object và attribute liên quan đến một request do Web client gửi. Một request có thể đi qua nhiều page và liên quan đến nhiều Web component; dữ liệu tạm thời cần hiển thị trên page có thể đặt trong scope này.
- **session** đại diện cho object và attribute liên quan đến một session được thiết lập giữa một user và server. Dữ liệu liên quan đến một user nên đặt trong session của chính user đó.
- **application** đại diện cho object và attribute liên quan đến toàn bộ Web application. Về bản chất, đây là một global scope trải rộng trên toàn bộ Web application, bao gồm nhiều page, request và session.

## Nên xử lý concurrent request của Servlet như thế nào?

Trong lịch sử, JSP từng cung cấp `<%@ page isThreadSafe="false" %>`, Servlet cũng cung cấp marker interface `SingleThreadModel`. `SingleThreadModel` đã deprecated từ Servlet 2.4 và bị xóa trong Jakarta Servlet 6.0; nó cũng không bảo đảm thread safety của Session và static state, vì vậy không nên dùng làm giải pháp hiện tại.

Cách làm đúng là cố gắng giữ Servlet stateless, đặt dữ liệu có thể thay đổi của từng request trong local variable của method, không lưu trong field của Servlet instance. Khi bắt buộc phải chia sẻ state, cần dùng cơ chế concurrency control hoặc thread-safe data structure phù hợp và cố gắng thu hẹp critical section.

## Có những kỹ thuật nào để thực hiện session tracking?

1. **Dùng Cookie**

   Gửi Cookie đến client.

   ```java
   Cookie c =new Cookie("name","value"); // Tạo Cookie
   c.setMaxAge(60*60*24); // Thiết lập thời hạn tối đa, ở đây là một ngày
   response.addCookie(c); // Đưa Cookie vào HTTP response
   ```

   Đọc Cookie từ client.

   ```java
   String name ="name";
   Cookie[]cookies =request.getCookies();
   if(cookies !=null){
      for(int i= 0;i<cookies.length;i++){
       Cookie cookie =cookies[i];
       if(name.equals(cookie.getName())) {
         // Nội dung xử lý ở đây.
         // Có thể lấy value.
         cookie.getValue();
       }

      }
    }

   ```

   **Ưu điểm:** Dữ liệu có thể được lưu lâu dài, không cần tài nguyên server, đơn giản, dựa trên Key-Value dạng text.

   **Nhược điểm:** Kích thước bị giới hạn, user có thể vô hiệu hóa chức năng Cookie; vì được lưu cục bộ nên có một số rủi ro bảo mật.

2. Viết lại URL

   Thêm thông tin session của user làm parameter request trong URL, hoặc thêm Session ID duy nhất vào cuối URL để nhận diện một session.

   **Ưu điểm:** Vẫn có thể sử dụng khi Cookie bị vô hiệu hóa.

   **Nhược điểm:** Phải encode URL của website, tất cả page phải được tạo động, không thể truy cập bằng URL được ghi lại từ trước.

3. Hidden form field

   ```html
   <input type="hidden" name="session" value="..." />
   ```

   **Ưu điểm:** Có thể sử dụng khi Cookie bị vô hiệu hóa.

   **Nhược điểm:** Tất cả page phải là kết quả sau khi submit form.

4. HttpSession

   HttpSession không chắc chắn tự động được tạo chỉ vì user truy cập website lần đầu. Khi code gọi `HttpServletRequest#getSession()`, hoặc một chức năng của framework/JSP yêu cầu Session cho request hiện tại, container mới tạo Session nếu nó chưa tồn tại. Có thể dùng `setAttribute()` và `getAttribute()` để lưu và đọc session attribute. Dữ liệu HttpSession do server quản lý, có thể được lưu cụ thể trong memory, distributed storage hoặc persistent medium, vì vậy không nên đưa object quá lớn vào đó. Khi deploy phân tán hoặc cần persist session, attribute thường cũng cần serializable; yêu cầu cụ thể phụ thuộc vào container và session storage solution.

## Sự khác biệt giữa Cookie và Session

Cookie và Session đều dùng để tracking identity của browser user, nhưng use case của chúng không hoàn toàn giống nhau.

**Cookie thường dùng để lưu thông tin user**, chẳng hạn: ① lưu thông tin user đã login trong Cookie để lần sau khi truy cập website, page có thể tự động điền một số thông tin login cơ bản; ② website thường có chức năng duy trì login, nghĩa là lần sau truy cập website không cần login lại, vì khi user login ta có thể lưu một Token trong Cookie, lần sau chỉ cần dựa vào giá trị Token để tìm user (vì lý do an toàn, khi login lại thường cần ghi đè Token); ③ sau khi login website một lần, truy cập các page khác của website không cần login lại. **Tác dụng chính của Session là ghi lại state của user thông qua server.** Use case điển hình là shopping cart. Khi muốn thêm sản phẩm vào shopping cart, hệ thống không biết thao tác được thực hiện bởi user nào vì HTTP protocol là stateless. Sau khi server tạo Session riêng cho user cụ thể, nó có thể nhận diện và tracking user đó.

Dữ liệu Cookie được lưu ở client (phía browser), còn dữ liệu Session được lưu ở server.

Cookie được lưu ở client, Session state thường do server quản lý, nhưng điều này không có nghĩa chỉ cần dùng Session thì mặc nhiên an toàn hơn. Session phổ biến vẫn dựa vào Cookie để truyền session identifier; nếu identifier bị đánh cắp, nó có thể bị replay. Session Cookie nên sử dụng identifier ngẫu nhiên có entropy cao và không mang ý nghĩa nghiệp vụ, truyền qua HTTPS, đồng thời thiết lập `Secure`, `HttpOnly`, `SameSite`, `Path` và `Domain` tùy trường hợp. Không nên ghi trực tiếp dữ liệu nghiệp vụ nhạy cảm như password, số thẻ ngân hàng vào Cookie; mã hóa cũng không thể thay thế việc bảo vệ integrity, expiration, revocation và authorization check ở server.

<!-- @include: @article-footer.snippet.md -->
