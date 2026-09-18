---
title: "Vì sao cả frontend và backend đều phải thực hiện data validation?"
description: "Giải thích chi tiết sự cần thiết của data validation ở frontend và backend, tầm quan trọng của parameter validation, authorization validation và các biện pháp bảo vệ để ngăn bypass validation ở frontend."
category: System Design
tag:
  - Security
head:
  - - meta
    - name: keywords
      content: data validation,frontend validation,backend validation,parameter validation,authorization validation,input validation,security protection,injection prevention
---

> Câu hỏi phỏng vấn liên quan:
>
> - Frontend đã validation rồi, backend còn cần validation nữa không?
> - Frontend đã thực hiện data validation, vì sao backend vẫn cần validation lại (thậm chí nghiêm ngặt hơn)?
> - Frontend/backend cần validation những nội dung nào?

Khi phát triển Web, dù viết trang frontend hay API backend, chúng ta đều không thể tránh việc làm việc với dữ liệu. Vậy làm thế nào để bảo đảm dữ liệu được truyền qua lại là đáng tin cậy và an toàn? Cần dựa vào **data validation**. Hơn nữa, frontend phải thực hiện, backend **càng phải thực hiện**, đồng thời còn phải thêm “khóa” quan trọng là **authorization validation**; không thể thiếu bất kỳ phần nào!

Vì sao lại nói như vậy? Hãy nghĩ xem, frontend validation chủ yếu nhằm cải thiện trải nghiệm người dùng và chặn một số dữ liệu “nhập bừa” rõ ràng không hợp lệ, nhưng người hiểu một chút kỹ thuật có thể bypass frontend validation cực kỳ dễ dàng (chẳng hạn gửi request trực tiếp bằng các công cụ như Postman). Vì vậy, **backend validation là tuyến phòng thủ cuối cùng và vững chắc nhất cho security và tính chính xác của dữ liệu trong hệ thống**. Nó phải bảo đảm dữ liệu đi vào hệ thống không chỉ đúng format mà còn phù hợp với business rule; quan trọng nhất là người thực hiện thao tác đó phải có **quyền**!

![](https://oss.javaguide.cn/github/javaguide/system-design/security/user-input-validation.png)

## Frontend validation

Frontend validation giống như một người gác cổng chu đáo. Mục đích chính là nhanh chóng cho người dùng biết chỗ nào không đúng ngay khi họ nhập dữ liệu để họ sửa, tránh việc submit rất lâu rồi backend báo không hợp lệ và phải làm lại. Lợi ích của việc này rất rõ ràng:

1. **Trải nghiệm người dùng tốt:** Có thông báo ngay khi nhập, biết lỗi ngay lập tức và dễ sửa, người dùng cảm thấy thao tác mượt mà, không khó chịu.
2. **Giảm áp lực cho backend:** Chặn trước ở frontend các lỗi format rõ ràng và dữ liệu thiếu trường bắt buộc, giảm request không hợp lệ gửi đến backend, tiết kiệm tài nguyên server và network traffic. Cần lưu ý rằng backend vẫn phải validation, chỉ là thêm frontend validation giúp giảm nhiều request không hợp lệ.

Vậy frontend thường cần validation những gì?

- **Validation trường bắt buộc:** Điều cơ bản nhất là các trường cần nhập không được để trống.
- **Validation format:** Chẳng hạn email phải có dạng email (như `xxx@xx.com`), số điện thoại phải là 11 chữ số. Đây là lúc regular expression phát huy tác dụng.
- **Validation dữ liệu nhập lặp lại:** Bảo đảm nội dung của hai lần nhập giống nhau, chẳng hạn trường “xác nhận mật khẩu” khi đăng ký.
- **Validation range/length:** Tuổi không thể là số âm, đúng không? Độ dài mật khẩu phải nằm trong khoảng 6 đến 20 ký tự, đúng không? Những điều này đều cần được kiểm tra.
- **Validation legality/business:** Chẳng hạn username đã được đăng ký chưa? Sản phẩm đã chọn còn trong kho không? Việc này phụ thuộc business cụ thể và cần phối hợp với backend.
- **Validation upload file:** Giới hạn file type (chẳng hạn chỉ hỗ trợ format `.jpg`, `.png`) và file size.
- **Validation security:** Phòng ngừa các hành vi xấu như XSS (cross-site scripting attack), xử lý input của người dùng để script do kẻ khác viết không thể chạy trên trang của chúng ta.
- ...vân vân, tùy theo yêu cầu business.

Tóm lại, trọng tâm của frontend validation là **hướng dẫn người dùng nhập đúng** và **cải thiện trải nghiệm tương tác**.

## Backend validation

Frontend validation chỉ là tuyến phòng thủ đầu tiên. Dù cải thiện trải nghiệm người dùng nhưng nó vẫn có thể bị bypass; yếu tố thực sự mang tính quyết định là backend validation. Backend cần xem mọi dữ liệu do frontend gửi đến là “có thể có vấn đề” và tiến hành kiểm tra toàn diện. Backend validation không chỉ bao phủ các kiểm tra cơ bản ở frontend (như format, range, length) mà còn cần validation nghiêm ngặt và chuyên sâu hơn để bảo đảm security và tính nhất quán của dữ liệu trong hệ thống. Sau đây là các nội dung trọng tâm của backend validation:

1. **Validation tính đầy đủ:** Các field được yêu cầu rõ trong API documentation phải tồn tại, chẳng hạn `userId` và `orderId`. Nếu thiếu bất kỳ field bắt buộc nào, backend phải trả về error ngay lập tức và từ chối xử lý request.
2. **Validation tính hợp lệ/tồn tại:** Xác minh dữ liệu truyền vào có thực sự hợp lệ hay không. Chẳng hạn `productId` được truyền đến có tồn tại trong database không? `couponId` đã hết hạn hoặc đã được sử dụng chưa? Thông thường cần kiểm tra database hoặc gọi service khác để xác nhận.
3. **Validation tính nhất quán:** Với thao tác liên quan đến nhiều data object, xác minh chúng có phù hợp với business logic hay không. Chẳng hạn trước khi cập nhật order status, cần bảo đảm status hiện tại của order cho phép sửa đổi, không thể chuyển trực tiếp từ “chưa thanh toán” sang “đã hoàn tất”. Validation tính nhất quán là yếu tố then chốt để bảo đảm dữ liệu được luân chuyển chính xác.
4. **Validation security:** Backend phải phòng ngừa nhiều loại malicious attack, bao gồm nhưng không giới hạn ở XSS và SQL injection. Mọi external input đều phải được filter và validation nghiêm ngặt, chẳng hạn sử dụng parameterized query để ngăn SQL injection hoặc escape HTML data trả về để tránh cross-site scripting attack.
5. ...Về cơ bản, validation nào frontend có thể thực hiện thì backend cũng phải thực hiện lại vì security.

Ở backend Java, lần nào cũng tự viết `if-else` để thực hiện các validation cơ bản này thì rất mất công. May mắn là Java community cung cấp cho chúng ta **Bean Validation**, một standard specification. Nó cho phép dùng **annotation** để khai báo trực tiếp validation rule trên property của JavaBean (chẳng hạn DTO object), rất thuận tiện.

- **JSR 303 (1.0):** Đặt nền tảng, giới thiệu những “người bạn cũ” như `@NotNull`, `@Size`, `@Min`, `@Max`.
- **JSR 349 (1.1):** Bổ sung validation cho method parameter và return value, cùng các cải tiến như validation group.
- **JSR 380 (2.0):** Hỗ trợ Java 8, hỗ trợ date/time API mới, đồng thời bổ sung các annotation thực tế hơn như `@NotEmpty`, `@NotBlank`, `@Email`.

Spring Boot thời kỳ đầu (khoảng trước 2.3.x): `spring-boot-starter-web` đã tích hợp sẵn `hibernate-validator`, không cần thêm gì.

Spring Boot 2.3.x trở đi: Để linh hoạt hơn, dependency liên quan đến validation được tách riêng. Bạn cần tự thêm dependency `spring-boot-starter-validation`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

Bean Validation specification và implementation của nó (chẳng hạn Hibernate Validator) cung cấp nhiều annotation để khai báo validation rule theo kiểu declarative. Dưới đây là một số annotation thường dùng và giải thích:

- `@NotNull`: Kiểm tra element được annotation áp dụng (bất kỳ type nào) không được là `null`.
- `@NotEmpty`: Kiểm tra element được annotation áp dụng (chẳng hạn `CharSequence`, `Collection`, `Map`, `Array`) không được là `null` và size/length không được bằng 0. Lưu ý: với string, `@NotEmpty` cho phép string chứa whitespace, chẳng hạn `" "`.
- `@NotBlank`: Kiểm tra `CharSequence` (chẳng hạn `String`) được annotation áp dụng không được là `null`, đồng thời length sau khi loại bỏ whitespace đầu và cuối phải lớn hơn 0 (tức là không được là string chỉ gồm whitespace).
- `@Null`: Kiểm tra element được annotation áp dụng bắt buộc phải là `null`.
- `@AssertTrue` / `@AssertFalse`: Kiểm tra element type `boolean` hoặc `Boolean` được annotation áp dụng bắt buộc phải là `true` / `false`.
- `@Min(value)` / `@Max(value)`: Kiểm tra value của numeric type (hoặc biểu diễn dạng string) được annotation áp dụng phải lớn hơn hoặc bằng / nhỏ hơn hoặc bằng `value` được chỉ định. Áp dụng cho integer type (`byte`, `short`, `int`, `long`, `BigInteger`, v.v.).
- `@DecimalMin(value)` / `@DecimalMax(value)`: Chức năng tương tự `@Min` / `@Max` nhưng áp dụng cho numeric type có phần thập phân (`BigDecimal`, `BigInteger`, `CharSequence`, `byte`, `short`, `int`, `long` và wrapper tương ứng). `value` phải là biểu diễn dạng string của một số.
- `@Size(min=, max=)`: Kiểm tra size/length của element được annotation áp dụng (chẳng hạn `CharSequence`, `Collection`, `Map`, `Array`) phải nằm trong range `min` và `max` được chỉ định (bao gồm cả boundary).
- `@Digits(integer=, fraction=)`: Kiểm tra value của numeric type (hoặc biểu diễn dạng string) được annotation áp dụng: số chữ số ở phần nguyên phải ≤ `integer`, số chữ số ở phần thập phân phải ≤ `fraction`.
- `@Pattern(regexp=, flags=)`: Kiểm tra `CharSequence` (chẳng hạn `String`) được annotation áp dụng có khớp regular expression (`regexp`) được chỉ định hay không. `flags` có thể chỉ định matching mode (chẳng hạn không phân biệt hoa thường).
- `@Email`: Kiểm tra `CharSequence` (chẳng hạn `String`) được annotation áp dụng có phù hợp với format Email hay không (đã tích hợp một regular expression tương đối nới lỏng).
- `@Past` / `@Future`: Kiểm tra date hoặc time type (`java.util.Date`, `java.util.Calendar`, các type trong package JSR 310 `java.time`) được annotation áp dụng có nằm trước / sau thời điểm hiện tại hay không.
- `@PastOrPresent` / `@FutureOrPresent`: Tương tự `@Past` / `@Future`, nhưng cho phép bằng thời điểm hiện tại.
- …

Khi method của Controller sử dụng annotation `@RequestBody` để nhận request body và bind nó vào một object, có thể thêm annotation `@Valid` trước parameter đó để trigger validation object. Nếu validation thất bại, nó sẽ throw `MethodArgumentNotValidException`.

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Person {
    @NotNull(message = "classId không được để trống")
    private String classId;

    @Size(max = 33)
    @NotNull(message = "name không được để trống")
    private String name;

    @Pattern(regexp = "((^Man$|^Woman$|^UGM$))", message = "giá trị sex không nằm trong phạm vi cho phép")
    @NotNull(message = "sex không được để trống")
    private String sex;

    @Email(message = "email không đúng định dạng")
    @NotNull(message = "email không được để trống")
    private String email;
}


@RestController
@RequestMapping("/api")
public class PersonController {
    @PostMapping("/person")
    public ResponseEntity<Person> getPerson(@RequestBody @Valid Person person) {
        return ResponseEntity.ok().body(person);
    }
}
```

Với dữ liệu simple type được mapping trực tiếp tới method parameter (chẳng hạn path variable `@PathVariable` hoặc request parameter `@RequestParam`), cách validation có một chút khác biệt:

1. **Thêm annotation `@Validated` trên Controller class:** Annotation này do Spring cung cấp (không thuộc JSR standard), giúp Spring xử lý annotation validation ở method level. **Đây là bước bắt buộc.**
2. **Đặt annotation validation trực tiếp trên method parameter:** Áp dụng trực tiếp các annotation validation như `@Min`, `@Max`, `@Size`, `@Pattern` vào parameter `@PathVariable` hoặc `@RequestParam` tương ứng.

Nhất định không được quên thêm annotation `@Validated` trên class, vì annotation này cho Spring biết cần validation method parameter.

```java
@RestController
@RequestMapping("/api")
@Validated // Bước quan trọng 1: phải thêm @Validated trên class
public class PersonController {

    @GetMapping("/person/{id}")
    public ResponseEntity<Integer> getPersonByID(
            @PathVariable("id")
            @Max(value = 5, message = "ID không được vượt quá 5") // Bước quan trọng 2: đặt annotation validation trực tiếp trên parameter
            Integer id
    ) {
        // Nếu id truyền vào > 5, Spring sẽ throw ConstraintViolationException trước khi đi vào method body.
        // Global exception handler cũng cần xử lý exception này.
        return ResponseEntity.ok().body(id);
    }

    @GetMapping("/person")
    public ResponseEntity<String> findPersonByName(
            @RequestParam("name")
            @NotBlank(message = "name không được để trống") // Đồng thời áp dụng cho @RequestParam
            @Size(max = 10, message = "độ dài name không được vượt quá 10")
            String name
    ) {
        return ResponseEntity.ok().body("Found person: " + name);
    }
}
```

Bean Validation chủ yếu giải quyết validation ở **tầng format và syntax của dữ liệu**. Nhưng chỉ như vậy vẫn chưa đủ.

## Authorization validation

Format dữ liệu đã được validation, không có vấn đề. Nhưng **user đang đăng nhập hiện tại có quyền thực hiện thao tác này không?** Đây chính là vấn đề **authorization validation** cần giải quyết. Chẳng hạn:

- User thông thường có thể sửa order của người khác không? (Không.)
- Guest có thể truy cập API của admin console không? (Không.)
- Guest có thể quản lý thông tin của user khác không? (Không.)
- VIP user có thể sử dụng coupon dành riêng không? (Có.)
- …

Data validation và authorization validation không phải trong mọi API đều tuân thủ cứng nhắc cùng một thứ tự. Thông thường cần hoàn tất request parsing, giới hạn length và format validation cơ bản, đồng thời xác nhận identity của user càng sớm càng tốt; trước khi tiết lộ với bên gọi việc resource có tồn tại hay trả về query result, bắt buộc phải hoàn tất coarse-grained và object-level authorization. Khi authorization phụ thuộc vào resource attribute, có thể đưa authorization condition trực tiếp vào query hoặc lập tức validation sau khi đọc, nhưng không được tiết lộ resource information ra bên ngoài khi chưa được authorization. Sau đó, đưa business consistency validation và authorization chi tiết hơn vào Service hoặc data access layer để thực thi theo transaction semantics.

Authorization validation quan tâm đến việc “**ai (Who)** có thể thực hiện **thao tác gì (Action)** trên **resource nào (What)**”. Dù flow được chia thành bao nhiêu layer, cũng không được trả về resource content khi chưa authorization, và không thể chỉ dựa vào một lần kiểm tra ở Controller hoặc frontend.

**Vì sao authorization validation quan trọng như vậy?**

- **Nền tảng security:** Ngăn truy cập và thao tác chưa được authorization, bảo vệ user data và system security.
- **Cô lập business:** Bảo đảm các role khác nhau (admin, user thông thường, VIP user, v.v.) chỉ có thể truy cập và thao tác trên các chức năng trong phạm vi quyền hạn của mình.
- **Yêu cầu compliance:** Nhiều quy định trong ngành có yêu cầu nghiêm ngặt về quyền truy cập dữ liệu.

Hiện nay, cách phổ biến trong backend Java là sử dụng security framework trưởng thành để triển khai authorization validation thay vì tự viết (dễ xảy ra lỗi và khó maintain).

1. **Spring Security (standard của ngành, khuyến nghị):** Dựa trên filter chain để intercept request, thực hiện authentication (bạn là ai?) và authorization (bạn có thể làm gì?). Spring Security mạnh, community sôi động và tích hợp liền mạch với Spring ecosystem. Tuy nhiên, configuration tương đối phức tạp và learning curve khá dốc.
2. **Apache Shiro:** Một security framework phổ biến khác, nhẹ hơn Spring Security và có API trực quan, dễ hiểu hơn. Nó cũng cung cấp các chức năng như authentication, authorization, session management và encryption. Với project không quen Spring hoặc cảm thấy Spring Security quá nặng, đây là một lựa chọn tốt.
3. **Sa-Token:** Lightweight Java authorization framework do cộng đồng trong nước phát triển. Hỗ trợ authentication và authorization, single sign-on, kick user offline, tự động gia hạn, v.v. So với Spring Security và Shiro, Sa-Token tích hợp sẵn nhiều chức năng dùng ngay hơn và cũng dễ sử dụng hơn.
4. **Kiểm tra thủ công (không khuyến nghị cho scenario phức tạp):** Tự lấy thông tin user hiện tại trong code ở Service layer hoặc Controller layer (chẳng hạn từ SecurityContextHolder hoặc Session), sau đó dùng `if-else` để kiểm tra role hoặc permission của user. Authorization logic bị coupling với business logic, code bị lặp, khó maintain và dễ bỏ sót. Chỉ phù hợp với authorization scenario rất đơn giản.

**Giới thiệu ngắn về authorization model:**

- **RBAC (Role-Based Access Control):** Role-Based Access Control. Gán role cho user, gán permission cho role. User sở hữu tổng hợp permission của tất cả role mà mình có. Đây là model phổ biến nhất.
- **ABAC (Attribute-Based Access Control):** Attribute-Based Access Control. Quyết định dựa trên user attribute, resource attribute, action attribute và environment attribute. Linh hoạt hơn nhưng cũng phức tạp hơn.

Thông thường, phần lớn system đều sử dụng RBAC authorization model hoặc phiên bản đơn giản hóa của nó. Có thể mô tả bằng hình như sau:

![Sơ đồ minh họa RBAC authorization model](https://oss.javaguide.cn/github/javaguide/system-design/security/design-of-authority-system/rbac.png)

Để xem giới thiệu chi tiết về authorization system design, có thể đọc bài viết này: [Giải thích chi tiết về authorization system design](https://javaguide.cn/system-design/security/design-of-authority-system.html).

## Tổng kết

Tóm lại, để xây dựng một Web application an toàn, ổn định và có trải nghiệm người dùng tốt, cần thiết lập cả ba “cửa kiểm soát”: frontend data validation, backend data validation và backend authorization validation; mỗi phần có trọng tâm riêng:

- **Frontend data validation:** Cải thiện trải nghiệm người dùng, giảm request không hợp lệ, là tuyến phòng thủ “thân thiện” đầu tiên.
- **Backend data validation:** Bảo đảm format dữ liệu đúng và phù hợp với business rule, là tuyến phòng thủ “kỹ thuật” ngăn “dirty data” đi vào database. Bean Validation cho phép dùng annotation để khai báo trực tiếp validation rule trên property của JavaBean (chẳng hạn DTO object), rất thuận tiện.
- **Backend authorization validation:** Bảo đảm “đúng người” làm “đúng việc”, là tuyến phòng thủ “security” ngăn thao tác vượt quyền. Các framework như Spring Security, Shiro và Sa-Token có thể giúp triển khai authorization validation.

## Tài liệu tham khảo

- OWASP Authorization Cheat Sheet: <https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html>
- Vì sao cả frontend và backend đều cần thực hiện data validation?: <https://juejin.cn/post/7306045519099658240>
- Giải thích chi tiết về authorization system design: <https://javaguide.cn/system-design/security/design-of-authority-system.html>
