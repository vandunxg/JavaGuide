---
title: "Tổng hợp các phương thức tấn công mạng thường gặp (bảo mật)"
description: "Tổng hợp các cuộc tấn công TCP/IP thường gặp và tư duy phòng vệ, bao quát DDoS, giả mạo IP/ARP, man-in-the-middle và các phương thức khác, nhấn mạnh thực tiễn phòng vệ trong engineering."
category: CS Basics
tag:
  - Computer Network
head:
  - - meta
    - name: keywords
      content: Tấn công mạng,DDoS,giả mạo IP,giả mạo ARP,tấn công man-in-the-middle,quét,phòng vệ
---

> Bài viết này được biên soạn và hoàn thiện từ bài [Các phương thức tấn công TCP/IP thường gặp - Ghi chép Nuanlan - 2021](https://mp.weixin.qq.com/s/AZwWrOlLxRSSi-ywBgZ0fA).

TCP/IP protocol stack hướng tới khả năng liên thông, nhưng khi thiết kế ban đầu, nhiều cơ chế chưa tính đến quy mô tấn công và cường độ đối kháng ngày nay.

Các cuộc tấn công IP spoofing, SYN Flood, DDoS, ARP spoofing, DNS hijacking nhìn bề ngoài khác nhau, nhưng về bản chất đều lợi dụng giả định tin cậy, điểm tiêu tốn tài nguyên hoặc chuỗi phân giải trong network protocol.

Bài viết này chủ yếu trả lời một số câu hỏi:

1. Các phương thức tấn công TCP/IP thường gặp lần lượt lợi dụng cơ chế nào?
2. Các cuộc tấn công như IP spoofing, SYN Flood, DDoS đại khái xảy ra như thế nào?
3. Các cuộc tấn công mạng thường gặp gây ra những ảnh hưởng nào?
4. Trước các cuộc tấn công này, thông thường có những tư duy phòng vệ cơ bản nào?

## IP spoofing

### IP là gì?

Trong network, mọi thiết bị đều được cấp một địa chỉ. Địa chỉ này giống như địa chỉ nhà của Xiaolan, gồm **số nhà và số phòng**. **Số nhà** được cấp cho toàn subnet, còn **số phòng** tương ứng với số được cấp cho computer trong subnet; đó chính là địa chỉ trong network. Số tương ứng với **số nhà** là network number, còn số tương ứng với **số phòng** là host number. Toàn bộ địa chỉ này là **IP address**.

### Có thể biết gì qua IP address?

Qua IP address, chúng ta có thể phán đoán vị trí của server cần truy cập để gửi message đến server. Thông thường, message do sender gửi trước tiên đi qua hub của subnet, được chuyển tiếp đến router gần nhất, sau đó dựa trên vị trí định tuyến để truy cập router tiếp theo, cho đến khi đến đích.

**Định dạng IP header**:

![Định dạng các trường trong IP packet header](https://oss.javaguide.cn/p3-juejin/843fd07074874ee0b695eca659411b42~tplv-k3u1fbpfcp-zoom-1.png)

### Kỹ thuật IP spoofing là gì?

Lừa gạt, đánh tráo, giả mạo!

Kỹ thuật IP spoofing là kỹ thuật giả mạo IP address của một host nào đó. Bằng cách ngụy trang IP address, một host có thể giả làm host khác, mà host này thường có một đặc quyền nào đó hoặc được host khác tin cậy.

Giả sử user hợp pháp **(1.1.1.1)** đã thiết lập TCP connection với server, attacker có thể thử giả mạo packet RST có source IP là **1.1.1.1** để ngắt connection. Tuy nhiên, chỉ giả mạo source IP là chưa đủ: packet còn phải khớp với connection four-tuple và vượt qua kiểm tra sequence number của RST ở phía nhận. Attacker nằm trên đường truyền có thể quan sát sequence number của connection; attacker không quan sát được traffic thì phải đoán sequence number có thể chấp nhận, còn TCP implementation hiện đại có thể giảm thiểu blind RST attack bằng Challenge ACK.

Nếu RST giả mạo vượt qua kiểm tra, server sẽ đóng connection tương ứng. Dữ liệu mà user hợp pháp gửi sau đó cũng không thể tiếp tục dùng connection này mà phải thiết lập lại connection. Chính vì attacker còn cần lấy được hoặc đoán đúng các tham số connection, kiểu tấn công này không phải cứ giả mạo nhiều source IP rồi gửi RST tùy ý là chắc chắn thành công.

![Attacker giả mạo source IP gửi RST segment để ngắt connection hợp pháp](https://oss.javaguide.cn/p3-juejin/7547a145adf9404aa3a05f01f5ca2e32~tplv-k3u1fbpfcp-zoom-1.png)

### Giảm thiểu IP spoofing như thế nào?

Mặc dù không thể ngăn chặn IP spoofing, nhưng có thể áp dụng biện pháp để ngăn packet giả mạo xâm nhập network. **Ingress filtering** là một biện pháp phòng vệ rất thường gặp để chống spoofing, như BCP38 (tài liệu best practice chung) đề cập. Ingress filtering là một hình thức lọc packet, thường được triển khai trên thiết bị ở [network edge](https://www.cloudflare.com/learning/serverless/glossary/what-is-edge-computing/), dùng để kiểm tra IP packet đi vào và xác định source header của nó. Nếu source header của packet không khớp với nguồn của nó hoặc có vẻ đáng ngờ, packet sẽ bị từ chối. Một số network còn triển khai egress filtering, kiểm tra IP packet rời khỏi network để bảo đảm packet có source header hợp lệ, ngăn user bên trong network dùng IP spoofing để phát động cuộc tấn công độc hại ra ngoài.

## SYN Flood (flood)

### SYN Flood là gì?

SYN Flood là một trong những cuộc tấn công DDoS (Distributed Denial of Service, từ chối dịch vụ phân tán) nguyên thủy và kinh điển nhất trên Internet, nhằm làm cạn kiệt tài nguyên server khả dụng, khiến server không thể truyền legitimate traffic.

SYN Flood lợi dụng cơ chế three-way handshake của TCP protocol. Attacker thường dùng tool hoặc botnet host bị kiểm soát để gửi đến server lượng lớn TCP SYN packet có source IP thay đổi hoặc source port thay đổi. Sau khi server phản hồi các packet này, nó sẽ tạo ra lượng lớn half-open connection. Khi system resource bị tiêu hao hết, server không thể cung cấp service bình thường.

Tăng performance của server hoặc cung cấp thêm connection capacity không đáng kể trước lượng packet khổng lồ của SYN Flood. Mấu chốt để phòng vệ SYN Flood là phán đoán connection request nào đến từ source thật, chặn request từ source không thật để bảo đảm request nghiệp vụ bình thường được phục vụ.

![SYN Flood attack làm cạn kiệt tài nguyên server qua lượng lớn half-open connection](https://oss.javaguide.cn/p3-juejin/2b3d2d4dc8f24890b5957df1c7d6feb8~tplv-k3u1fbpfcp-zoom-1.png)

### Nguyên lý của TCP SYN Flood attack là gì?

**TCP SYN Flood** attack lợi dụng three-way handshake của **TCP** (**SYN -> SYN/ACK -> ACK**). Giả sử bên khởi tạo connection là A, bên nhận connection là B, tức B lắng nghe connection request do A gửi trên một port nào đó (**Port**). Quy trình như hình dưới, bên trái là A, bên phải là B.

![Quy trình bình thường của TCP three-way handshake để thiết lập connection](https://oss.javaguide.cn/p3-juejin/a39355a1ea404323a11ca6644e009183~tplv-k3u1fbpfcp-zoom-1.png)

A trước tiên gửi message **SYN** (Synchronization) cho B, yêu cầu B chuẩn bị sẵn sàng nhận data; sau khi nhận được, B phản hồi message **SYN-ACK** (Synchronization-Acknowledgement) cho A. Message này có hai mục đích:

- Xác nhận với A rằng đã chuẩn bị sẵn sàng nhận data,
- Đồng thời yêu cầu A cũng chuẩn bị sẵn sàng nhận data. Lúc này B đã xác nhận trạng thái nhận với A và chờ A xác nhận, connection ở **half-open state (Half-Open)**, đúng như tên gọi, mới chỉ mở một nửa; sau khi nhận được, A lại gửi message **ACK** (Acknowledgement) cho B, xác nhận với B rằng cũng đã chuẩn bị sẵn sàng nhận data. Đến đây three-way handshake hoàn tất và **connection** được thiết lập.

Bạn có nhận thấy điểm then chốt nhất nằm ở việc hai bên có cùng chuyển sang **trạng thái có thể nhận message** theo yêu cầu của đối phương hay không không? Việc xác nhận trạng thái này chủ yếu dựa vào **sequence number của message** (SequenceNum) mà hai bên sắp sử dụng. **TCP** cần dùng **sequence number của message** để đánh dấu thứ tự gửi message, bảo đảm message đến ứng dụng tầng trên của bên nhận theo đúng thứ tự gửi.

**TCP** là connection **full-duplex** (Duplex), đồng thời hỗ trợ communication hai chiều, tức hai bên có thể đồng thời gửi message cho nhau. Trong đó, message **SYN** và **SYN-ACK** mở channel communication một chiều A→B (B biết sequence number của A); message **SYN-ACK** và **ACK** mở channel communication một chiều B→A (A biết sequence number của B).

Phần trên bàn về communication trong điều kiện hai bên trung thực và tuân thủ quy tắc.

Nhưng trong thực tế, network có thể không ổn định và làm packet bị mất, khiến handshake message không đến được đối phương; cũng có thể đối phương cố ý không tuân thủ quy tắc, cố ý trì hoãn hoặc không gửi handshake confirmation message.

Giả sử B cung cấp service qua một **TCP** port. Khi nhận message **SYN** của A, B tích cực phản hồi message **SYN-ACK**, khiến connection đi vào **half-open state**. Vì B không chắc message **SYN-ACK** mình gửi hoặc message ACK do A phản hồi có bị mất giữa đường hay không, nên B đặt một **Timer** cho mỗi half-open connection đang chờ hoàn tất. Nếu quá thời gian mà vẫn chưa nhận được message **ACK** của A, B gửi lại message **SYN-ACK** cho A, cho đến khi số lần retry vượt quá một ngưỡng nhất định thì từ bỏ.

![Lượng lớn half-open connection trong SYN Flood chiếm dụng tài nguyên server](https://oss.javaguide.cn/p3-juejin/7ff1daddcec44d61994f254e664987b4~tplv-k3u1fbpfcp-zoom-1.png)

Để giúp A kết nối thành công, B cần **phân bổ kernel resource** để duy trì half-open connection. Khi B phải đối mặt với lượng lớn connection A như hình trên, **SYN Flood** attack hình thành. Attacker A có thể điều khiển zombie host gửi lượng lớn message SYN cho B nhưng không phản hồi message ACK, hoặc thẳng tay giả mạo **Source IP** trong message SYN khiến message **SYN-ACK** B phản hồi bị chìm vào hư không. Điều này khiến B bị chiếm giữ bởi lượng lớn half-open connection chắc chắn không thể hoàn tất, cho đến khi resource cạn kiệt và ngừng phản hồi các connection request bình thường.

### Các hình thức thường gặp của SYN Flood là gì?

Malicious user có thể phát động SYN Flood attack theo ba cách khác nhau:

1. **Direct attack:** SYN flood attack không giả mạo IP address được gọi là direct attack. Trong kiểu tấn công này, attacker hoàn toàn không che giấu IP address của mình. Vì attacker phát động attack từ một source device duy nhất có IP address thật nên rất dễ phát hiện và loại bỏ attacker. Để khiến target machine ở trạng thái half-open, hacker sẽ ngăn personal machine phản hồi SYN-ACK packet của server. Thông thường có hai cách thực hiện: triển khai firewall rule để chặn mọi outgoing packet ngoài SYN packet; hoặc lọc tất cả SYN-ACK packet đi vào để ngăn chúng đến machine của malicious user. Trên thực tế, cách này hiếm khi được dùng vì kiểu attack này khá dễ giảm thiểu: chỉ cần chặn IP address của từng malicious system. Ngay cả khi attacker sử dụng botnet như [Mirai botnet](https://www.cloudflare.com/learning/ddos/glossary/mirai-botnet/), họ thường cũng không cố ý che giấu IP của infected device.
2. **Spoofing attack:** Malicious user cũng có thể giả mạo IP address của từng SYN packet mình gửi để ngăn biện pháp giảm thiểu và tăng độ khó truy ra danh tính. Dù packet có thể đã được ngụy trang, vẫn có thể truy nguyên nguồn qua các packet này. Công việc phát hiện rất khó, nhưng không phải không thể thực hiện; đặc biệt, nếu Internet service provider (ISP) sẵn sàng hỗ trợ thì sẽ dễ hơn.
3. **Distributed attack (DDoS):** Nếu dùng botnet để phát động attack, khả năng truy ra nguồn attack rất thấp. Khi mức độ làm rối tăng lên, attacker còn có thể ra lệnh cho mỗi distributed device giả mạo IP address của packet mình gửi. Ngay cả khi attacker sử dụng botnet như Mirai botnet, họ thường cũng không cố ý che giấu IP của infected device.

### Giảm thiểu SYN Flood như thế nào?

#### Mở rộng backlog queue

Mỗi operating system được cài trên target device đều cho phép một số lượng half-open connection nhất định. Để phản hồi lượng lớn SYN packet, một cách là tăng số lượng half-open connection tối đa mà operating system cho phép. Để mở rộng backlog tối đa thành công, system phải dự trữ thêm memory resource để xử lý các request mới. Nếu system không đủ memory để đáp ứng quy mô backlog queue tăng lên, performance của system sẽ bị ảnh hưởng tiêu cực, nhưng vẫn tốt hơn từ chối service.

#### Thu hồi TCP half-open connection được tạo sớm nhất

Một chiến lược giảm thiểu khác là sau khi backlog được lấp đầy, ghi đè half-open connection được tạo sớm nhất. Chiến lược này yêu cầu thời gian thiết lập hoàn toàn một connection hợp pháp phải ngắn hơn thời gian malicious SYN packet lấp đầy backlog. Khi lượng attack tăng hoặc quy mô backlog nhỏ hơn nhu cầu thực tế, biện pháp phòng vệ cụ thể này sẽ không còn hiệu quả.

#### SYN Cookie

Chiến lược này yêu cầu server tạo Cookie. Để tránh ngắt connection khi backlog đang được lấp đầy, server dùng SYN-ACK packet phản hồi mỗi connection request, sau đó xóa SYN request khỏi backlog và xóa request khỏi memory, bảo đảm port luôn mở và sẵn sàng thiết lập lại connection. Nếu connection là request hợp pháp và ACK packet cuối cùng đã được gửi từ client machine về server, server sẽ khôi phục (có một số hạn chế) entry trong SYN backlog queue. Dù biện pháp giảm thiểu này chắc chắn làm mất một số thông tin TCP connection, nó vẫn tốt hơn việc vì vậy mà phát động denial-of-service attack vào user hợp pháp.

## UDP Flood (flood)

### UDP Flood là gì?

**UDP Flood** cũng là một denial-of-service attack, gửi lượng lớn user datagram protocol (**UDP**) packet đến target server nhằm áp đảo năng lực xử lý và phản hồi của device đó. Firewall bảo vệ target server cũng có thể cạn kiệt vì **UDP** flood, từ đó dẫn đến denial of service đối với legitimate traffic.

### Nguyên lý của UDP Flood attack là gì?

**UDP Flood** chủ yếu lợi dụng các bước server thực hiện khi phản hồi **UDP** packet được gửi đến một trong các port của server. Trong điều kiện bình thường, khi server nhận **UDP** packet trên một port cụ thể, nó sẽ trải qua hai bước:

- Trước tiên server kiểm tra có program nào đang chạy và lắng nghe request trên port được chỉ định hay không.
- Nếu không có program nào nhận packet trên port đó, IPv4 protocol stack thường trả về **ICMP Destination Unreachable**, trong đó Type là 3, Code là 3, tức **Port Unreachable**. Đây không phải ICMP Echo packet mà Ping sử dụng; trong network thực tế, loại ICMP error này cũng có thể bị firewall loại bỏ hoặc bị protocol stack giới hạn rate.

Lấy một ví dụ. Giả sử hôm nay cần liên hệ với Xiaolan ở một khách sạn. Sau khi bộ phận chăm sóc khách hàng của khách sạn nhận cuộc gọi, họ trước tiên kiểm tra danh sách phòng để bảo đảm Xiaolan có trong phòng, sau đó chuyển cuộc gọi cho Xiaolan.

Trước hết, receptionist nhận cuộc gọi của caller yêu cầu kết nối đến một phòng cụ thể. Sau đó receptionist cần kiểm tra danh sách tất cả phòng để bảo đảm khách có ở phòng và sẵn sàng nghe máy. Không may là nếu lúc này đột nhiên tất cả đường dây điện thoại cùng sáng lên, họ sẽ nhanh chóng bị quá tải.

Khi server nhận từng **UDP** packet mới, nó sẽ xử lý request qua các bước trên và sử dụng system resource trong quá trình đó. Khi gửi **UDP** packet, mỗi packet đều chứa **IP** address của source device. Trong loại **DDoS** attack này, attacker thường không sử dụng **IP** address thật của mình mà giả mạo source **IP** address của **UDP** packet, nhờ đó che giấu vị trí thật của attacker và có khả năng làm bão hòa response packet từ target server.

Vì target server sử dụng resource để kiểm tra và phản hồi kết quả của từng **UDP** packet nhận được, khi nhận lượng lớn **UDP** packet, resource của target có thể nhanh chóng cạn kiệt, dẫn đến denial of service đối với normal traffic.

![UDP Flood tiêu hao tài nguyên server qua lượng lớn UDP packet](https://oss.javaguide.cn/p3-juejin/23dbbc8243a84ed181e088e38bffb37a~tplv-k3u1fbpfcp-zoom-1.png)

### Giảm thiểu UDP Flood như thế nào?

Phần lớn operating system giới hạn một phần response rate của **ICMP** packet để ngắt **DDoS** attack cần ICMP response. Một nhược điểm của biện pháp giảm thiểu này là trong quá trình attack, legitimate packet cũng có thể bị lọc. Nếu volume của **UDP Flood** đủ lớn để làm bão hòa state table của firewall trên target server, mọi biện pháp giảm thiểu xảy ra ở server level đều không đủ để xử lý bottleneck ở upstream của target device.

## HTTP Flood (flood)

### HTTP Flood là gì?

HTTP Flood là một cuộc tấn công DDoS (Distributed Denial of Service, từ chối dịch vụ phân tán) quy mô lớn, nhằm làm target server quá tải bằng HTTP request. Khi target bị bão hòa bởi request và không thể phản hồi normal traffic, denial of service sẽ xảy ra, từ chối các request khác từ real user.

![HTTP Flood làm quá tải target server qua lượng lớn request ở application layer](https://oss.javaguide.cn/p3-juejin/aa64869551d94c8d89fa80eaf4395bfa~tplv-k3u1fbpfcp-zoom-1.png)

### Nguyên lý attack của HTTP Flood là gì?

HTTP flood attack là một loại DDoS attack ở “layer 7”. Layer 7 là application layer trong OSI model, chỉ các Internet protocol như HTTP. HTTP là nền tảng của Internet request dựa trên browser, thường dùng để load webpage hoặc gửi form content qua Internet. Việc giảm thiểu application-layer attack đặc biệt phức tạp vì rất khó phân biệt malicious traffic với normal traffic.

Để đạt hiệu quả tối đa, malicious actor thường tận dụng hoặc tạo botnet nhằm mở rộng tối đa ảnh hưởng của attack. Bằng cách tận dụng nhiều device bị nhiễm malware, attacker có thể phát động lượng lớn attack traffic.

HTTP flood attack có hai loại:

- **HTTP GET attack**: Trong hình thức attack này, nhiều computer hoặc device khác phối hợp với nhau gửi nhiều request đến target server cho image, file hoặc asset khác. Khi target bị nhấn chìm bởi request và response đi vào, các request khác từ normal traffic source sẽ bị denial of service.
- **HTTP POST attack**: Nói chung, khi submit form trên website, server phải xử lý incoming request và đẩy data vào persistence layer (thường là database). So với processing capacity và bandwidth cần để gửi POST request, quá trình xử lý form data và chạy database command cần thiết có mức độ tiêu tốn tương đối cao. Attack này lợi dụng sự khác biệt về resource consumption tương đối, trực tiếp gửi nhiều POST request đến target server cho đến khi capacity của target server bão hòa và từ chối service.

### Phòng vệ HTTP Flood như thế nào?

Như đã nói ở trên, giảm thiểu layer 7 attack rất phức tạp và thường cần thực hiện từ nhiều phương diện. Một cách là challenge device phát request để kiểm tra nó có phải bot hay không, tương tự CAPTCHA test thường dùng khi tạo account online. Bằng cách đưa ra yêu cầu như JavaScript computation challenge, có thể giảm thiểu nhiều attack.

Các cách khác để chặn HTTP flood attack gồm sử dụng Web application firewall (WAF), quản lý IP reputation database để theo dõi và chọn lọc chặn malicious traffic, cùng dynamic analysis do engineer thực hiện. Cloudflare có lợi thế về quy mô với hơn 20 triệu Internet device, có thể phân tích traffic từ nhiều source và giảm thiểu attack tiềm ẩn bằng WAF rule được cập nhật nhanh cùng các chiến lược phòng vệ khác, từ đó loại bỏ DDoS traffic ở application layer.

## DNS Flood (flood)

### DNS Flood là gì?

Domain Name System (DNS) server là “danh bạ điện thoại” của Internet; Internet device dùng server này để tìm Web server cụ thể nhằm truy cập Internet content. DNS Flood attack là một cuộc tấn công denial-of-service phân tán (DDoS), trong đó attacker nhấn chìm DNS server của một domain bằng lượng lớn traffic để cố làm gián đoạn DNS resolution của domain đó. Nếu user không tìm được danh bạ, họ không thể tìm thấy address dùng để gọi resource cụ thể. Bằng cách làm gián đoạn DNS resolution, DNS Flood attack phá hỏng khả năng website, API hoặc Web application phản hồi legitimate traffic. Rất khó phân biệt DNS Flood attack với normal high traffic, vì lượng traffic quy mô lớn này thường đến từ nhiều unique address, truy vấn record thật của domain và mô phỏng legitimate traffic.

### Nguyên lý attack của DNS Flood là gì?

![DNS Flood nhấn chìm DNS server bằng lượng lớn DNS query](https://oss.javaguide.cn/p3-juejin/97ea11a212924900b10d159226783887~tplv-k3u1fbpfcp-zoom-1.png)

Chức năng của Domain Name System là chuyển name dễ nhớ (ví dụ example.com) thành address của website server khó nhớ (ví dụ 192.168.0.1), vì vậy tấn công thành công vào DNS infrastructure sẽ khiến phần lớn mọi người không thể sử dụng Internet. DNS Flood attack là một kiểu DNS-based attack tương đối mới, tăng mạnh sau sự nổi lên của [Internet of Things (IoT)](https://www.cloudflare.com/learning/ddos/glossary/internet-of-things-iot/) [botnet](https://www.cloudflare.com/learning/ddos/what-is-a-ddos-botnet/) băng thông cao (như [Mirai](https://www.cloudflare.com/learning/ddos/glossary/mirai-botnet/)). DNS Flood attack sử dụng connection băng thông cao của IP camera, DVR box và device IoT khác để trực tiếp nhấn chìm DNS server của nhà cung cấp lớn. Lượng request khổng lồ từ IoT device nhấn chìm service của DNS provider, ngăn legitimate user truy cập DNS server của provider.

DNS Flood attack khác với [DNS amplification attack](https://www.cloudflare.com/zh-cn/learning/ddos/dns-amplification-ddos-attack/). Khác với DNS Flood attack, DNS amplification attack phản xạ và khuếch đại traffic của DNS server không an toàn để che giấu nguồn attack và tăng hiệu quả attack. DNS amplification attack dùng device có connection bandwidth nhỏ gửi vô số request đến DNS server không an toàn. Các device này gửi request nhỏ cho record DNS rất lớn, nhưng khi gửi request, attacker giả mạo return address thành target victim. Hiệu ứng khuếch đại này giúp attacker phá hoại target lớn hơn bằng resource attack hạn chế.

### Phòng vệ DNS Flood như thế nào?

DNS Flood đã thay đổi phương thức attack truyền thống dựa trên amplification. Nhờ botnet băng thông cao dễ kiếm, attacker hiện có thể phát động attack vào tổ chức lớn. Trừ khi các IoT device bị xâm nhập được update hoặc thay thế, cách duy nhất để chống lại các attack này là sử dụng DNS system siêu lớn, phân tán cao để theo dõi, hấp thụ và chặn attack traffic theo thời gian thực.

## TCP reset attack

Trong **TCP** reset attack, attacker gửi RST packet giả mạo đến một hoặc cả hai bên communication, cố khiến bên nhận đóng connection sớm. TCP có gửi hoặc chấp nhận RST hay không phụ thuộc vào trạng thái connection hiện tại, sequence number, acknowledgment number và các field khác của packet. Với connection đã được thiết lập, bên nhận chỉ đóng connection sau khi RST vượt qua kiểm tra sequence number; RST nằm ngoài window sẽ bị loại bỏ, còn endpoint triển khai RFC 5961 protection sẽ gửi Challenge ACK cho RST nằm trong window nhưng không khớp chính xác.

**TCP** reset attack lợi dụng cơ chế này, gửi reset segment giả mạo đến bên communication để đánh lừa hai bên đóng TCP connection sớm. Nếu reset segment giả mạo hoàn toàn giống thật, receiver sẽ cho rằng nó hợp lệ và đóng **TCP** connection, ngăn connection tiếp tục trao đổi information. Server có thể tạo **TCP** connection mới để khôi phục communication, nhưng vẫn có thể bị attacker reset connection. May mắn là attacker cần một khoảng thời gian để tạo và gửi packet giả mạo, nên trong điều kiện thông thường kiểu attack này chỉ gây ảnh hưởng lớn đến long connection; với short connection, khi bạn chưa kịp attack thì đối phương đã trao đổi information xong.

TCP thông thường không xác thực TCP header bằng cryptography, vì vậy TLS không thể bảo vệ RST ở TCP layer. Khi cần xác thực packet ở layer thấp hơn, có thể dùng các cơ chế như IPsec hoặc TCP-AO, nhưng chúng yêu cầu cả hai bên communication và network environment cung cấp hỗ trợ tương ứng.

## Mô phỏng attack

> Experiment dưới đây được thực hiện trên hệ thống `OSX`, các system khác hãy tự kiểm tra.

Bây giờ hãy tổng hợp cần làm những gì để giả mạo một **TCP** reset packet:

- Sniff information được hai bên communication trao đổi.
- Intercept một segment có ACK flag được set thành 1 và đọc ACK number của nó.
- Giả mạo một TCP reset segment (đặt `RST` flag thành 1), có sequence number bằng ACK number của packet đã intercept ở trên. Đây chỉ là phương án trong điều kiện lý tưởng, giả sử tốc độ trao đổi information không quá nhanh. Trong phần lớn trường hợp, để tăng tỷ lệ thành công, có thể liên tục gửi reset packet có sequence number khác nhau.
- Gửi reset packet giả mạo đến một hoặc cả hai bên communication để ngắt connection.

Để experiment đơn giản, có thể dùng local computer giao tiếp với chính nó qua `localhost`, sau đó thực hiện TCP reset attack với chính mình. Cần các bước sau:

- Thiết lập một TCP connection giữa hai terminal.
- Viết attack program có thể sniff data của hai bên communication.
- Sửa attack program để giả mạo và gửi reset packet.

Bây giờ chính thức bắt đầu experiment.

> Thiết lập TCP connection

Có thể dùng công cụ netcat để thiết lập TCP connection, công cụ này được cài sẵn trên nhiều operating system. Mở terminal window thứ nhất và chạy command sau:

```bash
nc -nvl 8000
```

Command này khởi động một TCP service, lắng nghe trên port `8000`. Tiếp theo mở terminal window thứ hai và chạy command sau:

```bash
nc 127.0.0.1 8000
```

Command này sẽ thử thiết lập connection với service ở trên. Khi nhập một số ký tự trong một window, chúng sẽ được gửi qua TCP connection đến window còn lại và được in ra.

![Dùng nc thiết lập local TCP connection và truyền data](https://oss.javaguide.cn/p3-juejin/df0508cbf26446708cf98f8ad514dbea~tplv-k3u1fbpfcp-zoom-1.gif)

> Sniff traffic

Viết một attack program, dùng Python network library `scapy` để đọc data trao đổi giữa hai terminal window và in chúng ra terminal. Code khá dài, dưới đây là một phần; để xem full code, hãy reply TCP attack ở backend. Core của code là gọi sniff method của `scapy`:

![Code dùng Scapy sniff packet của local TCP connection](https://oss.javaguide.cn/p3-juejin/27feb834aa9d4b629fd938611ac9972e~tplv-k3u1fbpfcp-zoom-1.png)

Đoạn code này yêu cầu `scapy` sniff packet trên network interface `lo0` và ghi lại thông tin chi tiết của mọi TCP connection.

- **iface**: yêu cầu scapy lắng nghe trên network interface `lo0` (`localhost`).
- **lfilter**: đây là một filter, yêu cầu scapy bỏ qua mọi packet không thuộc TCP connection được chỉ định (hai bên communication đều là `localhost`, port number là `8000`).
- **prn**: scapy dùng function này để xử lý mọi packet phù hợp với rule của `lfilter`. Ví dụ trên chỉ in packet ra terminal; phần sau sẽ sửa function để giả mạo reset packet.
- **count**: số lượng packet cần sniff trước khi scapy function return.

> Gửi reset packet giả mạo

Bắt đầu sửa program để gửi TCP reset packet giả mạo nhằm thực hiện TCP reset attack. Theo phân tích ở trên, chỉ cần sửa function prn, yêu cầu nó kiểm tra packet, trích xuất parameter cần thiết, dùng các parameter này để giả mạo TCP reset packet rồi gửi đi.

Ví dụ, giả sử program intercept một segment đi từ (`src_ip`, `src_port`) đến (`dst_ip`, `dst_port`), segment đó đã set ACK flag thành 1 và ACK number là `100,000`. Attack program tiếp theo cần làm như sau:

- Vì packet giả mạo là response của packet đã intercept, source `IP/Port` của packet giả mạo phải là destination `IP/Port` của packet đã intercept và ngược lại.
- Set `RST` flag của packet giả mạo thành 1 để biểu thị đây là reset packet.
- Đặt sequence number của packet giả mạo bằng ACK number của packet đã intercept, vì đây là sequence number tiếp theo mà sender mong đợi nhận được.
- Gọi `send` method của `scapy` để gửi packet giả mạo đến sender của packet đã intercept.

Đối với program của tôi, chỉ cần bỏ comment dòng này và comment dòng ngay phía trên là có thể thực hiện attack toàn diện. Thiết lập TCP connection theo cách ở bước 1, mở window thứ ba để chạy attack program, sau đó nhập một số string vào một terminal của TCP connection, bạn sẽ thấy TCP connection bị ngắt!

> Experiment thêm

1. Có thể tiếp tục dùng attack program để experiment, tăng giảm 1 sequence number của packet giả mạo để xem chuyện gì xảy ra, xem nó có thực sự cần hoàn toàn giống ACK number của packet đã intercept hay không.
2. Mở `Wireshark`, lắng nghe network interface lo0 và dùng filter `ip.src == 127.0.0.1 && ip.dst == 127.0.0.1 && tcp.port == 8000` để lọc data không liên quan. Bạn có thể xem mọi chi tiết của TCP connection.
3. Gửi data stream nhanh hơn trên connection để khiến attack khó thực hiện hơn.

## Man-in-the-middle attack

Zhu Bajie muốn tỏ tình với Xiaolan nên viết một lá thư cho Xiaolan. Kết quả là người thứ ba Xiaohei chặn được lá thư, sửa nội dung và phá hoại giữa hai người. Ma Wencai chính là man-in-the-middle, còn hành động được thực hiện là man-in-the-middle attack. Tiếp theo hãy tìm hiểu man-in-the-middle attack là gì.

### Man-in-the-middle là gì?

Tên tiếng Anh của man-in-the-middle attack là Man-in-the-Middle Attack, gọi tắt là **MITM attack**. Đây là việc attacker lần lượt tạo connection độc lập với hai đầu communication, rồi trao đổi data mà attacker nhận được, khiến hai đầu communication tưởng rằng họ đang trực tiếp nói chuyện với nhau qua một private connection, nhưng trên thực tế toàn bộ session đều bị attacker kiểm soát hoàn toàn. Hãy xem hình:

![Man-in-the-middle attack chặn và sửa message của hai bên communication](https://oss.javaguide.cn/p3-juejin/d69b74e63981472b852797f2fa08976f~tplv-k3u1fbpfcp-zoom-1.png)

Từ hình này có thể thấy man-in-the-middle thực chất là attacker. Dựa trên nguyên lý này, có nhiều cách triển khai, chẳng hạn khi bạn duyệt một website không lành mạnh trên phone, phone sẽ cảnh báo website này có thể chứa virus và hỏi bạn có tiếp tục truy cập hay thực hiện thao tác khác.

### Nguyên lý của man-in-the-middle attack là gì?

Lấy một ví dụ, tôi ký một labor contract với công ty, mỗi bên giữ một bản. Nếu không biết ai đã sửa nội dung contract thì làm sao biết thật giả? Chỉ còn cách tìm một organization chuyên nghiệp để giám định, đương nhiên phải tốn tiền.

Trong security có câu: **Chúng ta không thể triệt tiêu network crime, chỉ có thể tìm cách tăng cost của network crime**. Vì không thể triệt tiêu tình huống này, chúng ta tìm cách tăng cost để thực hiện hành vi đó. Hôm nay chỉ cần tìm hiểu đơn giản về kiến thức network security cơ bản, cũng là một câu hỏi phỏng vấn thường gặp.

Để tránh tình trạng hai bên nói không giữ lời, hai bên đưa vào một organization thứ ba và giao bản gốc contract cho trusted third-party organization. Chỉ cần organization này không tự trộm cắp hoặc sửa đổi, contract tương đối an toàn.

**Nếu bên trong third-party organization không nghiêm ngặt hoặc dễ xảy ra sơ suất thì sao?**

Mặc dù đã giao bản gốc contract cho third-party organization, cần áp dụng biện pháp gì để ngăn nhân viên bên trong sửa đổi?

Một cách khả thi là đưa vào **digest algorithm**. Hash function ánh xạ data có độ dài bất kỳ thành digest có độ dài cố định. Hash không phải encryption và không cung cấp khả năng decryption ngược; các input khác nhau cũng có thể tạo ra digest giống nhau, vì vậy không thể gọi digest là giá trị duy nhất tuyệt đối. Với cryptographic hash function an toàn, khi input thay đổi, digest thường cũng thay đổi theo.

#### Các digest algorithm thường dùng là gì?

Các encryption algorithm thường dùng hiện nay gồm message digest algorithm và secure hash algorithm (**SHA**). **MD5** chuyển article có độ dài bất kỳ thành hash value 128-bit, nhưng năm 2004, **MD5** được chứng minh dễ xảy ra collision, tức hai bản gốc tạo ra digest giống nhau. Như vậy chẳng khác nào trao cho hacker một backdoor để dễ dàng giả mạo digest.

Vì vậy, trong phần lớn trường hợp sẽ chọn **SHA algorithm**.

**Nếu xuất hiện internal attacker thì sao?**

Tình huống có vẻ đã rất an toàn, về lý thuyết đã ngăn việc sửa contract. Nhưng nếu một employee đồng thời có quyền sửa contract và digest thì gây rối chỉ còn là vấn đề thời gian, vì không system nào có thể hoàn toàn ngăn employee tiếp xúc với sensitive information, trừ khi sensitive information không tồn tại. Vậy có thể cân nhắc lưu contract và digest tách biệt không?

**Làm thế nào bảo đảm employee không sửa contract?**

Điều này thực sự khá khó, nhưng cách giải quyết luôn nhiều hơn khó khăn. Đặt contract trong tay hai bên, còn digest trong third-party organization, sẽ làm tăng thêm độ khó sửa đổi.

**Nếu employee thông đồng với một user nào đó thì sao?**

Có vẻ đặt trong third-party organization vẫn chưa đủ tốt và vẫn tồn tại rủi ro đáng kể. Vì vậy cần tìm phương án mới, từ đó xuất hiện **digital signature và certificate**.

#### Digital certificate và signature có tác dụng gì?

Lấy một ví dụ tương tự. Sum và Mike ký contract. Sum dùng signature algorithm và private key của mình để tạo digital signature cho contract, sau đó gửi contract, signature và public key dùng để verify cho Mike.

![Sơ đồ tạo digital signature và verify signature bằng public key](https://oss.javaguide.cn/p3-juejin/e4b7d6fca78b45c8840c12411b717f2f~tplv-k3u1fbpfcp-zoom-1.png)

Sau khi nhận được, Mike dùng public key của Sum để verify signature. Verify thành công cho biết signature khớp với public key đó và nội dung contract hiện tại, có thể phát hiện contract đã bị sửa đổi hay chưa, đồng thời xác nhận signature do bên nắm giữ private key của Sum tạo ra.

Nếu Mike sửa nội dung contract, signature ban đầu sẽ không thể vượt qua verify; không có private key của Sum thì cũng không thể tạo signature hợp lệ cho contract đã sửa. Private key phải được Sum bảo quản cẩn thận, còn public key có thể cung cấp cho verifier.

Digital signature nên được hiểu là “ký bằng private key, verify bằng public key”, chứ không phải “encrypt bằng private key, decrypt bằng public key” theo nghĩa phổ biến. Quy trình toán học của RSA, ECDSA, EdDSA và các signature algorithm khác nhau, nên mô tả bằng signature và verification sẽ chính xác hơn.

Privacy? Không phải dọa mọi người đâu, information là transparent, anh bạn, nhưng vẫn nên cố gắng bảo vệ privacy cá nhân. Hôm nay hãy học symmetric encryption và asymmetric encryption.

Trước tiên hãy đọc chữ “key” này, tôi từng đọc sai, thực ra cách đọc khác.

#### Symmetric encryption là gì?

Symmetric encryption, đúng như tên gọi, bên encryption và bên decryption dùng cùng một key (secret key). Cụ thể hơn, sender dùng encryption algorithm và key tương ứng để encrypt information sắp gửi; còn receiver dùng decryption algorithm và cùng key để mở khóa information, từ đó có thể đọc information.

![Hai bên communication dùng cùng key để encrypt và decrypt trong symmetric encryption](https://oss.javaguide.cn/p3-juejin/ef81cb5e2f0a4d3d9ac5a44ecf97e3cc~tplv-k3u1fbpfcp-zoom-1.png)

#### Các symmetric encryption algorithm thường gặp là gì?

**DES**

Key mà DES sử dụng nhìn bề ngoài có 64 bit, nhưng chỉ 56 bit trong số đó thực sự được dùng cho algorithm; 8 bit còn lại có thể dùng để parity check rồi bị loại bỏ trong algorithm. Vì vậy, effective key length của **DES** là 56 bit, thường gọi key length của **DES** là 56 bit. Giả sử key có 56 bit, dùng brute-force để phá, số lượng key là 2 mũ 56. Nếu thực hiện một lần decryption mỗi nanosecond thì cần thời gian xấp xỉ một năm. Tất nhiên không ai làm vậy. **DES** hiện không còn là encryption method an toàn, chủ yếu vì key 56 bit quá ngắn.

![Sơ đồ symmetric encryption algorithm DES](https://oss.javaguide.cn/p3-juejin/9eb3a2bf6cf14132a890bc3447480eeb~tplv-k3u1fbpfcp-zoom-1.jpeg)

**IDEA**

International Data Encryption Algorithm. Key length là 128 bit, ưu điểm là không bị giới hạn bởi patent.

**AES**

Sau khi DES bị phá, không lâu sau **AES** algorithm được đưa ra, cung cấp ba lựa chọn length: 128 bit, 192 bit và 256 bit. Để bảo đảm performance không bị ảnh hưởng quá nhiều, chọn 128 là đủ.

**SM1 và SM4**

Các algorithm trước đều đến từ nước ngoài, còn trong nước tự nghiên cứu **SM1** và **SM4** theo national cryptography. S trong hai tên đều thuộc national standard và algorithm được công khai. Ưu điểm là nhận được sự ủng hộ và công nhận mạnh mẽ của quốc gia.

**Tổng kết**:

![Tổng kết so sánh các symmetric encryption algorithm thường gặp](https://oss.javaguide.cn/p3-juejin/578961e3175540e081e1432c409b075a~tplv-k3u1fbpfcp-zoom-1.png)

#### Các asymmetric encryption algorithm thường gặp là gì?

Trong symmetric encryption, sender và receiver dùng cùng key. Còn trong asymmetric encryption, sender và receiver dùng các key khác nhau. Nó chủ yếu giải quyết vấn đề ngăn rò rỉ trong quá trình key negotiation. Ví dụ trong symmetric encryption, Xiaolan encrypt message cần gửi rồi nói với bạn password là 123balala,ok; người khác rất dễ intercept password 123balala. Trong asymmetric encryption, Xiaolan nói với mọi người password là 123balala, nhưng man-in-the-middle có lấy được cũng vô ích vì không có private key. Vì vậy, asymmetric key chủ yếu giải quyết bài toán key distribution. Như hình dưới.

![Asymmetric encryption dùng public key và private key để encrypt và decrypt](https://oss.javaguide.cn/p3-juejin/153cf04a0ecc43c38003f3a1ab198cc0~tplv-k3u1fbpfcp-zoom-1.png)

Thực ra chúng ta thường xuyên dùng asymmetric encryption, chẳng hạn dùng nhiều server để xây dựng big data platform Hadoop. Để nhiều machine đăng nhập không cần password, chắc chắn sẽ liên quan đến key distribution. Tương tự, khi xây dựng Docker cluster cũng dùng các asymmetric encryption algorithm liên quan.

Các asymmetric encryption algorithm thường gặp:

- RSA (RSA encryption algorithm, RSA Algorithm): security dựa trên độ khó tính toán của integer factorization, được ứng dụng rộng rãi và có compatibility tốt. Nhược điểm là performance tương đối chậm; key càng dài (như 2048/4096 bit) thì security càng cao, nhưng computation overhead cũng tăng theo.
- ECC: được đề xuất dựa trên elliptic curve, hiện là asymmetric encryption algorithm có encryption strength cao nhất.
- SM2: cũng được thiết kế dựa trên bài toán elliptic curve, ưu điểm lớn nhất là được quốc gia công nhận và ủng hộ mạnh mẽ.

Tổng kết:

![Tổng kết so sánh các asymmetric encryption algorithm thường gặp](https://oss.javaguide.cn/p3-juejin/28b96fb797904d4b818ee237cdc7614c~tplv-k3u1fbpfcp-zoom-1.png)

#### Các hash algorithm thường gặp là gì?

Hash algorithm thường dùng trong integrity check, content addressing và các scenario khác, nhưng yêu cầu security ở mỗi scenario không giống nhau. Password verification value không thể xử lý như digest của file thông thường: server nên tạo salt riêng cho từng password và dùng hash scheme có cost parameter, phù hợp với password storage; đồng thời lưu salt, algorithm identifier và cost parameter để sau này tăng computation cost hoặc migrate algorithm.

**MD5** (không khuyến nghị)

MD5 có thể tạo message digest 128 bit, nhưng đã không còn khả năng collision resistance đáng tin cậy, không nên dùng cho digital signature, certificate, security integrity check hoặc password storage. Nếu chỉ dùng để phát hiện transmission error ngẫu nhiên trong environment không có đối kháng, cũng phải nói rõ nó chỉ là checksum và không cung cấp security guarantee. MD5, SHA-1 và một SHA-256 thông thường đều không phù hợp để trực tiếp lưu password.

**SHA**

Secure hash algorithm. **SHA** gồm các series như **SHA-1**, **SHA-2** và **SHA-3**. Nó ánh xạ input data thành hash value (hoặc message digest) có length cố định. Process này không thể đảo ngược, nhưng hash value không phải ciphertext và cũng không tương đương message authentication code. SHA-1 không còn phù hợp với security scenario cần collision resistance; system mới thường chọn algorithm cụ thể trong series SHA-2 hoặc SHA-3.

**SM3**

National cryptography algorithm **SM3**. Encryption strength xấp xỉ algorithm SHA-256. Chủ yếu do nhận được sự ủng hộ của quốc gia.

**Tổng kết**:

![Tổng kết so sánh các hash algorithm thường gặp](https://oss.javaguide.cn/p3-juejin/79c3c2f72d2f44c7abf2d73a49024495~tplv-k3u1fbpfcp-zoom-1.png)

Symmetric encryption, asymmetric cryptography và hash algorithm giải quyết các vấn đề khác nhau: symmetric encryption dùng để bảo vệ lượng lớn data, asymmetric cryptography có thể dùng cho key negotiation, encryption hoặc digital signature, còn hash algorithm dùng để tạo digest. Cần chọn solution cụ thể dựa trên mục tiêu confidentiality, integrity, identity authentication và password storage, không thể chỉ phán đoán dựa trên “có thể đảo ngược hay không”.

#### Third-party organization và certificate mechanism có tác dụng gì?

Vẫn còn một vấn đề: nếu lúc này Sum phủ nhận đã đưa public key và contract cho Mike thì sẽ nhanh chóng gặp rắc rối.

Vì vậy, việc Sum đã làm cần có đủ credibility, từ đó đưa vào **third-party organization và certificate mechanism**.

Certificate có credibility vì issuer của certificate có credibility. Vì vậy, nếu Sum muốn Mike công nhận public key của mình, Sum sẽ không trực tiếp đưa public key cho Mike mà cung cấp certificate chứa public key do third-party organization cấp. Nếu Mike cũng tin organization này và pháp luật cũng công nhận, trust relationship sẽ được thiết lập.

![Quy trình third-party organization cấp certificate và hoàn tất signature verification](https://oss.javaguide.cn/p3-juejin/b1a3dbf87e3e41ff894f39512a10f66d~tplv-k3u1fbpfcp-zoom-1.png)

Như hình trên, Sum gửi certificate application đến certificate authority. Sau khi verify application information, organization dùng private key của mình để tạo digital signature cho phần chờ ký của certificate. Sau khi nhận certificate, Mike dùng public key của issuer để verify signature; verification thành công cho biết certificate content không bị sửa đổi, đồng thời signature do bên nắm giữ private key của organization tạo ra.

Solution này phụ thuộc vào third-party organization cung cấp trust endorsement cho binding relationship giữa identity và public key. Nếu issuer bị compromise hoặc cấp certificate sai, certificate verification phụ thuộc vào nó có thể bị ảnh hưởng.

PKI thực tế thường dùng hierarchical structure gồm root CA, intermediate CA và end-entity certificate, thuận tiện cho việc cô lập root private key, ủy quyền cấp certificate và giới hạn mục đích sử dụng certificate. Chain dài hơn tự nó không tự động loại bỏ trust risk.

![Trust chain từ root certificate đến end-entity certificate](https://oss.javaguide.cn/p3-juejin/1481f0409da94ba6bb0fee69bf0996f8~tplv-k3u1fbpfcp-zoom-1.png)

Trong hình trên, root certificate authority có uy tín cao nhất cung cấp root certificate, sau đó root certificate authority cấp certificate cho organization cấp hai; organization cấp hai cấp certificate cho organization cấp ba; cuối cùng organization cấp ba cấp certificate cho Sum.

Khi verify certificate của Sum, cần dùng public key trong certificate của organization cấp ba để verify digital signature của certificate Sum.

Khi verify certificate của organization cấp ba, cần dùng public key trong certificate của organization cấp hai để verify digital signature của nó.

Khi verify certificate của organization cấp hai, cần dùng public key tương ứng với trusted root để verify digital signature của nó. Client còn phải kiểm tra certificate validity period, name, purpose, path constraint và các điều kiện khác, cuối cùng xác nhận path này có anchor vào trusted root cục bộ hay không.

Những điều trên tạo thành một certificate trust chain. Nếu một trusted CA trong chain bị compromise hoặc cấp certificate sai, nó có thể cấp fraudulent certificate trong phạm vi được ủy quyền mà không cần tất cả organization đồng lõa.

### Tránh man-in-the-middle attack như thế nào?

Sau khi biết nguyên lý và mức độ nguy hiểm của man-in-the-middle attack, hãy xem cách tránh nó. Có lẽ mọi người đều từng gặp tình huống sau:

![Browser cảnh báo security rằng certificate không được trust](https://oss.javaguide.cn/p3-juejin/0dde4b76be6240699312d822a3fe1ed3~tplv-k3u1fbpfcp-zoom-1.png)

Certificate warning của browser cho biết certificate verification không thành công. Nguyên nhân có thể là certificate hết hạn, domain không khớp, certificate chain không được trust, thời gian trên machine local sai hoặc server configuration sai, cũng có thể là man-in-the-middle attack; chỉ dựa vào warning UI thì không thể xác định nguyên nhân cụ thể, user không nên bỏ qua warning để tiếp tục truy cập.

App chịu kiểm soát có thể cân nhắc certificate pinning hoặc public key pinning trong threat model rõ ràng, nhưng đồng thời phải thiết kế certificate rotation, backup key và failure recovery mechanism; nếu không, client có thể không connect được khi certificate được update. Với truy cập bằng browser thông thường, cách đúng là dựa vào system trust store để hoàn tất kiểm tra certificate chain, domain và validity period, thay vì tự trust certificate không rõ nguồn.

## DDoS

Qua mô tả ở trên, nhiều attack phía trước đều thuộc DDoS attack, vì vậy hãy tổng kết ngắn gọn nội dung liên quan đến attack này.

Thực tế, các công ty Internet lớn trên toàn cầu đều từng hứng chịu lượng lớn **DDoS**.

Năm 2018, GitHub trong chớp mắt hứng chịu bandwidth attack lên đến 1.35Tbps. Cuộc DDoS attack này gần như có thể xem là DDoS attack có quy mô lớn nhất và sức công phá mạnh nhất trong lịch sử Internet. Sau khi GitHub bị attack, chỉ một tuần sau, DDoS attack lại bắt đầu tấn công các website như Google, Amazon và thậm chí Pornhub. Bandwidth cao nhất của các DDoS attack về sau cũng đạt 1Tbps.

### DDoS attack rốt cuộc là gì?

DDos có tên đầy đủ là Distributed Denial of Service, dịch là **distributed denial of service**. Đây là việc nhiều attacker ở các vị trí khác nhau đồng thời tấn công một hoặc một số target, là một phương thức attack quy mô lớn mang tính distributed và coordinated. DoS attack đơn lẻ thường theo kiểu one-to-one, lợi dụng một số khiếm khuyết của network protocol và operating system, dùng chiến lược **spoofing và masquerading** để phát động network attack, khiến website server bị lấp đầy bởi lượng lớn information yêu cầu response, tiêu hao network bandwidth hoặc system resource, khiến network hoặc system quá tải, tê liệt và ngừng cung cấp normal network service.

> Lấy một ví dụ

Tôi mở một Chongqing hotpot restaurant có năm mươi chỗ ngồi, nguyên liệu chất lượng và không lừa dối khách hàng. Bình thường cửa hàng rất đông khách và business đặc biệt phát đạt, trong khi hotpot restaurant của Er Gou ở đối diện lại không có khách. Để đối phó với tôi, Er Gou nghĩ ra một cách: gọi năm mươi người đến hotpot restaurant của tôi, ngồi đó nhưng không gọi món, khiến khách khác không thể ăn.

Ví dụ trên chính là DDoS attack điển hình. Nói chung, đó là việc attacker dùng zombie host phát động lượng lớn request đến target website trong thời gian ngắn, tiêu hao quy mô lớn host resource của target website khiến website không thể cung cấp service bình thường. Online game, Internet finance và các lĩnh vực khác là những ngành có tần suất DDoS attack cao.

Có nhiều attack method, chẳng hạn **ICMP Flood**, **UDP Flood**, **NTP Flood**, **SYN Flood**, **CC attack**, **DNS Query Flood** và nhiều loại khác.

### Ứng phó với DDoS attack như thế nào?

#### High-defense server

Vẫn lấy Chongqing hotpot restaurant làm ví dụ: high-defense server giống như tôi bổ sung hai security guard cho restaurant. Hai security guard này có thể bảo vệ restaurant khỏi sự quấy rối của kẻ xấu, đồng thời thường xuyên tuần tra xung quanh restaurant để ngăn kẻ xấu quấy rối.

High-defense server chủ yếu là server có thể tự cung cấp hard defense trên 50Gbps, giúp website chống denial-of-service attack, định kỳ scan network main node và các chức năng khác.

#### Blacklist

Đối mặt với kẻ xấu trong hotpot restaurant, trong cơn giận tôi chụp ảnh họ để lưu hồ sơ và cấm họ bước vào restaurant, nhưng đôi lúc cũng cấm cả người trông giống họ. Đây là thiết lập blacklist. Phương pháp này theo nguyên tắc “thà chặn nhầm một nghìn còn hơn bỏ sót một trăm”, sẽ chặn normal traffic và ảnh hưởng đến normal business.

#### DDoS scrubbing

**DDos** scrubbing giống như việc tôi phát hiện khách vào restaurant vài phút nhưng mãi không gọi món, nên đuổi họ ra khỏi restaurant.

**DDoS** scrubbing monitor user request data theo thời gian thực, kịp thời phát hiện traffic bất thường như **DOS** attack và scrub traffic bất thường đó mà không ảnh hưởng đến hoạt động normal business.

#### CDN acceleration

CDN acceleration có thể hiểu như sau: để giảm sự quấy rối của kẻ xấu, tôi chuyển hotpot restaurant lên online để nhận delivery order. Như vậy kẻ xấu không tìm được restaurant ở đâu và cũng không thể gây rối.

Trong thực tế, CDN service phân bổ website access traffic đến nhiều node. Một mặt, nó che giấu real IP của website; mặt khác, ngay cả khi hứng chịu **DDoS** attack, traffic cũng có thể được phân tán đến nhiều node để ngăn origin server sụp đổ.

## Tài liệu tham khảo

- HTTP flood attack - CloudFlare: <https://www.cloudflare.com/zh-cn/learning/ddos/http-flood-ddos-attack/>
- SYN flood attack: <https://www.cloudflare.com/zh-cn/learning/ddos/syn-flood-ddos-attack/>
- IP spoofing là gì?: <https://www.cloudflare.com/zh-cn/learning/ddos/glossary/ip-spoofing/>
- DNS Flood là gì? | DNS Flood DDoS attack: <https://www.cloudflare.com/zh-cn/learning/ddos/dns-flood-ddos-attack/>

<!-- @include: @article-footer.snippet.md -->
