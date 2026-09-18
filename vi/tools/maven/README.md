---
title: "Chuyên đề Maven: POM, coordinates, repository, dependency management, lifecycle, plugin và multi-module project"
description: "Lộ trình học Maven về phỏng vấn và build project, bao quát POM, coordinates, repository, dependency scope, lifecycle, plugin, multi-module project, Maven Wrapper và best practices, phù hợp với Java backend developer."
category: Công cụ phát triển
tag:
  - Maven
  - Build project
  - Dependency management
sitemap:
  changefreq: weekly
  priority: 0.85
head:
  - - meta
    - name: keywords
      content: Maven,POM,Maven coordinates,Maven repository,dependency management,dependency scope,Maven lifecycle,Maven plugin,multi-module project,Java project build
---

Maven là công cụ build và dependency management phổ biến nhất trong các Java backend project. Khi học Maven, bạn không nên chỉ biết copy `pom.xml`, mà còn cần hiểu các khái niệm cơ bản như coordinates, repository, transitive dependency, lifecycle, plugin và quản lý multi-module.

## Dành cho ai

- Những bạn đang học build project Java và dependency management.
- Developer dùng Maven để viết project nhưng thường bị vướng dependency conflict, version không thống nhất, quản lý multi-module hoặc CI build bị kẹt.
- Độc giả chuẩn bị phỏng vấn, cần trình bày rõ core concept và best practices của Maven.
- Engineer cần bảo trì project Spring Boot, microservice hoặc multi-module Java.

## Trọng tâm học

- POM là core config của Maven project, còn coordinates dùng để định danh duy nhất một artifact.
- Maven repository được chia thành local repository, private repository và central repository; dependency resolution sẽ tìm kiếm theo một thứ tự nhất định.
- Dependency scope, transitive dependency, dependency exclusion và version management quyết định các Jar package mà project sử dụng cuối cùng.
- Lifecycle định nghĩa các build phase, còn plugin chịu trách nhiệm thực thi các task như compile, test và package.
- Maven Wrapper có thể cố định version Maven mà project sử dụng, phù hợp với team collaboration và CI environment.
- Với multi-module project, cần chú ý đến parent POM, `dependencyManagement`, `pluginManagement` và module boundary.

## Thứ tự đọc đề xuất

1. [Tổng hợp core concept của Maven](./maven-core-concepts.md): trước tiên tìm hiểu POM, coordinates, repository, dependency, lifecycle, plugin và multi-module project.
2. [Best practices của Maven](./maven-best-practices.md): tiếp theo học standard directory structure, compiler version, BOM, dependency version management, Maven Wrapper, CI và các practice thường gặp.
3. Kết hợp với một project Spring Boot để xem `pom.xml`: tập trung vào parent project, dependency scope, plugin config và dependency tree cuối cùng.

## Bài viết cốt lõi

- [Tổng hợp core concept của Maven](./maven-core-concepts.md): giới thiệu một cách có hệ thống về định vị của Maven, POM, coordinates, repository, dependency, lifecycle, plugin và multi-module project.
- [Best practices của Maven](./maven-best-practices.md): tổng hợp standard directory structure, compiler version, thống nhất dependency version, Maven Wrapper, CI và các đề xuất khi sử dụng hằng ngày.

## Câu hỏi thường gặp

- Maven là gì? Nó chủ yếu giải quyết những vấn đề nào?
- POM, groupId, artifactId, version lần lượt là gì?
- Local repository, private repository và central repository khác nhau như thế nào?
- Maven transitive dependency là gì? Làm thế nào để kiểm tra dependency conflict?
- `dependencyManagement` và `dependencies` khác nhau như thế nào?
- Lifecycle và plugin của Maven có quan hệ gì?
- Vì sao project của team thường được khuyến nghị commit Maven Wrapper?
- `compile`, `provided`, `runtime`, `test` và các dependency scope khác có điểm gì khác nhau?
- Vì sao multi-module project thường cần parent POM?

## Chuyên đề liên quan

- [Hệ thống kiến thức về công cụ phát triển](../)
- [Tổng hợp core concept của Gradle](../gradle/gradle-core-concepts.md)
- [Chuyên đề Git](../git/)
- [Java Basics](../../java/basis/java-basic-questions-01.md)

<!-- @include: @article-footer.snippet.md -->
