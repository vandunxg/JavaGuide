---
title: "Giải thích chi tiết về rate limiting cho service: fixed window, sliding window, token bucket, leaky bucket và distributed rate limiting"
description: Giải thích chi tiết về nguyên lý và cách triển khai rate limiting cho service, trình bày hệ thống các thuật toán rate limiting như fixed window, sliding window, token bucket, leaky bucket, cùng các giải pháp rate limiting single-instance và distributed bằng Guava RateLimiter, Bucket4j, Sentinel, Redis Lua và Redisson.
category: High Availability
icon: "mdi:speedometer-slow"
tag:
  - Service Rate Limiting
  - High Concurrency
head:
  - - meta
    - name: keywords
      content: service rate limiting,rate limiting algorithm,fixed window,sliding window,token bucket,leaky bucket,single-instance rate limiting,distributed rate limiting,Guava RateLimiter,Bucket4j,Sentinel,Redis Lua,Redisson rate limiting,rate limiting interview questions
---

Đối với hệ thống phần mềm, rate limiting là giới hạn tốc độ request để tránh lượng request lớn đột ngột làm sập hệ thống. Suy cho cùng, năng lực xử lý của hệ thống phần mềm là có hạn. Nếu lượng request vượt quá năng lực xử lý, hệ thống phần mềm có thể sập ngay.

Rate limiting có thể khiến request của người dùng không được xử lý đúng hoặc không được xử lý ngay, nhưng đây thường là phương án tối ưu sau khi cân nhắc tính ổn định của hệ thống phần mềm.

Trong đời sống thực tế, rate limiting được áp dụng ở khắp nơi. Ví dụ, xếp hàng mua vé nhằm tránh quá nhiều người cùng tràn vào mua vé khiến nhân viên bán vé không thể xử lý.

## Có những thuật toán rate limiting phổ biến nào?

Giới thiệu đơn giản 4 thuật toán rate limiting rất dễ hiểu và dễ triển khai!

> Ảnh lấy từ bài viết ["Thực chiến rate limiting cho distributed service, đã giúp bạn tránh các cạm bẫy"](https://www.infoq.cn/article/Qg2tX8fyw5Vt-f3HH673) của InfoQ.

### Thuật toán fixed window counter

Fixed window thực chất là time window. Nguyên lý là chia thời gian thành các window có kích thước cố định, giới hạn số lượng hoặc tốc độ request trong mỗi window. Nói cách khác, thuật toán fixed window counter quy định số request hệ thống xử lý trong một đơn vị thời gian.

Giả sử quy định một API nào đó trong hệ thống chỉ được truy cập 33 lần mỗi phút, cách triển khai bằng thuật toán fixed window counter như sau:

- Chia thời gian thành các window có kích thước cố định, ở đây là một window mỗi phút.
- Dùng biến `counter` để ghi lại số request API hiện đã xử lý, giá trị ban đầu là 0 (nghĩa là API chưa xử lý request nào trong phút hiện tại).
- Sau mỗi request được xử lý trong một phút thì tăng `counter+1`. Khi `counter=33` (tức API đã được truy cập 33 lần trong phút đó), toàn bộ request tiếp theo sẽ bị từ chối.
- Khi hết một phút, reset `counter` về 0 và bắt đầu đếm lại.

![Thuật toán fixed window counter](https://static001.infoq.cn/resource/image/8d/15/8ded7a2b90e1482093f92fff555b3615.png)

Ưu điểm: triển khai đơn giản, dễ hiểu.

Nhược điểm:

- Rate limiting chưa đủ mượt. Ví dụ, giới hạn một API chỉ được truy cập 30 lần mỗi phút. Nếu 30 request đã đến trong 30 giây đầu, 30 giây tiếp theo sẽ không thể xử lý request, khiến trải nghiệm người dùng rất tệ!
- Tồn tại vấn đề đột biến ở ranh giới window. Ví dụ, giới hạn một API chỉ được truy cập 1000 lần mỗi phút. Nếu 1000 request vào trong giây cuối của phút thứ nhất, rồi 1000 request khác vào trong giây đầu của phút thứ hai, counter của window mới đã reset nên chỉ trong 2 giây có thể cho qua 2000 request, tương đương QPS cao hơn nhiều so với dự kiến là `1000/60≈16.7`.

### Thuật toán sliding window counter

**Thuật toán sliding window counter có thể xem là phiên bản nâng cấp của thuật toán fixed window counter**, với độ hạt rate limiting nhỏ hơn.

Cải tiến của sliding window counter so với fixed window counter là: **chia một window cố định thành nhiều sub-window nhỏ hơn (ô)**.

Ví dụ, API của chúng ta được giới hạn xử lý 60 request mỗi phút, có thể chia 1 phút thành 60 window. Cứ mỗi 1 giây di chuyển một lần, mỗi window một giây chỉ được xử lý không quá `60 (số request)/60 (số window)` request. Nếu tổng số request của window hiện tại vượt quá giới hạn thì không xử lý các request khác nữa.

Dễ thấy, **càng chia sliding window thành nhiều ô, chuyển động của sliding window càng mượt và thống kê rate limiting càng chính xác.**

![Thuật toán sliding window counter](https://static001.infoq.cn/resource/image/ae/15/ae4d3cd14efb8dc7046d691c90264715.png)

Ưu điểm:

- So với fixed window, thuật toán sliding window counter có thể giảm đáng kể vấn đề request tăng đột biến ở ranh giới window.
- So với fixed window, thuật toán sliding window counter có độ hạt nhỏ hơn, cho phép kiểm soát rate limiting chính xác hơn.

Nhược điểm:

- Tương tự fixed window, thuật toán sliding window counter vẫn có vấn đề rate limiting chưa đủ mượt.
- So với fixed window, thuật toán sliding window counter phức tạp hơn khi triển khai và tìm hiểu.

### Thuật toán leaky bucket

Có thể ví việc gửi request như đổ nước vào bucket, còn quá trình xử lý request như nước chảy ra khỏi leaky bucket. Nước chảy vào bucket với tốc độ bất kỳ và chảy ra với tốc độ cố định. Khi lượng nước chảy vào vượt quá sức chứa của bucket, phần nước thừa sẽ bị loại bỏ. Vì sức chứa bucket không đổi nên tốc độ tổng thể được đảm bảo.

Muốn triển khai thuật toán này cũng rất đơn giản: chuẩn bị một queue để lưu request, sau đó định kỳ lấy request từ queue ra thực thi (tư tưởng giống message queue dùng để giảm đỉnh và rate limiting).

![Thuật toán leaky bucket](https://static001.infoq.cn/resource/image/75/03/75938d1010138ce66e38c6ed0392f103.png)

Ưu điểm:

- Triển khai đơn giản, dễ hiểu.
- Có thể kiểm soát tốc độ rate limiting, tránh network congestion và system overload.

Nhược điểm:

- Không thể xử lý lưu lượng tăng đột ngột vì chỉ có thể xử lý request với tốc độ cố định, không thân thiện với việc sử dụng tài nguyên hệ thống.
- Nếu tốc độ nước chảy vào bucket (gửi request) luôn lớn hơn tốc độ nước chảy ra (xử lý request), bucket sẽ luôn đầy, một phần request mới sẽ bị loại bỏ, khiến chất lượng service giảm.

Trong các business scenario thực tế, phạm vi áp dụng của leaky bucket tương đối hẹp. Khi business cần cho phép một mức lưu lượng đột biến nhất định, token bucket thường phù hợp hơn. Nhưng trong scenario cần kiểm soát nghiêm ngặt tốc độ output (chẳng hạn API bên thứ ba có giới hạn QPS rõ ràng), leaky bucket vẫn là phương án phù hợp.

### Thuật toán token bucket

Thuật toán token bucket cũng khá đơn giản. Giống leaky bucket, nhân vật chính vẫn là bucket (thuật toán rate limiting này đúng là không thể tách khỏi bucket). Tuy nhiên, bucket hiện chứa token. Trước khi được xử lý, request cần lấy một token từ bucket (lấy là tiêu thụ). Nếu bucket không còn token, request phải chờ hoặc bị từ chối. Theo mức rate limiting, token được thêm vào bucket với tốc độ nhất định. Khi bucket đầy thì không thể tiếp tục thêm token.

![Thuật toán token bucket](https://static001.infoq.cn/resource/image/ec/93/eca0e5eaa35dac938c673fecf2ec9a93.png)

Ưu điểm:

- Có thể giới hạn tốc độ trung bình, đồng thời cho phép xử lý lưu lượng đột biến ngắn hạn trong phạm vi sức chứa của bucket.
- Có thể điều chỉnh động tốc độ tạo token.

Nhược điểm:

- Nếu tốc độ tạo token và sức chứa bucket được cấu hình không hợp lý, có thể phát sinh vấn đề như nhiều request bị loại bỏ hoặc system overload.
- So với các thuật toán rate limiting khác, việc triển khai và tìm hiểu phức tạp hơn.

## Rate limiting theo đối tượng nào?

Trong project thực tế, còn cần xác định đối tượng rate limiting, tức là rate limiting theo cái gì. Các đối tượng rate limiting phổ biến như sau:

- IP: rate limiting theo IP, phạm vi áp dụng rộng, đơn giản và trực tiếp.
- Business ID: chọn một business ID duy nhất để rate limiting có mục tiêu hơn. Ví dụ, rate limiting theo user ID.
- Cá nhân hóa: áp dụng strategy rate limiting khác nhau theo thuộc tính hoặc hành vi của người dùng. Ví dụ, không rate limiting VIP nhưng rate limiting user thường. Có thể điều chỉnh động strategy rate limiting theo các chỉ số vận hành của hệ thống (như QPS, số lượng invocation đồng thời, system load). Ví dụ, khi system load cao thì giảm số request được phép đi qua mỗi giây.

Rate limiting theo IP hiện là một phương án khá phổ biến. Tuy nhiên, trong ứng dụng thực tế cần chú ý lấy đúng địa chỉ IP thật của người dùng. Các cách thường dùng để lấy IP thật là trường X-Forwarded-For và trường TCP Options chứa thông tin IP nguồn thật. Dù trường X-Forwarded-For có thể bị giả mạo, nhiều project vẫn dùng trực tiếp cách này vì triển khai đơn giản và thuận tiện.

Ngoài các đối tượng rate limiting đã giới thiệu ở trên, còn có một số strategy đối tượng rate limiting phức tạp hơn. Ví dụ, Sentinel của Alibaba còn hỗ trợ [rate limiting dựa trên quan hệ invocation](https://github.com/alibaba/Sentinel/wiki/流量控制#基于调用关系的流量控制) (gồm rate limiting theo caller, rate limiting theo entry của call chain, rate limiting lưu lượng liên quan...) và [rate limiting theo hot parameter](https://github.com/alibaba/Sentinel/wiki/热点参数限流) với độ chi tiết cao hơn (thống kê hot parameter theo thời gian thực và kiểm soát lưu lượng của resource invocation tương ứng).

Ngoài ra, một project có thể kết hợp nhiều đối tượng rate limiting khác nhau theo nhu cầu business cụ thể.

## Triển khai single-instance rate limiting thế nào?

Single-instance rate limiting áp dụng cho ứng dụng có monolithic architecture.

Single-instance rate limiting có thể dùng trực tiếp utility class `RateLimiter` tích hợp sẵn trong Google Guava. `RateLimiter` dựa trên thuật toán token bucket và có thể xử lý lưu lượng đột biến.

> Địa chỉ Guava: <https://github.com/google/guava>

Ngoài implementation thuật toán token bucket cơ bản nhất (rate limiting burst mượt), `RateLimiter` của Guava còn cung cấp implementation thuật toán **rate limiting warm-up mượt**.

Rate limiting burst mượt là thêm token vào bucket theo tốc độ chỉ định, còn rate limiting warm-up mượt có một khoảng thời gian warm-up; trong thời gian đó, tốc độ sẽ dần tăng đến tốc độ đã cấu hình.

Dưới đây là hai ví dụ đơn giản.

Chỉ cần thêm dependency liên quan đến Guava vào project là có thể sử dụng.

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

> Ví dụ trong bài dựa trên Guava 31.x. Core API của `RateLimiter` vẫn tương thích với các phiên bản cao hơn.

Dưới đây là Demo rate limiting burst mượt đơn giản bằng Guava.

```java
import com.google.common.util.concurrent.RateLimiter;

/**
 * Tìm JavaGuide trên WeChat rồi trả lời "面试突击" (phỏng vấn cấp tốc)
 * để nhận miễn phí handbook phỏng vấn Java do tác giả tự biên soạn
 *
 * @author Guide
 * @date 2021/10/08 19:12
 **/
public class RateLimiterDemo {

    public static void main(String[] args) {
        // Thêm 5 token vào bucket mỗi 1 giây, tức thêm 1 token mỗi 0.2 giây
        RateLimiter rateLimiter = RateLimiter.create(5);
        for (int i = 0; i < 10; i++) {
            double sleepingTime = rateLimiter.acquire(1);
            System.out.printf("get 1 tokens: %ss%n", sleepingTime);
        }
    }
}

```

Output:

```bash
get 1 tokens: 0.0s
get 1 tokens: 0.188413s
get 1 tokens: 0.197811s
get 1 tokens: 0.198316s
get 1 tokens: 0.19864s
get 1 tokens: 0.199363s
get 1 tokens: 0.193997s
get 1 tokens: 0.199623s
get 1 tokens: 0.199357s
get 1 tokens: 0.195676s
```

Cần lưu ý, `acquire()` là cách lấy token blocking. Trong môi trường production thường nên dùng `tryAcquire(timeout)` để đặt giới hạn thời gian chờ, tránh thread bị block quá lâu. Thời gian chờ ở output đầu tiên là `0.0s` vì sau khi `RateLimiter.create(5)` được tạo, bucket đã có sẵn token khả dụng.

Dưới đây là Demo rate limiting warm-up mượt đơn giản bằng Guava.

```java
import com.google.common.util.concurrent.RateLimiter;
import java.util.concurrent.TimeUnit;

/**
 * Tìm JavaGuide trên WeChat rồi trả lời "面试突击" (phỏng vấn cấp tốc)
 * để nhận miễn phí handbook phỏng vấn Java do tác giả tự biên soạn
 *
 * @author Guide
 * @date 2021/10/08 19:12
 **/
public class RateLimiterDemo {

    public static void main(String[] args) {
        // Thêm 5 token vào bucket mỗi 1 giây, tức thêm 1 token mỗi 0.2 giây
        // Thời gian warm-up là 3 giây, tức trong 3 giây đầu tốc độ phát token sẽ dần tăng đến 1 token mỗi 0.2 giây
        RateLimiter rateLimiter = RateLimiter.create(5, 3, TimeUnit.SECONDS);
        for (int i = 0; i < 20; i++) {
            double sleepingTime = rateLimiter.acquire(1);
            System.out.printf("get 1 tokens: %sds%n", sleepingTime);
        }
    }
}
```

Output:

```bash
get 1 tokens: 0.0s
get 1 tokens: 0.561919s
get 1 tokens: 0.516931s
get 1 tokens: 0.463798s
get 1 tokens: 0.41286s
get 1 tokens: 0.356172s
get 1 tokens: 0.300489s
get 1 tokens: 0.252545s
get 1 tokens: 0.203996s
get 1 tokens: 0.198359s
```

Ngoài ra, **Bucket4j** là một rate limiting library rất tốt dựa trên thuật toán token/leaky bucket.

> Địa chỉ Bucket4j: <https://github.com/vladimir-bukhtoyarov/bucket4j>

So với utility class rate limiting của Guava, chức năng rate limiting của Bucket4j toàn diện hơn. Nó không chỉ hỗ trợ single-instance rate limiting và distributed rate limiting mà còn có thể tích hợp monitoring, kết hợp với Prometheus và Grafana.

Tuy nhiên, Guava dù chỉ là một utility library có chức năng toàn diện, chức năng rate limiting dùng ngay mà nó cung cấp vẫn khá hữu ích trong nhiều single-instance scenario.

Rate limiter tích hợp sẵn `RedisRateLimiter` của Spring Cloud Gateway được triển khai dựa trên Redis + Lua. Bucket4j và Resilience4j chủ yếu tích hợp với gateway thông qua extension độc lập hoặc component trong Spring ecosystem, không phải là quan hệ tiến hóa thay thế cho rate limiter tích hợp sẵn của Spring Cloud Gateway.

Resilience4j là một fault tolerance component nhẹ, lấy cảm hứng từ Hystrix. Sau khi [Netflix tuyên bố không còn tích cực phát triển Hystrix](https://github.com/Netflix/Hystrix/commit/a7df971cbaddd8c5e976b3cc5f14013fe6ad00e6), Spring và Netflix đều khuyến nghị dùng Resilience4j hơn để triển khai rate limiting và circuit breaker.

> Địa chỉ Resilience4j: <https://github.com/resilience4j/resilience4j>

Thông thường, để đảm bảo high availability của hệ thống, project nên triển khai rate limiting và circuit breaker cùng nhau.

Resilience4j không chỉ cung cấp rate limiting mà còn cung cấp các chức năng dùng ngay để đảm bảo high availability cho hệ thống như circuit breaker, load protection và automatic retry. Ngoài ra, Resilience4j có ecosystem tốt hơn, nhiều gateway cũng dùng Resilience4j để triển khai rate limiting và circuit breaker.

Vì vậy, trong phần lớn scenario, Resilience4j có thể là lựa chọn tốt hơn. Nếu chỉ là scenario rate limiting tương đối đơn giản, Guava hoặc Bucket4j cũng là lựa chọn tốt.

## Triển khai distributed rate limiting thế nào?

Distributed rate limiting áp dụng cho distributed/microservice architecture. Trong architecture này, một service có thể được deploy thành nhiều instance. Nếu cần kiểm soát quota toàn cục, phải để nhiều instance chia sẻ state rate limiting. Tuy nhiên, không phải mọi microservice scenario đều bắt buộc dùng distributed rate limiting: nếu quota có thể chia đều theo số instance (ví dụ tổng quota 1000 QPS, 10 instance mỗi instance giới hạn 100 QPS), single-instance rate limiting lại có ưu thế về performance và độ phức tạp triển khai.

Các giải pháp distributed rate limiting phổ biến:

- **Nhờ middleware thực hiện rate limiting**: có thể dùng Sentinel hoặc tự dùng Redis để triển khai logic rate limiting tương ứng.
- **Rate limiting ở gateway layer**: một giải pháp khá phổ biến, triển khai rate limiting trực tiếp ở gateway layer. Tuy nhiên, rate limiting ở gateway layer thường cũng cần middleware/framework hỗ trợ. Ví dụ, implementation distributed rate limiting `RedisRateLimiter` của Spring Cloud Gateway dựa trên Redis + Lua, và Spring Cloud Gateway cũng có thể tích hợp Sentinel để triển khai rate limiting.

Nếu muốn tự triển khai logic rate limiting dựa trên Redis, nên kết hợp với Lua script.

**Vì sao khuyến nghị dùng Redis + Lua?** Chủ yếu có hai lý do:

- **Giảm network overhead**: có thể dùng Lua script để thực thi hàng loạt nhiều Redis command. Các Redis command này được gửi đến Redis server và thực thi một lần, giúp giảm đáng kể network overhead.
- **Tính atomic**: một Lua script có thể được xem như một command để thực thi. Trong quá trình thực thi Lua script, không có script hoặc Redis command khác được thực thi đồng thời, đảm bảo operation không bị instruction khác chèn vào hoặc làm gián đoạn.

Cần lưu ý, Lua script được thực thi atomic trong Redis, đồng nghĩa trong thời gian script chạy, các operation khác của Redis sẽ bị block. Vì vậy, rate limiting script nên nhẹ nhất có thể, tránh tính toán phức tạp hoặc duyệt trên phạm vi lớn, đồng thời kết hợp cấu hình `lua-time-limit` của Redis để tránh thời gian thực thi script mất kiểm soát.

Ở đây không đưa code rate limiting script cụ thể vì trên mạng đã có nhiều rate limiting script tốt có sẵn để tham khảo. Ví dụ, rate limiting plugin `RateLimiter` của project gateway Apache ShenYu triển khai các thuật toán token bucket/concurrent token bucket, leaky bucket và sliding window dựa trên Redis + Lua.

> Địa chỉ ShenYu: <https://github.com/apache/incubator-shenyu>

![Rate limiting script của ShenYu](https://oss.javaguide.cn/github/javaguide/high-availability/limit-request/shenyu-ratelimit-lua-scripts.png)

Ngoài ra, nếu không muốn tự viết Lua script, có thể dùng trực tiếp `RRateLimiter` trong Redisson để triển khai distributed rate limiting. Implementation bên trong dựa trên Lua code + thuật toán token bucket.

Redisson là Redis client mã nguồn mở cho Java, cung cấp nhiều chức năng dùng ngay như implementation các data structure thường dùng trong Java, distributed lock, delay queue... Ngoài ra, Redisson còn hỗ trợ nhiều deployment architecture như Redis single-instance, Redis Sentinel và Redis Cluster.

Cách dùng `RRateLimiter` rất đơn giản. Trước hết, lấy một object `RRateLimiter` bằng cách lấy trực tiếp từ Redisson client. Sau đó chỉ cần thiết lập rule rate limiting.

```java
// Tạo một instance Redisson client
RedissonClient redissonClient = Redisson.create();
// Lấy object rate limiter có tên "javaguide.limiter"
RRateLimiter rateLimiter = redissonClient.getRateLimiter("javaguide.limiter");
// Thử đặt tốc độ rate limiter là 100 lần mỗi giờ
// RateType có hai loại, OVERALL là rate limiting toàn cục, PER_CLIENT là rate limiting theo từng Client (có thể xem là single-instance rate limiting)
rateLimiter.trySetRate(RateType.OVERALL, 100, 1, RateIntervalUnit.HOURS);
```

Tiếp theo, gọi method `acquire()` hoặc `tryAcquire()` để lấy permission.

```java
// Lấy một permission, nếu vượt quá tốc độ của rate limiter thì sẽ chờ
// acquire() là method đồng bộ, method bất đồng bộ tương ứng: acquireAsync()
rateLimiter.acquire(1);
// Thử lấy một permission trong 5 giây, trả về true nếu thành công, ngược lại trả về false
// tryAcquire() là method đồng bộ, method bất đồng bộ tương ứng: tryAcquireAsync()
boolean res = rateLimiter.tryAcquire(1, 5, TimeUnit.SECONDS);
```

Độ chính xác rate limiting của `RRateLimiter` bị ảnh hưởng bởi performance và network latency của Redis. Trong scenario concurrency rất cao có thể xảy ra vượt giới hạn nhẹ. Khi Redis gặp sự cố hoặc chuyển đổi master-slave, `tryAcquire()` có thể ném exception. Business layer cần xác định trước degradation strategy: chuyển về local rate limiting, từ chối trực tiếp hay cho phép trong thời gian ngắn; lựa chọn phụ thuộc vào sự đánh đổi giữa stability và availability của business.

## Tổng kết

Bài viết này chủ yếu giới thiệu các thuật toán rate limiting phổ biến, cách chọn đối tượng rate limiting, và cách triển khai single-instance rate limiting cũng như distributed rate limiting.

## Tham khảo

- Lightweight circuit breaker framework Resilience4j trong service governance: <https://xie.infoq.cn/article/14786e571c1a4143ad1ef8f19>
- Giải thích siêu chi tiết về nguyên lý rate limiting của Guava RateLimiter: <https://cloud.tencent.com/developer/article/1408819>
- Thực chiến Spring Cloud Gateway: phần rate limiting 👍: <https://www.aneasystone.com/archives/2020/08/spring-cloud-gateway-current-limiting.html>
- Giải thích chi tiết về nguyên lý triển khai distributed rate limiting của Redisson: <https://juejin.cn/post/7199882882138898489>
- Một bài viết giải thích chi tiết cách triển khai API rate limiting Java - Alibaba Cloud Developer: <https://mp.weixin.qq.com/s/A5VYjstIDeVvizNK2HkrTQ>
- Khảo sát và thực hành các giải pháp distributed rate limiting - Tencent Cloud Developer: <https://mp.weixin.qq.com/s/MJbEQROGlThrHSwCjYB_4Q>

<!-- @include: @article-footer.snippet.md -->
