---
title: Cách đánh giá năng lực kỹ thuật của programmer trong vòng phỏng vấn kỹ thuật đầu tiên
description: "Cách đánh giá năng lực kỹ thuật của programmer trong vòng phỏng vấn kỹ thuật đầu tiên: tổng hợp các khái niệm then chốt, vấn đề thường gặp và điểm thực hành xoay quanh kiến thức kỹ thuật và phỏng vấn, giúp bạn học tập và chuẩn bị phỏng vấn hiệu quả."
category: Tuyển tập bài viết kỹ thuật chất lượng cao
author: Cầm Thủy Ngọc
tag:
  - Phỏng vấn
head:
  - - meta
    - name: keywords
      content: phỏng vấn kỹ thuật,kỹ năng interviewer,đánh giá kỹ thuật,phương pháp phỏng vấn,nền tảng kỹ thuật,đánh giá kinh nghiệm dự án,ngân hàng câu hỏi phỏng vấn,chiều sâu kỹ thuật
---

> **Lời giới thiệu**: Bài viết thảo luận về phỏng vấn kỹ thuật từ cả góc độ interviewer và ứng viên! Rất đáng đọc!
>
> **Tổng quan nội dung:**
>
> - Kết hợp thực hành và lý thuyết. Ví dụ, sau khi ứng viên trình bày bố cục memory model của JVM, có thể hỏi tiếp: Những nguyên nhân nào có thể gây OOM, có biện pháp phòng ngừa nào? Bạn đã từng gặp vấn đề memory leak chưa? Làm thế nào để điều tra và giải quyết những vấn đề này?
> - Không nên đánh giá quá hai dự án. Vì để tìm hiểu sâu chi tiết một dự án sẽ tốn khá nhiều thời gian. Thông thường, ứng viên sẽ được yêu cầu chọn một dự án mà họ thấy thu hoạch được nhiều nhất/có tính thử thách nhất/ấn tượng sâu sắc nhất/thú vị nhất. Sau đó đặt câu hỏi xoay quanh dự án này. Thường bắt đầu từ bối cảnh dự án, đánh giá hiểu biết tổng thể về tech stack, module và tương tác của dự án, các vấn đề kỹ thuật có tính thử thách đã gặp trong dự án và solution, điều tra và giải quyết vấn đề, vấn đề maintainability của code, đảm bảo chất lượng engineering, v.v.
> - Hỏi nhiều, nói ít, để ứng viên thể hiện nhiều hơn. Dựa trên câu trả lời của ứng viên để dẫn dắt, hỏi sâu dần hoặc chuyển sang hướng khác một cách phù hợp.
>
> **Địa chỉ bài viết gốc**: <https://www.cnblogs.com/lovesqcc/p/15169365.html>

## Ba câu hỏi liên tiếp quan trọng

1. Bạn thấy con người này thế nào? [khả năng diễn đạt, khả năng giao tiếp, khả năng học tập, khả năng tổng kết, khả năng tự nhìn nhận và cải thiện, khả năng chịu áp lực, khả năng quản lý cảm xúc, sức ảnh hưởng, khả năng quản lý team]
2. Nếu để người này độc lập hoàn thành việc thiết kế và triển khai một dự án, bạn nghĩ họ có đảm nhiệm được không? [khả năng system design, khả năng quản lý dự án]
3. Bạn đánh giá thế nào về khả năng phân tích và giải quyết vấn đề của người này? [khả năng hiểu nguyên lý, khả năng ứng dụng thực tế]

## Mục tiêu và cách tư duy khi đánh giá

Trước hết cần xác định mục tiêu đánh giá của vòng phỏng vấn kỹ thuật đầu tiên:

- Nền tảng kỹ thuật của ứng viên;
- Tư duy và khả năng giải quyết vấn đề của ứng viên.

Nền tảng kỹ thuật là nền móng (phần nằm dưới tảng băng), chiếm bảy phần; tư duy và khả năng giải quyết vấn đề là khả năng đưa vào thực tế (phần lộ ra trên tảng băng), chiếm ba phần. Đánh giá nghiệp vụ và nền tảng kỹ thuật theo tỷ lệ ba-bảy.

Mục tiêu đánh giá cốt lõi: khả năng phân tích và giải quyết vấn đề.

Ở cấp độ kỹ thuật: chiều sâu + khả năng ứng dụng + độ rộng. Với ứng viên mới tốt nghiệp hoặc ứng viên tuyển dụng xã hội dưới cấp P6, nên chú trọng hơn vào chiều sâu + khả năng ứng dụng, độ rộng là điểm cộng; trên P6 có thể tăng mức chú trọng vào độ rộng.

- Ứng viên mới tốt nghiệp: nền tảng vững, tư duy nhanh nhạy. Nội dung đánh giá chính: data structure và algorithm cơ bản, process và concurrency, memory management, system call và cơ chế I/O, network protocol, database normalization và design, design pattern, design principle, thói quen lập trình;
- Ứng viên tuyển dụng xã hội: giàu kinh nghiệm, giỏi cả bên trong lẫn bên ngoài. Nội dung đánh giá chính: các cơ chế công nghệ nền tảng có chiều sâu nhất định, chẳng hạn Java memory model và memory leak, cơ chế JVM, cơ chế class loading, database index và query optimization, cache, message middleware, dự án, architecture design, engineering standard, v.v.

### Nền tảng kỹ thuật là gì?

Là interviewer kỹ thuật ở vòng đầu, làm thế nào để đánh giá nền tảng kỹ thuật? Rốt cuộc nền tảng kỹ thuật là gì? Là biết điều gì, hay biết cách tư duy? Kiến thức, với tư cách là một hệ thống nguyên lý hoàn thiện hiện có, cấu thành một phần quan trọng của nền tảng; còn biết cách tư duy cũng đặc biệt quan trọng. Như câu nói quen thuộc: biết nó là như vậy và biết vì sao nó lại như vậy. Biết nó là như vậy nghĩa là quen thuộc với hệ thống kiến thức hiện có; biết vì sao nó lại như vậy nghĩa là suy luận từ dưới lên, thực sự hiểu nguồn gốc và diễn biến của kiến thức, hiểu vì sao lại là như vậy chứ không phải như thế kia. Dù sao, đối với thế giới lập trình vốn có bản chất là logic, không có phương pháp cố định. Biết cách tư duy, có thể thiết kế và phát triển chặt chẽ, đi sâu vào chi tiết, đó chính là nền tảng kỹ thuật.

### Tại sao phải đánh giá nền tảng kỹ thuật?

Hai loại năng lực tư duy kỹ thuật quan trọng nhất của programmer là năng lực tư duy logic và năng lực thiết kế trừu tượng. Năng lực tư duy logic là nền tảng, năng lực thiết kế trừu tượng là cấp độ cao hơn. Đánh giá nền tảng kỹ thuật vừa có thể đánh giá đồng thời hai loại năng lực tư duy này. Có hiểu được các khái niệm kỹ thuật nền tảng và mối liên hệ giữa chúng hay không là đánh giá năng lực tư duy logic; có thể trừu tượng hóa vấn đề nghiệp vụ thành vấn đề kỹ thuật và tổ chức ánh xạ một cách hợp lý hay không là đánh giá năng lực thiết kế trừu tượng.

Phần lớn vấn đề nghiệp vụ đều có thể được trừu tượng hóa thành vấn đề kỹ thuật. Ở một ý nghĩa nào đó, vấn đề nghiệp vụ chỉ là cách diễn đạt mang tính lĩnh vực của vấn đề kỹ thuật.

Vì vậy, chỉ thông qua việc đánh giá nền tảng kỹ thuật của ứng viên mới có thể đánh giá được thực lực kỹ thuật thực sự của họ: chiều sâu và độ rộng kỹ thuật.

### Tại sao không chỉ đánh giá riêng khía cạnh nghiệp vụ?

Vì ứng viên thường khá quen thuộc với nghiệp vụ, có thể nói ra ngay theo solution hiện có, rất khó đánh giá được mức độ hiểu sâu, khả năng mở rộng theo chiều ngang và khả năng quy nạp tổng kết của ứng viên.

Ở điểm này, nên có chủ đích đánh giá khả năng quy nạp tổng kết của ứng viên: chẳng hạn, trong quá trình xây dựng, phát triển hoặc duy trì microservice/đảm bảo tính ổn định hoặc performance của hệ thống, bạn đã thu được những kinh nghiệm nào có thể chia sẻ?

### Tại sao phải đánh giá khía cạnh nghiệp vụ?

Điểm dễ bị bỏ qua khi đánh giá nền tảng kỹ thuật là những đặc điểm năng lực phi kỹ thuật của ứng viên, chẳng hạn khả năng tổ chức giao tiếp, khả năng dẫn dắt dự án, khả năng chịu áp lực, khả năng giải quyết vấn đề thực tế, sức ảnh hưởng trong team, các đặc điểm tính cách khác, v.v.

## Phương pháp đánh giá

### Đánh giá nền tảng kỹ thuật

Đánh giá nền tảng kỹ thuật thế nào? Thông qua cách hỏi hiệu quả, đa góc độ.

**Là gì - Tại sao**

“Là gì” đánh giá mức độ hiểu cơ bản về khái niệm, “tại sao” đánh giá nguyên lý triển khai của khái niệm.

Ví dụ: index là gì? Index được triển khai như thế nào?

**Dẫn dắt - hỏi theo chiều ngang - hỏi sâu**

Dẫn dắt, chẳng hạn “Bạn có quen với các công cụ synchronization của Java không?” để thăm dò. Sau khi nhận được câu trả lời khẳng định, có thể hỏi tiếp: “Bạn quen với những class công cụ synchronization nào?” để tìm hiểu độ rộng của ứng viên;

Sau khi nhận được câu trả lời của ứng viên, có thể hỏi tiếp: “Hãy nói về nguyên lý triển khai của ConcurrentHashMap hoặc AQS?”

Một người có thể trình bày nguyên lý kỹ thuật rõ ràng đến mức nào, bao gồm cả tư duy và chi tiết, cho thấy khả năng nắm vững kỹ thuật của người đó mạnh đến đâu.

**Hỏi theo từng mức độ và tầng bậc của chiều sâu**

Thiết lập câu hỏi ở ba tầng chiều sâu. Mỗi tầng chiều sâu có thể tương ứng với một mức độ sâu kỹ thuật.

- Câu hỏi thứ nhất ở tầng khái niệm cơ bản, đánh giá khả năng hiểu và chiều sâu hiểu khái niệm của ứng viên;
- Câu hỏi thứ hai ở tầng nguyên lý và cơ chế, đánh giá chiều sâu hiểu nội hàm và ngoại diên của khái niệm;
- Câu hỏi thứ ba ở tầng ứng dụng, đánh giá khả năng ứng dụng và mức độ nhanh nhạy trong tư duy của ứng viên.

**Hỏi theo kiểu nhảy cóc/đan chéo**

Ví dụ, khi nói đến việc tìm kiếm hiệu quả bằng hash, có thể nói về consistent hashing. Hai vấn đề vừa có liên quan vừa có nhiều điểm khác nhau. Đây cũng là một phương pháp đánh giá độ rộng kỹ thuật.

**Hỏi để tổng kết**

Ví dụ: Khi làm XXX, bạn thu được những kinh nghiệm nào có thể chia sẻ? Đánh giá khả năng quy nạp tổng kết của ứng viên.

**Kết hợp thực hành và lý thuyết**

- Ví dụ, sau khi ứng viên trình bày bố cục memory model của JVM, có thể hỏi tiếp: Những nguyên nhân nào có thể gây OOM, có biện pháp phòng ngừa nào? Bạn đã từng gặp vấn đề memory leak chưa? Làm thế nào để điều tra và giải quyết những vấn đề này?
- Ví dụ, nếu ứng viên đã đề cập đến SQL optimization và index optimization, thì có thể hỏi về nguyên lý triển khai của index và cách tạo index tối ưu;
- Ví dụ, nếu ứng viên đã đề cập đến transaction, thì có thể hỏi về nguyên lý triển khai transaction, isolation level, cách triển khai snapshot, v.v.;

**Kết hợp phần quen thuộc và phần không quen thuộc**

Hỏi cả những phần ứng viên ghi là quen thuộc trong CV và những phần không ghi. Ví dụ, nếu CV của ứng viên ghi: quen thuộc với JVM memory model, thì trước hết đánh giá về memory management (phần quen thuộc), sau đó đánh giá về Java concurrency utility class (phần chưa chắc quen thuộc).

**Kết hợp kiến thức chết và kiến thức sống**

Ví dụ: Có những search algorithm nào? Sequential search, binary search, hash search. Những điều này thường ai cũng nói ra được, đó cũng là “kiến thức chết”.

Những search algorithm này lần lượt phù hợp với trường hợp nào? Trong công việc của bạn, trường hợp nào đã sử dụng search algorithm nào? Tại sao? Đây là “kiến thức sống”.

**Những điều gặp phải trong học tập hoặc công việc**

Đôi khi, những vấn đề gặp phải trong học tập và công việc cũng có thể dùng làm câu hỏi phỏng vấn.

Ví dụ, gần đây khi học phần concurrency trong cuốn “Nhập môn hệ điều hành”, có một chương nói về cách làm cho data structure trở nên thread-safe. Ở đây có một số điểm có thể đặt câu hỏi: Làm thế nào để triển khai một lock? Làm thế nào để triển khai một counter thread-safe? Làm thế nào để triển khai một linked list thread-safe? Làm thế nào để triển khai một Map thread-safe? Làm thế nào để cải thiện performance của concurrency?

Những vấn đề gặp phải trong công việc cũng có thể được trừu tượng hóa, chắt lọc thành câu hỏi phỏng vấn về nền tảng kỹ thuật.

**Hỏi về mức độ phù hợp với tech stack**

Nếu một số công nghệ ứng viên sử dụng (được ghi trong CV) khá phù hợp với tech stack của công ty, có thể đặt câu hỏi chuyên sâu về những điểm công nghệ đó để đánh giá mức độ nắm vững của ứng viên. Nếu mức độ nắm vững khá tốt, mức độ phù hợp kỹ thuật sẽ tương đối cao hơn.

Tất nhiên, điều này không thể làm căn cứ để loại những ứng viên chưa sử dụng tech stack đó. Ví dụ, công ty sử dụng MongoDB và MySQL, còn một ứng viên chưa từng dùng Mongodb nhưng đã dùng nhiều hệ thống lưu trữ như MySQL, Redis, ES, HBase, thì mức độ phù hợp không hề kém hơn ứng viên chỉ từng dùng MySQL và Mongodb, vì độ rộng kỹ thuật mà họ tiếp xúc lớn hơn, có thể suy ra họ đủ khả năng nắm vững Mongodb.

**Ứng phó với phỏng vấn học thuộc câu hỏi**

Trước hết, phỏng vấn học thuộc câu hỏi cho thấy ít nhất ứng viên đã có chuẩn bị. Tất nhiên, phía tuyển dụng vẫn mong muốn tìm được ứng viên có năng lực chứ không chỉ ghi nhớ kiến thức.

Để ứng phó với phỏng vấn học thuộc câu hỏi, có thể dùng cách “dẫn dắt - hỏi theo chiều ngang - hỏi sâu”, trước hết tìm hiểu chiều sâu và độ rộng của ứng viên về một điểm kiến thức nào đó, sau đó đưa ra một bài toán ứng dụng thực tế để đánh giá họ có thể sử dụng kiến thức linh hoạt hay không.

Ví dụ, với cơ chế thread synchronization của Java, có thể đưa ra một bài toán: Thread A thực thi một đoạn code, sau đó tạo một task bất đồng bộ để thực thi trên thread B. Thread A cần đợi thread B thực thi xong mới có thể tiếp tục thực thi, hãy cho biết cách triển khai?

Mô hình “lý thuyết + bài toán ứng dụng”. Biết sự thay đổi của đối phương nhưng không biết hình thái thay đổi của ta. Hình thái thay đổi có vô số.

**Thực dụng, không đánh đố**

Đánh giá những kiến thức, kỹ năng và năng lực thường xuyên được sử dụng trong công việc, không đánh giá kiến thức hiếm gặp.

Ví dụ, tôi thiên về đánh giá ba loại: data structure và algorithm, concurrency, design. Vì cả ba đều rất nền tảng và cốt lõi.

**Hỏi theo kiểu liên kết tổng hợp**

Các kiến thức luôn liên hệ với nhau, không nên đánh giá riêng lẻ một điểm kiến thức.

Thiết kế một vấn đề khởi đầu, chẳng hạn search algorithm, rồi xuất phát từ vấn đề khởi đầu này để liên kết các điểm kiến thức. Ví dụ:

![](https://oss.javaguide.cn/github/javaguide/open-source-project/502996-20220211115505399-72788909.png)

Ở mỗi điểm kỹ thuật, đều có thể áp dụng các kỹ thuật hỏi ở trên để dẫn đến những nhánh vấn đề khác nhau. Đồng thời đánh giá chiều sâu, độ rộng và khả năng ứng dụng của ứng viên.

**Tạo ngân hàng câu hỏi phỏng vấn có tính cá nhân**

Mỗi interviewer kỹ thuật đều có một ngân hàng câu hỏi phỏng vấn. Hãy liên tục tích lũy ngân hàng câu hỏi phỏng vấn, những vấn đề bất chợt nghĩ ra trong ngày thường thì ghi lại ngay.

### Đánh giá khả năng giải quyết vấn đề

Chỉ có nền tảng kỹ thuật thôi vẫn chưa đủ. Thông thường tốt nhất nên kết hợp với nghiệp vụ thực tế, trừu tượng hóa vấn đề kỹ thuật từ nghiệp vụ trong dự án của ứng viên để đánh giá.

Trọng tâm của tư duy giải quyết vấn đề là tiến từng bước. Điều này cũng đòi hỏi khá cao ở interviewer, cần đồng thời có khả năng lắng nghe tốt, chiều sâu kỹ thuật và kinh nghiệm nghiệp vụ. Trước hết phải lắng nghe cẩn thận phần trình bày của ứng viên, tìm điểm bắt đầu kỹ thuật thích hợp rồi đặt câu hỏi. Nếu không đi vào được, việc đánh giá rất dễ thất bại.
Các vấn đề thường gặp:

- Về performance, qps, tps là bao nhiêu? Đã dùng biện pháp optimization nào, đạt được hiệu quả gì?
- Nếu có data volume lớn thì xử lý thế nào? Làm thế nào để đảm bảo tính ổn định?
- Bạn nghĩ điểm mấu chốt của function/module/system này nằm ở đâu? Có solution nào?
- Tại sao dùng XXX mà không dùng YYY?
- Làm thế nào để tạo index cho field dài?
- Còn solution hoặc hướng tư duy nào khác? Ưu nhược điểm của từng cách là gì?
- Khi tích hợp bên thứ ba, làm thế nào để ứng phó với sự không ổn định của external interface?
- Khi tích hợp bên thứ ba với số lượng lớn external system, làm thế nào đảm bảo maintainability của code?
- Có những scenario gây tổn thất tài sản? Scenario sự cố nghiêm trọng?
- Khi CPU tăng vọt trên production thì xử lý thế nào? Xử lý OOM ra sao? Khi I/O đọc ghi tăng đột biến thì điều tra thế nào?
- Trong quá trình chạy trên production đã từng xuất hiện vấn đề nào? Đã giải quyết thế nào?
- Vấn đề data consistency giữa nhiều subsystem?
- Nếu cần bổ sung một requirement XXX thì mở rộng thế nào?
- Nếu làm lại từ đầu, bạn nghĩ có thể cải thiện ở những phương diện nào?

Các vấn đề liên quan có thể hỏi về system:

- Phần lớn system đều có vấn đề liên quan đến performance. Nếu không có vấn đề performance thì có nghĩa là system nhỏ, mà system nhỏ thì không đáng để đánh giá;
- System vừa và lớn thường có vấn đề technology selection;
- Phần lớn system đều có không gian cải thiện;
- Phần lớn business system đều liên quan đến vấn đề extensibility và maintainability;
- Phần lớn business system quan trọng đều từng trải qua những bài học đau đớn trên production;
- System có data volume lớn chắc chắn có vấn đề về tính ổn định;
- System xử lý consumption chắc chắn có vấn đề về latency và backlog;
- Tích hợp third-party system chắc chắn liên quan đến vấn đề reliability;
- Distributed system chắc chắn liên quan đến vấn đề availability;
- Phối hợp giữa nhiều subsystem chắc chắn liên quan đến vấn đề data consistency;
- Transaction system có scenario gây tổn thất tài sản và scenario sự cố;

**Bài toán design**

- Ví dụ, nhiều machine cùng chia sẻ một lượng lớn business object, giữa các business object này có một số field liên kết bị lặp, làm thế nào để loại bỏ trùng lặp? Nếu field tương đối dài thì xử lý thế nào?
- Nếu trong thời gian ngắn có một lượng lớn request tràn vào, làm thế nào để đảm bảo tính ổn định của server?
- Ở cấp độ component: thiết kế một local cache? Thiết kế một distributed cache?
- Ở cấp độ module: thiết kế một task scheduling module? Cần cân nhắc những yếu tố nào?
- Ở cấp độ system: thiết kế một internal system, lấy dữ liệu bán hàng từ các department rồi thống kê thành report. Tính phức tạp thể hiện ở đâu? Những quality attribute then chốt là gì? Phân chia module, mối liên hệ giữa các module? Technology selection?

**Kinh nghiệm dự án**

Không nên đánh giá quá hai dự án. Vì để tìm hiểu sâu chi tiết một dự án sẽ tốn khá nhiều thời gian.

Thông thường, ứng viên sẽ được yêu cầu chọn một dự án mà họ thấy thu hoạch được nhiều nhất/có tính thử thách nhất/ấn tượng sâu sắc nhất/thú vị nhất/cảm thấy thất bại nhất. Sau đó đặt câu hỏi xoay quanh dự án này. Thường bắt đầu từ bối cảnh dự án, đánh giá hiểu biết tổng thể về tech stack, module và tương tác của dự án, các vấn đề kỹ thuật có tính thử thách đã gặp trong dự án và solution, điều tra và giải quyết vấn đề, vấn đề maintainability của code, đảm bảo chất lượng engineering, những điểm có thể cải thiện nếu làm lại từ đầu, v.v.

## Quy trình phỏng vấn

### Chuẩn bị trước

Interviewer cũng cần chuẩn bị một số việc. Chẳng hạn làm quen với thế mạnh kỹ năng, kinh nghiệm làm việc của ứng viên, rồi thiết kế buổi phỏng vấn.

Khi buổi phỏng vấn sắp bắt đầu, hãy chuẩn bị sẵn sàng. Ngoài ra, interviewer cũng cần hiểu một số tình hình cơ bản của công ty, đặc biệt là tech stack công ty sử dụng, toàn cảnh và định hướng nghiệp vụ, nội dung công việc, cơ chế thăng tiến, v.v. Ứng viên kỹ thuật thường hỏi khá nhiều về những điều này.

### Bắt đầu phỏng vấn

Thông thường bắt đầu bằng phần tự giới thiệu của ứng viên, nhưng ứng viên thường trình bày khá lan man, vì vậy tôi sẽ hỏi thẳng: Hãy nói về những ưu điểm của bạn và những điểm bạn thấy có thể cải thiện.

Sau đó bắt đầu phần hỏi kỹ thuật bằng một câu hỏi nền tảng tương đối đơn giản: Bạn quen với những search algorithm nào? Phần lớn mọi người có thể trả lời sequential search, binary search, hash search.

### Thiết kế câu hỏi

Đọc trước CV của ứng viên, lọc ra các keyword từ CV rồi thiết kế câu hỏi có chủ đích dựa trên những keyword này.

Ví dụ, nếu CV ứng viên đề cập đến MVVM, có thể hỏi sự khác nhau giữa MVVM và MVC; nếu đề cập đến observer pattern, có thể nói về observer pattern, đồng thời hỏi xem họ còn quen với những design pattern nào khác.

Có thể tuân theo nguyên tắc “thế mạnh - tiêu chuẩn - ngẫu nhiên”:

- Trước hết hỏi họ hứng thú và đầu tư nhiều vào mặt kỹ thuật nào (phần thế mạnh), dựa trên phần thế mạnh để trình bày nguyên lý và ứng dụng thực tế;
- Tiếp theo hỏi một số câu hỏi tiêu chuẩn, xem mức độ hiểu nguyên lý và ứng dụng thực tế của họ thế nào;
- Cuối cùng chọn ngẫu nhiên một câu hỏi, xem mức độ hiểu nguyên lý và ứng dụng thực tế của họ thế nào;

Với dự án cũng có thể làm tương tự:

- Trước hết hỏi về dự án khiến họ có cảm giác thành tựu nhất: tech stack, module và mối liên hệ, technology selection, vấn đề then chốt trong design, solution, chi tiết triển khai, không gian cải thiện;
- Tiếp theo hỏi về dự án khiến họ cảm thấy thất bại: vấn đề nằm ở đâu, đã nỗ lực những gì, cải thiện thế nào;

### Bầu không khí thoải mái

Ngay cả khi hỏi khá nhiều và khá khó, cũng cần chú ý duy trì bầu không khí thoải mái.

Trước phỏng vấn, có thể đùa vui phù hợp dựa trên thông tin cơ bản của ứng viên. Ví dụ, nếu ứng viên tên là Vương Khuê, tôi sẽ nói: Trước đây team chúng tôi có một người tên là Viên Khuê, mọi người đều gọi anh ấy là “Khuê gia”.

Trong quá trình phỏng vấn, đưa ra gợi ý phù hợp hoặc chia sẻ một chút quan điểm của mình cũng có thể làm giảm căng thẳng cho ứng viên.

### Học cách lắng nghe

Hỏi nhiều, nói ít, để ứng viên thể hiện nhiều hơn. Dựa trên câu trả lời của ứng viên để dẫn dắt, hỏi sâu dần hoặc chuyển sang hướng khác một cách phù hợp.

Hãy dẫn dắt ứng viên thể hiện mặt mạnh nhất của họ, để họ cảm thấy tốt hơn: dù sao cả hai bên đều đã bỏ ra thời gian và công sức cho một buổi phỏng vấn, đây không nên là dịp interviewer “Diss” ứng viên mà nên là cơ hội để hai bên giao tiếp tốt hơn. Rất có thể bạn cũng học được không ít điều từ ứng viên.

Phỏng vấn chỉ là việc hai bên có vai trò và lập trường khác nhau, nhưng điều đó không có nghĩa trình độ của interviewer nhất định cao hơn ứng viên.

### Ghi lại trọng điểm

Ghi chép câu trả lời của ứng viên một cách nghiêm túc và khách quan, cố gắng tránh mọi đánh giá chủ quan, cũng không gia công thêm (chẳng hạn tự tổng kết lại; khả năng tổng kết cũng là một đặc điểm của ứng viên).

### Luyện tập nhiều hơn

Phỏng vấn thử.

### Đưa ra phán đoán

Quá trình phỏng vấn là bước chuẩn bị, điều quan trọng là đưa ra phán đoán.

Điểm dễ khiến việc đưa ra phán đoán rơi vào ngộ nhận nhất là: tham sâu, cầu toàn. Luôn mong ứng viên vừa có kỹ thuật sâu vừa toàn diện. Thực tế, đây là một kỳ vọng xa vời. Nếu năng lực kỹ thuật của ứng viên vừa sâu vừa toàn diện, rất có thể họ sẽ rơi vào một trong hai tình huống: 1. ứng viên có lựa chọn tốt hơn; 2. ứng viên có thể thiếu sót ở phương diện khác, chẳng hạn phối hợp team.

Một thước đo tương đối phù hợp là: 1. Trình độ kỹ thuật của họ có đảm nhiệm được công việc hiện tại không; 2. Trình độ kỹ thuật của họ so với các thành viên trong cùng team thế nào; 3. Trình độ kỹ thuật của họ có tương đối phù hợp với số năm kinh nghiệm không, có tiềm năng đảm nhiệm task phức tạp hơn không.

### Mỗi độ tuổi coi trọng những điều khác nhau

Với engineer có dưới ba năm kinh nghiệm, nên coi trọng nền tảng kỹ thuật hơn, vì điều này đại diện cho tiềm năng tương lai của họ; đồng thời cũng đánh giá biểu hiện trong phát triển thực tế, chẳng hạn phối hợp team, kinh nghiệm nghiệp vụ, khả năng chịu áp lực, nhiệt tình và năng lực chủ động học tập.

Với engineer có trên ba năm kinh nghiệm, nên coi trọng kinh nghiệm nghiệp vụ và khả năng giải quyết vấn đề hơn, xem họ phân tích vấn đề cụ thể thế nào, đồng thời đánh giá chiều sâu và độ rộng nền tảng kỹ thuật trong phạm vi nghiệp vụ.

Về cách đánh giá trình độ kỹ thuật thực sự của một ứng viên và mức độ phù hợp với nhu cầu, tôi cũng đang học hỏi.

## Những bước đầu khi phỏng vấn

- Chuẩn bị sẵn camera và audio, có thể dùng tai nghe để kiểm tra trước.
- Đọc trước CV của ứng viên, lọc ra keyword và chuẩn bị một số câu hỏi cơ bản.
- Hỏi nhiều câu về nền tảng kỹ thuật để rèn cảm giác phỏng vấn.
- Hỏi sâu vừa phải về nguyên lý và triển khai.
- Nếu CV ứng viên có điểm nổi bật thì hỏi phần đó trước; nếu không, để ứng viên giới thiệu bối cảnh dự án rồi đặt câu hỏi dựa trên bối cảnh và kinh nghiệm dự án.
- Luyện tập kỹ thuật “hỏi liên tiếp” với số lượng nhỏ cho đến khi có thể sử dụng thành thạo.
- Chú trọng đánh giá khả năng phân tích và giải quyết vấn đề; nếu cần, có thể đưa ra một bài toán lập trình.
- Dành thời gian để đối phương hỏi: Bạn có điều gì muốn hỏi không? Đồng thời báo cho đối phương biết kết quả phỏng vấn sẽ được phản hồi trong vòng ba ngày làm việc.

## Đánh giá hiệu quả

Khi đã tương đối quen với vai trò interviewer kỹ thuật, cần nâng cao hiệu suất phỏng vấn. Tức là trong ít thời gian hơn vẫn đánh giá hiệu quả chiều sâu và độ rộng kỹ thuật của ứng viên. Có thể chuẩn bị một số câu hỏi thường gặp để làm bài kiểm tra tiêu chuẩn hóa.

Ví dụ, tôi thích đánh giá các chủ đề như memory management và algorithm, database index, cache, concurrency, system design, khả năng phân tích và tư duy vấn đề.

- Bạn quen với những data structure và algorithm dùng để tìm kiếm nào? Hãy chọn một loại bất kỳ để trình bày tư tưởng của nó và điểm thú vị theo bạn.
- Nếu chạy đến một method Java, bên trong tạo một list object, memory được phân bổ thế nào? Khi nào có thể gây stack overflow? Khi nào có thể gây OOM? Những nguyên nhân gây OOM là gì? Làm thế nào để tránh? Trên production đã từng gặp OOM chưa, đã giải quyết thế nào?
- Java generational garbage collection algorithm như thế nào? Garbage collector được chọn trong dự án hoạt động thế nào? Tại sao chọn collector này mà không phải collector kia?
- Java concurrency utility có những loại nào? Những tool khác nhau phù hợp với scenario nào?
- Nguyên lý triển khai của class `Atomic` atomic? Nguyên lý triển khai của `ConcurrentHashMap`?
- Làm thế nào để triển khai một reentrant lock?
- Hãy lấy một ví dụ trong dự án: những field nào đã dùng index? Tại sao là những field này? Bạn nghĩ còn không gian optimization nào không? Làm thế nào để tạo một index tốt?
- Cache có những parameter nào có thể thiết lập? Ảnh hưởng của từng parameter là gì?
- Redis có những expiration strategy nào? Làm thế nào để chọn expiration strategy cho redis?
- Làm thế nào để loại bỏ trùng lặp cho task phát hiện file virus?
- Bạn quen với những design pattern và design principle nào?
- Nếu xây dựng một module/full system từ 0 đến 1, bạn sẽ bắt đầu thế nào?

Nếu ứng viên không trả lời được, có thể hỏi: Nếu bạn thiết kế một XXX như vậy, bạn sẽ làm thế nào?

Tỷ lệ thời gian đại khái là: nền tảng kỹ thuật (25-30 phút) + dự án (20-25 phút) + ứng viên đặt câu hỏi (5-10 phút)

## Đôi lời dành cho ứng viên

**Tại sao ứng viên cần chú trọng nền tảng kỹ thuật**

Một thắc mắc thường gặp là: Phần lớn thời gian phát triển business system về cơ bản không liên quan đến việc thiết kế và triển khai data structure và algorithm, vậy tại sao phải đánh giá nguyên lý triển khai của `HashMap`? Tại sao phải học tốt các môn nền tảng như data structure và algorithm, operating system, network communication?

Giờ tôi có thể đưa ra một câu trả lời:

- Như đã nói ở trên, phần lớn vấn đề nghiệp vụ thực ra cuối cùng đều ánh xạ vào vấn đề kỹ thuật nền tảng: triển khai data structure và algorithm, memory management, concurrency control, network communication, v.v.; đây là nền móng để hiểu các chương trình Internet hiện đại quy mô lớn và giải quyết các vấn đề khó của chương trình, trừ khi bạn có thể tự chúc mình mãi mãi không gặp vấn đề khó nào và mãi mãi chỉ hài lòng với việc viết CRUD;
- Những nền tảng kỹ thuật này chính là nơi thú vị và đầy kích thích nhất trong thế giới lập trình. Nếu không hứng thú với chúng, rất khó đi sâu vào lĩnh vực này; chi bằng sớm chuyển nghề sang một công việc khác, thế giới phi kỹ thuật vẫn luôn thú vị và rộng lớn (đôi khi tôi cũng muốn ra ngoài đi đây đó nhiều hơn, không muốn giới hạn mình trong thế giới kỹ thuật);
- Nền tảng kỹ thuật là nội công của programmer, còn kỹ thuật cụ thể là chiêu thức. Chỉ có chiêu thức mà nội công không sâu, khi gặp cao thủ (sự cạnh tranh từ những người cùng nghề xuất sắc và các vấn đề khó) sẽ dễ dàng không chống đỡ nổi;
- Có nền tảng kỹ thuật chuyên môn vững chắc thì giới hạn có thể đạt được sẽ cao hơn, trong tương lai có nhiều khả năng đảm nhiệm việc giải quyết vấn đề kỹ thuật phức tạp, hoặc đưa ra solution tốt hơn cho cùng một vấn đề;
- Mọi người thích hợp tác với những người tương đồng với mình, người giỏi có xu hướng hợp tác với người giỏi để đạt hiệu quả tốt hơn; nếu phần lớn thành viên trong một team có nền tảng kỹ thuật tốt, khi một người có nền tảng kỹ thuật khá yếu gia nhập, chi phí phối hợp sẽ tăng lên; nếu muốn hợp tác với người giỏi để đạt kết quả tốt hơn, bạn cần ít nhất có nền tảng kỹ thuật đủ để phối hợp với người giỏi;
- Mở rộng thêm những năng lực khác trên nền tảng CRUD cũng là một lựa chọn tốt, nhưng đây sẽ không phải tư thế của một programmer thực thụ, cùng lắm là nhân sự ở các vị trí khác như product manager, project manager, HR, vận hành có nền tảng kỹ thuật. Đây là vấn đề lựa chọn nghề nghiệp, đã vượt ra ngoài phạm vi đánh giá programmer.

**Đừng để tâm đến việc không trả lời được một câu hỏi nào đó**

Nếu interviewer hỏi bạn nhiều câu hỏi mà có một số câu bạn không trả lời được, đừng để tâm. Rất có thể interviewer chỉ đang kiểm tra chiều sâu và độ rộng kỹ thuật của bạn, sau đó phán đoán xem bạn có đạt một mức nhất định hay không.

Điểm quan trọng là: Có những câu hỏi bạn trả lời rất sâu, cũng thể hiện khả năng tư duy sâu sắc của bạn.

Đây là điều tôi chỉ lĩnh hội được sau khi trở thành interviewer kỹ thuật. Tất nhiên, không phải interviewer kỹ thuật nào cũng nghĩ như vậy, nhưng tôi cho rằng đây là cách phù hợp hơn.

## Tài liệu tham khảo

- [9 ngộ nhận lớn của interviewer kỹ thuật](https://zhuanlan.zhihu.com/p/51404304)
- [Làm thế nào để trở thành một interviewer tốt?](https://www.zhihu.com/question/26240321)

<!-- @include: @article-footer.snippet.md -->
