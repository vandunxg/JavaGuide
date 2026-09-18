---
title: Trao đổi về cách chuẩn bị cho vòng phỏng vấn kỹ thuật đầu tiên từ góc nhìn của interviewer và candidate
description: "Trao đổi về cách chuẩn bị cho vòng phỏng vấn kỹ thuật đầu tiên từ góc nhìn của interviewer và candidate: hệ thống hóa các khái niệm then chốt, vấn đề thường gặp và điểm thực hành xoay quanh kiến thức kỹ thuật cùng tổng kết phỏng vấn, giúp bạn học tập và chuẩn bị phỏng vấn hiệu quả."
category: Tuyển tập bài viết kỹ thuật chất lượng cao
author: Qin Shuiyu
tag:
  - phỏng vấn
head:
  - - meta
    - name: keywords
      content: chuẩn bị phỏng vấn kỹ thuật,góc nhìn interviewer,góc nhìn candidate,nền tảng kỹ thuật,đánh giá nghiệp vụ,kỹ năng phỏng vấn,chiều sâu và độ rộng kỹ thuật,phương pháp luận phỏng vấn
---

> **Lời giới thiệu**: Thảo luận về phỏng vấn kỹ thuật từ góc nhìn của cả interviewer và ứng viên! Rất đáng đọc!
>
> **Tổng quan nội dung:**
>
> - Chỉ khi đánh giá ứng viên thông qua nền tảng kỹ thuật mới có thể đánh giá năng lực kỹ thuật thực sự của ứng viên: chiều sâu và độ rộng kỹ thuật.
> - Kết hợp thực chiến với lý thuyết. Ví dụ, sau khi ứng viên trình bày bố cục mô hình bộ nhớ JVM, có thể hỏi tiếp: Những nguyên nhân nào có thể gây ra OOM, có biện pháp phòng ngừa nào? Bạn đã từng gặp vấn đề memory leak chưa? Làm thế nào để điều tra và giải quyết những vấn đề này?
> - Không nên đánh giá quá hai dự án. Vì để đánh giá sâu chi tiết một dự án vẫn tốn khá nhiều thời gian. Thông thường, sẽ để ứng viên chọn một dự án mà họ cho là có nhiều thu hoạch nhất/có tính thử thách nhất/ấn tượng sâu sắc nhất/đặc biệt thú vị. Sau đó đặt câu hỏi xoay quanh dự án này. Thường sẽ bắt đầu từ bối cảnh dự án, đánh giá mức độ hiểu tổng thể về technology stack, module và tương tác của dự án, các vấn đề kỹ thuật có tính thử thách gặp phải trong dự án cùng giải pháp, việc điều tra và giải quyết vấn đề, vấn đề maintainability của code, bảo đảm chất lượng engineering, v.v.
> - Hỏi nhiều, nói ít, để ứng viên thể hiện nhiều hơn. Dựa trên câu trả lời của ứng viên để dẫn dắt, hỏi sâu dần hoặc chuyển sang chiều ngang một cách phù hợp.
>
> **Địa chỉ bài viết gốc:** <https://www.cnblogs.com/lovesqcc/p/15169365.html>

## Mục tiêu và tư duy đánh giá

Trước hết cần xác định mục tiêu đánh giá của vòng phỏng vấn kỹ thuật đầu tiên:

- Nền tảng kỹ thuật của ứng viên;
- Tư duy và năng lực giải quyết vấn đề của ứng viên.

Nền tảng kỹ thuật là nền móng (những phần bên dưới tảng băng), chiếm bảy phần; tư duy và năng lực giải quyết vấn đề là khả năng triển khai (phần lộ ra bên trên tảng băng), chiếm ba phần. Đánh giá nghiệp vụ và nền tảng kỹ thuật theo tỷ lệ ba-bảy.

## Đánh giá nền tảng kỹ thuật

### Vì sao phải đánh giá nền tảng kỹ thuật?

Hai năng lực tư duy kỹ thuật quan trọng nhất của programmer là năng lực tư duy logic và năng lực thiết kế trừu tượng. Năng lực tư duy logic là nền tảng, năng lực thiết kế trừu tượng là cấp độ cao hơn. Đánh giá nền tảng kỹ thuật vừa có thể đánh giá đồng thời hai năng lực tư duy này. Có hiểu các khái niệm kỹ thuật nền tảng và mối liên hệ giữa chúng hay không là cách đánh giá năng lực tư duy logic; có thể trừu tượng hóa vấn đề nghiệp vụ thành vấn đề kỹ thuật và tổ chức ánh xạ hợp lý hay không là cách đánh giá năng lực thiết kế trừu tượng.

Phần lớn vấn đề nghiệp vụ đều có thể được trừu tượng hóa thành vấn đề kỹ thuật. Ở một mức độ nào đó, vấn đề nghiệp vụ chỉ là cách diễn đạt theo lĩnh vực của vấn đề kỹ thuật.

Vì vậy, **chỉ thông qua đánh giá nền tảng kỹ thuật mới có thể đánh giá năng lực kỹ thuật thực sự của ứng viên: chiều sâu và độ rộng kỹ thuật.**

### Đánh giá nền tảng kỹ thuật như thế nào?

Đánh giá nền tảng kỹ thuật bằng cách đặt câu hỏi hiệu quả theo nhiều góc độ.

#### Là gì - vì sao

“Là gì” đánh giá mức độ hiểu cơ bản về khái niệm, “vì sao” đánh giá nguyên lý triển khai của khái niệm.

Ví dụ: index là gì? Index được triển khai như thế nào?

#### Dẫn dắt - hỏi theo chiều ngang - hỏi sâu

Dẫn dắt, chẳng hạn hỏi “Bạn có quen thuộc với các công cụ đồng bộ của Java không?” để thăm dò; sau khi nhận được câu trả lời khẳng định, có thể hỏi tiếp: “Bạn quen thuộc với những class công cụ đồng bộ nào?” để hiểu độ rộng của ứng viên;

Sau khi nhận được câu trả lời của ứng viên, có thể hỏi tiếp: “Hãy nói về nguyên lý triển khai của `ConcurrentHashMap` hoặc `AQS`?”

Một người có thể trình bày nguyên lý kỹ thuật rõ ràng đến mức nào, bao gồm cả tư duy và chi tiết, cho thấy năng lực nắm vững kỹ thuật của người đó mạnh đến đâu.

#### Hỏi theo kiểu nhảy cóc/đan xen

Ví dụ: khi nói đến việc tìm kiếm hiệu quả bằng hash, có thể trao đổi về thuật toán consistent hashing. Hai vấn đề vừa có liên quan vừa có nhiều điểm khác biệt. Đây cũng là một cách đánh giá độ rộng kỹ thuật.

#### Hỏi mang tính tổng kết

Ví dụ: Khi thực hiện XXX, bạn đã thu được những kinh nghiệm nào có thể chia sẻ? Đánh giá năng lực khái quát và tổng kết của ứng viên.

#### Kết hợp thực chiến với lý thuyết

Ví dụ, sau khi ứng viên trình bày bố cục mô hình bộ nhớ JVM, có thể hỏi tiếp: Những nguyên nhân nào có thể gây ra OOM, có biện pháp phòng ngừa nào? Bạn đã từng gặp vấn đề memory leak chưa? Làm thế nào để điều tra và giải quyết những vấn đề này?

Ví dụ, nếu ứng viên đề cập đến tối ưu hóa SQL và tối ưu hóa index, thì có thể trao đổi về nguyên lý triển khai của index và cách xây dựng index tối ưu;

Lại ví dụ, nếu ứng viên đề cập đến transaction, thì có thể trao đổi về nguyên lý triển khai transaction, isolation level, cách triển khai snapshot, v.v.;

#### Kết hợp phần quen thuộc và phần không quen thuộc

Vừa hỏi phần ứng viên ghi là quen thuộc trong CV, vừa hỏi phần không ghi. Ví dụ CV của ứng viên ghi: quen thuộc với mô hình bộ nhớ JVM, vậy tôi sẽ đánh giá về phần liên quan đến memory management (phần quen thuộc), sau đó đánh giá về các class công cụ concurrency của Java (phần chưa chắc quen thuộc).

#### Kết hợp kiến thức chết và kiến thức sống

Ví dụ, có những search algorithm nào? Sequential search, binary search, hash search. Đây là những thứ mọi người thường nói ra được, cũng là “kiến thức chết”.

Những search algorithm này phù hợp với trường hợp sử dụng nào? Trong công việc, bạn đã dùng search algorithm nào ở những trường hợp nào? Vì sao? Đây là “kiến thức sống”.

#### Những vấn đề gặp phải trong học tập hoặc công việc

Đôi khi, những vấn đề gặp phải trong học tập và công việc cũng có thể được dùng làm câu hỏi phỏng vấn.

Ví dụ, gần đây tôi đang học phần concurrency của cuốn 《Giới thiệu về hệ điều hành》, trong đó có một chương nói về cách làm cho data structure trở nên thread-safe. Ở đây có một số điểm có thể đặt câu hỏi: Làm thế nào để triển khai một lock? Làm thế nào để triển khai một counter thread-safe? Làm thế nào để triển khai một linked list thread-safe? Làm thế nào để triển khai một `Map` thread-safe? Làm thế nào để cải thiện performance của concurrency?

Những vấn đề gặp phải trong công việc cũng có thể được trừu tượng hóa và chắt lọc thành câu hỏi phỏng vấn nền tảng kỹ thuật.

#### Đặt câu hỏi theo mức độ phù hợp với technology stack

Nếu một số technology mà ứng viên sử dụng (như được ghi trong CV) khá phù hợp với technology stack của công ty, có thể đặt câu hỏi chuyên sâu về các điểm kỹ thuật đó để đánh giá mức độ nắm vững của ứng viên. Nếu mức độ nắm vững tốt, mức độ phù hợp kỹ thuật sẽ tương đối cao hơn.

Tất nhiên, điều này không thể là căn cứ để loại những ứng viên chưa sử dụng technology stack đó. Ví dụ công ty chúng tôi sử dụng `MongoDB` và `MySQL`, còn một ứng viên chưa từng dùng `Mongodb，` nhưng đã sử dụng nhiều hệ thống lưu trữ như `MySQL`, `Redis`, `ES`, `HBase`, thì mức độ phù hợp không hề kém hơn ứng viên chỉ từng sử dụng `MySQL` và `MongoDB`, vì độ rộng kỹ thuật mà người đó tiếp xúc lớn hơn, có thể suy ra rằng người đó đủ năng lực nắm vững `Mongodb`.

#### Tạo ngân hàng câu hỏi phỏng vấn có cá tính

Mỗi interviewer kỹ thuật đều có một ngân hàng câu hỏi phỏng vấn. Hãy liên tục tích lũy ngân hàng câu hỏi, khi bất chợt nghĩ ra vấn đề trong sinh hoạt hằng ngày thì ghi lại ngay.

## Đánh giá theo khía cạnh nghiệp vụ

### Vì sao phải đánh giá khía cạnh nghiệp vụ?

Điểm dễ bỏ sót khi đánh giá nền tảng kỹ thuật là những đặc điểm năng lực phi kỹ thuật của ứng viên, chẳng hạn năng lực giao tiếp và tổ chức, năng lực dẫn dắt dự án, năng lực chịu áp lực, năng lực giải quyết vấn đề thực tế, ảnh hưởng trong team và các đặc điểm tính cách khác.

### Vì sao không thể chỉ đánh giá khía cạnh nghiệp vụ?

Vì về nghiệp vụ thường khá quen thuộc, ứng viên có thể trực tiếp nói theo giải pháp hiện có, rất khó đánh giá mức độ hiểu sâu, khả năng mở rộng theo chiều ngang và năng lực khái quát, tổng kết của ứng viên.

Về điểm này, nên đánh giá có mục tiêu năng lực khái quát và tổng kết của ứng viên: ví dụ, trong quá trình xây dựng, phát triển hoặc bảo trì microservice/đảm bảo tính ổn định hoặc performance của hệ thống, bạn đã thu được những kinh nghiệm nào có thể chia sẻ?

## Đánh giá năng lực giải quyết vấn đề

Chỉ có nền tảng kỹ thuật thôi là chưa đủ; thông thường tốt nhất nên kết hợp với nghiệp vụ thực tế, dựa trên nghiệp vụ trong dự án của ứng viên để trừu tượng hóa thành vấn đề kỹ thuật rồi đánh giá.

Tư duy giải quyết vấn đề chú trọng tiến dần từng lớp. Điều này cũng đặt ra yêu cầu khá cao đối với interviewer, người cần đồng thời có năng lực lắng nghe tốt, chiều sâu kỹ thuật và kinh nghiệm nghiệp vụ. Trước hết phải chăm chú lắng nghe phần trình bày của ứng viên, tìm ra điểm đột phá kỹ thuật phù hợp, sau đó đặt câu hỏi. Nếu không thể đi vào vấn đề, việc đánh giá rất dễ thất bại.

### Thiết kế vấn đề

- Ví dụ giữa nhiều máy cần chia sẻ một lượng lớn business object, giữa các business object này có một số field kết hợp bị trùng lặp, làm thế nào để loại bỏ trùng lặp?
- Nếu trong chốc lát có một lượng lớn request tràn vào, làm thế nào để bảo đảm tính ổn định của server?

### Kinh nghiệm dự án

Không nên đánh giá quá hai dự án. Vì để đánh giá sâu chi tiết một dự án vẫn tốn khá nhiều thời gian.

Thông thường, sẽ để ứng viên chọn một dự án mà họ cho là có nhiều thu hoạch nhất/có tính thử thách nhất/ấn tượng sâu sắc nhất/đặc biệt thú vị. Sau đó đặt câu hỏi xoay quanh dự án này. Thường sẽ bắt đầu từ bối cảnh dự án, đánh giá mức độ hiểu tổng thể về technology stack, module và tương tác của dự án, các vấn đề kỹ thuật có tính thử thách gặp phải trong dự án cùng giải pháp, việc điều tra và giải quyết vấn đề, vấn đề maintainability của code, bảo đảm chất lượng engineering, v.v.

## Interviewer làm tốt một buổi phỏng vấn như thế nào?

### Chuẩn bị trước

Interviewer cũng cần chuẩn bị một số việc. Ví dụ, làm quen với thế mạnh kỹ năng, kinh nghiệm làm việc của ứng viên, v.v., và thiết kế buổi phỏng vấn.

Khi buổi phỏng vấn sắp bắt đầu, cần chuẩn bị đầy đủ. Ngoài ra, interviewer cũng cần hiểu một số tình hình cơ bản của công ty, đặc biệt là technology stack công ty sử dụng, bức tranh và định hướng nghiệp vụ, nội dung công việc, chế độ thăng tiến, v.v.; ứng viên kỹ thuật thường hỏi khá nhiều về những điểm này.

### Bắt đầu phỏng vấn

Thông thường sẽ bắt đầu bằng phần ứng viên tự giới thiệu, nhưng ứng viên thường trình bày khá lan man, vì vậy tôi sẽ trực tiếp hỏi: Hãy nói về những ưu điểm của bạn và những điểm bạn cho rằng có thể cải thiện.

Sau đó bắt đầu phần hỏi kỹ thuật bằng một câu hỏi nền tảng tương đối đơn giản: Bạn quen thuộc với những search algorithm nào? Đa số mọi người đều có thể trả lời sequential search, binary search, hash search.

### Thiết kế câu hỏi

Đọc trước CV của ứng viên, lọc ra các từ khóa từ CV, rồi dựa trên những từ khóa này để thiết kế câu hỏi có mục tiêu.

Ví dụ CV ứng viên đề cập đến `MVVM`, có thể hỏi sự khác biệt giữa `MVVM` và `MVC`; nếu đề cập đến observer pattern, có thể trao đổi về observer pattern, đồng thời hỏi xem người đó còn quen thuộc với những design pattern nào khác.

### Không khí thoải mái

Ngay cả khi những câu hỏi đặt ra khá nhiều và khá khó, cũng cần chú ý duy trì không khí thoải mái.

Trước buổi phỏng vấn, có thể đùa nhẹ phù hợp dựa trên thông tin cơ bản của ứng viên. Ví dụ một ứng viên tên Wang Kui, tôi sẽ nói: Trước đây team chúng tôi có một người tên Yuan Kui, mọi người đều gọi anh ấy là Kui gia.

Trong quá trình phỏng vấn, đưa ra gợi ý phù hợp hoặc chia sẻ một chút quan điểm của mình cũng có thể làm giảm căng thẳng cho ứng viên.

### Học cách lắng nghe

Hỏi nhiều, nói ít, để ứng viên thể hiện nhiều hơn. Dựa trên câu trả lời của ứng viên để dẫn dắt, hỏi sâu dần hoặc chuyển sang chiều ngang một cách phù hợp.

Dẫn dắt ứng viên thể hiện mặt mạnh nhất của họ, để họ cảm thấy thoải mái hơn: dù sao cả hai bên đều đã bỏ ra thời gian và công sức cho một buổi phỏng vấn, đây không nên là dịp interviewer Diss ứng viên, mà nên là cơ hội để hai bên trao đổi tốt hơn. Rất có thể bạn cũng học được không ít điều từ ứng viên.

Phỏng vấn chỉ là việc vai trò và lập trường của hai bên khác nhau, nhưng điều đó không có nghĩa trình độ của interviewer nhất định cao hơn ứng viên.

### Ghi lại trọng điểm

Ghi chép câu trả lời của ứng viên một cách nghiêm túc và khách quan, cố gắng tránh mọi đánh giá chủ quan, cũng không tự ý gia công (ví dụ tự mình tổng kết lại, vì năng lực tổng kết cũng là một đặc điểm của ứng viên).

## Đưa ra phán đoán

Quá trình phỏng vấn là bước chuẩn bị, điều quan trọng là đưa ra phán đoán.

Sai lầm dễ mắc nhất khi đưa ra phán đoán là tham sâu cầu toàn. Luôn mong ứng viên vừa có kỹ thuật chuyên sâu vừa toàn diện. Trên thực tế, đây là một kỳ vọng quá cao. Nếu năng lực kỹ thuật của ứng viên vừa chuyên sâu vừa toàn diện, rất có thể cũng sẽ gặp hai tình huống:

1. Ứng viên có lựa chọn tốt hơn;
2. Ứng viên có thể tồn tại thiếu sót ở những phương diện khác, chẳng hạn khả năng phối hợp trong team.

Một thước đo tương đối phù hợp là:

1. Trình độ kỹ thuật của họ có đáp ứng được công việc hiện tại hay không;
2. Trình độ kỹ thuật của họ so với các thành viên trong cùng team như thế nào;
3. Trình độ kỹ thuật của họ có tương xứng với số năm kinh nghiệm hay không, có tiềm năng đảm nhiệm những nhiệm vụ phức tạp hơn hay không.

**Độ tuổi khác nhau coi trọng những điều khác nhau.**

Đối với engineer có dưới ba năm kinh nghiệm, nên coi trọng nền tảng kỹ thuật hơn, vì điều này thể hiện tiềm năng tương lai của họ; đồng thời cũng đánh giá biểu hiện của họ trong phát triển thực tế, chẳng hạn khả năng phối hợp trong team, kinh nghiệm nghiệp vụ, năng lực chịu áp lực, sự nhiệt tình và năng lực chủ động học tập.

Đối với engineer có trên ba năm kinh nghiệm, nên coi trọng kinh nghiệm nghiệp vụ và năng lực giải quyết vấn đề hơn, xem họ phân tích vấn đề cụ thể như thế nào, đồng thời đánh giá chiều sâu và độ rộng nền tảng kỹ thuật trong phạm vi nghiệp vụ.

Về cách phán đoán trình độ kỹ thuật thực sự của một ứng viên và liệu họ có phù hợp với yêu cầu hay không, tôi cũng đang học hỏi.

## Đôi lời với ứng viên

### Chú trọng nền tảng kỹ thuật

Một thắc mắc thường gặp là: Phần lớn thời gian phát triển hệ thống nghiệp vụ không liên quan đến thiết kế và triển khai data structure và algorithm, vậy tại sao phải đánh giá nguyên lý triển khai của `HashMap`? Tại sao phải học tốt các môn nền tảng như data structure và algorithm, operating system, network communication?

Giờ đây tôi có thể đưa ra câu trả lời:

- Như đã nói ở trên, phần lớn vấn đề nghiệp vụ trên thực tế cuối cùng đều sẽ ánh xạ đến các vấn đề kỹ thuật nền tảng: triển khai data structure và algorithm, memory management, concurrency control, network communication, v.v.; đây là nền móng để hiểu các chương trình Internet hiện đại quy mô lớn cũng như giải quyết những vấn đề khó của chương trình, — trừ khi bạn có thể tự chúc mình sẽ không bao giờ gặp vấn đề khó và mãi chỉ hài lòng với việc viết CRUD;
- Những nền tảng kỹ thuật này chính là nơi thú vị và hấp dẫn nhất trong thế giới lập trình. Nếu không hứng thú với chúng, rất khó đi sâu vào lĩnh vực này; chi bằng sớm chuyển sang nghề khác, thế giới phi kỹ thuật vẫn luôn rộng lớn và tuyệt vời (đôi khi tôi cũng muốn ra ngoài đi đây đó nhiều hơn, không muốn bị giới hạn trong thế giới kỹ thuật);
- Nền tảng kỹ thuật là nội công của programmer, còn kỹ thuật cụ thể là chiêu thức. Chỉ có chiêu thức mà nội công không sâu, khi gặp cao thủ (sự cạnh tranh từ những người giỏi trong ngành và các vấn đề khó) thì rất dễ không chịu nổi một đòn;
- Có nền tảng kỹ thuật chuyên môn vững chắc sẽ đạt được giới hạn cao hơn, trong tương lai có nhiều khả năng đảm nhiệm việc giải quyết các vấn đề kỹ thuật phức tạp, hoặc đưa ra giải pháp tốt hơn cho cùng một vấn đề;
- Mọi người thích hợp tác với những người tương đồng với mình, người giỏi có xu hướng hợp tác với người giỏi để đạt được kết quả tốt hơn; nếu phần lớn thành viên trong một team có nền tảng kỹ thuật tốt, khi một người có nền tảng kỹ thuật tương đối yếu gia nhập, chi phí phối hợp sẽ tăng; nếu muốn hợp tác với người giỏi để đạt kết quả tốt hơn, bạn cần ít nhất có nền tảng kỹ thuật đủ để phối hợp với người giỏi;
- Mở rộng thêm những năng lực khác trên nền tảng CRUD cũng là một lựa chọn tốt, nhưng đây không phải tư thế của một programmer đúng nghĩa, nhiều nhất chỉ là một nhân sự ở các vị trí khác như product manager, project manager, HR, operations có nền tảng kỹ thuật. Đây là vấn đề lựa chọn nghề nghiệp, đã vượt ra ngoài phạm vi đánh giá programmer.

### Đừng bận tâm nếu không trả lời được một câu hỏi nào đó

Nếu interviewer hỏi bạn rất nhiều câu hỏi mà có một số câu bạn không trả lời được, đừng bận tâm. Rất có thể interviewer chỉ đang kiểm tra chiều sâu và độ rộng kỹ thuật của bạn, sau đó phán đoán xem bạn có đạt một mức chuẩn nào đó hay không.

Điểm quan trọng là: có những vấn đề bạn trả lời rất sâu, đồng thời thể hiện năng lực suy nghĩ sâu sắc của mình.

Đây là điều tôi chỉ lĩnh hội được sau khi trở thành interviewer kỹ thuật. Tất nhiên, không phải interviewer kỹ thuật nào cũng nghĩ như vậy, nhưng tôi cho rằng đây nên là cách phù hợp hơn.

<!-- @include: @article-footer.snippet.md -->
