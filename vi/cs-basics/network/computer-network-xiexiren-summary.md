---
title: "Tổng hợp nội dung Computer Networks (Xie Xiren)"
description: "Ghi chú học tập dựa trên giáo trình Computer Networks, hệ thống hóa các điểm kiến thức cốt lõi như thuật ngữ và mô hình phân tầng, thuận tiện cho việc ôn tập cuối kỳ và củng cố kiến thức phỏng vấn."
category: Kiến thức cơ bản về máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: Mạng máy tính,Xie Xiren,thuật ngữ,mô hình phân tầng,link,host,tổng hợp giáo trình
---

Ghi chú này được tôi tổng hợp khi học mạng máy tính năm hai đại học, phần lớn nội dung tham khảo từ [Computer Networks, phiên bản thứ bảy](https://www.elias.ltd/usr/local/etc/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C%EF%BC%88%E7%AC%AC7%E7%89%88%EF%BC%89%E8%B0%A2%E5%B8%8C%E4%BB%81.pdf) của thầy Xie Xiren.

Nội dung giáo trình mạng máy tính khá rời rạc: cần xâu chuỗi thuật ngữ, phân tầng, link, routing, transport layer và application layer để xem. Để việc ôn tập thuận tiện hơn, tôi đã tái cấu trúc các ghi chú ban đầu và bổ sung một số sơ đồ minh họa.

Bài viết này chủ yếu trả lời một số câu hỏi:

1. Các thuật ngữ cơ bản thường gặp trong mạng máy tính có ý nghĩa gì?
2. Nên hiểu các mô hình phân tầng OSI và TCP/IP như thế nào?
3. Link layer, network layer, transport layer và application layer lần lượt giải quyết vấn đề gì?
4. Khi ôn tập cuốn Computer Networks, những khái niệm nào dễ bị nhầm lẫn nhất?

![Tổng quan các điểm kiến thức trong giáo trình Computer Networks](https://oss.javaguide.cn/p3-juejin/fb5d8645cd55484ab0177f25a13e97db~tplv-k3u1fbpfcp-zoom-1.png)

Câu hỏi liên quan: [Đánh giá Computer Networks (phiên bản thứ bảy) của Xie Xiren như thế nào? - Zhihu](https://www.zhihu.com/question/327872966).

## 1. Tổng quan về mạng máy tính

### 1.1. Thuật ngữ cơ bản

1. **Node**: Node trong mạng có thể là computer, hub, switch hoặc router.
2. **Link**: Một đoạn đường truyền vật lý từ node này đến node khác, ở giữa không có giao điểm nào khác.
3. **Host**: Computer được kết nối với Internet.
4. **ISP (Internet Service Provider)**: Nhà cung cấp dịch vụ Internet.

   ![Định nghĩa ISP (Internet Service Provider)](https://oss.javaguide.cn/p3-juejin/e77e26123d404d438d0c5943e3c65893~tplv-k3u1fbpfcp-zoom-1.png)

5. **IXP (Internet eXchange Point)**: Vai trò chính của IXP là cho phép hai mạng kết nối trực tiếp và trao đổi packet mà không cần chuyển tiếp packet qua mạng thứ ba.

   ![Mức lưu lượng IXP trong chuyến nhảy dù Stratos — RIPE Labs](https://oss.javaguide.cn/p3-juejin/7f9a6ddaa09441ceac11cb77f7a69d8f~tplv-k3u1fbpfcp-zoom-1.png)

   <p style="text-align:center;font-size:13px;color:gray">https://labs.ripe.net/Members/fergalc/ixp-traffic-during-stratos-skydive</p>

6. **RFC (Request For Comments)**: Có nghĩa là “yêu cầu nhận xét”, bao gồm gần như toàn bộ tài liệu quan trọng bằng văn bản về Internet.
7. **WAN (Wide Area Network)**: Nhiệm vụ là truyền dữ liệu do host gửi đi qua khoảng cách dài.
8. **MAN (Metropolitan Area Network)**: Dùng để kết nối nhiều LAN với nhau.
9. **LAN (Local Area Network)**: Trường học hoặc doanh nghiệp thường có nhiều LAN được kết nối với nhau.

   ![MAN & WMAN | Mạng khu vực đô thị, mạng máy tính, cáp xoắn đôi](https://oss.javaguide.cn/p3-juejin/eb48d21b2e984a63a26250010d7adac4~tplv-k3u1fbpfcp-zoom-1.png)

   <p style="text-align:center;font-size:13px;color:gray">http://conexionesmanwman.blogspot.com/</p>

10. **PAN (Personal Area Network)**: Mạng kết nối các thiết bị điện tử thuộc quyền sử dụng cá nhân tại nơi cá nhân làm việc bằng công nghệ wireless.

    ![Ưu và nhược điểm của mạng khu vực cá nhân (PAN) - IT Release](https://oss.javaguide.cn/p3-juejin/54bd7b420388494fbe917e3c9c13f1a7~tplv-k3u1fbpfcp-zoom-1.png)

    <p style=”text-align:center;font-size:13px;color:gray”>https://www.itrelease.com/2018/07/advantages-and-disadvantages-of-personal-area-network-pan/</p>

11. **Packet**: Đơn vị dữ liệu được truyền trong Internet. Gồm header và data segment. Packet còn được gọi là gói tin, header có thể gọi là phần đầu gói tin.
12. **Store and forward**: Router nhận một packet, trước tiên kiểm tra packet có đúng hay không và lọc các packet lỗi do xung đột. Sau khi xác định packet đúng, router lấy địa chỉ đích, tìm địa chỉ port đầu ra cần gửi thông qua bảng tra cứu, rồi gửi packet đó đi.

    ![Quá trình router lưu trữ và chuyển tiếp packet](https://oss.javaguide.cn/p3-juejin/addb6b2211444a4da9e0ffc129dd444f~tplv-k3u1fbpfcp-zoom-1.gif)

13. **Bandwidth**: Trong mạng máy tính, biểu thị “data rate cao nhất” có thể truyền từ một điểm này đến điểm khác trong mạng trong một đơn vị thời gian. Thường dùng để biểu thị khả năng truyền dữ liệu của đường truyền mạng. Đơn vị là “bit trên giây”, ký hiệu là b/s.
14. **Throughput**: Biểu thị lượng dữ liệu đi qua một mạng (hoặc channel, interface) trong một đơn vị thời gian. Throughput thường được dùng hơn để đo mạng trong thực tế, nhằm biết thực tế có bao nhiêu dữ liệu có thể đi qua mạng. Throughput bị giới hạn bởi bandwidth hoặc tốc độ định mức của mạng.

### 1.2. Tổng hợp các điểm kiến thức quan trọng

1. **Mạng máy tính (gọi tắt là mạng) kết nối nhiều computer với nhau, còn Internet kết nối nhiều mạng với nhau, tức là một mạng của các mạng.**
2. Internet viết thường chữ i là danh từ chung, chỉ mạng được tạo thành từ nhiều mạng computer kết nối với nhau. Protocol (tức quy tắc giao tiếp) giữa các mạng này có thể tùy ý. Internet viết hoa chữ I là danh từ riêng, chỉ Internet cụ thể lớn nhất và mở, được tạo thành từ nhiều mạng kết nối với nhau trên toàn cầu, sử dụng protocol TCP/IP làm quy tắc giao tiếp và có tiền thân là ARPANET. Cách dịch được khuyến nghị của Internet là Internet, hiện nay thường gọi là Internet.
3. Router là thành phần then chốt để thực hiện packet switching, nhiệm vụ là chuyển tiếp packet nhận được, đây là chức năng quan trọng nhất của phần lõi mạng. Packet switching sử dụng kỹ thuật store and forward, nghĩa là chia một message (toàn bộ dữ liệu cần gửi) thành một số packet rồi truyền đi. Trước khi gửi message, trước tiên chia message dài thành từng data segment nhỏ hơn, có cùng độ dài. Thêm vào phía trước mỗi data segment một header gồm các thông tin điều khiển cần thiết để tạo thành một packet. Packet còn được gọi là gói tin. Packet là đơn vị dữ liệu được truyền trong Internet. Chính vì phần đầu packet chứa các thông tin điều khiển quan trọng như địa chỉ đích và địa chỉ nguồn, mỗi packet mới có thể tự chọn đường truyền và được chuyển đến đúng điểm cuối của việc truyền packet trong Internet.
4. Internet có thể được chia thành phần rìa và phần lõi theo cách thức hoạt động. Host nằm ở phần rìa mạng, có tác dụng xử lý thông tin. Phần lõi gồm nhiều mạng và router kết nối các mạng đó, có tác dụng cung cấp connectivity và switching.
5. Giao tiếp computer là giao tiếp giữa các process trong computer (tức các chương trình đang chạy). Hình thức giao tiếp được mạng máy tính sử dụng là client-server (hình thức C/S) và peer-to-peer (hình thức P2P).
6. Client và server đều chỉ application process tham gia vào giao tiếp. Client là bên yêu cầu service, server là bên cung cấp service.
7. Theo phạm vi hoạt động khác nhau, mạng máy tính được chia thành WAN, MAN, LAN và PAN.
8. **Các chỉ số performance thường dùng nhất của mạng máy tính là: rate, bandwidth, throughput, latency (latency gửi, latency xử lý, latency xếp hàng), tích bandwidth-latency, round-trip time và channel utilization.**
9. Network protocol, gọi tắt là protocol, là các quy tắc được thiết lập để trao đổi dữ liệu trong mạng. Các layer của mạng máy tính và tập hợp protocol của chúng được gọi là network architecture.
10. **Kiến trúc năm layer gồm application layer, transport layer, network layer (internet layer), data link layer và physical layer. Các protocol quan trọng nhất của transport layer là TCP và UDP, protocol quan trọng nhất của network layer là IP.**

![Tổng quan kiến trúc năm layer của mạng máy tính](https://oss.javaguide.cn/p3-juejin/acec0fa44041449b8088872dcd7c0b3a~tplv-k3u1fbpfcp-zoom-1.gif)

Phần dưới đây sẽ giới thiệu kiến trúc năm layer của mạng máy tính: **physical layer + data link layer + network layer (internet layer) + transport layer + application layer**.

## 2. Physical Layer

![Physical layer](https://oss.javaguide.cn/p3-juejin/cf1bfdd36e5f4bde94aea44bbe7a6f8a~tplv-k3u1fbpfcp-zoom-1.png)

### 2.1. Thuật ngữ cơ bản

1. **Data**: Thực thể vận chuyển message.
2. **Signal**: Biểu hiện điện hoặc điện từ của data. Nói cách khác, signal là đối tượng thích hợp để truyền trên transmission medium.
3. **Code symbol**: Khi dùng waveform trong time domain (gọi tắt là domain thời gian) để biểu thị digital signal, đó là waveform cơ bản đại diện cho các giá trị rời rạc khác nhau.
4. **Simplex**: Chỉ có thể giao tiếp theo một hướng, không có tương tác theo hướng ngược lại.
5. **Half duplex**: Hai bên giao tiếp đều có thể gửi thông tin, nhưng không thể cùng gửi đồng thời (đương nhiên cũng không thể cùng nhận).
6. **Full duplex**: Hai bên giao tiếp có thể đồng thời gửi và nhận thông tin.

   ![So sánh các hình thức giao tiếp simplex, half duplex và full duplex](https://oss.javaguide.cn/p3-juejin/b1f02095b7c34eafb3c255ee81f58c2a~tplv-k3u1fbpfcp-zoom-1.png)

7. **Distortion**: Mất đi tính chân thực, chủ yếu chỉ signal nhận được khác signal gửi đi, có hao mòn và suy giảm. Các yếu tố ảnh hưởng đến mức độ distortion: 1. tốc độ truyền code symbol 2. khoảng cách truyền signal 3. nhiễu 4. chất lượng transmission medium.

   ![Sơ đồ minh họa distortion khi truyền signal](https://oss.javaguide.cn/p3-juejin/f939342f543046459ffabdc476f7bca4~tplv-k3u1fbpfcp-zoom-1.png)

8. **Nyquist criterion**: Trong mọi channel, hiệu suất truyền code symbol có giới hạn trên. Nếu tốc độ truyền vượt quá giới hạn này, hiện tượng intersymbol interference nghiêm trọng sẽ xuất hiện, khiến bên nhận không thể phán đoán (tức nhận diện) code symbol.
9. **Shannon theorem**: Trong channel bị giới hạn bandwidth và có noise, data rate có giá trị giới hạn trên để không phát sinh lỗi.
10. **Baseband signal**: Signal đến từ source. Là digital signal hoặc analog signal chưa qua modulation.
11. **Bandpass signal (frequency-band signal)**: Sau khi baseband signal được modulation bởi carrier, phạm vi tần số của signal được chuyển đến band cao hơn để truyền trong channel (tức chỉ có thể đi qua channel trong một phạm vi tần số). Signal sau khi modulation này là bandpass signal.
12. **Modulation**: Quá trình xử lý thông tin của signal source rồi đưa vào carrier signal, khiến nó trở thành dạng thích hợp để truyền trong channel.
13. **Signal-to-noise ratio**: Tỷ lệ giữa average power của signal và average power của noise, ký hiệu là S/N. Signal-to-noise ratio (dB) = 10\*log10 (S/N).
14. **Channel multiplexing**: Nhiều user cùng chia sẻ một channel (không nhất thiết đồng thời).

    ![Kỹ thuật channel multiplexing](https://oss.javaguide.cn/p3-juejin/5d9bf7b3db324ae7a88fcedcbace45d8~tplv-k3u1fbpfcp-zoom-1.png)

15. **Bit rate**: Số bit được truyền trong một đơn vị thời gian (mỗi giây).
16. **Baud rate**: Số lần trạng thái modulation của carrier thay đổi trong một đơn vị thời gian. Là tốc độ modulation của data signal đối với carrier.
17. **Multiplexing**: Phương pháp chia sẻ channel.
18. **ADSL (Asymmetric Digital Subscriber Line)**: Đường dây thuê bao digital bất đối xứng.
19. **Hybrid Fiber-Coaxial network (mạng HFC)**: Một loại broadband access network cho cư dân được phát triển trên nền tảng mạng truyền hình cáp hiện đang có phạm vi phủ sóng rộng.

### 2.2. Tổng hợp các điểm kiến thức quan trọng

1. **Nhiệm vụ chính của physical layer là xác định một số đặc tính liên quan đến interface của transmission medium, như đặc tính cơ học, đặc tính điện, đặc tính chức năng và đặc tính quy trình.**
2. Một data communication system có thể chia thành ba phần lớn: source system, transmission system và destination system. Source system gồm source (hoặc source station, data source) và sender; destination system gồm receiver và endpoint.
3. **Mục đích của communication là truyền message. Voice, text, image đều là message; data là thực thể vận chuyển message. Signal là biểu hiện điện hoặc điện từ của data.**
4. Theo cách lấy giá trị của tham số đại diện cho message trong signal, signal có thể chia thành analog signal (hoặc continuous signal) và digital signal (hoặc discrete signal). Khi dùng waveform trong time domain (gọi tắt là domain thời gian) để biểu thị digital signal, waveform cơ bản đại diện cho các giá trị rời rạc khác nhau được gọi là code symbol.
5. Theo phương thức tương tác thông tin giữa hai bên, communication có thể chia thành one-way communication (hoặc simplex communication), two-way alternating communication (hoặc half duplex communication) và two-way simultaneous communication (full duplex communication).
6. Signal đến từ source được gọi là baseband signal. Signal muốn truyền trong channel cần trải qua modulation. Modulation được chia thành baseband modulation và bandpass modulation. Các phương pháp bandpass modulation cơ bản nhất là amplitude modulation, frequency modulation và phase modulation. Ngoài ra còn có các phương pháp modulation phức tạp hơn như quadrature amplitude modulation.
7. Để tăng tốc độ truyền data trong channel, có thể sử dụng transmission medium tốt hơn hoặc kỹ thuật modulation tiên tiến. Tuy nhiên, data rate không thể tăng tùy ý.
8. Transmission medium có thể chia thành hai loại lớn: guided transmission medium (twisted pair, coaxial cable, fiber optic) và unguided transmission medium (wireless, infrared, atmospheric laser).
9. Để sử dụng hiệu quả tài nguyên fiber optic, passive optical network (PON) được sử dụng rộng rãi giữa fiber optic backbone và user. Passive optical network không cần trang bị power supply, nên operating cost và management cost dài hạn đều rất thấp. Passive optical network phổ biến nhất là Ethernet passive optical network EPON và Gigabit passive optical network GPON.

### 2.3. Bổ sung

#### 2.3.1. Physical layer làm gì?

Việc physical layer chủ yếu làm là **truyền bit stream một cách trong suốt**. Cũng có thể mô tả nhiệm vụ chính của physical layer là xác định một số đặc tính của interface với transmission medium, gồm: đặc tính cơ học (các thuộc tính vật lý như hình dạng và kích thước của connector được sử dụng trong interface), đặc tính điện (phạm vi điện áp xuất hiện trên từng đường của cáp interface), đặc tính chức năng (ý nghĩa của một mức điện áp nhất định xuất hiện trên một đường) và đặc tính quy trình (thứ tự xuất hiện của các event có thể xảy ra với các chức năng khác nhau).

**Physical layer quan tâm đến cách truyền bit stream data trên transmission medium kết nối các computer, chứ không chỉ transmission medium cụ thể.** Có rất nhiều loại hardware device và transmission medium trong các mạng máy tính hiện nay, đồng thời cũng có nhiều phương thức communication khác nhau. Vai trò của physical layer chính là cố gắng che giấu sự khác biệt giữa các transmission medium và phương thức communication này, để data link layer phía trên physical layer không cảm nhận được các khác biệt đó. Nhờ vậy, data link layer chỉ cần quan tâm hoàn thành protocol và service của layer này, không cần quan tâm transmission medium và phương thức communication cụ thể của mạng là gì.

#### 2.3.2. Một số kỹ thuật channel multiplexing thường dùng

1. **Frequency division multiplexing (FDM)**: Tất cả user chiếm dụng các tài nguyên bandwidth khác nhau trong cùng một thời điểm.
2. **Time division multiplexing (TDM)**: Tất cả user chiếm dụng cùng một bandwidth ở các thời điểm khác nhau (chia theo thời gian, không chia theo tần số).
3. **Statistical time division multiplexing (Statistic TDM)**: Time division multiplexing được cải tiến, có thể tăng rõ rệt channel utilization.
4. **Code division multiplexing (CDM)**: User sử dụng các code pattern khác nhau được lựa chọn đặc biệt, nên không gây nhiễu lẫn nhau. Signal do hệ thống này gửi có khả năng chống nhiễu mạnh, spectrum tương tự white noise và khó bị đối phương phát hiện.
5. **Wavelength division multiplexing (WDM)**: Wavelength division multiplexing chính là frequency division multiplexing của ánh sáng.

#### 2.3.3. Một số kỹ thuật broadband access thường dùng, chủ yếu là ADSL và FTTx

Các phương thức broadband access từ user đến Internet gồm asymmetric digital subscriber line ADSL (dùng kỹ thuật digital để cải tạo đường dây điện thoại analog hiện có mà không cần đi dây lại. Phiên bản nhanh của ADSL là very-high-speed digital subscriber line VDSL), hybrid fiber-coaxial network HFC (là broadband access network cho cư dân được phát triển trên nền tảng mạng truyền hình cáp hiện đang có phạm vi phủ sóng rộng) và FTTx (tức fiber optic đến······).

## 3. Data Link Layer

![Data link layer](https://oss.javaguide.cn/p3-juejin/83ec6dafc8c14ca185bafb656d86f0b2~tplv-k3u1fbpfcp-zoom-1.png)

### 3.1. Thuật ngữ cơ bản

1. **Link**: Một đoạn physical link từ một node đến node lân cận.
2. **Data link**: Thêm hardware và software thực hiện protocol điều khiển việc vận chuyển data vào link sẽ tạo thành data link.
3. **CRC (Cyclic Redundancy Check)**: Để đảm bảo độ tin cậy của việc truyền data, CRC là một kỹ thuật kiểm tra lỗi được sử dụng rộng rãi ở data link layer.
4. **Frame**: Đơn vị truyền của data link layer, là protocol data unit gồm header của data link layer và packet mà nó mang theo.
5. **MTU (Maximum Transfer Uint)**: Maximum Transfer Unit. Giới hạn trên về độ dài phần data của frame.
6. **BER (Bit Error Rate)**: Tỷ lệ giữa số bit truyền lỗi và tổng số bit được truyền trong một khoảng thời gian.
7. **PPP (Point-to-Point Protocol)**: Point-to-Point Protocol. Là protocol data link layer được sử dụng khi computer của user giao tiếp với ISP. Dưới đây là sơ đồ minh họa frame PPP:
   ![Định dạng frame của PPP Point-to-Point Protocol](https://oss.javaguide.cn/p3-juejin/6b0310d3103c4149a725a28aaf001899~tplv-k3u1fbpfcp-zoom-1.jpeg)
8. **MAC address (Media Access Control hoặc Medium Access Control)**: Dịch nghĩa là media access control, còn gọi là physical address hoặc hardware address, dùng để xác định vị trí của network device. Trong mô hình OSI, network layer ở layer thứ ba phụ trách IP address, còn data link layer ở layer thứ hai phụ trách MAC address. Vì vậy một host có một MAC address, còn mỗi vị trí mạng có một IP address riêng. Address là một identifier quan trọng để nhận diện một system: “name chỉ ra resource chúng ta muốn tìm, address chỉ ra nơi resource tồn tại, routing cho biết cách đi đến đó”.

   ![Giải thích ARP (Address Resolution Protocol)](https://oss.javaguide.cn/p3-juejin/057b83e7ec5b4c149e56255a3be89141~tplv-k3u1fbpfcp-zoom-1.png)

9. **Bridge**: Network interconnection device dùng để thực hiện relay ở data link layer và kết nối từ hai LAN trở lên.
10. **Switch**: Theo nghĩa rộng, switch là device thực hiện information switching trong một communication system. Ở đây, switch hoạt động tại data link layer là switching hub, về bản chất là một bridge nhiều interface.

### 3.2. Tổng hợp các điểm kiến thức quan trọng

1. Link là một đoạn physical link từ node này đến node lân cận; data link thì thêm một số hardware cần thiết (như network adapter) và software (như implementation của protocol) trên cơ sở link.
2. Data link layer chủ yếu sử dụng hai loại **point-to-point channel** và **broadcast channel**.
3. Protocol data unit được truyền ở data link layer là frame. Ba vấn đề cơ bản của data link layer là: **đóng gói thành frame**, **truyền trong suốt** và **phát hiện lỗi**.
4. **CRC** là một phương pháp phát hiện lỗi, còn frame check sequence FCS là mã dư được thêm vào sau data.
5. **Point-to-Point Protocol PPP** là một trong các protocol được sử dụng nhiều nhất ở data link layer, có các đặc điểm: đơn giản, chỉ phát hiện lỗi chứ không sửa lỗi, không sử dụng sequence number, không thực hiện flow control và có thể đồng thời hỗ trợ nhiều protocol network layer.
6. PPPoE là protocol link layer dành cho host truy cập Internet băng rộng.
7. **Ưu điểm của LAN là: có chức năng broadcast, dễ dàng truy cập toàn mạng từ một station; thuận tiện mở rộng và phát triển dần hệ thống; tăng độ tin cậy, tính khả dụng và khả năng tồn tại của hệ thống.**
8. Computer giao tiếp với LAN bên ngoài cần thông qua communication adapter (hoặc network adapter), còn gọi là network interface card hoặc network card. **Hardware address của computer nằm trong ROM của adapter.**
9. Ethernet sử dụng phương thức hoạt động connectionless, không đánh số frame data được gửi đi và không yêu cầu bên kia gửi confirmation. Destination station nhận frame có lỗi thì loại bỏ frame đó, không làm gì khác.
10. Protocol Ethernet sử dụng là **Carrier Sense Multiple Access with Collision Detection CSMA/CD** có phát hiện collision. Đặc điểm của protocol là: **lắng nghe trước khi gửi, vừa gửi vừa lắng nghe; khi phát hiện collision xuất hiện trên bus thì lập tức dừng gửi. Sau đó chờ một khoảng thời gian ngẫu nhiên theo backoff algorithm rồi gửi lại.** Vì vậy, mỗi station vẫn có khả năng gặp collision trong một khoảng thời gian ngắn sau khi tự gửi data. Các station trên Ethernet cạnh tranh bình đẳng để sử dụng Ethernet channel.
11. Adapter của Ethernet có chức năng filtering, chỉ nhận unicast frame, broadcast frame và multicast frame.
12. Có thể dùng hub để mở rộng Ethernet ở physical layer (Ethernet sau khi mở rộng vẫn là một mạng).

### 3.3. Bổ sung

1. Đặc điểm của point-to-point channel và broadcast channel ở data link layer, cùng đặc điểm của các protocol được dùng bởi hai channel này (PPP và CSMA/CD).
2. Ba vấn đề cơ bản của data link layer: **đóng gói thành frame**, **truyền trong suốt**, **phát hiện lỗi**.
3. Hardware address MAC layer của Ethernet.
4. Vai trò và trường hợp sử dụng của adapter, repeater, hub, bridge và Ethernet switch.

## 4. Network Layer

![Network layer](https://oss.javaguide.cn/p3-juejin/775dc8136bec486aad4f1182c68f24cd~tplv-k3u1fbpfcp-zoom-1.png)

### 4.1. Thuật ngữ cơ bản

1. **Virtual Circuit**: Kênh truyền trong suốt hai chiều được thiết lập giữa các port logic hoặc physical của hai terminal device. Virtual circuit cho biết đây chỉ là một connection logic; các packet truyền dọc theo connection logic này theo phương thức store and forward, chứ không thực sự thiết lập một connection physical.
2. **IP (Internet Protocol)**: IP là một trong hai protocol quan trọng nhất trong hệ thống TCP/IP, đồng thời là core của internet layer trong kiến trúc TCP/IP. Các protocol đi kèm gồm ARP, RARP, ICMP và IGMP.
3. **ARP (Address Resolution Protocol)**: Address Resolution Protocol. ARP phân giải IP address thành hardware address.
4. **ICMP (Internet Control Message Protocol)**: Internet Control Message Protocol (ICMP cho phép host hoặc router báo cáo tình trạng lỗi và cung cấp báo cáo về các tình huống bất thường).
5. **Subnet mask**: Dùng để chỉ ra bit nào của một IP address biểu thị subnet nơi host nằm trong đó và bit nào biểu thị host. Subnet mask không thể tồn tại độc lập, mà phải được sử dụng cùng IP address.
6. **CIDR (Classless Inter-Domain Routing)**: Classless Inter-Domain Routing (đặc điểm là loại bỏ các address class A, B, C truyền thống và khái niệm chia subnet, đồng thời dùng “network prefix” với độ dài khác nhau để thay thế network number và subnet number trong address phân loại).
7. **Default route**: Route được router chọn khi không tìm thấy route có thể đi đến destination address trong routing table. Default route còn có thể giảm không gian routing table chiếm dụng và thời gian tìm kiếm routing table.
8. **Routing Algorithm**: Thành phần core của routing protocol. Internet sử dụng routing protocol adaptive và phân cấp.

### 4.2. Tổng hợp các điểm kiến thức quan trọng

1. **Network layer trong protocol TCP/IP chỉ cung cấp cho layer phía trên một datagram service đơn giản, linh hoạt, connectionless và best-effort delivery. Network layer không cam kết quality of service, không đảm bảo thời hạn delivery của packet; packet được truyền có thể bị lỗi, mất, trùng lặp hoặc sai thứ tự. Độ tin cậy của giao tiếp giữa các process do transport layer phụ trách.**
2. Có hai hình thức delivery trong Internet: một là direct delivery trong cùng network, không cần đi qua router; hai là indirect delivery với network khác, phải đi qua ít nhất một router, nhưng lần cuối nhất định là direct delivery.
3. IP address phân loại gồm network number field (chỉ ra network) và host number field (chỉ ra host). Class ở đầu network number field chỉ ra loại IP address. IP address là một cấu trúc address phân cấp. Khi tổ chức quản lý IP address phân phối IP address, tổ chức chỉ phân phối network number; host number do đơn vị nhận network number tự phân phối. Router chuyển tiếp packet dựa trên network number mà destination host kết nối vào. Một router kết nối ít nhất hai network, nên một router ít nhất phải có hai IP address khác nhau.
4. IP datagram gồm header và data. Phần trước của header có độ dài cố định, tổng cộng 20 byte, là phần mọi IP packet bắt buộc phải có (các trường quan trọng như source address, destination address và total length đều cố định trong header). Một số optional field có độ dài thay đổi được đặt sau phần cố định của header. Time to live trong IP header cho biết số router tối đa mà IP datagram có thể đi qua trong Internet, giúp ngăn IP datagram quay vòng vô hạn trong Internet.
5. **ARP phân giải IP address thành hardware address. ARP cache có thể giảm đáng kể traffic trên mạng. Nhờ đó, lần sau khi host giao tiếp với host có cùng address, host có thể tìm trực tiếp hardware address cần thiết trong cache, không cần lại gửi ARP request packet theo phương thức broadcast.**
6. Classless Inter-Domain Routing CIDR là một cách tốt để giải quyết tình trạng thiếu IP address hiện nay. Cách ghi CIDR thêm dấu slash “/” sau IP address, rồi ghi số bit mà prefix chiếm. Prefix (hoặc network prefix) dùng để chỉ network, phần sau prefix là suffix, dùng để chỉ host. CIDR nhóm các IP address liên tiếp có cùng prefix thành một “CIDR address block”, việc phân phối IP address được thực hiện theo CIDR address block.
7. Internet Control Message Protocol là protocol của IP layer. ICMP message được đặt làm data của IP datagram, thêm header rồi tạo thành IP datagram để gửi đi. Sử dụng ICMP datagram không nhằm thực hiện reliable transmission. ICMP cho phép host hoặc router báo cáo tình trạng lỗi và cung cấp báo cáo về các tình huống bất thường. ICMP message có hai loại: ICMP error report message và ICMP query message.
8. **Để giải quyết vấn đề cạn kiệt IP address, cách căn bản nhất là sử dụng phiên bản IP protocol mới có address space lớn hơn là IPv6.** Các thay đổi do IPv6 mang lại gồm ① address space lớn hơn (sử dụng address 128 bit) ② header format linh hoạt ③ option được cải tiến ④ hỗ trợ plug-and-play ⑤ hỗ trợ preallocation resource ⑥ header của IPv6 được đổi thành căn chỉnh 8 byte.
9. **Virtual private network VPN sử dụng Internet công cộng làm phương tiện communication giữa các private network của một organization. VPN sử dụng private address của Internet. Một VPN ít nhất phải có một router sở hữu global IP address hợp lệ, nhờ đó mới có thể giao tiếp qua Internet với VPN khác của system này. Mọi data truyền qua Internet đều cần được encryption.**
10. Đặc điểm của MPLS: ① hỗ trợ quality of service hướng connection ② hỗ trợ traffic engineering và cân bằng network load ③ hỗ trợ hiệu quả virtual private network VPN. MPLS gắn một “label” có độ dài cố định vào mỗi IP datagram tại ingress node, sau đó chuyển tiếp bằng hardware ở layer thứ hai (link layer) theo label (thực hiện label switching trong label switching router), nhờ đó tốc độ forwarding được tăng lên đáng kể.

## 5. Transport Layer

![Transport layer](https://oss.javaguide.cn/p3-juejin/9fe85e137e7f4f03a580512200a59609~tplv-k3u1fbpfcp-zoom-1.png)

### 5.1. Thuật ngữ cơ bản

1. **Process**: Thực thể của chương trình đang chạy trong computer.
2. **Giao tiếp giữa các application process**: Quá trình process của một host trao đổi data với một process trong host khác (cần lưu ý thêm rằng endpoint thực sự của communication không phải host mà là process trong host; nói cách khác, end-to-end communication là communication giữa các application process).
3. **Multiplexing và demultiplexing của transport layer**: Multiplexing nghĩa là các process khác nhau ở sender đều có thể truyền data thông qua cùng một transport layer protocol. Demultiplexing nghĩa là transport layer của receiver, sau khi bỏ header của message, có thể chuyển chính xác data đến destination application process.
4. **TCP (Transmission Control Protocol)**: Transmission Control Protocol.
5. **UDP (User Datagram Protocol)**: User Datagram Protocol.

   ![TCP và UDP](https://oss.javaguide.cn/p3-juejin/b136e69e0b9b426782f77623dcf098bd~tplv-k3u1fbpfcp-zoom-1.png)

6. **Port**: Mục đích của port là xác định process nào trên machine đối phương đang tương tác với mình, chẳng hạn port của MSN và QQ khác nhau; nếu không có port, process QQ và MSN có thể tương tác nhầm. Port còn được gọi là protocol port number.
7. **Stop-and-wait protocol**: Sender dừng gửi sau khi gửi xong mỗi packet và chờ confirmation của đối phương; sau khi nhận confirmation mới gửi packet tiếp theo.
8. **Flow control**: Khiến tốc độ gửi của sender không quá nhanh, vừa để receiver kịp nhận, vừa không làm mạng phát sinh congestion.
9. **Congestion control**: Ngăn quá nhiều data được đưa vào network, nhờ đó router hoặc link trong network không bị quá tải. Tiền đề của congestion control là network có thể chịu được network load hiện tại.

### 5.2. Tổng hợp các điểm kiến thức quan trọng

1. **Transport layer cung cấp logical communication giữa các application process. Điều đó có nghĩa communication của transport layer không phải truyền data trực tiếp thực sự giữa hai transport layer. Transport layer che giấu các chi tiết của network bên dưới với application layer (như network topology, routing protocol được sử dụng...), khiến các application process có cảm giác như giữa hai transport layer entity có một logical communication channel end-to-end.**
2. **Network layer cung cấp logical communication cho host, còn transport layer cung cấp logical communication end-to-end cho application process.**
3. Hai protocol quan trọng của transport layer là User Datagram Protocol UDP và Transmission Control Protocol TCP. Theo thuật ngữ OSI, data unit mà hai peer transport entity truyền khi giao tiếp được gọi là TPDU (Transport Protocol Data Unit). Tuy nhiên, trong hệ thống TCP/IP, tùy protocol được sử dụng là TCP hay UDP mà lần lượt gọi là TCP segment hoặc UDP user datagram.
4. **UDP không cần thiết lập connection trước khi truyền data; remote host sau khi nhận UDP message cũng không cần gửi confirmation. Dù UDP không cung cấp reliable delivery, trong một số trường hợp UDP vẫn là phương thức hoạt động hiệu quả nhất. TCP cung cấp service hướng connection. Trước khi truyền data phải thiết lập connection, sau khi truyền data xong phải giải phóng connection. TCP không cung cấp broadcast hoặc multicast service. Vì TCP phải cung cấp reliable, connection-oriented transmission service nên khó tránh việc tăng nhiều overhead như confirmation, flow control, timer và connection management. Điều này không chỉ khiến header của protocol data unit lớn hơn nhiều mà còn chiếm dụng nhiều tài nguyên processor.**
5. Hardware port là interface để các hardware device khác nhau tương tác, còn software port là một address để các protocol process ở application layer tương tác với transport entity giữa các layer. Header format của UDP và TCP đều có hai field quan trọng là source port và destination port. Khi transport layer nhận message của transport layer do IP layer chuyển lên, nó có thể dựa vào destination port number trong header để chuyển data đến destination application layer. (Hai process giao tiếp với nhau không chỉ cần biết IP address của đối phương mà còn cần biết port của đối phương để tìm application process trong computer đối phương.)
6. Transport layer dùng port number 16 bit để đánh dấu một port. Port number chỉ có ý nghĩa cục bộ, chỉ dùng để đánh dấu interface giữa các layer khi từng process trong application layer của computer tương tác với transport layer. Trong các computer khác nhau trên Internet, cùng một port number không có liên hệ với nhau. Protocol port number được gọi tắt là port. Mặc dù endpoint của communication là application process, chỉ cần chuyển message cần gửi đến một port thích hợp của destination host, phần việc còn lại (chuyển cuối cùng đến destination process) sẽ do TCP và UDP hoàn thành.
7. Port number của transport layer được chia thành port number dùng ở server (0&tilde;1023 được gán cho well-known port, 1024&tilde;49151 là registered port number) và port number tạm thời dùng ở client (49152&tilde;65535).
8. **Các đặc điểm chính của UDP là ① connectionless ② best-effort delivery ③ hướng message ④ không có congestion control ⑤ hỗ trợ giao tiếp one-to-one, one-to-many, many-to-one và many-to-many ⑥ header overhead nhỏ (chỉ có bốn field: source port, destination port, length và checksum).**
9. **Các đặc điểm chính của TCP là ① connection-oriented ② mỗi TCP connection chỉ có thể là one-to-one ③ cung cấp reliable delivery ④ cung cấp full duplex communication ⑤ hướng byte stream.**
10. **TCP dùng IP address của host cộng với port number trên host làm endpoint của TCP connection. Endpoint này được gọi là socket. Socket được biểu thị bằng (IP address: port number). Mỗi TCP connection được xác định duy nhất bởi hai endpoint ở hai đầu giao tiếp.**
11. Stop-and-wait protocol được dùng để thực hiện reliable transmission. Nguyên lý cơ bản là dừng gửi sau mỗi packet, chờ confirmation của đối phương. Sau khi nhận confirmation mới gửi packet tiếp theo.
12. Để tăng transmission efficiency, sender có thể không dùng stop-and-wait protocol kém hiệu quả mà dùng pipeline transmission. Pipeline transmission nghĩa là sender có thể liên tục gửi nhiều packet, không cần dừng lại chờ confirmation sau mỗi packet. Nhờ đó data liên tục được truyền trên channel mà không bị gián đoạn. Phương thức transmission này có thể tăng channel utilization rõ rệt.
13. Timeout retransmission trong stop-and-wait protocol nghĩa là chỉ cần quá một khoảng thời gian mà vẫn chưa nhận được confirmation thì retransmit packet đã gửi trước đó (cho rằng packet vừa gửi đã bị mất). Vì vậy, sau khi gửi mỗi packet cần thiết lập một timeout timer; retransmission time nên dài hơn một chút so với average round-trip time của packet data. Phương thức automatic retransmission này thường được gọi là automatic repeat request ARQ. Ngoài ra, nếu nhận packet trùng lặp trong stop-and-wait protocol thì loại bỏ packet đó, nhưng đồng thời vẫn phải gửi confirmation. Continuous ARQ protocol có thể tăng channel utilization. Sender duy trì một sending window, các packet nằm trong sending window có thể được gửi liên tục mà không cần chờ confirmation của đối phương. Receiver thường sử dụng cumulative acknowledgment, gửi confirmation cho packet cuối cùng đến theo đúng thứ tự, cho biết tất cả packet từ đầu đến packet này đã được nhận chính xác.
14. 20 byte đầu của TCP segment là cố định, sau đó có optional field dài 40 byte. Nếu sau khi thêm optional field, header length không phải bội số nguyên của 4 byte thì tiếp tục điền 0 ở phía sau. Vì vậy, độ dài TCP header có giá trị 20+4n byte, tối đa 60 byte.
15. **TCP sử dụng cơ chế sliding window. Sequence number trong sending window biểu thị sequence number được phép gửi. Phần phía sau của rear edge của sending window biểu thị phần đã gửi và nhận được confirmation, còn phần phía trước front edge của sending window biểu thị phần chưa được phép gửi. Rear edge của sending window có hai khả năng thay đổi: đứng yên (chưa nhận confirmation mới) và tiến về trước (đã nhận confirmation mới). Front edge của sending window thường liên tục tiến về trước. Nhìn chung, chúng ta luôn mong data transmission nhanh hơn. Nhưng nếu sender gửi data quá nhanh, receiver có thể không kịp nhận, dẫn đến mất data. Flow control chính là khiến tốc độ gửi của sender không quá nhanh để receiver kịp nhận.**
16. Trong một khoảng thời gian, nếu nhu cầu đối với một resource nào đó trong network vượt quá phần khả dụng mà resource đó có thể cung cấp, performance của network sẽ xấu đi. Tình trạng này gọi là congestion. Congestion control nhằm ngăn quá nhiều data được đưa vào network, để router hoặc link trong network không bị quá tải. Tiền đề của congestion control là network có thể chịu được network load hiện tại. Congestion control là một process mang tính toàn cục, liên quan đến tất cả host, tất cả router và mọi yếu tố liên quan đến việc làm giảm performance truyền dữ liệu của network. Ngược lại, flow control thường là control traffic point-to-point, là vấn đề end-to-end. Mục tiêu của flow control là kìm hãm tốc độ sender gửi data để receiver kịp nhận.
17. **Để thực hiện congestion control, TCP sender phải duy trì state variable congestion window cwnd. Kích thước congestion control window phụ thuộc mức độ congestion của network và thay đổi động. Sender đặt sending window của mình bằng giá trị nhỏ hơn giữa congestion window và receiving window của receiver.**
18. **TCP sử dụng bốn algorithm cho congestion control: slow start, congestion avoidance, fast retransmit và fast recovery. Ở network layer, router cũng có thể sử dụng packet discard policy thích hợp (như Active Queue Management AQM) để giảm congestion trong network.**
19. Ba giai đoạn của transport connection là: connection establishment, data transmission và connection release.
20. **Application process chủ động khởi tạo TCP connection được gọi là client, còn application process thụ động chờ connection establishment được gọi là server. TCP connection sử dụng cơ chế three-way handshake. Server phải xác nhận connection request của user, sau đó client phải xác nhận confirmation của server.**
21. TCP connection release sử dụng cơ chế four-way handshake. Bất kỳ bên nào cũng có thể gửi thông báo connection release sau khi truyền data xong, rồi chuyển sang half-closed state sau khi đối phương confirmation. Khi bên kia cũng không còn data để gửi, bên đó gửi thông báo connection release; sau khi đối phương confirmation thì TCP connection được đóng hoàn toàn.

### 5.3. Bổ sung (quan trọng)

Các điểm kiến thức sau cần được chú ý:

1. Ý nghĩa của port và socket.
2. Điểm khác nhau giữa UDP và TCP, cùng application scenario của hai protocol.
3. Nguyên lý thực hiện reliable transmission trên network không đáng tin cậy, stop-and-wait protocol và ARQ protocol.
4. Sliding window, flow control, congestion control và connection management của TCP.
5. Three-way handshake và four-way handshake của TCP.

## 6. Application Layer

![Application layer](https://oss.javaguide.cn/p3-juejin/0f13f0ee13b24af7bdddf56162eb6602~tplv-k3u1fbpfcp-zoom-1.png)

### 6.1. Thuật ngữ cơ bản

1. **Domain Name System (DNS)**: Domain Name System (DNS, Domain Name System) chuyển domain name mà con người có thể đọc (ví dụ www.baidu.com) thành IP address mà machine có thể đọc (ví dụ 220.181.38.148). Có thể hiểu nó như telephone directory được thiết kế riêng cho Internet.

   ![Quá trình DNS phân giải domain name thành IP address](https://oss.javaguide.cn/p3-juejin/e7da4b07947f4c0094d46dc96a067df0~tplv-k3u1fbpfcp-zoom-1.png)

   <p style="text-align:right;font-size:12px">https://www.seobility.net/en/wiki/HTTP_headers</p>

2. **File Transfer Protocol (FTP)**: FTP là viết tắt tiếng Anh của File Transfer Protocol, dùng để truyền file hai chiều trên Internet. Nó cũng là một application. Có các FTP application khác nhau tùy hệ điều hành, nhưng tất cả application này đều tuân theo cùng một protocol để truyền file. Khi sử dụng FTP, user thường gặp hai khái niệm: “download” và “upload”. “Download” file nghĩa là copy file từ remote host về computer của mình; “upload” file nghĩa là copy file từ computer của mình lên remote host. Theo ngôn ngữ Internet, user có thể thông qua client program để upload (download) file lên (từ) remote host.

   ![Quy trình hoạt động của FTP](https://oss.javaguide.cn/p3-juejin/f3f2caaa361045a38fb89bb9fee15bd3~tplv-k3u1fbpfcp-zoom-1.png)

3. **Trivial File Transfer Protocol (TFTP)**: TFTP (Trivial File Transfer Protocol, Trivial File Transfer Protocol) là protocol trong họ protocol TCP/IP, dùng để truyền file đơn giản giữa client và server, cung cấp file transfer service không phức tạp và ít overhead. Port number là 69.
4. **Remote Terminal Protocol (TELNET)**: Telnet protocol là một thành viên của họ protocol TCP/IP, là protocol tiêu chuẩn và phương thức chủ yếu của remote login service trên Internet. Nó cho phép user hoàn thành công việc trên remote host từ local computer. User sử dụng telnet program trên computer của terminal user để kết nối đến server. Terminal user có thể nhập command trong telnet program, các command này sẽ chạy trên server giống như được nhập trực tiếp trong console của server. Có thể control server ngay tại local. Để bắt đầu một telnet session, phải nhập username và password để login server. Telnet là phương thức thường dùng để remote control Web server.
5. **World Wide Web (WWW)**: WWW là viết tắt của World Wide Web (còn gọi là “Web”, “WWW”, “'W3'”, tên đầy đủ tiếng Anh là “World Wide Web”), thường gọi tắt là Web. Nó gồm Web client và Web server program. WWW cho phép Web client (thường là browser) truy cập và duyệt page trên Web server. Đây là một system gồm nhiều hypertext liên kết với nhau, được truy cập qua Internet. Trong system này, mỗi thứ hữu ích được gọi là một “resource” và được xác định bằng một “Uniform Resource Identifier” (URI) toàn cục. Các resource này được truyền đến user thông qua Hypertext Transfer Protocol, sau đó user nhận resource bằng cách click link. World Wide Web Consortium (tiếng Anh: World Wide Web Consortium, viết tắt W3C), còn gọi là W3C Council, được thành lập vào tháng 10 năm 1994 tại MIT Computer Science Laboratory. Người sáng lập World Wide Web Consortium là Tim Berners-Lee, nhà phát minh World Wide Web. World Wide Web không đồng nghĩa với Internet; World Wide Web chỉ là một trong các service mà Internet cung cấp, một service vận hành dựa trên Internet.
6. **Quy trình hoạt động khái quát của World Wide Web:**

   ![Quy trình hoạt động khái quát của World Wide Web](https://oss.javaguide.cn/p3-juejin/ba628fd37fdc4ba59c1a74eae32e03b1~tplv-k3u1fbpfcp-zoom-1.jpeg)

7. **Uniform Resource Locator (URL)**: Uniform Resource Locator là cách biểu thị ngắn gọn vị trí và phương thức truy cập của resource có thể lấy được từ Internet, là address của resource tiêu chuẩn trên Internet. Mỗi file trên Internet có một URL duy nhất, chứa thông tin chỉ ra vị trí file và cách browser nên xử lý file.
8. **Hypertext Transfer Protocol (HTTP)**: Hypertext Transfer Protocol (HTTP, HyperText Transfer Protocol) là một network protocol được sử dụng rộng rãi nhất trên Internet. Mọi file WWW đều phải tuân thủ tiêu chuẩn này. Mục đích ban đầu khi thiết kế HTTP là cung cấp phương thức publish và receive HTML page. Năm 1960, người Mỹ Ted Nelson đề xuất phương thức xử lý text information bằng computer và gọi là hypertext; đây trở thành nền tảng cho sự phát triển của kiến trúc tiêu chuẩn HTTP Hypertext Transfer Protocol.

   Bản chất của HTTP protocol là một communication format được browser và server thỏa thuận. Nguyên lý của HTTP như hình dưới đây:

   ![Quy trình request và response giữa HTTP client và server](https://oss.javaguide.cn/p3-juejin/8e3efca026654874bde8be88c96e1783~tplv-k3u1fbpfcp-zoom-1.jpeg)

9. **Proxy Server**: Proxy Server là một network entity, còn được gọi là Web cache. Proxy server tạm lưu một số request và response gần đây trên local disk. Khi request mới đến, nếu proxy server phát hiện request này giống request đang tạm lưu, nó trả về response đã lưu mà không cần lại truy cập resource đó trên Internet theo URL. Proxy server có thể hoạt động ở client hoặc server, cũng có thể hoạt động ở intermediate system.
10. **Simple Mail Transfer Protocol (SMTP)**: SMTP (Simple Mail Transfer Protocol) là một tập hợp các quy tắc dùng để truyền mail từ source address đến destination address, kiểm soát phương thức relay của mail. SMTP protocol thuộc họ protocol TCP/IP, giúp mỗi computer tìm destination tiếp theo khi gửi hoặc relay mail. Thông qua server được SMTP protocol chỉ định, có thể gửi E-mail đến mail server của người nhận; toàn bộ quá trình chỉ mất vài phút. SMTP server là mail sending server tuân theo SMTP protocol, dùng để gửi hoặc relay email.

    ![Quy trình gửi một email](https://oss.javaguide.cn/p3-juejin/2bdccb760474435aae52559f2ef9652f~tplv-k3u1fbpfcp-zoom-1.png)

    <p style="text-align:right;font-size:12px">https://www.campaignmonitor.com/resources/knowledge-base/what-is-the-code-that-makes-bcc-or-cc-operate-in-an-email/</p>

11. **Search engine**: Search engine là system thu thập information từ Internet theo strategy nhất định và bằng computer program cụ thể, tổ chức và xử lý information rồi cung cấp retrieval service cho user, hiển thị information liên quan đến nội dung user tìm kiếm. Search engine gồm full-text index, directory index, meta search engine, vertical search engine, federated search engine, portal search engine và free link list.

12. **Vertical search engine**: Vertical search engine là search engine chuyên nghiệp dành cho một industry, là sự phân loại và mở rộng của search engine. Nó tích hợp một loại information chuyên biệt trong web page database, trích xuất có định hướng các data cần thiết theo từng field rồi xử lý và trả về cho user dưới một hình thức nhất định. Vertical search là một mode service search engine mới được đề xuất để giải quyết các vấn đề của general search engine như lượng information lớn, query không chính xác và độ sâu chưa đủ. Nó cung cấp information và service có giá trị cho một domain cụ thể, một nhóm user cụ thể hoặc một nhu cầu cụ thể. Đặc điểm là “chuyên, tinh, sâu”; so với lượng information khổng lồ và thiếu trật tự của general search engine, vertical search engine tập trung, cụ thể và chuyên sâu hơn.
13. **Full-text index**: Full-text index technology hiện là công nghệ cốt lõi của search engine. Hãy thử tưởng tượng tìm một từ trong file dung lượng 1M có thể mất vài giây, trong file 100M có thể mất vài chục giây; nếu tìm trong file lớn hơn thì cần system overhead lớn hơn, mức overhead đó không thực tế. Vì mâu thuẫn này mà full-text index technology xuất hiện; đôi khi nó còn được gọi là inverted index technology.
14. **Directory index**: Directory index (search index/directory), đúng như tên gọi, lưu website vào các directory tương ứng theo từng category. Vì vậy, khi query information, user có thể chọn tìm bằng keyword hoặc tìm từng tầng theo directory phân loại.

### 6.2. Tổng hợp các điểm kiến thức quan trọng

1. File Transfer Protocol (FTP) sử dụng reliable transport service của TCP. FTP sử dụng client-server mode. Một FTP server process có thể đồng thời cung cấp service cho nhiều user. Khi truyền file, FTP client và server trước tiên phải thiết lập hai TCP connection song song: control connection và data connection. Data connection mới là connection thực sự dùng để truyền file.
2. Protocol được Web client và server sử dụng khi tương tác là Hypertext Transfer Protocol HTTP. HTTP sử dụng TCP connection để truyền đáng tin cậy. Tuy nhiên, bản thân HTTP là connectionless và stateless. HTTP/1.1 sử dụng persistent connection (chia thành non-pipelined mode và pipelined mode).
3. Email gửi mail đến mail server mà người nhận sử dụng và đặt vào mailbox của người nhận trong đó; người nhận có thể lên Internet bất cứ lúc nào để đọc mail tại mail server mình sử dụng, tương đương email box.
4. Một email system có ba thành phần cấu tạo quan trọng: user agent, mail server và mail protocol (gồm mail sending protocol như SMTP và mail reading protocol như POP3 và IMAP). User agent và mail server đều phải chạy các protocol này.

### 6.3. Bổ sung (quan trọng)

Các điểm kiến thức sau cần được chú ý:

1. Các protocol thường gặp của application layer (chú ý trọng tâm HTTP protocol).
2. Domain Name System - phân giải domain name thành IP address.
3. Quy trình khái quát khi truy cập một website.
4. Khái niệm system call và application programming interface.

<!-- @include: @article-footer.snippet.md -->
