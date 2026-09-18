---
title: Maven Best Practices
description: Tổng hợp các best practices thường gặp khi sử dụng Maven trong dự án Java, bao gồm cấu trúc thư mục chuẩn, phiên bản biên dịch, quản lý dependency, Profile, Maven Wrapper, build CI và sử dụng plugin.
category: Công cụ phát triển
head:
  - - meta
    - name: keywords
      content: Maven coordinates,Maven repository,Maven lifecycle,Maven multi-module management,Maven Wrapper,dependency management
---

> Bài viết này được JavaGuide dịch và hoàn thiện, địa chỉ bài viết gốc: <https://medium.com/@AlexanderObregon/maven-best-practices-tips-and-tricks-for-java-developers-438eca03f72b> .

Maven là một công cụ tự động hóa build dự án Java được sử dụng rộng rãi. Maven đơn giản hóa quy trình build và giúp chúng ta quản lý dependency. Có thể tham khảo bài viết này để tìm hiểu chi tiết về Maven: [Tổng hợp các khái niệm cốt lõi của Maven](./maven-core-concepts.md).

Bài viết này không đi sâu vào các khái niệm cơ bản của Maven, mà chủ yếu thảo luận về những vấn đề dễ gặp lỗi hơn trong dự án: cấu trúc thư mục, phiên bản biên dịch, phiên bản dependency, cấu hình môi trường, Wrapper, CI và quản lý plugin.

## Cấu trúc thư mục chuẩn của Maven

Maven tuân theo cấu trúc thư mục chuẩn để duy trì tính nhất quán giữa các dự án. Tuân theo cấu trúc này giúp các developer khác dễ hiểu dự án hơn.

Cấu trúc thư mục chuẩn của dự án Maven như sau:

```groovy
src/
  main/
    java/
    resources/
  test/
    java/
    resources/
pom.xml
```

- `src/main/java`: thư mục source code
- `src/main/resources`: thư mục resource
- `src/test/java`: thư mục test code
- `src/test/resources`: thư mục test resource

Đây chỉ là ví dụ đơn giản nhất về cấu trúc thư mục của một dự án Maven. Trong dự án thực tế, chúng ta còn phân chia chi tiết hơn theo quy ước của dự án.

## Chỉ định rõ phiên bản Java dùng để biên dịch

Không nên phụ thuộc vào phiên bản biên dịch mặc định của Maven hoặc plugin. Dự án nên khai báo rõ phiên bản Java mục tiêu trong `pom.xml`. Đối với các dự án Java hiện đại, nên ưu tiên sử dụng `maven.compiler.release`. Thuộc tính này tương ứng với `javac --release` và ổn định hơn so với việc chỉ cấu hình riêng `source` và `target`.

Cần lưu ý rằng `javac --release` được cung cấp từ JDK 9; Maven Compiler Plugin 3.13.0 trở lên cũng hỗ trợ `maven.compiler.release` trên JDK 8 và sẽ tự động chuyển thành `source` và `target`. Nếu dự án vẫn sử dụng plugin hoặc môi trường build cũ hơn, hãy cấu hình tường minh `source`, `target`.

Ví dụ, nếu dự án cần biên dịch theo Java 17, có thể viết như sau:

```xml
<properties>
  <maven.compiler.release>17</maven.compiler.release>
</properties>
```

Nếu cần cấu hình trực tiếp Maven Compiler Plugin, cũng có thể viết như sau:

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.15.0</version>
      <configuration>
        <release>17</release>
      </configuration>
    </plugin>
  </plugins>
</build>
```

Không nên viết giá trị của `release` theo định dạng cũ như `1.8`. Ví dụ, Java 8 viết là `8`, Java 17 viết là `17`, Java 21 viết là `21`.

## Quản lý dependency hiệu quả

Hệ thống quản lý dependency của Maven là một trong những tính năng mạnh nhất của Maven. Trong parent POM, định nghĩa phiên bản dependency dùng chung thông qua `dependencyManagement` có thể tránh việc nhiều module phải tự viết một phiên bản, từ đó giảm khả năng xảy ra xung đột dependency.

Ví dụ, giả sử chúng ta có một module cha và hai module con A và B, đồng thời muốn sử dụng JUnit 5 trong tất cả module. Có thể định nghĩa phiên bản JUnit trong file `pom.xml` của module cha thông qua `<dependencyManagement>`:

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter</artifactId>
      <version>5.10.2</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

Trong file `pom.xml` của module con A và B, chỉ cần tham chiếu `groupId` và `artifactId` của JUnit:

```xml
<dependencies>
  <dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
  </dependency>
</dependencies>
```

Đối với các ecosystem như Spring Boot và Spring Cloud đã cung cấp BOM, nên ưu tiên import BOM chính thức, sau đó lược bỏ phiên bản dependency cụ thể trong module nghiệp vụ. Cách này giúp giảm các vấn đề tương thích do “tự ghép phiên bản”.

## Sử dụng file cấu hình cho các môi trường khác nhau

File cấu hình Maven cho phép cấu hình thiết lập build cho các môi trường khác nhau, chẳng hạn như development, test và production. Định nghĩa profile trong file `pom.xml` và kích hoạt chúng bằng tham số dòng lệnh:

```xml
<profiles>
  <profile>
    <id>development</id>
    <activation>
      <activeByDefault>true</activeByDefault>
    </activation>
    <properties>
      <environment>dev</environment>
    </properties>
  </profile>
  <profile>
    <id>production</id>
    <properties>
      <environment>prod</environment>
    </properties>
  </profile>
</profiles>
```

Kích hoạt profile bằng dòng lệnh:

```bash
mvn clean install -P production
```

## Giữ pom.xml sạch sẽ và ngăn nắp

File `pom.xml` được tổ chức tốt sẽ dễ bảo trì và dễ hiểu hơn. Dưới đây là một số mẹo để duy trì file `pom.xml` sạch sẽ:

- Nhóm các dependency và plugin tương tự lại với nhau.
- Sử dụng comment để mô tả mục đích của dependency hoặc plugin cụ thể.
- Đặt các phiên bản dùng chung trong thẻ `<properties>`, hoặc quản lý tập trung trong `dependencyManagement` / `pluginManagement` của parent POM.

```xml
<properties>
  <junit.version>5.10.2</junit.version>
  <mockito.version>5.12.0</mockito.version>
</properties>
```

Phiên bản plugin cũng nên được khai báo tường minh. Không nên phụ thuộc vào phiên bản plugin mặc định của Maven, nếu không hành vi có thể khác nhau giữa các phiên bản Maven hoặc môi trường build khác nhau.

## Sử dụng Maven Wrapper

Maven Wrapper là một công cụ dùng để quản lý và sử dụng Maven, cho phép chạy và build dự án Maven mà không cần cài đặt Maven từ trước.

Tài liệu chính thức của Maven giới thiệu về Maven Wrapper như sau:

> The Maven Wrapper is an easy way to ensure a user of your Maven build has everything necessary to run your Maven build.
>
> Maven Wrapper là một cách đơn giản để đảm bảo người dùng build Maven có mọi thứ cần thiết để chạy Maven build.

Maven Wrapper có thể đảm bảo quy trình build sử dụng đúng phiên bản Maven, rất tiện lợi. Để sử dụng Maven Wrapper, hãy chạy lệnh sau trong thư mục dự án:

```bash
mvn wrapper:wrapper
```

Lệnh này sẽ tạo các file Maven Wrapper trong dự án. Sau đó, chúng ta có thể dùng `./mvnw` (hoặc `./mvnw.cmd` trên Windows) thay cho `mvn` để thực thi các lệnh Maven.

Với dự án nhóm, nên commit `mvnw`, `mvnw.cmd` và thư mục `.mvn/wrapper/`. Nhờ đó, thành viên mới hoặc môi trường CI không cần cài đặt trước phiên bản Maven được chỉ định mà vẫn có thể dùng phiên bản Maven do dự án khai báo để build.

## Tự động hóa build thông qua continuous integration

Tích hợp dự án Maven với hệ thống continuous integration (CI), chẳng hạn như Jenkins hoặc GitHub Actions, có thể đảm bảo code được tự động build, test và deploy. CI giúp phát hiện vấn đề sớm và cung cấp quy trình build nhất quán trong toàn bộ team. Dưới đây là một ví dụ đơn giản về workflow GitHub Actions cho dự án Maven:

```yaml
name: Java CI with Maven

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: "17"
          distribution: "temurin"
          cache: "maven"

      - name: Build with Maven
        run: ./mvnw -B clean verify
```

Trong CI, nên sử dụng `clean verify` vì lệnh này sẽ thực hiện test và các bước kiểm tra cần thiết. `install` sẽ cài artifact build vào local repository, chỉ cần sử dụng khi các bước tiếp theo thực sự phụ thuộc vào kết quả cài đặt cục bộ.

## Sử dụng Maven plugin để có thêm chức năng

Có nhiều Maven plugin có thể dùng để mở rộng chức năng của Maven. Một số plugin phổ biến gồm có (ba plugin đầu tiên là plugin đi kèm Maven, ba plugin cuối do bên thứ ba cung cấp):

- maven-surefire-plugin: cấu hình và thực thi unit test.
- maven-failsafe-plugin: cấu hình và thực thi integration test.
- maven-javadoc-plugin: tạo tài liệu dự án theo định dạng Javadoc.
- maven-checkstyle-plugin: bắt buộc tuân thủ coding standard và best practices.
- jacoco-maven-plugin: độ bao phủ unit test.
- sonar-maven-plugin: phân tích chất lượng code.
- ……

Ví dụ sử dụng jacoco-maven-plugin:

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.jacoco</groupId>
      <artifactId>jacoco-maven-plugin</artifactId>
      <version>0.8.12</version>
      <executions>
        <execution>
          <goals>
            <goal>prepare-agent</goal>
          </goals>
        </execution>
        <execution>
          <id>generate-code-coverage-report</id>
          <phase>test</phase>
          <goals>
            <goal>report</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

Nếu các plugin có sẵn này không đáp ứng được nhu cầu, chúng ta cũng có thể tự tạo plugin.

Hãy khám phá các plugin có thể sử dụng và cấu hình chúng trong file `pom.xml` để nâng cao quy trình phát triển.

## Tổng kết

Điều quan trọng nhất của Maven không phải là “có thể chạy được dự án hay không”, mà là giúp team sử dụng phương thức build nhất quán trong môi trường local, CI và deploy. Trong dự án thực tế, nên ưu tiên thực hiện những việc sau: sử dụng cấu trúc thư mục chuẩn, khai báo tường minh phiên bản Java và plugin, quản lý phiên bản dependency thông qua parent POM, BOM và `dependencyManagement`, commit Maven Wrapper, đồng thời cố định JDK và lệnh build Maven trong CI.
