---
title: Trong thời đại model mạnh, AI Coding Skills còn cần thiết không?
description: Từ trải nghiệm thực tế với Codex, Superpowers và grilling, thảo luận những Skill nào có thể xoá trong thời đại model mạnh, và workflow nào vẫn đáng duy trì lâu dài.
category: AI Coding thực chiến
tag:
  - AI Coding
  - Codex
  - Skills
  - Superpowers
head:
  - - meta
    - name: keywords
      content: AI Coding,Codex,Skills,Superpowers,grilling,AGENTS.md,Subagent,Plugin
---

Vài ngày trước, tôi thấy một câu hỏi trên Zhihu: **Sau khi Codex dùng GPT-5.6, Skills còn cần thiết đến mức nào?**

![Câu hỏi liệu còn cần Skills sau khi Codex sử dụng GPT-5.6](https://oss.javaguide.cn/github/javaguide/ai/skills/zhihu-codex-gpt56-skills-question.png)

Một câu hỏi và thay đổi trong cách tôi sử dụng không thể đại diện cho xu hướng của ngành. Nói chính xác hơn, tôi nhận thấy lợi ích của một phần Skill dành cho development đang giảm, nên muốn kiểm tra lại cái nào nên giữ, cái nào nên xoá.

Hiện nay khi thấy một Skill, trước tiên tôi sẽ xem nó thực sự cung cấp năng lực gì. Nếu không có tác dụng cụ thể thì chắc chắn tôi sẽ không cài.

Hai năm trước, model thường bỏ sót các bước khi làm việc, Skill viết càng chi tiết càng khiến người ta yên tâm. Hiện nay, các model như GPT-5.6, Kimi K3, Claude Fable 5 đã có thể hoàn thành không ít thao tác cơ bản, nên tôi cũng bắt đầu kiểm tra lại các Skill mình đang có.

> Dưới đây tôi sẽ lấy Codex làm Coding Agent để thảo luận, các trường hợp khác cũng tương tự.

## Nhiều bước cơ bản không còn cần dạy riêng

Trước đây, Skill dành cho development thường được viết thành một checklist thao tác: đọc project thế nào, tìm call chain ra sao, sửa code xong chạy test nào, trước khi tạo PR cần kiểm tra gì. Khi năng lực của model chưa đủ ổn định, những lời nhắc này thực sự hữu ích.

Hiện nay, khi yêu cầu Codex sửa một Bug, chỉ cần hiện tượng, kỳ vọng và tiêu chí nghiệm thu đủ rõ, nó thường có thể tự đọc code liên quan, lần theo call chain để định vị vấn đề, rồi hoàn thành implementation và verification cơ bản. Khi cấu trúc project phức tạp, entry point của test đặc biệt hoặc rủi ro thay đổi cao, vẫn cần nói rõ phải chạy những kiểm tra nào, không thể hoàn toàn để model tự đoán cách verification.

Với long task, approval và phối hợp nhiều Agent, bản thân Codex cũng cung cấp năng lực tương ứng. Khi cần mở rộng task flow hoặc năng lực bên ngoài, còn có thể sử dụng Skills, Plugins, MCP và Hooks. Tình trạng trước đây kiểu “không viết đầy đủ từng bước thì rất dễ đi lệch hướng” đã giảm đi nhiều.

![Claude Code PreToolUse Hook](https://oss.javaguide.cn/github/javaguide/ai/coding/claude-code-runs-rm-rf-tmp-build-what-happens.svg)

Nếu một `SKILL.md` không có constraint đặc thù của project, cũng không có script, template và mục kiểm tra, mà chỉ lặp lại các bước development thông thường, tôi thường sẽ không giữ lại. Nó không thêm được bao nhiêu thông tin mới cho model, nhưng có thể khiến một task nhỏ phải đi qua thêm vài bước.

## Cài quá nhiều Skill cũng có chi phí

Codex không đọc toàn bộ nội dung của mọi `SKILL.md` khi bắt đầu session. Trước tiên, nó nhận tên, description và path của từng Skill; chỉ sau khi task được match mới load nội dung chính. Cơ chế này gọi là progressive disclosure.

Theo [tài liệu Skills của Codex](https://learn.chatgpt.com/docs/build-skills), danh sách Skill ban đầu tối đa sử dụng 2% context window của model; khi không xác định được kích thước window, giới hạn là 8.000 ký tự. Sau khi vượt budget, Codex sẽ rút ngắn description trước. Khi số lượng tiếp tục tăng, một phần Skill có thể bị loại khỏi danh sách ban đầu.

Khi cài đến 100 Skill, thứ Agent nhìn thấy trước khi bắt đầu làm việc có thể đã không còn là 100 description đầy đủ.

Nếu description viết quá rộng, một thay đổi thông thường cũng có thể match với vài Skill. Khi các rule xung đột, Codex còn phải phán đoán hiện tại nên thực hiện workflow nào. Nếu trộn thêm vài tài liệu lâu ngày không được maintain, sau khi task đi lệch hướng sẽ rất khó tìm ra nguyên nhân ngay.

Context window lớn hơn cũng không xoá được vấn đề này. Old conversation, tool description và Skill description đều có thể đưa vào, nhưng constraint thực sự quan trọng của project thường chỉ có vài câu. Nội dung càng tạp, yêu cầu quan trọng càng dễ bị nhấn chìm.

![Vì sao context mất hiệu lực](https://oss.javaguide.cn/github/javaguide/ai/context-engineering/why-does-the-following-content-fail.png)

Skill khá giống bookmark trên trình duyệt. Ban đầu thấy gì cũng muốn lưu, luôn cảm thấy sau này sẽ cần. Nửa năm sau nhìn lại, những thứ thường dùng vẫn chỉ có vài cái.

Từ khi Skill xuất hiện đến nay, số Skill tôi thường dùng và sẵn sàng maintain lâu dài chưa đến 20. Ví dụ, khi viết lách tôi thường dùng [draw.io Drawing Skill](https://mp.weixin.qq.com/s/rAKCSFHB407v6fe35ix0rg), nó có thể cố định style của chart, đồng thời tránh các vấn đề layout tôi thường xuyên gặp phải. Những Skill như vậy tích luỹ preference cá nhân và kinh nghiệm cụ thể, rất khó để model tự xử lý mà luôn cho ra kết quả giống nhau.

## Rule nên đặt ở đâu

Nhiều Skill ngày càng dài là vì mọi người nhồi tất cả rule vào trong đó. Hiện nay tôi sẽ phân loại theo phạm vi tác dụng của rule:

- Convention của project phải tuân thủ trong mọi task thì đặt vào `AGENTS.md` hoặc file rule của project.
- Workflow chỉ dùng trong task cụ thể thì viết thành Skill.
- Nhánh điều tra tốn nhiều thời gian, tạo ra lượng lớn thông tin trung gian thì giao cho Subagent.
- Khi cần phân phối thống nhất Skills, Hooks, MCP và connector cho team thì đóng gói thành Plugin.
- Constraint mang tính cơ học, chỉ cần bỏ sót một lần là có thể xảy ra vấn đề, thì giao cho Hook, CI, linter hoặc test.

File rule quản lý cách project này luôn được thực hiện, Skill quản lý cách thực hiện khi gặp một loại task nào đó. Trộn hai thứ lại với nhau, cuối cùng thường sẽ tạo ra một `SKILL.md` rất dài, muốn quản lý mọi thứ nhưng lại khó maintain.

Skill có thể mang theo script, tài liệu tham khảo và template, được load khi match task và có nhu cầu. Khi team cần cài đặt và phân phối thống nhất, có thể dùng Plugin để đóng gói các năng lực này.

![Progressive disclosure (mô hình ba lớp)](https://oss.javaguide.cn/github/javaguide/ai/skills/skills-progressive-disclosure-three-layer-model.png)

Nếu muốn tìm hiểu có hệ thống về sự phân công giữa Skill với Prompt, MCP và Function Calling, có thể xem [Agent Skills là gì? Khác Prompt và MCP chính xác ở đâu?](https://javaguide.cn/ai/agent/skills.html). Bài viết này chỉ thảo luận cách chọn và xoá, không lặp lại phần triển khai kỹ thuật.

## Vì sao tôi hiếm khi còn dùng Superpowers

Các bộ Skills bao phủ toàn bộ development workflow như [Superpowers](https://github.com/obra/superpowers), hiện nay tôi dùng ít hơn.

Tôi đưa vấn đề này vào group để thảo luận, phản hồi của mọi người cũng khá giống nhau: **Superpowers dễ khiến task nhỏ phải gánh workflow quá nặng; `grilling` tuy sẽ liên tục hỏi thêm, nhưng thực sự có thể làm rõ requirement hơn.**

![Thảo luận của thành viên group về workflow quá nặng của Superpowers và việc Grilling giảm rework](https://oss.javaguide.cn/github/javaguide/ai/skills/group-chat-superpowers-grilling-feedback.png)

Superpowers cung cấp một phương pháp phát triển software hoàn chỉnh: trước tiên làm rõ requirement bằng brainstorming, sau đó dùng writing-plans để chia task, viết test và implementation theo test-driven-development, cô lập development bằng Git worktree, giao cho Subagent thực thi theo từng phần, cuối cùng tiến hành code review và verification trước khi hoàn tất.

Project phức tạp, codebase xa lạ và thay đổi rủi ro cao vẫn phù hợp với workflow này.

Nếu chỉ sửa một đoạn validation logic hoặc bổ sung một test rất nhỏ, ngay từ đầu đã đi hết “làm rõ requirement → design → plan → execution → review → verification” thì thời gian rất dễ bị chính workflow tiêu tốn. Workflow càng đầy đủ càng cần cân nhắc thời điểm bật; khi task chưa đủ phức tạp, nó sẽ chuyển từ bảo vệ thành gánh nặng.

Hiện nay Codex có thể điều chỉnh các bước trong quá trình execution, đồng thời sẽ yêu cầu xác nhận khi thiếu thông tin quan trọng. Nhiều task nhỏ chỉ cần bổ sung vài constraint của project, không cần mỗi lần đều áp dụng cả một cuốn hướng dẫn thao tác.

Third-party Skill còn có security risk. Bản thân `SKILL.md` là instruction dành cho Agent; nếu bên trong ẩn command nguy hiểm, script bất thường hoặc yêu cầu permission quá rộng, Agent có thể thực sự làm theo. Trước khi cài, ít nhất hãy xem qua `SKILL.md`, `scripts/` và `references/`; bộ toolkit càng lớn, càng phải làm rõ trước nó sẽ khiến Agent làm gì.

## Tôi thích tổ hợp nhẹ như mattpocock/skills hơn

Superpowers dùng một phương pháp hoàn chỉnh để bao phủ development process, còn [mattpocock/skills](https://github.com/mattpocock/skills) giống một toolkit có thể tách ra sử dụng hơn.

Các Skill trong repository này được thiết kế thành những module nhỏ, dễ sửa và có thể kết hợp. Khi requirement chưa được nghĩ rõ, có thể dùng `grilling` để hỏi thêm; Bug khó định vị thì bật thêm `diagnosing-bugs`; cần nghiêm ngặt thực hiện red-green-refactor thì dùng riêng `tdd`; chuẩn bị merge code thì để `code-review` kiểm tra. Chúng không mặc định yêu cầu mọi task phải đi hết cùng một workflow.

Cách tách này rất phù hợp với model mạnh hiện nay. Những bước Codex đã có thể hoàn thành thì không cần dạy lại; khâu nào liên tục xảy ra lỗi thì chỉ bổ sung khâu đó. Khi task trở nên phức tạp hơn, lại kết hợp vài Skill với nhau.

![Trải nghiệm của thành viên group về Lightweight Skills và Superpowers](https://oss.javaguide.cn/github/javaguide/ai/skills/group-chat-lightweight-skills-feedback.png)

Trong đó, tôi đặc biệt thích [`grilling`](https://github.com/mattpocock/skills/blob/main/skills/productivity/grilling/SKILL.md).

Rule của nó rất ngắn: trước khi bắt tay làm thì liên tục hỏi thêm, hỏi rõ plan, decision và dependency; mỗi lần chỉ hỏi một câu, chờ user trả lời rồi mới tiếp tục; tự kiểm tra các fact có thể tra được từ environment, còn decision cần trade-off thì giao cho user.

## Những Skill nào vẫn đáng giữ

Hiện nay tôi sẽ ưu tiên giữ lại ba loại Skill.

Loại thứ nhất là preference cá nhân và artifact cố định mà model khó tự đoán, chẳng hạn style bài viết, quy chuẩn chart, template nội bộ của công ty và release workflow của codebase cụ thể. Với các constraint này, mỗi lần delivery mới có thể cố gắng giữ cùng một tiêu chuẩn.

Loại thứ hai là task có professional judgment, script hoặc tài liệu tham khảo. Security review, xử lý document phức tạp, migration framework cụ thể và production check rất khó được bao phủ hết chi tiết chỉ bằng một Prompt. Skill có thể đặt các mục kiểm tra, tool script và nguồn bằng chứng cùng một chỗ, khi cần mới load.

Loại thứ ba là workflow có thể giảm sai hướng. `grilling` thuộc loại này: nó không thay Codex viết code, chỉ cố định việc xác nhận requirement trước khi bắt đầu. Model thực thi càng nhanh thì loại workflow “xác nhận hướng đi trước” càng đáng giữ.

## Tôi dùng grilling để làm rõ một requirement thực tế

Case này đến từ open source project [《Nền tảng phỏng vấn thông minh SpringAI》 (đã open source version 2.0)](https://javaguide.cn/zhuanlan/interview-guide.html). Khi đó tôi chuẩn bị kết nối mock interview với knowledge base, task giao cho `grilling` cũng rất trực tiếp: giúp tôi nghĩ rõ việc này.

Implementation hiện tại gần với “kết nối” hơn tôi dự đoán: knowledge base interview và mock interview thông thường đều đang sử dụng `InterviewSession`, phần trả lời, đánh giá và một số trang frontend cũng đã được dùng chung. Lần này chưa cần sửa tầng dưới trước, mà phải xác định product scope của phase đầu.

![Dùng Grilling để xác nhận phương án kết nối mock interview với knowledge base](https://oss.javaguide.cn/github/javaguide/ai/skills/grilling-springai-interview-platform-case.png)

Câu hỏi đầu tiên nó đặt ra là phase đầu nên làm “interview định hướng hoàn toàn dựa trên user profile”, hay để user vẫn chọn các Skill như Java, system design, còn knowledge base chỉ phụ trách bổ sung context.

Nó đề xuất làm phương án trước. Question bank generation, category, difficulty, fixed follow-up và rule chấm điểm hiện tại đều gần với chain này hơn; chỉ cần thống nhất entry point và history là có thể chạy thông. Phương án sau còn dẫn đến tỷ lệ trộn giữa câu hỏi Skill và câu hỏi knowledge base, RAG realtime, xung đột nguồn, căn cứ đánh giá và deduplication câu hỏi, phạm vi cải tạo sẽ lớn hơn nhiều.

Sau khi tôi xác nhận mục tiêu phase đầu, nó mới hỏi tiếp: một interview chỉ chọn một knowledge base, hay cho phép kết hợp nhiều knowledge base? Request hiện tại, field của session và việc filter question bank đều chỉ có một `knowledgeBaseId`, nên nó đề xuất phase đầu trước tiên giới hạn một knowledge base, đợi workflow ổn định rồi mới cân nhắc liên kết nhiều knowledge base.

Tiếp theo là entry point. Knowledge base interview đã có page độc lập, còn mock interview thông thường đi vào từ “trung tâm mock interview”. Cuối cùng xác định hai entry point cùng tồn tại, nhưng dùng chung một bộ config component và create API để tránh maintain hai bộ interaction logic.

Code còn chưa bắt đầu sửa, nhưng product scope, data model và cách reuse entry point đã được xác định.

Tôi muốn giữ lại `grilling`, vì model đã viết code đủ nhanh, phần lớn rework xảy ra do bắt đầu quá sớm: requirement scope chưa chốt, exception handling chưa trao đổi, user scenario và technical trade-off còn rất mơ hồ. Agent làm xong một mạch theo cách hiểu của mình, cuối cùng có thể vẫn phải làm lại từ đầu.

**Model càng mạnh, execution càng nhanh, cái giá của việc đi sai hướng cũng càng lớn.**

`grilling` không dạy model cách viết code, nó chỉ cố định “hỏi rõ trước khi bắt tay làm” thành một workflow có thể thực thi lặp lại. Model mạnh không tự động xoá bỏ giá trị của workflow này.

## Tiêu chuẩn cắt giảm Skill của tôi

Hiện nay mỗi khi cài một Skill, tôi sẽ hỏi thêm vài câu:

- Việc này vốn dĩ model đã làm được chưa?
- Khi không có nó, tôi có liên tục vấp ở cùng một chỗ không?
- Nó có tích luỹ script, template, tài liệu chuyên môn hoặc preference cá nhân không?
- Sau khi rule hết hiệu lực, tôi có thể kịp thời phát hiện và xoá nó không?

Các Skill chỉ nhắc “đọc project trước, viết code sau, cuối cùng chạy test” hiện nay tôi gần như xoá ngay. Nếu project thực sự có yêu cầu đặc biệt, viết vào `AGENTS.md` sẽ phù hợp hơn; phần test, format và security restriction có thể thực thi cơ học thì giao cho CI, Hook hoặc linter.

Chỉ khi liên tục vấp ở cùng một chỗ mới đáng viết riêng một Skill, đặc biệt là với task có cost of error cao. Chỉ giữ lại judgment quan trọng và action verification bên trong, giải quyết được vấn đề thì dừng, không tiện tay mở rộng thành một workflow lớn và toàn diện.

Muốn tìm hiểu hiện có những Skill nào và chúng phù hợp với task nào, có thể xem tiếp [Danh sách chọn AI Coding Skills](https://javaguide.cn/ai-coding/practices/programmer-essential-skills.html). Về sự phân công giữa `AGENTS.md`, permission, MCP, Skills và Scheduled Tasks trong Codex, có thể tham khảo [Hướng dẫn best practices của OpenAI Codex](https://javaguide.cn/ai-coding/practices/codex-best-practices.html).

Dù tiêu đề bài viết có viết “còn cần thiết không”, tôi không định xoá toàn bộ Skill. Tôi chỉ sẽ không thấy một cái là cài một cái nữa.

Hiện nay khi gặp Skill mới, trước tiên tôi sẽ không dùng nó và chạy thử một lần. Nếu vẫn làm tốt thì không cài; nếu cùng một vấn đề lặp lại nhiều lần, tôi mới giữ lại đoạn workflow nhỏ đó. Sau khi dọn dẹp như vậy, danh sách có thể ngắn đi đáng kể, nhưng tôi biết rõ vì sao mỗi Skill vẫn còn ở đó.
