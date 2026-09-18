---
title: Các dự án thực chiến Java mã nguồn mở chất lượng
description: Tuyển chọn các dự án thực chiến Java mã nguồn mở chất lượng, bao gồm nền tảng phát triển nhanh, hệ thống thương mại điện tử, quản lý quyền và các dự án có thể dùng để học tập và đưa vào CV.
category: Dự án mã nguồn mở
icon: "mdi:projector-screen-outline"
---

Dự án thực chiến phù hợp để quan sát cách mô hình hóa nghiệp vụ, kiểm soát quyền, truy cập dữ liệu, kiểm thử và deploy. Trước khi đưa vào CV, ít nhất hãy hoàn thành việc chạy local, hiểu một chuỗi nghiệp vụ cốt lõi và thực hiện một thay đổi có thể giải thích.

## Dự án mẫu chính thức

- [Spring Petclinic](https://github.com/spring-projects/spring-petclinic): ứng dụng mẫu do Spring chính thức duy trì. Quy mô nghiệp vụ không lớn, nhưng ranh giới giữa Web, truy cập dữ liệu, validation và kiểm thử khá rõ ràng, phù hợp để lần đầu đọc trọn vẹn một ứng dụng Spring.

## AI

- [interview-guide](https://github.com/Snailclimb/interview-guide): triển khai trên Spring Boot 4.0, Java 21, Spring AI, PostgreSQL, pgvector, RustFS và Redis, cung cấp các tính năng như phân tích CV, phỏng vấn mô phỏng bằng AI và truy vấn RAG từ knowledge base.
- [PaiAgent](https://github.com/itwanger/PaiAgent): nền tảng trực quan để điều phối workflow AI, cho phép kết hợp model và node xử lý bằng thao tác kéo thả, phù hợp để học cách định nghĩa workflow, lập lịch tác vụ và tích hợp nhiều model.

## Nền tảng phát triển nhanh

- [Snowy](https://gitee.com/xiaonuobase/snowy): nền tảng phát triển nhanh hỗ trợ thuật toán mật mã quốc gia, tách frontend/backend và mở rộng bằng plugin.
- [RuoYi](https://gitee.com/y_project/RuoYi): hệ thống quản lý quyền dựa trên Spring Boot, bao gồm các chức năng backend phổ biến như user, role, menu, department và dictionary, có thể chạy trực tiếp.
- [EuBackend](https://gitee.com/zhaoeryu/eu-backend): nền tảng phát triển nhanh nhẹ được xây dựng trên Spring Boot.
- [RuoYi-Vue-Pro](https://github.com/YunaiV/ruoyi-vue-pro): phiên bản Pro hoàn toàn mới của RuoYi-Vue, tối ưu và refactor toàn bộ chức năng, hỗ trợ data permission, SaaS multi-tenant, workflow Flowable, đăng nhập bên thứ ba, thanh toán và các tính năng khác.
- [RuoYi-Vue-Plus](https://gitee.com/dromara/RuoYi-Vue-Plus): phiên bản Plus hoàn toàn mới của RuoYi-Vue, viết lại toàn bộ chức năng của RuoYi-Vue, tích hợp Sa-Token, Mybatis-Plus, Jackson, SpringDoc, Hutool, đồng bộ OSS định kỳ và các tính năng khác.
- [pig](https://gitee.com/log4j/pig "pig"): hệ thống quản lý quyền RBAC dựa trên Spring Boot + Spring Cloud + OAuth2.
- [Guns](https://gitee.com/stylefeng/guns): framework nền tảng để phát triển ứng dụng Java hiện đại.
- [JeecgBoot](https://github.com/zhangdaiscott/jeecg-boot): nền tảng phát triển nhanh J2EE low-code dựa trên code generator, hỗ trợ tạo dự án có kiến trúc tách frontend/backend.
- [Erupt](https://gitee.com/erupt/erupt): framework full-stack low-code, dùng annotation Java để sinh động các page cùng chức năng thêm, xóa, sửa, tra cứu và kiểm soát quyền ở backend.
- [BallCat](https://github.com/ballcat-projects/ballcat): scaffold phát triển nhanh, ngoài quản lý quyền và scheduled task còn hỗ trợ lọc XSS, chống SQL injection và data masking.
- [JHipster](https://github.com/jhipster/generator-jhipster): nền tảng sinh ứng dụng, có thể tạo dự án kết hợp Spring Boot với các frontend framework như Angular và React.

## Hệ thống blog/forum

- [paicoding](https://github.com/itwanger/paicoding): community mã nguồn mở dễ dùng và mạnh mẽ, dựa trên technical stack chính của dòng Spring Boot và kèm tutorial chi tiết.
- [Halo](https://github.com/halo-dev/halo): công cụ xây dựng website mã nguồn mở, có thể dùng để tạo blog, knowledge base và website doanh nghiệp. Phù hợp với người quan tâm đến cơ chế plugin, quản lý nội dung và mở rộng theme.

## Hệ thống Wiki/tài liệu

- [kkFileView](https://gitee.com/kekingcn/file-online-preview): giải pháp preview tài liệu online, hỗ trợ preview gần như mọi định dạng tài liệu phổ biến, chẳng hạn doc, docx, ppt, pptx, wps, xls, xlsx, zip, rar, ofd, xmind, bpmn, eml, epub, 3ds, dwg, psd, mp4, mp3, v.v.

## Hệ thống quản lý file/cloud drive

- [free-fs](https://gitee.com/dh_free/free-fs): hệ thống quản lý cloud storage dựa trên Spring Boot, MyBatis-Plus, MySQL, Sa-Token và Layui, có thể kết nối với Qiniu Cloud và Alibaba Cloud OSS, hỗ trợ upload, download, preview, di chuyển, đổi tên và kiểm soát quyền.
- [zfile](https://github.com/zfile-dev/zfile): cloud drive online được xây dựng trên Spring Boot + Vue, hỗ trợ kết nối với S3, OneDrive, SharePoint, Google Drive, Duojiyun, Upyun, local storage, FTP, SFTP và các storage source khác; hỗ trợ duyệt ảnh online, phát audio/video, cùng các loại file text, Office, obj (3d) và các loại khác.

## Hệ thống thi cử/luyện bài

- [PlayEdu](https://github.com/PlayEdu/PlayEdu): hệ thống mã nguồn mở phù hợp để xây dựng nền tảng đào tạo nội bộ, nhằm giúp doanh nghiệp/tổ chức xây dựng nền tảng đào tạo nội bộ mang thương hiệu riêng.
- [uexam](https://gitee.com/mindskip/uexam): hệ thống thi online, bao gồm ngân hàng câu hỏi, đề thi, kỳ thi, chấm bài và quản lý user.

## Hệ thống thương mại điện tử

Chuỗi nghiệp vụ và dependency của dự án mall khá nhiều. Nếu chưa từng độc lập hoàn thành một dự án Spring Boot, nên bắt đầu từ ví dụ có quy mô nhỏ hơn, đừng trực tiếp coi các dự án dưới đây là template cho đồ án tốt nghiệp.

- [mall](https://github.com/macrozheng/mall "mall"): dự án thương mại điện tử bao gồm mall frontend và hệ thống quản trị backend, backend chủ yếu dùng Spring Boot và MyBatis.
- [mall-swarm](https://github.com/macrozheng/mall-swarm "mall-swarm"): hệ thống mall microservice, technical stack chủ yếu là Spring Cloud Greenwich, Spring Boot 2, MyBatis, Docker và Elasticsearch. Đọc một hệ thống có sẵn có giá trị tham khảo; trước khi chọn stack cho dự án mới cần đánh giá version framework và cập nhật bảo mật.

## Hệ thống bán vé

- [Damai](https://gitee.com/java-up-up/damai): dự án thực chiến mua vé concert, tập trung thể hiện giải pháp xử lý inventory, rate limiting và order trong trường hợp săn vé có concurrency cao.

## Ứng dụng theo ngành

- [DataEase](https://github.com/dataease/dataease): nền tảng BI và data visualization mã nguồn mở, phù hợp để học cách triển khai tiếp nhận data source, quản lý quyền, cấu hình chart và dashboard.
- [ThingsBoard](https://github.com/thingsboard/thingsboard): nền tảng IoT mã nguồn mở, bao phủ quản lý thiết bị, thu thập dữ liệu, xử lý rule và visualization. Quy mô dự án khá lớn, phù hợp hơn để đọc sau khi đã có nền tảng Spring và message system.
- [Apache Fineract](https://github.com/apache/fineract): nền tảng core banking mã nguồn mở có khả năng mở rộng, bao gồm các domain model tài chính như account, loan và transaction. Business rule phức tạp, phù hợp để học domain modeling nhưng không thích hợp làm dự án nhập môn Java.
- [Jeepay](https://gitee.com/jeequan/jeepay): hệ thống thanh toán mã nguồn mở, bao phủ các API transaction, refund, transfer và split settlement, đồng thời kết nối với các channel như WeChat, Alipay và UnionPay. Dự án thanh toán liên quan đến yêu cầu về tiền và compliance, phù hợp hơn để học business modeling và thiết kế API, không được dùng trực tiếp trong production khi chưa security audit.

## Tự xây lại bánh xe

- [guide-rpc-framework](https://github.com/Snailclimb/guide-rpc-framework): framework RPC được triển khai trên Netty, Kryo và ZooKeeper, kèm quá trình triển khai và tutorial đi kèm.
- [mini-spring](https://github.com/DerekYRC/mini-spring): framework Spring phiên bản đơn giản hóa, giúp bạn nhanh chóng làm quen với source code Spring và nắm được nguyên lý cốt lõi của Spring. Code được đơn giản hóa ở mức cao, giữ lại các chức năng cốt lõi của Spring như IoC, AOP, resource loader và các chức năng cốt lõi khác.
- [mini-spring-cloud](https://github.com/DerekYRC/mini-spring-cloud): Spring Cloud phiên bản đơn giản hóa được tự viết, dùng để hiểu các nguyên lý cốt lõi như service registration, configuration và invocation. Đọc thêm: [Tự viết một Spring Cloud phiên bản đơn giản hóa!](https://mp.weixin.qq.com/s/v3FUp-keswE2EhcTaLpSMQ).
