---
title: 10 câu hỏi phỏng vấn mở liên quan đến AI Coding
description: Bao quát kỹ thuật sử dụng các AI Coding IDE như Cursor, Claude Code, Trae, sự khác biệt giữa Spec Coding và Vibe Coding, cùng những câu hỏi phỏng vấn thường gặp về ảnh hưởng của AI đến phát triển backend.
category: AI Application Development
icon: "mdi:code-tags"
head:
  - - meta
    - name: keywords
      content: AI Coding,Cursor,Claude Code,Spec Coding,Vibe Coding,AI IDE,công cụ lập trình,backend development
---

Khi phỏng vấn tại Tencent, interviewer hỏi tôi: “Bạn đã dùng những công cụ AI Coding nào?”. Tôi trả lời: “Trae.”

Không khí đột nhiên im lặng hai giây. Tôi không hiểu tại sao interviewer lại im lặng, lúc đó còn nghĩ: “Có phải câu trả lời của mình chưa đủ cao cấp không?”.

Chỉ sau khi trượt phỏng vấn tôi mới nhận ra: Trae là sản phẩm của ByteDance, công cụ của Tencent là CodeBuddy, còn của Alibaba là Qoder.

Đùa vậy thôi! Hôm nay G chia sẻ 9 câu hỏi mở về AI Coding thường được hỏi trong các buổi phỏng vấn kỹ thuật dành cho ứng viên mới tốt nghiệp và ứng viên đã đi làm, hy vọng sẽ giúp ích cho bạn.

1. ⭐ **AI Coding IDE**: kỹ thuật sử dụng các công cụ như Cursor, Claude Code
2. ⭐ **Ảnh hưởng của AI đến phát triển backend**: AI có đào thải lập trình viên junior không? Rủi ro lớn nhất là gì?
3. ⭐ **Năng lực cạnh tranh cốt lõi trong tương lai**: năng lực cạnh tranh cốt lõi của backend engineer sau 3 năm là gì?

## Kỹ thuật sử dụng AI Coding IDE

### Bạn đã từng dùng AI Coding IDE nào chưa? Cảm nhận thế nào?

Cảm nhận chung hiện tại là: năng lực AI Coding tiến bộ rất nhanh. Nó đã phát triển từ việc code completion đơn giản vài năm trước thành một trợ lý engineering có thể cộng tác sâu.

Tôi tổng hợp một phương pháp sử dụng của riêng mình:

1. Khi tiếp nhận một project hoặc module phức tạp, tôi không trực tiếp yêu cầu AI viết code, mà trước tiên yêu cầu Cursor phân tích toàn bộ codebase và tạo một tài liệu bao gồm architecture cốt lõi, trách nhiệm của các module và data flow. Bước này rất quan trọng vì nó quyết định chất lượng cộng tác về sau. Chỉ khi tôi và AI có cùng cách hiểu về project thì output về sau mới ổn định và chất lượng cao.
2. Với mỗi development task độc lập, hãy mở một conversation mới và cung cấp context cần thiết, bao gồm background của requirement, các module liên quan và các constraint. Cách này giúp giảm context pollution, khiến code AI tạo ra chính xác hơn.
3. Định kỳ xóa các implementation dư thừa và code đã deprecated. Code cũ sẽ gây hiểu sai cho phán đoán của AI và làm tăng context noise.

### Nguyên tắc cốt lõi của AI Coding

AI là một knowledge base và công cụ hỗ trợ mạnh mẽ, có thể giúp chúng ta nhanh chóng triển khai functionality và học kiến thức mới. Nhưng nếu hoàn toàn phụ thuộc vào AI để viết code mà không hiểu nguyên lý, năng lực kỹ thuật cá nhân có thể suy giảm.

Một số nguyên tắc:

- Sau khi AI generate code, bắt buộc phải Review thủ công.
- Khi cần, tự viết lại các logic quan trọng.
- Các core path phải được kiểm thử tải và kiểm thử boundary.

Tôi muốn nâng cao efficiency, nhưng không đánh đổi năng lực kỹ thuật.

### ⭐ Kinh nghiệm thực tế với Cursor

> Ở đây lấy Cursor làm ví dụ, các AI IDE khác cũng tương tự.

1. **Nắm architecture trước rồi mới bắt tay làm**: dù tự viết code hay để AI generate code, trước tiên đều phải làm rõ requirement, architecture tổng thể và boundary của module. Nếu trực tiếp code khi architecture còn mơ hồ, rất dễ xuất hiện implementation trùng lặp hoặc conflict về trách nhiệm, khiến chi phí sửa đổi về sau còn cao hơn.
2. **Một Chat tập trung vào một functionality**: mở Chat mới cho functionality mới hoặc thay đổi lớn, đồng thời đưa mô tả structure của project hoặc tài liệu quan trọng vào context ngay từ đầu. Như vậy có thể tránh lịch sử conversation gây nhiễu.
3. **Viết guideline sau khi functionality được triển khai**: yêu cầu AI tổng hợp quá trình implementation và rút ra các bước dùng chung. Ví dụ, quy trình chuẩn để thêm API, cách implementation thống nhất cho việc export file. Những nội dung này có thể nhanh chóng được tái sử dụng cho các requirement tương tự sau này.
4. **Không phụ thuộc vào AI, chủ động review lại**: AI chỉ đóng vai trò hỗ trợ; sau khi code được generate, cần Review kỹ, hiểu nguyên lý và tối ưu những phần không hợp lý.
5. **Định kỳ xóa code không cần thiết**: dọn dẹp code dư thừa, giảm việc gây hiểu sai cho AI và giảm nhiễu context, nâng cao hiệu suất development.
6. **Tận dụng tốt file cấu hình**: `.cursorrules` định nghĩa quy tắc, style và các đoạn code thường dùng cho code AI generate; `.cursorignore` chỉ định các file / directory AI không được phép sửa, bảo vệ core code.
7. **Duy trì tài liệu liên tục**: sau khi project có thay đổi lớn, yêu cầu AI đồng bộ cập nhật tài liệu và ghi lại kinh nghiệm “vấp phải vấn đề”.
8. **Để AI “học” project trước**: với project lớn, trước tiên yêu cầu Cursor phân tích codebase, tạo tài liệu structure bao gồm architecture, trách nhiệm của directory và các core class, dùng làm context nền tảng cho development về sau.

### ⭐ Kỹ thuật sử dụng Claude Code

1. **Context window là tài nguyên đắt giá nhất của bạn**——bản chất của mọi kỹ thuật đều là giúp bạn sử dụng whiteboard này hiệu quả hơn.
2. **Lập kế hoạch trước rồi mới thực thi**——Plan Mode là khoản đầu tư cho thời gian về sau.
3. **`CLAUDE.md` tự tiến hóa**——biến những lần sửa thành rule để AI ngày càng dễ sử dụng hơn.
4. **Parallel là đòn bẩy efficiency lớn nhất**——nhiều instance + Worktree + subagent.
5. **Verification quan trọng hơn trust**——đưa cho Claude acceptance criteria để nó tự kiểm tra.
6. **`/compact` hiệu quả hơn việc sửa đi sửa lại**——sau khi context bị ô nhiễm, nén hoặc xóa sạch rồi bắt đầu lại sẽ tốt hơn.

Tôi đã từng chia sẻ riêng nội dung chi tiết về Claude Code: [Hướng dẫn sử dụng Claude Code](https://javaguide.cn/ai-coding/claudecode-tips.html).

## Ảnh hưởng của AI đến lập trình viên

### Bạn nhìn nhận thế nào về ảnh hưởng của AI đến phát triển backend?

AI sẽ không thay thế backend engineer, nhưng sẽ thay đổi phương thức làm việc và cấu trúc năng lực của backend engineer.

AI có thể giúp chúng ta xử lý các công việc lặp lại và mang tính pattern:

- **Ở cấp độ coding**: các AI tool thể hiện khá tốt trong việc generate **code mang tính pattern (Boilerplate)**, hiệu suất viết CRUD, unit test và glue code có thể tăng 50%~70%. Nhưng với **distributed constraint** (như gia hạn timeout của distributed lock, semantic Exactly-once của message queue, thiết kế idempotency cho API), AI có **rủi ro “hallucination”** rõ rệt——nó thường chỉ đưa ra code Happy Path, bỏ qua logic compensation khi có exception trong production, xử lý race condition và kiểm soát boundary của distributed transaction.
- **Ở cấp độ architecture**: AI đang tạo ra các application paradigm mới, chẳng hạn business workflow tự động hóa do intelligent agent (Agent) điều khiển; backend cần cung cấp các API capability linh hoạt và atomic hơn. Các API “lớn và đầy đủ” truyền thống đang từng bước được tách thành các atomic capability có thể được AI gọi.
- **Ở cấp độ vận hành và troubleshooting**: AI có thể hỗ trợ phân tích log, cảnh báo monitoring, thậm chí dự đoán system bottleneck. Ví dụ, tool dựa trên AIOps có thể tự động phân tích pattern của log bất thường và định vị root cause.

AI giúp backend engineer tập trung hơn vào business modeling, thiết kế complex system và các quyết định architecture, những công việc cốt lõi giàu tính sáng tạo hơn.

Với bản thân tôi, tôi thường thảo luận business và technical solution với AI; nó luôn mang lại cho tôi những gợi ý hữu ích——đặc biệt khi breakdown requirement và lựa chọn technology, AI có thể cung cấp góc nhìn đa chiều.

Theo kinh nghiệm thực tế, năng lực lập trình có AI hỗ trợ có thể được quy về hai khía cạnh:

- **Lập kế hoạch và delivery từ 0 đến 1**: đưa ra mô tả requirement, AI có thể tự hoàn thành việc lựa chọn technology và thiết kế architecture, phù hợp để nhanh chóng verify ý tưởng, nhưng solution vẫn cần được review thủ công.
- **Tối ưu incremental trên code hiện có**: trong codebase có complexity sẵn, AI có thể hiểu architecture hiện có, định vị vấn đề và hoàn thành việc tối ưu. Nhưng trường hợp solution AI đưa ra “trông có vẻ đúng” rồi thất bại khi lên production cũng không ít.

### Năng lực cạnh tranh cốt lõi của frontend và backend developer đã thay đổi

Nói thẳng, năng lực cạnh tranh cốt lõi của frontend và backend developer đã thay đổi.

Trước đây frontend cạnh tranh về tốc độ code và độ chính xác khi tái hiện, backend cạnh tranh về CRUD và các câu hỏi lý thuyết học thuộc lòng. Giờ đây AI đều làm được những việc này, vừa nhanh vừa không biết mệt, chỉ tốn một ít Token. Bạn mất nửa ngày để cắt một page, AI giải quyết trong mười phút; bạn viết CRUD trong hai giờ, AI nộp bài trong ba phút. Không phải những kỹ năng này vô dụng, mà là chúng không còn khan hiếm nên không còn đáng giá.

Frontend chịu tác động trực tiếp nhất. Tái hiện page, viết component và chỉnh style có mức độ pattern quá cao, đây là loại công việc LLM giỏi nhất. Nhưng thứ biến mất không phải vị trí frontend, mà là frontend “chỉ biết viết page”.

Frontend có năng lực cạnh tranh sẽ đi theo hai hướng: hoặc đào sâu——tối ưu performance, phân tích rendering pipeline, engineering infrastructure, những thứ AI không thể thay thế; hoặc đi vào những lĩnh vực khó——WebGL, data visualization quy mô lớn, nguyên lý bên dưới của cross-platform, chất lượng generate của AI kém, ngược lại trở thành lợi thế phòng thủ.

Backend khá hơn một chút, nhưng cũng đừng lạc quan. AI đã rất giỏi khi viết một API đơn lẻ; điểm yếu của nó là tư duy cấp system——service được tách thế nào, data model được thiết kế ra sao, consistency của cache được đảm bảo thế nào, capacity bottleneck nằm ở đâu. Những việc này cần kết hợp business scenario và technical debt để phán đoán tổng hợp; solution AI đưa ra “trông có vẻ đúng” nhưng lên production lại thất bại.

Năng lực cạnh tranh cốt lõi của backend đang chuyển hướng sang system design, stability governance và complex business modeling.

Bất kể frontend hay backend, có một năng lực đã trở thành basic skill: cộng tác hiệu quả với AI. Không phải chỉ biết dùng ChatGPT là đủ, mà phải có khả năng breakdown vấn đề, dẫn dắt output, đánh giá kết quả có đáng tin hay không và nhận diện security risk. Bạn chuyển từ “người viết code” thành “technical reviewer của AI”.

Những người generate code mà không xem logic có thể đạt efficiency cao trong ngắn hạn, nhưng về dài hạn đang tự chôn mìn——khi production xảy ra vấn đề, họ chỉ biết liên tục hỏi AI mà bản thân không có hướng troubleshooting nào.

### AI có đào thải lập trình viên junior không?

Trong ngắn hạn thì không, nhưng AI sẽ thay đổi hoàn toàn cấu trúc năng lực của lập trình viên junior.

Trước đây giá trị của junior engineer nằm ở:

- Viết CRUD
- Viết API cơ bản
- Viết câu lệnh SQL query
- Viết utility class / config cơ bản

Hiện tại AI đều có thể làm tốt những việc này, thậm chí hiệu quả hơn và ít lỗi hơn. Nhưng junior programmer sẽ không bị đào thải, chỉ là điểm tạo ra giá trị đã chuyển dịch.

Trong tương lai junior engineer cần có:

- **Năng lực breakdown requirement**: chuyển business requirement mơ hồ thành technical task rõ ràng.
- **Năng lực hiểu business**: hiểu domain model và business rule, thay vì chỉ “dịch requirement”.
- **Năng lực nhận thức architecture**: hiểu architecture tổng thể của system, biết code của mình nằm ở đâu trong system.
- **Năng lực diễn đạt Prompt**: có thể mô tả vấn đề chính xác để nhận được câu trả lời chất lượng cao từ AI.

AI làm ngưỡng bước vào lập trình thấp hơn, nhưng lại yêu cầu cao hơn về “năng lực hiểu”. Junior engineer trong tương lai giống một “AI coordinator” hơn, thay vì chỉ là “người viết code”.

Từ góc độ tuyển dụng của doanh nghiệp, nhu cầu về năng lực coding thuần túy sẽ giảm, nhưng nhu cầu về engineer “có thể tận dụng AI để nhanh chóng delivery business value” sẽ tăng.

### Rủi ro lớn nhất do AI mang lại là gì?

Theo tôi chủ yếu có ba cấp độ:

**1. Suy giảm năng lực kỹ thuật**

Phụ thuộc quá mức vào AI sẽ khiến năng lực kỹ thuật của engineer suy giảm, đặc biệt là:

- **Năng lực debugging giảm**: quen để AI troubleshooting vấn đề, hiểu biết của bản thân về nguyên lý bên dưới trở nên nông hơn.
- **Độ nhạy với code giảm**: năng lực phán đoán “code tốt” và “code tệ” yếu đi, thậm chí không biết code tốt là gì.
- **Tư duy architecture suy giảm**: lâu dài chỉ quan tâm đến việc triển khai functionality, bỏ qua thiết kế architecture và khả năng mở rộng.

**2. Mất kiểm soát architecture**

Code AI generate thường chú trọng “functionality hiện tại dùng được”, dễ bỏ qua sức khỏe lâu dài của architecture. Điều này phần lớn bắt nguồn từ **Vibe Coding**, tức dựa vào ý định mơ hồ để AI “tự do phát huy”.

- **Boundary của module mơ hồ**: AI có xu hướng “nhanh chóng hoàn thành functionality”, có thể trộn nhiều responsibility vào cùng một module. Nên làm rõ responsibility của module trước khi coding (Context Boundary theo phong cách DDD), dùng interface contract được định nghĩa trước để giới hạn phạm vi code AI generate.

- **Tích lũy technical debt**: để nhanh chóng triển khai functionality, AI có thể dùng hardcode, bỏ qua standard exception handling, đưa vào các anti-pattern như circular dependency không cần thiết. Những debt này sẽ làm chi phí refactor tăng đáng kể khi quy mô project phát triển.

- **Thiếu nhất quán về style**: code được generate trong các Chat session khác nhau có thể dùng naming convention, error handling pattern và log format khác nhau. Nên thông qua **Spec Coding** để định nghĩa trước technical specification và code style thống nhất (như `.cursorrules`), để AI luôn làm việc theo cùng một bộ rule.

- **Thiếu resource governance**: AI sẽ không tự động xem xét các resource constraint như kích thước connection pool, độ dài queue của thread pool và cache expiration strategy. Ví dụ, code được generate có thể tạo rất nhiều thread nhưng dùng unbounded queue, khiến memory overflow khi traffic tăng đột biến; hoặc dùng config mặc định của database connection pool, trở thành bottleneck khi concurrency cao.

- **Thích ứng với engineering convention**: architecture của code AI generate có thể hợp lý, nhưng thường cần con người kiểm soát việc thích ứng với engineering convention hiện có. Ví dụ, cách tổ chức filename, khác biệt về code style và strategy quản lý dependency——những code “trông có vẻ không vấn đề” này có thể gây rắc rối khi team cộng tác.

**3. Security risk (đặc biệt cần coi trọng)**

- **Lỗ hổng code**: AI có thể generate code chứa security vulnerability, các vấn đề thường gặp gồm:
  - **SQL injection**: dùng string concatenation thay vì parameterized query
  - **XSS**: không HTML escape user input
  - **Thiếu permission check**: thiếu permission check ở cấp API / method
  - **Rò rỉ sensitive information**: in key, Token hoặc password trong log
  - **Lỗ hổng dependency**: đưa vào third-party library đã biết có CVE
- **Rò rỉ data**: sử dụng không đúng cách có thể làm lộ code và business logic của công ty cho external model (đặc biệt là AI service được host trên cloud).
- **Supply chain risk**: dependency package do AI đề xuất có thể chứa lỗ hổng đã biết hoặc malicious code.
- **Rò rỉ key**: code AI generate có thể hardcode key, Token và các sensitive information khác.

**4. Failure mode trong distributed scenario (đặc biệt nguy hiểm)**

Code AI generate trong distributed environment rất dễ bỏ qua các constraint quan trọng, gây ra production incident:

| Failure mode                                   | Vấn đề thường gặp của AI                                                          | Production risk                                                             |
| ---------------------------------------------- | --------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| **Thiếu idempotency**                          | Không xem xét API idempotency, trực tiếp insert hoặc update                       | Network timeout retry gây duplicate data, trừ tiền nhiều lần                |
| **Race condition khi concurrency**             | Thiếu distributed lock hoặc cơ chế CAS                                            | Bán vượt tồn kho, concurrent modification ghi đè, sai lệch số liệu thống kê |
| **Boundary của distributed transaction mơ hồ** | Không làm rõ transaction boundary và rollback strategy                            | Data không nhất quán, một phần thành công một phần thất bại, khó truy vết   |
| **Thiếu timeout và degradation**               | Chỉ đặt timeout mặc định, không có circuit breaker và degradation logic           | Cascade failure, hiệu ứng avalanche, toàn bộ service không khả dụng         |
| **Connection pool leak**                       | Không release connection kịp thời hoặc cấu hình số lượng connection không phù hợp | Connection pool cạn, service giả chết, chỉ có thể khôi phục bằng restart    |

**Case điển hình**: khi AI generate code “trừ tồn kho”, thông thường chỉ viết `UPDATE stock SET count = count - 1 WHERE id = ?`, mà bỏ qua:

- Row lock hoặc distributed lock trong concurrency scenario
- Đảm bảo idempotency khi tồn kho không đủ (một request giống nhau không được trừ nhiều lần)
- Cơ chế compensation khi downstream service timeout
- Database connection timeout và circuit breaker strategy

**Strategy ứng phó**:

- Trong Spec, **ràng buộc rõ ràng**: yêu cầu AI generate code template cho distributed lock, idempotency check và compensation logic
- **Bắt buộc Code Review**: tập trung kiểm tra cross-service call, transaction boundary và nhánh exception handling
- **Xác minh bằng chaos engineering**: thông qua fault injection test để xác minh fault tolerance trong distributed scenario

Doanh nghiệp phải xây dựng security governance system đi kèm:

- **Bắt buộc code review**: code AI generate phải được Review thủ công.
- **Scanning tự động**: tích hợp SAST/SCA tool và bổ sung scanning nhắm vào các risk đặc thù của AI (như git-secrets, TruffleHog).
- **Bảo vệ architecture**: kết hợp với Spec Coding, dùng các tool như ArchUnit để thực hiện automated test cho architecture constraint.

### AI Coding đang khiến lập trình viên mệt hơn, cạnh tranh khốc liệt hơn?

Có người nói: “Tưởng có AI nâng efficiency thì sẽ nhẹ nhàng hơn? Tỉnh lại đi, nó không khiến bạn nhẹ nhàng hơn, nó chỉ khiến sếp nghĩ rằng một mình bạn có thể làm việc bằng ba người.”

Câu này nghe đau, nhưng đúng là cảm nhận thực tế của rất nhiều người.

AI khuếch đại năng lực của bạn. Trước đây viết được ba API một ngày đã thấy mình khá giỏi, giờ một ngày có thể viết mười API, tiện tay còn hoàn thành architecture design, test case và documentation. Dopamine tiết ra điên cuồng, bạn không nhịn được mà nhận thêm việc, vì sự tự tin “mình làm được” đã được AI phóng đại.

Nhưng vấn đề xuất hiện: efficiency càng cao, ham muốn của sếp càng phình to nhanh hơn. Ảo tưởng “một người là cả team” khiến số lượng tuyển dụng bị cắt một nửa trước, những người còn lại bị vắt kiệt. Trước đây bạn chỉ cần đào sâu một module, giờ phải đồng thời xử lý frontend, backend, task đa thread, thậm chí cả đống Agent.

Điều kỳ diệu hơn là vị trí ít đi, việc nhiều lên. Bạn không chỉ phải viết code, mà còn phải review code của AI, sửa Bug của AI, cuối cùng còn phải giải thích với lãnh đạo tại sao code AI generate vừa lên production đã sập. Đôi khi không phân biệt được là mình đang dùng AI hay AI đang dùng mình.

### ⭐ Năng lực cạnh tranh cốt lõi của backend engineer trong 3 năm tới là gì?

Theo tôi trọng tâm của năng lực cạnh tranh cốt lõi sẽ chuyển từ “năng lực viết code” sang bốn khía cạnh sau:

**1. Năng lực system design**

AI rất giỏi generate code cho một functionality đơn lẻ, nhưng **system-level design** vẫn cần engineer dẫn dắt:

- Tách service và phân chia boundary của module
- Cân nhắc giữa microservice và monolithic architecture
- Thiết kế data model và consistency strategy
- Strategy tiến hóa version của API
- Distributed transaction và idempotency design

**2. Năng lực complex business modeling**

Trước đây chúng ta nói AI không giỏi domain modeling, nhưng tình hình hiện tại đã thay đổi. AI đã rất mạnh ở breakdown requirement, sắp xếp rule và mô phỏng scenario.

Tuy nhiên, engineer vẫn cần phối hợp để chuyển business rule thành design có thể thực thi trong project hiện tại:

- Domain-driven design (DDD) modeling
- Trừu tượng hóa business workflow và thiết kế state machine
- Phân chia boundary context

**3. Năng lực performance và stability governance**

Code AI generate thường chỉ chú trọng correctness của functionality, mà bỏ qua đặc tính performance trong production environment:

- **P99 latency**: AI có thể generate N+1 query, SQL chưa thêm index và synchronous blocking call, khiến long-tail latency tăng vọt
- **Memory escape**: object creation và sử dụng closure không phù hợp có thể gây GC thường xuyên, thậm chí OOM
- **Connection pool phình to**: không giới hạn concurrency, không đặt timeout có thể khiến connection pool cạn, gây cascade failure

Engineer cần có năng lực **đo lường và tuning performance**:

- Tối ưu slow query SQL và thiết kế index (dùng EXPLAIN để phân tích execution plan)
- Thiết kế cache strategy và đảm bảo consistency (local cache vs distributed cache)
- Cải tạo theo hướng async và tuning parameter của thread pool (core thread count, queue capacity, reject strategy)
- Service degradation, circuit breaker và rate limiting solution (ứng dụng Sentinel, Hystrix)
- Capacity planning và elastic scaling (đánh giá mức QPS qua load test, tự động scale in/out)

**Phương thức verification**: sau khi code được AI generate, bắt buộc phải dùng load test (JMeter, Gatling) để xác minh P95/P99 latency, dùng JVM monitoring (MAT, Arthas) để điều tra memory leak, thay vì chỉ dựa vào functional test.

**4. Năng lực cộng tác với AI**

Việc cộng tác hiệu quả với AI tự thân đã là một năng lực cạnh tranh cốt lõi:

- **Diễn đạt requirement chính xác (năng lực Prompt)**: dùng structured Prompt (background-task-constraint-output format), tránh instruction mơ hồ
- **Chia nhỏ vấn đề và dẫn dắt AI**: chia task phức tạp thành các subtask có thể verify độc lập, dùng Chain-of-Thought để dẫn dắt reasoning
- **Đánh giá chất lượng output của AI**: xây dựng code Review checklist, chú ý correctness, security, performance và maintainability
- **Kiểm tra security và compliance của code**: nắm OWASP Top 10, có thể nhận diện security risk trong code AI generate
- **Kết hợp AI toolchain**: nắm cấu hình và cách sử dụng `.cursorrules`, custom Skills và IDE plugin

Về bản chất, đây là sự chuyển đổi vai trò từ “người viết code” sang “AI collaboration engineer”.

Chìa khóa của cạnh tranh trong tương lai không còn là “tốc độ tạo code”, mà là “chất lượng system design” và “năng lực delivery business value”.

## Tổng kết

AI Coding tool đang thay đổi sâu sắc phương thức làm việc của developer. Các tool như Cursor, Claude Code và Trae đã phát triển từ code completion thành engineering assistant có thể cộng tác sâu.

Từ Prompt đến Harness, chỉ trong bốn năm ngắn ngủi, việc viết code đang chuyển từ “tay nghề” của programmer thành “standard operation” của Agent. Có người nói: “Trong tương lai, có thể chỉ một CTO là quản lý được tất cả Agent, để chúng tạo ra toàn bộ code, deploy và sửa bug.” Câu này nghe khá cực đoan, nhưng nghĩ kỹ thì dường như cũng không hoàn toàn bất khả thi.

**Thứ thực sự quyết định sự nghiệp của bạn là cách bạn sử dụng những tool này và liệu trong quá trình sử dụng, bạn có duy trì tư duy sâu về kỹ thuật hay không.**

Nói thật, từ thời điểm này năm ngoái tôi đã khá lo lắng về sự phát triển của AI, đặc biệt là hướng Coding. Đến hôm nay, tốc độ tiến hóa nhanh đến vậy lại khiến tôi phần nào nhẹ nhõm. Biết viết code đang chuyển từ core skill thành basic literacy, giống như biết dùng Excel không được xem là năng lực cạnh tranh. Thứ thực sự có giá trị là định nghĩa vấn đề, thiết kế solution, kiểm soát quality và delivery business value.

Cuối cùng là một vài lời khuyên dành cho những người đang chuẩn bị phỏng vấn:

1. **Chỉ khi đã dùng thực tế mới có thể trả lời tốt**: khi interviewer hỏi về AI Coding tool, điều họ ngại nhất là “chỉ nghe nói chứ chưa dùng”. Dù chỉ dùng Cursor viết vài project nhỏ cũng tốt hơn chỉ xem tutorial.
2. **Xây dựng phương pháp của riêng mình**: đừng chỉ “biết dùng”, hãy có kinh nghiệm sử dụng và best practice của riêng mình, đây là điểm cộng trong phỏng vấn.
3. **Duy trì tư duy phản biện**: sau khi AI generate code bắt buộc phải Review, đây là basic literacy. Thể hiện thái độ này trong phỏng vấn sẽ khiến interviewer thấy bạn là một engineer đáng tin cậy.
4. **Theo dõi xu hướng kỹ thuật nhưng không lo âu**: AI sẽ thay đổi nhiều thứ, nhưng các năng lực cốt lõi như system design, tư duy architecture và hiểu business sẽ không lỗi thời.

Tận dụng tốt AI tool + duy trì tư duy độc lập, hai điều này không thể thiếu điều nào. Trong thời đại AI, biết đâu tương lai của programmer sẽ tỏa sáng trong mọi ngành nghề. Cùng cố gắng!
