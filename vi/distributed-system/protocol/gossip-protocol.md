---
title: "Giải thích chi tiết Gossip Protocol: Anti-Entropy, Rumor Mongering, SWIM và Eventual Consistency"
category: Distributed
description: "Giải thích chi tiết Gossip Protocol, giới thiệu mô hình truyền thông tin phi tập trung, Anti-Entropy, Rumor Mongering, mô hình Push/Pull, SWIM Protocol, Eventual Consistency và ứng dụng trong các hệ thống như Redis Cluster, Cassandra."
tag:
  - Distributed Protocols and Algorithms
  - Data Replication Protocols
  - Eventual Consistency
head:
  - - meta
    - name: keywords
      content: Gossip Protocol,SWIM Protocol,Anti-Entropy,Rumor Mongering,Eventual Consistency,phi tập trung,Redis Cluster,Cassandra,Distributed Protocols,Distributed Algorithms
---

## Bối cảnh

Trong hệ thống phân tán, chia sẻ state giữa các node là một nhu cầu cơ bản.

Một cách đơn giản là **broadcast tập trung**: node trung tâm đồng bộ thông tin tới tất cả node khác. Cách này phù hợp với hệ thống tập trung, nhưng có nhược điểm rõ ràng: khi số lượng node tăng, hiệu quả đồng bộ giảm (độ phức tạp O(N)), đồng thời quá phụ thuộc vào node trung tâm và có rủi ro single point of failure.

**Truyền thông tin phân tán** bằng **Gossip Protocol** cung cấp một phương án thay thế phi tập trung.

Nếu bạn chưa rõ Leader/Quorum và Gossip lần lượt phù hợp để giải quyết vấn đề nào, hãy đọc trước bài [Giải thích chi tiết Distributed Coordination](./centralized-and-decentralized.md). Bài Gossip này chỉ tập trung vào việc “state được truyền đi và hội tụ như thế nào”, không giải thích các vấn đề cần coordination mạnh như bầu chọn Leader, split-brain và Fencing Token.

![Cơ chế giao tiếp trong hệ thống phân tán: tập trung và phi tập trung](https://oss.javaguide.cn/github/javaguide/distributed-system/protocol/gossip-centralized-vs-decentralized.png)

## Giới thiệu Gossip Protocol

**Gossip** còn được gọi là **Epidemic Protocol**, lấy cảm hứng từ tính ngẫu nhiên của quá trình lây lan dịch bệnh. Ý tưởng cốt lõi là: mỗi node định kỳ chọn ngẫu nhiên một số node khác để trao đổi thông tin, khiến dữ liệu lan ra toàn bộ network như virus.

![Bản dịch Gossip](https://oss.javaguide.cn/github/javaguide/distributed-system/protocol/gossip.png)

Gossip Protocol lần đầu được Demers và cộng sự đề xuất trong bài báo năm 1987 [《Epidemic Algorithms for Replicated Database Maintenance》](https://dl.acm.org/doi/10.1145/41840.41841), nhằm giải quyết vấn đề đồng bộ replica của distributed database.

**Định nghĩa**: Gossip Protocol là một communication protocol **phi tập trung**, thông qua trao đổi thông tin ngẫu nhiên giữa các node, giúp state của tất cả node trong cluster đạt **Eventual Consistency**, với các điều kiện: môi trường không có Byzantine fault, không tồn tại network partition vĩnh viễn và các node liên tục trao đổi theo chu kỳ.

> **Phân biệt quan trọng**: Gossip là protocol truyền thông tin, **không phải consensus algorithm** (như Raft/Paxos). Consensus algorithm bảo đảm strong consistency và safety; Gossip chỉ bảo đảm Eventual Consistency, không phù hợp cho leader election hoặc state machine replication, là các trường hợp cần strong consistency.

**Đặc điểm chính**:

- **Phi tập trung**: không có node trung tâm, mọi node có địa vị ngang nhau
- **Fault tolerance cao**: chịu được node down, network partition và việc thêm/xóa node động
- **Hội tụ theo xác suất**: trong mô hình kinh điển với việc chọn node ngẫu nhiên đồng đều và fanout là hằng số, số round truyền kỳ vọng là O(log N) (ví dụ N=100 thì khoảng 5-7 round, phụ thuộc cụ thể vào fanout và packet loss rate)
- **Dư thừa message**: cùng một message có thể được nhận nhiều lần, cần cơ chế deduplication

## Ứng dụng của Gossip Protocol

Gossip Protocol được ứng dụng rộng rãi trong distributed system:

- **Redis Cluster**: dùng để đồng bộ state giữa các node và phát hiện failure
- **Apache Cassandra**: dùng để truyền thông tin về membership và state của node; replica repair dùng Anti-Entropy/repair (dựa trên Merkle Tree)
- **Consul**: dùng cho service discovery, failure detection và event broadcast (dựa trên SWIM Protocol)
- **Amazon Dynamo**: dùng cho Eventual Consistency của distributed storage

Lấy **Redis Cluster** (3.0+) làm ví dụ:

Redis Cluster là một giải pháp distributed cache phi tập trung. Các node trao đổi cluster state thông qua Gossip Protocol, bao gồm: thông tin node, slot allocation và node state (online/PFAIL/FAIL).

![Giải pháp cluster chính thức của Redis](https://oss.javaguide.cn/github/javaguide/distributed-system/protocol/up-fcacc1eefca6e51354a5f1fc9f2919f51ec.png)

**Các loại Gossip message**:

| Loại message | Công dụng                                     |
| ------------ | --------------------------------------------- |
| MEET         | Thêm node được chỉ định vào cluster           |
| PING         | Gửi định kỳ, trao đổi node state              |
| PONG         | Phản hồi PING, mang theo state của chính node |
| FAIL         | Broadcast failure marker của node             |

> Lưu ý: về mặt implementation, MEET/PING/PONG dùng chung một message structure; PONG là response của PING/MEET, còn MEET tương đương một PING “forced handshake”.

**Quy trình failure detection**:

1. Nếu node A không nhận được response từ B trong `cluster-node-timeout` (thường là 15s, giá trị cụ thể tùy configuration), A đánh dấu B là **PFAIL** (nghi ngờ down)
2. Nếu A nhận được báo cáo PFAIL của B từ các master node khác, và **hơn một nửa số master node** xác nhận B là PFAIL (báo cáo chưa hết hạn), A đánh dấu B là **FAIL** (đã down) rồi broadcast tới cluster

Hình dưới đây minh họa Redis Cluster có kiến trúc master-replica. Các đường nét đứt biểu thị việc các node giao tiếp bằng Gossip, còn đường liền biểu thị replication giữa master và replica.

![Các node trong Redis Cluster giao tiếp bằng Gossip](https://oss.javaguide.cn/github/javaguide/distributed-system/protocol/redis-cluster-gossip.png)

> Lưu ý: Redis Cluster chủ yếu truyền node/slot/failure information bằng incremental gossip qua PING/PONG (kèm timestamp/flag...), chứ không dùng quy trình đối soát Anti-Entropy dựa trên Merkle tree như Dynamo.

Để xem giới thiệu chi tiết về Redis Cluster, bạn có thể đọc bài [Giải thích chi tiết Redis Cluster (trả phí)](https://javaguide.cn/database/redis/redis-cluster.html).

## Mô hình truyền của Gossip Protocol

Gossip Protocol có hai mô hình truyền chính: **Anti-Entropy** và **Rumor Mongering**.

### Anti-Entropy

**Định nghĩa**: các node trao đổi **toàn bộ dữ liệu** (hoặc data summary), loại bỏ khác biệt để đạt Eventual Consistency.

Ý nghĩa vật lý của **Entropy** là mức độ hỗn loạn của hệ thống; Anti-Entropy nghĩa là **giảm khác biệt dữ liệu giữa các node và nâng cao consistency**.

Theo Wikipedia:

> Khái niệm Entropy bắt nguồn từ [vật lý học](https://zh.wikipedia.org/wiki/物理学), dùng để đo mức độ hỗn loạn của một hệ nhiệt động lực học. Entropy nên được hiểu là thước đo của tính không chắc chắn thay vì tính chắc chắn, vì nguồn tin càng ngẫu nhiên thì Entropy càng lớn.

Ở đây, bạn có thể hiểu Entropy trong Anti-Entropy là mức độ hỗn loạn/khác biệt của dữ liệu giữa các node. Anti-Entropy nghĩa là loại bỏ khác biệt dữ liệu giữa các node, tăng mức độ tương đồng giữa chúng và từ đó giảm giá trị Entropy.

**Ba cách implementation**:

| Cách thức | Mô tả                                             | Trường hợp sử dụng               |
| --------- | ------------------------------------------------- | -------------------------------- |
| Push      | Sender push toàn bộ dữ liệu của mình cho receiver | Sender có data mới               |
| Pull      | Receiver pull toàn bộ dữ liệu từ sender           | Data của receiver đã cũ          |
| Push-Pull | Hai chiều trao đổi dữ liệu và so sánh khác biệt   | Hiệu quả cao nhất, phổ biến nhất |

![Cơ chế Anti-Entropy: sequence diagram tương tác Push-Pull (Anti-Entropy)](https://oss.javaguide.cn/github/javaguide/distributed-system/protocol/gossip-anti-entropy-pushpull.png)

Pseudo-code như sau:

![Pseudo-code Anti-Entropy](https://oss.javaguide.cn/github/javaguide/distributed-system/protocol/up-df16e98bf71e872a7e1f01ca31cee93d77b.png)

**Đặc điểm hội tụ**: trong mô hình chọn node ngẫu nhiên đồng đều với fanout là hằng số, kỳ vọng sau O(log N) round sẽ bao phủ toàn bộ node (ước tính phổ biến có thể dùng cỡ log₂N).

Một số hệ thống (như InfluxDB) dùng **deterministic closed-loop scheduling** (chẳng hạn ring topology) thay cho việc chọn ngẫu nhiên, có thể hoàn tất đồng bộ trong số round xác định. Đây là một **implementation phái sinh về mặt engineering** của Anti-Entropy, không phải cơ chế cốt lõi của Gossip Protocol tiêu chuẩn. Deterministic scheduling hy sinh ưu thế fault tolerance của tính ngẫu nhiên để đổi lấy thời gian hội tụ có thể dự đoán.

![Deterministic closed-loop scheduling](https://oss.javaguide.cn/github/javaguide/distributed-system/protocol/raft-anti-entropyclosed-loop.png)

1. Node A push dữ liệu cho node B, node B nhận được dữ liệu mới nhất từ node A.
2. Node B push dữ liệu cho C, node C nhận được dữ liệu mới nhất từ node A và B.
3. Node C push dữ liệu cho A, node A nhận được dữ liệu mới nhất từ node B và C.
4. Node A lại push dữ liệu cho B để tạo thành closed loop, nhờ đó node B nhận được dữ liệu mới nhất từ node C.

**Trade-off**: closed-loop scheduling có thể hoàn tất đồng bộ trong thời gian xác định, nhưng hy sinh **fault tolerance** (failure của node trong ring ảnh hưởng đến propagation path), đồng thời khó thích ứng với việc thêm/xóa node động.

**Trường hợp sử dụng**: cần residual rate thấp (cố gắng không bỏ sót update), cho phép đối soát và repair định kỳ ở background; khi data volume lớn, bắt buộc phải dựa vào summary/tree và các phương pháp so sánh incremental để kiểm soát cost.

> **Production optimization**: trong distributed storage quy mô lớn (như Cassandra, DynamoDB), data volume của node có thể đạt mức TB, việc trực tiếp trao đổi toàn bộ dữ liệu là không thực tế. Production system dùng **Merkle Tree** để so sánh khác biệt incremental: trước tiên hai node trao đổi root hash của Merkle Tree; nếu có khác biệt thì đệ quy so sánh subtree, định vị khác biệt ở các level có tree height O(log M) (M là số entry trong phạm vi đó), sau đó chỉ truyền incremental data.

### Rumor Mongering

**Định nghĩa**: khi node có **data mới**, node trở thành active node, định kỳ broadcast data đó tới các node ngẫu nhiên cho đến khi tất cả node đều nhận được.

**Khác biệt so với Anti-Entropy**:

- Chỉ truyền **data mới** (Delta), không truyền toàn bộ data
- Sau khi nhận update, node chuyển sang trạng thái active và truyền định kỳ; sau khi nhiều lần tiếp xúc với các node đã biết update, node dừng truyền theo policy (count/probability/TTL)
- Phù hợp với trường hợp **số lượng node lớn**, **incremental data nhỏ**

> **Cơ chế deduplication**: trong production environment (như Redis Cluster), dùng **version number** hoặc **message ID** để deduplicate, tránh xử lý lặp cùng một message.

Hình dưới đây (lấy từ bài viết [INTRODUCTION TO GOSSIP](https://managementfromscratch.wordpress.com/2016/04/01/introduction-to-gossip/)) minh họa:

![Minh họa quá trình truyền Gossip](https://oss.javaguide.cn/github/javaguide/distributed-system/protocol/gossip-rumor-mongering.gif)

Pseudo-code như sau:

![](https://oss.javaguide.cn/github/javaguide/csdn/20210605170707933.png)

**Đặc điểm hội tụ**: trong mô hình chọn node ngẫu nhiên đồng đều với fanout là hằng số, sau O(log N) round sẽ bao phủ toàn bộ node với xác suất cao.

**Lưu ý**:

- Kiểm soát packet size, cố gắng tránh fragmentation (tùy path MTU, thường kiểm soát trong một network packet)
- Kết hợp cơ chế deduplication (như message ID, version number)
- Tránh message storm do update tần suất cao
- Dùng **Jitter (random jitter)** để phân tán thời điểm đồng bộ, tránh việc nhiều node đồng thời bắt đầu truyền gây ra avalanche

![Gossip Protocol: quá trình truyền ngẫu nhiên và hội tụ](https://oss.javaguide.cn/github/javaguide/distributed-system/protocol/gossip-propagation.png)

### Tổng kết

| Điểm chính         | Anti-Entropy                        | Rumor Mongering                                  |
| ------------------ | ----------------------------------- | ------------------------------------------------ |
| Nội dung truyền    | Toàn bộ data (hoặc summary)         | Chỉ data mới (Delta)                             |
| Trường hợp sử dụng | Số lượng node vừa phải              | Số lượng node lớn/thay đổi động                  |
| Message overhead   | Lớn                                 | Nhỏ                                              |
| Phạm vi hội tụ     | Hội tụ về data mới nhất (full sync) | Hội tụ về data đã biết (incremental propagation) |

## Ưu điểm và nhược điểm của Gossip Protocol

**Ưu điểm**:

1. **Implementation đơn giản**: logic của protocol đơn giản, dễ hiểu

2. **Fault tolerance cao**: chịu được node down, network partition và việc thêm/xóa node động. Trong điều kiện lý tưởng, node mới hoặc node restart cuối cùng chắc chắn sẽ đạt state nhất quán với các node khác.

3. **Khả năng scale tốt**: thời gian hội tụ là O(log N). Khi N lớn (chẳng hạn N > 100), propagation song song thường nhanh hơn unicast từ node trung tâm (cần O(N) round ở vế sau). Trong mô hình rumor spreading điển hình, cost là **tổng số message O(N log N)** (phụ thuộc cụ thể vào strategy và stopping condition), nên tồn tại overhead dư thừa.

**Nhược điểm**:

1. **Eventual Consistency**: message cần nhiều round truyền để bao phủ toàn bộ network, tồn tại khoảng thời gian không nhất quán. Thời gian cụ thể để đạt consistency phụ thuộc vào network condition, gossip interval (**tùy implementation configuration, thường là 100ms-1s**) và quy mô node.

2. **Không phù hợp với Byzantine environment**: thiết kế của Gossip Protocol giả định môi trường không có Byzantine fault, không xử lý node độc hại (node không giả mạo hoặc sửa đổi message).

3. **Message redundancy**: do tính ngẫu nhiên của quá trình truyền, cùng một node có thể nhận lặp cùng một message, cần kết hợp cơ chế deduplication.

## Tổng kết

- Gossip Protocol là một communication protocol **phi tập trung**, thông qua trao đổi thông tin ngẫu nhiên giữa các node để state của tất cả node trong cluster đạt **Eventual Consistency**
- **Không phải consensus algorithm**: Gossip không bảo đảm strong consistency/linearizability, không thể dùng cho leader election hoặc state machine replication; chỉ consensus algorithm (Raft/Paxos) mới bảo đảm safety và linearizability
- Đặc điểm cốt lõi: phi tập trung, fault tolerance cao, hội tụ O(log N)
- Hai mô hình truyền: **Anti-Entropy** (full data/summary), **Rumor Mongering** (incremental data)
- Ứng dụng điển hình: truyền metadata (Redis Cluster), storage với Eventual Consistency (Cassandra/DynamoDB)
- Trade-off: tính đơn giản và fault tolerance vs độ trễ Eventual Consistency và message redundancy

## Tài liệu tham khảo

- [Epidemic Algorithms for Replicated Database Maintenance](https://dl.acm.org/doi/10.1145/41840.41841) - Demers et al., 1987
- [Amazon Dynamo: All Things Distributed](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) - DeCandia et al., 2007
- [Redis Cluster Specification](https://redis.io/docs/management/scaling/)
- Giải thích chi tiết Gossip Protocol của Redis Cluster: <https://segmentfault.com/a/1190000038373546>
- 《Distributed Protocols and Algorithms in Practice》
- 《Redis Design and Implementation》

<!-- @include: @article-footer.snippet.md -->
