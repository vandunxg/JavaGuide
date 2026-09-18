---
title: "Giải thích chi tiết về Web real-time message push"
description: "Message push thường chỉ việc nhân viên vận hành website sử dụng một công cụ nào đó để chủ động push message đến webpage hiện tại của người dùng hoặc APP trên thiết bị di động."
category: System Design
icon: "mdi:message-text-outline"
head:
  - - meta
    - name: keywords
      content: "Web message push, real-time message, WebSocket, SSE, long polling, short polling, MQTT, giải pháp real-time communication"
---

> Địa chỉ bài viết gốc: <https://juejin.cn/post/7122014462181113887，JavaGuide> đã hoàn thiện và tổng hợp lại bài viết này.

Tôi có một người bạn xây dựng một website nhỏ, hiện muốn triển khai chức năng Web message push cho tin nhắn trong site. Đúng vậy, chính là chấm đỏ nhỏ trong hình dưới đây, một chức năng rất thường gặp.

![Web message push cho tin nhắn trong site](https://oss.javaguide.cn/github/javaguide/system-design/web-real-time-message-push/1460000042192380.png)

Tuy nhiên, bạn ấy vẫn chưa biết nên dùng cách nào. Vì vậy, tôi tổng hợp một số giải pháp và thực hiện đơn giản.

## Message push là gì?

Có khá nhiều trường hợp sử dụng push. Ví dụ, khi có người follow public account của tôi, tôi sẽ nhận được một message push để thu hút tôi click mở ứng dụng.

Message push thường chỉ việc nhân viên vận hành website sử dụng một công cụ nào đó để chủ động push message đến webpage hiện tại của người dùng hoặc APP trên thiết bị di động.

Message push thường được chia thành message push phía Web và message push phía mobile.

Ví dụ về message push phía mobile:

![Ví dụ về message push phía mobile](https://oss.javaguide.cn/github/javaguide/system-design/web-real-time-message-push/IKleJ9auR1Ojdicyr0bH.png)

Ví dụ về message push phía Web:

![Ví dụ về message push phía Web](https://oss.javaguide.cn/github/javaguide/system-design/web-real-time-message-push/image-20220819100512941.png)

Trước khi triển khai cụ thể, hãy phân tích lại requirement phía trên. Thực ra chức năng này rất đơn giản: chỉ cần khi một event nào đó được trigger (chủ động share resource hoặc backend chủ động push message), chấm đỏ thông báo trên Web page sẽ real-time `+1`.

Thông thường, backend sẽ có một số bảng message push để ghi lại các loại message khác nhau được push khi người dùng trigger các event khác nhau. Frontend chủ động query (pull) hoặc bị động nhận (push) tổng số message chưa đọc của người dùng.

![Bảng message push](https://oss.javaguide.cn/github/javaguide/system-design/web-real-time-message-push/1460000042192384.png)

Message push không ngoài hai hình thức push và pull. Hãy lần lượt tìm hiểu.

## Các giải pháp message push thường gặp

![Tổng quan các giải pháp Web real-time message push](https://oss.javaguide.cn/github/javaguide/system-design/web-real-time-message-push/web-real-time-message-push-overview.webp)

### Short polling

**Polling** có lẽ là cách đơn giản nhất để triển khai message push. Ở đây, tạm thời chia polling thành short polling và long polling.

Short polling khá dễ hiểu: trong một khoảng thời gian cố định, browser gửi HTTP request đến server, server real-time trả dữ liệu message chưa đọc về client, sau đó browser render và hiển thị dữ liệu.

Chỉ cần một JS timer đơn giản là có thể thực hiện: mỗi giây request một lần đến API lấy số message chưa đọc và hiển thị dữ liệu trả về.

```typescript
setInterval(() => {
  // Gọi phương thức
  messageCount().then((res) => {
    if (res.code === 200) {
      this.messageCount = res.data;
    }
  });
}, 1000);
```

Hiệu quả vẫn khá tốt. Short polling quả thực đơn giản, nhưng nhược điểm cũng rất rõ ràng: vì dữ liệu push không thường xuyên thay đổi, bất kể backend lúc đó có message mới hay không, client vẫn sẽ request, chắc chắn gây áp lực lớn cho server và lãng phí bandwidth cũng như resource của server.

### Long polling

Long polling là phiên bản cải tiến của short polling phía trên, vừa giảm lãng phí resource của server hết mức có thể, vừa bảo đảm tính real-time tương đối của message. Long polling được ứng dụng rất rộng rãi trong middleware, chẳng hạn Nacos và Apollo configuration center, cũng như message queue Kafka và RocketMQ đều sử dụng long polling.

Trong bài viết [Mô hình tương tác của Nacos configuration center là push hay pull?](https://mp.weixin.qq.com/s/94ftESkDoZI9gAGflLiGwg), tôi đã giới thiệu chi tiết nguyên lý triển khai long polling của Nacos. Nếu quan tâm, bạn có thể xem qua.

Nguyên lý của long polling thực ra gần giống polling, đều sử dụng cách polling. Tuy nhiên, nếu dữ liệu phía server không thay đổi, server sẽ **giữ** request cho đến khi dữ liệu thay đổi hoặc hết thời gian chờ thì mới trả về. Sau khi nhận response, client sẽ lập tức khởi tạo long polling tiếp theo.

Lần này, tôi dùng cách triển khai long polling của Apollo configuration center và sử dụng class `DeferredResult`. Đây là một cơ chế asynchronous request được Spring đóng gói sau Servlet 3.0, nghĩa trực tiếp là delayed result.

![Sơ đồ long polling](https://oss.javaguide.cn/github/javaguide/system-design/web-real-time-message-push/1460000042192386.png)

`DeferredResult` cho phép container giải phóng Servlet thread đang xử lý request hiện tại trước, sau đó application có thể chọn một thread bất kỳ, message callback hoặc event source khác để gọi `setResult()` và tiếp tục xử lý response. Bản thân `DeferredResult` không tự động khởi động một worker thread; business thực thi trên thread nào là do application quyết định.

Dưới đây, chúng ta dùng long polling để triển khai message push.

Vì một ID có thể được nhiều long polling request lắng nghe, tôi sử dụng cấu trúc `Multimap` do package Guava cung cấp để lưu long polling. Một key có thể tương ứng với nhiều value. Khi lắng nghe thấy key thay đổi, tất cả long polling tương ứng sẽ response. Sau khi frontend nhận version mới, frontend chủ động query API lấy số message chưa đọc và update dữ liệu trên page.

```java
@Controller
@RequestMapping("/polling")
public class PollingController {

    // Lưu tập hợp long polling đang lắng nghe một Id
    // Cấu trúc đồng bộ thread
    private static final Multimap<String, DeferredResult<String>> watchRequests =
            Multimaps.synchronizedMultimap(HashMultimap.create());
    // Version trong memory dùng cho demo; production thường nên dùng version persistent của chính business data
    private static final Map<String, Long> versions = new HashMap<>();

    /**
     * Thiết lập lắng nghe
     */
    @GetMapping(path = "watch/{id}")
    @ResponseBody
    public DeferredResult<String> watch(@PathVariable String id,
                                        @RequestParam(defaultValue = "0") long version) {
        // Thiết lập thời gian timeout cho deferred object
        DeferredResult<String> deferredResult = new DeferredResult<>(TIME_OUT, "timeout");
        // Xóa key khi asynchronous request hoàn tất để tránh memory leak
        deferredResult.onCompletion(() -> {
            watchRequests.remove(id, deferredResult);
        });
        // So sánh version và đăng ký lắng nghe phải nằm trong cùng một critical section,
        // tránh việc publish xảy ra ở giữa hai bước khiến notification bị mất
        synchronized (watchRequests) {
            long currentVersion = versions.getOrDefault(id, 0L);
            if (currentVersion != version) {
                deferredResult.setResult(Long.toString(currentVersion));
            } else {
                watchRequests.put(id, deferredResult);
            }
        }
        return deferredResult;
    }

    /**
     * Thay đổi dữ liệu
     */
    @PostMapping(path = "publish/{id}")
    @ResponseBody
    public String publish(@PathVariable String id) {
        // Trong cùng một synchronized block, trước tiên update version, sau đó xóa và copy snapshot của các listener
        Collection<DeferredResult<String>> deferredResults;
        long currentVersion;
        synchronized (watchRequests) {
            currentVersion = versions.merge(id, 1L, Long::sum);
            deferredResults = new ArrayList<>(watchRequests.removeAll(id));
        }
        for (DeferredResult<String> deferredResult : deferredResults) {
            deferredResult.setResult(Long.toString(currentVersion));
        }
        return "success";
    }
}
```

Ở đây, kết quả timeout của `DeferredResult` trả về giá trị quy ước sẵn là `"timeout"`. Sau khi nhận được giá trị này, frontend mang version cũ theo và lập tức thực hiện long polling tiếp theo. Khi nhận version mới, frontend trước tiên query business data mới nhất, sau đó dùng version này cho lần lắng nghe tiếp theo. Việc so sánh version, đăng ký lắng nghe và tăng version khi publish đều được thực hiện trong cùng một critical section. Vì vậy, ngay cả khi update xảy ra đúng lúc chuyển tiếp giữa hai request, lần lắng nghe tiếp theo vẫn lập tức phát hiện version thay đổi. Version trong memory ở ví dụ sẽ mất khi process restart; production nên ưu tiên dùng database version, message offset hoặc identifier persistent khác, đồng thời thiết kế strategy cleanup.

Không nên dùng HTTP 304 làm status “request timeout” thông thường: 304 chỉ dùng trong conditional request để biểu thị representation đã cache vẫn chưa thay đổi và không được chứa response body. Project production cũng có thể quy ước response không có content như 204; điều quan trọng là server và client thống nhất về semantics của timeout.

Hãy thử test: trước tiên page mang version đã biết gửi long polling request `/polling/watch/10086?version=0` để lắng nghe thay đổi message, request bị suspend. Ngay sau đó, chủ động thay đổi dữ liệu bằng `/polling/publish/10086`, long polling trả về version mới. Sau khi frontend query data mới nhất, frontend mang version mới gửi request tiếp theo và lặp lại như vậy.

So với short polling, long polling cải thiện performance rất nhiều, nhưng vẫn tạo ra khá nhiều request, đây là một điểm chưa hoàn hảo.

### iframe stream

iframe stream là việc chèn một tag `<iframe>` ẩn vào page, request API lấy số message trong `src`, từ đó tạo một persistent connection giữa server và client, để server liên tục truyền data đến `iframe`.

Dữ liệu truyền thường là HTML hoặc JavaScript được embed để đạt hiệu quả update page real-time.

![Sơ đồ iframe stream](https://oss.javaguide.cn/github/javaguide/system-design/web-real-time-message-push/1460000042192388.png)

Cách này đơn giản, frontend chỉ cần một tag `<iframe>` là xong.

```html
<iframe src="/iframe/message" style="display:none"></iframe>
```

Server cần giữ response và ghi dữ liệu mới vào khi có data mới, đồng thời kịp thời flush buffer. Không được dùng vòng lặp `while (true)` không có cơ chế chờ, interrupt và xử lý exception trong Servlet request thread để liên tục ghi response: cách này sẽ tiêu tốn CPU khi busy loop, chiếm container thread trong thời gian dài và không thể xử lý đúng khi client disconnect. Nếu bắt buộc phải duy trì iframe stream kiểu cũ, nên dùng asynchronous I/O của container và mô hình ghi theo event; system mới thường trực tiếp chọn SSE hoặc WebSocket.

iframe stream có server overhead rất lớn. Ngoài ra, browser như IE và Chrome sẽ luôn ở trạng thái loading, icon liên tục xoay, quả thực là cơn ác mộng của người mắc chứng cầu toàn.

![Hiệu quả của iframe stream](https://oss.javaguide.cn/github/javaguide/system-design/web-real-time-message-push/1460000042192389.png)

iframe stream rất kém thân thiện, đặc biệt không khuyến nghị.

### SSE (khuyến nghị)

Có thể nhiều người chưa biết: ngoài cơ chế quen thuộc là `WebSocket`, server push message đến client còn có một cơ chế khác là Server-Sent Events, viết tắt là SSE. Đây là cơ chế push message một chiều từ server đến client (browser).

Streaming conversation là một trường hợp sử dụng điển hình của SSE. Server có thể liên tục ghi phần content đã generate vào event stream, để user không cần chờ toàn bộ quá trình tính toán hoàn tất mới thấy kết quả.

![Conversation của ChatGPT sử dụng SSE](https://oss.javaguide.cn/github/javaguide/system-design/web-real-time-message-push/chatgpt-sse.png)

SSE dựa trên HTTP. Nó không để server tự tạo connection khi không có request, mà để client gửi request trước, sau đó server giữ HTTP response này và liên tục ghi dữ liệu.

![Minh họa SSE](https://oss.javaguide.cn/github/javaguide/system-design/web-real-time-message-push/1460000042192390.png)

SSE mở một channel một chiều giữa server và client. Response của server không còn là một data packet gửi một lần, mà là data stream kiểu `text/event-stream`, được stream từ server đến client khi data thay đổi.

Ý tưởng triển khai tổng thể hơi giống online video playback: video stream liên tục được push đến browser. Bạn cũng có thể hiểu đây là việc client thực hiện một lần download mất rất nhiều thời gian (do network không ổn định).

![Sơ đồ SSE](https://oss.javaguide.cn/github/javaguide/system-design/web-real-time-message-push/1460000042192391.png)

SSE và WebSocket có vai trò tương tự nhau: đều có thể tạo communication giữa server và browser, thực hiện server push message đến client. Tuy nhiên, chúng vẫn có một số khác biệt:

- SSE sử dụng HTTP và `text/event-stream`, thường dễ tích hợp hơn với Web service hiện có; WebSocket cần server hoặc container hỗ trợ protocol upgrade và WebSocket frame. SSE không yêu cầu bắt buộc phải deploy riêng một server.
- SSE là communication một chiều, chỉ server mới có thể communication một chiều đến client; WebSocket là full-duplex communication, nghĩa là hai phía có thể đồng thời gửi và nhận message.
- SSE đơn giản, development cost thấp, không cần thêm component; WebSocket cần parse data lần thứ hai, nên development barrier cao hơn một chút.
- SSE mặc định hỗ trợ reconnect khi disconnect; WebSocket cần tự triển khai.
- SSE chỉ truyền được text message, binary data cần encode trước khi truyền; WebSocket mặc định hỗ trợ truyền binary data.

![So sánh SSE và WebSocket](https://oss.javaguide.cn/github/javaguide/system-design/web-real-time-message-push/sse-vs-websocket-comparison.webp)

**Nên chọn SSE hay WebSocket?**

> Công nghệ không có tốt hay xấu, chỉ có phù hợp hay không.

SSE dường như chưa bao giờ được nhiều người biết đến. Một phần nguyên nhân là sự xuất hiện của WebSocket, cơ chế cung cấp protocol phong phú hơn để thực hiện bidirectional, full-duplex communication. Với game, instant messaging và các scenario cần update hai chiều gần real-time, channel hai chiều hấp dẫn hơn.

Tuy nhiên, trong một số trường hợp, client không cần gửi data. Bạn chỉ cần các update từ thao tác của server. Ví dụ: tin nhắn trong site, số message chưa đọc, status update, stock quote, số lượng monitor và các scenario khác. Xét cả độ khó lẫn cost triển khai, SSE đều có ưu thế hơn. Ngoài ra, SSE còn có nhiều capability mà WebSocket thiếu trong thiết kế, chẳng hạn auto reconnect, event ID và khả năng gửi event tùy ý.

Frontend chỉ cần thực hiện một HTTP request, mang theo ID duy nhất, mở event stream và lắng nghe event do server push.

```javascript
<script>
    let source = null;
    let userId = 7777
    if (window.EventSource) {
        // Thiết lập connection
        source = new EventSource('http://localhost:7777/sse/sub/'+userId);
        setMessageInnerHTML("Đã kết nối user=" + userId);
        /**
         * Khi connection được thiết lập sẽ trigger event open
         * Cách viết khác: source.onopen = function (event) {}
         */
        source.addEventListener('open', function (e) {
            setMessageInnerHTML("Đã thiết lập connection...");
        }, false);
        /**
         * Client nhận data từ server
         * Cách viết khác: source.onmessage = function (event) {}
         */
        source.addEventListener('message', function (e) {
            setMessageInnerHTML(e.data);
        });
    } else {
        setMessageInnerHTML("Trình duyệt của bạn không hỗ trợ SSE");
    }
</script>
```

Server triển khai đơn giản hơn: tạo một object `SseEmitter` và đưa vào `sseEmitterMap` để quản lý.

```java
private static Map<String, SseEmitter> sseEmitterMap = new ConcurrentHashMap<>();

/**
 * Tạo connection
 */
public static SseEmitter connect(String userId) {
    try {
        // 0 nghĩa là không thiết lập timeout ở application layer, không phải “mặc định 30 giây”
        SseEmitter sseEmitter = new SseEmitter(0L);
        // Đăng ký callback
        sseEmitter.onCompletion(completionCallBack(userId));
        sseEmitter.onError(errorCallBack(userId));
        sseEmitter.onTimeout(timeoutCallBack(userId));
        sseEmitterMap.put(userId, sseEmitter);
        count.getAndIncrement();
        return sseEmitter;
    } catch (Exception e) {
        log.info("Lỗi khi tạo SSE connection mới, user hiện tại: {}", userId);
    }
    return null;
}

/**
 * Gửi message cho user được chỉ định
 */
public static void sendMessage(String userId, String message) {

    if (sseEmitterMap.containsKey(userId)) {
        try {
            sseEmitterMap.get(userId).send(message);
        } catch (IOException e) {
            log.error("Lỗi khi push cho user [{}]:{}", userId, e.getMessage());
            removeUser(userId);
        }
    }
}
```

Ví dụ `Map<String, SseEmitter>` phía trên chỉ giữ một connection cho mỗi user; connection mới sẽ ghi đè connection cũ. Nếu cần hỗ trợ nhiều tab hoặc nhiều thiết bị, nên duy trì một nhóm `SseEmitter` cho mỗi user, đồng thời xóa connection tương ứng trong callback complete, timeout và error. `0L` cũng không có nghĩa proxy, gateway và container sẽ không bao giờ ngắt connection; production cần cấu hình heartbeat, timeout và strategy reconnect phía client.

**Lưu ý:** SSE không hỗ trợ browser IE, nhưng compatibility với các browser phổ biến khác khá tốt.

![Compatibility của SSE](https://oss.javaguide.cn/github/javaguide/system-design/web-real-time-message-push/1460000042192393.png)

### WebSocket

WebSocket có lẽ là một cách triển khai message push khá quen thuộc. Ở phần SSE phía trên, chúng ta cũng đã so sánh nó với WebSocket.

Đây là một protocol thực hiện full-duplex communication trên TCP connection để thiết lập channel communication giữa client và server. Browser và server chỉ cần một lần handshake là có thể trực tiếp tạo persistent connection giữa hai bên và truyền data hai chiều.

![Minh họa WebSocket](https://oss.javaguide.cn/github/javaguide/system-design/web-real-time-message-push/1460000042192394.png)

Quá trình hoạt động của WebSocket có thể chia thành các bước sau:

1. Client gửi một HTTP request đến server. Request header chứa các field như `Upgrade: websocket` và `Sec-WebSocket-Key`, biểu thị yêu cầu upgrade protocol thành WebSocket;
2. Sau khi nhận request, server thực hiện upgrade protocol. Nếu hỗ trợ WebSocket, server trả về HTTP status code 101, response header chứa các field như `Connection: Upgrade` và `Sec-WebSocket-Accept: xxx`, biểu thị upgrade lên WebSocket protocol thành công.
3. Client và server thiết lập một WebSocket connection để truyền data hai chiều. Data được truyền dưới dạng frame thay vì HTTP request và response truyền thống. Mỗi message WebSocket có thể bị chia thành nhiều data frame (đơn vị nhỏ nhất). Sender chia message thành nhiều frame gửi cho receiver; receiver nhận các message frame rồi lắp ghép các frame liên quan thành message hoàn chỉnh.
4. Client hoặc server có thể chủ động gửi một close frame để biểu thị muốn disconnect. Sau khi nhận được, phía còn lại cũng trả về một close frame, rồi hai bên đóng TCP connection.

Ngoài ra, sau khi thiết lập WebSocket connection, cần dùng heartbeat mechanism để duy trì tính ổn định và active của WebSocket connection.

Để tích hợp WebSocket vào SpringBoot, trước tiên import package liên quan đến WebSocket. So với SSE, cách này có development cost cao hơn.

```xml
<!-- Import websocket -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

Server dùng annotation `@ServerEndpoint` để đánh dấu class hiện tại là một WebSocket endpoint. Client có thể dùng `ws://localhost:7777/websocket/10086` để connect đến server. `userId` trong path chỉ phù hợp cho routing demo; trong production phải verify danh tính user khi handshake và bind connection với principal đã authenticated, không được tin `userId` do client tự điền.

```java
@Component
@Slf4j
@ServerEndpoint("/websocket/{userId}")
public class WebSocketServer {
    // Connection session với một client, dùng để gửi data cho client
    private Session session;
    private String userId;
    private static final CopyOnWriteArraySet<WebSocketServer> webSockets = new CopyOnWriteArraySet<>();
    // Dùng để lưu số connection online
    private static final Map<String, Session> sessionPool = new ConcurrentHashMap<>();
    /**
     * Method được gọi khi connection thành công
     */
    @OnOpen
    public void onOpen(Session session, @PathParam(value = "userId") String userId) {
        try {
            this.session = session;
            this.userId = userId;
            webSockets.add(this);
            sessionPool.put(userId, session);
            log.info("WebSocket message: có connection mới, tổng số là:" + webSockets.size());
        } catch (Exception e) {
            log.error("Khởi tạo WebSocket connection thất bại, userId={}", userId, e);
        }
    }
    /**
     * Dọn dẹp session hiện tại khi connection đóng
     */
    @OnClose
    public void onClose() {
        webSockets.remove(this);
        if (userId != null && session != null) {
            sessionPool.remove(userId, session);
        }
    }
    @OnError
    public void onError(Throwable error) {
        log.error("WebSocket connection exception, userId={}", userId, error);
    }
    /**
     * Method được gọi sau khi nhận message từ client
     */
    @OnMessage
    public void onMessage(String message) {
        log.info("WebSocket message: nhận message từ client:" + message);
    }
    /**
     * Đây là message point-to-point
     */
    public static boolean sendOneMessage(String userId, String message) {
        Session session = sessionPool.get(userId);
        if (session != null && session.isOpen()) {
            try {
                log.info("WebSocket message: message point-to-point:" + message);
                session.getAsyncRemote().sendText(message);
                return true;
            } catch (Exception e) {
                log.error("Gửi WebSocket message thất bại, userId={}", userId, e);
            }
        }
        return false;
    }
}
```

Ví dụ này cũng chỉ giữ một connection cho mỗi `userId` trong `sessionPool`. Khi hỗ trợ nhiều tab hoặc nhiều thiết bị, nên đổi value thành một session collection, đồng thời thực hiện send, heartbeat, backpressure và cleanup độc lập cho từng connection.

Nếu muốn trigger một lần server push thông qua HTTP API, có thể thêm một controller có parameter nhất quán với frontend:

```java
@RestController
@RequestMapping("/socket")
public class SocketController {

    @PostMapping("/publish")
    public ResponseEntity<Void> publish(@RequestParam String userId,
                                        @RequestParam String message) {
        return WebSocketServer.sendOneMessage(userId, message)
                ? ResponseEntity.accepted().build()
                : ResponseEntity.notFound().build();
    }
}
```

Controller này chỉ dùng để demo request contract. Trong production, vẫn phải authentication và authorization cho publish API; không được cho phép caller push message đến user khác thông qua `userId` tùy ý. Đồng thời cần giới hạn kích thước message và request rate.

Trong Spring Boot application sử dụng embedded Servlet container, thông thường còn cần inject `ServerEndpointExporter`. Component này đăng ký WebSocket endpoint sử dụng annotation `@ServerEndpoint`.

```java
@Configuration
public class WebSocketConfiguration {

    /**
     * Dùng để đăng ký WebSocket server sử dụng annotation @ServerEndpoint
     */
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }
}
```

Frontend khởi tạo và mở WebSocket connection, sau đó lắng nghe connection status, nhận data từ server hoặc gửi data đến server.

```javascript
<script>
    var ws = new WebSocket('ws://localhost:7777/websocket/10086');
    // Lấy connection status
    console.log('ws connection status: ' + ws.readyState);
    // Lắng nghe connection thành công
    ws.onopen = function () {
        console.log('ws connection status: ' + ws.readyState);
        // Gửi một data sau khi connection thành công
        ws.send('test1');
    }
    // Nhận và xử lý, hiển thị information server gửi về
    ws.onmessage = function (data) {
        console.log('Nhận message từ server:');
        console.log(data);
        // Đóng WebSocket connection sau khi hoàn tất communication
        ws.close();
    }
    // Lắng nghe event connection close
    ws.onclose = function () {
        // Lắng nghe trạng thái websocket trong toàn bộ quá trình
        console.log('ws connection status: ' + ws.readyState);
    }
    // Lắng nghe và xử lý event error
    ws.onerror = function (error) {
        console.log(error);
    }
    function sendMessage() {
        var content = $("#message").val();
        $.ajax({
            url: '/socket/publish',
            type: 'POST',
            data: { "userId": "10086", "message": content },
            success: function (data) {
                console.log(data)
            }
        })
    }
</script>
```

Sau khi page khởi tạo và thiết lập WebSocket connection, có thể thực hiện communication hai chiều. Hiệu quả khá tốt.

![](https://oss.javaguide.cn/github/javaguide/system-design/web-real-time-message-push/1460000042192395.png)

### MQTT

**MQTT protocol là gì?**

MQTT là một lightweight message protocol dựa trên mô hình publish/subscribe, lấy message thông qua việc subscribe topic và được ứng dụng rộng rãi trong IoT. Đặc tả OASIS hiện sử dụng trực tiếp tên “MQTT”, không còn mở rộng thành “Message Queue Telemetry Transport”.

Protocol này tách publisher và subscriber của message, nhờ đó có thể cung cấp reliable message service cho các thiết bị remote connection trong môi trường network không tin cậy. Cách sử dụng hơi giống MQ truyền thống.

![Ví dụ MQTT protocol](https://oss.javaguide.cn/github/javaguide/system-design/web-real-time-message-push/1460000022986325.png)

MQTT nằm ở application layer và cần chạy trên một ordered, lossless, bidirectional byte stream. Cách truyền tải phổ biến nhất là TCP (production thường kết hợp với TLS), cũng có thể thông qua transport như WebSocket, vốn cung cấp được semantics của byte stream này. Vì vậy, không nên đơn giản hóa thành “chỉ cần có TCP/IP là chắc chắn có thể dùng trực tiếp”.

**Tại sao cần dùng MQTT protocol?**

Tại sao MQTT protocol được ưa chuộng đến vậy trong IoT mà không phải protocol khác, chẳng hạn HTTP protocol quen thuộc hơn?

- Classic short-connection HTTP request-response model cần thiết bị request định kỳ hoặc duy trì long connection để lấy update từ server; MQTT trực tiếp cung cấp asynchronous publish/subscribe model trên long connection. Bản thân HTTP không thể bị khái quát là “synchronous protocol”; hành vi của HTTP/2, HTTP/3, SSE và WebSocket upgrade không giống short polling cổ điển.
- HTTP request do client khởi tạo, nhưng điều đó không có nghĩa server không thể stream data về hoặc thiết bị không thể nhận command. Ưu thế của MQTT là chuẩn hóa các capability như long connection, topic routing, subscription, QoS và session state.
- Khi cần gửi command đến nhiều thiết bị, MQTT broker có thể phân phối message đến tất cả subscriber theo topic; HTTP cũng có thể thực hiện chức năng tương tự, nhưng thường cần application tự quản lý connection, device group và retry semantics.

Phần giới thiệu và thực hành cụ thể về MQTT protocol, ở đây tôi không trình bày thêm. Bạn có thể tham khảo hai bài viết trước của tôi, trong đó cũng viết khá chi tiết.

- Giới thiệu MQTT protocol: [Tôi cũng không ngờ SpringBoot + RabbitMQ làm smart home lại đơn giản đến vậy](https://mp.weixin.qq.com/s/udFE6k9pPetIWsa6KeErrA)
- Dùng MQTT để triển khai message push: [Thực hành message chưa đọc (chấm đỏ nhỏ), frontend và RabbitMQ real-time message push, cực kỳ đơn giản](https://mp.weixin.qq.com/s/U-fUGr9i1MVa4PoVyiDFCg)

## Tổng kết

> Nội dung sau đây do JavaGuide bổ sung

|               | Giới thiệu                                                                                                                                                                                | Ưu điểm                                   | Nhược điểm                                                                                                |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| Short polling | Client định kỳ gửi request đến server, server trực tiếp trả response (kể cả khi không có data update)                                                                                     | Đơn giản, dễ hiểu, dễ triển khai          | Tính real-time kém, quá nhiều request không hiệu quả, việc tạo connection thường xuyên tốn nhiều resource |
| Long polling  | Khác short polling ở chỗ sau khi nhận request từ client, server chỉ trả request khi có data update                                                                                        | Giảm request không hiệu quả               | Request bị suspend gây lãng phí resource                                                                  |
| iframe stream | Tạo một persistent connection giữa server và client, server liên tục truyền data đến `iframe`.                                                                                            | Đơn giản, dễ hiểu, dễ triển khai          | Duy trì persistent connection làm tăng overhead, hiệu quả kém (icon liên tục xoay)                        |
| SSE           | Message push một chiều từ server đến client (browser).                                                                                                                                    | Đơn giản, dễ triển khai, nhiều capability | Không hỗ trợ communication hai chiều                                                                      |
| WebSocket     | Ngoài lúc khởi tạo connection dùng HTTP protocol, các thời điểm khác đều communication trực tiếp dựa trên TCP protocol, có thể thực hiện full-duplex communication giữa client và server. | Performance cao, overhead thấp            | Yêu cầu cao hơn với developer, triển khai tương đối phức tạp                                              |
| MQTT          | Lightweight communication protocol dựa trên mô hình publish/subscribe, lấy message thông qua việc subscribe topic tương ứng.                                                              | Mature, stable, lightweight               | Yêu cầu cao hơn với developer, triển khai tương đối phức tạp                                              |

<!-- @include: @article-footer.snippet.md -->
