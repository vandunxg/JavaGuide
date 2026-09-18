---
title: Vì sao khi quên mật khẩu chỉ có thể đặt lại, không thể cho biết mật khẩu cũ?
description: "Giải thích chi tiết vì sao khi quên mật khẩu, website chỉ có thể yêu cầu bạn đặt lại mật khẩu mà không thể cho biết mật khẩu cũ. Nguyên nhân cốt lõi là server sử dụng hash algorithm để lưu trữ mật khẩu; hash algorithm không thể đảo ngược nên không thể khôi phục mật khẩu ban đầu từ hash value. Bài viết cũng giới thiệu về bảo mật lưu trữ mật khẩu, cơ chế thêm salt, Bcrypt, bảo mật truyền mật khẩu và các kiến thức khác."
category:
  - System Design
tag:
  - Data Security
  - Password Security
  - Hash Algorithm
  - Câu hỏi phỏng vấn
head:
  - - meta
    - name: keywords
      content: password reset,password retrieval,hash algorithm,password storage,Bcrypt,salt,password security,câu hỏi phỏng vấn
---

Đây là một câu hỏi khá thú vị, nhiều công ty cũng từng hỏi trong phỏng vấn. Câu hỏi khá đơn giản, không biết bình thường khi đặt lại mật khẩu bạn có từng nghĩ đến vấn đề này chưa.

![Đặt lại mật khẩu tài khoản](https://oss.javaguide.cn/github/javaguide/system-design/security/reset-password-page.png)

Thực ra câu trả lời cho vấn đề này chỉ có một câu: **vì server cũng không biết mật khẩu cũ của bạn là gì**. Lập trình viên lưu mật khẩu gốc đã bị sa thải rồi 🤣.

Nếu server biết mật khẩu cũ của bạn thì đó là một rủi ro bảo mật nghiêm trọng.

Hãy cùng phân tích ngắn gọn.

Bài viết này sẽ không nói quá nhiều về các hash algorithm. Nếu bạn quan tâm, có thể xem bài viết này: [Tổng hợp các hash algorithm thường gặp](https://javaguide.cn/system-design/security/encryption-algorithms.html).

![](https://oss.javaguide.cn/github/javaguide/system-design/security/encryption-algorithms/javaguide-security-encryption-algorithms.png)

## Vì sao server không biết mật khẩu cũ của bạn?

Những ai từng làm phát triển có lẽ đều biết rằng khi lưu mật khẩu vào database, **tuyệt đối không được lưu trực tiếp dưới dạng plaintext**.

Nếu lưu dưới dạng plaintext, rủi ro quá lớn:

1. Dữ liệu database có nguy cơ bị đánh cắp.
2. Nhân viên nội bộ có quyền truy cập database có thể sử dụng với mục đích xấu.
3. Sau khi hacker xâm nhập, họ có thể lấy trực tiếp mật khẩu của tất cả người dùng.

Vì vậy, mật khẩu phải được xử lý trước khi lưu trữ. Cách xử lý này là sử dụng **hash algorithm**.

## Giới thiệu về hash algorithm

Hash algorithm còn được gọi là hàm băm hoặc algorithm tạo digest. Nó dùng để tạo một định danh duy nhất có độ dài cố định từ dữ liệu có độ dài bất kỳ, cũng được gọi là hash value, giá trị băm hoặc message digest (sau đây gọi chung là hash value).

![Minh họa hiệu quả của hash algorithm](https://oss.javaguide.cn/github/javaguide/system-design/security/encryption-algorithms/hash-function-effect-demonstration.png)

Hash algorithm có hai đặc điểm quan trọng:

1. **Tính không thể đảo ngược**: Không thể lấy lại giá trị ban đầu từ value sau khi hash. Đây là điểm cốt lõi!
2. **Tính xác định**: Cùng một input luôn tạo ra cùng một output.

Có một phép ví dụ rất trực quan: **mật khẩu bạn lưu giống như khoai tây đã thái sợi, không thể khôi phục thành củ khoai tây. Nhưng cách website kiểm tra mật khẩu có đúng hay không là thái mật khẩu mới bạn nhập thành sợi khoai tây một lần nữa, rồi xem hai đĩa khoai tây sợi có giống nhau không.**

Hai đặc điểm này khiến hash algorithm rất phù hợp để lưu trữ mật khẩu: server chỉ lưu hash value của mật khẩu, khi xác minh chỉ cần so sánh các hash value có giống nhau hay không.

### Phân loại hash algorithm

Hash algorithm có thể được chia đơn giản thành hai loại:

1. **Cryptographic hash algorithm**: Hash algorithm có tính bảo mật cao hơn, có thể cung cấp một mức bảo vệ toàn vẹn dữ liệu và chống sửa đổi dữ liệu nhất định, chống lại được một số phương thức tấn công, tính bảo mật tương đối cao nhưng performance kém hơn, phù hợp với các trường hợp sử dụng yêu cầu tính bảo mật cao. Ví dụ: SHA2, SHA3, SM3, RIPEMD-160, BLAKE2, v.v.
2. **Non-cryptographic hash algorithm**: Hash algorithm có tính bảo mật tương đối thấp, dễ bị ảnh hưởng bởi các phương thức tấn công như brute force, collision attack, nhưng performance cao hơn, phù hợp với các business không yêu cầu tính bảo mật. Ví dụ: CRC32, MurMurHash3, v.v.

Ngoài hai loại này, còn có một số hash algorithm đặc biệt, chẳng hạn **slow hash algorithm** có tính bảo mật cao hơn.

### Vì sao không khuyến nghị MD5?

Trước đây MD5 thường được dùng để mã hóa mật khẩu, nhưng hiện nay **không còn được khuyến nghị**, vì các lý do sau:

1. **Khả năng chống collision kém**: Tồn tại vấn đề weak collision, tức là nhiều input khác nhau có thể tạo ra cùng một MD5 value.
2. **Hash value ngắn**: Hash value 128 bit dễ bị rainbow table attack.
3. **Tốc độ tính toán quá nhanh**: Ngược lại khiến nó dễ bị brute force.

Có thể đọc bài viết này để xem giới thiệu chi tiết: [Đừng viết mã hóa mật khẩu bằng MD5 vào CV nữa!](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247542780&idx=1&sn=fb2fe3fb53fe596cc5b22e30766e0098&scene=21#wechat_redirect)

### Vì sao cần thêm salt?

Chỉ sử dụng hash algorithm để lưu trữ mật khẩu vẫn có nguy cơ bị **rainbow table attack**. Rainbow table là một bảng đối chiếu hash value được tính toán trước, attacker có thể nhanh chóng crack mật khẩu bằng cách tra bảng.

Salt là một value ngẫu nhiên được tạo riêng cho mỗi mật khẩu, password hash algorithm sẽ cùng tính toán salt và mật khẩu. Salt không cần được giữ bí mật, nhưng phải ngẫu nhiên và không được dùng lại giữa tất cả người dùng.

**Tác dụng của việc thêm salt**:

1. Tăng độ phức tạp và tính duy nhất của mật khẩu.
2. Khiến rainbow table attack mất tác dụng (salt của mỗi người dùng đều khác nhau).
3. Ngay cả khi hai người dùng sử dụng cùng một mật khẩu, hash value cũng khác nhau.

## Khuyến nghị về phương án lưu trữ mật khẩu

Mật khẩu nên được xử lý bằng algorithm được thiết kế riêng cho việc lưu trữ mật khẩu và có thể điều chỉnh chi phí tính toán, thay vì trực tiếp sử dụng các hash algorithm tốc độ cao như MD5, SHA-256, SHA-3. Ngay cả khi thêm salt cho hash tốc độ cao, attacker lấy được database vẫn có thể thử một lượng lớn mật khẩu ứng viên với tốc độ cao.

Hệ thống mới nên ưu tiên **Argon2id**. Nếu không khả dụng, có thể chọn scrypt tùy theo môi trường runtime; khi cần tương thích với hệ thống legacy có thể sử dụng Bcrypt với cấu hình hợp lý; khi có yêu cầu tuân thủ FIPS có thể sử dụng PBKDF2. Các parameter cụ thể cần được đánh giá và nâng cấp định kỳ dựa trên performance của server.

### Ví dụ Bcrypt

**Bcrypt** là hash algorithm được thiết kế riêng cho việc lưu trữ mật khẩu, thuộc loại slow hash algorithm. Nó tích hợp cơ chế salt và parameter cost:

- **salt**: Chuỗi được tạo ngẫu nhiên, dùng để trộn với mật khẩu nhằm tăng tính duy nhất của mật khẩu.
- **cost**: Kiểm soát số lần lặp, làm tăng thời gian tính toán và mức tiêu thụ resource.

Salt ngẫu nhiên của Bcrypt có thể ngăn chặn precomputation và rainbow table attack, parameter cost có thể tăng chi phí đoán mật khẩu offline, nhưng không thể khiến mật khẩu yếu trở nên không thể crack. Ngoài ra cần lưu ý, phần lớn implementation của Bcrypt chỉ xử lý 72 byte đầu tiên của mật khẩu; hệ thống không được tự động cắt ngắn mật khẩu một cách im lặng mà không thông báo.

Spring Security cung cấp `BCryptPasswordEncoder`. Dưới đây dùng nó để minh họa cách thiết lập cost một cách rõ ràng; việc chọn lựa cho hệ thống mới vẫn nên ưu tiên đánh giá Argon2id:

```java
@Bean
public PasswordEncoder passwordEncoder(){
    // cost nên được xác định thông qua performance test và điều chỉnh định kỳ khi năng lực phần cứng tăng lên.
    return new BCryptPasswordEncoder(12);
}
```

## Quy trình xác minh khi đăng nhập

Khi bạn nhập mật khẩu để đăng nhập, quy trình xác minh như sau:

1. Server lấy password hash encoding được lưu của người dùng từ database dựa trên username. Encoding này thường đã chứa algorithm identifier, parameter và salt ngẫu nhiên.
2. Server gọi phương thức xác minh do password hash library cung cấp, chẳng hạn `PasswordEncoder#matches` của Spring Security. Không được tự ghép salt và không được trực tiếp so sánh string.
3. Password library đọc salt và parameter trong encoding, thực hiện cùng phép tính với input của người dùng, rồi so sánh kết quả theo cách an toàn.
4. Nếu xác minh thành công thì mật khẩu đúng; nếu không thì mật khẩu sai. Sau khi xác minh thành công, còn có thể tính toán lại và nâng cấp password hash khi parameter đã quá cũ.

## Khi đặt lại mật khẩu, làm thế nào để biết mật khẩu mới có giống mật khẩu cũ không?

Có thể bạn đã nhận thấy một số website khi đặt lại mật khẩu sẽ thông báo "mật khẩu mới không được giống mật khẩu cũ". Vậy website làm thế nào để biết mật khẩu mới và mật khẩu cũ giống nhau?

Thực ra nguyên lý giống với việc xác minh mật khẩu có chính xác hay không:

1. Người dùng nhập mật khẩu mới.
2. Server gọi phương thức xác minh của password hash library, dùng mật khẩu mới để xác minh password hash cũ trong database, chẳng hạn `passwordEncoder.matches(newPassword, oldPasswordHash)`.
3. Nếu xác minh thành công thì mật khẩu mới và mật khẩu cũ giống nhau, từ chối thay đổi.
4. Nếu không giống nhau thì tạo lại salt ngẫu nhiên và password hash cho mật khẩu mới, không được dùng lại hash cũ hoặc tự cố định salt.

Vì vậy website không biết mật khẩu cũ của bạn là gì, mà chỉ so sánh xem hai đĩa "khoai tây sợi" có giống nhau hay không.

## Bảo mật khi truyền mật khẩu

Phần trước nói về bảo mật lưu trữ mật khẩu ở server, vậy mật khẩu có an toàn trong quá trình truyền không?

Có một câu hỏi phỏng vấn thường gặp: **Nếu một nhân viên biết cách mã hóa, chẳng phải người đó có thể lén chặn packet sau đó hoặc sau khi nghỉ việc, rồi mô phỏng việc mã hóa để lấy mật khẩu sao?**

Câu trả lời là: **lưu trữ và truyền dữ liệu vốn được xử lý riêng biệt**.

Một phương án bảo mật mật khẩu hoàn chỉnh cần đồng thời đảm bảo bảo mật lưu trữ và bảo mật truyền dữ liệu.

### Sử dụng HTTPS

HTTPS là nền tảng đảm bảo an toàn truyền dữ liệu. HTTP chạy trên TCP, toàn bộ nội dung truyền đi đều ở dạng plaintext, client và server đều không thể xác minh danh tính của đối phương. HTTPS là HTTP chạy trên SSL/TLS, toàn bộ nội dung truyền đi đều được mã hóa.

Có thể xem bài viết này để so sánh chi tiết HTTP và HTTPS: [HTTP vs HTTPS (application layer)](https://javaguide.cn/cs-basics/network/http-vs-https.html).

Đối với ứng dụng Web thông thường, HTTPS được cấu hình đúng là phương án nền tảng để bảo mật truyền mật khẩu. Server nên mặc định sử dụng TLS 1.3 và hỗ trợ TLS 1.2 khi cần tương thích; bắt buộc sử dụng HTTPS trên toàn site, bật HSTS, xác minh certificate đúng cách và vô hiệu hóa protocol lỗi thời cùng cipher suite yếu.

Việc browser sử dụng thêm một lớp mã hóa RSA tự định nghĩa thường không giải quyết được vấn đề client độc hại, frontend script bị xâm nhập hoặc password bị lộ tại điểm giải mã của server, mà ngược lại còn làm tăng rủi ro về phân phối key, lựa chọn padding và replay ciphertext. Vì vậy, không nên xem "client RSA + HTTPS" là một phương án chung bắt buộc phải áp dụng cho mọi hệ thống.

Một số hệ thống có yêu cầu tuân thủ rõ ràng hoặc threat model đặc biệt có thể bổ sung lớp bảo vệ application trên TLS, nhưng nên sử dụng protocol trưởng thành đã được review, đồng thời phải bao gồm random challenge, kiểm tra thời hạn và cơ chế chống replay; không thể chỉ thực hiện một lần public key encryption đơn giản.

Ngoài mã hóa khi truyền, còn cần giới hạn login attempt, tránh ghi log mật khẩu, sử dụng multi-factor authentication và phòng chống credential stuffing cùng credential attack.

## Cần lưu ý gì trong quy trình quên mật khẩu?

Trọng tâm của bài viết này là giải thích vì sao server không thể khôi phục mật khẩu cũ. Khi thực sự triển khai chức năng quên mật khẩu, còn cần chú ý các yêu cầu bảo mật sau:

- Dù tài khoản có tồn tại hay không, đều trả về thông báo nhất quán và cố gắng giữ response time gần tương đương để tránh user enumeration.
- Reset token được tạo bằng cryptographically secure random number generator, có entropy đủ lớn, chỉ được sử dụng một lần và hết hạn sau thời gian ngắn.
- Rate limit request reset và việc kiểm tra token; link reset chỉ sử dụng trusted domain và HTTPS, tránh để token bị lộ qua Referer.
- Sau khi đổi mật khẩu thành công, gửi security notification và tùy theo risk mà vô hiệu hóa các session hiện có, hoặc ít nhất cho phép người dùng logout các session khác chỉ bằng một lần nhấn.

## Tổng kết

Quay lại vấn đề ban đầu: vì sao khi quên mật khẩu chỉ có thể đặt lại mà không thể cho biết mật khẩu cũ?

Vì server lưu value đã được xử lý bằng hash algorithm của mật khẩu, **hash algorithm không thể đảo ngược**, không thể khôi phục mật khẩu ban đầu từ hash value. Đây là nguyên tắc cơ bản của bảo mật mật khẩu.

Nếu một website có thể trực tiếp cho biết mật khẩu cũ, điều đó cho thấy server đã lưu mật khẩu dưới dạng **plaintext hoặc dạng có thể đảo ngược**, thay vì chỉ lưu password hash chuyên dụng. Đây là một rủi ro bảo mật nghiêm trọng; bạn nên lập tức đổi mật khẩu và kiểm tra xem các website khác có dùng lại cùng mật khẩu hay không.

**Quan trọng hơn**: nếu bạn sử dụng cùng một mật khẩu trên mọi website, việc một website không đáng tin làm lộ mật khẩu cũng tương đương với việc mọi tài khoản của bạn đều gặp rủi ro. Vì vậy, **không được sử dụng cùng một mật khẩu trên mọi website**!

## Tham khảo

- OWASP Password Storage Cheat Sheet: <https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html>
- OWASP Forgot Password Cheat Sheet: <https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html>
- OWASP Transport Layer Security Cheat Sheet: <https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html>
