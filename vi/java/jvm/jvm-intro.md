---
title: Giải thích JVM bằng ngôn ngữ đời thường
description: Giới thiệu JVM, các thành phần cơ bản và quy trình class loading, thực thi theo cách dễ hiểu để nhanh chóng làm quen với nguyên lý của virtual machine.
category: Java
tag:
  - JVM
head:
  - - meta
    - name: keywords
      content: JVM Basics, class loading, method area, heap và stack, program counter, runtime data area
---

> Bài viết do [Nói lên điều ước của bạn](https://juejin.im/user/5c2400afe51d45451758aa96) đóng góp, địa chỉ bài gốc: <https://juejin.im/post/5e1505d0f265da5d5d744050>.

## Lời mở đầu

Nếu có vấn đề về cách dùng từ hoặc cách hiểu trong bài viết, hãy góp ý. Bài viết này chỉ nhằm đề cập chứ không đi sâu, nhưng sẽ cố gắng trình bày các điểm kiến thức một cách hiệu quả.

## I. Giới thiệu cơ bản về JVM

JVM là viết tắt của Java Virtual Machine, một máy tính tưởng tượng, đồng thời là một specification. Nó mô phỏng các chức năng máy tính khác nhau trên máy tính thực tế để thực hiện···

Được rồi, tạm gác câu nói quá chuyên môn này sang một bên: chỉ cần biết JVM thực chất giống một máy tính nhỏ chạy trên môi trường hệ điều hành như Windows hoặc Linux. Nó tương tác trực tiếp với hệ điều hành, không tương tác trực tiếp với phần cứng; còn hệ điều hành có thể giúp chúng ta thực hiện công việc tương tác với phần cứng.

![](https://static001.geekbang.org/infoq/da/da0380a04d9c04facd2add5f6dba06fa.png)

### 1.1 File Java được chạy như thế nào

Ví dụ, chúng ta viết một `HelloWorld.java`. Nếu tạm gác mọi thứ khác sang một bên, `HelloWorld.java` chẳng phải giống một file văn bản hay sao? Chỉ là file văn bản này được viết bằng tiếng Anh và có một mức thụt lề nhất định.

**JVM** không nhận biết file văn bản, nên cần **compile** file đó để tạo thành **HelloWorld.class**, một file binary mà JVM có thể đọc.

#### ① Class loader

Nếu **JVM** muốn thực thi file **`.class`**, chúng ta cần nạp file đó vào một **class loader**. Class loader giống như người vận chuyển, đưa toàn bộ file **`.class`** vào JVM.

![](https://static001.geekbang.org/infoq/2f/2f012fde94376f43a25dbe1dd07e0dd8.png)

#### ② Method area

**Method area** dùng để lưu các dữ liệu như metadata, chẳng hạn thông tin class, constant, static variable, compiled code···

Class loader sẽ đưa file `.class` vào khu vực này trước.

#### ③ Heap

**Heap** chủ yếu lưu object instance, array và các dữ liệu khác. Heap và method area đều thuộc **khu vực shared giữa các thread**. Thread shared chỉ có nghĩa là nhiều thread có thể truy cập các khu vực này, không có nghĩa bản thân khu vực đó “không thread-safe”; có xảy ra data race hay không phụ thuộc vào cách chương trình truy cập dữ liệu bên trong.

#### ④ Stack

**Stack** là không gian chạy code. Mỗi method chúng ta viết đều được đưa vào **stack** để chạy.

Chúng ta thường nghe đến hai thuật ngữ native method stack và Java Native Interface (JNI). Native method được triển khai bằng code native bên ngoài Java, ngôn ngữ triển khai phổ biến là C hoặc C++; cách triển khai cụ thể phụ thuộc vào virtual machine và native library.

#### ⑤ Program counter

Program counter ghi lại địa chỉ của JVM instruction mà thread hiện tại đang thực thi; sau khi thực thi branch, loop, xử lý exception hoặc chuyển thread, virtual machine dựa vào nó để tiếp tục thực thi. Giống stack, nó thuộc **khu vực private của thread**. Khi thực thi native method, specification không quy định giá trị của nó.

![](https://static001.geekbang.org/infoq/c6/c602f57ea9297f50bbc265f1821d6263.png)

#### Tóm tắt

1. Sau khi compile, file Java trở thành file bytecode `.class`.
2. File bytecode được class loader đưa vào JVM.
3. Trong runtime data area của virtual machine, method area và heap là khu vực thread shared; virtual machine stack, native method stack và program counter là khu vực thread private. Việc shared hay không chỉ là phạm vi visibility của memory area, không đồng nghĩa trực tiếp với thread-safe hay không thread-safe.

### 1.2 Ví dụ code đơn giản

Một class Student đơn giản

![](https://static001.geekbang.org/infoq/12/12f0b239db65b8a95f0ce90e9a580e4d.png)

Một method `main`

![](https://static001.geekbang.org/infoq/0c/0c6d94ab88a9f2b923f5fea3f95bc2eb.png)

Các bước thực thi method `main` như sau:

1. Sau khi compile `App.java` và nhận được `App.class`, thực thi `App.class`. Hệ thống sẽ khởi động một JVM process, tìm binary file tên `App.class` trong classpath, rồi load thông tin class `App` vào method area của runtime data area. Quá trình này gọi là class loading của class `App`.
2. JVM tìm entry point của chương trình `App` và thực thi method `main`.
3. Câu lệnh đầu tiên trong `main` là `Student student = new Student("tellUrDream")`, yêu cầu JVM tạo một object `Student`. Tuy nhiên lúc này method area chưa có thông tin class `Student`, nên JVM lập tức load class `Student` và đưa thông tin class `Student` vào method area.
4. Sau khi load class `Student`, JVM cấp phát memory cho một instance `Student` mới trên heap, sau đó gọi constructor để khởi tạo instance `Student`. Instance `Student` này giữ một reference **trỏ đến type information của class Student trong method area**.
5. Khi thực thi `student.sayName();`, JVM dựa vào reference của `student` để tìm object `student`, sau đó dựa vào reference mà object `student` giữ để định vị method table của type information class `Student` trong method area và lấy địa chỉ bytecode của `sayName()`.
6. Thực thi `sayName()`.

Thực ra không cần quan tâm quá nhiều; chỉ cần biết khi khởi tạo object instance, JVM sẽ tìm thông tin class trong method area, sau đó chạy method ở stack. Việc tìm method được thực hiện trong method table.

## II. Giới thiệu class loader

Như đã đề cập, class loader chịu trách nhiệm load file `.class`. Các file này có chữ ký file cụ thể ở phần đầu; class loader load nội dung bytecode của file class vào memory và chuyển nội dung đó thành runtime data structure trong method area. ClassLoader chỉ chịu trách nhiệm load file class, còn file có thể chạy được hay không do Execution Engine quyết định.

### 2.1 Quy trình của class loader

Từ khi class được load vào memory của virtual machine đến khi giải phóng memory có tổng cộng 7 bước: loading, verification, preparation, resolution, initialization, using và unloading. Trong đó, **verification, preparation và resolution được gọi chung là linking**.

#### 2.1.1 Loading

1. Load file class vào memory.
2. Chuyển static data structure thành runtime data structure trong method area.
3. Tạo một object `java.lang.Class` đại diện cho class trên heap, làm entry point để truy cập dữ liệu.

#### 2.1.2 Linking

1. Verification: bảo đảm class được load phù hợp với specification của JVM và an toàn; bảo đảm method của class được kiểm tra không thực hiện hành vi gây hại cho virtual machine khi chạy. Đây thực chất là một bước security check.
2. Preparation: cấp phát storage cho class variable (field `static`) và thiết lập initial value mặc định. Ví dụ, `static int a = 3` thường nhận giá trị mặc định `0` trong giai đoạn preparation, sau đó được gán `3` trong giai đoạn initialization. Vị trí lưu trữ cụ thể là chi tiết triển khai của virtual machine.
3. Resolution: virtual machine chuyển symbol reference trong runtime constant pool thành direct reference. `import` trong source code chỉ là cơ chế name resolution lúc compile, không phải symbol reference trong constant pool; biểu diễn cụ thể của direct reference cũng không nhất thiết là object address.

#### 2.1.3 Initialization

Initialization thực chất là quá trình thực thi class initialization method `<clinit>()`, đồng thời phải bảo đảm method `<clinit>()` của superclass đã thực thi xong trước đó. Method này do compiler tổng hợp, tuần tự thực thi explicit initialization của toàn bộ class variable (member variable có modifier `static`) và các câu lệnh trong static code block. Lúc này `static int a` ở giai đoạn preparation chuyển từ giá trị mặc định `0` thành giá trị explicit initialization `3`. Do thứ tự thực thi, nếu class variable được thay đổi lần nữa trong static code block ở giai đoạn initialization thì giá trị explicit initialization sẽ bị ghi đè; giá trị cuối cùng sẽ là giá trị được gán trong static code block.

> Lưu ý: trong bytecode file có hai loại initialization method: `<init>` cho non-static resource và `<clinit>` cho static resource. Class initialization method `<clinit>()` khác với constructor của class; các method này là những special method trong bytecode file chỉ JVM có thể nhận biết.

#### 2.1.4 Unloading

Class unloading là việc virtual machine thu hồi class metadata không còn reachable và các resource liên quan. Thông thường, class loader định nghĩa class đó cũng phải không còn reachable. Việc này khác với garbage collection của object thông thường; có unloading hay không và unloading khi nào do virtual machine implementation và garbage collector quyết định.

### 2.2 Thứ tự load của class loader

Lấy HotSpot của JDK 8 làm ví dụ, hierarchy của class loader thường gặp như sau. Sau khi module hóa ở JDK 9, Extension ClassLoader được thay thế bởi Platform ClassLoader và `rt.jar` không còn tồn tại:

1. BootStrap ClassLoader: `rt.jar`
2. Extension ClassLoader: load các file JAR mở rộng.
3. App ClassLoader: các file JAR trong classpath được chỉ định.
4. Custom ClassLoader: class loader tùy chỉnh.

### 2.3 Cơ chế parent delegation

Khi một class nhận được yêu cầu load, class đó không tự thử load trước mà ủy thác cho parent class loader thực hiện. Ví dụ, khi muốn `new` một `Person`, đây là class do chúng ta tự định nghĩa; để load class đó, trước hết yêu cầu sẽ được ủy thác cho App ClassLoader. Chỉ khi tất cả parent class loader đều phản hồi rằng không thể hoàn thành yêu cầu này (tức parent class loader không tìm thấy Class cần load), child class loader mới tự thử load.

Ưu điểm của cách này là khi load class nằm trong package `rt.jar`, bất kể loader nào thực hiện load, cuối cùng yêu cầu cũng được ủy thác cho BootStrap ClassLoader. Nhờ đó, sử dụng các class loader khác nhau vẫn nhận được cùng một kết quả.

Đây cũng có tác dụng cô lập, tránh để code của chúng ta ảnh hưởng đến code của JDK. Ví dụ, nếu tự định nghĩa một `java.lang.String`:

```java
package java.lang;
public class String {
    public static void main(String[] args) {
        System.out.println();
    }
}
```

Khi thử chạy hàm `main` của class hiện tại, code chắc chắn sẽ báo lỗi. Nguyên nhân là lúc load, JVM đã tìm thấy `java.lang.String` trong `rt.jar`, nhưng phát hiện class này không có method `main`.

## III. Runtime data area

### 3.1 Native method stack và program counter

Ví dụ, khi mở source code của class `Thread`, chúng ta sẽ thấy method `start0` có modifier keyword `native` và không có Java method body. Loại method này được triển khai bằng code native bên ngoài Java, ngôn ngữ triển khai phổ biến là C hoặc C++; virtual machine sử dụng native method stack để hỗ trợ thực thi native method.

Program counter ghi lại địa chỉ JVM instruction mà thread hiện tại đang thực thi. Đây cũng là runtime data area duy nhất mà Java Virtual Machine Specification không quy định bất kỳ trường hợp `OutOfMemoryError` nào. Bytecode interpreter chọn bytecode instruction tiếp theo cần thực thi bằng cách thay đổi giá trị program counter.

Nếu đang thực thi native method, specification không quy định giá trị của program counter.

### 3.2 Method area

Vai trò chính của method area là lưu class metadata, constant, static variable··· Khi lượng thông tin lưu trữ quá lớn và không thể đáp ứng việc cấp phát memory, lỗi sẽ được báo.

### 3.3 Virtual machine stack và virtual machine heap

Nói một câu: stack phụ trách việc chạy, heap phụ trách việc lưu trữ. Virtual machine stack chịu trách nhiệm chạy code, còn virtual machine heap chịu trách nhiệm lưu trữ dữ liệu.

#### 3.3.1 Khái niệm virtual machine stack

Đây là memory model để thực thi Java method. Mỗi lần gọi method sẽ tạo một stack frame; stack frame chứa local variable table, operand stack, dynamic linking, method return address và các thông tin khác, đồng thời được thread sở hữu riêng. Local variable table là một phần của stack frame.

```java
public class Person{
    int a = 1;

    public void doSomething(){
        int b = 2;
    }
}
```

#### 3.3.2 Exception có thể xảy ra trong virtual machine stack

Nếu độ sâu stack mà thread yêu cầu lớn hơn độ sâu tối đa của virtual machine stack, **StackOverflowError** sẽ được báo (lỗi này thường xuất hiện trong recursion). Java virtual machine cũng có thể mở rộng động, nhưng trong quá trình mở rộng sẽ liên tục request memory; khi không thể request đủ memory, **OutOfMemoryError** sẽ được báo.

#### 3.3.3 Vòng đời của virtual machine stack

Đối với stack, không có garbage collection. Khi chương trình kết thúc, không gian của stack sẽ tự nhiên được giải phóng. Vòng đời của stack nhất quán với thread mà nó thuộc về.

Bổ sung: method parameter, primitive type variable được định nghĩa bên trong method và object reference thường được lưu trong local variable table của stack frame hiện tại; object instance thường được cấp phát trên heap. Bytecode của method thuộc class metadata, không được cấp phát trên stack dưới dạng “instance method”.

#### 3.3.4 Việc thực thi virtual machine stack

Stack frame không phải bản thân method, mà là runtime data structure tương ứng với một lần gọi method, dùng để lưu local variable table, operand stack, dynamic linking và các thông tin khác. Bytecode và metadata của method không được lưu trong virtual machine stack.

Dữ liệu trong stack đều tồn tại dưới dạng stack frame. Đây là một data set về method và runtime data. Ví dụ, khi thực thi method `a`, một stack frame `A1` tương ứng sẽ được tạo rồi push vào stack. Tương tự, method `b` có `B1`, method `c` có `C1`. Khi thread thực thi xong, stack sẽ pop `C1` trước, sau đó là `B1`, `A1`. Đây là nguyên tắc first in, last out, hay last in, first out.

#### 3.3.5 Tái sử dụng local variable

Local variable table dùng để lưu method parameter và local variable được định nghĩa bên trong method. Dung lượng của nó lấy Slot làm đơn vị nhỏ nhất; một slot có thể lưu data type không quá 32 bit.

Virtual machine định vị Slot trong local variable table bằng index. Nếu capacity của local variable table là `n`, phạm vi index hợp lệ là `[0, n)`; `long` và `double` chiếm hai Slot liên tiếp. Method parameter được sắp xếp trong local variable table theo thứ tự do specification quy định. Để tiết kiệm không gian stack frame, sau khi vị trí thực thi vượt ra ngoài scope của một local variable, Slot của biến đó có thể được variable khác tái sử dụng; object reference vẫn được xem là valid trong local variable table sẽ tham gia quá trình scan GC Roots.

#### 3.3.6 Khái niệm virtual machine heap

Lấy generational garbage collector của HotSpot làm ví dụ, heap thường được chia thành **young generation** và **old generation**; young generation lại có thể chia thành **Eden** và hai khu vực **Survivor**. Trong một lần copy, một khu vực Survivor đóng vai trò from, khu vực còn lại đóng vai trò to. Tỷ lệ mặc định giữa Eden và một Survivor thường là **8:1:1**, nhưng kích thước thực tế có thể chịu ảnh hưởng của collector được sử dụng và adaptive strategy. Permanent generation là một implementation của method area trong HotSpot ở JDK 7 trở về trước, không tương đương với toàn bộ “non-heap memory”.

Heap memory chủ yếu lưu object, garbage collector sẽ xác định và thu hồi object không reachable trong đó. Non-heap memory theo cách gọi của HotSpot không chỉ bao gồm implementation của method area mà còn có các khu vực như Code Cache. Sau khi JDK 8 loại bỏ permanent generation, class metadata được lưu trong Metaspace do JVM quản lý; Metaspace dùng native memory thay vì Java heap. Các parameter liên quan:

```plain
MetaspaceSize: ngưỡng high-water ban đầu để trigger metadata GC, sau đó JVM sẽ điều chỉnh động
MaxMetaspaceSize: giới hạn trên của kích thước Metaspace
```

Sau khi loại bỏ permanent generation, sẽ không còn `java.lang.OutOfMemoryError: PermGen space` do permanent generation cạn kiệt. Tuy nhiên, Metaspace vẫn có thể throw `OutOfMemoryError: Metaspace` khi có quá nhiều class metadata hoặc đạt các giới hạn như `MaxMetaspaceSize`.

#### 3.3.7 Giới thiệu young generation Eden

Sau khi `new` một object, object sẽ trước tiên được đặt vào vùng memory được tách ra từ Eden làm storage. Tuy nhiên, heap memory là thread shared, nên có thể xuất hiện tình huống hai object dùng chung một vùng memory. Cách JVM xử lý là cấp trước cho mỗi thread một vùng memory liên tục và quy định vị trí lưu object; nếu không đủ space thì cấp phát thêm các vùng memory. Thao tác này gọi là TLAB, bạn có thể tìm hiểu thêm nếu quan tâm.

Khi không gian Eden không đủ để tiếp tục cấp phát object, Minor GC thường được trigger (tức GC xảy ra trong young generation); object còn sống có thể được copy sang Survivor hoặc promote thẳng lên old generation. Sau khi copy xong, hai Survivor from và to đổi vai trò cho nhau. Khi object đạt promotion threshold, object có thể đi vào old generation; trong các generational collector phổ biến của HotSpot, default của `-XX:MaxTenuringThreshold` thường là 15, nhưng promotion age thực tế còn chịu ảnh hưởng của dynamic age judgment, capacity của Survivor và strategy của collector, không phải mọi object đều cố định trải qua 15 lần Minor GC.

> 🐛 Đính chính: khi memory của Eden đầy thì Minor GC sẽ được trigger; Survivor0 đầy không trigger Minor GC.
>
> **Vậy object trong Survivor0 được garbage collection khi nào?**
>
> Giả sử Survivor0 đang đầy, lúc này lại trigger Minor GC và phát hiện Survivor0 vẫn đầy, không thể chứa thêm object. Khi đó, hệ thống sẽ thực hiện reachability analysis với object ở S0 và Eden, tìm các object còn hoạt động, copy chúng sang S1, đồng thời clear object ở khu vực S0 và Eden. Như vậy object không reachable được xóa, sau đó khu vực S0 và S1 đổi chỗ cho nhau.

Old generation chủ yếu lưu object sống lâu hoặc được promote trực tiếp. Khi old generation không đủ space, old generation collection hoặc Full GC có thể được trigger; hành vi cụ thể phụ thuộc vào garbage collector. Phạm vi pause của các collector khác nhau cũng không giống nhau, không thể khái quát rằng toàn bộ application thread luôn bị dừng.

Ngoài ra, nếu sau khi old generation thực hiện full GC mà vẫn không thể lưu object, OOM sẽ xảy ra. Khi đó heap memory trong virtual machine không đủ; nguyên nhân có thể là kích thước heap được thiết lập quá nhỏ, có thể điều chỉnh bằng parameter `-Xms`, `-Xmx`. Cũng có thể code tạo ra object vừa lớn vừa nhiều, đồng thời các object đó liên tục được reference khiến garbage collector không thể thu hồi chúng trong thời gian dài.

![](https://static001.geekbang.org/infoq/39/398255141fde8ba208f6c99f4edaa9fe.png)

Bổ sung về parameter `-XX:TargetSurvivorRatio`: không nhất thiết phải đạt `-XX:MaxTenuringThreshold` thì object mới được chuyển lên old generation. Ví dụ, object age 5 chiếm 30%, age 6 chiếm 36%, age 7 chiếm 34%. Khi thêm một age group (như age 6 trong ví dụ) khiến tổng dung lượng vượt quá Survivor space \* `TargetSurvivorRatio`, các object từ age group đó trở lên sẽ vào old generation (tức object age 6 và age 7 trong ví dụ sẽ được promote lên old generation). Khi đó không cần đợi đến 15 lần như `MaxTenuringThreshold` yêu cầu.

#### 3.3.8 Cách xác định object cần bị thu hồi

![](https://static001.geekbang.org/infoq/1b/1ba7f3cff6e07c6e9c6765cc4ef74997.png)

Trong hình, program counter, virtual machine stack và native method stack tồn tại cùng vòng đời của thread. Việc cấp phát và thu hồi memory đều xác định được. Khi thread kết thúc, memory tự nhiên được thu hồi, nên không cần quan tâm đến garbage collection. Java heap và method area thì khác: chúng là vùng shared giữa các thread, việc cấp phát và thu hồi memory đều là động. Vì vậy garbage collector chủ yếu quan tâm đến memory thuộc heap và method area.

Trước khi thu hồi, cần xác định object nào còn sống và object nào đã chết. Dưới đây là hai phương pháp tính toán cơ bản.

1. Tính bằng reference counter: thêm một reference counter cho object, mỗi lần reference đến object thì counter tăng một, khi reference mất hiệu lực thì giảm một; khi counter bằng `0`, object sẽ không được sử dụng nữa. Tuy nhiên, phương pháp này không thể thu hồi khi xuất hiện circular reference giữa các object.

2. Tính bằng reachability analysis: bắt đầu từ một tập GC Roots, tìm kiếm theo quan hệ reference giữa các object; path đi qua khi tìm kiếm được gọi là reference chain. Quan hệ reference giữa các object tạo thành graph, không phải binary tree. Khi không tồn tại reference chain nào giữa một object và GC Roots, object đó là unreachable. Các programming language thương mại phổ biến như Java, C# đều dùng cách này để xác định object còn sống hay không.

(Chỉ cần biết sơ qua) Trong Java, các object có thể làm GC Roots được chia thành các loại sau:

1. Object được reference trong virtual machine stack (local variable table trong stack frame) (local variable).
2. Object được reference bởi static variable trong method area (static variable).
3. Object được reference bởi constant trong method area.
4. Object được reference bởi JNI trong native method stack (tức method có modifier `native`) (JNI là cách Java virtual machine gọi function C tương ứng; thông qua JNI function cũng có thể tạo Java object mới. Ngoài ra, local reference hoặc global reference của object trong JNI đều khiến object mà chúng trỏ đến được đánh dấu là không thể thu hồi).
5. Java thread đã start nhưng chưa terminate.

Ưu điểm của phương pháp này là giải quyết được vấn đề circular reference. Collector cần có được quan hệ reference nhất quán giữa các object ở một số giai đoạn, thường gây ra pause Stop-The-World; concurrent collector hiện đại có thể thực hiện phần lớn công việc marking đồng thời với application thread, không phải toàn bộ quá trình reachability analysis đều cần “dừng tất cả process”.

#### 3.3.9 Cách tuyên bố object đã thực sự chết

Trước hết phải đề cập đến một method tên là **`finalize()`**.

`finalize()` là một method của class `Object`. Method `finalize()` của một object nhiều nhất được hệ thống tự động gọi một lần; nếu object khôi phục khả năng reachable thông qua method này, lần tiếp theo bị xác định là unreachable sẽ không được gọi lại.

Bổ sung: không khuyến khích gọi `finalize()` trong chương trình để tự cứu object. Thời điểm thực thi không xác định, thậm chí không bảo đảm chắc chắn sẽ thực thi; chi phí chạy cao và không thể bảo đảm thứ tự gọi của các object. `finalize()` bị deprecated trong Java 9 và được đánh dấu chờ loại bỏ trong Java 18. Khi cần quản lý resource ngoài heap, có thể tùy trường hợp dùng `try-with-resources` hoặc `java.lang.ref.Cleaner`; bản thân `Cleaner` không phải tên gọi chung của strong, soft, weak và phantom reference.

![](https://static001.geekbang.org/infoq/8d/8d7f0381c7d857c7ceb8ae5a5fef0f4a.png)

Đối với object đã override method `finalize()` nhưng method này chưa từng được thực thi, quy trình xử lý thường được khái quát thành hai lần marking:

1. Nếu reachability analysis không phát hiện reference chain nối object với GC Roots, object sẽ được mark lần đầu và đưa vào bước sàng lọc. Với object cần thực thi method `finalize()`, HotSpot có thể đưa finalization reference tương ứng vào pending queue để Finalizer thread xử lý bất đồng bộ.
2. Nếu trong method `finalize()` object thiết lập lại liên kết với object trên reference chain, các lần garbage collection sau sẽ đưa object ra khỏi tập “sắp được thu hồi”; nếu không, object vẫn có thể bị thu hồi. Quy trình này không có nghĩa garbage collector sẽ đồng bộ chờ method `finalize()` thực thi, cũng không bảo đảm method đó chắc chắn được gọi.

Nếu đã xác định object thực sự chết, chúng ta thu hồi rác như thế nào?

### 3.4 Garbage collection algorithm

Để tìm hiểu chi tiết về các garbage collection algorithm phổ biến, nên đọc bài viết này: [Giải thích chi tiết về garbage collection của JVM (trọng tâm)](https://javaguide.cn/java/jvm/jvm-garbage-collection.html).

### 3.5 (Chỉ cần biết) Các garbage collector khác nhau

Garbage collector trong HotSpot VM và các trường hợp sử dụng phù hợp

![](https://static001.geekbang.org/infoq/9f/9ff72176ab0bf58bc43e142f69427379.png)

Trong cấu hình Server HotSpot phổ biến của JDK 8, tổ hợp mặc định thường là Parallel Scavenge và Parallel Old; giá trị mặc định thực tế vẫn có thể chịu ảnh hưởng của virtual machine implementation, mode chạy và platform.

JDK 9 đặt G1 làm garbage collector mặc định của Server HotSpot. Các collector khác nhau có trade-off riêng giữa throughput, latency, memory usage và CPU overhead; không thể tách khỏi application load và JVM configuration để khẳng định collector nào luôn có pause ngắn nhất.

### 3.6 (Chỉ cần biết) Các parameter thường dùng của JVM

JVM có rất nhiều parameter. Ở đây chỉ liệt kê một số parameter quan trọng; bạn cũng có thể tra cứu thêm bằng các search engine.

| Tên parameter                | Ý nghĩa                                                | Mô tả                                                                                                                         |
| ---------------------------- | ------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------- |
| `-Xms`                       | Kích thước heap ban đầu                                | Default do adaptive strategy của HotSpot tính theo môi trường chạy, không nên viết thành tỷ lệ cố định so với physical memory |
| `-Xmx`                       | Kích thước heap tối đa                                 | Default do adaptive strategy của HotSpot tính theo môi trường chạy                                                            |
| `-Xmn`                       | Kích thước young generation                            | Chỉ áp dụng cho collector có khái niệm young generation cố định; đặt rõ sẽ hạn chế adaptive adjustment                        |
| `-XX:NewSize`                | Kích thước ban đầu của young generation                | Chỉ có hiệu lực với generational collector hỗ trợ parameter này                                                               |
| `-XX:MaxNewSize`             | Giá trị tối đa của young generation                    | Chỉ có hiệu lực với generational collector hỗ trợ parameter này                                                               |
| `-XX:PermSize`               | Kích thước ban đầu của permanent generation            | Chỉ áp dụng cho HotSpot của JDK 7 trở về trước; JDK 8 đã loại bỏ permanent generation                                         |
| `-XX:MaxPermSize`            | Giá trị tối đa của permanent generation                | Chỉ áp dụng cho HotSpot của JDK 7 trở về trước; JDK 8 đã loại bỏ permanent generation                                         |
| `-Xss`                       | Kích thước stack của mỗi thread                        | Default phụ thuộc platform và JVM version, nên thiết lập dựa trên số thread, độ sâu call và kết quả test                      |
| `-XX:NewRatio`               | Tỷ lệ capacity giữa old generation và young generation | Ví dụ giá trị 4 nghĩa là old generation : young generation = 4:1; có hiệu lực hay không phụ thuộc collector                   |
| `-XX:SurvivorRatio`          | Tỷ lệ capacity giữa Eden và một Survivor               | Ví dụ giá trị 8 thường cho Eden:From:To = 8:1:1; adaptive strategy có thể điều chỉnh kích thước thực tế                       |
| `-XX:+DisableExplicitGC`     | Bỏ qua explicit GC request do `System.gc()` phát ra    | Không đồng nghĩa tắt automatic garbage collection của JVM; cần đánh giá các trường hợp như direct memory trước khi dùng       |
| `-XX:PretenureSizeThreshold` | Threshold để object lớn đi thẳng vào old generation    | Có hỗ trợ và có hiệu lực thế nào phụ thuộc garbage collector                                                                  |
| `-XX:ParallelGCThreads`      | Số GC thread trong parallel phase của Stop-The-World   | Default do JVM tính dựa trên số processor khả dụng và các yếu tố khác                                                         |
| `-XX:MaxGCPauseMillis`       | Mục tiêu thời gian pause GC tối đa                     | Đây là soft target, không phải hard guarantee; hiệu quả cụ thể phụ thuộc garbage collector được sử dụng                       |

Ngoài ra còn có một số parameter về logging và CMS, nhưng ở đây không liệt kê từng parameter.

## IV. Một số khía cạnh về JVM tuning

Dựa trên các kiến thức JVM vừa đề cập, chúng ta có thể thử tuning JVM, chủ yếu là phần heap memory.

Đối với collector dùng layout generational cố định, Java heap có thể được xem gần đúng là tổng của young generation và old generation; permanent generation hoặc Metaspace không thuộc Java heap. Khi tổng heap size cố định, tăng young generation sẽ làm giảm không gian old generation, nhưng không có “giá trị tối ưu chính thức” áp dụng cho mọi application; cần test dựa trên object allocation, thời gian sống và collector được sử dụng.

### 4.1 Điều chỉnh max heap memory và min heap memory

`-Xmx` và `-Xms` lần lượt chỉ định giá trị tối đa và giá trị ban đầu của Java heap. Nếu không thiết lập rõ, default được HotSpot tính adaptive dựa trên JVM version, memory khả dụng, container limit và môi trường chạy; không nên ước tính theo một tỷ lệ cố định của physical memory.

HotSpot có thể điều chỉnh committed heap space trong khoảng `-Xms` đến `-Xmx`. `MinHeapFreeRatio` và `MaxHeapFreeRatio` là các parameter được một số collector dùng để điều khiển tỷ lệ free sau GC. Hành vi scale up/down và tỷ lệ mặc định phụ thuộc collector, JDK version và adaptive strategy, không thể khái quát thống nhất thành 40% và 70% cố định.

Đặt `-Xms` và `-Xmx` cùng một giá trị có thể tránh việc điều chỉnh committed heap size trong lúc chạy, nhưng sẽ reserve heap capacity lớn ngay từ khi startup; có dùng cách này hay không vẫn nên đánh giá dựa trên môi trường deploy và load.

Chúng ta thực thi code sau:

```java
System.out.println("Xmx=" + Runtime.getRuntime().maxMemory() / 1024.0 / 1024 + "M");    // Dung lượng tối đa của hệ thống
System.out.println("free mem=" + Runtime.getRuntime().freeMemory() / 1024.0 / 1024 + "M");  // Dung lượng trống của hệ thống
System.out.println("total mem=" + Runtime.getRuntime().totalMemory() / 1024.0 / 1024 + "M");  // Tổng dung lượng hiện có
```

Lưu ý: kích thước được thiết lập ở đây là Java heap size, tức young generation size + old generation size.

![](https://static001.geekbang.org/infoq/11/114f32ddd295b2e30444f42f6180538c.png)

Thiết lập một parameter trong VM options:

```plain
-Xmx20m -Xms5m -XX:+PrintGCDetails
```

![](https://static001.geekbang.org/infoq/7e/7ea0bf0dec20e44bf95128c571d6ef0e.png)

Khởi động lại method `main`.

![](https://static001.geekbang.org/infoq/c8/c89edbd0a147a791cfabdc37923c6836.png)

Ở đây GC báo Allocation Failure (cấp phát thất bại); sự việc xảy ra trong PSYoungGen, nghĩa là trong young generation.

Lúc này memory được cấp phát là 18M, free memory là 4.214195251464844M.

Bây giờ chúng ta tạo một byte array để xem, thực thi code sau:

```java
byte[] b = new byte[1 * 1024 * 1024];
System.out.println("Allocated 1M to the array");
System.out.println("Xmx=" + Runtime.getRuntime().maxMemory() / 1024.0 / 1024 + "M");  // Dung lượng tối đa của hệ thống
System.out.println("free mem=" + Runtime.getRuntime().freeMemory() / 1024.0 / 1024 + "M");  // Dung lượng trống của hệ thống
System.out.println("total mem=" + Runtime.getRuntime().totalMemory() / 1024.0 / 1024 + "M");
```

![](https://static001.geekbang.org/infoq/db/dbeb6aea0a90949f7d7fe4746ddb11a3.png)

Lúc này free memory lại giảm, nhưng total memory không thay đổi. Java sẽ cố gắng duy trì giá trị total mem ở mức min heap memory size.

```java
byte[] b = new byte[10 * 1024 * 1024];
System.out.println("Allocated 10M to the array");
System.out.println("Xmx=" + Runtime.getRuntime().maxMemory() / 1024.0 / 1024 + "M");  // Dung lượng tối đa của hệ thống
System.out.println("free mem=" + Runtime.getRuntime().freeMemory() / 1024.0 / 1024 + "M");  // Dung lượng trống của hệ thống
System.out.println("total mem=" + Runtime.getRuntime().totalMemory() / 1024.0 / 1024 + "M");  // Tổng dung lượng hiện có
```

![](https://static001.geekbang.org/infoq/b6/b6a7c522166dbd425dbb06eb56c9b071.png)

Lúc này chúng ta tạo một byte array 10M, min heap memory không thể đáp ứng nữa. Chúng ta sẽ thấy total memory hiện đã thành 15M; đây là kết quả của việc đã cấp phát thêm memory một lần.

Tiếp theo chạy lại code này:

```java
System.gc();
System.out.println("Xmx=" + Runtime.getRuntime().maxMemory() / 1024.0 / 1024 + "M");    // Dung lượng tối đa của hệ thống
System.out.println("free mem=" + Runtime.getRuntime().freeMemory() / 1024.0 / 1024 + "M");  // Dung lượng trống của hệ thống
System.out.println("total mem=" + Runtime.getRuntime().totalMemory() / 1024.0 / 1024 + "M");  // Tổng dung lượng hiện có
```

![](https://static001.geekbang.org/infoq/8d/8dd6e8fccfd1394b83251c136ee44ceb.png)

Lời gọi `System.gc()` ở đây chỉ gửi một request gợi ý garbage collection đến JVM, không bảo đảm chắc chắn Full GC sẽ thực thi. Lần chạy trong ảnh thực sự đã trigger collection tương ứng và thu nhỏ committed heap space, nhưng kết quả có thể khác với JVM parameter và collector khác nhau.

### 4.2 Điều chỉnh tỷ lệ giữa young generation và old generation

```plain
-XX:NewRatio --- Tỷ lệ giữa young generation (eden+2\*Survivor) và old generation (không bao gồm permanent area)

Ví dụ: -XX:NewRatio=4 nghĩa là young generation:old generation=1:4, tức young generation chiếm 1/5 toàn heap. Khi Xms=Xmx và đã thiết lập Xmn thì không cần thiết lập parameter này.
```

### 4.3 Điều chỉnh tỷ lệ giữa Survivor area và Eden area

```plain
-XX:SurvivorRatio (Survivor generation) --- Thiết lập tỷ lệ giữa hai Survivor và Eden

Ví dụ: 8 nghĩa là hai Survivor:Eden=2:8, tức một Survivor chiếm 1/10 young generation.
```

### 4.4 Thiết lập kích thước young generation và old generation

```plain
-XX:NewSize --- Thiết lập young generation size
-XX:MaxNewSize --- Thiết lập max value của young generation
```

Có thể test các tình huống khác nhau bằng cách thiết lập parameter khác nhau. Tỷ lệ ban đầu phổ biến của Eden và hai Survivor là 8:1:1, nhưng đây không phải lời giải tối ưu cho mọi application và collector; `-Xms` khác `-Xmx` cũng không có nghĩa chắc chắn sẽ dẫn đến nhiều lần GC.

### 4.5 Tóm tắt

Cần điều chỉnh kích thước young generation và Survivor area dựa trên load thực tế, garbage collector được sử dụng và GC log; không tồn tại tỷ lệ đề xuất cố định đúng với mọi application.

Khi OOM, nhớ Dump heap để có thể điều tra hiện trạng. Có thể dùng command dưới đây để output một file `.dump`; file này có thể được phân tích bằng các tool như VisualVM. Từ JDK 9, VisualVM không còn được cung cấp cùng Oracle JDK và cần cài đặt riêng.

```plain
-Xmx20m -Xms5m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=path to the log output
```

Thông thường cũng có thể viết script để gửi thông báo khi OOM xảy ra, chẳng hạn gửi email hoặc khởi động lại chương trình để xử lý.

### 4.6 Thiết lập permanent area (chỉ áp dụng cho HotSpot của JDK 7 trở về trước)

```plain
-XX:PermSize -XX:MaxPermSize
```

`PermSize` thiết lập initial size của permanent generation, `MaxPermSize` thiết lập giới hạn trên; default cụ thể phụ thuộc JVM version và platform, không thể viết thống nhất thành một tỷ lệ cố định của physical memory. Từ JDK 8 trở đi nên chú ý các parameter liên quan đến Metaspace thay vì hai parameter permanent generation này.

Mẹo: nếu heap space chưa dùng hết nhưng vẫn phát sinh OOM, nguyên nhân có thể là permanent area. Heap space thực tế chiếm rất ít nhưng permanent area overflow vẫn gây OOM.

### 4.7 JVM stack parameter tuning

#### 4.7.1 Điều chỉnh kích thước stack space của mỗi thread

Có thể dùng `-Xss` để điều chỉnh kích thước stack space của mỗi thread.

Kích thước mặc định của thread stack phụ thuộc JVM version, platform và mode chạy, không phải sau JDK 5 đều thống nhất là 1 MB. Khi các điều kiện khác giống nhau, giảm thread stack có thể dành địa chỉ cho nhiều platform thread hơn, nhưng số thread vẫn chịu giới hạn của operating system; stack quá nhỏ còn làm tăng nguy cơ `StackOverflowError`.

#### 4.7.2 Thiết lập kích thước thread stack

```plain
-XX:ThreadStackSize=<size>:
Thiết lập kích thước thread stack (0 means use default stack size)
```

Các parameter này đều có thể được test đơn giản bằng cách tự viết program; do giới hạn độ dài, ở đây không cung cấp demo nữa.

### 4.8 (Có thể bỏ qua) Giới thiệu các JVM parameter khác

Có rất nhiều parameter khác nhau, nên sẽ không trình bày tất cả; thực ra bạn cũng không nhất thiết phải đi sâu tìm hiểu từng parameter.

#### 4.8.1 Thiết lập kích thước large memory page

```plain
-XX:LargePageSizeInBytes=<size>
```

Parameter này dùng để thiết lập kích thước large memory page; có hiệu lực hay không phụ thuộc operating system, JVM build và cấu hình large page.

#### 4.8.2 Parameter cũ `UseFastAccessorMethods`

```plain
-XX:+UseFastAccessorMethods
```

HotSpot parameter cũ này tối ưu accessor cho việc truy cập bằng reflection, không phải “tối ưu nhanh cho primitive type”; JDK hiện đại không còn cung cấp parameter này.

#### 4.8.3 Tắt manual GC

```plain
-XX:+DisableExplicitGC:
Tắt System.gc() (parameter này cần được test nghiêm ngặt)
```

#### 4.8.4 Thiết lập tuổi tối đa của object

```plain
-XX:MaxTenuringThreshold
Thiết lập giới hạn trên của promotion age của object. Khi đặt bằng 0, generational collector hỗ trợ parameter này sẽ promote object sống trong young generation trực tiếp lên old generation mà không đi qua Survivor. Tăng giá trị này có thể khiến object trải qua nhiều lần copy hơn trong Survivor, nhưng promotion thực tế còn chịu ảnh hưởng của dynamic age judgment, capacity của Survivor và strategy của collector.
```

Không phải mọi garbage collector đều sử dụng cùng cơ chế generational và age promotion; có hiệu lực hay không cần xem documentation của collector hiện tại.

#### 4.8.5 Parameter cũ `AggressiveOpts`

```plain
-XX:+AggressiveOpts
```

Đây là parameter dùng trong HotSpot phiên bản cũ để bật experimental performance optimization, không thể hiểu đơn giản là “tăng tốc compile”, và đã bị loại bỏ trong JDK 12.

#### 4.8.6 Parameter cũ `UseBiasedLocking`

```plain
-XX:+UseBiasedLocking
```

Biased lock bị tắt mặc định và deprecated trong JDK 15; implementation liên quan sau đó đã bị loại bỏ khỏi HotSpot, không nên xem đây là parameter tuning dùng chung cho JDK hiện đại.

#### 4.8.7 Tắt class unloading

```plain
-Xnoclassgc
```

Parameter này tắt garbage collection đối với class, không phải garbage collection đối với object.

#### 4.8.8 Thiết lập thời gian tồn tại của object trong heap space

```plain
-XX:SoftRefLRUPolicyMSPerMB
Thiết lập thời gian tồn tại của SoftReference trên mỗi MB heap free space, default là 1s.
```

#### 4.8.9 Thiết lập cấp phát object trực tiếp trong old generation

```plain
-XX:PretenureSizeThreshold
Thiết lập kích thước mà object vượt quá sẽ được cấp phát trực tiếp trong old generation, default là 0.
```

#### 4.8.10 Thiết lập tỷ lệ diện tích Eden mà TLAB chiếm

```plain
-XX:TLABWasteTargetPercent
Thiết lập tỷ lệ phần trăm diện tích Eden mà TLAB chiếm, default là 1%.
```

## finally

Quả thật bài viết này khá dài. Bài viết tham khảo nhiều nguồn, bao gồm 《Phân tích chuyên sâu virtual machine》 và 《Giải thích chi tiết phỏng vấn Java Core》 của Geek Time, cùng Baidu và phần tổng hợp từ một số khóa học online trong quá trình tự học. Hy vọng bài viết hữu ích cho bạn. Cảm ơn.

<!-- @include: @article-footer.snippet.md -->
