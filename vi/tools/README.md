---
title: "Hệ thống kiến thức về công cụ phát triển: Maven, Gradle, Git, GitHub, Docker và IDEA"
description: Lộ trình học công cụ phát triển backend, bao quát Maven, Gradle, Git, GitHub, Docker, IDEA, xây dựng project, quản lý dependency, version control, cộng tác code và deploy container.
category: Công cụ phát triển
tag:
  - Công cụ phát triển
  - Maven
  - Git
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Công cụ phát triển,Maven,Gradle,Git,GitHub,Docker,IDEA,quản lý dependency,xây dựng project,version control,deploy container,phát triển backend
---

<!-- @include: @small-advertisement.snippet.md -->

Bộ **kiến thức về công cụ phát triển** này dành cho việc học backend và phát triển hằng ngày, sắp xếp các bài viết về công cụ phát triển của trang này theo thứ tự “xây dựng project -> quản lý dependency -> version control -> nâng cao hiệu quả cộng tác -> bàn giao container”.

Công cụ phát triển không chỉ là biết gõ vài lệnh; quan trọng hơn là hiểu vai trò của chúng trong cộng tác nhóm, chuẩn hóa engineering, tính nhất quán của môi trường và hiệu quả bàn giao.

## Dành cho ai

- Người đang học backend, cần bổ sung các công cụ engineering thường dùng.
- Độc giả chuẩn bị ứng tuyển fresher hoặc experienced, muốn trả lời vững hơn các câu hỏi về những công cụ như Maven, Git, Docker.
- Developer đã có thể viết code nghiệp vụ nhưng chưa thành thạo về xung đột dependency, cộng tác bằng Git branch, Docker image và quản lý container.
- Engineer muốn nâng cao hiệu quả xây dựng project, cộng tác code, bàn giao môi trường và phát triển hằng ngày.

## Trọng tâm học

- Maven và Gradle giải quyết các vấn đề về xây dựng project, quản lý dependency, lifecycle, Wrapper và mở rộng bằng plugin.
- Git là năng lực nền tảng của cộng tác nhóm; trọng tâm không phải là học thuộc lệnh mà là hiểu working tree, staging area, commit, branch, merge và xử lý conflict.
- GitHub không chỉ là nền tảng lưu trữ code mà còn hỗ trợ cộng tác open source, giới thiệu cá nhân, đọc code, tự động hóa bằng Actions và quản lý project.
- Docker chủ yếu giải quyết các vấn đề về tính nhất quán của môi trường, cô lập khi deploy, phân phối image, nhanh chóng dựng các service dependency ở local và orchestration ứng dụng nhiều container.
- Kiến thức về công cụ nên được luyện tập cùng project thực tế; chỉ xem khái niệm riêng lẻ dễ dẫn đến biết nhưng không dùng được.

## Thứ tự đọc đề xuất

1. [Tổng hợp core concept của Git](./git/git-intro.md): trước tiên nắm version control, commit, branch, merge và quy trình cộng tác.
2. [Tổng hợp core concept của Maven](./maven/maven-core-concepts.md): hiểu cách xây dựng Java project, POM, tọa độ, repository, dependency và lifecycle.
3. [Best practices của Maven](./maven/maven-best-practices.md): bổ sung quản lý version dependency, BOM, Maven Wrapper, CI và quy chuẩn sử dụng hằng ngày.
4. [Tổng hợp core concept của Docker](./docker/docker-intro.md): xây dựng nhận thức cơ bản về image, container, repository và Docker engine.
5. [Thực hành Docker](./docker/docker-in-action.md): luyện tập quản lý container, build image, data volume và xử lý các vấn đề thường gặp thông qua lệnh và tình huống.
6. [Tổng hợp core concept của Gradle](./gradle/gradle-core-concepts.md) và [Tổng hợp mẹo sử dụng GitHub](./git/github-tips.md): bổ sung Gradle Wrapper, GitHub Actions và kỹ năng đọc code theo nhu cầu project.

## Bài viết cốt lõi

### Xây dựng project và quản lý dependency

- [Chuyên đề Maven](./maven/): giải thích rõ core concept và best practices của Maven, là chuyên đề về công cụ được sử dụng phổ biến nhất để xây dựng Java backend project.
- [Tổng hợp core concept của Maven](./maven/maven-core-concepts.md): hiểu POM, tọa độ, repository, phạm vi dependency, lifecycle, plugin và multi-module project.
- [Best practices của Maven](./maven/maven-best-practices.md): tổng hợp cấu trúc thư mục tiêu chuẩn, version compile, quản lý dependency, Maven Wrapper, CI và các đề xuất thực tiễn thường dùng.
- [Tổng hợp core concept của Gradle](./gradle/gradle-core-concepts.md): tìm hiểu các core concept như Gradle, Groovy/Kotlin DSL, Gradle Wrapper, plugin và Task.

### Version control và cộng tác code

- [Chuyên đề Git](./git/): tập trung vào core concept, workflow của Git và các mẹo nâng cao hiệu quả với GitHub.
- [Tổng hợp core concept của Git](./git/git-intro.md): hiểu version control, working tree, staging area, commit, branch, merge, conflict và remote repository.
- [Tổng hợp mẹo sử dụng GitHub](./git/github-tips.md): tổng hợp các mẹo liên quan đến trang cá nhân, badge của project, đọc code, Actions, Explore/Trending và cộng tác open source.

### Container hóa và môi trường local

- [Chuyên đề Docker](./docker/): từ core concept đến thao tác thực tế, giúp hiểu việc bàn giao container và tính nhất quán của môi trường.
- [Tổng hợp core concept của Docker](./docker/docker-intro.md): hiểu container, image, repository, Docker engine cũng như sự khác biệt giữa container và virtual machine.
- [Thực hành Docker](./docker/docker-in-action.md): hoàn thành phần thực hành nhập môn Docker thông qua image, container, network, data volume, log và các lệnh xử lý sự cố.

### IDE và công cụ nâng cao hiệu quả

- [Tutorial IDEA](https://gitee.com/SnailClimb/awesome-idea-tutorial): tổng hợp cấu hình, plugin, shortcut và các mẹo nâng cao hiệu quả thường dùng của IntelliJ IDEA.

## Câu hỏi thường gặp

- POM, tọa độ, repository và phạm vi dependency của Maven lần lượt là gì?
- Lifecycle của Maven có quan hệ gì với plugin?
- Multi-module project của Maven quản lý dependency dùng chung và version build thông qua BOM, `dependencyManagement` và Maven Wrapper như thế nào?
- Gradle và Maven khác nhau thế nào? Khi nào cần tìm hiểu Gradle?
- Working tree, staging area, local repository và remote repository của Git lần lượt là gì?
- Git merge và rebase khác nhau thế nào? Nên xử lý conflict ra sao?
- Ngoài việc lưu trữ code, GitHub còn có thể giúp developer làm gì thông qua Profile README, Actions, Codespaces, Explore/Trending?
- Docker image và container có quan hệ gì? Container và virtual machine khác nhau thế nào?
- Vì sao Docker có thể giải quyết vấn đề môi trường development, test và deploy không nhất quán? Compose phù hợp để giải quyết vấn đề gì?

## Chuyên đề liên quan

- [Chuẩn bị phỏng vấn](../interview-preparation/)
- [Java Basics](../java/basis/java-basic-questions-01.md)
- [Spring&Spring Boot](../system-design/framework/spring/)
- [Open source project](../open-source-project/)

<!-- @include: @article-footer.snippet.md -->
