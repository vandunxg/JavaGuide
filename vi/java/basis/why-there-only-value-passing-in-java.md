---
title: "Giải thích chi tiết về pass-by-value trong Java"
description: "Giải thích vì sao Java chỉ có pass-by-value: phân tích cơ chế truyền parameter qua ví dụ, làm rõ những hiểu lầm về pass-by-value và pass-by-reference, cùng bản chất của formal parameter và actual argument."
category: Java
tag:
  - Java Basics
head:
  - - meta
    - name: keywords
      content: Java pass-by-value,pass-by-reference,parameter passing,formal parameter,actual argument,object reference,method invocation,Java parameter passing mechanism
---

Trước khi bắt đầu, hãy làm rõ hai khái niệm sau:

- formal parameter & actual argument
- pass-by-value & pass-by-reference

## Formal Parameter & Actual Argument

Khi định nghĩa method, ta có thể dùng **parameter** (method có parameter). Trong ngôn ngữ lập trình, parameter gồm:

- **Actual argument (actual parameter, argument)**: parameter được truyền vào function/method, phải có giá trị xác định.
- **Formal parameter (formal parameter, parameter)**: dùng để định nghĩa function/method và nhận actual argument, không cần có giá trị xác định.

```java
String hello = "Hello!";
// hello là actual argument
sayHello(hello);
// str là formal parameter
void sayHello(String str) {
    System.out.println(str);
}
```

## Pass-by-value & Pass-by-reference

Ngôn ngữ lập trình có hai cách truyền actual argument cho method (hoặc function):

- **Pass-by-value**: method nhận bản sao của giá trị actual argument.
- **Pass-by-reference**: method nhận trực tiếp địa chỉ của actual argument thay vì giá trị bên trong actual argument; đây chính là pointer. Khi đó formal parameter chính là actual argument, mọi thay đổi đối với formal parameter đều phản ánh vào actual argument, bao gồm cả việc gán lại giá trị.

Nhiều ngôn ngữ lập trình (chẳng hạn C++ và Pascal) cung cấp hai cách truyền parameter, nhưng Java chỉ có pass-by-value.

## Vì sao Java chỉ có pass-by-value?

**Vì sao nói Java chỉ có pass-by-value?** Không cần dài dòng, tôi sẽ chứng minh qua 3 ví dụ.

### Ví dụ 1: Truyền parameter thuộc primitive type

Code:

```java
public static void main(String[] args) {
    int num1 = 10;
    int num2 = 20;
    swap(num1, num2);
    System.out.println("num1 = " + num1);
    System.out.println("num2 = " + num2);
}

public static void swap(int a, int b) {
    int temp = a;
    a = b;
    b = temp;
    System.out.println("a = " + a);
    System.out.println("b = " + b);
}
```

Kết quả:

```plain
a = 20
b = 10
num1 = 10
num2 = 20
```

Phân tích:

Trong method `swap()`, giá trị của `a` và `b` được hoán đổi nhưng không ảnh hưởng đến `num1` và `num2`. Vì giá trị của `a` và `b` chỉ là bản sao của `num1` và `num2`, dù thay đổi bản sao thế nào cũng không ảnh hưởng đến bản gốc.

![](https://oss.javaguide.cn/github/javaguide/java/basis/java-value-passing-01.png)

Qua ví dụ trên, ta biết method không thể sửa parameter thuộc primitive type. Tuy nhiên, object reference được dùng làm parameter lại khác; hãy xem ví dụ 2.

### Ví dụ 2: Truyền parameter thuộc reference type 1

Code:

```java
  public static void main(String[] args) {
      int[] arr = { 1, 2, 3, 4, 5 };
      System.out.println(arr[0]);
      change(arr);
      System.out.println(arr[0]);
  }

  public static void change(int[] array) {
      // Đổi phần tử đầu tiên của array thành 0
      array[0] = 0;
  }
```

Output:

```plain
1
0
```

Phân tích:

![](https://oss.javaguide.cn/github/javaguide/java/basis/java-value-passing-02.png)

Sau ví dụ này, nhiều người hẳn sẽ cho rằng Java sử dụng pass-by-reference cho parameter thuộc reference type.

Thực ra không phải: thứ được truyền vẫn là giá trị, nhưng đó là bản sao của object reference.

Nói cách khác, parameter của method `change` là bản sao của reference value được lưu trong `arr`; vì vậy formal parameter và `arr` cùng trỏ đến một array object. Do đó, bên gọi có thể quan sát được thay đổi đối với nội dung array qua formal parameter.

Để thấy rõ hơn Java không sử dụng pass-by-reference cho parameter thuộc reference type, hãy xem ví dụ tiếp theo!

### Ví dụ 3: Truyền parameter thuộc reference type 2

```java
public class Person {
    private String name;
   // Lược bỏ constructor, getter và setter
}

public static void main(String[] args) {
    Person xiaoZhang = new Person("Xiao Zhang");
    Person xiaoLi = new Person("Xiao Li");
    swap(xiaoZhang, xiaoLi);
    System.out.println("xiaoZhang:" + xiaoZhang.getName());
    System.out.println("xiaoLi:" + xiaoLi.getName());
}

public static void swap(Person person1, Person person2) {
    Person temp = person1;
    person1 = person2;
    person2 = temp;
    System.out.println("person1:" + person1.getName());
    System.out.println("person2:" + person2.getName());
}
```

Output:

```plain
person1:Xiao Li
person2:Xiao Zhang
xiaoZhang:Xiao Zhang
xiaoLi:Xiao Li
```

Phân tích:

Chuyện gì xảy ra vậy? Hai formal parameter thuộc reference type được hoán đổi nhưng lại không ảnh hưởng đến actual argument!

Parameter `person1` và `person2` của method `swap` chỉ là bản sao của reference value được lưu trong actual argument `xiaoZhang` và `xiaoLi`. Vì vậy, việc hoán đổi `person1` và `person2` chỉ hoán đổi các bản sao của reference trong formal parameter, không thay đổi giá trị của các biến `xiaoZhang` và `xiaoLi` ở phía gọi.

![](https://oss.javaguide.cn/github/javaguide/java/basis/java-value-passing-03.png)

## Pass-by-reference thực sự như thế nào?

Đến đây, chắc bạn đã biết Java chỉ có pass-by-value, không có pass-by-reference.
Nhưng pass-by-reference thực sự là gì? Dưới đây là ví dụ bằng code `C++`, giúp bạn thấy rõ cơ chế này.

```C++
#include <iostream>

void incr(int& num)
{
    std::cout << "incr before: " << num << "\n";
    num++;
    std::cout << "incr after: " << num << "\n";
}

int main()
{
    int age = 10;
    std::cout << "invoke before: " << age << "\n";
    incr(age);
    std::cout << "invoke after: " << age << "\n";
}
```

Kết quả:

```plain
invoke before: 10
incr before: 10
incr after: 11
invoke after: 11
```

Phân tích: Trong function `incr`, việc sửa formal parameter có thể ảnh hưởng đến giá trị của actual argument. Lưu ý: kiểu dữ liệu của formal parameter trong `incr` phải là `int&` thì mới là pass-by-reference; nếu dùng `int` thì vẫn là pass-by-value!

## Vì sao Java không đưa pass-by-reference vào?

Pass-by-reference thoạt nhìn có vẻ tiện, vì có thể trực tiếp sửa giá trị của actual argument bên trong method. Vậy vì sao Java không đưa pass-by-reference vào?

**Lưu ý: Phần dưới đây là quan điểm cá nhân, không phải thông tin chính thức từ Java:**

1. Vì lý do an toàn, caller không biết các thao tác lên giá trị bên trong method (coi method là một interface, caller không quan tâm implementation cụ thể). Hãy thử tưởng tượng: bạn cầm thẻ ngân hàng đi rút tiền, muốn rút 100 nhưng tài khoản bị trừ 200, chẳng phải rất đáng sợ sao?
2. Cha đẻ của Java, James Gosling, đã nhìn thấy nhiều nhược điểm của C và C++ ngay từ khi thiết kế, nên muốn tạo ra một ngôn ngữ mới là Java. Khi thiết kế Java, ông tuân theo nguyên tắc đơn giản, dễ sử dụng và loại bỏ nhiều “đặc tính” mà developer có thể vô tình dùng sai gây ra vấn đề. Bản thân ngôn ngữ có ít thứ hơn thì developer cũng cần học ít thứ hơn.

## Tổng kết

Java truyền actual argument cho method (hoặc function) theo cách **pass-by-value**:

- Nếu parameter là primitive type thì rất đơn giản: thứ được truyền là bản sao của literal value thuộc primitive type.
- Nếu parameter là reference type thì thứ được truyền là bản sao của reference value. Formal parameter và actual argument ban đầu trỏ đến cùng một object, nhưng việc gán lại giá trị cho formal parameter sẽ không thay đổi biến actual argument.

## Tham khảo

- 《Java Core Technology, Volume I》, kiến thức cơ bản, phiên bản thứ mười, chương 4, mục 4.5
- [Java rốt cuộc là pass-by-value hay pass-by-reference? - Câu trả lời của Hollis - Zhihu](https://www.zhihu.com/question/31203609/answer/576030121)
- [Oracle Java Tutorials - Passing Information to a Method or a Constructor](https://docs.oracle.com/javase/tutorial/java/javaOO/arguments.html)
- [Interview with James Gosling, Father of Java](https://mappingthejourney.com/single-post/2017/06/29/episode-3-interview-with-james-gosling-father-of-java/)

<!-- @include: @article-footer.snippet.md -->
