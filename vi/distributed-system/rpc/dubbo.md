---
title: "Tổng hợp câu hỏi phỏng vấn Dubbo: nguyên lý kiến trúc, SPI, load balancing, service governance và cluster fault tolerance"
category: Distributed system
description: Tổng hợp các câu hỏi phỏng vấn Dubbo thường gặp, bao quát nguyên lý kiến trúc Dubbo, expose và reference service, cơ chế mở rộng SPI, load balancing, cluster fault tolerance, service governance, registry và các vấn đề production thường gặp.
tag:
  - RPC
  - Dubbo
head:
  - - meta
    - name: keywords
      content: Dubbo,Dubbo câu hỏi phỏng vấn,kiến trúc Dubbo,Dubbo SPI,load balancing,cluster fault tolerance,service governance,registry,RPC framework,distributed service framework
---

::: tip

- Dubbo3 đã được phát hành, bài viết này được viết dựa trên Dubbo2. Dubbo3 phát triển từ Dubbo2; bên cạnh việc giữ lại các tính năng cốt lõi, Dubbo3 đã nâng cấp toàn diện về tính dễ sử dụng, thực tiễn microservice quy mô rất lớn, khả năng tương thích với hạ tầng cloud native và thiết kế bảo mật.
- Nhiều link trong bài viết này đã không còn hoạt động, chủ yếu do tài liệu chính thức của Dubbo đã được chỉnh sửa khiến URL bị hỏng.

:::

Bài viết này mặc định bạn đã hiểu quy trình gọi cơ bản của RPC. Nếu chưa quen với các khái niệm dynamic proxy, serialization, network transmission và service discovery, bạn nên đọc trước [Giải thích chi tiết về RPC remote procedure call](./rpc-intro.md). Dubbo liên quan đến registry, load balancing và cluster fault tolerance; bạn cũng có thể đọc phần này cùng với [chuyên đề ZooKeeper](../distributed-process-coordination/zookeeper/).

Bài viết là phần tổng hợp về Dubbo dựa trên tài liệu chính thức và quá trình sử dụng thực tế của tác giả. Hoan nghênh bổ sung!

## Kiến thức cơ bản về Dubbo

### Dubbo là gì?

![Trang web chính thức Dubbo](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/rpc/dubbo.org-overview.png)

[Apache Dubbo](https://github.com/apache/dubbo) |ˈdʌbəʊ| là một WEB và RPC framework mã nguồn mở, nhẹ và có performance cao.

Theo giới thiệu trong [tài liệu chính thức của Dubbo](https://dubbo.apache.org/zh/), Dubbo cung cấp sáu năng lực cốt lõi:

1. RPC invocation có performance cao dựa trên interface proxy.
2. Fault tolerance và load balancing thông minh.
3. Tự động đăng ký và discovery service.
4. Khả năng mở rộng cao.
5. Điều phối traffic trong runtime.
6. Service governance và vận hành trực quan.

![Sáu năng lực cốt lõi do Dubbo cung cấp](https://oss.javaguide.cn/%E6%BA%90%E7%A0%81/dubbo/dubbo%E6%8F%90%E4%BE%9B%E7%9A%84%E5%85%AD%E5%A4%A7%E6%A0%B8%E5%BF%83%E8%83%BD%E5%8A%9B.png)

Nói đơn giản: **Dubbo không chỉ giúp chúng ta gọi remote service mà còn cung cấp một số tính năng dùng ngay, chẳng hạn như intelligent load balancing.**

Hiện Dubbo đã có gần 34,4 k Star.

Trong chương trình bình chọn **open source project của Trung Quốc năm 2020**, Dubbo đứng thứ 7 trong nhóm development framework và basic component. So với vài năm trước, độ phổ biến và thứ hạng đã giảm.

![](https://oss.javaguide.cn/%E6%BA%90%E7%A0%81/dubbo/image-20210107153159545.png)

Dubbo do Alibaba open source, sau đó gia nhập Apache. Chính sự xuất hiện của Dubbo đã khiến ngày càng nhiều công ty bắt đầu sử dụng và chấp nhận distributed architecture.

### Vì sao cần dùng Dubbo?

Cùng với sự phát triển của Internet, quy mô website ngày càng lớn và số lượng người dùng ngày càng tăng. Monolithic application architecture và vertical application architecture không còn đáp ứng được nhu cầu, vì vậy distributed service architecture ra đời.

Trong distributed service architecture, hệ thống được tách thành các service khác nhau, chẳng hạn SMS service và security service; mỗi service độc lập cung cấp một core service của hệ thống.

Ta có thể dùng các framework hỗ trợ remote invocation như Java RMI (Java Remote Method Invocation) và Hessian để expose và reference remote service một cách đơn giản. Tuy nhiên, khi số lượng service tăng, quan hệ gọi giữa các service trở nên phức tạp hơn. Khi áp lực truy cập của application ngày càng lớn, nhu cầu về load balancing và service monitoring cũng trở nên cấp thiết. Ta có thể dùng phần cứng như F5 để thực hiện load balancing, nhưng cách này làm tăng chi phí và có nguy cơ single point of failure.

Tuy nhiên, sự xuất hiện của Dubbo đã giải quyết các vấn đề trên. **Dubbo giúp chúng ta giải quyết những vấn đề gì?**

1. **Load balancing**: khi cùng một service được deploy trên các máy khác nhau, cần gọi service trên máy nào.
2. **Tạo service call chain**: cùng với sự phát triển của hệ thống, số lượng service tăng lên và quan hệ phụ thuộc giữa các service trở nên phức tạp, thậm chí không thể phân biệt application nào cần khởi động trước application nào; ngay cả architect cũng không thể mô tả đầy đủ quan hệ kiến trúc của application. Dubbo có thể giúp chúng ta xác định các service gọi lẫn nhau như thế nào.
3. **Thống kê áp lực và thời gian truy cập service, resource scheduling và governance**: quản lý capacity của cluster theo thời gian thực dựa trên áp lực truy cập, nâng cao mức sử dụng cluster.
4. ……

![Tổng quan năng lực Dubbo](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/rpc/dubbo-features-overview.jpg)

Ngoài distributed system, Dubbo cũng có thể được dùng trong microservice system đang phổ biến hiện nay. Tuy nhiên, vì Spring Cloud được sử dụng rộng rãi hơn trong microservice, nên khi nói đến Dubbo, phần lớn thường là trong bối cảnh distributed system.

**Vừa rồi chúng ta đã nhắc đến khái niệm distributed. Tiếp theo, hãy cùng tìm hiểu distributed là gì và vì sao cần distributed.**

## Kiến thức cơ bản về distributed system

### Distributed system là gì?

Distributed system, hay SOA distributed system, cốt lõi là service-oriented. Nói đơn giản, distributed system là tách toàn bộ hệ thống thành các service khác nhau rồi đặt các service đó trên các server khác nhau để giảm áp lực cho monolithic service, đồng thời tăng concurrency và performance. Chẳng hạn, e-commerce system có thể được tách đơn giản thành order system, product system, login system, v.v. Sau khi tách, mỗi service có thể được deploy trên một máy khác nhau; nếu một service có lượng truy cập lớn, service đó cũng có thể được deploy đồng thời trên nhiều máy.

![Sơ đồ minh họa distributed transaction](https://oss.javaguide.cn/java-guide-blog/%E5%88%86%E5%B8%83%E5%BC%8F%E4%BA%8B%E5%8A%A1%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

### Vì sao cần distributed system?

Về mặt development, code của monolithic application tập trung cùng một chỗ, còn code của distributed system được tách theo business. Vì vậy, mỗi team có thể phụ trách development một service, qua đó nâng cao hiệu suất development. Ngoài ra, sau khi code được tách theo business, việc maintenance và mở rộng cũng thuận tiện hơn.

Ngoài việc thuận tiện cho mở rộng và maintenance, theo tôi, tách hệ thống thành distributed system còn có thể nâng cao performance của toàn hệ thống. Hãy thử nghĩ xem: tách toàn bộ hệ thống thành các service/system khác nhau rồi deploy từng service/system riêng trên một server, chẳng phải sẽ nâng cao performance hệ thống ở mức đáng kể sao?

## Kiến trúc Dubbo

### Các role cốt lõi trong kiến trúc Dubbo là gì?

[Chương thiết kế framework trong tài liệu chính thức](https://dubbo.apache.org/zh/docs/v2.7/dev/design/) đã giới thiệu rất chi tiết; ở đây chỉ nhắc lại một số điểm quan trọng.

![Các role cốt lõi trong kiến trúc Dubbo](https://oss.javaguide.cn/%E6%BA%90%E7%A0%81/dubbo/dubbo-relation.jpg)

Giới thiệu ngắn gọn về các node trên và quan hệ giữa chúng:

- **Container:** service runtime container, chịu trách nhiệm load và run service provider. Bắt buộc.
- **Provider:** phía cung cấp service được expose, đăng ký service mình cung cấp với registry. Bắt buộc.
- **Consumer:** phía consumer gọi remote service, subscribe service mình cần từ registry. Bắt buộc.
- **Registry:** registry dùng để đăng ký và discovery service. Registry trả danh sách địa chỉ service provider cho consumer. Không bắt buộc.
- **Monitor:** monitoring center thống kê số lần và thời gian invocation service. Service consumer và provider định kỳ gửi dữ liệu thống kê đến monitoring center. Không bắt buộc.

### Bạn có biết khái niệm Invoker trong Dubbo không?

`Invoker` là một khái niệm rất quan trọng trong domain model của Dubbo. Nếu từng đọc source code Dubbo, bạn sẽ thấy nó vô số lần. Chẳng hạn, trong source code của phần load balancing được trình bày dưới đây có rất nhiều `Invoker`.

Nói đơn giản, `Invoker` là abstraction của remote invocation trong Dubbo.

![dubbo_rpc_invoke.jpg](https://oss.javaguide.cn/java-guide-blog/dubbo_rpc_invoke.jpg)

Theo tài liệu chính thức của Dubbo, `Invoker` được chia thành:

- Provider `Invoker`
- Consumer `Invoker`

Giả sử cần gọi một method từ xa, ta cần dynamic proxy để che giấu các chi tiết của remote invocation. Những chi tiết được che giấu này phụ thuộc vào implementation tương ứng của `Invoker`; `Invoker` thực hiện remote service invocation thực sự.

### Bạn có biết nguyên lý hoạt động của Dubbo không?

Hình dưới đây thể hiện thiết kế tổng thể của Dubbo. Từ dưới lên trên có mười layer, mỗi layer chỉ phụ thuộc một chiều.

> Nền xanh nhạt bên trái là interface mà service consumer sử dụng; nền xanh lá nhạt bên phải là interface mà service provider sử dụng; interface nằm trên trục giữa là interface được cả hai phía sử dụng.

![dubbo-framework](https://oss.javaguide.cn/source-code/dubbo/dubbo-framework.jpg)

- **config layer**: configuration liên quan đến Dubbo. Hỗ trợ cấu hình bằng code và cấu hình dựa trên Spring, lấy `ServiceConfig`, `ReferenceConfig` làm trung tâm.
- **proxy service layer**: yếu tố quan trọng giúp gọi remote method đơn giản như gọi local method; quá trình gọi thực tế phụ thuộc vào proxy class, lấy `ServiceProxy` làm trung tâm.
- **registry layer**: đóng gói việc đăng ký và discovery service address.
- **cluster routing layer**: đóng gói routing và load balancing của nhiều provider, đồng thời bridge với registry, lấy `Invoker` làm trung tâm.
- **monitor layer**: monitoring số lần và thời gian RPC invocation, lấy `Statistics` làm trung tâm.
- **remote invocation protocol layer**: đóng gói RPC invocation, lấy `Invocation`, `Result` làm trung tâm.
- **exchange layer**: đóng gói request-response mode, chuyển từ synchronous sang asynchronous, lấy `Request`, `Response` làm trung tâm.
- **network transport layer**: abstraction mina và netty thành interface thống nhất, lấy `Message` làm trung tâm.
- **data serialization layer**: serialization dữ liệu cần truyền qua network.

### Bạn có biết cơ chế SPI của Dubbo không? Làm thế nào để mở rộng default implementation trong Dubbo?

Cơ chế SPI (Service Provider Interface) được sử dụng rộng rãi trong các open source project. Nó giúp tìm implementation của service/feature một cách động, chẳng hạn strategy load balancing.

Nguyên lý cụ thể của SPI như sau: đặt các implementation của interface vào configuration file, đọc configuration file trong lúc program chạy và load implementation class bằng reflection. Nhờ vậy, có thể thay thế implementation class của interface khi runtime. Cách này tương tự tư tưởng decoupling của IoC.

Java tự cung cấp implementation của SPI. Tuy nhiên, Dubbo không sử dụng trực tiếp mà tăng cường cơ chế SPI native của Java để đáp ứng tốt hơn nhu cầu riêng.

**Vậy làm thế nào để mở rộng default implementation trong Dubbo?**

Ví dụ, nếu muốn tự implement strategy load balancing, ta tạo implementation class tương ứng `XxxLoadBalance`, implement interface `LoadBalance` hoặc extend class `AbstractLoadBalance`.

```java
package com.xxx;

import org.apache.dubbo.rpc.cluster.LoadBalance;
import org.apache.dubbo.rpc.Invoker;
import org.apache.dubbo.rpc.Invocation;
import org.apache.dubbo.rpc.RpcException;

public class XxxLoadBalance implements LoadBalance {
    public <T> Invoker<T> select(List<Invoker<T>> invokers, Invocation invocation) throws RpcException {
        // ...
    }
}
```

Chỉ cần ghi path của implementation class này vào file `META-INF/dubbo/org.apache.dubbo.rpc.cluster.LoadBalance` trong thư mục `resources`.

```java
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxLoadBalance.java (implement interface LoadBalance)
    |-resources
        |-META-INF
            |-dubbo
                |-org.apache.dubbo.rpc.cluster.LoadBalance (file plain text, nội dung: xxx=com.xxx.XxxLoadBalance)
```

`org.apache.dubbo.rpc.cluster.LoadBalance`

```plain
xxx=com.xxx.XxxLoadBalance
```

Dubbo còn có nhiều lựa chọn mở rộng khác; bạn có thể tìm thấy trong [tài liệu chính thức](https://cn.dubbo.apache.org/zh-cn/overview/home/).

### Bạn có biết microkernel architecture của Dubbo không?

Dubbo sử dụng mô hình microkernel (Microkernel) + plugin (Plugin), nói đơn giản là microkernel architecture. Microkernel chỉ chịu trách nhiệm lắp ghép plugin.

**Microkernel architecture là gì?** Cuốn sách 《Software Architecture Patterns》 giới thiệu như sau:

> Microkernel architecture pattern (đôi khi được gọi là plugin architecture pattern) là một pattern tự nhiên để triển khai product-based application. Product-based application đã được đóng gói, có các version khác nhau và có thể được download dưới dạng plugin bên thứ ba. Nhiều công ty cũng phát triển và phát hành các business application nội bộ của mình dưới dạng application có version, mô tả và plugin có thể load (đây cũng là đặc trưng của pattern này). Microkernel system cho phép người dùng thêm application bổ sung như plugin vào core application, từ đó cung cấp khả năng mở rộng và tách biệt chức năng.

Microkernel architecture gồm hai loại component: **core system** và **plug-in modules**.

![](https://oss.javaguide.cn/source-code/dubbo/%E5%BE%AE%E5%86%85%E6%A0%B8%E6%9E%B6%E6%9E%84%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

Core system cung cấp các năng lực cốt lõi cần thiết cho system, còn plug-in modules có thể mở rộng chức năng của system. Vì vậy, system dựa trên microkernel architecture rất dễ mở rộng chức năng.

Một số IDE quen thuộc cũng có thể được xem là được thiết kế dựa trên microkernel architecture. Hầu hết IDE, chẳng hạn IDEA và VSCode, đều cung cấp plugin để làm phong phú chức năng.

Chính vì Dubbo dựa trên microkernel architecture nên có thể tùy ý thay thế các feature của Dubbo. Ví dụ, nếu không hài lòng với implementation của serialization module trong Dubbo thì chỉ cần tự implement một serialization module.

Thông thường, microkernel sẽ dùng Factory, IoC, OSGi và các cách khác để quản lý lifecycle của plugin. Dubbo không muốn phụ thuộc vào IoC container như Spring, cũng không muốn tự tạo một IoC container nhỏ (over-design), vì vậy sử dụng cách Factory đơn giản nhất để quản lý plugin: **cơ chế SPI mở rộng chuẩn của JDK** (`java.util.ServiceLoader`).

### Một số câu hỏi tự kiểm tra về kiến trúc Dubbo

#### Bạn có biết vai trò của registry không?

Registry chịu trách nhiệm đăng ký và tìm kiếm service address, tương đương directory service. Provider và consumer chỉ tương tác với registry khi khởi động.

#### Sau khi service provider bị down, registry sẽ làm gì?

Registry lập tức push event notification cho consumer.

#### Còn vai trò của monitoring center?

Monitoring center chịu trách nhiệm thống kê số lần invocation, thời gian invocation của từng service, v.v.

#### Nếu cả registry và monitoring center đều down thì service có bị down hết không?

Không. Việc cả hai cùng down không ảnh hưởng đến provider và consumer đang chạy, vì consumer đã cache local danh sách provider. Registry và monitoring center đều không bắt buộc; service consumer có thể connect trực tiếp đến service provider.

## Strategy load balancing của Dubbo

### Load balancing là gì?

Trước hết, hãy xem một cách giải thích mang tính chính thức hơn. Đoạn dưới đây trích từ định nghĩa load balancing trên Wikipedia:

> Load balancing cải thiện việc phân phối workload giữa nhiều computing resource (chẳng hạn computer, computer cluster, network link, central processing unit hoặc disk drive). Load balancing nhằm tối ưu việc sử dụng resource, tối đa hóa throughput, tối thiểu hóa response time và tránh quá tải bất kỳ resource đơn lẻ nào. Sử dụng nhiều component có load balancing thay vì một component đơn lẻ có thể nâng cao reliability và availability nhờ redundancy. Load balancing thường liên quan đến software hoặc hardware chuyên dụng.

**Có thể phần giải thích trên hơi khó hiểu, hãy diễn đạt theo cách dễ hiểu hơn.**

Giả sử một service trong system có lượng truy cập rất lớn, ta deploy service đó trên nhiều server. Khi client gửi request, nhiều server đều có thể xử lý request này. Vì vậy, việc chọn đúng server xử lý request là rất quan trọng. Nếu chỉ có một server xử lý request của service đó thì việc deploy service trên nhiều server không còn ý nghĩa. Load balancing nhằm tránh việc một server phải xử lý cùng một request, dễ dẫn đến server down hoặc crash; ý nghĩa của load balancing có thể thấy rõ ngay từ tên gọi.

### Dubbo cung cấp những strategy load balancing nào?

Khi thực hiện cluster load balancing, Dubbo cung cấp nhiều strategy, mặc định là gọi ngẫu nhiên `random`. Ta cũng có thể tự mở rộng strategy load balancing (tham khảo cơ chế SPI của Dubbo).

Trong Dubbo, mọi implementation class của load balancing đều kế thừa `AbstractLoadBalance`. Class này implement interface `LoadBalance` và đóng gói một số logic chung.

```java
public abstract class AbstractLoadBalance implements LoadBalance {

    static int calculateWarmupWeight(int uptime, int warmup, int weight) {
    }

    @Override
    public <T> Invoker<T> select(List<Invoker<T>> invokers, URL url, Invocation invocation) {
    }

    protected abstract <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation);


    int getWeight(Invoker<?> invoker, Invocation invocation) {

    }
}
```

Các implementation class của `AbstractLoadBalance` gồm:

![](https://oss.javaguide.cn/java-guide-blog/image-20210326105257812.png)

Phần giới thiệu về load balancing trong tài liệu chính thức rất chi tiết. Bạn nên xem thêm tại [https://dubbo.apache.org/zh/docs/v2.7/dev/source/loadbalance/#m-zhdocsv27devsourceloadbalance](https://dubbo.apache.org/zh/docs/v2.7/dev/source/loadbalance/#m-zhdocsv27devsourceloadbalance).

#### RandomLoadBalance

Chọn ngẫu nhiên theo weight (implementation của weighted random algorithm). Đây là strategy load balancing mặc định của Dubbo.

Nguyên lý implementation cụ thể của `RandomLoadBalance` rất đơn giản. Giả sử có hai server cung cấp cùng một service là S1 và S2, weight của S1 là 7, weight của S2 là 3.

Phân bố các weight này trên trục tọa độ, ta được: S1->[0, 7), S2->[7, 10). Ta sinh một số ngẫu nhiên trong [0, 10); nếu số ngẫu nhiên rơi vào interval nào thì chọn server tương ứng xử lý request.

![RandomLoadBalance](https://oss.javaguide.cn/java-guide-blog/%20RandomLoadBalance.png)

Source code của `RandomLoadBalance` rất đơn giản, chỉ cần dành vài phút để xem.

> Source code dưới đây lấy từ version 2.7.9 mới nhất trên branch master của Dubbo.

```java
public class RandomLoadBalance extends AbstractLoadBalance {

    public static final String NAME = "random";

    @Override
    protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {

        int length = invokers.size();
        boolean sameWeight = true;
        int[] weights = new int[length];
        int totalWeight = 0;
        // Vòng lặp for dưới đây chủ yếu tính tổng weight totalWeight của tất cả provider của service,
        // đồng thời kiểm tra weight của từng provider có giống nhau hay không.
        for (int i = 0; i < length; i++) {
            int weight = getWeight(invokers.get(i), invocation);
            totalWeight += weight;
            weights[i] = totalWeight;
            if (sameWeight && totalWeight != weight * (i + 1)) {
                sameWeight = false;
            }
        }
        if (totalWeight > 0 && !sameWeight) {
            // Sinh ngẫu nhiên một số trong interval [0, totalWeight)
            int offset = ThreadLocalRandom.current().nextInt(totalWeight);
            // Xác định số đó rơi vào interval của provider nào
            for (int i = 0; i < length; i++) {
                if (offset < weights[i]) {
                    return invokers.get(i);
                }
            }

        return invokers.get(ThreadLocalRandom.current().nextInt(length));
    }

}

```

#### LeastActiveLoadBalance

`LeastActiveLoadBalance` có thể dịch là **load balancing theo số active nhỏ nhất**.

Tên gọi này hơi khó hình dung. Nếu không xem kỹ định nghĩa chính thức về active count, bạn gần như không biết nó dùng để làm gì.

Nói đơn giản: ở trạng thái ban đầu, active count của mọi provider đều bằng 0 (mỗi method cụ thể của mỗi provider có một active count tương ứng, sẽ được đề cập trong source code phía sau). Sau mỗi request, active count của provider tương ứng tăng 1; khi request xử lý xong, active count giảm 1.

Vì vậy, **Dubbo cho rằng provider có active count càng thấp thì tốc độ xử lý càng nhanh và performance càng tốt, nên ưu tiên gửi request cho provider có active count thấp.**

**Nếu có nhiều provider có active count bằng nhau thì sao?**

Rất đơn giản, chạy lại `RandomLoadBalance`.

```java
public class LeastActiveLoadBalance extends AbstractLoadBalance {

    public static final String NAME = "leastactive";

    @Override
    protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
        int length = invokers.size();
        int leastActive = -1;
        int leastCount = 0;
        int[] leastIndexes = new int[length];
        int[] weights = new int[length];
        int totalWeight = 0;
        int firstWeight = 0;
        boolean sameWeight = true;
        // Vòng lặp for này duyệt danh sách invokers để tìm Invoker có active count nhỏ nhất.
        // Nếu nhiều Invoker có cùng active count nhỏ nhất, ghi lại index của chúng trong tập invokers,
        // cộng dồn weight và so sánh các weight có bằng nhau hay không.
        for (int i = 0; i < length; i++) {
            Invoker<T> invoker = invokers.get(i);
            // Lấy active count tương ứng với invoker.
            int active = RpcStatus.getStatus(invoker.getUrl(), invocation.getMethodName()).getActive();
            int afterWarmup = getWeight(invoker, invocation);
            weights[i] = afterWarmup;
            if (leastActive == -1 || active < leastActive) {
                leastActive = active;
                leastCount = 1;
                leastIndexes[0] = i;
                totalWeight = afterWarmup;
                firstWeight = afterWarmup;
                sameWeight = true;
            } else if (active == leastActive) {
                leastIndexes[leastCount++] = i;
                totalWeight += afterWarmup;
                if (sameWeight && afterWarmup != firstWeight) {
                    sameWeight = false;
                }
            }
        }
       // Nếu chỉ một Invoker có active count nhỏ nhất, trả về Invoker đó.
         if (leastCount == 1) {
             return invokers.get(leastIndexes[0]);
         }
         // Nếu nhiều Invoker có cùng active count nhỏ nhất nhưng weight khác nhau,
         // cách xử lý ở đây giống với RandomLoadBalance.
         if (!sameWeight && totalWeight > 0) {
             int offsetWeight = ThreadLocalRandom.current().nextInt(totalWeight);
             for (int i = 0; i < leastCount; i++) {
                 int leastIndex = leastIndexes[i];
                 offsetWeight -= weights[leastIndex];
                 if (offsetWeight < 0) {
                     return invokers.get(leastIndex);
                 }
             }
         }
         return invokers.get(leastIndexes[ThreadLocalRandom.current().nextInt(leastCount)]);
     }
 }

```

Active count được lưu trong một `ConcurrentMap` của `RpcStatus`. Dựa trên URL và tên method mà provider được gọi, ta có thể lấy active count tương ứng. Nói cách khác, active count của từng method trong provider là độc lập với nhau.

```java
public class RpcStatus {

    private static final ConcurrentMap<String, ConcurrentMap<String, RpcStatus>> METHOD_STATISTICS =
            new ConcurrentHashMap<String, ConcurrentMap<String, RpcStatus>>();

   public static RpcStatus getStatus(URL url, String methodName) {
        String uri = url.toIdentityString();
        ConcurrentMap<String, RpcStatus> map = METHOD_STATISTICS.computeIfAbsent(uri, k -> new ConcurrentHashMap<>());
        return map.computeIfAbsent(methodName, k -> new RpcStatus());
    }
    public int getActive() {
        return active.get();
    }

}
```

#### ConsistentHashLoadBalance

`ConsistentHashLoadBalance` có lẽ cũng không xa lạ; strategy load balancing này thường được dùng trong database sharding và nhiều cluster.

`ConsistentHashLoadBalance` là **strategy consistent hash load balancing**. Trong `ConsistentHashLoadBalance` không có khái niệm weight; provider nào xử lý request phụ thuộc vào parameter của request, nghĩa là các request có cùng parameter luôn được gửi đến cùng một provider.

![](https://oss.javaguide.cn/java-guide-blog/consistent-hash-data-incline.jpg)

Ngoài ra, để tránh vấn đề data skew (node phân tán chưa đủ, nhiều request rơi vào cùng một node), Dubbo còn đưa vào khái niệm virtual node. Virtual node giúp các node phân tán hơn, cân bằng hiệu quả số lượng request của từng node.

![](https://oss.javaguide.cn/java-guide-blog/consistent-hash-invoker.jpg)

Tài liệu chính thức có phân tích source code chi tiết tại [https://dubbo.apache.org/zh/docs/v2.7/dev/source/loadbalance/#23-consistenthashloadbalance](https://dubbo.apache.org/zh/docs/v2.7/dev/source/loadbalance/#23-consistenthashloadbalance). Ngoài ra còn có [PR#5440](https://github.com/apache/dubbo/pull/5440) liên quan đến việc sửa một số bug của `ConsistentHashLoadBalance` trong version cũ. Nếu quan tâm, bạn có thể dành thêm thời gian nghiên cứu. Phần phân tích này không đi sâu hơn; bài tập dành cho bạn!

#### RoundRobinLoadBalance

Weighted round-robin load balancing.

Round-robin là phân phối request lần lượt cho từng provider. Weighted round-robin dựa trên round-robin và khiến nhiều request hơn được gửi đến provider có weight lớn hơn. Ví dụ, giả sử có hai server cung cấp cùng một service là S1 và S2, weight của S1 là 7, weight của S2 là 3.

Nếu có 10 request, S1 sẽ xử lý 7 request và S2 xử lý 3 request.

Tuy nhiên, với `RandomLoadBalance`, hoàn toàn có thể xảy ra trường hợp trong 10 request có 9 request do S1 xử lý (vấn đề xác suất).

Implementation của `RoundRobinLoadBalance` trong Dubbo đã được sửa và xây dựng lại nhiều lần. `RoundRobinLoadBalance` ở version Dubbo-2.6.5 là thuật toán weighted round-robin mượt.

## Serialization protocol của Dubbo

### Dubbo hỗ trợ những serialization method nào?

![Serialization protocol được Dubbo hỗ trợ](https://oss.javaguide.cn/github/javaguide/csdn/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0MzM3Mjcy,size_16,color_FFFFFF,t_70-20230309234143460.png)

Dubbo hỗ trợ nhiều serialization method: serialization tích hợp sẵn của JDK, hessian2, JSON, Kryo, FST, Protostuff, ProtoBuf, v.v.

Serialization method mặc định của Dubbo là hessian2.

### Bạn hiểu gì về các serialization protocol này?

Thông thường, ta không dùng trực tiếp serialization method tích hợp sẵn của JDK. Có hai nguyên nhân chính:

1. **Không hỗ trợ cross-language invocation**: nếu gọi service được phát triển bằng ngôn ngữ khác thì không hỗ trợ.
2. **Performance kém**: performance thấp hơn các serialization framework khác, chủ yếu vì byte array sau serialization có kích thước lớn, làm tăng chi phí truyền tải.

Vì lý do performance, ta thường không cân nhắc JSON serialization.

Protostuff, ProtoBuf và hessian2 đều là cross-language serialization method; nếu có nhu cầu cross-language thì có thể cân nhắc sử dụng.

Kryo và FST là hai serialization method được Dubbo đưa vào sau này, có performance rất tốt. Tuy nhiên, cả hai đều được thiết kế riêng cho Java. Một bài viết trên website chính thức của Dubbo đề xuất sử dụng Kryo làm serialization method trong production environment.

Tài liệu chính thức của Dubbo còn có [biểu đồ so sánh performance của các serialization protocol](https://dubbo.apache.org/zh/docs/v2.7/user/serialization/#m-zhdocsv27userserialization) để tham khảo.

![So sánh performance của các serialization protocol](https://oss.javaguide.cn/github/javaguide/distributed-system/rpc/dubbo-serialization-protocol-performance-comparison.png)

<!-- @include: @article-footer.snippet.md -->
