---
title: Một số bài toán thuật toán chuỗi thường gặp
description: Tổng hợp các thuật toán và dạng bài chuỗi thường gặp, tập trung giải thích nguyên lý KMP/BM, kỹ thuật cửa sổ trượt và giúp người đọc hiểu matching hiệu quả và cách triển khai.
category: Computer Fundamentals
tag:
  - Algorithms
head:
  - - meta
    - name: keywords
      content: thuật toán chuỗi,KMP,BM,cửa sổ trượt,chuỗi con,matching,độ phức tạp
---

> Tác giả: wwwxmu
>
> Địa chỉ bài viết gốc: <https://www.weiweiblog.cn/13string/>

## 1. Thuật toán KMP

Khi nói đến các bài toán chuỗi, không thể không nhắc đến thuật toán KMP. Thuật toán này dùng để giải quyết bài toán tìm kiếm chuỗi, tức là tìm vị trí xuất hiện của một chuỗi con W trong một chuỗi S. KMP giảm độ phức tạp thời gian của việc matching ký tự xuống O(m+n), còn độ phức tạp không gian chỉ là O(m). Phương pháp “tìm kiếm brute force” liên tục quay lui trong chuỗi chính, dẫn đến hiệu suất thấp. Trong khi đó, KMP tận dụng thông tin đã matching một phần, giữ con trỏ trên chuỗi chính không quay lui, đồng thời điều chỉnh con trỏ của chuỗi con để dịch chuỗi mẫu đến vị trí phù hợp nhất.

Chi tiết thuật toán, tham khảo:

- [Hiểu toàn diện KMP từ đầu đến cuối](https://blog.csdn.net/v_july_v/article/details/7041827)
- [Làm thế nào để hiểu và nắm vững thuật toán KMP tốt hơn?](https://www.zhihu.com/question/21923021)
- [Phân tích chi tiết thuật toán KMP](https://blog.sengxian.com/algorithms/kmp)
- [Minh họa thuật toán KMP](http://blog.jobbole.com/76611/)
- [Thuật toán matching chuỗi KMP mà ai cũng có thể hiểu 【phụ đề song ngữ】](https://www.bilibili.com/video/av3246487/?from=search&seid=17173603269940723925)
- [Thuật toán matching chuỗi KMP 1](https://www.bilibili.com/video/av11866460?from=search&seid=12730654434238709250)

**Ngoài ra, hãy tìm hiểu thêm thuật toán BM!**

> BM cũng là một thuật toán matching chuỗi chính xác. Nó so sánh từ phải sang trái, đồng thời áp dụng hai quy tắc heuristic là quy tắc ký tự xấu và quy tắc hậu tố tốt để quyết định khoảng cách dịch sang phải. Ý tưởng cơ bản là matching ký tự từ phải sang trái. Khi gặp ký tự không khớp, thuật toán lấy khoảng cách dịch phải lớn nhất từ bảng ký tự xấu và bảng hậu tố tốt, rồi dịch chuỗi mẫu sang phải để tiếp tục matching.
> Thuật toán KMP trong matching chuỗi: <http://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html>

## 2. Thay thế khoảng trắng

> Jianzhi Offer: Hãy triển khai một hàm thay thế mọi khoảng trắng trong một chuỗi bằng "%20". Ví dụ, `We Are Happy.` sau khi thay thế sẽ thành `We%20Are%20Happy`.

Có hai phương pháp: ① phương pháp thông thường; ② sử dụng API.

```java
//https://www.weiweiblog.cn/replacespace/
public class Solution {

  /**
   * Method 1: phương pháp thông thường. Duyệt chuỗi bằng String.charAt(i) và
   * String.valueOf(char).equals(" "), đồng thời kiểm tra phần tử có phải khoảng trắng hay không.
   * Nếu có thì thay bằng "%20", nếu không thì giữ nguyên.
   */
  public static String replaceSpace(StringBuffer str) {

    int length = str.length();
    // System.out.println("length=" + length);
    StringBuffer result = new StringBuffer();
    for (int i = 0; i < length; i++) {
      char b = str.charAt(i);
      if (String.valueOf(b).equals(" ")) {
        result.append("%20");
      } else {
        result.append(b);
      }
    }
    return result.toString();

  }

  /**
   * Method 2: sử dụng API để thay thế toàn bộ khoảng trắng, giải quyết bài toán bằng một dòng code.
   */
  public static String replaceSpace2(StringBuffer str) {

    return str.toString().replace(" ", "%20");
  }
}

```

Với trường hợp thay thế ký tự cố định, chẳng hạn khoảng trắng, phương pháp thứ hai có thể dùng `replace` để đạt hiệu suất tốt hơn!

```java
str.toString().replace(" ","%20");
```

## 3. Tiền tố chung dài nhất

> LeetCode: Viết một hàm tìm tiền tố chung dài nhất trong một mảng chuỗi. Nếu không tồn tại tiền tố chung, trả về chuỗi rỗng "".

Ví dụ 1:

```plain
Input: ["flower","flow","flight"]
Output: "fl"
```

Ví dụ 2:

```plain
Input: ["dog","racecar","car"]
Output: ""
Explanation: Input không có tiền tố chung.
```

Ý tưởng rất đơn giản! Trước tiên dùng `Arrays.sort(strs)` để sắp xếp mảng, sau đó lần lượt so sánh ký tự của phần tử đầu tiên và phần tử cuối cùng trong mảng từ trái sang phải!

```java
public class Main {
 public static String replaceSpace(String[] strs) {

  // Nếu giá trị kiểm tra không hợp lệ thì trả về chuỗi rỗng.
  if (!checkStrs(strs)) {
   return "";
  }
  // Độ dài mảng.
  int len = strs.length;
  // Dùng để lưu kết quả.
  StringBuilder res = new StringBuilder();
  // Sắp xếp các phần tử của mảng chuỗi theo thứ tự tăng dần (nếu có số thì số sẽ đứng trước).
  Arrays.sort(strs);
  int m = strs[0].length();
  int n = strs[len - 1].length();
  int num = Math.min(m, n);
  for (int i = 0; i < num; i++) {
   if (strs[0].charAt(i) == strs[len - 1].charAt(i)) {
    res.append(strs[0].charAt(i));
   } else
    break;

  }
  return res.toString();

 }

 private static boolean checkStrs(String[] strs) {
  boolean flag = false;
  if (strs != null) {
   // Duyệt strs để kiểm tra giá trị phần tử.
   for (int i = 0; i < strs.length; i++) {
    if (strs[i] != null && strs[i].length() != 0) {
     flag = true;
    } else {
     flag = false;
     break;
    }
   }
  }
  return flag;
 }

 // Test.
 public static void main(String[] args) {
  String[] strs = { "customer", "car", "cat" };
  // String[] strs = { "customer", "car", null };// Chuỗi rỗng.
  // String[] strs = {};// Chuỗi rỗng.
  // String[] strs = null;// Chuỗi rỗng.
  System.out.println(Main.replaceSpace(strs));// c
 }
}

```

## 4. Chuỗi palindrome

### 4.1. Palindrome dài nhất

> LeetCode: Cho một chuỗi gồm các chữ cái viết hoa và viết thường, hãy tìm palindrome dài nhất có thể tạo thành từ các chữ cái này. Trong quá trình tạo, cần phân biệt chữ hoa và chữ thường. Ví dụ, `"Aa"` không thể được xem là một chuỗi palindrome. Lưu ý: giả sử độ dài chuỗi không vượt quá 1010.
>
> Palindrome: “Chuỗi palindrome” là chuỗi đọc xuôi hay đọc ngược đều giống nhau, chẳng hạn "level" hoặc "noon". — Bách khoa Baidu, địa chỉ: <https://baike.baidu.com/item/%E5%9B%9E%E6%96%87%E4%B8%B2/1274921?fr=aladdin>

Ví dụ 1:

```plain
Input:
"abccccdd"

Output:
7

Explanation:
Palindrome dài nhất có thể tạo thành là "dccaccd", có độ dài 7.
```

Đã biết palindrome là gì, bây giờ xét hai trường hợp có thể tạo thành palindrome:

- Tổ hợp trong đó số lần xuất hiện của ký tự là số chẵn.
- **Tổ hợp trong đó số lần xuất hiện của ký tự là số chẵn + một ký tự xuất hiện nhiều nhất trong các ký tự có số lần xuất hiện lẻ** (tham khảo **[issue665](https://github.com/Snailclimb/JavaGuide/issues/665)**).

Chỉ cần thống kê số lần xuất hiện của ký tự, vì chỉ các cặp mới có thể tạo thành palindrome. Do cho phép một ký tự xuất hiện riêng ở giữa, chẳng hạn "abcba", nên nếu cuối cùng còn một chữ cái lẻ thì có thể cộng thêm 1 vào tổng độ dài. Trước tiên chuyển chuỗi thành mảng ký tự. Sau đó duyệt mảng, kiểm tra ký tự tương ứng có trong hashset hay không; nếu không thì thêm vào, nếu có thì tăng `count++`, rồi xóa ký tự đó! Nhờ vậy có thể đếm số cặp ký tự.

```java
//https://leetcode-cn.com/problems/longest-palindrome/description/
class Solution {
  public  int longestPalindrome(String s) {
    if (s.length() == 0)
      return 0;
    // Dùng để lưu ký tự.
    HashSet<Character> hashset = new HashSet<Character>();
    char[] chars = s.toCharArray();
    int count = 0;
    for (int i = 0; i < chars.length; i++) {
      if (!hashset.contains(chars[i])) {// Nếu hashset chưa có ký tự thì lưu ký tự vào.
        hashset.add(chars[i]);
      } else {// Nếu đã có thì tăng count++ (đã tìm được một cặp ký tự), sau đó xóa ký tự đó.
        hashset.remove(chars[i]);
        count++;
      }
    }
    return hashset.isEmpty() ? count * 2 : count * 2 + 1;
  }
}
```

### 4.2. Kiểm tra palindrome

> LeetCode: Cho một chuỗi, hãy kiểm tra xem chuỗi đó có phải palindrome hay không; chỉ xét các ký tự chữ cái và chữ số, không phân biệt chữ hoa và chữ thường. Giải thích: trong bài này, chuỗi rỗng được định nghĩa là một palindrome hợp lệ.

Ví dụ 1:

```plain
Input: "A man, a plan, a canal: Panama"
Output: true
```

Ví dụ 2:

```plain
Input: "race a car"
Output: false
```

```java
//https://leetcode-cn.com/problems/valid-palindrome/description/
class Solution {
  public  boolean isPalindrome(String s) {
    if (s.length() == 0)
      return true;
    int l = 0, r = s.length() - 1;
    while (l < r) {
      // Duyệt từ đầu và cuối về phía giữa.
      if (!Character.isLetterOrDigit(s.charAt(l))) {// Trường hợp ký tự không phải chữ cái hoặc chữ số.
        l++;
      } else if (!Character.isLetterOrDigit(s.charAt(r))) {// Trường hợp ký tự không phải chữ cái hoặc chữ số.
        r--;
      } else {
        // Kiểm tra hai ký tự có bằng nhau hay không.
        if (Character.toLowerCase(s.charAt(l)) != Character.toLowerCase(s.charAt(r)))
          return false;
        l++;
        r--;
      }
    }
    return true;
  }
}
```

### 4.3. Chuỗi con palindrome dài nhất

> LeetCode: Chuỗi con palindrome dài nhất. Cho một chuỗi s, hãy tìm chuỗi con palindrome dài nhất trong s. Có thể giả sử độ dài tối đa của s là 1000.

Ví dụ 1:

```plain
Input: "babad"
Output: "bab"
Note: "aba" cũng là một đáp án hợp lệ.
```

Ví dụ 2:

```plain
Input: "cbbd"
Output: "bb"
```

Lấy một phần tử làm tâm, lần lượt tính độ dài palindrome lớn nhất cho trường hợp độ dài chẵn và lẻ.

```java
//https://leetcode-cn.com/problems/longest-palindromic-substring/description/
class Solution {
  private int index, len;

  public String longestPalindrome(String s) {
    if (s.length() < 2)
      return s;
    for (int i = 0; i < s.length() - 1; i++) {
      PalindromeHelper(s, i, i);
      PalindromeHelper(s, i, i + 1);
    }
    return s.substring(index, index + len);
  }

  public void PalindromeHelper(String s, int l, int r) {
    while (l >= 0 && r < s.length() && s.charAt(l) == s.charAt(r)) {
      l--;
      r++;
    }
    if (len < r - l - 1) {
      index = l + 1;
      len = r - l - 1;
    }
  }
}
```

### 4.4. Dãy con palindrome dài nhất

> LeetCode: Dãy con palindrome dài nhất.
> Cho một chuỗi s, hãy tìm dãy con palindrome dài nhất trong đó. Có thể giả sử độ dài tối đa của s là 1000.
> **Điểm khác nhau giữa dãy con palindrome dài nhất ở bài này và chuỗi con palindrome dài nhất ở bài trước là: chuỗi con là một dãy liên tiếp trong chuỗi, còn dãy con là một dãy ký tự giữ nguyên vị trí tương đối trong chuỗi. Ví dụ, "bbbb" có thể là dãy con của chuỗi "bbbab", nhưng không phải chuỗi con.**

Cho một chuỗi s, hãy tìm dãy con palindrome dài nhất trong đó. Có thể giả sử độ dài tối đa của s là 1000.

Ví dụ 1:

```plain
Input:
"bbbab"
Output:
4
```

Một dãy con palindrome dài nhất có thể là "bbbb".

Ví dụ 2:

```plain
Input:
"cbbd"
Output:
2
```

Một dãy con palindrome dài nhất có thể là "bb".

**Quy hoạch động:** `dp[i][j] = dp[i+1][j-1] + 2 if s.charAt(i) == s.charAt(j) otherwise, dp[i][j] = Math.max(dp[i+1][j], dp[i][j-1])`

```java
class Solution {
    public int longestPalindromeSubseq(String s) {
        int len = s.length();
        int [][] dp = new int[len][len];
        for(int i = len - 1; i>=0; i--){
            dp[i][i] = 1;
            for(int j = i+1; j < len; j++){
                if(s.charAt(i) == s.charAt(j))
                    dp[i][j] = dp[i+1][j-1] + 2;
                else
                    dp[i][j] = Math.max(dp[i+1][j], dp[i][j-1]);
            }
        }
        return dp[0][len-1];
    }
}
```

## 5. Độ sâu dãy ngoặc

> iQIYI 2018 Java trong đợt tuyển dụng mùa thu:
> Một dãy ngoặc hợp lệ được định nghĩa như sau:
>
> 1. Chuỗi rỗng "" là một dãy ngoặc hợp lệ.
> 2. Nếu "X" và "Y" đều là các dãy ngoặc hợp lệ thì "XY" cũng là một dãy ngoặc hợp lệ.
> 3. Nếu "X" là một dãy ngoặc hợp lệ thì "(X)" cũng là một dãy ngoặc hợp lệ.
> 4. Mọi dãy ngoặc hợp lệ đều có thể được tạo ra từ các quy tắc trên.
>
> Ví dụ: "", "()", "()()", "((()))" đều là các dãy ngoặc hợp lệ.
> Với một dãy ngoặc hợp lệ, độ sâu của nó được định nghĩa như sau:
>
> 1. Độ sâu của chuỗi rỗng "" là 0.
> 2. Nếu độ sâu của chuỗi "X" là x, độ sâu của chuỗi "Y" là y thì độ sâu của chuỗi "XY" là max(x, y).
> 3. Nếu độ sâu của "X" là x thì độ sâu của chuỗi "(X)" là x+1.
>
> Ví dụ: độ sâu của "()()()" là 1, độ sâu của "((()))" là 3. Cho một dãy ngoặc hợp lệ, hãy tính độ sâu của dãy đó.

```plain
Mô tả input:
Input gồm một dãy ngoặc hợp lệ s, độ dài length của s (2 ≤ length ≤ 50), trong dãy chỉ chứa '(' và ')'.

Mô tả output:
In ra một số nguyên dương, tức độ sâu của dãy.
```

Ví dụ:

```plain
Input:
(())
Output:
2
```

Code:

```java
import java.util.Scanner;

/**
 * https://www.nowcoder.com/test/8246651/summary
 *
 * @author Snailclimb
 * @date 2018-09-06
  * @Description: Tính độ sâu của dãy ngoặc hợp lệ đã cho.
 */
public class Main {
  public static void main(String[] args) {
    Scanner sc = new Scanner(System.in);
    String s = sc.nextLine();
    int cnt = 0, max = 0, i;
    for (i = 0; i < s.length(); ++i) {
      if (s.charAt(i) == '(')
        cnt++;
      else
        cnt--;
      max = Math.max(max, cnt);
    }
    sc.close();
    System.out.println(max);
  }
}

```

## 6. Chuyển chuỗi thành số nguyên

> Jianzhi Offer: Chuyển một chuỗi thành một số nguyên (triển khai chức năng của `Integer.valueOf(string)`, nhưng trả về 0 khi string không đáp ứng yêu cầu về số), yêu cầu không được sử dụng hàm thư viện chuyển chuỗi thành số nguyên. Nếu giá trị là 0 hoặc chuỗi không phải một giá trị hợp lệ thì trả về 0.

```java
//https://www.weiweiblog.cn/strtoint/
public class Main {

  public static int StrToInt(String str) {
    if (str.length() == 0)
      return 0;
    char[] chars = str.toCharArray();
    // Kiểm tra có dấu hay không.
    int flag = 0;
    if (chars[0] == '+')
      flag = 1;
    else if (chars[0] == '-')
      flag = 2;
    int start = flag > 0 ? 1 : 0;
    int res = 0;// Lưu kết quả.
    for (int i = start; i < chars.length; i++) {
      if (Character.isDigit(chars[i])) {// Gọi Character.isDigit(char) để kiểm tra có phải chữ số hay không; trả về True nếu phải, ngược lại trả về False.
        int temp = chars[i] - '0';
        res = res * 10 + temp;
      } else {
        return 0;
      }
    }
   return flag != 2 ? res : -res;

  }

  public static void main(String[] args) {
    String s = "-12312312";
    System.out.println("Chuyển bằng hàm thư viện: " + Integer.valueOf(s));
    int res = Main.StrToInt(s);
    System.out.println("Chuyển bằng phương thức tự triển khai: " + res);

  }

}

```

## Trọng tâm ôn tập phỏng vấn

Các bài toán chuỗi trông có vẻ đa dạng, nhưng thực tế chỉ có một số template thường gặp: đếm bằng hash, hai con trỏ, cửa sổ trượt, KMP, palindrome và mô phỏng bằng stack.

| Dạng bài             | Phương pháp thường dùng         | Bài tiêu biểu                                                            |
| -------------------- | ------------------------------- | ------------------------------------------------------------------------ |
| Đếm ký tự            | Mảng hoặc hash table            | Valid Anagram, Group Anagrams                                            |
| Bài toán chuỗi con   | Cửa sổ trượt                    | Longest Substring Without Repeating Characters, Minimum Window Substring |
| Bài toán palindrome  | Hai con trỏ, mở rộng từ tâm, DP | Valid Palindrome, Longest Palindromic Substring                          |
| Matching chuỗi       | KMP, hash                       | Triển khai `strStr()`                                                    |
| Dấu ngoặc và giải mã | Stack                           | Valid Parentheses, Decode String                                         |
| Chuyển đổi số        | Mô phỏng                        | Chuyển chuỗi thành số nguyên                                             |

Khi xử lý bài toán chuỗi, trước tiên có thể đặt 3 câu hỏi:

1. Bài toán quan tâm đến chuỗi con hay dãy con? Chuỗi con liên tiếp, còn dãy con không yêu cầu liên tiếp.
2. Phạm vi của tập ký tự lớn đến đâu? Khi chỉ có chữ cái viết thường, đếm bằng mảng trực tiếp hơn dùng hash table.
3. Có cần xử lý các trường hợp biên như overflow, chuỗi rỗng, khoảng trắng và dấu hay không?

Một số điểm dễ mắc lỗi:

- Trong Java, `String` là immutable; nếu nối chuỗi thường xuyên thì nên dùng `StringBuilder`.
- Khi xử lý ký tự Unicode, `char` có thể không đủ; phần lớn bài toán thuật toán thông thường chỉ kiểm tra ASCII hoặc chữ cái viết thường.
- Chuỗi con palindrome và dãy con palindrome là hai dạng bài khác nhau; dạng đầu thường dùng mở rộng từ tâm, dạng sau thường dùng DP.
- Trong phỏng vấn KMP, thường không yêu cầu tự suy ra quy trình tính thủ công mảng `next` từ đầu, nhưng cần hiểu tác dụng của nó là bỏ qua prefix đã matching để tránh matching lặp lại.

<!-- @include: @article-footer.snippet.md -->
