---
title: "Hướng dẫn thực hành ZooKeeper: triển khai Docker, lệnh zkCli, lệnh bốn ký tự và client Curator"
category: Distributed
description: "Hướng dẫn thực hành ZooKeeper, bao quát cài đặt và triển khai bằng Docker, các lệnh zkCli thường dùng, lệnh bốn ký tự, các thao tác CRUD của client Curator Java và ví dụ distributed lock dựa trên ZooKeeper."
tag:
  - ZooKeeper
head:
  - - meta
    - name: keywords
      content: ZooKeeper,ZooKeeper thực hành,cài đặt ZooKeeper,zkCli,Curator,lệnh bốn ký tự,triển khai Docker,distributed lock,hướng dẫn ZooKeeper
---

Bài viết này minh họa ngắn gọn cách sử dụng các lệnh thường gặp của ZooKeeper và cách sử dụng cơ bản client Curator Java của ZooKeeper. Các nội dung được giới thiệu đều là thao tác cơ bản nhất, đáp ứng nhu cầu cơ bản trong công việc hằng ngày.

Bài viết thiên về thực hành, không giải thích vì sao ZooKeeper có thể thực hiện coordination. Bạn nên đọc [Hướng dẫn nhập môn ZooKeeper](./zookeeper-intro.md) trước để tìm hiểu ZNode, Watcher và Session, sau đó đọc [Giải thích chi tiết ZooKeeper nâng cao](./zookeeper-plus.md) hoặc [Giải thích chi tiết giao thức ZAB](../../protocol/zab.md) để bổ sung kiến thức về giao thức và cơ chế cluster.

Nếu bài viết có điểm nào cần cải thiện hoặc hoàn thiện, hãy góp ý trong phần bình luận để cùng tiến bộ!

## Cài đặt ZooKeeper

### Cài đặt zookeeper bằng Docker

**a. Tải ZooKeeper bằng Docker**

```shell
docker pull zookeeper:3.5.8
```

**b. Chạy ZooKeeper**

```shell
docker run -d --name zookeeper -p 2181:2181 zookeeper:3.5.8
```

### Kết nối dịch vụ ZooKeeper

**a. Vào container ZooKeeper**

Trước tiên dùng `docker ps` để xem ContainerID của ZooKeeper, sau đó dùng lệnh `docker exec -it ContainerID /bin/bash` để vào container.

**b. Vào thư mục bin, sau đó kết nối dịch vụ ZooKeeper bằng lệnh `./zkCli.sh -server 127.0.0.1:2181`**

```bash
root@eaf70fc620cb:/apache-zookeeper-3.5.8-bin# cd bin
```

Nếu thấy console in thành công thông tin dưới đây, nghĩa là bạn đã kết nối thành công đến dịch vụ ZooKeeper.

![Kết nối dịch vụ ZooKeeper](https://oss.javaguide.cn/github/javaguide/distributed-system/zookeeper/connect-zooKeeper-service.png)

## Minh họa các lệnh ZooKeeper thường dùng

### Xem các lệnh thường dùng (lệnh help)

Dùng lệnh `help` để xem các lệnh thường dùng của ZooKeeper.

### Tạo node (lệnh create)

Dùng lệnh `create` để tạo node node1 trong thư mục gốc, với chuỗi liên kết là "node1".

```shell
[zk: 127.0.0.1:2181(CONNECTED) 34] create /node1 “node1”
```

Dùng lệnh `create` để tạo node node1 trong thư mục gốc, với nội dung là số 123.

```shell
[zk: 127.0.0.1:2181(CONNECTED) 1] create /node1/node1.1 123
Created /node1/node1.1
```

### Cập nhật dữ liệu node (lệnh set)

```shell
[zk: 127.0.0.1:2181(CONNECTED) 11] set /node1 "set node1"
```

### Lấy dữ liệu node (lệnh get)

Lệnh `get` có thể lấy nội dung dữ liệu và trạng thái của node được chỉ định. Có thể thấy lệnh `set` đã đổi nội dung dữ liệu của node thành "set node1".

```shell
[zk: zookeeper(CONNECTED) 12] get -s /node1
set node1
cZxid = 0x47
ctime = Sun Jan 20 10:22:59 CST 2019
mZxid = 0x4b
mtime = Sun Jan 20 10:41:10 CST 2019
pZxid = 0x4a
cversion = 1
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 9
numChildren = 1

```

### Xem các node con trong một thư mục (lệnh ls)

Dùng lệnh `ls` để xem các node trong thư mục gốc.

```shell
[zk: 127.0.0.1:2181(CONNECTED) 37] ls /
[dubbo, ZooKeeper, node1]
```

Dùng lệnh `ls` để xem các node trong thư mục node1.

```shell
[zk: 127.0.0.1:2181(CONNECTED) 5] ls /node1
[node1.1]
```

Lệnh `ls` của ZooKeeper tương tự lệnh `ls` của Linux. Lệnh này liệt kê thông tin tất cả node con trong path tuyệt đối (chỉ liệt kê 1 cấp, không đệ quy).

### Xem trạng thái node (lệnh stat)

Dùng lệnh `stat` để xem trạng thái node.

```shell
[zk: 127.0.0.1:2181(CONNECTED) 10] stat /node1
cZxid = 0x47
ctime = Sun Jan 20 10:22:59 CST 2019
mZxid = 0x47
mtime = Sun Jan 20 10:22:59 CST 2019
pZxid = 0x4a
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 11
numChildren = 1
```

Một số thông tin hiển thị ở trên như cversion, aclVersion, numChildren đã được giới thiệu trong bài viết [Tổng hợp các khái niệm liên quan đến ZooKeeper (nhập môn)](https://javaguide.cn/distributed-system/distributed-process-coordination/zookeeper/zookeeper-intro.html).

### Xem thông tin và trạng thái node (lệnh ls2)

Lệnh `ls2` giống sự kết hợp giữa lệnh `ls` và lệnh `stat` hơn. Thông tin lệnh `ls2` trả về gồm 2 phần:

1. Danh sách node con
2. Thông tin stat của node hiện tại.

```shell
[zk: 127.0.0.1:2181(CONNECTED) 7] ls2 /node1
[node1.1]
cZxid = 0x47
ctime = Sun Jan 20 10:22:59 CST 2019
mZxid = 0x47
mtime = Sun Jan 20 10:22:59 CST 2019
pZxid = 0x4a
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 11
numChildren = 1

```

### Xóa node (lệnh delete)

Lệnh này rất đơn giản, nhưng cần lưu ý: nếu muốn xóa một node, node đó phải không có node con.

```shell
[zk: 127.0.0.1:2181(CONNECTED) 3] delete /node1/node1.1
```

Phần sau sẽ giới thiệu cách sử dụng Java client API và hai client ZooKeeper mã nguồn mở là ZkClient và Curator.

## Sử dụng cơ bản client Curator Java của ZooKeeper

Curator là một framework client ZooKeeper Java mã nguồn mở của Netflix. So với client `zookeeper` đi kèm ZooKeeper, Curator có phần đóng gói hoàn thiện hơn và nhiều API có thể được sử dụng khá thuận tiện.

![](https://oss.javaguide.cn/github/javaguide/distributed-system/zookeeper/curator.png)

Sau đây là minh họa đơn giản về cách sử dụng Curator.

Curator phiên bản 4.0+ hỗ trợ ZooKeeper 3.5.x khá tốt. Trước khi bắt đầu, hãy thêm các dependency dưới đây vào project.

```xml
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>4.2.0</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>4.2.0</version>
</dependency>
```

### Kết nối ZooKeeper client

Tạo đối tượng `CuratorFramework` bằng `CuratorFrameworkFactory`, sau đó gọi method `start()` của đối tượng `CuratorFramework`.

```java
private static final int BASE_SLEEP_TIME = 1000;
private static final int MAX_RETRIES = 3;

// Retry strategy. Retry 3 times, and will increase the sleep time between retries.
RetryPolicy retryPolicy = new ExponentialBackoffRetry(BASE_SLEEP_TIME, MAX_RETRIES);
CuratorFramework zkClient = CuratorFrameworkFactory.builder()
    // the server to connect to (can be a server list)
    .connectString("127.0.0.1:2181")
    .retryPolicy(retryPolicy)
    .build();
zkClient.start();
```

Giải thích một số tham số cơ bản:

- `baseSleepTimeMs`: thời gian chờ ban đầu giữa các lần retry
- `maxRetries`: số lần retry tối đa
- `connectString`: danh sách server cần kết nối
- `retryPolicy`: policy retry

### Thêm, xóa, sửa và truy vấn data node

#### Tạo node

Trong [Giải thích các khái niệm thường gặp của ZooKeeper](./zookeeper-intro.md), ta đã giới thiệu znode thường được chia thành 4 loại:

- **Persistent node (PERSISTENT)**: sau khi tạo sẽ luôn tồn tại, kể cả khi ZooKeeper cluster down, cho đến khi bị xóa.
- **Ephemeral node (EPHEMERAL)**: vòng đời của ephemeral node gắn với **client session**, **session biến mất thì node cũng biến mất**. Ngoài ra, **ephemeral node chỉ có thể là leaf node**, không thể tạo child node.
- **Persistent sequential node (PERSISTENT_SEQUENTIAL)**: ngoài các đặc điểm của persistent node, tên child node còn có tính tuần tự. Ví dụ `/node1/app0000000001`, `/node1/app0000000002`.
- **Ephemeral sequential node (EPHEMERAL_SEQUENTIAL)**: ngoài các đặc điểm của ephemeral node, tên child node còn có tính tuần tự.

Khi sử dụng ZooKeeper, bạn sẽ thấy class `CreateMode` thực tế có 7 loại znode, nhưng 4 loại nêu trên vẫn được dùng nhiều nhất.

**a. Tạo Persistent node**

Bạn có thể dùng hai cách dưới đây để tạo Persistent node.

```java
// Lưu ý: đoạn code dưới đây sẽ báo lỗi, nguyên nhân cụ thể được nói bên dưới
zkClient.create().forPath("/node1/00001");
zkClient.create().withMode(CreateMode.PERSISTENT).forPath("/node1/00002");
```

Tuy nhiên, khi chạy đoạn code trên sẽ báo lỗi vì node cha `node1` chưa được tạo.

Bạn có thể tạo node cha `node1` trước, sau đó chạy lại đoạn code trên.

```java
zkClient.create().forPath("/node1");
```

Cách được khuyến nghị hơn là dùng dòng code dưới đây. **`creatingParentsIfNeeded()` bảo đảm tự động tạo node cha khi node cha chưa tồn tại, rất hữu ích.**

```java
zkClient.create().creatingParentsIfNeeded().withMode(CreateMode.PERSISTENT).forPath("/node1/00001");
```

**b. Tạo Ephemeral node**

```java
zkClient.create().creatingParentsIfNeeded().withMode(CreateMode.EPHEMERAL).forPath("/node1/00001");
```

**c. Tạo node và chỉ định nội dung dữ liệu**

```java
zkClient.create().creatingParentsIfNeeded().withMode(CreateMode.EPHEMERAL).forPath("/node1/00001","java".getBytes());
zkClient.getData().forPath("/node1/00001");// Lấy nội dung dữ liệu của node, kết quả là mảng byte
```

**d. Kiểm tra tạo node thành công hay chưa**

```java
zkClient.checkExists().forPath("/node1/00001");// Nếu không phải null thì node đã được tạo thành công
```

#### Xóa node

**a. Xóa một node con**

```java
zkClient.delete().forPath("/node1/00001");
```

**b. Xóa một node và toàn bộ node con bên dưới**

```java
zkClient.delete().deletingChildrenIfNeeded().forPath("/node1");
```

#### Lấy/cập nhật nội dung dữ liệu của node

```java
zkClient.create().creatingParentsIfNeeded().withMode(CreateMode.EPHEMERAL).forPath("/node1/00001","java".getBytes());
zkClient.getData().forPath("/node1/00001");// Lấy nội dung dữ liệu của node
zkClient.setData().forPath("/node1/00001","c++".getBytes());// Cập nhật nội dung dữ liệu của node
```

#### Lấy path của tất cả node con của một node

```java
List<String> childrenPaths = zkClient.getChildren().forPath("/node1");
```

<!-- @include: @article-footer.snippet.md -->
