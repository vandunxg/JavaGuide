---
title: "Thực chiến thiết kế distributed ID: chiến lược tạo mã đơn hàng, coupon, thanh toán một mã và business ID"
category: Distributed
description: "Hướng dẫn thực chiến thiết kế distributed ID, kết hợp các tình huống mã đơn hàng, mã thanh toán, coupon và thanh toán một mã để giải thích nguyên tắc thiết kế, ngữ nghĩa nghiệp vụ, quy hoạch dung lượng, tính dễ đọc và chiến lược tạo ID duy nhất toàn cục."
tag:
  - Distributed ID
head:
  - - meta
    - name: keywords
      content: "Distributed ID,thiết kế Distributed ID,tạo mã đơn hàng,ID coupon,thanh toán một mã,business ID,ID duy nhất toàn cục,chiến lược tạo ID,thiết kế hệ thống phân tán"
---

::: tip

Tôi đọc được một bài của Baidu Geek thảo luận về thiết kế distributed ID qua các tình huống cụ thể và thấy khá hay. Vì vậy, tôi tổng hợp một phần nội dung bài viết vào đây. Bài gốc: [Nguyên lý kỹ thuật và thực chiến dự án của dịch vụ tạo distributed ID](https://mp.weixin.qq.com/s/bFDLb6U6EgI-DvCdLTq_QA) .

:::

Phần lớn dịch vụ tạo distributed ID trên Internet thường tập trung phân tích nguyên lý kỹ thuật, còn bài viết chọn dịch vụ tạo ID theo tình huống nghiệp vụ cụ thể thì khá hiếm.

Bài viết này kết hợp một số tình huống sử dụng để tìm hiểu sâu hơn các yêu cầu cụ thể đối với ID trong nghiệp vụ.

Nếu bạn chưa đọc nguyên lý và sự đánh đổi của UUID, segment mode, Snowflake, Leaf và Tinyid, nên đọc trước [Giải thích chi tiết các phương án tạo distributed ID](./distributed-id.md). Bài này thiên về thiết kế nghiệp vụ hơn: cùng là ID, mã đơn hàng quan tâm nhiều hơn đến khả năng tra cứu và tính an toàn, TraceId quan tâm nhiều hơn đến việc truyền qua call chain, còn short URL quan tâm nhiều hơn đến độ dài và tính dễ đọc.

## Tình huống một: Hệ thống đơn hàng

Mã QR thanh toán một mã khi mua hàng ở trung tâm thương mại, mã đơn hàng tạo ra khi đặt hàng, mã coupon được sử dụng và mã đổi hàng liên kết đều là những mã thường gặp khi mua sắm trực tuyến. Vậy tại sao có mã dài, có mã chỉ vài chữ số? Có mã nhìn là biết thông tin năm tháng ngày, có mã lại không thể hiện ý nghĩa gì? Dưới đây là phân tích chi tiết cách triển khai dịch vụ ID trong các tình huống khác nhau của hệ thống đơn hàng.

### 1. Thanh toán một mã

Thanh toán một mã thường chỉ việc một mã QR có thể được quét và thanh toán bằng Alipay hoặc WeChat.

Bản chất của mã QR là một chuỗi. Bản chất của mã tổng hợp là một địa chỉ liên kết. Người dùng quét trực tiếp một mã bằng Alipay hoặc WeChat để thanh toán, không phải lo dùng Alipay quét mã nhận tiền của WeChat hay dùng WeChat quét mã nhận tiền của Alipay. Điều này rút ngắn đáng kể thời gian quét mã thanh toán.

Nguyên lý triển khai là sau khi khách hàng dùng APP quét mã, backend của website sẽ xác định môi trường quét mã của khách hàng (WeChat, Alipay, QQ Wallet, JD Pay, UnionPay QuickPass, v.v.).

Nguyên lý xác định môi trường quét mã là dựa vào HTTP header của trình duyệt mở liên kết. Khi bất kỳ trình duyệt nào mở liên kết http, header của request đều có thông tin User-Agent (UA, user agent).

UA là một chuỗi header đặc biệt. Server có thể lần lượt nhận diện hệ điều hành và phiên bản, loại CPU, trình duyệt và phiên bản, rendering engine của trình duyệt, ngôn ngữ trình duyệt, plugin trình duyệt cùng nhiều thông tin khác mà khách hàng đang sử dụng.

Tên của sản phẩm thanh toán tương ứng ở mỗi channel khác nhau, cần xem kỹ phần giới thiệu API của từng sản phẩm thanh toán.

1. WeChat Pay: thanh toán JSAPI
2. Alipay: thanh toán trên mobile website
3. QQ Wallet: thanh toán official account

Bản chất của chúng đều là triển khai thanh toán HTML5 trong trình duyệt tích hợp của APP.

![Ví dụ thanh toán thành viên Wenku](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/distributed-id-design-pay-one-card.png)

Đội ngũ phát triển của Wenku đã tối ưu và lặp lại cách làm này. Mã QR thanh toán một mã được tạo động, liên kết trước với thông tin sản phẩm và giá do người dùng chọn, đồng thời cập nhật động theo sản phẩm đã chọn. Cách này không chỉ hỗ trợ khởi chạy thanh toán trên nhiều platform bằng một mã, mà còn cho phép hoàn tất thanh toán đơn hàng mà người dùng không cần chọn sản phẩm hay nhập số tiền, giúp quy trình mượt hơn. Chỉ sau khi người dùng thực sự quét mã, server mới lấy UID của người dùng từ frontend, kết hợp với thông tin sản phẩm được liên kết trong mã QR để thực sự tạo đơn hàng, gửi thông tin thanh toán đến bên thứ ba (qq, WeChat, Alipay). Bên thứ ba tạo đơn thanh toán rồi đẩy đến thiết bị người dùng để khởi chạy thanh toán.

Khác với thanh toán một mã cố định, ứng dụng của Wenku sử dụng mã QR động. Về bản chất, mã QR là một short URL, còn dịch vụ ID cung cấp tham số định danh duy nhất cho short URL. ID của short URL duy nhất được ánh xạ với thông tin đơn hàng của sản phẩm. Sự kết hợp sâu giữa kỹ thuật và nghiệp vụ đã rút ngắn quy trình thanh toán, cải thiện trải nghiệm thanh toán của người dùng.

### 2. Mã đơn hàng

Trong nghiệp vụ thực tế, mã đơn hàng tồn tại với tư cách mã định danh duy nhất của một đơn hàng, thường phục vụ các tình huống sau:

1. Khi đơn hàng của người dùng gặp vấn đề, cần tìm customer service để hỗ trợ;
2. Thực hiện thao tác trên đơn hàng, chẳng hạn thu tiền offline hoặc xác nhận đơn hàng;
3. Xử lý và theo dõi các quy trình đơn hàng nội bộ như đặt hàng, sửa đơn, hoàn tất đơn, hoàn đơn và hậu mãi.

Khi tìm kiếm thông tin liên quan đến đơn hàng, người ta thường dùng order ID làm mã định danh duy nhất. Điều này do tính duy nhất của quy tắc tạo mã đơn hàng quyết định. Về mặt kỹ thuật, ngoài các tính năng cần có của dịch vụ ID, thiết kế mã đơn hàng còn cần thể hiện một số đặc điểm:

**(1) An toàn thông tin**

Mã không được tiết lộ tình hình vận hành của công ty, chẳng hạn doanh số theo ngày, số thứ tự giao dịch của công ty, thông tin thương mại, số điện thoại hoặc số căn cước của người dùng. Mã cũng không được có quy luật tổng thể rõ ràng (có thể có quy luật cục bộ). Không được phép chỉ cần sửa một ký tự là tra cứu được thông tin của một đơn hàng khác.

Tương tự quy tắc tạo số báo danh thí sinh khi thi đại học, mã chắc chắn không được là các số liên tiếp. Nếu không, chỉ cần tra cứu lần lượt theo thứ tự là có thể tìm thấy điểm của thí sinh khác, điều này tuyệt đối không thể chấp nhận.

**(2) Dễ đọc một phần**

Số chữ số phải thuận tiện khi thao tác. Vì vậy, mã đơn hàng cần có độ dài vừa phải và có quy luật cục bộ. Điều này giúp customer service tra cứu khi đơn hàng bất thường hoặc cần trả hàng.

Mã đơn hàng quá dài hoặc khó đọc sẽ khiến customer service khó nhập và dễ sai hơn, làm ảnh hưởng đến trải nghiệm hậu mãi của người dùng. Vì vậy, trong tình huống nghiệp vụ thực tế, thiết kế mã đơn hàng thường mang theo một cách phù hợp một số thông tin được phép công khai và có ích cho tình huống sử dụng, chẳng hạn thời gian, thứ, loại, v.v. Cụ thể phụ thuộc vào tình huống sử dụng của mã tương ứng.

Ngoài ra, các thành phần tự tăng như thời gian và thứ trong thiết kế mã đơn hàng giúp giải quyết vấn đề trùng mã do nghiệp vụ tích lũy gây ra.

**(3) Hiệu suất tra cứu**

Mã đơn hàng phổ biến của các platform thương mại điện tử phần lớn chỉ gồm chữ số. Vừa có tính dễ đọc, kiểu `int` cũng có hiệu suất tra cứu cao hơn kiểu `varchar`, thân thiện hơn với nghiệp vụ online.

### 3. Coupon và mã đổi hàng

Coupon và mã đổi hàng là một trong những công cụ khuyến mãi phổ biến nhất để vận hành và quảng bá. Sử dụng chúng hợp lý có thể giúp người mua được lợi, đồng thời giúp người bán tăng doanh số sản phẩm. Các tình huống thường gặp:

1. Khi mua sản phẩm liên kết "Wenku VIP + thẻ năm QQ Music" tại Wenku, sau khi thanh toán thành công sẽ nhận được mã đổi thẻ năm QQ Music, dùng mã này trong app QQ Music để đổi thẻ thành viên âm nhạc một năm;
2. Coupon tiêu dùng do một số chính quyền địa phương phát trong thời kỳ dịch bệnh;
3. Đồ uống đóng chai thường có mã ưu đãi để nhập vào và đổi quà.

![Đổi quà bằng mã ưu đãi](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/distributed-id-design-coupon.png)

Về mặt kỹ thuật, một số tình huống phù hợp với việc tạo ID tức thời, chẳng hạn coupon nhận khi mua sắm trên platform thương mại điện tử: chỉ cần phân bổ thông tin coupon khi người dùng nhận. Một số tình huống kết hợp online và offline, chẳng hạn coupon thời kỳ dịch bệnh, mở thưởng dưới nắp chai, thẻ JD và thẻ siêu thị, lại cần tạo trước. Mã coupon tạo trước có các đặc điểm sau:

1. Tạo trước và cung cấp trước khi hoạt động chính thức bắt đầu để khởi động quảng bá;

2. Số lượng coupon lớn, tính theo vạn, thường từ 100.000 trở lên;

3. Không thể bẻ khóa hoặc làm giả mã coupon;

4. Hỗ trợ xác nhận sau khi sử dụng;

5. Coupon và mã đổi hàng là chiến lược phát tán rộng nên tỷ lệ sử dụng thấp, do đó không phù hợp lưu trữ bằng database **(tốn dung lượng nhưng dữ liệu thực sự hữu ích lại ít)**.

Về ý tưởng thiết kế, cần thiết kế một chiến lược tạo mã đổi hàng hiệu quả, hỗ trợ tạo trước, hỗ trợ kiểm tra, có nội dung ngắn gọn và bảo đảm mọi mã đổi hàng được tạo ra là duy nhất. Đây là một chiến lược encode/decode đặc biệt, dùng quy tắc encode/decode đã thỏa thuận để đáp ứng các yêu cầu trên.

Vì là một quy tắc encode/decode nên cần quy định encoding space, tức các ký tự tạo thành mã đổi hàng mà người dùng nhìn thấy. Encoding space gồm các ký tự a-z, A-Z và chữ số 0-9. Để tăng khả năng nhận diện mã đổi hàng, loại bỏ chữ O và I viết hoa. Các ký tự có thể dùng như sau, tổng cộng 60 ký tự:

abcdefghijklmnopqrstuvwxyzABCDEFGHJKLMNPQRSTUVWXZY0123456789

Đã nói ở trên, mã đổi hàng cần ngắn nhất có thể, nên khi thiết kế phải cân nhắc số ký tự. Giả sử giới hạn trên là 12 ký tự, encoding space có 60 ký tự, thì phạm vi có thể biểu diễn là 60^12=130606940160000000000000 (nghĩa là mã đổi hàng 12 ký tự có thể tạo ra số lượng khổng lồ, đủ để đội ngũ vận hành sử dụng thoải mái), khi chuyển sang hệ nhị phân là:

1001000100000000101110011001101101110011000000000000000000000 (61 bit)

**Phân tích thành phần mã đổi hàng**

Mã đổi hàng có thể được tạo trước mà không cần lưu riêng các thông tin này. Mỗi phương án ưu đãi có một nhóm mã đổi hàng độc lập (mỗi hoạt động vận hành do đội ngũ vận hành tổ chức có một nhóm mã khác nhau, không được dùng lẫn; ví dụ mã đổi hàng Double 11 không thể dùng cho hoạt động Double 12). Mỗi mã đổi hàng có số thứ tự riêng để tránh trùng lặp. Để bảo đảm mã đổi hàng hợp lệ, dữ liệu của mã cần được kiểm tra. Thành phần dữ liệu hiện tại của mã đổi hàng như sau:

ID phương án ưu đãi + số thứ tự mã đổi hàng i + mã kiểm tra

**Phương án encode**

1. Số thứ tự mã đổi hàng i đại diện cho việc mã hiện tại là mã thứ i trong hoạt động hiện tại. Phạm vi của số thứ tự mã đổi hàng quyết định số lượng mã đổi hàng mà hoạt động ưu đãi có thể phát hành. Hiện dùng 30 bit, biểu diễn được phạm vi: 1073741824 (1 tỷ mã coupon).
2. ID phương án ưu đãi đại diện cho ID của phương án hiện tại. Phạm vi của ID phương án ưu đãi quyết định số lần có thể tổ chức hoạt động ưu đãi. Hiện dùng 15 bit, biểu diễn được phạm vi: 32768 (xét đến tần suất hoạt động và giá trị ban đầu của ID là 10000, 15 bit là đủ; nếu mỗi ngày trong 365 ngày đều có hoạt động thì có thể dùng trong 54 năm).
3. Mã kiểm tra dùng để kiểm tra mã đổi hàng có hợp lệ hay không, chủ yếu nhằm kiểm tra nhanh tính đúng đắn của thông tin mã đổi hàng, đồng thời có thể dùng để lấp dữ liệu và tăng tính phân tán của dữ liệu. Mã kiểm tra dùng 13 bit, chia thành hai phần: 6 bit đầu và 7 bit sau.

Khi đi sâu hơn vào nghiệp vụ, còn có trường hợp phân biệt coupon dùng chung và coupon riêng. Hai loại có các đặc điểm sau, nên cách triển khai kỹ thuật cần được cân nhắc tùy tình huống.

1. Coupon dùng chung: nhiều người chơi có thể nhập để đổi, đồng thời có giới hạn tổng số lượng và thời hạn.
2. Coupon riêng: đội ngũ vận hành có thể thiết lập vật phẩm phần thưởng, thời hạn và số lượng của mã đổi hàng trong backend, sau đó backend tạo danh sách mã đổi hàng; sau khi đổi thì xác nhận mã.

## Tình huống hai: Tracing

### 1. Theo dõi log

Trong kiến trúc dịch vụ phân tán, một request Web đi vào từ gateway có thể gọi nhiều service để xử lý request và nhận kết quả cuối cùng. Trong quá trình này, giao tiếp giữa các service cũng là các request mạng độc lập. Bất kể service nào mà request đi qua gặp lỗi hoặc xử lý quá chậm đều ảnh hưởng đến frontend.

Để thuận tiện hơn trong việc tìm service gặp vấn đề ở khâu nào khi xử lý một request Web cần gọi nhiều service, giải pháp phổ biến hiện nay là đưa distributed tracing vào toàn hệ thống.

![Distributed tracing](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/distributed-id-design-tracing.png)

Trong distributed tracing có hai khái niệm quan trọng: trace và span. Trace là góc nhìn toàn bộ call chain của request trong hệ thống phân tán; span là góc nhìn bên trong các service khác nhau trong toàn bộ call chain. Các span kết hợp lại tạo thành góc nhìn của toàn bộ trace.

Trong toàn bộ call chain của request, request liên tục mang traceid truyền xuống các service downstream. Mỗi service bên trong cũng tạo spanid riêng để tạo góc nhìn call chain nội bộ, rồi truyền cùng traceid đến service downstream.

### 2. Quy tắc tạo TraceId

Trong tình huống này, ID được tạo không chỉ cần duy nhất mà còn cần hiệu suất tạo cao và throughput lớn. traceid cần có khả năng được instance server ở tầng tiếp nhận tự tạo. Nếu ID của mỗi trace đều phải request dịch vụ ID dùng chung, tài nguyên network bandwidth sẽ bị lãng phí hoàn toàn. Việc này còn chặn request của người dùng truyền xuống downstream, làm tăng thời gian phản hồi và tạo thêm rủi ro không cần thiết. Vì vậy, instance server nên tự tính traceid và spanid để tránh phụ thuộc vào service bên ngoài.

Quy tắc tạo: IP server + thời điểm tạo ID + sequence tự tăng + process ID hiện tại, ví dụ:

0ad1348f1403169275002100356696

8 ký tự đầu `0ad1348f` là IP của máy tạo TraceId. Đây là một số hexadecimal, cứ hai ký tự đại diện cho một phần trong IP. Chuyển từng cặp số này sang hệ thập phân sẽ thu được cách biểu diễn IP thường gặp là `10.209.52.143`. Bạn cũng có thể dựa vào quy luật này để tìm server đầu tiên mà request đi qua.

13 ký tự tiếp theo `1403169275002` là thời điểm tạo TraceId. Tiếp đó, 4 ký tự `1003` là sequence tự tăng, tăng từ 1000 đến 9000, sau khi đạt 9000 thì quay về 1000 và tiếp tục tăng. 5 ký tự cuối `56696` là process ID hiện tại. Để tránh xung đột TraceId khi một máy chạy nhiều process, process ID hiện tại được thêm vào cuối TraceId.

### 3. Quy tắc tạo SpanId

Span có nghĩa là tầng. Chẳng hạn, instance đầu tiên là tầng một; request được proxy hoặc phân phối đến instance tiếp theo để xử lý là tầng hai, và tiếp tục như vậy. Thông qua các tầng, SpanId biểu thị vị trí của call hiện tại trong toàn bộ cây call chain.

Giả sử instance server A nhận một request của người dùng và là root node của toàn bộ call chain. Các log không gọi service phát sinh khi tầng A xử lý request này đều có spanid bằng 0. Tầng A lần lượt gọi ba instance server B, C và D thông qua RPC, khi đó SpanId trong log của A lần lượt là 0.1, 0.2 và 0.3; trong B, C và D cũng lần lượt là 0.1, 0.2 và 0.3. Nếu hệ thống C lại gọi hai instance server E và F khi xử lý request, spanid tương ứng trong hệ thống C là 0.2.1 và 0.2.2; log tương ứng của hai hệ thống E và F cũng là 0.2.1 và 0.2.2.

Từ mô tả trên có thể thấy, nếu thu thập tất cả SpanId trong một call, ta có thể tạo thành một cây call chain hoàn chỉnh.

**Bản chất của việc tạo spanid là vừa truyền xuyên suốt qua các tầng, vừa kiểm soát số thứ tự tự tăng theo từng tầng.**

## Tình huống ba: Short URL

Short URL chủ yếu có hai chức năng là rút gọn và khôi phục URL. So với URL dài, short URL thuận tiện hơn khi truyền qua email, mạng xã hội, Weibo và điện thoại. Chẳng hạn, một URL vốn rất dài có thể được tạo thành short URL tương ứng thông qua dịch vụ short URL, tránh việc xuống dòng hoặc vượt giới hạn ký tự.

![Vai trò của short URL](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/distributed-id-design-short-url.png)

Các dịch vụ tạo ID phổ biến như ID tự tăng của MySQL, key tự tăng của Redis và segment mode đều tạo ra một chuỗi chữ số. Dịch vụ short URL chuyển URL dài của khách hàng thành short URL bằng cách nối ID dạng số mới tạo vào sau domain dwz.cn.

Nếu dùng trực tiếp ID dạng số, độ dài URL vẫn hơi lớn. Service có thể nén độ dài bằng cách chuyển ID dạng số sang một hệ cơ số cao hơn. Thuật toán chuyển cơ số để nén độ dài ngày càng được dùng nhiều trong triển khai short URL. Thuật toán này có thể tiếp tục rút ngắn URL. Thuật toán nén bằng chuyển cơ số có nhiều tình huống sử dụng trong đời sống, ví dụ:

- URL dài của khách hàng: <https://wenku.baidu.com/ndbusiness/browse/wenkuvipcashier?cashier_code=PCoperatebanner>
- Short URL ánh xạ từ ID: <https://dwz.cn/2047601319t66> (dùng để minh họa, có thể không mở được chính xác)
- Short URL sau khi chuyển cơ số: <https://dwz.cn/2ezwDJ0> (dùng để minh họa, có thể không mở được chính xác)

<!-- @include: @article-footer.snippet.md -->
