---
title: "HTTPS trong handshake: RSA và ECDHE khác nhau ở đâu? (Tầng ứng dụng)"
description: So sánh khác biệt cốt lõi giữa RSA key exchange và ECDHE key exchange trong TLS handshake, làm rõ forward secrecy, cách đặt tên cipher suite, thay đổi trong TLS 1.3 và các điểm phỏng vấn.
category: Computer Basics
tag:
  - Computer Network
head:
  - - meta
    - name: keywords
      content: HTTPS,RSA,ECDHE,TLS,handshake,forward secrecy,key exchange,cipher suite,TLS 1.3,PreMasterSecret
---

Nhiều người lần đầu học HTTPS sẽ lưu lại một ấn tượng khá sơ lược:

**HTTPS = HTTP + encryption, encryption = RSA. Vì vậy, HTTPS = RSA encryption.**

Cách hiểu này không phải tự nhiên mà có. Thời kỳ đầu, nhiều hệ thống HTTPS thực sự sử dụng rất nhiều cipher suite liên quan đến RSA, và nhiều tài liệu nhập môn cũng thích lấy RSA làm ví dụ.

Nhưng nói chính xác thì HTTPS chưa bao giờ đồng nghĩa với RSA encryption. Ngay cả trong thời đại TLS 1.0, TLS 1.1, RSA cũng chỉ là một trong các phương án có thể chọn; trong protocol còn có các phương thức key exchange như DHE. Đến TLS 1.3, static RSA key exchange đã bị loại bỏ, RSA chủ yếu xuất hiện ở các vị trí như certificate signature và authentication.

Vì vậy, điều bài viết này thực sự muốn so sánh không phải là “RSA và ECDHE, cái nào cao cấp hơn”.

**Trong RSA handshake, session key material do client tạo rồi mã hóa gửi cho server; trong ECDHE handshake, session key material không được truyền trực tiếp mà do client và server tự tính ra.**

Bài viết này chủ yếu trả lời các câu hỏi:

1. Vì sao HTTPS không đồng nghĩa với RSA encryption?
2. Session key material của RSA handshake và ECDHE handshake lần lượt được tạo ra như thế nào?
3. Vì sao ECDHE có thể cung cấp forward secrecy?
4. Vì sao TLS 1.3 loại bỏ static RSA key exchange?

Khi đã làm rõ các câu hỏi này, bạn sẽ có thể lần lượt hiểu được `PreMasterSecret`, `Server Key Exchange`, forward secrecy và lý do TLS 1.3 loại bỏ static RSA.

![RSA và ECDHE key exchange: khác biệt cốt lõi](https://oss.javaguide.cn/github/javaguide/cs-basics/network/https-rsa-ecdhe-rsa-and-ecdhe-key-exchange-core-differences.png)

## Hai vấn đề cốt lõi của TLS handshake

HTTPS vẫn sử dụng ngữ nghĩa của HTTP. Trong bối cảnh TLS 1.2 được thảo luận trọng tâm ở bài viết này, HTTP message được truyền qua TLS over TCP; còn HTTP/3 chạy trên QUIC tích hợp TLS 1.3 và không còn sử dụng TCP.

Sau khi handshake hoàn tất, dữ liệu nghiệp vụ thực sự được bảo vệ thường là các symmetric encryption algorithm như AES-GCM, chứ không phải dùng RSA để mã hóa toàn bộ request và response.

Ở đây có hai vấn đề.

**Vấn đề thứ nhất: browser và server cần negotiate một session key.**

Khi truyền HTTP request, Cookie và response body về sau, hai bên sẽ dùng session key này để symmetric encryption. Symmetric encryption phù hợp hơn với việc xử lý lượng lớn dữ liệu; asymmetric encryption có chi phí tính toán cao nên thường không được dùng để trực tiếp mã hóa toàn bộ nội dung web.

**Vấn đề thứ hai: browser cần xác nhận đối phương thực sự là website mục tiêu.**

Nếu chỉ “server gửi một public key cho browser”, thì attacker ở giữa cũng có thể gửi public key của mình. Browser tưởng đó là public key của website mục tiêu, rồi mã hóa thông tin bí mật gửi cho attacker. Certificate, CA và digital signature giải quyết vấn đề này: chứng minh public key thực sự được liên kết với domain đó, chứ không phải do ai đó trên đường truyền nhét vào.

RSA handshake và ECDHE handshake đều phải giải quyết hai vấn đề này. Chỉ là cách chúng giải quyết việc “session key đến từ đâu” khác nhau.

## RSA handshake: gửi key material đã mã hóa

### Quy trình handshake đầy đủ

Trước hết hãy xem RSA key exchange trong TLS 1.2.

Browser trước tiên gửi `ClientHello`. Message này mang theo TLS version mà client hỗ trợ, cipher suite được hỗ trợ và một random value `Client Random`.

Sau khi nhận được, server trả về `ServerHello`, chọn TLS version và cipher suite, đồng thời đưa ra một random value `Server Random`, sau đó gửi certificate của mình cho client.

Đến đây, client đã nhận được certificate của server. Nó sẽ verify certificate chain, domain, validity period, signature và các thông tin khác. Sau khi certificate verification thành công, client lấy RSA public key của server từ certificate.

Tiếp theo là bước quan trọng: client tạo một random value mới, tức `PreMasterSecret`. Trong RSA key exchange của TLS 1.2, giá trị này dài **48 byte**. Client dùng RSA public key trong certificate của server để mã hóa `PreMasterSecret`, rồi đặt kết quả mã hóa vào `Client Key Exchange` và gửi cho server.

Sau khi nhận được, server dùng RSA private key của mình để giải mã và lấy được cùng một `PreMasterSecret`.

Lúc này, client và server đều có ba phần material:

```text
Client Random
Server Random
PreMasterSecret
```

Hai bên tiếp tục derive `Master Secret` từ ba phần material này, rồi session key dùng về sau cũng tiếp tục được derive từ đây. Khi thực sự truyền HTTP request và response, các symmetric key được derive này mới là thứ được sử dụng.

Tóm gọn trong một câu:

**Session key material của RSA handshake do client tạo rồi “đóng gói” gửi cho server.**

Ở đây, “đóng gói” dựa vào RSA public key của server. Chỉ server có RSA private key tương ứng mới có thể mở gói này.

Thoạt nhìn, cách này khá hợp lý. Client tạo secret, server dùng private key để giải mã, hai bên nhận được cùng một phần material, sau đó kết hợp với hai random value để derive session key cho các bước tiếp theo.

Nhưng vấn đề cũng nằm ở đây.

### Không có forward secrecy: long-term private key quá quan trọng

Giả sử hôm nay attacker bắt được một đoạn HTTPS traffic, nhưng lúc đó không có private key của server nên không thể đọc nội dung. Khi ấy, attacker có thể lưu traffic lại trước.

Một năm sau, nếu RSA private key của server bị lộ thì chuyện gì sẽ xảy ra?

Trong RSA key exchange, `PreMasterSecret` mà client gửi năm đó được mã hóa bằng RSA public key của server. Nếu attacker đã bắt trọn các random value dạng plaintext trong handshake, tức `Client Random` và `Server Random`, đồng thời lưu `PreMasterSecret` đã mã hóa, thì khi kết hợp với private key của server bị lộ sau này, attacker có thể giải mã `PreMasterSecret` của thời điểm đó, rồi tiếp tục derive session key đã được dùng cho connection đó.

Dữ liệu cũ có cơ hội bị giải mã trở lại.

Cần lưu ý điều kiện ở đây: không phải “chỉ cần private key bị lộ là mọi traffic trong quá khứ chắc chắn giải mã được”. Attacker ít nhất phải lấy được handshake data và application data đủ đầy. Nếu chỉ có một chiều dữ liệu hoặc handshake log không đầy đủ thì ngay cả khi có private key, attacker chưa chắc khôi phục được session đó.

Nhưng xét về security design, rủi ro này đã đủ nghiêm trọng. Khi long-term private key trở thành master key để mở historical traffic, ảnh hưởng của nó không còn chỉ bao phủ các connection trong tương lai mà còn lan sang những communication đã xảy ra trong quá khứ.

Điều bị chỉ trích ở đây không phải bản thân RSA algorithm là “không thể dùng”. RSA vẫn có thể được dùng cho signature authentication và vẫn có thể xuất hiện trong certificate system. Vấn đề nằm ở việc “dùng private key không đổi trong thời gian dài của server để giải mã key material trong các historical handshake”.

Khi private key của server bị lộ, cái giá phải trả quá lớn.

![Static RSA thiếu forward secrecy: bắt trọn traffic + private key bị lộ có thể truy ngược historical traffic](https://oss.javaguide.cn/github/javaguide/cs-basics/network/https-rsa-ecdhe-static-rsa-lacks-forward-secrecy.png)

### Một gánh nặng lịch sử khác: padding oracle attack

RSA key exchange còn có một rắc rối ở tầng engineering: `PreMasterSecret` không được mã hóa trực tiếp mà trước hết được đóng gói theo format như RSAES-PKCS1-v1_5 rồi mới mã hóa.

Chi tiết này từng dẫn đến các padding oracle attack như Bleichenbacher.

Ý tưởng đại khái là: attacker không nhất thiết phải lấy được private key của server ngay lập tức, mà có thể liên tục tạo các ciphertext khác nhau gửi cho server, rồi quan sát sự khác biệt trong cách server xử lý “padding error, version error, length error”. Nếu server để lộ khác biệt ở error code, response time, log behavior hoặc cách đóng connection, attacker có thể từng bước tiến gần đến plaintext.

Điểm rắc rối của loại attack này là nó không chỉ là vấn đề toán học mà còn là vấn đề implementation.

TLS 1.2 từng đưa ra yêu cầu phòng vệ cho tình huống này: ngay cả khi giải mã thất bại, server cũng không được để lộ lý do thất bại cụ thể mà phải tiếp tục dùng random value để đi hết flow, tránh cho attacker phán đoán ciphertext có gần đúng format hay không thông qua behavior khác biệt.

Nhưng yêu cầu trong standard không đồng nghĩa với implementation đáng tin cậy. ROBOT attack năm 2017 một lần nữa cho thấy một số server vẫn có thể để lộ RSA decryption oracle do những khác biệt nhỏ trong behavior. Chỉ cần một điểm không nhất quán trong error code, thời gian xử lý, log hoặc branch path cũng có thể trở thành side channel.

Vì vậy, static RSA key exchange bị loại bỏ không chỉ vì không có forward secrecy, mà còn vì nó đẩy quá nhiều rủi ro vào các chi tiết implementation.

### Có thể bị downgrade về RSA không?

Ở đây cần bổ sung một điểm dễ bị hiểu nhầm.

Trong TLS 1.2, client mang danh sách cipher suite được hỗ trợ trong `ClientHello`, còn server chọn một suite mà cả hai bên đều hỗ trợ. Về lý thuyết, nếu server vẫn mở các static RSA key exchange suite như `TLS_RSA_*`, client cũ vẫn có thể tiếp tục dùng RSA handshake.

Nhưng điều đó không có nghĩa là “attacker ở giữa chỉ cần tùy ý xóa ECDHE khỏi ClientHello là có thể âm thầm downgrade connection xuống RSA”. `Finished` ở cuối handshake sẽ verify handshake transcript; việc sửa đơn giản `ClientHello` thường sẽ khiến verification thất bại và connection không thể được thiết lập.

Trong lịch sử thực sự đã xảy ra các attack liên quan đến downgrade, chẳng hạn FREAK và Logjam. Chúng lợi dụng việc một số client và server khi đó vẫn hỗ trợ các weak cipher suite dành cho export, sau đó kết hợp với vấn đề implementation và configuration để ép connection đi theo RSA_EXPORT hoặc DHE_EXPORT yếu hơn, chứ không phải “tùy ý xóa ECDHE là có thể âm thầm thành công”. TLS 1.3 thêm downgrade protection value vào `ServerHello.random` cũng nhắc chúng ta rằng protocol vẫn luôn phải bổ sung các lỗ hổng attack trong lịch sử này.

Điều thực sự cần quan tâm là configuration của server: nếu không còn cần tương thích với client quá cũ thì nên tắt static RSA key exchange suite và chỉ giữ các suite hỗ trợ forward secrecy. Nếu không, trong environment vẫn có thể tồn tại client hoặc configuration sai khiến RSA handshake được sử dụng.

Đây cũng là lý do khi kiểm tra TLS configuration cần xem kết quả cipher suite được negotiate thực tế. Chỉ nhìn “server hỗ trợ ECDHE” là chưa đủ; còn phải xem server có đồng thời giữ các suite cũ như `TLS_RSA_*` hay không.

## ECDHE handshake: hai bên negotiate key material

### Ý tưởng cốt lõi của DH

`DHE` trong ECDHE đến từ Diffie-Hellman Ephemeral, nghĩa là Diffie-Hellman tạm thời. `EC` ở phía trước là Elliptic Curve, biểu thị việc dựa trên elliptic curve.

Đừng bị cái tên làm cho sợ. Tạm thời chưa xét elliptic curve, hãy xem trước DH muốn giải quyết vấn đề gì.

Mục tiêu của DH khá thú vị: hai bên communication không truyền shared secret trực tiếp nhưng vẫn có thể tự tính ra cùng một shared secret.

Có thể hiểu sơ lược như sau:

Client tạo một temporary private key, chỉ giữ ở local, rồi tính ra một temporary public key và gửi cho server. Server cũng tạo một temporary private key, chỉ giữ ở local, rồi tính ra một temporary public key và gửi cho client.

Hai bên chỉ trao đổi public key. Attacker trên network có thể nhìn thấy các public key này nhưng không nhìn thấy temporary private key riêng của mỗi bên.

Tiếp theo, client dùng “temporary private key của mình + temporary public key của server” để tính shared secret; server dùng “temporary private key của mình + temporary public key của client” cũng tính ra cùng một shared secret.

Shared secret chưa từng được truyền trên network.

ECDHE chỉ đưa quy trình này vào elliptic curve system để thực hiện. Lý thuyết toán học của elliptic curve trừu tượng hơn, nhưng với cùng mức security strength, nó thường có thể dùng key ngắn hơn để đạt mức security tương đương; chi phí tính toán và truyền tải cũng thấp hơn DHE trên finite field truyền thống. Để hiểu TLS handshake, chỉ cần nhớ một câu:

**Session key material của ECDHE không do một bên tạo rồi gửi cho bên kia mà do hai bên negotiate thông qua temporary key.**

### Quy trình handshake đầy đủ

Tiếp theo hãy xem `ECDHE_RSA` handshake thường gặp trong TLS 1.2.

Client vẫn gửi `ClientHello` trước, trong đó có TLS version, cipher suite được hỗ trợ và `Client Random`. Server trả về `ServerHello`, chọn một cipher suite, chẳng hạn:

```text
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
```

Tên cipher suite này cần được tách ra xem xét, không thể thấy RSA rồi cho rằng nó vẫn dùng RSA để mã hóa session key.

- `ECDHE` biểu thị phương thức key exchange.
- `RSA` biểu thị phương thức authentication signature.
- `AES_256_GCM` biểu thị record data về sau sử dụng AES, key length là 256 bit và mode là GCM.
- `SHA384` chỉ định hash algorithm được TLS 1.2 PRF và `Finished` message sử dụng.

Bản thân GCM đã cung cấp integrity protection ở record layer, nên `SHA384` ở đây không còn biểu thị record-layer MAC mà chủ yếu tham gia vào key derivation và verification ở handshake phase.

Tiếp theo server gửi certificate. Lấy `ECDHE_RSA` làm ví dụ, RSA public key trong certificate chủ yếu dùng để verify server signature, chứ không phải để client dùng nó mã hóa `PreMasterSecret`.

Sau đó, ECDHE và RSA handshake bắt đầu đi theo hai nhánh khác nhau.

Trong ECDHE handshake, server sẽ gửi `Server Key Exchange`. Message này chứa elliptic curve parameter do server chọn và temporary ECDHE public key của server.

**Vấn đề là: làm sao client biết temporary ECDHE public key này không bị attacker ở giữa thay thế?**

**Câu trả lời là signature.**

Server dùng private key tương ứng với certificate để ký handshake parameter. Sau khi nhận được, client dùng public key trong certificate để verify signature. Nếu signature verification thành công, client có thể xác nhận: temporary ECDHE public key này thực sự đến từ server đang sở hữu certificate private key, không phải bị ai đó thay thế trên đường truyền.

Sau đó client cũng tạo temporary ECDHE private key và public key của mình, rồi gửi temporary public key của client cho server thông qua `Client Key Exchange`.

Đến bước này, hai bên đã có các material cần thiết để tính shared secret.

Client có:

```text
Temporary private key của client
Temporary public key của server
Client Random
Server Random
```

Server có:

```text
Temporary private key của server
Temporary public key của client
Client Random
Server Random
```

Hai bên tự tính ra cùng một shared secret, rồi derive session key dùng ở các bước tiếp theo.

Một lần nữa cần nhấn mạnh:

**RSA trong ECDHE_RSA không dùng để mã hóa session key được truyền đi. Nó dùng để chứng minh “các temporary parameter của ECDHE này thực sự do server gửi”.**

Đây cũng là điểm khiến nhiều người dễ hiểu nhầm nhất khi nhìn tên cipher suite.

![TLS 1.2 sử dụng ECDHE RSA để hoàn thành handshake và key negotiation](https://oss.javaguide.cn/github/javaguide/cs-basics/network/https-rsa-ecdhe-tls-1-2-ecdhe-rsa-handshake-process.png)

### Đọc tên cipher suite như thế nào

Tên cipher suite của TLS 1.2 thường có thể tách theo quy tắc:

```text
TLS_Key exchange algorithm_Authentication algorithm_WITH_Symmetric encryption algorithm_Hash algorithm
```

Ví dụ:

```text
TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
```

Có thể tách thành:

```text
ECDHE: Key exchange
RSA: Authentication, tức server signature
AES_128_GCM: Record-layer encryption algorithm về sau
SHA256: Hash algorithm dùng cho TLS 1.2 PRF và Finished message; nếu là GCM suite thì không còn đóng vai trò record-layer MAC
```

Xem thêm một ví dụ:

```text
TLS_RSA_WITH_AES_128_GCM_SHA256
```

Ở đây `RSA` xuất hiện trước `WITH` và không có `ECDHE`, biểu thị key exchange và authentication đều gắn với RSA. Đây là static RSA key exchange suite điển hình.

Đến TLS 1.3, cách đặt tên cipher suite đã thay đổi, ví dụ:

```text
TLS_AES_128_GCM_SHA256
```

Bạn sẽ thấy key exchange và authentication method không còn được ghi trong tên cipher suite. TLS 1.3 tách các thông tin này vào những extension và handshake message khác; tên cipher suite chủ yếu mô tả record-layer AEAD algorithm và hash algorithm được HKDF sử dụng.

Vì vậy, khi thấy `TLS_AES_128_GCM_SHA256` của TLS 1.3, đừng hiểu nhầm rằng nó “không có key exchange”. Key exchange vẫn tồn tại, chỉ là không được viết ra theo cách đặt tên của TLS 1.2.

![Tách tên cipher suite](https://oss.javaguide.cn/github/javaguide/cs-basics/network/https-rsa-ecdhe-cipher-suite-name-decomposition.png)

## Forward secrecy và chi phí performance

### Vì sao ECDHE có forward secrecy

Điểm mấu chốt nằm ở `E`, tức `Ephemeral`, nghĩa là tạm thời.

Private key trong ECDHE handshake không phải long-term private key của server certificate mà là temporary private key được sử dụng trong quá trình handshake. Sau khi connection kết thúc, về nguyên tắc không nên tiếp tục phụ thuộc vào temporary material này.

Kết quả là: hôm nay attacker bắt được traffic, đến một ngày trong tương lai lấy được certificate private key của server thì cũng không thể chỉ dựa vào long-term private key này để khôi phục temporary shared secret trong từng handshake trước đây. Bởi thứ thực sự tham gia key negotiation lúc đó là ECDHE temporary private key của handshake ấy, không phải certificate private key.

Certificate private key ở đây giống “bút ký” hơn là “chìa khóa két sắt”.

Trong RSA key exchange, server private key có thể trực tiếp mở `PreMasterSecret` do client gửi đến; trong ECDHE, server private key chỉ ký temporary parameter để chứng minh identity. Nó không trực tiếp tham gia tính shared secret của từng connection.

Sự thay đổi vai trò này quyết định khác biệt giữa hai phương thức trong việc bảo vệ historical traffic.

![Nguyên lý forward secrecy của ECDHE: long-term key và temporary key](https://oss.javaguide.cn/github/javaguide/cs-basics/network/https-rsa-ecdhe-ecdhe-forward-secrecy-principle-long-term-key-vs-ephemeral-key.png)

Tuy nhiên, forward secrecy không phải lá bùa bất tử.

Nếu chất lượng random value của server quá kém, temporary private key bị ghi vào log hoặc implementation xuất hiện memory leak thì ECDHE cũng không thể cứu bạn. Trong implementation thực tế, để giảm chi phí handshake, một số implementation có thể reuse temporary DH/ECDH private material trong thời gian ngắn: trong finite-field DH thường gọi là “reuse exponent”, còn trong ECDH thường gọi là “reuse temporary private key/scalar”. Nếu thời gian reuse quá dài, granularity của forward secrecy sẽ trở nên thô hơn.

Một loại rủi ro khác đến từ parameter validation. Chẳng hạn, nếu server không verify đúng xem elliptic curve point do client gửi có nằm trên curve hợp lệ hay không, nó có thể tạo cơ hội cho invalid-curve attack. Developer bình thường không nhất thiết trực tiếp viết tầng code này, nhưng điều đó nhắc chúng ta rằng cryptographic protocol không kết thúc ở việc “chọn đúng algorithm”; implementation và configuration của TLS library cũng quan trọng không kém.

### Ảnh hưởng của session resumption

Còn một điểm dễ bị bỏ qua: **session resumption.**

ECDHE full handshake phải thực hiện temporary key negotiation nên chi phí không thấp. Để giảm handshake overhead, TLS hỗ trợ session resumption. Khi client lần sau truy cập cùng một site, nó có thể thử reuse session state đã negotiate trước đó để tránh phải thực hiện full handshake mỗi lần.

Vấn đề là session resumption cũng có security boundary riêng.

Lấy TLS 1.2 session ticket làm ví dụ, server dùng ticket encryption key để bảo vệ session state; lần sau client mang ticket quay lại, server giải mã ticket rồi khôi phục session. Nếu ticket encryption key này không được rotate trong thời gian dài, một khi bị lộ, attacker có thể giải mã các ticket đã thu thập trong quá khứ và tiếp tục khôi phục key material của các session được resume liên quan.

Khi đó, forward secrecy window không còn là “một connection” mà bị kéo dài thành “lifecycle của ticket encryption key”.

Vì vậy, configuration production không thể chỉ xem “đã bật ECDHE hay chưa”. Cũng cần tính đến cách tạo và rotate session ticket key, việc có share key giữa nhiều machine hay không và mức độ ảnh hưởng nếu key bị lộ.

### Performance không miễn phí

ECDHE mang lại forward secrecy nhưng cũng có chi phí.

Main path của RSA key exchange là server dùng long-term RSA private key để mở `PreMasterSecret` do client gửi đến. Còn ECDHE_RSA cần hoàn thành ECDH negotiation tạm thời và ký temporary parameter của server.

Với service có concurrency cao, TLS handshake sẽ tiêu tốn CPU, đặc biệt khi có nhiều short connection và session resumption hit rate thấp.

Không thể đơn giản viết rằng “ECDHE chắc chắn chậm hơn RSA”. Chi phí thực tế phụ thuộc vào RSA key length, lựa chọn elliptic curve, signature algorithm, TLS library implementation, CPU instruction set, session resumption hit rate và nhiều yếu tố khác. Chẳng hạn, X25519, P-256, RSA 2048 và RSA 3072 có performance khác nhau trên các CPU và TLS library khác nhau.

Nếu thực sự muốn đánh giá cost, phương pháp đáng tin cậy nhất không phải là trích dẫn một con số cố định của người khác mà là benchmark trên machine mục tiêu. Ít nhất cần phân biệt ba việc:

```text
1. Thời gian của một cryptographic operation
2. Thời gian của một TLS handshake đầy đủ
3. Thời gian end-to-end của business request
```

Có thể dùng `openssl speed` để ước lượng sơ bộ order of magnitude của mục thứ nhất, chẳng hạn test năng lực tính toán RSA, ECDH và X25519; mục thứ hai cần xem TLS library và server configuration; mục thứ ba còn chịu ảnh hưởng của network, connection reuse và application logic.

Vì vậy, production sẽ không chỉ dựa vào “chuyển sang ECDHE” để giải quyết mọi vấn đề. Cách làm phổ biến hơn là kết hợp TLS 1.3, session resumption, certificate algorithm và curve selection hợp lý, khi cần thì dùng thêm hardware acceleration.

Security và performance không phải hai lựa chọn loại trừ nhau, nhưng cũng không thể giả vờ rằng không có cost.

## Những thay đổi trong TLS 1.3

Nếu chỉ xét TLS 1.2, RSA và ECDHE có thể được so sánh như hai phương thức key exchange.

Nhưng đến TLS 1.3, static RSA key exchange đã bị loại bỏ và cấu trúc handshake cũng thay đổi.

TLS 1.2 full handshake thường cần 2 RTT. Client gửi `ClientHello` trước, server trả về `ServerHello`, certificate và các handshake message liên quan; sau đó client gửi key exchange và `Finished`, cuối cùng server trả về `Finished`.

TLS 1.3 đưa key exchange parameter vào `key_share` trong `ClientHello` từ sớm hơn. Server có thể trả về `key_share` của mình ngay trong response đầu tiên, nên full handshake thường được rút xuống còn 1 RTT.

2 RTT thành 1 RTT tiết kiệm được bao nhiêu millisecond còn tùy thuộc network environment. Trong cùng một data center có thể chỉ là vài millisecond; với kết nối khác region, mobile network hoặc tình huống packet loss cao, việc giảm một RTT sẽ dễ nhận thấy hơn.

Tuy nhiên, TLS 1.3 cũng không phải lúc nào cũng ổn định ở 1 RTT. Nếu `key_share` client gửi không khớp với curve server hỗ trợ, server sẽ trả về `HelloRetryRequest`, yêu cầu client thay một bộ parameter khác rồi thử lại. Khi đó, handshake có thể lại gần 2 RTT.

Vì vậy, trong production, client và server nên hỗ trợ thống nhất các key negotiation group phổ biến, chẳng hạn `X25519` và `secp256r1`. Nếu không, lợi thế 1 RTT của TLS 1.3 có thể bị giảm.

![So sánh handshake RTT giữa TLS 1.2 và TLS 1.3](https://oss.javaguide.cn/github/javaguide/cs-basics/network/https-rsa-ecdhe-tls-1-2-vs-tls-1-3-handshake-rtt-comparison.png)

Còn hybrid post-quantum key exchange, 0-RTT, PSK-only và mTLS thuộc một hướng khác nên bài viết này không mở rộng.

## Tra nhanh khác biệt cốt lõi giữa RSA và ECDHE

Đặt cạnh nhau sẽ thấy khác biệt rất rõ.

| Hạng mục so sánh                    | RSA key exchange                                                                                   | ECDHE key exchange                                                                                                        |
| ----------------------------------- | -------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| Bối cảnh version phổ biến           | Có trong TLS 1.2 và các version cũ hơn                                                             | Phổ biến trong TLS 1.2, TLS 1.3 tiếp tục hướng key negotiation tạm thời                                                   |
| Session key material đến từ đâu     | Client tạo `PreMasterSecret`, mã hóa bằng RSA public key của server rồi gửi đi                     | Hai bên lần lượt tạo temporary key pair và dùng ECDHE để tính shared secret                                               |
| Vai trò của server private key      | Giải mã `PreMasterSecret` do client gửi đến                                                        | Ký ECDHE temporary parameter để chứng minh parameter đến từ server thật                                                   |
| Đã truyền gì trên network           | `PreMasterSecret` đã mã hóa                                                                        | Temporary public key của hai bên và parameter đã ký                                                                       |
| Có hỗ trợ forward secrecy không     | Không hỗ trợ                                                                                       | Có hỗ trợ, với điều kiện temporary key được tạo đúng và không được giữ lại sau khi sử dụng                                |
| Ảnh hưởng sau khi private key bị lộ | Historical traffic có thể bị giải mã nếu handshake data được bắt trọn                              | Thông thường không thể giải mã historical traffic chỉ bằng certificate private key                                        |
| Vấn đề điển hình                    | Long-term private key có giá trị quá lớn, tồn tại gánh nặng lịch sử của PKCS#1 v1.5 padding oracle | Handshake có thêm chi phí tính toán, parameter validation và temporary key management phụ thuộc chất lượng implementation |
| Tình hình trong TLS 1.3             | Static RSA key exchange đã bị loại bỏ                                                              | Temporary key negotiation trở thành hướng chính                                                                           |

![Tra nhanh so sánh RSA và ECDHE](https://oss.javaguide.cn/github/javaguide/cs-basics/network/https-rsa-ecdhe-rsa-vs-ecdhe-quick-reference.png)

### Cách hiểu sai thường gặp: ECDHE_RSA không phải cả hai algorithm đều encryption

Cái tên `ECDHE_RSA` rất dễ khiến người ta nghĩ rằng cả hai algorithm đều đang encryption. Lấy cipher suite này làm ví dụ:

```text
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
```

Vai trò ở đây là: ECDHE phụ trách key exchange, RSA phụ trách authentication signature, AES-256-GCM phụ trách bảo vệ record-layer data, còn SHA384 được dùng cho TLS 1.2 PRF và `Finished` verification. RSA certificate signature và RSA key exchange là hai việc khác nhau; việc trước vẫn phổ biến trong HTTPS hiện đại, còn việc sau đã bị loại khỏi TLS 1.3.

## Trả lời phỏng vấn: RSA và ECDHE trong HTTPS handshake khác nhau ở đâu?

Khác biệt cốt lõi giữa RSA và ECDHE nằm ở việc session key material được “truyền qua” hay “negotiate ra”.

Trong static RSA handshake của TLS 1.2, client tạo `PreMasterSecret`, mã hóa bằng RSA public key trong server certificate rồi gửi cho server; server tiếp tục dùng RSA private key để giải mã. Vấn đề là nếu attacker lưu handshake traffic của thời điểm đó, sau này private key của server lại bị lộ thì attacker có thể quay lại giải mã historical session key, nên phương thức này không có forward secrecy.

ECDHE không truyền shared secret trực tiếp. Client và server lần lượt tạo temporary key pair, trao đổi temporary public key rồi tự tính cùng một shared secret ở local. Certificate private key của server chủ yếu dùng cho signature authentication, chứng minh temporary parameter không bị attacker ở giữa thay thế, chứ không dùng để giải mã session key.

Khi phỏng vấn, bạn có thể trả lời như sau: static RSA handshake của TLS 1.2 do client tạo `PreMasterSecret`, sau đó dùng RSA public key của server để mã hóa và gửi đi; nếu attacker lưu trọn handshake traffic rồi sau đó lấy được private key của server, historical session có thể bị giải mã, vì vậy nó không có forward secrecy. ECDHE không truyền shared secret trực tiếp; sau khi hai bên trao đổi temporary public key, mỗi bên tự tính cùng một secret ở local; certificate private key chủ yếu dùng cho signature authentication chứ không trực tiếp giải mã session key. TLS 1.3 đã loại bỏ static RSA key exchange và dùng (EC)DHE, PSK hoặc cách kết hợp cả hai để thiết lập key.
