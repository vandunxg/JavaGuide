---
title: Docker thực chiến
description: Hiểu image và quản lý container của Docker qua thực hành, giải quyết vấn đề không nhất quán môi trường và hiệu quả bàn giao, nâng cao hiệu quả phối hợp giữa phát triển, kiểm thử và deploy.
category: Công cụ phát triển
tag:
  - Docker
head:
  - - meta
    - name: keywords
      content: Docker thực chiến,build image,quản lý container,tính nhất quán môi trường,deploy,performance
---

## Giới thiệu Docker

Trước khi bắt đầu, hãy cùng ôn lại ngắn gọn về Docker. Bạn có thể xem bài viết trước [Tổng hợp khái niệm cốt lõi của Docker](./docker-intro.md) để tìm hiểu đầy đủ hơn về các khái niệm.

### Docker là gì?

Có thể hiểu Docker từ các góc độ sau:

- Docker là một nền tảng software container phổ biến, được phát triển bằng ngôn ngữ Go.
- Docker có thể đóng gói ứng dụng và các dependency khi chạy vào image, giảm các vấn đề do môi trường phát triển, kiểm thử và deploy không nhất quán gây ra.
- Người dùng có thể dễ dàng tạo và sử dụng container, đưa ứng dụng của mình vào container. Container cũng có thể được quản lý version, sao chép, chia sẻ và chỉnh sửa giống như quản lý code thông thường.
- Docker có thể **đóng gói và cô lập process, thuộc công nghệ virtual hóa ở tầng operating system.** Vì process bị cô lập độc lập với host và các process bị cô lập khác nên chúng còn được gọi là container.

Địa chỉ website chính thức: <https://www.docker.com/> .

![Tìm hiểu container](https://oss.javaguide.cn/github/javaguide/tools/docker/container.png)

### Tại sao nên dùng Docker?

Docker cho phép developer đóng gói ứng dụng cùng các package dependency vào một container nhẹ và portable, sau đó deploy lên mọi máy Linux phổ biến, đồng thời cũng có thể thực hiện virtual hóa.

Container hoàn toàn sử dụng cơ chế sandbox, giữa chúng không có bất kỳ interface nào (tương tự app trên iPhone), quan trọng hơn là overhead performance của container cực thấp.

Trong quy trình phát triển truyền thống, project của chúng ta thường cần các service dependency như MySQL, Redis, Kafka. Nếu tất cả môi trường này đều được cài đặt và cấu hình thủ công, thao tác trên các system khác nhau sẽ khác biệt rất lớn, đồng thời dễ xuất hiện vấn đề “máy của tôi chạy được, máy của bạn thì không”.

Sự xuất hiện của Docker đã giải quyết hoàn hảo vấn đề này. Chúng ta có thể cài đặt các môi trường software như MySQL, Redis trong container, tách ứng dụng khỏi kiến trúc môi trường. Ưu điểm của cách này:

1. Môi trường runtime nhất quán, dễ migrate hơn
2. Đóng gói và cô lập process, các container không ảnh hưởng lẫn nhau, tận dụng system resource hiệu quả hơn
3. Có thể dùng image để tạo nhiều container nhất quán

Ngoài ra, cuốn sách open source [《Docker từ nhập môn đến thực hành》](https://yeasy.gitbook.io/docker_practice/introduction/why) cũng đã nêu lý do sử dụng Docker.

![](https://oss.javaguide.cn/github/javaguide/tools/docker/20210412220015698.png)

## Cài đặt Docker

### Windows

Windows nên cài Docker Desktop. Truy cập website chính thức của Docker để tải installer:

![Cài đặt Docker](https://oss.javaguide.cn/github/javaguide/tools/docker/docker-install-windows.png)

Sau đó nhấp vào `Get Started`:

![Cài đặt Docker](https://oss.javaguide.cn/github/javaguide/tools/docker/docker-install-windows-download.png)

Nhấp vào `Download for Windows` tại đây để tải xuống.

Hiện tại Docker Desktop for Windows khuyến nghị sử dụng backend WSL 2. Trước khi cài đặt, nên xác nhận system đáp ứng yêu cầu version của Docker Desktop và đã bật WSL 2. Trong một số trường hợp cũng có thể sử dụng backend Hyper-V, cách bật như sau. Mở Control Panel, chọn Programs:

![Bật Hyper-V](https://oss.javaguide.cn/github/javaguide/tools/docker/docker-windows-hyperv.png)

Nhấp vào `Turn Windows features on or off`:

![Bật Hyper-V](https://oss.javaguide.cn/github/javaguide/tools/docker/docker-windows-hyperv-enable.png)

Chọn `Hyper-V`, rồi nhấp OK:

![Bật Hyper-V](https://oss.javaguide.cn/github/javaguide/tools/docker/docker-windows-hyperv-check.png)

Sau khi hoàn tất thay đổi, cần restart computer.

Sau khi bật `Hyper-V`, có thể cài Docker Desktop. Mở installer, chờ một lát rồi nhấp `Ok`:

![Cài đặt Docker](https://oss.javaguide.cn/github/javaguide/tools/docker/docker-windows-hyperv-install.png)

Sau khi cài đặt xong, vẫn cần restart computer. Sau khi restart, nếu xuất hiện nội dung sau:

![Cài đặt Docker](https://oss.javaguide.cn/github/javaguide/tools/docker/docker-windows-hyperv-wsl2.png)

Nếu trong quá trình cài đặt được nhắc sử dụng WSL 2, thông thường nên ưu tiên chọn backend WSL 2. Đây là cách phổ biến hơn để chạy Linux container trên Windows; nếu môi trường của bạn bắt buộc phải dùng Hyper-V thì chuyển sang backend Hyper-V.

Vì đây là thao tác trên giao diện đồ họa nên ở đây không giới thiệu cụ thể cách sử dụng Docker Desktop.

### macOS

Chỉ cần dùng Homebrew để cài đặt:

```shell
brew install --cask docker
```

### Linux

Hãy xem cách cài đặt Docker trên Linux. Command cài đặt có khác biệt đôi chút giữa các distribution; trong môi trường production, nên ưu tiên tham khảo tài liệu chính thức của Docker. Ở đây dùng official install script để minh họa cách cài đặt nhanh trong môi trường test hoặc development.

Trong môi trường test hoặc development, để đơn giản hóa quy trình cài đặt, Docker cung cấp một install script tiện lợi. Sau khi chạy script này, mọi công việc chuẩn bị sẽ được tự động hoàn tất và version stable của Docker được cài đặt vào system.

```shell
curl -fsSL get.docker.com -o get-docker.sh
```

```shell
sh get-docker.sh --mirror Aliyun
```

Sau khi cài đặt xong, khởi động service:

```shell
systemctl start docker
```

Khuyến nghị bật tự khởi động cùng system, thực hiện command:

```shell
systemctl enable docker
```

## Một số khái niệm trong Docker

Trước khi chính thức học Docker, chúng ta cần hiểu một số khái niệm cốt lõi trong Docker:

### Image

Image là một template chỉ đọc. Image có thể được dùng để tạo Docker container; một image có thể tạo nhiều container.

### Container

Container là runtime instance được tạo từ image. Docker sử dụng container để chạy độc lập một hoặc một nhóm application. Container có thể được start, bắt đầu, stop và delete; mỗi container là một platform được cô lập và bảo đảm an toàn. Có thể xem container như một môi trường Linux đơn giản cùng application chạy bên trong. Định nghĩa của container gần như giống hệt image, cũng là một góc nhìn thống nhất của một tập hợp layer; điểm khác biệt duy nhất là layer trên cùng của container có thể đọc và ghi.

### Repository

Repository là nơi lưu trữ tập trung các image. Repository khác với registry server; registry server thường lưu trữ nhiều repository, mỗi repository lại chứa nhiều image, mỗi image có các tag khác nhau. Repository được chia thành public repository và private repository. Public repository phổ biến nhất là Docker Hub, nơi lưu trữ một lượng lớn image có thể tải trực tiếp.

### Tổng kết

Nói đơn giản, một image đại diện cho một software; chạy dựa trên một image sẽ tạo ra một program instance, và program instance này chính là container; repository dùng để lưu trữ tất cả image trong Docker.

Repository lại được chia thành remote repository và local repository, tương tự Maven. Nếu lần nào cũng download dependency từ remote thì hiệu quả sẽ giảm đáng kể. Vì vậy, Maven sẽ download dependency vào local repository khi truy cập lần đầu, lần thứ hai và thứ ba chỉ cần dùng dependency trong local repository. Remote repository và local repository của Docker cũng có tác dụng tương tự.

## Trải nghiệm Docker đầu tiên

Tiếp theo, chúng ta sẽ sử dụng Docker ở mức cơ bản, với ví dụ download một MySQL image.

Giống như GitHub, Docker cũng cung cấp Docker Hub để tra cứu địa chỉ và hướng dẫn sử dụng các image khác nhau. Trước tiên hãy truy cập Docker Hub: [https://hub.docker.com/](https://hub.docker.com/)

![Docker Hub](https://oss.javaguide.cn/github/javaguide/tools/docker/dockerhub-com.png)

Nhập `mysql` vào ô tìm kiếm ở góc trên bên trái rồi nhấn Enter:

![Tìm kiếm MySQL trên Docker Hub](https://oss.javaguide.cn/github/javaguide/tools/docker/dockerhub-mysql.png)

Có thể thấy có rất nhiều MySQL image liên quan. Nếu góc trên bên phải có nhãn `OFFICIAL IMAGE` thì đó là official image, vì vậy chúng ta nhấp vào MySQL image đầu tiên:

![MySQL official image](https://oss.javaguide.cn/github/javaguide/tools/docker/dockerhub-mysql-official-image.png)

Bên phải cung cấp command download MySQL image là `docker pull mysql`, nhưng command này sẽ pull version tương ứng với tag mặc định. Trong project thực tế, nên chỉ định rõ version tag để tránh môi trường không thể kiểm soát.

Nếu muốn download image của một version cụ thể, hãy nhấp vào `View Available Tags` bên dưới:

![Xem các version MySQL khác](https://oss.javaguide.cn/github/javaguide/tools/docker/dockerhub-mysql-view-available-tags.png)

Tại đây có thể xem image của nhiều version khác nhau, bên phải có command download. Ví dụ, nếu muốn download MySQL image version 8.4, có thể thực hiện:

```shell
docker pull mysql:8.4
```

Trong một số môi trường mạng, việc pull image từ Docker Hub có thể chậm hoặc thất bại. Tuy nhiên, không nên sao chép trực tiếp các địa chỉ mirror tăng tốc của bên thứ ba trên Internet: phạm vi áp dụng, chính sách đồng bộ và tính khả dụng của các service này có thể thay đổi bất cứ lúc nào.

Lấy dịch vụ container image ACR của Alibaba Cloud làm ví dụ: accelerator của dịch vụ này từ ngày 2 tháng 7 năm 2024 chỉ dành cho người dùng Alibaba Cloud sử dụng trên các sản phẩm Alibaba Cloud hỗ trợ truy cập public network, đồng thời chỉ hỗ trợ pull container image trong phạm vi giới hạn. Tài liệu chính thức hiện tại cũng cho biết service này đã dừng đồng bộ image mới nhất; máy không thuộc Alibaba Cloud truy cập địa chỉ accelerator sẽ nhận HTTP 403. Hạn chế cụ thể hãy căn cứ vào [thông báo điều chỉnh chức năng ACR image accelerator của Alibaba Cloud](https://help.aliyun.com/zh/acr/product-overview/product-change-acr-mirror-accelerator-function-adjustment-announcement) và [tài liệu chính thức về image accelerator](https://help.aliyun.com/zh/acr/user-guide/accelerate-the-pulls-of-docker-official-images).

Nếu sử dụng Alibaba Cloud ECS, có thể cấu hình theo địa chỉ riêng và tài liệu chính thức do console Alibaba Cloud cung cấp. Trong môi trường không thuộc Alibaba Cloud, không nên sao chép nguyên cấu hình trên; trong production, nên giảm sự phụ thuộc mạnh vào public image service bên ngoài, đồng bộ image cần thiết vào private repository tự xây dựng hoặc do cloud vendor cung cấp, đồng thời cố định version hoặc digest của image.

## Command image của Docker

Docker cần thường xuyên thao tác với các image liên quan, vì vậy trước tiên hãy tìm hiểu các command image trong Docker.

Nếu muốn xem Docker hiện có những image nào, có thể dùng command `docker images`.

```shell
[root@izrcf5u3j3q8xaz ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
mysql         8.4       f07dfa83b528   11 days ago     448MB
tomcat        latest    feba8d001e3f   2 weeks ago     649MB
nginx         latest    ae2feff98a0c   2 weeks ago     133MB
hello-world   latest    bf756fb1ae65   12 months ago   13.3kB
```

Trong đó, `REPOSITORY` là tên image, `TAG` là version marker, `IMAGE ID` là id của image (duy nhất), `CREATED` là thời gian tạo. Lưu ý thời gian này không phải thời gian chúng ta download image vào Docker, mà là thời gian image được creator tạo ra; `SIZE` là kích thước image.

Command này có thể query theo tên image cụ thể:

```shell
docker images mysql
```

Khi đó, tất cả MySQL image trong Docker sẽ được query:

```shell
[root@izrcf5u3j3q8xaz ~]# docker images mysql
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
mysql        8.4       0ebb5600241d   11 days ago     589MB
mysql        8.0       f07dfa83b528   11 days ago     596MB
```

Command này cũng có thể nhận tham số `-q`: `docker images -q`; `-q` nghĩa là chỉ hiển thị id của image:

```shell
[root@izrcf5u3j3q8xaz ~]# docker images -q
0ebb5600241d
f07dfa83b528
feba8d001e3f
d404d78aa797
```

Nếu muốn download image, sử dụng:

```shell
docker pull mysql:8.4
```

`docker pull` là command cố định, phía sau ghi tên image và version tag cần download; nếu không ghi version tag mà thực hiện trực tiếp `docker pull mysql`, Docker sẽ pull version tương ứng với tag mặc định.

Thông thường, trước khi download image, cần search xem image có những version nào để download đúng version. Sử dụng command:

```shell
docker search mysql
```

![](https://oss.javaguide.cn/github/javaguide/tools/docker/docker-search-mysql-terminal.png)

Tuy nhiên, `docker search` chỉ có thể search image repository, không thể liệt kê toàn bộ tag của một image. Để xem MySQL hỗ trợ những version nào, nên truy cập trực tiếp trang Tags của Docker Hub.

```shell
docker pull mysql:8.4
```

Nếu tag không tồn tại, khi thực hiện `docker pull` sẽ trả về lỗi tương tự `manifest unknown`:

![](https://oss.javaguide.cn/github/javaguide/tools/docker/docker-search-mysql-404-terminal.png)

Xóa image bằng command:

```shell
docker image rm mysql:8.4
```

Nếu không chỉ định version thì mặc định cũng sẽ xóa version mới nhất.

Cũng có thể xóa bằng cách chỉ định image id:

```shell
docker image rm bf756fb1ae65
```

Tuy nhiên lúc này sẽ xảy ra lỗi:

```shell
[root@izrcf5u3j3q8xaz ~]# docker image rm bf756fb1ae65
Error response from daemon: conflict: unable to delete bf756fb1ae65 (must be forced) - image is being used by stopped container d5b6c177c151
```

Nguyên nhân là image `hello-world` cần xóa đang được sử dụng bởi một container đã stop, nên không thể xóa image; lúc này cần bắt buộc thực hiện xóa:

```shell
docker image rm -f bf756fb1ae65
```

Command này sẽ xóa cả image và tất cả container được chạy bằng image đó, hãy thận trọng khi sử dụng.

Docker cũng cung cấp phiên bản rút gọn để xóa image: `docker rmi image-name:version-tag`.

Lúc này có thể kết hợp `rmi` và `-q` để thực hiện một số thao tác. Ví dụ, nếu muốn xóa tất cả MySQL image, cần query ID của MySQL image rồi thực hiện `docker rmi` lần lượt theo từng ID. Cũng có thể làm như sau:

```shell
docker rmi -f $(docker images mysql -q)
```

Trước tiên dùng `docker images mysql -q` để query tất cả image ID của MySQL; `-q` nghĩa là chỉ query ID, sau đó truyền các ID này làm tham số cho command `docker rmi -f`, như vậy tất cả MySQL image đều sẽ bị xóa.

## Command container của Docker

Sau khi nắm được các command liên quan đến image, chúng ta cần tìm hiểu các command container, vì container được xây dựng dựa trên image.

Nếu cần chạy một container từ image, sử dụng:

```shell
docker run tomcat:8.0-jre8
```

Tất nhiên, điều kiện để chạy là bạn phải có image này, vì vậy hãy download image trước:

```shell
docker pull tomcat:8.0-jre8
```

Sau khi download xong là có thể chạy. Sau khi chạy, xem các container hiện đang chạy bằng `docker ps`.

![](https://oss.javaguide.cn/github/javaguide/tools/docker/docker-ps-terminal.png)

Trong đó, `CONTAINER_ID` là id của container, `IMAGE` là tên image, `COMMAND` là command được thực thi trong container, `CREATED` là thời gian tạo container, `STATUS` là trạng thái container, `PORTS` là port mà service trong container listen, `NAMES` là tên container.

Tomcat chạy theo cách này không thể được truy cập trực tiếp từ bên ngoài vì container có tính cô lập. Nếu muốn truy cập trực tiếp Tomcat bên trong container qua port 8080, cần map port của host với port bên trong container:

```shell
docker run -p 8080:8080 tomcat:8.0-jre8
```

Giải thích tác dụng của hai port này (`8080:8080`): 8080 đầu tiên là port của host, 8080 thứ hai là port trong container. Truy cập port 8080 từ bên ngoài sẽ thông qua mapping để truy cập port 8080 trong container.

Lúc này bên ngoài có thể truy cập Tomcat:

![](https://oss.javaguide.cn/github/javaguide/tools/docker/docker-run-tomact-8080.png)

Nếu mapping như sau:

```shell
docker run -p 8088:8080 tomcat:8.0-jre8
```

Thì bên ngoài phải truy cập port 8088 mới có thể truy cập tomcat. Cần lưu ý mỗi container được chạy là độc lập với nhau, vì vậy chạy đồng thời nhiều tomcat container sẽ không gây xung đột port.

Container cũng có thể chạy ở background để không chiếm terminal:

```shell
docker run -d -p 8080:8080 tomcat:8.0-jre8
```

Khi start container, mặc định container sẽ được gán một tên, nhưng tên này có thể được thiết lập bằng command:

```shell
docker run -d -p 8080:8080 --name tomcat01 tomcat:8.0-jre8
```

Khi đó tên container là tomcat01, tên container phải là duy nhất.

Tiếp tục tìm hiểu một số tham số command trong `docker ps`, chẳng hạn `-a`:

```shell
docker ps -a
```

Tham số này sẽ liệt kê toàn bộ container đang chạy và không chạy.

Tham số `-q` chỉ query id của các container đang chạy: `docker ps -q`.

```shell
[root@izrcf5u3j3q8xaz ~]# docker ps -q
f3aac8ee94a3
074bf575249b
1d557472a708
4421848ba294
```

Nếu kết hợp sử dụng, sẽ query id của tất cả container đang chạy và không chạy: `docker ps -qa`.

```shell
[root@izrcf5u3j3q8xaz ~]# docker ps -aq
f3aac8ee94a3
7f7b0e80c841
074bf575249b
a1e830bddc4c
1d557472a708
4421848ba294
b0440c0a219a
c2f5d78c5d1a
5831d1bab2a6
d5b6c177c151
```

Tiếp theo là các command stop và restart container. Vì rất đơn giản nên không giới thiệu quá nhiều.

```shell
docker start c2f5d78c5d1a
```

Command này có thể start lại container đã stop; có thể start bằng id hoặc tên container.

```shell
docker restart c2f5d78c5d1a
```

Command này có thể restart container được chỉ định.

```shell
docker stop c2f5d78c5d1a
```

Command này có thể stop container được chỉ định.

```shell
docker kill c2f5d78c5d1a
```

Command này có thể kill trực tiếp container được chỉ định.

Tất cả command trên đều có thể sử dụng với cả id và tên container.

---

Sau khi container bị stop, dù container không còn chạy nhưng vẫn tồn tại. Nếu muốn xóa nó, sử dụng command:

```shell
docker rm d5b6c177c151
```

Cần lưu ý không cần viết toàn bộ id container, chỉ cần phần có thể xác định duy nhất.

Nếu muốn xóa container đang chạy, cần thêm tham số `-f` để bắt buộc xóa:

```shell
docker rm -f d5b6c177c151
```

Nếu muốn xóa tất cả container, có thể dùng command kết hợp:

```shell
docker rm -f $(docker ps -qa)
```

Trước tiên query id của tất cả container bằng `docker ps -qa`, sau đó xóa bằng `docker rm -f`.

---

Khi container chạy ở background, chúng ta không thể biết trạng thái chạy của container. Nếu cần xem runtime log của container, sử dụng command:

```shell
docker logs 289cc00dc5ed
```

Log hiển thị theo cách này không phải realtime. Nếu muốn hiển thị realtime, cần dùng tham số `-f`:

```shell
docker logs -f 289cc00dc5ed
```

Tham số `-t` còn có thể hiển thị timestamp của log, thường được dùng kết hợp với tham số `-f`:

```shell
docker logs -ft 289cc00dc5ed
```

---

Để xem trong container đang chạy những process nào, có thể dùng command:

```shell
docker top 289cc00dc5ed
```

Nếu muốn tương tác với container, sử dụng command:

```shell
docker exec -it 289cc00dc5ed bash
```

Lúc này terminal sẽ đi vào bên trong container, các command thực thi đều có hiệu lực trong container. Trong container chỉ có thể thực hiện một số command đơn giản như `ls`, `cd`; nếu muốn thoát terminal của container và quay lại CentOS, thực hiện `exit`.

Bây giờ chúng ta đã có thể vào terminal của container để thực hiện các thao tác liên quan, vậy làm thế nào để deploy một project vào tomcat container?

```shell
docker cp ./test.html 289cc00dc5ed:/usr/local/tomcat/webapps
```

Command `docker cp` có thể copy file từ CentOS vào container. `./test.html` là resource path trên CentOS, `289cc00dc5ed` là container id, `/usr/local/tomcat/webapps` là resource path trong container. Khi đó file `test.html` sẽ được copy vào path này.

```shell
[root@izrcf5u3j3q8xaz ~]# docker exec -it 289cc00dc5ed bash
root@289cc00dc5ed:/usr/local/tomcat# cd webapps
root@289cc00dc5ed:/usr/local/tomcat/webapps# ls
test.html
root@289cc00dc5ed:/usr/local/tomcat/webapps#
```

Nếu muốn copy file từ trong container ra CentOS thì chỉ cần viết ngược lại:

```shell
docker cp 289cc00dc5ed:/usr/local/tomcat/webapps/test.html ./
```

Vì vậy, nếu muốn deploy project, trước tiên upload project lên CentOS, sau đó copy project từ CentOS vào container, rồi start container.

---

Mặc dù dùng Docker để start software environment rất đơn giản, nhưng đồng thời cũng có một vấn đề: chúng ta không thể biết chi tiết bên trong container, chẳng hạn port đang listen, địa chỉ IP được bind... May mắn là Docker đã tính đến điều này, chỉ cần dùng command:

```shell
docker inspect 923c969b0d91
```

![](https://oss.javaguide.cn/github/javaguide/tools/docker/docker-inspect-terminal.png)

## Docker volume

Sau khi học các command container, hãy tìm hiểu volume trong Docker. Volume có thể thực hiện chia sẻ file giữa host và container. Ưu điểm là việc chỉnh sửa file trên host sẽ ảnh hưởng trực tiếp đến container mà không cần copy lại file từ host vào container.

Nếu muốn tạo volume giữa thư mục `/opt/apps` trên host và thư mục `webapps` trong container, cần viết command như sau:

```shell
docker run -d -p 8080:8080 --name tomcat01 -v /opt/apps:/usr/local/tomcat/webapps tomcat:8.0-jre8
```

Tuy nhiên lúc này truy cập tomcat sẽ phát hiện không thể truy cập:

![](https://oss.javaguide.cn/github/javaguide/tools/docker/docker-data-volume-webapp-8080.png)

Điều này cho thấy volume đã được thiết lập thành công. Docker sẽ đồng bộ thư mục `webapps` trong container với thư mục `/opt/apps`; lúc này `/opt/apps` đang rỗng, khiến thư mục `webapps` cũng trở thành thư mục rỗng nên không thể truy cập.

Lúc này chỉ cần thêm file vào thư mục `/opt/apps` thì thư mục `webapps` cũng sẽ có các file tương ứng, đạt được chia sẻ file. Hãy thử:

```shell
[root@centos-7 opt]# cd apps/
[root@centos-7 apps]# vim test.html
[root@centos-7 apps]# ls
test.html
[root@centos-7 apps]# cat test.html
<h1>This is a test html!</h1>
```

Đã tạo file `test.html` trong thư mục `/opt/apps`, vậy thư mục `webapps` trong container có file này không? Hãy vào terminal của container:

```shell
[root@centos-7 apps]# docker exec -it tomcat01 bash
root@115155c08687:/usr/local/tomcat# cd webapps/
root@115155c08687:/usr/local/tomcat/webapps# ls
test.html
```

Trong container quả thực đã có file này. Tiếp theo, chúng ta viết một Web application đơn giản:

```java
public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().println("Hello World!");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}
```

Đây là một Servlet rất đơn giản. Chúng ta đóng gói rồi upload nó vào `/opt/apps`, khi đó container chắc chắn sẽ đồng bộ file này. Sau đó truy cập:

![](https://oss.javaguide.cn/github/javaguide/tools/docker/docker-data-volume-webapp-8080-hello-world.png)

Cách này thường được gọi là bind mount vì thư mục host do chúng ta tự chỉ định. Docker còn cung cấp một cách dùng volume phổ biến khác: named volume.

```shell
docker run -d -p 8080:8080 --name tomcat01 -v aa:/usr/local/tomcat/webapps tomcat:8.0-jre8
```

Lúc này `aa` không phải thư mục trên host mà là tên volume. Docker sẽ tự động tạo volume tên `aa` và copy nội dung có sẵn trong thư mục `webapps` của container vào volume. Mặc định, volume do Docker quản lý nằm trong thư mục `/var/lib/docker/volumes`:

```shell
[root@centos-7 volumes]# pwd
/var/lib/docker/volumes
[root@centos-7 volumes]# cd aa/
[root@centos-7 aa]# ls
_data
[root@centos-7 aa]# cd _data/
[root@centos-7 _data]# ls
docs  examples  host-manager  manager  ROOT
```

Lúc này chỉ cần chỉnh sửa nội dung thư mục này là có thể ảnh hưởng đến container. Tuy nhiên, trong project thực tế không nên trực tiếp chỉnh sửa file dưới `/var/lib/docker/volumes`; hãy ưu tiên quản lý dữ liệu thông qua container, application hoặc thư mục bind mount được chỉ định rõ ràng.

---

Cuối cùng, hãy giới thiệu thêm một số command liên quan đến container và image:

```shell
docker commit -m "description" -a "image author" tomcat01 my_tomcat:1.0
```

Command này có thể đóng gói container thành image. Sau đó query image:

```shell
[root@centos-7 _data]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
my_tomcat           1.0                 79ab047fade5        2 seconds ago       463MB
tomcat              8                   a041be4a5ba5        2 weeks ago         533MB
mysql               8.4                 db2b37ec6181        2 months ago        589MB
```

Nếu muốn backup image, có thể dùng command:

```shell
docker save my_tomcat:1.0 -o my-tomcat-1.0.tar
```

```shell
[root@centos-7 ~]# docker save my_tomcat:1.0 -o my-tomcat-1.0.tar
[root@centos-7 ~]# ls
anaconda-ks.cfg  initial-setup-ks.cfg  Public  Videos  Documents  Music
get-docker.sh    my-tomcat-1.0.tar     Templates  Pictures  Downloads  Desktop
```

Nếu có image ở format `.tar`, làm thế nào để load nó vào Docker? Thực hiện command:

```shell
docker load -i my-tomcat-1.0.tar
```

```shell
root@centos-7 ~]# docker load -i my-tomcat-1.0.tar
b28ef0b6fef8: Loading layer [==================================================>]  105.5MB/105.5MB
0b703c74a09c: Loading layer [==================================================>]  23.99MB/23.99MB
......
Loaded image: my_tomcat:1.0
[root@centos-7 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
my_tomcat           1.0                 79ab047fade5        7 minutes ago       463MB
```

## Command kiểm tra thường dùng

Sau khi làm quen với Docker, command thực sự thường xuyên được dùng là command kiểm tra. Bạn nên quen thuộc với các command sau:

```shell
# Xem tham số khởi động, network, thư mục mount và environment variable của container
docker inspect tomcat01

# Xem log gần đây của container
docker logs --tail=100 tomcat01

# Liên tục xem log của container
docker logs -f tomcat01

# Xem mức sử dụng resource của container
docker stats

# Xem dung lượng disk Docker đang sử dụng
docker system df
```

Hãy cẩn thận khi dọn dẹp resource, đặc biệt với command có `-f`. `docker system prune` sẽ xóa container, network, image và build cache không được sử dụng. Nếu thêm `--volumes`, nó còn dọn cả volume không được sử dụng; database và dữ liệu test local đều có thể bị xóa.

<!-- @include: @article-footer.snippet.md -->
