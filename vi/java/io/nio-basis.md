---
title: Tổng hợp kiến thức cốt lõi về Java NIO
description: "Tổng hợp toàn diện kiến thức cốt lõi về Java NIO: giải thích chi tiết ba component cốt lõi Channel, Buffer, Selector, cách triển khai non-blocking I/O, kỹ thuật zero-copy và so sánh hiệu năng với I/O truyền thống."
category: Java
tag:
  - Java IO
  - Java Basics
head:
  - - meta
    - name: keywords
      content: Java NIO,Channel,Buffer,Selector,Non-blocking IO,I/O multiplexing,zero-copy,NIO core components
---

Trước khi học NIO, bạn cần tìm hiểu trước kiến thức lý thuyết cơ bản về các I/O model của máy tính. Nếu chưa biết, bạn có thể tham khảo bài viết này của tôi: [Giải thích chi tiết Java IO model](https://javaguide.cn/java/io/io-model.html).

## Giới thiệu về NIO

Trong Java I/O model truyền thống (BIO), các thao tác I/O được thực hiện theo cách blocking. Nói cách khác, khi một thread thực hiện một thao tác I/O, nó sẽ bị blocking cho đến khi thao tác hoàn tất. Mô hình blocking này có thể gây ra bottleneck hiệu năng khi xử lý nhiều connection concurrent, vì cần tạo một thread cho mỗi connection, trong khi việc tạo và chuyển đổi thread đều có chi phí.

Để giải quyết vấn đề này, Java 1.4 đã giới thiệu một I/O model mới — **NIO** (New IO, còn được gọi là Non-blocking IO). NIO khắc phục hạn chế của synchronous blocking I/O: cung cấp I/O non-blocking, hướng buffer và dựa trên channel trong Java standard code, có thể dùng một số lượng nhỏ thread để xử lý nhiều connection, từ đó cải thiện đáng kể hiệu suất I/O và concurrency.

Hình dưới đây là sơ đồ so sánh đơn giản cách BIO, NIO và AIO xử lý request từ client (về AIO, bạn có thể xem bài viết này của tôi: [Giải thích chi tiết Java IO model](https://javaguide.cn/java/io/io-model.html), không phải trọng tâm, chỉ cần hiểu là được).

![So sánh BIO, NIO và AIO](https://oss.javaguide.cn/github/javaguide/java/nio/bio-aio-nio.png)

⚠️Cần lưu ý: sử dụng NIO không nhất thiết có nghĩa là hiệu năng cao. Ưu thế hiệu năng của nó chủ yếu thể hiện trong môi trường network có concurrency cao và latency cao. Khi số lượng connection ít, mức độ concurrency thấp hoặc tốc độ truyền network nhanh, hiệu năng của NIO chưa chắc tốt hơn BIO truyền thống.

## Các component cốt lõi của NIO

NIO chủ yếu gồm ba component cốt lõi sau:

- **Buffer**: NIO thực hiện đọc và ghi dữ liệu thông qua buffer. Khi đọc, dữ liệu trong Channel được nạp vào Buffer; khi ghi, dữ liệu trong Buffer được ghi vào Channel.
- **Channel**: Channel biểu thị một connection đang mở tới các entity như file, socket. Tùy interface cụ thể, nó có thể hỗ trợ đọc, ghi hoặc hỗ trợ đồng thời cả hai.
- **Selector**: Cho phép một thread giám sát event ready của nhiều channel có thể select (`SelectableChannel`), qua đó thực hiện I/O multiplexing. Selector chịu trách nhiệm báo cáo event đã sẵn sàng hay chưa, không chịu trách nhiệm phân bổ thread xử lý.

Mối quan hệ giữa ba component được thể hiện trong hình dưới đây (chưa hiểu cũng không sao, phần sau sẽ giải thích chi tiết):

![Mối quan hệ giữa Buffer, Channel và Selector](https://oss.javaguide.cn/github/javaguide/java/nio/channel-buffer-selector.png)

Dưới đây là phần giới thiệu chi tiết về ba component này.

### Buffer

Trong BIO truyền thống, việc đọc và ghi dữ liệu hướng theo stream, được chia thành byte stream và character stream.

Trong thư viện NIO của Java 1.4, mọi dữ liệu đều được xử lý bằng buffer. Đây là điểm khác biệt quan trọng giữa thư viện mới và BIO trước đó, tương tự buffered stream trong BIO. Khi đọc dữ liệu bằng NIO, dữ liệu được đọc trực tiếp vào buffer. Khi ghi dữ liệu, dữ liệu được ghi vào buffer. Khi đọc và ghi dữ liệu bằng NIO, mọi thao tác đều được thực hiện thông qua buffer.

Các subclass của `Buffer` được thể hiện trong hình dưới đây. Trong đó, loại được sử dụng phổ biến nhất là `ByteBuffer`, có thể dùng để lưu trữ và thao tác với byte data.

![Các subclass của Buffer](https://oss.javaguide.cn/github/javaguide/java/nio/buffer-subclasses.png)

Bạn có thể hiểu Buffer là một sequence dữ liệu cùng type, tuyến tính và có giới hạn. Một số Buffer được hỗ trợ bởi array, nhưng các implementation như direct buffer không nhất thiết có underlying array có thể truy cập.

Để hiểu rõ hơn về buffer, chúng ta hãy xem nhanh bốn member variable được định nghĩa trong class `Buffer`:

```java
public abstract class Buffer {
    // Invariants: mark <= position <= limit <= capacity
    private int mark = -1;
    private int position = 0;
    private int limit;
    private int capacity;
}
```

Ý nghĩa cụ thể của bốn member variable này như sau:

1. Capacity (`capacity`): lượng dữ liệu tối đa mà `Buffer` có thể lưu trữ, được thiết lập khi tạo `Buffer` và không thể thay đổi;
2. Limit (`limit`): ranh giới dữ liệu có thể đọc/ghi trong `Buffer`. Ở write mode, `limit` biểu thị lượng dữ liệu nhiều nhất có thể ghi, thường bằng `capacity` (có thể thiết lập bằng method `limit(int newLimit)`); ở read mode, `limit` bằng kích thước dữ liệu thực tế đã ghi vào Buffer.
3. Position (`position`): vị trí (index) của dữ liệu tiếp theo có thể đọc/ghi. Khi chuyển từ write mode sang read mode (`flip`), `position` sẽ được đặt về 0 để có thể bắt đầu đọc/ghi từ đầu.
4. Mark (`mark`): `Buffer` cho phép định vị trực tiếp position tới vị trí mark này, đây là một thuộc tính tùy chọn;

Khi `mark` đã được định nghĩa, các variable trên thỏa mãn quan hệ sau: **0 <= mark <= position <= limit <= capacity**. Khi `mark` chưa được định nghĩa, gọi `reset()` sẽ ném ra `InvalidMarkException`.

Ngoài ra, Buffer có read mode và write mode, lần lượt dùng để đọc dữ liệu từ Buffer hoặc ghi dữ liệu vào Buffer. Sau khi Buffer được tạo, mặc định nó ở write mode; gọi `flip()` có thể chuyển sang read mode. Nếu muốn chuyển lại write mode, có thể gọi method `clear()` hoặc `compact()`.

![Mối quan hệ giữa position, limit và capacity](https://oss.javaguide.cn/github/javaguide/java/nio/JavaNIOBuffer.png)

![Mối quan hệ giữa position, limit và capacity](https://oss.javaguide.cn/github/javaguide/java/nio/NIOBufferClassAttributes.png)

Đối tượng `Buffer` không thể được tạo bằng cách gọi constructor thông qua `new`, chỉ có thể khởi tạo `Buffer` bằng static method.

Ở đây dùng `ByteBuffer` làm ví dụ:

```java
// Phân bổ heap memory
public static ByteBuffer allocate(int capacity);
// Phân bổ direct memory
public static ByteBuffer allocateDirect(int capacity);
```

Hai method cốt lõi nhất của Buffer:

1. `get`: đọc dữ liệu trong buffer
2. `put`: ghi dữ liệu vào buffer

Ngoài hai method trên, các method quan trọng khác:

- `flip`: chuyển buffer từ write mode sang read mode, đặt giá trị `limit` thành giá trị `position` hiện tại và đặt giá trị `position` về 0.
- `clear`: xóa buffer, chuyển buffer từ read mode sang write mode, đặt giá trị `position` về 0 và đặt giá trị `limit` thành giá trị `capacity`.
- …

Quá trình thay đổi dữ liệu trong Buffer:

```java
import java.nio.*;

public class CharBufferDemo {
    public static void main(String[] args) {
        // Phân bổ một CharBuffer có capacity là 8
        CharBuffer buffer = CharBuffer.allocate(8);
        System.out.println("Trạng thái ban đầu:");
        printState(buffer);

        // Ghi 3 ký tự vào buffer
        buffer.put('a').put('b').put('c');
        System.out.println("Trạng thái sau khi ghi 3 ký tự:");
        printState(buffer);

        // Gọi flip(), chuẩn bị đọc dữ liệu trong buffer, đặt position về 0 và limit về 3
        buffer.flip();
        System.out.println("Trạng thái sau khi gọi flip():");
        printState(buffer);

        // Đọc ký tự
        while (buffer.hasRemaining()) {
            System.out.print(buffer.get());
        }

        // Gọi clear(), xóa buffer, đặt position về 0 và limit về capacity
        buffer.clear();
        System.out.println("Trạng thái sau khi gọi clear():");
        printState(buffer);

    }

    // In ra capacity, limit và position của buffer
    private static void printState(CharBuffer buffer) {
        System.out.print("capacity: " + buffer.capacity());
        System.out.print(", limit: " + buffer.limit());
        System.out.print(", position: " + buffer.position());
        System.out.println("\n");
    }
}
```

Output:

```bash
Trạng thái ban đầu:
capacity: 8, limit: 8, position: 0

Trạng thái sau khi ghi 3 ký tự:
capacity: 8, limit: 8, position: 3

Đang chuẩn bị đọc dữ liệu trong buffer!

Trạng thái sau khi gọi flip():
capacity: 8, limit: 3, position: 0

Dữ liệu đã đọc: abc

Trạng thái sau khi gọi clear():
capacity: 8, limit: 8, position: 0
```

Để giúp dễ hình dung, tôi đã vẽ một hình ảnh thể hiện sự thay đổi của `capacity`, `limit` và `position` ở mỗi giai đoạn.

![Thay đổi của capacity, limit và position ở mỗi giai đoạn](https://oss.javaguide.cn/github/javaguide/java/nio/NIOBufferClassAttributesDataChanges.png)

### Channel

Channel là một channel tạo connection với data source như file, network socket. Chúng ta có thể dùng nó để đọc và ghi dữ liệu, giống như mở một đường ống nước để dữ liệu tự do chảy trong Channel.

Stream trong BIO là một chiều, được chia thành nhiều loại `InputStream` (input stream) và `OutputStream` (output stream), dữ liệu chỉ truyền theo một hướng. Các loại channel khác nhau có thể dùng để đọc, ghi hoặc đồng thời đọc ghi. Ví dụ, khả năng đọc ghi của `FileChannel` phụ thuộc vào cách mở, còn `SocketChannel` hỗ trợ cả đọc và ghi.

Channel tương tác với Buffer đã giới thiệu ở trên: khi đọc, dữ liệu trong Channel được nạp vào Buffer; khi ghi, dữ liệu trong Buffer được ghi vào Channel.

![Mối quan hệ giữa Channel và Buffer](https://oss.javaguide.cn/github/javaguide/java/nio/channel-buffer.png)

Ngoài ra, một số Channel (ví dụ `SocketChannel`) hỗ trợ đồng thời đọc và ghi, có thể ánh xạ trực tiếp hơn khả năng communication hai chiều của operating system bên dưới.

Các subclass của `Channel` được thể hiện trong hình dưới đây.

![Các subclass của Channel](https://oss.javaguide.cn/github/javaguide/java/nio/channel-subclasses.png)

Trong đó, các loại channel được sử dụng phổ biến nhất gồm:

- `FileChannel`: channel truy cập file;
- `SocketChannel`, `ServerSocketChannel`: channel communication TCP;
- `DatagramChannel`: channel communication UDP;

![Sơ đồ quan hệ inheritance của Channel](https://oss.javaguide.cn/github/javaguide/java/nio/channel-inheritance-relationship.png)

Hai method cốt lõi nhất của Channel:

1. `read`: đọc dữ liệu và ghi vào Buffer.
2. `write`: ghi dữ liệu trong Buffer vào Channel.

Ở đây dùng `FileChannel` để minh họa việc đọc dữ liệu file.

```java
RandomAccessFile reader = new RandomAccessFile("/Users/guide/Documents/test_read.in", "r");
FileChannel channel = reader.getChannel();
ByteBuffer buffer = ByteBuffer.allocate(1024);
channel.read(buffer);
```

### Selector

Selector là một component quan trọng trong NIO, cho phép một thread xử lý nhiều Channel. Selector dựa trên event-driven I/O multiplexing model. Nguyên lý hoạt động chủ yếu là: đăng ký event của channel thông qua Selector, sau đó Selector liên tục polling các Channel đã đăng ký. Khi event xảy ra, ví dụ một TCP connection mới được tiếp nhận trên một Channel hoặc xảy ra event đọc, ghi, Channel đó sẽ ở trạng thái ready và được Selector polling ra. Selector đưa các Channel liên quan vào ready set. Thông qua SelectionKey có thể lấy tập hợp Channel ready, sau đó thực hiện các thao tác I/O tương ứng trên những Channel ready đó.

![Sơ đồ hoạt động của Selector](https://oss.javaguide.cn/github/javaguide/java/nio/selector-channel-selectionkey.png)

Một multiplexer Selector có thể đồng thời giám sát nhiều Channel. Implementation bên dưới của Selector do `SelectorProvider` của platform quyết định, ví dụ trên Linux có thể dùng epoll, còn các platform khác sẽ dùng implementation tương ứng. Số lượng connection có thể tiếp nhận vẫn bị giới hạn bởi các resource như file descriptor, memory và system configuration.

Selector có thể lắng nghe bốn loại event sau:

1. `SelectionKey.OP_ACCEPT`: biểu thị event channel accept connection, thường dùng cho `ServerSocketChannel`.
2. `SelectionKey.OP_CONNECT`: biểu thị event channel hoàn tất connection, thường dùng cho `SocketChannel`.
3. `SelectionKey.OP_READ`: biểu thị event channel đã sẵn sàng đọc, nghĩa là có dữ liệu có thể đọc.
4. `SelectionKey.OP_WRITE`: biểu thị event channel đã sẵn sàng ghi, nghĩa là có thể ghi dữ liệu.

`Selector` là abstract class, có thể tạo instance Selector bằng cách gọi static method `open()` của class này. Selector có thể đồng thời giám sát trạng thái `IO` của nhiều `SelectableChannel`, là core của `non-blocking IO`.

Một instance Selector có ba tập hợp `SelectionKey`:

1. Tập hợp toàn bộ `SelectionKey`: biểu thị các `Channel` đã đăng ký trên Selector đó, có thể trả về tập hợp này thông qua method `keys()`.
2. Tập hợp `SelectionKey` được select: biểu thị toàn bộ Channel có thể lấy được thông qua method `select()` và cần xử lý `IO`, có thể trả về tập hợp này thông qua `selectedKeys()`.
3. Tập hợp `SelectionKey` đã cancel: biểu thị toàn bộ `Channel` đã bị hủy quan hệ đăng ký. Khi thực thi method `select()` lần tiếp theo, `SelectionKey` tương ứng với các `Channel` này sẽ bị xóa hoàn toàn. Chương trình thường không cần truy cập trực tiếp tập hợp này và cũng không có method truy cập được expose.

Minh họa đơn giản cách duyệt tập hợp `SelectionKey` được select và xử lý:

```java
Set<SelectionKey> selectedKeys = selector.selectedKeys();
Iterator<SelectionKey> keyIterator = selectedKeys.iterator();
while (keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();
    if (key != null) {
        if (key.isAcceptable()) {
            // ServerSocketChannel đã tiếp nhận connection mới
        } else if (key.isConnectable()) {
            // Một connection mới đã được thiết lập
        } else if (key.isReadable()) {
            // Channel có dữ liệu sẵn sàng để đọc
        } else if (key.isWritable()) {
            // Channel đã sẵn sàng ghi dữ liệu
        }
    }
    keyIterator.remove();
}
```

Selector còn cung cấp một loạt method liên quan đến `select()`:

- `int select()`: giám sát tất cả `Channel` đã đăng ký. Khi một trong số chúng có thao tác `IO` cần xử lý, method này trả về và thêm `SelectionKey` tương ứng vào tập hợp `SelectionKey` được select. Giá trị trả về là số lượng key mà ready set được cập nhật trong thao tác này.
- `int select(long timeout)`: thao tác `select()` có thể thiết lập thời gian timeout.
- `int selectNow()`: thực hiện thao tác `select()` trả về ngay lập tức. So với method `select()` không có argument, method này không blocking thread.
- `Selector wakeup()`: khiến method `select()` chưa trả về lập tức trả về.
- …

Ví dụ đơn giản sử dụng Selector để thực hiện network read/write:

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

public class NioSelectorExample {

  public static void main(String[] args) {
    try {
      ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
      serverSocketChannel.configureBlocking(false);
      serverSocketChannel.socket().bind(new InetSocketAddress(8080));

      Selector selector = Selector.open();
      // Đăng ký ServerSocketChannel với Selector và lắng nghe event OP_ACCEPT
      serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

      while (true) {
        int readyChannels = selector.select();

        if (readyChannels == 0) {
          continue;
        }

        Set<SelectionKey> selectedKeys = selector.selectedKeys();
        Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

        while (keyIterator.hasNext()) {
          SelectionKey key = keyIterator.next();

          if (key.isAcceptable()) {
            // Xử lý event connection
            ServerSocketChannel server = (ServerSocketChannel) key.channel();
            SocketChannel client = server.accept();
            client.configureBlocking(false);

            // Đăng ký channel của client với Selector và lắng nghe event OP_READ
            client.register(selector, SelectionKey.OP_READ);
          } else if (key.isReadable()) {
            // Xử lý event read
            SocketChannel client = (SocketChannel) key.channel();
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            int bytesRead = client.read(buffer);

            if (bytesRead > 0) {
              buffer.flip();
              System.out.println("Dữ liệu nhận được: " +new String(buffer.array(), 0, bytesRead));
              // Lưu dữ liệu chờ gửi và lắng nghe event OP_WRITE
              key.attach(ByteBuffer.wrap("Hello, Client!".getBytes()));
              key.interestOps(SelectionKey.OP_WRITE);
            } else if (bytesRead < 0) {
              // Client đã ngắt connection
              client.close();
            }
          } else if (key.isWritable()) {
            // Xử lý event write
            SocketChannel client = (SocketChannel) key.channel();
            ByteBuffer buffer = (ByteBuffer) key.attachment();
            client.write(buffer);

            // Non-blocking write có thể chỉ ghi một phần dữ liệu; chuyển lại sang OP_READ sau khi ghi xong toàn bộ
            if (!buffer.hasRemaining()) {
              key.attach(null);
              key.interestOps(SelectionKey.OP_READ);
            }
          }

          keyIterator.remove();
        }
      }
    } catch (IOException e) {
      e.printStackTrace();
    }
  }
}
```

Trong ví dụ, chúng ta tạo một server đơn giản lắng nghe port 8080, sử dụng Selector để xử lý event connection, read và write. Khi nhận dữ liệu từ client, server đọc dữ liệu và in ra console, sau đó phản hồi `"Hello, Client!"` cho client.

## NIO zero-copy

Zero-copy là một phương pháp phổ biến để cải thiện hiệu năng thao tác IO. Các open-source project hàng đầu như ActiveMQ, Kafka, RocketMQ, QMQ và Netty đều sử dụng zero-copy.

Zero-copy nghĩa là khi máy tính thực hiện thao tác IO, CPU không cần copy dữ liệu từ storage area này sang storage area khác, qua đó giảm context switch và thời gian CPU copy. Nói cách khác, zero-copy chủ yếu giải quyết vấn đề operating system liên tục copy dữ liệu khi xử lý thao tác I/O. Các kỹ thuật triển khai zero-copy phổ biến gồm: `mmap+write`, `sendfile` và `sendfile + DMA gather copy`.

Hình dưới đây thể hiện sơ đồ so sánh các kỹ thuật zero-copy:

|                            | CPU copy | DMA copy | System call | Context switch |
| -------------------------- | -------- | -------- | ----------- | -------------- |
| Phương pháp truyền thống   | 2        | 2        | read+write  | 4              |
| mmap+write                 | 1        | 2        | mmap+write  | 4              |
| sendfile                   | 1        | 2        | sendfile    | 2              |
| sendfile + DMA gather copy | 0        | 2        | sendfile    | 2              |

Có thể thấy, dù là I/O theo cách truyền thống hay sau khi áp dụng zero-copy, đều không thể thiếu 2 lần DMA (Direct Memory Access) copy. Vì cả hai lần DMA đều phụ thuộc vào hardware để hoàn thành. Zero-copy chủ yếu giảm CPU copy và context switch.

Java hỗ trợ zero-copy:

- `MappedByteBuffer` là implementation memory-mapped file do Java NIO cung cấp, có thể map một phần file vào memory. Cơ chế bên dưới phụ thuộc vào operating system, ví dụ trên Linux thường dựa trên `mmap`.
- `transferTo()/transferFrom()` của `FileChannel` có thể truyền byte trực tiếp giữa các channel. Nhiều operating system có thể tối ưu kiểu truyền này, ví dụ trên Linux có thể sử dụng `sendfile`. Implementation cụ thể phụ thuộc vào JDK và operating system. Về cách sử dụng `FileChannel`, bạn có thể xem bài viết này: [Cách sử dụng Java NIO FileChannel](https://www.cnblogs.com/robothy/p/14235598.html).

Ví dụ code:

```java
private void loadFileIntoMemory(File xmlFile) throws IOException {
  FileInputStream fis = new FileInputStream(xmlFile);
  // Tạo đối tượng FileChannel
  FileChannel fc = fis.getChannel();
  // FileChannel.map() map file vào direct memory và trả về đối tượng MappedByteBuffer
  MappedByteBuffer mmb = fc.map(FileChannel.MapMode.READ_ONLY, 0, fc.size());
  xmlFileBuffer = new byte[(int)fc.size()];
  mmb.get(xmlFileBuffer);
  fis.close();
}
```

## Tổng kết

Trong bài viết này, chúng ta chủ yếu đã giới thiệu các kiến thức cốt lõi về NIO, gồm các component cốt lõi của NIO và zero-copy.

Nếu cần dùng NIO để xây dựng network program, không nên dùng trực tiếp NIO native vì lập trình phức tạp và functionality quá yếu. Bạn nên dùng các network programming framework trưởng thành dựa trên NIO như Netty. Netty thực hiện một số tối ưu và mở rộng trên nền tảng NIO, ví dụ hỗ trợ nhiều protocol, hỗ trợ SSL/TLS, v.v.

## Tham khảo

- Phân tích sơ lược Java NIO: <https://tech.meituan.com/2016/11/04/nio.html>

- Người phỏng vấn hỏi: Bạn biết Java NIO không? <https://mp.weixin.qq.com/s/mZobf-U8OSYQfHfYBEB6KA>

- Java NIO: Buffer, Channel và Selector: <https://www.javadoop.com/post/java-nio>

<!-- @include: @article-footer.snippet.md -->
