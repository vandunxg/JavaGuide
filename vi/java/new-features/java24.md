---
title: Tổng quan các tính năng mới của Java 24
description: Tổng hợp các tính năng mới và thay đổi của JDK 24, thuận tiện theo dõi quá trình phát triển của Java.
category: Java
tag:
  - Tính năng mới của Java
head:
  - - meta
    - name: keywords
      content: Java 24,JDK24,cập nhật JEP,tính năng ngôn ngữ,cải tiến GC,tăng cường nền tảng
---

JDK 24 được phát hành vào tháng 3 năm 2025, đây là một phiên bản không phải LTS (bản hỗ trợ dài hạn). Phiên bản tiếp theo là phiên bản LTS **JDK 25**, được phát hành vào tháng 9 năm 2025.

JDK 24 có tổng cộng 24 tính năng mới. Bài viết này chọn một số tính năng mới quan trọng để giới thiệu chi tiết:

- [JEP 478: Key Derivation Function API (API hàm dẫn xuất khóa, preview)](https://openjdk.org/jeps/478)
- [JEP 483: Early Class-File Loading & Linking (tải và liên kết Class File sớm)](https://openjdk.org/jeps/483)
- [JEP 484: Class File API (API Class File)](https://openjdk.org/jeps/484)
- [JEP 485: Stream Gatherers (Stream Gatherer)](https://openjdk.org/jeps/485)
- [JEP 486: Disable the Security Manager (vô hiệu hóa vĩnh viễn Security Manager)](https://openjdk.org/jeps/486)
- [JEP 487: Scoped Values (Scoped Value, preview lần thứ tư)](https://openjdk.org/jeps/487)
- [JEP 495: Simplified Source Files and Instance Main Methods (Source File và Instance Main Method đơn giản hóa, preview lần thứ tư)](https://openjdk.org/jeps/495)
- [JEP 497: Quantum-Resistant Digital Signature Algorithm (ML-DSA) (thuật toán chữ ký số kháng lượng tử)](https://openjdk.org/jeps/497)
- [JEP 499: Structured Concurrency (Structured Concurrency, preview lần thứ tư)](https://openjdk.org/jeps/499)

Hình dưới đây cho thấy số lượng tính năng mới và thời điểm cập nhật của từng phiên bản từ JDK 8 đến JDK 25:

![](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

## JEP 478: Key Derivation Function API (API hàm dẫn xuất khóa, preview)

Key Derivation Function API cung cấp một nhóm interface tiêu chuẩn để dẫn xuất các khóa bổ sung từ khóa ban đầu và dữ liệu khác. API này có thể tạo nhiều khóa khác nhau cho các mục đích mã hóa khác nhau, chẳng hạn như mã hóa và xác thực, tránh sử dụng trực tiếp cùng một khóa nhiều lần. API này đang ở giai đoạn preview trong JDK 24.

Thông qua API này, developer có thể sử dụng các thuật toán dẫn xuất khóa do security provider triển khai, chẳng hạn như HKDF:

```java
// Tạo một đối tượng KDF, sử dụng thuật toán HKDF-SHA256
KDF hkdf = KDF.getInstance("HKDF-SHA256");

// Tạo parameter spec cho Extract và Expand
AlgorithmParameterSpec params =
    HKDFParameterSpec.ofExtract()
                     .addIKM(initialKeyMaterial) // Thiết lập key material ban đầu
                     .addSalt(salt)             // Thiết lập salt
                     .thenExpand(info, 32);     // Thiết lập thông tin mở rộng và độ dài đích

// Dẫn xuất một khóa AES dài 32 byte
SecretKey key = hkdf.deriveKey("AES", params);

// Có thể sử dụng cùng đối tượng KDF để thực hiện các thao tác dẫn xuất khóa khác
```

## JEP 483: Early Class-File Loading & Linking (tải và liên kết Class File sớm)

Trong JVM truyền thống, ứng dụng phải tải và liên kết class một cách dynamic mỗi khi khởi động. Tính năng này cache các class đã được tải và liên kết, giảm công việc lặp lại trong những lần khởi động sau. Trong benchmark Spring PetClinic do JEP 483 cung cấp, thời gian khởi động sau khi sử dụng AOT cache được rút ngắn tối đa khoảng 42%; lợi ích thực tế phụ thuộc vào ứng dụng và môi trường chạy.

Tối ưu hóa này không yêu cầu sửa code của ứng dụng, library hoặc framework, nhưng trước tiên cần ghi lại một lần chạy training và tạo AOT cache, sau đó load cache này khi khởi động chính thức. Các parameter liên quan gồm `-XX:AOTMode=record`, `-XX:AOTConfiguration`, `-XX:AOTMode=create` và `-XX:AOTCache`.

## JEP 484: Class File API (API Class File)

Class File API đã có preview lần đầu trong JDK 22 ([JEP 457](https://openjdk.org/jeps/457)), preview lần thứ hai trong JDK 23 và được hoàn thiện thêm ([JEP 466](https://openjdk.org/jeps/466)). Cuối cùng, tính năng này đã chính thức trở thành tính năng chuẩn trong JDK 24.

Mục tiêu của Class File API là cung cấp một nhóm API tiêu chuẩn hóa để phân tích, tạo và chuyển đổi Java Class File, thay thế sự phụ thuộc trước đây vào các library bên thứ ba như ASM trong việc xử lý Class File.

```java
// Tạo một đối tượng ClassFile, đây là entry point để thao tác với Class File.
ClassFile cf = ClassFile.of();
// Phân tích mảng byte thành ClassModel
ClassModel classModel = cf.parse(bytes);

// Xây dựng Class File mới, xóa mọi method bắt đầu bằng "debug"
byte[] newBytes = cf.build(classModel.thisClass().asSymbol(),
        classBuilder -> {
            // Duyệt qua mọi class element
            for (ClassElement ce : classModel) {
                // Kiểm tra có phải method và tên method có bắt đầu bằng "debug" hay không
                if (!(ce instanceof MethodModel mm
                        && mm.methodName().stringValue().startsWith("debug"))) {
                    // Thêm vào Class File mới
                    classBuilder.with(ce);
                }
            }
        });
```

## JEP 485: Stream Gatherers (Stream Gatherer)

Stream Gatherer `Stream::gather(Gatherer)` là một tính năng mới mạnh mẽ, cho phép developer định nghĩa các intermediate operation tùy chỉnh để thực hiện chuyển đổi dữ liệu phức tạp và linh hoạt hơn. Interface `Gatherer` là thành phần cốt lõi của tính năng này. Nó định nghĩa cách thu thập element từ stream, duy trì state trung gian và tạo result trong quá trình xử lý.

Khác với các operation tích hợp như `filter`, `map` hoặc `distinct`, `Stream::gather` cho phép developer thực hiện những task khó hoàn thành bằng các operation Stream tiêu chuẩn. Ví dụ, có thể dùng `Stream::gather` để thực hiện sliding window, loại bỏ phần tử trùng lặp theo rule tùy chỉnh hoặc chuyển đổi và aggregate state phức tạp hơn. Tính linh hoạt này mở rộng đáng kể phạm vi ứng dụng của Stream API, giúp developer xử lý các scenario xử lý dữ liệu phức tạp hơn.

Logic loại bỏ trùng lặp theo độ dài chuỗi được triển khai dựa trên `Stream::gather(Gatherer)`:

```java
var result = Stream.of("foo", "bar", "baz", "quux")
                   .gather(Gatherer.ofSequential(
                       HashSet::new, // Khởi tạo state là HashSet, dùng để lưu độ dài các chuỗi đã gặp
                       (set, str, downstream) -> {
                           if (set.add(str.length())) {
                               return downstream.push(str);
                           }
                           return true; // Tiếp tục xử lý stream
                       }
                   ))
                   .toList();// Chuyển đổi thành list

// Kết quả ==> [foo, quux]
```

## JEP 486: Disable the Security Manager (vô hiệu hóa vĩnh viễn Security Manager)

JDK 24 không còn cho phép bật `Security Manager`. Ngay cả khi sử dụng lệnh `java -Djava.security.manager` cũng không thể bật nó. Đây là một bước quan trọng trong quá trình loại bỏ dần tính năng này. Mặc dù `Security Manager` từng là công cụ quan trọng trong Java để hạn chế quyền của code, chẳng hạn quyền truy cập file system hoặc network, đọc hoặc ghi file nhạy cảm và thực thi system command, cộng đồng Java quyết định loại bỏ hoàn toàn nó do tính phức tạp cao, mức độ sử dụng thấp và chi phí bảo trì lớn.

## JEP 487: Scoped Values (Scoped Value, preview lần thứ tư)

Scoped Value có thể chia sẻ dữ liệu immutable giữa các thread và trong cùng một thread, có ưu thế hơn ThreadLocal, đặc biệt khi sử dụng nhiều virtual thread.

```java
final static ScopedValue<...> V = ScopedValue.newInstance();

// Trong một method nào đó
ScopedValue.where(V, <value>)
           .run(() -> { ... V.get() ... call methods ... });

// Trong một method được gọi trực tiếp hoặc gián tiếp từ lambda expression
... V.get() ...
```

Scoped Value cho phép chia sẻ dữ liệu an toàn và hiệu quả giữa các component trong chương trình lớn mà không cần truyền qua parameter của method.

## JEP 491: Virtual Threads Synchronization Without Pinning (synchronization của virtual thread không pin platform thread)

Cơ chế hoạt động giữa virtual thread và `synchronized` được tối ưu hóa. Khi virtual thread bị block trong method hoặc block `synchronized`, thông thường nó có thể giải phóng thread của hệ điều hành mà nó đang chiếm dụng (platform thread), tránh chiếm dụng platform thread trong thời gian dài và nhờ đó tăng khả năng concurrency của ứng dụng. Cơ chế này tránh hiện tượng “pinning”, tức virtual thread chiếm dụng platform thread trong thời gian dài và ngăn platform thread phục vụ các virtual thread khác.

Code Java hiện có sử dụng `synchronized` không cần sửa vẫn có thể hưởng lợi từ khả năng mở rộng của virtual thread. Ví dụ, một ứng dụng I/O-intensive nếu sử dụng platform thread truyền thống có thể bị giảm khả năng concurrency do thread bị block. Ngược lại, khi sử dụng virtual thread, ngay cả khi xảy ra block trong block `synchronized`, platform thread cũng không bị pin, cho phép platform thread tiếp tục phục vụ các virtual thread khác và cải thiện performance concurrency tổng thể.

## JEP 493: Linking Run-Time Images Without JMOD Files (liên kết run-time image không cần file JMOD)

Theo mặc định, JDK đồng thời bao gồm run-time image (các module cần thiết khi chạy) và file JMOD. Tính năng này cho phép tool jlink tạo run-time image tùy chỉnh mà không cần sử dụng file JMOD của JDK, làm giảm dung lượng cài đặt JDK khoảng 25%.

Giải thích:

- Jlink là command-line tool mới được phát hành cùng Java 9. Nó cho phép developer tạo JRE nhẹ và tùy chỉnh cho các ứng dụng Java dựa trên module.
- File JMOD là file archive dạng module, có thể chứa class file, native library, configuration file, header file, tuyên bố pháp lý và các nội dung khác.

## JEP 495: Simplified Source Files and Instance Main Methods (Source File và Instance Main Method đơn giản hóa, preview lần thứ tư)

Tính năng này chủ yếu đơn giản hóa khai báo method `main`. Đối với người mới học Java, khai báo method `main` này đưa vào quá nhiều khái niệm cú pháp Java, không thuận lợi cho việc bắt đầu nhanh.

Định nghĩa method `main` trước khi sử dụng tính năng này:

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

Định nghĩa method `main` sau khi sử dụng tính năng mới:

```java
class HelloWorld {
    void main() {
        System.out.println("Hello, World!");
    }
}
```

Đơn giản hóa thêm (unnamed class cho phép bỏ qua tên class):

```java
void main() {
   System.out.println("Hello, World!");
}
```

## JEP 497: Quantum-Resistant Digital Signature Algorithm (ML-DSA) (thuật toán chữ ký số kháng lượng tử)

JDK 24 bổ sung hỗ trợ triển khai thuật toán chữ ký số dựa trên lattice module có khả năng kháng lượng tử (Module-Lattice-Based Digital Signature Algorithm, **ML-DSA**), nhằm chuẩn bị đối phó với các mối đe dọa có thể do máy tính lượng tử trong tương lai gây ra.

ML-DSA là thuật toán kháng lượng tử được National Institute of Standards and Technology (NIST) của Hoa Kỳ tiêu chuẩn hóa trong FIPS 204, dùng cho chữ ký số và xác thực danh tính.

## JEP 498: Warnings When Using `sun.misc.Unsafe` Memory Access Methods (cảnh báo khi sử dụng method truy cập memory của `sun.misc.Unsafe`)

JDK 23 ([JEP 471](https://openjdk.org/jeps/471)) đề xuất deprecated các method truy cập memory trong `sun.misc.Unsafe`; những method này sẽ bị loại bỏ trong các phiên bản tương lai. Trong JDK 24, runtime sẽ phát cảnh báo khi một method truy cập memory bất kỳ của `sun.misc.Unsafe` được gọi lần đầu.

Các method không an toàn này đã có những phương án thay thế an toàn và hiệu quả:

- `java.lang.invoke.VarHandle`: được giới thiệu trong JDK 9 (JEP 193), cung cấp một phương thức an toàn và hiệu quả để thao tác với heap memory, bao gồm field của object, static field của class và element của array.
- `java.lang.foreign.MemorySegment`: được giới thiệu trong JDK 22 (JEP 454), cung cấp một phương thức an toàn và hiệu quả để truy cập off-heap memory, đôi khi hoạt động cùng `VarHandle`.

`MemorySegment` là một trong các type cốt lõi của Foreign Function & Memory API (API Foreign Function và Memory), dùng để truy cập an toàn heap memory hoặc off-heap memory; `VarHandle` là API tiêu chuẩn độc lập, có thể dùng để truy cập field, element của array hoặc truy cập dữ liệu theo memory layout. Foreign Function & Memory API đã trở thành tính năng chính thức trong JDK 22.

```java
import java.lang.foreign.*;
import java.lang.invoke.VarHandle;

// Class quản lý mảng integer trên off-heap
class OffHeapIntBuffer {

    // VarHandle dùng để truy cập element integer
    private static final VarHandle ELEM_VH = ValueLayout.JAVA_INT.arrayElementVarHandle();

    // Memory manager
    private final Arena arena;

    // Memory segment trên off-heap
    private final MemorySegment buffer;

    // Constructor, cấp phát không gian cho số lượng integer được chỉ định
    public OffHeapIntBuffer(long size) {
        this.arena  = Arena.ofShared();
        this.buffer = arena.allocate(ValueLayout.JAVA_INT, size);
    }

    // Giải phóng memory
    public void deallocate() {
        arena.close();
    }

    // Thiết lập value tại index được chỉ định theo chế độ volatile
    public void setVolatile(long index, int value) {
        ELEM_VH.setVolatile(buffer, 0L, index, value);
    }

    // Khởi tạo element trong range được chỉ định thành 0
    public void initialize(long start, long n) {
        buffer.asSlice(ValueLayout.JAVA_INT.byteSize() * start,
                       ValueLayout.JAVA_INT.byteSize() * n)
              .fill((byte) 0);
    }

    // Copy element trong range được chỉ định vào array mới
    public int[] copyToNewArray(long start, int n) {
        return buffer.asSlice(ValueLayout.JAVA_INT.byteSize() * start,
                              ValueLayout.JAVA_INT.byteSize() * n)
                     .toArray(ValueLayout.JAVA_INT);
    }
}
```

## JEP 499: Structured Concurrency (Structured Concurrency, preview lần thứ tư)

JDK 19 giới thiệu Structured Concurrency dưới dạng incubator API. Trong JDK 24, API này đang ở giai đoạn preview lần thứ tư, với mục đích đơn giản hóa việc lập trình multi-thread, không nhằm thay thế `java.util.concurrent`.

Structured Concurrency coi nhiều task chạy trong các thread khác nhau là một work unit duy nhất, từ đó đơn giản hóa việc xử lý lỗi, tăng reliability và nâng cao observability. Nói cách khác, Structured Concurrency giữ lại readability, maintainability và observability của code single-thread.

API cơ bản của Structured Concurrency là `StructuredTaskScope`. API này hỗ trợ tách task thành nhiều concurrent subtask, thực thi chúng trong các thread riêng và yêu cầu các subtask hoàn thành trước khi task chính tiếp tục.

Cách sử dụng cơ bản của `StructuredTaskScope` như sau:

```java
    try (var scope = new StructuredTaskScope<Object>()) {
        // Dùng method fork để tạo thread thực thi subtask
        Subtask<Integer> subtask1 = scope.fork(task1);
        Subtask<String> subtask2 = scope.fork(task2);
        // Chờ thread hoàn thành
        scope.join();
        // Xử lý result có thể bao gồm xử lý hoặc throw lại exception
        ... process results/exceptions ...
    } // close
```

Structured Concurrency đặc biệt phù hợp với virtual thread, là loại thread nhẹ do JDK triển khai. Nhiều virtual thread dùng chung một thread của hệ điều hành, nhờ đó cho phép tồn tại số lượng virtual thread rất lớn.
