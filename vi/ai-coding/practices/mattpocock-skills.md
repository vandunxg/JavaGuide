---
title: "mattpocock/skills: 4 Agent Skill về AI coding tôi khuyên dùng nhất"
description: "Giới thiệu chi tiết grilling, research, diagnosing-bugs và code-review trong mattpocock/skills, kết hợp các ví dụ dự án thực tế để giải thích chúng phù hợp giải quyết những điểm thất bại nào của AI coding, cũng như cách cài đặt và sử dụng theo nhu cầu."
category: AI Coding thực chiến
tag:
  - AI Coding
  - Skills
  - Codex
  - Claude Code
head:
  - - meta
    - name: keywords
      content: AI coding,Agent Skills,mattpocock skills,grilling,research,diagnosing-bugs,code-review,Codex,Claude Code,AI-assisted development,code review,làm rõ yêu cầu,chẩn đoán Bug
---

Xin chào, tôi là G nhỏ.

Trong hai bài viết [Danh sách lựa chọn AI coding Skill](https://javaguide.cn/ai-coding/practices/programmer-essential-skills.html) và [Trong thời đại model mạnh, còn cần cài AI coding Skill không?](https://javaguide.cn/ai-coding/practices/skill-selection-and-pruning.html), tôi đều đã đề cập đến [mattpocock/skills](https://github.com/mattpocock/skills), trong đó `grilling` còn được lấy làm ví dụ từ một dự án thực tế.

Có khá nhiều độc giả quan tâm đến `grilling`. Tuy nhiên, nhìn lại thì cả hai bài đều viết quá sơ lược. Bài viết chỉ để lại ấn tượng “để Agent liên tục đặt câu hỏi”, còn việc mỗi lần chỉ hỏi một câu, thông tin nào nên để Agent tự tìm hiểu, khi nào mới có thể bắt đầu thực hiện đều chưa được trình bày.

Vì vậy tôi đã đọc lại `SKILL.md` trong repository. `mattpocock/skills` chia các vấn đề engineering thường gặp thành những Skill nhỏ hơn, dễ chỉnh sửa và có thể kết hợp: khi yêu cầu mơ hồ thì bổ sung quy trình làm rõ yêu cầu, khi Bug khó tìm thì bổ sung quy trình chẩn đoán, khi chuẩn bị bàn giao thì bổ sung code review.

Cách phân chia này khá phù hợp với thói quen sử dụng của tôi. Những thao tác cơ bản mà Codex, Claude Code đã có thể hoàn thành ổn định thì không cần dạy lại mỗi lần; khâu nào thường xuyên phải làm lại thì thêm một đoạn quy trình nhỏ vào khâu đó.

Khi thảo luận trong nhóm, mọi người cũng đề cập đến vấn đề tương tự: bộ công cụ đầy đủ dễ khiến các task nhỏ phải gánh quy trình quá nặng, `grilling` tuy liên tục đặt câu hỏi nhưng quả thật có thể làm rõ yêu cầu hơn.

![Thảo luận trong nhóm về quy trình Superpowers quá nặng và Grilling giúp giảm việc làm lại](https://oss.javaguide.cn/github/javaguide/ai/skills/group-chat-superpowers-grilling-feedback.png)

Ngoài `grilling`, ba Skill `research`, `diagnosing-bugs` và `code-review` cũng rất đáng chú ý; bài viết này sẽ lần lượt chia sẻ về chúng.

## `grilling` không chỉ là để Agent hỏi thêm vài câu

Trong hai bài viết trước, thực ra tôi cũng đã viết quá đơn giản về `grilling`: bảo Agent đừng vội viết code mà hãy hỏi thêm vài câu trước. Chỉ cần thêm câu này vào Prompt thông thường cũng có thể làm được.

Phiên bản hiện tại của [`grilling`](https://github.com/mattpocock/skills/blob/main/skills/productivity/grilling/SKILL.md) rất ngắn, nhưng quy định khá chi tiết cách cuộc phỏng vấn tiếp tục.

![Quy tắc phỏng vấn đầy đủ của grilling Skill](https://oss.javaguide.cn/github/javaguide/ai/skills/grilling-skill-content.png)

Nó sẽ đặt câu hỏi theo decision tree, mỗi lần chỉ xử lý một quyết định. Câu trả lời trước đó có thể thay đổi các nhánh phía sau, vì vậy không thể cùng lúc ném ra hơn mười câu hỏi và để người dùng trả lời như điền bảng khảo sát.

Nó cũng tách facts và decisions. Project sử dụng framework nào, API hiện tại được thiết kế ra sao, database có một field nào đó hay không là những việc Agent nên tự đọc code và tài liệu. Giai đoạn đầu nên làm phương án nào, có cần tương thích với behavior cũ hay không, sẵn sàng chấp nhận bao nhiêu complexity thì giao cho người dùng.

Trước khi hai bên xác nhận đã đạt được common understanding, Agent cũng không được dựa theo phán đoán của mình để bắt đầu làm việc.

## `grilling`, `grill-me` và `grill-with-docs` khác nhau thế nào?

`grilling` là Skill phỏng vấn nền tảng có thể tái sử dụng; model có thể chủ động gọi, người dùng cũng có thể gọi trực tiếp, các Skill khác cũng có thể tái sử dụng nó. `/grill-me` là entrypoint thủ công rõ ràng hơn, bản thân nó chỉ chịu trách nhiệm khởi động một session `/grilling`.

Khi cuộc thảo luận tạo ra các domain term hoặc architectural decision cần sử dụng lâu dài, có thể chuyển sang [`/grill-with-docs`](https://github.com/mattpocock/skills/blob/main/skills/engineering/grill-with-docs/SKILL.md). Nó còn gọi `domain-modeling`: sau khi xác định thuật ngữ thì ghi vào `CONTEXT.md`, còn một số quyết định khó đảo ngược và sau này có thể trông kỳ lạ thì ghi lại thành ADR.

![Quan hệ gọi giữa các Skill của grill-with-docs](https://oss.javaguide.cn/github/javaguide/ai/skills/grill-with-docs-skill-content.png)

![Quy tắc domain modeling của domain-modeling Skill](https://oss.javaguide.cn/github/javaguide/ai/skills/domain-modeling-skill-content.png)

Có thể hiểu quan hệ giữa ba Skill như sau:

```text
grill-me ──────────> grilling
grill-with-docs ───> grilling + domain-modeling
```

Các quy tắc phỏng vấn tập trung trong `grilling`, các Skill khác trực tiếp tái sử dụng chúng.

[v1.1.0](https://github.com/mattpocock/skills/releases/tag/v1.1.0) lại chuyển bước xác nhận thành điều kiện dừng rõ ràng, đồng thời phân biệt facts của environment và decisions của người dùng. Quy tắc cũ có thể khiến Agent gọi kết hợp nó thuận tay thay người dùng quyết định về product; hiện nay những quyết định kiểu này bắt buộc phải lần lượt hỏi người dùng.

## Tôi đã dùng grilling để xác nhận yêu cầu về một buổi phỏng vấn knowledge base

Lần sử dụng thực tế này đến từ project open source [Nền tảng phỏng vấn thông minh SpringAI](https://javaguide.cn/zhuanlan/interview-guide.html) của tôi.

Khi đó tôi chuẩn bị kết nối mock interview với knowledge base, nên trực tiếp chọn `grilling`, task đưa ra chỉ có một câu: giúp tôi suy nghĩ rõ việc này.

Implementation hiện tại thực ra đã kết nối được một phần: mock interview thông thường và knowledge base interview đều sử dụng `InterviewSession`, việc trả lời, đánh giá bất đồng bộ và một phần các trang frontend cũng đã được dùng chung. Nếu tiếp tục thiết kế tầng dưới vào lúc đó, có thể sẽ sửa mất phần code vốn có thể giữ lại, trong khi phạm vi product cho giai đoạn đầu vẫn chưa được xác định.

![Dùng Grilling để xác nhận phương án kết nối mock interview với knowledge base](https://oss.javaguide.cn/github/javaguide/ai/skills/grilling-springai-interview-platform-case.png)

Quyết định đầu tiên mà `grilling` hỏi là knowledge base đóng vai trò gì trong interview.

Một phương án là hoàn toàn tạo interview theo hướng mục tiêu dựa trên profile người dùng; phương án kia là vẫn chọn các Skill như Java, system design..., còn knowledge base chỉ bổ sung context. Tôi chọn phương án đầu tiên, nhờ đó question bank generation, phân loại, độ khó, các câu hỏi follow-up cố định và rule chấm điểm hiện tại đều có thể tiếp tục sử dụng. Phương án sau sẽ còn dẫn đến tỷ lệ trộn giữa hai loại câu hỏi, RAG realtime, loại trùng câu hỏi, xung đột nguồn và căn cứ đánh giá.

Sau khi xác nhận quyết định này, nó mới đi vào nhánh tiếp theo: một interview gắn với một knowledge base hay cho phép kết hợp nhiều knowledge base?

Request parameter, field của session và việc filter question bank đều xoay quanh một `knowledgeBaseId`. Nhiều knowledge base còn phải xử lý việc hợp nhất kết quả retrieval, weight, nội dung trùng lặp và kiểm tra permission. Vì vậy giai đoạn đầu được giới hạn ở một knowledge base, không sửa trước association table và cấu trúc API.

Quyết định thứ ba là entrypoint. Knowledge base interview đã có page riêng, còn mock interview thông thường đi vào từ “trung tâm mock interview”. Cuối cùng giữ lại hai entrypoint, nhưng tầng dưới vẫn dùng chung `InterviewSession`, component cấu hình và API tạo cũng cố gắng dùng chung để tránh sau này phải maintain hai bộ logic tương tự.

Code vẫn chưa bắt đầu sửa, nhưng phạm vi giai đoạn đầu đã thu gọn thành ba lựa chọn: knowledge base interview thuần túy, một `knowledgeBaseId`, hai entrypoint dùng chung session và khả năng tạo.

`grilling` không viết product plan thay tôi. Nó đưa ra câu trả lời đề xuất và căn cứ từ code, còn việc đánh đổi vẫn do tôi xác nhận. Nếu câu trả lời đầu tiên đổi thành “Generic Skill + knowledge base context”, câu hỏi phía sau cũng sẽ không phải là một knowledge base hay nhiều knowledge base, mà sẽ chuyển sang cách trộn và đánh giá hai loại câu hỏi.

Hiện nay model viết code đã đủ nhanh. Khi phạm vi yêu cầu chưa được xác định, Agent cũng có thể nhanh chóng giao ra code, test và tài liệu. Nếu hướng đi sai, tất cả những sản phẩm này đều phải làm lại.

Tất nhiên, không phải task nào cũng cần trải qua một vòng “thẩm vấn” trước. Sửa một đoạn văn bản, bổ sung một kiểm tra null rõ ràng, thêm field theo pattern có sẵn đều đã có tiêu chí nghiệm thu rất cụ thể; làm trực tiếp thường tiết kiệm thời gian hơn. `grilling` cũng không thay thế test và code review, nó chỉ chịu trách nhiệm làm lộ ra những vấn đề chưa được quyết định trước khi bắt tay thực hiện.

`grilling` hiện tại cũng không giới hạn số lượng câu hỏi, nên yêu cầu phức tạp có thể được thảo luận khá lâu. Nếu lo cuộc phỏng vấn kéo dài quá lâu, có thể đặt budget 3–5 câu hỏi cho mỗi round. Sau khi round kết thúc, trước tiên tổng hợp các decision đã xác nhận và chưa xác nhận, sau đó để người dùng chọn có tiếp tục hay không.

Đừng chỉ viết “tối đa 5 câu hỏi”. Sau khi dùng hết budget, Agent vẫn không được tự bổ sung các decision còn lại hoặc trực tiếp bắt đầu làm việc.

## `research`: giao nhánh tra cứu tài liệu cho Agent

Khi upgrade một SDK của project, những việc như version hiện tại hỗ trợ parameter nào, API cũ bị deprecated từ khi nào, streaming event thay đổi ra sao không nên dựa vào trí nhớ của người dùng trả lời, cũng không phù hợp để main Agent vừa sửa code vừa lật tìm tài liệu dài.

[`research`](https://github.com/mattpocock/skills/blob/main/skills/engineering/research/SKILL.md) giao vấn đề cho Agent chạy background, chỉ tra official docs, source code, spec và first-party API. Kết luận được ghi vào một file Markdown trong repository và nêu rõ source, còn main Agent có thể tiếp tục xử lý công việc khác.

![Quy tắc tra cứu tài liệu và lưu kết quả của research Skill](https://oss.javaguide.cn/github/javaguide/ai/skills/research-skill-content.png)

Điều tôi đánh giá cao là nó cố định source tài liệu và deliverable: không dùng tutorial thứ cấp thay cho tài liệu chính thức, cũng không nhồi hàng chục trang quá trình tìm kiếm vào main session, mà chỉ giữ lại các kết luận có thể kiểm tra lại.

Sử dụng nó cần hai điều kiện: Agent hỗ trợ điều tra background hoặc Subagent, đồng thời project chấp nhận thêm một research document. Nếu chỉ tra một method signature thì mở thẳng official docs sẽ nhanh hơn; khi liên quan đến migration version, khác biệt protocol hoặc dependency xa lạ, hãy giao nhánh này cho Agent.

## `diagnosing-bugs`: trước tiên tạo một feedback loop có thể chuyển sang đỏ

Khi điều tra Bug, Agent rất dễ hình thành phán đoán quá sớm. Vừa thấy một branch đáng ngờ là lập tức sửa code rồi chạy lại test; nếu chưa sửa được thì tiếp tục đổi sang phán đoán khác. Thay đổi ngày càng nhiều, còn hiện tượng lỗi ban đầu lại không được reproduce ổn định.

[`diagnosing-bugs`](https://github.com/mattpocock/skills/blob/main/skills/engineering/diagnosing-bugs/SKILL.md) dành phần lớn effort cho giai đoạn đầu: trước tiên tạo một feedback loop có thể bắt chính xác Bug hiện tại.

![Quy trình chẩn đoán Bug của diagnosing-bugs Skill](https://oss.javaguide.cn/github/javaguide/ai/skills/diagnosing-bugs-skill-content.png)

Feedback loop có thể là một failed test, một đoạn `curl`, một CLI với input cố định, script Playwright hoặc replay request production. Nó phải bắt được lỗi ban đầu, chạy ổn định, đủ nhanh và Agent có thể tự thực thi.

Khi thực sự không thể reproduce, nó sẽ liệt kê các cách đã thử, sau đó xin người dùng cung cấp môi trường có thể reproduce, HAR, log, `core dump` hoặc permission instrumentation tạm thời trên production.

Sau khi chuẩn bị xong feedback loop, tiếp tục reproduce và thu nhỏ input. Tiếp đó liệt kê 3–5 hypothesis có thể bác bỏ, giải thích “nếu đây là nguyên nhân thì sau khi thay đổi điều gì, hiện tượng sẽ thay đổi ra sao”, rồi thêm breakpoint hoặc log có mục tiêu dựa trên prediction.

Ở giai đoạn fix, biến minimal reproduction thành regression test, quan sát nó fail trước tại đúng module interface, sau đó mới áp dụng fix. Trước khi kết thúc, chạy lại scenario ban đầu, xóa log tạm và chương trình debug có unique prefix, đồng thời ghi root cause cuối cùng vào commit hoặc PR.

Quy trình này phù hợp với Bug khó reproduce, performance regression và các vấn đề đã đoán sai vài lần. Compile error hoặc lỗi chính tả field rõ ràng thì không cần dựng trước một bộ quy trình chẩn đoán. Khi project không có test seam phù hợp, minimal reproduction cũng không thể chuyển thành test đáng tin cậy. Skill này sẽ ghi nhận vấn đề architecture, không cố viết một unit test không khớp với cách gọi thực tế.

## `code-review`: tách review coding standards và việc thực hiện yêu cầu

Code review thường chỉ xem implementation quality: tên có rõ ràng không, có logic trùng lặp không, error handling có hợp lý không, test có đủ không. Bản thân code có thể không tìm ra vấn đề lớn, nhưng lại implement sai requirement.

[`code-review`](https://github.com/mattpocock/skills/blob/main/skills/engineering/code-review/SKILL.md) chia review thành hai hướng `Standards` và `Spec`.

![Quy trình code review hai trục của code-review Skill](https://oss.javaguide.cn/github/javaguide/ai/skills/code-review-skill-content.png)

`Standards` sẽ đọc `CONTRIBUTING.md` và coding standards của repository, sau đó kiểm tra thay đổi có tuân thủ quy ước hay không. Version hiện tại còn tích hợp sẵn một nhóm `Fowler Code Smells`. Rule được ghi rõ trong repository được ưu tiên, còn Smell chỉ có thể làm manh mối phán đoán, không được trực tiếp tính là vi phạm.

`Spec` quay lại Issue, PRD hoặc technical plan ban đầu để kiểm tra nội dung bàn giao có thực sự bao phủ requirement gốc hay không. Hai hướng review do các Subagent chạy song song và độc lập hoàn thành, cuối cùng mới merge kết quả để tránh context phụ trách coding style ảnh hưởng đến việc kiểm tra requirement.

Trước khi review cũng cần cố định `commit`, branch, `tag` hoặc `main` làm mốc so sánh. Skill dựa trên `merge base` để xem `diff` từ `HEAD` trở đi, không xem chung chung toàn bộ repository.

Khi project không có PRD, Issue hoặc acceptance criteria, hướng `Spec` chỉ có thể bỏ qua; khi repository không có coding convention, `Standards` sẽ phụ thuộc nhiều hơn vào Code Smells chung. Review song song cũng yêu cầu host Agent hỗ trợ Subagent. CI, static check và manual domain review vẫn cần được giữ lại.

## Cách cài đặt

Repository này có thể được tích hợp vào Codex, Claude Code và các tool hỗ trợ Agent Skills khác thông qua installer của `skills.sh`:

```bash
npx skills@latest add mattpocock/skills
```

Installer sẽ cho bạn chọn Skill cụ thể và Agent đích. Nếu chỉ muốn trải nghiệm requirement interview, trước tiên có thể chọn `grill-me` và `grilling`. Nếu cần duy trì `CONTEXT.md` và ADR trong quá trình thảo luận, hãy chọn thêm `grill-with-docs` và `domain-modeling`.

Theo hướng dẫn hiện tại của project, trước khi sử dụng engineering workflow, còn cần chạy `/setup-matt-pocock-skills` một lần trong repository đích để xác nhận cách quản lý GitHub, Linear hoặc task local, đồng thời xác định label `Triage` và thư mục tài liệu của Agent.

Bạn cũng có thể trực tiếp nhờ Coding Agent cài đặt; ở đây lấy Codex làm ví dụ:

```text
Hãy cài đặt giúp tôi 4 Agent Skill từ repository mattpocock/skills: grilling, research, diagnosing-bugs, code-review.
```

![Codex dùng skill-installer để cài đặt mattpocock skills](https://oss.javaguide.cn/github/javaguide/ai/skills/codex-install-mattpocock-skills.png)

Sau khi cài đặt xong, thường phải đến lượt hội thoại tiếp theo thì chúng mới xuất hiện trong danh sách Skill khả dụng.

`tdd`, `to-spec` và `to-tickets` không được tách thành các mục riêng. TDD, specification và task decomposition đã là các engineering method phổ biến, nhiều Agent cũng có thể hoàn thành version cơ bản. Khi project áp dụng toàn bộ workflow “thảo luận → Spec → Tickets → implementation → review”, hãy kết hợp chúng.

4 Skill được chọn trong bài viết tương ứng với một số điểm thất bại mà hiện nay tôi quan tâm hơn: chưa xác định hướng trước khi bắt đầu, source tài liệu không đáng tin, chưa reproduce Bug đã bắt đầu phỏng đoán, code viết xong nhưng không đối chiếu với requirement gốc.

Lần đầu cài đặt không cần bật global. Trước tiên hãy giới hạn trong một repository, dùng hai hoặc ba task thực tế để quan sát số lần làm lại, thời gian thực thi và chất lượng deliverable. Nếu không có Skill mà model vẫn hoàn thành ổn định thì xóa đi; nếu cùng một vấn đề liên tục xuất hiện thì giữ lại đoạn workflow nhỏ đó.

Skill bên thứ ba là instruction giao cho Agent. Trước khi cài đặt, hãy đọc qua `SKILL.md`, sau đó kiểm tra `scripts/`, `references/` và các yêu cầu permission. Danh sách ngắn hơn không sao; biết vì sao mỗi Skill vẫn còn đó sẽ giúp yên tâm hơn khi sử dụng.
