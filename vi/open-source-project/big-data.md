---
title: Các dự án Big Data mã nguồn mở chất lượng cao cho Java
description: Đề xuất các dự án Big Data mã nguồn mở chất lượng cao cho Java, bao gồm các dự án về tính toán, lưu trữ, tích hợp và lakehouse như Hadoop, Spark, Flink, Beam, SeaTunnel, Iceberg, Hudi, Paimon.
category: Dự án mã nguồn mở
icon: "mdi:database-search-outline"
---

Các dự án Big Data trải rộng trên các lĩnh vực tính toán, lưu trữ, tích hợp dữ liệu và lakehouse, không thể chỉ so sánh chúng theo chiều ngang dựa trên độ phổ biến. Trước tiên hãy xác định vấn đề cần giải quyết, sau đó chọn dự án ở tầng tương ứng.

## Distributed computing

- [Apache Hadoop](https://github.com/apache/hadoop): Dự án nền tảng của hệ sinh thái lưu trữ phân tán và batch processing, bao gồm các module cốt lõi như HDFS, MapReduce và YARN.
- [Apache Spark](https://github.com/apache/spark): Unified analytics engine dùng để xử lý dữ liệu quy mô lớn, bao phủ các trường hợp sử dụng như batch processing, stream processing, SQL và machine learning.
- [Apache Flink](https://github.com/apache/flink): Stateful computing framework hướng đến bounded và unbounded data stream, thường được dùng cho xử lý dữ liệu real-time và các tác vụ stream-batch thống nhất.
- [Apache Beam](https://github.com/apache/beam): Programming model dùng để mô tả thống nhất các tác vụ batch processing và stream processing, có thể chạy cùng một Pipeline trên các engine như Flink và Spark thông qua các Runner khác nhau.
- [Apache Storm](https://github.com/apache/storm): Hệ thống distributed real-time computing, phù hợp để tìm hiểu kiến trúc stream processing thời kỳ đầu. Khi chọn công nghệ cho dự án mới, nên đồng thời so sánh với Flink và Spark Structured Streaming.

## Distributed storage

- [Apache HBase](https://github.com/apache/hbase): Distributed columnar storage được xây dựng trên hệ sinh thái Hadoop, phù hợp với các trường hợp sử dụng cần đọc ghi ngẫu nhiên quy mô lớn.

## Data integration

- [Apache SeaTunnel](https://github.com/apache/seatunnel): Công cụ distributed data integration cho dữ liệu khối lượng lớn, cung cấp nhiều connector cho các data source, có thể dùng để đồng bộ offline và đồng bộ real-time.
- [Flink CDC](https://github.com/apache/flink-cdc): Công cụ streaming data integration dựa trên Flink, phù hợp để nắm bắt các thay đổi trong database và xây dựng real-time data pipeline.
- [Apache Flume](https://github.com/apache/flume): Hệ thống phân tán dùng để thu thập, tổng hợp và truyền dữ liệu log. Dự án vẫn được tiếp tục maintain, nhưng chủ yếu hướng đến các trường hợp sử dụng thu thập log truyền thống; khi chọn công nghệ cho dự án mới, nên đánh giá cùng với các giải pháp như Kafka Connect và SeaTunnel.

## Lakehouse table format

- [Apache Iceberg](https://github.com/apache/iceberg): Open table format hướng đến các dataset phân tích quy mô lớn, hỗ trợ các khả năng như Schema evolution, partition evolution và snapshot.
- [Apache Hudi](https://github.com/apache/hudi): Nền tảng incremental processing hướng đến data lake, tập trung vào update, delete, change stream và incremental query.
- [Apache Paimon](https://github.com/apache/paimon): Table format hướng đến lakehouse real-time, hỗ trợ đọc ghi stream-batch trên Flink và Spark, phù hợp với các trường hợp sử dụng chú trọng khả năng cập nhật real-time.
