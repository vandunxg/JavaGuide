---
title: Giải thích chi tiết JMM (Java Memory Model)
description: Phân tích chuyên sâu Java Memory Model JMM gồm CPU cache model, cơ chế instruction reordering, nguyên tắc happens-before và bảo đảm memory visibility để hiểu đặc tả tầng dưới của concurrent programming.
category: Java
tag:
  - Java Concurrency
head:
  - - meta
    - name: keywords
      content: JMM,Java Memory Model,CPU cache,instruction reordering,happens-before,memory visibility,concurrent programming model
---

Với Java, bạn có thể xem **JMM (Java Memory Model)** là một tập hợp đặc tả liên quan đến concurrent programming do Java định nghĩa. Ngoài việc trừu tượng hóa mối quan hệ giữa thread và main memory, nó còn quy định quá trình chuyển đổi từ Java source code thành các instruction có thể thực thi trên CPU phải tuân thủ những nguyên tắc và đặc tả nào liên quan đến concurrency. Mục đích chính là **đơn giản hóa việc lập trình đa thread** và **tăng tính portable của chương trình**.

JMM chủ yếu định nghĩa **visibility** của một shared variable đối với các thread khác sau khi một thread thực hiện thao tác ghi.

Để hiểu thấu đáo JMM, chúng ta cần bắt đầu từ **CPU cache model** và **instruction reordering**.

## Bắt đầu từ CPU cache model

**Tại sao cần có CPU cache?** Tương tự cache mà chúng ta sử dụng khi phát triển hệ thống backend cho website, chẳng hạn Redis, nhằm giải quyết vấn đề tốc độ xử lý của chương trình không tương xứng với tốc độ truy cập relational database thông thường. **CPU cache nhằm giải quyết vấn đề tốc độ xử lý của CPU không tương xứng với tốc độ xử lý của memory.**

Thậm chí chúng ta có thể xem **memory là cache tốc độ cao của external storage**. Khi chương trình chạy, dữ liệu từ external storage được sao chép vào memory. Vì tốc độ xử lý của memory cao hơn external storage rất nhiều nên tốc độ xử lý được cải thiện.

Tóm lại: **CPU Cache cache dữ liệu của memory để giải quyết vấn đề tốc độ xử lý của CPU và memory không tương xứng; memory cache dữ liệu của disk để giải quyết vấn đề tốc độ truy cập disk quá chậm.**

Để dễ hiểu hơn, tôi vẽ một sơ đồ đơn giản về CPU Cache như sau.

> **🐛 Đính chính (tham khảo: [issue#1848](https://github.com/Snailclimb/JavaGuide/issues/1848))**: Hoàn thiện những điểm chưa chặt chẽ trong sơ đồ CPU cache model.

![Sơ đồ CPU cache model](https://oss.javaguide.cn/github/javaguide/java/concurrent/cpu-cache.png)

CPU Cache hiện đại thường được chia thành ba tầng, lần lượt gọi là L1, L2 và L3 Cache. Một số CPU có thể còn có L4 Cache, nhưng không bàn đến ở đây vì không phổ biến.

**Cách CPU Cache hoạt động:** trước tiên sao chép một bản dữ liệu vào CPU Cache. Khi CPU cần dùng, nó có thể đọc dữ liệu trực tiếp từ CPU Cache. Sau khi tính toán xong, dữ liệu nhận được từ phép tính sẽ được ghi lại vào Main Memory. Tuy nhiên, cách này tồn tại **vấn đề cache của memory không nhất quán**! Ví dụ, nếu tôi thực hiện thao tác `i++` và hai thread cùng thực hiện, giả sử cả hai thread đều đọc được `i=1` từ CPU Cache, sau khi cả hai thread tính `i++` rồi ghi lại vào Main Memory thì `i=2`, trong khi kết quả đúng phải là `i=3`.

**CPU có thể giải quyết vấn đề cache của memory không nhất quán bằng cách xây dựng cache consistency protocol (chẳng hạn [MESI protocol](https://zh.wikipedia.org/wiki/MESI%E5%8D%8F%E8%AE%AE)) hoặc các biện pháp khác.** Cache consistency protocol này chỉ những nguyên tắc và đặc tả cần tuân thủ khi CPU cache tương tác với main memory. Các CPU khác nhau thường sử dụng cache consistency protocol khác nhau.

![Cache consistency protocol](https://oss.javaguide.cn/github/javaguide/java/concurrent/cpu-cache-protocol.png)

Cache consistency của CPU được processor và memory subsystem phối hợp triển khai. Các processor architecture khác nhau còn quy định những memory access khác nhau có thể xuất hiện theo thứ tự nào khi được các processor khác quan sát; điều này thường được gọi là hardware memory model. JMM nằm ở tầng ngôn ngữ cao hơn, JVM cần ánh xạ yêu cầu của JMM vào các instruction và barrier do processor cụ thể cung cấp.

## Instruction reordering

Sau khi tìm hiểu CPU cache model, hãy xem một khái niệm quan trọng khác là **instruction reordering**.

Để tăng tốc độ thực thi và performance, khi máy tính thực thi program code, nó có thể reorder các instruction.

**Instruction reordering là gì?** Nói đơn giản, khi thực thi code, system không nhất thiết thực thi lần lượt theo thứ tự code mà bạn viết.

Có hai trường hợp instruction reordering phổ biến:

- **Compiler optimization reordering**: compiler, bao gồm JVM và JIT compiler, sắp xếp lại thứ tự thực thi của các câu lệnh với điều kiện không làm thay đổi semantics của chương trình single-thread.
- **Instruction-level parallel reordering**: processor hiện đại sử dụng instruction-level parallelism (Instruction-Level Parallelism, ILP) để thực thi chồng lấp nhiều instruction. Nếu không tồn tại data dependency, processor có thể thay đổi thứ tự thực thi của các machine instruction tương ứng với câu lệnh.

Ngoài ra, memory system cũng có “reordering”, nhưng không phải reordering theo nghĩa thực sự. Trong JMM, điều này thể hiện ở việc nội dung của main memory và local memory có thể không nhất quán, từ đó khiến chương trình gặp vấn đề khi chạy trong môi trường nhiều thread.

Java source code trải qua quá trình compile thành bytecode, interpret hoặc JIT compile, cuối cùng JVM thực thi các machine instruction tương ứng trên target platform. Compiler optimization, processor out-of-order execution và hành vi của memory subsystem đều có thể khiến nhiều thread quan sát thấy thứ tự khác với thứ tự trực quan của source code; chúng không tạo thành một “reordering pipeline” cố định và tuyến tính nghiêm ngặt.

**Instruction reordering có thể bảo đảm tính nhất quán của single-thread semantics, nhưng không có nghĩa vụ bảo đảm semantics giữa nhiều thread cũng nhất quán**, vì vậy instruction reordering có thể gây ra một số vấn đề trong môi trường nhiều thread.

Cách xử lý compiler optimization reordering và processor instruction reordering, trong đó instruction-level parallel reordering và memory system reordering đều thuộc processor-level instruction reordering, là khác nhau.

- Với compiler, ngăn reordering bằng cách cấm những loại compiler reordering cụ thể.

- Với processor, ngăn những loại processor reordering cụ thể bằng cách chèn memory barrier, đôi khi còn gọi là memory fence.

> Memory barrier, còn gọi là memory fence, dùng để ràng buộc thứ tự và visibility giữa các memory access cụ thể. Cách triển khai cụ thể khác nhau giữa các processor architecture, không thể hiểu một cách máy móc là “flush cache vào physical memory” hoặc “làm toàn bộ cache invalid”; JVM sẽ chọn instruction phù hợp với target platform để đáp ứng semantics của JMM.

## JMM (Java Memory Model)

### JMM là gì? Tại sao cần JMM?

Java là một trong những programming language đầu tiên thử cung cấp memory model. Do memory model thời kỳ đầu tồn tại một số hạn chế, chẳng hạn rất dễ làm suy yếu khả năng optimization của compiler, từ Java 5 Java bắt đầu sử dụng memory model mới [《JSR-133: Java Memory Model and Thread Specification》](http://www.cs.umd.edu/~pugh/java/memoryModel/CommunityReview.pdf).

Nói chung, programming language cũng có thể dùng trực tiếp memory model ở tầng operating system. Tuy nhiên, memory model của các operating system khác nhau. Nếu dùng trực tiếp memory model ở tầng operating system, cùng một bộ code có thể không chạy được khi chuyển sang operating system khác. Java là một platform-independent language nên cần tự cung cấp một memory model để che giấu khác biệt giữa các system.

Đó chỉ là một trong những lý do JMM tồn tại. Trên thực tế, với Java, bạn có thể xem JMM là một tập hợp đặc tả liên quan đến concurrent programming do Java định nghĩa. Ngoài việc trừu tượng hóa mối quan hệ giữa thread và main memory, nó còn quy định quá trình chuyển đổi từ Java source code thành instruction có thể thực thi trên CPU phải tuân thủ những nguyên tắc và đặc tả nào liên quan đến concurrency. Mục đích chính là đơn giản hóa việc lập trình đa thread và tăng tính portable của chương trình.

**Tại sao phải tuân thủ những nguyên tắc và đặc tả liên quan đến concurrency này?** Bởi vì trong concurrent programming, những thiết kế như CPU multi-level cache và instruction reordering có thể khiến chương trình phát sinh vấn đề khi chạy. Ví dụ, instruction reordering được đề cập ở trên có thể khiến chương trình nhiều thread thực thi sai. Vì vậy, JMM trừu tượng hóa nguyên tắc happens-before, sẽ được giới thiệu chi tiết ở phần sau, để giải quyết vấn đề instruction reordering này.

Nói ngắn gọn, JMM định nghĩa một số đặc tả để giải quyết những vấn đề này, giúp developer thuận tiện hơn khi phát triển chương trình nhiều thread. Với Java developer, bạn không cần hiểu nguyên lý tầng dưới; chỉ cần sử dụng các keyword và class liên quan đến concurrency, chẳng hạn `volatile`, `synchronized` và các `Lock` khác nhau, là có thể phát triển chương trình an toàn trong môi trường concurrency.

### JMM trừu tượng hóa mối quan hệ giữa thread và main memory như thế nào?

**Java Memory Model (JMM)** trừu tượng hóa mối quan hệ giữa thread và main memory. Chẳng hạn, shared variable giữa các thread phải được lưu trữ trong main memory.

Java đã có memory model từ những specification đầu tiên; Java 5 đã sửa đổi quan trọng memory model này thông qua JSR-133, đồng thời làm rõ và tăng cường semantics của `volatile`, `final` và happens-before. JMM cho phép JVM sử dụng register, cache và compiler optimization để triển khai việc truy cập shared variable; nếu chương trình không thiết lập synchronization relation cần thiết, một thread có thể không nhìn thấy lần ghi mới nhất của thread khác.

Điều này rất giống CPU cache model đã nói ở trên.

**Main memory là gì? Local memory là gì?**

- **Main memory**: JMM dùng khái niệm này để trừu tượng hóa các variable mà các thread có thể chia sẻ, bao gồm instance field, static field và array element. Method local variable, parameter và exception handler parameter không được chia sẻ giữa các thread, nên không thuộc shared variable được nói đến ở đây. Main memory là một khái niệm ở tầng specification, không tương đương với một vùng physical memory cụ thể.
- **Local memory**: mỗi thread có một local memory riêng. Local memory lưu bản sao của shared variable mà thread đó đã đọc / ghi. Mỗi thread chỉ có thể thao tác với variable trong local memory của chính nó, không thể trực tiếp truy cập local memory của thread khác. Nếu các thread cần giao tiếp, chúng phải thực hiện thông qua main memory. Local memory là một khái niệm được JMM trừu tượng hóa, không thực sự tồn tại; nó bao gồm cache, write buffer, register cũng như các optimization khác của hardware và compiler.

Sơ đồ trừu tượng của Java Memory Model như sau:

![JMM (Java Memory Model)](https://oss.javaguide.cn/github/javaguide/java/concurrent/jmm.png)

Theo hình trên, nếu thread 1 và thread 2 muốn giao tiếp với nhau thì phải trải qua hai bước sau:

1. Thread 1 đồng bộ value của bản sao shared variable đã được sửa trong local memory vào main memory.
2. Thread 2 đọc value của shared variable tương ứng từ main memory.

Nói cách khác, shared data giữa các thread cần tuân thủ rule của JMM để giao tiếp; chỉ khi thiết lập được happens-before relation tương ứng thông qua `volatile`, lock, thread start và termination thì JMM mới cung cấp visibility guarantee cho các lần ghi liên quan.

Tuy nhiên, trong môi trường nhiều thread, thao tác với một shared variable trong main memory có thể gây ra thread-safety issue. Ví dụ:

1. Thread 1 và thread 2 lần lượt thao tác với cùng một shared variable, một thread thực hiện sửa và thread còn lại thực hiện đọc.
2. Không thể xác định thread 2 đọc được value trước hay sau khi thread 1 sửa; cả hai đều có thể xảy ra, vì thread 1 và thread 2 đều trước tiên copy shared variable từ main memory vào working memory tương ứng.

Về synchronization protocol cụ thể giữa main memory và working memory, tức chi tiết triển khai về cách một variable được copy từ main memory vào working memory và cách đồng bộ từ working memory về main memory, Java Memory Model định nghĩa tám synchronization operation sau đây, chỉ cần hiểu, không cần học thuộc máy móc:

- **lock**: tác động lên variable trong main memory, đánh dấu variable là variable độc quyền của một thread.
- **unlock**: tác động lên variable trong main memory, gỡ trạng thái lock của variable. Chỉ variable đã được gỡ lock mới có thể được thread khác lock.
- **read**: tác động lên variable trong main memory, truyền value của variable từ main memory vào working memory của thread để thao tác load tiếp theo sử dụng.
- **load**: đặt value của variable nhận được từ main memory qua thao tác read vào bản sao của variable trong working memory.
- **use**: truyền value của một variable trong working memory cho execution engine. Mỗi khi VM gặp instruction sử dụng variable, nó sẽ thực hiện thao tác này.
- **assign**: tác động lên variable trong working memory, gán value nhận được từ execution engine cho variable trong working memory. Mỗi khi VM gặp bytecode instruction gán value cho variable, thao tác này được thực hiện.
- **store**: tác động lên variable trong working memory, truyền value của variable trong working memory vào main memory để thao tác write tiếp theo sử dụng.
- **write**: tác động lên variable trong main memory, đặt value của variable nhận được từ working memory qua thao tác store vào variable trong main memory.

Ngoài tám synchronization operation này, JMM còn quy định các synchronization rule sau để bảo đảm những synchronization operation được thực hiện chính xác, chỉ cần hiểu, không cần học thuộc máy móc:

- Không cho phép một thread đồng bộ data từ working memory về main memory mà không có lý do, tức chưa từng thực hiện thao tác assign nào.
- Một variable mới chỉ có thể “sinh ra” trong main memory. Không được trực tiếp sử dụng một variable chưa được initialize, tức chưa thực hiện load hoặc assign, trong working memory. Nói cách khác, trước khi thực hiện use và store với một variable, phải thực hiện assign và load trước.
- Tại cùng một thời điểm, chỉ một thread được phép thực hiện thao tác lock với một variable. Tuy nhiên, cùng một thread có thể thực hiện lock nhiều lần; sau nhiều lần lock, variable chỉ được unlock khi thực hiện đúng số lần unlock tương ứng.
- Nếu thực hiện thao tác lock với một variable, value của variable đó trong working memory sẽ bị xóa. Trước khi execution engine sử dụng variable, cần thực hiện lại load hoặc assign để initialize value của variable.
- Nếu một variable chưa từng được lock thì không được thực hiện unlock với nó; cũng không được unlock một variable đang bị thread khác lock.
- …

### Java memory area khác JMM như thế nào?

Đây là một câu hỏi khá phổ biến, nhiều người mới học rất dễ nhầm lẫn. **Java memory area và memory model là hai khái niệm hoàn toàn khác nhau:**

- JVM memory structure liên quan đến runtime area của Java Virtual Machine, định nghĩa cách JVM phân vùng để lưu trữ program data khi chạy. Chẳng hạn, heap chủ yếu dùng để lưu object instance.
- Java Memory Model liên quan đến concurrent programming của Java, trừu tượng hóa mối quan hệ giữa thread và main memory. Chẳng hạn, shared variable giữa các thread phải được lưu trong main memory. JMM quy định quá trình chuyển đổi từ Java source code thành instruction có thể thực thi trên CPU phải tuân thủ những nguyên tắc và đặc tả nào liên quan đến concurrency. Mục đích chính là đơn giản hóa việc lập trình đa thread và tăng tính portable của chương trình.

### Nguyên tắc happens-before là gì?

Khái niệm happens-before xuất hiện lần đầu trong paper [《Time, Clocks and the Ordering of Events in a Distributed System》](https://lamport.azurewebsites.net/pubs/time-clocks.pdf) do Leslie Lamport công bố năm 1978. Trong paper này, Leslie Lamport đề xuất khái niệm [logical clock](https://writings.sh/post/logical-clocks), trở thành algorithm logical clock đầu tiên. Trong distributed environment, sự thay đổi của logical clock được định nghĩa thông qua một loạt rule, từ đó có thể dùng logical clock để phán đoán thứ tự trước sau của các event trong distributed system. **Logical clock không đo bản thân thời gian, mà chỉ phân biệt thứ tự trước sau của event; về bản chất, nó định nghĩa một happens-before relation.**

Bối cảnh ra đời của khái niệm happens-before đã đề cập ở trên không phải trọng tâm, chỉ cần hiểu sơ lược.

JSR-133 đưa vào khái niệm happens-before để mô tả memory visibility giữa hai operation.

**Tại sao cần nguyên tắc happens-before?** Nguyên tắc happens-before ra đời để cân bằng giữa programmer với compiler và processor. Programmer mong muốn một strong memory model dễ hiểu và dễ lập trình, chỉ cần code theo rule đã định là được. Compiler và processor mong muốn một weak memory model ít ràng buộc hơn để tối ưu performance hết mức có thể. Design idea của nguyên tắc happens-before thực ra rất đơn giản:

- Để giảm ràng buộc đối với compiler và processor hết mức có thể, miễn không thay đổi execution result của chương trình, bao gồm single-thread program và multi-thread program thực thi đúng, compiler và processor có thể tùy ý thực hiện reordering optimization.
- Với reordering làm thay đổi execution result của chương trình, JMM yêu cầu compiler và processor phải cấm reordering đó.

Hình dưới đây là sơ đồ design idea của JMM mà tôi vẽ lại dựa trên một hình trong cuốn _Nghệ thuật lập trình concurrent của Java_.

![Design idea của JMM](https://oss.javaguide.cn/github/javaguide/java/concurrent/jmm-design-idea.png)

Sau khi hiểu design idea của nguyên tắc happens-before, hãy xem định nghĩa của JSR-133 về nguyên tắc happens-before:

- Nếu một operation happens-before operation khác, execution result của operation đầu tiên sẽ visible đối với operation thứ hai, đồng thời thứ tự thực thi của operation đầu tiên đứng trước operation thứ hai.
- Giữa hai operation tồn tại happens-before relation không có nghĩa implementation cụ thể của Java platform bắt buộc phải thực thi theo thứ tự do happens-before relation chỉ định. Nếu execution result sau reordering giống với result khi thực thi theo happens-before relation, JMM cũng cho phép reordering đó.

Xem đoạn code sau:

```java
int userNum = getUserNum();   // 1
int teacherNum = getTeacherNum();   // 2
int totalNum = userNum + teacherNum;  // 3
```

- 1 happens-before 2
- 2 happens-before 3
- 1 happens-before 3

Mặc dù 1 happens-before 2, nhưng reordering 1 và 2 không ảnh hưởng đến execution result của code, nên JMM cho phép compiler và processor thực hiện reordering này. Tuy nhiên, 1 và 2 phải được thực hiện trước 3, tức là 1, 2 happens-before 3.

**Ý nghĩa mà nguyên tắc happens-before biểu đạt thực ra không phải là một operation xảy ra trước operation khác, dù hiểu như vậy từ góc nhìn programmer cũng không gây trở ngại. Chính xác hơn, nó muốn biểu đạt rằng result của operation trước visible đối với operation sau, bất kể hai operation có ở cùng một thread hay không.**

Ví dụ, operation 1 happens-before operation 2. Ngay cả khi operation 1 và operation 2 không nằm trong cùng một thread, JMM vẫn bảo đảm result của operation 1 visible đối với operation 2.

### Những rule phổ biến của happens-before là gì? Hãy trình bày cách hiểu của bạn.

happens-before có nhiều rule. Dưới đây là năm rule thường dùng nhất:

1. **Program order rule**: trong cùng một thread, theo thứ tự code, operation được viết trước happens-before operation được viết sau;
2. **Monitor lock rule**: unlock một monitor happens-before lock tiếp theo trên cùng monitor đó;
3. **volatile variable rule**: thao tác ghi một `volatile` variable happens-before thao tác đọc tiếp theo trên cùng variable đó;
4. **Transitivity rule**: nếu A happens-before B và B happens-before C thì A happens-before C;
5. **Thread start rule**: method `start()` của object Thread happens-before mọi action của thread đó.

Danh sách này chưa đầy đủ, còn bao gồm rule về thread termination và thread interruption. Nếu không thể suy ra happens-before relation giữa hai conflicting access thông qua đầy đủ các rule, chúng có thể tạo thành data race; visibility và order của chúng không thể được bảo đảm theo trực giác của single-thread. Điều này không tương đương với việc JVM có thể vô điều kiện hoán đổi bất kỳ hai instruction nào; execution result vẫn chịu ràng buộc bởi consistency và causality rule của JMM.

### happens-before có quan hệ gì với JMM?

Quan hệ giữa happens-before và JMM được thể hiện trong hình sau:

![jmm-vs-happens-before](https://oss.javaguide.cn/github/javaguide/java/concurrent/jmm-vs-happens-before.png)

- JMM cung cấp cho programmer **“happens-before rule”** như program order rule và `volatile` variable rule. Đây là ảo giác về một **“strong memory model”**: programmer không cần quan tâm đến chi tiết reordering phức tạp ở tầng dưới, chỉ cần viết code theo các rule này là có thể bảo đảm memory visibility khi chạy nhiều thread.
- Khi thực thi, JVM ánh xạ happens-before rule vào implementation cụ thể. Để không làm mất performance trong khi vẫn bảo đảm correctness, JMM chỉ **“cấm reordering ảnh hưởng đến execution result”**. JMM cho phép reordering không ảnh hưởng đến execution result của single-thread.
- Tầng thấp nhất là **“reordering rule”** thực tế của compiler và processor.

Tóm lại, JMM giống như một intermediate layer: phía trên, nó cung cấp programming model đơn giản cho programmer thông qua happens-before; phía dưới, nó tận dụng performance của hardware bằng cách cấm một số reordering cụ thể. Design này vừa bảo đảm safety cho nhiều thread, vừa giải phóng tối đa performance của hardware.

## Xem lại ba đặc tính quan trọng của concurrent programming

### Atomicity

Một operation hoặc một nhóm operation hoặc được thực thi toàn bộ và không bị bất kỳ yếu tố nào can thiệp làm gián đoạn, hoặc hoàn toàn không được thực thi.

Trong Java, có thể dùng `synchronized`, các `Lock` và các atomic class khác nhau để triển khai atomicity.

`synchronized` và các `Lock` bảo đảm tại bất kỳ thời điểm nào chỉ có một thread truy cập code block đó, do đó có thể bảo đảm atomicity. Các atomic class dùng thao tác CAS (compare and swap), có thể cũng sử dụng keyword `volatile` hoặc `final`, để bảo đảm atomic operation.

### Visibility

Khi một thread sửa shared variable, các thread khác có thể lập tức nhìn thấy value mới nhất sau khi sửa.

Trong Java, có thể dùng `synchronized`, `volatile` và các `Lock` để triển khai visibility.

Nếu khai báo variable là `volatile`, happens-before relation sẽ được thiết lập giữa thao tác ghi variable đó và thao tác đọc sau đó. JVM phải bảo đảm semantics về visibility và order tương ứng, nhưng implementation cụ thể không bắt buộc phải truy cập physical main memory mỗi lần.

### Ordering

Do vấn đề instruction reordering, execution order của code chưa chắc là thứ tự viết code.

Khi nói về reordering ở trên, chúng ta cũng đã đề cập:

> **Instruction reordering có thể bảo đảm tính nhất quán của single-thread semantics, nhưng không có nghĩa vụ bảo đảm semantics giữa nhiều thread cũng nhất quán**, vì vậy instruction reordering có thể gây ra một số vấn đề trong môi trường nhiều thread.

Trong Java, `volatile` sẽ ràng buộc các reordering liên quan đến việc đọc ghi variable đó và có thể phá vỡ memory semantics của variable, nhưng không cấm mọi instruction reordering optimization.

## Tổng kết

- Java là một trong những language đầu tiên thử cung cấp memory model. Mục đích chính là đơn giản hóa việc lập trình đa thread và tăng tính portable của chương trình.
- CPU có thể giải quyết vấn đề cache của memory không nhất quán bằng cách xây dựng cache consistency protocol, chẳng hạn [MESI protocol](https://zh.wikipedia.org/wiki/MESI%E5%8D%8F%E8%AE%AE).
- Để tăng tốc độ thực thi và performance, khi máy tính thực thi program code, nó có thể reorder instruction. Nói đơn giản, system không nhất thiết thực thi code lần lượt theo thứ tự bạn viết. **Instruction reordering có thể bảo đảm tính nhất quán của single-thread semantics, nhưng không có nghĩa vụ bảo đảm semantics giữa nhiều thread cũng nhất quán**, vì vậy instruction reordering có thể gây ra một số vấn đề trong môi trường nhiều thread.
- Bạn có thể xem JMM là một tập hợp đặc tả liên quan đến concurrent programming do Java định nghĩa. Ngoài việc trừu tượng hóa mối quan hệ giữa thread và main memory, nó còn quy định quá trình chuyển đổi từ Java source code thành instruction có thể thực thi trên CPU phải tuân thủ những nguyên tắc và đặc tả nào liên quan đến concurrency. Mục đích chính là đơn giản hóa việc lập trình đa thread và tăng tính portable của chương trình.
- JSR-133 đưa vào khái niệm happens-before để mô tả memory visibility giữa hai operation.

## Tài liệu tham khảo

- _Nghệ thuật lập trình concurrent của Java_, chương 3: Java Memory Model
- _Java Multi-threading từ cơ bản đến nâng cao_: <http://concurrent.redspider.group/RedSpider.html>
- Nghiên cứu về Java memory access reordering: <https://tech.meituan.com/2014/09/23/java-memory-reordering.html>
- Này bạn, Java Memory Model (JMM) mà bạn cần đây: <https://xie.infoq.cn/article/739920a92d0d27e2053174ef2>
- JSR 133 (Java Memory Model) FAQ: <https://www.cs.umd.edu/~pugh/java/memoryModel/jsr-133-faq.html>

<!-- @include: @article-footer.snippet.md -->
