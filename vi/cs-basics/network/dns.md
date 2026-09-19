---
title: Giải thích chi tiết hệ thống tên miền DNS (tầng ứng dụng)
description: Giải thích chi tiết cấu trúc phân cấp và quy trình phân giải của DNS, bao quát truy vấn đệ quy/lặp, cache và authoritative server, làm rõ port tầng ứng dụng cùng các điểm chính trong tối ưu performance.
category: Computer Fundamentals
tag:
  - Computer Network
head:
  - - meta
    - name: keywords
      content: DNS,phân giải tên miền,truy vấn đệ quy,truy vấn lặp,cache,authoritative DNS,port 53,UDP
---

Sau khi nhập domain name vào thanh địa chỉ của trình duyệt, trước khi thực hiện HTTP request, trình duyệt thường phải phân giải DNS.

DNS giải quyết **bài toán ánh xạ giữa domain name và địa chỉ IP**. Thoạt nhìn, nó chỉ là “dịch domain name thành IP”, nhưng phía sau liên quan đến cả một hệ thống cơ chế gồm cache cục bộ, truy vấn đệ quy, truy vấn lặp, authoritative server, root server và chuyển đổi giữa UDP/TCP.

Bài viết này chủ yếu trả lời một số câu hỏi:

1. Vì sao DNS cần thiết kế theo nhiều tầng?
2. Một lần phân giải domain name hoàn chỉnh thường trải qua những bước nào?
3. Truy vấn đệ quy và truy vấn lặp khác nhau như thế nào?
4. Vì sao DNS thường dựa trên UDP, và trong trường hợp nào sẽ chuyển sang TCP?

![Tổng quan hệ thống phân giải domain name thành địa chỉ IP của DNS](https://oss.javaguide.cn/github/javaguide/cs-basics/network/dns-overview.png)

Trong thực tế, có một trường hợp trình duyệt có thể biết ánh xạ giữa domain name và địa chỉ IP mà không cần dùng đến DNS. Trình duyệt duy trì một danh sách `hosts` cục bộ. Thông thường, trước tiên trình duyệt sẽ kiểm tra domain name cần truy cập có trong danh sách `hosts` hay không. Nếu có, nó trực tiếp lấy bản ghi địa chỉ IP tương ứng. Nếu danh sách `hosts` cục bộ không có bản ghi ánh xạ giữa domain name và IP, DNS mới được sử dụng.

DNS được thiết kế theo cấu trúc database phân tán, phân cấp. **DNS là application layer protocol, thường sử dụng UDP và port 53**. Khi dữ liệu response vượt quá giới hạn độ dài của UDP packet (512 byte, EDNS0 có thể mở rộng lên lớn hơn), hoặc khi thực hiện truyền zone (Zone Transfer), nó sẽ chuyển sang TCP để bảo đảm dữ liệu đầy đủ.

![Tổng quan protocol của các tầng TCP/IP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/network-protocol-overview.png)

## DNS server

DNS có thể được mô tả theo hai chiều. Về phân cấp authoritative, có root server, top-level domain server và authoritative server của từng zone cụ thể; về phía truy vấn, có các vai trò như stub resolver, recursive resolver và forwarder. Cùng một phần mềm hoặc server có thể đảm nhận nhiều vai trò, nên các nhóm này không phải những “loại server” loại trừ lẫn nhau hay bao quát toàn bộ.

- Root DNS server. Root server cung cấp cho bên truy vấn thông tin referral đến top-level domain server.
- Top-level domain DNS server (TLD server). Top-level domain là hậu tố của domain name, chẳng hạn `com`, `org`, `net` và `edu`. Các quốc gia và khu vực cũng có top-level domain riêng, chẳng hạn `uk`, `fr` và `ca`. TLD server thường trả về thông tin referral của authoritative server cho domain mục tiêu.
- Authoritative DNS server. Authoritative server lưu dữ liệu của một hoặc nhiều DNS zone và đưa ra câu trả lời authoritative cho các query trong những zone đó.
- Recursive resolver. Local network, ISP hoặc public DNS service thường cung cấp recursive resolver. Nó nhận query của client, trước tiên kiểm tra cache, khi cần thì lần lượt query root server, TLD server và authoritative server. Recursive resolver là vai trò phía truy vấn, không phải một tầng trong phân cấp authoritative DNS.

**Trên thế giới thực sự chỉ có 13 root server sao?** Đây là một hiểu lầm kỹ thuật đã tồn tại từ lâu. Nếu tìm kiếm trên Internet, bạn vẫn có thể thấy nhiều bài viết cũ tuyên bố rằng “toàn cầu chỉ có 13 root server và tất cả đều do Hoa Kỳ kiểm soát”.

**Sự thật không phải như vậy.**

Về mặt logic, hệ thống root server có 13 identifier được đặt tên, từ `a.root-servers.net` đến `m.root-servers.net`, do 12 tổ chức vận hành độc lập phụ trách. Con số này liên quan đến giới hạn kích thước packet khi DNS sử dụng UDP trong giai đoạn đầu, nhưng không thể hiểu rằng toàn cầu chỉ có 13 physical server.

Đằng sau mỗi root server identifier có thể triển khai nhiều physical instance thông qua **IP Anycast**. BGP sẽ dựa trên network route hiện tại để hướng query đến instance phù hợp trên tuyến đường, không nhất thiết là instance gần nhất về mặt địa lý. Số lượng và địa điểm của các instance sẽ liên tục thay đổi; cần lấy dữ liệu thời gian thực từ **[Root-Servers.org](https://root-servers.org/)** làm chuẩn.

![Phân bố instance root server trên toàn cầu do Root-Servers.org hiển thị](https://oss.javaguide.cn/github/javaguide/cs-basics/network/root-servers-org.png)

## Quy trình hoạt động của DNS

Dưới đây là ví dụ minh họa quy trình query và resolution của DNS. Quy trình query và resolution của DNS được chia thành hai mode:

- **Lặp**
- **Đệ quy**

Hình dưới đây minh họa cách thường được sử dụng trong thực tế: query từ host gửi request đến local DNS server là đệ quy, các query còn lại là lặp.

![Quy trình resolution kết hợp truy vấn đệ quy và truy vấn lặp của DNS](https://oss.javaguide.cn/github/javaguide/cs-basics/network/DNS-process.png)

Hiện tại, host `cis.poly.edu` muốn biết địa chỉ IP của `gaia.cs.umass.edu`. Giả sử local DNS server của host `cis.poly.edu` là `dns.poly.edu`, còn authoritative DNS server của `gaia.cs.umass.edu` là `dns.cs.umass.edu`.

1. Trước tiên, host `cis.poly.edu` gửi một DNS request đến local DNS server `dns.poly.edu`. Query packet này chứa domain name cần phân giải là `gaia.cs.umass.edu`.
2. Local DNS server `dns.poly.edu` kiểm tra local cache, phát hiện không có record, đồng thời không biết cần tìm địa chỉ IP của `gaia.cs.umass.edu` ở đâu, nên buộc phải gửi request đến root server.
3. Root server nhận thấy query packet chứa top-level domain `edu`, nên cho local DNS biết có thể gửi request đến TLD DNS của `edu`, vì địa chỉ IP của domain mục tiêu rất có thể nằm ở đó.
4. Local DNS lấy được địa chỉ của TLD DNS server của `edu`, rồi gửi request đến đó để hỏi địa chỉ IP của `gaia.cs.umass.edu`.
5. TLD DNS server của `edu` vẫn không biết địa chỉ IP của domain được query, nhưng nhận thấy domain này có phần `umass.edu`, nên trả lời cho local DNS rằng authoritative server của `umass.edu` có thể đã ghi địa chỉ IP của domain mục tiêu.
6. Lần này, local DNS gửi request đến authoritative DNS server `dns.cs.umass.edu`.
7. Sau cùng, vì `gaia.cs.umass.edu` đã được đăng ký trên authoritative DNS server, tại đó có record địa chỉ IP của nó. Authoritative DNS trả thành công địa chỉ IP về local DNS.
8. Cuối cùng, local DNS lấy được địa chỉ IP của domain mục tiêu và trả nó về host gửi request.

Ngoài query dạng lặp, còn có query dạng đệ quy như hình dưới đây. Quy trình cụ thể tương tự như trên, chỉ khác về thứ tự.

![Quy trình truy vấn đệ quy để phân giải domain name của DNS](https://oss.javaguide.cn/github/javaguide/cs-basics/network/DNS-process2.png)

Recursive resolver sẽ cache referral và resource record nhận được từ các query trước đó, vì vậy nhiều query không cần lần nào cũng bắt đầu từ root server. Chỉ cần cache liên quan chưa hết TTL, resolver có thể trực tiếp liên hệ với TLD server hoặc authoritative server đã biết, từ đó rút ngắn đường đi của query và giảm tải cho upstream server.

## Định dạng DNS packet

Định dạng DNS packet như hình dưới đây:

![Định dạng các field của query packet và response packet DNS](https://oss.javaguide.cn/github/javaguide/cs-basics/network/DNS-packet.png)

DNS packet được chia thành query packet và response packet; cấu trúc packet của hai dạng này giống nhau.

- Identifier. 16 bit, dùng để nhận diện query đó. Identifier này sẽ được sao chép vào response packet cho query, để client dùng nó đối chiếu request đã gửi với response đã nhận.
- Flag. Bit nhận diện “query/response” dài 1 bit: `0` biểu thị query packet, `1` biểu thị response packet; bit “authoritative” dài 1 bit (khi một DNS server là authoritative DNS server của name được request và đây là response packet thì dùng flag “authoritative”); bit “recursion desired” dài 1 bit, dùng để yêu cầu rõ ràng việc thực hiện query đệ quy; bit “recursion available” dài 1 bit, dùng trong response packet để biểu thị DNS server hỗ trợ query đệ quy.
- Số lượng question, answer RR, authority RR và additional RR. Lần lượt biểu thị số lượng xuất hiện của 4 vùng dữ liệu tiếp theo.
- Question section. Chứa host name đang được query và loại question được hỏi.
- Answer section. Chứa resource record cho name trong request ban đầu. **Answer section của response packet có thể chứa nhiều RR, vì vậy một host name có thể có nhiều địa chỉ IP.**
- Authority section. Chứa record của các authoritative server khác.
- Additional section. Chứa các record hữu ích khác.

## DNS record

Khi trả lời query, DNS server cần query database của chính nó. Các entry trong database được gọi là **Resource Record (RR)**. RR cung cấp ánh xạ từ host name đến địa chỉ IP. RR là một tuple gồm 4 field `Name`, `Value`, `Type` và `TTL`.

![Các field trong tuple 4 thành phần của DNS resource record](https://oss.javaguide.cn/github/javaguide/cs-basics/network/20210506174303797.png)

`TTL` là thời gian tồn tại của record, quyết định thời điểm resource record cần bị xóa khỏi cache.

Giá trị của các field `Name` và `Value` phụ thuộc vào `Type`:

![Ý nghĩa của Name và Value trong các loại DNS resource record khác nhau](https://oss.javaguide.cn/github/javaguide/cs-basics/network/20210506170307897.png)

- Nếu `Type=A`, `Name` là thông tin host name, còn `Value` là địa chỉ IP tương ứng với host name đó. RR này ghi lại một ánh xạ từ host name đến địa chỉ IP.
- Nếu `Type=AAAA` (rất tương tự record `A`), điểm khác biệt duy nhất là record `A` sử dụng IPv4, còn record `AAAA` sử dụng IPv6.
- Nếu `Type=CNAME` (Canonical Name Record, record tên chuẩn), `Value` là canonical host name của host có alias là `Name`. Giá trị `Value` mới là canonical host name. Record `CNAME` ánh xạ một host name sang một host name khác. Record `CNAME` được dùng để tạo alias cho record `A` hiện có. Xem ví dụ bên dưới.
- Nếu `Type=NS`, `Name` là một domain, còn `Value` là host name của authoritative DNS server biết cách lấy địa chỉ IP của host trong domain đó. Thông thường, RR như vậy do TLD server phát hành.
- Nếu `Type=MX`, `Value` là canonical host name của mail server có alias là `Name`. Vì có record `MX`, mail server có thể dùng alias giống với các server khác. Để lấy canonical host name của mail server, cần request record `MX`; để lấy canonical host name của các server khác, cần request record `CNAME`.

Record `CNAME` luôn trỏ đến một domain khác, không phải địa chỉ IP. Giả sử có DNS zone sau:

```plain
NAME                    TYPE   VALUE
--------------------------------------------------
bar.example.com.        CNAME  foo.example.com.
foo.example.com.        A      192.0.2.23
```

Khi user query `bar.example.com`, DNS Server thực tế trả về địa chỉ IP của `foo.example.com`.

## Tham khảo

- DNS server types: <https://www.cloudflare.com/zh-cn/learning/dns/dns-server-types/>
- DNS Message Resource Record Field Formats: <http://www.tcpipguide.com/free/t_DNSMessageResourceRecordFieldFormats-2.htm>
- Understanding Different Types of Record in DNS Server: <https://www.mustbegeek.com/understanding-different-types-of-record-in-dns-server/>

<!-- @include: @article-footer.snippet.md -->
