---
title: Tổng hợp kiến thức cơ bản về lập trình Shell
description: Lập trình Shell rất hữu ích trong công việc phát triển hằng ngày. Hiện nay, các ngôn ngữ tự động hóa vận hành phổ biến nhất trên hệ thống Linux là Shell và Python. Bài viết này sẽ tổng hợp ngắn gọn kiến thức cơ bản về lập trình Shell, giúp bạn làm quen với lập trình Shell!
category: Computer Basics
tag:
  - Operating System
  - Linux
head:
  - - meta
    - name: keywords
      content: Shell,script,command,automation,operations,Linux,basic syntax
---

Lập trình Shell rất hữu ích trong công việc phát triển hằng ngày. Hiện nay, các ngôn ngữ tự động hóa vận hành phổ biến nhất trên hệ thống Linux là Shell và Python.

Bài viết này sẽ tổng hợp ngắn gọn kiến thức cơ bản về lập trình Shell, giúp bạn làm quen với lập trình Shell!

## Ghi chú về phiên bản

**Các ví dụ trong bài viết áp dụng cho bash phiên bản 4.0+**. bash ở các phiên bản khác nhau có thể có khác biệt ở một số tính năng, đặc biệt là:

- **Array**: bash 2.0+ hỗ trợ, còn POSIX sh thuần (như dash) không hỗ trợ.
- **Một số thao tác với string**: chẳng hạn `${var:offset:length}` có thể không được hỗ trợ ở các phiên bản cũ hơn.
- **Arithmetic expansion `$((...))`**: bash 2.0+ hỗ trợ.

Kiểm tra phiên bản bash của bạn:

```shell
bash --version
# hoặc
echo $BASH_VERSION
```

## Bước vào thế giới lập trình Shell

### Tại sao cần học Shell?

Khi học một thứ gì đó, phần lớn chúng ta đều hướng đến tính thực tiễn. Xét từ góc độ công việc, học Shell nhằm nâng cao hiệu suất làm việc, tăng output và giúp chúng ta hoàn thành nhiều việc hơn trong thời gian ngắn hơn.

Nhiều người cho rằng lập trình Shell thuộc mảng kiến thức vận hành, nên để nhân viên vận hành thực hiện, còn backend developer như chúng ta không cần học. Tôi cho rằng cách nói này hoàn toàn sai. So với những người chuyên làm Linux operations, yêu cầu về mức độ thành thạo lập trình Shell của chúng ta thấp hơn, nhưng lập trình Shell vẫn là thứ chúng ta bắt buộc phải nắm được!

Hiện nay, các ngôn ngữ tự động hóa vận hành phổ biến nhất trên hệ thống Linux là Shell và Python.

Giữa hai ngôn ngữ, Shell gần như là ngôn ngữ lập trình tự động hóa vận hành bắt buộc phải dùng trong các doanh nghiệp IT, đặc biệt không thể thiếu trong các khâu monitoring service, deploy nhanh nghiệp vụ, start/stop service, backup và xử lý dữ liệu, phân tích log trong công việc vận hành. Python phù hợp hơn với việc xử lý logic nghiệp vụ phức tạp, phát triển tool phần mềm vận hành phức tạp và thực hiện truy cập qua web. Shell là một command interpreter, dùng để diễn giải và thực thi các command và program do người dùng nhập. Đây là một cách tương tác đối thoại, trong đó command được phản hồi ngay sau khi nhập.

Ngoài ra, hiểu về lập trình Shell cũng là yêu cầu tuyển dụng backend developer của phần lớn các công ty Internet. Hình dưới đây là một số yêu cầu về lập trình Shell của các công ty Internet nổi tiếng mà tôi đã chụp lại.

![Yêu cầu về kỹ năng lập trình Shell của các công ty Internet lớn](https://oss.javaguide.cn/github/javaguide/cs-basics/shell/60190220.jpg)

### Shell là gì?

**Shell là command interpreter của hệ thống Linux/Unix**, đóng vai trò cầu nối giữa người dùng và kernel của operating system, chịu trách nhiệm nhận command người dùng nhập vào và gọi program tương ứng.

**Lập trình Shell** là quá trình dùng Shell interpreter (như bash) để kết hợp command, control structure (if/for/while), variable và function thành script tự động hóa. Shell vừa là command interpreter, vừa là một programming language hoàn chỉnh (hỗ trợ variable, array, function, flow control, pipe, redirection, v.v.).

**Các loại Shell phổ biến**:

- **bash** (Bourne Again Shell): Shell mặc định của hệ thống Linux, được dùng phổ biến nhất.
- **sh** (Bourne Shell): Shell truyền thống của Unix, theo chuẩn POSIX.
- **zsh**: Shell tương tác mạnh mẽ.
- **dash**: Shell nhẹ, `/bin/sh` mặc định của Ubuntu trỏ đến nó.
- **csh/tcsh**: Shell có phong cách C.

### Hello World trong lập trình Shell

Việc đầu tiên khi học bất kỳ programming language nào là output HelloWorld! Sau đây tôi sẽ trình bày cách output Hello World trong lập trình Shell, từ tạo file mới đến viết code Shell.

（1）Tạo file mới `helloworld.sh`: `touch helloworld.sh`, phần mở rộng là sh (sh là viết tắt của Shell) (phần mở rộng không ảnh hưởng đến việc thực thi script, chỉ cần dễ hiểu theo tên là được; nếu bạn dùng php để viết Shell script thì dùng phần mở rộng php cũng được).

（2）Cấp quyền thực thi cho script: `chmod +x helloworld.sh`

（3）Dùng command vim để sửa file helloworld.sh: `vim helloworld.sh` (vim file ------> vào file -----> command mode ------> nhấn i để vào edit mode -----> sửa file -------> nhấn Esc để vào bottom-line mode -----> nhập `:wq/q!` (nhập wq nghĩa là ghi nội dung và thoát, tức là lưu; nhập q! nghĩa là thoát cưỡng chế mà không lưu)).

Nội dung helloworld.sh như sau:

```shell
#!/bin/bash
set -euo pipefail  # Strict mode: thoát khi gặp lỗi, báo lỗi khi variable chưa định nghĩa, báo lỗi khi pipe thất bại
# Chương trình Shell đầu tiên, echo là command output trong Linux
echo "helloworld!"
```

Trong Shell, ký hiệu `#` biểu thị comment. **Dòng đầu tiên của Shell khá đặc biệt, thường bắt đầu bằng `#!` để chỉ định loại Shell được sử dụng. Ngoài bash Shell, Linux còn có nhiều phiên bản Shell khác như zsh, dash, v.v... Tuy nhiên bash Shell vẫn là loại được chúng ta sử dụng nhiều nhất.**

（4）Chạy script: `./helloworld.sh`. (Lưu ý, nhất định phải viết là `./helloworld.sh`, không phải `helloworld.sh`. Việc chạy các binary program khác cũng tương tự. Nếu viết trực tiếp `helloworld.sh`, hệ thống Linux sẽ tìm trong PATH xem có file tên helloworld.sh hay không; trong PATH chỉ có `/bin`, `/sbin`, `/usr/bin`, `/usr/sbin`, v.v., còn thư mục hiện tại thường không nằm trong PATH. Vì vậy phải viết `./helloworld.sh` để nói cho hệ thống biết hãy tìm trong thư mục hiện tại.)

![Hello World trong lập trình Shell](https://oss.javaguide.cn/github/javaguide/cs-basics/shell/55296212.jpg)

## Variable trong Shell

### Giới thiệu variable trong lập trình Shell

**Lập trình Shell thường có ba loại variable:**

1. **User-defined variable (local variable)**: mặc định chỉ có hiệu lực trong process Shell hiện tại, **subprocess không thể truy cập**. Nếu cần truyền cho subprocess, phải dùng `export` để khai báo thành environment variable.
2. **Environment variable**: chẳng hạn `PATH`, `HOME`, v.v., có thể được subprocess kế thừa. Dùng command `env` để xem tất cả environment variable, dùng command `set` để xem tất cả variable (bao gồm environment variable và local variable).
3. **Shell special variable**: special variable do Shell thiết lập (như `$?`, `$$`, `$!`, v.v.), dùng để lưu process state, parameter và các thông tin khác.

**Các environment variable thường dùng:**

> PATH quyết định Shell sẽ tìm command hoặc program trong những directory nào.
> HOME directory home của user hiện tại.
> HISTSIZE số lượng history.
> LOGNAME login name của user hiện tại.
> HOSTNAME tên host.
> SHELL loại Shell của user hiện tại.
> LANGUAGE environment variable liên quan đến language, có thể sửa environment variable này để hỗ trợ nhiều language.
> MAIL directory lưu mail của user hiện tại.
> PS1 prompt cơ bản, là # với root user và là \$ với user thông thường.

**Sử dụng environment variable đã được Linux định nghĩa:**

Ví dụ, để xem directory của user hiện tại, có thể dùng command `echo $HOME`; để xem loại Shell của user hiện tại, có thể dùng `echo $SHELL`. Có thể thấy cách sử dụng rất đơn giản.

**Sử dụng variable tự định nghĩa:**

```shell
#!/bin/bash
#variable tự định nghĩa hello
hello="hello world"
echo $hello
echo  "helloworld!"
```

![Sử dụng variable tự định nghĩa](https://oss.javaguide.cn/github/javaguide/cs-basics/shell/19835037.jpg)

**Lưu ý khi đặt tên variable trong lập trình Shell:**

- Tên chỉ được dùng chữ cái tiếng Anh, chữ số và dấu gạch dưới; ký tự đầu tiên không được bắt đầu bằng chữ số, nhưng có thể bắt đầu bằng dấu gạch dưới (`_`).
- Không được có khoảng trắng ở giữa, có thể dùng dấu gạch dưới (`_`).
- Không được dùng dấu câu.
- Không được dùng keyword trong bash (có thể dùng command help để xem keyword được reserved).

### Nhập môn string trong Shell

String là data type thường dùng và hữu ích nhất trong lập trình Shell (ngoài number và string ra thì cũng không có type nào khác thực sự tiện dụng), string có thể dùng single quote hoặc double quote. Điểm này khác với Java.

Trong single quote, mọi special character (như `$`, backtick, `\`, v.v.) đều mất ý nghĩa đặc biệt và được xem là literal.

Trong double quote, các character sau vẫn giữ ý nghĩa đặc biệt:

- `$`: variable expansion (như `$var`) và command substitution (như `$(cmd)` hoặc `` `cmd` ``)
- `\`: escape character
- `` ` `` hoặc `$()`: command substitution (khuyến nghị dùng syntax `$()`)
- `!`: history expansion (mặc định chỉ bật trong interactive Shell)
- `${}`: parameter expansion

**Lưu ý**: string trong single quote là **literal hoàn toàn**, còn string trong double quote sẽ thực hiện variable substitution và command substitution.

**String dùng single quote:**

```shell
#!/bin/bash
name='SnailClimb'
hello='Hello, I am $name!'
echo $hello
```

Nội dung output:

```plain
Hello, I am $name!
```

**String dùng double quote:**

```shell
#!/bin/bash
name='SnailClimb'
hello="Hello, I am $name!"
echo $hello
```

Nội dung output:

```plain
Hello, I am SnailClimb!
```

### Các thao tác thường gặp với string trong Shell

**Nối string:**

```shell
#!/bin/bash
name="SnailClimb"
# Nối bằng double quote
greeting="hello, "$name" !"
greeting_1="hello, ${name} !"
echo $greeting  $greeting_1
# Nối bằng single quote
greeting_2='hello, '$name' !'
greeting_3='hello, ${name} !'
echo $greeting_2  $greeting_3
```

Kết quả output:

![Kết quả output của command nối string trong Shell](https://oss.javaguide.cn/github/javaguide/cs-basics/shell/51148933.jpg)

**Lấy độ dài string:**

```shell
#!/bin/bash
# Lấy độ dài string
name="SnailClimb"
# Cách thứ nhất (khuyến nghị): built-in của bash
echo ${#name}  # output 10
# Cách thứ hai: external command (performance kém hơn)
expr length "$name"
```

Kết quả output:

```plain
10
10
```

**Giải thích**:

- Khuyến nghị dùng syntax `${#var}`, đây là chức năng built-in của bash và có performance tốt hơn.
- `expr` là external command, cần fork process nên performance kém hơn.
- **`expr length` là GNU extension**, không phải chuẩn POSIX. Trên BSD expr của macOS hoặc các system khác có thể không được hỗ trợ.
- Nếu cần tính portable, khuyến nghị dùng `${#var}` hoặc `expr "$var" : '.*'` (tương thích POSIX).

Khi dùng command expr, hai bên operator trong expression phải có khoảng trắng:

```shell
expr 5+6       # output trực tiếp 5+6 (không có khoảng trắng)
expr 5 + 6     # output 11 (có khoảng trắng)
# Khuyến nghị hơn là dùng arithmetic expansion của bash:
echo $((5 + 6))  # output 11
```

Với một số operator, còn cần dùng ký hiệu `\` để escape:

```shell
expr 5 * 6       # output lỗi (chưa escape)
expr 5 \* 6      # output 30 (escape đúng)
```

**Cắt substring:**

Cắt string đơn giản:

```shell
# Cắt 10 character từ character thứ 0 của string về sau (index bắt đầu từ 0)
str="SnailClimb is a great man"
echo ${str:0:10} #output:SnailClimb
```

Cắt theo expression:

```shell
#!/bin/bash
# author: amau

var="https://www.runoob.com/linux/linux-shell-variable.html"
# % biểu thị xóa kết quả match từ cuối, kết quả ngắn nhất
# %% biểu thị xóa kết quả match từ cuối, kết quả match dài nhất
# # biểu thị xóa kết quả match từ đầu, kết quả ngắn nhất
# ## biểu thị xóa kết quả match từ đầu, kết quả match dài nhất
# Lưu ý: * là wildcard, nghĩa là match bất kỳ số lượng character bất kỳ nào
s1=${var%%t*} #h
s2=${var%t*}  #https://www.runoob.com/linux/linux-shell-variable.h
s3=${var%%.*} #https://www
s4=${var#*/}  #/www.runoob.com/linux/linux-shell-variable.html
s5=${var##*/} #linux-shell-variable.html
```

### Array trong Shell

**bash 2.0+** hỗ trợ array một chiều (không hỗ trợ array nhiều chiều) và không giới hạn kích thước array.

**Lưu ý quan trọng**: array là **tính năng extension không thuộc POSIX của bash**, POSIX sh thuần (như dash) không hỗ trợ array. Nếu cần viết script portable, nên tránh dùng array.

Dưới đây là ví dụ code Shell về thao tác với array. Qua ví dụ này, bạn có thể biết cách tạo array, lấy độ dài array, lấy/xóa phần tử array ở vị trí cụ thể, xóa toàn bộ array và duyệt array.

```shell
#!/bin/bash
array=(1 2 3 4 5);
# Lấy độ dài array
length=${#array[@]}
# hoặc
length2=${#array[*]}
#output độ dài array
echo $length #output: 5
echo $length2 #output: 5
# output phần tử thứ ba của array
echo ${array[2]} #output: 3
unset 'array[1]' # Xóa phần tử có index là 1, tức phần tử thứ hai
for i in "${array[@]}"; do echo "$i"; done # Duyệt array, output: 1 3 4 5
unset array; # Xóa toàn bộ phần tử trong array
for i in "${array[@]}"; do echo "$i"; done # Duyệt array, array rỗng nên không có output
```

**Giải thích quan trọng: khoảng trống trong index của array**:

Sau khi dùng `unset array[1]` để xóa phần tử, array sẽ xuất hiện **khoảng trống trong index**:

```shell
#!/bin/bash
array=(1 2 3 4 5)
echo "Trước khi xóa: ${array[@]}"  # output: 1 2 3 4 5
echo "Giá trị index 1: ${array[1]}"  # output: 2

unset array[1]  # Xóa phần tử ở index 1
echo "Sau khi xóa: ${array[@]}"  # output: 1 3 4 5
echo "Giá trị index 1: ${array[1]}"  # output: (giá trị rỗng)
echo "Giá trị index 2: ${array[2]}"  # output: 3 (index 2 vẫn tồn tại)

# Index không liên tiếp khi duyệt
for index in "${!array[@]}"; do
    echo "Index[$index] = ${array[$index]}"
done
# output:
# Index[0] = 1
# Index[2] = 3
# Index[3] = 4
# Index[4] = 5
```

**Lưu ý**: sau khi xóa phần tử, truy cập bằng `${array[1]}` sẽ nhận được giá trị rỗng. Khi duyệt array, nên dùng `"${!array[@]}"` để lấy index hợp lệ hoặc dùng `"${array[@]}"` để duyệt trực tiếp các value.

## Operator cơ bản của Shell

Lập trình Shell hỗ trợ các operator sau:

- Arithmetic operator
- Relational operator
- Boolean operator
- String operator
- File test operator

### Arithmetic operator

| **Operator** | **Giải thích** | **Ví dụ**                                                         |
| ------------ | -------------- | ----------------------------------------------------------------- |
| **+**        | Cộng           | `expr $a + $b`                                                    |
| **-**        | Trừ            | `expr $a - $b`                                                    |
| **\***       | Nhân           | `expr $a \* $b` (lưu ý cần escape dấu sao)                        |
| **/**        | Chia           | `expr $b / $a`                                                    |
| **%**        | Lấy dư         | `expr $b % $a`                                                    |
| **=**        | Gán            | `a=$b` gán value của variable b cho a                             |
| **==**       | Bằng nhau      | `[ "$a" == "$b" ]` dùng để so sánh string, giống nhau trả về true |
| **!=**       | Khác nhau      | `[ "$a" != "$b" ]` dùng để so sánh string, khác nhau trả về true  |

**Khuyến nghị dùng arithmetic expansion built-in của bash**:

```shell
#!/bin/bash
a=3; b=3
val=$((a + b))  # arithmetic expansion của bash (khuyến nghị)
# output: Total value: 6
echo "Total value: $val"
```

**Giải thích**:

- `$((...))` là chức năng built-in của bash, không cần fork external process nên performance tốt hơn.
- **Không khuyến nghị** dùng command `expr` (cần fork process và hai bên operator phải có khoảng trắng).
- **Không khuyến nghị** dùng backtick `` `...` `` (đã cũ), nên dùng syntax `$(...)`.

**Nếu cần tương thích với POSIX sh**, có thể dùng:

```shell
val=$(expr "$a" + "$b")  # tương thích POSIX nhưng performance kém hơn
```

### Relational operator

Relational operator chỉ hỗ trợ number, không hỗ trợ string, trừ khi value của string là number.

| **Operator** | **Giải thích**                                                       | **Tiếng Anh tương ứng** |
| ------------ | -------------------------------------------------------------------- | ----------------------- |
| **-eq**      | Kiểm tra hai number có **bằng nhau** hay không                       | equal                   |
| **-ne**      | Kiểm tra hai number có **khác nhau** hay không                       | not equal               |
| **-gt**      | Kiểm tra number bên trái có **lớn hơn** bên phải hay không           | greater than            |
| **-lt**      | Kiểm tra number bên trái có **nhỏ hơn** bên phải hay không           | less than               |
| **-ge**      | Kiểm tra number bên trái có **lớn hơn hoặc bằng** bên phải hay không | greater equal           |
| **-le**      | Kiểm tra number bên trái có **nhỏ hơn hoặc bằng** bên phải hay không | less equal              |

Ví dụ đơn giản dưới đây minh họa cách dùng relational operator. Chương trình Shell sau sẽ output A khi score=100, nếu không thì output B.

```shell
#!/bin/bash
score=90;
maxscore=100;
if [[ $score -eq $maxscore ]]
then
   echo "A"
else
   echo "B"
fi
```

Kết quả output:

```plain
B
```

### Logical operator

| **Operator** | **Giải thích** | **Ví dụ**                                                       |
| ------------ | -------------- | --------------------------------------------------------------- |
| **&&**       | **AND** logic  | `[[ $a -lt 100 && $b -gt 100 ]]` (chỉ true khi tất cả đều true) |
| **\|\|**     | **OR** logic   | `[[ $a -lt 100 \|\| $b -gt 100 ]]` (chỉ cần một true là true)   |

**Phép logic trong arithmetic expansion**:

```shell
#!/bin/bash
a=$(( 1 && 0))
# output: 0; phép AND logic chỉ cho kết quả 1 khi hai vế đều là 1; nếu không kết quả là 0
echo $a;
```

**Thực thi command short-circuit (thường dùng trong production)**:

Trong automation vận hành và pipeline CI/CD, thường dùng `&&` và `||` để điều khiển flow thực thi của command chain. Cách này gọi là **thực thi short-circuit**:

```shell
#!/bin/bash
set -euo pipefail

# &&: chỉ thực thi command sau khi command trước đó thành công (trả về 0)
mkdir -p "/tmp/app_data" && echo "Directory đã sẵn sàng"

# ||: chỉ thực thi command sau khi command trước đó thất bại (trả về khác 0)
mkdir -p "/tmp/app_data" || echo "Tạo directory thất bại"

# Kết hợp: cách phòng vệ điển hình trong production
mkdir -p "/tmp/app_data" && echo "Directory đã sẵn sàng" || exit 1

# Ví dụ tình huống thực tế
# 1. Xóa file sau khi kiểm tra file tồn tại
[ -f "/tmp/old_file.log" ] && rm "/tmp/old_file.log"

# 2. Output error và thoát khi command thất bại
cd /app/config || { echo "Không thể vào directory cấu hình"; exit 1; }

# 3. Thực thi command có điều kiện
command1 && command2 || command3
# ⚠️ Lưu ý: cách viết này có bẫy!
# - Khi command1 thành công, thực thi command2
# - Khi command1 thất bại, thực thi command3
# - Nhưng nếu command1 thành công mà command2 thất bại, command3 vẫn được thực thi!
#
# ✅ Cách viết an toàn hơn (khuyến nghị):
if command1; then
    command2
else
    command3
fi
#
# Hoặc chỉ dùng tổ hợp && || khi biết chắc command2 sẽ không thất bại
```

**Lưu ý quan trọng**:

- Thực thi short-circuit phụ thuộc vào **exit code** của command: thành công trả về 0, thất bại trả về khác 0.
- Điều này khác với `&&` và `||` bên trong `[[ ]]`, vì hai operator sau được dùng để test condition.
- `command1 && command2 || command3` có bẫy: nếu command1 thành công nhưng command2 thất bại, command3 vẫn được thực thi.
- Trong production, đặc biệt khuyến nghị dùng cấu trúc if-then-else để bảo đảm logic rõ ràng.

### Boolean operator

| **Operator** | **Giải thích**                                                                                  | **Ví dụ**                                                  |
| ------------ | ----------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| **!**        | Đảo kết quả của expression. Nếu expression là true thì trả về false; nếu không thì trả về true. | `[ ! false ]` trả về false vì `false` là non-empty string. |
| **-o**       | Nếu có một expression là true thì trả về true.                                                  | `[ "$a" -lt 20 -o "$b" -gt 100 ]` trả về true.             |
| **-a**       | Chỉ trả về true khi cả hai expression đều là true.                                              | `[ "$a" -lt 20 -a "$b" -gt 100 ]` trả về false.            |

### String operator

| **Operator** | **Giải thích**                                        | **Ví dụ**                                |
| ------------ | ----------------------------------------------------- | ---------------------------------------- |
| **=**        | Kiểm tra hai string có **bằng nhau** hay không        | `[ "$a" = "$b" ]`                        |
| **!=**       | Kiểm tra hai string có **khác nhau** hay không        | `[ "$a" != "$b" ]`                       |
| **-z**       | Kiểm tra độ dài string có bằng **0** (zero) hay không | `[ -z "$a" ]` nếu rỗng trả về true       |
| **-n**       | Kiểm tra độ dài string có **khác 0** hay không        | `[ -n "$a" ]` nếu không rỗng trả về true |
| **str**      | Trực tiếp kiểm tra string có rỗng hay không           | `[ "$a" ]` nếu không rỗng trả về true    |

Ví dụ đơn giản:

```shell
#!/bin/bash
a="abc";
b="efg";
if [[ "$a" = "$b" ]]
then
   echo "a bằng b"
else
   echo "a khác b"
fi
```

Output:

```plain
a khác b
```

### Operator liên quan đến file

Dùng để kiểm tra các thuộc tính khác nhau của file Unix/Linux (như permission, type, v.v.).

- **Kiểm tra tồn tại và type:**
  - **-e file**: kiểm tra file (bao gồm directory) có tồn tại hay không.
  - **-f file**: kiểm tra có phải regular file hay không (không phải directory cũng không phải device file).
  - **-d file**: kiểm tra có phải directory hay không.
  - **-s file**: kiểm tra file có non-empty hay không (file size lớn hơn 0 thì trả về true).
  - **-b/-c/-p**: lần lượt kiểm tra có phải block device, character device, named pipe hay không.
- **Kiểm tra permission:**
  - **-r file**: kiểm tra file có thể đọc hay không.
  - **-w file**: kiểm tra file có thể ghi hay không.
  - **-x file**: kiểm tra file có thể thực thi hay không.
- **Kiểm tra special flag:**
  - **-u / -g / -k**: lần lượt kiểm tra file có được set SUID, SGID hoặc sticky bit hay không.

Cách sử dụng rất đơn giản. Ví dụ, ta định nghĩa path của một file là `file="/usr/learnshell/test.sh"`. Nếu muốn kiểm tra file này có thể đọc hay không, có thể viết `if [ -r $file ]`; nếu muốn kiểm tra có thể ghi hay không, có thể viết `-w $file`.

## Flow control của Shell

### Câu lệnh điều kiện if

Ví dụ đơn giản về câu lệnh điều kiện if else-if else:

```shell
#!/bin/bash
a=3;
b=9;
if [[ $a -eq $b ]]
then
   echo "a bằng b"
elif [[ $a -gt $b ]]
then
   echo "a lớn hơn b"
else
   echo "a nhỏ hơn b"
fi
```

Kết quả output:

```plain
a nhỏ hơn b
```

Qua ví dụ trên, chắc hẳn bạn đã nắm được câu lệnh điều kiện if trong lập trình Shell.

**Xử lý câu lệnh rỗng**: trong Shell, câu lệnh rỗng có thể được thực hiện bằng `:` (colon command) hoặc command `true`:

```shell
if [[ condition ]]; then
    :  # câu lệnh rỗng (không làm gì)
fi

# hoặc
if [[ condition ]]; then
    true  # câu lệnh rỗng
fi
```

Điều này hữu ích trong một số tình huống, chẳng hạn dùng làm placeholder trong vòng lặp while.

### Câu lệnh vòng lặp for

Ba ví dụ đơn giản dưới đây giúp làm quen với cách dùng cơ bản nhất của vòng lặp for. Trên thực tế, chức năng của vòng lặp for lớn hơn nhiều so với những gì thể hiện trong các ví dụ sau.

**Output data trong list hiện tại:**

```shell
for loop in 1 2 3 4 5
do
    echo "The value is: $loop"
done
```

**Tạo 10 số ngẫu nhiên:**

```shell
#!/bin/bash
for i in {0..9};
do
   echo $RANDOM;
done
```

**Output từ 1 đến 5:**

Thông thường khi gọi Shell variable cần thêm $, nhưng trong (()) của for thì không cần. Hãy xem ví dụ sau:

```shell
#!/bin/bash
length=5
for((i=1;i<=length;i++));do
    echo $i;
done;
```

### Câu lệnh while

**Câu lệnh vòng lặp while cơ bản:**

```shell
#!/bin/bash
int=1
while (( int <= 5 ))  # Trong arithmetic context, variable không cần $
do
    echo $int
    (( int++ ))  # Khuyến nghị dùng (( )) thay cho let
done
```

**Vòng lặp while có thể dùng để đọc thông tin từ keyboard:**

```shell
echo 'Nhấn <CTRL-D> để thoát'
echo -n 'Nhập bộ phim bạn yêu thích nhất: '
while read -r FILM  # option -r tắt escape backslash, tăng tính an toàn
do
    echo "Đúng vậy! $FILM là một bộ phim hay"
done
```

Nội dung output:

```plain
Nhấn <CTRL-D> để thoát
Nhập bộ phim bạn yêu thích nhất: Transformers
Đúng vậy! Transformers là một bộ phim hay
```

**Vòng lặp vô hạn:**

```shell
while true
do
    command
done
```

## Function trong Shell

### Function không có parameter và không có return value

```shell
#!/bin/bash
hello(){
    echo "Đây là function Shell đầu tiên của tôi!"
}
echo "-----Bắt đầu thực thi function-----"
hello
echo "-----Thực thi function hoàn tất-----"
```

Kết quả output:

```plain
-----Bắt đầu thực thi function-----
Đây là function Shell đầu tiên của tôi!
-----Thực thi function hoàn tất-----
```

### Function có return value

**Nhập hai number, cộng và output kết quả:**

```shell
#!/bin/bash
set -euo pipefail

funWithReturn(){
    local aNum
    local anotherNum
    echo "Nhập number thứ nhất: "
    read -r aNum
    echo "Nhập number thứ hai: "
    read -r anotherNum
    echo "Hai number lần lượt là $aNum và $anotherNum !"
    result=$((aNum + anotherNum))
}
result=0
funWithReturn
echo "Tổng của hai number đã nhập là $result"
```

**Giải thích quan trọng**:

- **Keyword `local`**: giới hạn variable trong function scope, tránh làm nhiễm global namespace.
- **`read -r`**: option `-r` tắt escape backslash, tăng tính an toàn.
- **Return value của function**: `return` thiết lập exit status từ 0-255, không phù hợp để truyền calculation result. Khi cần truyền data, có thể dùng standard output hoặc variable.

**Tại sao dùng local?**

- Trong script phức tạp hoặc khi import nhiều external script, non-local variable có thể bị ghi đè ngoài ý muốn.
- Global variable pollution có thể dẫn đến configuration drift hoặc logic privilege escalation khó truy vết.
- Dùng `local` là best practice của functional programming, tương tự khái niệm local variable trong các programming language khác.

Kết quả output:

```plain
Nhập number thứ nhất:
1
Nhập number thứ hai:
2
Hai number lần lượt là 1 và 2 !
Tổng của hai number đã nhập là 3
```

### Function có parameter

```shell
#!/bin/bash
funWithParam(){
    echo "Parameter thứ nhất là $1"
    echo "Parameter thứ hai là $2"
    echo "Tên script là $0"
    echo "Parameter thứ mười là ${10}"   # Lưu ý: khi parameter >= 10 phải dùng ${n}
    echo "Parameter thứ mười một là ${11}"
    echo "Tổng số parameter là $#"
    echo "Tất cả parameter là $*"         # output dưới dạng một string duy nhất
    echo "Tất cả parameter là $@"         # output dưới dạng các parameter độc lập (khuyến nghị)
}
funWithParam 1 2 3 4 5 6 7 8 9 34 73
```

Kết quả output:

```plain
Parameter thứ nhất là 1
Parameter thứ hai là 2
Tên script là ./script.sh
Parameter thứ mười là 34
Parameter thứ mười một là 73
Tổng số parameter là 11
Tất cả parameter là 1 2 3 4 5 6 7 8 9 34 73
Tất cả parameter là 1 2 3 4 5 6 7 8 9 34 73
```

**Lưu ý quan trọng**:

- **Positional parameter `$n` khi `n >= 10` bắt buộc phải dùng syntax `${n}`**.
- Ví dụ: `$10` sẽ được parse thành phép nối `$1` và literal `0`, không phải parameter thứ mười.
- `$0` biểu thị tên của chính script.
- `$#` biểu thị tổng số parameter.

**Khác biệt cốt lõi giữa `$*` và `$@`**:

| Expression | Không được quote              | Được bọc bằng double quote                                         |
| ---------- | ----------------------------- | ------------------------------------------------------------------ |
| `$*`       | Expand thành tất cả parameter | Expand thành **một string duy nhất** (gộp tất cả parameter)        |
| `$@`       | Expand thành tất cả parameter | Expand thành **các parameter độc lập** (mỗi parameter giữ độc lập) |

**So sánh qua ví dụ**:

```shell
#!/bin/bash
test_args() {
    echo "--- Dùng \$* (không quote)---"
    for arg in $*; do
        echo "Parameter: [$arg]"
    done

    echo -e "\n--- Dùng \$@ (không quote)---"
    for arg in $@; do
        echo "Parameter: [$arg]"
    done

    echo -e "\n--- Dùng \"\$*\" (double quote)---"
    for arg in "$*"; do
        echo "Parameter: [$arg]"
    done

    echo -e "\n--- Dùng \"\$@\" (double quote, khuyến nghị)---"
    for arg in "$@"; do
        echo "Parameter: [$arg]"
    done
}

# Gọi function, truyền parameter có khoảng trắng
test_args "hello world" "foo bar"
```

**Kết quả output**:

```plain
--- Dùng $* (không quote)---
Parameter: [hello]
Parameter: [world]
Parameter: [foo]
Parameter: [bar]

--- Dùng $@ (không quote)---
Parameter: [hello]
Parameter: [world]
Parameter: [foo]
Parameter: [bar]

--- Dùng "$*" (double quote)---
Parameter: [hello world foo bar]  # Gộp tất cả parameter thành một string

--- Dùng "$@" (double quote, khuyến nghị)---
Parameter: [hello world]  # Mỗi parameter giữ độc lập
Parameter: [foo bar]
```

**Kết luận**: khi truyền parameter, **luôn dùng `"$@"`** để bảo đảm tính độc lập của mỗi parameter (đặc biệt khi parameter có khoảng trắng).

## Best practice trong lập trình Shell

Sau khi nắm được kiến thức cơ bản về lập trình Shell, hiểu một số best practice sẽ giúp bạn viết script an toàn và hiệu quả hơn.

### Quy chuẩn cơ bản của script

**1. Quy chuẩn Shebang**:

```shell
#!/usr/bin/env bash
# Tìm bash qua PATH
set -euo pipefail
```

**Hai cách viết Shebang**:

- `#!/bin/bash`: chỉ định trực tiếp path của bash, phù hợp với environment cố định khi bạn biết vị trí bash.
- `#!/usr/bin/env bash`: tìm bash qua env, portable hơn, phù hợp với các system khác nhau (như macOS / Linux).

**Lựa chọn trong bài viết này**:

- Ví dụ tutorial dùng `#!/bin/bash`: ngắn gọn, rõ ràng, phù hợp để beginner hiểu.
- Ví dụ production dùng `#!/usr/bin/env bash`: nhấn mạnh tính portable.

**2. Quote variable**:

```shell
# Luôn bọc variable bằng double quote
echo "$var"     # khuyến nghị
echo $var       # có thể gây vấn đề word splitting và globbing
```

**3. Dùng shellcheck**:

```bash
shellcheck your_script.sh  # static analysis, phát hiện vấn đề thường gặp
```

**4. Syntax khuyến nghị**:

- Dùng `[[ ]]` thay cho `[ ]` (an toàn hơn, hỗ trợ pattern matching).
- Dùng `$((...))` thay cho `expr` (performance tốt hơn).
- Dùng `$(...)` thay cho backtick (có thể lồng nhau, rõ ràng hơn).
- Dùng `${n}` để truy cập positional parameter n >= 10.

### Nguyên lý hoạt động của pipefail

Theo mặc định, return value của pipe command chỉ phụ thuộc vào command cuối cùng. Sau khi bật `pipefail`, return value của pipe sẽ là return value của command thất bại cuối cùng, giúp tránh bỏ sót lỗi ở các bước giữa.

**So sánh qua ví dụ**:

```shell
# Mode mặc định (nguy hiểm)
cat huge_file.txt | grep "pattern" | head -n 10
# Dù cat thất bại (file không tồn tại), chỉ cần head thành công thì return code vẫn là 0

# Mode pipefail (an toàn)
set -o pipefail
cat huge_file.txt | grep "pattern" | head -n 10
# cat thất bại sẽ lập tức trả về error code, không bị bỏ qua
```

## Trước khi đưa Shell script vào production

Viết đúng syntax cơ bản mới chỉ là bước đầu. Sau khi script đi vào scheduled task, deployment flow hoặc machine production, còn phải xử lý việc thoát khi thất bại, file tạm, network timeout và exit code của background task. Dưới đây là một số ví dụ hoàn chỉnh.

Hành vi của `set -u` và `set -o pipefail` tương đối rõ ràng; `set -e` có nhiều exception hơn, hành vi trong conditional, function, sub Shell và command substitution đều có thể gây bất ngờ. Có thể xem `set -euo pipefail` là điểm khởi đầu cho script mới, nhưng không thể dùng nó thay cho việc xử lý error tường minh bằng `if ! command; then ... fi`. Việc expand variable vẫn nên thêm double quote tùy tình huống, còn variable bên trong function nên dùng `local` để giới hạn scope.

### Đặt tổng budget cho network request

Một request cần đồng thời giới hạn cả giai đoạn connect và toàn bộ thời gian truyền. Retry chỉ nên đặt ở một layer: nếu đồng thời bật vòng lặp bên ngoài và `curl --retry`, số request thực tế sẽ nhân lên, tổng thời gian cũng rất khó ước lượng. Function dưới đây do bên ngoài thống nhất kiểm soát số lần thử, đồng thời thêm delay ngẫu nhiên dạng integer từ 0 đến 2 giây sau mỗi lần thất bại:

```shell
#!/usr/bin/env bash

retry_request() {
    local url="$1"
    local max_attempts=5
    local attempt=1
    local delay
    local jitter

    while (( attempt <= max_attempts )); do
        if curl --fail --silent --show-error \
                --connect-timeout 3 \
                --max-time 10 \
                "$url"; then
            return 0
        fi

        if (( attempt == max_attempts )); then
            break
        fi

        delay=$((1 << (attempt - 1)))
        (( delay > 16 )) && delay=16
        jitter=$((RANDOM % 3))
        delay=$((delay + jitter))
        printf 'Request lần %d thất bại, retry sau %d giây\n' "$attempt" "$delay" >&2
        sleep "$delay"
        ((attempt += 1))
    done

    return 1
}

command -v curl >/dev/null 2>&1 || {
    echo "curl chưa được cài đặt" >&2
    exit 1
}

[[ $# -eq 1 ]] || {
    echo "Cách dùng: $0 <url>" >&2
    exit 1
}

retry_request "$1" || {
    echo "Request thất bại" >&2
    exit 1
}
```

Script thực tế còn phải căn cứ vào ngữ nghĩa của API để quyết định error nào có thể retry. Non-idempotent write request, authentication failure và parameter error thường không nên replay trực tiếp.

### File tạm và lock loại trừ lẫn nhau

Không dùng các path dễ đoán như `/tmp/data_$$`. `mktemp` sẽ tạo file hoặc directory một cách atomic; kết hợp với `trap`, nó có thể cleanup khi thoát bình thường và khi nhận các signal phổ biến:

```shell
#!/usr/bin/env bash

temp_dir=$(mktemp -d "${TMPDIR:-/tmp}/myapp.XXXXXXXX") || {
    echo "Không thể tạo directory tạm" >&2
    exit 1
}
cleanup() {
    rm -rf -- "$temp_dir"
}
trap cleanup EXIT
trap 'exit 129' HUP
trap 'exit 130' INT
trap 'exit 143' TERM

temp_file="$temp_dir/result.txt"
printf 'temporary data\n' > "$temp_file"
cat "$temp_file"
```

Khi cần ngăn script chạy lặp trên cùng một machine, có thể dùng `flock` cho lock file do chính application quản lý:

```shell
exec 9>/var/lock/myapp.lock || exit 1
flock -n 9 || {
    echo "Script đang chạy" >&2
    exit 1
}
```

`flock` là cooperative lock, process khác có thể chọn không tuân thủ. NFS client của Linux có thể mô phỏng nó thành whole-file `fcntl` lock, nhưng hành vi thực tế còn chịu ảnh hưởng của kernel client, server và các mount option như `local_lock`. Khi lock file nằm trên network file system, cần kiểm chứng bằng hai client độc lập trong environment deploy mục tiêu, không thể mặc định rằng nó chắc chắn có hiệu lực hoặc chắc chắn mất hiệu lực. Mutual exclusion giữa các machine cũng không thể chỉ viết một câu Redis `SET NX PX`: implementation còn phải xử lý unique token, conditional delete, lease renewal và failure model.

### Thu thập exit code của background task

`wait` không có parameter sẽ chờ tất cả background task, nhưng Bash trả về 0 và không cho biết task nào thất bại. Script dưới đây đếm byte của nhiều file đồng thời và lần lượt thu thập status của subprocess:

```shell
#!/usr/bin/env bash
set -u

pids=()
for file in "$@"; do
    wc -c -- "$file" &
    pids+=("$!")
done

exit_code=0
for pid in "${pids[@]}"; do
    if ! wait "$pid"; then
        echo "Background task $pid thực thi thất bại" >&2
        exit_code=1
    fi
done

exit "$exit_code"
```

Viết trực tiếp `while wait -n; do ...; done` cũng chưa đầy đủ: khi một task thất bại, `wait -n` trả về khác 0 khiến vòng lặp kết thúc ngay, các task còn lại có thể không được thu thập.

### Các hiểu lầm thường gặp

Không redirect cả standard output và standard error của toàn bộ command vào `/dev/null` trong thời gian dài, nếu không khi thất bại sẽ chỉ còn exit code mà không có thông tin chẩn đoán. Chỉ suppress output mà bạn chắc chắn không cần, còn error message hãy ghi vào log hoặc giữ trong standard error. Khi script phụ thuộc vào external command như `curl`, `jq`, v.v., hãy dùng `command -v` để kiểm tra ngay ở giai đoạn khởi động; khi pipe cần nhận biết lỗi của command ở giữa, hãy bật `set -o pipefail`.

### Xác minh trước khi go-live

Nội dung xác minh cần bám sát dependency thực tế của script. Network script tối thiểu phải bao phủ connect failure, timeout và HTTP status không thể retry; concurrent script kiểm tra exit code của từng subprocess; script tạo resource tạm còn phải xác minh việc cleanup hoàn tất sau khi thoát bình thường và sau khi bị signal interrupt. Các command fault injection sẽ sửa firewall, system time hoặc mount state, không phù hợp làm ví dụ chung có thể copy trực tiếp; nên thiết kế riêng trong test environment cô lập theo infrastructure thực tế.

## Tổng kết

Shell phù hợp để nối các command có sẵn thành flow automation nhỏ. Trước hết hãy nắm variable quoting, condition, loop, function và exit status, sau đó bổ sung timeout, cleanup và error handling tùy theo network, file và concurrent resource mà script thực tế sử dụng.

### Ôn lại các điểm kiến thức cốt lõi

| Module kiến thức         | Điểm chính                                                                                                                                      |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Variable**             | Phân biệt local variable, environment variable và special variable; dùng `local` để tránh global pollution; luôn bọc variable bằng double quote |
| **String**               | Khuyến nghị dùng double quote; hiểu khác biệt giữa single quote và double quote; nắm cách dùng `${#var}` để lấy length                          |
| **Array**                | bash 2.0+ hỗ trợ array (không thuộc POSIX); lưu ý khoảng trống trong index sau khi xóa phần tử                                                  |
| **Operator**             | Ưu tiên dùng `$((...))` cho arithmetic operation; `[[ ]]` an toàn hơn `[ ]`                                                                     |
| **Flow control**         | Dùng `[[ ]]` để test condition; tránh bẫy của `command1 && command2 \|\| command3`                                                              |
| **Function**             | Dùng `local` để giới hạn scope của variable; function chỉ có thể trả về exit code 0-255                                                         |
| **Command substitution** | Dùng `$(...)` thay cho backtick; dùng `read -r` để tăng tính an toàn                                                                            |

### Gợi ý học tập

Bắt đầu với các task ngắn như filter log, đổi tên file hàng loạt và thống kê file. Sau khi viết xong, trước hết dùng `bash -n` để kiểm tra syntax, sau đó dùng ShellCheck để tìm các vấn đề thường gặp như variable chưa quote, redirect sai, v.v. Sau khi script bắt đầu quản lý background process hoặc system service, hãy tiếp tục học signal, job control, `sed`, `awk` và `grep`. Khi script dài hơn vài trăm dòng, cần data structure phức tạp hoặc exception recovery, các ngôn ngữ đa dụng như Python thường dễ maintain hơn.

### Tài nguyên tham khảo

- **Tài liệu chính thức**: Bash Reference Manual (GNU)
- **Code checking**: ShellCheck - Shell Script Analysis Tool
- **Coding convention**: Google Shell Style Guide
- **Pitfall thường gặp**: Bash Pitfalls (http://mywiki.wooledge.org/BashPitfalls)
