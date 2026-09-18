---
title: Giải thích chi tiết nguyên lý và thuật toán load balancing
description: Bài viết giải thích chi tiết nguyên lý cốt lõi của load balancing, gồm sự khác nhau giữa load balancing Layer 4/Layer 7, so sánh load balancing phía server và phía client, các thuật toán load balancing thường gặp như round robin, weighted round robin, random, consistent hashing, cùng các giải pháp phổ biến như Nginx và LVS.
category: High Performance
head:
  - - meta
    - name: keywords
      content: load balancing,Layer 4 load balancing,Layer 7 load balancing,Nginx load balancing,LVS,load balancing algorithm,round robin,consistent hashing,client-side load balancing
---

## Load balancing là gì?

**Load balancing** là việc phân phối request của người dùng đến các server khác nhau để xử lý, nhằm nâng cao năng lực xử lý đồng thời và độ tin cậy tổng thể của hệ thống. Dịch vụ load balancing có thể do phần mềm hoặc phần cứng chuyên dụng đảm nhiệm. Thông thường, phần cứng có performance tốt hơn, còn phần mềm có giá rẻ hơn (sẽ giới thiệu chi tiết ở phần sau).

Hình dưới đây là hình minh họa trong bài viết thuộc phần «High Concurrency» của [Java Interview Guide](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247519384&idx=1&sn=bc7e71af75350b755f04ca4178395b1a&chksm=cea1c353f9d64a458f797696d4144b4d6e58639371a4612b8e4d106d83a66d2289e7b2cd7431&token=660789642&lang=zh_CN&scene=21#wechat_redirect). Có thể thấy hệ thống triển khai nhiều instance của service sản phẩm trên các server khác nhau. Để phân phối request truy cập service sản phẩm, chúng ta sử dụng load balancing.

![Nhiều service instance - load balancing](https://oss.javaguide.cn/github/javaguide/high-performance/load-balancing/multi-service-load-balancing.drawio.png)

Load balancing là một cách khá phổ biến và tương đối đơn giản để nâng cao năng lực xử lý đồng thời và độ tin cậy của hệ thống. Hầu như mọi hệ thống, dù là kiến trúc monolith hay kiến trúc microservice, đều sử dụng load balancing.

Tuy nhiên, load balancing chỉ giải quyết vấn đề “phân tán request đến nhiều node”, không tự động giải quyết các bottleneck nghiệp vụ như slow SQL, thread pool cạn kiệt, cache breakdown hay API phía dưới bị chậm. Nếu tất cả node backend đều chậm, load balancing chỉ có thể phân phối các request chậm đều hơn, không thể khiến hệ thống tự nhiên nhanh hơn.

## Load balancing được chia thành những loại nào?

Load balancing có thể được chia đơn giản thành **load balancing phía server** và **load balancing phía client**.

Load balancing phía server liên quan đến nhiều kiến thức hơn và cũng thường gặp hơn trong công việc, vì vậy tôi sẽ dành nhiều thời gian hơn để giới thiệu loại này.

### Load balancing phía server

**Load balancing phía server** chủ yếu nằm giữa **request bên ngoài hệ thống** và **gateway layer**, có thể được triển khai bằng **phần mềm** hoặc **phần cứng**.

Hình dưới đây là sơ đồ đơn giản về load balancing phía server dựa trên Nginx:

![Load balancing phía server dựa trên Nginx](https://oss.javaguide.cn/github/javaguide/high-performance/load-balancing/server-load-balancing.png)

**Hardware load balancing** được triển khai bằng thiết bị phần cứng chuyên dụng (như **F5, A10, Array**).

Ưu điểm của hardware load balancing là performance mạnh và ổn định, nhược điểm là quá đắt. Một thiết bị F5 dòng cơ bản cũng có giá tối thiểu hơn 200.000, phần lớn công ty không thể chi trả. Nếu business không lớn thì không cần nhất thiết dùng phần cứng để làm load balancing, software load balancing là đủ!

Trong quá trình phát triển hằng ngày, chúng ta thường rất khó tiếp xúc với hardware load balancing. Loại thường gặp hơn là **software load balancing**. Software load balancing được triển khai bằng phần mềm (như **LVS, Nginx, HAproxy**), performance tuy kém hơn nhưng giá rẻ! Một server Linux cơ bản chỉ có giá vài nghìn, loại có performance tốt hơn với giá 20.000~30.000 đã rất ổn.

Theo mô hình OSI, load balancing phía server còn có thể được chia thành:

- Load balancing Layer 2
- Load balancing Layer 3
- Load balancing Layer 4
- Load balancing Layer 7

Phổ biến nhất là load balancing Layer 4 và Layer 7, vì vậy bài viết này cũng tập trung giới thiệu hai loại đó.

> Website Nginx đã giới thiệu chi tiết về load balancing Layer 4 và Layer 7. Nếu quan tâm, bạn có thể xem:
>
> - [What Is Layer 4 Load Balancing?](https://www.nginx.com/resources/glossary/layer-4-load-balancing/)
> - [What Is Layer 7 Load Balancing?](https://www.nginx.com/resources/glossary/layer-7-load-balancing/)

![Mô hình bảy layer OSI](https://oss.javaguide.cn/github/javaguide/cs-basics/network/osi-7-model.png)

- **Load balancing Layer 4** hoạt động ở layer thứ tư của mô hình OSI, tức transport layer. Protocol chính ở layer này là TCP/UDP. Load balancer ở layer này có thể thấy địa chỉ source port và destination port trong packet, sau đó dựa trên các thông tin này để chuyển packet đến real server phía sau bằng một thuật toán load balancing nhất định. Nói cách khác, cốt lõi của load balancing Layer 4 là load balancing ở mức IP + port, không liên quan đến nội dung cụ thể của message.
- **Load balancing Layer 7** hoạt động ở layer thứ bảy của mô hình OSI, tức application layer. Protocol chính ở layer này là HTTP. Cách load balancing layer này route network request phức tạp hơn Layer 4. Nó sẽ đọc phần data của message (chẳng hạn message HTTP), sau đó đưa ra quyết định load balancing dựa trên nội dung đã đọc (như URL, Cookie). Nói cách khác, cốt lõi của load balancer Layer 7 là load balancing ở mức nội dung message (như URL, Cookie). Thiết bị thực hiện load balancing Layer 7 thường được gọi là **reverse proxy server**.

Load balancing Layer 7 tiêu tốn nhiều performance hơn Layer 4, nhưng cũng linh hoạt hơn, có thể route network request thông minh hơn. Chẳng hạn, có thể tối ưu theo nội dung request như cache, compression và encryption.

Nói ngắn gọn, **load balancing Layer 4 có performance mạnh, load balancing Layer 7 có chức năng mạnh hơn!** Tuy nhiên, với phần lớn business, chênh lệch performance giữa load balancing Layer 4 và Layer 7 về cơ bản có thể bỏ qua.

Khi lựa chọn thực tế, có thể phán đoán như sau:

| Tiêu chí             | Load balancing Layer 4                            | Load balancing Layer 7                             |
| -------------------- | ------------------------------------------------- | -------------------------------------------------- |
| Layer hoạt động      | TCP/UDP                                           | HTTP/HTTPS                                         |
| Căn cứ chuyển tiếp   | IP, port                                          | domain, path, Header, Cookie                       |
| Overhead performance | Thấp hơn                                          | Cao hơn                                            |
| Tính linh hoạt       | Yếu hơn                                           | Mạnh hơn                                           |
| Trường hợp điển hình | Cổng vào traffic lớn, database proxy, TCP service | Gateway, reverse proxy, gray release, path routing |

Nếu chỉ chuyển tiếp TCP traffic, Layer 4 trực tiếp hơn; nếu cần route theo URL, Header, Cookie hoặc thực hiện compression, cache, TLS termination thì Layer 7 phù hợp hơn.

Dưới đây là đoạn trích từ bài viết [What Is Layer 4 Load Balancing?](https://www.nginx.com/resources/glossary/layer-4-load-balancing/) trên website Nginx.

> Layer 4 load balancing was a popular architectural approach to traffic handling when commodity hardware was not as powerful as it is now, and the interaction between clients and application servers was much less complex. It requires less computation than more sophisticated load balancing methods (such as Layer 7), but CPU and memory are now sufficiently fast and cheap that the performance advantage for Layer 4 load balancing has become negligible or irrelevant in most situations.
>
> Layer 4 load balancing là một phương pháp kiến trúc xử lý traffic phổ biến khi commodity hardware chưa mạnh như hiện nay và tương tác giữa client với application server cũng đơn giản hơn nhiều. Nó cần ít computation hơn các phương pháp load balancing phức tạp hơn (như Layer 7), nhưng CPU và memory hiện nay đã đủ nhanh và rẻ, nên ưu thế performance của load balancing Layer 4 trong phần lớn trường hợp đã trở nên không đáng kể hoặc không còn quan trọng.

Trong công việc, chúng ta thường sử dụng **Nginx** để thực hiện load balancing Layer 7, còn LVS (Linux Virtual Server, load balancing Layer 4 của Linux kernel) để thực hiện load balancing Layer 4.

Tổng hợp các kiến thức thường gặp về Nginx đã có trong phần «Câu hỏi phỏng vấn kỹ thuật» của [Java Interview Guide](https://javaguide.cn/zhuanlan/java-mian-shi-zhi-bei.html). Nếu quan tâm, bạn có thể xem.

![](https://oss.javaguide.cn/github/javaguide/image-20220328105759300.png)

Tuy nhiên, phần lớn công ty không thực sự cần LVS. Chỉ các công ty lớn như Alibaba, Baidu, Tencent, eBay mới sử dụng, còn được dùng nhiều nhất vẫn là Nginx.

### Load balancing phía client

**Load balancing phía client** chủ yếu được sử dụng giữa các service khác nhau bên trong hệ thống, có thể triển khai bằng component load balancing có sẵn.

Trong load balancing phía client, client tự duy trì một danh sách địa chỉ server. Trước khi gửi request, client chọn một server cụ thể để xử lý request theo thuật toán load balancing tương ứng.

Load balancer phía client chạy cùng process với service, hay nói cách khác là trong cùng chương trình Java, nên không phát sinh network overhead bổ sung. Tuy nhiên, việc triển khai load balancing phía client bị giới hạn bởi ngôn ngữ lập trình. Chẳng hạn, Spring Cloud Load Balancer chỉ dùng được với ngôn ngữ Java.

Các framework microservice phổ biến trong hệ sinh thái Java như Dubbo và Spring Cloud đều tích hợp sẵn implementation load balancing phía client có thể dùng ngay. Dubbo mặc định tích hợp chức năng load balancing, còn Spring Cloud triển khai load balancing dưới dạng component, là một tùy chọn. Hai lựa chọn thường dùng là Spring Cloud Load Balancer (chính thức, được khuyến nghị) và Ribbon (Netflix, đã deprecated).

Hình dưới đây là sơ đồ đơn giản về load balancing phía client dựa trên Spring Cloud Load Balancer (Ribbon cũng tương tự):

![](https://oss.javaguide.cn/github/javaguide/high-performance/load-balancing/spring-cloud-lb-gateway.png)

## Những thuật toán load balancing thường gặp là gì?

### Random

**Random** là thuật toán load balancing đơn giản và trực tiếp nhất.

Nếu không cấu hình weight, xác suất được truy cập của mọi server là như nhau. Nếu cấu hình weight, server có weight càng cao thì xác suất được truy cập càng lớn.

Random không có weight phù hợp với cluster có performance server tương đương, trong đó mỗi server chịu cùng một load. Weighted random phù hợp với cluster có performance server khác nhau; weight giúp phân phối request hợp lý hơn.

Tuy nhiên, random có một nhược điểm khá rõ: một số máy có thể không được chọn trong một khoảng thời gian, vì đây là thuật toán xác suất. Ngay cả khi mọi máy có cùng weight, tình huống này vẫn có thể xảy ra.

Vì vậy, **round robin** ra đời!

### Round robin

Round robin lần lượt chọn từng server để xử lý, đồng thời cũng có thể cấu hình weight.

Nếu không cấu hình weight, mỗi request được lần lượt phân phối đến các server khác nhau theo thứ tự thời gian. Nếu cấu hình weight, server có weight càng cao thì được truy cập càng nhiều lần.

Round robin không có weight phù hợp với cluster có performance server tương đương, trong đó mỗi server chịu cùng một load. Weighted round robin phù hợp với cluster có performance server khác nhau; weight giúp phân phối request hợp lý hơn.

Trên nền tảng weighted round robin còn có các thuật toán load balancing được cải tiến thêm, chẳng hạn smooth weighted round robin.

Smooth weighted round robin lần đầu được triển khai trong Nginx. Bạn có thể tham khảo commit này: <https://github.com/phusion/nginx/commit/27e94984486058d73157038f7950a0a36ecc6e35>. Nếu đã học kỹ strategy load balancing của Dubbo, bạn sẽ nhận ra weighted round robin của Dubbo đã tham khảo thuật toán này và tiếp tục tối ưu.

![Thuật toán weighted round robin load balancing của Dubbo](https://oss.javaguide.cn/github/javaguide/high-performance/load-balancing/dubbo-round-robin-load-balance.png)

### Two-random

Two-random bổ sung thêm một lần random trên nền tảng random, chọn thêm một server. Sau đó, dựa trên load và các điều kiện khác của hai server để chọn server phù hợp nhất.

Ưu điểm của two-random là có thể điều chỉnh động load của các node backend, giúp load cân bằng hơn. Nếu chỉ dùng random một lần, một số server có thể bị quá tải trong khi một số server khác lại rảnh.

### Hash

Thông tin parameter của request được chuyển thành một hash value thông qua hash function, sau đó quyết định request được server nào xử lý dựa trên hash value.

Khi số lượng server không thay đổi, request có cùng parameter luôn được gửi đến cùng một server, chẳng hạn request từ cùng một IP hoặc cùng một user.

### Consistent Hash

Tương tự Hash, Consistent Hash cũng có thể khiến request có cùng parameter luôn được gửi đến cùng một server. Tuy nhiên, nó giải quyết được một số vấn đề của Hash.

Với Hash thông thường, khi số lượng server thay đổi, hash value sẽ lại rơi vào các server khác nhau. Điều này rõ ràng đi ngược mục đích ban đầu của việc sử dụng Hash. Ý tưởng cốt lõi của Consistent Hash là ánh xạ data và node lên cùng một hash ring, sau đó xác định data thuộc node nào theo thứ tự của hash value. Khi thêm hoặc xóa server, chỉ hash của server đó bị ảnh hưởng, không khiến toàn bộ hash key của service phải phân phối lại.

### Ít connection nhất

Khi có request mới, duyệt danh sách server node và chọn server có số connection nhỏ nhất để response request hiện tại. Nếu số connection bằng nhau, có thể dùng weighted random.

Thuật toán ít connection nhất dựa trên giả định lý tưởng rằng server càng có nhiều connection thì load càng cao. Tuy nhiên, trên thực tế, số connection không thể đại diện cho load thực tế của server: một số connection tiêu tốn nhiều resource hệ thống hơn, một số connection khác lại tiêu tốn ít hơn.

### Ít active nhất

Thuật toán ít active nhất tương tự thuật toán ít connection nhất nhưng khoa học hơn. Nó lấy số active connection làm tiêu chuẩn. Có thể hiểu active connection là số request đang được xử lý hiện tại. Số active càng thấp nghĩa là năng lực xử lý càng mạnh, từ đó server có năng lực xử lý mạnh sẽ nhận nhiều request hơn. Nếu số active bằng nhau, có thể dùng weighted random.

### Response time nhanh nhất

Khác với thuật toán ít connection nhất và ít active nhất, thuật toán response time nhanh nhất lấy response time làm tiêu chuẩn để chọn server xử lý. Client duy trì response time của từng server và chọn server có response time ngắn nhất cho mỗi request. Nếu response time bằng nhau, có thể dùng weighted random.

Thuật toán này có thể giúp request được xử lý nhanh hơn, nhưng có thể khiến traffic tập trung quá nhiều vào các server performance cao.

## Có thể thực hiện load balancing Layer 7 như thế nào?

Dưới đây là giới thiệu ngắn gọn về hai giải pháp load balancing Layer 7 thường dùng trong project: DNS resolution và reverse proxy.

Ngoài hai giải pháp này, các cách như HTTP redirect cũng có thể dùng để thực hiện load balancing. Tuy nhiên, DNS resolution và reverse proxy được sử dụng nhiều hơn và cũng được khuyến nghị hơn.

### DNS resolution

DNS resolution là cách triển khai load balancing Layer 7 xuất hiện tương đối sớm và khá đơn giản.

Nguyên lý thực hiện load balancing bằng DNS resolution như sau: cấu hình nhiều địa chỉ IP cho cùng một host record trên DNS server. Các địa chỉ IP này tương ứng với những server khác nhau. Khi user request domain, DNS server dùng thuật toán round robin để trả về địa chỉ IP, từ đó thực hiện load balancing dạng round robin.

![](https://oss.javaguide.cn/github/javaguide/high-performance/load-balancing/6997605302452f07e8b28d257d349bf0.png)

DNS resolution hiện nay gần như đều hỗ trợ cấu hình weight cho địa chỉ IP. Nhờ đó, việc phân phối request trong cluster có performance server khác nhau sẽ hợp lý hơn. Chẳng hạn, DNS của Alibaba Cloud mà tôi đang sử dụng cũng hỗ trợ cấu hình weight.

![](https://oss.javaguide.cn/github/javaguide/aliyun-dns-weight-setting.png)

### Reverse proxy

Client gửi request đến reverse proxy server. Reverse proxy server chọn server đích, lấy data rồi trả về client. Địa chỉ được expose ra bên ngoài là địa chỉ của reverse proxy server, còn IP của real server được ẩn đi. Reverse proxy “proxy” cho server đích, và quá trình này là transparent đối với client.

Nginx là reverse proxy server phổ biến nhất. Nó có thể phân phối đều request client nhận được đến tất cả server trong cluster server theo một rule nhất định (strategy load balancing).

Load balancing bằng reverse proxy cũng thuộc load balancing Layer 7.

![](https://oss.javaguide.cn/github/javaguide/nginx-load-balance.png)

### Health check và rút traffic

Sau khi load balancing thực sự được đưa vào production, health check quan trọng hơn bản thân thuật toán. Thuật toán quyết định “phân phối traffic thế nào trong điều kiện bình thường”, còn health check quyết định “node bất thường có tiếp tục nhận traffic hay không”.

Các cách health check thường gặp:

- **TCP check**: chỉ cần port kết nối được thì xem node là khả dụng. Phù hợp với load balancing Layer 4 nhưng không thể xác định business có thực sự bình thường hay không.
- **HTTP check**: định kỳ truy cập các API như `/health`, `/actuator/health`, rồi xác định node có khả dụng hay không dựa trên status code và nội dung response.
- **Business liveness check**: kiểm tra trạng thái của các component quan trọng như database, cache và service phụ thuộc. Phù hợp với core path, nhưng logic liveness check không được quá nặng.

Trong môi trường production, nên thiết lập cơ chế warm-up và rút traffic cho backend node. Khi node mới vừa khởi động, JIT, connection pool và cache vẫn chưa warm-up xong; đưa toàn bộ traffic vào ngay dễ gây dao động. Khi tắt node, cũng cần ngừng nhận request mới trước, rồi chờ các request đang xử lý hoàn tất.

## Load balancing phía client thường được thực hiện như thế nào?

Như đã nói ở trên, load balancing phía client có thể được triển khai bằng component load balancing có sẵn.

**Netflix Ribbon** và **Spring Cloud Load Balancer** hiện là hai component load balancing phổ biến nhất trong hệ sinh thái Java.

Ribbon là component load balancing lâu đời do Netflix phát triển, có chức năng khá đầy đủ và hỗ trợ nhiều strategy load balancing. Spring Cloud Load Balancer được Spring phát triển để thay thế Ribbon, có chức năng tương đối đơn giản hơn và hỗ trợ ít loại load balancing hơn.

7 strategy load balancing được Ribbon hỗ trợ:

- `RandomRule`: strategy random.
- `RoundRobinRule` (mặc định): strategy round robin.
- `WeightedResponseTimeRule`: strategy weight (xác định weight theo response time).
- `BestAvailableRule`: strategy ít connection nhất.
- `RetryRule`: strategy retry (lấy service theo strategy round robin. Nếu service instance nhận được là null hoặc đã hết hiệu lực, liên tục retry trong thời gian được chỉ định để lấy service. Nếu hết thời gian mà vẫn không lấy được service instance thì trả về null).
- `AvailabilityFilteringRule`: strategy ưu tiên availability (lọc service instance không healthy trước, sau đó chọn service instance có số connection nhỏ hơn).
- `ZoneAvoidanceRule`: strategy ưu tiên zone (chọn service instance dựa trên performance và availability của zone nơi service nằm).

2 strategy load balancing được Spring Cloud Load Balancer hỗ trợ:

- `RandomLoadBalancer`: strategy random.
- `RoundRobinLoadBalancer` (mặc định): strategy round robin.

```java
public class CustomLoadBalancerConfiguration {

    @Bean
    ReactorLoadBalancer<ServiceInstance> randomLoadBalancer(Environment environment,
            LoadBalancerClientFactory loadBalancerClientFactory) {
        String name = environment.getProperty(LoadBalancerClientFactory.PROPERTY_NAME);
        return new RandomLoadBalancer(loadBalancerClientFactory
                .getLazyProvider(name, ServiceInstanceListSupplier.class),
                name);
    }
}
```

Tuy nhiên, các strategy load balancing mà Spring Cloud Load Balancer hỗ trợ thực ra không chỉ có hai loại này. Các implementation class của `ServiceInstanceListSupplier` cũng cho phép nó hỗ trợ các strategy load balancing tương tự Ribbon. Có lẽ các strategy này được bổ sung dần về sau; nếu không đọc documentation chính thức thì rất khó phát hiện. Vì vậy, đọc documentation chính thức thực sự rất quan trọng!

Dưới đây là hai ví dụ chính thức:

- `ZonePreferenceServiceInstanceListSupplier`: thực hiện load balancing dựa trên zone.
- `HintBasedServiceInstanceListSupplier`: thực hiện load balancing dựa trên hint.

```java
public class CustomLoadBalancerConfiguration {
    // Sử dụng phương thức load balancing dựa trên zone
    @Bean
    public ServiceInstanceListSupplier discoveryClientServiceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSupplier.builder()
                    .withDiscoveryClient()
                    .withZonePreference()
                    .withCaching()
                    .build(context);
    }
}
```

Để biết thêm thông tin cập nhật và chi tiết hơn về Spring Cloud Load Balancer, bạn nên xem documentation chính thức: <https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#spring-cloud-loadbalancer>. Mọi thứ nên dựa trên documentation chính thức.

Strategy round robin về cơ bản đáp ứng nhu cầu của phần lớn project. Trong project thực tế, nếu không có yêu cầu đặc biệt thì chúng ta thường dùng strategy round robin mặc định. Ribbon và Spring Cloud Load Balancer đều hỗ trợ custom strategy load balancing.

Theo ý kiến cá nhân, nếu không bắt buộc phải dùng một chức năng hoặc strategy load balancing riêng của Ribbon, hãy ưu tiên Spring Cloud Load Balancer do Spring cung cấp.

Cuối cùng, hãy nói về lý do tôi không quá khuyến nghị dùng Ribbon.

Spring Cloud 2020.0.0 đã loại bỏ tất cả component của Netflix ngoài Eureka. Spring Cloud Hoxton.M2 là version đầu tiên hỗ trợ Spring Cloud Load Balancer để thay thế Netfix Ribbon.

Khi mới học microservice, chắc chắn chúng ta từng tiếp xúc với các component nổi tiếng cần thiết để xây dựng hệ thống microservice do Netflix open source như Feign, Ribbon, Zuul, Hystrix và Eureka. Cho đến nay vẫn có rất nhiều công ty sử dụng các component này. Không quá lời khi nói Netflix đã dẫn dắt sự phát triển của microservice trong Java tech stack.

![](https://oss.javaguide.cn/github/javaguide/SpringCloudNetflix.png)

**Vậy tại sao Spring Cloud lại vội loại bỏ các component của Netflix?** Chủ yếu vì năm 2018, Netflix tuyên bố các component cốt lõi mà họ open source như Hystrix, Ribbon, Zuul và Eureka đã chuyển sang trạng thái maintenance, không tiếp tục phát triển feature mới mà chỉ sửa bug. Vì vậy, Spring buộc phải cân nhắc loại bỏ các component của Netflix.

**Spring Cloud Alibaba** là một lựa chọn tốt, đặc biệt với các công ty và developer cá nhân trong nước.

## Khuyến nghị khi triển khai production

- **Xem metric trước rồi mới điều chỉnh thuật toán**: chú ý QPS, RT, P99, error rate, số connection và CPU/memory của từng node, thay vì chỉ xem số request có được phân phối đều hay không.
- **Cách ly node chậm kịp thời**: khi một node chậm đi, strategy response time nhanh nhất và ít active nhất có thể hữu ích, nhưng root cause vẫn cần được xử lý bằng health check, circuit breaker và rút traffic.
- **Tránh phụ thuộc vào session sticky**: nếu buộc phải dùng Cookie hoặc IP Hash để cố định session, cần đánh giá ảnh hưởng đến user khi scale hoặc khi node gặp sự cố. Nên đưa session vào shared storage như Redis hơn.
- **Gray release phải rollback được**: load balancing Layer 7 thường dùng Header, Cookie hoặc user ID để gray release; rule gray release phải có thể tắt nhanh.
- **Điều phối traffic liên vùng theo vị trí gần nhất**: business trên toàn quốc hoặc toàn cầu thường cần kết hợp DNS/GSLB để điều phối user đến data center hoặc availability zone gần hơn.

## Tài liệu tham khảo

- Hàng chất lượng | Giải pháp software load balancing Layer 4 của eBay: <https://mp.weixin.qq.com/s/bZMxLTECOK3mjdgiLbHj-g>
- HTTP Load Balancing (documentation chính thức của Nginx): <https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/>
- Giải thích load balancing dễ hiểu: <https://www.cnblogs.com/vivotech/p/14859041.html>

<!-- @include: @article-footer.snippet.md -->
