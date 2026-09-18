---
title: "Giải thích chi tiết cơ chế timeout và retry: thiết lập timeout, exponential backoff, jitter ngẫu nhiên, retry storm và idempotency"
description: "Giải thích chi tiết cơ chế timeout và retry, bao gồm connect timeout, read timeout, thiết lập timeout, retry theo khoảng cố định, linear backoff, exponential backoff, jitter ngẫu nhiên, retry storm, hiệu ứng avalanche, thiết kế idempotency và các framework retry Java như Spring Retry, Resilience4j."
category: High Availability
icon: "mdi:reload"
tag:
  - Timeout và Retry
  - High Availability
head:
  - - meta
    - name: keywords
      content: cơ chế timeout,cơ chế retry,thiết lập timeout,connect timeout,read timeout,exponential backoff,jitter ngẫu nhiên,retry storm,hiệu ứng avalanche,idempotency,câu hỏi phỏng vấn timeout và retry,Spring Retry,Resilience4j
---

Do tính bất định của network jitter, hardware failure, process exception, dependency service không khả dụng và các vấn đề khác, hệ thống hoặc service của chúng ta không thể luôn duy trì trạng thái khả dụng.

Để giảm tối đa ảnh hưởng do sự cố hệ thống hoặc service, chúng ta cần dùng cơ chế **timeout** và **retry**.

Ý tưởng cốt lõi của timeout và retry không khó hiểu, nhưng sử dụng đúng trong production lại có nhiều điểm cần lưu ý. Phần lớn hệ thống hoặc service có remote call mà bạn thường gặp đều áp dụng timeout và retry. Đặc biệt với microservice system, thiết lập timeout và retry đúng cách rất quan trọng. Monolithic service thường chỉ có network call tới database, cache, third-party API, middleware..., còn giữa các service trong microservice system cũng có network call.

## Cơ chế timeout

![Cơ chế timeout](https://oss.javaguide.cn/github/javaguide/high-availability/timeout-mechanism-overview.png)

### Cơ chế timeout là gì?

**Cơ chế timeout** nghĩa là khi một request vượt quá thời gian chỉ định (ví dụ 1s) mà vẫn chưa được xử lý, request đó sẽ bị hủy trực tiếp và ném ra exception hoặc error chỉ định (ví dụ `504 Gateway Timeout`).

Timeout mà chúng ta thường gặp có thể chia đơn giản thành 2 loại:

| Loại timeout        | Mô tả                                                                          | Giá trị đề xuất |
| ------------------- | ------------------------------------------------------------------------------ | --------------- |
| **Connect Timeout** | Thời gian chờ tối đa để client thiết lập TCP connection với server             | 1000ms ~ 5000ms |
| **Read Timeout**    | Thời gian tối đa client chờ peer trả dữ liệu sau khi connection được thiết lập | 1000ms ~ 3000ms |

Khi mới bắt đầu, bạn có thể hiểu connect timeout và read timeout trước, nhưng **production còn phải xem client có hỗ trợ cấu hình timeout theo nhiều chiều hơn không**:

| Giai đoạn timeout                   | Mô tả                                                              |
| ----------------------------------- | ------------------------------------------------------------------ |
| DNS resolution timeout              | Thời gian chờ tối đa để phân giải domain                           |
| Connection pool acquisition timeout | Thời gian chờ tối đa để lấy connection khả dụng từ connection pool |
| Connect Timeout                     | Thời gian tối đa để thiết lập TCP connection                       |
| TLS handshake timeout               | Thời gian chờ tối đa trong giai đoạn TLS/SSL handshake             |
| Write Timeout                       | Thời gian tối đa để ghi request body                               |
| Read / Response Timeout             | Thời gian chờ tối đa để đọc response data                          |
| Call Timeout / Deadline             | Tổng timeout của toàn bộ call, bao gồm mọi giai đoạn               |
| Idle connection cleanup timeout     | Thời gian sống tối đa của idle connection trong connection pool    |

> Phạm vi bao phủ của read timeout, response timeout và call timeout khác nhau giữa các HTTP/RPC client. Khi cấu hình, bắt buộc phải tham khảo tài liệu của implementation client cụ thể.

### Vì sao cần cơ chế timeout?

Nếu không thiết lập timeout, có thể xảy ra vấn đề **số lượng connection phía server bùng nổ** và **nhiều request bị dồn ứ**.

Các connection và request bị dồn ứ này sẽ tiêu tốn tài nguyên hệ thống, ảnh hưởng đến việc xử lý request mới. Trong trường hợp nghiêm trọng, chúng thậm chí có thể làm sập toàn bộ hệ thống hoặc service.

> Trước đây tôi từng gặp vấn đề tương tự trong một project thực tế: toàn bộ website không thể xử lý request bình thường, tải server gần như bị đẩy lên mức tối đa. Sau đó phát hiện nguyên nhân là cấu hình timeout sai kết hợp với việc xử lý request của client bất thường, khiến số connection phía server lên gần 400.000; lượng connection tồn đọng lớn như vậy đã trực tiếp làm sập toàn bộ hệ thống.

### Nên thiết lập timeout bao lâu?

Timeout nên dài bao lâu là vấn đề cần phán đoán dựa trên business scenario. **Đặt timeout quá cao hoặc quá thấp đều có rủi ro**:

| Cách thiết lập | Rủi ro                                                                                                                                          |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Quá cao**    | Giảm hiệu quả của cơ chế timeout, hệ thống vẫn có thể tích tụ nhiều slow request                                                                |
| **Quá thấp**   | Khi tốc độ xử lý hệ thống giảm (ví dụ request tăng đột ngột), nhiều request timeout rồi retry, làm tăng áp lực hệ thống và có thể gây avalanche |

Thông thường, chúng tôi đề xuất:

- **Read timeout**: Với internal RPC call của online service nhạy với latency, trước tiên có thể dựa trên **P99/P999 latency** lịch sử, **SLO** của critical path và tỷ lệ false timeout có thể chấp nhận để đặt giá trị ban đầu, sau đó điều chỉnh bằng load test. Ví dụ, kinh nghiệm của AWS là trước tiên xác định tỷ lệ false timeout có thể chấp nhận (chẳng hạn 0,1%), rồi chọn latency percentile tương ứng (như P99.9) làm điểm bắt đầu cho timeout.
- **Connect timeout**: Cần phân biệt theo network environment. Call qua mạng nội bộ cùng data center thường nên ngắn hơn; call cross-region, public network, proxy chain hoặc TLS cold connection có thể cần dài hơn. Trong scenario dùng connection pool cũng cần phân biệt lần đầu thiết lập connection và connection reuse; nên đặt trong khoảng **1000ms ~ 5000ms**.

**Không có silver bullet!** Timeout cụ thể nên đặt bao nhiêu vẫn phải được điều chỉnh và tối ưu dần dựa trên nhu cầu và tình trạng của project thực tế.

Trong microservice call chain, còn phải tuân theo một nguyên tắc: **timeout của downstream phải nhỏ hơn timeout của upstream**, đồng thời chừa đủ thời gian cho network overhead và local processing.

Cách được khuyến nghị hơn là truyền **deadline hoặc timeout budget**: request đầu vào có một tổng time budget; qua mỗi layer, cần trừ đi thời gian local processing, queueing, network và downstream call. Timeout của downstream phải nhỏ hơn budget còn lại của caller, thay vì mỗi layer tùy tiện cố định 3s, 2s, 1s. Nếu không, upstream đã timeout và trả về nhưng downstream vẫn tiếp tục thực thi request không còn tác dụng, dễ làm lãng phí thêm tài nguyên.

![Truyền timeout budget trong microservice call chain](https://oss.javaguide.cn/github/javaguide/high-availability/timeout-and-retry-optimized-timeout-budget-chain.png)

Tiến thêm một bước, tham khảo ý tưởng [cấu hình động các tham số Java thread pool của Meituan](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html), chúng ta cũng có thể biến timeout thành **parameter có thể cấu hình** thay vì cố định. Một cách khá đơn giản là đặt giá trị timeout trong configuration center. Nhờ vậy, có thể điều chỉnh động timeout theo trạng thái của hệ thống hoặc service.

> Việc điều chỉnh động cấu hình timeout cần gray release, đồng thời theo dõi P99/P999 latency, tỷ lệ timeout, thread pool queue, connection pool wait time và số lượng retry; không được trực tiếp triển khai rộng cho toàn bộ traffic.

### Quan hệ kết hợp giữa timeout, retry, circuit breaker và rate limiting

Chỉ thiết lập timeout và retry là chưa đủ. Timeout và retry là các biện pháp resilience, nhưng nếu downstream liên tục bất thường, retry ngược lại sẽ khuếch đại sự cố. Production thường cần kết hợp **timeout + retry + circuit breaker + rate limiting**:

| Cơ chế              | Trách nhiệm                              | Điểm chính                                                |
| ------------------- | ---------------------------------------- | --------------------------------------------------------- |
| **Timeout**         | Kịp thời giải phóng tài nguyên đang chờ  | Tránh request tích tụ vô hạn                              |
| **Retry**           | Xử lý transient failure                  | Bắt buộc giới hạn số lần, backoff và thêm Jitter          |
| **Rate Limit**      | Giới hạn lượng request vào hệ thống      | Bảo vệ downstream không bị traffic làm sập                |
| **Circuit Breaker** | Tránh tiếp tục gọi downstream bất thường | Fail fast, cho downstream thời gian khôi phục             |
| **Bulkhead**        | Giới hạn phạm vi ảnh hưởng của sự cố     | Cô lập bằng thread pool / semaphore, ngăn cascade failure |

## Cơ chế retry

![Cơ chế retry](https://oss.javaguide.cn/github/javaguide/high-availability/retry-mechanism-overview.png)

### Cơ chế retry là gì?

**Cơ chế retry** thường được dùng cùng cơ chế timeout, nghĩa là **thực hiện lại call cho cùng một business intent để tránh transient failure và intermittent failure**.

- **Transient failure**: sự cố tình cờ xuất hiện tại một thời điểm và không kéo dài.
- **Intermittent failure**: sự cố thỉnh thoảng xuất hiện trong một số tình huống, thường có tần suất thấp.

Retry là thực hiện lại call cho cùng một business intent, không nhất thiết là request giống hệt ở cấp byte. Vì vậy, khi liên quan đến signature, timestamp, request body stream và idempotency key, cần đảm bảo request retry vẫn hợp lệ và có thể được nhận diện là cùng một business operation.

Ý tưởng cốt lõi của retry là **tiêu tốn thêm tài nguyên để cố gắng tăng xác suất request được xử lý thành công**. Khi tần suất transient failure thấp và số client có giới hạn, load phát sinh từ retry thường có thể kiểm soát; nhưng khi tỷ lệ failure tăng hoặc nhiều client đồng thời trigger retry, traffic retry có thể trở thành đòn cuối cùng làm sập downstream, chính là **retry storm** sẽ được đề cập ở phần sau.

### Các retry strategy phổ biến là gì?

So sánh các retry strategy phổ biến:

| Strategy                          | Mô tả                                                         | Ưu điểm                            | Nhược điểm                                                          | Scenario phù hợp                                            |
| --------------------------------- | ------------------------------------------------------------- | ---------------------------------- | ------------------------------------------------------------------- | ----------------------------------------------------------- |
| **Retry theo khoảng cố định**     | Khoảng thời gian giữa mỗi lần retry giống nhau (ví dụ mỗi 1s) | Dễ triển khai                      | Có thể gây retry storm                                              | Thời gian khôi phục của target system ổn định và dễ dự đoán |
| **Retry với linear backoff**      | Khoảng thời gian tăng tuyến tính (ví dụ 1s, 2s, 3s)           | Nhẹ hơn khoảng cố định             | Tốc độ tăng chậm                                                    | Scenario thông thường                                       |
| **Retry với exponential backoff** | Khoảng thời gian tăng theo cấp số nhân (ví dụ 1s, 2s, 4s, 8s) | Nhẹ hơn khoảng cố định             | Nhiều client vẫn có thể retry đồng bộ, thời gian chờ có thể quá dài | Target system khôi phục lâu hoặc khó dự đoán                |
| **Exponential backoff có Jitter** | Thêm jitter ngẫu nhiên trên nền exponential backoff           | Tránh nhiều client retry đồng thời | Triển khai phức tạp hơn                                             | Được khuyến nghị trong distributed system                   |

**Trong distributed system, thường nên ưu tiên exponential backoff có Jitter**. Jitter có thể được triển khai dưới dạng full jitter (jitter hoàn toàn ngẫu nhiên), equal jitter (jitter chia đều) hoặc decorrelated jitter (jitter phi tương quan); mục tiêu đều là phân tán các lần retry đồng bộ của nhiều client.

![So sánh timeline của bốn retry strategy](https://oss.javaguide.cn/github/javaguide/high-availability/timeout-and-retry-optimized-retry-strategies-timeline.png)

### Nên đặt số lần retry bao nhiêu?

Không nên retry quá nhiều lần, nếu không vẫn sẽ gây áp lực lớn lên system load.

**Số lần retry của synchronous request online thường cần được kiểm soát chặt, phổ biến là 1～3 lần**. Ví dụ, nếu retry 3 lần:

- Sau khi request lần 1 thất bại, chờ 1 giây rồi retry.
- Sau khi request lần 2 thất bại, chờ 2 giây rồi retry.
- Sau khi request lần 3 thất bại, chờ 4 giây rồi retry.

Async task hoặc MQ consumer có thể retry nhiều hơn, nhưng bắt buộc phải kết hợp backoff, số lần retry tối đa, dead-letter queue, alert và manual compensation.

Retry nhiều lần không phải lúc nào cũng tốt. Giả sử read timeout là 1.5s, retry 3 lần và dùng exponential backoff 1s, 2s, 4s, tổng thời gian tối đa khoảng `1.5s × 4 + 1s + 2s + 4s = 13s`. Với online request, tail latency này có thể đã không thể chấp nhận. Vì vậy, read operation có thể retry tương đối tích cực, còn write operation bắt buộc kết hợp thiết kế idempotency; non-critical path cũng có thể chọn fail fast.

### Chỉ retry lỗi có thể retry

Bước quan trọng nhất khi triển khai retry là **phân biệt lỗi có thể retry và lỗi không thể retry**. Retry mù quáng sẽ lãng phí tài nguyên, thậm chí làm vấn đề nghiêm trọng hơn:

| Phân loại            | Scenario điển hình                                                          | Cách xử lý                                                  |
| -------------------- | --------------------------------------------------------------------------- | ----------------------------------------------------------- |
| **Có thể retry**     | Connect timeout, read timeout, connection reset, 502/503/504 tạm thời       | Retry kết hợp backoff strategy                              |
| **Có thể retry**     | 429 có response header `Retry-After` sau khi bị rate limit                  | Chờ theo `Retry-After` rồi retry                            |
| **Retry thận trọng** | Write operation như thanh toán, đặt hàng, trừ tồn kho, phát coupon          | Bắt buộc có idempotency key hoặc business unique constraint |
| **Không nên retry**  | Lỗi parameter (400), authorization failure (401/403), business rule failure | Trả về error trực tiếp                                      |
| **Không nên retry**  | Business exception như số dư không đủ, tồn kho không đủ                     | Trả về error trực tiếp                                      |

Thông thường chỉ retry timeout, connection bị ngắt, 5xx tạm thời và response được xác nhận cho phép retry sau khi rate limit; không retry lỗi validation parameter, authorization failure, business rule failure, 4xx rõ ràng và write operation không thể đảm bảo idempotency.

![Trước khi retry cần đánh giá: error type + operation idempotency](https://oss.javaguide.cn/github/javaguide/high-availability/timeout-and-retry-optimized-retry-idempotency-decision.png)

### Retry có những rủi ro nào?

Mặc dù cơ chế retry có thể tăng availability của hệ thống, sử dụng không đúng cũng gây ra rủi ro:

| Rủi ro                  | Mô tả                                                                                | Cách phòng tránh                                  |
| ----------------------- | ------------------------------------------------------------------------------------ | ------------------------------------------------- |
| **Retry storm**         | Nhiều client retry đồng thời, tiếp tục làm sập downstream service                    | Dùng exponential backoff có Jitter                |
| **Avalanche effect**    | Retry khiến upstream service cũng bắt đầu timeout rồi retry, tạo phản ứng dây chuyền | Thiết lập retry budget và circuit breaker         |
| **Duplicate operation** | Operation không idempotent bị thực thi lặp, gây inconsistent data                    | Đảm bảo operation idempotency                     |
| **Lãng phí tài nguyên** | Retry vô nghĩa với permanent failure                                                 | Phân biệt lỗi có thể retry và lỗi không thể retry |

**Retry Budget** là một strategy phòng tránh hiệu quả: giới hạn tỷ lệ số lần retry trên tổng số request trong một time window, chẳng hạn không vượt quá 10%. Khi triển khai, có thể thống kê original request và retry request theo caller, interface hoặc downstream; khi tỷ lệ retry vượt budget, các request tiếp theo sẽ fail fast hoặc chỉ cho phép lần thử đầu tiên, không retry nữa.

![Retry storm: khuếch đại request theo cấp số nhân](https://oss.javaguide.cn/github/javaguide/high-availability/timeout-and-retry-optimized-retry-storm-cascade.png)

### Vì sao retry bắt buộc phải xét đến idempotency?

Khi sử dụng timeout và retry trong project thực tế, cần đảm bảo **việc thực thi một lần hay nhiều lần cùng một operation tạo ra ảnh hưởng giống nhau lên system state**, đó chính là idempotency. Deduplication nhằm ngăn thực thi lặp, còn idempotency cho phép request lặp đến nhưng vẫn đảm bảo kết quả nhất quán; hai cơ chế thường được dùng kết hợp.

Khi nào một request có thể bị thực thi nhiều lần? Client chờ server hoàn thành request nhưng timeout, trong khi server đã thực thi request; chỉ là do network jitter ngắn hạn khiến response bị trễ trong quá trình gửi tới client.

> Ví dụ: người dùng thanh toán để mua một khóa học, nhưng request thanh toán bị retry khiến người dùng thanh toán hai lần cho cùng một khóa học. Với tình huống này, khi xử lý request mua khóa học, cần kiểm tra xem người dùng đã mua hay chưa. Như vậy sẽ không phát sinh việc mua trùng do retry.

Các cách phổ biến để triển khai idempotency:

| Cách thức                      | Mô tả                                                              | Scenario phù hợp                    |
| ------------------------------ | ------------------------------------------------------------------ | ----------------------------------- |
| **Unique request ID**          | Mỗi request mang một ID duy nhất, server deduplicate               | Scenario phổ biến                   |
| **Database unique constraint** | Dùng unique index của database để ngăn insert trùng                | Operation tạo mới                   |
| **Optimistic lock**            | Kiểm soát update thông qua version number                          | Operation update                    |
| **State machine**              | Kiểm soát bằng state transition, state đã xử lý sẽ không xử lý lại | Order, payment và các scenario khác |

Xét theo semantics của HTTP, RFC 9110 định nghĩa PUT, DELETE và safe methods (GET, HEAD, OPTIONS, TRACE) là các idempotent method. POST mặc định không có idempotent semantics, nhưng có thể triển khai business idempotency bằng idempotency key, business unique constraint và các cách khác. Tuy nhiên, protocol semantics không có nghĩa implementation phía server chắc chắn an toàn; với các operation có side effect như tạo order, trừ tiền, phát coupon và gửi message, server vẫn cần thiết kế idempotency một cách rõ ràng.

### Triển khai retry trong Java như thế nào?

Nếu tự viết code cho retry logic, có thể dùng vòng lặp `for` / `while` hoặc recursion. Tuy nhiên, thông thường không nên tự triển khai; nhiều open-source library bên thứ ba cung cấp implementation retry hoàn thiện hơn:

| Framework                             | Đặc điểm                                                                     | Version mới nhất   | Scenario phù hợp                                                                                                                                   |
| ------------------------------------- | ---------------------------------------------------------------------------- | ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Spring Retry**                      | Hệ sinh thái Spring, điều khiển bằng annotation, cấu hình đơn giản           | 2.0.12             | Project Spring Boot 2.x/3.x                                                                                                                        |
| **Spring Framework 7 built-in retry** | Tích hợp native `@Retryable`, không cần dependency bổ sung                   | Spring Framework 7 | Project Spring Boot 4.x                                                                                                                            |
| **Resilience4j**                      | Lightweight, phong cách functional, hỗ trợ circuit breaker, rate limiting... | 2.4.0              | Microservice project                                                                                                                               |
| **Failsafe**                          | Zero dependency, hỗ trợ async retry, timeout, circuit breaker...             | 3.3.2              | Scenario cần fine-grained control                                                                                                                  |
| **Guava Retrying**                    | Cấu hình retry strategy linh hoạt                                            | 2.0.0              | Java project phổ biến (⚠️ đã lâu không được cập nhật, không phải core module chính thức của Guava, cần đánh giá maintenance status trước khi dùng) |

#### Ví dụ Spring Retry

Ví dụ đơn giản sử dụng Spring Retry:

```java
@Configuration
@EnableRetry
public class RetryConfig {
}

@Retryable(
    retryFor = {RestClientException.class},
    maxAttempts = 3,
    backoff = @Backoff(delay = 1000, multiplier = 2)
)
public String callRemoteService(String requestId) {
    return remoteClient.call(requestId);
}

@Recover
public String recover(RestClientException e, String requestId) {
    // Logic dự phòng sau khi retry hết lượt
    return "fallback";
}
```

Tham số đầu tiên của `@Recover` là exception type, các tham số tiếp theo phải giống tham số của method `@Retryable`; Spring Retry match recovery method thông qua parameter type.

Cần lưu ý:

- `@Retryable` dựa trên Spring AOP proxy; call `this.method()` bên trong cùng một class sẽ không trigger retry, mà phải call thông qua proxy do Spring quản lý.
- Trong hệ sinh thái **Spring Boot 3.x / Spring Framework 6**, cần ưu tiên xác nhận tính tương thích giữa Spring Retry 2.x với version Spring của project và Java 17+; không dùng lẫn version cũ.
- Từ **Spring Boot 4.x / Spring Framework 7**, khả năng retry đã được tích hợp trong `spring-core`; dùng `@EnableResilientMethods` để kích hoạt, không cần thêm dependency `spring-retry`. Project Spring Retry đã chuyển sang maintenance mode (maintenance only), không tiếp nhận feature mới.

#### Ví dụ Resilience4j Retry

```java
RetryConfig config = RetryConfig.custom()
    .maxAttempts(3)
    .waitDuration(Duration.ofMillis(1000))
    .retryOnException(e -> e instanceof SocketTimeoutException
        || e instanceof ConnectException)
    .ignoreExceptions(BusinessException.class, AuthException.class)
    .intervalFunction(IntervalFunction.ofExponentialBackoff(1000, 2))
    .build();

Retry retry = Retry.of("remoteService", config);

// Lấy retry event để monitoring
retry.getEventPublisher()
    .onRetry(event -> log.warn("Retry attempt {}", event.getNumberOfRetryAttempts()));

String result = Retry.decorateSupplier(retry, () -> remoteClient.call(requestId))
    .get();
```

Khi kết hợp Retry với CircuitBreaker, cần lưu ý thứ tự decorator ảnh hưởng đến phạm vi thống kê: `Retry.decorateSupplier(CircuitBreaker.decorateSupplier(supplier))` nghĩa là retry thành công sẽ không mở circuit breaker; ngược lại, nếu đổi thứ tự wrapper thì mỗi lần retry đều được circuit breaker thống kê.

## Các metric cần theo dõi trong production

Sau khi timeout và retry được đưa lên production, bắt buộc phải dùng metric để xác minh; nếu không sẽ rất khó phát hiện các vấn đề như retry storm. Các metric quan trọng cần theo dõi:

| Nhóm metric       | Metric cụ thể                                                             |
| ----------------- | ------------------------------------------------------------------------- |
| Request           | Original request QPS, retry request QPS, tỷ lệ retry                      |
| Latency           | P99 / P999 latency, tỷ lệ timeout                                         |
| Success rate      | Final success rate, first-attempt success rate                            |
| Downstream health | Số lượng downstream 5xx, số lần rate limit, trạng thái circuit breaker    |
| Resource          | Độ sâu thread pool queue, connection pool wait time, số active connection |
| Retry budget      | Tỷ lệ tiêu thụ Retry Budget                                               |

## Tham khảo

- Quản lý thiết lập timeout giữa các microservice: <https://www.infoq.cn/article/eyrslar53l6hjm5yjgyx>
- Timeout, retry và jitter backoff (AWS Builders Library): <https://aws.amazon.com/cn/builders-library/timeouts-retries-and-backoff-with-jitter/>
- AWS Architecture Blog - Exponential Backoff and Jitter: <https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/>
- RFC 9110 - HTTP Semantics: <https://www.rfc-editor.org/rfc/rfc9110>

<!-- @include: @article-footer.snippet.md -->
