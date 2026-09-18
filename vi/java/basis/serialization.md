---
title: Java serialization giải thích chi tiết
description: "Phân tích chuyên sâu cơ chế serialization và deserialization của Java: giải thích chi tiết interface Serializable, keyword transient, tác dụng của serialVersionUID, lựa chọn serialization protocol và các trường hợp sử dụng như RPC, cache."
category: Java
tag:
  - Java Basics
head:
  - - meta
    - name: keywords
      content: Java serialization,deserialization,interface Serializable,keyword transient,serialVersionUID,serialization protocol,persistence đối tượng
---

## Serialization và deserialization là gì?

Nếu cần persist Java object, chẳng hạn lưu Java object vào file hoặc truyền Java object qua network, các trường hợp này đều cần dùng serialization.

Nói đơn giản:

- **Serialization**: chuyển data structure hoặc object thành dạng có thể lưu trữ hoặc truyền tải, thường là binary byte stream, cũng có thể là định dạng text như JSON, XML
- **Deserialization**: quá trình chuyển dữ liệu được tạo ra trong serialization thành data structure hoặc object ban đầu

Với ngôn ngữ lập trình hướng đối tượng như Java, object được serialization đều là object (Object), tức class (Class) sau khi được khởi tạo. Tuy nhiên, trong C++ là ngôn ngữ nửa hướng đối tượng, struct (structure) định nghĩa kiểu data structure, còn class tương ứng với kiểu object.

Dưới đây là các trường hợp sử dụng phổ biến của serialization và deserialization:

- Object cần được serialization trước khi truyền qua network (chẳng hạn khi thực hiện remote method call RPC), sau khi nhận object đã serialization thì cần deserialization;
- Cần serialization trước khi lưu object vào file, và cần deserialization khi đọc object từ file;
- Cần serialization trước khi lưu object vào database (như Redis), và cần deserialization khi đọc object từ cache database;
- Khi chuyển object thành byte representation cần lưu giữ lâu dài hoặc truyền qua các component, thường cần serialization; Java object thông thường được sử dụng trong JVM memory thì không cần serialization.

Wikipedia giới thiệu serialization như sau:

> **Serialization** trong xử lý dữ liệu của computer science là quá trình chuyển data structure hoặc trạng thái object thành định dạng có thể sử dụng (chẳng hạn lưu thành file, lưu trong buffer hoặc gửi qua network), để sau đó có thể khôi phục trạng thái ban đầu trong cùng một môi trường computer hoặc một môi trường khác. Khi thu được byte theo serialization format, có thể dùng chúng để tạo bản sao có cùng ngữ nghĩa với object ban đầu. Với nhiều object, chẳng hạn object phức tạp sử dụng nhiều reference, quá trình dựng lại bằng serialization này không dễ dàng. Object serialization trong lập trình hướng đối tượng không bao gồm các function mà object ban đầu liên kết tới. Quá trình này còn được gọi là object marshalling. Thao tác ngược của việc trích xuất data structure từ một chuỗi byte là deserialization (còn gọi là unmarshalling).

Tóm lại: **Mục đích chính của serialization là chuyển object thành dạng phù hợp để truyền qua network hoặc persist vào các medium như file system, database, cache.**

![](https://oss.javaguide.cn/github/javaguide/a478c74d-2c48-40ae-9374-87aacf05188c.png)

<p style="text-align:right;font-size:13px;color:gray">https://www.corejavaguru.com/java/serialization/interview-questions-1</p>

**Serialization protocol tương ứng với layer nào trong mô hình TCP/IP 4 layer?**

Chúng ta biết hai phía của network communication phải sử dụng và tuân thủ cùng một protocol. Mô hình TCP/IP 4 layer như sau, vậy serialization protocol thuộc layer nào?

1. Application layer
2. Transport layer
3. Network layer
4. Network interface layer

![Mô hình TCP/IP 4 layer](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-ip-4-model.png)

Như hình trên, trong OSI 7 layer protocol model, presentation layer chủ yếu xử lý và chuyển user data của application layer thành binary stream. Ngược lại, nó chuyển binary stream thành user data của application layer. Điều này chẳng phải tương ứng với serialization và deserialization sao?

Vì application layer, presentation layer và session layer trong OSI 7 layer protocol model đều tương ứng với application layer trong TCP/IP 4 layer model, nên serialization protocol là một phần của application layer trong TCP/IP protocol.

## Có những serialization protocol phổ biến nào?

Serialization tích hợp sẵn của JDK thường không được dùng vì hiệu suất serialization thấp và tồn tại vấn đề bảo mật. Các serialization protocol được dùng khá phổ biến gồm Hessian, Kryo, Protobuf, ProtoStuff; tất cả đều là serialization protocol dựa trên binary.

Các định dạng như JSON và XML thuộc nhóm text serialization. Dù khả năng đọc khá tốt nhưng performance kém, nên thường không được lựa chọn.

### Serialization tích hợp sẵn của JDK

Serialization tích hợp sẵn của JDK chỉ cần implement interface `java.io.Serializable`.

```java
@AllArgsConstructor
@NoArgsConstructor
@Getter
@Builder
@ToString
public class RpcRequest implements Serializable {
    private static final long serialVersionUID = 1905122041950251207L;
    private String requestId;
    private String interfaceName;
    private String methodName;
    private Object[] parameters;
    private Class<?>[] paramTypes;
    private RpcMessageTypeEnum rpcMessageTypeEnum;
}
```

**serialVersionUID có tác dụng gì?**

Serial number `serialVersionUID` có tác dụng kiểm soát version. Khi deserialization, hệ thống sẽ kiểm tra `serialVersionUID` trong stream có nhất quán với `serialVersionUID` của class hiện tại hay không; nếu không nhất quán sẽ throw `InvalidClassException`. Khuyến nghị mạnh mẽ mỗi serialization class đều tự chỉ định `serialVersionUID`. Nếu không khai báo tường minh, serialization runtime sẽ tính giá trị mặc định dựa trên cấu trúc class, chứ không phải do compiler tạo field.

**`serialVersionUID` được modifier bởi biến `static`; tại sao nó vẫn được “serialization”?**

~~Biến được modifier bởi `static` là static variable, nằm trong method area và bản thân nó không được serialization. `static` variable thuộc về class chứ không phải object. Sau khi deserialization, giá trị của `static` variable giống như được mặc định gán cho object, khiến ta có cảm giác `static` variable đã được serialization, nhưng thực tế chỉ là ảo giác.~~

**🐛 Đính chính (tham khảo [issue#2174](https://github.com/Snailclimb/JavaGuide/issues/2174))**:

Thông thường, `static` variable thuộc về class, không thuộc bất kỳ object instance riêng lẻ nào, nên bản thân chúng không được đưa vào data stream của object serialization. Serialization lưu trạng thái của object (tức giá trị của instance variable). Tuy nhiên, `serialVersionUID` là một trường hợp đặc biệt; serialization của `serialVersionUID` được xử lý đặc biệt. Điểm mấu chốt là `serialVersionUID` không được serialization như một phần của object state, mà được chính serialization mechanism sử dụng như một “fingerprint” hoặc “version number” đặc biệt.

Khi một object được serialization, `serialVersionUID` sẽ được ghi vào binary stream của serialization (giống như lưu một version number, chứ không phải lưu trạng thái bản thân `static` variable); khi deserialization, nó cũng được phân tích và kiểm tra tính nhất quán, qua đó xác minh version consistency của object đã serialization. Nếu hai giá trị không khớp, quá trình deserialization sẽ throw `InvalidClassException`, vì điều này thường có nghĩa là định nghĩa của class đã serialization đã thay đổi và có thể không còn tương thích.

Giải thích chính thức như sau:

> A serializable class can declare its own serialVersionUID explicitly by declaring a field named `"serialVersionUID"` that must be `static`, `final`, and of type `long`;
>
> Nếu muốn chỉ định tường minh `serialVersionUID`, cần dùng keyword `static` và `final` để modifier một variable kiểu `long` trong class; tên variable phải là `"serialVersionUID"`.

Nói cách khác, bản thân `serialVersionUID` (với vai trò static variable) thực sự không được serialization như object state. Tuy nhiên, giá trị của nó được Java serialization mechanism xử lý đặc biệt: được đọc và ghi vào serialization stream như một version identifier, dùng để kiểm tra version compatibility khi deserialization.

**Nếu có một số field không muốn serialization thì phải làm thế nào?**

Với variable không muốn serialization, có thể dùng keyword `transient` để modifier.

Tác dụng của keyword `transient`: ngăn serialization các variable trong instance được modifier bằng keyword này; khi object được deserialization, giá trị của variable được modifier bởi `transient` sẽ không được persist và khôi phục.

Ngoài ra, cần lưu ý một số điểm về `transient`:

- `transient` chỉ có thể modifier variable, không thể modifier class và method.
- Với variable được modifier bởi `transient`, sau deserialization giá trị variable sẽ được đặt thành default value của type. Ví dụ, nếu modifier cho type `int`, kết quả sau deserialization là `0`.
- `static` variable không thuộc bất kỳ Object nào, nên dù có được modifier bởi keyword `transient` hay không thì cũng không được serialization.

**Tại sao không khuyến nghị sử dụng serialization tích hợp sẵn của JDK?**

Chúng ta rất ít, hay gần như không bao giờ, sử dụng trực tiếp serialization tích hợp sẵn của JDK, chủ yếu vì các nguyên nhân sau:

- **Không hỗ trợ cross-language call**: nếu gọi service được phát triển bằng ngôn ngữ khác thì sẽ không được hỗ trợ.
- **Performance kém**: so với các serialization framework khác, performance thấp hơn; nguyên nhân chính là byte array sau serialization có kích thước lớn, làm tăng chi phí truyền tải.
- **Tồn tại vấn đề bảo mật**: bản thân serialization và deserialization không có vấn đề. Tuy nhiên, nếu input của dữ liệu deserialization có thể bị user kiểm soát, attacker có thể tạo input độc hại để deserialization tạo ra object ngoài dự kiến, đồng thời thực thi arbitrary code được tạo ra trong quá trình này. Bài đọc thêm: [Lỗ hổng bảo mật Java deserialization - Cryin](https://cryin.github.io/blog/secure-development-java-deserialization-vulnerability/), [Lỗ hổng bảo mật Java deserialization là gì? - Monica](https://www.zhihu.com/question/37562657/answer/1916596031).

### Kryo

Kryo là một tool serialization/deserialization hiệu năng cao. Nhờ đặc tính lưu trữ biến độ dài và sử dụng bytecode generation mechanism, Kryo có tốc độ chạy cao và bytecode size nhỏ.

Ngoài ra, Kryo đã là một serialization implementation rất mature, được sử dụng rộng rãi tại Twitter, Groupon, Yahoo cũng như nhiều open-source project nổi tiếng (như Hive, Storm).

[guide-rpc-framework](https://github.com/Snailclimb/guide-rpc-framework) sử dụng kryo để serialization; code liên quan đến serialization và deserialization như sau:

```java
/**
 * Kryo serialization class, Kryo serialization efficiency is very high, but only compatible with Java language
 *
 * @author shuang.kou
 * @createTime 2020-05-13 19:29:00
 */
@Slf4j
public class KryoSerializer implements Serializer {

    /**
     * Because Kryo is not thread safe. So, use ThreadLocal to store Kryo objects
     */
    private final ThreadLocal<Kryo> kryoThreadLocal = ThreadLocal.withInitial(() -> {
        Kryo kryo = new Kryo();
        kryo.register(RpcResponse.class);
        kryo.register(RpcRequest.class);
        return kryo;
    });

    @Override
    public byte[] serialize(Object obj) {
        try (ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
             Output output = new Output(byteArrayOutputStream)) {
            Kryo kryo = kryoThreadLocal.get();
            // Object->byte: serialize object into byte array
            kryo.writeObject(output, obj);
            kryoThreadLocal.remove();
            return output.toBytes();
        } catch (Exception e) {
            throw new SerializeException("Serialization failed");
        }
    }

    @Override
    public <T> T deserialize(byte[] bytes, Class<T> clazz) {
        try (ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(bytes);
             Input input = new Input(byteArrayInputStream)) {
            Kryo kryo = kryoThreadLocal.get();
            // byte->Object: deserialize object from byte array
            Object o = kryo.readObject(input, clazz);
            kryoThreadLocal.remove();
            return clazz.cast(o);
        } catch (Exception e) {
            throw new SerializeException("Deserialization failed");
        }
    }

}
```

GitHub address: [https://github.com/EsotericSoftware/kryo](https://github.com/EsotericSoftware/kryo).

### Protobuf

Protobuf bắt nguồn từ Google, performance khá tốt, đồng thời hỗ trợ nhiều language và cross-platform. Điểm bất tiện là cách sử dụng khá rườm rà vì cần tự định nghĩa IDL file và tạo serialization code tương ứng. Tuy không linh hoạt, nhưng mặt khác điều này cũng tránh nguy cơ serialization vulnerability của protobuf.

> Protobuf bao gồm định nghĩa serialization format, library của nhiều language và một IDL compiler. Thông thường cần định nghĩa proto file, sau đó dùng IDL compiler compile thành language cần dùng.

Một proto file đơn giản như sau:

```protobuf
// protobuf version
syntax = "proto3";
// SearchRequest sẽ được compile thành object tương ứng của các programming language khác nhau, chẳng hạn class trong Java, struct trong Go
message Person {
  // field kiểu string
  string name = 1;
  // field kiểu int
  int32 age = 2;
}
```

GitHub address: [https://github.com/protocolbuffers/protobuf](https://github.com/protocolbuffers/protobuf).

### ProtoStuff

Do Protobuf có khả năng sử dụng kém, “người anh” Protostuff đã ra đời.

protostuff dựa trên Google protobuf nhưng cung cấp nhiều function hơn và cách sử dụng đơn giản hơn. Dễ sử dụng hơn không có nghĩa là performance của ProtoStuff kém hơn.

GitHub address: [https://github.com/protostuff/protostuff](https://github.com/protostuff/protostuff).

### Hessian

Hessian là một lightweight, custom-described binary RPC protocol. Hessian là một serialization implementation khá lâu đời và cũng hỗ trợ cross-language.

![](https://oss.javaguide.cn/github/javaguide/8613ec4c-bde5-47bf-897e-99e0f90b9fa3.png)

Serialization mặc định được bật trong Dubbo2.x là Hessian2, tuy nhiên Dubbo đã chỉnh sửa Hessian2, chỉ là cấu trúc tổng thể vẫn gần như tương tự.

### Tổng kết

Kryo là serialization method chuyên biệt cho Java và có performance rất tốt. Nếu application của bạn chỉ hướng tới Java thì có thể cân nhắc sử dụng. Một bài viết trên website chính thức của Dubbo cũng đề cập rằng nên dùng Kryo làm serialization method trong production environment (địa chỉ bài viết: <https://cn.dubbo.apache.org/zh-cn/docsv2.7/user/serialization/>).

![](https://oss.javaguide.cn/github/javaguide/java/569e541a-22b2-4846-aa07-0ad479f07440-20230814090158124.png)

Các method serialization như Protobuf, ProtoStuff, hessian đều hỗ trợ cross-language; nếu có nhu cầu cross-language thì có thể cân nhắc sử dụng.

Ngoài các method serialization đã giới thiệu ở trên, còn có Thrift, Avro và các method khác.

<!-- @include: @article-footer.snippet.md -->
