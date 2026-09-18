---
title: Một số bài lập trình trong Sword Finger Offer
description: Tuyển chọn các bài lập trình thường gặp trong Sword Finger Offer, cung cấp nhiều cách tiếp cận và ví dụ như đệ quy, lặp, giúp ôn tập hiệu quả các dạng bài thường xuất hiện.
category: Kiến thức cơ bản về máy tính
tag:
  - Thuật toán
head:
  - - meta
    - name: keywords
      content: Sword Finger Offer,Fibonacci,đệ quy,lặp,danh sách liên kết,mảng,câu hỏi phỏng vấn
---

# Một số bài lập trình trong Sword Finger Offer

## Dãy Fibonacci

**Mô tả bài toán:**

Mọi người đều biết dãy Fibonacci. Bây giờ yêu cầu nhập một số nguyên n và xuất phần tử thứ n của dãy Fibonacci. n<=39

**Phân tích vấn đề:**

Có thể khẳng định bài này chắc chắn giải được bằng đệ quy, nhưng cách này có một vấn đề lớn: việc tính toán trùng lặp quá nhiều khi đệ quy sẽ gây tràn bộ nhớ. Ngoài ra có thể dùng phương pháp lặp, lưu kết quả trong quá trình tính bằng fn1 và fn2 rồi tái sử dụng. Dưới đây là code ví dụ của cả hai phương pháp và so sánh thời gian chạy của chúng.

**Code ví dụ:**

Dùng phương pháp lặp:

```java
int Fibonacci(int number) {
    if (number <= 0) {
        return 0;
    }
    if (number == 1 || number == 2) {
        return 1;
    }
    int first = 1, second = 1, third = 0;
    for (int i = 3; i <= number; i++) {
        third = first + second;
        first = second;
        second = third;
    }
    return third;
}
```

Dùng đệ quy:

```java
public int Fibonacci(int n) {
    if (n <= 0) {
        return 0;
    }
    if (n == 1||n==2) {
        return 1;
    }

    return Fibonacci(n - 2) + Fibonacci(n - 1);
}
```

## Bài toán bậc thang

**Mô tả bài toán:**

Một con ếch mỗi lần có thể nhảy lên 1 bậc hoặc 2 bậc. Hỏi con ếch đó có tổng cộng bao nhiêu cách để nhảy lên một cầu thang n bậc.

**Phân tích vấn đề:**

Phân tích theo cách thông thường:

> a. Có hai cách nhảy: 1 bậc hoặc 2 bậc. Giả sử lần đầu nhảy 1 bậc, còn lại n-1 bậc, số cách nhảy là f(n-1);
> b. Giả sử lần đầu nhảy 2 bậc, còn lại n-2 bậc, số cách nhảy là f(n-2);
> c. Từ giả thiết a, b suy ra tổng số cách nhảy là: f(n) = f(n-1) + f(n-2);
> d. Từ trường hợp thực tế suy ra: với 1 bậc thì f(1) = 1, với 2 bậc thì có f(2) = 2.

Phân tích bằng cách tìm quy luật:

> f(1) = 1, f(2) = 2, f(3) = 3, f(4) = 5, có thể tổng kết thành quy luật f(n) = f(n-1) + f(n-2). Nhưng tại sao lại có quy luật này? Giả sử hiện có 6 bậc, ta có thể từ bậc 5 nhảy một bước đến bậc 6. Có bao nhiêu cách nhảy đến bậc 5 thì có bấy nhiêu cách nhảy đến bậc 6. Ngoài ra ta cũng có thể từ bậc 4 nhảy hai bước đến bậc 6. Có bao nhiêu cách nhảy đến bậc 4 thì có bấy nhiêu cách nhảy đến bậc 6. Các trường hợp khác không thể từ bậc 3 nhảy đến bậc 6, v.v. Vì vậy cuối cùng là f(6) = f(5) + f(4); như vậy cũng dễ hiểu bài toán bậc thang biến thể.

**Vì vậy bài này thực chất là bài toán về dãy Fibonacci.**

Chỉ cần sửa một chút code của bài trước. Điểm khác biệt duy nhất là các phần tử ban đầu của bài này là 1 2 3 5 8……, còn bài trước là 1 1 2 3 5 ……。 Bài này cũng có thể dùng đệ quy, nhưng hiệu suất đệ quy quá thấp nên ở đây chỉ đưa ra code dùng phương pháp lặp.

**Code ví dụ:**

```java
int jumpFloor(int number) {
    if (number <= 0) {
        return 0;
    }
    if (number == 1) {
        return 1;
    }
    if (number == 2) {
        return 2;
    }
    int first = 1, second = 2, third = 0;
    for (int i = 3; i <= number; i++) {
        third = first + second;
        first = second;
        second = third;
    }
    return third;
}
```

## Bài toán bậc thang biến thể

**Mô tả bài toán:**

Một con ếch mỗi lần có thể nhảy lên 1 bậc, cũng có thể nhảy lên 2 bậc…… Nó cũng có thể nhảy lên n bậc. Hỏi con ếch đó có tổng cộng bao nhiêu cách để nhảy lên một cầu thang n bậc.

**Phân tích vấn đề:**

Giả sử n>=2, bước đầu tiên có n cách nhảy: nhảy 1 bậc, nhảy 2 bậc, cho đến nhảy n bậc.
Nhảy 1 bậc, còn n-1 bậc, số cách nhảy còn lại là f(n-1).
Nhảy 2 bậc, còn n-2 bậc, số cách nhảy còn lại là f(n-2).
……
Nhảy n-1 bậc, còn 1 bậc, số cách nhảy còn lại là f(1).
Nhảy n bậc, còn 0 bậc, số cách nhảy còn lại là f(0).
Vì vậy khi n>=2:
f(n)=f(n-1)+f(n-2)+...+f(1)
Vì f(n-1)=f(n-2)+f(n-3)+...+f(1)
Nên f(n)=2\*f(n-1). Lại có f(1)=1, suy ra **f(n)=2^(n-1)**.

**Code ví dụ:**

```java
int JumpFloorII(int number) {
    return 1 << --number;//2^(number-1) dùng phép dịch bit, nhanh hơn
}
```

**Bổ sung:**

Java có ba toán tử dịch bit:

1. `"<<"`: **toán tử dịch trái**, tương đương với lũy thừa của 2 với số mũ n
2. `">>"`: **toán tử dịch phải**, tương đương với chia cho lũy thừa của 2 với số mũ n
3. `">>>"`: **toán tử dịch phải không dấu**. Bất kể bit cao nhất trước khi dịch là 0 hay 1, phần trống phát sinh ở bên trái sau khi dịch đều được điền bằng 0. Tương tự `>>`.

```java
int a = 16;
int b = a << 2;//dịch trái 2 bit, tương đương với 16 * 2 lũy thừa 2, tức là 16 * 4
int c = a >> 2;//dịch phải 2 bit, tương đương với 16 / 2 lũy thừa 2, tức là 16 / 4
```

## Tìm kiếm trong mảng hai chiều

**Mô tả bài toán:**

Trong một mảng hai chiều, mỗi hàng được sắp xếp theo thứ tự tăng dần từ trái sang phải, mỗi cột được sắp xếp theo thứ tự tăng dần từ trên xuống dưới. Hãy hoàn thành một function nhận vào mảng hai chiều như vậy và một số nguyên, rồi xác định mảng có chứa số nguyên đó hay không.

**Phân tích vấn đề:**

Bài này tương đối đơn giản. Điều cần cân nhắc là cách làm có hiệu suất cao nhất. Có một cách khá dễ hiểu như sau:

> Ma trận có thứ tự. Nhìn từ góc dưới bên trái, số giảm dần khi đi lên và tăng dần khi đi sang phải.
> Vì vậy bắt đầu tìm từ góc dưới bên trái. Khi số cần tìm lớn hơn số ở góc dưới bên trái thì dịch sang phải.
> Khi số cần tìm nhỏ hơn số ở góc dưới bên trái thì dịch lên trên. Cách tìm này nhanh nhất.

**Code ví dụ:**

```java
public boolean Find(int target, int [][] array) {
    //Ý tưởng cơ bản là bắt đầu tìm từ góc dưới bên trái, cách này nhanh nhất
    int row = array.length-1;//hàng
    int column = 0;//cột
    //Điều kiện lặp đúng khi số hàng lớn hơn 0 và số cột hiện tại nhỏ hơn tổng số cột
    while((row >= 0)&& (column< array[0].length)){
        if(array[row][column] > target){
            row--;
        }else if(array[row][column] < target){
            column++;
        }else{
            return true;
        }
    }
    return false;
}
```

## Thay thế khoảng trắng

**Mô tả bài toán:**

Hãy cài đặt một function thay thế khoảng trắng trong một chuỗi bằng `"%20"`. Ví dụ, khi chuỗi là `We Are Happy.` thì sau khi thay thế sẽ trở thành `We%20Are%20Happy`.

**Phân tích vấn đề:**

Bài này không khó. Có thể lặp để kiểm tra ký tự trong chuỗi có phải khoảng trắng hay không. Nếu có thì dùng phương thức `append()` để nối thêm `"%20"`, nếu không thì nối ký tự ban đầu.

Ngoài ra có thể dùng trực tiếp `String.replace()` để thay thế khoảng trắng theo nghĩa đen, giải quyết bằng một dòng code.

**Code ví dụ:**

Cách làm thông thường:

```java
public String replaceSpace(StringBuffer str) {
    StringBuffer out = new StringBuffer();
    for (int i = 0; i < str.toString().length(); i++) {
        char b = str.charAt(i);
        if(String.valueOf(b).equals(" ")){
            out.append("%20");
        }else{
            out.append(b);
        }
    }
    return out.toString();
}
```

Giải quyết bằng một dòng code:

```java
public String replaceSpace(StringBuffer str) {
    return str.toString().replace(" ", "%20");
}
```

## Lũy thừa nguyên của một giá trị

**Mô tả bài toán:**

Cho một số thực kiểu `double` là `base` và một số nguyên kiểu `int` là `exponent`, hãy tính lũy thừa `exponent` của `base`.

**Phân tích vấn đề:**

Bài này có thể dùng **lũy thừa nhanh**. Cần xử lý hai biên: không thể lấy nghịch đảo khi cơ số bằng 0 và số mũ âm; `Integer.MIN_VALUE` sẽ bị tràn khi lấy số đối, vì vậy trước hết phải chuyển số mũ sang `long`.

Với điều kiện nghiệp vụ “có chính xác bằng 0 hay không”, có thể trực tiếp dùng `base == 0.0`. So sánh bằng epsilon sẽ khiến một cơ số rất nhỏ nhưng khác 0 bị nhận nhầm là 0.

Lũy thừa nhanh giảm một nửa số mũ trong mỗi vòng: khi bit hiện tại của số mũ là 1 thì nhân cơ số hiện tại vào kết quả; sau đó bình phương cơ số và dịch phải số mũ một bit. Độ phức tạp thời gian là O(logn).

**Độ phức tạp thời gian**: O(logn)

**Code ví dụ:**

```java
public class Solution {
    public double Power(double base, int exponent) {
        if (base == 0.0 && exponent < 0) {
            throw new ArithmeticException("zero cannot be raised to a negative exponent");
        }

        long exp = exponent;
        if (exp < 0) {
            base = 1.0 / base;
            exp = -exp;
        }

        double result = 1.0;
        while (exp > 0) {
            if ((exp & 1L) != 0) {
                result *= base;
            }
            base *= base;
            exp >>= 1;
        }
        return result;
    }
}
```

Tất nhiên bài này cũng có thể dùng cách đơn giản là nhân dồn. Nhưng độ phức tạp thời gian của cách này là O(n), nên không hiệu quả bằng cách trước.

```java
// Dùng phép nhân dồn
public double powerAnother(double base, int exponent) {
    if (base == 0.0 && exponent < 0) {
        throw new ArithmeticException("zero cannot be raised to a negative exponent");
    }
    long exp = exponent;
    if (exp < 0) {
        exp = -exp;
    }
    double result = 1.0;
    for (long i = 0; i < exp; i++) {
        result *= base;
    }
    if (exponent >= 0) {
        return result;
    }
    return 1.0 / result;
}
```

## Sắp xếp lại mảng để số lẻ đứng trước số chẵn

**Mô tả bài toán:**

Nhập một mảng số nguyên, cài đặt một function điều chỉnh thứ tự các số trong mảng sao cho tất cả số lẻ nằm ở nửa trước của mảng, tất cả số chẵn nằm ở nửa sau của mảng, đồng thời đảm bảo thứ tự tương đối giữa các số lẻ và giữa các số chẵn không thay đổi.

**Phân tích vấn đề:**

Bài này có khá nhiều cách giải. Dưới đây là một cách mà tôi thấy tương đối dễ hiểu:
Trước hết đếm số lượng số lẻ, giả sử là n, sau đó tạo một mảng mới có cùng độ dài. Tiếp theo lặp để kiểm tra phần tử trong mảng ban đầu là số chẵn hay số lẻ. Nếu là số lẻ thì bắt đầu từ phần tử có chỉ số 0 của mảng mới và thêm số lẻ đó vào; nếu là số chẵn thì bắt đầu từ chỉ số n của mảng mới và thêm số chẵn đó vào.

**Code ví dụ:**

Thuật toán có độ phức tạp thời gian O(n), độ phức tạp không gian O(n)

```java
public class Solution {
    public void reOrderArray(int [] array) {
        //Nếu độ dài mảng bằng 0 hoặc bằng 1 thì không làm gì, trả về ngay
        if(array.length==0||array.length==1)
            return;
        //oddCount: lưu số lượng số lẻ
        //oddBegin: bắt đầu thêm số lẻ từ đầu mảng
        int oddCount=0,oddBegin=0;
        //Tạo một mảng mới
        int[] newArray=new int[array.length];
        //Tính số lượng số lẻ trong mảng rồi bắt đầu thêm phần tử
        for(int i=0;i<array.length;i++){
            if((array[i]&1)==1) oddCount++;
        }
        for(int i=0;i<array.length;i++){
            //Nếu là số lẻ thì thêm phần tử từ đầu mảng mới
            //Nếu là số chẵn thì bắt đầu thêm phần tử từ oddCount (số lượng số lẻ trong mảng)
            if((array[i]&1)==1)
                newArray[oddBegin++]=array[i];
            else newArray[oddCount++]=array[i];
        }
        for(int i=0;i<array.length;i++){
            array[i]=newArray[i];
        }
    }
}
```

## Node thứ k tính từ cuối trong danh sách liên kết

**Mô tả bài toán:**

Nhập một danh sách liên kết, xuất node thứ k tính từ cuối trong danh sách đó.

**Phân tích vấn đề:**

**Tóm tắt trong một câu:**
Một trong hai pointer là p1 chạy trước. Sau khi p1 chạy đến node thứ k-1 thì pointer p2 bắt đầu chạy. Khi p1 chạy đến cuối, node mà p2 trỏ tới chính là node thứ k tính từ cuối.

**Hiểu đơn giản về ý tưởng:**

Giả sử trước hết số node của danh sách liên kết (độ dài) là n.
Quy luật một: muốn tìm node thứ k tính từ cuối thì cần tiến về trước bao nhiêu bước? Ví dụ node thứ nhất tính từ cuối cần đi n bước, vậy node thứ hai tính từ cuối thì sao? Rõ ràng cần đi n-1 bước. Vì vậy có thể rút ra quy luật: để tìm node thứ k tính từ cuối, cần đi về trước n-k+1 bước.

**Bắt đầu thuật toán:**

1. Đặt hai pointer p1 và p2 cùng trỏ vào head. Khi p1 đi k-1 bước thì dừng lại. Trước đó p2 không di chuyển.
2. Bước tiếp theo của p1 là bước thứ k, lúc này p2 bắt đầu di chuyển cùng. Vì sao p2 di chuyển tại thời điểm này? Hãy xem phân tích bên dưới.
3. Khi p1 đi đến cuối danh sách liên kết, tức là p1 đã đi n bước. Vì p2 bắt đầu di chuyển sau khi p1 đi k-1 bước, nên p1 và p2 luôn cách nhau k-1 bước. Vì vậy khi p1 đi n bước, p2 phải đi được n-(k-1) bước. Tức p2 đã đi n-k+1 bước. Lúc này p2 vừa khéo trỏ đúng vào node thứ k tính từ cuối theo quy luật một.
   Như vậy có dễ hiểu hơn không?

**Nội dung kiểm tra:**

Danh sách liên kết + tính robust của code

**Code ví dụ:**

```java
/*
//Lớp danh sách liên kết
public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}*/

//Độ phức tạp thời gian O(n), chỉ cần duyệt một lần
public class Solution {
    public ListNode FindKthToTail(ListNode head,int k) {
        ListNode pre=null,p=null;
        //Hai pointer cùng trỏ vào node đầu
        p=head;
        pre=head;
        //Ghi lại giá trị k
        int a=k;
        //Ghi lại số lượng node
        int count=0;
        //Pointer p chạy trước và ghi lại số node. Sau khi p chạy k-1 node thì pointer pre bắt đầu chạy,
        //khi pointer p chạy đến cuối, pointer mà pre trỏ tới chính là node thứ k tính từ cuối
        while(p!=null){
            p=p.next;
            count++;
            if(k<1){
                pre=pre.next;
            }
            k--;
        }
        //Nếu số node nhỏ hơn node thứ k tính từ cuối cần tìm thì trả về null
        if(count<a) return null;
        return pre;

    }
}
```

## Đảo ngược danh sách liên kết

**Mô tả bài toán:**

Nhập một danh sách liên kết, đảo ngược danh sách rồi xuất tất cả phần tử của danh sách.

**Phân tích vấn đề:**

Đây là một bài rất thường gặp về danh sách liên kết. Ý tưởng không khó, nhưng khi tự cài đặt có thể thật sự cảm thấy không biết bắt đầu từ đâu. Tôi tham khảo code của người khác.
Ý tưởng là dựa vào đặc điểm node trước trỏ tới node sau của danh sách liên kết, đưa các node phía sau lên phía trước.
Ví dụ trong hình dưới đây: đổi vị trí node 1 và node 2, sau đó cho node 3 trỏ tới node 2, node 4 trỏ tới node 3. Như vậy danh sách liên kết bên dưới đã được đảo ngược.

![Quá trình thay đổi liên kết giữa các node liền kề khi đảo ngược danh sách liên kết](https://oss.javaguide.cn/p3-juejin/844773c7300e4373922bb1a6ae2a55a3~tplv-k3u1fbpfcp-zoom-1.png)

**Nội dung kiểm tra:**

Danh sách liên kết + tính robust của code

**Code ví dụ:**

```java
/*
public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}*/
public class Solution {
    public ListNode ReverseList(ListNode head) {
       ListNode next = null;
       ListNode pre = null;
        while (head != null) {
              //Lưu node cần đảo lên đầu
               next = head.next;
               //Cho node cần đảo trỏ tới node trước đó đã được đảo
               head.next = pre;
               //Node trước đó đã được đảo lên đầu
               pre = head;
               //Liên tục đi về phía cuối danh sách liên kết
               head = next;
        }
        return pre;
    }
}
```

## Hợp nhất hai danh sách liên kết đã sắp xếp

**Mô tả bài toán:**

Nhập hai danh sách liên kết tăng dần, xuất danh sách liên kết sau khi hợp nhất hai danh sách. Tất nhiên danh sách sau khi hợp nhất phải thỏa mãn quy tắc không giảm.

**Phân tích vấn đề:**

Có thể phân tích như sau:

1. Giả sử có hai danh sách liên kết A, B;
2. So sánh giá trị của node đầu A1 của A với node đầu B1 của B. Giả sử A1 nhỏ hơn thì A1 là node đầu;
3. So sánh A2 với B1. Giả sử B1 nhỏ hơn thì A1 trỏ tới B1;
4. So sánh A2 với B2……
   Cứ lặp như vậy là được, khá dễ hiểu.

**Nội dung kiểm tra:**

Danh sách liên kết + tính robust của code

**Code ví dụ:**

Phiên bản không đệ quy:

```java
/*
public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}*/
public class Solution {
    public ListNode Merge(ListNode list1,ListNode list2) {
       //list1 rỗng thì trả về list2 ngay
       if(list1 == null){
            return list2;
        }
        //list2 rỗng thì trả về list1 ngay
        if(list2 == null){
            return list1;
        }
        ListNode mergeHead = null;
        ListNode current = null;
        //Khi list1 và list2 đều không rỗng
        while(list1!=null && list2!=null){
            //Chọn giá trị nhỏ hơn làm node đầu
            if(list1.val <= list2.val){
                if(mergeHead == null){
                   mergeHead = current = list1;
                }else{
                   current.next = list1;
                    //current lưu giá trị của node list1 vì lần sau còn dùng
                   current = list1;
                }
                //list1 trỏ tới node tiếp theo
                list1 = list1.next;
            }else{
                if(mergeHead == null){
                   mergeHead = current = list2;
                }else{
                   current.next = list2;
                     //current lưu giá trị của node list2 vì lần sau còn dùng
                   current = list2;
                }
                //list2 trỏ tới node tiếp theo
                list2 = list2.next;
            }
        }
        if(list1 == null){
            current.next = list2;
        }else{
            current.next = list1;
        }
        return mergeHead;
    }
}
```

Phiên bản đệ quy:

```java
public ListNode Merge(ListNode list1,ListNode list2) {
    if(list1 == null){
        return list2;
    }
    if(list2 == null){
        return list1;
    }
    if(list1.val <= list2.val){
        list1.next = Merge(list1.next, list2);
        return list1;
    }else{
        list2.next = Merge(list1, list2.next);
        return list2;
    }
}
```

## Dùng hai stack để cài đặt queue

**Mô tả bài toán:**

Dùng hai stack để cài đặt một queue, hoàn thành thao tác Push và Pop của queue. Các phần tử trong queue có kiểu int.

**Phân tích vấn đề:**

Trước hết ôn lại đặc điểm cơ bản của stack và queue:
**Stack:** vào sau ra trước (LIFO)
**Queue:** vào trước ra trước
Rõ ràng cần dựa vào một số phương thức cơ bản của stack mà JDK cung cấp để cài đặt. Hãy xem một số phương thức cơ bản của lớp Stack:

![Một số phương thức thường gặp của lớp Stack](https://oss.javaguide.cn/github/javaguide/cs-basics/algorithms/5985000.jpg)

Vì đề bài cho hai stack, có thể xử lý như sau: khi push thì push phần tử vào stack1; khi pop thì trước hết pop các phần tử của stack1 sang stack2, sau đó thực hiện thao tác pop trên stack2. Như vậy có thể đảm bảo thứ tự vào trước ra trước. (Âm [pop] âm [pop] thành dương [vào trước ra trước])

**Nội dung kiểm tra:**

Queue + stack

**Code ví dụ:**

```java
//Đáp án trong cuốn 《Hướng dẫn phỏng vấn code dành cho lập trình viên》 của Zuo Chengyun
import java.util.Stack;

public class Solution {
    Stack<Integer> stack1 = new Stack<Integer>();
    Stack<Integer> stack2 = new Stack<Integer>();

    //Khi thực hiện thao tác push, thêm phần tử vào stack1
    public void push(int node) {
        stack1.push(node);
    }

    public int pop() {
        //Nếu cả hai queue đều rỗng thì ném exception, nghĩa là người dùng chưa push phần tử nào
        if(stack1.empty()&&stack2.empty()){
            throw new RuntimeException("Queue is empty!");
        }
        //Nếu stack2 không rỗng thì thực hiện trực tiếp thao tác pop trên stack2
        if(stack2.empty()){
            while(!stack1.empty()){
                //Push các phần tử của stack1 vào stack2 theo thứ tự vào sau ra trước
                stack2.push(stack1.pop());
            }
        }
          return stack2.pop();
    }
}
```

## Chuỗi push và pop của stack

**Mô tả bài toán:**

Nhập hai chuỗi số nguyên. Chuỗi thứ nhất biểu thị thứ tự push vào stack, hãy xác định chuỗi thứ hai có phải là thứ tự pop của stack đó hay không. Giả sử tất cả số được push vào stack đều khác nhau. Ví dụ chuỗi 1,2,3,4,5 là thứ tự push của một stack, chuỗi 4,5,3,2,1 là một chuỗi pop tương ứng với chuỗi push đó, nhưng 4,3,5,1,2 không thể là chuỗi pop của chuỗi push đó. (Lưu ý: độ dài của hai chuỗi bằng nhau.)

**Phân tích bài toán:**

Tôi đã suy nghĩ khá lâu mà không có ý tưởng, sau đó tham khảo [đáp án của Alias](https://www.nowcoder.com/questionTerminal/d77d11405cc7470d82554cb392585106). Cách làm của bạn ấy cũng được viết rất chi tiết và khá dễ hiểu.

【Ý tưởng】Dùng một stack phụ, duyệt thứ tự push, trước tiên đưa phần tử đầu tiên vào stack, ở đây là 1, sau đó kiểm tra phần tử trên cùng của stack có phải phần tử đầu tiên trong thứ tự pop hay không, ở đây là 4. Rõ ràng 1≠4, nên tiếp tục push cho đến khi bằng nhau thì bắt đầu pop. Sau khi pop một phần tử thì dịch chuỗi pop về sau một vị trí, tiếp tục cho đến khi không bằng nhau. Lặp như vậy cho đến khi duyệt xong thứ tự push. Nếu stack phụ vẫn chưa rỗng thì chứng tỏ chuỗi pop không phải thứ tự pop của stack đó.

Ví dụ:

Push 1,2,3,4,5

Pop 4,5,3,2,1

Đầu tiên push 1 vào stack phụ, lúc này phần tử trên cùng là 1≠4, tiếp tục push 2.

Lúc này phần tử trên cùng là 2≠4, tiếp tục push 3.

Lúc này phần tử trên cùng là 3≠4, tiếp tục push 4.

Lúc này phần tử trên cùng là 4=4, pop 4, chuỗi pop dịch về sau một vị trí, lúc này là 5, stack phụ chứa 1,2,3.

Lúc này phần tử trên cùng là 3≠5, tiếp tục push 5.

Lúc này phần tử trên cùng là 5=5, pop 5, chuỗi pop dịch về sau một vị trí, lúc này là 3, stack phụ chứa 1,2,3.

……

Thực hiện lần lượt, cuối cùng stack phụ rỗng. Nếu không rỗng thì chứng tỏ chuỗi pop không phải thứ tự pop của stack đó.

**Nội dung kiểm tra:**

Stack

**Code ví dụ:**

```java
import java.util.ArrayList;
import java.util.Stack;
//Không nghĩ ra bài này, tham khảo đáp án của bạn Alias: https://www.nowcoder.com/questionTerminal/d77d11405cc7470d82554cb392585106
public class Solution {
    public boolean IsPopOrder(int [] pushA,int [] popA) {
        if(pushA.length == 0 || popA.length == 0)
            return false;
        Stack<Integer> s = new Stack<Integer>();
        //Dùng để đánh dấu vị trí trong chuỗi pop
        int popIndex = 0;
        for(int i = 0; i< pushA.length;i++){
            s.push(pushA[i]);
            //Nếu stack không rỗng và phần tử trên cùng bằng phần tử trong chuỗi pop
            while(!s.empty() &&s.peek() == popA[popIndex]){
                //Pop khỏi stack
                s.pop();
                //Dịch chuỗi pop về sau một vị trí
                popIndex++;
            }
        }
        return s.empty();
    }
}
```

<!-- @include: @article-footer.snippet.md -->
