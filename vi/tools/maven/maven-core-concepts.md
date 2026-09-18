---
title: "Tổng hợp core concept của Maven"
description: "Bản chất của Apache Maven là một công cụ quản lý và tìm hiểu software project. Dựa trên concept Project Object Model (POM), Maven có thể quản lý build, report và documentation của project từ một nguồn thông tin trung tâm."
category: Công cụ phát triển
head:
  - - meta
    - name: keywords
      content: Maven coordinates,Maven repository,Maven lifecycle,Maven multi-module management
---

> Nội dung này chủ yếu được tổng hợp từ tài liệu chính thức của Maven, có lược bỏ tương ứng và chỉ giữ lại các phần quan trọng, không đề cập thực hành mà chủ yếu giới thiệu một số concept quan trọng.

## Giới thiệu Maven

Tài liệu chính thức của [Maven](https://github.com/apache/maven) giới thiệu Maven như sau:

> Apache Maven is a software project management and comprehension tool. Based on the concept of a project object model (POM), Maven can manage a project's build, reporting and documentation from a central piece of information.
>
> Bản chất của Apache Maven là một công cụ quản lý và tìm hiểu software project. Dựa trên concept Project Object Model (POM), Maven có thể quản lý build, report và documentation của project từ một nguồn thông tin trung tâm.

**POM là gì?** Mỗi Maven project đều có một file `pom.xml` nằm trong thư mục gốc, chứa thông tin chi tiết về build lifecycle của project. Thông qua file `pom.xml`, bạn có thể định nghĩa coordinates của project, project dependency, project information, plugin information và các cấu hình khác.

Với developer, Maven chủ yếu có 3 tác dụng:

1. **Build project**: cung cấp phương thức build project tự động, chuẩn hóa và cross-platform.
2. **Dependency management**: quản lý resource (jar package) mà project phụ thuộc một cách thuận tiện, nhanh chóng, tránh xung đột version giữa các resource.
3. **Cấu trúc phát triển thống nhất**: cung cấp cấu trúc project chuẩn và thống nhất.

Phần này không giới thiệu cách sử dụng Maven cơ bản. Bạn nên xem tutorial [Maven in 5 Minutes](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html) trên website chính thức.

## Maven coordinates

Third-party library và plugin mà project phụ thuộc có thể gọi chung là artifact. Mỗi artifact có thể được định danh duy nhất bằng Maven coordinates, gồm các coordinate element sau:

- **groupId** (bắt buộc): định nghĩa tổ chức hoặc company mà Maven project hiện tại trực thuộc. groupId thường được chia thành nhiều phần, thông thường phần đầu là domain, phần thứ hai là company name. Domain lại chia thành org, com, cn..., trong đó org là tổ chức phi lợi nhuận, com là tổ chức thương mại, cn là China. Lấy project tomcat của Apache open source community làm ví dụ, groupId của project này là org.apache, domain là org (vì tomcat là non-profit project), company name là apache, còn artifactId là tomcat.
- **artifactId** (bắt buộc): định nghĩa tên của Maven project hiện tại, là identifier duy nhất của project, tương ứng với tên thư mục gốc của project.
- **version** (bắt buộc): định nghĩa version hiện tại của Maven project.
- **packaging** (tùy chọn): định nghĩa packaging method của Maven project (chẳng hạn jar, war...), mặc định là jar.
- **classifier** (tùy chọn): thường dùng để phân biệt các artifact có nội dung khác nhau được build từ cùng một POM; có thể là chuỗi bất kỳ và được nối sau version.

Chỉ cần cung cấp coordinates chính xác, bạn có thể tìm artifact tương ứng trong Maven repository để sử dụng.

Ví dụ (thêm EasyExcel do Alibaba open source):

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>easyexcel</artifactId>
    <version>3.1.1</version>
</dependency>
```

Bạn có thể tìm gần như mọi artifact có thể sử dụng trên website <https://mvnrepository.com/>. Nếu project dùng Maven làm build tool, bạn chắc chắn sẽ thường xuyên truy cập website này.

![Maven repository](https://oss.javaguide.cn/github/javaguide/tools/maven/mvnrepository.com.png)

## Maven dependencies

Nếu artifact được tạo ra bằng Maven build (chẳng hạn file Jar) được project khác tham chiếu, artifact đó chính là dependency của project khác.

### Cấu hình dependency

**Ví dụ thông tin cấu hình**:

```xml
<project>
    <dependencies>
        <dependency>
            <groupId></groupId>
            <artifactId></artifactId>
            <version></version>
            <type>...</type>
            <scope>...</scope>
            <optional>...</optional>
            <exclusions>
                <exclusion>
                  <groupId>...</groupId>
                  <artifactId>...</artifactId>
                </exclusion>
          </exclusions>
        </dependency>
      </dependencies>
</project>
```

**Giải thích cấu hình**:

- dependencies: một file pom.xml chỉ có thể có một tag như vậy, đây là tag tổng dùng để quản lý dependencies.
- dependency: nằm trong tag dependencies, có thể có nhiều tag; mỗi tag biểu thị một dependency của project.
- groupId,artifactId,version (bắt buộc): coordinates cơ bản của dependency. Với mọi dependency, coordinates cơ bản là quan trọng nhất; Maven phải dựa vào coordinates mới tìm được dependency cần thiết. Ý nghĩa cụ thể của các element này đã được giải thích ở trên nên không nhắc lại.
- type (tùy chọn): type của dependency, tương ứng với packaging được định nghĩa trong project coordinates. Phần lớn trường hợp không cần khai báo element này, giá trị mặc định là jar.
- scope (tùy chọn): scope của dependency, giá trị mặc định là compile.
- optional (tùy chọn): đánh dấu dependency có phải optional hay không.
- exclusions (tùy chọn): dùng để loại bỏ transitive dependency, chẳng hạn khi xảy ra xung đột jar package.

### Dependency scope

**classpath** dùng để chỉ định vị trí lưu file `.class`; class loader sẽ load file `.class` cần thiết từ path này vào memory.

Maven có ba classpath khác nhau cho compile, test execution và runtime:

- **Compile classpath**: có hiệu lực khi compile main code.
- **Test classpath**: có hiệu lực khi compile và chạy test code.
- **Runtime classpath**: có hiệu lực khi project chạy.

Maven có các dependency scope sau:

- **compile**: compile dependency scope (mặc định). Scope này có hiệu lực trong cả compile, test và runtime, tức là phải sử dụng dependency Jar package khi compile, test và chạy.
- **test**: test dependency scope. Nhìn từ tên có thể biết scope này chỉ dùng cho test; khi compile và chạy project thì không thể sử dụng dependency loại này. Ví dụ điển hình là JUnit, chỉ cần khi compile test code và chạy test code.
- **provided**: dependency scope này có hiệu lực khi compile và test, nhưng không có hiệu lực ở runtime. Ví dụ, `servlet-api.jar` đã được Tomcat cung cấp, bạn chỉ cần nó trong compile phase.
- **runtime**: runtime dependency scope, có hiệu lực khi test và chạy nhưng không có hiệu lực khi compile main code. Ví dụ điển hình là JDBC driver implementation.
- **system**: system dependency scope. Khi sử dụng scope system, phải chỉ định rõ path của dependency file qua element systemPath; dependency không được resolve qua Maven repository nên có thể khiến build không portable.

### Transitive dependency

### Dependency conflict

**1. Với Maven, trong cùng một groupId và artifactId chỉ có thể sử dụng một version.**

```xml
<dependency>
    <groupId>in.hocg.boot</groupId>
    <artifactId>mybatis-plus-spring-boot-starter</artifactId>
    <version>1.0.48</version>
</dependency>
<!-- Chỉ sử dụng dependency version 1.0.49 -->
<dependency>
    <groupId>in.hocg.boot</groupId>
    <artifactId>mybatis-plus-spring-boot-starter</artifactId>
    <version>1.0.49</version>
</dependency>
```

Nếu các dependency cùng type nhưng khác version tồn tại trong cùng một pom file, chỉ dependency được khai báo sau mới được import.

**2. Hai dependency của project cùng import một dependency nào đó.**

Ví dụ, project có dependency relationship như sau:

```plain
Dependency path 1: A -> B -> C -> X(1.0)
Dependency path 2: A -> D -> X(2.0)
```

Trên hai dependency path này có hai version của X. Để tránh dependency trùng lặp, Maven chỉ chọn một version để resolve.

**Maven sẽ resolve và sử dụng version X nào?**

Khi gặp vấn đề này, Maven tuân theo hai nguyên tắc **ưu tiên path ngắn hơn** và **ưu tiên thứ tự khai báo**. Quá trình giải quyết vấn đề này còn được gọi là **Maven dependency mediation**.

**Ưu tiên path ngắn hơn**

```plain
Dependency path 1: A -> B -> C -> X(1.0) // dist = 3
Dependency path 2: A -> D -> X(2.0) // dist = 2
```

Dependency path 2 ngắn hơn, vì vậy X(2.0) sẽ được resolve và sử dụng.

Tuy nhiên, bạn cũng có thể thấy nguyên tắc ưu tiên path ngắn hơn không áp dụng được trong mọi trường hợp. Với trường hợp hai path có độ dài bằng nhau như dưới đây, không thể chỉ giải quyết bằng nguyên tắc này:

```plain
Dependency path 1: A -> B -> X(1.0) // dist = 2
Dependency path 2: A -> D -> X(2.0) // dist = 2
```

Vì vậy, Maven tiếp tục định nghĩa nguyên tắc ưu tiên thứ tự khai báo.

Nguyên tắc đầu tiên của dependency mediation không thể giải quyết mọi vấn đề. Chẳng hạn, với dependency relationship A->B->Y(1.0), A-> C->Y(2.0), độ dài dependency path của Y(1.0) và Y(2.0) bằng nhau, đều là 2. Maven định nghĩa nguyên tắc thứ hai của dependency mediation:

**Ưu tiên thứ tự khai báo**

Khi độ dài dependency path bằng nhau, thứ tự khai báo dependency trong `pom.xml` quyết định dependency nào được resolve và sử dụng; dependency xuất hiện trước sẽ thắng. Trong ví dụ này, nếu dependency của B được khai báo trước D thì X (1.0) sẽ được resolve và sử dụng.

```xml
<!-- A pom.xml -->
<dependencies>
    ...
    dependency B
    ...
    dependency D
</dependencies>
```

### Loại bỏ dependency

Chỉ dựa vào Maven để thực hiện dependency mediation không phù hợp trong nhiều trường hợp; bạn cần chủ động loại bỏ dependency.

Ví dụ, project hiện tại có dependency relationship như sau:

```plain
Dependency path 1: A -> B -> C -> X(1.5) // dist = 3
Dependency path 2: A -> D -> X(1.0) // dist = 2
```

Theo nguyên tắc ưu tiên path ngắn hơn, X(1.0) sẽ được resolve và sử dụng, tức là thực tế sử dụng version 1.0 của X.

Nhưng điều này có thể gây ra vấn đề: nếu C dependency sử dụng một class chỉ có trong X version 1.5, khi chạy project sẽ báo lỗi `NoClassDefFoundError`. Nếu C dependency sử dụng một method chỉ có trong X version 1.5, khi chạy project sẽ báo lỗi `NoSuchMethodError`.

Bây giờ bạn đã biết vì sao Maven project thường xuyên báo lỗi `NoClassDefFoundError` và `NoSuchMethodError` rồi chứ?

**Giải quyết thế nào?** Bạn có thể dùng tag `exclusion` để chủ động loại bỏ X(1.0).

```xml
<dependency>
    ......
    <exclusions>
      <exclusion>
        <artifactId>x</artifactId>
        <groupId>org.apache.x</groupId>
      </exclusion>
    </exclusions>
</dependency>
```

Thông thường khi giải quyết dependency conflict, chúng ta ưu tiên giữ version cao hơn. Lý do là phần lớn jar package vẫn đảm bảo backward compatibility khi nâng version.

Nếu version cao hơn đã sửa một số class hoặc method của version thấp hơn thì không thể giữ trực tiếp version cao hơn; thay vào đó nên cân nhắc tối ưu upper-level dependency, chẳng hạn nâng version của upper-level dependency.

Vẫn với ví dụ trên:

```plain
Dependency path 1: A -> B -> C -> X(1.5) // dist = 3
Dependency path 2: A -> D -> X(1.0) // dist = 2
```

Ta giữ version 1.5 của X, nhưng version này đã xóa một số class có trong X version 1.0. Khi đó, có thể cân nhắc nâng version của D lên version tương thích với X.

## Maven repositories

Trong Maven world, mọi dependency, plugin hoặc output của project build đều có thể gọi là **artifact**.

Coordinates và dependency là cách biểu diễn logic của artifact trong Maven world, còn cách biểu diễn vật lý của artifact là file. Maven quản lý thống nhất các file này thông qua repository. Mọi artifact đều có một bộ coordinates để định danh duy nhất. Sau khi có repository, bạn không cần import artifact thủ công mà chỉ cần cung cấp coordinates là có thể tìm artifact đó trong Maven repository.

Maven repository được chia thành:

- **Local repository**: một thư mục trên computer chạy Maven, dùng để cache artifact tải từ xa và chứa các artifact tạm thời chưa publish. Có thể xem cấu hình path local repository của Maven trong file `settings.xml`; path mặc định là `${user.home}/.m2/repository`.
- **Remote repository**: Maven repository do official organization hoặc tổ chức khác duy trì.

Maven remote repository có thể chia thành:

- **Central repository**: repository do Maven community duy trì, lưu package của phần lớn open source software và là default configuration của Maven, không cần developer cấu hình thêm. Để thuận tiện tra cứu, repository còn cung cấp [địa chỉ tìm kiếm](https://search.maven.org/), giúp developer tìm coordinates của artifact cần thiết nhanh hơn.
- **Private repository**: một loại remote Maven repository đặc biệt, là repository service được dựng trong LAN. Private repository thường được cấu hình làm mirror của remote repository trên Internet để Maven user trong LAN sử dụng.
- **Public repository khác**: một số public repository nhằm tăng tốc truy cập (chẳng hạn Alibaba Cloud Maven mirror repository), hoặc chứa một số artifact không có trong central repository.

Thứ tự tìm dependency package của Maven:

1. Tìm trong local repository trước, nếu có thì sử dụng trực tiếp.
2. Nếu không tìm thấy trong local repository, tìm trong remote repository và download package về local repository.
3. Nếu không tìm thấy trong remote repository thì báo lỗi.

## Maven lifecycle

Maven lifecycle nhằm abstract và thống nhất toàn bộ build process, bao gồm gần như mọi build step như clean, initialize, compile, test, package, integration test, verify, deploy project và generate site.

Maven định nghĩa 3 lifecycle `META-INF/plexus/components.xml`:

- `default` lifecycle
- `clean` lifecycle
- `site` lifecycle

Các lifecycle này độc lập với nhau, mỗi lifecycle gồm nhiều phase. Các phase có thứ tự, tức là phase sau phụ thuộc phase trước. Khi thực thi một phase, các phase đứng trước nó sẽ được thực thi trước.

Cú pháp command để thực thi Maven lifecycle:

```bash
mvn phase [phase2] ...[phase n]
```

### `default` lifecycle

`default` lifecycle được định nghĩa khi không có plugin nào liên kết, là lifecycle chính của Maven dùng để build application, gồm tổng cộng 23 phase.

```xml
<phases>
  <!-- Xác minh project chính xác và mọi thông tin cần thiết đều có để hoàn tất build process -->
  <phase>validate</phase>
  <!-- Thiết lập trạng thái khởi tạo, chẳng hạn thiết lập property -->
  <phase>initialize</phase>
  <!-- Generate source code cần được đưa vào compile phase -->
  <phase>generate-sources</phase>
  <!-- Xử lý source code -->
  <phase>process-sources</phase>
  <!-- Generate resource cần được đưa vào package -->
  <phase>generate-resources</phase>
  <!-- Copy và xử lý resource vào target directory, chuẩn bị cho package phase. -->
  <phase>process-resources</phase>
  <!-- Compile source code của project -->
  <phase>compile</phase>
  <!-- Post-process các file được generate khi compile, chẳng hạn bytecode enhancement/optimization cho Java class -->
  <phase>process-classes</phase>
  <!-- Generate mọi test source code cần được đưa vào compile phase -->
  <phase>generate-test-sources</phase>
  <!-- Xử lý test source code -->
  <phase>process-test-sources</phase>
  <!-- Generate test source code cần được đưa vào compile phase -->
  <phase>generate-test-resources</phase>
  <!-- Xử lý các file được generate khi compile từ test code -->
  <phase>process-test-resources</phase>
  <!-- Compile test source code -->
  <phase>test-compile</phase>
  <!-- Xử lý các file được generate khi compile từ test code -->
  <phase>process-test-classes</phase>
  <!-- Chạy test bằng unit test framework phù hợp (JUnit là một trong số đó) -->
  <phase>test</phase>
  <!-- Thực hiện mọi thao tác cần thiết để chuẩn bị trước khi package thực tế -->
  <phase>prepare-package</phase>
  <!-- Lấy code đã compile và package thành format có thể phân phối, chẳng hạn file JAR, WAR hoặc EAR -->
  <phase>package</phase>
  <!-- Thực hiện thao tác cần thiết trước integration test, chẳng hạn thiết lập environment cần thiết -->
  <phase>pre-integration-test</phase>
  <!-- Xử lý và khi cần thì deploy package vào environment để chạy integration test -->
  <phase>integration-test</phase>
  <!-- Thực hiện thao tác cần thiết sau integration test, chẳng hạn clean environment -->
  <phase>post-integration-test</phase>
  <!-- Chạy mọi check để xác minh package hợp lệ và đạt quality standard -->
  <phase>verify</phase>
  <!-- Cài package vào local repository để project khác trong local có thể dùng làm dependency -->
  <phase>install</phase>
  <!-- Copy package cuối cùng vào remote repository để chia sẻ với developer và project khác -->
  <phase>deploy</phase>
</phases>
```

Theo lý thuyết dependency giữa các phase đã nêu, khi chạy command `mvn test`, mọi phase từ validate đến test sẽ được thực thi. Điều này giải thích vì sao code của project có thể tự động compile khi chạy test.

### `clean` lifecycle

Mục đích của clean lifecycle là clean project, gồm 3 phase:

1. pre-clean
2. clean
3. post-clean

```xml
<phases>
  <!-- Thực hiện một số công việc cần hoàn tất trước clean -->
  <phase>pre-clean</phase>
  <!-- Xóa mọi file được tạo từ build trước -->
  <phase>clean</phase>
  <!-- Thực hiện ngay một số công việc cần hoàn tất sau clean -->
  <phase>post-clean</phase>
</phases>
<default-phases>
  <clean>
    org.apache.maven.plugins:maven-clean-plugin:2.5:clean
  </clean>
</default-phases>
```

Theo lý thuyết dependency giữa các phase đã nêu, khi chạy `mvn clean`, pre-clean và clean phase trong clean lifecycle sẽ được thực thi.

### `site` lifecycle

Mục đích của site lifecycle là tạo và publish site của project, gồm 4 phase:

1. pre-site
2. site
3. post-site
4. site-deploy

```xml
<phases>
  <!-- Thực hiện một số công việc cần hoàn tất trước khi generate site documentation -->
  <phase>pre-site</phase>
  <!-- Generate site documentation của project -->
  <phase>site</phase>
  <!-- Thực hiện công việc cần hoàn tất sau khi generate site documentation và chuẩn bị deploy -->
  <phase>post-site</phase>
  <!-- Deploy site documentation đã generate lên server cụ thể -->
  <phase>site-deploy</phase>
</phases>
<default-phases>
  <site>
    org.apache.maven.plugins:maven-site-plugin:3.3:site
  </site>
  <site-deploy>
    org.apache.maven.plugins:maven-site-plugin:3.3:deploy
  </site-deploy>
</default-phases>
```

Maven có thể tự động generate một site thân thiện dựa trên thông tin trong `pom.xml`, thuận tiện cho team trao đổi và publish project information.

## Maven plugins

Về bản chất, Maven là một plugin execution framework; mọi execution process đều do từng plugin độc lập hoàn thành. Các command chúng ta thường dùng như install, clean, deploy thực chất đều là các Maven plugin riêng lẻ ở tầng dưới. Có thể tham khảo tài liệu chính thức về core plugin của Maven tại <https://maven.apache.org/plugins/index.html>.

Plugin path mặc định trong local: `${user.home}/.m2/repository/org/apache/maven/plugins`

![](https://oss.javaguide.cn/github/javaguide/tools/maven/maven-plugins.png)

Ngoài plugin đi kèm Maven, còn có một số plugin do bên thứ ba cung cấp, chẳng hạn plugin test coverage jacoco-maven-plugin, plugin maven-checkstyle-plugin giúp developer phát hiện phần code không đúng quy chuẩn, và plugin sonar-maven-plugin để phân tích code quality. Ngoài ra, bạn cũng có thể tự viết plugin để đáp ứng nhu cầu của mình.

Ví dụ sử dụng jacoco-maven-plugin:

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.jacoco</groupId>
      <artifactId>jacoco-maven-plugin</artifactId>
      <version>0.8.8</version>
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

Bạn có thể hiểu Maven plugin là một tập hợp task. User có thể chạy task của plugin được chỉ định trực tiếp từ command line, hoặc mount plugin task vào build lifecycle để nó chạy cùng lifecycle.

Maven plugin được chia thành hai loại:

- **Build plugins**: chạy trong quá trình build.
- **Reporting plugins**: chạy trong quá trình generate site.

## Maven multi-module management

Nói đơn giản, multi-module management là chia một project thành nhiều module, mỗi module chỉ phụ trách một chức năng riêng. Biểu hiện trực quan là một Maven project có nhiều hơn một file `pom.xml`, với nhiều file `pom.xml` trong các directory khác nhau để thực hiện multi-module management.

Ngoài việc giúp phát triển và quản lý project thuận tiện hơn, multi-module management còn có các lợi ích sau:

1. Giảm coupling giữa code (từ coupling ở cấp class nâng lên coupling ở cấp jar package).
2. Giảm lặp lại, nâng cao khả năng reuse.
3. Mỗi module đều có thể tự mô tả (thông qua module name hoặc module documentation).
4. Module cũng chuẩn hóa việc phân chia code boundary, developer dễ dàng xác định phần mình phụ trách thông qua module.

Trong multi-module management sẽ có một parent module, các module khác đều là child module. Parent module thường chỉ có một `pom.xml` và không có nội dung khác. `pom.xml` của parent module thường chỉ định version của từng dependency, các child module cần include và plugin cần dùng. Tuy nhiên, cần chú ý rằng nếu dependency chỉ được sử dụng trong một child project thì có thể import trực tiếp trong pom.xml của child project để tránh parent pom quá cồng kềnh.

Như hình dưới đây, Dubbo project được chia thành nhiều child module như dubbo-common (common logic module), dubbo-remoting (remote communication module), dubbo-rpc (remote call module).

![](https://oss.javaguide.cn/github/javaguide/tools/maven/dubbo-maven-multi-module.png)

## Bài viết đề xuất

- [Security classmate explains Maven arbitration mechanism in indirect dependency scenarios - Alibaba Developer - 2022](https://mp.weixin.qq.com/s/flniMiP-eu3JSBnswfd_Ew)
- [Sử dụng Java build tool hiệu quả | Maven - Alibaba Developer - 2022](https://mp.weixin.qq.com/s/Wvq7t2FC58jaCh4UFJ6GGQ)
- [Câu chuyện repackage Maven của security classmate - Alibaba Developer - 2022](https://mp.weixin.qq.com/s/xsJkB0onUkakrVH0wejcIg)

## Tài liệu tham khảo

- 《Maven thực chiến》
- Introduction to Repositories - Tài liệu chính thức của Maven: <https://maven.apache.org/guides/introduction/introduction-to-repositories.html>
- Introduction to the Build Lifecycle - Tài liệu chính thức của Maven: <https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Lifecycle_Reference>
- Maven dependency scope: <https://www.mvnbook.com/maven-dependency.html>
- Giải quyết Maven dependency conflict, bài này là đủ!: <https://www.cnblogs.com/qdhxhz/p/16363532.html>
- Multi-Module Project with Maven: <https://www.baeldung.com/maven-multi-module>

<!-- @include: @article-footer.snippet.md -->
