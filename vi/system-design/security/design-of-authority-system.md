---
title: Giải thích chi tiết thiết kế hệ thống phân quyền
description: Role-Based Access Control (RBAC) là cơ chế cấp các permission tương ứng thông qua role của user, giúp thực hiện access control linh hoạt; so với việc cấp permission trực tiếp cho user, cách này đơn giản, hiệu quả và có khả năng mở rộng hơn.
category: System Design
tag:
  - Security
head:
  - - meta
    - name: keywords
      content: thiết kế hệ thống phân quyền,RBAC,ABAC,role và permission của user,permission tài nguyên,mô hình permission,kiểm tra permission,hệ thống authorization
---

<!-- @include: @article-header.snippet.md -->

> Tác giả: Nhóm kỹ thuật Zhuanzhuan
>
> Bài gốc: <https://mp.weixin.qq.com/s/ONMuELjdHYa0yQceTj01Iw>

## Vấn đề và hiện trạng của hệ thống phân quyền cũ

Trước đây, công ty Zhuanzhuan chưa có một hệ thống quản lý phân quyền thống nhất. Việc quản lý phân quyền do từng bộ phận tự phát triển hoặc sử dụng hệ thống phân quyền của bộ phận khác, sự thiếu thống nhất này đã gây ra nhiều vấn đề:

1. Các bộ phận liên tục xây dựng lại những chức năng tương tự, chi phí bảo trì cao.
2. Mỗi hệ thống chỉ giải quyết một phần trường hợp sử dụng, giải pháp chưa đủ tổng quát. Khi chọn giải pháp cho dự án mới, không có phương án quản lý phân quyền đáng tin cậy.
3. Thiếu quản lý log và quy trình phê duyệt thống nhất, rất khó truy vết thông tin authorization.

Dựa trên các vấn đề trên, cuối năm ngoái công ty đã khởi động việc xây dựng hệ thống phân quyền thống nhất Zhuanzhuan. Mục tiêu là phát triển một hệ thống quản lý phân quyền linh hoạt, dễ sử dụng và an toàn để các bộ phận sử dụng.

## Cách thiết kế hệ thống phân quyền trong ngành

Hiện nay, ngành chủ yếu có hai mô hình permission, lần lượt được giới thiệu dưới đây:

- **Role-Based Access Control (RBAC)**
- **Attribute-Based Access Control (ABAC)**

### Mô hình RBAC

**Role-Based Access Control (RBAC)** là cơ chế cấp các permission tương ứng thông qua role của user, giúp thực hiện access control linh hoạt; so với việc cấp permission trực tiếp cho user, cách này đơn giản, hiệu quả và có khả năng mở rộng hơn.

Một user có thể sở hữu nhiều role, mỗi role lại có thể được gán nhiều permission. Như vậy, ta xây dựng được mô hình authorization “user - role - permission”. Trong mô hình này, user và role, role và permission đều có quan hệ nhiều-nhiều.

Mô tả bằng sơ đồ như sau:

![Sơ đồ mô hình permission RBAC](https://oss.javaguide.cn/github/javaguide/system-design/security/design-of-authority-system/rbac.png)

Khi sử dụng `mô hình RBAC`, ta phân tích tình huống thực tế của user, dựa trên trách nhiệm và nhu cầu chung để cấp cho họ các role khác nhau. Quan hệ `user -> role -> permission` này giúp ta không cần quản lý permission của từng user riêng lẻ; user nhận các permission cần thiết từ role được cấp.

Lấy một trường hợp đơn giản (hệ thống phân quyền của Gitlab) làm ví dụ: hệ thống user có ba role là `Admin`, `Maintainer` và `Operator`. Ba role này có các permission khác nhau. Chẳng hạn, chỉ `Admin` có permission tạo và xóa repository code, các role khác không có. Khi cấp role `Admin` cho một user, user đó sẽ có hai permission **tạo repository code** và **xóa repository code**.

Với `mô hình RBAC`, khi có nhiều user cùng sở hữu một permission, ta chỉ cần tạo role có permission đó rồi phân bổ role cho các user khác nhau. Về sau, chỉ cần sửa permission của role là permission của toàn bộ user trong role sẽ tự động được cập nhật.

### Mô hình ABAC

**Attribute-Based Access Control (ABAC)** là một mô hình authorization linh hoạt hơn `mô hình RBAC`. Nguyên lý của nó là dùng nhiều attribute để phán đoán động một thao tác có được phép hay không. Mô hình này được sử dụng khá phổ biến trong các hệ thống cloud, chẳng hạn AWS và Alibaba Cloud.

Hãy xem xét các trường hợp kiểm soát permission sau:

1. Cấp cho một người permission edit một cuốn sách cụ thể.
2. Khi department sở hữu một document giống với department của user, user có thể truy cập document đó.
3. Khi user là owner của một document và trạng thái document là bản nháp, user có thể edit document đó.
4. Cấm người thuộc department A truy cập system B trước 9 giờ sáng.
5. Cấm truy cập system A với tư cách administrator ở mọi nơi ngoài Shanghai.
6. User có permission thao tác với các order được tạo trước ngày 2022-06-07.

Có thể thấy các trường hợp trên rất khó thực hiện bằng `mô hình RBAC`, vì `mô hình RBAC` chỉ mô tả user có thể thực hiện thao tác gì, nhưng bản thân `mô hình RBAC` không mô tả điều kiện của thao tác và dữ liệu được thao tác. Đây lại chính là ưu điểm của `mô hình ABAC`: tư tưởng của `mô hình ABAC` là dựa trên attribute của user, dữ liệu được truy cập và các yếu tố môi trường để tính toán động xem user có permission thực hiện thao tác hay không.

#### Nguyên lý của mô hình ABAC

Trong `mô hình ABAC`, việc một thao tác có được phép hay không được quyết định bằng cách tính toán động dựa trên object, resource, operation và thông tin environment.

- **Object**: object là user đang gửi request truy cập resource. Attribute của user bao gồm ID, tài nguyên cá nhân, role, department và tư cách thành viên trong organization.
- **Resource**: resource là asset hoặc object mà user hiện tại muốn truy cập, chẳng hạn file, data, server, thậm chí API.
- **Operation**: operation là thao tác user muốn thực hiện trên resource. Các operation thường gặp gồm “đọc”, “ghi”, “edit”, “copy” và “xóa”.
- **Environment**: environment là context của mỗi access request. Attribute của environment bao gồm thời gian và vị trí truy cập, thiết bị của object, protocol giao tiếp và độ mạnh của encryption.

Trong quá trình thực thi statement quyết định của `mô hình ABAC`, decision engine sẽ dựa trên statement đã định nghĩa, kết hợp các yếu tố như object, resource, operation và environment để tính toán động kết quả quyết định. Mỗi khi có access request, hệ thống decision của `mô hình ABAC` sẽ phân tích xem các giá trị attribute có khớp với policy đã thiết lập hay không. Nếu có policy khớp, access request sẽ được cho phép.

## Tư tưởng thiết kế của hệ thống phân quyền mới

Kết hợp với hiện trạng nghiệp vụ của Zhuanzhuan, `mô hình RBAC` đáp ứng phần lớn trường hợp sử dụng của các bộ phận, đồng thời chi phí phát triển thấp hơn nhiều so với hệ thống phân quyền dựa trên `mô hình ABAC`. Vì vậy, hệ thống phân quyền mới chọn triển khai dựa trên `mô hình RBAC`. Với những hệ thống nghiệp vụ thực sự không thể đáp ứng, tạm thời không hỗ trợ để bảo đảm hệ thống mới được triển khai nhanh và các bộ phận có thể sử dụng sớm hơn.

`Mô hình RBAC` tiêu chuẩn hoàn toàn tuân theo chuỗi `user -> role -> permission`, tức permission của user được kiểm soát hoàn toàn bởi các role user sở hữu. Tuy nhiên, cách này có một nhược điểm: muốn thêm permission cho user thì phải tạo thêm một role, khiến hiệu quả thao tác thực tế khá thấp. Vì vậy, trên cơ sở `mô hình RBAC`, chúng tôi bổ sung khả năng thêm permission trực tiếp cho user. Nghĩa là vừa có thể thêm role cho user, vừa có thể thêm permission trực tiếp cho user. Cuối cùng, permission của user được cấu thành từ các role và permission độc lập mà user sở hữu.

**Mô hình permission của hệ thống phân quyền mới**: permission cuối cùng của user = permission do các role user sở hữu mang lại + permission được cấu hình riêng cho user; hai phần này lấy hợp.

Phương án của hệ thống phân quyền mới như hình dưới đây:

![Phương án hệ thống phân quyền mới](https://oss.javaguide.cn/github/javaguide/system-design/security/design-of-authority-system/new-authority-system-design.png)

- Trước hết, toàn bộ user của tập đoàn (bao gồm user bên ngoài) được quản lý thống nhất thông qua chức năng **đăng nhập và đăng ký thống nhất**. Đồng thời, hệ thống được kết nối với module thông tin cơ cấu tổ chức của công ty, bảo đảm thông tin của cùng một người nhất quán trong mọi system. Điều này cũng tạo khả năng quản lý permission dựa trên cơ cấu tổ chức về sau.
- Tiếp theo, vì hệ thống phân quyền mới cần phục vụ toàn bộ nghiệp vụ của tập đoàn nên phải hỗ trợ quản lý permission cho nhiều system. Trước khi quản lý permission, user cần chọn system tương ứng rồi cấu hình thông tin **menu permission** và **data permission** của system đó, từ đó thiết lập các permission point của system. _PS: Phần giải thích cụ thể về menu permission và data permission sẽ được trình bày chi tiết bên dưới._
- Cuối cùng, tạo các role khác nhau trong system và cấu hình permission point cho từng role. Chẳng hạn, role store manager có permission thao tác với store staff, permission xem data của cửa hàng đó, v.v. Sau khi cấu hình role này, về sau chỉ cần thêm role này cho store manager là họ sẽ có các permission tương ứng.

Sau khi hoàn tất các cấu hình trên, có thể quản lý permission của user. Có hai cách thêm permission cho user:

1. Chọn user trước rồi thêm permission. Cách này có thể thêm bất kỳ role hoặc permission point menu/data nào cho user.
2. Chọn role trước rồi liên kết user. Cách này chỉ có thể thêm role cho user, không thể thêm riêng permission point menu/data.

Phương án thiết kế cụ thể của hai cách này sẽ được giải thích chi tiết ở phần sau.

### Quản lý permission của chính hệ thống phân quyền

Đối với hệ thống phân quyền, trước hết cần thiết kế tốt việc quản lý permission của chính hệ thống, tức là phải quản lý được “ai có thể vào hệ thống phân quyền, ai có thể quản lý permission của các system khác”. User của hệ thống phân quyền được chia thành ba loại:

1. **Super administrator**: có toàn bộ permission thao tác của hệ thống phân quyền, có thể thực hiện mọi thao tác của chính hệ thống và quản lý các thao tác quản trị của application system đã kết nối với hệ thống phân quyền.
2. **User thao tác permission**: user có role super administrator của ít nhất một application system đã kết nối. Các thao tác user này có thể thực hiện bị giới hạn trong phạm vi permission của application system mà user sở hữu. User thao tác permission là một identity, không cần phân bổ mà tự động nhận được theo rule.
3. **User thông thường**: user thông thường cũng có thể được xem là một identity. Ngoài hai loại user trên, tất cả user còn lại đều là user thông thường. Họ chỉ có thể đăng ký kết nối system và truy cập trang đăng ký permission.

### Định nghĩa loại permission

Trong hệ thống phân quyền mới, permission được chia thành hai loại:

- **Menu function permission**: bao gồm navigation directory của system, permission truy cập menu, cùng permission thao tác button và API.
- **Data permission**: bao gồm permission về phạm vi truy vấn data. Trong các system khác nhau, permission này thường được gọi là “organization”, “site”, v.v. Trong hệ thống phân quyền mới, tất cả đều được gọi thống nhất là “organization” để quản lý data permission.

### Phân loại role mặc định

Mỗi system được thiết kế ba role mặc định để đáp ứng nhu cầu quản lý permission cơ bản:

- **Super administrator**: role này có toàn bộ permission của system, có thể sửa các cấu hình như permission role của system và cấp permission cho user khác.
- **System administrator**: role này có khả năng cấp permission cho user khác và sửa các cấu hình như permission role của system, nhưng bản thân role không có permission nào.
- **Authorization administrator**: role này có khả năng cấp permission cho user khác, nhưng phạm vi authorization không vượt quá permission mà bản thân user sở hữu.

> Ví dụ: authorization administrator A có thể thêm permission cho user B, nhưng phạm vi thêm vào phải nhỏ hơn hoặc bằng permission mà user A đã sở hữu.

Qua cách phân chia này, **sở hữu permission** và **sở hữu khả năng authorization** được tách thành hai phần, đáp ứng mọi trường hợp kiểm soát permission.

## Thiết kế module cốt lõi của hệ thống phân quyền mới

Phần trên đã giới thiệu tư tưởng thiết kế tổng thể của hệ thống phân quyền mới. Tiếp theo, chúng ta sẽ lần lượt giới thiệu thiết kế các module cốt lõi.

### Quản lý system/menu/data permission

Các bước kết nối một system mới vào hệ thống phân quyền gồm:

1. Tạo system.
2. Cấu hình menu function permission.
3. Cấu hình data permission (tùy chọn).
4. Tạo role của system.

Các bước 1, 2 và 3 đều được hoàn thành trong module quản lý system. Quy trình cụ thể như hình dưới đây:

![Sơ đồ quy trình kết nối system](https://oss.javaguide.cn/github/javaguide/system-design/security/design-of-authority-system/new-authority-system-design-access-flow-chart.png)

User có thể thực hiện các thao tác CRUD trên thông tin cơ bản của system. Các system được phân biệt duy nhất bằng `mã system`. Đồng thời, `mã system` cũng được dùng làm prefix của mã menu và data permission. Thiết kế này bảo đảm tính duy nhất toàn cục của mã permission.

Ví dụ, nếu mã system là `test_online`, định dạng mã menu của system đó sẽ là `test_online:m_xxx`.

Thiết kế giao diện quản lý system như sau:

![Thiết kế giao diện quản lý system](https://oss.javaguide.cn/github/javaguide/system-design/security/design-of-authority-system/new-authority-system-management-interface.png)

#### Quản lý menu

Trước hết, hệ thống phân quyền mới phân loại menu thành `directory`, `menu` và `operation`, như hình dưới đây:

![Giao diện quản lý menu](https://oss.javaguide.cn/github/javaguide/system-design/security/design-of-authority-system/new-authority-system-menu.png)

Ý nghĩa tương ứng của chúng là:

- **Directory**: directory cấp cao nhất của application system, thường nằm bên phải logo của system.
- **Menu**: menu nhiều cấp ở bên trái application system, thường nằm bên dưới logo của system, đồng thời cũng là cấu trúc menu được sử dụng phổ biến nhất.
- **Operation**: các phần trong page như button, interface và những thành phần khác có thể định nghĩa là operation hoặc page element.

Thiết kế giao diện quản lý menu như sau:

![Thiết kế giao diện quản lý menu](https://oss.javaguide.cn/github/javaguide/system-design/security/design-of-authority-system/new-authority-system-menu-management-interface.png)

Việc sử dụng data menu permission cũng cung cấp hai mode:

- **Dynamic menu mode**: trong mode này, việc thêm và xóa menu hoàn toàn do hệ thống phân quyền tiếp quản. Nghĩa là khi thêm menu trong hệ thống phân quyền, application system cũng sẽ đồng bộ thêm menu. Ưu điểm của mode này là sửa menu không cần đưa project lên production.
- **Static menu mode**: việc thêm và xóa menu do frontend của application system kiểm soát, hệ thống phân quyền chỉ kiểm soát access permission. Trong mode này, hệ thống phân quyền chỉ có thể xác định user có permission của menu hiện tại hay không; việc hiển thị cụ thể do frontend quyết định dựa trên data permission.

Cần đặc biệt lưu ý: ẩn directory, menu hoặc button ở frontend chỉ nhằm cải thiện trải nghiệm user, không thể được xem là security boundary thực sự. Dù dùng dynamic hay static menu mode, backend vẫn phải mặc định từ chối access chưa được authorization rõ ràng và kiểm tra trong từng request xem user hiện tại có quyền thực hiện operation tương ứng hay không. Khi liên quan đến data cụ thể, còn phải tiếp tục kiểm tra quyền sở hữu resource, tenant, organization và phạm vi data; không thể chỉ kiểm tra user có một menu nào đó hay không, cũng không được tin tưởng user ID, organization ID hoặc resource ID do client truyền vào.

### Quản lý role và user

Quản lý role và user đều là các module cốt lõi có thể trực tiếp thay đổi permission của user. Tư tưởng thiết kế tổng thể như hình dưới đây:

![Thiết kế module quản lý role và user](https://oss.javaguide.cn/github/javaguide/system-design/security/design-of-authority-system/role-and-user-management.png)

Trọng tâm thiết kế module này là phải tính đến thao tác hàng loạt. Dù là liên kết user thông qua role hay thêm/xóa/reset permission hàng loạt cho user, hệ thống đều cần thiết kế tốt các trường hợp thao tác hàng loạt.

### Đăng ký permission

Ngoài việc thêm permission cho user khác, hệ thống phân quyền mới còn hỗ trợ user tự đăng ký permission. Bên cạnh flow approval thông thường (đăng ký, phê duyệt, xem), module này có một chức năng khá đặc biệt: làm thế nào để user chọn đúng permission mình cần. Vì vậy, trong thiết kế module, ngoài việc chọn trực tiếp role, còn hỗ trợ chọn ngược role thông qua menu/data permission point, như hình dưới đây:

![Giao diện đăng ký permission](https://oss.javaguide.cn/github/javaguide/system-design/security/design-of-authority-system/permission-application.png)

### Log thao tác

Log thao tác của system được chia thành hai loại:

1. **Log lịch sử thao tác**: log các thao tác quan trọng mà user có thể xem và tra cứu.
2. **Log service**: log được tạo ra trong quá trình service của system vận hành. Lượng thông tin của log service lớn hơn log lịch sử thao tác nhưng không thuận tiện cho việc tìm kiếm và xem. Vì vậy, hệ thống phân quyền cần cung cấp chức năng log lịch sử thao tác.

Trong hệ thống phân quyền mới, mọi thao tác của user có thể chia thành ba loại: thêm mới, cập nhật và xóa. Tất cả module cũng có thể liệt kê, chẳng hạn quản lý user, quản lý role, quản lý menu, v.v. Sau khi xác định rõ các thông tin này, một log có thể được trừu tượng hóa thành: ai (Who), vào thời gian nào (When), đã thực hiện những thao tác gì trên module nào của những ai (Target).

Lưu toàn bộ record vào database theo cách này sẽ giúp việc xem và lọc log thuận tiện hơn.

## Tổng kết và triển vọng

Đến đây, các tư tưởng thiết kế cốt lõi và module của hệ thống phân quyền mới đã được giới thiệu. Hệ thống mới đã được nhiều nghiệp vụ nội bộ Zhuanzhuan kết nối và sử dụng, việc quản lý permission thuận tiện hơn trước rất nhiều. Là một hệ thống nền tảng của mỗi công ty, thiết kế linh hoạt và hoàn chỉnh của hệ thống phân quyền có thể giúp hoạt động nghiệp vụ phát triển hiệu quả hơn trong tương lai.

Hai bài tiếp theo:

- [Thiết kế và triển khai hệ thống phân quyền thống nhất Zhuanzhuan (phần triển khai backend)](https://mp.weixin.qq.com/s/hFTDckfxhSnoM_McP18Vkg)
- [Thiết kế và triển khai hệ thống phân quyền thống nhất Zhuanzhuan (phần triển khai frontend)](https://mp.weixin.qq.com/s/a_P4JAwxgunhfmJvpBnWYA)

## Tài liệu tham khảo

- OWASP Authorization Cheat Sheet: <https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html>
- Chọn mô hình access control phù hợp: <https://docs.authing.cn/v2/guides/access-control/choose-the-right-access-control-model.html>

<!-- @include: @article-footer.snippet.md -->
