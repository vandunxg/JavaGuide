---
title: Tổng hợp giải pháp data desensitization
description: Giải thích chi tiết giải pháp data desensitization, bao gồm quy tắc desensitization cho số điện thoại, số căn cước, số thẻ ngân hàng và cách triển khai bằng công cụ Hutool.
category: System Design
tag:
  - Security
head:
  - - meta
    - name: keywords
      content: data desensitization,bảo vệ quyền riêng tư,desensitization số điện thoại,desensitization số căn cước,quy tắc masking,dữ liệu nhạy cảm,dữ liệu kiểm thử,tuân thủ
---

<!-- @include: @article-header.snippet.md -->

> Bài viết này được biên tập lại từ [Hutool: Một dòng code xử lý data desensitization - Jingdong Cloud Developer](https://mp.weixin.qq.com/s/1qFWczesU50ndPPLtABHFg).

## Data desensitization là gì

### Định nghĩa data desensitization

Baike Baidu định nghĩa data desensitization như sau:

> Data desensitization là việc biến đổi dữ liệu đối với một số thông tin nhạy cảm thông qua các quy tắc desensitization, nhằm bảo vệ đáng tin cậy dữ liệu nhạy cảm và riêng tư. Nhờ đó, có thể sử dụng an toàn các tập dữ liệu thực đã được desensitization trong môi trường development, testing, các môi trường non-production khác và môi trường outsourcing. Khi liên quan đến dữ liệu bảo mật của khách hàng hoặc một số dữ liệu nhạy cảm về thương mại, cần cải biến dữ liệu thực và cung cấp cho việc testing mà không vi phạm các điều kiện của quy tắc hệ thống; các thông tin cá nhân như số căn cước, số điện thoại, số thẻ và mã khách hàng đều cần được desensitization. Đây là một trong các kỹ thuật bảo mật database.

Nói chung, data desensitization là việc biến đổi dữ liệu đối với một số thông tin nhạy cảm thông qua các quy tắc desensitization, nhằm bảo vệ đáng tin cậy dữ liệu nhạy cảm và riêng tư.

Trong quá trình data desensitization, thường sử dụng các algorithm và kỹ thuật khác nhau để xử lý dữ liệu theo từng nhu cầu và scenario. Ví dụ, với số căn cước, có thể dùng algorithm masking để giữ lại vài chữ số đầu và thay các chữ số còn lại bằng “X” hoặc "\*"; với tên, có thể dùng algorithm pseudonymization để thay tên thật bằng một bí danh được tạo ngẫu nhiên.

### Các quy tắc desensitization thường dùng

Các quy tắc desensitization thường dùng nhằm bảo vệ an toàn dữ liệu nhạy cảm bằng cách biến đổi hoặc chỉnh sửa dữ liệu nhạy cảm trong quá trình xử lý và lưu trữ.

Dưới đây là một số quy tắc desensitization phổ biến:

- Thay thế (thường dùng): Thay một ký tự hoặc chuỗi ký tự cụ thể trong dữ liệu nhạy cảm bằng ký tự khác. Ví dụ, thay một số chữ số ở giữa số thẻ tín dụng bằng dấu sao (\*) hoặc ký tự khác.
- Xóa: Xóa ngẫu nhiên một phần nội dung của dữ liệu nhạy cảm. Ví dụ, xóa ngẫu nhiên 3 chữ số của số điện thoại.
- Xáo trộn: Làm xáo trộn thứ tự của một số ký tự hoặc field trong dữ liệu gốc. Ví dụ, đổi chỗ xen kẽ các vị trí ngẫu nhiên của số căn cước.
- Thêm nhiễu: Đưa một số sai số hoặc nhiễu vào dữ liệu để đạt hiệu quả desensitization. Ví dụ, thêm một số ký tự được tạo ngẫu nhiên vào dữ liệu nhạy cảm.
- Encryption hoặc tokenization (thường dùng): Khi cần khôi phục plaintext, có thể dùng encryption algorithm có bảo vệ tính toàn vẹn; khi không cần khôi phục plaintext, có thể chọn truncation, tokenization hoặc HMAC với key độc lập tùy mục đích. Các hàm hash như MD5 và SHA-256 không phải encryption algorithm; việc trực tiếp hash không có key đối với dữ liệu có cấu trúc như số thẻ ngân hàng còn có thể bị enumeration. Có thể tham khảo bài tổng hợp encryption algorithm phổ biến tại: <https://javaguide.cn/system-design/security/encryption-algorithms.html>.
- ……

## Các công cụ desensitization thường dùng

### Hutool

Hutool là một bộ công cụ cơ bản cho Java, đóng gói các method của JDK về file, stream, encryption và decryption, encoding conversion, regex, thread, XML cùng nhiều phần khác, tạo thành các lớp Util khác nhau, đồng thời cung cấp các component sau:

|       Module       |                                                       Giới thiệu                                                        |
| :----------------: | :---------------------------------------------------------------------------------------------------------------------: |
|     hutool-aop     |                            Đóng gói dynamic proxy của JDK, cung cấp hỗ trợ AOP không cần IOC                            |
| hutool-bloomFilter |                              Bloom filter, cung cấp Bloom filter của một số hash algorithm                              |
|    hutool-cache    |                                                Triển khai cache đơn giản                                                |
|    hutool-core     |                                  Core, bao gồm thao tác Bean, date và nhiều Util khác                                   |
|    hutool-cron     |                          Module scheduled task, cung cấp scheduled task với biểu thức Crontab                           |
|   hutool-crypto    |                   Module encryption và decryption, đóng gói symmetric, asymmetric và digest algorithm                   |
|     hutool-db      |                        Thao tác dữ liệu được đóng gói bằng JDBC, dựa trên tư tưởng ActiveRecord                         |
|     hutool-dfa     |                                        Tìm kiếm nhiều keyword dựa trên model DFA                                        |
|    hutool-extra    | Module mở rộng, đóng gói các thành phần bên thứ ba (template engine, email, Servlet, QR code, Emoji, FTP, tokenizer...) |
|    hutool-http     |                                     Đóng gói HTTP client dựa trên HttpUrlConnection                                     |
|     hutool-log     |                                       Facade log tự động nhận diện implementation                                       |
|   hutool-script    |                                     Đóng gói việc thực thi script, ví dụ Javascript                                     |
|   hutool-setting   |                               Đóng gói file Setting và Properties với nhiều chức năng hơn                               |
|   hutool-system    |                                  Đóng gói việc gọi system parameter (thông tin JVM...)                                  |
|    hutool-json     |                                                     Triển khai JSON                                                     |
|   hutool-captcha   |                                                Triển khai image captcha                                                 |
|     hutool-poi     |                                            Đóng gói Excel và Word trong POI                                             |
|   hutool-socket    |                                        Đóng gói Socket NIO và AIO dựa trên Java                                         |
|     hutool-jwt     |                                        Triển khai đóng gói JSON Web Token (JWT)                                         |

Có thể import từng module riêng theo nhu cầu, hoặc import toàn bộ module bằng cách import `hutool-all`; công cụ desensitization được sử dụng trong bài viết này nằm trong module `hutool.core`.

Phiên bản Hutool mới nhất hiện hỗ trợ các loại dữ liệu desensitization sau, về cơ bản bao phủ các thông tin nhạy cảm thường gặp:

1. User id
2. Tên tiếng Trung
3. Số căn cước
4. Số điện thoại cố định
5. Số điện thoại di động
6. Địa chỉ
7. Email
8. Password
9. Biển số xe mainland China, bao gồm xe thông thường và xe năng lượng mới
10. Thẻ ngân hàng

#### Triển khai desensitization bằng một dòng code

Method desensitization do Hutool cung cấp như hình dưới đây:

![](https://oss.javaguide.cn/github/javaguide/system-design/security/2023-08-01-10-2119fnVCIDozqHgRGx.png)

Lưu ý: Hutool thay thông tin nhạy cảm bằng `*`; phần triển khai cụ thể nằm trong method `StrUtil.hide`. Nếu muốn tùy chỉnh ký hiệu ẩn, có thể copy source code của Hutool và triển khai lại.

Dưới đây lấy desensitization của số điện thoại di động, số thẻ ngân hàng, số căn cước và password làm ví dụ, cùng với code testing tương ứng.

```java
import cn.hutool.core.util.DesensitizedUtil;
import org.junit.Test;
import org.springframework.boot.test.context.Spring BootTest;

/**
 *
 * @description: Triển khai data desensitization bằng Hutool
 */
@SpringBootTest
public class HuToolDesensitizationTest {

    @Test
    public void testPhoneDesensitization(){
        String phone="13723231234";
        System.out.println(DesensitizedUtil.mobilePhone(phone)); // output: 137****1234
    }
    @Test
    public void testBankCardDesensitization(){
        String bankCard="6217000130008255666";
        System.out.println(DesensitizedUtil.bankCard(bankCard)); // output: 6217 **** **** *** 5666
    }

    @Test
    public void testIdCardNumDesensitization(){
        String idCardNum="411021199901102321";
        // Chỉ hiển thị 4 chữ số đầu và 2 chữ số cuối
        System.out.println(DesensitizedUtil.idCardNum(idCardNum,4,2)); // output: 4110************21
    }
    @Test
    public void testPasswordDesensitization(){
        String password="www.jd.com_35711";
        System.out.println(DesensitizedUtil.password(password)); // output: ****************
    }
}
```

Trên đây là cách sử dụng utility class được Hutool đóng gói để triển khai data desensitization.

Cần đặc biệt lưu ý: password không phải field thông thường có thể tiếp tục hiển thị hoặc lưu trữ sau khi “desensitization”. Password cần được xử lý sớm bằng password hash algorithm chuyên dụng ngay tại entry point của server; không được lưu trữ, trả về hoặc ghi vào log dưới dạng plaintext. `password()` ở đây chỉ thay toàn bộ chuỗi bằng `*`, không thể thay thế password hash.

#### Kết hợp JackSon để triển khai desensitization bằng annotation

Sau khi có utility class cho data desensitization, nếu frontend có nhiều nơi cần hiển thị dữ liệu, không thể gọi một utility class ở từng nơi vì code sẽ rất dư thừa. Vậy làm thế nào để hoàn thành data desensitization một cách gọn gàng bằng annotation?

Nếu project là web project dựa trên Spring Boot, có thể tận dụng custom serialization của Jackson được Spring Boot tích hợp sẵn. Nguyên lý triển khai thực chất là thực hiện desensitization khi serialize JSON để render và trả về frontend.

**Bước 1: Enum của desensitization strategy.**

```java
/**
 * @author
 * @description: Enum của desensitization strategy
 */
public enum DesensitizationTypeEnum {
    // Tùy chỉnh
    MY_RULE,
    // User id
    USER_ID,
    // Tên tiếng Trung
    CHINESE_NAME,
    // Số căn cước
    ID_CARD,
    // Số điện thoại cố định
    FIXED_PHONE,
    // Số điện thoại di động
    MOBILE_PHONE,
    // Địa chỉ
    ADDRESS,
    // Email
    EMAIL,
    // Password
    PASSWORD,
    // Biển số xe mainland China, bao gồm xe thông thường và xe năng lượng mới
    CAR_LICENSE,
    // Thẻ ngân hàng
    BANK_CARD
}
```

Đoạn trên thể hiện các loại desensitization được hỗ trợ.

**Bước 2: Định nghĩa annotation `Desensitization` dùng cho desensitization.**

- `@Retention (RetentionPolicy.RUNTIME)`: có hiệu lực tại runtime.
- `@Target (ElementType.FIELD)`: có thể dùng trên field.
- `@JacksonAnnotationsInside`: annotation này là một meta-annotation, chủ yếu dùng để đóng gói và sử dụng cùng các annotation khác.
- `@JsonSerialize`: như đã nói ở trên, annotation này dùng để tùy chỉnh serialization; có thể dùng trên annotation, method, field hoặc class, có hiệu lực tại runtime và thực hiện custom serialization thông qua các method được override trong class serialization được cung cấp.

```java
/**
 * @author
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@JacksonAnnotationsInside
@JsonSerialize(using = DesensitizationSerialize.class)
public @interface Desensitization {
    /**
     * Loại dữ liệu desensitization; startInclude và endExclude có hiệu lực khi là MY_RULE
     */
    DesensitizationTypeEnum type() default DesensitizationTypeEnum.MY_RULE;

    /**
     * Vị trí bắt đầu desensitization (bao gồm)
     */
    int startInclude() default 0;

    /**
     * Vị trí kết thúc desensitization (không bao gồm)
     */
    int endExclude() default 0;
}
```

Lưu ý: vị trí bắt đầu và kết thúc chỉ có hiệu lực khi sử dụng custom desensitization enum `MY_RULE`.

**Bước 3: Tạo custom serialization class**

Đây là bước then chốt để triển khai data desensitization. Custom serialization class kế thừa `JsonSerializer`, triển khai interface `ContextualSerializer` và override hai method.

```java
/**
 * @author
 * @description: Custom serialization class
 */
@AllArgsConstructor
@NoArgsConstructor
public class DesensitizationSerialize extends JsonSerializer<String> implements ContextualSerializer {
    private DesensitizationTypeEnum type;

    private Integer startInclude;

    private Integer endExclude;

    @Override
    public void serialize(String str, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
        switch (type) {
            // Desensitization theo custom type
            case MY_RULE:
                jsonGenerator.writeString(CharSequenceUtil.hide(str, startInclude, endExclude));
                break;
            // Desensitization userId
            case USER_ID:
                jsonGenerator.writeString(String.valueOf(DesensitizedUtil.userId()));
                break;
            // Desensitization tên tiếng Trung
            case CHINESE_NAME:
                jsonGenerator.writeString(DesensitizedUtil.chineseName(String.valueOf(str)));
                break;
            // Desensitization số căn cước
            case ID_CARD:
                jsonGenerator.writeString(DesensitizedUtil.idCardNum(String.valueOf(str), 1, 2));
                break;
            // Desensitization số điện thoại cố định
            case FIXED_PHONE:
                jsonGenerator.writeString(DesensitizedUtil.fixedPhone(String.valueOf(str)));
                break;
            // Desensitization số điện thoại di động
            case MOBILE_PHONE:
                jsonGenerator.writeString(DesensitizedUtil.mobilePhone(String.valueOf(str)));
                break;
            // Desensitization địa chỉ
            case ADDRESS:
                jsonGenerator.writeString(DesensitizedUtil.address(String.valueOf(str), 8));
                break;
            // Desensitization email
            case EMAIL:
                jsonGenerator.writeString(DesensitizedUtil.email(String.valueOf(str)));
                break;
            // Desensitization password
            case PASSWORD:
                jsonGenerator.writeString(DesensitizedUtil.password(String.valueOf(str)));
                break;
            // Desensitization biển số xe
            case CAR_LICENSE:
                jsonGenerator.writeString(DesensitizedUtil.carLicense(String.valueOf(str)));
                break;
            // Desensitization thẻ ngân hàng
            case BANK_CARD:
                jsonGenerator.writeString(DesensitizedUtil.bankCard(String.valueOf(str)));
                break;
            default:
        }

    }

    @Override
    public JsonSerializer<?> createContextual(SerializerProvider serializerProvider, BeanProperty beanProperty) throws JsonMappingException {
        if (beanProperty != null) {
            // Kiểm tra data type có phải String hay không
            if (Objects.equals(beanProperty.getType().getRawClass(), String.class)) {
                // Lấy annotation đã định nghĩa
                Desensitization desensitization = beanProperty.getAnnotation(Desensitization.class);
                // Nếu là null
                if (desensitization == null) {
                    desensitization = beanProperty.getContextAnnotation(Desensitization.class);
                }
                // Nếu không là null
                if (desensitization != null) {
                    // Tạo instance của serialization class đã định nghĩa và trả về; tham số là type,
                    // vị trí bắt đầu và vị trí kết thúc được định nghĩa trong annotation.
                    return new DesensitizationSerialize(desensitization.type(), desensitization.startInclude(),
                            desensitization.endExclude());
                }
            }

            return serializerProvider.findValueSerializer(beanProperty.getType(), beanProperty);
        }
        return serializerProvider.findNullValueSerializer(null);
    }
}
```

Sau ba bước trên, đã hoàn thành việc triển khai data desensitization bằng annotation. Tiếp theo là testing.

Trước tiên định nghĩa một POJO để testing, rồi thêm strategy cần desensitization vào các field tương ứng.

```java
/**
 *
 * @description:
 */
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class TestPojo {

    private String userName;

    @Desensitization(type = DesensitizationTypeEnum.MOBILE_PHONE)
    private String phone;

    @Desensitization(type = DesensitizationTypeEnum.MY_RULE, startInclude = 0, endExclude = 2)
    private String address;
}
```

Tiếp theo viết một controller để testing:

```java
@RestController
public class TestController {

    @RequestMapping("/test")
    public TestPojo testDesensitization(){
        TestPojo testPojo = new TestPojo();
        testPojo.setUserName("Tôi là tên người dùng");
        testPojo.setAddress("Trái Đất - Bắc Kinh - quận Thông Châu - trụ sở Jingdong, tòa nhà số 2");
        testPojo.setPhone("13782946666");
        return testPojo;
    }

}
```

![](https://oss.javaguide.cn/github/javaguide/system-design/security/2023-08-02-16-497DdCBy8vbf2D69g.png)

Có thể thấy chúng ta đã triển khai data desensitization thành công.

### Apache ShardingSphere

ShardingSphere là một ecosystem gồm các giải pháp middleware database phân tán mã nguồn mở, được tạo thành từ ba product độc lập là Sharding-JDBC, Sharding-Proxy và Sharding-Sidecar (đang lên kế hoạch). Tất cả đều cung cấp các chức năng chuẩn hóa như data sharding, distributed transaction và database governance.

Apache ShardingSphere có một module data desensitization, tích hợp các chức năng data desensitization thường dùng. Nguyên lý cơ bản là parse và intercept SQL do user nhập, sau đó rewrite SQL theo cấu hình desensitization của user để encryption field gốc và decryption encryption field. Cuối cùng đạt được việc lưu trữ và query encryption/decryption mà user không cần nhận biết.

Apache ShardingSphere có thể tự động hóa và minh bạch hóa quy trình data desensitization, user không cần quan tâm đến chi tiết triển khai bên trong của desensitization. Ngoài ra, công cụ cung cấp nhiều desensitization strategy built-in và của bên thứ ba (AKS), user chỉ cần cấu hình đơn giản là có thể sử dụng.

Địa chỉ tài liệu chính thức: <https://shardingsphere.apache.org/document/4.1.1/cn/features/orchestration/encrypt/>.

### FastJSON

Khi phát triển web project, ngoài serialization tool mặc định của Spring, FastJson cũng là một tool serialization thường dùng cho Spring Web RESTful API.

FastJSON chủ yếu có hai cách triển khai data desensitization:

- Dựa trên annotation `@JSONField`: cần custom một serialization class dùng cho desensitization, sau đó chỉ định serialization type custom bằng `serializeUsing` trong `@JSONField` trên field cần desensitization.
- Dựa trên serialization filter: cần triển khai interface `ValueFilter`, override method `process` để hoàn thành custom desensitization, rồi sử dụng custom conversion strategy khi chuyển đổi JSON. Có thể tham khảo bài viết sau để xem triển khai cụ thể: <https://juejin.cn/post/7067916686141161479>.

### Mybatis-Mate

Trước tiên giới thiệu mối quan hệ giữa MyBatis, MyBatis-Plus và Mybatis-Mate:

- MyBatis là một persistence framework ưu tú, hỗ trợ SQL tùy chỉnh, stored procedure và advanced mapping.
- MyBatis-Plus là một tool enhancement của MyBatis, có thể đơn giản hóa đáng kể việc phát triển persistence layer.
- Mybatis-Mate là module cấp enterprise cung cấp cho MyBatis-Plus, hướng đến việc xử lý dữ liệu linh hoạt và gọn gàng hơn. Tuy nhiên, cần cấu hình license key trước khi sử dụng (có tính phí).

Mybatis-Mate hỗ trợ data desensitization cho sensitive word, tích hợp sẵn 9 quy tắc desensitization thường dùng như số điện thoại di động, email và số thẻ ngân hàng.

```java
@FieldSensitive("testStrategy")
private String username;

@Configuration
public class SensitiveStrategyConfig {

    /**
     * Inject desensitization strategy
     */
    @Bean
    public ISensitiveStrategy sensitiveStrategy() {
        // Custom xử lý desensitization cho type testStrategy
        return new SensitiveStrategy().addStrategy("testStrategy", t -> t + "***test***");
    }
}

// Bỏ qua xử lý desensitization, dùng trong scenario editing
RequestDataTransfer.skipSensitive();
```

### MyBatis-Flex

Tương tự MybatisPlus, MyBatis-Flex cũng là một enhancement framework của MyBatis. MyBatis-Flex cũng cung cấp chức năng data desensitization và có thể sử dụng miễn phí.

MyBatis-Flex cung cấp annotation `@ColumnMask()` cùng 9 quy tắc desensitization built-in, có thể dùng ngay:

```java
/**
 * Các cách data desensitization built-in
 */
public class Masks {
    /**
     * Desensitization số điện thoại di động
     */
    public static final String MOBILE = "mobile";
    /**
     * Desensitization số điện thoại cố định
     */
    public static final String FIXED_PHONE = "fixed_phone";
    /**
     * Desensitization số căn cước
     */
    public static final String ID_CARD_NUMBER = "id_card_number";
    /**
     * Desensitization tên tiếng Trung
     */
    public static final String CHINESE_NAME = "chinese_name";
    /**
     * Desensitization địa chỉ
     */
    public static final String ADDRESS = "address";
    /**
     * Desensitization email
     */
    public static final String EMAIL = "email";
    /**
     * Desensitization password
     */
    public static final String PASSWORD = "password";
    /**
     * Desensitization biển số xe
     */
    public static final String CAR_LICENSE = "car_license";
    /**
     * Desensitization số thẻ ngân hàng
     */
    public static final String BANK_CARD_NUMBER = "bank_card_number";
    //...
}
```

Ví dụ sử dụng:

```java
@Table("tb_account")
public class Account {

    @Id(keyType = KeyType.Auto)
    private Long id;

    @ColumnMask(Masks.CHINESE_NAME)
    private String userName;

    @ColumnMask(Masks.EMAIL)
    private String email;

}
```

Nếu các quy tắc desensitization built-in này không đáp ứng yêu cầu, vẫn có thể custom quy tắc desensitization.

1. Đăng ký quy tắc desensitization mới thông qua `MaskManager`:

```java
MaskManager.registerMaskProcessor("Tên quy tắc tùy chỉnh"
        , data -> {
            return data;
        })
```

2. Sử dụng quy tắc desensitization tùy chỉnh:

```java
@Table("tb_account")
public class Account {

    @Id(keyType = KeyType.Auto)
    private Long id;

    @ColumnMask("Tên quy tắc tùy chỉnh")
    private String userName;
}
```

Ngoài ra, với scenario cần bỏ qua xử lý desensitization, chẳng hạn khi vào trang editing để sửa dữ liệu user, MyBatis-Flex cũng cung cấp hỗ trợ tương ứng:

1. **`MaskManager#execWithoutMask`** (khuyến nghị): method này sử dụng template method design pattern, đảm bảo tự động khôi phục xử lý desensitization sau khi bỏ qua xử lý và thực thi logic liên quan.
2. **`MaskManager#skipMask`**: bỏ qua xử lý desensitization.
3. **`MaskManager#restoreMask`**: khôi phục xử lý desensitization, đảm bảo các thao tác tiếp theo tiếp tục sử dụng desensitization logic.

Triển khai của method `MaskManager#execWithoutMask` như sau:

```java
public static <T> T execWithoutMask(Supplier<T> supplier) {
    try {
        skipMask();
        return supplier.get();
    } finally {
        restoreMask();
    }
}
```

Các method `skipMask` và `restoreMask` của `MaskManager` thường được sử dụng cùng nhau; khuyến nghị dùng pattern `try{...}finally{...}`.

## Tổng kết

Bài viết này chủ yếu giới thiệu:

- Định nghĩa data desensitization: data desensitization là việc biến đổi dữ liệu đối với một số thông tin nhạy cảm thông qua các quy tắc desensitization, nhằm bảo vệ đáng tin cậy dữ liệu nhạy cảm và riêng tư.
- Các quy tắc desensitization thường dùng: thay thế, xóa, xáo trộn, thêm nhiễu và encryption.
- Các công cụ desensitization thường dùng: Hutool, Apache ShardingSphere, FastJSON, Mybatis-Mate và MyBatis-Flex.

## Tham khảo

- OWASP Cryptographic Storage Cheat Sheet: <https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html>
- PCI SSC FAQ 1492 - PAN masking: <https://www.pcisecuritystandards.org/faqs/1492/>
- Website chính thức của Hutool: <https://hutool.cn/docs/#/>
- Trao đổi về cách custom data desensitization: <https://juejin.cn/post/7046567603971719204>
- Triển khai data desensitization bằng FastJSON: <https://juejin.cn/post/7067916686141161479>

<!-- @include: @article-footer.snippet.md -->
