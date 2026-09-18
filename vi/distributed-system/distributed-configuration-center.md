---
title: "Giải thích chi tiết và so sánh configuration center phân tán: Apollo, Nacos, Spring Cloud Config và K8s ConfigMap"
description: "Giải thích chi tiết về nguyên lý và lựa chọn configuration center phân tán, bao quát khác biệt kiến trúc, cơ chế đẩy cấu hình, gray release, thiết kế high availability và các trọng điểm phỏng vấn của Apollo, Nacos, Spring Cloud Config và Kubernetes ConfigMap."
category: "Hệ thống phân tán"
keywords:
  - "configuration center"
head:
  - - meta
    - name: keywords
      content: "configuration center,configuration center phân tán,Apollo,Nacos,Spring Cloud Config,Kubernetes ConfigMap,đẩy cấu hình,long polling,gray release,câu hỏi phỏng vấn configuration center"
---

## Vì sao cần dùng configuration center?

Trong kiến trúc microservice, ứng dụng được tách thành nhiều service triển khai độc lập, mỗi service có cấu hình riêng (địa chỉ service, tham số database, feature toggle...). Số lượng cấu hình tăng cùng số lượng service, môi trường và cluster. Cách dùng file cấu hình truyền thống có các vấn đề sau:

- **Phải restart khi sửa**: dù cấu hình nằm trong codebase hay file bên ngoài, nhiều ứng dụng phải restart process thì cấu hình mới mới có hiệu lực.
- **Gắn với phát hành**: nếu cấu hình nằm trong codebase, thay đổi cấu hình thường phải gắn với việc phát hành code, khó gray release và rollback độc lập.
- **Bảo mật yếu**: ghi trực tiếp cấu hình nhạy cảm (mật khẩu database, API Key) vào codebase dễ dẫn đến rò rỉ.
- **Thiếu kiểm soát quyền**: không thể kiểm soát quyền chi tiết đối với các thao tác xem, sửa và phát hành cấu hình.
- **Cấu hình phân tán, khó quản lý**: cấu hình của nhiều môi trường (development/test/production) và nhiều cluster nằm rải rác, khó bảo trì tập trung.

Ngoài ra, configuration center thường cung cấp các khả năng tăng cường sau:

- **Quản lý version**: ghi lại người sửa, thời gian sửa và nội dung của mỗi lần thay đổi cấu hình, hỗ trợ rollback một lần bấm.
- **Gray release**: trước tiên đẩy cấu hình cho một phần instance để xác minh, giảm rủi ro thay đổi (Apollo và Nacos 1.1.0+ hỗ trợ).

![Apollo configuration center](https://oss.javaguide.cn/github/javaguide/config-center/view-release-history.png)

Tất nhiên, không phải hệ thống nào cũng cần configuration center. Với ứng dụng monolith, một môi trường, ít cấu hình và tần suất thay đổi thấp, `application-{profile}.yml`, environment variable hoặc Kubernetes ConfigMap + rolling restart thường đã đủ. Configuration center sẽ làm tăng chi phí vận hành, failure domain và chuỗi điều tra sự cố; team nhỏ hoặc cấu hình ít thay đổi không cần over-engineering.

Từ góc nhìn distributed system, configuration center là control plane điển hình: nó quyết định “hiện tại nên dùng cấu hình nào”, còn client chịu trách nhiệm pull, cache, listen và refresh. Việc phát hành, gray release, rollback và local snapshot của client đều là sự đánh đổi giữa quyết định tập trung và khả năng chịu lỗi của client. Có thể đặt sự đánh đổi này cạnh [Giải thích chi tiết về distributed coordination](./protocol/centralized-and-decentralized.md) để hiểu cùng nhau.

Configuration center cũng thường xuất hiện cùng [RPC](./rpc/) và [API gateway](./api-gateway.md): RPC framework cần lấy địa chỉ service, timeout, retry, degradation và các cấu hình governance khác; gateway cũng có thể dựa vào configuration center để thực hiện dynamic routing và gray rule. Chúng giải quyết các vấn đề khác nhau, nhưng trong hệ thống microservice thực tế thường nằm trên cùng một chain.

## Có những configuration center phổ biến nào? Chọn thế nào?

| Giải pháp                                                           | Trạng thái          | Đặc điểm                                                               |
| ------------------------------------------------------------------- | ------------------- | ---------------------------------------------------------------------- |
| [Spring Cloud Config](https://cloud.spring.io/spring-cloud-config/) | Đang hoạt động      | Hỗ trợ native trong hệ sinh thái Spring, lưu trữ dựa trên Git          |
| [Nacos](https://github.com/alibaba/nacos)                           | Đang hoạt động      | Alibaba open source, hợp nhất configuration center + service discovery |
| [Apollo](https://github.com/apolloconfig/apollo)                    | Đang hoạt động      | Ctrip open source, khả năng quản lý cấu hình, phân quyền và audit mạnh |
| K8s ConfigMap                                                       | Đang hoạt động      | Giải pháp native của Kubernetes                                        |
| Disconf / Qconf                                                     | Lâu không hoạt động | Không khuyến nghị dùng cho project mới                                 |

**Đề xuất lựa chọn**:

- Chỉ cần configuration center → **Apollo** (khả năng quản lý chi tiết hơn) hoặc **Nacos** (khởi động single-node nhẹ hơn)
- Cần configuration center + service discovery → **Nacos**
- Dùng hệ sinh thái Spring Cloud và ưu tiên đơn giản → **Spring Cloud Config**
- Môi trường Kubernetes → mount **K8s ConfigMap** bằng Volume + application layer listen file. Khi ConfigMap được mount dưới dạng Volume, kubelet sẽ đồng bộ định kỳ; thời điểm có thể nhìn thấy cuối cùng phụ thuộc vào chu kỳ đồng bộ của kubelet và cách local cache lan truyền. Cách dùng environment variable và mount `subPath` sẽ không tự động cập nhật. Hot reload có thể dùng inotify để listen file đã mount, hoặc dùng Spring Cloud Kubernetes listen thay đổi ConfigMap qua K8s Watch API và trigger refresh.

**Apollo vs Nacos vs Spring Cloud Config**

> **Ghi chú version**: So sánh dưới đây dựa trên Apollo 2.x, Nacos 2.x và Spring Cloud Config 4.x/5.x. Hệ Spring Boot 3 thường tương ứng với Spring Cloud Config 4.x; hệ Spring Boot 4 tương ứng với Spring Cloud 2025.x release train mới hơn. Nếu vẫn dùng hệ Spring Boot 2 thì tương ứng với Spring Cloud Config 3.x.

| Hạng mục                      | Apollo                                                    | Nacos                                                              | Spring Cloud Config                                       |
| ----------------------------- | --------------------------------------------------------- | ------------------------------------------------------------------ | --------------------------------------------------------- |
| Giao diện cấu hình            | Có (phân quyền, audit và quy trình phát hành khá đầy đủ)  | Có                                                                 | Không (thường thao tác qua nền tảng Git)                  |
| Cấu hình có hiệu lực realtime | Có (HTTP long polling, thường nhận biết ở mức giây)       | Có (thông báo thay đổi qua gRPC + client pull)                     | Bán realtime (cần trigger refresh hoặc broadcast qua Bus) |
| Quản lý version               | Native                                                    | Native                                                             | Phụ thuộc Git                                             |
| Quản lý quyền                 | Có (nhiều cấp theo application/namespace/environment)     | Có                                                                 | Phụ thuộc nền tảng Git                                    |
| Gray release                  | Có (rule chi tiết hơn)                                    | Có (1.1.0+, khả năng tương đối cơ bản)                             | Không hỗ trợ                                              |
| Rollback cấu hình             | Có                                                        | Có                                                                 | Phụ thuộc Git                                             |
| Thông báo cảnh báo            | Có                                                        | Có                                                                 | Không hỗ trợ                                              |
| Đa ngôn ngữ                   | Có (Open API / client đa ngôn ngữ)                        | Có (Open API / client đa ngôn ngữ)                                 | Thiên về ứng dụng Spring                                  |
| Đa môi trường                 | Có (thường cô lập vật lý)                                 | Có (thường cô lập logic bằng Namespace)                            | Cần kết hợp nhiều Git repository                          |
| Component phụ thuộc           | MySQL (registry mặc định được nhúng trong Config Service) | MySQL bên ngoài (khuyến nghị cho production) / Derby nhúng + JRaft | Git + message queue tùy chọn                              |

**So sánh chuyên sâu**:

1. **Apollo**: mô hình quyền, audit phát hành, diff trước khi phát hành và các tính năng quản lý như gray rule chi tiết hơn, phù hợp với team yêu cầu cao về governance cấu hình. Trong trường hợp cô lập vật lý nhiều môi trường (FAT/UAT/PROD), mỗi môi trường cần triển khai Config Service, Admin Service và database riêng; yêu cầu vận hành ở mức tương đối cao.
2. **Nacos**: hợp nhất configuration + registry center, triển khai đơn giản (single-node chỉ cần một file Jar). Cluster production nên dùng MySQL bên ngoài; Derby nhúng + JRaft phù hợp hơn với môi trường test hoặc quy mô nhỏ. Mô hình Namespace/Group/DataId của Nacos dễ bắt đầu, nhưng việc cô lập môi trường thường thiên về cô lập logic.
3. **Spring Cloud Config**: kiến trúc đơn giản nhất (dựa trên Git), nhưng tính realtime kém và cần component bổ sung để tự động refresh.

## Ranh giới giữa configuration center, registry center và K8s ConfigMap

- **Application configuration center (Apollo/Nacos/Spring Cloud Config)**: chủ yếu giải quyết việc quản lý tập trung, audit, gray release và dynamic refresh các cấu hình ứng dụng như business parameter, toggle, threshold và connection information.
- **Service registry (Eureka/Nacos/Consul)**: chủ yếu giải quyết việc đăng ký, discovery instance và đồng bộ health status của service. Nacos đồng thời cung cấp khả năng configuration center và service registry, nhưng hai loại trách nhiệm vẫn khác nhau.
- **Kubernetes ConfigMap**: chủ yếu giải quyết việc quản lý runtime configuration của container như tham số khởi động Pod, environment variable và file mount; không tự cung cấp approval khi phát hành, gray rule và refresh object trong ứng dụng.
- **Service Mesh / Ingress configuration**: chủ yếu giải quyết các governance strategy như traffic routing, circuit breaker, retry, timeout và gray traffic; configuration object thường là CRD hoặc resource của control plane.

## Các điểm chính khi thiết kế configuration center

Khi thiết kế hoặc lựa chọn configuration center, cần chú ý các khả năng sau:

### 1. Cơ chế đẩy cấu hình

| Chế độ           | Tính realtime                 | Áp lực lên server                                          | Độ phức tạp triển khai | Trường hợp sử dụng       |
| ---------------- | ----------------------------- | ---------------------------------------------------------- | ---------------------- | ------------------------ |
| **Push**         | Cao (mức millisecond)         | Cao (cần duy trì connection)                               | Cao                    | Yêu cầu realtime cao     |
| **Pull**         | Thấp (mức giây ~ phút)        | Cao (polling không hiệu quả)                               | Thấp                   | Rất ít thay đổi cấu hình |
| **Long polling** | Trung bình đến cao (mức giây) | Trung bình (áp lực memory lớn khi có rất nhiều connection) | Trung bình             | **Giải pháp phổ biến**   |

> **Giải thích cơ chế đẩy**:
>
> - **Apollo**: dùng HTTP long polling. Client gửi request; nếu server có thay đổi, server trả về ngay; nếu không có thay đổi, request bị treo (server mặc định khoảng 60s, read timeout của client thường dài hơn), trong thời gian đó hễ có thay đổi thì server lập tức response.
> - **Nacos 2.x**: chain service discovery được nâng cấp thành gRPC bidirectional stream, realtime tốt hơn; nói chính xác hơn, chain của configuration center là mô hình hai giai đoạn “thông báo thay đổi + client pull”: server thông báo một cấu hình đã thay đổi, sau đó client pull nội dung cấu hình mới khi cần.
>
> **Lưu ý**: nói chính xác, long polling vẫn là request pull do client khởi tạo, chỉ là server treo request để đạt gần realtime; bảng trên tách riêng long polling theo thông lệ ngành để nhấn mạnh đặc điểm vận hành. Long polling tiết kiệm CPU và network overhead hơn short polling, nhưng khi số lượng client đạt hàng trăm nghìn, server vẫn phải duy trì rất nhiều request bị treo. Ví dụ với Apollo, server treo request dựa trên Spring MVC `DeferredResult`, bên dưới dựa vào tính năng async của Servlet 3.0 và Tomcat NIO Connector để xử lý, nên vẫn có giới hạn về memory và số lượng connection.

### 2. Danh sách chức năng bắt buộc

- **Kiểm soát quyền**: việc xem, sửa và phát hành cấu hình cần được cấp quyền theo cấp độ.
- **Audit log**: ghi đầy đủ người thao tác, thời gian và nội dung thay đổi cấu hình.
- **Quản lý version**: mỗi lần phát hành tạo một version number, hỗ trợ rollback về bất kỳ version lịch sử nào.
- **Gray release**: trước tiên đẩy cấu hình đến một phần instance, sau khi xác minh đạt mới phát hành toàn bộ.
- **Cô lập đa môi trường**: quản lý riêng cấu hình của môi trường development, test và production.
- **Triển khai high availability**: bản thân configuration center cần triển khai theo cluster để tránh single point of failure.

### 3. Khả năng chịu lỗi của client và thứ tự khởi động

Configuration center là hạ tầng; khi không khả dụng, nó sẽ ảnh hưởng đến nhiều ứng dụng nghiệp vụ. Vì vậy client phải có khả năng chịu lỗi:

- **Multi-level cache**: ưu tiên đọc cấu hình trong memory; khi configuration center không khả dụng thì đọc local snapshot; nếu local snapshot cũng không tồn tại thì dùng default value dự phòng trong code hoặc từ chối khởi động.
- **Khởi động degradation**: với cấu hình không quan trọng, có thể khởi động trước bằng local snapshot rồi kết nối bất đồng bộ đến configuration center; với cấu hình quan trọng như địa chỉ database và encryption key, có thể chọn “không có cấu hình thì không khởi động”.
- **Reconnect khi mất kết nối**: sau khi long polling hoặc long connection bị ngắt, client nên reconnect theo backoff strategy để tránh làm configuration center quá tải bởi traffic tức thời khi khôi phục.
- **Ranh giới refresh**: dynamic refresh không có nghĩa mọi object đều tự động thay đổi. Chẳng hạn, trong Spring, giá trị đã được inject vào field thông thường, field `final` hoặc logic conditional assembly có thể không refresh như mong đợi; cần kết hợp `@RefreshScope`, listener hoặc thiết kế lại lifecycle của Bean.

## Giới thiệu thiết kế configuration center qua Apollo

### Giới thiệu Apollo

Theo giới thiệu chính thức của Apollo:

> [Apollo](https://github.com/apolloconfig/apollo) (Apollo) là configuration center phân tán do bộ phận framework của Ctrip phát triển, có thể quản lý tập trung cấu hình của ứng dụng trong nhiều môi trường và cluster khác nhau. Sau khi sửa cấu hình, cấu hình có thể được đẩy realtime đến phía ứng dụng; Apollo cũng có các đặc tính như quản lý quyền và governance quy trình chuẩn hóa, phù hợp với trường hợp quản lý cấu hình microservice.
>
> Server được phát triển dựa trên Spring Boot và Spring Cloud. Sau khi đóng gói, server có thể chạy trực tiếp mà không cần cài thêm application container như Tomcat.
>
> Java client không phụ thuộc framework nào, có thể chạy trong mọi Java runtime environment, đồng thời hỗ trợ tốt môi trường Spring/Spring Boot.

Các đặc tính cốt lõi của Apollo:

- **Thay đổi cấu hình có hiệu lực realtime (hot release)**: dựa trên long polling, có thể nhận cấu hình mới trong vòng 1s.
- **Gray release**: chỉ đẩy cấu hình đến một phần application, giảm rủi ro thay đổi.
- **Triển khai đơn giản**: một môi trường chỉ phụ thuộc MySQL; registry center đi kèm Apollo (mặc định là Eureka) chạy dưới dạng nhúng trong process Config Service, không cần triển khai độc lập. Khi cô lập vật lý nhiều môi trường, cần triển khai một bộ Config Service, Admin Service và database riêng cho mỗi môi trường.
- **Đa ngôn ngữ**: cung cấp HTTP interface, không giới hạn ngôn ngữ lập trình.

Xem [hướng dẫn sử dụng chính thức của Apollo](https://www.apolloconfig.com/#/zh/) để biết cách sử dụng Apollo.

### Phân tích kiến trúc Apollo

Mô hình cơ bản của Apollo do bên chính thức cung cấp (nguồn ảnh: tài liệu chính thức Apollo - Apollo Design):

![](https://img-blog.csdnimg.cn/a75ccb863e4a401d947c87bb14af7dc3.png)

1. Người dùng sửa/phát hành cấu hình trong configuration center Apollo.
2. Configuration center Apollo thông báo cấu hình ứng dụng đã thay đổi.
3. Ứng dụng truy cập configuration center Apollo để lấy cấu hình mới nhất.

Sơ đồ kiến trúc chính thức (nguồn ảnh: tài liệu chính thức Apollo - Apollo Design):

![](https://img-blog.csdnimg.cn/79c7445f9dbc45adb45699d40ef50f44.png)

### Mô tả component

| Component          | Tác dụng                                                                                                                                                     | Port mặc định        |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------- |
| **Portal**         | Giao diện quản lý Web, cung cấp khả năng quản lý cấu hình trực quan                                                                                          | 8070                 |
| **Client**         | Client SDK, cung cấp khả năng lấy cấu hình và listen thay đổi                                                                                                | -                    |
| **Meta Server**    | Entry point của service discovery, cùng process với Config Service, cung cấp địa chỉ service cho Client/Portal                                               | 8080                 |
| **Config Service** | Cung cấp interface đọc cấu hình và thông báo long polling cho Client gọi; đồng thời nhúng registry center                                                    | 8080                 |
| **Admin Service**  | Cung cấp interface quản lý cấu hình cho Portal gọi                                                                                                           | 8090                 |
| **Eureka (nhúng)** | Instance registry center được nhúng trong cùng process với Config Service, phục vụ registry discovery của Config/Admin Service; không cần triển khai độc lập | Giống Config Service |
| **MySQL**          | Lưu trữ dữ liệu cấu hình và metadata                                                                                                                         | 3306                 |

Apollo 2.0+ hỗ trợ thay thế implementation của service registry discovery qua SPI, chẳng hạn tích hợp Nacos, Consul hoặc Polaris. Tuy nhiên, trong mô hình triển khai mặc định, Eureka là khả năng được nhúng trong Config Service; không nên hiểu rằng cần vận hành một Eureka cluster bên ngoài độc lập.

### Quy trình cốt lõi

**Phía Client (lấy cấu hình)**:

1. Khi khởi động, Client truy cập Meta Server để lấy danh sách địa chỉ Config Service.
2. Client cache địa chỉ service cục bộ (vẫn có thể dùng khi Eureka gặp sự cố).
3. Client gửi request long polling để lấy cấu hình.
4. Sau khi phát hiện cấu hình thay đổi, Config Service lập tức response.
5. Client cập nhật memory cache, trigger change callback và **bất đồng bộ persist vào local file system**. Thư mục cache mặc định trên Linux/Mac là `/opt/data/{appId}/config-cache/`, trên Windows là `C:\opt\data\{appId}\config-cache\`; cũng có thể tùy chỉnh qua system property `apollo.cache-dir`.

> **Cơ chế disaster recovery**: ngay cả khi toàn bộ Config Service bị down và ứng dụng restart, Client vẫn có thể đọc cấu hình đã cache từ local disk để hoàn tất khởi động, bảo đảm availability của ứng dụng không phụ thuộc cứng vào configuration center.

**Phía Portal (phát hành cấu hình)**:

1. Người dùng sửa cấu hình trong Portal và bấm phát hành.
2. Portal gọi API phát hành của Admin Service.
3. Admin Service ghi cấu hình vào MySQL và tạo version phát hành.
4. Config Service thông báo cấu hình đã thay đổi cho Client qua long polling.
5. Client pull lại cấu hình mới nhất.

### Ví dụ sử dụng Client

Lấy cấu hình:

```java
Config config = ConfigService.getAppConfig();
String someKey = "someKeyFromDefaultNamespace";
String someDefaultValue = "someDefaultValueForTheKey";
String value = config.getProperty(someKey, someDefaultValue);
```

Listen thay đổi cấu hình:

```java
Config config = ConfigService.getAppConfig();
config.addChangeListener(new ConfigChangeListener() {
    @Override
    public void onChange(ConfigChangeEvent changeEvent) {
        // Xử lý thay đổi cấu hình
        for (String key : changeEvent.changedKeys()) {
            ConfigChange change = changeEvent.getChange(key);
            System.out.println(String.format(
                "Key: %s, Old: %s, New: %s",
                key, change.getOldValue(), change.getNewValue()));
        }
    }
});
```

Trong project Spring Boot, production code thường không gọi trực tiếp low-level API ở khắp nơi, mà dùng Spring integration của Apollo để inject và refresh cấu hình. Chẳng hạn dùng `@EnableApolloConfig` để bật Apollo, dùng `@Value`, `@ConfigurationProperties` hoặc `@ApolloConfigChangeListener` để listen thay đổi. Cần lưu ý rằng conditional assembly class (chẳng hạn `@ConditionalOnProperty`) và complex Bean đã khởi tạo xong không nhất thiết tự động được tạo lại khi cấu hình thay đổi; thay đổi cấu hình quan trọng vẫn cần được kiểm chứng cùng refresh strategy của nghiệp vụ.

## Mô hình cốt lõi của Nacos configuration center

Nacos đồng thời cung cấp khả năng configuration center và service discovery; đây cũng là một trong những khác biệt chính giữa Nacos và Apollo. Từ góc nhìn configuration center, Nacos thường dùng mô hình ba tầng để định vị một cấu hình:

- **Namespace**: thường dùng để cô lập môi trường hoặc tenant, chẳng hạn dev, test và prod.
- **Group**: thường dùng cho business domain hoặc application group, mặc định là `DEFAULT_GROUP`.
- **DataId**: identifier của file cấu hình hoặc config item cụ thể, chẳng hạn `order-service.yaml`.

Cần phân biệt hình thức triển khai khi nói về lưu trữ cấu hình của Nacos:

- **Production cluster**: khuyến nghị dùng MySQL bên ngoài để lưu trữ dữ liệu cấu hình; consistency của dữ liệu chủ yếu được bảo đảm bởi high availability solution của MySQL.
- **Storage nhúng**: Nacos cũng hỗ trợ storage nhúng như Derby. Ở cluster mode, Nacos dùng JRaft để tạo logical cluster từ storage nhúng của các node, phù hợp với test, quy mô nhỏ hoặc trường hợp đặc biệt nhạy cảm với chi phí vận hành, nhưng độ phức tạp khi điều tra sự cố cao hơn.

Sau khi Nacos 2.x đưa vào long connection gRPC, connection overhead giữa client và server thấp hơn HTTP long polling của 1.x. Khi cấu hình thay đổi, server thông báo cho client rằng “một cấu hình đã thay đổi”, sau đó client pull nội dung cấu hình mới nhất và callback listener. Cách này tránh đưa trực tiếp nội dung cấu hình lớn vào chain thông báo, đồng thời giúp client thực hiện local snapshot và disaster recovery.

Nacos client cũng duy trì local snapshot. Khi configuration center không khả dụng, client có thể đọc file snapshot/failover cục bộ để tiếp tục khởi động hoặc chạy; path cache cụ thể thay đổi theo version client, namespace, địa chỉ server, Group và DataId. Khi điều tra sự cố, nên căn cứ vào log của client version mục tiêu và thư mục local `nacos/config`.

## Tham khảo

- [Tài liệu chính thức Nacos](https://nacos.io/docs/latest/what-is-nacos/)
- [Triển khai Nacos cluster mode](https://nacos.io/docs/v2.5/manual/admin/deployment/deployment-cluster/)
- [Tài liệu chính thức Apollo](https://www.apolloconfig.com/#/zh/README)
- [Repository Apollo trên GitHub](https://github.com/apolloconfig/apollo)
- [Tài liệu chính thức Spring Cloud Config](https://cloud.spring.io/spring-cloud-config/)
- [Ma trận tương thích version Spring Cloud](https://spring.io/spring-cloud)
- [Tài liệu chính thức Kubernetes ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Nacos 1.1.0 phát hành, hỗ trợ gray configuration](https://nacos.io/zh-cn/blog/nacos%201.1.0.html)
- [Thực tiễn Apollo tại Youzan](https://mp.weixin.qq.com/s/Ge14UeY9Gm2Hrk--E47eJQ)
- [So sánh lựa chọn configuration center cho microservice](https://www.itshangxp.com/spring-cloud/spring-cloud-config-center/)

<!-- @include: @planet.snippet.md -->
