---
title: Giải thích chi tiết CompletableFuture
description: "Giải thích chi tiết về lập trình bất đồng bộ với CompletableFuture: trình bày toàn diện core API của CompletableFuture, điều phối task bất đồng bộ, kết hợp thenCompose/thenCombine, tổng hợp allOf/anyOf, cấu hình thread pool và best practices."
category: Java
tag:
  - Java Concurrency
head:
  - - meta
    - name: keywords
      content: CompletableFuture,lập trình bất đồng bộ,điều phối bất đồng bộ,Future,thenCompose,thenCombine,allOf,tác vụ song song
---

Trong dự án thực tế, một API có thể cần đồng thời lấy nhiều loại dữ liệu khác nhau rồi tổng hợp để trả về, đây là một trường hợp khá thường gặp. Ví dụ: khi người dùng yêu cầu lấy thông tin đơn hàng, hệ thống có thể cần đồng thời lấy thông tin người dùng, chi tiết sản phẩm, thông tin logistics, đề xuất sản phẩm và các dữ liệu khác.

Nếu thực thi tuần tự (lần lượt thực hiện từng task theo thứ tự), tốc độ phản hồi của API sẽ rất chậm. Xét thấy phần lớn các task này **không có quan hệ thứ tự trước sau**, ta có thể **thực thi song song**. Chẳng hạn, khi gọi để lấy chi tiết sản phẩm, ta có thể đồng thời gọi để lấy thông tin logistics. Bằng cách thực thi song song nhiều task, tốc độ phản hồi của API sẽ được cải thiện đáng kể.

![](https://oss.javaguide.cn/github/javaguide/high-performance/serial-to-parallel.png)

Với các task có quan hệ thứ tự gọi trước sau, ta có thể điều phối task.

![](https://oss.javaguide.cn/github/javaguide/high-performance/serial-to-parallel2.png)

1. Chỉ sau khi lấy được thông tin người dùng mới có thể gọi API chi tiết sản phẩm và thông tin logistics.
2. Chỉ sau khi lấy thành công chi tiết sản phẩm và thông tin logistics mới có thể gọi API đề xuất sản phẩm.

Các trường hợp có thể cần điều phối task bất đồng bộ bằng multi-thread (đây chỉ là ví dụ, dữ liệu không nhất thiết được trả về trong một lần, API có thể được tách ra):

1. Trang chủ: chẳng hạn, trang chủ của một cộng đồng kỹ thuật có thể cần đồng thời lấy danh sách bài viết đề xuất, banner quảng cáo, bảng xếp hạng bài viết, chủ đề nổi bật và các thông tin khác.
2. Trang chi tiết: chẳng hạn, trang chi tiết bài viết của một cộng đồng kỹ thuật có thể cần đồng thời lấy thông tin tác giả, chi tiết bài viết, bình luận bài viết và các thông tin khác.
3. Module thống kê: chẳng hạn, module thống kê backend của một cộng đồng kỹ thuật có thể cần đồng thời lấy tổng số người hâm mộ, tổng hợp dữ liệu bài viết (lượt đọc, lượt bình luận, lượt lưu) và các thông tin khác.

Đối với chương trình Java, `CompletableFuture` được giới thiệu từ Java 8 có thể giúp chúng ta điều phối nhiều task, với chức năng rất mạnh.

Bài viết này là phần nhập môn đơn giản về `CompletableFuture`, giúp bạn làm quen với các API thường dùng của `CompletableFuture`.

## Giới thiệu Future

Interface `Future` là ứng dụng điển hình của tư tưởng bất đồng bộ, chủ yếu dùng trong các trường hợp cần thực thi task tốn thời gian, tránh việc chương trình cứ phải chờ tại chỗ cho đến khi task hoàn tất khiến hiệu suất thực thi quá thấp. Cụ thể, khi thực thi một task tốn thời gian, ta có thể giao task đó cho một thread con thực thi bất đồng bộ, đồng thời làm việc khác mà không phải chờ một cách bị động. Sau khi hoàn tất công việc của mình, ta lấy kết quả thực thi của task thông qua `Future`. Nhờ đó, hiệu suất thực thi của chương trình được nâng cao rõ rệt.

Đây chính là **Future pattern** kinh điển trong multi-thread. Bạn có thể xem nó như một design pattern, với tư tưởng cốt lõi là gọi bất đồng bộ, chủ yếu được dùng trong lĩnh vực multi-thread và không riêng của ngôn ngữ Java.

Trong Java, `Future` là một generic interface nằm trong package `java.util.concurrent`. Nó có 5 abstract method kinh điển, chủ yếu gồm 4 nhóm chức năng dưới đây; từ JDK 19, `resultNow()`, `exceptionNow()` và `state()` cũng được bổ sung dưới dạng default method.

- Hủy task;
- Kiểm tra task có bị hủy hay không;
- Kiểm tra task đã thực thi xong hay chưa;
- Lấy kết quả thực thi của task.

```java
// V đại diện cho kiểu giá trị trả về của task do Future thực thi
public interface Future<V> {
    // Hủy thực thi task
    // Hủy thành công trả về true, ngược lại trả về false
    boolean cancel(boolean mayInterruptIfRunning);
    // Kiểm tra task có bị hủy hay không
    boolean isCancelled();
    // Kiểm tra task đã thực thi xong hay chưa
    boolean isDone();
    // Lấy kết quả thực thi của task
    V get() throws InterruptedException, ExecutionException;
    // Nếu không trả về kết quả tính toán trong thời gian chỉ định thì ném exception TimeoutException
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

Có thể hiểu đơn giản như sau: ta có một task và giao task đó cho `Future` xử lý. Trong thời gian task thực thi, ta có thể làm bất cứ việc gì mình muốn. Trong khoảng thời gian đó, ta cũng có thể hủy task và lấy trạng thái thực thi của task. Sau một khoảng thời gian, ta có thể trực tiếp lấy kết quả thực thi của task từ `Future`.

## Giới thiệu CompletableFuture

Trong quá trình sử dụng thực tế, `Future` có một số hạn chế, chẳng hạn không hỗ trợ kết hợp điều phối task bất đồng bộ, còn method `get()` để lấy kết quả tính toán là một blocking call.

Class `CompletableFuture` được giới thiệu từ Java 8 có thể giải quyết các thiếu sót này của `Future`. Ngoài việc cung cấp các tính năng `Future` tiện dụng và mạnh mẽ hơn, `CompletableFuture` còn cung cấp khả năng lập trình hàm, kết hợp điều phối task bất đồng bộ (có thể nối nhiều task bất đồng bộ thành một chain call hoàn chỉnh) và các khả năng khác.

Dưới đây, chúng ta cùng xem nhanh định nghĩa của class `CompletableFuture`.

```java
public class CompletableFuture<T> implements Future<T>, CompletionStage<T> {
}
```

Có thể thấy `CompletableFuture` đồng thời implement interface `Future` và `CompletionStage`.

![](https://oss.javaguide.cn/github/javaguide/java/concurrent/completablefuture-class-diagram.jpg)

Ngoài việc cung cấp các tính năng `Future` tiện dụng và mạnh mẽ hơn, `CompletableFuture` còn cung cấp khả năng lập trình hàm.

![](https://oss.javaguide.cn/javaguide/image-20210902092441434.png)

Interface `Future` có 5 method:

- `boolean cancel(boolean mayInterruptIfRunning)`: thử hủy thực thi task.
- `boolean isCancelled()`: kiểm tra task có bị hủy hay không.
- `boolean isDone()`: kiểm tra task đã thực thi xong hay chưa.
- `get()`: chờ task thực thi xong và lấy kết quả tính toán.
- `get(long timeout, TimeUnit unit)`: bổ sung thời gian timeout.

Interface `CompletionStage` mô tả một giai đoạn của phép tính bất đồng bộ. Nhiều phép tính có thể được chia thành nhiều giai đoạn hoặc bước; khi đó có thể dùng nó để kết hợp tất cả bước, tạo thành một pipeline tính toán bất đồng bộ.

Interface `CompletionStage` có khá nhiều method, khả năng lập trình hàm của `CompletableFuture` được interface này cung cấp. Từ các tham số method của interface này, bạn có thể nhận thấy chúng sử dụng rất nhiều tính năng lập trình hàm được Java 8 giới thiệu.

![](https://oss.javaguide.cn/javaguide/image-20210902093026059.png)

Do có nhiều method nên không thể giải thích lần lượt từng method ở đây; phần dưới sẽ giới thiệu cách sử dụng phần lớn các method thường gặp.

## Các thao tác thường gặp với CompletableFuture

### Tạo CompletableFuture

Các cách thường gặp để tạo object `CompletableFuture` như sau:

1. Dùng keyword `new`.
2. Dựa trên static factory method có sẵn của `CompletableFuture`: `runAsync()`, `supplyAsync()`.

#### Keyword new

Cách tạo object `CompletableFuture` bằng keyword `new` có thể được xem là dùng `CompletableFuture` như một `Future`.

Trong open source project [guide-rpc-framework](https://github.com/Snailclimb/guide-rpc-framework) của mình, tôi cũng tạo object `CompletableFuture` theo cách này.

Cùng xem một ví dụ đơn giản.

Ta tạo một `CompletableFuture` có kiểu giá trị kết quả là `RpcResponse<Object>`, bạn có thể xem `resultFuture` như vật mang kết quả của phép tính bất đồng bộ.

```java
CompletableFuture<RpcResponse<Object>> resultFuture = new CompletableFuture<>();
```

Giả sử tại một thời điểm nào đó trong tương lai, ta nhận được kết quả cuối cùng. Khi đó, ta có thể gọi method `complete()` và truyền kết quả vào, biểu thị `resultFuture` đã hoàn tất.

```java
// Method complete() chỉ có thể được gọi một lần, các lần gọi sau sẽ bị bỏ qua.
resultFuture.complete(rpcResponse);
```

Bạn có thể dùng method `isDone()` để kiểm tra đã hoàn tất hay chưa.

```java
public boolean isDone() {
    return result != null;
}
```

Lấy kết quả tính toán bất đồng bộ cũng rất đơn giản, chỉ cần gọi trực tiếp method `get()`. Thread gọi method `get()` sẽ blocking cho đến khi `CompletableFuture` hoàn tất phép tính.

```java
rpcResponse = completableFuture.get();
```

Nếu đã biết kết quả tính toán, bạn có thể dùng static method `completedFuture()` để tạo `CompletableFuture`.

```java
CompletableFuture<String> future = CompletableFuture.completedFuture("hello!");
assertEquals("hello!", future.get());
```

Bên trong, method `completedFuture()` gọi method `new` có tham số, chỉ khác là method này không public.

```java
public static <U> CompletableFuture<U> completedFuture(U value) {
    return new CompletableFuture<U>((value == null) ? NIL : value);
}
```

#### Static factory method

Hai method này có thể giúp ta đóng gói logic tính toán.

```java
static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier);
// Dùng thread pool tùy chỉnh (khuyến nghị)
static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor);
static CompletableFuture<Void> runAsync(Runnable runnable);
// Dùng thread pool tùy chỉnh (khuyến nghị)
static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor);
```

Tham số mà method `runAsync()` nhận là `Runnable`, đây là một functional interface và không cho phép trả về giá trị. Khi cần thao tác bất đồng bộ nhưng không quan tâm đến kết quả trả về, bạn có thể dùng method `runAsync()`.

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

Method `supplyAsync()` nhận tham số là `Supplier<U>`, đây cũng là một functional interface, trong đó `U` là kiểu của giá trị kết quả trả về.

```java
@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```

Khi cần thao tác bất đồng bộ và quan tâm đến kết quả trả về, bạn có thể dùng method `supplyAsync()`.

```java
CompletableFuture<Void> future = CompletableFuture.runAsync(() -> System.out.println("hello!"));
future.get();// In ra "hello!"
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "hello!");
assertEquals("hello!", future2.get());
```

### Xử lý kết quả hoàn tất bất đồng bộ

Sau khi lấy được kết quả tính toán bất đồng bộ, ta còn có thể xử lý tiếp. Một số method thường dùng:

- `thenApply()`
- `thenAccept()`
- `thenRun()`
- `whenComplete()`

Method `thenApply()` nhận một instance `Function` để xử lý kết quả.

```java
// Callback không bất đồng bộ: có thể do thread hoàn tất stage trước đó thực thi; nếu stage đã hoàn tất, cũng có thể do thread đang gọi hiện tại thực thi
public <U> CompletableFuture<U> thenApply(
    Function<? super T,? extends U> fn) {
    return uniApplyStage(null, fn);
}

// Dùng thread pool ForkJoinPool mặc định (không khuyến nghị)
public <U> CompletableFuture<U> thenApplyAsync(
    Function<? super T,? extends U> fn) {
    return uniApplyStage(defaultExecutor(), fn);
}
// Dùng thread pool tùy chỉnh (khuyến nghị)
public <U> CompletableFuture<U> thenApplyAsync(
    Function<? super T,? extends U> fn, Executor executor) {
    return uniApplyStage(screenExecutor(executor), fn);
}
```

Ví dụ sử dụng method `thenApply()`:

```java
CompletableFuture<String> future = CompletableFuture.completedFuture("hello!")
        .thenApply(s -> s + "world!");
assertEquals("hello!world!", future.get());
// Lần gọi này sẽ bị bỏ qua.
future.thenApply(s -> s + "nice!");
assertEquals("hello!world!", future.get());
```

Bạn cũng có thể gọi **theo chain**:

```java
CompletableFuture<String> future = CompletableFuture.completedFuture("hello!")
        .thenApply(s -> s + "world!").thenApply(s -> s + "nice!");
assertEquals("hello!world!nice!", future.get());
```

**Nếu không cần lấy kết quả trả về từ callback function, bạn có thể dùng `thenAccept()` hoặc `thenRun()`. Điểm khác nhau giữa hai method này là `thenRun()` không thể truy cập kết quả tính toán bất đồng bộ.**

Tham số của method `thenAccept()` là `Consumer<? super T>`.

```java
public CompletableFuture<Void> thenAccept(Consumer<? super T> action) {
    return uniAcceptStage(null, action);
}

public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action) {
    return uniAcceptStage(defaultExecutor(), action);
}

public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action,
                                               Executor executor) {
    return uniAcceptStage(screenExecutor(executor), action);
}
```

Đúng như tên gọi, `Consumer` là interface kiểu consumer, có thể nhận một object input rồi thực hiện "tiêu thụ" object đó.

```java
@FunctionalInterface
public interface Consumer<T> {

    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

Tham số của method `thenRun()` là `Runnable`.

```java
public CompletableFuture<Void> thenRun(Runnable action) {
    return uniRunStage(null, action);
}

public CompletableFuture<Void> thenRunAsync(Runnable action) {
    return uniRunStage(defaultExecutor(), action);
}

public CompletableFuture<Void> thenRunAsync(Runnable action,
                                            Executor executor) {
    return uniRunStage(screenExecutor(executor), action);
}
```

Ví dụ sử dụng `thenAccept()` và `thenRun()`:

```java
CompletableFuture.completedFuture("hello!")
        .thenApply(s -> s + "world!").thenApply(s -> s + "nice!").thenAccept(System.out::println);//hello!world!nice!

CompletableFuture.completedFuture("hello!")
        .thenApply(s -> s + "world!").thenApply(s -> s + "nice!").thenRun(() -> System.out.println("hello!"));//hello!
```

Tham số của method `whenComplete()` là `BiConsumer<? super T, ? super Throwable>`.

```java
public CompletableFuture<T> whenComplete(
    BiConsumer<? super T, ? super Throwable> action) {
    return uniWhenCompleteStage(null, action);
}


public CompletableFuture<T> whenCompleteAsync(
    BiConsumer<? super T, ? super Throwable> action) {
    return uniWhenCompleteStage(defaultExecutor(), action);
}
// Dùng thread pool tùy chỉnh (khuyến nghị)
public CompletableFuture<T> whenCompleteAsync(
    BiConsumer<? super T, ? super Throwable> action, Executor executor) {
    return uniWhenCompleteStage(screenExecutor(executor), action);
}
```

So với `Consumer`, `BiConsumer` có thể nhận 2 object input rồi thực hiện "tiêu thụ" chúng.

```java
@FunctionalInterface
public interface BiConsumer<T, U> {
    void accept(T t, U u);

    default BiConsumer<T, U> andThen(BiConsumer<? super T, ? super U> after) {
        Objects.requireNonNull(after);

        return (l, r) -> {
            accept(l, r);
            after.accept(l, r);
        };
    }
}
```

Ví dụ sử dụng `whenComplete()`:

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "hello!")
        .whenComplete((res, ex) -> {
            // res đại diện cho kết quả trả về
            // ex có kiểu Throwable, đại diện cho exception được ném ra
            System.out.println(res);
            // Không có exception được ném ra nên có giá trị null
            assertNull(ex);
        });
assertEquals("hello!", future.get());
```

### Xử lý exception

Bạn có thể dùng method `handle()` để xử lý trường hợp có thể phát sinh exception trong quá trình thực thi task.

```java
public <U> CompletableFuture<U> handle(
    BiFunction<? super T, Throwable, ? extends U> fn) {
    return uniHandleStage(null, fn);
}

public <U> CompletableFuture<U> handleAsync(
    BiFunction<? super T, Throwable, ? extends U> fn) {
    return uniHandleStage(defaultExecutor(), fn);
}

public <U> CompletableFuture<U> handleAsync(
    BiFunction<? super T, Throwable, ? extends U> fn, Executor executor) {
    return uniHandleStage(screenExecutor(executor), fn);
}
```

Code ví dụ:

```java
CompletableFuture<String> future
        = CompletableFuture.supplyAsync(() -> {
    if (true) {
        throw new RuntimeException("Computation error!");
    }
    return "hello!";
}).handle((res, ex) -> {
    // res đại diện cho kết quả trả về
    // ex có kiểu Throwable, đại diện cho exception được ném ra
    return res != null ? res : "world!";
});
assertEquals("world!", future.get());
```

Bạn cũng có thể dùng method `exceptionally()` để xử lý trường hợp exception.

```java
CompletableFuture<String> future
        = CompletableFuture.supplyAsync(() -> {
    if (true) {
        throw new RuntimeException("Computation error!");
    }
    return "hello!";
}).exceptionally(ex -> {
    System.out.println(ex.toString());// CompletionException
    return "world!";
});
assertEquals("world!", future.get());
```

Nếu muốn kết quả của `CompletableFuture` chính là exception, bạn có thể dùng method `completeExceptionally()` để gán giá trị cho nó.

```java
CompletableFuture<String> completableFuture = new CompletableFuture<>();
// ...
completableFuture.completeExceptionally(
  new RuntimeException("Calculation failed!"));
// ...
completableFuture.get(); // ExecutionException
```

### Kết hợp CompletableFuture

Bạn có thể dùng `thenCompose()` để nối hai object `CompletableFuture` theo thứ tự, tạo thành task chain bất đồng bộ. Tác dụng của method này là dùng kết quả trả về của task trước làm tham số input cho task sau, từ đó hình thành quan hệ phụ thuộc.

```java
public <U> CompletableFuture<U> thenCompose(
    Function<? super T, ? extends CompletionStage<U>> fn) {
    return uniComposeStage(null, fn);
}

public <U> CompletableFuture<U> thenComposeAsync(
    Function<? super T, ? extends CompletionStage<U>> fn) {
    return uniComposeStage(defaultExecutor(), fn);
}

public <U> CompletableFuture<U> thenComposeAsync(
    Function<? super T, ? extends CompletionStage<U>> fn,
    Executor executor) {
    return uniComposeStage(screenExecutor(executor), fn);
}
```

Ví dụ sử dụng method `thenCompose()`:

```java
CompletableFuture<String> future
        = CompletableFuture.supplyAsync(() -> "hello!")
        .thenCompose(s -> CompletableFuture.supplyAsync(() -> s + "world!"));
assertEquals("hello!world!", future.get());
```

Trong phát triển thực tế, method này rất hữu ích. Chẳng hạn, task1 và task2 đều được thực thi bất đồng bộ, nhưng task1 phải thực thi xong thì task2 mới có thể bắt đầu (task2 phụ thuộc vào kết quả thực thi của task1).

Tương tự method `thenCompose()`, còn có method `thenCombine()`, cũng dùng để kết hợp hai object `CompletableFuture`.

```java
CompletableFuture<String> completableFuture
        = CompletableFuture.supplyAsync(() -> "hello!")
        .thenCombine(CompletableFuture.supplyAsync(
                () -> "world!"), (s1, s2) -> s1 + s2)
        .thenCompose(s -> CompletableFuture.supplyAsync(() -> s + "nice!"));
assertEquals("hello!world!nice!", completableFuture.get());
```

**Vậy `thenCompose()` và `thenCombine()` khác nhau thế nào?**

- `thenCompose()` có thể nối hai object `CompletableFuture` và dùng kết quả trả về của task trước làm tham số cho task sau, giữa chúng tồn tại thứ tự trước sau.
- `thenCombine()` sẽ hợp nhất kết quả của hai stage sau khi cả hai stage đều hoàn tất bình thường. Hai stage có thể độc lập với nhau, nhưng có thực thi song song hay không phụ thuộc vào cách tạo chúng và executor được sử dụng; bản thân `thenCombine()` không chịu trách nhiệm khởi động task.

Ngoài `thenCompose()` và `thenCombine()`, còn có một số method khác để kết hợp `CompletableFuture` nhằm tạo ra các hiệu quả khác nhau, đáp ứng các nhu cầu nghiệp vụ khác nhau.

Ví dụ, khi task1 và task2 đều hoàn tất bình thường, có thể dùng `acceptEither()` để task3 nhận kết quả của task hoàn tất trước trong hai task. Cần lưu ý rằng method này không phải selector đáng tin cậy cho "kết quả thành công đầu tiên": chỉ cần một trong hai stage hoàn tất bất thường, kết quả của stage trả về sẽ tuân theo quy tắc exception của `CompletionStage` đối với tổ hợp either.

```java
public CompletableFuture<Void> acceptEither(
    CompletionStage<? extends T> other, Consumer<? super T> action) {
    return orAcceptStage(null, other, action);
}

public CompletableFuture<Void> acceptEitherAsync(
    CompletionStage<? extends T> other, Consumer<? super T> action) {
    return orAcceptStage(asyncPool, other, action);
}
```

Một ví dụ đơn giản:

```java
CompletableFuture<String> task = CompletableFuture.supplyAsync(() -> {
    System.out.println("Task 1 bắt đầu thực thi, thời gian hiện tại: " + System.currentTimeMillis());
    try {
        Thread.sleep(500);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("Task 1 thực thi xong, thời gian hiện tại: " + System.currentTimeMillis());
    return "task1";
});

CompletableFuture<String> task2 = CompletableFuture.supplyAsync(() -> {
    System.out.println("Task 2 bắt đầu thực thi, thời gian hiện tại: " + System.currentTimeMillis());
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("Task 2 thực thi xong, thời gian hiện tại: " + System.currentTimeMillis());
    return "task2";
});

task.acceptEitherAsync(task2, (res) -> {
    System.out.println("Task 3 bắt đầu thực thi, thời gian hiện tại: " + System.currentTimeMillis());
    System.out.println("Kết quả của task trước: " + res);
});

// Thêm một khoảng delay để đảm bảo task bất đồng bộ có đủ thời gian hoàn tất
try {
    Thread.sleep(2000);
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

Output:

```plain
Task 1 bắt đầu thực thi, thời gian hiện tại: 1695088058520
Task 2 bắt đầu thực thi, thời gian hiện tại: 1695088058521
Task 1 thực thi xong, thời gian hiện tại: 1695088059023
Task 3 bắt đầu thực thi, thời gian hiện tại: 1695088059023
Kết quả của task trước: task1
Task 2 thực thi xong, thời gian hiện tại: 1695088059523
```

Khi cả hai stage đều hoàn tất bình thường, `acceptEitherAsync()` sẽ dùng kết quả của một stage đã hoàn tất để thực thi bất đồng bộ task 3, thường là kết quả của stage hoàn tất trước. Tuy nhiên, nếu một stage hoàn tất bất thường còn stage kia chưa hoàn tất hoặc hoàn tất bình thường, đặc tả không đảm bảo stage trả về cuối cùng chắc chắn hoàn tất bình thường hay bất thường. Vì vậy, không thể dựa vào method này để bỏ qua exception xảy ra trước và tiếp tục chờ kết quả thành công từ stage còn lại.

### Chờ nhiều CompletableFuture hoàn tất

Bạn có thể dùng static method `allOf()` của `CompletableFuture` để chờ nhiều `CompletableFuture` hoàn tất toàn bộ. `allOf()` chỉ kết hợp trạng thái hoàn tất của các stage đã tồn tại, không chịu trách nhiệm khởi động các task này; task có thực thi song song hay không phụ thuộc vào cách tạo chúng và executor.

Trong dự án thực tế, ta thường cần chạy song song nhiều task không liên quan với nhau. Các task này không có quan hệ phụ thuộc và có thể chạy độc lập.

Ví dụ, ta cần đọc và xử lý 6 file. 6 task này không có sự phụ thuộc về thứ tự thực thi, nhưng khi trả về cho người dùng, ta cần thống kê và tổng hợp kết quả xử lý của các file. Trong trường hợp này, ta có thể dùng nhiều `CompletableFuture` chạy song song để xử lý.

Code ví dụ:

```java
CompletableFuture<Void> task1 =
  CompletableFuture.supplyAsync(()->{
    // Thao tác nghiệp vụ tùy chỉnh
  });
......
CompletableFuture<Void> task6 =
  CompletableFuture.supplyAsync(()->{
    // Thao tác nghiệp vụ tùy chỉnh
  });
......
 CompletableFuture<Void> headerFuture=CompletableFuture.allOf(task1,.....,task6);

   try {
     headerFuture.join();
   } catch (Exception ex) {
     ......
   }
System.out.println("all done. ");
```

Method thường được đem ra so sánh với `allOf()` là `anyOf()`.

**Method `allOf()` sẽ chờ tất cả `CompletableFuture` thực thi xong rồi mới trả về.**

```java
Random rand = new Random();
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(1000 + rand.nextInt(1000));
    } catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        System.out.println("future1 hoàn tất...");
    }
    return "abc";
});
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(1000 + rand.nextInt(1000));
    } catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        System.out.println("future2 hoàn tất...");
    }
    return "efg";
});
```

Gọi `join()` có thể khiến chương trình tiếp tục thực thi sau khi cả `future1` và `future2` đều chạy xong.

```java
CompletableFuture<Void> completableFuture = CompletableFuture.allOf(future1, future2);
completableFuture.join();
assertTrue(completableFuture.isDone());
System.out.println("tất cả future đã hoàn tất...");
```

Output:

```plain
future1 hoàn tất...
future2 hoàn tất...
tất cả future đã hoàn tất...
```

**Method `anyOf()` không chờ tất cả `CompletableFuture` thực thi xong rồi mới trả về, chỉ cần một future hoàn tất là đủ!**

```java
CompletableFuture<Object> f = CompletableFuture.anyOf(future1, future2);
System.out.println(f.get());
```

Output có thể là:

```plain
future2 hoàn tất...
efg
```

Hoặc cũng có thể là:

```plain
future1 hoàn tất...
abc
```

## Khuyến nghị khi sử dụng CompletableFuture

### Sử dụng thread pool tùy chỉnh

Trong các code ví dụ phía trên, để thuận tiện, ta không chọn thread pool tùy chỉnh. Trong dự án thực tế, đây là điều không nên.

Trong implementation mặc định của `CompletableFuture`, các method bất đồng bộ không truyền `Executor` một cách tường minh thường sử dụng `ForkJoinPool.commonPool()` dùng chung trên toàn cục; subclass có thể thay đổi executor mặc định của các method bất đồng bộ non-static bằng cách override `defaultExecutor()`. Điều này có nghĩa là nếu application, nhiều library hoặc framework cùng dùng implementation mặc định, các task bất đồng bộ liên quan thường sẽ dùng chung một thread pool.

Mặc dù `ForkJoinPool` có hiệu suất rất cao, khi đồng thời submit một lượng lớn task, nó có thể gây tranh chấp tài nguyên và thread starvation, từ đó ảnh hưởng đến performance của hệ thống.

Để tránh các vấn đề này, nên cung cấp thread pool tùy chỉnh cho `CompletableFuture`, với các ưu điểm sau:

- **Tính cô lập**: phân bổ thread pool riêng cho các task khác nhau, tránh tranh chấp tài nguyên của thread pool toàn cục.
- **Kiểm soát tài nguyên**: điều chỉnh kích thước thread pool và kiểu queue theo đặc tính của task để tối ưu performance.
- **Xử lý exception**: xử lý tốt hơn các exception trong thread thông qua custom `ThreadFactory`.

```java
private ThreadPoolExecutor executor = new ThreadPoolExecutor(10, 10,
        0L, TimeUnit.MILLISECONDS,
        new LinkedBlockingQueue<Runnable>());

CompletableFuture.runAsync(() -> {
     //...
}, executor);
```

### Hạn chế sử dụng get()

Method `get()` của `CompletableFuture` là blocking, nên hạn chế sử dụng. Nếu bắt buộc phải dùng, cần thêm thời gian timeout, nếu không có thể khiến main thread phải chờ mãi và không thể thực thi task khác.

```java
    CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
        try {
            Thread.sleep(10_000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "Hello, world!";
    });

    // Lấy giá trị trả về của task bất đồng bộ, đặt thời gian timeout là 5 giây
    try {
        String result = future.get(5, TimeUnit.SECONDS);
        System.out.println(result);
    } catch (InterruptedException | ExecutionException | TimeoutException e) {
        // Xử lý exception
        e.printStackTrace();
    }
}
```

Đoạn code trên ném exception `TimeoutException` khi gọi `get()`. Nhờ đó, ta có thể thực hiện thao tác tương ứng trong phần xử lý exception, chẳng hạn hủy task, retry task, ghi log và các thao tác khác.

### Xử lý exception đúng cách

Khi sử dụng `CompletableFuture`, nhất định phải xử lý exception đúng cách để tránh mất exception hoặc phát sinh vấn đề không thể kiểm soát.

Dưới đây là một số khuyến nghị:

- `whenComplete` thực thi callback khi stage hoàn tất bình thường hoặc bất thường, phù hợp để quan sát kết quả và ghi lại exception; nó mặc định giữ nguyên kết quả hoặc exception của stage ban đầu, không dùng để chuyển exception thành kết quả bình thường.
- `exceptionally` chỉ thực thi khi stage hoàn tất bất thường và dùng giá trị trả về của callback để khôi phục thành kết quả bình thường; nếu cần tiếp tục truyền exception, có thể tường minh ném exception trong callback.
- `handle` luôn thực thi bất kể stage hoàn tất bình thường hay bất thường, đồng thời tạo kết quả mới dựa trên kết quả và exception.
- `CompletableFuture.allOf` có thể chờ nhiều stage hoàn tất toàn bộ; chỉ cần một stage hoàn tất bất thường thì `CompletableFuture` trả về cũng hoàn tất bất thường, nhưng vẫn cần kiểm tra riêng từng stage để lấy kết quả hoặc exception của từng task.
- …

### Kết hợp hợp lý nhiều task bất đồng bộ

Sử dụng đúng các method `thenCompose()`, `thenCombine()`, `acceptEither()`, `allOf()`, `anyOf()` và các method khác để kết hợp nhiều task bất đồng bộ, đáp ứng nhu cầu nghiệp vụ thực tế và nâng cao hiệu suất thực thi của chương trình.

Trong thực tế sử dụng, ta cũng có thể sử dụng hoặc tham khảo các framework điều phối task bất đồng bộ có sẵn, chẳng hạn [asyncTool](https://gitee.com/jd-platform-opensource/asyncTool) của JD.

![Tài liệu README của asyncTool](https://oss.javaguide.cn/github/javaguide/java/concurrent/asyncTool-readme.png)

## Lời kết

Bài viết này chỉ giới thiệu đơn giản các khái niệm cốt lõi và một số API thường dùng của `CompletableFuture`. Nếu muốn học sâu hơn, bạn cũng có thể tìm đọc thêm một số sách và blog; chẳng hạn, một vài bài viết dưới đây khá hữu ích:

- [Nguyên lý và thực tiễn CompletableFuture - Bất đồng bộ hóa API phía merchant giao đồ ăn - Đội ngũ kỹ thuật Meituan](https://tech.meituan.com/2022/05/12/principles-and-practices-of-completablefuture.html): bài viết này giới thiệu chi tiết việc sử dụng `CompletableFuture` trong dự án thực tế. Tham khảo bài viết này, bạn có thể tối ưu các trường hợp tương tự trong dự án, đây cũng có thể xem là một điểm cộng nhỏ. Cách tối ưu performance này tương đối đơn giản mà hiệu quả cũng khá tốt!
- [Đọc source code RocketMQ, học ba công cụ lớn của lập trình concurrent - Chia sẻ thực chiến Java của Yong Ge](https://mp.weixin.qq.com/s/32Ak-WFLynQfpn0Cg0N-0A): bài viết này giới thiệu ứng dụng `CompletableFuture` trong RocketMQ. Cụ thể, từ RocketMQ 4.7, RocketMQ đã đưa `CompletableFuture` vào để thực hiện xử lý message bất đồng bộ.

Ngoài ra, các bạn G cũng nên xem framework concurrent [asyncTool](https://gitee.com/jd-platform-opensource/asyncTool) của JD, trong đó sử dụng rất nhiều `CompletableFuture`.

<!-- @include: @article-footer.snippet.md -->
