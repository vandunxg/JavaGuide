---
title: Công cụ phát triển mã nguồn mở chất lượng cao cho Java
description: Khuyến nghị các công cụ phát triển mã nguồn mở chất lượng cao cho Java, bao quát các công cụ thiết yếu để kiểm tra chất lượng code, phân tích bảo mật code, build project, test framework và deploy container.
category: Dự án mã nguồn mở
icon: "mdi:tools"
---

Trang này tập hợp các GUI, CLI, platform hoạt động độc lập và các công cụ kiểm tra tích hợp vào build. Các thư viện dùng chung có thể được dùng trực tiếp làm dependency của mã nghiệp vụ được đặt tại [thư viện công cụ](./tool-library.md).

## Chất lượng code

- [SonarQube](https://github.com/SonarSource/sonarqube "sonarqube"): platform kiểm tra chất lượng code liên tục, dùng để phát hiện lỗi, security hotspot, code trùng lặp và vấn đề về khả năng bảo trì.
- [Spotless](https://github.com/diffplug/spotless): Spotless là công cụ format code hỗ trợ nhiều ngôn ngữ, hỗ trợ build dưới dạng Plugin cho Maven và Gradle.
- [Checkstyle](https://github.com/checkstyle/checkstyle "checkstyle"): kiểm tra Java source code có tuân thủ coding convention hay không, có thể tích hợp vào quy trình build của Maven, Gradle và CI.
- [PMD](https://github.com/pmd/pmd "pmd"): static code analyzer đa ngôn ngữ có khả năng mở rộng.
- [SpotBugs](https://github.com/spotbugs/spotbugs "spotbugs"): kế nhiệm FindBugs, tìm lỗi trong Java code thông qua static analysis bytecode.
- [Error Prone](https://github.com/google/error-prone): bắt các lỗi coding Java thường gặp trong giai đoạn compile, có thể tích hợp vào quy trình build của `javac`, Maven và Gradle.
- [NullAway](https://github.com/uber/NullAway): công cụ static check null pointer của Java với overhead build thấp, thường được dùng cùng Error Prone. Công cụ này phụ thuộc vào nullability annotation và việc áp dụng incremental, không thể thay thế test runtime.
- [ArchUnit](https://github.com/TNG/ArchUnit): dùng test code Java để khai báo và kiểm tra các architectural rule như phân tầng, hướng dependency, cấu trúc package, phù hợp để biến quy ước của team thành automated test.

## Bảo mật code

- [OpenTaint](https://github.com/seqra/opentaint/blob/main/docs/translations/README.zh.md "opentaint"): công cụ taint analysis/SAST mã nguồn mở dành cho ứng dụng Java, Kotlin và Spring Boot, có thể dùng để phát hiện các rủi ro bảo mật như SQL injection, XSS và SSRF.

## Build project

- [Maven](https://maven.apache.org/): một công cụ quản lý và tìm hiểu project phần mềm. Dựa trên khái niệm Project Object Model (POM), Maven có thể quản lý build, report và tài liệu của project từ một nguồn thông tin trung tâm. Giới thiệu chi tiết: [Tổng hợp các khái niệm cốt lõi của Maven](https://javaguide.cn/tools/maven/maven-core-concepts.html).
- [Gradle](https://gradle.org/) : một công cụ build automation mã nguồn mở, đủ linh hoạt để build gần như mọi loại phần mềm. Gradle đưa ra rất ít giả định về việc bạn muốn build gì hoặc build như thế nào, nhờ đó Gradle đặc biệt linh hoạt. Giới thiệu chi tiết: [Tổng hợp các khái niệm cốt lõi của Gradle](https://javaguide.cn/tools/gradle/gradle-core-concepts.html).

## Decompilation

- [JADX](https://github.com/skylot/jadx): công cụ CLI và GUI dùng để tạo Java source code từ file Android Dex và Apk.

## Database

### Database management

- [Chat2DB](https://github.com/OtterMind/Chat2DB): database client và SQL workbench đa nền tảng, hỗ trợ nhiều database, đồng thời cung cấp các khả năng hỗ trợ bằng AI như generate, giải thích và tối ưu SQL.
- [Beekeeper Studio](https://github.com/beekeeper-studio/beekeeper-studio): công cụ database management đa nền tảng, giao diện đẹp, hỗ trợ SQLite, MySQL, MariaDB, Postgres, CockroachDB, SQL Server và Amazon Redshift.
- [DBeaver](https://github.com/dbeaver/dbeaver): công cụ database management đa nền tảng dựa trên Java. Bản Community hỗ trợ nhiều database như MySQL, PostgreSQL, MariaDB, SQLite, Oracle, Db2 và SQL Server.
- [Kangaroo](https://gitee.com/dbkangaroo/kangaroo): database management client đa nền tảng, hỗ trợ các database như SQLite, MySQL và PostgreSQL, cùng các thao tác query, modeling, sync và import/export.

### Redis

- [Another Redis Desktop Manager](https://github.com/qishibo/AnotherRedisDesktopManager/blob/master/README.zh-CN.md): Redis desktop client đa nền tảng, tương thích với Windows, macOS và Linux.
- [Tiny RDM](https://github.com/tiny-craft/tiny-rdm): Redis desktop client đa nền tảng dựa trên WebView2, tương thích với Windows, macOS và Linux.
- [RedisInsight](https://github.com/RedisInsight/RedisInsight): GUI chính thức của Redis, dùng để duyệt key-value, chạy command, phân tích memory usage và xử lý slow query.
- [CacheCloud](https://github.com/sohutv/cachecloud): platform cloud management cho Redis, hỗ trợ quản lý hiệu quả nhiều architecture của Redis (Standalone, Sentinel, Cluster), giảm chi phí vận hành Redis quy mô lớn, nâng cao khả năng kiểm soát và hiệu suất sử dụng tài nguyên.
- [RedisShake](https://github.com/tair-opensource/RedisShake): công cụ dùng để xử lý và migrate dữ liệu Redis.

## Docker

- [Portainer](https://github.com/portainer/portainer): quản lý các resource như container, image, network và storage thông qua giao diện Web.

## Kafka

- [Kafbat UI](https://github.com/kafbat/kafka-ui): Web UI mã nguồn mở dùng để monitor và quản lý Apache Kafka cluster.
- [Kafdrop](https://github.com/obsidiandynamics/kafdrop): Web UI dùng để xem Kafka Topic, message và Consumer Group.
- [Redpanda Console](https://github.com/redpanda-data/console): Web management interface dành cho Kafka và Redpanda, có thể duyệt Topic, Consumer Group, Schema và message realtime.
