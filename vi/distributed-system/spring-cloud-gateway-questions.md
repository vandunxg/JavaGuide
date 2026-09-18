---
title: "Tổng hợp câu hỏi phỏng vấn Spring Cloud Gateway: route, Predicate, Filter, rate limiting, circuit breaker và nguyên lý hoạt động"
category: Distributed
description: "Tổng hợp các câu hỏi phỏng vấn Spring Cloud Gateway thường gặp, bao quát khái niệm cốt lõi, matching route, Predicate, GatewayFilter, GlobalFilter, rate limiting và circuit breaker, load balancing, xử lý CORS và các vấn đề thường gặp trong production."
tag:
  - API Gateway
  - Spring Cloud
head:
  - - meta
    - name: keywords
      content: Spring Cloud Gateway,Spring Cloud Gateway interview questions,API Gateway,Predicate,GatewayFilter,GlobalFilter,rate limiting tại gateway,circuit breaker tại gateway,microservice gateway
---

> Bài viết này được biên soạn và hoàn thiện dựa trên [6000 chữ | 16 hình | Tìm hiểu sâu nguyên lý của Spring Cloud Gateway - Ngộ Không nói về kiến trúc](https://mp.weixin.qq.com/s/XjFYsP1IUqNzWqXZdJn-Aw).

Bài viết chỉ tập trung giải thích các điểm thường gặp trong phỏng vấn về Spring Cloud Gateway. Nếu bạn chưa hiểu rõ tại sao API Gateway tồn tại, quan hệ giữa gateway và RPC, cũng như cách chọn Zuul / Gateway / Kong / APISIX, hãy đọc trước [Giải thích chi tiết về API Gateway](./api-gateway.md).

## Spring Cloud Gateway là gì?

Spring Cloud Gateway thuộc hệ sinh thái Spring Cloud, ra đời chủ yếu để thay thế **Zuul 1.x**. Zuul 1.x dựa trên kiến trúc Servlet blocking I/O nên hiệu năng có giới hạn trong các tình huống concurrent cao. Zuul 2.x tuy sử dụng kiến trúc Netty non-blocking nhưng Spring Cloud không chính thức tích hợp Zuul 2.x. Spring Cloud Gateway xuất hiện sớm hơn Zuul 2.x.

Để nâng cao performance của gateway, Spring Cloud Gateway dựa trên Spring WebFlux. Spring WebFlux sử dụng thư viện Reactor để triển khai reactive programming model, bên dưới dựa trên Netty để thực hiện I/O synchronous non-blocking.

![](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/api-gateway/springcloud-gateway-%20demo.png)

Spring Cloud Gateway không chỉ cung cấp phương thức routing thống nhất mà còn cung cấp các chức năng cơ bản của gateway dựa trên Filter chain, chẳng hạn như security, monitoring/metrics và rate limiting.

Spring Cloud Gateway và Zuul 2.x không khác nhau nhiều, đều xử lý request thông qua filter. Tuy nhiên, hiện nay Spring Cloud Gateway được khuyến nghị hơn Zuul vì hệ sinh thái Spring Cloud hỗ trợ nó tốt hơn.

- GitHub: <https://github.com/spring-cloud/spring-cloud-gateway>
- Trang chính thức: <https://spring.io/projects/spring-cloud-gateway>

## Spring Cloud Gateway hoạt động như thế nào?

Quy trình hoạt động của Spring Cloud Gateway như hình dưới đây:

![Quy trình hoạt động của Spring Cloud Gateway](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/api-gateway/spring-cloud-gateway-workflow.png)

Đây là hình trong blog chính thức của Spring, xem bài viết gốc tại: <https://spring.io/blog/2022/08/26/creating-a-custom-spring-cloud-gateway-filter>.

Phân tích quy trình:

1. **Xác định route**: Sau khi request của client đến gateway, request trước tiên đi qua Gateway Handler Mapping. Thành phần này kiểm tra assertion (Predicate) để xem request phù hợp với route nào; route đó ánh xạ tới một service backend.
2. **Lọc request**: Sau đó request đến Gateway Web Handler. Tại đây có nhiều filter tạo thành filter chain, có thể intercept và sửa request, chẳng hạn thêm request header, kiểm tra tham số. Cách này hơi giống lọc nước thải. Tiếp đó request được forward tới service backend thực tế. Về mặt logic, các filter này có thể gọi là Pre-Filters. Pre có thể hiểu là “trước...”.
3. **Service xử lý**: Service backend xử lý request.
4. **Lọc response**: Sau khi backend xử lý xong, kết quả được trả về các filter của Gateway để xử lý lần nữa. Về mặt logic, chúng có thể gọi là Post-Filters. Post có thể hiểu là “sau...”.
5. **Trả response**: Sau khi được filter xử lý, response được trả về client.

Tóm lại: request của client trước tiên tìm route phù hợp thông qua các rule matching, từ đó ánh xạ tới service cụ thể. Sau đó request đi qua filter rồi được forward tới service. Service xử lý xong, request lại đi qua filter và cuối cùng được trả về client.

## Assertion của Spring Cloud Gateway là gì?

Từ assertion (Predicate) nghe khá trừu tượng. Có thể hiểu đây là việc kiểm tra một điều kiện của request: nếu kết quả là true thì match route hiện tại, nếu là false thì tiếp tục match các route khác.

Trong Gateway, nếu request do client gửi thỏa mãn điều kiện assertion thì request sẽ ánh xạ tới route được chỉ định và được forward tới service tương ứng để xử lý.

Ví dụ cấu hình assertion dưới đây thiết lập hai route và một cấu hình assertion predicates. Khi URL của request chứa `api/thirdparty`, request sẽ match route đầu tiên `route_thirdparty`.

![Ví dụ cấu hình assertion](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/api-gateway/spring-cloud-gateway-predicate-example.png)

Các rule assertion của route thường gặp như hình dưới đây:

![Các rule assertion của route trong Spring Cloud Gateway](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/api-gateway/spring-cloud-gateway-predicate-rules.png)

## Quan hệ giữa route và assertion trong Spring Cloud Gateway là gì?

Quan hệ tương ứng giữa Route và Predicate như sau:

![Quan hệ tương ứng giữa route và assertion](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/api-gateway/spring-cloud-gateway-predicate-route.png)

- **Một-nhiều**: Một route rule có thể chứa nhiều assertion. Như hình trên, route Route1 được cấu hình ba assertion Predicate.
- **Đồng thời thỏa mãn**: Nếu một route rule có nhiều assertion thì request phải đồng thời thỏa mãn tất cả mới được match. Như hình trên, route Route2 có hai assertion; request của client phải đồng thời thỏa mãn cả hai mới match Route2.
- **Match thành công đầu tiên**: Nếu một request có thể match nhiều route thì request được ánh xạ tới route đầu tiên match thành công. Như hình trên, request của client thỏa mãn assertion của Route3 và Route4, nhưng cấu hình của Route3 đứng trước trong file cấu hình nên chỉ match Route3.

## Spring Cloud Gateway triển khai dynamic route như thế nào?

Khi sử dụng Spring Cloud Gateway, các phương án trong tài liệu chính thức thường dựa trên file cấu hình hoặc cấu hình bằng code.

Là entry point của microservice, Spring Cloud Gateway cần hạn chế restart. Việc thay đổi cấu hình hiện nay phải restart service không đáp ứng được nhu cầu dynamic refresh và thay đổi realtime trong production, nên cần cấu hình gateway động khi Spring Cloud Gateway đang chạy.

Có nhiều cách triển khai dynamic route. Một cách được khuyến nghị là sử dụng Nacos làm service registry. Spring Cloud Gateway có thể lấy metadata của service từ registry, chẳng hạn tên service và path, rồi tự động tạo route rule dựa trên các thông tin này. Khi bạn thêm, xóa hoặc cập nhật service instance, gateway sẽ tự động nhận biết và điều chỉnh route rule tương ứng mà không cần tự duy trì cấu hình route.

Thực tế, không cần tự triển khai thủ công các bước phức tạp này. Có thể dùng Nacos Server và Spring Cloud Alibaba Nacos Config để thực hiện thay đổi cấu hình động. Tài liệu chính thức: <https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-config>.

## Spring Cloud Gateway có những filter nào?

Filter có thể được chia thành hai loại theo request và response:

- **Loại Pre**: Intercept và sửa request trước khi request được forward tới microservice, chẳng hạn kiểm tra tham số, kiểm tra quyền, monitoring traffic, ghi log và chuyển đổi protocol.
- **Loại Post**: Sau khi microservice xử lý request và trả response về gateway, gateway có thể xử lý lần nữa, chẳng hạn sửa nội dung hoặc header của response, ghi log và monitoring traffic.

Một cách phân loại khác là theo phạm vi tác động của filter:

- **GatewayFilter**: Filter cục bộ, áp dụng cho một route hoặc một nhóm route. Màu đỏ biểu thị các filter được sử dụng tương đối phổ biến.
- **GlobalFilter**: Filter toàn cục, áp dụng cho tất cả route.

### Filter cục bộ

Các filter cục bộ thường gặp như hình dưới đây:

![](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/api-gateway/spring-cloud-gateway-gatewayfilters.png)

Cách sử dụng cụ thể như sau: nếu URL match thành công thì xóa `api` khỏi URL.

```yaml
filters: # filter
  - RewritePath=/api/(?<segment>.*),/$\{segment} # Thay "api" trong đường dẫn forward bằng chuỗi rỗng
```

Tất nhiên cũng có thể tự định nghĩa filter, nhưng bài viết này không đi sâu.

### Filter toàn cục

Các filter toàn cục thường gặp như hình dưới đây:

![](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/api-gateway/spring-cloud-gateway-globalfilters.png)

Cách sử dụng phổ biến nhất của GlobalFilter là thực hiện load balancing. Cấu hình như sau:

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: route_member # Rule route của microservice bên thứ ba
          uri: lb://passjava-member # Load balancing, forward request tới service passjava-member đã đăng ký trong registry
          predicates: # assertion
            - Path=/api/member/** # Nếu path request từ frontend chứa api/member thì áp dụng rule route này
          filters: # filter
            - RewritePath=/api/(?<segment>.*),/$\{segment} # Thay api trong đường dẫn forward bằng chuỗi rỗng
```

Ở đây có keyword `lb`, sử dụng GlobalFilter `LoadBalancerClientFilter`. Khi route này được match, request sẽ được forward tới service passjava-member và hỗ trợ load balancing. Tức là trước tiên passjava-member được resolve thành host và port của microservice thực tế, sau đó mới forward tới microservice đó.

## Spring Cloud Gateway có hỗ trợ rate limiting không?

Spring Cloud Gateway có sẵn filter rate limiting, interface tương ứng là `RateLimiter`. Interface `RateLimiter` chỉ có một implementation là `RedisRateLimiter` (dùng Redis + Lua để thực hiện rate limiting), nhưng chức năng rate limiting khá đơn giản và khó sử dụng.

Từ Sentinel phiên bản 1.6.0, Sentinel đã tích hợp module tương thích với Spring Cloud Gateway, cung cấp rate limiting theo hai loại resource: theo route và theo API tùy chỉnh. Nói cách khác, Spring Cloud Gateway có thể kết hợp với Sentinel để thực hiện traffic control mạnh hơn cho gateway.

## Spring Cloud Gateway tùy chỉnh xử lý exception toàn cục như thế nào?

Trong project SpringBoot, để bắt exception toàn cục chỉ cần cấu hình `@RestControllerAdvice` và `@ExceptionHandler`. Tuy nhiên, cách này không phù hợp với Spring Cloud Gateway.

Spring Cloud Gateway cung cấp nhiều cách xử lý toàn cục. Một cách thường dùng là triển khai `ErrorWebExceptionHandler` và override method `handle` của nó.

```java
@Order(-1)
@Component
@RequiredArgsConstructor
public class GlobalErrorWebExceptionHandler implements ErrorWebExceptionHandler {
    private final ObjectMapper objectMapper;

    @Override
    public Mono<Void> handle(ServerWebExchange exchange, Throwable ex) {
    // ...
    }
}
```

## Tham khảo

- Tài liệu chính thức của Spring Cloud Gateway: <https://cloud.spring.io/spring-cloud-gateway/reference/html/>
- Creating a custom Spring Cloud Gateway Filter: <https://spring.io/blog/2022/08/26/creating-a-custom-spring-cloud-gateway-filter>
- Xử lý exception toàn cục: <https://zhuanlan.zhihu.com/p/347028665>

<!-- @include: @article-footer.snippet.md -->
