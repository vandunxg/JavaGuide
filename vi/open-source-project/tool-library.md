---
title: Các thư viện công cụ mã nguồn mở Java thường dùng
description: "Đề xuất các thư viện công cụ Java mã nguồn mở, bao gồm những dependency thường dùng như Lombok, Guava, Jackson, Hibernate Validator, Tika và PDFBox."
category: Dự án mã nguồn mở
icon: "mdi:library-outline"
---

Trang này tập hợp các Java library được thêm vào code thông qua Maven hoặc Gradle. GUI, CLI và management platform cần cài đặt hoặc deploy riêng được đặt trong [Development Tools](./tools.md).

## Công cụ cơ bản

- [Lombok](https://github.com/rzwitserloot/lombok): tạo `getter`, `setter`, `equals`, `hashCode`, `toString`, constructor và các boilerplate code khác thông qua annotation. Nó phụ thuộc vào compile-time annotation processing, vì vậy trước khi sử dụng trong team cần thống nhất IDE plugin và build configuration.
- [Guava](https://github.com/google/guava "guava"): Java core library do Google duy trì, cung cấp `Multimap`, `Multiset`, `BiMap`, immutable collection cùng các công cụ I/O, hash và xử lý string.
- [Hutool](https://github.com/looly/hutool "hutool"): Java utility library tổng hợp, bao phủ các thao tác thường gặp như file, string, date, encryption, cache và logging. Có khá nhiều module, nên đưa vào theo nhu cầu thực tế, tránh xem nó là đáp án mặc định cho mọi vấn đề.

## Xử lý JSON

- [Jackson Databind](https://github.com/FasterXML/jackson-databind): module data binding tổng quát của Jackson, dùng để chuyển đổi giữa JSON và Java object. Khi upgrade cần đồng thời theo dõi Jackson BOM và security advisory, tránh trộn lẫn phiên bản giữa các module.
- [Gson](https://github.com/google/gson): Java library mã nguồn mở của Google dùng để serialization và deserialization JSON, API đơn giản, phù hợp với các trường hợp có quy mô dependency nhỏ.

## Validation tham số

- [Hibernate Validator](https://github.com/hibernate/hibernate-validator): reference implementation của Jakarta Validation, cho phép khai báo constraint của field, method parameter và return value thông qua annotation.

## Phân tích sự cố và tối ưu performance

- [Arthas](https://github.com/alibaba/arthas "arthas"): Java diagnostic tool mã nguồn mở của Alibaba, có thể monitor và chẩn đoán Java application theo thời gian thực. Tool cung cấp nhiều command và tính năng để phân tích vấn đề performance của application, bao gồm resource consumption và load time trong quá trình startup.
- [Async Profiler](https://github.com/async-profiler/async-profiler): Java performance profiling tool bất đồng bộ với overhead thấp, dùng để thu thập và phân tích dữ liệu performance của application.

## Xử lý file và media

### Phân tích tài liệu

- [Tika](https://github.com/apache/tika): Apache Tika toolkit có thể detect và extract metadata cùng text từ hơn một nghìn loại file khác nhau, chẳng hạn PPT, XLS và PDF.

### Metadata của hình ảnh và media

- [TwelveMonkeys ImageIO](https://github.com/haraldk/TwelveMonkeys): bổ sung thêm plugin và extension cho nhiều image format của Java ImageIO, phù hợp với các trường hợp cần đọc format TIFF, WebP, PSD và các format khác.
- [metadata-extractor](https://github.com/drewnoakes/metadata-extractor): extract metadata như EXIF, IPTC, XMP và ICC từ hình ảnh, video và audio.

### Excel

- [FastExcel](https://github.com/fast-excel/fastexcel): tool đọc và ghi Excel cho Java với performance cao và mức sử dụng memory thấp.
- [Excel Spring Boot Starter](https://github.com/pig-mesh/excel-spring-boot-starter): Spring Boot Starter được implement dựa trên FastExcel, dùng để đơn giản hóa thao tác đọc và ghi Excel.

### PDF

Với nhu cầu tạo PDF đơn giản, có thể chọn OpenPDF; khi cần parse, convert và extract text, có thể chọn Apache PDFBox. iText có nhiều tính năng hơn, nhưng community edition sử dụng AGPL license, nên closed-source commercial project cần đánh giá commercial license.

- [x-easypdf](https://gitee.com/dromara/x-easypdf): PDF framework được xây dựng dựa trên PDFBox và FOP, hỗ trợ export và edit PDF, phù hợp với các trường hợp tạo tài liệu thông thường.
- [iText](https://github.com/itext/itext7): Java library dùng để tạo, edit và enhance PDF document. Community edition sử dụng AGPL license, closed-source commercial project thường cần mua commercial license.
- [OpenPDF](https://github.com/LibrePDF/OpenPDF): PDF library sử dụng dual license LGPL/MPL, phát triển từ các version đầu của iText, phù hợp với nhu cầu tạo và edit PDF thường gặp.
- [Apache PDFBox](https://github.com/apache/pdfbox): sử dụng Apache license, hỗ trợ tạo, parse, convert và extract text từ PDF, có đầy đủ tính năng nhưng learning cost của API tương đối cao.
- [Apache FOP](https://github.com/apache/xmlgraphics-fop): dùng để chuyển đổi `XSL-FO` (Extensible Stylesheet Language Formatting Objects) thành PDF và các output format khác.

## SMS và email

- [SMS4J](https://github.com/dromara/SMS4J): framework tích hợp SMS, giải quyết quy trình rườm rà khi kết nối nhiều SMS SDK.
- [Simple Java Mail](https://github.com/bbottema/simple-java-mail): Java mail library nhẹ, hỗ trợ các trường hợp email như attachment, embedded image, signature và encryption.

## Cryptography

- [Bouncy Castle for Java](https://github.com/bcgit/bc-java): bản phân phối Bouncy Castle cho Java, cung cấp khả năng về cryptographic algorithm, certificate, CMS, OpenPGP và các tính năng khác. Không nên tự lắp ghép cryptography implementation; khi sử dụng cần đánh giá version dựa trên license của project, yêu cầu compliance và security update.

## Khác

- [oshi](https://github.com/oshi/oshi "oshi"): library cung cấp thông tin về operating system và hardware (native) cho Java, dựa trên JNA.
- [ip2region](https://github.com/lionsoul2014/ip2region): offline IP address geolocation library, cung cấp query client đa ngôn ngữ và data file nhỏ gọn. Kết quả định vị IP có giới hạn về độ chính xác, không nên được xem là căn cứ duy nhất cho quyết định về danh tính hoặc compliance.
- [agrona](https://github.com/real-logic/agrona): các data structure Java performance cao (`Buffers`, `Lists`, `Maps`, `Scalable Timer Wheel`……) và các utility method.
