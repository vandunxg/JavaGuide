---
title: "Giải thích chi tiết Bài toán các vị tướng Byzantine: đồng thuận phân tán, 3m+1 node và khả năng chịu lỗi BFT"
description: "Giải thích chi tiết Bài toán các vị tướng Byzantine, trình bày về Byzantine Fault, đồng thuận phân tán, tính an toàn và tính hoạt động, thông điệp truyền miệng OM(m), thông điệp có chữ ký, yêu cầu 3m+1 node, sự khác biệt giữa CFT và BFT, cùng ứng dụng trong blockchain."
category: Distributed
tag:
  - Distributed Protocols and Algorithms
  - Consensus Algorithms
  - BFT
head:
  - - meta
    - name: keywords
      content: Byzantine Generals Problem,Byzantine Fault Tolerance,BFT,PBFT,distributed consensus,consensus algorithms,3m+1,OM(m),signed messages,CFT,blockchain consensus,distributed systems
---

Một vài node dịch vụ đều nói mình đúng, client nên tin ai?

Trong các hệ thống backend hằng ngày, vấn đề này thường không cực đoan đến vậy. Redis master-slave, ZooKeeper, etcd, Nacos và database replication thường gặp machine crash, network chập chờn, disk failure, process restart hơn. Node thường không cố ý lừa bạn; nó chỉ không phản hồi, phản hồi chậm hoặc tạm thời mất kết nối với cluster.

Bài toán các vị tướng Byzantine thảo luận một tình huống rắc rối hơn: **trong hệ thống có những node biểu hiện không thể dự đoán, thậm chí có thể gửi thông tin mâu thuẫn cho các đối tượng khác nhau**.

Đây là giá trị của bài toán trong distributed system. Câu chuyện quân sự thời cổ đại chỉ là cách diễn đạt; điều thực sự cần nói là “đạt được đồng thuận giữa các thành viên không đáng tin cậy”.

Nếu bạn đang học các hệ thống điều phối phổ biến như Raft, ZAB, ZooKeeper và etcd, trước hết cần phân biệt hai mô hình failure: chúng thường xử lý crash failure và network partition, không giả định node cố ý nói dối; bài toán Byzantine thảo luận một mô hình failure mạnh hơn. Nếu muốn xem consensus trong bối cảnh không có Byzantine, bạn có thể đọc tiếp [Giải thích chi tiết thuật toán Raft](./raft-algorithm.md) và [Giải thích chi tiết giao thức ZAB](./zab.md).

## Bài toán các vị tướng Byzantine là gì?

Bài toán các vị tướng Byzantine do Leslie Lamport, Robert Shostak và Marshall Pease đề xuất trong bài báo [The Byzantine Generals Problem](https://www.microsoft.com/en-us/research/publication/byzantine-generals-problem/) công bố năm 1982. Bài báo được đăng trên ACM Transactions on Programming Languages and Systems vào tháng 7 năm 1982.

Lamport là một nhân vật không thể bỏ qua trong lĩnh vực distributed system. Ông nhận ACM A.M. Turing Award năm 2013, thường được gọi là giải Turing, nhờ những đóng góp nền tảng cho distributed system và concurrent system. Ông cũng là developer đầu tiên của hệ thống dàn trang tài liệu LaTeX.

Phiên bản câu chuyện của bài toán đại khái như sau:

Nhiều vị tướng Byzantine lần lượt chỉ huy quân đội bao vây một thành phố địch. Mỗi đội quân phân tán ở một vị trí khác nhau, các vị tướng không thể họp trực tiếp mà chỉ có thể truyền tin qua messenger. Họ cần quyết định ngày mai sẽ tấn công hay rút lui. Tấn công riêng lẻ sẽ thất bại; chỉ khi đủ nhiều đội quân cùng hành động mới có cơ hội chiến thắng.

Rắc rối nằm ở chỗ trong số các vị tướng có thể có kẻ phản bội.

Kẻ phản bội không nhất thiết chỉ “không thực hiện mệnh lệnh”. Hắn có thể nói với tướng A rằng hãy tấn công, nói với tướng B rằng hãy rút lui; cũng có thể giả mạo thông tin mình nghe được để dụ các vị tướng trung thành đưa ra những quyết định khác nhau. Trong computer system, hành vi tương tự không nhất thiết xuất phát từ ác ý chủ quan, mà cũng có thể đến từ software bug, state corruption, lỗi memory hoặc disk, hay hành vi bất thường sau khi node bị xâm nhập. Cuối cùng, nếu một bộ phận tướng trung thành tấn công còn bộ phận khác rút lui, toàn bộ kế hoạch tác chiến sẽ thất bại.

Đặt vào distributed system, tướng là node, messenger là network message, còn tấn công/rút lui là một giá trị cần đạt đồng thuận, chẳng hạn:

- Node nào trở thành Leader?
- Một log entry có được commit hay không?
- Một transaction có hợp lệ hay không?
- State machine sẽ thực thi command nào ở bước tiếp theo?

Vì vậy, bài toán này có thể được diễn đạt bằng ngôn ngữ engineering như sau:

> Khi một số node có thể failure, nói dối, giả mạo thông tin hoặc gửi thông tin mâu thuẫn, làm thế nào để tất cả node bình thường đạt đồng thuận về cùng một kết quả?

![Bối cảnh cơ bản của Bài toán các vị tướng Byzantine](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/byzantine-generals-problem-problem-overview.png)

Cần làm rõ một chi tiết trước. Nhiều bài viết thường đưa cả việc messenger bị sát hại, message bị mất hoặc bị sửa vào câu chuyện để giúp dễ hiểu “communication không đáng tin cậy”. Nhưng để chứng minh hình thức, mô hình “thông điệp truyền miệng” trong bài báo của Lamport lại giả định communication system thỏa mãn một số điều kiện: message đã gửi sẽ được chuyển đến chính xác, receiver biết message do ai gửi và việc không nhận được message cũng có thể được phát hiện.

Vì vậy, điểm khó mà bài báo thực sự cần xử lý còn tiến thêm một bước so với việc “network làm mất packet”: **người gửi message có thể hành động ác ý**.

## Đạt đồng thuận cần thỏa mãn điều gì?

Trong Bài toán các vị tướng Byzantine có một commander và một số lieutenant. Commander cần gửi command tấn công hoặc rút lui cho các lieutenant; hệ thống mong muốn thỏa mãn hai điều kiện:

- **Tính nhất quán (Agreement)**: mọi lieutenant trung thành cuối cùng thực thi cùng một command.
- **Tính hợp lệ (Validity)**: nếu commander trung thành, mọi lieutenant trung thành đều phải thực thi command do commander gửi.

Điều kiện thứ nhất yêu cầu mọi người không bị chia rẽ. Điều kiện thứ hai yêu cầu hệ thống không vì chống kẻ phản bội mà làm mất cả command đúng của commander trung thành.

Nói chính xác hơn, mô hình commander-lieutenant trong bài báo của Lamport gần với **Byzantine Broadcast** hoặc **Interactive Consistency** hơn. Nó có quan hệ rất gần với consensus thông thường, nhưng interface không hoàn toàn giống nhau: consensus thông thường thường cho phép mỗi node đề xuất một initial value, sau đó yêu cầu các node bình thường quyết định cùng một value; còn mô hình commander-lieutenant do một commander gửi command và lieutenant chịu trách nhiệm phán đoán cần thực thi gì.

Trong bài toán distributed consensus tổng quát hơn, người ta còn quan tâm đến termination, tức node bình thường cuối cùng phải đưa ra quyết định và không được chờ vô hạn. Một số định nghĩa còn bổ sung integrity: một node nhiều nhất chỉ được quyết định một lần. Trong các hệ thống engineering, timeout, retry, election round và view change thường phục vụ mục tiêu này.

![Minh họa majority vote của các node trung thành](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/byzantine-generals-problem-honest-majority.png)

Trước hết hãy thu hẹp bài toán về câu chuyện các vị tướng. Giả sử chỉ có 3 vị tướng A, B, C, trong đó A là commander còn B và C là lieutenant. Chỉ cần 1 vị tướng là kẻ phản bội thì tình huống sẽ bị mắc kẹt.

Từ góc nhìn cục bộ của lieutenant trung thành B, hai quá trình thực thi sau có thể hoàn toàn giống nhau:

- A là kẻ phản bội: A nói với B “tấn công”, nói với C “rút lui”; C trung thành thuật lại cho B rằng “rút lui”.
- C là kẻ phản bội: A là commander trung thành, nói với cả B và C “tấn công”; nhưng C lại nói dối với B rằng A đã nói “rút lui”.

Trong hai tình huống này, B đều thấy: A trực tiếp nói với mình “tấn công”, còn C thuật lại rằng A nói “rút lui”. Chỉ dựa vào message mình nhận được, B không thể phán đoán rốt cuộc A đang lừa mình hay C đang nói dối.

Đây là điểm khó của trường hợp 3 tướng 1 kẻ phản bội. Node trung thành không thiếu quy tắc vote; thứ thực sự thiếu là thông tin để phán đoán “ai đang nói dối”. Nếu xây dựng một tình huống đối xứng cho lieutenant trung thành còn lại, hai lieutenant trung thành sẽ bị đẩy đến những quyết định khác nhau, cuối cùng vi phạm tính nhất quán. Bài báo của Lamport từng nhắc rằng loại bài toán này rất dễ bị trực giác dẫn sai khi chứng minh; bài báo cuối cùng chứng minh bằng phép quy giản: chỉ sử dụng thông điệp truyền miệng thì nếu muốn chịu được `m` kẻ phản bội, cần ít nhất `3m + 1` vị tướng. Nói cách khác, số tướng trung thành phải lớn hơn `2/3` tổng số.

![Ba vị tướng không thể chịu được một kẻ phản bội](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/byzantine-generals-problem-three-general-impossibility.png)

Chịu được 1 kẻ phản bội cần ít nhất 4 vị tướng; chịu được 2 kẻ phản bội cần ít nhất 7 vị tướng.

## Nhất quán không có nghĩa là chắc chắn có thể tiếp tục chạy

Khi học consensus protocol, cần phân biệt hai tính chất:

- **Tính an toàn (Safety)**: không được quyết định hai kết quả xung đột với nhau.
- **Tính hoạt động (Liveness)**: hệ thống cuối cùng có thể tiếp tục tiến triển và đưa ra quyết định.

Nhiều protocol ưu tiên bảo vệ tính an toàn trong tình huống bất thường. Ví dụ, khi cluster Raft không lấy được majority, nó sẽ dừng commit log mới thay vì để hai network partition lần lượt commit các log xung đột với nhau. Việc dừng tiến triển ảnh hưởng đến availability, nhưng ít nhất không làm trạng thái bị ghi sai.

Trong hệ thống hoàn toàn asynchronous, độ trễ message không có upper bound. Chỉ dựa vào timeout thì không thể phán đoán nghiêm ngặt một node thực sự đã crash hay chỉ chậm hơn một chút. Kết quả FLP còn chỉ ra rằng trong mô hình hoàn toàn asynchronous, dù chỉ có một process có thể crash, deterministic consensus protocol vẫn tồn tại execution không thể termination.

Vì vậy, hệ thống thực tế thường đưa vào các giả định hoặc cơ chế bổ sung để khôi phục liveness, chẳng hạn eventual synchrony, randomization, failure detector, retry và view change. Khi nói PBFT có thể chạy trong network asynchronous như Internet, cũng cần hiểu theo hướng này: safety và liveness không phải cùng một cam kết.

![Safety và liveness trong consensus protocol](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/byzantine-generals-problem-safety-liveness.png)

## Thông điệp truyền miệng: vì sao cần 3m + 1?

Bài báo của Lamport trước hết thảo luận **Oral Messages**, thường được dịch là thông điệp truyền miệng.

Mô hình thông điệp truyền miệng có 3 tiền đề:

1. Message đã gửi sẽ được chuyển đến chính xác.
2. Receiver biết message do ai gửi.
3. Có thể phát hiện việc không nhận được message.

Những giả định này đã mạnh hơn network thực tế rất nhiều. Trong hệ thống thực tế, message có thể bị mất, độ trễ có thể không dự đoán được; việc phát hiện một node thực sự đã dừng hay chỉ chậm thường chỉ có thể được phán đoán gần đúng bằng timeout.

Ngay cả với các giả định mạnh này, nếu message không có signature thì kẻ phản bội vẫn có thể nói những điều khác nhau với những người khác nhau. Để bù đắp điều đó, protocol cần cho các lieutenant thuật lại cho nhau command mình nghe được, đồng thời trải các mâu thuẫn ra qua nhiều round message.

Thuật toán thông điệp truyền miệng thường được ký hiệu là `OM(m)`, trong đó `m` biểu thị số kẻ phản bội tối đa.

Khi `m = 0`, quy trình rất đơn giản: commander gửi command cho từng lieutenant, lieutenant thực hiện theo.

Khi `m > 0`, quy trình được mở rộng đệ quy:

1. Commander gửi command cho tất cả lieutenant.
2. Mỗi lieutenant chuyển tiếp command mình nhận được cho các lieutenant khác.
3. Nếu vẫn cần chịu được nhiều kẻ phản bội hơn, tiếp tục để node nhận được lời thuật lại chuyển tiếp ra ngoài.
4. Cuối cùng, mỗi lieutenant trung thành áp dụng cùng một hàm `majority` lên tập value nhận được; nếu không có majority, có thể dùng default value, trong bài báo default value là rút lui. Bài báo cũng đề cập rằng nếu value domain có thứ tự thì có thể lấy median. Điểm mấu chốt là mọi lieutenant trung thành đều dùng cùng một quy tắc deterministic.

![Nhiều round thuật lại trong mô hình thông điệp truyền miệng OM(m)](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/byzantine-generals-problem-oral-messages.png)

Có thể dùng `m = 1` để xem vì sao thuật toán này cần 4 vị tướng.

Giả sử A là commander, B, C, D là lieutenant và hệ thống có tối đa 1 kẻ phản bội. Nếu A là kẻ phản bội, hắn có thể gửi tấn công cho B, gửi rút lui cho C và D. Tiếp đó B, C, D sẽ thuật lại cho nhau command mình nhận được từ A:

- B nói với C, D: A bảo tôi tấn công.
- C nói với B, D: A bảo tôi rút lui.
- D nói với B, C: A bảo tôi rút lui.

Đối với B, những gì nhận được là “tấn công, rút lui, rút lui”, majority là rút lui. C và D cũng thấy majority giống như vậy. Nhờ đó, dù commander A hành động ác ý, các lieutenant trung thành vẫn có thể đạt đồng thuận.

Điểm mấu chốt ở đây không chỉ là “thiểu số phục tùng đa số”. Nếu commander trung thành, các lieutenant trung thành sẽ nhận và chuyển tiếp cùng một value; nhóm trung thành tạo thành majority, qua đó bảo đảm validity. Nếu commander là kẻ phản bội, tác dụng quan trọng hơn của protocol là giúp tất cả lieutenant trung thành nhận được cùng một value vector, sau đó áp dụng cùng một hàm `majority` lên value vector này, qua đó bảo đảm agreement.

Khi số node không đủ, thông tin cục bộ mà các node trung thành nhìn thấy không thể phân biệt các tình huống failure khác nhau; quy tắc vote có đẹp đến đâu cũng không có tác dụng.

Tuy nhiên, `OM(m)` giống một thuật toán lý thuyết giúp hiểu kết luận hơn, không phù hợp để đưa trực tiếp vào business system. Nó yêu cầu biết upper bound `m` của số kẻ phản bội, cần nhiều round message forwarding đệ quy, và lưu lượng communication tăng nhanh theo số node và số failure có thể chịu được. Hệ thống thực tế thường chuyển sang các protocol mang tính engineering hơn.

## Thông điệp có chữ ký: có signature thì bài toán sẽ đơn giản hơn sao?

Tiếp theo, bài báo thảo luận **Signed Messages**, tức thông điệp có chữ ký.

Thông điệp có chữ ký bổ sung hai khả năng trên nền tảng thông điệp truyền miệng:

- Signature của tướng trung thành không thể bị giả mạo; có thể phát hiện khi nội dung message bị thay đổi.
- Bất kỳ ai cũng có thể verify signature có thực sự đến từ vị tướng tương ứng hay không.

Sau khi có signature, kẻ phản bội vẫn có thể nói dối, nhưng rất khó nói dối thay node trung thành. Nếu B nhận được message “C nói A bảo mọi người rút lui”, B có thể kiểm tra message này có signature của C hay không, đồng thời kiểm tra command của A được thuật lại bên trong có signature của A hay không.

Điều này làm suy yếu khả năng rắc rối nhất của kẻ phản bội: bịa ra những phiên bản khác nhau cho những người khác nhau nhưng khiến người khác không thể truy nguyên.

Signature giải quyết việc xác thực nguồn và tính toàn vẹn của message, chứ không bảo đảm người ký trung thực. Commander phản bội vẫn có thể tự ký hai command xung đột với nhau; điểm khác biệt là hai command này đều để lại bằng chứng có thể verify, các node khác có thể tiếp tục chuyển tiếp bằng chứng đó, để mọi lieutenant trung thành cuối cùng nhìn thấy cùng một nhóm thông tin mâu thuẫn.

Trong mô hình thông điệp có chữ ký, Lamport đưa ra thuật toán `SM(m)`. Thuật toán này có thể thỏa mãn IC1 và IC2 khi tồn tại tối đa `m` kẻ phản bội, không còn cần `3m + 1` node như trong mô hình thông điệp truyền miệng. Bài báo gốc còn chỉ ra rằng nếu tổng số node nhỏ hơn `m + 2`, bài toán là tầm thường, vì khi đó hệ thống có thể không có nổi 2 tướng trung thành, nên không thể nói đến tính nhất quán có ý nghĩa giữa các node trung thành. Vì vậy, nên hiểu `n >= m + 2` là “tồn tại ít nhất hai node trung thành để bài toán có ý nghĩa thực tế”, không nên xem nó là một lower bound về khả năng chịu lỗi cùng loại với `3m + 1`.

Cuối cùng, `SM(m)` cũng không đơn giản là majority vote. Mỗi lieutenant trung thành duy trì một tập command `V` đã nhận, sau đó thực thi hàm `choice(V)` deterministic được thống nhất chung. Nếu commander phản bội lần lượt ký “tấn công” và “rút lui”, các lieutenant trung thành cuối cùng nhận được cùng một tập, rồi thực thi cùng một `choice` trên tập đó, nên kết quả tự nhiên nhất quán.

![Lan truyền thông tin trong mô hình thông điệp có chữ ký SM(m)](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/byzantine-generals-problem-signed-messages.png)

Điều này không có nghĩa mọi BFT system trong thực tế đều chỉ cần `m + 2` node. Phần này thảo luận một bài toán interactive consistency trong model cụ thể của bài báo Lamport. Hệ thống thực tế còn phải xem xét asynchronous network, performance, client request, state machine replication, view change, malicious client, replay attack và các vấn đề khác. Bài báo gốc cũng đề cập rằng nếu muốn thực thi `SM(m)` lặp đi lặp lại, cần gắn sequence number vào value để tránh signature message cũ bị replay. Các protocol thực dụng như PBFT thường vẫn dùng `3f + 1` replica để chịu được `f` Byzantine failure node.

Điểm này rất dễ nhầm: signature có thể khiến “ai đã nói gì” trở nên có thể verify, nhưng không loại bỏ các vấn đề về quorum intersection, message delay vô hạn và state machine replication.

## Byzantine failure khác ordinary failure như thế nào?

Trong backend engineering thường nói đến failure, nhưng failure có nhiều cấp độ khác nhau.

| Loại failure                                    | Biểu hiện điển hình                                                      | Mô tả                                                                           |
| ----------------------------------------------- | ------------------------------------------------------------------------ | ------------------------------------------------------------------------------- |
| Crash failure (Crash Fault)                     | Process dừng, machine crash                                              | Node không tiếp tục thực thi                                                    |
| Omission/Timing failure (Omission/Timing Fault) | Message mất, delay, network partition, response quá chậm                 | Node có thể vẫn còn hoạt động, nhưng communication không hoàn thành như dự kiến |
| Byzantine failure (Byzantine Fault)             | Gửi thông tin mâu thuẫn, tính toán sai, state corruption, hành động ác ý | Hành vi có thể tùy ý lệch khỏi protocol                                         |

![Crash failure, omission/timing failure và Byzantine failure](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/byzantine-generals-problem-fault-models.png)

Paxos, Raft và ZAB thường thuộc nhóm CFT (Crash Fault Tolerance, khả năng chịu crash failure). Chúng giả định node không cố ý hành động ác ý, nhiều nhất chỉ không phản hồi, phản hồi chậm, mất network hoặc crash. Lấy Raft làm ví dụ, phần giới thiệu chính thức của Raft nêu rằng cluster gồm 5 server có thể tiếp tục hoạt động khi 2 server failure; nếu failure nhiều hơn, hệ thống sẽ dừng tiến triển nhưng không trả về kết quả sai.

BFT (Byzantine Fault Tolerance, khả năng chịu Byzantine failure) xử lý mô hình failure mạnh hơn. Node có thể vẫn hoạt động và communication bình thường, nhưng nội dung nó gửi ra không đáng tin. PBFT là protocol state machine replication chịu Byzantine failure thực dụng kinh điển. Trong bài báo OSDI năm 1999 [Practical Byzantine Fault Tolerance](https://www.usenix.org/conference/osdi-99/practical-byzantine-fault-tolerance), Castro và Liskov đã triển khai một dịch vụ NFS chịu Byzantine failure; trong điều kiện bình thường, dịch vụ chỉ chậm hơn NFS tiêu chuẩn chưa replication 3%.

PBFT cho phép network message bị mất, delay, duplicate và out-of-order, đồng thời cho phép failure node tùy ý lệch khỏi protocol. Safety của nó không phụ thuộc vào synchronous assumption: ngay cả khi network không ổn định trong thời gian dài, các replica bình thường cũng không commit các operation xung đột với nhau. Tuy nhiên, liveness của nó phụ thuộc vào điều kiện eventual synchrony yếu hơn; node bình thường và message không được delay vô hạn.

Trong model tiêu chuẩn, PBFT dùng `3f + 1` replica để chịu được tối đa `f` Byzantine failure. Signature và MAC được dùng để authenticate message, chống giả mạo và replay; `3f + 1` dùng để bảo đảm giữa các quorum tồn tại đủ nhiều node bình thường giao nhau. Hai yếu tố này giải quyết những vấn đề khác nhau.

Các model phổ biến có thể được so sánh sơ lược như sau:

| Model                                   | Số failure có thể chịu được | Số replica tối thiểu phổ biến | Nguyên nhân chính                                                  |
| --------------------------------------- | --------------------------- | ----------------------------- | ------------------------------------------------------------------ |
| CFT                                     | `f` crash failure           | `2f + 1`                      | Các node còn lại vẫn phải tạo thành majority                       |
| State machine replication BFT kinh điển | `f` Byzantine failure       | `3f + 1`                      | Trong quorum `2f + 1` phải có ít nhất `f + 1` node bình thường     |
| Lamport signed messages model           | `m` kẻ phản bội             | Không còn yêu cầu `3m + 1`    | Signature khiến message mâu thuẫn có thể được verify và lan truyền |

![So sánh CFT, BFT và signed messages model](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/byzantine-generals-problem-cft-vs-bft.png)

Bảng này chỉ là tổng hợp các model phổ biến, không phải định luật chung mà mọi protocol đều vô điều kiện tuân theo. Trusted hardware, hybrid failure model, các network assumption khác nhau và security goal khác nhau đều có thể làm thay đổi yêu cầu về số replica.

Phần lớn ordinary Java backend system không cần BFT, vì các service node thường thuộc cùng một tổ chức, cùng một permission system và cùng một hệ thống vận hành; node mặc định được tin cậy. Vấn đề chính cần giải quyết là high availability, master-slave failover, log consistency và tránh split-brain, chứ không phải ngăn node của mình chủ động nói dối các node khác.

Nhưng trong những trường hợp sau, không thể dễ dàng bỏ qua Byzantine failure:

- Public blockchain, consortium blockchain và hệ thống clearing and settlement xuyên tổ chức.
- Nhiều bên cùng duy trì ledger hoặc state nhưng không hoàn toàn trust lẫn nhau.
- Cần chịu được việc node sau khi bị xâm nhập vẫn tiếp tục gửi message có format hợp lệ nhưng sai.
- Hệ thống state machine replication có security level cao.

Một số hệ thống nói rằng mình đã dùng “consensus”, nhưng failure assumption phía sau consensus algorithm khác nhau rất nhiều. Chỉ nói “đã dùng Raft” không thể chứng minh hệ thống chống được malicious node; chỉ nói “đã dùng signature” cũng không thể chứng minh hệ thống đã triển khai đầy đủ Byzantine fault tolerance.

## Bài toán các vị tướng Byzantine liên quan gì đến blockchain?

Nhiều người lần đầu nghe đến Bài toán các vị tướng Byzantine là qua các bài viết về blockchain.

Mối liên hệ này là đúng, nhưng cũng đừng đánh đồng hai khái niệm. Bài toán các vị tướng Byzantine là một bài toán distributed consensus nền tảng hơn; blockchain chỉ là một nhóm application scenario trong đó. Node trong public blockchain network đến từ các chủ thể khác nhau, node có thể hành động ác ý và network cũng có thể delay rất lớn, nên bản chất nó phải đối mặt với Byzantine failure.

Tuy nhiên, PoW, PoS và các protocol BFT trong blockchain không nằm cùng layer với thông điệp truyền miệng và thông điệp có chữ ký trong bài báo Lamport:

- Bài báo Lamport đưa ra formal problem và các solution ban đầu, tập trung vào cách node trung thành đạt đồng thuận khi có kẻ phản bội.
- Các protocol như PBFT, HotStuff và Tendermint gần với engineering implementation hơn, tập trung vào replica replication, voting phase, view change và commit rule.
- PoW và PoS còn đưa vào các thiết kế như economic cost, stake, probabilistic confirmation, longest chain hoặc finality.

Threshold trong blockchain cũng không nhất thiết được tính theo số node. PoW thường quan tâm đến tỷ lệ computational power mà attacker nắm giữ; PoS và các protocol kiểu Tendermint thường quan tâm đến stake hoặc voting weight mà malicious validator nắm giữ. Dù attacker chỉ kiểm soát một số ít node, nếu kiểm soát weight đủ lớn thì vẫn có thể vượt qua security threshold của protocol.

![Threshold trong blockchain](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/byzantine-generals-problem-blockchain-weight-thresholds.png)

Ngoài ra, PoW và PoS giống cơ chế chống Sybil, lựa chọn leader và phân bổ weight hơn. Một system hoàn chỉnh còn cần block proposal, fork choice, vote hoặc finality rule; không thể đánh đồng riêng chúng với complete consensus protocol.

Vì vậy, trong learning path, Bài toán các vị tướng Byzantine phù hợp hơn khi được dùng làm kiến thức nền trước khi học consensus algorithm. Trước hết hãy hiểu bài toán này, sau đó học Paxos, Raft, ZAB, PBFT và blockchain consensus; nhiều khái niệm sẽ trở nên dễ tiếp cận hơn.

## Trả lời thế nào khi phỏng vấn?

Nếu interviewer hỏi “Bài toán các vị tướng Byzantine là gì?”, có thể trả lời theo thứ tự sau:

1. Nói về bài toán trước: trong distributed system, một số node có thể failure hoặc hành động ác ý, nhưng các node bình thường vẫn phải đạt đồng thuận về một value.
2. Nói về điểm khó: Byzantine node có thể gửi thông tin mâu thuẫn cho các node khác nhau, khiến node bình thường rất khó phán đoán ai đang nói dối.
3. Bổ sung một kết luận quan trọng: trong model chỉ sử dụng thông điệp truyền miệng, để chịu được `m` kẻ phản bội cần ít nhất `3m + 1` node; 3 node không thể chịu được 1 kẻ phản bội.
4. Nêu hướng giải quyết: bài báo Lamport đưa ra hai nhóm solution là thông điệp truyền miệng và thông điệp có chữ ký; trong hệ thống thực tế, algorithm CFT phổ biến có Paxos, Raft và ZAB, algorithm BFT phổ biến có PBFT, HotStuff, Tendermint.
5. Cuối cùng liên hệ với engineering: phần lớn ordinary backend system dùng CFT vì node mặc định được tin cậy; các scenario xuyên tổ chức, blockchain và nhạy cảm về security mới cần cân nhắc BFT nhiều hơn.

Có thể trả lời tương đối đầy đủ như sau:

> Bài toán các vị tướng Byzantine mô tả cách distributed system đạt consensus khi tồn tại malicious node hoặc abnormal node. Nó khó hơn crash failure thông thường vì Byzantine node có thể gửi các message khác nhau cho những node khác nhau, phá vỡ phán đoán nhất quán giữa các node bình thường. Bài báo Lamport chứng minh rằng trong mô hình thông điệp truyền miệng, nếu muốn chịu được `m` kẻ phản bội thì cần ít nhất `3m + 1` node; mô hình thông điệp có chữ ký không còn yêu cầu `3m + 1` vì command mâu thuẫn có thể được verify và lan truyền. Paxos, Raft và ZAB chủ yếu xử lý crash failure, mặc định node không cố ý nói dối; các algorithm như PBFT xử lý Byzantine failure, phù hợp với scenario các node không hoàn toàn trust lẫn nhau, đồng thời thường cần `3f + 1` replica để chịu được `f` Byzantine failure.

Như vậy đã đủ để xử lý phần lớn câu hỏi phỏng vấn backend. Trừ khi vị trí công việc liên quan rõ ràng đến blockchain, database phân tán hoặc triển khai consensus protocol, không cần suy diễn đầy đủ chứng minh đệ quy của `OM(m)` ngay tại chỗ.

## Tài liệu tham khảo

- [The Byzantine Generals Problem - Microsoft Research](https://www.microsoft.com/en-us/research/publication/byzantine-generals-problem/)
- [The Byzantine Generals Problem PDF](https://lamport.azurewebsites.net/pubs/byz.pdf)
- [Leslie Barry Lamport - A.M. Turing Award Laureate](https://amturing.acm.org/award_winners/lamport_1205376.cfm)
- [Raft Consensus Algorithm - Official Introduction](https://raft.github.io/)
- [Practical Byzantine Fault Tolerance - USENIX](https://www.usenix.org/conference/osdi-99/practical-byzantine-fault-tolerance)
- [Impossibility of Distributed Consensus with One Faulty Process](https://groups.csail.mit.edu/tds/papers/Lynch/jacm85.pdf)
- [HotStuff: BFT Consensus in the Lens of Blockchain](https://arxiv.org/abs/1803.05069)
- [What is Tendermint](https://docs.tendermint.com/master/introduction/what-is-tendermint.html)
- [About the Byzantine Generals Problem](https://justinzhangonline.wordpress.com/2010/01/13/%E6%9C%89%E5%85%B3%E6%8B%9C%E5%8D%A0%E5%BA%AD%E5%B0%86%E5%86%9B%E9%97%AE%E9%A2%98/)
- [Understanding the Byzantine Generals Problem OM Version with Text and Illustrations](https://marslenjoy.medium.com/%E5%9B%BE%E7%A0%81%E5%B9%B6%E8%8C%82%E4%B8%80%E6%96%87%E7%9C%8B%E6%87%82%E6%8B%9C%E5%8D%A0%E5%BA%AD%E5%B0%86%E5%86%9B%E9%97%AE%E9%A2%98om%E7%89%88-49e2dcbb629c)
- [The Byzantine Generals Problem - TheByte](https://www.thebyte.com.cn/consensus/The-Byzantine-General-Problem.html)

<!-- @include: @article-footer.snippet.md -->
