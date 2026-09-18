---
title: Tổng hợp các thuật toán mã hóa thường gặp
description: Giải thích chi tiết các thuật toán mã hóa thường gặp, bao quát các thuật toán mã hóa đối xứng và bất đối xứng như AES, RSA cùng nguyên lý và trường hợp sử dụng của các thuật toán hash như MD5, SHA.
category: System Design
tag:
  - Security
  - Hash Algorithms
head:
  - - meta
    - name: keywords
      content: thuật toán mã hóa,AES,RSA,thuật toán hash,thuật toán digest,HTTPS,mã hóa đối xứng,mã hóa bất đối xứng,BCrypt
---

Thuật toán mã hóa là kỹ thuật dùng phương pháp toán học để biến đổi dữ liệu, nhằm bảo vệ an toàn dữ liệu và ngăn người không được ủy quyền đọc hoặc sửa đổi dữ liệu. Thuật toán mã hóa có thể chia thành ba loại chính: thuật toán mã hóa đối xứng, thuật toán mã hóa bất đối xứng và thuật toán hash (còn gọi là thuật toán digest).

Các trường hợp thường gặp cần dùng thuật toán mã hóa trong phát triển hằng ngày:

1. Mật khẩu lưu trong database cần được thêm salt rồi dùng thuật toán hash (ví dụ BCrypt) để mã hóa.
2. Các dữ liệu nhạy cảm như số thẻ ngân hàng, số định danh lưu trong database cần dùng thuật toán mã hóa đối xứng (ví dụ AES) để lưu trữ.
3. Dữ liệu nhạy cảm truyền qua network như số thẻ ngân hàng, số định danh cần dùng HTTPS + thuật toán mã hóa bất đối xứng (như RSA) để bảo đảm an toàn dữ liệu truyền.
4. ……

ps: Nói chính xác thì thuật toán hash thực ra không thuộc thuật toán mã hóa, mà chỉ có thể được dùng trong một số trường hợp mã hóa (ví dụ mã hóa mật khẩu); hai loại này có thể xem là quan hệ song song. Thuật toán mã hóa thường chỉ thuật toán có thể chuyển plaintext thành ciphertext, đồng thời khôi phục ciphertext về plaintext bằng một cách nào đó (như dùng key). Còn thuật toán hash là một quá trình một chiều, chuyển thông tin đầu vào thành một giá trị hash có độ dài cố định và trông như ngẫu nhiên, nhưng quá trình này không thể đảo ngược, nghĩa là không thể khôi phục thông tin gốc từ giá trị hash.

## Thuật toán hash

Thuật toán hash còn được gọi là hàm băm hoặc thuật toán digest. Nó có tác dụng tạo một định danh duy nhất có độ dài cố định cho dữ liệu có độ dài bất kỳ, còn gọi là giá trị hash, giá trị băm hoặc message digest (sau đây gọi chung là giá trị hash).

![Minh họa hiệu quả của thuật toán hash](https://oss.javaguide.cn/github/javaguide/system-design/security/encryption-algorithms/hash-function-effect-demonstration.png)

Thuật toán hash là không thể đảo ngược, bạn không thể lấy lại giá trị ban đầu từ giá trị sau khi hash.

Giá trị hash có thể được dùng để kiểm tra tính toàn vẹn và tính nhất quán của dữ liệu.

Hai ví dụ thực tế:

- Khi lưu mật khẩu vào database, dùng thuật toán hash để mã hóa, sau đó so sánh giá trị hash của mật khẩu người dùng nhập với giá trị hash lưu trong database để xác định mật khẩu có chính xác hay không.
- Khi tải một file, có thể so sánh giá trị hash của file với giá trị hash do bên cung cấp chính thức đưa ra để xác định file có bị sửa đổi hoặc hỏng hay không;

Đặc điểm của thuật toán này là không thể đảo ngược:

- Không thể khôi phục dữ liệu gốc từ giá trị hash.
- Mọi thay đổi của dữ liệu gốc đều làm giá trị hash thay đổi rất lớn.

Thuật toán hash có thể chia đơn giản thành hai loại:

1. **Thuật toán hash mật mã**: thuật toán hash có độ an toàn cao hơn, có thể cung cấp một mức độ bảo vệ tính toàn vẹn và chống giả mạo dữ liệu nhất định, chống lại một số phương thức tấn công, độ an toàn tương đối cao nhưng performance kém hơn, phù hợp với các trường hợp yêu cầu độ an toàn cao. Ví dụ: SHA2, SHA3, SM3, RIPEMD-160, BLAKE2, v.v.
2. **Thuật toán hash không dùng cho mật mã**: thuật toán hash có độ an toàn tương đối thấp, dễ chịu ảnh hưởng của các phương thức tấn công như brute-force, collision attack, nhưng performance cao hơn, phù hợp với các nghiệp vụ không yêu cầu an toàn. Ví dụ: CRC32, MurMurHash3, v.v.

Ngoài hai loại này còn có một số thuật toán hash đặc biệt, chẳng hạn **thuật toán slow hash** có độ an toàn cao hơn.

Một số thuật toán hash thường gặp:

- MD (Message Digest, thuật toán message digest): MD2, MD4, MD5, v.v.; không còn được khuyến nghị sử dụng.
- SHA (Secure Hash Algorithm, thuật toán hash an toàn): dòng SHA-1 có độ an toàn thấp, dòng SHA2 và SHA3 có độ an toàn cao hơn.
- Thuật toán mật mã nội địa: ví dụ SM2, SM3, SM4; trong đó SM2 là thuật toán mã hóa bất đối xứng, SM4 là thuật toán mã hóa đối xứng, SM3 là thuật toán hash (độ an toàn và hiệu năng tương đương SHA-256 nhưng phù hợp hơn với môi trường ứng dụng trong nước).
- Bcrypt (thuật toán hash mật khẩu): thuật toán hash mật khẩu dựa trên thuật toán mã hóa Blowfish, được thiết kế riêng cho mã hóa mật khẩu, có độ an toàn cao và thuộc loại slow hash.
- MAC (Message Authentication Code, thuật toán message authentication code): HMAC là một MAC dựa trên hash, có thể kết hợp với bất kỳ thuật toán hash an toàn nào, ví dụ SHA-256.
- CRC (Cyclic Redundancy Check, kiểm tra dư thừa vòng): CRC32 là một thuật toán CRC, tạo ra giá trị kiểm tra 32 bit, thường dùng trong các trường hợp kiểm tra tính toàn vẹn dữ liệu, kiểm tra file, v.v.
- SipHash: đây không phải là hàm hash mã hóa không có key truyền thống (như SHA-256), mà là PRF (Pseudo-Random Function) có key. Phải kết hợp với một key ngẫu nhiên thì mới thực sự có khả năng chống collision attack. Nó được thiết kế để đạt cân bằng giữa tốc độ và độ an toàn, dùng để phòng thủ trước [hash flooding DoS attack](https://aumasson.jp/siphash/siphashdos_29c3_slides.pdf). Rust mặc định dùng SipHash làm thuật toán hash (hiện là SipHash-1-3); từ Redis 4.0, thuật toán hash của dictionary (dict) đã chuyển từ MurmurHash2 ban đầu sang SipHash (hiện là SipHash-1-2).
- MurMurHash: thuật toán hash không dùng cho mật mã nhanh và kinh điển; phiên bản mới nhất hiện nay là MurMurHash3, có thể tạo giá trị hash 32 bit hoặc 128 bit;
- ……

Thuật toán hash thường không cần key, nhưng cũng có một số thuật toán hash đặc biệt cần key. Ví dụ, MAC và SipHash là các thuật toán hash dựa trên key; chúng thêm một key vào nền tảng thuật toán hash, nhờ đó chỉ người biết key mới có thể xác minh tính toàn vẹn và nguồn gốc dữ liệu.

### MD

Thuật toán MD có nhiều phiên bản, gồm MD2, MD4, MD5, trong đó MD5 là phiên bản được dùng phổ biến nhất và có thể tạo giá trị hash 128 bit (16 byte). Về độ an toàn: MD5 > MD4 > MD2. Ngoài các phiên bản này còn có một số thuật toán cải tiến dựa trên MD4 hoặc MD5, như RIPEMD, HAVAL, v.v.

Ngay cả thuật toán MD an toàn nhất là MD5 cũng có nguy cơ bị phá, kẻ tấn công có thể dùng brute-force hoặc rainbow table attack để tìm giá trị hash giống dữ liệu gốc, từ đó phá dữ liệu.

Để tăng độ khó khi phá, thông thường có thể thêm salt. Trong cryptography, salt là một chuỗi đặc biệt được chèn vào vị trí cố định bất kỳ trong mật khẩu, khiến kết quả sau khi hash khác với kết quả hash từ mật khẩu gốc. Quá trình này gọi là thêm salt.

Thêm salt rồi thì có an toàn không? Không hẳn. Điều này chỉ làm tăng độ khó khi phá, không có nghĩa là không thể phá. Ngoài ra, bản thân thuật toán MD5 tồn tại vấn đề weak collision, tức nhiều input khác nhau tạo ra cùng một giá trị MD5.

Vì vậy, thuật toán MD không còn được khuyến nghị sử dụng; nên dùng các thuật toán hash an toàn hơn như SHA-2, Bcrypt.

Java cung cấp hỗ trợ cho dòng thuật toán MD, gồm MD2, MD5.

Ví dụ code MD5 (chưa thêm salt):

```java
String originalString = "Java Learning + Interview Guide: javaguide.cn";
// Tạo đối tượng digest MD5
MessageDigest messageDigest = MessageDigest.getInstance("MD5");
messageDigest.update(originalString.getBytes(StandardCharsets.UTF_8));
// Tính giá trị hash
byte[] result = messageDigest.digest();
// Chuyển giá trị hash thành chuỗi thập lục phân
String hexString = new HexBinaryAdapter().marshal(result);
System.out.println("Original String: " + originalString);
System.out.println("MD5 Hash: " + hexString.toLowerCase());
```

Output:

```bash
Original String: Java Learning + Interview Guide: javaguide.cn
MD5 Hash: fb246796f5b1b60d4d0268c817c608fa
```

### SHA

Dòng thuật toán SHA (Secure Hash Algorithm) là một nhóm thuật toán hash mật mã, dùng để ánh xạ dữ liệu có độ dài bất kỳ thành giá trị hash có độ dài cố định. Dòng thuật toán SHA do Cơ quan An ninh Quốc gia Hoa Kỳ (NSA) thiết kế vào năm 1993, hiện có ba phiên bản: SHA-1, SHA-2, SHA-3.

Thuật toán SHA-1 ánh xạ dữ liệu có độ dài bất kỳ thành giá trị hash 160 bit. Tuy nhiên, thuật toán SHA-1 có một số thiếu sót nghiêm trọng như độ an toàn thấp, dễ chịu collision attack và length extension attack. Vì vậy, thuật toán SHA-1 không còn được khuyến nghị sử dụng. Họ SHA-2 (như SHA-256, SHA-384, SHA-512, v.v.) và dòng SHA-3 là các phương án thay thế cho SHA-1, đều cung cấp độ an toàn cao hơn và giá trị hash dài hơn.

Họ SHA-2 được cải tiến dựa trên thuật toán SHA-1, sử dụng quá trình tính toán phức tạp hơn và nhiều round hơn, khiến kẻ tấn công khó tìm được collision hơn thông qua tính toán trước hoặc tình cờ.

Để tìm một thuật toán hash mật mã an toàn và tiên tiến hơn, National Institute of Standards and Technology (viết tắt là NIST) của Hoa Kỳ đã công khai kêu gọi các thuật toán ứng viên cho SHA-3 vào năm 2007. NIST nhận được tổng cộng 64 phương án thuật toán. Sau nhiều vòng đánh giá và sàng lọc, năm 2012 NIST công bố thuật toán Keccak chiến thắng và trở thành thuật toán tiêu chuẩn của SHA-3 (SHA-3 không có quan hệ trực tiếp với SHA-2). Thuật toán Keccak có cách thiết kế hoàn toàn khác MD và SHA-1/2, đó là Sponge Construction, khiến các phương thức tấn công truyền thống không thể áp dụng trực tiếp vào tấn công SHA-3 (có thể chống tất cả phương thức tấn công đã biết hiện nay, gồm collision attack, length extension attack, differential attack, v.v.).

Do thuật toán SHA-2 chưa xuất hiện lỗ hổng an toàn nghiêm trọng và có performance trong phần mềm cao hơn, phần lớn mọi người vẫn có xu hướng sử dụng thuật toán SHA-2.

So với thuật toán MD5, thuật toán SHA-2 mạnh hơn chủ yếu vì hai lý do:

- Độ dài giá trị hash lớn hơn: ví dụ giá trị hash của thuật toán SHA-256 dài 256 bit, còn MD5 dài 128 bit, làm tăng độ khó để kẻ tấn công brute-force hoặc rainbow table attack.
- Khả năng chống collision mạnh hơn: thuật toán SHA sử dụng quá trình tính toán phức tạp hơn và nhiều round hơn, khiến kẻ tấn công khó tìm collision hơn thông qua tính toán trước hoặc tình cờ. Hiện chưa tìm thấy hai dữ liệu khác nhau nào có cùng giá trị hash SHA-256.

Tất nhiên, SHA-2 cũng không an toàn tuyệt đối và có nguy cơ bị brute-force hoặc rainbow table attack. Vì vậy, trong ứng dụng thực tế, thêm salt vẫn là điều không thể thiếu.

Java cung cấp hỗ trợ cho dòng thuật toán SHA, gồm SHA-1, SHA-256, SHA-384 và SHA-512.

Ví dụ code SHA-256 (chưa thêm salt):

```java
String originalString = "Java Learning + Interview Guide: javaguide.cn";
// Tạo đối tượng digest SHA-256
MessageDigest messageDigest = MessageDigest.getInstance("SHA-256");
messageDigest.update(originalString.getBytes());
// Tính giá trị hash
byte[] result = messageDigest.digest();
// Chuyển giá trị hash thành chuỗi thập lục phân
String hexString = new HexBinaryAdapter().marshal(result);
System.out.println("Original String: " + originalString);
System.out.println("SHA-256 Hash: " + hexString.toLowerCase());
```

Output:

```bash
Original String: Java Learning + Interview Guide: javaguide.cn
SHA-256 Hash: 184eb7e1d7fb002444098c9bde3403c6f6722c93ecfac242c0e35cd9ed3b41cd
```

### Bcrypt

Thuật toán Bcrypt là thuật toán hash mật khẩu dựa trên thuật toán mã hóa Blowfish, được thiết kế riêng cho mã hóa mật khẩu và có độ an toàn cao.

Do Bcrypt sử dụng hai cơ chế salt và cost, nó có thể ngăn chặn hiệu quả rainbow table attack và brute-force attack, từ đó bảo đảm an toàn mật khẩu. salt là một chuỗi được tạo ngẫu nhiên, dùng để trộn với mật khẩu nhằm tăng độ phức tạp và tính duy nhất của mật khẩu. cost là một tham số số, dùng để kiểm soát số lần lặp của thuật toán Bcrypt, làm tăng thời gian tính toán và tài nguyên tiêu thụ khi hash mật khẩu.

Thuật toán Bcrypt có thể điều chỉnh độ phức tạp mã hóa tùy tình hình thực tế, thiết lập các giá trị cost và salt khác nhau để đáp ứng các yêu cầu an toàn khác nhau, có tính linh hoạt cao.

Spring Security, framework bảo mật của ứng dụng Java, hỗ trợ nhiều password encoder. Trong đó, `BCryptPasswordEncoder` là một encoder được khuyến nghị chính thức, sử dụng thuật toán BCrypt để mã hóa và lưu trữ mật khẩu người dùng.

```java
@Bean
public PasswordEncoder passwordEncoder(){
    return new BCryptPasswordEncoder();
}
```

## Mã hóa đối xứng

Thuật toán mã hóa đối xứng là thuật toán sử dụng cùng một key để mã hóa và giải mã, còn gọi là thuật toán mã hóa shared key.

![Mã hóa đối xứng](https://oss.javaguide.cn/github/javaguide/system-design/security/encryption-algorithms/symmetric-encryption.png)

Các thuật toán mã hóa đối xứng thường gặp gồm DES, 3DES, AES, v.v.

### DES và 3DES

DES (Data Encryption Standard) sử dụng key 64 bit (độ dài key hiệu dụng là 56 bit, 8 bit là bit parity) để mã hóa plaintext 64 bit.

Mặc dù DES mỗi lần chỉ có thể mã hóa 64 bit, nhưng chỉ cần chia plaintext thành các block 64 bit là có thể mã hóa plaintext có độ dài bất kỳ. Nếu độ dài plaintext không phải bội số của 64 bit thì phải padding; các mode thường dùng gồm PKCS5Padding, PKCS7Padding, NOPADDING.

Ý tưởng cơ bản của thuật toán mã hóa DES là chia plaintext 64 bit thành hai nửa, thực hiện nhiều round biến đổi trên mỗi nửa rồi ghép lại thành ciphertext 64 bit. Các biến đổi này gồm permutation, XOR, selection, shift, v.v. Mỗi round sử dụng một subkey, các subkey này đều được tạo từ cùng một master key 56 bit. Thuật toán mã hóa DES thực hiện tổng cộng 16 round biến đổi, sau đó thực hiện thêm một inverse permutation để thu được ciphertext cuối cùng.

![DES (Data Encryption Standard)](https://oss.javaguide.cn/github/javaguide/system-design/security/des-steps.jpg)

Đây là một thuật toán mã hóa đối xứng kinh điển nhưng cũng có thiếu sót rõ ràng: key 56 bit không đủ an toàn và đã được chứng minh là có thể bị phá trong thời gian ngắn.

Để tăng độ an toàn của thuật toán DES, người ta đưa ra một số biến thể hoặc phương án thay thế, chẳng hạn 3DES (Triple DES).

3DES (Triple DES) là thuật toán mã hóa chuyển tiếp từ DES sang AES, sử dụng 2 hoặc 3 key 56 bit để mã hóa dữ liệu ba lần. 3DES tương đương với việc áp dụng thuật toán mã hóa đối xứng DES ba lần cho mỗi block dữ liệu.

Để tương thích với DES thông thường, 3DES không trực tiếp dùng cách mã hóa -> mã hóa -> mã hóa, mà dùng cách mã hóa -> giải mã -> mã hóa. Khi cả ba key giống nhau, hai bước đầu triệt tiêu lẫn nhau, tương đương chỉ thực hiện một lần mã hóa, nhờ đó có thể tương thích với thuật toán mã hóa DES thông thường. 3DES an toàn hơn DES nhưng tốc độ xử lý không cao.

### AES

Thuật toán AES (Advanced Encryption Standard) là một thuật toán mã hóa symmetric key tiên tiến hơn, sử dụng key 128 bit, 192 bit hoặc 256 bit để mã hóa hoặc giải mã dữ liệu; key càng dài thì độ an toàn càng cao.

AES cũng là một block cipher, độ dài block chỉ có thể là 128 bit, nghĩa là mỗi block có 16 byte. Thuật toán mã hóa AES có nhiều mode of operation như ECB, CBC, OFB, CFB, CTR, XTS, OCB, GCM (mode được sử dụng rộng rãi nhất hiện nay). Tham số và quy trình mã hóa của các mode khác nhau, nhưng phần lõi vẫn là thuật toán AES.

Tương tự DES, một số mode của AES cần padding plaintext không phải bội số của 128 bit. Tuy nhiên, GCM là mode authenticated encryption (AEAD) được xây dựng dựa trên block cipher, có thể xử lý plaintext độ dài bất kỳ, vì vậy trong Java thường dùng `AES/GCM/NoPadding`. GCM vừa cung cấp tính bí mật vừa kiểm tra tính toàn vẹn của ciphertext, nhưng yêu cầu IV (Nonce) dưới cùng một key không được lặp lại.

AES nhanh hơn 3DES và an toàn hơn.

![AES (Advanced Encryption Standard)](https://oss.javaguide.cn/github/javaguide/system-design/security/aes-steps.jpg)

So sánh đơn giản thuật toán DES và AES (hình ảnh từ: [RSA vs. AES Encryption: Key Differences Explained](https://cheapsslweb.com/blog/rsa-vs-aes-encryption)):

![So sánh DES và AES](https://oss.javaguide.cn/github/javaguide/system-design/security/des-vs-aes.png)

Ví dụ code triển khai AES-GCM dựa trên Java. Ví dụ mã hóa IV được tạo ngẫu nhiên mỗi lần cùng với ciphertext; key AES trong môi trường production nên được KMS, HSM hoặc KeyStore tạo và bảo quản, không hard-code trong source code:

```java
private static final String AES_TRANSFORMATION = "AES/GCM/NoPadding";
private static final int GCM_IV_LENGTH = 12;
private static final int GCM_TAG_LENGTH = 128;
private static final SecureRandom SECURE_RANDOM = new SecureRandom();

/**
 * Mã hóa
 */
public static String encrypt(String data, SecretKey secretKey) throws GeneralSecurityException {
    byte[] iv = new byte[GCM_IV_LENGTH];
    SECURE_RANDOM.nextBytes(iv);

    Cipher cipher = Cipher.getInstance(AES_TRANSFORMATION);
    cipher.init(Cipher.ENCRYPT_MODE, secretKey, new GCMParameterSpec(GCM_TAG_LENGTH, iv));
    byte[] encryptedBytes = cipher.doFinal(data.getBytes(StandardCharsets.UTF_8));

    // IV không cần giữ bí mật, nhưng khi giải mã phải dùng cùng IV, nên lưu IV cùng ciphertext.
    ByteBuffer byteBuffer = ByteBuffer.allocate(iv.length + encryptedBytes.length);
    byteBuffer.put(iv);
    byteBuffer.put(encryptedBytes);
    return Base64.getEncoder().encodeToString(byteBuffer.array());
}

/**
 * Giải mã
 */
public static String decrypt(String encryptedData, SecretKey secretKey) throws GeneralSecurityException {
    byte[] input = Base64.getDecoder().decode(encryptedData);
    int tagLengthInBytes = GCM_TAG_LENGTH / Byte.SIZE;
    if (input.length < GCM_IV_LENGTH + tagLengthInBytes) {
        throw new IllegalArgumentException("Invalid encrypted data");
    }

    ByteBuffer byteBuffer = ByteBuffer.wrap(input);
    byte[] iv = new byte[GCM_IV_LENGTH];
    byteBuffer.get(iv);
    byte[] encryptedBytes = new byte[byteBuffer.remaining()];
    byteBuffer.get(encryptedBytes);

    Cipher cipher = Cipher.getInstance(AES_TRANSFORMATION);
    cipher.init(Cipher.DECRYPT_MODE, secretKey, new GCMParameterSpec(GCM_TAG_LENGTH, iv));
    byte[] decryptedBytes = cipher.doFinal(encryptedBytes);
    return new String(decryptedBytes, StandardCharsets.UTF_8);
}

public static void main(String[] args) throws Exception {
    // Chỉ dùng để minh họa. Môi trường production nên lấy key từ KMS, HSM hoặc KeyStore.
    KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
    keyGenerator.init(256);
    SecretKey secretKey = keyGenerator.generateKey();

    String originalString = "Java Learning + Interview Guide: javaguide.cn";
    String encryptedData = encrypt(originalString, secretKey);
    String decryptedData = decrypt(encryptedData, secretKey);
    System.out.println("Original String: " + originalString);
    System.out.println("AES Encrypted Data : " + encryptedData);
    System.out.println("AES Decrypted Data : " + decryptedData);
}
```

Output:

```bash
Original String: Java Learning + Interview Guide: javaguide.cn
AES Encrypted Data : <Base64 string khác nhau mỗi lần chạy>
AES Decrypted Data : Java Learning + Interview Guide: javaguide.cn
```

## Mã hóa bất đối xứng

Thuật toán mã hóa bất đối xứng là thuật toán sử dụng các key khác nhau để mã hóa và giải mã, còn gọi là thuật toán mã hóa public key. Hai key này khác nhau, một key gọi là public key, key còn lại gọi là private key. Public key có thể công khai cho bất kỳ ai sử dụng, còn private key phải được giữ bí mật.

Nếu dùng public key để mã hóa dữ liệu thì chỉ private key tương ứng mới có thể giải mã. Digital signature là một loại thao tác khác: bên gửi dùng private key để tạo chữ ký, bên nhận dùng public key để xác minh chữ ký. Không nên đơn giản hiểu digital signature là "mã hóa bằng private key, giải mã bằng public key"; trong dự án thực tế cần lần lượt sử dụng encryption API và signature API.

![Mã hóa bất đối xứng](https://oss.javaguide.cn/github/javaguide/system-design/security/encryption-algorithms/asymmetric-encryption.png)

Các thuật toán public key thường gặp gồm RSA và các thuật toán dựa trên elliptic curve. Khả năng cụ thể của chúng khác nhau: RSA có thể dùng để mã hóa và ký; DSA chỉ dùng để ký; ECC là tên gọi chung của một loại thuật toán, bao gồm các scheme khác nhau dùng để ký hoặc key agreement.

### RSA

Thuật toán RSA (Rivest–Shamir–Adleman algorithm) là thuật toán mã hóa bất đối xứng dựa trên độ khó của bài toán phân tích thừa số số lớn. Nó cần chọn hai số nguyên tố lớn làm một phần của private key, sau đó tính tích của chúng làm một phần của public key (việc tìm hai số nguyên tố lớn tương đối đơn giản, còn phân tích tích của chúng thành thừa số lại cực kỳ khó). Để xem giới thiệu chi tiết về nguyên lý thuật toán RSA, có thể tham khảo bài viết [Bạn thực sự hiểu thuật toán mã hóa RSA không? - Tiểu Phó Ca](https://www.cnblogs.com/xiaofuge/p/16954187.html).

Độ an toàn của thuật toán RSA phụ thuộc vào độ khó của việc phân tích thừa số số lớn. Hiện đã có public key RSA 512 bit và 768 bit bị phân tích thành công, vì vậy nên sử dụng key có độ dài từ 2048 bit trở lên.

Ưu điểm của thuật toán RSA là đơn giản, dễ sử dụng, có thể dùng để mã hóa dữ liệu và digital signature; nhược điểm là tốc độ tính toán chậm, không phù hợp để mã hóa lượng dữ liệu lớn.

Thuật toán RSA hiện là thuật toán mã hóa bất đối xứng được sử dụng rộng rãi nhất; các protocol như SSL/TLS, SSH đều sử dụng thuật toán RSA.

![SHA-256 có mã hóa RSA trong thuật toán ký certificate HTTPS](https://oss.javaguide.cn/github/javaguide/system-design/security/encryption-algorithms/https-rsa-sha-256.png)

RSA có tốc độ tính toán chậm và độ dài dữ liệu có thể xử lý trực tiếp bị giới hạn, nên trong dự án thực tế thường dùng hybrid encryption: tạo ngẫu nhiên symmetric key, dùng AES-GCM để mã hóa dữ liệu nghiệp vụ, sau đó dùng RSA-OAEP để mã hóa symmetric key. Ví dụ dưới đây chỉ minh họa cách dùng RSA-OAEP để mã hóa dữ liệu ngắn:

```java
private static final String RSA_ALGORITHM = "RSA";
private static final String RSA_TRANSFORMATION = "RSA/ECB/OAEPPadding";
private static final OAEPParameterSpec OAEP_SHA_256 = new OAEPParameterSpec(
        "SHA-256",
        "MGF1",
        MGF1ParameterSpec.SHA256,
        PSource.PSpecified.DEFAULT
);

/**
 * Tạo cặp key RSA
 */
public static KeyPair generateKeyPair() throws NoSuchAlgorithmException {
    KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(RSA_ALGORITHM);
    // Kích thước key là 2048 bit
    keyPairGenerator.initialize(2048);
    return keyPairGenerator.generateKeyPair();
}

/**
 * Dùng public key để mã hóa dữ liệu
 */
public static String encrypt(String data, PublicKey publicKey) throws Exception {
    Cipher cipher = Cipher.getInstance(RSA_TRANSFORMATION);
    cipher.init(Cipher.ENCRYPT_MODE, publicKey, OAEP_SHA_256);
    byte[] encryptedData = cipher.doFinal(data.getBytes(StandardCharsets.UTF_8));
    return Base64.getEncoder().encodeToString(encryptedData);
}

/**
 * Dùng private key để giải mã dữ liệu
 */
public static String decrypt(String encryptedData, PrivateKey privateKey) throws Exception {
    byte[] decodedData = Base64.getDecoder().decode(encryptedData);
    Cipher cipher = Cipher.getInstance(RSA_TRANSFORMATION);
    cipher.init(Cipher.DECRYPT_MODE, privateKey, OAEP_SHA_256);
    byte[] decryptedData = cipher.doFinal(decodedData);
    return new String(decryptedData, StandardCharsets.UTF_8);
}

public static void main(String[] args) throws Exception {
    KeyPair keyPair = generateKeyPair();
    PublicKey publicKey = keyPair.getPublic();
    PrivateKey privateKey = keyPair.getPrivate();
    String originalString = "Java Learning + Interview Guide: javaguide.cn";
    String encryptedData = encrypt(originalString, publicKey);
    String decryptedData = decrypt(encryptedData, privateKey);
    System.out.println("Original String: " + originalString);
    System.out.println("RSA Encrypted Data : " + encryptedData);
    System.out.println("RSA Decrypted Data : " + decryptedData);
}
```

Output:

```bash
Original String: Java Learning + Interview Guide: javaguide.cn
RSA Encrypted Data : <Base64 string khác nhau mỗi lần chạy>
RSA Decrypted Data : Java Learning + Interview Guide: javaguide.cn
```

### DSA

DSA (Digital Signature Algorithm) là một thuật toán digital signature, độ an toàn dựa trên bài toán logarit rời rạc. Nó chỉ dùng để tạo và xác minh digital signature, không dùng để mã hóa dữ liệu. Signature cũng không phải là "digest được mã hóa bằng private key, digest được giải mã bằng public key": bên gửi dùng private key và thuật toán signature để tạo signature, bên nhận dùng public key và thuật toán verification để phán đoán signature có hợp lệ hay không.

DSA chủ yếu được dùng để tương thích với hệ thống legacy. NIST FIPS 186-5 không còn phê duyệt việc dùng DSA để tạo digital signature mới, chỉ cho phép xác minh signature legacy được tạo trước khi tiêu chuẩn được triển khai. Hệ thống mới thường nên chọn RSA-PSS, ECDSA hoặc EdDSA theo yêu cầu của protocol và khả năng tương thích, đồng thời sử dụng `Signature` API do thư viện cryptography đã được kiểm chứng cung cấp.

## Tổng kết

Bài viết này giới thiệu ba loại thuật toán mã hóa: thuật toán hash, thuật toán mã hóa đối xứng và thuật toán mã hóa bất đối xứng.

- Thuật toán hash là kỹ thuật dùng phương pháp toán học để tạo định danh duy nhất có độ dài cố định cho dữ liệu, có thể dùng để kiểm tra tính toàn vẹn và tính nhất quán của dữ liệu. Các thuật toán hash thường gặp gồm MD, SHA, MAC, v.v.
- Thuật toán mã hóa đối xứng là thuật toán sử dụng cùng một key để mã hóa và giải mã, có thể dùng để bảo vệ tính an toàn và tính bí mật của dữ liệu. Các thuật toán mã hóa đối xứng thường gặp gồm DES, 3DES, AES, v.v.
- Public key cryptography sử dụng cặp public key và private key, có thể hỗ trợ mã hóa, digital signature hoặc key agreement, nhưng khả năng cụ thể phụ thuộc vào thuật toán. Ví dụ RSA có thể dùng để mã hóa và ký, còn DSA chỉ dùng để ký.

## Tham khảo

- NIST SP 800-38D - Recommendation for Block Cipher Modes of Operation: GCM and GMAC: <https://csrc.nist.gov/pubs/sp/800/38/d/final>
- NIST FIPS 186-5 - Digital Signature Standard: <https://csrc.nist.gov/pubs/fips/186-5/final>
- Java `Cipher` API: <https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/crypto/Cipher.html>
- Tìm hiểu sâu về perfect hash - Tencent Technology Engineering: <https://mp.weixin.qq.com/s/M8Wcj8sZ7UF1CMr887Puog>
- Cryptography thực tiễn dành cho developer (2) — hàm hash: <https://thiscute.world/posts/practical-cryptography-basics-2-hash/>
- Chuyến du hành an toàn kỳ diệu: thuật toán DSA: <https://zhuanlan.zhihu.com/p/347025157>
- Giới thiệu mã hóa AES-GCM: <https://juejin.cn/post/6844904122676690951>
- Java AES 256 GCM Encryption and Decryption Example | JCE Unlimited Strength: <https://www.javainterviewpoint.com/java-aes-256-gcm-encryption-and-decryption/>

<!-- @include: @article-footer.snippet.md -->
