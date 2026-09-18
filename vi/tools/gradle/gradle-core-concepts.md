---
title: Tổng hợp các khái niệm cốt lõi của Gradle
description: Gradle là một công cụ build tự động chạy trên JVM, hỗ trợ điều phối task linh hoạt, quản lý dependency, mở rộng bằng plugin và build nhiều project.
category: Công cụ phát triển
head:
  - - meta
    - name: keywords
      content: Gradle,Groovy,Gradle Wrapper,Gradle Wrapper,Gradle Plugin
---

> Nội dung này chủ yếu được tổng hợp từ tài liệu chính thức của Gradle, có lược bớt tương ứng, chỉ giữ lại những phần tương đối quan trọng, không đề cập thực hành mà chủ yếu giới thiệu một số khái niệm quan trọng.

Nội dung về Gradle là phần tùy chọn, bạn có thể quyết định có học hay không tùy theo nhu cầu. Trong các project Java backend trong nước, Maven vẫn phổ biến hơn, nhưng Gradle cũng được sử dụng nhiều trong Android, một số project Spring Boot và các project cần tùy biến cao quy trình build.

## Giới thiệu Gradle

Tài liệu chính thức của Gradle giới thiệu về Gradle như sau:

> Gradle is an open-source [build automation](https://en.wikipedia.org/wiki/Build_automation) tool flexible enough to build almost any type of software. Gradle makes few assumptions about what you’re trying to build or how to build it. This makes Gradle particularly flexible.
>
> Gradle là một công cụ build automation mã nguồn mở, đủ linh hoạt để build gần như mọi loại phần mềm. Gradle đưa ra rất ít giả định về thứ bạn muốn build hoặc cách build nó. Điều này khiến Gradle đặc biệt linh hoạt.

Nói đơn giản, Gradle là một công cụ build project tự động chạy trên JVM, dùng để hỗ trợ chúng ta hoàn thành các task build như compile, test, đóng gói và phát hành.

Đối với developer, Gradle chủ yếu có 3 tác dụng:

1. **Build project**: cung cấp cách build project tự động tiêu chuẩn, đa nền tảng.
2. **Quản lý dependency**: quản lý tài nguyên dependency của project (gói jar) thuận tiện, nhanh chóng, tránh xung đột version giữa các tài nguyên.
3. **Thống nhất cấu trúc development**: cung cấp cấu trúc project tiêu chuẩn, thống nhất.

Build script của Gradle có thể được viết bằng Groovy DSL hoặc Kotlin DSL. Hiện nay Kotlin DSL cũng rất phổ biến trong các project mới, thường có gợi ý kiểu và hỗ trợ IDE tốt hơn.

## Giới thiệu Groovy

Gradle là một chương trình chạy trên JVM, build script có thể được viết bằng Groovy hoặc Kotlin. Trong lịch sử, nhiều ví dụ Gradle sử dụng Groovy DSL, vì vậy biết một chút cú pháp Groovy trước sẽ rất hữu ích khi đọc các project cũ.

Groovy là một scripting language chạy trên JVM, là ngôn ngữ dynamic được mở rộng dựa trên Java, có cú pháp rất giống Java và có thể sử dụng thư viện Java. Groovy có thể được dùng cho lập trình hướng đối tượng hoặc làm một scripting language thuần túy. Về thiết kế ngôn ngữ, Groovy tiếp thu các đặc điểm ưu việt của Java, Python, Ruby và Smalltalk, chẳng hạn như chuyển đổi kiểu dynamic, closure và hỗ trợ metaprogramming.

Chúng ta có thể học Groovy theo cách học Java, chi phí học tương đối thấp. Ngay cả khi quên cú pháp Groovy trong quá trình development, bạn vẫn có thể tiếp tục coding bằng cú pháp Java.

Có rất nhiều ngôn ngữ dựa trên JVM, chẳng hạn như Groovy, Kotlin, Java, Scala. Cuối cùng, chúng đều compile thành file Java bytecode và chạy trên JVM.

## Ưu điểm của Gradle

Gradle là build system thế hệ mới, có nhiều ưu điểm như hiệu quả và linh hoạt, được sử dụng rộng rãi trong Java development. Không chỉ Android chọn Gradle làm build system chính thức, ngày càng nhiều project Java như Spring Boot cũng dần migrate sang Gradle.

- Về tính linh hoạt, Gradle hỗ trợ viết script bằng ngôn ngữ Groovy, tập trung vào tính linh hoạt của quy trình build, phù hợp với các project có độ phức tạp build cao và có thể hoàn thành các quy trình build rất phức tạp.
- Về độ chi tiết, Gradle chia nhỏ build đến từng task. Hơn nữa, source code của toàn bộ Task đều là mã nguồn mở. Sau khi nắm được toàn bộ quy trình đóng gói này, chúng ta có thể thay đổi quy trình thực thi một cách dynamic bằng cách sửa Task.
- Về khả năng mở rộng, Gradle hỗ trợ cơ chế plugin, vì vậy chúng ta có thể tái sử dụng các plugin này dễ dàng, giống như tái sử dụng library.

## Giới thiệu Gradle Wrapper

Tài liệu chính thức của Gradle giới thiệu về Gradle Wrapper như sau:

> The recommended way to execute any Gradle build is with the help of the Gradle Wrapper (in short just “Wrapper”). The Wrapper is a script that invokes a declared version of Gradle, downloading it beforehand if necessary. As a result, developers can get up and running with a Gradle project quickly without having to follow manual installation processes saving your company time and money.
>
> Cách được khuyến nghị để thực thi mọi Gradle build là sử dụng Gradle Wrapper (gọi ngắn gọn là “Wrapper”). Wrapper là một script gọi version Gradle đã khai báo, tải version đó xuống trước nếu cần. Nhờ vậy, developer có thể nhanh chóng khởi chạy và sử dụng project Gradle mà không cần thực hiện quy trình cài đặt thủ công, giúp công ty tiết kiệm thời gian và chi phí.

Có thể hiểu Gradle Wrapper là một Wrapper bọc Gradle, để mọi phương thức Gradle build đều chạy với sự hỗ trợ của Gradle Wrapper.

Sơ đồ quy trình hoạt động của Gradle Wrapper như sau (nguồn ảnh từ [giới thiệu Gradle Wrapper trong tài liệu chính thức](https://docs.gradle.org/current/userguide/gradle_wrapper.html)):

![Quy trình hoạt động của Wrapper](https://oss.javaguide.cn/github/javaguide/csdn/efa7a0006b04051e2b84cd116c6ccdfc.png)

Toàn bộ quy trình chủ yếu gồm 3 bước sau:

1. Khi vừa tạo project, nếu version được chỉ định chưa được tải xuống, Gradle sẽ tải gói nén version tương ứng từ server Gradle;
2. Sau khi tải xong, cần giải nén trước rồi thực thi batch file;
3. Trong các lần build project tiếp theo, version Gradle đã giải nén này sẽ được tái sử dụng.

Gradle Wrapper mang lại cho chúng ta những lợi ích sau:

1. Chuẩn hóa project trên version Gradle được chỉ định, từ đó giúp build đáng tin cậy và mạnh mẽ hơn.
2. Có thể chạy project Gradle trên máy tính mà không cần cài đặt môi trường Gradle.
3. Cung cấp version Gradle mới cho những user và môi trường thực thi khác nhau (chẳng hạn IDE hoặc continuous integration server) đơn giản như thay đổi định nghĩa Wrapper.

### Tạo Gradle Wrapper

Nếu muốn tạo Gradle Wrapper lần đầu, trước tiên local phải có Gradle khả dụng. Gradle đã tích hợp Wrapper Task, chỉ cần chạy lệnh `gradle wrapper` ở thư mục root của project là có thể tạo Gradle Wrapper.

Khi thực thi lệnh `gradle wrapper`, có thể chỉ định một số tham số để điều khiển việc tạo wrapper. Cụ thể có các tham số cấu hình sau:

- `--gradle-version`: dùng để chỉ định version Gradle được sử dụng.
- `--gradle-distribution-url`: dùng để chỉ định URL tải Gradle distribution, giá trị này thường có dạng `https://services.gradle.org/distributions/gradle-${gradleVersion}-bin.zip`.
- `--gradle-distribution-sha256-sum`: dùng để chỉ định giá trị checksum SHA-256 của gói nén distribution, có thể giảm rủi ro file tải xuống bị tamper.

Sau khi thực thi lệnh `gradle wrapper`, Gradle Wrapper được tạo xong, thư mục root của project sẽ có các file sau:

```plain
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
└── gradlew.bat
```

Ý nghĩa của từng file như sau:

- `gradle-wrapper.jar`: chứa logic code runtime của Gradle.
- `gradle-wrapper.properties`: định nghĩa version Gradle và các thuộc tính hành vi của Gradle runtime.
- `gradlew`: wrapper script dùng để thực thi lệnh Gradle trên platform Linux/macOS.
- `gradlew.bat`: wrapper script dùng để thực thi lệnh Gradle trên platform Windows.

Nội dung file `gradle-wrapper.properties` như sau:

```properties
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-6.0.1-bin.zip
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
```

- `distributionBase`: thư mục cha dùng để lưu trữ sau khi Gradle được giải nén.
- `distributionPath`: thư mục con của directory do `distributionBase` chỉ định. `distributionBase+distributionPath` chính là directory cụ thể lưu Gradle sau khi giải nén.
- `distributionUrl`: địa chỉ tải gói nén của version Gradle được chỉ định.
- `zipStoreBase`: thư mục cha dùng để lưu trữ gói nén Gradle sau khi tải xuống.
- `zipStorePath`: thư mục con của directory do `zipStoreBase` chỉ định. `zipStoreBase+zipStorePath` chính là vị trí lưu gói nén Gradle.

### Cập nhật Gradle Wrapper

Có 2 cách cập nhật Gradle Wrapper:

1. Sửa trực tiếp field `distributionUrl`, sau đó thực thi lệnh Gradle.
2. Thực thi `./gradlew wrapper --gradle-version [version]`.

Lệnh sau sẽ nâng version Gradle lên 9.5.1.

```shell
./gradlew wrapper --gradle-version 9.5.1
```

Thuộc tính `distributionUrl` trong file `gradle-wrapper.properties` cũng thay đổi.

```properties
distributionUrl=https\://services.gradle.org/distributions/gradle-9.5.1-bin.zip
```

Sau khi project đã tạo Wrapper, trong quá trình build hằng ngày nên ưu tiên sử dụng `./gradlew build` thay vì trực tiếp sử dụng `gradle build` được cài trên máy local. Cách này đảm bảo các thành viên trong team và CI sử dụng cùng một version Gradle.

### Tùy chỉnh Gradle Wrapper

Gradle đã tích hợp Wrapper Task, vì vậy việc build Gradle Wrapper sẽ tạo file thuộc tính của Gradle Wrapper. File thuộc tính này có thể được thiết lập thông qua custom Wrapper Task. Ví dụ, nếu muốn thay đổi version Gradle cần tải xuống thành 9.5.1, có thể cấu hình như sau:

```groovy
tasks.wrapper {
    gradleVersion = '9.5.1'
}
```

Cũng có thể thiết lập các cấu hình như địa chỉ tải gói nén Gradle distribution và path lưu trữ local sau khi Gradle được giải nén.

```groovy
tasks.wrapper {
    gradleVersion = '9.5.1'
    distributionUrl = '../../gradle-9.5.1-bin.zip'
    distributionPath = 'wrapper/dists'
}
```

Thuộc tính `distributionUrl` có thể được đặt thành directory project local hoặc một địa chỉ network.

## Gradle Task

Trong Gradle, task (Task) là một đơn vị công việc riêng lẻ được thực thi trong quá trình build.

Gradle build dựa trên Task. Khi chạy project, thực chất là đang thực thi một loạt Task, chẳng hạn Task compile Java source code và Task tạo file jar.

Cách khai báo Task như sau (ngoài ra còn một số cách khai báo khác):

```groovy
// Khai báo một Task có tên helloTask
task helloTask{
     doLast{
       println "Hello"
     }
}
```

Sau khi tạo một Task, có thể thêm các Action khác nhau cho Task tùy theo nhu cầu. “doLast” ở trên là thêm một Action vào cuối queue.

```groovy
 // Thêm Action vào đầu queue Action
 Task doFirst(Action<? super Task> action);
 Task doFirst(Closure action);

 // Thêm Action vào cuối queue Action
 Task doLast(Action<? super Task> action);
 Task doLast(Closure action);

 // Xóa tất cả Action
 Task deleteAllActions();
```

Một Task có thể có nhiều Acton, các Acton được thực thi từ đầu queue đến cuối queue.

Action đại diện cho từng function, method; mỗi Task là một execution graph được tạo thành theo thứ tự từ một nhóm Action.

Keyword khai báo dependency của Task là `dependsOn`, hỗ trợ khai báo một hoặc nhiều dependency:

```groovy
task first {
 doLast {
        println "+++++first+++++"
    }
}
task second {
 doLast {
        println "+++++second+++++"
    }
}

// Chỉ định nhiều task dependency
task print(dependsOn :[second,first]) {
 doLast {
      logger.quiet "Chỉ định nhiều task dependency"
    }
}

// Chỉ định một task dependency
task third(dependsOn : print) {
 doLast {
      println '+++++third+++++'
    }
}
```

Trước khi thực thi Task, các Task dependency của nó sẽ được thực thi trước.

Chúng ta cũng có thể thiết lập default Task; ngay cả khi không gọi default Task trong script, nó vẫn được thực thi.

```groovy
defaultTasks 'clean', 'run'

task clean {
    doLast {
        println 'Default Cleaning!'
    }
}

task run {
    doLast {
        println 'Default Running!'
    }
}
```

Gradle cũng tích hợp sẵn nhiều Task, chẳng hạn copy (copy file) và delete (xóa file).

```groovy
task deleteFile(type: Delete) {
    delete "C:\\Users\\guide\\Desktop\\test"
}
```

## Gradle Plugin

Gradle cung cấp một bộ cơ chế build cốt lõi, còn Gradle plugin là một số logic build cụ thể chạy trên cơ chế này, về bản chất giống với file `.gradle`. Có thể xem Gradle plugin là công cụ đóng gói một loạt Task và thực thi chúng.

Gradle plugin chủ yếu được chia thành 2 loại:

- Script plugin: script plugin là một file script thông thường, có thể được import vào các build script khác.
- Binary plugin / object plugin: được định nghĩa trong một plugin module riêng biệt, các module khác apply plugin thông qua Plugin ID. Vì cách này thân thiện hơn với việc publish và tái sử dụng, Gradle plugin mà chúng ta thường tiếp xúc thường ở dạng binary plugin.

Mặc dù Gradle plugin và file .gradle về bản chất không khác nhau, file `.gradle` cũng có thể cung cấp chức năng tương tự Gradle plugin. Tuy nhiên, Gradle plugin sử dụng module độc lập để đóng gói logic build, nên xét từ việc bắt đầu development, trải nghiệm tổng thể của Gradle plugin thân thiện hơn.

- **Tái sử dụng logic:** cung cấp cùng một logic để tái sử dụng cho nhiều project tương tự, giảm chi phí bảo trì lặp lại logic tương tự. Đương nhiên file `.gradle` cũng có thể tái sử dụng logic, nhưng Gradle plugin đóng gói tốt hơn;
- **Publish component:** có thể publish plugin lên Maven repository để quản lý, các project khác có thể dependency thông qua plugin ID. Đương nhiên file `.gradle` cũng có thể được đặt ở một path remote để project khác tham chiếu;
- **Cấu hình build:** Gradle plugin có thể khai báo plugin extension để expose các thuộc tính có thể cấu hình, cung cấp khả năng tùy biến. Đương nhiên file `.gradle` cũng làm được, nhưng việc triển khai sẽ phức tạp hơn.

## Vòng đời Gradle build

Vòng đời Gradle build có ba giai đoạn: **giai đoạn initialization, giai đoạn configuration** và **giai đoạn execution**.

![](https://oss.javaguide.cn/github/javaguide/csdn/dadbdf59fccd9a2ebf60a2d018541e52.png)

Giữa giai đoạn initialization và giai đoạn configuration, sau khi giai đoạn configuration kết thúc và sau khi giai đoạn execution kết thúc, chúng ta đều có thể thêm một số Hook tùy biến.

![](https://oss.javaguide.cn/github/javaguide/csdn/5c297ccc4dac83229ff3e19caee9d1d2.png)

### Giai đoạn initialization

Gradle hỗ trợ build single project và multi-project. Trong giai đoạn initialization, Gradle xác định những project nào sẽ tham gia build và tạo một [Project instance](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html) cho từng project. Về bản chất, đây là việc thực thi script `settings.gradle` để đọc xem toàn bộ project có bao nhiêu Project instance.

### Giai đoạn configuration

Trong giai đoạn configuration, Gradle parse file `build.gradle` của từng project, tạo subset task sẽ được thực thi và xác định quan hệ giữa các task để giai đoạn execution thực thi theo thứ tự, đồng thời thực hiện một số cấu hình initialization cho task.

Mỗi `build.gradle` tương ứng với một Project object. Code được thực thi trong giai đoạn configuration bao gồm các statement, closure và statement cấu hình trong Task ở `build.gradle`.

Sau khi giai đoạn configuration kết thúc, Gradle sẽ tạo một **directed acyclic graph** dựa trên quan hệ dependency của Task.

### Giai đoạn execution

Trong giai đoạn execution, Gradle thực thi subset task cần thực thi đã được tạo và cấu hình trong giai đoạn configuration.

## Tham khảo

- Tài liệu chính thức Gradle: <https://docs.gradle.org/current/userguide/userguide.html>
- Tutorial nhập môn Gradle: <https://www.imooc.com/wiki/gradlebase>
- Chỉ cần đọc bài này để nhập môn Groovy nhanh: <https://cloud.tencent.com/developer/article/1358357>
- 【Gradle】Giải thích chi tiết vòng đời Gradle: <https://juejin.cn/post/7067719629874921508>
- Hướng dẫn bạn tự tay tùy chỉnh Gradle plugin —— Series Gradle (2): <https://www.cnblogs.com/pengxurui/p/16281537.html>
- Hướng dẫn tránh các vấn đề thường gặp khi dùng Gradle -- tìm hiểu Plugin, Task, quy trình build: <https://juejin.cn/post/6889090530593112077>

<!-- @include: @article-footer.snippet.md -->
