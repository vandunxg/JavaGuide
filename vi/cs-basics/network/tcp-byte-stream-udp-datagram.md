---
title: "Vì sao TCP hướng byte stream, UDP hướng datagram? (Transport Layer)"
description: "Làm rõ khác biệt bản chất giữa byte stream của TCP và datagram của UDP, phân tích nguyên nhân cùng giải pháp của sticky packet/packet splitting, bao quát các chủ đề phỏng vấn thường gặp như Nagle, Delayed ACK."
category: Computer Basics
tag:
  - Computer Network
head:
  - - meta
    - name: keywords
      content: TCP,UDP,byte stream,datagram,sticky packet,packet splitting,message boundary,Nagle,Delayed ACK,TCP_NODELAY
---

Ở phần trước đã nói TCP hướng byte stream, UDP hướng datagram. Điểm này trông giống một câu định nghĩa, nhưng nhiều vấn đề sticky packet, packet splitting thực ra đều bắt nguồn từ đây.

Trước hết nói kết luận: **TCP chỉ đảm bảo byte đến nơi một cách tin cậy, đúng thứ tự, không đảm bảo ranh giới message của application layer; UDP sẽ giữ lại ranh giới datagram mà application layer giao cho nó.**

Bài viết này chủ yếu trả lời các câu hỏi:

1. Vì sao nói TCP hướng byte stream, UDP hướng datagram?
2. Sticky packet, packet splitting của TCP hình thành như thế nào?
3. Application layer nên định nghĩa ranh giới message ra sao?
4. Vì sao Nagle algorithm và Delayed ACK có thể khiến packet nhỏ chậm hơn?

Lấy một ví dụ, application layer gửi liên tiếp hai message:

```
Message 1: hello
Message 2: world
```

Nếu gửi bằng UDP, thông thường sẽ tương ứng với hai UDP datagram. Khi gọi `recvfrom()` ở phía nhận, dữ liệu cũng được đọc theo datagram: mỗi lần đọc một UDP datagram, không hợp nhất datagram của hai lần gửi thành một stream. Trong receive queue của UDP, mỗi phần tử là một datagram, vì vậy ranh giới message được giữ lại một cách tự nhiên.

Tuy nhiên, ở đây có một chi tiết: UDP giữ lại ranh giới datagram của transport layer, không có nghĩa nó phù hợp để gửi message tùy ý lớn. Khi datagram quá lớn, IP layer bên dưới vẫn có thể phân mảnh; khi buffer phía nhận quá nhỏ, cũng có thể xảy ra truncation. Vì vậy, “hướng datagram” của UDP không có nghĩa là “gửi lớn bao nhiêu cũng được”, mà là nó không trừu tượng hóa application data thành một byte stream liên tục như TCP. RFC 768 định nghĩa UDP là datagram mode và nêu rõ nó chỉ cung cấp cơ chế giao thức tối thiểu, không đảm bảo delivery tin cậy và loại bỏ trùng lặp.

Nếu gửi bằng TCP thì không thể hiểu như vậy. Application layer gọi `send()` hai lần chỉ đơn giản là ghi hai đoạn byte vào send buffer của kernel. Những byte này được gửi lúc nào, được gộp thành bao nhiêu TCP segment, phía đối diện đọc được bao nhiêu trong một lần `recv()`, đều không do hai lần `send()` này trực tiếp quyết định.

Ví dụ, phía nhận có thể đọc được một lần (sticky packet):

```
helloworld
```

Hoặc cũng có thể đọc qua nhiều lần (packet splitting):

```
hel
lowor
ld
```

Đây không phải TCP gặp lỗi, mà vốn là cách TCP hoạt động. TCP xử lý một byte stream liên tục; nó chỉ quan tâm các byte có đến nơi một cách tin cậy, đúng thứ tự hay không, không quan tâm “message thứ mấy” do application layer định nghĩa bắt đầu từ đâu và kết thúc ở đâu. RFC 9293 cũng nêu rõ TCP segment và ranh giới của `send()` / socket write ở application layer thường không tương ứng một-một; TCP không đảm bảo ranh giới buffer đọc/ghi của application liên quan đến ranh giới phân đoạn trên network.

![Ranh giới message của TCP và UDP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-udp-byte-stream-tcp-udp-message-boundary.png)

Vì vậy, cách gọi “TCP sticky packet/packet splitting” giống một hiện tượng nhìn từ góc độ application layer hơn. Nói chính xác, TCP không có khái niệm “packet”, nó truyền một byte stream liên tục. Điều thực sự cần giải quyết là: **application-layer protocol định nghĩa ranh giới message như thế nào.**

#### Vì sao sticky packet và packet splitting xuất hiện?

Có một số nguyên nhân thường gặp.

**1. TCP là byte stream protocol, không có ranh giới message của application layer.**

TCP chịu trách nhiệm đưa byte đến phía đối diện một cách tin cậy, đúng thứ tự, nhưng không ghi lại rằng “20 byte này là message đầu tiên, 30 byte kia là message thứ hai”.

**2. Một lần `send()` không tương đương một lần gửi trên network.**

`send()` thành công thường chỉ có nghĩa là data đã được copy từ application process vào send buffer của kernel. Còn khi nào thực sự được gửi đi, được tách thành bao nhiêu TCP segment, phải phụ thuộc vào MSS, send window, congestion window, Nagle algorithm, NIC queue và các yếu tố khác.

**3. Một lần `recv()` cũng không tương đương đọc được một message hoàn chỉnh.**

Phía nhận chỉ lấy byte từ TCP receive buffer. Buffer có thể đã chứa nhiều message, cũng có thể mới chỉ có nửa message. `recv()` chỉ copy data hiện có thể đọc cho application, không giúp bạn tách theo message nghiệp vụ.

**4. Tối ưu packet nhỏ có thể thay đổi thời điểm gửi.**

Các cơ chế như Nagle algorithm, Delayed ACK, Linux tự động hợp nhất các lần ghi nhỏ đều có thể ảnh hưởng đến thời điểm gửi data nhỏ. Ví dụ, từ Linux 3.14 đã có `tcp_autocorking`, kernel sẽ cố gắng hợp nhất các lần ghi nhỏ liên tiếp để giảm số packet gửi đi; application cũng có thể dùng `TCP_CORK` để kiểm soát rõ thời điểm “bỏ cork” và gửi.

Đó cũng là lý do protocol encoding/decoding rất quan trọng trong Netty, Dubbo, custom RPC, IM gateway và game service. Chỉ cần tầng bên dưới dùng TCP thì application layer bắt buộc phải định nghĩa rõ ranh giới message.

![Vì sao TCP sticky packet / packet splitting xuất hiện?](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-udp-byte-stream-tcp-sticky-split-causes.png)

#### Giải quyết TCP sticky packet/packet splitting như thế nào?

Chỉ có một ý tưởng cốt lõi: **để phía nhận biết một message kết thúc ở đâu.**

![Application layer định nghĩa ranh giới message như thế nào?](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-udp-byte-stream-tcp-message-boundary-solutions.png)

Có ba cách thường dùng.

**1. Độ dài cố định**

Quy định mỗi message có độ dài cố định, ví dụ 64 byte. Mỗi khi đọc đủ 64 byte, phía nhận coi đó là một message hoàn chỉnh.

Cách này dễ triển khai nhưng kém linh hoạt. Message ngắn phải được padding, gây lãng phí không gian; message dài lại phải tách thêm. Nó phù hợp với trường hợp format message rất cố định, không phù hợp lắm với protocol nghiệp vụ tổng quát.

**2. Delimiter**

Thêm delimiter đặc biệt giữa các message, ví dụ newline `\n`, `\r\n`, hoặc end marker tự định nghĩa.

```
hello\n
world\n
```

Phía nhận liên tục đọc data từ buffer; chỉ cần gặp delimiter thì tách ra một message hoàn chỉnh. Nhiều text protocol cũng dùng cách tương tự.

Cách này trực quan, nhưng cần chú ý hai vấn đề: thứ nhất, delimiter có thể vừa đúng xuất hiện trong message body, khi đó cần escape; thứ hai, bản thân delimiter cũng có thể bị tách giữa hai lần đọc, nên khi parse, phía nhận không được giả định một lần `recv()` luôn đọc đủ delimiter.

**3. Length header**

Đây là cách phổ biến hơn trong engineering. Đặt cố định một length field trong protocol header để biểu thị message body phía sau có bao nhiêu byte.

```
| 4 byte độ dài | message body |
```

Phía nhận trước tiên đọc protocol header có độ dài cố định, parse độ dài message body, sau đó tiếp tục đọc số byte đã chỉ định. Nếu chưa đọc đủ thì tiếp tục chờ; nếu đọc thừa thì giữ các byte dư trong buffer, làm phần đầu của message tiếp theo.

Nhiều binary protocol và RPC protocol dùng cách này. Khi thiết kế thực tế, protocol header thường không chỉ chứa length mà còn có các field như magic number, version, message type, sequence number, serialization method.

Length header cũng có điểm cần chú ý. Phải quy ước byte order cho length field, thường dùng network byte order; đồng thời phải giới hạn độ dài body tối đa để tránh phía đối diện gửi một giá trị length quá lớn làm cạn memory. Khi parse protocol trong production, không thể chỉ xét đường đi bình thường mà còn phải xử lý half packet, length bất thường, connection đóng giữa chừng, request được tạo độc hại và các trường hợp khác.

#### Vì sao Nagle algorithm và Delayed ACK khiến packet nhỏ chậm hơn?

Khi nói về sticky packet, người ta thường hỏi thêm về Nagle algorithm.

Mục tiêu của Nagle algorithm là giảm số lượng packet nhỏ. Khi băng thông network thời kỳ đầu còn hạn chế, nếu application mỗi lần chỉ ghi 1 byte nhưng header TCP/IP lại có hàng chục byte, network sẽ đầy các “packet nhỏ”, hiệu suất rất thấp. RFC 896 thảo luận về small-packet problem này và đề xuất rằng khi connection vẫn còn data chưa được xác nhận, data nhỏ mới có thể tạm hoãn gửi, đợi ACK đến rồi mới tiếp tục gửi.

Delayed ACK là một tối ưu ở phía nhận. Sau khi nhận data, phía nhận không nhất thiết gửi ACK ngay mà có thể đợi một khoảng thời gian ngắn, xem có thể gửi ACK cùng với data cần trả về hay không, nhằm giảm số pure ACK packet. RFC 9293 cũng gọi strategy “ít hơn một ACK cho mỗi data segment” này là delayed ACK.

Hai cơ chế này khi xem riêng đều có lý, nhưng khi kết hợp có thể khuếch đại latency. Tình huống điển hình là:

```
Client write data nhỏ A
Client ngay sau đó write data nhỏ B
Client chờ response từ server
```

![Vì sao Nagle + Delayed ACK có thể khiến packet nhỏ chậm hơn?](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-udp-byte-stream-nagle-delayed-ack-latency.png)

Data nhỏ A đã được gửi đi, data nhỏ B có thể bị Nagle algorithm tạm giữ trong send buffer để chờ ACK của A. Sau khi server nhận A, nếu tạm thời không có business response cần trả về thì Delayed ACK lại có thể trì hoãn việc gửi ACK. Vì vậy, sender chờ ACK, receiver chờ thêm data hoặc chờ delayed ACK timer, khiến latency bị khuếch đại.

Loại vấn đề này dễ nhận thấy hơn trong RPC ngắn, interactive protocol, game synchronization và remote terminal.

Cách giải quyết không phải là “mù quáng tắt Nagle”. Cách làm ổn định hơn là:

- Những lần ghi nhỏ có thể hợp nhất thì hợp nhất thành một message hoàn chỉnh ở application layer trước, rồi gọi `write()` một lần.
- Trong mô hình request/response, cố gắng tránh gọi `write()` nhỏ liên tiếp nhiều lần rồi lập tức chờ response.
- Với connection nhạy cảm về latency và có message rất nhỏ, có thể đánh giá việc bật `TCP_NODELAY` để data nhỏ được gửi sớm hơn.
- Với trường hợp ưu tiên throughput và muốn gom đủ data rồi mới gửi, có thể đánh giá `TCP_CORK` trên Linux, nhưng nó không phù hợp khi viết code cross-platform.
- Trước khi điều chỉnh parameter, hãy bắt packet để xác nhận, đừng thấy “chậm” là lập tức sửa socket option.

Trong Java, nhiều network framework sẽ expose config `TCP_NODELAY`, ví dụ `ChannelOption.TCP_NODELAY` của Netty. Nó thực sự có thể giảm thời gian chờ của message nhỏ, nhưng cũng có thể làm tăng số packet nhỏ. Với service QPS cao, cần xem trade-off này cùng với message size, RTT, throughput, CPU và số packet qua NIC. Linux `tcp(7)` cũng nêu rằng `TCP_NODELAY` sẽ tắt Nagle algorithm, còn `TCP_CORK` dùng để tránh gửi frame chưa hoàn chỉnh và đợi application xác nhận “có thể gửi” rồi mới gửi.

#### Trả lời thế nào khi phỏng vấn?

Có thể trả lời như sau:

TCP hướng byte stream. Data được application layer ghi vào buffer của kernel; TCP chỉ đảm bảo các byte này đến phía đối diện một cách tin cậy, đúng thứ tự, không đảm bảo một lần `send()` tương ứng một lần `recv()`, cũng không giữ ranh giới message của application layer. Vì vậy, phía nhận có thể đọc nhiều message trong một lần, cũng có thể chỉ đọc được nửa message. Đây là hiện tượng thường gọi là sticky packet, packet splitting.

UDP hướng datagram. Một lần data application layer giao cho UDP sẽ được gửi dưới dạng một UDP datagram; phía nhận cũng đọc theo datagram nên ranh giới message được giữ lại một cách tự nhiên. Tuy nhiên, UDP không đảm bảo đến nơi tin cậy và cũng không đảm bảo thứ tự.

Bản chất của việc giải quyết TCP sticky packet/packet splitting là application-layer protocol tự định nghĩa ranh giới message. Các cách thường gặp gồm độ dài cố định, delimiter và length header. Trong engineering, length header được dùng phổ biến hơn vì phù hợp hơn với binary protocol và message có độ dài thay đổi, nhưng phải xử lý byte order, giới hạn độ dài tối đa, buffer của half packet và việc connection đóng bất thường.

<!-- @include: @article-footer.snippet.md -->
