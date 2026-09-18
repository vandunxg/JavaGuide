---
title: "Giải thích chi tiết cấu trúc Class file"
description: "Giới thiệu cấu trúc Class file bytecode Java và các thành phần cốt lõi như constant pool, hỗ trợ hiểu sản phẩm biên dịch."
category: Java
tag:
  - JVM
head:
  - - meta
    - name: keywords
      content: Class file,constant pool,magic number,version,field,method,attribute
---

## Ôn lại bytecode

Trong Java, code mà JVM có thể hiểu được gọi là `bytecode` (tức file có phần mở rộng `.class`). Nó không hướng đến bất kỳ processor cụ thể nào mà chỉ hướng đến virtual machine. Thông qua bytecode, ngôn ngữ Java phần nào giải quyết vấn đề hiệu suất thực thi thấp của các ngôn ngữ thông dịch truyền thống, đồng thời vẫn giữ được đặc điểm portable của ngôn ngữ thông dịch. Vì vậy, chương trình Java có hiệu suất tương đối cao khi chạy. Ngoài ra, do bytecode không nhắm đến một máy cụ thể nên chương trình Java có thể chạy trên máy tính sử dụng nhiều hệ điều hành khác nhau mà không cần biên dịch lại.

Clojure (một phương ngữ của ngôn ngữ Lisp), Groovy, Scala, JRuby, Kotlin và các ngôn ngữ khác đều chạy trên JVM. Hình dưới đây cho thấy các ngôn ngữ khác nhau được các compiler khác nhau biên dịch thành file `.class`, sau đó chạy trên JVM. Có thể dùng [WinHex](https://www.x-ways.net/winhex/) để xem format nhị phân của file `.class`.

![Các ngôn ngữ lập trình chạy trên JVM](https://oss.javaguide.cn/github/javaguide/java/basis/java-virtual-machine-program-language-os.png)

Có thể nói file `.class` là cầu nối quan trọng giữa các ngôn ngữ khác nhau và JVM, đồng thời cũng là một nguyên nhân quan trọng giúp Java cross-platform.

## Tổng hợp cấu trúc Class file

Theo đặc tả JVM, Class file được định nghĩa bằng `ClassFile`, khá giống với struct của ngôn ngữ C.

Cấu trúc của `ClassFile` như sau:

```java
ClassFile {
    u4             magic; //Dấu hiệu của Class file
    u2             minor_version;//Số hiệu phiên bản phụ của Class
    u2             major_version;//Số hiệu phiên bản chính của Class
    u2             constant_pool_count;//Số lượng constant pool
    cp_info        constant_pool[constant_pool_count-1];//Constant pool
    u2             access_flags;//Access flag của Class
    u2             this_class;//Class hiện tại
    u2             super_class;//Superclass
    u2             interfaces_count;//Số lượng interface
    u2             interfaces[interfaces_count];//Một class có thể implement nhiều interface
    u2             fields_count;//Số lượng field
    field_info     fields[fields_count];//Một class có thể có nhiều field
    u2             methods_count;//Số lượng method
    method_info    methods[methods_count];//Một class có thể có nhiều method
    u2             attributes_count;//Số lượng attribute trong attribute table của class này
    attribute_info attributes[attributes_count];//Tập hợp attribute table
}
```

Thông qua việc phân tích nội dung của `ClassFile`, ta có thể biết thành phần của Class file.

![Phân tích nội dung ClassFile](https://oss.javaguide.cn/java-guide-blog/16d5ec47609818fc.jpeg)

Hình dưới đây được xem bằng plugin `jclasslib` của IDEA, giúp bạn thấy cấu trúc Class file trực quan hơn.

![](https://oss.javaguide.cn/java-guide-blog/image-20210401170711475.png)

`jclasslib` không chỉ cho phép xem trực quan file bytecode tương ứng với một class, mà còn có thể xem các thông tin như thông tin cơ bản của class, constant pool, interface, attribute và function.

Dưới đây sẽ giới thiệu chi tiết một số component liên quan đến cấu trúc Class file.

### Magic number

```java
    u4             magic; //Dấu hiệu của Class file
```

4 byte đầu tiên của mỗi Class file được gọi là magic number. Tác dụng duy nhất của nó là **xác định file này có phải là Class file mà virtual machine có thể tiếp nhận hay không**. Đặc tả Java quy định magic number là giá trị cố định: 0xCAFEBABE. Nếu file được đọc không bắt đầu bằng magic number này, JVM sẽ từ chối load file.

### Version của Class file (Minor&Major Version)

```java
    u2             minor_version;//Số hiệu phiên bản phụ của Class
    u2             major_version;//Số hiệu phiên bản chính của Class
```

Bốn byte ngay sau magic number lưu version của Class file: byte thứ 5 và 6 là **minor version**, byte thứ 7 và 8 là **major version**.

Mỗi khi Java phát hành một major version (chẳng hạn Java 8, Java 9), major version sẽ tăng 1. Bạn có thể dùng command `javap -v` để nhanh chóng xem thông tin version của Class file.

JVM version cao có thể thực thi Class file do compiler version thấp tạo ra, nhưng JVM version thấp không thể thực thi Class file do compiler version cao tạo ra. Vì vậy, trong quá trình development thực tế, cần bảo đảm version JDK dùng để development và version JDK trong production environment nhất quán.

### Constant pool

```java
    u2             constant_pool_count;//Số lượng constant pool
    cp_info        constant_pool[constant_pool_count-1];//Constant pool
```

Ngay sau major và minor version là constant pool. Số lượng constant pool là `constant_pool_count-1` (**constant pool counter bắt đầu đếm từ 1, để trống constant pool entry thứ 0 là có chủ đích đặc biệt; index bằng 0 nghĩa là “không reference đến bất kỳ constant pool entry nào”**).

Constant pool chủ yếu lưu hai loại constant: literal và symbolic reference. Literal khá gần với khái niệm constant ở tầng ngôn ngữ Java, chẳng hạn text string và giá trị constant được khai báo bằng `final`. Symbolic reference thuộc về khái niệm trong compiler theory, gồm ba loại constant sau:

- Fully qualified name của class và interface
- Name và descriptor của field
- Name và descriptor của method

Mỗi constant trong constant pool là một table. Đặc tả JVM hiện tại định nghĩa 17 loại constant pool table. Chúng có một đặc điểm chung: **phần đầu là một `tag` kiểu `u1`, dùng để nhận diện type của constant hiện tại.**

|               Type               | Flag (`tag`) |                  Description                  |
| :------------------------------: | :----------: | :-------------------------------------------: |
|        CONSTANT_utf8_info        |      1       |         String được encode bằng UTF-8         |
|      CONSTANT_Integer_info       |      3       |                Integer literal                |
|       CONSTANT_Float_info        |      4       |                 Float literal                 |
|        CONSTANT_Long_info        |      5       |                 Long literal                  |
|       CONSTANT_Double_info       |      6       |        Double-precision float literal         |
|       CONSTANT_Class_info        |      7       |  Symbolic reference của class hoặc interface  |
|       CONSTANT_String_info       |      8       |                String literal                 |
|      CONSTANT_FieldRef_info      |      9       |         Symbolic reference của field          |
|     CONSTANT_MethodRef_info      |      10      |   Symbolic reference của method trong class   |
| CONSTANT_InterfaceMethodRef_info |      11      | Symbolic reference của method trong interface |
|    CONSTANT_NameAndType_info     |      12      |   Symbolic reference của field hoặc method    |
|     CONSTANT_MethodType_info     |      16      |              Chỉ ra method type               |
|    CONSTANT_MethodHandle_info    |      15      |            Biểu thị method handle             |
|      CONSTANT_Dynamic_info       |      17      |           Biểu thị dynamic constant           |
|   CONSTANT_InvokeDynamic_info    |      18      |     Biểu thị một dynamic method call site     |
|       CONSTANT_Module_info       |      19      |                Biểu thị module                |
|      CONSTANT_Package_info       |      20      |         Biểu thị package trong module         |

Có thể dùng instruction `javap -v class-name` để xem thông tin constant pool của file `.class` (`javap -v class-name -> temp.txt`: xuất kết quả vào file temp.txt).

### Access flag (Access Flags)

```java
    u2             access_flags;//Access flag của Class
```

Sau khi constant pool kết thúc, hai byte tiếp theo là access flag. Flag này dùng để nhận diện một số thông tin access ở cấp class hoặc interface, bao gồm Class này là class hay interface, có phải kiểu `public` hoặc `abstract` hay không, nếu là class thì có được khai báo là `final` hay không, v.v.

Access và modifier của class:

![Access và modifier của class](https://oss.javaguide.cn/github/javaguide/java/%E8%AE%BF%E9%97%AE%E6%A0%87%E5%BF%97.png)

Ta định nghĩa một class `Employee`:

```java
package top.snailclimb.bean;
public class Employee {
   ...
}
```

Dùng instruction `javap -v class-name` để xem access flag của class.

![Xem access flag của class](https://oss.javaguide.cn/github/javaguide/java/%E6%9F%A5%E7%9C%8B%E7%B1%BB%E7%9A%84%E8%AE%BF%E9%97%AE%E6%A0%87%E5%BF%97.png)

### Tập hợp index của class hiện tại (This Class), superclass (Super Class) và interface (Interfaces)

```java
    u2             this_class;//Class hiện tại
    u2             super_class;//Superclass
    u2             interfaces_count;//Số lượng interface
    u2             interfaces[interfaces_count];//Một class có thể implement nhiều interface
```

Quan hệ inheritance của Java class được xác định bởi ba thành phần: class index, superclass index và tập hợp interface index. Class index, superclass index và tập hợp interface index lần lượt nằm sau access flag.

Class index dùng để xác định fully qualified name của class này. Superclass index dùng để xác định fully qualified name của superclass của class này. Do ngôn ngữ Java chỉ hỗ trợ single inheritance nên chỉ có một superclass index. Ngoại trừ `java.lang.Object`, mọi Java class đều có superclass, vì vậy superclass index của mọi Java class ngoài `java.lang.Object` đều khác 0.

Tập hợp interface index dùng để mô tả class này implement những interface nào. Các interface được implement sẽ được sắp xếp từ trái sang phải trong tập hợp interface index theo thứ tự sau `implements` (nếu bản thân class này là interface thì là `extends`).

### Tập hợp field table (Fields)

```java
    u2             fields_count;//Số lượng field
    field_info     fields[fields_count];//Một class có thể có nhiều field
```

Field table (`field_info`) dùng để mô tả các variable được khai báo trong interface hoặc class. Field bao gồm class-level variable và instance variable, nhưng không bao gồm local variable được khai báo bên trong method.

**Cấu trúc của `field_info` (field table):**

![Cấu trúc field table](https://oss.javaguide.cn/github/javaguide/java/%E5%AD%97%E6%AE%B5%E8%A1%A8%E7%9A%84%E7%BB%93%E6%9E%84.png)

- **access_flags:** scope của field (modifier `public`, `private`, `protected`), là instance variable hay class variable (modifier `static`), có thể được serialize hay không (modifier `transient`), tính mutable (`final`), visibility (modifier `volatile`, có bắt buộc đọc/ghi từ main memory hay không).
- **name_index:** reference đến constant pool, biểu thị name của field;
- **descriptor_index:** reference đến constant pool, biểu thị descriptor của field và method;
- **attributes_count:** một field còn có thêm một số attribute, `attributes_count` lưu số lượng attribute;
- **attributes[attributes_count]:** lưu nội dung cụ thể của các attribute.

Trong các thông tin trên, mỗi modifier đều là boolean: hoặc có modifier nào đó, hoặc không có. Vì vậy, chúng rất phù hợp để biểu thị bằng bit flag. Còn name của field và data type mà field được định nghĩa là những giá trị không thể cố định, chỉ có thể reference đến constant trong constant pool để mô tả.

**Các giá trị của `access_flag` của field:**

![Các giá trị của `access_flag` của field](https://oss.javaguide.cn/github/javaguide/java/jvm/class-file-fields-access_flag.png)

### Tập hợp method table (Methods)

```java
    u2             methods_count;//Số lượng method
    method_info    methods[methods_count];//Một class có thể có nhiều method
```

`methods_count` biểu thị số lượng method, còn `method_info` biểu thị method table.

Trong format lưu trữ của Class file, mô tả method gần như hoàn toàn giống với mô tả field. Cấu trúc của method table cũng giống field table, lần lượt gồm access flag, name index, descriptor index và tập hợp attribute table.

**Cấu trúc của `method_info` (method table):**

![Cấu trúc method table](https://oss.javaguide.cn/github/javaguide/java/%E6%96%B9%E6%B3%95%E8%A1%A8%E7%9A%84%E7%BB%93%E6%9E%84.png)

**Các giá trị của `access_flag` của method:**

![Các giá trị của `access_flag` của method](https://oss.javaguide.cn/github/javaguide/java/jvm/class-file-methods-access_flag.png)

Lưu ý: vì modifier `volatile` và modifier `transient` không thể dùng cho method nên access flag của method table không có hai flag tương ứng này. Tuy nhiên, do các keyword như `synchronized`, `native`, `abstract` được dùng để modifier method, access flag cũng có thêm các flag tương ứng với những keyword này.

### Tập hợp attribute table (Attributes)

```java
   u2             attributes_count;//Số lượng attribute trong attribute table của class này
   attribute_info attributes[attributes_count];//Tập hợp attribute table
```

Trong Class file, field table và method table đều có thể mang tập hợp attribute table riêng để mô tả thông tin dành riêng cho một số trường hợp. Khác với yêu cầu về thứ tự, độ dài và nội dung của các data item khác trong Class file, giới hạn của tập hợp attribute table tương đối linh hoạt hơn. Các attribute table không cần có thứ tự nghiêm ngặt. Chỉ cần không trùng với tên attribute đã có, compiler do bất kỳ ai implement đều có thể ghi thông tin attribute do mình định nghĩa vào attribute table. Khi chạy, JVM sẽ bỏ qua các attribute mà nó không nhận diện.

## Tham khảo

- 《Thực chiến JVM》
- Chapter 4. The class File Format - Java Virtual Machine Specification: <https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html>
- Phân tích ví dụ về cấu trúc file JAVA CLASS: <https://coolshell.cn/articles/9229.html>
- 《Giải thích bằng sơ đồ nguyên lý JVM》 1.2.2, giải thích chi tiết về constant pool trong Class file (phần 1): <https://blog.csdn.net/luanlouis/article/details/39960815>

<!-- @include: @article-footer.snippet.md -->
