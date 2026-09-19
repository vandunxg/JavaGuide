---
title: Tổng hợp câu hỏi phỏng vấn NoSQL cơ bản thường gặp
description: Tổng hợp câu hỏi phỏng vấn và kiến thức cơ bản về cơ sở dữ liệu NoSQL, bao gồm sự khác biệt giữa NoSQL và SQL, ưu điểm của NoSQL, bốn loại cơ sở dữ liệu NoSQL (key-value, document, graph, wide-column) cùng các sản phẩm tiêu biểu như Redis, MongoDB, Neo4j và trường hợp sử dụng của chúng.
category: Database
tag:
  - NoSQL
  - MongoDB
  - Redis
head:
  - - meta
    - name: keywords
      content: NoSQL,Redis,MongoDB,HBase,Cassandra,cơ sở dữ liệu key-value,cơ sở dữ liệu document,cơ sở dữ liệu graph,lưu trữ wide-column,sự khác biệt giữa SQL và NoSQL
---

## NoSQL là gì?

NoSQL (viết tắt của Not Only SQL) là tên gọi chung cho các cơ sở dữ liệu non-relational, chủ yếu hướng đến việc lưu trữ dữ liệu dạng key-value, document và graph. Cơ sở dữ liệu NoSQL vốn hỗ trợ các đặc tính như mô hình distributed, data redundancy và data sharding, nhằm cung cấp giải pháp lưu trữ dữ liệu có khả năng mở rộng, tính sẵn sàng cao và hiệu năng cao.

Một hiểu lầm phổ biến là cơ sở dữ liệu NoSQL hay cơ sở dữ liệu non-relational không thể lưu trữ tốt dữ liệu relational. Cơ sở dữ liệu NoSQL có thể lưu trữ dữ liệu relational, chỉ là cách lưu trữ của chúng khác với cơ sở dữ liệu relational.

Các đại diện của cơ sở dữ liệu NoSQL: HBase, Cassandra, MongoDB, Redis.

![](https://oss.javaguide.cn/github/javaguide/database/mongodb/sql-nosql-tushi.png)

## SQL và NoSQL khác nhau như thế nào?

|                         | Cơ sở dữ liệu SQL                                                                                                          | Cơ sở dữ liệu NoSQL                                                                                                                                                                                                                        |
| :---------------------- | -------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Mô hình lưu trữ dữ liệu | Lưu trữ có cấu trúc, dạng bảng với hàng và cột cố định                                                                     | Lưu trữ phi cấu trúc. Document: JSON document, key-value: cặp key-value, wide-column: bảng chứa hàng và cột động, graph: node và edge                                                                                                      |
| Lịch sử phát triển      | Phát triển vào những năm 1970, tập trung giảm dữ liệu trùng lặp                                                            | Phát triển vào cuối những năm 2000, tập trung nâng cao khả năng mở rộng và giảm chi phí lưu trữ dữ liệu quy mô lớn                                                                                                                         |
| Ví dụ                   | Oracle, MySQL, Microsoft SQL Server, PostgreSQL                                                                            | Document: MongoDB, CouchDB, key-value: Redis, DynamoDB, wide-column: Cassandra, HBase, graph: Neo4j, Amazon Neptune, Giraph                                                                                                                |
| Thuộc tính ACID         | Cung cấp các thuộc tính atomicity, consistency, isolation và durability (ACID)                                             | Thường không hỗ trợ ACID transaction; để có khả năng mở rộng và hiệu năng cao, chúng phải đánh đổi ở một mức độ nhất định. Một số ít hỗ trợ, chẳng hạn MongoDB. Tuy nhiên, hỗ trợ ACID transaction của MongoDB vẫn khác với MySQL.         |
| Hiệu năng               | Hiệu năng thường phụ thuộc vào disk subsystem. Để đạt hiệu năng tốt nhất, thường cần tối ưu query, index và cấu trúc bảng. | Hiệu năng thường do quy mô của hardware cluster bên dưới, network latency và ứng dụng gọi đến quyết định.                                                                                                                                  |
| Mở rộng                 | Vertical scaling (mở rộng bằng server mạnh hơn), read-write splitting, database sharding và table sharding                 | Horizontal scaling (mở rộng theo chiều ngang bằng cách tăng số server, thường dựa trên cơ chế sharding)                                                                                                                                    |
| Công dụng               | Lưu trữ dữ liệu cho các dự án enterprise thông thường                                                                      | Phạm vi sử dụng rộng; chẳng hạn graph database hỗ trợ phân tích và duyệt quan hệ giữa các dữ liệu liên kết với nhau, còn key-value database có thể xử lý việc mở rộng dữ liệu ở quy mô lớn và các trạng thái thay đổi với tần suất rất cao |
| Cú pháp query           | Structured Query Language (SQL)                                                                                            | Cú pháp truy cập dữ liệu có thể khác nhau tùy cơ sở dữ liệu                                                                                                                                                                                |

## Cơ sở dữ liệu NoSQL có những ưu điểm gì?

Cơ sở dữ liệu NoSQL rất phù hợp với nhiều ứng dụng hiện đại như ứng dụng mobile, Web và game. Những ứng dụng này cần cơ sở dữ liệu linh hoạt, có khả năng mở rộng, hiệu năng cao và nhiều tính năng để mang lại trải nghiệm người dùng tốt.

- **Tính linh hoạt:** Cơ sở dữ liệu NoSQL thường cung cấp schema linh hoạt, giúp phát triển theo vòng lặp nhanh hơn. Mô hình dữ liệu linh hoạt khiến cơ sở dữ liệu NoSQL trở thành lựa chọn lý tưởng cho dữ liệu semi-structured và unstructured.
- **Khả năng mở rộng:** Cơ sở dữ liệu NoSQL thường được thiết kế để horizontal scaling bằng cách sử dụng hardware cluster phân tán, thay vì vertical scaling bằng cách bổ sung server đắt tiền và mạnh hơn.
- **Hiệu năng cao:** Cơ sở dữ liệu NoSQL được tối ưu cho các mô hình dữ liệu và access pattern cụ thể, nhờ đó có thể đạt hiệu năng cao hơn so với việc cố gắng dùng cơ sở dữ liệu relational để thực hiện chức năng tương tự.
- **Tính năng mạnh mẽ:** Cơ sở dữ liệu NoSQL cung cấp API và các data type mạnh mẽ, được xây dựng chuyên biệt cho các mô hình dữ liệu tương ứng.

## Cơ sở dữ liệu NoSQL có những loại nào?

Cơ sở dữ liệu NoSQL chủ yếu có thể chia thành bốn loại sau:

- **Key-value:** Cơ sở dữ liệu key-value là một loại cơ sở dữ liệu tương đối đơn giản, trong đó mỗi item đều chứa key và value. Đây là loại cơ sở dữ liệu NoSQL linh hoạt nhất, vì ứng dụng có toàn quyền kiểm soát nội dung được lưu trong trường value mà không có bất kỳ hạn chế nào. Redis và DynanoDB là hai cơ sở dữ liệu key-value rất phổ biến.
- **Document:** Dữ liệu trong cơ sở dữ liệu document được lưu trong các document tương tự object dạng JSON (JavaScript Object Notation), rất rõ ràng và trực quan. Mỗi document chứa các cặp field và value. Các value có thể thuộc nhiều type khác nhau, bao gồm string, number, boolean, array hoặc object, và cấu trúc của chúng thường phù hợp với object mà developer sử dụng trong code. MongoDB là một cơ sở dữ liệu document rất phổ biến.
- **Graph:** Cơ sở dữ liệu graph được thiết kế để dễ dàng xây dựng và chạy các ứng dụng làm việc trên các tập dữ liệu có mức độ liên kết cao. Các trường hợp sử dụng điển hình của cơ sở dữ liệu graph gồm social network, recommendation engine, fraud detection và knowledge graph. Neo4j và Giraph là hai cơ sở dữ liệu graph rất phổ biến.
- **Wide-column:** Cơ sở dữ liệu lưu trữ wide-column rất phù hợp với các ứng dụng cần lưu trữ lượng dữ liệu lớn. Cassandra và HBase là hai cơ sở dữ liệu lưu trữ wide-column rất phổ biến.

Hình ảnh dưới đây lấy từ [tài liệu chính thức của Microsoft | Dữ liệu relational và dữ liệu NoSQL](https://learn.microsoft.com/en-us/dotnet/architecture/cloud-native/relational-vs-nosql-data).

![Mô hình dữ liệu NoSQL](https://oss.javaguide.cn/github/javaguide/database/mongodb/types-of-nosql-datastores.png)

## Tham khảo

- NoSQL là gì? - Tài liệu chính thức MongoDB: <https://www.mongodb.com/zh-cn/nosql-explained>
- NoSQL là gì? - AWS: <https://aws.amazon.com/cn/nosql/>
- NoSQL vs. SQL Databases - Tài liệu chính thức MongoDB: <https://www.mongodb.com/zh-cn/nosql-explained/nosql-vs-sql>

<!-- @include: @article-footer.snippet.md -->
