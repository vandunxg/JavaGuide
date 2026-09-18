---
title: "Giải thích chi tiết API Gateway: chức năng cốt lõi, nguyên lý hoạt động và lựa chọn Spring Cloud Gateway / Kong / APISIX"
category: Hệ thống phân tán
description: "Giải thích chi tiết kiến thức cốt lõi về API Gateway, bao gồm định tuyến request, authentication và authorization, rate limiting và circuit breaker, load balancing, gray release, kiến trúc gateway hai tầng, cùng so sánh các gateway phổ biến như Spring Cloud Gateway, Kong, APISIX và ShenYu."
tag:
  - API Gateway
head:
  - - meta
    - name: keywords
      content: API Gateway,microservice gateway,Spring Cloud Gateway,Kong,APISIX,ShenYu,Zuul,rate limiting và circuit breaker,load balancing,lựa chọn gateway,câu hỏi phỏng vấn gateway
---

## API Gateway là gì?

API Gateway (API Gateway) là **điểm vào thống nhất** nằm giữa client và các backend service. Mọi request từ client đều đi qua gateway trước, sau đó gateway route request đến service đích cụ thể.

Trong nhóm bài viết về hệ thống phân tán này, gateway thuộc lớp “điểm vào traffic”. [RPC](./rpc/) chủ yếu nói về cách các service gọi lẫn nhau; gateway nói về cách route, authentication và authorization, rate limiting, gray release và chuyển đổi protocol sau khi request bên ngoài đi vào hệ thống. Nếu bạn chỉ muốn xem chi tiết về route, Predicate, Filter và rate limiting của Spring Cloud Gateway, có thể đọc tiếp [Tổng hợp câu hỏi phỏng vấn Spring Cloud Gateway](./spring-cloud-gateway-questions.md).

### Giá trị cốt lõi

Trong kiến trúc microservice, một hệ thống được tách thành nhiều service. Các chức năng như **security authentication, traffic control, log, monitoring** là nhu cầu chung của mọi service. Nếu không có gateway, chúng ta phải triển khai riêng các chức năng này trong từng service, dẫn đến:

- **Code trùng lặp**: cùng một logic được triển khai dư thừa ở nhiều service
- **Quản lý phân tán**: thiếu cấu hình thống nhất và góc nhìn monitoring tập trung
- **Chi phí bảo trì cao**: thay đổi chức năng phải sửa tất cả service

![Sơ đồ API Gateway](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/api-gateway-overview.png)

### Trách nhiệm cốt lõi

Gateway có nhiều chức năng, nhưng có thể khái quát thành hai việc:

| Trách nhiệm         | Mô tả                                                        | Chức năng điển hình                                                       |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------------------- |
| **Forward request** | Route request của client đến service đích đúng               | Dynamic routing, load balancing, chuyển đổi protocol                      |
| **Filter request**  | Intercept và xử lý trước/sau khi request đến backend service | Authentication, kiểm tra quyền, rate limiting và circuit breaker, ghi log |

Bảng trên mô tả hai trách nhiệm cốt lõi ở tầng khái quát. Khi triển khai thành các năng lực cụ thể, chúng phát triển thành hơn mười chức năng gateway được liệt kê ở phần tiếp theo.

Gateway có thể cung cấp các chức năng như forward request, security authentication (authentication danh tính/quyền), traffic control, load balancing, degradation và circuit breaker, log, monitoring, kiểm tra parameter, chuyển đổi protocol.

**Vị trí của gateway trong kiến trúc microservice**: mọi request từ client trước tiên đến gateway. Gateway chịu trách nhiệm authentication và authorization thống nhất, traffic control và phân phối route; backend service tập trung xử lý business logic.

### Triển khai high availability

Việc thêm gateway tạo thêm một lần forward qua network (trong môi trường mạng nội bộ, tổn hao performance thường có thể bỏ qua), nhưng đồng thời cũng tạo ra rủi ro single point mới. Vì vậy, bản thân gateway service phải bảo đảm high availability:

Như hình dưới đây, lớp ngoài của gateway service dùng Nginx (hoặc thiết bị/software load balancing khác) để forward traffic và đạt high availability. Khi deploy Nginx cũng cần cân nhắc high availability để tránh single point risk.

![Server-side load balancing dựa trên Nginx](https://oss.javaguide.cn/github/javaguide/high-performance/load-balancing/server-load-balancing.png)

## Gateway có thể cung cấp những chức năng nào?

Phần lớn gateway có thể cung cấp các chức năng sau (một số chức năng cần framework hoặc middleware khác hỗ trợ):

- **Forward request**: forward request đến microservice đích.
- **Load balancing**: dựa trên tải của từng microservice instance hoặc policy load balancing cụ thể để thực hiện load balancing động cho request.
- **Security authentication**: xác thực danh tính của user request và chỉ cho phép client đáng tin cậy truy cập API; đồng thời có thể dùng RBAC hoặc cách tương tự để authorization.
- **Kiểm tra parameter**: hỗ trợ logic mapping và validation parameter.
- **Ghi log**: ghi lại log hành vi của mọi request để sử dụng về sau.
- **Monitoring và alert**: monitoring từ các chỉ số business, chỉ số machine, chỉ số JVM và cung cấp cơ chế alert tương ứng.
- **Traffic control**: kiểm soát traffic của request, tức giới hạn số request trong một khoảng thời gian.
- **Circuit breaker và degradation**: monitoring realtime thông tin thống kê của request; khi đạt ngưỡng failure đã cấu hình thì tự động circuit breaker và trả về giá trị mặc định.
- **Cache response**: khi user request lấy dữ liệu static hoặc ít thay đổi, dữ liệu nhận được sau nhiều request trong một khoảng thời gian có thể giống nhau. Khi đó có thể cache response. User request sẽ nhận response trực tiếp ở gateway mà không cần truy cập business service, giảm tải cho business service.
- **Aggregate response**: trong một số trường hợp, response mà user request cần có thể đến từ nhiều business service. Gateway dùng chung có thể aggregate đơn giản, nhưng aggregation phức tạp nên đặt ở tầng BFF (Backend For Frontend) hoặc GraphQL để tránh đưa business orchestration vào infrastructure layer.
- **Gray release**: dynamic route request đến các version service khác nhau (một dạng gray release cơ bản).
- **Xử lý exception**: với exception response do business service trả về, gateway có thể chuyển đổi trước khi trả cho user. Nhờ đó có thể ẩn chi tiết exception phía business và chuyển thành error message thân thiện với user.
- **Tài liệu API**: nếu dự định expose API cho developer bên ngoài tổ chức, cần cân nhắc dùng tài liệu API như Swagger hoặc OpenAPI.
- **Chuyển đổi protocol**: tích hợp các microservice khác nhau về style và công nghệ triển khai, dựa trên REST, AMQP, Dubbo..., thông qua chuyển đổi protocol; cung cấp service thống nhất cho các client cụ thể như Web/Mobile và open platform.
- **Quản lý certificate**: deploy SSL certificate lên API Gateway, quản lý interface từ một entry point thống nhất để giảm độ phức tạp khi thay certificate.

Cần lưu ý rằng gateway không phù hợp để chứa mọi logic. Việc validation business rule phức tạp, mapping field phức tạp, long transaction, business logic của long connection và business authorization chi tiết thường nên được xử lý ở business service, BFF hoặc GraphQL. Gateway phù hợp hơn với năng lực ở tầng protocol, năng lực dùng chung và liên service, tránh phát triển thành “monolithic gateway” khó bảo trì.

Hình dưới đây lấy từ bài viết [Thiết kế và triển khai API Gateway Shepherd quy mô hàng chục tỷ request - Nhóm kỹ thuật Meituan - 2021](https://mp.weixin.qq.com/s/iITqdIiHi3XGKq6u6FRVdg).

![](https://oss.javaguide.cn/github/javaguide/distributed-system/api-gateway/up-35e102c633bbe8e0dea1e075ea3fee5dcfb.png)

## Có những gateway system phổ biến nào?

### Netflix Zuul

Zuul là gateway service do Netflix phát triển, cung cấp dynamic routing, monitoring, resilience và security. Zuul được phát triển trên Java và có thể phối hợp với các component như Eureka, Ribbon và Hystrix.

Kiến trúc cốt lõi của Zuul như sau:

![Kiến trúc cốt lõi của Zuul](https://oss.javaguide.cn/github/javaguide/distributed-system/api-gateway/zuul-core-architecture.webp)

Zuul chủ yếu dùng filter (tương tự AOP) để filter request, từ đó triển khai các chức năng cần có của gateway.

![Vòng đời request của Zuul](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/api-gateway/zuul-request-lifecycle.webp)

Chúng ta có thể tự định nghĩa filter để xử lý request. Hệ sinh thái Zuul cũng có nhiều filter có sẵn để sử dụng. Ví dụ, rate limiting có thể dùng extension cộng đồng [spring-cloud-zuul-ratelimit](https://github.com/marcosbarbero/spring-cloud-zuul-ratelimit) (chỉ nhằm minh họa). Cần phân biệt rằng Hystrix chủ yếu phụ trách circuit breaker, timeout, degradation và isolation bằng thread pool/semaphore, không phải component rate limiting QPS theo nghĩa chặt.

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
<dependency>
    <groupId>com.marcosbarbero.cloud</groupId>
    <artifactId>spring-cloud-zuul-ratelimit</artifactId>
    <version>2.2.0.RELEASE</version>
</dependency>
```

[Zuul 1.x](https://netflixtechblog.com/announcing-zuul-edge-service-in-the-cloud-ab3af5be08ee) dựa trên synchronous IO nên performance kém. [Zuul 2.x](https://netflixtechblog.com/open-sourcing-zuul-2-82ea476cb2b3) dùng Netty để triển khai asynchronous IO, giúp performance được cải thiện đáng kể.

![Kiến trúc Zuul2](https://oss.javaguide.cn/github/javaguide/distributed-system/api-gateway/zuul2-core-architecture.png)

> **Lưu ý quan trọng**: các module Zuul 1.x, Ribbon và Hystrix trong Spring Cloud Netflix đã chuyển sang maintenance mode; các bản phát hành chính của Spring Cloud cũng đã chuyển sang Spring Cloud Gateway từ lâu. Dù Netflix đã open source Zuul 2.x, Zuul 2.x chưa được tích hợp vào các version chính của Spring Cloud. Với project mới dùng technology stack Spring Cloud, không nên tiếp tục chọn Zuul 1.x; project đang vận hành nên dần migrate sang Spring Cloud Gateway hoặc gateway hiện đại khác theo từng đợt refactor.

- GitHub: <https://github.com/Netflix/zuul>
- Wiki chính thức: <https://github.com/Netflix/zuul/wiki>

### Spring Cloud Gateway

Spring Cloud Gateway là gateway trong hệ sinh thái Spring Cloud, được tạo ra với mục tiêu thay thế gateway lâu đời **Zuul** (chính xác là Zuul 1.x). Cần lưu ý rằng Spring Cloud Gateway bắt đầu phát triển trước Zuul 2.x; hai bên đi theo các hướng tiến hóa technology khác nhau.

#### Vì sao Spring Cloud Gateway có performance tốt hơn?

| Version                  | Mô hình IO                        | Mô hình thread             | Throughput | Latency |
| ------------------------ | --------------------------------- | -------------------------- | ---------- | ------- |
| **Zuul 1.x**             | Synchronous blocking (Servlet)    | Một thread cho mỗi request | Thấp       | Cao     |
| **Zuul 2.x**             | Asynchronous non-blocking (Netty) | Event loop                 | Cao        | Thấp    |
| **Spring Cloud Gateway** | Asynchronous non-blocking (Netty) | Event loop                 | Cao        | Thấp    |

Spring Cloud Gateway dựa trên **Spring WebFlux**, không phải Spring Web MVC truyền thống. Spring WebFlux dùng thư viện **Reactor** để triển khai mô hình reactive programming; tầng dưới dựa trên **Netty** để thực hiện I/O asynchronous non-blocking.

**Ưu điểm của reactive programming**:

- **Non-blocking I/O**: không cần cấp thread riêng cho từng request, chỉ cần ít thread cũng có thể xử lý nhiều connection đồng thời.
- **Cơ chế backpressure**: backpressure của Reactor chủ yếu tác động lên reactive chain bên trong gateway process, tránh để request in-flight làm quá tải chính process đó. Overload protection end-to-end vẫn cần rate limiting, bulkhead, timeout và circuit breaker tường minh, ví dụ các filter `RequestRateLimiter`, `CircuitBreaker`.
- **Hiệu quả sử dụng resource cao**: chi phí context switch của thread giảm đáng kể.

#### Khái niệm cốt lõi

Các component cốt lõi của Spring Cloud Gateway gồm ba phần:

1. **Route (route)**: building block cơ bản của gateway, gồm ID, target URI, tập predicate và tập filter.
2. **Predicate**: được triển khai dựa trên functional interface `Predicate` của Java 8, dùng để match HTTP request (như path, method, request header...). Ví dụ `Path=/api/users/**`, `Method=GET`, `Header=X-Request-Id, \d+`; nhiều Predicate được kết hợp bằng phép AND logic.
3. **Filter**: instance của `GatewayFilter`, dùng để sửa request và response trước hoặc sau khi request được gửi đến downstream service.

Spring Cloud Gateway và Zuul 2.x đều xử lý request thông qua filter, nhưng Spring Cloud Gateway tích hợp chặt chẽ hơn với hệ sinh thái Spring (như Eureka, Consul và Config). Hiện nay, với project mới dùng Java, Spring Cloud Gateway thường là lựa chọn phổ biến hơn. Cần lưu ý rằng tài liệu Spring Cloud Gateway 4.x/5.x đã đồng thời cung cấp hai dạng Server và Proxy Exchange, với path tương thích riêng cho WebFlux và Web MVC. Team quen với technology stack Servlet và tạm thời không muốn đưa thêm độ phức tạp của reactive programming vào có thể đánh giá Spring Cloud Gateway Server MVC.

Về ranh giới năng lực, cần nói rõ rằng Spring Cloud Gateway có filter `RequestRateLimiter` tích hợp sẵn; implementation Redis phổ biến dựa trên token bucket, nhưng rate limiting bằng Redis vẫn cần thêm reactive Redis dependency tương ứng. Năng lực circuit breaker được cung cấp thông qua Spring Cloud CircuitBreaker để thích ứng với Resilience4j, cần thêm các dependency như `spring-cloud-starter-circuitbreaker-reactor-resilience4j`. Cấu hình route cũng không chỉ là “cấu hình trong memory”: mặc định có thể viết trong YAML, hoặc kết nối với external configuration thông qua `RouteDefinitionRepository`, Redis route repository hay implementation tự định nghĩa, rồi kết hợp cơ chế refresh để có hiệu lực động.

- GitHub: <https://github.com/spring-cloud/spring-cloud-gateway>
- Website: <https://spring.io/projects/spring-cloud-gateway>

### OpenResty

Theo giới thiệu chính thức:

> OpenResty là một nền tảng Web performance cao dựa trên Nginx và Lua, tích hợp nhiều thư viện Lua được hoàn thiện, module bên thứ ba và phần lớn dependency. Nền tảng này giúp xây dựng thuận tiện các dynamic Web application, Web service và dynamic gateway có thể xử lý concurrency cực cao và khả năng mở rộng rất lớn.

![Mối quan hệ giữa OpenResty, Nginx và Lua](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/api-gatewaynginx-lua-openresty.png)

OpenResty dựa trên Nginx, chủ yếu vì năng lực xử lý concurrency cao của Nginx. Tuy nhiên, do Nginx được phát triển bằng C nên ngưỡng phát triển thứ cấp khá cao. Nếu muốn triển khai logic hoặc chức năng custom trên Nginx, cần viết module bằng C và compile lại Nginx.

Để giải quyết vấn đề này, OpenResty tích hợp và duy trì các module như `ngx_http_lua_module`, `ngx_stream_lua_module`, đưa Lua/LuaJIT vào Nginx. Nhờ đó có thể nhúng Lua script bên trong Nginx và mở rộng chức năng gateway bằng Lua đơn giản, chẳng hạn triển khai custom route rule, filter và cache policy.

> Lua là một dynamic scripting language rất nhanh, tốc độ thực thi gần với C. LuaJIT là một just-in-time compiler của Lua, có thể cải thiện đáng kể hiệu suất thực thi Lua code. LuaJIT precompile và cache một số hàm Lua và utility library thường dùng, nhờ đó lần gọi sau có thể dùng trực tiếp bytecode đã cache và tăng đáng kể tốc độ thực thi.

Về nhập môn OpenResty và thực hành bảo mật gateway, nên đọc bài viết [Nhập môn OpenResty và thực hành bảo mật gateway mà mọi backend developer nên biết](https://mp.weixin.qq.com/s/3HglZs06W95vF3tSa3KrXw).

- GitHub: <https://github.com/openresty/openresty>
- Website: <https://openresty.org/>

### Kong

Kong là gateway system performance cao, cloud-native, có khả năng mở rộng và hệ sinh thái phong phú, dựa trên [OpenResty](https://github.com/openresty/) (Nginx + Lua). Kong chủ yếu gồm 3 component:

- Kong Server: server dựa trên Nginx, dùng để nhận API request.
- PostgreSQL: dùng để lưu operation data (mô hình database truyền thống). Kong trước đây cũng hỗ trợ Cassandra, nhưng từ Kong Gateway 3.4 đã loại bỏ hỗ trợ Cassandra DB; không nên chọn Cassandra cho deployment mới.
- Kong Manager: công cụ UI quản lý chính thức, cung cấp chức năng quản lý, monitoring và cấu hình API trực quan (có bản OSS open source và bản Enterprise). Cũng có thể quản lý bằng RESTful Admin API.

![](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/api-gateway/kong-way.webp)

Mode truyền thống của Kong phụ thuộc external database để lưu configuration, kiến trúc tương đối phức tạp và cần bảo đảm thêm high availability cho database layer. Nhưng từ version **Kong 1.1**, Kong đã hỗ trợ **DB-less mode (mode không database)**:

- **Mode truyền thống**: dùng PostgreSQL lưu configuration, phù hợp với trường hợp cần quản lý API data persistent qua Admin API.
- **DB-less mode**: dùng file configuration dạng khai báo để quản lý, không cần deploy database, kiến trúc nhẹ hơn.
- **Hybrid mode**: control plane dùng database quản lý configuration, data plane không kết nối trực tiếp với database, phù hợp với việc phân phối configuration giữa nhiều cluster và region.
- **Kubernetes Ingress mode**: dùng ConfigMap hoặc CRD (Kubernetes Custom Resource Definitions) để quản lý configuration, không cần database; đây là cách dùng phổ biến trong môi trường K8s.

> **Lưu ý**: phần thảo luận sau đây về vấn đề high availability của Kong chủ yếu nhắm đến mode database truyền thống. Khi dùng mode Ingress Controller trong môi trường K8s, hoặc dùng DB-less/Hybrid mode, kiến trúc và trọng tâm vận hành sẽ khác rõ rệt.

Kong cung cấp cơ chế plugin để mở rộng chức năng. Plugin được thực thi trong lifecycle của vòng lặp request-response API. Ví dụ, bật plugin Zipkin trên service:

```shell
$ curl -X POST http://kong:8001/services/{service}/plugins \
    --data "name=zipkin"  \
    --data "config.http_endpoint=http://your.zipkin.collector:9411/api/v2/spans" \
    --data "config.sample_ratio=0.001"
```

Bản thân Kong là một Lua application, được bọc thêm một lớp trên nền OpenResty. Về bản chất, Kong dùng Lua nhúng vào Nginx để trao cho Nginx khả năng lập trình, nhờ đó có thể làm rất nhiều việc ở tầng Nginx dưới dạng plugin, chẳng hạn rate limiting, security access policy, route và load balancing. Để viết Kong plugin, cần tuân theo specification viết plugin của Kong, viết Lua script custom, load script vào Kong rồi reference nó.

![](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/api-gateway/kong-gateway-overview.png)

Ngoài Lua, Kong còn hỗ trợ phát triển plugin bằng Go, JavaScript, Python và các ngôn ngữ khác nhờ PDK (Plugin Development Kit) tương ứng.

Về giới thiệu chi tiết Kong plugin, nên đọc [tài liệu chính thức](https://docs.konghq.com/gateway/latest/kong-plugins/), nội dung khá đầy đủ.

- GitHub: <https://github.com/Kong/kong>
- Website: <https://konghq.com/kong>

### APISIX

APISIX là gateway system performance cao, cloud-native và có khả năng mở rộng, dựa trên OpenResty và etcd.

> etcd là một hệ thống lưu trữ Key-Value phân tán, high availability và open source, được phát triển bằng Go, dùng protocol Raft để thực hiện distributed consensus.

So với API Gateway truyền thống, APISIX có dynamic routing và hot loading plugin, đặc biệt phù hợp với API management trong hệ thống microservice. Ngoài ra, APISIX cũng dễ dàng kết nối với các công cụ DevOps như SkyWalking (hệ thống distributed tracing), Zipkin (hệ thống distributed tracing) và Prometheus (hệ thống monitoring).

![Sơ đồ kiến trúc APISIX](https://oss.javaguide.cn/github/javaguide/distributed-system/api-gateway/apisix-architecture.png)

Là project thay thế Nginx và Kong, APISIX đã trở thành Apache top-level project vào ngày 2020-07-15. Hiện nay có nhiều doanh nghiệp nổi tiếng tại Trung Quốc (như Kingsoft, Youzan, iQIYI, Tencent và KE Holdings) dùng APISIX để xử lý business traffic cốt lõi.

APISIX cạnh tranh mạnh với Kong ở dynamic routing, hot update configuration dựa trên etcd, hệ sinh thái plugin và console. Tuy nhiên, performance của API Gateway phụ thuộc rất nhiều vào version, deployment topology, số lượng route, độ dài plugin chain, kích thước request body, việc bật TLS và cách reuse connection. Bạn nên benchmark theo scenario của mình, không nên áp dụng nguyên văn thông điệp marketing của bất kỳ vendor nào.

APISIX cũng hỗ trợ phát triển plugin custom. Ngoài Lua, developer còn có thể dùng hai cách sau để tránh chi phí học Lua:

- Dùng Plugin Runner để hỗ trợ thêm nhiều ngôn ngữ lập trình phổ biến (như Java, Python và Go). Nhờ cách này, backend engineer có thể dùng local RPC communication và ngôn ngữ quen thuộc để phát triển plugin APISIX. Ưu điểm là giảm cost development và tăng efficiency, nhưng sẽ có một phần tổn hao performance.
- Dùng Wasm (WebAssembly) để phát triển plugin. Wasm được nhúng vào APISIX; user có thể dùng Wasm compile thành Wasm bytecode rồi chạy trong APISIX.

> Wasm là binary instruction format của virtual machine dựa trên stack, một low-level assembly language, được thiết kế để rất gần machine code đã compile và performance native. Wasm ban đầu được xây dựng cho browser, nhưng cùng với sự trưởng thành của technology, ngày càng có nhiều use case ở phía server.

![](https://oss.javaguide.cn/github/javaguide/distributed-system/api-gateway/up-a240d3b113cde647f5850f4c7cc55d4ff5c.png)

- GitHub: <https://github.com/apache/apisix>
- Website: <https://apisix.apache.org/zh/>

### ShenYu

ShenYu là một reactive gateway có khả năng mở rộng và performance cao, dựa trên WebFlux, đồng thời là Apache top-level open source project.

![Kiến trúc ShenYu](https://oss.javaguide.cn/github/javaguide/distributed-system/api-gateway/shenyu-architecture.png)

ShenYu mở rộng chức năng thông qua plugin. Plugin là phần cốt lõi của ShenYu, đồng thời có thể mở rộng và hot plug. Mỗi plugin triển khai một chức năng khác nhau. ShenYu tích hợp sẵn các plugin như rate limiting, circuit breaker, forward, rewrite, redirect và monitoring route.

- GitHub: <https://github.com/apache/shenyu>
- Website: <https://shenyu.apache.org/>

### Bảng so sánh gateway

| Đặc tính                             | Zuul 1.x                                                           | Zuul 2.x                       | Spring Cloud Gateway                                                                    | Kong                                 | APISIX                        | ShenYu                           |
| ------------------------------------ | ------------------------------------------------------------------ | ------------------------------ | --------------------------------------------------------------------------------------- | ------------------------------------ | ----------------------------- | -------------------------------- |
| **Mô hình IO**                       | Synchronous blocking                                               | Asynchronous non-blocking      | Asynchronous non-blocking                                                               | Asynchronous non-blocking            | Asynchronous non-blocking     | Asynchronous non-blocking        |
| **Technology tầng dưới**             | Servlet                                                            | Netty                          | WebFlux/Server MVC + Netty/Servlet container                                            | OpenResty (Nginx + Lua)              | OpenResty + etcd              | WebFlux + Netty                  |
| **Performance**                      | Thấp                                                               | Cao                            | Cao                                                                                     | Rất cao                              | Rất cao                       | Cao                              |
| **Dynamic configuration**            | Mặc định cần restart, có thể mở rộng                               | Có, cần tự xây dựng hệ thống   | Có                                                                                      | Có                                   | Có (hot update)               | Có                               |
| **Nơi lưu configuration**            | Local configuration/memory                                         | Custom                         | Mặc định YAML/memory, có thể kết nối Redis hoặc route repository custom                 | PostgreSQL / YAML / K8s CRD          | etcd (phân tán)               | Memory/database/service registry |
| **Rate limiting và circuit breaker** | Rate limiting dựa trên extension, circuit breaker dựa trên Hystrix | Cần tích hợp                   | Rate limiting tích hợp `RequestRateLimiter`; circuit breaker cần kết nối CircuitBreaker | Plugin                               | Plugin                        | Plugin                           |
| **Hệ sinh thái**                     | Netflix (maintenance mode)                                         | Netflix                        | Spring Cloud                                                                            | Kong / hệ sinh thái plugin           | Apache                        | Apache                           |
| **Độ phức tạp vận hành**             | Thấp                                                               | Trung bình                     | Thấp đến trung bình                                                                     | Trung bình (DB-less) / cao (DB Mode) | Trung bình                    | Trung bình                       |
| **Learning curve**                   | Thoải                                                              | Thoải                          | Thoải đến trung bình                                                                    | Dốc (Lua)                            | Dốc (Lua)                     | Thoải (Java)                     |
| **Scenario phù hợp**                 | Bảo trì hệ thống legacy                                            | Hệ thống Netflix đang vận hành | Hệ sinh thái Spring Cloud                                                               | Cloud-native, đa ngôn ngữ            | Cloud-native, performance cao | Hệ sinh thái Java                |

“Performance” trong bảng trên chỉ là phân loại theo kinh nghiệm, không thể thay thế benchmark. Performance thực tế chịu ảnh hưởng của số lượng route, độ dài plugin chain, kích thước request body, việc bật TLS, tỷ lệ reuse connection, tỷ lệ sampling log, latency downstream và các yếu tố khác. Trước khi chọn, nên dùng các tool như wrk2, vegeta và k6 để đo lại theo traffic profile của mình, đồng thời tập trung quan sát latency P99/P999, error rate và mức sử dụng CPU/memory thay vì chỉ nhìn QPS đơn thuần.

## Lựa chọn như thế nào?

Khi chọn API Gateway, cần cân nhắc tổng hợp technology stack, yêu cầu performance, năng lực của team và cost vận hành.

### Kiến trúc gateway hai tầng

Trong hệ thống microservice vừa và lớn, cách làm phổ biến là tách gateway thành hai tầng:

- **Traffic gateway**: thường deploy ở edge layer, có thể chọn Kong, APISIX, Nginx/OpenResty, Envoy Gateway...; phụ trách các năng lực thiên về infrastructure như SSL termination, WAF, global rate limiting, IP allowlist/denylist, DDoS protection và protocol adaptation.
- **Business gateway**: thường deploy trong internal network, có thể chọn Spring Cloud Gateway, ShenYu hoặc gateway tự phát triển; phụ trách các năng lực gần business system hơn như microservice routing, authorization chi tiết, validation parameter, gray routing và business-side observability.

Nếu hệ thống có quy mô nhỏ, chỉ một business domain và team nhân lực hạn chế, gateway một tầng thường đơn giản hơn. Khi trách nhiệm traffic governance bên ngoài và business routing bên trong bắt đầu ảnh hưởng lẫn nhau, tách thành hai tầng sẽ thận trọng hơn. Nếu team đã sử dụng Service Mesh toàn diện, cũng có thể đánh giá việc dùng trực tiếp Envoy/Istio Ingress làm traffic entry point north-south để tránh có quá nhiều proxy layer.

| Scenario                           | Giải pháp đề xuất                                                             | Lý do                                                                                                                                          |
| ---------------------------------- | ----------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Hệ sinh thái Spring Cloud**      | Spring Cloud Gateway                                                          | Tích hợp liền mạch với Spring Boot/Spring Cloud, configuration đơn giản                                                                        |
| **Performance cao / cloud-native** | APISIX                                                                        | Hot update dựa trên etcd, performance tốt, kiến trúc cloud-native                                                                              |
| **Hệ sinh thái đa ngôn ngữ**       | Kong                                                                          | Plugin phong phú, hỗ trợ phát triển đa ngôn ngữ, community trưởng thành                                                                        |
| **Hệ thống Netflix đang vận hành** | Giữ hiện trạng và lập kế hoạch migrate                                        | Zuul 1.x, Ribbon, Hystrix... đã vào maintenance mode; không nên tiếp tục mở rộng trong project mới                                             |
| **Kiến trúc hai tầng**             | Kong/APISIX/Envoy (traffic gateway) + Spring Cloud Gateway (business gateway) | Traffic gateway xử lý SSL, WAF, global rate limiting; business gateway xử lý authentication microservice, validation parameter và gray routing |

## Tham khảo

- Tài liệu chính thức Spring Cloud Gateway: <https://docs.spring.io/spring-cloud-gateway/reference/>
- Giải thích maintenance mode của Spring Cloud Netflix: <https://cloud.spring.io/spring-cloud-netflix/multi/multi__modules_in_maintenance_mode.html>
- Kong Gateway 3.4 Breaking Changes: <https://docs.konghq.com/gateway/latest/breaking-changes/34x/>
- ASF công bố Apache APISIX trở thành top-level project: <https://news.apache.org/foundation/entry/the-apache-software-foundation-announces66>
- RFC 9113 HTTP/2: <https://www.ietf.org/rfc/rfc9113.html>
- Hướng dẫn phát triển Kong plugin [dễ hiểu]: <https://cloud.tencent.com/developer/article/2104299>
- Thực hành API Gateway Kong: <https://xie.infoq.cn/article/10e4dab2de0bdb6f2c3c93da6>
- Giới thiệu nguyên lý và ứng dụng Spring Cloud Gateway: <https://blog.fintopia.tech/60e27b0e2078082a378ec5ed/>
- Vì sao microservice cần API Gateway?: <https://apisix.apache.org/zh/blog/2023/03/08/why-do-microservices-need-an-api-gateway/>

<!-- @include: @article-footer.snippet.md -->
