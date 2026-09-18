---
title: "Tổng hợp design pattern trong Java IO"
description: "Phân tích chuyên sâu design pattern trong Java IO: giải thích chi tiết ứng dụng Decorator trong BufferedInputStream, cách triển khai Adapter bằng InputStreamReader, thiết kế Template Method của InputStream, tìm hiểu kiến trúc thư viện class Java IO."
category: Java
tag:
  - Java IO
  - Java Basics
head:
  - - meta
    - name: keywords
      content: Java IO design pattern,Decorator pattern,Adapter pattern,Template Method pattern,FilterInputStream,thiết kế IO stream
---

Trong bài này, chúng ta cùng xem qua những ứng dụng của các design pattern có thể học được từ IO.

## Decorator pattern

**Decorator (Decorator) pattern** có thể mở rộng chức năng của object mà không thay đổi object ban đầu.

Decorator pattern dùng composition thay cho inheritance để mở rộng chức năng của class ban đầu, nên thực tế hơn trong những trường hợp quan hệ inheritance tương đối phức tạp (quan hệ inheritance giữa các class trong IO cũng khá phức tạp).

Đối với byte stream, `FilterInputStream` (input stream tương ứng) và `FilterOutputStream` (output stream tương ứng) là phần cốt lõi của Decorator pattern, lần lượt dùng để tăng cường chức năng của các object thuộc subclass của `InputStream` và `OutputStream`.

Các class như `BufferedInputStream` (byte buffered input stream), `DataInputStream` đều là subclass của `FilterInputStream`; `BufferedOutputStream` (byte buffered output stream), `DataOutputStream` đều là subclass của `FilterOutputStream`.

Ví dụ, chúng ta có thể dùng `BufferedInputStream` (byte buffered input stream) để tăng cường chức năng của `FileInputStream`.

Constructor của `BufferedInputStream` như sau:

```java
public BufferedInputStream(InputStream in) {
    this(in, DEFAULT_BUFFER_SIZE);
}

public BufferedInputStream(InputStream in, int size) {
    super(in);
    if (size <= 0) {
        throw new IllegalArgumentException("Buffer size <= 0");
    }
    buf = new byte[size];
}
```

Có thể thấy, một trong các tham số của constructor `BufferedInputStream` là `InputStream`.

Ví dụ code `BufferedInputStream`:

```java
try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream("input.txt"))) {
    int content;
    long skip = bis.skip(2);
    while ((content = bis.read()) != -1) {
        System.out.print((char) content);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

Lúc này, có thể bạn sẽ thắc mắc: **Tại sao chúng ta không tạo thẳng một `BufferedFileInputStream` (file input stream có buffer) nhỉ?**

```java
BufferedFileInputStream bfis = new BufferedFileInputStream("input.txt");
```

Nếu `InputStream` có ít subclass thì làm như vậy không có vấn đề gì. Tuy nhiên, `InputStream` có quá nhiều subclass và quan hệ inheritance cũng quá phức tạp. Nếu tùy chỉnh một buffered input stream tương ứng cho từng subclass, chẳng phải sẽ rất phiền phức sao?

Nếu bạn quen thuộc với IO stream, bạn sẽ nhận ra `ZipInputStream` và `ZipOutputStream` cũng có thể lần lượt tăng cường khả năng của `BufferedInputStream` và `BufferedOutputStream`.

```java
BufferedInputStream bis = new BufferedInputStream(new FileInputStream(fileName));
ZipInputStream zis = new ZipInputStream(bis);

BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(fileName));
ZipOutputStream zipOut = new ZipOutputStream(bos);
```

`ZipInputStream` và `ZipOutputStream` lần lượt kế thừa từ `InflaterInputStream` và `DeflaterOutputStream`.

```java
public
class InflaterInputStream extends FilterInputStream {
}

public
class DeflaterOutputStream extends FilterOutputStream {
}

```

Đây cũng là một đặc điểm rất quan trọng của Decorator pattern: có thể lồng nhiều decorator để sử dụng trên class ban đầu.

Để thực hiện hiệu ứng này, decorator class cần kế thừa cùng abstract class với class ban đầu hoặc implement cùng interface. Các decorator class và class ban đầu liên quan đến IO được giới thiệu ở trên có parent class chung là `InputStream` và `OutputStream`.

Đối với character stream, `BufferedReader` có thể dùng để bổ sung chức năng cho subclass của `Reader` (character input stream), còn `BufferedWriter` có thể dùng để bổ sung chức năng cho subclass của `Writer` (character output stream).

```java
BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(new FileOutputStream(fileName), "UTF-8"));
```

Có rất nhiều ví dụ về việc áp dụng Decorator pattern trong IO stream, không cần cố ghi nhớ, hoàn toàn không cần thiết! Sau khi nắm được cốt lõi của Decorator pattern, khi sử dụng bạn sẽ tự nhiên nhận ra những nơi nào đã áp dụng Decorator pattern.

## Adapter pattern

**Adapter (Adapter Pattern)** chủ yếu dùng để điều phối hoạt động giữa các class có interface không tương thích. Bạn có thể liên tưởng đến power adapter thường dùng trong đời sống hằng ngày.

Trong Adapter pattern, object hoặc class được adapt được gọi là **Adaptee**, còn object hoặc class tác động lên adaptee được gọi là **Adapter**. Adapter được chia thành object adapter và class adapter. Class adapter dùng quan hệ inheritance để triển khai, còn object adapter dùng quan hệ composition để triển khai.

Interface của character stream và byte stream trong IO stream khác nhau. Việc chúng có thể phối hợp hoạt động được với nhau là nhờ Adapter pattern, chính xác hơn là object adapter. Thông qua adapter, chúng ta có thể adapt một byte stream object thành character stream object, từ đó có thể trực tiếp dùng byte stream object để đọc hoặc ghi dữ liệu dạng character.

`InputStreamReader` và `OutputStreamWriter` là hai adapter, đồng thời cũng là cầu nối giữa byte stream và character stream. `InputStreamReader` dùng `StreamDecoder` (stream decoder) để decode byte, **thực hiện chuyển đổi byte stream thành character stream**; `OutputStreamWriter` dùng `StreamEncoder` (stream encoder) để encode character, thực hiện chuyển đổi character stream thành byte stream.

Các subclass của `InputStream` và `OutputStream` là adaptee, còn `InputStreamReader` và `OutputStreamWriter` là adapter.

```java
// InputStreamReader là adapter, FileInputStream là class được adapt
InputStreamReader isr = new InputStreamReader(new FileInputStream(fileName), "UTF-8");
// BufferedReader tăng cường chức năng của InputStreamReader (Decorator pattern)
BufferedReader bufferedReader = new BufferedReader(isr);
```

Một phần source code của `java.io.InputStreamReader`:

```java
public class InputStreamReader extends Reader {
    // Object dùng để decode
    private final StreamDecoder sd;
    public InputStreamReader(InputStream in) {
        super(in);
        try {
            // Lấy object StreamDecoder
            sd = StreamDecoder.forInputStreamReader(in, this, (String)null);
        } catch (UnsupportedEncodingException e) {
            throw new Error(e);
        }
    }
    // Dùng object StreamDecoder để thực hiện công việc đọc cụ thể
    public int read() throws IOException {
        return sd.read();
    }
}
```

Một phần source code của `java.io.OutputStreamWriter`:

```java
public class OutputStreamWriter extends Writer {
    // Object dùng để encode
    private final StreamEncoder se;
    public OutputStreamWriter(OutputStream out) {
        super(out);
        try {
           // Lấy object StreamEncoder
            se = StreamEncoder.forOutputStreamWriter(out, this, (String)null);
        } catch (UnsupportedEncodingException e) {
            throw new Error(e);
        }
    }
    // Dùng object StreamEncoder để thực hiện công việc ghi cụ thể
    public void write(int c) throws IOException {
        se.write(c);
    }
}
```

**Adapter pattern và Decorator pattern khác nhau như thế nào?**

**Decorator pattern** tập trung hơn vào việc tăng cường động chức năng của class ban đầu. Decorator class cần kế thừa cùng abstract class với class ban đầu hoặc implement cùng interface. Ngoài ra, Decorator pattern hỗ trợ lồng nhiều decorator để sử dụng trên class ban đầu.

**Adapter pattern** tập trung hơn vào việc cho phép các class có interface không tương thích và không thể tương tác cùng làm việc với nhau. Khi gọi method tương ứng của adapter, bên trong adapter sẽ gọi method của adaptee hoặc class liên quan đến adapter. Quá trình này là trong suốt. Ví dụ, `InputStreamReader` và `OutputStreamWriter` lần lượt adapt byte input stream, byte output stream thành character input stream, character output stream, đồng thời hoàn tất việc encode/decode giữa byte và character ở bên trong.

```java
Reader reader = new InputStreamReader(inputStream, StandardCharsets.UTF_8);
Writer writer = new OutputStreamWriter(outputStream, StandardCharsets.UTF_8);
```

Adapter và adaptee không cần kế thừa cùng abstract class hoặc implement cùng interface.

Ngoài ra, class `FutureTask` sử dụng Adapter pattern. Inner class `RunnableAdapter` của `Executors` là một adapter, dùng để adapt `Runnable` thành `Callable`.

`FutureTask` có một constructor nhận tham số `Runnable`:

```java
public FutureTask(Runnable runnable, V result) {
    // Gọi method callable của class Executors
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;
}
```

Method và adapter tương ứng trong `Executors`:

```java
// Thực tế gọi constructor của inner class RunnableAdapter trong Executors
public static <T> Callable<T> callable(Runnable task, T result) {
    if (task == null)
        throw new NullPointerException();
    return new RunnableAdapter<T>(task, result);
}
// Adapter
static final class RunnableAdapter<T> implements Callable<T> {
    final Runnable task;
    final T result;
    RunnableAdapter(Runnable task, T result) {
        this.task = task;
        this.result = result;
    }
    public T call() {
        task.run();
        return result;
    }
}
```

## Factory pattern

Factory pattern dùng để tạo object. NIO sử dụng Factory pattern rất nhiều. Ví dụ, method `newInputStream` của class `Files` dùng để tạo object `InputStream` (static factory), method `get` của class `Paths` tạo object `Path` (static factory), method `getPath` của class `ZipFileSystem` (class thuộc package `sun.nio`, là một số implementation bên trong liên quan đến `java.nio`) tạo object `Path` (simple factory).

```java
InputStream is = Files.newInputStream(Paths.get(generatorLogoPath))
```

## Observer pattern

Dịch vụ theo dõi thư mục file trong NIO sử dụng Observer pattern.

Dịch vụ theo dõi thư mục file trong NIO dựa trên interface `WatchService` và interface `Watchable`. `WatchService` là observer, còn `Watchable` là subject được quan sát.

Interface `Watchable` định nghĩa method `register` dùng để đăng ký object vào `WatchService` (monitoring service) và gắn các event cần theo dõi.

```java
public interface Path
    extends Comparable<Path>, Iterable<Path>, Watchable{
}

public interface Watchable {
    WatchKey register(WatchService watcher,
                      WatchEvent.Kind<?>[] events,
                      WatchEvent.Modifier... modifiers)
        throws IOException;
}
```

`WatchService` dùng để theo dõi thay đổi của thư mục file. Một object `WatchService` có thể theo dõi nhiều thư mục file.

```java
// Tạo object WatchService
WatchService watchService = FileSystems.getDefault().newWatchService();

// Khởi tạo class Path của thư mục được theo dõi:
Path path = Paths.get("workingDirectory");
// Đăng ký object path vào WatchService (monitoring service)
WatchKey watchKey = path.register(
    watchService, StandardWatchEventKinds...);
```

Tham số thứ hai `events` (các event cần theo dõi) của method `register` trong class `Path` là varargs, nghĩa là chúng ta có thể theo dõi nhiều event cùng lúc.

```java
WatchKey register(WatchService watcher,
                  WatchEvent.Kind<?>... events)
    throws IOException;
```

Có 3 event theo dõi thường dùng:

- `StandardWatchEventKinds.ENTRY_CREATE`: tạo file.
- `StandardWatchEventKinds.ENTRY_DELETE` : xoá file.
- `StandardWatchEventKinds.ENTRY_MODIFY` : sửa file.

Method `register` trả về object `WatchKey`. Thông qua object `WatchKey`, có thể lấy thông tin cụ thể của event, chẳng hạn file trong thư mục đã được tạo, xoá hay sửa, và tên cụ thể của file đã được tạo, xoá hoặc sửa.

```java
WatchKey key;
while ((key = watchService.take()) != null) {
    for (WatchEvent<?> event : key.pollEvents()) {
      // Có thể gọi method của object WatchEvent để xử lý, chẳng hạn in thông tin context cụ thể của event
    }
    key.reset();
}
```

Implementation cụ thể của `WatchService` phụ thuộc vào file system và Provider bên dưới: implementation có thể trực tiếp sử dụng cơ chế file event native hoặc chuyển sang polling định kỳ. Dưới đây là source code rút gọn của implementation polling.

```java
class PollingWatchService
    extends AbstractWatchService
{
    // Định nghĩa một daemon thread (daemon thread) để polling phát hiện thay đổi file
    private final ScheduledExecutorService scheduledExecutor;

    PollingWatchService() {
        scheduledExecutor = Executors
            .newSingleThreadScheduledExecutor(new ThreadFactory() {
                 @Override
                 public Thread newThread(Runnable r) {
                     Thread t = new Thread(r);
                     t.setDaemon(true);
                     return t;
                 }});
    }

  void enable(Set<? extends WatchEvent.Kind<?>> events, long period) {
    synchronized (this) {
      // Cập nhật event theo dõi
      this.events = events;

        // Bật polling định kỳ
      Runnable thunk = new Runnable() { public void run() { poll(); }};
      this.poller = scheduledExecutor
        .scheduleAtFixedRate(thunk, period, period, TimeUnit.SECONDS);
    }
  }
}
```

## Tham khảo

- Patterns in Java APIs: <http://cecs.wright.edu/~tkprasad/courses/ceg860/paper/node26.html>
- Decorator pattern: Học Decorator pattern thông qua phân tích source code thư viện Java IO: <https://time.geekbang.org/column/article/204845>
- Package sun.nio là gì, có phải code Java không? - RednaxelaFX <https://www.zhihu.com/question/29237781/answer/43653953>

<!-- @include: @article-footer.snippet.md -->
