---
title: Giải thích chi tiết giao thức NAT (network layer)
description: Phân tích cơ chế chuyển đổi địa chỉ và ánh xạ port của NAT, kết hợp với giao tiếp LAN/WAN và bảng chuyển đổi để hiểu chi tiết thực tế của mạng gia đình và doanh nghiệp.
category: Computer Basics
tag:
  - Computer Networks
head:
  - - meta
    - name: keywords
      content: NAT,chuyển đổi địa chỉ,ánh xạ port,LAN,WAN,connection tracking,DHCP
---

Nhiều thiết bị trong mạng gia đình và mạng nội bộ công ty sử dụng private IP, chẳng hạn như `192.168.x.x`, `10.x.x.x`. Các địa chỉ này không thể được route trực tiếp trên public network, nhưng thiết bị trong mạng nội bộ vẫn có thể truy cập Internet.

Đằng sau việc này thường có NAT hoạt động. NAT chuyển đổi giữa địa chỉ mạng nội bộ và địa chỉ public, giúp nhiều thiết bị trong mạng nội bộ dùng chung một hoặc một số ít public IP để giao tiếp ra bên ngoài.

Bài viết này chủ yếu trả lời một số câu hỏi:

1. NAT chủ yếu giải quyết vấn đề gì?
2. NAT conversion table ghi lại địa chỉ mạng nội bộ, địa chỉ public và ánh xạ port như thế nào?
3. Khi host trong mạng nội bộ truy cập public network, source IP và port sẽ thay đổi như thế nào?
4. NAT có những hạn chế nào, chẳng hạn tại sao việc external chủ động truy cập host trong mạng nội bộ lại phức tạp hơn?

## Trường hợp sử dụng

**Giao thức NAT (Network Address Translation)** có trường hợp sử dụng đúng như tên gọi của nó: chuyển đổi địa chỉ mạng, được áp dụng trong quá trình chuyển đổi địa chỉ từ mạng nội bộ sang mạng bên ngoài. Cụ thể, trong một subnet nhỏ (mạng cục bộ, Local Area Network, LAN), mỗi host sử dụng IP trong cùng một LAN, nhưng bên ngoài LAN đó, trong mạng diện rộng (Wide Area Network, WAN), cần một IP thống nhất để xác định vị trí của LAN đó trên toàn bộ Internet.

Trường hợp này thực ra không khó hiểu. Cùng với sự xuất hiện của các văn phòng nhỏ và văn phòng gia đình (Small Office, Home Office, SOHO), để quản lý các SOHO này, nhiều subnet được thiết kế, khiến số lượng host trên toàn Internet trở nên rất lớn. Nếu mỗi host đều có một IP “duy nhất tuyệt đối”, khả năng biểu diễn của địa chỉ IPv4 có thể nhanh chóng đạt giới hạn ($2^{32}$). Vì vậy, trên thực tế, IP trong subnet SOHO là “tương đối”, phần nào giảm áp lực phân bổ địa chỉ IPv4.

“Đại diện” của subnet SOHO, tức cửa sổ kết nối với bên ngoài, thường do router đảm nhiệm. Phía LAN của router quản lý một subnet nhỏ, còn WAN interface của nó mới là interface thực sự tham gia vào Internet và có một “địa chỉ duy nhất tuyệt đối”. Giao thức NAT chính là thành phần đóng vai trò chuyển đổi địa chỉ khi host trong LAN giao tiếp với bên ngoài LAN.

## Chi tiết

![NAT chuyển đổi địa chỉ private trong mạng nội bộ thành địa chỉ public](https://oss.javaguide.cn/github/javaguide/cs-basics/network/nat-demo.png)

Giả sử bối cảnh hiện tại như hình trên. Ở giữa là một router, phía bên phải tổ chức một LAN có network number là `10.0.0/24`. IP của LAN-side interface là `10.0.0.4`, và trong subnet đó có ít nhất ba host, lần lượt là `10.0.0.1`, `10.0.0.2` và `10.0.0.3`. Phía bên trái router kết nối với WAN, IP của WAN-side interface là `138.76.29.7`.

Trước hết, cần nêu các sự thật sau từ những thông tin trên:

1. Network address của subnet bên phải router là `10.0.0.0/24` (network prefix chiếm 24 bit, host number chiếm 8 bit), địa chỉ của ba host và địa chỉ LAN-side interface của router đều do DHCP protocol quy định. DHCP này chạy bên trong router (router tự duy trì một DHCP server nhỏ), từ đó cung cấp DHCP service cho các thiết bị trong subnet.
2. Địa chỉ WAN-side interface của router cũng do DHCP protocol quy định, nhưng địa chỉ này do router nhận từ ISP (Internet Service Provider), tức DHCP thường chạy trên DHCP server trong khu vực nơi router đặt.

Hiện tại, router còn chạy giao thức NAT bên trong để cung cấp dịch vụ chuyển đổi địa chỉ cho giao tiếp LAN-WAN. Vì vậy, một cấu trúc rất quan trọng là **NAT conversion table**. Để giải thích chi tiết hoạt động của NAT, giả sử có request sau:

1. Host `10.0.0.1` gửi HTTP request (chẳng hạn request page) đến Web server có IP `128.119.40.186` (port 80). Lúc này, host `10.0.0.1` được gán ngẫu nhiên một port, chẳng hạn `3345`, làm source port của request này, rồi gửi request đến router (destination address là `128.119.40.186`, nhưng request sẽ đến `10.0.0.4` trước).
2. `10.0.0.4`, tức LAN interface của router, nhận request từ `10.0.0.1`. Router gán cho request này một source port mới, chẳng hạn `5001`, rồi gửi request packet đến WAN interface `138.76.29.7`. Đồng thời, trong NAT conversion table ghi lại một conversion record **138.76.29.7:5001——10.0.0.1:3345**.
3. Request packet đến WAN interface rồi tiếp tục được gửi đến destination host `128.119.40.186`.

Sau đó, response sẽ diễn ra như sau:

1. Host `128.119.40.186` nhận request, tạo response packet rồi gửi đến destination `138.76.29.7:5001`.
2. Response packet đến WAN interface của router. Router tra NAT conversion table, phát hiện `138.76.29.7:5001` có record trong conversion table, từ đó chuyển destination address và destination port thành `10.0.0.1:3345`, rồi gửi đến `10.0.0.4`.
3. Response packet sau khi chuyển đổi đến LAN interface của router, sau đó được forward đến destination `10.0.0.1`.

![Cung cấp chuyển đổi địa chỉ cho giao tiếp LAN-WAN](https://oss.javaguide.cn/github/javaguide/cs-basics/network/nat-demo2.png)

🐛 Đính chính (xem [issue#2009](https://github.com/Snailclimb/JavaGuide/issues/2009)): giá trị Dest ở bước thứ tư trong hình trên phải là `10.0.0.1:3345` thay vì ~~`138.76.29.7:5001`~~; đây là lỗi viết nhầm.

## Trọng tâm

Với quy trình trên, cần nhấn mạnh các điểm sau:

1. Trường port có 16 bit, không thể suy ra rằng phía sau một NAT nhiều nhất chỉ có khoảng 65.500 host. Giới hạn của không gian port là số lượng conversion mapping có thể được duy trì đồng thời trong điều kiện địa chỉ bên ngoài, transport protocol, mapping behavior và mapping lifetime cụ thể, chứ không phải tổng số host trong mạng nội bộ. Một host có thể tạo nhiều mapping, NAT cũng có thể sử dụng nhiều public address.
2. Đối với destination server, nó không bao giờ biết “rốt cuộc host nào đã gửi request cho mình”, mà chỉ biết đây là request được router forward từ `138.76.29.7:5001`. Vì vậy, có thể nói **router đóng vai trò che chắn giữa WAN và LAN**: mọi packet do host nội bộ gửi ra ngoài đều có cùng một IP (khác port), còn mọi packet từ bên ngoài gửi vào nội bộ cũng chỉ có một destination (khác port); sau khi được NAT chuyển đổi, packet từ bên ngoài mới có thể đến đúng host nội bộ.
3. Việc NAT có reuse mapping sẵn có hay không không thể chỉ dựa vào IP nội bộ. Mapping ít nhất phải phân biệt transport protocol, internal IP và internal port; việc có phụ thuộc vào remote address và port hay không còn tùy mapping behavior cụ thể của NAT. Chỉ khi packet khớp với mapping sẵn có, NAT mới có thể reuse public address và port tương ứng.

Tóm tắt các đặc điểm của giao thức NAT:

1. Giao thức NAT ẩn LAN khỏi WAN, qua đó giảm áp lực phân bổ địa chỉ IPv4 một cách hiệu quả.
2. Khi IP của host trong LAN thay đổi, không cần thông báo cho WAN.
3. Khi ISP của WAN thay đổi interface address, không cần thông báo cho các host trong LAN.
4. NAT ẩn địa chỉ và topology bên trong; filtering behavior của nhiều NAT device còn khiến traffic bên ngoài không có mapping sẵn khó trực tiếp đến host nội bộ. Tuy nhiên, thứ quyết định packet inbound nào có thể đi qua là filtering policy, không phải bản thân việc chuyển đổi địa chỉ. NAT không thể thay thế stateful firewall, access control và các biện pháp bảo mật cho host.

Tuy nhiên, do tính đặc thù, giao thức NAT cũng gây ra một số tranh luận. Có thể bạn đã nhận ra rằng, **khi xác định một host nội bộ bên ngoài LAN, giao thức NAT sử dụng port, vì các IP đều giống nhau**. Việc dùng port để address host có thể gây ra một số hiểu lầm. Ngoài ra, router là device thuộc network layer nhưng lại sửa nội dung packet của transport layer (sửa source IP và port), cũng là hành vi không chuẩn. Tuy vậy, dù là sản phẩm của thời đại IPv4, giao thức NAT đã giúp giải quyết rất nhiều vấn đề vốn khó xử lý và vẫn được sử dụng cho đến ngày nay.

<!-- @include: @article-footer.snippet.md -->
