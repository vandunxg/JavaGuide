---
title: Tổng hợp các khái niệm cốt lõi của Docker
description: Hệ thống hóa các khái niệm cốt lõi của Docker và sự khác biệt giữa container với máy ảo, nắm được mối quan hệ giữa image, container và repository cũng như giá trị thực tế của chúng trong delivery và deploy.
category: Công cụ phát triển
tag:
  - Docker
head:
  - - meta
    - name: keywords
      content: Docker,container,image,repository,engine,isolation,so sánh với máy ảo,deploy
---

Bài viết này chủ yếu trình bày các khái niệm cốt lõi, mô hình vận hành và các trường hợp sử dụng thường gặp của Docker, không đi sâu vào quá trình cài đặt. Cài đặt, thực hành command và khởi động service local có thể xem trong bài [Thực hành Docker](./docker-in-action.md) ở phần sau.

## Giới thiệu về container

Docker là một nền tảng container phần mềm phổ biến. Muốn hiểu Docker, trước hết cần hiểu container thực sự giải quyết vấn đề gì.

### Container là gì?

#### Hãy xem cách giải thích tương đối chính thức về container

**Tóm tắt trong một câu: container là đơn vị tiêu chuẩn hóa dùng để đóng gói phần mềm, phục vụ development, delivery và deploy.**

- **Container image là một software package độc lập, nhẹ và có thể thực thi**, chứa mọi thứ cần thiết để phần mềm chạy: code, runtime environment, system tool, system library và setting.
- **Phần mềm container hóa phù hợp với các ứng dụng dựa trên Linux và Windows, có thể luôn chạy nhất quán trong mọi environment.**
- **Container mang lại tính độc lập cho phần mềm**, giúp phần mềm không bị ảnh hưởng bởi khác biệt của environment bên ngoài (ví dụ khác biệt giữa environment development và staging), từ đó giúp giảm xung đột khi các team chạy những phần mềm khác nhau trên cùng infrastructure.

#### Hãy xem cách giải thích dễ hiểu hơn về container

Nếu cần mô tả container theo cách dễ hiểu, tôi cho rằng container giống như một nơi chứa đồ, như balo có thể đựng nhiều loại văn phòng phẩm, tủ quần áo có thể chứa nhiều loại quần áo, giá giày có thể để nhiều loại giày. Container mà chúng ta nói đến hiện nay có thể thiên về việc chứa các application như website, program, thậm chí cả system environment.

![Tìm hiểu về container](https://oss.javaguide.cn/github/javaguide/tools/docker/container.png)

### Minh họa physical machine, virtual machine và container

Phần sau sẽ giới thiệu chi tiết về sự khác biệt giữa virtual machine và container. Ở đây chỉ thông qua hình ảnh trên Internet để giúp mọi người hiểu sâu hơn về physical machine, virtual machine và container (hình ảnh dưới đây có nguồn từ Internet).

**Physical machine:**

![Physical machine](https://oss.javaguide.cn/github/javaguide/tools/docker/%E7%89%A9%E7%90%86%E6%9C%BA%E5%9B%BE%E8%A7%A3.jpeg)

**Virtual machine:**

![Virtual machine](https://oss.javaguide.cn/github/javaguide/tools/docker/%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%9B%BE%E8%A7%A3.jpeg)

**Container:**

![](https://oss.javaguide.cn/javaguide/image-20211110104003678.png)

Thông qua ba hình minh họa trừu tượng trên, có thể khái quát bằng phép so sánh rằng: **container virtualize operating system chứ không phải hardware, các container dùng chung một bộ tài nguyên operating system. Công nghệ virtual machine virtualize một bộ hardware, sau đó chạy một operating system hoàn chỉnh trên đó. Vì vậy, mức độ isolation của container thấp hơn một chút.**

### Container VS virtual machine

Mỗi khi nhắc đến container, chúng ta không thể không so sánh nó với virtual machine. Theo tôi, không có chuyện bên nào sẽ thay thế bên nào, mà hai bên có thể cùng tồn tại hài hòa.

Nói đơn giản: **container và virtual machine có ưu thế isolation và phân bổ resource tương tự nhau, nhưng chức năng khác nhau, vì container virtualize operating system chứ không phải hardware, nên container dễ port hơn và hiệu suất cũng cao hơn.**

Công nghệ virtual machine truyền thống virtualize một bộ hardware, sau đó chạy một operating system hoàn chỉnh trên đó, rồi chạy các application process cần thiết trên system này; còn application process trong container chạy trực tiếp trên kernel của host, container không có kernel riêng và cũng không thực hiện hardware virtualization. Vì vậy container nhẹ hơn virtual machine truyền thống.

![](https://oss.javaguide.cn/javaguide/2e2b95eebf60b6d03f6c1476f4d7c697.png)

**So sánh container và virtual machine**:

![](https://oss.javaguide.cn/javaguide/4ef8691d67eb1eb53217099d0a691eb5.png)

- Container là một application-layer abstraction, dùng để đóng gói code và dependency resource cùng nhau. Nhiều container có thể chạy trên cùng một machine, dùng chung operating system kernel, nhưng mỗi container chạy như một process độc lập trong user space. So với virtual machine, **container chiếm ít không gian hơn** (container image thường chỉ có kích thước vài chục MB), **có thể hoàn tất việc start ngay lập tức**.

- Virtual machine (VM) là một physical hardware-layer abstraction, dùng để biến một server thành nhiều server. Hypervisor cho phép nhiều VM chạy trên một machine. Mỗi VM chứa một bộ operating system hoàn chỉnh, một hoặc nhiều application, các binary và library resource cần thiết, vì vậy **chiếm nhiều không gian**. Hơn nữa, VM **cũng start rất chậm**.

Thông qua website chính thức của Docker, chúng ta biết được rất nhiều ưu điểm của Docker, nhưng cũng không cần hoàn toàn phủ nhận công nghệ virtual machine, vì hai bên có các trường hợp sử dụng khác nhau. **Virtual machine giỏi hơn trong việc isolation triệt để toàn bộ runtime environment**. Ví dụ, cloud service provider thường dùng công nghệ virtual machine để isolation các user khác nhau. Còn **Docker thường dùng để isolation các application khác nhau**, như frontend, backend và database.

Theo tôi, không có chuyện bên nào sẽ thay thế bên nào, mà hai bên có thể cùng tồn tại hài hòa.

![](https://oss.javaguide.cn/javaguide/056c87751b9dd7b56f4264240fe96d00.png)

## Giới thiệu Docker

### Docker là gì?

Có thể hiểu Docker từ một số góc độ sau:

- **Docker là một nền tảng container phần mềm.**
- **Docker** được phát triển bằng Go, dựa trên các capability như cgroups, namespaces và UnionFS của Linux kernel để encapsulation và isolation process, thuộc công nghệ virtualization ở tầng operating system.
- Docker có thể đóng gói application và runtime dependency vào image, giảm các vấn đề do environment development, test và deploy không nhất quán gây ra.
- User có thể dễ dàng tạo và sử dụng container, đưa application của mình vào container. Container còn có thể được version management, copy, share và modify, giống như quản lý code thông thường.

**Tư tưởng của Docker**:

- **Container vận chuyển hàng**: giống như container vận chuyển hàng trên biển, Docker container chứa application và toàn bộ dependency của nó, bảo đảm chạy theo cùng một cách trong mọi environment.
- **Standardization**: cách vận chuyển, cách storage, API interface.
- **Isolation**: mỗi Docker container chạy trong environment isolation riêng, tách biệt với host machine và các container khác.

### Đặc điểm của Docker container

- **Lightweight**: nhiều Docker container chạy trên một machine có thể dùng chung operating system kernel của machine đó; chúng có thể start nhanh và chỉ chiếm rất ít compute resource và memory resource. Image được xây dựng thông qua các filesystem layer và dùng chung một số common file. Nhờ đó có thể giảm tối đa disk usage và download image nhanh hơn.
- **Standard**: Docker container dựa trên open standard, có thể chạy trên mọi version Linux phổ biến, Microsoft Windows và mọi infrastructure, bao gồm VM, bare-metal server và cloud.
- **Secure**: isolation mà Docker cung cấp cho application không chỉ là isolation lẫn nhau, mà còn độc lập với infrastructure bên dưới. Docker mặc định cung cấp isolation mạnh nhất, vì vậy khi application gặp vấn đề thì đó chỉ là vấn đề của một container, không ảnh hưởng đến toàn bộ machine.

### Vì sao nên dùng Docker?

- Image của Docker cung cấp runtime environment hoàn chỉnh ngoài kernel, bảo đảm tính nhất quán của application runtime environment, nhờ đó không còn xuất hiện những vấn đề như “đoạn code này chạy trên machine của tôi vẫn ổn mà”; — runtime environment nhất quán
- Có thể đạt thời gian start ở mức giây, thậm chí millisecond. Tiết kiệm đáng kể thời gian development, test và deploy. — thời gian start nhanh hơn
- Tránh dùng chung server, resource sẽ ít bị ảnh hưởng bởi user khác. — isolation
- Giỏi xử lý áp lực sử dụng server bùng phát tập trung; — elastic scaling, mở rộng nhanh
- Có thể dễ dàng migrate application đang chạy trên một platform sang platform khác mà không cần lo environment thay đổi khiến application không thể chạy bình thường. — dễ migrate
- Dùng Docker có thể thực hiện continuous integration, continuous delivery và deploy thông qua việc tùy chỉnh application image. — continuous delivery và deploy

---

## Các khái niệm cơ bản của Docker

Docker có ba khái niệm cơ bản rất quan trọng: image (Image), container (Container) và repository (Repository).

Hiểu ba khái niệm này là hiểu toàn bộ lifecycle của Docker.

![](https://oss.javaguide.cn/github/javaguide/tools/docker/docker-build-run.jpeg)

### Image: một filesystem đặc biệt

**Operating system được chia thành kernel và user space**. Với Linux, sau khi kernel start, nó mount root filesystem để cung cấp hỗ trợ cho user space. Docker image (Image) tương đương với một root filesystem.

**Docker image là một filesystem đặc biệt, ngoài việc cung cấp các file như program, library, resource và configuration cần cho container runtime, nó còn chứa một số configuration parameter được chuẩn bị cho runtime (như anonymous volume, environment variable, user).** Image không chứa dynamic data nào, nội dung của nó cũng không thay đổi sau khi build.

Khi thiết kế Docker, công nghệ **Union FS** được tận dụng đầy đủ để thiết kế image theo **kiến trúc layered storage**. Image thực tế được hợp thành từ nhiều filesystem layer.

**Khi build image, các layer được build từng lớp, layer trước là base của layer sau. Sau khi build xong mỗi layer, layer đó sẽ không thay đổi nữa, mọi thay đổi ở layer sau chỉ xảy ra trong chính layer đó.** Ví dụ, thao tác xóa file của layer trước thực tế không xóa file ở layer trước, mà chỉ đánh dấu file đó đã bị xóa trong layer hiện tại. Khi container cuối cùng chạy, dù không nhìn thấy file này, thực tế file vẫn luôn đi theo image. Vì vậy, khi build image, cần đặc biệt cẩn thận; mỗi layer nên chỉ chứa những thứ cần thêm vào layer đó, mọi thứ dư thừa cần được dọn sạch trước khi hoàn tất việc build layer.

Đặc điểm của layered storage còn khiến việc reuse và customize image dễ dàng hơn. Thậm chí có thể dùng image đã build trước đó làm base layer, sau đó tiếp tục thêm layer mới để customize nội dung cần thiết và build image mới.

### Container: entity tại runtime của image

Mối quan hệ giữa image (Image) và container (Container) giống như class và instance trong object-oriented programming: image là definition tĩnh, **container là entity của image tại runtime. Container có thể được create, start, stop, delete, pause**.

**Bản chất của container là process, nhưng khác với process thực thi trực tiếp trên host, container process chạy trong một namespace độc lập thuộc về chính nó. Như đã nói ở trên, image dùng layered storage, container cũng vậy.**

**Lifecycle của container storage layer giống với container; khi container bị hủy, container storage layer cũng bị hủy theo. Vì vậy, mọi thông tin lưu trong container storage layer sẽ mất theo khi container bị delete.**

Theo yêu cầu của Docker best practice, **container không nên ghi business data vào storage layer của nó**, container storage layer nên giữ stateless hết mức có thể. **File cần persistence nên được ghi vào data volume (Volume) hoặc bind host directory**; các thao tác đọc ghi này sẽ bypass container storage layer và ghi trực tiếp xuống host machine hoặc network storage, có performance và stability tốt hơn. Lifecycle của data volume độc lập với container; sau khi container bị delete, data volume sẽ không tự động bị delete.

### Repository: nơi lưu trữ tập trung image file

Sau khi build image, có thể dễ dàng chạy nó trên host hiện tại. Tuy nhiên, **nếu cần dùng image này trên server khác, chúng ta cần một service lưu trữ và phân phối image tập trung; Docker Registry chính là một service như vậy.**

Một Docker Registry có thể chứa nhiều repository (Repository); mỗi repository có thể chứa nhiều tag (Tag); mỗi tag tương ứng với một image. Vì vậy: **image repository là nơi Docker dùng để lưu trữ tập trung image file, tương tự code repository mà chúng ta thường dùng trước đây.**

Thông thường, **một repository sẽ chứa image của nhiều version khác nhau của cùng một software**, còn **tag thường dùng để tương ứng với từng version của software**. Có thể chỉ định image cụ thể theo format `<tên repository>:<tag>`. Nếu không cung cấp tag, `latest` sẽ được dùng làm tag mặc định. Tuy nhiên trong production environment không nên phụ thuộc vào `latest`, tốt nhất nên chỉ định rõ version tag để thuận tiện rollback và troubleshoot.

**Bổ sung về khái niệm public service của Docker Registry và private Docker Registry:**

**Public service của Docker Registry** là service Registry mở cho user sử dụng và cho phép user quản lý image. Thông thường các public service này cho phép user upload và download image public miễn phí, đồng thời có thể cung cấp service tính phí để user quản lý private image.

Public service Registry được sử dụng phổ biến nhất là **Docker Hub** chính thức, đây cũng là Registry mặc định và có rất nhiều official image chất lượng cao. Địa chỉ website là: [https://hub.docker.com/](https://hub.docker.com/ "https://hub.docker.com/"). Docker Hub được giới thiệu chính thức như sau:

> Docker Hub là một service do Docker chính thức cung cấp, dùng để tìm kiếm và chia sẻ container image với team của bạn.

Ví dụ, khi muốn search image cần dùng:

![Search image bằng Docker Hub](https://oss.javaguide.cn/github/javaguide/tools/docker/Screen%20Shot%202019-11-04%20at%208.21.39%20PM.png)

Trong kết quả search của Docker Hub, có một số thông tin quan trọng giúp chọn image phù hợp:

- **OFFICIAL Image**: cho biết image do Docker chính thức cung cấp và maintain, tương đối ổn định và an toàn hơn.
- **Stars**: có ý nghĩa gần giống lượt like, tương tự Star của GitHub.
- **Downloads**: cho biết số lần image được pull, về cơ bản có thể thể hiện tần suất image được sử dụng.

Ngoài việc trực tiếp search image qua website Docker Hub, cũng có thể dùng command `docker search` để search image trong Docker Hub, kết quả search là như nhau.

```bash
➜  ~ docker search mysql
NAME                              DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   8763                [OK]
mariadb                           MariaDB is a community-developed fork of MyS…   3073                [OK]
mysql/mysql-server                Optimized MySQL Server Docker images. Create…   650                                     [OK]
```

Khi truy cập **Docker Hub** trong nước, tốc độ có thể khá chậm. Project doanh nghiệp thường kết hợp image repository nội bộ của công ty hoặc image repository của cloud provider để cache và phân phối image.

Ngoài việc sử dụng public service, user còn có thể **tự dựng private Docker Registry trên local**. Docker chính thức cung cấp Docker Registry image, có thể dùng trực tiếp làm private Registry service. Open-source Docker Registry image chỉ cung cấp server-side implementation của Docker Registry API, đủ để hỗ trợ Docker command và không ảnh hưởng đến việc sử dụng. Tuy nhiên, nó không bao gồm GUI và các chức năng nâng cao như image maintenance, user management và access control.

### Mối quan hệ giữa Image, Container và Repository

Hình dưới đây thể hiện rất trực quan mối quan hệ giữa Image, Container, Repository và Registry/Hub:

![Kiến trúc Docker](https://oss.javaguide.cn/github/javaguide/tools/docker/docker-regitstry.png)

- Dockerfile là một text file, chứa một loạt instruction và parameter, dùng để định nghĩa cách build Docker image. Khi chạy command `docker build` và chỉ định một Dockerfile, Docker sẽ đọc các instruction trong Dockerfile, từng bước build image mới và lưu nó trên local.
- Command `docker pull` có thể download image từ Registry/Hub được chỉ định về local, mặc định sử dụng Docker Hub.
- Command `docker run` có thể create một container mới từ local image và start nó. Nếu local không có image, Docker sẽ thử pull image từ Registry/Hub trước.
- Command `docker push` có thể upload Docker image local lên Registry/Hub được chỉ định.

Các Docker command cơ bản nêu trên sẽ được giới thiệu chi tiết trong bài thực hành ở phần sau.

### Build Ship and Run

Các khái niệm của Docker về cơ bản đã được trình bày xong, tiếp theo hãy nói về: Build, Ship, and Run.

Nếu search website chính thức của Docker, sẽ thấy dòng chữ sau: **“Docker - Build, Ship, and Run Any App, Anywhere”**. Vậy Build, Ship, and Run rốt cuộc đang làm gì?

![](https://oss.javaguide.cn/github/javaguide/tools/docker/docker-build-ship-run.jpg)

- **Build (build image)**: image giống như container vận chuyển hàng, bao gồm file, runtime environment và các resource khác.
- **Ship (vận chuyển image)**: vận chuyển giữa host và repository, trong đó repository giống như một siêu cảng.
- **Run (chạy image)**: image đang chạy chính là container, container là nơi program chạy.

Quá trình Docker chạy cũng chính là lấy image từ repository về local, sau đó dùng một command để chạy image và biến nó thành container. Vì vậy, Docker cũng thường được gọi là docker worker hoặc docker handler, tương tự cách dịch tiếng Trung của Docker là người bốc dỡ vận chuyển.

## Các command thường dùng của Docker

### Command cơ bản

```bash
docker version # Xem version Docker
docker images # Xem toàn bộ image đã download, tương đương command: docker image ls
docker container ls # Xem toàn bộ container
docker ps # Xem container đang chạy
docker image prune # Dọn image file không được sử dụng. -a/--all sẽ xóa tất cả image không được container sử dụng
```

### Pull image

Registry được command `docker pull` sử dụng mặc định là Docker Hub. Khi thực thi command `docker pull` mà không chỉ định địa chỉ Registry, Docker sẽ pull image từ Docker Hub.

```bash
docker search mysql # Xem image liên quan đến MySQL
docker pull mysql:8.4 # Pull MySQL image
docker image ls # Xem toàn bộ image đã download
```

### Build image

Khi chạy command `docker build` và chỉ định một Dockerfile, Docker sẽ đọc instruction trong Dockerfile, từng bước build image mới và lưu nó trên local.

```bash
# image-name là tên image, 1.0.0 là version hoặc tag của image
docker build -t image-name:1.0.0 .
```

Cần lưu ý: tên file của Dockerfile không nhất thiết phải là Dockerfile và cũng không nhất thiết phải nằm trong root directory của build context. Dùng option `-f` hoặc `--file` có thể chỉ định bất kỳ file nào ở bất kỳ vị trí nào làm Dockerfile. Tuy nhiên, thông thường mọi người quen dùng tên mặc định `Dockerfile` và đặt nó trong build context directory của image.

### Xóa image

Ví dụ, ta muốn xóa MySQL image đã download.

Trước khi xóa image bằng `docker rmi [image]` (tương đương `docker image rm [image]`), trước hết cần bảo đảm image này không bị container reference. Có thể xóa theo tag name hoặc image ID, cũng có thể dùng command `docker ps` đã nói ở trên để xem có container nào đang sử dụng nó hay không.

```shell
➜  ~ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
c4cd691d9f80        mysql:5.7           "docker-entrypoint.s…"   7 weeks ago         Up 12 days          0.0.0.0:3306->3306/tcp, 33060/tcp   mysql
```

Có thể thấy `mysql:5.7` đang được container có ID `c4cd691d9f80` reference, cần dùng `docker stop c4cd691d9f80` hoặc `docker stop mysql` để pause container này trước.

Sau đó xem ID của MySQL image:

```shell
➜  ~ docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
mysql                   5.7                 f6509bac4980        3 months ago        373MB
```

Có thể xóa bằng `IMAGE ID` hoặc `REPOSITORY:TAG`:

```shell
docker rmi f6509bac4980 # hoặc docker rmi mysql:5.7
```

### Push image

Command `docker push` dùng để upload Docker image local lên Registry/Hub được chỉ định.

```bash
# Push image lên private image repository Harbor
# harbor.example.com là địa chỉ private image repository, ubuntu là tên image, 18.04 là version tag của image
docker push harbor.example.com/ubuntu:18.04
```

Trước khi push image, cần bảo đảm Docker image cần push đã được build trên local. Ngoài ra, nhất định phải login vào image repository tương ứng trước.

## Data management của Docker

Có hai cách chính để quản lý data trong container:

1. Data volume (Volumes)
2. Mount host directory (Bind mounts)

![Data management của Docker](https://oss.javaguide.cn/github/javaguide/tools/docker/docker-data-management.png)

Data volume là data storage area do Docker quản lý, có các đặc điểm sau:

- Có thể share và reuse giữa các container.
- Dù container bị delete, data trong data volume cũng không tự động bị delete, qua đó bảo đảm data persistence.
- Thay đổi đối với data volume có hiệu lực ngay lập tức.
- Update đối với data volume không ảnh hưởng đến image.

```bash
# Tạo một data volume
docker volume create my-vol
# Xem toàn bộ data volume
docker volume ls
# Xem thông tin cụ thể của data volume
docker volume inspect my-vol
# Xóa data volume được chỉ định
docker volume rm my-vol
```

Khi dùng command `docker run`, sử dụng flag `--mount` để mount một hoặc nhiều data volume vào container.

Cũng có thể dùng flag `--mount` để mount file hoặc directory trên host machine vào container, giúp container truy cập trực tiếp filesystem của host machine. Quyền mặc định khi Docker mount host directory là read-write; user cũng có thể thêm `readonly` để chỉ định read-only.

## Docker Compose

### Docker Compose là gì? Dùng để làm gì?

Docker Compose là tool định nghĩa và vận hành multi-container application do Docker chính thức cung cấp. Thông qua Compose, developer có thể dùng một YAML file để mô tả nhiều service, network, port và data volume mà application phụ thuộc, sau đó dùng một command để start hoặc stop cả nhóm service.

Docker Compose là một open-source project, địa chỉ: <https://github.com/docker/compose>.

Các chức năng cốt lõi của Docker Compose:

- **Multi-container management**: cho phép user định nghĩa và quản lý nhiều container trong một YAML file.
- **Service orchestration**: cấu hình network và dependency giữa các container.
- **Start và stop bằng một command**: dễ dàng start và stop toàn bộ application thông qua các command như `docker compose up` và `docker compose down`.

Docker Compose đơn giản hóa quá trình development, test và deploy multi-container application, nâng cao productivity của development team, đồng thời giảm complexity và management cost của application deploy.

### Cấu trúc cơ bản của Docker Compose file

Docker Compose file là core của Docker Compose tool, dùng để định nghĩa và cấu hình multi-container Docker application. File này thường có tên `compose.yaml` hoặc `docker-compose.yml`, được viết theo format YAML (YAML Ain't Markup Language).

Cấu trúc cơ bản của Docker Compose file như sau:

- **Service (services):** định nghĩa từng container (service) trong application. Mỗi service có thể dùng image, environment setting và dependency khác nhau.
  - **Image (image):** start container từ image được chỉ định, có thể là storage repository, tag hoặc image ID.
  - **Command (command):** tùy chọn, override command mặc định được thực thi sau khi container start. Chạy command hoặc script cụ thể khi start service, thường dùng để start application hoặc thực thi initialization script.
  - **Port (ports):** tùy chọn, mapping port của container và host machine.
  - **Dependency (depends_on):** cấu hình dependency start giữa các service. Ví dụ khi backend service phụ thuộc database service, có thể start database trước rồi mới start backend.
  - **Environment variable (environment):** tùy chọn, thiết lập environment variable cần cho service runtime.
  - **Restart (restart):** tùy chọn, kiểm soát restart policy của container. Khi container exit, tự động restart theo policy được chỉ định.
  - **Service volume (volumes):** tùy chọn, định nghĩa volume mà service sử dụng, dùng cho data persistence hoặc share data giữa các container.
  - **Build (build):** chỉ định build context path của Dockerfile để build image hoặc dùng một configuration object chi tiết.
- **Network (networks):** định nghĩa network connection giữa các container.
- **Volume (volumes):** định nghĩa data volume dùng cho data persistence và sharing. Thường dùng để persistence data như database storage, configuration file và log.

```yaml
services:
  service-name-1:
    image: nginx:stable
    command: ["nginx", "-g", "daemon off;"]
    environment:
      TZ: Asia/Shanghai
    volumes:
      - web_data:/usr/share/nginx/html
    networks:
      - app_net
    ports:
      - "8080:80"
    restart: unless-stopped
    depends_on:
      - service-name-2
  service-name-2:
    image: redis:7
    networks:
      - app_net

volumes:
  web_data:

networks:
  app_net:
```

### Các command thường dùng của Docker Compose

#### Start

`docker compose up` sẽ tạo và start container theo service được định nghĩa trong Compose file, đồng thời connect chúng vào network do Compose tạo. Nếu file không declare custom network, Compose sẽ tự động tạo default network.

```bash
# Tìm file compose.yaml hoặc docker-compose.yml trong directory hiện tại và start application theo các service được định nghĩa trong đó
docker compose up
# Start ở background
docker compose up -d
# Force recreate toàn bộ container, ngay cả khi chúng đã tồn tại
docker compose up --force-recreate
# Rebuild image
docker compose up --build
# Chỉ định tên service cần start thay vì start tất cả service
# Có thể chỉ định nhiều service cùng lúc, phân tách bằng space.
docker compose up service-name
```

Ngoài ra, nếu tên Compose file không phải `compose.yaml` hoặc `docker-compose.yml`, có thể chỉ định bằng parameter `-f`.

```bash
docker compose -f compose.prod.yaml up
```

#### Stop

`docker compose down` dùng để stop và remove container, network được start thông qua `docker compose up`.

```bash
# Tìm Compose file trong directory hiện tại
# Remove container và network đã start theo định nghĩa trong file
docker compose down
# Stop container nhưng không remove
docker compose stop
# Stop service được chỉ định
docker compose stop service-name
```

Tương tự, nếu tên Compose file không phải `compose.yaml` hoặc `docker-compose.yml`, có thể chỉ định bằng parameter `-f`.

```bash
docker compose -f compose.prod.yaml down
```

#### Xem

`docker compose ps` dùng để xem thông tin status của toàn bộ container được start thông qua `docker compose up`.

```bash
# Xem thông tin status của toàn bộ container
docker compose ps
# Chỉ hiển thị tên service
docker compose ps --services
# Xem container của service được chỉ định
docker compose ps service-name
```

#### Khác

| Command                  | Giới thiệu                                   |
| ------------------------ | -------------------------------------------- |
| `docker compose version` | Xem version                                  |
| `docker compose images`  | Liệt kê image được toàn bộ container sử dụng |
| `docker compose kill`    | Force stop container của service             |
| `docker compose exec`    | Execute command trong container              |
| `docker compose logs`    | Xem log                                      |
| `docker compose pause`   | Pause service                                |
| `docker compose unpause` | Resume service                               |
| `docker compose push`    | Push service image                           |
| `docker compose start`   | Start service hiện đang stop                 |
| `docker compose stop`    | Stop service hiện đang chạy                  |
| `docker compose rm`      | Xóa service container đã stop                |
| `docker compose top`     | Xem process                                  |

## Nguyên lý bên trong Docker

Trước hết, Docker là phần mềm dựa trên công nghệ lightweight virtualization. Vậy công nghệ virtualization là gì?

Nói đơn giản, có thể định nghĩa công nghệ virtualization như sau:

> Công nghệ virtualization là một công nghệ quản lý resource, trừu tượng hóa và chuyển đổi các [physical resource](https://zh.wikipedia.org/wiki/計算機科學) khác nhau của computer ([CPU](https://zh.wikipedia.org/wiki/CPU), [memory](https://zh.wikipedia.org/wiki/内存), [disk space](https://zh.wikipedia.org/wiki/磁盘空间), [network adapter](https://zh.wikipedia.org/wiki/網路適配器), v.v.), sau đó hiển thị chúng và cho phép phân chia, kết hợp thành một hoặc nhiều environment cấu hình computer. Qua đó phá vỡ rào cản không thể phân chia giữa các cấu trúc physical, giúp user sử dụng hardware resource của computer theo cách tốt hơn so với configuration ban đầu. Các phần virtualization mới của resource này không bị giới hạn bởi cách thiết lập, vị trí địa lý hoặc configuration physical của resource hiện có. Resource virtualization thường được nhắc đến bao gồm compute capability và data storage.

Công nghệ Docker dựa trên công nghệ LXC (Linux container - Linux container) virtual container.

> LXC, tên gọi bắt nguồn từ viết tắt của Linux software container (Linux Containers), là công nghệ virtualization ở tầng operating system (Operating system–level virtualization), một user-space interface cho chức năng container của Linux kernel. Nó đóng gói software system của application thành một software container (Container), bên trong chứa chính code của application software cùng kernel và library của operating system cần thiết. Thông qua namespace thống nhất và API dùng chung để phân bổ hardware resource có thể sử dụng cho các software container khác nhau, tạo ra environment sandbox độc lập cho application chạy, giúp Linux user dễ dàng create và manage system hoặc application container.

Công nghệ LXC chủ yếu dựa vào chức năng CGroup và namespace do Linux kernel cung cấp để thực hiện; thông qua LXC có thể cung cấp cho software một operating system runtime environment độc lập.

**Giới thiệu cgroup và namespace:**

- **namespace là cách Linux kernel dùng để isolation kernel resource.** Thông qua namespace, một số process chỉ có thể nhìn thấy một phần resource liên quan đến chúng, trong khi process khác cũng chỉ nhìn thấy resource liên quan đến chính chúng; hai nhóm process này hoàn toàn không cảm nhận được sự tồn tại của nhau. Cách implement cụ thể là chỉ định resource liên quan của một hoặc nhiều process vào cùng một namespace. Linux namespaces là sự encapsulation và isolation đối với global system resource, khiến các process ở những namespace khác nhau có global system resource độc lập; thay đổi system resource trong một namespace chỉ ảnh hưởng đến process trong namespace hiện tại, không ảnh hưởng đến process trong namespace khác.

  (Nội dung giới thiệu namespace ở trên lấy từ <https://www.cnblogs.com/sparkdev/p/9365405.html>, có thể xem thêm nội dung về namespace trong bài viết này).

- **CGroup là viết tắt của Control Groups, là một cơ chế do Linux kernel cung cấp để limit, record và isolation physical resource (như cpu, memory, i/o, v.v.) mà process group (process groups) sử dụng.**

  (Nội dung giới thiệu CGroup ở trên lấy từ <https://www.ibm.com/developerworks/cn/linux/1506_cgroup/index.html>, có thể xem thêm nội dung về CGroup trong bài viết này).

**So sánh cgroup và namespace:**

Cả hai đều group process, nhưng vai trò của chúng về bản chất vẫn khác nhau. namespace nhằm isolation resource giữa các process group, còn cgroup nhằm monitor và limit resource một cách thống nhất cho một process group.

## Tổng kết

Bài viết này trình bày chi tiết một số khái niệm và command thường gặp trong Docker. Từ zero đến thực hành có thể xem bài viết [Docker từ nhập môn đến thực hành](https://javaguide.cn/tools/docker/docker-in-action.html), nội dung rất chi tiết!

Ngoài ra, xin giới thiệu thêm một cuốn sách open-source có chất lượng rất cao là ["Docker từ nhập môn đến thực hành"](https://yeasy.gitbook.io/docker_practice/introduction/why), nội dung cuốn sách rất mới; dù sao nội dung sách là open-source nên có thể được cải thiện bất cứ lúc nào.

![Homepage website "Docker từ nhập môn đến thực hành"](https://oss.javaguide.cn/github/javaguide/tools/docker/docker-getting-started-practice-website-homepage.png)

## Tài liệu tham khảo

- [Docker Compose: Hướng dẫn toàn diện từ nền tảng đến ứng dụng thực tế](https://juejin.cn/post/7306756690727747610)
- [Linux Namespace và Cgroup](https://segmentfault.com/a/1190000009732550)
- [LXC vs Docker: Why Docker is Better](https://www.upguard.com/articles/docker-vs-lxc "LXC vs Docker: Why Docker is Better")
- [Giới thiệu CGroup, ví dụ ứng dụng và mô tả nguyên lý](https://www.ibm.com/developerworks/cn/linux/1506_cgroup/index.html)

<!-- @include: @article-footer.snippet.md -->
