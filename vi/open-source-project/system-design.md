---
title: Các framework mã nguồn mở và dự án hạ tầng Java chất lượng
description: Đề xuất các framework mã nguồn mở và dự án hạ tầng Java chất lượng, bao quát Web framework, microservice, database, message queue, observability và các component high availability.
category: Dự án mã nguồn mở
icon: "mdi:palette-swatch-outline"
---

Trang này tập hợp các framework, middleware và hạ tầng được dùng làm dependency của ứng dụng hoặc deploy độc lập. Các hệ thống có đầy đủ chức năng nghiệp vụ được đặt trong [dự án thực hành](./practical-project.md), còn các dependency Java nhỏ dùng chung được đặt trong [thư viện tiện ích](./tool-library.md).

## Framework nền tảng

### Web framework

- [Spring Boot](https://github.com/spring-projects/spring-boot "spring-boot"): Dùng để xây dựng các ứng dụng Spring độc lập, có thể chạy được, cung cấp các khả năng như auto-configuration, Web server nhúng, giám sát production và externalized configuration.
- [Solon](https://gitee.com/opensolon/solon): Framework phát triển ứng dụng enterprise Java trong nước, hướng đến mọi trường hợp sử dụng.
- [Javalin](https://github.com/tipsy/javalin): Một Web framework lightweight, đồng thời hỗ trợ Java và Kotlin, được Microsoft, Red Hat, Uber và các công ty khác sử dụng.
- [Play Framework](https://github.com/playframework/playframework): Web framework tốc độ cao dành cho Java và Scala.
- [Blade](https://github.com/lets-blade/blade): Web framework hướng đến sự đơn giản và hiệu quả, dựa trên Java8 + Netty4.

### Microservice / cloud-native

- [Armeria](https://github.com/line/armeria): Phù hợp để xây dựng microservice, hỗ trợ các công nghệ như [gRPC](https://grpc.io/), [Thrift](https://thrift.apache.org/), [Kotlin](https://kotlinlang.org/), [Retrofit](https://square.github.io/retrofit/), [Spring Boot](https://spring.io/projects/spring-boot) và [Dropwizard](https://www.dropwizard.io/).
- [Quarkus](https://github.com/quarkusio/quarkus): Framework Java dành cho môi trường cloud-native và container, hỗ trợ khởi động nhanh, mức sử dụng memory thấp hơn và native image.
- [Helidon](https://github.com/helidon-io/helidon): Một nhóm thư viện Java dùng để viết microservice, hỗ trợ hai programming model là Helidon MP và Helidon SE.

### Tài liệu API

- [Swagger Core](https://github.com/swagger-api/swagger-core): Bản triển khai Swagger bằng Java, dùng annotation và model để tạo tài liệu OpenAPI.
- [springdoc-openapi](https://github.com/springdoc/springdoc-openapi): Tạo tài liệu OpenAPI 3 cho ứng dụng Spring Boot và cung cấp tích hợp Swagger UI. Trước khi sử dụng cần xác nhận version tương thích với version Spring Boot hiện tại.

### Mapping Bean

- [MapStruct](https://github.com/mapstruct/mapstruct) (khuyến nghị): Java annotation processor tuân thủ đặc tả JSR 269, tạo code mapping Bean type-safe ở giai đoạn compile, không phụ thuộc runtime reflection.
- [MapStruct Plus](https://github.com/linpeilie/mapstruct-plus): Phiên bản nâng cao của MapStruct, hỗ trợ tự động tạo interface Mapper.

### Khác

- [Guice](https://github.com/google/guice): Một dependency injection framework lightweight mã nguồn mở của Google, tương đương một Spring Boot lightweight với chức năng được tinh giản tối đa. Trong một số trường hợp, nó rất hữu ích, chẳng hạn khi dự án chỉ cần dependency injection mà không cần các tính năng như AOP.
- [Spring Batch](https://github.com/spring-projects/spring-batch): Batch framework hướng đến việc đọc, xử lý và ghi một lượng lớn record. Nó phụ trách execution model của job và step, không phải scheduling framework như Quartz hay XXL-JOB.

## Authentication và authorization

### Authentication quyền hạn

- [Sa-Token](https://github.com/dromara/sa-token): Framework authentication quyền hạn Java lightweight, tích hợp sẵn authentication, authorization, single sign-on, kick user offline và tự động gia hạn, có API style trực tiếp hơn Spring Security.
- [Spring Security](https://github.com/spring-projects/spring-security): Security framework chính thức của Spring, có thể dùng cho identity verification, authorization, encryption và session management, hiện là Java security framework được sử dụng rộng rãi nhất.
- [Shiro](https://github.com/apache/shiro): Java security framework, có chức năng tương tự Spring Security nhưng dễ sử dụng hơn.

### Đăng nhập bên thứ ba

- [WxJava](https://github.com/Wechat-Group/WxJava): Java SDK phát triển WeChat, hỗ trợ các trường hợp sử dụng như WeChat Pay, Open Platform, Mini Program, WeCom và Official Account.
- [pac4j](https://github.com/pac4j/pac4j): Java security engine, hỗ trợ các protocol như OpenID Connect, OAuth, SAML, CAS, LDAP và JWT, có thể tích hợp nhiều Web framework.

### Single sign-on (SSO)

- [CAS](https://github.com/apereo/cas): Giải pháp single sign-on trên network đa ngôn ngữ dành cho enterprise.
- [MaxKey](https://gitee.com/dromara/MaxKey): Hệ thống authentication single sign-on, cung cấp các chức năng quản lý identity user (IDM), identity authentication (AM), single sign-on (SSO), quản lý quyền RBAC và quản lý resource an toàn, theo chuẩn và mở.
- [Keycloak](https://github.com/keycloak/keycloak): Hệ thống identity authentication và access management miễn phí, mã nguồn mở, hỗ trợ chức năng single sign-on có khả năng cấu hình cao.

## Network communication

- [Netty](https://github.com/netty/netty): Network framework asynchronous event-driven dựa trên NIO, dùng để phát triển các ứng dụng network như TCP, UDP và HTTP.
- [Retrofit](https://github.com/square/retrofit): HTTP client type-safe phù hợp cho Android và Java. HTTP request của Retrofit sử dụng thư viện [OkHttp](https://square.github.io/okhttp/) (một network framework được sử dụng rộng rãi).
- [Forest](https://gitee.com/dromara/forest): Java HTTP client framework declarative, gọi REST API bên thứ ba thông qua interface và annotation, có thể là lựa chọn ngoài Retrofit và OpenFeign.
- [OpenFeign](https://github.com/OpenFeign/feign): Java HTTP client declarative, mô tả remote call thông qua interface và annotation, phù hợp cho việc gọi HTTP API giữa các service.
- [gRPC-Java](https://github.com/grpc/grpc-java): Bản triển khai gRPC bằng Java, dựa trên HTTP/2 và Protocol Buffers, phù hợp cho các trường hợp RPC cần interface type-safe và stream communication.

## Database

### Database connection pool

- [Druid](https://github.com/alibaba/druid): Database connection pool JDBC có khả năng monitoring.
- [HikariCP](https://github.com/brettwooldridge/HikariCP): JDBC connection pool lightweight, hiệu năng cao, đồng thời là implementation connection pool mặc định của Spring Boot.

### Database framework

- [MyBatis-Plus](https://github.com/baomidou/mybatis-plus): Công cụ nâng cao cho [MyBatis](https://mybatis.org/mybatis-3/), cung cấp các khả năng như CRUD dùng chung, condition builder, pagination và tạo code.
- [MyBatis-Flex](https://gitee.com/mybatis-flex/mybatis-flex): Framework nâng cao cho MyBatis, hỗ trợ CRUD, pagination query, multi-table query và batch operation.
- [jOOQ](https://github.com/jOOQ/jOOQ): Cách tốt nhất để viết SQL bằng Java.
- [Redisson](https://github.com/redisson/redisson "redisson"): Redisson là một Java in-memory data grid (In-Memory Data Grid) đặt trên Redis, tận dụng đầy đủ ưu thế của Redis key-value database để cung cấp cho Java developer một loạt utility class phổ biến mang tính distributed. Ví dụ: các Java object distributed (`Set`, `SortedSet`, `Map`, `List`, `Queue`, `Deque`, v.v.), distributed lock, v.v. Xem chi tiết tại: [giới thiệu dự án Redisson](https://github.com/redisson/redisson/wiki/Redisson%E9%A1%B9%E7%9B%AE%E4%BB%8B%E7%BB%8D).

### Data synchronization

- [Canal](https://github.com/alibaba/canal "canal"): Cung cấp incremental data subscription và consumption thông qua việc phân tích MySQL Binlog, thường dùng cho cache synchronization, data heterogeneity và xử lý change event.
- [DataX](https://github.com/alibaba/DataX "DataX"): DataX là công cụ/nền tảng data synchronization offline được sử dụng rộng rãi trong Tập đoàn Alibaba, thực hiện data synchronization hiệu quả giữa nhiều heterogeneous data source như MySQL, Oracle, SqlServer, Postgre, HDFS, Hive, ADS, HBase, TableStore(OTS), MaxCompute(ODPS), DRDS, v.v.

### Time-series database

- [IoTDB](https://github.com/apache/iotdb): Time-series database trong nước được viết bằng Java, cung cấp cho user các dịch vụ thu thập, lưu trữ và phân tích data. Tích hợp liền mạch với Hadoop, Spark và các công cụ visualization (như Grafana), đáp ứng nhu cầu lưu trữ lượng lớn data, ghi data throughput cao và query, phân tích data phức tạp trong lĩnh vực industrial IoT.
- [QuestDB](https://github.com/questdb/questdb): Time-series database mã nguồn mở hướng đến throughput ghi cao và SQL query latency thấp, cung cấp các phương thức kết nối như PostgreSQL wire protocol, REST và InfluxDB Line Protocol.

## Search engine

- [Elasticsearch](https://github.com/elastic/elasticsearch "elasticsearch") (khuyến nghị): Search engine mã nguồn mở, distributed, RESTful.
- [Meilisearch](https://github.com/meilisearch/meilisearch): Search engine mạnh, nhanh, mã nguồn mở, dễ sử dụng và deploy, hỗ trợ Chinese search (không cần thêm configuration).
- [Solr](https://github.com/apache/solr): Distributed search platform được xây dựng trên Apache Lucene.

## Testing

### Testing framework

- [JUnit 5](https://github.com/junit-team/junit5): Unit testing framework thường dùng trong hệ sinh thái Java và JVM.
- [Mockito](https://github.com/mockito/mockito): Java Mock testing framework, có thể dùng test double để cô lập các object khó tạo hoặc phụ thuộc hệ thống bên ngoài.
- [AssertJ](https://github.com/assertj/assertj): Thư viện fluent assertion dành cho Java và JVM, phù hợp để nâng cao khả năng dễ đọc của test assertion và chất lượng error message.
- [WireMock](https://github.com/tomakehurst/wiremock): Công cụ mô phỏng HTTP service (Mock your APIs).
- [Testcontainers](https://github.com/testcontainers/testcontainers-java): Khởi động container dùng một lần cho integration test, hỗ trợ database, message queue, browser và các dependency khác có thể containerize.

Đọc thêm:

- [The Practical Test Pyramid- Martin Fowler](https://martinfowler.com/articles/practical-test-pyramid.html) (Một bài viết rất hay, nhưng bằng tiếng Anh)

### Testing platform

- [MeterSphere](https://github.com/metersphere/metersphere): Testing platform continuous mã nguồn mở, bao quát các chức năng như test management, API testing, performance testing và team collaboration.

### API debugging

- [Insomnia](https://github.com/Kong/insomnia): API client cross-platform, hỗ trợ REST, GraphQL, WebSocket, SSE và gRPC, đồng thời cung cấp các phương thức lưu trữ data local và Git.
- [Hoppscotch](https://github.com/hoppscotch/hoppscotch): API testing tool mã nguồn mở, được định vị là lựa chọn mã nguồn mở thay thế cho các sản phẩm như Postman và Insomnia.
- [Restful Fast Request](https://github.com/dromara/fast-request): Postman dành cho IDEA, gồm API debugging tool + API management tool + API search tool.

## Task scheduling

- [Quartz](https://github.com/quartz-scheduler/quartz): Một scheduling framework mã nguồn mở rất phổ biến, là framework tiêu biểu hoặc standard trong lĩnh vực Java scheduled task. Nhiều scheduling framework khác được phát triển dựa trên `quartz`, chẳng hạn `elastic-job` của Dangdang là distributed scheduling solution được phát triển mở rộng dựa trên `quartz`.
- [XXL-JOB](https://github.com/xuxueli/xxl-job): Distributed task scheduling platform, cung cấp các khả năng như task management, scheduling log, failover và mở rộng executor.
- [ElasticJob](https://github.com/apache/shardingsphere-elasticjob): Distributed scheduling solution dựa trên Quartz và ZooKeeper, cung cấp các khả năng như sharding, scale-out linh hoạt và failover.
- [DolphinScheduler](https://github.com/apache/dolphinscheduler): Distributed, dễ mở rộng, visual workflow task scheduling platform, dùng để orchestrate các task có dependency phức tạp.

## Workflow

1. [Flowable](https://github.com/flowable/flowable-engine) : Phát triển từ một nhánh của Activiti5, có chức năng phong phú. Trên cơ sở Activiti, nó bổ sung nhiều chức năng nâng cao như hỗ trợ mạnh hơn cho CMMN (Case Management Model and Notation), DMN (Decision Model and Notation), cùng các lựa chọn tích hợp linh hoạt hơn.
2. [Activiti](https://github.com/Activiti/Activiti): Phần mở rộng chức năng tương đối thận trọng, phù hợp với các ứng dụng enterprise truyền thống cần BPMN 2.0 workflow engine ổn định.
3. [Warm-Flow](https://gitee.com/dromara/warm-flow): Workflow engine mã nguồn mở trong nước, có đặc điểm đơn giản, lightweight nhưng không đơn điệu, đầy đủ chức năng, component độc lập và có thể mở rộng.
4. [FlowLong](https://gitee.com/aizuda/flowlong): Workflow engine mã nguồn mở trong nước, được xây dựng chuyên biệt cho quy trình phê duyệt mang đặc trưng Trung Quốc.

## Distributed

### API gateway

- [Kong](https://github.com/Kong/kong "kong"): Kong là một distributed microservice abstraction layer cloud-native, nhanh và có khả năng scale (còn được gọi là API gateway, API middleware hoặc trong một số trường hợp là service mesh). Được phát hành dưới dạng dự án mã nguồn mở vào năm 2015, giá trị cốt lõi của nó là performance cao và khả năng mở rộng.
- [ShenYu](https://github.com/apache/shenyu): API gateway reactive, hiệu năng cao, có khả năng mở rộng, phù hợp với các trường hợp microservice.
- [Spring Cloud Gateway](https://github.com/spring-cloud/spring-cloud-gateway): API gateway của Spring Cloud, cung cấp các khả năng như routing, filter, rate limiting và tích hợp service discovery.
- [Zuul](https://github.com/Netflix/zuul): L7 application gateway mã nguồn mở của Netflix, cung cấp các khả năng như dynamic routing, monitoring, elasticity và security.

### Configuration center

- [Apollo](https://github.com/ctripcorp/apollo "apollo") (khuyến nghị): Apollo là distributed configuration center do bộ phận framework của Ctrip phát triển, có thể quản lý tập trung configuration của application ở các environment và cluster khác nhau, push theo thời gian thực đến application sau khi configuration thay đổi, đồng thời có các đặc tính như quản lý permission và governance process theo chuẩn, phù hợp với các trường hợp quản lý configuration cho microservice.
- [Nacos](https://github.com/alibaba/nacos) (khuyến nghị): Nacos là component service registration và discovery do Spring Cloud Alibaba cung cấp, tương tự Consul và Eureka. Ngoài ra, nó còn cung cấp chức năng distributed configuration management.
- [Spring Cloud Config](https://github.com/spring-cloud/spring-cloud-config): Spring Cloud Config là configuration center ra đời sớm nhất trong hệ sinh thái Spring Cloud. Dù sau đó Consul được phát hành và có thể thay thế chức năng configuration center, Config vẫn phù hợp với dự án Spring Cloud và có thể triển khai chức năng thông qua configuration đơn giản.
- [Consul](https://github.com/hashicorp/consul): Consul là phần mềm mã nguồn mở do HashiCorp phát hành, cung cấp các chức năng như service governance, configuration center và control bus trong hệ thống microservice. Mỗi chức năng có thể được sử dụng riêng theo nhu cầu hoặc cùng nhau để xây dựng một service mesh toàn diện. Tóm lại, Consul cung cấp một giải pháp service mesh hoàn chỉnh.

### Distributed tracing

- [SkyWalking](https://github.com/apache/skywalking "skywalking"): Application performance monitoring và observability platform hướng đến các hệ thống distributed, microservice và cloud-native.
- [OpenTelemetry Java](https://github.com/open-telemetry/opentelemetry-java): Java API và SDK của OpenTelemetry, dùng để tạo và export dữ liệu telemetry Trace, Metric và Log. Nó phụ trách instrumentation và collection, không tương đương backend storage và visualization platform.

### Distributed lock

- [Redisson](https://github.com/redisson/redisson "redisson"): Redisson cung cấp hỗ trợ toàn diện và mạnh mẽ cho distributed lock, vượt qua implementation Redis lock đơn giản.
- [ShedLock](https://github.com/lukas-krecan/ShedLock): Lock scheduled task trong môi trường deploy distributed, tránh cùng một task được thực thi đồng thời trên nhiều instance. Nó chỉ giải quyết mutual exclusion của scheduled task, không phải giải pháp distributed lock dùng chung.

## High-performance

### Multithreading

- [Dynamic Tp](https://github.com/dromara/dynamic-tp): Dynamic thread pool lightweight, tích hợp chức năng monitoring và alerting, quản lý thread pool của third-party middleware, dựa trên configuration center phổ biến (đã hỗ trợ Nacos, Apollo, Zookeeper, Consul, Etcd, có thể custom implementation thông qua SPI).

### Cache

#### Local cache

- [Caffeine](https://github.com/ben-manes/caffeine): Thư viện local cache Java hiệu năng cao, hỗ trợ eviction strategy dựa trên capacity, time và reference.
- [Guava](https://github.com/google/guava): Java core library của Google, tích hợp implementation local cache tương đối hoàn chỉnh.

#### Distributed cache

- [Redis](https://github.com/redis/redis): Một in-memory database được phát triển bằng ngôn ngữ C, lựa chọn hàng đầu cho distributed cache.
- [Dragonfly](https://github.com/dragonflydb/dragonfly): In-memory database tương thích với Redis và Memcached API. Trước khi migration vẫn cần kiểm tra command coverage, persistence, cluster capability và client behavior, không thể trực tiếp thay thế chỉ dựa trên tuyên bố tương thích protocol.

#### Multi-level cache

- [JetCache](https://github.com/alibaba/jetcache): Cache framework mã nguồn mở của Alibaba, hỗ trợ các chức năng như multi-level cache, tự động refresh distributed cache và TTL.

### Message queue

**Distributed queue**:

- [RocketMQ](https://github.com/apache/rocketmq "RocketMQ"): Một distributed message middleware hiệu năng cao, throughput cao do Alibaba mã nguồn mở.
- [Kafka](https://github.com/apache/kafka "Kafka"): Kafka là một distributed message system dựa trên publish / subscribe.
- [RabbitMQ](https://github.com/rabbitmq/rabbitmq-server): Message queue được phát triển bằng Erlang và triển khai protocol AMQP.

### Read-write separation và database sharding

- [ShardingSphere](https://github.com/apache/shardingsphere): ShardingSphere là một ecosystem gồm các giải pháp distributed database middleware mã nguồn mở, bao gồm 3 sản phẩm độc lập với nhau là Sharding-JDBC, Sharding-Proxy và Sharding-Sidecar (đang lên kế hoạch).

## High availability

### Rate limiting

Distributed rate limiting:

- [Sentinel](https://github.com/alibaba/Sentinel) (khuyến nghị): Component bảo vệ high availability hướng đến distributed service architecture, lấy traffic làm điểm bắt đầu, giúp user đảm bảo tính ổn định của microservice từ nhiều khía cạnh như traffic control, circuit breaking và degradation, system adaptive protection.

Single-instance rate limiting:

- [Bucket4j](https://github.com/vladimir-bukhtoyarov/bucket4j): Một rate limiting library rất tốt dựa trên token bucket / leaky bucket algorithm.
- [Resilience4j](https://github.com/resilience4j/resilience4j): Fault tolerance component lightweight, cung cấp các khả năng như circuit breaking, rate limiting, retry và isolation.

### Monitoring

- [Spring Boot Admin](https://github.com/codecentric/spring-boot-admin): Quản lý và monitoring ứng dụng Spring Boot.
- [Metrics](https://github.com/dropwizard/metrics): Thu thập metric ở cấp JVM và application. Nhờ đó bạn biết điều gì đang xảy ra.

### Logging

- ELK: Tổ hợp Elasticsearch, Logstash và Kibana.
- Elastic Stack: Bổ sung các component data collection như Beats trên nền tảng ELK.
- EFK: Giải pháp logging dùng [Fluentd](https://github.com/fluent/fluentd) thay cho Logstash.

## Bytecode manipulation

- [ASM](https://asm.ow2.io/): Framework thao tác và phân tích Java bytecode dùng chung. Nó có thể được dùng để sửa trực tiếp class hiện có ở dạng binary hoặc tạo class động.
- [Byte Buddy](https://github.com/raphw/byte-buddy): Thư viện tạo và thao tác Java bytecode, có thể tạo và sửa Java class tại runtime mà không cần gọi Java compiler.
- [Javassist](https://github.com/jboss-javassist/javassist): Thư viện chỉnh sửa Java bytecode động.
- [Recaf](https://github.com/Col-E/Recaf): Java bytecode editor hiện đại, dựa trên ASM (framework thao tác Java bytecode) để sửa bytecode, giúp đơn giản hóa quá trình chỉnh sửa Java application đã compile.
