---
title: Giải thích chi tiết quá trình class loading
description: Phân tích các giai đoạn và chi tiết then chốt của quá trình class loading trong JVM, tìm hiểu hành vi cụ thể của verification, preparation, resolution và initialization.
category: Java
tag:
  - JVM
head:
  - - meta
    - name: keywords
      content: class loading,loading,verification,preparation,resolution,initialization,clinit,constant pool
---

## Vòng đời của class

Từ khi được load vào memory của virtual machine đến khi được unload khỏi memory, toàn bộ vòng đời của class có thể được khái quát thành 7 giai đoạn: loading, verification, preparation, resolution, initialization, using và unloading. Trong đó, verification, preparation và resolution có thể gọi chung là linking.

Thứ tự của 7 giai đoạn này như hình dưới đây:

![Vòng đời đầy đủ của một class](https://oss.javaguide.cn/github/javaguide/java/jvm/lifecycle-of-a-class.png)

## Quá trình class loading

**Class file cần được load vào virtual machine thì mới có thể chạy và sử dụng. Vậy virtual machine load các Class file này như thế nào?**

Hệ thống load file thuộc kiểu Class chủ yếu qua ba bước: **loading -> linking -> initialization**. Quá trình linking lại được chia thành ba bước: **verification -> preparation -> resolution**.

![Quá trình class loading](https://oss.javaguide.cn/github/javaguide/java/jvm/class-loading-procedure.png)

Xem chi tiết tại [Java Virtual Machine Specification - 5.3. Creation and Loading](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.3 "Java Virtual Machine Specification - 5.3. Creation and Loading").

### Loading

Đây là bước đầu tiên trong quá trình class loading, chủ yếu hoàn thành 3 việc sau:

1. Lấy binary byte stream định nghĩa class này thông qua fully qualified name.
2. Chuyển static storage structure do byte stream đại diện thành runtime data structure của method area.
3. Tạo trong memory một đối tượng `Class` đại diện cho class này, làm entry point để truy cập dữ liệu trong method area.

Ba điểm trên không được specification của virtual machine quy định cụ thể nên rất linh hoạt. Ví dụ, “lấy binary byte stream định nghĩa class này thông qua fully qualified name” không chỉ rõ lấy từ đâu (`ZIP`, `JAR`, `EAR`, `WAR`, network, được dynamic proxy tạo ra trong runtime, được tạo từ file khác như `JSP`...), cũng không chỉ rõ lấy như thế nào.

Bước loading chủ yếu được thực hiện thông qua **class loader** mà chúng ta sẽ tìm hiểu sau. Có nhiều loại class loader; khi muốn load một class, class loader nào thực hiện việc đó được quyết định bởi **parent delegation model** (tuy nhiên, chúng ta cũng có thể phá vỡ parent delegation model).

> Class loader và parent delegation model cũng là các kiến thức rất quan trọng. Phần này được giới thiệu chi tiết trong bài [Giải thích chi tiết về class loader](https://javaguide.cn/java/jvm/classloader.html "Giải thích chi tiết về class loader"). Khi đọc bài này, bạn chỉ cần biết có các khái niệm đó là được.

Mỗi non-array class hoặc interface đều được tạo bởi một class loader nào đó. Array class không được tạo thông qua `ClassLoader` mà được JVM tự động tạo khi cần; defining class loader của reference type array giống với defining class loader của component type, còn `getClassLoader()` của primitive type array trả về `null`.

Giai đoạn loading của một non-array class (thao tác lấy binary byte stream của class) có tính kiểm soát rất cao. Thông thường có thể kế thừa `ClassLoader` và override `findClass()` để kiểm soát cách lấy byte stream, đồng thời giữ lại quy trình parent delegation do `loadClass()` triển khai; chỉ cần override `loadClass()` khi thực sự muốn thay đổi quy tắc delegation.

Một số thao tác trong loading và linking (chẳng hạn một phần thao tác kiểm tra format của bytecode file) được tiến hành đan xen. Khi loading chưa kết thúc, linking có thể đã bắt đầu.

### Verification

**Verification là bước đầu tiên của linking. Mục đích của giai đoạn này là bảo đảm thông tin trong byte stream của Class file đáp ứng toàn bộ yêu cầu của Java Virtual Machine Specification, để khi được chạy như code, những thông tin này không gây nguy hại cho security của chính virtual machine.**

Giai đoạn verification tiêu tốn tương đối nhiều resource trong toàn bộ quá trình class loading, nhưng rất cần thiết vì có thể ngăn chặn hiệu quả việc thực thi malicious code. Security của chương trình luôn là ưu tiên hàng đầu.

HotSpot từng cung cấp `-Xverify:none` và `-noverify` để tắt phần lớn verification của class, nhưng việc này làm suy yếu kiểm tra security của bytecode và không nên được dùng như một biện pháp tối ưu hóa chung trong production. Hai option này đã bị deprecated trong JDK 13; các JDK hiện đại còn có thể bỏ qua hoặc loại bỏ chúng.

Giai đoạn verification chủ yếu gồm bốn bước kiểm tra:

1. File format verification (kiểm tra format của Class file)
2. Metadata verification (kiểm tra semantic của bytecode)
3. Bytecode verification (kiểm tra semantic của chương trình)
4. Symbolic reference verification (kiểm tra tính đúng đắn của class)

![Sơ đồ giai đoạn verification](https://oss.javaguide.cn/github/javaguide/java/jvm/class-loading-process-verification.png)

Giai đoạn file format verification dựa trên binary byte stream của class, chủ yếu nhằm bảo đảm byte stream đầu vào có thể được parse và lưu trữ chính xác trong method area, đồng thời format phù hợp với yêu cầu mô tả thông tin của một Java type. Ngoài giai đoạn này, ba giai đoạn verification còn lại đều được thực hiện trên storage structure trong method area và không còn đọc hay thao tác trực tiếp trên byte stream nữa.

> Method area là một logical area trong JVM runtime data area, là memory area được các thread dùng chung. Khi virtual machine muốn sử dụng một class, nó cần đọc và parse Class file để lấy thông tin liên quan, sau đó lưu thông tin vào method area. Method area lưu trữ **class information, field information, method information, constant, static variable, code cache do JIT compiler biên dịch và các dữ liệu khác** của những class đã được virtual machine load.
>
> Để tìm hiểu chi tiết về method area, bạn nên đọc bài [Giải thích chi tiết Java memory area](https://javaguide.cn/java/jvm/memory-area.html "Giải thích chi tiết Java memory area").

Symbolic reference verification xảy ra trong giai đoạn resolution của quá trình class loading, cụ thể là khi JVM chuyển symbolic reference thành direct reference (giai đoạn resolution sẽ giới thiệu symbolic reference và direct reference).

Mục đích chính của symbolic reference verification là bảo đảm giai đoạn resolution có thể thực hiện bình thường. Nếu không vượt qua symbolic reference verification, JVM sẽ ném exception, chẳng hạn:

- `java.lang.IllegalAccessError`: exception này được ném khi class cố truy cập hoặc sửa một field mà nó không có quyền truy cập, hoặc gọi một method mà nó không có quyền truy cập.
- `java.lang.NoSuchFieldError`: exception này được ném khi class cố truy cập hoặc sửa một field cụ thể của object, nhưng object đó không còn chứa field này.
- `java.lang.NoSuchMethodError`: exception này được ném khi class cố truy cập một method cụ thể nhưng method đó không tồn tại.
- ...

### Preparation

**Preparation là giai đoạn chính thức cấp phát memory cho class variable và thiết lập initial value cho class variable**, toàn bộ memory này đều được cấp phát trong method area. Cần lưu ý những điểm sau:

1. Memory được cấp phát ở đây chỉ bao gồm class variable (Class Variables, tức static variable, là variable được khai báo với keyword `static`, chỉ liên quan đến class nên được gọi là class variable), không bao gồm instance variable. Instance variable được cấp phát cùng object instance trên Java heap khi object được instantiate.
2. Về mặt khái niệm, memory được class variable sử dụng đều phải được cấp phát trong **method area**. Tuy nhiên cần lưu ý rằng trước JDK 7, khi HotSpot dùng permanent generation để triển khai method area, cách triển khai hoàn toàn phù hợp với khái niệm này. Từ JDK 7 trở đi, HotSpot đã chuyển string constant pool, static variable và các thành phần vốn đặt trong permanent generation sang heap; khi đó class variable được lưu cùng Class object trong Java heap. Bài đọc thêm: [Hiểu sâu về JVM (phiên bản 3), errata #75](https://github.com/fenixsoft/jvm_book/issues/75 "Errata của Hiểu sâu về JVM (phiên bản 3), #75")
3. Initial value được thiết lập ở đây thường là zero value mặc định của data type (chẳng hạn 0, 0L, null, false...). Ví dụ, với `public static int value=111`, `value` thường nhận 0 trong giai đoạn preparation và chỉ được gán 111 trong giai đoạn initialization. Trường hợp đặc biệt là khi field có thuộc tính `ConstantValue`, giai đoạn preparation sẽ gán trực tiếp value do thuộc tính đó chỉ định; compile-time constant như `public static final int value=111` là ví dụ điển hình, nhưng không phải mọi field `static final` đều có thuộc tính `ConstantValue`.

**Zero value của primitive data type** (hình trích từ Hiểu sâu về JVM, phiên bản 3, mục 7.3.3):

![Zero value của primitive data type](https://oss.javaguide.cn/github/javaguide/java/%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E7%9A%84%E9%9B%B6%E5%80%BC.png)

### Resolution

**Resolution là quá trình virtual machine xác định động target cụ thể từ symbolic reference trong runtime constant pool.** Specification hiện tại đề cập đến symbolic reference của class hoặc interface, field, method, interface method, method type, method handle, dynamic call site và dynamic constant.

Giải thích về symbolic reference và direct reference trong mục 7.3.4, phiên bản thứ ba của Hiểu sâu về JVM như sau:

![Symbolic reference và direct reference](https://oss.javaguide.cn/github/javaguide/java/jvm/symbol-reference-and-direct-reference.png)

Ví dụ, khi chương trình gọi method, virtual machine cần xác định method thực tế sẽ được gọi dựa trên symbolic reference của method. Các virtual machine như HotSpot có thể dùng method table, entry address hoặc các internal structure khác để tăng tốc việc gọi, nhưng các cách biểu diễn này thuộc implementation detail, không phải method table offset được Java Virtual Machine Specification quy định thống nhất.

Tóm lại, resolution là quá trình virtual machine xác định target cụ thể như class, field, method hoặc dynamic call site dựa trên symbolic reference trong runtime constant pool; internal representation của direct reference không nhất thiết phải là memory pointer hoặc fixed offset.

### Initialization

**Giai đoạn initialization sẽ thực thi initialization method `<clinit>()` của class hoặc interface (nếu compiler đã tạo method này). Đây là bước cuối cùng của quá trình class loading.**

> Giải thích: compiler tạo `<clinit>()` dựa trên static field initialization expression và static initialization block; nếu không có class initialization code cần thực thi thì Class file có thể không chứa method này.

JVM sẽ synchronize quá trình initialization của class hoặc interface, bảo đảm tại cùng một thời điểm chỉ có một thread thực thi initialization method của nó; điều này không có nghĩa bản thân method `<clinit>()` có Java lock. Các thread khác có thể bị block trong khi chờ initialization của class hoàn tất.

Đối với giai đoạn initialization, các trường hợp active use chính được specification liệt kê bao gồm:

1. Khi gặp 4 bytecode instruction `new`, `getstatic`, `putstatic` hoặc `invokestatic`:
   - `new`: tạo một class instance object.
   - `getstatic`, `putstatic`: đọc hoặc thiết lập static field của một type (ngoại trừ static field được khai báo với `final` và đã được đưa kết quả vào constant pool trong compile time).
   - `invokestatic`: gọi static method của class.
2. Khi dùng method trong package `java.lang.reflect` để reflection call class, chẳng hạn `Class.forName("...")`, `newInstance()`... Nếu class chưa được initialize thì phải trigger initialization của nó.
3. Khi initialize một class, nếu superclass của nó chưa được initialize thì trước tiên trigger initialization của superclass.
4. Khi virtual machine khởi động, user cần định nghĩa một main class để thực thi (class chứa method `main`), virtual machine sẽ initialize class này trước.
5. Khi lần đầu gọi `MethodHandle` có resolution result là `REF_getStatic`, `REF_putStatic`, `REF_invokeStatic` hoặc `REF_newInvokeSpecial`, cần initialize class hoặc interface khai báo target đó.
6. **Bổ sung, từ [issue745](https://github.com/Snailclimb/JavaGuide/issues/745 "issue745")** Khi một interface định nghĩa default method được bổ sung trong JDK 8 (interface method được khai báo với keyword `default`), nếu một implementation class của interface đó được initialize thì interface đó phải được initialize trước implementation class.

## Class unloading

> Nội dung về unloading được lấy từ [issue#662](https://github.com/Snailclimb/JavaGuide/issues/662 "issue#662") và được **[guang19](https://github.com/guang19 "guang19")** bổ sung, hoàn thiện.

Class unloading là quá trình JVM thu hồi method area representation của một class hoặc interface cùng các resource liên quan. Theo Java Language Specification, class hoặc interface chỉ có thể được unload khi defining class loader của nó có thể bị GC; class hoặc interface do bootstrap class loader định nghĩa không thể unload. Trong các ứng dụng HotSpot phổ biến, việc này thường xảy ra với custom class loader có thể được GC và các class do nó định nghĩa.

Trong HotSpot, để xác định một class có thể được unload hay không, thường có thể dựa vào ba điều kiện sau:

1. Tất cả instance object của class đó đã được GC, nghĩa là trên heap không còn instance object của class đó.
2. Class đó không còn được reference ở bất kỳ nơi nào khác.
3. Instance của class loader của class đó đã được GC.

Việc đáp ứng các điều kiện này chỉ có nghĩa class đó đủ điều kiện để được unload, không bảo đảm JVM sẽ unload ngay lập tức hoặc chắc chắn unload nó. Class do bootstrap class loader tạo sẽ không được unload; class do custom class loader tạo thì có thể được unload.

Thông thường, bootstrap class loader, platform class loader và application class loader có sẵn trong JDK sẽ tồn tại lâu dài cùng JVM; instance của custom class loader thì có thể trở nên unreachable, vì vậy class do nó định nghĩa có thể đủ điều kiện unloading. Từ JDK 9, extension class loader trước đây đã được thay thế bằng platform class loader.

**Tài liệu tham khảo**

- Hiểu sâu về JVM
- Thực chiến JVM
- Chapter 5. Loading, Linking, and Initializing - Java Virtual Machine Specification: <https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.4>

<!-- @include: @article-footer.snippet.md -->
